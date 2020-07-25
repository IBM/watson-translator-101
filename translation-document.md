# Translation Doocuments


## Objective

IBM Watson™ Language Translator allows you to translate text programmatically from one language into another language.

Translate files from one language to another while preserving the original format. More than 12 different file formats can be translated, including MS Office, Open Office, and PDF.

Make sure the document you want to translate meets the following `Maximum file size` requirements:
- 20 MB for service instances on Standard, Advanced, and Premium plans
- 2 MB for service instances on the Lite plan


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

1.  Submit a request to translate `sample.pdf` from English to Spanish.

    ```
    curl -X POST --user "apikey:$apikey" --form "file=@sample.pdf" --form "source=en" --form "target=es" "$url/v3/documents?version=2018-05-01"
    ```

    The command uses `apikey` for API access credential. It takes input file, source language and target language as parameters.

1. It returns the following information including `document-id`.

    ```
    {
        "document_id" : "e3b10684-83b0-457b-8c74-0774b41ff9e5",
        "filename" : "sample.pdf",
        "model_id" : "en-es",
        "source" : "en",
        "target" : "es",
        "status" : "processing",
        "created" : "2020-06-22T21:35:12Z"
    }
    ```

1. Record the `document_id`.

    ```
    export DOC_ID=<document_id>
    ```

1. To translate a document with a `custom` model, use the `model_id` parameter instead. For example, the following request translates the document with the custom model identified by the model ID `96221b69-8e46-42e4-a3c1-808e17c787ca` (which is NOT available for the workshop).

    curl -X POST --user "apikey:$apikey" --form "file=@sample.pdf" --form "model_id=96221b69-8e46-42e4-a3c1-808e17c787ca" "$url/v3/documents?version=2018-05-01"

    >Note, you are going to create custom model in the next section.

1. Check the translation status

    After you have submitted a document for translation, you can check the translation status to find out when the translated document is available to download. Replace the document ID `e3b10684-83b0-457b-8c74-0774b41ff9e5` with yours.

    ```
    curl -X GET --user "apikey:$apikey" "$url/v3/documents/$DOC_ID?version=2018-05-01"
    ```

1. The document tramslation of `sample.pdf` should take no time before it becomoes available.

    ```
    {
        "document_id" : "e3b10684-83b0-457b-8c74-0774b41ff9e5",
        "filename" : "sample.pdf",
        "model_id" : "en-es",
        "source" : "en",
        "target" : "es",
        "status" : "available",
        "created" : "2020-06-22T21:35:12Z",
        "completed" : "2020-06-22T21:35:15Z",
        "word_count" : 10,
        "character_count" : 49
    }
    ```

1. Download the translated document. The following example request saves the translated document with document ID `$DOC_ID` to the file `sample-es.pdf`. 

    ```
    curl -X GET --user "apikey:$apikey" --output "sample-es.pdf" "$url/v3/documents/$DOC_ID/translated_document?version=2018-05-01"
    ```

1. Review the translated document `sample-es.pdf`. It is downloaded to the current working folder.

1. Delete documents. To delete the original submission of `sample.pdf` and its Spanish translation, execute the command below. Replace the `e3b10684-83b0-457b-8c74-0774b41ff9e5` with your document ID.

    ```
    curl -X DELETE --user "apikey:$apikey" "$url/v3/documents/$DOC_ID?version=2018-05-01"
    ```

    >Note: Original documents and any associated translated documents are deleted automatically after they have not been used for a certain period of time. 


## Related Links

There is lots of great information, tutorials, articles, etc on the [IBM Developer site](https://developer.ibm.com) as well as broader web. Here are a subset of good examples related to data understanding, visualization and processing:

- [Getting started with Language Translator](https://cloud.ibm.com/docs/language-translator?topic=language-translator-gettingstarted)
- [IBM Cloud API Docs - Language Translator](https://cloud.ibm.com/apidocs/language-translator)


## General Links

- [IBM Developer](https://developer.ibm.com)
