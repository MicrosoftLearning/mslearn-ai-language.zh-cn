---
lab:
  title: 提取自定义实体
  module: Module 3 - Getting Started with Natural Language Processing
---

# 提取自定义实体

除了其他自然语言处理功能外，Azure AI 语言服务还支持从各种文件中提取自定义实体。

若要测试自定义实体的提取，我们将创建模型并通过 Azure AI Language Studio 对其进行训练，然后使用命令行应用程序对其进行测试。

## 预配 *Azure AI 语言*资源

如果订阅中尚无资源，则需要预配 **Azure AI 语言服务**资源。 此外，请使用自定义文本分类，你需要启用**自定义文本分类和提取**功能。

1. 在浏览器中，打开 Azure 门户 (`https://portal.azure.com`)，并使用 Microsoft 帐户登录。
1. 选择“创建资源”按钮，搜索“语言”，并创建“语言服务”资源**********。 在“选择其他功能”页面上**，选择包含**自定义命名实体识别提取**的自定义功能。 使用以下设置创建资源：
    - **订阅**：*Azure 订阅*
    - **资源组**：选择或创建资源组
    - **区域**：*选择任何可用区域*
    - **名称**：*输入唯一名称*
    - **定价层**：选择 **F0**（*免费*层）；如果 F 不可用，请选择 **S**（*标准*层）。
    - 存储帐户：新存储帐户
      - 存储帐户名称：输入唯一的名称。
      - 存储帐户类型：标准 LRS
    - 负责任的 AI 声明：已选中。

1. 选择“查看 + 创建”，再选择“创建”以预配资源。
1. 等待部署完成，然后转到部署的资源。
1. 查看“密钥和终结点”**** 页。 在练习的后面部分，你将需要此页面上的信息。

## 上传示例广告

创建 Azure AI 语言服务和存储帐户后，需上传示例广告供稍后训练模型。

1. 在新的浏览器选项卡中，从中下载分类的示例广告 `https://aka.ms/entity-extraction-ads` ，并将文件提取到所选的文件夹。

2. 在 Azure 门户中，导航到你创建的存储帐户，然后选择它。

3. 在存储帐户中选择“配置”，位于设置**下方**，屏幕启用“**允许 Blob 匿名访问**”选项，然后选择“**保存**”。****

4. 在存储帐户中，从位于“数据存储”下方的左侧菜单中选择“容器”。 在出现的屏幕上，选择“+ 容器”。 将容器命名为 `classifieds`，将公共访问级别设置为“容器(对容器和 Blob 进行匿名读取访问)”。

    > **注意**：为实际解决方案配置存储帐户时，请注意分配适当的访问级别。 若要详细了解每个访问级别，请参阅 [Azure 存储文档](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure)。

5. 创建容器后，选择它并单击“上传”按钮以上传下载的示例广告。

## 创建自定义命名实体识别项目

现在，你已准备好创建自定义命名实体识别项目。 此项目提供了一个用于生成、训练和部署模型的工作场所。

> 还可以通过 REST API 创建、生成、训练和部署模型

