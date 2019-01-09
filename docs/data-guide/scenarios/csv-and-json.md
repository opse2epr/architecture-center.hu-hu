---
title: CSV- és JSON-fájlok feldolgozása
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 78d57ac4f229e863e676bf3cad0140864cd243d1
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/08/2019
ms.locfileid: "54113982"
---
# <a name="working-with-csv-and-json-files-for-data-solutions"></a>A data-megoldások a fürt megosztott kötetei szolgáltatás és a JSON-fájlok használata

CSV- és JSON amelyek valószínűleg a leggyakrabban használt formátumok feldolgozására, és kicserélni, strukturálatlan és részben strukturált adatok tárolására használható.

## <a name="about-csv-format"></a>Tudnivalók a CSV formátum

CSV (vesszővel tagolt értékek)-fájlok gyakran használják a táblázatos adatokat egyszerű szöveges rendszerek között. Általában tartalmaznak egy fejléc sorra, amely az adatok oszlopnevek biztosít, hanem egyéb számítanak félig strukturált. Ez az az oka, hogy az CSV-k természetesen nem jelenítik meg hierarchikus és relációs adatokat. Adatkapcsolatok általában kezeli a több CSV-fájlok, ahol a külső kulcsokat egy vagy több fájl oszlopai vannak tárolva, de ezeket a fájlokat kapcsolatai nem ki magát a formátum. CSV formátumú fájlok felhasználhatja más elválasztó mellett vesszővel válassza el egymástól, például a lapok és a szóközöket.

Annak ellenére, hogy korlátait CSV-fájlok használata népszerű választás, ha az adatcseréhez, mivel számos különféle üzleti, fogyasztói és tudományos alkalmazásokat támogatja őket. Például az adatbázis és a táblázatkezelő programok importálása és exportálása CSV-fájlok. A legtöbb batch- és stream adatokat feldolgozó motorokkal, például a Spark és Hadoop-, ehhez hasonlóan natív módon támogatja a szerializálásához és deszerializálásához CSV-formátumú fájlok, és kínálnak az olvasási séma alkalmazására. Ez megkönnyíti az adatokkal való munka a, azzal, lekérdezéseket és az információkat tárolja a hatékonyabb adatformátum; Ez gyorsabb feldolgozása a lehetőségeket kínál.

## <a name="about-json-format"></a>Tudnivalók a JSON-formátumban

Kulcs-érték párok félig strukturált formátumban (JavaScript Object Notation) JSON-adatok értéke jelöli. A rendszer JSON-XML fájlok, gyakran összehasonlítja is képesek az adattárolás hierarchikus formátumban, a szülő-gyermek jelölt adatok beágyazott módon. Mindkettő saját leíró és emberi olvasásra alkalmas, de JSON-dokumentumok általában sokkal kisebb, és az exchange online-adatokhoz, különösen a REST-alapú webszolgáltatások megjelenésével népszerű használatukat.

JSON-formátumú fájlok szabályozhatják CSV számos előnnyel jár:

- JSON megőrzi a hierarchikus struktúra, megkönnyítve a kapcsolódó adatok tárolásához egy dokumentum, és összetett kapcsolatokat jelölik.
- Szinte bármelyik programozási nyelvével natív támogatást nyújt a JSON deszerializálása objektumokba, vagy egyszerű JSON-szerializálás kódtárakat biztosít.
- JSON támogatja a listák az objektumok, így egy relációs adatmodellbe rendezetlen listák fordításának elkerülése érdekében.
- JSON-ja olyan gyakran használt fájlformátum NoSQL-adatbázisok, például a MongoDB, a Couchbase és az Azure Cosmos DB-hez.

Nagy mennyiségű adatot a hálózaton keresztül érkező már JSON formátumban van, mivel a legtöbb, a web-alapú programozási nyelv támogatja a JSON-fájllal működik, natív módon, vagy külső kódtáraiban szerializálható és deszerializálható JSON-adatok használatával. Az univerzális támogatása JSON formátumban logikai struktúra adatreprezentációt keresztül exchange formátumokat a gyakran használt adatokkal, és a ritkán használt adatok adattárolás annak használatára vezetett.

Számos batch- és stream adatokat feldolgozó motorokkal natív módon támogatja a JSON-szerializálást és deszerializálást. Bár előfordulhat, hogy egy több teljesítményre optimalizált formátumban, például a Parquet vagy Avro, végső soron tárolt JSON-dokumentumok tárolt adatok a nyers adatokat a forrás hiteles, ami kritikus fontosságú az újrafeldolgozás az adatok igény szerint funkcionál.

