---
title: 'CAF: A Cloud bevezetési keretrendszert – első lépések'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: Az első fázisa egy vállalati digitális átalakulást az Azure a felhőalapú technológiák bevezetése áttekintését ismerteti.
author: petertaylor9999
ms.date: 2/11/2019
ms.openlocfilehash: bfa325ded8c39915ad4d495b4309b700abd71cc2
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55898357"
---
# <a name="getting-started-with-the-cloud-adoption-framework"></a>A Cloud bevezetési keretrendszert – első lépések

A **digitális átalakulás** a felhőalapú számítástechnika jelöli a shift működtesse a helyszíni a felhőben működő. Ez a változás az üzleti tevékenységet folytató új módszereket vezetett be – például a digitális átalakulás kerülnek a figyelem középpontjába a szoftver- és adatközponti hardverek tőkeberuházását működő felhőhasználatát a felhőbeli erőforrások használatát. Nézzük meg, hogyan kezdheti el az a [az Azure-hoz a Microsoft Cloud bevezetési keretrendszert](../overview.md).

## <a name="the-digital-transformation-process"></a>A digitális átalakítási folyamat

A felhő bevezetésének sikeresek lehetnek, vállalati elő kell készítenie a szervezet, személyek és folyamatok készen áll a digitális átalakításban. Minden vállalati szervezeti struktúrája eltér, így nincsenek a vállalati felkészültség időkorlátokat. Ez a dokumentum ismerteti a magas szintű lépéseket készen állnak a vállalat is igénybe vehet. A szervezet egy részletes tervet a listán szereplő lépések elvégzéséhez időt kell.

A magas szintű folyamata a digitális átalakítás esetén:

1. Hozzon létre egy felhőalapú stratégia csapat. Ez a csapat felelős vezető a digitális átalakulást. Fontos továbbá ebben a szakaszban egy cégirányítási csapat és a egy biztonsági csapat, a digitális átalakítás kialakításához.
2. A felhő stratégia csapat ismerje meg, új és más a felhőalapú technológiák terén.
3. A felhő stratégia csapat vállalati előkészíti a digitális átalakulás üzleti kis létrehozásával - számba veszi a jelenlegi hézagok, az üzleti stratégia, és határozza meg azokat a magas szintű megoldások.
4. Magas szintű megoldások az üzleti csoportok igazítása A saját megtervezését és megvalósítását az egyes megoldások az egyes üzleti csoportokban résztvevők azonosítása.
5. Fordítja le a meglévő szerepköröket, a képességek és a felhőalapú szerepkörök, -szaktudását és folyamat folyamat.

<!--6. Develop processes for operating in the cloud to make solutions more robust in terms of availability, resiliency, and security.
1. Optimize solutions for performance, scalability, and cost efficiency.-->

## <a name="step-1-create-a-cloud-strategy-team"></a>1. lépés: Felhőalapú stratégia csapat létrehozása

Az első lépés a vállalati digitális átalakulást van vonzó részéről érkezett Cégvezetők felhőalapú stratégia csapat (Kínai téli idő szerint) a szervezeten belül. Ez a csapat pénzügyi, IT-infrastruktúráját és alkalmazásbiztonsági csoportokból részéről érkezett Cégvezetők áll. Ezek a csapatok segíthet a felhőalapú elemzési és a Kísérletezési fázisban.

Például egy felhőalapú stratégia csapat sikerült meghatározni a műszaki Igazgatója szerint, és a vállalati architektúra csapat, informatikai pénzügyi, vezető technologists különböző informatikai alkalmazások csoportokból (HR, Pénzügy és így tovább), és az infrastruktúráról, biztonság, vezetők tagjaiból és a csapatok hálózattal.

