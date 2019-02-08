---
title: 'CAF: Központi telepítési gyorsítás eszközök az Azure-ban'
description: Központi telepítési gyorsítás eszközök az Azure-ban
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
author: BrianBlanchard
ms.openlocfilehash: 8d2cae87b8a3a11bfde87aafc04d4d2ef55cbbe9
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899358"
---
# <a name="deployment-acceleration-tools-in-azure"></a>Központi telepítési gyorsítás eszközök az Azure-ban

[Üzembe helyezés gyorsítás](overview.md) egyike a [öt oktatnak, felhőalapú Cégirányítási](../governance-disciplines.md). Ez fegyelem módját a házirendeket szabályozó eszköz konfigurációs vagy telepítési összpontosít. Felhőalapú irányítási öt állapítják konfigurációs cégirányítási telepítési, konfigurációs igazítás és magas rendelkezésre ÁLLÁS/Vészhelyreállítás stratégiák magában foglalja. Ez lehet manuális tevékenységek vagy teljesen automatizált fejlesztési és üzemeltetési tevékenységeket. Mindkét esetben a házirendek lenne nagyjából ugyanazok maradnak.

Felhőalapú jegyektől, a felhő őrei és a felhőben dolgozó tervezők érdekeltek cégirányítási egyes várhatóan sok időt a a telepítési gyorsítás szabályok betartását, amely egységes irányelvek és követelmények között több felhő bevezetését erőfeszítések befektetni. Az eszközök a eszközláncban fontosak a felhő Cégirányítási csapatának, és kell lennie a képzési terv a csapat a kiemelt fontosságú.

Az Azure-eszközök, amelyek segíthetnek a házirendek és a folyamatok, amelyek támogatják a cégirányítási fegyelem részletes listáját a következő:

|  |Azure Policy  |Azure-beli felügyeleti csoportok  |Az Azure Resource Manager-sablonok  |Azure Blueprints  | Azure Resource Graph | Azure Cost Management |
|---------|---------|---------|---------|---------|---------|---------|
|Vállalati házirendek megvalósítása     |Igen |Nem  |Nem  |Nem | Nem |Nem |
|Szabályzatok alkalmazása az előfizetések között     |Szükséges |Igen  |Nem  |Nem | Nem |Nem |
|Meghatározott erőforrások üzembe helyezése     |Nem |Nem  |Igen  |Nem | Nem |Nem |
|Teljes mértékben megfelelővé vált környezetek létrehozása      |Szükséges |Szükséges  |Szükséges  |Igen | Nem |Nem |
|Naplózási házirend      |Igen |Nem  |Nem  |Nem | Nem |Nem |
|Az Azure-erőforrások lekérdezése      |Nem |Nem  |Nem  |Nem |Igen |Nem |
|Jelentés az erőforrások költségét      |Nem |Nem  |Nem  |Nem |Nem |Igen |

Az alábbiakban meghatározott telepítési gyorsítás célok megvalósításához esetlegesen szükséges további eszközöket. Gyakran ezekkel az eszközökkel a cégirányítási csapat kívül használt, de továbbra is egy szakterületi telepítési gyorsulás aspektusaihoz számít.

|  |Azure Portal  |Az Azure Resource Manager-sablonok  |Azure Policy  | Azure DevOps | Azure Backup | Azure Site Recovery |
|---------|---------|---------|---------|---------|---------|---------|
|Kézi telepítés (egyetlen eszköz)     | Igen | Igen  | Nem  | Nem hatékony | Nem | Igen |
|Manuális telepítést (a teljes környezet)     | Nem hatékony | Igen | Nem  | Nem hatékony | Nem | Igen |
|Automatizált üzembehelyezési (a teljes környezet)     | Nem  | Igen  | Nem  | Igen  | Nem | Igen |
|Egyetlen eszköz konfigurációjának frissítése     | Igen | Igen | Nem hatékony | Nem hatékony | Nem | Igen – a replikáció során |
|A teljes környezet konfigurációjának frissítése     | Nem hatékony | Igen | Igen | Igen  | Nem | Igen – a replikáció során |
|Konfigurációs csúszásokat kezelése     | Nem hatékony | Nem hatékony | Igen  | Igen  | Nem | Igen – a replikáció során |
|Kód üzembe helyezése és konfigurálása az eszközök (DevOps) egy automatikus folyamat létrehozása     | Nem | Nem | Nem | Igen | Nem | Nem |
|Adatok helyreállítása leállás vagy szolgáltatói szerződés megsértéséig     | Nem | Nem | Nem | Igen | Igen | Igen |
|Alkalmazások és adatok helyreállítása leállás vagy szolgáltatói szerződés megsértéséig     | Nem | Nem | Nem | Igen | Nem | Igen |

Azokat az Azure natív eszközei a fent említett, kivéve a harmadik felektől származó eszközök használatuk megkönnyítése érdekében az üzembe helyezési Acceleration és DevOps-telepítések gyakori.
