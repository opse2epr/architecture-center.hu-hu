---
title: "Azure-erőforrások elnevezési szabályai"
description: "Azure-erőforrások elnevezési szabályainak. Virtuális gépek neve, storage-fiókok, hálózatok, virtuális hálózatok, alhálózatok és egyéb Azure entitások"
author: telmosampaio
ms.date: 05/18/2017
pnp.series.title: Best Practices
ms.openlocfilehash: 20f090c4cc39f96887cd58ffd2a9c7736e77de57
ms.sourcegitcommit: 1c0465cea4ceb9ba9bb5e8f1a8a04d3ba2fa5acd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/02/2018
---
# <a name="naming-conventions"></a>Elnevezési konvenciók

[!INCLUDE [header](../_includes/header.md)]

Ez a cikk az elnevezési szabályok és az Azure-erőforrások korlátozások összefoglalását és egy alapkonfiguráció elnevezési konvencióira vonatkozó javaslatok szerepelnek.  Használható ezek a javaslatok kiindulási pontként a saját konkrét egyezmények az igényeinek megfelelően.

A Microsoft Azure-erőforrásoknál nevét a választott fontos, mert:

* Is nehézkes később módosíthatja a nevet.
* Nevének meg kell felelnie az adott erőforrástípus követelményeinek.

Egységes elnevezési konvenciókat könnyebben erőforrások megkereséséhez. Ezek a szerepkör erőforrás-megoldásban is utalhat.

A kulcs az elnevezési konvenciókat sikeres létrehozó és a következő azokat az alkalmazásokat és a szervezetek között.

## <a name="naming-subscriptions"></a>Elnevezési előfizetések
Azure-előfizetések elnevezésekor részletes nevek ellenőrizze, és az egyes előfizetések törlése céljának ismertetése.  Sok előfizetések környezetben működik, a következő megosztott elnevezési javítható átláthatóság érdekében.

Az ajánlott mintázatát elnevezési előfizetések van:

`<Company> <Department (optional)> <Product Line (optional)> <Environment>`

* Vállalati kell minden egyes előfizetés esetén. Egyes vállalatok azonban állhat gyermek vállalataihoz szervezeti struktúráját. Ezeknek a vállalatoknak egy központi informatikai csoport által is kezelhető. Ebben az esetben azok sikerült meghatározni azzal, hogy mind a szülő vállalat neve (*Contoso*) és a gyermek Cégnév (*Northwind*).
* Részleg egy nevet a szervezeten belül, amikor egy csoport működik. Az elem nem kötelező, a névtéren belül.
* Sor a termék vagy a részleg alkalmazottai végzett függvény adott névvel. Ez nem a belső hálózati szolgáltatások és alkalmazások általában kötelező. Azonban ajánlott könnyen elkülönítését és azonosító (például számlázási rekordok egyértelmű elkülönítése mint) igénylő nyilvánosan elérhető szolgáltatások használatára.
* A környezete az alkalmazások vagy szolgáltatások, például Dev, a QA vagy a termék telepítési életciklus leíró nevet.

| Vállalat | Részleg | Sor termék vagy szolgáltatás | Környezet | Teljes név |
| --- | --- | --- | --- | --- |
| Contoso |SocialGaming |AwesomeService |Éles |Contoso SocialGaming AwesomeService éles |
| Contoso |SocialGaming |AwesomeService |Fejlesztői |Contoso SocialGaming AwesomeService fejlesztői |
| Contoso |IT |InternalApps |Éles |Contoso informatikai InternalApps éles |
| Contoso |IT |InternalApps |Fejlesztői |Contoso informatikai InternalApps fejlesztői |

További információk a nagyobb vállalatok előfizetések rendezésének módját, a [irányítás útmutató részletes utasításokkal megadott előfizetés][scaffold].

## <a name="use-affixes-to-avoid-ambiguity"></a>Használja a félreérthetőség elkerülése érdekében kell

