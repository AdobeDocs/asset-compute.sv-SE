---
title: Arkitektur för  [!DNL Asset Compute Service]
description: Hur [!DNL Asset Compute Service] API, program och SDK fungerar tillsammans för att tillhandahålla en molnbaserad resurshanteringstjänst.
exl-id: 658ee4b7-5eb1-4109-b263-1b7d705e49d6
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '478'
ht-degree: 0%

---

# Arkitektur för [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] är byggd ovanpå den serverlösa Adobe [!DNL `I/O Runtime`]-plattformen. Det ger stöd för Adobe Sensei innehållstjänster för resurser. Den anropande klienten (endast [!DNL Experience Manager] som [!DNL Cloud Service] stöds) får den Adobe Sensei-genererade information som den sökte efter resursen. Den returnerade informationen är i JSON-format.

[!DNL Asset Compute Service] kan utökas genom att skapa anpassade program baserade på [!DNL Adobe Developer App Builder]. Dessa anpassade program är [!DNL Project Adobe Developer App Builder] headless-appar och utför uppgifter som att lägga till anpassade konverteringsverktyg eller anropa externa API:er för att utföra bildåtgärder.

[!DNL Project Adobe Developer App Builder] är ett ramverk för att skapa och distribuera anpassade webbprogram på Adobe [!DNL `I/O Runtime`]. Om du vill skapa anpassade program kan utvecklarna använda [!DNL React Spectrum] (Adobe UI-verktygslådan), skapa mikrotjänster, skapa anpassade händelser och ordna API:er. Se [dokumentation om Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/overview).

Arkitekturen bygger på följande:

* Programmen har en modularitet som bara innehåller det som behövs för en viss uppgift och gör det möjligt att koppla ihop program från varandra och hålla dem lätta.

* Det serverlösa konceptet [!DNL Adobe I/O] Runtime ger många fördelar: asynkron, mycket skalbar, isolerad, jobbbaserad bearbetning, som passar perfekt för resursbearbetning.

* Binär molnlagring innehåller de funktioner som krävs för att lagra och komma åt resursfiler och återgivningar individuellt, utan att fullständiga behörigheter krävs för lagringen, med hjälp av försignerade URL-referenser. Överföringsacceleration, CDN-cachning och samlokalisering av datorprogram med molnlagring ger optimal åtkomst till innehåll med låg latens. Både AWS- och Azure-moln stöds.

![Arkitektur för Asset compute-tjänsten](assets/architecture-diagram.png)

*Bild: Arkitekturen för [!DNL Asset Compute Service] och hur den integreras med [!DNL Experience Manager], lagringsutrymme och bearbetningsprogram.*

Arkitekturen består av följande delar:

* **Ett API- och orkestreringslager** tar emot begäranden (i JSON-format) som instruerar tjänsten att omvandla en källresurs till flera renderingar. Förfrågningarna är asynkrona och returneras med ett aktiverings-ID som är jobb-ID. Instruktionerna är enbart deklarativa och för allt standardbearbetningsarbete (till exempel generering av miniatyrer, textrahering) anger konsumenterna bara önskat resultat, men inte de program som hanterar vissa återgivningar. Allmänna API-funktioner som autentisering, analys och hastighetsbegränsning hanteras med Adobe API Gateway framför tjänsten och hanterar alla begäranden som går till [!DNL Adobe I/O] Runtime. Programdirigeringen görs dynamiskt av orkestreringslagret. Klienterna definierar anpassade program för särskilda renderingar, som levereras med en egen uppsättning unika parametrar. Programkörningen kan vara helt parallell eftersom de är separata serverlösa funktioner i Adobe [!DNL `I/O Runtime`].

* **Program för att bearbeta resurser** som är specialiserade på vissa typer av filformat eller målåtergivningar. Ett program fungerar ungefär som UNIX® pipe-konceptet: en indatafil omvandlas till en eller flera utdatafiler.

* **Ett [vanligt programbibliotek](https://github.com/adobe/asset-compute-sdk)** hanterar vanliga åtgärder. Du kan till exempel hämta källfilen, överföra återgivningar, felrapportering, skicka händelser och övervaka. Den här designen ser till att programutvecklingen förblir enkel, enligt det serverlösa konceptet, med interaktioner som är begränsade till det lokala filsystemet.

<!-- TBD:

* About the YAML file?
* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
