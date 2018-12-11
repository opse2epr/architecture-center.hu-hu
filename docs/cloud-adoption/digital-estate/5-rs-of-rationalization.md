---
title: A 5 ésszerűsítés az Rs
titleSuffix: Enterprise Cloud Adoption
description: Ismerteti, amely egy digitális hagyatéki ésszerűsítéséről esetén érhetők el a beállításokat
author: BrianBlanchard
ms.date: 12/10/2018
ms.openlocfilehash: 4e2765198b64c36470adc9fbe35872e4714780e8
ms.sourcegitcommit: e7f8676bbffe500fc4d6deb603b7c0b7ba1884a6
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/10/2018
ms.locfileid: "53179452"
---
# <a name="enterprise-cloud-adoption-the-5-rs-of-rationalization"></a>Enterprise Cloud Adoption: A 5 ésszerűsítés az Rs

Felhőbeli ésszerűsítés az a folyamat kiértékelésének adategységeket az áttelepítés vagy a felhőben minden egyes eszköz modernizálhatja a legjobb módja határozza meg. Ésszerűsítés a folyamattal kapcsolatos további információk: [digitális hagyatéki mi?](overview.md)

Az "5 Rs ésszerűsítés az" felsorolt Itt a ésszerűsítés leggyakrabban használt beállításait ismertetik.

## <a name="rehost"></a>Újratárolás

Más néven "lift- and -shift," egy áthelyezési erőfeszítés helyezi át a jelenlegi állapot az eszközintelligencia a kiválasztott felhőszolgáltató, az általános architektúrát minimális módosítása.

Közös illesztőprogramot lehetnek:

* Csökkentheti a tőkeráfordítás mértékét
* Szabadítson fel lemezterületet a datacenter
* Gyors felhőbeli megtérülés

A mennyiségi elemzés tényezők:

* Virtuálisgép-méret (CPU, memória, tároló)
* Függőségek (hálózati forgalom)
* Az eszközintelligencia-kompatibilitás

Minőségi elemzési tényezők:

* Tolerancia módosítás
* Üzleti prioritásokhoz
* Kritikus fontosságú üzleti eseményeket
* Folyamat függőségek

## <a name="refactor"></a>Újrabontás

Szolgáltatásbeállítások (PaaS) a platformszolgáltatás csökkentheti a működési költségeket számos alkalmazás. Egy alkalmazásnak a PaaS-alapú modell némileg bontani körültekintő lehet.

Az alkalmazás fejlesztési folyamat a kódot, hogy az új üzleti lehetőségek eljuttatható alkalmazások újrabontása újrabontása is hivatkozik.

Közös illesztőprogramot lehetnek:

* Gyorsabb, rövidebb, frissítések
* Kód hordozhatóság
* (Erőforrás, sebességét, költség) nagyobb felhő hatékonyságát.

A mennyiségi elemzés tényezők:

* Az eszközintelligencia mérete (CPU, memória, tároló)
* Függőségek (hálózati forgalom)
* A felhasználói adatforgalmat (ideje az oldalon, lapmegtekintés betöltési idő)
* Fejlesztői platform (nyelveket, adatplatform, középső rétegbeli szolgáltatások)

Minőségi elemzési tényezők:

* Folyamatos üzletmenet, befektetés
* Beállítások/ütemtervek tartalékkapacitás
* Üzleti folyamat függőségek

## <a name="rearchitect"></a>Újratervezés

Egyes elévült alkalmazások nem kompatibilis a felhőszolgáltatók miatt az architekturális döntések, amikor az alkalmazás lett létrehozva. Ezekben az esetekben az alkalmazás szükségessé kell rearchitected átalakítása előtt.

Más esetekben az alkalmazásokat, amelyek a felhő-kompatibilis, de nem felhőbeli natív előnnyel jár, eredményezhet költséghatékonyságot és a működési hatékonyságot átszervezése a natív felhőalkalmazások-megoldás által.

Közös illesztőprogramot lehetnek:

* Alkalmazás méretezhetőség és mozgékonyság
* Új felhőalapú képességek könnyebben elfogadása
* Többféle technológiai csomaggal

A mennyiségi elemzés tényezők:

* Az eszközintelligencia mérete (CPU, memória, tároló)
* Függőségek (hálózati forgalom)
* A felhasználói adatforgalmat (ideje az oldalon, lapmegtekintés betöltési idő)
* Fejlesztői platform (nyelveket, adatplatform, középső rétegbeli szolgáltatások)

Minőségi elemzési tényezők:

* Egyre bővülő üzleti befektetéseit
* A működési költségek
* A lehetséges visszajelzési hurkokat és a DevOps, befektetés

## <a name="rebuild"></a>Újraépítés

Bizonyos esetekben a különbözeti, amely kell meg lehet oldani a átviszi az alkalmazás lehet túl nagy, befektetés további igazoló. Ez különösen igaz a alkalmazásokat, amelyek segítségével az üzleti, de a rendszer most már nem támogatott vagy a helytelen igazítású, hogy az üzleti folyamatok végrehajtása még ma az igényeinek. Ebben az esetben egy új kódbázis egy felhőbeli natív megközelítés igazodva jön létre.

Közös illesztőprogramot lehetnek:

* Gyorsabb innováció
* Gyorsabb alkalmazáskészítés
* Működési költségek csökkentése

A mennyiségi elemzés tényezők:

* Az eszközintelligencia mérete (CPU, memória, tároló)
* Függőségek (hálózati forgalom)
* A felhasználói adatforgalmat (ideje az oldalon, lapmegtekintés betöltési idő)
* Fejlesztői platform (nyelveket, adatplatform, középső rétegbeli szolgáltatások)

Minőségi elemzési tényezők:

* Végfelhasználói elégedettség elutasítása
* Üzleti folyamatok korlátozott funkciók szerint
* A lehetséges díj, a felhasználói élményt és a bevétel nyereség

## <a name="replace"></a>Csere

Megoldások általában a legjobb technológiával és rendelkezésre álló módszer használatával időben vannak megvalósítva. Bizonyos esetekben az üzemeltetett alkalmazás szükséges funkciót összes szoftvert, mint a szoftverszolgáltatások (SaaS) alkalmazások is megfelelnek. Ezekben az esetekben egy számítási feladat sikerült kell nyár kerül piacra jövőbeli kell cserélni, hatékonyan eltávolítaná az átalakítási beavatkozást.

Közös illesztőprogramot lehetnek:

* Iparág-– gyakorlati tanácsok körül szabványosítása
* Gyorsabb üzleti folyamat megközelítések driven elfogadása
* Foglalja le újból a fejlesztési beruházások kiterjeszthetők az alkalmazásokat, amelyek versenyképes differenciálás vagy előnyeit létrehozása

A mennyiségi elemzés tényezők:

* Általános működési költségek csökkentése
* Virtuálisgép-méret (CPU, memória, tároló)
* Függőségek (hálózati forgalom)
* Eszközök megszűnik

Minőségi elemzési tényezők:

* A Cost haszon elemzése a jelenlegi architektúra vs Szolgáltatottszoftver-megoldás
* Üzleti folyamatok térképek
* Adatsémák
* Egyéni vagy automatizált folyamatokkal

## <a name="next-steps"></a>További lépések

Együttesen ezek ésszerűsítés 5 Rs is alkalmazható egy digitális hagyatéki ésszerűsítés minden alkalmazás jövőbeli állapota kapcsolatos döntéseket.

> [!div class="nextstepaction"]
> [Mi az a digial hagyatéki?](overview.md)