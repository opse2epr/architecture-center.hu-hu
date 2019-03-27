---
title: Szabad formátumú szöveges keresés feldolgozása
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: ba1685d12834b04e7d231929c53453b51527cce2
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298934"
---
# <a name="processing-free-form-text-for-search"></a>Szabad formátumú szöveges keresés feldolgozása

Támogatja a keresést, a szabad formátumú szöveges feldolgozási végezhetők tájékoztató bekezdéseket adjon hozzá szöveget tartalmazó dokumentumokat.

Szöveges keresés működése hozhat létre, egy speciális index, amely a kockában dokumentumok-gyűjteményeken. Egy ügyfélalkalmazás elküld egy lekérdezést, amely tartalmazza a keresési feltételeket. A lekérdezés visszaad egy eredményhalmazt, álló dokumentumok arról, hogy minden egyes dokumentum felel meg a keresési feltételek szerint rendezett listája. Az eredményhalmaz is, amelyben a dokumentum felel meg a feltételeket, a környezetet, amely lehetővé teszi az alkalmazás számára, jelölje ki a megfelelő kifejezést a dokumentum.

![Keresés folyamat ábrája](./images/search-pipeline.png)

Szabad formátumú szöveges feldolgozási hasznos, gyakorlatban hasznosítható zajos szöveges adatok nagy mennyiségű adatokat hozhat létre. Az eredményeket egy jól definiált, így bármikor lekérdezhető struktúra biztosíthat strukturálatlan dokumentumokat.

## <a name="challenges"></a>Problémák

- Szabad formátumú szöveges dokumentumok gyűjteményét feldolgozása általában nagy számítási igényes, valamint időt igénybe vevő.
- Annak érdekében, hogy a szabad formátumú szöveges hatékony keresést, a search-index támogatnia kell az intelligens keresés a feltételeket, amelyek hasonló konstrukció alapján. Például a keresési indexek beépített morfológiai és a nyelvi származtató, hogy a lekérdezések a "Futtatás" egyezni fog dokumentumok, amelyek tartalmazzák a "futott" és "fut".

## <a name="architecture"></a>Architektúra

A legtöbb esetben a forrás szöveget a dokumentumok betöltésével objektum storage például az Azure Storage vagy az Azure Data Lake Store-bA. Kivétel teljes szöveges keresést az SQL Server-vagy Azure SQL Database használata. Ebben az esetben a dokumentum-adatok betöltése az adatbázis által felügyelt táblákba. Tárolt, miután a dokumentumok feldolgozása történik a batch az index létrehozásához.

## <a name="technology-choices"></a>Technológiai lehetőségek

Azure Search az Elasticsearch és a Solr HDInsight a search-index létrehozására vonatkozó lehetőségek között. Ezek a technológiák mindegyike fel lehet tölteni egy keresési index dokumentumok gyűjteményét. Az Azure Search indexelők, amely képes automatikusan feltölti az indexet a dokumentumokra és egyszerű szövegként a Excel és a PDF formátum biztosít. A HDInsight az Apache Solr indexelésére használhatja, számos különböző, például egyszerű szöveges, Word és PDF bináris fájlokat. Az index jön létre, ha az ügyfelek hozzáférhetnek a REST API segítségével a search felületén.

Ha a szöveges adatokat az SQL Server vagy az Azure SQL Database, a teljes szöveges keresés, a database-be épített is használhatja. Az adatbázis tölti fel. az index a szöveg, bináris vagy ugyanabban az adatbázisban tárolt XML-adataiban. Ügyfelek keresése a T-SQL lekérdezések használatával.

További információkért lásd: [keresés az adattárak](../technology-choices/search-options.md).
