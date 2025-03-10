---
lab:
  title: 识别和合成语音
  module: Module 4 - Create speech-enabled apps with Azure AI services
---

# 识别和合成语音

语音服务是一种 Azure 认知服务，提供与语音相关的各种功能，包括：

- 语音转文本 API，使你能够实现语音识别（将可听到的口头语言转换为文本）。
- 文本转语音 API，使你能够实现语音合成（将文本转换为可听到的语音）。

在本练习中，你将使用这两种 API 来实现语音时钟应用程序。

> **注意**：此练习需要使用配有扬声器/耳机的计算机。 为了获得最佳体验，还需要麦克风。 某些托管的虚拟环境可能能够通过本地麦克风捕获音频，但如果这不起作用（或者根本没有麦克风），则可以使用提供的音频文件进行语音输入。 请仔细按照说明进行操作，因为需要根据使用的是麦克风还是音频文件来选择不同的选项。

## 为语音预配 Azure 资源

如果订阅中还没有认知服务资源，则需要预配认知服务资源。

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
1. 在顶部的搜索字段中，搜索 **Azure AI 服务**，然后按 **Enter**，然后在结果中选择“**在语音服务**下**创建**”。
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

你将使用 Visual Studio Code 开发语音应用。 你的应用的代码文件已在 GitHub 存储库中提供。

> **提示**：如果已克隆 **mslearn-ai-language** 存储库，请在 Visual Studio Code 中打开它。 否则，请按照以下步骤将其克隆到开发环境。

1. 启动 Visual Studio Code。
1. 打开面板 (SHIFT+CTRL+P) 并运行“**Git: Clone**”命令，以将 `https://github.com/MicrosoftLearning/mslearn-ai-language` 存储库克隆到本地文件夹（任意文件夹均可）。
1. 克隆存储库后，在 Visual Studio Code 中打开文件夹。

    > **注意**：如果 Visual Studio Code 显示一条弹出消息，提示你信任打开的代码，请单击弹出窗口中的“是，我信任该作者”选项。

1. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择**以后再说**。

## 配置应用程序

已提供适用于 C# 和 Python 的应用程序。 这两个应用具有相同的功能。 在本练习中，你将使用 Azure OpenAI 资源完成应用程序的一些关键部分以进行启用。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 **Labfiles/07-speech** 文件夹，并根据你的语言首选项展开 **C-Sharp** 文件夹或 **Python** 文件夹和它所包含的 **speaking-clock** 文件夹。 每个文件夹都包含要在其中集成 Azure AI 语音功能的应用的语言特定代码文件。
1. 右键单击 speaking-clock 文件夹，并打开集成终端。 然后通过运行适用于你的语言首选项的命令，安装语音 SDK 包：

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.30.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.30.0
    ```

1. 在 **资源管理器** 窗格中，在 **speaking-clock** 文件夹中，打开首选语言的配置文件

    - **C#** ：appsettings.json
    - **Python**：.env

1. 更新配置值，以包含**所创建的 Azure AI 语音资源中的区域**和**密钥**（在 Azure 门户 中 Azure AI 语音资源的“密钥和终结点 **”页上可用**）。

    > **注意**：请务必为资源添加 *区域*，<u>而不是</u> 终结点！

1. 保存此配置文件。

## 添加代码以使用 Azure AI 语音 SDK

1. 请注意，speaking-clock 文件夹中包含客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**：speaking-clock.py

    打开代码文件，并在顶部的现有命名空间引用下找到注释“导入命名空间”。 然后在此注释下添加以下语言特定的代码，以导入使用语音 SDK 所需的命名空间：

    **C#** ：Program.cs

    ```csharp
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    ```

    **Python**：speaking-clock.py

    ```python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

