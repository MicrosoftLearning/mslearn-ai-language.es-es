---
lab:
  title: Clasificación de texto personalizada
  description: Aplique clasificaciones personalizadas a la entrada de texto mediante Lenguaje de Azure AI.
---

# Clasificación de texto personalizada

Lenguaje de Azure AI proporciona varias capacidades de PLN, incluida la identificación de frases clave, el resumen de texto y el análisis de sentimiento. El servicio de lenguaje también proporciona características personalizadas, como la respuesta a preguntas personalizada y la clasificación de texto personalizado.

Para probar la clasificación de texto personalizado del servicio Lenguaje de Azure AI, configurará el modelo mediante Language Studio y, a continuación, usará una aplicación de Python para probarlo.

Aunque este ejercicio se basa en Python, puede desarrollar aplicaciones de clasificación de texto mediante varios SDK específicos del lenguaje, como:

- [Biblioteca cliente de Azure AI Text Analytics para Python](https://pypi.org/project/azure-ai-textanalytics/)
- [Biblioteca cliente de Azure AI Text Analytics para .NET](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [Biblioteca cliente de Azure AI Text Analytics para JavaScript](https://www.npmjs.com/package/@azure/ai-text-analytics)

Este ejercicio dura aproximadamente **35** minutos.

## Aprovisionar un recurso de *Lenguaje de Azure AI*

Si aún no tiene uno en su suscripción, deberá aprovisionar un recurso del servicio **Lenguaje de Azure AI**. Además, use la clasificación de texto personalizada, debe habilitar la característica **Clasificación y extracción de texto personalizado**.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
1. Seleccione **Crear un recurso**.
1. En el campo de búsqueda, busca **Servicio de lenguaje**. A continuación, en los resultados, seleccione **Crear** bajo **Servicio de lenguaje**.
1. Seleccione el cuadro que incluye **clasificación de texto personalizado**. Después, seleccione **Continuar para crear el recurso**.
1. Cree un recurso con los valores siguientes:
    - **Suscripción**: *su suscripción a Azure*.
    - **Grupo de recursos**: *seleccione o cree un grupo de recursos*.
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
    - **Nombre**: *escriba un nombre único*.
    - **Plan de tarifa**: seleccione **F0** (*gratis*), o **S** (*estándar*) si F no está disponible.
    - **Cuenta de almacenamiento**: nueva cuenta de almacenamiento.
      - **Nombre de la cuenta de almacenamiento**: *escriba un nombre único*.
      - **Tipo de cuenta de almacenamiento**: LRS estándar.
    - **Aviso de IA responsable**: seleccionado.

1. Seleccione **Revisar y crear** y **Crear** para aprovisionar el recurso.
1. Espere a que se complete la implementación y, a continuación, vaya al grupo de recursos.
1. Busque la cuenta de almacenamiento que creó, selecciónela y compruebe que el _tipo de cuenta_ es **StorageV2**. Si es v1, actualice el tipo de la cuenta de almacenamiento en esa página de recursos.

## Configuración del acceso basado en roles para el usuario

> **NOTA**: Si omite este paso, recibirá un error 403 al intentar conectarse a su proyecto personalizado. Es importante que el usuario actual tenga este rol para acceder a los datos del blob de la cuenta de almacenamiento, incluso si es el propietario de la cuenta de almacenamiento.

1. Vaya a la página de la cuenta de almacenamiento en Azure Portal.
2. Seleccione **Control de acceso (IAM)** en el menú de navegación izquierdo.
3. Seleccione **Agregar** para agregar asignaciones de roles y elija el rol **Propietario de datos de Storage Blob** en la cuenta de almacenamiento.
4. En **Asignar acceso a**, seleccione **Usuario, grupo o entidad de servicio**.
5. Elija **Seleccionar miembros**.
6. Seleccione su usuario. Puede buscar por nombres de usuario en el campo **Seleccionar**.

## Carga de artículos de muestra

Una vez que haya creado el servicio de Lenguaje de Azure AI y la cuenta de almacenamiento, deberá cargar artículos de ejemplo para entrenar el modelo más adelante.

1. En una nueva pestaña del explorador, descargue artículos de ejemplo de `https://aka.ms/classification-articles` y extraiga los archivos en una carpeta de su elección.

1. En Azure Portal, navegue hasta la cuenta de almacenamiento que ha creado y selecciónela.

1. En la cuenta de almacenamiento, seleccione **Configuración**, que se encuentra bajo **Configuración**. En la pantalla Configuración, habilite la opción **Permitir el acceso anónimo de blobs** y, a continuación, seleccione **Guardar**.

1. Seleccione **Contenedores** en el menú de la izquierda, que se encuentra debajo de **Almacenamiento de datos**. En la pantalla que aparece, seleccione **+ Contenedor**. Denomine al contenedor `articles` y establezca el **Nivel de acceso anónimo** en **Contenedor (acceso de lectura anónimo para contenedores y blobs)**.

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
1. En la página **Seleccionar tipo de proyecto**, seleccione **Clasificación de etiqueta única**. Luego, seleccione **Siguiente**.
1. En el panel **Escribir información básica**, establezca lo siguiente:
    - **Nombre**: `ClassifyLab`  
    - **Idioma principal del texto**: inglés (US)
    - **Descripción**: `Custom text lab`

1. Seleccione **Siguiente**.
1. En la página **Elegir contenedor**, establezca la lista desplegable **Contenedor del almacén de blobs** en el contenedor de *artículos*.
1. Seleccione la opción **No, necesito etiquetar mis archivos como parte de este proyecto**. Luego, seleccione **Siguiente**.
1. Seleccione **Create project** (Crear proyecto).

> **Sugerencia**: si recibes un error sobre que no estás autorizado para realizar esta operación, tendrás que agregar una asignación de roles. Para solucionarlo, agregamos el rol "Colaborador de datos de blob de almacenamiento" en la cuenta de almacenamiento para el usuario que ejecuta el laboratorio. Encontrarás más información [en la página de documentación](https://learn.microsoft.com/azure/ai-services/language-service/custom-named-entity-recognition/how-to/create-project?tabs=portal%2Clanguage-studio#enable-identity-management-for-your-resource).

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

## Preparación para desarrollar una aplicación en Cloud Shell

Para probar las funcionalidades de clasificación de texto personalizado del servicio Lenguaje de Azure AI, desarrollará una aplicación de consola sencilla en Azure Cloud Shell.

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

1. Una vez clonado el repo, ve a la carpeta que contiene los archivos de código de aplicación:  

    ```
   cd mslearn-ai-language/Labfiles/04-text-classification/Python/classify-text
    ```

## Configuración de la aplicación

1. En el panel de línea de comandos, ejecute el siguiente comando para ver los archivos de código de la carpeta **classify-text**:

    ```
   ls -a -l
    ```

    Los archivos incluyen un archivo de configuración (**.env**) y un archivo de código (**classify-text.py**). El texto que analizará la aplicación se encuentra en la subcarpeta **articles**.

1. Cree un entorno virtual de Python e instale el paquete del SDK de Text Analytics para Lenguaje de Azure AI y otros paquetes necesarios mediante la ejecución del comando siguiente:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-textanalytics==5.3.0
    ```

1. Escriba el siguiente comando para editar el archivo de configuración de la aplicación:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. Actualice los valores de configuración para incluir el **punto de conexión** y una **clave** del recurso de lenguaje de Azure que creó (disponible en la página **Claves y punto de conexión** del recurso de Lenguaje de Azure AI en Azure Portal). El archivo ya debe contener los nombres de proyecto e implementación del modelo de clasificación de texto.
1. Después de reemplazar los marcadores de posición, en el editor de código, usa el comando **CTRL+S** o usa la acción de **hacer clic con el botón derecho > Guardar** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** o la acción de **hacer clic con el botón derecho > Salir** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

## Agregue código para clasificar documentos

1. Escriba el siguiente comando para editar el archivo de código de la aplicación:

    ```
    code classify-text.py
    ```

1. Revise el código existente. Agregará código para trabajar con el SDK de Text Analytics para Lenguaje de IA.

    > **Sugerencia**: al agregar código al archivo de código, asegúrate de mantener la sangría correcta.

1. En la parte superior del archivo de código, en las referencias de espacio de nombres existentes, busque el comentario **Importar espacios de nombres** y agregue el código siguiente para importar los espacios de nombres que necesitará usar el SDK de Text Analytics:

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. En la función **main**, observe que ya se han proporcionado el código para cargar el punto de conexión y la clave del servicio Lenguaje de Azure AI y los nombres de proyecto e implementación del archivo de configuración. A continuación, busque el comentario **Crear cliente mediante el punto de conexión y la clave** y agregue el código siguiente para crear un cliente para de análisis de texto:

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Observe que el código existente lee todos los archivos de la carpeta **articles** y crea una lista con su contenido. A continuación, busque el comentario **Obtener clasificaciones** y agregue el siguiente código:

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

1. Guarde los cambios (CTRL+S) y, a continuación, escriba el siguiente comando para ejecutar el programa (maximice el panel de Cloud Shell y cambie el tamaño de los paneles para ver más texto en el panel de línea de comandos):

    ```
   python classify-text.py
    ```

1. Observe la salida. La aplicación debe enumerar una clasificación y una puntuación de confianza para cada archivo de texto.

## Limpieza

Cuando ya no necesite el proyecto, puede eliminarlo desde la página **proyectos** de Language Studio. También puede quitar el servicio de Lenguaje de Azure AI y la cuenta de almacenamiento asociada en [Azure Portal](https://portal.azure.com).
