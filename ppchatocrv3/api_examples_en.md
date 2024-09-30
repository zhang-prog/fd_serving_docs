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

- **`analyzeImage`**

    Analyzes images using computer vision models to obtain OCR, table recognition results, etc., and extracts key information from the images.

    `POST /chatocr-vision`

    - Request body attributes:

        | Name | Type | Description | Required |
        |------|------|-------------|----------|
        | `image` | `string` | The URL of an image file or PDF file accessible by the service, or the Base64 encoded result of the content of the above-mentioned file types. For PDF files with more than 10 pages, only the content of the first 10 pages will be used. | Yes |
        | `fileType` | `integer` | File type. `0` indicates a PDF file, `1` indicates an image file. If this attribute is not present in the request body, the service will attempt to automatically infer the file type based on the URL. | No |
        | `useOricls` | `boolean` | Whether to enable document image orientation classification. This feature is enabled by default. | No |
        | `useCurve` | `boolean` | Whether to enable seal text detection. This feature is enabled by default. | No |
        | `useUvdoc` | `boolean` | Whether to enable text image correction. This feature is enabled by default. | No |
        | `inferenceParams` | `object` | Inference parameters. | No |

        Attributes of `inferenceParams`:

        | Name | Type | Description | Required |
        |------|------|-------------|----------|
        | `maxLongSide` | `integer` | During inference, if the length of the longer side of the input image for the text detection model is greater than `maxLongSide`, the image will be scaled so that the length of the longer side equals `maxLongSide`. | No |

    - When the request is processed successfully, the `result` of the response body has the following attributes:

        | Name | Type | Description |
        |------|------|-------------|
        | `visionResults` | `array` | Analysis results obtained using computer vision models. The array length is 1 (for image input) or the smaller of the number of document pages and 10 (for PDF input). For PDF input, each element in the array represents the processing result of each page in the PDF file. |
        | `visionInfo` | `object` | Key information in the image, which can be used as input for other operations. |

        Each element in `visionResults` is an `object` with the following attributes:

        | Name | Type | Description |
        |------|------|-------------|
        | `texts` | `array` | Text positions, contents, and scores. |
        | `tables` | `array` | Table positions and contents. |
        | `inputImage` | `string` | Input image. The image is in JPEG format and encoded using Base64. |
        | `ocrImage` | `string` | OCR result image. The image is in JPEG format and encoded using Base64. |
        | `layoutImage` | `string` | Layout area detection result image. The image is in JPEG format and encoded using Base64. |

        Each element in `texts` is an `object` with the following attributes:

        | Name | Type | Description |
        |------|------|-------------|
        | `poly` | `array` | Text position. The elements in the array are the vertex coordinates of the polygon enclosing the text in```markdown
### chat

Interact with large language models to extract key information.

`POST /chatocr-vision`

- Request body attributes:

    | Name | Type | Description | Required |
    |------|------|-------------|----------|
    |`keys`|`array`|List of keywords.|Yes|
    |`visionInfo`|`object`|Key information from the image. Provided by the `analyzeImage` operation.|Yes|
    |`taskDescription`|`string`|Task prompt.|No|
    |`rules`|`string`|Extraction rules. Used to customize the information extraction rules, e.g., to specify output formats.|No|
    |`fewShot`|`string`|Example prompts.|No|
    |`useVectorStore`|`boolean`|Whether to enable the vector database. Enabled by default.|No|
    |`vectorStore`|`object`|Serialized result of the vector database. Provided by the `buildVectorStore` operation.|No|
    |`retrievalResult`|`string`|Knowledge retrieval result. Provided by the `retrieveKnowledge` operation.|No|
    |`returnPrompts`|`boolean`|Whether to return the prompts used. Enabled by default.|No|
    |`llmName`|`string`|Name of the large language model.|No|
    |`llmParams`|`object`|API parameters for the large language model.|No|

    Currently, `llmParams` can take the following form:

    ```json
    {
      "apiType": "qianfan",
      "apiKey": "{Qianfan Platform API key}",
      "secretKey": "{Qianfan Platform secret key}"
    }
    ```

- On successful request processing, the `result` in the response body has the following attributes:

    | Name | Type | Description |
    |------|------|-------------|
    |`chatResult`|`string`|Extracted key information result.|
    |`prompts`|`object`|Prompts used.|

    Attributes of `prompts`:

    | Name | Type | Description |
    |------|------|-------------|
    |`ocr`|`string`|OCR prompt.|
    |`table`|`string`|Table prompt.|
    |`html`|`string`|HTML prompt.|
