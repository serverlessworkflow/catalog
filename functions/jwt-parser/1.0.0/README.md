# JWT Parser Function

The JWT Parser function allows SonataFlow workflows to parse and extract information from JWT (JSON Web Token) tokens. This is particularly useful for accessing user information and claims from authentication tokens passed in workflow headers.

## Overview

This function provides three main operations:
- **parse**: Extract the complete JWT payload as a JSON object
- **extractUser**: Extract standard user information from JWT claims (sub, preferred_username, email, etc.)
- **extractClaim**: Extract a specific claim by name

## Usage

### Basic JWT Parsing

```json
{
  "functions": [
    {
      "name": "parseJWT",
      "type": "custom",
      "operation": "jwt-parser"
    }
  ],
  "states": [
    {
      "name": "parseToken",
      "type": "operation",
      "actions": [
        {
          "name": "parseAction",
          "functionRef": {
            "refName": "parseJWT",
            "arguments": {
              "token": "${ $WORKFLOW.headers.\"Authorization\" }",
              "operation": "parse"
            }
          }
        }
      ]
    }
  ]
}
```

### Extract User Information

```json
{
  "functions": [
    {
      "name": "extractUser",
      "type": "custom",
      "operation": "jwt-parser:extractUser"
    }
  ],
  "states": [
    {
      "name": "extractUserName",
      "type": "operation",
      "actions": [
        {
          "name": "extractUserAction",
          "functionRef": {
            "refName": "extractUser",
            "arguments": {
              "token": "${ $WORKFLOW.headers.\"X-Authorization-acme_financial_auth\" }"
            }
          }
        }
      ],
      "stateDataFilter": {
        "output": "${ { user: .result.preferred_username } }"
      }
    }
  ]
}
```

### Extract Specific Claim

```json
{
  "functions": [
    {
      "name": "extractClaim",
      "type": "custom", 
      "operation": "jwt-parser:extractClaim"
    }
  ],
  "states": [
    {
      "name": "extractRole",
      "type": "operation",
      "actions": [
        {
          "name": "extractRoleAction",
          "functionRef": {
            "refName": "extractClaim",
            "arguments": {
              "token": "${ $WORKFLOW.headers.\"Authorization\" }",
              "claim": "role"
            }
          }
        }
      ]
    }
  ]
}
```

## Complete Example

Here's a complete workflow that demonstrates JWT parsing for user personalization:

```json
{
  "id": "jwt_example",
  "version": "1.0",
  "name": "JWT Token Processing Example",
  "start": "extractUser",
  "functions": [
    {
      "name": "extractUser",
      "type": "custom",
      "operation": "jwt-parser:extractUser"
    }
  ],
  "states": [
    {
      "name": "extractUser",
      "type": "operation",
      "actions": [
        {
          "name": "extractUserAction",
          "functionRef": {
            "refName": "extractUser",
            "arguments": {
              "token": "${ $WORKFLOW.headers.\"X-Authorization-acme_financial_auth\" }"
            }
          }
        }
      ],
      "stateDataFilter": {
        "output": "${ { user: .result.preferred_username } }"
      },
      "transition": "personalizedResponse"
    },
    {
      "name": "personalizedResponse",
      "type": "inject",
      "data": {
        "approved": true
      },
      "stateDataFilter": {
        "output": "${ { message: \"Congrats \\(.user)! Your request has been approved!\", approved } }"
      },
      "end": true
    }
  ]
}
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `token` | string | Yes | The JWT token to parse (Bearer prefix will be automatically removed) |
| `operation` | string | No | Operation to perform: "parse", "extractUser", or "extractClaim" (default: "parse") |
| `claim` | string | No | Name of specific claim to extract (required when operation is "extractClaim") |

## Output

The function returns a JSON object containing:
- For `parse`: Complete JWT payload
- For `extractUser`: Standard user claims (sub, preferred_username, email, name, etc.)
- For `extractClaim`: The specific claim value

## Token Format Support

The function supports JWT tokens in various formats:
- Raw JWT token: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`
- Bearer token: `Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

## Error Handling

The function will fail if:
- Token is null or empty
- Token format is invalid
- Required claim parameter is missing for extractClaim operation

## Security Note

This function extracts claims from JWT tokens without signature verification. In production environments, ensure proper token validation is performed at the API gateway or authentication layer before tokens reach the workflow.
