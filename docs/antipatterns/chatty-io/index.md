---
title: "Forgalmas I/O kizárási minta"
description: "Ha sok I/O-kérés érkezik be, az negatívan befolyásolhatja a teljesítményt és a válaszkészséget."
author: dragon119
ms.openlocfilehash: 4f0e0e455ceb58317d3029d8ab4631d476802499
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/23/2018
---
# <a name="chatty-io-antipattern"></a><span data-ttu-id="00fe9-103">Forgalmas I/O kizárási minta</span><span class="sxs-lookup"><span data-stu-id="00fe9-103">Chatty I/O antipattern</span></span>

<span data-ttu-id="00fe9-104">A nagy számban érkező I/O-kérések halmozott hatása jelentősen kihathat a teljesítményre és a válaszkészségre.</span><span class="sxs-lookup"><span data-stu-id="00fe9-104">The cumulative effect of a large number of I/O requests can have a significant impact on performance and responsiveness.</span></span>

## <a name="problem-description"></a><span data-ttu-id="00fe9-105">A probléma leírása</span><span class="sxs-lookup"><span data-stu-id="00fe9-105">Problem description</span></span>

<span data-ttu-id="00fe9-106">A hálózati hívások és az egyéb I/O-műveletek természetüknél fogva lassabbak, mint a számítási feladatok.</span><span class="sxs-lookup"><span data-stu-id="00fe9-106">Network calls and other I/O operations are inherently slow compared to compute tasks.</span></span> <span data-ttu-id="00fe9-107">Az I/O-kérések rendszerint jelentős terhelést okoznak, és a sok I/O-művelet halmozott hatása lelassíthatja a rendszert.</span><span class="sxs-lookup"><span data-stu-id="00fe9-107">Each I/O request typically has significant overhead, and the cumulative effect of numerous I/O operations can slow down the system.</span></span> <span data-ttu-id="00fe9-108">Alább találhatók a forgalmas I/O gyakori okai.</span><span class="sxs-lookup"><span data-stu-id="00fe9-108">Here are some common causes of chatty I/O.</span></span>

### <a name="reading-and-writing-individual-records-to-a-database-as-distinct-requests"></a><span data-ttu-id="00fe9-109">Az egyes rekordok adatbázisba való írása és olvasása külön kérésekként</span><span class="sxs-lookup"><span data-stu-id="00fe9-109">Reading and writing individual records to a database as distinct requests</span></span>

<span data-ttu-id="00fe9-110">Az alábbi példa egy termékadatbázisból olvas.</span><span class="sxs-lookup"><span data-stu-id="00fe9-110">The following example reads from a database of products.</span></span> <span data-ttu-id="00fe9-111">Három tábla van: `Product`, `ProductSubcategory` és `ProductPriceListHistory`.</span><span class="sxs-lookup"><span data-stu-id="00fe9-111">There are three tables, `Product`, `ProductSubcategory`, and `ProductPriceListHistory`.</span></span> <span data-ttu-id="00fe9-112">A kód egy lekérdezésekből álló sorozat végrehajtásával egy alkategória összes termékét lekéri az árakra vonatkozó információkkal együtt.</span><span class="sxs-lookup"><span data-stu-id="00fe9-112">The code retrieves all of the products in a subcategory, along with the pricing information, by executing a series of queries:</span></span>  

1. <span data-ttu-id="00fe9-113">Az alkategória lekérdezése a `ProductSubcategory` táblából.</span><span class="sxs-lookup"><span data-stu-id="00fe9-113">Query the subcategory from the `ProductSubcategory` table.</span></span>
2. <span data-ttu-id="00fe9-114">Az alkategória összes termékének megkeresése a `Product` tábla lekérdezésével.</span><span class="sxs-lookup"><span data-stu-id="00fe9-114">Find all products in that subcategory by querying the `Product` table.</span></span>
3. <span data-ttu-id="00fe9-115">Az árképzési adatok lekérdezése minden termékről a `ProductPriceListHistory` táblából.</span><span class="sxs-lookup"><span data-stu-id="00fe9-115">For each product, query the pricing data from the `ProductPriceListHistory` table.</span></span>

<span data-ttu-id="00fe9-116">Az alkalmazás az [Entity Framework][ef]öt használja az adatbázis lekérdezéséhez.</span><span class="sxs-lookup"><span data-stu-id="00fe9-116">The application uses [Entity Framework][ef] to query the database.</span></span> <span data-ttu-id="00fe9-117">A teljes kódmintát [itt][code-sample] találja.</span><span class="sxs-lookup"><span data-stu-id="00fe9-117">You can find the complete sample [here][code-sample].</span></span> 

```csharp
public async Task<IHttpActionResult> GetProductsInSubCategoryAsync(int subcategoryId)
{
    using (var context = GetContext())
    {
        // Get product subcategory.
        var productSubcategory = await context.ProductSubcategories
                .Where(psc => psc.ProductSubcategoryId == subcategoryId)
                .FirstOrDefaultAsync();

        // Find products in that category.
        productSubcategory.Product = await context.Products
            .Where(p => subcategoryId == p.ProductSubcategoryId)
            .ToListAsync();

        // Find price history for each product.
        foreach (var prod in productSubcategory.Product)
        {
            int productId = prod.ProductId;
            var productListPriceHistory = await context.ProductListPriceHistory
                .Where(pl => pl.ProductId == productId)
                .ToListAsync();
            prod.ProductListPriceHistory = productListPriceHistory;
        }
        return Ok(productSubcategory);
    }
}
```

