---
title: "A szoftverminőség alappillérei"
description: "A software quality, a méretezhetőség, a rendelkezésre állási, a rugalmasság, a felügyeleti és a biztonsági öt oszlopok ismerteti."
author: MikeWasson
ms.openlocfilehash: 1d5e30602cafa0d39f92de3101974e77ae258595
ms.sourcegitcommit: a7aae13569e165d4e768ce0aaaac154ba612934f
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/30/2018
---
# <a name="pillars-of-software-quality"></a>A szoftverminőség alappillérei 

Egy sikeres felhőalkalmazás a szoftverminőség következő öt alappillérére koncentrál: skálázhatóság, rendelkezésre állás, rugalmasság, felügyelet és biztonság.

| Oszlop | Leírás |
|--------|-------------|
| Méretezhetőség | A rendszer azon képessége, megnövekedett terhelés kezelése érdekében. |
| Rendelkezésre állás | Az idő a rendszer működési, és üzemel arányát. |
| Resiliency | Azon képessége, a rendszer helyreállni hibák után, és továbbra is működik. |
| Kezelés | Operatív folyamatokat, amelyek operációs rendszert futtató üzemben hagyja. |
| Biztonság | Alkalmazások és adatok védelme a érintő fenyegetésekkel szemben. |

## <a name="scalability"></a>Méretezhetőség

Méretezhetőség azt a képességet, a rendszer a megnövekedett terhelés kezelése érdekében. Két fő módszer, amely egy alkalmazás is van. Függőleges méretezés (skálázás *mentése*) azt jelenti, hogy a kapacitás erőforrás, például növelése nagyobb Virtuálisgép-méret használatával. Horizontális skálázás (skálázás *kimenő*) erőforrás, például a virtuális gépek vagy adatbázis-replikák új példányok hozzáadásával. 

Horizontális skálázás rendelkezik keresztül függőleges skálázás jelentős előnyökkel jár:

- Igaz felhőbeli skálázással. Alkalmazások futtatását több száz vagy akár több ezer csomópontokat, amelyek nem lehetséges egycsomópontos méretezik elérése tervezhető.
- Horizontális skálázhatóságot rugalmas. Ha több példány is hozzáadhat a terhelés növekedésével, vagy távolítsa el őket csendesebb időszakokban.
- Méretezési out után is hozzáadhatja automatikusan, ütemezés szerint vagy a terhelés változásai. 
- Meg lehet, vertikális felskálázásával olcsóbbak. Több kis virtuális gépeken futó költség lehet kisebb, mint egyetlen nagy virtuális gép. 
- Horizontális skálázás is tovább fejlesztheti rugalmasságot redundancia hozzáadásával. Ha egy példány leáll, az alkalmazás folyamatosan működik.

Függőleges skálázás előnye, hogy ezt megteheti az alkalmazás változtatás nélkül. De egy bizonyos ponton korlátozni, ahol nem méretezhető esetleges további fel lesz találati. Ezen a ponton minden további skálázás kell vízszintes. 

Horizontális skálázhatóságot úgy kell megtervezni a rendszerbe. Például lehet horizontálisan virtuális gépek által egy terheléselosztó mögött helyezi őket. Azonban minden a készlet képesnek kell lennie minden ügyfél kérelem kezelésére, ezért az alkalmazás kell lennie az állapot nélküli vagy -állapot tárolása külsőleg (például, egy elosztott gyorsítótárban). Kezeli a szolgáltatások gyakran rendelkezik horizontális skálázás, és az automatikus skálázást beépített PaaS. Az egyszerű skálázás ezeket a szolgáltatásokat az egyik legnagyobb előnye PaaS services használatával.

Több példány hozzáadásánál többről nem jelenti azt, az alkalmazás lehessen méretezni, azonban. Akkor lehet, hogy egyszerűen leküldéses valahol máshol szűk. Például ha egy webes előtér-több ügyfélkérelmet kezelni, előfordulhat, hogy vált zárolási contentions az adatbázisban. Ezután fontolja meg a további intézkedések, például az egyidejű hozzáférések optimista vagy particionálás, ahhoz, hogy az adatbázis további átviteli adatok kellene.

