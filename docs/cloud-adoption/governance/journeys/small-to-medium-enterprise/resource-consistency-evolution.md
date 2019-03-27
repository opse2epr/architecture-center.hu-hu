---
title: 'CAF: Kis és közepes méretű vállalati – erőforrás-konzisztencia fejlődést szem előtt tartva'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Kis és közepes méretű vállalati – erőforrás-konzisztencia fejlődést szem előtt tartva
author: BrianBlanchard
ms.openlocfilehash: caacd62da9da526ec01ab935896598065f64d9d1
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299282"
---
# <a name="small-to-medium-enterprise-resource-consistency-evolution"></a>Small-to-medium enterprise: Az erőforrás-konzisztencia fejlődése

Ez a cikk a narratíva haladásával erőforrás konzisztencia vezérlőelemeket alapvető fontosságú alkalmazások támogatásához.

## <a name="evolution-of-the-narrative"></a>A narratíva fejlődésére

Új felhasználói élményt, új előrejelzési eszközök és az áttelepített infrastruktúra továbbra is halad. Az üzleti már készen áll a termelési kapacitásban azok az eszközök használatának megkezdéséhez.

### <a name="evolution-of-the-current-state"></a>Az aktuális állapot a fejlődést szem előtt tartva

Ez narratíva korábbi szakaszában az alkalmazások fejlesztését és BI Csapatok is majdnem készen áll a vevő és pénzügyi adatok integrálása éles számítási feladatokra. Az informatikai csapat a DR-adatközponthoz kivonás alatt volt.

Azóta, néhány dolog változott, amely hatással lesz a cégirányítási:

- Informatikai kivonta a DR adatközpont ütemezett 100 %-ban. A folyamat egy éles környezetben az adatközpontban eszközök száma meghatározott felhőbeli migrálás csatolásával.
- Most már készen áll az alkalmazás fejlesztői csapatok éles forgalom.
- A BI-csapattól készen áll az előrejelzési díjat és az operációs rendszereken vissza betekintést hírcsatorna éles környezetben az adatközpontban.

### <a name="evolution-of-the-future-state"></a>A jövőbeli állapot a fejlődést szem előtt tartva

Mielőtt Azure-környezetek éles üzleti folyamatokat használ, a felhőbeli műveletek részletes kell. Együtt irányítás érdekében továbbfejlesztett változata szükséges eszközök működtetik megfelelően kell biztosítani.

A jelenlegi és jövőbeli állapot módosítások elérhetővé, amely az új házirend-utasítások lesz szükség az új kockázatokat.

## <a name="evolution-of-tangible-risks"></a>A képzés résztvevői hasznos képességekkel kockázatok a fejlődést szem előtt tartva

**Üzleti megszakítás**: Van egy új platform okozó megakadás az üzleti szempontból kritikus üzleti folyamatok rejlő kockázatát. Az IT-üzemeltetői csapatnak, és a csapatok különböző felhőalapú felválthatók végrehajtása is viszonylag tapasztalatlan a felhőbeli műveletek. Ez növeli annak kockázatát, megszakítás és kell problémák elhárításáról és az irányadó.

Az üzleti kockázat bővíthet egy műszaki kockázatok száma:

- Külső behatolás vagy szolgáltatásmegtagadási támadások ellen előfordulhat, hogy egy üzleti megszakítás
- Kritikus fontosságú eszközök előfordulhat, hogy nem lehet megfelelően felderíteni, és emiatt előfordulhat, hogy nem megfelelően működik
- Előfordulhat, hogy a meglévő felügyeleti folyamatok nem támogatja felderítetlen vagy mislabeled eszközök.
- A konfigurációt a központilag telepített eszközök nem felel meg teljesítmény verziójával kapcsolatos elvárások
- Naplózás előfordulhat, hogy nem lehet megfelelően rögzíti és, hogy a teljesítményproblémák szervizelési központosított.
- Helyreállítási házirendek is sikertelen, vagy a vártnál hosszabb időt vesz igénybe.
- Inkonzisztens központi telepítési folyamat, amely az adatszivárgás vagy -megszakítások kezelése vezethet biztonsági rések eredményezhet.
- Konfigurációs csúszásokat vagy kihagyott javítások nem kívánt biztonsági rések, így adatszivárgás vagy -megszakítások kezelése eredményezhet.
- Konfigurációs előfordulhat, hogy kényszeríti ki a meghatározott SLA követelményeinek, vagy lekötött recovery követelményeinek.
- Korlátozási követelményeinek sikertelen lehet a telepített operációs rendszerek vagy alkalmazások
- Sok csapatok számára a felhőben dolgozik nincs inkonzisztencia veszélye.

