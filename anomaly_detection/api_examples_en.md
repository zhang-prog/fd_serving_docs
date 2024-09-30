For all operations provided by the service:

- Both the response body and the request body for POST requests are JSON data (JSON objects).
- When the request is processed successfully, the response status code is `200`, and the response body attributes are as follows:

    |Name|Type|Description|
    |-|-|-|
    |`errorCode`|`integer`|Error code. Fixed as `0`.|
    |`errorMsg`|`string`|Error message. Fixed as `"Success"`.|

    The response body may also have a `result` attribute of type `object`, which stores the operation result information.

- When the request is not processed successfully, the response body attributes are as follows:

    |Name|Type|Description|
    |-|-|-|
    |`errorCode`|`integer`|Error code. Same as the response status code.|
    |`errorMsg`|`string`|Error message.|

Operations provided by the service:

- **`infer`**

    Performs anomaly detection on images.

    `POST /anomaly-detection`

    - Request body attributes:

        |Name|Type|Description|Required|
        |-|-|-|-|
        |`image`|`string`|The URL of the image file accessible by the service or the Base64 encoded result of the image file content.|Yes|

    - When the request is processed successfully, the `result` of the response body has the following attributes:

        |Name|Type|Description|
        |-|-|-|
        |`labelMap`|`array`|Records the class label of each pixel in the image (arranged in row-major order). Where `255` represents an anomaly point, and `0` represents a non-anomaly point.|
        |`size`|`array`|Image shape. The elements in the array are the height and width of the image in order.|
        |`image`|`string`|Anomaly detection result image. The image is in JPEG format and encoded in Base64.|

        Example of `result`:

        ```json
        {
          "labelMap": [
            0,
            0,
            255,
            0
          ],
          "size": [
            2,
            2
          ],
          "image": "xxxxxx"
        }
        ```
