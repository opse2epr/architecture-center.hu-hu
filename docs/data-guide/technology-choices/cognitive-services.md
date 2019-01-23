---
title: A cognitive services technológia kiválasztása
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: AI
ms.openlocfilehash: e8bbdf4874d5fc669e1c08add8cf212217d176a4
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54482646"
---
# <a name="choosing-a-microsoft-cognitive-services-technology"></a>A Microsoft cognitive services technológia kiválasztása

A Microsoft cognitive services olyan felhőalapú API-k használható a mesterséges intelligenciát (AI) használó alkalmazások és adatok elkezdenek beérkezni. Megadják Önnek imagenet modellek, amelyek az adatok nem és az Ön részéről nincs modell betanítása igénylő alkalmazás használatra kész. A Microsoft által fejlesztett a cognitive services mesterséges Intelligencia és kutatási csapata, és kihasználhatja a legújabb deep learning-algoritmusok. Ezek a HTTP REST-felületeihez keresztül használjuk fel. Emellett az SDK-k számos gyakori alkalmazási fejlesztési keretrendszerek érhető el.

A cognitive services a következők:

- Szövegelemzés
- Számítógépes látástechnológia
- Videóelemzés
- Beszédfelismerés, beszédfelismerési és beszédgenerálási
- Beszédfelismerés
- Intelligens keresést

Fő előnyök:

- Minimális fejlesztési tevékenységi állapot-az-a legújabb AI-szolgáltatások számára.
- Könnyen integrálható az alkalmazásokba via HTTP REST-felületeihez.
- Beépített támogatást nyújt, a cognitive Servicest az Azure Data Lake Analytics felhasználása.

Szempontok:

- Csak a weben keresztül érhető el. Internetkapcsolat általában szükség. Egy kivétel ez alól a Custom Vision Service, amelynek segítségével exportálhatja az előrejelzéshez az eszközökön és az IoT edge betanított modell.

- Jelentős testreszabási támogatják, az elérhető szolgáltatások előfordulhat, hogy nem kockázatiszint összes prediktív elemzési.

<!-- markdownlint-disable MD026 -->

## <a name="what-are-your-options-when-choosing-amongst-the-cognitive-services"></a>Mik azok a beállítások többek között a cognitive services kiválasztásakor?

<!-- markdownlint-disable MD026 -->

Az Azure-ban érhetők el a Cognitive Services tucat. Ezek jelenlegi listája támogatja a funkcionális terület szerint osztályozva könyvtár érhető el:

- [Vision](https://azure.microsoft.com/services/cognitive-services/directory/vision/)
- [Beszédfelismerés](https://azure.microsoft.com/services/cognitive-services/directory/speech/)
- [Knowledge](https://azure.microsoft.com/services/cognitive-services/directory/know/)
- [Search](https://azure.microsoft.com/services/cognitive-services/directory/search/)
- [Nyelv](https://azure.microsoft.com/services/cognitive-services/directory/lang/)

## <a name="key-selection-criteria"></a>Fontosabb kritériumok

Így szűkítheti, első lépésként a kérdések megválaszolása:

- Milyen típusú adatokat, foglalkoznak? Szűkítenie a lehetőségeit, dolgozik, a bemeneti adatok típusa alapján. Ha a bemeneti szöveg, például válassza ki a szolgáltatásokból, amelyek egy bemeneti szöveg típusát.

- Rendelkezik a modell betanításához az adatokat? Ha igen, fontolja meg az egyéni szolgáltatásokról, amelyek lehetővé teszik az adatokat ad meg, a nagyobb pontosság és a teljesítmény az alapul szolgáló modelleket taníthat be.

## <a name="capability-matrix"></a>Képességmátrix

A következő táblázat összefoglalja a fő különbségeket, a képességek.

### <a name="uses-prebuilt-models"></a>Használja az előre összeállított modellek

|                                                   |             Bemenet típusa              |                                                                                Legfontosabb előnyök                                                                                |
|---------------------------------------------------|-------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                Szövegelemzések API                 |                Szöveg                 |                                                       Kiértékelheti a közhangulatot és különböző témákat, hogy a felhasználók szándékainak megértésére.                                                        |
|                Entitáskapcsolási API                 |                Szöveg                 |                                               Tegye hatékonyabbá alkalmazása adatkapcsolatait elnevezett entitásfelismeréssel és -egyértelműsítéssel.                                               |
| Language Understanding Intelligent Service (LUIS) |                Szöveg                 |                                                          Megtaníthatja az alkalmazásokat arra, hogy értelmezzék a felhasználók parancsait.                                                          |
|                 A QnA Maker szolgáltatás                 |                Szöveg                 |                                             Formázott – gyakori kérdések információkat, könnyen átlátható válaszokat nyerhet ki.                                              |
|              Linguistic Analysis API              |                Szöveg                 |                                                            Egyszerűsítheti a bonyolult nyelvi szerkezeteket és szövegelemzést.                                                             |
|           Knowledge Exploration szolgáltatás           |                Szöveg                 |                                          Strukturált adatokon természetes nyelvi bemenetekkel végezhet interaktív kereséseket.                                          |
|              Web Language Model API               |                Szöveg                 |                                                         Használja a webes méretezésű adatokkal tanított prediktív nyelvi modellek.                                                         |
|              Academic Knowledge API               |                Szöveg                 |                                        Koppintson a tudományos tartalmát, a Bing által a Microsoft Academic Graph gazdag.                                         |
|               Bing Autosuggest API                |                Szöveg                 |                                                        Adja meg az alkalmazás intelligens automatikus kiegészítési lehetőségeket a keres.                                                        |
|               Bing Spell Check API                |                Szöveg                 |                                                             Észlelheti és kijavíthatja a helyesírási hibákat az alkalmazásban.                                                             |
|                Translator Text API                |                Szöveg                 |                                                                           Gépi fordítás.                                                                            |
|                Javaslatok API                |                Szöveg                 |                                                             Előre jelezni, és javasolhat bizonyosan.                                                              |
|              Bing Entity Search API               |       Szöveg (webes keresési lekérdezés)       |                                                           Azonosítsa, és az Entitásadatok webes bővítésével.                                                           |
|               Bing Image Search API               |       Szöveg (webes keresési lekérdezés)       |                                                                            Kereshet a lemezképeket.                                                                             |
|               Bing News Search API                |       Szöveg (webes keresési lekérdezés)       |                                                                             Hírek keresése.                                                                              |
|               Bing Video Search API               |       Szöveg (webes keresési lekérdezés)       |                                                                            Videók keresése.                                                                             |
|                Bing Web Search API                |       Szöveg (webes keresési lekérdezés)       |                                                        Kifinomult keresés a webes dokumentumok milliárdjaiban.                                                        |
|                  Bing Speech API                  |           Szöveg és beszéd            |                                                                  Beszéddé alakíthatja az írott szöveget, majd vissza.                                                                   |
|              Beszélőfelismerés API              |               Beszédfelismerés                |                                                       Beszéd használata azonosíthatja és hitelesítheti az egyes beszélőket.                                                        |
|               Translator Speech API               |               Beszédfelismerés                |                                                                   Hajtsa végre a valós idejű beszédfordítás.                                                                   |
|                Computer Vision API                |    Képek (vagy a videó keretek)    | Rendszerképekből döntéstámogató információkat nyerhet ki, automatikusan hozzon létre fényképek leírása, címkék célosztályából, hírességek, szöveg kinyerése és pontos miniatűrök létrehozása. |
|                 Content Moderator                 |        Szöveg, képek vagy videó        |                                                               Automatizált kép, szöveg és videomoderálás.                                                                |
|                    Emotion API                    | Képek (fényképek az emberi témák) |                                                              Az emberek alkalmazását a tartomány érzelmeket azonosít.                                                               |
|                     Arcfelismerési API                      | Képek (fényképek az emberi témák) |                                                       Észlelheti, azonosíthatja, elemezheti, rendezheti és címkézheti az arcokat a fényképeken.                                                       |
|                   Video Indexer                   |                Videó                |                        Videóelemzések hangulatát, átirat-beszéd átalakítás, például fordítása speech, arcokat és érzelmeket azonosíthat ismeri fel és kulcsszavakkal.                         |

### <a name="trained-with-custom-data-you-provide"></a>A megadott egyéni adatok betanított

| | Bemenet típusa | Legfontosabb előnyök |
| --- | --- | --- |
| Egyéni vizuális szolgáltatás | Képek (vagy a videó keretek) | Testre szabhatja a saját számítógépes látástechnológiai modelljeit. |
| Custom Speech Service | Beszédfelismerés | Kiküszöbölheti a beszédfelismerést akadályozó tényezők például a különféle stílusú, háttérzajból és szókincsből eredőket. |
| Egyéni döntési szolgáltatás | Webes tartalom (például az RSS-hírcsatorna) | Machine learning segítségével automatikusan kiválasztja a megfelelő tartalmat a kezdőlap |
| Bing Custom Search API | Szöveg (webes keresési lekérdezés) | Kereskedelmi színvonalú keresési eszköz egyéni keresési. |
