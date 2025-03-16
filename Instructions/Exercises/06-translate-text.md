---
lab:
  title: 翻译文本
  module: Module 3 - Getting Started with Natural Language Processing
---
{% assign site.title = page.lab.title %}

# 翻译文本

翻译服务是一种认知服务，使你能够在不同语言之间翻译文本。 在本练习中，你将使用它来创建一个简单的应用，它将任何受支持的语言中的输入翻译为所选的目标语言。

## 预配 Azure AI 翻译资源

如果订阅中还没有认知服务资源，则需要预配认知服务资源。

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
1. 在顶部的搜索字段中，搜索 **Azure AI 服务**，然后按 **Enter**，然后在结果中的翻译**下**选择“**创建**”。
1. 使用以下设置创建资源：
    - **订阅**：*Azure 订阅*
    - **资源组**：*创建或选择资源组*
    - **区域**：*选择任何可用区域*
    - **名称**：*输入唯一名称*
    - **定价层**：选择 **F0**（*免费*层）；如果 F 不可用，请选择 **S**（*标准*层）。
    - 负责任的 AI 声明：同意
1. 选择“查看 + 创建”，再选择“创建”以预配资源。
1. 等待部署完成，然后转到部署的资源。
1. 查看“密钥和终结点”页。 在练习的后面部分，你将需要此页面上的信息。

## 准备在 Visual Studio Code 中开发应用

你将使用 Visual Studio Code 开发文本翻译应用。 你的应用的代码文件已在 GitHub 存储库中提供。

> **提示**：如果已克隆 **mslearn-ai-language** 存储库，请在 Visual Studio Code 中打开它。 否则，请按照以下步骤将其克隆到开发环境。

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行“**Git：Clone**”命令，以将 `https://github.com/MicrosoftLearning/mslearn-ai-language` 存储库克隆到本地文件夹（任意文件夹均可）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。

    > **注意**：如果 Visual Studio Code 显示一条弹出消息，提示你信任打开的代码，请单击弹出窗口中的“是，我信任该作者”选项。

4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择**以后再说**。

## 配置应用程序

已提供适用于 C# 和 Python 的应用程序。 这两个应用具有相同的功能。 在本练习中，你将使用 Azure OpenAI 资源完成应用程序的一些关键部分以进行启用。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 **Labfiles/06b-translator-sdk** 文件夹，并根据你的语言首选项展开 **C-Sharp** 文件夹或 **Python** 文件夹及其所包含的 **translate-text** 文件夹。 每个文件夹都包含要在其中集成 Azure AI 翻译功能的应用的语言特定的代码文件。
2. 右键单击包含代码文件的 **translate-text** 文件夹，并打开集成终端。 然后通过运行适用于你的语言首选项的命令，安装人脸 SDK 包：

    **C#：**

    ```
    dotnet add package Azure.AI.Translation.Text --version 1.0.0-beta.1
    ```

    **Python**：

    ```
    pip install azure-ai-translation-text==1.0.0b1
    ```

3. 在 **“资源管理器”** 窗格中的 **translate-text** 文件夹中，打开首选语言的配置文件

    - **C#** ：appsettings.json
    - **Python**：.env
    
4. 更新配置值，以包含**你创建的 Azure AI 翻译 资源中的区域**和**密钥**（在 Azure 门户 中 Azure AI 翻译资源的“密钥和终结点 **”页上可用**）。

    > **注意**：请务必为资源添加 *区域* ， <u>而不是</u> 终结点！

5. 保存此配置文件。

## 添加代码以翻译文本

现在，你已准备好使用 Azure AI 翻译翻译文本。

1. 请注意，text-translation 文件夹中包含客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**：translate.py

    打开代码文件，并在顶部的现有命名空间引用下找到注释“导入命名空间”。 然后在此注释下添加以下特定于语言的代码，以导入使用文本分析 SDK 所需的命名空间：

    **C#**：Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.Translation.Text;
    ```

    **Python**：translate.py

    ```python
    # import namespaces
    from azure.ai.translation.text import *
    from azure.ai.translation.text.models import InputTextItem
    ```

1. 请注意，Main 函数中已提供用于加载配置设置的代码。
1. 然后，找到注释“使用终结点和密钥创建客户端”，并添加以下代码为文本分析 API 创建客户端：

    **C#**：Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credential = new(translatorKey);
    TextTranslationClient client = new(credential, translatorRegion);
    ```

    **Python**：translate.py

    ```python
    # Create client using endpoint and key
    credential = TranslatorCredential(translatorKey, translatorRegion)
    client = TextTranslationClient(credential)
    ```

