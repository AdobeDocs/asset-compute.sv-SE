---
title: Introduktion till  [!DNL Asset Compute Service]
description: '[!DNL Asset Compute Service] är en resurshanteringstjänst i molnet som minskar komplexiteten och förbättrar skalbarheten.'
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '278'
ht-degree: 0%

---

# Översikt över [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] är en skalbar och utökningsbar tjänst för [!DNL Adobe Experience Cloud] som bearbetar digitala resurser. Det kan omvandla bild, video, dokument och andra filformat till olika renderingar, bland annat miniatyrbilder, extraherad text och metadata samt arkiv.

Utvecklare kan plugin-program för anpassade resurser (kallas även anpassade arbetare) för att hantera anpassade användningsfall. Tjänsten fungerar på Adobe [!DNL I/O Runtime]. Den kan utökas via [!DNL Adobe Developer App Builder] headless-appar skrivna i Node.js. De kan utföra anpassade åtgärder som att anropa externa API:er för att utföra bildåtgärder eller utnyttja stödet för [!DNL Adobe Sensei].

[!DNL Adobe Developer App Builder] är ett ramverk för att skapa och distribuera anpassade webbprogram på Adobe [!DNL I/O Runtime] för att utöka Adobe Experience Cloud lösningar. Om du vill skapa anpassade program kan utvecklarna använda [!DNL React Spectrum] (Adobe UI-verktygslådan), skapa mikrotjänster, skapa anpassade händelser och ordna API:er. Se [dokumentation om Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview/).

>[!NOTE]
>
>För närvarande kan [!DNL Asset Compute Service] bara användas via [!DNL Experience Manager] som [!DNL Cloud Service]. Administratörer skapar bearbetningsprofiler som kan anropa [!DNL Asset Compute Service] för att skicka resurser för bearbetning. Se [Använda resursmikrotjänster och bearbetningsprofiler](https://experienceleague.adobe.com/sv/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

## Användningsexempel som stöds av [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] har stöd för ett fåtal vanliga användningsfall, t.ex. grundläggande bildbearbetning, Adobe-programspecifika konverteringar och anpassade program som hanterar komplexa affärsbehov.

Du kan använda webbtjänsten [!DNL Asset Compute] för att generera miniatyrbilder för olika filtyper, bildåtergivning med hög kvalitet för de [filformat som stöds](https://experienceleague.adobe.com/sv/docs/experience-manager-cloud-service/content/assets/file-format-support). Se [användningsfall som stöds via anpassad konfiguration](https://experienceleague.adobe.com/sv/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>Tjänsten tillhandahåller inte resurslagring. Användarna tillhandahåller den och tillhandahåller referenser till källfiler och återgivningsfiler i molnlagringen.

<!-- TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use [!DNL Adobe I/O] Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [Översikt över resursbearbetning med tillgångsmikrotjänster i [!DNL Adobe Experience Manager] som en [!DNL Cloud Service]](https://experienceleague.adobe.com/sv/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview).
>* [Dokumentation för Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview).
>* [Filformat som stöds för bearbetning](https://experienceleague.adobe.com/sv/docs/experience-manager-cloud-service/content/assets/file-format-support).

<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
