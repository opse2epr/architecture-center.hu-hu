---
title: Nagyvállalati – erőforrás-konzisztencia fejlődést szem előtt tartva
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Nagyvállalati – erőforrás-konzisztencia fejlődést szem előtt tartva
author: BrianBlanchard
ms.openlocfilehash: 9fc6ca015310c02338463d3c8aa53ddfc74699df
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899202"
---
# <a name="large-enterprise-resource-consistency-evolution"></a>Nagyvállalati: Erőforrás konzisztencia fejlődést szem előtt tartva

Ez a cikk a narratíva haladásával erőforrás konzisztencia-ellenőrzések ad hozzá a cégirányítási MVP alapvető fontosságú alkalmazásokhoz.

## <a name="evolution-of-the-narrative"></a>A narratíva fejlődésére

A felhő bevezetésének csapatok megfelelt a védett adatok áthelyezéséhez szükséges összes követelmény. Az ilyen alkalmazások SLA kötelezettségvállalásait pedig az üzleti és a támogatási szükségességét az IT-üzemeltetőktől származnak. Közvetlenül a munkatársaival, a két adatközpont migrálásáról, mögött több alkalmazás fejlesztői és BI Csapatok minden készen indítása az új megoldások éles környezetben. IT-üzemeltetők a felhőbeli műveletek gondolkodási az új és meglévő működési folyamatok gyorsan integrálni.

### <a name="evolution-of-current-state"></a>A jelenlegi állapotában a fejlődést szem előtt tartva

- Informatikai aktívan áthelyezése éles számítási feladatok, védett adatokkal az Azure-bA. Alacsony prioritású számítási feladatok számos éles forgalom szolgálnak ki. Többet is vágható keresztül, amint az IT-üzemeltetési a számítási feladatok készültségi bejelentkezik.
- Az alkalmazás fejlesztői csapatok készen áll az éles forgalmat.
- A BI-csapattól készen áll az előrejelzéseket és elemzéseket kaphat a műveletek a három üzleti egység futtató rendszerek integrálásához.

### <a name="evolution-of-the-future-state"></a>A jövőbeli állapot a fejlődést szem előtt tartva

- IT-üzemeltetők a felhőbeli műveletek gondolkodási az új és meglévő működési folyamatok gyorsan integrálni.

A jelenlegi és jövőbeli állapot módosítások elérhetővé, amely az új házirend-utasítások lesz szükség az új kockázatokat.

## <a name="evolution-of-tangible-risks"></a>A képzés résztvevői hasznos képességekkel kockázatok a fejlődést szem előtt tartva

**Üzleti megszakítás**: Van egy új platform okozó megakadás az üzleti szempontból kritikus üzleti folyamatok rejlő kockázatát. Az IT-üzemeltetői csapatnak, és a csapatok különböző felhőalapú felválthatók végrehajtása is viszonylag tapasztalatlan a felhőbeli műveletek. Ez növeli annak kockázatát, megszakítás és kell problémák elhárításáról és az irányadó.

Az üzleti kockázat több technikai kockázattal bővíthet:

- Helytelen igazítású működési folyamatok valamilyen okból kimaradás lép, amely nem észlelhető vagy nem szervizelt gyorsan vezethet.
- Külső behatolás vagy szolgáltatásmegtagadási támadások ellen előfordulhat, hogy egy üzleti megszakítás
- Kritikus fontosságú eszközök nem lehet, hogy megfelelően felderíthető és ezért nem megfelelően az működtetni.
- Előfordulhat, hogy a meglévő felügyeleti folyamatok nem támogatja felderítetlen vagy mislabeled eszközök.
- Az üzembe helyezett eszközök konfigurációs előfordulhat, hogy nem felelnek meg a teljesítmény követelményeinek.
- Naplózás előfordulhat, hogy nem lehet megfelelően rögzíti és, hogy a teljesítményproblémák szervizelési központosított.
- Helyreállítási házirendek is sikertelen, vagy a vártnál hosszabb időt vesz igénybe.
- Inkonzisztens központi telepítési folyamat, amely az adatszivárgás vagy -megszakítások kezelése vezethet biztonsági rések eredményezhet.
- Konfigurációs csúszásokat vagy kihagyott javítások nem kívánt biztonsági rések, így adatszivárgás vagy -megszakítások kezelése eredményezhet.
- Konfigurációs előfordulhat, hogy kényszeríti ki a meghatározott SLA követelményeinek, vagy lekötött recovery követelményeinek.
- Telepített operációs rendszerek vagy alkalmazások előfordulhat, hogy teljesíti az operációs rendszer és alkalmazás követelmények megerősítése.
- Nincs több csapat dolgozik a felhőben miatt inkonzisztencia veszélye.

