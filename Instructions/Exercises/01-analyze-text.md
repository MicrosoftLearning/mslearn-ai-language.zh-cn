---
lab:
  title: 分析文本
  module: Module 3 - Develop natural language processing solutions
---

# 分析文本

语言服务是一种支持文本分析的认知服务，包括语言检测、情绪分析、关键短语提取和实体识别。

例如，假设一家旅行社想要处理已提交到公司网站的酒店评论。 通过使用语言服务，他们可以确定每条评论所采用的语言、评论所包含的情绪（积极、中立或消极）、可能指示评论中所讨论主题的关键短语，以及命名实体，例如评论中提及的地点、地标或人。

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
1. 选择“查看 + 创建”。
1. 等待部署完成，然后转到部署的资源。
1. 查看“密钥和终结点”**** 页。 在练习的后面部分，你将需要此页面上的信息。

## 准备在 Visual Studio Code 中开发应用

你将使用 Visual Studio Code 开发文本分析应用。 你的应用的代码文件已在 GitHub 存储库中提供。

> **提示**：如果已克隆 **mslearn-ai-language** 存储库，请在 Visual Studio Code 中打开它。 否则，请按照以下步骤将其克隆到开发环境。

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行“Git: 克隆”命令，以将 `https://github.com/MicrosoftLearning/mslearn-ai-language` 存储库克隆到本地文件夹（任意文件夹均可）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择“以后再说”。

## 配置应用程序

已提供适用于 C# 和 Python 的应用程序，以及用于测试汇总情况的示例文本文件。 这两个应用具有相同的功能。 首先，你将完成应用程序的一些关键部分，使其能够使用 Azure AI 语言资源。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 05-analyze-text 文件夹，并根据你的语言首选项展开 C-Sharp 文件夹或 Python 文件夹   。 每个文件夹都包含要在其中集成 Azure AI 语言文本分析功能的应用的语言特定文件。
2. 右键单击 text-analysis 文件夹，并打开集成终端。 然后通过运行适合你的语言首选项的命令来安装 Azure AI 语言文本分析 SDK 包：

    **C#：**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**：

    ```
    pip install azure-ai-textanalytics==5.3.0
    ```

3. 在 **资源管理器** 窗格中的文本 **分析** 文件夹中，打开首选语言的配置文件

    - **C#** ：appsettings.json
    - **Python**：.env
    
4. 更新配置值以包括**创建的 Azure 语言资源的终结点**和**密钥**（在 Azure 门户 中 Azure AI 语言资源的“密钥和终结点 **”页上可用**）
5. 保存此配置文件。

6. 请注意，text-analysis 文件夹中包含客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**：text-analysis.py

    打开代码文件，并在顶部的现有命名空间引用下找到注释“导入命名空间”。 然后在此注释下添加以下特定于语言的代码，以导入使用文本分析 SDK 所需的命名空间：

    **C#**：Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**：text-analysis.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

7. 请注意，Main 函数中已经提供从配置文件加载认知服务终结点和密钥的代码。 然后，找到注释“使用终结点和密钥创建客户端”，并添加以下代码为文本分析 API 创建客户端：

    **C#**：Programs.cs

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**：text-analysis.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

8. 保存你的更改并返回到 text-analysis 文件夹的集成终端，然后输入以下命令以运行程序：

    - **C#** ：`dotnet run`
    - **Python**：`python text-analysis.py`

    > **提示**：可以使用 **终端工具栏中的“最大化面板大小** ”图标**^** 查看更多控制台文本。

9. 观察输出，因为代码应正常运行且不出现错误，并在“reviews”文件夹中显示每个评论文本文件的内容。 应用程序为文本分析 API 成功创建客户端，但未使用该客户端。 我们将在下一过程中修复此问题。

## 添加代码以检测语言

为文本分析 API 创建客户端后，现在，让我们使用该客户端来检测每条评论所用的语言。

