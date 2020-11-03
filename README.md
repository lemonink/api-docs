# Docs for [LemonInk](https://lemonink.co) public ebook watermarking API

Below, you'll find the documentation of LemonInk's public ebook watermarking API.

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

To modify those record issue `PUT` requests to a resource specific URLs like `https://api.lemonink.co/v1/masters/ae929117-4059-4b0f-a14e-f2703ab9dfda`. Contents of the requests should be the same as when creating those resource but you can omit the attributes that you're not changing at the moment.

To remove masters or files issue `DELETE` requests to resource specific URLs (like above).


## 4. Watermarking

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
    "token": "a-token-unique-for-this-transaction",
    "master_id": "id-of-your-master-file",
    "user_id": "your-account-id"
  }
}
````

Now for the last part:

## 5. Delivering watermarked files

In order to deliver the protected file to your customer, you need to generate a link to the transaction. You can either download the file to your server and then serve it by yourself or you can give your user a direct link to our servers. Regardless of how you wish to deliver the files, here is how the delivery links are constructed:

````
https://dl.lemonink.co/transactions/a-token-unique-for-this-transaction/id-of-the-new-transaction.format
````
Where format is one of `epub`, `mobi` or `pdf`.

Et voilà!

## Contact us!

If you find the above docs lacking, or need any more info, don't hestitate to drop us a line at [hello@lemonink.co](mailto:hello@lemonink.co).
