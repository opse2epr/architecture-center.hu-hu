---
title: "Egy kognitív technológia kiválasztása"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: d97e166abed4670e4bdc797cc8075be3314e677a
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-microsoft-cognitive-services-technology"></a>A Microsoft kognitív technológia kiválasztása

Microsoft kognitív szolgáltatások felhőalapú API-k segítségével is mesterséges intelligencia (AI) alkalmazások és adatok zajlik. Jelentenek az pretrained modellek, amelyek az adatok és a nem modell betanítási beavatkozást igénylő alkalmazás használatra kész. A Microsoft által fejlesztett a kognitív szolgáltatások AI és kutatási vonja össze, és kihasználhatják a legújabb mély tanulási algoritmus. HTTP REST kapcsolatokon használják fel. Emellett a SDK-k esetén sok közös alkalmazás fejlesztési keretrendszerek érhetők el.

A kognitív szolgáltatások a következők:

* Szövegelemzés
* Számítógép stratégiai
* Videóelemzés
* A beszédfelismerés és létrehozása
* Természetes nyelvű ismertetése
* Intelligens keresési

Főbb előnyei:

* Az-a-legkorszerűbb AI számára elérhető minimális fejlesztési.
* Egyszerű integráció alkalmazásokba HTTP REST-felületek használatával.
* Beépített támogatása az Azure Data Lake Analytics kognitív szolgáltatások fel.

Szempontok:

* Csak a weben keresztül érhető el. Internetkapcsolat általában szükség. Egy kivétel ez alól az egyéni stratégiai szolgáltatás, amelynek betanított modell előrejelzés eszközök és az IoT-peremhálózaton exportálhatja.
* Jelentős testreszabási támogatják, de az elérhető szolgáltatások nem követelményeinek. a prediktív elemzés minden. Előfordulhat, hogy megfelelően.

