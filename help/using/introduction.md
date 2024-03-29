---
title: Introduktion till [!DNL Asset Compute Service]
description: "[!DNL Asset Compute Service] är en molnbaserad resurshanteringstjänst som minskar komplexiteten och förbättrar skalbarheten."
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '309'
ht-degree: 0%

---

# Översikt över [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] är en skalbar och utbyggbar tjänst i [!DNL Adobe Experience Cloud] för att bearbeta digitalt material. Det kan omvandla bild, video, dokument och andra filformat till olika renderingar, bland annat miniatyrer, extraherad text och metadata samt arkiv.

Utvecklare kan plugin-program för anpassade resurser (kallas även anpassade arbetare) för att hantera anpassade användningsfall. Tjänsten fungerar på [!DNL Adobe I/O] runtime. Den kan utökas genom [!DNL Adobe Developer App Builder] headless-appar skrivna i Node.js. Dessa kan utföra anpassade åtgärder som att anropa externa API:er för att utföra bildåtgärder eller utnyttja [!DNL Adobe Sensei] support.

[!DNL Adobe Developer App Builder] är ett ramverk för att bygga och driftsätta anpassade webbapplikationer på [!DNL Adobe I/O] runtime för att utöka Adobe Experience Cloud lösningar. För att skapa anpassade program kan utvecklarna utnyttja [!DNL React Spectrum] (Adobe’s UI toolkit), skapa mikrotjänster, skapa anpassade händelser och samordna API:er. Se [dokumentation för Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview/).

>[!NOTE]
>
>För närvarande är [!DNL Asset Compute Service] kan endast användas via [!DNL Experience Manager] som [!DNL Cloud Service]. Administratörer skapar bearbetningsprofiler som kan anropa [!DNL Asset Compute Service] för att överföra resurser för bearbetning. Se [använda mikrotjänster och bearbetningsprofiler](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

## Användningsexempel som stöds av [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] har stöd för ett fåtal vanliga användningsområden, t.ex. grundläggande bildbehandling, Adobe applikationsspecifika konverteringar, och skräddarsydda applikationer som tillgodoser komplexa affärsbehov.

Du kan använda [!DNL Asset Compute] webbtjänst för att generera miniatyrbilder för olika filtyper, bildåtergivning med hög kvalitet för [filformat som stöds](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html). Se [användningsfall som stöds via anpassad konfiguration](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

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
>* [Översikt över bearbetning av resurser med hjälp av mikrotjänster på [!DNL Adobe Experience Manager] som [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html).
>* [Dokumentation för Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview).
>* [Filformat som kan bearbetas](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).

<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
