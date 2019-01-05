---
title: Az Azure-erőforrások elnevezési konvenciói
titleSuffix: Best practices for cloud applications
description: Virtuális gépek, tárfiókok, hálózatok, virtuális hálózatok, alhálózatok és egyéb Azure-entitások elnevezése javaslatok.
author: telmosampaio
ms.date: 10/19/2018
ms.custom: seodec18
ms.openlocfilehash: f0349b5db7eb15037bd92567eaf917b5d044daa0
ms.sourcegitcommit: 036cd03c39f941567e0de4bae87f4e2aa8c84cf8
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/05/2019
ms.locfileid: "54058199"
---
# <a name="naming-conventions-for-azure-resources"></a>Az Azure-erőforrások elnevezési konvenciói

Ez a cikk az Azure-erőforrások elnevezési szabályainak és korlátozásainak összegzését, valamint az elnevezési konvenciókkal kapcsolatos alapvető javaslatokat tartalmazza. Ezeket a javaslatokat kiindulópontként használhatja saját, egyéni igényeire szabott konvencióinak létrehozásához.

A Microsoft Azure-erőforrások nevének megválasztása a következő okokból fontos:

- Az utólagos névváltoztatás körülményes.
- A neveknek meg kell felelniük az adott erőforrástípusra vonatkozó követelményeknek.

Egységes elnevezési konvenciók használata esetén az erőforrások könnyebben megtalálhatók. Emellett azt is jelezheti, hogy az adott erőforrás milyen szerepkört tölt be a megoldáson belül.

A jól működő elnevezési konvenciók titka, hogy minden alkalmazásra és az egész vállalatra egységesen vonatkozzanak.

## <a name="naming-subscriptions"></a>Előfizetések elnevezése

Az Azure-előfizetések elnevezésekor részletes nevek használatával egyértelműen jelezhető az egyes előfizetések kontextusa és célja. Sok előfizetést tartalmazó környezetekben az egységes elnevezési konvenciók segítik a tájékozódást.

Az előfizetések elnevezésének ajánlott mintája a következő:

`<Company> <Department (optional)> <Product Line (optional)> <Environment>`

- A vállalat általában minden előfizetés esetén azonos. Egyes vállalatok szervezeti felépítésében azonban leányvállalatok is szerepelhetnek. Ezeket a vállalatokat egy központi informatikai csoport felügyelheti. Ilyen esetekben a vállalatok elkülöníthetők az anyavállalat (*Contoso*) és a leányvállalat (*Northwind*) nevének használatával.
- Szervezeti egység nevét, amely tartalmazza a felhasználók csoport a szervezeten belül. Ez a névtérben nem kötelező.
- A termékvonal egy termék vagy szolgáltatás neve, amelyet a részlegen belül hoznak létre vagy szolgáltatnak. Ez belső szolgáltatások és alkalmazások esetén általában nem kötelező. Határozottan javasolt azonban a használata olyan nyilvános szolgáltatások esetén, ahol egyszerű elkülöníthetőségre és azonosításra van szükség (például a számlázási rekordok könnyű elkülöníthetőségére).
- A környezet (pl. fejlesztési, minőségbiztosítási vagy éles környezet) az alkalmazások vagy szolgáltatások üzembehelyezési életciklusára utal.

| Vállalat | Részleg | Termékvonal vagy szolgáltatás | Környezet | Teljes név |
| --- | --- | --- | --- | --- |
| Contoso |SocialGaming |AwesomeService |Production |Contoso SocialGaming AwesomeService Production |
| Contoso |SocialGaming |AwesomeService |Dev |Contoso SocialGaming AwesomeService Dev |
| Contoso |IT |InternalApps |Production |Contoso IT InternalApps Production |
| Contoso |IT |InternalApps |Dev |Contoso IT InternalApps Dev |

A nagyobb vállalatok előfizetéseinek további információkért lásd: [Azure enterprise scaffold - előíró előfizetés-irányítás](/azure/architecture/cloud-adoption/appendix/azure-scaffold).

## <a name="use-affixes-to-avoid-ambiguity"></a>Elő- és utótagok használata a félreérthetőség elkerülése érdekében

