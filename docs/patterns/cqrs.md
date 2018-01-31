---
title: CQRS
description: "Különböző felületek használatával elkülönítheti az adatolvasó műveleteket az adatfrissítő műveletektől."
keywords: "Kialakítási mintája"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: 80f4a8880cf2212acf82dadb67b0181e1cbae099
ms.sourcegitcommit: a7aae13569e165d4e768ce0aaaac154ba612934f
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/30/2018
---
# <a name="command-and-query-responsibility-segregation-cqrs-pattern"></a>Szabály parancs és a lekérdezés felelősségi elkülönítése (CQRS)

[!INCLUDE [header](../_includes/header.md)]

Különböző felületek használatával elkülönítheti az adatolvasó műveleteket az adatfrissítő műveletektől. Maximalizálhatja a teljesítményt, a méretezhetőség és a biztonsági. Támogatja a rendszer fejlődéséhez magasabb rugalmassága időbeli, előfordulhat, hogy a frissítési parancsok, amely a tartományi szinten egyesítési ütközések.

## <a name="context-and-problem"></a>A környezetben, és probléma

Hagyományos felügyeleti rendszerekkel parancsokat (az adatok frissítések) és a lekérdezések (a kérelmek adatok) ugyanazokat a egyetlen tárházban entitások elleni végre. Ezeket az entitásokat is lehet a sorok egy relációs adatbázisban, például az SQL Server egy vagy több táblájában.

Általában az ezekben a rendszerekben összes létrehozása, Olvasás, frissítés és Törlés (CRUD) operations entitás azonos jellemzőkkel is vonatkozik. Például egy átviteli-objektumot (DTO) jelölő, az ügyfél az adatelérési réteg (DAL) által az adatok tárolójából beolvasni, és a képernyőn látható. A felhasználó frissíti a DTO egyes mezőit (lehet, hogy a adatkötés), és a DTO majd kerül vissza az adattár a DAL által. Az azonos DTO szolgál az olvasási és írási műveletet. Az ábra azt mutatja be, egy hagyományos CRUD-architektúrát.

![A hagyományos CRUD-architektúra](./_images/command-and-query-responsibility-segregation-cqrs-tradition-crud.png)

Hagyományos CRUD munkahelyi tervez, megfelelő, ha csak korlátozott üzleti logika vonatkozik, az adatok műveletek. Scaffold mechanizmusok fejlesztői eszközök hozhatnak létre a adatok hozzáférési kód nagyon gyorsan, amely majd testreszabható, amennyiben szükséges.

A hagyományos CRUD megközelítés azonban néhány hátránya rendelkezik:

- Ez gyakran azt jelenti, hogy nem egyeznek az adatokhoz, például további oszlopok vagy frissítenie kell a megfelelő annak ellenére, hogy ezek nem egy művelet kötelező tulajdonságok olvasási és írási ábrázolásai között.

- Az adatok versengés kockázatok, rekordok az együttműködést tartományban, ahol több gyakrabban fog működni, ugyanazokat az adatokat, párhuzamos adattárban zárolásakor. Vagy frissítés, ha optimista zárolás egyidejű frissítések okozza. A kockázatok növelje a összetettségét és -növelés sebességét, a rendszer. Emellett a hagyományos megközelítés teljesítményére negatív hatással lehet az adattárban és az adatelérési réteg betöltése, és a lekérdezések összetettsége adatbeolvasás szükséges.

- Segítségükkel kezelése biztonsági és engedélyek összetettebb mert minden egyes egység közölt olvasási és írási műveletek végrehajtását, amelyek esetleg felfedi a hibás környezetből adatokat.

> Bemutatják a CRUD megközelítés határain, lásd: [CRUD, csak ha akkor is biztosít az](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/).

## <a name="solution"></a>Megoldás

Parancs és a lekérdezés felelősségi elkülönítése (CQRS) származik, a minta-adatok (lekérdezések) olvasása műveletek a rétegmodellről, amely a műveleteket (parancsok) adatokat frissítő külön-felületek használatával. Ez azt jelenti, hogy az adatok modellek lekérdezésekor használja, és frissíti eltérőek. A modellek majd lehet különíteni, bár ez nem egy abszolút követelmény az alábbi ábrán látható módon.

![Egy alapszintű CQRS architektúrája](./_images/command-and-query-responsibility-segregation-cqrs-basic.png)

