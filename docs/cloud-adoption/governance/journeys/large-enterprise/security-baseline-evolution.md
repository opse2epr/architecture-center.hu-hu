---
title: 'CAF: Nagyvállalati – alapvető biztonsági fejlődést szem előtt tartva '
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Nagyvállalati – alapvető biztonsági fejlődést szem előtt tartva
author: BrianBlanchard
ms.openlocfilehash: e9e8b7dc1eaeb3a8555326a51b2548ad668e0171
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298442"
---
# <a name="large-enterprise-security-baseline-evolution"></a>Nagyvállalati: A biztonsági alapkonfiguráció fejlődése

Ez a cikk a narratíva haladásával biztonsági ellenőrzéseket, amely támogatja a védett adatokról a felhőre való hozzáadásával.

## <a name="evolution-of-the-narrative"></a>A narratíva fejlődésére

A CIO fordított együttműködés a munkatársakkal és a vállalat jogi személyzet hónap. A kiberbiztonsági szakértelmet felügyeleti tanácsadó alkalmazták a meglévő IT-biztonsági segítségével, és az informatikai szabályozással csapatok vázlatszintű új házirenddel védett adatok. A csoport tudta elősegítik a meglévő házirendet, lecserélése a tábla támogatási személyazonosításra alkalmas adatok és a pénzügyi adatok jóváhagyott felhőszolgáltatók által üzemeltetett lehet. Ez szükséges biztonsági követelményeket és a egy cégirányítási bevezetése feldolgozni a, és hogy ezek a házirendek dokumentálása.

Az elmúlt 12 hónap a felhő bevezetésének csapatok törölte a legtöbb az 5000 eszközök a két adatközpont-án megszűnik. A 350 nem kompatibilis eszközök lett áthelyezve, egy másik adatközpontjára. Csak a 1,250 tartalmazó virtuális gépek védett adatok továbbra is.

### <a name="evolution-of-the-cloud-governance-team"></a>A felhő Cégirányítási csapat a fejlődést szem előtt tartva

A felhő Cégirányítási csapat továbbra is fejlesztheti tovább a narratíva együtt. A két igazgatótanácsi tagja a csoport immár legtiszteletreméltóbb a felhőben dolgozó tervezők, a vállalat között. Nőtt. a konfigurációs szkripteket gyűjteményét, új teams kiszolgálókon innovatív új üzembe helyezésekhez. A felhő Cégirányítási csapata is emelkedett. Legutóbb az IT-üzemeltetési csapat tagjai csatlakoztak a felhő Cégirányítási csapat tevékenységek fel a felhőbeli műveletek. A felhőben dolgozó tervezők t létrehozó szakemberektől tanulhat, ezáltal elősegíti az ebben a közösségi felhő őrei és a felhő megoldásgyorsítók láthatók.

Bár a különbség a változás is, azt fontos különbség létrehozását, cégirányítási való használatra optimalizált informatikai kulturális környezetet. A felhő gondviselője megtisztítja a kantinjaik innovatív felhőben dolgozó tervezők által készített, és a két lehetséges szerepkör természetes fennakadások nélkül használható, és célok ellentétes. Egy felhőalapú gazdagépőr segít biztonságban a felhőben, így más felhőben dolgozó tervezők gyorsabban és kevesebb kantinjaik helyezheti át. Egy felhőalapú gyorsító mindkét funkciókat hajtja végre, de is részt vesz a gyorsabb üzembe helyezési és bevezetési, váljon az innováció gyorsító, valamint a felhő irányítási öt oktatnak a defender-sablonok létrehozása.

### <a name="evolution-of-the-current-state"></a>Az aktuális állapot a fejlődést szem előtt tartva

Ez narratíva korábbi szakaszában a vállalat kezdte két adatközpont kivonása folyamatán. A folyamatos erőfeszítés magában foglalja az örökölt hitelesítési követelmények, amely az identitásra építve, ismertetett továbbfejlesztett változata szükséges az egyes alkalmazások migrálása a [előző cikk](identity-baseline-evolution.md).

Azóta, néhány dolog változott, amely hatással lesz a cégirányítási:

- Több ezer informatikai és üzleti eszközök alkalmazott a felhőbe.
- Az alkalmazásfejlesztési csoportnak van megvalósítva, folyamatos integráció és készregyártás (CI/CD) folyamat üzembe helyezése felhőbeli natív jobb felhasználói élményt. Alkalmazás nem interakcióba védett adatok, így nem éles üzemre.
- Az üzleti intelligencia csapat belül informatikai aktívan programnyelveket logisztika, a leltárral és a külső adatok a felhőben lévő adatokat. Ezeket az adatokat használja fel új előrejelzéseket, üzleti folyamatok sikerült formázása. Azonban ezen előrejelzéseket és insights azokat nem döntéstámogató ügyfél és a pénzügyi adatok integrálhatók a data platform.
- Az informatikai csapat két adatközpont kivonása az a CIO és a pénzügyi igazgató az Igazgatókra folyik. Az eszközök a két adatközpontban szinte 3500 elavult, vagy át.
- A házirendek lefektetése személyazonosításra alkalmas adatok és pénzügyi adatok rendelkezik lett modernizálta. Azonban az új vállalati házirendeket is függ, a kapcsolódó biztonsági és cégirányítási szabályzatok megvalósítását. Csoportok továbbra is leáll.

### <a name="evolution-of-the-future-state"></a>A jövőbeli állapot a fejlődést szem előtt tartva

- Az alkalmazások fejlesztését és BI Csapatok korai kísérletek rendelkezik látható megtalálhatja a javítási lehetőségeket az ügyfeleknek, környezetet és az adatvezérelt döntések. Mindkét csapatok, ha a következő 18 hónapban a felhő bevezetésének bontsa ezen megoldások éles környezetben történő telepítésével.
- Informatikai öt további adatközpontok áttelepítése az Azure-ban, amely tovább csökkentheti informatikai költségeit, és adja meg a nagyobb üzleti agilitás üzleti indoklásának fejlesztett ki. Bár kisebb méretezési csoportban, ezek az adatközpontok használatból való kivonást egyaránt várhatóan duplán a teljes költséget takaríthat meg.
- A szükséges biztonsági és cégirányítási szabályzatokat, eszközök és folyamatok megvalósításához beruházna és üzemeltetési költségek mellett költségvetések hagyott jóvá. A várt költségmegtakarításokat, az Adatközpont használatból való kivonást egyaránt az fizetnie ezen új kezdeményezést több mint elegendő is. Informatikai és üzleti vezető vagyunk abban a befektetés, annak érdekében, biztosítson a más területeken értéket ad vissza. Az ezen Felhőbeli Cégirányítási csapat egy felismert csapat dedikált vezetői és volt szükség létszámnövelésre vált.
- A cloud bevezetési csoportok, felhőalapú Cégirányítási csapat, IT-biztonsági csoportja és az informatikai szabályozással csapat együttesen érvényesítik az, hogy a felhő bevezetésének csapatok számára a védett adatok áttelepítése a felhőbe biztonsági és cégirányítási követelmények.

## <a name="evolution-of-tangible-risks"></a>A képzés résztvevői hasznos képességekkel kockázatok a fejlődést szem előtt tartva

**Adatok illetéktelen behatolás**: Nincs kötelezettség és a kapcsolódó adatok bármely új adatplatformra elfogadásakor rejlő növekedését. A felhőalapú technológiák bevezetése szerelők feladatkörei, ami csökkentheti a kockázat-megoldások megvalósításának nőtt. Annak érdekében, hogy ezeket a technikusok feladatok teljesítése egy hatékony biztonsági és szabályozási stratégiát kell végrehajtani.

Az üzleti kockázat néhány műszaki kockázatok bővíthet:

- Alapvető fontosságú alkalmazások vagy a védett adatok véletlenül is telepíthetők.
- Védett adatok tárolása miatt gyenge titkosítás döntések során előfordulhat, hogy ki lehetnek téve.
- Jogosulatlan felhasználók is hozzáférhetnek a védett adatok.
- Külső behatolás védett adatokhoz való hozzáférést eredményezhet.
- Külső behatolás vagy szolgáltatásmegtagadási támadások ellen egy üzleti megszakítás eredményezheti.
- Szervezet vagy a munkaviszony módosításokat a védett adatokhoz való jogosulatlan hozzáférést tehet lehetővé.
- Új biztonsági rések előfordulhat, hogy hozzon létre lehetőségeit betörésre vagy illetéktelen hozzáférést.
- Inkonzisztens központi telepítési folyamat, amely az adatszivárgás vagy -megszakítások kezelése vezethet biztonsági rések eredményezhet.
- Konfigurációs csúszásokat vagy kihagyott javítások nem kívánt biztonsági rések, így adatszivárgás vagy -megszakítások kezelése eredményezhet.
- Különálló edge-eszközök növelheti a hálózat az üzemeltetés költségeit.
- Különálló eszközkonfigurációkat figyelmetlenség konfigurációban és a biztonság sérüléseinek vezethet.
- A Kiberbiztonsági csapata insists annak a kockázata, a szállító zárolási az egyetlen felhőszolgáltató platform előállító titkosítási kulcsok a. Bár ez a jogcím alaptalan, a csapat jelenleg elfogadták.

