---
title: "DRAP: Digital Rights Access Point"
subtitle: "Version 1 specification 1.0, and a design note for version 2"
date: 7 October 2026
controller: "Global Privacy Rights (Transparency Lab, transparencylab.ca)"
status: "DRAP v1 implemented and live 7 October 2026 (Transparency Lab notice 1.3.1). Part 2 is a design note, not a specification."
---

# DRAP: Digital Rights Access Point

Part 1 (sections 1 to 11) specifies DRAP v1, rights requests by receipt, as implemented on transparencylab.ca/notice#rights. Part 2 (section 12) records the design constraints for a later version.

# Part 1. DRAP v1: rights requests

## 1. Purpose

A Digital Rights Access Point (DRAP) is the single public place where a person exercises their rights over what a controller holds under a receipt. The receipt is the key. No account is needed and no identification is asked for.

DRAP v1 replaced the "Your rights" section of transparencylab.ca/notice, which offered one withdrawal box. Under DRAP v1, one receipt field leads to a checkbox for each right the controller declares, and every request produces a receipt of its own.

## 2. Terms

| Term | Meaning |
|---|---|
| Receipt | The consent or notice receipt number issued to the person when they gave the information, for example `tl-sa-20261007-3f9c41d2a07b5e18` |
| Rights request | One submission to the DRAP: a receipt plus one or more rights ticked |
| Rights request receipt | The receipt the DRAP issues for a rights request, in the form `tl-rr-YYYYMMDD-` followed by 16 hex characters |
| Automated right | A right the DRAP carries out at once, with no person involved |
| Handled right | A right that a person at the controller answers |

## 3. Requirements

| # | Requirement |
|---|---|
| D1 | **Receipt as key.** A rights request is made with a receipt number alone. The DRAP MUST NOT ask for a name, account or identity document. |
| D2 | **Symmetry.** Withdrawing consent MUST be no harder than giving it. For a receipt the controller holds automatically, ticking "Withdraw my consent" takes effect when the request is sent. |
| D3 | **Every request is receipted.** Each rights request MUST return a rights request receipt that states the receipt it concerns, the notice version, each right asked for, and the outcome or due date for each. |
| D4 | **Outcome for each right.** The result MUST state, right by right, whether it was done, or the date by which it will be acknowledged and answered. |
| D5 | **Minimum personal data.** A reply address is optional. It and any text entered are kept only until the request is closed, then deleted. |
| D6 | **Logged without personal data.** Every request and outcome is entered in a hash-chained, append-only DRAP log that holds receipt numbers, rights and outcomes, and no reply addresses or free text. |
| D7 | **Declared.** The controller record (CIR) and rights record (`rights.json`) MUST point to the DRAP, so it can be found by a person and by a machine. |

This specification sets no response times. A controller running a DRAP answers within the times its own controller record states and the law applicable to it requires.

## 4. Rights in v1

The rights are the five declared in `rights.json`. Each appears on the page in plain words, with the legal term in brackets.

| Right (`rights.json`) | Label on the page | Input needed | Self-assessment receipt (`tl-sa-`) | Notice receipt (`TL-2026-NOTICE-`) |
|---|---|---|---|---|
| `access` | See what is held (access) | None | Automated: the record is shown on screen | Handled: reply needed |
| `rectification` | Correct something (rectification) | Text, required: "What should be corrected?" | Handled | Handled |
| `erasure` | Delete it (erasure) | None | Automated: deleted at once | Handled |
| `object` | Stop a use of it (object) | Text, optional: "Which use?" | Automated: processing stops and the record is deleted (the basis is consent) | Handled |
| `withdraw_consent` | Withdraw my consent | None | Automated: deleted at once | Handled |

**Overlap.** Where the basis is consent, erasure, objection and withdrawal all end in deletion. The person may tick any of them. The DRAP carries out one deletion, and the rights request receipt lists each right ticked against the single action taken.

**Order.** Rights are carried out in this order: access, rectification, object, withdraw consent, erasure. So a person who ticks both "See what is held" and "Delete it" is shown the record before it is deleted.

**Unknown receipts.** A receipt number in neither format is refused with: "That is not a receipt issued by Transparency Lab." No request is logged.

**Handled rights need a way to reply.** If a handled right is ticked and no reply address is given, the page says the answer can be collected with the rights request receipt (section 5.3) and asks whether to continue.

## 5. Interface