<span data-ttu-id="00fe9-118">Ez a példa világosan bemutatja a problémát, viszont egy O/RM gyakran elfedheti, ha a gyermekrekordokat implicit módon egyenként olvassa be.</span><span class="sxs-lookup"><span data-stu-id="00fe9-118">This example shows the problem explicitly, but sometimes an O/RM can mask the problem, if it implicitly fetches child records one at a time.</span></span> <span data-ttu-id="00fe9-119">Ez az úgynevezett „N+1 probléma”.</span><span class="sxs-lookup"><span data-stu-id="00fe9-119">This is known as the "N+1 problem".</span></span> 

### <a name="implementing-a-single-logical-operation-as-a-series-of-http-requests"></a><span data-ttu-id="00fe9-120">Egyetlen logikai művelet megvalósítása HTTP-kéréssorozatként</span><span class="sxs-lookup"><span data-stu-id="00fe9-120">Implementing a single logical operation as a series of HTTP requests</span></span>

<span data-ttu-id="00fe9-121">Ez gyakran előfordulhat, amikor a fejlesztők objektumorientált paradigmát követnek, a távoli objektumokat pedig úgy kezelik, mintha helyi objektumok lennének a memóriában.</span><span class="sxs-lookup"><span data-stu-id="00fe9-121">This often happens when developers try to follow an object-oriented paradigm, and treat remote objects as if they were local objects in memory.</span></span> <span data-ttu-id="00fe9-122">Ez túlzott hálózati forgalmat eredményezhet.</span><span class="sxs-lookup"><span data-stu-id="00fe9-122">This can result in too many network round trips.</span></span> <span data-ttu-id="00fe9-123">A következő webes API például az egyéni HTTP GET metódusokkal felfedi a `User` objektumok egyéni tulajdonságait.</span><span class="sxs-lookup"><span data-stu-id="00fe9-123">For example, the following web API exposes the individual properties of `User` objects through individual HTTP GET methods.</span></span> 

```csharp
public class UserController : ApiController
{
    [HttpGet]
    [Route("users/{id:int}/username")]
    public HttpResponseMessage GetUserName(int id)
    {
        ...
    }

    [HttpGet]
    [Route("users/{id:int}/gender")]
    public HttpResponseMessage GetGender(int id)
    {
        ...
    }

    [HttpGet]
    [Route("users/{id:int}/dateofbirth")]
    public HttpResponseMessage GetDateOfBirth(int id)
    {
        ...
    }
}
```

<span data-ttu-id="00fe9-124">Ez a módszer sem hibás, de a legtöbb ügyfélnek több tulajdonságra van szüksége minden `User` esetében, ez pedig a következőhöz hasonló ügyfélkódot eredményezi.</span><span class="sxs-lookup"><span data-stu-id="00fe9-124">While there's nothing technically wrong with this approach, most clients will probably need to get several properties for each `User`, resulting in client code like the following.</span></span> 

```csharp
HttpResponseMessage response = await client.GetAsync("users/1/username");
response.EnsureSuccessStatusCode();
var userName = await response.Content.ReadAsStringAsync();

response = await client.GetAsync("users/1/gender");
response.EnsureSuccessStatusCode();
var gender = await response.Content.ReadAsStringAsync();

response = await client.GetAsync("users/1/dateofbirth");
response.EnsureSuccessStatusCode();
var dob = await response.Content.ReadAsStringAsync();
```

### <a name="reading-and-writing-to-a-file-on-disk"></a><span data-ttu-id="00fe9-125">Olvasás és írás lemezen lévő fájlba</span><span class="sxs-lookup"><span data-stu-id="00fe9-125">Reading and writing to a file on disk</span></span>

<span data-ttu-id="00fe9-126">A fájl I/O egy fájl megnyitását és a megfelelő pontra való helyezését jelenti az adatok írása és olvasása előtt.</span><span class="sxs-lookup"><span data-stu-id="00fe9-126">File I/O involves opening a file and moving to the appropriate point before reading or writing data.</span></span> <span data-ttu-id="00fe9-127">A művelet befejezésekor a rendszer bezárhatja a fájlt az operációs rendszer erőforrásainak megtakarítása érdekében.</span><span class="sxs-lookup"><span data-stu-id="00fe9-127">When the operation is complete, the file might be closed to save operating system resources.</span></span> <span data-ttu-id="00fe9-128">Egy olyan alkalmazás, amely folyamatosan kis mennyiségű információt olvas és ír egy fájlba, jelentős mennyiségű I/O-terhelést okoz.</span><span class="sxs-lookup"><span data-stu-id="00fe9-128">An application that continually reads and writes small amounts of information to a file will generate significant I/O overhead.</span></span> <span data-ttu-id="00fe9-129">A kis írási kérések töredezettséghez is vezethetnek, ami tovább lassíthatja a későbbi I/O-műveleteket.</span><span class="sxs-lookup"><span data-stu-id="00fe9-129">Small write requests can also lead to file fragmentation, slowing subsequent I/O operations still further.</span></span> 

