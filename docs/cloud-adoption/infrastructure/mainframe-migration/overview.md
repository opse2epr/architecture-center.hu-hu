---
title: 'Nagyszámítógépek Migrálása: Áttekintés'
description: A nagyszámítógépes környezetek alkalmazásokat át az Azure-ba, a bevált, magas rendelkezésre állású és méretezhető infrastruktúrát Nagyszámítógépek a jelenleg futó rendszerek.
author: njray
ms.date: 12/27/2018
ms.openlocfilehash: 41fc799f15500276ada1667121e5f1fce3413a3a
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55899088"
---
# <a name="mainframe-migration-overview"></a>A nagyszámítógépes áttelepítése – áttekintés

Számos cégek és szervezetek számára néhány vagy összes azok nagyszámítógépes számítási feladatok, alkalmazásokat és adatbázisokat a felhőbe való áthelyezését előnyeit. Az Azure felhőszinten nélkül Nagyszámítógépek társított rugalmasságára számos nagyszámítógép-hez hasonló funkciókat biztosít.

A kifejezés nagyszámítógépes általában nagy számítógéprendszer hivatkozik, de túlnyomó többsége jelenleg üzembe helyezett Nagyszámítógépek IBM rendszer Z kiszolgálók vagy az IBM plug-compatible rendszerek MVS, DOS, VSE, az operációs rendszer/390 vagy z/OS rendszerű. Nagyszámítógépes rendszereket továbbra is számos iparágban alapvető fontosságú információk rendszerek futtatására használható, és olyan hely az magas jellemző forgatókönyvek, például a nagy, nagy volumenű tranzakció-intenzív IT-környezetek rendelkeznek.

A felhőbe való migrálás lehetővé teszi a vállalatok számára, hogy modernizálása az infrastruktúrán. A cloud services elérhetővé teheti nagyszámítógépes alkalmazások pedig az értéket, amely azt megoldás lehet, amikor a szervezet erre szükség van. Számos számítási feladatok kis kód módosítása, például adatbázisok nevei frissítése továbbíthatók az Azure-bA. Áttelepítheti a fokozatos áttérést összetettebb számítási feladatokat.

A legtöbb Fortune 500-as cégek az Azure már fut a kritikus fontosságú számítási feladatokhoz. Az Azure jelentős bottom-line ösztönzők motiválhatják számos migrálási projektet. Vállalatok általában helyezze át a fejlesztési, és tesztelheti a számítási feladatokat az Azure-ban először, DevOps, az e-mailek és a vész-helyreállítási szolgáltatás követ.

## <a name="intended-audience"></a>Célközönség

Ha az Ön által választandó áttelepítés vagy a beállítás a cloud services hozzáadásával az informatikai környezetében, ez az útmutató hasznos információkat nyújt.

Ez az útmutató segítséget nyújt az informatikai szervezetek az áttelepítés beszélgetés indítása. Előfordulhat, hogy jobban megismerte az Azure és a felhőalapú infrastruktúrák mint, az Nagyszámítógépek, így ez az útmutató áttekintést Nagyszámítógépek működéséről, elindul, és folytatja a meghatározásához, mit, és hogyan telepítheti át különböző stratégiák.

## <a name="mainframe-architecture"></a>A nagyszámítógépes architektúra

A késedelmes 1950s a nagyszámítógépek futtathat nagy mennyiségű online tranzakciók és a kötegelt feldolgozás készültek, vertikális felskálázás kiszolgálóként. Emiatt a nagyszámítógépek rendelkezik (más néven zöld képernyők) online tranzakció-űrlapok és nagy teljesítményű i/o-rendszerek futtatások kötegelt feldolgozásához.

Nagyszámítógépek magas megbízhatósággal és rendelkezésre állás érdekében, hogy, és képesek hatalmas online tranzakció- és kötegelt feladatok futtatása az ismert. Egy tranzakció eredménye egyetlen kérelem, általában az adott felhasználó által kezdeményezett feldolgozási egy részét. Tranzakciók több más forrásokból, beleértve a weblapok, távoli munkaállomások és más információs rendszerekkel alkalmazások is származhatnak. Egy tranzakció is aktiválható automatikusan, egy előre meghatározott időpontban az alábbi ábrán látható módon.

![Egy tipikus IBM Nagyszámítógépek architektúra összetevői](../../_images/mainframe-migration/zOS-architectural-layers.png)

Egy tipikus IBM Nagyszámítógépek architektúra az alábbi gyakori összetevőket tartalmazza:

- **Előtér-rendszerek esetén:** Felhasználók terminálok, weblapok vagy távoli munkaállomások tranzakciók is kezdeményezhető. Nagyszámítógépes alkalmazások gyakran rendelkeznek az egyéni felhasználói felületek, megőrizhetőek a az Azure-ban migrálás után. A Terminálszolgáltatások emulátory systému továbbra is nagyszámítógépes alkalmazások eléréséhez használt, és zöld képernyős terminálok is nevezik.

