---
title: "Magas rendelkezésre állás a Azure-alkalmazások"
description: "A műszaki áttekintés és és a magas rendelkezésre állású alkalmazások támaszkodva a Microsoft Azure ad részletes tájékoztatást."
author: adamglick
ms.date: 05/31/2017
ms.openlocfilehash: 46b7b802326a8de03546528aaeb1a1c6419d41db
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
[!INCLUDE [header](../_includes/header.md)]
# <a name="high-availability-for-applications-built-on-microsoft-azure"></a>Magas rendelkezésre állás a Microsoft Azure épülő alkalmazások
A magas rendelkezésre állású alkalmazások elnyeli a rendelkezésre állási, a betöltés és a függő szolgáltatások, valamint a hardver ideiglenes hibák ingadozását. Az alkalmazás továbbra is fennáll, üzleti követelmények vagy alkalmazás szolgáltatásiszint-szerződések (SLA) által megadott akadálytalanul végrehajtásához.

## <a name="azure-high-availability-features"></a>Az Azure magas rendelkezésre állású szolgáltatások
Azure számos beépített platform szolgáltatást, amely támogatja a magas rendelkezésre állású alkalmazások rendelkezik. Ez a szakasz ismerteti azokat a kulcsfontosságú szolgáltatásokat egy részénél.

### <a name="fabric-controller"></a>Fabric-vezérlő
Az Azure fabric controller kiépítését, és az Azure számítási példányokért állapotát figyeli. A fabric controller figyeli a hardver állapotát, és a gazdagép és vendég gépen példányok szoftver. Ha hibát észlel, automatikusan áthelyezése a Virtuálisgép-példányok által karbantartott SLA-k. További hiba és a frissítési tartományok fogalma a compute SLA-t támogatja.

Amikor a felhőalapú szolgáltatás szerepkör több példánya van telepítve, Azure különböző tartalék tartományok ezek a példányok telepíti. A tartalék tartomány határ lényegében egy másik hardverre állvány ugyanabban a régióban. Tartalék tartományok csökkenti annak a valószínűsége, hogy a honosított hardverhiba megzavarja a szolgáltatás egy alkalmazás. A feldolgozói szerepkörök vagy a webes szerepkörök tartalék tartományok száma nem lehet kezelni. A fabric controller a dedikált erőforrások, amelyek az Azure által kezelt alkalmazásokból külön. Mivel az Azure rendszer magja körül, 100 %-os hasznos üzemidő igényel. Azt figyeli, és kezeli a szerepkörpéldányok tartalék tartományok között.

Az alábbi ábrán látható Azure megosztott erőforrások, hogy a háló vezérlő telepíti, és kezeli a különböző tartalék tartományok között.

![Tartalék tartomány elkülönítési egyszerűsített nézete](./images/high-availability-azure-applications/fault-domain-isolation.png)

Míg tartalék tartományok fizikai színkivonatok hiba elhárítása érdekében, a frissítési tartományok példány elkülönítése határozza meg, mely egy szolgáltatás példányának frissíti, az adott időpontban logikai egységek. Alapértelmezés szerint az öt frissítési tartományok az üzemeltetett szolgáltatás üzembe helyezéshez vannak definiálva. Azonban módosíthatja ezt az értéket a szolgáltatásdefiníciós fájlban. Például ha a webes szerepkör nyolc példányai, nincsenek két-három frissítési tartományok példány és egy frissítési tartományt két példánya. Azure határozza meg, hogy a frissítés során a frissítési tartományok száma alapján. További információkért lásd: [felhőalapú szolgáltatás frissítése](/azure/cloud-services/cloud-services-update-azure-service/).

### <a name="features-in-other-services"></a>Egyéb szolgáltatások szolgáltatásai
A platform funkciói, amely támogatja a magas rendelkezésre állású számítási erőforrásokat, valamint az Azure magas rendelkezésre állású szolgáltatások beágyazása az egyéb szolgáltatások. Például az Azure Storage Azure-tárfiókja adatait legalább három replikáit tárolja. Emellett lehetővé teszi az adatok másolatát tárolja egy másodlagos régióban a georeplikáció. Az Azure Content Delivery Network lehetővé teszi, hogy a blobok gyorsítótárazni a világ redundancia, a méretezhetőség és a kisebb késést biztosít. Az Azure SQL-adatbázis, valamint a több replikáit tárolja.

