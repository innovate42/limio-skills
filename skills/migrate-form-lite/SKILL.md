---
name: migrate-form-lite
description: Migrate a React component from @limio/ui-form to @limio/form-lite. Handles import replacement, state management refactoring (two-way binding to explicit one-way flow), validation pattern updates, and form submission changes. Use when asked to migrate a component from ui-form to form-lite.
version: 1.1.0
---

# Migrate ui-form to form-lite

You are migrating a React component from `@limio/ui-form` (Redux two-way binding) to `@limio/form-lite` (explicit one-way data flow).

## Key Principle

form-lite is **state-agnostic**. It handles validation, submission, and browser events — not field state synchronisation. Two approaches coexist:

1. **Uncontrolled (preferred)** — DOM owns state, use `defaultValue`. Only pass `onChange` for side effects.
2. **Controlled** — Redux owns state, pass `value` + `onChange`. Used when other components need to read field values from Redux.

**Decision rule:** Does the parent form have a form-level `onChange` bridge? If yes, prefer uncontrolled — changes reach Redux via event bubbling. If not, or if the component must react immediately to changes, use controlled.

## Reference Documents

Read these before starting:
- `services/shop-components/form-lite/README.md` — form-lite API
- `docs/refactors/current/form-library-unification.md` — architectural decisions

## Existing Patterns to Study

- `services/components/form/index.tsx` — Form wrapper with `handleOrderSubmission` HOC, form-level `onChange` bridge, `defaultValues`
- `services/components/single-page-checkout/CustomerDetails/CustomerDetails.tsx` — Controlled pattern
- `services/components/field/index.tsx` — Generic field wrapper, `readOnly={isSubmitting}`, `FormLiteInput type="select"`
- `services/components/address-fields-hup/index.tsx` — Uncontrolled fields, programmatic DOM updates for address prefill

## 0. Diff the Platform Equivalent First (MANDATORY)

Most client components are forks of a platform component. Before migrating, find the platform equivalent in the monorepo and study how IT was migrated — anything the platform deleted must be deleted, not ported:

```bash
git log --oneline --follow -15 -- services/components/<equivalent>/index.tsx
```

Diff the commits around the form-lite migration. The platform did not just swap imports — it **moved responsibilities to the backend**. Porting the old frontend logic keeps effects the platform intentionally killed and can fight platform mechanisms, causing confusing UI bugs.

Known responsibilities that moved off the frontend (release 115):

- **Delivery→billing copy (LI-8237):** the checkbox is named `setBillingTheSameAsDelivery`; when the order carries `"on"`, backend order normalisation (`packages/limio-data/src/objects/object_types/subscriptions/normalise.ts`) maps delivery to billing at submission, and the payment manager reads the same flag for billing country. The frontend copy effect was deleted; the base `address-fields` simply **hides billing fields** (`return null`) when the flag is set. A client checkbox with a different name (e.g. `sameAsDeliveryCheck`) bypasses ALL of this — rename it and delete the copy logic rather than porting it.
- **`address1` concatenation (LI-10003):** street name + building number concatenation happens in backend order normalisation. Delete the frontend `useEffect` that dispatched `address1`.
- **Country-sync-on-load dispatch (LI-10000):** deleted with no replacement. On form-lite the select's DOM value reaches the order at submit regardless of change events. Also deleted: `locale` state and the country change handler — `useCountrySpecificValidation` reads `useFormField(`${addressType}.country`)` instead.

If the client component has an effect or Redux dispatch with no counterpart in the current platform component, the default is to delete it and let the platform mechanism take over — only keep it if you can name the client-specific behaviour it provides.

## 1. Analyse the Component

Identify these patterns, then classify effort as Low (import swap only), Medium (useForm changes), or High (useFormField with setValue — needs rewrite):