### 5.1 The page

The "Your rights" section of /notice, anchor `#rights`, heading "Your rights: digital rights access point". In order:

1. One line: "Use the receipt number you were given. You do not need an account, and you do not have to identify yourself. Keep your receipt private: anyone holding it can use these rights."
2. Receipt number field.
3. "What would you like to do? Tick any that apply." The five checkboxes from section 4, with the text boxes opening only when their right is ticked.
4. Reply address, optional: "Only if you want a written reply. Leave empty to stay anonymous."
5. Send request.
6. Note: "Withdrawal and deletion of a self-assessment take effect immediately. Other requests are answered by a person. You receive a rights request receipt either way."

The result replaces the form: the rights request receipt, a line for each right with "done now" or its due dates, the record itself where access was automated, and a button to copy the rights request receipt.

### 5.2 Request

`POST https://0pn.transparencylab.ca/api/self-assessment/drap/request`

```json
{
  "receipt_id": "tl-sa-20261007-3f9c41d2a07b5e18",
  "notice_version": "1.3.0",
  "rights": ["access", "erasure"],
  "rectification_text": null,
  "object_text": null,
  "reply_email": null
}
```

Response:

```json
{
  "rr_id": "tl-rr-20261007-8c21e0f4b39a7d65",
  "issued_at": "2026-10-07T14:02:11Z",
  "receipt_id": "tl-sa-20261007-3f9c41d2a07b5e18",
  "notice_version": "1.3.0",
  "outcomes": [
    { "right": "access", "status": "done", "done_at": "2026-10-07T14:02:11Z" },
    { "right": "erasure", "status": "done", "done_at": "2026-10-07T14:02:11Z" }
  ],
  "record": { "...": "the record as held, before deletion" }
}
```

A handled right returns `"status": "pending"`, and moves to `"answered"` when a person answers it. A right that does not apply returns `"status": "not_applicable"` with a reason.

### 5.3 Status

`POST https://0pn.transparencylab.ca/api/self-assessment/drap/status` with `{ "rr_id": "tl-rr-..." }` returns the outcomes and, once answered, the answer text. The rights request receipt is sent in the body, never in the address, because it acts as a key.

## 6. Records

| Store | Holds | Kept |
|---|---|---|
| `drap_requests` | rr_id, receipt_id, rights, outcomes, dates, rectification and object text, reply address, answer | Text and reply address deleted when the request closes; the answer kept 30 days for collection; the row deleted 24 months after closing |
| `drap_log` | seq, at, rr_id, receipt_id, rights, outcome, previous hash, entry hash | Permanent, append-only |

No IP address, cookie or browser fingerprint is stored. The legal basis for processing a rights request is legal obligation, and the notice records it as its own purpose. The DRAP runs on the Montreal Lab service, beside the existing withdrawal route, which stays in place for any page that already calls it. Handled requests are answered by a person at the controller, who sees the request text and reply address only until the request is closed.

## 7. Declaration changes

1. **CIR.** `rights_access_point` becomes `https://transparencylab.ca/notice#rights`, plus:
   ```json
   "drap": {
     "version": "DRAP-1",
     "page": "https://transparencylab.ca/notice#rights",
     "request_endpoint": "https://0pn.transparencylab.ca/api/self-assessment/drap/request",
     "status_endpoint": "https://0pn.transparencylab.ca/api/self-assessment/drap/status",
     "rights": ["access", "rectification", "erasure", "object", "withdraw_consent"],
     "identification_required": false
   }
   ```
2. **rights.json.** The same `drap` block. The `object` description becomes "Object to processing. Offered at the top level, not buried." (removes the dash).
3. **Notice.** Version 1.3.0, recorded in the notice event log as a material change (seq 3), because answering rights requests is a new purpose. Version 1.3.1 (seq 4) corrected a script error that stopped the form sending in 1.3.0.

### 7.1 Relationship to the ANCR Extension for ISO/IEC TS 27560:2023, Release 1

DRAP is a companion to the Extension, not part of it. Release 1 asks that the Notice Event Log support hooks for withdrawal, objection and rights exercise events (7.4.3), and leaves additional event types to companion specifications. DRAP implements those hooks as follows.

