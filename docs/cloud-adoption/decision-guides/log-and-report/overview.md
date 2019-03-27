---
title: 'CAF: Naplózás és jelentéskészítés döntési útmutató'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Naplózás, jelentéseket és az áttelepítés az Azure alapvető szolgáltatásai a "figyelés" ismerje meg.
author: rotycenh
ms.openlocfilehash: 8ab9a159b438a4ac95289d5eb5c0c0a2f4b399ae
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299439"
---
# <a name="logging-and-reporting-decision-guide"></a>Naplózás és jelentéskészítés döntési útmutató

Minden olyan mechanizmust kell értesítésére IT-csoportoknak, teljesítményének, rendelkezésre állását és a biztonsági problémákat, mielőtt az komoly problémát okoznának. Egy sikeres monitorozási stratégia lehetővé teszi, hogy jobban megismerhesse, hogyan hajtja végre az egyes összetevők, a számítási és hálózatkezelési infrastruktúrát alkotó. Nyilvános felhőbe történő migrálás kontextusában naplózás és a felszínre hozza a fontos eseményeket és mérőszámokat, a megfelelő informatikai munkatársak, miközben a meglévő figyelési rendszereket, a jelentéskészítési integrációja fontos biztosítását a szervezet megfelel-e hasznos üzemidőt garantál, biztonsági és megfelelőségi célok házirend.

![Naplózás, jelentéseket és megfigyelési lehetőségek a legkevésbé küldik az ábrázolást a legösszetettebb, az alábbi hivatkozások a jump igazítva](../../_images/discovery-guides/discovery-guide-logs-and-reporting.png)

