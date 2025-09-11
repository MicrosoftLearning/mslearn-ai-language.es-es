---
lab:
  title: Análisis de texto
  description: 'Use Lenguaje de Azure AI para analizar texto, lo que incluye la detección del idioma, el análisis de sentimiento, la extracción de frases clave y el reconocimiento de entidades.'
---

# Análisis de texto

**Lenguaje de Azure AI** admite el análisis de texto, lo que incluye la detección del idioma, el análisis de sentimiento, la extracción de frases clave y el reconocimiento de entidades.

Por ejemplo, supongamos que una agencia de viajes quiere procesar las reseñas de hoteles que se han enviado al sitio web de la empresa. Mediante el Lenguaje de Azure AI, la agencia puede determinar el idioma en el que se ha escrito cada reseña, la opinión (positiva, neutra o negativa) de las reseñas, las frases clave que podrían indicar los temas principales que se tratan en la reseña y las entidades con nombre, como lugares, puntos de referencia o personas mencionadas en las reseñas. En este ejercicio, usará el SDK de Python de Lenguaje de Azure AI para análisis de texto para implementar una aplicación sencilla de reseña de hoteles basada en este ejemplo.

Aunque este ejercicio se basa en Python, puede desarrollar aplicaciones similares usando varios SDK específicos del lenguaje, como:

- [Biblioteca cliente de Azure AI Text Analytics para Python](https://pypi.org/project/azure-ai-textanalytics/)
- [Biblioteca cliente de Azure AI Text Analytics para .NET](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [Biblioteca cliente de Azure AI Text Analytics para JavaScript](https://www.npmjs.com/package/@azure/ai-text-analytics)

Este ejercicio dura aproximadamente **30** minutos.

## Aprovisionar un recurso de *Lenguaje de Azure AI*

Si aún no tiene uno en su suscripción, deberá aprovisionar un recurso del servicio de **Lenguaje de Azure AI** en su suscripción de Azure.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
1. Seleccione **Crear un recurso**.
1. En el campo de búsqueda, busca **Servicio de lenguaje**. A continuación, en los resultados, seleccione **Crear** bajo **Servicio de lenguaje**.
1. Seleccione **Continuar para crear el recurso**.
1. Aprovisione el recurso mediante la siguiente configuración:
    - **Suscripción**: *su suscripción a Azure*.
    - **Grupo de recursos**: *seleccione o cree un grupo de recursos*.
    - **Región**: *elija cualquier región disponible*
    - **Nombre**: *escriba un nombre único*.
    - **Plan de tarifa**: seleccione **F0** (*gratis*) o **S** (*estándar*) si F no está disponible.
    - **Aviso de IA responsable**: Aceptar.
1. Selecciona **Revisar y crear** y **Crear** para aprovisionar el recurso.
1. Espera a que se complete la implementación y, a continuación, ve al recurso implementado.
1. Visualiza la página **Claves y punto de conexión** en la sección **Administración de recursos**. Necesitarás la información de esta página más adelante en el ejercicio.

## Clonación del repositorio para este curso

Vas a desarrollar el código mediante Cloud Shell desde Azure Portal. Los archivos de código de la aplicación se han proporcionado en un repositorio de GitHub.

1. En Azure Portal, usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell***. Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    **<font color="red">Asegúrate de que has cambiado a la versión clásica de Cloud Shell antes de continuar.</font>**

1. En el panel de PowerShell, escribe los siguientes comandos para clonar el repo de GitHub para este ejercicio:

    ```
    rm -r mslearn-ai-language -f
    git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **Sugerencia**: Al escribir comandos en Cloud Shell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

1. Una vez clonado el repo, ve a la carpeta que contiene los archivos de código de aplicación:  

    ```
    cd mslearn-ai-language/Labfiles/01-analyze-text/Python/text-analysis
    ```

## Configuración de la aplicación

1. En el panel de línea de comandos, ejecute el siguiente comando para ver los archivos de código de la carpeta **text-analysis**:

    ```
   ls -a -l
    ```

    Los archivos incluyen un archivo de configuración (**.env**) y un archivo de código (**text-analysis.py**). El texto que analizará la aplicación se encuentra en la subcarpeta **reviews**.

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

1. Actualice los valores de configuración para incluir el **punto de conexión** y una **clave** del recurso de Lenguaje de Azure que ha creado (disponible en la página **Claves y punto de conexión** de su recurso de Lenguaje de Azure AI en Azure Portal)
1. Después de reemplazar los marcadores de posición, en el editor de código, usa el comando **CTRL+S** o usa la acción de **hacer clic con el botón derecho > Guardar** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** o la acción de **hacer clic con el botón derecho > Salir** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

## Adición de código para conectar con el recurso de Lenguaje de Azure AI

1. Escriba el siguiente comando para editar el archivo de código de la aplicación:

    ```
    code text-analysis.py
    ```

1. Revise el código existente. Agregará código para trabajar con el SDK de Text Analytics para Lenguaje de IA.

    > **Sugerencia**: al agregar código al archivo de código, asegúrate de mantener la sangría correcta.

1. En la parte superior del archivo de código, en las referencias de espacio de nombres existentes, busque el comentario **Importar espacios de nombres** y agregue el código siguiente para importar los espacios de nombres que necesitará usar el SDK de Text Analytics:

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. En la función **main**, observe que ya se ha proporcionado el código para cargar el punto de conexión del servicio Lenguaje de Azure AI y la clave del archivo de configuración. A continuación, busque el comentario **Create client using endpoint and key**(Crear cliente mediante el punto de conexión y la clave) y agregue el código siguiente para crear un cliente para Text Analysis API:

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Guarde los cambios (CTRL+S) y, a continuación, escriba el siguiente comando para ejecutar el programa (maximice el panel de Cloud Shell y cambie el tamaño de los paneles para ver más texto en el panel de línea de comandos):

    ```
   python text-analysis.py
    ```

1. Observe la salida, ya que el código debe ejecutarse sin errores y mostrar el contenido de cada archivo de texto de revisión en la carpeta **reviews**. La aplicación crea correctamente un cliente para Text Analytics API, pero no lo usa. Esto lo corregiremos en la sección siguiente.

## Agregar código para detectar el idioma

Ahora que ha creado un cliente para la API, vamos a usarlo para detectar el idioma en el que se escribe cada revisión.

1. En el editor de código, busque el comentario **Obtener idioma**. A continuación, agregue el código necesario para detectar el idioma de cada documento de reseña:

    ```python
   # Get language
   detectedLanguage = ai_client.detect_language(documents=[text])[0]
   print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

     > **Nota**: *En este ejemplo, cada revisión se analiza individualmente, lo que da lugar a una llamada independiente al servicio para cada archivo. Un enfoque alternativo es crear una colección de documentos y pasarlos al servicio en una sola llamada. En ambos enfoques, la respuesta del servicio consta de una colección de documentos; por eso, en el código de Python anterior, se especifica el índice del primer (y único) documento en la respuesta ([0]).*

1. Guarda los cambios. A continuación, vuelva a ejecutar el programa.
1. Observe la salida y tenga en cuenta que esta vez se identifica el idioma de cada reseña.

## Agregar código para evaluar la opinión

El *análisis de sentimiento* es una técnica que se usa habitualmente para clasificar el texto como *positivo* o *negativo* (o posiblemente *neutro* o *mixto*). Se usa normalmente para analizar publicaciones en redes sociales, reseñas de productos y otros elementos en los que la opinión del texto puede proporcionar información útil.

1. En el editor de código, busque el comentario **Obtener opinión**. A continuación, agregue el código necesario para detectar la opinión de cada documento de reseña:

    ```python
   # Get sentiment
   sentimentAnalysis = ai_client.analyze_sentiment(documents=[text])[0]
   print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

1. Guarda los cambios. A continuación, cierre el editor de código y vuelva a ejecutar el programa.
1. Observe la salida y que se detecta la opinión de las revisiones.

## Agregar código para identificar frases clave

Puede ser útil identificar frases clave en un cuerpo de texto para ayudar a determinar los temas principales que se tratan.

1. En el editor de código, busque el comentario **Obtener frases clave**. A continuación, agregue el código necesario para detectar las frases clave en cada documento de reseña:

    ```python
   # Get key phrases
   phrases = ai_client.extract_key_phrases(documents=[text])[0].key_phrases
   if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

1. Guarde los cambios y vuelva a ejecutar el programa.
1. Observe el resultado y que cada documento contiene frases clave que proporcionan información sobre el tema de la revisión.

## Agregar entidades para extraer datos

A menudo, los documentos u otros cuerpos de texto mencionan personas, lugares, períodos de tiempo u otras entidades. Text Analytics API puede detectar varias categorías (y subcategorías) de la entidad en el texto.

1. En el editor de código, busque el comentario **Obtener entidades**. A continuación, agregue el código necesario para identificar las entidades que se mencionan en cada reseña:

    ```python
   # Get entities
   entities = ai_client.recognize_entities(documents=[text])[0].entities
   if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

1. Guarde los cambios y vuelva a ejecutar el programa.
1. Observe la salida y las entidades que se han detectado en el texto.

## Agregar código para extraer entidades vinculadas

Además de las entidades clasificadas, Text Analytics API puede detectar entidades para las que hay vínculos conocidos a orígenes de datos, como Wikipedia.

1. En el editor de código, busque el comentario **Obtener entidades vinculadas**. A continuación, agregue el código necesario para identificar las entidades vinculadas que se mencionan en cada reseña:

    ```python
   # Get linked entities
   entities = ai_client.recognize_linked_entities(documents=[text])[0].entities
   if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

1. Guarde los cambios y vuelva a ejecutar el programa.
1. Observe la salida y las entidades vinculadas que se identifican.

## Limpieza de recursos

Sugerencia: Si ha terminado de explorar el servicio Lenguaje de Azure AI, puede eliminar los recursos que creó en este ejercicio. A continuación, se indica cómo puede hacerlo.

1. Cierre del panel de Azure Cloud Shell
1. En Azure Portal, vaya al recurso de Lenguaje de Azure AI que creó en este laboratorio.
1. En la página del recurso, seleccione **Eliminar** y siga las instrucciones para eliminar el recurso.

## Información adicional

Para más información sobre cómo usar **Lenguaje de Azure AI**, consulte la [documentación](https://learn.microsoft.com/azure/ai-services/language-service/).
