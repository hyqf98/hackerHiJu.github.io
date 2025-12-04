---
title: MCP服务
date: 2025-04-09 02:23:56
updated: 2025-04-09 02:23:56
tags:
  - AI
  - MCP
comments: false
categories: AI
thumbnail: https://images.unsplash.com/photo-1682687220923-c58b9a4592ae?crop=entropy&cs=srgb&fm=jpg&ixid=M3w2NDU1OTF8MHwxfHJhbmRvbXx8fHx8fHx8fDE3NDQxNzk4MzV8&ixlib=rb-4.0.3&q=85&w=1920&h=1080
published: false
---
# MCP服务

## 1. 简介

模型上下文协议（MCP）是一个创新的开源协议，它重新定义了大语言模型（LLM）与外部世界的互动方式。MCP 提供了一种标准化方法，使任意大语言模型能够轻松连接各种数据源和工具，实现信息的无缝访问和处理。MCP 就像是 AI 应用程序的 USB-C 接口，为 AI 模型提供了一种标准化的方式来连接不同的数据源和工具。
### 1.1 服务架构

![MCP服务架构|500](images/MCP服务架构.svg)

### 1.2 Agent架构

![Agent架构|500](images/Agent架构.svg)
### 1.3 MCP流程调用

![|500](images/MCP流程调用.svg)

### 1.4 官方架构
MCP就像是USB-C一样，可以让不同设备通过相同的接口连接在一起

![|500](images/MCP服务-1744869631283.png)

## 2. Python MCP

### 2.1 简单客户端

通过启动本地的一个客户端来实现循环对话调用大模型

```python
import asyncio  
from mcp import ClientSession  
from openai import OpenAI  
from contextlib import AsyncExitStack  
  
BASE_URL = "https://dashscope.aliyuncs.com/compatible-mode/v1"  
MODEL_NAME = "qwen-max"  
KEY = "" 
  
class MCPClient:  
  
    def __init__(self):  
        """初始化MCP客户端"""  
        self.session = None  
        self.exit_stack = AsyncExitStack()  
        self.openai_api_key = KEY  
        self.openai_api_base = BASE_URL  
        self.model = MODEL_NAME  
  
        if not self.openai_api_key:  
            raise ValueError("请设置您的OpenAI API密钥")  
  
        self.client = OpenAI(  
            api_key=self.openai_api_key,  
            base_url=self.openai_api_base,  
        )  
  
    # 处理对话请求  
    async def process_query(self, query: str) -> str:  
        messages = [{  
            "role": "system",  
            "content": "你是一个智能助手，帮助用户回答问题",  
        }, {  
            "role": "user",  
            "content": query,  
        }]  
        try:  
            response = await asyncio.get_event_loop().run_in_executor(  
                None,  
                lambda: self.client.chat.completions.create(  
                    model=MODEL_NAME,  
                    messages=messages  
                )  
            )  
            return response.choices[0].message.content  
        except Exception as e:  
            return f"调用OpenAI API错误：{str(e)}"  
  
    async def connect_to_mock_server(self):  
        """模拟 MCP 服务器的连接 """        print("Connecting to mock MCP server...")  
  
    async def chat_loop(self):  
        """运行交互式聊天循环"""  
        print("\n MCP 客户端已启动！输入 ‘quit’ 退出")  
  
        while True:  
            try:  
                user_input = input("请输入您的问题：").strip()  
                if user_input.lower() == "quit":  
                    print("退出交互式聊天")  
                    break  
                response = await self.process_query(user_input)  
                print(f"大模型：{response}")  
            except Exception as e:  
                print(f"发生错误：{str(e)}")  
  
    async def cleanup(self):  
        """清理资源"""  
        print("Cleaning up resources...")  
        await self.exit_stack.aclose()  
  
  
async def main():  
    mcp_client = MCPClient()  
  
    try:  
        await mcp_client.connect_to_mock_server()  
        await mcp_client.chat_loop()  
    finally:  
        await mcp_client.cleanup()  
  
  
if __name__ == '__main__':  
    asyncio.run(main())
```

### 2.2 MCP服务器通讯机制

**Model Context Protocol(MCP)** 是一种由Anthropic开源的协议，旨在将大型语言模型直接连接至数据源，实现无缝集成。根据 MCP 的规范，当前支持两种传输方式:
- 标准输入输出(stdio)：打开文件流的方式进行传输（同一个服务器，不需要通过端口监听）
- 基于HTTP的服务器推送事件(SSE)：在不同的机器分布式部署

而近期，开发者在MCP 的 GitHub 仓库中提交了一项提案，建议采用 **"可流式传输的HTTP（Streamable Http）"** 来替代现有的 HTTP+SSE方案。此举旨在解决当前远程 MCP 传输方式的关键限制，同时保留其优势。HTTP和SSE(服务器推送事件)在数据传输方式上存在明显区别

- 通信方式
	- HTTP：采用请求-响应模式，客户端发送请求，服务器返回响应，每次请求都是独立的 
	- SSE：允许服务器通过单个持久的HTTP连接，持续向客户端推送数据连
- 连接特性
	- HTTP：每次请求通常建立新的连接，虽然在HTTP/1.1中引入了持久连是短连接
	- SSE：适用于需要服务器主动向客户端推送数据的场景，如实时通知、股票行情更新等

注意：SSE仅支持服务器向客户端的单向通信，而websocket则是双向通信


|     传输方式      | 是否需要同时启动服务器 | 是否支持远程连接 |     使用场景      |
| :-----------: | :---------: | :------: | :-----------: |
| stdio（标准输入输出） |      ✔      |    ❌     | 本地通信，低延迟，高速交互 |
|  http（网络api）  |      ❌      |    ✔     |  分布式架构，远程通信   |
### 2.3 简单天气查询服务端

