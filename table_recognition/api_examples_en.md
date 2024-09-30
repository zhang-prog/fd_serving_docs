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

    Locate and recognize tables in images.

    `POST /table-recognition`

    - Request body attributes:

        | Name | Type | Description | Required |
        |------|------|-------------|----------|
        |`image`|`string`|The URL of an image file accessible by the service or the Base64 encoded result of the image file content.|Yes|
        |`inferenceParams`|`object`|Inference parameters.|No|

        Attributes of `inferenceParams`:

        | Name | Type | Description | Required |
        |------|------|-------------|----------|
        |`maxLongSide`|`integer`|During inference, if the length of the longer side of the input image for the text detection model is greater than `maxLongSide`, the image will be scaled so that the length of the longer side equals `maxLongSide`.|No|

    - When the request is processed successfully, the `result` of the response body has the following attributes:

        | Name | Type | Description |
        |------|------|-------------|
        |`tables`|`array`|Positions and contents of tables.|
        |`layoutImage`|`string`|Layout area detection result image. The image is in JPEG format and encoded using Base64.|
        |`ocrImage`|`string`|OCR result image. The image is in JPEG format and encoded using Base64.|

        Each element in `tables` is an `object` with the following attributes:

        | Name | Type | Description |
        |------|------|-------------|
        |`bbox`|`array`|Table position. The elements in the array are the x-coordinate of the top-left corner, the y-coordinate of the top-left corner, the x-coordinate of the bottom-right corner, and the y-coordinate of the bottom-right corner of the bounding box, respectively.|
        |`html`|`string`|Table recognition result in HTML format.|
