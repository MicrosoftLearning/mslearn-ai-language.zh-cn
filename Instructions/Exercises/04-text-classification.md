---
lab:
  title: 自定义文本分类
  module: Module 3 - Getting Started with Natural Language Processing
---

# 自定义文本分类

Azure AI 语言提供多项 NLP 功能，包括关键短语标识、文本摘要和情绪分析。 语言服务还提供自定义功能，如自定义问题解答和自定义文本分类。

为了测试 Azure AI 语言服务的自定义文本分类，我们将使用语言工作室配置模型，然后使用在 Cloud Shell 中运行的小型命令行应用程序对其进行测试。 此处使用的模式和功能同样适用于实际应用程序。

## 预配 *Azure AI 语言*资源

如果订阅中尚无资源，则需要预配 **Azure AI 语言服务**资源。 此外，请使用自定义文本分类，你需要启用**自定义文本分类和提取**功能。

1. 在浏览器中，打开 Azure 门户 (`https://portal.azure.com`)，并使用 Microsoft 帐户登录。
1. 选择门户顶部的搜索字段，搜索 `Azure AI services`，并创建**语言服务**资源。
1. 选择包含“自定义文本分类”的框。 然后选择“继续创建资源”。
1. 使用以下设置创建资源：
    - **订阅**：Azure 订阅。
    - **资源组**：*选择或创建资源组*。
    - **区域**：*选择任何可用区域*：
    - **名称**：输入唯一名称。
    - **定价层**：选择 **F0**（*免费*层）；如果 F 不可用，请选择 **S**（*标准*层）。
    - 存储帐户：新存储帐户
      - 存储帐户名称：输入唯一的名称。
      - 存储帐户类型：标准 LRS
    - 负责任的 AI 声明：已选中。

1. 选择“查看 + 创建”，再选择“创建”以预配资源。
1. 等待部署完成，然后转到部署的资源。
1. 查看“密钥和终结点”**** 页。 在练习的后面部分，你将需要此页面上的信息。

## 上传示例文章

创建 Azure AI 语言服务和存储帐户后，需上传示例文章供稍后训练模型。

1. 在新的浏览器选项卡中，从 `https://aka.ms/classification-articles` 下载示例文章并将文件提取到所选文件夹。

1. 在 Azure 门户中，导航到你创建的存储帐户，然后选择它。

1. 在存储帐户中，选择位于“设置”**** 下方的“配置”****。 在“配置”屏幕中，启用“允许 Blob 匿名访问”**** 选项，然后选择“保存”****。

1. 从“数据存储”**** 下方的左侧菜单中选择“容器”****。 在出现的屏幕上，选择“+ 容器”。 将容器命名为 `articles`，将公共访问级别设置为“容器(对容器和 Blob 进行匿名读取访问)”。

    > **注意**：为实际解决方案配置存储帐户时，请注意分配适当的访问级别。 若要详细了解每个访问级别，请参阅 [Azure 存储文档](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure)。

1. 创建容器后，选择它，然后选择“上传”按钮。 选择“浏览文件”以浏览下载的示例文章。 然后，选择“上传”。

## 创建自定义文本分类项目

配置完成后，创建自定义文本分类项目。 此项目提供了一个用于生成、训练和部署模型的工作场所。

> **注意**：此实验室利用的是 **Language Studio**，但你也可以通过 REST API 创建、生成、训练和部署模型。

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

1. 在门户顶部的“新建”**** 菜单中，选择“自定义文本分类”****。
1. 此时将显示“连接存储”页。 所有值均已填充。 选择“下一步”。
1. 在“选择项目类型”页上，选择“单一标签分类”。 然后选择**下一步**。
1. 在“输入基本信息”窗格中，设置以下内容：
    - **名称**：`ClassifyLab`  
    - 文本主要语言：英语(美国)
    - **说明**：`Custom text lab`

1. 选择**下一步**。
1. 在“选择容器”**** 页面上，将“Blob 存储容器”**** 下拉列表设置为“文章”** 容器。
1. 选择“否，我需要将我的文件标记为此项目的一部分”**** 选项。 然后选择**下一步**。
1. 选择“创建项目”。

## 标记数据

创建项目后，需要标记数据以训练模型如何对文本进行分类。

