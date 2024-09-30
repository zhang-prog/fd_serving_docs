<details>  
<summary>API参考</summary>  
  
对于服务提供的所有操作：

- 响应体以及POST请求的请求体均为JSON数据（JSON对象）。
- 当请求处理成功时，响应状态码为`200`，响应体的属性如下：

    |名称|类型|含义|
    |-|-|-|
    |`errorCode`|`integer`|错误码。固定为`0`。|
    |`errorMsg`|`string`|错误说明。固定为`"Success"`。|

    响应体还可能有`result`属性，类型为`object`，其中存储操作结果信息。

- 当请求处理未成功时，响应体的属性如下：

    |名称|类型|含义|
    |-|-|-|
    |`errorCode`|`integer`|错误码。与响应状态码相同。|
    |`errorMsg`|`string`|错误说明。|

服务提供的操作如下：

- **`analyzeImage`**

    使用计算机视觉模型对图像进行分析，获得OCR、表格识别结果等，并提取图像中的关键信息。

    `POST /chatocr-vision`

    - 请求体的属性如下：

        |名称|类型|含义|是否必填|
        |-|-|-|-|
        |`image`|`string`|服务可访问的图像文件或PDF文件的URL，或上述类型文件内容的Base64编码结果。对于超过10页的PDF文件，只有前10页的内容会被使用。|是|
        |`fileType`|`integer`|文件类型。`0`表示PDF文件，`1`表示图像文件。若请求体无此属性，则服务将尝试根据URL自动推断文件类型。|否|
        |`useOricls`|`boolean`|是否启用文档图像方向分类功能。默认启用该功能。|否|
        |`useCurve`|`boolean`|是否启用印章文本检测功能。默认启用该功能。|否|
        |`useUvdoc`|`boolean`|是否启用文本图像矫正功能。默认启用该功能。|否|
        |`inferenceParams`|`object`|推理参数。|否|

        `inferenceParams`的属性如下：

        |名称|类型|含义|是否必填|
        |-|-|-|-|
        |`maxLongSide`|`integer`|推理时，若文本检测模型的输入图像较长边的长度大于`maxLongSide`，则将对图像进行缩放，使其较长边的长度等于`maxLongSide`。|否|

    - 请求处理成功时，响应体的`result`具有如下属性：

        |名称|类型|含义|
        |-|-|-|
        |`visionResults`|`array`|使用计算机视觉模型得到的分析结果。数组长度为1（对于图像输入）或文档页数与10中的较小者（对于PDF输入）。对于PDF输入，数组中的每个元素依次表示PDF文件中每一页的处理结果。|
        |`visionInfo`|`object`|图像中的关键信息，可用作其他操作的输入。|

        `visionResults`中的每个元素为一个`object`，具有如下属性：

        |名称|类型|含义|
        |-|-|-|
        |`texts`|`array`|文本位置、内容和得分。|
        |`tables`|`array`|表格位置和内容。|
        |`inputImage`|`string`|输入图像。图像为JPEG格式，使用Base64编码。|
        |`ocrImage`|`string`|OCR结果图。图像为JPEG格式，使用Base64编码。|
        |`layoutImage`|`string`|版面区域检测结果图。图像为JPEG格式，使用Base64编码。|

        `texts`中的每个元素为一个`object`，具有如下属性：

        |名称|类型|含义|
        |-|-|-|
        |`poly`|`array`|文本位置。数组中元素依次为包围文本的多边形的顶点坐标。|
        |`text`|`string`|文本内容。|
        |`score`|`number`|文本识别得分。|

        `tables`中的每个元素为一个`object`，具有如下属性：

        |名称|类型|含义|
        |-|-|-|
        |`bbox`|`array`|表格位置。数组中元素依次为边界框左上角x坐标、左上角y坐标、右下角x坐标以及右下角y坐标。|
        |`html`|`string`|HTML格式的表格识别结果。|

