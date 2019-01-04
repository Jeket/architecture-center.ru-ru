---
title: Антишаблон отправки множественных операций ввода-вывода
titleSuffix: Performance antipatterns for cloud apps
description: Большое количество запросов ввода-вывода может привести к снижению производительности и скорости реагирования.
author: dragon119
ms.date: 06/05/2017
ms.custom: seodec18
ms.openlocfilehash: c018e365d0a6244f77d119ad59f601e9c7ea965c
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011231"
---
# <a name="chatty-io-antipattern"></a><span data-ttu-id="51583-103">Антишаблон отправки множественных операций ввода-вывода</span><span class="sxs-lookup"><span data-stu-id="51583-103">Chatty I/O antipattern</span></span>

<span data-ttu-id="51583-104">Большое число запросов ввода-вывода может оказать существенное влияние на производительность и скорость реагирования.</span><span class="sxs-lookup"><span data-stu-id="51583-104">The cumulative effect of a large number of I/O requests can have a significant impact on performance and responsiveness.</span></span>

## <a name="problem-description"></a><span data-ttu-id="51583-105">Описание проблемы</span><span class="sxs-lookup"><span data-stu-id="51583-105">Problem description</span></span>

<span data-ttu-id="51583-106">Сетевые вызовы и другие операции ввода-вывода очень медленные по сравнению с вычислительными задачами.</span><span class="sxs-lookup"><span data-stu-id="51583-106">Network calls and other I/O operations are inherently slow compared to compute tasks.</span></span> <span data-ttu-id="51583-107">Каждый запрос ввода-вывода обычно имеет значительную нагрузку, и общее влияние огромного числа операций ввода-вывода может снизить производительность системы.</span><span class="sxs-lookup"><span data-stu-id="51583-107">Each I/O request typically has significant overhead, and the cumulative effect of numerous I/O operations can slow down the system.</span></span> <span data-ttu-id="51583-108">Ниже приведены некоторые из основных причин отправки множественных операций ввода-вывода.</span><span class="sxs-lookup"><span data-stu-id="51583-108">Here are some common causes of chatty I/O.</span></span>

### <a name="reading-and-writing-individual-records-to-a-database-as-distinct-requests"></a><span data-ttu-id="51583-109">Чтение и запись отдельных записей в базе данных в качестве уникальных запросов</span><span class="sxs-lookup"><span data-stu-id="51583-109">Reading and writing individual records to a database as distinct requests</span></span>

<span data-ttu-id="51583-110">В указанном ниже примере считываются записи из базы данных продуктов.</span><span class="sxs-lookup"><span data-stu-id="51583-110">The following example reads from a database of products.</span></span> <span data-ttu-id="51583-111">Имеется три таблицы: `Product`, `ProductSubcategory` и `ProductPriceListHistory`.</span><span class="sxs-lookup"><span data-stu-id="51583-111">There are three tables, `Product`, `ProductSubcategory`, and `ProductPriceListHistory`.</span></span> <span data-ttu-id="51583-112">Указанный ниже код извлекает все продукты в подкатегории, а также сведения о ценах путем выполнения последовательности запросов.</span><span class="sxs-lookup"><span data-stu-id="51583-112">The code retrieves all of the products in a subcategory, along with the pricing information, by executing a series of queries:</span></span>  

1. <span data-ttu-id="51583-113">Выполните запрос к подкатегории из таблицы `ProductSubcategory`.</span><span class="sxs-lookup"><span data-stu-id="51583-113">Query the subcategory from the `ProductSubcategory` table.</span></span>
2. <span data-ttu-id="51583-114">Найдите все продукты в этой подкатегории, выполнив запрос к таблице `Product`.</span><span class="sxs-lookup"><span data-stu-id="51583-114">Find all products in that subcategory by querying the `Product` table.</span></span>
3. <span data-ttu-id="51583-115">Для каждого продукта запросите данные о ценах из таблицы `ProductPriceListHistory`.</span><span class="sxs-lookup"><span data-stu-id="51583-115">For each product, query the pricing data from the `ProductPriceListHistory` table.</span></span>

