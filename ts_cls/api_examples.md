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

    对时序数据进行分类。

    `POST /time-series-classification`

    - 请求体的属性如下：

        |名称|类型|含义|是否必填|
        |-|-|-|-|
        |`csv`|`string`|服务可访问的CSV文件的URL或CSV文件内容的Base64编码结果。CSV文件需要使用UTF-8编码。|是|

    - 请求处理成功时，响应体的`result`具有如下属性：

        |名称|类型|含义|
        |-|-|-|
        |`label`|`string`|类别标签。|
        |`score`|`number`|类别得分。|
        |`image`|`string`|时序分类结果图。图像为JPEG格式，使用Base64编码。|

        `result`示例如下：

        ```json
        {
          "label": "running",
          "score": 0.97,
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

API_URL = "http://localhost:8080/time-series-classification" # 服务URL
csv_path = "./test.csv"
output_image_path = "./out.jpg"

# 对本地图像进行Base64编码
with open(csv_path, "rb") as file:
    csv_bytes = file.read()
    csv_data = base64.b64encode(csv_bytes).decode("ascii")

payload = {"csv": csv_data}

# 调用API
response = requests.post(API_URL, json=payload)

# 处理接口返回数据
assert response.status_code == 200
result = response.json()["result"]
with open(output_image_path, "wb") as f:
    f.write(base64.b64decode(result["image"]))
print(f"Output image saved at  {output_image_path}")
print(f"label: {result['label']}, score: {result['score']}")
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
    const std::string csvPath = "./test.csv";
    const std::string outputImagePath = "./out.jpg";

    httplib::Headers headers = {
        {"Content-Type", "application/json"}
    };

    // 进行Base64编码
    std::ifstream file(csvPath, std::ios::binary | std::ios::ate);
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
    jsonObj["csv"] = encodedImage;
    std::string body = jsonObj.dump();

    // 调用API
    auto response = client.Post("/time-series-classification", headers, body, "application/json");
    // 处理接口返回数据
    if (response && response->status == 200) {
        nlohmann::json jsonResponse = nlohmann::json::parse(response->body);
        auto result = jsonResponse["result"];

        // 保存图像
        encodedImage = result["image"];
        std::string decodedString = base64::from_base64(encodedImage);
        std::vector<unsigned char> decodedImage(decodedString.begin(), decodedString.end());
        std::ofstream outputImage(outputImagePath, std::ios::binary | std::ios::out);
        if (outputImage.is_open()) {
            outputImage.write(reinterpret_cast<char*>(decodedImage.data()), decodedImage.size());
            outputImage.close();
            std::cout << "Output image saved at " << outputImagePath << std::endl;
        } else {
            std::cerr << "Unable to open file for writing: " << outputImagePath << std::endl;
        }
        std::cout << "label: " << result["label"] << ", score: " << result["score"] << std::endl;
    } else {
        std::cout << "Failed to send HTTP request." << std::endl;
        std::cout << response->body << std::endl;
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
        String API_URL = "http://localhost:8080/time-series-classification";
        String csvPath = "./test.csv";
        String outputImagePath = "./out.jpg";

        // 对本地csv进行Base64编码
        File file = new File(csvPath);
        byte[] fileContent = java.nio.file.Files.readAllBytes(file.toPath());
        String csvData = Base64.getEncoder().encodeToString(fileContent);

        ObjectMapper objectMapper = new ObjectMapper();
        ObjectNode params = objectMapper.createObjectNode();
        params.put("csv", csvData);

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

                // 保存返回的图像
                String base64Image = result.get("image").asText();
                byte[] imageBytes = Base64.getDecoder().decode(base64Image);
                try (FileOutputStream fos = new FileOutputStream(outputImagePath)) {
                    fos.write(imageBytes);
                }
                System.out.println("Output image saved at " + outputImagePath);
                System.out.println("label: " + result.get("label").asText() + ", score: " + result.get("score").asText());
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
	API_URL := "http://localhost:8080/time-series-classification"
	csvPath := "./test.csv";
	outputImagePath := "./out.jpg"

	// 读取csv文件并进行Base64编码
	csvBytes, err := ioutil.ReadFile(csvPath)
	if err != nil {
		fmt.Println("Error reading image file:", err)
		return
	}
	csvData := base64.StdEncoding.EncodeToString(csvBytes)

	payload := map[string]string{"csv": csvData} // Base64编码的文件内容
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

	// 处理返回数据
	body, err := ioutil.ReadAll(res.Body)
	if err != nil {
		fmt.Println("Error reading response body:", err)
		return
	}
	type Response struct {
		Result struct {
			Image string `json:"image"`
			Label string `json:"label"`
            Score string `json:"score"`
		} `json:"result"`
	}
	var respData Response
	err = json.Unmarshal([]byte(string(body)), &respData)
	if err != nil {
		fmt.Println("Error unmarshaling response body:", err)
		return
	}

	// 将Base64编码的图像数据解码并保存为文件
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
	fmt.Printf("label: %s, score: %s\n", respData.Result.Label, respData.Result.Score)
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
    static readonly string API_URL = "http://localhost:8080/time-series-classification";
    static readonly string csvPath = "./test.csv";
    static readonly string outputImagePath = "./out.jpg";

    static async Task Main(string[] args)
    {
        var httpClient = new HttpClient();

        // 对本地csv文件进行Base64编码
        byte[] csveBytes = File.ReadAllBytes(csvPath);
        string csvData = Convert.ToBase64String(csveBytes);

        var payload = new JObject{ { "csv", csvData } }; // Base64编码的文件内容
        var content = new StringContent(payload.ToString(), Encoding.UTF8, "application/json");

        // 调用API
        HttpResponseMessage response = await httpClient.PostAsync(API_URL, content);
        response.EnsureSuccessStatusCode();

        // 处理接口返回数据
        string responseBody = await response.Content.ReadAsStringAsync();
        JObject jsonResponse = JObject.Parse(responseBody);

        // 保存图像
        string base64Image = jsonResponse["result"]["image"].ToString();
        byte[] outputImageBytes = Convert.FromBase64String(base64Image);
        File.WriteAllBytes(outputImagePath, outputImageBytes);
        Console.WriteLine($"Output image saved at {outputImagePath}");

        string label = jsonResponse["result"]["label"].ToString();
        string score = jsonResponse["result"]["score"].ToString();
        Console.WriteLine($"label: {label}, score: {score}");
    }
}
```
  
