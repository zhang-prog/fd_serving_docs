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

- **`infer`**

    获取图像OCR结果。

    `POST /ocr`

    - 请求体的属性如下：

        |名称|类型|含义|是否必填|
        |-|-|-|-|
        |`image`|`string`|服务可访问的图像文件的URL或图像文件内容的Base64编码结果。|是|
        |`inferenceParams`|`object`|推理参数。|否|

        `inferenceParams`的属性如下：

        |名称|类型|含义|是否必填|
        |-|-|-|-|
        |`maxLongSide`|`integer`|推理时，若文本检测模型的输入图像较长边的长度大于`maxLongSide`，则将对图像进行缩放，使其较长边的长度等于`maxLongSide`。|否|

    - 请求处理成功时，响应体的`result`具有如下属性：

        |名称|类型|含义|
        |-|-|-|
        |`texts`|`array`|文本位置、内容和得分。|
        |`image`|`string`|OCR结果图，其中标注检测到的文本位置。图像为JPEG格式，使用Base64编码。|

        `texts`中的每个元素为一个`object`，具有如下属性：

        |名称|类型|含义|
        |-|-|-|
        |`poly`|`array`|文本位置。数组中元素依次为包围文本的多边形的顶点坐标。|
        |`text`|`string`|文本内容。|
        |`score`|`number`|文本识别得分。|

        `result`示例如下：

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
              "text": "北京南站",
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
              "text": "天津站",
              "score": 0.5
            }
          ],
          "image": "xxxxxx"
        }
        ```

</details>

<details>
<summary>多语言调用服务示例</summary>  

<details>  
<summary>Python</summary>  
  
```python
import base64
import requests

API_URL = "http://localhost:8080/ocr" # 服务URL
image_path = "./demo.jpg"
output_image_path = "./out.jpg"

# 对本地图像进行Base64编码
with open(image_path, "rb") as file:
    image_bytes = file.read()
    image_data = base64.b64encode(image_bytes).decode("ascii")

payload = {"image": image_data}  # Base64编码的文件内容或者图像URL

# 调用API
response = requests.post(API_URL, json=payload)

# 处理接口返回数据
assert response.status_code == 200
result = response.json()["result"]
with open(output_image_path, "wb") as file:
    file.write(base64.b64decode(result["image"]))
print(f"Output image saved at {output_image_path}")
print("\nTexts:")
print(result["texts"])
```
  
</details>

<details>  
<summary>C++</summary>  
  
```cpp
#include <iostream>
#include "cpp-httplib/httplib.h" // https://github.com/Huiyicc/cpp-httplib
#include "nlohmann/json.hpp" // https://github.com/nlohmann/json
#include "base64.hpp" // https://github.com/tobiaslocker/base64

int main() {
    httplib::Client client("localhost:8080");
    const std::string imagePath = "./demo.jpg";
    const std::string outputImagePath = "./out.jpg";

    httplib::Headers headers = {
        {"Content-Type", "application/json"}
    };

    // 对本地图像进行Base64编码
    std::ifstream file(imagePath, std::ios::binary | std::ios::ate);
    std::streamsize size = file.tellg();
    file.seekg(0, std::ios::beg);

    std::vector<char> buffer(size);
    if (!file.read(buffer.data(), size)) {
        std::cerr << "Error reading file." << std::endl;
        return 1;
    }
    std::string bufferStr(reinterpret_cast<const char*>(buffer.data()), buffer.size());
    std::string encodedImage = base64::to_base64(bufferStr);

    nlohmann::json jsonObj;
    jsonObj["image"] = encodedImage;
    std::string body = jsonObj.dump();

    // 调用API
    auto response = client.Post("/ocr", headers, body, "application/json");
    // 处理接口返回数据
    if (response && response->status == 200) {
        nlohmann::json jsonResponse = nlohmann::json::parse(response->body);
        auto result = jsonResponse["result"];

        encodedImage = result["image"];
        std::string decodedString = base64::from_base64(encodedImage);
        std::vector<unsigned char> decodedImage(decodedString.begin(), decodedString.end());
        std::ofstream outputImage(outPutImagePath, std::ios::binary | std::ios::out);
        if (outputImage.is_open()) {
            outputImage.write(reinterpret_cast<char*>(decodedImage.data()), decodedImage.size());
            outputImage.close();
            std::cout << "Output image saved at " << outPutImagePath << std::endl;
        } else {
            std::cerr << "Unable to open file for writing: " << outPutImagePath << std::endl;
        }

        auto texts = result["texts"];
        std::cout << "\nTexts:" << std::endl;
        for (const auto& category : texts) {
            std::cout << category << std::endl;
        }
    } else {
        std::cout << "Failed to send HTTP request." << std::endl;
        return 1;
    }

    return 0;
}
```
  
</details>

<details>  
<summary>Java</summary>  
  
```java
import okhttp3.*;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.node.ObjectNode;

