# Docs for [LemonInk](https://lemonink.co) public ebook watermarking API

Below, you'll find the documentation of LemonInk's public ebook watermarking API.

1. [Prerequisites](#1-prerequisites)
2. [Authentication & Headers](#2-authentication--headers)
3. [Uploading files](#3-uploading-files)
4. [Watermarking orders (recommended)](#4-watermarking-orders-recommended)
5. [Watermarking files (advanced)](#5-watermarking-files-advanced)
6. [Delivering watermarked files](#6-delivering-watermarked-files)

## 1. Prerequisites

To be able to perform actions described in the next sections you need to have an active account at [lemonink.co](https://lemonink.co).

You should also know IDs assigned to your master files by LemonInk — you can check those in your master file repository after logging in to LemonInk.

For the [Watermarking orders](#4-watermarking-orders-recommended) flow (which is the recommended way to integrate) you'll additionally need a *store* connected to your account and your products linked to your master files. Connect the store in the LemonInk webapp; product mappings can then be managed in the webapp or through the API.

## 2. Authentication & Headers

Before you try sending any requests to the API, generate an API key. You can do that in your [account settings](https://lemonink.co/configuration/api_keys) after logging in to LemonInk. In the following examples we'll use a fake key: `change-to-your-api-key`.

To authenticate any requests made to our API, you must include an authorization header. Also remember about a correct JSON Content-Type header. So each of your requests must have the following headers:

*Note:* to test how API works, generate a test API key and use it when making calls. This will behave as usual, but won't actualy make any changes and won't cost you a dime.

````
Authorization: Token token=change-to-your-api-key
Content-Type: application/json
````

## 3. Uploading files

You have to first upload some files before watermarking them. It's easy enough to do from our webapp, but you can integrate this feature with your app as well using this method.

### First: create a "master" file.

````
POST https://api.lemonink.co/v1/masters
Authorization: Token token=change-to-your-api-key
Content-Type: application/json

{
  "master": {
    "name": "My Beautiful Book"
  }
}
````

wich will return a response like this:

````
{
  "master": {
    "id": "id-of-the-new-master",
    "name": "My Beautiful Book"
  }
}
````

*Note:* The ID you'll receive in this response is what you'll later need for making watermarking requests.

### Next: attach some files

For each of the formats of your book you need to attach one file to the master. In this case you'll be sending a request in `form-data` format:

````
POST https://api.lemonink.co/v1/master_files
Authorization: Token token=change-to-your-api-key
Content-Type: multipart/form-data; boundary={boundary string}

--{boundary string}--
Content-Disposition: form-data; name="master_file[master_id]",

id-of-the-new-master
--{boundary string}
Content-Disposition: form-data; name="master_file[file]"; filename="My_Beautiful_Book.epub"
Content-Type: application/epub+zip,

--{boundary string}--
Content-Disposition: form-data; name="master_file[name]",

My_Beautiful_Book.epub
--{boundary string}--
````

wich will return a response like this (still in `json` format):

````
{
  "master_file": {
    "id": "id-of-the-new-master-file",
    "master_id": "id-of-the-new-master",
    "name": "My_Beautiful_Book.epub"
  }
}
````

### Last but not least: updating masters and files

To modify those records, issue `PUT` requests to a resource specific URLs like `https://api.lemonink.co/v1/masters/ae929117-4059-4b0f-a14e-f2703ab9dfda`. Contents of the requests should be the same as when creating those resource but you can omit the attributes that you're not changing at the moment.

To remove masters or files issue `DELETE` requests to resource specific URLs (like above).

## 4. Watermarking orders (recommended)

This is the recommended way to integrate LemonInk with your shop. Instead of creating one watermarking transaction per file, you tell us about an order placed in your shop and we'll figure out which masters need to be watermarked, run the watermarking and (optionally) email the download link straight to your customer.

### Before your first order: set up a store and link products

Orders always belong to a *store*. A store represents a single shop you're operating and lets you link products to master files so that we can figure out which masters to watermark. Stores are configured in the LemonInk webapp.

1. *Add store*. Log in to LemonInk and go to [Stores](https://lemonink.co/stores). Click "Connect your store" and pick the "Custom" integration. Give it a name and save — you'll need the store ID from the URL (or the "Copy ID" button on the store list) when sending orders.
2. *Link products*. Go to [Your files](https://lemonink.co/masters) and link each file with associated product (or products) in your store. You can also [manage product mappings through the API](#managing-product-mappings-through-the-api). A single product can be linked to multiple masters (useful for bundles), and a single master can be linked to multiple products.
3. *Setup email delivery*. Optionally, configure email delivery on the store page — when enabled, LemonInk will email the download link directly to your customer once the order is processed.

*Note:* The "Custom" integration is for when you're sending orders from your own backend. We also offer dedicated Shopify and Shoper integrations that don't require any of this — if you're on one of those platforms, just install the LemonInk app from their respective app stores.

### Finding your stores through the API

Use the Stores API instead of copying IDs from the webapp when configuring an integration script:

````
GET https://api.lemonink.co/v1/stores
Authorization: Token token=change-to-your-api-key
````

The response contains every store belonging to your account. Store responses expose only the store's `id` and `name`:

````
{
  "stores": [
    {
      "id": "id-of-your-store",
      "name": "My Store"
    }
  ]
}
````

Fetch one store with `GET /v1/stores/id-of-your-store`. The response wraps the same fields in a `store` object. Both requests return `200 OK` when successful.

### Managing product mappings through the API

A product mapping links one product in a store to one master. Product IDs are non-empty, opaque strings supplied by your store; LemonInk does not look them up in or validate them against the store's catalogue when a mapping is created.

Create a mapping with:

````
POST https://api.lemonink.co/v1/product_mappings
Authorization: Token token=change-to-your-api-key
Content-Type: application/json

{
  "product_mapping": {
    "store_id": "id-of-your-store",
    "master_id": "id-of-your-master",
    "product_id": "id-of-product-in-your-store"
  }
}
````

A successful request returns `201 Created`:

````
{
  "product_mapping": {
    "id": "id-of-the-product-mapping",
    "store_id": "id-of-your-store",
    "master_id": "id-of-your-master",
    "product_id": "id-of-product-in-your-store"
  }
}
````

The combination of `store_id`, `master_id` and `product_id` must be unique. Creating the same mapping twice, or using a blank `product_id`, returns `422 Unprocessable Content`. A product may still be linked to several masters, which is how bundles are represented, and a master may be linked to several products.

List every mapping belonging to your account with `GET /v1/product_mappings`. The list is not paginated. You can narrow it with any combination of these exact-match filters:

````
GET https://api.lemonink.co/v1/product_mappings?filter[store_id]=id-of-your-store
GET https://api.lemonink.co/v1/product_mappings?filter[master_id]=id-of-your-master
GET https://api.lemonink.co/v1/product_mappings?filter[product_id]=id-of-product-in-your-store
````

When more than one filter is supplied, all of them must match. The response is a `product_mappings` array containing only the same four fields shown above. Fetch one mapping with `GET /v1/product_mappings/id-of-the-product-mapping`; its response wraps those fields in a `product_mapping` object. Both requests return `200 OK` when successful.

Remove a mapping with:

````
DELETE https://api.lemonink.co/v1/product_mappings/id-of-the-product-mapping
Authorization: Token token=change-to-your-api-key
````

A successful deletion returns `204 No Content` with an empty response body. Product mappings cannot be updated; delete the old mapping and create a new one when its store, master or product ID changes.

Fetching or deleting an unknown resource ID returns `404 Not Found`, as does creating a mapping with an unknown store or master ID. An index filter for another account's store or master simply returns an empty `product_mappings` array.

### Sending an order

Once your store is set up, send your orders to the `/orders` endpoint:

````
POST https://api.lemonink.co/v1/orders
Authorization: Token token=change-to-your-api-key
Content-Type: application/json

{
  "order": {
    "store_id": "id-of-your-store",
    "external_id": "order-id-in-your-system",
    "number": "ORDER-1234",
    "customer": {
      "email": "christine.smith@example.com",
      "first_name": "Christine",
      "last_name": "Smith"
    },
    "line_items": [
      { "product_id": "id-of-product-in-your-shop" },
      { "product_id": "id-of-another-product-in-your-shop" }
    ]
  }
}
````

And for such a request, you'll receive a response like this:

````
{
  "order": {
    "id": "id-of-the-new-order",
    "external_id": "order-id-in-your-system",
    "status": "received",
    "download_url": "https://dl.lemonink.co/orders/api/a-token-unique-for-this-order/id-of-the-new-order",
    "processed_at": null,
    "deliver_via_email": null
  }
}
````

A few things to know about the request:

- `store_id` is required and must point to an active store you own.
- `external_id` is required and must be unique within the store — it's the ID of the order in your system. Sending the same `external_id` twice will return a `422` error, which makes the endpoint safe to retry.
- `number` is only used for display purposes (in your customer's download page and in the watermark, if your template uses `order_number`).
- `customer` describes the buyer. Supported keys are `id`, `email`, `name`, `first_name`, `last_name` and `company_name`. Any other keys are silently dropped. Provide whichever keys your watermarking template needs — at minimum, `email` is recommended (and required when email delivery is enabled, see below). `id` is your store's ID for the customer. It is only used to group [LCP-protected](#protecting-orders-with-readium-lcp-drm) books under one passphrase.
- `line_items` is an array of items in the order. Each item has a single supported key — `product_id` — which must match a product ID you've linked in your store. Items whose product IDs aren't linked to any master are silently ignored. If none of the items match a linked product, the order ends up in `skipped` status.
- `deliver_via_email` is optional. When omitted, the store's default setting is used. Set it to `true` or `false` to override on a per-order basis. When email delivery is on, `customer.email` must be present.

### Checking the status of an order

Order processing is asynchronous — when LemonInk receives your request, the order starts in `received` status and moves through `processing` to one of the terminal statuses. You can poll the order to see where it is:

````
GET https://api.lemonink.co/v1/orders/id-of-the-new-order
Authorization: Token token=change-to-your-api-key
````

The response has the same shape as the create response. The `status` field is one of:

- `received` — we got the request and it's queued for processing.
- `processing` — we're watermarking files for this order.
- `completed` — at least one file was successfully watermarked and the download link is ready.
- `skipped` — none of the line items matched a linked product, so there was nothing to watermark.
- `failed` — something went wrong. We monitor failures on our side, but feel free to contact us if a specific order keeps failing.

`processed_at` will be set once the order leaves `received`/`processing`.

### Protecting orders with Readium LCP DRM

By default every book in an order is watermarked. If [Readium LCP](https://lemonink.co/blog/2026-08-31-drm-it-is-readium-lcp-support-is-now-live-in-lemonink) is enabled for your account, you can protect books with LCP DRM instead. Choose the protection method in the LemonInk webapp for your whole account, a store, a single file or for specific product in your store.

## 5. Watermarking files (advanced)

If the [Watermarking orders](#4-watermarking-orders-recommended) flow doesn't fit your case — for example, when prefer to link files to products on your end — you can use the lower-level Transactions API directly.

To watermark a file, send a POST request to the `/transactions` endpoint, providing details in a JSON body. For example:

````
POST https://api.lemonink.co/v1/transactions
Authorization: Token token=change-to-your-api-key
Content-Type: application/json

{
  "transaction": {
    "master_id": "id-of-your-master-file",
    "watermark_value": "A text that will be inserted into the file"
  }
}
````

And for such a request, you'll receive a response like this:

````
{
  "transaction": {
    "id": "id-of-the-new-transaction",
    "watermark_value": "A text that will be inserted into the file",
    "token": "a-token-unique-for-this-transaction",
    "master_id": "id-of-your-master-file",
    "user_id": "your-account-id"
  }
}
````

You can also replace `watermark_value` with `watermark_params` which will be interpolated into the template you've configured for your account:

````
POST https://api.lemonink.co/v1/transactions
Authorization: Token token=change-to-your-api-key
Content-Type: application/json

{
  "transaction": {
    "master_id": "id-of-your-master-file",
    "watermark_params": {
      "customer_email": "christine.smith@example.com",
      "order_number": "44321"
    }
  }
}
````

Look into your watermarking configuration for the list of the params you'll need — make sure to include all that are used in your template.

### Protecting files with Readium LCP DRM

If Readium LCP is enabled for your account, a transaction can protect a file with LCP DRM instead of a watermark. Set `protection_method` to `lcp` and, optionally, pass `lcp_params`:

````
POST https://api.lemonink.co/v1/transactions
Authorization: Token token=change-to-your-api-key
Content-Type: application/json

{
  "transaction": {
    "master_id": "id-of-your-master-file",
    "protection_method": "lcp",
    "lcp_params": {
      "customer_ref": "customer-42",
      "license_ends_at": "2027-12-31T23:59:59Z"
    }
  }
}
````

- `protection_method` is either `watermark` or `lcp`. When you leave it out, the transaction uses the protection method configured in the webapp.
- `lcp_params.customer_ref` is optional. It's an identifier of your choice for the buyer. All LCP transactions with the same `customer_ref` share one passphrase, so a customer can open every book they bought with it. Without it, each transaction gets its own passphrase.
- `lcp_params.license_ends_at` is optional. It's an ISO 8601 date and time when the license expires. When omitted, the master's availability period applies, if one is set. Otherwise the license never expires.
- `watermark_value` and `watermark_params` aren't required for LCP transactions.

LCP can only protect EPUB and PDF files. If the master has any other format (for example MOBI or an MP3 audiobook), the request is rejected with `422 Unprocessable Content` and an error with the code `validation_lcp_formats_not_supported`. This also happens when LCP comes from your webapp settings and not from the request.

The response of an LCP transaction lists its formats as `lcpl` (an LCP-protected EPUB) and `lcpdf` (an LCP-protected PDF). The customer downloads an LCP license file and needs the passphrase to open it in an LCP-compatible reading app. The passphrase is shown on the transaction's download page, available in the response's `download_url` field.

## 6. Delivering watermarked files

How you deliver the watermarked files to your customer depends on which flow you used to create them.

### From the orders flow

Each successfully completed order has a `download_url` you can hand to your customer:

````
https://dl.lemonink.co/orders/api/a-token-unique-for-this-order/id-of-the-new-order
````

This is a download page where the customer can pick the format they want. The token in the URL doubles as an authorization token, so anyone with the link can download.

If you've enabled email delivery on the store, you can skip this step entirely — LemonInk will email the link to `customer.email` once the order is `completed`.

### From the transactions flow

To deliver a transaction's protected file to your customer, generate a link to the transaction. You can either download the file to your server and then serve it by yourself or you can give your user a direct link to our servers. Regardless of how you wish to deliver the files, here is how the delivery links are constructed:

````
https://dl.lemonink.co/transactions/a-token-unique-for-this-transaction/id-of-the-new-transaction.format
````
Where format is one of `epub`, `mobi` or `pdf`. For [LCP-protected transactions](#protecting-files-with-readium-lcp-drm), use `lcpl` or `lcpdf` instead. Both download an `.lcpl` license file. Your customer will also need the passphrase, which is shown on the transaction's `download_url` page and isn't included in the API response.

Et voilà!

## Contact us!

If you find the above docs lacking, or need any more info, don't hestitate to drop us a line at [hello@lemonink.co](mailto:hello@lemonink.co).
