# Build Parallel Corpus Custom Model


## Overview

Most of the provided translation models in `Language Translator` can be extended to learn custom terms and phrases or a general style that's derived from your translation data.

The `Language Translator` service supports two types of customization requests. You can either customize a model with a forced glossary or with a corpus that contains parallel sentences.

- Use a `forced glossary` to force certain terms and phrases to be translated in a specific way.
- Use a `parallel corpus` when you want your custom model to learn from general translation patterns in your samples. What your model learns from a parallel corpus can improve translation results for input text that the model hasn't been trained on.

The general improvements from parallel corpus customization are less predictable than the mandatory results you get from forced glossary customization. 

Use a parallel corpus to provide additional translations for the base model to learn from. This helps to adapt the base model to a specific domain. How the resulting custom model translates text depends on the model's combined understanding of the parallel corpus and the base model.
- Training data format: TMX (UTF-8 encoded)
- Maximum length of translation pairs: 80 source words
- Minimum number of translation pairs: 5,000
- Maximum file size: 250 MB
- You can submit multiple parallel corpus files by repeating the parallel_corpus multipart form parameter as long as the cumulative size of the files doesn't exceed 250 MB.

> Note, Make sure that your Language Translator service instance is on an Advanced or Premium pricing plan. The Lite and Standard plans do not support customization.


## Objective

IBM Watson™ Language Translator allows you to translate text programmatically from one language into another language.

You build a `parallel corpus` custom model in this section.


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

The following is a small piece of an English to French `parallel corpus` that was downloaded from the MultiUN collection available on the OPUS open parallel corpus website. You can download a compressed version of the entire TMX file.

```
<?xml version="1.0" encoding="UTF-8" ?>
<tmx version="1.4">
<header creationdate="Tue Jan 29 12:49:40 2013"
          srclang="en"
          adminlang="en"
          o-tmf="unknown"
          segtype="sentence"
          creationtool="Uplug"
          creationtoolversion="unknown"
          datatype="PlainText" />
  <body>
    <tu>
      <tuv xml:lang="en"><seg>RESOLUTION 55/100</seg></tuv>
      <tuv xml:lang="fr"><seg>RÉSOLUTION 55/100</seg></tuv>
    </tu>
    <tu>
      <tuv xml:lang="en"><seg>Adopted at the 81st plenary meeting, on 4 December 2000, on the recommendation of the Committee (A/55/602/Add.2 and Corr.1, para. 94),The draft resolution recommended in the report was sponsored in the Committee by: Bolivia, Cuba, El Salvador, Ghana and Honduras. by a recorded vote of 106 to 1, with 67 abstentions, as follows:</seg></tuv>
      <tuv xml:lang="fr"><seg>Adoptée à la 81e séance plénière, le 4 décembre 2000, sur la recommandation de la Commission (A/55/602/Add.2, par. 94)Le projet de résolution recommandé dans le rapport de la Commission avait pour auteurs les pays suivants: Bolivie, Cuba, El Salvador, Ghana et Honduras., par 106 voix contre une, avec 67 abstentions, à la suite d'un vote enregistré, les voix s'étant réparties comme suit:</seg></tuv>
    </tu>
    ...
```


## Steps

To build a `parallel corpus` custom model,

1. Go to the terminal window that you have configured in the previous section.

1. The terminal window should have been ready for making API calls. If not, execute command

    ```
    export apikey=<your API key>
    export url=<your url>
    ```

1. Identify if a specific model, for example `en-fr`, supports customization, execute command

    ```
    curl --user "apikey:$apikey" "$url/v3/models?source=en&target=fr&version=2018-05-01"
    ```

1. It returns the following JSON data. `"customizable" : true` shows that the model supports customization.

    ```
    {
        "models" : [ {
            "model_id" : "en-fr",
            "source" : "en",
            "target" : "fr",
            "base_model_id" : "",
            "domain" : "general",
            "customizable" : true,
            "default_model" : true,
            "owner" : "",
            "status" : "available",
            "name" : "en-fr",
            "training_log" : null
        } ]
    }
    ```

1. Optionally, you may execute the command below and retrieve all models for customization support.

    ```
    curl --user apikey:$apikey "$url/v3/models?version=2018-05-01"
    ```

1. Create your training data. 

    For this exercise, a TMX file `en-fr-6000-ParallelCorpus.tmx` is provided in the repo.

1. Train your custom model. 

    Use the `Create model` method to train your model. In your request, specify the model ID of a customizable base model, and training data in the parallel_corpus parameters.

    ```
    curl -X POST --user "apikey:$apikey" --form parallel_corpus=@en-fr-6000-ParallelCorpus.tmx "$url/v3/models?version=2018-05-01&base_model_id=en-fr&name=custom-english-to-french"
    ```

    You can upload multiple parallel_corpus files in one request. All uploaded parallel_corpus files combined, your parallel corpus must contain at least 5,000 parallel sentences to train successfully.

1. The command returns

    ```
    {
        "model_id" : "43745eda-7fde-4998-a62a-26cf0e795973",
        "source" : "en",
        "target" : "fr",
        "base_model_id" : "en-fr",
        "domain" : "general",
        "customizable" : true,
        "default_model" : false,
        "owner" : "1e5f399b-605d-4e83-b07e-534da85b86a9",
        "status" : "dispatching",
        "name" : "custom-english-to-french",
        "training_log" : null
    }
    ```

    The API response will contain details about your custom model, including its model ID.

1. Record the `model_id`.

    ```
    export MODELID=<model_id>
    ```