通过 **\@mcp.tool()** 来标注服务端提供的工具有哪些

```python
from typing import Any  
from mcp.server.fastmcp import FastMCP  
mcp = FastMCP("WeatherServer")  
  
async def fetch_weather(city: str) -> dict[str, Any] | None:  
    """  
    获取天气的api  
    :param city: 城市名称（需要使用英文，如Beijing）  
    :return: 天气数据字典；若出错返回包含 error信息的字典  
    """    return {  
        "城市": f"{city}\n",  
        "温度": "25.5°C\n",  
        "湿度": "25%\n",  
        "风俗": "12 m/s\n",  
        "天气": "晴\n",  
    } 
  
@mcp.tool()  
async def get_weather(city: str) -> dict[str, Any] | None:  
    """  
    获取天气  
    :param city: 城市名称（需要使用英文，如Beijing）  
    :return: 天气数据字典；若出错返回包含 error信息的字典  
    """    return await fetch_weather(city)  
  
if __name__ == '__main__':  
    # 使用标准 I/O 方式运行MCP服务器  
    mcp.run(transport='stdio')
```

### 2.4 天气查询客户端

```python
import asyncio  
import json  
from typing import Optional  
  
from mcp import ClientSession, StdioServerParameters  
from mcp.client.stdio import stdio_client  
from openai import OpenAI  
from contextlib import AsyncExitStack  
  
BASE_URL = "https://dashscope.aliyuncs.com/compatible-mode/v1"  
MODEL_NAME = "qwen-max"  
KEY = "xxx"  
  
  
class MCPClient:  
  
    def __init__(self):  
        """初始化MCP客户端"""  
        self.openai_api_key = KEY  
        self.openai_api_base = BASE_URL  
        self.model = MODEL_NAME  
  
        if not self.openai_api_key:  
            raise ValueError("请设置您的OpenAI API密钥")  
  
        self.client = OpenAI(  
            api_key=self.openai_api_key,  
            base_url=self.openai_api_base,  
        )  
        self.session: Optional[ClientSession] = None  
        self.exit_stack = AsyncExitStack()  
  
    # 处理对话请求  
    async def process_query(self, query: str) -> str:  
        messages = [{  
            "role": "system",  
            "content": "你是一个智能助手，帮助用户回答问题",  
        }, {  
            "role": "user",  
            "content": query,  
        }]  
  
        # 获取到工具列表  
        response = await self.session.list_tools()  
        available_tools = [  
            {  
                "type": "function",  
                "function": {  
                    "name": tool.name,  
                    "description": tool.description,  
                    "input_schema": tool.inputSchema,  
                }  
            }  
            for tool in response.tools]  
  
        try:  
            response = await asyncio.get_event_loop().run_in_executor(  
                None,  
                lambda: self.client.chat.completions.create(  
                    model=MODEL_NAME,  
                    messages=messages,  
                    tools=available_tools,  
                )  
            )  
            content = response.choices[0]  
            if content.finish_reason == "tool_calls":  
                # 如果使用的是工具，解析工具  
                tool_call = content.message.tool_calls[0]  
                tool_name = tool_call.function.name  
                tool_args = json.loads(tool_call.function.arguments)  
  
                # 执行工具  
                result = await self.session.call_tool(tool_name, tool_args)  
                print(f"\n\n工具调用:[{tool_name}]，参数:[{tool_args}]")  
  
                # 将工具返回结果存入message中，model_dump()克隆一下消息  
                messages.append(content.message.model_dump())  
                messages.append({  
                    "role": "tool",  
                    "content": result.content[0].text,  
                    "tool_call_id": tool_call.id,  
                })  
  
                response = self.client.chat.completions.create(  
                    model=MODEL_NAME,  
                    messages=messages,  
                )  
  
                return response.choices[0].message.content  
  
            # 正常返回  
            return content.message.content  
        except Exception as e:  
            return f"调用OpenAI API错误：{str(e)}"  
  
    async def connect_to_server(self, server_script_path: str):  
        """连接到 MCP 服务器的连接 """        is_python = server_script_path.endswith(".py")  
        is_js = server_script_path.endswith(".js")  
        if not (is_python or is_js):  
            raise ValueError("服务器脚本路径必须以 .py 或 .js 结尾")  
  
        command = "python" if is_python else "node"  
  
        server_params = StdioServerParameters(  
            command=command,  
            args=[server_script_path],  
            env=None  
        )  
  
        stdio_transport = await self.exit_stack.enter_async_context(  
            stdio_client(server_params)  
        )  
        read, write = stdio_transport  
        self.session = await self.exit_stack.enter_async_context(ClientSession(read, write))  
        # 初始化会话  
        await self.session.initialize()  
        # 列出工具  
        response = await self.session.list_tools()  
        tools = response.tools  
        print("\n已经连接到服务器，支持以下工具：", [tools.name for tools in tools])  
  
    async def chat_loop(self):  
        """运行交互式聊天循环"""  
        print("\n MCP客户端已启动！输入 ‘quit’ 退出")  
  
        while True:  
            try:  
                user_input = input("请输入您的问题：").strip()  
                if user_input.lower() == "quit":  
                    print("退出交互式聊天")  
                    break  
                response = await self.process_query(user_input)  
                print(f"大模型：{response}")  
            except Exception as e:  
                print(f"发生错误：{str(e)}")  
  
    async def cleanup(self):  
        """清理资源"""  
        print("Cleaning up resources...")  
        await self.exit_stack.aclose()  
  
  
async def main():  
    mcp_client = MCPClient()  
  
    try:  
        await mcp_client.connect_to_server("./mcp_server.py")  
        await mcp_client.chat_loop()  
    finally:  
        await mcp_client.cleanup()  
  
  
if __name__ == '__main__':  
    asyncio.run(main())
```

