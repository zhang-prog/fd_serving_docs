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

    Performs instance segmentation on an image.

    `POST /instance-segmentation`

    - The request body attributes are as follows:

        | Name | Type | Description | Required |
        |------|------|-------------|----------|
        |`image`|`string`|The URL of an image file accessible by the service or the Base64 encoded result of the image file content.|Yes|

    - When the request is processed successfully, the `result` of the response body has the following attributes:

        | Name | Type | Description |
        |------|------|-------------|
        |`instances`|`array`|Information about the locations and categories of instances.|
        |`image`|`string`|The result image of instance segmentation. The image is in JPEG format and encoded in Base64.|

        Each element in `instances` is an `object` with the following attributes:

        | Name | Type | Description |
        |------|------|-------------|
        |`bbox`|`array`|The location of the instance. The elements in the array are the x-coordinate of the top-left corner, the y-coordinate of the top-left corner, the x-coordinate of the bottom-right corner, and the y-coordinate of the bottom-right corner of the bounding box, respectively.|
        |`categoryId`|`integer`|The ID of the instance category.|
        |`score`|`number`|The score of the instance.|
        |`mask`|`object`|The segmentation mask of the instance.|

        The attributes of `mask` are as follows:

        | Name | Type | Description |
        |------|------|-------------|
        |`rleResult`|`str`|The run-length encoding result of the mask.|
        |`size`|`array`|The shape of the mask. The elements in the array are the height and width of the mask, respectively.|

        An example of `result` is as follows:

        ```json
        {
          "instances": [
            {
              "bbox": [
                162.39381408691406,
                83.88176727294922,
                624.0797119140625,
                343.4986877441406
              ],
              "categoryId": 33,
              "score": 0.8691174983978271,
              "mask": {
                "rleResult": "xxxxxx",
                "size": [
                  259,
                  462
                ]
              }
            }
          ],
          "image": "xxxxxx"
        }
        ```