1. 在左侧，选择“数据标签”（如果尚未选择）。 你将看到上传到存储帐户的文件列表。
1. 在右侧的“活动”窗格中，选择“+ 添加类”。  本实验室中的文章划分为需要创建的四个类：`Classifieds`、`Sports`、`News`和`Entertainment`。

    ![显示标记数据页面和添加类按钮的屏幕截图。](../media/tag-data-add-class-new.png#lightbox)

1. 创建四个类后，选择“文章 1”开始。 此时可阅读文章、定义此文件属于哪个类，以及将其分配给哪个数据集（训练或测试）。
1. 使用右侧的“活动”**** 窗格为每篇文章分配相应的类和数据集（训练或测试）。  可以从右侧的标签列表中选择标签，并使用“活动”窗格底部的选项将每篇文章设置为**训练**或**测试**。 选择“下一个文档”**** 即可移动到下一个文档。 在本实验室中，我们将定义哪些文章用于训练模型，哪些文章用于测试模型：

    | 项目  | 类  | 数据集  |
    |---------|---------|---------|
    | 文章 1 | 体育游戏 | 培训 |
    | 文章 10 | 新闻 | 培训 |
    | 文章 11 | 娱乐 | 正在测试 |
    | 文章 12 | 新闻 | 正在测试 |
    | 文章 13 | 体育游戏 | 正在测试 |
    | 文章 2 | 体育游戏 | 培训 |
    | 文章 3 | 分类广告 | 培训 |
    | 文章 4 | 分类广告 | 培训 |
    | 文章 5 | 娱乐 | 培训 |
    | 文章 6 | 娱乐 | 培训 |
    | 文章 7 | 新闻 | 培训 |
    | 文章 8 | 新闻 | 培训 |
    | 文章 9 | 娱乐 | 培训 |

    > **注意**：Language Studio 中的文件按字母顺序列出，这也是上述列表不按顺序列出的原因。 在标记文章时，请确保访问这两页文档。

1. 选择“保存标签”以保存标签。

## 训练模型

标记好数据后，需要训练模型。

1. 在左侧菜单中，选择“训练作业”。
1. 选择“开始训练作业”****。
1. 训练名为 `ClassifyArticles` 的新模型。
1. 选择“使用手动拆分训练和测试数据”。

    > **提示**：在你自己的分类项目中，Azure AI 语言服务将自动按百分比拆分测试集，这对大型数据集很有帮助。 对于较小的数据集，请务必使用正确的类分布进行训练。

1. 选择“训练”****

> **重要说明**：训练模型有时可能需要几分钟。 完成后，你将收到通知。

## 评估模型

在文本分类的实际应用中，务必要评估和改进模型，以验证其是否按预期执行。

1. 选择“模型性能”，然后选择 ClassifyArticles 模型。 可在其中查看模型的评分、性能指标以及训练时间。 如果模型的评分不是 100%，则表示用于测试的文档之一未按照其标签进行评估。 这些失败可帮助你了解需要改进的地方。
1. 选择“测试集详细信息”选项卡。如果存在任何错误，可以通过此选项卡查看指定用于测试的文章，模型预测这些文章的内容，以及这些文章是否与其测试标签冲突。 选项卡默认仅显示不正确的预测。 可以切换“仅显示不匹配项”选项，以查看指定用于测试的所有文章，以及每个文章的预测结果。

## 部署模型

对模型的训练感到满意后，就可以部署模型，以便开始通过 API 对文本进行分类。

1. 在左侧面板中，选择“部署模型”。
1. 选择“添加部署”****，然后在“创建新的部署名称”**** 字段中输入`articles`，接着在“模型”**** 字段中选择“ClassifyArticles”****。
1. 选择“部署”以部署模型。
1. 部署模型后，使页面保持打开状态。 下一步需要使用项目和部署名称。

## 准备在 Visual Studio Code 中开发应用

若要测试 Azure AI 语言服务的自定义文本分类功能，你需要在 Visual Studio Code 中开发一个简单的控制台应用程序。

> **提示**：如果已克隆 **mslearn-ai-language** 存储库，请在 Visual Studio Code 中打开它。 否则，请按照以下步骤将其克隆到开发环境。

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行“**Git：克隆**”命令，以将 `https://github.com/MicrosoftLearning/mslearn-ai-language` 存储库克隆到本地文件夹（任意文件夹均可）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择**以后再说**。

## 配置应用程序

已提供适用于 C# 和 Python 的应用程序，以及用于测试汇总情况的示例文本文件。 这两个应用具有相同的功能。 首先，你将完成应用程序的一些关键部分，使其能够使用 Azure AI 语言资源。

1. 在 Visual Studio Code 的“Explorer”**** 窗格中，浏览到 **Labfiles/04-text-classification** 文件夹，并根据你的语言首选项展开 **CSharp** 或 **Python** 文件夹，及其包含的 **classify-text** 文件夹。 每个文件夹都包含要在其中集成 Azure AI 语言文本分类功能的应用的语言特定文件。
1. 右键单击包含代码文件的 **classify-text** 文件夹，然后打开集成终端。 然后通过运行适合你的语言首选项的命令来安装 Azure AI 语言文本分析 SDK 包：

    **C#：**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**：

    ```
    pip install azure-ai-textanalytics==5.3.0
    ```

1. 在“Explorer”**** 窗格中的 **classify-text** 文件夹中，打开首选语言的配置文件

    - **C#** ：appsettings.json
    - **Python**：.env
    
1. 更新配置值，以包含创建的 Azure 语言资源的**终结点**和**密钥**（位于 Azure 门户中 Azure AI 语言资源的“密钥和终结点”**** 页面上）。 该文件应已包含文本分类模型的项目和部署名称。
1. 保存此配置文件。

## 添加代码以对文档进行分类

现在，你已准备好使用 Azure AI 语言服务对文档进行分类。

1. 展开 **“classify-text”** 文件夹中的 **“文章”** 文件夹以查看应用程序将分类的文本文章。
1. 在 **“classify-text”** 文件夹中，打开客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**：classify-text.py

1. 找到注释 **“导入命名空间”**。 然后在此注释下添加以下特定于语言的代码，以导入使用文本分析 SDK 所需的命名空间：

    **C#**：Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**：classify-text.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. 在 **Main** 函数中，请注意，已提供用于加载 Azure AI 语言服务终结点和密钥的代码，以及配置文件中的项目和部署名称。 然后，找到注释“使用终结点和密钥创建客户端”，并添加以下代码为文本分析 API 创建客户端：

    **C#**：Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**：classify-text.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. 在 **Main** 函数中，请注意，现有代码读取项目**文件夹中的所有文件**，并创建包含其内容的列表。 然后找到注释 **“获取分类** ”并添加以下代码：

    **C#** ：Program.cs

    ```csharp
    // Get Classifications
    ClassifyDocumentOperation operation = await aiClient.SingleLabelClassifyAsync(WaitUntil.Completed, batchedDocuments, projectName, deploymentName);

    int fileNo = 0;
    await foreach (ClassifyDocumentResultCollection documentsInPage in operation.Value)
    {
        
        foreach (ClassifyDocumentResult documentResult in documentsInPage)
        {
            Console.WriteLine(files[fileNo].Name);
            if (documentResult.HasError)
            {
                Console.WriteLine($"  Error!");
                Console.WriteLine($"  Document error code: {documentResult.Error.ErrorCode}");
                Console.WriteLine($"  Message: {documentResult.Error.Message}");
                continue;
            }

            Console.WriteLine($"  Predicted the following class:");
            Console.WriteLine();

            foreach (ClassificationCategory classification in documentResult.ClassificationCategories)
            {
                Console.WriteLine($"  Category: {classification.Category}");
                Console.WriteLine($"  Confidence score: {classification.ConfidenceScore}");
                Console.WriteLine();
            }
            fileNo++;
        }
    }
    ```
    
    **Python**：classify-text.py

    ```Python
    # Get Classifications
    operation = ai_client.begin_single_label_classify(
        batchedDocuments,
        project_name=project_name,
        deployment_name=deployment_name
    )

    document_results = operation.result()

    for doc, classification_result in zip(files, document_results):
        if classification_result.kind == "CustomDocumentClassification":
            classification = classification_result.classifications[0]
            print("{} was classified as '{}' with confidence score {}.".format(
                doc, classification.category, classification.confidence_score)
            )
        elif classification_result.is_error is True:
            print("{} has an error with code '{}' and message '{}'".format(
                doc, classification_result.error.code, classification_result.error.message)
            )
    ```

1. 保存对文件所做的更改。

## 测试应用程序

现在已准备好测试应用程序。

1. 在 read-text 文件夹的集成终端中，输入以下命令以运行程序：

    - **C#** ：`dotnet run`
    - **Python**：`python classify-text.py`

    > **提示**：可以使用 **终端工具栏中的“最大化面板大小** ”图标**^** 查看更多控制台文本。

1. 观察输出。 应用程序应列出每个文本文件的分类和置信度分数。


## 清理

如果不再需要项目，可从语言工作室的“项目”页将其删除。 还可以在 [Azure 门户](https://portal.azure.com)中删除 Azure AI 语言服务和关联的存储帐户。