Szintén fontos két magas szintű csoportok alkotják: egy cégirányítási csapat és a egy biztonsági csoport. Ezek a csapatok felelősek kialakítása, megvalósítása és a folyamatban lévő sablonesemények naplózását a vállalati cégirányítási és biztonsági házirendjeivel. A cégirányítási kell dolgoznia tagokat, amelyeket már használta az eszköz védelmét, Költségkezelés, Csoportházirend és a kapcsolódó témaköröket. A biztonsági csapat tagjait, amelyek vannak jól ismeri az aktuális biztonsági szabványokon, valamint a vállalat biztonsági követelményeinek van szükség.

![Felhőalapú stratégia csapat, szabályozási és biztonsági csapatok számára](../_images/getting-started-overview-1.png)

A cégirányítási csapat felelős tervezése és megvalósítása a vállalati irányítás modell a felhőben, valamint üzembe helyezése és karbantartása a megosztott infrastruktúra-eszközök, amelyek részei a digitális átalakulást. Ezek az eszközök közé tartozik a hardverek, a szoftverek, és a felhőbeli erőforrások szükségesek, hogy a helyszíni hálózat csatlakoztatása virtuális hálózatok a felhőben.

A biztonsági csapat felelős, amelyek segítenek a vállalati biztonsági házirend a felhőben, szorosan működik az irányítási csapatával. A biztonsági csapat a biztonsági határ a helyszíni hálózat tartalmazza a virtuális hálózatok a felhőben a bővítmény tulajdonosa. Ez igénybe vehet a tulajdonos és a felhőbeli virtuális hálózaton a bejövő és kimenő tűzfalak fenntartása, valamint biztosítja az eszközöket és a házirend megakadályozásának jogosulatlan erőforrások központi telepítése formájában.

## <a name="step-2-learn-whats-new-in-the-cloud"></a>2. lépés: Megtudhatja, Miben változott a felhőben

A következő lépése a vállalati digitális átalakulást további információ a felhő-számítástechnika változik hogyan módja a vállalat üzleti tevékenysége a felhő stratégia csapat tagjainak szól. Ez az előkészítési és a módosítások az üzleti, személyek és technológia megtervezése. Fontos a felhő stratégia csapata tagjaihoz, mi az új és más, mint a korábban megszokott helyszíni a felhőben.

![Felhőalapú stratégia, szabályozási és biztonsági csapatok ismerje meg, hogy a felhőben működő ajánlott eljárásai.](../_images/getting-started-overview-2.png)

A kiindulási pont, a felhő megértéséhez van tanulási [Azure működéséről](what-is-azure.md) magas szinten. Ezt követően ismerje meg a alapjait [irányítás az Azure-ban](what-is-governance.md) előkészítésekor [erőforrás hozzáférés-kezelés](azure-resource-access.md).

Speciális tanulási a fogalmakkal és tervezési útmutatókat a tartalomjegyzék cégirányítási szakaszában a cégirányítási csapat tekintse át. Az infrastruktúra és a számítási feladatok szakaszok hasznosak tipikus architektúrák és a felhőalapú számítási feladatokat.

## <a name="step-3-identify-gaps-in-business-strategy"></a>3. lépés: Üzleti stratégia hiányosságok azonosítása

A következő lépés a felhő stratégia csapat számbavétele az üzleti problémák, amelyek a digitális átalakulás megoldásra van szüksége van. Vállalati például előfordulhat, hogy rendelkezik egy meglévő helyszíni adatközpont teljes életciklusa hardver cseréje szükséges. Egy másik példa vállalati-piacra új funkciók és szolgáltatások nehézségeik léphetnek fel, és a verseny is csökken. Ezek hézagok képviselik a *célok* , a vállalati digitális átalakulást.

Üzleti stratégia hiányosságok is kell besorolni, a következő kategóriákba sorolhatók:

|Kategória|Leírás|
|-----|-----|
|Költségkezelés|A technológia a vállalat fizet módon eseményáramlási kimaradást jelöli.|
|Szabályozás|A folyamatok engedményezhetné szembeni helytelen használat költség túllépését, a biztonsági problémák vagy a megfelelőségi problémák eredményezheti, hogy a vállalat által használt eseményáramlási kimaradást jelöli. |
|Megfelelőség|A vállalati betartja a saját belső folyamatok és házirendek, valamint a külső törvényeknek, előírásoknak és szabványok ugyanúgy eseményáramlási kimaradást jelöli. |
|Biztonság|A vállalati külső fenyegetések elleni védelmet biztosít a technológia és az eszközök ugyanúgy eseményáramlási kimaradást jelöli. |
|Adatirányítás|A vállalat kezeli az adatokat, különösen a vásárlói adatokat módon eseményáramlási kimaradást jelöli. Például az új általános adatvédelmi rendelet (GDPR) az Európai Unióban, szigorú követelményeket állít fel, amely lehet szükség, új hardver- és ügyféladatok védelméről.|

A vállalat rendelkezik az összes üzleti stratégia hézagok sorolhatók ezekben a kategóriákban, miután a következő lépésben feladata annak megállapítása, mindegyikhez egy magas szintű megoldások.

Az alábbi táblázatban néhány példa látható:

|Üzleti stratégia térköz|kategória &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;|Megoldás &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|
|-----|-----|-----|
| Jelenleg az üzemeltetett szolgáltatás a helyszíni problémákat észlel a rendelkezésre állás, rugalmasság és méretezhetőség megugrása, amely használati nagyjából tíz százalékát idő alatt. A helyi adatközpontban lévő kiszolgálók olyan teljes életciklusa. Vállalati IT azt javasolja, hogy új vásárlási a helyszíni hardver előírásoknak megugrása kezelésére használható.| Költségkezelés | Érintett meglévő helyszíni számítási feladatok migrálása kellene fizetnie, csak a felhőben, méretezhető erőforrásokhoz. |
| Külső felügyeleti törvényeknek és szabályozásoknak kell a vállalat igazodnia kell beállítani a szabványos vezérlőket, hogy új hardver- és az inaktív adatok titkosításának megkövetelése. | Adatirányítás | Adatok áthelyezése az Azure storage service encryption az inaktív adatokat. |
| A helyi adatközpontban lévő üzemeltetett szolgáltatások lett tapasztaló elosztott szolgáltatásmegtagadásos (DDoS) támadásoktól a nyilvánosan elérhető szolgáltatásait. A támadások nehezen csökkentése és a szükséges új hardveres, szoftveres és biztonsági csoporthoz, és hatékony kezelésére. | Biztonság | Szolgáltatások migrálása az Azure-ba, és kihasználhatja az Azure DDoS Protection.|

Ha az összes üzleti stratégia található hézagok enumerált, és magas szintű megoldások megállapítása, rangsorolja a listában. A lista az üzleti stratégia hiányosságokat, és a vállalati rövid és hosszú távú célok az egyes kategóriák egymáshoz igazításával prioritása is lehet. Például, ha a rövid távú célja, hogy csökkentse a vállalati informatikai költségek a következő két pénzügyi negyedéven belülre esik, az üzleti hiányosságok a *Költségkezelés* kategória az előre jelzett költséget mentése társított minden egyes prioritása is lehet.

Ez a folyamat kimenete üzleti kategóriák igazítva magas szintű megoldások stack rangsorolt listáját.

## <a name="step-4-align-high-level-solutions-with-business-groups-to-design-solutions"></a>4. lépés: Magas szintű megoldások az üzleti csoportok tervezési megoldások igazítása

Most, hogy a kitűzött célokat a digitális átalakulás érdekében bevezetett lettek besorolva, előnyt élvez, és magas szintű megoldások javasolt, van-e a következő lépés a felhő stratégia csapat számára az egyes üzleti tervezési és megvalósítási csapatok számára a magas szintű megoldások mindegyikének igazítása csoportok.

