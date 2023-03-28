# Fonbnk Top Up

Quickly and easily add airtime credits to your customer's phone with our convenient top-up API.

# The Dashboard
Our web dashboard provides an easy way to stay on top of the activity in your account. It is personalized for your account, and all top-up requests you create using the API will appear in this dashboard.

### Dashboard URLs:
| Environment | URL |
|---------|------------|
| Sandbox | https://sandbox.top-up.fonbnk.com |
| Production | https://top-up.fonbnk.com|

# Sandbox Testing
When you interact with Sandbox version of the API, you have the same experience as you would in production, but you won’t get charged real tokens and won’t receive real top-ups just yet.


# API Details
## Overview
The API is organized around REST. The API accepts json-encoded request bodies, returns JSON-encoded responses, and uses standard HTTP response codes and verbs.

## API servers 
| Environment | Server URL <SERVER_URL> |
|---------|------------|
| Sandbox | https://dev-aten.fonbnk-services.com |
| Production | https://aten.fonbnk-services.com|


## Request Authentication
All requests should be signed using a HMAC256 algorithm and provided `clientId` and `clientSecret`.

## How to get the signature of the request?
1. Generate a timestamp (Epoch Unix Timestamp)
2. Convert the request data to a JSON string and get the MD5 hash of it
3. Convert the body MD5 hash to base64
4. Concatenate the converted body MD5 hash, timestamp and the endpoint that is called
   `{bodyMD5Hash}:{timestamp}:{endpoint}`
5. Decode the base64 encoded clientSecret
6. Compute the SHA256 hash of the concatenated string. Use decoded clientSecret as a key. Convert the result to base64
7. Add the clientId, signature, and timestamp to HTTP headers

The following pseudocode example demonstrates and explains how to sign a request
```
bodyMD5Hash = Base64 ( MD5 ( UTF8 ( [BODY] ) ) );
timestamp = CurrentTimestamp();
stringToSign = bodyMD5Hash + ":" + timestamp + ":" + endpoint;
signature = Base64 ( HMAC-SHA256 ( Base64-Decode ( clientSecret ), UTF8 ( concatenatedString ) ) );
```
Typescript example
```typescript
import crypto from 'crypto';
import utf8 from 'utf8';

const generateSignature = ({
  clientSecret,
  requestData,
  timestamp,
  endpoint,
}: {
  clientSecret: string;
  requestData?: object; //empty for GET request
  timestamp: string;
  endpoint: string;
}) => {
  let body = '';
  if (requestData) {
    body = JSON.stringify(requestData);
  }
  const hash = crypto.createHash('md5');
  let hmac = crypto.createHmac('sha256', Buffer.from(clientSecret, 'base64'));
  let contentMD5 = hash.update(utf8.encode(body)).digest('base64');
  let stringToSign = `${contentMD5}:${timestamp}:${endpoint}`;
  hmac.update(stringToSign);
  return hmac.digest('base64');
};

```
## Each request should include the following headers

Header     | Description                                           |Example
-----------|-------------------------------------------------------|-------
x-client-id| `clientId` provided to you                              |vXVMhQlr5+sq4cPdCD5b4W0T=
x-timestamp| A UNIX timestamp you generate before sending a request to us. Please generate this timestamp right before sending a request to us and re-generate it for every request. |1663240633
x-signature| Computed signature using `clientSecret` provided to you. |Y90dweZduRFNEF8MsmEUExBg8b8ha=

## API Methods

### Create top-up request
Create a request to top-up a specified phone number.  
This endpoint reduces your account's balance and initiates a top-up.
Additionally, you can pass the carrier name to top-up a specific carrier.
If you don't pass the carrier name, we will automatically detect the carrier and top-up the phone number, which may
take a few seconds to complete.
Furthermore, you can pass the strategy parameter to specify the strategy for fulfilling the top-up request. The default strategy is `best_price`.
Available strategies are:

Strategy   | Description
-----------|-------------------------------------------------------
best_price| At first, we try to fulfill the request using the market price (p2p). If we can't, we will fulfill the request using the wholesale price.                               
wholesale| We fulfill the request only using the wholesale price.
market| We fulfill the request only using the market price (p2p).

#### Request
```
Request URL
[POST] <SERVER_URL>/api/v1/top-up/create-request

Request body
{
    airtimeAmount: 100,
    recipientPhoneNumber: "254XXXXXXXXX",
    carrierName: "Safaricom Kenya", //optional
    strategy: "best_price", //optional
}

Endpoint for signature
/api/v1/top-up/create-request

Request Headers 
x-client-id: vXVMhQlr5+sq4cPdCD5b4W0T6wM53nDGraxtadiavbg= 
x-timestamp: 1663240633
x-signature: Y90dweZduRFNEF8MsmEUExBg8b8ha5SLYHz5uoYO8wA= 
```