1. `<Form formData={...} onChange={...}>` wrapper
2. `useFormField()` returning `{ value, setValue }`
3. `useForm()` for `isSubmitting`, `wasValidated`, `form`
4. `useFormGroup()` calls
5. Event listeners (`form.addEventListener` / `form.addAsyncEventListener`)
6. `<Select>` component usage
7. `<SubmitButton>` usage
8. `disabled={isSubmitting}` patterns
9. Custom hooks using `useForm`/`useFormField` (e.g. `useHasDeliveryItems`)
10. Third-party UI components inside `FormGroup` (MUI, custom checkboxes)
11. `disableStyling` prop on `FormGroup`
12. `useEffect`s that sync field values to Redux/basket — candidates for deletion

## 2. Import Mapping

```
@limio/ui-form                    @limio/form-lite / @limio/form-context
─────────────────                 ──────────────────────────────────────
Form                          →   FormLite
Input                         →   FormLiteInput
Select                        →   FormLiteInput with type="select"
Label                         →   FormLiteLabel
FormGroup                     →   FormLiteGroup
FormFeedback                  →   FormLiteFeedback
FormSection                   →   FormLiteSection
SubmitButton                  →   FormLiteSubmitButton (uses children, not submitLabel)
useForm                       →   useFormContext (from @limio/form-context)
useFormGroup                  →   useFormGroupContext (from @limio/form-context)
FormPhoneInputField           →   from @limio/form-lite/custom/ (same path structure)
Label.Skeleton                →   FormLiteLabel.Skeleton
Input.Skeleton                →   FormLiteInput.Skeleton
```

### useFormContext API

```typescript
const { form, isSubmitting, wasValidated, submitError, defaultValues, useFormField } = useFormContext()
```

- `useFormField(fieldName)` — returns a string (current DOM value via `useSyncExternalStore`). No `setValue`.
- `form.setCustomValidities({ fieldName: "message" })` — programmatic validation by field name
- `form.getState()` — full form state as nested object
- `form.clearValidity()` / `form.reportValidity()` / `form.requestSubmit()`

## 3. State Management Migration

### Pattern A: Form wrapper — remove formData, use onChange bridge

```typescript
// BEFORE (ui-form)
<Form formData={order} onChange={(updatedOrder) => dispatch(updateOrderAction(updatedOrder))} onSubmit={handleSubmit}>
  <Input name="customerDetails.email" />
</Form>

// AFTER (form-lite) — onChange is now the native form change event
const onChange = React.useCallback((e: React.ChangeEvent<HTMLFormElement>) => {
  const fieldName = e.target.name
  const fieldValue = e.target.type === "checkbox" ? e.target.checked : e.target.value
  dispatch(setOrderPathStateAction({ path: fieldName, value: fieldValue }))
}, [dispatch])

<FormLite onChange={onChange} onSubmit={handleSubmit} defaultValues={prefillData}>
  <FormLiteInput name="customerDetails.email" defaultInvalidMessage="Email is required" />
</FormLite>
```

FormLiteInput auto-resolves `defaultValue` from the form's `defaultValues` using the field `name` as a dot-separated path. You only need explicit `defaultValue` when the parent form doesn't provide `defaultValues`.

### The onChange bridge — what it means for child components

If the parent form already has a form-level `onChange` bridge (e.g. `LimioForm`), child fields **do not need their own onChange handlers for Redux sync**. Changes bubble up. This means:

- **Delete** field-level `onChange` callbacks that only dispatched to Redux (e.g. `handleCountryChange`)
- **Delete** `useEffect`s that synced field values to Redux/basket — they compensated for ui-form's architecture. Remove associated imports (`useBasket`, `useStore`, `updateOrderAction`, etc.)
- **Delete** local state mirroring field values (e.g. `const [locale, setLocale]`) — replace with `useFormField` reading the DOM
- **Delete** props that only supported removed code (e.g. `onBlur` handlers that just forwarded)
- Fields become uncontrolled: `defaultValue` + no `value`, no `onChange`

