# Circle CCTP API Reference - Comprehensive Summary

> Source: Circle Developer Documentation (CCTP API endpoints)
> Generated: 2026-03-04

## Base URLs

| Environment | Base URL |
|-------------|----------|
| Sandbox | https://iris-api-sandbox.circle.com |
| Production | https://iris-api.circle.com |

## Common Response Headers

- **X-Request-Id**: UUID v4 identifier (developer-provided or Circle-generated), useful for support communications.

---

## Table of Contents

1. Get Attestation (V1 Legacy)
2. Get USDC Transfer Fees (V2)
3. Get Fast Burn USDC Allowance (V2)
4. Get Messages V2
5. Get Messages (V1 Legacy)
6. Get Public Keys V2
7. Get Public Keys (V1 Legacy)
8. Re-attest Message (V2)

---

## 1. Get Attestation (V1 Legacy)

**Endpoint:** `GET /v1/attestations/{messageHash}`
**Operation ID:** `getAttestation`
**Tag:** CCTP V1 (Legacy)

### Purpose
Retrieves the signed attestation for a USDC burn event on the source chain.

### Path Parameters

| Parameter | Type | Required | Pattern | Description |
|-----------|------|----------|---------|-------------|
| messageHash | string | Yes | `^0x[a-fA-F0-9]{64}$` | Message hash generated via keccak256 of message bytes from the MessageSent event |

**Example:** `0x912f22a13e9ccb979b621500f6952b2afd6e75be7eadaed93fc2625fe11c52a2`

### Response (200 OK)

**Schema:** `GetAttestationV1Response`

| Property | Type | Nullable | Description |
|----------|------|----------|-------------|
| attestation | string | Yes | Signed attestation matching the provided messageHash; null when the event is detected but attestation awaits block confirmations |
| status | string (enum) | No | `complete` (attestation is signed) or `pending_confirmations` (awaiting additional block confirmations) |

### Example Responses

**Complete:**
```json
{
  "attestation": "0xdc485fb2f9a8f68c871f4ca7386dee9086ff9d4387756990c9c4b9280338325252866861f9495dce3128cd524d525c44e8e7b731dedd3098a618dcc19c45be1e1c",
  "status": "complete"
}
```

**Pending:**
```json
{
  "attestation": null,
  "status": "pending_confirmations"
}
```

### Error Responses

| Status | Description |
|--------|-------------|
| 404 | Not found. `{"code": 404, "message": "Not found."}` |

---

## 2. Get USDC Transfer Fees (V2)

**Endpoint:** `GET /v2/burn/USDC/fees/{sourceDomainId}/{destDomainId}`
**Operation ID:** `getBurnUsdcFees`
**Tag:** CCTP

### Purpose
Retrieves the fees for a USDC cross-chain transfer between source and destination domains.

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| sourceDomainId | integer (>=0) | Yes | Source domain identifier for a blockchain on CCTP |
| destDomainId | integer (>=0) | Yes | Destination domain identifier for a blockchain on CCTP |

### Query Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| forward | boolean | No | false | Whether to include fees for using the Circle Forwarder in the return value |
| hyperCoreDeposit | boolean | No | false | Whether to include the forwarding fee for depositing into HyperCore (only valid when forward=true and destination is HyperEVM) |

### Response (200 OK)

Returns an **array** of fee objects.

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| finalityThreshold | integer | Yes | The finality threshold (e.g., block confirmations) used to determine whether the transfer qualifies as a Fast or Standard Transfer |
| minimumFee | number | Yes | Minimum fees for the transfer, expressed in basis points (bps). 1 bps = 0.01% |
| forwardFee | object | No | Gas and forwarding fees for Circle Forwarder (in USDC minor units) |
| forwardFee.low | integer | No | The low gas estimate plus forwarding fee |
| forwardFee.medium | integer | No | The medium gas estimate plus forwarding fee |
| forwardFee.high | integer | No | The high gas estimate plus forwarding fee |

### Example Response