Azure platformon rendelkezésre állási funkcióval mélyebb tárgyalását lásd: [rugalmassági műszaki útmutatót](index.md). Lásd még: [ajánlott eljárások a Windows Azure felügyeleti teendők központjaként szolgáltatások tervezéséhez](https://azure.microsoft.com/blog/best-practices-for-designing-large-scale-services-on-windows-azure/).

Azure több funkció, amely támogatja a magas rendelkezésre állást biztosít, de fontos tudni, hogy azok korlátai:

* Számítás Azure biztosítja azt, hogy a szerepkörök rendelkezésre álló és fut, de az nem észleli, ha az alkalmazás fut, vagy túlterhelt.
* Az Azure SQL Database adatait replikálja a rendszer szinkron módon történik a régión belül. Lehetősége van aktív georeplikáció, amely lehetővé teszi, hogy legfeljebb négy további adatbázis-másolatok a ugyanabban a régióban (vagy különböző régiókban). Ezek az adatbázis-replikák nincsenek időpontban biztonsági mentések, SQL-adatbázis időpontban a biztonsági mentési lehetőségeket nyújtja. További információkért lásd: [automatikus biztonsági mentések használatával Azure SQL-adatbázis helyreállítása: pont időponthoz kötött visszaállítás](/azure/sql-database/sql-database-recovery-using-backups#point-in-time-restore).
* Az Azure Storage tábla adatai és a blob adatait egy másik régióban alapértelmezés szerint lesznek replikálva. Azonban nem férhet hozzá a replikák mindaddig, amíg a Microsoft úgy dönt, hogy áthelyezze a feladatokat a másodlagos helyre. Régió feladatátvétel csak a hosszan tartó régió kiterjedő szolgáltatás szüneteltetése során, és nincs nincs SLA-t a földrajzi-feladatátvételi idő. Is fontos megjegyezni, hogy sérült adatainak gyors között osztja el a replikához. Ezen okok miatt platform rendelkezésre állási funkcióval rendelkező alkalmazás-specifikus rendelkezésre állási funkciót, beleértve a időpontban biztonsági mentéseket a blob típusú adatok létrehozása a blob pillanatkép funkció kell kiegészíteni.

### <a name="availability-sets-for-azure-virtual-machines"></a>Rendelkezésre állási készletek az Azure virtuális gépek
Ez a dokumentum elsősorban a felhőalapú szolgáltatásokat, amelyek platform,--szolgáltatás (PaaS) modellt használnak összpontosít. Van még adott rendelkezésre állási funkcióval az Azure virtuális gépek, amelyek olyan infrastruktúra,--szolgáltatás (IaaS) modellt használnak. A virtuális gépek magas rendelkezésre állás elérése érdekében rendelkezésre állási készletek, amelyek fault és frissítési tartományt hasonló funkcióval szolgálnak kell használnia. Egy rendelkezésre állási csoportot belül Azure állítható be a virtuális gépek oly módon, hogy a honosított hardver hibák és karbantartási tevékenységeket, hogy a csoportban található összes gép leáll. Rendelkezésre állási csoportok az Azure garantált szolgáltatási szintje a virtuális gépek rendelkezésre állásának eléréséhez szükséges.

Az alábbi ábrán látható, két rendelkezésre állási készletek és az SQL Server virtuális gépek, illetve.

![Rendelkezésre állási készletek az Azure virtuális gépek](./images/high-availability-azure-applications/availability-set-for-azure-virtual-machines.png)

> [!NOTE]
> A fenti ábrán az SQL Server telepítve és fut a virtuális gépeken. Ez eltér az Azure SQL Database, amely felügyelt szolgáltatásként adatbázist biztosít.
> 
> 

## <a name="application-strategies-for-high-availability"></a>Alkalmazás stratégiák a magas rendelkezésre állás
A magas rendelkezésre állás érdekében a legtöbb alkalmazás stratégiák tartalmaz, amely redundancia vagy alkalmazás-összetevők közötti rögzített függőségek eltávolítása. Alkalmazás tervét támogatnia kell a hibatűrés Azure vagy harmadik féltől származó szolgáltatással szórványos leállások során. A következő szakaszok ismertetik a felhőalapú szolgáltatások rendelkezésre állásának javítása az alkalmazás minták.

### <a name="asynchronous-communication-and-durable-queues"></a>Aszinkron kommunikációt és a tartós várólisták
Rendelkezésre állás biztosításához az Azure-alkalmazások javítása érdekében fontolja meg lazán összekapcsolt szolgáltatások közötti aszinkron kommunikációhoz. Ebben a mintában üzenetek kerülnek vagy tárolási sorok vagy az Azure Service Bus-üzenetsorok későbbi feldolgozásra. Üzenet a várólistában írásakor vezérlő azonnal visszaadja a küldőnek. (Általában valósul meg a feldolgozói szerepkör) az alkalmazás egy másik szolgáltatás feldolgozza az üzenetet. Ha a feldolgozási szolgáltatás leáll, az üzenetek gyűlik össze a várólista a feldolgozási szolgáltatás visszaállításáig. Az előtér-feladó és az üzenet processzor között nincs közvetlen függőség van. Ezzel a megoldással a szinkron hívások okozó szűk keresztmetszetek az elosztott alkalmazásokban.

Ez a minta egy változata Azure Storage (BLOB, tábla vagy várólisták) vagy a Service Bus-üzenetsorok adatbázis sikertelen hívások adatait tárolja. Például egy szinkron (például az Azure SQL Database) egy másik szolgáltatásba az alkalmazáson belül hibát jelez, ismételten. Előfordulhat, hogy a kérésre szerializálni a tartós tárba. Később bármikor Ha a szolgáltatás vagy az adatbázis online elérhető lesz, az alkalmazás is küldje el újra a kérelmet tárolóból. Ebben a modellben különbség, hogy a köztes hely szolgál csak során hibák, nem az alkalmazás munkafolyamata során.

Mindkét esetben aszinkron kommunikációt és köztes tárolási megakadályozhatják, hogy egy leállított háttér-szolgáltatás a teljes alkalmazás leáll. A logikai közvetítő várólisták szolgál. Sorkezelési szolgáltatások közötti kiválasztásáról további információkért lásd: [Azure várólisták és az Azure Service Bus-üzenetsorok &mdash; képest és ellentétben](/azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted/).

### <a name="fault-detection-and-retry-logic"></a>Hiba észlelése, és próbálkozzon újra logika
Egyik fontos szempontja a magas rendelkezésre állású alkalmazások kialakításának újrapróbálkozási logika egy szolgáltatás, amely ideiglenesen nem érhető el kezelésére kód használata. Azure Storage mind az Azure Service Bus SDK újabb verzióiban az újrapróbálkozások natív módon támogatja. Az egyéni újrapróbálkozási logika biztosítása az alkalmazás további információkért lásd: a [újrapróbálkozási mintát](../patterns/retry.md).

### <a name="reference-data-pattern-for-high-availability"></a>A magas rendelkezésre állású hivatkozás adatmintát
Referenciaadatok az alkalmazás csak olvasható adatai. Ezeket az adatokat biztosít a az üzleti környezet, amelyben az alkalmazás adatokat állít elő tranzakciós üzleti művelet során. Pillanatkép készítése a referencia-adatokat a tranzakció befejeződött időpontjában tranzakciós adatok integritásának függ.

Referenciaadatok szükség az alkalmazás megfelelő működését. Számos olyan alkalmazás létrehozása és karbantartása a referenciaadatok; törzsadat-felügyelet (MDM) rendszerek gyakran ez a művelet végrehajtására szolgál(nak). Ezek a rendszerek a referenciaadatok életciklusának felelősek. Referenciaadatok például termékkatalógus, alkalmazott master, részek master és berendezések fő. Referenciaadatok kezdeményezhetik a szervezeten kívülre, például irányítószám vagy adó díjszabás is. Referenciaadatok rendelkezésre állásának növelése kapcsolatos olyan stratégiák nehezen általában kevesebb mint a tranzakciós adatok. Referenciaadatok azt az előnyt, hogy legtöbbször nem módosítható.

Hivatkozási adatokat használó Azure webes és feldolgozói szerepkörök lehet kapcsolódni a autonóm futási időben telepítésével a referenciaadatok együtt az alkalmazás. Ez a megközelítés ideális, ha a helyi tárterület mérete lehetővé teszi, hogy a központi telepítés. Beágyazott SQL-adatbázisok, NoSQL-adatbázisok vagy XML-fájlok helyben telepített Azure számítási méretezési egységek önállóan számára. Azonban rendelkeznie kell egy olyan mechanizmus az adatokon az egyes szerepkörökhöz anélkül, hogy újbóli üzembe helyezése. Ehhez az szükséges, helyezze a referenciaadatok frissítéseit a tárolási felhővégpontnak (például az Azure Blob-tároló vagy az SQL-adatbázis). Adja hozzá mindegyik szerepkör, amely letölti a frissítéseket a számítási csomópontok szerepkör indításkor a kódot. Azt is megteheti adja hozzá a kódot, amely lehetővé teszi a rendszergazdáknak a szerepkör-példányok a kényszerített letölteni.

Magasabb rendelkezésre állását, a szerepkörök is tartalmaznia kell a hivatkozás adatkészletet abban az esetben, ha tárolási nem működik. Szerepkör indításához referenciaadatok alapvető vannak beállítva, míg a tárolási erőforrások érhető el a frissítéseket.

![Alkalmazás magas rendelkezésre állású autonóm számítási csomóponton keresztül](./images/high-availability-azure-applications/application-high-availability-through-autonomous-compute-nodes.png)

Ebben a mintában az új telepítések vagy a szerepkörpéldányok hosszabb időt vehet igénybe elindítani a referenciaadatok nagy mennyiségű letöltése vagy telepítése. Előfordulhat, hogy ez kompromisszumot azonnal referenciaadatok gyakorló szerepkörönként helyett attól függően, külső tárolószolgáltatások önállóan elfogadható.

### <a name="transactional-data-pattern-for-high-availability"></a>A magas rendelkezésre állású tranzakciós adatmintát
Tranzakciós adatok az adatai, amely az alkalmazás hoz létre egy vállalati környezetben. Tranzakciós adatok, amely az alkalmazás megvalósítja az üzleti folyamatokat és a referenciaadatok, amely támogatja ezeket a folyamatokat kombinációja. Tranzakciós adatok tartalmazhatnak rendelések, speciális szállítási értesítéseket, számlákat és felhasználói kapcsolat kezelő (CRM) lehetőséget. Tranzakciós adatok külső rendszerekkel rögzítésére, vagy további feldolgozásra van megadva.

Referenciaadatok a rendszerek, adatok felelős belül módosíthatja. Ezért tranzakciós adatok minimalizálása érdekében a szemantikai konzisztencia vonatkozó külső függőségeket időpontban hivatkozás adatkörnyezet kell menteni. Például a termék eltávolíthatja a katalógusból több hónap után egy rendelés teljesül. Mértékű hivatkozás adatkörnyezetet tárolása a tranzakcióhoz lehetőség szerinti használata ajánlott. Ezt a módszert megőrzi a tranzakcióhoz tartozó szemantikáját, még akkor is, ha a referenciaadatok megváltozik, a tranzakció rögzített.

Ahogy korábban említettük ömlesztve berendezés és aszinkron kommunikációt használó architektúrák biztosíthat magasabb szintű elérhetőségét. Ez igaz, valamint a tranzakciós adatokat, de a megvalósítás összetettebb. Hagyományos tranzakciós minták általában támaszkodnak az adatbázist a tranzakció biztosítása. Köztes rétegek vezetnek be, amikor az alkalmazás kódja helyesen kezelni kell az adatokat a különböző rétegek elegendő konzisztencia és tartós biztosításához.

Az alábbi sorrendben, amely elválasztja a feldolgozási tranzakciós adatok rögzítése munkafolyamat ismerteti:

1. Webes számítási csomópont: jelen adatokra hivatkoztak.
2. A külső tárhelyen: köztes tranzakciós adatok mentése.
3. Webes számítási csomópont: végezze el a végfelhasználói tranzakció.
4. Webes számítási csomópont: a hivatkozási adatokat kontextusú a befejezett tranzakciós adatokat küldeni a kiszámítható választ ad garantáltan ideiglenes tartós tároló.
5. Webes számítási csomópont: a végfelhasználói tranzakció befejezésekor, jelezze.
6. Háttér számítási csomópont: a tranzakciós adatokat nyerhet ki, a feldolgozást szükség esetén és a jelenlegi rendszer elküldi a végleges tárolási helyét.

A következő ábrán egy lehetséges Ez a kialakítás megvalósítása az Azure által üzemeltetett felhőalapú szolgáltatás.

![Laza kapcsoló magas rendelkezésre állása](./images/disaster-recovery-high-availability-azure-applications/application-high-availability-through-loose-coupling.png)

A fenti ábrán szaggatott nyilak az aszinkron feldolgozás jelzi. Az előtér-webkiszolgáló szerepkör nincs tisztában az aszinkron feldolgozás. Ennek eredménye a tranzakció-a végső rendeltetési hely alapján a jelenlegi rendszer tárolására. A késés, amely a aszinkron modell vezet be, mert a tranzakciós adatok nincs azonnal lekérdezhetők. Ezért a tranzakciós adatok tárolóegységekhez kell menteni a gyorsítótár, vagy a felhasználói munkamenet az azonnali UI teljesítéséhez szükséges.

A webes szerepkör az autonóm az infrastruktúra többi részétől. A rendelkezésre állási profil a webes szerepkör és az Azure üzenetsor-kezelési és nem a teljes infrastruktúra. Túl magas rendelkezésre állású ezzel a megközelítéssel a webes szerepkör vízszintesen, a háttér-tároló független méretezése. Ez a magas rendelkezésre állású modell hatással lehetnek a műveletek gazdasági. További összetevők, például az Azure várólisták és feldolgozói szerepkörök hatással lehet a havi használati költségek.

Az előző ábrán látható, a tranzakciós adatok leválasztott megközelítést több megvalósítása. Nincsenek sok más lehetőség esetében. Az alábbiakban néhány más:

* A webes szerepkör és a tároló várólista közötti lehet, hogy helyezhető el a feldolgozói szerepkör.
* A Service Bus-üzenetsorba helyett az Azure Storage üzenetsorába használható.
* A végső cél Azure Storage vagy egy másik adatbázis-szolgáltató lehet.
* Az Azure Cache a webes réteg a közvetlen gyorsítótár követelmények biztosításához a tranzakció után lehet használni.

### <a name="scalability-patterns"></a>Méretezhetőség minták
Fontos megjegyezni, hogy a méretezhetőség egy felhőalapú szolgáltatás rendelkezésre állási közvetlen hatással van. Ha a megnövekedett terhelés miatt a szolgáltatás lefagyottnak, a felhasználói egyensúlyozhatom, hogy az alkalmazás nem működik. Kövesse a bevált gyakorlatokat méretezhetőségre, a várt alkalmazásterhelés és a későbbi verziójával kapcsolatos elvárások alapján. Sok szempontok, például egy vagy több tárfiókot, több adatbázis közötti megosztás, és a gyorsítótár stratégiák méretezési maximalizálva magában foglalja. Ezek a minták kapcsolatos részletes információkért lásd: [ajánlott eljárások a Microsoft Azure felügyeleti teendők központjaként szolgáltatások tervezéséhez](https://azure.microsoft.com/blog/best-practices-for-designing-large-scale-services-on-windows-azure/).

## <a name="next-steps"></a>Következő lépések
Az adatsorozat dokumentumok vész-helyreállítási és magas rendelkezésre állás a Microsoft Azure épülő alkalmazások tartalmazza. A következő cikk az adatsorozat [Microsoft Azure épülő alkalmazások vész-helyreállítási](disaster-recovery-azure-applications.md).