Az Azure-erőforrások elnevezésekor közös előtagok vagy utótagok használata javasolt az erőforrás típusának és kontextusának azonosításához. A típusra, a metaadatokra és a kontextusra vonatkozó információk programozott módon elérhetők, de a közös elő- és utótagok használatával egyszerűsíthető a vizuális azonosítás. Ha elő- és utótagokat is beépít az elnevezési konvenciókba, fontos egyértelműen megadni, hogy az adott toldalék a szó elejére (előtag) vagy a szó végére (utótag) kerül-e.

Vegyük példaként egy számítási motort futtató szolgáltatás két lehetséges nevét:

- SvcCalculationEngine (előtag)
- CalculationEngineSvc (utótag)

Az elő- és utótagok az adott erőforrások különböző aspektusaira utalhatnak. Az alábbi táblázat néhány általános példát mutat be.

| Aspektus | Példa | Megjegyzések |
| --- | --- | --- |
| Környezet |dev, prod, QA (fejlesztői, éles, minőségbiztosítási) |Az erőforrás környezetét határozza meg |
| Hely |uw (USA nyugati régiója), ue (USA keleti régiója) |A régiót jelöli, amelyben az erőforrás üzembe van helyezve |
| Példány |1, 2... |Az olyan erőforrásokhoz, például virtuális gépeket vagy a hálózati adapterek több megnevezett példánnyal rendelkezik. |
| Termék vagy szolgáltatás |szolgáltatás |Az erőforrás által támogatott terméket, alkalmazást vagy szolgáltatást jelöli |
| Szerepkör |sql, web, messaging (sql, web, üzenetkezelés) |A társított erőforrás szerepét jelöli |

Ha vállalata vagy projektjei számára egy adott elnevezési konvenciót fejlesztésével, fontos válasszon olyan közös elő-és és elhelyezésének (elő- vagy utótag).

## <a name="naming-rules-and-restrictions"></a>Elnevezési szabályok és korlátozások

Az Azure-ban minden egyes erőforrás- vagy szolgáltatástípus megszab bizonyos elnevezési korlátokat, és meghatározza a hatókört. Az elnevezési konvencióknak vagy mintáknak meg kell felelniük az adott elnevezési szabályoknak és a hatókörnek. Például egy virtuális gép neve leképezhető DNS-névvé (ezáltal az egész Azure-ban egyedinek kell lennie), de egy VNET nevének hatóköre csak arra az erőforráscsoportra terjed ki, amelyen belül létrehozták.

Általában nem ajánlott a speciális karakterek (`-` vagy `_`) használata a nevek első és utolsó karaktereként. Ezek a karakterek a legtöbb érvényesítési szabállyal ütköznek.

### <a name="general"></a>Általános kérdések

| Entitás | Hatókör | Hossz | Kis- és nagybetűk | Érvényes karakterek | Javasolt minta | Példa |
| --- | --- | --- | --- | --- | --- | --- |
|Erőforráscsoport |Előfizetés |1–90 |Kis- és nagybetűk megkülönböztetése nélkül |Alfanumerikus, aláhúzásjel, zárójel, kötőjel, időszak (kivéve a végén), és a Unicode-karaktereket, amelyek megfelelnek a következő reguláris kifejezésre dokumentált [Itt](/rest/api/resources/resourcegroups/createorupdate). |`<service short name>-<environment>-rg` |`profx-prod-rg` |
|Rendelkezésre állási csoport |Erőforráscsoport |1–80 |Kis- és nagybetűk megkülönböztetése nélkül |Alfanumerikus karakterek, aláhúzásjel és kötőjel |`<service-short-name>-<context>-as` |`profx-sql-as` |
|Címke |Társított entitás |512 (név), 256 (érték) |Kis- és nagybetűk megkülönböztetése nélkül |Alfanumerikus karakterek |`"key" : "value"` |`"department" : "Central IT"` |

### <a name="compute"></a>Compute