Az Azure-erőforrások elnevezésekor közös előtagok vagy utótagok segítségével azonosíthatja, típusa és az erőforrás környezetben ajánlott.  Információkat a típusát, a metaadatok, miközben a környezetben, elérhető programozott módon, közös elő-/ utótagok alkalmazása egyszerűbbé teszi a vizuális azonosítása.  Ha elő-/ utótagok beépítése az elnevezési, fontos egyértelműen adja meg, hogy a utótag (előtag) nevének elején vagy végén (utótag).

Például az alábbiakban egy szolgáltatáshoz, egy számítási program üzemeltető két lehetséges név:

* SvcCalculationEngine (előtag)
* CalculationEngineSvc (utótag)

A konkrét erőforrásokat leíró különböző szempontjairól elő-/ utótagok hivatkozhat. Az alábbi táblázat néhány példát mutat általában akkor használható.

| Aspektusa | Példa | Megjegyzések |
| --- | --- | --- |
| Környezet |fejlesztői, termék, QA |Azonosítja a környezetben, az erőforrás |
| Tartózkodási hely |UW (US Nyugat), ue (amerikai keleti) |Azonosítja a régió, ahol az erőforrás van telepítve |
| Példány |01, 02 |Az erőforrások, amelyeken egynél több megnevezett példány (webkiszolgálók, stb.). |
| Termék vagy szolgáltatás |szolgáltatás |Azonosítja a termék, alkalmazás vagy szolgáltatás, amely az erőforrás támogatja |
| Szerepkör |SQL, webalkalmazás, üzenetküldés |Azonosítja a kapcsolódó erőforrás szerepe |

A vállalat vagy a projektek adott elnevezési fejleszt, esetén fontos a Válasszon olyan közös elő-/ utótagok és pozíciójuk (utótag vagy előtag).

## <a name="naming-rules-and-restrictions"></a>Elnevezési szabályok és korlátozások

Az Azure-ban minden egyes erőforrásokhoz vagy szolgáltatásokhoz típus érvénybe lépteti a korlátozások és a hatókör; elnevezési készlete bármely elnevezési vagy -mintát meg kell felelnie a szükséges elnevezési szabályok és a hatókör.  Például egy virtuális gép nevét egy DNS-név van leképezve (és így egyedinek kell lennie az összes Azure szükséges), a virtuális hálózat nevét hatókörét az erőforráscsoport, amely akkor jön létre.

Általában kerülje az olyan speciális karakterek (`-` vagy `_`) bármely nevét az első vagy utolsó karakterként. Ezek a karakterek, akkor sikertelen lesz a legtöbb ellenőrzési szabályok.

### <a name="general"></a>Általános

| Entitás | Hatókör | Hossz | Kis-és nagybetűk | Érvényes karaktereket | Javasolt minta | Példa |
| --- | --- | --- | --- | --- | --- | --- |
|Erőforráscsoport |Előfizetés |1-90 |Kis-és nagybetűk megkülönböztetése nélkül |Alfanumerikus, aláhúzásjelet, kerek zárójeleket tartalmazhatnak, kötőjelet és időszak (kivéve záró) |`<service short name>-<environment>-rg` |`profx-prod-rg` |
|Rendelkezésre állási csoport |Erőforráscsoport |1-80 |Kis-és nagybetűk megkülönböztetése nélkül |Alfanumerikus, aláhúzásjelet és kötőjel |`<service-short-name>-<context>-as` |`profx-sql-as` |
|Címke |Társított entitás |512 (név), 256 (érték) |Kis-és nagybetűk megkülönböztetése nélkül |Alfanumerikus |`"key" : "value"` |`"department" : "Central IT"` |

### <a name="compute"></a>Compute

