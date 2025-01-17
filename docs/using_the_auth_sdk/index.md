# Using the Auth SDK

The MOSIP Authentication SDK can perform both yes/no and KYC authentications, with support for multi-factor authentication using biometrics, demographic data, and OTPs (see [Authentication Types](https://docs.mosip.io/1.2.0/id-authentication#authentication-types){:target="_blank"} for more information). In this guide, however, only demographic data and OTPs are used for yes/no and KYC authentication since a third-party SDK is needed to capture biometric data.

## Auth SDK Example

Here is a working example of an authentication request.

``` python title="Yes/No Auth"
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

Before authenticating, a `MOSIPAuthenticator` object that uses the config file described in the previous section has to be created.

Create a `Dynaconf` object whose `settings_files` points to the path of the previously created `config.toml`, then use this object to create the `MOSIPAuthenticator` object.

``` python title="Yes/No Auth" hl_lines="2 3 5 6"
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

The next sections detail how to use the SDK to perform yes/no and KYC authentications.