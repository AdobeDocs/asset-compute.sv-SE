---
title: Förstå hur ett anpassat program fungerar
description: Internt arbete i  [!DNL Asset Compute Service] anpassat program för att förstå hur det fungerar.
exl-id: a3ee6549-9411-4839-9eff-62947d8f0e42
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '691'
ht-degree: 0%

---

# Internt i ett anpassat program {#how-custom-application-works}

Använd följande bild för att förstå hela arbetsflödet när en digital resurs bearbetas med ett anpassat program av en klient.

![Anpassat programarbetsflöde](assets/customworker.svg)

*Bild: Steg som används vid bearbetning av en resurs med Adobe [!DNL Asset Compute Service].*

## Registrering {#registration}

Klienten måste anropa [`/register`](api.md#register) en gång före den första begäran till [`/process`](api.md#process-request) så att den kan ställa in och hämta journal-URL:en för att ta emot Adobe [!DNL I/O Events] -händelser för Adobe Asset Compute.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

JavaScript-biblioteket [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) kan användas i NodeJS-program för att hantera alla nödvändiga steg från registrering, bearbetning till asynkron händelsehantering. Mer information om obligatoriska rubriker finns i [Autentisering och auktorisering](api.md).

## Bearbetar {#processing}

Klienten skickar en [bearbetningsbegäran](api.md#process-request).

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

Klienten ansvarar för att formatera återgivningarna korrekt med försignerade URL:er. JavaScript-biblioteket [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) kan användas i NodeJS-program för att försignera URL:er. För närvarande stöder biblioteket endast Azure Blob Storage och AWS S3 Containers.

Bearbetningsbegäran returnerar en `requestId` som kan användas för avsökning av [!DNL Adobe I/O]-händelser.

Nedan följer ett exempel på en anpassad begäran om programbearbetning.

```json
{
    "source": "https://www.adobe.com/some-source-file.jpg",
    "renditions" : [
        {
            "worker": "https://my-project-namespace.adobeioruntime.net/api/v1/web/my-namespace-version/my-worker",
            "name": "rendition1.jpg",
            "target": "https://some-presigned-put-url-for-rendition1.jpg",
        }
    ],
    "userData": {
        "my-asset-id": "1234567890"
    }
}
```

[!DNL Asset Compute Service] skickar begäranden om anpassade programåtergivningar till det anpassade programmet. Den använder en HTTP POST till den angivna program-URL:en, som är den skyddade webbåtgärds-URL:en från App Builder. Alla förfrågningar använder HTTPS-protokollet för att maximera datasäkerheten.

[Asset Compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) som används av ett anpassat program hanterar HTTP POST-begäran. Det hanterar också hämtning av källan, överföring av återgivningar, sändning av Adobe [!DNL I/O Events] och felhantering.

<!-- TBD: Add the application diagram. -->

### Programkod {#application-code}

Anpassad kod behöver bara tillhandahålla ett återanrop som tar den lokalt tillgängliga källfilen (`source.path`). `rendition.path` är platsen där det slutliga resultatet av en resursbearbetningsbegäran ska placeras. Det anpassade programmet använder återanropet för att omvandla de lokalt tillgängliga källfilerna till en återgivningsfil med det namn som skickades (`rendition.path`). Ett anpassat program måste skriva till `rendition.path` för att skapa en återgivning:

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

// worker() is the entry point in the SDK "framework".
// The asynchronous function defined is the rendition callback.
exports.main = worker(async (source, rendition) => {

    // Tip: custom worker parameters are available in rendition.instructions.
    console.log(rendition.instructions.name); // should print out `rendition.jpg`.

    // Simplest example: copy the source file to the rendition file destination so as to transfer the asset as is without processing.
    await fs.copyFile(source.path, rendition.path);
});
```

### Hämta källfiler {#download-source}

Ett anpassat program hanterar bara lokala filer. [Asset Compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) hanterar nedladdningen av källfilen.

### Skapa återgivning {#rendition-creation}

SDK anropar en asynkron [återgivningscallback-funktion](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) för varje återgivning.

Callback-funktionen har åtkomst till objekten [source](https://github.com/adobe/asset-compute-sdk#source) och [rendition](https://github.com/adobe/asset-compute-sdk#rendition). `source.path` finns redan och är sökvägen till den lokala kopian av källfilen. `rendition.path` är sökvägen där den bearbetade återgivningen måste lagras. Om inte flaggan [disableSourceDownload](https://github.com/adobe/asset-compute-sdk#worker-options-optional) anges måste programmet använda exakt `rendition.path`. Annars kan inte SDK hitta eller identifiera återgivningsfilen och misslyckas.

Den alltför enkla framställningen av exemplet görs för att illustrera och fokusera på anatomin i ett anpassat program. Programmet kopierar bara källfilen till återgivningsmålet.

Mer information om återgivningens återanropsparametrar finns i [Asset Compute SDK API](https://github.com/adobe/asset-compute-sdk#api-details).

### Överför renderingar {#upload-rendition}

När varje återgivning har skapats och lagrats i en fil med sökvägen som tillhandahålls av `rendition.path`, överför [&#x200B; Asset Compute SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) varje återgivning till ett molnlagringsutrymme (antingen AWS eller Azure). Ett anpassat program hämtar flera återgivningar samtidigt om, och bara om, den inkommande begäran har flera återgivningar som pekar på samma program-URL. Överföringen till molnlagringen görs efter varje återgivning och innan återanropet för nästa återgivning körs.

`batchWorker()` har ett annat beteende. Alla återgivningar bearbetas, och först när alla har bearbetats överförs de.

## [!DNL Adobe I/O] händelser {#aio-events}

SDK skickar Adobe [!DNL I/O Events] för varje återgivning. Dessa händelser är antingen av typen `rendition_created` eller `rendition_failed` beroende på resultatet. Mer information finns i [Asset Compute asynkrona händelser](api.md#asynchronous-events).

## Ta emot [!DNL Adobe I/O] händelser {#receive-aio-events}

Klienten avsöker Adobe [!DNL I/O Events]-journalen enligt dess konsumtionslogik. Den inledande journal-URL:en är den som anges i API-svaret för `/register`. Händelser kan identifieras med hjälp av `requestId` som finns i händelserna och är samma som returneras i `/process`. Varje återgivning har en separat händelse som skickas så snart återgivningen har överförts (eller misslyckats). När klienten tar emot en matchande händelse kan den visa eller på annat sätt hantera de resulterande återgivningarna.

JavaScript-biblioteket [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) gör journalavsökningen enkel med metoden `waitActivation()` för att hämta alla händelser.

```javascript
const events = await assetCompute.waitActivation(requestId);
await Promise.all(events.map(event => {
    if (event.type === "rendition_created") {
        // get rendition from cloud storage location
    }
    else if (event.type === "rendition_failed") {
        // failed to process
    }
    else {
        // other event types
        // (could be added in the future)
    }
}));
```

Mer information om hur du hämtar journalhändelser finns i Adobe [[!DNL I/O Events] API](https://developer.adobe.com/events/docs/guides/api/journaling_api/).

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
