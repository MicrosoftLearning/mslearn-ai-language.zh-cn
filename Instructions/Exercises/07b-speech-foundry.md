---
lab:
  title: 识别和合成语音（Azure AI Foundry 版本）
  module: Module 4 - Create speech-enabled apps with Azure AI services
---

<!--
Possibly update to use standalone AI Service instead of Foundry?
-->

# 识别和合成语音

语音服务是一种 Azure 认知服务，提供与语音相关的各种功能，包括：

- 语音转文本 API，使你能够实现语音识别（将可听到的口头语言转换为文本）。
- 文本转语音 API，使你能够实现语音合成（将文本转换为可听到的语音）。

在本练习中，你将使用这两种 API 来实现语音时钟应用程序。

> **备注** 此练习设计为在 Azure Cloud Shell 中完成，不支持直接访问计算机的声音硬件。 因此，实验室将使用音频文件作为语音输入和输出流。 提供了使用麦克风和扬声器实现相同结果的代码，作为参考。

## 创建 Azure AI Foundry 项目

让我们首先创建 Azure AI Foundry 项目。

1. 在 Web 浏览器中打开 [Azure AI Foundry 门户](https://ai.azure.com)，网址为：`https://ai.azure.com`，然后使用 Azure 凭据登录。 关闭首次登录时打开的任何使用技巧或快速入门窗格，如有必要，请使用左上角的 **Azure AI Foundry** 徽标导航到主页，如下图所示：

    ![Azure AI Foundry 门户的屏幕截图。](./ai-foundry/media/ai-foundry-home.png)

1. 在主页中，选择“**+ 创建项目**”。
1. 在“**创建项目**”向导中，输入项目的有效名称，如果建议使用现有中心，请选择新建中心的选项。 然后查看将自动创建的 Azure 资源以支持中心和项目。
1. 选择“**自定义**”并为中心指定以下设置：
    - **中心名称**：*中心的有效名称*
    - **订阅**：Azure 订阅
    - **资源组**：*创建或选择资源组*
    - **位置**：选择任何可用区域
    - **连接 Azure AI 服务或 Azure OpenAI**：*新建 AI 服务资源*
    - **连接 Azure AI 搜索**：*使用唯一名称创建新的 Azure AI 搜索资源*

1. 选择“**下一步**”查看配置。 然后，选择“**创建**”并等待该进程完成。
1. 创建项目后，关闭显示的所有使用技巧，并查看 Azure AI Foundry 门户中的项目页面，如下图所示：

    ![Azure AI Foundry 门户中 Azure AI 项目详细信息的屏幕截图。](./ai-foundry/media/ai-foundry-project.png)

## 准备和配置语音时钟应用

1. 在 Azure AI Foundry 门户中，查看项目的“**概述**”页。
1. 在“**项目详细信息**”区域中，记下项目的“**项目连接字符串**”和“**位置**”。你将使用连接字符串连接到客户端应用程序中的项目，并且需要连接到 Azure AI 服务语音终结点的位置。
1. 打开新的浏览器选项卡（使 Azure AI Foundry 门户在现有选项卡中保持打开状态）。 然后在新选项卡中，浏览到 [Azure 门户](https://portal.azure.com)，网址为：`https://portal.azure.com`；如果出现提示，请使用 Azure 凭据登录。
1. 使用页面顶部搜索栏右侧的 **[\>_]** 按钮在 Azure 门户中创建新的 Cloud Shell，选择 ***PowerShell*** 环境。 Cloud Shell 在 Azure 门户底部的窗格中提供命令行接口。

    > **备注**：如果以前创建了使用 *Bash* 环境的 Cloud Shell，请将其切换到 ***PowerShell***。

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

    > **提示**：将命令粘贴到 cloudshell 中时，输出可能会占用大量屏幕缓冲区。 可以通过输入 `cls` 命令来清除屏幕，以便更轻松地专注于每项任务。

1. 在 PowerShell 窗格中，输入以下命令以克隆包含此练习的 GitHub 存储库：

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language mslearn-ai-language
    ```

    ***按照所选编程语言的步骤操作。***

1. 克隆存储库后，导航到包含语音时钟应用程序代码文件的文件夹：  

    **Python**

    ```
   cd mslearn-ai-language/labfiles/07b-speech/python/speaking-clock
    ```

    **C#**

    ```
   cd mslearn-ai-language/labfiles/07b-speech/c-sharp/speaking-clock
    ```

1. 在 Cloud Shell 命令行窗格中，输入以下命令安装将使用的库：

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects azure-cognitiveservices-speech==1.42.0
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --prerelease
   dotnet add package Microsoft.CognitiveServices.Speech --version 1.42.0
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

    该文件已在代码编辑器中打开。

1. 在代码文件中，将 **your_project_endpoint** 和 **your_location** 占位符替换为项目的连接字符串和位置（从 Azure AI Foundry 门户中的项目“**概述**”页复制）。
1. 替换占位符后，使用 **Ctrl+S** 命令保存更改，然后使用 **Ctrl+Q** 命令关闭代码编辑器，同时使 Cloud Shell 命令行保持打开状态。

## 添加代码以使用 Azure AI 语音 SDK

> **提示**：添加代码时，请务必保持正确的缩进。

1. 输入以下命令以编辑已提供的代码文件：

    **Python**

    ```
   code speaking-clock.py
    ```

    **C#**

    ```
   code Program.cs
    ```

1. 在代码文件顶部的现有命名空间引用下，找到注释“**导入命名空间**”。 然后，在此注释下，添加以下特定语言的代码，以导入将 Azure AI 语音 SDK 与 Azure AI Foundry 项目中的 Azure AI 服务资源一起使用时所需的命名空间：

    **Python**

    ```python
   # Import namespaces
   from dotenv import load_dotenv
   from azure.ai.projects.models import ConnectionType
   from azure.identity import DefaultAzureCredential
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.projects import AIProjectClient
   import azure.cognitiveservices.speech as speech_sdk
    ```

    **C#**

    ```csharp
   // Import namespaces
   using Azure.Identity;
   using Azure.AI.Projects;
   using Microsoft.CognitiveServices.Speech;
   using Microsoft.CognitiveServices.Speech.Audio;
    ```

1. 在 **main** 函数的注释“**获取配置设置**”下，请注意，代码将加载配置文件中定义的项目连接字符串和位置。

1. 在注释“**从项目获取 AI 语音终结点和密钥**”下，添加以下代码：

    **Python**

    ```python
   # Get AI Services key from the project
   project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())

   ai_svc_connection = project_client.connections.get_default(
      connection_type=ConnectionType.AZURE_AI_SERVICES,
      include_credentials=True, 
    )

   ai_svc_key = ai_svc_connection.key

    ```

    **C#**

    ```csharp
   // Get AI Services key from the project
   var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());

   ConnectionResponse aiSvcConnection = projectClient.GetConnectionsClient().GetDefaultConnection(ConnectionType.AzureAIServices, true);

   var apiKeyAuthProperties = aiSvcConnection.Properties as ConnectionPropertiesApiKeyAuth;

   var aiSvcKey = apiKeyAuthProperties.Credentials.Key;
    ```

    此代码连接到 Azure AI Foundry 项目，获取其默认的 AI 服务连接资源，并检索使用它所需的身份验证密钥。

1. 在注释“**配置语音服务**”下，添加以下代码以使用 AI 服务密钥和项目区域，配置与 Azure AI 服务语音终结点的连接

   **Python**

    ```python
   # Configure speech service
   speech_config = speech_sdk.SpeechConfig(ai_svc_key, location)
   print('Ready to use speech service in:', speech_config.region)
    ```

    **C#**

    ```csharp
   // Configure speech service
   speechConfig = SpeechConfig.FromSubscription(aiSvcKey, location);
   Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    ```

1. 保存更改 (*CTRL+S*)，但使代码编辑器保持打开状态。

## 运行应用程序

到目前为止，应用除了连接到 Azure AI Foundry 项目来检索使用语音服务所需的详细信息之外，不执行任何操作，但运行该应用并检查其在添加语音功能之前是否正常工作非常有用。

1. 在代码编辑器下面的命令行中，输入以下 Azure CLI 命令来确定会话登录的 Azure 帐户：

    ```
   az account show
    ```

    生成的 JSON 输出应包含 Azure 帐户的详细信息和正在使用的订阅（应与在其中创建了 Azure AI Foundry 项目的订阅相同）。

    应用将 Azure 凭据用于运行该凭据的上下文，以验证与项目的连接。 在生产环境中，可以将应用配置为使用托管标识运行。 在此开发环境中，它将使用经过验证的 Cloud Shell 会话凭据。

    > **备注**：可以使用 `az login` Azure CLI 命令登录到开发环境中的 Azure。 在这种情况下，Cloud Shell 已使用登录到门户的 Azure 凭据登录；因此，不需要显式登录。 要详细了解如何使用 Azure CLI 对 Azure 进行验证，请参阅[使用 Azure CLI 对 Azure 进行验证](https://learn.microsoft.com/cli/azure/authenticate-azure-cli)。

1. 在命令行中，输入以下特定语言的命令以运行语音时钟应用：

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. 如果你使用 C#，则可以忽略关于在异步方法中使用 await 运算符的任何警告 - 我们之后将修复此问题。 代码应显示应用程序将使用的语音服务资源的区域。 成功运行表示应用已连接到 Azure AI Foundry 项目，并检索了它使用 Azure AI 语音服务所需的密钥。

## 添加代码以识别语音

为项目的 Azure AI 服务资源中的语音服务添加了 **SpeechConfig** 之后，现在，你可以使用“**语音转换为文本**” API 来识别语音并将其转录为文本。

在此过程中，语音输入是从音频文件捕获的，可以在此处播放：

<video controls src="ai-foundry/media/Time.mp4" title="几点了？" width="150"></video>

1. 在 Main 函数中，请注意代码使用 TranscribeCommand 函数来接受语音输入 。 然后，在 TranscribeCommand 函数的注释“配置语音识别”下，添加以下适当代码以创建 SpeechRecognizer 客户端，该客户端可用于识别和转录音频文件中的语音  ：

    **Python**

    ```python
   # Configure speech recognition
   current_dir = os.getcwd()
   audioFile = current_dir + '/time.wav'
   audio_config = speech_sdk.AudioConfig(filename=audioFile)
   speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

    **C#**

    ```csharp
   // Configure speech recognition
   string audioFile = "time.wav";
   using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
   using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

1. 在 TranscribeCommand 函数的注释“处理语音输入”下，添加以下代码以侦听语音输入，注意不要替换返回命令的函数末尾的代码 ：

    **Python**

    ```python
   # Process speech input
   print("Listening...")
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

    **C#**

    ```csharp
   // Process speech input
   Console.WriteLine("Listening...");
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

1. 保存更改 (*Ctrl+S*)，然后在代码编辑器下面的命令行中输入以下命令来运行程序：

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. 查看应用程序的输出，该输出应在音频文件中成功“听到”语音并返回适当的响应（请注意，Azure Cloud Shell 可能在与你的时区不同的服务器上运行！)

    > **提示**：如果 SpeechRecognizer 遇到错误，它将生成“已取消”结果。 然后，应用程序中的代码将显示错误消息。 最可能的原因是配置文件中的区域值不正确。

## 合成语音

语音时钟应用程序接受语音输入，但它实际上并不生成语音输出！ 让我们通过添加代码以合成语音来解决此问题。

再次，由于 Cloud Shell 的硬件限制，我们将合成的语音输出定向到文件。

1. 在程序的 Main 函数中，请注意代码使用 TellTime 函数来告知用户当前时间 。
1. 在 TellTime 函数的注释“配置语音合成”下，添加以下代码以创建可用于生成语音输出的 SpeechSynthesizer 客户端  ：

    **Python**

    ```python
   # Configure speech synthesis
   output_file = "output.wav"
   speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
   audio_config = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config,)
    ```

    **C#**

    ```csharp
   // Configure speech synthesis
   var outputFile = "output.wav";
   speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
   using var audioConfig = AudioConfig.FromWavFileOutput(outputFile);
   using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig, audioConfig);
    ```

1. 在 TellTime 函数的注释“合成语音输出”下，添加以下代码以生成语音输出，注意不要替换输出响应的函数末尾的代码 ：

    **Python**

    ```python
   # Synthesize spoken output
   speak = speech_synthesizer.speak_text_async(response_text).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
       print(speak.reason)
   else:
       print("Spoken output saved in " + outputFile)
    ```

    **C#**

    ```csharp
   // Synthesize spoken output
   SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
   if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
   {
       Console.WriteLine(speak.Reason);
   }
   else
   {
       Console.WriteLine("Spoken output saved in " + outputFile);
   }
    ```

1. 保存更改 (*Ctrl+S*)，然后在代码编辑器下面的命令行中输入以下命令来运行程序：

   **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. 查看应用程序的输出，该输出应指示语音输出已保存在文件中。
1. 如果媒体播放器能够播放.wav 音频文件，请在 Cloud Shell 窗格的工具栏中，使用“**上传/下载文件**”按钮从应用文件夹下载音频文件，然后播放：

    **Python**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/Python/speaking-clock/output.wav`

    **C#**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/C-Sharp/speaking-clock/output.wav`

    该文件的声音应类似于此：

    <video controls src="./ai-foundry/media/Output.mp4" title="时间是 2:15" width="150"></video>

