# NexPay Payments API <small>Alpha 1.0</small>
![Twitter](https://img.shields.io/twitter/follow/nexpayxyz?style=social)
![website](https://img.shields.io/website?down_color=red&down_message=Offline&style=flat-square&up_color=green&up_message=Online&url=https%3A%2F%2Fnexpay.xyz)
![status](https://img.shields.io/uptimerobot/status/m787630985-b0cb5ac7af9a089ff4178c3e?style=flat-square)
![GitHub](https://img.shields.io/github/license/NexPay/NexPay-api?style=flat-square)
> Get started with NexPay Payments API to accept NexPay cards directly on your website or app. This documentation is currently still being developed and may not be fully accurate.

> **Heads-up!** The NexPay Payments API is currently in alpha and must not be used in a production environment.

## **Getting started**

### Registering as a merchant

1. Go to **https://nexpay.xyz/merchant/** and create a merchant account.
2. Visit the _integrations_ page to generate your test `username` and `password`. Live credentials will become available once this API is moved out of Beta.

### API Authentication
- All API calls require the merchant to authenticate using HTTP basic authentication using their `username` and `password`.

### HTTP method
- All API calls will use the POST method unless another method is specifically mentioned.

### Understanding responses
The NexPay API will always return a JSON response. More options like XML will be made available soon.

### Example cURL request
> curl --location --request POST 'https://nexpay.xyz/api/' \
--header 'Authorization: Basic `username`:`password`' \
--form 'merchant_id="1234"' \
--form 'amount="100"' \
--form 'card_number="2501012345678900"' \
--form 'valid_month="12"' \
--form 'valid_year="12"' \
--form 'cvv="000"' \
--form 'success_url="https://example.com/payment-success/"' \
--form 'fail_url="https://example.com/payment-failed/"'

### Example response - Success
>   {<br>
        "transaction_id": "000A0B000CD000.12345678",<br>
        "success": true,<br>
        "timezone": "UTC",<br>
        "date": "2021-03-25",<br>
        "time": 1234567890,<br>
        "exp_time": 1234567890,<br>
        "amount": 500,<br>
        "status": "code_sent",<br>
        "status1": "futher_authentication_required",<br>
        "message": "Success: OTP for transaction has been sent. Further authentication is required to complete this transaction.",<br>
        "otp_page": "https://nexpay.xyz/api/otp_page.php?payment=000A0B000CD000.12345678&cd=5d41402abc4b2a76b9719d911017c592",<br>
        "code": 614,<br>
        "code1": 616<br>
    }

### Example response - Failure
>   {<br>
        "status": "authentication_required",<br>
        "message": "Error! Authentication required. Please read the documentation available at https://nexpay.xyz/docs/",<br>
        "code": 615<br>
    }


## **Creating a payment intent**

- Every payment starts with a 'Payment Intent.' A Payment intent is created on the server side and includes amount, merchant ID and transaction ID.
- A payment intent will be valid only for 10 minutes after it has been made and the payment must be complete within this timeframe.

### Prerequisites

1. To create a payment intent, you will need your `username` and `password` which can be generated after you register as a merchant.
2. Additionally, you will also need to pass your **Merchant ID** as `merchant_id` which you can find in the _profile_ section of your merchant dashboard.
3. You must also pass the transaction **Amount** as `amount` in the request in the smallest currency units. To charge $100.99, you will add 10099 as the amount.
4. The users **Card number**, **cvv**, **expiry month**, and **expiry year** must be passed as `card_number`, `cvv`, `valid_month` and `valid_year` respectively.
5. To initiate a payment intent, you must also provide a `success_url` and a `fail_url` to which the user will be redirected to after the payment is complete based on the result of the payment.

### Parameters

|   Parameter      | Datatype | Description                                                                   | Length  |
| :--------------- |:--------:| ----------------------------------------------------------------------------- | :-----: |
| `merchant_id`    | Integer  | Merchant ID given to the merchant at the time of account registration         | 12      |
| `amount`         | Integer  | Amount in the smallest currency denomination                                  | 1 - 6   |
| `card_number`    | String   | Card Number of the NexPay card being charged                                  | 16      |
| `valid_month`    | String   | The validity month of the card                                                | 2       |
| `valid_year`     | String   | The validity year of the card                                                 | 2 / 4   |
| `cvv`            | String   | The CVV of the card                                                           | 3       |
| `success_url`    | String   | The URL where the user will be redirected to if the transaction is successful | 1 - 128 |
| `fail_url`       | String   | The URL where the user will be redirected to if the transaction fails         | 1 - 128 |

### Creating the intent

> **Heads-up!** NexPay does not allow recurring payments or allow merchants to store card information. Doing so is a breach of the merchant agreement and may result. Every NexPay card payment must be authorized by the cardholder.

1. Start by collecting the amount from the user or by setting this directly on the server side.
2. Collect the users NexPay card number, expiry date and CVV. This can be done with a simple form.
3. Make a POST request to the NexPay server at https://nexpay.xyz/api with all the required parameters as mentioned in the Prerequisites section.
4. Parse the JSON response. The response will include a `otp_page` to which the cardholder must be redirected to where they will enter their OTP to authorize the transaction.

## **Capturing a payment**
Payments can be captured using the following two methods
Payments can be captured in 2 ways, by checking if the payment is complete using the `getpaymentdetails` API or by setting a redirect URL to which the user will be redirected to after authenticating the payment.

### With redirect
The user will be redirected to either `success_url` or `fail_url` depending on the payment outcome.

### Using an API call
Merchants can make a `"getpaymentdetails"` API call to with the `transaction_id` and their `merchant_id`. NexPay will return all the details of the payment intent excluding private information such as card numbers. These responses will use status codes that are defined in this documentation. Reffer to the getpaymentdetails section for more.

## **Get payment details**

### Parameters

|   Parameter      | Datatype | Description                                                             | Length  |
| :--------------- |:--------:| ----------------------------------------------------------------------- | :-----: |
| `merchant_id`    | Integer  | Merchant ID given to the merchant at the time of account registration   | 12      |
| `transaction_id` | String   | Transaction ID of the payment. Received after creating a payment intent | 20 - 25 |

### Example response - Success
The below example is shown when a payment with the given `payment_id` is found and the payment has already been completed. Code: 623.
>   {<br>
        "transaction_id" : "ABCD1234.ABCD1234",<br>
        "date": "2021-03-25",<br>
        "time": 0000000000,<br>
        "exp_time": 0000000000,<br>
        "amount": 1000,<br>
        "code": 623<br>
    }

### Example response - Failure
The below error is shown when a payment with the given `payment_id` is not found. Code: 624.
>   {<br>
        "transaction_id" : *null*,<br>
        "date": "YYYY-MM-DD",<br>
        "time": 0000000000,<br>
        "code": 624<br>
    }

## **Canceling a payment**
Payments can be cancelled within 10 minutes after creating them. Payments are set to automatically expire after 10 minutes. We recommend cancelling payments before they automatically expire.

### Parameters for cancelling payment

|   Parameter      | Datatype | Description                                                             | Length  |
| :--------------- |:--------:| ----------------------------------------------------------------------- | :-----: |
| `merchant_id`    | Integer  | Merchant ID given to the merchant at the time of account registration   | 12      |
| `transaction_id` | String   | Transaction ID of the payment. Received after creating a payment intent | 20 - 25 |

### Example response - Success
The below example shows an error shown when the transaction ID was not found for the payment
> hello

### Example response - Failure
> hello

## **NexPay status codes**

NexPay uses status codes to show the staus of payments.

| Code  | Description  | Status |
| ----- | :------------------------------------------------------------------------------------------------------------------ | ----------------------------- |
| `599` | Payment pending, waiting for cardholder authentication                                                              | `payment_pending` |
| `600` | Not numeric                                                                                                         | `not_numeric` |
| `601` | API call is missing required parameters                                                                             | `missing_parameter` |
| `602` | Merchant not found                                                                                                  | `merchant_not_found` |
| `603` | Merchant Username and/or Password is incorrect                                                                      | `invalid_authorization` |
| `604` | Parameter given has invalid length | `parameter_length_invalid` |
| `605` | Card expired | `card_expired` |
| `606` | Card is inactive, may have been blocked | `card_invalid` |
| `607` | Card cannot be used for transactions online | `transaction_not_allowed` |
| `608` | Card declined for unavailable balance or exceeds daily limits | `declined` |
| `609` | Zero amount, the payment amount is set to 0 | `zero_amount_error` |
| `610` | Cardholder account authentication failed | `account_auth_fail` |
| `611` | Card may have been issued by NexPay but is invalid | `card_invalid` |
| `612` | Card not issued by NexPay | `not_nexpay_card` |
| `613` | Card did not pass Luhn check | `luhn_failed` |
| `614` | OTP sent to cardholder. | `otp_sent` |
| `615` | Basic authentication required with username and password | `authentication_required` |
| `616` | Awaiting OTP authentication | `futher_authentication_required` |
| `617` | Payment canceled by user | `payment_cancelled` |
| `618` | The OTP page was refreshed causing the payment to fail | `refresh_fail` |
| `619` | The OTP page was active for longer than 180 seconds and has automatically expired | `authentication_timeout` |
| `620` | No HTTPS: The NexPay server cannot gurantee that the connection is secured with HTTPS/SSL and has failed the transaction | `not_secured` |
| `621` | The payment intent has automatically expired after the 10 minute limit | `payment_expired` |
| `622` | OTP authentication failed as it did not match | `otp_authentication_failed` |
| `623` | Payment successfully complete | `payment_complete` |
| `624` | Payment not found. Invalid transaction ID. | `payment_not_found` |

[NexPay](https://nexpay.xyz/)
[Documentation](https://nexpay.xyz/docs/)