<span data-ttu-id="00fe9-130">A következő példa `FileStream`et használ egy `Customer` objektum fájlba való írására.</span><span class="sxs-lookup"><span data-stu-id="00fe9-130">The following example uses a `FileStream` to write a `Customer` object to a file.</span></span> <span data-ttu-id="00fe9-131">A `FileStream` létrehozása megnyitja, az eltávolítása pedig bezárja a fájlt.</span><span class="sxs-lookup"><span data-stu-id="00fe9-131">Creating the `FileStream` opens the file, and disposing it closes the file.</span></span> <span data-ttu-id="00fe9-132">(A `using` utasítás automatikusan elveti a `FileStream` objektumot.) Ha az alkalmazás ismételten meghívja ezt a metódust új ügyfelek hozzáadásakor, az I/O-terhelés gyorsan megnövekedhet.</span><span class="sxs-lookup"><span data-stu-id="00fe9-132">(The `using` statement automatically disposes the `FileStream` object.) If the application calls this method repeatedly as new customers are added, the I/O overhead can accumulate quickly.</span></span>

```csharp
private async Task SaveCustomerToFileAsync(Customer cust)
{
    using (Stream fileStream = new FileStream(CustomersFileName, FileMode.Append))
    {
        BinaryFormatter formatter = new BinaryFormatter();
        byte [] data = null;
        using (MemoryStream memStream = new MemoryStream())
        {
            formatter.Serialize(memStream, cust);
            data = memStream.ToArray();
        }
        await fileStream.WriteAsync(data, 0, data.Length);
    }
}
```

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="00fe9-133">A probléma megoldása</span><span class="sxs-lookup"><span data-stu-id="00fe9-133">How to fix the problem</span></span>

<span data-ttu-id="00fe9-134">Csökkentse az I/O-kérések számát úgy, hogy az adatokat kevesebb, de nagyobb méretű kérésekbe csomagolja.</span><span class="sxs-lookup"><span data-stu-id="00fe9-134">Reduce the number of I/O requests by packaging the data into larger, fewer requests.</span></span>

<span data-ttu-id="00fe9-135">Egy lekérdezéssel olvassa be az adatokat az adatbázisból több kisebb lekérdezés helyett.</span><span class="sxs-lookup"><span data-stu-id="00fe9-135">Fetch data from a database as a single query, instead of several smaller queries.</span></span> <span data-ttu-id="00fe9-136">Itt található a termékinformációkat lekérő kód átdolgozott verziója.</span><span class="sxs-lookup"><span data-stu-id="00fe9-136">Here's a revised version of the code that retrieves product information.</span></span>

```csharp
public async Task<IHttpActionResult> GetProductCategoryDetailsAsync(int subCategoryId)
{
    using (var context = GetContext())
    {
        var subCategory = await context.ProductSubcategories
                .Where(psc => psc.ProductSubcategoryId == subCategoryId)
                .Include("Product.ProductListPriceHistory")
                .FirstOrDefaultAsync();

        if (subCategory == null)
            return NotFound();

        return Ok(subCategory);
    }
}
```

<span data-ttu-id="00fe9-137">Kövesse a REST webes API-kra vonatkozó tervezési alapelveit.</span><span class="sxs-lookup"><span data-stu-id="00fe9-137">Follow REST design principles for web APIs.</span></span> <span data-ttu-id="00fe9-138">Itt található a korábbi példában használt webes API átdolgozott verziója.</span><span class="sxs-lookup"><span data-stu-id="00fe9-138">Here's a revised version of the web API from the earlier example.</span></span> <span data-ttu-id="00fe9-139">Ahelyett, hogy minden tulajdonsághoz külön GET metódust használna, elérhető egyetlen GET metódus, amely visszaadja a `User` értékét.</span><span class="sxs-lookup"><span data-stu-id="00fe9-139">Instead of separate GET methods for each property, there is a single GET method that returns the `User`.</span></span> <span data-ttu-id="00fe9-140">Emiatt a kérésenkénti választörzsek mérete nagyobb lesz, de az ügyfelek valószínűleg kevesebb API-hívást hajtanak végre.</span><span class="sxs-lookup"><span data-stu-id="00fe9-140">This results in a larger response body per request, but each client is likely to make fewer API calls.</span></span>

```csharp
public class UserController : ApiController
{
    [HttpGet]
    [Route("users/{id:int}")]
    public HttpResponseMessage GetUser(int id)
    {
        ...
    }
}

// Client code
HttpResponseMessage response = await client.GetAsync("users/1");
response.EnsureSuccessStatusCode();
var user = await response.Content.ReadAsStringAsync();
```

<span data-ttu-id="00fe9-141">A fájl I/O csökkentése érdekében fontolja meg az adatok memóriában történő pufferelését, majd a pufferelt adatok fájlba való írását egyetlen műveletben.</span><span class="sxs-lookup"><span data-stu-id="00fe9-141">For file I/O, consider buffering data in memory and then writing the buffered data to a file as a single operation.</span></span> <span data-ttu-id="00fe9-142">Ez csökkenti a fájl ismételt megnyitása és bezárása által okozott terhelést, és a lemezen lévő fájl töredezettségének csökkentésében is segít.</span><span class="sxs-lookup"><span data-stu-id="00fe9-142">This approach reduces the overhead from repeatedly opening and closing the file, and helps to reduce fragmentation of the file on disk.</span></span>