## 使用语音合成标记语言

语音合成标记语言 (SSML) 使你能够自定义使用基于 XML 的格式合成语音的方式。

1. 在 TellTime 函数中，将注释“合成语音输出”下的所有当前代码替换为以下代码（保留注释“输出响应”下的代码）  ：

    **Python**

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
   else:
       print("Spoken output saved in " + outputFile)
    ```

   **C#**

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
   else
   {
        Console.WriteLine("Spoken output saved in " + outputFile);
   }
    ```

1. 保存你的更改并返回到 speaking-clock 文件夹的集成终端，然后输入以下命令以运行程序：

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. 查看应用程序的输出，该输出应指示语音输出已保存在文件中。
1. 再次，如果媒体播放器能够播放.wav 音频文件，请在 Cloud Shell 窗格的工具栏中，使用“**上传/下载文件**”按钮从应用文件夹下载音频文件，然后播放：

    **Python**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/Python/speaking-clock/output.wav`

    **C#**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/C-Sharp/speaking-clock/output.wav`

    该文件的声音应类似于此：
    
    <video controls src="./ai-foundry/media/Output2.mp4" title="时间是 5:30。 结束此实验室的时间。" width="150"></video>

## 清理

如果已完成 Azure AI 语音的探索，则应删除在本练习中创建的资源，以避免产生不必要的 Azure 成本。

