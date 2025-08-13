# JWT Parser Function

The JWT Parser function allows Serverless Workflow 1.x workflows to parse and extract information from JWT (JSON Web Token) tokens using jq expressions. This function decodes the JWT payload and optionally extracts specific claims.

## Overview

This function uses a `set` task with jq expressions to:
- Decode JWT tokens (with or without "Bearer " prefix)
- Extract the complete JWT payload as JSON
- Optionally extract specific claims using jq paths

## Usage

### Basic JWT Parsing (Complete Payload)

```yaml
document:
  dsl: 1.0.0-alpha1
  namespace: examples
  name: jwt-parsing
  version: 1.0.0
do:
  - parseToken:
      use: jwt-parser
      with:
        token: ${ .headers.authorization }
```

### Extract Specific Claims

```yaml
document:
  dsl: 1.0.0-alpha1
  namespace: examples  
  name: jwt-user-extraction
  version: 1.0.0
do:
  - extractUsername:
      use: jwt-parser
      with:
        token: ${ .headers.authorization }
        claimPath: ".preferred_username"
  - extractEmail:
      use: jwt-parser
      with:
        token: ${ .headers.authorization }
        claimPath: ".email"
```

### Multiple Claim Extraction

```yaml
document:
  dsl: 1.0.0-alpha1
  namespace: examples
  name: jwt-multi-claims
  version: 1.0.0
do:
  - getUserInfo:
      use: jwt-parser
      with:
        token: ${ .headers["x-authorization-acme_financial_auth"] }
  - processUserData:
      use: set
      set:
        username: ${ .result.preferred_username }
        email: ${ .result.email }
        userId: ${ .result.sub }
        message: ${ "Welcome " + .username + "! Your request has been processed." }
```

## Complete Example - Loan Approval with User Personalization

```yaml
document:
  dsl: 1.0.0-alpha1
  namespace: examples
  name: loan-approval-jwt
  version: 1.0.0
do:
  - extractUserInfo:
      use: jwt-parser
      with:
        token: ${ .headers["x-authorization-acme_financial_auth"] }
  - processLoanApproval:
      use: set
      set:
        user: ${ .result.preferred_username }
        userId: ${ .result.sub }
        email: ${ .result.email }
        loanApproved: true
        message: ${ "Congrats " + .user + "! Your loan has been approved!" }
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `token` | string | Yes | The JWT token to parse (Bearer prefix will be automatically handled) |
| `claimPath` | string | No | jq path to extract specific claim (e.g., ".sub", ".preferred_username", ".email") |

## Output

The function returns:
- `claims`: The complete decoded JWT payload as JSON object
- `result`: Either the complete payload (if no claimPath) or the specific claim value (if claimPath provided)

## Token Format Support

The function supports JWT tokens in various formats:
- Raw JWT token: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`
- Bearer token: `Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

## jq Expression Details

The function uses these jq expressions:
- **Token cleanup**: Removes "Bearer " prefix if present
- **JWT decoding**: Splits token, extracts payload (part 1), base64 decodes, and parses JSON
- **Claim extraction**: Uses jq path navigation to extract specific claims

## Common Claim Paths

- `.sub` - Subject (user ID)
- `.preferred_username` - Username  
- `.email` - Email address
- `.name` - Full name
- `.given_name` - First name
- `.family_name` - Last name
- `.roles` - User roles array
- `.exp` - Expiration timestamp
- `.iat` - Issued at timestamp

## Error Handling

The jq expressions will fail if:
- Token is null or empty
- Token format is invalid (not 3 parts separated by dots)
- JWT payload is not valid base64 or JSON
- Specified claimPath does not exist

## Security Note

This function extracts claims from JWT tokens without signature verification. In production environments, ensure proper token validation is performed at the API gateway or authentication layer before tokens reach the workflow.
