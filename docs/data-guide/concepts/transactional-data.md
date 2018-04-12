---
title: Tranzakciós adatok
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: b7fdbb403d2a438ebc59e40ef58ed8067489dddc
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/31/2018
---
# <a name="transactional-data"></a>Tranzakciós adatok

Tranzakciós adatok, amely nyomon követi a kapcsolati, a szervezetek tevékenységekkel kapcsolatos információkat. A kapcsolati jellemzően üzleti tranzakciók, például az ügyfelektől, szállítók, a leltározást, végrehajtott rendeléseket és a szolgáltatások kézbesíteni való áthaladás termékek kifizetéseket kapott. Tranzakciós események, amelyek megfelelnek a tranzakciók magukat, általában a egy time dimenzió, bizonyos számértékeket és egyéb adatok mutató hivatkozásokat tartalmaz. 

Tranzakciók általában kell lenniük *atomi* és *konzisztens*. Atomicity azt jelenti, hogy a teljes tranzakció mindig sikeres vagy sikertelen munka egy egységként, soha nem egy félig kitöltött állapotban marad. A tranzakció nem hajtható végre, ha az adatbázis-rendszer kell visszaállítása volt meg, hogy a tranzakció részeként műveleteket. A hagyományos RDBMS a visszaállítás automatikusan történik, a tranzakció nem hajtható végre, ha. Konzisztencia azt jelenti, hogy tranzakciók mindig hagyja meg az adatokat állapota érvénytelen. (Ezek a atomicity és konzisztencia nagyon kötetlen leírását. Nincsenek formális definíciója ezeket a tulajdonságokat, például a [sav](https://en.wikipedia.org/wiki/ACID).)

Tranzakciós adatbázisok is támogatja az erős konzisztencia, különböző zárolási, például pesszimista zárolás használatával gondoskodjon arról, hogy minden adatot a vállalati keretén belül kifejezetten konzisztens tranzakciókhoz, a felhasználók és a folyamatok. 

A leggyakoribb üzembe helyezési architektúrája tranzakciós adatait használó az adatréteg tároló 3-rétegű architektúra. A 3-rétegű architektúra jellemzően a bemutatási szint, üzleti logikai rétegből és adatok tárolási réteg áll. A kapcsolódó központi telepítési architektúrája a [N szintű](/azure/architecture/guide/architecture-styles/n-tier) architektúra, amely több közel-Rétegek kezelése üzleti logikát.

![Példa egy 3 szintű alkalmazás](./images/three-tier-application.png)

## <a name="typical-traits-of-transactional-data"></a>Tipikus jellemzők tranzakciós adatok

Tranzakciós adatok általában a következő jellemzőkkel rendelkezik:

| Követelmény | Leírás |
| --- | --- |
| Normalizálási | Magas normalizált |
| Séma | Írás, erősen kényszerített séma|
| Konzisztencia | Az erős konzisztencia sav biztosítja, hogy |
| Integritás | Integritása magas szintű |
| Használja a tranzakciók | Igen |
| Zárolási stratégia | Pesszimista vagy optimista|
| Updateable | Igen |
| Appendable | Igen |
| Számítási feladat | Nagy mennyiségű írási műveletekről, mérsékelt olvassa be |
| Indexelés | Elsődleges és másodlagos indexek |
| Datum mérete | Kis, közepes méretű |
| Modell | Relációs |
| Adatok alakzat | A táblázatos |
| Lekérdezés rugalmasságot | Rugalmas |
| Méretezés | Kis (MB) és nagy (néhány több TB-nyi) | 

## <a name="see-also"></a>Lásd még:

[Online tranzakció-feldolgozási](../scenarios/online-transaction-processing.md)
