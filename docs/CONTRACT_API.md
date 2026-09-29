# Linkora API Reference

---

## Contract Function Quick-Reference

The tables below list every public function on `LinkoraContract` grouped by module. Auth requirements use the following shorthand:

- **caller** — the first `Address` argument calls `require_auth()` on itself.
- **Admin role** — caller must hold the `Admin` role granted via `grant_role`.
- **Upgrader role** — caller must hold the `Upgrader` role.
- **Moderator role** — caller must hold the `Moderator` role.
- **Multi-sig** — a `Vec<Address>` of signers all call `require_auth()`; must meet the pool's approval threshold.
- **none** — read-only; no auth required.

---

### Initialisation & Access Control

| Function      | Auth             | Inputs                                                | Returns |
| ------------- | ---------------- | ----------------------------------------------------- | ------- |
| `initialize`  | caller (`admin`) | `admin: Address`, `treasury: Address`, `fee_bps: u32` | `()`    |
| `grant_role`  | Admin role       | `admin: Address`, `account: Address`, `role: Role`    | `()`    |
| `revoke_role` | Admin role       | `admin: Address`, `account: Address`, `role: Role`    | `()`    |
| `has_role`    | none             | `account: Address`, `role: Role`                      | `bool`  |

---

### Social — Profiles

| Function                  | Auth            | Inputs                                                        | Returns           |
| ------------------------- | --------------- | ------------------------------------------------------------- | ----------------- |
| `set_profile`             | caller (`user`) | `user: Address`, `username: String`, `creator_token: Address` | `()`              |
| `get_profile`             | none            | `user: Address`                                               | `Option<Profile>` |
| `get_profile_count`       | none            | —                                                             | `u64`             |
| `delete_profile`          | caller (`user`) | `user: Address`                                               | `()`              |
| `batch_cleanup_profile`   | none            | `user: Address`, `max_entries: u32`                           | `()`              |
| `get_address_by_username` | none            | `username: String`                                            | `Option<Address>` |

---

### Social — Follow Graph

| Function               | Auth                | Inputs                                         | Returns        |
| ---------------------- | ------------------- | ---------------------------------------------- | -------------- |
| `follow`               | caller (`follower`) | `follower: Address`, `followee: Address`       | `()`           |
| `unfollow`             | caller (`follower`) | `follower: Address`, `followee: Address`       | `()`           |
| `get_following`        | none                | `user: Address`, `offset: u32`, `limit: u32`   | `Vec<Address>` |
| `get_followers`        | none                | `user: Address`, `offset: u32`, `limit: u32`   | `Vec<Address>` |
| `batch_follow`         | caller (`follower`) | `follower: Address`, `followees: Vec<Address>` | `()`           |
| `batch_unfollow`       | caller (`follower`) | `follower: Address`, `followees: Vec<Address>` | `()`           |
| `migrate_follow_graph` | Admin role          | `admin: Address`, `users: Vec<Address>`        | `()`           |
| `block_user`           | caller (`blocker`)  | `blocker: Address`, `blocked: Address`         | `()`           |
| `unblock_user`         | caller (`blocker`)  | `blocker: Address`, `blocked: Address`         | `()`           |
| `is_blocked`           | none                | `blocker: Address`, `blocked: Address`         | `bool`         |

---

### Posts

| Function              | Auth              | Inputs                                                              | Returns         |
| --------------------- | ----------------- | ------------------------------------------------------------------- | --------------- |
| `create_post`         | caller (`author`) | `author: Address`, `content: String`                                | `u64` (post ID) |
| `get_post`            | none              | `id: u64`                                                           | `Option<Post>`  |
| `get_post_count`      | none              | —                                                                   | `u64`           |
| `delete_post`         | caller (`author`) | `author: Address`, `post_id: u64`                                   | `()`            |
| `batch_cleanup_post`  | none              | `post_id: u64`, `max_entries: u32`                                  | `()`            |
| `get_posts_by_author` | none              | `author: Address`, `offset: u32`, `limit: u32`                      | `Vec<u64>`      |
| `like_post`           | caller (`user`)   | `user: Address`, `post_id: u64`                                     | `()`            |
| `batch_like`          | caller (`user`)   | `user: Address`, `post_ids: Vec<u64>`                               | `()`            |
| `get_like_count`      | none              | `post_id: u64`                                                      | `u64`           |
| `has_liked`           | none              | `user: Address`, `post_id: u64`                                     | `bool`          |
| `tip`                 | caller (`tipper`) | `tipper: Address`, `post_id: u64`, `token: Address`, `amount: i128` | `()`            |

