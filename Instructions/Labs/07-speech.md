---
lab:
  title: Reconocimiento y síntesis de voz
  description: Implementar un reloj parlante que convierta la voz en texto y el texto en voz.
---

# Reconocimiento y síntesis de voz

**Voz de Azure AI** es un servicio que proporciona la funcionalidad relacionada con la voz, que incluye lo siguiente:

- Una API de *conversión de voz a texto* que permite implementar el reconocimiento de voz (conversión de texto oral audible en texto).
- Una API de *conversión de texto a voz* que permite implementar la síntesis de voz (conversión de texto en voz audible).

En este ejercicio, usará estas dos API para implementar una aplicación de reloj de voz.

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

## Preparación y configuración de la aplicación del reloj que habla

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

1. Una vez clonado el repositorio, ve a la carpeta que contiene los archivos de código de la aplicación del reloj que habla:  

    ```
   cd mslearn-ai-language/Labfiles/07-speech/Python/speaking-clock
    ```

1. En el panel de línea de comandos, ejecute el siguiente comando para ver los archivos de código de la carpeta **speaking-clock**:

    ```
   ls -a -l
    ```

    Los archivos incluyen un archivo de configuración (**.env**) y un archivo de código (**speaking-clock.py**). Los archivos de audio que usará la aplicación se encuentran en la subcarpeta **audio**.

1. Cree un entorno virtual de Python e instale el paquete del SDK de Voz de Azure AI y otros paquetes necesarios ejecutando el siguiente comando:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-cognitiveservices-speech==1.42.0
    ```

1. Use el comando siguiente para editar el archivo de configuración:

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
   code speaking-clock.py
    ```

1. En la parte superior del archivo de código, en las referencias de espacios de nombres existentes, busque el comentario **Import namespaces**. A continuación, en este comentario, agregue el siguiente código específico según el lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Voz de Azure AI:

    ```python
   # Import namespaces
   from azure.core.credentials import AzureKeyCredential
   import azure.cognitiveservices.speech as speech_sdk
    ```

1. En la función **main**, en el comentario **Obtener valores de configuración**, observe que el código carga la clave y la región que definió en el archivo de configuración.

1. Busque el comentario **Configurar servicio de voz** y agregue el siguiente código para usar la clave y la región de servicios de IA para configurar tu conexión con el punto de conexión de Voz de los servicios de Azure AI:

    ```python
   # Configure speech service
   speech_config = speech_sdk.SpeechConfig(speech_key, speech_region)
   print('Ready to use speech service in:', speech_config.region)
    ```

1. Guarda los cambios (*CTRL+S*), pero deje abierto el editor de código.

## Ejecución de la aplicación

Hasta ahora, la aplicación no hace nada más que conectarse al servicio de Voz de Azure AI, pero resulta útil ejecutarla y comprobar que funciona antes de agregar funcionalidad de voz.

1. En la línea de comandos, escribe el siguiente comando para ejecutar la aplicación de reloj parlante:

    ```
   python speaking-clock.py
    ```

    El código debe mostrar la región del recurso del servicio de voz que usará la aplicación. Una ejecución correcta indica que la aplicación se ha conectado al recurso de Voz de Azure AI.

## Agregar código para reconocer voz

Ahora que tienes una **SpeechConfig** para el servicio de voz en tu recurso de Servicios de Azure AI del proyecto, puedes usar la API **Speech-to-text** para reconocer la voz y transcribirla a texto.

En este procedimiento, la entrada de voz se captura desde un archivo de audio, que puedes reproducir aquí:

<video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Time.mp4" title="¿Qué hora tenemos?" width="150"></video>