## <a name="evolution-of-the-policy-statements"></a>A házirend-utasítások fejlődésére

Az alábbi eltérésekkel házirend segítségével csökkentheti a kockázatokat és útmutató új végrehajtása. A lista megjelenését hosszú, de az ezek a házirendek bevezetését könnyebb, mint akkor jelenik meg.

1. Az összes telepített erőforrás kell lennie az alapján kategorizáljuk, kritikusság és az adatok besorolás. Besorolások vannak, át kell néznie a felhő Cégirányítási csapatot és az alkalmazás a felhőbe való üzembe helyezés előtt.
2. Az alkalmazásokat, amelyek tárolni, vagy hozzá a védett adatokhoz is eltérően, amelyek nem kezelhető. Minimális, érdemes lehet szegmentált a védett adatok jogosulatlan hozzáférés elkerülése érdekében.
3. Védett adatok inaktív állapotban titkosítani kell.
4. Emelt szintű engedélyekkel a védett adatokat tartalmazó minden szegmensben kivételt kell lennie. Az ilyen kivételek lesz rögzítve a Cloud Cégirányítási csapatával, és rendszeresen ellenőrzi.
5. Alhálózatok, amely tartalmazza a védett adatok más alhálózatokkal elkülönítve kell lennie. Védett adatok az alhálózatok közötti hálózati adatforgalom rendszeresen naplózva lesz.
6. Nincs alhálózat, amely tartalmazza a védett adatok közvetlenül elérhetők a nyilvános interneten keresztül vagy az adatközpontok között. Köztes alhálózati works keresztül hozzáférést ezekhez az alhálózatokhoz kell átirányítani. Ezekhez az alhálózatokhoz való hozzáférés teljes csomag vizsgálatát, és blokkolja a függvények által elvégezhető tűzfal megoldás segítségével kell származnia.
7. Irányítás azokat az eszközöket kell naplózása és érvényesíti a hálózati konfigurációs követelményei határozzák meg a biztonsági csapat.
8. Cégirányítási eszközök által jóváhagyott lemezképeket csak a virtuális gép üzembe helyezése kell korlátozni.
9. Amikor csak lehetséges, csomópont-konfiguráció kezelése bármely vendég operációs rendszer konfigurációja kell alkalmazni házirend követelményeinek. Csomópont-konfiguráció felügyelet be kell tartani a meglévő befektetések a csoportházirend-objektumot (GPO) az erőforrás-konfigurációhoz.
10. Cégirányítási eszközök ellenőrizni fogja, hogy az automatikus frissítések engedélyezve vannak-e az összes telepített erőforrás. Ha lehetséges, az automatikus frissítéseket alkalmazza. Nem kényszeríti ki azokat az eszközöket, amikor csomópont-szintű szabálysértések kell tekintse át a felügyeleti csoportok és műveletek házirendek megfelelően javítja. Eszközök, amelyek nem frissülnek automatikusan a tulajdonosa az IT-üzemeltetési folyamatok szerepelnie kell.
11. Az új előfizetési lehetőségekről, vagy bármilyen üzleti szempontból alapvető fontosságú alkalmazások és a védett adatok felügyeleti csoportok létrehozását egy át a felhőbe Cégirányítási csapatától megfelelő tervezet-hozzárendelést a kell.
12. A minimális jogosultságon alapuló hozzáférés-modell, bármely előfizetéssel, amely üzleti szempontból kritikus alkalmazásokat tartalmaz, vagy a védett adatok lépnek érvénybe.
13. A felhő szállító integrálható a meglévő helyszíni megoldás által felügyelt titkosítási kulcsok kell lennie.
14. A felhő szállító képes a peremhálózati eszköz a meglévő megoldást kell lennie, és minden kötelező konfigurációs előírást megvédéséhez nyilvánosan elérhetővé tett hálózathatár.
15. A felhő szállító az adatátvitelt az edge meglévő megoldás keresztül irányítja át a globális WAN megosztott kapcsolatot támogató képesnek kell lennie.
16. Trendek és az azokat kihasználó támadások ellen, amelyek hatással lehetnek a magánfelhők számára meg kell vizsgálni rendszeresen a biztonsági csapat számára a felhőben használt alapvető biztonsági eszközök által.
17. Központi telepítési eszközkészlet jóvá kell hagynia a felhő Cégirányítási csapatától folyamatos irányítás az üzembe helyezett eszközök.
18. Üzembehelyezési szkriptek kell fenntartani egy központi tárházban elérhető-e a felhő Cégirányítási csapat rendszeres áttekintése és a naplózási.
19. Cégirányítási folyamatok naplózások üzembehelyezési ponton kell megadni, és az összes eszköz közötti összhangot rendszeres ciklusokat, tartalmaznia kell.
20. Minden olyan ügyfél-hitelesítést igénylő alkalmazások üzembe helyezését, amely kompatibilis a belső felhasználók számára az elsődleges identitásszolgáltató jóváhagyott identitásszolgáltatót kell használnia.
21. Felhőalapú Cégirányítási folyamatok tartalmaznia kell a negyedéves ismertetők az identitásra építve csapatok számára a rosszindulatú vagy, meg kell akadályozni a felhő-eszköz konfigurációja használati minták azonosításához.

