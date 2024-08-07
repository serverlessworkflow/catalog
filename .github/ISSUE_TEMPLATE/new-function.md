---
name: New Function Request
about: Propose a new function to add to the Serverless Workflow Catalog
labels: feature
---

# New Function Proposal

Thank you for proposing a new function for the Serverless Workflow Catalog! To help us evaluate and integrate your proposal, please provide as much detail as possible by filling out the following template.

## Function Overview

**Function Name:**

<!-- Provide the name of the proposed custom function. Use lowercase and hyphens only (e.g., `my-new-function`). -->

**Function Version:**

<!-- Specify the initial version of the function (e.g., `1.0.0`). -->

**Function Description:**

<!-- Provide a brief description of what the function does and its intended use case. -->

## Function Definition

**Parameters:**

<!-- List and describe the parameters that the function will accept. Include details about types, required fields, default values, and any constraints. -->

**Return Values:**

<!-- Describe what the function returns and the type of the return value. -->

**Example Usage:**

```yaml
# Provide an example of how the function would be used in a Serverless Workflow. Include sample input and expected output.

document:
  dsl: '1.0.0'
  namespace: samples
  name: use-new-function
  version: '1.0.0'
do:
  - callNewFunction:
      call: my-new-function:1.0.0
      with:
        parameter1: value1
        parameter2: value2
