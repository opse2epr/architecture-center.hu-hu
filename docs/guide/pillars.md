---
title: A szoftverminőség alappillérei
description: A cikk a szoftverminőség öt alappillérét ismerteti, melyek a skálázhatóság, a rendelkezésre állás, a rugalmasság, a felügyelet és a biztonság.
author: MikeWasson
ms.openlocfilehash: 117706046ca1a9b7f3203a99737347809d0c323f
ms.sourcegitcommit: 85334ab0ccb072dac80de78aa82bcfa0f0044d3f
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/11/2018
ms.locfileid: "35252787"
---
# <a name="pillars-of-software-quality"></a>A szoftverminőség alappillérei 

Egy sikeres felhőalkalmazás a szoftverminőség következő öt alappillérére koncentrál: skálázhatóság, rendelkezésre állás, rugalmasság, felügyelet és biztonság.

| Pillér | Leírás |
|--------|-------------|
| Méretezhetőség | A rendszer megnövekedett terhelés kezelésére vonatkozó képessége. |
| Rendelkezésre állás | Az az időarány, amíg a rendszer működik és üzemel. |
| Rugalmasság | A rendszer azon képessége, hogy helyreálljon a hibák után, és folytassa a működést. |
| Kezelés | A rendszert termelési állapotban tartó működési folyamatok. |
| Biztonság | Az alkalmazások és adatok védelme a fenyegetésekkel szemben. |

## <a name="scalability"></a>Méretezhetőség

A méretezhetőség a rendszernek a megnövekedett terhelés kezelésére vonatkozó képessége. Az alkalmazásokat két fő módszerrel lehet skálázni. A vertikális skálázás (*felskálázás*) az erőforrások kapacitásának növelését jelenti, például egy nagyobb méretű virtuális gép használatával. A horizontális skálázás (*bővítés*) új erőforráspéldányok hozzáadását jelenti, például virtuális gépeket vagy adatbázis-replikákat. 

A horizontális skálázásnak jelentős előnyei vannak a vertikális skálázással szemben:

- Valódi felhőbeli skálázás. Az alkalmazások megtervezhetők úgy, hogy több száz vagy akár több ezer csomóponton fussanak, olyan léptéket elérve, amely egyetlen csomóponton nem lehetséges.
- A horizontális skálázás rugalmas. Hozzáadhat több példányt, ha növekszik a terhelés, a csendesebb időszakokban pedig eltávolíthatja azokat.
- A horizontális skálázás aktiválható automatikusan, ütemezés szerint vagy a terhelés változásaira reagálva. 
- A horizontális skálázás olcsóbb is lehet, mint a vertikális. Több kis méretű virtuális gép futtatása kevesebb költséggel jár, mint egy nagy méretűé. 
- A horizontális skálázás a rugalmasságot is javíthatja a redundancia révén. Ha egy példány leáll, az alkalmazás továbbra is működik.

A vertikális skálázás előnye, hogy elvégezhető az alkalmazás módosítása nélkül is. Egy bizonyos ponton azonban el fog érni egy korlátot, ahonnan már nem lehetséges a további felskálázás. Ezen a ponton a további skálázás csak horizontális lehet. 

A horizontális skálázást bele kell tervezni a rendszerbe. A virtuális gépek horizontális skálázásához például a virtuális gépek egy terheléselosztó mögé helyezhetők. A készlet mindegyik virtuális gépének képesnek kell lennie azonban arra, hogy bármilyen ügyfélkérést kezeljen, ezért az alkalmazásnak állapot nélkülinek kell lennie, vagy külsőleg kell tárolnia az állapotokat (például egy elosztott gyorsítótárban). A felügyelt PaaS-szolgáltatások gyakran rendelkeznek beépített horizontális skálázással és automatikus skálázással. A szolgáltatások egyszerű skálázhatósága a PaaS-szolgáltatások használatának egyik jelentős előnye.

