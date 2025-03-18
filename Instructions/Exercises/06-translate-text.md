---
lab:
  title: Traducción de texto
  module: Module 3 - Getting Started with Natural Language Processing
---
{% assign site.title = page.lab.title %}

# Traducción de texto

**Traductor de Azure AI** es un servicio que permite traducir texto entre idiomas. En este ejercicio, lo usará para crear una aplicación sencilla que traduzca la entrada en cualquier idioma admitido al idioma de destino que prefiera.

## Aprovisionar un recurso del *Traductor de Azure AI*

Si aún no tiene uno en su suscripción, deberá aprovisionar un recurso del **Traductor de Azure AI**.

1. Inicie sesión en Azure Portal en `https://portal.azure.com` y regístrese con la cuenta de Microsoft asociada a su suscripción de Azure.
1. En el campo de búsqueda de la parte superior, busque **Servicios de Azure AI** y presione **Entrar** y, a continuación, seleccione **Crear** en **Traductor** en los resultados.
1. Cree un recurso con los valores siguientes:
    - **Suscripción**: *suscripción de Azure*
    - **Grupo de recursos**: *elija o cree un grupo de recursos*
    - **Región**: *elija cualquier región disponible*
    - **Nombre**: *escriba un nombre único*
    - **Plan de tarifa**: seleccione **F0** (*gratis*), o **S** (*estándar*) si F no está disponible.
    - **Aviso de IA responsable**: Aceptar.
1. Seleccione **Revisar y crear** y **Crear** para aprovisionar el recurso.
1. Espere a que se complete la implementación y, a continuación, vaya al recurso implementado.
1. Consulte la página **Claves y punto de conexión**. Necesitará la información de esta página más adelante en el ejercicio.

## Preparación para desarrollar una aplicación en Visual Studio Code

Desarrollará la aplicación de traducción de texto mediante Visual Studio Code. Los archivos de código de la aplicación se han proporcionado en un repositorio de GitHub.

> **Sugerencia**: Si ya ha clonado el repositorio **mslearn-ai-language**, ábralo en Visual Studio Code. De lo contrario, siga estos pasos para clonarlo en el entorno de desarrollo.

1. Inicie Visual Studio Code.
2. Abra la paleta (Mayús + Ctrl + P) y ejecute un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/mslearn-ai-language` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abra la carpeta en Visual Studio Code.

    > **Nota**: Si Visual Studio Code muestra un mensaje emergente para solicitarle que confíe en el código que está abriendo, haga clic en la opción **Sí, confío en los autores** en el elemento emergente.

4. Espere mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: Si se le pide que agregue los recursos necesarios para compilar y depurar, seleccione **Ahora no**.

## Configuración de la aplicación

Se han proporcionado aplicaciones para C# y Python. Las dos aplicaciones tienen la misma funcionalidad. Primero, completará algunas partes clave de la aplicación para habilitar que utilice el recurso del Traductor de Azure AI.

1. En Visual Studio Code, en el panel **Explorador**, vaya a la carpeta **Labfiles/06b-translator-sdk** y expanda la carpeta **CSharp** o **Python** según sus preferencias de lenguaje y la carpeta **translate-text** que contiene. Cada carpeta contiene los archivos de código específicos del lenguaje de una aplicación en la que va a integrar la funcionalidad del Traductor de Azure AI.
2. Haga clic con el botón derecho en la carpeta **translate-text** que contiene los archivos de código y abra un terminal integrado. A continuación, instale el paquete del SDK del Traductor de Azure AI mediante la ejecución del comando adecuado para sus preferencias de lenguaje:

    **C#**:

    ```
    dotnet add package Azure.AI.Translation.Text --version 1.0.0-beta.1
    ```

    **Python**:

    ```
    pip install azure-ai-translation-text==1.0.0b1
    ```

3. En el panel **Explorador**, en la carpeta **translate-text**, abra el archivo de configuración para su idioma preferido.

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Actualice los valores de configuración para incluir la **región** y una **clave** del recurso del Traductor de Azure AI que creó (disponible en la página **Claves y punto de conexión** del recurso del Traductor de Azure AI en Azure Portal).

    > **NOTA**: Asegúrese de agregar la *región* del recurso, <u>no</u> el punto de conexión.

5. Guarde el archivo de configuración.

## Agregar código para traducir texto

Ahora está listo para usar el Traductor de Azure AI para traducir texto.

1. Tenga en cuenta que la carpeta **translate-text** contiene un archivo de código para la aplicación cliente:

    - **C#**: Program.cs
    - **Python**: translate.py

    Abra el archivo de código y, en la parte superior, en las referencias de espacio de nombres existentes, busque el comentario **Importar espacios de nombres**. A continuación, en este comentario, agregue el siguiente código específico del lenguaje para importar los espacios de nombres que necesitará para usar el SDK de Text Analytics:

    **C#**: Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.Translation.Text;
    ```

    **Python**: translate.py

    ```python
    # import namespaces
    from azure.ai.translation.text import *
    from azure.ai.translation.text.models import InputTextItem
    ```

