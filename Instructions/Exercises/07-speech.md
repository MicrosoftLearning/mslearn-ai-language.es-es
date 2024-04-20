---
lab:
  title: Reconocimiento y síntesis de voz
  module: Module 4 - Create speech-enabled apps with Azure AI services
---

# Reconocimiento y síntesis de voz

**Voz de Azure AI** es un servicio que proporciona la funcionalidad relacionada con la voz, que incluye lo siguiente:

- Una API de *conversión de voz a texto* que permite implementar el reconocimiento de voz (conversión de texto oral audible en texto).
- Una API de *conversión de texto a voz* que permite implementar la síntesis de voz (conversión de texto en voz audible).

En este ejercicio, usará estas dos API para implementar una aplicación de reloj de voz.

> **Nota**: Este ejercicio requiere que use un equipo con altavoces o auriculares. Para obtener la mejor experiencia, también se requiere un micrófono. Algunos entornos virtuales hospedados pueden capturar audio del micrófono local, pero si esto no funciona (o no tiene micrófono), puede usar un archivo de audio proporcionado para la entrada de voz. Siga las instrucciones detenidamente, ya que deberá elegir diferentes opciones en función de si usa un micrófono o el archivo de audio.

## Aprovisionar un recurso de *Voz de Azure AI*

Si aún no tiene uno en su suscripción, deberá aprovisionar un recurso de **Voz de Azure AI**.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
1. En el campo de búsqueda de la parte superior, busque **Servicios de Azure AI** y presione **Entrar** y, a continuación, seleccione **Crear** en **Servicio de Voz** en los resultados.
1. Cree un recurso con los valores siguientes:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos*
    - **Región**: *elija cualquier región disponible*
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa**: seleccione **F0** (*gratis*), o **S** (*estándar*) si F no está disponible.
    - **Aviso de IA responsable**: Aceptar.
1. Seleccione **Revisar y crear** y **Crear** para aprovisionar el recurso.
1. Espere a que se complete la implementación y, a continuación, vaya al recurso implementado.
1. Consulte la página **Claves y punto de conexión**. Necesitará la información de esta página más adelante en el ejercicio.

## Preparación para desarrollar una aplicación en Visual Studio Code

Desarrollará la aplicación de voz mediante Visual Studio Code. Los archivos de código de la aplicación se han proporcionado en un repositorio de GitHub.

> **Sugerencia**: Si ya ha clonado el repositorio **mslearn-ai-language**, ábralo en Visual Studio Code. De lo contrario, siga estos pasos para clonarlo en el entorno de desarrollo.

1. Inicie Visual Studio Code.
1. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/mslearn-ai-language` en una carpeta local (no importa qué carpeta).
1. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
1. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## Configuración de la aplicación

Se han proporcionado aplicaciones para C# y Python. Las dos aplicaciones tienen la misma funcionalidad. Primero, completará algunas partes clave de la aplicación para habilitar que utilice el recurso de Voz de Azure AI.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **Labfiles/07-speech** y expanda la carpeta **CSharp** o **Python** según sus preferencias de lenguaje y la carpeta **speaking-clock** que contiene. Cada carpeta contiene los archivos de código específicos del lenguaje de una aplicación en la que va a integrar la funcionalidad Voz de Azure AI.
1. Haga clic con el botón derecho en la carpeta **speaking-clock** que contiene los archivos de código y abra un terminal integrado. A continuación, instale el paquete del SDK de Voz de Azure AI mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.30.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.30.0
    ```

1. En el panel **Explorador**, en la carpeta **speaking-clock**, abra el archivo de configuración para su lenguaje preferido

    - **C#**: appsettings.json
    - **Python**: .env

1. Actualice los valores de configuración para incluir la **región** y una **clave** del recurso de Voz de Azure AI que creó (disponible en la página **Claves y punto de conexión** para el recurso de Voz de Azure AI en Azure Portal).

    > **NOTA**: Asegúrese de agregar la *región* del recurso, <u>no</u> el punto de conexión.

1. Guarde el archivo de configuración.

## Agregue código para usar el SDK de Voz de Azure AI

