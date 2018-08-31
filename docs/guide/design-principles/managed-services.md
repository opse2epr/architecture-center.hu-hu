---
title: Használjon felügyelt szolgáltatásokat
description: Amikor lehetséges, a szolgáltatásként nyújtott infrastruktúra (IaaS) helyett használjon szolgáltatásként nyújtott platformot (PaaS)
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: f6777a19e126a8a7f64be05dfad9bc503d27b1c3
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 08/31/2018
ms.locfileid: "43325774"
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


