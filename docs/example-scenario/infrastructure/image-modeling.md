---
title: A digitális lemezképen alapuló modellezés felgyorsítása az Azure-ban
titleSuffix: Azure Example Scenarios
description: A digitális lemezképen alapuló modellezés felgyorsítása az Azure-ban az Avere és az Agisoft PhotoScan segítségével
author: adamboeglin
ms.date: 01/11/2019
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: cat-team, Linux, HPC
social_image_url: /azure/architecture/example-scenario/infrastructure/media/architecture-image-modeling.png
ms.openlocfilehash: 0acfebf0ecdd81becdc939bfd5c4d55184fbae83
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299208"
---
# <a name="accelerate-digital-image-based-modeling-on-azure"></a>A digitális lemezképen alapuló modellezés felgyorsítása az Azure-ban

Ebben a példában a forgatókönyvben minden olyan szervezet, amely az Azure infrastruktúra--szolgáltatásként (IaaS) hajthat végre lemezképalapú modellezési szeretne architektúra és kialakítás útmutatást nyújt. A forgatókönyv photogrammetry szoftvereket futtató Azure virtuális gépeken (VM-EK) használatával nagy teljesítményű tárolási, feldolgozási időt gyorsító lett tervezve. A környezet felfelé és lefelé igény szerint skálázhatók, és támogatja a terabájt tárhely teljesítményét feláldozása nélkül.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Alkalmazási helyzetek a következők:

- Modellezés és mérési épületek, mérnöki és a törvényszéki objektuma jelenetek.
- A számítógép játékokat és filmek vizuális effektusok létrehozása.
- Digitális rendszerképekből közvetve a mérések objektumok hasonlóan városi tervezési különféle méretezhető és más alkalmazások létrehozásához.

## <a name="architecture"></a>Architektúra

Ebben a példában a Agisoft PhotoScan photogrammetry szoftver Avere vFXT tárhely használatát ismerteti. A keretrendszer népszerűségének földrajzi információk system (GIS) alkalmazások, kulturális örökségének dokumentáció, játékfejlesztés és vizuális effektusok éles PhotoScan választott. Ideális Bezárás tartományon kívüli photogrammetry és a légi felvételeken photogrammetry.