import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.Base64;

public class Main {
    public static void main(String[] args) throws IOException {
        String API_URL = "https://localhost:8080/ocr"; // 服务URL
        String imagePath = "./demo.jpg"; // 本地图像
        String outputImagePath = "./out.jpg"; // 输出图像

        // 对本地图像进行Base64编码
        File file = new File(imagePath);
        byte[] fileContent = java.nio.file.Files.readAllBytes(file.toPath());
        String imageData = Base64.getEncoder().encodeToString(fileContent);

        ObjectMapper objectMapper = new ObjectMapper();
        ObjectNode params = objectMapper.createObjectNode();
        params.put("image", imageData); // Base64编码的文件内容或者图像URL

        // 创建 OkHttpClient 实例
        OkHttpClient client = new OkHttpClient();
        MediaType JSON = MediaType.Companion.get("application/json; charset=utf-8");
        RequestBody body = RequestBody.Companion.create(params.toString(), JSON);
        Request request = new Request.Builder()
                .url(API_URL)
                .post(body)
                .build();

        // 调用API并处理接口返回数据
        try (Response response = client.newCall(request).execute()) {
            if (response.isSuccessful()) {
                String responseBody = response.body().string();
                JsonNode resultNode = objectMapper.readTree(responseBody);
                JsonNode result = resultNode.get("result");
                String base64Image = result.get("image").asText();
                JsonNode texts = result.get("texts");

                byte[] imageBytes = Base64.getDecoder().decode(base64Image);
                try (FileOutputStream fos = new FileOutputStream(outputImagePath)) {
                    fos.write(imageBytes);
                }
                System.out.println("Output image saved at " + outputImagePath);
                System.out.println("\nTexts: " + texts.toString());
            } else {
                System.err.println("Request failed with code: " + response.code());
            }
        }
    }
}
```
  
</details>

<details>  
<summary>Go</summary>  
  
```go
package main

import (
	"bytes"
	"encoding/base64"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"
)

func main() {
	API_URL := "https://localhost:8080/ocr"
	imagePath := "./demo.jpg"
	outputImagePath := "./out.jpg"

	// 对本地图像进行Base64编码
	imageBytes, err := ioutil.ReadFile(imagePath)
	if err != nil {
		fmt.Println("Error reading image file:", err)
		return
	}
	imageData := base64.StdEncoding.EncodeToString(imageBytes)

	payload := map[string]string{"image": imageData} // Base64编码的文件内容或者图像URL
	payloadBytes, err := json.Marshal(payload)
	if err != nil {
		fmt.Println("Error marshaling payload:", err)
		return
	}

	// 调用API
	client := &http.Client{}
	req, err := http.NewRequest("POST", API_URL, bytes.NewBuffer(payloadBytes))
	if err != nil {
		fmt.Println("Error creating request:", err)
		return
	}

	res, err := client.Do(req)
	if err != nil {
		fmt.Println("Error sending request:", err)
		return
	}
	defer res.Body.Close()

    // 处理接口返回数据
	body, err := ioutil.ReadAll(res.Body)
	if err != nil {
		fmt.Println("Error reading response body:", err)
		return
	}
	type Response struct {
		Result struct {
			Image      string   `json:"image"`
			Texts []map[string]interface{} `json:"texts"`
		} `json:"result"`
	}
	var respData Response
	err = json.Unmarshal([]byte(string(body)), &respData)
	if err != nil {
		fmt.Println("Error unmarshaling response body:", err)
		return
	}

	outputImageData, err := base64.StdEncoding.DecodeString(respData.Result.Image)
	if err != nil {
		fmt.Println("Error decoding base64 image data:", err)
		return
	}
	err = ioutil.WriteFile(outputImagePath, outputImageData, 0644)
	if err != nil {
		fmt.Println("Error writing image to file:", err)
		return
	}
	fmt.Printf("Image saved at %s.jpg\n", outputImagePath)
	fmt.Println("\nTexts:")
	for _, category := range respData.Result.Texts {
		fmt.Println(category)
	}
}
```
  
</details>

<details>  
<summary>C#</summary>  
  
```csharp
using System;
using System.IO;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Text;
using System.Threading.Tasks;
using Newtonsoft.Json.Linq;