Mindig tartson teljesítményét, és terheléses tesztelés a potenciális szűk keresztmetszetek kereséséhez. A rendszer, például adatbázisok, állapot-nyilvántartó részei a leggyakoribb szűk keresztmetszetek okozza, és horizontálisan méretezhető gondos tervezés szükséges. Egy szűk feloldása felfedheti máshol más szűk keresztmetszeteket.

Használja a [méretezhetőség ellenőrzőlista] [ scalability-checklist] áttekintheti a méretezhetőség szempontból kialakítását.

### <a name="scalability-guidance"></a>Méretezhetőség útmutató

- [A kialakítási minták a méretezhetőség és teljesítmény][scalability-patterns]
- Gyakorlati tanácsok: [automatikus skálázás][autoscale], [feladatok háttérben][background-jobs], [gyorsítótárazását] [ caching], [CDN][cdn], [adatparticionálás][data-partitioning]

## <a name="availability"></a>Rendelkezésre állás

Rendelkezésre állási ideje, hogy a rendszer működési, és üzemel arányának. Általában a hasznos üzemidő százalékaként értendő. Az alkalmazáshibák infrastrukturális problémák megoldását és rendszerterhelés összes csökkentheti rendelkezésre állása. 

A felhőalapú alkalmazások szolgáltatásiszint-célkitűzés (SLO), amely egyértelműen határozza meg a várt rendelkezésre állását, és hogyan kell mérni a rendelkezésre állási kell rendelkeznie. Rendelkezésre állási meghatározásakor keresse meg a kritikus elérési úton. Lehet, hogy az előtér-webkiszolgáló képes kiszolgálni az ügyfélkéréseket, de minden tranzakció sikertelen, mert nem tud kapcsolódni az adatbázishoz, ha az alkalmazás nem érhető el a felhasználók számára. 

Rendelkezésre állási gyakran leírt tekintetében "9s" &mdash; például a "négy 9s" azt jelenti, hogy 99,99 % hasznos üzemideje. Az alábbi táblázat a lehetséges összesített állásidő másik rendelkezésre állási szinten.

| % Hasznos üzemidő | Állásidő hetente | Állásidő havonta | Állásidő évente |
|----------|-------------------|--------------------|-------------------|
| 99% | 1,68 óra | 7,2 óra | 3,65 nap |
| 99.9% | 10 perc | 43,2 perc | 8,76 óra |
| 99.95% | 5 perc | 21,6 perc | 4,38 óra |
| 99.99% | 1 perc | 4,32 perc | 52,56 perc |
| 99.999% | 6 másodperc | 26 másodpercben | 5,26 perc |

Figyelje meg, hogy 99 %-os hasznos üzemidő hetente szinte 2 óra szolgáltatás kimaradás lehetett lefordítani. Számos alkalmazás, különösen a felhasználók felé néző alkalmazások, amelyek nem elfogadható slo-t. Másrészről, öt 9s (99.999 %) azt jelenti, hogy az állásidő legfeljebb 5 perc egy *év*. Nehéz elég csak kimaradás, amely gyorsan észlelésére, hát alone a probléma megoldását. Nagyon magas rendelkezésre állás eléréséhez (99,99 % vagy magasabb), nem elegendő a helyreállni hibák után manuális beavatkozásra. Az alkalmazás legyen önálló diagnosztizálni és önjavítás, amely, ahol rugalmassági kritikus fontosságú válik.

Az Azure a szolgáltatási szint szerződés (SLA) hasznos üzemidő és a kapcsolat a Microsoft felé vállalt kötelezettségeinket ismerteti. Ha az SLA-t egy adott szolgáltatáshoz 99,95 %, akkor kell látnia a szolgáltatás 99,95 %-ában elérhető legyen.

Alkalmazások általában több szolgáltatás függenek. Állásidő vagy szolgáltatást valószínűségét általában független. Tegyük fel, hogy az alkalmazás két 99,9 %-os SLA-t a szolgáltatás függ. Mindkét szolgáltatás esetében a összetett szolgáltatásiszint-szerződést 99,9 %-os &times; 99,9 %-os &asymp; 99.8 % vagy egy önálló valamivel kisebb, mint minden szolgáltatást. 

Használja a [rendelkezésre állási ellenőrzőlista] [ availability-checklist] áttekintheti a Tervező egy rendelkezésre állási tükrözik.

### <a name="availability-guidance"></a>Rendelkezésre állási útmutató

