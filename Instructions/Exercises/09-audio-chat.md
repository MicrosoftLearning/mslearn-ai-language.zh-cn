---
lab:
  title: 开发支持音频的聊天应用
  description: 了解如何使用 Azure AI Foundry 生成支持音频输入的生成式 AI 应用。
---

# 开发支持音频的聊天应用

在本练习中，你将使用 *Phi-4-multimodal-instruct* 生成式 AI 模型生成对包含音频文件的提示的响应。 你将开发一款应用，该应用使用 Azure AI Foundry 和 Azure AI 模型推理服务为杂货店中的新鲜农产品提供 AI 帮助。

此练习大约需要 **30** 分钟。

## 创建 Azure AI Foundry 项目

让我们首先创建 Azure AI Foundry 项目。

1. 在 Web 浏览器中打开 [Azure AI Foundry 门户](https://ai.azure.com)，网址为：`https://ai.azure.com`，然后使用 Azure 凭据登录。 关闭首次登录时打开的任何使用技巧或快速入门窗格，如有必要，请使用左上角的 **Azure AI Foundry** 徽标导航到主页，如下图所示：

    ![Azure AI Foundry 门户的屏幕截图。](../media/ai-foundry-home.png)

1. 在主页中，选择“**+ 创建项目**”。
1. 在**创建项目**向导中，输入项目的有效名，如果出现建议使用现有中心的提示，请选择新建中心的选项。 然后查看将自动创建的 Azure 资源以支持中心和项目。
1. 选择“**自定义**”并为中心指定以下设置：
    - **中心名称**：*中心的有效名称*
    - **订阅**：Azure 订阅
    - **资源组**：*创建或选择资源组*
    - **位置**：选择以下任一区域\*：
        - 美国东部
        - 美国东部 2
        - 美国中北部
        - 美国中南部
        - 瑞典中部
        - 美国西部
        - 美国西部 3
    - **连接 Azure AI 服务或 Azure OpenAI**：*新建 AI 服务资源*
    - **连接 Azure AI 搜索**：跳过连接

    > \*撰写本文时，我们将在本练习中使用的 *Phi-4-multimodal-instruct* 模型在这些区域中可用。 可以在 [Azure AI Foundry 文档](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-serverless-availability#region-availability)中查看特定模型的最新区域可用性。 如果在稍后的练习中达到区域配额限制，则可能需要在其他区域中创建另一个资源。

1. 选择“**下一步**”查看配置。 然后，选择“**创建**”并等待该进程完成。
1. 创建项目后，关闭显示的所有使用技巧，并查看 Azure AI Foundry 门户中的项目页面，如下图所示：

    ![Azure AI Foundry 门户中 Azure AI 项目详细信息的屏幕截图。](../media/ai-foundry-project.png)

## 部署多模式模型

现在，你已准备好部署支持基于音频的输入的多模式模型。 有多个模型可供选择，包括 OpenAI *GPT-4o* 模型。 在本练习中，我们将使用 *Phi-4-multimodal-instruct* 模型。

1. 在 Azure AI Foundry 项目页右上角的工具栏中，使用**预览功能** (**&#9215;**) 图标启用**将模型部署到 Azure AI 模型推理服务**功能。 此功能可确保模型部署可供 Azure AI 推理服务使用，你可在应用程序代码中使用该服务。
1. 在项目左侧窗格的“**我的资产**”部分中，选择“**模型 + 终结点**”页。
1. 在“**模型 + 终结点**”页的“**模型部署**”选项卡中，在“**+ 部署模型**”菜单中，选择“**部署基础模型**”。
1. 在列表中搜索“**Phi-4-multimodal-instruct**”模型，然后选择并确认此模型。
1. 如果出现提示，请同意许可协议，然后在部署详细信息中选择“**自定义**”并使用以下设置部署模型：
    - **部署名**：*有效的模型部署名*
    - **部署类型**：全局标准
    - **部署详细信息**：*使用默认设置*
1. 等待部署预配状态为“**完成**”。

## 创建客户端应用程序

部署模型后，可以在客户端应用程序中使用部署。

> **提示**：可以选择使用 Python 或 Microsoft C# 开发解决方案。 按照所选语言的相应部分中的说明进行操作。

### 准备应用程序配置

1. 在 Azure AI Foundry 门户中，查看项目的“**概述**”页。
1. 在“**项目详细信息**”区域中，记下**项目连接字符串**。 你将使用此连接字符串连接到客户端应用程序中的项目。
1. 打开新的浏览器选项卡（使 Azure AI Foundry 门户在现有选项卡中保持打开状态）。 然后在新选项卡中，浏览到 [Azure 门户](https://portal.azure.com)，网址为：`https://portal.azure.com`；如果出现提示，请使用 Azure 凭据登录。

    关闭任何欢迎通知以查看 Azure 门户主页。

1. 使用页面顶部搜索栏右侧的 **[\>_]** 按钮在 Azure 门户中新建 Cloud Shell，选择订阅中不含存储的 ***PowerShell*** 环境。

    在 Azure 门户底部的窗格中，Cloud Shell 提供命令行接口。 可以调整此窗格的大小或最大化此窗格，以便更易于使用。

    > **备注**：如果以前创建了使用 *Bash* 环境的 Cloud Shell，请将其切换到 ***PowerShell***。

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

    **<font color="red">在继续作之前，请确保已切换到 Cloud Shell 的经典版本。</font>**

1. 在 Cloud Shell 窗格中，输入以下命令以克隆包含此练习代码文件的 GitHub 存储库（键入命令，或将其复制到剪贴板后，在命令行中右键单击并粘贴为纯文本）：


    ```
    rm -r mslearn-ai-audio -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-language mslearn-ai-audio
    ```

    > **提示**：将命令粘贴到 cloudshell 中时，输出可能会占用大量屏幕缓冲区。 可以通过输入 `cls` 命令来清除屏幕，以便更轻松地专注于每项任务。

1. 克隆存储库后，导航到包含应用程序代码文件的文件夹：  

    **Python**

    ```
    cd mslearn-ai-audio/Labfiles/09-audio-chat/python
    ```

    **C#**

    ```
    cd mslearn-ai-audio/Labfiles/09-audio-chat/c-sharp
    ```

1. 在 Cloud Shell 命令行窗格中，输入以下命令安装将使用的库：

    **Python**

    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```

    **C#**

    ```
    dotnet add package Azure.Identity
    dotnet add package Azure.AI.Inference --version 1.0.0-beta.3
    dotnet add package Azure.AI.Projects --version 1.0.0-beta.3
    ```

1. 输入以下命令以编辑已提供的配置文件：

    **Python**

    ```
    code .env
    ```

    **C#**

    ```
    code appsettings.json
    ```

    文件应在代码编辑器中打开。

1. 在代码文件中，将 **your_project_connection_string** 占位符替换为项目的连接字符串（从 Azure AI Foundry 门户中的项目“**概述**”页复制），并将 **your_model_deployment** 占位符替换为分配给 Phi-4-multimodal-instruct 模型部署的名称。

1. 替换占位符后，在代码编辑器中使用“**CTRL+S**”命令或“**右键单击 > 保存**”保存更改，然后使用“**CTRL+Q**”命令或“**右键单击 > 退出**”关闭代码编辑器，同时保持 Cloud Shell 命令行打开。

### 写入代码以连接到项目并获取模型的聊天客户端

> **提示**：添加代码时，请务必保持正确的缩进。

1. 输入以下命令以编辑已提供的代码文件：

    **Python**

    ```
    code audio-chat.py
    ```

    **C#**

    ```
    code Program.cs
    ```

1. 在代码文件中，请注意在文件顶部添加的现有语句，以导入必要的 SDK 命名空间。 然后，查找注释“**添加引用**”，添加以下代码以引用之前安装的库中的命名空间：

    **Python**

    ```python
    # Add references
    from dotenv import load_dotenv
    from azure.identity import DefaultAzureCredential
    from azure.ai.projects import AIProjectClient
    from azure.ai.inference.models import (
        SystemMessage,
        UserMessage,
        TextContentItem,
        AudioContentItem,
        InputAudio,
        AudioContentFormat,
    )
    ```

    **C#**

    ```csharp
    // Add references
    using Azure.Identity;
    using Azure.AI.Projects;
    using Azure.AI.Inference;
    ```

1. 在 **main** 函数的注释“**获取配置设置**”下，请注意，代码将加载配置文件中定义的项目连接字符串和模型部署名称值。

1. 查找注释“**初始化项目客户端**”，添加以下代码，以使用当前登录所使用的 Azure 凭据连接到 Azure AI Foundry 项目：

    **Python**

    ```python
    # Initialize the project client
    project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```csharp
    // Initialize the project client
    var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
    ```

1. 查找注释“**获取聊天客户端**”，添加以下代码以创建与模型聊天的客户端对象：

    **Python**

    ```python
    # Get a chat client
    chat_client = project_client.inference.get_chat_completions_client(model=model_deployment)
    ```

    **C#**

    ```csharp
    // Get a chat client
    ChatCompletionsClient chat = projectClient.GetChatCompletionsClient();
    ```
    

### 写入代码以提交基于 URL 的音频提示

1. 在 **audio-chat.py** 文件的代码编辑器中，在循环部分的注释“**获取对音频输入的响应**”下，添加以下代码，以提交包含以下音频的提示：

    <video controls src="../media/manzanas.mp4" title="对苹果的请求" width="150"></video>

    **Python**

    ```python
    # Get a response to audio input
    file_path = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/manzanas.mp3"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(
                [
                    TextContentItem(text=prompt),
                    {
                        "type": "audio_url",
                        "audio_url": {"url": file_path}
                    }
                ]
            )
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
    // Get a response to audio input
    string audioUrl = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/manzanas.mp3";
    var requestOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage(
                new ChatMessageTextContentItem(prompt),
                new ChatMessageAudioContentItem(new Uri(audioUrl))),
        },
        Model = model_deployment
    };
    var response = chat.Complete(requestOptions);
    Console.WriteLine(response.Value.Content);
    ```

1. 使用 **Ctrl+S** 命令保存对代码文件的更改。 还可以根据需要关闭代码编辑器 (**CTRL+Q**)。

1. 在代码编辑器下方的 Cloud Shell 命令行窗格中，输入以下命令以运行应用：

    **Python**

    ```
    python audio-chat.py
    ```

    **C#**

    ```
    dotnet run
    ```

1. 出现提示时，输入提示 `What is this customer saying in English?`

1. 查看回应。

### 使用不同的提示

1. 在应用代码的代码编辑器中，查找之前在注释“**获取对音频输入的响应**”下添加的代码。 然后，按如下所示修改代码以选择其他音频文件：

    <video controls src="../media/caomei.mp4" title="对草莓的请求" width="150"></video>

    **Python**

    ```python
    # Get a response to audio input
    file_path = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/caomei.mp3"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(
                [
                    TextContentItem(text=prompt),
                    {
                        "type": "audio_url",
                        "audio_url": {"url": file_path}
                    }
                ]
            )
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
    // Get a response to audio input
    string audioUrl = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/caomei.mp3";
    var requestOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage(
                new ChatMessageTextContentItem(prompt),
                new ChatMessageAudioContentItem(new Uri(audioUrl))),
        },
        Model = model_deployment
    };
    var response = chat.Complete(requestOptions);
    Console.WriteLine(response.Value.Content);
    ```

1. 使用 **Ctrl+S** 命令保存对代码文件的更改。 还可以根据需要关闭代码编辑器 (**CTRL+Q**)。

1. 在代码编辑器下方的 Cloud Shell 命令行窗格中，输入以下命令以运行应用：

    **Python**

    ```
    python audio-chat.py
    ```

    **C#**

    ```
    dotnet run
    ```

1. 出现提示时，输入以下提示：

    ```
    A customer left this voice message, can you summarize it?
    ```

1. 查看回应。 然后输入`quit`以退出程序。

    > **备注**：在此简单应用中，我们尚未实现用于保留对话历史记录的逻辑；因此模型会将每个提示视为一个新请求，且没有上一提示的上下文。

1. 可以继续运行应用，选择不同的提示类型并尝试不同的提示。 完成后，输入`quit`退出程序。

    如果有时间，可以修改代码以使用不同的系统提示以及自己的可通过 Internet 访问的音频文件。

    > **备注**：在此简单应用中，我们尚未实现用于保留对话历史记录的逻辑；因此模型会将每个提示视为一个新请求，且没有上一提示的上下文。

## 总结

在本练习中，你使用 Azure AI Foundry 和 Azure AI 推理 SDK 创建客户端应用程序，该应用程序使用多模式模型来生成对音频的响应。

## 清理

如果已完成对 Azure AI Foundry 的探索，则应删除在本练习中创建的资源，以避免产生不必要的 Azure 成本。

1. 返回到包含 Azure 门户的浏览器选项卡（或在新的浏览器选项卡中重新打开 [Azure 门户](https://portal.azure.com)，网址为：`https://portal.azure.com`），查看已在其中部署本练习中使用的资源的资源组内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。
