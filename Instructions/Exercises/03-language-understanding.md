---
lab:
  title: Crear un modelo de reconocimiento del lenguaje con el servicio de Lenguaje de Azure AI
  module: Module 5 - Create language understanding solutions
---

# Creación de un modelo de reconocimiento del lenguaje con el servicio de lenguaje

> **NOTA** La característica de reconocimiento del lenguaje conversacional del servicio Lenguaje de Azure AI está actualmente en versión preliminar y sujeta a cambios. En algunos casos, se puede producir un error en el entrenamiento del modelo; si esto sucede, inténtelo de nuevo.  

El servicio Lenguaje de Azure AI permite definir un modelo de *reconocimiento del lenguaje conversacional* que las aplicaciones pueden usar para interpretar la entrada de lenguaje natural de los usuarios, predecir su *intención* (lo que quieren lograr) e identificar las *entidades* a las que se debe aplicar la intención.

Por ejemplo, es posible que se espere que un modelo de lenguaje conversacional para una aplicación de reloj procese la entrada de la siguiente manera:

*¿Qué hora es en Londres?*

Este tipo de entrada es un ejemplo de una *expresión* (algo que un usuario podría decir o escribir), para la que la *intención* deseada es obtener la hora en una ubicación específica (una *entidad*); en este caso, Londres.

> **NOTA** La tarea del modelo de lenguaje conversacional es predecir la intención del usuario e identificar las entidades a las que se aplica la intención. El trabajo de un modelo de lenguaje conversacional <u>no</u> es realizar las acciones necesarias para satisfacer esa intención. Por ejemplo, una aplicación de reloj puede usar un modelo de lenguaje conversacional para distinguir que el usuario quiere saber la hora en Londres. No obstante, la propia aplicación cliente debe implementar la lógica para determinar la hora correcta y mostrarla al usuario.

## Aprovisionar un recurso de *Lenguaje de Azure AI*

Si aún no tiene uno en su suscripción, deberá aprovisionar un recurso del servicio de **Lenguaje de Azure AI** en su suscripción de Azure.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
1. En el campo de búsqueda de la parte superior, busque **Servicios de Azure AI**. A continuación, en los resultados, seleccione **Crear** bajo **Servicio de lenguaje**.
1. Seleccione **Continuar para crear el recurso**.
1. Aprovisione el recurso mediante la siguiente configuración:
    - **Suscripción**: *su suscripción a Azure*.
    - **Grupo de recursos**: *seleccione o cree un grupo de recursos*.
    - **Región**: *elija cualquier región disponible*
    - **Nombre**: *escriba un nombre único*.
    - **Plan de tarifa**: seleccione **F0** (*gratis*) o **S** (*estándar*) si F no está disponible.
    - **Aviso de IA responsable**: Aceptar.
1. Seleccione **Revisar y crear** y **Crear** para aprovisionar el recurso.
1. Espere a que se complete la implementación y, a continuación, vaya al recurso implementado.
1. Consulte la página **Claves y punto de conexión**. Necesitará la información de esta página más adelante en el ejercicio.

## Creación de un proyecto de reconocimiento del lenguaje conversacional

Ahora que ha creado un recurso de creación, puede usarlo para crear un proyecto de reconocimiento del lenguaje conversacional.

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
    4. En la parte superior de la página, haga clic en **Language Studio** para volver a la página principal de Language Studio

1. En la parte superior del portal, en el menú **Crear nuevo**, seleccione **Comprensión del lenguaje de conversación**.

1. En el cuadro de diálogo **Crear un proyecto**, en la página **Escribir información básica**, escriba los detalles siguientes y seleccione **Siguiente**:
    - **Nombre**: `Clock`
    - **Utterances primary language** (Idioma principal de las expresiones): inglés
    - **¿Habilitar varios idiomas en el proyecto?** : *no seleccionado*
    - **Descripción**: `Natural language clock`

1. En la página **Revisar y finalizar**, seleccione **Crear**.

### Crear intenciones

Lo primero que haremos en el nuevo proyecto es definir algunas intenciones. En última instancia, el modelo predecirá cuál de estas intenciones solicita un usuario al enviar una expresión de lenguaje natural.

