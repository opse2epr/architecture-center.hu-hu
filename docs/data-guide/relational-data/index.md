---
title: Relációs adatok
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: fa2fef27f47acadf00cbfc821c7432c07a3947be
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/03/2018
---
# <a name="traditional-relational-database-solutions"></a>Hagyományos relációsadatbázis-megoldások

Relációs adatnak a relációs modell használatával modellezett adatokat nevezzük. Ebben a modellben az adatok rekordként vannak kifejezve. A *rekord* attribútum-érték párok készletét jelenti. Egy rekord például lehet a következő: (itemid = 5, orderid = 1, item = "Chair", amount = 200.00). Az ugyanazon attribútumokkal rendelkező rekordokat nevezzük *relációnak*. 

A relációk természetes megjelenítési formája a tábla, amelyben a rekordok egy-egy sornak felelnek meg. Ugyanakkor a rekordok a sorokkal szemben nem rendelkeznek meghatározott sorrenddel. Az egyes táblák oszlopait (fejléceit) az adatbázisséma határozza meg. Minden oszlophoz meg van határozva egy név és egy adattípus, amely az oszlopban tárolt összes értékre vonatkozik, a tábla összes sorában.

![Relációs adatbázis adatait megjelenítő példa](../images/example-relational.png)

Az adatokat relációs modellel rendszerező adattárakat relációs adatbázisnak nevezzük. Az elsődleges kulcsok egyedileg azonosítják a tábla sorait. A külső kulcsok mezői egy másik táblában található sorra hivatkoznak úgy, hogy az adott táblában a másik tábla elsődleges kulcsára hivatkoznak. A külső kulcsok a hivatkozási integritás megőrzésére szolgálnak, mivel biztosítják, hogy a hivatkozott sorokat ne lehessen módosítani vagy törölni, amíg a rájuk hivatkozó sor függ tőlük. 

![Relációs adatbázis adatait megjelenítő példa](../images/example-relational2.png)

A relációs adatbázisok többféle korlátozástípus használatát támogatják, hogy biztosítható legyen az adatok integritása:

- Az egyedi korlátozások azt biztosítják, hogy az oszlopban csak egyedi értékek szerepeljenek. 

- A külső kulcsokra vonatkozó korlátozások a két tábla adatai közti kapcsolat alkalmazását kényszerítik. A külső kulcs egy másik tábla elsődleges kulcsára vagy más egyedi kulcsára hivatkozik. A külső kulcsokra vonatkozó korlátozás kényszeríti a hivatkozási integritás fenntartását, és nem engedélyezi az olyan módosításokat, amelyek érvénytelen külsőkulcs-értéket eredményeznének.

- Az ellenőrző korlátozások, más néven entitásintegritási korlátozások azt korlátozzák, hogy milyen értékek tárolhatók az egyes oszlopokban, illetve ugyanazon különböző oszlopaiban. 

A legtöbb relációs adatbázis a Structured Query Language (SQL-) nyelvet használja, amely lehetővé teszi a deklaratív lekérdezések használatát. A lekérdezés a kívánt eredményt határozza meg, a végrehajtandó lépéseket nem. A motor ezután eldönti, hogyan lehet a leghatékonyabban végrehajtani a lekérdezést. Ezzel szemben a procedurális megközelítés használatakor a lekérdezőprogram konkrétan meghatározza a kívánt lépéseket. A relációs adatbázis azonban eljárások és függvények formájában tudja tárolni a végrehajtható kódok rutinjait, ami lehetővé teszi a deklaratív és procedurális megközelítés együttes használatát.

A relációs adatbázisok *indexeket* használnak a lekérdezési teljesítmény javításához. Az elsődleges kulcs által használt elsődleges indexek a lemezen való tárolási sorrend szerint határozzák meg az adatok sorrendjét. A másodlagos indexek a mezők alternatív kombinációit határozzák meg, hogy hatékonyan le lehessen kérdezni a kívánt sorokat anélkül, hogy a tényleges helyük megváltozna a lemezen.

Mivel a relációs adatbázisok kikényszerítik a hivatkozási integritás fenntartását, a relációs adatbázisok skálázása nehéz feladat lehet. Ennek az az oka, hogy a lekérdezési és beszúrási műveletek számos táblát érinthetnek. A relációs adatbázisok horizontális felskálázása történhet az adatok *horizontális skálázásával*, de ez gondosan megtervezett sémát igényel. További információ: [Horizontális skálázási minta](../../patterns/sharding.md).

Ha az adatok nem relációs jellegűek, vagy a követelményeik nem alkalmasak relációs adatbázisban való használatra, fontolja meg egy [Nem relációs vagy NoSQL-](../big-data/non-relational-data.md) adattár használatát.
