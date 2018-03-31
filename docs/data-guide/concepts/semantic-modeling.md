---
title: "Szemantikai modellezés"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 343d17af0d933d515c724a062237c8d5df3a9e31
ms.sourcegitcommit: 29fbcb1eec44802d2c01b6d3bcf7d7bd0bae65fc
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/27/2018
---
# <a name="semantic-modeling"></a>Szemantikai modellezés

A szemantikai adatmodell a fogalmi modell, amely leírja a benne található adatelemek szerinti. Szervezetek sokszor a saját feltételei dolgot, egyes esetekben a szinonimák, vagy akár különböző jelentésekkel azonos kifejezés. Például egy készlet adatbázis előfordulhat, hogy nyomon berendezés, eszköz-Azonosítóval és a sorszámot, de az értékesítési adatbázis utalhat a sorozatszám eszköz azonosítóként Nincs egyszerű mód más ezeket az értékeket egy modell, amely leírja a kapcsolat nélkül. 

Szemantikai modellezési, hogy a felhasználók nem szükséges tudni, hogy az alapul szolgáló adatstruktúrák biztosítanak az adatbázisséma egy Elvonási szint. Így könnyebben a végfelhasználók számára az adatok lekérdezése nélkül aggregátumok és illesztések az alapul szolgáló séma keresztül hajtja végre. Emellett általában oszlopok váltják fel felhasználóbarát nevek, és az adatok jelentését viszonylag kézenfekvő legyenek.

Szemantikai modellezési olvasási-nehéz helyzetekben, például elemzés és üzleti intelligencia (OLAP) használt döntő többsége figyelésekor további írási műveleteket tranzakciós adatok feldolgozási (OLTP). Egy tipikus szemantikai réteg jellege miatt legtöbbször nem:

- Összesítési viselkedéshez, hogy a jelentéskészítési eszközök jelennek meg azok megfelelően vannak beállítva.
- Üzleti logika és számítások határozza meg.
- Idő célú számítások tartoznak.
- Adatok gyakran integrálva van a különböző forrásokból származó. 

Hagyományosan a szemantikai réteg felett van elhelyezve egy adatraktár ezen okok miatt.

![A szemantikai réteg között adatraktárt és jelentéskészítési eszköz, például ábrája](./images/semantic-modeling.png)

A szemantikai modellek két elsődleges típusa van:

* **A táblázatos**. Relációs modellezési szerkezetek (modell, táblázatok, oszlopok) használja. Belsőleg metaadatok (kockák, dimenziókat, mértékeket) szerkezetek modellezési OLAP öröklődik. Kód és parancsfájl használata az OLAP-metaadatok.
* **Többdimenziós**. Hagyományos OLAP modellezési szerkezetek (kockák, dimenziókat, mértékeket) használja.

Megfelelő Azure szolgáltatás:
- [Az Azure Analysis Services](https://azure.microsoft.com/services/analysis-services/)

## <a name="example-use-case"></a>Példa használati eset

Egy szervezet nagy adatbázisban tárolt adatok rendelkezik. Azt szeretné elérhetővé tenni az adatok üzleti felhasználók és a felhasználók a saját jelentések létrehozásához, és néhány elemzést. Egy elem csak a felhasználók közvetlen hozzáférésének az adatbázisban. Van azonban több hátrányai ezzel, beleértve a biztonság kezelése és hozzáférés szabályozása. Az adatbázis, a táblák, illetve az oszlopok, beleértve a Tervező is, egy felhasználó megértése nehéz lehet. Felhasználók kellene, hogy mely táblák lekérdezéséhez, hogyan kell csatlakoztatni azokat a táblákat és más üzleti logika, amely a megfelelő eredményt kell alkalmazni. Felhasználók például még a kezdéshez SQL lekérdezésnyelvet tudnia kell. Általában ez vezet, jelentéskészítő elérhető metrikák több felhasználó, de eltérő eredményt.

Egy másik lehetőség egy foglalják magukban a felhasználók a szemantikai modell szükséges információk. A szemantikai modell könnyebben lehet lekérdezni a felhasználók az általuk választott jelentéskészítési eszközzel. A szemantikai modell által szolgáltatott adatokat a rendszer egy adatraktárból, győződjön meg arról, hogy minden felhasználó tekintse meg a valóságnak egyetlen verziója hívja elő. A szemantikai modell emellett rövid tábla, az oszlopnevek, táblák, a leírásokat, a számítások és a sorszintű biztonság közötti kapcsolatokat.

## <a name="typical-traits-of-semantic-modeling"></a>A szemantikai modellezési tipikus jellemzők

A szemantikai modellezési és elemzésfeldolgozási általában a következő jellemzőkkel rendelkezik:

| Követelmény | Leírás |
| --- | --- |
| Séma | Írás, erősen kényszerített séma|
| Használja a tranzakciók | Nem |
| Zárolási stratégia | Nincs |
| Frissíthető | Nem (általában szükséges adatkocka recomputing) |
| Appendable | Nem (általában szükséges adatkocka recomputing) |
| Számítási feladat | Nagy mennyiségű olvasási, csak olvasható |
| Indexelés | Többdimenziós indexelő |
| Datum mérete | Kis, közepes méretű |
| Modell | Multidimensional |
| Adatok alakzat:| Adatkocka vagy csillag/snowflake séma |
| Lekérdezés rugalmasságot | Rugalmas |
| Skála: | Nagy (10 egység-100-as egység GB) |

## <a name="see-also"></a>Lásd még

- [Adatraktározás](../scenarios/data-warehousing.md)
- [Online analitikus feldolgozási (OLAP)](../scenarios/online-analytical-processing.md)
