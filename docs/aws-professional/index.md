---
title: Azure AWS-szakembereknek
description: Megismerkedhet a Microsoft Azure-fiókok, -szolgáltatások és -platform alapjaival. Emellett az AWS és az Azure platform legfontosabb hasonlóságait és különbségeit is megismerheti. Hasznosítsa az AWS-sel kapcsolatos tapasztalatait az Azure-ban.
keywords: AWS-szakértők, az Azure összehasonlítása, az AWS összehasonlítása, azure és aws közötti különbségek, azure és aws
author: lbrader
ms.date: 03/24/2017
pnp.series.title: Azure for AWS Professionals
ms.openlocfilehash: 0af0890d383d22db0ed9d3b445cdd5b561b498ae
ms.sourcegitcommit: f665226cec96ec818ca06ac6c2d83edb23c9f29c
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/16/2018
ms.locfileid: "31012620"
---
# <a name="azure-for-aws-professionals"></a>Azure AWS-szakembereknek

Ennek a cikknek a segítségével az Amazon Web Services- (AWS-) szakértők megismerkedhetnek a Microsoft Azure-fiókok, -szolgáltatások és -platform alapjaival. Emellett az AWS és az Azure platform legfontosabb hasonlóságait és különbségeit is ismerteti.

Az oktatóanyagból a következőket sajátíthatja el:

* A fiókok és az erőforrások rendszerezésének módja az Azure-ban.
* Az elérhető megoldások struktúrája az Azure-ban.
* A legfontosabb Azure-szolgáltatások és AWS-szolgáltatások közötti különbségek.

Az Azure és az AWS a képességeit az idők során egymástól függetlenül fejlesztette ki, ezért fontos implementációbeli és tervezésbeli különbségekkel kell számolni.

## <a name="overview"></a>Áttekintés

Az AWS-hez hasonlóan a Microsoft Azure is egy számítási, tárolási, adatbázis- és hálózati szolgáltatásokból álló alapkészlet köré épült. A két platform számos esetben – mind a kínált termékek, mind a szolgáltatások tekintetében – egyezőségeket mutat. Mind az AWS, mind az Azure használatával magas rendelkezésre állású, Windows- vagy Linux-gazdagépen futó megoldásokat hozhat létre. Így ha a Linux- vagy OSS-alapú technológiák fejlesztése terén jártas, mindkét platform előnyeit képes lesz kihasználni.

Mindkét platform hasonló képességekkel rendelkezik, azonban a funkciókat kiszolgáló erőforrások sokszor eltérő módon vannak rendszerezve. A megoldások létrehozásához szükséges szolgáltatások közötti átjárhatóság azonban nem mindig szembetűnő. Az is előfordul, hogy egy adott szolgáltatás az egyik platformon létezik, míg a másikon nem. Érdemes vetnie egy pillantást az [Azure- és AWS-szolgáltatások összehasonlítására](services.md).

## <a name="accounts-and-subscriptions"></a>Fiókok és előfizetések

