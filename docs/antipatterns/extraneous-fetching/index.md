---
title: Антишаблон лишней выборки
description: Извлечение большего количества данных, чем требуется для бизнес-операции, может привести к ненужному снижению производительности операции ввода-вывода и снизить скорость реагирования.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 7a72bfd3e4b2e206f3266a046fac2083224ecb4f
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2018
---
# <a name="extraneous-fetching-antipattern"></a><span data-ttu-id="4f5e3-103">Антишаблон лишней выборки</span><span class="sxs-lookup"><span data-stu-id="4f5e3-103">Extraneous Fetching antipattern</span></span>

<span data-ttu-id="4f5e3-104">Извлечение большего количества данных, чем требуется для бизнес-операции, может привести к ненужному снижению производительности операции ввода-вывода и снизить скорость реагирования.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-104">Retrieving more data than needed for a business operation can result in unnecessary I/O overhead and reduce responsiveness.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="4f5e3-105">Описание проблемы</span><span class="sxs-lookup"><span data-stu-id="4f5e3-105">Problem description</span></span>

<span data-ttu-id="4f5e3-106">Антишаблон может возникнуть, если приложение пытается уменьшить количество запросов ввода-вывода путем извлечения всех данных, которые *могут* потребоваться.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-106">This antipattern can occur if the application tries to minimize I/O requests by retrieving all of the data that it *might* need.</span></span> <span data-ttu-id="4f5e3-107">Обычно это связано с излишней компенсацией для антишаблона [отправки множественных операций ввода-вывода][chatty-io].</span><span class="sxs-lookup"><span data-stu-id="4f5e3-107">This is often a result of overcompensating for the [Chatty I/O][chatty-io] antipattern.</span></span> <span data-ttu-id="4f5e3-108">Например, приложение может получать сведения о каждом продукте в базе данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-108">For example, an application might fetch the details for every product in a database.</span></span> <span data-ttu-id="4f5e3-109">Однако пользователю нужно только подмножество сведений (некоторые могут не интересовать клиентов) и не нужно видеть *все* продукты одновременно.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-109">But the user may need just a subset of the details (some may not be relevant to customers), and probably doesn't need to see *all* of the products at once.</span></span> <span data-ttu-id="4f5e3-110">Даже если пользователь просматривает весь каталог, имеет смысл разбить результаты. Например, чтобы одновременно просматривать только 20 результатов.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-110">Even if the user is browsing the entire catalog, it would make sense to paginate the results &mdash; showing 20 at a time, for example.</span></span>

<span data-ttu-id="4f5e3-111">Еще одна причина этой проблемы — выполнение некорректных рекомендаций по разработке и программированию.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-111">Another source of this problem is following poor programming or design practices.</span></span> <span data-ttu-id="4f5e3-112">Например, в указанном ниже коде используется Entity Framework, чтобы получить полные сведения по каждому продукту.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-112">For example, the following code uses Entity Framework to fetch the complete details for every product.</span></span> <span data-ttu-id="4f5e3-113">Затем результаты фильтруются, чтобы возвратить только подмножество полей, отклоняя все остальное.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-113">Then it filters the results to return only a subset of the fields, discarding the rest.</span></span> <span data-ttu-id="4f5e3-114">Полный пример см. [здесь][sample-app].</span><span class="sxs-lookup"><span data-stu-id="4f5e3-114">You can find the complete sample [here][sample-app].</span></span>

```csharp
public async Task<IHttpActionResult> GetAllFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Execute the query. This happens at the database.
        var products = await context.Products.ToListAsync();

        // Project fields from the query results. This happens in application memory.
        var result = products.Select(p => new ProductInfo { Id = p.ProductId, Name = p.Name });
        return Ok(result);
    }
}
```

<span data-ttu-id="4f5e3-115">В указанном ниже примере приложение извлекает данные, чтобы выполнить агрегирование, которое также можно выполнить в базе данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-115">In the next example, the application retrieves data to perform an aggregation that could be done by the database instead.</span></span> <span data-ttu-id="4f5e3-116">Приложение вычисляет общий объем продаж путем извлечения каждой записи для всех оплаченных заказов, а затем вычисляет суму этих записей.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-116">The application calculates total sales by getting every record for all orders sold, and then computing the sum over those records.</span></span> <span data-ttu-id="4f5e3-117">Полный пример см. [здесь][sample-app].</span><span class="sxs-lookup"><span data-stu-id="4f5e3-117">You can find the complete sample [here][sample-app].</span></span>

```csharp
public async Task<IHttpActionResult> AggregateOnClientAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Fetch all order totals from the database.
        var orderAmounts = await context.SalesOrderHeaders.Select(soh => soh.TotalDue).ToListAsync();

        // Sum the order totals in memory.
        var total = orderAmounts.Sum();
        return Ok(total);
    }
}
```

