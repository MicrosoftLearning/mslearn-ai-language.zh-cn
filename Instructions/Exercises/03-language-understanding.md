---
lab:
  title: 使用语言服务创建语言理解模型
  module: Module 5 - Create language understanding solutions
---

# 使用语言服务创建语言理解模型

> **注意**：语言服务的对话语言理解功能目前为预览版，随时可能会发生更改。 在某些情况下，模型训练可能会失败 - 如果发生这种情况，请重试。  

语言服务使你能够定义一个对话语言理解模型，应用程序可使用该模型来解释来自用户的自然语言输入、预测用户的意向（他们想要实现的目标），以及识别应该应用该意向的任何实体  。

例如，时钟应用程序的对话语言模型应会处理如下输入：

伦敦现在几点了？

这种输入是一种语句（用户可能说出或键入的内容）示例，对于此示例，预期意向是获取特定位置（一个实体）的时间；在本例中，此位置为“伦敦”  。

> **注意**：对话语言模型的任务是预测用户的意向，并识别应用该意向的任何实体。 实际执行满足意向所需的操作不是对话语言模型的工作<u></u>。 例如，时钟应用程序可以使用对话语言模型来识别用户想要知道的伦敦时间，但随后客户端应用程序本身必须实现此逻辑才能确定正确的时间并将其呈现给用户。

## 预配 *Azure AI 语言*资源

如果订阅中还没有认知服务资源，则需要预配认知服务资源。

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
1. 使用门户顶部的搜索字段搜索“Azure AI 服务”。 选择“语言服务”资源下的“创建”。
1. 选择“继续创建资源”****。
1. 使用以下设置预配资源：
    - **订阅**：Azure 订阅。
    - **资源组**：*创建或选择资源组*
    - **区域**：选择任何可用区域
    - **名称**：输入唯一名称。
    - **定价层**：选择 **F0**（*免费*层）；如果 F 不可用，请选择 **S**（*标准*层）。
    - 负责任的 AI 声明：同意
1. 选择“查看 + 创建”，再选择“创建”以预配资源。
1. 等待部署完成，然后转到部署的资源。
1. 查看“密钥和终结点”**** 页。 在练习的后面部分，你将需要此页面上的信息。

## 创建对话语言理解项目

现在，你已经创建了创作资源，接下来，可以使用它来创建对话语言理解项目。

1. 在新的浏览器选项卡中，打开 Azure AI Language Studio 门户 (`https://language.cognitive.azure.com/`)，然后使用与 Azure 订阅关联的 Microsoft 帐户登录。

1. 如果系统提示选择语言资源，请选择以下设置：

    - **Azure Directory**：包含订阅的 Azure 目录。
    - **订阅**：Azure 订阅。
    - **资源类型**：语言。
    - **语言资源**：先前创建的 Azure AI 语言资源。

    如果系统未提示你选择语言资源，原因可能是订阅中有多个语言资源；在这种情况下，你应该：

    1. 在页面顶部的栏中，选择“设置(⚙)”**** 按钮。
    2. 在“设置”页上，查看“资源”选项卡。
    3. 选择刚刚创建的语言资源，然后单击“切换资源”。
    4. 在页面顶部，单击“Language Studio”**** 以返回到“Language Studio”主页。

1. 在门户顶部的“新建”菜单中，选择“对话语言理解”。

1. 在“创建项目”对话框中，在“输入基本信息”页上输入以下详细信息，然后单击“下一步”  ：
    - **名称**：`Clock`
    - 言语主要语言：英语
    - **在项目中启用多种语言？** ：未选定
    - **说明**：`Natural language clock`

1. 在“查看并完成”页上，选择“创建标签” 。

### 创建意向

在新项目中，首先要做的是定义一些意向。 该模型最终将预测用户在提交自然语言话语时请求的这些意向中的哪一个。

> **提示**：在处理项目时，如果系统显示了一些提示，请阅读这些提示并单击“了解”将其关闭，也可单击“全部跳过” 。