#### Response
A successful request will return the following JSON encoded response

```javascript
{
    requestId: "Y90dweZduRFNEF8Msm",
    status: "pending",
    airtimeAmount: 100,
    carrier: "Safaricom Kenya",
    recipientPhoneNumber: "XXXXXXXXXXXX",
    date: "2022-09-16T07:05:46.126Z"
}
```

### List of available carriers
Returns a list of available carriers

#### Request
```
Request URL
[GET] <SERVER_URL>/api/v1/top-up/carriers

Endpoint for signature
/api/v1/top-up/carriers

Request Headers 
x-client-id: vXVMhQlr5+sq4cPdCD5b4W0T6wM53nDGraxtadiavbg= 
x-timestamp: 1663240633
x-signature: Y90dweZduRFNEF8MsmEUExBg8b8ha5SLYHz5uoYO8wA= 
```

#### Response
A successful request will return the following JSON encoded response

```javascript
[
    "MTN Nigeria",
    "Airtel Nigeria",
    "9mobile Nigeria",
    "Glo Nigeria",
    "Safaricom Kenya",
    "Airtel Kenya",
    "Telkom Kenya",
]
```

### Get top-up request by id
Get the details of a top-up request

#### Request
```
Request URL
[GET] <SERVER_URL>/api/v1/top-up/request/<REQUEST_ID>

Endpoint for signature 
/api/v1/top-up/request/<REQUEST_ID>

Request Headers 
x-client-id: vXVMhQlr5+sq4cPdCD5b4W0T6wM53nDGraxtadiavbg= 
x-timestamp: 1663240633
x-signature: Y90dweZduRFNEF8MsmEUExBg8b8ha5SLYHz5uoYO8wA= 
```

#### Response
A successful request will return the following JSON encoded response

```javascript
{
    requestId: "Y90dweZduRFNEF8Msm",
    usdAmount: 0.83,
    exchangeRate: 120,
    status: "completed",
    airtimeAmount: 100,
    recipientPhoneNumber: "XXXXXXXXXXXX",
    date: "2022-09-16T07:05:46.126Z"
}
```

### Top up request statuses

Status|Description
-------|----
pending|The top-up request was succesfully created 
completed|The recipient got the airtime
failed|The top-up request failed, your account will be refunded

### Balance
Get account balance
#### Request
```
Request URL
[GET] <SERVER_URL>/api/v1/top-up/balance

Endpoint for signature 
/api/v1/top-up/balance

Request Headers 
x-client-id: vXVMhQlr5+sq4cPdCD5b4W0T6wM53nDGraxtadiavbg= 
x-timestamp: 1663240633
x-signature: Y90dweZduRFNEF8MsmEUExBg8b8ha5SLYHz5uoYO8wA= 
```

#### Response
A successful request will return the following JSON encoded response

```javascript
{
    balance: 1000.00
}
```

### Verify top-up request
This endpoint estimates the top-up cost and validates the data you need to provide to create a top-up request.  
This endpoint doesn't reduce your account's balance.
Additionally, you can pass the carrier name to top-up a specific carrier.
If you don't pass the carrier name, we will automatically detect the carrier and top-up the phone number, but it may
take a few seconds to complete.
Furthermore, you can pass the strategy parameter to specify the price estimation strategy. The default strategy is `best_price`.
Available strategies are: `best_price`, `wholesale`, `market`.

#### Request
```
Request URL
[POST] <SERVER_URL>/api/v1/top-up/verify-request

Request body
{
    airtimeAmount: 100,
    recipientPhoneNumber: "254XXXXXXXXX",
    carrierName: "Safaricom Kenya", //optional
    strategy: "best_price", //optional
}

Endpoint for signature 
/api/v1/top-up/verify-request

Request Headers 
x-client-id: vXVMhQlr5+sq4cPdCD5b4W0T6wM53nDGraxtadiavbg= 
x-timestamp: 1663240633
x-signature: Y90dweZduRFNEF8MsmEUExBg8b8ha5SLYHz5uoYO8wA= 
```

#### Response
A successful request will return the following JSON encoded response

```javascript
{
    carrier: "Safaricom Kenya",
    usdAmount: 0.83,
    exchangeRate: 120,
    airtimeAmount: 100,
    recipientPhoneNumber: "254XXXXXXXXX"
}
```


