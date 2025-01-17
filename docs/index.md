# Auth SDK Guide

This guide details the step-by-step of how to use the [MOSIP Authentication Python SDK](https://docs.mosip.io/1.2.0/modules/id-authentication-services/mosip-authentication-sdk){:target="_blank"}.

## Pre-Requisites

1. Download the MOSIP-signed certificate, the IDA partner certificate, and the keystore files [here](https://drive.google.com/drive/folders/1H72f30k_ubB5rReEHmYLE7n0tMFyUvd2?usp=sharing){:target="_blank"}. These are the certificates returned by MOSIP after [partner onboarding](partner_onboarding.md). The password for the keystore files will be provided.

2. Obtain a UIN to be used for testing out the SDK. For MOSIP's Collab environment, this can be done via [this Google form](https://docs.google.com/forms/d/e/1FAIpQLSc2I0CQqlYRIrEmcJ3J3tKlYOVNcYNj88YZe4MMwU2RZTrjOA/viewform){:target="_blank"}. Otherwise, [this set of mock data](https://docs.esignet.io/try-it-out/using-mock-data){:target="_blank"} can also be used for testing.

3. Install **Python** and **pip**.


## Installing the Auth SDK

Use `pip install` to install the SDK.

```sh
pip install git+https://github.com/mosip/ida-auth-sdk.git@v0.9.0
```

## Configuring the Auth SDK

1. Create a `config.toml` configuration file. A sample configuration file can be seen below.

        [mosip_auth]
        timestamp_format = "%Y-%m-%dT%H:%M:%S"
        ida_auth_version = "1.0"
        ida_auth_request_demo_id = "mosip.identity.auth"
        ida_auth_request_kyc_id = "mosip.identity.kyc"
        ida_auth_request_otp_id = "mosip.identity.otp"
        ida_auth_env = "Staging"
        authorization_header_constant = "Authorization"
        partner_apikey = ""
        partner_misp_lk = ""
        partner_id = ""

        [mosip_auth_server]
        ida_auth_domain_uri = ""
        ida_auth_url = ""

        [crypto_encrypt]
        symmetric_key_size = 256
        symmetric_nonce_size = 128
        symmetric_gcm_tag_size = 128
        encrypt_cert_path = ""
        decrypt_p12_file_path = ""
        decrypt_p12_file_password = ""

        [crypto_signature]
        algorithm = "RS256"
        sign_p12_file_path =  ""
        sign_p12_file_password = ""

        [logging]
        log_file_path = "authenticator.log"
        log_format = "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
        loglevel = "DEBUG" # can be one of "DEBUG" | "INFO" | "WARNING" | "ERROR" | "CRITICAL"
    
2. For testing purposes, input the following partner values into the configuration file:
  
    - **partner_apikey**: `906308`
    - **partner_misp_lk**: `9FChkkfixwsSFSvBaA7oCOkfNnBUj2XIZObAtyOGHsIAyG0JOG`
    - **partner_id**: `mpartner-default-piidtl`

    These values are normally received from MOSIP during partner onboarding.

3. For the ID Authentication URLS, input the following values:

    - **ida_auth_domain_url**: `https://api-internal.collab.mosip.net`
    - **ida_auth_url**: `https://api.collab.mosip.net/idauthentication/v1`

4. For the certificate and keystore file paths, input the following values:

    - **encrypt_cert_path**: the path to the `ida_partner.pem` certificate
    - **decrypt_p12_file_path**: the path to the `keystore.p12` certificate
    - **decrypt_p12_file_password**: the password of the `keystore.p12` certificate
    - **sign_p12_file_path**: the path to the `keystore_sign.p12` certificate
    - **sign_p12_file_password**: the password of the `keystore_sign.p12` certificate

The next sections show how to use the SDK for different types of authentication.

## More Information

[MOSIP Auth SDK Configuration Parameters](https://mosip.atlassian.net/wiki/external/NjczOWRmMGQ0MzUxNGQ0OTgzYzg1MTk3ZGU2N2ExNjg){:target="_blank"}