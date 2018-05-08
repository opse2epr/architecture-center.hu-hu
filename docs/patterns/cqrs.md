---
title: CQRS
description: Különböző felületek használatával elkülönítheti az adatolvasó műveleteket az adatfrissítő műveletektől.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: c2832aa806909c6f0aab8b6345ffb8162eb59903
ms.sourcegitcommit: d08f6ee27e1e8a623aeee32d298e616bc9bb87ff
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 05/07/2018
---
# <a name="command-and-query-responsibility-segregation-cqrs-pattern"></a>Command and Query Responsibility Segregation (CQRS) minta

[!INCLUDE [header](../_includes/header.md)]

Különböző felületek használatával elkülönítheti az adatolvasó műveleteket az adatfrissítő műveletektől. Ezzel maximalizálhatja a teljesítményt, a méretezhetőséget és a biztonságot. Támogatja a rendszer fejlődéséhez időbeli keresztül nagyobb rugalmasságot biztosít, és megakadályozza, hogy a frissítési parancsok, amely a tartományi szinten egyesítési ütközések.

## <a name="context-and-problem"></a>Kontextus és probléma

A hagyományos adatkezelő rendszerek esetében mind a parancsokat (az adatok frissítéseit), mind a lekérdezéseket (az adatokra irányuló kéréseket) ugyanazon az egyetlen adattárban található entitáskészleten hajtja végre a rendszer. Ezek az entitások lehetnek egy az SQL Serverhez hasonló relációs adatbázisban található egy vagy több tábla soraiból álló halmazok.

Ezekben a rendszerekben általában minden létrehozási, olvasási, frissítési és törlési (CRUD) művelet az entitás ugyanazon formáján mennek végbe. Például az adat-hozzáférési réteg (DAL) beolvas egy ügyfelet képviselő adatátviteli objektumot (DTO-t) az adattárból, és az megjelenik a képernyőn. A felhasználó frissíti a DTO egyes mezőit (például adatkötéssel), majd a DAL ismételten menti DTO-t az adattárban. A rendszer az olvasási és írási műveletetekhez is ugyanazt a DTO-t használja. Az ábrán egy hagyományos CRUD-architektúra látható.

![Hagyományos CRUD-architektúra](./_images/command-and-query-responsibility-segregation-cqrs-tradition-crud.png)

A hagyományos CRUD-kialakítások akkor működnek jól, ha csak korlátozott üzleti logikát alkalmaz az adatműveletekre. A fejlesztői eszközök által biztosított felépítő mechanizmus nagyon gyorsan képes az adat-hozzáférési kód létrehozására, amely aztán szükség szerint testreszabható.

A hagyományos CRUD megközelítésnek azonban néhány hátránya is van:

- Ez gyakran azt jelenti, hogy az adatok írási és olvasási ábrázolása nem egyezik, például az egyik több oszloppal vagy tulajdonsággal rendelkezik, amelyeket akkor is megfelelően kell frissíteni, ha nem szükségesek egy adott művelethez.

- Fennáll az adatok közti verseny kialakulásának veszélye, ha a rekordok egy olyan együttműködési tartomány adattárában vannak zárolva, amelyben ugyanazon az adatkészleten párhuzamosan több aktor is végez műveleteket. Illetve ha optimista zárolás használatakor az egyidejű frissítések ütköznek egymással. Az említett kockázatok esélye a rendszer összetettségének és teljesítményének növekedésével egyre nagyobb lesz. Emellett a hagyományos megközelítés negatív hatással lehet a teljesítményre az adattárra és az adatelérési rétegre eső terhelés, valamint a lekérdezések az adatok beolvasásához szükséges bonyolultsága miatt.

- Ez bonyolultabbá teheti a biztonság és az engedélyek kezelését, mivel minden entitáson végrehajthatók olvasási és írási műveletek is, ezáltal az adatok nem megfelelő kontextusban is megjelenhetnek.

> A CRUD megközelítés korlátairól részletesebben a [CRUD – csak akkor, ha megengedheti](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/) című cikkben olvashat.

## <a name="solution"></a>Megoldás