## <a name="evolution-of-the-policy-statements"></a>A házirend-utasítások fejlődésére

Az alábbi eltérésekkel házirend segítségével csökkentheti a kockázatokat és útmutató új végrehajtása. A lista megjelenését hosszú, de az ezek a házirendek bevezetését könnyebb, mint akkor jelenik meg.

1. Az összes telepített erőforrás kell lennie az alapján kategorizáljuk, kritikusság és az adatok besorolás. Besorolások vannak, át kell néznie a felhő Cégirányítási csapatot és az alkalmazás tulajdonosa, a felhőben való üzembe helyezés előtt.
2. Alapvető fontosságú alkalmazásokat tartalmazó alhálózatokhoz kell védeni a tűzfal megoldás képes a behatolások észlelése és elhárítása a támadásokkal szemben.
3. Irányítás azokat az eszközöket kell naplózása és az alapvető biztonsági csoport által definiált hálózati konfigurációkra kényszerítése.
4. Cégirányítási eszközök ellenőrizni kell, hogy az összes eszköz kapcsolatos alapvető fontosságú alkalmazásokhoz, vagy a védett adatok szerepelnek megfogyatkoznak erőforrás és optimalizálás figyelése.
5. Cégirányítási eszközök ellenőrizni kell, hogy a naplózási adatok megfelelő szintű az összes alapvető fontosságú alkalmazáshoz összegyűjtött vagy a védett adatok.
6. Cégirányítási folyamat ellenőrizni kell, hogy a biztonsági mentési, helyreállítási és SLA betartása megfelelően megvalósított alapvető fontosságú alkalmazások és a védett adatok.
7. Cégirányítási eszközök által jóváhagyott lemezképeket csak a virtuális gépek üzembe helyezése kell korlátozni.
8. Irányítás azokat az eszközöket ki kell kényszerítenie, hogy vannak-e az automatikus frissítések **. Ebben az esetben** az összes üzembe helyezett eszközök, amelyek támogatják az alapvető fontosságú alkalmazásokhoz. Szabálysértések lehet tekintse át a felügyeleti csoportokhoz, és operations házirendek megfelelően javítja. Eszközök, amelyek nem frissülnek automatikusan a tulajdonosa az IT-üzemeltetési folyamatok szerepelnie kell.
9. Cégirányítási eszközök ellenőrizni kell a címkézés kapcsolatos költségek, kritikusság, SLA-t, alkalmazások és adatok besorolása. Minden érték igazítás kell a felhő Cégirányítási csapata által kezelt előre meghatározott értékeket.
10. Cégirányítási folyamatok naplózások üzembehelyezési ponton kell megadni, és az összes eszköz közötti összhangot rendszeres ciklusokat, tartalmaznia kell.
11. Trendek és az azokat kihasználó támadások ellen, amelyek hatással lehetnek a magánfelhők számára meg kell vizsgálni rendszeresen a biztonsági csapat számára a felhőben használt alapvető biztonsági eszközök által.
12. Kiadási éles környezetbe, mielőtt az összes alapvető fontosságú alkalmazások és a védett adatok hozzá kell adni a kijelölt működési figyelési megoldáshoz. A kiválasztott IT-üzemeltetési eszközök által nem felderített eszközök nem oldhatók fel éles környezetben való használatra. A megfelelő központi telepítési folyamat eszközök lesz észlelhető a későbbiekben biztosítása érdekében, hogy az eszközök felderíthető szükséges módosításokat kell bocsátani.
13. Felderítés, hogy az eszköz méretezési az lehet érvényesíteni a felügyeleti csoportok ellenőrzése, hogy az eszköz megfelel-e a teljesítményre vonatkozó követelmények.
14. Központi telepítési eszközkészlet jóvá kell hagynia a felhő Cégirányítási csapatától folyamatos irányítás az üzembe helyezett eszközök.
15. Üzembehelyezési szkriptek fenn kell tartani a központi tárházban elérhető-e a felhő Cégirányítási csapat rendszeres áttekintése és a naplózási.
16. Cégirányítási felülvizsgálati folyamatok ellenőrizni kell, hogy üzembe helyezett eszközök megfelelően legyenek konfigurálva SLA-t és a helyreállítási követelményekkel összhangban.

