# Docs for [LemonInk](https://lemonink.co) public API

Below, you'll find the documentation of LemonInk's public watermarking API.

1. [Prerequisites](#1-prerequisites)
2. [Authentication & Headers](#2-authentication--headers)
3. [Watermarking](#3-watermarking)
4. [Delivering watermarked files](#4-delivering-watermarked-files)
 
## 1. Prerequisites

To be able to perform actions described in the next sections you need to have an active account at [lemonink.co](https://lemonink.co).

And lastly, you should know IDs assigned to your master files by LemonInk - you can check those in your master file repository after logging in to LemonInk. 

## 2. Authentication & Headers

Before you try sending any requests to the API, generate an API key. You can do that in your [account settings](https://lemonink.co/account/api-keys) after logging in to LemonInk. In the following examples we'll use a fake key: `change-to-your-api-key`.

To authenticate any requests made to our API, you must include an authorization header. Also remember about a correct JSON Content-Type header. So each of your requests must have the following headers:

*Note:* to test how API works, generate a test API token and use it when making calls. This will behave as usual, but won't actualy make any changes and won't cost you a dime.

````
Authorization: Token token=change-to-your-api-key
Content-Type: application/json
````

## 3. Watermarking

Watermarking using LemonInk's API is simply done by creating a `Transaction` resource. To do so, you need to send a POST request to the `/transactions` endpoint, providing details in a JSON body. For example:

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
    "watermark_value: "A text that will be inserted into the file",
    "token": "a-very-secret-token-unique-for-this-transaction",
    "master_id": "id-of-your-master-file",
    "user_id": "your-account-id"
  }
}
````

Now for the last part:

## 4. Delivering watermarked files

In order to deliver the protected file to your cusotmer, you need to generate a link to the transaction. You can either download the file to your server and then serve it by yourself or you can give your user a direct link to our servers. Regardless of how you wish to deliver the files, here is how the delivery links are constructed:

````
https://dl.lemonink.co/transactions/a-very-secret-token-unique-for-this-transaction/id-of-the-new-transaction
````

Et voilà!

## Contact us!

If you find the above docs lacking, or need any more info, don't hestitate to drop us a line at [hello@lemonink.co](mailto:hello@lemonink.co).