### 2.5 MCP概念

- Tools：服务器暴露可执行功能，供LLM调用以与外部系统交互
- Resources：服务器暴露数据和内容，供客户端读取并作为LLM上下文
- Prompts：服务器定义可复用的提示模板，引导LLM交互
- Sampling：让服务器借助客户端向LLM发起完成请求，实现复杂的智能行为
- Roots：客户端给服务器指定的一些地址，用来高速服务器该关注哪些资源和去哪里找这些资源

#### 2.5.1 Tools

服务器所支持的工具能力，使用提供的装饰器就可以定义对应的工具

```python
@mcp.tool()  
async def get_weather(city: str) -> dict[str, Any] | None:  
    """  
    获取天气  
    :param city: 城市名称（需要使用英文，如Beijing）  
    :return: 天气数据字典；若出错返回包含 error信息的字典  
    """    return await fetch_weather(city)
```

服务端连接通了session会话就可以通过对应的代码来进行查询支持哪些工具

```python
session.list_tools()
```

#### 2.5.2 Resources
类似于服务端定义了一个api接口用于查询数据，可以给大模型提供上下文

```python
@mcp.resource(uri="echo://hello")  
def resource() -> str:  
    """Echo a message as a resource"""  
    return f"Resource echo: hello"  
  
  
@mcp.resource(uri="echo://{message}/{age}")  
def message(message: str, age: int) -> str:  
    """Echo a message as a resource"""  
    return f"你好，{message}，{age}"
```

服务端查询时，如果使用了 {message} 作为占位符会解析为 **resource_templates** 使用 **list_resources** 只能获取到普通的资源
```python
# 查询资源  
resources = await self.session.list_resources()  
print("\n已经连接到服务器，支持以下资源：", [resources.name for resources in resources.resources])  
  
# 查询资源  
templates = await self.session.list_resource_templates()  
print("\n已经连接到服务器，支持以下模板资源：", [resources.name for resources in templates.resourceTemplates])  
  
resource_result = await self.session.read_resource("echo://hello")  
for content in resource_result.contents:
	# 对返回的字符串进行编码
    print(f"读取资源内容：{unquote(content.text)}")  
  
resource_result = await self.session.read_resource("echo://张三/18")  
for content in resource_result.contents:  
    print(f"读取资源内容：{unquote(content.text)}")
```

#### 2.5.3 Prompt
提示词，用于在服务端定义好自己的提示词来进行复用

```python
@mcp.prompt()  
def review_code(code: str) -> str:  
    return f"Please review this code:\n\n{code}"  
  
@mcp.prompt()  
def debug_error(error: str) -> list[base.Message]:  
    return [  
        base.UserMessage("I'm seeing this error:"),  
        base.UserMessage(error),  
        base.AssistantMessage("I'll help debug that. What have you tried so far?"),  
    ]
```

```python
# 查询提示词  
prompt_result = await self.session.list_prompts()  
print("\n已经连接到服务器，支持以下提示词：", [prompt.name for prompt in prompt_result.prompts])  
  
get_prompt = await self.session.get_prompt("review_code", { "code": "hello world"})  
for message in get_prompt.messages:  
    print(f"提示词内容：{message}")
```

#### 2.5.4 Images
MCP提供的一个Image类，可以自动处理图像数据

```python
from mcp.server.fastmcp import FastMCP, Image

@mcp.tool()  
def create_thumbnail(image_url: str) -> Image:  
    """Create a thumbnail from an image"""  
    img = PILImage.open(image_url)  
    img.thumbnail((100, 100))  
    return Image(data=img.tobytes(), format="jpg")
```

```python
# 调用图片工具  
image = await self.session.call_tool("create_thumbnail", {"image_url": "/Users/Documents/图片/WechatIMG47.jpg"})  
print("读取图片资源：", image)
```

#### 2.5.5 Context
Context 对象为您的工具和资源提供对 MCP 功能的访问权限，在服务端的工具中可以调用对应的资源数据

```python
from mcp.server.fastmcp import FastMCP, Context

@mcp.tool()  
async def test(city: str, ctx: Context) -> str:  
    """  
    获取天气  
    :param city: 城市名称（需要使用英文，如Beijing）  
    :return: 天气描述  
    """    get_weather_city = await ctx.read_resource(f"echo://{city}/25")  
    result: str = ""  
    for content in get_weather_city:  
        result += unquote(content.content)  
    return result
```

```python
# 调用天气工具使用Context对象  
weather = await self.session.call_tool(name="test", arguments={"city": "北京"})  
print("天气信息：", weather)
```

#### 2.5.6 Server

在测试时使用的例子，都是使用的框架默认提供的方式来构建对应的工具、资源等信息，如果需要自己来组建工具则可以通过自定义的方式来实现服务端通过什么方式来组合对应的工具、资源等信息。

- @asynccontextmanager  ：将方法包裹为一个支持with as的对象，要求返回的数据是一个生成器对象
- Callable：函数的类型标识方式，前面 ... 表示多个参数，后面表示返回值是任意类型

自定义Server提供了更加灵活的方式来组合资源、工具，包括服务启动的生命周期流程的控制

