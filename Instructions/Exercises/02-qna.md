---
lab:
  title: Creación de una solución de respuesta a preguntas
  module: Module 6 - Create question answering solutions with Azure AI Language
---

# Creación de una solución de respuesta a preguntas

Uno de los escenarios de conversación más comunes es proporcionar soporte técnico a través de knowledge bases de preguntas más frecuentes (P+F). Muchas organizaciones publican preguntas más frecuentes como documentos o páginas web, lo que funciona bien para un pequeño conjunto de pares de preguntas y respuestas, pero los documentos grandes pueden ser difíciles y lentos de buscar.

**Lenguaje de Azure AI** incluye una capacidad de *respuesta a preguntas* que permite crear una knowledge base de pares de preguntas y respuestas consultables mediante la entrada de lenguaje natural. Normalmente, esta capacidad se usa como un recurso que un bot puede emplear para buscar respuestas a las preguntas enviadas por los usuarios.

## Aprovisionar un recurso de *Lenguaje de Azure AI*

Si aún no tiene uno en su suscripción, deberá aprovisionar un recurso del **servicio de Lenguaje de Azure AI**. Además, para crear y hospedar una knowledge base para la respuesta a preguntas, debe habilitar la característica **Respuesta a preguntas**.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
1. En el campo de búsqueda de la parte superior, escriba **Servicios de Azure AI** y presione **Entrar**.
1. Seleccione **Crear** en el recurso **Servicio de lenguaje**, en los resultados.
1. **Seleccione** el bloque **Personalizar respuesta a preguntas**. Después, seleccione **Continuar para crear el recurso**. Deberá especificar la siguiente configuración:

    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos*.
    - **Región**: *elija cualquier ubicación disponible*
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa**: seleccione **F0** (*gratis*), o **S** (*estándar*) si F no está disponible.
    - **Región de Azure Search**: *elija una ubicación en la misma región global que el recurso de Lenguaje*
    - **Plan de tarifa de Azure Search**: Gratis (F) (*Si este plan no está disponible, seleccione Básico (B)* )
    - **Aviso de IA responsable**: *Aceptar*

1. Seleccione **Crear y revisar** y, luego seleccione **Crear**.

    > **NOTA** Respuestas a preguntas personalizadas usa Azure Search para indexar y consultar la knowledge base de preguntas y respuestas.

1. Espere a que se complete la implementación y, a continuación, vaya al recurso implementado.
1. Consulte la página **Claves y punto de conexión**. Necesitará la información de esta página más adelante en el ejercicio.

## Creación de un proyecto de respuesta a preguntas