---

### Pools

| Function                | Auth                  | Inputs                                                                           | Returns                |
| ----------------------- | --------------------- | -------------------------------------------------------------------------------- | ---------------------- |
| `create_pool`           | caller (first signer) | `signers: Vec<Address>`, `pool_id: Symbol`, `token: Address`, `threshold: u32`   | `()`                   |
| `pool_deposit`          | caller (`depositor`)  | `depositor: Address`, `pool_id: Symbol`, `token: Address`, `amount: i128`        | `()`                   |
| `pool_withdraw`         | Multi-sig             | `signers: Vec<Address>`, `pool_id: Symbol`, `recipient: Address`, `amount: i128` | `()`                   |
| `get_pool`              | none                  | `pool_id: Symbol`                                                                | `Option<Pool>`         |
| `get_pool_admins`       | none                  | `pool_id: Symbol`                                                                | `Option<Vec<Address>>` |
| `add_pool_admin`        | Multi-sig             | `signers: Vec<Address>`, `pool_id: Symbol`, `new_admin: Address`                 | `()`                   |
| `remove_pool_admin`     | Multi-sig             | `signers: Vec<Address>`, `pool_id: Symbol`, `admin: Address`                     | `()`                   |
| `update_pool_threshold` | Multi-sig             | `signers: Vec<Address>`, `pool_id: Symbol`, `threshold: u32`                     | `()`                   |

---

### Governance

| Function           | Auth                | Inputs                                                                                                                                   | Returns             |
| ------------------ | ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------- |
| `gov_init_config`  | Admin role          | `admin: Address`, `quorum: u32`, `time_lock_ledgers: u32`, `vote_window_ledgers: u32`, `quorum_decay_rate_bps: u32`, `quorum_floor: u32` | `()`                |
| `gov_get_config`   | none                | —                                                                                                                                        | `GovConfig`         |
| `gov_propose`      | caller (`proposer`) | `proposer: Address`, `parameter: GovParameter`, `new_value: u64`, `new_address: Option<Address>`                                         | `u64` (proposal ID) |
| `gov_vote`         | caller (`voter`)    | `voter: Address`, `proposal_id: u64`, `support: bool`                                                                                    | `()`                |
| `effective_quorum` | none                | `proposal_id: u64`                                                                                                                       | `u32`               |
| `gov_execute`      | Admin role          | `admin: Address`, `proposal_id: u64`                                                                                                     | `()`                |
| `gov_veto`         | Multi-sig           | `signers: Vec<Address>`, `pool_id: Symbol`, `proposal_id: u64`                                                                           | `()`                |
| `gov_get_proposal` | none                | `proposal_id: u64`                                                                                                                       | `GovProposal`       |

---

### Analytics Oracle

| Function                       | Auth       | Inputs                                                               | Returns |
| ------------------------------ | ---------- | -------------------------------------------------------------------- | ------- |
| `register_oracle`              | Admin role | `admin: Address`, `name: Symbol`, `pubkey: BytesN<32>`               | `()`    |
| `verify_analytics_attestation` | none       | `oracle_name: Symbol`, `report_cbor: Bytes`, `signature: BytesN<64>` | `()`    |

---

### Moderation

| Function           | Auth                | Inputs                                                                                                 | Returns          |
| ------------------ | ------------------- | ------------------------------------------------------------------------------------------------------ | ---------------- |
| `report_post`      | caller (`reporter`) | `reporter: Address`, `post_id: u64`, `token: Address`, `stake_amount: i128`, `reason_hash: BytesN<32>` | `()`             |
| `review_report`    | Moderator role      | `moderator: Address`, `post_id: u64`, `reporter: Address`, `verdict: ReportStatus`                     | `()`             |
| `get_report`       | none                | `post_id: u64`, `reporter: Address`                                                                    | `Option<Report>` |
| `get_report_count` | none                | `post_id: u64`                                                                                         | `u32`            |