Egyszerűen több példány hozzáadása azonban nem jelenti azonban azt, hogy az alkalmazás skálázható lesz. Előfordulhat, hogy egyszerűen csak máshol alakul ki a szűk keresztmetszet. Ha például egy webes előteret több ügyfélkérés kezelésére skálázik, az zárolási versenyt válthat ki az adatbázisban. Ezután mérlegelni kell olyan további intézkedések megtételét, mint az optimista egyidejűség vagy az adatparticionálás, nagyobb átviteli sebesség biztosítása érdekében az adatbázis felé.

A potenciális szűk keresztmetszetek kiszűréséhez mindig végezzen teljesítmény- és terheléstesztelést. A szűk keresztmetszeteket leggyakrabban a rendszer állapottal rendelkező összetevői, például az adatbázisok okozzák, ezért gondosan meg kell tervezni a horizontális skálázásukat. Egy szűk keresztmetszet elhárítása további szűk keresztmetszeteket tárhat fel máshol.

A [skálázhatósági ellenőrzőlista][scalability-checklist] alapján áttekintheti a terveit skálázhatósági szempontból.

### <a name="scalability-guidance"></a>Skálázhatósági útmutató

- [Tervezési minták a megfelelő skálázhatósághoz és teljesítményhez][scalability-patterns]
- Ajánlott eljárások: [Automatikus skálázás][autoscale], [háttérfeladatok][background-jobs], [gyorsítótárazás][caching], [CDN][cdn], [adatparticionálás][data-partitioning].

## <a name="availability"></a>Rendelkezésre állás

A rendelkezésre állás az az időarány, amíg a rendszer működik és üzemel. Általában az üzemidő százalékaként értendő. Az alkalmazáshibák, infrastruktúraproblémák és a rendszerterhelés egyaránt csökkenthetik a rendelkezésre állást. 

A felhőalapú alkalmazásoknak rendelkezniük kell szolgáltatásiszint-célkitűzéssel (service level objective, SLO), amely egyértelműen meghatározza az elvárt rendelkezésre állást és annak mérését. A rendelkezésre állás meghatározásakor a kritikus útvonalat kell figyelembe venni. Lehet, hogy a webes előtér képes kiszolgálni az ügyfélkéréseket, ha azonban minden tranzakció sikertelen, mert az előtér nem tud kapcsolódni az adatbázishoz, akkor az alkalmazás nem áll a felhasználók rendelkezésére. 

A rendelkezésre állást gyakran „kilencesekkel” írják le, például a „négy 9-es” 99,99%-os üzemidőt jelent. Az alábbi táblázat a különböző rendelkezésre állási szintek lehetséges halmozott állásidejét mutatja be.

| Üzemidő %-ban | Állásidő hetente | Állásidő havonta | Állásidő évente |
|----------|-------------------|--------------------|-------------------|
| 99% | 1,68 óra | 7,2 óra | 3,65 nap |
| 99.9% | 10 perc | 43,2 perc | 8,76 óra |
| 99.95% | 5 perc | 21,6 perc | 4,38 óra |
| 99.99% | 1 perc | 4,32 perc | 52,56 perc |
| 99.999% | 6 másodperc | 26 másodperc | 5,26 perc |

Vegye figyelembe, hogy a 99%-os üzemidő hetente majdnem 2 óra szolgáltatáskimaradást jelenthet. Számos alkalmazás, különösen a felhasználók felé néző alkalmazások esetén ez az SLO nem fogadható el. Másrészről öt 9-es (99,999 %) legfeljebb 5 perc állásidőt jelent *évente*. Egy kimaradást ilyen gyorsan észlelni is kihívást jelent, nemhogy meg is oldani a problémát. A nagyon magas (99,99% vagy nagyobb) rendelkezésre állás eléréséhez nem támaszkodhat manuális beavatkozásra a hibákból való helyreállításhoz. Az alkalmazásnak képesnek kell lennie az öndiagnózisra és az önjavításra. Ekkor válok kritikus fontosságúvá a rugalmasság.

Az Azure-ban a szolgáltatási szerződés (SLA) ismerteti a Microsoftnak az üzemidővel és hálózati elérhetőséggel kapcsolatos vállalásait. Ha egy adott szolgáltatás SLA-ja 99,95%-os, az azt jelenti, hogy a szolgáltatás várhatóan az idő 99,95%-ában rendelkezésre áll.

