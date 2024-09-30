For all operations provided by the service:

- Both the response body and the request body for POST requests are JSON data (JSON objects).
- When the request is processed successfully, the response status code is `200`, and the response body attributes are as follows:

    | Name | Type | Description |
    |------|------|-------------|
    | `errorCode` | `integer` | Error code. Fixed as `0`. |
    | `errorMsg` | `string` | Error description. Fixed as `"Success"`. |

    The response body may also have a `result` attribute of type `object`, which stores the operation result information.

- When the request is not processed successfully, the response body attributes are as follows:

    | Name | Type | Description |
    |------|------|-------------|
    | `errorCode` | `integer` | Error code. Same as the response status code. |
    | `errorMsg` | `string` | Error description. |

Operations provided by the service are as follows:

- **`infer`**

    Performs object detection on an image.

    `POST /object-detection`

    - The request body attributes are as follows:

        | Name | Type | Description | Required |
        |------|------|-------------|----------|
        | `image` | `string` | The URL of an image file accessible by the service or the Base64 encoded result of the image file content. | Yes |

    - When the request is processed successfully, the `result` of the response body has the following attributes:

        | Name | Type | Description |
        |------|------|-------------|
        | `detectedObjects` | `array` | Information about the location and category of the detected objects. |
        | `image` | `string` | The image of the object detection result. The image is in JPEG format and encoded in Base64. |

        Each element in `detectedObjects` is an `object` with the following attributes:

        | Name | Type | Description |
        |------|------|-------------|
        | `bbox` | `array` | The location of the object. The elements in the array are the x-coordinate of the top-left corner, the y-coordinate of the top-left corner, the x-coordinate of the bottom-right corner, and the y-coordinate of the bottom-right corner of the bounding box, respectively. |
        | `categoryId` | `integer` | The ID of the object category. |
        | `score` | `number` | The score of the object. |

        An example of `result` is as follows:

        ```json
        {
          "detectedObjects": [
            {
              "bbox": [
                404.4967956542969,
                90.15770721435547,
                506.2465515136719,
                285.4187316894531
              ],
              "categoryId": 0,
              "score": 0.7418514490127563
            },
            {
              "bbox": [
                155.33145141601562,
                81.10954284667969,
                199.71136474609375,
                167.4235382080078
              ],
              "categoryId": 1,
              "score": 0.7328268885612488
            }
          ],
          "image": "xxxxxx"
        }
        ```