```json
[
  {
    "finalityThreshold": 1000,
    "minimumFee": 1,
    "forwardFee": { "low": 90, "medium": 110, "high": 160 }
  },
  {
    "finalityThreshold": 2000,
    "minimumFee": 0,
    "forwardFee": { "low": 90, "medium": 110, "high": 160 }
  }
]
```

### Key Notes
- Fees are expressed as **basis points** (1 bps = 0.01%).
- Forward fees are measured in **USDC minor units** (i.e., 1 = 0.000001 USDC).
- The response can contain multiple objects representing different finality thresholds (Fast vs Standard).

### Error Responses

| Status | Description |
|--------|-------------|
| 400 | Bad request |
| 404 | Not found |

---

## 3. Get Fast Burn USDC Allowance (V2)

**Endpoint:** `GET /v2/fastBurn/USDC/allowance`
**Tag:** CCTP

### Purpose
Retrieves the current USDC Fast Burn allowance remaining.

### Request Parameters
None. Simple GET request with no parameters.

### Response (200 OK)

**Schema:** `USDCFastBurnAllowanceResponseV2`

| Property | Type | Description |
|----------|------|-------------|
| allowance | number | The current USDC Fast Burn allowance remaining, in full units of USDC up to 6 decimals |
| lastUpdated | string (ISO 8601) | UTC datetime when allowance was last updated |

### Example Response

```json
{
  "allowance": 123999.999999,
  "lastUpdated": "2025-01-23T10:00:00Z"
}
```

### Key Notes
- No authentication required (security array is empty in spec).
- Allowance precision supports up to **6 decimal places** (matching USDC 6 decimal places).
- The allowance value is in **full USDC units** (not minor units).

---

## 4. Get Messages V2

**Endpoint:** `GET /v2/messages/{sourceDomainId}`
**Operation ID:** `getMessagesV2`
**Tag:** CCTP

### Purpose
Retrieves messages and attestations for a given source domain, filtered by transaction hash or nonce.

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| sourceDomainId | integer (>=0) | Yes | Source domain identifier for a blockchain on CCTP |

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| transactionHash | string | Conditional | Transaction hash to filter messages. At least one of transactionHash or nonce is required |
| nonce | string | Conditional | Nonce to filter messages. At least one of transactionHash or nonce is required |

**Example transactionHash:** `0x912f22a13e9ccb979b621500f6952b2afd6e75be7eadaed93fc2625fe11c52a2`

### Response (200 OK)

**Schema:** `MessagesV2Response`

**Top-level:**

| Property | Type | Description |
|----------|------|-------------|
| messages | array of MessageV2 | Array of message objects |

**MessageV2 Object:**

| Property | Type | Nullable | Description |
|----------|------|----------|-------------|
| message | string | No | Hex-encoded message. `0x` if the attestation is not yet available |
| eventNonce | string | No | Nonce associated with the message |
| attestation | string | Yes | The attestation. `PENDING` if not yet available |
| decodedMessage | DecodedMessageV2 | Yes | Decoded message representation (null if decoding fails) |
| cctpVersion | integer | No | Enum: 1, 2 |
| status | string | No | Enum: `complete`, `pending_confirmations` |
| delayReason | string | Yes | Enum: `insufficient_fee`, `amount_above_max`, `insufficient_allowance_available` |
| forwardState | string | No | e.g., `PENDING` |
| forwardTxHash | string | No | Pattern: `^0x[a-fA-F0-9]+$` |

**DecodedMessageV2 Object:**

| Property | Type | Description |
|----------|------|-------------|
| sourceDomain | string | Domain ID |
| destinationDomain | string | Domain ID |
| nonce | string | Message nonce |
| sender | string | Address (`^0x[a-fA-F0-9]{40}$`) |
| recipient | string | Address |
| destinationCaller | string | Address |
| minFinalityThreshold | string | Enum: `1000`, `2000` |
| finalityThresholdExecuted | string | Enum: `1000`, `2000` |
| messageBody | string | Application-specific message data |
| decodedMessageBody | DecodedMessageBodyV2 | Decoded body |

**DecodedMessageBodyV2 Object:**