</details>

<details>  
<summary>Node.js</summary>  
  
```js
const axios = require('axios');
const fs = require('fs');

const API_URL = 'http://localhost:8080/time-series-classification'
const csvPath = "./test.csv";
const outputImagePath = './out.jpg'

let config = {
   method: 'POST',
   maxBodyLength: Infinity,
   url: API_URL,
   data: JSON.stringify({
    'csv': encodeImageToBase64(csvPath)  // Base64编码的文件内容
  })
};

// 读取csv文件并转换为Base64
function encodeImageToBase64(filePath) {
  const bitmap = fs.readFileSync(filePath);
  return Buffer.from(bitmap).toString('base64');
}

axios.request(config)
.then((response) => {
    const result = response.data["result"];

    // 保存图像
    const imageBuffer = Buffer.from(result["image"], 'base64');
    fs.writeFile(outputImagePath, imageBuffer, (err) => {
      if (err) throw err;
      console.log(`Output image saved at ${outputImagePath}`);
    });

    console.log(`label: ${result["image"]}, score: ${result["score"]}`);

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

$API_URL = "http://localhost:8080/time-series-classification"; // 服务URL
$csv_path = "./test.csv";
$output_image_path = "./out.jpg";

// 对本地csv文件进行Base64编码
$csv_data = base64_encode(file_get_contents($csv_path));
$payload = array("csv" => $csv_data); // Base64编码的文件内容

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
echo "label: " . $result["label"] . ", score: " . $result["score"];

?>
```
  
</details>

</details>
