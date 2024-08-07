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
    /log                        👈 The name of the custom function. Must be lowercased and contain exclusively alphanumeric characters, with the exception of '-'.
        /1.0.0                  👈 The semantic version of the custom function.
            /function.yaml      👈 The function's definition.
            /OWNERS.md          👈 Describes the owners/authors of the custom function.
            /README.md          👈 Describes the custom function.
        /2.0.0
            ...
```

## Usage

Below is an example of how to usea custom function in a Serverless Workflow:

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

We welcome contributions to the Serverless Workflow Catalog! If you have developed a custom function that you believe would be a valuable addition to the catalog, please follow the steps below to submit it:

1. **Fork the Repository**
   - Create a fork of the Serverless Workflow Catalog repository on GitHub. This will allow you to make changes without affecting the main repository.

2. **Clone Your Fork**
   - Clone your forked repository to your local development environment:
     ```bash
     git clone https://github.com/YOUR_USERNAME/catalog.git
     ```

3. **Create a New Directory for Your Function**
   - Inside the `./functions` directory, create a new directory for your custom function. Use a lowercase name with alphanumeric characters and hyphens (`-`), and include the version number:
     ```plaintext
     ./functions
         /my-custom-function
             /1.0.0
                 /function.yaml
                 /OWNERS.md
                 /README.md
     ```

4. **Define Your Function**
   - Create a `function.yaml` file inside the versioned directory with the definition of your custom function. Refer to the [documentation](https://github.com/serverlessworkflow/specification/blob/main/dsl.md#creating-a-custom-function) for details on how to structure this file.

5. **Create Documentation**
   - Write a `README.md` file that describes your custom function, including its purpose, parameters, usage examples, and any other relevant information.

6. **Specify Ownership**
   - Create an `OWNERS.md` file to list the individuals or teams responsible for maintaining the custom function. Include their names and contact information.

7. **Test Your Function**
   - Test your custom function to ensure it works as expected. Validate that it integrates properly with the Serverless Workflow DSL.

8. **Push Your Changes**
   - Commit your changes and push them to your forked repository:
     ```bash
     git add .
     git commit -m "Add new custom function my-custom-function version 1.0.0"
     git push origin main
     ```

9. **Submit a Pull Request**
   - Go to the original Serverless Workflow Catalog repository on GitHub and open a pull request to merge your changes from your fork. Provide a clear description of your changes and any relevant details about the custom function.

10. **Review and Feedback**
    - The repository maintainers will review your pull request. They may provide feedback or request changes. Be responsive to any comments or suggestions.

By following these steps and guidelines, you contribute to the growing catalog of custom functions, enhancing the capabilities of the Serverless Workflow DSL for the community.

For any questions or support, please contact the repository maintainers or refer to the [contribution guidelines](https://github.com/serverlessworkflow/catalog/blob/main/CONTRIBUTING.md).