1. 在程序的 Main 函数中，找到注释“获取语言” 。 然后，在此注释下，添加检测每个评论文档中的语言所需的代码：

    **C#**：Programs.cs

    ```csharp
    // Get language
    DetectedLanguage detectedLanguage = aiClient.DetectLanguage(text);
    Console.WriteLine($"\nLanguage: {detectedLanguage.Name}");
    ```

    **Python**：text-analysis.py

    ```python
    # Get language
    detectedLanguage = ai_client.detect_language(documents=[text])[0]
    print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

     > **注意**：此示例对每条评论单独进行了分析，从而导致对每个文件分别调用服务。*另一种方法是创建文档集合，并在单次调用中将它们传递给服务。* 在这两种方法中，服务的响应都包含文档集合；这就是为什么在上面的 Python 代码中，指定了响应 ([0]) 中第一个（也是唯一一个）文档的索引。

1. 保存所做的更改。 返回到 read-text 文件夹的集成终端，输入以下命令以运行程序：
1. 观察输出，注意此次已确定每条评论的语言。

## 添加代码以评估情绪

情绪分析是一种常用技术，可将文本分类为积极情绪或消极情绪（也可能为中立或混合情绪）    。 它通常用于分析社交媒体文章、产品评论和其他项目，其中，文本的情绪可能提供有用的见解。

1. 在程序的 Main 函数中，找到注释“获取情绪” 。 然后，在此注释下，添加检测每个评论文档的情绪所需的代码：

    **C#** ：Program.cs

    ```csharp
    // Get sentiment
    DocumentSentiment sentimentAnalysis = aiClient.AnalyzeSentiment(text);
    Console.WriteLine($"\nSentiment: {sentimentAnalysis.Sentiment}");
    ```

    **Python**：text-analysis.py

    ```python
    # Get sentiment
    sentimentAnalysis = ai_client.analyze_sentiment(documents=[text])[0]
    print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

1. 保存所做的更改。 返回到 read-text 文件夹的集成终端，输入以下命令以运行程序：
1. 观察输出，注意已检测到评论的情绪。

## 添加代码以标识关键短语

识别正文中的关键短语可帮助确定文本所讨论的主题。

1. 在程序的 Main 函数中，找到注释“获取关键短语” 。 然后，在此注释下，添加检测每个评论文档中的关键短语所需的代码：

    **C#** ：Program.cs

    ```csharp
    // Get key phrases
    KeyPhraseCollection phrases = aiClient.ExtractKeyPhrases(text);
    if (phrases.Count > 0)
    {
        Console.WriteLine("\nKey Phrases:");
        foreach(string phrase in phrases)
        {
            Console.WriteLine($"\t{phrase}");
        }
    }
    ```

    **Python**：text-analysis.py

    ```python
    # Get key phrases
    phrases = ai_client.extract_key_phrases(documents=[text])[0].key_phrases
    if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

1. 保存所做的更改。 返回到 read-text 文件夹的集成终端，输入以下命令以运行程序：
1. 观察输出，注意每个文档都包含一些关键短语，你可通过这些短语了解评论的内容。

## 添加实体以提取数据

通常，文档或其他文本正文会提及人物、地点、时期或其他实体。 文本分析 API 可以检测文本中多个类别（和子类别）的实体。

1. 在程序的 Main 函数中，找到注释“获取实体” 。 然后，在此注释下，添加识别每条评论中所提及的实体所需的代码：

    **C#** ：Program.cs

    ```csharp
    // Get entities
    CategorizedEntityCollection entities = aiClient.RecognizeEntities(text);
    if (entities.Count > 0)
    {
        Console.WriteLine("\nEntities:");
        foreach(CategorizedEntity entity in entities)
        {
            Console.WriteLine($"\t{entity.Text} ({entity.Category})");
        }
    }
    ```

    **Python**：text-analysis.py

    ```python
    # Get entities
    entities = ai_client.recognize_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

1. 保存所做的更改。 返回到 read-text 文件夹的集成终端，输入以下命令以运行程序：
1. 观察输出，注意文本中已检测到的实体。

## 添加代码以提取链接实体

除了分类实体外，文本分析 API 还可以检测与数据源（如 Wikipedia）存在已知链接的实体。

1. 在程序的 Main 函数中，找到注释“获取链接实体” 。 然后，在此注释下，添加识别每条评论中所提及的链接实体所需的代码：

    **C#** ：Program.cs

    ```csharp
    // Get linked entities
    LinkedEntityCollection linkedEntities = aiClient.RecognizeLinkedEntities(text);
    if (linkedEntities.Count > 0)
    {
        Console.WriteLine("\nLinks:");
        foreach(LinkedEntity linkedEntity in linkedEntities)
        {
            Console.WriteLine($"\t{linkedEntity.Name} ({linkedEntity.Url})");
        }
    }
    ```

    **Python**：text-analysis.py

    ```python
    # Get linked entities
    entities = ai_client.recognize_linked_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

1. 保存所做的更改。 返回到 read-text 文件夹的集成终端，输入以下命令以运行程序：
1. 观察输出，注意已识别的链接实体。

## 清理资源

提示：如果已完成浏览 Azure Cosmos DB，则可以删除在本练习中创建的资源组。 下面介绍如何操作：

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。

2. 浏览到在此实验室中创建的 Azure AI 语言资源。

3. 在资源页上，选择“删除 **”** 并按照说明删除资源。

## 详细信息

有关详细信息，请参阅 Azure AI 语言文档。
