---
tags: [silia, sam, aws, patterns, infrastructure]
created: 2026-07-03
---

# SAM Template Patterns

## Handler Naming Convention

All Lambda handlers must use the `Min` suffix because webpack outputs `{name}Min.js`:

```yaml
# WRONG
Handler: createGrant.handler

# CORRECT
Handler: createGrantMin.handler
```

## Region References

Use `${AWS::Region}` (CloudFormation pseudo-parameter) instead of `${Region}` (template parameter) in:
- KMS ViaService conditions
- API endpoint outputs
- Any ARN constructions

```yaml
# WRONG
kms:ViaService: !Sub dynamodb.${Region}.amazonaws.com

# CORRECT
kms:ViaService: !Sub dynamodb.${AWS::Region}.amazonaws.com
```

## BasePathMapping + Lambda Paths

When using `BasePathMapping`, Lambda paths should NOT include the module prefix:

```yaml
# BasePathMapping
BasePath: access  # Adds /access to all routes

# Lambda paths - DON'T include /access
Path: /grants           # Results in /access/grants ✓
Path: /grants/{grantId} # Results in /access/grants/{id} ✓

# WRONG - causes double prefix
Path: /access/grants    # Results in /access/access/grants ✗
```

## KMS Key Policy for DynamoDB

Always include `kms:CreateGrant` for DynamoDB SSE:

```yaml
Action:
  - kms:Encrypt
  - kms:Decrypt
  - kms:ReEncrypt*
  - kms:GenerateDataKey*
  - kms:DescribeKey
  - kms:CreateGrant  # Required for DynamoDB
```

## Import Case Sensitivity

Import paths must match exact file casing. macOS is case-insensitive but Linux CI is case-sensitive:

```typescript
// WRONG (fails on Linux)
import { UserModel } from '../../../../User/domain/models/User.model';

// CORRECT (file is actually user.model.ts)
import { UserModel } from '../../../../User/domain/models/user.model';
```

## Standard Template Structure

```
Parameters
Globals
  Function
    Environment Variables
    VpcConfig
Conditions
Resources
  Api (API Gateway)
  BasePathMapping (for custom domain)
  DynamoDB Tables
  KMS Keys
  SQS Queues
  Lambda Functions
Outputs
```

---
Related: [[Tables Refactor Deployment Runbook]]