A csapatok igénybe vehet a rangsorolt listája, és haladjon végig az egyes magas szintű megoldások minden megoldást. A tervezési folyamat magában foglalja a specifikációnak új infrastruktúra és az új számítási feladatok. Is előfordulhatnak, személyek és a folyamatok, akkor hajtsa végre a szerepkör-módosításokat. Emellett az is rendkívül fontos ezen a ponton minden tekintse át az egyes kialakítások a cégirányítási és a biztonsági csoportokhoz közé tartozik a tervezési csapat. Minden tervhez belül kell maradnia a házirendek és a szabályozási és biztonsági csapatok által meghatározott eljárást, és ezek a csapatok szerepelnie kell a végső bejelentkezési minden egyes kialakítások.

![A felhő stratégia csapat kéz magas szintű megoldások tervezésének és megvalósításának Teams ki.](../_images/getting-started-overview-3.png)

A kialakítás az egyes megoldások nem triviális feladat, és a tervek létrehozásakor figyelembe kell venni a többi csapatban más megoldás mintákat a környezetben. Például ha a tervek számos egy meglévő helyszíni alkalmazások és szolgáltatások áttelepítése a felhőbe, valószínűleg hatékonyabb, ha ezek együtt csoportot, és a egy teljes áttelepítési stratégiájának tervezése. Egy másik példa hogy nem lehetséges áttelepítéséhez, bizonyos meglévő helyszíni alkalmazások és szolgáltatások és a megoldás lehet őket az új fejlesztési vagy külső szolgáltatásokkal. Ebben az esetben ez hatékonyabb lehet csoportosítani ezeket együtt, és meghatározhatja a meghatározásához, ha egy külső szolgáltatás használható-e egynél több megoldás közöttük némi átfedés.

A megoldás kialakításának befejeződése után a csapat a megvalósítási fázis minden tervhez helyezi. A megvalósítási fázis az egyes megoldásterv standard-projekt felügyeleti folyamatok segítségével is futtathatók.

## <a name="step-5-translate-existing-roles-skills-and-process-for-the-cloud"></a>5. lépés: Fordítása a meglévő szerepköröket, -szaktudását és a felhő folyamata

Minden egyes fázisban fejlődést során az informatikai iparágban előzményeit a legtöbb iparági módosításainak gyakran alkalmazott szerepkörök változása jelöli. Során Nagyszámítógépek való váltás az ügyfél-/ kiszolgálómodellt a szerepkört üzemeltető számítógép nagymértékben eltűnt, váltotta fel a rendszergazdához. Amikor virtualizálási korát megérkezett, csökken a követelmény a fizikai kiszolgálók használatához egyéni felhasználók számára, virtualizációs szakemberek számára kell cserélni. Ehhez hasonlóan, intézmények változnak, a felhőalapú számítástechnika, szerepkörök valószínűleg változni fognak újra. Adatközpont-szakemberek például felhőalapú pénzügyi elemzők előfordulhat, hogy helyettesíteni. Olyan esetekben, ahol informatikai beosztásukat nem változtak, még akkor is, a napi munka szerepkörök jelentősen fejlődtek.

Informatikai előfordulhat, hogy nyugtalan a szerepkörökről és pozíciók, akkor vegye figyelembe, hogy képességek külön készletét lesz szükség a felhőalapú megoldások támogatása. Azonban, hogy lehetőségének Agilis alkalmazottak, akik vizsgálata, és ismerje meg az új felhőalapú technológiákat nincs szükség. Vezethet az elfogadását a cloud services és a szervezeti ismertetése, és kihasználni a kapcsolódó változások is.

### <a name="capturing-concerns"></a>A problémák rögzítése

A digitális átalakulás során minden egyes csapat rögzíteni kell minden személyzeti aggályokat információforrásait. A problémák rögzítésekor határozza meg a következőket:

* Probléma típusa. Például a munkavállalók ellenállóbbá tehesse a feladatkörhöz abban a digitális átalakulás változásokat lehet.
* A probléma, ha a nem neki címzett hatását. Például a digitális átalakulás szembeni ellenállás lehet a szükséges módosítások végrehajtásához lassú feldolgozók eredményez.
* A terület, a probléma megoldása érdekében. Például ha az informatikai részleg feldolgozók szerezni az új ismeretek migrál, az informatikai érdekelt terület legjobb rendelkezik a ezen probléma megoldásának. Törölje az egyes problémák azonosítása a terület lehet, és ezekben az esetekben szükség lehet eszkalálni a végrehajtó vezetői.

### <a name="identify-gaps"></a>Hézagok azonosítása

Egy másik aspektusa feldolgozása révén a vállalati digitális átalakulást problémái azonosítja **hézagok**. Eseményáramlási kimaradást egy szerepkör, szakértelem vagy folyamat, a digitális átalakítást, amely jelenleg nem létezik a vállalat számára szükséges.

Első lépésként új kötelezettségeit abban a digitális átalakulás fektetve az új feladatokról és kivezetés aktuális feladatok számbavétele. Azonosítsa a területen minden felelősséget igazodik. Új Felelősségek meghatározása különös figyelmet fordítanak igazított terület van. Számos területen látnia néhány feladatkört kiterjedhet, és ez jelenti, hogy rögzíteni kell fontos jobb elrendezésben lehetőséget. Abban az esetben, ahol nincs terület azonosított felelősként rögzítse a virtuális gépet, a térköz.

Ezután azonosítsa a feladata támogatásához szükséges ismeretek. Határozza meg, ha a vállalat rendelkezik-e az ezekkel a meglévő erőforrások. Ha nincsenek meglévő erőforrások, állapítsa meg, milyen képzési programok vagy szakembereket beszerzési szükség. Határozza meg, hogy az időkeretet, amely szerint felelőssége támogatnia kell a digitális átalakulás érdekében.

Végül azonosítsa a szerepköröket, amely végrehajtja az ezekkel. A meglévő munkatársak némelyike feltételezi a szerepkör részei, és más esetekben szükség lehet egy teljesen új szerepkör.

### <a name="partner-across-teams"></a>Különböző csapatokkal partner

A ismeretek szükségesek, hogy a szervezet digitális átalakításában található hézagok kitöltésének általában nem korlátozódik egyetlen szerepkörrel, vagy akár egy részleghez. Képességek fog rendelkezni, kapcsolatok és függőségekkel foglalkozik, egy egyetlen vagy több szerepkört is kiterjedhet, és ezeket a szerepköröket több megyéiben előfordulhat, hogy létezik. Például a számítási feladatok felelőse igényelhetnek valaki a az informatikai szerepkör kiépítése alapvető erőforrások, például az előfizetésekhez és erőforráscsoportokhoz.

A függőségek kezelése a szerepkörök között munkafolyamat megvalósító a szervezet új folyamatok képviseli. A fenti példában nincsenek számos különböző típusú folyamat, amely képes támogatni a számítási feladatok felelőse és az informatikai szerepkör közötti kapcsolat. Például a folyamat kezeléséhez egy munkafolyamat eszközzel hozható létre, vagy egy egyszerű e-mail sablon használható.

Nyomon követheti a függőségeket, és jegyezze fel a folyamatot, amely támogatja őket, és attól, hogy a folyamat létezik-e jelenleg. Az folyamat, amely szükséges az eszközök győződjön meg arról, hogy az idősor minden olyan eszközt, üzembe helyezéséhez igazítja a teljes digitális átalakulás ütemezésével.

## <a name="next-steps"></a>További lépések

A digitális átalakulás iteratív folyamat, és minden egyes ismétléskor az érintett csoportok hatékonyabb lesz.

> [!div class="nextstepaction"]
> [Az Azure működésének megismerése](what-is-azure.md)
