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
| 16.11.2021 | Added information for AT business rules and Engine "AT-METADATA" |
| 22.03.2022 | Added information about Introduction of Simple Business Rule Format |
| 03.06.2022 | Updated the Certificates used to sign the downloadable structures |
| 07.12.2022 | Added information about Introduction of New Business Rule Format (and removed the Simple Business Rule Format) |

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
MIIB6zCCAZGgAwIBAgIKAYDM2HHZBVwwtTAKBggqhkjOPQQDAjBQMQswCQYDVQQG
EwJBVDEPMA0GA1UECgwGQk1TR1BLMQowCAYDVQQLDAFRMQwwCgYDVQQFEwMwMDIx
FjAUBgNVBAMMDUFUIERHQyBDU0NBIDIwHhcNMjIwNTE2MTIyOTM4WhcNMjMwNjE2
MTIyOTM4WjBRMQswCQYDVQQGEwJBVDEPMA0GA1UECgwGQk1TR1BLMQowCAYDVQQL
DAFRMQ8wDQYDVQQFEwYwMDIwMDIxFDASBgNVBAMMC0FUIERHQyBUTCAyMFkwEwYH
KoZIzj0CAQYIKoZIzj0DAQcDQgAE29KpT1eIKsy5Jx3J0xpPLW+fEBF7ma9943/j
4Z+o1TytLVok9cWjsdasWCS/zcRyAh7HBL+oyMWdFBOWENCQ76NSMFAwDgYDVR0P
AQH/BAQDAgeAMB0GA1UdDgQWBBQYmsL5sXTdMCyW4UtP5BMxq+UAVzAfBgNVHSME
GDAWgBRsSZFrO9SANI2CSK201pEayDltvTAKBggqhkjOPQQDAgNIADBFAiBToWg7
aGFDKcahC/dT/y5Fq1AQjQ0MyR5eZfydwzayNAIhAJwIWbWaF8bz6nIkoRrVgdf1
rgURIBDJ4WO02mZCVfLu
-----END CERTIFICATE-----
```

**Productive system** (lists hosted on https://dgc-trust.qr.gv.at)

```
-----BEGIN CERTIFICATE-----
MIIB1DCCAXmgAwIBAgIKAYDcOWBmNxlPgDAKBggqhkjOPQQDAjBEMQswCQYDVQQG
EwJBVDEPMA0GA1UECgwGQk1TR1BLMQwwCgYDVQQFEwMwMDIxFjAUBgNVBAMMDUFU
IERHQyBDU0NBIDIwHhcNMjIwNTE5MTIwOTQ5WhcNMjMwNjE5MTIwOTQ5WjBFMQsw
CQYDVQQGEwJBVDEPMA0GA1UECgwGQk1TR1BLMQ8wDQYDVQQFEwYwMDIwMDIxFDAS
BgNVBAMMC0FUIERHQyBUTCAyMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEl2tm
d16CBHXwcBN0r1Uy+CmNW/b2V0BNP85y5N3JZeo/8l9ey/jIe5mol9fFcGTk9bCk
8zphVo0SreHa5aWrQKNSMFAwDgYDVR0PAQH/BAQDAgeAMB0GA1UdDgQWBBRTwp6d
cDGcPUB6IwdDja/a3ncM0TAfBgNVHSMEGDAWgBQvWRbxO3tS9HatiMTvp8sD9Rwy
wTAKBggqhkjOPQQDAgNJADBGAiEAleZ8CcLG4FK4kty+sN0APZmT6LfEE2kzznyV
yEepU0gCIQCGaqJpOwPXBmgoOsehnJkA0+TZX8V2p1Bg/nqnuYqXFg==
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

## New Modern Business Rule Format
### Motivation
Due to several reasons we introduce a new format for representing the business rules which determine whether a green pass certificate is valid or not:

-   Readability and Maintainability of the "old" (current) EU format is extremely difficult and prone to errors, because
    -   for each federal state and each profile (currently Entry and Night Club) dozens of files with only minimal differences need to be maintained
    -   the actual rules are split across multiple files
    -   the actual evaluation rules (in JSONLogic) need to be redefined in each file again which is highly prone to errors
