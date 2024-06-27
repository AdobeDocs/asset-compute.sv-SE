---
title: Testa och felsöka [!DNL Asset Compute Service] anpassat program
description: Testa och felsöka [!DNL Asset Compute Service] anpassat program.
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '775'
ht-degree: 0%

---

# Testa och felsöka ett anpassat program {#test-debug-custom-worker}

## Kör enhetstester för ett anpassat program {#test-custom-worker}

Installera [Docker Desktop](https://www.docker.com/get-started) på din dator. Om du vill testa en anpassad arbetare kör du följande kommando i programmets rot:

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

Det här kommandot kör ett anpassat enhetstestramverk för programåtgärder i Asset compute i projektet enligt beskrivningen nedan. Den är ansluten via en konfiguration i `package.json` -fil. Det går också att använda JavaScript-enhetstester som Jest. The `aio app test` kör båda.

The [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) plugin-programmet är inbäddat som ett utvecklingsberoende i det anpassade programmet, så att det inte behöver installeras på build/test-system.

### Ramverk för testning av programenhet {#unit-test-framework}

Med testmiljön i Asset Compute kan du testa program utan att skriva någon kod. Den bygger på principen för källa till återgivning av program. En viss fil- och mappstruktur måste ställas in för att definiera testfall med testkällfiler, valfria parametrar, förväntade återgivningar och anpassade valideringsskript. Som standard jämförs återgivningarna för bytelikhet. Dessutom kan externa HTTP-tjänster enkelt modelleras med enkla JSON-filer.

### Lägg till tester {#add-tests}

Tester förväntas inne i `test` på projektets rotnivå. Testfallen för varje program ska ligga i sökvägen `test/asset-compute/<worker-name>`, med en mapp för varje testfall:

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

Titta på [exempel på anpassade program](https://github.com/adobe/asset-compute-example-workers/) för några exempel. Nedan finns en detaljerad referens.

### Testa utdata {#test-output}

The `build` i Adobe Developer App Builder-programmets rot innehåller detaljerade testresultat och loggar för det anpassade programmet. De här detaljerna visas även i utdata från `aio app test` -kommando.

### Visa externa tjänster {#mock-external-services}

Du kan simulera externa servicesamtal i funktionsmakron genom att skapa `mock-<HOST_NAME>.json` filer för dina testscenarier, där HOST_NAME är den specifika värd som du tänker imitera. Ett exempel på användningsfall är ett program som gör ett separat anrop till S3. Den nya teststrukturen skulle se ut så här:

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

Mock-filen är ett JSON-formaterat http-svar. Mer information finns i [den här dokumentationen](https://www.mock-server.com/mock_server/creating_expectations.html). Om det finns flera värdnamn att matcha definierar du flera `mock-<mocked-host>.json` filer. Nedan visas ett exempel på en exempelfil för `google.com` namngiven `mock-google.com.json`:

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

Exemplet `worker-animal-pictures` innehåller en [dummy-fil](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) för den Wikimedia-tjänst den interagerar med.

#### Dela filer i testfall {#share-files-across-test-cases}

Adobe rekommenderar att du använder relativa symboler om du delar dem `file.*`, `params.json` eller `validate` skript i flera tester. De stöds av Git. Ge de delade filerna ett unikt namn, eftersom du kan ha andra. I exemplet nedan blandas och matchar testerna några delade filer och deras egna:

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

Feltestfall får inte innehålla förväntat `rendition.*` och bör definiera den förväntade `errorReason` innanför `params.json` -fil.

>[!NOTE]
>
>Om ett testfall inte innehåller ett förväntat `rendition.*` och definierar inte den förväntade `errorReason` innanför `params.json` -filen, antas vara ett felfall med `errorReason`.

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

Se en komplett lista och en beskrivning av [Felorsaker i asset compute](https://github.com/adobe/asset-compute-commons#error-reasons).

## Felsöka ett anpassat program {#debug-custom-worker}

Följande steg visar hur du kan felsöka ditt anpassade program med Visual Studio Code. Det gör det möjligt att se liveloggar, träffbrytpunkter och stega igenom kod samt ladda om lokala kodändringar live vid varje aktivering.

The `aio` körklar automatiserar många av dessa steg. Gå till avsnittet Felsöka programmet i [Adobe Developer App Builder-dokumentation](https://developer.adobe.com/app-builder/docs/getting_started/first_app). För tillfället innehåller stegen nedan en lösning.

1. Installera den senaste [wskdebug](https://github.com/apache/openwhisk-wskdebug) från GitHub och [ngrok](https://www.npmjs.com/package/ngrok).

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Gör tillägg till dina användarinställningar i JSON-filen. Den gamla Visual Studio Code debugger används hela tiden. Den nya har [vissa problem](https://github.com/apache/openwhisk-wskdebug/issues/74) med wskdebug: `"debug.javascript.usePreview": false`.
1. Stäng alla instanser av program som är öppna via `aio app run`.
1. Distribuera den senaste koden med `aio app deploy`.
1. Kör endast utvecklingsverktyget Asset compute med `aio asset-compute devtool`. Håll den öppen.
1. I Visual Studio Code Editor lägger du till följande felsökningskonfiguration i `launch.json`:

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

   Hämta `ACTION NAME` från utdata från `aio app deploy`.

1. Välj `wskdebug worker` i körnings-/felsökningskonfigurationen och tryck på uppspelningsikonen. Vänta tills den visas **[!UICONTROL Klar för aktivering]** i **[!UICONTROL Felsökningskonsolen]** -fönstret.

1. Klicka **[!UICONTROL run]** i utvecklingsverktyget. Du kan se vilka åtgärder som körs i Visual Studio-kodredigeraren och att loggarna börjar visas.

1. Ange en brytpunkt i koden. Kör igen och det borde slå.

Alla kodändringar läses in i realtid och träder i kraft så snart nästa aktivering sker.

>[!NOTE]
>
>Det finns två aktiveringar för varje begäran i anpassade program. Den första begäran är en webbåtgärd som anropar sig själv asynkront i SDK-koden. Den andra aktiveringen är den som slår koden.
