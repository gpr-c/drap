# DRAP: Digital Rights Access Point

See the Operational Transparency Notice demo at [transparencylab.ca/notice](https://transparencylab.ca/notice), and try out the Digital Rights Access Point at [transparencylab.ca/notice#rights](https://transparencylab.ca/notice#rights).

A Digital Rights Access Point is the single public place where a person exercises their rights over what a controller holds about them, using the receipt number they were given. No account is needed and no identification is asked for. Every request gets a rights request receipt of its own.

**Live demonstration:** [transparencylab.ca/notice#rights](https://transparencylab.ca/notice#rights), the rights section of the Transparency Lab Operational Transparency Notice

**Specification:** [DRAP v1](DRAP-v1.md)

## What v1 does

- One receipt field, and a checkbox for each right: see what is held (access), correct something (rectification), delete it (erasure), stop a use of it (object), withdraw consent.
- An optional reply address. Leave it empty to stay anonymous.
- A rights request receipt (`tl-rr-...`) for every request, with the outcome or status of each right, and a status check by that receipt.
- Where consent is the basis, withdrawal, objection and erasure take effect at once and are recorded as authorization state changes against the notice version.

## Machine-readable records

The demonstration controller declares its access point in its records:

- Controller Identification Record: [/.well-known/transparency/cir.json](https://transparencylab.ca/.well-known/transparency/cir.json) (`privacy_access_point` and `drap`)
- Rights record: [/.well-known/transparency/rights.json](https://transparencylab.ca/.well-known/transparency/rights.json)
- Notice Event Log: [/.well-known/transparency/nel.json](https://transparencylab.ca/.well-known/transparency/nel.json)
- DID document: [did:web:transparencylab.ca](https://transparencylab.ca/.well-known/did.json)

## Relationship to standards

DRAP is a companion to the ANCR Extension for ISO/IEC TS 27560:2023, Release 1. It implements non-exclusion (criterion C7) and the rights in the consent row of Annex C. It is not part of the Extension and is not endorsed by any standards body.

## Status

Version 1, live since 7 October 2026 at Transparency Lab, a Global Privacy Rights lab. Global Privacy Rights is the publisher.

## Licence

This specification is published under [Creative Commons Attribution 4.0](https://creativecommons.org/licenses/by/4.0/).
