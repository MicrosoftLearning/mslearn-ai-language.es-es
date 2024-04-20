---
lab:
  title: Traducir voz
  module: Module 8 - Translate speech with Azure AI Speech
---

# Traducir voz

Voz de Azure AI incluye una API Speech Translation que puede usar para traducir el lenguaje hablado. Por ejemplo, supongamos que quiere desarrollar una aplicación de traductor que los usuarios puedan usar al viajar a lugares donde no hablan el idioma local. Podrían decir frases como "¿Dónde está la estación?" o "Necesito encontrar un restaurante" en su propio idioma y hacer que los traduzca al idioma local.

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
1. Seleccione **Revisar y crear** y, después, **Crear** para aprovisionar el recurso.
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

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **Labfiles/08-speech-translation** y expanda la carpeta **CSharp** o **Python** según sus preferencias de lenguaje y la carpeta **Traductor** que contiene. Cada carpeta contiene los archivos de código específicos del lenguaje de una aplicación en la que va a integrar la funcionalidad Voz de Azure AI.
1. Haga clic con el botón derecho en la carpeta **Traductor** que contiene los archivos de código y abra un terminal integrado. A continuación, instale el paquete del SDK de Voz de Azure AI mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.30.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.30.0
    ```

1. En el panel **Explorador**, en la carpeta **Traductor**, abra el archivo de configuración para su idioma preferido.

    - **C#**: appsettings.json
    - **Python**: .env

1. Actualice los valores de configuración para incluir la **región** y una **clave** del recurso de Voz de Azure AI que creó (disponible en la página **Claves y punto de conexión** para el recurso de Voz de Azure AI en Azure Portal).

    > **NOTA**: Asegúrese de agregar la *región* del recurso, <u>no</u> el punto de conexión.

1. Guarde el archivo de configuración.

## Agregar código para usar el SDK de Voz

1. Tenga en cuenta que la carpeta **translator** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: translator.py

    Abra el archivo de código y, en la parte superior, en las referencias de espacio de nombres existentes, busque el comentario **Importar espacios de nombres**. A continuación, en este comentario, agregue el siguiente código específico según el lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Voz de Azure AI:

    **C#** : Program.cs

    ```csharp
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Translation;
    ```

    **Python**: translator.py

    ```python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

1. En la función **Main (Principal)**, fíjese en que ya se ha proporcionado código para cargar la clave del servicio Voz de Azure AI y la región del archivo de configuración. Debe usar estas variables para crear **SpeechTranslationConfig** para el recurso de Voz de Azure AI, que usará para traducir el lenguaje hablado. Agregue el código siguiente en el comentario **Configurar traducción**:

    **C#** : Program.cs

    ```csharp
    // Configure translation
    translationConfig = SpeechTranslationConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    translationConfig.SpeechRecognitionLanguage = "en-US";
    translationConfig.AddTargetLanguage("fr");
    translationConfig.AddTargetLanguage("es");
    translationConfig.AddTargetLanguage("hi");
    Console.WriteLine("Ready to translate from " + translationConfig.SpeechRecognitionLanguage);
    ```

    **Python**: translator.py

    ```python
    # Configure translation
    translation_config = speech_sdk.translation.SpeechTranslationConfig(ai_key, ai_region)
    translation_config.speech_recognition_language = 'en-US'
    translation_config.add_target_language('fr')
    translation_config.add_target_language('es')
    translation_config.add_target_language('hi')
    print('Ready to translate from',translation_config.speech_recognition_language)
    ```

1. Usará **SpeechTranslationConfig** para traducir voz a texto, pero también usará **SpeechConfig** para sintetizar las traducciones en voz. Agregue el código siguiente en el comentario **Configurar voz**:

    **C#** : Program.cs

    ```csharp
    // Configure speech
    speechConfig = SpeechConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    ```

    **Python**: translator.py

    ```python
    # Configure speech
    speech_config = speech_sdk.SpeechConfig(ai_key, ai_region)
    ```

1. Guarde los cambios y vuelva al terminal integrado de la carpeta **translator** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. Si usa C#, puede omitir las advertencias sobre el uso del operador **await** en métodos asincrónicos; lo corregiremos más adelante. El código debe mostrar un mensaje que indica que está listo para traducir desde en-US y le solicite un idioma de destino. Presione ENTRAR para finalizar el programa.

## Implementación de la traducción de voz

Ahora que tiene **SpeechTranslationConfig** para el servicio Voz de Azure AI, puede usar la API Azure AI Speech Translation para reconocer y traducir voz.

> **IMPORTANTE**: En esta sección se incluyen instrucciones para dos procedimientos alternativos. Siga el primer procedimiento si tiene un micrófono que funcione. Siga el segundo procedimiento si desea simular la entrada hablada mediante un archivo de audio.

