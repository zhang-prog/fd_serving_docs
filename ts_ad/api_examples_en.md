For all operations provided by the service:

- Both the response body and the request body for POST requests are JSON data (JSON objects).
- When the request is processed successfully, the response status code is `200`, and the attributes of the response body are as follows:

    | Name | Type | Description |
    |------|------|-------------|
    |`errorCode`|`integer`|Error code. Fixed as `0`.|
    |`errorMsg`|`string`|Error description. Fixed as `"Success"`.|

    The response body may also have a `result` attribute of type `object`, which stores the operation result information.

- When the request is not processed successfully, the attributes of the response body are as follows:

    | Name | Type | Description |
    |------|------|-------------|
    |`errorCode`|`integer`|Error code. Same as the response status code.|
    |`errorMsg`|`string`|Error description.|

Operations provided by the service:

- **`infer`**

    Performs time-series anomaly detection.

    `POST /time-series-anomaly-detection`

    - Attributes of the request body:

        | Name | Type | Description | Required |
        |------|------|-------------|----------|
        |`csv`|`string`|The URL of a CSV file accessible by the service or the Base64 encoded result of the CSV file content. The CSV file must be encoded in UTF-8.|Yes|

    - When the request is processed successfully, the `result` of the response body has the following attributes:

        | Name | Type | Description |
        |------|------|-------------|
        |`csv`|`string`|Time-series anomaly detection results in CSV format. Encoded in UTF-8+Base64.|

        An example of `result` is as follows:

        ```json
        {
          "csv": "xxxxxx"
        }
        ```