1. 请注意，Main 函数中已经提供了从配置文件加载认知服务密钥和区域的代码。 必须使用这些变量为认知服务资源创建 SpeechConfig。 在注释“配置语音服务”下添加以下代码：

    **C#** ：Program.cs

    ```csharp
    // Configure speech service
    speechConfig = SpeechConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    
    // Configure voice
    speechConfig.SpeechSynthesisVoiceName = "en-US-AriaNeural";
    ```

    **Python**：speaking-clock.py

    ```python
    # Configure speech service
    speech_config = speech_sdk.SpeechConfig(ai_key, ai_region)
    print('Ready to use speech service in:', speech_config.region)
    ```

1. 保存你的更改并返回到 speaking-clock 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. 如果你使用 C#，则可以忽略关于在异步方法中使用 await 运算符的任何警告 - 我们之后将修复此问题。 代码应显示应用程序将使用的语音服务资源的区域。

## 添加代码以识别语音

为认知服务资源中的语音服务添加了 SpeechConfig 之后，现在，你可以使用语音转文本 API 来识别语音并将其转录为文本 。

> **重要说明**：本节包含两个替代过程的说明。 如果有工作麦克风，请按照第一个过程操作。 如果要使用音频文件模拟口语输入，请遵循第二个过程。

### 如果麦克风正常工作

1. 在程序的 Main 函数中，请注意代码使用 TranscribeCommand 函数来接受语音输入 。
1. 在 TranscribeCommand 函数的注释“配置语音识别”下，添加以下适当代码以创建 SpeechRecognizer 客户端，该客户端可用于识别和转录使用默认系统麦克风的语音  ：

    **C#**

    ```csharp
    // Configure speech recognition
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    Console.WriteLine("Speak now...");
    ```

    **Python**

    ```python
    # Configure speech recognition
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    print('Speak now...')
    ```

1. 现在，进入下面的“添加代码以处理转录的命令”部分。

---

### 或者，使用来自文件的音频输入

1. 在终端窗口中，输入以下命令以安装可用于播放音频文件的库：

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

1. 在程序的代码文件中的现有命名空间导入下，添加以下代码以导入刚刚安装的库：

    **C#** ：Program.cs

    ```csharp
    using System.Media;
    ```

    **Python**：speaking-clock.py

    ```python
    from playsound import playsound
    ```

1. 在 Main 函数中，请注意代码使用 TranscribeCommand 函数来接受语音输入 。 然后，在 TranscribeCommand 函数的注释“配置语音识别”下，添加以下适当代码以创建 SpeechRecognizer 客户端，该客户端可用于识别和转录音频文件中的语音  ：

    **C#** ：Program.cs

    ```csharp
    // Configure speech recognition
    string audioFile = "time.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

    **Python**：speaking-clock.py

    ```python
    # Configure speech recognition
    current_dir = os.getcwd()
    audioFile = current_dir + '\\time.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

---

### 添加代码以处理转录的命令

1. 在 TranscribeCommand 函数的注释“处理语音输入”下，添加以下代码以侦听语音输入，注意不要替换返回命令的函数末尾的代码 ：

    **C#** ：Program.cs

    ```csharp
    // Process speech input
    SpeechRecognitionResult speech = await speechRecognizer.RecognizeOnceAsync();
    if (speech.Reason == ResultReason.RecognizedSpeech)
    {
        command = speech.Text;
        Console.WriteLine(command);
    }
    else
    {
        Console.WriteLine(speech.Reason);
        if (speech.Reason == ResultReason.Canceled)
        {
            var cancellation = CancellationDetails.FromResult(speech);
            Console.WriteLine(cancellation.Reason);
            Console.WriteLine(cancellation.ErrorDetails);
        }
    }
    ```

    **Python**：speaking-clock.py

    ```python
    # Process speech input
    speech = speech_recognizer.recognize_once_async().get()
    if speech.reason == speech_sdk.ResultReason.RecognizedSpeech:
        command = speech.text
        print(command)
    else:
        print(speech.reason)
        if speech.reason == speech_sdk.ResultReason.Canceled:
            cancellation = speech.cancellation_details
            print(cancellation.reason)
            print(cancellation.error_details)
    ```