1. Check the status of the new custom model.

    Model training might take anywhere from a couple of minutes (for forced glossaries) to several hours (for large parallel corpora) depending on how much training data is involved. To see if your model is available, use the `Get model details` method and specify the model ID that you received in the service response of the previous step. Also, you can check the status of all of your models with the List models method.

    The following command gets information for the model identified by the model ID `$MODELID`.

    ```
    curl --user "apikey:$apikey" "$url/v3/models/$MODELID?version=2018-05-01"
    ```

    It returns

    ```
    {
        "model_id" : "43745eda-7fde-4998-a62a-26cf0e795973",
        "source" : "en",
        "target" : "fr",
        "base_model_id" : "en-fr",
        "domain" : "general",
        "customizable" : true,
        "default_model" : false,
        "owner" : "1e5f399b-605d-4e83-b07e-534da85b86a9",
        "status" : "training",
        "name" : "custom-english-to-french",
        "training_log" : null
    }
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
        "model_id" : "43745eda-7fde-4998-a62a-26cf0e795973",
        "source" : "en",
        "target" : "fr",
        "base_model_id" : "en-fr",
        "domain" : "general",
        "customizable" : true,
        "default_model" : false,
        "owner" : "1e5f399b-605d-4e83-b07e-534da85b86a9",
        "status" : "available",
        "name" : "custom-english-to-french",
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
            "translation" : "Bonjour, Lee Zhang. Veuillez ne pas vous garer dans l'allée."
        } ],
        "word_count" : 13,
        "character_count" : 49
    }
    ```

    Neither `Lee Zhang` nor `alley` were translated in the way that you defined in the TMX file `en-fr-6000-ParallelCorpus.tmx`. Use a `parallel corpus` when you want your custom model to learn from general translation patterns in your sample TMX file. What your model learns from a parallel corpus can improve translation results for input text that the model hasn't been trained on. However, the general improvements from parallel corpus customization are less predictable than the mandatory results you get from forced glossary customization.

1. You can apply a `forced glossary` to a model that has been customized with a parallel corpus. 

    ```
    curl -X POST --user "apikey:$apikey" --form forced_glossary=@en-fr-ForcedGlossary.tmx "$url/v3/models?version=2018-05-01&base_model_id=$MODELID&name=custom-english-to-french-2"
    ```

1. The command returns

    ```
    {
        "model_id" : "6277bffd-f884-4c5c-9825-ac41136f1b69",
        "source" : "en",
        "target" : "fr",
        "base_model_id" : "43745eda-7fde-4998-a62a-26cf0e795973",
        "domain" : "general",
        "customizable" : false,
        "default_model" : false,
        "owner" : "1e5f399b-605d-4e83-b07e-534da85b86a9",
        "status" : "dispatching",
        "name" : "custom-english-to-french-2",
        "training_log" : null
    }
    ```

    The API response will contain details about your custom model, including its model ID.

1. Record the new model ID.

    ```
    export MODELID2=<new model ID>
    ```

1. Check the status of the new custom model.

    The following command gets information for the model identified by the model ID `$MODELID2`. 

    ```
    curl --user "apikey:$apikey" "$url/v3/models/$MODELID2?version=2018-05-01"
    ```

1. When the model status is available, your model is ready to use with your service instance.

    ```
    {
        "model_id" : "6277bffd-f884-4c5c-9825-ac41136f1b69",
        "source" : "en",
        "target" : "fr",
        "base_model_id" : "43745eda-7fde-4998-a62a-26cf0e795973",
        "domain" : "general",
        "customizable" : false,
        "default_model" : false,
        "owner" : "1e5f399b-605d-4e83-b07e-534da85b86a9",
        "status" : "available",
        "name" : "custom-english-to-french-2",
        "training_log" : null
    }
    ```

1. Translate text with your new custom model.

    To use your custom model, specify the text that you want to translate and the custom model's model ID in the Translate method. The following command translates text with the custom model identified by the model ID `MODELID2`.

    ```
    curl -X POST --user "apikey:$apikey" --header "Content-Type: application/json" --data "{\"text\":\"Hello, Lee Zhang. Please don't park in the alley.\",\"model_id\":\"$MODELID2\"}" "$url/v3/translate?version=2018-05-01"
    ```

1. It returns

    ```
    {
        "translations" : [ {
            "translation" : "Bonjour, Lijing Zhang. Veuillez ne pas vous garer dans l'胡同."
        } ],
        "word_count" : 13,
        "character_count" : 49
    }
    ```

    This time, both `Lee Zhang` and `alley` were translated in the way that you defined in the TMX file `en-fr-ForcedGlossary.tmx`.

    > Note: the sentence was translated to `Bonjour, Lee Zhang. Veuillez ne pas vous garer dans l'allée.` last time.

1. Deleting a custom translation model. **Don't run the command if you plan to use the custom model again**.

    To delete a custom translation model, use the Delete model method. The following command deletes the translation model with the model ID `$MODELID2` & `$MODELID`. 

    ```
    curl -X DELETE --user "apikey:$apikey" "$url/v3/models/$MODELID?version=2018-05-01"
    curl -X DELETE --user "apikey:$apikey" "$url/v3/models/$MODELID2?version=2018-05-01"
    ```


## Related Links

There is lots of great information, tutorials, articles, etc on the [IBM Developer site](https://developer.ibm.com) as well as broader web. Here are a subset of good examples related to data understanding, visualization and processing:

- [Getting started with Language Translator](https://cloud.ibm.com/docs/language-translator?topic=language-translator-gettingstarted)
- [IBM Cloud API Docs - Language Translator](https://cloud.ibm.com/apidocs/language-translator)


## General Links

- [IBM Developer](https://developer.ibm.com)
