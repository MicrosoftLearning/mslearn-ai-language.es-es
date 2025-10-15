---
lab:
  title: Desarrollo de un agente de voz en directo de Voz de Azure AI
  description: Aprenda a crear una aplicación web para permitir interacciones de voz en tiempo real con un agente en directo de Voz de Azure AI.
---

# Desarrollo de un agente de voz en directo de Voz de Azure AI

En este ejercicio, completará una aplicación web de Python basada en Flask que permite interacciones de voz en tiempo real con un agente. Agregará el código para inicializar la sesión y controlar los eventos de sesión. Usará un script de implementación que implementa el modelo de IA; crea una imagen de la aplicación en Azure Container Registry (ACR) mediante tareas de ACR y, a continuación, crea una instancia de Azure App Service que extrae la imagen. Para probar la aplicación, necesitará un dispositivo de audio con funcionalidades de micrófono y altavoz.

Aunque este ejercicio se basa en Python, puede desarrollar aplicaciones similares usando otros SDK específicos del lenguaje, como:

- [Biblioteca cliente de Azure VoiceLive para .NET](https://www.nuget.org/packages/Azure.AI.VoiceLive/)

Tareas realizadas en este ejercicio:

* Descargar los archivos base de la aplicación
* Agregar código para completar la aplicación web
* Revisar la base de código general
* Actualizar y ejecutar el script de implementación
* Ver y probar la aplicación

Este ejercicio se realiza en aproximadamente **30** minutos.

## Iniciar Azure Cloud Shell y descargar los archivos

En esta sección del ejercicio descargará un archivo comprimido que contiene los archivos base de la aplicación.

1. En el explorador, ve a Azure Portal [https://portal.azure.com](https://portal.azure.com) e inicia sesión con tus credenciales de Azure, si se te solicita.

1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***Bash***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *PowerShell*, cámbiala a ***Bash***.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

1. Ejecute el siguiente comando en el shell de **Bash** para descargar y descomprimir los archivos del ejercicio. El segundo comando también cambiará al directorio de los archivos del ejercicio.

    ```bash
    wget https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/downloads/python/voice-live-web.zip
    ```

    ```
    unzip voice-live-web.zip && cd voice-live-web
    ```

## Agregar código para completar la aplicación web

Ahora que se han descargado los archivos del ejercicio, el siguiente paso es agregar código para completar la aplicación. Siga estos pasos en Cloud Shell. 

>**Sugerencia:** Para cambiar el tamaño de Cloud Shell a fin de mostrar más información y código, arrastre el borde superior. También puede usar los botones de minimizar y maximizar para cambiar entre Cloud Shell y la interfaz principal del portal.

Ejecute el siguiente comando para cambiar al directorio *src* antes de continuar con el ejercicio.

```bash
cd src
```

### Agregar código para implementar el asistente en directo de voz

En esta sección, agregará código para implementar el asistente en directo de voz. El método **\_\_init\_\_** inicializa el asistente de voz almacenando los parámetros de conexión de Azure VoiceLive (punto de conexión, credenciales, modelo, voz e instrucciones del sistema) y configurando variables de estado en tiempo de ejecución para administrar el ciclo de vida de la conexión y controlar las interrupciones del usuario durante las conversaciones. El método **start** importa los componentes necesarios del SDK de Azure VoiceLive que se usarán para establecer la conexión de WebSocket y configurar la sesión de voz en tiempo real.

1. Ejecute el siguiente comando para abrir el archivo *flask_app.py* y editarlo.

    ```bash
    code flask_app.py
    ```

1. Busque el comentario **# BEGIN VOICE LIVE ASSISTANT IMPLEMENTATION - ALIGN CODE WITH COMMENT** en el código. Copie el código siguiente y escríbalo justo debajo del comentario. Asegúrese de comprobar la sangría.

    ```python
    def __init__(
        self,
        endpoint: str,
        credential,
        model: str,
        voice: str,
        instructions: str,
        state_callback=None,
    ):
        # Store Azure Voice Live connection and configuration parameters
        self.endpoint = endpoint
        self.credential = credential
        self.model = model
        self.voice = voice
        self.instructions = instructions
        
        # Initialize runtime state - connection established in start()
        self.connection = None
        self._response_cancelled = False  # Used to handle user interruptions
        self._stopping = False  # Signals graceful shutdown
        self.state_callback = state_callback or (lambda *_: None)

    async def start(self):
        # Import Voice Live SDK components needed for establishing connection and configuring session
        from azure.ai.voicelive.aio import connect  # type: ignore
        from azure.ai.voicelive.models import (
            RequestSession,
            ServerVad,
            AzureStandardVoice,
            Modality,
            InputAudioFormat,
            OutputAudioFormat,
        )  # type: ignore
    ```

1. Escriba **ctrl+s** para guardar los cambios y mantenga el editor abierto para la sección siguiente.

### Agregar código para implementar el asistente en directo de voz

En esta sección, agregará código para configurar la sesión en directo de voz. Este especifica las modalidades (solo audio no es compatible con la API), las instrucciones del sistema que definen el comportamiento del asistente, la voz de Azure TTS para las respuestas, el formato de audio para las secuencias de entrada y salida y la detección de actividad de voz del lado servidor (VAD), que especifica cómo detecta el modelo cuando los usuarios comienzan a hablar y se paran.

1. Busque el comentario **# BEGIN CONFIGURE VOICE LIVE SESSION - ALIGN CODE WITH COMMENT** en el código. Copie el código siguiente y escríbalo justo debajo del comentario. Asegúrese de comprobar la sangría.

    ```python
    # Configure VoiceLive session with audio/text modalities and voice activity detection
    session_config = RequestSession(
        modalities=[Modality.TEXT, Modality.AUDIO],
        instructions=self.instructions,
        voice=voice_cfg,
        input_audio_format=InputAudioFormat.PCM16,
        output_audio_format=OutputAudioFormat.PCM16,
        turn_detection=ServerVad(threshold=0.5, prefix_padding_ms=300, silence_duration_ms=500),
    )
    await conn.session.update(session=session_config)
    ```

1. Escriba **ctrl+s** para guardar los cambios y mantenga el editor abierto para la sección siguiente.

### Agregar código para controlar los eventos de sesión

En esta sección, agregará código para agregar controladores de eventos para la sesión en directo de voz. Los controladores de eventos responden a los eventos de sesión de VoiceLive clave durante el ciclo de vida de la conversación: **_handle_session_updated** señala cuándo la sesión está lista para la entrada del usuario; **_handle_speech_started** detecta cuándo el usuario comienza a hablar e implementa la lógica de interrupción deteniendo cualquier reproducción de audio del asistente en curso y cancelando las respuestas en curso para permitir el flujo de conversación natural; y **_handle_speech_stopped** controla cuándo el usuario ha terminado de hablar y el asistente comienza a procesar la entrada.

1. Busque el comentario **# BEGIN HANDLE SESSION EVENTS - ALIGN CODE WITH COMMENT** en el código. Copie el código siguiente y escríbalo justo debajo del comentario; no olvide comprobar la sangría.

    ```python
    async def _handle_event(self, event, conn, verbose=False):
        """Handle Voice Live events with clear separation by event type."""
        # Import event types for processing different Voice Live server events
        from azure.ai.voicelive.models import ServerEventType
        
        event_type = event.type
        if verbose:
            _broadcast({"type": "log", "level": "debug", "event_type": str(event_type)})
        
        # Route Voice Live server events to appropriate handlers
        if event_type == ServerEventType.SESSION_UPDATED:
            await self._handle_session_updated()
        elif event_type == ServerEventType.INPUT_AUDIO_BUFFER_SPEECH_STARTED:
            await self._handle_speech_started(conn)
        elif event_type == ServerEventType.INPUT_AUDIO_BUFFER_SPEECH_STOPPED:
            await self._handle_speech_stopped()
        elif event_type == ServerEventType.RESPONSE_AUDIO_DELTA:
            await self._handle_audio_delta(event)
        elif event_type == ServerEventType.RESPONSE_AUDIO_DONE:
            await self._handle_audio_done()
        elif event_type == ServerEventType.RESPONSE_DONE:
            # Reset cancellation flag but don't change state - _handle_audio_done already did
            self._response_cancelled = False
        elif event_type == ServerEventType.ERROR:
            await self._handle_error(event)

    async def _handle_session_updated(self):
        """Session is ready for conversation."""
        self.state_callback("ready", "Session ready. You can start speaking now.")

    async def _handle_speech_started(self, conn):
        """User started speaking - handle interruption if needed."""
        self.state_callback("listening", "Listening… speak now")
        
        try:
            # Stop any ongoing audio playback on the client side
            _broadcast({"type": "control", "action": "stop_playback"})
            
            # If assistant is currently speaking or processing, cancel the response to allow interruption
            current_state = assistant_state.get("state")
            if current_state in {"assistant_speaking", "processing"}:
                self._response_cancelled = True
                await conn.response.cancel()
                _broadcast({"type": "log", "level": "debug", 
                          "msg": f"Interrupted assistant during {current_state}"})
            else:
                _broadcast({"type": "log", "level": "debug", 
                          "msg": f"User speaking during {current_state} - no cancellation needed"})
        except Exception as e:
            _broadcast({"type": "log", "level": "debug", 
                      "msg": f"Exception in speech handler: {e}"})

    async def _handle_speech_stopped(self):
        """User stopped speaking - processing input."""
        self.state_callback("processing", "Processing your input…")

    async def _handle_audio_delta(self, event):
        """Stream assistant audio to clients."""
        if self._response_cancelled:
            return  # Skip cancelled responses
            
        # Update state when assistant starts speaking
        if assistant_state.get("state") != "assistant_speaking":
            self.state_callback("assistant_speaking", "Assistant speaking…")
        
        # Extract and broadcast Voice Live audio delta as base64 to WebSocket clients
        audio_data = getattr(event, "delta", None)
        if audio_data:
            audio_b64 = base64.b64encode(audio_data).decode("utf-8")
            _broadcast({"type": "audio", "audio": audio_b64})

    async def _handle_audio_done(self):
        """Assistant finished speaking."""
        self._response_cancelled = False
        self.state_callback("ready", "Assistant finished. You can speak again.")

    async def _handle_error(self, event):
        """Handle Voice Live errors."""
        error = getattr(event, "error", None)
        message = getattr(error, "message", "Unknown error") if error else "Unknown error"
        self.state_callback("error", f"Error: {message}")

    def request_stop(self):
        self._stopping = True
    ```

1. Escriba **ctrl+s** para guardar los cambios y mantenga el editor abierto para la sección siguiente.

### Revisar el código de la aplicación

Hasta ahora, ha agregado código a la aplicación para implementar el agente y controlar los eventos del agente. Dedique unos minutos a revisar el código completo y los comentarios para comprender mejor cómo la aplicación controla el estado y las operaciones del cliente.

1. Cuando haya terminado, presione **ctrl+q** para salir del editor. 

## Actualizar y ejecutar el script de implementación

En esta sección realizará un pequeño cambio en el script de implementación **azdeploy.sh** y, a continuación, ejecutará la implementación. 

### Actualizar el script de implementación

Solo hay dos valores que debe cambiar en la parte superior del script de implementación **azdeploy.sh**. 

* El valor **rg** especifica el grupo de recursos que va a contener la implementación. Puede aceptar el valor predeterminado o escribir su propio valor si necesita implementarlo en un grupo de recursos específico.

* El valor de **ubicación** establece la región de la implementación. El modelo *gpt-4o* que se usa en el ejercicio se puede implementar en otras regiones, pero puede haber límites en cualquier región determinada. Si se produce un error en la implementación en la región elegida, pruebe **eastus2** o **swedencentral**. 

    ```
    rg="rg-voicelive" # Replace with your resource group
    location="eastus2" # Or a location near you
    ```

1. Ejecute los comandos siguientes en Cloud Shell para empezar a editar el script de implementación.

    ```bash
    cd ~/01-voice-live-web
    ```
    
    ```bash
    code azdeploy.sh
    ```

1. Actualice los valores de **rg** y **location** para satisfacer sus necesidades y, a continuación, presione **ctrl+s** para guardar los cambios y **ctrl+q** para salir del editor.

### Ejecutar el script de implementación

El script de implementación implementa el modelo de IA y crea los recursos necesarios en Azure para ejecutar una aplicación contenedorizada en App Service.

1. Ejecute el comando siguiente en Cloud Shell para empezar a implementar los recursos de Azure y la aplicación.

    ```bash
    bash azdeploy.sh
    ```

1. Seleccione **opción 1** para la implementación inicial.

    La implementación debe completarse en 5 o 10 minutos. Durante la implementación, es posible que se le solicite la siguiente información o acciones:
    
    * Si se le pide que se autentique en Azure, siga las instrucciones que se le presentan.
    * Si se le pide que seleccione una suscripción, use las teclas de dirección para resaltar la suscripción y presione **Entrar**. 
    * Si ve algunas advertencias durante la implementación, puede omitirlas.
    * Si se produce un error en la implementación del modelo de IA, cambie la región del script de implementación e inténtelo de nuevo. 
    * En ocasiones, las regiones de Azure pueden estar ocupadas e interrumpir la cronología de las implementaciones. Si se produce un error en la implementación después de que la implementación del modelo, vuelva a ejecutar el script de implementación.

## Ver y probar la aplicación

Cuando la implementación se completa, aparece un mensaje "Implementación finalizada" en el shell junto con un vínculo a la aplicación web. Puede seleccionar ese vínculo o ir al recurso de App Service e iniciar la aplicación desde allí. La aplicación puede tardar unos minutos en cargarse. 

1. Seleccione el botón **Iniciar sesión** para conectarse al modelo.
1. Se le pedirá que proporcione a la aplicación acceso a los dispositivos de audio.
1. Cuando la aplicación le pida que empiece a hablar, comience a hablar con el modelo.

Solución de problemas:

* Si la aplicación notifica que faltan variables de entorno, reinicie la aplicación en App Service.
* Si ve que un número excesivo de mensajes de *fragmentos de audio* en el registro se muestra en la aplicación, seleccione **Detener sesión** y, a continuación, vuelva a iniciar la sesión. 
* Si la aplicación no funciona, vuelva a comprobar que ha agregado todo el código y que la sangría es correcta. Si necesita realizar cambios, vuelva a ejecutar la implementación y seleccione **opción 2** para actualizar solo la imagen.

## Limpieza de recursos

Ejecute el siguiente comando en Cloud Shell para quitar todos los recursos implementados para este ejercicio. Se le pedirá que confirme la eliminación de los recursos.

```
azd down --purge
```