## <a name="when-to-use-csv-or-json-formats"></a>Mikor érdemes használni a CSV vagy JSON formátumban

CSV-k exportálása és adatokat, és feldolgozás, analitika és gépi tanulási leggyakrabban szolgálnak. JSON-formátumú fájlok rendelkezik ugyanazokat az előnyöket, de gyakran használt adatokkal exchange megoldások gyakori. JSON-dokumentumok gyakran által küldött webes és mobil eszközök, az online tranzakció végrehajtása IoT (eszközök internetes hálózata) eszközök a egyirányú és kétirányú kommunikációra, vagy ügyfélalkalmazások kommunikál a SaaS- és PaaS-szolgáltatások vagy a kiszolgáló nélküli architektúrák.

CSV- és JSON formátumok egyaránt megkönnyíti exchange-adatok továbbítását különböző rendszerek és eszközök között. Részben strukturált formátumukban rugalmasságáról szinte bármilyen típusú adat átvitele, és ezek a formátumok univerzális támogatása, így egyszerű a használata. Mind a nyers azokban az esetekben a feldolgozott adatokat tároló bináris formátumban is a hatékonyabb lekérdezéséhez hiteles forrásaként használható.

## <a name="working-with-csv-and-json-data-in-azure"></a>CSV- és JSON-adatokat az Azure-használata

Az Azure használata a fürt megosztott kötetei szolgáltatás és a JSON-fájlok igényeitől függően számos megoldást nyújt. E fájlok esetében elsődleges alkotóelemeit hely az Azure Storage vagy az Azure Data Lake Store. A legtöbb Azure-szolgáltatások ezekkel a munka, és más szöveges fájlok integrálását vagy objektumokat tároló szolgáltatás. Bizonyos esetekben azonban Ön dönthet, közvetlenül az adatok importálása az Azure SQL vagy más adattárolóbeli tárolására. SQL Server rendszer tárolása és használata JSON-dokumentumok, amely megkönnyíti a natív támogatását [importálása és az ilyen típusú fájlok feldolgozása](/sql/relational-databases/json/import-json-documents-into-sql-server). Egy olyan segédprogram, például az SQL tömeges importálása az egyszerűen használható [importálása CSV-fájlok](/sql/relational-databases/json/import-json-documents-into-sql-server).

JSON-fájlok közvetlenül az Azure Blob Storage-ból anélkül, hogy importálná őket az Azure SQL is lekérdezheti. Ez a megközelítés egy teljes példa: [JSON-fájlokat az Azure SQL használata](https://medium.com/@mauridb/work-with-json-files-with-azure-sql-8946f066ddd4). Jelenleg ez a beállítás nem érhető el, a CSV-fájlok.

A forgatókönyvtől függően előfordulhat, hogy végre [kötegelt feldolgozás](../big-data/batch-processing.md) vagy [valós idejű feldolgozás](../big-data/real-time-processing.md) az adatok.

## <a name="challenges"></a>Problémák

Nincsenek áttekinthet néhány problémát, ezek a formátumok használatakor érdemes figyelembe venni:

- Bármely korlátozások az adatmodellen alapuló, anélkül CSV és a JSON-fájlok során gyakran fordul elő, az adatok sérülésének ("a szemétgyűjtési, szemétgyűjtési meg"). Például van egy dátum/idő objektumot, vagy a fájl nincs fogalmát, a fájl formátuma nem akadályozza meg, például beszúrás, egy dátummező, az "ABC123".

- A ritka elérésű tárolási megoldásként CSV és a JSON-fájlok használatával nem méretezhetők jól, ha a big Data típusú adatok használata. A legtöbb esetben azok nem bonthatók partíciók párhuzamos feldolgozásra, és nem lehet tömörített, valamint az olyan bináris formátumot. Ez gyakran vezet feldolgozását, és ezek az adatok tárolása például Parquet és ORC (optimalizált sor Oszlopalapú), amelyek is biztosítanak az indexeket és a tárolt adatok beágyazott statisztikája olvasásra optimalizált formátumokba.

- Szükség lehet a alkalmazni egy sémát a szolgáltatásban tárolt részben strukturált adatok lekérdezéséhez és kielemzéséhez könnyebb. Általában ehhez az adatok tárolása egy másik képernyő, amely megfelel az a környezet adattárolási igényeinek megfelelően, például egy adatbázison belül.
