---
title: Testa och [!DNL Asset Compute Service] felsöka anpassade program.
description: Testa och [!DNL Asset Compute Service] felsöka anpassade program.
translation-type: tm+mt
source-git-commit: 54afa44d8d662ee1499a385f504fca073ab6c347
workflow-type: tm+mt
source-wordcount: '788'
ht-degree: 0%

---


# Testa och felsöka ett anpassat program {#test-debug-custom-worker}

## Kör enhetstester för ett anpassat program {#test-custom-worker}

Installera [Docker Desktop](https://www.docker.com/get-started) på datorn. Om du vill testa en anpassad arbetare kör du följande kommando i programmets rot:

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `adobe-asset-compute test-worker` command in the root of the custom application application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

Detta kör ett anpassat ramverk för enhetstestning för programåtgärder för tillgångsberäkning i projektet enligt beskrivningen nedan. Den sammanfogas med en konfiguration i `package.json` filen. Det går också att använda JavaScript-enhetstester som Jest. `aio app test` kör båda.

Plugin-programmet [aio-cli-plugin-program-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) är inbäddat som utvecklingsberoende i det anpassade programmet så att det inte behöver installeras på bygg-/testsystem.

### Ramverk för testning av programenhet {#unit-test-framework}

Med testramverket för applikationsenhet Asset Compute kan du testa applikationer utan att behöva skriva någon kod. Den bygger på principen för källa till återgivning av program. En viss fil- och mappstruktur måste konfigureras för att definiera testfall med testkällfiler, valfria parametrar, förväntade återgivningar och anpassade valideringsskript. Som standard jämförs återgivningarna för bytelikhet. Dessutom kan externa HTTP-tjänster enkelt modelleras med enkla JSON-filer.

### Lägg till tester {#add-tests}

Testerna förväntas i `test` mappen på rotnivån i AIO-projektet. Testfallen för varje program ska finnas i sökvägen `test/asset-compute/<worker-name>`med en mapp för varje testfall:

```yaml
action/
manifest.yml
package.json
...
test/
  asset-compute/
    <worker-name>/
        <testcase1>/
            file.jpg
            params.json
            rendition.png
        <testcase2>/
            file.jpg
            params.json
            rendition.gif
            validate
        <testcase3>/
            file.jpg
            params.json
            rendition.png
            mock-adobe.com.json
            mock-console.adobe.io.json
```

Ta en titt på [exempel på anpassade program](https://github.com/adobe/asset-compute-example-workers/) . Nedan finns en detaljerad referens.

### Testa utdata {#test-output}

Detaljerade testutdata, inklusive loggarna för det anpassade programmet, finns tillgängliga i mappen `build` i roten av appen Firefly, vilket visas i `aio app test` utdata.

### Visa externa tjänster {#mock-external-services}

Det går att skapa dummies för externa serviceanrop i funktionsmakron genom att definiera `mock-<HOST_NAME>.json` filer i testfall, där HOST_NAME är den värd som du vill skapa en dummy för. Ett exempel är ett program som gör ett separat anrop till S3. Den nya teststrukturen skulle se ut så här:

```json
test/
  asset-compute/
    <worker-name>/
      <testcase3>/
        file.jpg
        params.json
        rendition.png
        mock-<HOST_NAME1>.json
        mock-<HOST_NAME2>.json
```

Mock-filen är ett JSON-formaterat http-svar. Mer information finns i [den här dokumentationen](https://www.mock-server.com/mock_server/creating_expectations.html). Om det finns flera värdnamn att sätta samman definierar du flera `mock-<mocked-host>.json` filer. Nedan visas ett exempel på en modellfil för `google.com` namnet `mock-google.com.json`:

```json
[{
    "httpRequest": {
        "path": "/images/hello.txt"
        "method": "GET"
    },
    "httpResponse": {
        "statusCode": 200,
        "body": {
          "message": "hello world!"
        }
    }
}]
```

Exemplet `worker-animal-pictures` innehåller en [exempelfil](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) för den Wikimedia-tjänst som interagerar med den.

#### Dela filer i testfall {#share-files-across-test-cases}

Vi rekommenderar att du använder relativa symboler om du delar `file.*`, `params.json` eller `validate` skript i flera tester. De stöds med Git. Ge de delade filerna ett unikt namn, eftersom du kan ha andra. I exemplet nedan blandas och matchar testerna några delade filer och deras egna:

```json
tests/
    file-one.jpg
    params-resize.json
    params-crop.json
    validate-image-compare

    jpg-png-resize/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-resize.json
        rendition.png
        validate    - symlink: ../validate-image-compare

    jpg2-png-crop/
        file.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate    - symlink: ../validate-image-compare

    jpg-gif-crop/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate
```

### Testa förväntade fel {#test-unexpected-errors}

Feltestfall ska inte innehålla en förväntad `rendition.*` fil och ska definiera den förväntade `errorReason` filen i `params.json` filen.

Struktur för feltest:

```json
<error_test_case>/
    file.jpg
    params.json
```

Parameterfil med felorsak:

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

Se fullständig lista och beskrivning av orsaker till [tillgångsberäkning](https://github.com/adobe/asset-compute-commons#error-reasons).

## Felsöka ett anpassat program {#debug-custom-worker}

Följande steg visar hur du kan felsöka ditt anpassade program med Visual Studio Code. Det gör det möjligt att se liveloggar, träffbrytpunkter och stega igenom kod samt ladda om lokala kodändringar live vid varje aktivering.

Många av dessa steg automatiseras vanligtvis `aio` som de ska, se avsnittet Felsöka programmet i dokumentationen [för](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md)felsökning. För tillfället innehåller stegen nedan en lösning.

1. Installera den senaste [wskdebug](https://github.com/apache/openwhisk-wskdebug) från GitHub och den valfria [ngrok](https://www.npmjs.com/package/ngrok).

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Lägg till i JSON-filen för användarinställningar. Den nya felsökningsfunktionen för VS-kod används hela tiden. Den nya felsökningsfunktionen har [vissa problem](https://github.com/apache/openwhisk-wskdebug/issues/74) med wskdebug: `"debug.javascript.usePreview": false`.
1. Stäng alla instanser av program som är öppna via `aio app run`.
1. Distribuera den senaste koden med `aio app deploy`.
1. Kör bara verktyget Resursberäkning med `npx adobe-asset-compute devtool`. Håll den öppen.
1. I VS-kodredigeraren lägger du till nedanstående felsökningskonfiguration i din `launch.json`:

   ```json
   {
     "type": "node",
     "request": "launch",
     "name": "wskdebug worker",
     "runtimeExecutable": "wskdebug",
     "args": [
       "PROJECT-0.0.1/__secured_worker",           // Replace this with your ACTION NAME
       "${workspaceFolder}/actions/worker/index.js",
       "-l",
       "--ngrok"
     ],
     "localRoot": "${workspaceFolder}",
     "remoteRoot": "/code",
     "outputCapture": "std",
     "timeout": 30000
   }
   ```

   Hämta ÅTGÄRDENS NAMN från utdata från `aio app deploy`. Det ser ut som `Your deployed actions -> TypicalCoffeeCat-0.0.1/__secured_worker`.

1. Välj `wskdebug worker` i körnings-/felsökningskonfigurationen och tryck på uppspelningsikonen. Vänta tills den visas **[!UICONTROL Klar för aktivering]** i **[!UICONTROL felsökningskonsolen]** .

1. Klicka på **[!UICONTROL Kör]** i utvecklingsverktyget. Du kan se åtgärderna som körs i VS-kodredigeraren och loggarna visas.

1. Ange en brytpunkt i koden, kör igen och det ska tryckas.

Alla kodändringar läses in i realtid och träder i kraft så snart nästa aktivering sker.

>[!NOTE]
>
>Det finns två aktiveringar för varje begäran i anpassade program. Den första begäran är en webbåtgärd som anropar sig själv asynkront i SDK-koden. Den andra aktiveringen är den som slår koden.