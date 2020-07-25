# Translation API Test Drive


## Objective

IBM Watson™ Language Translator allows you to translate text programmatically from one language into another language.

You call `Translator` API to translate sample text in this section.


## Tools Used

- Watson Language Translator


## Requirements

- [IBM Cloud Account](https://cloud.ibm.com)


## Steps

To test drive `Translator` API,

1. Go to the terminal window that you have configured in the previous section.

1. The terminal window should have been ready for making API calls. If not, execute command

    ```
    export apikey=<your API key>
    export url=<your url>
    ```

1.  Translate `Hello, world!` to Spanish by calling `Translator` API.

    ```
    curl -X POST -u "apikey:$apikey" --header "Content-Type: application/json" --data "{\"text\": [\"Hello, world! \", \"How are you?\"], \"model_id\":\"en-es\"}" "$url/v3/translate?version=2018-05-01"
    ```

    The command uses `apikey` for API access credential. It returns the translation result in JSON format. It uses `en-es` translation model to translate `Hello, World!` and `How are you?` to Spanish.

1. It returns

    ```
    {
        "translations" : [ {
            "translation" : "¡Hola, mundo! "
        }, {
            "translation" : "¿Cómo estás?"
        } ],
        "word_count" : 5,
        "character_count" : 26
    }
    ```

1.  Translate `Hello, world!` to Chinese by calling `Translator` API.

    ```
    curl -X POST -u "apikey:$apikey" --header "Content-Type: application/json" --data "{\"text\": [\"Hello, world! \", \"How are you?\"], \"model_id\":\"en-zh\"}" "$url/v3/translate?version=2018-05-01"

    The command uses `apikey` for API access credential. It returns the translation result in JSON format. It uses `en-zh` translation model to translate `Hello, World!` and `How are you?` to Chinese.

1. It returns

    ```
    {
        "translations" : [ {
            "translation" : "你好世界 "
        }, {
            "translation" : "你怎么样"
        } ],
        "word_count" : 5,
        "character_count" : 26
    }
    ```

1. Optionally, translate `Hello, world!` to your prefered lanaguage. To retrieve a list of `Translator model_id`,

    ```
    curl --user apikey:$apikey "$url/v3/models?version=2018-05-01"
    ```

1. Identify language of a text by calling `Translator` API.

    ```
    curl -X POST -u "apikey:$apikey" --header "Content-Type: text/plain" --data "Language Translator translates text from one language to another" "$url/v3/identify?version=2018-05-01"
    ```

    The command uses `apikey` for API access credential. It returns the translation result in `PLAIN TEXT` format. It identifies the language of text `Language Translator translates text from one language to another`.

1. It returns a list of possible languages with different confidence.

    ```
    {
            "languages" : [ {
            "language" : "en",
            "confidence" : 0.9999979447464287
        },

        ......]
    }
    ```


## Related Links

There is lots of great information, tutorials, articles, etc on the [IBM Developer site](https://developer.ibm.com) as well as broader web. Here are a subset of good examples related to data understanding, visualization and processing:

- [Getting started with Language Translator](https://cloud.ibm.com/docs/language-translator?topic=language-translator-gettingstarted)
- [IBM Cloud API Docs - Language Translator](https://cloud.ibm.com/apidocs/language-translator)


## General Links

- [IBM Developer](https://developer.ibm.com)
