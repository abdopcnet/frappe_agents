# ERPNext Reference

## Module Structure

**Core Modules:**

```
erpnext/
├── accounts/        # Financial accounting
├── selling/         # Sales (Quotation, Sales Order, Delivery Note, Sales Invoice)
├── buying/          # Procurement (Purchase Order, Purchase Receipt, Purchase Invoice)
├── stock/           # Inventory management
├── manufacturing/   # Production
├── crm/             # Customer relationship
├── projects/        # Project management
└── controllers/     # Base controllers
```

## Key DocTypes

**Accounts:**

- Sales Invoice, Purchase Invoice, Payment Entry, Journal Entry
- GL Entry, Account, Cost Center, Fiscal Year

**Selling:**

- Quotation → Sales Order → Delivery Note → Sales Invoice

**Buying:**

- Supplier Quotation → Purchase Order → Purchase Receipt → Purchase Invoice

**Stock:**

- Stock Entry, Item, Warehouse, Stock Ledger Entry
- Batch, Serial Number

## Common Controllers

**Base Controllers:**

```python
from erpnext.controllers.accounts_controller import AccountsController
from erpnext.selling.doctype.sales_order.sales_order import SalesOrder

class CustomSalesInvoice(AccountsController):
    def validate(self):
        super().validate()
        # Custom validation
```

## Transaction Flow

**Sales Flow:**

```
Quotation → Sales Order → Delivery Note → Sales Invoice → Payment Entry
```

**Purchase Flow:**

```
Purchase Order → Purchase Receipt → Purchase Invoice → Payment Entry
```

## Common APIs

**Party Operations:**

```python
from erpnext.accounts.party import get_party_account_balance

balance = get_party_account_balance(
    party_type="Customer",
    party="CUST-001",
    company="Company"
)
```

**Item Operations:**

```python
from erpnext.stock.get_item_details import get_item_details, get_item_price

price = get_item_price({
    "item_code": "ITEM-001",
    "price_list": "Standard Selling",
    "customer": "CUST-001"
})
```

**Stock Operations:**

```python
from erpnext.stock.utils import get_stock_balance

balance = get_stock_balance("ITEM-001", "Warehouse")
```

**Currency:**

```python
from erpnext.setup.utils import get_exchange_rate

rate = get_exchange_rate("USD", "EGP", "2025-01-01")
```

## Customization Patterns

**Extend Controllers:**

```python
from erpnext.selling.doctype.sales_invoice.sales_invoice import SalesInvoice

class CustomSalesInvoice(SalesInvoice):
    def validate(self):
        super().validate()
        # Custom logic
```

**Use Hooks:**

```python
# hooks.py
doc_events = {
    "Sales Invoice": {
        "validate": "app.custom.validate_invoice"
    }
}
```

**Override Methods:**

```python
# hooks.py
override_whitelisted_methods = {
    "erpnext.selling.doctype.sales_order.sales_order.make_delivery_note":
        "app.custom.make_custom_delivery_note"
}
```
