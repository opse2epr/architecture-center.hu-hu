---
title: Fürt megosztott kötetei szolgáltatás és a JSON-fájlok feldolgozása
description: ''
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 02e684d562cfe555f9e3596ad0a2f1a00d05c7a7
ms.sourcegitcommit: 51f49026ec46af0860de55f6c082490e46792794
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/03/2018
ms.locfileid: "30301078"
---
# <a name="working-with-csv-and-json-files-for-data-solutions"></a>A data-megoldások a fürt megosztott kötetei szolgáltatás és a JSON fájlok használata

Fürt megosztott kötetei szolgáltatás és a JSON várhatóan a legfontosabb választásával dolgozhat fel, cseréjét, és strukturálatlan és félig strukturált adatok tárolására használt formátum. 

## <a name="about-csv-format"></a>Tudnivalók a CSV formátum

CSV (vesszővel tagolt) fájlok általában használhatók az exchange-adatok egyszerű szövegként rendszerek között. Általában tartalmaz, amely az adatok oszlopnevek fejlécsort, hanem egyéb minősülnek félig strukturált. Ez az az oka, a megosztott fürtkötetek természetesen nem tudja ábrázolni a hierarchikus és relációs adatok. Adatok kapcsolatok általában kezeli, és több CSV-fájlok, amikor külső kulcsokat egy vagy több fájl oszlopait vannak tárolva, de ezeket a fájlokat kapcsolatai nem szerint van megadva a formátum magát. CSV formátumú fájlok mellett vesszővel válassza el egymástól, például a lapok és szóközöket más elválasztó karaktert használhat.

Annak ellenére, hogy azok korlátai CSV-fájlok a népszerű választás az adatcsere, azért, mert a számos üzleti fogyasztói és tudományos alkalmazások támogatja őket. Például az adatbázis és a táblázatkezelő programok importálása és exportálása CSV-fájlok. A legtöbb kötegelt és adatfolyam adatok feldolgozórendszerekkel, például a külső és a Hadoop, hasonlóan natív módon szerializálása és deszerializálása CSV formátumú fájlokat támogatja, és a séma alkalmazása olvasáskor módjai nyújtsanak. Ez leegyszerűsíti az adatok, beállítások lekérdezése rajta, és az információkat tárolja a gyorsabb feldolgozás hatékonyabb adatformátum felajánlásával használatát.

## <a name="about-json-format"></a>Kapcsolatos JSON formátumban

Kulcs-érték párok félig strukturált (JavaScript Object Notation) JSON-adatokat jelzi. A rendszer JSON XML, gyakran összehasonlítja módon is képesek az adattárolás gyermek jelölt adatok beágyazott annak szülő hierarchikus formátumban. Mindkettő önálló leíró és olvasható emberi, de JSON-dokumentumok általában sokkal kisebb, ami népszerű használatuk online adatcsere, különösen a REST-alapú webes szolgáltatások bevezetése. 

JSON-formátumú fájlok CSV prioritást számos előnnyel jár:

* JSON tart fenn hierarchikus struktúrákat, így könnyebben egy dokumentumban kapcsolódó adatok tárolásához és összetett jelentik.
* A legtöbb programozási nyelv natív támogatást nyújt a JSON-objektumokba deszerializálása, vagy adjon meg egyszerűsített JSON szerializálási szalagtárak.
* JSON támogatja a listák az objektumok, útmutatás nyújtása a zavaró fordítást listák relációs adatmodell elkerülése érdekében.
* JSON-egy gyakran használt fájlformátum a NoSQL-adatbázisok, például a MongoDB, Couchbase és Azure Cosmos DB ja.

Nagy mennyiségű adat a hálózaton keresztül érkező már JSON formátumban van, mert a legtöbb, a web-alapú programozási nyelveket támogatja a natív módon, vagy külső szalagtárak szerializálása és deszerializálása JSON-adatok JSON használata. A JSON univerzális támogatása vezetett történő használatát, logikai formátumokban adatok struktúra képviselik, exchange formátumokat a gyakran használt adatokkal, és ritkán használt adatok tárolására.