## <a name="evolution-of-the-best-practices"></a>Ajánlott eljárást a fejlődést szem előtt tartva

A cikk ezen szakasza a cégirányítási MVP megtervezése és az Azure új szabályzatokat és az Azure Cost Management implementációja fejlődik. Két kialakítási módosítások együtt, a vállalati szabályzat új utasításokat fog teljesítéséhez.

Ebben a példában kitalált élménye alábbi feltételezzük, hogy a védett adatok fejlődést szem előtt tartva már elvégzett. Épület, ajánlott eljárások használatával, a következő hozzáadja readying egy előfizetést az üzletmenet szempontjából kritikus alkalmazások működési a figyelési követelmények.

**Vállalati IT-előfizetés**: Adja hozzá a következő, a vállalati IT-előfizetés, amely a kiszolgálókezelési.

1. Külső függőségként, a felhőalapú üzemeltetési csapat kell megadni a működési figyelési eszközök, üzleti folytonosság és vész-helyreállítási (BCDR) azokat az eszközöket és automatizált szervizelési hibakeresését. A felhő Cégirányítási csapat használatával majd támogatja a szükséges felderítési folyamat.
    1. Az ezt a használati esetet a felhőalapú üzemeltetési csapat választotta, az Azure Monitor figyelési alapvető fontosságú alkalmazások esetén az elsődleges eszközként.
    2. A csapat az Azure Site Recovery is választotta, az elsődleges BCDR-eszközök.
2. Az Azure Site Recovery-megvalósítás
    1. Meghatározhatja, és az Azure Vault a biztonsági mentési és helyreállítási folyamatok
    2. Hozzon létre egy Azure Resource Management-sablon, a tároló létrehozásához az egyes előfizetésekben
3. Az Azure Monitor végrehajtása
    1. Alapvető fontosságú előfizetés azonosítja, miután egy log analytics-munkaterület is létrehozható PowerShell-lel. Ez a központi telepítés előtti folyamat.

**Az egyes felhőalapú bevezetési előfizetés**: A következő biztosítja, hogy minden egyes előfizetés-e a megoldás által felderíthető és készen áll a BCDR-eljárások szerepelnek.

1. Az Azure Policy az alapvető fontosságú csomópont
    1. Naplózási, és csak a standard szintű szerepköreit kényszerítése.
    2. Naplózási és alkalmazás az összes storage-fiók titkosítás kényszerítésére.
    3. Naplózási és kényszerítése jóváhagyott hálózati alhálózat és virtuális hálózatok száma hálózati adapterenként.
    4. Naplózási és a felhasználó által megadott útvonaltáblák korlátozás kényszerítése.
    5. Naplózási és a Windows és Linux rendszerű virtuális gépekhez a Log Analytics-ügynökök telepítésének kényszerítéséhez.
2. Az Azure tervezet
    1. Hozzon létre egy nevű tervezet `mission-critical-workloads-and-protected-data`. Ez a megoldás eszközök mellett a védett adatok tervezet érvényes lesz.
    2. Adja hozzá az új Azure-házirendeket a tervezethez.
    3. A tervezet vonatkoznak minden olyan kritikus fontosságú alkalmazások üzemeltetéséhez várt előfizetésből.

## <a name="conclusion"></a>Összegzés

Ezeket a folyamatokat és az MVP segít mérsékelni az számos olyan kockázatok cégirányítási módosításai társított erőforrás-szabályozás hozzáadása. Együttesen adnak hozzá a helyreállítás, teljesítményalapú méretezést, illetve biztosítson hatékony eszközöket a felhő-kompatibilis műveleteket a szükséges ellenőrzések figyelése.

## <a name="next-steps"></a>További lépések

Felhőre való áttérés folyamatosan fejlődnek, és további üzleti értéket, a kockázatokat és a felhő cégirányítási kell is fejlődik. Tartson a folyamatban lévő fiktív vállalat a következő trigger akkor, ha a méretezési csoport a központi telepítés meghaladja a felhőbe 1000 eszközök vagy havi kiadások meghaladja a 10 000 USD havonta. Ezen a ponton a felhő Cégirányítási csapat hozzáadja a Cost Management szabályozza.

> [!div class="nextstepaction"]
> [A Cost Management fejlődést szem előtt tartva](./cost-management-evolution.md)