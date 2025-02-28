---
title: Send sms notifications
linktitle: Send sms notifications
description: Endpoint for sending an SMS notification to one or more recipient.
weight: 50
toc: true
---

## Endpoint

POST /orders/sms

## Authentication

This API requires authentication and the request must also include one of the following:

- Maskinporten scope __altinn:serviceowner/notifications.create__ (for external system callers)
- Platform Access Token (for Altinn Apps and internal Altinn systems)

See [Authentication and Authorization](/notifications/reference/api/#authentication--authorization) for more information.

## Request

### Content-type

application/json


### Request body
The request body must contain the order request formatted as an
[SmsNotificationOrderRequestExt](https://github.com/Altinn/altinn-notifications/blob/main/src/Altinn.Notifications/Models/SmsNotificationOrderRequestExt.cs)
and serialized as JSON.


### Required order request properties

#### __body__
Type: _string_

The contents of the SMS.

#### recipients
Type: _List of [RecipientExt](https://github.com/Altinn/altinn-notifications/blob/main/src/Altinn.Notifications/Models/RecipientExt.cs)_

A list containing one or more recipient objects, each recipient containing either a mobile number,
a national identity number or an organization number.

### Optional order request properties
#### __senderNumber__

Type: _string_

Default: _Altinn_
The string to use as the sender of the SMS.

#### requestedSendTime
Type: _DateTime_

Default: Current time

The date and time (with time zone specification) when the notification should be sent to recipient.

#### sendersReference
Type: _string_

An internal reference for notification creator to lookup or identify the notification in
the future. Could be a case number or another id. It is recommended, but not required,
that the sender's reference is unique within the organization's notification orders.

#### resourceId
Type: _string_

The id of the Altinn resource the notification should be related to as the id appears in the Altinn Resource Registry. 
For an Altinn app the format of the resource is is `app_{org}_{app}` e.g. app_ttd_apps-test.

#### ignoreReservation
Type: _boolan_

A boolean indicating wether the notification content satisfies the requirements for overriding KRR reservations
when sending notifications to an individual.

#### conditionEndpoint
Type: _Url_

The URL to use when checking whether the condition to send the notification is met.

## Response

### Response codes
- 202 Accepted: The request was accepted and a notification order has been successfully generated.
- 400 Bad Request: The request was invalid. Refer to problem details in response body for further information.
- 401 Unauthorized: Indicates a missing, invalid or expired authorization header.
- 403 Forbidden: Indicates missing or invalid scope or Platform Access Token.

### Content-Type
- application/json

### Response body

The response body is formatted as an
[NotificationOrderRequestResponseExt.cs](hhttps://github.com/Altinn/altinn-notifications/blob/main/src/Altinn.Notifications/Models/NotificationOrderRequestResponseExt.cs)
and serialized as JSON.

Find a short description of each property below.

#### orderId
Type: _GUID_

The generated id for the notification order.

#### recipientLookup\*
Type: [_RecipientLookupResultExt_](https://github.com/Altinn/altinn-notifications/blob/main/src/Altinn.Notifications/Models/RecipientLookupResultExt.cs)

The result object describing the result of the recipient lookup containing the properties below.

  - _status_: the result of the initial lookup
  - _isReserved_: a list containing national identity numbers for recipients that are reserved
  - _missingContact_: a list containing national identity numbers and organization numbers for recipients where contact
    details for selected notification channel were not identified


|   Status   |                                   Description                |
| :--------: | :----------------------------------------------------------: |
| Success | The recipient lookup was successful for all recipients.         |
| PartialSuccess | The recipient lookup was successful for some recipients. |
| Failed  | The recipient lookup failed for all recipients.                 |

\* Property is only included if order request requires recipient lookup

### Response headers

#### Location
Type: _URL_

The self link for the generated notification order

## Examples

### Request
{{% notice info %}}
In the example we have included place holders for both the Platform Access and Altinn token.

__You only need one of them__, reference the [Authentication section](#authentication) for which one applies to your use case.
{{% /notice %}}


```bash
curl --location 'https://platform.altinn.no/notifications/api/v1/orders/sms' \
--header 'Content-Type: application/json' \
--header 'PlatformAccessToken: [INSERT PLATFORM ACCESS TOKEN]' \
--header 'Authorization: Bearer [INSERT ALTINN TOKEN]' \
--data-raw '{
    "sendersReference": "ref-2024-01-01",
	"body": "A text message to be sent immediately from an org.",
    "recipients":[
        {"mobileNumber":"+4799999999"},
        {"nationalIdentityNumber":"11876995923"},
        {"organizationNumber":"311000179"}]
}'
```

### Response

#### 202 Accepted
In cases where reservation check or address lookup of recipients is required for an order,
the initial result of the lookup will be included in the response. This includes
a list containing national identity number for all reserved persons and a list containing national identity number
or organization number for the recipients we could not find contact details for.


Headers:

```bash
-- header 'Location: https://platform.altinn.no/notifications/api/v1/orders/f1a1cc30-197f-4f34-8304-006ce4945fd1'
```

Body:

```json
{
    "orderId": "f1a1cc30-197f-4f34-8304-006ce4945fd1",
    "recipientLookup": {
        "status": "PartialSuccess",
        "isReserved": ["11876995923"],
        "missingContact": []
    }
}
```

#### 400 Bad Request
Response contains a problem details object with the error message in the detail property.

```json
{
    "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
    "title": "One or more validation errors occurred.",
    "status": 400,
    "traceId": "00-9ac2962c93d79629aa5c3744e4259663-344b49720aa49b0a-00",
    "errors": {
        "Subject": [
            "'Body' must not be empty."
        ]
    }
}
```
