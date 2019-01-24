---
title: Keresés adattár kiválasztása
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: c0362ff3bc6c115399892d0f066650aaa96af2dd
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486556"
---
# <a name="choosing-a-search-data-store-in-azure"></a>Az Azure search adattár kiválasztása

Ez a cikk összehasonlítja az adattárolók keresése az Azure-ban technológiai lehetőségek. Értesítés keresése adattár hozhat létre, és speciális indexeket a szabad formátumú szöveges keresés végrehajtásához tárolására szolgál. A szöveges indexelt várakozó előfordulhat, hogy egy különálló adattárral, például a blob storage-bA. Egy alkalmazás elküld egy lekérdezést a keresés adattárba, és az eredmény az egyező dokumentumok listája. Ebben a forgatókönyvben kapcsolatos további információkért lásd: [szabad formátumú szöveges keresés feldolgozása](../scenarios/search.md).

<!-- markdownlint-disable MD026 -->

## <a name="what-are-your-options-when-choosing-a-search-data-store"></a>Mik azok a beállítások kiválasztásakor egy keresési adatok tárolására?

<!-- markdownlint-enable MD026 -->

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

| | Azure Search | Elasticsearch | A Solr HDInsight | SQL-adatbázis |
| --- | --- | --- | --- | --- |
| A felügyelt szolgáltatás | Igen | Nem | Igen | Igen |  
| REST API | Igen | Igen | Igen | Nincs |
| Programozhatóság | .NET | Java | Java | T-SQL |
| A dokumentum az indexelők a gyakori fájltípusokból (PDF, DOCX, TXT és így tovább) | Igen | Nem | Igen | Nincs |

### <a name="manageability-capabilities"></a>Kezelhetőségi képességeit

| | Azure Search | Elasticsearch | A Solr HDInsight | SQL-adatbázis |
| --- | --- | --- | --- | --- |
| Frissíthető séma | Nincs | Igen | Igen | Igen |
| Támogatja a horizontális felskálázás  | Igen | Igen | Igen | Nincs |

### <a name="analytic-workload-capabilities"></a>Elemzési számítási feladatok képességek

| | Azure Search | Elasticsearch | A Solr HDInsight | SQL-adatbázis |
| --- | --- | --- | --- | --- |
| A teljes szöveges keresés túl az elemzést támogatja | Nincs | Igen | Igen | Igen |
| A log analytics kifejlesztése | Nincs | Igen (elk-val) |  Nincs | Nincs |
| Szemantikai támogatja a keresést | Igen (keresési hasonló csak dokumentumok) | Igen | Igen | Igen |

### <a name="security-capabilities"></a>Biztonsági képességek

| | Azure Search | Elasticsearch | A Solr HDInsight | SQL-adatbázis |
| --- | --- | --- | --- | --- |
| Sorszintű biztonság | Részlegesen (csoportazonosító szerinti szűréshez használandó alkalmazás lekérdezést igényel) | Részlegesen (csoportazonosító szerinti szűréshez használandó alkalmazás lekérdezést igényel) | Igen | Igen |
| Transzparens adattitkosítás | Nincs | Nem | Nem | Igen |  
| Adott IP-címek való hozzáférés korlátozása | Nincs | Igen | Igen | Igen |
| Korlátozhatja a hozzáférést, csak a virtuális hálózati hozzáférés engedélyezése | Nincs | Igen | Igen | Igen |  
| Az Active Directory-hitelesítés (integrált hitelesítés) | Nincs | Nem | Nem | Igen |

## <a name="see-also"></a>Lásd még

[Szabad formátumú szöveges keresés feldolgozása](../scenarios/search.md)
