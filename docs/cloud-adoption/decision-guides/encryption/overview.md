---
title: 'CAF: Titkosítás'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: További információ a titkosítási Azure áttelepítések core szolgáltatás.
author: rotycenh
ms.openlocfilehash: 8735800fe481a0e0292a6a0c7555e883e29f8ee2
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298473"
---
# <a name="encryption-decision-guide"></a>Titkosítási döntési útmutató

Titkosított adatokat védi jogosulatlan hozzáféréssel szemben. Megfelelően megvalósított titkosítási házirend biztosít további biztonsági rétegeket, a felhőalapú számítási feladatokhoz és a támadók és a szervezet és a hálózatok kívül és belül is a jogosulatlan felhasználóktól őröket.

![Titkosítási beállítások a legkevésbé küldik az ábrázolást a legösszetettebb, az alábbi hivatkozások a jump igazítva](../../_images/discovery-guides/discovery-guide-encryption.png)

Ugrás ide: [Kulcskezelés](#key-management) | [adattitkosítás](#data-encryption) | [további](#learn-more)

Titkosítási stratégia a vállalati házirend és megfelelőség megbízások összpontosít. Erőforrások titkosítása általában célszerű, és az Azure-ban például az Azure Storage és Azure SQL Database számos szolgáltatás titkosítási alapértelmezés szerint engedélyezi. Titkosítási azonban rendelkezik, amely növelheti a késést és a teljes erőforrás-használat költségeit.

Erőforrás-igényű számítási feladatokhoz, titkosítás és a teljesítmény közötti megfelelő egyensúly beírásával és annak meghatározása, adatok és az adatforgalom titkosított hogyan lehet alapvető. Titkosítási mechanizmusok eltérőek lehetnek a költségek és munkaráfordítás alól, és a döntéseket hozhat, hogy a rendszer titkosítást alkalmaz, és hogyan tárolhatja és kezelheti a létfontosságú kulcsok és titkok befolyásolhatja a műszaki és a házirend követelményeinek.

Vállalati házirend és megfelelőség külső eszközillesztőket a legnagyobb egy titkosítási stratégia tervezésekor. Az Azure biztosít, amely megfelel a titkosított adatok, e inaktív és átvitel vonatkozó gyakori követelmények több standard mechanizmusokat. Azonban a házirendek és a megfelelőségi követelményeknek, hogy igény szerint szigorúbb vezérlők, például szabványos titkos kulcsok és kulcsok kezelését, titkosítási használatban lévő vagy adott adattitkosítás, ki kell támogatja ezeket a titkosítási ennél kifinomultabb stratégiát fejlesztéséhez követelmények.

## <a name="key-management"></a>Kulcskezelés

A modern kulcskezelés rendszerek kell nyújt támogatást a kulcsokat hardveres biztonsági modulokban (HSM) használatával nagyobb védelme tárolja. Egy kulcskezelő system, így fontos a szervezet képes létrehozni és tárolni a titkosítási kulcsokat, fontos jelszavak, kapcsolati karakterláncok és más informatikai bizalmas adatokat.

Felhőbe történő migrálás megtervezésekor a következő táblázat ismerteti, hogyan tárolhatja és kezelheti a titkosítási kulcsokat, tanúsítványok és titkos kódok, amelyek létfontosságúak a biztonságos és kezelhető magánfelhők létrehozása:

| Kérdés | Natív felhőalapú | Hibrid | Helyszíni követelmények |
|---------------------------------------------------------------------------------------------------------------------------------------|--------------|--------|-------------|
| Nem a szervezet nem rendelkezik a központosított kulcs és titkos kódok kezelése?                                                                    | Igen          | Nem     | Nem          |
| Szükség lesz korlátozni a kulcsok és titkok eszközökhöz, a helyszíni hardver létrehozása során ezek a kulcsok használatára a felhőben? | Nem           | Igen    | Nem          |
| Rendelkezik a szervezete szabályokat, vagy a házirendekkel, amely megakadályozná a kulcsok nem külső helyszínen tárolt?                | Nem           | Nem     | Igen         |

### <a name="cloud-native"></a>Natív felhőalapú

Felhőbeli natív kulcskezelés, az összes kulcsot és titkos kulcsok létrehozott, felügyelt, és egy felhőalapú tárolót, például az Azure Key Vaultban tárolt. Ez a megközelítés leegyszerűsíti a kulcskezelési, például a kulcs biztonsági mentése, a storage és a megújítási kapcsolódó számos informatikai feladatokat.

Felhőbeli natív kulcskezelés rendszert használ a következőket feltételezi:

- A felhőmegoldás kulcskezelés létrehozása, kezelése és a szervezet kulcsok és titkok üzemeltető megbízik.
- A helyszíni alkalmazások és szolgáltatások, a titkosítási szolgáltatások vagy a titkos kulcsok hozzáférhetnek a felhőbeli kulcskezelés rendszerhez való hozzáférés használó engedélyezi.

### <a name="hybrid-bring-your-own-key"></a>Hibrid (a saját kulcs használata)

Ezzel a módszerrel hozzon létre dedikált HSM hardveren kulcsokat, a helyszíni környezetben, majd a kulcsok átvitele a key vault védelmet biztosítani a felhőnek a felhőbeli erőforrások segítségével.

Hibrid kulcskezelés feltételezések: Hibrid kulcskezelés rendszert használ a következőket feltételezi:

- Az alapul szolgáló biztonsági és a vezérlő infrastruktúra, a üzemeltetési és a kulcsok és titkos kulcsok használatával a felhőalapú platform megbízik.
- A felhőben üzemeltetett alkalmazások és szolgáltatások eléri és használja a kulcsok és titkos kulcsok egy hatékony és biztonságos módon.
- Szabályozási vagy a szervezeti házirend létrehozásának és felügyeletének kulcsokat a helyszíni és a szervezet titkok tartani szükséges.

### <a name="on-premises-hold-your-own-key"></a>A helyszíni (saját tulajdonú kulcs)

Bizonyos esetekben előfordulhat, szabályozási, házirend, vagy technikai okokból, miért nem lehet tárolni kulcsok nyilvános felhőszolgáltatások által biztosított kulcs felügyeleti rendszeren. Ezekben az esetekben tarthatja karban a kulcsokat a helyszíni hardver használatával, és üzembe helyezése mechanizmussal teheti lehetővé a felhőalapú erőforrások eléréséhez ezek a kulcsok titkosítási célokra. Előfordulhat, hogy egy tartsa a saját kulcs megközelítés nem kompatibilis az összes felhőalapú szolgáltatásához.

A helyszíni kulcskezelő feltételezések: Egy helyszíni kulcskezelő rendszerrel feltételezi, hogy a következő:

- Szabályozási vagy a szervezeti házirend szerint kell tartani a létrehozása, kezelése, és a szervezet kulcsok és titkok helyszíni üzemeltetéséhez.
- Felhőalapú alkalmazások és szolgáltatások által használt titkosítási szolgáltatásokat vagy titkos kulcsok elérése, hozzáférhet a helyszíni kulcskezelő rendszer.

## <a name="data-encryption"></a>Adattitkosítás

Van több különböző állam, az adatok különböző titkosítási igények tervezésekor a titkosítási házirend:

| Adat-állapota | Adatok |
|-----|-----|
| Az átvitt adatok | Belső hálózati forgalmat, internetes kapcsolatok, adatközpontok, illetve virtuális hálózatok közötti kapcsolatot |
| Inaktív adat    | Adatbázisok, fájlok, virtuális meghajtók, PaaS-tároló |
| Használatban lévő adatok     | Adatok betöltése, a RAM-MAL vagy a Processzor-gyorsítótárak |

### <a name="data-in-transit"></a>Az átvitt adatok

Az átvitt adatokat olyan adat, az a belső, adatközpontok, vagy a külső hálózatok közötti vagy az interneten keresztül erőforrások közötti áthelyezése.

Az átvitt adatok titkosítása SSL/TLS-protokollok igénylése a forgalom általában történik. A külső hálózathoz vagy a nyilvános interneten felhőben futó erőforrások között áthaladó forgalom mindig titkosított marad. PaaS-erőforrásokhoz általánosan is SSL/TLS titkosítás kényszerítéséhez a forgalom alapértelmezés szerint. E kényszeríti a titkosítást a virtuális hálózatokon lévő üzemeltetett IaaS-erőforrások közötti forgalom a felhő bevezetésének csapat és a számítási feladatok felelőse döntés, és általában ajánlott.

**Titkosított adatokat átvitel közben feltételezések**. Az átvitt adatok megfelelő titkosítási szabályzat megvalósítása feltételezi, hogy a következő:

- A nyilvános interneten SSL/TLS-protokollok használatával kommunikál az összes nyilvánosan elérhető végpontot a felhőalapú környezetben.
- Ha a felhőalapú hálózatokhoz kapcsolódik a helyszíni vagy más külső hálózati a nyilvános interneten keresztül, használja a titkosított VPN-protokoll.
- Ha a helyszíni hálózatok csatlakoztatását felhőalapú vagy más külső hálózathoz, például az ExpressRoute dedikált WAN kapcsolatot használ, használhatja a VPN vagy más titkosítási készülék helyszíni párosítva a megfelelő virtuális VPN- vagy titkosítási berendezés üzembe helyezett a felhő-hálózathoz.
- Ha bizalmas adatok, naplók vagy más informatikai szakemberek számára látható diagnosztikai jelentések nem szerepelnek, titkosítani fog minden forgalmat a virtuális hálózati erőforrások között.

### <a name="data-at-rest"></a>Inaktív adat

Az inaktív adatok nem aktívan áthelyezését vagy feldolgozva, beleértve a fájlok, adatbázisok, virtuális gép meghajtók, PaaS-storage-fiókok vagy hasonló eszköz adatokat jelöli. A tárolt adatok titkosítása védi a virtuális eszközök vagy a fájlok jogosulatlan hozzáféréstől, külső hálózati behatolás, a belső felhasználók rosszindulatú vagy véletlen kiadások.

PaaS-tárolási és adatbázis-erőforrásokat általában a titkosítás kényszerítéséhez alapértelmezés szerint. IaaS-virtuális erőforrások védve legyenek a kulcskezelés rendszerben tárolt titkosítási kulcsok segítségével Virtuálislemez titkosítással.

Az inaktív adatok titkosítását is magában foglalja a fejlettebb adatbázist titkosítási technikák, például az oszlopszintű és sor fájlszintű titkosítás, amely számos egyéb biztosít szabályozhatja, hogy pontosan milyen adatok alatt álló védett.

Az általános házirend- és megfelelőségi követelményeket, a tárolt adatok érzékenysége és a teljesítmény-követelmények a számítási feladatok kell meghatározni, hogy mely eszközök titkosításának megkövetelése.

**Titkosított adatok Rest feltételezések**. Inaktív adatok titkosításához feltételezi, hogy a következő:

- Nem nyilvános felhasználásra böngészésre adatokat tárolja.
- A számítási feladatok elfogadhatja a lemeztitkosítás hozzáadott késés költségét.

### <a name="data-in-use"></a>Használatban lévő adatok

Használatban lévő adatok titkosítását magában foglalja az adatok védelme nem állandó tárolókba, például a CPU vagy RAM gyorsítótárak az. A technológiák – például titkosítást teljes memória, enklávé technológiák, például az Intel biztonságos Guard Extensions (SGX) használata. Ide tartoznak az titkosítási módszerek, például homomorphic titkosítást, amely segítségével biztonságos, megbízható végrehajtási környezeteket hozhat létre.

**Használati feltételek az adatok titkosítása**. Használatban lévő adatok titkosításához feltételezi, hogy a következő:

- Ön mindig, elkülönítve az alapul szolgáló felhőplatformot vezet az adatok tulajdonjoga, még akkor is, a RAM-MAL és CPU szinten.

## <a name="learn-more"></a>Részletek

Titkosítás és kulcskezelés az Azure platformon további információt a következő témakörben találhat.

- [Azure-titkosítás áttekintése](/azure/security/security-azure-encryption-overview). Hogyan Azure titkosítást használ mind az inaktív adatok és az átvitt adatok biztonságossá tételéhez részletes leírása.
- [Azure Key Vault](/azure/key-vault/key-vault-overview). A Key Vault, az elsődleges kulcs felügyeleti rendszer, tárolására és kezelésére a titkosítási kulcsok, titkos kódok és tanúsítványok Azure-ban.
- [Az Azure-ban bizalmas számítási](/solutions/confidential-compute). Bizalmas számítástechnikát az Azure biztosít eszközöket és technológiájával létrehozott megbízható végrehajtási környezetekben vagy más titkosítási mechanizmusok használatban lévő adatok védelmét.

## <a name="next-steps"></a>További lépések

Ismerje meg, hogyan a szoftver meghatározott hálózatok biztosítják a magánfelhők számára a virtualizált hálózati funkciók.

> [!div class="nextstepaction"]
> [Melyik szoftveralapú hálózati minta az üzembe helyezés esetén ajánlott használni?](../software-defined-network/overview.md)