Az alkalmazások általában több szolgáltatástól is függenek. Az egyes szolgáltatások leállásának valószínűsége általában egymástól független. Tegyük fel például, hogy egy alkalmazás két olyan szolgáltatástól függ, amelyek SLA-ja egyaránt 99,9 %. A két szolgáltatás összesített SLA-ja 99,9% &times; 99,9% &asymp; 99,8%, vagyis valamivel kisebb, mint az egyes szolgáltatások saját SLA-ja. 

A [rendelkezésre állási ellenőrzőlista][availability-checklist] alapján áttekintheti a terveit rendelkezésre állási szempontból.

### <a name="availability-guidance"></a>Rendelkezésre állási útmutató

- [Tervezési minták a rendelkezésre álláshoz][availability-patterns]
- Ajánlott eljárások: [Automatikus skálázás][autoscale], [háttérfeladatok][background-jobs].

## <a name="resiliency"></a>Rugalmasság

A rugalmasság a rendszer azon képessége, hogy helyreálljon a hibák után, és folytassa a működést. A rugalmasság célja, hogy az alkalmazás egy hibát követően teljesen működőképes állapotba térjen vissza. A rugalmasság szorosan kapcsolódik a rendelkezésre álláshoz.

A hagyományos alkalmazásfejlesztés során a hangsúly a hibák közötti átlagos idő csökkentésén volt. Az erőfeszítések arra irányultak, hogy a rendszer ne hibásodjon meg. A felhőalapú informatika másik megközelítést igényel több tényező miatt:

- Az elosztott rendszerek összetettek, és egy adott ponton bekövetkező hiba az egész rendszerre kihathat.
- A felhőalapú környezetek költségei közös hardverek használatával alacsonyan tarthatók, így számítani lehet a hardverek időnkénti meghibásodására. 
- Az alkalmazások általában külső szolgáltatásoktól is függenek, amelyek ideiglenesen elérhetetlenné válhatnak, vagy nagy forgalmú felhasználókat szabályozhatnak. 
- Napjainkban a felhasználók elvárják, hogy az alkalmazások a nap 24 órájában elérhetők legyenek és soha ne legyenek offline.

Mindezen tényezők miatt a felhőalapú alkalmazásokat úgy kell megtervezni, hogy számítsanak az időnként előforduló hibákra, és azokból helyre tudjanak állni. Az Azure számos beépített rugalmassági funkcióval rendelkezik. Például: 

- Az Azure Storage, az SQL Database és a Cosmos DB egyaránt biztosítanak beépített adatreplikációt, egy adott régión belül és régiók között is.
- Az Azure Managed Disks felügyelt lemezi automatikusan különböző tárolóskálázási egységekbe kerülnek a hardverhibák hatásának csökkentése érdekében.
- Az egyes rendelkezésre állási csoportokhoz tartozó virtuális gépek több tartalék tartományon vannak elosztva. A tartalék tartományok az azonos tápforrással és hálózati kapcsolóval rendelkező virtuális gépek csoportjai. A virtuális gépek több tartalék tartományon való elosztása korlátozza a hardvermeghibásodások, hálózatkimaradások vagy a tápellátás megszakadásának hatását.

Említett, továbbra is szeretné rugalmassági build az alkalmazásba. Rugalmassági stratégiák az architektúra minden szintjén alkalmazhatók. Bizonyos megoldások inkább taktikai jellegűek – például egy távoli hívás újrapróbálása egy átmeneti hálózati hiba után. Más megoldások inkább stratégiai jellegűek, például egy teljes alkalmazás feladatátvétele egy másodlagos régióba. A taktikai megoldások nagy hatással járhatnak. Jóllehet ritkán fordul elő, hogy egy teljes régió kimaradást tapasztaljon, az átmeneti problémák, például a hálózati torlódás azonban gyakoribbak, így először ezekre kell felkészülni. Fontos a megfelelő monitorozás és diagnosztika megléte is a bekövetkező hibák észlelése és a kiváltó okok felderítése érdekében.

