# Registrar API: Concept and AI Agent Instructions for an OpenAPI Specification

## 1. Goal

Define an OpenAPI specification for the **public read operations** of a national Wallet-Relying Party Registrar API.

Write operations such as registration, update, suspension, cancellation, certificate issuance, administrative verification, and appeal workflows are out of scope for this first version and will be specified later.

The primary goal of the read API is **transparency and public monitoring**. Civil society, journalists, researchers, supervisory bodies, wallet providers, relying party certificate providers, and other ecosystem participants shall be able to inspect which Wallet-Relying Parties are registered, which intended uses they declared, which attestations and claims they intend to request from wallets, which entitlements they rely on, and which certificates are associated with those registrations.

The API shall support both directions of analysis.

Starting from a Wallet-Relying Party, users should be able to answer:

- Which company, public body, natural person, or other entity is registered?
- Under which entitlement or sub-entitlement is it registered?
- For which services is it registered?
- For which intended uses does it rely on wallet data?
- Which credentials, attestations, and claims does it intend to request?
- Does it rely on an intermediary?
- Which access certificates and registration certificates exist or existed?
- When did a specific intended use become active?
- When was it revoked, suspended, cancelled, expired, or superseded?

Starting from requested wallet data, users should be able to answer:

- Which Wallet-Relying Parties request a specific credential type, such as PID?
- Which Wallet-Relying Parties request a specific claim from e.g. a PID, such as age-over-18, family name, birth date, address, or another attribute?
- For which declared purpose are these claims requested?
- Which entities request unusually broad data sets?
- Which entities request data through intermediaries?
- Which intended uses were active at a specific point in time, for example in 2025?

The API is meant to enable overasking detection, public accountability, ecosystem transparency, reproducible monitoring, and change analysis over time.

## 2. Regulatory alignment

The API shall remain aligned with Commission Implementing Regulation (EU) 2025/848 on the registration of wallet-relying parties.

The API shall therefore satisfy at least the following requirements:

- It shall expose information about registered Wallet-Relying Parties from the national register.
- It shall be publicly available online.
- It shall be available in a machine-readable form suitable for automated processing.
- It shall be published as an OpenAPI version 3 specification.
- It shall use JSON.
- The JSON information made available by or on behalf of the registrar shall be electronically signed or sealed using JSON Web Signatures where required.
- It shall use JWS JSON serialization.
  - JWS Compact Serialization SHOULD NOT be used for public monitor statements, because JSON Serialization is easier to process, easier to document in OpenAPI, and supports multiple signatures.
- It shall allow access without prior authentication for public read, search, and list operations.
- It shall allow search and list requests using defined parameters, including at least:
  - official or business registration number;
  - name of the Wallet-Relying Party;
  - privacy policy URL;
  - entitlement;
  - sub-entitlement, if defined by the Member State;
  - whether the Wallet-Relying Party relies on an intermediary;
  - the associated intermediary, where applicable.
- If a query matches at least one Wallet-Relying Party, the response shall provide one or more statements containing the relevant public registration information.
- Public responses shall include current and historic Wallet-Relying Party Access Certificates and Wallet-Relying Party Registration Certificates, where applicable.
- The API shall provide page-based access to the register, including pagination metadata and links to next pages where applicable.
- Public responses shall exclude the physical address from public registration information where required.
- The API shall be designed with security by default and by design, especially for integrity, availability, abuse resistance, and reliable access to public information.

## 3. Challenge to the ARF TS API approach

Do not simply copy an API design that assumes all relevant monitoring use cases can be served efficiently through one broad query endpoint.

The legal requirement for a “single common API” shall be interpreted as **one common API surface and one interoperable OpenAPI specification**, not as a single overloaded endpoint.

The API should therefore use a resource-oriented design that supports efficient querying, indexing, pagination, filtering, deduplication, and incremental synchronization.

The API shall avoid forcing users to download or query the entire register just to answer common monitoring questions such as:

