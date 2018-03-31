---
title: "Online analitikus feldolgozási (OLAP)"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: f5ceea9c9dd03812e92fff811e54316edc22b59c
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
---
# <a name="online-analytical-processing-olap"></a>Online analitikus feldolgozási (OLAP)

Online analitikus feldolgozási (OLAP) egy olyan technológia, nagy üzleti adatbázisok rendezi, amely támogatja az összetett elemzést. Anélkül, hogy negatívan befolyásolná a tranzakciós rendszerek bonyolult elemzési lekérdezések végrehajtásához használható.

Az adatbázisok, amely egy üzleti használ a tranzakciók és nevezzük [online tranzakció-feldolgozási (OLTP)](online-transaction-processing.md) adatbázisok. Ezeket az adatbázisokat általában azt jelzi, hogy a rendszer a megadott egyszerre csak egy rendelkeznek. Gyakran tartalmaznak, amely a szervezet számára a legértékesebb sok. Az OLTP, a használt adatbázisok azonban nem képesek elemzés céljából. Ezért válaszok lekérése ezeket az adatbázisokat, idő és erőfeszítés tekintetében költségesnek számít kinyerni. OLAP-rendszerek arra tervezték, hogy az adatok magas performant az üzleti intelligenciával kapcsolatos információk kinyerése érdekében módon. Ennek oka az, olvasási, írási alacsony munkaterhelések gyakori vannak optimalizálva, OLAP-adatbázisokat.

![Az Azure-ban OLAP](./images/olap-data-pipeline.png) 

## <a name="when-to-use-this-solution"></a>Ez a megoldás használatával

Vegye figyelembe a OLAP a következő esetekben:

- Összetett analitikai és ad hoc lekérdezéseket gyorsan, anélkül, hogy negatívan befolyásolná az OLTP rendszerek hajt végre kell. 
- Kívánt biztosít üzleti felhasználók egyszerűen az adatok alapján jelentések készítése
- Adjon meg egy számot az összesítések, amelyek lehetővé teszik a felhasználók gyors, konzisztens eredmények el szeretné. 

OLAP különösen fontos összesített számítások alkalmazásának keresztül nagy mennyiségű adat. OLAP-rendszerek olvasás-nehéz helyzetekben, például elemzés és a business intelligence vannak optimalizálva. OLAP lehetővé teszi a felhasználóknak szegmens többdimenziós adatok időszeletekre tekintheti meg (például egy olyan kimutatás) két dimenzió vagy adott értékek szerint szűrje az adatokat. Ezt a folyamatot nevezik néha "szeletelhető és feldarabolható" az adatokat, függetlenül attól, hogy az adatok particionálása több adatforrások keresztül hajtható végre. Ez segít a felhasználók kereshetnek trendek, helyszíni minták, és áttekintheti az adatokat a hagyományos adatelemzés részleteit ismerete nélkül.

[A szemantikai modellek](../concepts/semantic-modeling.md) üzleti felhasználók absztrakt kapcsolat összetett szolgáltatásokkal, és könnyebben adatok gyors elemzését.

## <a name="challenges"></a>Kihívásai

Az összes, az OLAP-rendszerek előnyöket biztosítanak azok néhány kihívást eredményez:

- Az OLTP rendszerek adatok folyamatosan frissül a különböző forrásokból folyik a tranzakciók, mivel OLAP-adattároló általában sokkal lassabb időközönként, attól függően, hogy az üzleti igények frissítése. Ez azt jelenti, hogy az OLAP-rendszerek alkalmasabbak módosítások azonnali válaszokat, hanem stratégiai üzleti döntéseket hozhat. Bizonyos fokú adatok tisztításokat és az orchestration kell az OLAP-adattároló naprakész állapotban tartása érdekében meg kell tervezni.
- Hagyományos, normalizált, relációs táblákat a OLTP rendszerek, eltérően OLAP adatmodellekben általában többdimenziós. Így nehéz vagy lehetetlen közvetlenül hozzárendelése egyedkapcsolat vagy objektumorientált modellek, ahol minden attribútum egy oszlopra van leképezve. OLAP-rendszerek általában inkább egy csillag vagy snowflake séma hagyományos normalizálási helyett.

## <a name="olap-in-azure"></a>Az Azure-ban OLAP

Az Azure data tárolt OLTP rendszerek például az Azure SQL Database másolódik az OLAP-rendszert, például a [Azure Analysis Services](/azure/analysis-services/analysis-services-overview). Adatok feltárása és a képi megjelenítés eszközök, például [Power BI](https://powerbi.microsoft.com), az Excel és a külső beállítások Analysis Services-kiszolgálókkal, és a modellezett adatok magas interaktív és vizuálisan részletes betekintést biztosít a felhasználók. Az adatáramlás OLTP adatokból való OLAP általában vezénylését használatával hajtható végre SQL Server Integration Services használatával [Azure Data Factory](/azure/data-factory/concepts-integration-runtime).

## <a name="technology-choices"></a>Technológiai lehetőségek

- [Online analitikus feldolgozási (OLAP) adattároló](../technology-choices/olap-data-stores.md)