1. En la función **Main (Principal)**, observe que el código existente lee los ajustes de configuración.
1. Busque el comentario **Crear cliente mediante el punto de conexión y la clave** y agregue el siguiente código:

    **C#**: Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credential = new(translatorKey);
    TextTranslationClient client = new(credential, translatorRegion);
    ```

    **Python**: translate.py

    ```python
    # Create client using endpoint and key
    credential = TranslatorCredential(translatorKey, translatorRegion)
    client = TextTranslationClient(credential)
    ```

1. Busque el comentario **Elija idioma de destino** y agregue el siguiente código, que usa el servicio Traductor de texto para devolver la lista de idiomas admitidos para la traducción y solicita al usuario que seleccione un código de idioma para el idioma de destino.

    **C#**: Programs.cs

    ```csharp
    // Choose target language
    Response<GetLanguagesResult> languagesResponse = await client.GetLanguagesAsync(scope:"translation").ConfigureAwait(false);
    GetLanguagesResult languages = languagesResponse.Value;
    Console.WriteLine($"{languages.Translation.Count} languages available.\n(See https://learn.microsoft.com/azure/ai-services/translator/language-support#translation)");
    Console.WriteLine("Enter a target language code for translation (for example, 'en'):");
    string targetLanguage = "xx";
    bool languageSupported = false;
    while (!languageSupported)
    {
        targetLanguage = Console.ReadLine();
        if (languages.Translation.ContainsKey(targetLanguage))
        {
            languageSupported = true;
        }
        else
        {
            Console.WriteLine($"{targetLanguage} is not a supported language.");
        }

    }
    ```

    **Python**: translate.py

    ```python
    # Choose target language
    languagesResponse = client.get_languages(scope="translation")
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

1. Busque el comentario **Traducir texto** y agregue el siguiente código, que solicita repetidamente al usuario que el texto se traduzca, usa el servicio de Traductor de Azure AI para traducirlo al idioma de destino (detecta el idioma de origen automáticamente) y muestra los resultados hasta que el usuario escriba *salir*.

    **C#**: Programs.cs

    ```csharp
    // Translate text
    string inputText = "";
    while (inputText.ToLower() != "quit")
    {
        Console.WriteLine("Enter text to translate ('quit' to exit)");
        inputText = Console.ReadLine();
        if (inputText.ToLower() != "quit")
        {
            Response<IReadOnlyList<TranslatedTextItem>> translationResponse = await client.TranslateAsync(targetLanguage, inputText).ConfigureAwait(false);
            IReadOnlyList<TranslatedTextItem> translations = translationResponse.Value;
            TranslatedTextItem translation = translations[0];
            string sourceLanguage = translation?.DetectedLanguage?.Language;
            Console.WriteLine($"'{inputText}' translated from {sourceLanguage} to {translation?.Translations[0].To} as '{translation?.Translations?[0]?.Text}'.");
        }
    } 
    ```

    **Python**: translate.py

    ```python
    # Translate text
    inputText = ""
    while inputText.lower() != "quit":
        inputText = input("Enter text to translate ('quit' to exit):")
        if inputText != "quit":
            input_text_elements = [InputTextItem(text=inputText)]
            translationResponse = client.translate(content=input_text_elements, to=[targetLanguage])
            translation = translationResponse[0] if translationResponse else None
            if translation:
                sourceLanguage = translation.detected_language
                for translated_text in translation.translations:
                    print(f"'{inputText}' was translated from {sourceLanguage.language} to {translated_text.to} as '{translated_text.text}'.")
    ```

1. Guarde los cambios en el archivo de código.

## Prueba de la aplicación

La aplicación ya se puede probar.

1. En el terminal integrado de la carpeta **Traducir texto**, escriba el siguiente comando para ejecutar el programa:

    - **C#**: `dotnet run`
    - **Python**: `python translate.py`

    > **Sugerencia**: Puede usar el icono **Maximizar el tamaño del panel **(**^**) en la barra de herramientas del terminal para ver más del texto de la consola.

1. Cuando se le solicite, escriba un idioma de destino válido en la lista que se muestra.
1. Escriba una frase para traducir (por ejemplo, `This is a test` , o `C'est un test`) y vea los resultados, que deben detectar el idioma de origen y traducir el texto al idioma de destino.
1. Cuando haya terminado, escriba `quit`. Puede volver a ejecutar la aplicación y elegir otro idioma de destino.

## Limpiar

Cuando ya no necesite el proyecto, puede eliminar el recurso del Traductor de Azure AI en el [Azure Portal](https://portal.azure.com).