Amikor egy alkalmazást rugalmasnak tervez meg, ismernie kell a rendelkezésreállási követelményeket. Mennyi állásidő fogadható el? Ez részben a költség függvénye. Mennyibe kerül a lehetséges állásidő az üzletének? Mennyit kell befektetnie az alkalmazás magas rendelkezésre állásúvá tételéhez?

A [rugalmassági ellenőrzőlista][resiliency-checklist] alapján áttekintheti a terveit rugalmassági szempontból.

### <a name="resiliency-guidance"></a>A rugalmasságra vonatkozó útmutatás

- [Rugalmas alkalmazások tervezése az Azure-hoz][resiliency]
- [Tervezési minták a rugalmassághoz][resiliency-patterns]
- Ajánlott eljárások: [Átmeneti hibák kezelése][transient-fault-handling], [újrapróbálkozási útmutatás adott szolgáltatásoknál][retry-service-specific].

## <a name="management-and-devops"></a>Felügyelet és fejlesztés és üzemeltetés

Ez a pillér azokat a működési folyamatokat takarja, amelyek biztosítják egy alkalmazás futását az éles környezetben.

Az üzemelő példányoknak megbízhatónak és kiszámíthatónak kell lenniük. Automatizálni kell őket az emberi hibák esélyének csökkentése érdekében. Gyors és rutinszerű folyamatoknak kell lenniük, amelyek nem lassulnak le az új funkciók vagy hibajavítások megjelenésekor. Hasonlóan fontos, hogy ha egy frissítéssel problémák adódnak, a rendszer képes legyen gyorsan visszaállni vagy továbblépni.

A monitorozás és a diagnosztika létfontosságú. A felhőalapú alkalmazások egy távoli adatközpontban futnak, ahol nem lehetséges teljesen uralni az infrastruktúrát, vagy bizonyos esetekben, az operációs rendszert. A nagyméretű alkalmazások esetén nem célszerű minden egyes virtuális gépre bejelentkezni egy hiba elhárítása vagy a naplófájlok áttekintése céljából. A PaaS-szolgáltatások esetén az is előfordulhat, hogy nincs is olyan dedikált virtuális gép, amelyre be lehetne jelentkezni. A monitorozás és diagnosztika a rendszerrel kapcsolatos adatokat biztosítanak, amelyekből megállapítható, hogy a hibák mikor és hol történnek. Minden rendszernek megfigyelhetőnek kell lennie. Használjon egy általános és következetes naplózási sémát, amely lehetővé teszi az események összevetését a rendszerek tekintetében.

A monitorozás és diagnosztika folyamata több különálló fázisból áll:

- Rendszerállapot. A nyers adatok létrehozása az alkalmazásnaplókból, webkiszolgálói naplókból, az Azure platform beépített diagnosztikai eszközeiből és egyéb forrásokból.
- Gyűjtés és tárolás. Az adatok konszolidálása egy helyen.
- Elemzés és diagnosztika. A problémák elhárításához és a rendszer általános állapotának áttekintéséhez.
- Vizualizáció és riasztások. A telemetriaadatok használata a tendenciák kimutatásához és az üzemeltetési csapatok riasztásához.

A [fejlesztési és üzemeltetési ellenőrzőlista][devops-checklist] alapján áttekintheti a terveit a felügyelet és a fejlesztés és üzemeltetés szempontjából.

### <a name="management-and-devops-guidance"></a>Felügyeleti és fejlesztési és üzemeltetési útmutató

- [Tervezési minták a felügyelethez és monitorozáshoz][management-patterns]
- Ajánlott eljárások: [Monitorozás és diagnosztika][monitoring].

## <a name="security"></a>Biztonság

A biztonsági megoldásokat alaposan át kell gondolni az alkalmazás teljes életciklusára vonatkozóan, a tervezéstől és implementálástól a központi telepítésig és üzemeltetésig. Az Azure platform védelmet nyújt különböző fenyegetések, például hálózati behatolások és DDoS-támadások ellen. Az alkalmazásokba és a fejlesztési és üzemeltetési folyamatokba azonban így is be kell építeni biztonsági megoldásokat.

Az alábbiak néhány tágabb, megfontolásra érdemes biztonsági területről olvashat. 

### <a name="identity-management"></a>Identitáskezelés

