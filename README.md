# WooCommerce → Zoho CRM Integration

A local integration between a WooCommerce store (running on WordPress via LocalWP) and Zoho CRM. When a customer places an order, the integration automatically creates a **Contact** and a linked **Deal** in Zoho CRM using order data.

## 🎯 Objective

Automatically sync new WooCommerce orders into Zoho CRM:
- Create/update a **Contact** with customer billing details
- Create a **Deal** with the order value, linked to that Contact

## 🛠️ Tools Used

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
├── deluge_script.deluge      # Main integration script (Zoho Deluge Function)
└── screenshots/               # Evidence of successful execution
```

## 🔑 Setup Notes

Before running this script yourself, you'll need to replace the following placeholders in `deluge_script.deluge`:

| Placeholder | Description |
|---|---|
| `YOUR_WOOCOMMERCE_CONSUMER_KEY` | WooCommerce REST API Consumer Key (WooCommerce → Settings → Advanced → REST API) |
| `YOUR_WOOCOMMERCE_CONSUMER_SECRET` | WooCommerce REST API Consumer Secret |
| `YOUR_STORE_URL` | Your local WooCommerce store URL (e.g. `https://yourstore.local`) |

You'll also need:
1. A Zoho CRM account with a **Self Client** created in the [Zoho API Console](https://api-console.zoho.com/) to obtain OAuth2 credentials.
2. A **Standalone Deluge Function** in Zoho CRM (Setup → Developer Hub → Functions) with **REST API (API Key)** enabled, to receive the webhook payload.
3. A **WooCommerce Webhook** (WooCommerce → Settings → Advanced → Webhooks) pointing to the Zoho Function's REST API URL, with topic set to **Order updated**.

## ⚠️ Challenge Encountered & How It Was Solved

The task specifies *"No hosting is needed — everything runs locally"*, but Deluge functions execute on Zoho's cloud servers, which cannot directly reach a `.local` address running only on a developer's machine. This is an inherent conflict between the two requirements once Deluge (cloud-executed) is used as instructed.

**Resolution approach:**
- Instead of having the cloud-side script *pull* data from the local store (impossible without exposing the local machine, e.g. via ngrok), the integration was inverted: the **local WooCommerce store pushes** order data outward to Zoho via a Webhook — an outbound request, which requires no hosting or open ports on the local machine.
- On the Zoho side, a **Standalone Function with REST API (API Key) access** was used as the webhook receiver. Initial attempts to bind incoming payload fields directly to typed function arguments (`map`, `list`) failed silently (fields returned `null`), because this Zoho function type does not automatically bind arbitrary external JSON payloads to argument names.
- The fix was to accept the raw webhook payload as a single **string** argument and parse it manually inside the function body (`.get("body")` then `.get("billing")...`), which correctly exposed the nested order data.

This was verified end-to-end: WooCommerce successfully triggers the webhook on order status changes, and the Deluge function successfully creates a Contact and a linked Deal in Zoho CRM from the real order payload.

## ✅ Manual Testing (via Postman)

Before automating via webhook, both APIs were tested manually and confirmed working:
- `GET /wp-json/wc/v3/orders` — successfully retrieved order data (customer info, line items, totals) from the local WooCommerce store (over HTTPS, via LocalWP's SSL).
- `POST /crm/v8/Contacts` and `POST /crm/v8/Deals` — successfully created a linked Contact and Deal in Zoho CRM using OAuth2 Bearer token authentication.

Screenshots of these successful test calls are included in `/screenshots`.