A Command and Query Responsibility Segregation (CQRS) olyan minta, amely külön felületek használatával elválasztja az adatolvasási műveleteket (lekérdezéseket) az adatfrissítési műveletektől (parancsoktól). Ez azt jelenti, hogy a lekérdezésekhez és a frissítésekhez nem ugyanazt az adatmodellt használja. A modellek az ábrán látható módon elkülöníthetők, bár ez nem alapvető követelmény.

![Alapszintű CQRS-architektúra](./_images/command-and-query-responsibility-segregation-cqrs-basic.png)

A CRUD-alapú rendszerekben használt egyetlen adatmodellhez képest a CQRS-alapú rendszerekben a külön lekérdezési és frissítési modellek használata egyszerűbbé teszi a tervezést és a megvalósítást. Azonban ennek egyik hátránya, hogy a CRUD-kialakításoktól eltérően a CQRS-kód nem hozható létre automatikusan felépítő mechanizmusok használatával.

Az olvasási adatok lekérdezési modellje és az írási adatok frissítési modellje ugyanahhoz a fizikai tárolóhoz fér hozzá, például az SQL-nézetek vagy menet közben létrehozott leképezések használatával. Gyakori eljárás azonban az adatok külön fizikai tárolóban való elkülönítése a teljesítmény, a méretezhetőség és a biztonság maximalizálása érdekében, ahogy azt a következő ábra is mutatja.

![CQRS-architektúra külön olvasási és írási tárolókkal](./_images/command-and-query-responsibility-segregation-cqrs-separate-stores.png)

Az olvasási tároló lehet az írási tároló csak olvasható replikája, de az olvasási és írási tárolók lehetnek teljesen eltérő szerkezetűek is. Az olvasási tároló több, csak olvasható replikájának használatával jelentősen növelhető a lekérdezések teljesítménye és az alkalmazás felhasználói felületének válaszideje, különösen az elosztott forgatókönyvekben, amelyekben a csak olvasható replikák az alkalmazáspéldányokhoz közel találhatók. Egyes adatbázis-rendszerek (SQL Server) további funkciókat kínálnak, például a rendelkezésre állás maximalizálását szolgáló feladatátvételi replikákat.

Az olvasási és írási tárolók különválasztása lehetővé teszi a terhelésnek megfelelő méretezésüket is. Az olvasási tárolók például jellemzően jóval nagyobb terhelésnek vannak kitéve, mint az írási tárolók.

Ha a lekérdezési/olvasási modell denormalizált adatokat tartalmaz (lásd: [Materialized View minta](materialized-view.md)), akkor a teljesítmény az alkalmazásban szereplő egyes nézetek adatainak beolvasásakor vagy a rendszerbeli adatok lekérdezésekor maximalizálható.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

