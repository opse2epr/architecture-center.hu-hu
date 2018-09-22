---
title: Linux rendszerű virtuális asztalok Citrix-szel
description: Bevált forgatókönyv létrehozásához a VDI-környezetben az Azure-on a Citrix használata Linux rendszerű asztali számítógépeken.
author: miguelangelopereira
ms.date: 09/12/2018
ms.openlocfilehash: f86319680e06ef1b3076e8d6a98f6097a343ec17
ms.sourcegitcommit: b7e521ba317f4fcd3253c80ac0c0a355eaaa56c5
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/21/2018
ms.locfileid: "46534573"
---
# <a name="linux-virtual-desktops-with-citrix"></a>Linux rendszerű virtuális asztalok Citrix-szel

Ebben a forgatókönyvben minden iparágban, amelyet egy virtuális asztali infrastruktúra (VDI) Linux rendszerű asztali számítógépeken alkalmazható.  A folyamat egy felhasználói asztalról szigeten él a kiszolgáló az adatközpontban található virtuális gépen futó VDI hivatkozik. Az ügyfél ezen forgatókönyv esetén a VDI igényeiknek Citrix-alapú megoldás használatát választotta.

Szokás a szervezet számára, hogy heterogén környezetek több eszközzel és az alkalmazottak által használt operációs rendszereket. Bár a magas szintű biztonsági kihívást jelenthet, így konzisztens alkalmazás-hozzáférési. Linux rendszerű asztali számítógépeken VDI megoldás lehetővé teszi a biztosít hozzáférést, függetlenül az eszköz vagy felhasználó által használt operációs rendszer.

A minta megoldás bizonyos előnyöket nyújtják:

- A megosztott Linux rendszerű virtuális asztalok ROI megnövelni ugyanazon az infrastruktúrán történő hozzáférést ad a további felhasználókat. Központosított VDI-környezetben lévő erőforrások konszolidálásával a végfelhasználói eszközöket nem kell lenniük nagy teljesítményűek.
- Függetlenül a végfelhasználói eszközön egységes lesz a teljesítmény.
- Linuxos alkalmazások hozzáférést biztosít (beleértve a nem Linux), bármilyen eszközön.
- Bizalmas adatokat az összes elosztott alkalmazott leköthetőek az Azure-adatközpontban.

## <a name="potential-use-cases"></a>A lehetséges alkalmazási helyzetek

Ebben a forgatókönyvben a következő használati esetekhez, vegye figyelembe:

- Biztonságos hozzáférést biztosíthat az alapvető fontosságú, specializált Linux VDI asztalok Linux vagy a nem Linux rendszerű eszközök

## <a name="architecture"></a>Architektúra