Ugrás ide: [A figyelési infrastruktúra tervezési](#planning-your-monitoring-infrastructure) | [Felhőbeli natív](#cloud-native) | [helyszíni bővítmény](#on-premises-extension) | [átjáró összesítés ](#gateway-aggregation)  |  [Hibrid monitorozási (helyszíni)](#hybrid-monitoring-on-premises) | [hibrid monitorozás (felhőalapú)](#hybrid-monitoring-cloud-based) | [többfelhős ](#multi-cloud)  |  [További](#learn-more)

A kihasználás is egy felhőalapú, naplózás és jelentéskészítés a stratégia elsősorban meglévő befektetéseken alapul meghatározásakor a szervezet által végrehajtott üzemeltetési folyamatokat, és bizonyos mértékű követelményeinek, már támogatja a többszörös felhőstratégia.

Többféle módon napló és a jelentés a felhőben végzett tevékenységeit. Felhőbeli natív és a központi naplózási két gyakori szoftverek olyan szoftverszolgáltatások (SaaS) közül, amelyek vezérlik az előfizetések kialakítása és az előfizetések száma.

## <a name="planning-your-monitoring-infrastructure"></a>A figyelési infrastruktúra megtervezése

Az üzembe helyezés megtervezésekor vegye figyelembe a naplózási adatok tárolására, és hogyan fogja integrálni a jelentéskészítés és a felhőalapú szolgáltatások a meglévő folyamatok és eszközök az kell.

| Kérdés | Natív felhőalapú | A helyszíni kiterjesztése | Hibrid monitorozás | Átjáró-összesítési |
|-----|-----|-----|-----|-----|
| Rendelkezik egy meglévő helyszíni figyelési infrastruktúra? | Nem | Igen | Igen |  Nem |
| Megakadályozza, hogy a külső tárolási helyek naplóadatok tárolási követelményei vannak? | Nem | Igen | Nem | Nem |
| Szükség van felhőmonitorozás a helyszíni rendszerekkel való integrálásához? | Nem | Nem | Igen | Nem |
Szükség van folyamatban vagy szűrő telemetriai adatokat, mielőtt elküldené azokat a monitorozási rendszerek? | Nem | Nem | Nem | Igen |

### <a name="cloud-native"></a>Natív felhőalapú

Ha a szervezet jelenleg nem rendelkezik meghatározott naplózási és jelentési rendszerrel, vagy ha a tervezett felhőbeli üzemelő példány nem szükséges a meglévő helyszíni vagy más külső felügyeleti rendszerek integrációját, a felhőbeli natív Szolgáltatottszoftver-megoldás a legegyszerűbb megoldás.

Ebben a forgatókönyvben a naplózási adatokat rögzíti, és a számítási feladatok során, hogy a folyamat és felszíni információkat az informatikai részleg érhetők el a cloud platform részeként a naplózási funkciót és jelentéskészítési eszközök, mint az azonos felhőalapú környezetben tárolt.

Felhőbeli natív naplózási megoldások alkalmazásokon és szolgáltatásokon futtatható az ad hoc előfizetést vagy a számítási feladat kísérleti vagy kisebb környezetekben, és a Teljesítménynapló-adatok figyelése a teljes felhőalapú hagyatéki centralizált módon vannak rendszerezve.

**Felhőbeli natív feltételezések**. Egy felhőbeli natív naplózás használatával, és a jelentéskészítő rendszer feltételezi, hogy a következő:

- Nem kell integrálni a napló adatokat, a felhő számítási feladatokat a meglévő helyi rendszerek.
- Ön nem használja a jelentéskészítési felhőalapú rendszerek a helyszíni rendszerek figyeléséhez.

### <a name="on-premises-extension"></a>A helyszíni kiterjesztése

A felhő telemetriát integrálható a helyszíni rendszerek, amelyek nem támogatják hibrid naplózási és jelentéskészítési, vagy nem támogatott az alkalmazások és szolgáltatások használatával egy minimális mennyiségű átalakítását kell esetben kell üzembe helyezése a figyelés virtuális gépek számára fog adatokat küldeni a napló közvetlenül a helyszíni rendszerek ahelyett, hogy a felhőalapú környezetben való tárolásukról ügynökök.

Ez a módszer támogatja, a felhőbeli erőforrások kell tudni közvetlenül kommunikálhat a helyszíni rendszerek együttes [hibrid hálózatkezelés](../software-defined-network/hybrid.md) és [üzemeltetett tartomány felhőszolgáltatások](../identity/overview.md#cloud-hosted-domain-services). A helyen a felhőbeli virtuális hálózatot egy hálózati kiterjesztése a helyszíni környezetben működik. Ezért üzemeltetett felhőalapú számítási feladatokhoz közvetlenül tudnak kommunikálni a helyi naplózás és jelentéskészítés a rendszer.

Ez a megközelítés a meglévő befektetések figyelési eszközök korlátozott módosítással a felhőben üzembe helyezett alkalmazások és szolgáltatások nagybetűre alakítja. Ez gyakran az a leggyorsabb módja lift-and-shift-migráció során figyelésére is alkalmas. Azonban a rendszer nem rögzíti a felhőalapú PaaS és SaaS erőforrások által létrehozott naplóadatok, és kihagyják, a cloud platform maga például a virtuális gép állapota által létrehozott bármely virtuális gépekkel kapcsolatos naplók. Ez a minta eredményeképpen kell ideiglenes megoldást, amíg egy átfogóbb hibrid figyelési megoldás.

A helyszíni csak Előfeltételek:

- Csak a helyszíni környezetben, a naplóadatok biztosítani technikai követelmények támogatásához, vagy jogszabályi miatt kell vagy a házirend követelményeinek.
- A helyszíni rendszerek nem támogatják a hibrid naplózási és jelentéskészítési vagy az átjáró-összesítési megoldások.
- A felhőalapú alkalmazások küldhet telemetriát közvetlenül a helyszíni rendszerek naplózási vagy a monitorozási ügynökök küldje el a helyszíni számítási feladatok virtuális gépekre telepíthető.
- A számítási feladatokhoz a felhőalapú és a naplózás és jelentéskészítés igénylő PaaS és SaaS szolgáltatások nem függnek.

### <a name="gateway-aggregation"></a>Átjáró-összesítési

Az olyan forgatókönyvekben, ahol felhőalapú telemetriai adatok mennyisége rendkívül nagy méretű vagy a meglévő helyszíni rendszere előtt fel lehessen dolgozni módosított adatokat kell naplózni, a naplóadatok [átjáró-összesítési](../../../patterns/gateway-aggregation.md) szolgáltatás lehet szükség.

Egy átjárószolgáltatás telepítve van a felhőbeli szolgáltató. Ezt követően megfelelő alkalmazásokat és szolgáltatásokat az átjáró egy alapértelmezett naplózás rendszer helyett telemetriai adatokat küldenek vannak konfigurálva. Az átjáró ezután képes feldolgozni az adatokat: összesítés, egyesítése vagy egyéb formázás, mielőtt elküldené azokat a figyelési szolgáltatás feldolgozási és elemzési.

Emellett az átjáró összesítéséhez és natív felhőalapú vagy hibrid rendszerek kötött telemetriai adatok előfeldolgozása használható.

Átjáró-összesítési feltételezések:

- A felhőalapú alkalmazások és szolgáltatások származó telemetriaadatok nagyon magas szintű várt.
- Formázása vagy más módon optimalizálása a telemetriai adatok mielőtt elküldené azokat a monitorozási rendszerek kell.
- A monitorozási rendszerek az API-k vagy más mechanizmusok érhető el, hogy Teljesítménynapló-adatok feldolgozása után az átjáró által rendelkezik.

### <a name="hybrid-monitoring-on-premises"></a>Hibrid monitorozási (helyszíni)

Egy hibrid monitorozási megoldás kombinálja a mind a helyszíni adatok naplózása, és a felhőbeli erőforrások biztosít egy integrált képet kaphat az informatikai hagyatéki működési állapot.

Ha rendelkezik egy meglévő alatt beüzemelhető a helyszíni rendszerek, amelyek nehéz lenne, vagy cserélje le a költséges, szükség lehet a felhőbeli számítási feladatok származó telemetriai adatok integrálása a már létező helyszíni megoldások monitorozása. A hibrid helyi monitorozási rendszer, a helyszíni telemetriai adatai továbbra is a meglévő helyszíni monitorozási rendszer. Felhőalapú telemetriai adatokat a rendszer közvetlenül figyelése felhőbeli vagy érkezik, vagy az adatok mellett a számítási feladatokat a felhőben tárolt, és ezután lefordított és elemezhető a helyszíni rendszerbe rendszeres időközönként.

**A helyszíni hibrid feltételezések monitorozási**. Egy helyszíni naplózás és jelentéskészítés hibrid monitorozás a rendszer feltételezi, hogy a következő:

- Szeretné használni a meglévő helyszíni reporting rendszerek figyelése felhőbeli számítási feladatokat.
- Naplózási adatok helyszíni tulajdonjogának karbantartása kell.
- A helyi felügyeleti rendszerek API-k vagy más mechanizmusok érhető el a felhőalapú rendszerek származó naplóadatokat, hogy rendelkezik.

> [!TIP]
> Iteratív jellege típusú felhőbeli migrálás részeként a különböző natív és a helyi figyelést egy részleges hibrid megközelítés átállás valószínű. Győződjön meg arról, hogy a módosítások a monitorozási architektúra megfelel azoknak a teljes informatikai és üzemeltetési folyamatokat.

### <a name="hybrid-monitoring-cloud-based"></a>Hibrid monitorozási (felhőalapú)

Ha nem rendelkezik egy lenyűgöző kell fenntartani egy helyi monitorozási rendszer, vagy szeretné lecserélni a helyszíni rendszerek Szolgáltatottszoftver-megoldás figyelése, azt is beállíthatja a helyszíni naplóadatok integrálása egy központosított felhőbeli monitorozási rendszer.

Megközelítés a helyszíni tükrözés középre, ebben a forgatókönyvben felhőbeli számítási feladatok használja az alapértelmezett felhőalapú naplózási mechanizmus és a helyszíni alkalmazások és szolgáltatások lenne telemetriai directory küld a felhő alapú naplózási rendszer vagy Összesítés adatok rendszeres időközönként a felhőalapú rendszerbe támogatunk. A felhő alapú monitorozási rendszer ezután az elsődleges figyelés és jelentéskészítés a rendszer a teljes informatikai hagyatéki szolgál.

Felhőalapú hibrid monitorozási feltételezések: Felhőalapú naplózás és figyelés hibrid rendszerek reporting feltételezi, hogy a következő:

- Ön nem függ a meglévő helyszíni rendszerek figyelése.
- A számítási feladatok nem rendelkezik szabályozási és naplózási adatok helyi tárolására vonatkozó követelményeket.
- A felhőalapú rendszerek rendelkezik az API-k vagy más mechanizmusok érhető el, hogy a helyszíni alkalmazások és szolgáltatások naplóadatait.

### <a name="multi-cloud"></a>Több felhő

Naplózás és jelentéskészítési képességekkel integrálása egy többfelhős platformon bonyolult feladatnak bizonyulhat. Platformok közötti kínált szolgáltatások gyakran nem hasonlíthatók össze közvetlenül, és ezek a szolgáltatások által biztosított naplózás és a telemetria képességei eltérnek is.
Több felhő naplózási támogatási gyakran szükséges átjárószolgáltatásokat Teljesítménynapló-adatok feldolgozása egy közös formátumra hibrid naplózási megoldás az adatok elküldése előtt.

## <a name="learn-more"></a>Részletek

[Az Azure Monitor](/azure/azure-monitor/overview) jelentéskészítés és a monitorozás az Azure-szolgáltatás az alapértelmezett. Itt:

- Egy alkalmazás telemetriai adatainak, gazdagép telemetriai adatok (például a VM-EK), tárolómetrikák, az Azure platform metrikák és eseménynaplók gyűjtése egységes platform.
- Vizualizáció, lekérdezések, riasztások és analitikai eszközei. Azt is lehetővé teszi virtuális gépek, a vendég operációs rendszerek, a virtuális hálózatok és a számítási feladatok alkalmazásesemények betekintést.
- [REST API-k](/azure/monitoring-and-diagnostics/monitoring-rest-api-walkthrough) külső szolgáltatásokkal és monitorozási és riasztási szolgáltatásokkal az automation-integráció
- [Integráció](/azure/monitoring-and-diagnostics/monitoring-partners) számos népszerű külső szállítókkal.
