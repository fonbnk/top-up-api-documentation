# Fonbnk Top Up

Quickly and easily add airtime credits to your customer's phone with our convenient top-up API.

# The Dashboard
Our web dashboard provides an easy way to stay on top of the activity in your account. It is personalized for your account, and all top-up requests you create using the API will appear in this dashboard.

### Dashboard URLs:
Sandbox: `https://sandbox.top-up.fonbnk.com` [see sandbox testing](https://github.com/fonbnk/top-up-api-documentation#sandbox-testing)

Production: `https://top-up.fonbnk.com`

# Sandbox Testing
When you interact with Sandbox version of the API, you have the same experience as you would in production, but you won’t get charged real tokens and won’t receive real top-ups just yet.


# API 
## Overview
The API is organized around REST. The API accepts json-encoded request bodies, returns JSON-encoded responses, and uses standard HTTP response codes and verbs.

## API servers 
Sandbox: `https://dev-aten.fonbnk-services.com` [see sandbox testing](https://github.com/fonbnk/top-up-api-documentation/edit/main/README.md#sandbox-testing)

Production: `https://aten.fonbnk-services.com`

## Request Authentication
All requests should be signed using a HMAC256 algorithm and provided clientId and clientSecret.

## How to get the signature of the request?
1. Generate a timestamp (Epoch Unix Timestamp)
2. Convert the request body to a JSON string and get the MD5 hash of it
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
  body,
  timestamp,
  endpoint,
}: {
  clientSecret: string;
  body: object;
  timestamp: string;
  endpoint: string;
}) => {
  const hash = crypto.createHash('md5');
  let hmac = crypto.createHmac('sha256', Buffer.from(clientSecret, 'base64'));
  let contentMD5 = hash.update(utf8.encode(JSON.stringify(body))).digest('base64');
  let stringToSign = `${contentMD5}:${timestamp}:${endpoint}`;
  hmac.update(stringToSign);
  return hmac.digest('base64');
};

```
## Each request should include the following headers

Header     | Description                                           |Example
-----------|-------------------------------------------------------|-------
x-client-id| clientId provided to you                              |vXVMhQlr5+sq4cPdCD5b4W0T=
x-timestamp| timestamp of request                                  |1663240633
x-signature| computed signature using clientSecret provided to you |Y90dweZduRFNEF8MsmEUExBg8b8ha=

## API Methods

### Balance
Get MIN tokens account balance
#### Request
```
Request URL
[GET] <SERVER_URL>/api/v1/top-up/balance

Request Headers 
x-client-id: vXVMhQlr5+sq4cPdCD5b4W0T6wM53nDGraxtadiavbg= 
x-timestamp: 1663240633
x-signature: Y90dweZduRFNEF8MsmEUExBg8b8ha5SLYHz5uoYO8wA= 
```

In this case, the endpoint for signature will be `/api/v1/top-up/balance`

#### Response
A successful request will return the following JSON encoded response

```javascript
{
    balance: 100000
}
```

### Verify top-up request
This endpoint estimates the top-up cost and validates the data you need to provide to create a top-up request.  
This endpoint doesn't charge MIN tokens from your balance.

#### Request
```
Request URL
[POST] <SERVER_URL>/api/v1/top-up/verify-request

Request body
{
    airtimeAmount: 100,
    recipientPhoneNumber: "254XXXXXXXXX",
}

Request Headers 
x-client-id: vXVMhQlr5+sq4cPdCD5b4W0T6wM53nDGraxtadiavbg= 
x-timestamp: 1663240633
x-signature: Y90dweZduRFNEF8MsmEUExBg8b8ha5SLYHz5uoYO8wA= 
```

In this case, the endpoint for signature will be `/api/v1/top-up/verify-request`

#### Response
A successful request will return the following JSON encoded response

```javascript
{
    carrier: "Safaricom Kenya",
    minTokenAmount: 83,
    minTokenToAirtimeRate: 1.2,
    airtimeAmount: 100,
    recipientPhoneNumber: "254XXXXXXXXX"
}
```

### Create top-up request
Create a request to top-up a specified phone number 
This endpoint charges MIN tokens from your balance.

#### Request
```
Request URL
[POST] <SERVER_URL>/api/v1/top-up/create-request

Request body
{
    airtimeAmount: 100,
    recipientPhoneNumber: "254XXXXXXXXX",
}

Request Headers 
x-client-id: vXVMhQlr5+sq4cPdCD5b4W0T6wM53nDGraxtadiavbg= 
x-timestamp: 1663240633
x-signature: Y90dweZduRFNEF8MsmEUExBg8b8ha5SLYHz5uoYO8wA= 
```

In this case, the endpoint for signature will be `/api/v1/top-up/create-request`

#### Response
A successful request will return the following JSON encoded response

```javascript
{
    requestId: "Y90dweZduRFNEF8Msm",
    carrier: "Safaricom Kenya",
    minTokenAmount: 83,
    minTokenToAirtimeRate: 1.2,
    airtimeAmount: 100,
    recipientPhoneNumber: "254XXXXXXXXX",
    date: "2022-09-16T07:05:46.126Z"
}
```

### Get top-up request by id
Get the details of a top-up request

#### Request
```
Request URL
[GET] <SERVER_URL>/api/v1/top-up/request/<REQUEST_ID>

Request Headers 
x-client-id: vXVMhQlr5+sq4cPdCD5b4W0T6wM53nDGraxtadiavbg= 
x-timestamp: 1663240633
x-signature: Y90dweZduRFNEF8MsmEUExBg8b8ha5SLYHz5uoYO8wA= 
```

In this case the endpoint for signature will be `/api/v1/top-up/request/<REQUEST_ID>`

#### Response
A successful request will return the following JSON encoded response

```javascript
{
    requestId: "Y90dweZduRFNEF8Msm",
    minTokenAmount: 83,
    minTokenToAirtimeRate: 1.2,
    status: "pending",
    airtimeAmount: 100,
    recipientPhoneNumber: "254XXXXXXXXX",
    date: "2022-09-16T07:05:46.126Z"
}
```

## Top up request statuses

Status|Description
-------|----
pending|The top-up request was succesfully created 
acknowledged|The top-up request has been sent and will be executed now
executed|The top-up request was executed
completed|The recipient got the airtime
failed|The top-up request failed, MIN tokens will be refunded
