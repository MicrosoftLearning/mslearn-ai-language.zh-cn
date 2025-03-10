---
lab:
  title: 创建问题解答解决方案
  module: Module 6 - Create question answering solutions with Azure AI Language
---

# 创建问题解答解决方案

最常见的对话方案之一是通过常见问题解答 (FAQ) 知识库提供支持。 许多组织将 FAQ 发布为文档或网页，这适用于少量的问答对，但针对大型文档搜索可能很麻烦且十分耗时。

语言服务包含一项问题解答功能，通过该功能，你能够创建可使用自然语言输入来查询的问答对知识库，该功能通常用作一种资源，机器人可以用它来查找用户提交的问题的答案。

## 预配 *Azure AI 语言*资源

如果订阅中尚无资源，则需要预配 **Azure AI 语言服务**资源。 此外，若要创建和托管用于问答的知识库，需要启用 **“问答**”功能。

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
1. 选择“创建资源”。
1. 在搜索字段中，搜索“**语言服务**”。 选择“语言服务”资源下的“创建”。
1. 选择“**自定义问题解答**”块。 然后选择“继续创建资源”。 你需要输入以下设置：

    - **订阅**：Azure 订阅
    - **资源组**：*创建或选择资源组*
    - 区域：选择任何可用位置
    - **名称**：*输入唯一名称*
    - **定价层**：选择 **F0**（*免费*层）；如果 F 不可用，请选择 **S**（*标准*层）。
    - **Azure 搜索位置**：在与语言资源相同的全球区域中选择一个位置。
    - **Azure 搜索定价层**：免费 (F)（如果该层级不可用，请选择基本 (B)）
    - **负责任的 AI 声明**：同意

1. 依次选择“查看 + 创建”、“创建”。

    > 自定义问题解答使用 Azure 搜索来对问答知识库编制索引和进行查询。

1. 等待部署完成，然后转到部署的资源。
1. 查看“密钥和终结点”页。 在练习的后面部分，你将需要此页面上的信息。

## 创建问题解答项目

