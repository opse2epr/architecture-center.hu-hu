---
title: Tervezés a változás szolgálatában
description: A fejlődést szem előtt tartó tervezés kulcsfontosságú a folyamatos innováció szempontjából.
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: bbd5699e257663514cf7bb8b856fe35f51799c73
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 08/31/2018
ms.locfileid: "43325706"
---
# <a name="design-for-evolution"></a>Tervezzen a fejlődést szem előtt tartva

## <a name="an-evolutionary-design-is-key-for-continuous-innovation"></a>A fejlődést szem előtt tartó tervezés kulcsfontosságú a folyamatos innováció szempontjából

Idővel minden sikeres alkalmazás változásokon esik át, aminek például hibajavítás, funkcióbővítés, új technológiák bevezetése vagy a meglévő rendszerek méretezhetőbbé és rugalmasabbá tétele lehet az oka. Ha egy alkalmazás minden része szoros kapcsolatban áll egymással, az nagyon nehézzé teszi a rendszer módosítását. Az alkalmazás egyik részének a módosítása működésképtelenné tehet egy másik részt, vagy a teljes kódbázist érintő változásokkal járhat.

Ez a probléma nem korlátozódik a monolitikus alkalmazásokra. Egy alkalmazás, hiába választható szét különböző szolgáltatásokra, továbbra is mutathatja azt a fajta szoros összekapcsolódást, ami merevvé és törékennyé teszi a rendszert. Ha azonban a szolgáltatásokat a fejlődés szem előtt tartásával tervezik, a csapatok előállhatnak innovatív megoldásokkal, és folyamatosan gondoskodhatnak az új funkciók bevezetéséről. 

A mikroszolgáltatások a fejlődésközpontú tervezés megvalósításának egyre népszerűbb módjává válnak, mert választ nyújtanak az itt említett problémák többségére.

## <a name="recommendations"></a>Javaslatok

**Törekvés a nagyfokú kohézióra és a laza kapcsolódásokra**. A szolgáltatás akkor jellemezhető *kohézióval*, ha logikailag összetartozó funkciókat biztosít. A szolgáltatások akkor *kapcsolódnak lazán*, ha az egyik szolgáltatás módosítása nem jár együtt egy másik módosításával. A nagyfokú kohézió általában azt eredményezi, hogy egy funkció változtatásai a többi kapcsolódó funkció módosítását is szükségessé teszik. Ha úgy látja, hogy egy szolgáltatás frissítése a többi szolgáltatás összehangolt frissítését kívánja meg, az annak a jele lehet, hogy nincs kohézió a szolgáltatások között. A célja a tartományvezérelt tervezési (DDD) egyik ezeken a határokon belül azonosításához.

**Környezettel kapcsolatos ismeretek beépítése**. Amikor egy ügyfél egy szolgáltatást használ, az adott környezetre (tartományra, területre) vonatkozó üzleti szabályok érvényesítésének felelőssége nem hárulhat az ügyfélre. Ehelyett a szolgáltatásnak kell magában foglalnia a környezettel kapcsolatos mindazon ismereteket, amelyek a felelősségi körébe tartoznak. Ennek hiányában minden egyes ügyfélnek érvényesítenie kell az üzleti szabályokat, aminek az lesz az eredménye, hogy a környezettel kapcsolatos ismeretek szétoszlanak az alkalmazás különböző részei között. 

**Aszinkron üzenetkezelés használata**. Az aszinkron üzenetkezelés lehetővé teszi az üzenet előállítójának elkülönítését az üzenet feldolgozójától. Az előállító nem függ az üzenetre válaszoló vagy adott műveletet végrehajtó feldolgozótól. Közzétevői/előfizetői architektúra esetén az előállító adott esetben azt sem fogja tudni, hogy ki az üzenet felhasználója. Az új szolgáltatások egyszerűen használhatják fel az üzeneteket, az előállító bármiféle módosítása nélkül.

**A környezettel kapcsolatos ismeretek átjáróba való beépítésének mellőzése**. Az átjárók hasznosak lehetnek a mikroszolgáltatásokra épülő architektúrában, ha útválasztásról, protokollfordításról, terheléselosztásról vagy hitelesítésről van szó. Az átjárókat azonban korlátozni kell az ilyen jellegű infrastrukturális funkciók ellátására. Nem foglalhatnak magukban a környezettel kapcsolatos ismereteket, ha el szeretnénk kerülni a túlzott függőséget.

**Nyílt felületek közzététele**. Kerülje a szolgáltatások között található egyéni fordítási rétegek létrehozását. A szolgáltatásoknak ehelyett egy jól meghatározott API-egyezménnyel rendelkező API-t kell közzétenniük. Az API-nak rendelkeznie kell verziószámmal, hogy tovább lehessen fejleszteni a visszamenőleges kompatibilitás megtartása mellett. Így a szolgáltatásokat a tőlük függő felsőbb szintű szolgáltatások összehangolt frissítése nélkül frissítheti. A nyilvános szolgáltatásoknak egy RESTful API-t kell közzétenniük HTTP-n keresztül. Teljesítménybeli megfontolásokból a háttérszolgáltatások használhatnak RPC stílusú üzenetkezelési protokollt. 

**Tervezés és tesztelés szolgáltatási szerződések alapján**. Ha a szolgáltatások jól meghatározott API-kat tesznek közzé, azok a fejlesztés és tesztelés alapjául szolgálhatnak. Így egy adott szolgáltatás fejlesztéséhez és teszteléséhez a tőle függő szolgáltatásokat nem kell felpörgetnie. (Az integrálást és a terheléstesztelést természetesen továbbra is a tényleges szolgáltatásokon kell végrehajtani.)

**Infrastruktúra elválasztása az üzleti logikától**. Nem ajánlott keverni az üzleti logikát az infrastruktúrához kapcsolódó funkciókkal, például az üzenetkezeléssel vagy az adatmegőrzéssel. Ha ezeket nem különíti el, az üzleti logika módosításai az infrastruktúrarétegek frissítését is szükségessé teszik, és fordítva. 

**Azonos szerepet betöltő funkciók külön szolgáltatásba foglalása**. Ha például több szolgáltatásnak is kell kéréseket hitelesítenie, ezt a funkciót egy külön szolgáltatásba helyezheti át. A hitelesítési szolgáltatást ezután az azt használó többi szolgáltatás módosítása nélkül is fejlesztheti, például új hitelesítési folyamat hozzáadásával.

**A szolgáltatások egymástól független üzembe helyezése**. Ha a fejlesztési és üzemeltetési csapat az alkalmazás többi szolgáltatásától függetlenül helyezhet üzembe egy adott szolgáltatást, a frissítéseket gyorsabban és biztonságosabban lehet végrehajtani. A hibajavításokra és az új funkciók bevezetésére gyorsabb ütemben kerülhet sor. Tervezéskor ügyeljen arra, hogy az alkalmazás és a kibocsátási folyamat egyaránt támogassa a független frissítéseket.