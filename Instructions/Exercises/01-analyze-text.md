---
lab:
  title: Análisis de texto
  module: Module 3 - Develop natural language processing solutions
---

# Análisis de texto

El **Lenguaje de Azure** es compatible con el análisis de texto, que incluye la detección del idioma, el análisis de opinión, la extracción de frases clave y el reconocimiento de entidades.

Por ejemplo, supongamos que una agencia de viajes quiere procesar las reseñas de hoteles que se han enviado al sitio web de la empresa. Mediante el Lenguaje de Azure AI, pueden determinar el idioma en el que se ha escrito cada reseña, su opinión (positiva, neutra o negativa), las frases clave que podrían indicar los temas principales que se tratan en ella y las entidades con nombre, como lugares, puntos de referencia o personas mencionadas en ellas.

## Aprovisionar un recurso de *Lenguaje de Azure AI*

Si aún no tiene uno en su suscripción, deberá aprovisionar un recurso del servicio de **Lenguaje de Azure AI** en su suscripción de Azure.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
1. En el campo de búsqueda de la parte superior, busca **Servicios de Azure AI**. Luego, en los resultados selecciona **Crear** en el recurso **Servicio de lenguaje**.
1. Seleccione **Continuar para crear el recurso**.
1. Aprovisione el recurso mediante la siguiente configuración:
    - **Suscripción**: *su suscripción a Azure*.
    - **Grupo de recursos**: *seleccione o cree un grupo de recursos*.
    - **Región**: *elige cualquier región disponible*.
    - **Nombre**: *escriba un nombre único*.
    - **Plan de tarifa**: seleccione **F0** (*gratis*) o **S** (*estándar*) si F no está disponible.
    - **Aviso de IA responsable**: acepta.
1. Seleccione **Revisar + crear**.
1. Espere a que se complete la implementación y, a continuación, vaya al recurso implementado.
1. Consulte la página **Claves y punto de conexión**. Necesitará la información de esta página más adelante en el ejercicio.

## Preparación para desarrollar una aplicación en Visual Studio Code

Va a desarrollar la aplicación de análisis de texto mediante Visual Studio Code. Los archivos de código de la aplicación se han proporcionado en un repositorio de GitHub.

