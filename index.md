## Virtual Sales Data Controller (V-SDC) API

## Configuration & Requirements
- Is mandatory to use valid certificates during communications, server application will validate client certificate against Certificate Authority(CA) during SSL handshake.
- You will need your merchant/taxpayer account TIN number, please contact system admin to obtain one valid cert. You should use it as param when it's necessary (issuing invoices).
- This endpoint is used to issue receipts to system, server will validate it against aTAX, sign it and send to aTAX server. This endpoint accepts JSON/XML format as payload.

## Invoice- Fiscalize invoice
Regular invoice format defines minimum data that has to be signed and submitted to the Tax Authority. Invoice consist of following data.

### Params

Field | Type | Description
------------ | ------------ | -------------
tin | String | Tax payer id Size range: ..15
date | Date | Date and Time of Issue (ISO 8601)
place **optional** | String | Place of issue Size range: ..15
bid **optional** | String | Buyer Id
ccid **optional** | String | Provided by buyer for automatic expense/budget tracking Size range: ..15
itype | String | Invoice Type Allowed values: "Normal", "Pro Forma", "Training"
ttype | String | Transaction Type Allowed values: "Sale", "Refund"
mcr | String | Device Id Size range: ..15
ino | String | Invoice Number Size range: ..15
journal | String | The text to be printed after a successful transaction. It may contain some placeholders to be replaced with the relevant information by the VSDC server. [Placeholder reference](#placeholders)
phone **optional** | String | Use this filed to send conformation SMS to buyer
hr | BigDecimal | High VAT Rate
hb | BigDecimal | High Taxable Amount
ht | BigDecimal | High VAT Amount
mr | BigDecimal | Mid VAT Rate
mb | BigDecimal | Mid Taxable Amount
mt | BigDecimal | Mid VAT Amount
lr | BigDecimal | Low VAT Rate
lb | BigDecimal | Low Taxable Amount
lt | BigDecimal | Low VAT Amount
zr | BigDecimal | Zero VAT Rate
zb | BigDecimal | Zero Taxable Amount
zt | BigDecimal | Zero VAT Amount
ct **optional** | BigDecimal | Consumption tax
cr **optional** | BigDecimal | Consumption rate
lines **optional** | Object[] | An array with all the invoice lines included in the transaction


### Extended lines
Add to lines param

Field | Type | Description
------------ | ------------ | -------------
ean | String | The EAN code for this item Size range: 8..18
name | String | The item description Size range: ..60
quantity | BigDecimal | Number of units included in this line
base | BigDecimal | Taxable amount per item
discount | BigDecimal | Applied discount per item
code | String | Tax code used for this line
total | BigDecimal | Total amount of the line


<a name="placeholders"></a>### JOURNAL placeholders
```
$$date$$: V-SDC Date And Time 
$$nev$$: V-SDC Number 
$$numberRecu$$: Number and Type of Invoice 
$idata$$: Internal Data 
$signature$$: Digital Signature
```

### Request example
```
curl 'https://HOST' \	
	-X POST \
	-H 'Content-type: application/json' \
	--data 'JSON_ENCODED_INVOICE' \
	-v \
	--cert SERIAL_NUMBER.PEM:PASS \
	--cacert CA.PEM
 ```
Where:
- JSON_ENCODED_INVOICE
```
{
	"tin": "123456789",
	"date": "2014-12-31 21:07:36 -0400",
	"place": "xxx",
	"bid": "000000000000001",
	"ccid": "000000000000001",
	"itype": "Training",
	"ttype": "Sale",
	"mcr": "777777777",
	"ino": "xxxxxxxxxxxxxxx",
	"hr": 0,
	"hb": 1000,
	"ht": 0,
	"mr": 18,
	"mb": 2000,
	"mt": 360,
	"lr": 16,
	"lb": 3000,
	"lt": 480,
	"zr": 32,
	"zb": 4000,
	"zt": 1280,
	"journal": "..."
}
```
- SERIAL_NUMBER.PEM, certificate in PEM format
- PASS, pass phrase used when you extract certificate 
- CA.PEM, certificate authority in PEM format

### Responses
**HTTP/1.1 200 OK**

Field | Type | Description
------------ | ------------ | -------------
success | Boolean | Whether the request was successfully processed or not
status | String | 
message | String | Any relevant information regarding the received invoice
errors | String | Array of errors found processing ticket
hash | String | MD5 string representing ticket data (Cash Register ID, Invoice No, Date & hour)
journal | String | The journal text updated with newly genezr data
vsdcNumber | Long | This virtual device's serial number for reference
signingDate | String | V-SDC date and time of signing
internalData | String | Hash of internal data (aka counters)
signature | String | Signature of the invoice
invoiceNumber | String | Number of Invoice
invoiceType | String | Type of Invoice
verificationUrl | String | URL used to validate ticket against aTax. This field should be used to print QR code
totalInvoices | Integer | Total number of invoices signed by the V-SDC tenant

```
{
     "success": true,
     "status": null,
     "message": "Request successfully processed",
     "errors": [],
     "hash": "OaYvSLBCq8Y/...",
     "journal": "",
     "vsdcNumber": "MCR_...",
     "signingDate": "2015-06-29 06:56:27 -0400",
     "internalData": "+A3xaCNiHjY...",
     "signature": "OC0HbiGJok...",
     "invoiceNumber": "xxxyyy001",
     "invoiceType": "Normal",
     "verificationUrl": "https://...",
     "totalInvoices": 181
}
```

**HTTP/1.1 400 Bad Request**

Field | Type | Description
------------ | ------------ | -------------
success | Boolean | Whether the request was successfully processed or not
message | String | Any relevant information regarding the received invoice
errors | String[] | A list of the errors found while processing the received invoice

Wrong tin
```
{
  "success":false,
  "status":null,
  "message":"TIN <XYZ> does not exist.",
  "errors":[]
}
```

Wrong mrc
```
{
    "success": false,
    "message": null,
    "errors": ["MCR <000000000000001> does not exist."]
}
```

Wrong tax
```
{
  "success":false,
  "status":null,
  "message":"Tax Rate expected <0.000000> but was <25.000000>",
  "errors":[] 
}
```

Journal is missing
```
{
  "success":false,
  "status":null,
  "message":"Tax Rate expected <0.000000> but was <25.000000>",
  "errors":[] 
}
```

Tin/Mrc pair is not setup properly
```
{
     "success":false,
     "status":null,
     "message":"There was an error with your MCR/TIN  (777777777/123456789) combination. Please try again.",
     "errors":["There was an error with your MCR/TIN  (777777777/123456789) combination. Please try again."]
}
```

**HTTP/1.1 401 Unauthorized**

The request has not been applied because it lacks valid authentication credentials for the target resource. Maybe client certificate is missing, invalid o revoked.