- Which Wallet-Relying Parties request PID?
- Which Wallet-Relying Parties request the birth date claim?
- Which Wallet-Relying Parties request age-over-18?
- Which Wallet-Relying Parties use a specific intermediary?
- Which Wallet-Relying Parties changed their intended uses in the last seven days?
- Which registrations were suspended, cancelled, revoked, or updated?
- Which relying parties have registration certificates for a specific intended use?
- Which intended uses were active during a specific year?

The API shall support these use cases directly through well-defined query parameters and/or dedicated index endpoints.

## 4. Core architecture

The public Registrar API shall be designed as a **certificate-derived transparency API**.

The registrar has an internal database in which all issued Wallet-Relying Party Access Certificates and Wallet-Relying Party Registration Certificates are stored.

This internal certificate database acts as the authoritative transaction source for the public monitor database.

The public monitor database is a derived read model built from issued certificates and their lifecycle events.

Only data that has been published through an issued certificate shall be considered part of the public monitor state. Internal draft registrations, applications, pending changes, or registrar-side administrative data shall not appear in the public monitor API unless they have resulted in an issued certificate or a publicly visible certificate status change.

This allows external actors to reconstruct the public monitor state for a specific point in time, for example the state of the register on `2025-12-31`, by replaying issued certificates and their revocation/status events.

The architecture shall distinguish between the following layers:

```text
Internal registrar database
  └─ applications
  └─ administrative registrations
  └─ checks and decisions
  └─ drafts
  └─ issued certificates

Certificate transaction store
  └─ issued access certificates
  └─ issued registration certificates
  └─ revocation/status events
  └─ suspension/cancellation events
  └─ publication timestamps

Public monitor projection
  └─ deduplicated Wallet-Relying Parties
  └─ deduplicated intended uses
  └─ requested credential index
  └─ requested claim index
  └─ provided attestation index
  └─ certificate evidence links
```

The most important rule is:

```text
Certificates = authoritative public transaction source
Certificate events = replayable history
Monitor database = derived read model
WRPs / intended uses / claims = deduplicated projections
```

## 5. Core principle

The public API shall distinguish between:

1. **Certificate events**  
   The append-only or monotonic history of issued, revoked, expired, suspended, cancelled, reinstated, or superseded certificates.

2. **Certificate evidence**  
   The actual Access Certificates and Registration Certificates, including signed payloads, status references, certificate chains, and verification material.

3. **Canonical monitor entities**  
   Deduplicated public views derived from the certificate events, such as:
   - Wallet-Relying Parties;
   - intended uses;
   - requested credentials;
   - provided attestations;
   - intermediary relationships.

The monitor API shall expose both the canonical deduplicated state and the underlying certificate evidence.

## 6. Design principles

The OpenAPI specification shall follow these principles.

### 6.1 Public read access

All public read operations shall be accessible without authentication.

Authentication and authorization schemes for administrative clients are out of scope for the read API, since they are not required for public monitoring and transparency. Instead, the API read process should be harmonized between the member state implementations to allow public monitoring and ecosystem transparency across the EU.

### 6.2 Resource-oriented API

Use separate resources for Wallet-Relying Parties, intended uses, requested credentials, provided attestations, certificates, signed statements, and certificate events.

### 6.3 Certificate-derived monitor state

The public monitor state shall be derived from issued certificates and certificate lifecycle events.

The API shall not expose unpublished internal registrar data as if it were publicly active registration data.

### 6.4 Search and monitoring first

The API shall priorities discoverability, filtering, aggregation, auditability, and reproducible monitoring over administrative registration workflows.

### 6.5 Stable identifiers

Every Wallet-Relying Party, intended use, certificate, statement, and certificate event shall have stable identifiers suitable for citation, caching, verification, and long-term monitoring.

### 6.6 Signed authoritative evidence

Search endpoints may return lightweight result objects and unsigned operational metadata, but the authoritative registration data shall be available as signed/sealed certificate payloads or signed/sealed statements.

### 6.7 Pagination and incremental sync

List endpoints shall support cursor-based pagination.

Monitoring clients shall be able to fetch certificate events or changes since a given timestamp, cursor, or monotonic sequence number.

### 6.8 Claim-level transparency

The API shall make requested credentials searchable and shall support claim-level filtering within credential scope.

### 6.9 Certificate history

