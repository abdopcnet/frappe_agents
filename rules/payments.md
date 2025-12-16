# Payments App - Development Guide

## Overview

Payments app provides payment gateway integrations for Frappe Framework. Supports multiple payment gateways (Razorpay, Stripe, PayPal, Paytm, Braintree) for accepting online payments.

For general Frappe guidelines, see `frappe.md`. For general AI guidelines, see `AGENTS.md`.

---

## 📋 Table of Contents

1. [App Architecture](#app-architecture)
2. [Supported Payment Gateways](#supported-payment-gateways)
3. [Payment Flow](#payment-flow)
4. [Integration Patterns](#integration-patterns)
5. [Common Use Cases](#common-use-cases)
6. [Developer Notes](#developer-notes)

---

## App Architecture

### Application Structure

```
/home/frappe/frappe-bench/apps/payments/
├── payments/
│   ├── hooks.py                     # App hooks
│   ├── payments/                    # Payments module
│   │   └── doctype/
│   │       └── payment_gateway/     # Payment Gateway DocType
│   ├── payment_gateways/            # Payment Gateways module
│   │   └── doctype/
│   │       ├── razorpay_settings/
│   │       ├── stripe_settings/
│   │       ├── paypal_settings/
│   │       ├── paytm_settings/
│   │       ├── braintree_settings/
│   │       └── gocardless_settings/
│   ├── utils/                       # Utility functions
│   │   ├── utils.py                # Core utilities
│   │   └── __init__.py             # Exported functions
│   ├── overrides/                   # DocType overrides
│   │   └── payment_webform.py      # Web Form payment override
│   ├── templates/                   # Checkout pages
│   └── public/                      # Client-side JS
└── README.md
```

### Key Modules

**1. Payments Module:**
- `Payment Gateway` DocType - Creates gateway links and configuration

**2. Payment Gateways Module:**
- Gateway-specific settings DocTypes
- Integration controllers
- API wrappers

**3. Utils:**
- Common payment functions
- Gateway controller factory
- Custom field management

---

## Supported Payment Gateways

### 1. Razorpay

**Location:** `/home/frappe/frappe-bench/apps/payments/payments/payment_gateways/doctype/razorpay_settings/`

**Features:**
- One-time payments
- Subscription/recurring payments
- Order creation
- Payment capture (manual/automatic)
- Webhooks support
- Addons for subscriptions

**Supported Currencies:** INR (primary), USD, EUR, GBP, AUD, CAD, SGD, AED, etc.

**Setup Fields:**
```python
{
    "api_key": "rzp_test_XXXX",
    "api_secret": "API Secret (encrypted)"
}
```

### 2. Stripe

**Location:** `/home/frappe/frappe-bench/apps/payments/payments/payment_gateways/doctype/stripe_settings/`

**Features:**
- One-time payments
- Subscription management
- Payment intents
- Webhook support
- Strong customer authentication (SCA)

**Supported Currencies:** USD, EUR, GBP, INR, AUD, CAD, JPY, CHF, SEK, NOK, DKK, MXN, BRL, HKD, etc.

**Minimum Charge Amounts:**
```python
{
    "USD": 0.50,
    "EUR": 0.50,
    "GBP": 0.30,
    "JPY": 50,
    "INR": 0.50
}
```

### 3. PayPal

**Location:** `/home/frappe/frappe-bench/apps/payments/payments/payment_gateways/doctype/paypal_settings/`

**Features:**
- Express checkout
- Recurring payments/subscriptions
- IPN (Instant Payment Notification)
- Billing agreements

**Modes:**
- Sandbox (testing)
- Live (production)

**Setup Fields:**
```python
{
    "api_username": "PayPal API username",
    "api_password": "API password",
    "signature": "API signature"
}
```

### 4. Paytm

**Location:** `/home/frappe/frappe-bench/apps/payments/payments/payment_gateways/doctype/paytm_settings/`

**Features:**
- One-time payments
- Checksum validation
- Mobile wallet integration

**Primary Currency:** INR

### 5. Braintree

**Location:** `/home/frappe/frappe-bench/apps/payments/payments/payment_gateways/doctype/braintree_settings/`

**Features:**
- Credit card payments
- PayPal integration
- Vault payment methods

### 6. GoCardless

**Location:** `/home/frappe/frappe-bench/apps/payments/payments/payment_gateways/doctype/gocardless_settings/`

**Features:**
- Direct debit payments
- Mandate management
- Webhooks

---

## Payment Flow

### Standard Payment Process

```
1. User initiates payment
   ↓
2. Create Integration Request (log)
   ↓
3. Create order/transaction with gateway
   ↓
4. Redirect user to gateway checkout
   ↓
5. User completes payment
   ↓
6. Gateway callback/webhook
   ↓
7. Verify payment signature
   ↓
8. Update Integration Request status
   ↓
9. Call reference doctype's on_payment_authorized()
```

### Integration Request DocType

**Purpose:** Tracks all payment transactions

**Key Fields:**
```python
{
    "integration_request_service": "Razorpay/Stripe/etc",
    "status": "Queued/Authorized/Completed/Failed",
    "reference_doctype": "Payment Request/Sales Order",
    "reference_docname": "PR-00001",
    "data": "JSON data",
    "error": "Error message if failed"
}
```

**Status Flow:**
- `Queued` → Payment initiated
- `Authorized` → Payment authorized (not captured)
- `Completed` → Payment captured/completed
- `Failed` → Payment failed

---

## Integration Patterns

### 1. Basic Payment Integration

**Step 1: Get Gateway Controller**
```python
from payments.utils import get_payment_gateway_controller

# Get controller for specific gateway
controller = get_payment_gateway_controller("Razorpay")
```

**Step 2: Validate Currency**
```python
# Check if gateway supports currency
controller().validate_transaction_currency("INR")
```

**Step 3: Create Payment**
```python
payment_details = {
    "amount": 600,  # In currency units (INR 600)
    "title": "Payment for Order #111",
    "description": "Payment via cart",
    "reference_doctype": "Sales Order",
    "reference_docname": "SO-00001",
    "payer_email": "customer@example.com",
    "payer_name": "John Doe",
    "order_id": "111",
    "currency": "INR",
    "payment_gateway": "Razorpay"
}

# Get payment URL
url = controller().get_payment_url(**payment_details)

# Redirect user to this URL
frappe.local.response["type"] = "redirect"
frappe.local.response["location"] = url
```

**Step 4: Handle Callback**

In reference DocType (e.g., Sales Order):
```python
def on_payment_authorized(self, payment_status):
    """Called when payment is completed."""
    if payment_status == "Completed":
        self.payment_status = "Paid"
        self.save()
        # Send confirmation email, update inventory, etc.
```

### 2. Razorpay Integration (Client-Side)

**Include Script:**
```html
{{ include_script('/assets/payments/js/razorpay.js') }}
```

**Backend - Create Order:**
```python
def get_razorpay_order(self):
    from payments.utils import get_payment_gateway_controller
    
    controller = get_payment_gateway_controller("Razorpay")
    
    payment_details = {
        "amount": self.total_amount,
        "currency": "INR",
        "receipt": self.name,
        "reference_doctype": self.doctype,
        "reference_docname": self.name
    }
    
    return controller.create_order(**payment_details)
```

**Frontend - Show Checkout:**
```javascript
frappe.provide('payments.razorpay_checkout');

payments.razorpay_checkout = class RazorpayCheckout {
    constructor(opts) {
        this.doctype = opts.doctype;
        this.docname = opts.docname;
    }
    
    show() {
        frappe.call({
            method: 'get_razorpay_order',
            doc: frappe.get_doc(this.doctype, this.docname),
            callback: (r) => {
                const options = {
                    key: r.message.key,
                    amount: r.message.amount,
                    currency: r.message.currency,
                    order_id: r.message.order_id,
                    name: "My Company",
                    description: "Payment for " + this.docname,
                    handler: (response) => {
                        this.on_payment_success(response);
                    }
                };
                
                const rzp = new Razorpay(options);
                rzp.open();
            }
        });
    }
    
    on_payment_success(response) {
        // Handle success
    }
};
```

### 3. Subscription/Recurring Payments

**Razorpay Subscription:**
```python
payment_details = {
    "amount": 1000,
    "currency": "INR",
    "reference_doctype": "Subscription",
    "reference_docname": "SUB-00001",
    "subscription_details": {
        "plan_id": "plan_xxxxx",          # Razorpay plan ID
        "customer_notify": 1,
        "quantity": 1,
        "total_count": 12,                # 12 billing cycles
        "start_date": "2025-01-01",
        "billing_period": "Month",        # Day/Week/Month/Year
        "billing_frequency": 1,           # Every 1 month
        "upfront_amount": 1000            # First payment
    }
}

url = controller().get_payment_url(**payment_details)
```

**Addons (Additional Charges):**
```python
controller().setup_addon(
    settings=razorpay_settings,
    subscription_id="sub_xxxxx",
    addons=[
        {
            "item": {
                "name": "Upgrade",
                "amount": 500,
                "currency": "INR",
                "description": "Plan upgrade"
            },
            "quantity": 1
        }
    ]
)
```

### 4. Web Form Payments

**Enable Payment on Web Form:**
```python
# Web Form DocType has custom fields:
{
    "accept_payment": 1,
    "payment_gateway": "Razorpay",
    "amount": 1000,
    "currency": "INR"
}
```

**Override in hooks.py:**
```python
# hooks.py
override_whitelisted_methods = {
    "frappe.website.doctype.web_form.web_form.accept": 
    "payments.overrides.payment_webform.accept"
}
```

**Automatic Payment Flow:**
- User submits web form
- Payment page shown if `accept_payment` enabled
- After payment, form submission completes

---

## Common Use Cases

### 1. E-commerce Checkout

```python
@frappe.whitelist()
def create_payment(order_id):
    order = frappe.get_doc("Sales Order", order_id)
    
    controller = get_payment_gateway_controller("Razorpay")
    
    payment_details = {
        "amount": order.grand_total,
        "currency": order.currency,
        "payer_email": order.customer_email,
        "payer_name": order.customer_name,
        "order_id": order.name,
        "reference_doctype": "Sales Order",
        "reference_docname": order.name
    }
    
    return controller().get_payment_url(**payment_details)
```

### 2. Membership Subscription

```python
def subscribe_member(member):
    controller = get_payment_gateway_controller("Razorpay")
    
    payment_details = {
        "amount": 999,  # Monthly fee
        "currency": "INR",
        "reference_doctype": "Member",
        "reference_docname": member.name,
        "subscription_details": {
            "plan_id": "plan_monthly",
            "start_date": today(),
            "customer_notify": 1
        }
    }
    
    return controller().get_payment_url(**payment_details)
```

### 3. Event Registration Payment

```python
# In Event Participant DocType
def on_submit(self):
    if self.requires_payment:
        self.create_payment_request()

def create_payment_request(self):
    from payments.utils import get_payment_gateway_controller
    
    controller = get_payment_gateway_controller("Stripe")
    
    payment_url = controller().get_payment_url(
        amount=self.registration_fee,
        currency="USD",
        payer_email=self.email,
        reference_doctype=self.doctype,
        reference_docname=self.name
    )
    
    # Send email with payment link
    frappe.sendmail(
        recipients=[self.email],
        subject="Complete Your Registration",
        message=f"Click here to pay: {payment_url}"
    )
```

### 4. Invoice Payment

```python
# Sales Invoice with Payment Gateway
def get_payment_url(self):
    controller = get_payment_gateway_controller(self.payment_gateway)
    
    return controller().get_payment_url(
        amount=self.outstanding_amount,
        currency=self.currency,
        title=f"Payment for {self.name}",
        payer_email=self.customer_email,
        reference_doctype=self.doctype,
        reference_docname=self.name
    )
```

---

## Developer Notes

### Utility Functions

**File:** `/home/frappe/frappe-bench/apps/payments/payments/utils/utils.py`

**Key Functions:**
```python
# Get payment gateway controller
from payments.utils import get_payment_gateway_controller
controller = get_payment_gateway_controller("Razorpay")

# Create Payment Gateway DocType
from payments.utils import create_payment_gateway
create_payment_gateway("Razorpay", settings=razorpay_settings)

# Get checkout URL
from payments.utils import get_checkout_url
url = get_checkout_url(**payment_details)
```

### Custom Fields

**Installation:** Adds custom fields to Web Form for payments
**Uninstallation:** Removes custom fields

**Fields Added:**
```python
{
    "accept_payment": "Check",
    "payment_gateway": "Link to Payment Gateway",
    "amount": "Currency",
    "currency": "Link to Currency"
}
```

### Webhooks

**Razorpay Webhook:**
```python
@frappe.whitelist(allow_guest=True)
def razorpay_subscription_callback():
    # Verify signature
    # Update subscription status
    # Call on_payment_authorized
```

**Stripe Webhook:**
- Automatic webhook handling
- Event-based triggers (payment_intent.succeeded, etc.)

**PayPal IPN:**
```python
@frappe.whitelist(allow_guest=True)
def ipn_handler():
    # Validate IPN request
    # Update payment status
```

### Payment Capture

**Razorpay Auto-Capture:**
```python
# Scheduled task (hourly)
from payments.payment_gateways.doctype.razorpay_settings.razorpay_settings import capture_payment

capture_payment()
```

**Process:**
1. Get all "Authorized" Integration Requests
2. Capture payment via API
3. Update status to "Completed"

### Testing

**Razorpay Test Mode:**
```python
api_key = "rzp_test_XXXXXXXXX"
api_secret = "test_secret_key"
```

**Test Cards:**
- Success: `4111 1111 1111 1111`
- Failure: `4000 0000 0000 0002`

**Stripe Test Mode:**
```python
publishable_key = "pk_test_XXXXXXXXX"
secret_key = "sk_test_XXXXXXXXX"
```

**Test Cards:**
- Success: `4242 4242 4242 4242`
- 3D Secure: `4000 0027 6000 3184`

---

## Best Practices

### 1. Always Validate Currency

```python
controller = get_payment_gateway_controller(gateway)
if not controller().validate_transaction_currency(currency):
    frappe.throw(f"{gateway} does not support {currency}")
```

### 2. Use Integration Request

All payments are logged automatically. Check status:
```python
integration_request = frappe.get_doc("Integration Request", request_name)
if integration_request.status == "Completed":
    # Process order
```

### 3. Handle Callbacks Properly

```python
def on_payment_authorized(self, payment_status):
    """IMPORTANT: This is called by payment gateway."""
    try:
        if payment_status in ["Completed", "Authorized"]:
            # Update document
            self.db_set("payment_status", "Paid")
            
            # Send notifications
            self.notify_payment_success()
            
            # Update inventory, etc.
            self.process_payment()
    except Exception:
        frappe.log_error(frappe.get_traceback())
```

### 4. Secure API Credentials

```python
# Store secrets in DocType password fields
api_secret = doc.get_password("api_secret")

# Never log API credentials
# Never expose in client-side code
```

### 5. Error Handling

```python
try:
    order = controller.create_order(**payment_details)
except Exception as e:
    frappe.log_error(frappe.get_traceback(), "Payment Gateway Error")
    frappe.throw(_("Could not create payment. Please try again."))
```

---

## Installation

```bash
# Install app
bench get-app payments
bench --site [sitename] install-app payments

# Configure payment gateway
# Navigate to: Payment Gateways > Razorpay Settings
# Add API Key and API Secret
# Save

# Test payment
# Create a Payment Request or Sales Order
# Use payment integration
```

---

## Quick Commands

```bash
# Install
bench get-app payments
bench --site [sitename] install-app payments

# Migrate
bench --site [sitename] migrate

# Capture authorized payments (manual)
bench --site [sitename] console
>>> from payments.payment_gateways.doctype.razorpay_settings.razorpay_settings import capture_payment
>>> capture_payment()

# Clear cache
bench clear-cache
```

---

## Important Notes

**Requirements:**
- ✅ Valid payment gateway account (Razorpay/Stripe/etc.)
- ✅ API credentials (API key, secret)
- ✅ SSL certificate for production (HTTPS required)
- ✅ Webhook endpoint configuration (for callbacks)

**Web Form Payments:**
- Automatically adds payment fields to Web Form
- Payment collected before form submission
- Works with all supported gateways

**Subscription Payments:**
- Require plan creation on gateway dashboard
- Use plan ID in subscription_details
- Handle webhook notifications for renewals

**Currency Support:**
- Each gateway has specific currency support
- Always validate currency before payment
- Amount conversion handled automatically (rupee to paisa for Razorpay)

---

**Repository:** https://github.com/frappe/payments  
**License:** MIT  
**Documentation:** https://github.com/frappe/payments/blob/develop/README.md
