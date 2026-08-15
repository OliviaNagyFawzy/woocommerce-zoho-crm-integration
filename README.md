# WooCommerce → Zoho CRM Integration

A local integration between a WooCommerce store (running on WordPress via LocalWP) and Zoho CRM. When a customer places an order, the integration automatically creates a **Contact** and a linked **Deal** in Zoho CRM using order data.

# Objective

Automatically sync new WooCommerce orders into Zoho CRM:
- Create/update a **Contact** with customer billing details
- Create a **Deal** with the order value, linked to that Contact

# Tools Used

- **LocalWP** — local WordPress environment
- **WooCommerce** — e-commerce plugin + REST API
- **Zoho CRM** — CRM platform (Contacts & Deals modules)
- **Deluge** — Zoho's scripting language, used to write the integration function
- **Postman** — used to test both APIs manually before writing the final script

## 🔄 How It Works

1. A customer places an order on the local WooCommerce store.
2. A WooCommerce **Webhook** (triggered on `Order updated`) sends the order payload to a Zoho CRM **Deluge Function**, exposed via Zoho's REST API.
3. The Deluge function parses the incoming payload and extracts:
   - Customer name, email, phone, address (from `billing`)
   - Order total
4. The function calls `zoho.crm.createRecord()` twice:
   - Once to create/update the **Contact**
   - Once to create a **Deal**, linked to that Contact, with the order amount

## 📁 Project Structure

```
woocommerce-zoho-crm-integration/
├── README.md
├── script.deluge      # Main integration script (Zoho Deluge Function)
└── screenshots/               # Evidence of successful execution
```


#
## ✅ Manual Testing (via Postman)

Before automating via webhook, both APIs were tested manually and confirmed working:
- `GET /wp-json/wc/v3/orders` — successfully retrieved order data (customer info, line items, totals) from the local WooCommerce store (over HTTPS, via LocalWP's SSL).
- `POST /crm/v8/Contacts` and `POST /crm/v8/Deals` — successfully created a linked Contact and Deal in Zoho CRM using OAuth2 Bearer token authentication.
> Note: The test Contact and Deal created during this manual Postman testing were deleted afterward, as they were created solely to verify the API connections and were not meant to remain as production data.

Screenshots of these successful test calls are included in `/screenshots`.

