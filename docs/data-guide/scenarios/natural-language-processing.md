---
title: Természetes nyelvek feldolgozása
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 0afd8ac9a8a2e56f79ade0b2e10328630866c03c
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/03/2018
ms.locfileid: "30297971"
---
# <a name="natural-language-processing"></a>Természetes nyelvek feldolgozása

Természetes nyelvű feldolgozás (NLP) olyan feladatokhoz, mint a céggel kapcsolatos véleményeket elemzés, témakör észlelési, nyelvi észlelési, kulcs kifejezés kivonása és dokumentum kategorizálási szolgál.

![](./images/nlp-pipeline.png)

## <a name="when-to-use-this-solution"></a>Ez a megoldás használatával

NLP segítségével dokumentumok, például címkézése a bizalmas dokumentumok besorolását vagy kéretlen információk küldésére. További feldolgozási vagy keresési NLP kimenete használható. Egy másik NLP a rendeltetése, hogy összesítse a szöveg a dokumentumban található entitások azonosításával. Ezeket az entitásokat is használható a címke dokumentumok kulcsszavak, amely lehetővé teszi, hogy a Keresés és a tartalom alapján lekérése. Entitások témakörökre összesítéseket minden a dokumentumban található Fontos témakörök ismertetik a lehet kombinálni. Az észlelt témakörök a dokumentumokat navigációs kategorizálását, vagy operációs rendszer megadott a kijelölt téma kapcsolódó dokumentumok használható. Egy másik a NLP használata céggel kapcsolatos véleményeket, annak ellenőrzéséhez, a pozitív vagy negatív hangvételt érhet el, a dokumentum szövege pontozása céljából. Ezek a módszerek sok technikák feldolgozására, például a természetes nyelv használata: 

- **Jogkivonatokat létrehozó**. A szöveg bontani szavakat vagy kifejezéseket.
- **Származó és Lemmatizálás**. Így azaz, hogy a különböző szavak normalizálása űrlapok van leképezve a kanonikus szót jelentéssel bír. Például "fut" és "Itt futott" térkép "futtatni." 
- **Entitás kibontási**. A szöveg témái azonosítása.
- **Rész melyik észlelési**. Azonosító szöveg művelet, főnév, participle, művelet kifejezés és így tovább.
- **Határ észlelési mondatában**. Annak ellenőrzése, hogy teljes mondatokat bekezdést belül.

Használata NLP információinak és kibontani szabad formátumú szöveg, a kiindulási pont esetén általában a nyers objektum storage például az Azure Storage vagy az Azure Data Lake Store-ban tárolt dokumentumokhoz. 

## <a name="challenges"></a>Problémák

- Feldolgozására szabad formátumú szöveg dokumentumok gyűjteménye általában számításilag erőforrás-igényesek, valamint intenzív idő alatt.
- Nélkül egy szabványos dokumentum formátuma nem nagyon nehéz feladat, szabad formátumú szöveg-feldolgozást használó adott tények kibontani a dokumentum következetesen pontos eredmények elérése érdekében. Például, gondolja át, hogy a szöveg megjelenítése számla&mdash;nehézkes hozhat létre egy folyamatot, amely helyesen kibontja a számla számát és a számla dátum számlák tetszőleges számú szállítók között lehet.

## <a name="architecture"></a>Architektúra

A NLP megoldás szabad formátumú szöveg feldolgozás bekezdést tartalmazó dokumentumokon végzett történik. Általános architektúrája lehet egy [kötegfeldolgozási](../big-data/batch-processing.md) vagy [valós idejű stream feldolgozási](../big-data/real-time-processing.md) architektúra.

Tényleges feldolgozását a kívánt eredmény függ, de a folyamat szempontjából NLP olyan kötegelt vagy a valós idejű módon lehet alkalmazni. Például véleményeket analysis használható szöveg blokkok szemben a céggel kapcsolatos véleményeket pontszám létrehozásához. Ez sikerült végezhető kötegfolyamat futtatásával az tárolójában, vagy kisebb kivonatokból üzenetküldési szolgáltatáson keresztül áramló adatok valós idejű adatok alapján.

## <a name="technology-choices"></a>Technológiai lehetőségek

- [Természetes nyelvű feldolgozása](../technology-choices/natural-language-processing.md)