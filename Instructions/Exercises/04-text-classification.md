---
lab:
  title: Clasificación de texto personalizada
  module: Module 3 - Getting Started with Natural Language Processing
---

# Clasificación de texto personalizada

Lenguaje de Azure AI proporciona varias capacidades de PLN, incluida la identificación de frases clave, el resumen de texto y el análisis de sentimiento. El servicio de lenguaje también proporciona características personalizadas, como la respuesta a preguntas personalizada y la clasificación de texto personalizado.

Para probar la clasificación de texto personalizado del servicio de Lenguaje de Azure AI, configuraremos el modelo mediante Language Studio y, después, usaremos una pequeña aplicación de línea de comandos que se ejecuta en Cloud Shell para probarlo. El mismo patrón y funcionalidad que se usa aquí se puede seguir para las aplicaciones del mundo real.

## Aprovisionar un recurso de *Lenguaje de Azure AI*

Si aún no tiene uno en su suscripción, deberá aprovisionar un recurso del **servicio de Lenguaje de Azure AI**. Además, use la clasificación de texto personalizado, debe habilitar la característica **Clasificación y extracción de texto personalizado**.

1. En un explorador, abra Azure Portal en `https://portal.azure.com` e inicie sesión con su cuenta de Microsoft.
1. Seleccione el campo de búsqueda en la parte superior del portal, busque `Azure AI services` y cree un recurso de **servicio de lenguaje.**
1. Seleccione el cuadro que incluye **clasificación de texto personalizado**. Después, seleccione **Continuar para crear el recurso**.
1. Cree un recurso con los valores siguientes:
    - **Suscripción**: *su suscripción a Azure*.
    - **Grupo de recursos**: *seleccione o cree un grupo de recursos*.
    - **Región**: *elija cualquier región disponible*:
    - **Nombre**: *escriba un nombre único*.
    - **Plan de tarifa**: seleccione **F0** (*gratis*), o **S** (*estándar*) si F no está disponible.
    - **Cuenta de almacenamiento**: nueva cuenta de almacenamiento.
      - **Nombre de la cuenta de almacenamiento**: *escriba un nombre único*.
      - **Tipo de cuenta de almacenamiento**: LRS estándar.
    - **Aviso de IA responsable**: seleccionado.

1. Seleccione **Revisar y crear** y **Crear** para aprovisionar el recurso.
1. Espere a que se complete la implementación y, a continuación, vaya al recurso implementado.
1. Consulte la página **Claves y punto de conexión**. Necesitará la información de esta página más adelante en el ejercicio.

## Carga de artículos de muestra

Una vez que haya creado el servicio de Lenguaje de Azure AI y la cuenta de almacenamiento, deberá cargar artículos de ejemplo para entrenar el modelo más adelante.

1. En una nueva pestaña del explorador, descargue artículos de ejemplo de `https://aka.ms/classification-articles` y extraiga los archivos en una carpeta de su elección.

1. En Azure Portal, navegue hasta la cuenta de almacenamiento que ha creado y selecciónela.

1. En la cuenta de almacenamiento, seleccione **Configuración**, que se encuentra bajo **Configuración**. En la pantalla Configuración, habilite la opción **Permitir el acceso anónimo de blobs** y, a continuación, seleccione **Guardar**.

1. Seleccione **Contenedores** en el menú de la izquierda, que se encuentra debajo de **Almacenamiento de datos**. En la pantalla que aparece, seleccione **+ Contenedor**. Denomine al contenedor `articles` y establezca el **Nivel de anónimo** en **Contenedor (acceso de lectura anónimo para contenedores y blobs)**.

    > **NOTA**: Al configurar una cuenta de almacenamiento para una solución real, tenga cuidado de asignar el nivel de acceso adecuado. Para más información sobre cada nivel de acceso, consulte la [documentación sobre Azure Storage](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure).

1. Después de crear el contenedor, selecciónelo y, a continuación, seleccione el botón **Cargar**. Seleccione **Buscar archivos** para buscar los artículos de ejemplo que ha descargado. Después, seleccione **Cargar**.

## Crear un proyecto de clasificación de texto personalizado

Una vez completada la configuración, cree un proyecto de clasificación de texto personalizado. Este proyecto proporciona un lugar de trabajo para compilar, entrenar e implementar el modelo.

> **NOTA:** Este laboratorio usa **Language Studio**, pero también puede crear, compilar, entrenar e implementar el modelo a través de la API de REST.

