---
title: Vállalati integráció üzenetsorokat és események használata
titleSuffix: Azure Reference Architectures
description: Javasolt architektúra megvalósítása az Azure Logic Apps, az Azure API Management, az Azure Service Bus és az Azure Event Grid, egy vállalati integrációs minta.
author: mattfarm
ms.reviewer: jonfan, estfan, LADocs
ms.topic: reference-architecture
ms.date: 12/03/2018
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: integration-services
ms.openlocfilehash: 4c9d2e201bcfc077990d746a1decd55ede2f220a
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299183"
---
# <a name="enterprise-integration-on-azure-using-message-queues-and-events"></a>Vállalati integráció az Azure-üzenetsorok és események használata

Ez a referenciaarchitektúra integrálható a vállalati háttérrendszerekhez, üzenetsorokat és események használata különítse el a szolgáltatásokat a nagyobb méretezhetőség és megbízhatóság. A háttérrendszerekhez szoftver feltétlenül tartalmazzák a szoftverszolgáltatások (SaaS) rendszerek, az Azure-szolgáltatásokkal, mint, és a vállalat meglévő webszolgáltatások.

![Az üzenetsorok és események használata vállalati integráció a referencia-architektúra](./_images/enterprise-integration-queues-events.png)

## <a name="architecture"></a>Architektúra

Itt látható architektúrában épül, amely egy egyszerűbb architektúra látható [alapszintű enterprise integration][basic-enterprise-integration]. Az architektúra használ [Logic Apps] [ logic-apps] szolgáltatásokat tartalmazó munkafolyamat és [az API Management] [ apim] katalógusok az API-k létrehozásához.

Ez a verzió, az architektúra két összetevőből, amelyek segítségével a rendszer több megbízható és méretezhető bővült:

- **[Az Azure Service Bus][service-bus]**. Service Bus az egy biztonságos, megbízható közvetítő.

- **[Az Azure Event Grid][event-grid]**. Event Grid-esemény-útválasztó szolgáltatás. Használja a [közzétételi/előfizetési](../../patterns/publisher-subscriber.md) (pub/sub) modellt alkalmaznak eseménykezelési.

Egy közvetítő aszinkron kommunikációt biztosít számos előnnyel jár a háttérszolgáltatásoknak küldött közvetlen, szinkron hívások keresztül:

- Terheléskiegyenlítés kezelni a számítási feladatok használatával adatlöketekkel biztosít a [terhelés üzenetsor-alapú terheléskiegyenlítési minta](../../patterns/queue-based-load-leveling.md).
- Megbízhatóan nyomon követi a hosszan futó munkafolyamatokat, több lépést vagy több alkalmazást érintő előrehaladását.
- Segítséget nyújt az alkalmazások szétválaszthatók.
- Integrálható a meglévő üzenet-alapú rendszerekhez.
- Lehetővé teszi a munkahelyi használatát a várólistára kerül, ha a háttérrendszer nem érhető el.

Event Grid lehetővé teszi, hogy a különböző összetevők reagálhat rájuk, azok a rendszer, nem pedig hagyatkoznia a lekérdezéses vagy ütemezett feladatokat. Csakúgy, mint egy üzenet-várólista nyújt segítséget az alkalmazások és szolgáltatások szétválaszthatók. Egy alkalmazás vagy szolgáltatás tehetik közzé az eseményeket, és bármely érdekelt előfizetők értesítést fog kapni. A küldő frissítése nélkül adhat hozzá új előfizetők számára.

Sok Azure-szolgáltatás támogatja az Event Gridbe küldő eseményeket. Például egy logikai alkalmazást egy eseményt, amikor új fájlokat adnak hozzá egy blob tároló figyelheti. Ez a minta lehetővé teszi a reaktív munkafolyamatok, ahol fájlt feltölteni, vagy egy üzenetet elhelyezése az üzenetsorban elindít egy sorozatát folyamatokat. A folyamatok előfordulhat, hogy hajtható végre párhuzamosan, vagy meghatározott sorrendben.

## <a name="recommendations"></a>Javaslatok

A javaslatok ismertetett [alapszintű enterprise integration] [ basic-enterprise-integration] ebben az architektúrában a alkalmazni. Az alábbi javaslatokat is érvényesek:

### <a name="service-bus"></a>Service Bus

A Service Bus rendelkezik kétféle kézbesítési *lekéréses* vagy *leküldéses*. A lekéréses modell a fogadó folyamatosan üzeneteket kérdez le új. Lekérdezés nem elég hatékony, lehet, főleg, ha sok várólisták, hogy minden egyes néhány üzenetek fogadása, vagy ha nincs sok időt közötti üzenetek. A leküldéses modell, a Service Bus Event Griden keresztül eseményt küld, ha új üzenetek vannak. A fogadó feliratkozik az eseményre. Az esemény kiváltásakor a fogadó a következő köteg az üzenetek a Service Bus kér le.

