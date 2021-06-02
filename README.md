

# Austrian implementation of the EU Digital COVID Certificates

This overview points to the relevant Austrian/EU repositories that contain the specifications for the EU Digital COVID Certificates and the source code of the Austrian implementation. This page and the repository structure are work in progress and will be updated continuously.

version overview
|Date (version)|Description |
|--|--|
| 02.06.2021 | initial version |


**General repositories**

* EHN repositories for various open source projects, which contain components for QR generation/verification or even complete creation/validation chains. Initially, all Austrian components are published at this place, since there is a lively exchange and collaboration of developers: https://github.com/ehn-digital-green-development
* Repositories for reference implementations provided by the EU: https://github.com/eu-digital-green-certificates/
* Specification for creating the QR codes: https://github.com/ehn-digital-green-development/hcert-spec/blob/main/hcert_spec.md

**Austrian source code**

All Austrian source code will be forked/transferred to this place in the next weeks. The source code is published with the [Apache 2 license](https://www.apache.org/licenses/LICENSE-2.0) to allow for a flexible use. For now, the code is published and maintained in the EHN repositories. The following overview provides an inventory of the published code and its purpose:

* **Kotlin-based creation/validation core**: 
	* https://github.com/ehn-digital-green-development/hcert-kotlin
	* This repository contains a multi-platform Kotlin project that provides JavaScript/JVM libs
		* for creating the Austrian QR-Codes (JVM)
		* for validating the EU QR-Codes (JVM/JavaScript) for native Android (Kotlin/JVM) or Browser-based apps (JavaScript)
        * for generating the Austrian trust list (which distributes the signature certificates (DSCs) of the other member states to the validation apps)
        * for the test-runner which validates the correctness of the validation library against the test-data of all member states (including corner cases) provided here: https://github.com/eu-digital-green-certificates/dgc-testdata 
*   **Demo Service, that generates test codes (based on the Kotlin creation core)**: https://dgc.a-sit.at/ehn/
*   **Swift Validation core for native iOS**: https://github.com/ehn-digital-green-development/ValidationCore
*   **Demo Apps for Android and iOS**: Disclaimer: These are only demonstration apps that where used for testing and evaluation purposes, they do not represent the final output for the Austrian validation apps, which need to consider the legal requirements by Austrian law. The demo apps will not be published in the App Stores and will not be used for the validation of real codes. The iOS app is based on the iOS validation core, the Android app uses the Kotlin core:
	* https://github.com/ehn-digital-green-development/hcert-app-kotlin
	* https://github.com/ehn-digital-green-development/hcert-app-swift

**Further plans**

The JavaScript library provided by the Kotlin multiplatform project will be used for the Austrian Web Apps for QR validation. The source code of this Web Apps will also be published. All validation apps adhere to the privacy principles: Validation of the qr-codes must be done offline (for the Web App: all the code must be present on the client), online verification is not allowed due to the possibility of user tracing. Online requests by the apps are limited to retrieving information on the trusted signature certificates of other member states and later revocation lists.

**Legal/Impressum**

[Bundesministerium für Soziales, Gesundheit, Pflege und Konsumentenschutz](https://www.sozialministerium.at/)  
Stubenring 1  
1010 Wien  
Telefon: +43 1 71100 – 0  
Fax: +43 1 7158258  

The current source code is being developed by [A-SIT Plus](https://www.a-sit.at/kooperationen/a-sit-plus/). The code of other Austrian contributors will be added in the near future (e.g. Web App UI). Due to the open source nature, contributions of others are included as well, the code for the validation core is also used by others. All source code is published under the [Apache 2 license].(https://www.apache.org/licenses/LICENSE-2.0). 
