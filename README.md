# VELORA STORE - E-Commerce with PayPal Sandbox Integration

A luxury e-commerce web application integrated with **PayPal Sandbox Checkout JS SDK v2**. Features a curated catalog, responsive shopping cart, shipping form validation, itemized order breakdowns, async payment capture, and an instant order confirmation receipt screen.

---

## 📋 Table of Contents

- [Features](#-features)
- [Project Architecture & File Structure](#-project-architecture--file-structure)
- [Environment Configuration](#-environment-configuration)
- [PayPal Integration Details](#-paypal-integration-details)
- [Complete End-to-End Testing Guide](#-complete-end-to-end-testing-guide)
- [Security Audit & Best Practices](#-security-audit--best-practices)

---

## ✨ Features

- **Modern Luxury UI & Typography**: Custom luxury aesthetic using Google Fonts (`Cormorant Garamond` and `DM Sans`) with dark/cream glassmorphism elements.
- **Dynamic Product Catalog**: Filter products by category (*Fragrance, Skincare, Accessories, Home*) with badges and price indicators.
- **Responsive Cart Overlay**: Real-time item count badges, quantity increment/decrement controls, subtotal calculation, 8% tax calculation, and free shipping calculation.
- **Pre-Checkout Validation (`onClick`)**: Blocks PayPal popup initialization if required shipping fields (*First Name, Last Name, Email, Street Address, City, Zip*) or cart items are invalid.
- **PayPal JS SDK v2 Integration**:
  - `intent=capture` and `currency=USD`.
  - Itemized `createOrder` breakdown matching PayPal's v2 schema (`name`, `sku`, `unit_amount`, `quantity`, `tax_total`, `shipping`).
  - `async/await` order capture (`actions.order.capture()`) inside `onApprove`.
  - Double-click idempotency guard (`isCapturing`).
  - Error and cancellation callbacks (`onError` and `onCancel`).
- **Success Receipt Modal**: Displays Order ID, Transaction ID, Payer ID, Customer Name, Customer Email, Amount Paid, Payment Status, Order Date, and Estimated Delivery Date.
- **Clean Checkout Reset**: Clearing cart array, resetting form inputs, destroying/re-rendering PayPal button instances, and smooth scrolling to products.

---

## 📁 Project Architecture & File Structure

```
ecommerce-with-paypal-integration/
├── index.html          # VELORA STORE storefront, cart overlay, login & checkout modals
├── bill.html           # Dedicated LUXE Checkout page with PayPal integration
├── .env                # Local environment variables containing PayPal Sandbox credentials
├── .env.example        # Environment variables template for team setup
├── .gitignore          # Git exclusion rules ensuring .env is never tracked
└── README.md           # Comprehensive project documentation & testing guide
```

---

## ⚙️ Environment Configuration

Credentials are stored in `.env` and kept out of source control via `.gitignore`.

### Sample `.env` Configuration

```env
# REST API App Client ID
PAYPAL_SANDBOX_CLIENT_ID=ATP2ILBhpr6tRsfYc2K74

# Sandbox Account & Business Details
PAYPAL_SANDBOX_ACCOUNT_ID=CPL6AU5WHY79C
PAYPAL_SANDBOX_BUSINESS_EMAIL=sb-43vqum52296077@business.example.com

# NVP/SOAP API Credentials
PAYPAL_SANDBOX_API_USERNAME=sb-43vqum52296077_api1.business.example.com
PAYPAL_SANDBOX_API_PASSWORD=AJKWAJ2LWW34EXWP
PAYPAL_SANDBOX_API_SIGNATURE=AcjqmKeyxz4ba8ALvJJNfDY4ZV7nAuNe9LOLiA63tQeJBqnaAKvjl5ml

# Environment Settings
PAYPAL_CURRENCY=USD
PAYPAL_ENVIRONMENT=sandbox
```

---

## 💳 PayPal Integration Details

### 1. SDK Loading Script
```html
<!-- PayPal Sandbox SDK Script -->
<script src="https://www.paypal.com/sdk/js?client-id=sb&currency=USD&intent=capture"></script>
```
*Note: `client-id=sb` is PayPal's active public Sandbox test ID for client-side browser testing. Replace `sb` with your production/personal Sandbox Client ID from [developer.paypal.com](https://developer.paypal.com).*

### 2. Pre-Validation Guard (`onClick`)
```javascript
onClick: function(data, actions) {
    if (!validateShippingForm()) {
        return actions.reject(); // Prevents PayPal popup from opening
    }
    return actions.resolve();
}
```

### 3. Itemized Order Breakdown (`createOrder`)
```javascript
createOrder: function(data, actions) {
    const subtotal = roundCurrency(getSubtotal());
    const tax = roundCurrency(subtotal * 0.08);
    const grandTotal = roundCurrency(subtotal + tax);

    return actions.order.create({
        intent: 'CAPTURE',
        purchase_units: [{
            description: 'VELORA Store Purchase',
            amount: {
                currency_code: 'USD',
                value: grandTotal.toFixed(2),
                breakdown: {
                    item_total: { currency_code: 'USD', value: subtotal.toFixed(2) },
                    tax_total: { currency_code: 'USD', value: tax.toFixed(2) }
                }
            },
            items: cart.map(item => ({
                name: item.name,
                sku: item.sku || ('SKU-' + item.id),
                unit_amount: { currency_code: 'USD', value: roundCurrency(item.price).toFixed(2) },
                quantity: item.qty.toString(),
                category: 'PHYSICAL_GOODS'
            }))
        }]
    });
}
```

### 4. Async Capture & Receipt (`onApprove`)
```javascript
onApprove: async function(data, actions) {
    const details = await actions.order.capture();
    showSuccess({
        orderId: details.id,
        txnId: details.purchase_units[0].payments.captures[0].id,
        payerId: details.payer.payer_id,
        buyerName: details.payer.name.given_name,
        buyerEmail: details.payer.email_address,
        amountPaid: '$' + details.purchase_units[0].payments.captures[0].amount.value
    });
}
```

---

## 🧪 Complete End-to-End Testing Guide

Follow these steps to perform a complete Sandbox checkout test:

### Step 1: Open Storefront
Open `index.html` in any web browser or local server (e.g. VS Code Live Server / `http://localhost:5500`).

### Step 2: Add Items to Cart
1. Click **+ Add** on **Noir Velour Elixir** ($185.00).
2. Click **+ Add** on **Lumière Renewal Serum** ($220.00).
3. Verify the navigation bar displays `🛍 Cart (2)`.

### Step 3: Open Checkout Modal
1. Click the **Cart 🛍** button to open the side panel overlay.
2. Confirm Subtotal ($405.00), Tax 8% ($32.40), and Total ($437.40).
3. Click **Proceed to Checkout**.

### Step 4: Test Validation Block (Negative Testing)
1. Leave the shipping form empty.
2. Click the yellow **PayPal** button.
3. Verify that a toast error message (`Please fill in all shipping fields.`) appears and the PayPal popup is **blocked**.

### Step 5: Fill Valid Shipping Details
Enter valid shipping inputs:
- **First Name**: Jane
- **Last Name**: Doe
- **Email Address**: `jane.doe@example.com`
- **Street Address**: 123 Luxury Way
- **City**: New York
- **Zip Code**: 10001

### Step 6: Complete PayPal Sandbox Authorization
1. Click the yellow **PayPal** button.
2. In the PayPal popup window, log in with a PayPal Sandbox Personal Buyer account or use Sandbox test credentials:
   - **Buyer Email**: `sb-43vqum52296077@business.example.com`
3. Click **Complete Purchase** or **Pay Now**.

### Step 7: Verify Receipt Screen
1. Confirm the PayPal popup closes automatically.
2. Verify the **Payment Successful** screen displays:
   - **Order ID**: (e.g., `5O15683169829483B`)
   - **Transaction ID**: (e.g., `85V38291LS012984M`)
   - **Status**: `COMPLETED`
   - **Customer Name & Email**
   - **Total Paid**: `$437.40`
   - **Estimated Delivery**: 3-5 Business Days

### Step 8: Test Reset Lifecycle
1. Click **Continue Shopping**.
2. Verify:
   - Shopping cart count resets to `0`.
   - Checkout modal and cart overlay close.
   - Shipping form inputs are cleared.
   - Page scrolls back to top catalog smoothy.

---

## 🔒 Security Audit & Best Practices

- **Git Credential Exclusion**: `.env` containing API credentials is listed in `.gitignore` to prevent secret leaks to public repositories.
- **Float Rounding (`roundCurrency`)**: Math calculations utilize `Math.round((val + EPSILON) * 100) / 100` to prevent IEEE 754 precision drift.
- **Production Backend Recommendation**: For live production deployment, implement a backend server (e.g., Node.js / Express or Python) to handle `/v2/checkout/orders` creation and capture server-side using your Client Secret, eliminating client-side price tampering risks.