```python
from contextlib import asynccontextmanager  
from typing import AsyncIterator, Callable, Any  
  
import mcp.server.stdio  
import mcp.types as types  
from fastmcp import Context  
from mcp.server.lowlevel import NotificationOptions, Server  
from mcp.server.models import InitializationOptions  
  
  
@asynccontextmanager  
async def server_lifespan(server: Server) -> AsyncIterator[dict]:  
    """Manage server startup and shutdown lifecycle."""  
    # Initialize resources on startup  
    try:  
        yield {"message": "hello"}  
    finally:  
        print("最终执行")  
  
  
class ExampleServer(Server):  
    tools: types.Tool = []  
  
    def __init__(self, name: str, lifespan: Any):  
        super().__init__(name, lifespan)  
  
    def tool(  
            self, name: str | None = None, description: str | None = None  
    ) -> Callable[[Callable[..., Any]], Callable[..., Any]]:  
        if callable(name):  
            raise TypeError(  
                "The @tool decorator was used incorrectly. "  
                "Did you forget to call it? Use @tool() instead of @tool"            )  
  
        def decorator(fn: Callable[..., Any]) -> Callable[..., Any]:  
            self.tools.append(  
                types.Tool(  
                    name=name or fn.__name__,  
                    description=description or fn.__doc__ or "",  
                    inputSchema={  
                        "properties": {  
                            "city": {  
                                "title": "City",  
                                "type": "string"  
                            }  
                        },  
                        "required": ["city"],  
                        "title": "example-tool",  
                        "type": "object"  
                    }  
                )  
            )  
            return fn  
  
        return decorator  
  
    def get_tools(self):  
        return self.tools  
  
  
# 指定服务的生命周期函数  
server = ExampleServer("example-server", lifespan=server_lifespan)  
  
  
@server.list_prompts()  
async def handle_list_prompts() -> list[types.Prompt]:  
    return [  
        types.Prompt(  
            name="example-prompt",  
            description="An example prompt template",  
            arguments=[  
                types.PromptArgument(  
                    name="arg1", description="Example argument", required=True  
                )  
            ],  
        )  
    ]  
  
  
@server.get_prompt()  
async def handle_get_prompt(  
        name: str, arguments: dict[str, str] | None  
) -> types.GetPromptResult:  
    if name != "example-prompt":  
        raise ValueError(f"Unknown prompt: {name}")  
  
    return types.GetPromptResult(  
        description="Example prompt",  
        messages=[  
            types.PromptMessage(  
                role="user",  
                content=types.TextContent(type="text", text="Example prompt text"),  
            )  
        ],  
    )  
  
  
@server.list_tools()  
async def handle_list_tools() -> list[types.Tool]:  
    return server.get_tools()  
  
  
@server.list_resources()  
async def handle_list_resources() -> list[types.Resource]:  
    return [  
        types.Resource(  
            name="example-resource",  
            description="An example resource",  
            content=types.TextContent(type="text", text="Example resource text"),  
        )  
    ]  
  
  
@server.tool()  
async def test(city: str, ctx: Context) -> str:  
    """测试方法"""  
    return ctx.lifespan_context["db"]  
  
  
async def run():  
    async with mcp.server.stdio.stdio_server() as (read_stream, write_stream):  
        await server.run(  
            read_stream,  
            write_stream,  
            InitializationOptions(  
                server_name="example",  
                server_version="0.1.0",  
                capabilities=server.get_capabilities(  
                    notification_options=NotificationOptions(),  
                    experimental_capabilities={},  
                ),  
            ),  
        )  
  
  
if __name__ == "__main__":  
    import asyncio  
  
    asyncio.run(run())
```

### 2.6 Sampling

MCP为我们提供的一个在执行工具前后可以执行的一些操作，类似回调函数；下面一个简单例子说明了如何进行注册 **Sampling** 来进行函数的回调

```python
# 服务端  
from mcp.server import FastMCP  
from mcp.server.fastmcp import Context  
from mcp.types import SamplingMessage, TextContent  
  
mcp = FastMCP('file_server')  
  
  
@mcp.tool()  
async def delete_file(file_path: str, ctx: Context):  
    # 创建 SamplingMessage 用于触发 sampling callback 函数  
    mcp.get_context()  
    result = await ctx.session.create_message(  
        messages=[  
            SamplingMessage(  
                role='user',  
                content=TextContent(type='text', text=f'是否要删除文件: {file_path} (Y)')  
            )  
        ],  
        max_tokens=100  
    )  
  
    # 获取到 sampling callback 函数的返回值，并根据返回值进行处理  
    if result.content.text == 'Y':  
        return f'文件 {file_path} 已被删除！！'  
  
  
if __name__ == '__main__':  
    mcp.run(transport='stdio')
```

```python
# 客户端  
import asyncio  
  
from mcp.client.stdio import stdio_client  
from mcp import ClientSession, StdioServerParameters  
from mcp.shared.context import RequestContext  
from mcp.types import (  
    TextContent,  
    CreateMessageRequestParams,  
    CreateMessageResult,  
)  
  
server_params = StdioServerParameters(  
    command='python',  
    args=['./mcp_sampling_server.py'],  
)  
  
  
async def sampling_callback(context: RequestContext[ClientSession, None], params: CreateMessageRequestParams):  
    # 获取工具发送的消息并显示给用户  
    input_message = input(params.messages[0].content.text)  
    # 将用户输入发送回工具  
    return CreateMessageResult(  
        role='user',  
        content=TextContent(  
            type='text',  
            text=input_message.strip().upper() or 'Y'  
        ),  
        model='user-input',  
        stopReason='endTurn'  
    )  
  
  
async def main():  
    async with stdio_client(server_params) as (stdio, write):  
        async with ClientSession(  
                stdio, write,  
                # 设置 sampling_callback 对应的方法  
                sampling_callback=sampling_callback  
        ) as session:  
            await session.initialize()  
            res = await session.call_tool(  
                'delete_file',  
                {'file_path': 'xxx.txt'}  
            )  
            # 获取工具最后执行完的返回结果  
            print(res)  
  
  
if __name__ == '__main__':  
    asyncio.run(main())
```

上面的客户端使用的 **create_session**打印数据是无法显示到控制台命令行里面，所以可以替换为 **create_connected_server_and_client_session**