<span data-ttu-id="51583-116">Приложение использует [Entity Framework][ef], чтобы запросить базу данных.</span><span class="sxs-lookup"><span data-stu-id="51583-116">The application uses [Entity Framework][ef] to query the database.</span></span> <span data-ttu-id="51583-117">Полный пример см. [здесь][code-sample].</span><span class="sxs-lookup"><span data-stu-id="51583-117">You can find the complete sample [here][code-sample].</span></span>

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

<span data-ttu-id="51583-118">В этом примере проблема показана явным образом, но иногда объектно-реляционное отображение может маскировать проблему, если она неявно получает дочерние записи по одной.</span><span class="sxs-lookup"><span data-stu-id="51583-118">This example shows the problem explicitly, but sometimes an O/RM can mask the problem, if it implicitly fetches child records one at a time.</span></span> <span data-ttu-id="51583-119">Это называется проблемой N+1.</span><span class="sxs-lookup"><span data-stu-id="51583-119">This is known as the "N+1 problem".</span></span>

### <a name="implementing-a-single-logical-operation-as-a-series-of-http-requests"></a><span data-ttu-id="51583-120">Реализация одной логической операции в виде последовательности HTTP-запросов</span><span class="sxs-lookup"><span data-stu-id="51583-120">Implementing a single logical operation as a series of HTTP requests</span></span>

<span data-ttu-id="51583-121">Это часто происходит, когда разработчики пытаются следовать объектно-ориентированной парадигме и обрабатывать удаленные объекты как локальные объекты в памяти.</span><span class="sxs-lookup"><span data-stu-id="51583-121">This often happens when developers try to follow an object-oriented paradigm, and treat remote objects as if they were local objects in memory.</span></span> <span data-ttu-id="51583-122">Это может вызвать множество круговых путей в сети.</span><span class="sxs-lookup"><span data-stu-id="51583-122">This can result in too many network round trips.</span></span> <span data-ttu-id="51583-123">Например, следующий веб-API предоставляет отдельные свойства объектов `User` с помощью отдельных методов HTTP GET.</span><span class="sxs-lookup"><span data-stu-id="51583-123">For example, the following web API exposes the individual properties of `User` objects through individual HTTP GET methods.</span></span>

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

<span data-ttu-id="51583-124">Хотя с технической точки зрения при таком подходе все правильно, большинству клиентов, скорее всего, понадобится получить несколько свойств для каждого `User`. В результате будет получен следующий код клиента.</span><span class="sxs-lookup"><span data-stu-id="51583-124">While there's nothing technically wrong with this approach, most clients will probably need to get several properties for each `User`, resulting in client code like the following.</span></span>

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

### <a name="reading-and-writing-to-a-file-on-disk"></a><span data-ttu-id="51583-125">Запись данных в файл на диске и их считывание</span><span class="sxs-lookup"><span data-stu-id="51583-125">Reading and writing to a file on disk</span></span>

<span data-ttu-id="51583-126">При операции файлового ввода-вывода выполняется открытие и перемещение файла в соответствующую точку перед считыванием или записью данных.</span><span class="sxs-lookup"><span data-stu-id="51583-126">File I/O involves opening a file and moving to the appropriate point before reading or writing data.</span></span> <span data-ttu-id="51583-127">По завершении операции файл можно закрыть, чтобы сохранить ресурсы операционной системы.</span><span class="sxs-lookup"><span data-stu-id="51583-127">When the operation is complete, the file might be closed to save operating system resources.</span></span> <span data-ttu-id="51583-128">Приложение, которое постоянно записывает небольшие объемы информации в файл и считывает их, значительно увеличивает нагрузку операций ввода-вывода.</span><span class="sxs-lookup"><span data-stu-id="51583-128">An application that continually reads and writes small amounts of information to a file will generate significant I/O overhead.</span></span> <span data-ttu-id="51583-129">Небольшие запросы на запись также могут вызвать фрагментацию файла, замедлив последующие операции ввода-вывода.</span><span class="sxs-lookup"><span data-stu-id="51583-129">Small write requests can also lead to file fragmentation, slowing subsequent I/O operations still further.</span></span>

