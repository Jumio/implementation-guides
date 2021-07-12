![Jumio](/images/Jumio-ID-Verification-Banner.png)

# Configuring Settings in the Customer Portal

You can customize your settings, brand your ID Verification page, and manage your API credentials in the Customer Portal. Save changes with your Customer Portal password to activate them.

### Revision history

Information about changes to features and improvements documented in each release is available in our [Revision history](/netverify/README.md).
<br>


## Table of Contents
- [Application settings - General](#application-settings--general)
  - [Callback, Error, and Success URLs](#callback-error-and-success-urls)
  - [Select Capture Method](#select-capture-method)
  - [Skip "Start ID verification" screen](#skip-start-id-verification-screen)
- [Application settings - Redirect](#application-settings--redirect)
  - [Domain Name Prefix](#domain-name-prefix)
  - [Default Locale](#default-locale)
- [Customize client](#customize-client)
  - [Images](#images)
- [Accepted IDs](#accepted-ids)
- [Data settings](#data-settings)
    - [Mandatory Fields](#mandatory-fields)
    - [Optional Fields](#optional-fields)
    - [Data Retention Fields](#data-retention)
- [Multi Documents](#multi-documents)
    - [Custom](#custom)
- [Two-factor Authentication](#two-factor-authentication)

---
## Application Settings — General

### Callback, Error, and Success URLs

Define a **Callback URL** to automatically receive verification results and extracted user data from Jumio when a transaction is completed.

Define a **Success URL** to direct your customer after a successful user journey.

Define an **Error URL** to direct your customer when the verification process ends with an error or a failed submission attempt.


#### Required:

* HTTPS using the [TLS Protocol](https://tools.ietf.org/html/rfc5246) (most recent version recommended)
* Valid URL [(RFC-3986)](https://tools.ietf.org/html/rfc3986) using [ASCII characters](https://tools.ietf.org/html/rfc2046) or [IDNA Punycode](https://tools.ietf.org/html/rfc3492)

#### Forbidden:

* IP addresses, ports, certain query parameters used by Jumio, and fragment identifiers are not allowed.
* Personally identifiable information (PII) is not allowed in any form.
* Domains that do not belong to the customer (Using a callback URL that does not belong to the customer will expose PII data to an unknown endpoint. Using domains that do not belong to the customer will spam third party services which is not allowed by our policies.)

Jumio appends the following parameters to your Success or Error URL to redirect your user at the conclusion of the user journey. These cannot be used as part of your Success or Error URL:

|Name|Description|
|:---|:---|
|transactionStatus| • `SUCCESS` for successful submissions. <br> • `ERROR`for errors and failure after 3 attempts.|
|customerInternalReference|Your internal reference for the transaction.|
|transactionReference|Jumio reference number for the transaction.|
|errorCode|Displayed when `transactionStatus` is `ERROR`.|


### Select Capture Method

Here you can specify how your user can submit an ID or Identity image for verification.<br>
<br>
Choose from:

- **Webcam and image upload**
- **Webcam only**
- **Image upload only**



|⚠️  Selecting "Webcam only" means some mobile browsers will not be supported.
|:----------|
<br>

### Skip "Start ID verification" screen

Select this checkbox to bypass the introductory screen in the ID Verification client.

---
## Application Settings — Redirect

### Domain Name Prefix

You can optionally define a domain name prefix (`https://example.netverify.com`) for the URL of your ID Verification page.


- Allowed characters are: letters `a-z`, numbers `0-9`, `-`
- Must not start or end with: `-`
- Max. 63 characters

### Default Locale

Select a language from the dropdown list to set your display language for ID Verification. If no language is selected, ID Verification will be displayed in English (US).<br>

Choose from:<br>

- Arabic
- Armenian
- Bulgarian
- Chinese (China)
- Chinese (Hong Kong)
- Croatian
- Czech
- Danish
- Dutch
- English
- English (United Kingdom)
- Estonian
- Finnish
- French
- Georgian
- German
- Greek
- Hebrew
- Hungarian
- Indonesian
- Italian
- Japanese
- Khmer
- Korean
- Lithuanian
- Malay
- Norwegian
- Polish
- Portuguese
- Portuguese (Brazil)
- Romanian
- Russian
- Slovak
- Spanish
- Spanish (Mexico)
- Swedish
- Thai
- Turkish
- Vietnamese

---
## Customize client

### Colors

Specify primary and secondary colors for each locale to give ID Verification your own look and feel.

Any configuration which is not set will first default to the root language (e.g. EN\_GB to EN), then to your default setup, and finally to the Jumio default.

You can also reset all colors to the Jumio default.


### Images

Add a **Header image** for each locale to brand your ID Verification page.

Add a **Success image** and **Error image** for each locale to be displayed on the Jumio default success and error pages when you do not specify your own **successUrl** and **errorUrl**.

Any configuration which is not set will first default to the root language (e.g. EN\_GB to EN), then to your default setup, and finally to the Jumio default.

All images must be formatted as [JPG](https://jpeg.org/jpeg/) or [PNG](https://en.wikipedia.org/wiki/Portable_Network_Graphics) and must not exceed 5 MB.

---
## Accepted IDs
You can configure accepted IDs **Per region** or **Per country**.

To configure **Per region**, select **All**, **None**, or **Jumio supported** from the dropdown. Your region selections can be copied to the **Per country** configuration page for further customization. The default includes all countries and ID types supported by Jumio at the time when your account was created.

Select the checkbox to automatically accept newly supported IDs by Jumio.

---
## Data Settings
You can choose which fields should be extracted during the ID verification process.

### Mandatory Fields
Mandatory fields will be returned in the callback for all Jumio supported IDs, if enabled.
* Date of birth
* ID number
* First name
* Last name

### Optional Fields
Optional fields will be returned in the callback under certain conditions, if enabled.
* Expiry Date (if availble on the ID)
* US state (if US ID)
* Personal number (if available on the ID)
* MRZ check (if passport for MRZ check supported for the ID card)

**Note:** To perform the MRZ check, the fields date of birth, expiry and personal number will be processed during the ID verification and returned in the callback, if available on the ID.

**Supported countries for ID card MRZ check:**<br>

- Albania
- Argentina
- Austria
- Bosnia & Herzegovina
- Bulgaria
- Chile
- Croatia
- Czech Republic
- Denmark
- Dominican Republic
- Ecuador
- El Salvador
- Estonia
- Finland
- France
- Georgia
- Germany
- Guatemala
- Hungary
- Italy
- Kazakhstan
- Kenya
- Latvia
- Liechtenstein
- Lithuania
- Macedonia
- Malta
- Mexico
- Moldova
- Montenegro
- Netherlands
- Pakistan
- Paraguay
- Peru
- Poland
- Portugal
- Romania
- Serbia
- Slovakia
- Slovenia
- Spain
- Sweden
- Switzerland
- Turkey
- United Arab Emirates

### Data Retention
Select a time interval for permanent purge of sensitive data (360 days by default) or keep your data forever. The deletion of fraud transactions can be enabled (excluded by default).

If you update your data retention settings to a shorter span of time, records falling in the range of dates between your previous and new settings will be deleted automatically.

---
## Multi Documents

### Custom

You can create your own custom document types for Document Verification transactions. When submitting a new custom document type, it may take up to five minutes for your new custom document to appear in the Customer Portal.

Specify all three of the following:
* A unique document code (constraint: blanks are not allowed; max. length: 10 characters)
* A name
* A default label name in English

It is possible to add translations for different languages.

**Note:** The name and default label can be updated after creation. The document code cannot be changed. Custom document types can be disabled but not deleted, and the document code cannot be reused.

---

# Two-factor Authentication

If you want to enable two-factor authentication for your Jumio customer portal please contact support@jumio.com. Once enabled, users will be guided through the setup upon their first login to obtain a security code using the "Google Authenticator" app.


---
&copy; Jumio Corporation, 395 Page Mill Road, Suite 150 Palo Alto, CA 94306
