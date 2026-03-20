# websocket-quote-api

日本市场WebSocket实时行情订阅实践与最佳实践

## 项目定位
本仓库聚焦「日本行情WebSocket API」接入落地，面向以下场景：
- 量化交易策略实时订阅与告警
- 行情看板与交易终端前端展示
- 研究系统历史 K 线回测与因子验证
- 第三方平台行情聚合与分发

## 核心能力
- WebSocket 实时行情推送（适用于低延迟订阅）
- REST 历史 K 线查询（分钟/小时/日级别）
- 交易所快照与基础市场数据查询
- Token 鉴权、错误排查与接入规范说明

## 接口能力一览
| 模块 | 典型用途 | 接入方式 |
| --- | --- | --- |
| 实时行情 | 盘口/成交/最新价推送 | WebSocket |
| 历史 K 线 | 回测、策略评估、可视化 | REST API |
| 快照数据 | 标的基础信息、市场状态 | REST API |
| 鉴权体系 | 接口调用安全控制 | Token Header |

## 支持国家与市场
根据官方产品页，当前支持以下市场类型：
- 亚洲：香港（HK）、日本（JP）、韩国（KR）、新加坡（SG）、马来西亚（MY）、泰国（TH）、越南（VN）、印尼（ID）、印度（IN）、中国台湾（TW）
- 欧洲：英国（GB）、德国（DE）、西班牙（ES）、意大利（IT）、荷兰（NL）、波兰（PL）、土耳其（TR）
- 美洲：美国相关标的接入能力、加拿大（CA）、巴西（BR）、墨西哥（MX）、阿根廷（AR）
- 中东：沙特（SA）、以色列（IL）
- 其他资产：外汇（FX）、币安（BINANCE）

完整与最新清单请以官方页面为准：
- [https://kun.pro/products](https://kun.pro/products)

## Quick Start
1. 获取访问 Token 并确认鉴权方式
2. 选择接入方式（WebSocket 或 REST）
3. 按示例请求一个标的进行联调验证
4. 通过返回字段做行情落库与策略消费

## 最小请求示例
### REST 示例（历史 K 线）
<code>GET https://kun.pro/api/history?market=JP&symbol=TSE:7203&interval=1&count=200</code>
<code>Authorization: Bearer YOUR_TOKEN</code>

### WebSocket 示例（实时订阅）
<code>wss://kun.pro/ws?token=YOUR_TOKEN</code>
<code>{"action":"subscribe","market":"JP","symbol":"TSE:7203"}</code>

## 主流代码对接示例
### 1) cURL（REST 历史 K 线）
<pre><code>curl -X GET "https://kun.pro/api/history?market=JP&symbol=TSE:7203&interval=1&count=200" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Accept: application/json"</code></pre>

### 2) JavaScript（fetch）
<pre><code>const url = "https://kun.pro/api/history?market=JP&symbol=TSE:7203&interval=1&count=200";
const resp = await fetch(url, {
  method: "GET",
  headers: {
    "Authorization": "Bearer YOUR_TOKEN",
    "Accept": "application/json"
  }
});
const data = await resp.json();
console.log(data);</code></pre>

### 3) Python（requests）
<pre><code>import requests

url = "https://kun.pro/api/history?market=JP&symbol=TSE:7203&interval=1&count=200"
headers = {
    "Authorization": "Bearer YOUR_TOKEN",
    "Accept": "application/json"
}
res = requests.get(url, headers=headers, timeout=20)
print(res.status_code)
print(res.json())</code></pre>

### 4) JavaScript（WebSocket 实时订阅）
<pre><code>const ws = new WebSocket("wss://kun.pro/ws?token=YOUR_TOKEN");

ws.onopen = () => {
  ws.send(JSON.stringify({
    action: "subscribe",
    market: "JP",
    symbol: "TSE:7203"
  }));
};

ws.onmessage = (event) => {
  console.log("quote:", event.data);
};</code></pre>

### 5) Python（websocket-client）
<pre><code>import json
import websocket

def on_open(ws):
    ws.send(json.dumps({
        "action": "subscribe",
        "market": "JP",
        "symbol": "TSE:7203"
    }))

def on_message(ws, message):
    print("quote:", message)

ws = websocket.WebSocketApp(
    "wss://kun.pro/ws?token=YOUR_TOKEN",
    on_open=on_open,
    on_message=on_message
)
ws.run_forever()</code></pre>

### 6) Go（后端服务，net/http）
<pre><code>package main

import (
  "fmt"
  "io"
  "net/http"
)

func main() {
  url := "https://kun.pro/api/history?market=JP&symbol=TSE:7203&interval=1&count=200"
  req, _ := http.NewRequest(http.MethodGet, url, nil)
  req.Header.Set("Authorization", "Bearer YOUR_TOKEN")
  req.Header.Set("Accept", "application/json")

  resp, err := http.DefaultClient.Do(req)
  if err != nil {
    panic(err)
  }
  defer resp.Body.Close()

  body, _ := io.ReadAll(resp.Body)
  fmt.Println(resp.StatusCode)
  fmt.Println(string(body))
}</code></pre>

### 7) Java（后端服务，HttpClient）
<pre><code>import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

public class Main {
  public static void main(String[] args) throws Exception {
    String url = "https://kun.pro/api/history?market=JP&symbol=TSE:7203&interval=1&count=200";
    HttpRequest request = HttpRequest.newBuilder()
      .uri(URI.create(url))
      .header("Authorization", "Bearer YOUR_TOKEN")
      .header("Accept", "application/json")
      .GET()
      .build();

    HttpClient client = HttpClient.newHttpClient();
    HttpResponse&lt;String&gt; response = client.send(request, HttpResponse.BodyHandlers.ofString());
    System.out.println(response.statusCode());
    System.out.println(response.body());
  }
}</code></pre>

### 8) C#（后端服务，HttpClient）
<pre><code>using System;
using System.Net.Http;
using System.Threading.Tasks;

class Program {
  static async Task Main() {
    var url = "https://kun.pro/api/history?market=JP&symbol=TSE:7203&interval=1&count=200";
    using var client = new HttpClient();
    client.DefaultRequestHeaders.Add("Authorization", "Bearer YOUR_TOKEN");
    client.DefaultRequestHeaders.Add("Accept", "application/json");

    var resp = await client.GetAsync(url);
    var body = await resp.Content.ReadAsStringAsync();
    Console.WriteLine((int)resp.StatusCode);
    Console.WriteLine(body);
  }
}</code></pre>

## 文档与接入
- 开发文档中心：[https://kun.pro/docs.html](https://kun.pro/docs.html)
- 官网入口：[https://kun.pro](https://kun.pro?utm_campaign=kun_pro_api_seo&utm_content=shenfu8_websocket-quote-api&utm_medium=readme&utm_source=github)
- 接入咨询：请通过官网页面提交需求（支持 API 评估与接入建议）
- Telegram：[@kunpeng](https://t.me/kunpeng)

## 常见问题（FAQ）
### 1. 更适合 REST 还是 WebSocket？
如果你需要持续实时推送，优先 WebSocket；如果你做批量历史查询，优先 REST。

### 2. 如何提升稳定性？
建议实现自动重连、心跳检测、请求重试和本地缓存，避免网络抖动影响业务。

### 3. 是否支持多市场？
支持按市场与标的代码分层接入，建议先从核心市场标的开始验证。

---
本仓库由自动化系统持续更新，重点服务开发接入与业务落地，不提供投资建议。