---

### Protocol Parameters & Admin

| Function                   | Auth        | Inputs                                    | Returns           |
| -------------------------- | ----------- | ----------------------------------------- | ----------------- |
| `set_fee`                  | Admin role  | `admin: Address`, `fee_bps: u32`          | `()`              |
| `get_fee_bps`              | none        | —                                         | `u32`             |
| `set_treasury`             | Admin role  | `admin: Address`, `treasury: Address`     | `()`              |
| `get_treasury`             | none        | —                                         | `Option<Address>` |
| `set_tip_cooldown_window`  | Admin role  | `admin: Address`, `cooldown_ledgers: u32` | `()`              |
| `get_tip_cooldown_window`  | none        | —                                         | `u32`             |
| `set_max_post_content_len` | Admin role  | `admin: Address`, `max_len: u32`          | `()`              |
| `get_max_post_content_len` | none        | —                                         | `u32`             |
| `set_max_bio_len`          | Admin role  | `admin: Address`, `max_len: u32`          | `()`              |
| `get_max_bio_len`          | none        | —                                         | `u32`             |
| `set_rent_rate_bps`        | Admin role  | `admin: Address`, `rate: u32`             | `()`              |
| `get_rent_rate_bps`        | none        | —                                         | `u32`             |
| `pause`                    | Pauser role | `admin: Address`                          | `()`              |
| `unpause`                  | Pauser role | `admin: Address`                          | `()`              |

---

### Rent & Storage

| Function                | Auth            | Inputs                                            | Returns                 |
| ----------------------- | --------------- | ------------------------------------------------- | ----------------------- |
| `pay_rent`              | caller (`user`) | `user: Address`, `token: Address`, `amount: i128` | `()`                    |
| `get_rent_expiry`       | none            | `user: Address`                                   | `u32` (ledger sequence) |
| `batch_bump_user_graph` | Admin role      | `admin: Address`, `user: Address`                 | `u32` (keys bumped)     |

---

### Upgrade

| Function             | Auth          | Inputs                                           | Returns         |
| -------------------- | ------------- | ------------------------------------------------ | --------------- |
| `propose_upgrade`    | Upgrader role | `upgrader: Address`, `new_wasm_hash: BytesN<32>` | `()`            |
| `execute_upgrade`    | Upgrader role | `upgrader: Address`                              | `()`            |
| `upgrade`            | Upgrader role | `upgrader: Address`, `new_wasm_hash: BytesN<32>` | `()`            |
| `get_contract_state` | none          | —                                                | `ContractState` |

---

### Credentials & DM Keys

| Function                   | Auth                                  | Inputs                                                                                 | Returns              |
| -------------------------- | ------------------------------------- | -------------------------------------------------------------------------------------- | -------------------- |
| `set_credential_authority` | Admin role                            | `admin: Address`, `pubkey: BytesN<32>`                                                 | `()`                 |
| `update_credential_root`   | caller (`user`) + authority signature | `user: Address`, `new_root: BytesN<32>`, `signature: BytesN<64>`                       | `()`                 |
| `verify_credential`        | none (mutating)                       | `user: Address`, `proof: Vec<BytesN<32>>`, `leaf: BytesN<32>`, `nullifier: BytesN<32>` | `bool`               |
| `get_credential_root`      | none                                  | `user: Address`                                                                        | `Option<BytesN<32>>` |
| `publish_dm_key`           | caller (`user`)                       | `user: Address`, `x25519_pubkey: BytesN<32>`                                           | `()`                 |
| `get_dm_key`               | none                                  | `user: Address`                                                                        | `Option<BytesN<32>>` |

---

> **Scope note.** The HTTP request authentication reference for the indexer REST API
> continues below. The Soroban storage layout and full event schema are in
> `packages/contracts/contracts/linkora-contracts/src/lib.rs`.

---

## Table of Contents

