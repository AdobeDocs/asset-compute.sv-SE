---
title: Felsökning [!DNL Asset Compute Service]
description: Felsöka och felsöka anpassade program med [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '273'
ht-degree: 0%

---

# Felsökning {#troubleshoot}

Några allmänna felsökningstips som kan hjälpa dig att felsöka med Asset Compute Service är:

* Kontrollera att JavaScript-programmet inte kraschar vid start. Sådana krascher är vanligtvis relaterade till ett saknat bibliotek eller ett beroende.
* Kontrollera att det finns referenser till alla beroenden som ska installeras i programmets `package.json` -fil.
* Se till att eventuella fel som kan uppstå vid rensning vid fel inte genererar egna fel som döljer det ursprungliga problemet.

* När utvecklingsverktyget startas för första gången med ett nytt [!DNL Asset Compute Service] integreringen kan misslyckas med den första bearbetningsbegäran om Asset compute Events Journal inte är helt inställd. Vänta en stund tills journalen har konfigurerats innan du skickar en ny begäran.
* Se till att alla nödvändiga API:er är Asset compute, Adobe [!DNL I/O Events], Events Management och Runtime ingår i Adobe [!DNL `I/O Project`] och Workspace att undvika `/register` eller `/process` fel vid begäran.

## Logga in problem via Adobe [!DNL aio-cli] {#login-via-aio-cli}

Om du har problem med att logga in på [!DNL Adobe Developer Console] [genom Adobe [!DNL aio-cli]](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli)lägger du sedan manuellt till de autentiseringsuppgifter som krävs för att utveckla, testa och distribuera ditt anpassade program:

1. Navigera till ditt Adobe Developer App Builder-projekt och din arbetsyta på [Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis) och tryck **[!UICONTROL Ladda ned]** från det övre högra hörnet. Öppna den här filen och spara denna JSON på en säker plats på datorn.

1. Navigera till ENV-filen i ditt Adobe Developer App Builder-program.

1. Lägg till Adobe [!DNL I/O Runtime] autentiseringsuppgifter. Hämta Adobe [!DNL I/O Runtime] autentiseringsuppgifter från den hämtade JSON-filen. Autentiseringsuppgifterna är under `project.workspace.services.runtime`. Lägg till [!DNL Adobe I/O] Autentiseringsuppgifterna för körning i `AIO_runtime_XXX` variabler:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Lägg till den absoluta sökvägen till den hämtade JSON-filen i steg 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Konfigurera resten av [obligatoriska autentiseringsuppgifter](develop-custom-application.md) behövs för utvecklarverktyget.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
