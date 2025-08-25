# JWT Parser Function

A Serverless Workflow 1.x function that decodes JWT (JSON Web Token) payloads using jq expressions. This function provides pure jq-based JWT parsing without external dependencies.

## Technical Implementation

This function implements JWT payload extraction through a `run` task using jq expressions:

**Core JWT Decoding Logic:**
```jq
(if (.token | startswith("Bearer ")) then .token[7:] else .token end) |
split(".") |
if length != 3 then error("Invalid JWT format: must have 3 parts") else .[1] end |
@base64d |
fromjson
```

**Technical Steps:**
1. **Prefix Handling**: Detects and removes "Bearer " prefix if present
2. **Token Splitting**: Splits JWT on "." to isolate header, payload, signature
3. **Format Validation**: Ensures exactly 3 parts (header.payload.signature)
4. **Payload Extraction**: Selects the middle part (index 1) containing claims
5. **Base64 Decoding**: Uses `@base64d` to decode the base64url payload
6. **JSON Parsing**: Converts decoded string to JSON object with `fromjson`
7. **Optional Claim Navigation**: Uses jq path expressions for specific claim extraction

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
      call: jwt-parser
      with:
        token: ${ .headers.authorization }
        # Returns: complete JWT payload as JSON object
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
      call: jwt-parser
      with:
        token: ${ .headers.authorization }
        claimPath: ".sub"
        # Returns: "user-id-123"
  - extractNestedClaim:
      call: jwt-parser
      with:
        token: ${ .headers.authorization }
        claimPath: ".custom.department"
        # Returns: "engineering"
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
      call: jwt-parser
      with:
        token: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIn0.signature"
  - parseBearerToken:
      call: jwt-parser
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

**When `claimPath` is NOT provided** (complete payload):
```json
{
  "sub": "user-123",
  "preferred_username": "john.doe",
  "email": "john@example.com",
  "exp": 1234567890,
  "iat": 1234567800
}
```

**When `claimPath` is provided** (specific claim value):
```json
"user-123"
```
or
```json
"engineering"
```

## Technical Details

### JWT Token Structure
```
header.payload.signature
```
The function processes the **payload** section (index 1 after splitting on ".")

### jq Expression Breakdown

**JWT Decoding Expression:**
```jq
(if (.token | startswith("Bearer ")) then .token[7:] else .token end) |
split(".") |
if length != 3 then error("Invalid JWT format: must have 3 parts") else .[1] end |
@base64d |
fromjson
```

**Claim Path Expression:**
```jq
((if (.token | startswith("Bearer ")) then .token[7:] else .token end) |
split(".") |
if length != 3 then error("Invalid JWT format: must have 3 parts") else .[1] end |
@base64d |
fromjson) as $decoded |
if (.claimPath // null) != null then
  (.claimPath | split(".") | map(select(. != ""))) as $path |
  $decoded | getpath($path)
else
  $decoded
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
- **Path navigation**: Uses `getpath()` with string splitting for safe claim extraction
- **Prefix handling**: Conditional logic automatically detects and strips "Bearer " prefix
- **Pipeline processing**: Uses jq pipe operators for clean data transformation flow
- **Null safety**: Uses `has("claimPath")` to check for optional parameter presence

## Technical Validation

### JWT Structure Validation
The function expects standard JWT format: `header.payload.signature`
- **Header**: Algorithm and token type (ignored)
- **Payload**: Base64URL-encoded JSON claims (processed)  
- **Signature**: Cryptographic signature (ignored)

### Base64URL Decoding
JWT uses Base64URL encoding (RFC 4648 Section 5):
- Uses `-` and `_` instead of `+` and `/`
- No padding characters required
- jq's `@base64d` handles this automatically

### Claim Path Syntax
Claim paths follow jq object navigation:
- `.sub` - Direct property access
- `.custom.department` - Nested object access  
- `.roles[0]` - Array element access
- Path components split on `.` and filtered for non-empty strings