| Entitás | Hatókör | Hossz | Kis- és nagybetűk | Érvényes karakterek | Javasolt minta | Példa |
| --- | --- | --- | --- | --- | --- | --- |
|Virtuális gép |Erőforráscsoport |1–15 (Windows), 1–64 (Linux) |Kis- és nagybetűk megkülönböztetése nélkül |Alfanumerikus karakterek és kötőjel |`<name>-<role>-vm<number>` |`profx-sql-vm1` |
|Függvényalkalmazás | Globális |1–60 |Kis- és nagybetűk megkülönböztetése nélkül |Alfanumerikus karakterek és kötőjel |`<name>-func` |`calcprofit-func` |

> [!NOTE]
> Az Azure-ban a virtuális gépek két különböző névvel rendelkeznek: egy virtuálisgép-névvel és egy gazdagépnévvel. Amikor létrehoz egy virtuális gépet a portálon, ugyanaz a név lesz a gazdagépnév és a virtuális gép erőforrásneve is. A fenti korlátozások a gazdagépnévre vonatkoznak. A tényleges erőforrásnév legfeljebb 64 karakterből állhat.

### <a name="storage"></a>Storage

| Entitás | Hatókör | Hossz | Kis- és nagybetűk | Érvényes karakterek | Javasolt minta | Példa |
| --- | --- | --- | --- | --- | --- | --- |
|Tárfiók neve (adatok) |Globális |3–24 |Kisbetűs |Alfanumerikus karakterek |`<globally unique name><number>` (egy függvénnyel számítson ki egy egyedi GUID azonosítót a tárfiókok elnevezéséhez) |`profxdata001` |
|Tárfiók neve (lemezek) |Globális |3–24 |Kisbetűs |Alfanumerikus karakterek |`<vm name without hyphens>st<number>` |`profxsql001st0` |
| Tárolónév |Tárfiók |3-63 |Kisbetűs |Alfanumerikus karakterek és kötőjel |`<context>` |`logs` |
|A blob neve | Tároló |1–1024 |Kis- és nagybetűk megkülönböztetése |Bármely URL-karakter |`<variable based on blob usage>` |`<variable based on blob usage>` |
|Üzenetsor neve |Tárfiók |3-63 |Kisbetűs |Alfanumerikus karakterek és kötőjel |`<service short name>-<context>-<num>` |`awesomeservice-messages-001` |
|Tábla neve | Tárfiók |3-63 |Kis- és nagybetűk megkülönböztetése nélkül |Alfanumerikus karakterek |`<service short name><context>` |`awesomeservicelogs` |
|Fájlnév | Tárfiók |3-63 |Kisbetűs | Alfanumerikus karakterek |`<variable based on blob usage>` |`<variable based on blob usage>` |
|Data Lake Store | Globális |3–24 |Kisbetűs | Alfanumerikus karakterek |`<name>dls` |`telemetrydls` |

### <a name="networking"></a>Hálózat

| Entitás | Hatókör | Hossz | Kis- és nagybetűk | Érvényes karakterek | Javasolt minta | Példa |
| --- | --- | --- | --- | --- | --- | --- |
|Virtuális hálózat (VNet) |Erőforráscsoport |2–64 |Kis- és nagybetűk megkülönböztetése nélkül |Alfanumerikus karakterek, kötőjelet, aláhúzásjelet és időszak |`<service short name>-vnet` |`profx-vnet` |
|Alhálózat |Szülő VNet |2–80 |Kis- és nagybetűk megkülönböztetése nélkül |Alfanumerikus karakterek, kötőjelet, aláhúzásjelet és időszak |`<descriptive context>` |`web` |
|Hálózati adapter |Erőforráscsoport |1–80 |Kis- és nagybetűk megkülönböztetése nélkül |Alfanumerikus karakterek, kötőjelet, aláhúzásjelet és időszak |`<vmname>-nic<num>` |`profx-sql1-vm1-nic1` |
|Hálózati biztonsági csoport |Erőforráscsoport |1–80 |Kis- és nagybetűk megkülönböztetése nélkül |Alfanumerikus karakterek, kötőjelet, aláhúzásjelet és időszak |`<service short name>-<context>-nsg` |`profx-app-nsg` |
|Hálózat biztonsági csoport szabálya |Erőforráscsoport |1–80 |Kis- és nagybetűk megkülönböztetése nélkül |Alfanumerikus karakterek, kötőjelet, aláhúzásjelet és időszak |`<descriptive context>` |`sql-allow` |
|Nyilvános IP-cím |Erőforráscsoport |1–80 |Kis- és nagybetűk megkülönböztetése nélkül |Alfanumerikus karakterek, kötőjelet, aláhúzásjelet és időszak |`<vm or service name>-pip` |`profx-sql1-vm1-pip` |
|Load Balancer |Erőforráscsoport |1–80 |Kis- és nagybetűk megkülönböztetése nélkül |Alfanumerikus karakterek, kötőjelet, aláhúzásjelet és időszak |`<service or role>-lb` |`profx-lb` |
|Terheléselosztási szabályok konfigurációja |Load Balancer |1–80 |Kis- és nagybetűk megkülönböztetése nélkül |Alfanumerikus karakterek, kötőjelet, aláhúzásjelet és időszak |`<descriptive context>` |`http` |
|Azure Application Gateway |Erőforráscsoport |1–80 |Kis- és nagybetűk megkülönböztetése nélkül |Alfanumerikus karakterek, kötőjelet, aláhúzásjelet és időszak |`<service or role>-agw` |`profx-agw` |
|Traffic Manager-profil |Erőforráscsoport |1–63 |Kis- és nagybetűk megkülönböztetése nélkül |Alfanumerikus karakterek, kötőjelet és időszak |`<descriptive context>` |`app1` |

