---
title: 'CAF: Biztonsági alapkonfiguráció mintautasítások házirend'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Biztonsági alapkonfiguráció mintautasítások házirend
author: BrianBlanchard
ms.openlocfilehash: ac40e022f8dff0c3021c04cd9d6ae42d28afaf1f
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899097"
---
# <a name="security-baseline-sample-policy-statements"></a>Biztonsági alapkonfiguráció mintautasítások házirend

Egyes felhőszolgáltatási házirend-utasítások szolgálnak, amelyeket a kockázati értékelés során azonosított adott kockázatkezelés. Ezek az utasítások kell biztosítania a kockázatok tömör összegzését, és azokkal a tervek. Minden egyes utasítás definíciójának tartalmaznia kell adatot megtalálhatja:

- Technikai kockázat – a kockázatokat, ez a szabályzat foglalkozni fog összegzését.
- Házirendutasítás – a házirend követelményeinek törölje összefoglaló leírását.
- Műszaki beállítások – gyakorlati ajánlásokat készítünk, előírásoknak vagy egyéb útmutatást, amely az IT-csoportoknak és fejlesztők is használhatják, ha a házirend megvalósítása.

A következő házirend mintautasítások cím számos gyakori üzleti biztonsági kockázatokat, és a példaként szolgálnak, hogy a tényleges házirend-utasítások saját szervezete igényei szerint címzés tervezése során hivatkozhat. Ezek a példák nem jelentenek a proscriptive kell, és potenciálisan több házirend-beállítások kezelésére vonatkozó bármely egyetlen azonosított kockázattal rendelkező. Szorosan együttműködnek az üzleti, biztonsági és informatikai csapata hozza létre a legjobb házirend megoldások azonosítását az adott biztonsági kockázatokat.  

## <a name="asset-classification"></a>Eszközbesorolás

**Technikai kockázati**: Eszközök nem megfelelően azonosítja, kritikus fontosságú vagy érintő bizalmas adatok nem jelenhet meg a megfelelő védelmet és esetleges adatszivárgások vagy üzleti megszakadását.

**Házirendutasítás**: Az összes telepített erőforrás kell lennie az alapján kategorizáljuk, kritikusság és az adatok besorolás. Besorolások kell vizsgálni a felhő Cégirányítási csapatot és az alkalmazás tulajdonosa, a felhőben való üzembe helyezés előtt.

**A lehetséges tervezési beállítás**: Létesíteni [szabványok címkézés erőforrás](../../decision-guides/resource-tagging/overview.md) , és győződjön meg, hogy az informatikai részleg alkalmazhatja őket módon történő használatával üzembe helyezett erőforrásokat [Azure-erőforráscímkék](/azure/azure-resource-manager/resource-group-using-tags).

## <a name="data-encryption"></a>Adattitkosítás

**Technikai kockázati**: Annak a kockázata, a védett adatok elérhetővé váljon tárolás közben.

**Házirendutasítás**: Védett adatok inaktív állapotban titkosítani kell.

**A lehetséges tervezési beállítás**: Tekintse meg a [Azure-titkosítás áttekintése](/azure/security/security-azure-encryption-overview) cikkben említett hogyan megy végbe az adatok inaktív adatok titkosítása az Azure platformon.  

## <a name="network-isolation"></a>A hálózatok elkülönítéséhez

**Technikai kockázati**: Hálózatok és alhálózatok belüli hálózatok közötti kapcsolatot a lehetséges biztonsági résekről, az adatszivárgás vagy járó üzleti szempontból alapvető fontosságú szolgáltatásokat is előfordulhat, hogy mutatja be.

**Házirendutasítás**: Alhálózatok, amely tartalmazza a védett adatok más alhálózatokkal elkülönítve kell lennie. Védett adatok az alhálózatok közötti hálózati forgalmat, hogy rendszeresen naplózni kell.

**A lehetséges tervezési beállítás**: Az Azure-ban, hálózat és alhálózat-elkülönítést kezelhető [Azure-beli virtuális hálózatok](/azure/virtual-network/virtual-networks-overview).

## <a name="secure-external-access"></a>Külső hozzáférés biztonságossá tétele

**Technikai kockázati**: Behatolás, adatok illetéktelen kitettség vagy üzleti megszakítás eredményez a kockázata, engedélyezi a hozzáférést a számítási feladatok a nyilvános interneten keresztül mutatja be.