<span data-ttu-id="4f5e3-118">В следующем примере показана простая проблема, вызванная тем, как платформа Entity Framework использует LINQ to Entities.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-118">The next example shows a subtle problem caused by the way Entity Framework uses LINQ to Entities.</span></span> 

```csharp
var query = from p in context.Products.AsEnumerable()
            where p.SellStartDate < DateTime.Now.AddDays(-7) // AddDays cannot be mapped by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

<span data-ttu-id="4f5e3-119">Приложение пытается найти продукты, с даты продажи (`SellStartDate`) которых прошло больше недели.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-119">The application is trying to find products with a `SellStartDate` more than a week old.</span></span> <span data-ttu-id="4f5e3-120">В большинстве случаев LINQ to Entities будет преобразовывать предложение `where` в инструкцию SQL, которая выполняется в базе данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-120">In most cases, LINQ to Entities would translate a `where` clause to a SQL statement that is executed by the database.</span></span> <span data-ttu-id="4f5e3-121">В этом случает, однако, LINQ to Entities не может сопоставить метод `AddDays` с SQL.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-121">In this case, however, LINQ to Entities cannot map the `AddDays` method to SQL.</span></span> <span data-ttu-id="4f5e3-122">Вместо этого возвращается каждая строка таблицы `Product`, а результаты фильтруются в памяти.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-122">Instead, every row from the `Product` table is returned, and the results are filtered in memory.</span></span> 

<span data-ttu-id="4f5e3-123">Вызов метода `AsEnumerable` указывает, что имеется проблема.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-123">The call to `AsEnumerable` is a hint that there is a problem.</span></span> <span data-ttu-id="4f5e3-124">Этот метод преобразовывает результаты в интерфейс `IEnumerable`.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-124">This method converts the results to an `IEnumerable` interface.</span></span> <span data-ttu-id="4f5e3-125">Тем не менее `IEnumerable` поддерживает фильтрацию, которая выполняется на стороне *клиента*, а не базы данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-125">Although `IEnumerable` supports filtering, the filtering is done on the *client* side, not the database.</span></span> <span data-ttu-id="4f5e3-126">По умолчанию LINQ to Entities использует `IQueryable`, что передает ответственность за фильтрацию источнику данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-126">By default, LINQ to Entities uses `IQueryable`, which passes the responsibility for filtering to the data source.</span></span> 

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="4f5e3-127">Как устранить проблему</span><span class="sxs-lookup"><span data-stu-id="4f5e3-127">How to fix the problem</span></span>

<span data-ttu-id="4f5e3-128">Избегайте получения огромных объемов данных, которые могут быстро устареть или быть удалены. Извлекайте только данные, которые нужны для выполнения операции.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-128">Avoid fetching large volumes of data that may quickly become outdated or might be discarded, and only fetch the data needed for the operation being performed.</span></span> 

<span data-ttu-id="4f5e3-129">Вместо того чтобы извлекать каждый столбец из таблицы, а затем их фильтровать, выберите необходимые столбцы в базе данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-129">Instead of getting every column from a table and then filtering them, select the columns that you need from the database.</span></span>

```csharp
public async Task<IHttpActionResult> GetRequiredFieldsAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Project fields as part of the query itself
        var result = await context.Products
            .Select(p => new ProductInfo {Id = p.ProductId, Name = p.Name})
            .ToListAsync();
        return Ok(result);
    }
}
```

<span data-ttu-id="4f5e3-130">Аналогичным образом выполните агрегирование в базе данных, а не в памяти приложения.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-130">Similarly, perform aggregation in the database and not in application memory.</span></span>

```csharp
public async Task<IHttpActionResult> AggregateOnDatabaseAsync()
{
    using (var context = new AdventureWorksContext())
    {
        // Sum the order totals as part of the database query.
        var total = await context.SalesOrderHeaders.SumAsync(soh => soh.TotalDue);
        return Ok(total);
    }
}
```

<span data-ttu-id="4f5e3-131">При использовании Entity Framework убедитесь, что запросы LINQ разрешаются с помощью интерфейса `IQueryable`, а не `IEnumerable`.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-131">When using Entity Framework, ensure that LINQ queries are resolved using the `IQueryable`interface and not `IEnumerable`.</span></span> <span data-ttu-id="4f5e3-132">Возможно, может потребоваться настроить запрос для использования только функций, которые можно сопоставить с источником данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-132">You may need to adjust the query to use only functions that can be mapped to the data source.</span></span> <span data-ttu-id="4f5e3-133">Вы можете выполнить рефакторинг указанного выше примера, чтобы удалить метод `AddDays` из очереди, что позволяет выполнить фильтрацию в базе данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-133">The earlier example can be refactored to remove the `AddDays` method from the query, allowing filtering to be done by the database.</span></span>

```csharp
DateTime dateSince = DateTime.Now.AddDays(-7); // AddDays has been factored out. 
var query = from p in context.Products
            where p.SellStartDate < dateSince // This criterion can be passed to the database by LINQ to Entities
            select ...;

