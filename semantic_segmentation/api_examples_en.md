For all operations provided by the service:

- Both the response body and the request body for POST requests are JSON data (JSON objects).
- When the request is processed successfully, the response status code is `200`, and the response body attributes are as follows:

    | Name | Type | Description |
    |------|------|-------------|
    |`errorCode`|`integer`|Error code. Fixed as `0`.|
    |`errorMsg`|`string`|Error description. Fixed as `"Success"`.|

    The response body may also have a `result` attribute of type `object`, which stores the operation result information.

- When the request is not processed successfully, the response body attributes are as follows:

    | Name | Type | Description |
    |------|------|-------------|
    |`errorCode`|`integer`|Error code. Same as the response status code.|
    |`errorMsg`|`string`|Error description.|

Operations provided by the service are as follows:

- **`infer`**

    Performs semantic segmentation on an image.

    `POST /semantic-segmentation`

    - The request body attributes are as follows:

        | Name | Type | Description | Required |
        |------|------|-------------|----------|
        |`image`|`string`|The URL of an image file accessible by the service or the Base64 encoded result of the image file content.|Yes|

    - When the request is processed successfully, the `result` of the response body has the following attributes:

        | Name | Type | Description |
        |------|------|-------------|
        |`labelMap`|`array`|Records the class label of each pixel in the image (arranged in row-major order).|
        |`size`|`array`|Image shape. The elements in the array are the height and width of the image, respectively.|
        |`image`|`string`|The semantic segmentation result image. The image is in JPEG format and encoded using Base64.|

        An example of `result` is as follows:

        ```json
        {
          "labelMap": [
            0,
            0,
            1,
            2
          ],
          "size": [
            2,
            2
          ],
          "image": "xxxxxx"
        }
        ```
