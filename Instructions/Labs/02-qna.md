---
lab:
  title: Creación de una solución de respuesta a preguntas
  description: Usar Lenguaje de Azure AI para crear una solución personalizada de respuesta a preguntas.
---

# Creación de una solución de respuesta a preguntas

Uno de los escenarios de conversación más comunes es proporcionar soporte técnico a través de knowledge bases de preguntas más frecuentes (P+F). Muchas organizaciones publican preguntas más frecuentes como documentos o páginas web, lo que funciona bien para un pequeño conjunto de pares de preguntas y respuestas, pero los documentos grandes pueden ser difíciles y lentos de buscar.

**Lenguaje de Azure AI** incluye una capacidad de *respuesta a preguntas* que permite crear una knowledge base de pares de preguntas y respuestas consultables mediante la entrada de lenguaje natural. Normalmente, esta capacidad se usa como un recurso que un bot puede emplear para buscar respuestas a las preguntas enviadas por los usuarios. En este ejercicio, usará el SDK de Python de Lenguaje Azure AI para análisis de texto para implementar una aplicación sencilla de respuesta a preguntas.

Aunque este ejercicio se basa en Python, puede desarrollar aplicaciones similares usando varios SDK específicos del lenguaje, como:

- [Biblioteca cliente de respuesta a preguntas de Azure AI Language Service para Python](https://pypi.org/project/azure-ai-language-questionanswering/)
- [Biblioteca cliente de respuesta a preguntas de Azure AI Language Service para .NET](https://www.nuget.org/packages/Azure.AI.Language.QuestionAnswering)

Este ejercicio dura aproximadamente **20** minutos.

## Aprovisionar un recurso de *Lenguaje de Azure AI*

Si aún no tiene uno en su suscripción, deberá aprovisionar un recurso del **servicio de Lenguaje de Azure AI**. Además, para crear y hospedar una knowledge base para la respuesta a preguntas, debe habilitar la característica **Respuesta a preguntas**.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
1. Seleccione **Crear un recurso**.
1. En el campo de búsqueda, busca **Servicio de lenguaje**. A continuación, en los resultados, seleccione **Crear** bajo **Servicio de lenguaje**.
1. Selecciona el bloque **Personalizar respuesta a preguntas**. Después, seleccione **Continuar para crear el recurso**. Deberá especificar la siguiente configuración:

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
1. Visualiza la página **Claves y punto de conexión** en la sección **Administración de recursos**. Necesitará la información de esta página más adelante en el ejercicio.

## Creación de un proyecto de respuesta a preguntas

Si quiere crear una knowledge base para responder a preguntas en el recurso de Lenguaje de Azure AI, puede usar el portal Language Studio para crear un proyecto de respuesta a preguntas. En este caso, creará una knowledge base que contiene preguntas y respuestas sobre [Microsoft Learn](https://learn.microsoft.com/training/).

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
1. En el asistente para ***Crear un proyecto**, en la página **Elegir configuración de idioma**, selecciona la opción para **Seleccionar el idioma de todos los proyectos** y selecciona **Inglés** como idioma. Seleccione **Siguiente**.
1. En la página **Escribir información básica**, escriba los siguientes detalles:
    - **Nombre** `LearnFAQ`
    - **Descripción**: `FAQ for Microsoft Learn`
    - **Respuesta predeterminada cuando no se devuelve ninguna respuesta**: `Sorry, I don't understand the question`
1. Seleccione **Siguiente**.
1. En la página **Revisar y finalizar**, seleccione **Crear proyecto**.

## Agregar orígenes la knowledge base

Puede crear una knowledge base desde cero, pero es habitual empezar importando preguntas y respuestas desde una página o documento de preguntas frecuentes existente. En este caso, importarás datos desde una página web de preguntas más frecuentes existente para Microsoft Learn. También importarás algunas preguntas y respuestas predefinidas de tipo charla para admitir intercambios de conversacionales comunes.

1. En la página **Administrar orígenes** del proyecto de respuesta a preguntas, en la lista **&#9547; Agregar origen**, seleccione **Direcciones URL**. A continuación, en el cuadro de diálogo **Agregar URL**, seleccione **&#9547; Agregar URL** y configure el siguiente nombre y URL antes de seleccionar **Agregar todo** para agregarla a la knowledge base:
    - **Nombre**: `Learn FAQ Page`
    - **URL**: `https://learn.microsoft.com/en-us/training/support/faq?pivots=general`
1. En la página **Administrar orígenes** del proyecto de respuesta a preguntas, en la lista **&#9547; Agregar origen**, seleccione **Charla**. En el cuadro de diálogo **Agregar charla**, seleccione **Amistosa** y seleccione **Agregar charla**.

## Edición de la base de conocimiento

La knowledge base se ha rellenado con pares de preguntas y respuestas de las preguntas más frecuentes de Microsoft Learn, complementadas con un conjunto de pares de preguntas y respuestas de *charla* conversacional. Puede ampliar la knowledge base agregando pares de preguntas y respuestas adicionales.

1. En el proyecto **LearnFAQ** de Language Studio, seleccione la página **Editar knowledge base** para ver los pares de pregunta y respuesta existentes. (si se muestran algunas sugerencias, léalas y elija **Entendido** para descartarlas, o seleccione **Omitir todo**)
1. En la knowledge base, en la pestaña **Pares de preguntas y respuestas**, seleccione **&#65291;** y cree un nuevo par de pregunta y respuesta con la siguiente configuración:
    - **Origen**: `https://learn.microsoft.com/en-us/training/support/faq?pivots=general`
    - **Pregunta**: `What are the different types of modules on Microsoft Learn?`
    - **Respuesta**: `Microsoft Learn offers various types of training modules, including role-based learning paths, product-specific modules, and hands-on labs. Each module contains units with lessons and knowledge checks to help you learn at your own pace.`
1. Selecciona **Listo.**
1. En la página de la pregunta **¿Cuáles son los distintos tipos de módulos de Microsoft Learn?** que se crea, expanda la sección **Preguntas alternativas**. A continuación, agregue la pregunta alternativa `How are training modules organized?`.

    En algunos casos, tiene sentido permitir que el usuario realice un seguimiento de la respuesta mediante la creación de una conversación *multiturno* que permita al usuario refinar iterativamente la pregunta para llegar a la respuesta que necesita.

1. En la respuesta especificada para la pregunta de los tipos de módulos, expanda **Solicitudes de seguimiento** y agregue la siguiente solicitud de seguimiento:
    - **Texto que se muestra en el símbolo del sistema al usuario**: `Learn more about training`.
    - Seleccione la pestaña **Crear vínculo a nuevo par** y escriba este texto: `You can explore modules and learning paths on the [Microsoft Learn training page](https://learn.microsoft.com/training/).`
    - Seleccione **Mostrar solo en el flujo contextual**. Esta opción garantiza que la respuesta solo se devuelva en el contexto de una pregunta de seguimiento de la pregunta original de los tipos de módulos.
1. Seleccione **Agregar consulta**.

## Entrenamiento y prueba de la knowledge base

Ahora que ha creado una knowledge base, es el momento de probarla en Language Studio.

1. Guarde los cambios en la knowledge base, para ello, seleccione el botón **Guardar** debajo de la pestaña **Pares de respuesta a preguntas** de la izquierda.
1. Una vez guardados los cambios, seleccione el botón **Probar** para abrir el panel de pruebas.
1. En el panel de pruebas, en la parte superior, anule la selección de **Incluir respuesta corta** (si aún no se ha anulado la selección). A continuación, en la parte inferior, escriba el mensaje `Hello`. Debería devolver una respuesta adecuada.
1. En el panel de prueba, en la parte inferior, escriba el mensaje `What is Microsoft Learn?`. Se debe devolver una respuesta adecuada de las preguntas más frecuentes.
1. Escriba el mensaje `Thanks!`. Debería devolverse una respuesta de charla adecuada.
1. Escriba el mensaje `What are the different types of modules on Microsoft Learn?`. Debería devolverse la respuesta que creó junto con un vínculo de solicitud de seguimiento.
1. Seleccione el vínculo de seguimiento **Más información sobre el entrenamiento**. Se devolverá la respuesta de seguimiento con un vínculo a la página de entrenamiento.
1. Cuando haya terminado de probar la knowledge base, cierre el panel de prueba.

## Implementar la knowledge base

La knowledge base proporciona un servicio back-end que las aplicaciones cliente pueden usar para responder preguntas. Ahora ya está listo para publicar la knowledge base y acceder a su interfaz REST desde un cliente.

1. En el proyecto **LearnFAQ** de Language Studio, selecciona la página **Implementar knowledge base** desde el menú de navegación de la izquierda.
1. En la parte superior de la página, seleccione **Implementar**. A continuación, seleccione **Implementar** para confirmar que quiere implementar la knowledge base.
1. Una vez completada la implementación, seleccione **Obtener URL de predicción** para ver el punto de conexión REST de la knowledge base y tenga en cuenta que la solicitud de ejemplo incluye parámetros para lo siguiente:
    - **projectName**: el nombre del proyecto (que debe ser *LearnFAQ*)
    - **deploymentName**: el nombre de la implementación (que debe ser *production*)
1. Cierre el cuadro de diálogo de la URL de predicción.

## Preparación para desarrollar una aplicación en Cloud Shell

Desarrollará la aplicación de respuesta a preguntas mediante Cloud Shell en Azure Portal. Los archivos de código de la aplicación se han proporcionado en un repositorio de GitHub.

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
    cd mslearn-ai-language/Labfiles/02-qna/Python/qna-app
    ```

## Configuración de la aplicación

1. En el panel de línea de comandos, ejecute el siguiente comando para ver los archivos de código de la carpeta **qna-app**:

    ```
   ls -a -l
    ```

    Los archivos incluyen un archivo de configuración (**.env**) y un archivo de código (**qna-app.py**).

1. Cree un entorno virtual de Python e instale el paquete del SDK de respuesta a preguntas de Lenguaje de Azure AI y otros paquetes necesarios mediante la ejecución del siguiente comando:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-language-questionanswering
    ```

1. Use el comando siguiente para editar el archivo de configuración:

    ```
    code .env
    ```

    El archivo se abre en un editor de código.

1. En el archivo de código, actualice los valores de configuración que contiene para reflejar el **punto de conexión** y una **clave** de autenticación para el recurso de Lenguaje de Azure que creó (disponible en la página **Claves y punto de conexión** del recurso de Lenguaje de Azure AI en Azure Portal). El nombre del proyecto y el nombre de implementación de la knowledge base también deben estar en este archivo.
1. Después de reemplazar los marcadores de posición, en el editor de código, usa el comando **CTRL+S** o usa la acción de **hacer clic con el botón derecho > Guardar** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** o la acción de **hacer clic con el botón derecho > Salir** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

## Adición de código para usar la base de conocimiento

1. Escriba el siguiente comando para editar el archivo de código de la aplicación:

    ```
    code qna-app.py
    ```

1. Revise el código existente. Agregará código para trabajar con la base de conocimiento.

    > **Sugerencia**: al agregar código al archivo de código, asegúrate de mantener la sangría correcta.

1. En el archivo de código, busque el comentario **Importar espacios de nombres**. A continuación, en este comentario, agregue el siguiente código específico del lenguaje para importar los espacios de nombres que necesitará para usar el SDK de respuesta a preguntas:

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.language.questionanswering import QuestionAnsweringClient
    ```

1. En la función **main**, observe que ya se ha proporcionado el código para cargar el punto de conexión del servicio Lenguaje de Azure AI y la clave del archivo de configuración. A continuación, busque el comentario **Crear cliente mediante el punto de conexión y la clave** y agregue el código siguiente para crear un cliente de respuesta a preguntas:

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = QuestionAnsweringClient(endpoint=ai_endpoint, credential=credential)
    ```

1. En el archivo de código, busque el comentario **Enviar una pregunta y mostrar la respuesta** y agregue el código siguiente para leer repetidamente preguntas desde la línea de comandos, enviarlas al servicio y mostrar los detalles de las respuestas:

    ```Python
   # Submit a question and display the answer
   user_question = ''
   while True:
        user_question = input('\nQuestion:\n')
        if user_question.lower() == "quit":                
            break
        response = ai_client.get_answers(question=user_question,
                                        project_name=ai_project_name,
                                        deployment_name=ai_deployment_name)
        for candidate in response.answers:
            print(candidate.answer)
            print("Confidence: {}".format(candidate.confidence))
            print("Source: {}".format(candidate.source))
    ```

1. Guarde los cambios (CTRL+S) y, a continuación, escriba el siguiente comando para ejecutar el programa (maximice el panel de Cloud Shell y cambie el tamaño de los paneles para ver más texto en el panel de línea de comandos):

    ```
   python qna-app.py
    ```

1. Cuando se le solicite, escriba una pregunta que se enviará al proyecto de respuesta a preguntas; por ejemplo `What is a learning path?`.
1. Revise la respuesta que se devuelve.
1. Haga más preguntas. Cuando haya terminado, escriba `quit`.

## Limpieza de recursos

Sugerencia: Si ha terminado de explorar el servicio Lenguaje de Azure AI, puede eliminar los recursos que creó en este ejercicio. A continuación, se indica cómo puede hacerlo.

1. Cierre del panel de Azure Cloud Shell
1. En Azure Portal, vaya al recurso de Lenguaje de Azure AI que creó en este laboratorio.
1. En la página del recurso, seleccione **Eliminar** y siga las instrucciones para eliminar el recurso.

## Más información

Para más información sobre la respuesta a preguntas de Lenguaje de Azure AI, consulte la documentación de [Lenguaje de Azure AI](https://learn.microsoft.com/azure/ai-services/language-service/question-answering/overview).
