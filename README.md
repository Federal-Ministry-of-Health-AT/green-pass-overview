# Austrian implementation of the EU Digital COVID Certificates

This overview points to the relevant Austrian/EU repositories that contain the specifications for the EU Digital COVID Certificates (further abbrev. as DCC), the architecture and the source code of the Austrian implementation. This page and the repository structure are work in progress and will be updated continuously.

Versions
|Date (version)|Description |
|--|--|
| 02.06.2021 | initial version |
| 29.07.2021 | Details on architecture, trust lists/business rules/value sets |
| 05.08.2021 | Fixed links to greencheck.gv.at and test-systems, thx to Mathias Panzenböck |
| 19.08.2021 | Added information on national business rules |
| 06.10.2021 | Added information for Viennese business rules |
| 16.11.2021 | Added information for AT business rules and Engine "AT-METADATA"|

# General information
The following leads provided the entry points for the relevant EU repositories, which contain code and information.

* EHN repositories for various open source projects, which contain components for QR generation/verification or even complete creation/validation chains. Many of the Austrian components are published at this place, since there is a lively exchange and collaboration of developers: https://github.com/ehn-dcc-development/
* Repositories for reference implementations provided by the EU: https://github.com/eu-digital-green-certificates/
* Specification for creating the QR codes: https://github.com/ehn-dcc-development/hcert-spec/blob/main/hcert_spec.md
* Quality assurance test sets of member states DCCs: https://github.com/eu-digital-green-certificates/dcc-quality-assurance
* Business Rules: https://github.com/eu-digital-green-certificates/dgc-business-rules-testdata