1. 保存你的更改并返回到 speaking-clock 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. 如果使用麦克风，请对着麦克风清晰地说出“what time is it?”。 程序应转录你的语音输入并显示时间（基于运行代码的计算机的本地时间，此时间可能并非你所在位置的正确时间）。

    SpeechRecognizer 会为你提供大约 5 秒钟的说话时间。 如果它未检测到任何语音输入，则会生成“无匹配”结果。

    如果 SpeechRecognizer 遇到错误，它将生成“已取消”结果。 然后，应用程序中的代码将显示错误消息。 最可能的原因是配置文件中的密钥或区域不正确。

## 合成语音

语音时钟应用程序接受语音输入，但它实际上并不生成语音输出！ 让我们通过添加代码以合成语音来解决此问题。

1. 在程序的 Main 函数中，请注意代码使用 TellTime 函数来告知用户当前时间 。
1. 在 TellTime 函数的注释“配置语音合成”下，添加以下代码以创建可用于生成语音输出的 SpeechSynthesizer 客户端  ：

    **C#** ：Program.cs

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python**：speaking-clock.py

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

    > 默认音频配置使用默认系统音频设备进行输出，因此你无需显式提供 AudioConfig。 如果需要将音频输出重定向到某个文件，可以使用包含文件路径的 AudioConfig 来执行此操作。

1. 在 TellTime 函数的注释“合成语音输出”下，添加以下代码以生成语音输出，注意不要替换输出响应的函数末尾的代码 ：

    **C#** ：Program.cs

    ```csharp
    // Synthesize spoken output
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**：speaking-clock.py

    ```python
    # Synthesize spoken output
    speak = speech_synthesizer.speak_text_async(response_text).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. 保存你的更改并返回到 speaking-clock 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. 出现提示时，对着麦克风清晰地说出“what time is it?”。 程序应通过语音方式告知你时间。

## 使用其他语音

语音时钟应用程序使用可更改的默认语音。 语音服务支持一系列标准语音，以及更类似于人的神经语音 。 此外，还可以创建自定义语音。

> **注意**：有关神经语音和标准语音的列表，请参阅 [Speech Studio 中的语音库](https://speech.microsoft.com/portal/voicegallery) 。

1. 在 TellTime 函数的注释“配置语音合成”下，按如下方式修改代码以指定替代语音，然后再创建 SpeechSynthesizer 客户端  ：

   **C#** ：Program.cs

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-LibbyNeural"; // change this
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python**：speaking-clock.py

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = 'en-GB-LibbyNeural' # change this
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

1. 保存你的更改并返回到 speaking-clock 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. 出现提示时，对着麦克风清晰地说出“what time is it?”。 程序应通过指定的语音告知你时间。

## 使用语音合成标记语言

语音合成标记语言 (SSML) 使你能够自定义使用基于 XML 的格式合成语音的方式。

1. 在 TellTime 函数中，将注释“合成语音输出”下的所有当前代码替换为以下代码（保留注释“输出响应”下的代码）  ：

   **C#** ：Program.cs

    ```csharp
    // Synthesize spoken output
    string responseSsml = $@"
        <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'>
            <voice name='en-GB-LibbyNeural'>
                {responseText}
                <break strength='weak'/>
                Time to end this lab!
            </voice>
        </speak>";
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakSsmlAsync(responseSsml);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**：speaking-clock.py

    ```python
    # Synthesize spoken output
    responseSsml = " \
        <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'> \
            <voice name='en-GB-LibbyNeural'> \
                {} \
                <break strength='weak'/> \
                Time to end this lab! \
            </voice> \
        </speak>".format(response_text)
    speak = speech_synthesizer.speak_ssml_async(responseSsml).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. 保存你的更改并返回到 speaking-clock 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. 出现提示时，对着麦克风清晰地说出“what time is it?”。 程序应以 SSML 中指定的语音（替代 SpeechConfig 中指定的语音）告知你时间，然后在停顿一下之后告知你现在可以结束本实验室！

## 详细信息

有关如何使用语音转文本 API 和文本转语音 API 的详细信息，请参阅[语音转文本文档](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text)和[文本转语音文档](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech) 。
