# Business Logic Flaw Allows Reuse of Discount Coupons and Purchase of Hidden Product - Halloween Store (Intigriti Challenge 1026)

## Overview

This challenge can be solved by combining two business logic issues: exposure of a hidden product through a publicly accessible API and improper coupon validation that allows the same discount code to be redeemed multiple times using different letter casing.

By abusing these flaws, it is possible to significantly reduce the price of a hidden product, purchase it, and obtain the challenge flag.

## Enumeration

Upon visiting the homepage, a coupon code is immediately disclosed:

```text
NIGHT20
```

Further enumeration of the application's functionality revealed a newsletter subscription feature. Submitting an email address returned an additional discount code:

```json
{
  "code": "NEWS30"
}
```

At this stage, two valid coupon codes were available for testing:

```text
NIGHT20
NEWS30
```

## Source Code Analysis

Inspecting the client-side JavaScript file:

```text
/static/js/app.js
```

revealed several API endpoints used by the application, including:

```text
/api/products
/api/cart/items
```

Since the product endpoint was accessible without restrictions, it was possible to enumerate all available products.

## Discovering the Hidden Product

Sending a request to the products endpoint:

```http
GET /api/products
```

returned a hidden product that was not visible through the normal user interface:

```json
{
  "id": "midnight-vault",
  "name": "The Midnight Collection",
  "subtitle": "One box. Made only once.",
  "description": "Our numbered Halloween archive: rare fruit jellies, black cocoa truffles and hand-wrapped midnight caramels in a keepsake chest.",
  "price": 66600,
  "weight": "2.4 kg",
  "category": "Gift boxes",
  "image": 5,
  "badge": "Only one available"
}
```

The important value here is the product identifier:

```text
midnight-vault
```

## Adding the Hidden Product to the Cart

Although the product was hidden from the frontend, the backend accepted direct requests referencing the product ID.

A request was sent to:

```http
POST /api/cart/items
```

with the following payload:

```json
{
  "productId": "midnight-vault"
}
```

The product was successfully added to the shopping cart, demonstrating that no server-side restrictions existed to prevent access to hidden products.

## Coupon Validation Bypass

The next step was analyzing how coupon validation was performed.

The application accepted both:

```text
NIGHT20
NEWS30
```

Testing showed that coupons were handled in a case-insensitive manner. However, uniqueness checks were not properly enforced.

For example:

```text
NEWS30
NeWS30
```

and

```text
NIGHT20
nIGHT20
```

were accepted as separate submissions despite representing the same coupon value.

By repeatedly applying differently cased versions of the same coupon, multiple discounts could be stacked together.

This significantly reduced the final price of the hidden product.

## Obtaining the Flag

After adding **The Midnight Collection** to the cart and repeatedly applying the available coupons using different letter casing, the discounted purchase could be completed successfully.

The challenge then revealed the flag:

```text
INTIGRITI{094638e6-ba2f-4c50-8bb4-41ced0d5debd}
```

## Root Cause

The challenge contains two business logic flaws:

1. Hidden products are exposed through a public API endpoint.
2. Coupon validation is case-insensitive but fails to prevent duplicate redemption after normalization.

## Impact

An attacker can:

- Discover hidden or unpublished products.
- Add restricted products directly to the cart.
- Reuse discount coupons multiple times.
- Stack discounts to reduce product prices beyond intended limits.
- Bypass business restrictions enforced by the frontend.

These issues undermine pricing controls and the intended purchasing workflow.
