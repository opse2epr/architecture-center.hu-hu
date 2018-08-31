---
title: N szintű architektúrastílus
description: Ismerteti az Azure N szintű architektúráinak előnyeit, kihívásait és ajánlott eljárásait
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: 2a113cefec8bd1c6c524030fbc459851094c09d6
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 08/31/2018
ms.locfileid: "43325750"
---
# <a name="n-tier-architecture-style"></a>N szintű architektúrastílus

Az N szintű architektúra az alkalmazást **logikai rétegekre** és **fizikai szintekre** osztja fel. 

![](./images/n-tier-logical.svg)

A rétegek a felelősségi körök különválasztására és a függőségek kezelésére használhatók. Minden réteghez egy adott felelősségi kör tartozik. A magasabb rétegek használhatják az alsóbb rétegek szolgáltatásait, de ez fordítva nem lehetséges. 

A szintek fizikailag vannak elkülönítve, és külön gépeken futnak. Egy szint közvetlenül is hívhat egy másik szintet, de aszinkron üzenetkezelést (üzenet-várólistát) is használhat. Az egyes rétegeket üzemeltethetik ugyan a saját szintjeik, de ez nem feltétlenül szükséges. Több réteg is üzemelhet ugyanazon a szinten. A szintek fizikai elválasztása fokozza a méretezhetőséget és a rugalmasságot, de a fokozott hálózati kommunikáció miatt késleltetéssel jár. 

Egy hagyományos háromszintű alkalmazás egy bemutatási, egy középső és egy adatbázisszintből áll. A középső szint használata nem kötelező. Az összetettebb alkalmazások háromnál több szinttel is rendelkezhetnek. A fenti ábrán egy alkalmazás látható két középső szinttel, amelyek különböző funkcióterületeket képviselnek. 

Egy N szintű alkalmazás **zárt rétegű architektúrával** vagy **nyitott rétegű architektúrával** rendelkezhet:

- Zárt rétegű architektúrában egy réteg csak a közvetlenül alatta lévő réteget hívhatja. 
- Nyitott rétegű architektúrában egy réteg az alatta lévők bármelyikét hívhatja. 

A zárt architektúrák korlátozzák a rétegek közötti függőségeket, de előfordulhat, hogy szükségtelen hálózati forgalmat hoznak létre, ha egy réteg egyszerűen továbbítja a kérelmeket a következő réteg felé. 

## <a name="when-to-use-this-architecture"></a>Mikor érdemes ezt az architektúrát használni?

Az N szintű architektúrákat általában szolgáltatásként nyújtott infrastruktúraként (IaaS) implementálják, ahol minden szint külön virtuálisgép-csoportokon fut. Az N szintű alkalmazásokat azonban nem feltétlenül kell tiszta IaaS-ként megvalósítani. Sokszor célszerű az architektúrára bizonyos részeihez – különösen a gyorsítótárazáshoz, az üzenetküldéshez és az adattároláshoz – felügyelt szolgáltatásokat használni.

Fontolja meg az N szintű architektúra használatát:

- Egyszerű webalkalmazások esetén. 
- Helyszíni alkalmazások Azure-ba történő migrálásakor, ha csak minimális mértékű újrabontásra van szükség.
- Helyszíni és felhőalapú alkalmazások egységesített fejlesztésekor.

Az N szintű architektúrákat a leggyakrabban hagyományos helyszíni alkalmazások használják, ezért ideális megoldást jelentenek meglévő számítási feladatok Azure-ba történő migrálásához.

## <a name="benefits"></a>Előnyök

- Hordozhatóság a felhőalapú és helyszíni alkalmazások, valamint a felhőplatformok között.
- A legtöbb fejlesztő gyorsan megtanulja a használatát.
- Természetes továbblépést jelent a hagyományos alkalmazásmodellből.
- Nyitott a heterogén környezetek (Windows/Linux) számára.