The API shall expose current and historic Wallet-Relying Party Access Certificates and Wallet-Relying Party Registration Certificates where applicable.

### 6.10 Privacy-aware public data

The API shall not expose fields that must not be public, especially physical address where the public API is required to exclude it.

### 6.11 Interoperability

Use standard JSON Schema definitions, stable enum values, URI-based entitlement identifiers, and OpenAPI-compatible representations.

### 6.12 Efficient implementation

Avoid endpoint designs that require full-table scans for common searches.

The data model should allow indexing by:

- WRP identifier;
- legal name;
- trade name;
- entitlement;
- sub-entitlement;
- intended-use fingerprint;
- intended-use status;
- credential format;
- credential type;
- claim path;
- privacy policy URL;
- intermediary;
- certificate type;
- certificate status;
- registration timestamp;
- revocation timestamp;
- last update timestamp.

### 6.13 Human and machine usability

The same underlying data model should support both a human-readable national website and the machine-readable API.

## 7. Data model

The OpenAPI components shall include JSON Schema definitions for at least:

- `RegistryMetadata`
- `WalletRelyingParty`
- `WalletRelyingPartySummary`
- `Identifier`
- `MultiLangString`
- `IntendedUse`
- `IntendedUseSummary`
- `RequestedCredential`
- `RequestedClaim`
- `ProvidedAttestation`
- `SupervisoryAuthority`
- `IntermediaryReference`
- `CertificateSummary`
- `CertificateDetail`
- `CertificatePayload`
- `CertificateStatus`
- `CertificateEvent`
- `SignedStatement`
- `Pagination`
- `ErrorResponse`

The data model shall map to the registration certificate payload where possible.

The following field names from the registration certificate example should be considered for compatibility or explicit mapping:

- `name`
- `sub_ln`
- `sub_gn`
- `sub_fn`
- `sub`
- `country`
- `registry_uri`
- `srv_description`
- `entitlements`
- `privacy_policy`
- `info_uri`
- `support_uri`
- `supervisory_authority`
- `policy_id`
- `certificate_policy`
- `iat`
- `status`
- `purpose`
- `credentials`
- `provides_attestations`
- `intermediary`

However, the public API does not have to expose exactly the same flat structure everywhere. It may use a normalized resource model as long as the signed certificate payloads, signed statements, and certificate evidence remain available.

## 8. Wallet-Relying Party model

A Wallet-Relying Party is a canonical public monitor entity derived from one or more access certificates and/or registration certificates.

`GET /wrps/{wrp_id}` shall not embed all intended uses by default, because a single relying party may have hundreds of intended uses.

The WRP detail response shall include WRP-level metadata, counters, and links to paginated resources.

Example WRP-level fields:

- `wrp_id`
- `display_name`
- `legal_name`
- `trade_name`
- `identifiers`
- `country`
- `info_uri`
- `support_uri`
- `privacy_policy`
- `entitlements`
- `is_public_sector_body`
- `uses_intermediary`
- `intermediaries`
- `supervisory_authority`
- `status`
- `first_seen_at`
- `last_confirmed_at`
- `intended_use_count`
- `active_intended_use_count`
- `certificate_count`
- `links`

The response shall include links such as:

```json
{
  "links": {
    "intended_uses": "/wrps/{wrp_id}/intended-uses",
    "certificates": "/wrps/{wrp_id}/certificates",
    "certificate_events": "/wrps/{wrp_id}/certificate-events",
    "statements": "/wrps/{wrp_id}/statements"
  }
}
```

The API may support an `include` parameter for convenience, but it shall not allow unbounded expansion of intended uses.

Examples:

```text
GET /wrps/{wrp_id}?include=intended_use_summary
GET /wrps/{wrp_id}?include=intended_uses&limit_intended_uses=25
```

If embedded intended uses are supported, the API shall enforce a small maximum preview limit and require pagination for the full collection.

## 9. Intended use as first-class resource

An intended use shall be treated as a first-class public monitor entity.

An intended use may be created by the issuance of a Wallet-Relying Party Registration Certificate that contains a purpose and the related requested credentials and claims.

An intended use may become inactive when:

- the registration certificate that introduced it is revoked;
- all certificates supporting it are revoked;
- the certificate expires and is not replaced by an equivalent valid certificate;
- the registration is cancelled;
- the registration is suspended;
- a newer certificate supersedes the previous intended use;
- the relying party explicitly revokes or cancels the intended use.

The API shall expose lifecycle metadata for each canonical intended use:

- `intended_use_id`
- `intended_use_fingerprint`
- `wrp_id`
- `purpose`
- `requested_credentials`
- `requested_claims`
- `privacy_policy`
- `entitlement`
- `sub_entitlement`
- `intermediary`, where applicable
- `status`
- `first_registered_at`
- `last_confirmed_at`
- `revoked_at`
- `expired_at`
- `suspended_at`
- `cancelled_at`
- `superseded_at`
- `valid_from`
- `valid_until`
- `supporting_certificate_count`
- `current_supporting_certificate_count`
- `supporting_certificate_ids`
- `current_supporting_certificate_ids`
- `revocation_evidence`, where applicable
- `links`

## 10. Deduplication of intended uses

The monitor database shall deduplicate intended uses.

If the same Wallet-Relying Party receives multiple registration certificates containing the same intended use, the public API shall not show duplicate intended-use entries.

Instead, the API shall create one canonical intended-use record and link it to all registration certificates that support or supported that intended use.

Two intended uses shall be considered equal if their canonicalized content is equal.

The canonicalization shall include at least:

- Wallet-Relying Party stable identifier;
- purpose text, normalized by language;
- requested credential format;
- requested credential type, such as `vct` or `doctype`;
- requested claim paths;
- privacy policy URI;
- applicable entitlement or sub-entitlement;
- intermediary reference, where applicable;
- relying-party registration context, where applicable.

The canonicalization shall exclude certificate-specific metadata such as:

- certificate ID;
- certificate issuance timestamp;
- certificate validity timestamp;
- signature;
- status-list index;
- status-list URI;
- certificate serial number;
- certificate chain;
- proof material.

The monitor database shall compute a stable `intended_use_fingerprint` over the canonicalized intended-use content.

The public intended-use ID may either be:

- a registrar-assigned stable ID, if the registrar assigns one; or
- a deterministic ID derived from the Wallet-Relying Party identifier and the `intended_use_fingerprint`.

Example:

```text
intended_use_id = hash(wrp_id + canonical_intended_use_payload)
```

This prevents duplicate entries where multiple registration certificates express the same intended use.

## 11. Intended use equality and versioning

If an intended use changes materially, it shall create a new canonical intended-use version.

Material changes include:

- adding or removing requested credentials;
- adding or removing requested claims;
- changing the purpose;
- changing the privacy policy;
- changing the intermediary;
- changing the applicable entitlement or sub-entitlement.

The previous intended use version may become `superseded`, `revoked`, `expired`, or remain historically visible depending on the certificate status events.

The API shall not overwrite history.

## 12. Certificate event model

The OpenAPI specification shall include a certificate event resource.

A certificate event represents a public transaction relevant for reconstructing monitor state.

Example event types:

- `access_certificate_issued`
- `access_certificate_revoked`
- `access_certificate_expired`
- `access_certificate_suspended`
- `access_certificate_reinstated`
- `registration_certificate_issued`
- `registration_certificate_revoked`
- `registration_certificate_expired`
- `registration_certificate_suspended`
- `registration_certificate_reinstated`
- `registration_certificate_superseded`
- `registration_suspended`
- `registration_cancelled`
- `registration_reinstated`

Each event shall include:

- `event_id`
- `event_sequence`
- `event_type`
- `occurred_at`
- `published_at`
- `wrp_id`
- `certificate_id`
- `certificate_type`
- `certificate_fingerprint`
- `status`
- `status_list_uri`, where applicable
- `status_list_index`, where applicable
- `previous_status`, where applicable
- `new_status`, where applicable
- `source`
- links to certificate detail and signed statement

The `event_sequence` shall be monotonic where possible. This allows external monitors to consume all events in a stable order.

## 13. Time travel and point-in-time reconstruction

The public API shall support reconstruction of the public monitor state for a specific point in time.

The API shall support an `at` query parameter on monitor endpoints.