<span data-ttu-id="51583-130">В указанном ниже примере используется `FileStream` для записи объекта `Customer` в файл.</span><span class="sxs-lookup"><span data-stu-id="51583-130">The following example uses a `FileStream` to write a `Customer` object to a file.</span></span> <span data-ttu-id="51583-131">Создайте иди удалите `FileStream`, чтобы открыть или закрыть файл соответственно.</span><span class="sxs-lookup"><span data-stu-id="51583-131">Creating the `FileStream` opens the file, and disposing it closes the file.</span></span> <span data-ttu-id="51583-132">(Инструкция `using` автоматически удаляет объект `FileStream`.) Если приложение многократно вызывает этот метод при добавлении новых клиентов, нагрузка операций ввода-вывода может быстро увеличиться.</span><span class="sxs-lookup"><span data-stu-id="51583-132">(The `using` statement automatically disposes the `FileStream` object.) If the application calls this method repeatedly as new customers are added, the I/O overhead can accumulate quickly.</span></span>

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

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="51583-133">Как устранить проблему</span><span class="sxs-lookup"><span data-stu-id="51583-133">How to fix the problem</span></span>

<span data-ttu-id="51583-134">Сократите число запросов ввода-вывода, упаковав данные в меньшее количество запросов большего размера.</span><span class="sxs-lookup"><span data-stu-id="51583-134">Reduce the number of I/O requests by packaging the data into larger, fewer requests.</span></span>

<span data-ttu-id="51583-135">Вместо того чтобы выполнять несколько небольших запросов, извлеките данные из базы данных с помощью одного запроса.</span><span class="sxs-lookup"><span data-stu-id="51583-135">Fetch data from a database as a single query, instead of several smaller queries.</span></span> <span data-ttu-id="51583-136">Ниже приведена измененная версия кода, который извлекает сведения о продукте.</span><span class="sxs-lookup"><span data-stu-id="51583-136">Here's a revised version of the code that retrieves product information.</span></span>

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

<span data-ttu-id="51583-137">Следуйте принципам проектирования REST для веб-API.</span><span class="sxs-lookup"><span data-stu-id="51583-137">Follow REST design principles for web APIs.</span></span> <span data-ttu-id="51583-138">Ниже приведена измененная версия веб-API из примера выше.</span><span class="sxs-lookup"><span data-stu-id="51583-138">Here's a revised version of the web API from the earlier example.</span></span> <span data-ttu-id="51583-139">Вместо того чтобы выполнять методы GET отдельно для каждого свойства, имеется один метод GET, который возвращает `User`.</span><span class="sxs-lookup"><span data-stu-id="51583-139">Instead of separate GET methods for each property, there is a single GET method that returns the `User`.</span></span> <span data-ttu-id="51583-140">Это увеличивает текст ответа на запрос, однако каждый клиент сможет снизить число вызовов API.</span><span class="sxs-lookup"><span data-stu-id="51583-140">This results in a larger response body per request, but each client is likely to make fewer API calls.</span></span>

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

<span data-ttu-id="51583-141">Для файлового ввода-вывода рекомендуется выполнить буферизацию данных в памяти, а затем запись буферизованных данных в файл в рамках одной операции.</span><span class="sxs-lookup"><span data-stu-id="51583-141">For file I/O, consider buffering data in memory and then writing the buffered data to a file as a single operation.</span></span> <span data-ttu-id="51583-142">Такой подход снижает дополнительную нагрузку, связанную с неоднократным открытием и закрытием файла, и помогает снизить уровень фрагментации файла на диске.</span><span class="sxs-lookup"><span data-stu-id="51583-142">This approach reduces the overhead from repeatedly opening and closing the file, and helps to reduce fragmentation of the file on disk.</span></span>

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

## <a name="considerations"></a><span data-ttu-id="51583-143">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="51583-143">Considerations</span></span>

- <span data-ttu-id="51583-144">В первых двух примерах выполняется *меньшее* число вызовов ввода-вывода, но в каждом из них можно получить *большее* количество информации.</span><span class="sxs-lookup"><span data-stu-id="51583-144">The first two examples make *fewer* I/O calls, but each one retrieves *more* information.</span></span> <span data-ttu-id="51583-145">Необходимо найти компромисс между этими двумя факторами.</span><span class="sxs-lookup"><span data-stu-id="51583-145">You must consider the tradeoff between these two factors.</span></span> <span data-ttu-id="51583-146">Правильный ответ будет зависеть от фактических вариантов использования.</span><span class="sxs-lookup"><span data-stu-id="51583-146">The right answer will depend on the actual usage patterns.</span></span> <span data-ttu-id="51583-147">Например, в примере веб-API может оказаться, что клиентам часто требуется только имя клиента.</span><span class="sxs-lookup"><span data-stu-id="51583-147">For example, in the web API example, it might turn out that clients often need just the user name.</span></span> <span data-ttu-id="51583-148">В этом случае может быть целесообразно предоставлять его как отдельный вызов API.</span><span class="sxs-lookup"><span data-stu-id="51583-148">In that case, it might make sense to expose it as a separate API call.</span></span> <span data-ttu-id="51583-149">Дополнительные сведения см. в статье [Антишаблон лишней выборки][extraneous-fetching].</span><span class="sxs-lookup"><span data-stu-id="51583-149">For more information, see the [Extraneous Fetching][extraneous-fetching] antipattern.</span></span>