# Architecture
![alt text](https://github.com/Federal-Ministry-of-Health-AT/green-pass-overview/blob/main/arch.png?raw=true)

 - **EU Gateway**: Central gateway operated by the EU which distributes information to the member states (trust list, business rules, value sets). Access to the EU Gateway is not public, but only possible via authenticated connections from backend services in the member states (Client TLS authentication required).
 - **AT Gateway client**: Austrian backend client which establishes a secure, authenticated connection to the EU gateway to retrieve information. The retrieved information is processed and distributed via Austrian services operated by BRZ. The following files are created every hour and can be used for a max. of 48 hours in an offline scenario.
	- **Trust list:** This list contains all the document signer certificates (DSC) of the DCC issuer states, which issue DCCs. These signature certificates are required to validated the validity and authenticity of the digital signature of the presented QR-Codes.
	 - **Business rules AT/EU**: 
		 - **EU**: These rules represent the legislation of the different member states, which are used to check the validity of the DCCs when entering a specific country. E.g., such rules specify the expiration date of vaccination-DCCs, test-DCCs or define which vaccines are allowed. The "EU-rules" are retrieved from the central EU Gateway.
		 - **AT**: Austria also publishes the national rules for its validation apps, those rules are not uploaded to the EU Gateway but can be used by Austrian Apps to verify the validity of the QR Code against various profiles (e.g., entry tests for restaurants).
	 - **Value sets**: This data contains all the information on the mapping of raw values to human-readable values. E.g., the QR-code stores the value "EU/1/20/1528" for the vaccine. By using the value set this can be mapped to the vaccine "Comirnaty" where BioNTech Manufacturing GmbH is the marketing authorisation holder. Those value sets are kept up2date.
	 - **Profiles**: It is planned to publish the meta information for the profiles (e.g. entry tests) in the near future. This meta information includes profile names, links to the respective AT rules, logos, descriptions in English/German etc. The idea is to provide the profiles for automated processing in the validation apps to keep the number of required updates at a minimum. More information will be added here as soon as those profiles are available.
 - **Basic Validation**: This represents the validation component, which verifies the structural correctness and the authenticity of a DCC.
 - **Business Rules Validation**: This component is used to validate the high level business rules against the DCC (e.g., how long is a vaccine accepted in a member state, which vaccines are accepted, or how long a certain test ist considered as valid.
 


## Basic Validation
Austria started early to work together with the EU task force to create validation cores for multiple platforms, which validate the structural correctness and authenticity of the QR-codes. The source code of these cores is published with the [Apache 2 license](https://www.apache.org/licenses/LICENSE-2.0) to allow for a flexible use. The code was published very early by Austria in the EHN repositories, but soon an active community has evolved, which provided important feedback and contributions. The code is used by multiple validation apps/creation chains. The AT-libraries offer functions to validated and parse the Austrian information on trust lists/business rules/value sets. The following overview provides an inventory of the published code and its purpose. All of these libraries are maintained and kept up2date:

 - **Kotlin-based creation/validation core**: https://github.com/ehn-dcc-development/hcert-kotlin
	* This repository contains a multi-platform Kotlin project that provides JavaScript/JVM libs
		* for **creating the Austrian DCCs** (JVM) (used by the backend service for creating the Austrian DCCs)
		* for **validating the DCCs** for native Android (Kotlin/JVM) or Browser-based apps (JavaScript): The JavaScript library is used by the Austrian validation apps https://qr.gv.at, https://greencheck.gv.at (and its native counter part on Android/iOS) and the BRZ wallet apps (for checking the codes internally).
        * for downloading/generating the Austrian trust list/business rule list/value sets
        * for the test-runner which validates the correctness of the validation library against the test-data of all member states (including corner cases) provided here: https://github.com/eu-digital-green-certificates/dgc-testdata 
 -   **Swift Validation core for native iOS**: https://github.com/ehn-dcc-development/ValidationCore: This library is used by the native app used by the Austrian police and the BRZ wallet app for validating the code internally.

## Business rules validation
Business rules allow to determine, if a person holding a DCC would be allowed to enter another country based on their vaccination, recovery or test status. Those rules are defined by each country, distributed via the EU Gateway and can be checked by rule engines, e.g. implemented in the Verifier Apps or Wallets.

The rules file provided by the Austrian implementations (see below) offer two categories of rules:
 - **Global rules** that apply when entering Austria; those rules are also uploaded to the EU-Gateway. Within the provided file, they are stored with the country ID "AT" and an empty "region" field.
 - **National rules** that only apply in Austria for the various profiles. Within the provided file, those rules are stored with the country ID "AT" and profile IDs that are kept within the "region" field. The profile ID is defined as follows: **Type[-State[-MD]]**
      - The **Type** of the rule defines
           - **ET**: rules for Eintrittstest/entry test
           - **BG**: rules for Berufsgruppen/Special Occupation
           - **NG**: rules for Nachtgastronomie/night clubs
      - Federal **State** (Bundesland) defines, where the rule applies: BGLD, KTN, NOE, OOE, SBG, STMK, T, VBG, W
      - The **MD** (METADATA) part of the Region field indicates, that the rule is not a CERTLOGIC rule (which returns true or false). It is a rule with the same logic as the corresponding CERTLOGIC rule but it returns a date indicating until when it is valid. This date can e.g. be displayed in wallet apps. If such a feature is not needed, the MD rules can be ignored. If a rule is a MD rule, the Engine field is set to “AT-METADATA”.

Detailed information on rules engines and test data:
 - The specification, examples and reference implementations (especially JS) can be found here: https://github.com/ehn-dcc-development/dgc-business-rules
 - Rules and test data of member states: https://github.com/eu-digital-green-certificates/dgc-business-rules-testdata: The definition of rules from various contries in CertLogic format can be viewed here.
 - Implementations of JsonLogic for different platforms:
	 - iOS implementation: https://github.com/eu-digital-green-certificates/dgc-certlogic-ios
	 - Android implementation in Kotlin: https://github.com/eu-digital-green-certificates/dgc-certlogic-android
 - To check all rules in the testdata repository automatically against different DCC payloads, use https://dcc-crosscheck.vercel.app/

## Details on trust lists/business rules/value sets

 - **Austrian acceptance system for testing**:
	 - **Trust list** (test signature certificates from the EU are included, codes from https://github.com/eu-digital-green-certificates/dcc-quality-assurance can be validated, MUST NOT be used for productive apps)
		 - **Content**: https://dgc-trusttest.qr.gv.at/trustlist
		 - **Signature**: https://dgc-trusttest.qr.gv.at/trustlistsig
	 - **Business rules**:
		 - **Content**: https://dgc-trusttest.qr.gv.at/rules
		 - **Signature**: https://dgc-trusttest.qr.gv.at/rulessig
	 - **Value Sets**:
		 - **Content**: https://dgc-trusttest.qr.gv.at/valuesets 
		 - **Signature**: https://dgc-trusttest.qr.gv.at/valuesetssig 

 - **Austrian productive system**:
	 - **Trust list** (productive codes from the EU and Austria can be validated, MUST be used in productive apps)
		 - **Content**: https://dgc-trust.qr.gv.at/trustlist
		 - **Signature**: https://dgc-trust.qr.gv.at/trustlistsig
	 - **Business rules**:
		 - **Content**: https://dgc-trust.qr.gv.at/rules
		 - **Signature**: https://dgc-trust.qr.gv.at/rulessig
	 - **Value Sets**:
		 - **Content**: https://dgc-trust.qr.gv.at/valuesets 
		 - **Signature**: https://dgc-trust.qr.gv.at/valuesetssig 

The provided files are based on the COSE/CBOR standards which are also used for the DCCs, whereby the "content" file contains the CBOR-encoded data (rules, value sets, trust list data) and the "signature" file represents a signed COSE/CWT structure, which contains the hash-value of the respective "content" file. The Austrian validation cores provide functions to validate the files and extract the content.

Signature certificates which are used to signed the structures (required for validation):
**Acceptance system** (lists hosted on https://dgc-trusttest.qr.gv.at)
```
-----BEGIN CERTIFICATE-----
MIIB6zCCAZGgAwIBAgIKAXmEuohlRbR2qzAKBggqhkjOPQQDAjBQMQswCQYDVQQG
EwJBVDEPMA0GA1UECgwGQk1TR1BLMQowCAYDVQQLDAFRMQwwCgYDVQQFEwMwMDEx
FjAUBgNVBAMMDUFUIERHQyBDU0NBIDEwHhcNMjEwNTE5MTMwNDQ3WhcNMjIwNjE5
MTMwNDQ3WjBRMQswCQYDVQQGEwJBVDEPMA0GA1UECgwGQk1TR1BLMQowCAYDVQQL
DAFRMQ8wDQYDVQQFEwYwMDEwMDExFDASBgNVBAMMC0FUIERHQyBUTCAxMFkwEwYH
KoZIzj0CAQYIKoZIzj0DAQcDQgAE29KpT1eIKsy5Jx3J0xpPLW+fEBF7ma9943/j
4Z+o1TytLVok9cWjsdasWCS/zcRyAh7HBL+oyMWdFBOWENCQ76NSMFAwDgYDVR0P
AQH/BAQDAgeAMB0GA1UdDgQWBBQYmsL5sXTdMCyW4UtP5BMxq+UAVzAfBgNVHSME
GDAWgBR2sKi2xkUpGC1Cr5ehwL0hniIsJzAKBggqhkjOPQQDAgNIADBFAiBse17k
F5F43q9mRGettRDLprASrxsDO9XxUUp3ObjcWQIhALfUWnserGEPiD7Pa25tg9lj
wkrqDrMdZHZ39qb+Jf/E
-----END CERTIFICATE-----
```

**Productive system** (lists hosted on https://dgc-trust.qr.gv.at)

```
-----BEGIN CERTIFICATE-----
MIIB1DCCAXmgAwIBAgIKAXnM+Z3eG2QgVzAKBggqhkjOPQQDAjBEMQswCQYDVQQG
EwJBVDEPMA0GA1UECgwGQk1TR1BLMQwwCgYDVQQFEwMwMDExFjAUBgNVBAMMDUFU
IERHQyBDU0NBIDEwHhcNMjEwNjAyMTM0NjIxWhcNMjIwNzAyMTM0NjIxWjBFMQsw
CQYDVQQGEwJBVDEPMA0GA1UECgwGQk1TR1BLMQ8wDQYDVQQFEwYwMDEwMDExFDAS
BgNVBAMMC0FUIERHQyBUTCAxMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEl2tm
d16CBHXwcBN0r1Uy+CmNW/b2V0BNP85y5N3JZeo/8l9ey/jIe5mol9fFcGTk9bCk
8zphVo0SreHa5aWrQKNSMFAwDgYDVR0PAQH/BAQDAgeAMB0GA1UdDgQWBBRTwp6d
cDGcPUB6IwdDja/a3ncM0TAfBgNVHSMEGDAWgBQfIqwcZRYptMGYs2Nvv90Jnbt7
ezAKBggqhkjOPQQDAgNJADBGAiEAlR0x3CRuQV/zwHTd2R9WNqZMabXv5XqwHt72
qtgnjRgCIQCZHIHbCvlgg5uL8ZJQzAxLavqF2w6uUxYVrvYDj2Cqjw==
-----END CERTIFICATE-----
```

**Information on the format used by the files**:
For examples, and the source code for generating and validating the CBOR structures we refer to the https://github.com/ehn-dcc-development/hcert-kotlin repository.

The content file of the trust list is a CBOR structure. Relevant information on how to validate a DCC and the purpose of the KID can be found here: https://github.com/ehn-dcc-development/hcert-spec/blob/main/hcert_spec.md We can define the schema loosely in this way:

```
TRUSTLIST := {
  c := [{
    c := the bytes of the X.509 encoded certificate
    i := the bytes of the KID (first 8 bytes of SHA-256 digest of the certificate)
  }]
}

```

The content file of the Business Rules is a CBOR structure. Further information on the business rules format can be found in the following EU repositories: https://github.com/ehn-dcc-development/dgc-business-rules and https://github.com/eu-digital-green-certificates/dgc-business-rules-testdata
We can define the schema loosely in this way:

```
BUSINESSRULES := {
  r := [{
    i := the identifier of one rule as a String
    r := the raw JSON of one rule as a String
  }]
}

```


The content file of the Value Sets is a CBOR structure. Details on the values sets can be found here: https://github.com/ehn-dcc-development/ehn-dcc-valuesets
We can define the schema loosely in this way:

```
VALUESETS := {
  v := [{
    n := the name of the value set as a String
    v := the raw JSON of one value set as a String
  }]
}

```

The signature file contains a COSE Sign1Message with a CWT like this:

```
CWT := {
  NOT_BEFORE (5) := timestamp (seconds since UNIX epoch)
  EXPIRATION (4) := timestamp (seconds since UNIX epoch)
  SUBJECT (2)    := bytes of the SHA-256 digest of the content file
}

```

An (non-normative) example of the signature file is:

```
d28450a3182a020448f7b1e7dd8e75cc120126a0582ca30258201a10e4cab0f604840cfd52cb6c0f
423cf19de9d6c2cb0359583d97fe200f7f24041a0002a300050058400204f76cd98082adaaebe623
e7932d203586b18b38d13ea81cf8811e787d8812d753d51d465edc8a3044b8ba82e24083b7535fd0
ce3306f52665871b69923cd1

```

... decoded to the following COSE structure:

```
18([
  h'A3182A020448F7B1E7DD8E75CC120126',
  {},
  h'A30258201A10E4CAB0F604840CFD52CB6C0F423CF19DE9D6C2CB0359583D97FE200F7F24041A
    0002A3000500',
  h'0204F76CD98082ADAAEBE623E7932D203586B18B38D13EA81CF8811E787D8812D753D51D465E
    DC8A3044B8BA82E24083B7535FD0CE3306F52665871B69923CD1'
])

```

... with this protected header:

```
{
  42: 2,                  // the version number of the format
  4: h'F7B1E7DD8E75CC12', // the KID of the signing certificate
  1: -7,                  // the key type of the signing certificate, i.e. EC
}

```

... and this CBOR content:

```
{
  2: h'1A10E4CAB0F604840CFD52CB6C0F423C
       F19DE9D6C2CB0359583D97FE200F7F24', // bytes of the SHA-256 digest of the content file
  5: 1619788643,                          // timestamp, after which the signature is valid
  4: 1619961443                           // timestamp, before which the signature is valid
}
```

# Additional links (demo, demo-apps)
* **Demo Service, that generates test codes (based on the Kotlin creation core)**: https://dgc.a-sit.at/ehn/

* **Demo Apps for Android and iOS**: Disclaimer: These are only demonstration apps that where used for testing and evaluation purposes, they do not represent the final output for the Austrian validation apps, which need to consider the legal requirements by Austrian law. The demo apps will not be published in the App Stores and will not be used for the validation of real codes. The iOS app is based on the iOS validation core, the Android app uses the Kotlin core:
	* https://github.com/ehn-dcc-development/hcert-app-kotlin
	* https://github.com/ehn-dcc-development/hcert-app-swift

# Accessing "trust list", "business rules" and "value sets"

In order to operate verifier and/or wallet apps, data from the “trust list”, "business rules" and “value sets”  is required. This data can be loaded via the URLs listed above. Please note the "fair-use policy": As the data in the lists changes quite rarely (maximum once per day) a polling interval of e.g. 4 hours is more than enough.

# Further repositories
* Green Pass App (Android and iOS, OpenSource): https://github.com/BRZ-GmbH/

# Legal/Impressum

[Bundesministerium für Soziales, Gesundheit, Pflege und Konsumentenschutz](https://www.sozialministerium.at/)  
Stubenring 1  
1010 Wien  
Telefon: +43 1 71100 – 0  
Fax: +43 1 7158258  

The validation source code is being developed by [A-SIT Plus](https://www.a-sit.at/kooperationen/a-sit-plus/). The code of other Austrian contributors will be added in the near future (e.g. Web App UI). Due to the open source nature, contributions of others are included as well, the code for the validation core is also used by others. All source code is published under the [Apache 2 license].(https://www.apache.org/licenses/LICENSE-2.0). 

