---
title: Linux rendszerű virtuális asztali környezetek a Citrix-szel
description: VDI-környezetet hozhat létre Linux rendszerű asztali környezetekhez a Citrix használatával az Azure-ban.
author: miguelangelopereira
ms.date: 09/12/2018
ms.custom: fasttrack
ms.openlocfilehash: d48163638da05fa075814d3a255ca783610741f8
ms.sourcegitcommit: a0e8d11543751d681953717f6e78173e597ae207
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/06/2018
ms.locfileid: "53004780"
---
# <a name="linux-virtual-desktops-with-citrix"></a>Linux rendszerű virtuális asztalok Citrix-szel

Ebben a példaforgatókönyvben esetén minden iparág, amelyet egy virtuális asztali infrastruktúra (VDI) Linux rendszerű asztali számítógépeken alkalmazható. A folyamat egy felhasználói asztalról szigeten él a kiszolgáló az adatközpontban található virtuális gépen futó VDI hivatkozik. Ebben a forgatókönyvben az ügyfél VDI igényeiknek Citrix-alapú megoldás használatát választotta.

Szervezetek gyakran rendelkeznek heterogén környezetekből több eszközzel és az alkalmazottak által használt operációs rendszereket. Miközben fenntartja a biztonságos környezetet biztosít a konzisztens alkalmazás-hozzáférési kihívást jelenthet. Egy Linux rendszerű asztali számítógépeken VDI-megoldás lehetővé teszi a szervezet számára biztosít hozzáférést, attól függetlenül, az eszköz vagy felhasználó által használt operációs rendszer.

Ez a minta megoldás egyes értékelemek közé tartozik a következő:
* Az elvárt megtérülési rátát ugyanazon az infrastruktúrán történő hozzáférést ad a több felhasználó által megosztott Linux rendszerű virtuális asztalok a magasabb lesz. Központosított VDI-környezetben lévő erőforrások konszolidálásával a végfelhasználói eszközöket nem kell lenniük nagy teljesítményűek.
* Függetlenül a végfelhasználói eszközön egységes lesz a teljesítmény.
* Felhasználók hozzáférhessenek a Linux-alkalmazásokhoz (beleértve a nem Linux rendszerű eszközök), bármilyen eszközről.
* Bizalmas adatokat az összes elosztott alkalmazott leköthetőek az Azure-adatközpontban.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Ebben a forgatókönyvben a következő használati esetekhez, vegye figyelembe:

* Biztonságos hozzáférés biztosítására alapvető fontosságú, specializált Linux VDI asztalok Linux vagy a nem Linux rendszerű eszközökről

## <a name="architecture"></a>Architektúra

