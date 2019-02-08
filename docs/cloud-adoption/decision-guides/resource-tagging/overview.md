---
title: 'CAF: Erőforrás-szervezetben és címkézési döntési útmutató'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Ismerje meg erőforrás-szervezetben és a címkézés Azure áttelepítések core szolgáltatás.
author: rotycenh
ms.openlocfilehash: 0da9f698fc7ac2db876409ad451adf7b12cf6e6b
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899106"
---
# <a name="resource-organization-and-tagging-decision-guide"></a>Erőforrás-szervezetben és címkézési döntési útmutató

Felhőalapú erőforrások rendszerezéséhez az egyik legfontosabb feladatai informatikai, kivéve, ha nagyon egyszerű üzembe. Az erőforrások rendszerezéséhez szolgálja ki a három elsődleges célja:

- **Az erőforrás-kezelés**. Az IT-csoportoknak kell gyorsan megtalálhatja az adott számítási feladatokhoz, környezetek, tulajdonosi csoportok vagy egyéb fontos információk társított erőforrások. Erőforrások rendszerezése fontos szervezeti szerepköröket és a hozzáférési engedélyeket az erőforrás-kezelés hozzárendelését.
- **Műveletek**. Amellett erőforrások számára könnyebb informatikai részleg felügyelje, a megfelelő szervezeti séma lehetővé teszi, hogy kihasználni az automatizálás az erőforrás-létrehozás, a műveletek figyelése és a fejlesztési és üzemeltetési folyamatok létrehozásának részeként.
- **Nyilvántartás**. Így üzleti csoportok felhőalapú erőforrás-használat tisztában van szükség, hogy megérteni, milyen számítási feladatokat és a csapatok mely erőforrásokat használják. Támogatja a módszerekkel, például a jóváírás és költséghelyi elszámolási, a felhőbeli erőforrások kell tulajdonosi és a használat lehetséges felépítésére.

## <a name="tagging-decision-guide"></a>Címkézési döntési útmutató

![Ábrázolási beállítások és a legkevésbé címkézése a legösszetettebb, az alábbi hivatkozások a jump igazítva](../../_images/discovery-guides/discovery-guide-tagging.png)