```typescript
// BEFORE — local state + callback to sync Redux
const [locale, setLocale] = useState(country)
const handleCountryChange = useCallback((e) => {
  dispatch(updateBillingAddressAction({ country: e.target.value }))
  setLocale(e.target.value)
}, [dispatch])

// AFTER — delete callback, delete state, read DOM via useFormField
const { useFormField } = useFormContext()
const addressCountry = useFormField(`${addressType}.country`)

<FormLiteInput type="select" name={`${addressType}.country`} defaultValue={country} />
```

### Pattern B: Controlled fields (when onChange bridge isn't enough)

When a component needs to react immediately to changes (show/hide sections, trigger API calls):

```typescript
const customerDetails = useSelector(state => state.order.customerDetails)

const inputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  const [, field] = e.target.name.split(".")
  dispatch(updateCustomerDetailsAction({ [field]: e.target.value }))
}

<FormLiteInput
  name="customerDetails.firstName"
  value={customerDetails?.firstName ?? ""}
  onChange={inputChange}
  required
  defaultInvalidMessage={t(invalidFirstNameMessage)}
/>
```

### Pattern C: Replace useFormField destructuring

```typescript
// BEFORE — returns { value, setValue }
const { value: sameAsDelivery } = form.useFormField("sameAsDeliveryCheck")

// AFTER — returns the string value directly
const { useFormField } = useFormContext()
const sameAsDelivery = useFormField("sameAsDeliveryCheck")
```

If `setValue` was used: either remove it (DOM handles the value now) or use programmatic DOM updates (Pattern D).

### Pattern D: Programmatic field updates (prefills, address lookup, cross-field sync)

When external logic needs to set field values (address auto-complete, copying delivery→billing):

```typescript
for (const [fieldName, value] of Object.entries(fieldsToUpdate)) {
  const el = document.querySelector(`[name="${fieldName}"]`) || document.getElementById(fieldName)
  if (el) {
    el.value = value || ""
    el.dispatchEvent(new Event("change", { bubbles: true }))
  }
}
```

The event dispatch ensures the form-level `onChange` bridge picks up the change. Use this for prefills and lookups (address autocomplete) only — **do NOT use it for cross-field copy like billing=delivery**. That responsibility moved to backend order normalisation via the `setBillingTheSameAsDelivery` flag (see section 0); porting a frontend copy bypasses the platform mechanism and risks the payment manager reading stale billing data.

### Pattern E: Headless setValue components

Components that only call `setValue()` and render nothing:

```typescript
// BEFORE
function AutoSelectPayment({ paymentType }) {
  const { setValue } = useFormField("paymentType")
  useEffect(() => { setValue(paymentType) }, [])
  return null
}

// AFTER — hidden input, or move to parent form's defaultValues
function AutoSelectPayment({ paymentType }) {
  return <input type="hidden" name="paymentType" value={paymentType} />
}
```

### Pattern F: Checkboxes with non-boolean values

Checkboxes using `setValue("on"/"off")` or inverted logic — use a hidden input:

```typescript
const [checked, setChecked] = useState(true)
const [formValue, setFormValue] = useState("off")

const handleChange = (event) => {
  event.stopPropagation() // CRITICAL: see stopPropagation note below
  setChecked(event.target.checked)
  setFormValue(event.target.checked ? "off" : "on")
}

return (
  <FormLiteGroup>
    <input type="checkbox" checked={checked} onChange={handleChange} />
    <input type="hidden" name={name} value={formValue} />
  </FormLiteGroup>
)
```

**`stopPropagation` on nameless inputs is critical.** The visible checkbox has no `name` — without `stopPropagation`, its change event bubbles to the form's `onChange` bridge, writing `"": false` into the order. DynamoDB rejects empty attribute names → 500 on submission. This applies to **any** nameless interactive element inside a form with an onChange bridge.

