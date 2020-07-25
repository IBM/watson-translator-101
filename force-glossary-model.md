# Build Force Glossary Custom Model


## Overview

Most of the provided translation models in `Language Translator` can be extended to learn custom terms and phrases or a general style that's derived from your translation data.

The `Language Translator` service supports two types of customization requests. You can either customize a model with a forced glossary or with a corpus that contains parallel sentences.

- Use a `forced glossary` to force certain terms and phrases to be translated in a specific way.
- Use a `parallel corpus` when you want your custom model to learn from general translation patterns in your samples. What your model learns from a parallel corpus can improve translation results for input text that the model hasn't been trained on.

The general improvements from parallel corpus customization are less predictable than the mandatory results you get from forced glossary customization. 

Use a forced glossary to set mandatory translations for specific terms and phrases. If you want specific control over translation behavior, use a forced glossary.
- Training data format: TMX (UTF-8 encoded)
- Maximum file size: 10 MB
- You can apply a forced glossary to a base model, or to a model that has been customized with a parallel corpus
- Limit one forced glossary per model

> Note: Forced glossary examples are sensitive to `capitalization`, so make sure that your training data reflects the capitalization of content that your application will encounter.

> Note, Make sure that your Language Translator service instance is on an Advanced or Premium pricing plan. The Lite and Standard plans do not support customization.


## Objective

IBM Watson™ Language Translator allows you to translate text programmatically from one language into another language.

You build a `forced glossary` custom model in this section.


## Tools Used

- Watson Language Translator


## Requirements