- <span data-ttu-id="51583-150">При считывании данных не делайте запросы ввода-вывода слишком большими.</span><span class="sxs-lookup"><span data-stu-id="51583-150">When reading data, do not make your I/O requests too large.</span></span> <span data-ttu-id="51583-151">Приложения должны извлекать только необходимую информацию.</span><span class="sxs-lookup"><span data-stu-id="51583-151">An application should only retrieve the information that it is likely to use.</span></span>

- <span data-ttu-id="51583-152">Иногда полезно секционировать информацию для объекта на два блока: *часто используемые данные*, которые учитывают большинство запросов, и *данные, доступ к которым осуществляется нечасто*.</span><span class="sxs-lookup"><span data-stu-id="51583-152">Sometimes it helps to partition the information for an object into two chunks, *frequently accessed data* that accounts for most requests, and *less frequently accessed data* that is used rarely.</span></span> <span data-ttu-id="51583-153">Как правило, наиболее часто используемые данные — это относительно небольшая часть общего объема данных для объекта. Поэтому, возвращая лишь эту часть, можно значительно снизить нагрузку операций ввода-вывода.</span><span class="sxs-lookup"><span data-stu-id="51583-153">Often the most frequently accessed data is a relatively small portion of the total data for an object, so returning just that portion can save significant I/O overhead.</span></span>

- <span data-ttu-id="51583-154">При записи данных не блокируйте ресурсы дольше, чем это необходимо. Это снизит вероятность конфликтов во время долгих операций.</span><span class="sxs-lookup"><span data-stu-id="51583-154">When writing data, avoid locking resources for longer than necessary, to reduce the chances of contention during a lengthy operation.</span></span> <span data-ttu-id="51583-155">Если операция записи охватывает несколько хранилищ данных, файлов или служб, используйте другой в конечном счете согласованный подход.</span><span class="sxs-lookup"><span data-stu-id="51583-155">If a write operation spans multiple data stores, files, or services, then adopt an eventually consistent approach.</span></span> <span data-ttu-id="51583-156">Ознакомьтесь с [руководством по согласованности данных][data-consistency-guidance].</span><span class="sxs-lookup"><span data-stu-id="51583-156">See [Data Consistency guidance][data-consistency-guidance].</span></span>