For simple checkboxes (value is just "on"/absent), use native behaviour:

```typescript
<FormLiteInput type="checkbox" name={name} defaultChecked={true} defaultInvalidMessage="Required" required />
```

### Pattern G: Form wrapper with local state (non-Redux)

Components using `<Form formData={...} onChange={...}>` with `useState` instead of Redux:

```typescript
// AFTER — form-level onChange updates local state, or read FormData on submit
const onChange = React.useCallback((e) => {
  const { name, value, type, checked } = e.target
  setOrderData(prev => R.assocPath(name.split("."), type === "checkbox" ? checked : value, prev))
}, [])

<FormLite onChange={onChange} onSubmit={handleSubmit} />

// Or simpler: read FormData directly in onSubmit if you don't need live local state
```

### Additional patterns

- **`FormLiteFeedback` with no children** — auto-displays the validation message from `defaultInvalidMessage` or `validation`. No need to duplicate the message.
- **Mixing form-lite layout with design-system inputs** — Use `FormLiteGroup`/`FormLiteLabel` for layout with `Input` from `@limio/design-system` directly when the input doesn't need form validation integration (e.g. consent checkboxes that only trigger side effects).
- **`submitError` for post-submission error handling** — headless components can read `submitError` from `useFormContext()` and call `form.setCustomValidities()` to flag specific fields.
- **Custom wrappers around ui-form primitives** — Some components import a local `FormFeedback` wrapper that uses `useFormGroup` internally. These need migrating too. `FormLiteFeedback` reads `customError` from group context, which covers most cases. If the wrapper uses `dangerouslySetInnerHTML`, that's the one thing `FormLiteFeedback` doesn't handle.

## 4. Validation

The `validation` prop API is identical. Add `defaultInvalidMessage` (the message for native validation failures like `required`):

```typescript
<FormLiteInput
  name="email"
  defaultInvalidMessage="Invalid email"
  validation={(input) => input.value.includes("@") ? "" : "Invalid email"}
  required
/>
<FormLiteFeedback>Invalid email</FormLiteFeedback>
```

Validation runs on submit, not on change. `useCountrySpecificValidation` works identically.

### Combining async `setCustomValidities` with a sync `validation` prop

When a component does async server-side validation (`form.setCustomValidities({ name: msg })` on a failed API call) **and** has a sync `validation` prop for format checks, the two interact in a subtle way.

`FormLiteInput`'s built-in `checkValidity` fires on blur (and on form validate events). It runs the `validation` prop:
- Non-empty return → `setCustomValidity(message)` — input stays invalid
- Empty return → `setCustomValidity("")` — which **clears** any previously-set async custom validity

Result: the first time the user blurs the field after an async-set error, the error wipes even though the value hasn't changed.

**Fix:** preserve async-set validity in the `validation` prop; clear explicitly in `onChange`.

```tsx
const emailValidation = (input: HTMLInputElement): string => {
  if (!validateEmail(input.value)) return invalidMessage
  // Preserve async-set custom validity across blur (b2b user, blocked products, etc.)
  if (input.validity.customError && input.validationMessage) {
    return input.validationMessage
  }
  return ""
}

<FormLiteInput
  validation={emailValidation}
  onChange={(e) => {
    // User edited the value — clear async error so they get a fresh API check on resubmit
    e.target.setCustomValidity("")
  }}
/>
```

Pass the raw tenant prop to `fail()` (e.g. `fail(b2bUserValidityMessage || "")`) — don't invent hardcoded fallbacks. If the tenant cleared the CMS field, `setCustomValidity("")` marks the field valid and no inline error shows, which matches the tenant's intent.

