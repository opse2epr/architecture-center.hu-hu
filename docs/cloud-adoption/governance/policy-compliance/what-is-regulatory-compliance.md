---
title: 'CAF: Bevezetés a szabályozásoknak való megfelelőséget'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Bevezetés a szabályozásoknak való megfelelőséget
author: BrianBlanchard
ms.openlocfilehash: 3703367dd03b63e5ecf86408ab29dafc7a6e494d
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58298605"
---
# <a name="introduction-to-regulatory-compliance"></a>Bevezetés a szabályozásoknak való megfelelőséget

Ez egy bevezető cikk kapcsolatos szabályozásoknak való megfelelőséget, ezért nem célja egy megfelelőségi stratégia megvalósításához. Csak az általános tájékoztatási feladata. Vonatkozó részletes információkért [az Azure megfelelőségi ajánlatainak](https://aka.ms/allcompliance) érhető el, a [Microsoft Trust Center]. Továbbá minden letölthető dokumentáció érhető el az Azure-ügyfelek egy titoktartási szerződés a a [Microsoft Szolgáltatásmegbízhatósági portált](https://servicetrust.microsoft.com/).

A jogszabályoknak való megfelelőség a szakágankénti és a folyamatkiszolgáló annak biztosítására, hogy a vállalat a következő földrajzi vagy iparági szervei által beállított törvényi hivatkozik. Az informatikai szabályozásoknak való megfelelőséget, személyek és/vagy folyamatok vállalati rendszerek figyeléséhez annak érdekében, hogy észleléséhez és megelőzéséhez megsértésének házirendekkel és eljárásokkal állítja be a vonatkozó törvényeknek és szabályozásoknak. Ez pedig egy nagyon nagy kiterjedésű felügyeletéhez és folyamatok vonatkozik. Attól függően, az iparági és földrajzi hely ezeket a folyamatokat nagyon hosszú és összetett válhat.

A multinacionális szervezet számára (különösen az erősen szabályozott iparágakban, például az egészségügyi és pénzügyi szolgáltatóknak) megfelelőségi nagy kihívást jelenthet. Abound szabványoknak és előírásoknak, és természetesen változnak gyakran, megnehezítve a vállalkozások számára, hogy a folyamatosan fejlődő nemzetközi elektronikus adatkezelési törvényi témaköreit.

Biztonsági vezérlők, a szervezetek ismerje meg az osztálynak a feladatkörök kapcsolatos szabályozásoknak való megfelelőséget, a felhőben. A felhőszolgáltatók arról, hogy a platformok és a szolgáltatások megfelelő törekedni. De szervezetek is kell győződjön meg arról, hogy alkalmazásaikat, és a harmadik fél által nyújtott megfelelnek. Szabályozott iparágakban, felhőszolgáltatásokat használó alkalmazások hasonló módon a felhőszolgáltató a minősítési lehet szükség.

Számos iparágból és földrajzi területről megfelelőségi szabályzat leírását a következők:

## <a name="hipaa"></a>HIPAA

A egészségügyi alkalmazás, hogy a folyamatok védett egészségügyi információ (PHI)-e az adatvédelmi szabály és a biztonsági szabály vonatkozik az egészségügyi információk Portability and Accountability Act (HIPAA) belül egyaránt érvényesek. Minimális HIPAA valószínűleg igényel, hogy egy egészségügyi üzleti kap, amely fog megakadályozhatja a bármely fogadott vagy létrehozott Phi-k a felhőbeli szolgáltató által írt biztosítékok.

## <a name="pci"></a>PCI

Fizetési Card Industry Data Security Standard (PCI DSS) egy védett információkat biztonsági szabvány, a szervezet számára, amelyek kezelik termékeire, a fő kártya rendszerek, beleértve a Visa, Mastercard, American Express, felderítési és JCB hitelkártya. A PCI szabvány a kártya márkái által megbízott, és a fizetési kártya iparági biztonsági szabványok Tanácsa felügyeli. A standard kártyabirtokosok adatok csökkentése érdekében a hitelkártyacsalást-vezérlők növelése érdekében jött létre. Megfelelőségi érvényesség évente, vagy által egy külső minősített biztonsági kockázatokat (DSS) foglalt követelményeknek, vagy által egy cég jellemző belső biztonsági kockázatokat (ISA) akik kezelése a tranzakciók, nagy mennyiségű szervezetek számára a megfelelőség (ROC) létrehoz egy jelentést, vagy által egy önértékelés kérdőív (SAQ) vállalatok számára.

## <a name="pii"></a>PII

Személyes azonosításra alkalmas adatok (PII) minden olyan datapoint egy fogyasztói, alkalmazott, partner vagy bármely más élő vagy jogi személy azonosítására használható. Számos új azokat, különösen a foglalkoznak, adatvédelem és az egyes személyazonosításra alkalmas adatok van szükség, hogy maguk vállalkozások felel meg, és a megfelelőség és a bármely felmerülő problémák jelentésére.

## <a name="gdpr"></a>GDPR

Ez a terület legfontosabb végbement egyik, a legutóbbi hatálybalépésével tovább bővült az Európai Bizottság, az általános adatvédelmi rendelet (GDPR), megerősítése az Európai Unióban egyéni felhasználók számára az adatok védelmére terveztek. Általános adatvédelmi rendelet igényli az egyéni felhasználók számára vonatkozó adatokat (például "nevét, egy otthoni cím, egy fényképet, egy e-mail-címet, banki részletei, közösségi hálózati webhelyek, orvosi adatok vagy a számítógép IP-cím bejegyzések") megőrzi az EU-kiszolgálók és a ki nem viszi át azt. Vállalatok felhasználók bármilyen adatokat adatszivárgás értesítést kap, és, amely a vállalatok megbízások rendelkezik egy adatvédelmi tisztviselő is szükséges. Más országok van, vagy fejleszt szabályzat hasonló típusú.

## <a name="compliant-foundation-in-azure"></a>Az Azure-ban alapot

Annak érdekében, felel meg a saját megfelelőségi kötelezettségeket szabályozott iparágakban és piacon világszerte ügyfelek, Azure fenntartja az ágazat legnagyobb megfelelőségi portfóliójával &mdash; technológiai spektrumunk kihasználtságának növelését (ajánlatok teljes száma), valamint mélysége (száma ügyfelek által használt szolgáltatások értékelés hatókörébe). Az Azure megfelelőségi ajánlatainak négy szegmensek szerint vannak csoportosítva: globálisan érvényes US Government, iparág-specifikus, és a régió-/ országspecifikus.

Az Azure megfelelőségi ajánlatainak a különböző típusú biztosítékok, beleértve a hivatalos minősítései, tanúsítványok, ellenőrzés, engedélyek és értékelések független, külső naplózási cégek, valamint szerződéses módosítások által előállított alapuló önértékelésekben, és a Microsoft által gyártott felhasználói útmutatót. Minden ajánlat leírása a jelen dokumentum egy naprakész scope utasítás, amely azt jelzi, mely Azure-ügyfelek által használt szolgáltatások terjed ki az értékelés, valamint a letölthető, amellyel a felhasználók a saját megfelelőségi forrásokra mutató hivatkozásokat biztosít kötelezettségek.

További információk az Azure megfelelőségi ajánlatainak érhető el a [Microsoft Trust Center](/trustcenter/compliance/complianceofferings). Továbbá minden letölthető dokumentáció érhető el az Azure-ügyfelek egy titoktartási szerződés a a [Szolgáltatásmegbízhatósági portálon](https://servicetrust.microsoft.com) a következő szakaszokban:

* Naplózási jelentések: FedRAMP, GRC assessment, ISO, PCI DSS és SOC jelentések szakaszokat tartalmazza
* Adatforrások védelmét: Tartalmazza a megfelelőségi útmutatók, – gyakori kérdések és tanulmányok és toll tesztelése és a biztonsági értékelések szakasz