Az Azure-szolgáltatások megvásárlásakor számos díjszabás közül választhat a cég mérete és szükségletei alapján. A részletekért tekintse meg a [díjszabás áttekintését](https://azure.microsoft.com/pricing/) tartalmazó oldalt.

Az [Azure-előfizetések](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-infrastructure-subscription-accounts-guidelines/) olyan csoportosított erőforrások, amelyek egy hozzárendelt tulajdonossal rendelkeznek, aki a számlázásért és az engedélyek kezeléséért felelős. Az AWS-sel ellentétben, ahol az AWS-fiókon belül létrehozott bármely erőforrás kizárólag ahhoz a fiókhoz kötődik, az előfizetések a tulajdonosi fiókoktól függetlenül léteznek, és szükség esetén új tulajdonosokhoz rendelhetők.

![Az AWS-fiókok és az Azure-előfizetések szerkezetének és tulajdonosi viszonyainak összehasonlítása](./images/azure-aws-account-compare.png "Az AWS-fiókok és az Azure-előfizetések szerkezetének és tulajdonosi viszonyainak összehasonlítása")
<br/>*Az AWS-fiókok és az Azure-előfizetések szerkezetének és tulajdonosi viszonyainak összehasonlítása*
<br/><br/>

Az előfizetésekhez három különböző típusú rendszergazdai fiók tartozhat:

-   **Fiókadminisztrátor** – Az előfizetés tulajdonosa, illetve az a fiók, amelyik az előfizetésben használt erőforrásokért fizet. A fiókadminisztrátor csak az előfizetés tulajdonjogának átruházásával módosítható.

-   **Szolgáltatás-rendszergazda** – Ez a fiók jogosultsággal rendelkezik erőforrások létrehozásához és kezeléséhez az előfizetésben, de nem felelős a számlázásért. A fiókadminisztrátor és a szolgáltatás-rendszergazda alapértelmezés szerint ugyanazon fiókhoz van rendelve. A fiókadminisztrátor külön felhasználót rendelhet a szolgáltatás-rendszergazda fiókjához, az előfizetés műszaki üzemeltetési feladatainak ellátása érdekében. Minden előfizetéshez egy szolgáltatás-rendszergazda tartozik.

-   **Társadminisztrátor** – Egy előfizetéshez egyszerre több társadminisztrátor is hozzárendelhető. A társadminisztrátorok nem módosíthatják a szolgáltatás-rendszergazdát, ezenkívül azonban teljes körűen kezelhetik az előfizetéshez tartozó erőforrásokat és felhasználókat.

Az előfizetési szint alatt a felhasználói szerepkörök és az egyéni engedélyek adott erőforrásokhoz is hozzárendelhetők, hasonlóan ahhoz, ahogyan az AWS-ben az IAM-felhasználók és -csoportok engedélykiosztása működik. Az Azure-ban minden felhasználói fiók egy Microsoft-fiókhoz vagy egy (Azure Active Directoryval felügyelt) munkahelyi vagy intézményi fiókhoz van rendelve.

Az AWS-fiókokhoz hasonlóan az előfizetések alapértelmezett szolgáltatási kvótákkal és korlátokkal rendelkeznek. A korlátok teljes listáját [az Azure-előfizetések és -szolgáltatások korlátozásait, kvótáit és megkötéseit](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/) ismertető cikkben találja.
Ezek a korlátok a maximális értékre növelhetők [egy támogatási kérelem a felügyeleti portálon történő kitöltésével](https://blogs.msdn.microsoft.com/girishp/2015/09/20/increasing-core-quota-limits-in-azure/).

### <a name="see-also"></a>Lásd még

-   [Azure-rendszergazdai szerepkörök hozzáadása vagy módosítása](https://azure.microsoft.com/documentation/articles/billing-add-change-azure-subscription-administrator/)

-   [Az Azure számlázási és napi használati adatainak letöltése](https://azure.microsoft.com/documentation/articles/billing-download-azure-invoice-daily-usage-date/)

## <a name="resource-management"></a>Erőforrás-kezelés

Az Azure-ban az „erőforrás” kifejezés ugyanúgy fordul elő, mint az AWS-ben, tehát bármilyen számítási példányra, tárolási objektumra, hálózati eszközre vagy egyéb, a platformon létrehozható vagy konfigurálható entitásra utalhat.

Az Azure-erőforrások üzembe helyezése és kezelése az alábbi két modell egyike alapján történik: az [Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview) vagy a régebbi, [klasszikus Azure üzemi modell](/azure/azure-resource-manager/resource-manager-deployment-model).
Minden új erőforrás létrehozása a Resource Manager-alapú modell alapján történik.

### <a name="resource-groups"></a>Erőforráscsoportok

Az Azure és az AWS is rendelkezik „erőforráscsoportnak” nevezett entitásokkal, amelyek a virtuális gépekhez, tárterületekhez és virtuális hálózati eszközökhöz hasonló erőforrások rendszerezésére szolgálnak. Az [Azure-erőforráscsoportok](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/) azonban nem hasonlíthatók össze közvetlenül az AWS-erőforráscsoportokkal.

Míg az AWS-ben az erőforrások több erőforráscsoporthoz is tartozhatnak, az Azure-erőforrások mindig csak egy erőforráscsoporthoz tartoznak. Az egyik erőforráscsoportban létrehozott erőforrások más csoportba is áthelyezhetők, de egyszerre csak egy erőforráscsoportnak lehetnek tagjai. Az erőforráscsoportok az Azure Resource Manager által használt alapvető csoportosítási egységek.

Az erőforrások [címkék](https://azure.microsoft.com/documentation/articles/resource-group-using-tags/) segítségével is rendszerezhetők.
A címkék olyan kulcs-érték párok, amelyekkel az erőforráscsoport-tagságtól függetlenül csoportosíthatók az erőforrások az előfizetésen belül.

### <a name="management-interfaces"></a>Felügyeleti felületek

Az Azure több módot is kínál az erőforrások kezelésére:

-   [Webes felület](https://azure.microsoft.com/documentation/articles/resource-group-portal/).
    Az AWS irányítópultjához hasonlóan az Azure Portal teljes körű, webalapú kezelőfelületet biztosít az Azure-erőforrásokhoz.

-   [REST API](https://azure.microsoft.com/documentation/articles/resource-manager-rest-api/).
    Az Azure Resource Manager REST API szoftveres hozzáférést nyújt az Azure Portalon elérhető legtöbb funkcióhoz.

-   [Parancssor](https://azure.microsoft.com/documentation/articles/xplat-cli-azure-resource-manager/).
    Az Azure CLI 2.0 eszköz egy, az Azure-erőforrások létrehozására és kezelésére alkalmas parancssori felületet biztosít. Az Azure CLI csak [Windows, Linux és Mac OS rendszeren](https://aka.ms/azurecli2) érhető el.

-   [PowerShell](https://azure.microsoft.com/documentation/articles/powershell-azure-resource-manager/).
    Az Azure PowerShell-modulok segítségével automatizált felügyeleti feladatokat futtathat egy parancsfájl használatával. A PowerShell csak [Windows, Linux és Mac OS rendszeren](https://github.com/PowerShell/PowerShell) érhető el.

-   [Sablonok](https://azure.microsoft.com/documentation/articles/resource-group-authoring-templates/).
    Az Azure Resource Manager-sablonok az AWS CloudFormation szolgáltatásához hasonló, JSON-sablonra épülő erőforráskezelési képességeket nyújtanak.

Az ilyen kezelőfelületekben az erőforráscsoport központi szerepet játszik az Azure-erőforrások létrehozásában, üzembe helyezésében vagy módosításában. Ez hasonlít a „verem” az AWS-erőforrások a CloudFormation-üzembehelyezések során történő csoportosításakor betöltött szerepére.

A kezelőfelületek szintaxisa és szerkezete különbözik az AWS-beli megfelelőiktől, azonban összehasonlítható képességekkel is rendelkeznek. Továbbá számos, az AWS-ben használt, harmadik féltől származó felügyeleti eszköz – például a [Hashicorp Terraform](https://www.terraform.io/docs/providers/azurerm/) és a [Netflix Spinnaker](http://www.spinnaker.io/) – szintén elérhető az Azure-ban.

### <a name="see-also"></a>Lásd még

-   [Az Azure-erőforráscsoportokkal kapcsolatos irányelvek](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-infrastructure-resource-groups-guidelines/)

## <a name="regions-and-zones-high-availability"></a>Régiók és zónák (magas rendelkezésre állás)

A hibák a hatásukat illetően eltérőek lehetnek. Bizonyos hardveres hibák – például egy meghibásodott lemez – csak egyetlen gazdagépet érintenek. Egy meghibásodott hálózati kapcsoló egy teljes kiszolgálószekrényt érinthet. Kevésbé gyakori meghibásodások teljes adatközpontok működését zavarhatják meg, például egy áramkimaradás esetén az adatközpontban. Ritkán egy teljes régió válhat elérhetetlenné.

Az alkalmazások rugalmasságának egyik legfőbb eleme a redundancia. Ezt a redundanciát azonban meg kell terveznie az alkalmazás fejlesztésekor. A szükséges redundancia szintje az üzleti követelményeitől függ. Nem szükséges minden alkalmazáshoz régiók közötti redundancia, hogy védelmet nyújtson a regionális kimaradással szemben. Általában a nagyobb redundancia és megbízhatóság magasabb költséget és összetettséget von magával.  

Az AWS-ben egy régió kettő vagy több rendelkezésre állási zónára oszlik. A rendelkezésre állási zóna megfelel a földrajzi régió egyik fizikailag elkülönített adatközpontjának. Az Azure számos funkciót kínál, amellyel az alkalmazások számára biztosítja a redundanciát a meghibásodások minden szintjén. Ezek közé tartoznak a **rendelkezésre állási csoportok**, a **rendelkezésre állási zónák** és a **párosított régiók**. 

![](../resiliency/images/redundancy.svg)

Az alábbi táblázat bemutatja ezek mindegyikét.

| &nbsp; | Rendelkezésre állási csoport | Rendelkezésre állási zóna | Párosított régió |
|--------|------------------|-------------------|---------------|
| Hiba hatóköre | Kiszolgálószekrény | Adatközpont | Régió |
| Kérések útválasztása | Load Balancer | Zónák közötti terheléselosztó | Traffic Manager |
| Hálózati késleltetés | Nagyon alacsony | Alacsony | Közepes vagy magas |
| Virtuális hálózat  | VNet | VNet | Régiók közötti virtuális hálózatok közötti társviszony |

### <a name="availability-sets"></a>Rendelkezésre állási csoportok 

A helyi hardverhibák, például lemezek vagy hálózati kapcsolók meghibásodása elleni védelem érdekében helyezzen üzembe két vagy több virtuális gépet egy rendelkezésre állási csoportban. Egy rendelkezésre állási csoport két vagy több *tartalék tartományból* áll, amelyek közös áramforrással és hálózati kapcsolóval rendelkeznek. A rendelkezésre állási csoportban található virtuális gépek el vannak osztva a tartalék tartományok között, ezért ha hardverhiba merül fel az egyik tartalék tartományban, a hálózati forgalom átirányítható a többi tartalék tartományban található virtuális gépekre. További információt a rendelkezésre állási csoportokról [a Windows rendszerű virtuális gépek rendelkezésre állásának az Azure-ban történő kezelését](/azure/virtual-machines/windows/manage-availability) ismertető cikkben talál.

A virtuális gépek a rendelkezésre állási csoporthoz való hozzáadásukkor egy [frissítési tartományt](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-manage-availability/) is kapnak. A frissítési tartomány virtuális gépek olyan csoportja, amelyben egyezik a tervezett karbantartási események időpontja. A virtuális gépek több tartományban való elosztásával biztosítható, hogy a tervezett frissítési és javítási események egyszerre csak a szóban forgó virtuális gépek egy részét érintsék.

A rendelkezésre állási csoportokat a példány az alkalmazásban betöltött szerepe szerint kell meghatározni, hogy minden példány működőképes legyen minden szerepkörben. Például egy háromszintű webalkalmazásban külön rendelkezésre állási csoportot kell létrehozni a felhasználói felülethez, az alkalmazáshoz és az adatrétegekhez is.

![Azure-beli rendelkezésre állási csoportok az egyes alkalmazás-szerepkörökhöz](./images/three-tier-example.png "Rendelkezésre állási csoportok az egyes alkalmazás-szerepkörökhöz")

### <a name="availability-zones"></a>Rendelkezésre állási zónák

A [rendelkezésre állási zónák](/azure/availability-zones/az-overview) egy Azure-régió fizikailag elkülönített zónáit jelentik. Mindegyik rendelkezésre állási zóna különálló áramforrással, hálózattal és hűtéssel rendelkezik. Ha a virtuális gépeket több rendelkezésre állási zónában helyezi üzembe, azzal segít megvédeni az alkalmazásokat a teljes adatközpontra kiterjedő meghibásodásokkal szemben. 

### <a name="paired-regions"></a>Párosított régiók

Annak érdekében, hogy védelmet nyújtson egy alkalmazás számára regionális kimaradás esetén, helyezze üzembe az alkalmazást egyszerre több régióban, és az [Azure Traffic Manager][traffic-manager] használatával ossza el az internetes forgalmat a különböző régiók között. Minden egyes Azure-régió párban áll egy másikkal. Ezek együtt egy [régiópárt][paired-regions] alkotnak. Dél-Brazília kivételével a régiópárok azonos földrajzi helyen találhatók, hogy adóügyi és törvényi szempontból megfeleljenek az adatok tárolási helyére vonatkozó előírásoknak.

A rendelkezésre állási zónáktól eltérően, amelyek fizikailag különálló adatközpontok, de egymáshoz viszonylag közeli földrajzi helyen találhatók, a párosított régiókat általában legalább 500 kilométernyi távolság választja el egymástól. Ennek célja, hogy biztosíthassuk azt, hogy a nagy méretű katasztrófák se érintsék a pár mindkét tagját. A szomszédos párok esetén konfigurálható az adatbázisok és a tárolási szolgáltatások adatainak szinkronizálása, és megadható, hogy a platformfrissítések egyszerre csak a pár egyik tagján legyenek kibocsátva.

Az Azure [georedundáns tárolási szolgáltatásának](https://azure.microsoft.com/documentation/articles/storage-redundancy/#geo-redundant-storage) biztonsági mentése automatikusan a megfelelő párosított régióra történik. Minden egyéb erőforrás esetében a párosított régiókat használó teljesen redundáns megoldás létrehozása egyet jelent a megoldás teljes másolatának létrehozásával mindkét régióban.


### <a name="see-also"></a>Lásd még

-   [Az Azure-beli virtuális gépek régiók szerinti csoportosítása és rendelkezésre állása](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-regions-and-availability/)

-   [Magas rendelkezésre állás az Azure-alkalmazásokhoz](../resiliency/high-availability-azure-applications.md)

-   [Azure-alkalmazások vészhelyreállítása](../resiliency/disaster-recovery-azure-applications.md)

-   [Az Azure-beli linuxos virtuális gépek tervezett karbantartása](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-planned-maintenance/)

## <a name="services"></a>Szolgáltatások

A szolgáltatások platformok közötti megfeleltethetőségének teljes listájáért tekintse meg az [AWS és az Azure szolgáltatás-összehasonlító mátrixát](https://aka.ms/azure4aws-services).

Nem minden Azure-termék és -szolgáltatás érhető el minden régióban. A részletekért tekintse meg a [termékek régiónkénti](https://azure.microsoft.com/regions/services/) elérhetőségét. Az egyes Azure-termékek vagy -szolgáltatások minimális üzemideje és szolgáltatáskiesés esetén alkalmazott jóváírási szabályzata a [szolgáltatói szerződések](https://azure.microsoft.com/support/legal/sla/) oldalán tekinthető meg.

Az alábbi szakaszok röviden összefoglalják a leggyakrabban használt funkciók és szolgáltatások az AWS és az Azure platform közötti különbségeit.

### <a name="compute-services"></a>Számítási szolgáltatások

#### <a name="ec2-instances-and-azure-virtual-machines"></a>EC2-példányok és Azure-beli virtuális gépek

Bár az AWS-példánytípusok és az Azure-beli virtuális gépek méretei hasonlóan bonthatók le, a RAM, a CPU és a tárolási képességeik különböznek.

-   [Amazon EC2-példánytípusok](https://aws.amazon.com/ec2/instance-types/)

-   [A virtuális gépek méretei az Azure-ban (Windows)](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/)

-   [A virtuális gépek méretei az Azure-ban (Linux)](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-sizes/)

Az AWS másodpercalapú számlázásával ellentétben az Azure-beli igény szerinti virtuális gépek percalapú számlázást alkalmaznak.

Az Azure-ban nem állnak rendelkezésre az EC2-beli kihasználatlan példányoknak vagy dedikált gazdagépeknek megfelelő megoldások.

#### <a name="ebs-and-azure-storage-for-vm-disks"></a>EBS és Azure Storage virtuálisgép-lemezekhez

Az Azure-beli virtuális gépek tartós adattárolását a Blob Storage tárolóban található [adatlemezek](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-about-disks-vhds/) teszik lehetővé. Ez hasonló ahhoz, ahogyan az EC2-példányok tárolják a lemezköteteket az Elastic Block Store (EBS) tárolóban. Az [ideiglenes Azure-tároló](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/) ugyanazt a kis késleltetésű, ideiglenes írási-olvasási tárolót is nyújtja, mint amelyet az EC2-példánytároló (más néven ideiglenes tároló).

Az [Azure Premium Storage](https://docs.microsoft.com/azure/storage/storage-premium-storage) használatával érhető el a nagyobb teljesítményigényű lemezhasználat.
Ez hasonló az AWS által nyújtott kiosztott IOPS-tárolási lehetőségekhez.

#### <a name="lambda-azure-functions-azure-web-jobs-and-azure-logic-apps"></a>Lambda, Azure Functions, Azure Web-Jobs és Azure Logic Apps

Az [Azure Functions](https://azure.microsoft.com/services/functions/) az AWS Lambda elsődleges megfelelője a kiszolgáló nélküli, igény szerinti kódok kiszolgálásában.
De a Lambda-funkciók más Azure-szolgáltatásokkal is megfeleltethetők:

-   A [WebJobs](https://azure.microsoft.com/documentation/articles/web-sites-create-web-jobs/) lehetővé teszi, hogy ütemezett vagy folyamatosan futó háttérfeladatokat hozzon létre.

-   A [Logic Apps](https://azure.microsoft.com/services/logic-apps/) kommunikációs, integrációs és üzleti szabálykezelési szolgáltatásokat nyújt.

#### <a name="autoscaling-azure-vm-scaling-and-azure-app-service-autoscale"></a>Automatikus skálázás, Azure-beli virtuális gépek méretezése és Azure App Service automatikus méretezése

Az Azure-ban az automatikus skálázást két szolgáltatás kezeli:

-   A [virtuális gépek méretezési csoportjai](https://azure.microsoft.com/documentation/articles/virtual-machine-scale-sets-overview/) lehetővé teszik a virtuális gépek egyező készletének üzembe helyezését és kezelését. A példányok automatikusan skálázható száma a teljesítményigényektől függ.

-   Az [App Service automatikus méretezése](https://azure.microsoft.com/documentation/articles/web-sites-scale/) az Azure App Service-megoldások automatikus méretezésének lehetőségét nyújtja.


#### <a name="container-service"></a>Container Service
Az [Azure Container Service](https://docs.microsoft.com/azure/container-service/container-service-intro) támogatja a Docker Swarmon, a Kubernetesen vagy a DC/OS-en keresztül kezelt Docker-tárolókat.

#### <a name="other-compute-services"></a>Egyéb számítási szolgáltatások 


Az Azure számos olyan számítási szolgáltatást nyújt, amelyeknek nincs közvetlen megfelelője az AWS-ben:

-   Az [Azure Batch](https://azure.microsoft.com/documentation/articles/batch-technical-overview/) lehetővé teszi a virtuális gépek skálázható gyűjteményén átnyúló, nagy számítási igényű munkák kezelését.

-   A [Service Fabric](https://azure.microsoft.com/documentation/articles/service-fabric-overview/) a skálázható [mikroszolgáltatás](https://azure.microsoft.com/documentation/articles/service-fabric-overview-microservices/)-megoldások fejlesztésére és üzemeltetésére szolgáló platform.

#### <a name="see-also"></a>Lásd még

-   [Linuxos virtuális gép létrehozása az Azure-ban a portál használatával](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-quick-create-portal/)

-   [Azure-referenciaarchitektúra: Linuxos virtuális gép futtatása az Azure-ban](https://azure.microsoft.com/documentation/articles/guidance-compute-single-vm-linux/)

-   [Ismerkedés a Node.js-webalkalmazásokkal az Azure App Service-ben](https://azure.microsoft.com/documentation/articles/app-service-web-nodejs-get-started/)

-   [Azure-referenciaarchitektúra: Alapszintű webalkalmazás](https://azure.microsoft.com/documentation/articles/guidance-web-apps-basic/)

-   [Az első Azure-függvény létrehozása](https://azure.microsoft.com/documentation/articles/functions-create-first-azure-function/)

### <a name="storage"></a>Tárolás

#### <a name="s3ebsefs-and-azure-storage"></a>S3/EBS/EFS és Azure Storage

Az AWS platformon a felhőalapú tárolás elsődlegesen három szolgáltatásra osztható fel:

-   **Simple Storage Service (S3)** – alapszintű objektumtároló. Az internetről elérhető API-val teszi elérhetővé az adatokat.

-   **Elastic Block Storage (EBS)** – blokkszintű tároló, amely egyetlen virtuális gép elérésére szolgál.

-   **Elastic File System (EFS)** – több ezer EC2-példány megosztott tárolójaként használandó fájltároló.

Az Azure Storage-ban az előfizetéshez kötött [tárfiókokkal](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) a következő tárolási szolgáltatásokat hozhatja létre és kezelheti:

-   [Blob Storage](https://azure.microsoft.com/documentation/articles/storage-create-storage-account/) – képes tárolni bármilyen szöveget vagy bináris adatot, például dokumentumot, médiafájlt vagy egy alkalmazástelepítőt. A Blob Storage tárolót beállíthatja magánjellegű hozzáféréshez vagy a tartalmak nyilvános, az interneten keresztüli megosztásához. A Blob Storage ugyanazt a célt szolgálja, mint az AWS S3 és az EBS.
-   [Table Storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/) – a strukturált adatkészleteket tárolja. A Table Storage a NoSQL-kulcsattribútumok adattára, amely gyors fejlesztési lehetőségeket és nagy adatmennyiségek gyors elérését biztosítja. Hasonló az AWS SimpleDB és DynamoDB szolgáltatásához.

-   [Queue Storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) – üzenetküldést biztosít a munkafolyamat-feldolgozáshoz és a felhőszolgáltatás összetevői közötti kommunikációhoz.

-   [File Storage](https://azure.microsoft.com/documentation/articles/storage-java-how-to-use-file-storage/) – közös tárterületet biztosít a szabványos SMB protokollt használó örökölt alkalmazások számára. A fájltároló ugyanúgy használható, mint az AWS platform EFS-e.
 
#### <a name="glacier-and-azure-storage"></a>Glacier és Azure Storage 

Az [Azure Archive Blob Storage](/azure/storage/blobs/storage-blob-storage-tiers#archive-access-tier) az AWS Glacier társzolgáltatáshoz hasonlítható. Olyan adatok tárolására tervezték, amelyeket legalább 180 napig tárol, és amelyeknél nem okoz gondot a lekérés több órás késése. 

Olyan adatokhoz, amelyeket ritkán használ, azonban az elérésükre azonnal szükség van, az [Azure Blob Storage-réteg](/azure/storage/blobs/storage-blob-storage-tiers#cool-access-tier) olcsóbb tárhelyet kínál, mint a szabványos blobtárolók. Ez a tárolási szint az AWS S3 ritka elérésű társzolgáltatásához hasonlítható.

#### <a name="see-also"></a>Lásd még

-   [A Microsoft Azure Storage teljesítmény- és skálázhatósági ellenőrzőlistája](https://azure.microsoft.com/documentation/articles/storage-performance-checklist/)

-   [Biztonsági útmutató az Azure Storage-hoz](https://azure.microsoft.com/documentation/articles/storage-security-guide/)

-   [Minták és gyakorlatok: Content Delivery Network- (CDN-) útmutató](https://azure.microsoft.com/documentation/articles/best-practices-cdn/)

### <a name="networking"></a>Hálózat

#### <a name="elastic-load-balancing-azure-load-balancer-and-azure-application-gateway"></a>Elastic Load Balancing, Azure Load Balancer és Azure Application Gateway

A két Elastic Load Balancing-szolgáltatás Azure-beli megfelelői a következők:

-   A [Load Balancer](https://azure.microsoft.com/documentation/articles/load-balancer-overview/) ugyanazokat a képességeket nyújtja, mint az AWS Classic Load Balancer, így a hálózati szinten több virtuális gép között oszthatja el az adatforgalmat. Feladatátvételi képességet is nyújt.

-   Az [Application Gateway](https://azure.microsoft.com/documentation/articles/application-gateway-introduction/) alkalmazásszintű, szabályalapú útválasztást nyújt, amely az AWS Application Load Balancerhez hasonlítható.

#### <a name="route-53-azure-dns-and-azure-traffic-manager"></a>Route 53, Azure DNS és Azure Traffic Manager

Az AWS-ben a Route 53 a DNS-nevek kezelését és DNS-szintű forgalom-útválasztást és feladatátvételi szolgáltatásokat is nyújt. Az Azure-ban ezt két szolgáltatás kezeli:

-   Az [Azure DNS](https://azure.microsoft.com/documentation/services/dns/) tartomány- és DNS-kezelést nyújt.

-   A [Traffic Manager][traffic-manager] DNS-szintű forgalom-útválasztást, terheléselosztást és feladatátvételi képességeket nyújt.

#### <a name="direct-connect-and-azure-expressroute"></a>Direct Connect és Azure ExpressRoute

Az Azure hasonló, helyek közötti dedikált kapcsolatokat nyújt az [ExpressRoute](https://azure.microsoft.com/documentation/services/expressroute/) szolgáltatáson keresztül. Az ExpressRoute lehetővé teszi, hogy a helyi hálózatot közvetlenül az Azure-erőforrásokhoz csatlakoztassa egy dedikált magánhálózati kapcsolaton keresztül. Az Azure hagyományosabb, [helyek közötti VPN-kapcsolatokat](https://azure.microsoft.com/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/) is elérhetővé tesz, alacsonyabb áron.

#### <a name="see-also"></a>Lásd még

-   [Virtuális hálózat létrehozása az Azure Portallal](https://azure.microsoft.com/documentation/articles/virtual-networks-create-vnet-arm-pportal/)

-   [Az Azure-beli virtuális hálózatok megtervezése és kialakítása](https://azure.microsoft.com/documentation/articles/virtual-network-vnet-plan-design-arm/)

-   [Ajánlott Azure-hálózati biztonsági eljárások](https://azure.microsoft.com/documentation/articles/azure-security-network-security-best-practices/)

### <a name="database-services"></a>Adatbázis–szolgáltatások

#### <a name="rds-and-azure-relational-database-services"></a>RDS és Azure relációsadatbázis-szolgáltatások

Az Azure több különböző relációsadatbázis-szolgáltatást kínál, amelyek az AWS Relational Database Service (RDS) megfelelői.

-   [SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview)
-   [Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/overview)
-   [Azure Database for PostgreSQL](https://docs.microsoft.com/azure/postgresql/overview)

Egyéb adatbázismotorok, például az [SQL Server](https://azure.microsoft.com/services/virtual-machines/sql-server/), az [Oracle](https://azure.microsoft.com/campaigns/oracle/) és a [MySQL](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-classic-mysql-2008r2/), Azure-beli virtuálisgép-példányokkal helyezhetők üzembe.

Az AWS RDS költségeit a példány által használt hardveres erőforrások (például a CPU, RAM, tárhely és hálózati sávszélesség) mennyisége határozza meg. Az Azure adatbázis-szolgáltatásokban a költségek az adatbázis méretétől, az egyidejű kapcsolatok számától és az átviteli sebességektől függnek.

#### <a name="see-also"></a>Lásd még

-   [Azure SQL Database-oktatóanyagok](https://azure.microsoft.com/documentation/articles/sql-database-explore-tutorials/)

-   [Georeplikáció konfigurálása az Azure SQL Database-adatbázishoz az Azure Portalon](https://azure.microsoft.com/documentation/articles/sql-database-geo-replication-portal/)

-   [A Cosmos DB, egy NoSQL-alapú JSON-adatbázis bemutatása](/azure/cosmos-db/sql-api-introduction)

-   [Az Azure Table Storage használata Node.js-sel](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-table-storage/)

### <a name="security-and-identity"></a>Biztonság és identitás

#### <a name="directory-service-and-azure-active-directory"></a>Címtárszolgáltatás és Azure Active Directory

Az Azure a következő ajánlatokra osztja a címtárszolgáltatásokat:

-   Az [Azure Active Directory](https://azure.microsoft.com/documentation/articles/active-directory-whatis/) egy felhőalapú címtár- és identitáskezelési szolgáltatás.

-   Az [Azure Active Directory B2B](https://azure.microsoft.com/documentation/articles/active-directory-b2b-collaboration-overview/) lehetővé teszi a vállalati alkalmazások elérését a partner által kezelt identitásokról.

-   Az [Azure Active Directory B2C](https://azure.microsoft.com/documentation/articles/active-directory-b2c-overview/) egy egyszeri bejelentkezésre és az ügyfelek által használt alkalmazások felhasználókezelésére szolgáló szolgáltatásajánlat.

-   Az [Azure Active Directory Domain Services](https://azure.microsoft.com/documentation/articles/active-directory-ds-overview/) egy üzemeltetett tartományvezérlő szolgáltatás, amely lehetővé teszi az Active Directoryval kompatibilis tartományokhoz való csatlakozást és a felhasználókezelési funkciókat.

#### <a name="web-application-firewall"></a>Web application firewall (Webalkalmazási tűzfal)

Az [Application Gateway webalkalmazási tűzfal](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/) mellett külső felek, például a [Barracuda Networks](https://azure.microsoft.com/marketplace/partners/barracudanetworks/waf/) [webalkalmazási tűzfalát](https://azure.microsoft.com/documentation/articles/application-gateway-webapplicationfirewall-overview/) is használhatja.

#### <a name="see-also"></a>Lásd még

-   [Ismerkedés a Microsoft Azure biztonsági szolgáltatásaival](https://azure.microsoft.com/documentation/articles/azure-security-getting-started/)

-   [Az Azure-beli identitáskezelés és hozzáférés-vezérlés ajánlott biztonsági eljárásai](https://azure.microsoft.com/documentation/articles/azure-security-identity-management-best-practices/)

### <a name="application-and-messaging-services"></a>Alkalmazás- és üzenetküldési szolgáltatások

#### <a name="simple-email-service"></a>Egyszerű levelezési szolgáltatás

Az AWS egyszerű levelezési szolgáltatást (SES) nyújt az értesítési, tranzakciókkal kapcsolatos vagy marketingcélú e-mailek küldéséhez. Az Azure-ban külső megoldások (például a [Sendgrid](https://sendgrid.com/partners/azure/)) nyújtanak levelezési szolgáltatásokat.

#### <a name="simple-queueing-service"></a>Egyszerű üzenetsor-szolgáltatás

Az AWS egyszerű üzenetsor-szolgáltatása (SQS) egy üzenetkezelési rendszert nyújt az alkalmazások, szolgáltatások és eszközök az AWS platformon belüli összekapcsolásához. Az Azure két szolgáltatással rendelkezik, amelyek hasonló funkciókat nyújtanak:

-   A [Queue Storage](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/) egy felhőalapú üzenetkezelő szolgáltatás, amely lehetővé teszi az Azure platformon lévő alkalmazás-összetevők közötti kommunikációt.

-   A [Service Bus](https://azure.microsoft.com/services/service-bus/) egy robusztusabb üzenetküldő rendszer az alkalmazások, szolgáltatások és eszközök összekapcsolásához. A kapcsolódó [Service Bus Relay](https://docs.microsoft.com/azure/service-bus-relay/relay-what-is-it) használatával a Service Bus távolról futtatott alkalmazásokhoz és szolgáltatásokhoz is kapcsolódhat.

#### <a name="device-farm"></a>Device Farm

A Device Farm eszközök közötti tesztelési szolgáltatásokat nyújt. Az Azure-ban a [Xamarin Test Cloud](https://www.xamarin.com/test-cloud) nyújt hasonló, eszközök közötti előtértesztelést a mobileszközökhöz.

Az előtértesztelés mellett az [Azure DevTest Labs](https://azure.microsoft.com/services/devtest-lab/) háttértesztelési erőforrásokat nyújt a linuxos és windowsos környezetekhez.

#### <a name="see-also"></a>Lásd még

-   [How to use Queue storage from Node.js (A Queue Storage használata Node.js-sel)](https://azure.microsoft.com/documentation/articles/storage-nodejs-how-to-use-queues/)

-   [How to use Service Bus Queues](https://azure.microsoft.com/documentation/articles/service-bus-nodejs-how-to-use-queues/) (A Service Bus-üzenetsorok használata)

### <a name="analytics-and-big-data"></a>Elemzés és big data

[A Cortana Intelligence csomag](https://azure.microsoft.com/suites/cortana-intelligence-suite/) olyan Azure-termékek és -szolgáltatásokból áll, amelyek sok adat rögzítésére, rendszerezésére, elemzésére és megjelenítésére szolgálnak. A Cortana csomag a következő szolgáltatásokból áll:

-   A [HDInsight](https://azure.microsoft.com/documentation/services/hdinsight/) egy felügyelt Apache-disztribúció, amelybe beletartozik a Hadoop, a Spark, a Storm vagy a HBase.

-   A [Data Factory](https://azure.microsoft.com/documentation/services/data-factory/) adat-előkészítést és adatfolyamat-funkciókat nyújt.

-   Az [SQL Data Warehouse](https://azure.microsoft.com/documentation/services/sql-data-warehouse/) egy nagy méretű relációs adattároló.

-   A [Data Lake Store](https://azure.microsoft.com/documentation/services/data-lake-store/) egy nagy méretű tároló, amely a big data koncepción alapuló adatelemzési számítási feladatokra szolgál.

-   A [Machine Learning](https://azure.microsoft.com/documentation/services/machine-learning/) prediktív elemzések felépítésére és adatokon való alkalmazására szolgál.

-   A [Stream Analytics](https://azure.microsoft.com/documentation/services/stream-analytics/) valós idejű adatelemzést tesz lehetővé.

-   A [Data Lake Analytics](https://azure.microsoft.com/documentation/articles/data-lake-analytics-overview/) egy nagy méretű elemzési szolgáltatás, amely a Data Lake Store-ral való használathoz lett optimalizálva.

-   A [PowerBI](https://powerbi.microsoft.com/) az adatok megjelenítésére szolgál.

#### <a name="see-also"></a>Lásd még

-   [Cortana Intelligence Gallery](https://gallery.cortanaintelligence.com/)

-   [A Microsoft big data-megoldásainak ismertetése](https://msdn.microsoft.com/library/dn749804.aspx)

-   [Azure Data Lake és Azure HDInsight blog](https://blogs.msdn.microsoft.com/azuredatalake/)

### <a name="internet-of-things"></a>Eszközök internetes hálózata

#### <a name="see-also"></a>Lásd még

-   [Ismerkedés az Azure IoT Hub szolgáltatással](https://azure.microsoft.com/documentation/articles/iot-hub-csharp-csharp-getstarted/)

-   [Az IoT Hub és az Event Hubs összehasonlítása](https://azure.microsoft.com/documentation/articles/iot-hub-compare-event-hubs/)

### <a name="mobile-services"></a>Mobilszolgáltatások

#### <a name="notifications"></a>Értesítések

A Notification Hubs nem támogatja az SMS-ek vagy e-mail-üzenetek küldését, ezért külső szolgáltatásokra van szükség az ilyen típusú üzenetküldéshez.

#### <a name="see-also"></a>Lásd még

-   [Android-alkalmazás létrehozása](https://azure.microsoft.com/documentation/articles/app-service-mobile-android-get-started/)

-   [Hitelesítés és engedélyezés az Azure Mobile Apps megoldásban](https://azure.microsoft.com/documentation/articles/app-service-mobile-auth/)

-   [Leküldéses értesítések küldése az Azure Notification Hubs használatával](https://azure.microsoft.com/documentation/articles/notification-hubs-android-push-notification-google-fcm-get-started/)

### <a name="management-and-monitoring"></a>Kezelés és monitorozás

#### <a name="see-also"></a>Lásd még
-   [Megfigyelési és diagnosztikai útmutató](https://azure.microsoft.com/documentation/articles/best-practices-monitoring/)

-   [Az Azure Resource Manager-sablonok létrehozásának ajánlott eljárásai](https://azure.microsoft.com/documentation/articles/resource-manager-template-best-practices/)

-   [Az Azure Resource Manager gyorsindítási sablonjai](https://azure.microsoft.com/documentation/templates/)


## <a name="next-steps"></a>További lépések

-   [Az AWS és az Azure szolgáltatás-összehasonlító mátrixa](https://aka.ms/azure4aws-services)

-   [Az Azure platform interaktív nagyméretű áttekintő képe](http://azureplatform.azurewebsites.net/)

-   [Bevezetés az Azure használatába](https://azure.microsoft.com/get-started/)

-   [Azure-megoldások architektúrái](https://azure.microsoft.com/solutions/architecture/)

-   [Azure-referenciaarchitektúrák](https://azure.microsoft.com/documentation/articles/guidance-architecture/)

-   [Minták és gyakorlatok: Azure-útmutató](https://azure.microsoft.com/documentation/articles/guidance/)

-   [Ingyenes online tanfolyam: Microsoft Azure AWS-szakértőknek](http://aka.ms/azureforaws)


<!-- links -->

[paired-regions]: https://azure.microsoft.com/documentation/articles/best-practices-availability-paired-regions/
[traffic-manager]: /azure/traffic-manager/