**Preserve the original's error-bucket-to-message mapping.** If the pre-migration component routed errors through an internal bucket (e.g. `setError("validation")` → `usersApiNotSuccessfulValidationMessage` via a `useSetValidity` hook or `getFeedbackMessage()` switch), trace each bucket to the prop it resolved to and pass that same prop to `fail()` for that branch. Don't swap in the "semantically more obvious" prop (e.g. `invalidMessage` for a format-invalid branch) just because it reads better — that silently changes tenant-facing copy. If the original mapped `!validateEmail(email)` → `"validation"` → `usersApiNotSuccessfulValidationMessage`, the migrated `fail()` call for that branch must also use `usersApiNotSuccessfulValidationMessage`.

## 5. Select Migration

`<Select>` → `<FormLiteInput type="select">`. Add empty `<option value="">` for required selects:

```typescript
<FormLiteInput type="select" name="country" required defaultValue={country} defaultInvalidMessage="Please select a country">
  {required && <option value=""></option>}
  {countries.map(c => <option key={c.code} value={c.code}>{c.name}</option>)}
</FormLiteInput>
```

## 6. Event Listeners

Same API, different hook: `useForm()` → `useFormContext()`. The `form.addEventListener`/`addAsyncEventListener` signatures are identical.

## 7. Submission and Controls

**SubmitButton:** `<SubmitButton submitLabel="Buy" />` → `<FormLiteSubmitButton>Buy</FormLiteSubmitButton>`. Props: `children`, `className`, `disabled`, `variant`. Shows `LoadingSpinner` automatically.

**isSubmitting:** `disabled={isSubmitting}` → `readOnly={isSubmitting}`. Preserves value in FormData.

**If the component uses async `setCustomValidities()` during submit:** use `inert={isSubmitting}` instead. Some browsers treat `readOnly` (and `disabled`) as barring the element from constraint validation — `setCustomValidity` silently does nothing, no `invalid` event fires, `FormLiteFeedback` never displays the error. `inert` locks interaction without touching validity and is accessible (skipped by the a11y tree).

Apply the default Limio inert visual treatment via a class name on the input, and tell the user to paste the matching CSS snippet into the page builder's custom CSS:

```tsx
<FormLiteInput
  inert={isSubmitting}
  className={isSubmitting ? "email-field-submitting" : undefined}
  ...
/>
```