1. En el archivo de código, observe que el código usa la función **TranscribeCommand** para aceptar la entrada hablada. A continuación, en la función **TranscribeCommand**, busque el comentario **Configurar reconocimiento de voz** y agregue a continuación el código adecuado para crear un cliente **SpeechRecognizer** que se pueda usar para reconocer y transcribir la voz de un archivo de audio:

    ```python
   # Configure speech recognition
   current_dir = os.getcwd()
   audioFile = current_dir + '/time.wav'
   audio_config = speech_sdk.AudioConfig(filename=audioFile)
   speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

1. En la función **TranscribeCommand**, en el comentario **Process speech input** (Procesar entrada de voz), agregue el código siguiente para escuchar la entrada hablada, y tenga cuidado de no reemplazar el código al final de la función que devuelve el comando:

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

1. Guarde los cambios (*CTRL+S*) y, a continuación, en la línea de comandos de debajo del editor de código, vuelva a ejecutar el programa:
1. Revise la salida, que debería "escuchar" correctamente la voz del archivo de audio y devolver una respuesta adecuada (tenga en cuenta que Azure Cloud Shell puede estar ejecutándose en un servidor que se encuentra en una zona horaria diferente a la suya).

    > **Consejo**: Si SpeechRecognizer encuentra un error, genera el resultado "Cancelado". A continuación, el código de la aplicación mostrará el mensaje de error. La causa más probable es un valor de región incorrecto en el archivo de configuración.

## Sintetizar voz

La aplicación de reloj de voz acepta la entrada hablada, pero en realidad no habla. Vamos a corregirlo agregando código para sintetizar la voz.

Una vez más, debido a las limitaciones de hardware de Cloud Shell, dirigiremos la salida de voz sintetizada a un archivo.

1. En el archivo de código, observe que el código usa la función **TellTime** para indicar al usuario la hora actual.
1. En la función **TellTime**, en el comentario **Configure speech synthesis** (Configurar síntesis de voz), agregue el código siguiente para crear un cliente **SpeechSynthesizer** que se pueda usar para generar salida de voz:

    ```python
   # Configure speech synthesis
   output_file = "output.wav"
   speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
   audio_config = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config,)
    ```

1. En la función **TellTime**, en el comentario **Synthesize spoken output** (Sintetizar salida de voz), agregue el código siguiente para generar la salida hablada y tenga cuidado de no reemplazar el código al final de la función que imprime la respuesta:

    ```python
   # Synthesize spoken output
   speak = speech_synthesizer.speak_text_async(response_text).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
   else:
        print("Spoken output saved in " + output_file)
    ```

1. Guarde los cambios (*CTRL+S*) y vuelva a ejecutar el programa, que indicará que la salida hablada se guardó en un archivo.

1. Si tiene un reproductor multimedia capaz de reproducir archivos de audio .wav, descargue el archivo que se generó escribiendo el siguiente comando:

    ```
   download ./output.wav
    ```

    El comando download crea un vínculo emergente en la parte inferior derecha del explorador, que puedes seleccionar para descargar y abrir el archivo.

    El archivo debe sonar de forma similar a:

    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Output.mp4" title="La hora es 2:15" width="150"></video>

## Uso de Lenguaje de marcado de síntesis de voz

El lenguaje de marcado de síntesis de voz (SSML) permite personalizar la forma en que se sintetiza la voz mediante un formato basado en XML.

1. En la función **TellTime**, reemplace todo el código actual en el comentario **Synthesize spoken output** (Sintentizar salida de voz) por el código siguiente (deje el código del comentario **Print the response** [Imprimir la respuesta]):

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
       print("Spoken output saved in " + output_file)
    ```

1. Guarde los cambios y vuelva a ejecutar el programa, que indicará una vez más que la salida hablada se guardó en un archivo.
1. Descargue y reproduzca el archivo generado, que debería sonar como este:
    
    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Output2.mp4" title="La hora es 5:30. Es hora de finalizar este laboratorio." width="150"></video>

## Limpieza

Si has terminado de explorar Voz de Azure AI, debes eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. Cierre del panel de Azure Cloud Shell
1. En Azure Portal, vaya al recurso de Voz de Azure AI que creó en este laboratorio.
1. En la página del recurso, seleccione **Eliminar** y siga las instrucciones para eliminar el recurso.

## ¿Qué ocurre si tienes un micrófono y un altavoz?

En este ejercicio, el entorno de Azure Cloud Shell que usamos no admite hardware de audio, por lo que ha usado archivos de audio para la entrada y la salida de voz. Veamos cómo se puede modificar el código para usar hardware de audio si lo tiene.

### Uso del reconocimiento de voz con un micrófono

Si tienes un micrófono, puedes usar el código siguiente para capturar la entrada hablada para el reconocimiento de voz:

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

> **Nota**: El micrófono predeterminado del sistema es la entrada de audio predeterminada, por lo que también podrías omitir audioConfig en conjunto.

### Uso de la síntesis de voz con un altavoz

Si tienes un altavoz, puedes usar el código siguiente para sintetizar la voz.

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

> **Nota**: El altavoz predeterminado del sistema es la salida de audio predeterminada, por lo que también podrías omitir audioConfig en conjunto.

## Más información

Para más información sobre el uso de las API de **conversión de voz en texto** y de **conversión de texto a voz**, consulte la [documentación de conversión de voz en texto](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text) y la [documentación de conversión de texto a voz](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech).