1. Tenga en cuenta que la carpeta **speaking-clock** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: speaking-clock.py

    Abra el archivo de código y, en la parte superior, en las referencias de espacio de nombres existentes, busque el comentario **Importar espacios de nombres**. A continuación, en este comentario, agregue el siguiente código específico según el lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Voz de Azure AI:

    **C#** : Program.cs

    ```csharp
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    ```

    **Python**: speaking-clock.py

    ```python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

1. En la función **Main (Principal)**, fíjese en que ya se ha proporcionado código para cargar la clave del servicio y la región del archivo de configuración. Debe usar estas variables para crear **SpeechConfig** para el recurso de Voz de Azure AI. Agregue el código siguiente en el comentario **Configure speech service** (Configurar el servicio de voz):

    **C#** : Program.cs

    ```csharp
    // Configure speech service
    speechConfig = SpeechConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    
    // Configure voice
    speechConfig.SpeechSynthesisVoiceName = "en-US-AriaNeural";
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech service
    speech_config = speech_sdk.SpeechConfig(ai_key, ai_region)
    print('Ready to use speech service in:', speech_config.region)
    ```

1. Guarde los cambios y vuelva al terminal integrado de la carpeta **speaking-clock** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Si usa C#, puede omitir las advertencias sobre el uso del operador **await** en métodos asincrónicos; lo corregiremos más adelante. El código debe mostrar la región del recurso del servicio de voz que usará la aplicación.

## Agregar código para reconocer voz

Ahora que tiene **SpeechConfig** para el servicio de voz en el recurso de Voz de Azure AI, puede usar la API de **Speech-to-text** para reconocer voz y transcribirla a texto.

> **IMPORTANTE**: En esta sección se incluyen instrucciones para dos procedimientos alternativos. Siga el primer procedimiento si tiene un micrófono que funcione. Siga el segundo procedimiento si desea simular la entrada hablada mediante un archivo de audio.

### Si tiene un micrófono que funcione

1. En la función **Main** del programa, observe que el código usa la función **TranscribeCommand** para aceptar la entrada hablada.
1. En la función **TranscribeCommand**, en el comentario **Configurar Reconocimiento de voz**, agregue el código adecuado siguiente para crear un cliente **SpeechRecognizer** que se pueda usar para reconocer y transcribir la voz mediante el micrófono predeterminado del sistema:

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

1. Ahora vaya directamente a la sección **Adición de código para procesar el comando transcrito** a continuación.

---

### Como alternativa, use la entrada de audio de un archivo

1. En la ventana de terminal, escriba el siguiente comando para instalar una biblioteca que pueda usar para reproducir el archivo de audio:

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

1. En el archivo de código del programa, en las importaciones del espacio de nombres existente, agregue el código siguiente para importar la biblioteca que acaba de instalar:

    **C#** : Program.cs

    ```csharp
    using System.Media;
    ```

    **Python**: speaking-clock.py

    ```python
    from playsound import playsound
    ```

1. En la función **Main**, observe que el código usa la función **TranscribeCommand** para aceptar la entrada hablada. A continuación, en la función **TranscribeCommand**, en el comentario **Configurar Reconocimiento de voz**, agregue el código adecuado siguiente para crear un cliente **SpeechRecognizer** que se pueda usar para reconocer y transcribir la voz de un archivo de audio:

    **C#** : Program.cs

    ```csharp
    // Configure speech recognition
    string audioFile = "time.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech recognition
    current_dir = os.getcwd()
    audioFile = current_dir + '\\time.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

---

### Adición de código para procesar el comando transcrito

1. En la función **TranscribeCommand**, en el comentario **Process speech input** (Procesar entrada de voz), agregue el código siguiente para escuchar la entrada hablada, y tenga cuidado de no reemplazar el código al final de la función que devuelve el comando:

    **C#** : Program.cs

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

    **Python**: speaking-clock.py

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

