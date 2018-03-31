# <a name="data-lakes"></a>Adatok tavakat

Data lake tárháza tároló, amely nagy mennyiségű adatot tárol a natív, nyers formátumban. Data lake tárolók terabájt és adatmennyiségig skálázás vannak optimalizálva. Az adatok általában több heterogén forrásból származik, és lehet, hogy strukturált, félig strukturált vagy strukturálatlan. A data lake lényege, minden eredeti, nyomtatónak állapotában tárolásához. Ez a módszer eltér a hagyományos [adatraktár](../scenarios/data-warehousing.md), amely átalakítja, és feldolgozza azokat az adatfeldolgozást időpontjában.

Data lake előnyei:

- Adatok soha nem fordul elő, mert az adatok tárolását a nyers formátumban. Ez akkor big Data típusú adatok környezetben, különösen akkor hasznos, ha nem lehet felmérni előre következtetéseket érhetők el az adatokat.
- A felhasználók az adatokba, és a saját lekérdezések létrehozása.
- Valószínűleg gyorsabb, mint a hagyományos ETL-eszközök.
- Rugalmasabb, mint egy adatraktár, mert a strukturálatlan és félig strukturált adatok képes tárolni. 

A teljes data lake megoldás mind a tároló, mind a feldolgozási áll. Data lake tároló úgy van kialakítva, a hibatűrést, a végtelen méretezhetőség és a nagy átviteli adatfeldolgozást az adatok különböző alakra. Data lake feldolgozási magában foglalja egy vagy több feldolgozórendszerekkel ezen célok szem előtt-val készült, és léptékű data lake-ben tárolt adatokat is működik.

## <a name="when-to-use-a-data-lake"></a>Mikor érdemes használni a data lake

A data lake tipikus használati többek között [adatfeltárás](../scenarios/interactive-data-exploration.md), adatelemzés és a gépi tanulás. 

Data lake is működhet, és az adatok az adatforráshoz. Ezt a módszert használja a nyers adatok azokat a data lake okozhatnak, és majd egy strukturált lekérdezhető formátumra alakítja át. Általában ez a transzformáció használ egy [ELT](../scenarios/etl.md#extract-load-and-transform-elt) (kivonat-betöltési-átalakítási) folyamatban, ahol az adatok okozhatnak, és át legyenek-e érvényben. Már van relációs adatforrásból közvetlenül az adatraktárba, előfordulhat, hogy nyissa meg a rendszer kihagyja a data lake ETL folyamat.

Data lake tárolók gyakran használják adatfolyam-esemény vagy az IoT-forgatókönyvek esetén, mert nagy mennyiségű relációs és nonrelational adatok átalakítása vagy a séma meghatározása nélkül is megőrzik. A kis késleltetésű kis írása jelentős mennyiségű kezelésére beépített, és jelentős átviteli sebességet vannak optimalizálva.

## <a name="challenges"></a>Kihívásai

- A séma vagy a leíró metaadatok hiánya az adatok tehet rögzített használatához vagy lekérdezni.
- Az adatok szemantikai egységességének hiánya teheti kihívást elemzés végrehajtásához az adatokat, kivéve, ha felhasználók, adatelemzés magas gyakorlott.
- Nehéz garantálják az adatok, amelyek a data lake lehet. 
- A megfelelő irányítás nélkül hozzáférés és adatvédelem kérdéseket nem problémákat. Milyen információkat állapotra vált, a data lake, ki férhet hozzá az adatokat, és milyen célból?
- Data lake nem lehet a legjobb módszer, amely már relációs adatok integrálására.
- Önálló data lake nem biztosít integrált vagy körű nézetek a teljes szervezetben. 
- Data lake válhat egy megállapított ground valójában soha nem elemzett vagy az elemzések bányásszák adatokat.

## <a name="relevant-azure-services"></a>Megfelelő Azure-szolgáltatások

- [Data Lake Store](/azure/data-lake-store/) kapacitású, a Hadoop-kompatibilis tára.
- [A Data Lake Analytics](/azure/data-lake-analytics/) van egy igény szerinti analytics feladat service egyszerűbbé teheti a big data elemzést.

