---
title: 'CAF: Erőforrás konzisztencia döntési útmutató'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Tudnivalók az erőforrás konzisztencia, egy Azure áttelepítés tervezése során.
author: rotycenh
ms.openlocfilehash: 3159e4b7aeddfdd99261c0f68591998d741f3359
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640046"
---
# <a name="caf-resource-consistency-decision-guide"></a>CAF: Erőforrás konzisztencia döntési útmutató

Azure [előfizetések kialakítása](../subscriptions/overview.md) határozza meg, miként célszerű megszerveznie a felhőbeli eszközöket a szervezet struktúra, a számlázási eljárások és a számítási feladatok követelményeit. Ezen a szinten minden olyan mappaszerkezet mellett a szervezeti cégirányítási házirend követelményeinek teljesítése között a felhőbeli hagyatéki szüksége van arra, konzisztens módon rendszerezheti, üzembe helyezése és egy előfizetésen belüli erőforrások kezelése.

![A legkevésbé erőforrás konzisztenciabeállítás küldik az ábrázolást a legösszetettebb, az alábbi hivatkozások a jump igazítva](../../_images/discovery-guides/discovery-guide-resource-consistency.png)

Ugrás ide: [Alapvető csoportosítási](#basic-grouping) | [üzembe helyezési konzisztencia](#deployment-consistency) | [házirend konzisztencia](#policy-consistency) | [hierarchikus konzisztencia](#hierarchical-consistency)  |  [Automatikus konzisztencia](#automated-consistency)

A felhő hagyatéki konzisztencia erőforrásigényei szintjét kapcsolatos döntéseket elsősorban alakítják tényezők: áttelepítés utáni digitális hagyatéki mérete, üzleti vagy a meglévő előfizetésében eligazíthatja nem illő környezeti követelmények tervezési megközelítés, vagy a kell irányítási erőforrások üzembe helyezése után idővel kényszerítése. 

Mivel ezek a tényezők növekedése fontosként, egységes üzembe helyezést, a csoportosítás és a felhőbeli erőforrások kezelését biztosító előnyeit fontosabbá válik. További speciális szintek növekvő igényeknek erőforrás konzisztencia megvalósítása igényel nagyobb munkát, automatizálás, azokat az eszközöket és konzisztencia kényszerítési töltött, és ennek eredményeként egy több időt töltött változáskezelés és nyomon követését.

## <a name="basic-grouping"></a>Alapszintű csoportosítás

Az Azure-ban [erőforráscsoportok](/azure/azure-resource-manager/resource-group-overview#resource-groups) egy alapvető erőforrás adatrendezési mechanizmus logikailag csoportosíthatók az erőforrások egy előfizetésen belül.

Erőforráscsoportok tárolóként szolgálnak egy közös életciklussal rendelkező erőforrások, vagy megosztott felügyeleti korlátozásokat, például a csoportházirend vagy a szerepköralapú hozzáférés-vezérlés (RBAC) követelményeinek. Erőforráscsoportok nem ágyazható be, és az erőforrások csak egy erőforráscsoporthoz is tartozhatnak. Minden erőforrás egy erőforráscsoportba tartozó műveleteket hajthat végre bizonyos műveleteket. Például egy erőforráscsoport törlése eltávolítja az adott csoportban lévő összes erőforrást. Nincsenek közös mintát gyakran két kategóriába oszthatók, erőforráscsoportok létrehozásakor:

- Hagyományos informatikai számítási feladatok: Leggyakrabban használt elemek belül ugyanaz az életciklusuk, ilyen lehet például egy alkalmazás szerint csoportosítva. Alkalmazás által a csoportosítás lehetővé teszi egyéni alkalmazás felügyeletét.
- Agilis IT-munkaterhelést: A külső ügyfelek által használt felhőalkalmazások összpontosítanak. Ezeket az erőforráscsoportokat gyakran jelenik meg a működési rétegeket, (például a webes szint vagy alkalmazás szinten) üzembe helyezését és felügyeletét.

## <a name="deployment-consistency"></a>Üzembe helyezés konzisztencia

Épület csoportosítási mechanizmus az alapvető erőforrás felett, az Azure platform rendszert biztosít a sablonok használatával helyezhet üzembe az erőforrásokat a felhőalapú környezetben. A sablonokkal hozhat létre egységes szervezet és elnevezési konvenciók tevékenységprofilok, amikor az erőforrás üzembe helyezési és kezelési kialakítási szempontjaival kényszerítése.

[Az Azure Resource Manager-sablonok](/azure/azure-resource-manager/resource-group-overview#template-deployment) többször is telepíthet az erőforrások egy előre meghatározott konfigurációs és erőforrásfájlról csoportszerkezetet használatával konzisztens lesz. Resource Manager-sablonok segítségével meghatározhatja a vonatkozó szabványok a központi telepítések alapjaként.

Rendelkezhet például egy standard sablon, amely tartalmazza a két virtuális gépet, a kiszolgálók közötti forgalom elosztására terheléselosztó kombinálva webkiszolgálók web server számítási feladatok üzembe helyezéséhez. Ezután felhasználhatja a sablon szerkezete azonos virtuális gépek létrehozása és a terheléselosztó, amikor szükség van az ilyen típusú számítási feladatok, a központi telepítés nevét és IP-címek csak módosítása vesz részt.

Vegye figyelembe, hogy programozott módon is üzembe helyezheti ezeket a sablonokat, majd a CI/CD-rendszerekkel való integrációt segítik őket.

## <a name="policy-consistency"></a>A házirend konzisztencia

Győződjön meg arról, hogy a cégirányítási szabályzatok érvényesek-e, ha az erőforrások jönnek létre, az erőforrás csoportosítási kialakításának magában foglalja a egyik gyakran alkalmazott konfiguráció használatával az erőforrások üzembe helyezésekor.

Erőforráscsoportok és a Resource Manager-sablonok szabványosított kombinálásával kényszerítheti a szabványok milyen beállításokat szükséges központi telepítés és milyen [Azure Policy](/azure/governance/policy/overview) szabályok érvényesek minden egyes erőforráscsoportra vagy erőforrásra.

Például előfordulhat, hogy rendelkezik egy követelményt, amely a központi informatikai csoport által felügyelt közös alhálózathoz csatlakozzon az előfizetésben telepített összes virtuális gép. Létrehozhat egy standard sablon, amely ehhez hozzon létre egy külön erőforráscsoportot a számítási feladathoz tartozó, és nincs szükség virtuális gépek üzembe helyezése számítási feladatok virtuális gépek üzembe helyezéséhez. Ez az erőforráscsoport házirend szabály a kizárólag a hálózati adapterek, amelyet egyesíteni kell a megosztott alhálózathoz az erőforráscsoporton belül kell.

Részletesebb érvényesítésének a házirendet érintő döntések belül üzembe helyezése a felhőben, lásd: [házirend betartatása](../policy-enforcement/overview.md).

## <a name="hierarchical-consistency"></a>Hierarchikus konzisztencia

Erőforráscsoportok lehetővé teszi, hogy támogatja az Azure Policy szabályok alkalmazása a szervezetben az előfizetésen belüli hierarchia szintű és hozzáférés-vezérlés egy erőforráscsoport szintjén. Azonban a felhő hagyatéki méretének növekedésével szükség lehet, mint az Azure nagyvállalati szerződéssel vállalati/szervezeti egység/fiók vagy előfizetés hierarchia használatával támogatható bonyolultabb előfizetések közötti cégirányítási követelmények támogatására. 

[Az Azure felügyeleti csoportok](../subscriptions/overview.md#management-groups) lehetővé teszi, hogy a szervezet előfizetés kifinomultabb szervezeti felépítés struktúra a nagyvállalati szerződés által létrehozott egy másik hierarchiában csoportosítási előfizetések által. Ez a hierarchia másik lehetővé teszi, hogy több előfizetést és az erőforrások hozzáférés-vezérléshez és a házirend kényszerítő mechanizmust alkalmazhat. Felügyeleti csoport-hierarchiái a megfelelő műveleteket és üzleti cégirányítási követelmények a felhő hagyatéki előfizetésekre használható. 

## <a name="automated-consistency"></a>Az automatikus konzisztencia

Nagyméretű felhőbeli üzembe globális irányítás fontosabb, és a bonyolultabb lesz. Elengedhetetlen, az automatikusan alkalmazza és cégirányítási követelmények kényszerítése az erőforrások üzembe helyezésekor, valamint a meglévő telepítések frissített követelményeinek.

[Az Azure tervezetek](/azure/governance/blueprints/overview) szervezetek támogatja a nagy méretű felhőalapú ingatlanadót globális irányítása az Azure-ban. Tervezetek helyezze át a teljes üzembe helyezés vezénylések erőforrások telepítését és a szabályok alkalmazása képes létrehozni szabványos Azure Resource Manager-sablonok által biztosított képességek mellett. Tervezetek versioning, képesség a alkalmazni a frissítéseket az összes előfizetés ahol a tervezet használta, valamint az üzembe helyezett előfizetések jogosulatlan létrehozásának és módosításának erőforrások elkerülése érdekében zárolhat támogatja.

A központi telepítési csomagok lehetővé teszik informatikai és fejlesztői csapatok gyorsan üzembe helyezheti az új számítási feladatok és a hálózati eszközök, amelyek megfelelnek a szervezeti házirend követelményeinek megváltoztatása. Tervezetek CI/CD-folyamatok átdolgozott szabályozási előírások alkalmazandó központi telepítések, frissítését is integrálható.

## <a name="next-steps"></a>További lépések

Ismerje meg, hogyan erőforrás-kiosztási és címkézése további rendszerezésére és a felhőbeli erőforrások kezelését használhatók.

> [!div class="nextstepaction"]
> [Erőforrás-kiosztási és címkézése](../resource-tagging/overview.md)