| Property | Type | Description |
|----------|------|-------------|
| burnToken | string | Token address |
| mintRecipient | string | Recipient address |
| amount | string | Amount of burned tokens |
| messageSender | string | Sender address |
| maxFee | string | Maximum fee to pay on destination domain in burnToken units |
| feeExecuted | string | Actual fee charged on destination domain in burnToken units |
| expirationBlock | string | Block expiration number |
| hookData | string | Arbitrary data executed on destination domain |

### Key Notes
- Messages for a given transaction hash are ordered by **ascending log index**.
- At least one query parameter (transactionHash OR nonce) is **mandatory**.
- attestation field may be null or show PENDING during processing.
- decodedMessage and decodedMessageBody may be null if decoding fails.
- delayReason explains why a fast transfer might be delayed.

### Error Responses

| Status | Description |
|--------|-------------|
| 400 | Bad request |
| 404 | Not found |

---

## 5. Get Messages (V1 Legacy)

**Endpoint:** `GET /v1/messages/{sourceDomainId}/{transactionHash}`
**Operation ID:** `getMessages`
**Tag:** CCTP V1 (Legacy)

### Purpose
Retrieves messages for a given source domain and transaction hash (legacy V1 API).

### Path Parameters

| Parameter | Type | Required | Pattern | Description |
|-----------|------|----------|---------|-------------|
| sourceDomainId | integer (>=0) | Yes | - | Source domain identifier for a blockchain on CCTP |
| transactionHash | string | Yes | `^0x[a-fA-F0-9]{64}$` | Transaction hash that contains the message being transferred |

### Response (200 OK)

**Schema:** `MessagesV1Response`

| Property | Type | Description |
|----------|------|-------------|
| messages | array of MessageV1 | Array of message objects |

**MessageV1 Object:**

| Property | Type | Description |
|----------|------|-------------|
| attestation | string | Signed attestation. `PENDING` if the event has been seen but the attestation is still pending block confirmations |
| message | string | Raw message bytes in hexadecimal format |
| eventNonce | string | The nonce associated with the message |

### Example Response

```json
{
  "messages": [
    {
      "attestation": "0xdc485fb2f9a8f68c871f4ca7386dee9086ff9d438775699...",
      "message": "0x000000000000000500000003000000000001...",
      "eventNonce": "9682"
    }
  ]
}
```

### Error Responses

| Status | Description |
|--------|-------------|
| 404 | Specified resource was not found |

### Key Notes
- This is the **legacy V1** endpoint. Prefer V2 (`/v2/messages/{sourceDomainId}`) for new integrations.
- V1 requires transactionHash in the path (not as query param like V2).
- V1 response does not include decoded message fields, forward state, or delay reasons.

---

## 6. Get Public Keys V2

**Endpoint:** `GET /v2/publicKeys`
**Operation ID:** `getPublicKeysV2`
**Tag:** CCTP

### Purpose
Retrieves public keys for validating attestations across all supported versions of CCTP.

### Request Parameters
None. Simple GET request with no parameters.

### Response (200 OK)

**Schema:** `PublicKeysV2Response`

| Property | Type | Description |
|----------|------|-------------|
| publicKeys | array of PublicKey | Array of public key objects |

**PublicKey Object:**

| Property | Type | Description |
|----------|------|-------------|
| publicKey | string | Hex-encoded public key |
| cctpVersion | integer | Enum: 1 or 2 |

### Example Response

```json
{
  "publicKeys": [
    {
      "publicKey": "0x04fc192351b97838713efbc63351e3b71607cc7fc0a74fadaa12d39a693713529bf392c0eeaff62eff2f06b47a4c7cd5f83159e4145444f817d5e7f24e256c6278",
      "cctpVersion": 1
    }
  ]
}
```

### Key Notes
- Returns public keys tagged with their CCTP version, allowing verification of attestations from both V1 and V2.

### Error Responses

| Status | Description |
|--------|-------------|
| 400 | Bad request |

---

## 7. Get Public Keys (V1 Legacy)

**Endpoint:** `GET /v1/publicKeys`
**Operation ID:** `getPublicKeys`
**Tag:** CCTP V1 (Legacy)

