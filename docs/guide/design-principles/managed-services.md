---
title: Felügyelt szolgáltatásokkal
description: Ha lehetséges, használja a platform (PaaS) szolgáltatás infrastruktúrán szolgáltatásként (IaaS)
author: MikeWasson
layout: LandingPage
ms.openlocfilehash: 7156c073db3e047fb38e031309ddb637a9e44c02
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="use-managed-services"></a>Felügyelt szolgáltatásokkal

## <a name="when-possible-use-platform-as-a-service-paas-rather-than-infrastructure-as-a-service-iaas"></a>Ha lehetséges, használja a platform (PaaS) szolgáltatás infrastruktúra helyett szolgáltatásként (IaaS)

Infrastruktúra-szolgáltatási működik, mintha a kijelzők egy mezőt. Bármi hozhat létre, de saját kezűleg össze van. Felügyelt szolgáltatásainak használata könnyebben konfigurálásához és felügyeletéhez. Nem kell virtuális gépek kiépítése, a Vnetek beállítása, a javítások és frissítések, és az összes társított egy virtuális gépen futó szoftvereket a más terhelés kezelésére.

Tegyük fel, hogy az alkalmazás az üzenet-várólista kell. Beállíthat úgy a virtuális gép, a saját üzenetküldési szolgáltatás RabbitMQ hasonlót használatával. De Azure Service Bus már biztosít szolgáltatásként, megbízható üzenetküldés, és egyszerűbb beállítása. Hozzon létre egy Service Bus-névtér (amely a telepítési parancsfájl részeként végezhető), majd hívja Service Bus az ügyfél SDK használatával. 

Természetesen az alkalmazás konkrét követelmények, amelyek egy IaaS-módszert is megfelelő lehet, hogy rendelkezik. Azonban akkor is, ha az alkalmazás IaaS alapul, keressen helyen, ahol lehet természetes átfogó felügyelt szolgáltatásokat. Ezek közé tartozik a gyorsítótár, a sorokhoz és az adatok tárolására.

| Futtatása helyett... | Érdemes lehet... |
|-----------------------|-------------|
| Active Directory | Azure Active Directory tartományi szolgáltatások |
| Elasticsearch | Azure Search |
| Hadoop | HDInsight |
| IIS | App Service |
| MongoDB | Cosmos DB |
| Redis | Azure Redis Cache |
| SQL Server | Azure SQL Database |