| Entitás | Hatókör | Hossz | Kis-és nagybetűk | Érvényes karaktereket | Javasolt minta | Példa |
| --- | --- | --- | --- | --- | --- | --- |
|Virtuális gép |Erőforráscsoport |1 – 15 (Windows), 1-64 (Linux) |Kis-és nagybetűk megkülönböztetése nélkül |Alfanumerikus, aláhúzásjelet és kötőjel |`<name>-<role>-vm<number>` |`profx-sql-vm1` |
|Függvényalkalmazás | Globális |1-60 |Kis-és nagybetűk megkülönböztetése nélkül |Alfanumerikus és kötőjelet tartalmazhat |`<name>-func` |`calcprofit-func` |

> [!NOTE]
> Virtuális gépek Azure-ban két különböző névvel rendelkezik: virtuális gép nevét, és az állomásnevet. Amikor a virtuális gép létrehozása a portálon, azonos nevű szolgál az állomásnevet és a virtuálisgép-erőforrás nevét. A fenti korlátozások vonatkoznak a névhez. A tényleges erőforrás neve legfeljebb 64 karakter hosszúságú lehet.

### <a name="storage"></a>Tárhely

| Entitás | Hatókör | Hossz | Kis-és nagybetűk | Érvényes karaktereket | Javasolt minta | Példa |
| --- | --- | --- | --- | --- | --- | --- |
|A tárfiók neve (adatok) |Globális |3-24 |kisbetűk |Alfanumerikus |`<globally unique name><number>`(függvény használható storage-fiókok elnevezési egyedi guid kiszámításához) |`profxdata001` |
|A tárfiók neve (lemez) |Globális |3-24 |kisbetűk |Alfanumerikus |`<vm name without dashes>st<number>` |`profxsql001st0` |
| Tároló neve |Tárfiók |3-63 |kisbetűk |Alfanumerikus, valamint a kötőjel |`<context>` |`logs` |
|Blob neve | Tároló |1-1024 |Kis-és nagybetűket |A URL-cím karaktereket |`<variable based on blob usage>` |`<variable based on blob usage>` |
|Üzenetsor neve |Tárfiók |3-63 |kisbetűk |Alfanumerikus, valamint a kötőjel |`<service short name>-<context>-<num>` |`awesomeservice-messages-001` |
|Tábla neve | Tárfiók |3-63 |Kis-és nagybetűk megkülönböztetése nélkül |Alfanumerikus |`<service short name><context>` |`awesomeservicelogs` |
|Fájl neve | Tárfiók |3-63 |kisbetűk | Alfanumerikus |`<variable based on blob usage>` |`<variable based on blob usage>` |
|Data Lake Store | Globális |3-24 |kisbetűk | Alfanumerikus |`<name>-dls` |`telemetry-dls` |

### <a name="networking"></a>Hálózatkezelés

| Entitás | Hatókör | Hossz | Kis-és nagybetűk | Érvényes karaktereket | Javasolt minta | Példa |
| --- | --- | --- | --- | --- | --- | --- |
|Virtuális hálózathoz (VNet) |Erőforráscsoport |2-64 |Kis-és nagybetűk megkülönböztetése nélkül |Alfanumerikus, kötőjelet, aláhúzásjelet és időszak |`<service short name>-vnet` |`profx-vnet` |
|Alhálózat |Szülő virtuális hálózat |2-80 |Kis-és nagybetűk megkülönböztetése nélkül |Alfanumerikus, aláhúzásjelet, kötőjelet és időszak |`<descriptive context>` |`web` |
|Hálózati adapter |Erőforráscsoport |1-80 |Kis-és nagybetűk megkülönböztetése nélkül |Alfanumerikus, kötőjelet, aláhúzásjelet és időszak |`<vmname>-nic<num>` |`profx-sql1-nic1` |
|Hálózati biztonsági csoport |Erőforráscsoport |1-80 |Kis-és nagybetűk megkülönböztetése nélkül |Alfanumerikus, kötőjelet, aláhúzásjelet és időszak |`<service short name>-<context>-nsg` |`profx-app-nsg` |
|Hálózati biztonsági csoportra vonatkozó szabály |Erőforráscsoport |1-80 |Kis-és nagybetűk megkülönböztetése nélkül |Alfanumerikus, kötőjelet, aláhúzásjelet és időszak |`<descriptive context>` |`sql-allow` |
|Nyilvános IP-cím |Erőforráscsoport |1-80 |Kis-és nagybetűk megkülönböztetése nélkül |Alfanumerikus, kötőjelet, aláhúzásjelet és időszak |`<vm or service name>-pip` |`profx-sql1-pip` |
|Load Balancer |Erőforráscsoport |1-80 |Kis-és nagybetűk megkülönböztetése nélkül |Alfanumerikus, kötőjelet, aláhúzásjelet és időszak |`<service or role>-lb` |`profx-lb` |
|Elosztott terhelésű szabályok Config betöltése |Load Balancer |1-80 |Kis-és nagybetűk megkülönböztetése nélkül |Alfanumerikus, kötőjelet, aláhúzásjelet és időszak |`<descriptive context>` |`http` |
|Azure Application Gateway |Erőforráscsoport |1-80 |Kis-és nagybetűk megkülönböztetése nélkül |Alfanumerikus, kötőjelet, aláhúzásjelet és időszak |`<service or role>-agw` |`profx-agw` |
|Traffic Manager-profil |Erőforráscsoport |1-63 |Kis-és nagybetűk megkülönböztetése nélkül |Alfanumerikus, kötőjelet és időszak |`<descriptive context>` |`app1` |