> **Sugerencia**: Al trabajar en el proyecto, si se muestran algunas sugerencias, léalas y seleccione **Entendido** para descartarlas, o bien seleccione **Omitir todo**.

1. En la página **Definición de esquema**, en la pestaña **Intenciones**, seleccione **&#65291; Agregar** para agregar una nueva intención denominada `GetTime`.
1. Compruebe que aparezca la intención **GetTime** (junto con la intención predeterminada **Ninguna**). A continuación, agregue las siguientes intenciones adicionales:
    - `GetDay`
    - `GetDate`

### Etiquete cada intención con expresiones de ejemplo

Para ayudar al modelo a predecir qué intención solicita un usuario, debe etiquetar cada intención con algunas expresiones de ejemplo.

1. En el panel de la izquierda, seleccione la página **Etiquetado de datos**.

> **Sugerencia**: Puede expandir el panel con el icono **>>** para ver los nombres de página y ocultarlo de nuevo con el icono **<<**.

1. Seleccione la nueva intención **GetTime** y escriba la expresión `what is the time?`. Esto agrega la expresión como entrada de ejemplo para la intención.
1. Agregue las siguientes expresiones adicionales para la intención **GetTime**:
    - `what's the time?`
    - `what time is it?`
    - `tell me the time`

    > **NOTA** Para agregar una nueva expresión, escribe la expresión en el cuadro de texto junto a la intención y presiona ENTRAR. 

1. Seleccione la intención **GetDay** y agregue las siguientes expresiones como entrada de ejemplo:
    - `what day is it?`
    - `what's the day?`
    - `what is the day today?`
    - `what day of the week is it?`

1. Seleccione la intención **GetDate** y agregue las siguientes expresiones:
    - `what date is it?`
    - `what's the date?`
    - `what is the date today?`
    - `what's today's date?`

1. Después de agregar expresiones para cada una de las intenciones, seleccione **Guardar cambios**.

### Entrenamiento y prueba del modelo

Ahora que ha agregado algunas intenciones, vamos a entrenar el modelo de lenguaje y ver si puede predecirlas correctamente a partir de la entrada del usuario.

1. En el panel de la izquierda, seleccione **Trabajos de entrenamiento**. Luego, seleccione **+ Iniciar un trabajo de entrenamiento**.

1. En el cuadro de diálogo **Iniciar un trabajo de entrenamiento**, seleccione la opción para entrenar un nuevo modelo y asígnele el nombre `Clock`. Seleccione el modo **Entrenamiento estándar** y las opciones de **división de datos** predeterminadas.

1. Para comenzar el proceso de entrenamiento del modelo, seleccione **Entrenar**.

1. Una vez completado el entrenamiento (lo que puede tardar varios minutos), el **estado** del trabajo cambiará a **El entrenamiento se realizó correctamente**.