Sok kötegelt és az adatfolyam adatok feldolgozórendszerekkel natív módon támogatja a JSON-szerializálás és a deszerializálás. Bár a JSON-dokumentumok belül található adatok végső soron tárolhatjuk egy több teljesítményre optimalizált formátumban, például Parquet vagy Avro, a nyers adatok a igazság, ami az újrafeldolgozás az adatok igény szerint kritikus forrását funkcionál.

## <a name="when-to-use-csv-or-json-formats"></a>Mikor érdemes használni a CSV- vagy JSON formátumban

A CSV-k exportálása és adatokat, és számára történő feldolgozásakor. az elemzések és a gépi tanulás általában több használhatók. JSON-formátumú fájlokat a azonos szempontból előnyökkel járhat, de a gyakran használt adatokkal exchange megoldások gyakori. JSON-dokumentumok gyakran által küldött webes és mobil eszközök online tranzakciók végrehajtása (az eszközök internetes hálózata) IoT-eszközök az egyirányú vagy kétirányú kommunikációra, vagy ügyfélalkalmazások kommunikál a Szolgáltatottszoftver-és PaaS vagy kiszolgáló nélküli architektúrák. 

Fürt megosztott kötetei szolgáltatás és a JSON fájlformátumok mindkettő könnyen adatcserében működik közre eltérő rendszerekhez, vagy az eszközök között. Félig strukturált formátumukban engedélyezése rugalmasságot szinte bármilyen típusú adatok átvitele, és ezek a formátumok univerzális támogatása akkor tegye őket egyszerű történő együttműködésre. Mindkét igazság azokban az esetekben a feldolgozott adatokat tároló bináris formátumú hatékonyabb lekérdezése a nyers forrásaként használható. 

## <a name="working-with-csv-and-json-data-in-azure"></a>Az Azure-ban CSV és a JSON adatok használata

Azure számos megoldást nyújt a fürt megosztott kötetei szolgáltatás és a JSON-fájlok igényeitől függően használata. Elsődleges követően a fájlok helye vagy az Azure Storage, vagy az Azure Data Lake Store. A legtöbb Azure ezekkel munka szolgáltatását, és valamelyik objektum tárolási szolgáltatás integrálása más szöveges fájlokat. Bizonyos esetekben azonban úgy is dönthet, hogy közvetlenül az adatok importálása Azure SQL- vagy más adattárolóbeli tárolására. Natív támogatás tárolásához és használata JSON-dokumentumokat, ami megkönnyíti az SQL Server rendelkezik [importálja, és dolgozza fel ilyen típusú fájlok](/sql/relational-databases/json/import-json-documents-into-sql-server). A segédprogram például SQL tömeges importálásához könnyen használható [importálása CSV-fájlok](/sql/relational-databases/json/import-json-documents-into-sql-server).

A forgatókönyvtől függően előfordulhat, hogy végre [kötegfeldolgozási](../big-data/batch-processing.md) vagy [valós idejű feldolgozással](../big-data/real-time-processing.md) az adatok.

## <a name="challenges"></a>Problémák

Ezek a formátumok használata során figyelembe kell venni néhány akadályok merülnek fel:

* Az adatmodell bármely korlátozások, nélkül CSV és a JSON fájlok vannak téve az adatok sérülésének ("a szemétgyűjtés, szemétgyűjtési el"). Például van egy dátum/idő objektumot, vagy a fájl nincs fogalmát, a fájl formátuma nem akadályozzák meg "ABC123" elhelyezése egy mező, például.

* Fürt megosztott kötetei szolgáltatás és a JSON-fájlok használatával a cold tárolási megoldásként nem méretezhetők jól big Data típusú adatok használata. A legtöbb esetben ezek nem bonthatók partíciókat a párhuzamos feldolgozást, és nem lehet tömörített, valamint az olyan bináris formátum. Ez gyakran vezet feldolgozásához, és ezek az adatok tárolása Parquet és az ORC (optimalizált sor oszlopos), amelyek is indexek és beágyazott statisztikája található adatokat például olvasásra optimalizált formátumokba.

* Szükség lehet a séma alkalmazása a félig strukturált adatok könnyebb lekérdezéséhez és elemzéséhez. Általában ehhez az adatok tárolása, hogy megfelel-e a környezet adattárolási igények kielégítésére, például egy adatbázis.