## <a name="organize-resources-with-tags"></a>Címkékkel rendelkező erőforrások rendszerezése

Az Azure Resource Manager tetszőleges szöveges karakterláncok azonosíthatja környezetben, és automatizálási egyszerűsítésére címkézési entitások támogatja.  Például a címke `"sqlVersion: "sql2014ee"` azonosíthatóak a virtuális gépeken futó SQL Server 2014 Enterprise Edition egy automatizált parancsfájl futtatási előzetesen központi telepítésben.  Címkék választhasson, és fokozza a kiválasztott elnevezési szabályai oldalán környezet használatával.

> [!TIP]
> Egy más címkék előnye, hogy a címkék span erőforráscsoportok, lehetővé téve a hivatkozásra, és különböző központi telepítések egységességét entitások összefüggéseket.

Minden erőforrás vagy erőforráscsoport rendelkezhet legfeljebb **15** címkék. A címke neve legfeljebb 512 karakter, a címke értéke pedig legfeljebb 256 karakter hosszúságú lehet.

További információkat az erőforrás-címkézéssel, [az Azure-erőforrások rendszerezése címkék használatával](/azure/azure-resource-manager/resource-group-using-tags/).

A közös címkézési használati esetek a következők:

* **Számlázási**; Erőforrások csoportosítása és a társítás számlázással vagy az ingyenesen elérhető biztonsági kódokat.
* **Környezeti azonosító szolgáltatás**; Olyan azonosítására erőforrások erőforráscsoportok közötti a gyakori műveletek és csoportosítási
* **Hozzáférés-vezérlési és a biztonsági környezet**; Rendszergazdai szerepkör azonosítója alapján portfóliót, a rendszer, a szolgáltatás, a app, a példány, stb.

> [!TIP]
> Korai - gyakran címke címke.  Szeretné, hogy a rendszer címkézés alapterv jobb, és módosítsa az idő ahelyett, hogy a bekövetkeztek alakítanunk keresztül.

Néhány gyakori címkézési megközelítések példát:

| Címke neve | Kulcs | Példa | Megjegyzés |
| --- | --- | --- | --- |
| / Belső számlázási jóváírási azonosítója |BillTo |`IT-Chargeback-1234` |Egy belső i/o- vagy számlázási kódot |
| Operátor vagy közvetlenül felelős személy (DRI) |Felügyeli |`joe@contoso.com` |Az alias vagy e-mail cím |
| Projekt neve |Projektnév |`myproject` |A projekt vagy termék sor neve |
| Projekt verziója |projectVersion |`3.4` |A projekt vagy termék sor verziója |
| Környezet |Környezet |`<Production, Staging, QA >` |Környezeti azonosítója |
| Díjcsomag |réteg |`Front End, Back End, Data` |Réteg vagy szerepkör/környezetben azonosítása |
| Adatprofil |dataProfile |`Public, Confidential, Restricted, Internal` |Az erőforrás tárolt adatok érzékenységének |