1. 在“架构定义”页的“意向”选项卡中，选择“&#65291; 添加”以添加名为 GetTime 的新意向   。
1. **验证是否已列出 GetTime** 意向（以及默认**的 None** 意向）。 然后添加以下附加意向：
    - `GetDay`
    - `GetDate`

### 使用示例话语标记每个意向

为了帮助模型预测用户请求的意向，必须使用一些示例话语来标记每个意向。

1. 在左侧窗格中，选择 **“数据标记** ”页。

> **提示**：可以使用图标展开窗格>>**** 以查看页面名称，并使用图标再次**<<** 隐藏它。

1. 选择新的 **GetTime** 意向并输入话语 `what is the time?`。 这会添加话语作为意向的示例输入。
1. 为 GetTime** 意向添加以下附加陈述**：
    - `what's the time?`
    - `what time is it?`
    - `tell me the time`

1. 在 GetTime 意图中，添加以下言语作为示例用户输入：
    - `what day is it?`
    - `what's the day?`
    - `what is the day today?`
    - `what day of the week is it?`

1. 选择 **GetDate** 意向并为其添加以下话语：
    - `what date is it?`
    - `what's the date?`
    - `what is the date today?`
    - `what's today's date?`

1. 为每个意向添加话语后，选择“ **保存更改**”。

### 训练和测试模型

现在你已经添加了一些意向，接下来训练语言模型，看看它是否能够通过用户输入正确预测这些意向。

1. 在左侧，选择“训练作业”****。 单击“+ 启动训练作业”****。

1. 在“启动训练作业”**** 对话框中，选择训练新模型这一选项，将其命名为 `Clock`。 选择 **标准训练** 模式和默认 **的数据拆分** 选项。

1. 要开始训练模型的过程，请单击“训练”。

1. 训练完成后（这可能需要几分钟），作业“状态”将变为“训练成功” 。

1. 选择“模型性能”页，然后选择“时钟”模型 。 查看总体和每个意向的评估指标（精准率、召回率和 F1 分数），以及由在训练时执行的评估生成的混淆矩阵（请注意，由于样本语句数量较少，并非所有意向都可能包含在结果中）   。

    > **注意**：若要详细了解评估指标，请参阅[文档](/azure/ai-services/language-service/conversational-language-understanding/concepts/evaluation-metrics)

1. 在“部署模型”页上，选择“添加部署” 。

1. 在“添加部署”对话框中，选择“新建部署名称”，然后输入“生产”

1. 在 **“模型 **”字段中选择时钟**模型**，然后选择“**部署**”。 此部署可能需要花一些时间。

1. 部署模型后，在“测试部署”页上，选择“生产”部署 。

1. 输入以下文本，然后选择“运行测试”：

    `what's the time now?`

    查看返回的结果，请注意，该结果包含预测意向（应为 GetTime）以及一个置信度分数，该分数表示模型计算出预测意向的概率。 “JSON”选项卡显示每个潜在意向的对比置信度（置信度分数最高的潜在意向就是预测意向）

1. 清除文本框，然后使用以下文本运行另一个测试：

    `tell me the time`

    再次查看预测意图和置信度分数。

1. 请尝试以下内容：

    `what's the day today?`

    模型有望预测 GetDay 意图。

## 添加实体

到目前为止，你已定义了一些与意图有关的简单言语。 大多数实际应用程序都包含更复杂的言语，必须从这些言语中提取特定的数据实体才能获取有关意图的更多上下文。

### 添加学习实体

最常见的一种实体是学习实体，在此实体中，模型根据示例学习识别实体值。

1. 在“语言工作室”中，返回到“架构定义”页，然后在“实体”选项卡中选择“&#65291; 添加”以添加新实体  。

1. 在“添加实体”对话框中，输入实体名称“位置”并确保选中“学习”  。 然后选择法人。

1. **创建位置**实体后，返回到 **“数据标记**”页。
1. **选择 GetTime** 意向并输入以下新示例话语：

    `what time is it in London?`

