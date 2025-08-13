# JWT Parser Function

A Serverless Workflow 1.x function that decodes JWT (JSON Web Token) payloads using jq expressions. This function provides pure jq-based JWT parsing without external dependencies.

## Technical Implementation

This function implements JWT payload extraction through a `set` task using jq expressions:

**Core JWT Decoding Logic:**
```jq
if (.token | startswith("Bearer ")) then
  (.token[7:] | split(".")[1] | @base64d | fromjson)
else
  (.token | split(".")[1] | @base64d | fromjson)
end
```

**Technical Steps:**
1. **Prefix Handling**: Detects and removes "Bearer " prefix if present
2. **Token Splitting**: Splits JWT on "." to isolate header, payload, signature
3. **Payload Extraction**: Selects the middle part (index 1) containing claims
4. **Base64 Decoding**: Uses `@base64d` to decode the base64url payload
5. **JSON Parsing**: Converts decoded string to JSON object with `fromjson`
6. **Optional Claim Navigation**: Uses jq path expressions for specific claim extraction

## Usage

### Complete Payload Extraction

```yaml
document:
  dsl: 1.0.0-alpha1
  namespace: technical
  name: jwt-full-decode
  version: 1.0.0
do:
  - decodeJWT:
      use: jwt-parser
      with:
        token: ${ .headers.authorization }
        # Returns: { claims: {...}, result: {...} }
```

### Specific Claim Extraction

```yaml
document:
  dsl: 1.0.0-alpha1
  namespace: technical
  name: jwt-claim-extraction
  version: 1.0.0
do:
  - extractSubject:
      use: jwt-parser
      with:
        token: ${ .headers.authorization }
        claimPath: ".sub"
        # Returns: { claims: {...}, result: "user-id-123" }
  - extractNestedClaim:
      use: jwt-parser
      with:
        token: ${ .headers.authorization }
        claimPath: ".custom.department"
        # Returns: { claims: {...}, result: "engineering" }
```

### Token Format Handling

```yaml
document:
  dsl: 1.0.0-alpha1
  namespace: technical
  name: jwt-format-handling
  version: 1.0.0
do:
  - parseRawToken:
      use: jwt-parser
      with:
        token: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIn0.signature"
  - parseBearerToken:
      use: jwt-parser
      with:
        token: "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIn0.signature"
        # Both handle the token format automatically
```

## Function Specification

### Input Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `token` | string | Yes | JWT token (raw or with "Bearer " prefix) |
| `claimPath` | string | No | jq path expression for specific claim extraction |

### Output Structure
```json
{
  "claims": {
    "sub": "user-123",
    "preferred_username": "john.doe",
    "email": "john@example.com",
    "exp": 1234567890,
    "iat": 1234567800
  },
  "result": "..." // Complete payload or specific claim based on claimPath
}
```

## Technical Details

### JWT Token Structure
```
header.payload.signature
```
The function processes the **payload** section (index 1 after splitting on ".")

### jq Expression Breakdown

**Primary Expression:**
```jq
if (.token | startswith("Bearer ")) then
  (.token[7:] | split(".")[1] | @base64d | fromjson)
else
  (.token | split(".")[1] | @base64d | fromjson)
end
```

**Claim Path Expression:**
```jq
if .claimPath then
  .claims | getpath(.claimPath | split(".") | map(select(. != "")))
else
  .claims
end
```

### Supported Token Formats
- **Raw JWT**: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0In0.sig`
- **Bearer Format**: `Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0In0.sig`

### Claim Path Examples
| Path | Description | Example Result |
|------|-------------|----------------|
| `.sub` | Subject identifier | `"user-123"` |
| `.preferred_username` | Username | `"john.doe"` |
| `.custom.department` | Nested custom claim | `"engineering"` |
| `.roles[0]` | First role in array | `"admin"` |

## Error Conditions

The function will fail with jq errors if:
- **Invalid token format**: Not exactly 3 parts separated by dots
- **Invalid base64**: Payload section cannot be base64 decoded
- **Invalid JSON**: Decoded payload is not valid JSON
- **Invalid claim path**: Specified jq path does not exist in payload

## Implementation Notes

- **No signature verification**: Function extracts claims without cryptographic validation
- **Base64URL decoding**: Uses jq's `@base64d` which handles base64url format
- **Path navigation**: Uses jq's `getpath()` for safe claim extraction
- **Prefix handling**: Automatically detects and strips "Bearer " prefix