```csharp
// Save a list of customer objects to a file
private async Task SaveCustomerListToFileAsync(List<Customer> customers)
{
    using (Stream fileStream = new FileStream(CustomersFileName, FileMode.Append))
    {
        BinaryFormatter formatter = new BinaryFormatter();
        foreach (var cust in customers)
        {
            byte[] data = null;
            using (MemoryStream memStream = new MemoryStream())
            {
                formatter.Serialize(memStream, cust);
                data = memStream.ToArray();
            }
            await fileStream.WriteAsync(data, 0, data.Length);
        }
    }
}

// In-memory buffer for customers.
List<Customer> customers = new List<Customers>();

// Create a new customer and add it to the buffer
var cust = new Customer(...);
customers.Add(cust);

// Add more customers to the list as they are created
...

// Save the contents of the list, writing all customers in a single operation
await SaveCustomerListToFileAsync(customers);
```

## <a name="considerations"></a><span data-ttu-id="00fe9-143">Megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="00fe9-143">Considerations</span></span>

- <span data-ttu-id="00fe9-144">Az első két példa *kevesebb* I/O-hívást intéz, de mindegyik *több* információt kér le.</span><span class="sxs-lookup"><span data-stu-id="00fe9-144">The first two examples make *fewer* I/O calls, but each one retrieves *more* information.</span></span> <span data-ttu-id="00fe9-145">Meg kell találnia az egyensúlyt a két tényező között.</span><span class="sxs-lookup"><span data-stu-id="00fe9-145">You must consider the tradeoff between these two factors.</span></span> <span data-ttu-id="00fe9-146">A legjobb megoldás a tényleges felhasználási mintáktól függ.</span><span class="sxs-lookup"><span data-stu-id="00fe9-146">The right answer will depend on the actual usage patterns.</span></span> <span data-ttu-id="00fe9-147">A webes API példájában előfordulhat, hogy az ügyfeleknek csak a felhasználónévre van szükségük.</span><span class="sxs-lookup"><span data-stu-id="00fe9-147">For example, in the web API example, it might turn out that clients often need just the user name.</span></span> <span data-ttu-id="00fe9-148">Ebben az esetben célszerű lehet külön API-hívásként elérhetővé tenni a nevet.</span><span class="sxs-lookup"><span data-stu-id="00fe9-148">In that case, it might make sense to expose it as a separate API call.</span></span> <span data-ttu-id="00fe9-149">További információkért lásd a [Felesleges beolvasásokat][extraneous-fetching] ismertető kizárási mintát.</span><span class="sxs-lookup"><span data-stu-id="00fe9-149">For more information, see the [Extraneous Fetching][extraneous-fetching] antipattern.</span></span>

- <span data-ttu-id="00fe9-150">Az adatok olvasásakor ne küldjön túl nagy méretű I/O-kéréseket.</span><span class="sxs-lookup"><span data-stu-id="00fe9-150">When reading data, do not make your I/O requests too large.</span></span> <span data-ttu-id="00fe9-151">Az alkalmazásoknak csak azt az információt célszerű lehívniuk, amelyet valószínű, hogy fel is fog használni.</span><span class="sxs-lookup"><span data-stu-id="00fe9-151">An application should only retrieve the information that it is likely to use.</span></span> 

- <span data-ttu-id="00fe9-152">Néha hasznos egy objektumra vonatkozó információ két tömbre történő particionálása: *gyakran lehívott adatokra*, amelyekre a legtöbb kérés irányul, illetve *kevésbé gyakran lehívott adatokra*, amelyeket ritkán használnak.</span><span class="sxs-lookup"><span data-stu-id="00fe9-152">Sometimes it helps to partition the information for an object into two chunks, *frequently accessed data* that accounts for most requests, and *less frequently accessed data* that is used rarely.</span></span> <span data-ttu-id="00fe9-153">Sokszor a leggyakrabban lehívott adatok az objektum teljes adatainak viszonylag kis részét jelentik, ezért ha a rendszer csak ezt a részt hívja le, azzal jelentős mértékű I/O-terhelést takaríthat meg.</span><span class="sxs-lookup"><span data-stu-id="00fe9-153">Often the most frequently accessed data is a relatively small portion of the total data for an object, so returning just that portion can save significant I/O overhead.</span></span>

- <span data-ttu-id="00fe9-154">Az adatok írásakor kerülje az erőforrások szükségesnél tovább történő zárolását, így csökkentheti a versengés kialakulásának esélyét a hosszabb műveletek során.</span><span class="sxs-lookup"><span data-stu-id="00fe9-154">When writing data, avoid locking resources for longer than necessary, to reduce the chances of contention during a lengthy operation.</span></span> <span data-ttu-id="00fe9-155">Ha az írási művelet több adattárolót, fájlt vagy szolgáltatást is érint, akkor használjon olyan módszert, amely végül konzisztens eredményekhez vezet.</span><span class="sxs-lookup"><span data-stu-id="00fe9-155">If a write operation spans multiple data stores, files, or services, then adopt an eventually consistent approach.</span></span> <span data-ttu-id="00fe9-156">Lásd: [Adatkonzisztencia-útmutató][data-consistency-guidance].</span><span class="sxs-lookup"><span data-stu-id="00fe9-156">See [Data Consistency guidance][data-consistency-guidance].</span></span>