### Si tiene un micrófono que funcione

1. En la función **Main** del programa, tenga en cuenta que el código usa la función **Translate** para traducir la entrada hablada.
1. En la función **Translate**, en el comentario **Traducir voz**, agregue el código siguiente para crear un cliente **TranslationRecognizer** que se pueda usar para reconocer y traducir la voz mediante el micrófono predeterminado del sistema para la entrada.

    **C#** : Program.cs

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

    **Python**: translator.py

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

    > **NOTA**: El código de la aplicación traduce la entrada a los tres idiomas en una sola llamada. Solo se muestra la traducción del idioma específico, pero puede recuperar cualquiera de las traducciones especificando el código de idioma de destino en la colección **traducciones** del resultado.

1. Ahora vaya directamente a la sección **Ejecución del programa** que aparece a continuación.

---

### Como alternativa, use la entrada de audio de un archivo

1. En la ventana de terminal, escriba el siguiente comando para instalar una biblioteca que pueda usar para reproducir el archivo de audio:

    **C#** : Program.cs

    ```csharp
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**: translator.py

    ```python
    pip install playsound==1.3.0
    ```

1. En el archivo de código del programa, en las importaciones del espacio de nombres existente, agregue el código siguiente para importar la biblioteca que acaba de instalar:

    **C#** : Program.cs

    ```csharp
    using System.Media;
    ```

    **Python**: translator.py

    ```python
    from playsound import playsound
    ```

1. En la función **Main** del programa, tenga en cuenta que el código usa la función **Translate** para traducir la entrada hablada. A continuación, en la función **Translate**, en el comentario **Traducir voz**, agregue el código siguiente para crear un cliente **TranslationRecognizer** que se pueda usar para reconocer y traducir la voz de un archivo.

    **C#** : Program.cs

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

    **Python**: translator.py

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

### Ejecución del programa

1. Guarde los cambios y vuelva al terminal integrado de la carpeta **translator** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. Cuando se le solicite, escriba un código de idioma válido(*fr*, *es* o *hi*) y, si usa un micrófono, hable claramente y diga "¿dónde está la estación?" o alguna otra frase que pueda usar al viajar. El programa debe transcribir la entrada hablada y traducirla al idioma especificado (francés, español o hindi). Repita este proceso e intente cada idioma admitido por la aplicación. Cuando haya terminado, presione ENTRAR para finalizar el programa.

    TranslationRecognizer proporciona unos 5 segundos para hablar. Si no detecta ninguna entrada hablada, genera un resultado "Sin coincidencia". Es posible que la traducción a Hindi no siempre se muestre correctamente en la ventana de la consola debido a problemas de codificación de caracteres.

> **Nota**: El código de la aplicación traduce la entrada a los tres idiomas en una sola llamada. Solo se muestra la traducción del idioma específico, pero puede recuperar cualquiera de las traducciones especificando el código de idioma de destino en la colección **traducciones** del resultado.

## Síntesis de la traducción a voz

Hasta ahora, la aplicación traduce la entrada hablada en texto, lo que puede ser suficiente si necesita pedir ayuda a alguien mientras viaja. Sin embargo, sería mejor que la traducción se pronunciara en voz alta en una voz adecuada.

1. En la función **Translate**, en el comentario **Síntesis de la traducción**, agregue el código siguiente para usar un cliente **SpeechSynthesizer** para sintetizar la traducción como voz a través del altavoz predeterminado:

    **C#** : Program.cs

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

    **Python**: translator.py

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

1. Guarde los cambios y vuelva al terminal integrado de la carpeta **translator** y escriba el siguiente comando para ejecutar el programa:

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. Cuando se le solicite, escriba un código de idioma válido *(fr*, *es* o *hi*) y, a continuación, hable claramente en el micrófono y diga una frase que podría usar al viajar. El programa debería transcribir la entrada hablada y responder con una traducción hablada. Repita este proceso e intente cada idioma admitido por la aplicación. Cuando haya terminado, presione **ENTRAR** para finalizar el programa.

> **NOTA**
> *En este ejemplo, usó **SpeechTranslationConfig** para traducir la voz en texto, y luego usó **SpeechConfig** para sintetizar la traducción como voz. Puede usar **SpeechTranslationConfig** para sintetizar la traducción directamente, pero solo funciona cuando se traduce a un solo idioma y da como resultado una secuencia de audio que normalmente se guarda como un archivo en lugar de enviarse directamente a un altavoz.*

## Información adicional

Para obtener más información sobre el uso de la API Azure AI Speech Translation, consulte la [documentación de traducción de voz](https://learn.microsoft.com/azure/ai-services/speech-service/speech-translation).