List<Product> products = query.ToList();
```

## <a name="considerations"></a><span data-ttu-id="4f5e3-134">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="4f5e3-134">Considerations</span></span>

- <span data-ttu-id="4f5e3-135">В некоторых случаях можно повысить производительность путем горизонтального секционирования данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-135">In some cases, you can improve performance by partitioning data horizontally.</span></span> <span data-ttu-id="4f5e3-136">Если различные операции получают доступ к различным атрибутам данных, горизонтальное секционирование может уменьшить количество конфликтов.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-136">If different operations access different attributes of the data, horizontal partitioning may reduce contention.</span></span> <span data-ttu-id="4f5e3-137">Зачастую большинство операций выполняются для небольшого подмножества данных, поэтому распределение этой нагрузки может повысить производительность.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-137">Often, most operations are run against a small subset of the data, so spreading this load may improve performance.</span></span> <span data-ttu-id="4f5e3-138">Ознакомьтесь со статьей [Секционирование данных][data-partitioning].</span><span class="sxs-lookup"><span data-stu-id="4f5e3-138">See [Data partitioning][data-partitioning].</span></span>

- <span data-ttu-id="4f5e3-139">Реализуйте разбиение и извлекайте только ограниченное количество записей одновременно в операциях, которые должны поддерживать неограниченные запросы.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-139">For operations that have to support unbounded queries, implement pagination and only fetch a limited number of entities at a time.</span></span> <span data-ttu-id="4f5e3-140">Например, если клиент просматривает каталог продуктов, за раз можно отображать только одну страницу результатов.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-140">For example, if a customer is browsing a product catalog, you can show one page of results at a time.</span></span>

- <span data-ttu-id="4f5e3-141">По возможности воспользуйтесь функциями, встроенными в хранилище данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-141">When possible, take advantage of features built into the data store.</span></span> <span data-ttu-id="4f5e3-142">Например, базы данных SQL обычно предоставляют агрегатные функции.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-142">For example, SQL databases typically provide aggregate functions.</span></span> 

- <span data-ttu-id="4f5e3-143">Если вы используете хранилище данных, которое не поддерживает соответствующую функцию, например агрегирование, можно хранить рассчитанный результат в другом месте и обновлять значения при добавлении и обновлении записей, чтобы приложение не пересчитывало значение каждый раз, когда оно требуется.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-143">If you're using a data store that doesn't support a particular function, such as aggregration, you could store the calculated result elsewhere, updating the value as records are added or updated, so the application doesn't have to recalculate the value each time it's needed.</span></span>

- <span data-ttu-id="4f5e3-144">Если вы видите, что запросы извлекают большое количество полей, просмотрите исходный код, чтобы определить, являются ли все эти поля обязательными.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-144">If you see that requests are retrieving a large number of fields, examine the source code to determine whether all of these fields are actually necessary.</span></span> <span data-ttu-id="4f5e3-145">Иногда эти запросы являются результатом неверно созданного запроса `SELECT *`.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-145">Sometimes these requests are the result of poorly designed `SELECT *` query.</span></span> 

- <span data-ttu-id="4f5e3-146">Аналогичным образом, если есть запросы, которые извлекают большое количество записей, это может означать, что приложение выполняет неправильную фильтрацию данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-146">Similarly, requests that retrieve a large number of entities may be sign that the application is not filtering data correctly.</span></span> <span data-ttu-id="4f5e3-147">Убедитесь, что все эти записи действительно необходимы.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-147">Verify that all of these entities are actually needed.</span></span> <span data-ttu-id="4f5e3-148">По возможности выполните фильтрацию на стороне базы данных, например с помощью предложений `WHERE` в SQL.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-148">Use database-side filtering if possible, for example, by using `WHERE` clauses in SQL.</span></span> 

- <span data-ttu-id="4f5e3-149">Разгрузка обработки в базу данных не всегда является лучшим вариантом.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-149">Offloading processing to the database is not always the best option.</span></span> <span data-ttu-id="4f5e3-150">Этот подход можно использовать, только если база данных разработана и оптимизирована для выполнения этой задачи.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-150">Only use this strategy when the database is designed or optimized to do so.</span></span> <span data-ttu-id="4f5e3-151">Большинство систем базы данных оптимизированы для выполнения некоторых функций, однако их нельзя использовать в качестве модулей приложения общего назначения.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-151">Most database systems are highly optimized for certain functions, but are not designed to act as general-purpose application engines.</span></span> <span data-ttu-id="4f5e3-152">Дополнительные сведения см. в статье [Антишаблон занятости базы данных][BusyDatabase].</span><span class="sxs-lookup"><span data-stu-id="4f5e3-152">For more information, see the [Busy Database antipattern][BusyDatabase].</span></span>


## <a name="how-to-detect-the-problem"></a><span data-ttu-id="4f5e3-153">Как определить проблему</span><span class="sxs-lookup"><span data-stu-id="4f5e3-153">How to detect the problem</span></span>

<span data-ttu-id="4f5e3-154">Признаками лишней выборки являются высокая задержка и низкая пропускная способность.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-154">Symptoms of extraneous fetching include high latency and low throughput.</span></span> <span data-ttu-id="4f5e3-155">Если данные извлекаются из хранилища данных, также может увеличиться число конфликтов.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-155">If the data is retrieved from a data store, increased contention is also probable.</span></span> <span data-ttu-id="4f5e3-156">Пользователи могут сообщить об увеличении времени отклика или ошибках, вызванных завершением времени ожидания служб. Эти сбои могут возвращать ошибки HTTP 500 (внутренняя ошибка сервера) или ошибки HTTP 503 (служба недоступна).</span><span class="sxs-lookup"><span data-stu-id="4f5e3-156">End users are likely to report extended response times or failures caused by services timing out. These failures could return HTTP 500 (Internal Server) errors or HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="4f5e3-157">Изучите журналы событий веб-сервера, которые, скорее всего, содержат более подробные сведения о причинах и обстоятельствах возникновения ошибок.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-157">Examine the event logs for the web server, which are likely to contain more detailed information about the causes and circumstances of the errors.</span></span>

<span data-ttu-id="4f5e3-158">Признаки этого антишаблона и некоторой полученной телеметрии могут быть очень похожи на признаки [антишаблона монолитной сохраняемости][MonolithicPersistence].</span><span class="sxs-lookup"><span data-stu-id="4f5e3-158">The symptoms of this antipattern and some of the telemetry obtained might be very similar to those of the [Monolithic Persistence antipattern][MonolithicPersistence].</span></span> 

<span data-ttu-id="4f5e3-159">Чтобы определить причину, сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="4f5e3-159">You can perform the following steps to help identify the cause:</span></span>

1. <span data-ttu-id="4f5e3-160">Определите медленные рабочие нагрузки или транзакции, выполнив нагрузочное тестирование, наблюдение за процессами или другие методы для сбора данных инструментирования.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-160">Identify slow workloads or transactions by performing load-testing, process monitoring, or other methods of capturing instrumentation data.</span></span>
2. <span data-ttu-id="4f5e3-161">Понаблюдайте за шаблонами поведения в системе.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-161">Observe any behavioral patterns exhibited by the system.</span></span> <span data-ttu-id="4f5e3-162">Проверьте, имеются ли определенные ограничения для количества транзакций в секунду или количества пользователей.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-162">Are there particular limits in terms of transactions per second or volume of users?</span></span>
3. <span data-ttu-id="4f5e3-163">Выполните корреляцию экземпляров медленных рабочих нагрузок с шаблонами поведения.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-163">Correlate the instances of slow workloads with behavioral patterns.</span></span>
4. <span data-ttu-id="4f5e3-164">Определите используемые хранилища данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-164">Identify the data stores being used.</span></span> <span data-ttu-id="4f5e3-165">Для каждого источника данных выполните анализ данных телеметрии нижнего уровня, чтобы просмотреть поведение операций.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-165">For each data source, run lower level telemetry to observe the behavior of operations.</span></span>
6. <span data-ttu-id="4f5e3-166">Определите все медленно выполняемые запросы, которые ссылаются на эти источники данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-166">Identify any slow-running queries that reference these data sources.</span></span>
7. <span data-ttu-id="4f5e3-167">Выполните анализ медленно выполняемых запросов для конкретных ресурсов и выясните, как используются и потребляются данные.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-167">Perform a resource-specific analysis of the slow-running queries and ascertain how the data is used and consumed.</span></span>

<span data-ttu-id="4f5e3-168">Обратите внимание, имеются ли любые из следующих признаков:</span><span class="sxs-lookup"><span data-stu-id="4f5e3-168">Look for any of these symptoms:</span></span>

- <span data-ttu-id="4f5e3-169">Частые и большие запросы ввода-вывода к тому самому ресурсу или хранилищу данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-169">Frequent, large I/O requests made to the same resource or data store.</span></span>
- <span data-ttu-id="4f5e3-170">Конфликты в общем ресурсе или хранилище данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-170">Contention in a shared resource or data store.</span></span>
- <span data-ttu-id="4f5e3-171">Операцию, которая часто получает большие объемы данных по сети.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-171">An operation that frequently receives large volumes of data over the network.</span></span>
- <span data-ttu-id="4f5e3-172">Приложения и службы, которые тратят много времени на ожидание завершения операций ввода-вывода.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-172">Applications and services spending significant time waiting for I/O to complete.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="4f5e3-173">Пример диагностики</span><span class="sxs-lookup"><span data-stu-id="4f5e3-173">Example diagnosis</span></span>    

<span data-ttu-id="4f5e3-174">Для указанных выше примеров можно выполнить действия из следующих разделов.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-174">The following sections apply these steps to the previous examples.</span></span>

### <a name="identify-slow-workloads"></a><span data-ttu-id="4f5e3-175">Определение медленных рабочих нагрузок</span><span class="sxs-lookup"><span data-stu-id="4f5e3-175">Identify slow workloads</span></span>

<span data-ttu-id="4f5e3-176">На приведенном ниже графике показаны результаты производительности из нагрузочного теста, в котором было смоделировано до 400 параллельных пользователей, выполняющих указанный выше метод `GetAllFieldsAsync`.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-176">This graph shows performance results from a load test that simulated up to 400 concurrent users running the `GetAllFieldsAsync` method shown earlier.</span></span> <span data-ttu-id="4f5e3-177">Пропускная способность медленно снижается по мере увеличения нагрузки.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-177">Throughput diminishes slowly as the load increases.</span></span> <span data-ttu-id="4f5e3-178">Среднее время отклика возрастает по мере увеличения рабочей нагрузки.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-178">Average response time goes up as the workload increases.</span></span> 

![Результаты нагрузочного теста метода GetAllFieldsAsync][Load-Test-Results-Client-Side1]

<span data-ttu-id="4f5e3-180">Нагрузочный текст операции `AggregateOnClientAsync` демонстрирует идентичный шаблон.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-180">A load test for the `AggregateOnClientAsync` operation shows a similar pattern.</span></span> <span data-ttu-id="4f5e3-181">Объем запросов достаточно стабильный.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-181">The volume of requests is reasonably stable.</span></span> <span data-ttu-id="4f5e3-182">Среднее время отклика увеличивается вместе с рабочей нагрузкой, однако более медленно, чем на предыдущем графике.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-182">The average response time increases with the workload, although more slowly than the previous graph.</span></span>

![Результаты нагрузочного теста метода AggregateOnClientAsync][Load-Test-Results-Client-Side2]

### <a name="correlate-slow-workloads-with-behavioral-patterns"></a><span data-ttu-id="4f5e3-184">Корреляция медленных рабочих нагрузок с шаблонами поведения</span><span class="sxs-lookup"><span data-stu-id="4f5e3-184">Correlate slow workloads with behavioral patterns</span></span>

<span data-ttu-id="4f5e3-185">Любая корреляция между регулярными периодами высокого использования и замедленной производительности может указать на проблемные области.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-185">Any correlation between regular periods of high usage and slowing performance can indicate areas of concern.</span></span> <span data-ttu-id="4f5e3-186">Тщательно изучите профиль производительности функциональности, которая предположительно медленно выполняется, чтобы определить, совпадает ли она с нагрузочным тестированием, выполненным ранее.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-186">Closely examine the performance profile of functionality that is suspected to be slow running, to determine whether it matches the load testing performed earlier.</span></span>

<span data-ttu-id="4f5e3-187">Выполните нагрузочный тест той же функциональности, используя поэтапные пользовательские нагрузки, чтобы определить точку, в которой производительность существенно снижается или происходит полный сбой.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-187">Load test the same functionality using step-based user loads, to find the point where performance drops significantly or fails completely.</span></span> <span data-ttu-id="4f5e3-188">Если эта точка находится в пределах границ ожидаемого использования в реальном времени, изучите способ реализации функциональности.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-188">If that point falls within the bounds of your expected real-world usage, examine how the functionality is implemented.</span></span>

<span data-ttu-id="4f5e3-189">Операция с низкой производительностью не всегда является проблемой, а именно если она не выполняется, когда система сильно загружена, не критически важная по времени и не влияет отрицательно на производительность важных операций.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-189">A slow operation is not necessarily a problem, if it is not being performed when the system is under stress, is not time critical, and does not negatively affect the performance of other important operations.</span></span> <span data-ttu-id="4f5e3-190">Например, создание ежемесячной операционной статистики может быть длительной операцией. Возможно, ее лучше выполнить как процесс пакетной обработки и запустить в качестве низкоприоритетного задания.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-190">For example, generating monthly operational statistics might be a long-running operation, but it can probably be performed as a batch process and run as a low priority job.</span></span> <span data-ttu-id="4f5e3-191">С другой стороны, запрос каталога продуктов клиентами — это критически важная бизнес-операция.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-191">On the other hand, customers querying the product catalog is a critical business operation.</span></span> <span data-ttu-id="4f5e3-192">Сосредоточьтесь на телеметрии, созданной этими критически важными операциями, чтобы увидеть, как изменяется производительность в периоды высокой нагрузки.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-192">Focus on the telemetry generated by these critical operations to see how the performance varies during periods of high usage.</span></span>

### <a name="identify-data-sources-in-slow-workloads"></a><span data-ttu-id="4f5e3-193">Определение источников данных в медленных рабочих нагрузках</span><span class="sxs-lookup"><span data-stu-id="4f5e3-193">Identify data sources in slow workloads</span></span>

<span data-ttu-id="4f5e3-194">Если вы подозреваете, что низкая производительность службы связана со способом извлечения данных, проследите, как приложение взаимодействует с используемыми репозиториями.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-194">If you suspect that a service is performing poorly because of the way it retrieves data, investigate how the application interacts with the repositories it uses.</span></span> <span data-ttu-id="4f5e3-195">Выполните мониторинг активной системы, чтобы узнать, доступ к каким ресурсам осуществляется во время периодов низкой производительности.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-195">Monitor the live system to see which sources are accessed during periods of poor performance.</span></span> 

<span data-ttu-id="4f5e3-196">Для каждого источника данных выполните инструментирование системы, чтобы получить следующие данные:</span><span class="sxs-lookup"><span data-stu-id="4f5e3-196">For each data source, instrument the system to capture the following:</span></span>

- <span data-ttu-id="4f5e3-197">Частоту доступа к каждому хранилищу данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-197">The frequency that each data store is accessed.</span></span>
- <span data-ttu-id="4f5e3-198">Объем входящих и исходящих данных в хранилище данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-198">The volume of data entering and exiting the data store.</span></span>
- <span data-ttu-id="4f5e3-199">Время этих операций, в частности задержки запросов.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-199">The timing of these operations, especially the latency of requests.</span></span>
- <span data-ttu-id="4f5e3-200">Характер и частоту всех ошибок, возникающих при доступе к каждому хранилищу данных в условиях обычной загрузки.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-200">The nature and rate of any errors that occur while accessing each data store under typical load.</span></span>

<span data-ttu-id="4f5e3-201">Сравните эти сведения с объемом данных, возвращенных приложением клиенту.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-201">Compare this information against the volume of data being returned by the application to the client.</span></span> <span data-ttu-id="4f5e3-202">Отслеживайте соотношение объема данных, возвращенного хранилищем данных, с объемом данных, возвращенным клиенту.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-202">Track the ratio of the volume of data returned by the data store against the volume of data returned to the client.</span></span> <span data-ttu-id="4f5e3-203">В случае больших различий определите, извлекает ли приложение ненужные данные.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-203">If there is any large disparity, investigate to determine whether the application is fetching data that it doesn't need.</span></span>

<span data-ttu-id="4f5e3-204">Вы можете получить эти данные, отслеживая активную систему и выполнив трассировку жизненного цикла каждого пользовательского запроса. Кроме того, можно смоделировать набор искусственных рабочих нагрузок и выполнить их в тестовой системе.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-204">You may be able to capture this data by observing the live system and tracing the lifecycle of each user request, or you can model a series of synthetic workloads and run them against a test system.</span></span>

<span data-ttu-id="4f5e3-205">На приведенных ниже графиках показаны данные телеметрии, собранные с помощью [New Relic APM][new-relic] во время выполнения нагрузочного теста метода `GetAllFieldsAsync`.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-205">The following graphs show telemetry captured using [New Relic APM][new-relic] during a load test of the `GetAllFieldsAsync` method.</span></span> <span data-ttu-id="4f5e3-206">Обратите внимание на разницу между объемами данных, полученными из базы данных, и соответствующих ответов HTTP.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-206">Note the difference between the volumes of data received from the database and the corresponding HTTP responses.</span></span>

![Данные телеметрии для метода GetAllFieldsAsync][TelemetryAllFields]

<span data-ttu-id="4f5e3-208">Для каждого запроса база данных вернула 80 503 байт, но ответ клиенту содержит только 19 855 байт (около 25 % от размера ответа базы данных).</span><span class="sxs-lookup"><span data-stu-id="4f5e3-208">For each request, the database returned 80,503 bytes, but the response to the client only contained 19,855 bytes, about 25% of the size of the database response.</span></span> <span data-ttu-id="4f5e3-209">Размер данных, возвращенных клиенту, может отличаться в зависимости от формата.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-209">The size of the data returned to the client can vary depending on the format.</span></span> <span data-ttu-id="4f5e3-210">Для этого нагрузочного теста клиент запросил данные JSON.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-210">For this load test, the client requested JSON data.</span></span> <span data-ttu-id="4f5e3-211">Размер ответа отдельного тестирования с помощью XML (не показано) составлял 35 655 байт или 44 % от размера ответа базы данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-211">Separate testing using XML (not shown) had a response size of 35,655 bytes, or 44% of the size of the database response.</span></span>

<span data-ttu-id="4f5e3-212">Нагрузочный тест метода `AggregateOnClientAsync` показывает более предельные результаты.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-212">The load test for the `AggregateOnClientAsync` method shows more extreme results.</span></span> <span data-ttu-id="4f5e3-213">В этом случае в каждом тесте выполнялся запрос, который извлекал более 280 КБ данных из базы данных, однако размер запроса JSON составлял всего лишь 14 байт.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-213">In this case, each test performed a query that retrieved over 280Kb of data from the database, but the JSON response was a mere 14 bytes.</span></span> <span data-ttu-id="4f5e3-214">Такое сильное расхождение связано с тем, что метод рассчитывал агрегированный результат из большого объема данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-214">The wide disparity is because the method calculates an aggregated result from a large volume of data.</span></span>

![Данные телеметрии для метода AggregateOnClientAsync][TelemetryAggregateOnClient]

### <a name="identify-and-analyze-slow-queries"></a><span data-ttu-id="4f5e3-216">Определение и анализ медленных запросов</span><span class="sxs-lookup"><span data-stu-id="4f5e3-216">Identify and analyze slow queries</span></span>

<span data-ttu-id="4f5e3-217">Найдите запросы базы данных, которые используют больше всего ресурсов и имеют наибольшее время обработки.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-217">Look for database queries that consume the most resources and take the most time to execute.</span></span> <span data-ttu-id="4f5e3-218">Вы можете выполнить инструментирование, чтобы определить время начала и завершения многих операций базы данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-218">You can add instrumentation to find the start and completion times for many database operations.</span></span> <span data-ttu-id="4f5e3-219">Многие хранилища данных содержат подробные сведения о том, как выполняются и оптимизируются запросы.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-219">Many data stores also provide in-depth information on how queries are performed and optimized.</span></span> <span data-ttu-id="4f5e3-220">Например, панель производительности запросов на портале управления базы данных SQL Azure позволяет выбрать запрос и просмотреть подробные сведения о производительности среды выполнения.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-220">For example, the Query Performance pane in the Azure SQL Database management portal lets you select a query and view detailed runtime performance information.</span></span> <span data-ttu-id="4f5e3-221">Запрос, созданный с помощью операции `GetAllFieldsAsync`, выглядит так:</span><span class="sxs-lookup"><span data-stu-id="4f5e3-221">Here is the query generated by the `GetAllFieldsAsync` operation:</span></span>

![Панель сведений о запросе на портале управления базы данных SQL для Microsoft Azure][QueryDetails]


## <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="4f5e3-223">Реализация решения и проверка результатов</span><span class="sxs-lookup"><span data-stu-id="4f5e3-223">Implement the solution and verify the result</span></span>

<span data-ttu-id="4f5e3-224">Когда метод `GetRequiredFieldsAsync` будет использовать инструкцию SELECT на стороне базы данных, нагрузочное тестирование будет отображать следующие результаты.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-224">After changing the `GetRequiredFieldsAsync` method to use a SELECT statement on the database side, load testing showed the following results.</span></span>

![Результаты нагрузочного теста метода GetRequiredFieldsAsync][Load-Test-Results-Database-Side1]

<span data-ttu-id="4f5e3-226">Этот нагрузочный тест использовал то же развертывание и ту же имитированную рабочую нагрузку 400 параллельных пользователей, что и раньше.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-226">This load test used the same deployment and the same simulated workload of 400 concurrent users as before.</span></span> <span data-ttu-id="4f5e3-227">На графике показана гораздо меньшая задержка.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-227">The graph shows much lower latency.</span></span> <span data-ttu-id="4f5e3-228">Время отклика растет вместе с нагрузкой примерно до 1,3 секунд по сравнению с 4 секундами в описанном выше случае.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-228">Response time rises with load to approximately 1.3 seconds, compared to 4 seconds in the previous case.</span></span> <span data-ttu-id="4f5e3-229">Пропускная способность также является более высокой при 350 запросах в секунду по сравнению со 100, как ранее.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-229">The throughput is also higher at 350 requests per second compared to 100 earlier.</span></span> <span data-ttu-id="4f5e3-230">Объем данных, полученных из базы данных, теперь более точно соответствует размеру сообщений с ответом HTTP.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-230">The volume of data retrieved from the database now closely matches the size of the HTTP response messages.</span></span>

![Данные телеметрии для метода GetRequiredFieldsAsync][TelemetryRequiredFields]

<span data-ttu-id="4f5e3-232">При нагрузочном тестировании с помощью метода `AggregateOnDatabaseAsync` будут получены следующие результаты:</span><span class="sxs-lookup"><span data-stu-id="4f5e3-232">Load testing using the `AggregateOnDatabaseAsync` method generates the following results:</span></span>

![Результаты нагрузочного теста метода AggregateOnDatabaseAsync][Load-Test-Results-Database-Side2]

<span data-ttu-id="4f5e3-234">Среднее время отклика теперь минимальное.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-234">The average response time is now minimal.</span></span> <span data-ttu-id="4f5e3-235">Это последовательность интенсивного повышения производительности, которая в первую очередь вызвана огромным снижением операций ввода-вывода в базе данных.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-235">This is an order of magnitude improvement in performance, caused primarily by the large reduction in I/O from the database.</span></span> 

<span data-ttu-id="4f5e3-236">Ниже приведены соответствующие данные телеметрии для метода `AggregateOnDatabaseAsync`.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-236">Here is the corresponding telemetry for the `AggregateOnDatabaseAsync` method.</span></span> <span data-ttu-id="4f5e3-237">Объем данных, полученных из базы данных, был значительно снижен (с 280 КБ до 53 байт за одну транзакцию).</span><span class="sxs-lookup"><span data-stu-id="4f5e3-237">The amount of data retrieved from the database was vastly reduced, from over 280Kb per transaction to 53 bytes.</span></span> <span data-ttu-id="4f5e3-238">В результате максимальное количество непрерывных запросов в минуту было увеличено примерно с 2000 до 25 000.</span><span class="sxs-lookup"><span data-stu-id="4f5e3-238">As a result, the maximum sustained number of requests per minute was raised from around 2,000 to over 25,000.</span></span>

![Данные телеметрии для метода AggregateOnDatabaseAsync][TelemetryAggregateInDatabaseAsync]


## <a name="related-resources"></a><span data-ttu-id="4f5e3-240">Связанные ресурсы</span><span class="sxs-lookup"><span data-stu-id="4f5e3-240">Related resources</span></span>

- <span data-ttu-id="4f5e3-241">[Антишаблон занятости базы данных][BusyDatabase]</span><span class="sxs-lookup"><span data-stu-id="4f5e3-241">[Busy Database antipattern][BusyDatabase]</span></span>
- <span data-ttu-id="4f5e3-242">[Антишаблон отправки множественных операций ввода-вывода][chatty-io]</span><span class="sxs-lookup"><span data-stu-id="4f5e3-242">[Chatty I/O antipattern][chatty-io]</span></span>
- <span data-ttu-id="4f5e3-243">[Секционирование данных][data-partitioning]</span><span class="sxs-lookup"><span data-stu-id="4f5e3-243">[Data partitioning best practices][data-partitioning]</span></span>


[BusyDatabase]: ../busy-database/index.md
[data-partitioning]: ../../best-practices/data-partitioning.md
[new-relic]: https://newrelic.com/application-monitoring

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ExtraneousFetching

[chatty-io]: ../chatty-io/index.md
[MonolithicPersistence]: ../monolithic-persistence/index.md
[Load-Test-Results-Client-Side1]:_images/LoadTestResultsClientSide1.jpg
[Load-Test-Results-Client-Side2]:_images/LoadTestResultsClientSide2.jpg
[Load-Test-Results-Database-Side1]:_images/LoadTestResultsDatabaseSide1.jpg
[Load-Test-Results-Database-Side2]:_images/LoadTestResultsDatabaseSide2.jpg
[QueryPerformanceZoomed]: _images/QueryPerformanceZoomed.jpg
[QueryDetails]: _images/QueryDetails.jpg
[TelemetryAllFields]: _images/TelemetryAllFields.jpg
[TelemetryAggregateOnClient]: _images/TelemetryAggregateOnClient.jpg
[TelemetryRequiredFields]: _images/TelemetryRequiredFields.jpg
[TelemetryAggregateInDatabaseAsync]: _images/TelemetryAggregateInDatabase.jpg