1. 返回到包含 Azure 门户的浏览器选项卡（或在新的浏览器选项卡中重新打开 [Azure 门户](https://portal.azure.com)，网址为：`https://portal.azure.com`），查看已在其中部署本练习中使用的资源的资源组内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。

## 如果你有麦克风和扬声器，该怎么办？

在此练习中，使用了音频文件作为语音输入和输出。 让我们看看如何修改代码以使用音频硬件。

### 使用麦克风进行语音识别

如果你有麦克风，可以使用以下代码捕获语音输入进行语音识别：

**Python**

```python
# Configure speech recognition
audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
print('Speak now...')

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

**C#**

```csharp
// Configure speech recognition
using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
Console.WriteLine("Speak now...");

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

> **备注**：系统默认麦克风是默认音频输入，因此还可以完全省略 AudioConfig！

### 将语音合成与扬声器配合使用

如果有扬声器，可以使用以下代码合成语音。

**Python**

```python
response_text = 'The time is {}:{:02d}'.format(now.hour,now.minute)

# Configure speech synthesis
speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
audio_config = speech_sdk.audio.AudioOutputConfig(use_default_speaker=True)
speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config)

# Synthesize spoken output
speak = speech_synthesizer.speak_text_async(response_text).get()
if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
    print(speak.reason)
```

**C#**

```csharp
var now = DateTime.Now;
string responseText = "The time is " + now.Hour.ToString() + ":" + now.Minute.ToString("D2");

// Configure speech synthesis
speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
using var audioConfig = AudioConfig.FromDefaultSpeakerOutput();
using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig, audioConfig);

// Synthesize spoken output
SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
{
    Console.WriteLine(speak.Reason);
}
```

> **备注**：系统默认扬声器是默认音频输出，因此还可以完全省略 AudioConfig！

## 详细信息

有关如何使用语音转文本 API 和文本转语音 API 的详细信息，请参阅[语音转文本文档](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text)和[文本转语音文档](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech) 。