Amikor létrehoz egy logikai alkalmazást a Service Bus-üzenetek lefoglalhatja, az Event Grid integráció a leküldéses modell használatát javasoljuk. Ez a legtöbbször további költséghatékony megoldás, mert a logikai alkalmazás nem kell lekérdeznie, a Service Bus. További információkért lásd: [Azure Service Bus – Event Grid integráció áttekintése](/azure/service-bus-messaging/service-bus-to-event-grid-integration-concept). Jelenleg a Service Bus [prémium szintű](https://azure.microsoft.com/pricing/details/service-bus/) az Event Grid értesítések megadása kötelező.

Használat [PeekLock](/azure/service-bus-messaging/service-bus-messaging-overview#queues) üzenetek csoportja eléréséhez. PeekLock használatakor a logikai alkalmazás lépésekkel végrehajtása vagy a megszakítása előtt minden üzenetet érvényesítéséhez. Ez a megközelítés véletlen üzenet adatvesztés elleni védelmet biztosít.

### <a name="event-grid"></a>Event Grid

Ha egy Event Grid-trigger akkor aktiválódik, azt jelenti, hogy *legalább egy* esemény történt. Tételezzük fel, ha egy logikai alkalmazást egy Event Grid-triggerek, a Service Bus-üzenet beolvasása, azt kell, hogy több üzenetet lehet rendelkezésre állnak.

Event Grid egy kiszolgáló nélküli modellt használ. A számlázás a műveletek (esemény végrehajtások) száma alapján lesz kiszámítva. További információkért lásd: [Event Grid díjszabási](https://azure.microsoft.com/pricing/details/event-grid/). Jelenleg nincsenek szint szempontok az Event Gridhez.

## <a name="scalability-considerations"></a>Méretezési szempontok

Magasabb szintű skálázhatóság érdekében a Service Bus prémium szintű horizontálisan üzenetkezelési egységek száma. Prémium szintű konfigurációk egy, kettő vagy négy üzenetkezelési egységre is rendelkezhet. A Service Bus méretezésével kapcsolatos további információkért lásd: [ajánlott eljárások a teljesítmény Service Bus-üzenetkezelés használatával](/azure/service-bus-messaging/service-bus-performance-improvements).

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Tekintse át az SLA-t az egyes szolgáltatásokhoz:

- [Az API Management szolgáltatásiszint-szerződés][apim-sla]
- [Event Grid szolgáltatásiszint-szerződés][event-grid-sla]
- [A Logic Apps szolgáltatásiszint-szerződés][logic-apps-sla]
- [Service Busra vonatkozó SLA][sb-sla]

Ahhoz, hogy feladatátvétel súlyos kimaradás során, vegye fontolóra geo-vészhelyreállítás a Service Bus prémium szintű. További információkért lásd: [Azure Service Bus geo-disaster recovery](/azure/service-bus-messaging/service-bus-geo-dr).

## <a name="security-considerations"></a>Biztonsági szempontok

A Service Bus biztonságos, használja a közös hozzáférésű jogosultságkód (SAS). Is hozzáférést biztosít egy felhasználó a megadott jogokat a Service Bus-erőforrások használatával [SAS hitelesítési](/azure/service-bus-messaging/service-bus-sas). További információkért lásd: [Service Bus-hitelesítés és engedélyezés](/azure/service-bus-messaging/service-bus-authentication-and-authorization).

Ha szeretne közzétenni egy HTTP-végpontot, Service Bus-üzenetsorba, például az új üzeneteket tehet közzé az API Management védheti a várólista fronting a végpont által. A végpont tanúsítványokat vagy OAuth-hitelesítés megfelelő majd gondoskodhat. A legegyszerűbben úgy, hogy a végpont biztonságos köztes egy logikai alkalmazást használ egy HTTP-kérés/válasz eseményindító.

Az Event Grid szolgáltatás egy érvényesítési kód eseménykézbesítés védi. Ha a Logic Apps használatával az események felhasználásához, érvényesítés automatikusan történik. További információkért lásd: [Event Grid biztonsági és hitelesítési](/azure/event-grid/security-authentication).

[apim]: /azure/api-management
[apim-sla]: https://azure.microsoft.com/support/legal/sla/api-management/
[event-grid]: /azure/event-grid/
[event-grid-sla]: https://azure.microsoft.com/support/legal/sla/event-grid
[logic-apps]: /azure/logic-apps/logic-apps-overview
[logic-apps-sla]: https://azure.microsoft.com/support/legal/sla/logic-apps
[sb-sla]: https://azure.microsoft.com/support/legal/sla/service-bus/
[service-bus]: /azure/service-bus-messaging/
[basic-enterprise-integration]: ./basic-enterprise-integration.md