- **Alkalmazásrétegek:** Nagyszámítógépek általában tartalmaznak egy ügyfél információk verziókezelő rendszer (CICS), az IBM z/OS nagyszámítógépes gyakran használják az IBM információk felügyeleti rendszer (IMS), egy üzenet-alapú tranzakciókezelő egy vezető tranzakció felügyeleti csomag. Batch-rendszerek kezelni a felhasználóifiók-rekordok nagy mennyiségű nagy átviteli sebességű adatok frissítései.

- **Kód:** Nagyszámítógépek által használt programozási nyelvek például Fortran, COBOL, PL / I és természetes. Feladat vezérlő nyelv (JCL) segítségével z/OS használata.

- **Adatbázisréteg:** Egy közös relációs adatbázist kezelő rendszer (DBMS) z/os IBM DD2. Az általa felügyelt adattárakon-nevű *dbspace területek* , amely tartalmaz egy vagy több tábla és a fizikai adatkészletek nevű tárolókészletet rendelt *dbextents*. Két fontos adatbázis-összetevői a következők: a címtár, amely azonosítja a tárolókészletet az adatok helye és a napló, amely tartalmaz egy rekordot az adatbázison végrehajtott műveletek. Különböző fájlszintű adatformátumok a célnyelven támogatottak. Z/os DB2 virtuális tárolási hozzáférési módszer (VSAM) adatkészletek általában használ az adatok tárolásához.

- **Felügyeleti csomag:** IBM Nagyszámítógépek például ütemezés TWS-OPC, például a szoftverek, eszközök a nyomtatási és a kimeneti management például CA-(KKT) és a nyomtatási sor, és a egy forráskódú verziókezelő rendszer kódot. Biztonságos hozzáférés-vezérlés az operációs rendszer z/a resource access control létesítmény (RACF) kezeli. Az adatbázis-kezelő az adatbázisban található adatok hozzáférést biztosít, és saját partícióval z/OS-környezetben futtatható.

- **LPAR:** Logikai partíciókkal vagy LPARs, oszthatja a számítási erőforrások szolgálnak. Egy fizikai nagyszámítógépes több LPARs van felosztva.

- **z/OS:** Egy 64 bites operációs rendszer leggyakrabban használt IBM Nagyszámítógépek.

IBM-rendszerek például CICS tranzakció figyelő segítségével nyomon követheti és kezelheti az üzleti tranzakciók minden aspektusát. CICS erőforrás-megosztás, az adatok integritását, és a végrehajtás rangsorolási kezeli. CICS engedélyezi a felhasználók, erőforrásokat foglal le, és adatbázis-kéréseket továbbít az alkalmazás egy adatbázis-kezelőhöz, például az IBM DB2-höz.

A pontosabb hangolása CICS gyakran használnak IMS/TM (korábbi nevén IMS/Data kommunikáció vagy IMS/tartományvezérlő). IMS úgy lett kialakítva, hogy csökkenteni a redundáns adattárolás érdekében az adatok egyetlen másolatának megtartásával. A folyamat során állapot fenntartására és üzleti funkciók rögzítése a tárolóban, a tranzakciós figyelője kiegészíti CICS.

## <a name="mainframe-operations"></a>A nagyszámítógépes műveletek

Tipikus nagyszámítógépes műveletek a következők:

- **Online:** Számítási feladat például tranzakció-feldolgozás, az adatbázis-kezelés és a kapcsolatok. Gyakran házirenddefiníción IBM DB2, CICS és z/OS-összekötők használatával.

- **Batch:** Feladatok futtatása minden hétköznap reggel például rendszeres ütemezés szerint általában a felhasználó beavatkozása nélkül. Batch-feladatok Micro fókusz Enterprise Server és a BMC Control-M szoftverek például egy JCL emulator használatával alapján Windows vagy Linux-alapú rendszereken futtatható.

- **Feladat vezérlő nyelv (JCL):** Adja meg a batch-feladatok feldolgozásához szükséges erőforrásokat. JCL ruház z/OS feladat vezérlő utasítások biztosítják ezt az információt. Alapszintű JCL hat különböző típusú utasításokat tartalmazza: FELADAT, ASSGN, DLBL, mértékben, LIBDEF és EXEC. Egy feladat több EXEC utasítások (lépések) is tartalmazhat, és az egyes lépések több LIBDEF, ASSGN, DLBL és mértékben utasítások lehetnek.

- **A program kezdeti betöltés (IPL):**  Egy másolatot az operációs rendszer betöltése a lemezről egy feldolgozó tényleges storage-ba, és azt futtatja hivatkozik. Helyreállítás üzemkimaradás IPLs szolgálnak. Egy IPL úgy működik, mint a rendszerindítást az operációs rendszert a Windows vagy Linux rendszerű virtuális gépekhez.

## <a name="next-steps"></a>További lépések

> [!div class="nextstepaction"]
> [Tévhitet és tények](myths-and-facts.md)
