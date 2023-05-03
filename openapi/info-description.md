# Introduction

Use these APIs to programmatically interact with ShopAgain. Under the hood, ShopAgain has a `company` corresponding to your ecommerce store (Shopify, WooCommerce or Omnibus). Each company has two api keys - `public_api_key` that is used within your javascript code and a `api_key` that is used for accessing these API described here. Your `api_key` is private and should never be shared with anyone.

Please contact support to access these keys.

API Endpoint: https://api.shopagain.io/
Webhook Endpoint https://webhooks.shopagain.io/

# Authentication

To use ShopAgain obtain `api_key` for your company and then add to your request in headers as `api-key: <your api key here>`.

Here is an example api call to create a product collection

```
curl -XPOST https://api.shopagain.io/v1/product_collections -H "api-key: <api_key>" -H "content-type:application/json" -d '{"external_collection_id":"123","external_product_ids":["1234"],"title":"Shoes","description":"A great collection of shoes","handle":"shoes","collection_type":"custom","image_url":"https://somecdn.com/image/image1.jpg","updated_at":"2022-06-27T08:48:27-04:00","published_at":"2022-06-27T08:48:27-04:00"}'
```

We only accept `content-type: application/json` for all the requests and return json responses unless we encounter some error.

# Status Codes and Error handling

We use relevant HTTP codes for processing your request. HTTP `200` and `201` are success status code and anything else is a failure. Usually you will receive a `201 - Created` on HTTP Post but we can queue the request and send `200 - OK` and `{"success": true, "message": "queued"}` as message when we are experiencing high loads. You should retry any failed requests

If you receive a HTTP `429 - Too many requests` you should reduce the number of api calls that you are making.

# Guideline for syncing data with ShopAgain

Use the relevant HTTP endpoints to make post requests to sync data for customer, orders or product, etc. Currently HTTP post requests dont generate campaign events, so we recommend using them for only initial data sync. We are evaluating the use of the APIs and this behavior may be changed later.

Once your data is synced on install, we recommend using webhooks with proper event types to send data. 

HTTP calls are rate-limited to 10 requests/minute.

# Webhooks

Webhooks allows us to queue your data and process them efficiently. There are no rate limits and our apis are generally more resilient to downtime. Webhooks allows you to send various different updates on state changes. For example, an `order` can go through several update like `create`, `fulfillled`, `updated`, `cancelled`, you can send all these updates with correct event names so that we can store them and trigger relevant `automation campaigns`.

We do not rate limit webhook calls.

Valid event_types:

`carts/create`, `carts/update`, `checkouts/create`, `checkouts/update`, `checkouts/delete`,
 `contacts/create`, `contacts/update`, `contacts/delete`, `contacts/enable`, `contacts/disable`,
 `orders/cancelled`, `orders/create`, `orders/edited`, `orders/fulfilled`, `orders/paid`, `orders/partially_fulfilled`, `orders/updated`, `orders/delete`, `products/create`, `products/update`, `products/delete`,  `refunds/create`, `collections/create`, `collections/delete`, `collections/update`

`store_domain` must match the configured detail in company.

You need to send a metadata alongwith every request:

```
{
  "metadata": {
    "version": "1",
    "webhook_topic": "products/create",
    "store_domain": "myawesomestore.com",
    "event_id": "123"
  },
  "payload": {
    "name": "test1",
    "external_product_id": "1205",
    "price": 10.0,
    "currency": "USD",
    "created_at": "2023-04-29T21:44:34+00:00",
    "updated_at": "2023-04-29T21:45:34+00:00",
    "variants": [
      {
        "price": 5.0
      }
    ]
  }
}
```