- <span data-ttu-id="00fe9-157">Ha puffereli az adatokat a memóriában az írása előtt, akkor az adatok a folyamatleállások esetén sebezhetővé válnak.</span><span class="sxs-lookup"><span data-stu-id="00fe9-157">If you buffer data in memory before writing it, the data is vulnerable if the process crashes.</span></span> <span data-ttu-id="00fe9-158">Ha az adatátvitel viszonylag ritka, vagy adatlöketek jellemzik, akkor biztonságosabb az adatok tartósabb, külső üzenetsorban (például az [Event Hubsban](http://azure.microsoft.com/services/event-hubs/)) történő pufferelése.</span><span class="sxs-lookup"><span data-stu-id="00fe9-158">If the data rate typically has bursts or is relatively sparse, it may be safer to buffer the data in an external durable queue such as [Event Hubs](http://azure.microsoft.com/services/event-hubs/).</span></span>

- <span data-ttu-id="00fe9-159">Fontolja meg a szolgáltatásból vagy adatbázisból lekért adatok gyorsítótárazását.</span><span class="sxs-lookup"><span data-stu-id="00fe9-159">Consider caching data that you retrieve from a service or a database.</span></span> <span data-ttu-id="00fe9-160">Ezzel segíthet csökkenteni az I/O-műveletek mennyiségét, mert elkerüli az ugyanazon adatokra vonatkozó ismételt kéréseket.</span><span class="sxs-lookup"><span data-stu-id="00fe9-160">This can help to reduce the volume of I/O by avoiding repeated requests for the same data.</span></span> <span data-ttu-id="00fe9-161">További információkért lásd a [Gyorsítótárazás – Ajánlott eljárások][caching-guidance] című témakört.</span><span class="sxs-lookup"><span data-stu-id="00fe9-161">For more information, see [Caching best practices][caching-guidance].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="00fe9-162">A probléma észlelése</span><span class="sxs-lookup"><span data-stu-id="00fe9-162">How to detect the problem</span></span>

<span data-ttu-id="00fe9-163">A forgalmas I/O tünetei a magas késleltetés és az alacsony átviteli sebesség.</span><span class="sxs-lookup"><span data-stu-id="00fe9-163">Symptoms of chatty I/O include high latency and low throughput.</span></span> <span data-ttu-id="00fe9-164">A végfelhasználók várhatóan magas válaszidőket vagy a szolgáltatások időtúllépéséből eredő hibákat fognak jelezni, mivel nagyobb a versengés az I/O-erőforrásokért.</span><span class="sxs-lookup"><span data-stu-id="00fe9-164">End users are likely to report extended response times or failures caused by services timing out, due to increased contention for I/O resources.</span></span>

<span data-ttu-id="00fe9-165">A következő lépéseket végezheti el az esetlegesen felmerülő problémák okának meghatározásához:</span><span class="sxs-lookup"><span data-stu-id="00fe9-165">You can perform the following steps to help identify the causes of any problems:</span></span>

1. <span data-ttu-id="00fe9-166">Végezzen folyamatfigyelést a termelési rendszeren a hosszú válaszidejű műveletek azonosításához.</span><span class="sxs-lookup"><span data-stu-id="00fe9-166">Perform process monitoring of the production system to identify operations with poor response times.</span></span>
2. <span data-ttu-id="00fe9-167">Az előző lépésben azonosított műveletek mindegyikénél végezzen terheléstesztelést.</span><span class="sxs-lookup"><span data-stu-id="00fe9-167">Perform load testing of each operation identified in the previous step.</span></span>
3. <span data-ttu-id="00fe9-168">A terheléstesztek alatt gyűjtsön telemetriai adatokat a műveletek által eszközölt adathozzáférési kérésekről.</span><span class="sxs-lookup"><span data-stu-id="00fe9-168">During the load tests, gather telemetry data about the data access requests made by each operation.</span></span>
4. <span data-ttu-id="00fe9-169">Gyűjtsön részletes statisztikát minden adattárba küldött kérésről.</span><span class="sxs-lookup"><span data-stu-id="00fe9-169">Gather detailed statistics for each request sent to a data store.</span></span>
5. <span data-ttu-id="00fe9-170">Mérje fel az alkalmazást a tesztkörnyezetben, hogy meghatározza, hol fordulhatnak elő esetleg szűk I/O keresztmetszetek.</span><span class="sxs-lookup"><span data-stu-id="00fe9-170">Profile the application in the test environment to establish where possible I/O bottlenecks might be occurring.</span></span> 

<span data-ttu-id="00fe9-171">Az alábbi tüneteket figyelje:</span><span class="sxs-lookup"><span data-stu-id="00fe9-171">Look for any of these symptoms:</span></span>

- <span data-ttu-id="00fe9-172">Nagy számú kis I/O kérés jön létre ugyanahhoz a fájlhoz.</span><span class="sxs-lookup"><span data-stu-id="00fe9-172">A large number of small I/O requests made to the same file.</span></span>
- <span data-ttu-id="00fe9-173">Egy alkalmazáspéldány nagy számú kis hálózati kérést hoz létre ugyanahhoz a szolgáltatáshoz.</span><span class="sxs-lookup"><span data-stu-id="00fe9-173">A large number of small network requests made by an application instance to the same service.</span></span>
- <span data-ttu-id="00fe9-174">Egy alkalmazáspéldány nagy számú kis kérést hoz létre ugyanahhoz az adattárhoz.</span><span class="sxs-lookup"><span data-stu-id="00fe9-174">A large number of small requests made by an application instance to the same data store.</span></span>
- <span data-ttu-id="00fe9-175">Az alkalmazások és szolgáltatások I/O korlátozottá válnak.</span><span class="sxs-lookup"><span data-stu-id="00fe9-175">Applications and services becoming I/O bound.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="00fe9-176">Diagnosztikai példa</span><span class="sxs-lookup"><span data-stu-id="00fe9-176">Example diagnosis</span></span>

<span data-ttu-id="00fe9-177">Az alábbi szakaszok a korábban bemutatott adatbázis-lekérdezést végző példára alkalmazzák ezeket a lépéseket.</span><span class="sxs-lookup"><span data-stu-id="00fe9-177">The following sections apply these steps to the example shown earlier that queries a database.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="00fe9-178">Az alkalmazás terheléstesztje</span><span class="sxs-lookup"><span data-stu-id="00fe9-178">Load test the application</span></span>

<span data-ttu-id="00fe9-179">Ez a diagram bemutatja a terheléstesztelés eredményeit.</span><span class="sxs-lookup"><span data-stu-id="00fe9-179">This graph shows the results of load testing.</span></span> <span data-ttu-id="00fe9-180">A medián válaszidőt a rendszer kérésenként 10 másodpercekben méri.</span><span class="sxs-lookup"><span data-stu-id="00fe9-180">Median response time is measured in 10s of seconds per request.</span></span> <span data-ttu-id="00fe9-181">A diagram nagy késést mutat.</span><span class="sxs-lookup"><span data-stu-id="00fe9-181">The graph shows very high latency.</span></span> <span data-ttu-id="00fe9-182">1000 felhasználós tesztelésnél előfordulhat, hogy egy felhasználónak akár egy percet is várnia kell, amíg megkapja a lekérdezés eredményét.</span><span class="sxs-lookup"><span data-stu-id="00fe9-182">With a load of 1000 users, a user might have to wait for nearly a minute to see the results of a query.</span></span> 

![A legfontosabb mutatók terheléstesztelési eredményei a forgalmas I/O mintaalkalmazáshoz][key-indicators-chatty-io]

> [!NOTE]
> <span data-ttu-id="00fe9-184">Az alkalmazás egy Azure App Service webalkalmazásban lett üzembe helyezve az Azure SQL Database segítségével.</span><span class="sxs-lookup"><span data-stu-id="00fe9-184">The application was deployed as an Azure App Service web app, using Azure SQL Database.</span></span> <span data-ttu-id="00fe9-185">A terhelésteszt egy szimulált lépéses munkaterhelést használt legfeljebb 1000 párhuzamos felhasználóval.</span><span class="sxs-lookup"><span data-stu-id="00fe9-185">The load test used a simulated step workload of up to 1000 concurrent users.</span></span> <span data-ttu-id="00fe9-186">Az adatbázis legfeljebb 1000 párhuzamos kapcsolatot támogató kapcsolatkészlettel van konfigurálva, hogy csökkenjen az esélye annak, hogy a kapcsolatokért aló versengés befolyásolja az eredményt.</span><span class="sxs-lookup"><span data-stu-id="00fe9-186">The database was configured with a connection pool supporting up to 1000 concurrent connections, to reduce the chance that contention for connections would affect the results.</span></span> 

### <a name="monitor-the-application"></a><span data-ttu-id="00fe9-187">Az alkalmazás figyelése</span><span class="sxs-lookup"><span data-stu-id="00fe9-187">Monitor the application</span></span>

<span data-ttu-id="00fe9-188">Az alkalmazásteljesítmény-figyelő (APM) csomaggal rögzítheti és elemezheti a forgalmas I/O azonosítására képes kulcs mérőszámokat.</span><span class="sxs-lookup"><span data-stu-id="00fe9-188">You can use an application performance monitoring (APM) package to capture and analyze the key metrics that might identify chatty I/O.</span></span> <span data-ttu-id="00fe9-189">Az, hogy melyik mérőszám lesz fontos, az I/O munkaterheléstől függ.</span><span class="sxs-lookup"><span data-stu-id="00fe9-189">Which metrics are important will depend on the I/O workload.</span></span> <span data-ttu-id="00fe9-190">Ebben a példában az adatbázis-lekérdezések voltak az érdekes I/O kérések.</span><span class="sxs-lookup"><span data-stu-id="00fe9-190">For this example, the interesting I/O requests were the database queries.</span></span> 

<span data-ttu-id="00fe9-191">Az alábbi ábra a [New Relic APM][new-relic] használatával létrehozott eredményeket mutatja.</span><span class="sxs-lookup"><span data-stu-id="00fe9-191">The following image shows results generated using [New Relic APM][new-relic].</span></span> <span data-ttu-id="00fe9-192">Az adatbázis átlagos válaszidejének csúcsa körülbelül 5,6 másodperc volt kérésenként a legnagyobb munkaterhelés közben.</span><span class="sxs-lookup"><span data-stu-id="00fe9-192">The average database response time peaked at approximately 5.6 seconds per request during the maximum workload.</span></span> <span data-ttu-id="00fe9-193">A rendszer a teszt során átlagosan 410 kérés támogatására volt képes percenként.</span><span class="sxs-lookup"><span data-stu-id="00fe9-193">The system was able to support an average of 410 requests per minute throughout the test.</span></span>

![Az AdventureWorks2012 adatbázist elérő adatforgalom áttekintése][databasetraffic]

### <a name="gather-detailed-data-access-information"></a><span data-ttu-id="00fe9-195">Részletes adathozzáférési információk gyűjtése</span><span class="sxs-lookup"><span data-stu-id="00fe9-195">Gather detailed data access information</span></span>

<span data-ttu-id="00fe9-196">Ha figyelési adatok mélyebb vizsgálata megmutatja, hogy az alkalmazás három különböző SQL SELECT utasítást hajt végre.</span><span class="sxs-lookup"><span data-stu-id="00fe9-196">Digging deeper into the monitoring data shows the application executes three different SQL SELECT statements.</span></span> <span data-ttu-id="00fe9-197">Ezek megfelelnek az Entity Framework által a `ProductListPriceHistory`, `Product` és `ProductSubcategory` táblákból való adatbeolvasásokra létrehozott kéréseknek.</span><span class="sxs-lookup"><span data-stu-id="00fe9-197">These correspond to the requests generated by Entity Framework to fetch data from the `ProductListPriceHistory`, `Product`, and `ProductSubcategory` tables.</span></span>
<span data-ttu-id="00fe9-198">Ráadásul a `ProductListPriceHistory` táblából adatokat lekérő lekérdezés a messze leggyakoribban végrehajtott SELECT utasítás, nagyságrendekkel gyakoribb a többinél.</span><span class="sxs-lookup"><span data-stu-id="00fe9-198">Furthermore, the query that retrieves data from the `ProductListPriceHistory` table is by far the most frequently executed SELECT statement, by an order of magnitude.</span></span>

![A tesztelt mintaalkalmazás által végrehajtott lekérdezések][queries]

<span data-ttu-id="00fe9-200">Kiderül, hogy a korábban bemutatott `GetProductsInSubCategoryAsync` metódus 45 SELECT lekérdezést végez el.</span><span class="sxs-lookup"><span data-stu-id="00fe9-200">It turns out that the `GetProductsInSubCategoryAsync` method, shown earlier, performs 45 SELECT queries.</span></span> <span data-ttu-id="00fe9-201">Az alkalmazás minden lekérdezéshez megnyit egy új SQL-kapcsolatot.</span><span class="sxs-lookup"><span data-stu-id="00fe9-201">Each query causes the application to open a new SQL connection.</span></span>

![Lekérdezési statisztika a tesztelt mintaalkalmazáshoz][queries2]

> [!NOTE]
> <span data-ttu-id="00fe9-203">Az ábra nyomkövetési adatokat tartalmaz a terhelésteszt `GetProductsInSubCategoryAsync` műveletének leglassabb példányához.</span><span class="sxs-lookup"><span data-stu-id="00fe9-203">This image shows trace information for the slowest instance of the `GetProductsInSubCategoryAsync` operation in the load test.</span></span> <span data-ttu-id="00fe9-204">Éles környezetben hasznos megvizsgálni a leglassabb példányok nyomkövetését, így látható, hogy van-e problémára utaló mintázat.</span><span class="sxs-lookup"><span data-stu-id="00fe9-204">In a production environment, it's useful to examine traces of the slowest instances, to see if there is a pattern that suggests a problem.</span></span> <span data-ttu-id="00fe9-205">Ha csak az átlagértékeket veszi figyelembe, előfordulhat, hogy elkerüli a figyelmét egy olyan probléma, amely terhelés alatt sokkal súlyosabbá válik.</span><span class="sxs-lookup"><span data-stu-id="00fe9-205">If you just look at the average values, you might overlook problems that will get dramatically worse under load.</span></span>

<span data-ttu-id="00fe9-206">A következő ábrán a ténylegesen kiadott SQL-utasítások láthatók.</span><span class="sxs-lookup"><span data-stu-id="00fe9-206">The next image shows the actual SQL statements that were issued.</span></span> <span data-ttu-id="00fe9-207">Az árinformációkat beolvasó lekérdezés a termék alkategória minden egyes termékéhez fut.</span><span class="sxs-lookup"><span data-stu-id="00fe9-207">The query that fetches price information is run for each individual product in the product subcategory.</span></span> <span data-ttu-id="00fe9-208">Egy csatlakozással jelentősen csökkenthető lenne az adatbázishívások száma.</span><span class="sxs-lookup"><span data-stu-id="00fe9-208">Using a join would considerably reduce the number of database calls.</span></span>

![Lekérdezési adatok a tesztelt mintaalkalmazáshoz][queries3]

<span data-ttu-id="00fe9-210">Ha O/RM-et (például Entity Framework) használ, az SQL-lekérdezések nyomon követésével bepillantást nyerhet abba, hogy hogyan fordítja az O/RM a programozott hívásokat SQL-utasításokká, és észlelhet területeket, ahol optimalizálható az adathozzáférés.</span><span class="sxs-lookup"><span data-stu-id="00fe9-210">If you are using an O/RM, such as Entity Framework, tracing the SQL queries can provide insight into how the O/RM translates programmatic calls into SQL statements, and indicate areas where data access might be optimized.</span></span> 

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="00fe9-211">A megoldás megvalósítása és az eredmény ellenőrzése</span><span class="sxs-lookup"><span data-stu-id="00fe9-211">Implement the solution and verify the result</span></span>

<span data-ttu-id="00fe9-212">A hívás Entity Frameworkbe való újraírása az alább eredményeket hozta.</span><span class="sxs-lookup"><span data-stu-id="00fe9-212">Rewriting the call to Entity Framework produced the following results.</span></span>

![A legfontosabb mutatók terheléstesztelési eredményei a sűrű API-hoz a forgalmas I/O mintaalkalmazásban][key-indicators-chunky-io]

<span data-ttu-id="00fe9-214">Ez a terhelésteszt ugyanazon az üzemelő példányon lett elvégezve, ugyanazzal a terhelési profillal.</span><span class="sxs-lookup"><span data-stu-id="00fe9-214">This load test was performed on the same deployment, using the same load profile.</span></span> <span data-ttu-id="00fe9-215">Ez alkalommal a diagram sokkal alacsonyabb késést mutat.</span><span class="sxs-lookup"><span data-stu-id="00fe9-215">This time the graph shows much lower latency.</span></span> <span data-ttu-id="00fe9-216">A kérések átlagos ideje 1000 felhasználónál 5 és 6 másodperc között van, a legmagasabb érték majdnem egy perc volt.</span><span class="sxs-lookup"><span data-stu-id="00fe9-216">The average request time at 1000 users is between 5 and 6 seconds, down from nearly a minute.</span></span>

<span data-ttu-id="00fe9-217">Ez alkalommal a rendszer átlagosan 3970 kérést támogatott percenként (ez az érték a korábbi tesztben 410 volt).</span><span class="sxs-lookup"><span data-stu-id="00fe9-217">This time the system supported an average of 3,970 requests per minute, compared to 410 for the earlier test.</span></span>

![Tranzakció-áttekintés a sűrű API-hoz][databasetraffic2]

<span data-ttu-id="00fe9-219">Az SQL-utasítás nyomon követéséből látható, hogy a rendszer az összes adatot egyetlen SELECT utasítással olvasta be.</span><span class="sxs-lookup"><span data-stu-id="00fe9-219">Tracing the SQL statement shows that all the data is fetched in a single SELECT statement.</span></span> <span data-ttu-id="00fe9-220">Noha ez a lekérdezés lényegesen összetettebb, műveletenként csak egyszer kell elvégezni.</span><span class="sxs-lookup"><span data-stu-id="00fe9-220">Although this query is considerably more complex, it is performed only once per operation.</span></span> <span data-ttu-id="00fe9-221">És noha az összetett illesztések költségessé válhatnak, a relációsadatbázis-rendszerek ilyen típusú lekérdezésekre vannak optimalizálva.</span><span class="sxs-lookup"><span data-stu-id="00fe9-221">And while complex joins can become expensive, relational database systems are optimized for this type of query.</span></span>  

![Lekérdezési adatok a sűrű API-hoz][queries4]

## <a name="related-resources"></a><span data-ttu-id="00fe9-223">Kapcsolódó források (lehet, hogy a cikkek angol nyelvűek)</span><span class="sxs-lookup"><span data-stu-id="00fe9-223">Related resources</span></span>

- <span data-ttu-id="00fe9-224">[API-tervezés – Ajánlott eljárások][api-design]</span><span class="sxs-lookup"><span data-stu-id="00fe9-224">[API Design best practices][api-design]</span></span>
- <span data-ttu-id="00fe9-225">[Gyorsítótárazás – Ajánlott eljárások][caching-guidance]</span><span class="sxs-lookup"><span data-stu-id="00fe9-225">[Caching best practices][caching-guidance]</span></span>
- <span data-ttu-id="00fe9-226">[Adatkonzisztencia – Ismertető][data-consistency-guidance]</span><span class="sxs-lookup"><span data-stu-id="00fe9-226">[Data Consistency Primer][data-consistency-guidance]</span></span>
- <span data-ttu-id="00fe9-227">[Felesleges beolvasások– kizárási minta][extraneous-fetching]</span><span class="sxs-lookup"><span data-stu-id="00fe9-227">[Extraneous Fetching antipattern][extraneous-fetching]</span></span>
- <span data-ttu-id="00fe9-228">[Nincs gyorsítótárazás – kizárási minta][no-cache]</span><span class="sxs-lookup"><span data-stu-id="00fe9-228">[No Caching antipattern][no-cache]</span></span>

[api-design]: ../../best-practices/api-design.md
[caching-guidance]: ../../best-practices/caching.md
[code-sample]:  https://github.com/mspnp/performance-optimization/tree/master/ChattyIO
[data-consistency-guidance]: http://https://msdn.microsoft.com/library/dn589800.aspx
[ef]: /ef/
[extraneous-fetching]: ../extraneous-fetching/index.md
[new-relic]: https://newrelic.com/application-monitoring
[no-cache]: ../no-caching/index.md

[key-indicators-chatty-io]: _images/ChattyIO.jpg
[key-indicators-chunky-io]: _images/ChunkyIO.jpg
[databasetraffic]: _images/DatabaseTraffic.jpg
[databasetraffic2]: _images/DatabaseTraffic2.jpg
[queries]: _images/DatabaseQueries.jpg
[queries2]: _images/DatabaseQueries2.jpg
[queries3]: _images/DatabaseQueries3.jpg
[queries4]: _images/DatabaseQueries4.jpg

