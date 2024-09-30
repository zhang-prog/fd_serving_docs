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

Operations provided by the service are as follows:

- **`infer`**

    Classify images.

    `POST /image-classification`

    - The request body attributes are as follows:

        | Name | Type | Description | Required |
        |------|------|-------------|----------|
        |`image`|`string`|The URL of an image file accessible by the service or the Base64 encoded result of the image file content.|Yes|
        |`inferenceParams`|`object`|Inference parameters.|No|

        The attributes of `inferenceParams` are as follows:

        | Name | Type | Description | Required |
        |------|------|-------------|----------|
        |`topK`|`integer`|Only the top `topK` categories with the highest scores will be retained in the results.|No|

    - When the request is processed successfully, the `result` of the response body has the following attributes:

        | Name | Type | Description |
        |------|------|-------------|
        |`categories`|`array`|Image category information.|
        |`image`|`string`|The image classification result image. The image is in JPEG format and encoded using Base64.|

        Each element in `categories` is an `object` with the following attributes:

        | Name | Type | Description |
        |------|------|-------------|
        |`id`|`integer`|Category ID.|
        |`name`|`string`|Category name.|
        |`score`|`number`|Category score.|

        An example of `result` is as follows:

        ```json
        {
          "categories": [
            {
              "id": 5,
              "name": "Rabbit",
              "score": 0.93
            }
          ],
          "image": "xxxxxx"
        }
        ```