[![](./media/azure-citrix-sample-diagram.png "Architektúra ábrája")](./media/azure-citrix-sample-diagram.png#lightbox)

A példaforgatókönyv azt szemlélteti, amely lehetővé teszi a vállalati hálózat eléréséhez a Linux rendszerű virtuális asztalok:

* Az ExpressRoute gyors és megbízható kapcsolatot a felhőbe a helyszíni környezet és az Azure között jön létre.
* A Citrix XenDeskop megoldás üzembe helyezett virtuális asztali infrastruktúra.
* Futtassa a CitrixVDA Ubuntu (vagy egy másik támogatott disztribúció).
* Az Azure hálózati biztonsági csoportok a megfelelő hálózati hozzáférés-vezérlési listák lesznek érvényesek.
* Citrix ADC (NetScaler) teszi közzé, és a terhelést a Citrix-szolgáltatásokat.
* Az Active Directory Domain Services tartományi használandó csatlakozzon a Citrix-kiszolgálók. VDA kiszolgálók nem lesz tartományhoz.
* Hibrid Azure File Sync lehetővé teszi, megosztott tárolót a megoldás között. Ha például használat távoli készletkövetést megoldások.

Ebben a forgatókönyvben a következő termékváltozatok használhatók:

- A Citrix ADC (NetScaler): 2-x D4sv3 [NetScaler 12.0-s VPX Standard Edition 200 MB/s PAYG kép](https://azuremarketplace.microsoft.com/pt-br/marketplace/apps/citrix.netscalervpx-120?tab=PlansAndPrice)
- A Citrix licenckiszolgáló: 1 x D2s v3
- A Citrix VDA: 4 x D8s v3
- A Citrix kirakat: 2 x D2s v3
- Citrix kézbesítési vezérlő: 2 x D2s v3
- Tartományvezérlő: 2 x D2sv3
- Az Azure-kiszolgálók: 2 x D2sv3

> [!NOTE]
> (Nem a NetScaler) a licencek bring-your-saját licenc (használata BYOL)

### <a name="components"></a>Összetevők

- [Az Azure Virtual Network](/azure/virtual-network/virtual-networks-overview) lehetővé teszi az erőforrások, például virtuális gépek biztonságosan kommunikálnak egymással, az internethez, és a helyszíni hálózatokkal. Virtuális hálózatok adja meg az elkülönítés és Szegmentálás, szűrő és útvonal-forgalom és helyek közötti kapcsolat engedélyezését. Ebben a forgatókönyvben minden erőforrás egy virtuális hálózat lesz használható.
- [Azure-beli hálózati biztonsági csoportok](/azure/virtual-network/security-overview) tartalmaznak, amelyek engedélyezik vagy megtagadják a bejövő vagy kimenő hálózati forgalmat a forrás vagy cél IP-cím, port és protokoll alapján biztonsági szabályokból álló listát. Ebben a forgatókönyvben a virtuális hálózatok a hálózati biztonsági csoport szabályait, amelyek korlátozzák a forgalmat az alkalmazás-összetevők közötti biztonságosak.
- [Az Azure load balancer](/azure/application-gateway/overview) osztja el a szabályok és az állapotmintákat megfelelően bejövő forgalmat. Egy terheléselosztót biztosít alacsony késéssel és nagy átviteli sebességet, és akár több milliónyi összes TCP és UDP-alkalmazás méretezhető. Belső terheléselosztó szolgál ebben a forgatókönyvben a Citrix NetScaler a forgalom elosztását.
- [Hibrid Azure File Sync](https://github.com/MicrosoftDocs/azure-docs/edit/master/articles/storage/files/storage-sync-files-planning.md) minden megosztott tároló használható. A storage hibrid File Sync használatával két fájlkiszolgálók replikálja.
- [Az Azure SQL Database](/azure/sql-database/sql-database-technical-overview) egy relációs adatbázis-a-szolgáltatás (DBaaS) a Microsoft SQL Server adatbázismotor legújabb stabil verziója alapján. A Citrix-adatbázisokat üzemeltető lesz.
- [Az ExpressRoute](/azure/expressroute/expressroute-introduction) lehetővé teszi, kiterjesztheti helyszíni hálózatait a Microsoft cloud, amelyet egy kapcsolatszolgáltató biztosít egy privát kapcsolaton keresztül. 
- [Az active Directory Domain Services szolgál a Directory Services és a felhasználó hitelesítése
- [Az Azure globális csoportok](/azure/virtual-machines/windows/tutorial-availability-sets) biztosíthatja, hogy az Azure-ban üzembe helyezett virtuális gépek egy fürtben több elkülönített hardvercsomópont között legyenek elosztva. Ezáltal biztosítható, hogy ha hardveres vagy szoftveres hiba fordul elő az Azure-ban, az a virtuális gépeknek csak egy részhalmazát érintse, és a teljes megoldás továbbra is elérhető és működőképes maradjon. 
- [A Citrix ADC (NetScaler)](https://www.citrix.com/products/citrix-adc) van egy alkalmazáskézbesítési vezérlőként, amely végrehajtja az alkalmazásspecifikus forgalomelemzés intelligensen terjesztéséhez, optimalizálhatja és biztonságossá tétele (4. rétegbeli – 7. rétegbeli) 4-Layer-7. rétegbeli hálózati forgalmat a webes alkalmazásokhoz. 
- [A Citrix kirakat](https://www.citrix.com/products/citrix-virtual-apps-and-desktops/citrix-storefront.html) van egy vállalati alkalmazás-áruházban, amely biztonságosabbá teszi és leegyszerűsíti a telepítéseket, a modern, páratlan csaknem felhasználói környezet megvalósítása Citrix fogadó között bármilyen platformon. Kirakat megkönnyíti a többhelyes és többverziós Citrix virtuális alkalmazások és asztali környezeteit. 
- [A Citrix licenckiszolgáló](https://www.citrix.com/buy/licensing/overview.html) Citrix termékek licenceinek fogják kezelni.
- [A Citrix XenDesktops VDA](https://docs.citrix.com/en-us/citrix-virtual-apps-desktops-service) lehetővé teszi, hogy az alkalmazások és asztali gépeket. A VDA, amely az alkalmazások vagy a felhasználó virtuális asztalok fut a gépen telepítve van. Ez lehetővé teszi a gépek regisztrálása az Alkalmazáskézbesítési vezérlő, és a nagy felbontású élmény (HDX) a felhasználó-eszköz kapcsolat kezeléséhez.
- [Citrix kézbesítési vezérlő](https://docs.citrix.com/en-us/xenapp-and-xendesktop/7-15-ltsr/manage-deployment/delivery-controllers) a kiszolgálóoldali összetevő feladata a felhasználói hozzáférés kezelése és közvetítést, és a kapcsolatok optimalizálásával. Tartományvezérlők is asztali és kiszolgáló-rendszerképek létrehozása Machine létrehozási szolgáltatások biztosításához.

### <a name="alternatives"></a>Alternatív megoldások

- Nincsenek a VDI megoldásokkal, például a VMware, Workspot és egyéb Azure-ban támogatott több partnerek. Ez az architektúra meghatározott minta egy kivitelezett projektet, Citrix használt alapul.
- A Citrix biztosít egy felhőszolgáltatás, amely kivonatolja, ez az architektúra egy része. Alternatív megoldás lehet. További információkért lásd: [Citix-felhő](https://www.citrix.com/products/citrix-cloud).

## <a name="considerations"></a>Megfontolandó szempontok

- Ellenőrizze a [Citrix Linux követelmények](https://docs.citrix.com/en-us/linux-virtual-delivery-agent/current-release/system-requirements).
- Késés hatással lehet a teljes megoldás. Éles környezetben tesztelheti ennek megfelelően.
- A forgatókönyvtől függően a megoldás lehet szükségük gpu-kkal rendelkező virtuális gépek VDA. Ebben a megoldásban feltételezzük, hogy GPU ne legyen kötelező.

### <a name="availability-scalability-and-security"></a>Rendelkezésre állási, méretezhetőségi és biztonsági

- Ez a minta megoldás magas rendelkezésre állású összes szerepköre az engedélyezési kiszolgálótól eltérő lett tervezve. A környezet továbbra is működik egy 30 napos türelmi időszak alatt, ha a licenckiszolgáló offline állapotban van, nincs további redundancia jelentkezhet a kiszolgálón van szükség.
- Biztosít hasonló szerepkörök összes kiszolgálójára kell telepíteni a [rendelkezésre állási csoportok](/azure/virtual-machines/windows/manage-availability#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy).
- Ez a minta megoldás nem tartalmazza a vész-helyreállítási képességeit. [Az Azure Site Recovery](/azure/site-recovery/site-recovery-overview) lehet a megfelelő bővítményt az ezzel a kialakítással.
- Éles környezet felügyeleti megoldást kell végrehajtani, például [biztonsági mentési](/azure/backup/backup-introduction-to-azure-backup), [figyelési](/azure/monitoring-and-diagnostics/monitoring-overview) és [frissítéskezelés](/azure/automation/automation-update-management).
- Ez a minta megoldás működnie kell az egyidejű körülbelül 250 (körülbelül 50 – 60 VDA kiszolgálónként) vegyes használat rendelkező felhasználók. De, amely nagy mértékben függött használt alkalmazások típusa. Üzemi használatra szigorú terheléses tesztelés kell végrehajtani.

## <a name="deploy-this-scenario"></a>Ez a forgatókönyv megvalósítható

Központi telepítési információkat talál a hivatalos [Citrix dokumentáció](https://docs.citrix.com/en-us/citrix-virtual-apps-desktops/install-configure.html).

## <a name="pricing"></a>Díjszabás

- A Citrix XenDesktop-licencek nem szerepelnek az Azure szolgáltatási díjak.
- A Citrix NetScaler licenc használatalapú díjszabási modellel tartalmazza.
- A fenntartott példányok használatával jelentősen csökkenti a megoldás a számítási költségeket.
- A lehetőség nem része az ExpressRoute költségeit.

## <a name="next-steps"></a>További lépések

- Ellenőrizze a Citrix dokumentációjában, tervezési és telepítési [Itt](https://docs.citrix.com/en-us/citrix-virtual-apps-desktops/install-configure).
- A Citrix ADC (NetScaler) az Azure-beli üzembe helyezéséhez tekintse át a Resource Manager-sablonok a Citrix által biztosított [Itt](https://github.com/citrix/netscaler-azure-templates).
