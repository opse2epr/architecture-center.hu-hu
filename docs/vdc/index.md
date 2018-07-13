---
title: Azure Virtual Datacenter
description: A Microsoft Azure Virtual Datacenter lehetséges erőforrásai
keywords: Azure
layout: LandingPage
ms.openlocfilehash: e37ac347247b2960b85832a20a2b57eec18f65a7
ms.sourcegitcommit: 776b8c1efc662d42273a33de3b82ec69e3cd80c5
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/12/2018
ms.locfileid: "38987512"
---
# <a name="azure-virtual-datacenter-and-the-enterprise-control-plane"></a>Az Azure Virtual Datacenter és a vállalati vezérlősík

Az Azure Virtual Datacenter arra szolgál, hogy a lehető legtöbbet hozza ki az Azure-felhőplatform képességeiből a meglévő biztonsági és hálózatkezelési szabályzatok betartása mellett. Amikor vállalati számítási feladatokat helyeznek üzembe a felhőben, az informatikai cégeknek és üzleti egységeknek meg kell találniuk a szabályozás és a fejlesztőknek nyújtott rugalmasság közötti egyensúlyt. Az Azure Virtual Datacenter különböző modelleket kínál ennek kialakítására, a szabályozásra helyezve nagyobb hangsúlyt.
 
## <a name="resources"></a>További források
<table>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="http://aka.ms/VDC/Concepts"><img src="../_images/virtual-datacenter.svg" alt="Virtual Datacenter eBook" /></a></td>
    <td>
        <h3><a href="http://aka.ms/VDC/Concepts">Azure Virtual Datacenter: Alapfogalmak</a></h3>
        <p>Ez az e-könyv bemutatja, hogyan helyezhet üzembe vállalati számítási feladatokat az Azure-felhőplatformon a meglévő biztonsági és hálózatkezelési szabályzatok betartása mellett.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="/azure/networking/networking-virtual-datacenter"><img src="./images/vdc-network.png" alt="Network Perspective" /></a></td>
    <td>
        <h3><a href="/azure/networking/networking-virtual-datacenter">Azure Virtual Datacenter: A hálózati nézőpont</a></h3>
        <p>Ez az online cikk áttekintést nyújt a hálózati mintákról és kialakításokról, amelyekkel megoldhatók az architektúra-méretezéssel, teljesítménnyel és biztonsággal kapcsolatos, a felhőbe való tömeges áthelyezés során felmerülő problémák.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="http://aka.ms/VDC/Lift"><img src="./images/vdc-lift-and-shift.png" alt="Lift and Shift Guide" /></a></td>
    <td>
        <h3><a href="http://aka.ms/VDC/Lift">Azure Virtual Datacenter: Átemelési útmutató </a></h3>
        <p>Ez a tanulmány bemutatja a folyamatot, amellyel a vállalat informatikai munkatársai és döntéshozói azonosíthatják az Azure-ba migrálni kívánt alkalmazásokat és kiszolgálókat, majd végrehajthatják a migrálást az átemelési módszerrel, amellyel minimalizálható a további szükséges fejlesztési költség, valamint és optimalizálhatók a felhőalapú üzemeltetési lehetőségek.</p>
    </td>
</tr>
<tr>
    <td style="width: 64px; vertical-align: middle;"><a href="http://aka.ms/VDC/Deck"><img src="./images/vdc-deck.png" alt="Presentation Deck" /></a></td>
    <td>
        <h3><a href="http://aka.ms/VDC/Deck">Virtual Datacenter: Prezentáció </a></h3>
        <p>Ez a prezentáció útmutatást ad az Azure virtuális adatközpont használatához, és bemutatja az eszközeit. Sorra veszi a virtuális adatközpontok céljait, az ügyfélszempontokat, az Azure-régiókat, a virtuális adatközpontok automatizálásának elemeit, az ipari és megbízható Azure virtuális adatközpontok listáját, végül pedig egy CIO-útmutatásra épülő akciótervet. Támogatási és képzési információkat is tartalmaz.</p>
    </td>
</tr>
</table>

## <a name="what-is-the-azure-virtual-datacenter"></a>Mi az Azure Virtual Datacenter?

A számítási feladatok felhőbeli üzembe helyezése olyan fokú megbízhatóság kialakítását és fenntartását igényli, mint amilyen a meglévő adatközpontjait jellemzi. Az Azure Virtual Datacenter útmutatójának első modellje ezt az igényt a virtuális infrastruktúra zárolásával éri el. Ez a megközelítés nem alkalmas mindenki számára. Kifejezetten a vállalati informatikai csoportoknak nyújt útmutatást, amelyeknek a helyszíni infrastruktúrát kell bővíteniük az Azure-beli nyilvános felhőben. Ezt a megközelítést a megbízható adatközpont kiterjesztésének nevezzük. Később számos további modell is elérhető lesz, többek között olyanok is, amelyek biztonságos internet-hozzáférést kínálnak közvetlenül a virtuális adatközpontból.

<img src="./images/vdc-components.svg" alt="Virtual Datacenter components" style="max-width:700px;"/>

Az Azure Virtual Datacenter működését a következő négy összetevő teszi lehetővé: identitás, titkosítás, szoftveralapú hálózatkezelés és megfelelőség (beleértve a naplókat és a jelentéskészítést).

Az Azure virtuálisadatközpont-modellben alkalmazhat elkülönítési szabályzatokat, a jól ismert fizikai adatközpontokhoz hasonlóbbá teheti a felhőt, továbbá elérheti a szükséges mértékű biztonságot és bizalmat. Ezt négy összetevő teszi lehetővé, amelyek minden vállalati informatikai csapat számára ismerősek: szoftveralapú hálózatkezelés, titkosítás, identitáskezelés és az Azure platform alapul szolgáló megfelelőségi szabványai és tanúsítványai. Ezek szükségesek ahhoz, hogy a virtuális adatközpont a meglévő infrastruktúra megbízható kiterjesztésévé váljon.


Olvassa tovább az <a href="http://aka.ms/VDC/eBook">Azure Virtual Datacenter: Alapfogalmak</a> című e-könyvet.