若要在语言资中创建问题解答知识库，可以使用 Language Studio 门户创建问题解答项目。 在本例中，你将创建包含有关 [Microsoft Learn](https://docs.microsoft.com/learn) 的问答知识库。

1. 在新的浏览器标签页中，转到 Language Studio 门户 ()，然后使用与 Azure 订阅关联的 Microsoft 帐户登录。
1. 如果系统提示选择语言资源，请选择以下设置：
    - **Azure Directory**：包含订阅的 Azure 目录。
    - **订阅**：Azure 订阅。
    - 资源语言****
    - **语言资源**：先前创建的语言资源。

    如果系统未提示你选择语言资源，原因可能是订阅中有多个语言资源；在这种情况下，你应该：

    1. 在页面顶部的栏中，选择“设置(⚙)”**** 按钮。
    2. 在“设置”页上，查看“资源”选项卡。
    3. 选择刚刚创建的语言资源，然后单击“切换资源”。
    4. 在页面顶部，单击“Language Studio”以返回到 Language Studio 主页。

1. 在门户顶部的“新建”菜单中，选择“自定义问题解答” 。
1. 在*“**创建项目**”向导中的“**选择语言设置**”页上，选择“**为此资源中所有的项目设置语言**”选项，然后选择“**英语**”作为语言。 然后，选择“下一步”。
1. 在“输入基本信息”页上，输入以下详细信息，然后单击“下一步”：
    - **名称***`LearnFAQ`
    - **说明**：`FAQ for Microsoft Learn`
    - 未返回任何答案时的默认答案：
1. 选择**下一步**。
1. 在“检查并完成”页上，单击“创建项目”。

## 将源添加到知识库

可以从头开始创建知识库，但通常先从现有的 FAQ 页面或文档中导入问答。 在本案例中，你将从 Microsoft Learn 的现有常见问题解答网页导入数据，并且还将导入一些预定义的“聊天”问题和答案，以支持常见的对话交流。

1. 在问题解答项目的“管理源”页上，在"&#9547; 添加源"列表中，选择“URL”  。 然后在“添加 URL”对话框中，单击“&#9547; 添加 url”并添加以下 URL，然后单击“全部添加”以将其添加到知识库  ：
    - **名称**：`Learn FAQ Page`
    - **URL**：`https://docs.microsoft.com/en-us/learn/support/faq`
1. 在问题解答项目的“管理源”页上，在“&#9547; 添加源”列表中，选择“聊天”  。 在“添加聊天”对话框中，选择“友好”，然后单击“添加聊天”  。

## 编辑知识库

你的知识库已填充了 Microsoft Learn FAQ 中的问答对，并补充了一组对话式聊天问答对。 可以通过添加更多问答对来扩展知识库。

1. 在 Language Studio 中的 LearnFAQ 项目中，选择“编辑知识库”页以查看现有问答对（如果显示了一些提示，请阅读提示，然后单击“知道了”将其消除，或单击“跳过所有提示”   ）
1. 在知识库的“问题解答对”选项卡中，选择“&#65291;”，然后使用以下设置创建新的问题解答对 ：
    - **源**：`https://docs.microsoft.com/en-us/learn/support/faq`
    - **问题**：`What are Microsoft credentials?`
    - **答案**：`Microsoft credentials enable you to validate and prove your skills with Microsoft technologies.`
1. 选择“完成”。
1. 在创建的“Microsoft 凭据是什么？”问题页面中，展开“备用问题”********。 然后添加备用问题 `How can I demonstrate my Microsoft technology skills?`。

    在某些情况下，让用户能够跟进答案以创建多回合对话是有意义的，这让用户能够以迭代方式细化问题，从而得到他们所需的答案。

1. 在为认证问题输入的答案下，展开“跟进提示”，添加以下跟进提示：
    - 在提示中向用户显示的文本：`Learn more about credentials`。
    - 选择“创建指向新对的链接”，然后输入以下文本：`You can learn more about credentials on the [Microsoft credentials page](https://docs.microsoft.com/learn/credentials/).`
    - 仅在上下文流中显示：选定。 此选项可确保仅在原始认证问题的跟进问题的上下文中返回答案。
1. 选择“ **添加提示**”。

## 训练和测试知识库

现在你已有了知识库，接下来可以在 Language Studio 中对它进行测试。

1. 通过选择**左侧“问题答案对 **”选项卡下的**“保存**”按钮，将更改保存到知识库。
1. 保存更改后，选择“测试”以打开测试窗格。
1. 在测试窗格中的顶部，取消选择 **“包括短答案响应** ”（如果尚未取消选择）。 然后在底部输入消息 `Hello`。 应会返回相应的响应。
1. 在测试窗格的底部，输入消息 `What is Microsoft Learn?`。 应会返回来自常见问题解答的适当答复。
1. 输入消息 `Thanks!` 应返回相应的聊天响应。
1. 输入消息 `Tell me about Microsoft credentials`。 创建的答案应与跟进提示链接一起返回。
1. 选择“详细了解认证”跟进提示链接。 应会返回带有指向认证页面的链接的跟进答案。
1. 完成知识库测试后，关闭测试窗格。

## 部署知识库

知识库提供了客户端应用程序可以用来回答问题的后端服务。 现在，可以准备发布知识库并从客户端访问其 REST 接口了。

1. 在 Language Studio 的 **LearnFAQ** 项目中，从左侧导航菜单中选择“**部署知识库**”页面。
1. 在页面顶部，选择“删除”。 然后单击“部署”以确认要部署知识库。
1. 部署完成后，选择“**获取预测 URL**”以查看知识库的 REST 终结点，并注意示例请求包含参数：
    - **projectName**：项目的名称（应为 *LearnFAQ*）
    - **deploymentName**：部署的名称（应 *为生产*）
1. 关闭预测 URL 对话框。

## 准备在 Visual Studio Code 中开发应用

你将使用 Visual Studio Code 开发问题解答应用。 你的应用的代码文件已在 GitHub 存储库中提供。

> **提示**：如果已克隆 **mslearn-ai-language** 存储库，请在 Visual Studio Code 中打开它。 否则，请按照以下步骤将其克隆到开发环境。

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行“**Git: Clone**”命令，以将 `https://github.com/MicrosoftLearning/mslearn-ai-language` 存储库克隆到本地文件夹（任意文件夹均可）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。

    > **注意**：如果 Visual Studio Code 显示一条弹出消息，提示你信任打开的代码，请单击弹出窗口中的“是，我信任该作者”选项。

4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择**以后再说**。

## 配置应用程序

已提供适用于 C# 和 Python 的应用程序，以及用于测试汇总情况的示例文本文件。 这两个应用具有相同的功能。 首先，你将完成应用程序的一些关键部分，使其能够使用 Azure AI 语言资源。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 19-face 文件夹，并根据你的语言首选项展开 C-Sharp 文件夹或 Python 文件夹。 每个文件夹都包含要在其中集成 Azure AI 语言问答功能的应用的语言特定文件。
2. 右键单击 04-containers 文件夹，并打开集成终端。 然后通过运行适用于你的语言首选项的命令，安装语言理解 SDK 包：

    **C#：**

    ```
    dotnet add package Azure.AI.Language.QuestionAnswering
    ```

    **Python**：

    ```
    pip install azure-ai-language-questionanswering
    ```

3. 在 **资源管理器** 窗格中的 **qna-app** 文件夹中，打开首选语言的配置文件

    - **C#** ：appsettings.json
    - **Python**：.env
    
4. 更新配置值，以包含创建的 Azure 语言资源的**终结点**和**密钥**（位于 Azure 门户中 Azure AI 语言资源的“密钥和终结点”**** 页面上）。 部署知识库的项目名称和部署名称也应在此文件中。
5. 保存此配置文件。

## 将代码添加到应用程序中

现在，你已准备好添加导入所需 SDK 库所需的代码，建立与已部署项目经过身份验证的连接，并提交问题。

1. 请注意，face-api 文件夹中包含客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**：qna-app.py

    打开代码文件，并在顶部的现有命名空间引用下找到注释“导入命名空间”。 然后在此注释下添加以下特定于语言的代码，以导入使用文本分析 SDK 所需的命名空间：

    **C#**：Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.Language.QuestionAnswering;
    ```

    **Python**：qna-app.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.questionanswering import QuestionAnsweringClient
    ```

1. 请注意，Main 函数中已经提供从配置文件加载认知服务终结点和密钥的代码。 然后，找到注释“使用终结点和密钥创建客户端”，并添加以下代码为文本分析 API 创建客户端：

    **C#**：Programs.cs

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    QuestionAnsweringClient aiClient = new QuestionAnsweringClient(endpoint, credentials);
    ```

    **Python**：qna-app.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = QuestionAnsweringClient(endpoint=ai_endpoint, credential=credential)
    ```

1. 在 **Main** 函数中，找到注释 **提交问题并显示答案**，并添加以下代码以从命令行重复读取问题，将其提交到服务，并显示答案的详细信息：

    **C#**：Programs.cs

    ```C#
    // Submit a question and display the answer
    string user_question = "";
    while (true)
        {
            Console.Write("Question: ");
            user_question = Console.ReadLine();
            if (user_question.ToLower() == "quit")
                break;
            QuestionAnsweringProject project = new QuestionAnsweringProject(projectName, deploymentName);
            Response<AnswersResult> response = aiClient.GetAnswers(user_question, project);
            foreach (KnowledgeBaseAnswer answer in response.Value.Answers)
            {
                Console.WriteLine(answer.Answer);
                Console.WriteLine($"Confidence: {answer.Confidence:P2}");
                Console.WriteLine($"Source: {answer.Source}");
                Console.WriteLine();
            }
        }
    ```

    **Python**：qna-app.py

    ```Python
    # Submit a question and display the answer
    user_question = ''
    while True:
        user_question = input('\nQuestion:\n')
        if user_question.lower() == "quit":                
            break
        response = ai_client.get_answers(question=user_question,
                                        project_name=ai_project_name,
                                        deployment_name=ai_deployment_name)
        for candidate in response.answers:
            print(candidate.answer)
            print("Confidence: {}".format(candidate.confidence))
            print("Source: {}".format(candidate.source))
    ```

1. 保存你的更改并返回到 face-api 文件夹的集成终端，然后输入以下命令以运行程序：

    - **C#** ：`dotnet run`
    - **Python**：`python qna-app.py`

    > **提示**：可以使用 **终端工具栏中的“最大化面板大小** ”图标**^** 查看更多控制台文本。

1. 出现提示时，输入要提交到问题解答项目的问题;例如 `What is a learning path?`。
1. 查看返回的输出：
1. 提出开放式问题。 完成后，输入 `quit`。

## 清理资源

提示：如果已完成浏览 Azure Cosmos DB，则可以删除在本练习中创建的资源组。 操作步骤如下：

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 浏览到在此实验室中创建的 Azure AI 语言资源。
3. 在资源页上，选择“删除 **”** 并按照说明删除资源。

## 详细信息

若要了解有关 Azure AI 语言的详细信息，请参阅 [Azure AI 语言文档](https://learn.microsoft.com/azure/ai-services/language-service/question-answering/overview)
