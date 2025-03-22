---
lab:
  title: Traducción de texto
  module: Module 3 - Getting Started with Natural Language Processing
---
{% assign site.title = page.lab.title %}

# Traducción de texto

**Traductor de Azure AI** es un servicio que permite traducir texto entre idiomas. En este ejercicio, lo usarás para crear una aplicación sencilla que traduzca la entrada en cualquier idioma admitido al idioma de destino que prefiera.

## Aprovisionar un recurso del *Traductor de Azure AI*

Si aún no tienes uno en tu suscripción, deberás aprovisionar un recurso del **Traductor de Azure AI**.

1. Inicia sesión en Azure Portal en `https://portal.azure.com` y regístrate con la cuenta de Microsoft asociada a tu suscripción a Azure.
1. En el campo de búsqueda de la parte superior, busca **Servicios de Azure AI** y presiona **Entrar** y, a continuación, selecciona **Crear** en **Traductor** en los resultados.
1. Crea un recurso con los valores siguientes:
    - **Suscripción**: *suscripción a Azure*
    - **Grupo de recursos**: *elige o crea un grupo de recursos*
    - **Región**: *elige cualquier región disponible*
    - **Nombre**: *escribe un nombre único*
    - **Plan de tarifa**: selecciona **F0** (*gratis*), o **S** (*estándar*) si F no está disponible.
    - **Aviso de IA responsable**: Aceptar.
1. Selecciona **Revisar y crear** y **Crear** para aprovisionar el recurso.
1. Espera a que se complete la implementación y, a continuación, ve al recurso implementado.
1. Consulta la página **Claves y punto de conexión**. Necesitarás la información de esta página más adelante en el ejercicio.

## Preparación para desarrollar una aplicación en Visual Studio Code

Desarrollarás la aplicación de traducción de texto mediante Visual Studio Code. Los archivos de código de la aplicación se han proporcionado en un repositorio de GitHub.

> **Sugerencia**: si ya has clonado el repositorio **mslearn-ai-language**, ábrelo en Visual Studio Code. De lo contrario, sigue estos pasos para clonarlo en el entorno de desarrollo.

1. Inicia Visual Studio Code.
2. Abre la paleta (Mayús + Ctrl + P) y ejecuta un comando **Git: Clone** para clonar el repositorio `https://github.com/MicrosoftLearning/mslearn-ai-language` en una carpeta local (no importa qué carpeta).
3. Cuando se haya clonado el repositorio, abre la carpeta en Visual Studio Code.

    > **Nota**: si Visual Studio Code muestra un mensaje emergente para solicitarte que confíes en el código que estás abriendo, haz clic en la opción **Yes, I trust the authors** en el elemento emergente.

4. Espera mientras se instalan archivos adicionales para admitir los proyectos de código de C# en el repositorio.

    > **Nota**: si se te pide que agregues los recursos necesarios para compilar y depurar, selecciona **Ahora no**.

## Configuración de la aplicación

Se han proporcionado aplicaciones para C# y Python. Las dos aplicaciones tienen la misma funcionalidad. Primero, completarás algunas partes clave de la aplicación para habilitar que utilice el recurso del Traductor de Azure AI.

1. En Visual Studio Code, en el panel **Explorador**, ve a la carpeta **Labfiles/06b-translator-sdk** y expande la carpeta **CSharp** o **Python** según tu preferencia de idioma y la carpeta **translate-text** que contiene. Cada carpeta contiene los archivos de código específicos del idioma de una aplicación en la que vas a integrar la funcionalidad del Traductor de Azure AI.
2. Haz clic con el botón derecho en la carpeta **translate-text** que contiene los archivos de código y abre un terminal integrado. A continuación, instala el paquete del SDK del Traductor de Azure AI mediante la ejecución del comando adecuado para tu preferencia de idioma:

    **C#**:

    ```
    dotnet add package Azure.AI.Translation.Text --version 1.0.0-beta.1
    ```

    **Python**:

    ```
    pip install azure-ai-translation-text==1.0.0b1
    ```

3. En el panel **Explorador**, en la carpeta **translate-text**, abre el archivo de configuración para tu idioma preferido.

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Actualiza los valores de configuración para incluir la **región** y una **clave** del recurso del Traductor de Azure AI que creaste (disponible en la página **Claves y punto de conexión** del recurso del Traductor de Azure AI en Azure Portal).

    > **NOTA**: asegúrate de agregar la *región* del recurso, <u>no</u> el punto de conexión.

5. Guarda el archivo de configuración.

## Agregar código para traducir texto

Ahora estás listo para usar el Traductor de Azure AI para traducir texto.

1. Ten en cuenta que la carpeta **translate-text** contiene un archivo de código para la aplicación cliente:

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
