---
title: "[!DNL Asset Compute Service] HTTP API"
description: "[!DNL Asset Compute Service] HTTP API för att skapa anpassade program."
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '2862'
ht-degree: 0%

---

# [!DNL Asset Compute Service] HTTP API {#asset-compute-http-api}

API:t används endast i utvecklingssyfte. API:t anges som ett sammanhang när du utvecklar anpassade program. [!DNL Adobe Experience Manager] som [!DNL Cloud Service] använder API:t för att skicka bearbetningsinformationen till ett anpassat program. Mer information finns i [Använda resursmikrotjänster och Bearbeta profiler](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>[!DNL Asset Compute Service] är bara tillgängligt för användning med [!DNL Experience Manager] som [!DNL Cloud Service].

Alla klienter för HTTP-API:t [!DNL Asset Compute Service] måste följa detta högnivåflöde:

1. En klient har etablerats som ett [!DNL Adobe Developer Console]-projekt i en IMS-organisation. Varje separat klient (system eller miljö) kräver ett eget separat projekt för att separera händelsedataflödet.

1. En klient genererar en åtkomsttoken för det tekniska kontot med hjälp av [JWT-autentisering (tjänstkonto)](https://developer.adobe.com/developer-console/docs/guides/).

1. En klient anropar bara [`/register`](#register) en gång för att hämta journal-URL:en.

1. En klient anropar [`/process`](#process-request) för varje resurs som den vill generera återgivningar för. Anropet är asynkront.

1. En klient avsöker regelbundet journalen för att [ta emot händelser](#asynchronous-events). Den tar emot händelser för varje begärd återgivning när återgivningen har bearbetats (`rendition_created` händelsetyp) eller om det finns ett fel (`rendition_failed` händelsetyp).

Modulen [adobe-asset-compute-client](https://github.com/adobe/asset-compute-client) gör det enkelt att använda API:t i Node.js-koden.

## Autentisering och auktorisering {#authentication-and-authorization}

Alla API:er kräver åtkomsttokenautentisering. Förfrågningarna måste ange följande rubriker:

1. `Authorization`-huvud med bearer-token, som är den tekniska kontotoken, har tagits emot via [JWT exchange](https://developer.adobe.com/developer-console/docs/guides/) från Adobe Developer Console-projekt. [omfattningarna](#scopes) beskrivs nedan.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id`-huvud med IMS-organisations-ID.

1. `x-api-key` med klient-ID från projektet [!DNL Adobe Developers Console].

### Omfång {#scopes}

Kontrollera följande scope för åtkomsttoken:

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

Dessa omfattningar kräver att projektet [!DNL Adobe Developer Console] prenumererar på tjänsterna `Asset Compute`, `I/O Events` och `I/O Management API`. Uppdelningen av enskilda omfattningar är:

* Grundläggande
   * omfattningar: `openid,AdobeID`

* Asset compute
   * metascope: `asset_compute_meta`
   * omfattningar: `asset_compute,read_organizations`

* Adobe [!DNL `I/O Events`]
   * metascope: `event_receiver_api`
   * omfattningar: `event_receiver,event_receiver_api`

* Adobe [!DNL `I/O Management API`]
   * metascope: `ent_adobeio_sdk`
   * omfattningar: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## Registrering {#register}

Varje klient för [!DNL Asset Compute service] - ett unikt [!DNL Adobe Developer Console]-projekt som prenumererar på tjänsten - måste [registrera](#register-request) innan bearbetningen kan göras. Registreringssteget returnerar den unika händelsjournal som krävs för att hämta asynkrona händelser från återgivningsbearbetningen.

När livscykeln är slut kan en klient [avregistrera](#unregister-request).

### Registrera förfrågan {#register-request}

Detta API-anrop konfigurerar en [!DNL Asset Compute]-klient och tillhandahåller händelsens journal-URL. Den här processen är en depotent åtgärd och behöver bara anropas en gång för varje klient. Den kan anropas igen för att hämta journal-URL:en.

| Parameter | Värde |
|--------------------------|------------------------------------------------------|
| Metod | `POST` |
| Bana | `/register` |
| Sidhuvud `Authorization` | Alla [auktoriseringsrelaterade rubriker](#authentication-and-authorization). |
| Sidhuvud `x-request-id` | Valfritt, angivet av klienter för en unik slutidentifierare av bearbetningsbegäranden mellan olika system. |
| Begärandetext | Måste vara tomt. |

### Registrera svar {#register-response}

| Parameter | Värde |
|-----------------------|------------------------------------------------------|
| MIME-typ | `application/json` |
| Sidhuvud `X-Request-Id` | Antingen samma som begärandehuvudet `X-Request-Id` eller en unikt genererad. Används för att identifiera förfrågningar mellan system, supportförfrågningar eller båda. |
| Svarstext | Ett JSON-objekt med fälten `journal`, `ok` eller `requestId`. |

HTTP-statuskoderna är:

* **200 lyckades**: När begäran lyckades. URL:en `journal` får meddelanden om resultatet av den asynkrona bearbetning som initierats med hjälp av `/process`. Den varnar om `rendition_created` händelser när de har slutförts, eller `rendition_failed` händelser om processen misslyckas.

  ```json
  {
      "ok": true,
      "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "requestId": "1234567890"
  }
  ```

* **401 Obehörig**: inträffar när begäran inte har giltig [autentisering](#authentication-and-authorization). Ett exempel kan vara en ogiltig åtkomsttoken eller en ogiltig API-nyckel.

* **403 Otillåten**: inträffar när begäran inte har giltig [auktorisering](#authentication-and-authorization). Ett exempel kan vara en giltig åtkomsttoken, men Adobe Developer Console-projektet (tekniskt konto) prenumererar inte på alla nödvändiga tjänster.

* **429 För många begäranden**: inträffar när den här klienten eller på annat sätt överbelastar systemet. Klienter bör försöka igen med en [exponentiell säkerhetskopiering](https://en.wikipedia.org/wiki/Exponential_backoff). Brödtexten är tom.
* **4xx-fel**: När det fanns något annat klientfel och registreringen misslyckades. Vanligtvis returneras ett JSON-svar som detta, men det garanteras inte för alla fel:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx-fel**: inträffar när det finns något annat fel på serversidan och registreringen misslyckades. Vanligtvis returneras ett JSON-svar som detta, men det garanteras inte för alla fel:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

### Avregistrera förfrågan {#unregister-request}

Detta API-anrop avregistrerar en [!DNL Asset Compute]-klient. När avregistreringen är klar går det inte längre att anropa `/process`. Om API-anropet för en oregistrerad klient eller en ännu inte registrerad klient används returneras felet `404`.

| Parameter | Värde |
|--------------------------|------------------------------------------------------|
| Metod | `POST` |
| Bana | `/unregister` |
| Sidhuvud `Authorization` | Alla [auktoriseringsrelaterade rubriker](#authentication-and-authorization). |
| Sidhuvud `x-request-id` | Valfritt. Klienterna kan ange den för en unik slutidentifierare av bearbetningsförfrågningarna mellan olika system. |
| Begärandetext | Tom. |

### Avregistrera svar {#unregister-response}

| Parameter | Värde |
|-----------------------|------------------------------------------------------|
| MIME-typ | `application/json` |
| Sidhuvud `X-Request-Id` | Antingen samma som begärandehuvudet `X-Request-Id` eller en unikt genererad. Används för att identifiera förfrågningar mellan system eller supportförfrågningar. |
| Svarstext | Ett JSON-objekt med fälten `ok` och `requestId`. |

Statuskoderna är:

* **200 lyckades**: inträffar när registreringen och journalen hittas och tas bort.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **401 Obehörig**: inträffar när begäran inte har giltig [autentisering](#authentication-and-authorization). Ett exempel kan vara en ogiltig åtkomsttoken eller en ogiltig API-nyckel.

* **403 Otillåten**: inträffar när begäran inte har giltig [auktorisering](#authentication-and-authorization). Ett exempel kan vara en giltig åtkomsttoken, men Adobe Developer Console-projektet (tekniskt konto) prenumererar inte på alla nödvändiga tjänster.

* **404 Det gick inte att hitta**: Den här statusen visas när de angivna autentiseringsuppgifterna är oregistrerade eller ogiltiga.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **429 För många begäranden**: inträffar när systemet är överbelastat. Klienter bör försöka igen med en [exponentiell säkerhetskopiering](https://en.wikipedia.org/wiki/Exponential_backoff). Brödtexten är tom.

* **4xx-fel**: inträffar när det finns något annat klientfel och avregistreringen misslyckades. Vanligtvis returneras ett JSON-svar som detta, men det garanteras inte för alla fel:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx-fel**: inträffar när det finns något annat fel på serversidan och registreringen misslyckades. Vanligtvis returneras ett JSON-svar som detta, men det garanteras inte för alla fel:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

## Bearbeta begäran {#process-request}

Åtgärden `process` skickar ett jobb som omvandlar en källresurs till flera återgivningar, baserat på instruktionerna i begäran. Meddelanden om slutförande (händelsetyp `rendition_created`) eller eventuella fel (händelsetyp `rendition_failed`) skickas till en händelseljournal som måste hämtas med [`/register`](#register) en gång innan du kan göra ett antal `/process`-begäranden. Felaktigt utformade begäranden misslyckas omedelbart med 400-felkod.

Binärfiler refereras med URL:er, som försignerade URL:er för Amazon AWS S3 eller Azure Blob Storage SAS-URL:er. Används för att både läsa `source`-resursen (`GET` URL:er) och skriva återgivningarna (`PUT` URL:er). Klienten ansvarar för att generera dessa försignerade URL:er.

| Parameter | Värde |
|--------------------------|------------------------------------------------------|
| Metod | `POST` |
| Bana | `/process` |
| MIME-typ | `application/json` |
| Sidhuvud `Authorization` | Alla [auktoriseringsrelaterade rubriker](#authentication-and-authorization). |
| Sidhuvud `x-request-id` | Valfritt. Klienter kan ange en unik slutidentifierare för att spåra bearbetningsbegäranden i olika system. |
| Begärandetext | Det måste vara i JSON-format för processbegäran enligt beskrivningen nedan. Den innehåller instruktioner om vilken resurs som ska bearbetas och vilka renderingar som ska genereras. |

### Bearbeta begäran JSON {#process-request-json}

Begärandetexten för `/process` är ett JSON-objekt med detta högnivåschema:

```json
{
    "source": "",
    "renditions" : []
}
```

De tillgängliga fälten är:

| Namn | Typ | Beskrivning | Exempel |
|--------------|----------|-------------|---------|
| `source` | `string` | URL för källresursen som bearbetas. Valfritt, baserat på begärt återgivningsformat (till exempel `fmt=zip`). | `"http://example.com/image.jpg"` |
| `source` | `object` | Beskriver källresursen som bearbetas. Se beskrivningen av [Source-objektfält](#source-object-fields) nedan. Valfritt baserat på begärt återgivningsformat (till exempel `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | Återgivningar att generera från källfilen. Varje återgivningsobjekt har stöd för en [återgivningsinstruktion](#rendition-instructions). Obligatoriskt. | `[{ "target": "https://....", "fmt": "png" }]` |

`source` kan antingen vara en `<string>` som ses som en URL eller en `<object>` med ett extra fält. Följande varianter är lika:

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### Source objektfält {#source-object-fields}

| Namn | Typ | Beskrivning | Exempel |
|-----------|----------|-------------|---------|
| `url` | `string` | URL för källresursen som ska bearbetas. Obligatoriskt. | `"http://example.com/image.jpg"` |
| `name` | `string` | Source resursfilnamn. Ett filtillägg i namnet kan användas om ingen MIME-typ identifieras. Den har prioritet över det filnamn som anges i URL-sökvägen. Den har dessutom prioritet över filnamnet i `content-disposition`-huvudet för den binära resursen. Standardvärdet är &quot;file&quot;. | `"image.jpg"` |
| `size` | `number` | Source filstorlek i byte. Prioriterar över `content-length` huvud för den binära resursen. | `10234` |
| `mimetype` | `string` | MIME-typ för Source-resursfil. Prioriterar huvuden `content-type` för den binära resursen. | `"image/jpeg"` |

### Ett fullständigt exempel på `process`-begäran {#complete-process-request-example}

```json
{
    "source": "https://www.adobe.com/content/dam/acom/en/lobby/lobby-bg-bts2017-logged-out-1440x860.jpg",
    "renditions" : [{
            "name": "image.48x48.png",
            "target": "https://some-presigned-put-url-for-image.48x48.png",
            "fmt": "png",
            "width": 48,
            "height": 48
        },{
            "name": "image.200x200.jpg",
            "target": "https://some-presigned-put-url-for-image.200x200.jpg",
            "fmt": "jpg",
            "width": 200,
            "height": 200
        },{
            "name": "cqdam.xmp.xml",
            "target": "https://some-presigned-put-url-for-cqdam.xmp.xml",
            "fmt": "xmp"
        },{
            "name": "cqdam.text.txt",
            "target": "https://some-presigned-put-url-for-cqdam.text.txt",
            "fmt": "text"
    }]
}
```

## Processsvar {#process-response}

`/process`-begäran returneras omedelbart med ett lyckat eller misslyckat svar baserat på den grundläggande begärandevalideringen. Faktisk bearbetning av resurser sker asynkront.

| Parameter | Värde |
|-----------------------|------------------------------------------------------|
| MIME-typ | `application/json` |
| Sidhuvud `X-Request-Id` | Antingen samma som begärandehuvudet `X-Request-Id` eller en unikt genererad. Används för att identifiera förfrågningar mellan system eller supportförfrågningar. |
| Svarstext | Ett JSON-objekt med fälten `ok` och `requestId`. |

Statuskoder:

* **200 lyckades**: Om begäran skickades utan fel. Svar-JSON innehåller `"ok": true`:

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **400 Ogiltig begäran**: Om begäran är felaktigt strukturerad, till exempel om den saknar obligatoriska fält i JSON-nyttolasten. Svar-JSON innehåller `"ok": false`:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **401 Obehörig**: När begäran inte har giltig [autentisering](#authentication-and-authorization). Ett exempel kan vara en ogiltig åtkomsttoken eller en ogiltig API-nyckel.
* **403 Otillåten**: När begäran inte har giltig [auktorisering](#authentication-and-authorization). Ett exempel kan vara en giltig åtkomsttoken, men Adobe Developer Console-projektet (tekniskt konto) prenumererar inte på alla nödvändiga tjänster.
* **429 För många begäranden**: Inträffar när systemet överbelastas, antingen på grund av den här klienten eller på grund av den totala efterfrågan. Klienterna kan försöka igen med en [exponentiell säkerhetskopiering](https://en.wikipedia.org/wiki/Exponential_backoff). Brödtexten är tom.
* **4xx-fel**: När det fanns något annat klientfel. Vanligtvis returneras ett JSON-svar som detta, men det garanteras inte för alla fel:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx-fel**: När det fanns något annat serverfel. Vanligtvis returneras ett JSON-svar som detta, men det garanteras inte för alla fel:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

De flesta klienter vill antagligen försöka utföra samma begäran igen med [exponentiell säkerhetskopiering](https://en.wikipedia.org/wiki/Exponential_backoff) vid ett fel *förutom* konfigurationsproblem som 401 eller 403, eller ogiltiga begäranden som 400. Förutom en vanlig hastighetsbegränsning med 429 svar kan ett tillfälligt avbrott eller en tillfällig begränsning leda till 5 x fel. Det vore då tillrådligt att försöka igen efter en viss tid.

Alla JSON-svar (om sådana finns) innehåller `requestId`, som är samma värde som huvudet `X-Request-Id`. Adobe rekommenderar att du läser från sidhuvudet eftersom det alltid finns. `requestId` returneras också i alla händelser som rör bearbetning av begäranden som `requestId`. Klienterna får inte anta något om formatet för den här strängen. Det är en ogenomskinlig strängidentifierare.

## Anmäl dig till efterbearbetning {#opt-in-to-post-processing}

[Asset Compute SDK](https://github.com/adobe/asset-compute-sdk) har stöd för en uppsättning grundläggande alternativ för efterbearbetning av bilder. Anpassade arbetare kan uttryckligen välja att efterbearbeta genom att ange fältet `postProcess` för återgivningsobjektet till `true`.

De användningsområden som stöds är:

* Beskärning är en återgivning av en rektangel vars gränser definieras genom crop.w, crop.h, crop.x och crop.y. Beskärningsinformationen anges i återgivningsobjektets `instructions.crop`-fält.
* Ändra storlek på bilder med bredden, höjden eller båda. `instructions.width` och `instructions.height` definierar det i återgivningsobjektet. Om du bara vill ändra storlek med bredd eller höjd anger du bara ett värde. Beräkningstjänsten bevarar proportionerna.
* Ange kvaliteten för en JPEG-bild. `instructions.quality` definierar det i återgivningsobjektet. En kvalitetsnivå på 100 representerar den högsta kvaliteten, medan lägre tal innebär en kvalitetsförsämring.
* Skapa sammanflätade bilder. `instructions.interlace` definierar det i återgivningsobjektet.
* Ställ in DPI för att justera den återgivna storleken för DPI-publicering genom att justera den skala som används på pixlarna. `instructions.dpi` definierar den i återgivningsobjektet för att ändra dpi-upplösning. Om du vill ändra storlek på bilden så att den får samma storlek med en annan upplösning använder du `convertToDpi`-instruktionerna.
* Ändra bildens storlek så att dess återgivna bredd eller höjd förblir densamma som originalet vid den angivna målupplösningen (DPI). `instructions.convertToDpi` definierar det i återgivningsobjektet.

## Vattenstämpelresurser {#add-watermark}

[Asset Compute SDK](https://github.com/adobe/asset-compute-sdk) har stöd för att lägga till en vattenstämpel i bildfilerna PNG, JPEG, TIFF och GIF. Vattenstämpeln läggs till efter återgivningsinstruktionerna i objektet `watermark` på återgivningen.

Vattenstämplar görs under efterbearbetningen av återgivningen. För vattenstämpelresurser väljer den anpassade arbetaren [efterbearbetning](#opt-in-to-post-processing) genom att ange fältet `postProcess` för återgivningsobjektet till `true`. Om arbetaren inte väljer att delta används inte vattenstämplar, även om vattenstämpelobjektet har angetts för återgivningsobjektet i begäran.

## Återgivningsinstruktioner {#rendition-instructions}

Följande är tillgängliga alternativ för arrayen `renditions` i [`/process`](#process-request).

### Vanliga fält {#common-fields}

| Namn | Typ | Beskrivning | Exempel |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | Målformatet för återgivningar kan också vara `text` för textrahering och `xmp` för att extrahera XMP metadata som xml. Se [format som stöds](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support) | `png` |
| `worker` | `string` | URL för ett [anpassat program](develop-custom-application.md). Måste vara en `https://`-URL. Om det här fältet finns skapar ett anpassat program återgivningen. Alla andra inställda återgivningsfält används sedan i det anpassade programmet. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | Den URL som den genererade återgivningen ska överföras till med HTTP PUT. | `http://w.com/img.jpg` |
| `target` | `object` | Multipart-försignerad URL-överföringsinformation för den genererade återgivningen. Den här informationen gäller för [AEM/Oak Direct Binary Upload](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) med det här [multipart-överföringsbeteendet](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>Fält:<ul><li>`urls`: strängmatris, en för varje försignerad del-URL</li><li>`minPartSize`: den minsta storleken som ska användas för en del = url</li><li>`maxPartSize`: den största storleken som kan användas för en del = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Valfritt. Klienten styr det reserverade utrymmet och skickar det vidare på samma sätt som renderingshändelser. Gör att en klient kan lägga till anpassad information för att identifiera återgivningshändelser. Den får inte ändras eller förlitas i anpassade program, eftersom kunderna kan ändra den när som helst. | `{ ... }` |

### Återgivningsspecifika fält {#rendition-specific-fields}

En lista över de filformat som stöds finns i [Filformat som stöds](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support).

| Namn | Typ | Beskrivning | Exempel |
|-------------------|----------|-------------|---------|
| `*` | `*` | Avancerade, anpassade fält kan läggas till som ett [anpassat program](develop-custom-application.md) förstår. | |
| `embedBinaryLimit` | `number` i byte | När filstorleken för återgivningen är mindre än det angivna värdet inkluderas den i händelsen som skickas när återgivningen har skapats. Den största tillåtna storleken för inbäddning är 32 kB (32 x 1 024 byte). Om en återgivning är större än `embedBinaryLimit`-gränsen placeras den på en plats i molnlagringen och bäddas inte in i händelsen. | `3276` |
| `width` | `number` | Bredd i pixlar. endast för bildåtergivningar. | `200` |
| `height` | `number` | Höjd i pixlar. endast för bildåtergivningar. | `200` |
|                   |          | Proportionerna behålls alltid om: <ul> <li> Både `width` och `height` har angetts och bilden får plats i storleken samtidigt som proportionerna behålls </li><li> Om bara `width` eller `height` anges används motsvarande dimension för den resulterande bilden, samtidigt som proportionerna behålls</li><li> Om `width` eller `height` inte anges används den ursprungliga bildens pixelstorlek. Det beror på källtypen. För vissa format, till exempel PDF, används en standardstorlek. Det kan finnas en maximal storleksgräns.</li></ul> | |
| `quality` | `number` | Ange jpeg-kvalitet i intervallet `1` till `100`. Gäller endast för bildåtergivningar. | `90` |
| `xmp` | `string` | Används endast vid XMP av metadatatillbakaskrivning, är det base64-kodade XMP att skriva tillbaka till den angivna återgivningen. | |
| `interlace` | `bool` | Skapa sammanflätad PNG, GIF eller progressiv JPEG genom att ange den till `true`. Det påverkar inte andra filformat. | |
| `jpegSize` | `number` | Ungefärlig storlek på filen JPEG i byte. Den åsidosätter alla `quality`-inställningar. Det påverkar inte andra format. | |
| `dpi` | `number` eller `object` | Ange x- och y-DPI. För enkelhetens skull kan den också anges till ett enda tal, som används för både x och y. Det påverkar inte själva bilden. | `96` eller `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` eller `object` | x- och y-DPI samplar om värden med bibehållen fysisk storlek. För enkelhetens skull kan den också anges till ett enda tal, som används för både x och y. | `96` eller `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Lista över filer som ska inkluderas i ZIP-arkivet (`fmt=zip`). Varje post kan antingen vara en URL-sträng eller ett objekt med fälten:<ul><li>`url`: URL för hämtning av fil</li><li>`path`: Lagra filen under den här sökvägen i ZIP</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Duplicerad hantering för ZIP-arkiv (`fmt=zip`). Som standard genererar flera filer som lagras under samma sökväg i ZIP ett fel. Om `duplicate` anges till `ignore` sparas bara den första resursen och resten ignoreras. | `ignore` |
| `watermark` | `object` | Innehåller instruktioner om [vattenstämpeln](#watermark-specific-fields). |  |

### Vattenstämpelsspecifika fält {#watermark-specific-fields}

PNG-formatet används som vattenstämpel.

| Namn | Typ | Beskrivning | Exempel |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Skala för vattenstämpeln, mellan `0.0` och `1.0`. `1.0` betyder att vattenstämpeln har sin ursprungliga skala (1:1) och de lägre värdena minskar storleken på vattenstämpeln. | Värdet `0.5` betyder halva den ursprungliga storleken. |
| `image` | `url` | URL till PNG-filen som ska användas för vattenstämpel. | |

## Asynkrona händelser {#asynchronous-events}

När bearbetningen av en återgivning är klar eller när ett fel inträffar, skickas en händelse till Adobe [!DNL `I/O Events Journal`]. Klienterna måste lyssna på den journal-URL som tillhandahålls via [`/register`](#register). Journalsvaret innehåller en `event`-matris som består av ett objekt för varje händelse, varav fältet `event` innehåller den faktiska händelsens nyttolast.

Adobe [!DNL `I/O Events`]-typen för alla händelser i [!DNL Asset Compute Service] är `asset_compute`. Journalen prenumererar automatiskt på endast den här händelsetypen och det finns inget ytterligare krav på att filtrera baserat på händelsetypen [!DNL Adobe Developer]. Tjänstspecifika händelsetyper är tillgängliga i egenskapen `type` för händelsen.

### Händelsetyper {#event-types}

| Händelse | Beskrivning |
|---------------------|-------------|
| `rendition_created` | Skickat för varje återgivning som har bearbetats och överförts. |
| `rendition_failed` | Skickat för varje återgivning som inte kunde bearbetas eller överföras. |

### Händelseattribut {#event-attributes}

| Attribut | Typ | Händelse | Beskrivning |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Tidsstämpel när händelsen skickades i förenklat utökat [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601)-format, enligt definition i JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | ID för den ursprungliga begäran till `/process`, samma som för `X-Request-Id`. |
| `source` | `object` | `*` | `source` för `/process`-begäran. |
| `userData` | `object` | `*` | `userData` för återgivningen från `/process`-begäran om den anges. |
| `rendition` | `object` | `rendition_*` | Motsvarande återgivningsobjekt skickades i `/process`. |
| `metadata` | `object` | `rendition_created` | Återgivningens [metadata](#metadata)-egenskaper. |
| `errorReason` | `string` | `rendition_failed` | Återgivningsfel: [orsak](#error-reasons), om sådan finns. |
| `errorMessage` | `string` | `rendition_failed` | Texten med mer information om eventuella återgivningsfel. |

### Metadata {#metadata}

| Egenskap | Beskrivning |
|--------|-------------|
| `repo:size` | Återgivningens storlek i byte. |
| `repo:sha1` | Återgivningens sha1-sammanfattning. |
| `dc:format` | Återgivningens MIME-typ. |
| `repo:encoding` | Teckenuppsättningskoden för återgivningen om det är ett textbaserat format. |
| `tiff:ImageWidth` | Återgivningens bredd i pixlar. Finns endast för bildåtergivningar. |
| `tiff:ImageLength` | Återgivningens längd i pixlar. Finns endast för bildåtergivningar. |

### Felorsaker {#error-reasons}

| Orsak | Beskrivning |
|---------|-------------|
| `RenditionFormatUnsupported` | Det begärda återgivningsformatet stöds inte för den angivna källan. |
| `SourceUnsupported` | Den specifika källan stöds inte trots att typen stöds. |
| `SourceCorrupt` | Källdata är skadade. Den innehåller tomma filer. |
| `RenditionTooLarge` | Det gick inte att överföra återgivningen med de försignerade URL:erna i `target`. Den faktiska återgivningsstorleken är tillgänglig som metadata i `repo:size` och används av klienten för att bearbeta återgivningen med rätt antal försignerade URL:er. |
| `GenericError` | Andra oväntade fel. |