Fontolja meg az Azure Active Directory (Azure AD) használatát a felhasználók hitelesítéséhez és engedélyezéséhez. Az Azure AD egy teljes körűen felügyelt identitás- és hozzáférés-felügyeleti szolgáltatás. A használatával létrehozhatók tartományok, amelyek csak az Azure-on léteznek, vagy integrálhatók a helyszíni Active Directory-identitásokkal. Az Azure AD emellett integrálható az Office365 és a Dynamics CRM Online szolgáltatással, valamint számos külső SaaS-alkalmazással is. A felhasználók felé néző alkalmazások esetén az Azure Active Directory B2C lehetővé teszi a felhasználók számára, hogy a hitelesítéshez a meglévő közösségi fiókjaikat használják (például Facebook, Google vagy LinkedIn), vagy létrehozzanak egy új, az Azure AD által felügyelt fiókot.

Ha integrálni szeretné a helyszíni Active Directory-környezetet egy Azure-hálózattal, számos megközelítés lehetséges a követelményei alapján. További információért tekintse meg az [identitáskezelési][identity-ref-arch] referenciaarchitektúrákat.

### <a name="protecting-your-infrastructure"></a>Az infrastruktúra védelme 

Szabályozza az üzembe helyezett Azure-erőforrásokhoz való hozzáférést. Minden Azure-előfizetés [bizalmi kapcsolattal][ad-subscriptions] rendelkezik egy Azure AD-bérlővel. A [szerepköralapú hozzáférés-vezérléssel][rbac] (Role-Based Access Control, RBAC) szabályozható a vállalat felhasználóinak hozzáférése az üzembe helyezett Azure-erőforrásokhoz. A hozzáférés biztosításához RBAC-szerepköröket rendelhet felhasználókhoz, csoportokhoz és alkalmazásokhoz egy bizonyos hatókörben. A hatókör előfizetés, erőforráscsoport vagy egyetlen erőforrás lehet. [Naplózza][resource-manager-auditing] az infrastruktúra összes módosítását. 

### <a name="application-security"></a>Alkalmazások biztonsága

Általánosságban véve a biztonsághoz kapcsolódó ajánlott eljárások a felhőbeli alkalmazásfejlesztésre is érvényesek. Ezek közé tartozik többek között az SSL használata mindenhol, a védekezés CSRF- és XSS-támadások ellen, az SQL-injektálási támadások megelőzése stb. 

A felhőalkalmazások gyakran használnak hozzáférési kulcsokkal rendelkező felügyelt szolgáltatásokat. Soha ne tárolja ezeket a hozzáférési kulcsokat a forráskezelőben. Vegye fontolóra az alkalmazások titkos kulcsainak az Azure Key Vaultban történő tárolását.

### <a name="data-sovereignty-and-encryption"></a>Az adatok elkülönítése és titkosítása

Ügyeljen arra, hogy az adatai a megfelelő geopolitikai zónában maradjanak az Azure magas rendelkezésre állású szolgáltatásainak használatakor. Az Azure georeplikált tárolása a [párosított régió][paired-region] koncepcióját használja az ugyanazon geopolitikai régión belül. 

A Key Vault használatával védje a titkosítási kulcsokat és a titkos kulcsokat. A Key Vault lehetővé teszi, hogy hardveres biztonsági modulokkal (HSM) védett kulcsokkal titkosítsa a kulcsokat és a titkos kulcsokat. A tárolt adatok titkosítását számos Azure-beli tároló és adatbázis-szolgáltatás támogatja, köztük az [Azure Storage][storage-encryption], az [Azure SQL Database][sql-db-encryption], az [ Azure SQL Data Warehouse][data-warehouse-encryption] és a [Cosmos DB][cosmosdb-encryption].

### <a name="security-resources"></a>Biztonsággal kapcsolatos információforrások

- Az [Azure Security Center][security-center] az ügyfelek összes Azure-előfizetésére kiterjedő, integrált biztonsági monitorozást és szabályzatkezelést biztosít. 
- [Az Azure Security dokumentációja][security-documentation]
- [Microsoft adatvédelmi központ][trust-center]



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
