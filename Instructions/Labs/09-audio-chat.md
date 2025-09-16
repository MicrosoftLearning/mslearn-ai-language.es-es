---
lab:
  title: Desarrollo de una aplicación de chat habilitado para audio
  description: Aprende a usar Fundición de IA de Azure para crear una aplicación de IA generativa que admita entradas de audio.
---

# Desarrollo de una aplicación de chat habilitado para audio

En este ejercicio, usarás el modelo de IA generativa *Phi-4-multimodal-instruct* para generar respuestas a indicaciones que incluyen archivos de imágenes. Desarrollará una aplicación que proporcione asistencia con IA para una empresa de distribución usando Fundición de IA de Azure y el SDK de OpenAI para Python para resumir los mensajes de voz dejados por los clientes.

Aunque este ejercicio se basa en Python, puede desarrollar aplicaciones similares mediante varios SDK específicos del lenguaje; incluidos los siguientes:

- [Proyectos de Azure AI para Python](https://pypi.org/project/azure-ai-projects)
- [Biblioteca de OpenAI para Python](https://pypi.org/project/openai/)
- [Proyectos de Azure AI para Microsoft .NET](https://www.nuget.org/packages/Azure.AI.Projects)
- [Biblioteca cliente de Azure OpenAI para Microsoft .NET](https://www.nuget.org/packages/Azure.AI.OpenAI)
- [Proyectos de Azure AI para JavaScript](https://www.npmjs.com/package/@azure/ai-projects)
- [Biblioteca de Azure OpenAI para TypeScript](https://www.npmjs.com/package/@azure/openai)

Este ejercicio dura aproximadamente **30** minutos.

## Creación de un proyecto de Fundición de IA de Azure

Comencemos con la implementación de un modelo en un proyecto de Fundición de IA de Azure.

1. En un explorador web, abre el [Portal de la Fundición de IA de Azure](https://ai.azure.com) en `https://ai.azure.com` e inicia sesión con tus credenciales de Azure. Cierra las sugerencias o paneles de inicio rápido que se abran la primera vez que inicias sesión y, si es necesario, usa el logotipo de **Fundición de IA de Azure** en la parte superior izquierda para navegar a la página principal, que es similar a la siguiente imagen:

    ![Captura de pantalla del Portal de la Fundición de IA de Azure.](../media/ai-foundry-home.png)

1. En la página principal, en la sección **Explorar modelos y funcionalidades**, busca el modelo `Phi-4-multimodal-instruct`, que usaremos en nuestro proyecto.
1. En los resultados de la búsqueda, seleccione el modelo **Phi-4-multimodal-instruct** para ver sus detalles y, a continuación, en la parte superior de la página del modelo, seleccione **Usar este modelo**.
1. Cuando se te pida que crees un proyecto, escribe un nombre válido para el proyecto y expande **Opciones avanzadas**.
1. Selecciona **Personalizar** y especifica la siguiente configuración para el centro:
    - **Recurso de Fundición de IA de Azure**: *un nombre válido para el recurso de Fundición de IA de Azure*
    - **Suscripción**: *suscripción a Azure*
    - **Grupo de recursos**: *crea o selecciona un grupo de recursos*
    - **Región**: *seleccione cualquiera (se recomienda Fundición de IA\*).

    > \* Algunos de los recursos de Azure AI están restringidos por cuotas de modelo regionales. En caso de que se alcance un límite de cuota más adelante en el ejercicio, es posible que tengas que crear otro recurso en otra región. Puede comprobar la disponibilidad regional más reciente de modelos específicos en la [documentación de Fundición de IA de Azure](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-serverless-availability#region-availability).

1. Selecciona **Crear** y espera a que tu proyecto se cree.

    La operación puede tardar unos instantes en completarse.

1. Seleccione **Aceptar y continuar** para aceptar los términos del modelo y, a continuación, seleccione **Implementar** para completar la implementación del modelo Phi.

1. Cuando se cree el proyecto, los detalles del modelo se abrirán automáticamente. Observe el nombre de la implementación del modelo; que debe ser **Phi-4-multimodal-instruct**.

1. En el panel de navegación de la izquierda, selecciona **Información general** para ver la página principal del proyecto; que tiene este aspecto:

    > **Nota**: si se muestra un error de *permisos insuficientes**, usa el botón **Reparar** para resolverlo.

    ![Captura de pantalla de la página de información general de un proyecto de Fundición de IA de Azure.](../media/ai-foundry-project.png)

## Creación de una aplicación cliente

Ahora que ha implementado un modelo, puedes usar los SDK de la Fundición de IA de Azure y de inferencia del modelo de Azure AI para desarrollar una aplicación que chatee con él.

> **Sugerencia**: puedes elegir desarrollar la solución mediante C# de Python o Microsoft. Sigue las instrucciones de la sección adecuada para el idioma elegido.

### Preparación de la configuración de aplicación

1. En el Portal de la Fundición de IA de Azure, mira la página **Información general** del proyecto.
1. En el área **Detalles del proyecto**, anota el **punto de conexión del proyecto de Fundición de IA de Azure**. Usarás este punto de conexión para conectarte al proyecto en una aplicación cliente.
1. Abre una nueva pestaña del explorador (mantén el Portal de la Fundición de IA de Azure abierto en la pestaña existente). En la nueva pestaña, explora [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` e inicia sesión con tus credenciales de Azure, si se te solicita.

    Cierra las notificaciones de bienvenida para ver la página principal de Azure Portal.

1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell*** sin almacenamiento en tu suscripción.

    Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal. Puedes cambiar el tamaño o maximizar este panel para facilitar el trabajo.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    **<font color="red">Asegúrate de que has cambiado a la versión clásica de Cloud Shell antes de continuar.</font>**

1. En el panel de Cloud Shell, escribe los siguientes comandos para clonar el repositorio de GitHub que contiene los archivos de código de este ejercicio (escribe el comando o cópialo en el Portapapeles y, a continuación, haz clic con el botón derecho en la línea de comandos y pega como texto sin formato):

    ```
   rm -r mslearn-ai-audio -f
   git clone https://github.com/MicrosoftLearning/mslearn-ai-language
    ```

    > **Sugerencia**: al pegar comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

1. Una vez clonado el repo, ve a la carpeta que contiene los archivos de código de aplicación:  

    ```
   cd mslearn-ai-language/Labfiles/09-audio-chat/Python
    ````

1. En el panel de la línea de comandos de Cloud Shell, escribe el siguiente comando para instalar las bibliotecas que vas a usar:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-ai-projects openai
    ```

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    ```
   code .env
    ```

    El archivo debería abrirse en un editor de código.

1. En el archivo de código, reemplace el marcador de posición **your_project_endpoint** por el punto de conexión de su proyecto (copiado de la página **Información general** del proyecto en el portal de Fundición de IA de Azure) y el marcador de posición **your_model_deployment** por el nombre que asignó a la implementación del modelo Phi-4-multimodal-instruct.

1. Después de reemplazar los marcadores de posición, en el editor de código, usa el comando **CTRL+S** o **Clic derecho > Guardar** para guardar los cambios y después usa el comando **CTRL+Q** o **Clic derecho > Salir** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

### Escritura de código para conectarte al proyecto y obtener un cliente de chat para el modelo

> **Sugerencia**: al agregar código, asegúrate de mantener la sangría correcta.

1. Escriba el siguiente comando para editar el archivo de código:

    ```
   code audio-chat.py
    ```

1. En el archivo de código, observa las instrucciones existentes que se han agregado en la parte superior del archivo para importar los espacios de nombres de SDK necesarios. Después, busca el comentario **Add references** y agrega el código siguiente para hacer referencia a los espacios de nombres de las bibliotecas que has instalado anteriormente:

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
    ```

1. En la función **principal**, en el comentario **Obtener ajustes de configuración**, observa que el código carga los valores de la cadena de conexión del proyecto y del nombre de implementación del modelo que has definido en el archivo de configuración.

1. Busque el comentario **Inicializar el cliente del proyecto** y agregue el siguiente código para conectar con el proyecto de Fundición de IA de Azure:

    > **Sugerencia**: Ten cuidado de mantener el nivel de sangría correcto del código.

    ```python
   # Initialize the project client
   project_client = AIProjectClient(            
       credential=DefaultAzureCredential(
           exclude_environment_credential=True,
           exclude_managed_identity_credential=True
       ),
       endpoint=project_endpoint,
   )
    ```

1. Busca el comentario **Get a chat client** y agrega el siguiente código para crear un objeto de cliente para chatear con el modelo:

    ```python
   # Get a chat client
   openai_client = project_client.get_openai_client(api_version="2024-10-21")
    ```

### Creación de código para enviar una indicación basada en audio

Antes de enviar el mensaje, es necesario codificar el archivo de audio para la solicitud. Luego, podemos adjuntar los datos de audio al mensaje del usuario con una indicación para el LLM. Tenga en cuenta que el código incluye un bucle para permitir que un usuario escriba una indicación hasta que escriba "quit". 

1. En el comentario **Codificar el archivo de audio**, escriba el código siguiente para preparar el siguiente archivo de audio:

    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/avocados.mp4" title="Una solicitud de aguacates" width="150"></video>

    ```python
   # Encode the audio file
   file_path = "https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Labfiles/09-audio-chat/data/avocados.mp3"
   response = requests.get(file_path)
   response.raise_for_status()
   audio_data = base64.b64encode(response.content).decode('utf-8')
    ```

1. En el comentario **Obtener una respuesta a la entrada de audio**, agregue el código siguiente para enviar una indicación:

    ```python
   # Get a response to audio input
   response = openai_client.chat.completions.create(
       model=model_deployment,
       messages=[
           {"role": "system", "content": system_message},
           { "role": "user",
               "content": [
               { 
                   "type": "text",
                   "text": prompt
               },
               {
                   "type": "input_audio",
                   "input_audio": {
                       "data": audio_data,
                       "format": "mp3"
                   }
               }
           ] }
       ]
   )
   print(response.choices[0].message.content)
    ```

1. Usa el comando **CTRL+S** para guardar los cambios en el archivo de código. También puedes cerrar el editor de código (**CTRL+Q**) si lo deseas.

### Inicie sesión en Azure y ejecuta la aplicación.

1. En el panel de línea de comandos de Cloud Shell, escribe el siguiente comando para iniciar sesión en Azure.

    ```
   az login
    ```

    **<font color="red">Debes iniciar sesión en Azure, aunque la sesión de Cloud Shell ya esté autenticada.</font>**

    > **Nota**: en la mayoría de los escenarios, el uso de *inicio de sesión de az* será suficiente. Sin embargo, si tienes suscripciones en varios inquilinos, es posible que tengas que especificar el inquilino mediante el parámetro *--tenant*. Consulta [Inicio de sesión en Azure de forma interactiva mediante la CLI de Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obtener más información.
    
1. Cuando se te solicite, sigue las instrucciones para abrir la página de inicio de sesión en una nueva pestaña y escribe el código de autenticación proporcionado y las credenciales de Azure. A continuación, completa el proceso de inicio de sesión en la línea de comandos y selecciona la suscripción que contiene el centro de Fundición de IA de Azure si se te solicita.

1. En el panel de línea de comandos de Cloud Shell, escribe el siguiente comando para ejecutar la aplicación:

    ```
   python audio-chat.py
    ```

1. Cuando se te solicite, escribe la indicación 

    ```
   Can you summarize this customer's voice message?
    ```

1. Revise la respuesta.

### Usa otro archivo de audio

1. En el editor de código de la aplicación, busque el código que ha agregado anteriormente en el comentario **Codificar el archivo de audio**. A continuación, modifique la dirección URL de la ruta de acceso del archivo como se indica a continuación para usar otro archivo de audio para la solicitud (dejando el código existente después de la ruta de acceso al archivo):

    ```python
   # Encode the audio file
   file_path = "https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Labfiles/09-audio-chat/data/fresas.mp3"
    ```

    El nuevo archivo suena como este:

    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/fresas.mp4" title="Una solicitud de fresas" width="150"></video>

 1. Usa el comando **CTRL+S** para guardar los cambios en el archivo de código. También puedes cerrar el editor de código (**CTRL+Q**) si lo deseas.

1. En el panel de línea de comandos de Cloud Shell, debajo del editor de código, escribe el siguiente comando para ejecutar la aplicación:

    ```
   python audio-chat.py
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
