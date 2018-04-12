---
title: A keresési tárolóban kiválasztása
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: ead07e307e96696faa5ddf48505eee378027523c
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/06/2018
---
# <a name="choosing-a-search-data-store-in-azure"></a>A keresési tárolóban kiválasztása az Azure-ban

Ez a cikk az Azure search adattárolókhoz technológia lehetőségeit hasonlítja össze. Címsorának adattároló létrehozásához, és speciális indexek keresés végrehajtásához szabad formátumú szöveg tárolására szolgál. A szöveges indexelt a külön tárolóban, például a blob Storage tárolóban található. Az alkalmazás elküld egy keresési adattárba, és az eredmény a megfelelő dokumentumok listáját. Ebben a forgatókönyvben kapcsolatos további információkért lásd: [szabad formátumú szöveg, a keresés feldolgozása](../scenarios/search.md). 

## <a name="what-are-your-options-when-choosing-a-search-data-store"></a>Mik a lehetőségek egy keresési adatainak kiválasztásakor tárolására?
Az Azure, a következő adatokat tároló összes core követelményeknek megfelelő Search szabad formátumú szöveg adatok alapján egy keresési indexszel megadásával:
- [Azure Search](/azure/search/search-what-is-azure-search)
- [Elasticsearch](https://azuremarketplace.microsoft.com/marketplace/apps/elastic.elasticsearch?tab=Overview)
- [Solr HDInsight](/azure/hdinsight/hdinsight-hadoop-solr-install-linux)
- [Az Azure SQL adatbázis teljes szöveges keresés](/sql/relational-databases/search/full-text-search)


## <a name="key-selection-criteria"></a>Kulcs kiválasztási feltételek

Keresés használata esetén az igényeinek megfelelő keresési adattároló kiválasztása ezen kérdések megválaszolásával megkezdéséhez:

- A saját kiszolgálóit kezelése helyett egy felügyelt szolgáltatás van szüksége?

- Megadhatja a sémát indexeli tervezési időben? Ha nem, válassza ki, amely támogatja a frissíthető sémák.

- Szüksége van az index csak a teljes szöveges keresés, vagy is szüksége van gyors teszi a numerikus adatokat és egyéb analytics? Ha túl is a teljes szöveges keresés van szüksége, fontolja meg a beállítások, amelyek támogatják a további elemzés.

- Szüksége van a naplóelemzési naplógyűjtést, összesítési és indexelt adatokat megjelenítések támogatása egy keresési indexszel? Ha igen, érdemes lehet Elasticsearch, amely része a napló analytics verem.

- Van szüksége az indexelési adatok közös dokumentum formátumok például PDF, Word, PowerPoint és Excel? Ha igen, válassza ki, hogy a dokumentum indexelők biztosít.

- Rendelkezik-e az adatbázis konkrét igényeinek? Ha igen, vegye figyelembe az alábbi funkciókat.

## <a name="capability-matrix"></a>Képesség mátrix

A következő táblázat összefoglalja a főbb változásai képességeit.

### <a name="general-capabilities"></a>Általános képességek

| | Azure Search | Elasticsearch | Solr HDInsight | SQL Database | 
| --- | --- | --- | --- | --- | 
| Van a felügyelt | Igen | Nem | Igen | Igen |  
| REST API | Igen | Igen | Igen | Nem |
| Programozható | .NET | Java | Java | T-SQL | 
| A dokumentum indexelők közös fájltípusok (PDF, DOCX, TXT és így tovább) | Igen | Nem | Igen | Nem |

### <a name="manageability-capabilities"></a>Kezelhetőségi képességeit

| | Azure Search | Elasticsearch | Solr HDInsight | SQL Database | 
| --- | --- | --- | --- | --- |
| Frissíthető séma | Nem | Igen | Igen | Igen |
| Támogatja a bővített kapacitású  | Igen | Igen | Igen | Nem |

### <a name="analytic-workload-capabilities"></a>Elemzési munkaterhelés képességek

| | Azure Search | Elasticsearch | Solr HDInsight | SQL Databash | 
| --- | --- | --- | --- | --- | 
| Támogatja a teljes szöveges keresés túl elemzés | Nem | Igen | Igen | Igen |
| A napló analytics verem része | Nem | Igen (ELK) |  Nem | Nem |
| Támogatja a szemantikai keresése | Igen (keresési hasonló csak dokumentumok) | Igen | Igen | Igen | 

### <a name="security-capabilities"></a>Biztonsági képességei

| | Azure Search | Elasticsearch | Solr HDInsight | SQL Databash | 
| --- | --- | --- | --- | --- | 
| Sorszintű biztonság | Részleges (az alkalmazás lekérdezés a szűréshez csoportazonosító szükséges) | Részleges (az alkalmazás lekérdezés a szűréshez csoportazonosító szükséges) | Igen | Igen | 
| Transzparens adattitkosítás | Nem | Nem | Nem | Igen |  
| Hozzáférés korlátozása IP-címek | Nem | Igen | Igen | Igen |   
| Hozzáférés korlátozása, hogy csak a virtuális hálózati hozzáférés engedélyezése | Nem | Igen | Igen | Igen |  
| Active Directory authentication (integrált hitelesítés) | Nem | Nem | Nem | Igen | 

## <a name="see-also"></a>Lásd még

[A keresés szabad formátumú szöveg feldolgozása](../scenarios/search.md)