1. 在新的浏览器选项卡中，打开 Azure AI Language Studio 门户 (`https://language.cognitive.azure.com/`)，然后使用与 Azure 订阅关联的 Microsoft 帐户登录。
1. 如果系统提示选择语言资源，请选择以下设置：

    - **Azure Directory**：包含订阅的 Azure 目录。
    - **订阅**：Azure 订阅。
    - **资源类型**：语言。
    - **语言资源**：先前创建的 Azure AI 语言资源。

    如果系统未提示你选择语言资源，原因可能是订阅中有多个语言资源；在这种情况下，你应该：

    1. 在页面顶部的栏上，选择“设置 (&#9881;)”**** 按钮。
    2. 在“设置”页上，查看“资源”选项卡。
    3. 选择刚刚创建的语言资源，然后单击“切换资源”。
    4. 在页面顶部，单击“Language Studio”以返回到 Language Studio 主页。

1. 在门户顶部的“新建****”菜单中，选择“自定义命名实体识别****”。

1. 创建一个具有以下设置的新项目：
    - 连接存储：此值可能已填充。如果资源尚未更改，请将资源更改为 customner
    - 基本信息：
    - **名称**：`CustomEntityLab`
        - 文本主要语言：英语(美国)
        - 数据集是否包含不使用相同语言的文档? 否：
        - **说明**：`Custom entities in classified ads`
    - **容器**：
        - **Blob 存储容器**：分类
        - **你的文件是否标有类？**：不，我需要将我的文件标记为该项目的一部分

> **提示**：如果收到有关无权执行此操作的错误，则需要添加角色分配。 为修复此错误，我们在运行实验室的用户的存储账户上添加了角色“存储 Blob 数据参与者”。 可在[文档页上](https://learn.microsoft.com/azure/ai-services/language-service/custom-named-entity-recognition/how-to/create-project?tabs=portal%2Clanguage-studio#enable-identity-management-for-your-resource)找到更多详细信息。

## 标记数据

创建项目后，需要标记数据以训练模型如何识别实体。

1. **如果“数据标签**”页尚未打开，请在左侧窗格中选择“**数据标签**”。 你将看到上传到存储帐户的文件列表。
1. 在右侧的“活动”窗格中，选择“+ 添加实体”。
1.  重复上一步骤 o 创建以下实体：
    - `Price`
    - `Location`
1. 创建三个实体后，请选择 **Ad 1.txt** ，以便你可以阅读它。
1. 在 *Ad 1.txt* 中： 
    1. 突出显示柴火*的文本*面线，然后选择 **ItemForSale** 实体。
    1. 突出显示丹佛、CO* 文本*并选择 **“位置”** 实体。
    1. 突出显示文本 *$90* 并选择 **Price** 实体。
**1.In“活动**”窗格，请注意，本文档将添加到数据集以训练模型。
1. 使用“下一个文档”**** 按钮移动到下一个文档，并继续将文本分配给整个文档集的相应实体，并将它们全部添加到训练数据集。
1. 标记最后一个文档（*Ad 9.txt*）后，保存标签。

## 训练模型

标记好数据后，需要训练模型。

1. 在左侧，选择“训练作业”****。
2. 单击“+ 启动训练作业”****。
3. 训练名为 `ExtractAds` 的新模型。
4. 选择“从训练数据中自动拆分测试集”

    > 在自己的提取项目中，使用最适合数据的测试拆分。 对于更一致的数据和更大的数据集，Azure AI 语言服务将自动按百分比拆分测试集。 对于更小的数据集，请务必使用各种可能的输入文档进行训练。

5. 单击“训练”

    > 训练模型有时可能需要几分钟。 完成后，你将收到通知。

## 评估模型

在实际应用中，务必要评估和改进模型，以验证其是否按预期执行。 左侧的两页显示已训练模型的详细信息，以及任何失败的测试。

在左侧菜单中选择“模型性能”，然后选择 `ExtractAds` 模型。 可在其中查看模型的评分、性能指标以及训练时间。 你将能够查看是否有任何测试文档失败，这些失败有助于了解改进之处。

## 部署模型

对模型的训练感到满意后，就可以部署模型，以便开始通过 API 提取实体。

1. 在左侧面板中，选择“部署模型”。
2. 选择“添加部署”，输入名称 `AdEntities`，然后从模型下拉列表中选择 
3. 单击“部署”以部署模型

## 准备在 Visual Studio Code 中开发应用

若要测试 Azure AI 语言服务的自定义实体提取功能，需要在 Visual Studio Code 中开发一个简单的控制台应用程序。

> **提示**：如果已克隆 **mslearn-ai-language** 存储库，请在 Visual Studio Code 中打开它。 否则，请按照以下步骤将其克隆到开发环境。

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行“**Git：克隆**”命令，以将 `https://github.com/MicrosoftLearning/mslearn-ai-language` 存储库克隆到本地文件夹（任意文件夹均可）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。

    > **注意**：如果 Visual Studio Code 显示一条弹出消息，提示你信任打开的代码，请单击弹出窗口中的“是，我信任该作者”选项****。

4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择**以后再说**。

## 配置应用程序

已提供适用于 C# 和 Python 的应用程序。 这两个应用具有相同的功能。 首先，你将完成应用程序的一些关键部分，使其能够使用 Azure AI 语言资源。

1. 在 Visual Studio Code 的**资源管理器**窗格中，浏览到 **Labfiles/05-custom-entity-recognition** 文件夹，并根据语言首选项和**它所包含的自定义实体**文件夹展开 **CSharp** 或 **Python** 文件夹。 每个文件夹都包含要在其中集成 Azure AI 语言文本分类功能的应用的语言特定文件。
1. 右键单击 04-containers 文件夹，并打开集成终端。 然后通过运行适合你的语言首选项的命令来安装 Azure AI 语言文本分析 SDK 包：

    **C#：**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**：

    ```
    pip install azure-ai-textanalytics==5.3.0
    ```

1. 在 **“资源管理器”** 窗格中的 **“自定义实体** ”文件夹中，打开首选语言的配置文件

    - **C#** ：appsettings.json
    - **Python**：.env
    
1. 更新配置值，以包含创建的 Azure 语言资源的**终结点**和**密钥**（位于 Azure 门户中 Azure AI 语言资源的“密钥和终结点”**** 页面上）。 该文件应已包含自定义实体提取模型的项目和部署名称。
1. 保存此配置文件。

## 添加实体以提取数据

现在，你已准备好使用 Azure AI 语言服务从文本中提取自定义实体。

1. 展开**自定义实体**文件夹中的广告**文件夹**以查看应用程序将分析的分类广告。
1. 在 **自定义实体** 文件夹中，打开客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**：custom-entities.py

1. 找到注释 **“导入命名空间”**。 然后在此注释下添加以下特定于语言的代码，以导入使用文本分析 SDK 所需的命名空间：

    **C#**：Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**：custom-entities.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. 在 **Main** 函数中，请注意，已提供用于加载 Azure AI 语言服务终结点和密钥的代码，以及配置文件中的项目和部署名称。 然后，找到注释“使用终结点和密钥创建客户端”，并添加以下代码为文本分析 API 创建客户端：

    **C#**：Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credentials = new(aiSvcKey);
    Uri endpoint = new(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new(endpoint, credentials);
    ```

    **Python**：custom-entities.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. 在 **Main** 函数中，注意现有代码会读取“广告****”文件夹中的所有文件，并创建包含其内容的列表。 对于 C# 代码，将使用 **TextDocumentInput** 对象的列表以 ID 和语言的形式包含文件名。 在 Python 中，使用文本内容的简单列表。
1. 查找注释 **提取实体** 并添加以下代码：

    **C#** ：Program.cs

    ```csharp
    // Extract entities
    RecognizeCustomEntitiesOperation operation = await aiClient.RecognizeCustomEntitiesAsync(WaitUntil.Completed, batchedDocuments, projectName, deploymentName);

    await foreach (RecognizeCustomEntitiesResultCollection documentsInPage in operation.Value)
    {
        foreach (RecognizeEntitiesResult documentResult in documentsInPage)
        {
            Console.WriteLine($"Result for \"{documentResult.Id}\":");

            if (documentResult.HasError)
            {
                Console.WriteLine($"  Error!");
                Console.WriteLine($"  Document error code: {documentResult.Error.ErrorCode}");
                Console.WriteLine($"  Message: {documentResult.Error.Message}");
                Console.WriteLine();
                continue;
            }

            Console.WriteLine($"  Recognized {documentResult.Entities.Count} entities:");

            foreach (CategorizedEntity entity in documentResult.Entities)
            {
                Console.WriteLine($"  Entity: {entity.Text}");
                Console.WriteLine($"  Category: {entity.Category}");
                Console.WriteLine($"  Offset: {entity.Offset}");
                Console.WriteLine($"  Length: {entity.Length}");
                Console.WriteLine($"  ConfidenceScore: {entity.ConfidenceScore}");
                Console.WriteLine($"  SubCategory: {entity.SubCategory}");
                Console.WriteLine();
            }

            Console.WriteLine();
        }
    }
    ```

    **Python**：custom-entities.py

    ```Python
    # Extract entities
    operation = ai_client.begin_recognize_custom_entities(
        batchedDocuments,
        project_name=project_name,
        deployment_name=deployment_name
    )

    document_results = operation.result()

    for doc, custom_entities_result in zip(files, document_results):
        print(doc)
        if custom_entities_result.kind == "CustomEntityRecognition":
            for entity in custom_entities_result.entities:
                print(
                    "\tEntity '{}' has category '{}' with confidence score of '{}'".format(
                        entity.text, entity.category, entity.confidence_score
                    )
                )
        elif custom_entities_result.is_error is True:
            print("\tError with code '{}' and message '{}'".format(
                custom_entities_result.error.code, custom_entities_result.error.message
                )
            )
    ```

1. 保存对文件所做的更改。

## 测试应用程序

现在已准备好测试应用程序。

1. 在 **classify-text** 文件夹中的集成终端中，输入以下命令来运行程序：

    - **C#** ：`dotnet run`
    - **Python**：`python custom-entities.py`

    > **提示**：可以使用 **终端工具栏中的“最大化面板大小** ”图标**^** 查看更多控制台文本。

1. 观察输出。 应用程序应列出每个文本文件中找到的实体的详细信息。

## 清理

如果不再需要项目，可从语言工作室的“项目”页将其删除。 还可以在 [Azure 门户](https://portal.azure.com)中删除 Azure AI 语言服务和关联的存储帐户。