## <a name="tips-and-tricks"></a>Tippek és trükkök

Bizonyos típusú erőforrás a kiosztási és egyezmények további gondot lehet szükség.

### <a name="virtual-machines"></a>Virtuális gépek

Nagyobb topológiákat, különösen a virtuális gépek gondosan elnevezési leegyszerűsíti a szerepkör és a célja az egyes gépek azonosításához, és sokkal kiszámíthatóbbá scripting engedélyezése.

### <a name="storage-accounts-and-storage-entities"></a>Storage-fiókok és a tárolási entitások

Nincsenek két elsődleges használati esetek tárfiókok - lemezek biztonsági virtuális gépekhez, és BLOB, üzenetsorok és táblák adatainak tárolásához.  Tárolás virtuális gépek lemezei által használt fiókok kell hajtsa végre az elnevezési társítása azokat a szülő virtuális gép nevét (és a potenciális kell több storage-fiókok csúcskategóriás VM SKU, is érvényes szám utótag).

> [!TIP]
> Storage-fiókok – az adatok vagy a lemez - érdemes egy elnevezési konvenciója, amely lehetővé teszi több storage-fiókok javítható (azaz mindig használatával egy numerikus utótagot).

Már lehetséges állítson be egy egyéni tartománynevet, az Azure Storage-fiókban lévő Blobadatok eléréséhez. A Blob szolgáltatás az alapértelmezett végpont az https://<name>. blob.core.windows.net ".

Ha egyéni tartományt (például www.contoso.com) csatlakoztat a blob végpontja, a tárfiók, is elérhető, de a tárfiókban lévő adatokat blob-tartomány segítségével. Egy egyéni tartománynevet, például a `http://mystorage.blob.core.windows.net/mycontainer/myblob` érhető el. `http://www.contoso.com/mycontainer/myblob`.

Ez a szolgáltatás konfigurálásával kapcsolatos további információkért tekintse meg [állítson be egy egyéni tartománynevet a Blob storage-végponthoz](/azure/storage/storage-custom-domain-name/).

A BLOB, a tárolók és a táblázatok elnevezési további információkért tekintse meg az alábbi listában:

* [Elnevezésekor és a hivatkozó, tárolók, Blobok és metaadatok](https://msdn.microsoft.com/library/dd135715.aspx)
* [Üzenetsorok és metaadatok elnevezése](https://msdn.microsoft.com/library/dd179349.aspx)
* [Táblák elnevezése](https://msdn.microsoft.com/library/azure/dd179338.aspx)

A blob neve tartalmazhat bármilyen karakter, de foglalt URL-cím karaktereket megfelelően kell megjelölni. Kerülje a blob neveket, amelyek végződhet pontra (.), perjellel (/), vagy egy feladatütemezési, vagy a kettő. A perjel az egyezmény a **virtuális** directory elválasztó. Ne használjon egy fordított perjel (\\) egy blob neve. Az ügyfél API-k előfordulhat, hogy engedélyezi-e, de sikertelen lesz a kivonat-megfelelően, és az aláírás nem fog egyezni.

Nincs módosítható egy tárfiókhoz vagy a tároló neve után lett létrehozva. Ha azt szeretné, hogy egy új nevét, akkor törölje azt, és hozzon létre egy újat.

> [!TIP]
> Azt javasoljuk, hogy kapcsolatot hoz létre az összes storage-fiókok és típusok elnevezési előtt az új szolgáltatás vagy alkalmazás fejlesztésének alkalmazná.

<!-- links -->

[scaffold]: /azure/azure-resource-manager/resource-manager-subscription-governance