1. 添加该语句后，选择“伦敦”一词，然后在出现的下拉列表中，选择“位置”以指示“伦敦”即为一个位置示例。

1. 为 **GetTime** 意向添加另一个示例话语：

    `Tell me the time in Paris?`

1. 添加该语句后，选择“巴黎”一词，并将其映射到“位置”实体。

1. 为 **GetTime** 意向添加另一个示例话语：

    `what's the time in New York?`

1. 添加该语句后，选择“纽约”一词，并将其映射到“位置”实体。

1. 单击“保存更改”以保存新的语句。

### 添加列表实体

在某些情况下，实体的有效值可被限制为一系列特定术语和同义词；这可帮助应用识别言语中的实体实例。

1. 在“语言工作室”中，返回到“架构定义”页，然后在“实体”选项卡中选择“&#65291; 添加”以添加新实体  。

1. 在“添加实体”对话框中，输入实体名称“工作日”并选择“列表”实体类型  。

1. 在“工作日”实体的页面上 **，在 **“学习**”部分中，确保 **“不需要**”处于选中状态。** 然后，在 **“列表** ”部分中，选择 **&#65291;添加新列表**。 然后输入以下值和同义词，并单击“保存”：

    | 列表键 | 同义词|
    |-------------------|---------|
    | `Sunday` | `Sun` |

1. 重复上一步以添加以下列表组件：

    | 值 | 同义词|
    |-------------------|---------|
    | `Monday` | `Mon` |
    | `Tuesday` | `Tue, Tues` |
    | `Wednesday` | `Wed, Weds` |
    | `Thursday` | `Thur, Thurs` |
    | `Friday` | `Fri` |
    | `Saturday` | `Sat` |

1. 添加和保存列表值后，返回到 **“数据标记** ”页。
1. **选择 GetDate** 意向并输入以下新示例话语：

    `what date was it on Saturday?`

1. 添加该语句后，选择“星期六”一词，然后在出现的下拉列表中选择“工作日”。

1. 为 **GetDate** 意向添加另一个示例话语：

    `what date will it be on Friday?`

1. 添加语句后，将“星期五”映射到“工作日”实体 。

1. 为 **GetDate** 意向添加另一个示例话语：

    `what will the date be on Thurs?`

1. 添加语句后，将“星期四”映射到“工作日”实体 。

1. 单击“保存更改”以保存新的语句。

### 添加预生成实体

语言服务提供了一组通常在对话应用程序中使用的预生成实体。

1. 在“语言工作室”中，返回到“架构定义”页，然后在“实体”选项卡中选择“&#65291; 添加”以添加新实体  。

1. 在“添加实体”对话框中，输入实体名称“日期”并选择“预生成”实体类型  。

1. 在“日期”实体的页面上 **，在 **“已学习**”部分中，确保**未选中“不需要**”。** 然后，在 **预生成** 部分中，选择 **&#65291;添加新的预生成**。

1. 在“选择预生成”列表中，选择“DateTime”，然后单击“保存”  。
1. 添加预生成实体后，返回到“数据标记”**** 页
1. **选择 GetDay** 意向并输入以下新示例话语：

    `what day was 01/01/1901?`

1. 添加该语句后，选择“1901 年 1 月 1 日”，然后在出现的下拉列表中选择“日期”。

1. 为 **GetDay** 意向添加另一个示例话语：

    `what day will it be on Dec 31st 2099?`

1. 添加语句后，将“2099 年 12 月 31 日”映射到“日期”实体 。

1. 单击“保存更改”以保存新的语句。

### 重新训练模型

现在，你已经修改了架构，接下来，需要重新训练并重新测试模型。

1. 在“训练作业”页上，选择“开始训练作业” 。

1. 在“开始训练作业”对话框中，选择用于覆盖现有模型的选项并指定“时钟”模型 。 若要训练模型，请选择“训练”。 如果出现提示，请确认要覆盖现有模型。

1. 训练完成后，作业“状态”将更新为“训练已成功” 。