```python
# 客户端
from mcp.shared.memory import (
    create_connected_server_and_client_session as create_session
)
# 这里需要引入服务端的 app 对象
from mcp_sampling_server import mcp

async def sampling_callback(context, params):
    ...

async def main():
    async with create_session(
        mcp._mcp_server,
        sampling_callback=sampling_callback
    ) as client_session:
        await client_session.call_tool(
            'delete_file', 
            {'file_path': 'xxx.txt'}
        )

if __name__ == '__main__':
    asyncio.run(main())
```

### 2.7 MCP生命周期

MCP 生命周期分为3个阶段：
- 初始化
- 交互通信中
- 服务被关闭
因此，我们可以在这个三个阶段的开始和结束来做一些事情，比如创建数据库连接和关闭数据库连接、记录日志、记录工具使用信息等。

```python
import httpx
from dataclasses import dataclass
from contextlib import asynccontextmanager

from mcp.server import FastMCP
from mcp.server.fastmcp import Context

@dataclass
# 初始化一个生命周期上下文对象
class AppContext:
    # 里面有一个字段用于存储请求历史
    histories: dict

@asynccontextmanager
async def app_lifespan(server):
    # 在 MCP 初始化时执行
    histories = {}
    try:
        # 每次通信会把这个上下文通过参数传入工具
        yield AppContext(histories=histories)
    finally:
        # 当 MCP 服务关闭时执行
        print(histories)

mcp = FastMCP(
    'web-search', 
    # 设置生命周期监听函数
    lifespan=app_lifespan
)

@mcp.tool()
# 第一个参数会被传入上下文对象
async def web_search(ctx: Context, query: str) -> str:
    """
    搜索互联网内容
    Args:
        query: 要搜索内容
    Returns:
        搜索结果的总结
    """
    # 如果之前问过同样的问题，就直接返回缓存
    histories = ctx.request_context.lifespan_context.histories
    if query in histories：
    	return histories[query]

    async with httpx.AsyncClient() as client:
        response = await client.post(
            'https://open.bigmodel.cn/api/paas/v4/tools',
            headers={'Authorization': 'YOUR API KEY'},
            json={
                'tool': 'web-search-pro',
                'messages': [
                    {'role': 'user', 'content': query}
                ],
                'stream': False
            }
        )

        res_data = []
        for choice in response.json()['choices']:
            for message in choice['message']['tool_calls']:
                search_results = message.get('search_result')
                if not search_results:
                    continue
                for result in search_results:
                    res_data.append(result['content'])

        return_data = '\n\n\n'.join(res_data)

        # 将查询值和返回值存入到 histories 中
        ctx.request_context.lifespan_context.histories[query] = return_data
        return return_data

if __name__ == "__main__":
    mcp.run()
```
### 2.8 MCP接入SSE

通过轻量级web框架 **starlette** 来接入 sse协议

```cmd
pip install starlette
```

```python
# 服务端  
import asyncio  
  
from mcp.server.fastmcp import FastMCP  
from starlette.applications import Starlette  
from mcp.server.sse import SseServerTransport  
from starlette.requests import Request  
from starlette.routing import Mount, Route  
from mcp.server import Server  
import uvicorn  
  
mcp = FastMCP('weather')  
  
  
@mcp.tool()  
async def get_weather(city: str) -> str:  
    """获取天气信息"""  
    return "天气信息"  
  
  
def create_starlette_app(mcp_server: Server, *, debug: bool = False) -> Starlette:  
    """创建一个starlette应用来支持sse协议"""  
    sse = SseServerTransport("/messages/")  
  
    async def handle_sse(request: Request) -> None:  
        async with sse.connect_sse(  
                request.scope,  
                request.receive,  
                request._send,  
        ) as (read_stream, write_stream):  
            await mcp_server.run(  
                read_stream,  
                write_stream,  
                mcp_server.create_initialization_options(),  
            )  
  
    return Starlette(  
        debug=debug,  
        routes=[  
            Route("/sse", endpoint=handle_sse),  
            Mount("/messages/", app=sse.handle_post_message),  
        ],  
    )  
  
  
if __name__ == '__main__':  
    mcp_server = mcp._mcp_server  
  
    starlette_app = create_starlette_app(mcp_server, debug=True)  
  
    uvicorn.run(starlette_app, port=9000)
```

```python
import asyncio  
from contextlib import AsyncExitStack  
from typing import Optional  
  
from mcp import ClientSession  
from mcp.client.sse import sse_client  
  
  
class MCPClient:  
    def __init__(self):  
        self.session: Optional[ClientSession] = None  
        self.exit_stack = AsyncExitStack()  
  
    async def connect_to_sse_server(self, server_url: str):  
        """Connect to an MCP server running with SSE transport"""  
  
        async with sse_client(url=server_url) as (read, write):  
            async with ClientSession(read, write) as session:  
                await session.initialize()  
                print("初始化SSE客户端...")  
                response = await session.list_tools()  
                tools = response.tools  
                print("\n获取到的工具:", [tool.name for tool in tools])  
  
    async def cleanup(self):  
        """Properly clean up the session and streams"""  
  
  
async def main():  
    client = MCPClient()  
    try:  
        await client.connect_to_sse_server(server_url="http://127.0.0.1:9000/sse")  
    finally:  
        pass
  
  
if __name__ == "__main__":  
    import sys  
  
    asyncio.run(main())
```
## 3. Spring AI MCP

### 3.1 依赖版本

Spring AI组件提供MCP服务组件框架集成，目前框架只支持 **Spring Boot 3.4.x**