## <a name="evolution-of-the-best-practices"></a>Ajánlott eljárást a fejlődést szem előtt tartva

A cikk ezen szakasza a cégirányítási MVP megtervezése és az Azure új szabályzatokat és az Azure Cost Management implementációja fejlődik. Két kialakítási módosítások együtt, a vállalati szabályzat új utasításokat fog teljesítéséhez.

Az új ajánlott eljárásokat két kategóriába sorolhatók: Vállalati IT (Hub) és a felhő bevezetésének (irány).

**Egy vállalati informatikai hub/küllő előfizetés központosíthatja az alapvető biztonsági létrehozó**: Ez a legjobb, az a meglévő cégirányítási kapacitás burkolt be, egy [Hub küllő topológia közös szolgáltatásokkal][shared-services], a felhő Cégirányítási csapat néhány kulcsfontosságú kiegészítésekkel.

1. Az Azure DevOps-adattár. Egy tárház létrehozása az Azure DevOps tároló és a verziónak megfelelő Azure Resource Manager-sablonok és a parancsfájlalapú konfigurációk
2. Küllős sablont.
    1. Az útmutató a [a megosztott szolgáltatások Referenciaarchitektúra Küllős] [ shared-services] az eszközök egy vállalati IT-központ szükséges a Resource Manager-sablonok létrehozására használható.
    2. Használja ezeket a sablonokat, ez a struktúra lehet tenni megismételhető, egy központi adatirányítási stratégia részeként.
    3. A jelenlegi referenciaarchitektúra mellett célszerű, hogy egy hálózati biztonsági csoport (NSG) sablont kell létrehozni a virtuális hálózathoz, a tűzfal üzemeltetésére vonatkozó port blokkolja-e vagy engedélyezési követelmények rögzítése. Ez az NSG különbözni fog előzetes NSG-k, mert az első NSG nyilvános forgalom engedélyezésére egy vnetbe fogja.
3. Az Azure-házirendek létrehozása. Hozzon létre egy házirendet nevű `Hub NSG Enforcement` kényszeríteni az NSG-hez rendelt bármely ebben az előfizetésben létrehozott virtuális hálózat konfigurációját. A beépített szabályzatok Vendég konfiguráció a következőképpen alkalmazni:
    1. Naplózza, hogy a Windows webkiszolgálók biztonságos kommunikációs protokollokat használ.
    2. Naplózza, hogy a jelszó biztonsági beállításai helyesen vannak-e beállítva a Linux és Windows-gépeken belül.
4. Vállalati informatikai tervezet
    1. Hozzon létre egy Azure tervezet nevű `corporate-it-subscription`.
    2. Adja meg az eseményközpont/küllő sablonok és a Hub NSG házirend.