## <a name="challenges"></a>Problémák

- Könnyen előfordulhat, hogy a kialakított középső szint csakis CRUD-műveleteket végez az adatbázison, így további késleltetést okoz anélkül, hogy hasznos munkát végezne. 
- A monolitikus kialakítás megakadályozza a szolgáltatások független üzembe helyezését.
- Egy IaaS-alkalmazás kezelése több munkával jár, mint egy kizárólag felügyelt szolgáltatásokat használó alkalmazásé. 
- A nagyobb rendszerekben pedig a hálózati biztonság kezelése is nehézségeket okozhat.

## <a name="best-practices"></a>Ajánlott eljárások

- Használja az automatikus skálázást a terhelésben bekövetkező változások kezelésére. Lásd az [automatikus skálázással kapcsolatos ajánlott eljárásokat][autoscaling].
- Aszinkron üzenetkezeléssel különítse el a szinteket.
- Gyorsítótárazza a félig statikus adatokat. Lásd a [gyorsítótárazás ajánlott eljárásait][caching].
- Konfigurálja az adatbázisszintet magas rendelkezésre álláshoz olyan megoldások használatával, amilyenek például az [SQL Server Always On rendelkezésre állási csoportok][sql-always-on].
- Helyezzen el webalkalmazási tűzfalat (web application firewall, WAF) a kezelőfelület és az internet között.
- Minden szintet a saját alhálózatában helyezzen el, és használjon alhálózatokat biztonsági határként. 
- Korlátozza a hozzáférést az adatszinthez úgy, hogy csak a középső szint(ek)ről engedélyezi a kérelmeket.

## <a name="n-tier-architecture-on-virtual-machines"></a>N szintű architektúra virtuális gépeken

Ez a szakasz egy virtuális gépeken futó ajánlott N szintű architektúrát ismertet. 

![](./images/n-tier-physical.png)

Minden szint két vagy több virtuális gépből áll, amelyek egy rendelkezésre állási vagy virtuálisgép-méretezési csoportban vannak elhelyezve. A több virtuális gép rugalmasságot biztosít arra az esetre, ha az egyik leállna. Terheléselosztók segítségével oszthatók szét a kérelmek egy szint virtuális gépei között. A szintek vízszintesen skálázhatók, ha további virtuális gépeket ad hozzá a készlethez. 

Minden egyes szint a saját alhálózatában van elhelyezve, ami azt jelenti, hogy a belső IP-címeik azonos címtartományba esnek. Ez megkönnyíti a hálózati biztonsági csoportra (network security group, NSG) vonatkozó szabályok és az útválasztási táblázatok alkalmazását az egyes szintekre.

A webes és üzleti szintek állapot nélküliek. Bármelyik virtuális gép képes kezelni bármilyen, az adott szintre vonatkozó kérést. Az adatszintnek egy replikált adatbázisból kell állnia. Windows esetén az SQL Server Always On rendelkezésre állási csoportok használata javasolt a magas rendelkezésre állás érdekében. Linux esetén válasszon olyan adatbázist, amely támogatja a replikációt, például az Apache Cassandrát. 

A hálózati biztonsági csoportok (NSG-k) korlátozzák az egyes szintekhez való hozzáférést. Az adatbázisszint például csak az üzleti szintről való hozzáférést engedélyezi.

További részletekért és egy üzembe helyezhető Resource Manager-sablonért tekintse meg a következő referenciaarchitektúrákat:

- [Windows rendszerű virtuális gépek futtatása N szintű alkalmazáshoz][n-tier-windows]
- [Linux rendszerű virtuális gépek futtatása N szintű alkalmazáshoz][n-tier-linux]

### <a name="additional-considerations"></a>Néhány fontos megjegyzés

- Az N szintű architektúrák nem korlátozódnak három szintre. Összetettebb alkalmazások esetében gyakori, hogy további szinteket építenek ki. Ebben az esetben érdemes lehet 7-es szintű útválasztást használni annak érdekében, hogy a kérelmek egy adott szintre érkezzenek.

