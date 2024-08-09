# Log Function

The Log function allows you to log messages in a Serverless Workflow. You can customize the log output, format, and more, providing flexibility for monitoring and debugging your workflows.

## Parameters

| Name | Type | Required | Description |
|:-----|:----:|:--------:|:------------|
| `message` | `string` | `yes` | The message to log. <br>The message can include placeholders that will be replaced by values from `arguments`. |
| `level` | `string` | `no` | The severity level of the log message. <br>Supported values are: `trace`, `debug`, `information`, `warning`, `error`, `critical`. <br>Default to `information`. |
| `context` | `string` | `no` | The log's source context. |
| `arguments` | `map` | `no` | A key/value mapping used to interpolate the log message.<br>Placeholders in the `message` string will be replaced by corresponding values from this object. |
| `output` | `string` | `no` | The path to the file to append the log message to.<br>If not set, the log will be written to `STDOUT`. |
| `timestamp` | `boolean` | `no` | Determines whether or not to include a timestamp in the log message. |
| `format` | `string` | `no` | A format string for the log message.<br>It supports placeholders `{TIMESTAMP}`, `{LEVEL}`, `{CONTEXT}`, and `{MESSAGE}`.<br>Default to `'{TIMESTAMP} [{LEVEL}] ({CONTEXT}): {MESSAGE}'`. |

## Usage

Below is an example of how to use the **Log** custom function in a Serverless Workflow:

```yaml
document:
  dsl: '1.0.0'
  namespace: samples
  name: use-cataloged-function
  version: '1.0.0'
do:
  - log:
      call: log:1.0.0                                           #could also be called using the function's url instead: https://github.com/serverlessworkflow/catalog/functions/log/1.0.0
      with:
        message: Processing file {filename} in task {task_id}
        level: debug                                            #if not set, would default to 'information'
        context: FileProcessingTask                             
        arguments:
          filename: data.csv
          task_id: task-1234
        output: /var/log/workflow.log                           #if not set, the log would be printed to STDOUT
        timestamp: true
        format: '{TIMESTAMP} [{LEVEL}] ({CONTEXT}): {MESSAGE}'
```