Si quiere crear una knowledge base para responder a preguntas en el recurso de Lenguaje de Azure AI, puede usar el portal Language Studio para crear un proyecto de respuesta a preguntas. En este caso, creará una knowledge base que contiene preguntas y respuestas sobre [Microsoft Learn](https://docs.microsoft.com/learn).

1. En una pestaña nueva del explorador, vaya el portal de Language Studio en [https://language.cognitive.azure.com/](https://language.cognitive.azure.com/) e inicie sesión con la cuenta de Microsoft asociada a su suscripción de Azure.
1. Si se le pide que elija un recurso de Language, seleccione la siguiente configuración:
    - **Directorio de Azure**: directorio de Azure que contiene la suscripción.
    - **Suscripción de Azure**: su suscripción a Azure.
    - **Tipo de recurso**: idioma
    - **Nombre del recurso**: el recurso de Lenguaje de Azure AI que creó antes.

    Si <u>no</u> se le pide que elija un recurso de Language, puede deberse a que tiene varios recursos de Language en la suscripción, en cuyo caso:

    1. En la barra de la parte superior de la página, seleccione el botón **Configuración (&#9881;)**.
    2. En la página **Configuración,** vea la pestaña **Recursos**.
    3. Seleccione el recurso de Language que acaba de crear y haga clic en **Switch resource** (Cambiar recurso).
    4. En la parte superior de la página, haga clic en **Language Studio** para volver a la página principal de Language Studio.

1. En la parte superior del portal, en el menú **Crear nuevo**, seleccione **Respuesta a preguntas personalizada**.
1. En el asistente para ***crear un proyecto**, en la página **Elegir configuración de idioma**, seleccione la opción para **establecer el idioma de todos los proyectos de este recurso** y seleccione **Inglés** como idioma. Luego, seleccione **Siguiente**.
1. En la página **Escribir información básica**, escriba los siguientes detalles:
    - **Nombre** `LearnFAQ`
    - **Descripción**: `FAQ for Microsoft Learn`
    - **Respuesta predeterminada cuando no se devuelve ninguna respuesta**: `Sorry, I don't understand the question`
1. Seleccione **Siguiente**.
1. En la página **Revisar y finalizar**, seleccione **Crear proyecto**.

## Agregar orígenes la knowledge base

Puede crear una knowledge base desde cero, pero es habitual empezar importando preguntas y respuestas desde una página o documento de preguntas frecuentes existente. En este caso, importará datos desde una página web de preguntas más frecuentes existente para Microsoft Learn. También importará algunas preguntas y respuestas predefinidas de tipo charla para admitir intercambios de conversacionales comunes.

1. En la página **Administrar orígenes** del proyecto de respuesta a preguntas, en la lista **&#9547; Agregar origen**, seleccione **Direcciones URL**. A continuación, en el cuadro de diálogo **Agregar URL**, seleccione **&#9547; Agregar URL** y configure el siguiente nombre y URL antes de seleccionar **Agregar todo** para agregarla a la knowledge base:
    - **Nombre**: `Learn FAQ Page`
    - **URL**: `https://docs.microsoft.com/en-us/learn/support/faq`
1. En la página **Administrar orígenes** del proyecto de respuesta a preguntas, en la lista **&#9547; Agregar origen**, seleccione **Charla**. En el cuadro de diálogo **Agregar charla**, seleccione **Amistosa** y seleccione **Agregar charla**.

## Edición de la base de conocimiento

La knowledge base se ha rellenado con pares de preguntas y respuestas de las preguntas más frecuentes de Microsoft Learn, complementadas con un conjunto de pares de preguntas y respuestas de *charla* conversacional. Puede ampliar la knowledge base agregando pares de preguntas y respuestas adicionales.

1. En el proyecto **LearnFAQ** de Language Studio, seleccione la página **Editar knowledge base** para ver los pares de pregunta y respuesta existentes. (si se muestran algunas sugerencias, léalas y elija **Entendido** para descartarlas, o seleccione **Omitir todo**)
1. En la knowledge base, en la pestaña **Pares de preguntas y respuestas**, seleccione **&#65291;** y cree un nuevo par de pregunta y respuesta con la siguiente configuración:
    - **Origen**: `https://docs.microsoft.com/en-us/learn/support/faq`
    - **Pregunta**: `What are Microsoft credentials?`
    - **Respuesta**: `Microsoft credentials enable you to validate and prove your skills with Microsoft technologies.`
1. Seleccione **Listo**.
1. En la página de la pregunta **¿Qué son las credenciales de Microsoft?** que se crea, expanda **Preguntas alternativas**. A continuación, agregue la pregunta alternativa `How can I demonstrate my Microsoft technology skills?`.

    En algunos casos, tiene sentido permitir que el usuario realice un seguimiento de la respuesta mediante la creación de una conversación *multiturno* que permita al usuario refinar iterativamente la pregunta para llegar a la respuesta que necesita.

1. En la respuesta que escribió para la pregunta sobre certificación, expanda **Solicitudes de seguimiento** y agregue la siguiente solicitud de seguimiento:
    - **Texto que se muestra en el símbolo del sistema al usuario**: `Learn more about credentials`.
    - Seleccione la pestaña **Crear vínculo a nuevo par** y escriba este texto: `You can learn more about credentials on the [Microsoft credentials page](https://docs.microsoft.com/learn/credentials/).`
    - Seleccione **Mostrar solo en el flujo contextual**. Esta opción garantiza que la respuesta solo se devuelve en el contexto de una pregunta de seguimiento a partir de la pregunta de certificación original.
1. Seleccione **Agregar consulta**.

## Entrenamiento y prueba de la knowledge base

Ahora que ha creado una knowledge base, es el momento de probarla en Language Studio.

1. Guarde los cambios en la knowledge base, para ello, seleccione el botón **Guardar** debajo de la pestaña **Pares de respuesta a preguntas** de la izquierda.
1. Una vez guardados los cambios, seleccione el botón **Probar** para abrir el panel de pruebas.
1. En el panel de pruebas, en la parte superior, anule la selección de **Incluir respuesta corta** (si aún no se ha anulado la selección). A continuación, en la parte inferior, escriba el mensaje `Hello`. Debería devolver una respuesta adecuada.
1. En el panel de prueba, en la parte inferior, escriba el mensaje `What is Microsoft Learn?`. Se debe devolver una respuesta adecuada de las preguntas más frecuentes.
1. Escriba el mensaje `Thanks!`. Debería devolverse una respuesta de charla adecuada.
1. Escriba el mensaje `Tell me about Microsoft credentials`. Debería devolverse la respuesta que creó junto con un vínculo de solicitud de seguimiento.
1. Seleccione el vínculo de seguimiento **Más información sobre las credenciales**. Debería devolverse la respuesta de seguimiento con un vínculo a la página de certificación.
1. Cuando haya terminado de probar la knowledge base, cierre el panel de prueba.

## Implementar la knowledge base

La knowledge base proporciona un servicio back-end que las aplicaciones cliente pueden usar para responder preguntas. Ahora ya está listo para publicar la knowledge base y acceder a su interfaz REST desde un cliente.

1. En el proyecto **LearnFAQ** de Language Studio, seleccione la página **Implementar knowledge base**.
1. En la parte superior de la página, seleccione **Implementar**. A continuación, seleccione **Implementar** para confirmar que quiere implementar la knowledge base.
1. Una vez completada la implementación, seleccione **Obtener URL de predicción** para ver el punto de conexión REST de la knowledge base y tenga en cuenta que la solicitud de ejemplo incluye parámetros para lo siguiente:
    - **projectName**: el nombre del proyecto (que debe ser *LearnFAQ*)
    - **deploymentName**: el nombre de la implementación (que debe ser *production*)
1. Cierre el cuadro de diálogo de la URL de predicción.

## Preparación para desarrollar una aplicación en Visual Studio Code

Desarrollará la aplicación de respuesta a preguntas mediante Visual Studio Code. Los archivos de código de la aplicación se han proporcionado en un repositorio de GitHub.

> **Sugerencia**: Si ya ha clonado el repositorio **mslearn-ai-language**, ábralo en Visual Studio Code. De lo contrario, siga estos pasos para clonarlo en el entorno de desarrollo.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/mslearn-ai-language` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.

    > **Nota**: Si Visual Studio Code muestra un mensaje emergente para solicitarle que confíe en el código que está abriendo, haga clic en la opción **Sí, confío en los autores** en el elemento emergente.

4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## Configuración de la aplicación

Se han proporcionado aplicaciones para C# y Python, así como un archivo de texto de ejemplo que usará para probar el resumen. Las dos aplicaciones tienen la misma funcionalidad. Primero, completará algunas partes clave de la aplicación para habilitar que utilice el recurso de Lenguaje de Azure AI.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **Labfiles/02-qna** y expanda la carpeta **CSharp** o **Python** según sus preferencias de lenguaje y la carpeta **qna-app** que contiene. Cada carpeta contiene los archivos específicos del lenguaje de una aplicación en la que va a integrar la funcionalidad de respuesta a preguntas de Lenguaje de Azure AI.
2. Haga clic con el botón derecho en la carpeta **qna-app** que contiene los archivos de código y abra un terminal integrado. A continuación, instale el paquete del SDK de respuesta a preguntas de Lenguaje de Azure AI mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#:**

    ```
    dotnet add package Azure.AI.Language.QuestionAnswering
    ```

    **Python**:

    ```
    pip install azure-ai-language-questionanswering
    ```

3. En el panel **Explorador**, en la carpeta **qna-app**, abra el archivo de configuración para su lenguaje preferido.

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Actualice los valores de configuración para incluir el **punto de conexión** y una **clave** del recurso de Lenguaje de Azure que creó (disponible en la página **Claves y punto de conexión** del recurso de Lenguaje de Azure AI en Azure Portal). El nombre del proyecto y el nombre de implementación de la knowledge base también deben estar en este archivo.
5. Guarde el archivo de configuración.

## Agregar código a la aplicación

Ahora está listo para agregar el código necesario para importar las bibliotecas de SDK necesarias, establecer una conexión autenticada al proyecto implementado y enviar preguntas.

1. Tenga en cuenta que la carpeta **qna-app** contiene un archivo de código para la aplicación cliente:

    - **C#**: Program.cs
    - **Python**: qna-app.py

    Abra el archivo de código y, en la parte superior, en las referencias de espacio de nombres existentes, busque el comentario **Importar espacios de nombres**. A continuación, en este comentario, agregue el siguiente código específico del lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Text Analytics:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.Language.QuestionAnswering;
    ```

    **Python**: qna-app.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.questionanswering import QuestionAnsweringClient
    ```

1. En la función **Main (Principal)**, observe que ya se ha proporcionado código para cargar la clave y el punto de conexión de servicio de Lenguaje de Azure AI del archivo de configuración. A continuación, busque el comentario **Create client using endpoint and key**(Crear cliente mediante el punto de conexión y la clave) y agregue el código siguiente para crear un cliente para Text Analysis API:

    **C#**: Programs.cs

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    QuestionAnsweringClient aiClient = new QuestionAnsweringClient(endpoint, credentials);
    ```

    **Python**: qna-app.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = QuestionAnsweringClient(endpoint=ai_endpoint, credential=credential)
    ```

1. En la función **Main (Principal)**, busque el comentario **Enviar una pregunta y mostrar la respuesta** y agregue el siguiente código para leer repetidamente preguntas de la línea de comandos, enviarlas al servicio y mostrar los detalles de las respuestas:

    **C#**: Programs.cs

    ```C#
    // Submit a question and display the answer
    string user_question = "";
    while (user_question.ToLower() != "quit")
        {
            Console.Write("Question: ");
            user_question = Console.ReadLine();
            QuestionAnsweringProject project = new QuestionAnsweringProject(projectName, deploymentName);
            Response<AnswersResult> response = aiClient.GetAnswers(user_question, project);
            foreach (KnowledgeBaseAnswer answer in response.Value.Answers)
            {
                Console.WriteLine(answer.Answer);
                Console.WriteLine($"Confidence: {answer.Confidence:P2}");
                Console.WriteLine($"Source: {answer.Source}");
                Console.WriteLine();
            }
        }
    ```

    **Python**: qna-app.py

    ```Python
    # Submit a question and display the answer
    user_question = ''
    while user_question.lower() != 'quit':
        user_question = input('\nQuestion:\n')
        response = ai_client.get_answers(question=user_question,
                                        project_name=ai_project_name,
                                        deployment_name=ai_deployment_name)
        for candidate in response.answers:
            print(candidate.answer)
            print("Confidence: {}".format(candidate.confidence))
            print("Source: {}".format(candidate.source))
    ```

1. Guarde los cambios y vuelva al terminal integrado de la carpeta **qna-app** y escriba el siguiente comando para ejecutar el programa:

    - **C#**: `dotnet run`
    - **Python**: `python qna-app.py`

    > **Sugerencia**: Puede usar el icono **Maximizar el tamaño del panel **(**^**) en la barra de herramientas del terminal para ver más del texto de la consola.

1. Cuando se le solicite, escriba una pregunta que se enviará al proyecto de respuesta a preguntas; por ejemplo `What is a learning path?`.
1. Revise la respuesta que se devuelve.
1. Haga más preguntas. Cuando haya terminado, escriba `quit`.

## Limpieza de recursos

Sugerencia: Si ha terminado de explorar el servicio Lenguaje de Azure AI, puede eliminar los recursos que creó en este ejercicio. A continuación, se indica cómo puede hacerlo.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
2. Vaya al recurso de Lenguaje de Azure AI que creó en este laboratorio.
3. En la página del recurso, seleccione **Eliminar** y siga las instrucciones para eliminar el recurso.

## Más información

Para más información sobre la respuesta a preguntas de Lenguaje de Azure AI, consulte la documentación de [Lenguaje de Azure AI](https://learn.microsoft.com/azure/ai-services/language-service/question-answering/overview).
