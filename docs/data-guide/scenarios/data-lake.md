# <a name="data-lakes"></a>Data lake tárolók

Data lake storage tárháza nagy mennyiségű adatot tárol a natív, nyers formában. Data lake Store több terabájtnyi és több petabájtnyi adat skálázás vannak optimalizálva. Az adatok általában több heterogén forrásokból származnak, és strukturált, félig strukturált vagy strukturálatlan lehet. A data lake lényege, minden eredeti, a nyomtatónak állapotában tárolásához. Ez a módszer eltér a hagyományos [adatraktár](../relational-data/data-warehousing.md), amely alakítja át, és feldolgozza az adatokat feldolgozó időpontjában.

Data lake előnyei:

- Adatok soha nem fordul elő, mert a nyers formátumban tárolja az adatokat. Ez különösen hasznos a big data környezetben, ha lehetséges, hogy nem tudja előre következtetéseket érhetők el az adatokat.
- A felhasználók az adatok és a saját lekérdezések létrehozása.
- Gyorsabb, mint a hagyományos ETL-eszközöket is lehet.
- Rugalmasabb, mint egy adattárházat, mert a strukturálatlan és részben strukturált adatokat képes tárolni.

Egy teljes data lake megoldás áll egyaránt tárolás és feldolgozás céljából. Data lake storage hibatűrést, korlátlan méretezhetőséget és a különböző bármilyen típusú és méretű adatok nagy átviteli sebességű feldolgozási lett tervezve. Egy Data lake feldolgozási foglalja magában, vagy további feldolgozó motorokkal a következő célok szem előtt készült, és működhet a nagy mennyiségű data lake-ben tárolt adatok.

## <a name="when-to-use-a-data-lake"></a>Data lake használata

A data lake szokásos használati módjai többek között [adatfeltárás](./interactive-data-exploration.md), adatelemzési és machine learning.

Data lake is működhet, az adatforrás egy data warehouse-hoz. Ezzel a módszerrel a nyers adatok betöltődnek a data lake, és ezután lekérdezhető strukturált formában alakítja át. Általában az átalakítás használ egy [ELT](../relational-data/etl.md#extract-load-and-transform-elt) (kinyerési, betöltési, átalakítási) folyamatban, ahol az adatok betöltött és helyen alakította át. Már van relációs adatforrásból közvetlenül az adatraktárba, előfordulhat, hogy lépjen, egy ETL-folyamattal, a rendszer kihagyja a data lake használatával.

Data lake Store gyakran használják eseménystreamelés vagy IoT-forgatókönyveket, mert azok a nagy mennyiségű relációs és nem relációs adatok átalakítás vagy séma definíció nélküli megtarthatóak. Nagy mennyiségű, apró adatírási műveleteket kis késleltetésű kezelésére beépített, és optimalizált teljesítményt.

## <a name="challenges"></a>Problémák

- A séma vagy a leíró metaadatok hiánya is győződjön meg arról, az adatok lekérdezése vagy rögzített.
- Az adatok egységességének szemantikai hiánya teheti, hogy nagy kihívást jelentő elemzését, az adatokat, kivéve, ha felhasználók képzett címen adatelemzés.
- A data lake az adatokat a minőség biztosítása is nehézkes lehet.
- A megfelelő irányítás nélkül a hozzáférés-vezérlési és adatvédelmi problémák problémák is lehet. Milyen információt fogja a data lake, ki férhet hozzá az adatokat, és milyen célokra?
- Data lake nem feltétlenül a legjobb módszer, amely már relációs adatokat integrálhat.
- Önmagában egy data lake nem tartalmaz integrált vagy átfogó nézeteket a szervezeten belül.
- Data lake válhat egy megállapított földön adatokhoz, hogy valójában soha nem elemezte vagy insights célú adatbányászatra sem használja.

## <a name="relevant-azure-services"></a>Kapcsolódó Azure-szolgáltatásokat

- [Data Lake Store](/azure/data-lake-store/) egy rendkívül nagy kapacitású, Hadoop-kompatibilis adattár.
- [A Data Lake Analytics](/azure/data-lake-analytics/) szolgáltatás egy igény szerinti elemzési feladatokat végző szolgáltatás a big data-elemzés egyszerűsítésére.
