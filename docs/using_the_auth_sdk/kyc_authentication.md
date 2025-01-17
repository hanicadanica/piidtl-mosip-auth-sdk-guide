# KYC Authentication

KYC (Know Your Client) Authentication is used to obtain a set of attributes related to an individual (e.g. their full address, their name in another language) given their ID and another piece of identifying data, such as their biometrics or an OTP. The set of attributes that can be shared with a relying party depends on the policy set by MOSIP, so it is possible that not all individual attributes are shared to a relying party.

## Authenticating using Demographic Data

KYC authentication can be performed using the `authenticator.kyc()` method shown below. Similar to a yes/no authentication request, the ID of the individual and the type of the ID must be provided in the KYC authentication request, along with some demographic data (using a `DemographicsModel` object) and individual consent.

``` python title="KYC Auth" hl_lines="1 8-15"
from mosip_auth_sdk.models import DemographicsModel
from mosip_auth_sdk import MOSIPAuthenticator
from dynaconf import Dynaconf

config = Dynaconf(settings_files=["./config.toml"], environments=False)
authenticator = MOSIPAuthenticator(config=config)

demographics_data = DemographicsModel(name=[{"language": "eng", "value": "James Rodrigious"}])

response = authenticator.kyc(
    individual_id="2047631038",
    individual_id_type="UIN",
    demographic_data=demographics_data,
    consent=True,
)

response_body = response.json()
print(response_body)
```

Just like with yes/no authentication, the response from MOSIP can be obtained using `response.json()`. The individual attributes in this response, however, are still encrypted in the `identity` parameter:

``` json title="Successful KYC Auth without Decryption"
{
    "transactionID": "2024805211",
    "version": "1.0",
    "id": "mosip.identity.kyc",
    "errors": None,
    "responseTime": "2025-01-16T10:51:10.692Z",
    "response": {
        "kycStatus": True,
        "authToken": "257988270191056778127490615141298322",
        "thumbprint": [thumbprint],
        "identity": [identity_data],
        "sessionKey": [session_key]
    }
}
```

To obtain the individual attributes, decrypt the response using `authenticator.decrypt_response()`:

``` python title="Decrypting the Response" hl_lines="17-19"
from mosip_auth_sdk.models import DemographicsModel
from mosip_auth_sdk import MOSIPAuthenticator
from dynaconf import Dynaconf

config = Dynaconf(settings_files=["./config.toml"], environments=False)
authenticator = MOSIPAuthenticator(config=config)

demographics_data = DemographicsModel(name=[{"language": "eng", "value": "James Rodrigious"}])

response = authenticator.kyc(
    individual_id="2047631038",
    individual_id_type="UIN",
    demographic_data=demographics_data,
    consent=True,
)

response_body = response.json()
decrypted_response = authenticator.decrypt_response(response_body)
print(decrypted_response)
```

The decrypted response of a successful authentication request is shown below. Attributes in multiple languages, when available, are also included in the response.

``` json title="Decrypted Response of Successful KYC Auth"
{
    "location1_eng": "ABC City",
    "name_eng": "James Rodrigious",
    "gender_eng": "Male",
    "location1_ara": "الرباط",
    "name_ara": "روناك ناياك",
    "gender_ara": "ذكر",
    "location1_fra": "ABC City",
    "name_fra": "James Rodrigious",
    "gender_fra": "Male",
    "phone": "8763740607",
    "dob": "1992/04/29",
    "email": "james@test.io",
    "face": [face_data]
}
```

If the KYC authentication request was unsuccessful, the response would look something like this:

``` json title="Unsuccessful KYC Auth with Error"
{
    "transactionID": "4460816640",
    "version": "1.0",
    "id": "mosip.identity.kyc",
    "errors": [
        {
            "errorCode": "IDA-DEA-001",
            "errorMessage": "Demographic data name in eng did not match",
            "actionMessage": "Please re-enter your name in eng"
        }
    ],
    "responseTime": "2025-01-16T10:53:52.884Z",
    "response": {
        "kycStatus": False,
        "authToken": "257988270191056778127490615141298322",
        "thumbprint": None,
        "identity": None,
        "sessionKey": None
    }
}
```

Note that the encrypted attributes in `identity` are not included in the response since the authentication request was unsuccessful.

## How it Works

Below is a sequence diagram showing the flow of a KYC authentication request using the SDK.

Similar to yes/no authentication, an individual gives their ID and some demographic data to the relying party who then uses the SDK's `authenticator.kyc()` method to perform a KYC authentication. When this method is invoked, the SDK sends a POST request containing the ID and the demographic data to the KYC authentication endpoint of MOSIP. In response, MOSIP sends a JSON containing the encrypted individual attributes and additional metadata. Which attributes are shared to the relying party are determined by the policy set by MOSIP. The SDK then decrypts the response using `authenticator.decrypt_response()` to get the plaintext individual attributes which are returned to the relying party.

``` mermaid
sequenceDiagram
  autonumber
  Individual->>Relying Party: UIN, demographic data<br>(e.g. name, date of birth)
  Relying Party->>SDK: authenticator.kyc(UIN, demographic data)
  SDK->>MOSIP: POST<br>/idauthentication/v1/kyc/[partner_misp_lk]/[partner_id]/[partner_api_key]
  MOSIP->>SDK: encrypted attributes + metadata
  SDK->>SDK: authenticator.decrypt_response(response)
  SDK->>Relying Party: decrypted attributes + metadata
```