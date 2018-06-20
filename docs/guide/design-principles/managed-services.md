---
title: Használjon felügyelt szolgáltatásokat
description: Amikor lehetséges, a szolgáltatásként nyújtott infrastruktúra (IaaS) helyett használjon szolgáltatásként nyújtott platformot (PaaS)
author: MikeWasson
ms.openlocfilehash: 6d3cfb2e97b5a9b25bb1afd72059e981ef45c0d8
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206939"
---
# <a name="use-managed-services"></a>Használjon felügyelt szolgáltatásokat

## <a name="when-possible-use-platform-as-a-service-paas-rather-than-infrastructure-as-a-service-iaas"></a>Amikor lehetséges, a szolgáltatásként nyújtott infrastruktúra (IaaS) helyett használjon szolgáltatásként nyújtott platformot (PaaS)

Az IaaS olyan, mintha egy doboznyi alkatrész lenne. Bármit létrehozhat, de saját magának kell összeállítania. A felügyelt szolgáltatások konfigurálása és felügyelete egyszerűbb. Nem kell virtuális gépeket üzembe helyeznie, virtuális hálózatokat beállítania, javításokat és frissítéseket kezelnie, illetve egyéb olyan feladatokat elvégeznie, amelyek a szoftverek virtuális gépen való futtatásához kapcsolódnak.

Tegyük fel, hogy az alkalmazásnak üzenetsorra van szüksége. Beállíthat saját üzenetküldési szolgáltatást egy virtuális gépen a RabbitMQ vagy hasonló szolgáltatás segítségével. Az Azure Service Bus azonban már biztosít egy megbízható üzenetküldési szolgáltatást, és a beállítása is egyszerűbb. Egyszerűen hozzon létre egy Service Bus-névteret (ez az üzembehelyezési szkripttel is elvégezhető), majd hívja meg a Service Bust az ügyfél SDK használatával. 

Természetesen az alkalmazásnak lehetnek konkrét követelményei, amelyek miatt az IaaS módszer megfelelőbb lehet. Azonban még az IaaS-alapú alkalmazások esetén is érdemes olyan helyeket keresnie, ahol természetes lehet a felügyelt szolgáltatások használata. Ezek közé tartoznak az üzenetsorok, a gyorsítótár és az adattárolás.

| Az alábbiak futtatása helyett... | Fontolja meg a következőket... |
|-----------------------|-------------|
| Active Directory | Active Directory Domain Services |
| Elasticsearch | Azure Search |
| Hadoop | HDInsight |
| IIS | App Service |
| MongoDB | Cosmos DB |
| Redis | Azure Redis gyorsítótár |
| SQL Server | Azure SQL Database |


