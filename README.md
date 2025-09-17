# Deno版


## 🚀 快速开始


### Deno Deploy部署

Deno Deploy是一个全球分布式的边缘计算平台，非常适合部署Deno应用。

#### 步骤：

1. **准备代码**
   - Fork本仓库，顺便点个小心心

2. **登录Deno Deploy**
   - 访问 [https://dash.deno.com/](https://dash.deno.com/)
   - 使用GitHub账号登录

3. **创建新项目**
   - 点击"New Project"按钮
   - 选择你的GitHub仓库
   - 选择包含`main.ts`文件

4. **配置环境变量**
   - 在项目设置中，添加环境变量
     - 具体配置见env.example

5. **部署**
   - 点击"Deploy"按钮
   - 等待部署完成

### 本地部署

**环境要求**
- Deno 1.40+
- 现代浏览器或 Node.js 环境

### 安装运行

```bash
# 克隆项目
git clone https://github.com/Luotianyi-0712/z.ai2api_deno.git
cd z.ai2api_deno

# 使用 Deno 运行
deno task start

# 或开发模式（自动重载）
deno task dev
```

服务启动后访问：http://localhost:8080/v1/models

### 基础使用

#### OpenAI API 客户端

```typescript
import OpenAI from 'openai';

// 初始化客户端
const client = new OpenAI({
  baseURL: "http://localhost:8080/v1",
  apiKey: "your-auth-token"  // 替换为你的 AUTH_TOKEN
});

// 普通对话
const response = await client.chat.completions.create({
  model: "GLM-4.5",
  messages: [{ role: "user", content: "你好，介绍一下 TypeScript" }],
  stream: false
});

console.log(response.choices[0].message.content);
```

### Docker 部署

```bash
cd deploy
docker-compose up -d
```

## 📖 详细指南

### 支持的模型

| 模型 | 上游ID | 描述 | 特性 |
|------|--------|------|------|
| `GLM-4.5` | 0727-360B-API | 标准模型 | 通用对话，平衡性能 |
| `GLM-4.5-Thinking` | 0727-360B-API | 思考模型 | 显示推理过程，透明度高 |
| `GLM-4.5-Search` | 0727-360B-API | 搜索模型 | 实时网络搜索，信息更新 |
| `GLM-4.5-Air` | 0727-106B-API | 轻量模型 | 快速响应，高效推理 |
| `GLM-4.5V` | glm-4.5v | ❌ 暂不支持 |   |

### Function Call 功能

```typescript
// 定义工具
const tools = [{
  type: "function",
  function: {
    name: "get_weather",
    description: "获取天气信息",
    parameters: {
      type: "object",
      properties: {
        city: { type: "string", description: "城市名称" }
      },
      required: ["city"]
    }
  }
}];

// 使用工具
const response = await client.chat.completions.create({
  model: "GLM-4.5",
  messages: [{ role: "user", content: "北京天气怎么样？" }],
  tools: tools,
  tool_choice: "auto"
});
```

### 流式响应

```typescript
const response = await client.chat.completions.create({
  model: "GLM-4.5-Thinking",
  messages: [{ role: "user", content: "解释量子计算" }],
  stream: true
});

for await (const chunk of response) {
  const content = chunk.choices[0].delta.content;
  const reasoning = chunk.choices[0].delta.reasoning_content;
  
  if (content) {
    process.stdout.write(content);
  }
  if (reasoning) {
    console.log(`\n🤔 思考: ${reasoning}\n`);
  }
}
```

## ⚙️ 配置说明

### 环境变量配置

| 变量名 | 默认值 | 说明 |
|--------|--------|------|
| `AUTH_TOKEN` | `sk-your-api-key` | 客户端认证密钥 |
| `API_ENDPOINT` | `https://chat.z.ai/api/chat/completions` | 上游 API 地址 |
| `LISTEN_PORT` | `8080` | 服务监听端口 |
| `PRIMARY_MODEL` | `GLM-4.5` | 主要模型名称 |
| `THINKING_MODEL` | `GLM-4.5-Thinking` | 思考模型名称 |
| `SEARCH_MODEL` | `GLM-4.5-Search` | 搜索模型名称 |
| `AIR_MODEL` | `GLM-4.5-Air` | Air 模型名称 |
| `DEBUG_LOGGING` | `true` | 调试日志开关 |
| `THINKING_PROCESSING` | `think` | 思考内容处理策略 |
| `ANONYMOUS_MODE` | `true` | 匿名模式开关 |
| `TOOL_SUPPORT` | `true` | Function Call 功能开关 |
| `SKIP_AUTH_TOKEN` | `false` | 跳过认证令牌验证 |
| `SCAN_LIMIT` | `200000` | 扫描限制 |
| `BACKUP_TOKEN` | `eyJhbGciOiJFUzI1NiIsInR5cCI6IkpXVCJ9...` | Z.ai 固定访问令牌 |

### 思考内容处理策略

- `think` - 转换为 `<thinking>` 标签（OpenAI 兼容）
- `strip` - 移除思考内容
- `raw` - 保留原始格式

## 🎯 使用场景

### 1. AI 应用开发

```typescript
// 集成到现有应用
import OpenAI from 'openai';

const client = new OpenAI({
  baseURL: "http://localhost:8080/v1",
  apiKey: "your-token"
});

// 智能客服
async function chatWithAI(message: string): Promise<string> {
  const response = await client.chat.completions.create({
    model: "GLM-4.5",
    messages: [{ role: "user", content: message }]
  });
  return response.choices[0].message.content || "";
}
```

### 2. 多模型对比测试

```typescript
const models = ["GLM-4.5", "GLM-4.5-Thinking", "GLM-4.5-Search", "GLM-4.5-Air"];

for (const model of models) {
  const response = await client.chat.completions.create({
    model: model,
    messages: [{ role: "user", content: "什么是机器学习？" }]
  });
  console.log(`\n=== ${model} ===`);
  console.log(response.choices[0].message.content);
}
```

### 3. 工具调用集成

```typescript
// 结合外部 API
async function callExternalAPI(toolName: string, arguments: any): Promise<any> {
  // 执行实际工具调用
  return result;
}

// 处理工具调用
if (response.choices[0].message.tool_calls) {
  for (const toolCall of response.choices[0].message.tool_calls) {
    const result = await callExternalAPI(
      toolCall.function.name,
      JSON.parse(toolCall.function.arguments)
    );
    // 将结果返回给模型继续对话
  }
}
```

## ❓ 常见问题

**Q: 如何获取 AUTH_TOKEN？**
A: `AUTH_TOKEN` 为自己自定义的api key，在环境变量中配置，需要保证客户端与服务端一致。

**Q: 如何通过 Claude Code 使用本服务？**

A: 创建 [zai.js](https://gist.githubusercontent.com/musistudio/b35402d6f9c95c64269c7666b8405348/raw/f108d66fa050f308387938f149a2b14a295d29e9/gistfile1.txt) 这个ccr插件放在`./.claude-code-router/plugins`目录下，配置 `./.claude-code-router/config.json` 指向本服务地址，使用 `AUTH_TOKEN` 进行认证。

zai.js
``` javascript
const crypto = require("crypto");

function generateUUID() {
  const bytes = crypto.randomBytes(16);

  // 设置版本号 (4)
  bytes[6] = (bytes[6] & 0x0f) | 0x40;
  // 设置变体 (10)
  bytes[8] = (bytes[8] & 0x3f) | 0x80;

  // 转换为UUID格式: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
  const hex = bytes.toString("hex");
  return `${hex.slice(0, 8)}-${hex.slice(8, 12)}-${hex.slice(
    12,
    16
  )}-${hex.slice(16, 20)}-${hex.slice(20)}`;
}

class ZAITransformer {
  name = "zai";

  constructor(options) {
    this.options = options;
  }

  async getToken() {
    return fetch("https://chat.z.ai/api/v1/auths/", {
      headers: {
        "User-Agent":
          "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/139.0.0.0 Safari/537.36",
        Referer: "https://chat.z.ai/",
      },
    })
      .then((res) => res.json())
      .then((res) => res.token);
  }

  async transformRequestIn(request, provider) {
    const token = await this.getToken();
    const messages = [];
    for (const origMsg of request.messages || []) {
      const msg = { ...origMsg };
      if (msg.role === "system") {
        msg.role = "user";
        if (Array.isArray(msg.content)) {
          msg.content = [
            {
              type: "text",
              text: "This is a system command, you must enforce compliance.",
            },
            ...msg.content,
          ];
        } else if (typeof msg.content === "string") {
          msg.content = `This is a system command, you must enforce compliance.${msg.content}`;
        }
      } else if (msg.role === "user") {
        if (Array.isArray(msg.content)) {
          const newContent = [];
          for (const part of msg.content) {
            if (
              part?.type === "image_url" &&
              part?.image_url?.url &&
              typeof part.image_url.url === "string" &&
              !part.image_url.url.startsWith("http")
            ) {
              // 上传图片
              newContent.push(part);
            } else {
              newContent.push(part);
            }
          }
          msg.content = newContent;
        }
      }
      messages.push(msg);
    }
    return {
      body: {
        stream: true,
        model: request.model,
        messages: messages,
        params: {},
        features: {
          image_generation: false,
          web_search: false,
          auto_web_search: false,
          preview_mode: false,
          flags: [],
          features: [],
          enable_thinking: !!request.reasoning,
        },
        variables: {
          "{{USER_NAME}}": "Guest",
          "{{USER_LOCATION}}": "Unknown",
          "{{CURRENT_DATETIME}}": new Date()
            .toISOString()
            .slice(0, 19)
            .replace("T", " "),
          "{{CURRENT_DATE}}": new Date().toISOString().slice(0, 10),
          "{{CURRENT_TIME}}": new Date().toISOString().slice(11, 19),
          "{{CURRENT_WEEKDAY}}": new Date().toLocaleDateString("en-US", {
            weekday: "long",
          }),
          "{{CURRENT_TIMEZONE}":
            Intl.DateTimeFormat().resolvedOptions().timeZone,
          "{{USER_LANGUAGE}}": "zh-CN",
        },
        model_item: {},
        tools:
          !request.reasoning && request.tools?.length
            ? request.tools
            : undefined,
        chat_id: generateUUID(),
        id: generateUUID(),
      },
      config: {
        url: new URL("https://chat.z.ai/api/chat/completions"),
        headers: {
          Accept: "*/*",
          "Accept-Language": "zh-CN",
          Authorization: `Bearer ${token || ""}`,
          "Cache-Control": "no-cache",
          Connection: "keep-alive",
          "Content-Type": "application/json",
          Origin: "https://chat.z.ai",
          Pragma: "no-cache",
          Referer: "https://chat.z.ai/",
          "Sec-Fetch-Dest": "empty",
          "Sec-Fetch-Mode": "cors",
          "Sec-Fetch-Site": "same-origin",
          "User-Agent":
            "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/16.0 Safari/605.1.15",
          "X-FE-Version": "prod-fe-1.0.77",
        },
      },
    };
  }

  async transformResponseOut(response, context) {
    if (response.headers.get("Content-Type")?.includes("application/json")) {
      let jsonResponse = await response.json();
      const res = {
        id: jsonResponse.id,
        choices: [
          {
            finish_reason: jsonResponse.choices[0].finish_reason || null,
            index: 0,
            message: {
              content: jsonResponse.choices[0].message?.content || "",
              role: "assistant",
              tool_calls:
                jsonResponse.choices[0].message?.tool_calls || undefined,
            },
          },
        ],
        created: parseInt(new Date().getTime() / 1000 + "", 10),
        model: jsonResponse.model,
        object: "chat.completion",
        usage: jsonResponse.usage || {
          completion_tokens: 0,
          prompt_tokens: 0,
          total_tokens: 0,
        },
      };
      return new Response(JSON.stringify(res), {
        status: response.status,
        statusText: response.statusText,
        headers: response.headers,
      });
    } else if (response.headers.get("Content-Type")?.includes("stream")) {
      if (!response.body) {
        return response;
      }
      const isStream = !!context.req.body.stream;
      const result = {
        id: "",
        choices: [
          {
            finish_reason: null,
            index: 0,
            message: {
              content: "",
              role: "assistant",
            },
          },
        ],
        created: parseInt(new Date().getTime() / 1000 + "", 10),
        model: "",
        object: "chat.completion",
        usage: {
          completion_tokens: 0,
          prompt_tokens: 0,
          total_tokens: 0,
        },
      };

      const decoder = new TextDecoder();
      const encoder = new TextEncoder();

      let currentId = "";
      let currentModel = context?.req?.body?.model || "";

      let hasToolCall = false;
      let toolArgs = "";
      let toolId = "";
      let toolCallUsage = null;
      let contentIndex = 0;
      let hasThinking = false;

      const processLine = (line, controller, reader) => {
        console.log(line);

        if (line.startsWith("data:")) {
          const chunkStr = line.slice(5).trim();
          if (chunkStr) {
            try {
              let chunk = JSON.parse(chunkStr);

              if (chunk.type === "chat:completion") {
                const data = chunk.data;

                // 保存ID和模型信息
                if (data.id) currentId = data.id;
                if (data.model) currentModel = data.model;

                if (data.phase === "tool_call") {
                  if (!hasToolCall) hasToolCall = true;
                  const blocks = data.edit_content.split("<glm_block >");
                  blocks.forEach((block, index) => {
                    if (!block.includes("</glm_block>")) return;
                    if (index === 0) {
                      toolArgs += data.edit_content.slice(
                        0,
                        data.edit_content.indexOf('"result') - 3
                      );
                    } else {
                      if (toolId) {
                        try {
                          toolArgs += '"';
                          const params = JSON.parse(toolArgs);
                          if (!isStream) {
                            result.choices[0].message.tool_calls.slice(
                              -1
                            )[0].function.arguments = params;
                          } else {
                            const deltaRes = {
                              choices: [
                                {
                                  delta: {
                                    role: "assistant",
                                    content: null,
                                    tool_calls: [
                                      {
                                        id: toolId,
                                        type: "function",
                                        function: {
                                          name: null,
                                          arguments: params,
                                        },
                                      },
                                    ],
                                  },
                                  finish_reason: null,
                                  index: contentIndex,
                                  logprobs: null,
                                },
                              ],
                              created: parseInt(
                                new Date().getTime() / 1000 + "",
                                10
                              ),
                              id: currentId || "",
                              model: currentModel || "",
                              object: "chat.completion.chunk",
                              system_fingerprint: "fp_zai_001",
                            };
                            controller.enqueue(
                              encoder.encode(
                                `data: ${JSON.stringify(deltaRes)}\n\n`
                              )
                            );
                          }
                        } catch (e) {
                          console.log("解析错误", toolArgs);
                        } finally {
                          toolArgs = "";
                          toolId = "";
                        }
                      }
                      contentIndex += 1;
                      const content = JSON.parse(block.slice(0, -12));
                      toolId = content.data.metadata.id;
                      toolArgs += JSON.stringify(
                        content.data.metadata.arguments
                      ).slice(0, -1);

                      if (!isStream) {
                        if (!result.choices[0].message.tool_calls) {
                          result.choices[0].message.tool_calls = [];
                        }
                        result.choices[0].message.tool_calls.push({
                          id: toolId,
                          type: "function",
                          function: {
                            name: content.data.metadata.name,
                            arguments: "",
                          },
                        });
                      } else {
                        const startRes = {
                          choices: [
                            {
                              delta: {
                                role: "assistant",
                                content: null,
                                tool_calls: [
                                  {
                                    id: toolId,
                                    type: "function",
                                    function: {
                                      name: content.data.metadata.name,
                                      arguments: "",
                                    },
                                  },
                                ],
                              },
                              finish_reason: null,
                              index: contentIndex,
                              logprobs: null,
                            },
                          ],
                          created: parseInt(
                            new Date().getTime() / 1000 + "",
                            10
                          ),
                          id: currentId || "",
                          model: currentModel || "",
                          object: "chat.completion.chunk",
                          system_fingerprint: "fp_zai_001",
                        };
                        controller.enqueue(
                          encoder.encode(
                            `data: ${JSON.stringify(startRes)}\n\n`
                          )
                        );
                      }
                    }
                  });
                } else if (data.phase === "other") {
                  if (hasToolCall && data.usage) {
                    toolCallUsage = data.usage;
                  }
                  if (hasToolCall && data.edit_content?.startsWith("null,")) {
                    toolArgs += '"';
                    hasToolCall = false;
                    try {
                      const params = JSON.parse(toolArgs);
                      if (!isStream) {
                        result.choices[0].message.tool_calls.slice(
                          -1
                        )[0].function.arguments = params;
                        result.usage = toolCallUsage;
                        result.choices[0].finish_reason = "tool_calls";
                      } else {
                        const toolCallDelta = {
                          id: toolId,
                          type: "function",
                          function: {
                            name: null,
                            arguments: params,
                          },
                        };
                        const deltaRes = {
                          choices: [
                            {
                              delta: {
                                role: "assistant",
                                content: null,
                                tool_calls: [toolCallDelta],
                              },
                              finish_reason: null,
                              index: 0,
                              logprobs: null,
                            },
                          ],
                          created: parseInt(
                            new Date().getTime() / 1000 + "",
                            10
                          ),
                          id: currentId || "",
                          model: currentModel || "",
                          object: "chat.completion.chunk",
                          system_fingerprint: "fp_zai_001",
                        };
                        controller.enqueue(
                          encoder.encode(
                            `data: ${JSON.stringify(deltaRes)}\n\n`
                          )
                        );

                        const finishRes = {
                          choices: [
                            {
                              delta: {
                                role: "assistant",
                                content: null,
                                tool_calls: [],
                              },
                              finish_reason: "tool_calls",
                              index: 0,
                              logprobs: null,
                            },
                          ],
                          created: parseInt(
                            new Date().getTime() / 1000 + "",
                            10
                          ),
                          id: currentId || "",
                          usage: toolCallUsage || undefined,
                          model: currentModel || "",
                          object: "chat.completion.chunk",
                          system_fingerprint: "fp_zai_001",
                        };
                        controller.enqueue(
                          encoder.encode(
                            `data: ${JSON.stringify(finishRes)}\n\n`
                          )
                        );

                        controller.enqueue(encoder.encode(`data: [DONE]\n\n`));
                      }

                      reader.cancel();
                    } catch (e) {
                      console.log("错误", toolArgs);
                    }
                  }
                } else if (data.phase === "thinking") {
                  if (!hasThinking) hasThinking = true;
                  if (data.delta_content) {
                    const content = data.delta_content.startsWith("<details")
                      ? data.delta_content.split("</summary>\n>").pop().trim()
                      : data.delta_content;
                    if (!isStream) {
                      if (!result.choices[0].message?.thinking?.content) {
                        result.choices[0].message.thinking = {
                          content,
                        };
                      } else {
                        result.choices[0].message.thinking.content += content;
                      }
                    } else {
                      const msg = {
                        choices: [
                          {
                            delta: {
                              role: "assistant",
                              thinking: {
                                content,
                              },
                            },
                            finish_reason: null,
                            index: 0,
                            logprobs: null,
                          },
                        ],
                        created: parseInt(new Date().getTime() / 1000 + "", 10),
                        id: currentId || "",
                        model: currentModel || "",
                        object: "chat.completion.chunk",
                        system_fingerprint: "fp_zai_001",
                      };
                      controller.enqueue(
                        encoder.encode(`data: ${JSON.stringify(msg)}\n\n`)
                      );
                    }
                  }
                } else if (data.phase === "answer" && !hasToolCall) {
                  console.log(result.choices[0].message);
                  if (
                    data.edit_content &&
                    data.edit_content.includes("</details>\n")
                  ) {
                    if (hasThinking) {
                      const signature = Date.now().toString();
                      if (!isStream) {
                        result.choices[0].message.thinking.signature =
                          signature;
                      } else {
                        const msg = {
                          choices: [
                            {
                              delta: {
                                role: "assistant",
                                thinking: {
                                  content: "",
                                  signature,
                                },
                              },
                              finish_reason: null,
                              index: 0,
                              logprobs: null,
                            },
                          ],
                          created: parseInt(
                            new Date().getTime() / 1000 + "",
                            10
                          ),
                          id: currentId || "",
                          model: currentModel || "",
                          object: "chat.completion.chunk",
                          system_fingerprint: "fp_zai_001",
                        };
                        controller.enqueue(
                          encoder.encode(`data: ${JSON.stringify(msg)}\n\n`)
                        );
                        contentIndex++;
                      }
                    }
                    const content = data.edit_content
                      .split("</details>\n")
                      .pop();
                    if (content) {
                      if (!isStream) {
                        result.choices[0].message.content += content;
                      } else {
                        const msg = {
                          choices: [
                            {
                              delta: {
                                role: "assistant",
                                content,
                              },
                              finish_reason: null,
                              index: 0,
                              logprobs: null,
                            },
                          ],
                          created: parseInt(
                            new Date().getTime() / 1000 + "",
                            10
                          ),
                          id: currentId || "",
                          model: currentModel || "",
                          object: "chat.completion.chunk",
                          system_fingerprint: "fp_zai_001",
                        };
                        controller.enqueue(
                          encoder.encode(`data: ${JSON.stringify(msg)}\n\n`)
                        );
                      }
                    }
                  }
                  if (data.delta_content) {
                    if (!isStream) {
                      result.choices[0].message.content += data.delta_content;
                    } else {
                      const msg = {
                        choices: [
                          {
                            delta: {
                              role: "assistant",
                              content: data.delta_content,
                            },
                            finish_reason: null,
                            index: 0,
                            logprobs: null,
                          },
                        ],
                        created: parseInt(new Date().getTime() / 1000 + "", 10),
                        id: currentId || "",
                        model: currentModel || "",
                        object: "chat.completion.chunk",
                        system_fingerprint: "fp_zai_001",
                      };
                      controller.enqueue(
                        encoder.encode(`data: ${JSON.stringify(msg)}\n\n`)
                      );
                    }
                  }
                  if (data.usage && !hasToolCall) {
                    if (!isStream) {
                      result.choices[0].finish_reason = "stop";
                      result.choices[0].usage = data.usage;
                    } else {
                      const msg = {
                        choices: [
                          {
                            delta: {
                              role: "assistant",
                              content: "",
                            },
                            finish_reason: "stop",
                            index: 0,
                            logprobs: null,
                          },
                        ],
                        usage: data.usage,
                        created: parseInt(new Date().getTime() / 1000 + "", 10),
                        id: currentId || "",
                        model: currentModel || "",
                        object: "chat.completion.chunk",
                        system_fingerprint: "fp_zai_001",
                      };
                      controller.enqueue(
                        encoder.encode(`data: ${JSON.stringify(msg)}\n\n`)
                      );
                    }
                  }
                }
              }
            } catch (error) {
              console.error(error);
            }
          }
        }
      };

      if (!isStream) {
        const reader = response.body.getReader();
        let buffer = "";
        while (true) {
          const { done, value } = await reader.read();
          if (done) {
            break;
          }
          buffer += decoder.decode(value, { stream: true });
          const lines = buffer.split("\n");
          buffer = lines.pop() || "";
          for (const line of lines) {
            processLine(line, null, reader);
          }
        }

        return new Response(JSON.stringify(result), {
          status: response.status,
          statusText: response.statusText,
          headers: {
            "Content-Type": "application/json",
          },
        });
      }

      const stream = new ReadableStream({
        start: async (controller) => {
          const reader = response.body.getReader();
          let buffer = "";
          try {
            while (true) {
              const { done, value } = await reader.read();
              if (done) {
                // 发送[DONE]消息并清理状态
                controller.enqueue(encoder.encode(`data: [DONE]\n\n`));
                break;
              }

              buffer += decoder.decode(value, { stream: true });
              const lines = buffer.split("\n");

              buffer = lines.pop() || "";

              for (const line of lines) {
                processLine(line, controller, reader);
              }
            }
          } catch (error) {
            controller.error(error);
          } finally {
            controller.close();
          }
        },
      });

      return new Response(stream, {
        status: response.status,
        statusText: response.statusText,
        headers: {
          "Content-Type": "text/event-stream",
          "Cache-Control": "no-cache",
          Connection: "keep-alive",
        },
      });
    }
    return response;
  }
}

module.exports = ZAITransformer;

```

示例配置：
```json
{
  "LOG": false,
  "LOG_LEVEL": "debug",
  "CLAUDE_PATH": "",
  "HOST": "127.0.0.1",
  "PORT": 3456,
  "APIKEY": "sk-your-api-key",
  "API_TIMEOUT_MS": "600000",
  "PROXY_URL": "",
  "transformers": [
    {
      "name": "zai",
      "path": "/Users/sanbo/.claude-code-router/plugins/zai.js",
      "options": {}
    }
  ],
  "Providers": [
    {
      "name": "GLM",
      "api_base_url": "https://my2ldgg.deno.dev/v1/chat/completions",
      "api_key": "sk-your-api-key",
      "models": [
        "GLM-4.5",
        "GLM-4.5-Air"
      ],
      "transformers": {
        "use": [
          "zai"
        ]
      }
    }
  ],
  "Router": {
    "default": "GLM,GLM-4.5",
    "background": "GLM,GLM-4.5",
    "think": "GLM,GLM-4.5",
    "longContext": "GLM,GLM-4.5",
    "longContextThreshold": 60000,
    "webSearch": "GLM,GLM-4.5",
    "image": "GLM,GLM-4.5"
  }
}
```

**Q: 匿名模式是什么？**
A: 匿名模式使用临时 token，避免对话历史共享，保护隐私。

**Q: Function Call 如何工作？**
A: 通过智能提示注入实现，将工具定义转换为系统提示。

**Q: 支持哪些 OpenAI 功能？**
A: 支持聊天完成、模型列表、流式响应、工具调用等核心功能。

**Q: Function Call 如何优化？**
A: 改进了工具调用的请求响应结构，支持更复杂的工具链调用和并行执行。

**Q: 如何选择合适的模型？**
A: 
- **GLM-4.5**: 通用场景，性能和效果平衡
- **GLM-4.5-Thinking**: 需要了解推理过程的场景
- **GLM-4.5-Search**: 需要实时信息的场景
- **GLM-4.5-Air**: 高并发、低延迟要求的场景

**Q: 如何自定义配置？**
A: 通过环境变量配置，推荐使用 `.env` 文件。

## 🔑 获取 Z.ai API Token

要使用完整的多模态功能，需要获取正式的 Z.ai API Token：

### 方式 1: 通过 Z.ai 网站

1. 访问 [Z.ai 官网](https://chat.z.ai)
2. 注册账户并登录，进入 [Z.ai API Keys](https://z.ai/manage-apikey/apikey-list) 设置页面，在该页面设置 _**个人 API Token**_
3. 将 Token 放置在 `BACKUP_TOKEN` 环境变量中

### 方式 2: 浏览器开发者工具（临时方案）

1. 打开 [Z.ai 聊天界面](https://chat.z.ai)
2. 按 F12 打开开发者工具
3. 切换到 "Application" 或 "存储" 标签
4. 查看 Local Storage 中的认证 token
5. 复制 token 值设置为环境变量

> ⚠️ **注意**: 方式 2 获取的 token 可能有时效性，建议使用方式 1 获取长期有效的 API Token。
> ❗ **重要提示**: 多模态模型需要**官方 Z.ai API 非匿名 Token**，匿名 token 不支持多媒体处理。


## 🏗️ 技术架构

```
┌──────────────┐      ┌─────────────────────────┐      ┌─────────────────┐
│   OpenAI     │      │                         │      │                 │
│  Client      │────▶│    Oak Server (Deno)    │────▶│   Z.AI API      │
└──────────────┘      │                         │      │                 │
┌──────────────┐      │ ┌─────────────────────┐ │      │ ┌─────────────┐ │
│ Claude Code  │      │ │ /v1/chat/completions│ │      │ │0727-360B-API│ │
│   Router     │────▶│ └─────────────────────┘ │      │ └─────────────┘ │
└──────────────┘      │ ┌─────────────────────┐ │      │ ┌─────────────┐ │
                      │ │    /v1/models       │ │────▶│ │0727-106B-API│ │
                      │ └─────────────────────┘ │      │ └─────────────┘ │
                      │ ┌─────────────────────┐ │      │                 │
                      │ │  Enhanced Tools     │ │      └─────────────────┘
                      │ └─────────────────────┘ │
                      └─────────────────────────┘
                           OpenAI Compatible API
```

### 项目结构

```
z.ai2api-deno/
├── app/
│   ├── core/
│   │   ├── config.ts              # 配置管理
│   │   ├── openai.ts              # OpenAI API 实现
│   │   └── response_handlers.ts   # 响应处理器
│   ├── models/
│   │   └── schemas.ts             # Zod 模型定义
│   ├── utils/
│   │   ├── helpers.ts             # 辅助函数
│   │   ├── tools.ts               # 增强工具调用处理
│   │   └── sse_parser.ts          # SSE 流式解析器
├── deploy/                        # Docker 部署配置
├── main.ts                        # Oak 应用入口
├── deno.json                      # Deno 项目配置
└── README.md                      # 项目文档
```

## 🆚 Deno vs Python 版本对比

| 特性 | Python 版本 | Deno 版本 |
|------|-------------|-----------|
| 运行时 | Python 3.8+ | Deno 1.40+ |
| 框架 | FastAPI | Oak |
| 类型系统 | Pydantic | Zod |
| 包管理 | pip/uv | 内置 |
| 启动速度 | 较慢 | 更快 |
| 内存占用 | 较高 | 较低 |
| 类型安全 | 运行时验证 | 编译时检查 |
| 部署大小 | 较大 | 更小 |

## 🤝 贡献指南

我们欢迎所有形式的贡献！
请确保代码符合 Deno 标准，并更新相关文档。

## 📄 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

## ⚠️ 免责声明

- 本项目与 Z.AI 官方无关
- 使用前请确保遵守 Z.AI 服务条款
- 请勿用于商业用途或违反使用条款的场景
- 项目仅供学习和研究使用

---

<div align="center">
Made with ❤️ by the Linux.do community fork
</div>
