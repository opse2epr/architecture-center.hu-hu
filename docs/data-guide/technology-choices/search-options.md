---
title: Keresés adattár kiválasztása
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 9576bbba9609a04ccc7851d55dd28853ffc6b701
ms.sourcegitcommit: f6be2825bf2d37dfe25cfab92b9e3973a6b51e16
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 10/08/2018
ms.locfileid: "48858180"
---
# <a name="choosing-a-search-data-store-in-azure"></a>Az Azure search adattár kiválasztása

Ez a cikk összehasonlítja az adattárolók keresése az Azure-ban technológiai lehetőségek. Értesítés keresése adattár hozhat létre, és speciális indexeket a szabad formátumú szöveges keresés végrehajtásához tárolására szolgál. A szöveges indexelt várakozó előfordulhat, hogy egy különálló adattárral, például a blob storage-bA. Egy alkalmazás elküld egy lekérdezést a keresés adattárba, és az eredmény az egyező dokumentumok listája. Ebben a forgatókönyvben kapcsolatos további információkért lásd: [szabad formátumú szöveges keresés feldolgozása](../scenarios/search.md). 

## <a name="what-are-your-options-when-choosing-a-search-data-store"></a>Mik azok a beállítások kiválasztásakor egy keresési adatok tárolására?
Az Azure-ban mind a következő adattárakat felel meg a keresési szabad formátumú szöveges adatok alapvető követelményei azáltal, hogy a keresési index:
- [Azure Search](/azure/search/search-what-is-azure-search)
- [Elasticsearch](https://azuremarketplace.microsoft.com/marketplace/apps/elastic.elasticsearch?tab=Overview)
- [A Solr HDInsight](/azure/hdinsight/hdinsight-hadoop-solr-install-linux)
- [Az Azure SQL-adatbázis teljes szöveges keresés](/sql/relational-databases/search/full-text-search)


## <a name="key-selection-criteria"></a>Fontosabb kritériumok

Keresési forgatókönyvek esetén az igényeinek megfelelő keresési adattár kiválasztásával ezen kérdések megválaszolásával kezdete:

- Szeretne egy felügyelt szolgáltatás, hanem a saját kiszolgálók kezelése?

- Megadhatja az indexsémában már a tervezés során? Ha nem, válassza ki, amely támogatja a frissíthető sémák.

- Van szüksége az index csak a teljes szöveges keresés, vagy is szüksége van gyors összesítés numerikus adatokat és más elemzési? Ha a teljes szöveges keresési funkciókat van szüksége, fontolja meg, amelyek támogatják a további elemzési lehetőségeket.

- Szükség van a keresési index naplógyűjtés, aggregációs és indexelt adatok Vizualizációk támogatása a log Analytics? Ha igen, érdemes lehet az Elasticsearch, a log analytics-verem része.

- Szükség van az adatok indexelése a gyakori dokumentum formátumban, például PDF, Word, PowerPoint és Excel? Ha igen, válasszon egy beállítást, amely dokumentumot indexelők biztosít.

- Rendelkezik-e az adatbázis konkrét igényeinek? Ha igen, fontolja meg az alább felsorolt biztonsági funkciók.

## <a name="capability-matrix"></a>Képességmátrix

A következő táblázat összefoglalja a fő különbségeket, a képességek.

### <a name="general-capabilities"></a>Általános képességek

| | Azure Search | Elasticsearch | A Solr HDInsight | SQL Database | 
| --- | --- | --- | --- | --- | 
| A felügyelt szolgáltatás | Igen | Nem | Igen | Igen |  
| REST API | Igen | Igen | Igen | Nem |
| Programozhatóság | .NET | Java | Java | T-SQL | 
| A dokumentum az indexelők a gyakori fájltípusokból (PDF, DOCX, TXT és így tovább) | Igen | Nem | Igen | Nem |

### <a name="manageability-capabilities"></a>Kezelhetőségi képességeit

| | Azure Search | Elasticsearch | A Solr HDInsight | SQL Database | 
| --- | --- | --- | --- | --- |
| Frissíthető séma | Nem | Igen | Igen | Igen |
| Támogatja a horizontális felskálázás  | Igen | Igen | Igen | Nem |

### <a name="analytic-workload-capabilities"></a>Elemzési számítási feladatok képességek

| | Azure Search | Elasticsearch | A Solr HDInsight | SQL Database | 
| --- | --- | --- | --- | --- | 
| A teljes szöveges keresés túl az elemzést támogatja | Nem | Igen | Igen | Igen |
| A log analytics kifejlesztése | Nem | Igen (elk-val) |  Nem | Nem |
| Szemantikai támogatja a keresést | Igen (keresési hasonló csak dokumentumok) | Igen | Igen | Igen | 

### <a name="security-capabilities"></a>Biztonsági képességei

| | Azure Search | Elasticsearch | A Solr HDInsight | SQL Database | 
| --- | --- | --- | --- | --- | 
| Sorszintű biztonság | Részlegesen (csoportazonosító szerinti szűréshez használandó alkalmazás lekérdezést igényel) | Részlegesen (csoportazonosító szerinti szűréshez használandó alkalmazás lekérdezést igényel) | Igen | Igen | 
| Transzparens adattitkosítás | Nem | Nem | Nem | Igen |  
| Adott IP-címek való hozzáférés korlátozása | Nem | Igen | Igen | Igen |   
| Korlátozhatja a hozzáférést, csak a virtuális hálózati hozzáférés engedélyezése | Nem | Igen | Igen | Igen |  
| Az Active Directory-hitelesítés (integrált hitelesítés) | Nem | Nem | Nem | Igen | 

## <a name="see-also"></a>Lásd még

[Szabad formátumú szöveges keresés feldolgozása](../scenarios/search.md)