- **`buildVectorStore`**

    构建向量数据库。

    `POST /chatocr-vector`

    - 请求体的属性如下：

        |名称|类型|含义|是否必填|
        |-|-|-|-|
        |`visionInfo`|`object`|图像中的关键信息。由`analyzeImage`操作提供。|是|
        |`minChars`|`integer`|启用向量数据库的最小数据长度。|否|
        |`llmRequestInterval`|`number`|调用大语言模型API的间隔时间。|否|
        |`llmName`|`string`|大语言模型名称。|否|
        |`llmParams`|`object`|大语言模型API参数。|否|

        当前，`llmParams`可以采用如下两种形式之一：
        
        ```json
        {
          "apiType": "qianfan",
          "apiKey": "{千帆平台API key}",
          "secretKey": "{千帆平台secret key}"
        }
        ```

        ```json
        {
          "apiType": "{aistudio}",
          "accessToken": "{AI Studio访问令牌}"
        }
        ```

    - 请求处理成功时，响应体的`result`具有如下属性：

        |名称|类型|含义|
        |-|-|-|
        |`vectorStore`|`object`|向量数据库序列化结果，可用作其他操作的输入。|

- **`retrieveKnowledge`**

    进行知识检索。

    `POST /chatocr-retrieval`

    - 请求体的属性如下：

        |名称|类型|含义|是否必填|
        |-|-|-|-|
        |`keys`|`array`|关键词列表。|是|
        |`vectorStore`|`object`|向量数据库序列化结果。由`buildVectorStore`操作提供。|是|
        |`visionInfo`|`object`|图像中的关键信息。由`analyzeImage`操作提供。|是|
        |`llmName`|`string`|大语言模型名称。|否|
        |`llmParams`|`object`|大语言模型API参数。|否|

        当前，`llmParams`可以采用如下两种形式之一：
        
        ```json
        {
          "apiType": "qianfan",
          "apiKey": "{千帆平台API key}",
          "secretKey": "{千帆平台secret key}"
        }
        ```

        ```json
        {
          "apiType": "{aistudio}",
          "accessToken": "{AI Studio访问令牌}"
        }
        ```

    - 请求处理成功时，响应体的`result`具有如下属性：

        |名称|类型|含义|
        |-|-|-|
        |`retrievalResult`|`string`|知识检索结果，可用作其他操作的输入。|

- **`chat`**

    与大语言模型交互，利用大语言模型提炼关键信息。

    `POST /chatocr-vision`

    - 请求体的属性如下：

        |名称|类型|含义|是否必填|
        |-|-|-|-|
        |`keys`|`array`|关键词列表。|是|
        |`visionInfo`|`object`|图像中的关键信息。由`analyzeImage`操作提供。|是|
        |`taskDescription`|`string`|提示词任务。|否|
        |`rules`|`string`|提示词规则。用于自定义信息抽取规则，例如规范输出格式。|否|
        |`fewShot`|`string`|提示词示例。|否|
        |`useVectorStore`|`boolean`|是否启用向量数据库。默认启用。|否|
        |`vectorStore`|`object`|向量数据库序列化结果。由`buildVectorStore`操作提供。|否|
        |`retrievalResult`|`string`|知识检索结果。由`retrieveKnowledge`操作提供。|否|
        |`returnPrompts`|`boolean`|是否返回使用的提示词。默认启用。|否|
        |`llmName`|`string`|大语言模型名称。|否|
        |`llmParams`|`object`|大语言模型API参数。|否|

        当前，`llmParams`可以采用如下两种形式之一：
        
        ```json
        {
          "apiType": "qianfan",
          "apiKey": "{千帆平台API key}",
          "secretKey": "{千帆平台secret key}"
        }
        ```

        ```json
        {
          "apiType": "{aistudio}",
          "accessToken": "{AI Studio访问令牌}"
        }
        ```

    - 请求处理成功时，响应体的`result`具有如下属性：

        |名称|类型|含义|
        |-|-|-|
        |`chatResult`|`string`|关键信息抽取结果。|
        |`prompts`|`object`|使用的提示词。|

        `prompts`的属性如下：

        |名称|类型|含义|
        |-|-|-|
        |`ocr`|`string`|OCR提示词。|
        |`table`|`string`|表格提示词。|
        |`html`|`string`|HTML提示词。|