Examples:

```text
GET /wrps?at=2025-12-31T23:59:59Z
GET /intended-uses?at=2025-12-31T23:59:59Z
GET /requested-credentials?at=2025-12-31T23:59:59Z
GET /wrps/{wrp_id}/intended-uses?at=2025-12-31T23:59:59Z
```

When `at` is provided, the API shall return the state that can be derived from all certificate events published up to that point in time.

The API shall also support interval queries for monitoring.

Examples:

```text
GET /certificate-events?from=2025-01-01T00:00:00Z&to=2025-12-31T23:59:59Z
GET /intended-uses?registered_from=2025-01-01T00:00:00Z&registered_to=2025-12-31T23:59:59Z
GET /intended-uses?revoked_from=2025-01-01T00:00:00Z&revoked_to=2025-12-31T23:59:59Z
```

## 14. Proposed API resources

The OpenAPI specification shall define two layers of public read endpoints:

1. Evidence / transaction endpoints.
2. Derived monitor endpoints.

## 15. Registry metadata endpoints

### `GET /.well-known/registrar-api`

Returns machine-readable metadata about the registrar API, including:

- registrar identifier;
- country;
- API base URL;
- OpenAPI document URL;
- human-readable registry website URL;
- supported API version;
- supported signing algorithms;
- supported credential formats;
- supported entitlement URI namespace;
- timestamp of the latest registry update;
- URL of registrar signing keys or trust anchor information.

### `GET /openapi.json`

Returns the OpenAPI specification.

## 16. Evidence and transaction endpoints

These endpoints expose the certificate-derived transaction history.

### `GET /certificate-events`

Returns the replayable sequence of certificate events.

This is the main endpoint for external actors that want to rebuild their own monitor database.

It shall support filters:

- `from`
- `to`
- `cursor`
- `limit`
- `wrp_id`
- `certificate_type`
- `event_type`
- `status`
- `include_payload`
- `at`

The API shall guarantee stable ordering, preferably by a monotonic sequence number and publication timestamp.

### `GET /certificate-events/{event_id}`

Returns a single certificate event.

### `GET /certificates`

Searches current and historic certificates associated with Wallet-Relying Parties.

Support filters such as:

- `wrp_id`
- `certificate_type`: `access_certificate` or `registration_certificate`
- `status`: active, expired, revoked, suspended, cancelled, historic
- `serial_number`
- `subject_identifier`
- `issuer`
- `valid_at`
- `issued_after`
- `issued_before`
- `limit`
- `cursor`

### `GET /certificates/{certificate_id}`

Returns metadata and the encoded certificate or certificate object.

For Wallet-Relying Party Access Certificates, include the certificate chain or the public certificate representation where applicable.

For Wallet-Relying Party Registration Certificates, include the signed JWT/JWS or other standard representation, where applicable.

### `GET /certificates/{certificate_id}/payload`

Returns the decoded public payload of a certificate, if the certificate type supports a structured payload.

The response shall clearly distinguish between:

- decoded convenience representation;
- original signed representation;
- verification material.

### `GET /certificates/{certificate_id}/status`

Returns the current and historic status information of a certificate.

### `GET /wrps/{wrp_id}/certificates`

Returns all certificates associated with one Wallet-Relying Party.

### `GET /wrps/{wrp_id}/certificate-events`

Returns all certificate events associated with one Wallet-Relying Party.

## 17. Derived monitor endpoints

These endpoints expose the deduplicated monitor state derived from certificates.

### `GET /wrps`

Searches Wallet-Relying Parties.

Support filters such as:

- `q`: free-text search over legal name, trade name, service name, and identifiers;
- `identifier`: official or business registration number;
- `identifier_type`: EUID, LEI, VAT, EORI, national register number, tax reference, or other national identifier;
- `name`: legal name or trade name;
- `country`;
- `entitlement`;
- `sub_entitlement`;
- `privacy_policy_url`;
- `uses_intermediary`;
- `intermediary_identifier`;
- `status`;
- `updated_since`;
- `at`;
- `limit`;
- `cursor`.

The response shall be paginated and may return compact search results, but each result shall link to the full signed statement and certificate evidence.

