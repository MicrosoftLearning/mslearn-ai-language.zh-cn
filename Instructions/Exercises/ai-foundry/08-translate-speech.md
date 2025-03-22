---
lab:
  title: 翻译语音
  module: Module 8 - Translate speech with Azure AI Speech
---

# 翻译语音

Azure AI 语音包括一个可用于翻译口语的语音翻译 API。 例如，假设你想要开发一种翻译工具应用程序，当人们在不会讲当地语言的地方旅行时便可以使用该应用程序。 他们能够以自己的语言说出诸如“车站在哪里？” 或“我需要寻找药店”之类的惯用语，并让该应用程序将其翻译为当地语言。

> **注意**：此练习需要使用配有扬声器/耳机的计算机。 为了获得最佳体验，还需要麦克风。 某些托管的虚拟环境可能能够通过本地麦克风捕获音频，但如果这不起作用（或者根本没有麦克风），则可以使用提供的音频文件进行语音输入。 请仔细按照说明进行操作，因为需要根据使用的是麦克风还是音频文件来选择不同的选项。

## 预配 *Azure AI 语音*资源

如果订阅中还没有 Azure AI 语音资源，则需要预配 Azure AI 语音资源。

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
1. 在顶部的搜索字段中，搜索 **Azure AI 服务**，然后按 **Enter**，然后在结果中选择“**在语音服务**下**创建**”。
1. 使用以下设置创建资源：
    - **订阅**：*Azure 订阅*
    - **资源组**：*创建或选择资源组*
    - **区域**：*选择任何可用区域*
    - **名称**：*输入唯一名称*
    - **定价层**：选择 **F0**（*免费*层）；如果 F 不可用，请选择 **S**（*标准*层）。
    - 负责任的 AI 声明：同意
1. 选择“查看 + 创建”，然后选择“创建”以预配资源。
1. 等待部署完成，然后转到部署的资源。
1. 查看“密钥和终结点”页。 在练习的后面部分，你将需要此页面上的信息。

## 准备在 Visual Studio Code 中开发应用

你将使用 Visual Studio Code 开发语音应用。 你的应用的代码文件已在 GitHub 存储库中提供。

> **提示**：如果已克隆 **mslearn-ai-language** 存储库，请在 Visual Studio Code 中打开它。 否则，请按照以下步骤将其克隆到开发环境。

1. 启动 Visual Studio Code。
1. 打开面板 (SHIFT+CTRL+P) 并运行“**Git：Clone**”命令，以将 `https://github.com/MicrosoftLearning/mslearn-ai-language` 存储库克隆到本地文件夹（任意文件夹均可）。
1. 克隆存储库后，在 Visual Studio Code 中打开文件夹。

    > **注意**：如果 Visual Studio Code 显示一条弹出消息，提示你信任打开的代码，请单击弹出窗口中的“是，我信任该作者”选项。

1. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择**以后再说**。

## 配置应用程序

已提供适用于 C# 和 Python 的应用程序。 这两个应用具有相同的功能。 在本练习中，你将使用 Azure AI 语音资源完成应用程序的一些关键部分以进行启用。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 **Labfiles/08-speech-translation** 文件夹，并根据你的语言首选项和它所包含的 **translator** 文件夹展开 **C-Sharp** 文件夹或 **Python** 文件夹。 每个文件夹都包含要在其中集成 Azure AI 语音功能的应用的语言特定代码文件。
1. 右键单击 translator 文件夹，并打开集成终端。 然后通过运行适用于你的语言首选项的命令，安装语音 SDK 包：

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.30.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.30.0
    ```

1. 在资源管理器窗格中的 “translator”文件夹中，打开首选语言的配置文件

    - **C#** ：appsettings.json
    - **Python**：.env

1. 更新配置值，以包含**所创建的 Azure AI 语音资源中的区域**和**密钥**（在 Azure 门户 中 Azure AI 语音资源的“密钥和终结点 **”页上可用**）。

    > **注意**：请务必为资源添加 *区域*，<u>而不是</u> 终结点！

1. 保存此配置文件。

## 添加代码以使用语音 SDK

1. 请注意，translator 文件夹中包含客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**：translator.py

    打开代码文件，并在顶部的现有命名空间引用下找到注释“导入命名空间”。 然后在此注释下添加以下语言特定的代码，以导入使用  Azure AI 语音 SDK 所需的命名空间：

    **C#** ：Program.cs

    ```csharp
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Translation;
    ```

    **Python**：translator.py

    ```python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

1. 请注意，**Main** 函数中已经提供了从配置文件加载 Azure AI 语音密钥和区域的代码。 必须使用这些变量为 Azure AI 语音资源创建将用于翻译语音输入的 **SpeechTranslationConfig**。 在注释“配置翻译”下添加以下代码：

    **C#** ：Program.cs

    ```csharp
    // Configure translation
    translationConfig = SpeechTranslationConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    translationConfig.SpeechRecognitionLanguage = "en-US";
    translationConfig.AddTargetLanguage("fr");
    translationConfig.AddTargetLanguage("es");
    translationConfig.AddTargetLanguage("hi");
    Console.WriteLine("Ready to translate from " + translationConfig.SpeechRecognitionLanguage);
    ```

    **Python**：translator.py

    ```python
    # Configure translation
    translation_config = speech_sdk.translation.SpeechTranslationConfig(ai_key, ai_region)
    translation_config.speech_recognition_language = 'en-US'
    translation_config.add_target_language('fr')
    translation_config.add_target_language('es')
    translation_config.add_target_language('hi')
    print('Ready to translate from',translation_config.speech_recognition_language)
    ```

