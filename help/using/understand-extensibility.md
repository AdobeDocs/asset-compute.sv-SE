---
title: Förstå om att utöka  [!DNL Asset Compute Service]
description: När och hur du ska utöka  [!DNL Asset Compute Service] funktionen för anpassad bearbetning av resurser.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '229'
ht-degree: 0%

---

# Introduktion till utökningsbarhet {#introduction-to-extensibilty}

Många återgivningskrav, som konvertering till format och storleksändring av bilder, hanteras av [Bearbeta profiler i [!DNL Experience Manager] som en [!DNL Cloud Service]](https://experienceleague.adobe.com/sv/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview). Mer komplexa affärsbehov kan behöva en skräddarsydd lösning som passar organisationens behov. [!DNL Asset Compute Service] kan utökas genom att skapa anpassade program som anropas från Bearbeta profiler i [!DNL Experience Manager]. Dessa anpassade program uppfyller de [användningsfall som stöds](https://experienceleague.adobe.com/sv/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>[!DNL Asset Compute Service] är bara tillgängligt för användning med [!DNL Experience Manager] som [!DNL Cloud Service].

De anpassade programmen är headless [Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder)-appar. Det är enkelt att utöka [!DNL Asset Compute Service] med anpassade program via utvecklarverktygen för [Asset Compute SDK](https://github.com/adobe/asset-compute-sdk) och Adobe Developer App Builder. Med dessa verktyg kan utvecklare fokusera på affärslogik. Det är lika enkelt att skapa anpassade program som att skapa en åtgärd utan servern Adobe [!DNL I/O Runtime]. Det är en enda Node.js-JavaScript-funktion. Det [grundläggande anpassade programexemplet](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) illustrerar det.

## Krav och etableringskrav {#prerequisites-and-provisioning}

Kontrollera att du uppfyller följande krav:

* Adobe Developer App Builder verktyg är installerade på datorn.
* En [!DNL Experience Cloud]-organisation. Mer information finns på [Starta din App Builder-resa](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* Experience Organization måste ha [!DNL Experience Manager] som [!DNL Cloud Service] aktiverat.
* Organisationen [!DNL Adobe Experience Cloud] är en del av programmet [!DNL Adobe Developer App Builder] för förhandstitt på utvecklare. Gå till [så här ansöker du om åtkomst](https://developer.adobe.com/app-builder/docs/overview/getting_access).
* Se till att utvecklarrollen eller administratörsbehörigheten finns i organisationen för utvecklaren.
* Kontrollera att Adobe [[!DNL aio-cli]](https://github.com/adobe/aio-cli) är installerat lokalt.

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