```xml
<repositories>  
    <repository>        
	    <id>spring-snapshots</id>  
        <name>Spring Snapshots</name>  
        <url>https://repo.spring.io/snapshot</url>  
        <releases>            
			<enabled>false</enabled>  
        </releases>
	</repository>    
	<repository>        
		<name>Central Portal Snapshots</name>  
	     <id>central-portal-snapshots</id>  
	     <url>https://central.sonatype.com/repository/maven-snapshots/</url>  
	     <releases>           
		     <enabled>false</enabled>  
	    </releases>
	    <snapshots>            
		    <enabled>true</enabled>  
		</snapshots>
	</repository>
</repositories>
```

```xml
<dependency>  
    <groupId>org.springframework.ai</groupId>  
    <artifactId>spring-ai-bom</artifactId>  
    <version>1.0.0-M7</version>  
    <type>pom</type>  
    <scope>import</scope>  
</dependency>
```

注意：1.0.0-M6版本开始之后就不需要在配置上面的仓库了，中央仓库已经上传了

### 3.2 简单MCP服务端

```xml
<dependencies>  
    <dependency>  
	    <groupId>org.springframework.ai</groupId>  
	    <artifactId>spring-ai-starter-mcp-server</artifactId>  
	</dependency>
</dependencies>
```

#### 3.2.1 启动类
```java
@SpringBootApplication  
public class McpServerApplication {  
  
    /**  
     * Main     *     * @param args args  
     * @since 1.0.0  
     */    public static void main(String[] args) {  
        SpringApplication.run(McpServerApplication.class, args);  
    }  
  
}
```

#### 3.2.2 工具服务
```java
@Service  
public class WeatherService {  
  
    /**  
     * Query weather     *     * @param city city  
     * @return the string  
     * @since 1.0.0  
     */    @Tool(description = "天气查询")  
    public String queryWeather(String city) {  
        return "天气查询";  
    }  
  
}
```

#### 3.2.3 注册工具
```java
@Configuration  
public class McpServerAutoConfiguration {  
  
    /**  
     * Weather tools     *     * @param weatherService weather service  
     * @return the tool callback provider  
     * @since 1.0.0  
     */    @Bean  
    public ToolCallbackProvider weatherTools(WeatherService weatherService) {  
        return MethodToolCallbackProvider.builder()  
                .toolObjects(weatherService)  
                .build();  
    }  
  
}
```

#### 3.2.4 配置文件
```yml
spring:  
  ai:  
    mcp:  
      server:  
        name: stdio-mcp-server  
        version: 1.0.0  
        type: SYNC  
        stdio: true  
  main:  
    banner-mode: off  
logging:  
  pattern:  
    console:
```

注意：创建的配置文件，必须要禁用banner否则客户端读取时就会出现以下错误
```txt
10:15:40.979 [pool-1-thread-1] ERROR io.modelcontextprotocol.client.transport.StdioClientTransport -- Error processing inbound message for line: 
com.fasterxml.jackson.databind.exc.MismatchedInputException: No content to map due to end-of-input
 at [Source: REDACTED (`StreamReadFeature.INCLUDE_SOURCE_IN_LOCATION` disabled); line: 1]
	at com.fasterxml.jackson.databind.exc.MismatchedInputException.from(MismatchedInputException.java:59)
	at com.fasterxml.jackson.databind.ObjectMapper._initForReading(ObjectMapper.java:5008)
	at com.fasterxml.jackson.databind.ObjectMapper._readMapAndClose(ObjectMapper.java:4910)
	at com.fasterxml.jackson.databind.ObjectMapper.readValue(ObjectMapper.java:3860)
	at com.fasterxml.jackson.databind.ObjectMapper.readValue(ObjectMapper.java:3843)
	at io.modelcontextprotocol.spec.McpSchema.deserializeJsonRpcMessage(McpSchema.java:153)
	at io.modelcontextprotocol.client.transport.StdioClientTransport.lambda$startInboundProcessing$6(StdioClientTransport.java:261)
	at reactor.core.scheduler.SchedulerTask.call(SchedulerTask.java:68)
	at reactor.core.scheduler.SchedulerTask.call(SchedulerTask.java:28)
	at java.base/java.util.concurrent.FutureTask.run$$$capture(FutureTask.java:264)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1136)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:635)
	at java.base/java.lang.Thread.run(Thread.java:840)
```

### 3.3 简单客户端

```xml
<dependencies>  
    <dependency>  
	    <groupId>org.springframework.ai</groupId>  
	    <artifactId>spring-ai-starter-mcp-client</artifactId>  
	</dependency>
</dependencies>
```

#### 3.3.1 启动类
```java
@SpringBootApplication  
public class McpClientApplication {  
  
    /**  
     * Main     *     * @param args args  
     * @since 1.0.0  
     */    public static void main(String[] args) {  
        SpringApplication.run(McpClientApplication.class, args);  
    }  
  
    @Bean  
    public CommandLineRunner predefinedQuestions() {  
        return args -> {  
            String jarFile = "xxx.jar";  
            ServerParameters serverParameters = new ServerParameters.Builder("java")  
                    .args("-jar", "-Dfile.encoding=UTF-8", "-Dspring.ai.mcp.server.stdio=true", jarFile)  
                    .build();  
            StdioClientTransport transport = new StdioClientTransport(serverParameters);  
            McpSyncClient syncClient = McpClient.sync(transport).build();  
            syncClient.listTools().tools().forEach(tool -> {  
                System.out.println("工具数据：" + tool.name());  
            });  
        };
    } 
}
```

### 3.4 通讯方式

#### 3.4.1 stdio

标准的输入输出的方式进行通讯，跟python一样可以指定java启动某个java包，也可以使用npx启动某个npm的包

```yaml
spring:  
  ai:  
    mcp:  
      client:  
        stdio:  
          connections:  
            server1:  
              command: java  
              args:  
                - -jar  
                - xxx.jar  
            server2:  
              command: npx  
              args:  
                - -y  
                - "@modelcontextprotocol/server-filesystem"  
                - /Users/haijun/Work/my-work/
```