## <a name="evolution-of-the-policy-statements"></a>A házirend-utasítások fejlődésére

Az alábbi eltérésekkel házirend segítségével csökkentheti a kockázatokat és útmutató új végrehajtása. A lista hosszú úgy tűnik, de ezek a házirendek bevezetésének lehet könnyebb, mint a megjelenik.

1. Az összes telepített erőforrás kell lennie az alapján kategorizáljuk, kritikusság és az adatok besorolás. Besorolások vannak, át kell néznie a felhő Cégirányítási csapatot és az alkalmazás tulajdonosa, a felhőben való üzembe helyezés előtt
2. Alapvető fontosságú alkalmazásokat tartalmazó alhálózatokhoz kell védeni a tűzfal megoldás képes a behatolások észlelése és elhárítása a támadásokkal szemben.
3. Irányítás azokat az eszközöket kell naplózása és érvényesíti a hálózati konfigurációs követelményei határozzák meg a biztonsági csapat
4. Cégirányítási eszközök ellenőrizni kell, hogy az összes eszköz kapcsolatos alapvető fontosságú alkalmazások vagy a védett adatok szerepelnek megfogyatkoznak erőforrás és optimalizálás figyelése.
5. Cégirányítási eszközök ellenőrizni kell, hogy a naplózási adatok megfelelő szintű az összes alapvető fontosságú alkalmazáshoz összegyűjtött vagy a védett adatok.
6. Cégirányítási folyamat ellenőrizni kell, hogy a biztonsági mentési, helyreállítási és SLA betartása megfelelően megvalósított alapvető fontosságú alkalmazások és a védett adatok.
7. Cégirányítási eszközök által jóváhagyott lemezképeket csak a virtuális gépi környezetekben kell korlátozni.
8. Irányítás azokat az eszközöket ki kell kényszerítenie, hogy az automatikus frissítések. Ebben az esetben az összes üzembe helyezett eszközök, amelyek támogatják az alapvető fontosságú alkalmazásokhoz. Szabálysértések lehet tekintse át a felügyeleti csoportokhoz, és operations házirendek megfelelően javítja. Eszközök, amelyek nem frissülnek automatikusan a tulajdonosa az IT-üzemeltetési folyamatok szerepelnie kell.
9. Cégirányítási eszközök ellenőrizni kell a címkézés kapcsolatos költségek, kritikusság, SLA-t, alkalmazások és adatok besorolása. Előre meghatározott értékeket a cégirányítási csapata által kezelt összes értéket kell igazítás.
10. Cégirányítási folyamatok naplózások üzembehelyezési ponton kell megadni, és az összes eszköz közötti összhangot rendszeres ciklusokat, tartalmaznia kell.
11. Trendek és az azokat kihasználó támadások ellen, amelyek hatással lehetnek a magánfelhők számára meg kell vizsgálni rendszeresen a biztonsági csapat számára a felhőben használt biztonsági eszközök által.
12. Mielőtt éles környezetben az összes alapvető fontosságú alkalmazások és a védett adatok hozzá kell adni a kijelölt működési figyelési megoldáshoz. Éles környezetben való használatra, amelyek nem azok észleljék a kiválasztott eszközöket, IT-üzemeltetési eszközökre nem oldhatók fel. A megfelelő központi telepítési folyamat eszközök lesz észlelhető a későbbiekben biztosítása érdekében, hogy az eszközök felderíthető szükséges módosításokat kell bocsátani.
13. Esetén a felderítés a felügyeleti csoportok méretezés lesz annak érdekében, hogy az eszközök hálózatiteljesítmény-igények kielégítéséhez, adategységek
14. Központi telepítési eszközkészlet jóvá kell hagynia a felhő Cégirányítási csapatától folyamatos irányítás az üzembe helyezett eszközök.
15. Üzembehelyezési szkriptek kell fenntartani egy központi tárházban elérhető-e a felhő Cégirányítási csapat rendszeres áttekintése és a naplózási.
16. Cégirányítási felülvizsgálati folyamatok ellenőrizni kell, hogy üzembe helyezett eszközök megfelelően legyenek konfigurálva SLA-t és a helyreállítási követelményekkel összhangban.

## <a name="evolution-of-the-best-practices"></a>Ajánlott eljárást a fejlődést szem előtt tartva

