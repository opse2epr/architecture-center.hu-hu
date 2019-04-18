---
title: A rugalmasság és a rendelkezésre állás az Azure-alkalmazások üzembe helyezése
description: Megbízható üzemelő példányok létrehozásához az Azure-ban módszerek áttekintése
author: MikeWasson
ms.date: 04/10/2019
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-design-principles
ms.openlocfilehash: d8df3fe1c3353c60462959a625ba13ae47b96cf8
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59646555"
---
# <a name="deploying-azure-applications-for-resiliency-and-availability"></a>A rugalmasság és a rendelkezésre állás az Azure-alkalmazások üzembe helyezése

Kiépítése, és az Azure-erőforrások, az alkalmazáskód és a konfigurációs beállítások frissítése, megismételhető és kiszámítható folyamat segítséget nyújt a hibák és állásidő elkerülése érdekében. Azt javasoljuk, hogy a központi telepítésre, amely igény szerint futtathat és Újrafuttatás, ha hiba lép fel automatizált folyamatok. Ha az üzembe helyezési folyamatok zökkenőmentesen futnak, folyamat dokumentációja tárolhatja őket így.

## <a name="automate-processes"></a>Folyamatok automatizálása

Aktiválja az erőforrások igény szerinti, megoldások gyors üzembe helyezését, emberi hibák minimalizálása és egységes és megismételhető eredményeket, ügyeljen arra, központi telepítések és frissítések automatizálása.

### <a name="automate-as-many-processes-as-possible"></a>A lehető legtöbb folyamatok automatizálása

A legmegbízhatóbb üzembe helyezési folyamatok és *idempotens* &mdash; , megismételhető, hogy ugyanazt az eredményt.