[![](./media/azure-citrix-sample-diagram.png "Architektúra ábrája")](./media/azure-citrix-sample-diagram.png#lightbox)

Ez a minta megoldás lesz engedélyezése a vállalati hálózati hozzáférés Linux rendszerű virtuális asztalok:

- Az ExpressRoute jön létre a helyszíni környezet és az Azure között a gyors és megbízható kapcsolatot a felhőbe
- A Citrix XenDeskop megoldás üzembe helyezett virtuális asztali infrastruktúra
- A CitrixVDA Futtatás Ubuntu (vagy egy másik támogatott disztribúció)
- Az Azure hálózati biztonsági csoportok érvényes lesz a megfelelő hálózati hozzáférés-vezérlési listák
- Citrix ADC (Netscaler) teszi közzé, és a terhelést a Citrix-szolgáltatások
- Az Active Directory Domain Services tartományi használandó csatlakozzon a Citrix-kiszolgálók. VDA kiszolgálók nem lesz tartományhoz.
- Hibrid Azure File Sync lehetővé teszi, megosztott tárolót a megoldás között. Ha például használat távoli/home-megoldások.

A minta megoldás a következő termékváltozatok használhatók:

- A Citrix ADC (Netscaler): 2-x D4sv3 [NetScaler 12.0-s VPX Standard Edition 200 MB/s PAYG kép](https://azuremarketplace.microsoft.com/pt-br/marketplace/apps/citrix.netscalervpx-120?tab=PlansAndPrice)
- A Citrix licenckiszolgáló: 1 x D2s v3
- A Citrix VDA: 4 x D8s v3
- A Citrix kirakat: 2 x D2s v3
- Citrix kézbesítési vezérlő: 2 x D2s v3
- Tartományvezérlő: 2 x D2sv3
- Az Azure-kiszolgálók: 2 x D2sv3

> [!Note]
> A licenceket (mellett Netscaler) bring-your-saját licenc (használata BYOL)

### <a name="components"></a>Összetevők

- [Az Azure Virtual Network](/azure/virtual-network/virtual-networks-overview) lehetővé teszi az erőforrások, például virtuális gépek biztonságosan kommunikálnak egymással, az internethez, és a helyszíni hálózatokkal. Virtuális hálózatok adja meg az elkülönítés és Szegmentálás, szűrő és útvonal-forgalom és helyek közötti kapcsolat engedélyezését. Egy virtuális hálózat összes erőforrásának a forgatókönyv esetén használható.
- [Azure-beli hálózati biztonsági csoportok](/azure/virtual-network/security-overview) tartalmaznak, amelyek engedélyezik vagy megtagadják a bejövő vagy kimenő hálózati forgalmat a forrás vagy cél IP-cím, port és protokoll alapján biztonsági szabályokból álló listát. Ebben a forgatókönyvben a virtuális hálózatok a hálózati biztonsági csoport szabályait, amelyek korlátozzák a forgalmat az alkalmazás-összetevők közötti biztonságosak.
- [Az Azure load balancer](/azure/application-gateway/overview) osztja el a szabályok és az állapotmintákat megfelelően bejövő forgalmat. Egy terheléselosztót biztosít alacsony késéssel és nagy átviteli sebességet, és akár több milliónyi összes TCP és UDP-alkalmazás méretezhető. Belső terheléselosztó szolgál ebben a forgatókönyvben a Citrix Netscaler a forgalom elosztását.
- [Hibrid Azure File Sync](https://github.com/MicrosoftDocs/azure-docs/edit/master/articles/storage/files/storage-sync-files-planning.md) minden megosztott tároló használható. A storage hibrid File Sync használatával két fájlkiszolgálók replikálja.
- [Az Azure SQL Database](https://docs.microsoft.com/en-us/azure/sql-database/) egy relációs adatbázis-a-szolgáltatás (DBaaS) a Microsoft SQL Server adatbázismotor legújabb stabil verziója alapján. A Citrix-adatbázisokat üzemeltető lesz.
- [Az ExpressRoute](https://docs.microsoft.com/en-us/azure/expressroute/expressroute-introduction) lehetővé teszi, kiterjesztheti helyszíni hálózatait a Microsoft cloud, amelyet egy kapcsolatszolgáltató biztosít egy privát kapcsolaton keresztül. 
- [Az active Directory Domain Services szolgál a Directory Services és a felhasználó hitelesítése
- [Az Azure globális csoportok](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/tutorial-availability-sets) biztosíthatja, hogy az Azure-ban üzembe helyezett virtuális gépek egy fürtben több elkülönített hardvercsomópont között legyenek elosztva. Ezáltal biztosítható, hogy ha hardveres vagy szoftveres hiba fordul elő az Azure-ban, az a virtuális gépeknek csak egy részhalmazát érintse, és a teljes megoldás továbbra is elérhető és működőképes maradjon. 
- [A Citrix ADC (Netscaler)](https://www.citrix.com/products/citrix-adc/) van egy alkalmazáskézbesítési vezérlőként, amely végrehajtja az alkalmazásspecifikus forgalomelemzés intelligensen terjesztéséhez, optimalizálhatja és biztonságossá tétele (4. rétegbeli – 7. rétegbeli) 4-Layer-7. rétegbeli hálózati forgalmat a webes alkalmazásokhoz. 
- [A Citrix kirakat](https://www.citrix.com/products/citrix-virtual-apps-and-desktops/citrix-storefront.html) van egy vállalati alkalmazás-áruházban, amely biztonságosabbá teszi és leegyszerűsíti a telepítéseket, a modern, páratlan csaknem felhasználói környezet megvalósítása Citrix fogadó között bármilyen platformon. Kirakat megkönnyíti a többhelyes és többverziós Citrix virtuális alkalmazások és asztali környezeteit. 
- [A Citrix licenckiszolgáló](https://www.citrix.com/buy/licensing/overview.html) fogja kezelni a licenceket a Citrix-termékek.
- [A Citrix XenDesktops VDA](https://docs.citrix.com/en-us/citrix-virtual-apps-desktops-service.html) lehetővé teszi, hogy az alkalmazások és asztali gépeket. A VDA, amely az alkalmazások vagy a felhasználó virtuális asztalok fut a gépen telepítve van. Ez lehetővé teszi a gépek regisztrálása az Alkalmazáskézbesítési vezérlő, és a nagy felbontású élmény (HDX) a felhasználó-eszköz kapcsolat kezeléséhez.
- [Citrix kézbesítési vezérlő](https://docs.citrix.com/en-us/xenapp-and-xendesktop/7-15-ltsr/manage-deployment/delivery-controllers.html) a kiszolgálóoldali összetevő felelős a felhasználói hozzáférés kezelése és közvetítést, és a kapcsolatok optimalizálásával. Tartományvezérlők is asztali és kiszolgáló-rendszerképek létrehozása Machine létrehozási szolgáltatások biztosításához.

### <a name="alternatives"></a>Alternatív megoldások

- Nincsenek a VDI megoldásokkal, például a VMware, Workspot és egyéb Azure-ban támogatott több partnerek. Ez az architektúra meghatározott minta egy kivitelezett projektet, Citrix használt alapul.
- A Citrix biztosít egy Felhőszolgáltatás, amely kivonatolja, ez az architektúra egy része. Alternatív megoldás lehet. Lásd: [Citix-felhő](https://www.citrix.com/products/citrix-cloud/)

## <a name="considerations"></a>Megfontolandó szempontok

- Ellenőrizze a [Citrix Linux-követelmények](https://docs.citrix.com/en-us/linux-virtual-delivery-agent/current-release/system-requirements.html) 
- Késés hatással lehet a teljes megoldás. Éles környezetben tesztelheti ennek megfelelően.
- A forgatókönyvtől függően a megoldás lehet szükségük gpu-kkal rendelkező virtuális gépek VDA. Ebben a megoldásban feltételezzük, hogy GPU ne legyen kötelező.

### <a name="availability-scalability-and-security"></a>Rendelkezésre állási, méretezhetőségi és biztonsági

- Ez a minta megoldás magas rendelkezésre állás mellett a licencelési kiszolgáló összes szerepköre lett tervezve. A környezet továbbra is működik egy 30 napos türelmi időszakban, ha a licenckiszolgáló offline állapotban van, nincs további redundancia jelentkezhet a kiszolgálón van szükség.
- Biztosít hasonló szerepkörök összes kiszolgálójára kell telepíteni a [rendelkezésre állási csoportok](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/manage-availability#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy).
- Ez a minta megoldás nem tartalmazza a vész-helyreállítási képességeit. [Az Azure Site Recovery](https://docs.microsoft.com/en-us/azure/site-recovery/site-recovery-overview) lehet a megfelelő bővítményt az ezzel a kialakítással.
- Éles környezet felügyeleti megoldást kell végrehajtani, például [biztonsági mentési](https://docs.microsoft.com/en-us/azure/backup/backup-introduction-to-azure-backup), [figyelési](https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/monitoring-overview) és [frissítéskezelés](https://docs.microsoft.com/en-us/azure/automation/automation-update-management).
- Ez a minta megoldás működnie kell az egyidejű körülbelül 250 (körülbelül 50 – 60 VDA kiszolgálónként) vegyes használat rendelkező felhasználók. De, amely nagy mértékben függött használt alkalmazások típusa. Üzemi használatra szigorú terheléses tesztelés kell végrehajtani.

## <a name="deploy-this-scenario"></a>Ez a forgatókönyv megvalósítható

Központi telepítési információkat talál a hivatalos [Citrix dokumentáció](https://docs.citrix.com/en-us/citrix-virtual-apps-desktops/install-configure.html).

## <a name="pricing"></a>Díjszabás

- A Citrix XenDestop licenceket nem szerepelnek az Azure szolgáltatási díjak
- A Citrix Netscaler licenccel a használatalapú díjszabási modellel
- Fenntartott példányok használatával jelentősen csökkentheti a számítási költség a megoldás
- Az ExpressRoute költségeit a lehetőség nem része

## <a name="next-steps"></a>További lépések

- Ellenőrizze a Citrix dokumentációjában, tervezési és telepítési [Itt](https://docs.citrix.com/en-us/citrix-virtual-apps-desktops/install-configure.html)
- A Citrix ADC (Netscaler) az Azure-beli üzembe helyezéséhez. Ellenőrizze a Resource Manager-sablonok a Citrix által biztosított [Itt](https://github.com/citrix/netscaler-azure-templates). 
