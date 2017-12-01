---
layout: post
title: Validating and Registering an Azure Event Grid WebHook in Node.js
date: "2017-12-01"
categories:
  - event-grid
  - azure
  - nodejs
---

It is possible to register your own webhook endpoint with [Azure Event Grid](https://docs.microsoft.com/en-us/azure/event-grid/).  To do so you need to pass the [Event Grid Validation process](https://docs.microsoft.com/en-us/azure/event-grid/security-authentication) which happens when you first subscribe your endpoint to a Event Grid Topic.  

At subscription time, Event Grid will make a HTTP `POST` request to you endpoint with a header value of `Aeg-Event-Type: SubscriptionValidation`.  Inside the request there will be a validation code that you need to echo back to the service.  A sample request will look like:

```json
[{
  "id": "2d1781af-3a4c-4d7c-bd0c-e34b19da4e66",
  "topic": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "subject": "",
  "data": {
    "validationCode": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6"
  },
  "eventType": "Microsoft.EventGrid.SubscriptionValidationEvent",
  "eventTime": "2017-08-06T22:09:30.740323Z"
}]
```

And the response expected is:

```json
{
  "validationResponse": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6"
}
```

You can read about all the details of [Event Grid security and authentication](https://docs.microsoft.com/en-us/azure/event-grid/security-authentication). 

> Note: The endpoint must be `https`.  To debug your function locally, you can use `ngrok` as described in this post on [Locally debugging an Azure Function Triggered by Event Grid](https://blogs.msdn.microsoft.com/brandonh/2017/11/30/locally-debugging-an-azure-function-triggered-by-azure-event-grid/).  The general concept of using `ngrok` can be used even though we are not using Functions.

## Handling the Response in Node.js
Handling the response in Node.js is fairly straight forward by checking the header type, event type and then returning the 200 status with the validation body.  Here is an example in Express.js

```javascript
app.post('/event', (req, res) => {
    var header = req.get("Aeg-Event-Type");
    if(header && header === 'SubscriptionValidation'){
         var event = req.body[0]
         var isValidationEvent = event && event.data && 
                                 event.data.validationCode &&
                                 event.eventType && event.eventType == 'Microsoft.EventGrid.SubscriptionValidationEvent'
         if(isValidationEvent){
             return res.send({
                "validationResponse": event.data.validationCode
            })
         }
    }

    // Do something on other event types 
    console.log(req.body)
    res.send(req.body)
})
```

## Testing it out
Create a topic:

```bash
az group create --name eventgridrg
az eventgrid topic create --name nodejs -l westus2 -g eventgridrg
```

Set up ngrok in a separate terminal (optionally tag on --log "stdout" --log-level "debug" if running [ngrok from WSL](https://github.com/Microsoft/WSL/issues/1951))

```bash
./ngrok http 3000 #optional --log "stdout" --log-level "debug"
```

Register your ngrok https endpoint with Event Grid:

```bash
az eventgrid topic event-subscription create --name expressapp \
          --endpoint https://994a01e1.ngrok.io/event \
          -g eventgridrg \
          --topic-name nodejs
```

The registration should succeed with `"provisioningState": "Succeeded"' in the response because it has the validation code.  Once it is finished registering send a request and get the response:

```bash
# get endpoint and key
endpoint=$(az eventgrid topic show --name nodejs -g eventgridrg --query "endpoint" --output tsv)
key=$(az eventgrid topic key list --name nodejs -g eventgridrg --query "key1" --output tsv)

# use the example event from docs 
body=$(eval echo "'$(curl https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/event-grid/customevent.json)'")

# post the event
curl -X POST -H "aeg-sas-key: $key" -d "$body" $endpoint
```

In your terminal where the app is running you should see the log output of the custom event:

```bash
node index.js

#output
Example app listening on port 3000!
[ { id: '10107',
    eventType: 'recordInserted',
    subject: 'myapp/vehicles/motorcycles',
    eventTime: '2017-12-01T20:28:59+00:00',
    data: { make: 'Ducati', model: 'Monster' },
    topic: '/SUBSCRIPTIONS/B9D9436A-0C07-4FE8-B779-xxxxxxxxxxx/RESOURCEGROUPS/EVENTGRIDRG/PROVIDERS/MICROSOFT.EVENTGRID/TOPICS/NODEJS' } ]
```