A cikk ezen szakasza a cégirányítási MVP megtervezése és az Azure új szabályzatokat és az Azure Cost Management implementációja fejlődik. Két kialakítási módosítások együtt, a vállalati szabályzat új utasításokat fog teljesítéséhez.

1. A felhőalapú üzemeltetési csapat működési figyelési eszközök meghatározzuk, és automatizált szervizelési azokat az eszközöket. A felhő Cégirányítási csapat támogatni fogja a felderítési folyamatokat. Az ezt a használati esetet a felhőalapú üzemeltetési csapat választotta, az Azure Monitor figyelési alapvető fontosságú alkalmazások esetén az elsődleges eszközként.
2. Hozzon létre egy tárház Azure DevOps áruházbeli és verziónak megfelelő Resource Manager-sablonok és a parancsfájlalapú konfigurációk.
3. Az Azure Vault végrehajtása:
    1. Meghatározhatja, és az Azure Vault a biztonsági mentési és helyreállítási folyamatokat.
    2. -Tároló létrehozása a Resource Manager-sablon létrehozása az egyes előfizetésekben.
4. Az Azure Policy az összes előfizetés frissítése:
    1. Naplózási és kritikusság és az adatok besorolás kényszerítése az összes előfizetés olyan előfizetéseket, az üzletmenet szempontjából kritikus fontosságú eszközök azonosításához.
    2. Naplózás, és csak a jóváhagyott lemezképeket használatának kényszerítése.
5. Az Azure Monitor végrehajtása:
    1. Miután egy alapvető fontosságú előfizetés azonosítja, hozzon létre egy PowerShell-lel az Azure Monitor-munkaterületet. Ez a központi telepítés előtti folyamat.
    2. Tesztelés üzembe helyezés során a felhőalapú üzemeltetési csapat telepíti a szükséges az ügynökök és a tesztek felderítés.
6. Az összes alapvető fontosságú alkalmazásokat tartalmazó előfizetés az Azure Policy frissítését.
    1. Naplózási és az alkalmazás egy NSG-t az összes hálózati adapterekhez és alhálózatokhoz kényszerítése. Hálózati és informatikai biztonsági adja meg az NSG-t.
    2. Naplózás, és mindegyik hálózati interfész jóváhagyott alhálózatokat és virtuális hálózatok használatának kényszerítése.
    3. Naplózási és a felhasználó által megadott útvonaltáblák korlátozás kényszerítése.
    4. Naplózási és minden virtuális gép az Azure Monitor-ügynökök telepítésének kényszerítéséhez.
    5. Naplózási és kényszerítéséhez, hogy az Azure-tár létezik-e az előfizetésben.
7. Tűzfal-konfiguráció:
    1. Azonosíthatja a biztonsági követelményeknek megfelelő Azure tűzfal-konfiguráció. Lehetőségként azonosítson egy külső berendezés, amely az Azure-ral kompatibilis.
    2. Hozzon létre egy Resource Manager-sablon üzembe helyezéséhez szükséges konfigurációk a tűzfal.
8. Az Azure tervezet:
    1. Hozzon létre egy új Azure tervezet nevű `protected-data`.
    2. A tűzfal és az Azure tároló-sablonok hozzáadása a tervezethez.
    3. Adja hozzá az új szabályzatok a védett adatok előfizetésekhez.
    4. A tervrajz közzététele alapvető fontosságú alkalmazások üzemeltetése szánt felügyeleti csoporthoz.
    5. Az új tervezet alkalmazása minden érintett előfizetés, valamint a meglévő tervek.

## <a name="conclusion"></a>Összegzés

Ezeket a további folyamatok és a cégirányítási MVP módosításai segítségével lehetővé tevő erőforrás-szabályozása a kockázatokat. Együtt adnak hozzá a helyreállítás, teljesítményalapú méretezést, illetve biztosítson hatékony eszközöket a felhő-kompatibilis műveleteket ellenőrzés.

## <a name="next-steps"></a>További lépések

Felhőre való áttérés folyamatosan fejlődnek, és további üzleti értéket, kockázatokat és a felhő cégirányítási kell is fejlődik. Tartson a folyamatban lévő fiktív vállalat a következő trigger akkor, ha a méretezési csoport a központi telepítés meghaladja a 100 eszközök a felhőben vagy a havi kiadások meghaladja a 1000 / hó. Ezen a ponton a felhő Cégirányítási csapat hozzáadja a Cost Management szabályozza.

> [!div class="nextstepaction"]
> [A Cost Management fejlődést szem előtt tartva](./cost-management-evolution.md)