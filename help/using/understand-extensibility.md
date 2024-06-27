---
title: Förstå om att utöka [!DNL Asset Compute Service]
description: När och hur du ska utöka [!DNL Asset Compute Service] funktioner för att hantera egna resurser.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '229'
ht-degree: 0%

---

# Introduktion till utökningsbarhet {#introduction-to-extensibilty}

Många krav på återgivning, till exempel konvertering till format och storleksändring av bilder, hanteras av [Bearbetar profiler i [!DNL Experience Manager] som [!DNL Cloud Service]](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview). Mer komplexa affärsbehov kan behöva en skräddarsydd lösning som passar organisationens behov. [!DNL Asset Compute Service] kan utökas genom att skapa anpassade program som anropas från Bearbeta profiler i [!DNL Experience Manager]. Dessa anpassade program är anpassade till [användningsfall som stöds](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>[!DNL Asset Compute Service] är endast tillgänglig för användning med [!DNL Experience Manager] som [!DNL Cloud Service].

De anpassade programmen är headless [Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder) appar. Utöka [!DNL Asset Compute Service] med skräddarsydda program på ett enkelt sätt via [Asset compute SDK](https://github.com/adobe/asset-compute-sdk) och Adobe Developer App Builder utvecklingsverktyg. Med dessa verktyg kan utvecklare fokusera på affärslogik. Det är enkelt att skapa anpassade program än att skapa en Adobe utan vanlig server [!DNL I/O Runtime] åtgärd. Det är en enda Node.js JavaScript-funktion. The [exempel på grundläggande anpassad applikation](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) illustrerar det.

## Krav och etableringskrav {#prerequisites-and-provisioning}

Kontrollera att du uppfyller följande krav:

* Adobe Developer App Builder verktyg är installerade på datorn.
* An [!DNL Experience Cloud] organisation. Mer information finns på [Starta din App Builder Journey](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* Experience Organization måste ha [!DNL Experience Manager] som [!DNL Cloud Service] aktiverat.
* [!DNL Adobe Experience Cloud] organisationen ingår i [!DNL Adobe Developer App Builder] &quot;developer sneak peek program&quot;. Gå till [hur man ansöker om åtkomst](https://developer.adobe.com/app-builder/docs/overview/getting_access).
* Se till att utvecklarrollen eller administratörsbehörigheten finns i organisationen för utvecklaren.
* Se till att Adobe [[!DNL aio-cli]](https://github.com/adobe/aio-cli) installeras lokalt.

<!-- TBD for later:

* What all accesses and licenses are required?
* What all permissions are required to create, debug, and deploy custom applications?
* How do developers get access and provision the required apps?
* What is repository management?
* Anything on security and data transfer?
* What about handling personal or sensitive information?
* Custom application SLA is dependent on SLAs of various services it depends on.
* Document how the devs can get to know the KPIs of their custom applications. The KPIs are dependent on the performance at Adobe's side, amongst other things.
-->