> **Sugerencia**: Si ya ha clonado el repositorio **mslearn-ai-language**, ábralo en Visual Studio Code. De lo contrario, siga estos pasos para clonarlo en el entorno de desarrollo.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/mslearn-ai-language` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.
4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## Configuración de la aplicación

Se han proporcionado aplicaciones para C# y Python, así como un archivo de texto de ejemplo que usará para probar el resumen. Las dos aplicaciones tienen la misma funcionalidad. Primero, completará algunas partes clave de la aplicación para que pueda usar su recurso de Lenguaje de Azure AI.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **Labfiles/01-analyze-text** y expanda la carpeta **CSharp** o **Python** según sus preferencias de lenguaje y la carpeta **text-analytics** que contiene. Cada carpeta contiene los archivos específicos del lenguaje de una aplicación en la que va a integrar la funcionalidad de análisis de texto de Lenguaje de Azure AI.
2. Haga clic con el botón derecho en la carpeta **text-analysis** que contiene sus archivos de código y abra un terminal integrado. Después, instale el paquete del SDK de Text Analytics de Lenguaje de Azure AI mediante la ejecución del comando adecuado para sus preferencias de lenguaje: Para el ejercicio de Python, instale también el paquete `dotenv`:

    **C#:**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python**:

    ```
    pip install azure-ai-textanalytics==5.3.0
    pip install python-dotenv
    ```

3. En el panel **Explorador**, en la carpeta **text-analytics**, abra el archivo de configuración para su lenguaje preferido.

    - **C#** : appsettings.json
    - **Python**: .env
    
4. Actualice los valores de configuración para incluir el **punto de conexión** y una **clave** del recurso de Lenguaje de Azure que ha creado (disponible en la página **Claves y punto de conexión** de su recurso de Lenguaje de Azure AI en Azure Portal)
5. Guarde el archivo de configuración.

6. Observe que la carpeta **text-analysis** contiene un archivo de código para la aplicación cliente:

    - **C#** : Program.cs
    - **Python**: text-analysis.py

    Abra el archivo de código y, en la parte superior, en las referencias de espacio de nombres existentes, busque el comentario **Importar espacios de nombres**. A continuación, en este comentario, agregue el siguiente código específico del lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Text Analytics:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**: text-analysis.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

7. En la función **Principal**, observa que ya se ha proporcionado código para cargar la clave y el punto de conexión del servicio de Lenguaje de Azure AI desde el archivo de configuración. A continuación, busque el comentario **Create client using endpoint and key**(Crear cliente mediante el punto de conexión y la clave) y agregue el código siguiente para crear un cliente para Text Analysis API:

    **C#**: Programs.cs

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**: text-analysis.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

8. Guarde los cambios y vuelva al terminal integrado de la carpeta **text-analysis** y escriba el siguiente comando para ejecutar el programa:

    - **C#** : `dotnet run`
    - **Python**: `python text-analysis.py`

    > **Sugerencia**: Puede usar el icono **Maximizar el tamaño del panel** (**^**) en la barra de herramientas del terminal para ver más del texto de la consola.

9. Observe la salida, ya que el código debe ejecutarse sin errores y mostrar el contenido de cada archivo de texto de revisión en la carpeta **reviews**. La aplicación crea correctamente un cliente para Text Analytics API, pero no lo usa. Lo corregiremos en el siguiente procedimiento.

## Agregar código para detectar el idioma

Ahora que has creado un cliente para la API, vamos a usarlo para detectar el idioma en el que se escribe cada reseña.

1. En la función **Main** del programa, busque el comentario **Get language** (Obtener idioma). A continuación, en este comentario, agregue el código necesario para detectar el idioma de cada documento de revisión:

    **C#**: Programs.cs

    ```csharp
    // Get language
    DetectedLanguage detectedLanguage = aiClient.DetectLanguage(text);
    Console.WriteLine($"\nLanguage: {detectedLanguage.Name}");
    ```

    **Python**: text-analysis.py

    ```python
    # Get language
    detectedLanguage = ai_client.detect_language(documents=[text])[0]
    print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

     > **Nota**: *En este ejemplo, cada revisión se analiza individualmente, lo que da lugar a una llamada independiente al servicio para cada archivo. Un enfoque alternativo es crear una colección de documentos y pasarlos al servicio en una sola llamada. En ambos enfoques, la respuesta del servicio consta de una colección de documentos; por eso, en el código de Python anterior, se especifica el índice del primer (y único) documento en la respuesta ([0]).*

1. Guarde los cambios. Vuelva al terminal integrado de la carpeta **text-analysis** y vuelva a ejecutar el programa:
1. Observe la salida y tenga en cuenta que esta vez se identifica el idioma de cada reseña.

## Agregar código para evaluar la opinión

El *análisis de sentimiento* es una técnica que se usa habitualmente para clasificar el texto como *positivo* o *negativo* (o posiblemente *neutro* o *mixto*). Se usa normalmente para analizar publicaciones en redes sociales, reseñas de productos y otros elementos en los que la opinión del texto puede proporcionar información útil.

