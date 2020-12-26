# Sample_Webhook
## Overview
To get started working on the business logic for the biz tracking app we'll setup a few things for testing. The workflow will be. 
1. Listen for payment webhook.
1. Take id and realmid from payment webhook and submit a payment api query.
1. Save data from api query.

## Onboard
These 2 apps show how authentication and the api are implemented. 
- [Sample Node Oauth app](https://github.com/IntuitDeveloper/OAuth2.0-demo-nodejs) will authenticate to api and perform a request to the [get companyInfo](https://developer.intuit.com/app/developer/qbo/docs/api/accounting/most-commonly-used/companyinfo) endpoint.
- [Sample Node Webhook notification app](https://github.com/IntuitDeveloper/SampleApp-WebhookNotifications-nodejs) will authenticate and listen for events passed from a webhook url.

## Workflow
The webhook notification passes labels of the payment operation as opposed to its details. To get those details we'll perform a follow on request.

The webhook notification:

```JSON
{
   "eventNotifications":[
      {
         "realmId":"4620816365157277890",
         "dataChangeEvent":{
            "entities":[
               {
                  "name":"Payment",
                  "id":"149",
                  "operation":"Create",
                  "lastUpdated":"2020-12-26T17:56:53.000Z"
               }
            ]
         }
      }
   ]
}
```

With this data we can query the payments api for this event.
The query:

```
GET /v3/company/4620816365157277890/query?query=select * from Payment Where Id='149'&minorversion=55

Content type:application/text
Production Base URL:https://quickbooks.api.intuit.com
Sandbox Base URL:https://sandbox-quickbooks.api.intuit.com
```

The response:

```JSON
{
 "QueryResponse": {
  "Payment": [
   {
    "CustomerRef": {
     "value": "63",
     "name": "fsdafas"
    },
    "DepositToAccountRef": {
     "value": "4"
    },
    "PaymentMethodRef": {
     "value": "1"
    },
    "TotalAmt": 5000,
    "UnappliedAmt": 5000,
    "ProcessPayment": false,
    "domain": "QBO",
    "sparse": false,
    "Id": "149",
    "SyncToken": "0",
    "MetaData": {
     "CreateTime": "2020-12-26T09:56:53-08:00",
     "LastUpdatedTime": "2020-12-26T09:56:53-08:00"
    },
    "TxnDate": "2020-12-26",
    "CurrencyRef": {
     "value": "USD",
     "name": "United States Dollar"
    },
    "Line": []
   }
  ],
  "startPosition": 1,
  "maxResults": 1
 },
 "time": "2020-12-26T10:41:11.184-08:00"
}
```

## Ambiguity
- Do we save TotalAmt/UnappliedAmt?
- How will payments for employees vs. contractors be standardized?