A jelen cikk fogalmait vonatkoznak minden olyan nagy teljesítményű feldolgozási (HPC) számítási feladatok alapján az ütemező és a feldolgozó csomópontok infrastruktúra.  A számítási feladat Avere vFXT a kiváló teljesítmény teljesítménytesztekben során lett kiválasztva.  Azonban a forgatókönyv leválasztja-e a feldolgozási a tároló, hogy használható-e egyéb tárolási megoldások (lásd: [alternatívák](#alternatives) dokumentum későbbi részében).

Ez az architektúra az Active Directory-tartományvezérlőket az Azure-erőforrásokhoz való hozzáférést, és adja meg a belső névfeloldást keresztül a tartomány nevét (DNS) is tartalmaz. A Jump mezők rendszergazdai hozzáférést biztosítanak a Windows és Linux rendszerű virtuális gépek, amelyek a megoldás futtatása.

![Architektúra ábrája](./media/architecture-image-modeling.png)

1. Felhasználói PhotoScan lemezképeket számos küldi el.
2. Windows virtuális gép, amely a fő csomópontot funkcionál, és átirányítja a felhasználói lemezképek az PhotoScan ütemező futtatja.
3. PhotoScan a fényképek közös pontok keresi, és a geometriai (háló), a grafikus processzorral (GPU) rendelkező virtuális gépeken futó PhotoScan feldolgozó csomópontok használatával hoz létre.
4. Avere vFXT biztosít egy nagy teljesítményű tárolási megoldás az Azure-ban a hálózati fájlrendszer (NFSv3) 3-as verziójú alapján, és legalább négy virtuális gépek áll.
5. PhotoScan rendereli a modellt.

### <a name="components"></a>Összetevők

- [Agisoft PhotoScan](http://www.agisoft.com/): Az PhotoScan ütemező Windows 2016 kiszolgáló virtuális gépen fut, és a feldolgozó csomópontok öt virtuális gépek használata a CentOS Linux 7.5 rendszerű gpu-kkal.
- [Avere vFXT](/azure/avere-vfxt/avere-vfxt-overview) gyorsítótárazási megoldás, amely a objektum tárolási és a hagyományos hálózati tárolóeszközök (NAS) használja a nagyméretű adathalmazok tárolási költségek optimalizálására a fájl. Ezek a következők:
  - Avere vezérlő. Ez a virtuális gép, amely a Avere vFXT fürtöt telepít, és Ubuntu LTS-18.04 futtatja a szkriptet hajt végre. A virtuális gép később használható, hozzáadása vagy eltávolítása a fürtcsomópontokat, és semmisítse meg a fürtöt is.
  - vFXT fürt. Legalább három virtuális gépet használ, egyet a Avere vFXT csomópontok Avere OS 5.0.2.1 alapján. Ezek a virtuális gépek alkotják a vFXT fürt, az Azure Blob storage-van csatolva.
- [A Microsoft Active Directory-tartományvezérlők](/windows/desktop/ad/active-directory-domain-services) a gazdagép tartomány erőforrások hozzáférésének engedélyezéséhez, és adja meg a DNS-név feloldása. Avere vFXT számos olyan rekordot ad hozzá &mdash; például minden egyes rekord vFXT fürtben az egyes Avere vFXT csomópontok IP-cím mutat. Az ezzel a beállítással az összes virtuális gépek használata a Ciklikus időszeleteléses minta vFXT eléréséhez exportálja.
- [Más virtuális gépek](/azure/virtual-machines/) szolgál, a rendszergazda által az ütemezőt és a feldolgozó csomópontok eléréséhez használt mezők ugorhat. A Windows jumpbox megadása kötelező, hogy a rendszergazda távoli asztali protokollon keresztül a fő csomópont eléréséhez. A második jumpbox nem kötelező, és a Linux fut a feldolgozó csomópontok felügyeletéhez.
- [Hálózati biztonsági csoportok](/azure/virtual-network/manage-network-security-group) (NSG-k) nyilvános IP-cím (PIP) való hozzáférés korlátozására, és lehetővé teszi a 3389-es port és a 22-es hozzáférés a virtuális gépekhez csatolt a Jumpbox alhálózatára.
- [Virtuális hálózatok közötti társviszony](/azure/virtual-network/virtual-network-peering-overview) PhotoScan virtuális hálózatot csatlakoztat egy Avere virtuális hálózathoz.
- [Az Azure Blob storage](/azure/storage/blobs/storage-blobs-introduction) Avere vFXT együttműködik, a core filer a feldolgozás alatt véglegesített adatokat tárolja. Avere vFXT azonosítja az aktív adatok Azure Blob és a tartós állapotú meghajtókhoz (SSD) be azt egy PhotoScan feladat futása közben a számítási csomópontokat a gyorsítótárazáshoz használt csomagokban tárolt. Ha a módosításokat, az adatok a aszinkron módon véglegesített térjen vissza a core filer.
- [Az Azure Key Vault](/azure/key-vault/key-vault-overview) PhotoScan aktiváló kódot és rendszergazdai jelszavak tárolására szolgál.

### <a name="alternatives"></a>Alternatív megoldások

- HPC-fürt kezelése az Azure-szolgáltatások előnyeit, használja az Azure CycleCloud vagy az Azure Batch ahelyett, hogy az erőforrásokat sablonok vagy parancsfájlok segítségével.
- Üzembe helyezése a BeeGFS párhuzamos virtuális fájlrendszer Avere vFXT helyett az Azure háttér-tárolóként. Használja a [BeeGFS sablon](https://github.com/paulomarquesc/beegfs-template) a teljes körű megoldás az Azure-ban üzembe helyezéséhez.
- A tárolási megoldás szerkesztőprogramban, például GlusterFS, a Lustre vagy a Windows közvetlen tárolóhelyek üzembe helyezése. Ehhez szerkessze a [PhotoScan sablon](https://github.com/paulomarquesc/photoscan-template) a tárolási megoldásnak szeretne együttműködni.
- Helyezze üzembe a feldolgozó csomópontok Linux, az alapértelmezett beállítás helyett a Windows operációs rendszerrel. Ha Windows-csomópontok, tároló-integrációs beállítások nem hajtja végre a központi telepítési sablonok által. Kell manuálisan a környezet integrálható a meglévő tárolási megoldás, vagy testre szabhatja a PhotoScan sablont adja meg az ilyen automation leírtak szerint a [tárház](https://github.com/paulomarquesc/photoscan-template/blob/master/docs/AverePostDeploymentSteps.md).

## <a name="considerations"></a>Megfontolandó szempontok

Ebben a forgatókönyvben kifejezetten a nagy teljesítményű tárolást biztosít a HPC számítási feladat célja, Windows vagy Linux rendszeren telepítve van-e. Általánosságban véve a HPC számítási feladatok a tárolási konfiguráció meg kell egyeznie a helyszíni üzemelő használt megfelelő ajánlott eljárásokkal.

Telepítési megfontolások függnek az alkalmazások és szolgáltatások használt, de pár megjegyzéseket a alkalmazni:

- Nagy teljesítményű alkalmazások létrehozását, használja az Azure Premium Storage és [optimalizálása az alkalmazásrétegre](/azure/virtual-machines/windows/premium-storage-performance). Az Azure Blob-gyakori hozzáférés a tárolási költségek optimalizálására [ritkáról gyakori elérésű hozzáférési szint](/azure/storage/blobs/storage-blob-storage-tiers).
- A storage használata [replikációs beállítás](/azure/storage/common/storage-redundancy) , amely megfelel a rendelkezésre állás és teljesítmény. Ebben a példában Avere vFXT van konfigurálva a magas rendelkezésre állás érdekében a helyileg redundáns tárolás (LRS) alapértelmezés szerint. A terheléselosztást, a telepítő használatban lévő összes virtuális gép a Ciklikus időszeleteléses minta vFXT eléréséhez exportálja.
- Ha Windows-ügyfelek és a Linux-ügyfelek a háttérrendszer tárolási fognak használni, Samba kiszolgálók használatával támogatja a Windows-csomópontok. A [verzió](https://github.com/paulomarquesc/beegfs-template) ebben a példában a forgatókönyv BeeGFS alapján használja Samba támogatásához a HPC számítási feladatok (PhotoScan) futó Windows ütemező csomópontján. Ahhoz, mint az intelligens helyettesíti a DNS ciklikus időszeletelése helyezünk üzembe egy terheléselosztót.
- Az olyan futtassa HPC-alkalmazások legjobb használatával a virtuális gép típusát a [Windows](/azure/virtual-machines/windows/sizes-hpc) vagy [Linux](/azure/virtual-machines/linux/sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) számítási feladatot.
- A HPC számítási feladatok a tárolási erőforrások elkülönítése, mindegyik a saját virtuális hálózat üzembe helyezése, majd a virtuális hálózat [társviszony-létesítés](/azure/virtual-network/virtual-network-peering-overview) a két csatlakozni. Társviszony-létesítés különböző virtuális hálózatok és útvonalak forgalmát a Microsoft gerincinfrastruktúráján keresztül a magánhálózati IP-címek csak az erőforrások között alacsony késleltetésű, nagy sávszélességű kapcsolatot hoz létre.

### <a name="security"></a>Biztonság

Ebben a példában a HPC számítási feladat nagy teljesítményű tárolási megoldás telepítése összpontosít, és nem biztonsági megoldásokat. Győződjön meg arról, hogy a változások a biztonsági csoport.

A fokozott biztonság érdekében ez a példa az infrastruktúra lehetővé teszi az összes Windows virtuális gép kell a tartományhoz csatlakoztatott és az Active Directoryt használja a központi hitelesítési. Az összes virtuális gép egyéni DNS-szolgáltatásokat is biztosít. A környezet védelme érdekében ez a sablon támaszkodik [hálózati biztonsági csoportok (NSG-k)](/azure/virtual-network/security-overview). Az NSG-k alapvető forgalomszűrőinek és a biztonsági szabályok kínálnak.

Vegye figyelembe az alábbi további az ebben a forgatókönyvben a biztonság növelése érdekében:

- Hálózati virtuális berendezések, például a Fortinet, a Checkpoint és a Juniper használata.
- Alkalmazása [szerepköralapú hozzáférés-vezérlés](/azure/role-based-access-control/overview) az erőforráscsoportokhoz.
- Engedélyezze a virtuális gép [igény szerinti](/azure/security-center/security-center-just-in-time) hozzáférni, ha a jump mezők érhetők el az interneten keresztül.
- Használat [Azure Key Vault](/azure/key-vault/quick-create-portal) a rendszergazdai fiókok által használt jelszavak tárolására.

## <a name="pricing"></a>Díjszabás

Ez a forgatókönyv futtatásával járó költségeket is nagyban attól függően, több tényező befolyásolja.  A szám és a virtuális gépek, mennyi tárhelyre szükség, és mennyi ideig a feladat végrehajtásához mérete határozza meg a költségek.

Az alábbi minta költsége a profil a [Azure díjkalkulátorát](https://azure.com/e/42362ddfd2e245a28a8e78bc609c80f3) Avere vFXT és PhotoScan tipikus konfigurációja alapján:

- 1 A1\_v2 Ubuntu virtuális Gépet szeretné futtatni a Avere vezérlő.
- 3 D16s\_v3 Avere operációs rendszer virtuális gépeket, a vFXT fürt Avere vFXT csomópontok mindegyikéhez.
- 5 NC24\_v2 Linux rendszerű virtuális gépek a GPU-kkal szükség szerint a PhotoScan feldolgozó csomópontok számára.
- 1 D8s\_v3 CentOS VM PhotoScan scheduler csomópont.
- 1 DS2\_v2 CentOS rendszergazda jumpbox használja.
- 2 DS2\_v2 virtuális gép számára az Active Directory-tartományvezérlők.
- Prémium szintű managed disks.
- Általános célú v2 (GPv2) a Blob storage, az LRS és a gyakori elérésű hozzáférési szint, (csak GPv2-tárfiókokban fiókszinten elérhető a hozzáférési réteg attribútum).
- Virtuális hálózat 10 TB-nyi adatátvitel támogatását.

Ez az architektúra kapcsolatos részletekért lásd: a [e-könyv](https://azure.microsoft.com/en-us/resources/deploy-agisoft-photoscan-on-azure-with-azere-vfxt-for-azure-or-beegfs/). Hogyan díjszabását szeretné módosítani az adott használati esetekhez megtekintéséhez válassza ki a megfelelő a várható telepítési díjkalkulátor különböző méretű virtuális gépek.

## <a name="deployment"></a>Környezet

A részletes üzembe helyezéséhez szükséges összes előfeltételnek Avere FxT vagy beegfs rendszerekhez, többek között ebben az architektúrában az e-könyv letöltése: [Az Azure-Avere vFXT Agisoft PhotoScan üzembe az Azure-ban vagy BeeGFS](https://azure.microsoft.com/en-us/resources/deploy-agisoft-photoscan-on-azure-with-azere-vfxt-for-azure-or-beegfs/).

## <a name="related-resources"></a>Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)

A következő erőforrások sáv több információt nyújt az összetevőkről használt fel ebben a forgatókönyvben a batch-megoldások az Azure olyan alternatív módszert.

- Áttekintése [Avere vFXT az Azure-hoz](/azure/avere-vfxt/avere-vfxt-overview)
- [Agisoft PhotoScan](https://www.agisoft.com/) kezdőlapja
- [Az Azure Storage teljesítmény- és skálázhatósági ellenőrzőlistája](/azure/storage/common/storage-performance-checklist)
- [A Microsoft Azure-rendszerek párhuzamos virtuális fájlrendszer: A teljesítménytesztek Lustre, GlusterFS és beegfs rendszerekhez](https://azure.microsoft.com/mediahandler/files/resourcefiles/parallel-virtual-file-systems-on-microsoft-azure/Parallel_Virtual_File_Systems_on_Microsoft_Azure.pdf) (PDF)
- A példaforgatókönyv [számítógéppel támogatott mérnöki (CAE-) az Azure-ban](/azure/architecture/example-scenario/apps/hpc-saas)
- [Az Azure-ban a HPC](https://azure.microsoft.com/en-us/solutions/high-performance-computing/) kezdőlapja
- Áttekintése [Big Compute: HPC &amp; Microsoft Batch](https://azure.microsoft.com/en-us/solutions/big-compute/)