- [Kialakítási minta a rendelkezésre állás érdekében][availability-patterns]
- Gyakorlati tanácsok: [automatikus skálázás][autoscale], [háttér-feladatok][background-jobs]

## <a name="resiliency"></a>Resiliency

Rugalmasság azt a képességet, a rendszer helyreállni hibák után, és továbbra is működik. A rugalmasság célja térjen vissza az alkalmazás teljesen működőképes állapotba, miután hiba lép fel. Rugalmasság szorosan kapcsolódnak a rendelkezésre állási.

A hagyományos alkalmazásfejlesztés történt, így a fókusz a hibák (MTBF) közötti idő csökkentése. Elérhető telt megakadályozhatja, hogy a rendszer a megfelelő működése közben. A felhőalapú informatika meghatározására egy másik alaposan szükség, több tényezők miatt:

- Elosztott rendszerek bonyolult, és egy pontján hiba potenciálisan kaszkádolt is a rendszerben.
- Felhőalapú környezetek költségek mindig alacsony révén a hagyományos hardvereken, eseti hardverhibák kell számítani. 
- Alkalmazások általában külső szolgáltatásokat, amelyek esetleg átmenetileg elérhetetlenné válik, vagy nagy mennyiségű felhasználók szabályozás függenek. 
- Az aktuális felhasználó várt az alkalmazás elérhető 24/7 nélkül bármikor offline állapotra vált.

Ezek a tényezők jelenti, hogy a felhőalapú alkalmazások mindegyikét kell megtervezni, várt alkalmi hibák, és hárítsa el azokat. Azure a platform beépített számos rugalmassági lehetőséggel rendelkezik. Például: 

- Az Azure Storage, SQL-adatbázis és a Cosmos DB összes biztosít beépített replikációs, mind a régión belül, és régiók között.
- Azure-lemezeket felügyelt automatikusan kerülnek különböző tárolási méretezési egységeit, a hardver meghibásodása hatásait korlátozni.
- Virtuális gépek rendelkezésre állási csoportba több tartalék tartományok vannak elosztva. A tartalék tartomány olyan virtuális gépek, amelyek egy közös power forrás- és a hálózati kapcsolóhoz. Virtuális gépek terjednek tartalék tartományokban korlátozza a fizikai hardver meghibásodása, a hálózati kimaradások vagy a power megszakításokat hatását.

Említett, továbbra is szeretné rugalmassági építenie az alkalmazást. Rugalmasság stratégiák a architektúra minden szinten alkalmazható. Néhány azok mérséklési több más ajánlások taktikai jellegűek &mdash; például egy átmeneti hálózati hiba után újból próbálkozik a távoli hívásban. Egyéb megoldást több stratégiai, például a teljes alkalmazás másodlagos régióba feladatátvételét. Taktikai megoldást teheti a nagy különbség. Habár ritkán fordul elő, egy teljes régió egy problémákat tapasztalhat, átmeneti problémákat, például hálózati torlódás gyakori &mdash; így cél először ezeket. A megfelelő figyelési és diagnosztika az is fontos, mind hibák észlelés, ha akkor fordulhat elő, és keresi okait.

Az alkalmazás rugalmas tervezésekor ismernie kell a rendelkezésre állási követelményeinek. Mennyi állásidő fogadható el? Ez részben a költség függvénye. Mennyibe kerül a lehetséges állásidő az üzletének? Mennyit kell befektetnie az alkalmazás magas rendelkezésre állásúvá tételéhez?

Használja a [rugalmassági ellenőrzőlista] [ resiliency-checklist] rugalmassági szempontból a kialakítás áttekintése.

### <a name="resiliency-guidance"></a>A rugalmasságra vonatkozó útmutatás

- [Az Azure-rugalmas alkalmazások tervezése][resiliency]
- [Rugalmasság tervminták][resiliency-patterns]
- Gyakorlati tanácsok: [átmeneti hiba kezelési][transient-fault-handling], [ismételje meg az adott szolgáltatások útmutató][retry-service-specific]

## <a name="management-and-devops"></a>Felügyeleti és Devopok

Ez az oszlop megtartása éles környezetben futó alkalmazáshoz műveletek folyamatait ismerteti.