1. Guarde los cambios y vuelva al terminal integrado de la carpeta **speaking-clock** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Si usa un micrófono, hable con claridad y diga "¿qué hora es?". El programa debe transcribir la entrada hablada y mostrar la hora (en función de la hora local del equipo donde se ejecuta el código, que puede que no sea la hora correcta donde se encuentra).

    SpeechRecognizer proporciona unos 5 segundos para hablar. Si no detecta ninguna entrada hablada, genera un resultado "Sin coincidencia".

    Si SpeechRecognizer encuentra un error, genera el resultado "Cancelado". A continuación, el código de la aplicación mostrará el mensaje de error. La causa más probable es una clave o región incorrectas en el archivo de configuración.

## Sintetizar voz

La aplicación de reloj de voz acepta la entrada hablada, pero en realidad no habla. Vamos a corregirlo agregando código para sintetizar la voz.

1. En la función **Main** del programa, observe que el código usa la función **TellTime** para decir al usuario la hora actual.
1. En la función **TellTime**, en el comentario **Configure speech synthesis** (Configurar síntesis de voz), agregue el código siguiente para crear un cliente **SpeechSynthesizer** que se pueda usar para generar salida de voz:

    **C#** : Program.cs

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

    > **NOTA** La configuración de audio predeterminada usa el dispositivo de audio del sistema predeterminado para la salida, así que no es necesario proporcionar explícitamente **AudioConfig**. Si necesita redirigir la salida de audio a un archivo, puede usar **AudioConfig** con una ruta de acceso de archivo para hacerlo.

1. En la función **TellTime**, en el comentario **Synthesize spoken output** (Sintetizar salida de voz), agregue el código siguiente para generar la salida hablada y tenga cuidado de no reemplazar el código al final de la función que imprime la respuesta:

    **C#** : Program.cs

    ```csharp
    // Synthesize spoken output
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python**: speaking-clock.py

    ```python
    # Synthesize spoken output
    speak = speech_synthesizer.speak_text_async(response_text).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. Guarde los cambios y vuelva al terminal integrado de la carpeta **speaking-clock** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Cuando se le solicite, hable con claridad hacia el micrófono y diga "¿qué hora es?". El programa debe hablar, indicando la hora.

## Uso de una voz distinta

La aplicación de reloj de voz usa una voz predeterminada, que se puede cambiar. El servicio Voz admite una variedad de voces *estándar*, así como voces *neuronales* más parecidas a las humanas. También puede crear voces *personalizadas*.

> **Nota**: Para obtener una lista de voces neuronales y estándar, consulte [Galería de voz](https://speech.microsoft.com/portal/voicegallery) en Speech Studio.

1. En la función **TellTime**, en el comentario **Configure speech synthesis** (Configurar síntesis de voz), modifique el código tal como se indica a continuación para especificar una voz alternativa antes de crear un cliente **SpeechSynthesizer**:

   **C#** : Program.cs

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-LibbyNeural"; // change this
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python**: speaking-clock.py

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = 'en-GB-LibbyNeural' # change this
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

1. Guarde los cambios y vuelva al terminal integrado de la carpeta **speaking-clock** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Cuando se le solicite, hable con claridad hacia el micrófono y diga "¿qué hora es?". El programa debe hablar en la voz especificada, indicando la hora.

## Uso de Lenguaje de marcado de síntesis de voz

El lenguaje de marcado de síntesis de voz (SSML) permite personalizar la forma en que se sintetiza la voz mediante un formato basado en XML.

1. En la función **TellTime**, reemplace todo el código actual en el comentario **Synthesize spoken output** (Sintentizar salida de voz) por el código siguiente (deje el código del comentario **Print the response** [Imprimir la respuesta]):

   **C#** : Program.cs

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

    **Python**: speaking-clock.py

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

1. Guarde los cambios y vuelva al terminal integrado de la carpeta **speaking-clock** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Cuando se le solicite, hable con claridad hacia el micrófono y diga "¿qué hora es?". El programa debe hablar con la voz especificada en el SSML (reemplazando a la voz especificada en SpeechConfig), indicando la hora y, después de una pausa, le indicará que es el momento de finalizar este laboratorio, que lo es.

## Información adicional

Para más información sobre el uso de las API de **conversión de voz en texto** y de **conversión de texto a voz**, consulte la [documentación de conversión de voz en texto](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text) y la [documentación de conversión de texto a voz](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech).
