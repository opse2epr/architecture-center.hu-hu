---
title: A természetes nyelvi feldolgozási technológia kiválasztása
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: bac0318a587a944c104360eb31223cc8755c1860
ms.sourcegitcommit: e9eb2b895037da0633ef3ccebdea2fcce047620f
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 10/30/2018
ms.locfileid: "50251770"
---
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a>Az Azure-ban technológia feldolgozása természetes nyelv kiválasztása

Szabad formátumú szöveges feldolgozási hajtja végre az szöveg-, keresési, támogatása céljából általában bekezdéseit tartalmazó dokumentumok, de is szolgál, mint a hangulatelemzés, témafelismerés, természetes nyelvi feldolgozást (NLP) feladatait más nyelv észlelése, a kulcsszókeresést és kategorizálási dokumentum. Ez a cikk a technológiai lehetőségekhez támogatásához a NLP feladatokat-kiszolgálóként működő összpontosít.

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a>Mik azok a beállítások egy NLP szolgáltatás kiválasztásakor?

Az Azure-ban a következő szolgáltatásokat nyújtanak a természetes nyelvi feldolgozási (NLP) képességek:

- [Az Azure HDInsight Spark-és Spark MLlib](/azure/hdinsight/spark/apache-spark-overview)
- [Azure Databricks](/azure/azure-databricks/what-is-azure-databricks)
- [A Microsoft Cognitive Services](/azure/cognitive-services/welcome)

## <a name="key-selection-criteria"></a>Fontosabb kritériumok

Így szűkítheti, első lépésként a kérdések megválaszolása:

- Biztosan előre összeállított modellek használata? Ha igen, érdemes a Microsoft Cognitive Services által kínált az API-k használatával.

- Szükség van egy nagy forrásgyűjteményébe álló szöveges adatok elleni egyéni modelleket taníthat be? Ha igen, fontolja meg az Azure HDInsight Spark MLlib és a Spark NLP.

- A jogkivonatok, származtató, morfológiai és kifejezés gyakoriság/inverz dokumentum gyakorisága (TF/IDF) alacsony szintű NLP képességek szükségesek? Ha igen, fontolja meg az Azure HDInsight Spark MLlib és a Spark NLP.

- Egyszerű, magas szintű NLP szolgáltatásokat, például az entitás- és leképezés azonosítása, témafelismerés, helyesírás-ellenőrzés vagy hangulatelemzés kell? Ha igen, érdemes a Microsoft Cognitive Services által kínált az API-k használatával.

## <a name="capability-matrix"></a>Képességmátrix

A következő táblázat összefoglalja a fő különbségeket, a képességek.  

### <a name="general-capabilities"></a>Általános képességek

| | Azure HDInsight | Microsoft Cognitive Services |
| --- | --- | --- |
| Modellek imagenet kínál szolgáltatásként | Nem | Igen |
| REST API | Igen | Igen |
| Programozhatóság | Python, Scala, Java | C#, Java, Node.js, Python, PHP, Ruby |
| Big data jellegű adatkészletek és a nagy dokumentumok feldolgozásának támogatásához | Igen | Nem |

### <a name="low-level-natural-language-processing-capabilities"></a>Az alsó szintű természetes nyelvi feldolgozási képességek

| | Azure HDInsight | Microsoft Cognitive Services |  
| --- | --- | --- | 
| Jogkivonatokat létrehozó | Igen (Spark NLP) | Igen (nyelvi elemzési API-t) |
| Szótőkereső | Igen (Spark NLP) | Nem |
| Lemmatizer | Igen (Spark NLP) | Nem |
| Szövegrészeket része | Igen (Spark NLP) | Igen (nyelvi elemzési API-t) |
| Kifejezés gyakoriság/inverz-dokumentum gyakorisága (TF/IDF) | Igen (Spark MLlib) | Nem |
| Karakterlánc-hasonlóság&mdash;távolság számítási szerkesztése | Igen (Spark MLlib) | Nem |
| N-Gram típusú számítási | Igen (Spark MLlib) | Nem |
| Állítsa le a word eltávolítása | Igen (Spark MLlib) | Nem |

### <a name="high-level-natural-language-processing-capabilities"></a>Magas szintű természetes nyelvi feldolgozási képességek

| | Azure HDInsight | Microsoft Cognitive Services |
| --- | --- | --- | 
| Entitás/szándékot azonosítása és kinyerése | Nem | Igen (Language Understanding Intelligent Service (LUIS) API) |    
| Téma azonosítása | Igen (Spark NLP) | Igen (szövegelemzési API-val) |
| Helyesírás-ellenőrzés | Igen (Spark NLP) | Igen (a Bing Spell Check API bemutatása) |
| Hangulatelemzés | Igen (Spark NLP) | Igen (szövegelemzési API-val) |
| Nyelvfelismerés | Nem | Igen (szövegelemzési API-val) |
| Angol mellett több nyelvet is támogat | Nem | Igen (eltérő API-t) |

## <a name="see-also"></a>Lásd még

[Természetes nyelvű feldolgozása](../scenarios/natural-language-processing.md)