- [IBM Cloud Account](https://cloud.ibm.com)


## Creating TMX file

To provide a glossary or corpus of terms for training the `Language Translator` service, create a valid UTF-8 encoded document that conforms to the Translation Memory Exchange (TMX) version 1.4 specification. TMX is an XML specification that is designed for machine-translation tools.

Each term and translation pair must be enclosed in `<tu>` tags:

```
<tu>
    <tuv xml:lang="en">
    <seg>patent</seg>
    </tuv>

    <tuv xml:lang="fr">
    <seg>brevet</seg>
    </tuv>
</tu>
```

The `xml:lang` attribute in the `<tuv>` tag identifies the language in which a term is expressed, while the `<seg>` tag contains the term or the translation.

The `Language Translator` service uses only the `<tu>`, `<tuv>`, and `<seg>` elements. All other elements in the example are required to make a valid TMX file, or are informational, but are not used by the service.


## Sampe TMX file

Below is a sample TMX file that can be used to create `forced glossary` custom model. It has three translation pairs. The first pair forces "international business machines" to be preserved in English during translation. This type of forced translation might be useful if you want to preserve improperly capitalized company names in informal communications. The second pair forces the model to always use "brevet" when translating "patent". The third pair shows that the `forced glossary` is forceful and you can even include Chinese in your `en-es` model.

```
<?xml version="1.0" encoding="UTF-8"?>
<tmx version="1.4">
  <header creationtool="" creationtoolversion=""
    segtype="phrase" o-tmf="" adminlang="en"
    srclang="en" datatype="PlainText" o-encoding="UTF-8" />
  <body>
    <tu>
      <tuv xml:lang="en">
        <seg>international business machines</seg>
      </tuv>
      <tuv xml:lang="es">
        <seg>International Business Machines</seg>
      </tuv>
    </tu>
    <tu>
      <tuv xml:lang="en">
        <seg>patent</seg>
      </tuv>
      <tuv xml:lang="es">
        <seg>patentar</seg>
      </tuv>
    </tu>
    <tu>
      <tuv xml:lang="en">
        <seg>alley</seg>
      </tuv>
      <tuv xml:lang="es">
        <seg>胡同</seg>
      </tuv>
    </tu>
  </body>
</tmx>
```


## Steps

To build a `forced glossary` custom model,

1. Go to the terminal window that you have configured in the previous section.

1. The terminal window should have been ready for making API calls. If not, execute command

    ```
    export apikey=<your API key>
    export url=<your url>
    ```

1. Identify if a specific model, for example `en-es`, supports customization, execute command

    ```
    curl --user "apikey:$apikey" "$url/v3/models?source=en&target=es&version=2018-05-01"
    ```

1. It returns the following JSON data. `"customizable" : true` shows that the model supports customization.

    ```
    {
        "models" : [ {
            "model_id" : "en-es",
            "source" : "en",
            "target" : "es",
            "base_model_id" : "",
            "domain" : "general",
            "customizable" : true,
            "default_model" : true,
            "owner" : "",
            "status" : "available",
            "name" : "en-es",
            "training_log" : null
        } ]
    }
    ```

1. Optionally, you may execute the command below and retrieve all models and verify their customization support.

    ```
    curl --user apikey:$apikey "$url/v3/models?version=2018-05-01"
    ```

1. Create your training data. 

    For this exercise, a TMX file `en-es-ForcedGlossary.tmx` is provided in the repo.

1. Train your custom model. 

    Use the `Create model` method to train your model. In your request, specify the model ID of a customizable base model, and training data in either the forced_glossary or parallel_corpus parameters.

    ```
    curl -X POST --user "apikey:$apikey" --form forced_glossary=@en-es-ForcedGlossary.tmx "$url/v3/models?version=2018-05-01&base_model_id=en-es&name=custom-english-to-spanish"
    ```

    The customizations in the file completely overwrite the domain translaton data, including high frequency or high confidence phrase translations. You can upload only one glossary with a file size less than 10 MB per call. A forced glossary should contain single words or short phrases.

1. The command returns

    ```
    {
        "model_id" : "a6c701aa-9c3e-4d08-ad7d-f8113e501608",
        "source" : "en",
        "target" : "es",
        "base_model_id" : "en-es",
        "domain" : "general",
        "customizable" : false,
        "default_model" : false,
        "owner" : "1e5f399b-605d-4e83-b07e-534da85b86a9",
        "status" : "dispatching",
        "name" : "custom-english-to-spanish",
        "training_log" : null
    }
    ```

    The API response will contain details about your custom model, including its `model_id`.

1. Record the `model_id`.

    ```
    export MODELID=<model_id>
    ```

1. Check the status of the new custom model.

    Model training might take anywhere from a couple of minutes (for forced glossaries) to several hours (for large parallel corpora) depending on how much training data is involved. To check if your model is available, use the `Get model details` method and specify the model ID that you received in the service response of the previous step. Also, you can check the status of all of your models with the List models method.

    The following command gets information for the model identified by the model ID `$MODELID`.

    ```
    curl --user "apikey:$apikey" "$url/v3/models/$MODELID?version=2018-05-01"
    ```

    The status response attribute describes the state of the model in the training process:
    - uploading
    - uploaded
    - dispatching
    - queued
    - training
    - trained
    - publishing
    - available

1. When the model status is available, your model is ready to use with your service instance.

    ```
    {
        "model_id" : "a6c701aa-9c3e-4d08-ad7d-f8113e501608",
        "source" : "en",
        "target" : "es",
        "base_model_id" : "en-es",
        "domain" : "general",
        "customizable" : false,
        "default_model" : false,
        "owner" : "1e5f399b-605d-4e83-b07e-534da85b86a9",
        "status" : "available",
        "name" : "custom-english-to-spanish",
        "training_log" : null
    }
    ```

1. Translate text with your new custom model.

    To use your custom model, specify the text that you want to translate and the custom model's model ID in the Translate method.

    The following command translates text with the custom model identified by the model ID `$MODELID`.

    ```
    curl -X POST --user "apikey:$apikey" --header "Content-Type: application/json" --data "{\"text\":\"Hello, Lee Zhang. Please don't park in the alley.\",\"model_id\":\"$MODELID\"}" "$url/v3/translate?version=2018-05-01"
    ```

1. It returns

    ```
    {
        "translations" : [ {
            "translation" : "Hola, Lijing Zhang. Por favor, no estacione en el 胡同."
        } ],
        "word_count" : 13,
        "character_count" : 49
    }
    ```

    Both `Lee Zhang` and `alley` were translated in the way that you defined in the TMX file.

1. Translate text with the base model.

    ```
    curl -X POST --user "apikey:$apikey" --header "Content-Type: application/json" --data "{\"text\":\"Hello, Lee Zhang. Please don't park in the alley.\",\"model_id\":\"en-es\"}" "$url/v3/translate?version=2018-05-01"
    ```

1. It returns

    ```
    {
        "translations" : [ {
            "translation" : "Hola, Lee Zhang. Por favor, no estacione en el callejón."
        } ],
        "word_count" : 13,
        "character_count" : 49
    }
    ```

1. Compare the translation result of the custom model and the base model. The term `alley` and `Lee Zhang` were translated as we expected.

1. Deleting a custom translation model. **Don't run the command if you plan to use the custom model again**.

    To delete a custom translation model, use the Delete model method.

    The following command deletes the translation model with model ID `$MODELID`.

    ```
    $ curl -X DELETE --user "apikey:$apikey" "$url/v3/models/$MODELID?version=2018-05-01"
    ```


## Related Links

There is lots of great information, tutorials, articles, etc on the [IBM Developer site](https://developer.ibm.com) as well as broader web. Here are a subset of good examples related to data understanding, visualization and processing:

- [Getting started with Language Translator](https://cloud.ibm.com/docs/language-translator?topic=language-translator-gettingstarted)
- [IBM Cloud API Docs - Language Translator](https://cloud.ibm.com/apidocs/language-translator)


## General Links

- [IBM Developer](https://developer.ibm.com)
