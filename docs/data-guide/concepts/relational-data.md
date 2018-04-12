---
title: Relációs adatok
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 55c7354b8cec13318bbf3fda1c648cde17288854
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/31/2018
---
# <a name="relational-data"></a>Relációs adatok

Relációs adatok az adatai modellezése a relációs modell használatával. Ebben a modellben adatok kifejezve rekordokat. A *rekordot* attribútum-érték párok halmaza. Például lehet, hogy egy rekord (elemazonosító = 5, orderid = 1, item = "Szék", összeg = 200,00). A rekordokat, hogy az összes az azonos attribútumokkal nevezzük egy *kapcsolat*. 

Kapcsolatok természetes jelentésekként jelennek meg a táblák, ahol ki van téve a táblakonstruktor minden rekordjának egy sor a táblában. Azonban sora explicit a sorrend, eltérően rekordokat. Az adatbázis-séma határozza meg, hogy minden tábla oszlopai (fejlécek). Egyes oszlopok nevét és a tábla összes sorát keresztül az adott oszlopban tárolt összes értékek adattípusa van definiálva.

![Példa egy relációs adatbázist használ az adatok jelennek meg](./images/example-relational.png)

A tárolóban, amely rendezi az adatokat, és a relációs modell egy relációs adatbázisban nevezzük. Az elsődleges kulcsok egyedi módon azonosítja az egy táblázatban levő sorok. Idegen kulcs mező egy tábla tekintse meg a másik tábla elsődleges kulcsa Vezérlőpultjának egy másik tábla sorának használható. Külső kulcsok segítségével győződjön meg arról, hogy a hivatkozott sorok nem módosítani vagy törölni, amíg a hivatkozó sor függ azoktól a hivatkozási integritás karbantartása. 

![Példa egy relációs adatbázist használ az adatok jelennek meg](./images/example-relational2.png)

Relációs adatbázisok, amelyek segítenek az adatok sértetlenségének biztosítása megkötéseket különböző típusainak használatát lehetővé:

- Egyedi korlátozások győződjön meg arról, hogy egy oszlop összes értékének egyediek. 

- Külső kulcsra vonatkozó megkötések kényszerítése az adatokat a két tábla közötti kapcsolat. Egy külső kulcs az elsődleges kulcs vagy egy másik egyedi kulcs hivatkozik egy másik táblából. Egy külső kulcsmegkötés integritás, visszautasítsa érvénytelen idegen kulcs értékei okozó módosítások.

- Ellenőrzési korlátozásokban, más néven entitás integritási korlátozásait, a határértékeket belül csak egy oszlop, vagy a kapcsolatot az ugyanabban a sorban szereplő értékekre tárolható. 

A legtöbb relációs adatbázisok használja a Structured Query Language (SQL) nyelv, amely lehetővé teszi, hogy a deklaratív megközelítése lekérdezése. A lekérdezés a kívánt eredményt, de nem a lekérdezés végrehajtása a lépéseket ismerteti. A motor majd úgy dönt, a legjobb módszer a lekérdezés végrehajtásához. Ez eltér a eljárási módszert használja, ha a lekérdezés program határozza meg a feldolgozási lépést explicit módon. Relációs adatbázisok azonban tárolhatja végrehajtható kód rutinok formájában tárolt eljárásokat és függvényeket, amely lehetővé teszi a deklaratív és eljárási szempontok keveréke.

Lekérdezés teljesítmény javítása érdekében a relációs adatbázisok használja *indexek*. Elsődleges indexek, az elsődleges kulcs által használt, az adatok sorrendje határozza meg azt, akkor a lemezen helyezkedik el. Másodlagos indexek biztosítanak egy, a mezők alternatív kombinációja, így a kívánt sorokat kérdezhetők le hatékonyan, anélkül, hogy újra rendezésére a teljes tárolt adatokat.

Mivel a hivatkozási integritás relációs adatbázisok kihívást skálázás egy relációs adatbázisban válhat. Ennek oka az, a lekérdezés vagy insert művelet lehet, hogy touch korlátlan számú táblát. Ki lehet terjeszteni a relációs által *horizontális* az adatokat, de ez a séma gondos tervezés szükséges. További információkért lásd: [horizontális mintát](../../patterns/sharding.md).

Ha nem relációs adatokat, vagy nem alkalmas a relációs adatbázis-követelmények vonatkoznak, fontolja meg egy [nem relációs vagy NoSQL](./non-relational-data.md) adattár.
