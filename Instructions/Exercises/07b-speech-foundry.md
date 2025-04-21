---
lab:
  title: Reconocimiento y síntesis de voz (versión de Fundición de IA de Azure)
  module: Module 4 - Create speech-enabled apps with Azure AI services
---

<!--
Possibly update to use standalone AI Service instead of Foundry?
-->

# Reconocimiento y síntesis de voz

**Voz de Azure AI** es un servicio que proporciona la funcionalidad relacionada con la voz, que incluye lo siguiente:

- Una API de *conversión de voz a texto* que permite implementar el reconocimiento de voz (conversión de texto oral audible en texto).
- Una API de *conversión de texto a voz* que permite implementar la síntesis de voz (conversión de texto en voz audible).

En este ejercicio, usará estas dos API para implementar una aplicación de reloj de voz.

> **NOTA** Este ejercicio está diseñado para completarse en Azure Cloud Shell, donde no se admite el acceso directo al hardware de sonido del equipo. Por lo tanto, el laboratorio usará archivos de audio para flujos de entrada y salida de voz. El código para lograr los mismos resultados con un micrófono y un altavoz se proporciona como referencia.

## Creación de un proyecto de Fundición de IA de Azure

Comencemos creando un proyecto de Fundición de IA de Azure.

1. En un explorador web, abre el [Portal de la Fundición de IA de Azure](https://ai.azure.com) en `https://ai.azure.com` e inicia sesión con tus credenciales de Azure. Cierra las sugerencias o paneles de inicio rápido que se abran la primera vez que inicias sesión y, si es necesario, usa el logotipo de **Fundición de IA de Azure** en la parte superior izquierda para navegar a la página principal, que es similar a la siguiente imagen:

    ![Captura de pantalla del Portal de la Fundición de IA de Azure.](./ai-foundry/media/ai-foundry-home.png)

1. En la página principal, selecciona **+Crear proyecto**.
1. En el asistente para **crear un proyecto**, escribe un nombre válido y si se te sugiere un centro existente, elige la opción para crear uno nuevo. A continuación, revisa los recursos de Azure que se crearán automáticamente para admitir el centro y el proyecto.
1. Selecciona **Personalizar** y especifica la siguiente configuración para el centro:
    - **Nombre del centro**: *proporciona un nombre para el centro*.
    - **Suscripción**: *suscripción a Azure*
    - **Grupo de recursos**: *crea o selecciona un grupo de recursos*.
    - **Ubicación**: elige cualquier región disponible
    - **Conectar Servicios de Azure AI o Azure OpenAI**: *Crear un nuevo servicio de IA*
    - **Conexión de Búsqueda de Azure AI**: *crea de un nuevo recurso de Búsqueda de Azure AI con un nombre único*

1. Selecciona **Siguiente** y revisa tu configuración. Luego, selecciona **Crear** y espera a que se complete el proceso.
1. Cuando se cree el proyecto, cierra las sugerencias que se muestran y revisa la página del proyecto en el Portal de la Fundición de IA de Azure, que debe tener un aspecto similar a la siguiente imagen:

    ![Captura de pantalla de los detalles de un proyecto de Azure AI en el Portal de la Fundición de IA de Azure.](./ai-foundry/media/ai-foundry-project.png)

## Preparación y configuración de la aplicación del reloj que habla

1. En el Portal de la Fundición de IA de Azure, mira la página **Información general** del proyecto.
1. En el área **Detalles del proyecto**, anota la **Cadena de conexión del proyecto** y la **ubicación** de tu proyecto. Usarás la cadena de conexión para conectarte a tu proyecto en una aplicación cliente, y necesitarás la ubicación para conectarte al punto de conexión de Servicios de Azure AI Speech.
1. Abre una nueva pestaña del explorador (mantén el Portal de la Fundición de IA de Azure abierto en la pestaña existente). En la nueva pestaña, explora [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` e inicia sesión con tus credenciales de Azure, si se te solicita.
1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    > **Sugerencia**: al pegar comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

1. En el panel de PowerShell, escribe los siguientes comandos para clonar el repo de GitHub para este ejercicio:

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language mslearn-ai-language
    ```

    ***Ahora sigue los pasos del lenguaje de programación que hayas elegido.***

1. Una vez clonado el repositorio, ve a la carpeta que contiene los archivos de código de la aplicación del reloj que habla:  

    **Python**

    ```
   cd mslearn-ai-language/labfiles/07b-speech/python/speaking-clock
    ```

    **C#**

    ```
   cd mslearn-ai-language/labfiles/07b-speech/c-sharp/speaking-clock
    ```

1. En el panel de la línea de comandos de Cloud Shell, escribe el siguiente comando para instalar las bibliotecas que vas a usar:

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

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, reemplaza los marcadores de posición **your_project_endpoint** y **your_location** por la cadena de conexión y la ubicación de tu proyecto (copiadas de la página **Información general** del proyecto del Portal de la Fundición de IA de Azure).
1. Después de reemplazar los marcadores de posición, usa el comando **CTRL+S** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

## Agregue código para usar el SDK de Voz de Azure AI

> **Sugerencia**: al agregar código, asegúrate de mantener la sangría correcta.

1. Escribe el siguiente comando para editar el archivo de código que se ha proporcionado:

    **Python**

    ```
   code speaking-clock.py
    ```

    **C#**

    ```
   code Program.cs
    ```

1. En la parte superior del archivo de código, en las referencias de espacios de nombres existentes, busque el comentario **Import namespaces**. Después, en este comentario, agrega el siguiente código específico del idioma para importar los espacios de nombres que necesitarás para usar el SDK de Voz de Azure AI con el recurso Servicios de Azure AI en tu proyecto de Fundición de IA de Azure:

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

1. En la función **principal**, en el comentario **Obtener configuración**, observa que el código carga la cadena de conexión del proyecto y la ubicación que has definido en el archivo de configuración.

1. Agrega el siguiente código en el comentario **Obtener punto de conexión y clave de Voz de AI del proyecto**:

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

    Este código se conecta al proyecto de Fundición de IA de Azure, obtiene su recurso conectado predeterminado de Servicios de IA y recupera la clave de autenticación necesaria para usarlo.

1. En el comentario **Configurar servicio de voz**, agrega el siguiente código para usar la clave de Servicios de IA y la región de tu proyecto para configurar tu conexión con el punto de conexión de Voz de Servicios de Azure AI

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

1. Guarda los cambios (*CTRL+S*), pero deje abierto el editor de código.

## Ejecución de la aplicación

Por ahora, la aplicación no hace nada más que conectarse al proyecto de Fundición de IA de Azure para recuperar los detalles necesarios para usar el servicio Voz, pero resulta útil ejecutarlo y comprobar que funciona antes de agregar la funcionalidad de voz.

1. En la línea de comandos debajo del editor de código, escribe el siguiente comando de la CLI de Azure para determinar la cuenta de Azure que ha iniciado sesión para la sesión:

    ```
   az account show
    ```

    La salida JSON resultante debe incluir detalles de la cuenta de Azure y la suscripción en la que estás trabajando (que debe ser la misma suscripción en la que has creado el proyecto de Fundición de IA de Azure).

    La aplicación usa las credenciales de Azure para el contexto en el que se ejecuta para autenticar la conexión al proyecto. En un entorno de producción, la aplicación puede configurarse para ejecutarse mediante una identidad administrada. En este entorno de desarrollo, usará las credenciales de sesión de Cloud Shell autenticadas.

    > **Nota**: Puedes iniciar sesión en Azure en el entorno de desarrollo mediante el comando de la CLI de Azure `az login`. En este caso, Cloud Shell ya ha iniciado sesión con las credenciales de Azure con las que has iniciado sesión en el portal; por lo tanto, iniciar sesión explícitamente no es necesario. Para obtener más información sobre cómo usar la CLI de Azure para autenticarte en Azure, consulta [Autenticarse en Azure mediante la CLI de Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli).

1. En la línea de comandos, escribe el siguiente comando específico del idioma para ejecutar la aplicación de reloj que habla:

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Si usa C#, puede omitir las advertencias sobre el uso del operador **await** en métodos asincrónicos; lo corregiremos más adelante. El código debe mostrar la región del recurso del servicio de voz que usará la aplicación. Una ejecución correcta indica que la aplicación se ha conectado a tu proyecto de Fundición de IA de Azure y ha recuperado la clave que necesita para usar el servicio Voz de Azure AI.

## Agregar código para reconocer voz

Ahora que tienes una **SpeechConfig** para el servicio de voz en tu recurso de Servicios de Azure AI del proyecto, puedes usar la API **Speech-to-text** para reconocer la voz y transcribirla a texto.

En este procedimiento, la entrada de voz se captura desde un archivo de audio, que puedes reproducir aquí:

<video controls src="ai-foundry/media/Time.mp4" title="¿Qué hora tenemos?" width="150"></video>

1. En la función **Main**, observe que el código usa la función **TranscribeCommand** para aceptar la entrada hablada. A continuación, en la función **TranscribeCommand**, en el comentario **Configurar Reconocimiento de voz**, agregue el código adecuado siguiente para crear un cliente **SpeechRecognizer** que se pueda usar para reconocer y transcribir la voz de un archivo de audio:

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

1. En la función **TranscribeCommand**, en el comentario **Process speech input** (Procesar entrada de voz), agregue el código siguiente para escuchar la entrada hablada, y tenga cuidado de no reemplazar el código al final de la función que devuelve el comando:

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

1. Guarda los cambios (*CTRL+S*) y después, en la línea de comandos situada debajo del editor de código, escribe el siguiente comando para ejecutar el programa:

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Revisa la salida de la aplicación, que debería "escuchar" correctamente la voz en el archivo de audio y devolver una respuesta adecuada (ten en cuenta que Azure Cloud Shell puede estar ejecutándose en un servidor que se encuentra en una zona horaria diferente a la tuya).

    > **Consejo**: Si SpeechRecognizer encuentra un error, genera el resultado "Cancelado". A continuación, el código de la aplicación mostrará el mensaje de error. La causa más probable es un valor de región incorrecto en el archivo de configuración.

## Sintetizar voz

La aplicación de reloj de voz acepta la entrada hablada, pero en realidad no habla. Vamos a corregirlo agregando código para sintetizar la voz.

Una vez más, debido a las limitaciones de hardware de Cloud Shell, dirigiremos la salida de voz sintetizada a un archivo.

1. En la función **Main** del programa, observe que el código usa la función **TellTime** para decir al usuario la hora actual.
1. En la función **TellTime**, en el comentario **Configure speech synthesis** (Configurar síntesis de voz), agregue el código siguiente para crear un cliente **SpeechSynthesizer** que se pueda usar para generar salida de voz:

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

1. En la función **TellTime**, en el comentario **Synthesize spoken output** (Sintetizar salida de voz), agregue el código siguiente para generar la salida hablada y tenga cuidado de no reemplazar el código al final de la función que imprime la respuesta:

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

1. Guarda los cambios (*CTRL+S*) y después, en la línea de comandos situada debajo del editor de código, escribe el siguiente comando para ejecutar el programa:

   **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Revisa la salida de la aplicación, que debe indicar que la salida hablada se guardó en un archivo.
1. Si tienes un reproductor multimedia capaz de reproducir archivos de audio .wav, en la barra de herramientas del panel de Cloud Shell, usa el botón **Cargar/descargar archivos** para descargar el archivo de audio de la carpeta de la aplicación y, a continuación, reproducirlo:

    **Python**

    /inicio/*usuario*`/mslearn-ai-language/Labfiles/07b-speech/Python/speaking-clock/output.wav`

    **C#**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/C-Sharp/speaking-clock/output.wav`

    El archivo debe sonar de forma similar a:

    <video controls src="./ai-foundry/media/Output.mp4" title="La hora es 2:15" width="150"></video>

## Uso de Lenguaje de marcado de síntesis de voz

El lenguaje de marcado de síntesis de voz (SSML) permite personalizar la forma en que se sintetiza la voz mediante un formato basado en XML.

1. En la función **TellTime**, reemplace todo el código actual en el comentario **Synthesize spoken output** (Sintentizar salida de voz) por el código siguiente (deje el código del comentario **Print the response** [Imprimir la respuesta]):

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

1. Guarde los cambios y vuelva al terminal integrado de la carpeta **speaking-clock** y escriba el siguiente comando para ejecutar el programa:

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Revisa la salida de la aplicación, que debe indicar que la salida hablada se guardó en un archivo.
1. Una vez más, si tienes un reproductor multimedia capaz de reproducir archivos de audio .wav, en la barra de herramientas del panel de Cloud Shell, usa el botón **Cargar/descargar archivos** para descargar el archivo de audio de la carpeta de la aplicación y, a continuación, reproducirlo:

    **Python**

    /inicio/*usuario*`/mslearn-ai-language/Labfiles/07b-speech/Python/speaking-clock/output.wav`

    **C#**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/C-Sharp/speaking-clock/output.wav`

    El archivo debe sonar de forma similar a:
    
    <video controls src="./ai-foundry/media/Output2.mp4" title="La hora es 5:30. Es hora de finalizar este laboratorio." width="150"></video>

## Limpieza

Si has terminado de explorar Voz de Azure AI, debes eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. Vuelve a la pestaña del explorador que contiene Azure Portal (o vuelve a abrir [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` en una nueva pestaña del explorador) y mira el contenido del grupo de recursos donde implementó los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.

## ¿Qué ocurre si tienes un micrófono y un altavoz?

En este ejercicio, usarás un archivo de audio como entrada de voz. Veamos cómo se puede modificar el código para usar hardware de audio.

### Uso del reconocimiento de voz con un micrófono

Si tienes un micrófono, puedes usar el código siguiente para capturar la entrada hablada para el reconocimiento de voz:

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

> **Nota**: El micrófono predeterminado del sistema es la entrada de audio predeterminada, por lo que también podrías omitir audioConfig en conjunto.

### Uso de la síntesis de voz con un altavoz

Si tienes un altavoz, puedes usar el código siguiente para sintetizar la voz.

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

> **Nota**: El altavoz predeterminado del sistema es la salida de audio predeterminada, por lo que también podrías omitir audioConfig en conjunto.

## Más información

Para más información sobre el uso de las API de **conversión de voz en texto** y de **conversión de texto a voz**, consulte la [documentación de conversión de voz en texto](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text) y la [documentación de conversión de texto a voz](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech).
