---
title: A keresés szabad formátumú szöveg feldolgozása
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: e53730bd2e179c82399e0f92b6c5ce7f451a2f46
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
ms.locfileid: "29291837"
---
# <a name="processing-free-form-text-for-search"></a>A keresés szabad formátumú szöveg feldolgozása

Támogatja a keresést, a szabad formátumú szöveg feldolgozási Indexalapú bekezdést tartalmazó dokumentumokat.

Szöveges keresés működését hozhat létre, egy speciális indexet, amely a dokumentumok gyűjteménye elleni-e a kockában. Egy ügyfélalkalmazás elküld egy, amely tartalmazza a keresett kifejezéseket. A lekérdezés egy eredményhalmaz álló dokumentumok mennyire minden egyes dokumentum megfelel a keresési feltételek szerint rendezett listáját adja vissza. Az eredménykészlet is tartalmazhatják, amelyben a dokumentum megfelel a feltételeknek, a környezettel, amely lehetővé teszi az alkalmazások, jelölje ki a megfelelő kifejezést a dokumentumban. 

![](./images/search-pipeline.png)

Szabad formátumú szöveg feldolgozási eredményezhet nagy mennyiségű adat zajos szöveg hasznos, végrehajthatóként adatait. Az eredmények biztosíthat strukturálatlan dokumentumok jól meghatározott, így bármikor lekérdezhető struktúrában.


## <a name="challenges"></a>Kihívásai

- Feldolgozására szabad formátumú szöveg dokumentumok gyűjteménye általában számításilag intenzív, valamint időt igénybe vevő.
- Ahhoz, hogy a keresett hatékonyan szabad formátumú szöveg, a search-index támogatnia kell az intelligens egyeztetésű keresési feltételeket, amelyek rendelkeznek egy hasonló konstrukció alapján. Például felel meg a keresési indexek beépített Lemmatizálás és nyelvi származó, hogy a "Futtatás" lekérdezések a dokumentumok, amelyek tartalmazzák a "Futtatás" és "fut".

## <a name="architecture"></a>Architektúra

A legtöbb esetben a forrás szövegfájlok töltődnek be a objektum tárhelyen, többek között az Azure Storage vagy az Azure Data Lake Store. Kivétel által használt SQL Server vagy az Azure SQL adatbázis teljes szöveges keresés. Ebben az esetben a dokumentum adatok betöltése az adatbázis által kezelt táblákba. Miután tárolja, a dokumentumok feldolgozása hozza létre az indexet a kötegben.

## <a name="technology-choices"></a>Technológiai lehetőségek

A keresési index létrehozását a választható lehetőségek Azure Search Elasticsearch és HDInsight Solr. Ezek a technológiák mindegyikének feltöltheti egy keresési indexszel, a dokumentumok a gyűjteményből. Az Azure Search biztosít, amely automatikusan feltöltheti az Excel és a PDF formátum egyszerű szöveges kezdve dokumentumok index indexelők. A HDInsight Apache Solr indexelheti számos különböző, beleértve az egyszerű szöveges, a Word és PDF bináris fájljait. Az index összeállított, ha az ügyfelek hozzáférhetnek a REST API segítségével a keresési felület. 

Ha az SQL Server vagy az Azure SQL Database szöveg adatai, a teljes szöveges keresés az adatbázisba részét képező is használhatja. Az adatbázis tölti fel az index a szöveg, bináris vagy XML-adatok ugyanabban az adatbázisban tárolja. Ügyfelek keresése a T-SQL-lekérdezések használatával. 

További információkért lásd: [adattárolókhoz keresési](../technology-choices/search-options.md).