- Az adattárak az olvasási és írási műveletek számára külön fizikai tárolókban való elosztásával növelhető a rendszer teljesítménye és biztonsága, azonban ez a rugalmasság és a végleges konzisztencia szempontjából bonyolíthatja a helyzetet. Az olvasási modell tárolóját frissíteni kell ahhoz, hogy megjelenjenek benne az írási modellen végzett módosítások, így nehéz lehet megállapítani, ha egy felhasználó elavult olvasási adatokon alapuló kérelmet bocsát ki, ami azt jelenti, hogy a művelet nem végezhető el.

    > A végleges konzisztencia leírása az [Adatkonzisztencia – Ismertető](https://msdn.microsoft.com/library/dn589800.aspx) című cikkben található.

- A CQRS-t érdemes lehet a rendszernek csak az olyan korlátozott részein használni, ahol a leginkább hasznos.

- A végleges konzisztencia kialakításának általános megközelítése az Event Sourcing és a CQRS együttes használata. Ily módon az írási modell egy hozzáfűzéssel bővíthető eseménystream, amelyet a parancsok végrehajtása vezérel. Ezek az események az olvasási modellként működő materializált nézetek frissítéséhez használatosak. Bővebb információ: [Event Sourcing és CQRS](/azure/architecture/patterns/cqrs#event-sourcing-and-cqrs).

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Az alábbi helyzetekben alkalmazza ezt a mintát:

- Együttműködési tartományok, ahol ugyanazokon az adatokon egyszerre több műveletet is végrehajt a rendszer. A CQRS révén megfelelő részletességgel adhatók meg a parancsok a tartományi szint egyesítési ütközéseinek minimalizálása érdekében (bármely fellépő ütközés egyesíthető a parancs által), még akkor is, ha látszólag ugyanazon adattípus frissítéséről van szó.

- Feladatalapú felhasználói felületek, ahol a felhasználók lépésenként haladnak végig egy összetett folyamaton, vagy összetett tartományi modellekkel. Ezenkívül olyan csapatok számára is hasznos, akik már ismerik a tartományvezérelt tervezési (DDD) technikákat. Az írási modell egy üzleti logikát, bemenet-ellenőrzést és üzleti ellenőrzést tartalmazó teljes parancsfeldolgozási veremmel rendelkezik annak biztosítása érdekében, hogy mindig minden konzisztens legyen az írási modell minden egyes összesítésével (a társított objektumok minden egyes fürtjét az adatok változásának egységeként kezeli). Az olvasási modell nem tartalmaz üzleti logikát és érvényesítési vermet, csupán egy DTO-t ad vissza a nézetmodellben való használatra. Az olvasási modell véglegesen konzisztens az írási modellel.

- Olyan forgatókönyvek, amelyekben az adatolvasási teljesítményt az adatírási teljesítménytől külön kell finomhangolni, különösen, ha az olvasási/írási arány nagyon magas, illetve ha horizontális skálázásra van szükség. Az olvasási műveletek száma például számos rendszerben sokszorta több az írási műveletek számánál. Ezen igény kiszolgálása érdekében fontolja meg az olvasási modell kiterjesztését, az írási modellt viszont egyetlen, vagy mindössze néhány példányon futtassa. Alacsony számú írási modell-példány használatával csökkenthető az egyesítési ütközések előfordulása is.

- Olyan forgatókönyvek, amelyekben egy fejlesztői csapat az írási modell részét képező összetett tartományi modellre összpontosíthat, egy másik csapat pedig az olvasási modellre és a felhasználói felületekre.

- Olyan forgatókönyvek, amelyekben a rendszer az idő előrehaladtával várhatóan fejlődni fog, így a modell több verzióját is tartalmazhatja, illetve ahol az üzleti szabályok rendszeresen változnak.

- Integráció más rendszerekkel, különösen az Event Sourcinggal kombinálva, amely esetben egy alrendszer átmeneti meghibásodása nem lehet hatással a többi alrendszer rendelkezésre állására.

A minta használata a következő esetekben nem ajánlott:

- Ahol a tartomány vagy az üzleti szabályok egyszerűek.

- Ahol egy egyszerű CRUD stílusú felhasználói felület és az ahhoz kapcsolódó adat-hozzáférési műveletek is elegendőek.

- A teljes rendszeren történő megvalósítás esetén. Egy általános adatkezelési forgatókönyvnek vannak olyan adott összetevői, amelyek esetében a CQRS hasznos lehet, viszont számottevő és szükségtelen összetettséget jelenthet, ha nincs rá igazán szükség.

## <a name="event-sourcing-and-cqrs"></a>Az Event Sourcing és a CQRS

A CQRS mintát gyakran használják az Event Sourcing mintával együtt. A CQRS-alapú rendszerek külön olvasási és írási adatmodelleket használnak, amelyek mindegyike a kapcsolódó feladatokhoz van igazítva, és gyakran fizikailag elkülönített tárolóban találhatók. Az [Event Sourcing](event-sourcing.md) minta használata esetén az események tárolója és az információk hivatalos forrása az írási modell. A CQRS-alapú rendszerek olvasási modellje az adatok materializált nézetit biztosítja, általában nagy mértékben denormalizált nézetekként. Ezek a nézetek az alkalmazás felhasználói felületeihez és megjelenítési követelményeihez vannak igazítva, ezzel pedig maximalizálható a megjelenítési és lekérdezési teljesítmény.

Ha egy adott időpont aktuális adatai helyett az eseménystreamet használja írási tárolóként, azzal elkerülhetők az egyetlen összesítésen felmerülő frissítési ütközések, valamint maximalizálható a teljesítmény és a méretezhetőség. Az események az olvasási tároló feltöltéséhez használt adatok materializált nézetének aszinkron létrehozásához használhatók.

Mivel az eseménytároló az információk hivatalos forrása, a materializált nézetek törölhetők, és a rendszer fejlődésekor az összes múltbeli esemény visszajátszható a jelenlegi állapot új ábrázolásának létrehozása érdekében, illetve ha az olvasási modell módosítására van szükség. A materializált nézetek lényegében az adatok tartós, csak olvasható gyorsítótáraként működnek.

A CQRS és az Event Sourcing minta együttes használatakor vegye figyelembe az alábbiakat:

- Mint minden olyan rendszernél, ahol az írási és az olvasási adattárak elkülönülnek, az ezen mintán alapuló rendszerek esetében csak a végleges konzisztencia valósul meg. Az esemény létrehozása és az adattár frissítése között bizonyos fokú késésre lehet számítani.

- A minta összetetté teszi a rendszert, mivel kód létrehozására van szükség az események indításához és kezeléséhez, valamint a lekérdezésekhez vagy az olvasási modellhez szükséges megfelelő nézetek vagy objektumok összeállításához vagy frissítéséhez. A CQRS minta az Event Sourcing mintával való együttes használatából eredő összetettség megnehezítheti a sikeres megvalósítást, és a rendszerek tervezése szempontjából is eltérő megközelítést igényel. Az Event Sourcing azonban megkönnyítheti a tartomány modellezését, illetve a nézetek újraépítését vagy új nézetek létrehozását, mivel az adatok módosításainak célját megőrzi.

- A materializált nézetek létrehozásához jelentős feldolgozási időre és erőforrás-használatra van szükség, ha azokat az adott entitások vagy entitásgyűjtemények eseményeinek visszajátszásával és kezelésével az olvasási modellben vagy az adatok leképezéseiben használja. Ez különösen akkor igaz, ha hosszú időszakok értékeinek összesítésére vagy elemzésére van szükség, mivel előfordulhat, hogy minden kapcsolódó eseményt meg kell vizsgálni. Ezt megoldhatja az adatok üzemezett időközönként elkészített pillanatképeinek használatával, például egy adott művelet elvégzéseinek összesített számával vagy egy entitás aktuális állapotával.

## <a name="example"></a>Példa

Az alábbi kód részleteket mutat be egy CQRS-megvalósítási példából, amely külön definíciót használ az olvasási és írási modellekhez. A modellfelületek nem határozzák meg az alapul szolgáló adattárak jellemzőit, továbbá a felületek elkülönülése miatt a fejlődésük és finomhangolásuk is függetlenül valósulhat meg.

A következő kód bemutatja az olvasási modell definícióját.

```csharp
// Query interface
namespace ReadModel
{
  public interface ProductsDao
  {
    ProductDisplay FindById(int productId);
    ICollection<ProductDisplay> FindByName(string name);
    ICollection<ProductInventory> FindOutOfStockProducts();
    ICollection<ProductDisplay> FindRelatedProducts(int productId);
  }

  public class ProductDisplay
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal UnitPrice { get; set; }
    public bool IsOutOfStock { get; set; }
    public double UserRating { get; set; }
  }

  public class ProductInventory
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public int CurrentStock { get; set; }
  }
}
```

A rendszer lehetővé teszi a felhasználóknak számára a termékek értékelését. Az alkalmazás kódja ezt az alábbi kódban szereplő `RateProduct` parancs használatával éri el.

```csharp
public interface ICommand
{
  Guid Id { get; }
}

public class RateProduct : ICommand
{
  public RateProduct()
  {
    this.Id = Guid.NewGuid();
  }
  public Guid Id { get; set; }
  public int ProductId { get; set; }
  public int Rating { get; set; }
  public int UserId {get; set; }
}
```

A rendszer a `ProductsCommandHandler` osztályt használja az alkalmazás által küldött parancsok kezelésére. Az ügyfelek általában egy üzenetkezelő rendszeren, például egy üzenetsoron keresztül küldenek a parancsokat a tartománynak. A parancskezelő fogadja ezeket a parancsokat, és meghívja a tartományi felület metódusait. Az egyes parancsok részletességének célja a kérések ütközési esélyeinek csökkentése. A következő kód a `ProductsCommandHandler` osztály vázlatát mutatja be.

```csharp
public class ProductsCommandHandler :
    ICommandHandler<AddNewProduct>,
    ICommandHandler<RateProduct>,
    ICommandHandler<AddToInventory>,
    ICommandHandler<ConfirmItemShipped>,
    ICommandHandler<UpdateStockFromInventoryRecount>
{
  private readonly IRepository<Product> repository;

  public ProductsCommandHandler (IRepository<Product> repository)
  {
    this.repository = repository;
  }

  void Handle (AddNewProduct command)
  {
    ...
  }

  void Handle (RateProduct command)
  {
    var product = repository.Find(command.ProductId);
    if (product != null)
    {
      product.RateProduct(command.UserId, command.Rating);
      repository.Save(product);
    }
  }

  void Handle (AddToInventory command)
  {
    ...
  }

  void Handle (ConfirmItemsShipped command)
  {
    ...
  }

  void Handle (UpdateStockFromInventoryRecount command)
  {
    ...
  }
}
```

A következő kód az írási modell `IProductsDomain` felületét mutatja be.

```csharp
public interface IProductsDomain
{
  void AddNewProduct(int id, string name, string description, decimal price);
  void RateProduct(int userId, int rating);
  void AddToInventory(int productId, int quantity);
  void ConfirmItemsShipped(int productId, int quantity);
  void UpdateStockFromInventoryRecount(int productId, int updatedQuantity);
}
```

Azt is figyelje meg, hogy az `IProductsDomain` felület olyan metódusokat tartalmaz, amelyek jelentéssel bírnak a tartományban. Egy CRUD-környezetben ezek a metódusok a legtöbb esetben általános névvel lennénk ellátva (például `Save` vagy `Update`), és az egyetlen argumentumuk egy DTO lenne. A CQRS-megközelítés kialakítható úgy, hogy megfeleljen a vállalat üzleti és leltárkezelő rendszerei igényeinek.

## <a name="related-patterns-and-guidance"></a>Kapcsolódó minták és útmutatók

Az alábbi minták és útmutatások hasznosak lehetnek a minta használatakor:

- A CQRS és más architektúrastílusok összehasonlításáért lásd: [Architektúrastílusok](/azure/architecture/guide/architecture-styles/) és [CQRS architektúrastílus](/azure/architecture/guide/architecture-styles/cqrs).

- [Adatkonzisztencia – Ismertető](https://msdn.microsoft.com/library/dn589800.aspx). Részletezi az olvasási és írási adattárak végleges konzisztenciája által okozott általánosan felmerülő problémákat a CQRS minta használatakor, illetve azt is, hogy ezek a problémák hogyan oldhatók meg.

- [Adatparticionálási útmutató](https://msdn.microsoft.com/library/dn589795.aspx). Ismerteti, hogyan oszthatók fel a CQRS mintában használt olvasási és írási adattárak olyan partíciókra, amelyek a méretezhetőség, a versenyhelyzetek kialakulásának csökkentése és a teljesítmény optimalizálása érdekében külön kezelhetők és érhetők el.

- [Event Sourcing minta](event-sourcing.md). Részletesebben ismerteti, hogyan használható az Event Sourcing a CQRS mintával a feladatok egyszerűsítéséhez az összetett tartományokban a teljesítmény, a méretezhetőség és a válaszkészség növelése mellett. Emellett azt is bemutatja, hogyan biztosítható a tranzakciós adatok konzisztenciája a teljes körű naplók és előzmények fenntartása mellett, ami lehetővé teszi a kompenzáló műveletek végrehajtását.

- [A Materialized View minta](materialized-view.md). A CQRS megvalósítás olvasási modellje tartalmazhatja az írási modell adatainak materializált nézeteit, illetve a materializált nézetek létrehozására is használható.

- Minták és gyakorlatok útmutatója – [A CQRS felfedezése](http://aka.ms/cqrs). [A Command Query Responsibility Segregation minta](https://msdn.microsoft.com/library/jj591573.aspx) című cikk bemutatja a mintát, illetve részletesen taglalja, hogy mikor érdemes azt használni, az [Epilógus: tapasztalatok](https://msdn.microsoft.com/library/jj591568.aspx) című cikk pedig segít a minta használata során felmerülő problémák egy részének megértésében.

- A [Martin Fowler – CQRS](http://martinfowler.com/bliki/CQRS.html) című bejegyzés ismerteti a minta alapvető működését, továbbá egyéb hasznos források hivatkozásait is tartalmazza.

- Greg Young [bejegyzéseiben](http://codebetter.com/gregyoung/) számos szempontból járja körül a CQRS mintát.