1. 选择“模型性能”页，然后选择“时钟”模型 。 查看总体和每个意向的评估指标（精准率、召回率和 F1 分数），以及由在训练时执行的评估生成的混淆矩阵（请注意，由于样本语句数量较少，并非所有意向都可能包含在结果中）   。

1. 在“部署模型”页上，选择“添加部署” 。

1. 在“添加部署”对话框中，选择“覆盖现有部署名称”，然后选择“生产”  。

1. 在 **“模型 **”字段中选择时钟**模型**，然后选择“**部署**”以部署它。 这可能需要一些时间。

1. 部署模型后，在“测试部署”页上，选择“生产”部署，然后使用以下文本对其进行测试 ：

    `what's the time in Edinburgh?`

1. 查看返回的结果，该结果应会预测“GetTime”意向和具有文本值“爱丁堡”的“位置”实体 。

1. 尝试测试以下言语：

    `what time is it in Tokyo?`

    `what date is it on Friday?`

    `what's the date on Weds?`

    `what day was 01/01/2020?`

    `what day will Mar 7th 2030 be?`

## 使用客户端应用中的模型

在实际项目中，你需要反复优化意图和实体、重新训练并重新测试，直到对预测性能感到满意为止。 如果对该模型进行了测试并对其预测性能感到满意，随后可以通过调用其 REST 接口在客户端应用中使用它。

### 准备在 Visual Studio Code 中开发应用

你将使用 Visual Studio Code 开发语言理解应用。 你的应用的代码文件已在 GitHub 存储库中提供。

> **提示**：如果已克隆 **mslearn-ai-language** 存储库，请在 Visual Studio Code 中打开它。 否则，请按照以下步骤将其克隆到开发环境。

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行“**Git：克隆**”命令，以将 `https://github.com/MicrosoftLearning/mslearn-ai-language` 存储库克隆到本地文件夹（任意文件夹均可）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。

    > **注意**：如果 Visual Studio Code 显示一条弹出消息，提示你信任打开的代码，请单击弹出窗口中的“是，我信任该作者”选项****。

4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择**以后再说**。

### 配置应用程序

已提供适用于 C# 和 Python 的应用程序，以及用于测试汇总情况的示例文本文件。 这两个应用具有相同的功能。 首先，你将完成应用程序的一些关键部分，使其能够使用 Azure AI 语言资源。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 10-luis-client 文件夹，并根据你的语言首选项展开 C-Sharp 文件夹或 Python 文件夹   。 每个文件夹都包含要在其中集成 Azure AI 语言问答功能的应用的语言特定文件。
2. 右键单击 clock-client 文件夹，并打开集成终端。 然后通过运行适用于你的语言首选项的命令，安装语言理解 SDK 包：

    **C#：**

    ```
    dotnet add package Azure.AI.Language.Conversations --version 1.1.0
    ```

    **Python**：

    ```
    pip install azure-ai-language-conversations
    ```

3. 在 **资源管理器** 窗格中，在 **时钟客户端** 文件夹中，打开首选语言的配置文件

    - **C#** ：appsettings.json
    - **Python**：.env
    
4. 更新配置值，以包含创建的 Azure 语言资源的**终结点**和**密钥**（位于 Azure 门户中 Azure AI 语言资源的“密钥和终结点”**** 页面上）。
5. 保存此配置文件。

### 将代码添加到应用程序中

现在，你已准备好添加导入所需 SDK 库所需的代码，建立与已部署项目经过身份验证的连接，并提交问题。

1. 请注意，clock-client 文件夹中包含客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**：clock-client.py

    打开代码文件，并在顶部的现有命名空间引用下找到注释“导入命名空间”。 然后在此注释下添加以下特定于语言的代码，以导入使用文本分析 SDK 所需的命名空间：

    **C#**：Programs.cs

    ```c#
    // import namespaces
    using Azure;
    using Azure.AI.Language.Conversations;
    ```

    **Python**：clock-client.py

    ```python
    # Import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.conversations import ConversationAnalysisClient
    ```

