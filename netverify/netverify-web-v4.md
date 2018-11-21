![Jumio](/images/netverify.png)

# Netverify Web v4 Implementation Guide

This is a reference manual and configuration guide for the new Netverify Web client. It describes how to initiate a transaction, how to customize your settings and branding, and how to display Netverify to your users.
<br>
### Revision history

Information about changes to features and improvements documented in each release is available in our [Revision history](/netverify/README.md).
<br>

## Table of contents

- [Initiating a Netverify transaction](#initiating-a-netverify-transaction)
	- [Authentication and encryption](#authentication-and-encryption)
	- [Request headers](#request-headers)
	- [Request body](#request-body)
		- [Supported workflowId values](#supported-workflowid-values)
		- [Supported presets values](#supported-presets-values)
		- [Supported locale values](#supported-locale-values)
	- [Response](#response)
	- [Examples](#examples)
		- [Sample request](#sample-request)
		- [Sample response](#sample-response)
- [Configuring settings in the Customer Portal](#configuring-settings-in-the-customer-portal)
	- [Application settings - General](#application-settings--general)
		- [Callback, Error, and Success URLs](#callback-error-and-success-urls)
		- [Capture method](#capture-method)
		- [Skip "Start ID verification" screen](#skip-start-id-verification-screen)
		- [Authorization token lifetime](#authorization-token-lifetime)
	- [Application settings - Redirect](#application-settings--redirect)
		- [Domain name prefix](#domain-name-prefix)
		- [Default locale](#default-locale)
	- [Customize client](#customize-client)
		- [Colors](#colors)
		- [Images](#images)
- [Displaying Netverify](#displaying-netverify)
	- [Using Netverify in an iFrame](#using-netverify-in-an-iframe)
		- [Width and height](#width-and-height)
		- [Example HTML](#example-html)
		- [Optional iFrame Logging](#optional-iframe-logging)
			- [Example iFrame logging code](#example-iframe-logging-code)
- [After the user journey](#after-the-user-journey)
	- [Sample success redirect](#sample-success-redirect)
	- [Sample error redirect](#sample-error-redirect)
- [Supported browsers](#supported-browsers) 		



<br>

---
# Initiating a Netverify transaction

Call the RESTful API POST endpoint **/initiate** with a JSON object containing the properties described below to create a transaction for each user. You will receive a JSON object in the response containing a timestamp, Jumio transaction reference (scan reference), and a URL which you can use to present Netverify to your user.

**HTTP Request Method:** `POST`<br>
**REST URL (US)**: `https://netverify.com/api/v4/initiate`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/v4/initiate`<br>

<br>

## Authentication and encryption
Netverify API calls are protected using [HTTP Basic Authentication](https://tools.ietf.org/html/rfc7617). Your Basic Auth credentials are constructed using your API token as the user-id and your API secret as the password. You can view and manage your API token and secret in the Customer Portal under **Settings > API credentials**.
<br>

|⚠️ Never share your API token, API secret, or Basic Auth credentials with *anyone* — not even Jumio Support.
|:----------|

The [TLS Protocol](https://tools.ietf.org/html/rfc5246) is required to securely transmit your data, and we strongly recommend using the latest version. For information on cipher suites supported by Jumio during the TLS handshake see [Supported cipher suites](/netverify/supported-cipher-suites.md).

<br>

## Request headers

The following fields are required in the header section of your request:<br>

`Accept: application/json`<br>
`Content-Type: application/json`<br>
`Content-Length:`  (see [RFC-7230](https://tools.ietf.org/html/rfc7230#section-3.3.2))<br>
`Authorization:` (see [RFC 7617](https://tools.ietf.org/html/rfc7617))<br>
`User-Agent: YourCompany YourApp/v1.0`<br>

|ℹ️ Jumio requires the `User-Agent` value to reflect your business or entity name for API troubleshooting.|
|:---|

<br>

## Request body

The body of your **initiate** API request allows you to

- provide your own internal tracking information for the user and transaction.
- specify what user information is captured and by which method.
- indicate where the user should be directed after the user journey.
- select the language to be displayed.
- preset options to enhance the user journey.

|ℹ️ Values set in your API request will override the corresponding settings configured in the Customer Portal.
|:----------|

<br>

**Required items appear in bold type.**  

|Name               |Type   | Max. length|Description                                                                                                  |
|:---                      |:---    |:---        |:---                                                                                                          |
|**customerInternalReference**<sup>1</sup>|string |100        |Your internal reference for the transaction.                                                               |
|**userReference**<sup>1</sup>           |string |100        |Your internal reference for the user.                                                                   |
|reportingCriteria        |string |100        |Your reporting criteria for the transaction.                                                                      |
|successUrl<sup>2</sup>               |string |2047        |Redirects to this URL after a successful transaction.<br>Overrides [Success URL](#callback-error-and-success-urls) in the Customer Portal.|
|errorUrl<sup>2</sup>                 |string |255        |Redirects to this URL after an unsuccessful transaction.<br>Overrides [Error URL](#callback-error-and-success-urls) in the Customer Portal.|
|callbackUrl<sup>2</sup>              |string |255        |Sends verification result to this URL upon completion.<br>Overrides [Callback URL](#callback-error-and-success-urls) in the Customer Portal.|
|workflowId               |integer |3          |Applies this acquisition workflow to the transaction.<br>Overrides [Capture method](#capture-method) in the Customer Portal.<br>See [supported workflowId values](#supported-workflowid-values).|
|presets<sup>1</sup>                  |JSON   |	  -    |Preset options to enhance the user journey.<br>See [supported preset values](#supported-presets-values).|-             |
|locale                   |string |5          |Renders content in the specified language.<br>Overrides [Default locale](#default-locale) in the Customer Portal.<br>See [supported locale values](#supported-locale-values).|
|tokenLifetimeInMinutes  |number 	|Max. value: 86400 |Time in minutes until the authorization token expires. (minimum: 5, maximum: 86400)<br>Overrides [Authorization token lifetime](#authorization-token-lifetime) in the Customer Portal.  |

<sup>1</sup> Values **must not** contain Personally Identifiable Information (PII) or other sensitive data such as email addresses.<br>
<sup>2</sup> See URL constraints for [Callback, Error, and Success URLs](#callback-error-and-success-urls).

<br>

## Supported `workflowId` values
Acquisition workflows allow you to set a combination of verification and capture method options.

|⚠️ Identity Verification must be enabled for your account to use an ID + Identity `workflowId`.
|:----------|

|Value |Verification type |Capture method |
|:-----|:-----------------|:--------------|
|100   |ID only           |camera + upload|
|101   |ID only           |camera only    |
|102   |ID only           |upload only    |
|200   |ID + Identity     |camera + upload|
|201   |ID + Identity     |camera only    |
|202   |ID + Identity     |upload only    |

<br>

## Supported `presets` values
It is possible to specify presets for **ID Verification**, for **Identity Verification**, for both together, or for neither. For each preset you use, all values must be passed together as a JSON array (see [Sample request](#sample-request)) for the request to be valid.
<br>

### ID Verification: preset country and document type
Preset the country and document type to bypass the selection screen.

**Required items appear in bold type.**

|Name   |Type    |Max. length    |Description    |
|:------------|:-------|:--------------|:--------------|
|**index**|integer|1| must be set to `1`|
|**country**|string|3|Possible values:<br>•	[ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code <br /> • `XKX` (Kosovo) |
|**type**|string|15|Possible values:<br>• `PASSPORT`<br>•	`DRIVING_LICENSE`<br>•	`ID_CARD`|
<br>

### Identity Verification: preset Liveness Detection phrase
Preset a custom Liveness Detection phrase for the transaction.<br>

|⚠️ Identity Verification and Liveness Detection must be enabled for your account to use this preset.
|:----------|
<br>

**Required items appear in bold type.**

|Name   |Type    |Max. length    |Description    |
|:------------|:-------|:--------------|:--------------|
|**index**|integer|1| must be set to `2`|
|**phrase**<sup>1</sup>|string|30|Possible values:<br>• alpha-numeric Latin characters (upper or lower case) and spaces |

<sup>1</sup> Values **must not** contain Personally Identifiable Information (PII) or other sensitive data such as email addresses.

<br>


## Supported `locale` values
Hyphenated combination of [ISO 639-1:2002 alpha-2](https://en.wikipedia.org/wiki/ISO_639-1) language code plus [ISO 3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) country (where applicable).


|Value  |Locale|
|:--------------|:--------------|
|bg|Bulgarian|
|cs|Czech|
|da|Danish|
|de|German|
|el|Greek|
|en|American English (**default**)|
|en-GB|British English|
|es|Spanish|
|es-MX|Mexican Spanish|
|et|Estonian|
|fi|Finnish|
|fr|French|
|hu|Hungarian|
|it|Italian|
|ja|Japanese|
|ko|Korean|
|lt|Lithuanian|
|nl|Dutch|
|no|Norwegian|
|pl|Polish|
|pt|Portuguese|
|pt-BR|Brazilian Portuguese|
|ro|Romanian|
|ru|Russian|
|sk|Slovak|
|sv|Swedish|
|tr|Turkish|
|vl|Vietnamese|
|zh-CN|Simplified Chinese|
|zh-HK|Traditional Chinese|

---
## Response
Unsuccessful requests will return the relevant [HTTP status code](https://tools.ietf.org/html/rfc7231#section-6) and information about the cause of the error.

Successful requests will return HTTP status code `200 OK` along with a JSON object containing the information described below.

**Required items appear in bold type.**

|Name|Type|Max. length|Description|
|:----|:----|:----|:----|
|**timestamp**|String|24|Timestamp (UTC) of the response.<br>Format: *YYYY-MM-DDThh:mm:ss.SSSZ*|
|**redirectUrl**|String|255|URL used to load the Netverify client.|
|**transactionReference**|String|36|Jumio reference number for the transaction.|

---
## Examples
### Sample request

~~~
POST https://netverify.com/api/v4/initiate/ HTTP/1.1
Accept: application/json
Content-Type: application/json
Content-Length: 1234
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx
{
  "customerInternalReference" : "transaction_1234",
  "userReference" : "user_1234",
  "successUrl" : "https://www.yourcompany.com/success",
  "errorUrl" : "https://www.yourcompany.com/error",
  "callbackUrl" : "https://www.yourcompany.com/callback",
  "reportingCriteria" : "myReport1234",
  "workflowId" : 200,
  "presets" :
    [
      {
        "index"   : 1,
        "country" : "AUT",
        "type"    : "PASSPORT"
      },{      
        "index"   : 2,
        "phrase" : "MY CUSTOM PHRASE"     
      }
    ],
  "locale" : "en-GB"
}

~~~

|⚠️ Sample requests cannot be run as-is. Replace example data with your own values.
|:----------|
<br>

### Sample response

~~~
{
  "timestamp": "2018-07-03T08:23:12.494Z",
  "transactionReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "redirectUrl": "https://yourcompany.netverify.com/web/v4/app?locale=en-GB&authorizationToken=xxx"
}
~~~

<br>

---
# Configuring settings in the Customer Portal

In the **Settings** screen of the Customer Portal you can customize your settings and brand your Netverify page. <br>Save changes using your Customer Portal password to activate them.
<br>

|ℹ️ Values set in your API request will override the corresponding settings configured in the Customer Portal.
|:----------|

<br>

## Application settings — General

### Callback, Error, and Success URLs

Define a **Callback URL** to receive verification results and extracted user data from Jumio when a transaction is completed. For more information, see our [Callback](/netverify/callback.md) documentation.

Define a **Success URL** to direct the user after images are accepted for processing. If no **Success URL** is specified in the Customer Portal or the **initiate** API request, the Jumio default success page will be displayed, including any custom [images](#images) you have specified.

Define an **Error URL** to direct the user when the verification process ends with an error or a failure after 3 submission attempts. If no **Error URL** is specified in the Customer Portal or the **initiate** API request, the Jumio default error page will be displayed, including any custom [images](#images) you have specified.



#### URL requirements:

* HTTPS using the [TLS Protocol](https://tools.ietf.org/html/rfc5246) (most recent version recommended)
* Valid [URL](https://tools.ietf.org/html/rfc3986) using [ASCII characters](https://en.wikipedia.org/wiki/ASCII) or [IDNA Punycode](https://tools.ietf.org/html/rfc3492)

#### URL restrictions:

* IP addresses, ports, certain query parameters and fragment identifiers are not allowed.
* Personally identifiable information (PII) is not allowed in any form.

Jumio appends the following parameters to your Success or Error URL to redirect your user at the conclusion of the user journey. These cannot be used as part of your Success or Error URL:

|Name|Description|
|:---|:---|
|transactionStatus| • `SUCCESS` for successful submissions. <br> • `ERROR`for errors and failure after 3 attempts.|
|customerInternalReference|Your internal reference for the transaction.|
|transactionReference|Jumio reference number for the transaction.|
|errorCode|Displayed when `transactionStatus` is `ERROR`.|

<br>

### Capture method

Specify how your user can submit their ID or Identity image for verification.<br>
<br>
Choose from:

- **Webcam and image upload**
- **Webcam only**
- **Image upload only**



|⚠️ Selecting "Webcam only" means some [mobile browsers](#mobile) will not be supported.
|:----------|

<br>

### Skip "Start ID verification" screen

Select this checkbox to bypass the introductory screen in the Netverify Web client.

<br>

### Authorization token lifetime

Specify the duration of time for which your `redirectUrl` will remain valid. Enter the value in minutes (minimum 5, maximum 86400). The default value is 30 minutes.

<br>

## Application settings — Redirect

### Domain name prefix

You can optionally define a domain name prefix (`https://yourcompany.netverify.com`) for the URL of your Netverify page.


- Allowed characters are letters `a-z`, numbers `0-9`, `-`
- Must not start or end with `-`
- Max. 63 characters

<br>

### Default locale

Select a language from the dropdown list to set your display language for Netverify. If no language is selected, Netverify will be displayed in **English (US)**.<br>

Choose from:<br>

- English
- English (United Kingdom)
- German
- Turkish
- Finnish
- Norwegian
- Polish
- Swedish
- Russian
- Portuguese
- Portuguese (Brazil)
- Spanish
- Spanish (Mexico)
- Italian
- French
- Dutch
- Bulgarian
- Chinese (China)
- Chinese (Hong Kong)
- Czech
- Danish
- Greek
- Hungarian
- Japanese
- Korean
- Romanian
- Slovak
- Vietnamese
- Lithuanian
- Estonian

---

<br>

## Customize client

### Colors

Specify primary and secondary colors for each locale to give Netverify your own look and feel.

Any locale which is not configured will first default to the root language (e.g. EN\_GB to EN), then to your default configuration, and finally to the Jumio default.

You can also reset all colors to the Jumio default.


### Images

Add a **Header image** for each locale to brand your Netverify page.

Add a **Success image** and **Error image** for each locale to be displayed on the Jumio default success and error pages when you do not specify your own [Success URL](#callback-error-and-success-urls) and [Error URL](#callback-error-and-success-urls).

Any locale which is not configured will first default to the root language (e.g. EN\_GB to EN), then to your default configuration, and finally to the Jumio default.

All images must be formatted as [JPG](https://jpeg.org/jpeg/) or [PNG](https://en.wikipedia.org/wiki/Portable_Network_Graphics) and must not exceed 5 MB.

---

<br>

# Displaying Netverify

The **redirectUrl** returned in the response to your **initate** API call, which loads your customized Netverify page, can be used in several ways:

* within an iFrame on your web page
* as a link on your web page
* as a link shared securely with a user

## Using Netverify in an iFrame
If you want to embed Netverify on a web page, place the iFrame tag in your HTML code where you want the client to appear. Use the `redirectUrl` as value of the `src` attribute. The `allow="camera"` attribute must be included to enable the camera for image capture in [supported browsers](#supported-browsers).

### Width and height
We recommend adhering to the responsive breaking points in the table below. The Netverify Web client will responsively fill the dimensions of your iFrame.

|Size class |Width|Height|
|:-------|---:|-------:|
|Large|≥ 900 px|≥ 710 px|
|Medium| 640 px|660 px|
|Small|560 px|600 px|
|X-Small|≤ 480 px|≤ 535 px|

### Example HTML  
```
<iframe src="https://yourcompany.netverify.com/web/v4/app?locale=en-GB&authorizationToken=xxx" width="930" height="750" allow="camera"></iframe>
```



<!---
### Using the Customer Portal - do not display

You can create a transaction manually in the Customer Portal under the "Create verfication" tab. The values you can specify are equal to the related API request parameters described in the previous section of the guide.
--->
<br>



### Optional iFrame logging

When the Netverify client is embedded in an iFrame<sup>1</sup>, it will communicate with the containing page using the JavaScript [`window.postMessage()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) method to send events containing pre-defined data. This allows the containing page to react to events as they occur (e.g., by directing to a new page once the `success` event is received). Events include data that allows the containing page to identify which Netverify transaction triggered the event. Events are generated in a stateless way, so that each event contains general contextual information about the transaction (e.g., transaction reference, authorization token, etc.) in addition to data about the specific event that occurred.

Using JavaScript, the containing page can receive the notification and consume the data it contains by listening for the `message` event on the global `window` object and reacting to it as needed. The data passed by the Netverify Web client in this notification is represented as JSON in the `data` string property of the listener method's `event` argument. Parsing this JSON string results in an object with the properties described below.

All data is encoded with [UTF-8](https://tools.ietf.org/html/rfc3629).
<br>
<br>
<sup>1</sup> This functionality is not available for instances of Netverify running in a standalone window or tab.<br>

### `event.data` object

**Required items appear in bold type.**  

|Property|Type|Description
|:-------|:---|:----------|
|**authorizationToken**|string|Authorization token, valid for a specified duration.|
|**transactionReference**|string|Jumio reference number for the transaction.|
|**customerInternalReference**|string|Your internal reference for the transaction.|
|**eventType**|integer|Type of event that has occurred.<br>Possible values: <br>• `510` (application state-change)|
|**dateTime**|string|UTC timestamp of the event in the browser.<br>Format: *YYYY-MM-DDThh:mm:ss.SSSZ*|
|**payload**|JSON object|Information specific to the event generated. <br>(see [`event.data.payload` object](#eventdatapayload-object))|

<br>

### `event.data.payload` object

**Required items appear in bold type.**  

|Name|Type|Description|
|:-------|:---|:----------|
|**value**|string|Possible values:<br>• `loaded` (Netverify loaded in the user's browser.)<br>• `success` (Images were accepted for verification.)<br>• `error` (Verification could not be completed due to an error.)|
|metainfo|JSON object|Additional meta-information for error events. <br>(see [`metainfo` object](#metainfo-object))|

<br>

### `event.data.payload.metainfo` object

**Required items appear in bold type.**

|Property|Type|Description|
|:-------|:---|:----------|
|**codeExternal**|integer|[see **errorCode** values](#after-the-user-journey)|
<br>

### Example iFrame logging code
~~~javascript
function receiveMessage(event) {
	var data = window.JSON.parse(event.data);
	console.log('Netverify Web was loaded in an iframe.');
	console.log('auth token:', data.authorizationToken);
	console.log('transaction reference:', data.transactionReference);
	console.log('customer internal reference:', data.customerInternalReference);
	console.log('event type:', data.eventType);
	console.log('date-time:', data.dateTime);
	console.log('event value:', data.payload.value);
	console.log('event metainfo:', data.payload.metainfo);
}
window.addEventListener("message", receiveMessage, false);
~~~

<br>

## After the user journey

At the end of the user journey, the user is directed to your **Success URL** if the images they submitted were accepted for processing. If no **Success URL** has been defined, the Jumio default success page will be displayed, including any custom success [image](#images) you have specified in the Customer Portal.

If acceptable images are not provided after three attempts (see [Reject reasons](/netverify/callback.md#reject-reason)), the user is directed to your **Error URL**. If no **Error URL** has been defined, the Jumio default error page will be displayed, including any custom error [image](#images) you have specified in the Customer Portal.

To display relevant information on your success or error page, you can use the following parameters which we append when redirecting to your `successUrl` or `errorUrl` as HTTP `GET` query string parameters<sup>1</sup>. It is also possible to set `successUrl` and `errorUrl` to the same address, by using the query parameter `transactionStatus`.<br>

**Required items appear in bold type.**

|Name|Description|
|:---|:---|
|**transactionStatus**|Possible values:<br>• `SUCCESS` for successful submissions. <br> • `ERROR`for errors and failure after 3 attempts.|
|**customerInternalReference**|Your internal reference for the transaction.|
|**transactionReference**|Jumio reference number for the transaction.|
|errorCode|Displayed when `transactionStatus` is `ERROR`.<br>Possible values: <br>• `9100` (Error occurred on our server.)<br>• `9200` (Authorization token missing, invalid, or expired.)<br>• `9210` (Session expired after the user journey started.)<br>• `9300` (Error occurred transmitting image to our server.)<br>• `9400` (Error occurred during verification step.)<br>• `9800` (User has no network connection.)<br>• `9801` (Unexpected error occurred in the client.)<br>• `9810` (Problem while communicating with our server.)<br>• `9820` (File upload not enabled and camera unavailable.)<br>• `9835` (No acceptable submission in 3 attempts.)|

<sup>1</sup> Because HTTP `GET` parameters can be manipulated on the client side, they may be used for display purposes only.

### Sample success redirect

```
https://www.yourcompany.com/success/?transactionStatus=SUCCESS&customerInternalReference=YOUR_REF&transactionReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

### Sample error redirect

```
https://www.yourcompany.com/error/?transactionStatus=ERROR&customerInternalReference=YOUR_REF&transactionReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&errorCode=9820
```


---
## Supported browsers

Jumio offers guaranteed support for Netverify on the following browsers and the latest major version of each operating system.



### Desktop

|Browser|Major version|Operating system |Supports<br>image upload |Supports<br>HTML5 video stream |
|:---|:---|:---|:---:|:---:|
|Google Chrome|current +<br> 1 previous|Windows + Mac|X|X|
|Mozilla Firefox|current +<br>1 previous|Windows + Mac|X|X|
|Apple Safari|current|Mac|X|X|
|Microsoft Internet Explorer|current|Windows|X| |
|Microsoft Edge|current|Windows|X|X|


### Mobile

Netverify Web v4 does not support WebViews.

|Browser name|Major browser version|Operating system |Supports<br>image upload |Supports<br>HTML5 video stream |
|:---|:---|:---|:---:|:---:|
|Google Chrome |current |Android|X|X|
|Apple Safari |current |iOS|X|X|




---
&copy; Jumio Corp. 268 Lambert Avenue, Palo Alto, CA 94306