## <a name="what-are-your-options-when-choosing-amongst-the-cognitive-services"></a>Mik azok a beállítások között a kognitív szolgáltatások kiválasztásakor?
Az Azure nincsenek kognitív szolgáltatások több tucatnyi elérhető. Ezek az aktuális átjáróra a listában egy könyvtárban támogatják-e a funkcionális terület szerint érhető el:
- [Vision](https://azure.microsoft.com/services/cognitive-services/directory/vision/)
- [Speech](https://azure.microsoft.com/services/cognitive-services/directory/speech/)
- [Knowledge](https://azure.microsoft.com/services/cognitive-services/directory/know/)
- [Search](https://azure.microsoft.com/services/cognitive-services/directory/search/)
- [Nyelv](https://azure.microsoft.com/services/cognitive-services/directory/lang/)

## <a name="key-selection-criteria"></a>Kulcs kiválasztási feltételek

A választási lehetőségek szűkítéséhez indítása ezen kérdések megválaszolásával:

- Milyen típusú adatok akkor foglalkoznak? Szűkítenie a lehetőségeit dolgozunk a bemeneti adatok típusa alapján. Például, ha a bemeneti szöveg, jelölje be az a szöveg egy bemeneti típussal rendelkező szolgáltatásokból. 

- Rendelkezik a modell betanításához az adatokat? Ha igen, fontolja meg az egyéni szolgáltatások, amelyek lehetővé teszik a mögöttes adatokat ad meg, a pontosságot és teljesítmény modellek betanítása. 

## <a name="capability-matrix"></a>Képesség mátrix

A következő táblázat összefoglalja a főbb változásai képességeit. 

### <a name="uses-prebuilt-models"></a>Használja az előre elkészített modellek
| | A bemeneti típus | Fő előnyt |
| --- | --- | --- |
| Szövegelemzések API | Szöveg | Értékelje ki a céggel kapcsolatos véleményeket, és szeretné tudni, hogy a felhasználók milyen témaköröket. |
| Entitás Linking API| Szöveg | Tegye hatékonyabbá alkalmazása adatkapcsolatait elnevezett entitásfelismeréssel és -egyértelműsítéssel. |
| Language Understanding Intelligent Service (LUIS, intelligens hangfelismerési szolgáltatás)| Szöveg | Megtaníthatja alkalmazásait, hogy értsék a felhasználói parancsokat. |
| Kérdések és válaszok készítő szolgáltatás| Szöveg | Gyakran ismételt kérdések formázott adatait conversational, egyszerűen lépjen válaszok átalakítást. |
| Nyelvi elemzési API | Szöveg | Összetett nyelvi fogalmak egyszerűsítése, és a következő szöveg elemzése. |
| Knowledge Exploration szolgáltatás | Szöveg | Strukturált adatokon természetes nyelvi bemenetekkel végezhet interaktív kereséseket. | 
| Webes nyelvi modell API | Szöveg | A webalkalmazás skálázása adatokkal betanítása prediktív nyelvi modellek használata. | 
| Academic Knowledge API | Szöveg | Koppintson a rengeteg hasznos academic tartalom a Microsoft által a Bing Academic Graph be. |
| Bing Autosuggest API | Szöveg | Adja meg az alkalmazás intelligens automatikus kiegészítési beállítások keres. |
| Bing Spell Check API | Szöveg | Észleli, és javítsa ki a helyesen adta-e hibák az alkalmazásban. |
| Translator Text API | Szöveg | Gépi fordítással. |
| Javaslatok API | Szöveg | Előre jelezni, és szeretné, hogy az ügyfelek elemek javasoljuk. |
| Bing Entity Search API | Szöveg (webes keresési lekérdezés) | Határozza meg és kiegészítheti a webes entitás adatait. |
| Bing Képkeresési API | Szöveg (webes keresési lekérdezés) | Keresse meg a képeket. |
| Bing Hírkeresési API | Szöveg (webes keresési lekérdezés) | Keresse meg a híreket. |
| Bing Videókeresési API | Szöveg (webes keresési lekérdezés) | Keresse meg a videók. |
| Bing Webes keresési API | Szöveg (webes keresési lekérdezés) | Kifinomult keresés a webes dokumentumok milliárdjaiban. |.
| Bing Speech API | Szöveg- vagy beszéd | Beszéd szöveggé alakítani, és vissza. |
| Beszélőfelismerés API | Beszéd | Beszéd segítségével azonosíthatja, és egyes hangszórók hitelesítéséhez. |
| Beszédfordító API | Beszéd | Hajtsa végre a valós idejű beszéd fordítását. |
| Computer Vision API | Képet (vagy a videó keretek) | Alkalmas lemezképek végrehajthatóként adatait, automatikusan létrehozása fényképek leírása, címkék származnia, celebrities ismeri fel, bontsa ki a szöveget és pontos miniatűrök létrehozása. |
| Tartalommoderátor | Szöveg, képek vagy videó | Automatizált lemezképet, a szöveg és a videó moderálás. |
| Emotion API | Képek (fényképek a emberi témák) | A tartomány érzelmek emberi témák azonosításához. |
| Face API | Képek (fényképek a emberi témák) | Arcok felismerése, azonosítása, elemzése, rendszerezése és megjelölése a képeken. |
| Videóindexelő | Videó | Véleményeket, Beszélgetés szövegének beszéd átalakítás, például video insights, amelyek beszéd átalakítás, lapokat és érzelmek ismeri fel, és bontsa ki a kulcsszavak. | 

### <a name="trained-with-custom-data-you-provide"></a>A megadott egyéni adatok betanítása
| | A bemeneti típus | Fő előnyt |
| --- | --- | --- |
| Egyéni vizuális szolgáltatás | Képet (vagy a videó keretek) | Saját számítógép stratégiai modellek testreszabása. |
| Egyéni beszédszolgáltatás | Beszéd | Beszéd nevéből felismert tartománynév használatával például a beszéd stílus, háttérzaj és szóhasználatának kapcsolatos. | 
| Egyéni döntési szolgáltatás | Webes tartalmat (például RSS-hírcsatorna) | Használja a gépi tanulási válasszák ki a megfelelő tartalmat a kezdőlap |
| Bing Custom Search API | Szöveg (webes keresési lekérdezés) | Kereskedelmi szintű keresési eszköz. |