class Program
{
    static readonly string API_URL = "https://localhost:8080/ocr";
    static readonly string imagePath = "./demo.jpg";
    static readonly string outputImagePath = "./out.jpg";

    static async Task Main(string[] args)
    {
        var httpClient = new HttpClient();

        // 对本地图像进行Base64编码
        byte[] imageBytes = File.ReadAllBytes(imagePath);
        string image_data = Convert.ToBase64String(imageBytes);

        var payload = new JObject{ { "image", image_data } }; // Base64编码的文件内容或者图像URL
        var content = new StringContent(payload.ToString(), Encoding.UTF8, "application/json");

        // 调用API
        HttpResponseMessage response = await httpClient.PostAsync(API_URL, content);
        response.EnsureSuccessStatusCode();

        // 处理接口返回数据
        string responseBody = await response.Content.ReadAsStringAsync();
        JObject jsonResponse = JObject.Parse(responseBody);

        string base64Image = jsonResponse["result"]["image"].ToString();
        byte[] outputImageBytes = Convert.FromBase64String(base64Image);

        File.WriteAllBytes(outputImagePath, outputImageBytes);
        Console.WriteLine($"Output image saved at {outputImagePath}");
        Console.WriteLine("\nTexts:");
        Console.WriteLine(jsonResponse["result"]["texts"].ToString());
    }
}
```
  
</details>

<details>  
<summary>Node.js</summary>  
  
```js
const axios = require('axios');
const fs = require('fs');

const API_URL = 'https://localhost:8080/ocr'
const imagePath = './demo.jpg'
const outputImagePath = "./out.jpg";

let config = {
   method: 'POST',
   maxBodyLength: Infinity,
   url: API_URL,
   data: JSON.stringify({
    'image': encodeImageToBase64(imagePath)  // Base64编码的文件内容或者图像URL
  })
};

// 对本地图像进行Base64编码
function encodeImageToBase64(filePath) {
  const bitmap = fs.readFileSync(filePath);
  return Buffer.from(bitmap).toString('base64');
}

// 调用API
axios.request(config)
.then((response) => {
    // 处理接口返回数据
    const result = response.data["result"];
    const imageBuffer = Buffer.from(result["image"], 'base64');
    fs.writeFile(outputImagePath, imageBuffer, (err) => {
      if (err) throw err;
      console.log(`Output image saved at ${outputImagePath}`);
    });
    console.log("\nTexts:");
    console.log(result["texts"]);
})
.catch((error) => {
  console.log(error);
});
```
  
</details>

<details>  
<summary>PHP</summary>  
  
```php
<?php

$API_URL = "http://localhost:8080/ocr"; // 服务URL
$image_path = "./demo.jpg";
$output_image_path = "./out.jpg";

// 对本地图像进行Base64编码
$image_data = base64_encode(file_get_contents($image_path));
$payload = array("image" => $image_data); // Base64编码的文件内容或者图像URL

// 调用API
$ch = curl_init($API_URL);
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($payload));
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$response = curl_exec($ch);
curl_close($ch);

// 处理接口返回数据
$result = json_decode($response, true)["result"];
file_put_contents($output_image_path, base64_decode($result["image"]));
echo "Output image saved at " . $output_image_path . "\n";
echo "\nTexts:\n";
print_r($result["texts"]);

?>
```
  
</details>

</details>
