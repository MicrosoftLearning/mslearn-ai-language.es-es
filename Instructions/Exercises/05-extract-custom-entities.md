---
lab:
  title: Extracción de entidades personalizadas
  module: Module 3 - Getting Started with Natural Language Processing
---

# Extracción de entidades personalizadas

Además de otras funcionalidades de procesamiento de lenguaje natural, el servicio Lenguaje de Azure AI permite definir entidades personalizadas y extraer instancias de ellas a partir de texto.

Para probar la extracción de entidades personalizadas, se creará un modelo y se entrenará mediante Azure AI Language Studio y, después, se usará una aplicación de línea de comandos para probarlo.

## Aprovisionar un recurso de *Lenguaje de Azure AI*

Si aún no tiene uno en su suscripción, deberá aprovisionar un recurso del servicio **Lenguaje de Azure AI**. Además, use la clasificación de texto personalizada, debe habilitar la característica **Clasificación y extracción de texto personalizado**.

1. En un explorador, abra Azure Portal en `https://portal.azure.com` e inicie sesión en la cuenta de Microsoft.
1. Seleccione el botón **Crear un recurso**, busque *Lenguaje* y cree un recurso del **Servicio Language**. Cuando esté en la página *Seleccionar características adicionales*, seleccione la característica personalizada que contiene **Extracción de reconocimiento de entidades con nombre personalizado**. Cree el recurso con los valores siguientes:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *seleccione o cree un grupo de recursos*
    - **Región**: *elija cualquier región disponible*
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa**: seleccione **F0** (*gratis*) o **S** (*estándar*) si F no está disponible.
    - **Cuenta de almacenamiento**: nueva cuenta de almacenamiento.:
      - **Nombre de la cuenta de almacenamiento**: *escriba un nombre único*.
      - **Tipo de cuenta de almacenamiento**: LRS estándar.
    - **Aviso de IA responsable**: seleccionado.

1. Seleccione **Revisar y crear** y **Crear** para aprovisionar el recurso.
1. Espere a que se complete la implementación y, a continuación, vaya al recurso implementado.
1. Consulte la página **Claves y punto de conexión**. Necesitará la información de esta página más adelante en el ejercicio.

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

## Etiquetado de los datos

Ahora que se ha creado el proyecto, debe etiquetar los datos para entrenar el modelo sobre cómo identificar las entidades.

1. Si la página **Etiquetado de datos** aún no está abierta, en el panel de la izquierda, seleccione **Etiquetado de datos**. Verá una lista de los archivos que ha cargado en la cuenta de almacenamiento.
1. En el lado derecho, en el panel **Actividad**, seleccione **Agregar entidad** y agregue una nueva entidad llamada `ItemForSale`.
1.  Repita el paso anterior para crear las entidades siguientes:
    - `Price`
    - `Location`
1. Después de crear las tres entidades, seleccione **Ad 1.txt** para que pueda leerlo.
1. En *Ad 1.txt*: 
    1. Resalte el texto *face cord of firewood* y seleccione la entidad **ItemForSale**.
    1. Resalte el texto *Denver, CO* y seleccione la entidad **Location**.
    1. Resalte el texto *$90* y seleccione la entidad **Price**.
1. En el panel **Actividad**, tenga en cuenta que este documento se agregará al conjunto de datos para entrenar el modelo.
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

## Preparación para desarrollar una aplicación en Visual Studio Code

Para probar las funcionalidades de extracción de entidades personalizadas del servicio de Lenguaje de Azure AI, desarrollará una aplicación de consola sencilla en Visual Studio Code.