Központi telepítések megbízható és kiszámítható kell lennie. Akkor érdemes automatizálni, csökkenti annak esélyét keletkező emberi hibákat. Gyors és szokásos folyamat, így azok nem lelassíthatja az új szolgáltatásokat vagy hibajavítások kell. Egyaránt fontos, meg kell tudni gyorsan állíthatja vissza, illetve ha egy frissítés problémák vannak az előregörgetéseket.

Megfigyelési és diagnosztikai fontosságúak. Felhőalapú alkalmazások futtatása egy távoli adatközpontban Ha nem rendelkezik teljes körű vezérlés az infrastruktúra vagy bizonyos esetekben az operációs rendszer. A nagyméretű alkalmazások nincs gyakorlati problémákat vagy naplófájlok keletkezik virtuális gépek bejelentkezni. PaaS-szolgáltatásokkal akkor valószínűleg egy dedikált virtuális Gépet, hogy be tudjon jelentkezni. Megfigyelési és diagnosztikai biztosítják a rendszer betekintést megállapításához, hogy mikor és hol történt hiba. Minden rendszer megfigyelhető kell legyen. Egy általános és következetes naplózási séma, amely lehetővé teszi az események összefüggéseket rendszeren használható.

A figyelés és diagnosztika folyamat több különböző fázisa van:

- Instrumentation. A nyers adatok generálása az alkalmazásnaplókat, webkiszolgáló naplózza, az Azure platformon, és a más forrásokból épített diagnosztika.
- Adatgyűjtési és -tárolás. Egyesítse az adatok egy helyen.
- Elemzés és elemzés céljából. Problémák elhárításához, és tekintse meg az általános állapotát.
- A képi megjelenítés és riasztásokat. Telemetriai adatok segítségével tendenciákat, vagy a műveleti csapata riasztást.

Használja a [DevOps ellenőrzőlista] [ devops-checklist] áttekintheti a Tervező egy felügyeleti, a DevOps tükrözik.

### <a name="management-and-devops-guidance"></a>Útmutató felügyeleti és Devopok

- [Kezelési és figyelési tervminták][management-patterns]
- Gyakorlati tanácsok: [megfigyelési és diagnosztikai][monitoring]

## <a name="security"></a>Biztonság

Az alkalmazás, a tervezési és megvalósítási történő központi telepítése és üzemeltetése teljes életciklusát biztonságért át kell gondolni. Az Azure platform különböző fenyegetések, például hálózati behatolás és DDoS-támadások elleni védelmet biztosít. Azonban továbbra is készítéséhez szükséges biztonsági az alkalmazásba, és a DevOps folyamatokba.

Az alábbiakban néhány átfogó biztonsági kérdést kell figyelembe venni. 

### <a name="identity-management"></a>Identitáskezelés

Fontolja meg az Azure Active Directory (Azure AD) segítségével a felhasználók hitelesítéséhez és engedélyezéséhez. Az Azure AD egy olyan teljes körűen felügyelt identitások és hozzáférések felügyeleti szolgáltatás. Tartományok, amelyek csak az Azure-on létezik, vagy integrálható a helyszíni Active Directory identitások létrehozására használható. Az Azure AD is jól integrálható az Office365, a Dynamics CRM Online-hoz és a sok külső SaaS-alkalmazásokhoz. Felhasználók felé néző alkalmazásokhoz Azure Active Directory B2C segítségével a felhasználók azokkal a meglévő közösségi fiókok (például a Facebook, Google vagy LinkedIn), vagy hozzon létre egy új felhasználói fiókot az Azure AD által felügyelt.

Ha azt szeretné, a helyszíni Active Directory-környezeten integrálása az Azure-hálózatot, több módon is, a követelményeitől függően. További információkért lásd: a [Identity Management] [ identity-ref-arch] architektúrák hivatkozik.

### <a name="protecting-your-infrastructure"></a>Az infrastruktúra védelme 

Az üzembe helyezett Azure erőforrásokhoz való hozzáférés szabályozása. Minden Azure-előfizetéssel rendelkezik egy [megbízhatósági kapcsolat] [ ad-subscriptions] az Azure AD-bérlő. Használjon [szerepköralapú hozzáférés-vezérlés] [ rbac] (RBAC) adja meg a szervezeten belüli felhasználók Azure-erőforrások a megfelelő engedélyeket. Adjon hozzáférést RBAC szerepkört rendel felhasználókat és csoportokat egy adott hatókörben. A hatókör lehet előfizetés, egy erőforráscsoport vagy egy erőforrást. [Naplózási] [ resource-manager-auditing] infrastruktúra összes módosítását. 

