---
title: Utveckla för  [!DNL Asset Compute Service]
description: Skapa anpassade program med  [!DNL Asset Compute Service].
exl-id: a0c59752-564b-4bb6-9833-ab7c58a7f38e
source-git-commit: 63f83ff33ac6cd090fac4f6db18000155f464643
workflow-type: tm+mt
source-wordcount: '1489'
ht-degree: 0%

---

# Utveckla ett anpassat program {#develop}

Innan du börjar utveckla ett anpassat program:

* Kontrollera att alla [krav](/help/using/understand-extensibility.md#prerequisites-and-provisioning) är uppfyllda.
* Installera de [nödvändiga programverktygen](/help/using/setup-environment.md#create-dev-environment).
* Se [Konfigurera miljön](setup-environment.md) för att kontrollera att du är redo att skapa ett anpassat program.

## Skapa ett anpassat program {#create-custom-application}

Kontrollera att [Adobe aio-cli](https://github.com/adobe/aio-cli) är installerat lokalt.

1. Om du vill skapa ett anpassat program [skapar du ett App Builder-projekt](https://developer.adobe.com/app-builder/docs/get_started/app_builder_get_started/first-app#4-bootstrapping-new-app-using-the-cli). Kör `aio app init <app-name>` i terminalen om du vill göra det.

   Om du inte redan har loggat in uppmanar det här kommandot en webbläsare att be dig logga in på [Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis) med din Adobe ID. Mer information om hur du loggar in från klippet finns i [här](https://developer.adobe.com/app-builder/docs/get_started/app_builder_get_started/first-app#3-signing-in-from-cli).

   Adobe rekommenderar att du loggar in först. Om du har problem följer du instruktionerna [för att skapa en app utan att logga in](https://developer.adobe.com/app-builder/docs/get_started/app_builder_get_started/first-app#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. När du har loggat in följer du instruktionerna i CLI och väljer `Organization`, `Project` och `Workspace` som ska användas för programmet. Välj det projekt och den arbetsyta som du skapade när du [konfigurerade miljön](setup-environment.md). När du uppmanas till det `Which extension point(s) do you wish to implement ?`, se till att du väljer `DX Asset Compute Worker`:

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

1. Välj `Which Adobe I/O App features do you want to enable for this project?` när du uppmanas till det med `Actions`. Avmarkera alternativet `Web Assets` eftersom webbresurser använder olika autentiserings- och auktoriseringskontroller.

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. När du uppmanas till det `Which type of sample actions do you want to create?`, se till att du väljer `Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. Följ resten av instruktionerna och öppna det nya programmet i Visual Studio Code (eller din favoritkodredigerare). Den innehåller ställningar och exempelkod för ett anpassat program.

   Läs här om de [viktigaste komponenterna i en App Builder-app](https://developer.adobe.com/app-builder/docs/get_started/app_builder_get_started/first-app#5-anatomy-of-an-app-builder-application).

   Mallprogrammet använder Adobe [Asset Compute SDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) för att överföra, hämta och organisera programåtergivningar, så att utvecklare bara behöver implementera den anpassade programlogiken. I mappen `actions/<worker-name>` är filen `index.js` den plats där den anpassade programkoden ska läggas till.

Se [exempel på anpassade program](#try-sample) för exempel och idéer på anpassade program.

### Lägg till autentiseringsuppgifter {#add-credentials}

När du loggar in när du skapar programmet samlas de flesta App Builder-inloggningsuppgifterna in i din ENV-fil. Om du använder utvecklingsverktyget måste du dock ha ytterligare autentiseringsuppgifter.

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### Lagringsuppgifter för utvecklingsverktyget {#developer-tool-credentials}

Verktyget för utvecklare att utvärdera anpassade appar med [!DNL Asset Compute service] kräver att en molnlagringsbehållare används. Den här behållaren är viktig för att lagra testfiler och för att ta emot och presentera återgivningar som produceras av programmen.

>[!NOTE]
>
>Den här behållaren är skild från molnlagringen för [!DNL Adobe Experience Manager] som en [!DNL Cloud Service]. Det gäller endast för utveckling och testning med Asset Compute utvecklingsverktyg.

Kontrollera att du har åtkomst till en [molnlagringsbehållare](https://github.com/adobe/asset-compute-devtool#prerequisites) som stöds. Den här behållaren används tillsammans av olika utvecklare för olika projekt när det behövs.

#### Lägg till autentiseringsuppgifter i ENV-filen {#add-credentials-env-file}

Infoga de efterföljande autentiseringsuppgifterna för utvecklingsverktyget i filen `.env`. Filen finns i roten av ditt App Builder-projekt:
<!--
1. Add the absolute path to the private key file created while adding services to your App Builder Project:

    ```conf
    ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
    ```

   >[!NOTE]
   >
   >JWT is deprecated and Private Key is not available for download. While we are working on updating the testing tools, note that custom workers created using OAuth can be deployed but devtools would not work.
-->
1. Hämta filen från Adobe Developer Console. Gå till projektets rot och klicka på &quot;Hämta alla&quot; i det övre högra hörnet. Filen hämtas med `<namespace>-<workspace>.json` som filnamn. Gör något av följande:

   * Byt namn på filen till `console.json` och flytta den i projektets rot.
   * Du kan också lägga till den absoluta sökvägen till JSON-filen för Adobe Developer Console-integrering. Den här filen är samma [`console.json`](https://developer.adobe.com/app-builder/docs/get_started/app_builder_get_started/first-app#42-developer-is-not-logged-in-as-enterprise-organization-user)-fil som hämtas på din projektarbetsyta.

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
>Filen `config.json` innehåller autentiseringsuppgifter. Lägg till JSON-filen i din `.gitignore`-fil från ditt projekt för att förhindra att den delas. Detsamma gäller för `.env`- och `.aio`-filer.

## Kör programmet {#run-custom-application}

Konfigurera [inloggningsuppgifterna](#developer-tool-credentials) korrekt innan du kör programmet med utvecklarverktyget i Asset Compute.

Om du vill köra programmet i utvecklingsverktyget använder du kommandot `aio app run`. Den distribuerar åtgärden till Adobe [!DNL I/O Runtime] och startar utvecklingsverktyget på den lokala datorn. Det här verktyget används för att testa programbegäranden under utveckling. Här är ett exempel på en renderingsförfrågan:

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
>Använd inte flaggan `--local` med kommandot `run`. Det fungerar inte med [!DNL Asset Compute] anpassade program och Asset Compute utvecklarverktyg. Anpassade program anropas av tjänsten [!DNL Asset Compute] som inte har åtkomst till åtgärder som körs på utvecklarens lokala datorer.

Se [här](test-custom-application.md) om hur du testar och felsöker programmet. När du är klar med utvecklingen av ditt anpassade program [distribuerar du ditt anpassade program](deploy-custom-application.md).

## Prova exempelprogrammet från Adobe {#try-sample}

Följande är exempel på anpassade program:

* [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [worker-Animal-images](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### Anpassat mallprogram {#template-custom-application}

[worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) är ett mallprogram. Det genererar en återgivning genom att bara kopiera källfilen. Innehållet i det här programmet är den mall som togs emot när `Adobe Asset Compute` valdes när AIR-programmet skapades.

Programfilen, [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js), använder [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) för att hämta källfilen, ordna varje återgivningsbearbetning och överföra de återgivningar som blir resultatet tillbaka till molnlagringen.

[`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) som definieras i programkoden är var all programbearbetningslogik ska utföras. Återgivningsåteranropet i `worker-basic` kopierar bara källfilens innehåll till återgivningsfilen.

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

[`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) skapar till exempel en hämtningsbegäran till en statisk URL från Wikimedia med hjälp av biblioteket [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer).

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

Ett exempel på en programfil som använder en anpassad parameter är följande:

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

`example-worker-animal-pictures` skickar en anpassad parameter [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) för att avgöra vilken fil som ska hämtas från Wikimedia.

## Stöd för autentisering och auktorisering {#authentication-authorization-support}

Som standard följer Asset Compute anpassade program autentisering och autentisering för App Builder-projektet. Aktiveras genom att `require-adobe-auth`-anteckningen ställs in på `true` i `manifest.yml`.

### Få åtkomst till andra Adobe API:er {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

Lägg till API-tjänsterna på arbetsytan för konsolen [!DNL Asset Compute] som skapades i konfigurationen. De här tjänsterna ingår i JWT-åtkomsttoken som genererats av [!DNL Asset Compute Service]. Token och andra autentiseringsuppgifter är tillgängliga inuti programåtgärdsobjektet `params`.

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### Skicka autentiseringsuppgifter för system från tredje part {#pass-credentials-for-tp}

Om du vill hantera autentiseringsuppgifter för andra externa tjänster skickar du dem som standardparametrar för åtgärderna. De krypteras automatiskt vid transitering. Mer information finns i [Skapa åtgärder i utvecklarhandboken för Adobe I/O Runtime](https://developer.adobe.com/app-builder/docs/guides/runtime_guides/creating-actions#). Ange dem sedan med hjälp av miljövariabler under distributionen. De här parametrarna kan nås i `params`-objektet inuti åtgärden.

Ange standardparametrarna i `inputs` i `manifest.yml`:

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

Uttrycket `$VAR` läser värdet från en miljövariabel med namnet `VAR`.

Under utvecklingen kan du tilldela värdet i den lokala `.env`-filen. Orsaken är att `aio` automatiskt importerar miljövariabler från `.env`-filer, tillsammans med de variabler som anges av det inledande skalet. I det här exemplet ser filen `.env` ut så här:

```CONF
#...
SECRET_KEY=secret-value
```

För produktionsdistribution kan du ange miljövariabler i CI-systemet, till exempel med hjälp av hemligheter i GitHub-åtgärder. Slutligen får du tillgång till standardparametrarna i själva programmet:

```javascript
const key = params.secretKey;
```

## Storleksändra program {#sizing-workers}

Ett program körs i en behållare i Adobe [!DNL I/O Runtime] med [limits](https://developer.adobe.com/app-builder/docs/guides/runtime_guides/system-settings#) som kan konfigureras via `manifest.yml`:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

På grund av omfattande bearbetning i Asset Compute-program måste du justera dessa gränser för optimala prestanda (tillräckligt stor för att hantera binära resurser) och effektivitet (inte slösa bort resurser på grund av oanvänt behållarminne).

Standardtidsgränsen för åtgärder i körtid är en minut, men den kan ökas genom att `timeout`-gränsen anges (i millisekunder). Om du förväntar dig att bearbeta större filer ökar du den här tiden. Tänk på hur lång tid det tar att hämta källan, bearbeta filen och överföra återgivningen. Om en åtgärd gör timeout, d.v.s. inte returnerar aktiveringen före den angivna tidsgränsen, tas behållaren bort och används inte igen.

Asset Compute-program är till sin natur ofta nätverks- och disk-indata eller -utdata. Källfilen måste hämtas först. Bearbetningen är ofta resurskrävande och återgivningarna överförs sedan igen.

Du kan ange det minne som tilldelats en åtgärdsbehållare i megabyte med parametern `memorySize`. För närvarande definierar den här parametern också hur mycket CPU-åtkomst behållaren får, och viktigast av allt är det en viktig del av kostnaden för att använda körningsversionen (större behållare kostar mer). Använd ett större värde här när bearbetningen kräver mer minne eller CPU, men var försiktig så att du inte slösar bort resurser ju större behållare det är, desto lägre blir den totala genomströmningen.

Dessutom är det möjligt att kontrollera åtgärdssamtidighet i en behållare med inställningen `concurrency`. Den här inställningen är antalet samtidiga aktiveringar som en enskild behållare (av samma åtgärd) får. I den här modellen fungerar åtgärdsbehållaren som en Node.js-server som tar emot flera samtidiga begäranden, upp till den gränsen. Standardvärdet `memorySize` i körningsversionen är 200 MB, vilket är idealiskt för mindre App Builder-åtgärder. För Asset Compute-program kan det här standardvärdet bli för stort på grund av deras hårdvarubearbetning och diskanvändning. Vissa program kanske inte fungerar bra med samtidig aktivitet beroende på implementering. Asset Compute SDK säkerställer att aktiveringarna separeras genom att filer skrivs till olika unika mappar.

Testa program för att hitta de optimala siffrorna för `concurrency` och `memorySize`. Större behållare = högre minnesgräns kan möjliggöra mer samtidighet men kan också vara onödigt för lägre trafik.