> **Sugerencia**: Si ya ha clonado el repositorio **mslearn-ai-language**, ábralo en Visual Studio Code. De lo contrario, siga estos pasos para clonarlo en el entorno de desarrollo.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/mslearn-ai-language` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## Configuración de la aplicación

Se han proporcionado aplicaciones para C# y Python. Las dos aplicaciones tienen la misma funcionalidad. Primero, completará algunas partes clave de la aplicación para que pueda usar su recurso de Lenguaje de Azure AI.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **Labfiles/05-custom-entity-recognition** y expanda la carpeta **CSharp** o **Python** según sus preferencias de lenguaje y la carpeta **custom-entities** que contiene. Cada carpeta contiene los archivos específicos del lenguaje de una aplicación en la que va a integrar la funcionalidad de clasificación de Lenguaje de Azure AI.
1. Haga clic con el botón derecho en la carpeta **custom-entities** que contiene sus archivos de código y abra un terminal integrado. Después, instale el paquete del SDK de Text Analytics de Lenguaje de Azure AI mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#**:

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**:

    ```
    pip install azure-ai-textanalytics==5.3.0
    ```

1. En el panel **Explorador**, en la carpeta **custom-entities**, abra el archivo de configuración para su lenguaje preferido.

    - **C#**: appsettings.json
    - **Python**: .env
    
1. Actualice los valores de configuración para incluir el **punto de conexión** y una **clave** del recurso de Lenguaje de Azure que creó (disponible en la página **Claves y punto de conexión** del recurso de Lenguaje de Azure AI en Azure Portal) El archivo ya debe contener los nombres de proyecto e implementación del modelo de extracción de entidades personalizados.
1. Guarde el archivo de configuración.

## Agregar entidades para extraer datos

Ahora tiene todo listo para usar el servicio de Lenguaje de Azure AI para extraer entidades personalizadas del texto.

1. Expanda la carpeta **ads** en la carpeta **custom-entities** para ver los anuncios clasificados que analizará la aplicación.
1. En la carpeta **custom-entities**, abra el archivo de código para la aplicación cliente:

    - **C#**: Program.cs
    - **Python**: custom-entities.py

1. Busque el comentario **Importar espacios de nombres**. A continuación, en este comentario, agregue el siguiente código específico del lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Text Analytics:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**: custom-entities.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. En la función **Principal**, observe que ya se han proporcionado código para cargar la clave y el punto de conexión del servicio de Lenguaje de Azure AI, así como el nombre del proyecto y de la implementación desde el archivo de configuración. A continuación, busque el comentario **Create client using endpoint and key**(Crear cliente mediante el punto de conexión y la clave) y agregue el código siguiente para crear un cliente para Text Analysis API:

    **C#**: Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credentials = new(aiSvcKey);
    Uri endpoint = new(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new(endpoint, credentials);
    ```

    **Python**: custom-entities.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. En la función **Principal**, observe que el código existente lee todos los archivos de la carpeta **anuncios** y crea una lista con su contenido. En el caso del código de C#, se usa una lista de objetos **TextDocumentInput** para incluir el nombre de archivo como identificador y el lenguaje. En Python se usa una lista sencilla del contenido del texto.
1. Busque el comentario **Extraer entidades** y agregue el código siguiente:

    **C#**: Program.cs

    ```csharp
    // Extract entities
    RecognizeCustomEntitiesOperation operation = await aiClient.RecognizeCustomEntitiesAsync(WaitUntil.Completed, batchedDocuments, projectName, deploymentName);

    await foreach (RecognizeCustomEntitiesResultCollection documentsInPage in operation.Value)
    {
        foreach (RecognizeEntitiesResult documentResult in documentsInPage)
        {
            Console.WriteLine($"Result for \"{documentResult.Id}\":");

            if (documentResult.HasError)
            {
                Console.WriteLine($"  Error!");
                Console.WriteLine($"  Document error code: {documentResult.Error.ErrorCode}");
                Console.WriteLine($"  Message: {documentResult.Error.Message}");
                Console.WriteLine();
                continue;
            }

            Console.WriteLine($"  Recognized {documentResult.Entities.Count} entities:");

            foreach (CategorizedEntity entity in documentResult.Entities)
            {
                Console.WriteLine($"  Entity: {entity.Text}");
                Console.WriteLine($"  Category: {entity.Category}");
                Console.WriteLine($"  Offset: {entity.Offset}");
                Console.WriteLine($"  Length: {entity.Length}");
                Console.WriteLine($"  ConfidenceScore: {entity.ConfidenceScore}");
                Console.WriteLine($"  SubCategory: {entity.SubCategory}");
                Console.WriteLine();
            }

            Console.WriteLine();
        }
    }
    ```

    **Python**: custom-entities.py

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

1. Guarde los cambios en el archivo de código.

## Prueba de la aplicación

La aplicación ya se puede probar.

1. En el terminal integrado de la carpeta **classify-text** escriba el siguiente comando para ejecutar el programa:

    - **C#**: `dotnet run`
    - **Python**: `python custom-entities.py`

    > **Sugerencia**: Puede usar el icono **Maximizar el tamaño del panel** (**^**) en la barra de herramientas del terminal para ver más del texto de la consola.

1. Observe la salida. La aplicación debe enumerar los detalles de las entidades que se encuentran en cada archivo de texto.

## Limpiar

Cuando ya no necesite el proyecto, puede eliminarlo desde la página de **proyectos** en Language Studio. También puede quitar el servicio de Lenguaje de Azure AI y la cuenta de almacenamiento asociada en [Azure Portal](https://portal.azure.com).
