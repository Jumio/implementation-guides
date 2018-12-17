![Jumio](/images/netverify.png)

# performNetverify Implementation Guide

This is a reference manual and configuration guide for the performNetverify product. It illustrates how to implement the performNetverify API.

---
### Revision history

Information about changes to features and improvements documented in each release is available in our [Revision history](/netverify/README.md).
<br>
---

## Table of Contents
- [Release notes](#release-notes)
- [Using performNetverify](#using-performnetverify)
  - [Authentication and encryption](#authentication-and-encryption)
  - [Request headers](#request-headers)
  - [Request body](#request-body)
  - [Response](#response)
  - [Examples](#examples)
    - [Sample request](#sample-request)
    - [Sample response](#sample-response)
- [Configuring settings in the Customer Portal](#configuring-settings-in-the-customer-portal)
	- [Application settings - General](#application-settings--general)
		- [Callback URL](#callback-url)



---
# Using performNetverify

**performNetverify** offers a Restful ID verification API without Jumio-hosted user interface. Simply send an HTTP POST including the customer's ID image to the URL below. You will immediately receive a JSON response with a Jumio scan reference and a timestamp. A callback will be sent after the verification is complete (see [Callback](/netverify/callback.md)).

**HTTP Request Method:** `POST`<br>
**REST URL (US)**: `https://netverify.com/api/netverify/v2/performNetverify`<br>
**REST URL (EU)**: `https://lon.netverify.com/api/netverify/v2/performNetverify`<br>

## Authentication and encryption

Netverify API calls are protected using [HTTP Basic Authentication](https://tools.ietf.org/html/rfc7617). Your Basic Auth credentials are constructed using your API token as the user-id and your API secret as the password. You can view and manage your API token and secret in the Customer Portal under **Settings > API credentials**.
<br>

|⚠️ Never share your API token, API secret, or Basic Auth credentials with *anyone* — not even Jumio Support.
|:----------|

The [TLS Protocol](https://tools.ietf.org/html/rfc5246) is required to securely transmit your data, and we strongly recommend using the latest version. For information on cipher suites supported by Jumio during the TLS handshake see [supported cipher suites](/netverify/supported-cipher-suites.md).

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

**Note:** Calls with missing or suspicious headers, suspicious paramter values, or without HTTP Basic Authentication will result in HTTP status code 403 Forbidden.
<br>

## Request body

|ℹ️ Values set in your API request will override the corresponding settings configured in the Customer Portal.
|:----------|

**Required fields appear in bold type.**

|Parameter|Type|Max. length|Description|
|:---|:---|:---|:---|
|**merchantIdScanReference** \*|String|100|Your reference for each scan must not contain sensitive data like PII (Personally Identifiable Information) or account login|
|**frontsideImage** \*|String|Max. 5MB & <8000 pixels per side|Base64 encoded image of ID front side|
|**faceImage** \*|String|Max. 5MB & <8000 pixels per side|Base64 encoded image of face<br> **\*Mandatory if Face match enabled**|
|**country** \*|String|3|Possible countries:<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>• XKX (Kosovo)|
|**idType** \*|String|255|PASSPORT, DRIVING\_LICENSE, ID\_CARD, VISA|
|frontsideImageMimeType|String||Mime type of front side image<br>Possible values: image/jpeg (default), image/png|
|faceImageMimeType|String||Mime type of face image<br>Possible values: image/jpeg (default), image/png|
|backsideImage|String|Max. 5MB & <8000 pixels per side|Base64 encoded image of ID back side |
|backsideImageMimeType|String||Mime type of back side image<br>Possible values: image/jpeg (default), image/png|
|enabledFields|String|100|Defines fields which will be extracted during the ID verification. If a field is not listed in this parameter, it will not be processed for this transaction, regardless of customer portal settings.<br> **Note:** Face match and Address extraction will not be processed unless enabled for your account. If you want to enable them, please contact your Customer Success Manager, or reach out to Jumio Support.<br>See [supported documents for address extraction](#supported-documents-for-address-extraction).<br>Possible values:<br>"idNumber,idFirstName,idLastName,idDob, idExpiry,idUsState,idPersonalNumber,idFaceMatch,idAddress"|
|merchantReportingCriteria|String|100|Your reporting criteria for each scan|
|customerId|String|100|Identification of the customer must not contain sensitive data like PII (Personally Identificable Information) or account login|
|callbackUrl|String|255|Callback URL for the confirmation after the verification is completed (for constraints see [Callback URL](/netverify/portal-settings.md#callback-url))|
|firstName|String|100|First name of the customer|
|lastName|String|100|Last name of the customer|
|usState|String|100|Possible values if idType = PASSPORT or ID\_CARD:<br>• Last two characters of [ISO 3166-2: US](http://en.wikipedia.org/wiki/ISO_3166-2:US) state code<br>• [ISO 3166-1](http://en.wikipedia.org/wiki/ISO_3166-1) country name<br>• Kosovo<br>• [ISO 3166-1 alpha-2](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) country code<br>• XK (Kosovo)<br>• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br>• XKX (Kosovo)<br><br>If idType = DRIVING\_LICENSE:<br>• Last two characters of [ISO 3166-2: US state code](http://en.wikipedia.org/wiki/ISO_3166-2:US)|
|expiry|String||Date of expiry in the format YYYY-MM-DD|
|number|String|100|Identification number of the document|
|dob|String||Date of birth in the format YYYY-MM-DD|
|callbackGranularity|String|255|Possible values:<br>• onFinish (default): Callback is only sent after the whole verification<br>• onAllSteps: Additional callback is sent when the images are received|
|personalNumber|String|14|Personal number of the document|

<br>

### Supported documents for address extraction

|Country    |ID card    |Driving license    |Passport    |Callback format |
|:------------|:-------|:--------------|:--------------|:-------|
|Australia|No|Yes|No|US|
|Canada|No|Yes|No|US|
|France|Yes|Yes|Yes|Raw|
|Germany|Yes|No|No|EU|
|Ireland|No|Yes|No|Raw|
|Mexico|Yes|No|No|US|
|Romania|Yes|No|No|Raw|
|Singapore|Yes|No|No|Raw|
|Spain|Yes|No|No|EU|
|United Kingdom|No|Yes|No|Raw|
|United States|No|Yes|No|US|
---
## Response

|Parameter|Type|Max Length|Description|
|:----|:----|:----|:----|
|timestamp|String||Timestamp (UTC) of the response <br>format: YYYY-MM-DDThh:mm:ss.SSSZ|
|jumioIdScanReference|String|36|Jumio's reference number for each scan|

---
## Examples
### Sample request

```
POST https://netverify.com/api/netverify/v2/performNetverify HTTP/1.1
Accept: application/json
Content-Type: application/json
Content-Length: 1234
User-Agent: Example Corp SampleApp/1.0.1
Authorization: Basic xxxxxxxxxxxxxxxxxxxxxxxxxxxxx

{
  "merchantIdScanReference": "YOURSCANREFERENCE",
  "frontsideImage": "YOURBASE64STRING",
  "country": "XKX",
  "idType": "PASSPORT"
}
```
|⚠️ Sample requests cannot be run as-is. Replace example data with your own parameter values.
|:----------|
<br>

### Sample Response
```
{
  "timestamp": "2017-08-16T10:37:44.623Z",
  "jumioIdScanReference": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

---
# Configuring settings in the Customer Portal

In the **Settings** screen of the Customer Portal you can customize your settings and brand your Netverify page. <br>Save changes using your Customer Portal password to activate them.
<br>

|ℹ️ Values set in your API request will override the corresponding settings configured in the Customer Portal.
|:----------|

<br>

## Application settings — General

### Callback URL

Define a **Callback URL** to receive verification results and extracted user data from Jumio when a transaction is completed. For more information, see our [Callback](/netverify/callback.md) documentation.

#### URL requirements:

* HTTPS using the [TLS Protocol](https://tools.ietf.org/html/rfc5246) (most recent version recommended)
* Valid [URL](https://tools.ietf.org/html/rfc3986) using [ASCII characters](https://en.wikipedia.org/wiki/ASCII) or [IDNA Punycode](https://tools.ietf.org/html/rfc3492)

#### URL restrictions:

* IP addresses, ports, query parameters and fragment identifiers are not allowed.
* Personally identifiable information (PII) is not allowed in any form.


---
&copy; Jumio Corp. 268 Lambert Avenue, Palo Alto, CA 94306