1. En la función **Main** del programa, busque el comentario **Get sentiment** (Obtener opinión). A continuación, en este comentario, agregue el código necesario para detectar la opinión de cada documento de revisión:

    **C#** : Program.cs

    ```csharp
    // Get sentiment
    DocumentSentiment sentimentAnalysis = aiClient.AnalyzeSentiment(text);
    Console.WriteLine($"\nSentiment: {sentimentAnalysis.Sentiment}");
    ```

    **Python**: text-analysis.py

    ```python
    # Get sentiment
    sentimentAnalysis = ai_client.analyze_sentiment(documents=[text])[0]
    print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

1. Guarde los cambios. Vuelva al terminal integrado de la carpeta **text-analysis** y vuelva a ejecutar el programa:
1. Observe la salida y que se detecta la opinión de las revisiones.

## Agregar código para identificar frases clave

Puede ser útil identificar frases clave en un cuerpo de texto para ayudar a determinar los temas principales que se tratan.

1. En la función **Main** del programa, busque el comentario **Get key phrases** (Obtener frases clave). A continuación, en este comentario, agregue el código necesario para detectar las frases clave en cada documento de revisión:

    **C#** : Program.cs

    ```csharp
    // Get key phrases
    KeyPhraseCollection phrases = aiClient.ExtractKeyPhrases(text);
    if (phrases.Count > 0)
    {
        Console.WriteLine("\nKey Phrases:");
        foreach(string phrase in phrases)
        {
            Console.WriteLine($"\t{phrase}");
        }
    }
    ```

    **Python**: text-analysis.py

    ```python
    # Get key phrases
    phrases = ai_client.extract_key_phrases(documents=[text])[0].key_phrases
    if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

1. Guarde los cambios. Vuelva al terminal integrado de la carpeta **text-analysis** y vuelva a ejecutar el programa:
1. Observe la salida y que cada documento contiene frases clave que proporcionan información sobre el tema de la revisión.

## Agregar entidades para extraer datos

A menudo, los documentos u otros cuerpos de texto mencionan personas, lugares, períodos de tiempo u otras entidades. Text Analytics API puede detectar varias categorías (y subcategorías) de la entidad en el texto.

1. En la función **Main** del programa, busque el comentario **Get entities** (Obtener entidades). A continuación, en este comentario, agregue el código necesario para identificar las entidades que se mencionan en cada revisión:

    **C#** : Program.cs

    ```csharp
    // Get entities
    CategorizedEntityCollection entities = aiClient.RecognizeEntities(text);
    if (entities.Count > 0)
    {
        Console.WriteLine("\nEntities:");
        foreach(CategorizedEntity entity in entities)
        {
            Console.WriteLine($"\t{entity.Text} ({entity.Category})");
        }
    }
    ```

    **Python**: text-analysis.py

    ```python
    # Get entities
    entities = ai_client.recognize_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

1. Guarde los cambios. Vuelva al terminal integrado de la carpeta **text-analysis** y vuelva a ejecutar el programa:
1. Observe la salida y las entidades que se han detectado en el texto.

## Agregar código para extraer entidades vinculadas

Además de las entidades clasificadas, Text Analytics API puede detectar entidades para las que hay vínculos conocidos a orígenes de datos, como Wikipedia.

1. En la función **Main** del programa, busque el comentario **Get linked entities** (Obtener entidades vinculadas). A continuación, en este comentario, agregue el código necesario para identificar las entidades vinculadas que se mencionan en cada revisión:

    **C#** : Program.cs

    ```csharp
    // Get linked entities
    LinkedEntityCollection linkedEntities = aiClient.RecognizeLinkedEntities(text);
    if (linkedEntities.Count > 0)
    {
        Console.WriteLine("\nLinks:");
        foreach(LinkedEntity linkedEntity in linkedEntities)
        {
            Console.WriteLine($"\t{linkedEntity.Name} ({linkedEntity.Url})");
        }
    }
    ```

    **Python**: text-analysis.py

    ```python
    # Get linked entities
    entities = ai_client.recognize_linked_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

1. Guarde los cambios. Vuelva al terminal integrado de la carpeta **text-analysis** y vuelva a ejecutar el programa:
1. Observe la salida y las entidades vinculadas que se identifican.

## Limpieza de recursos

Sugerencia: Si ha terminado de explorar el servicio Lenguaje de Azure AI, puede eliminar los recursos que creó en este ejercicio. A continuación, se indica cómo puede hacerlo.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.

2. Vaya al recurso de Lenguaje de Azure AI que creó en este laboratorio.

3. En la página del recurso, seleccione **Eliminar** y siga las instrucciones para eliminar el recurso.

## Más información

Para más información sobre el uso de **Lenguaje de Azure AI**, consulta la [Documentación](https://learn.microsoft.com/azure/ai-services/language-service/).
