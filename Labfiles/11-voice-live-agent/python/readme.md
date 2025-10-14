# Requisitos

## Ejecución en Cloud Shell

* Suscripción de Azure con acceso a OpenAI
* Si se ejecuta en Azure Cloud Shell, elija el shell de Bash. La CLI de Azure y Azure Developer CLI se incluyen en Cloud Shell.

## Ejecución en modo local

* Puede ejecutar la aplicación web localmente después de ejecutar el script de implementación:
    * [Azure Developer CLI (azd)](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/install-azd)
    * [CLI de Azure](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
    * Suscripción de Azure con acceso a OpenAI


## Variables de entorno

El archivo `.env` se crea mediante el script *azdeploy.sh*. El punto de conexión del modelo de IA, la clave de API y el nombre del modelo se agregan durante la implementación de los recursos.

## Implementación de recursos de Azure

El script `azdeploy.sh` proporcionado crea los recursos necesarios en Azure:

* Cambie las dos variables de la parte superior del script para que coincidan con sus necesidades; no cambie nada más.
* El script:
    * Implementa el modelo *gpt-4o* mediante AZD.
    * Crea el servicio Azure Container Registry.
    * Usa tareas de ACR para compilar e implementar la imagen de Dockerfile en ACR.
    * Crea el plan de App Service.
    * Crea la aplicación web de App Service.
    * Configura la aplicación web para la imagen de contenedor en ACR.
    * Configura las variables de entorno de la aplicación web.
    * El script proporcionará el punto de conexión de App Service.

El script proporciona dos opciones de implementación: 1. Implementación completa; y 2. Implementación repetida solo de la imagen. La opción 2 es solo para después de la implementación cuando desea experimentar con cambios en la aplicación. 

> Nota: Puede ejecutar el script en PowerShell o Bash mediante el comando `bash azdeploy.sh`; este comando también le permite ejecutar el script en Bash sin tener que convertirlo en un ejecutable.

## Desarrollo local

### Aprovisionamiento del modelo de IA en Azure

Puede ejecutar el proyecto localmente y aprovisionar únicamente el modelo de IA siguiendo estos pasos:

1. **Inicialice el entorno** (elija un nombre descriptivo):

   ```bash
   azd env new gpt-realtime-lab --confirm
   # or: azd env new your-name-gpt-experiment --confirm
   ```
   
   **Importante**: Este nombre se convierte en parte de los nombres de recursos de Azure.  
   La marca `--confirm` establece este entorno como predeterminado sin preguntar.

1. **Establezca el grupo de recursos**:

   ```bash
   azd env set AZURE_RESOURCE_GROUP "rg-your-name-gpt"
   ```

1. **Inicie sesión y aprovisione los recursos de IA**:

   ```bash
   az login
   azd provision
   ```

    > **Importante**: NO ejecute `azd deploy`; la aplicación no está configurada en las plantillas de AZD.

Si solo aprovisionó el modelo mediante el método `azd provision`, DEBE crear un archivo `.env` en la raíz del directorio con las siguientes entradas:

```
AZURE_VOICE_LIVE_ENDPOINT=""
AZURE_VOICE_LIVE_API_KEY=""
VOICE_LIVE_MODEL=""
VOICE_LIVE_VOICE="en-US-JennyNeural"
VOICE_LIVE_INSTRUCTIONS="You are a helpful AI assistant with a focus on world history. Respond naturally and conversationally. Keep your responses concise but engaging."
VOICE_LIVE_VERBOSE="" #Suppresses excessive logging to the terminal if running locally
```

Notas:

1. El punto de conexión es el punto de conexión del modelo y solo debe incluir `https://<proj-name>.cognitiveservices.azure.com`.
1. La clave de API es la clave del modelo.
1. El modelo es el nombre del modelo usado durante la implementación.
1. Puede recuperar estos valores del portal de Fundición de IA.

### Ejecución local del proyecto

El proyecto se creó y administró mediante **uv**, pero no es necesario ejecutarlo. 

Si tiene instalado **uv**:

* Ejecute `uv venv` para crear el entorno.
* Ejecute `uv sync` para agregar paquetes.
* Alias creado para la aplicación web: `uv run web` para iniciar el script `flask_app.py`.
* Archivo requirements.txt creado con `uv pip compile pyproject.toml -o requirements.txt`.

Si no tiene instalado **uv**:

* Crear entorno: `python -m venv .venv`
* Activar entorno: `.\.venv\Scripts\Activate.ps1`
* Instalar dependencias: `pip install -r requirements.txt`
* Ejecutar aplicación (desde la raíz del proyecto): `python .\src\real_time_voice\flask_app.py`