Ugrás ide: [Alapkonfiguráció elnevezési konvenciók](#baseline-naming-conventions) | [minták címkézés erőforrás](#resource-tagging-patterns) | [elnevezésével és címkézés házirend](#naming-and-tagging-policy) | [további](#learn-more)

A címkézési módszer lehet egyszerű vagy összetett, és a támogató felhőalapú számítási feladatokhoz, integrálható a teljes üzleti minden aspektusát vonatkozó információk kezelése informatikai csapata hozza létre a hangsúlyt.

Egy informatikai igazított címkézési fókusz figyelési eszközök összetettségének csökkentésére, és a funkciókat és sokkal könnyebben besorolás alapján felügyeleti döntéseket.

Címkézés nem informatikai házirendek is tartalmazó sémák egy nagyobb időbefektetést követelne címkézési olyan szabványok gyűjteménye, amelyek üzleti érdekében, és a karbantartásához szabványokat létrehozására lehet szükség. Azonban ez a folyamat eredménye egy címkézési system, ami egy továbbfejlesztett lehetőség a fiókra a költségek és az IT-eszközeivel értékét. Ezt a társítást egy eszköz érték, a üzemeltetési költségek egyike a költségek center érzete módosítása első lépése a szervezet számítástechnikai környezetét.

## <a name="baseline-naming-conventions"></a>Alapkonfiguráció elnevezési konvenciók

Szabványos elnevezési szolgál kiindulópontként a a felhőben futó erőforrások rendszerezéséhez. A megfelelően felépített elnevezési rendszer lehetővé teszi, hogy gyorsan azonosíthatja a felügyeleti és számlázási célból erőforrásokat. Ha meglévő informatikai elnevezési konvenciók a szervezet többi részének, érdemes lehet-e a felhő-erőforrások elnevezési konvenciói velük kell igazítani, vagy ha külön felhőalapú szabványok kell létesítenie.

Vegye figyelembe továbbá, hogy különböző Azure-erőforrástípusra jellemzők különböző [elnevezési követelményeknek](../../../best-practices/naming-conventions.md#naming-rules-and-restrictions). Az elnevezési konvenciók elnevezési követelménynek kompatibilisnek kell lennie.

## <a name="resource-tagging-patterns"></a>Erőforrás-címkézési minták

Csak egy egységes kulcselnevezési konvenció kifinomultabb vagy szervezethez is biztosítanak, felhőalapú platform támogatása lehetővé teszi az erőforrások emellett címkézhetők.

*A címkék* erőforrások csatolt metaadatokat elemet. Címkék karakterláncok kulcs-érték párokból áll. A párok szerepeltetni kívánt értékek Öntől, de globális címkék és a egy átfogó elnevezési és a címkézés, a házirend részeként a konzisztens alkalmazása egy általános cégirányítási házirend kritikus részét képezi.

Az alábbiakban néhány gyakori címkézési minták példát:

<!-- markdownlint-disable MD033 -->

| Tag típusa | Példák | Leírás |
|-----|-----|-----|
| funkcionális            | alkalmazás = catalogsearch1 <br/>szint = web <br/>webkiszolgáló = apache<br/>env = éles <br/>env = átmeneti <br/>env = dev                 | Saját céljának belül egy munkaterhelés –, az erőforrások kategorizálása milyen környezet már telepítve, vagy egyéb funkciók és a műveleti adatokat.                                 |
| Besorolás        | confidentiality=private<br/>SLA = 24hours                                 | Hogyan használja fel azokat, és milyen házirendek vonatkoznak, erőforrás osztályozza                               |
| Könyvelés            | részleg = pénzügyi <br/>Projekt = catalogsearch <br/>régió = Észak-Amerika | Lehetővé teszi, hogy a számlázási szervezetben adott csoportokhoz hozzárendelni kívánt erőforrás |
| Partneri kapcsolat           | tulajdonos = jsmith <br/>contactalias = catsearchowners<br/>érintettek = felhasználó1, felhasználó2; Felhasználó3<br/>                       | Ismerteti, milyen személyek (alkalmazáson kívül informatikai) kapcsolatos, vagy ellenkező esetben az erőforrás által érintett                      |
| Cél               | businessprocess támogatási =<br/>businessimpact=moderate<br/>revenueimpact=high   | Üzleti funkciók, amelyek hatékonyabban támogatják a befektetési döntések erőforrások igazítása  |

<!-- markdownlint-enable MD033 -->

## <a name="naming-and-tagging-policy"></a>Kiosztási és címkézés házirend

A kiosztási és címkézési szabályzat idővel fejlődik. Azonban a felhőbe történő migrálás kezdettől fogva core szervezeti prioritások meghatározásakor, kritikus fontosságú. A tervezési folyamat részeként a alaposan gondolja át az alábbi kérdésekre:

- Hogyan érdemes az elnevezési és címkézési házirendek integrálható a meglévő elnevezési és a szervezeti szabályzatok a szervezeten belül?
- Végrehajtja a visszatérítés vagy visszajelzés könyvelési rendszer? Hogyan történik a részlegek, üzleti csoportokat vagy a szervezeti struktúrában jelölt csapatok?
- Címkézési információkról igazolni szükséges összes erőforrást? Címkézési információkról megvalósításához, vagy nem implementálja az egyes csapatok legfeljebb marad?
- Nem, amelyek részleteket címkézési kell egy erőforrás jogszabályi követelményeknek? Mi a helyzet a műveleti adatokat, például az üzemidővel kapcsolatos elvárásainak, javítási ütemezések, vagy a biztonsági követelményeknek?

## <a name="learn-more"></a>Részletek

Kiosztási és az Azure-ban címkézés kapcsolatos további információkért lásd:

- [Azure-erőforrások elnevezési konvenciói](../../../best-practices/naming-conventions.md). Tekintse meg ez az útmutató az Azure felhő – alapok helyről az Azure-erőforrások ajánlott elnevezési konvenciói.
- [Az Azure-erőforrások rendszerezése címkék használatával](/azure/azure-resource-manager/resource-group-using-tags?toc=/azure/billing/TOC.json). Címkékkel láthatja az Azure-ban az erőforráscsoportot és az egyes erőforrások szintjén, egyaránt mozgásteret biztosít az alkalmazott címkék alapján elszámolási jelentések részletességét.

## <a name="next-steps"></a>További lépések

Ismerje meg, hogyan titkosítási felhőkörnyezetben az adatok védelmére használható.

> [!div class="nextstepaction"]
> [Titkosítás](../encryption/overview.md)