Az egyetlen adatmodell CRUD-alapú rendszerekben használt képest, CQRS-alapú rendszerek adatainak külön lekérdezési és frissítési modellek használatának egyszerűbbé teszi a tervezési és megvalósítási. Azonban egyik hátránya, hogy eltérően CRUD tervek CQRS kód nem automatikusan jön létre scaffold mechanizmusok használatával.

A lekérdezési modelljét olvasási adatok és a frissítési modell az adatok írása ugyanarra a fizikai tároló, elérheti, lehet, hogy az SQL-nézetek használatával vagy parancsprogramok leképezések létrehozásakor. Azonban általában külön az adatok különböző fizikai tárolók maximalizálása a teljesítmény, méretezhetőség és biztonság, a következő ábrán látható módon.

![Egy, külön olvasási és írási tárolók CQRS architektúrája](./_images/command-and-query-responsibility-segregation-cqrs-separate-stores.png)

Az olvasási tároló lehet egy csak olvasható replika az írási tároló, vagy olvasási és írási tárolókat, hogy egy másik struktúra regisztrálását. Az olvasási tároló több csak olvasható replika használatával jelentősen növelheti lekérdezés teljesítmény- és alkalmazás felhasználói felületén reakcióidőt, különösen az elosztott forgatókönyvek, ahol csak olvasható replika találhatók megközelíti a alkalmazáspéldányok. Egyes adatbázis-rendszerek (SQL Server) adja meg a további funkciók, például a feladatátvételi replikák rendelkezésre állásának maximalizálását.

Olvasási és írási tárolókat elválasztását is lehetővé teszi, hogy mindegyik felel meg a betöltési megfelelően méretezhető. Olvasási tárolók például jellemzően előforduló sokkal nagyobb terhelést, mint a tárolók írása.

Ha a lekérdezés/olvasás modell tartalmaz a nem normalizált adatok (lásd: [materializált nézet mintát](materialized-view.md)), teljesítmény teljes méretű adatok az egyes nézetek egy alkalmazás olvasásakor, vagy a rendszer az adatok lekérdezésekor.

## <a name="issues-and-considerations"></a>Problémákat és szempontok

Ebben a mintában megvalósításához meghatározásakor, vegye figyelembe a következő szempontokat:

- Az adatok felosztásával tárolás külön fizikai tárolók a Read és írási műveletek növelheti a teljesítményt és a rendszer biztonsági, de azt is hozzáadhat a rugalmasság és a végleges konzisztencia tekintetében összetettségi. Az olvasási modell tároló frissíteni kell, hogy az írási modell tároló módosításait, és nehézkes észlelés, ha egy felhasználó egy kérelem alapján elavult adatolvasási, ami azt jelenti, hogy a művelet nem fejezhető ki lehet.

    > A végleges konzisztencia leírását lásd a [adatok konzisztencia ismertetése](https://msdn.microsoft.com/library/dn589800.aspx).

- Érdemes lehet a rendszer korlátozott szakaszok a legyenek számára a legértékesebb CQRS alkalmazni.

- Egy tipikus megoldás telepítésével a végleges konzisztencia, esemény, hogy az írási modell egy hozzáfűző csak az események streamjét a parancsok végrehajtása által vezérelt forrás CQRS együtt használandó. Ezek az események materializált nézet olvasási modellként viselkedő szolgálnak. További információ: [esemény forrás- és CQRS](https://msdn.microsoft.com/library/dn568103.aspx#EventSourcingandCQRS).

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

Használja ezt a mintát a következő esetekben:

- Együttműködési tartományok, ahol több műveleteket, párhuzamos ugyanazokat az adatokat. CQRS adhatók meg minimálisra csökkentése érdekében a tartományi szinten (felmerülő ütközés egyesíthető a parancs), egyesítési ütközések elég lépésköz parancsok is megtalálható az azonos típusú adatok frissítésekor.

- Feladat-alapú felhasználói felületek, ahol felhasználók végigvezeti egy összetett folyamat lépésből áll, vagy összetett tartomány modellekkel való sorozataként. Ezenkívül hasznos csapatának már ismeri a tartomány-központú kialakítás (nnn) technikák. Az írási modellnek van egy parancs feldolgozása verem üzleti logikát, bemenet-ellenőrzéshez és üzleti-ellenőrzéssel győződjön meg arról, hogy minden mindig konzisztens minden, a aggregátumok (minden az adatok változásait egy egységként kezelt kapcsolódó objektumok fürt) a az írási modell. Az olvasási modell üzleti logika és érvényesítési verem rendelkezik, és csak egy nézet modell adja vissza egy DTO való használatra. Az olvasási modell idővel konzisztenssé váljanak a írási modell.

- Forgatókönyvek, ahol adatok olvasási műveletek teljesítményének finom kell beállított külön teljesítményének adatok írási műveletekről, különösen akkor, ha az olvasási/írási arányt nagyon magas, és ha horizontális skálázás szükség. Például számos rendszer az olvasási műveletek sokszor több az írási műveletek számát. Ez teljesítése érdekében fontolja meg az olvasási modell kiterjesztése, de az írási modell futó csak egy vagy néhány példányát. A kis mennyiségű írási modell példánnyal is segíti a egyesítési ütközések előfordulása minimalizálása érdekében.

- Azok az esetek, amikor a fejlesztők egy csoportot, amely része a írási modell összetett tartománymodell összpontosíthat, így egy másik csapatban az olvasási és a felhasználói felületek összpontosíthat.

- Esetek, amikor a rendszer várhatóan verzióinformációk, így a modell több verziót tartalmaz, vagy ha az üzleti szabályok rendszeresen módosítsa.

- Integráció más rendszerekkel, különösen, valamint esemény forrás, amennyiben a egy alrendszer historikus hibája nem befolyásolja a többi rendelkezésre állását.

Ez a minta a következő esetekben nem ajánlott:

- A tartomány vagy az üzleti szabályok esetén egyszerű.

- Ha egy egyszerű CRUD-működő felhasználói felület és a kapcsolódó adatelérési műveletek elegendőek.

- A teljes rendszer végrehajtásához. Vannak olyan általános felügyeleti forgatókönyvet ahol CQRS hasznosak lehetnek, de jelentős és szükségtelen összetettségét, adhat hozzá, amikor nem szükséges az egyes összetevőinek.

## <a name="event-sourcing-and-cqrs"></a>Esemény forrás- és CQRS

CQRS mintát gyakran használják az esemény forrás mintát együtt. CQRS-alapú rendszerek külön olvasási használja, és írja adatmodellekben, minden egyes igazított kapcsolódó feladatok, és gyakran fizikailag elkülönülő tárolóban található. Használata esetén a [esemény forrás](event-sourcing.md) mintát, a tároló események írási modellje, és a hivatalos információforrás. A CQRS rendszerbe olvasási modelljét biztosít az adatok, materializált nézetek általában magas denormalizált nézetek. Ezek a nézetek a felületek vannak szabva, és megjeleníti az alkalmazás, amely segít a maximalizálása érdekében is megjelennek, illetve lekérdezési teljesítmény követelményeinek.

Az események streamjét használata az írási tároló ahelyett, hogy a tényleges adatok egy időben, ezzel elkerülheti a egyetlen összesítő frissítés ütközéseket, és a lehető legnagyobbra teljesítményét és méretezhetőségét. Az események aszinkron módon az olvasási tároló feltöltésére használatos olyan adatok materializált nézetek létrehozása céljából használható.

Mivel az esemény tároló információk hivatalos forrását, úgy is módosíthat a materializált nézeteket, és a korábbi eseményeket, hozzon létre egy új ábrázolása aktuális állapotát, ha a rendszer követi, vagy ha módosítania kell az olvasási modell visszajátszásos. A materializált nézetekhez érvényben az adatok tartósságát, csak olvasható gyorsítótárának.

Az esemény forrás mintát együtt CQRS használ, vegye figyelembe a következőket:

- Csakúgy, mint bármely rendszer, ha külön-e írási és olvasási tárolókat, ez a minta alapján rendszereket csak idővel konzisztenssé. Néhány generálása az esemény és az adattár frissítendő késleltetés lesz.

- A minta összetettsége ad hozzá, mert kód kezdeményezése és kezelhetnek eseményeket, és állítsa össze vagy a megfelelő nézet, és a lekérdezések, illetve egy olvasási modell által igényelt objektumokat frissíteni kell létrehozni. Az esemény forrás mintázat használata esetén a CQRS minta összetettsége is megnehezítik sikeres végrehajtása, és a rendszer tervezése eltérő megközelítést igényel. Azonban esemény forrás megkönnyítheti a modell a tartományhoz, és megkönnyíti a nézetek építse újra, vagy újakat létrehozni, mert az adatok változásait szándékával megőrződik.

- Generálása materializált nézetek az olvasási modell használatra, vagy az adatok visszajátszását, és konkrét személyek vagy entitások gyűjteményét események kezelésére leképezések megkövetelheti jelentős feldolgozási idő és erőforrás-használat. Ez azért különösen igaz, ha hosszú távú igényli összegzési vagy értékek elemzése, mert előfordulhat, hogy a kapcsolódó események kell vizsgálni. A megoldást pillanatképeket készíteni a végrehajtási rendszeres időközönként, például egy adott művelet történt teljes száma, vagy egy entitás jelenlegi állapotával.

## <a name="example"></a>Példa

A következő kód bemutatja, néhány példa a CQRS megvalósítása, amely más definíciók használ az olvasási és írási modellek kivonata. A modell felületek nem előírják a mögöttes adatokat tároló jellemzője, és fejlődnek, és lehet, hogy finomhangolható egymástól függetlenül konfigurációkezelővel elkülönül egymástól.

A következő kód bemutatja az olvasási modell-definícióban.

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

A rendszer lehetővé teszi a felhasználóknak arány termékek. Az alkalmazás kódjának elvégzi ezt használja a `RateProduct` parancs az alábbi kódban látható.

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

A rendszer használja a `ProductsCommandHandler` osztály kezelni az alkalmazás által küldött parancsokat. Ügyfelek általában parancsainak elküldését a tartomány egy üzenetkezelési rendszerén, például egy üzenetsort keresztül. A parancskezelő elfogadja ezeket a parancsokat, és a tartományi kapcsolat módszereket hív. Az egyes parancsok granularitási célja, hogy csökkenti annak esélyét ütköző kérelmek. A következő kód bemutatja áttekintése, amelyek a `ProductsCommandHandler` osztály.

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

A következő kódot tartalmazza a `IProductsDomain` írási modellből felületet.

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

Is figyelje meg, hogyan a `IProductsDomain` felület, amely megegyezik a tartomány módszerek tartalmazza. Általában CRUD környezetben ezek a módszerek kellene általános nevek például `Save` vagy `Update`, és egy DTO egyetlen argumentumaként. A CQRS megközelítés is kell megtervezni, hogy a szervezet üzleti és a szoftverleltár-kezelő rendszerek igényeinek.

## <a name="related-patterns-and-guidance"></a>Útmutató és a kapcsolódó minták

A következő mintákat és útmutatókat hasznosak, ha ezt a mintát megvalósító:

- Egy CQRS más architekturális stílusú, lásd: [architektúra stílusok](/azure/architecture/guide/architecture-styles/) és [CQRS architektúra stílus](/azure/architecture/guide/architecture-styles/cqrs).

- [Adatok konzisztencia ismertetése](https://msdn.microsoft.com/library/dn589800.aspx). A problémákat, amelyek között az olvasás a végleges konzisztencia miatt általában hibát, és adattároló írni a CQRS mintát, és hogyan oldhatók fel ezeket a problémákat ismerteti.

- [Útmutatás particionálás adatok](https://msdn.microsoft.com/library/dn589795.aspx). Ismerteti, hogyan CQRS mintájában használt olvasási és írási adattárolókhoz lehet osztani kívánt kezelhetők, és külön-külön érhető el, méretezhetőség javítása érdekében csökkentheti a versengés, és a teljesítmény optimalizálása.

- [Minta forrás esemény](event-sourcing.md). Részletesebben ismerteti, hogyan esemény forrás használható CQRS mintájú összetett tartományok feladatok egyszerűbbé teheti a teljesítmény, méretezhetőség és reakcióidőt fokozása mellett. Valamint hogy hogyan konzisztencia tranzakciós adatok teljes naplózási előzmények kezelése és engedélyezheti a kompenzációs műveletek előzményeinek megőrzése.

- [A materializált nézet mintát](materialized-view.md). A CQRS megvalósításának olvasási modelljét tartalmazhat írási modell adatok materializált nézeteket, vagy az olvasási modell használható materializált nézetek létrehozása céljából.

- A minták és gyakorlatok útmutató [CQRS út](http://aka.ms/cqrs). Különösen [vezet be, a parancs lekérdezés felelősségi elkülönítése mintát](https://msdn.microsoft.com/library/jj591573.aspx) felderíti a mintázat és mikor célszerű, és [zárszó: megszerzett tapasztalatok](https://msdn.microsoft.com/library/jj591568.aspx) segít megérteni a problémák némelyikéről amely kapja meg, ha ezt a mintát használja.

- A post [által Anna Fowler CQRS](http://martinfowler.com/bliki/CQRS.html), amely mintát és hivatkozások használatának alapjait ismerteti más hasznos erőforrásokhoz.

- [Greg Young bejegyzéseket](http://codebetter.com/gregyoung/), amely megismerkedhet a CQRS mintát sok aspektusait.