- <span data-ttu-id="51583-157">Если выполнить буферизацию данных в памяти до их записи, данные будут уязвимы в случае сбоя процесса.</span><span class="sxs-lookup"><span data-stu-id="51583-157">If you buffer data in memory before writing it, the data is vulnerable if the process crashes.</span></span> <span data-ttu-id="51583-158">Если скорость передачи данных имеет скачки или является относительно разреженной, будет безопаснее выполнить буферизацию данных во внешней долгосрочной очереди, например в [Центрах событий](https://azure.microsoft.com/services/event-hubs/).</span><span class="sxs-lookup"><span data-stu-id="51583-158">If the data rate typically has bursts or is relatively sparse, it may be safer to buffer the data in an external durable queue such as [Event Hubs](https://azure.microsoft.com/services/event-hubs/).</span></span>

- <span data-ttu-id="51583-159">Рассмотрите возможность кэширования данных, извлекаемых из службы или базы данных.</span><span class="sxs-lookup"><span data-stu-id="51583-159">Consider caching data that you retrieve from a service or a database.</span></span> <span data-ttu-id="51583-160">Это снизит объем операций ввода-вывода, позволяя избежать повторных запросов для одних и тех же данных.</span><span class="sxs-lookup"><span data-stu-id="51583-160">This can help to reduce the volume of I/O by avoiding repeated requests for the same data.</span></span> <span data-ttu-id="51583-161">Дополнительные сведения см. в статье о [кэшировании][caching-guidance].</span><span class="sxs-lookup"><span data-stu-id="51583-161">For more information, see [Caching best practices][caching-guidance].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="51583-162">Как определить проблему</span><span class="sxs-lookup"><span data-stu-id="51583-162">How to detect the problem</span></span>

<span data-ttu-id="51583-163">Признаками отправки множественных операций ввода-вывода являются высокая задержка и низкая пропускная способность.</span><span class="sxs-lookup"><span data-stu-id="51583-163">Symptoms of chatty I/O include high latency and low throughput.</span></span> <span data-ttu-id="51583-164">Пользователи могут сообщить об увеличении времени отклика или ошибках, вызванных завершением времени ожидания служб, из-за увеличения числа конфликтов ресурсов операций ввода-вывода.</span><span class="sxs-lookup"><span data-stu-id="51583-164">End users are likely to report extended response times or failures caused by services timing out, due to increased contention for I/O resources.</span></span>

<span data-ttu-id="51583-165">Чтобы определить причину любой проблемы, сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="51583-165">You can perform the following steps to help identify the causes of any problems:</span></span>

1. <span data-ttu-id="51583-166">Выполните мониторинг рабочей системы, чтобы определить операции с низким временем отклика.</span><span class="sxs-lookup"><span data-stu-id="51583-166">Perform process monitoring of the production system to identify operations with poor response times.</span></span>
2. <span data-ttu-id="51583-167">Выполните нагрузочное тестирование каждой операции, указанной на предыдущем шаге.</span><span class="sxs-lookup"><span data-stu-id="51583-167">Perform load testing of each operation identified in the previous step.</span></span>
3. <span data-ttu-id="51583-168">Во время нагрузочных тестирований соберите данные телеметрии о запросах на доступ к данным, выполненных каждой операцией.</span><span class="sxs-lookup"><span data-stu-id="51583-168">During the load tests, gather telemetry data about the data access requests made by each operation.</span></span>
4. <span data-ttu-id="51583-169">Соберите подробную статистику для всех запросов, отправленных в хранилище данных.</span><span class="sxs-lookup"><span data-stu-id="51583-169">Gather detailed statistics for each request sent to a data store.</span></span>
5. <span data-ttu-id="51583-170">Выполните профилирование приложения в тестовой среде, чтобы определить, где могут возникнуть проблемы с производительностью операций ввода-вывода.</span><span class="sxs-lookup"><span data-stu-id="51583-170">Profile the application in the test environment to establish where possible I/O bottlenecks might be occurring.</span></span>

<span data-ttu-id="51583-171">Обратите внимание, имеются ли любые из следующих признаков:</span><span class="sxs-lookup"><span data-stu-id="51583-171">Look for any of these symptoms:</span></span>

- <span data-ttu-id="51583-172">Большое количество небольших запросов ввода-вывода к одному и тому же файлу.</span><span class="sxs-lookup"><span data-stu-id="51583-172">A large number of small I/O requests made to the same file.</span></span>
- <span data-ttu-id="51583-173">Большое количество небольших сетевых запросов, выполненных экземпляром приложения к одной и той же службе.</span><span class="sxs-lookup"><span data-stu-id="51583-173">A large number of small network requests made by an application instance to the same service.</span></span>
- <span data-ttu-id="51583-174">Большое количество небольших запросов, выполненных экземпляром приложения к одному и тому же хранилищу данных.</span><span class="sxs-lookup"><span data-stu-id="51583-174">A large number of small requests made by an application instance to the same data store.</span></span>
- <span data-ttu-id="51583-175">Приложения и службы, у которых возникают ограничения ввода-вывода.</span><span class="sxs-lookup"><span data-stu-id="51583-175">Applications and services becoming I/O bound.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="51583-176">Пример диагностики</span><span class="sxs-lookup"><span data-stu-id="51583-176">Example diagnosis</span></span>

<span data-ttu-id="51583-177">В следующих разделах эти шаги применяются к указанному выше примеру, который запрашивает базу данных.</span><span class="sxs-lookup"><span data-stu-id="51583-177">The following sections apply these steps to the example shown earlier that queries a database.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="51583-178">Нагрузочное тестирование приложения</span><span class="sxs-lookup"><span data-stu-id="51583-178">Load test the application</span></span>

<span data-ttu-id="51583-179">На приведенном ниже графике показаны результаты нагрузочного тестирования.</span><span class="sxs-lookup"><span data-stu-id="51583-179">This graph shows the results of load testing.</span></span> <span data-ttu-id="51583-180">Среднее время отклика измеряется десятками секунд на запрос.</span><span class="sxs-lookup"><span data-stu-id="51583-180">Median response time is measured in 10s of seconds per request.</span></span> <span data-ttu-id="51583-181">На графике показана очень большая задержка.</span><span class="sxs-lookup"><span data-stu-id="51583-181">The graph shows very high latency.</span></span> <span data-ttu-id="51583-182">С нагрузкой в 1000 пользователей результаты запроса для одного пользователя могут возвращаться где-то через минуту.</span><span class="sxs-lookup"><span data-stu-id="51583-182">With a load of 1000 users, a user might have to wait for nearly a minute to see the results of a query.</span></span>

![Результаты нагрузочного теста ключевых индикаторов для примера приложения с множественными операциями ввода-вывода][key-indicators-chatty-io]

> [!NOTE]
> <span data-ttu-id="51583-184">Приложение было развернуто как веб-приложение службы приложений Azure с помощью базы данных SQL Azure.</span><span class="sxs-lookup"><span data-stu-id="51583-184">The application was deployed as an Azure App Service web app, using Azure SQL Database.</span></span> <span data-ttu-id="51583-185">При нагрузочном тесте использовалась смоделированная поэтапная рабочая нагрузка до 1000 параллельных пользователей.</span><span class="sxs-lookup"><span data-stu-id="51583-185">The load test used a simulated step workload of up to 1000 concurrent users.</span></span> <span data-ttu-id="51583-186">Базы данных была настроена с пулом подключений с поддержкой до 1000 параллельных пользователей, чтобы снизить вероятность влияния конфликта подключений на результаты.</span><span class="sxs-lookup"><span data-stu-id="51583-186">The database was configured with a connection pool supporting up to 1000 concurrent connections, to reduce the chance that contention for connections would affect the results.</span></span>

### <a name="monitor-the-application"></a><span data-ttu-id="51583-187">Мониторинг приложения</span><span class="sxs-lookup"><span data-stu-id="51583-187">Monitor the application</span></span>

<span data-ttu-id="51583-188">Вы можете использовать пакет мониторинга производительности приложения, чтобы собирать и анализировать ключевые метрики, которые могут определить множественные операции ввода-вывода.</span><span class="sxs-lookup"><span data-stu-id="51583-188">You can use an application performance monitoring (APM) package to capture and analyze the key metrics that might identify chatty I/O.</span></span> <span data-ttu-id="51583-189">Какие из метрик важные, определяется в зависимости от рабочей нагрузки операций ввода-вывода.</span><span class="sxs-lookup"><span data-stu-id="51583-189">Which metrics are important will depend on the I/O workload.</span></span> <span data-ttu-id="51583-190">В этом примере интересными запросами ввода-вывода были запросы базы данных.</span><span class="sxs-lookup"><span data-stu-id="51583-190">For this example, the interesting I/O requests were the database queries.</span></span>

<span data-ttu-id="51583-191">На указанном ниже рисунке показаны результаты, созданные с помощью [New Relic APM][new-relic].</span><span class="sxs-lookup"><span data-stu-id="51583-191">The following image shows results generated using [New Relic APM][new-relic].</span></span> <span data-ttu-id="51583-192">Среднее время отклика базы данных достигает максимума приблизительно за 5,6 секунд на запрос во время максимальной рабочей нагрузки.</span><span class="sxs-lookup"><span data-stu-id="51583-192">The average database response time peaked at approximately 5.6 seconds per request during the maximum workload.</span></span> <span data-ttu-id="51583-193">На протяжении теста система в среднем поддерживала около 410 запросов в минуту.</span><span class="sxs-lookup"><span data-stu-id="51583-193">The system was able to support an average of 410 requests per minute throughout the test.</span></span>

![Обзор трафика, поступающего в базу данных AdventureWorks2012][databasetraffic]

### <a name="gather-detailed-data-access-information"></a><span data-ttu-id="51583-195">Сбор подробных сведений о доступе к данным</span><span class="sxs-lookup"><span data-stu-id="51583-195">Gather detailed data access information</span></span>

<span data-ttu-id="51583-196">Более подробное изучение данных мониторинга показывает, что приложение выполняет три разные инструкции SQL SELECT.</span><span class="sxs-lookup"><span data-stu-id="51583-196">Digging deeper into the monitoring data shows the application executes three different SQL SELECT statements.</span></span> <span data-ttu-id="51583-197">Они соответствуют запросам, созданным Entity Framework для получения данных из таблиц `ProductListPriceHistory`, `Product` и `ProductSubcategory`.</span><span class="sxs-lookup"><span data-stu-id="51583-197">These correspond to the requests generated by Entity Framework to fetch data from the `ProductListPriceHistory`, `Product`, and `ProductSubcategory` tables.</span></span>
<span data-ttu-id="51583-198">Кроме того, запрос, который извлекает данные из таблицы `ProductListPriceHistory`, — это самая часто выполняемая инструкция SELECT в порядке величины.</span><span class="sxs-lookup"><span data-stu-id="51583-198">Furthermore, the query that retrieves data from the `ProductListPriceHistory` table is by far the most frequently executed SELECT statement, by an order of magnitude.</span></span>

![Запросы, выполняемые тестируемым примером приложения][queries]

<span data-ttu-id="51583-200">Оказывается, что показанный выше метод `GetProductsInSubCategoryAsync` выполняет 45 запросов SELECT.</span><span class="sxs-lookup"><span data-stu-id="51583-200">It turns out that the `GetProductsInSubCategoryAsync` method, shown earlier, performs 45 SELECT queries.</span></span> <span data-ttu-id="51583-201">Каждый запрос приводит к открытию нового подключения SQL приложением.</span><span class="sxs-lookup"><span data-stu-id="51583-201">Each query causes the application to open a new SQL connection.</span></span>

![Статистика запросов для тестируемого примера приложения][queries2]

> [!NOTE]
> <span data-ttu-id="51583-203">На этом изображении показаны сведения о трассировке для самого медленного экземпляра операции `GetProductsInSubCategoryAsync` в нагрузочном тесте.</span><span class="sxs-lookup"><span data-stu-id="51583-203">This image shows trace information for the slowest instance of the `GetProductsInSubCategoryAsync` operation in the load test.</span></span> <span data-ttu-id="51583-204">В рабочей среде полезно проверять трассировки самых медленных экземпляров, чтобы увидеть, есть ли шаблон, который свидетельствует о проблеме.</span><span class="sxs-lookup"><span data-stu-id="51583-204">In a production environment, it's useful to examine traces of the slowest instances, to see if there is a pattern that suggests a problem.</span></span> <span data-ttu-id="51583-205">Если просто посмотреть на средние значения, можно не увидеть проблем, которые значительно усугубятся при нагрузке.</span><span class="sxs-lookup"><span data-stu-id="51583-205">If you just look at the average values, you might overlook problems that will get dramatically worse under load.</span></span>

<span data-ttu-id="51583-206">На указанном ниже изображении показаны фактические выданные инструкции SQL.</span><span class="sxs-lookup"><span data-stu-id="51583-206">The next image shows the actual SQL statements that were issued.</span></span> <span data-ttu-id="51583-207">Запрос, который извлекает сведения о ценах, выполняется для каждого отдельного продукта в подкатегории продуктов.</span><span class="sxs-lookup"><span data-stu-id="51583-207">The query that fetches price information is run for each individual product in the product subcategory.</span></span> <span data-ttu-id="51583-208">Использование соединения может значительно снизить количество вызовов базы данных.</span><span class="sxs-lookup"><span data-stu-id="51583-208">Using a join would considerably reduce the number of database calls.</span></span>

![Сведения о запросе для тестируемого примера приложения][queries3]

<span data-ttu-id="51583-210">Если вы используете объектно-реляционное отображение, например Entity Framework, выполнение трассировки запросов SQL может обеспечить глубокое понимание того, как объектно-реляционное отображение преобразовывает программные вызовы в инструкции SQL и определяет области, в которых доступ к данным может быть оптимизирован.</span><span class="sxs-lookup"><span data-stu-id="51583-210">If you are using an O/RM, such as Entity Framework, tracing the SQL queries can provide insight into how the O/RM translates programmatic calls into SQL statements, and indicate areas where data access might be optimized.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="51583-211">Реализация решения и проверка результатов</span><span class="sxs-lookup"><span data-stu-id="51583-211">Implement the solution and verify the result</span></span>

<span data-ttu-id="51583-212">Перезапись вызова в Entity Framework создает следующие результаты.</span><span class="sxs-lookup"><span data-stu-id="51583-212">Rewriting the call to Entity Framework produced the following results.</span></span>

![Результаты нагрузочного теста ключевых индикаторов для больших API в примере приложений с множественными операциями ввода-вывода][key-indicators-chunky-io]

<span data-ttu-id="51583-214">Этот нагрузочный тест выполнен в той же развернутой службе с использованием того же профиля нагрузки.</span><span class="sxs-lookup"><span data-stu-id="51583-214">This load test was performed on the same deployment, using the same load profile.</span></span> <span data-ttu-id="51583-215">В этот раз на графике отображается гораздо меньшая задержка.</span><span class="sxs-lookup"><span data-stu-id="51583-215">This time the graph shows much lower latency.</span></span> <span data-ttu-id="51583-216">Среднее время запроса для 1000 пользователей сократилось от 5 до 6 секунд с минуты.</span><span class="sxs-lookup"><span data-stu-id="51583-216">The average request time at 1000 users is between 5 and 6 seconds, down from nearly a minute.</span></span>

<span data-ttu-id="51583-217">В этот раз система поддерживала в среднем 3970 запросов в минуту по сравнению с 410 в более раннем тестировании.</span><span class="sxs-lookup"><span data-stu-id="51583-217">This time the system supported an average of 3,970 requests per minute, compared to 410 for the earlier test.</span></span>

![Общие сведения о транзакции для больших API][databasetraffic2]

<span data-ttu-id="51583-219">Трассировка инструкции SQL показывает, что все данные извлекаются в одной инструкции SELECT.</span><span class="sxs-lookup"><span data-stu-id="51583-219">Tracing the SQL statement shows that all the data is fetched in a single SELECT statement.</span></span> <span data-ttu-id="51583-220">Однако этот запрос является более сложным и для каждой операции выполняется только раз.</span><span class="sxs-lookup"><span data-stu-id="51583-220">Although this query is considerably more complex, it is performed only once per operation.</span></span> <span data-ttu-id="51583-221">И хотя сложные соединения могут стать затратными, реляционные СУБД оптимизированы для этого типа запроса.</span><span class="sxs-lookup"><span data-stu-id="51583-221">And while complex joins can become expensive, relational database systems are optimized for this type of query.</span></span>

![Сведения о запросе для больших API][queries4]

## <a name="related-resources"></a><span data-ttu-id="51583-223">Связанные ресурсы</span><span class="sxs-lookup"><span data-stu-id="51583-223">Related resources</span></span>

- <span data-ttu-id="51583-224">[Руководство по проектированию API][api-design]</span><span class="sxs-lookup"><span data-stu-id="51583-224">[API Design best practices][api-design]</span></span>
- <span data-ttu-id="51583-225">[Рекомендации по кэшированию][caching-guidance]</span><span class="sxs-lookup"><span data-stu-id="51583-225">[Caching best practices][caching-guidance]</span></span>
- <span data-ttu-id="51583-226">[Руководство по согласованности данных][data-consistency-guidance]</span><span class="sxs-lookup"><span data-stu-id="51583-226">[Data Consistency Primer][data-consistency-guidance]</span></span>
- <span data-ttu-id="51583-227">[Антишаблон лишней выборки][extraneous-fetching]</span><span class="sxs-lookup"><span data-stu-id="51583-227">[Extraneous Fetching antipattern][extraneous-fetching]</span></span>
- <span data-ttu-id="51583-228">[Антишаблон без кэширования][no-cache]</span><span class="sxs-lookup"><span data-stu-id="51583-228">[No Caching antipattern][no-cache]</span></span>

[api-design]: ../../best-practices/api-design.md
[caching-guidance]: ../../best-practices/caching.md
[code-sample]:  https://github.com/mspnp/performance-optimization/tree/master/ChattyIO
[data-consistency-guidance]: https://msdn.microsoft.com/library/dn589800.aspx
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
