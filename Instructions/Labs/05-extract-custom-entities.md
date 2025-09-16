---
lab:
  title: Extracción de entidades personalizadas
  description: Entrenar un modelo para extraer entidades personalizadas de la entrada de texto mediante Lenguaje de Azure AI.
---

# Extracción de entidades personalizadas

Además de otras funcionalidades de procesamiento de lenguaje natural, el servicio Lenguaje de Azure AI permite definir entidades personalizadas y extraer instancias de ellas a partir de texto.

Para probar la extracción de entidades personalizadas, se creará un modelo y se entrenará mediante Lenguaje de Azure AI Studio y, después, se usará una aplicación de línea de comandos para probarlo.

Aunque este ejercicio se basa en Python, puede desarrollar aplicaciones de clasificación de texto mediante varios SDK específicos del lenguaje, como:

- [Biblioteca cliente de Azure AI Text Analytics para Python](https://pypi.org/project/azure-ai-textanalytics/)
- [Biblioteca cliente de Azure AI Text Analytics para .NET](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [Biblioteca cliente de Azure AI Text Analytics para JavaScript](https://www.npmjs.com/package/@azure/ai-text-analytics)

Este ejercicio dura aproximadamente **35** minutos.

## Aprovisionar un recurso de *Lenguaje de Azure AI*

Si aún no tiene uno en su suscripción, deberá aprovisionar un recurso del servicio **Lenguaje de Azure AI**. Además, use la clasificación de texto personalizada, debe habilitar la característica **Clasificación y extracción de texto personalizado**.

1. En un explorador, abra Azure Portal en `https://portal.azure.com` e inicie sesión en la cuenta de Microsoft.
1. Seleccione el botón **Crear un recurso**, busque *Lenguaje* y cree un recurso del **Servicio Language**. Cuando esté en la página *Seleccionar características adicionales*, seleccione la característica personalizada que contiene **Extracción de reconocimiento de entidades con nombre personalizado**. Cree el recurso con los valores siguientes:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *seleccione o cree un grupo de recursos*
    - **Región**: *selecciona una de las siguientes regiones*\*
        - Este de Australia
        - Centro de la India
        - Este de EE. UU.
        - Este de EE. UU. 2
        - Norte de Europa
        - Centro-sur de EE. UU.
        - Norte de Suiza
        - Sur de Reino Unido
        - Oeste de Europa
        - Oeste de EE. UU. 2
        - Oeste de EE. UU. 3
    - **Nombre**: *escribe un nombre único*
    - **Plan de tarifa**: seleccione **F0** (*gratis*) o **S** (*estándar*) si F no está disponible.
    - **Cuenta de almacenamiento**: nueva cuenta de almacenamiento.:
      - **Nombre de la cuenta de almacenamiento**: *escriba un nombre único*.
      - **Tipo de cuenta de almacenamiento**: LRS estándar.
    - **Aviso de IA responsable**: seleccionado.

1. Seleccione **Revisar y crear** y **Crear** para aprovisionar el recurso.
1. Espera a que se complete la implementación y, a continuación, ve al recurso implementado.
1. Consulta la página **Claves y punto de conexión**. Necesitarás la información de esta página más adelante en el ejercicio.

## Configuración del acceso basado en roles para el usuario

> **NOTA**: si omites este paso, obtendrás el error 403 al intentar conectarte a tu proyecto personalizado. Es importante que el usuario actual tenga este rol para acceder a los datos del blob de la cuenta de almacenamiento, incluso si es el propietario de la cuenta de almacenamiento.

1. Vaya a la página de la cuenta de almacenamiento en Azure Portal.
2. Seleccione **Control de acceso (IAM)** en el menú de navegación izquierdo.
3. Selecciona **Agregar** para Agregar asignaciones de roles y elige el rol **colaborador de datos de blob de almacenamiento** en la cuenta de almacenamiento.
4. En **Asignar acceso a**, seleccione **Usuario, grupo o entidad de servicio**.
5. Elija **Seleccionar miembros**.
6. Seleccione su usuario. Puede buscar por nombres de usuario en el campo **Seleccionar**.

## Carga de anuncios de ejemplo

Una vez que haya creado el servicio de Lenguaje de Azure AI y la cuenta de almacenamiento, tendrá que cargar anuncios de ejemplo para entrenar el modelo más adelante.

1. En una nueva pestaña del explorador, descargue anuncios clasificados de ejemplo de `https://aka.ms/entity-extraction-ads` y extraiga los archivos en una carpeta de su elección.

2. En Azure Portal, navegue hasta la cuenta de almacenamiento que ha creado y selecciónela.

3. En la cuenta de almacenamiento, seleccione **Configuración**, que se encuentra junto a **Configuración** y habilite la opción **Permitir el acceso anónimo de blobs** y, a continuación, seleccione **Guardar**.

4. Seleccione **Contenedores** en el menú de la izquierda, que se encuentra debajo de **Almacenamiento de datos**. En la pantalla que aparece, seleccione **+ Contenedor**. Denomine al contenedor `classifieds` y establezca el **Nivel de acceso anónimo** en **Contenedor (acceso de lectura anónimo para contenedores y blobs)**.

    > **NOTA**: Al configurar una cuenta de almacenamiento para una solución real, tenga cuidado de asignar el nivel de acceso adecuado. Para obtener más información sobre cada nivel de acceso, consulte la [documentación sobre Azure Storage](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure).

5. Después de crear el contenedor, selecciónelo y haga clic en el botón **Cargar** y cargue los anuncios de ejemplo que ha descargado.

## Creación de un proyecto de reconocimiento de entidades con nombre personalizado

Ahora está listo para crear un proyecto de reconocimiento de entidades con nombre personalizado. Este proyecto proporciona un lugar de trabajo para compilar, entrenar e implementar el modelo.

> **NOTA**: También puede crear, compilar, entrenar e implementar el modelo mediante la API de REST.

1. En una pestaña nueva del explorador, abra el portal de Azure AI Language Studio en `https://language.cognitive.azure.com/` e inicie sesión con la cuenta de Microsoft asociada a su suscripción de Azure.
1. Si se le pide que elija un recurso de Language, seleccione la configuración siguiente:

    - **Directorio de Azure**: directorio de Azure que contiene la suscripción.
    - **Suscripción de Azure**: su suscripción a Azure.
    - **Tipo de recurso**: idioma.
    - **Recurso de lenguaje**: el recurso de Lenguaje de Azure AI que creó antes.

    Si <u>no</u> se le pide que elija un recurso de Language, puede deberse a que tiene varios recursos de Language en la suscripción, en cuyo caso:

    1. En la barra de la parte superior de la página, seleccione el botón**Configuración (&#9881;)**.
    2. En la página **Configuración,** vea la pestaña **Recursos**.
    3. Seleccione el recurso de Language que acaba de crear y haga clic en **Switch resource** (Cambiar recurso).
    4. En la parte superior de la página, haga clic en **Language Studio** para volver a la página principal de Language Studio.

1. En la parte superior del portal, en el menú **Crear nuevo**, seleccione **Reconocimiento de entidades con nombre personalizado**.

1. Cree un proyecto con la siguiente configuración:
    - **Conectar almacenamiento**: *es probable que este valor ya esté rellenado. Cámbielo a su cuenta de almacenamiento si no se ha cambiado todavía*
    - **Información básica:**
    - **Nombre**: `CustomEntityLab`
        - **Idioma principal del texto**: inglés (US)
        - **¿Su conjunto de datos incluye documentos que no están en la misma lengua?** *No*
        - **Descripción**: `Custom entities in classified ads`
    - **Contenedor**:
        - **Contenedor de almacenamiento de blobs**: clasificados
        - **¿Los archivos están etiquetados con clases?**: No, necesito etiquetar mis archivos como parte de este proyecto.

> **Sugerencia**: si recibes un error sobre que no estás autorizado para realizar esta operación, tendrás que agregar una asignación de roles. Para solucionarlo, agregamos el rol "Colaborador de datos de blob de almacenamiento" en la cuenta de almacenamiento para el usuario que ejecuta el laboratorio. Encontrarás más información [en la página de documentación](https://learn.microsoft.com/azure/ai-services/language-service/custom-named-entity-recognition/how-to/create-project?tabs=portal%2Clanguage-studio#enable-identity-management-for-your-resource).

## Etiquetado de los datos

Ahora que se ha creado el proyecto, debe etiquetar los datos para entrenar el modelo sobre cómo identificar las entidades.

1. Si la página **Etiquetado de datos** aún no está abierta, en el panel de la izquierda, seleccione **Etiquetado de datos**. Verá una lista de los archivos que ha cargado en la cuenta de almacenamiento.
1. En el lado derecho, en el panel **Actividad**, seleccione **Agregar entidad** y agregue una nueva entidad llamada `ItemForSale`.
1.  Repite el paso anterior para crear las entidades siguientes:
    - `Price`
    - `Location`
1. Después de crear las tres entidades, seleccione **Ad 1.txt** para que pueda leerlo.
1. En *Ad 1.txt*: 
    1. Resalte el texto *face cord of firewood* y seleccione la entidad **ItemForSale**.
    1. Resalte el texto *Denver, CO* y seleccione la entidad **Location**.
    1. Resalte el texto *$90* y seleccione la entidad **Price**.
1. En el panel **Actividad**, ten en cuenta que este documento se agregará al conjunto de datos para entrenar el modelo.
1. Use el botón **Siguiente documento** para pasar al siguiente documento y continúe asignando texto a las entidades adecuadas para todo el conjunto de documentos, agregándolos todos al conjunto de datos de entrenamiento.
1. Cuando haya etiquetado el último documento (*Ad 9.txt*), guarde las etiquetas.

## Entrenamiento de un modelo

Después de etiquetar los datos, debe entrenar el modelo.

1. En el panel de la izquierda, seleccione **Trabajos de entrenamiento**.
2. Seleccione **Iniciar un trabajo de entrenamiento**
3. Entrene un nuevo modelo denominado `ExtractAds`
4. Elija **Automatically split the testing set from training data** (Dividir automáticamente el conjunto de pruebas de los datos de entrenamiento).

    > **SUGERENCIA**: En proyectos de extracción propios, use la división de pruebas que mejor se adapte a los datos. Para datos más coherentes y conjuntos de datos más grandes, el servicio Lenguaje de Azure AI dividirá automáticamente las pruebas establecidas por porcentaje. Con conjuntos de datos más pequeños, es importante realizar el entrenamiento con la variedad adecuada de documentos de entrada posibles.

5. Haga clic en **Entrenar**.

    > **IMPORTANTE**: El entrenamiento del modelo a veces puede tardar varios minutos. Recibirá una notificación cuando se complete.

## Evaluación del modelo

En las aplicaciones reales, es importante evaluar y mejorar el modelo para comprobar que funciona según lo previsto. En dos páginas de la izquierda se muestran los detalles del modelo entrenado y las pruebas con errores.

Seleccione **Rendimiento del modelo** en el menú izquierdo y seleccione el modelo `ExtractAds`. Puede ver la puntuación del modelo, las métricas de rendimiento y cuándo se ha entrenado. Podrá ver si se ha producido un error en los documentos de prueba y estos errores le ayudarán a comprender dónde mejorar.

## Implementación del modelo

Cuando esté satisfecho con el entrenamiento del modelo, es el momento de implementarlo, lo que le permite empezar a extraer entidades mediante la API.

1. En el panel de la izquierda, seleccione **Implementar un modelo**.
2. Seleccione **Agregar implementación** y, después, escriba el nombre `AdEntities` y seleccione el modelo **ExtractAds**.
3. Haga clic en **Implementar** para implementar el modelo..

## Preparación para desarrollar una aplicación en Cloud Shell

Para probar las funcionalidades de extracción de entidades personalizadas del servicio Lenguaje de Azure AI, desarrollará una aplicación de consola sencilla en Azure Cloud Shell.

1. En Azure Portal, usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    **<font color="red">Asegúrate de que has cambiado a la versión clásica de Cloud Shell antes de continuar.</font>**

1. En el panel de PowerShell, escribe los siguientes comandos para clonar el repo de GitHub para este ejercicio:

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **Sugerencia**: al pegar comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.
    ```

1. After the repo has been cloned, navigate to the folder containing the application code files:  

    ```
    cd mslearn-ai-language/Labfiles/05-custom-entity-recognition/Python/custom-entities
    ```

## Configure your application

1. In the command line pane, run the following command to view the code files in the **custom-entities** folder:

    ```
   ls -a -l
    ```

    The files include a configuration file (**.env**) and a code file (**custom-entities.py**). The text your application will analyze is in the **ads** subfolder.

1. Create a Python virtual environment and install the Azure AI Language Text Analytics SDK package and other required packages by running the following command:

    ```
   python -m venv labenv ./labenv/bin/Activate.ps1 pip install -r requirements.txt azure-ai-textanalytics==5.3.0
    ```

1. Enter the following command to edit the application configuration file:

    ```
   code .env
    ```

    The file is opened in a code editor.

1. Update the configuration values to include the  **endpoint** and a **key** from the Azure Language resource you created (available on the **Keys and Endpoint** page for your Azure AI Language resource in the Azure portal).The file should already contain the project and deployment names for your custom entity extraction model.
1. After you've replaced the placeholders, within the code editor, use the **CTRL+S** command or **Right-click > Save** to save your changes and then use the **CTRL+Q** command or **Right-click > Quit** to close the code editor while keeping the cloud shell command line open.

## Add code to extract entities

1. Enter the following command to edit the application code file:

    ```
    code custom-entities.py
    ```

1. Review the existing code. You will add code to work with the AI Language Text Analytics SDK.

    > **Tip**: As you add code to the code file, be sure to maintain the correct indentation.

1. At the top of the code file, under the existing namespace references, find the comment **Import namespaces** and add the following code to import the namespaces you will need to use the Text Analytics SDK:

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. En la función **main**, observe que ya se han proporcionado el código para cargar el punto de conexión y la clave del servicio Lenguaje de Azure AI y los nombres de proyecto e implementación del archivo de configuración. A continuación, busque el comentario **Crear cliente mediante el punto de conexión y la clave** y agregue el código siguiente para crear un cliente de análisis de texto:

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Observe que el código existente lee todos los archivos de la carpeta **ads** y crea una lista con su contenido. Busque el comentario **Extraer entidades** y agregue el código siguiente:

    ```Python
   # Extract entities
   operation = ai_client.begin_recognize_custom_entities(
        batchedDocuments,
        project_name=project_name,
        deployment_name=deployment_name
   )

   document_results = operation.result()

   for doc, custom_entities_result in zip(files, document_results):
        print(doc)
        if custom_entities_result.kind == "CustomEntityRecognition":
            for entity in custom_entities_result.entities:
                print(
                    "\tEntity '{}' has category '{}' with confidence score of '{}'".format(
                        entity.text, entity.category, entity.confidence_score
                    )
                )
        elif custom_entities_result.is_error is True:
            print("\tError with code '{}' and message '{}'".format(
                custom_entities_result.error.code, custom_entities_result.error.message
                )
            )
    ```

1. Guarde los cambios (CTRL+S) y, a continuación, escriba el siguiente comando para ejecutar el programa (maximice el panel de Cloud Shell y cambie el tamaño de los paneles para ver más texto en el panel de línea de comandos):

    ```
   python custom-entities.py
    ```

1. Observe la salida. La aplicación debe enumerar los detalles de las entidades que se encuentran en cada archivo de texto.

## Limpiar

Cuando ya no necesite el proyecto, puede eliminarlo desde la página de **proyectos** en Language Studio. También puede quitar el servicio de Lenguaje de Azure AI y la cuenta de almacenamiento asociada en [Azure Portal](https://portal.azure.com).
