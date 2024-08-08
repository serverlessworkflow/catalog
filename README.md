# Serverless Workflow Catalog

This repository is dedicated to storing and managing custom functions, also known as extensions, that enhance the capabilities of the Serverless Workflow Domain Specific Language (DSL). This repository serves as a central hub where developers can contribute, discover, and utilize various extensions designed to extend the native functionality of the Serverless Workflow DSL. These extensions enable users to implement specific tasks, custom logic, or integrations that are not covered by the core language features.

For information on how to create a custom function, please refer to the [documentation](https://github.com/serverlessworkflow/specification/blob/main/dsl.md#creating-a-custom-function).

## Features

- **Collection of Custom Functions**: The repository hosts a diverse collection of custom functions contributed by the community. These functions can be used to perform specialized tasks, integrate with external services, or apply custom logic within workflows.

- **Modular and Reusable**: Each function in the catalog is designed to be modular and reusable, allowing developers to easily incorporate them into their workflows without extensive modifications.

- **Documentation and Examples**: Detailed documentation and usage examples are provided for each function. This helps users understand how to integrate and use the functions effectively within their workflows.

- **Community Contributions**: The repository encourages contributions from the Serverless Workflow community. Developers can submit their custom functions for inclusion in the catalog, fostering a collaborative environment.

- **Version Control and Updates**: The catalog maintains version control for each function, ensuring that users can access the latest updates and improvements. This also allows for backward compatibility and stable integration in existing workflows.

- **Discoverability**: Users can easily search and browse through the catalog to find functions that meet their specific needs. The organized structure and tagging system facilitate quick discovery of relevant extensions.

By providing a centralized repository for custom functions, the "catalog" repository enhances the flexibility and power of the Serverless Workflow DSL, enabling users to create more complex and tailored workflow solutions.

## Structure

Cataloged functions must follow the following structure:

```
./functions
    /log                        👈 The name of the custom function. It must be lowercase and contain exclusively alphanumeric characters, except for '-'.
        /1.0.0                  👈 The semantic version of the custom function.
            /function.yaml      👈 The function's definition.
            /OWNERS.md          👈 Describes the owners/authors of the custom function.
            /README.md          👈 Describes the custom function.
        /2.0.0
            ...
```

## Usage

Below is an example of how to use a custom function in a Serverless Workflow:

```yaml
document:
  dsl: '1.0.0'
  namespace: samples
  name: use-cataloged-function
  version: '1.0.0'
do:
  - callCatalogedFunction:
      call: https://github.com/serverlessworkflow/catalog/functions/log/1.0.0 #link to the repository directory that contains the function.yaml of the custom function to call.
      #call: log:1.0.0 <--- this would also be a correct way to reference the function, as it is registered in the official catalog.
      with:
        message: Hello, world!
```

## Contributing

We welcome contributions from the community! If you would like to contribute to the Serverless Workflow Catalog, please follow our contribution guidelines.

For detailed instructions on how to contribute, please refer to the [CONTRIBUTING.md](CONTRIBUTING.md) file.

Thank you for helping us improve the Serverless Workflow Catalog!