### `GET /wrps/{wrp_id}`

Returns the public representation of a single Wallet-Relying Party, excluding non-public fields.

The response shall include:

- stable WRP identifier;
- legal name or natural person name, where applicable;
- trade name;
- public identifiers;
- country;
- info URI;
- support/contact URI where public;
- service descriptions;
- public sector body flag;
- entitlements;
- provided attestations, where applicable;
- intermediary relationships;
- supervisory authority;
- privacy policy links;
- current status;
- timestamps;
- counters;
- links to intended uses;
- links to certificates;
- links to certificate events;
- links to signed statements.

The response shall not embed all intended uses by default.

### `GET /wrps/{wrp_id}/intended-uses`

Returns the paginated canonical intended uses associated with one Wallet-Relying Party.

Support filters such as:

- `status`
- `purpose_q`
- `credential_format`
- `credential_type`
- `claim_path`
- `registered_from`
- `registered_to`
- `revoked_from`
- `revoked_to`
- `at`
- `limit`
- `cursor`

### `GET /intended-uses`

Searches canonical intended uses independently from the WRP object.

This endpoint is critical for monitoring because intended uses are the level at which purposes and requested data are declared.

Support filters such as:

- `purpose_q`: free-text search over purpose descriptions;
- `wrp_id`;
- `wrp_identifier`;
- `credential_format`;
- `credential_type`;
- `claim_path`;
- `claim_name`;
- `entitlement`;
- `sub_entitlement`;
- `privacy_policy_url`;
- `uses_intermediary`;
- `intermediary_identifier`;
- `status`;
- `registered_from`;
- `registered_to`;
- `revoked_from`;
- `revoked_to`;
- `updated_since`;
- `at`;
- `limit`;
- `cursor`.

`GET /intended-uses` shall return canonical intended uses, not raw certificate payload entries.

### `GET /intended-uses/{intended_use_id}`

Returns a single canonical intended use, including:

- intended use identifier;
- intended use fingerprint;
- WRP reference;
- localized purpose descriptions;
- requested credentials;
- privacy policy URL for this intended use;
- applicable entitlement or sub-entitlement;
- intermediary reference, where applicable;
- supervisory authority;
- registration certificate references;
- signed statement references;
- lifecycle metadata;
- status and history metadata.

### `GET /intended-uses/{intended_use_id}/certificates`

Returns all certificates that support or supported a canonical intended use.

This allows users to verify why the monitor considers an intended use active, revoked, expired, or historically valid.

### `GET /intended-uses/{intended_use_id}/certificate-events`

Returns all certificate events relevant for the lifecycle of a canonical intended use.

### `GET /requested-credentials`

Searches registered requested credentials and claims.

Support filters such as:

- `format`: e.g. `dc+sd-jwt`, `mso_mdoc`;
- `vct`;
- `doctype`;
- `claim_path`;
- `claim_name`;
- `claim_value`;
- `wrp_id`;
- `entitlement`;
- `purpose_q`;
- `status`;
- `registered_from`;
- `revoked_from`;
- `updated_since`;
- `at`;
- `limit`;
- `cursor`.

This endpoint shall enable questions such as:

- Which Wallet-Relying Parties request `urn:eudi:pid:de:1`?
- Which Wallet-Relying Parties request mdoc PID `age_over_18`?
- Which Wallet-Relying Parties request birth date or address claims from the PID?

### `GET /provided-attestations`

Searches attestations that Wallet-Relying Parties intend to issue, relevant for entities registered as PID providers, QEAA providers, non-qualified EAA providers, or public EAA providers.

Support filters such as:

- `format`
- `vct`
- `doctype`
- `schema_uri`
- `provider_wrp_id`
- `entitlement`
- `status`
- `at`
- `limit`
- `cursor`

### `GET /intermediaries`

Searches intermediaries and intermediary relationships derived from certificates.

Support filters such as:

- `intermediary_identifier`
- `intermediary_name`
- `wrp_id`
- `status`
- `at`
- `limit`
- `cursor`

## 18. Signed statement endpoints

### `GET /statements/{statement_id}`

Returns the authoritative signed/sealed JSON statement for a Wallet-Relying Party, intended use, certificate history snapshot, or monitor projection snapshot.

