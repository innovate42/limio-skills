# limioProps Type Reference

This document covers every `limioProps` type available for Limio components, with complete examples and notes on naming conventions.

---

## String

Basic text input.

```json
{
  "id": "headline",
  "label": "Headline",
  "type": "string",
  "default": "Welcome to Our Plans"
}
```

```json
{
  "id": "ctaUrl",
  "label": "CTA Link URL",
  "type": "string",
  "default": "/checkout"
}
```

---

## Boolean

Toggle switch (true/false).

```json
{
  "id": "showImage",
  "label": "Show image",
  "type": "boolean",
  "default": true
}
```

```json
{
  "id": "showFeatures",
  "label": "Show features list",
  "type": "boolean",
  "default": true
}
```

---

## Number

Numeric input. Note: the `default` value is a string representation of the number.

```json
{
  "id": "cardWidth",
  "label": "Card width",
  "type": "number",
  "default": "2"
}
```

```json
{
  "id": "maxColumns",
  "label": "Max columns",
  "type": "number",
  "default": "3"
}
```

---

## Rich Text (HTML)

HTML content with a rich text editor in the Limio Page Builder. Use `"type": "richtext"` (lowercase). No special suffix is needed on the prop ID.

```json
{
  "id": "description",
  "label": "Description",
  "type": "richtext",
  "default": "<p>Subscribe today and get access to all features.</p>"
}
```

```json
{
  "id": "termsContent",
  "label": "Terms & Conditions",
  "type": "richtext",
  "default": "<p>By subscribing you agree to our <a href='/terms'>terms</a>.</p>"
}
```

**Important:** Always sanitize rich text before rendering:
```javascript
import xss from "xss"
<div dangerouslySetInnerHTML={{ __html: xss(description || "") }} />
```

---

## Color

Color picker input. Use `"type": "color"`. No special suffix is needed on the prop ID.

```json
{
  "id": "primaryColor",
  "label": "Primary color",
  "type": "color",
  "default": "#635BFF"
}
```

```json
{
  "id": "backgroundColor",
  "label": "Background color",
  "type": "color",
  "default": "#f6f9fc"
}
```

**Usage pattern:** Pass color props through CSS custom properties:
```javascript
<div style={{ "--primary": primaryColor }}>
```

---

## DateTime

Date and time picker. Value is an ISO 8601 string.

```json
{
  "id": "expiryDateTime",
  "label": "Expiry date",
  "type": "datetime",
  "default": "2025-12-10T11:30:42.809Z"
}
```

```json
{
  "id": "promotionStart",
  "label": "Promotion start",
  "type": "datetime",
  "default": "2025-01-01T00:00:00.000Z"
}
```

---

## Picklist (Dropdown)

Single-select dropdown with predefined options. Each option has `id`, `label`, and `value`.

```json
{
  "id": "theme",
  "label": "Theme",
  "type": "picklist",
  "options": [
    { "id": "light", "label": "Light", "value": "light" },
    { "id": "dark", "label": "Dark", "value": "dark" },
    { "id": "auto", "label": "Auto (System)", "value": "auto" }
  ],
  "default": "light"
}
```

```json
{
  "id": "layout",
  "label": "Layout style",
  "type": "picklist",
  "options": [
    { "id": "grid", "label": "Grid", "value": "grid" },
    { "id": "list", "label": "List", "value": "list" },
    { "id": "carousel", "label": "Carousel", "value": "carousel" }
  ],
  "default": "grid"
}
```

**Note:** The `default` value should match one of the option `value` fields.

---

## List (Array of Objects)

Configurable array of objects. Each item is an `{id, label}` object (with optional additional fields).

```json
{
  "id": "groupLabels",
  "label": "Group Labels",
  "type": "list",
  "fields": {
    "name": { "id": "id", "label": "ID", "type": "string" },
    "url": { "id": "label", "label": "Label", "type": "string" }
  },
  "default": [
    { "id": "monthly", "label": "Monthly" },
    { "id": "annual", "label": "Annual" }
  ]
}
```

### List with Thumbnail Field

```json
{
  "id": "groupLabels",
  "label": "Group Labels",
  "type": "list",
  "fields": {
    "name": { "id": "id", "label": "ID", "type": "string" },
    "url": { "id": "label", "label": "Label", "type": "string" },
    "thumbnail": { "id": "thumbnail", "label": "Thumbnail", "type": "string", "format": "uri", "purpose": "image" }
  },
  "default": [
    { "id": "monthly", "label": "Monthly" },
    { "id": "annual", "label": "Annual" }
  ]
}
```

### List with More Fields

```json
{
  "id": "navLinks",
  "label": "Navigation Links",
  "type": "list",
  "fields": {
    "name": { "id": "id", "label": "ID", "type": "string" },
    "url": { "id": "label", "label": "Label", "type": "string" },
    "href": { "id": "href", "label": "URL", "type": "string" }
  },
  "default": [
    { "id": "home", "label": "Home", "href": "/" },
    { "id": "pricing", "label": "Pricing", "href": "/pricing" }
  ]
}
```

**Important:** List items are `{id, label}` objects at minimum, not plain strings.

---

## Schema

Schema type for structured data with radio buttons or checkboxes.

### Radio (Single Select)

```json
{
  "id": "alignment",
  "label": "Text Alignment",
  "type": "schema",
  "schema": {
    "type": "string",
    "enum": ["left", "center", "right"],
    "enumNames": ["Left", "Center", "Right"]
  },
  "uiSchema": {
    "ui:widget": "radio"
  },
  "default": "center"
}
```

### Checkbox (Multi Select)

```json
{
  "id": "visibleSections",
  "label": "Visible Sections",
  "type": "schema",
  "schema": {
    "type": "array",
    "items": {
      "type": "string",
      "enum": ["header", "features", "pricing", "footer"],
      "enumNames": ["Header", "Features", "Pricing", "Footer"]
    },
    "uniqueItems": true
  },
  "uiSchema": {
    "ui:widget": "checkboxes"
  },
  "default": ["header", "features", "pricing"]
}
```

---

## Naming Conventions

| Type | Example ID |
|------|------------|
| string | `headline` |
| boolean | `showImage` |
| number | `cardWidth` |
| richtext | `description` |
| color | `primaryColor` |
| datetime | `expiryDateTime` |
| picklist | `theme` |
| list | `groupLabels` |
| schema | `alignment` |

**Key rules:**
- No special suffixes are needed — the `type` field determines the Page Builder control (rich text editor, color picker, etc.)
- Use camelCase for prop IDs
- Use human-readable strings for labels
