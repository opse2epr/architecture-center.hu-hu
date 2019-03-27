---
title: Használjon felügyelt szolgáltatásokat
titleSuffix: Azure Application Architecture Guide
description: Ha lehetséges, használjon platformot (PaaS) szolgáltatás infrastruktúrán szolgáltatás (IaaS).
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: f08914e1eacc4f02fdb16093fe590f46249e3da3
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299657"
---
# <a name="use-managed-services"></a>Használjon felügyelt szolgáltatásokat

## <a name="when-possible-use-platform-as-a-service-paas-rather-than-infrastructure-as-a-service-iaas"></a>Amikor lehetséges, a szolgáltatásként nyújtott infrastruktúra (IaaS) helyett használjon szolgáltatásként nyújtott platformot (PaaS)

Az IaaS olyan, mintha egy doboznyi alkatrész lenne. Bármit létrehozhat, de saját magának kell összeállítania. A felügyelt szolgáltatások konfigurálása és felügyelete egyszerűbb. Nem kell virtuális gépeket üzembe helyeznie, virtuális hálózatokat beállítania, javításokat és frissítéseket kezelnie, illetve egyéb olyan feladatokat elvégeznie, amelyek a szoftverek virtuális gépen való futtatásához kapcsolódnak.

Tegyük fel, hogy az alkalmazásnak üzenetsorra van szüksége. Beállíthat saját üzenetküldési szolgáltatást egy virtuális gépen a RabbitMQ vagy hasonló szolgáltatás segítségével. Az Azure Service Bus azonban már biztosít egy megbízható üzenetküldési szolgáltatást, és a beállítása is egyszerűbb. Egyszerűen hozzon létre egy Service Bus-névteret (ez az üzembehelyezési szkripttel is elvégezhető), majd hívja meg a Service Bust az ügyfél SDK használatával.

Természetesen az alkalmazásnak lehetnek konkrét követelményei, amelyek miatt az IaaS módszer megfelelőbb lehet. Azonban még az IaaS-alapú alkalmazások esetén is érdemes olyan helyeket keresnie, ahol természetes lehet a felügyelt szolgáltatások használata. Ezek közé tartoznak az üzenetsorok, a gyorsítótár és az adattárolás.

| Az alábbiak futtatása helyett... | Fontolja meg a következőket... |
|-----------------------|-------------|
| Active Directory | Azure Active Directory tartományi szolgáltatások |
| Elasticsearch | Azure Search |
| Hadoop | HDInsight |
| IIS | App Service |
| MongoDB | Cosmos DB |
| Redis | Azure Redis Cache |
| SQL Server | Azure SQL Database |