- Automatizálhatja az Azure-erőforrások kiépítését, használhatja a [Terraform](/azure/virtual-machines/windows/infrastructure-automation#terraform), [Ansible](/azure/virtual-machines/windows/infrastructure-automation#ansible), [Chef](/azure/virtual-machines/windows/infrastructure-automation#chef), [Puppet](/azure/virtual-machines/windows/infrastructure-automation#puppet), [Azure PowerShell](/powershell/azure/overview), [az Azure parancssori felület (CLI)](/cli/azure), vagy [Azure Resource Manager-sablonok](/azure/azure-resource-manager/resource-group-overview#template-deployment).
- A virtuális gépek konfigurálásához használható [a cloud-init](/azure/virtual-machines/windows/infrastructure-automation#cloud-init) (Linux virtuális gépek esetén) vagy [Azure Automation Állapotkonfiguráció](/azure/automation/automation-dsc-overview) (DSC).
- Alkalmazás üzembe helyezésének automatizálása, használható [Azure DevOps-szolgáltatásokkal](/azure/virtual-machines/windows/infrastructure-automation#azure-devops-services), [Jenkins](/azure/virtual-machines/windows/infrastructure-automation#jenkins), vagy más CI/CD-megoldás.

Ajánlott eljárásként hozzon létre egy adattár a gyors hozzáférés érdekében magyarázatát paramétereket és a példa a parancsfájl a dokumentált kategorizált automatizálási szkriptek. Ebben a dokumentációban az Azure-környezetek szinkronban tartani, és kezelheti a tárház elsődleges személy kijelölése.

Automatizálási szkriptek is aktiválhatja a vész-helyreállítási igény szerinti erőforrások.

### <a name="use-code-to-provision-and-configure-infrastructure"></a>Kód üzembe helyezni, és az infrastruktúra konfigurálása

Ez a gyakorlat nevű *infrastruktúra mint kód,* deklaratív vagy imperatív módszerrel (vagy a kettő kombinációjával) használhatja.

- [Resource Manager-sablonok](/azure/azure-resource-manager/resource-group-overview#template-deployment) deklaratív módszer egy példát.
- [PowerShell](/powershell/azure/overview) parancsfájlok használata az imperatív módszer egy példája.

### <a name="practice-immutable-infrastructure"></a>Eljárás nem módosítható infrastruktúra

Más szóval ne módosítsa az infrastruktúra éles üzembe helyezése után. Alkalmi változtatások alkalmazása után, hogy nem tudja pontosan mi változott, így a rendszer a hibaelhárítás nehezen lehet.

### <a name="automate-and-test-deployment-and-maintenance-tasks"></a>Automatizálhatja és központi telepítési és karbantartási feladatok tesztelése

Az elosztott alkalmazások, amelyek együtt kell működniük a több részből állnak. Üzembe helyezés bevált eljárások, például a parancsfájlokat, frissítése és a konfiguráció ellenőrzése és az üzembe helyezési folyamat automatizálható kell előnyeit. Tesztelje az összes folyamat teljes győződjön meg arról, hogy a hibák nem okoznak további állásidőt.

### <a name="implement-deployment-security-measures"></a>Üzembe helyezés biztonsági intézkedések végrehajtása

Az összes központi telepítési eszközök tartalmaznia kell az üzembe helyezett alkalmazás védelme érdekében biztonsági korlátozások. Határozza meg gondosan az üzembe helyezési szabályzatokat léptethet, és az emberi beavatkozások szükségességének minimálisra csökkentése.

## <a name="deploy-to-multiple-regions-and-instances"></a>Üzembe helyezés több régióban és példányok

Olyan alkalmazás, amely egy szolgáltatás egyetlen példánya függ egy meghibásodási pontot hoz létre. Rugalmasság és méretezhetőség javítása érdekében építhet ki több példányt.

- A [Azure App Service](/azure/app-service/app-service-value-prop-what-is/), jelölje be egy [App Service-csomag](/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview/) , amelynek keretében több példányt.
- A [Azure Cloud Services](/azure/cloud-services/cloud-services-choose-me), konfigurálható, a szerepkörök használatára [több példány](/azure/cloud-services/cloud-services-choose-me/#scaling-and-management).
- A [Azure Virtual Machines](/azure/virtual-machines/virtual-machines-windows-about/?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json), győződjön meg arról, hogy az architektúra tartalmaz-e egynél több virtuális gép és minden virtuális gép szerepel egy [rendelkezésre állási csoport](/azure/virtual-machines/virtual-machines-windows-manage-availability/).

### <a name="consider-deploying-across-multiple-regions"></a>Érdemes megfontolni a több régióban

Javasoljuk, hogy az összes, de legalább kritikus fontosságú alkalmazások és alkalmazásszolgáltatások több régióban. Ha az alkalmazás egyetlen régióban üzembe, az esemény ritkán fordul elő, hogy a teljes régió nem érhető el, az alkalmazás is nem érhető el.

Ha egyetlen régióban történő üzembe helyezéséhez, érdemes lehet egy másodlagos régióba váratlan hibájára való válaszként ismételt üzembe helyezés előkészítése.

### <a name="redeploy-to-a-secondary-region"></a>Ismételt üzembe helyezése egy másodlagos régióba

Alkalmazások és adatbázisok futtatja a replikáció nem egy önálló, elsődleges régióban, ha a stratégia lehet egy másik régióban üzembe helyezése. Ez a megoldás, megfizethető, de leginkább megfelelő, nem kritikus fontosságú alkalmazásokhoz, amely tűri hosszabb helyreállítási időt.

Ha úgy dönt, ezt a stratégiát, automatizálhatja a lehető legnagyobb mértékben újbóli üzembe helyezés folyamatát, és újbóli üzembe helyezés forgatókönyvek bevonni a vészhelyreállítási válasz tesztelése.

Automatizálhatja a újbóli üzembe helyezés során, érdemes lehet [Azure Site Recovery](/azure/site-recovery/).

## <a name="use-staging-and-production-environments"></a>Átmeneti és éles környezetek használata

Hasznosnak találta az átmeneti és éles környezetekben, a frissítések leküldése az éles környezetben erősen szabályozott módon, és a nem várt üzembe helyezési problémák minimalizálása érdekében.

- [*Kék-zöld üzembe helyezés*](https://martinfowler.com/bliki/BlueGreenDeployment.html) magában foglalja a frissítés telepítése az élő alkalmazástól elkülönített éles környezetben. Az üzemelő példány ellenőrzése után irányítsa át a forgalmat a frissített verzióhoz. Ehhez a egyirányú azt használja a [előkészítési ponton](/azure/app-service/web-sites-staged-publishing) éles áthelyezés előtt üzembe helyezésének ütemezése az Azure App Service-ben érhető el.
- [*Tesztcsoportos kiadások*](https://martinfowler.com/bliki/CanaryRelease.html) hasonlóak a kék-zöld üzembe. Helyett vált minden forgalmat a frissített alkalmazást, a forgalom csak egy kis részét a új központi telepítési útvonal. Ha probléma merül fel, visszavált a régi üzemelő példányra. Ellenkező esetben fokozatosan további forgalom irányítása az új verzióra. Azure App Service használata, használhatja a tesztelés az éles funkció canary kiadás kezeléséhez.

## <a name="create-a-rollback-plan-for-deployment-and-updates"></a>Központi telepítési és frissítési visszaállítási terv létrehozása

Ha nem sikerül a telepítés, az alkalmazás válhat elérhetetlenné. Hogy minimalizálják az állásidőt, tervezze meg a visszaállítási folyamat egy utolsó ismert jó verzióra való visszatéréshez. Például egy stratégia visszafordítani a változásokat, adatbázisok és az alkalmazása függ más szolgáltatásokat.

Használja az Azure App Service-ben, ha egy utolsó ismert jó tárhely beállítása, és ezzel állítja vissza egy webalkalmazás vagy API-alkalmazás üzembe helyezése.

## <a name="log-and-audit-deployments"></a>Napló- és vizsgálati központi telepítések

A lehető sokkal verzióspecifikus információk rögzítéséhez egy hatékony naplózás stratégia megvalósításához. A kétlépcsős üzembe helyezési módszereket használja, ha egynél több verziója, az alkalmazás éles környezetben fog futni. Ha probléma merül fel, határozza meg, melyik okozza.

## <a name="document-release-processes"></a>A dokumentum-kiadási folyamatok

Nélkül részletes kibocsátási folyamat dokumentációja az operátornak egy rossz frissítés központi telepítését, vagy előfordulhat, hogy helytelen az alkalmazás beállításainak konfigurálása. Egyértelműen határozza meg és dokumentálja a kibocsátási folyamat, és győződjön meg arról, hogy azt a teljes üzemeltetési csapat rendelkezésére.

## <a name="next-steps"></a>További lépések

> [!div class="nextstepaction"]
> [A megbízhatóság alkalmazás állapot figyelése](./monitoring.md)
