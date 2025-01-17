# OTP Authentication

Instead of demographic data, OTPs can also be used for both yes/no and KYC authentications. They can also be used *in combination* with demographic data. The Authentication SDK can be used to generate OTPs and verify them in authentication requests.

## Generating an OTP

To generate an OTP for an individual, use the `authenticator.genotp()` method shown below. Provide the ID of the individual and the type of ID, then specify whether to send the OTP to the individual's registered email or phone number (or both).

``` python title="Generating an OTP" hl_lines="7-12"
from mosip_auth_sdk import MOSIPAuthenticator
from dynaconf import Dynaconf

config = Dynaconf(settings_files=["./config.toml"], environments=False)
authenticator = MOSIPAuthenticator(config=config)

response = authenticator.genotp(
    individual_id="2047631038",
    individual_id_type="UIN",
    email=True,
    phone=True,
)

response_body = response.json()
print(response_body)
```

Use `response.json()` to get the response, which would look something like this:

``` json title="Successful OTP Generation"
{
    "id": "mosip.identity.otp",
    "version": "1.0",
    "transactionID": "1929353076",
    "responseTime": "2025-01-16T10:58:24.262Z",
    "errors": None,
    "response": {
        "maskedMobile": "XXXXXX0607",
        "maskedEmail": "XXmXX@test.io"
    }
}
```

## Using the OTP in Authentication Requests

To use OTPs in authentication requests, save the `transactionID` from the response in the OTP generation request detailed above. Input this along with the actual generated OTP into the `authenticator.auth()` or `authenticator.kyc()` method. For testing purposes, the OTP is always `111111` when using [mock data](https://docs.esignet.io/try-it-out/using-mock-data){:target="_blank"}.

``` python title="Using OTP in a KYC Auth Request" hl_lines="15 21-22"
from mosip_auth_sdk import MOSIPAuthenticator
from dynaconf import Dynaconf

config = Dynaconf(settings_files=["./config.toml"], environments=False)
authenticator = MOSIPAuthenticator(config=config)

response = authenticator.genotp(
    individual_id="2047631038",
    individual_id_type="UIN",
    email=True,
    phone=True,
)

response_body = response.json()
transaction_id = response_body["transactionID"]

response = authenticator.kyc(
    individual_id="2047631038",
    individual_id_type="UIN",
    consent=True,
    otp_value="111111",
    txn_id=transaction_id,
)

response_body = response.json()
decrypted_response = authenticator.decrypt_response(response_body)
print(decrypted_response)
```

When the correct OTP and transaction ID are provided, the authentication is successful. On the other hand, the following shows the response when an incorrect OTP is inputted:

``` json title="Using an Incorrect OTP"
{
    "transactionID": "2094084565",
    "version": "1.0",
    "id": "mosip.identity.kyc",
    "errors": [
        {
            "errorCode": "IDA-OTA-004",
            "errorMessage": "OTP is invalid",
            "actionMessage": "Please provide correct OTP value"
        }
    ],
    "responseTime": "2025-01-16T11:02:15.233Z",
    "response": {
        "kycStatus": False,
        "authToken": "257988270191056778127490615141298322",
        "thumbprint": None,
        "identity": None,
        "sessionKey": None
    }
}
```

The response shown below is what is received when an incorrect transaction ID is inputted:

``` json title="Using an Incorrect Transaction ID"
{
    "transactionID": None,
    "version": None,
    "id": None,
    "errors": [
        {
            "errorCode": "IDA-OTA-005",
            "errorMessage": "Input transactionID does not match transactionID of OTP Request"
        }
    ],
    "responseTime": "2025-01-16T11:04:00.084Z",
    "response": {
        "kycStatus": False,
        "authToken": None,
        "thumbprint": None,
        "identity": None,
        "sessionKey": None
    }
}
```

It is important to note that either demographic data, an OTP, or both must be provided when performing yes/no or KYC authentications. If neither is provided, the following response is received:

``` json title="No Demographic Data or OTP Provided"
{
    "transactionID": None,
    "version": None,
    "id": None,
    "errors": [
        {
            "errorCode": "IDA-MLC-008",
            "errorMessage": "No authentication type selected"
        }
    ],
    "responseTime": "2025-01-17T11:47:24.895Z",
    "response": {
        "kycStatus": False,
        "authToken": None,
        "thumbprint": None,
        "identity": None,
        "sessionKey": None
    }
}
```

## How it Works

Below is a sequence diagram showing how the OTP generation works using the Authentication SDK.

When an individual requests for an OTP from a relying party by providing their ID, the relying party uses the `authenticator.genotp()` method of the SDK to generate the OTP. Through this method, the SDK sends a POST request containing the ID to the OTP generation endpoint of MOSIP. MOSIP then generates and sends the OTP to the individual specified by the ID. It also responds to the SDK's request with the transaction ID and other metadata. The OTP (received by the individual) and the transaction ID can then be used by the relying party when authenticating using the SDK.

``` mermaid
sequenceDiagram
  autonumber
  Individual->>Relying Party: UIN
  Relying Party->>SDK: authenticator.genotp(UIN)
  SDK->>MOSIP: POST<br>/idauthentication/v1/otp/[partner_misp_lk]/[partner_id]/[partner_api_key]
  MOSIP->>SDK: metadata (including transaction ID)
  MOSIP->>Individual: OTP
  SDK->>Relying Party: metadata (including transaction ID)
  activate Relying Party
  Individual->>Relying Party: OTP
  Relying Party->>SDK: authenticator.auth(UIN, OTP, transaction_id)
  deactivate Relying Party
  SDK->>MOSIP: POST<br>/idauthentication/v1/auth/[partner_misp_lk]/[partner_id]/[partner_api_key]
  MOSIP->>SDK: authStatus (true/false) + metadata
  SDK->>Relying Party: authStatus (true/false) + metadata
```