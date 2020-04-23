<a name="table-of-contents"></a>
## Table of contents

- [Get Started](#get-started)
- AUTH API
	- [Register New User](#register-new-user)
	- [Pair Device](#pair-device)
	- [Setup PIN Code](#setup-pin-code)
	- [Get Devices](#get-devices)
	- [Unpair Devices](#unpair-devices)
	- [Send Push](#send-push)
	- [Get Device 2FA](#get-device-2fa)
	- [Cancel Device 2FA](#cancel-device-2fa)
	- [Get User Info](#get-user-info)
	- [Query Callback Status](#query-callback-status)
	- [Verify User TOTP](#verify-user-totp)
  - [Send Login OTP Email](#send-login-otp)
- Testing
	- [Mock Server](#mock-server)
	- [CURL Testing Commands](#curl-testing-commands)
- Appendix
	- [Callback Definition](#callback-definition)
	- [API Callback List](#api-callback-list)

<a name="get-started"></a>
# Get Started

- Get API code and API secret on web admin console
- Refer to [mock server](#mock-server) sample code 

# AUTH API

<a name="register-new-user"></a>
## Register New User

Register a new user.

**`POST`** /v1/api/users 

- [Sample curl command](#curl-register-new-user)

##### Request Format

An example of the request:

###### Post body

```json
{
  "name": "JOHN DOE",
  "account": "johndoe",
  "email": "johndoe@example.com",
  "locale": "en",
  "bound_limit": 1
}
```

The request includes the following parameters:

###### Post body

| Field | Type  | Description |
| :---  | :---  | :---        |
| name | string | User name |
| account | string | User account (SYSTEM UNIQUE ID) |
| email | string | User email (OPTIONAL) |
| locale | string | User preference locale (en, zh-TW, zh-CN, ja, ko) |
| bound_limit | int | The upper bound of binding deivce. <=0: unlimited |

> The `account` field is a system-wise unique ID

##### Response Format

An example of a successful response:


```json
{
  "account": "johndoe",
  "email": "johndoe@example.com"
}
```

The response includes the following parameters:

| Field | Type  | Description |
| :---  | :---  | :---        |
| account  | string | Registered user's account  |
| email  | string | Registered user's email  |

#### Possible Errors

| HTTP Status Code | Error Code  | Error Message |
| :---             | :---        | :---          |
| 400  | 103 | Account already exists |

##### Refer to [Common Errors](#common-errors)

##### [Back to top](#table-of-contents)


<a name="pair-device"></a>
## Pair Device

Pair device with user. 
> This API has a callback.

**`POST`** /v1/api/devices?account=`USER_ACCOUNT`

- [Sample curl command](#curl-pair-device)

##### Request Format

An example of the reques:

###### API with query string

```
/v1/api/devices?account=johndoe
```

The request includes the following parameters:

###### Query string

| Field | Type  | Description |
| :---  | :---  | :---        |
| account      | string | Requester account      |

##### Response Format

An example of a successful response:


```json
{
  "order_id": 10000000001,
  "url": "http://192.168.101.71:8080/v1/auth/devices?token=Fekj7FFHaKZ1M2M7NT2tK9ziT3UT7q3pHzY7i27oUDVb"
}
```

The response includes the following parameters:

| Field | Type  | Description |
| :---  | :---  | :---        |
| order_id | int64 | Callback ID. Use this ID to identify specific callback. |
| url   | string | URL to bind device with user. Convert to QR code then scan with CYBAVO Auth APP. |

> Only retrieve token value from url then covert to QR code to pair.

#### Callback

An example of a callback:

```
{
  "behavior_result": 2,
  "behavior_type": 1,
  "order_id": 10000000001,
  "service_id": 5
}
```
- [View callback definition](#callback-definition)

#### Possible Errors

##### Refer to [Common Errors](#common-errors)

##### [Back to top](#table-of-contents)


<a name="setup-pin-code"></a>
## Setup PIN Code

Setup user PIN code.

**`POST`** /v1/api/user/pin?account=`USER_ACCOUNT`

- [Sample curl command](#curl-setup-pin-code)

##### Request Format

An example of the request:

###### API with query string

```
/v1/api/users/pin?account=johndoe
```

The request includes the following parameters:

###### Query string

| Field | Type  | Description |
| :---  | :---  | :---        |
| account | string | Requester account |
| service_id | int64 | Requester service ID |

##### Response Format

An example of a successful response:

```json
{
  "order_id": 20000000008
}
```

The response includes the following parameters:

| Field | Type  | Description |
| :---  | :---  | :---        |
| order_id | int64 | Callback ID. Use this ID to identify specific callback. |

#### Callback

An example of a callback:

```
{
  "behavior_result": 2,
  "behavior_type": 2,
  "order_id": 20000000008,
  "service_id": 2
}
```
- [View callback definition](#callback-definition)

#### Possible Errors

##### Refer to [Common Errors](#common-errors)

##### [Back to top](#table-of-contents)


<a name="get-devices"></a>
## Get Devices

Get user paired devices.

**`GET`** /v1/api/devices?account=`USER_ACCOUNT`

- [Sample curl command](#curl-get-devices)

##### Request Format

An example of the request:

###### API with query string

```
/v1/api/devices?account=johndoe
```

The request includes the following parameters:

###### Query string

| Field | Type  | Description |
| :---  | :---  | :---        |
| account      | string | Requester account |

##### Response Format

An example of a successful response:

```json
{
  "devices": [
    {
      "name": "OnePlus A0001",
      "platform": "Android 7.1.2",
      "device_id": "83u7FaTEFxDXadF73vMoGfftqpb2VUkqPEcL9wEoUQYC",
      "service_id": 2,
      "last_active_time": 1574305381,
      "create_time": 1574304819
    }
  ]
}
```

The response includes the following parameters:

| Field | Type  | Description |
| :---  | :---  | :---        |
| name | string | Device name |
| platform | string | Device platform |
| device_id | string | Device UID |
| service_id | int64 | ID of linked service |
| last\_active\_time | int64 | Last device active unix time in UTC |
| create\_time | int64 | Device creation unix time in UTC |

#### Possible Errors

##### Refer to [Common Errors](#common-errors)

##### [Back to top](#table-of-contents)


<a name="unpair-devices"></a>
## Unpair Devices

Unpair user's devices.

**`DELETE`** /v1/api/devices?account=`USER_ACCOUNT`

- [Sample curl command](#curl-unpair-devices)

##### Request Format

An example of the request:

###### API with query string

```
/v1/api/devices?account=johndoe
```

###### Post body

```json
{
  "devices": ["DBLVRUU9dSHjYFBUNQTJrsxMiJN8iz6iRKdXtVdHQawC"]
}
```

The request includes the following parameters:

###### Query string

| Field | Type  | Description |
| :---  | :---  | :---        |
| account | string | Requester account |

###### Post body

| Field | Type  | Description |
| :---  | :---  | :---        |
| devices | array of string | ID of devices about to unpairing |

##### Response Format

An example of a successful response:

```json
{
  "removed_devices": [
    "DBLVRUU9dSHjYFBUNQTJrsxMiJN8iz6iRKdXtVdHQawC"
  ]
}
```

The response includes the following parameters:

| Field | Type  | Description |
| :---  | :---  | :---        |
| removed_devices | array of string | ID of unpaired devices |

#### Possible Errors

##### Refer to [Common Errors](#common-errors)

##### [Back to top](#table-of-contents)


<a name="send-push"></a>
## Send Push

Send push notification to devices.

**`POST`** /v1/api/devices/2fa?account=`USER_ACCOUNT`

- [Sample curl command](#curl-send-push)

##### Request Format

An example of the request:

###### API with query string

```
/v1/api/devices/2fa?account=johndoe
```

###### Post body

```json
{
  "type": 1,
  "title": "TITLE OF PUSH",
  "body": "BODY OF PUSH",
  "data": {
    "custom_field01": "CUSTOM MSG",
    "custom_field02": "CUSTOM MSG 02"
  },
  "client_ip": "192.168.0.111",
  "client_platform": 1
}
```

The request includes the following parameters:

###### Query string

| Field | Type  | Description |
| :---  | :---  | :---        |
| account | string | Requester account |

###### Post body

| Field | Type  | Description |
| :---  | :---  | :---        |
| type | int | Service event type |
| title | string | Title of push notification |
| body | string | Body of push notification |
| data | object | User defined custom data |
| client_ip | string | Requester IP address |
| client_platform | int | Refer to [Platform](#platform-definition) |


##### Response Format

An example of a successful response:

```json
{
  "success_devices": [
    "DBLVRUU9dSHjYFBUNQTJrsxMiJN8iz6iRKdXtVdHQawC"
  ],
  "action": 2,
  "order_id": 10000000027,
  "matched_policy": {
    "policy_id": 0,
    "rule_type": 0
  }
}
```

The response includes the following parameters:

| Field | Type  | Description |
| :---  | :---  | :---        |
| success_devices | array of string | ID of devices will get push |
| action | int | Refer to [Action](#action-definition) |
| order_id | int64 | ID of callback |
| matched_policy | object | Matched policy info, refet to [Rule Type](#rule-type-definition) |

#### Callback

An example of a callback:

```
{
  "behavior_result": 2,
  "behavior_type": 1,
  "order_id": 10000000027,
  "service_id": 2
}
```
- [View callback definition](#callback-definition)

#### Possible Errors

##### Refer to [Common Errors](#common-errors)

##### [Back to top](#table-of-contents)


<a name="get-device-2fa"></a>
## Get Device 2FA

Get user's devices 2FA status.

**`GET`** /v1/api/users/2fa?account=`USER_ACCOUNT`&start\_index=`FROM`&request\_number=`COUNT`

**`GET`** /v1/api/users/2fa?account=`USER_ACCOUNT`&order\_id=`CALLBACK_ID`

- [Sample curl command](#curl-get-device-2fa)

##### Request Format

An example of the request:

###### API with query string

```
/v1/api/users/2fa?account=johndoe&start_index=0&request_number=10
/v1/api/users/2fa?account=johndoe&order_id=10000000012
```

The request includes the following parameters:

###### Query string

| Field | Type  | Description |
| :---  | :---  | :---        |
| account      | string | Requester account |

##### Response Format

An example of a successful response:

```json
{
  "items": [
    {
      "order_id": 10000000031,
      "type": 272,
      "user_action": 2,
      "state": 1,
      "updated_time": 1574405409,
      "message_type": 1,
      "message_title": "TITLE OF PUSH",
      "message_body": "BODY OF PUSH",
      "device_sent": 1
    }
  ]
}
```

The response includes the following parameters:

| Field | Type  | Description |
| :---  | :---  | :---        |
| order_id | int64 | Callback ID |
| type | int | Refer to [2FA Type](#2fa-type-definition) |
| user\_action | int | Refer to [User Action](#user-action-definition) |
| state | int | Refer to [2FA State](#2fa-state-definition) |
| updated\_time | int64 | 2FA update unix time in UTC |
| message_title | string | Title of push notification  |
| message_body | string | Body of push notification  |
| message_type | int64 | Service event type |
| device_sent | int | Device count that received this push notification |

#### Possible Errors

##### Refer to [Common Errors](#common-errors)

##### [Back to top](#table-of-contents)


<a name="cancel-device-2fa"></a>
## Cancel Device 2FA

Cancel the 2FA sent via `/v1/api/devices/2fa` API.

**`DELETE`** /v1/api/users/2fa/`ORDER_ID`?account=`USER_ACCOUNT`

- [Sample curl command](#curl-cancel-device-2fa)

##### Request Format

An example of the request:

###### API with query string

```
/v1/api/users/2fa/10000000012?account=johndoe
```

The request includes the following parameters:

###### Query string

| Field | Type  | Description |
| :---  | :---  | :---        |
| account | string | Requester account |

##### Response Format

An example of a successful response:

```json
{
  "canceled_devices": [
    "DBLVRUU9dSHjYFBUNQTJrsxMiJN8iz6iRKdXtVdHQawC"
  ]
}
```

The response includes the following parameters:

| Field | Type  | Description |
| :---  | :---  | :---        |
| canceled_devices | array of string | Devices' 2FA has been canceled |

#### Possible Errors

##### Refer to [Common Errors](#common-errors)

##### [Back to top](#table-of-contents)


<a name="get-user-info"></a>
## Get User Info

Get user info.

**`GET`** /v1/api/users/me?account=`USER_ACCOUNT`

- [Sample curl command](#curl-get-user-info)

##### Request Format

An example of the request:

###### API with query string

```
/v1/api/users/me?account=johndoe
```

The request includes the following parameters:

###### Query string

| Field | Type  | Description |
| :---  | :---  | :---        |
| account | string | Requester account |

##### Response Format

An example of a successful response:

```json
{
  "account": "johndoe",
  "user_email": "johndoe@example.com",
  "service_id": 2,
  "device_count": 1,
  "is_setup_pin": true
}
```

The response includes the following parameters:

| Field | Type  | Description |
| :---  | :---  | :---        |
| account | string | User account |
| user_email | string | User email |
| service\_id | int64 | Service ID |
| device\_count | int | Count of paired devices |
| is\_setup\_pin | boolean | PIN is set or not |

#### Possible Errors

##### Refer to [Common Errors](#common-errors)

##### [Back to top](#table-of-contents)


<a name="query-callback-status"></a>
## Query Callback Status

Query callbacks' status belong to requester's company.

**`POST`** /v1/api/order/status

- [Sample curl command](#curl-query-callback-status)

##### Request Format

An example of the request:

###### API with query string

```
/v1/api/order/status?account=johndoe
```

###### Post body

```json
{
  "order_ids": [
    10000000006,
    20000000010
  ]
}
```

The request includes the following parameters:

###### Query string

| Field | Type  | Description |
| :---  | :---  | :---        |
| account | string | Requester account |

###### Post body

| Field | Type  | Description |
| :---  | :---  | :---        |
| order_ids | array | ID of callbacks |

##### Response Format

An example of a successful response:

```json
{
  "order_status": [
    {
      "is_exist": true,
      "order_id": 10000000006,
      "behavior_type": 1,
      "behavior_result": 0,
      "addon": {}
    },
    {
      "is_exist": true,
      "order_id": 20000000010,
      "behavior_type": 2,
      "behavior_result": 2,
      "addon": {}
    }
  ]
}
```

The response includes the following parameters:

| Field | Type  | Description |
| :---  | :---  | :---        |
| is_exist | boolean | This order_id exists or not |
| order_id | int64 | ID of callback |
| behavior_type | int | [View callback definition](#callback-definition) |
| behavior_result | int | [View callback definition](#callback-definition) |
| addon | object | The custom fields of request |


<a name="verify-user-totp"></a>
## Verify User TOTP

Verify user's totp.

**`GET`** /v1/api/users/totpverify?account=`USER_ACCOUNT`&code=`TOTP`

- [Sample curl command](#curl-verify-user-totp)

##### Request Format

An example of the request:

###### API with query string

```
/v1/api/users/totpverify?account=johndoe&code=539826
```

The request includes the following parameters:

###### Query string

| Field | Type  | Description |
| :---  | :---  | :---        |
| account | string | Requester account |
| code | int | totp |

##### Response Format

An example of a successful response:

```json
{
  "result": true
}
```

The response includes the following parameters:

| Field | Type  | Description |
| :---  | :---  | :---        |
| result | boolean | The result of totp verification |

<a name="send-login-otp"></a>
## Send Login OTP to Email

Users click the link in email to login

**`POST`** /v1/api/users/emailotp?account=`USER_ACCOUNT`

- [Sample curl command](#curl-send-login-otp)

##### Request Format

An example of the request:

###### API with query string

```
/v1/api/users/emailotp?account=johndoe
```

###### Post body

```json
{
  "url": "https://serviceprovider.com.tw/redirect",
  "duration": 30,
}
```

The request includes the following parameters:

###### Query string

| Field | Type  | Description |
| :---  | :---  | :---        |
| account | string | Requester account |

##### Response Format

An example of a successful response:

```json
{
}
```


<a name="mock-server"></a>
# Mock Server

## Setup Configuration
>	Set following configuration in mockserver.app.conf
>> Require API code and API secret on web admin console

```
"api_server_url": "BACKEND_SERVER_URL",
"api_code": "SERVICE_API_CODE",
"api_secret": "SERVICE_API_SECRET"
```

## Register mock server URL
>	Operate on web admin console

Callback URL

```
http://localhost:8892/v1/mock/callback
```

## How to compile
- Execute
	- npm install
	- npm start

> Required version: node v10.19.0 or later (npm v6.13.4 or later)

<a name="curl-testing-commands"></a>
## CURL Testing Commands

<a name="curl-register-new-user"></a>
#### Register a new user
```
curl -X POST -H "Content-Type: application/json" -d '{"account":"johndoe","name":"JOHN DOE","email":"johndoe@example.com","locale":"en"}' \
"http://localhost:8892/v1/mock/users"
```
- [API definition](#register-new-user)

<a name="curl-pair-device"></a>
#### Pair device
```
curl -X POST -H "Content-Type: application/json" "http://localhost:8892/v1/mock/devices?account=johndoe"
```
- [API definition](#pair-device)

<a name="curl-setup-pin-code"></a>
#### Setup PIN code
```
curl -X POST -H "Content-Type: application/json" "http://localhost:8892/v1/mock/users/pin?account=johndoe"
```
- [API definition](#setup-pin-code)

<a name="curl-get-devices"></a>
#### Get devices
```
curl -X GET "http://localhost:8892/v1/mock/devices?account=johndoe"
```
- [API definition](#get-devices)

<a name="curl-unpair-devices"></a>
#### Remove devices
```
curl -X DELETE -H "Content-Type: application/json" -d '{"devices":["DEVICE_ID"]}' \
"http://localhost:8892/v1/mock/devices?account=johndoe"
```
- [API definition](#unpair-devices)

<a name="curl-send-push"></a>
#### Send push
```
curl -X POST -H "Content-Type: application/json" -d '{"type":1,"title":"PUSH TITLE","body":"PUSH DESC","data":{"field_1_key":"field_1_value","field_2_key":"field_2_value"},"client_ip":"192.168.101.71","client_platform":1}' \
"http://localhost:8892/v1/mock/devices/2fa?account=johndoe"
```
- [API definition](#send-push)

<a name="curl-get-device-2fa"></a>
#### Get device 2FA
```
curl -X GET "http://localhost:8892/v1/mock/users/2fa?account=johndoe&start_index=FROM&request_number=COUNT"
curl -X GET "http://localhost:8892/v1/mock/users/2fa?account=johndoe&order_id=ORDER_ID"
```
- [API definition](#get-device-2fa)

<a name="curl-cancel-device-2fa"></a>
#### Cancel device 2FA
```
curl -X DELETE -H "Content-Type: application/json" "http://localhost:8892/v1/mock/users/2fa/ORDER_ID?account=Erline"
```
- [API definition](#cancel-device-2fa)

<a name="curl-query-user-info"></a>
#### Query user info
```
curl -X GET "http://localhost:8892/v1/mock/users/me?account=johndoe"
```
- [API definition](#query-user-info)

<a name="curl-query-callback-status"></a>
#### Query callback status
```
curl -X POST -H "Content-Type: application/json" -d '{"order_ids":[10000000002,10000000003]}' \
"http://localhost:8892/v1/mock/order/status?account=johndoe"
```
- [API definition](#query-callback-status)

<a name="curl-verify-user-totp"></a>
#### Verify user TOTP
```
curl http://localhost:8892/v1/mock/users/totpverify?account=johndoe&code=539826
```
- [API definition](#verify-user-totp)

<a name="curl-send-login-otp"></a>
#### Send Login OTP to Email
```
curl -X POST -H "Content-Type: application/json" -d '{"url":"http://localhost:8080", "duration":10}' \
http://localhost:8892/v1/mock/users/emailotp?account=johndoe
```
- [API definition](#send-login-otp)

##### [Back to top](#table-of-contents)


# Appendix

<a name="callback-definition"></a>
## Callback Definition

<table>
  <tr>
    <td>Field</td>
    <td>Type</td>
    <td>Description</td>
  </tr>
  <tr>
    <td>order_id</td>
    <td> int64 </td>
    <td>ID of callback</td>
  </tr>
  <tr>
    <td>service_id</td>
    <td>int64</td>
    <td>ID of request service</td>
  </tr>
  <tr>
    <td>behavior_type</td>
    <td>int</td>
    <td rowspan="3">
      <b>1</b> - Pair Device<br>
      <b>2</b> - Setup PIN<br>
      <b>9</b> - Custom Message<br>
    </td>
  </tr>
  <tr/><tr/>
  <tr>
    <td>behavior_result</td>
    <td>int</td>
    <td rowspan="5">
      <b>0</b> - Pending<br>
      <b>1</b> - Rejected<br>
      <b>2</b> - Accepted<br>
      <b>3</b> - Expired<br>
      <b>4</b> - Failed
    </td>
  </tr>
</table>

##### [Back to top](#table-of-contents)

<a name="platform-definition"></a>
## Platform Definition

| ID   | Description |
| :--- | :---        |
| 1  | Android |
| 2  | iOS |
| 4  | Browser |

##### [Back to top](#table-of-contents)

<a name="action-definition"></a>
## Action Definition

| ID   | Description |
| :--- | :---        |
| 0  | No Action |
| 1  | Always Allow |
| 2  | Require 2FA |
| 3  | Deny Access |

##### [Back to top](#table-of-contents)

<a name="user-action-definition"></a>
## User Action Definition

| ID   | Description |
| :--- | :---        |
| 0  | Pending |
| 1  | Accept |
| 2  | Reject |

##### [Back to top](#table-of-contents)

<a name="2fa-type-definition"></a>
## 2FA Type Definition

| ID   | Description |
| :--- | :---        |
| 545 | Set User PIN |
| 272 | Service Accept/Reject Event |
| 769 | Service PIN Code Event |

##### [Back to top](#table-of-contents)

<a name="2fa-state-definition"></a>
## 2FA State Definition

| ID   | Description |
| :--- | :---        |
| 0 | Created |
| 1 | Done |
| 2 | Passed |
| 3 | Canceled |
| 4 | Failed |

##### [Back to top](#table-of-contents)

<a name="rule-type-definition"></a>
## Rule Type Definition

| ID   | Description |
| :--- | :---        |
| 1  | Global Rule |
| 2  | User Defined Rule |
| 3  | Risk Control Rule |

##### [Back to top](#table-of-contents)

<a name="common-errors"></a>
## Common Errors

| HTTP Status Code | Error Code  | Error Message |
| :---             | :---        | :---          |
| 400  | 112 | Invalid parameter |
| 400  | - | JSON unmarshal failure reason |
| 403  | - | Forbidden |
| 403  | 703 | Operation failed |
| 503  | 903 | KMS out of serivce. Please try again later. |

##### [Back to top](#table-of-contents)
