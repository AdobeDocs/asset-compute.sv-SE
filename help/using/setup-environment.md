---
title: Ange den utvecklingsmiljö som krävs för [!DNL Asset Compute Service]
description: Konfigurera utvecklarmiljö för [!DNL Asset Compute Service] för att börja skapa och testa anpassad kod.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '324'
ht-degree: 0%

---

# Konfigurera en utvecklarmiljö {#create-dev-environment}

Så här skapar du en konfiguration som du kan utveckla för [!DNL Asset Compute Service], följer dessa krav och instruktioner.

1. [Hämta åtkomst och autentiseringsuppgifter](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials) for [!DNL Adobe Developer App Builder].

1. [Konfigurera den lokala miljön](https://developer.adobe.com/app-builder/docs/getting_started/#local-environment-set-up) och de verktyg som krävs.

1. Några andra verktyg som hjälper dig att komma igång smidigt är:

   * [Git](https://git-scm.com/)
   * [Docker Desktop](https://www.docker.com/get-started)
   * [NodeJS](https://nodejs.org) (v14 LTS, udda versioner rekommenderas inte) och [NPM](https://www.npmjs.com). Användare av OS X HomeBrew kan göra `brew install node` för att installera båda. I annat fall hämtar du den från [NodJS nedladdningssida](https://nodejs.org/en/)
   * En IDE som är bra för NodeJS rekommenderar Adobe [Visual Studio Code (VS Code)](https://code.visualstudio.com) eftersom det är den IDE som stöds för felsökaren. Du kan använda vilken annan IDE som helst som kodredigerare, men avancerad användning (till exempel felsökare) stöds ännu inte
   * Installera den senaste Adobe [[!DNL aio-cli]](https://github.com/adobe/aio-cli) (`aio`)
   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. Se till att du uppfyller [krav](/help/using/understand-extensibility.md#prerequisites-and-provisioning)

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## Konfigurera ett App Builder-projekt {#create-App-Builder-project}

1. Kontrollera att det finns en systemadministratör eller utvecklarroll i [!DNL Experience Cloud] organisation. En systemadministratör i [Admin Console](https://adminconsole.adobe.com/overview), anger den här rollen.

1. Logga in på [Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis). Se till att du är en del av samma [!DNL Experience Cloud] organisationen som [!DNL Experience Manager] som [!DNL Cloud Service] integrering. Mer information om Adobe Developer Console finns på [Konsoldokumentation](https://developer.adobe.com/developer-console/docs/guides/).

1. [Skapa ett App Builder-projekt](https://developer.adobe.com/app-builder/docs/getting_started/first_app/). Klicka **[!UICONTROL Skapa nytt projekt]** > **[!UICONTROL Projekt från mall]**. Välj App Builder. Ett nytt App Builder Project med två arbetsytor skapas: `Production` och `Stage`. Lägg till ytterligare arbetsytor, till exempel `Development`, efter behov.

1. I App Builder Project väljer du en arbetsyta och prenumererar på de tjänster som behövs för Asset Compute. Klicka **Lägg till i projekt** > **API** och lägga till `Asset Compute`, `IO Events`och `IO Events Management` tjänster. När du lägger till det första API:t uppmanas du att skapa en privat nyckel. Spara informationen på datorn när du behöver den här nyckeln för att testa det anpassade programmet med utvecklingsverktyget.

## Nästa steg {#next-step}

När miljön är klar är du redo att [skapa ett anpassat program](develop-custom-application.md).

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
