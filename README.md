# Fonbnk Inc. top-up API

## API Structure
The API uses an HTTP/REST architecture. The parameters of an HTTP GET request must be specified in the query string. API responses are encoded in JSON format and use standard HTTP response codes.

## API Security
The API is secured using HMAC256 signatures. API requests are only accepted over HTTPS.

## Request Authentication
API client requests are authenticated and authorized using a supplied clientId and clientSecret. For every request, the API client must include the clientId and sign the request using the clientSecret.

## Computing the Signature
The algorithm used to compute the signature is described in the followings steps
1. Generate a timestamp (Epoch Unix Timestamp)
2. Convert the request body to a JSON string and get the MD5 hash of it
3. Convert the body MD5 hash to base64
4. Concatenate the converted body MD5 hash, timestamp and the endpoint that called
   `{bodyMD5Hash}:{timestamp}:{endpoint}`
5. Decode the base64 encoded clientSecret
6. Compute the SHA256 hash of the concatenated string. Use decoded clientSecret as a key. Convert the result to base64
7. Add the clientId, signature and timestamp to HTTP headers
The following pseudocode example demonstrates and explains how to sign a request
```
bodyMD5Hash = Base64 ( MD5 ( UTF8 ( [BODY] ) ) );
timestamp = CurrentTimestamp();
stringToSign = bodyMD5Hash + ":" + timestamp + ":" + endpoint;
signature = Base64 ( HMAC-SHA256 ( Base64-Decode ( clientSecret ), UTF8 ( concatenatedString ) ) );
```
## Request headers
```
x-client-id: vXVMhQlr5+sq4cPdCD5b4W0T6wM53nDGraxtadiavbg= 
x-timestamp: 1663240633
x-signature: Y90dweZduRFNEF8MsmEUExBg8b8ha5SLYHz5uoYO8wA= 
```
## API Methods
Name|Description
----|-----------
[Get balance](#balance-request)|Get MIN tokens account balance
[Verify top-up request](#verify-top-up-request)|Verify data for creating a request to top-up a specified phone number
[Create top-up request](#create-top-up-request)|Create a request to top-up a specified phone number
[Get top-up request by id](#get-top-up-request-by-id)|Get the details of an top-up request

## Balance request
```
Request URL
[GET] https://dev-aten.fonbnk-services.com/api/v1/top-up/balance

Request Headers 
x-client-id: vXVMhQlr5+sq4cPdCD5b4W0T6wM53nDGraxtadiavbg= 
x-timestamp: 1663240633
x-signature: Y90dweZduRFNEF8MsmEUExBg8b8ha5SLYHz5uoYO8wA= 
```

In this case the endpoint for signature will be `/api/v1/top-up/balance`

## Balance response
A successful request will return the following JSON encoded response

**HTTP 200 OK**
```javascript
{
    balance: 100000
}
```

## Verify top-up request
```
Request URL
[POST] https://dev-aten.fonbnk-services.com/api/v1/top-up/verify-request

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

In this case the endpoint for signature will be `/api/v1/top-up/verify-request`

## Verify top-up request response
A successful request will return the following JSON encoded response

**HTTP 200 OK**
```javascript
{
    carrier: "Safaricom Kenya",
    minCost: 83,
    exchangeRate: 1.2,
    airtimeAmount: 100,
    recipientPhoneNumber: "254XXXXXXXXX"
}
```

## Create top-up request
```
Request URL
[POST] https://dev-aten.fonbnk-services.com/api/v1/top-up/create-request

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

In this case the endpoint for signature will be `/api/v1/top-up/create-request`

## Create top-up request response
A successful request will return the following JSON encoded response

**HTTP 200 OK**
```javascript
{
    requestId: "Y90dweZduRFNEF8Msm",
    carrier: "Safaricom Kenya",
    minCost: 83,
    exchangeRate: 1.2,
    airtimeAmount: 100,
    recipientPhoneNumber: "254XXXXXXXXX"
}
```

## Get top-up request by id
```
Request URL
[GET] https://dev-aten.fonbnk-services.com/api/v1/top-up/request/<REQUEST_ID>

Request Headers 
x-client-id: vXVMhQlr5+sq4cPdCD5b4W0T6wM53nDGraxtadiavbg= 
x-timestamp: 1663240633
x-signature: Y90dweZduRFNEF8MsmEUExBg8b8ha5SLYHz5uoYO8wA= 
```

In this case the endpoint for signature will be `/api/v1/top-up/request/<REQUEST_ID>`

## Get top-up request by id response
A successful request will return the following JSON encoded response

**HTTP 200 OK**
```javascript
{
    requestId: "Y90dweZduRFNEF8Msm",
    airtimeAmount: 100,
    status: "pending",
    recipientPhoneNumber: "254XXXXXXXXX"
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
