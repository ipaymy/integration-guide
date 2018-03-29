# Integration Guide via OAuth 2

## Authorization

1. First, the user from your app will open a link to `https://partner-dev-fe.ipaymy.com/app/authorize?client_id=A6F8A4F7-F9EA-459A-B219-0BB215D826C9&response_type=code&state=[your-state-id-here]`. A simple way to implement this is via an anchor element:
```html
<a href="https://partner-dev-fe.ipaymy.com/app/authorize?client_id=A6F8A4F7-F9EA-459A-B219-0BB215D826C9&response_type=code&state=aGSZssdxSAdGHZH">Connect with iPaymy</a>
```

2. On this page, the user will be asked to log in to their iPaymy account, or create one. It will then ask the user to authorize your app to create payment requests on their behalf. The user will also select the company to associate with your app.
3. If successful, the user will be redirected back to your `CALLBACK_URI` of choice.
```url
https://app.com/callback?
code="<YOUR_AUTH_CODE>"
&state=aGSZssdxSAdGHZH
&company_id=6BB09700-3110-11E8-BA5E-E908EC24792D
```
There are 3 query parameters:
1. `code`: exchange this auth code for a `access_token`, which will allow you to make API calls.
2. `state`: verify that this `state` is the same `state` which you passed earlier on in the step 1.
3. `company_id`: use this `company_id` when making a payment request later on.

## Token Exchange

1. Make a `POST` request to `https://partner-api.ipaymy.com/api/v1.1/oauth/token` with json body:
```json
{
  "grant_type": "authorization_code",
  "client_id": "<YOUR_CLIENT_ID>",
  "client_secret": "<YOUR_CLIENT_SECRET>",
  "code": "<YOUR_AUTH_CODE>"
}
```
If successful, you should receive a json response:
```json
{
  "token_type": "Bearer",
  "expires_in": 1209600,
  "access_token": "<YOUR_ACCESS_TOKEN>",
  "refresh_token": "<YOUR_REFRESH_TOKEN>"
}
```


## Making Payment Requests

1. Make a `POST` request to `https://partner-api.ipaymy.com/api/v1.1/companies/[company-id]/payment_requests` with header `Authorization: Bearer <YOUR_ACCESS_TOKEN>`, and json body:
```json
{
  "idempotency_key": "7AB09700-3110-11E8-BA5E-E908EC24792D",
  "payment_type": "payroll",
  "payees": [
    {
      "amount": 2000.40,
      "currency": "SGD",
      "recipient_name": "Andrew Goh",
      "comments": "SALARY FOR JULY 2018",
      "account_number": "1231123901",
      "bank_name": "dbs"
    }
  ]
}
```
- the `idempotency_key` is a uuid string generated on your end so that you can retry the `POST` request with the same idempotency key, should it fail.

2. If it is successful, you will receive a response that includes a URL to redirect the user to in order to complete their payment.
```json
{
  "error": "0",
  "message": {
    "id": "efac849b-defc-4f54-b9f6-7a2e7f005098",
    "redirect_url": "https://partner-dev-fe.ipaymy.com/payments/efac849b-defc-4f54-b9f6-7a2e7f005098",
  }
}
```
On this page, the user will key in their credit card details securely, and iPaymy will calculate the license fee applicable for the credit card which the user keys in.


# Useful Resources
- [Authorization Code Request](https://www.oauth.com/oauth2-servers/access-tokens/authorization-code-request/)
