---
lab:
  title: Traducir voz
  description: Traducir voz a voz e implementarlo en su propia aplicación.
---

# Traducir voz

Voz de Azure AI incluye una API Speech Translation que puede usar para traducir el lenguaje hablado. Por ejemplo, supongamos que quiere desarrollar una aplicación de traductor que los usuarios puedan usar al viajar a lugares donde no hablan el idioma local. Podrían decir frases como "¿Dónde está la estación?" o "Necesito encontrar un restaurante" en su propio idioma y hacer que los traduzca al idioma local. En este ejercicio, usará el SDK de Voz de Azure AI para Python para crear una aplicación sencilla basada en este ejemplo.

Aunque este ejercicio se basa en Python, puede desarrollar aplicaciones similares usando varios SDK específicos del lenguaje, como:

- [SDK de Voz de Azure AI para Python](https://pypi.org/project/azure-cognitiveservices-speech/)
- [SDK de Voz de Azure AI para .NET](https://www.nuget.org/packages/Microsoft.CognitiveServices.Speech)
- [SDK de Voz de Azure AI para JavaScript](https://www.npmjs.com/package/microsoft-cognitiveservices-speech-sdk)

Este ejercicio dura aproximadamente **30** minutos.

> **NOTA** Este ejercicio está diseñado para completarse en Azure Cloud Shell, donde no se admite el acceso directo al hardware de sonido del equipo. Por lo tanto, el laboratorio usará archivos de audio para flujos de entrada y salida de voz. El código para lograr los mismos resultados con un micrófono y un altavoz se proporciona como referencia.

## Crear un recurso de Voz de Azure AI

Comencemos creando un recurso de Azure AI Speech.

1. Abra [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` e inicie sesión con la cuenta de Microsoft asociada a su suscripción de Azure.
1. En el campo de búsqueda superior, busque **Servicio de voz**. Selecciónelo de la lista y, a continuación, seleccione **Crear**.
1. Aprovisione el recurso mediante la siguiente configuración:
    - **Suscripción**: *su suscripción a Azure*.
    - **Grupo de recursos**: *seleccione o cree un grupo de recursos*.
    - **Región**: *elija cualquier región disponible*
    - **Nombre**: *escriba un nombre único*.
    - **Plan de tarifa**: seleccione **F0** (*gratis*), o **S** (*estándar*) si F no está disponible.
1. Selecciona **Revisar y crear** y **Crear** para aprovisionar el recurso.
1. Espera a que se complete la implementación y, a continuación, ve al recurso implementado.
1. Visualiza la página **Claves y punto de conexión** en la sección **Administración de recursos**. Necesitarás la información de esta página más adelante en el ejercicio.

## Preparación para desarrollar una aplicación en Cloud Shell

1. Deje abierta la página **Claves y punto de conexión**, y use el botón **[\>_]** situado a la derecha de la barra de búsqueda de la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal; seleccione un entorno de ***PowerShell***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    **<font color="red">Asegúrate de que has cambiado a la versión clásica de Cloud Shell antes de continuar.</font>**

1. En el panel de PowerShell, escribe los siguientes comandos para clonar el repo de GitHub para este ejercicio:

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **Sugerencia**: Al escribir comandos en Cloud Shell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

1. Una vez que se haya clonado el repositorio, vaya a la carpeta que contiene los archivos de código:

    ```
   cd mslearn-ai-language/Labfiles/08-speech-translation/Python/translator
    ```

1. En el panel de línea de comandos, ejecute el siguiente comando para ver los archivos de código en la carpeta **translator**:

    ```
   ls -a -l
    ```

    Los archivos incluyen un archivo de configuración (**.env**) y un archivo de código (**translator.py**).

1. Cree un entorno virtual de Python e instale el paquete del SDK de Voz de Azure AI y otros paquetes necesarios ejecutando el siguiente comando:

    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install -r requirements.txt azure-cognitiveservices-speech==1.42.0
    ```

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. Actualice los valores de configuración para incluir la **región** y una **clave** del recurso de Voz de Azure AI que creó (disponible en la página **Claves y punto de conexión** del recurso Traductor de Azure AI en Azure Portal).
1. Después de reemplazar los marcadores de posición, usa el comando **CTRL+S** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

## Agregue código para usar el SDK de Voz de Azure AI

> **Sugerencia**: al agregar código, asegúrate de mantener la sangría correcta.

1. Escribe el siguiente comando para editar el archivo de código que se ha proporcionado:

    ```
   code translator.py
    ```

1. En la parte superior del archivo de código, en las referencias de espacios de nombres existentes, busque el comentario **Import namespaces**. A continuación, en este comentario, agregue el siguiente código específico según el lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Voz de Azure AI:

    ```python
   # Import namespaces
   from azure.core.credentials import AzureKeyCredential
   import azure.cognitiveservices.speech as speech_sdk
    ```

1. En la función **main**, en el comentario **Obtener valores de configuración**, observe que el código carga la clave y la región que definió en el archivo de configuración.

1. Busque el código siguiente en el comentario **Configurar traducción** y agregue el código siguiente para configurar la conexión al punto de conexión de Voz de los servicios de Azure AI:

    ```python
   # Configure translation
   translation_config = speech_sdk.translation.SpeechTranslationConfig(speech_key, speech_region)
   translation_config.speech_recognition_language = 'en-US'
   translation_config.add_target_language('fr')
   translation_config.add_target_language('es')
   translation_config.add_target_language('hi')
   print('Ready to translate from',translation_config.speech_recognition_language)
    ```

1. Usará **SpeechTranslationConfig** para traducir voz a texto, pero también usará **SpeechConfig** para sintetizar las traducciones en voz. Agregue el código siguiente en el comentario **Configurar voz**:

    ```python
   # Configure speech
   speech_config = speech_sdk.SpeechConfig(speech_key, speech_region)
   print('Ready to use speech service in:', speech_config.region)
    ```

1. Guarda los cambios (*CTRL+S*), pero deje abierto el editor de código.

## Ejecución de la aplicación

Hasta ahora, la aplicación no hace nada más que conectarse al recurso de Voz de Azure AI, pero resulta útil ejecutarla y comprobar que funciona antes de agregar funcionalidad de voz.

1. En el panel de línea de comandos, escriba el siguiente comando para ejecutar la aplicación de traducción:

    ```
   python translator.py
    ```

    El código debe mostrar la región del recurso del servicio de voz que usará la aplicación y un mensaje de que está listo para traducir de en-US y que el solicita un idioma de destino. Una ejecución correcta indica que la aplicación se ha conectado al servicio de Voz de Azure AI. Presione ENTRAR para finalizar el programa.

## Implementación de la traducción de voz

Ahora que tiene **SpeechTranslationConfig** para el servicio Voz de Azure AI, puede usar la API Azure AI Speech Translation para reconocer y traducir voz.

1. En el archivo de código, observe que el código usa la función **Translate** para traducir la entrada hablada. A continuación, en la función **Translate**, en el comentario **Traducir voz**, agregue el código siguiente para crear un cliente **TranslationRecognizer** que se pueda usar para reconocer y traducir la voz de un archivo.

    ```python
   # Translate speech
   current_dir = os.getcwd()
   audioFile = current_dir + '/station.wav'
   audio_config_in = speech_sdk.AudioConfig(filename=audioFile)
   translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config_in)
   print("Getting speech from file...")
   result = translator.recognize_once_async().get()
   print('Translating "{}"'.format(result.text))
   translation = result.translations[targetLanguage]
   print(translation)
    ```

1. Guarde los cambios (*CTRL+S*) y vuelva a ejecutar el programa:

    ```
   python translator.py
    ```

1. Cuando se le solicite, escriba un código de idioma válido (*fr*, *es* o *hi*). El programa debe transcribir el archivo de entrada y traducirlo al idioma especificado (francés, español o hindi). Repita este proceso e intente cada idioma admitido por la aplicación.

    > **NOTA**: Es posible que la traducción a Hindi no siempre se muestre correctamente en la ventana de la consola debido a problemas de codificación de caracteres.

1. Cuando haya terminado, presione ENTRAR para finalizar el programa.

> **Nota**: El código de la aplicación traduce la entrada a los tres idiomas en una sola llamada. Solo se muestra la traducción del idioma específico, pero puede recuperar cualquiera de las traducciones especificando el código de idioma de destino en la colección **traducciones** del resultado.

## Síntesis de la traducción a voz

Hasta ahora, la aplicación traduce la entrada hablada en texto, lo que puede ser suficiente si necesita pedir ayuda a alguien mientras viaja. Sin embargo, sería mejor que la traducción se pronunciara en voz alta en una voz adecuada.

> **Nota**: Debido a las limitaciones de hardware de Cloud Shell, dirigiremos la salida de voz sintetizada a un archivo.

1. En la función **Translate**, en el comentario **Sintetizar traducción**, agregue el código siguiente para usar un cliente **SpeechSynthesizer** para sintetizar la traducción como voz y guardarlo como un archivo .wav:

    ```python
   # Synthesize translation
   output_file = "output.wav"
   voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
   }
   speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
   audio_config_out = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config_out)
   speak = speech_synthesizer.speak_text_async(translation).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
   else:
        print("Spoken output saved in " + output_file)
    ```

1. Guarde los cambios (*CTRL+S*) y vuelva a ejecutar el programa:

    ```
   python translator.py
    ```

1. Revisa la salida de la aplicación, que debe indicar que la salida hablada se guardó en un archivo. Cuando haya terminado, presione **ENTRAR** para finalizar el programa.
1. Si tiene un reproductor multimedia capaz de reproducir archivos de audio .wav, descargue el archivo que se generó escribiendo el siguiente comando:

    ```
   download ./output.wav
    ```

    El comando download crea un vínculo emergente en la parte inferior derecha del explorador, que puedes seleccionar para descargar y abrir el archivo.

> **NOTA**
> *En este ejemplo, ha usado **SpeechTranslationConfig** para traducir voz a texto y, a continuación, ha usado **SpeechConfig** para sintetizar la traducción como voz. De hecho, puede usar **SpeechTranslationConfig** para sintetizar la traducción directamente, pero esto solo funciona al traducir a un solo idioma, y da como resultado una secuencia de audio que normalmente se guarda como un archivo.*

## Limpieza de recursos

Si ha terminado de explorar el servicio Voz de Azure AI, puede eliminar los recursos que creó en este ejercicio. A continuación, se indica cómo puede hacerlo.

1. Cierre del panel de Azure Cloud Shell
1. En Azure Portal, vaya al recurso de Voz de Azure AI que creó en este laboratorio.
1. En la página del recurso, seleccione **Eliminar** y siga las instrucciones para eliminar el recurso.

## ¿Qué ocurre si tienes un micrófono y un altavoz?

En este ejercicio, el entorno de Azure Cloud Shell que usamos no admite hardware de audio, por lo que ha usado archivos de audio para la entrada y la salida de voz. Veamos cómo se puede modificar el código para usar hardware de audio si lo tiene.

### Uso de la traducción de voz con un micrófono

1. Si tienes un micrófono, puedes usar el código siguiente para capturar la entrada hablada para la traducción de voz:

    ```python
   # Translate speech
   audio_config_in = speech_sdk.AudioConfig(use_default_microphone=True)
   translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config_in)
   print("Speak now...")
   result = translator.recognize_once_async().get()
   print('Translating "{}"'.format(result.text))
   translation = result.translations[targetLanguage]
   print(translation)
    ```

> **Nota**: El micrófono predeterminado del sistema es la entrada de audio predeterminada, por lo que también podrías omitir audioConfig en conjunto.

### Uso de la síntesis de voz con un altavoz

1. Si tienes un altavoz, puedes usar el código siguiente para sintetizar la voz.
    
    ```python
   # Synthesize translation
   voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
   }
   speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
   audio_config_out = speech_sdk.audio.AudioOutputConfig(use_default_speaker=True)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config_out)
   speak = speech_synthesizer.speak_text_async(translation).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

> **Nota**: El altavoz predeterminado del sistema es la salida de audio predeterminada, por lo que también podrías omitir audioConfig en conjunto.

## Más información

Para obtener más información sobre el uso de la API Azure AI Speech Translation, consulte la [documentación de traducción de voz](https://learn.microsoft.com/azure/ai-services/speech-service/speech-translation).
