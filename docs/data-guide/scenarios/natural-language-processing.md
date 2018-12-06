---
title: Természetes nyelvek feldolgozása
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 32f2a5e3a0baddc765fb36ccc42fe2626faa5eba
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/05/2018
ms.locfileid: "52901439"
---
# <a name="natural-language-processing"></a>Természetes nyelvek feldolgozása

Természetes nyelvi feldolgozást (NLP) olyan feladatokhoz, mint a hangulatelemzés, témafelismerés, nyelv észlelése, kulcsszókeresést és kategorizálási dokumentum szolgál.

![](./images/nlp-pipeline.png)

## <a name="when-to-use-this-solution"></a>Ez a megoldás használata

NLP használhatja például a bizalmas dokumentumok címkézés a dokumentumok besorolását vagy kéretlen információk küldésére. NLP kimenetét további feldolgozás vagy keresési használható. Nlp-vel egy másik használatát, hogy szöveges összegzés szempontja a dokumentumban található entitások azonosítása. Ezeket az entitásokat is használható a címke dokumentumokhoz kulcsszavas, amely lehetővé teszi, hogy a Keresés és a tartalom alapján lekéréséhez. Entitások témakörök ikonjával állapotösszegzéseket, amelyek a jelen dokumentum fontos témakörök ismertetik azokat lehet kombinálni. Észlelt témaköreit használhatók, így a Navigálás a dokumentumok csoportosítására vagy kapcsolódó dokumentumokat adott a kijelölt téma számbavétele. Nlp-vel egy másik használatát, hogy pontozása szövege hangulatát, annak ellenőrzéséhez, a dokumentum pozitív vagy negatív képviselő hangvételét. Ezek a módszerek természetes nyelvi feldolgozás, mint például a számos módszerek használatáról: 

- **Jogkivonatokat létrehozó**. Szöveg felosztása szavakat vagy kifejezéseket.
- **Amely szerint és morfológiai**. Normalizálása, így azt, hogy a különböző szavak jelentése megegyezik a canonical szót leképezése űrlapok. Például "fut" és "futott" leképezés "futtatásához." 
- **Entitások kinyeréséhez**. A szövegben témák azonosítása.
- **Rész melyik észlelési**. Azonosító szöveg egy művelet, főnév, participle, művelet kifejezés és így tovább.
- **Határ észlelési Mondatkezdő**. Annak ellenőrzése, hogy teljes mondatokból bekezdést belül.

NLP használatával szabad formátumú szöveges információinak és kinyerése, a kiindulási pont esetén általában a objektum storage például az Azure Storage vagy az Azure Data Lake Store-ban tárolt nyers dokumentumokat. 

## <a name="challenges"></a>Problémák

- Szabad formátumú szöveges dokumentumok gyűjteményét feldolgozása általában nagy számítási erőforrás-igényesek, valamint időigényes folyamatban van.
- Szabványos dokumentumformátuma nélkül lehet nagyon nehéz szabad formátumú szöveges-feldolgozást használó konkrét tényekre kibontani a dokumentumot folyamatosan pontos eredmények elérése érdekében. Például, gondolja át, hogy egy szöveges ábrázolása a számla&mdash;hozhat létre olyan folyamat, amely megfelelően kinyeri a számla száma és a számla dátuma számlák szállítók bármennyi nehézkes lehet.

## <a name="architecture"></a>Architektúra

A megoldás nlp-vel a szabad formátumú szöveges feldolgozás tájékoztató bekezdéseket adjon hozzá szöveget tartalmazó dokumentumokon történik. Az általános architektúrát lehet egy [kötegelt feldolgozás](../big-data/batch-processing.md) vagy [valós idejű adatfolyam-feldolgozás](../big-data/real-time-processing.md) architektúra.

A tényleges feldolgozását a kívánt eredmény alapján változik, de tekintetében a folyamatban, nlp-vel a batch vagy a valós idejű el lesznek érvényesek. Például hangulatelemzés használható elleni blokkolja a szöveg egy véleménypontszámot előállításához. Ez sikerült hajtható végre olyan kötegelt feldolgozást futtat a storage-ban, vagy kisebb adattömbökben az adatok áramlása az üzenetküldő szolgáltatás használatával valós idejű adatok.

## <a name="technology-choices"></a>Technológiai lehetőségek

- [Természetes nyelvű feldolgozása](../technology-choices/natural-language-processing.md)