1. [Stellar HTTP Authentication (v1)](#1-stellar-http-authentication-v1)
2. [The Signed Message](#2-the-signed-message)
3. [Canonical Path](#3-canonical-path)
4. [Body Hash](#4-body-hash)
5. [The Authorization Header](#5-the-authorization-header)
6. [Verification Order and Error Codes](#6-verification-order-and-error-codes)
7. [Worked Example](#7-worked-example)
8. [Protected Endpoints](#8-protected-endpoints)
9. [Known Limitations of v1](#9-known-limitations-of-v1)

---

## 1. Stellar HTTP Authentication (v1)

Protected endpoints authenticate the caller by an Ed25519 signature over a message derived
from the request itself. The signature commits to the HTTP method, the request path, and a
hash of the request body, so a credential captured on one endpoint cannot be replayed
against another.

The canonical implementation lives in `packages/types/src/auth.ts` (`buildAuthMessage`,
`canonicalizeAuthPath`) and is shared by the verifier and the first-party clients. External
integrators should implement the format below directly — see the
[migration guide](./MIGRATION_AUTH_V1.md) for a self-contained reference implementation.

### Why the `v1:` prefix

The message begins with a version tag so the scheme can be rotated without a flag day. A
future `v2:` can add fields (a nonce, header binding, the query string) while a server
accepts both prefixes during a transition window, routing each credential to the verifier
that matches its tag. Without the prefix, any change to the layout would require every
client and every server to cut over in the same instant.

The prefix is part of the signed bytes, so it cannot be swapped by an attacker to downgrade
a v2 credential into a v1 one.

---

## 2. The Signed Message

```
v1:{METHOD}:{canonicalPath}:{address}:{timestamp}:{bodyHash}
```

| Field           | Description                                                                |
| --------------- | -------------------------------------------------------------------------- |
| `v1`            | Literal version tag.                                                       |
| `METHOD`        | HTTP method, **upper-cased**: `GET`, `POST`, `PATCH`, `DELETE`.            |
| `canonicalPath` | Request path after canonicalisation — see [§3](#3-canonical-path).         |
| `address`       | Signer's Stellar public key (`G…`), exactly as sent in the header payload. |
| `timestamp`     | Unix epoch in **milliseconds**, as an integer with no separators.          |
| `bodyHash`      | Lowercase hex SHA-256 of the raw body — see [§4](#4-body-hash).            |

Fields are joined with a literal `:`. No field is escaped or length-prefixed. A path _can_
legitimately contain a `:` (`/api/items:batchGet` is a valid URL), so the message is not
unambiguous by construction — it is unambiguous because the last three fields have fixed
shapes (a 56-character base32 address, digits, 64 hex characters), which lets the suffix be
read unambiguously from the right no matter what the path contains.

What gets signed is the **SHA-256 digest of this message**, not the message itself:

```
signature = Ed25519_sign(privateKey, SHA256(message))
```

The digest is signed as 32 raw bytes. The signature is the raw 64-byte Ed25519 output,
base64-encoded for transport.

> **Wallet signers do not do this today.** The rule above is what the server verifies, and it
> is what a signer holding the key directly (`Keypair.sign`) produces. Browser wallets are a
> different matter: Freighter's `signBlob` treats its argument as an opaque **message**, wraps
> it in the SEP-0053 envelope and signs `SHA256("Stellar Signed Message:\n" + blob)` — never
> the 32 raw bytes. It also returns a `Buffer`, not the base64 string its TypeScript signature
> promises. Both facts are verified against a real Freighter 2.0.0 signature. Consequently the
> first-party browser client **does not currently interoperate with this server**, and
> reconciling the two is an open decision, not something this document describes as settled.
> If you sign with a wallet, expect the envelope; if you sign with a library, follow the rule
> above.

---

## 3. Canonical Path

The server canonicalises `req.originalUrl`, which always carries the **full path including
any router mount prefix**. Clients must sign the same absolute path.

The rule, in order:

1. Drop everything from the first `?` onward.
2. Strip all trailing `/`.
3. If the result is empty, use `/`.

| Raw path                 | Canonical path  |
| ------------------------ | --------------- |
| `/api/follows`           | `/api/follows`  |
| `/api/follows/`          | `/api/follows`  |
| `/api/follows//`         | `/api/follows`  |
| `/api/follows?cursor=5`  | `/api/follows`  |
| `/api/follows/?cursor=5` | `/api/follows`  |
| `/`                      | `/`             |
| `//`                     | `/`             |
| `/api//follows`          | `/api//follows` |

Two consequences worth internalising:

- Sign `/api/notifications/preferences`, never the router-relative `/preferences`. Signing
  the relative path produces a different message and the request comes back `401`.
- If your base URL carries a path segment of its own (`https://example.com/indexer`), that
  segment reaches the server as part of `req.originalUrl` but never appears in the path you
  passed to the signer. Keep the base URL an origin only, or include the prefix in the
  signed path.

---

## 4. Body Hash

`bodyHash` is the lowercase hex SHA-256 of the **exact bytes** sent as the request body.

When there is no body, hash the empty string:

```
SHA256("") = e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
```

This is the value used for every `GET`, and for any request sent without a body. The server
reaches the same value either way: a request with no JSON content type never triggers
body-parser's `verify` hook and its absent raw body is hashed as empty, while a JSON request
with an empty body triggers the hook with a zero-length buffer, which hashes to the same
digest.

> **Hash the bytes you send, not the object you serialised.** Serialise once into a
> variable, then use that same variable for both the hash and the request body. Serialising
> separately for each can yield different bytes — key order, whitespace, number
> formatting — and the signature would then commit to a body that was never sent, producing
> a `401` that is very hard to read.

---

## 5. The Authorization Header

```
Authorization: StellarSig <base64(JSON)>
```

The base64 payload decodes to a JSON object with exactly these three fields:

```json
{
  "address": "G...",
  "timestamp": 1735689600000,
  "signature": "<base64 of the raw 64-byte Ed25519 signature>"
}
```

The header value is the literal `StellarSig`, one space, then the base64 payload. The parser
splits on a single space and rejects anything else, so the payload must not contain spaces.

`address` and `timestamp` are sent in the clear **and** covered by the signature — the
server uses the transmitted values to rebuild the message, then checks the signature against
it. Tampering with either produces a message that no longer verifies.

---

## 6. Verification Order and Error Codes

The server checks in this order and returns on the first failure:

| Order | Check                                 | Status | Code                  |
| ----- | ------------------------------------- | ------ | --------------------- |
| 1     | Header present, parsable, all fields  | `400`  | `INVALID_AUTH_HEADER` |
| 2     | `timestamp` is not in the future      | `403`  | `INVALID_TIMESTAMP`   |
| 3     | `timestamp` is at most 30 s old       | `403`  | `EXPIRED_TIMESTAMP`   |
| 4     | Signature matches the rebuilt message | `401`  | `INVALID_SIGNATURE`   |

Errors carry the shape:

```json
{
  "error": {
    "code": "INVALID_SIGNATURE",
    "message": "Invalid signature",
    "requestId": "..."
  }
}
```

**The order matters when diagnosing a replay.** Timestamp checks run before the signature
check, so an expired credential returns `403` regardless of which path it is presented at —
you will not see the `401` that a wrong path would otherwise produce. A credential replayed
against a different path _within_ the tolerance window is what yields `401`.

The tolerance is 30 000 ms, defined by `SIGNATURE_TIMESTAMP_TOLERANCE_MS` in
`services/indexer/src/middleware/stellarAuth.ts`. Clock skew between client and server eats
directly into this budget; a client running more than 30 s fast is rejected outright as
`INVALID_TIMESTAMP`.

---

## 7. Worked Example

Every value below is reproducible — the keypair is seeded with 32 bytes of `0x07`.

```
address     GDVEU3DD4KOFECV66VIHWEZOYX4ZKR3WV27L464SIIPOU2IUI3JCZA57
method      POST
path        /api/follows
timestamp   1735689600000
body        {"followee":"GBRPYHIL2CI3FNQ4BXLFMNDLFJUNPU2HY3ZMFSHONUCEOASW7QC7OX2H"}
```

Step 1 — hash the body:

```
bodyHash = 420fe3b7b804bb866cbfeb113d913ba1543099424aed7694297cc3f042ee29a9
```

Step 2 — build the message:

```
v1:POST:/api/follows:GDVEU3DD4KOFECV66VIHWEZOYX4ZKR3WV27L464SIIPOU2IUI3JCZA57:1735689600000:420fe3b7b804bb866cbfeb113d913ba1543099424aed7694297cc3f042ee29a9
```

Step 3 — digest it (base64, for inspection):

```
1bWsu8SkJG10Q5vEoWQZdo3ZIbseNrILzJmuOsP2VyM=
```

Step 4 — sign the digest and base64 the signature:

```
evOK1CFjMZSTvHtuXrxhDVImMQi9JByqzurli1avrRrEPyjvhhQHYwyAmJLvicSAgHEnOte0qXFBlafhOHjtAw==
```

Step 5 — assemble the header:

```
Authorization: StellarSig eyJhZGRyZXNzIjoiR0RWRVUzREQ0S09GRUNWNjZWSUhXRVpPWVg0WktSM1dWMjdMNDY0U0lJUE9VMklVSTNKQ1pBNTciLCJ0aW1lc3RhbXAiOjE3MzU2ODk2MDAwMDAsInNpZ25hdHVyZSI6ImV2T0sxQ0ZqTVpTVHZIdHVYcnhoRFZJbU1RaTlKQnlxenVybGkxYXZyUnJFUHlqdmhoUUhZd3lBbUpMdmljU0FnSEVuT3RlMHFYRkJsYWZoT0hqdEF3PT0ifQ==
```

Note that this example's timestamp is long past, so replaying it against a live server
returns `403 EXPIRED_TIMESTAMP`. It is for verifying your message construction, not for
testing a live endpoint.

---

## 8. Protected Endpoints

| Method | Path                             | Body |
| ------ | -------------------------------- | ---- |
| `POST` | `/api/notifications/register`    | JSON |
| `POST` | `/api/notifications/deregister`  | JSON |
| `GET`  | `/api/notifications/preferences` | none |
| `POST` | `/api/notifications/preferences` | JSON |
| `POST` | `/api/messages`                  | JSON |

All other indexer endpoints are public reads and take no `Authorization` header.

---

## 9. Known Limitations of v1

The signature covers the method, the canonical path, and the body. Everything else about the
request — headers, query string, transport — is outside it. Three consequences deserve to be
stated plainly.

### 9.1 The query string is not signed

Canonicalisation strips everything after `?` before signing, so query parameters are **not**
authenticated. A credential captured for `/api/posts?limit=10` is equally valid for
`/api/posts?limit=1000` within the tolerance window.

No currently protected endpoint reads query parameters, so there is nothing to tamper with
today. That is a property of the current route set, not a guarantee of the scheme — **do not
add a query parameter to a protected endpoint** without moving it into the body or extending
the scheme to `v2`.

### 9.2 There is no nonce — identical replay stays possible

The scheme binds a credential to _a_ request shape, not to _one_ delivery of it. Anyone who
observes a request can resend it byte-for-byte — same method, same path, same body — and it
will be accepted until the 30-second window closes. For a non-idempotent endpoint that means
the action happens twice.

The window is the only bound. Defending against identical replay needs a nonce carried in
the signed message plus server-side storage of spent nonces for at least the tolerance
period, which v1 does not have. Until then, TLS is what stands between an observer and a
replayable credential: **these endpoints must not be served over plaintext HTTP.**

### 9.3 A non-JSON body would go unsigned

The server captures the raw body through `express.json({ verify })`, and body-parser only
invokes that hook when the request's `Content-Type` is JSON. For any other content type the
raw body is never captured, `bodyHash` falls back to the hash of the empty string, and the
body travels **entirely outside the signature** — a `multipart/form-data` upload on a
protected route would be freely tamperable while the request still verifies.

Every protected endpoint is JSON-only today, so this is latent rather than live. It becomes a
live vulnerability the moment a protected route accepts another content type. **Any new
protected endpoint must be JSON**, or the raw-body capture in
`services/indexer/src/middleware/rawBody.ts` must be widened to cover its content type first.
