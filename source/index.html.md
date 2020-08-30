---
title: Camaloon Print on Demand API (v1)

language_tabs: # must be one of https://git.io/vQNgJ
  - shell: code

toc_footers:
  - <a href='https://camaloon.com/print_on_demand'>Camaloon Print on Demand</a>

search: false

code_clipboard: true
---

# Introduction

Welcome to Camaloon Print On Demand API (version 1.0)!

This API allows Print On Demand clients with ad hoc ecommerce platforms to connect to Camaloon Print On Demand platform.

The API is asynchronous and event based (via webhooks).

# Webhooks to Camaloon

In this section we are going to describe webhooks originated by events on the origin ecommerce platform.

## Configuration

### Webhook endpoint

Webhooks to Camaloon must be sent as a **POST** request to the following endpoint:  

<aside class="success">
<span style="font-weight: bold; font-size: 1.2em; font-family: monospace">https://camaloon.com/print_on_demand/generic/webhooks/receive</span>
</aside>

### Webhook headers

Webhooks sent to Camaloon must contain the following headers:

**`X-CAMALOON-SOURCE-DOMAIN`**: Your store domain, the same used when creating the store in Camaloon.  
**`X-CAMALOON-TOPIC`**: Event name (ex: `order_paid`).  
**`X-CAMALOON-HMAC-SHA256`**: Base64 digest of the request body payload using the API secret key provided by Camaloon.  

## Order paid event

When an order is **paid** in your ecommerce platform, a POST request should be sent to the [Camaloon webhook endpoint](#endpoint) with the `X-CAMALOON-TOPIC=order_paid` header.

> Request payload example

```json
{
  "id": 1,
  "ref": "20200901003",
  "currency": "USD",
  "tax_rate": 0.12,
  "created_at": "2020-09-01T16:09:54-04:00",
  "processed_at":"2020-09-01T16:09:55-04:00",
  "shipping_price": 3.2,
  "shipping_info": {
    "first_name": "Raymond B",
    "last_name": "Gordon",
    "company_name": "Gordon & Sons",
    "telephone": "509-259-7576",
    "address": "1317 Dane Street",
    "address2": "",
    "zip": "99032",
    "city": "Sprague",
    "province": "Washington",
    "country_code": "US"
  },
  "billing_info": {
    "first_name": "Raymond B",
    "last_name": "Gordon",
    "company_name": "Gordon & Sons",
    "telephone": "509-259-7576",
    "address": "1317 Dane Street",
    "address2": "",
    "zip": "99032",
    "city": "Sprague",
    "province": "Washington",
    "country_code": "US"
  },
  "line_items": [
    {
      "sku": "063e8da93379f8c1aee0",
      "description": "Magic mug",
      "quantity": 10,
      "price": 82.84
    }
  ]
}
```

See on the right an example of request payload of a paid order.

### Order properties

Parameter | Optional | Description
--------- | ---- | -----------
`id` | no | Order id in your system.
`ref` | yes | Human name of the order.
`currency` | no | Currency [ISO 4217 code](https://en.wikipedia.org/wiki/ISO_4217) of the order.
`tax_rate` | no | The tax rate applied to the order to calculate the tax price.
`created_at` | no | The [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) date and time when an order has been created.
`processed_at` | no | The [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) date and time when an order has been processed (paid).
`shipping_price` | no | Total shipping price, without tax, in the order's currency.
`shipping_info` | no | The mailing address to where the order will be shipped. See [address properties](#address-properties).
`billing_info` | yes | The billing address of the customer. If not provided, shipping info will be used. See [address properties](#address-properties).
`line_items` | no | Items purchased in the order. See [line item properties](#line-item-properties).

### Address properties

Parameter | Optional | Description
--------- | ---- | -----------
`first_name` | no | The first name of the person associated with the address.
`last_name` | no | The last name of the person associated with the address.
`company_name` | yes | The company of the person associated with the address.
`telephone` | yes | The phone number at the address or client's phone. Max length: 15
`address` | no | The street address.
`address2` | yes | An optional additional field for the street address.
`zip` | no | Valid postal code of the address.
`city` | no | The city, town, or village of the address.
`province` | no | The name of the region (province, state, prefecture, …) of the address.
`country_code` | no | The two-letter [ISO 3166-2](https://en.wikipedia.org/wiki/ISO_3166-2) code for the country of the address.


### Line item properties

Parameter | Optional | Description
--------- | ---- | -----------
`sku` | no | Stock Keeping Unit. If matches one of the sku's on your Camaloon store, the item will be processed by Camaloon.
`description` | yes | Name of the product or description of the line item.
`quantity` | no | The number of items that were purchased.
`price` | no | The total final price of the line item, without taxes.

### Error handling

Webhooks are processed asynchronously, but the data received is validated according to the previous described schema.
If the data does not follow the schema, a HTTP status code `400` is returned and along a `JSON` with a list of errors.

# Webhooks from Camaloon

In this section we are going to describe webhooks originated by events on the Camaloon's side.

## Configuration

### Webhook endpoint

Endpoints for the different events originated in Camaloon can be configured in the store backoffice.

You will create a webhook, selecting the event name and define the endpoint that will process the webhook on your side.

### Webhook headers

Webhooks originated in Camaloon include the following headers.

**`X-CAMALOON-TOPIC`**: Event name (ex: `shipment_sent`).  
**`X-CAMALOON-HMAC-SHA256`**: Base64 digest of the request body payload using the API secret key provided by Camaloon.  


## Shipment sent event

When an order has been fulfilled by Camaloon and a shipment has been collected by the carrier service, a webhook will be sent with the tracking shipment info.  

<aside class="notice">
NOTE: one or multiple shipments can be sent from Camaloon for the same order. This means that a shipment can contain all or part of the line items produced by Camaloon.
</aside>

> Request payload example

```json
{
  "order_id": 1,
  "tracking_number": "9414 1058 1746 0398 1325 11",
  "tracking_url": "https://tools.usps.com/go/TrackConfirmAction?qtc_tLabels1=9414105817460398132511",
  "notify_customer": true,
  "line_items": [
    {
      "sku": "063e8da93379f8c1aee0",
      "quantity": 10
    }
  ]
}
```

See on the right an example of request payload of a shipment sent.

### Error handling

Camaloon will listen to the HTTP response code of the webhook request, expecting a `200` or `204` code.

If the code returned is `410`, the webhook will be removed permanently.

Any other response code will make the webhook retry 3 times at most. If a webhook fails 20 times consecutively, it will be temporarily disabled.