-   The current format is highly inefficient. As of May 2022 the business rules in the old format are about 640 KB even though there are almost no restrictions in place and the new format requires less than 30KB for the same rules.
-   The current format is not very flexible. Each new profile (e.g. Theater, 2G, Workplace, etc) means copying hundreds of files and applying only minimal changes.
-   The metadata (until when is a certificate valid) is completely separate from the actual rules in the current format.
### Features of the new format
The new format offers several features and improvements which lead to a much simpler maintenance and greatly improved readability:

-   Flexible definition of profiles (Entry, Night Club)
-   Predefinition and Reuse of JSON Logic Conditions
-   Easy definition for which federal states a rules is applicable (through white and blacklisting)
-   Reuse of a complete set of rules (if the same rules apply to multiple profiles, the rules have to be defined only once)
-   Defintion of groups (e.g. separate by age)
-   Definition of metadata (valid from/until) directly within the rules
-   Support for extermal Conditions (Preparation to include other certificates to determine the validity of a certificate)
### URLs
-   **Austrian acceptance system for testing**:
    -   **Content**: [https://dgc-trusttest.qr.gv.at/extendedrulesbin](https://dgc-trusttest.qr.gv.at/extendedrulesbin)
    -   **Signature**: [https://dgc-trusttest.qr.gv.at/extendedrulessig](https://dgc-trusttest.qr.gv.at/extendedrulessig)
-   **Austrian productive system**:
    -   **Content**: [https://dgc-trust.qr.gv.at/extendedrulesbin](https://dgc-trust.qr.gv.at/extendedrulesbin)
    -   **Signature**: [https://dgc-trust.qr.gv.at/extendedrulessig](https://dgc-trust.qr.gv.at/extendedrulessig)
## Current Example
[Attached](new_business_rules_11052022.json) is the file in the new business rule format which represents the valid rules for Entry and Night Club as of May 11, 2022.
## Payload

### Full Payload
```
{
  "profiles": [
    { profile payload }
  ],
  "conditions": {
    "condition": {
      condition payload
    }
  }
  "rules" [
    { rule payload }
  ]
}
```
### Profile Payload
```
{
  "id": string,
  "name": {
    "de": string,
    "en": string
  },
  "links": {
    "W": string,
    "all": string,
  }
}
```
### Condition Payload
```
{
  "logic": "json logic string",
  "violation_description": {
    "de": string,
    "en": string
  }
}
```
### Rule Payload
```
{
  "id": string,
  "schema_version": number,
  "regions": {
    "include": [ list of region strings ]
    "exclude": [ list of region strings ]
  },
  "valid_from": timestamp string,
  "valid_until": timestamp string,
  "certificate_type": "vaccination|test|recovery|vaccination_exemption",
  "certificate_type_conditions": [
    list of condition names - linked with AND
  ],
  "general_conditions": [
    list of condition names - linked with AND
  ],
  "groups": {
    "group_name": [
      [
        first list of condition names - linked with AND
      ],
      [
        second list of condition names - linked with AND - linked to first list with OR
      ]
    ]
  },
  "profiles": {
    "profile_id" {
      "all|group_name": {
        "conditions": [
          [
            first list of condition names - linked with AND
          ],
          [
            second list of condition names - linked with AND - linked to first list with OR
          ]
        ],
        "valid_from": [
          {
            validity payload
          },
          {
            validity payload
          }
        ]
         "valid_until": [
          {
            validity payload
          },
          {
            validity payload
          }
        ],
        "invalid": boolean,
        "equal_to_profile": profile name,
        "linked_conditions": [
          {
            "violation_description": {
              "de": string,
              "en": string
            }
            "conditions": [
              list of condition names - linked with AND
            ]
          },
          {
             "violation_description": {
              "de": string,
              "en": string
            }
            "conditions": [
              list of condition names - linked with AND
            ]
           }
        ]
      }
    }
  }
}
```
### Validity Payload
```
{
  "value": "placeholder from certificate value"
  "plus_unit": "minute|hour|day|month",
  "plus_value": number,
  "format": "date|dateTime",
  "max": timestamp string,
  "conditions": [
    [
      first list of condition names - linked with AND
    ],
    [
      second list of condition names - linked with AND - linked to first list with OR
    ]
  ],
  "modifier": "startOfDay|endOfDay|startOfMonth|endOfMonth"
}
```
## Field / Payload Descriptions
### profiles

List of "profiles". A profile defines an area that defines a separate set of rules for the validity of certificates. This can be either a G-rule (e.g., 1G, 2G, 3G) or a certain physical area (e.g. restaurant, theater, night club).

### Profile Payload
```
{
  "id": string,
  "name": {
    "de": string,
    "en": string
  },
  "links": {
    "W": string,
    "all": string,
  }
}
```
-   id (mandatory): unique key for this profile
-   name (mandatory): Display name for the profile. Germand and Englisch localization are defined in the subelements de and en.
-   links (optional): links for additional information about the profile. Specific links for individual federal states can also be defined, and a fallback for "all" should be definied in all cases if a link is desired.

Here is a sample for two profiles Entry and Night Club:
```
{
  "profiles": [
    {
      "id": "Entry",
      "name": {
        "de": "Eintritt",
        "en": "Entry"
      },
      "links": {
        "W": "https://coronavirus.wien.gv.at/oeffentliches-leben/",
        "all": "https://www.sozialministerium.at/Informationen-zum-Coronavirus/Coronavirus---Aktuelle-Maßnahmen.html"
      }
    },
    {
      "id": "NightClub",
      "name": {
        "de": "Nachtgastronomie",
        "en": "Night Club"
      },
      "links": {
        "W": "https://coronavirus.wien.gv.at/oeffentliches-leben/",
        "NOE": "https://www.noe.gv.at/noe/Coronavirus/Aktuelle_Massnahmen.html",
        "all": "https://www.sozialministerium.at/Informationen-zum-Coronavirus/Coronavirus---Aktuelle-Maßnahmen.html"
      }
    }
  ]
}
```
### conditions

A map with JSONLogic conditions. The keys are identifiers for the conditions which can then be used in the rules later on.
```
{
  "logic": "json logic string",
  "violation_description": {
    "de": string,
    "en": string
  }
}
```
-   violation_description (optional): An error text that can be used when this condition is violated for a certificate. German and Englisch localizations are defined in the subelements de and en.
-   logic (mandatory): JSONLogic condition as string (including the proper string escapes). The condition has to return a boolean value if this condition is valid for a certificate or not. In the old/current business rule format this is the full content of the field "Logic" (converted to a string with proper escapes".

Here is a sample rule that checks if the test result of a test certificate is negative:
```
{
  "conditions": {
    "isNegativeTestResult": {
      "violation_description": {
        "de": "Testresultat ist positiv",
        "en": "Test result is positive"
      },
      "logic": "{\"if\":[{\"var\":\"payload.t.0\"},{\"===\":[{\"var\":\"payload.t.0.tr\"},\"260415000\"]},true,false]}"
    }
  }
}
```
### rules

List of individual rules. All rules that apply to a given certificate based on the certificate_type will be evaluated. For each certificate and federal state only a single rule should apply and the individual should not overlap. This has to be kept in mind by the creator of the rules!

-   id: An internal ID for this rule
-   schema_version: Schema-Version of this rule. Can be used to determine if a given rule can be evaluated at all if changes to the structure or external rules are introduced at a later point in time.
-   federal_states: Defines for which federal states this rule applies.
-   certificate_type: The type of certificate for which this rule applies. Can be one of the following values: test, vaccination, recovery oder vaccination_exemption.
-   certificate_type_conditions: A list of conditions (combined with AND) to check if a rule is relevant for a given certificate. E.g. is a vaccination certificate, contains only one certificate.
-   valid_from: ISO-Date (zB "2021-01-01T22:00:00Z") from which point in time the rule is valid. If the field is not defined, there is no check for the valid_from date and the rule is valid from long ago.
-   valid_until: ISO-Date (zB "2021-01-01T22:00:00Z") until when this rule is valid. If the field is not defined, there is no check for the valid_until date and the rule is valid until forever.
-   general_conditions: A list of conditions combined with AND which all must be evaluated to true for the certificate to be valid. E.g. is an accepted vaccine, the vaccination date is after the validation time, the certificate is no revoked certificate.
-   groups: A map to define distinct groups for which different individual rules apply. These groups can then be used in the conditions for the individual profiles. If no groups are defined an automatic group named "all" is defined and used for this rule. If the definition of the groups does not lead to distinct groups it is undefined which group a given certificate is assigned to.
-   profiles: A map of individual conditions for a single profile. The key is the id of the individual profile.

### rules → federal_states

-   include: A list of short identifiers for federal states for which this rule applies.
-   exclude: A list of short identifiers for federal states, for which this rule does not apply. This list has a higher priority than the include list.

Possible short identifiers for federal states are:

-   **BGLD** for Burgendland
-   **KTN** for Kärnten
-   **NOE** for Niederösterreich
-   **OOE** for Oberösterreich
-   **SBG** for Salzburg
-   **STMK** for Steiermark
-   **T** for Tirol
-   **VBG** for Vorarlberg
-   **W** for Wien
-   **all** for all federal states

### rules → groups

Defines groups by specifying a list of conditions. For each group you can define a list of lists of conditions. The conditions in the inner list are combined with AND, while the individual lists are combined with OR.

The following shows the definition of two groups - for people under and over 18, including the required conditions to check for the age.
```
{
  "profiles": [
    {
      "id": "Entry"
    },
    {
      "id": "NightClub"
    }
  ],
  "conditions": {
    "isYoungerThan18Years": {
      "logic": "{\"if\":[{\"after\":[{\"plusTime\":[{\"var\":\"payload.dob\"},18,\"year\"]},{\"plusTime\":[{\"var\":\"external.validationClock\"},0,\"day\"]}]},true,false]}"
    },
    "isOlderThan18Years": {
      "logic": "{\"if\":[{\"not-after\":[{\"plusTime\":[{\"var\":\"payload.dob\"},18,\"year\"]},{\"plusTime\":[{\"var\":\"external.validationClock\"},0,\"day\"]}]},true,false]}"
    }
  },
  "rules": [
    {
      "id": "rule-1",
      "groups": {
        "ageUpTo18": [
          [
            "isYoungerThan18Years"
          ]
        ],
        "age18AndOlder": [
          [
            "isOlderThan18Years"
          ]
        ]
      },
      "profiles": {
        "Entry": {
          "ageUpTo18": {
          },
          "age18AndOlder": {
          }
        },
        "NightClub": {
          "ageUpTo18": {
            "equal_to_profile": "Entry"
          },
          "age18AndOlder": {
            "equal_to_profile": "Entry"
          }
        }
      }
    }
  ]
}
```
If groups are defined for any given rule, the groups must then also be used when defining the rules for the profiles. A mixture of using the implicit group "all" and defined groups is not possible.

Additionally when using the equal_to_profile field to reference and reuse the rules of a different profile, all groups must be used and defined for that profile as well (see Night Club in the sample above).

### rules → profiles (einzelner Eintrag)

Defines the conditions and validity for certificate in a profile. Each entry is a map with keys for the individual groups (either "all" if no groups are defined or each individual group)
-   invalid: Can be set to true if the evaluated certificate is invalid for this profile/group.
-   valid_from: See detailed description rules -> profiles -> details -> valid_from/_until
-   valid_until: See detailed description unter rules -> profiles -> details -> valid_from/_until
-   equal_to_profile: Can be used if this profile equals another profile (in this rule!). The rules for the referenced profile must have the same structure, e.g. they must also define the same groups. If any of the other fields are defined in addition to this one, those fields overwrite the values defined in the referenced profile.
-   linked_conditions: A list of external conditions which can only be evaluated by the containing application. This can be used to define dependencies between certificates, like certificate A is only valid if the user has a a valid certificate B for this profile. See external conditions for a detailed description of conditions that the app can support at the moment. The evaluated conditions (matching and failed) are explicitly provided when evaluating so that the app can provide special functionality to e.g. link to another certificate.
-   conditions: The conditions for this profile. Defined as a list of lists of conditions. The conditions in the inner list are combined with AND, while the individual lists are combined with OR.

The conditions are defined on two levels to support AND as well as OR combinations of conditions. The first level is used for combining groups of conditions with OR (= one of the groups of conditions has to evaluate to true). The second level contains names/keys of conditions which are defined at the root level of the new business rule format under conditions. The conditions on this second level are combined with AND (= all the conditions have to evaluate to true)

Following is a sample of the technical representation and a plain-text description afterwards:
```
"conditions": [
  [
    "bedingung_eins",
    "bedingung_zwei"
  ],
  [
    "bedingung_drei",
    "bedingung_vier",
    "bedingung_fuenf"
  ]
]
```
This entry is used when either bedingung_eins and bedingung_zwei are evaluted to true OR if bedingung_drei and bedingung_vier and bedingung_fuenf are evaluated to true.

### rules → profiles → details → valid_from / valid_until

The valid_from and valid_until block for a profile define the validity to display to the user. It is possible to define multiple validity payload for valid_until and valid_from which contain different conditions. When evaluating the rules, all the validities are evaluated and return to the application and the application needs to determine which is the correct one.
```
{
  "value": "iso date or placeholder from certificate value"
  "plus_unit": "minute|hour|day|month",
  "plus_value": number,
  "format": "date|dateTime",
  "max": timestamp string,
  "conditions": [
    [
      first list of condition names - linked with AND
    ],
    [
      second list of condition names - linked with AND - linked to first list with OR
    ]
  ],
  "modifier": "startOfDay|endOfDay|startOfMonth|endOfMonth"
}
```
-   value (mandatory): Either an ISO-Date (eg "2021-01-01T22:00:00Z") or a placeholder referencing a value from the certificate. The placeholder has to be surrounding by #, e.g. #payload.v.0.dt# for the date of vaccination or e.g. #[payload.t.0.sc#](http://payload.t.0.sc) for the date of sample collection of a test.
-   plus_unit (optional): A time unit - plus_unit * plus_interval will be added to the value. Optional and can be omitted. Possible values are minute, hour, day, month
-   plus_interval (optional): A number value - plus_unit * plus_interval will be added to the value. Optional and can be omitted.
-   max (optional): An ISO-Date (eg "2021-01-01T22:00:00Z"), which will be used as the maximum date for the validity, even if the value of + plus_unit * plus_interval results in a later date.
-   format (optional): An optional format who the resolved date should be formatted. (date when formatting which only day, month, year or dateTime when formatting with date including hour and day).
-   modifier (optional): Optional Modifier to modify the resolved date.
-   conditions (optional): A list of conditions that have to resolve to true for this entry to be used and returned. If conditions are empty or omitted, the entry always applies or can be used as a fallback.

  

The conditions are defined on two levels to support AND as well as OR combinations of conditions. The first level is used for combining groups of conditions with OR (= one of the groups of conditions has to evaluate to true). The second level contains names/keys of conditions which are defined at the root level of the new business rule format under conditions. The conditions on this second level are combined with AND (= all the conditions have to evaluate to true)

Following is a sample of the technical representation and a plain-text description afterwards:
```
"conditions": [
  [
    "bedingung_eins",
    "bedingung_zwei"
  ],
  [
    "bedingung_drei",
    "bedingung_vier",
    "bedingung_fuenf"
  ]
]
```
This entry is used when either bedingung_eins and bedingung_zwei are evaluted to true OR if bedingung_drei and bedingung_vier and bedingung_fuenf are evaluated to true.

# Additional links (demo, demo-apps)
* **Demo Service, that generates test codes (based on the Kotlin creation core)**: https://dgc.a-sit.at/ehn/

* **Demo Apps for Android and iOS**: Disclaimer: These are only demonstration apps that where used for testing and evaluation purposes, they do not represent the final output for the Austrian validation apps, which need to consider the legal requirements by Austrian law. The demo apps will not be published in the App Stores and will not be used for the validation of real codes. The iOS app is based on the iOS validation core, the Android app uses the Kotlin core:
	* https://github.com/ehn-dcc-development/hcert-app-kotlin
	* https://github.com/ehn-dcc-development/hcert-app-swift

# Accessing "trust list", "business rules" and "value sets"

In order to operate verifier and/or wallet apps, data from the “trust list”, "business rules" and “value sets”  is required. This data can be loaded via the URLs listed above. Please note the "fair-use policy": As the data in the lists changes quite rarely (maximum once per day) a polling interval of e.g. 4 hours is more than enough. Clients may be blocked for any violation of this policy.

# Further repositories
* Green Pass App (Android and iOS, OpenSource): https://github.com/BRZ-GmbH/

# Legal/Impressum

[Bundesministerium für Soziales, Gesundheit, Pflege und Konsumentenschutz](https://www.sozialministerium.at/)  
Stubenring 1  
1010 Wien  
Telefon: +43 1 71100 – 0  
Fax: +43 1 7158258  

The validation source code is being developed by [A-SIT Plus](https://www.a-sit.at/kooperationen/a-sit-plus/). The code of other Austrian contributors will be added in the near future (e.g. Web App UI). Due to the open source nature, contributions of others are included as well, the code for the validation core is also used by others. All source code is published under the [Apache 2 license].(https://www.apache.org/licenses/LICENSE-2.0). 