### Purpose
Retrieves a list of the currently active public keys for verifying attestation signatures.

### Request Parameters
None.

### Response (200 OK)

**Schema:** `PublicKeysV1Response`

| Property | Type | Description |
|----------|------|-------------|
| publicKeys | array of string | Array of hex-encoded public key strings (unique items) |

### Example Response

```json
{
  "publicKeys": [
    "0x04fc192351b97838713efbc63351e3b71607cc7fc0a74fadaa12d39a693713529bf392c0eeaff62eff2f06b47a4c7cd5f83159e4145444f817d5e7f24e256c6278"
  ]
}
```

### Key Notes
- **Legacy V1 endpoint**. Unlike V2, returns a flat array of strings (no cctpVersion field).
- No authentication required.

---

## 8. Re-attest Message (V2)

**Endpoint:** `POST /v2/reattest/{nonce}`
**Tag:** CCTP

### Purpose
Enables relayers to obtain higher finality levels than originally requested on the source chain. The fee remains charged since allowance was already reserved.

### Path Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| nonce | string | Yes | The nonce of the pre-finality message to re-attest as finalized |

**Example:** `234`

### Request Body
None.

### Response (200 OK)

**Schema:** `ReattestationResponseV2`

| Property | Type | Description |
|----------|------|-------------|
| message | string | Confirmation that the re-attestation process has started |
| nonce | string | The nonce associated with the message |

### Example Response

```json
{
  "message": "Re-attestation successfully requested for nonce.",
  "nonce": "9682"
}
```

### Key Notes
- This is a **POST** endpoint (the only non-GET in this group).
- Used by relayers to upgrade finality level after initial fast transfer.
- Fee is still charged because allowance was already reserved at the time of the original transfer.

### Error Responses

| Status | Description |
|--------|-------------|
| 400 | Bad request |
| 404 | Not found |

---

## Quick Reference: All Endpoints

| # | Method | Endpoint | Version | Description |
|---|--------|----------|---------|-------------|
| 1 | GET | `/v1/attestations/{messageHash}` | V1 (Legacy) | Get signed attestation for a burn event |
| 2 | GET | `/v2/burn/USDC/fees/{sourceDomainId}/{destDomainId}` | V2 | Get USDC transfer fees between domains |
| 3 | GET | `/v2/fastBurn/USDC/allowance` | V2 | Get current fast burn USDC allowance |
| 4 | GET | `/v2/messages/{sourceDomainId}` | V2 | Get messages and attestations by domain |
| 5 | GET | `/v1/messages/{sourceDomainId}/{transactionHash}` | V1 (Legacy) | Get messages by domain and tx hash |
| 6 | GET | `/v2/publicKeys` | V2 | Get attestation public keys (with version) |
| 7 | GET | `/v1/publicKeys` | V1 (Legacy) | Get attestation public keys (flat list) |
| 8 | POST | `/v2/reattest/{nonce}` | V2 | Re-attest a message for higher finality |

## CCTP Domain IDs (Common)

The sourceDomainId and destDomainId parameters reference CCTP domain identifiers. Refer to Circle documentation for the full domain mapping. Common domains include Ethereum, Avalanche, Arbitrum, Base, etc.

## V1 vs V2 Comparison

| Feature | V1 (Legacy) | V2 |
|---------|-------------|-----|
| Messages endpoint | Requires txHash in path | Supports txHash or nonce as query params |
| Message response | Raw attestation + message + nonce only | Includes decoded message, forward state, delay reason, CCTP version |
| Public keys | Flat string array | Objects with cctpVersion field |
| Attestation | Via messageHash | Via messages endpoint |
| Fee information | Not available | Full fee breakdown with forward fees |
| Fast burn allowance | Not available | Available |
| Re-attestation | Not available | Available via POST |
| Finality thresholds | Not exposed | 1000 (fast) and 2000 (standard) |

## Authentication
- Most endpoints have an **empty security array** (no authentication required).
- The X-Request-Id header can be provided for request tracking.