Page-builder CSS the user needs to add (class-name scope lets the `!important` flags beat Limio's own `!important` rules that otherwise override inline `style` and plain class rules):

```css
/* set the field to look disabled during submission */
.lmo .email-field-submitting {
  background-color: hsl(var(--tw-form-disabled)) !important;
  opacity: 0.5 !important;
}
```

Don't ship this CSS inside the custom component — the build pipeline strips non-class selectors and can't reliably beat Limio's `!important` rules from a component stylesheet. Inline `style` also loses to those rules. Class name + page-builder CSS is the confirmed working combination.

`readOnly={true}` remains valid for permanently locked fields that don't use async validity (display-only prefills, fields the tenant config has explicitly marked read-only). The `inert` + styling workaround only applies when async `setCustomValidities` needs to survive the "submitting" state.

**form-order HOC:** Works with both. Remove `formData`, onChange becomes native event bridge:

```typescript
const OrderForm = handleOrderSubmission(FormLite)
<OrderForm onChange={onChange} onSubmitError={handleError} defaultValues={prefillData} />
```

**Direct submission:** `e.value.formData` → `new FormData(e.target)` or use `form.getState()`.

**defaultValues:** FormLiteInput auto-resolves from parent form's `defaultValues` via dot-path lookup. Individual `defaultValue` only needed when overriding or when parent doesn't provide `defaultValues`.

## 8. Common Mistakes

1. **Don't pass `formData`** — use `defaultValues` or per-field `defaultValue`
2. **Don't destructure `useFormField`** — returns a string, not `{ value, setValue }`
3. **Don't use `disabled={isSubmitting}`** — use `readOnly={isSubmitting}`, or `inert={isSubmitting}` if the component uses async `setCustomValidities()` (see section 7)
4. **Don't forget `defaultInvalidMessage`** — required prop on FormLiteInput
5. **Don't use `<Select>`** — use `<FormLiteInput type="select">`
6. **`stopPropagation` on nameless inputs** — prevents `"": value` in the order object → DynamoDB 500
7. **Watch for local wrappers** — custom `FormFeedback` or `Input` wrappers using `useFormGroup`/`useFormField` internally need migrating too
8. **`pattern={regex || undefined}`** — if the tenant's regex prop defaults to `""`, React renders `<input pattern="">` which becomes `^(?:)$` (matches empty only). Any non-empty value fails validation with "Please match the format requested." Always fall back to `undefined` when the prop is empty.
9. **`readOnly`/`disabled` can bar `setCustomValidity` in some browsers** — `setCustomValidity(msg)` silently no-ops and no `invalid` event fires. Use `inert={isSubmitting}` if the component does async validity.
10. **`fail("")` silently clears custom validity** — `setCustomValidity("")` marks the field valid (the documented clear behaviour). If a branch might receive an empty tenant config prop, the user sees no inline error even though submission is blocked. Acceptable if it matches tenant intent; otherwise confirm a non-empty default in package.json.
11. **Don't add `@limio/form-lite` to the component's `package.json` dependencies** — it is resolved at build time and does not need to be declared. Adding it causes the component build to fail.

## 9. Browser Validation Tooltip (only implement if the user asks)

ui-form's `<Form>` element had `onInvalid={(e) => e.preventDefault()}`, which suppressed the browser's native validation tooltip. form-lite's form-level `onInvalid` only tracks for the data layer — no `preventDefault`. Result: the native tooltip bubble can appear alongside inline `FormLiteFeedback`, visually duplicating the error message.

**Do not implement a fix by default.** This discrepancy is a form-lite library gap; the skill does not edit form-lite.

If the user explicitly asks for the native tooltip to be suppressed, the per-component fix is a `useEffect` attaching a DOM listener to the input:

```tsx
useEffect(() => {
  const input = document.getElementById(componentId) as HTMLInputElement | null
  if (!input) return
  const handler = (e: Event) => e.preventDefault()
  input.addEventListener("invalid", handler)
  return () => input.removeEventListener("invalid", handler)
}, [componentId])
```

**Before implementing, raise this caveat with the user:**

> This fix only suppresses the native tooltip for this one custom component. Other fields on the same page — First Name Field, Last Name Field, and other out-of-the-box form subcomponents — will still trigger the browser's native tooltip for their own invalid states. Tooltips will appear/disappear inconsistently depending on which field is invalid. Fixing this globally would need a change to form-lite itself, which is out of scope for a migration. The user either accepts the inconsistency, or leaves the tooltip in place until form-lite is patched.

## 10. Sprint Version Compatibility (ask up front)

Before starting the migration, ask the user what sprint version the customer's shop deployment is on. The answer determines whether the migrated component can be tested on its own, needs a test wrapper, or whether the form-lite API surface even covers what the migration needs.

- **Sprint 115 or later:** the main checkout form (`services/components/form`) is already on form-lite. The migrated component drops into the existing form — no wrapper needed.
- **Sprint 114:** the main checkout form is still ui-form, but `@limio/form-lite` and `@limio/form-context` exist with the full API (`form.setCustomValidities`, `addAsyncEventListener`, etc.). A form-lite child inside the ui-form LimioForm throws `form.registerFormElement is not a function` at mount. For end-to-end testing, a **test wrapper** needs to be deployed alongside the migrated component.
- **Sprint 106–113 (older customers):** `@limio/form-lite` exists but its API surface is smaller. Confirm what the migration relies on before committing:
  - `form.setCustomValidities({ name: msg })` — **not available**. Swap to direct DOM validity: `document.getElementById(componentId).setCustomValidity(msg)`. Pair with `form.reportValidity()` to surface the inline message, or call the validation prop's path so `FormLiteGroup`'s `setCustomMessage` updates.
  - `form.getState()` — not available. Read state from Redux (`store.getState()`) or `new FormData(formEl)`.
  - `useFormField(name)` — not in the context at these versions. Read via DOM (`document.querySelector`) or Redux selector.
  - `submitError`, `defaultValues` on the context — not available. Work from Redux or props.
  - `addAsyncEventListener` / `addEventListener` — **available** from v106 onward, so the wrapper's submit-time async validation pattern still works.
  - The wrapper itself (section below) still mounts correctly: `<FormLite>` renders, child form-lite components plumb via the local `FormGroupContext`, and `setOrderAction + R.assocPath` is the correct dispatch shape for these versions.

  In short: a migrated component using the newer `form.setCustomValidities` API needs a small per-call shim (direct `input.setCustomValidity`) before it will run on v106–v113. Check each form-lite API call against the target version's `services/shop-components/form-lite/FormLite.js` before deploying.

### Building the test wrapper (pre-115 only)

Copy the 115 `services/components/form/index.tsx` into a new custom component folder. Adapt for pre-115 runtime:

- `setOrderPathStateAction` doesn't exist pre-115. Replace with `setOrderAction` and build the new order with `R.assocPath`:

  ```tsx
  import { useDispatch, useStore, setOrderAction } from "@limio/shop-redux/src/shop/redux"
  import * as R from "ramda"

  const onChange = React.useCallback((e) => {
    const fieldName = e.target.name
    if (!fieldName) return
    const fieldValue = e.target.type === "checkbox" ? e.target.checked : e.target.value
    const { order } = store.getState()
    const nextOrder = R.assocPath(fieldName.split("."), fieldValue, order)
    dispatch(setOrderAction(nextOrder))
  }, [dispatch, store])
  ```
- Drop imports that don't resolve in custom bundles (see section 11):
  - `handleOrderSubmission` HOC from `@limio/form-order` → use `<FormLite>` directly with a local `onSubmit` + `onPostSubmit`
  - `uiMessageForError` from `@limio/shop-exception` → inline `checkoutError.message`
  - `mergeTrackingTags` from `@limio/tracking` → inline simpler URL construction
- Swap `@limio/shop` imports for `@limio/shop-redux` (re-exports `useDispatch`/`useStore` and is externalized at runtime)

Package the wrapper with empty `dependencies: {}` and list the migrated component in its `limioSubcomponents`.

### Deploying and using the test wrapper

Tell the user to:

1. Build and deploy both the wrapper and the migrated field as custom components
2. In the Limio page builder, swap the page's `Form` component for the custom wrapper
3. Add the migrated field as a subcomponent inside the wrapper
4. **Caveat:** without `PaymentManager` and `handleOrderSubmission`, real checkout submission won't complete. The wrapper is suitable for validating the migrated component's logic (async API calls, error display, custom validity) but not the full payment flow. For end-to-end production testing, the customer needs to be on 115.

## 11. Deployment Caveats for Custom Components

Applies to any form-lite component deployed as a standalone custom component to a Limio customer.

- **Empty `"dependencies": {}`** in the custom component's `package.json` is the working pattern. `workspace:^` deps fail the shop-build's `npm install` step when `USE_ASSET_NPMISTALL` is set.
- **`@limio/form-lite` is not externalized** in the component-bundler's webpack config. Each custom component bundles its own copy. Within a single component, `FormLiteGroup`/`FormLiteInput`/`FormLiteFeedback` communicate correctly (same bundle). Across components, each bundle has its own `FormGroupContext` object — only the shared `@limio/form-context` binds them at the form level.
- **Packages whose `"main"` points at TypeScript source can't be imported into custom components.** Webpack has no `.ts`/`.tsx` in resolve.extensions for custom bundling. `@limio/form-order` (`main: ./src/OrderForm.tsx`) and `@limio/shop-exception` (`main: ./src/index.ts`) fall into this category. Either inline the helpers you need or rewrite to avoid the dependency.
- **CSS inside custom components has non-class selectors stripped by the build pipeline.** Attribute selectors like `input[inert]` don't reliably attach. Stick to class selectors in CSS; if even class-based CSS won't attach, fall back to inline `style` on the React element.

## 12. Checklist

**Before starting:**
- [ ] Diff the platform equivalent component pre/post its form-lite migration (section 0). List every effect/dispatch the platform deleted — delete them in the client component too, don't port them
- [ ] If the component has a same-as-delivery checkbox, rename it to `setBillingTheSameAsDelivery`, delete the copy logic, and hide the billing fields when checked (section 0)
- [ ] Ask the user what sprint version the customer's deployment is on. If pre-115, plan a test wrapper alongside the migrated component (section 10). If pre-114 (v106–v113), also check every form-lite API the migration needs against that version's `services/shop-components/form-lite/FormLite.js` — `form.setCustomValidities`, `form.getState`, `useFormField`, `submitError`, and `defaultValues` may be missing and need shims

**Imports:**
- [ ] `@limio/ui-form` → `@limio/form-lite` / `@limio/form-context`
- [ ] `FormPhoneInputField` from `@limio/form-lite/custom/` if used

**Form wrapper:**
- [ ] `<Form>` → `<FormLite>`, remove `formData`, replace `onChange` with native event handler
- [ ] Add `defaultValues` if needed. Update `onSubmit` if it read `e.value.formData`

**Fields:**
- [ ] Check parent for onChange bridge → prefer uncontrolled
- [ ] `<Input>` → `<FormLiteInput>`, `<Select>` → `<FormLiteInput type="select">`
- [ ] Add `defaultInvalidMessage`. Replace `disabled={isSubmitting}` with `readOnly={isSubmitting}` — or `inert={isSubmitting}` (plus the default inert styling, section 7) if the component does async `setCustomValidities()`
- [ ] `pattern={regex || undefined}` — never render `pattern=""`
- [ ] If combining `validation` prop with async `setCustomValidities`: guard `validation` to preserve `customError`, and explicitly clear in `onChange` (section 4)
- [ ] Delete field-level Redux dispatch handlers covered by the bridge
- [ ] Delete `useEffect`s that only synced values to Redux/basket. Remove dead imports/props/state

**State:**
- [ ] Remove `{ value, setValue }` destructuring → `const val = useFormField("name")`
- [ ] Replace `setValue` calls with DOM updates + `dispatchEvent(new Event("change", { bubbles: true }))`

**Layout:** Apply import mapping table from Section 2 (FormGroup→FormLiteGroup, etc.)

## 13. Execution

1. Ask the user what sprint version the customer's deployment is on. If pre-115, flag that a test wrapper will be needed for end-to-end testing (section 10)
2. Diff the platform equivalent pre/post migration (section 0) — build the list of effects/dispatches the platform deleted
3. Read the target component thoroughly
4. Check parent form for onChange bridge → determines controlled vs uncontrolled
5. Classify effort (Low/Medium/High)
6. Present migration plan to user, including which effects will be deleted (not ported) per the section 0 diff
7. Migrate imports
8. Migrate form wrapper (remove formData, handle onChange, add defaultValues)
9. Migrate fields (defaultInvalidMessage, `inert`/`readOnly`/`value`/`defaultValue`, `pattern` fallback)
10. Migrate selects (`<FormLiteInput type="select">`)
11. Rewrite useFormField usage (remove destructuring, handle setValue removal)
12. If the component uses async `setCustomValidities`, apply the preservation pattern in the `validation` prop and the clear-on-change (section 4)
13. Delete redundant code (useEffects, Redux sync callbacks, dead imports) — everything on the section 0 deletion list
14. Update event listeners (hook import only)
15. Run tests, review with user
