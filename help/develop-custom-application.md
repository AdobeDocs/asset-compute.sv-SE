---
title: Utveckla för [!DNL Asset Compute Service]
description: Skapa anpassade program med [!DNL Asset Compute Service].
exl-id: a0c59752-564b-4bb6-9833-ab7c58a7f38e
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '1618'
ht-degree: 0%

---

# Utveckla ett anpassat program {#develop}

Innan du börjar utveckla ett anpassat program:

* Se till att alla [krav](/help/understand-extensibility.md#prerequisites-and-provisioning) är uppfyllda.
* Installera [nödvändiga programverktyg](/help/setup-environment.md#create-dev-environment).
* Se [konfigurera miljön](setup-environment.md) för att vara säker på att du är redo att skapa ett anpassat program.

## Skapa ett anpassat program {#create-custom-application}

Se till att du har [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) installeras lokalt.

1. Om du vill skapa ett anpassat program [skapa ett App Builder-projekt](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#4-bootstrapping-new-app-using-the-cli). Om du vill göra det kör du `aio app init <app-name>` i terminalen.

   Om du inte redan har loggat in uppmanas du att logga in i [Adobe Developer Console](https://console.adobe.io/) med din Adobe ID. Se [här](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli) om du vill ha mer information om hur du loggar in från klippet.

   Adobe rekommenderar att du loggar in. Om du har problem följer du instruktionerna [för att skapa en app utan att logga in](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. När du har loggat in följer du instruktionerna i CLI och väljer `Organization`, `Project`och `Workspace` som ska användas för programmet. Välj det projekt och den arbetsyta du skapade när du skapade [konfigurera miljön](setup-environment.md). När du uppmanas `Which extension point(s) do you wish to implement ?`, se till att markera `DX Asset Compute Worker`:

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyAdobe Developer App BuilderProject
   ? Which extension point(s) do you wish to implement ? (Press <space> to select, <a>
   to toggle all, <i> to invert selection)
   ❯◯ DX Experience Cloud SPA
   ◯ DX Asset Compute Worker
   ```

1. När du uppmanas till detta `Which Adobe I/O App features do you want to enable for this project?`, markera `Actions`. Se till att avmarkera `Web Assets` som webbresurser använder olika autentiserings- och auktoriseringskontroller.

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. När du uppmanas `Which type of sample actions do you want to create?`, se till att markera `Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. Följ resten av instruktionerna och öppna det nya programmet i Visual Studio Code (eller din favoritkodredigerare). Den innehåller ställningar och exempelkod för ett anpassat program.

   Läs här om [huvudkomponenterna i en App Builder-app](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#5-anatomy-of-an-app-builder-application).

   Mallprogrammet utnyttjar vår [asset compute SDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) för överföring, hämtning och samordning av programåtergivningar så att utvecklare bara behöver implementera den anpassade programlogiken. Innanför `actions/<worker-name>` mapp, `index.js` där du ska lägga till den anpassade programkoden.

Se [exempel på anpassade program](#try-sample) för exempel och idéer för anpassade program.

### Lägg till autentiseringsuppgifter {#add-credentials}

När du loggar in när du skapar programmet samlas de flesta App Builder-inloggningsuppgifterna in i din ENV-fil. Om du använder utvecklingsverktyget måste du dock ha ytterligare autentiseringsuppgifter.

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### Lagringsuppgifter för utvecklingsverktyget {#developer-tool-credentials}

Det utvecklarverktyg som används för att testa anpassade program med det faktiska [!DNL Asset Compute service] kräver en lagringsbehållare i molnet för testfiler och för att ta emot och visa återgivningar som genererats av program.

>[!NOTE]
>
>Detta är skilt från molnlagringen av [!DNL Adobe Experience Manager] som [!DNL Cloud Service]. Det gäller endast för utveckling och testning med utvecklingsverktyget Asset compute.

Se till att du har tillgång till en [molnlagringsbehållare som stöds](https://github.com/adobe/asset-compute-devtool#prerequisites). Den här behållaren kan delas av flera utvecklare i olika projekt efter behov.

#### Lägg till autentiseringsuppgifter i ENV-filen {#add-credentials-env-file}

Lägg till följande autentiseringsuppgifter för utvecklingsverktyget i ENV-filen i roten för ditt App Builder-projekt:

1. Lägg till den absoluta sökvägen till filen med den privata nyckeln som skapades när du lade till tjänster i ditt App Builder-projekt:

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. Hämta filen från Adobe Developer Console. Gå till projektets rot och klicka på &quot;Hämta alla&quot; i det övre högra hörnet. Filen hämtas med `<namespace>-<workspace>.json` som filnamn. Gör något av följande:

   * Byt namn på filen som `console.json` och flytta den i projektets rot.
   * Du kan också lägga till den absoluta sökvägen till JSON-filen för integrering av Adobe Developer Console. Detta är detsamma [`console.json`](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user) fil som hämtas till din projektarbetsyta.

      ```conf
      ASSET_COMPUTE_INTEGRATION_FILE_PATH=
      ```

1. Lägg till autentiseringsuppgifter för S3 eller Azure-lagring. Du behöver bara tillgång till en molnlagringslösning.

   ```conf
   # S3 credentials
   S3_BUCKET=
   AWS_ACCESS_KEY_ID=
   AWS_SECRET_ACCESS_KEY=
   AWS_REGION=
   
   # Azure Storage credentials
   AZURE_STORAGE_ACCOUNT=
   AZURE_STORAGE_KEY=
   AZURE_STORAGE_CONTAINER_NAME=
   ```

>[!TIP]
>
>The `config.json` filen innehåller autentiseringsuppgifter. I ditt projekt lägger du till JSON-filen i `.gitignore` för att förhindra att filen delas. Samma sak gäller för .env- och .aio-filerna.

## Kör programmet {#run-custom-application}

Innan du kör programmet med utvecklarverktyget i Asset compute måste du konfigurera [autentiseringsuppgifter](#developer-tool-credentials).

Om du vill köra programmet i utvecklingsverktyget använder du `aio app run` -kommando. Den implementerar åtgärden på [!DNL Adobe I/O] Kör och starta utvecklingsverktyget på den lokala datorn. Det här verktyget används för att testa programbegäranden under utveckling. Här är ett exempel på en renderingsförfrågan:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg"
    }
]
```

>[!NOTE]
>
>Använd inte `--local` flagga med `run` -kommando. Det fungerar inte med [!DNL Asset Compute] anpassade program och utvecklingsverktyget Asset compute. Anpassade program anropas av [!DNL Asset Compute Service] som inte kan komma åt åtgärder som körs på utvecklarens lokala datorer.

Se [här](test-custom-application.md) hur du testar och felsöker programmet. När du är klar med utvecklingen av ditt anpassade program [distribuera ditt anpassade program](deploy-custom-application.md).

## Prova exempelprogrammet från Adobe {#try-sample}

Följande är exempel på anpassade program:

* [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [djurbilder](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### Anpassat mallprogram {#template-custom-application}

The [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) är ett mallprogram. Det genererar en återgivning genom att bara kopiera källfilen. Innehållet i det här programmet är den mall som tas emot när du väljer `Adobe Asset Compute` när du skapar AIR-appen.

Programfilen, [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) använder [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) om du vill hämta källfilen, samordna varje renderingsbearbetning och överföra de resulterande renderingarna tillbaka till molnlagringen.

The [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) som definieras inuti programkoden, är var all programbearbetningslogik ska utföras. Återgivningsåteranrop i `worker-basic` kopierar bara källfilens innehåll till återgivningsfilen.

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## Anropa ett externt API {#call-external-api}

I programkoden kan du göra externa API-anrop som hjälp vid programbearbetning. Ett exempel på en programfil som anropar ett externt API visas nedan.

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

Till exempel [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) gör en hämtningsbegäran till en statisk URL från Wikimedia med [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) bibliotek.

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### Skicka egna parametrar {#pass-custom-parameters}

Du kan skicka egna definierade parametrar via återgivningsobjekten. De kan refereras inuti programmet i [`rendition` instruktioner](https://github.com/adobe/asset-compute-sdk#rendition). Ett exempel på ett återgivningsobjekt är:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

Ett exempel på en programfil som använder en anpassad parameter är:

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

The `example-worker-animal-pictures` skickar en anpassad parameter [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) för att avgöra vilken fil som ska hämtas från Wikimedia.

## Stöd för autentisering och auktorisering {#authentication-authorization-support}

Som standard levereras anpassade Asset compute-program med behörighets- och autentiseringskontroller för App Builder-projekt. Det här är aktiverat genom att ställa in `require-adobe-auth` anteckning till `true` i `manifest.yml`.

### Åtkomst till andra Adobe-API:er {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

Lägg till API-tjänsterna i [!DNL Asset Compute] Konsolens arbetsyta skapades i konfigurationen. Dessa tjänster är en del av JWT-åtkomsttoken som genereras av [!DNL Asset Compute Service]. Token och andra inloggningsuppgifter är tillgängliga inuti programåtgärden `params` -objekt.

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### Skicka autentiseringsuppgifter för system från tredje part {#pass-credentials-for-tp}

Om du vill hantera autentiseringsuppgifter för andra externa tjänster skickar du dessa som standardparametrar för åtgärderna. Dessa krypteras automatiskt under överföring. Mer information finns i [skapa åtgärder i utvecklarhandboken för Runtime](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md). Ange dem sedan med hjälp av miljövariabler under distributionen. Dessa parametrar finns i `params` -objekt inuti funktionsmakrot.

Ange standardparametrar i `inputs` i `manifest.yml`:

```yaml
packages:
  __APP_PACKAGE__:
    actions:
      worker:
        function: 'index.js'
        runtime: 'nodejs:10'
        web: true
        inputs:
           secretKey: $SECRET_KEY
        annotations:
          require-adobe-auth: true
```

The `$VAR` uttrycket läser värdet från en miljövariabel med namnet `VAR`.

Under utvecklingen kan värdet anges i den lokala ENV-filen som `aio` läser automatiskt miljövariabler från ENV-filer utöver de variabler som anges i anropsgränssnittet. I det här exemplet ser ENV-filen ut så här:

```CONF
#...
SECRET_KEY=secret-value
```

För produktionsdistribution kan du ange miljövariabler i CI-systemet, till exempel med hjälp av hemligheter i GitHub-åtgärder. Slutligen får du tillgång till standardparametrarna i själva programmet:

```javascript
const key = params.secretKey;
```

## Storleksändra program {#sizing-workers}

Ett program körs i en behållare i [!DNL Adobe I/O] Körning med [gränser](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md) som kan konfigureras via `manifest.yml`:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

På grund av den mer omfattande bearbetning som vanligtvis utförs av Asset compute-program är det troligare att man måste justera dessa gränsvärden för optimala prestanda (tillräckligt stor för att hantera binära resurser) och effektivitet (inte slösa resurser på grund av oanvänt behållarminne).

Standardtidsgränsen för åtgärder i körtid är en minut, men den kan ökas genom att ställa in `timeout` gräns (i millisekunder). Om du förväntar dig att bearbeta större filer ökar du den här tiden. Tänk på hur lång tid det tar att hämta källan, bearbeta filen och överföra återgivningen. Om en åtgärd gör timeout, d.v.s. inte returnerar aktiveringen före den angivna tidsgränsen, ignoreras behållaren och återanvänds inte.

Asset compute-applikationer är till sin natur ofta nätverks- och diskingångs- eller utdatabunden. Källfilen måste först laddas ned, bearbetningen är ofta resurskrävande och sedan överförs de resulterande återgivningarna igen.

Minnet som är tillgängligt för en åtgärdsbehållare anges av `memorySize` i MB. För närvarande definierar detta även hur mycket processoråtkomst behållaren får, och viktigast av allt är det en viktig del av kostnaden för att använda körningsversionen (större behållare kostar mer). Använd ett större värde här när bearbetningen kräver mer minne eller processorkapacitet, men var noga med att inte slösa bort resurser eftersom ju större behållare det är, desto lägre blir den totala genomströmningen.

Dessutom är det möjligt att styra åtgärdssamtidighet i en behållare med `concurrency` inställning. Detta är antalet samtidiga aktiveringar som en enskild behållare (av samma åtgärd) får. I den här modellen fungerar åtgärdsbehållaren som en Node.js-server som tar emot flera samtidiga begäranden, upp till den gränsen. Om den inte anges är standardvärdet 200, vilket är bra för mindre App Builder-åtgärder, men vanligtvis för stort för Asset compute-program med tanke på deras mer intensiva lokala bearbetning och diskaktivitet. Vissa program kanske inte fungerar bra med samtidig aktivitet beroende på implementering. Asset compute SDK säkerställer att aktiveringarna separeras genom att filer skrivs till olika unika mappar.

Testa program för att hitta det optimala antalet för `concurrency` och `memorySize`. Större behållare = högre minnesgräns kan möjliggöra mer samtidighet men kan också vara onödigt för lägre trafik.
