---
title: Technológia feldolgozása természetes nyelv kiválasztása
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: dacf7bf9cf3e9efed212f34da93c1470954965cf
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a>Az Azure-ban technológia feldolgozása természetes nyelv kiválasztása

Szabad formátumú szöveg feldolgozása bekezdést, általában a keresési, támogatása érdekében tartalmazó dokumentumokat hajtja végre, de más természetes nyelvű (NLP) műveletek véleményeket elemzés, témakör észlelése, például is szolgál nyelvi megkereshetők, a kulcs kifejezés kivonása és a dokumentum kategorizálási. Ez a cikk a technológia lehetőségek támogatásához a NLP feladatok szoftvercsomagokat összpontosít.

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a>Mik azok a beállítások a NLP szolgáltatásnak kiválasztásakor?

Azure a következő szolgáltatásokat biztosítja az természetes nyelvű feldolgozása (NLP) képességek:

- [Az Azure HDInsight Spark-és Spark MLlib](/azure/hdinsight/spark/apache-spark-overview)
- [Microsoft kognitív szolgáltatások](/azure/#pivot=products&panel=cognitive)

## <a name="key-selection-criteria"></a>Kulcs kiválasztási feltételek

A választási lehetőségek szűkítéséhez indítása ezen kérdések megválaszolásával:

- Szeretné előre elkészített modellek használata? Ha igen, akkor fontolja meg a Microsoft kognitív szolgáltatások által nyújtott API-val.

- Kell elleni szöveg adatok nagy corpus egyéni modellek betanítása? Ha igen, érdemes lehet Azure HDInsight Spark MLlib és Spark NLP.

- Alacsony szintű NLP szolgáltatásokat, például a lexikális elemekké alakításának, amelynek, Lemmatizálás és kifejezés gyakoriság/inverz dokumentum gyakorisága (TF/IDF) kell? Ha igen, érdemes lehet Azure HDInsight Spark MLlib és Spark NLP.

- Kell-e egyszerű, magas szintű NLP szolgáltatásokat, például az entitáshoz és szándéka azonosítása, témakör észlelési, helyesírás-ellenőrző vagy véleményeket elemzés? Ha igen, akkor fontolja meg a Microsoft kognitív szolgáltatások által nyújtott API-val.

## <a name="capability-matrix"></a>Képesség mátrix

A következő táblázat összefoglalja a főbb változásai képességeit.  

### <a name="general-capabilities"></a>Általános képességek

| | Azure HDInsight | Microsoft Cognitive Services |
| --- | --- | --- |
| Pretrained modellek biztosít szolgáltatásként | Nem | Igen |
| REST API | Igen | Igen |
| Programozható | Python, Scala, Java | C#, Java, Node.js, Python, PHP, Ruby |
| Big data készletek és nagy dokumentumok feldolgozásának támogatásához | Igen | Nem |

### <a name="low-level-natural-language-processing-capabilities"></a>Alacsony szintű természetes nyelvű feldolgozási képességek

| | Azure HDInsight | Microsoft Cognitive Services |  
| --- | --- | --- | 
| Jogkivonatokat létrehozó | Igen (Spark NLP) | Igen (nyelvi elemzés API) |
| Stemmer | Igen (Spark NLP) | Nem |
| Lemmatizer | Igen (Spark NLP) | Nem |
| A beszédfelismerés szabvány szerinti címkézést része | Igen (Spark NLP) | Igen (nyelvi elemzés API) |
| Kifejezés gyakoriság/inverz-dokumentum gyakorisága (TF/IDF) | Igen (Spark MLlib) | Nem |
| Karakterlánc-hasonlóság&mdash;távolság számítási szerkesztése | Igen (Spark MLlib) | Nem |
| N-gramot kiszámítása | Igen (Spark MLlib) | Nem |
| Állítsa le a word eltávolítása | Igen (Spark MLlib) | Nem |

### <a name="high-level-natural-language-processing-capabilities"></a>Magas szintű természetes nyelvű feldolgozási képességek

| | Azure HDInsight | Microsoft Cognitive Services |
| --- | --- | --- | 
| Entitás/leképezés azonosító & kivonása | Nem | Igen (nyelv ismertetése intelligens szolgáltatással (LUIS) API-t) |    
| Téma azonosítása | Igen (Spark NLP) | Igen (Szövegelemzések API) |
| Helyesírás-ellenőrzési | Igen (Spark NLP) | Igen (Bing helyesírás-ellenőrző API-t) |
| Véleményelemzés | Igen (Spark NLP) | Igen (Szövegelemzések API) |
| Nyelvfelismerés | Nem | Igen (Szövegelemzések API) |
| Több nyelv használatát mellett angol nyelven | Nem | Igen (függ a API) |

## <a name="see-also"></a>Lásd még

[Természetes nyelvű feldolgozása](../scenarios/natural-language-processing.md)