#### 3.4.2 sse

要支持sse可以使用以下依赖组件，下面的主键也同时支持stdio的方式通信

```xml
<dependency> 
	<groupId>org.springframework.ai</groupId> 
	<artifactId>spring-ai-starter-mcp-server-webmvc</artifactId> 
</dependency>
```

客户端配置
```yml
spring:  
  ai:  
    mcp:  
      client:  
        sse:  
          connections:  
            server1: http://localhost:8080
```

服务端配置
```java
spring:  
  ai:  
    mcp:  
      server:  
        name: stdio-mcp-server  
        version: 1.0.0  
        sse-endpoint: /sse  
        sse-message-endpoint: /mcp/message
```

只需要修改配置即可，其余代码跟上面例子使用一样

#### 3.4.3 webflux

webflux的依赖包是同时兼容了，sse、stdio等方式进行通信，其余逻辑都是一样只是底层的代码变成了响应式
```xml
<dependency> 
	<groupId>org.springframework.ai</groupId> 
	<artifactId>spring-ai-starter-mcp-server-webflux</artifactId> 
</dependency>

<dependency>  
    <groupId>org.springframework.ai</groupId>  
    <artifactId>spring-ai-starter-mcp-client-webflux</artifactId>  
</dependency>
```

### 3.5 常用API

#### 3.5.1 Tools

##### ToolCallbackProvider

![|500](images/MCP服务-1744872737197.png)

- MethodToolCallbackProvider：方法工具回调函数提供器
- StaticToolCallbackProvider：里面包含了FunctionCallback包装了一层
- SyncMcpToolCallbackProvider/AsyncMcpToolCallbackProvider：同步异步工具回调函数提供器

```java
@Bean  
public ToolCallbackProvider weatherTools(WeatherService weatherService) {  
    return MethodToolCallbackProvider.builder()  
            .toolObjects(weatherService)  
            .build();  
}
```

使用更低级的方式进行创建
```java
@Bean  
public List<McpServerFeatures.SyncToolSpecification> weatherTools2() {  
    McpSchema.JsonSchema schema = new McpSchema.JsonSchema("weather", Map.of("city", "string"), List.of("city"), true);  
    McpSchema.Tool tool = new McpSchema.Tool("weatherTool2", "天气查询2", schema);  
  
    McpServerFeatures.SyncToolSpecification specification = new McpServerFeatures.SyncToolSpecification(tool, (McpSyncServerExchange exchange, Map<String, Object> params) -> {  
        return new McpSchema.CallToolResult("", false);  
    });  
    return List.of(specification);  
}
```
##### ToolCallback
通过上面的提供器最终构建的还是 **ToolCallback** 接口类型的实例对象

![|500](images/MCP服务-1744873575006.png)

- MethodToolCallback：普通@Tool标识的方法工具回调函数
- FunctionToolCallback：函数回调工具
- SyncMcpToolCallback/AsyncMcpToolCallback：同步异步工具回调函数

#### 3.5.2 Resource

资源，用于给mcp客户端提供资源的查询能力，例如：api接口，数据库查询。可以理解为spring mvc接口

```java
@Bean  
public List<McpServerFeatures.SyncResourceSpecification> resource() {  
    McpSchema.Annotations annotations = new McpSchema.Annotations(List.of(McpSchema.Role.USER), 0.1);  
    var systemInfoResource = new McpSchema.Resource("/v1/api/query", "queryDatabase", "数据库查询", "", annotations);  
    var resourceSpecification = new McpServerFeatures.SyncResourceSpecification(systemInfoResource, (exchange, request) -> {  
        try {  
            var systemInfo = Map.of("order", "叔叔叔叔");  
            String jsonContent = new ObjectMapper().writeValueAsString(systemInfo);  
            return new McpSchema.ReadResourceResult(  
                    List.of(new McpSchema.TextResourceContents(request.uri(), "application/json", jsonContent)));  
        } catch (Exception e) {  
            throw new RuntimeException("Failed to generate system info", e);  
        }  
    });  
    return List.of(resourceSpecification);  
}
```

#### 3.5.3 Prompt

提示词管理

```java
@Bean  
public List<McpServerFeatures.SyncPromptSpecification> prompts1() {  
    var prompt = new McpSchema.Prompt("greeting", "一个友好的提示词模板",  
            List.of(new McpSchema.PromptArgument("name", "名称", true)));  
  
    var promptSpecification = new McpServerFeatures.SyncPromptSpecification(prompt,  
            (exchange, getPromptRequest) -> {  
                // 获取到里面的参数  
                String nameArgument = (String) getPromptRequest.arguments().get("name");  
                if (nameArgument == null) {  
                    nameArgument = "朋友";  
                }  
                var userMessage = new McpSchema.PromptMessage(McpSchema.Role.USER, new McpSchema.TextContent("你好 " + nameArgument + "! 我可以帮助你什么?"));  
                return new McpSchema.GetPromptResult("个性化问候语", List.of(userMessage));  
            });  
    return List.of(promptSpecification);  
}
```

#### 3.5.4 Roots

根资源管理，目的是用于告诉服务端需要关注的资源范围。当客户端连接到服务器时，会声明应该关注哪些根资源。根资源可以是文件路径，也可以是http url地址

> file:///home/user/projects/myapp
> https://api.example.com/v1

第一步先配置客户端支持Roots根资源
```java
@Component  
public static class CustomMcpSyncClientCustomizer implements McpSyncClientCustomizer {  
    @Override  
    public void customize(String serverConfigurationName, McpClient.SyncSpec spec) {  
  
        McpSchema.ClientCapabilities clientCapabilities = McpSchema.ClientCapabilities.builder()  
        .experimental(Map.of())  
        .roots(true)  
        .sampling()  
        .build();  
        spec.capabilities(clientCapabilities);  
  
        spec.loggingConsumer((McpSchema.LoggingMessageNotification log) -> {  
            System.out.println("消息提醒：" + log.data());  
        });  
    }  
}
```