### <a name="containers"></a>Containers

| Entitás | Hatókör | Hossz | Kis- és nagybetűk | Érvényes karakterek | Javasolt minta | Példa |
| --- | --- | --- | --- | --- | --- | --- |
|Container Registry | Globális |5 – 50 |Kis- és nagybetűk megkülönböztetése nélkül | Alfanumerikus karakterek |`<service short name>registry` |`app1registry` |

## <a name="organize-resources-with-tags"></a>Erőforrások rendszerezése címkékkel

Az Azure Resource Manager támogatja a tetszőleges szöveges sztringek használatát az entitások címkézésekor a környezet azonosítása és az automatizálás megkönnyítése érdekében. Ha például a címke `"sqlVersion"="sql2014ee"` azonosíthatóak a virtuális gépeken futó SQL Server 2014 Enterprise Edition. A címkék a választott elnevezési konvenciókkal együtt kiegészíthetik és pontosíthatják a környezeti információkat.

> [!TIP]
> A címkék használatának másik előnye, hogy több erőforráscsoportra is érvényesek, ezáltal lehetővé teszik az eltérő környezetekben lévő entitások összekötését.

Minden erőforrás vagy erőforráscsoport legfeljebb **15** címkével rendelkezhet. A címke neve legfeljebb 512 karakter, a címke értéke pedig legfeljebb 256 karakter hosszúságú lehet.

Az erőforrások címkézéséről [Az Azure-erőforrások címkék használatával történő rendszerezését](/azure/azure-resource-manager/resource-group-using-tags/) ismertető cikkben talál további információt.

A címkéket például a következő esetekben szokták gyakran használni:

- **Számlázási**. Erőforrások csoportosítása és társítása számlázási vagy költséghelyi biztonsági kódokat.
- **Szolgáltatás környezetének azonosítása**. Több erőforráscsoportban található erőforrások csoportjainak azonosítása gyakori műveletekhez és csoportosítás.
- **Hozzáférés-vezérlési és biztonsági környezet**. Rendszergazdai szerepkör azonosítása portfólió, rendszer, szolgáltatás, alkalmazás, példány stb alapján.

> [!TIP]
> Címke korai, címkét. Jobb, ha elsőként egy alapszintű címkézési sémát használ, amelyet később az igényeihez igazíthat, mintha az egészet utólag kell elvégeznie.

Néhány gyakori címkézési módszer bemutatása példákkal:

| Címke neve | Kulcs | Példa | Megjegyzés |
| --- | --- | --- | --- |
| Számlázás / Belső költséghelyi elszámolás azonosítója |BillTo |`IT-Chargeback-1234` |Belső I/O vagy számlázási kód |
| Kezelő vagy közvetlenül felelős személy (DRI) |managedBy |`joe@contoso.com` |Alias vagy e-mail-cím |
| Projekt neve |projectName |`myproject` |A projekt vagy termékvonal neve |
| Projekt verziója |projectVersion |`3.4` |A projekt vagy termékvonal verziója |
| Környezet |environment |`<Production, Staging, QA >` |Környezeti azonosító |
| Szint |tier |`Front End, Back End, Data` |Szint vagy szerepkör/környezet azonosítója |
| Adatprofil |dataProfile |`Public, Confidential, Restricted, Internal` |Az erőforrásban tárolt adatok bizalmassága |

## <a name="tips-and-tricks"></a>Tippek és trükkök

Bizonyos erőforrástípusok elnevezése és konvenciói nagyobb figyelmet igényelhetnek.

### <a name="virtual-machines"></a>Virtual machines (Virtuális gépek)

Különösen nagyobb topológiák esetén a virtuális gépek jól átgondolt elnevezésével könnyebben azonosítható az egyes gépek szerepe és feladata, ami kiszámíthatóbb szkripthasználatot eredményez.

### <a name="storage-accounts-and-storage-entities"></a>Tárfiókok és tárolóentitások

Nincsenek tárfiókok két elsődleges használati eset: a virtuális gépek lemezeinek biztonsági mentése, valamint adatok tárolása a blobokhoz, üzenetsorokhoz és táblákhoz. A virtuális gépek lemezeihez használt tárfiókoknak követniük kell azt az elnevezési konvenciót, amely a szülő virtuális géphez társítja őket (és mivel a csúcskategóriás virtuális gépek termékváltozataihoz több tárfiókra is szükség lehet, használjon numerikus utótagot).

> [!TIP]
> A tárfiókoknak (akár adatokat, akár lemezeket tárolnak) olyan elnevezési konvenciót kell követniük, amely lehetővé teszi több tárfiók használatát (vagyis mindig numerikus utótagot kell használniuk).

Konfigurálhat például egy egyedi tartománynevet az Azure Storage-fiókjában tárolt blobadatokhoz való hozzáféréshez. A blobszolgáltatás alapértelmezett végpontja van `https://<name>.blob.core.windows.net`.

De ha egy egyéni tartomány leképezése (például `www.contoso.com`), a tárfiók blobvégpontjához, is elérhető említett tartomány használatával a tárfiókban lévő blobadatokat. Egyéni tartománynév használatával például a `https://mystorage.blob.core.windows.net/mycontainer/myblob` a `https://www.contoso.com/mycontainer/myblob` címen is elérhető.

A szolgáltatás konfigurálásáról az [egyéni tartományév Blob Storage-végponthoz való konfigurálását](/azure/storage/storage-custom-domain-name/) ismertető cikkben talál további információt.

A blobok, tárolók és táblák elnevezéséről a következő listában talál további információt:

- [Tárolók, blobok és metaadatok elnevezése és hivatkozása](https://msdn.microsoft.com/library/dd135715.aspx)
- [Üzenetsorok és metaadatok elnevezése](https://msdn.microsoft.com/library/dd179349.aspx)
- [Táblák elnevezése](https://msdn.microsoft.com/library/azure/dd179338.aspx)

A blobnév bármilyen karakterkombinációt tartalmazhat, de a foglalt URL-karaktereket escape-karakterrel kell jelölni. Ne használjon olyan blobneveket, amelyek ponttal (.), perjellel (/) vagy a kettő sorozatával/kombinációjával végződnek. A perjel egyezményesen a *virtuális* könyvtárak elválasztására szolgál. Ne használjon fordított perjelet (\\) a blob nevében. Lehetséges, hogy az ügyfél API-k engedélyezik a használatát, a kivonatolás azonban sikertelen lehet, az aláírások pedig nem fognak egyezni.

A tárfiókok és a tárolók neve a létrehozásuk után nem módosítható. Ha új nevet kíván használni, törölnie kell, majd újra létre kell hoznia az adott elemet.

> [!TIP]
> Javasoljuk, hogy mielőtt belevágna az új szolgáltatás vagy alkalmazás fejlesztésébe, alakítson ki egy minden tárfiókra és típusra vonatkozó elnevezési konvenciót.

