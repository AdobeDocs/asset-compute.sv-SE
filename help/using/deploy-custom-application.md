---
title: Distribuera  [!DNL Asset Compute Service] anpassat program
description: Distribuera  [!DNL Asset Compute Service] anpassat program.
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '170'
ht-degree: 0%

---

# Distribuera ett anpassat program {#deploy-custom-application}

Om du vill distribuera ditt program använder du kommandot [AIR-appdistribution](https://github.com/adobe/aio-cli#aio-appdeploy). I terminalen visar kommandot en URL för att komma åt det anpassade programmet. URL:en har formatet `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Använd kommandot [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action) om du vill hämta samma URL utan att omdistribuera programmet.

Använd URL:en i en [Bearbetningsprofil i [!DNL Experience Manager] som en [!DNL Cloud Service]](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use) för att integrera ditt program med [!DNL Experience Manager] som en [!DNL Cloud Service].

Kontrollera att ditt App Builder-projekt och din arbetsyta motsvarar [!DNL Experience Manager] som en [!DNL Cloud Service]-miljö där du vill använda åtgärden. Den har olika miljöer för utveckling, staging och produktion. Du kan verifiera miljön genom att kontrollera inloggningsuppgifterna för `AIO_runtime_*` som är definierade i ENV-filen i Adobe Developer App Builder-programmets rot. Om du till exempel vill distribuera till en `Stage`-arbetsyta har `AIO_runtime_namespace` formatet `xxxxxx_xxxxxxxxx_stage`. Om du vill integrera med [!DNL Experience Manager] som en [!DNL Cloud Service]-produktionsmiljö använder du program-URL:er från din Adobe Developer App Builder `Production`-arbetsyta.

>[!CAUTION]
>
>Använd inte en personlig arbetsyta i viktiga [!DNL Experience Manager]-miljöer.

>[!MORELIKETHIS]
>
>* [Förstå och hantera miljöer i [!DNL Experience Manager] som en [!DNL Cloud Service]](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/manage-environments).