客户端启动后添加一个根资源
```java
@Bean  
public CommandLineRunner predefinedQuestions(List<McpSyncClient> mcpSyncClients) {  
    return args -> {  
        mcpSyncClients.forEach(mcpSyncClient -> {    
            McpSchema.Root root = new McpSchema.Root("http://127.0.0.1", "端点");  
            mcpSyncClient.addRoot(root);  
        });  
    };  
}
```

服务监听根资源哪些需要监听，如果有资源发生变化则通过日志进行提醒
```java
@Bean  
public BiConsumer<McpSyncServerExchange, List<McpSchema.Root>> rootsChangeHandler() {  
    return (exchange, roots) -> {  
        roots.forEach(root -> {  
            String uri = root.uri();  
            System.out.println("roots资源:" + uri);  
  
            McpSchema.LoggingMessageNotification notification = McpSchema.LoggingMessageNotification.builder()  
                    .data("资源发生了变化")  
                    .level(McpSchema.LoggingLevel.INFO)  
                    .logger("rootsChangeHandler")  
                    .build();  
            exchange.loggingNotification(notification);  
        });  
    };  
}
```

#### 3.5.5 Sampling

客户端提供给服务端的回调函数
```java
@Component  
public static class CustomMcpSyncClientCustomizer implements McpSyncClientCustomizer {  
    @Override  
    public void customize(String serverConfigurationName, McpClient.SyncSpec spec) {  
  
        // Sets a custom sampling handler for processing message creation requests.  
        spec.sampling((McpSchema.CreateMessageRequest messageRequest) -> {  
            // Handle sampling  
            List<McpSchema.SamplingMessage> messages = messageRequest.messages();  
            McpSchema.CreateMessageResult messageResult = null;  
            for (McpSchema.SamplingMessage message : messages) {  
                McpSchema.Content content = message.content();  
                if ("text".equals(content.type())) {  
                    McpSchema.TextContent textContent = (McpSchema.TextContent) content;  
                    System.out.println("收到服务端的回调函数：" + textContent.text());  
  
                    messageResult = McpSchema.CreateMessageResult.builder()  
                            .message("我确认：" + textContent.text())  
                            .build();  
                }  
            }  
            return messageResult;  
        });   
    }  
}
```

服务端调用工具时发起调用sampling工具的请求
```java
/**  
 * Query weather * * @param city city  
 * @return the string  
 * @since 1.0.0  
 */@Tool(description = "天气查询")  
public String queryWeather(String city, ToolContext toolContext) {  
    System.out.println("回调函数确认：" + this.callMcpSampling(toolContext, city));  
    return "天气查询";  
}  
  
/**  
 * Call mcp sampling * * @param toolContext tool context  
 * @param city        city  
 * @return the string  
 * @since 1.0.0  
 */private String callMcpSampling(ToolContext toolContext, String city) {  
    StringBuilder stringBuilder = new StringBuilder();  
    McpToolUtils.getMcpExchange(toolContext)  
            .ifPresent(exchange -> {  
                if (exchange.getClientCapabilities().sampling() != null) {  
                    McpSchema.CreateMessageRequest messageRequestBuilder = McpSchema.CreateMessageRequest.builder()  
                            .messages(List.of(new McpSchema.SamplingMessage(McpSchema.Role.USER,  
                                    new McpSchema.TextContent("你确定要查询:" + city))))  
                            .build();  
                    exchange.createMessage(messageRequestBuilder);  
                    stringBuilder.append(messageRequestBuilder);  
                }  
            });  
    return stringBuilder.toString();  
}
```

#### 3.5.6 监听器
当工具、资源、提示词等资源进行变动时，都可以通过配置对应的消费者来监听数据的变动

```java
@Component  
public static class CustomMcpSyncClientCustomizer implements McpSyncClientCustomizer {  
    @Override  
    public void customize(String serverConfigurationName, McpClient.SyncSpec spec) {  
  
        // Customize the request timeout configuration
        spec.requestTimeout(Duration.ofSeconds(30));  
        McpSchema.ClientCapabilities clientCapabilities = McpSchema.ClientCapabilities.builder()  
                .experimental(Map.of())  
                .roots(true)  
                .sampling()
                .build();  
        spec.capabilities(clientCapabilities);  
  
        // Sets a custom sampling handler for processing message creation requests.  
        spec.sampling((McpSchema.CreateMessageRequest messageRequest) -> {  
            // Handle sampling  
            return null;  
        });  
  
        spec.toolsChangeConsumer((List<McpSchema.Tool> tools) -> {  
        });  
  
        spec.resourcesChangeConsumer((List<McpSchema.Resource> resources) -> {  
        });  
  
        spec.promptsChangeConsumer((List<McpSchema.Prompt> prompts) -> {  
        });
  
  
        // 日志打印
        spec.loggingConsumer((McpSchema.LoggingMessageNotification log) -> {  
            System.out.println("消息提醒：" + log.data());  
        });  
    } 
}
```

## 4. MCP交互协议


## 5. 实战


## 6. 开源MCP市场

- [魔搭社区MCP广场](https://www.modelscope.cn/mcp)
- [MCP Server](https://mcp.so/)
- [MCP客户端和服务端推荐](https://github.com/yzfly/Awesome-MCP-ZH?tab=readme-ov-file)
- [Awesome MCP Servers](https://mcpservers.org/)
- [Cursor MCP](https://cursor.directory/)
- [官方提供的MCP服务](https://github.com/modelcontextprotocol/servers)