1. 找到注释 **“选择目标语言**”并添加以下代码，该代码使用 Text 翻译 服务返回支持的语言列表进行翻译，并提示用户为目标语言选择语言代码。

    **C#**：Programs.cs

    ```csharp
    // Choose target language
    Response<GetLanguagesResult> languagesResponse = await client.GetLanguagesAsync(scope:"translation").ConfigureAwait(false);
    GetLanguagesResult languages = languagesResponse.Value;
    Console.WriteLine($"{languages.Translation.Count} languages available.\n(See https://learn.microsoft.com/azure/ai-services/translator/language-support#translation)");
    Console.WriteLine("Enter a target language code for translation (for example, 'en'):");
    string targetLanguage = "xx";
    bool languageSupported = false;
    while (!languageSupported)
    {
        targetLanguage = Console.ReadLine();
        if (languages.Translation.ContainsKey(targetLanguage))
        {
            languageSupported = true;
        }
        else
        {
            Console.WriteLine($"{targetLanguage} is not a supported language.");
        }

    }
    ```

    **Python**：translate.py

    ```python
    # Choose target language
    languagesResponse = client.get_languages(scope="translation")
    print("{} languages supported.".format(len(languagesResponse.translation)))
    print("(See https://learn.microsoft.com/azure/ai-services/translator/language-support#translation)")
    print("Enter a target language code for translation (for example, 'en'):")
    targetLanguage = "xx"
    supportedLanguage = False
    while supportedLanguage == False:
        targetLanguage = input()
        if  targetLanguage in languagesResponse.translation.keys():
            supportedLanguage = True
        else:
            print("{} is not a supported language.".format(targetLanguage))
    ```

1. 找到注释**翻译文本**并添加以下代码，该代码反复提示用户翻译文本，使用 Azure AI 翻译 服务将其翻译为目标语言（自动检测源语言），并显示结果，直到用户输入*退出*。

    **C#**：Programs.cs

    ```csharp
    // Translate text
    string inputText = "";
    while (inputText.ToLower() != "quit")
    {
        Console.WriteLine("Enter text to translate ('quit' to exit)");
        inputText = Console.ReadLine();
        if (inputText.ToLower() != "quit")
        {
            Response<IReadOnlyList<TranslatedTextItem>> translationResponse = await client.TranslateAsync(targetLanguage, inputText).ConfigureAwait(false);
            IReadOnlyList<TranslatedTextItem> translations = translationResponse.Value;
            TranslatedTextItem translation = translations[0];
            string sourceLanguage = translation?.DetectedLanguage?.Language;
            Console.WriteLine($"'{inputText}' translated from {sourceLanguage} to {translation?.Translations[0].To} as '{translation?.Translations?[0]?.Text}'.");
        }
    } 
    ```

    **Python**：translate.py

    ```python
    # Translate text
    inputText = ""
    while inputText.lower() != "quit":
        inputText = input("Enter text to translate ('quit' to exit):")
        if inputText != "quit":
            input_text_elements = [InputTextItem(text=inputText)]
            translationResponse = client.translate(content=input_text_elements, to=[targetLanguage])
            translation = translationResponse[0] if translationResponse else None
            if translation:
                sourceLanguage = translation.detected_language
                for translated_text in translation.translations:
                    print(f"'{inputText}' was translated from {sourceLanguage.language} to {translated_text.to} as '{translated_text.text}'.")
    ```

1. 保存对文件所做的更改。

## 测试应用程序

现在已准备好测试应用程序。

1. 在 **Translate text** 文件夹的集成终端中，输入以下命令以运行程序：

    - **C#** ：`dotnet run`
    - **Python**：`python translate.py`

    > **提示**：可以使用 **终端工具栏中的“最大化面板大小** ”图标 (**^**) 查看更多控制台文本。

1. 出现提示时，输入显示列表中的有效目标语言。
1. 输入要翻译的短语（例如 `This is a test` 或 `C'est un test`）并查看结果，结果应检测源语言并将文本翻译为目标语言。
1. 完成后，输入 `quit`。 可以再次运行应用程序并选择其他目标语言。

## 清理

不再需要项目时，可以在Azure 门户[中删除 Azure AI 翻译资源](https://portal.azure.com)。
