# Partner Onboarding

Before the Authentication SDK can be used, an authentication partner (you) must first be onboarded into the MOSIP system. This involves generating certificates, submitting them to MOSIP to be approved and signed, and receiving the signed certificates, a partner ID, and an API key.

Details of the requirements can be found [here](https://docs.mosip.io/1.2.0/partners#authentication-partner-ap){:target="_blank"}.

## Generating Certificates

In order to be onboarded as a partner, partner certificates must be created and submitted to MOSIP who, in turn, will sign and return the certificates ready to be used for authentication.

MOSIP has provided a Windows script for generating the required certificates, found [here](https://drive.google.com/drive/folders/1mNrTD1D7xf8QMHKSaDk0PWLSGSRHgxP_?usp=sharing){:target="_blank"}. Change the appropriate attributes in `cert.properties` configuration file as needed, and make sure to have `npm` installed as it is needed by the script. Note that this script and the certificates it generates are to be used for testing only.

Once the certificates have been generated in the `certs/` folder, obtain the `Client.pem`, `RootCA.pem`, and `keystore.p12` files. These are the certificates that have to be submitted to MOSIP.

## Submitting Certificates to MOSIP

After obtaining the necessary certificates, they have to be submitted to MOSIP in order to be onboarded as an authentication partner.

### Via the Partner Management Portal

Currently, MOSIP has a Partner Management Portal in their [Collab environment](https://collab.mosip.net/){:target="_blank"} where partners should be able to upload their certificates. Details on how to do so can be found [here](https://docs.mosip.io/1.2.0/collab-getting-started-guide/collab-pmp-guide){:target="_blank"} and [here](https://docs.mosip.io/1.2.0/modules/partner-management-services/pms-existing/auth-credential-partner){:target="_blank"}.

### Via a Direct Email to MOSIP

If the Partner Management Portal is unavailable or the certificates cannot be uploaded, it may also be possible to email MOSIP at [getsupport@mosip.io](mailto:getsupport&#64;mosip.io) for manual partner onboarding.

## After Successful Onboarding

Once an authentication partner has been onboarded, they should have the following:

- a MOSIP-signed certificate
- an IDA partner certificate
- a partner ID
- a partner API key
- an MISP license key

The latter four items are used as-is in the Authentication SDK. The MOSIP-signed certificate, on the other hand, is used to sign the authentication partner's `keystore.p12` keystore file (as described in [Generating Certificates](#generating-certificates)).

### Preparing the Signed Keystore File

For signing the keystore file, MOSIP recommends using [Keystore Explorer](https://keystore-explorer.org/){:target="_blank"}.

1. Download Keystore Explorer, open it, and select **Open an existing KeyStore**.
2. Open the `keystore.p12` file and enter its password.
3. Right-click on the keystore entry and select **Import CA Reply > From File**.
4. Change the type of file to search for to **PEM files (.pem)** and select the MOSIP-signed certificate.
5. Save the changes to a new keystore file such as `keystore_signed.p12`.

The `keystore.p12` and `keystore_signed.p12` files can now be used along with the IDA partner certificate to authenticate using the Authentication SDK.