The API shall define clearly whether the response uses:

- JWS Compact Serialization;
- JWS JSON Serialization;
- a JSON envelope containing payload and signature information.

The OpenAPI specification shall include the media types accordingly.

Search endpoints may return unsigned operational metadata, but they shall link to signed statements and certificate evidence for verification.

### `GET /wrps/{wrp_id}/statements`

Returns statements associated with one Wallet-Relying Party.

### `GET /intended-uses/{intended_use_id}/statements`

Returns statements associated with one canonical intended use.

## 19. Optional aggregation endpoints

The agent may include optional aggregation endpoints if useful.

### `GET /statistics`

Returns public high-level statistics, such as:

- number of registered Wallet-Relying Parties;
- number by entitlement;
- number by status;
- number of intended uses;
- most requested credential types;
- most requested claims;
- number using intermediaries;
- number of active, revoked, expired, suspended, and cancelled intended uses.

### `GET /credential-types`

Returns a normalized list of credential types known in registered intended uses.

These aggregation endpoints are optional, but useful for monitoring and UI development.

## 20. Search semantics

The OpenAPI specification shall define search semantics explicitly.

- String searches should support exact and partial matching where required.
- Free-text search should be separate from exact identifier lookup.
- URI-based filters shall support exact matching.
- Claim paths shall be represented as arrays in data objects, but query parameters may use dot notation or repeated parameters.
- Claim searches SHALL be credential-scoped. APIs SHALL require `credential_type` for claim-only index endpoints.
- Credential format filters shall support at least SD-JWT VC and ISO mdoc representations.
- Pagination shall use opaque cursors, not offset-based pagination, to support stable monitoring.
- Responses shall include `next_cursor` if more results are available.
- Responses shall include `total_count` only if the implementation can provide it efficiently; otherwise it should be optional.
- Time filters shall use RFC 3339 timestamps.
- All timestamps shall be UTC.
- The `at` parameter shall define a point-in-time view derived from certificate events known at that time.
- Interval parameters such as `registered_from`, `registered_to`, `revoked_from`, and `revoked_to` shall filter lifecycle events, not arbitrary update timestamps.

## 21. Response signing strategy

The OpenAPI specification shall distinguish between three kinds of responses.

### 21.1 Search/list metadata

Pagination metadata, query echo, result counts, and lightweight search summaries may be unsigned operational metadata.

### 21.2 Authoritative certificate evidence

The actual issued Access Certificates and Registration Certificates are authoritative evidence and shall be retrievable.

The API shall provide verification material or references required to verify signatures, certificate chains, status, and revocation information.

### 21.3 Authoritative registry statements

The actual public registration information shall be available as electronically signed or sealed JSON statements where required.

The API shall provide links from search results to signed statement resources so monitoring clients can verify authoritative data.

## 22. Privacy and data minimisation

The public API shall not expose non-public registration information.

In particular:

- The physical address registered with the registrar shall not be included in public API responses if it is excluded by the Implementing Act.
- Contact details shall only be exposed as required or permitted for public transparency and support.
- Natural person Wallet-Relying Parties require careful modelling to avoid unnecessary exposure of personal data.
- The OpenAPI specification shall document which fields are public, which are omitted, and which may only exist in internal/write-side systems.
- Internal drafts, rejected applications, incomplete registrations, and unpublished registrar-side notes shall not be exposed.

## 23. Pagination and synchronization

All endpoints returning lists shall support cursor-based pagination.

The cursor shall be opaque.

The `certificate-events` endpoint shall be suitable for full external synchronization.

External monitors should be able to:

1. Start from an empty database.
2. Fetch all certificate events from the beginning of the public event log.
3. Replay all issued, revoked, expired, suspended, cancelled, and superseded certificate events.
4. Build their own monitor projection.
5. Continue synchronization by fetching only new events after the last consumed cursor or sequence number.

Offset-based pagination should not be used for monitor reconstruction because the underlying data may change while the client paginates.

## 24. Error handling

The OpenAPI specification shall define a consistent error model.

Use standard HTTP status codes, including at least:

- `400 Bad Request` for invalid query parameters;
- `404 Not Found` for unknown resources;
- `406 Not Acceptable` for unsupported media types;
- `429 Too Many Requests` for rate limiting;
- `500 Internal Server Error`;
- `503 Service Unavailable`.

The error response shall include:

- `type`;
- `title`;
- `status`;
- `detail`;
- `instance`;
- optional `code`;
- optional `invalid_params`.

## 25. Example lifecycle scenario

A relying party has three registration certificates.

Certificate A, issued on `2025-01-10`, contains:

- purpose: “I need your age because you buy alcohol”;
- credential: PID;
- claim: `age_equal_or_over.18`.

Certificate B, issued on `2025-02-01`, contains the same intended use.

Certificate C, issued on `2025-06-01`, contains the same intended use plus an additional claim.

The monitor database shall show:

1. One canonical intended use for Certificate A and Certificate B, with:
   - `first_registered_at = 2025-01-10T00:00:00Z`;
   - `last_confirmed_at = 2025-02-01T00:00:00Z`;
   - `supporting_certificate_count = 2`.

2. A second canonical intended-use version for Certificate C, because the requested data changed.

If Certificate A is revoked but Certificate B remains valid, the first canonical intended use remains active.

If all certificates supporting the canonical intended use are revoked or expired, the intended use becomes inactive and receives `revoked_at`, `expired_at`, or another applicable lifecycle timestamp.

## 26. Non-goals

The first version of the OpenAPI specification shall not define:

- registration write operations;
- update operations;
- suspension or cancellation write operations;
- authentication for administrative clients;
- certificate issuance workflows;
- internal registrar verification workflows;
- payment or fee handling;
- business process workflows for appeal or redress;
- internal review notes;
- unpublished registrar decisions.

These can be added later in a separate administrative API.

## 27. AI agent task

Generate an OpenAPI version 3 specification for the public read-only Registrar API described above.

The output shall be a complete OpenAPI YAML file.

The agent shall:

1. Use the Implementing Act as the legal baseline.
2. Use the registration certificate example and ETSI TS 119 475 as input for the signed statement and registration certificate data model.
3. Use the common set of relying party information as input for mandatory public fields.
4. Do not blindly copy the ARF TS endpoint design.
5. Preserve alignment with the Implementing Act by keeping one common public API surface.
6. Design efficient resource-oriented endpoints for monitoring.
7. Model the public API as a certificate-derived transparency API.
8. Include raw certificate evidence endpoints.
9. Include certificate event endpoints for external monitor reconstruction.
10. Include derived deduplicated monitor endpoints.
11. Include point-in-time reconstruction using an `at` parameter.
12. Include interval queries for registered and revoked intended uses.
13. Include canonical intended-use fingerprints to avoid duplicate entries.
14. Link every derived monitor entity to the certificates and certificate events that prove it.
15. Include schemas, parameters, responses, examples, pagination, and error models.
16. Mark all operations as public read-only.
17. Include no write operations.
18. Include TODO comments only where the source material is ambiguous or where a national decision is required.
19. Prefer explicit schemas over untyped objects.
20. Include examples for:
    - searching by company name;
    - searching by entitlement;
    - searching by PID credential type;
    - searching by age-over-18 claim;
    - retrieving a WRP detail response;
    - retrieving paginated intended uses for a WRP with 300+ intended uses;
    - retrieving an intended use;
    - retrieving a signed statement;
    - retrieving certificate history;
    - retrieving certificate events;
    - retrieving changes since a timestamp;
    - reconstructing the monitor state for a specific point in time.

The generated OpenAPI specification should be suitable as a first implementation contract for a public transparency API and as a basis for later administrative write APIs.

## 28. Key architectural summary

The final OpenAPI specification shall be based on the following architectural summary:

```text
Certificates are the transaction source.
Certificate events are the replayable history.
The monitor database is a derived public projection.
Wallet-Relying Parties are deduplicated monitor entities.
Intended uses are deduplicated first-class monitor entities.
Requested credentials and claims are searchable indexes.
Every derived record links back to its certificate evidence.
The API supports point-in-time reconstruction.
The API exposes no unpublished internal registrar data.
```
