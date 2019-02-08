---
title: 'Nagyszámítógépek migrálása: Tévhitet és tények'
description: A nagyszámítógépes környezetek alkalmazásokat át az Azure-ba, a bevált, magas rendelkezésre állású és méretezhető infrastruktúrát Nagyszámítógépek a jelenleg futó rendszerek.
author: njray
ms.date: 12/27/2018
ms.openlocfilehash: bcad01ec044d2d802b055e328a9496aae7b33311
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899138"
---
# <a name="mainframe-myths-and-facts"></a>A nagyszámítógépes tévhitet és tények

Nagyszámítógépek. ábra ezzel a beállítással hangsúlyosan számítástechnika előzményeit, és magas adott munkaterhelés konkrét működőképes marad. A legtöbb elfogadja, hogy a nagyszámítógépek régóta működési eljárásokkal, amely megbízható és hatékony környezetek, így a már bizonyítottan megbízható platformra legyenek. Szoftver-használat, mérni / másodperc (MIPS), 1 millió utasítások alapján fut, és részletes használati jelentésekben érhetők el ingyenesen készít biztonsági másolatot.

A megbízhatóság, a rendelkezésre állás és a feldolgozási kapacitását a nagyszámítógépek végrehajtása szinte mythical méreteket. A nagyszámítógépes számítási feladatok, amelyek a leginkább megfelelő Azure kiértékelése, először szeretne a tévhitet megkülönböztetni a valóság.

## <a name="myth-mainframes-never-go-down-and-have-a-minimum-of-five-9s-of-availability"></a>Miközben: Nagyszámítógépek soha nem leáll, és legalább öt 9-es rendelkezésre állása

Nagyszámítógépes hardvereket és operációs rendszereket, megbízhatóbb és stabilabb is megtekinthetők. De a valóságban ez a karbantartás és újraindítások (néven a program kezdeti betölti vagy IPLs) kell ütemezni, hogy az állásidő. Ha ezek a feladatok számítanak, a nagyszámítógépes megoldás gyakran van közelebb két vagy három rendelkezésre állást, amely megfelel a csúcskategóriás, Intel-alapú kiszolgálók 9-es.

Nagyszámítógépek is egyéb kiszolgálók do katasztrófák is sebezhető marad, és szünetmentes tápegység (UPS) rendszerek ilyen típusú hibák kezeléséhez szükséges.

## <a name="myth-mainframes-have-limitless-scalability"></a>Miközben: Nagyszámítógépek rendelkezik korlátlan méretezhetőség

A nagyszámítógépes méretezhetőség attól függ, hogy a kapacitás, a rendszer szoftverek, például a vásárlói adatokat vezérlőrendszer (CICS) és a nagyszámítógépes motorok és a tárolás új példányok kapacitását. Egyes nagy vállalatok, amelyek Nagyszámítógépek testreszabott azok CICS a teljesítmény, és rendelkezik egyéb outgrown a legnagyobb rendelkezésre álló Nagyszámítógépek képességét.

## <a name="myth-intel-based-servers-are-not-as-powerful-as-mainframes"></a>Miközben: Intel-alapú kiszolgálók amelyek nem nagy teljesítményűek, mint nagyszámítógépek

Az új core sűrű, Intel-alapú rendszerek sokkal számítási kapacitását Nagyszámítógépek is lehet.

## <a name="myth-the-cloud-cannot-accommodate-mission-critical-applications-for-large-companies-such-as-financial-institutions"></a>Miközben: A felhő nem tud biztosítja az alapvető fontosságú alkalmazásokat nagy vállalatok, például a pénzügyi intézmények számára

Bár előfordulhat, hogy egyes elszigetelt példányai, ha felhőalapú megoldások rövid tartozik, ez általában akkor fordul az alkalmazás algoritmusok nem terjeszthetők. Ezek néhány példa a kivételek, a szabály nem.

## <a name="summary"></a>Összegzés

Ezzel az Azure kínál egy másik platform, amely képes a nagyszámítógépes egyenértékű funkciókat és szolgáltatásokat, és sokkal alacsonyabb költségekkel. Emellett a teljes birtoklási költség (TCO) a felhő-előfizetés-alapú, használat alapú költségek modell sokkal kevésbé költséges, mint a nagygépes is.

## <a name="next-steps"></a>További lépések

> [!div class="nextstepaction"]
> [Győződjön meg arról, a kapcsoló Nagyszámítógépek az Azure-bA](migration-strategies.md)