1. Seleccione la página **Rendimiento del modelo** y, luego, elija el modelo **Reloj**. Revise las métricas de evaluación generales y por intención (*precisión*, *coincidencia* y *puntuación F1*) y la *matriz de confusión* generadas por la evaluación que se realizó durante el entrenamiento. Tenga en cuenta que, debido al número reducido de expresiones de ejemplo, puede que no todas las intenciones se incluyan en los resultados.

    > **NOTA**: Para obtener más información sobre las métricas de evaluación, consulte la [documentación](https://learn.microsoft.com/azure/ai-services/language-service/conversational-language-understanding/concepts/evaluation-metrics).

1. Vaya a la página **Implementar un modelo**, seleccione **Agregar implementación**.

1. En el cuadro de diálogo **Agregar implementación**, seleccione **Crear un nuevo nombre de implementación** y, a continuación, escriba `production`.

1. Seleccione el modelo **Reloj** en el campo **Modelo** y, a continuación, seleccione **Implementar**. La implementación puede llevar un tiempo.

1. Cuando el modelo se haya implementado, seleccione la página **Probar implementaciones**, luego seleccione la implementación **Producción** en el campo **Nombre de la implementación**.

1. Escriba el siguiente texto en el cuadro de texto vacío y, después, seleccione **Ejecutar la prueba**:

    `what's the time now?`

    Revise el resultado que se devuelve y observe que incluye la intención pronosticada (que debe ser **GetTime**) y una puntuación de confianza que indica la probabilidad que el modelo calculó para la dicha intención. En la pestaña JSON se muestra la confianza comparativa para cada posible intención (la que tiene la puntuación de confianza más alta es la intención pronosticada).

1. Borre el cuadro de texto y, a continuación, ejecute otra prueba con el texto siguiente:

    `tell me the time`

    De nuevo, revise la intención pronosticada y la puntuación de confianza.

1. Pruebe con este texto:

    `what's the day today?`

    Esperamos que el modelo prediga la intención **GetDay**.

## Agregar entidades

Hasta ahora ha definido algunas expresiones simples que se asignan a intenciones. La mayoría de las aplicaciones reales incluyen expresiones más complejas de las que se deben extraer entidades de datos específicas para obtener más contexto para la intención.

### Adición de una entidad aprendida

El tipo más común de entidad es la una entidad *aprendida*, en la que el modelo aprende a identificar valores de entidad en función de ejemplos.

1. En Language Studio, vuelva a la página **Definición de esquema** y, en la pestaña **Entidades**, seleccione **&#65291; Agregar** para agregar una nueva entidad.

1. En el cuadro de diálogo **Agregar una entidad**, escriba el nombre de la entidad `Location` y asegúrese de la pestaña **Aprendida** está seleccionado. A continuación, seleccione **Agregar entidad**.

1. Una vez creada la entidad **Ubicación**, vuelva a la página **Etiquetado de datos**.
1. Seleccione la intención **GetTime** y escriba la siguiente nueva expresión de ejemplo:

    `what time is it in London?`

1. Cuando se haya agregado la expresión, seleccione la palabra **London** y, en la lista desplegable que aparece, seleccione **Location** para indicar que "London" es un ejemplo de ubicación.

1. Agregue otra expresión de ejemplo adicional para la intención **GetTime**:

    `Tell me the time in Paris?`

1. Cuando se haya agregado la expresión, seleccione la palabra **Paris** y asígnela a la entidad **Location**.

1. Agregue otra expresión de ejemplo adicional para la intención **GetTime**:

    `what's the time in New York?`

1. Cuando se haya agregado la expresión, seleccione las palabras **New York** y asígnelas a la entidad **Location**.

1. Seleccione **Guardar cambios** para guardar las nuevas expresiones.

### Agregar una entidad *list*

En algunos casos, los valores válidos para una entidad se pueden restringir a una lista de términos y sinónimos específicos, que pueden ayudar a la aplicación a identificar instancias de la entidad en expresiones.

1. En Language Studio, vuelva a la página **Definición de esquema** y, en la pestaña **Entidades**, seleccione **&#65291; Agregar** para agregar una nueva entidad.

1. En el cuadro de diálogo **Agregar una entidad**, escriba el nombre de entidad `Weekday` y seleccione la pestaña de entidad **Lista**. Luego seleccione **Agregar entidad**.

1. En la página de la entidad **Día de la semana**, en la sección **Aprendido**, asegúrese de que **No es necesario** está seleccionado. A continuación, en la sección **Lista**, seleccione **&#65291; Agregar lista nueva**. A continuación, escriba el siguiente valor y sinónimo y seleccione **Guardar**:

    | Clave de la lista | Sinónimos|
    |-------------------|---------|
    | `Sunday` | `Sun` |

    > **NOTA** Para especificar los campos de la nueva lista, inserta el valor `Sunday` en el campo de texto y, después, haz clic en el campo donde se muestra "Escriba el valor y pulse Entrar...", escribe los sinónimos y presiona ENTRAR.

1. Repita el paso anterior para agregar los siguientes componentes de lista:

    | Valor | Sinónimos|
    |-------------------|---------|
    | `Monday` | `Mon` |
    | `Tuesday` | `Tue, Tues` |
    | `Wednesday` | `Wed, Weds` |
    | `Thursday` | `Thur, Thurs` |
    | `Friday` | `Fri` |
    | `Saturday` | `Sat` |

1. Después de agregar y guardar los valores de lista, vuelva a la página **Etiquetado de datos**.
1. Seleccione la intención **GetDate** y escriba la siguiente nueva expresión de ejemplo:

    `what date was it on Saturday?`

1. Tras agregar la expresión, seleccione la palabra ***Saturday*** y, en la lista desplegable que aparece, seleccione **Weekday**.

1. Agregue otra expresión de ejemplo adicional para la intención **GetDate**:

    `what date will it be on Friday?`

1. Cuando se haya agregado la expresión, asigne **Friday** a la entidad **Weekday**.

1. Agregue otra expresión de ejemplo adicional para la intención **GetDate**:

    `what will the date be on Thurs?`

1. Cuando se haya agregado la expresión, asigne **Thurs** a la entidad **Weekday**.

1. Seleccione **Guardar cambios** para guardar las nuevas expresiones.

### Adición de una entidad *precompilada*

El servicio Leguaje de Azure AI proporciona un conjunto de entidades *precompiladas* que se usan normalmente en aplicaciones conversacionales.

1. En Language Studio, vuelva a la página **Definición de esquema** y, en la pestaña **Entidades**, seleccione **&#65291; Agregar** para agregar una nueva entidad.

1. En el cuadro de diálogo **Agregar una entidad**, escriba el nombre de entidad `Date` y seleccione la pestaña de entidad **Lista**. Luego seleccione **Agregar entidad**.

1. En la página de la entidad **Fecha**, en la sección **Aprendido**, asegúrese de que **No es necesario** está seleccionado. A continuación, en la sección **Precompilado**, seleccione **&#65291; Agregar precompilación nueva**.

1. En la lista **Seleccionar precompilación**, seleccione **DateTime** y, a continuación, seleccione **Guardar**.
1. Después de agregar la entidad precompilada, vuelva a la página **Etiquetado de datos**
1. Seleccione la intención **GetDay** y escriba la siguiente nueva expresión de ejemplo:

    `what day was 01/01/1901?`

1. Tras agregar la expresión, seleccione ***01/01/1901*** y, en la lista desplegable que aparece, seleccione **Date**.

1. Agregue otra expresión de ejemplo para la intención **GetDay**:

    `what day will it be on Dec 31st 2099?`

1. Cuando se haya agregado la expresión, asigne **Dec 31st 2099** a la entidad **Date**.

1. Seleccione **Guardar cambios** para guardar las nuevas expresiones.

### Nuevo entrenamiento del modelo

Ahora que ha modificado el esquema, debe volver a entrenar y probar el modo.

1. En la página **Entrenamiento de trabajos**, seleccione **Iniciar un trabajo de entrenamiento**.

1. En el cuadro de diálogo **Iniciar un trabajo de entrenamiento**, seleccione **sobrescribir un modelo existente** y especifique el modelo **Reloj**. Para entrenar el modelo, seleccione **Entrenar**. Si se le solicita, confirme que desea sobrescribir el modelo existente.

1. Cuando se complete el entrenamiento, el **estado** del trabajo se actualizará a **El entrenamiento se realizó correctamente**.

1. Seleccione la página **Rendimiento del modelo** y, luego, elija el modelo **Reloj**. Revise las métricas de evaluación generales (*precisión*, *coincidencia* y *puntuación F1*) y la *matriz de confusión* generadas por la evaluación que se realizó durante el entrenamiento (tenga en cuenta que, debido al número reducido de expresiones de ejemplo, puede que no todas las intenciones se incluyan en los resultados).

1. En la página **Implementación de un modelo**, seleccione **Agregar implementación**.

1. En el cuadro de diálogo **Agregar implementación**, seleccione **Invalidar un nombre de implementación existente** y, a continuación, seleccione **producción**.

1. Seleccione el modelo **Reloj** en el campo **Modelo** y, a continuación, seleccione **Implementar** para implementarlo. Esto puede llevar algo de tiempo.

1. Después de implementar el modelo, en la página **Probar implementaciones**, seleccione la implementación **Producción**, en **Nombre de implementación** y, luego, pruébela con el siguiente texto:

    `what's the time in Edinburgh?`

1. Revise el resultado que se devuelve, que debería predecir la intención **GetTime** y una entidad **Location** con el valor de texto "Edinburgh".

1. Pruebe las siguientes expresiones:

    `what time is it in Tokyo?`

    `what date is it on Friday?`

    `what's the date on Weds?`

    `what day was 01/01/2020?`

    `what day will Mar 7th 2030 be?`

## Uso del modelo desde una aplicación cliente

En un proyecto real, refinaría de forma iterativa las intenciones y las entidades, y volvería a entrenar y a probar la aplicación hasta que estuviera satisfecho con el rendimiento predictivo. Después, cuando lo haya probado y dé por bueno su rendimiento predictivo, puede usarlo en una aplicación cliente mediante una llamada a su interfaz de REST o al SDK específico de runtime.

### Preparación para desarrollar una aplicación en Visual Studio Code

Desarrollará la aplicación Language Understanding mediante Visual Studio Code. Los archivos de código de la aplicación se han proporcionado en un repositorio de GitHub.

> **Sugerencia**: Si ya ha clonado el repositorio **mslearn-ai-language**, ábralo en Visual Studio Code. De lo contrario, siga estos pasos para clonarlo en el entorno de desarrollo.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/mslearn-ai-language` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.

    > **Nota**: Si Visual Studio Code muestra un mensaje emergente para solicitarle que confíe en el código que está abriendo, haga clic en la opción **Sí, confío en los autores** en el elemento emergente.

4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

### Configuración de la aplicación

Se han proporcionado aplicaciones para C# y Python, así como un archivo de texto de ejemplo que usará para probar el resumen. Las dos aplicaciones tienen la misma funcionalidad. Primero, completará algunas partes clave de la aplicación para habilitar que utilice el recurso de Lenguaje de Azure AI.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **Labfiles/03-language** y expanda la carpeta **CSharp** o **Python** según sus preferencias de lenguaje y la carpeta **clock-client** que contiene. Cada carpeta contiene los archivos específicos del lenguaje de una aplicación en la que va a integrar la funcionalidad de respuesta a preguntas de Lenguaje de Azure AI.
2. Haga clic con el botón derecho en la carpeta **clock- client** que contiene los archivos de código y abra un terminal integrado. A continuación, instale el paquete del SDK de reconocimiento del lenguaje conversacional de Lenguaje de Azure AI mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#:**

    ```
    dotnet add package Azure.AI.Language.Conversations --version 1.1.0
    ```

    **Python**:

    ```
    pip install azure-ai-language-conversations
    ```

3. En el panel **Explorador**, en la carpeta **clock-client**, abra el archivo de configuración para su lenguaje preferido.

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Actualice los valores de configuración para incluir el **punto de conexión** y una **clave** del recurso de Lenguaje de Azure que creó (disponible en la página **Claves y punto de conexión** del recurso de Lenguaje de Azure AI en Azure Portal)
5. Guarde el archivo de configuración.

### Agregar código a la aplicación

Ahora está listo para agregar el código necesario para importar las bibliotecas de SDK necesarias, establecer una conexión autenticada al proyecto implementado y enviar preguntas.

1. Tenga en cuenta que la carpeta **clock-client** contiene un archivo de código para la aplicación cliente:

    - **C#**: Program.cs
    - **Python**: clock-client.py

    Abra el archivo de código y, en la parte superior, en las referencias de espacio de nombres existentes, busque el comentario **Importar espacios de nombres**. A continuación, en este comentario, agregue el siguiente código específico del lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Text Analytics:

    **C#**: Programs.cs

    ```c#
    // import namespaces
    using Azure;
    using Azure.AI.Language.Conversations;
    ```

    **Python**: clock-client.py

    ```python
    # Import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.conversations import ConversationAnalysisClient
    ```

1. En la función **Main**, fíjese en que ya se ha proporcionado código para cargar el punto de conexión de predicción y la clave del archivo de configuración. A continuación, busque el comentario **Crear un cliente para el modelo de Language Service** y agregue el código siguiente para crear un cliente de predicción para la aplicación de Language Service:

    **C#**: Programs.cs

    ```c#
    // Create a client for the Language service model
    Uri endpoint = new Uri(predictionEndpoint);
    AzureKeyCredential credential = new AzureKeyCredential(predictionKey);

    ConversationAnalysisClient client = new ConversationAnalysisClient(endpoint, credential);
    ```

    **Python**: clock-client.py

    ```python
    # Create a client for the Language service model
    client = ConversationAnalysisClient(
        ls_prediction_endpoint, AzureKeyCredential(ls_prediction_key))
    ```

1. Tenga en cuenta que el código de la función **Main** solicita entrada del usuario hasta que ese escribe "quit". Dentro de este bucle, busque el comentario **Llamada al modelo de Language Service para obtener la intención y las entidades** y agregue el código siguiente:

    **C#**: Programs.cs

    ```c#
    // Call the Language service model to get intent and entities
    var projectName = "Clock";
    var deploymentName = "production";
    var data = new
    {
        analysisInput = new
        {
            conversationItem = new
            {
                text = userText,
                id = "1",
                participantId = "1",
            }
        },
        parameters = new
        {
            projectName,
            deploymentName,
            // Use Utf16CodeUnit for strings in .NET.
            stringIndexType = "Utf16CodeUnit",
        },
        kind = "Conversation",
    };
    // Send request
    Response response = await client.AnalyzeConversationAsync(RequestContent.Create(data));
    dynamic conversationalTaskResult = response.Content.ToDynamicFromJson(JsonPropertyNames.CamelCase);
    dynamic conversationPrediction = conversationalTaskResult.Result.Prediction;   
    var options = new JsonSerializerOptions { WriteIndented = true };
    Console.WriteLine(JsonSerializer.Serialize(conversationalTaskResult, options));
    Console.WriteLine("--------------------\n");
    Console.WriteLine(userText);
    var topIntent = "";
    if (conversationPrediction.Intents[0].ConfidenceScore > 0.5)
    {
        topIntent = conversationPrediction.TopIntent;
    }
    ```

    **Python**: clock-client.py

    ```python
    # Call the Language service model to get intent and entities
    cls_project = 'Clock'
    deployment_slot = 'production'

    with client:
        query = userText
        result = client.analyze_conversation(
            task={
                "kind": "Conversation",
                "analysisInput": {
                    "conversationItem": {
                        "participantId": "1",
                        "id": "1",
                        "modality": "text",
                        "language": "en",
                        "text": query
                    },
                    "isLoggingEnabled": False
                },
                "parameters": {
                    "projectName": cls_project,
                    "deploymentName": deployment_slot,
                    "verbose": True
                }
            }
        )

    top_intent = result["result"]["prediction"]["topIntent"]
    entities = result["result"]["prediction"]["entities"]

    print("view top intent:")
    print("\ttop intent: {}".format(result["result"]["prediction"]["topIntent"]))
    print("\tcategory: {}".format(result["result"]["prediction"]["intents"][0]["category"]))
    print("\tconfidence score: {}\n".format(result["result"]["prediction"]["intents"][0]["confidenceScore"]))

    print("view entities:")
    for entity in entities:
        print("\tcategory: {}".format(entity["category"]))
        print("\ttext: {}".format(entity["text"]))
        print("\tconfidence score: {}".format(entity["confidenceScore"]))

    print("query: {}".format(result["result"]["query"]))
    ```

    La llamada al modelo de Language Service devuelve una predicción o un resultado, que incluye la intención superior (la más probable), así como cualquier entidad detectada en la expresión de entrada. La aplicación cliente debe usar ahora esa predicción para determinar y realizar la acción adecuada.

1. Busque el comentario **Apply the appropriate action** (Aplicar la acción adecuada) y agregue el código siguiente, que comprueba las intenciones admitidas por la aplicación (**GetTime**,**GetDate** y **GetDay**) y determina si se ha detectado alguna entidad pertinente, antes de llamar a una función existente para generar una respuesta adecuada.

    **C#**: Programs.cs

    ```c#
    // Apply the appropriate action
    switch (topIntent)
    {
        case "GetTime":
            var location = "local";           
            // Check for a location entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Location")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    location = entity.Text;
                }
            }
            // Get the time for the specified location
            string timeResponse = GetTime(location);
            Console.WriteLine(timeResponse);
            break;
        case "GetDay":
            var date = DateTime.Today.ToShortDateString();            
            // Check for a Date entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Date")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    date = entity.Text;
                }
            }            
            // Get the day for the specified date
            string dayResponse = GetDay(date);
            Console.WriteLine(dayResponse);
            break;
        case "GetDate":
            var day = DateTime.Today.DayOfWeek.ToString();
            // Check for entities            
            // Check for a Weekday entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Weekday")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    day = entity.Text;
                }
            }          
            // Get the date for the specified day
            string dateResponse = GetDate(day);
            Console.WriteLine(dateResponse);
            break;
        default:
            // Some other intent (for example, "None") was predicted
            Console.WriteLine("Try asking me for the time, the day, or the date.");
            break;
    }
    ```

    **Python**: clock-client.py

    ```python
    # Apply the appropriate action
    if top_intent == 'GetTime':
        location = 'local'
        # Check for entities
        if len(entities) > 0:
            # Check for a location entity
            for entity in entities:
                if 'Location' == entity["category"]:
                    # ML entities are strings, get the first one
                    location = entity["text"]
        # Get the time for the specified location
        print(GetTime(location))

    elif top_intent == 'GetDay':
        date_string = date.today().strftime("%m/%d/%Y")
        # Check for entities
        if len(entities) > 0:
            # Check for a Date entity
            for entity in entities:
                if 'Date' == entity["category"]:
                    # Regex entities are strings, get the first one
                    date_string = entity["text"]
        # Get the day for the specified date
        print(GetDay(date_string))

    elif top_intent == 'GetDate':
        day = 'today'
        # Check for entities
        if len(entities) > 0:
            # Check for a Weekday entity
            for entity in entities:
                if 'Weekday' == entity["category"]:
                # List entities are lists
                    day = entity["text"]
        # Get the date for the specified day
        print(GetDate(day))

    else:
        # Some other intent (for example, "None") was predicted
        print('Try asking me for the time, the day, or the date.')
    ```

1. Guarde los cambios y vuelva al terminal integrado de la carpeta **clock-client** y escriba el siguiente comando para ejecutar el programa:

    - **C#** : `dotnet run`
    - **Python**: `python clock-client.py`

    > **Sugerencia**: Puede usar el icono **Maximizar el tamaño del panel **(**^**) en la barra de herramientas del terminal para ver más del texto de la consola.

1. Cuando se le solicite, escriba las expresiones para probar la aplicación. Por ejemplo, pruebe lo siguiente:

    *Hello*

    *¿Qué hora tenemos?*

    *¿Qué hora es en Londres?*

    *¿Qué fecha tenemos?*

    *¿Qué fecha es el domingo?*

    *¿Qué día es?*

    *¿Qué día es 01/01/2025?*

    > **Nota**: La lógica de la aplicación es deliberadamente sencilla y tiene una serie de limitaciones. Por ejemplo, al obtener la hora, solo se admite un conjunto restringido de ciudades y se omite el horario de verano. El objetivo es ver un ejemplo de un patrón típico para usar Language Service en el que la aplicación debe:
    >   1. Conectarse a un punto de conexión de la predicción.
    >   2. Enviar una expresión para obtener una predicción.
    >   3. Implementar la lógica para responder correctamente a la intención y las entidades predichos.

1. Cuando haya terminado de probar, escriba *quit*.

## Limpieza de recursos

Sugerencia: Si ha terminado de explorar el servicio Lenguaje de Azure AI, puede eliminar los recursos que creó en este ejercicio. A continuación, se indica cómo puede hacerlo.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
2. Vaya al recurso de Lenguaje de Azure AI que creó en este laboratorio.
3. En la página del recurso, seleccione **Eliminar** y siga las instrucciones para eliminar el recurso.

## Más información

Para más información sobre el reconocimiento del lenguaje conversacional de Lenguaje de Azure AI, consulte la [documentación de Lenguaje de Azure AI](https://learn.microsoft.com/azure/ai-services/language-service/conversational-language-understanding/overview).
