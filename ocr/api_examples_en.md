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

Operations provided by the service:

- **`infer`**

    Obtain OCR results from an image.

    `POST /ocr`

    - Request body attributes:

        | Name | Type | Description | Required |
        |------|------|-------------|----------|
        |`image`|`string`|The URL of an image file accessible by the service or the Base64 encoded result of the image file content.|Yes|
        |`inferenceParams`|`object`|Inference parameters.|No|

        Attributes of `inferenceParams`:

        | Name | Type | Description | Required |
        |------|------|-------------|----------|
        |`maxLongSide`|`integer`|During inference, if the length of the longer side of the input image for the text detection model is greater than `maxLongSide`, the image will be scaled so that the length of the longer side equals `maxLongSide`.|No|

    - When the request is processed successfully, the `result` in the response body has the following attributes:

        | Name | Type | Description |
        |------|------|-------------|
        |`texts`|`array`|Positions, contents, and scores of texts.|
        |`image`|`string`|OCR result image with detected text positions annotated. The image is in JPEG format and encoded in Base64.|

        Each element in `texts` is an `object` with the following attributes:

        | Name | Type | Description |
        |------|------|-------------|
        |`poly`|`array`|Text position. Elements in the array are the vertex coordinates of the polygon enclosing the text.|
        |`text`|`string`|Text content.|
        |`score`|`number`|Text recognition score.|

        Example of `result`:

        ```json
        {
          "texts": [
            {
              "poly": [
                [
                  444,
                  244
                ],
                [
                  705,
                  244
                ],
                [
                  705,
                  311
                ],
                [
                  444,
                  311
                ]
              ],
              "text": "Beijing South Railway Station",
              "score": 0.9
            },
            {
              "poly": [
                [
                  992,
                  248
                ],
                [
                  1263,
                  251
                ],
                [
                  1263,
                  318
                ],
                [
                  992,
                  315
                ]
              ],
              "text": "Tianjin Railway Station",
              "score": 0.5
            }
          ],
          "image": "xxxxxx"
        }
        ```
