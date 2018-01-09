---
title: "Háttérkiszolgálókon Frontends minta"
description: "Elkülönített, adott előtérbeli alkalmazások vagy felületek által használt háttérszolgáltatásokat hozhat létre."
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 87acd39d021c5e44594a2e7c9574e4dd363ce83b
ms.sourcegitcommit: c93f1b210b3deff17cc969fb66133bc6399cfd10
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/05/2018
---
# <a name="backends-for-frontends-pattern"></a>Háttérkiszolgálókon Frontends minta

Elkülönített, adott előtérbeli alkalmazások vagy felületek által használt háttérszolgáltatásokat hozhat létre. Ebben a mintában akkor hasznos, ha el szeretné kerülni, több kapcsolathoz egyetlen háttérkiszolgálón testreszabása. Ebben a mintában Sam Newman először le lett.

## <a name="context-and-problem"></a>A környezetben, és probléma

Egy alkalmazás kezdetben irányuló egy asztali webes felhasználói felület. Általában háttérszolgáltatás fejlesztése párhuzamosan, amely az adott felhasználói felület szükséges szolgáltatásokat biztosítja. Az alkalmazás felhasználói bázis növekedésével mobilalkalmazás, amelyek fejlett ugyanazt a háttér együtt kell működnie. A háttérszolgáltatáshoz egy általános célú háttér, az asztali és mobil felületek követelményeinek szolgáltató lesz.

Azonban a képességei a mobileszközök különböznek jelentősen asztali a feltételek mérete, a böngésző teljesítménye, és korlátozások megjelenítéséhez. Ennek eredményeképpen a mobilalkalmazás-háttérrendszer követelményei eltérnek a az asztali webes felhasználói felület. 

Ezek a különbségek a háttérkiszolgáló versengő követelmények eredményez. A háttérrendszer mindkét az asztal kiszolgálására rendszeres és jelentős módosítását igényli az webes felhasználói Felületet és a mobilalkalmazás. Gyakran külön felület csapatok meg, amely a háttérkiszolgálón legyen, a szűk keresztmetszetek a fejlesztési folyamat minden egyes előtér. Ütköző frissítésekre vonatkozó követelményeket, és hogy a szolgáltatás működik-e mindkét frontends eredményezhet nagy munka költségeik központilag telepíthető egy erőforráson.

![](./_images/backend-for-frontend.png) 

A fejlesztési tevékenység a háttérszolgáltatáshoz összpontosít, mert egy külön csapat kezeli és tartja karban a háttérrendszer lehet létrehozni. Végül az eredmény a kapcsolata megszakad a kapcsolat és a háttérkiszolgáló a fejlesztési csapat, terhet helyezi a háttér-csoport a felhasználói felület különböző csapatok versengő követelményeinek egyenleg között. Ha a háttérkiszolgáló módosítását igényli az egy illesztőfelület-csoportot, ezeket a módosításokat össze kell vetni más felület csapatok ahhoz, azok integrálhatók a háttérkiszolgálón. 

## <a name="solution"></a>Megoldás

Hozzon létre egy háttér egy felhasználói felületet. Konfigurálva finomhangolhatják a viselkedést és az előtérbeli környezet igényeinek leginkább megfelelő anélkül, hogy más előtér lép érintő háttérkészletek teljesítménye.

![](./_images/backend-for-frontend-example.png) 

Mivel több olyan illesztőt háttérkészletek, azt is lehet optimalizálni kapcsolat. Ennek eredményeképpen lesz kisebb, kevésbé összetett, és valószínűleg gyorsabb, mint az általános háttérkiszolgálón, amely megfelel az összes illesztő megkísérli. Minden felületet csoport rendelkezik saját háttér vezérlésére önállóan, és nem függ egy központosított háttérkiszolgálón fejlesztői csapat a. A nyelv kiválasztása kiadás ütemben történik, alkalmazások és szolgáltatások és szolgáltatások integrációja a a háttérbeli rangsorolási felület csapata rugalmasan segítségével.

További információkért lásd: [mintát: háttérkiszolgálókon a Frontends](http://samnewman.io/patterns/architectural/bff/).

## <a name="issues-and-considerations"></a>Problémákat és szempontok

- Fontolja meg a telepítendő hány háttérkiszolgálókon.
- Ha különböző felületekhez (mint például a mobil ügyfelek) az azonos kéréseket fog, fontolja meg a legyen az végre kell hajtani a háttérkiszolgálón minden egyes, vagy ha elegendő egyetlen háttérkiszolgálón.
- Kód ismétlődést szolgáltatásban nagyon valószínű, ebben a mintában végrehajtása során.
- Előtér-arra irányul, háttér szolgáltatások csak kell tartalmaznia, ügyfél-specifikus logika és viselkedését. Általános üzleti logika és más globális szolgáltatások kezelt máshol az alkalmazásban.
- Gondolja át hogyan ebben a mintában előfordulhat, hogy megjelennek a fejlesztői csapat feladatait.
- Vegye figyelembe, hogy mennyi ideig tart ebben a mintában végrehajtásához. Részéről az erőfeszítés, az új háttérkiszolgálókon kialakításának gyakorisága műszaki hitelviszonyt, miközben Ön folytatja a meglévő általános háttér támogatásához?

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Ez mintát, mikor használja:

- A megosztott vagy általános célú háttérszolgáltatást fenn kell tartani, fejlesztési jelentős terhelés mellett.
- A háttérkiszolgáló az adott ügyfél felületek követelményeinek optimalizálni szeretné.
- Testreszabások több felületek olyan általános célú háttérkiszolgálón történik.
- Másik nyelvet jobban egy másik felhasználói felület a háttérkiszolgáló olyan környezethez a legalkalmasabb.

Ez a minta nem alkalmasak lehetnek:

- Ha felületek kéréseket az ugyanazon vagy hasonló háttérkiszolgálóra.
- Csak egy kapcsolat használatakor a háttérrendszer kommunikál.

## <a name="related-guidance"></a>Kapcsolódó útmutatók

- [Átjáró összesítési minta](./gateway-aggregation.md)
- [Átjáró Offloading minta](./gateway-offloading.md)
- [Átjáró útválasztás minta](./gateway-routing.md)


