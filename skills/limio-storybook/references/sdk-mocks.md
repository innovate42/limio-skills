# SDK Mock Templates

These mock files provide a complete Limio SDK environment for Storybook development.

## packages/limio/sdk/index.js

```javascript
export * from "./src/context"

export function getPropsFromPackageJson(packageData) {
    const limioProps = packageData.limioProps || []
    const defaults = {}
    limioProps.forEach(prop => {
        if (prop.default !== undefined) {
            defaults[prop.id] = prop.default
        }
    })
    return defaults
}
```

## packages/limio/sdk/src/context.js

```javascript
import * as React from "react"

const LimioContext = React.createContext({})
export const ComponentContext = React.createContext({})

// ===== Mock Data =====

const mockOffers = [
    {
        id: "offer-monthly-001", name: "Monthly Plan", path: "/offers/monthly", type: "item",
        data: {
            attributes: {
                display_name__limio: "Monthly", display_price__limio: "<p>$9.99/mo</p>",
                detailed_display_price__limio: "<p>Billed monthly</p>", cta_text__limio: "Subscribe",
                group__limio: "monthly", best_value__limio: false,
                offer_features__limio: "<ul><li>Unlimited access</li><li>Cancel anytime</li></ul>",
                payment_types__limio: ["card"], checkout_description__limio: "Monthly subscription",
                price__limio: [{ type: "recurring", value: 9.99, currencyCode: "USD" }],
                term__limio: { type: "months", length: 1, renewal_trigger: "auto", renewal_type: "term" },
            },
            price: [{ value: 9.99, currencyCode: "USD", type: "recurring", trigger: "subscription_start", repeat_interval: 1, repeat_interval_type: "months" }],
            products: [{ path: "/products/standard", name: "Standard", attributes: { display_name__limio: "Standard Plan", product_code__limio: "STANDARD" } }],
            attachments: []
        }
    },
    {
        id: "offer-annual-002", name: "Annual Plan", path: "/offers/annual", type: "item",
        data: {
            attributes: {
                display_name__limio: "Annual", display_price__limio: "<p><s>$119.88</s> $99.99/yr</p>",
                detailed_display_price__limio: "<p>Billed annually — save 17%</p>", cta_text__limio: "Subscribe & Save",
                group__limio: "annual", best_value__limio: true, badge_text__limio: "Best Value",
                offer_features__limio: "<ul><li>Unlimited access</li><li>Priority support</li><li>Cancel anytime</li></ul>",
                payment_types__limio: ["card", "paypal"], checkout_description__limio: "Annual subscription",
                price__limio: [{ type: "recurring", value: 99.99, currencyCode: "USD" }],
                term__limio: { type: "years", length: 1, renewal_trigger: "auto", renewal_type: "term" },
            },
            price: [{ value: 99.99, currencyCode: "USD", type: "recurring", trigger: "subscription_start", repeat_interval: 1, repeat_interval_type: "years" }],
            products: [{ path: "/products/standard", name: "Standard", attributes: { display_name__limio: "Standard Plan", product_code__limio: "STANDARD" } }],
            attachments: []
        }
    },
    {
        id: "offer-premium-003", name: "Premium Monthly", path: "/offers/premium", type: "item",
        data: {
            attributes: {
                display_name__limio: "Premium", display_price__limio: "<p>$19.99/mo</p>",
                detailed_display_price__limio: "<p>Billed monthly</p>", cta_text__limio: "Go Premium",
                group__limio: "monthly", best_value__limio: false,
                offer_features__limio: "<ul><li>Everything in Standard</li><li>Advanced analytics</li><li>API access</li><li>Dedicated support</li></ul>",
                payment_types__limio: ["card", "paypal"], checkout_description__limio: "Premium monthly subscription",
                price__limio: [{ type: "recurring", value: 19.99, currencyCode: "USD" }],
                term__limio: { type: "months", length: 1, renewal_trigger: "auto", renewal_type: "term" },
            },
            price: [{ value: 19.99, currencyCode: "USD", type: "recurring", trigger: "subscription_start", repeat_interval: 1, repeat_interval_type: "months" }],
            products: [{ path: "/products/premium", name: "Premium", attributes: { display_name__limio: "Premium Plan", product_code__limio: "PREMIUM" } }],
            attachments: []
        }
    }
]

const mockBasketItems = [
    {
        name: "Monthly Plan", id: "basket-item-001",
        offer: mockOffers[0], details: "",
        price: { summary: { headline: "<p>$9.99/mo</p>" }, currency: "USD", amount: 9.99 },
        products: mockOffers[0].data.products
    }
]

const mockUser = {
    username: "mock-user-001",
    attributes: { email: "alex@example.com", email_verified: true, firstName: "Alex", lastName: "Johnson", sub: "mock-user-001" },
    subscriptions: [
        {
            name: "Pro Plan Monthly", status: "active", record_type: "subscription",
            id: "sub-001", reference: "REF001", created: "2024-01-15T00:00:00Z", mode: "production",
            offers: [{
                name: "Pro Plan", quantity: 1,
                data: {
                    start: "2024-01-15T00:00:00Z", record_subtype: "base",
                    offer: {
                        data: {
                            attributes: { display_name__limio: "Pro Plan", price__limio: [{ type: "recurring", value: 9.99, currencyCode: "USD" }], term__limio: { type: "months", length: 1, renewal_trigger: "auto", renewal_type: "term" } },
                            products: [{ name: "Pro Access", attributes: { display_name__limio: "Pro Access", product_code__limio: "STANDARD" } }]
                        }
                    }
                },
                price: { summary: { headline: "$9.99/mo" }, currency: "USD", amount: 9.99 }, products: []
            }],
            schedule: [
                { id: "sched-001", data: { date: "2024-01-15T00:00:00Z", amount: "9.99", currency: "USD", type: "payment", description: "Pro Plan — Monthly" }, status: "active" },
                { id: "sched-002", data: { date: "2024-02-15T00:00:00Z", amount: "9.99", currency: "USD", type: "payment", description: "Pro Plan — Monthly" }, status: "active" },
                { id: "sched-003", data: { date: "2027-07-15T00:00:00Z", amount: "9.99", currency: "USD", type: "payment", description: "Pro Plan — Monthly" }, status: "active" }
            ]
        },
        {
            name: "Enterprise Annual", status: "active", record_type: "subscription",
            id: "sub-002", reference: "REF002", created: "2024-03-15T09:30:00Z", mode: "production",
            offers: [{
                name: "Enterprise Plan", quantity: 1,
                data: {
                    start: "2024-03-15T09:30:00Z", record_subtype: "base",
                    offer: {
                        data: {
                            attributes: { display_name__limio: "Enterprise Plan", price__limio: [{ type: "recurring", value: 499, currencyCode: "USD" }], term__limio: { type: "years", length: 1, renewal_trigger: "auto", renewal_type: "term" } },
                            products: [{ name: "Enterprise Access", attributes: { display_name__limio: "Enterprise Access", product_code__limio: "ENTERPRISE" } }]
                        }
                    }
                },
                price: { summary: { headline: "$499/year" }, currency: "USD", amount: 499 }, products: []
            }],
            schedule: [
                { id: "sched-010", data: { date: "2024-03-15T09:30:00Z", amount: "499.00", currency: "USD", type: "payment", description: "Enterprise Plan — Annual" }, status: "active" },
                { id: "sched-011", data: { date: "2027-03-15T09:30:00Z", amount: "499.00", currency: "USD", type: "payment", description: "Enterprise Plan — Annual" }, status: "active" }
            ]
        },
        {
            name: "Starter Monthly", status: "cancelled", record_type: "subscription",
            id: "sub-003", reference: "REF003", created: "2023-06-01T08:00:00Z", mode: "production",
            offers: [{
                name: "Starter Plan", quantity: 1,
                data: {
                    start: "2023-06-01T08:00:00Z", end: "2023-12-01T08:00:00Z", record_subtype: "base",
                    offer: {
                        data: {
                            attributes: { display_name__limio: "Starter Plan", price__limio: [{ type: "recurring", value: 4.99, currencyCode: "USD" }], term__limio: { type: "months", length: 1, renewal_trigger: "auto", renewal_type: "term" } },
                            products: [{ name: "Starter Access", attributes: { display_name__limio: "Starter Access", product_code__limio: "STARTER" } }]
                        }
                    }
                },
                price: { summary: { headline: "$4.99/mo" }, currency: "USD", amount: 4.99 }, products: []
            }],
            schedule: [
                { id: "sched-020", data: { date: "2023-06-01T08:00:00Z", amount: "4.99", currency: "USD", type: "payment", description: "Starter Plan — Monthly" }, status: "active" },
                { id: "sched-021", data: { date: "2023-11-01T08:00:00Z", amount: "4.99", currency: "USD", type: "payment", description: "Starter Plan — Monthly" }, status: "cancelled" }
            ]
        }
    ],
    loginStatus: "logged-in", loaded: true, token: "mock-jwt-token"
}

const dummyContext = {
    pageBuilder__limio: false,
    shop: {
        campaign: { name: "Demo Campaign", path: "/campaigns/demo", attributes: { push_to_checkout__limio: true } },
        offers: mockOffers,
        addOns: [],
        tag: "/tags/demo",
        basketItems: mockBasketItems,
        addToBasket: (offer) => console.log("Added to basket:", offer),
    },
    user: mockUser
}

// ===== Hooks =====

export function useCampaign() {
    React.useContext(LimioContext)
    const { campaign, offers, addOns } = dummyContext.shop
    return { campaign, offers, addOns }
}

export function useBasket() {
    React.useContext(LimioContext)
    const { basketItems, addToBasket } = dummyContext.shop
    return {
        orderItems: basketItems, basketLoading: false, formattedTotal: "$9.99",
        initiateCheckout: async (data) => console.log("Checkout initiated:", data),
        addOfferToBasket: async (data) => console.log("Added:", data),
        removeFromBasket: async (data) => console.log("Removed:", data),
        navigateToCheckout: async () => console.log("Navigate to checkout"),
        clearOrderItems: () => console.log("Cart cleared"),
    }
}

export function useUser() {
    React.useContext(LimioContext)
    return mockUser
}

export function useSubscriptions() {
    React.useContext(LimioContext)
    return { subscriptions: mockUser.subscriptions }
}

export function useLimioContext() {
    React.useContext(LimioContext)
    return { isInPageBuilder: false }
}

export function useComponentProps(defaultProps) {
    const context = React.useContext(ComponentContext)
    return React.useMemo(() => ({ ...defaultProps, ...context }), [context, defaultProps])
}

export function useCheckout() {
    return {
        useCheckoutSelector: (callback) => callback({
            order: { orderDate: new Date().toISOString(), basketItems: mockBasketItems, orderItems: mockBasketItems, customerDetails: { firstName: "Alex", lastName: "Johnson", email: "alex@example.com" } },
            display: { orderTotal: { orderSubtotal: "$9.99", orderTotal: "$9.99", currency: "USD", taxSummary: [] } }
        })
    }
}

export function groupOffers(offers = [], groupLabels = []) {
    const groups = {}
    for (const offer of offers) {
        const group = offer?.data?.attributes?.group__limio || "other"
        groups[group] = groups[group] || []
        groups[group].push(offer)
    }
    return Object.keys(groups).map(groupId => {
        const match = groupLabels.find(g => g.id === groupId) || { id: groupId, label: groupId, thumbnail: "" }
        return { groupId, id: groupId, label: match.label, offers: groups[groupId], thumbnail: match.thumbnail }
    })
}

export function formatCurrencyForCurrentLocale(amount, currency) {
    return new Intl.NumberFormat("en-US", { style: "currency", currency }).format(amount)
}

export function ErrorBoundary({ children }) {
    return <>{children}</>
}

// ===== Provider =====

export function LimioProvider({ children, value = dummyContext }) {
    return <LimioContext.Provider value={value}>{children}</LimioContext.Provider>
}
```

## packages/limio/shop/src/shop/checkout/basket.js

```javascript
export function getCurrentBasketId() {
    return "mock-basket-id"
}
```

## packages/limio/internal-checkout-sdk/index.js

```javascript
import { useCheckout } from "@limio/sdk"
export { useCheckout }
```