1. 请注意，Main 函数中已经提供从配置文件加载预测终结点和密钥的代码。 下一步是找到注释“为语言服务模型创建客户端”，并添加以下代码来为语言服务应用创建预测客户端：

    **C#**：Programs.cs

    ```c#
    // Create a client for the Language service model
    Uri endpoint = new Uri(predictionEndpoint);
    AzureKeyCredential credential = new AzureKeyCredential(predictionKey);

    ConversationAnalysisClient client = new ConversationAnalysisClient(endpoint, credential);
    ```

    **Python**：clock-client.py

    ```python
    # Create a client for the Language service model
    client = ConversationAnalysisClient(
        ls_prediction_endpoint, AzureKeyCredential(ls_prediction_key))
    ```

1. 请注意，Main 函数中的代码会提示用户进行输入，直到用户输入“quit”。 在此循环中找到注释“调用语言服务模型以获取意向和实体”并添加以下代码：

    **C#**：Programs.cs

    ```c#
    // Call the Language service model to get intent and entities
    var projectName = "Clock";
    var deploymentName = "production";
    var data = new
    {
        analysisInput = new
        {
            conversationItem = new
            {
                text = userText,
                id = "1",
                participantId = "1",
            }
        },
        parameters = new
        {
            projectName,
            deploymentName,
            // Use Utf16CodeUnit for strings in .NET.
            stringIndexType = "Utf16CodeUnit",
        },
        kind = "Conversation",
    };
    // Send request
    Response response = await client.AnalyzeConversationAsync(RequestContent.Create(data));
    dynamic conversationalTaskResult = response.Content.ToDynamicFromJson(JsonPropertyNames.CamelCase);
    dynamic conversationPrediction = conversationalTaskResult.Result.Prediction;   
    var options = new JsonSerializerOptions { WriteIndented = true };
    Console.WriteLine(JsonSerializer.Serialize(conversationalTaskResult, options));
    Console.WriteLine("--------------------\n");
    Console.WriteLine(userText);
    var topIntent = "";
    if (conversationPrediction.Intents[0].ConfidenceScore > 0.5)
    {
        topIntent = conversationPrediction.TopIntent;
    }
    ```

    **Python**：clock-client.py

    ```python
    # Call the Language service model to get intent and entities
    cls_project = 'Clock'
    deployment_slot = 'production'

    with client:
        query = userText
        result = client.analyze_conversation(
            task={
                "kind": "Conversation",
                "analysisInput": {
                    "conversationItem": {
                        "participantId": "1",
                        "id": "1",
                        "modality": "text",
                        "language": "en",
                        "text": query
                    },
                    "isLoggingEnabled": False
                },
                "parameters": {
                    "projectName": cls_project,
                    "deploymentName": deployment_slot,
                    "verbose": True
                }
            }
        )

    top_intent = result["result"]["prediction"]["topIntent"]
    entities = result["result"]["prediction"]["entities"]

    print("view top intent:")
    print("\ttop intent: {}".format(result["result"]["prediction"]["topIntent"]))
    print("\tcategory: {}".format(result["result"]["prediction"]["intents"][0]["category"]))
    print("\tconfidence score: {}\n".format(result["result"]["prediction"]["intents"][0]["confidenceScore"]))

    print("view entities:")
    for entity in entities:
        print("\tcategory: {}".format(entity["category"]))
        print("\ttext: {}".format(entity["text"]))
        print("\tconfidence score: {}".format(entity["confidenceScore"]))

    print("query: {}".format(result["result"]["query"]))
    ```

    对语言服务模型的调用会返回一个预测/结果，其中包括排名第一的（最有可能的）意向以及在输入语句中检测到的所有实体。 你的客户端应用程序现在必须使用该预测来确定并执行适当的操作。

