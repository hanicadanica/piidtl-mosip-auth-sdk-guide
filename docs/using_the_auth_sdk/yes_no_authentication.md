# Yes/No Authentication

Yes/No authentication is used to verify if the given attributes of an individual (in this case demographic data) and the individual's ID (which could be a UIN or VID) are a match. It essentially tells the relying party (i.e. authentication partner like banks and government institutes) whether or not an individual is who they say they are based on the given information.

## Authenticating using Demographic Data

Yes/no authentication can be performed using the `authenticator.auth()` method shown below. Provide the ID of the individual and the type of the ID. In addition, the demographic data to be verified must be provided in the form of a `DemographicsModel` object. Consent must also be given by the individual, but for the purposes of testing this can just be set to `True` when using the SDK.

``` python title="Yes/No Auth" hl_lines="1 8-15"
from mosip_auth_sdk.models import DemographicsModel
from mosip_auth_sdk import MOSIPAuthenticator
from dynaconf import Dynaconf

config = Dynaconf(settings_files=["./config.toml"], environments=False)
authenticator = MOSIPAuthenticator(config=config)

demographics_data = DemographicsModel(dob="1992/04/29")

response = authenticator.auth(
    individual_id="2047631038",
    individual_id_type="UIN",
    demographic_data=demographics_data,
    consent=True,
)

response_body = response.json()
print(response_body)
```

To obtain the response from MOSIP, simply use `response.json()`.

``` python title="Yes/No Auth" hl_lines="17-18"
from mosip_auth_sdk.models import DemographicsModel
from mosip_auth_sdk import MOSIPAuthenticator
from dynaconf import Dynaconf

config = Dynaconf(settings_files=["./config.toml"], environments=False)
authenticator = MOSIPAuthenticator(config=config)

demographics_data = DemographicsModel(dob="1992/04/29")

response = authenticator.auth(
    individual_id="2047631038",
    individual_id_type="UIN",
    demographic_data=demographics_data,
    consent=True,
)

response_body = response.json()
print(response_body)
```

The following example response is what a successful yes/no authentication request (i.e. the given ID and the given demographic data match) would look like:

``` json title="Successful Yes/No Auth"
{
    "transactionID": "4170364546",
    "version": "1.0",
    "id": "mosip.identity.auth",
    "errors": None,
    "responseTime": "2025-01-16T10:40:40.511Z",
    "response": {
        "authStatus": True,
        "authToken": "257988270191056778127490615141298322"
    }
}
```

On the other hand, this is what the response would look like if the yes/no authentication request was unsuccessful:

``` json title="Unsuccessful Yes/No Auth with Error"
{
    "transactionID": "6742462846",
    "version": "1.0",
    "id": "mosip.identity.auth",
    "errors": [
        {
            "errorCode": "IDA-DEA-001",
            "errorMessage": "Demographic data dob did not match",
            "actionMessage": "Please re-enter your dob"
        }
    ],
    "responseTime": "2025-01-16T10:42:34.064Z",
    "response": {
        "authStatus": False,
        "authToken": "257988270191056778127490615141298322"
    }
}
```

## DemographicsModel

Any number and combination of demographic attributes can be used for authentication (e.g. name only, name and date of birth, phone number and email). A `DemographicsModel` object is created with these attributes and is passed into the authentication function. Some of the attributes that can be used in `DemographicsModel` are:

   - `age`
   - `dob`
   - `phoneNumber`
   - `emailId`
   - `postalCode`
   - `name`
   - `gender`
   - `fullAddress`

Because MOSIP has support for multiple languages, attributes such as `name`, `gender`, and `fullAddress` need to have their languages specified. These attributes are represented by a list of dictionaries, where each dictionary contains a language and the attribute written in that language.

``` python title="Yes/No Auth with Name Specified" hl_lines="8"
from mosip_auth_sdk.models import DemographicsModel
from mosip_auth_sdk import MOSIPAuthenticator
from dynaconf import Dynaconf

config = Dynaconf(settings_files=["./config.toml"], environments=False)
authenticator = MOSIPAuthenticator(config=config)

demographics_data = DemographicsModel(name=[{"language": "eng", "value": "James Rodrigious"}])

response = authenticator.auth(
    individual_id="2047631038",
    individual_id_type="UIN",
    demographic_data=demographics_data,
    consent=True,
)

response_body = response.json()
print(response_body)
```

## How it Works

Below is a sequence diagram showing the flow of a yes/no authentication request using the SDK.

An individual first gives their ID and some demographic data (like their name or date of birth) to the relying party. The relying party then uses the SDK's `authenticator.auth()` method to perform a yes/no authentication. This method makes the SDK send a POST request containing the ID and the demographic data to the ID authentication endpoint of MOSIP. In response, MOSIP sends a JSON containing the authentication status with additional data such as the transaction (i.e. request) ID and response time. The SDK then forwards this JSON as is to the relying party.

``` mermaid
sequenceDiagram
  autonumber
  Individual->>Relying Party: UIN, demographic data<br>(e.g. name, date of birth)
  Relying Party->>SDK: authenticator.auth(UIN, demographic data)
  SDK->>MOSIP: POST<br>/idauthentication/v1/auth/[partner_misp_lk]/[partner_id]/[partner_api_key]
  MOSIP->>SDK: authStatus (true/false) + metadata
  SDK->>Relying Party: authStatus (true/false) + metadata
```