5. A kezdeti felügyeleti csoport-hierarchia bővítése.
    1. Minden egyes felügyeleti csoport által kért a védett adatok támogatása a `corporate-it-subscription-blueprint` tervezet gyorsított hub megoldást nyújt.
    2. Ebben a példában kitalált felügyeleti csoportok közé tartozik a regionális hierarchia mellett egy üzleti egység hierarchiája, mert ez a megoldás telepíti az egyes régiókban.
    3. A felügyeleti csoport hierarchia minden olyan régióhoz, nevű előfizetést hoz létre `Corporate IT Subscription`.
    4. Alkalmazza a `corporate-it-subscription-blueprint` tervezet egyes regionális példányokhoz.
    5. Egy központ meghatározzák az egyes régiókban mindegyik üzleti egység számára. Megjegyzés: További költségek csökkentését, elért, de a megosztási hubs különböző üzleti egységek az egyes régiókban lehet.
6. Integrálhatja a csoportházirend-objektumok (GPO) Desired State Configuration (DSC) rétegen keresztül:
    1. Csoportházirend-objektum átalakítása DSC – a [Microsoft Baseline felügyeletére irányuló projektnek](https://github.com/Microsoft/BaselineManagement) a Github felgyorsíthatják a beavatkozást. * Kell DSC tartsa a Resource Manager-sablonok párhuzamosan a tárházban.
    2. Helyezze üzembe az Azure Automation Állapotkonfiguráció a vállalati IT-előfizetés minden olyan példánya. Az Azure Automation DSC alkalmazni a felügyeleti csoporton belül támogatott előfizetések üzembe helyezett virtuális gépekhez használható.
    3. Az aktuális ütemterv tervezi egyéni Vendég konfigurációs szabályzatainak engedélyezéséhez. Ha ezt a szolgáltatást akkor szabadul fel, az Azure Automation használata az ajánlott eljárás a már nem lesz szükség.

**A felhő bevezetését előfizetéséhez (irány) további cégirányítási alkalmazása**: Való létrehozásakor a `Corporate IT Subscription`, az MVP dedikált alkalmazás archetypes támogatja az egyes előfizetésekhez alkalmazott cégirányítási kisebb módosítások hozhat létre gyors fejlődést szem előtt tartva.

Az előzetes fejlesztések, a bevált gyakorlat az NSG-k definiálva, amely blokkolja a nyilvános és a belső forgalom szerepel az engedélyezési listán. Emellett az Azure tervezet ideiglenesen létrehozott DMZ-t és az Active Directory-funkciókat. A fejlődést szem előtt tartva a azt fogja a Teljesítménybeállítások azokat az eszközöket egy kicsit, az Azure tervezet új verziójának létrehozásához.

1. A hálózati társviszony-létesítési sablont. Ez a sablon a vállalati IT-előfizetés az agyi virtuális hálózat az egyes előfizetésekben szereplő virtuális hálózat lesz társviszonyt.
    1. Az előző szakaszban útmutatást [a megosztott szolgáltatások Referenciaarchitektúra Küllős] [ shared-services] engedélyezésének a virtuális hálózatok közötti társviszonyt Resource Manager-sablonnal létrehozott.
    2. Módosíthatja a szegélyhálózat (DMZ) sablont, a korábbi cégirányítási rendszergazdánál veszi kezdetét, hogy a sablon használható segítségképp.
    3. Most már adunk lényegében, virtuális hálózatok közötti társviszonyt a DMZ-t virtuális hálózathoz, a helyi peremhálózati eszköz korábban csatlakoztatott VPN-kapcsolaton keresztül.
    4. Is javasolt, hogy a VPN el kell távolítani a sablon, valamint, hogy nincs adatforgalom legyen irányítva a helyszíni adatközpont közvetlenül a anélkül, hogy áthaladnának a vállalati IT-előfizetés és a tűzfal megoldás.
    5. További [hálózati konfiguráció](/azure/automation/automation-dsc-overview#network-planning) köteles által az Azure Automation DSC üzemeltetett virtuális gépekre vonatkozik.
2. Módosítsa az NSG-t. Az NSG-ben minden nyilvános és a közvetlen helyszíni forgalom blokkolása. A csak a bejövő forgalmat a virtuális hálózatok közötti társ a vállalati IT-előfizetés keretében kell azzá.
    1. Az előzetes rendszergazdánál veszi kezdetét az NSG-KET hoztuk létre nyilvános forgalom és az engedélyezés az összes belső forgalom blokkolása. Most szeretnénk adattípusoktól shift az NSG-t.
    2. Az új NSG-konfiguráció blokkolnia kell a nyilvános és a helyi adatközpontból az összes forgalmat.
    3. Írja be a virtuális hálózatok közötti forgalom csak érkezik, a virtuális hálózatról a VNet társ másik oldalon.
3. Az Azure Security Center végrehajtása
    1. Az Azure Security Center konfigurálása, amely tartalmazza a védett adatok besorolások felügyeleti csoporthoz.
    2. Állítsa be az automatikus kiépítést a javítási a megfelelőség biztosítása alapértelmezés szerint.
    3. Operációs rendszer biztonsági konfigurációival létesíteni. IT-biztonsági konfigurációjának meghatározása.
    4. IT-biztonsági támogatja az Azure Security Center kezdeti használatát. Áttérés a számítástechnikai biztonsági a security center használatát, ugyanakkor megtarthatja a hozzáférés-irányítási folyamatos fejlesztési célokra
    5. Hozzon létre egy Resource Manager-sablon tükröző egy előfizetésen belül az Azure Security Center-konfigurációja szükséges módosításokat.
4. Frissítse az Azure Policy minden előfizetés esetén.
    1. Naplózási és kritikusság és az adatok besorolás kényszerítése a minden felügyeleti csoportok és az előfizetések azonosíthatja a védett adatok besorolásokkal rendelkező előfizetése.
    2. Naplózási és jóváhagyott operációsrendszer-lemezképeket csak kényszerítése.
    3. Naplózási és kényszeríthet Vendég konfigurációkat az egyes csomópontok biztonsági követelmények alapján.
5. Frissítés Azure Policy minden előfizetés esetén, amely tartalmazza az adatbesorolásokat védett.
    1. Naplózási és csak a standard szintű szerepköreit kényszerítése
    2. Naplózási és alkalmazás az összes storage-fiókok és az egyes csomópontokon inaktív fájl titkosítás kényszerítésére.
    3. Naplózási és kényszeríteni az alkalmazás új verziójának a Szegélyhálózat NSG-t.
    4. Naplózási és kényszerítése jóváhagyott hálózati alhálózat és virtuális hálózatok száma hálózati adapterenként.
    5. Naplózási és a felhasználó által megadott útvonaltáblák korlátozás kényszerítése.
6. Az Azure tervezet:
    1. Hozzon létre egy Azure tervezet nevű `protected-data`.
    2. Adja hozzá a virtuális hálózatok közötti társ, az NSG-t és az Azure Security Center a tervezet-sablonokat.
    3. Győződjön meg arról, az előző fejlődést szem előtt tartva az Active Directory sablonnal nem szerepel a tervezet. Az Active Directory függőségek biztosítja a vállalati IT-előfizetés.
    4. Állítsa le minden olyan meglévő Active Directory telepített virtuális gépek a korábbi fejlődést szem előtt tartva.
    5. Adja hozzá az új szabályzatok a védett adatok előfizetésekhez.
    6. A tervrajz közzététele kívánja futtatni a védett adatok felügyeleti csoporthoz.
    7. Az új tervezet alkalmazása minden érintett előfizetésben meglévő tervezetek együtt.

## <a name="conclusion"></a>Összegzés

Ezeket a folyamatokat és az MVP segít mérsékelni az számos olyan kockázatok cégirányítási módosításai társított biztonsági cégirányítási hozzáadása. Együttesen adnak hozzá a hálózati, identitáskezelési és biztonsági adatok védelméhez szükséges eszközöket.

## <a name="next-steps"></a>További lépések

Felhőre való áttérés folyamatosan fejlődnek, és további üzleti értéket, kockázatokat és a felhő cégirányítási igényeknek is. Tartson a folyamatban szereplő fiktív cég a következő lépés, hogy üzleti szempontból alapvető fontosságú számítási feladatait támogatja. Ez az a pont, amikor szükség van az erőforrás konzisztencia-ellenőrzések.

> [!div class="nextstepaction"]
> [Erőforrás konzisztencia fejlődést szem előtt tartva](./resource-consistency-evolution.md)

<!-- links -->

[shared-services]: ../../../../reference-architectures/hybrid-networking/shared-services.md