### <a name="application-security"></a>Alkalmazások biztonsága

Alkalmazásfejlesztési ajánlott biztonsági eljárások általánosságban továbbra is érvényesek a felhőben. Ezek közé tartozik többek között az SSL tetszőleges helyszínről, CSRF, és lehetővé támadások elleni védelem megakadályozza az SQL injektálási támadásokat, és így tovább. 

Felhőalapú alkalmazások gyakran használnak a hívóbetűket felügyelt szolgáltatásokat. Soha ne ellenőrizze a verziókövetési rendszerrel be ezeket. Érdemes tárolni alkalmazás titkos kulcsok Azure Key Vault.

### <a name="data-sovereignty-and-encryption"></a>Adatok közös joghatóság alá és titkosítás

Győződjön meg arról, hogy az adatok továbbra is a megfelelő geopolitikai zónában amikor Azure használatával a magas rendelkezésre állású. Azure georeplikált tárolási fogalmát alkalmazza a [párosított régió] [ paired-region] geopolitikai ugyanabban a régióban. 

Key Vault segítségével megakadályozhatja a titkosítási kulcsok és titkos kulcsok. Key Vault használatával titkosíthatja kulcsok és titkos hardveres biztonsági modulokkal (HSM) védett kulcsokkal. Sok Azure storage és DB services támogatja az adatok titkosítását, beleértve a [Azure Storage][storage-encryption], [Azure SQL Database][sql-db-encryption], [ Azure SQL Data Warehouse][data-warehouse-encryption], és [Cosmos DB][cosmosdb-encryption].

### <a name="security-resources"></a>Biztonsági erőforrások

- [Az Azure Security Center] [ security-center] biztosít integrált biztonsági monitorozást és az Azure-előfizetések. 
- [Az Azure biztonsági dokumentációja][security-documentation]
- [A Microsoft biztonsági és adatkezelési központ][trust-center]



<!-- links -->

[dr-guidance]: ../resiliency/disaster-recovery-azure-applications.md
[identity-ref-arch]: ../reference-architectures/identity/index.md
[resiliency]: ../resiliency/index.md

[ad-subscriptions]: /azure/active-directory/active-directory-how-subscriptions-associated-directory
[data-warehouse-encryption]: /azure/data-lake-store/data-lake-store-security-overview#data-protection
[cosmosdb-encryption]: /azure/cosmos-db/database-security
[rbac]: /azure/active-directory/role-based-access-control-what-is
[paired-region]: /azure/best-practices-availability-paired-regions
[resource-manager-auditing]: /azure/azure-resource-manager/resource-group-audit
[security-blog]: https://azure.microsoft.com/blog/tag/security/
[security-center]: https://azure.microsoft.com/services/security-center/
[security-documentation]: /azure/security/
[sql-db-encryption]: /azure/sql-database/sql-database-always-encrypted-azure-key-vault
[storage-encryption]: /azure/storage/storage-service-encryption
[trust-center]: https://azure.microsoft.com/support/trust-center/
 

<!-- patterns -->
[availability-patterns]: ../patterns/category/availability.md
[management-patterns]: ../patterns/category/management-monitoring.md
[resiliency-patterns]: ../patterns/category/resiliency.md
[scalability-patterns]: ../patterns/category/performance-scalability.md


<!-- practices -->
[autoscale]: ../best-practices/auto-scaling.md
[background-jobs]: ../best-practices/background-jobs.md
[caching]: ../best-practices/caching.md
[cdn]: ../best-practices/cdn.md
[data-partitioning]: ../best-practices/data-partitioning.md
[monitoring]: ../best-practices/monitoring.md
[retry-service-specific]: ../best-practices/retry-service-specific.md
[transient-fault-handling]: ../best-practices/transient-faults.md


<!-- checklist -->
[availability-checklist]: ../checklist/availability.md
[devops-checklist]: ../checklist/dev-ops.md
[resiliency-checklist]: ../checklist/resiliency.md
[scalability-checklist]: ../checklist/scalability.md