1. En una pestaña nueva del explorador, abra el portal de Azure AI Language Studio en `https://language.cognitive.azure.com/` e inicie sesión con la cuenta de Microsoft asociada a su suscripción de Azure.
1. Si se le pide que elija un recurso de Language, seleccione la configuración siguiente:

    - **Directorio de Azure**: directorio de Azure que contiene la suscripción.
    - **Suscripción de Azure**: su suscripción a Azure.
    - **Tipo de recurso**: idioma.
    - **Recurso de lenguaje**: el recurso de Lenguaje de Azure AI que creó antes.

    Si <u>no</u> se le pide que elija un recurso de Language, puede deberse a que tiene varios recursos de Language en la suscripción, en cuyo caso:

    1. En la barra de la parte superior de la página, seleccione el botón **Configuración (&#9881;)**.
    2. En la página **Configuración,** vea la pestaña **Recursos**.
    3. Seleccione el recurso de Language que acaba de crear y haga clic en **Switch resource** (Cambiar recurso).
    4. En la parte superior de la página, haga clic en **Language Studio** para volver a la página principal de Language Studio

1. En la parte superior del portal, en el menú **Crear nuevo**, seleccione **Clasificación de texto personalizado**.
1. Aparecerá la página **Conexión con el almacenamiento**. Todos los valores ya se habrán rellenado. Seleccione **Siguiente**.
1. En la página **Seleccionar tipo de proyecto**, seleccione **Clasificación de etiqueta única**. Seleccione **Siguiente**.
1. En el panel **Escribir información básica**, establezca lo siguiente:
    - **Nombre**: `ClassifyLab`  
    - **Idioma principal del texto**: inglés (US)
    - **Descripción**: `Custom text lab`

1. Seleccione **Siguiente**.
1. En la página **Elegir contenedor**, establezca la lista desplegable **Contenedor del almacén de blobs** en el contenedor de *artículos*.
1. Seleccione la opción **No, necesito etiquetar mis archivos como parte de este proyecto**. Seleccione **Siguiente**.
1. Seleccione **Create project** (Crear proyecto).

## Etiquetado de los datos

Ahora que el proyecto está creado, debe etiquetar los datos para entrenar el modelo en cómo clasificar texto.

1. A la izquierda, seleccione **Etiquetado de datos**, si aún no está seleccionado. Verá una lista de los archivos que ha cargado en la cuenta de almacenamiento.
1. En el lado derecho, en el panel **Actividad**, seleccione **+ Agregar clase**.  Los artículos de este laboratorio se dividen en cuatro clases que deberá crear: `Classifieds`, `Sports`, `News`, y `Entertainment`.

    ![Captura de pantalla en la que se muestra la página de etiquetado de datos y el botón de adición de clases.](../media/tag-data-add-class-new.png#lightbox)

1. Después de crear las cuatro clases, seleccione **Artículo 1** para empezar. Aquí puede leer el artículo, definir a qué clase pertenece el archivo y a qué conjunto de datos (de entrenamiento o de prueba) asignarlo.
1. Asigne a cada artículo la clase y el conjunto de datos adecuados (de entrenamiento o de prueba) mediante el panel **Actividad** de la derecha.  Puede seleccionar una etiqueta de la lista de etiquetas de la derecha y establecer cada artículo en **entrenamiento** o **prueba** mediante las opciones de la parte inferior del panel Actividad. Seleccione **Siguiente documento** para pasar al documento siguiente. Para los fines de este laboratorio, definiremos cuáles se usarán para entrenar el modelo y cuáles se usarán para probar el modelo:

    | Artículo  | Clase  | Dataset  |
    |---------|---------|---------|
    | Artículo 1 | Deportes | Cursos |
    | Artículo 10 | Novedades | Cursos |
    | Artículo 11 | Entretenimiento | Prueba |
    | Artículo 12 | Novedades | Prueba |
    | Artículo 13 | Deportes | Prueba |
    | Artículo 2 | Deportes | Cursos |
    | Artículo 3 | Clasificados | Cursos |
    | Artículo 4 | Clasificados | Cursos |
    | Artículo 5 | Entretenimiento | Cursos |
    | Artículo 6 | Entretenimiento | Cursos |
    | Artículo 7 | Novedades | Cursos |
    | Artículo 8 | Novedades | Cursos |
    | Artículo 9 | Entretenimiento | Cursos |

    > **NOTA** Los archivos de Language Studio se enumeran alfabéticamente, por lo que la lista anterior no está en orden secuencial. Asegúrese de visitar ambas páginas de documentos al etiquetar los artículos.

1. Seleccione **Guardar etiquetas** para guardar las etiquetas.

## Entrenamiento de un modelo

Después de etiquetar los datos, debe entrenar el modelo.

1. Seleccione **Trabajos de entrenamiento** en el menú de la izquierda.
1. Seleccione **Iniciar un trabajo de entrenamiento**.
1. Entrene un nuevo modelo denominado `ClassifyArticles`.
1. Seleccione **Usar una división manual de datos de entrenamiento y pruebas**.

    > **SUGERENCIA** En sus propios proyectos de clasificación, el servicio de Lenguaje de Azure AI dividirá de forma automática el conjunto de pruebas establecido por porcentaje, lo que resulta útil para un conjunto de datos grande. Con conjuntos de datos más pequeños, es importante entrenar con la distribución de clase correcta.

1. Seleccione **Entrenar**.

> **IMPORTANTE** Entrenar el modelo a veces puede tardar varios minutos. Recibirá una notificación cuando se complete.

## Evaluación del modelo

En las aplicaciones reales de clasificación de texto, es importante evaluar y mejorar el modelo para comprobar que funciona según lo previsto.

1. Seleccione **Rendimiento del modelo** y seleccione su modelo **ClassifyArticles**. Puede ver la puntuación del modelo, las métricas de rendimiento y cuándo se ha entrenado. Si la puntuación del modelo no es del 100 %, esto significa que uno de los documentos usados para las pruebas no se ha evaluado de la manera en que se ha etiquetado. Estos errores pueden ayudarle a comprender dónde mejorar.
1. Seleccione la pestaña **Detalles del conjunto de pruebas**. Si hay algún error, esta pestaña le permite ver los artículos que haya definido para las pruebas, de qué manera los predijo el modelo, y por último, si el resultado entra en conflicto con su etiqueta de prueba. El comportamiento predeterminado de la pestaña es mostrar solo predicciones incorrectas. Puede alternar la opción **Mostrar solo incoherencias** para ver todos los artículos que ha definido para las pruebas y cuál fue la predicción para cada uno de ellos.

## Implementación del modelo

Cuando esté satisfecho con el entrenamiento del modelo, es el momento de implementarlo, lo que le permite empezar a clasificar el texto a través de la API.

1. En el panel de la izquierda, seleccione **Modelo de implementación**.
1. Seleccione **Agregar implementación**, escriba `articles` en el campo **Crear un nuevo nombre de implementación**, y seleccione **ClassifyArticles** en el campo **Modelo**.
1. Seleccione **Implementar** para implementar el modelo.
1. Una vez implementado el modelo, deje esa página abierta. Necesitará el proyecto y el nombre de la implementación en el paso siguiente.

## Preparación para desarrollar una aplicación en Visual Studio Code

Para probar las capacidades de clasificación de texto personalizadas del servicio Lenguaje de Azure AI, desarrollará una aplicación de consola sencilla en Visual Studio Code.

> **Sugerencia**: Si ya ha clonado el repositorio **mslearn-ai-language**, ábralo en Visual Studio Code. De lo contrario, siga estos pasos para clonarlo en su entorno de desarrollo.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/mslearn-ai-language` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## Configuración de la aplicación

Se han proporcionado aplicaciones para C# y Python, así como un archivo de texto de ejemplo que usará para probar el resumen. Las dos aplicaciones tienen la misma funcionalidad. Primero, completará algunas partes clave de la aplicación para habilitar que utilice el recurso de Lenguaje de Azure AI.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **Labfiles/04-text-classification** y expanda la carpeta **CSharp** o **Python** según sus preferencias de lenguaje y la carpeta **classify-text** que contiene. Cada carpeta contiene los archivos específicos del lenguaje de una aplicación en la que va a integrar la funcionalidad de clasificación de texto de Lenguaje de Azure AI.
1. Haga clic con el botón derecho en la carpeta **classify-text** que contiene los archivos de código y abra un terminal integrado. A continuación, instale el paquete del SDK de Lenguaje de Azure AI Text Analytics mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#:**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**:

    ```
    pip install azure-ai-textanalytics==5.3.0
    ```

1. En el panel **Explorador**, en la carpeta **classify-text**, abra el archivo de configuración para su idioma preferido.

    - **C#** : appsettings.json
    - **Python**: .env
    
1. Actualice los valores de configuración para incluir el **punto de conexión** y una **clave** del recurso de Lenguaje de Azure que creó (disponible en la página **Claves y punto de conexión** del recurso de Lenguaje de Azure AI en Azure Portal). El archivo ya debe contener los nombres de proyecto e implementación del modelo de clasificación de texto.
1. Guarde el archivo de configuración.

## Agregue código para clasificar documentos

Ahora está listo para usar el servicio Lenguaje ded Azure AI para clasificar documentos.

1. Expanda la carpeta **Artículos** de la carpeta **classify-text** para ver los artículos de texto que clasificará la aplicación.
1. En la carpeta **classify-text**, abra el archivo de código para la aplicación cliente:

    - **C#**: Program.cs
    - **Python**: classify-text.py

1. Busque el comentario **Importar espacios de nombres**. A continuación, en este comentario, agregue el siguiente código específico del lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Text Analytics:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**: classify-text.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. En la función **Main (Principal)**, observe que ya se ha proporcionado código para cargar la clave y el punto de conexión del servicio Lenguaje de Azure AI, y el nombre de proyecto y de implementación del archivo de configuración. A continuación, busque el comentario **Create client using endpoint and key**(Crear cliente mediante el punto de conexión y la clave) y agregue el código siguiente para crear un cliente para Text Analysis API:

    **C#**: Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**: classify-text.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. En la función **Main (Principal)**, tenga en cuenta que el código existente lee todos los archivos de la carpeta **Artículos** y crea una lista con su contenido. A continuación, busque el comentario **Obtener clasificaciones** y agregue el siguiente código:

    **C#**: Program.cs

    ```csharp
    // Get Classifications
    ClassifyDocumentOperation operation = await aiClient.SingleLabelClassifyAsync(WaitUntil.Completed, batchedDocuments, projectName, deploymentName);

    int fileNo = 0;
    await foreach (ClassifyDocumentResultCollection documentsInPage in operation.Value)
    {
        
        foreach (ClassifyDocumentResult documentResult in documentsInPage)
        {
            Console.WriteLine(files[fileNo].Name);
            if (documentResult.HasError)
            {
                Console.WriteLine($"  Error!");
                Console.WriteLine($"  Document error code: {documentResult.Error.ErrorCode}");
                Console.WriteLine($"  Message: {documentResult.Error.Message}");
                continue;
            }

            Console.WriteLine($"  Predicted the following class:");
            Console.WriteLine();

            foreach (ClassificationCategory classification in documentResult.ClassificationCategories)
            {
                Console.WriteLine($"  Category: {classification.Category}");
                Console.WriteLine($"  Confidence score: {classification.ConfidenceScore}");
                Console.WriteLine();
            }
            fileNo++;
        }
    }
    ```
    
    **Python**: classify-text.py

    ```Python
    # Get Classifications
    operation = ai_client.begin_single_label_classify(
        batchedDocuments,
        project_name=project_name,
        deployment_name=deployment_name
    )

    document_results = operation.result()

    for doc, classification_result in zip(files, document_results):
        if classification_result.kind == "CustomDocumentClassification":
            classification = classification_result.classifications[0]
            print("{} was classified as '{}' with confidence score {}.".format(
                doc, classification.category, classification.confidence_score)
            )
        elif classification_result.is_error is True:
            print("{} has an error with code '{}' and message '{}'".format(
                doc, classification_result.error.code, classification_result.error.message)
            )
    ```

1. Guarde los cambios en el archivo de código.

## Prueba de la aplicación

La aplicación ya se puede probar.

1. En el terminal integrado de la carpeta **classify-text**, escriba el siguiente comando para ejecutar el programa:

    - **C#**: `dotnet run`
    - **Python**: `python classify-text.py`

    > **Sugerencia**: Puede usar el icono **Maximizar el tamaño del panel **(**^**) en la barra de herramientas del terminal para ver más del texto de la consola.

1. Observe la salida. La aplicación debe enumerar una clasificación y una puntuación de confianza para cada archivo de texto.


## Limpieza

Cuando ya no necesite el proyecto, puede eliminarlo desde la página de **proyectos** en Language Studio. También puede quitar el servicio de Lenguaje de Azure AI y la cuenta de almacenamiento asociada en [Azure Portal](https://portal.azure.com).