1. 找到注释“应用适当的操作”，并添加以下代码，该代码检查是否有应用程序支持的意向（GetTime、GetDate 和 GetDay）并确定是否检测到相关实体，然后调用一个现有函数来生成适当的响应   。

    **C#**：Programs.cs

    ```c#
    // Apply the appropriate action
    switch (topIntent)
    {
        case "GetTime":
            var location = "local";           
            // Check for a location entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Location")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    location = entity.Text;
                }
            }
            // Get the time for the specified location
            string timeResponse = GetTime(location);
            Console.WriteLine(timeResponse);
            break;
        case "GetDay":
            var date = DateTime.Today.ToShortDateString();            
            // Check for a Date entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Date")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    date = entity.Text;
                }
            }            
            // Get the day for the specified date
            string dayResponse = GetDay(date);
            Console.WriteLine(dayResponse);
            break;
        case "GetDate":
            var day = DateTime.Today.DayOfWeek.ToString();
            // Check for entities            
            // Check for a Weekday entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Weekday")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    day = entity.Text;
                }
            }          
            // Get the date for the specified day
            string dateResponse = GetDate(day);
            Console.WriteLine(dateResponse);
            break;
        default:
            // Some other intent (for example, "None") was predicted
            Console.WriteLine("Try asking me for the time, the day, or the date.");
            break;
    }
    ```

    **Python**：clock-client.py

    ```python
    # Apply the appropriate action
    if top_intent == 'GetTime':
        location = 'local'
        # Check for entities
        if len(entities) > 0:
            # Check for a location entity
            for entity in entities:
                if 'Location' == entity["category"]:
                    # ML entities are strings, get the first one
                    location = entity["text"]
        # Get the time for the specified location
        print(GetTime(location))

    elif top_intent == 'GetDay':
        date_string = date.today().strftime("%m/%d/%Y")
        # Check for entities
        if len(entities) > 0:
            # Check for a Date entity
            for entity in entities:
                if 'Date' == entity["category"]:
                    # Regex entities are strings, get the first one
                    date_string = entity["text"]
        # Get the day for the specified date
        print(GetDay(date_string))

    elif top_intent == 'GetDate':
        day = 'today'
        # Check for entities
        if len(entities) > 0:
            # Check for a Weekday entity
            for entity in entities:
                if 'Weekday' == entity["category"]:
                # List entities are lists
                    day = entity["text"]
        # Get the date for the specified day
        print(GetDate(day))

    else:
        # Some other intent (for example, "None") was predicted
        print('Try asking me for the time, the day, or the date.')
    ```

1. 保存你的更改并返回到 clock-client 文件夹的集成终端，然后输入以下命令以运行程序：

    - **C#** ：`dotnet run`
    - **Python**：`python clock-client.py`

    > **提示**：可以使用 **终端工具栏中的“最大化面板大小** ”图标**^** 查看更多控制台文本。

1. 系统提示时，输入言语来测试应用程序。 例如，尝试以下操作：

    *Hello*

    *几点了？*

    伦敦现在几点了？

    今天是几月几号？

    星期天是几号？

    今天星期几？

    2025 年 1 月 1 日是星期几？

    > **注意**：该应用程序中的逻辑设计得很简单，并且有许多限制。 例如在获取时间时支持的城市是有限的，并且没有考虑夏令时。 我们的目标是通过一个示例了解使用语言服务的典型模式，在该模式中，你的应用程序必须：
    >   1. 连接到预测终结点。
    >   2. 通过提交一个言语来获取预测结果。
    >   3. 实现逻辑，针对预测的意图和实体作出适当的响应。

1. 完成测试后，请输入“quit”。

## 清理资源

提示：如果已完成浏览 Azure Cosmos DB，则可以删除在本练习中创建的资源组。 操作步骤如下：

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 浏览到在此实验室中创建的 Azure AI 语言资源。
3. 在资源页上，选择“删除 **”** 并按照说明删除资源。

## 详细信息

要了解有关语言理解的详细信息，请参阅 [Azure AI 语言文档](https://learn.microsoft.com/azure/ai-services/language-service/conversational-language-understanding/overview)。