- A szintek méretezhetőségi, megbízhatósági és biztonsági határokat képeznek. Érdemes külön szinteket használni olyan szolgáltatásokhoz, amelyeket ezeken a területeken eltérő követelmények jellemeznek.

- Használjon virtuálisgép-méretezési csoportokat az automatikus skálázáshoz.

- Keresse meg azokat a helyeket az architektúrában, ahol egy felügyelt szolgáltatás jelentős újrabontás nélkül használható. Fordítson különös figyelmet a gyorsítótárazásra, az üzenetküldésre, a tárolásra és az adatbázisokra. 

- A nagyobb biztonság érdekében helyezzen hálózati DMZ-t az alkalmazás elé. A DMZ hálózati virtuális berendezéseket (network virtual appliance, NVA) tartalmaz, amelyek különböző biztonsági funkciókat implementálnak, például tűzfalakat és csomagvizsgálatot. További információkért lásd a [hálózati DMZ referenciaarchitektúráit][dmz].

- A magas rendelkezésre állás érdekében helyezzen két vagy több NVA-t egy rendelkezésre állási csoportba egy külső terheléselosztóval. Így eloszthatja az internetes kérelmeket a különböző példányokon. További információkért lásd a [magas rendelkezésre állású hálózati virtuális berendezések üzembe helyezésével][ha-nva] foglalkozó témakört.

- Ne engedélyezze a közvetlen RDP- vagy SSH-hozzáférést az alkalmazáskódot futtató virtuális gépekhez. Ehelyett tegye kötelezővé, hogy az operátorok bejelentkezzenek egy jumpboxba, vagyis bástyagazdagépbe. Ez egy, a hálózaton található virtuális gép, amelyet a rendszergazdák a többi virtuális géphez való kapcsolódásra használnak. A jumpbox rendelkezik NSG-vel, amely csak a jóváhagyott nyilvános IP-címekről való RDP- és SSH-kapcsolódást teszi lehetővé.

- Helyek közötti virtuális magánhálózat (VPN) vagy Azure ExpressRoute használatával kiterjesztheti az Azure-beli virtuális hálózatot a helyszíni hálózatra. További információkért lásd a [hibrid hálózatok referenciaarchitektúráját][hybrid-network].

- Ha a cég vagy intézmény Active Directoryt használ az identitások kezelésére, érdemes lehet kiterjeszteni az Active Directory-környezetet az Azure-beli virtuális hálózatra. További információkért lásd az [identitáskezelés referenciaarchitektúráját][identity].

- Ha magasabb rendelkezésre állás szükséges, mint amelyet az Azure SLA nyújt a virtuális gépek számára, replikálja az alkalmazást két régióban, a feladatátvételre pedig használja az Azure Traffic Managert. További információkért lásd a [Windows virtuális gépek több régióban történő futtatásával][multiregion-windows] vagy a [Linux virtuális gépek több régióban történő futtatásával][multiregion-linux] foglalkozó témakört.

[autoscaling]: ../../best-practices/auto-scaling.md
[caching]: ../../best-practices/caching.md
[dmz]: ../../reference-architectures/dmz/index.md
[ha-nva]: ../../reference-architectures/dmz/nva-ha.md
[hybrid-network]: ../../reference-architectures/hybrid-networking/index.md
[identity]: ../../reference-architectures/identity/index.md
[multiregion-linux]: ../../reference-architectures/virtual-machines-linux/multi-region-application.md
[multiregion-windows]: ../../reference-architectures/virtual-machines-windows/multi-region-application.md
[n-tier-linux]: ../../reference-architectures/virtual-machines-linux/n-tier.md
[n-tier-windows]: ../../reference-architectures/virtual-machines-windows/n-tier.md
[sql-always-on]: /sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server