| Release 1 | How DRAP meets it |
|---|---|
| C7 and 7.1.3, non-exclusion | Each `privacy_access_point` modality (web, API, email) is usable with a receipt number alone, with no credential. |
| 7.1.1, CIR minimum field set | The CIR carries `controller_identification_record_id`, `controller_public_id_uri` and `privacy_access_point` as an array of type, value and label. |
| 7.3.1, anonymity by default | Requests and status checks need no identification. |
| Annex C, consent row | Withdrawal, access, rectification, erasure and objection are each offered. |
| Annex C, legal_obligation row | Answering rights requests is its own purpose with lawful basis `legal_obligation` and an authority reference. |
| 7.2.5 and C8, authorization state | Withdrawal, objection and erasure each create a new Authorization State Object instance that supersedes the prior one, with `authorization_state_changed` and `record_validity_changed` entries against the notice version. |
| 7.4.4, processing events | Each deletion is recorded as a processing event with `processing_event_id`, `notice_version_reference`, `controller_identification_record_id` and `purpose`, separate from the notice lifecycle. |

Authorization state records and the DRAP log are bilateral. They carry receipt numbers and are held by the controller, not published. The public Notice Event Log records only notice lifecycle events.

## 8. Security

- A receipt is a bearer key with 64 random bits. Guessing one is not practical, so the DRAP may confirm whether a receipt is held.
- Requests are rate limited at the service.
- CORS allows transparencylab.ca only.
- Everything returned under access is escaped before display.
- A rights request receipt does not grant further rights. It only reads the status of its own request.

## 9. Acceptance tests

| # | Test | Pass |
|---|---|---|
| T1 | Withdraw only, held `tl-sa` receipt | Deleted at once; receipt shows "done now"; log entry has no personal data |
| T2 | Access and erasure together | Record shown, then deleted; both outcomes "done" |
| T3 | Rectification without text | Refused with a request for the text |
| T4 | Any right, `TL-2026-NOTICE` receipt | Pending, answered later through the status endpoint |
| T5 | Unknown receipt format | Refused, nothing logged |
| T6 | Reply address given, request closed | Address and text gone from `drap_requests` |
| T7 | Status with `tl-rr` id | Returns outcomes only for that request |
| T8 | CIR and rights.json | Both carry the `drap` block and resolve |
| T9 | Notice 1.3.0 | Event log contains the version |
| T10 | Phone width, keyboard only | Form usable, every checkbox reachable and labelled |

Results, 7 October 2026. T1 to T7 passed against a local copy of the service. Live: access and withdrawal together on a test receipt in Chrome (record shown, then deleted, rights request receipt issued), status by rights request receipt, the existing withdrawal route, and T8 and T9 against the published records. Not yet run: T10 (phone width, keyboard only).

## 10. Not in v1

- Data portability and control. See the design note in Part 2.
- Rights over records not held under a receipt, which would require identifying the person.
- Other controllers. The `drap` block is written so another controller can declare one later.
- A signed or notarised rights request receipt. That belongs to Level 2 of the assurance ladder.

## 11. Decisions taken

1. For a `TL-2026-NOTICE` receipt, an answer is sent by email where a reply address was given, and can always be collected through the status endpoint with the rights request receipt.
2. A closed request row is kept for 24 months without its text or reply address, then deleted. The request log is permanent and holds no personal data beyond receipt numbers.
3. The page spells out "digital rights access point". "DRAP" appears only in machine-readable records.

# Part 2. Design note for a later version

## 12. Portability and control: constraints, not a specification

Portability and control are not specified in v1. They are not built until a real data source and a real grantee exist. Four constraints carry forward from it.

1. **A receipt is not a token.** A receipt is evidence, kept and shown. Access to data held in place uses short-lived tokens bound to the grantee, derived from a grant the receipt records, and revoked when the grant is withdrawn. A receipt alone never reaches data.
2. **Read access is a copy.** Once a grantee has read data, revoking the grant stops later reads, not the copy. A read grant is an export with an audit trail. The value of access in place is in two scopes: query or attestation (an answer without disclosure, such as "over 18" without a date of birth) and notification of change.
3. **Use the existing standards.** Grants are carried over UMA 2.0 or GNAP (RFC 9635), not a new protocol. Terms missing from the W3C Data Privacy Vocabulary (a grant, an access scope, an access event) are proposed to DPV, not published in a separate namespace.
4. **The access log must be independent.** A log kept by the controller is not evidence against the controller. Independent logging belongs with Level 2 of the assurance ladder.

First candidate use: age assurance by attestation.
