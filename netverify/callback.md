![Jumio](/images/jumio_feature_graphic.jpg)

# Callback

The callback is the authoritative answer from Jumio. Specify a callback URL (for constraints see [Configuring Settings in the Customer Portal](/netverify/portal-settings.md#callback-error-and-success-urls)) to automatically receive the result for each transaction.

### Revision history

Information about changes to features and improvements documented in each release is available in our [Revision history](/netverify/README.md).

## Table of contents

- [Jumio Callback IP List](#jumio-callback-ip-list)
- [Callback for Netverify](#callback-for-netverify)
  - [Supported Documents for Address Extraction](#supported-documents-for-address-extraction)
  - [Netverify masking](#netverify-masking)
- [Callback for Authentication](#callback-for-authentication)
- [Callback for Document Verification](#callback-for-document-verification)
  - [Supported Documents for Data Extraction](#supported-documents-for-data-extraction)
- [Netverify Retrieval API](#netverify-retrieval-api)



---

## Jumio callback IP list

Whitelist these IP addresses for callbacks, and use them to verify that the callback originated from Jumio.

**US data center:** </br>

34.202.241.227<br>
34.226.103.119<br>
34.226.254.127<br>
52.52.51.178<br>
52.53.95.123<br>
54.67.101.173<br>

Use the hostname `callback.jumio.com` to look up the most current IP addresses.<br>

**EU data center:**</br>

34.253.41.236<br>
35.157.27.193<br>
52.48.0.25<br>
52.57.194.92<br>
52.58.113.86<br>
52.209.180.134<br>

Use the hostname `callback.lon.jumio.com` to look up the most current IP addresses.<br>

**SGP data center:**</br>

3.0.109.121<br>
52.76.184.73<br>
52.77.102.92<br>

Use the hostname `callback.core-sgp.jumio.com` to look up the most current IP addresses.<br>

## Callback for Netverify

An HTTP POST request is sent to your specified callback URL containing an `application/x-www-form-urlencoded` formatted string with the transaction result.

To specify a global callback URL in the Customer Portal, see [Configuring Settings in the Customer Portal](/netverify/portal-settings.md#callback-error-and-success-urls).

A callback URL can also be specified per transaction. See instructions for [Netverify Web v4](/netverify/netverify-web-v4.md#request-body), [performNetverify](/netverify/performNetverify.md#request-body), and our SDK for [Android](https://github.com/Jumio/mobile-sdk-android/blob/master/docs/integration_authentication.md#callback) and [iOS](https://github.com/Jumio/mobile-sdk-ios/blob/master/docs/integration_authentication.md#configuration).

For Android/iOS: ID verification must be enabled to receive the callback.

#### Netverify Web

|User journey state       | transaction state    | Callback|
|:---------------|:--------|:------------|
|Not started |Pending => Failed|Transaction will be cleaned-up from pending to failed after the authorization token lifetime expires<br>Callback: Verification status NO\_ID\_UPLOADED |
|Drop off during first attept| Pending => Failed| Transaction will be finished 15 minutes after the last update<br>Callback: Verification status NO\_ID\_UPLOADED|
|Drop off during second or third attempt  | Done  |Transaction will be finished 15 minutes after the last update<br>Callback: Verification status ERROR\_NOT\_READABLE\_ID with previous reject reason |
|Finished       | Done  |Callback: Verification status depends on the result |

#### Netverify Mobile (Android/iOS)

|User journey state       | transaction state    | Callback|
|:---------------|:--------|:------------|
|Drop off   | Pending => Failed  |Transaction will be cleaned-up from pending to failed 15 minutes after last update<br/>Callback: verification status NO\_ID\_UPLOADED|
|Finished   | Done  |ID verification will be performed<br/>Callback: verification status depends on the result (see table below)|


### Parameters

The following parameters are posted to your callback URL for Netverify Web, performNetverify and Netverify Mobile iOS/Android.

**Required items appear in bold type.**  

|Parameter       | Max. Length    | Description| Notes |
|:---------------|:--------|:------------| ------ |
|**callBackType**   |   |NETVERIFYID | |
|**jumioIdScanReference**   |36  |Jumio’s reference number for each transaction | |
|**verificationStatus**   |   |Possible states:<br/>•	APPROVED\_VERIFIED<br/> •	DENIED\_FRAUD<br/>•	DENIED\_UNSUPPORTED\_ID\_TYPE <sup>1</sup><br/> •	DENIED\_UNSUPPORTED\_ID\_COUNTRY <sup>1</sup><br/>•	ERROR\_NOT\_READABLE\_ID<br/> •	NO\_ID\_UPLOADED | |
|**idScanStatus**  |   |SUCCESS if verificationStatus = APPROVED\_VERIFIED, otherwise ERROR | |
|**idScanSource**  |   |Possible values:<br>•	WEB (NVW3 / NVW4 before capture method is selected)<br>•	WEB\_CAM (NVW3 / NVW4 with camera capture)<br>•	WEB\_UPLOAD (NVW3 / NVW4 with file upload)<br>•	REDIRECT (NVW3 redirect before capture method is selected)<br>•	REDIRECT\_CAM (NVW3 redirect with camera capture)<br>•	REDIRECT\_UPLOAD (NVW3 redirect with file upload)<br>•	API (performNetverify)<br>• SDK (mobile) | |
|**idCheckDataPositions**   |   |•	OK if verificationStatus = APPROVED\_VERIFIED<br> •	otherwise N/A | |
|**idCheckDocumentValidation**   |   |•	OK if verificationStatus = APPROVED\_VERIFIED<br> •	otherwise N/A  | |
|**idCheckHologram**   |   |•	OK if verificationStatus = APPROVED\_VERIFIED<br> •	otherwise N/A | |
|**idCheckMRZcode**  |   |•	OK for passports and supported ID cards if verificationStatus = APPROVED\_VERIFIED and MRZ check is enabled<br> •	otherwise N/A  |not returned for DEU, HKG, JPN, NLD, SGP, KOR if NV masking is enabled <sup>4</sup>|
|**idCheckMicroprint** |   |•	OK if verificationStatus = APPROVED\_VERIFIED<br> •	otherwise N/A | |
|**idCheckSecurityFeatures**   |   |•	OK if verificationStatus = APPROVED\_VERIFIED<br> •	otherwise N/A  | |
|**idCheckSignature**  |   |•	OK if verificationStatus = APPROVED\_VERIFIED<br> •	otherwise N/A | |
|**transactionDate**   |   |Timestamp (UTC) of the transaction creation<br>format: [YYYY-MM-DDThh:mm:ss.SSSZ](#timestamp-format) | |
|**callbackDate** |   |Timestamp  (UTC) of the callback creation <br>format: [YYYY-MM-DDThh:mm:ss.SSSZ](#timestamp-format) | |
|identityVerification   |   |Identity verification as JSON object, <br /> **ONLY** if verificationStatus = APPROVED\_VERIFIED <br/><br/> see table [Identity Verification](#identity-verification) below|activation required |
|idType   |   |Possible types:<br/>•	PASSPORT<br/>•	DRIVING\_LICENSE<br/>•	ID\_CARD<br>•	VISA *<br/><br/>* Currently, Jumio only supports US and China visas in certain cases. Visas from other countries will be rejected as unsupported with idType = VISA | |
|idSubtype   |255  |Possible subtypes if idType = ID\_CARD<br/>•	NATIONAL\_ID<br/>•	CONSULAR\_ID<br/>•	ELECTORAL\_ID<br/>•	RESIDENT\_PERMIT\_ID<br/>•	TAX\_ID <br/>•	STUDENT\_ID <br/>•	PASSPORT\_CARD\_ID <br/>•	MILITARY\_ID <br/>•	PUBLIC\_SAFETY\_ID <br/>•	HEALTH\_ID <br/>• OTHER\_ID<br/>•	VISA <br/>•	UNKNOWN<br/><br/>Possible subtypes if idType = DRIVING\_LICENSE<br/>•	REGULAR\_DRIVING\_LICENSE<br/>•	LEARNING\_DRIVING\_LICENSE <br/><br/>Possible subtypes if idType = PASSPORT<br/>•	E\_PASSPORT (only for mobile) | |
|idCountry   |3  |Possible countries:<br/>•	[ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br/>•	XKX (Kosovo)| |
|rejectReason   |   |Reject reason as JSON object if verificationStatus = DENIED\_FRAUD or ERROR\_NOT\_READABLE\_ID, see tables below  | |
|idScanImage   |255  |URL to retrieve the image of the transaction (JPEG or PNG) if available <sup>2</sup> | |
|idScanImageFace   |255  |URL to retrieve the face image of the transaction (JPEG or PNG) if available <sup>2</sup>| |
|idScanImageBackside   |255  |URL to retrieve the back	side image of the transaction (JPEG or PNG) if available <sup>2</sup>| |
|idNumber   |200  |Identification number of the document as available on the ID if enabled, otherwise if provided | |
|idFirstName   |200  |•	First name of the customer as available on the ID if enabled, otherwise if provided<br/><br>•	For following documents, `N/A` returned (if name contains non-Latin characters)<br /> -	if idCountry = CHN and idType = DRIVING\_LICENSE or ID\_CARD<br> -	if idCountry = KOR and idType = DRIVING\_LICENSE or ID\_CARD<br> -	if idCountry = JPN and idType = DRIVING\_LICENSE<br> -	if idCountry = RUS and idType = ID\_CARD| |
|idLastName   |200  |•	Last name of the customer as available on the ID if enabled, otherwise if provided<br/><br>•	For following documents, Chinese name as printed on the document returned (if enabled) <br /> -	if idCountry = CHN and idType = DRIVING\_LICENSE or ID\_CARD <br>(first name and last name)<br><br>•	Only if full name is printed in Latin characters<br /> - if idCountry = KOR and idType = DRIVING\_LICENSE <br>(first name and last name)<br /><br> •	For following documents, `N/A` returned (if name contains non-Latin characters) <br /> -	if idCountry = CHN and idType = DRIVING\_LICENSE or ID\_CARD<br> -	if idCountry = KOR and idType = DRIVING\_LICENSE or ID\_CARD<br> -	if idCountry = JPN and idType = DRIVING\_LICENSE<br> -	if idCountry = RUS and idType = ID\_CARD |activation required for Chinese name extraction |
|idDob   |10  |Date of birth in the format YYYY-MM-DD as available on the ID if enabled, otherwise if provided <br /><br />If idCountry = HKG, IND date of birth can be incomplete, possible values e.g.:<br />•	Year-Month-Day: 1990-12-09 <br />•	Year only: 1990-01-01 <br />•	Year-Month: 1990-12-01 <br />•	Year-Day: 1990-01-09<br />Additional parameter `originDob` will be provided|not returned for KOR if NV masking is enabled <sup>4</sup>|
|idExpiry   |10  |Date of expiry in the format YYYY-MM-DD as available on the ID if enabled, otherwise if provided | |
|idUsState   |255  |Possible values if idType = PASSPORT or ID\_CARD:<br/>•	Last two characters of [ISO 3166-2:US](http://en.wikipedia.org/wiki/ISO_3166-2:US) state code<br/>•	Last 2-3 characters of [ISO 3166-2:AU](http://en.wikipedia.org/wiki/ISO_3166-2:AU) state code<br/>•	Last two characters of [ISO 3166-2:CA](http://en.wikipedia.org/wiki/ISO_3166-2:CA) state code<br/>• [ISO 3166-1](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country name<br/>• XKX (Kosovo)<br/>• Free text - if it can't be mapped to a state/country code<br/><br/>If idType = DRIVING\_LICENSE:<br/>•	Last two characters of [ISO 3166-2:US](http://en.wikipedia.org/wiki/ISO_3166-2:US) state code<br/>•	Last 2-3 characters of [ISO 3166-2:AU](http://en.wikipedia.org/wiki/ISO_3166-2:AU) state code<br/>•	Last two characters of [ISO 3166-2:CA](http://en.wikipedia.org/wiki/ISO_3166-2:CA) state code| |
|personalNumber   |14  |Personal number of the document • if idType = PASSPORT and if data available on the document |not returned for NLD, DEU, KOR if NV masking is enabled <sup>4</sup> |
|idAddress   |   |Address as JSON object in US, EU or raw format, see tables below <sup>3</sup> |activation required |
|merchantIdScanReference   |100  |Your reference for each transaction | |
|merchantReportingCriteria   |100  |Your reporting criteria for each transaction | |
|customerId   |100  |ID of the customer as provided | |
|clientIp   |   |IP address of the client in the format xxx.xxx.xxx.xxx | |
|firstAttemptDate   |  |Timestamp (UTC) of the first transaction attempt <br>format: [YYYY-MM-DDThh:mm:ss.SSSZ](#timestamp-format) | |
|optionalData1   |255  |Optional field of MRZ line 1 | not returned for NLD ID if NV masking is enabled <sup>4</sup>|
|optionalData2   |255  |Optional field of MRZ line 2 | |
|dni   |255  |DNI as available on the ID if idCountry = ESP and idSubtype = NATIONAL\_ID  | |
|curp   |255  |CURP is available if idCountry = MEX and idType = PASSPORT, DRIVING\_LICENSE, or ID\_CARD and idSubtype = ELECTORAL\_ID  |activation required |
|gender   |2  |Possible values: M, F<br>• if idCountry = FRA,HKG and idSubtype = NATIONAL\_ID (MRZ type CNIS)<br> •	if idCountry = BHR,SGP and idType = PASSPORT, ID\_CARD, DRIVING\_LICENSE <br>• if idCountry = PHL and idType = DRIVING\_LICENSE <br>• if idType = VISA and additional extraction for Visa enabled <br>• if readable: best effort| |
|presetCountry   | 3  |Possible countries:<br />•	[ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code <br /> • XKX (Kosovo)| |
|presetIdType   |    |Possible ID types: PASSPORT, DRIVING\_LICENSE, ID\_CARD| |
|dlCarPermission|255 |Only available if:<br/> •Extraction supported for specific country<br/>•verificationStatus = APPROVED\_VERIFIED<br/><br/>Possible values:<br /> • YES<br /> • NO<br /> • NOT\_READABLE|activation required|
|dlCategories   | JSON object  |Driver license categories as JSON object, see table below <br /><br />Supported countries:<br />• Austria<br />• Belgium<br />• Bulgaria<br />• France<br />• Germany<br />• Great Britain<br />• Italy<br />• Latvia<br />• Lithuania<br />• Netherlands<br />• Romania<br />• Spain<br />• Taiwan|activation required |
|nationality |3| • if idCountry = BHR, HKG, MYS, PHL, SGP |activation required |
|passportNumber|255|Passport number if idType = VISA and additional extraction for Visa enabled|activation required |
|durationOfStay|255|Duration of stay if idType = VISA and additional extraction for Visa enabled|activation required |
|numberOfEntries|255|Number of entries if idType = VISA and additional extraction for Visa enabled|activation required |
|visaCategory|255|Visa category if idType = VISA and additional extraction for Visa enabled|activation required |
|originDob|10|Original format of date of birth if idCountry = IND <br/>Possible values e.g.: <br />• Year/Month/Day: 1990/12/09 <br />• Year only: 1990// <br />• Year/Month: 1990/12/ <br />• Year/Day: 1990//09|<br /> |
|issuingAuthority|50|Issuing authority of the document (if issuing authority extraction is enabled) <br>• if idCountry = ITA |activation required |
|issuingDate|10|Issuing date of the document (if issuing date extraction enabled) |activation required |
|issuingPlace|50|Issuing place of the document (if issuing place extraction is enabled)<br>• if idCountry = COL, ITA |activation required |
|livenessImages| |URLs to the liveness images of the transaction (JPEG or PNG) if available <sup>5</sup> | |
|placeOfBirth|256|Place of birth of document holder<br>• if idCountry = AFG, AGO, AIA, ALB, AND, ARE, ARM, ATG, AUS, AUT, AZE, BDI, BEL, BEN, BFA, BGD, BGR, BHR, BHS, BIH, BLR, BLZ, BMU, BOL, BRA, BRB, BRN, BTN, BWA, CAF, CAN, DEU, ESP, FRA, GBR, HKG, HUN, IDN, IND, IRL, ITA, NGA, NLD, PAK, PHL, POL, PRT, ROU, SGP, RUS, TUR, UKR, USA| |
|facemap|255|URL to the facemap of the transaction if available |activation required |
|taxNumber|255|Tax number of the document<br>• if idCountry = ITA and idType = HEALTH\_ID or TAX\_ID |activation required |
|cpf|255|CPF number of the document|activation required |
|registrationNumber|255|Registration number of the document |activation required |
|mothersName|255|Name of the document holder's mother |activation required |
|fathersName|255|Name of the document holder's father |activation required |
|personalIdentificationNumber|255|Personal identification number as available on the ID<br>• if idCountry = GEO and idSubtype = PASSPORT<br>• if idCountry = COL and idSubtype = ID\_CARD |activation required |
|rgNumber|255|"General Registration" number for idCountry = BRA |activation required|

<sup>1</sup> Transaction is declined as unsupported if the ID is not supported by Jumio, or not marked as accepted in your customer portal settings.<br/>
<sup>2</sup> For ID types that are configured to support a separate scan of the front side and back side, there is a separate image of the front side (idScanImage) and the back side (idScanImageBackside). If Identity Verification is enabled, there is also a picture of the face (idScanImageFace).<br>
<sup>3</sup> Address recognition is performed for supported IDs, if enabled. Please note, there are three different address formats (US, EU, Raw). Please check [Supported documents for Address Extraction](#supported-documents-for-address-extraction) to see which format applies to specific IDs. The different address parameters are a part of the JSON object, if they are available on the ID.<br>
<sup>4</sup> Fields containing certain kinds of personally identifying information are not returned if NV masking is enabled for the Netherlands, Germany, or South Korea. See [Netverify masking](#netverify-masking) for more information.<br>
<sup>5</sup> Liveness images are returned only for transactions containing Identity Verification submitted via the Android and iOS SDKs. The number of images can vary and may not be returned in chronological order.

#### Retrieving images
Use HTTP **GET** with **Basic Authorization** using your API token and secret as userid and password.<br>
**Header**: The following parameters are mandatory in the header section of your request.<br>
- `Accept: image/jpeg, image/png`<br>
- `User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`<br /><br />
The value for **User-Agent** must contain a reference to your business or entity for Jumio to be able to identify your requests. (e.g. YourCompanyName YourAppName/1.0.0). Without a proper User-Agent header, Jumio will take longer to diagnose API issues.<br>

The TLS protocol is required during the TLS handshake (see [Supported cipher suites](/netverify/supported-cipher-suites.md)) and we strongly recommend using the latest version.

#### Timestamp format

Timestamp are sent in format: `YYYY-MM-DDThh:mm:ss.SSSZ` with constraint that SSS (milliseconds) can either be
- Not included: `YYYY-MM-DDThh:mm:ssZ`
- 1 digit: `YYYY-MM-DDThh:mm:ss.SZ`
- 2 digits:  `YYYY-MM-DDThh:mm:ss.SSZ`
- 3 digits: `YYYY-MM-DDThh:mm:ss.SSSZ`

We encourage to use a standard library to convert the timestamp received from Jumio as the timeformat is valid with and without the SSS milliseconds. <br/><br>


### Supported documents for address extraction

|Country    |ID card    |Driving license    |Passport    |Callback format<br>(until August 15)|Callback format<br>(from August 15)<sup>1</sup>|
|:------------|:-------|:--------------|:--------------|:-------|:-------|
|Australia|No|Yes|No|US|Raw|
|Bahrain|No|Yes|No|Raw|Raw|
|Canada|No|Yes|No|US|Raw|
|France|Yes|Yes|Yes|Raw|Raw|
|Germany|Yes|No|No|EU|Raw|
|Indonesia|Yes|No|No|Raw|Raw|
|Ireland|No|Yes|No|Raw|Raw|
|Malaysia|Yes|No|No|Raw|Raw|
|Malta|Yes|Yes|No|Raw|Raw|
|Mexico|Yes|No|No|US|Raw|
|Peru|Yes|Yes|No|Raw|Raw|
|Romania|Yes|No|No|Raw|Raw|
|Singapore|Yes|No|No|Raw|Raw|
|Spain|Yes|No|No|EU|Raw|
|United Kingdom|No|Yes|No|Raw|Raw|
|United States|Yes|Yes|No|US|Raw|

<sup>1</sup> see [Upcoming Address format changes](#upcoming-address-format-changes)

#### US address format

|Parameter `idAddress`       | Max. Length    | Description|
|:---------------|:--------|:------------|
|city   |64  |City |
|stateCode   |6  |[ISO 3166-2](http://en.wikipedia.org/wiki/ISO_3166-2) state code |
|streetName   |64  |Street name |
|streetSuffix   |14  |Street suffix abbreviation<br/>Examples: [US](http://www.gis.co.clay.mn.us/USPS.htm#suffix), [Canada](http://www.canadapost.ca/tools/pg/manual/PGaddress-e.asp#1423617), [Australia](https://auspost.com.au/media/documents/australia-post-addressing-standards-1999.pdf) |
|streetDirection   |255  |Street direction abbreviation<br/>Examples: US (E=EAST, W=WEST, N=NORTH, S=SOUTH), [Canada](http://www.canadapost.ca/tools/pg/manual/PGaddress-e.asp#1403220), [Australia](https://auspost.com.au/media/documents/australia-post-addressing-standards-1999.pdf) |
|streetNumber   |14  |Street number |
|unitDesignator   |14  |Unit designator abbreviation<br/>Examples: [US](http://www.gis.co.clay.mn.us/USPS.htm#secunitdesig), [Canada](http://www.canadapost.ca/tools/pg/manual/PGaddress-e.asp#1380473), [Australia](https://auspost.com.au/media/documents/australia-post-addressing-standards-1999.pdf) |
|unitNumber   |14  |Unit number |
|zip   |14  |Zip code |
|zipExtension   |20  |Zip extension |
|country   |3  |Possible countries:<br/>•	[ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br/>•	XKX (Kosovo) |

#### EU address format

|Parameter `idAddress`       | Max. Length    | Description|
|:---------------|:--------|:------------|
|city   |64  |City |
|province   |64  |Province |
|streetName   |64  |Street name |
|streetNumber   |15  |Street number |
|unitDetails   |64  |Unit details |
|postalCode   |15  |Postal code |
|country   |3  |Possible countries:<br/>•	[ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br/>•	XKX (Kosovo) |

#### Raw address format

|Parameter `idAddress`      | Max. Length    | Description|
|:---------------|:--------|:------------|
|line1   |100  |Line item 1 |
|line2   |100  |Line item 2 |
|line3   |100  |Line item 3 |
|line4   |100  |Line item 4 |
|line5   |100  |Line item 5 |
|country   |3  |Possible countries:<br/>•	[ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br/>•	XKX (Kosovo)  |
|postalCode   |15  |Postal code |
|city   |64  |City |
|subdivision   |100  |Subdivision (Region, State, Province, Emirate, Department, ...) |

#### Upcoming Address format changes

On August 17, 2020 onwards, Jumio is going to streamline the EU and US format into a single Raw format. In addition to that, we will also be providing a new parameter called formattedAddress which will contain  the entire address in a comma separated format.

**Example**

|US Address Format|Raw Address Format|
|:------------------------|:--------|
|"streetNumber": "150"<br>"streetDirection":"N"<br>"streetName":"Alberny"<br>"streetSuffix":"Dr"<br>"unitDesignator":"APT"<br>"unitNumber": "7"|"line1":"150 N Alberny Dr APT. 7"|
|"city":"New Mexico"|city":"New Mexico"|
|"stateCode":"US-AL"|"subdivision":"US-AL"|
|"zip":"77358"<br>"zipExtension":"5555"|"postalCode":"77358-5555"|
|"country":"USA"|"country":"USA"|
||“formattedAddress”: “150 N Alberny Dr APT. 7, New Mexico, US-AL 77358-5555, United States”|

|EU Address Format|Raw Address Format|
|:------------------------|:--------|
|"streetNumber":"180"<br>"streetName":"Main Street"<br>"unitDetails":"ABC"|"line1": "180 Main Street ABC"|
|"city": "Graz"|"city": "Graz"|
|"province": "Steiermark"|"subdivision": "Steiermark"|
|"postalCode": "2424"|"postalCode": "2424"|
|"country": "AUT"|"country": "AUT"|
||“formattedAddress”: “180 Main Street ABC, Graz, Steiermark 2424, Austria”|

### Reject reason

|Parameter `rejectReason` | Type   | Max. Length    | Description|
|:------------------------|:--------|:--------|:------------|
|rejectReasonCode |String| 5  |see below |
|rejectReasonDescription |String |64  |Possible codes and descriptions for verification status DENIED\_FRAUD:<br>100	MANIPULATED\_DOCUMENT<br/>105	FRAUDSTER<br/>106	FAKE<br/>107	PHOTO\_MISMATCH<br/>108	MRZ\_CHECK\_FAILED<br/>109	PUNCHED\_DOCUMENT<br/>110	CHIP\_DATA\_MANIPULATED (only available for ePassport)<br/>111	MISMATCH\_PRINTED\_BARCODE_DATA<br><br>Possible codes and descriptions for verificationStatus = ERROR\_NOT\_READABLE\_ID:<br/>102	PHOTOCOPY\_BLACK\_WHITE<br/>103	PHOTOCOPY\_COLOR (for sources WEB\_CAM and REDIRECT\_CAM)<br/>104	DIGITAL\_COPY<br/>200	NOT\_READABLE\_DOCUMENT<br/>201	NO\_DOCUMENT<br/>202	SAMPLE\_DOCUMENT<br/>206	MISSING\_BACK<br/>207	WRONG\_DOCUMENT\_PAGE<br/>209	MISSING\_SIGNATURE<br/>210	CAMERA\_BLACK\_WHITE<br/>211	DIFFERENT\_PERSONS\_SHOWN (documents of multiple people in one image)<br/>300	MANUAL\_REJECTION|
|rejectReasonDetails |Object  |   |Reject reason details as JSON array containing JSON objects if rejectReasonCode = 100 or 200, see table below |


### Reject reason details

|Parameter `rejectReasonDetails` |  Type    | Max. Length    | Description|
|:-------------------------------|:---------|:---------------|:------------|
|detailsCode   |String | 5 | see below |
|detailsDescription|String|32| Possible codes and descriptions for rejectReasonCode = 100:<br/>1001	PHOTO<br/>1002	DOCUMENT\_NUMBER<br>1003	EXPIRY<br/>1004	DOB<br/>1005	NAME<br/>1006	ADDRESS<br/>1007	SECURITY\_CHECKS<br/>1008	SIGNATURE<br>1009	PERSONAL\_NUMBER<br>10011 PLACE_OF_BIRTH<br><br>Possible codes and descriptions for rejectReasonCode = 200:<br/>2001	BLURRED<br/>2002	BAD\_QUALITY<br/>2003	MISSING\_PART\_DOCUMENT<br/>2004	HIDDEN\_PART\_DOCUMENT<br/>2005	DAMAGED\_DOCUMENT<br/>2006 GLARE |


### Identity Verification

**Required items appear in bold type.**  

|Parameter `identityVerification`       | Max. Length    | Description|
|:---------------|:--------|:------------|
|**similarity** <sup>1</sup>   |  |Possible values:<br/> •	MATCH<br />•	NO\_MATCH<br />•	NOT\_POSSIBLE (not executed or bad quality of face on document)|
|**validity** <sup>2</sup>   |  |Possible values:<br/> •	TRUE<br />•	FALSE |
|reason   |  |Provided if validity = FALSE<br/>Possible values:<br />• SELFIE\_CROPPED\_FROM\_ID<br />•	ENTIRE\_ID\_USED\_AS\_SELFIE<br />•	MULTIPLE\_PEOPLE<br />•	SELFIE\_IS\_SCREEN\_PAPER\_VIDEO<br />•	SELFIE\_MANIPULATED<br />• AGE\_DIFFERENCE\_TOO\_BIG<br />•	NO\_FACE\_PRESENT<br />•	FACE\_NOT\_FULLY\_VISIBLE<br />•	BAD\_QUALITY<br />•	BLACK\_AND\_WHITE<br />•	LIVENESS\_FAILED <sup>3</sup>|
|similarityDecision   |  |Only visible if setting is turned on within your account. For questions about this feature, please contact your Support. <br/><br/>Possible values:<br/> •	MANUAL<br />•	AUTOMATED|
|similarityScore   |  |Only visible if setting is turned on within your account. For questions about this feature, please contact your Support.|

<!--- |handwrittenNoteMatches	|	|Only visible if setting is turned on within your account. For questions about this feature, please contact your Support. <br/><br/>Possible values:<br/> •	TRUE<br />•	FALSE| --->

<sup>1</sup> Is the person on the selfie the same as the one on the document?<br>
<sup>2</sup> Is it a live person?<br>
<sup>3</sup> Potential reasons `LIVENESS_FAILED`:

- User tries to spoof the system
- User does not want to show his face at all but wants to complete the onboarding
- User does not look straight into the camera
- User does not finish the the Identity Verification process
- User has bad lighting conditions (too dark, too bright, reflections on face, not enough contrast, …)
- User is covering (parts) of his face with a scarf, hat, or something similar
- User is not able to align his face with the oval
- A different person is performing the Identity Verification in the second step than in the first one

<br/>

### Driver License Categories

**Required items appear in bold type.**

|Parameter `dlCategories`       | Max. Length    | Description|
|:---------------|:--------|:------------|
|**category**  | 10 |String in Latin or Traditional Chinese characters|
|issueDate   |  |Issue date in the format YYYY-MM-DD|
|expiryDate   |  |Date of expiry in the format YYYY-MM-DD as available on the driver license|
|isReadable   |  |Possible value:<br>• FALSE |
|availability   |  |Possible values:<br>• yes<br />• no<br />• not readable|


## Netverify Masking

Extracting certain sensitive information from identity documents in the Netherlands, Germany, and South Korea is prohibited by law for customers with a business presence in those countries. These customers can elect to enable Netverify masking to protect this sensitive data.

When masking is enabled for these countries, certain fields cannot be extracted and returned in the callback. The information contained in these fields is masked before the user's image is stored.

|Country|Document|Affected parameters (not returned in the callback)|
|:---|:---|:---|
Germany|Passport|idCheckMRZcode, idNumber|
Germany|ID card|idCheckMRZcode, idNumber|
Hong Kong|Passport|idCheckMRZcode, idNumber|
Hong Kong|ID card|idNumber|
Japan|ID card|idNumber|
Netherlands|Passport|idCheckMRZcode, personalNumber|
Netherlands|ID card|idCheckMRZcode, personalNumber|
Netherlands|Driver license|image is masked, no extracted data fields are affected|
Singapore|Passport|idCheckMRZcode,idNumber|
Singapore|Driver license|idNumber|
Singapore|ID card|idNumber|
South Korea|Passport|idCheckMRZcode, personalNumber|
South Korea|ID card|idNumber, idDob|
South Korea|Driver license|idDob|

### Sample Callbacks

#### Sample callback (URL-encoded POST): Approved and verified

```
idExpiry=2022-12-31&idType=PASSPORT&idDob=1990-01-01&idCheckSignature=OK&idCheckDataPositions=OK&idCheckHologram=OK&idCheckMicroprint=OK&idCheckDocumentValidation=OK&idCountry=USA&idScanSource=SDK&idFirstName=FIRSTNAME&verificationStatus=APPROVED_VERIFIED&jumioIdScanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&personalNumber=N%2FA&merchantIdScanReference=YOURIDSCANREFERENCE&idCheckSecurityFeatures=OK&idCheckMRZcode=OK&idScanImage=https%3A%2F%2Fnetverify.com%2Frecognition%2Fv1%2Fidscan%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Ffront&callBackType=NETVERIFYID&clientIp=xxx.xxx.xxx.xxx&idLastName=LASTNAME&idAddress=%7B%22country%22%3A%22USA%22%2C%22stateCode%22%3A%22US%2DOH%22%7D&idScanStatus=SUCCESS&identityVerification=%7B%22similarity%22%3A%22MATCH%22%2C%22validity%22%3Atrue%7D&idNumber=P1234
```

#### Sample callback (URL-encoded POST): Fraud

```
idType=PASSPORT&idCheckSignature=N%2FA&rejectReason=%7B%20%22rejectReasonCode%22%3A%22100%22%2C%20%22rejectReasonDescription%22%3A%22MANIPULATED_DOCUMENT%22%2C%20%22rejectReasonDetails%22%3A%20%5B%7B%20%22detailsCode%22%3A%20%221001%22%2C%20%22detailsDescription%22%3A%20%22PHOTO%22%20%7D%2C%7B%20%22detailsCode%22%3A%20%221004%22%2C%20%22detailsDescription%22%3A%20%22DOB%22%20%7D%5D%7D&idCheckDataPositions=N%2FA&idCheckHologram=N%2FA&idCheckMicroprint=N%2FA&idCheckDocumentValidation=N%2FA&idCountry=USA&idScanSource=SDK&verificationStatus=DENIED_FRAUD&jumioIdScanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&merchantIdScanReference=YOURSCANREFERENCE&idCheckSecurityFeatures=N%2FA&idCheckMRZcode=N%2FA&idScanImage=https%3A%2F%2Fnetverify.com%2Frecognition%2Fv1%2Fidscan%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Ffront&callBackType=NETVERIFYID&clientIp=xxx.xxx.xxx.xxx&idScanStatus=ERROR
```
<!--
update sample callback fraud with extracted data TODO
-->
---

## Callback for Authentication

An HTTP POST request is sent to your specified callback URL containing an `application/x-www-form-urlencoded` formatted string with the transaction result.

To specify a global callback URL in the Customer Portal, see [Configuring Settings in the Customer Portal](/netverify/portal-settings.md#callback-error-and-success-urls).

A callback URL can also be specified per transaction in our [Android](https://github.com/Jumio/mobile-sdk-android/blob/master/docs/integration_authentication.md#callback) and [iOS](https://github.com/Jumio/mobile-sdk-ios/blob/master/docs/integration_authentication.md#configuration) SDK.


### Parameters

**Required items appear in bold type.**  

|Parameter       | Type    | Max. Length|  Description|
|:---------------|:--------|:----------: |:------------|
|**callbackDate**| string  |  | Timestamp of the callback in the format: <br>[YYYY-MM-DDThh:mm:ss.SSSZ](#timestamp-format-1)|
|**transactionReference**| string  |36 |Jumio’s reference number for the Authentication transaction|
|**enrollmentTransactionReference**| string  |36 |Jumio’s reference number of the enrollment transaction (ID)|
|**transactionResult**| string  | |Possible values:<br>•	PASSED<br>•	FAILED<br>•	INVALID<br>•	EXPIRED|
|**transactionDate**| string  |  |Timestamp of the transaction in the format: <br>[YYYY-MM-DDThh:mm:ss.SSSZ](#timestamp-format-1)|
|**scanSource**|string||Possible values:<br>•	SDK<br>•	WEB|
|**callBackType**|string ||NETVERIFY_AUTHENTICATION|
|idScanImageFace | JSON array/object | |URL to retrieve the face image of the transaction (JPEG or PNG) <sup>1</sup> |
|livenessImages| JSON array  |  | URLs to the liveness images of the transaction (JPEG or PNG) <sup>1</sup> <sup>2</sup>|
|userReference |string  |  |Your internal reference for the user. |

<sup>1</sup> Retrieve the images of the transaction.<br>
<sup>2</sup> The number of images can vary and may not be returned in chronological order.

#### Retrieving Images
Use HTTP **GET** with **Basic Authorization** using your API token and secret, as userid and password.<br>
**Header**: The following parameters are mandatory in the header section of your request.<br>
- `Accept: image/jpeg, image/png`<br>
- `User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`<br /><br />
The value for **User-Agent** must contain a reference to your business or entity for Jumio to be able to identify your requests. (e.g. YourCompanyName YourAppName/1.0.0). Without a proper User-Agent header, Jumio will take longer to diagnose API issues.<br>

The TLS protocol is required during the TLS handshake (see [Supported cipher suites](/netverify/supported-cipher-suites.md)) and we strongly recommend using the latest version.

#### Timestamp format

Timestamp are sent in format: `YYYY-MM-DDThh:mm:ss.SSSZ` with constraint that SSS (milliseconds) can either be
- Not included: `YYYY-MM-DDThh:mm:ssZ`
- 1 digit: `YYYY-MM-DDThh:mm:ss.SZ`
- 2 digits:  `YYYY-MM-DDThh:mm:ss.SSZ`
- 3 digits: `YYYY-MM-DDThh:mm:ss.SSSZ`

We encourage to use a standard library to convert the timestamp received from Jumio as the timeformat is valid with and without the SSS milliseconds. <br/><br>

### Sample Callback

#### Sample callback (URL-encoded POST): Passed

```
scanSource=SDK&transactionReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&enrollmentTransactionReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&callBackType=NETVERIFY_AUTHENTICATION&livenessImages=%5B%22https%3A%2F%2Fnetverify.com%2Fapi%2Fnetverify%2Fv2%2Fauthentications%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fimages%2Fliveness%2F1%22%2C%22https%3A%2F%2Fnetverify.com%2Fapi%2Fnetverify%2Fv2%2Fauthentications%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fimages%2Fliveness%2F2%22%2C%22https%3A%2F%2Fnetverify.com%2Fapi%2Fnetverify%2Fv2%2Fauthentications%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fimages%2Fliveness%2F3%22%2C%22https%3A%2F%2Fnetverify.com%2Fapi%2Fnetverify%2Fv2%2Fauthentications%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fimages%2Fliveness%2F4%22%2C%22https%3A%2F%2Fnetverify.com%2Fapi%2Fnetverify%2Fv2%2Fauthentications%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fimages%2Fliveness%2F5%22%2C%22https%3A%2F%2Fnetverify.com%2Fapi%2Fnetverify%2Fv2%2Fauthentications%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fimages%2Fliveness%2F6%22%2C%22https%3A%2F%2Fnetverify.com%2Fapi%2Fnetverify%2Fv2%2Fauthentications%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fimages%2Fliveness%2F7%22%5D&idScanImageFace=https%3A%2F%2Fnetverify.com%2Fapi%2Fnetverify%2Fv2%2Fauthentications%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fimages%2Fface&callbackDate=2019-05-30T08%3A37%3A35.822Z&transactionDate=2019-05-29T08%3A37%3A24.344Z&transactionResult=PASSED
```

---

## Callback for Document Verification
(formerly Netverify Multi Document)

### Parameters

The following parameters are posted to your callback URL for Document Verification and Document Verification API.

**Required items appear in bold type.**  

|Parameter       | Type    | Max. Length|  Description|
|:---------------|:--------|:----------: |:------------|
|**scanReference**| String  |36 |Jumio's reference number for each transaction|
|**timestamp**| String  |  |Timestamp (UTC) of the response <br>format: [YYYY-MM-DDThh:mm:ss.SSSZ](#timestamp-format-2)|
|**transaction**| JSON object  |  |Transaction related data, see table below|
|document   | JSON object  |       |Document related data if transaction status = DONE, see table |

### Transaction

**Required items appear in bold type.**  

|Parameter `transaction`       | Type    | Max. Length|  Description|
|:---------------|:--------|:----------:|:------------|
|**date**    					        | String  |    |Timestamp (UTC) of the transaction creation<br>format: [YYYY-MM-DDThh:mm:ss.SSSZ](#timestamp-format-2)|
|**status**   | String  |    |Possible states:<br>•	DONE<br>•	FAILED (if initialized acquisition is not successfully finalized within 5 minutes after creation/last update)|
|**source**      							| String  |    |Possible values: <br>• DOC\_UPLOAD (Document Verification)<br>• DOC\_API (Document Verification API)<br>• DOC\_SDK (Document Verification Mobile)|
|**merchantScanReference** 	| String  |255 |Your reference for each transaction |
|**customerId**       				| String  |255 |ID of the customer|
|merchantReportingCriteria    | String  |255 |Your reporting criteria for each transaction|
|clientIp       	| String  |100  |IP address of the client if provided for the Document Verification API |

### Document

**Required items appear in bold type.**  

|Parameter `document`      | Type    | Max. Length|  Description|
|:-------------------------|:--------|:----------:|:------------|
|**status**  	| String  |    |Possible states: <br/> ⦁ UPLOADED (default) <sup>1</sup> <br/> ⦁	EXTRACTED if supported document for data extraction provided <sup>2</sup> <br/> ⦁	DISCARDED if no supported document for data extraction provided |
|**country**  | String  |3   |Possible countries: <br/> ⦁ [ISO 3166-1 alpha-3](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code <br/> ⦁	XKX (Kosovo) |
|**type**     | String  |    |Possible types: <br>⦁ CC (Credit card, front and back side)<br/>⦁	BS (Bank statement, front side) <br/>⦁	IC (Insurance card, front side) <br/>⦁	UB (Utility bill, front side) <br/>⦁	CAAP (Cash advance application, front and back side) <br/>⦁	CRC (Corporate resolution certificate, front and back side) <br/>⦁	CCS (Credit card statement, front and back side) <br/>⦁	LAG (Lease agreement, front and back side) <br/>⦁	LOAP (Loan application, front and back side) <br/>⦁	MOAP (Mortgage application, front and back side) <br/>⦁	TR (Tax return, front and back side) <br/>⦁	VT (Vehicle title, front side) <br/>⦁	VC (Voided check, front side) <br/>⦁	STUC (Student card, front side) <br/>⦁	HCC (Health care card, front side) <br/>⦁	CB (Council bill, front side) <br/>⦁	SENC (Seniors card, front side) <br/>⦁	MEDC (Medicare card, front side) <br/>⦁	BC (Birth certificate, front side) <br/>⦁	WWCC (Working with children check, front side) <br/>⦁	SS (Superannuation statement, front side) <br/>⦁	TAC (Trade association card, front side) <br/>⦁	SEL (School enrolment letter, front side) <br/>⦁	PB (Phone bill, front side) <br/>⦁	SSC (Social security card, front side) <br/>⦁	CUSTOM (Custom document type)<br/>⦁	OTHER (Other document type)|
|**images** 	| JSON array  |  |URLs to the images of the transaction (JPEG or PNG) <sup>3</sup> |
|originalDocument |String | | URL to the originally submitted document of the transaction (PDF) if available <sup>3</sup> |
|customDocumentCode | String  |100 |Your custom document code (maintained in your Jumio customer portal) if type = CUSTOM |
|extractedData | JSON object  | |Extracted data if status = EXTRACTED, see [Supported documents for Data Extraction](#supported-documents-for-data-extraction)|

<sup>1</sup> This also applies for document type `CCS` where no masking was needed as no full PAN is displayed.<br>
<sup>2</sup> If masking has been done, status will be always EXTRACTED as well. This applies to all documents uploaded with type `CC`, as well as to documents uploaded with type `CCS` where a ful PAN is displayed.<br>
<sup>3</sup> Retrieve the images of the transaction.

#### Retrieving Images
Use HTTP **GET** with **Basic Authorization** using your API token and secret, as userid and password.<br>
**Header**: The following parameters are mandatory in the header section of your request.<br>
- `Accept: image/jpeg, image/png`<br>
- `User-Agent: YOURCOMPANYNAME YOURAPPLICATIONNAME/VERSION`<br /><br />
The value for **User-Agent** must contain a reference to your business or entity for Jumio to be able to identify your requests. (e.g. YourCompanyName YourAppName/1.0.0). Without a proper User-Agent header, Jumio will take longer to diagnose API issues.<br>

The TLS protocol is required during the TLS handshake (see [Supported cipher suites](/netverify/supported-cipher-suites.md)) and we strongly recommend using the latest version.

#### Timestamp format

Timestamp are sent in format: `YYYY-MM-DDThh:mm:ss.SSSZ` with constraint that SSS (milliseconds) can either be:

- Not included: `YYYY-MM-DDThh:mm:ssZ`
- 1 digit: `YYYY-MM-DDThh:mm:ss.SZ`
- 2 digits:  `YYYY-MM-DDThh:mm:ss.SSZ`
- 3 digits: `YYYY-MM-DDThh:mm:ss.SSSZ`

We encourage to use a standard library to convert the timestamp received from Jumio as the timeformat is valid with and without the SSS milliseconds. <br/><br>

### Extracted Data

|Parameter `extractedData`      | Type    | Max. Length|  Description|
|:---------------|:--------|:----------:|:------------|
|firstName  | String  |255    |First name if readable|
|lastName |String |255 |Last name if readable|
|name |String | 100 | Full name if readable|
|accountNumber |String |34|Bank account number of the customer from a bank statement|
|pan |String |20 |Personal account number of credit card|
|issueDate  | String  |  |Issue date in the format YYYY-MM-DD|
|expiryDate |String | |Date of expiry in the format YYYY-MM-DD|
|ssn   | String  |255 |Social security number if readable|
|signatureAvailable  | String  |   |`true` if signature available, otherwise `false`|
|swiftCode	| String  | 20  | BIC/SWIFT code |
|address	| JSON object  |  | Address as JSON object in raw format if status = EXTRACTED, see table below |

### Raw address format

|Parameter `address`      | Max. Length|  Description|
|:---------------|:----------:|:------------|
|line1 |100 |Line item 1 |
|line2 |100 |Line item 2 |
|line3 |100 |Line item 3 |
|line4 |100 |Line item 4 |
|line5 |100 |Line item 5 |
|countryCode |3|Possible countries of residence: <br />• [ISO 3166-1 alpha-3](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) country code<br />• XKX (Kosovo)|
|postalCode |15 |Postal code|
|city |64 |City |
|subdivision |50 |Name of subdivision |

### Supported documents for data extraction

<!-- removed with R151
|Country      | Type | Extracted data |
|:------------|:-----|:---------------|
|AUS |UB |name, issueDate, address, dueDate |
|AUS |BS |name, issueDate, address, accountNumber |
|AUS |CCS |name, issueDate, address, cardNumberLastFourDigits |
|CAN |UB |name, issueDate, address, dueDate |
|CAN |BS |name, issueDate, address, accountNumber |
|CAN |CCS |name, issueDate, address, cardNumberLastFourDigits |
|FRA |BS |name, issueDate, address, accountNumber |
|GBR |UB |name, issueDate, address, dueDate |
|GBR |BS |name, issueDate, address, accountNumber |
|GBR |CCS |name, issueDate, address, cardNumberLastFourDigits |
|USA |UB |name, issueDate, address, dueDate |
|USA |BS |name, issueDate, address, accountNumber |
|USA |CCS |name, issueDate, address, cardNumberLastFourDigits |
|USA |SSC |firstName, lastName, ssn, signatureAvailable  |
|all |CC |name, pan, expiryDate |

-->


**Name**, **Address**, and **Issuing Date** will be extracted for all documents printed in a Latin-script character set, provided that a minimum of two of these three data points are available for extraction. If the document does not meet these extraction criteria, only the document image will be saved — no data extraction will be performed.

For the following specific document types, additional data will be extracted.

|Type | Extracted data |
|:---------------|:----------|
|BS (bank statement) |name, issueDate, address, accountNumber, swiftCode |
|CC (credit card) |name, pan, expiryDate (currently returned as issueDate)|
|UB (utility bill) |name, issueDate, address, dueDate |
|CCS (credit card statement) |name, issueDate, address, cardNumberLastFourDigits |
|SSC (Social Security card) <sup>1</sup> |firstName, lastName, ssn, signatureAvailable  |

<sup>1</sup> USA only.



### Sample Callbacks

#### Sample callback (URL-encoded POST): UPLOADED
```
timestamp=2017-06-06T12%3A06%3A49.016Z&scanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&document=%7B%22type%22%3A%22SSC%22%2C%22country%22%3A%22AUT%22%2C%22images%22%3A%5B%22https%3A%2F%2Fretrieval.netverify.com%2Fapi%2Fnetverify%2Fv2%2Fdocuments%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fpages%2F1%22%5D%2C%22status%22%3A%22UPLOADED%22%7D&transaction=%7B%22customerId%22%3A%22CUSTOMERID%22%2C%22date%22%3A%222014-10-17T06%3A37%3A51.969Z%22%2C%22merchantScanReference%22%3A%22YOURSCANREFERENCE%22%2C%22source%22%3A%22DOC_SDK%22%2C%22status%22%3A%22DONE%22%7D
```

#### Sample callback (URL-encoded POST): EXTRACTED
```
timestamp=2017-06-06T12%3A06%3A49.016Z&scanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&document=%7B%22type%22%3A%22SSC%22%2C%22country%22%3A%22USA%22%2C%22extractedData%22%3A%7B%22firstName%22%3A%22FIRSTNAME%22%2C%22lastName%22%3A%22LASTNAME%22%2C%22signatureAvailable%22%3Atrue%2C%22ssn%22%3A%22xxxxxxxxx%22%7D%2C%22images%22%3A%5B%22https%3A%2F%2Fretrieval.netverify.com%2Fapi%2Fnetverify%2Fv2%2Fdocuments%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fpages%2F1%22%5D%2C%22status%22%3A%22EXTRACTED%22%7D&transaction=%7B%22customerId%22%3A%22CUSTOMERID%22%2C%22date%22%3A%222014-10-17T06%3A37%3A51.969Z%22%2C%22merchantScanReference%22%3A%22YOURSCANREFERENCE%22%2C%22source%22%3A%22DOC_SDK%22%2C%22status%22%3A%22DONE%22%7D
```

#### Sample callback (URL-encoded POST): DISCARDED
```
timestamp=2017-06-06T12%3A06%3A49.016Z&scanReference=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx&document=%7B%22type%22%3A%22SSC%22%2C%22country%22%3A%22USA%22%2C%22images%22%3A%5B%22https%3A%2F%2Fretrieval.netverify.com%2Fapi%2Fnetverify%2Fv2%2Fdocuments%2Fxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%2Fpages%2F1%22%5D%2C%22status%22%3A%22DISCARDED%22%7D&transaction=%7B%22customerId%22%3A%22CUSTOMERID%22%2C%22date%22%3A%222014-10-17T06%3A37%3A51.969Z%22%2C%22merchantScanReference%22%3A%22YOURSCANREFERENCE%22%2C%22source%22%3A%22DOC_SDK%22%2C%22status%22%3A%22DONE%22%7D
```

---
## Netverify Retrieval API
If your server was not able to receive or process the callback, you can use the [Retrieval API](/netverify/netverify-retrieval-api.md) to retrieve the results of your transaction.


---
&copy; Jumio Corporation, 395 Page Mill Road, Suite 150 Palo Alto, CA 94306