</details>

<details>
<summary>多语言调用服务示例</summary>  

<details>  
<summary>Python</summary>  
  
```python
import base64
import pprint
import sys

import requests


API_BASE_URL = "http://0.0.0.0:8080"
API_KEY = "{千帆平台API key}"
SECRET_KEY = "{千帆平台secret key}"
LLM_NAME = "ernie-3.5"
LLM_PARAMS = {
    "apiType": "qianfan", 
    "apiKey": API_KEY, 
    "secretKey": SECRET_KEY,
}


if __name__ == "__main__":
    file_path = "./demo.jpg"
    keys = ["电话"]

    payload = {
        "file": file_url,
        "useOricls": True,
        "useCurve": True,
        "useUvdoc": True,
    }
    resp_vision = requests.post(url=f"{API_BASE_URL}/chatocr-vision", json=payload)
    if resp_vision.status_code != 200:
        print(
            f"Request to chatocr-vision failed with status code {resp_vision.status_code}."
        )
        pprint.pp(resp_vision.json())
        sys.exit(1)
    result_vision = resp_vision.json()["result"]

    for i, res in enumerate(result_vision["visionResults"]):
        print("Texts:")
        pprint.pp(res["texts"])
        print("Tables:")
        pprint.pp(res["tables"])
        ocr_img_path = f"ocr_{i}.jpg"
        with open(ocr_img_path, "wb") as f:
            f.write(base64.b64decode(res["ocrImage"]))
        layout_img_path = f"layout_{i}.jpg"
        with open(layout_img_path, "wb") as f:
            f.write(base64.b64decode(res["layoutImage"]))
        print(f"Output images saved at {ocr_img_path} and {layout_img_path}")
        print("")
    print("="*50 + "\n\n")

    payload = {
        "visionInfo": result_vision["visionInfo"],
        "minChars": 200,
        "llmRequestInterval": 1000,
        "llmName": LLM_NAME,
        "llmParams": LLM_PARAMS,
    }
    resp_vector = requests.post(url=f"{API_BASE_URL}/chatocr-vector", json=payload)
    if resp_vector.status_code != 200:
        print(
            f"Request to chatocr-vector failed with status code {resp_vector.status_code}."
        )
        pprint.pp(resp_vector.json())
        sys.exit(1)
    result_vector = resp_vector.json()["result"]
    print("="*50 + "\n\n")

    payload = {
        "keys": keys,
        "vectorStore": result_vector["vectorStore"],
        "visionInfo": result_vision["visionInfo"],
        "llmName": LLM_NAME,
        "llmParams": LLM_PARAMS,
    }
    resp_retrieval = requests.post(url=f"{API_BASE_URL}/chatocr-retrieval", json=payload)
    if resp_retrieval.status_code != 200:
        print(
            f"Request to chatocr-retrieval failed with status code {resp_retrieval.status_code}."
        )
        pprint.pp(resp_retrieval.json())
        sys.exit(1)
    result_retrieval = resp_retrieval.json()["result"]
    print("Knowledge retrieval result:")
    print(result_retrieval["retrievalResult"])
    print("="*50 + "\n\n")

    payload = {
        "keys": keys,
        "visionInfo": result_vision["visionInfo"],
        "taskDescription": "",
        "rules": "",
        "fewShot": "",
        "useVectorStore": True,
        "vectorStore": result_vector["vectorStore"],
        "retrievalResult": result_retrieval["retrievalResult"],
        "returnPrompts": True,
        "llmName": LLM_NAME,
        "llmParams": LLM_PARAMS,
    }
    resp_chat = requests.post(url=f"{API_BASE_URL}/chatocr-chat", json=payload)
    if resp_chat.status_code != 200:
        print(
            f"Request to chatocr-chat failed with status code {resp_chat.status_code}."
        )
        pprint.pp(resp_chat.json())
        sys.exit(1)
    result_chat = resp_chat.json()["result"]
    print("Prompts:")
    pprint.pp(result_chat["prompts"])
    print("Final result:")
    print(len(result_chat["chatResult"]))
```
  
</details>
</details>
