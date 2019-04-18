---
title: A mikroszolgáltatások referenciaimplementációja az Azure Kubernetes Service esetében
description: Ez a referenciaimplementáció a mikroszolgáltatási architektúra ajánlott eljárásait mutatja be
author: MikeWasson
ms.date: 02/26/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: 17e275e5b5f45233f7467192402cb28fce35c57b
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59068905"
---
# <a name="designing-a-microservices-architecture"></a>Mikroszolgáltatási architektúra tervezése

A mikroszolgáltatás egy népszerű architekturális stílussá vált a rugalmas, hatékonyan méretezhető, függetlenül üzembe helyezhető és gyorsan fejleszthető felhőalkalmazások létrehozásához. Azonban ahhoz, hogy ez több legyen, mint egy divatos kifejezés, a mikroszolgáltatások esetén új megközelítés szükséges az alkalmazások tervezésekor és létrehozásakor.

Ezekben a cikkekben megismerjük, hogyan hozhat létre és futtathat egy mikroszolgáltatási architektúrát az Azure-ban. A témakörök a következők:

- [Szolgáltatások közötti kommunikáció](./interservice-communication.md)
- [API-tervezés](./api-design.md)
- [API-átjárók](./gateway.md)
- [Adatkezelési szempontok](./data-considerations.md)
- [Tervezési minták](./patterns.md)

## <a name="prerequisites"></a>Előfeltételek

A cikkek elolvasása előtt a következővel kezdhet:

- [A mikroszolgáltatási architektúrák bemutatása](../introduction.md). Megismerheti a mikroszolgáltatások előnyeit és kihívásait, valamint azt, hogy mikor érdemes ezt az architektúrastílust használni.
- [Mikroszolgáltatások modellezése tartományelemzéssel](../model/domain-analysis.md). A mikroszolgáltatás-modellezés tartományvezérelt megközelítésének megismerése.

## <a name="reference-implementation"></a>Referenciaimplementáció

A mikroszolgáltatási architektúrák ajánlott eljárásainak illusztrálása érdekében létrehoztunk egy referenciaimplementációt, a Drone Delivery alkalmazást. Ez az implementáció az Azure Kubernetes Service (AKS) használatával fut a Kubernetes rendszeren. A referenciaimplementációt a [GitHubon][drone-ri] találja meg.

![A Drone Delivery alkalmazás architektúrája](../images/drone-delivery.png)

## <a name="scenario"></a>Forgatókönyv

A Fabrikam vállalat egy drónos szállítási szolgáltatást indít. A cég egy drónokból álló flottát kezel. A vállalkozások regisztrálnak a szolgáltatásban, és a felhasználók kérhetik egy termék drónos kézbesítését. Amikor az ügyfél ütemezi a felvételt, egy háttérrendszer hozzárendel egy drónt, és értesíti a felhasználót a várható szállítási időről. Amíg a kézbesítés folyamatban van, az ügyfél nyomon követheti a drón helyét, miközben a becsült érkezési időpont folyamatosan frissül.

Ez a forgatókönyv egy viszonylag összetett tartományt foglal magába. A vállalat számára problémát jelenthet a drónok ütemezése, a csomagok nyomon követése, a felhasználói fiókok felügyelete, valamint az előzményadatok tárolása és elemzése. Ezenkívül a Fabrikam gyorsan piacra szeretne lépni, majd gyors iterációkat kíván végezni, új funkciók és képességek hozzáadásával. Az alkalmazásnak felhőszinten kell üzemelnie, magas szolgáltatási szintű célkitűzéssel (service level objective, SLO). A Fabrikam emellett arra számít, hogy a rendszer különböző részeinek nagyon eltérő adattárolási és lekérdezési igényei lesznek. A megfontolandó szempontok alapján a Fabrikam azt a döntést hozta, hogy egy mikroszolgáltatási architektúrát választ a Drone Delivery alkalmazáshoz.

> [!NOTE]
> Amennyiben segítségre van szüksége a mikroszolgáltatási architektúra és egyéb architekturális stílusok közötti választáshoz, tekintse meg az [Azure-alkalmazásarchitektúrákhoz készült útmutatót](../../guide/index.md).

A referenciaimplementáció Kubernetest és [Azure Kubernetes Service-t](/azure/aks/) (AKS) használ. A magas szintű architekturális döntések és kihívások azonban minden tárolóvezénylőre érvényesek, beleértve az [Azure Service Fabricet](/azure/service-fabric/) is.

<!-- links -->

[drone-ri]: https://github.com/mspnp/microservices-reference-implementation/tree/v0.1.0-orig

## <a name="next-steps"></a>További lépések

> [!div class="nextstepaction"]
> [Számítási lehetőség kiválasztása](./compute-options.md)