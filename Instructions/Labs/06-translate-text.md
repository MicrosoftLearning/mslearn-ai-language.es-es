---
lab:
  title: Traducción de texto
  description: Traduzca el texto proporcionado entre los idiomas admitidos con Traductor de Azure AI.
---

# Traducción de texto

**Traductor de Azure AI** es un servicio que permite traducir texto entre idiomas. En este ejercicio, lo usarás para crear una aplicación sencilla que traduzca la entrada en cualquier idioma admitido al idioma de destino que prefiera.

Aunque este ejercicio se basa en Python, puede desarrollar aplicaciones similares usando varios SDK específicos del lenguaje, como:

- [Biblioteca cliente de Azure AI Translation para Python](https://pypi.org/project/azure-ai-translation-text/)
- [Biblioteca cliente de Azure AI Translation para .NET](https://www.nuget.org/packages/Azure.AI.Translation.Text)
- [Biblioteca cliente de Azure AI Translation para JavaScript](https://www.npmjs.com/package/@azure-rest/ai-translation-text)

Este ejercicio dura aproximadamente **30** minutos.

## Aprovisionar un recurso del *Traductor de Azure AI*

Si aún no tienes uno en tu suscripción, deberás aprovisionar un recurso del **Traductor de Azure AI**.

1. Inicia sesión en Azure Portal en `https://portal.azure.com` y regístrate con la cuenta de Microsoft asociada a tu suscripción a Azure.
1. En el campo de búsqueda de la parte superior, busque **Translators** y seleccione **Translators** en los resultados.
1. Crea un recurso con los valores siguientes:
    - **Suscripción**: *suscripción a Azure*
    - **Grupo de recursos**: *elige o crea un grupo de recursos*
    - **Región**: *elige cualquier región disponible*
    - **Nombre**: *escribe un nombre único*
    - **Plan de tarifa**: selecciona **F0** (*gratis*), o **S** (*estándar*) si F no está disponible.
1. Selecciona **Revisar y crear** y **Crear** para aprovisionar el recurso.
1. Espera a que se complete la implementación y, a continuación, ve al recurso implementado.
1. Consulta la página **Claves y punto de conexión**. Necesitarás la información de esta página más adelante en el ejercicio.

## Preparación para desarrollar una aplicación en Cloud Shell

Para probar las funcionalidades de traducción de texto de Traductor de Azure AI, desarrollará una aplicación de consola sencilla en Azure Cloud Shell.

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
   cd mslearn-ai-language/Labfiles/06-translator-sdk/Python/translate-text
    ```

## Configuración de la aplicación

1. En el panel de línea de comandos, ejecute el siguiente comando para ver los archivos de código en la carpeta **translate-text**:

    ```
   ls -a -l
    ```

    Los archivos incluyen un archivo de configuración (**.env**) y un archivo de código (**translate.py**).

1. Cree un entorno virtual de Python e instale el paquete del SDK de Azure AI Translation y otros paquetes necesarios mediante la ejecución del siguiente comando:

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-translation-text==1.0.1
    ```

1. Escriba el siguiente comando para editar el archivo de configuración de la aplicación:

    ```
   code .env
    ```

    El archivo se abre en un editor de código.

1. Actualiza los valores de configuración para incluir la **región** y una **clave** del recurso del Traductor de Azure AI que creaste (disponible en la página **Claves y punto de conexión** del recurso del Traductor de Azure AI en Azure Portal).

    > **NOTA**: asegúrate de agregar la *región* del recurso, <u>no</u> el punto de conexión.

1. Después de reemplazar los marcadores de posición, en el editor de código, usa el comando **CTRL+S** o usa la acción de **hacer clic con el botón derecho > Guardar** para guardar los cambios y, a continuación, usa el comando **CTRL+Q** o la acción de **hacer clic con el botón derecho > Salir** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

## Agregar código para traducir texto

1. Escriba el siguiente comando para editar el archivo de código de la aplicación:

    ```
   code translate.py
    ```

1. Revise el código existente. Agregará código para trabajar con el SDK de Azure AI Translation.

    > **Sugerencia**: al agregar código al archivo de código, asegúrate de mantener la sangría correcta.

1. En la parte superior del archivo de código, en las referencias de espacio de nombres existentes, busque el comentario **Importar espacios de nombres** y agregue el código siguiente para importar los espacios de nombres que necesitará usar el SDK de traducción:

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.translation.text import *
   from azure.ai.translation.text.models import InputTextItem
    ```

1. En la función **main**, tenga en cuenta que el código existente lee las opciones de configuración.
1. Busque el comentario **Crear cliente mediante el punto de conexión y la clave** y agregue el siguiente código:

    ```python
   # Create client using endpoint and key
   credential = AzureKeyCredential(translatorKey)
   client = TextTranslationClient(credential=credential, region=translatorRegion)
    ```

1. Busque el comentario **Elegir idioma de destino** y agregue el código siguiente, que usa el servicio Text Translator para devolver la lista de idiomas de traducción admitidos y pide al usuario que seleccione un código de idioma para el idioma de destino:

    ```python
   # Choose target language
   languagesResponse = client.get_supported_languages(scope="translation")
   print("{} languages supported.".format(len(languagesResponse.translation)))
   print("(See https://learn.microsoft.com/azure/ai-services/translator/language-support#translation)")
   print("Enter a target language code for translation (for example, 'en'):")
   targetLanguage = "xx"
   supportedLanguage = False
   while supportedLanguage == False:
        targetLanguage = input()
        if  targetLanguage in languagesResponse.translation.keys():
            supportedLanguage = True
        else:
            print("{} is not a supported language.".format(targetLanguage))
    ```

1. Busque el comentario **Traducir texto** y agregue el código siguiente, que solicita repetidamente al usuario texto para traducir, utiliza el servicio Traductor de Azure AI para traducirlo al idioma de destino (detectando automáticamente el idioma de origen) y muestra los resultados hasta que el usuario escribe *quit*:

    ```python
   # Translate text
   inputText = ""
   while inputText.lower() != "quit":
        inputText = input("Enter text to translate ('quit' to exit):")
        if inputText != "quit":
            input_text_elements = [InputTextItem(text=inputText)]
            translationResponse = client.translate(body=input_text_elements, to_language=[targetLanguage])
            translation = translationResponse[0] if translationResponse else None
            if translation:
                sourceLanguage = translation.detected_language
                for translated_text in translation.translations:
                    print(f"'{inputText}' was translated from {sourceLanguage.language} to {translated_text.to} as '{translated_text.text}'.")
    ```

1. Guarde los cambios (CTRL+S) y, a continuación, escriba el siguiente comando para ejecutar el programa (maximice el panel de Cloud Shell y cambie el tamaño de los paneles para ver más texto en el panel de línea de comandos):

    ```
   python translate.py
    ```

1. Cuando se le solicite, escriba un idioma de destino válido en la lista que se muestra.
1. Escriba una frase para traducir (por ejemplo, `This is a test` , o `C'est un test`) y vea los resultados, que deben detectar el idioma de origen y traducir el texto al idioma de destino.
1. Cuando haya terminado, escriba `quit`. Puede volver a ejecutar la aplicación y elegir otro idioma de destino.

## Limpieza de recursos

Si ha terminado de explorar el servicio Azure AI Translator, puede eliminar los recursos que creó en este ejercicio. A continuación, se indica cómo puede hacerlo.

1. Cierre del panel de Azure Cloud Shell
1. En Azure Portal, vaya al recurso Traductor de Azure AI que creó en este laboratorio.
1. En la página del recurso, seleccione **Eliminar** y siga las instrucciones para eliminar el recurso.

## Más información

Para más información sobre el servicio de **Traductor de Azure AI**, consulta la [documentación de Traductor de Azure AI](https://learn.microsoft.com/azure/ai-services/translator/).
