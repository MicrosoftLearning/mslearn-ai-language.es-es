---
lab:
  title: Desarrollo de una aplicación de chat habilitado para audio
  description: Aprende a usar Fundición de IA de Azure para crear una aplicación de IA generativa que admita entradas de audio.
---

# Desarrollo de una aplicación de chat habilitado para audio

En este ejercicio, usarás el modelo de IA generativa *Phi-4-multimodal-instruct* para generar respuestas a indicaciones que incluyen archivos de imágenes. Desarrollarás una aplicación que proporcione asistencia de inteligencia artificial para una empresa de distribución mediante Fundición de IA de Azure y el servicio de inferencia del modelo de Azure AI para resumir los mensajes de voz dejados por los clientes.

Este ejercicio dura aproximadamente **30** minutos.

## Creación de un proyecto de Fundición de IA de Azure

Comencemos creando un proyecto de Fundición de IA de Azure.

1. En un explorador web, abre el [Portal de la Fundición de IA de Azure](https://ai.azure.com) en `https://ai.azure.com` e inicia sesión con tus credenciales de Azure. Cierra las sugerencias o paneles de inicio rápido que se abran la primera vez que inicias sesión y, si es necesario, usa el logotipo de **Fundición de IA de Azure** en la parte superior izquierda para navegar a la página principal, que es similar a la siguiente imagen:

    ![Captura de pantalla del Portal de la Fundición de IA de Azure.](../media/ai-foundry-home.png)

1. En la página principal, selecciona **+Crear proyecto**.
1. En el asistente para **crear un proyecto**, escribe un nombre válido y si se te sugiere un centro existente, elige la opción para crear uno nuevo. A continuación, revisa los recursos de Azure que se crearán automáticamente para admitir el centro y el proyecto.
1. Selecciona **Personalizar** y especifica la siguiente configuración para el centro:
    - **Nombre del centro**: *un nombre válido para el centro*
    - **Suscripción**: *suscripción a Azure*
    - **Grupo de recursos**: *crea o selecciona un grupo de recursos*
    - **Ubicación**: selecciona cualquiera de las siguientes regiones\*:
        - Este de EE. UU.
        - Este de EE. UU. 2
        - Centro-Norte de EE. UU
        - Centro-sur de EE. UU.
        - Centro de Suecia
        - Oeste de EE. UU.
        - Oeste de EE. UU. 3
    - **Conectar Servicios de Azure AI o Azure OpenAI**: *crea un nuevo recurso de servicios de IA*
    - **Conectar Búsqueda de Azure AI**: omite la conexión

    > \* En el momento de escribir esto, el modelo de Microsoft *Phi-4-multimodal-instruct* que usaremos en este ejercicio estaba disponible en estas regiones. Puedes comprobar la disponibilidad regional más reciente de los modelos específicos en la [documentación de Fundición de IA de Azure](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-serverless-availability#region-availability). En caso de que se alcance un límite de cuota regional más adelante en el ejercicio, es posible que tengas que crear otro recurso en otra región.

1. Selecciona **Siguiente** y revisa tu configuración. Luego, selecciona **Crear** y espera a que se complete el proceso.
1. Cuando se cree el proyecto, cierra las sugerencias que se muestran y revisa la página del proyecto en el Portal de la Fundición de IA de Azure, que debe tener un aspecto similar a la siguiente imagen:

    ![Captura de pantalla de los detalles de un proyecto de Azure AI en el Portal de la Fundición de IA de Azure.](../media/ai-foundry-project.png)

## Implementación de un modelo multimodal

Ahora estás listo para implementar un modelo multimodal que pueda admitir la entrada basada en audio. Hay varios modelos entre los que puedes elegir, incluido el modelo *gpt-4o* de OpenAI. En este ejercicio, usaremos un modelo *Phi-4-multimodal-instruct*.

1. En la barra de herramientas de la parte superior derecha de la página del proyecto de Fundición de IA de Azure, usa el icono **Características de versión preliminar** (**&#9215;**) para asegurarte de que está habilitada la característica **Implementar modelos en el servicio de inferencia de modelos de Azure AI**. Esta característica garantiza que la implementación del modelo esté disponible para el servicio de inferencia de Azure AI, que usarás en el código de aplicación.
1. En el panel de la izquierda de tu proyecto, en la sección **Mis recursos**, selecciona la página **Modelos y puntos de conexión**.
1. En la página **Modelos y puntos de conexión**, en la pestaña **Implementaciones de modelos**, en el menú **+ Implementar modelo**, selecciona **Implementar modelo base**.
1. Busca el modelo **Phi-4-multimodal-instruct** en la lista y, a continuación, selecciónalo y confírmalo.
1. Acepta el contrato de licencia si se te solicita y, a continuación, implementa el modelo con la siguiente configuración seleccionando **Personalizar** en los detalles de implementación:
    - **Nombre de implementación**: *un nombre válido para la implementación de modelo*
    - **Tipo de implementación**: estándar global
    - **Detalles de implementación**: *usa la configuración predeterminada*
1. Espera a que se **complete** el estado de aprovisionamiento de la implementación.

## Creación de una aplicación cliente

Ahora que has implementado el modelo, puedes usar la implementación en una aplicación cliente.

> **Sugerencia**: puedes elegir desarrollar la solución mediante C# de Python o Microsoft. Sigue las instrucciones de la sección adecuada para el idioma elegido.

### Preparación de la configuración de aplicación

1. En el Portal de la Fundición de IA de Azure, mira la página **Información general** del proyecto.
1. En el área **Detalles del proyecto**, anota la **Cadena de conexión del proyecto**. Usarás esta cadena de conexión para conectarte al proyecto en una aplicación cliente.
1. Abre una nueva pestaña del explorador (mantén el Portal de la Fundición de IA de Azure abierto en la pestaña existente). En la nueva pestaña, explora [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` e inicia sesión con tus credenciales de Azure, si se te solicita.

    Cierra las notificaciones de bienvenida para ver la página principal de Azure Portal.

1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell*** sin almacenamiento en tu suscripción.

    Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal. Puedes cambiar el tamaño o maximizar este panel para facilitar el trabajo.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    **<font color="red">Asegúrate de que has cambiado a la versión clásica de Cloud Shell antes de continuar.</font>**

1. En el panel de Cloud Shell, escribe los siguientes comandos para clonar el repositorio de GitHub que contiene los archivos de código de este ejercicio (escribe el comando o cópialo en el Portapapeles y haz clic con el botón derecho en la línea de comandos y pega como texto sin formato):


    ```
    rm -r mslearn-ai-audio -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-language mslearn-ai-audio
    ```

    > **Sugerencia**: al pegar comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

1. Una vez clonado el repo, ve a la carpeta que contiene los archivos de código de aplicación:  

    **Python**

    ```
    cd mslearn-ai-audio/Labfiles/09-audio-chat/Python
    ```

    **C#**

    ```
    cd mslearn-ai-audio/Labfiles/09-audio-chat/C-sharp
    ```

1. En el panel de la línea de comandos de Cloud Shell, escribe el siguiente comando para instalar las bibliotecas que vas a usar:

    **Python**

    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install -r requirements.txt azure-identity azure-ai-projects azure-ai-inference
    ```

    **C#**

    ```
    dotnet add package Azure.Identity
    dotnet add package Azure.AI.Inference --version 1.0.0-beta.3
    dotnet add package Azure.AI.Projects --version 1.0.0-beta.3
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

    El archivo debería abrirse en un editor de código.

1. En el archivo de código, reemplaza el marcador de posición **your_project_connection_string** por la cadena de conexión del proyecto (copiado de la página **Información general** del proyecto en el Portal de la Fundición de IA de Azure) y el marcador de posición **your_model_deployment** por el nombre que asignaste a la implementación de modelo Phi-4-multimodal-instruct.

1. Después de reemplazar los marcadores de posición, en el editor de código, usa el comando **CTRL+S** o **Clic derecho > Guardar** para guardar los cambios y después usa el comando **CTRL+Q** o **Clic derecho > Salir** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Escritura de código para conectarte al proyecto y obtener un cliente de chat para el modelo

> **Sugerencia**: al agregar código, asegúrate de mantener la sangría correcta.

1. Escribe el siguiente comando para editar el archivo de código que se ha proporcionado:

    **Python**

    ```
    code audio-chat.py
    ```

    **C#**

    ```
    code Program.cs
    ```

1. En el archivo de código, observa las instrucciones existentes que se han agregado en la parte superior del archivo para importar los espacios de nombres de SDK necesarios. Después, busca el comentario **Add references** y agrega el código siguiente para hacer referencia a los espacios de nombres de las bibliotecas que has instalado anteriormente:

    **Python**

    ```python
    # Add references
    from dotenv import load_dotenv
    from azure.identity import DefaultAzureCredential
    from azure.ai.projects import AIProjectClient
    from azure.ai.inference.models import (
        SystemMessage,
        UserMessage,
        TextContentItem,
    )
    ```

    **C#**

    ```csharp
    // Add references
    using Azure.Identity;
    using Azure.AI.Projects;
    using Azure.AI.Inference;
    ```

1. En la función **principal**, en el comentario **Obtener ajustes de configuración**, observa que el código carga los valores de la cadena de conexión del proyecto y del nombre de implementación del modelo que has definido en el archivo de configuración.

1. Busca el comentario **Initialize the project client** y agrega el siguiente código para conectarte a tu proyecto de Fundición de IA de Azure mediante las credenciales de Azure con las que has iniciado sesión:

    **Python**

    ```python
    # Initialize the project client
    project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```csharp
    // Initialize the project client
    var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
    ```

1. Busca el comentario **Get a chat client** y agrega el siguiente código para crear un objeto de cliente para chatear con el modelo:

    **Python**

    ```python
    # Get a chat client
    chat_client = project_client.inference.get_chat_completions_client(model=model_deployment)
    ```

    **C#**

    ```csharp
    // Get a chat client
    ChatCompletionsClient chat = projectClient.GetChatCompletionsClient();
    ```
    

### Escritura de código para enviar una indicación de audio basada en direcciones URL

1. En el editor de código para el archivo **audio-chat.py**, en la sección bucle, en el comentario **Obtener una respuesta para la entrada de audio**, agrega el código siguiente para enviar una indicación que incluya el siguiente audio:

    <video controls src="../media/avocados.mp4" title="Una solicitud de aguacates" width="150"></video>

    **Python**

    ```python
    # Get a response to audio input
    file_path = "https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Labfiles/09-audio-chat/data/avocados.mp3"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(
                [
                    TextContentItem(text=prompt),
                    {
                        "type": "audio_url",
                        "audio_url": {"url": file_path}
                    }
                ]
            )
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
    // Get a response to audio input
    string audioUrl = "https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Labfiles/09-audio-chat/data/avocados.mp3";
    var requestOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage(
                new ChatMessageTextContentItem(prompt),
                new ChatMessageAudioContentItem(new Uri(audioUrl))),
        },
        Model = model_deployment
    };
    var response = chat.Complete(requestOptions);
    Console.WriteLine(response.Value.Content);
    ```

1. Usa el comando **CTRL+S** para guardar los cambios en el archivo de código. También puedes cerrar el editor de código (**CTRL+Q**) si lo deseas.

1. En el panel de línea de comandos de Cloud Shell, debajo del editor de código, escribe el siguiente comando para ejecutar la aplicación:

    **Python**

    ```
    python audio-chat.py
    ```

    **C#**

    ```
    dotnet run
    ```

1. Cuando se te solicite, escribe la indicación 

    ```
    Can you summarize this customer's voice message?
    ```

1. Revisa la respuesta.

### Usa otro archivo de audio

1. En el editor de código de tu aplicación, busca el código que has agregado anteriormente en el comentario **Get a response to audio input**. Después, modifica el código de la siguiente manera para seleccionar un archivo de audio diferente:

    <video controls src="../media/fresas.mp4" title="Una solicitud de fresas" width="150"></video>

    **Python**

    ```python
    # Get a response to audio input
    file_path = "https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Labfiles/09-audio-chat/data/fresas.mp3"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(
                [
                    TextContentItem(text=prompt),
                    {
                        "type": "audio_url",
                        "audio_url": {"url": file_path}
                    }
                ]
            )
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
    // Get a response to audio input
    string audioUrl = "https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Labfiles/09-audio-chat/data/fresas.mp3";
    var requestOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage(
                new ChatMessageTextContentItem(prompt),
                new ChatMessageAudioContentItem(new Uri(audioUrl))),
        },
        Model = model_deployment
    };
    var response = chat.Complete(requestOptions);
    Console.WriteLine(response.Value.Content);
    ```

1. Usa el comando **CTRL+S** para guardar los cambios en el archivo de código. También puedes cerrar el editor de código (**CTRL+Q**) si lo deseas.

1. En el panel de línea de comandos de Cloud Shell, debajo del editor de código, escribe el siguiente comando para ejecutar la aplicación:

    **Python**

    ```
    python audio-chat.py
    ```

    **C#**

    ```
    dotnet run
    ```

1. Cuando se te solicite, escribe la siguiente indicación: 
    
    ```
    Can you summarize this customer's voice message? Is it time-sensitive?
    ```

1. Revisa la respuesta. A continuación, escribe `quit` para salir del programa.

    > **Nota**: en esta aplicación sencilla, no hemos implementado una lógica para conservar el historial de conversaciones, por lo que el modelo tratará cada indicación como una nueva solicitud sin contexto de la indicación anterior.

1. Puedes seguir ejecutando la aplicación, eligiendo diferentes tipos de indicaciones y probando indicaciones diferentes. Cuando hayas terminado, presiona `quit` para salir del programa.

    Si tienes tiempo, puedes modificar el código para usar una indicación del sistema diferente y tus propios archivos de audio accesibles por Internet.

    > **Nota**: en esta aplicación sencilla, no hemos implementado una lógica para conservar el historial de conversaciones, por lo que el modelo tratará cada indicación como una nueva solicitud sin contexto de la indicación anterior.

## Resumen

En este ejercicio, has usado Fundición de IA de Azure y el SDK de inferencia de Azure AI para crear una aplicación cliente que usa un modelo multimodal para generar respuestas para audio.

## Limpieza

Si has terminado de explorar Fundición de IA de Azure, deberías eliminar los recursos que has creado en este ejercicio para evitar incurrir en costes innecesarios de Azure.

1. Vuelve a la pestaña del explorador que contiene Azure Portal (o vuelve a abrir [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` en una nueva pestaña del explorador) y mira el contenido del grupo de recursos donde implementó los recursos usados en este ejercicio.
1. Selecciona **Eliminar grupo de recursos** en la barra de herramientas.
1. Escribe el nombre del grupo de recursos y confirma que deseas eliminarlo.