**Házirendutasítás**: Nincs alhálózat, amely tartalmazza a védett adatok közvetlenül elérhető lesz nyilvános interneten keresztül vagy az adatközpontok között. Köztes alhálózati works keresztül hozzáférést ezekhez az alhálózatokhoz kell átirányítani. Ezekhez az alhálózatokhoz való hozzáférés teljes teljesítményű csomag ellenőrzésének és blokkoló funkciók képes tűzfal megoldás segítségével kell származnia.

**A lehetséges tervezési beállítás**: Az Azure-ban, biztonságos nyilvános végpontok üzembe helyezésével egy [szegélyhálózat (DMZ) között a nyilvános interneten és a felhő alapú hálózat](/azure/architecture/reference-architectures/dmz/secure-vnet-dmz).

## <a name="ddos-protection"></a>DDoS Protection

**Technikai kockázati**: Az elosztott szolgáltatásmegtagadási (DDoS-) támadásokat egy üzleti megszakítás eredményezhet.

**Házirendutasítás**: Helyezze üzembe az összes nyilvánosan elérhető hálózat végpontok automatizált DDoS veszélyelhárítási mechanizmust.

**A lehetséges tervezési beállítás**: Használat [Azure DDoS Protection](/azure/virtual-network/ddos-protection-overview) DDoS-támadások által okozott fennakadások minimalizálása érdekében.

## <a name="secure-on-premises-connectivity"></a>Biztonságos helyszíni kapcsolatok

**Technikai kockázati**: A felhő hálózata és a helyszínen a nyilvános interneten keresztül nem titkosított forgalmat téve a hozzáférés, kockázatra vonatkozó adatok bemutatása.

**Házirendutasítás**: Az összes kapcsolat a helyszíni és a felhő között hálózatokat kell végrehajtani egy biztonságos, titkosított VPN-kapcsolat vagy egy dedikált privát WAN-kapcsolaton keresztül.

**A lehetséges tervezési beállítás**: Az Azure-ban, használjon expressroute-on vagy az Azure VPN privát kapcsolatot a helyszíni és a felhő között hálózatok.

## <a name="network-monitoring-and-enforcement"></a>A hálózatfigyelés és végrehajtás

**Technikai kockázati**: Hálózati konfiguráció módosításait az új biztonsági rések és az adatok felfedésének kockázatát kockázatok vezethet.

**Házirendutasítás**: Irányítás azokat az eszközöket kell naplózása és az alapvető biztonsági csoport által definiált hálózati konfigurációkra kényszerítése.

**A lehetséges tervezési beállítás**: Az Azure-ban, hálózati tevékenység segítségével követhetők [Azure Network Watcher](/azure/network-watcher/network-watcher-monitoring-overview), és [az Azure Security Center](/azure/security-center/security-center-network-recommendations) segít azonosítani a biztonsági rések. Az Azure Policy lehetővé teszi, hogy korlátozzák a hálózati erőforrások és erőforrás-konfigurációs házirend szerint a biztonsági csapat által meghatározott.

## <a name="security-review"></a>Biztonsági felülvizsgálat

**Technikai kockázati**: Az idő múlásával új biztonsági fenyegetések és támadások típusok alakulnak ki, a kockázati kitettség vagy a felhőbeli erőforrások megszakadását növelése.

**Házirendutasítás**: Trendek és a potenciális biztonsági réseket, amelyek hatással lehetnek a magánfelhők számára meg kell vizsgálni rendszeresen a biztonsági csapat számára a felhőben használt alapvető biztonsági eszközök által.

**A lehetséges tervezési beállítás**: Rendszeres biztonsági felülvizsgálat felel meg, amely tartalmazza a vonatkozó létesíteni informatikai és cégirányítási csapat tagjai. Tekintse át a meglévő biztonsági adatok és metrikák létrehozására, a jelenlegi házirend és az eszközök alapvető biztonsági rések, és frissítse a szabályzat minden olyan új kockázatok csökkentése érdekében.

## <a name="next-steps"></a>További lépések

A minták segítségével a jelen cikkben említett kiindulási pontként házirendek fejlesztése, hogy cím konkrét biztonsági kockázatokat, amelyek igazodnak a cloud bevezetési terveket.

A saját egyéni házirend-utasítások kapcsolatos alapvető biztonsági fejlesztésének megkezdéséhez töltse le a [biztonsági alapterv sablon](template.md).

-K gyorsabb elfogadása, ezen a területen, válassza a [gyakorlatban hasznosítható Cégirányítási utazás](../journeys/overview.md) , hogy a legtöbb szorosan igazodik a környezetében. Ezután módosítsa a kialakítás révén az adott vállalati házirendet érintő döntések.

> [!div class="nextstepaction"]
> [Gyakorlatban hasznosítható Cégirányítási Journey](../journeys/overview.md)