1. 将使用 SpeechTranslationConfig 将语音转换为文本，但还将使用 SpeechConfig 将译文合成为语音 。 在注释“配置语音”下添加以下代码：

    **C#** ：Program.cs

    ```csharp
    // Configure speech
    speechConfig = SpeechConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    ```

    **Python**：translator.py

    ```python
    # Configure speech
    speech_config = speech_sdk.SpeechConfig(ai_key, ai_region)
    ```

1. 保存你的更改并返回到 translator 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. 如果你使用 C#，则可以忽略关于在异步方法中使用 await 运算符的任何警告 - 我们之后将修复此问题。 代码应显示一条消息，指示已准备好从 en-US 进行翻译。 按 Enter 结束程序。

## 实现语音翻译

现在，你已经为 Azure AI 语音资源中的语音服务添加了 **SpeechTranslationConfig**，接下来，可以使用 Azure AI 语音翻译 API 来识别和翻译语音。

> **重要说明**：本节包含两个替代过程的说明。 如果有工作麦克风，请按照第一个过程操作。 如果要使用音频文件模拟口语输入，请遵循第二个过程。

### 如果麦克风正常工作

1. 在程序的 Main 函数中，请注意代码使用 Translate 函数来翻译语音输入 。
1. 在 Translate 函数的注释“翻译语音”下，添加以下代码以创建 TranslationRecognizer 客户端，该客户端可用于识别和翻译使用默认系统麦克风输入的语音  。

    **C#** ：Program.cs

    ```csharp
    // Translate speech
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Speak now...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```

    **Python**：translator.py

    ```python
    # Translate speech
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config)
    print("Speak now...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

    > **注意**：应用程序中的代码在一次调用中将输入翻译为三种语言。 仅显示特定语言的翻译，但你可以通过在结果的翻译集合中指定目标语言代码来检索任何翻译。

1. 现在，进入下面的“运行程序”部分。

---

### 或者，使用来自文件的音频输入

1. 在终端窗口中，输入以下命令以安装可用于播放音频文件的库：

    **C#** ：Program.cs

    ```csharp
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**：translator.py

    ```python
    pip install playsound==1.3.0
    ```

1. 在程序的代码文件中的现有命名空间导入下，添加以下代码以导入刚刚安装的库：

    **C#** ：Program.cs

    ```csharp
    using System.Media;
    ```

    **Python**：translator.py

    ```python
    from playsound import playsound
    ```

1. 在程序的 Main 函数中，请注意代码使用 Translate 函数来翻译语音输入 。 然后，在 Translate 函数的注释“翻译语音”下，添加以下代码以创建 TranslationRecognizer 客户端，该客户端可用于识别和翻译文件中的语音  。

    **C#** ：Program.cs

    ```csharp
    // Translate speech
    string audioFile = "station.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Getting speech from file...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```

    **Python**：translator.py

    ```python
    # Translate speech
    audioFile = 'station.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config)
    print("Getting speech from file...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

---

### 运行程序

1. 保存你的更改并返回到 translator 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. 出现提示时，输入有效的语言代码（fr、es 或 hi），然后，如果使用的是麦克风，请对着麦克风清晰地说出“车站在哪里？” 或出国旅行时可能使用的一些其他惯用语。 程序应转录你的语音输入并将其翻译为你指定的语言（法语、西班牙语或印地语）。 重复此过程，尝试使用应用程序支持的各种语言。 完成后，按 Enter 结束程序。

    注意：TranslationRecognizer 会为你提供大约 5 秒钟的说话时间。 如果它未检测到任何语音输入，则会生成“无匹配”结果。 由于字符编码问题，控制台窗口中可能无法始终正确显示印地语译文。

> **注意**：应用程序中的代码在一次调用中将输入翻译为三种语言。 仅显示特定语言的翻译，但你可以通过在结果的翻译集合中指定目标语言代码来检索任何翻译。

## 将翻译合成为语音

到目前为止，应用程序已将语音输入转换为文本；如果你在旅行时需要向其他人寻求帮助，这可能已经足够。 不过，最好是能够通过适当的语音将译文大声朗读出来。

1. 在 Translate 函数的注释“合成翻译”下，添加以下代码以使用 SpeechSynthesizer 客户端通过默认扬声器将翻译合成为语音  ：

    **C#** ：Program.cs

    ```csharp
    // Synthesize translation
    var voices = new Dictionary<string, string>
                    {
                        ["fr"] = "fr-FR-HenriNeural",
                        ["es"] = "es-ES-ElviraNeural",
                        ["hi"] = "hi-IN-MadhurNeural"
                    };
    speechConfig.SpeechSynthesisVoiceName = voices[targetLanguage];
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(translation);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**：translator.py

    ```python
    # Synthesize translation
    voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
    }
    speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    speak = speech_synthesizer.speak_text_async(translation).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. 保存你的更改并返回到 translator 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. 出现提示时，输入有效的语言代码（fr、es 或 hi），然后对着麦克风清晰地说出出国旅行时可能使用的惯用语  。 程序应转录你的语音输入并通过语音翻译作出响应。 重复此过程，尝试使用应用程序支持的各种语言。 完成后，按 Enter 结束程序。

> 注意：在此示例中，你使用 SpeechTranslationConfig 将语音转换为了文本，然后使用 SpeechConfig 将翻译合成为了语音。  事实上，你可以使用 SpeechTranslationConfig 直接合成翻译，但这仅适用于翻译为一种语言的情况，并且会生成通常是保存为文件而不是直接发送给扬声器的音频流。

## 详细信息

有关如何使用 Azure AI 语音翻译 API 的详细信息，请参阅语音翻译文档。
