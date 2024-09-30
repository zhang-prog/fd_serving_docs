For all operations provided by the service:

- Both the response body and the request body for POST requests are JSON data (JSON objects).
- When the request is processed successfully, the response status code is `200`, and the response body attributes are as follows:

    | Name | Type | Description |
    |------|------|-------------|
    |`errorCode`|`integer`|Error code. Fixed as `0`.|
    |`errorMsg`|`string`|Error message. Fixed as `"Success"`.|

    The response body may also have a `result` attribute of type `object`, which stores the operation result information.

- When the request is not processed successfully, the response body attributes are as follows:

    | Name | Type | Description |
    |------|------|-------------|
    |`errorCode`|`integer`|Error code. Same as the response status code.|
    |`errorMsg`|`string`|Error message.|

Operations provided by the service:

- **`infer`**

    Classify time-series data.

    `POST /time-series-classification`

    - The request body attributes are as follows:

        | Name | Type | Description | Required |
        |------|------|-------------|----------|
        |`csv`|`string`|The URL of a CSV file accessible by the service or the Base64 encoded result of the CSV file content. The CSV file must be encoded in UTF-8.|Yes|

    - When the request is processed successfully, the `result` in the response body has the following attributes:

        | Name | Type | Description |
        |------|------|-------------|
        |`label`|`string`|Class label.|
        |`score`|`number`|Class score.|

        An example of `result` is as follows:

        ```json
        {
          "label": "running",
          "score": 0.97
        }
        ```
