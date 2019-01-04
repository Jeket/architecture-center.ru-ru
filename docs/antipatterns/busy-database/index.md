---
title: Антишаблон занятости базы данных
titleSuffix: Performance antipatterns for cloud apps
description: Разгрузка обработки на сервер базы данных может привести к проблемам с производительностью и масштабируемостью.
author: dragon119
ms.date: 06/05/2017
ms.custom: seodec18
ms.openlocfilehash: 11bce03aed2e988d0a814b3298818715ba42c1c5
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54011469"
---
# <a name="busy-database-antipattern"></a><span data-ttu-id="06dc9-103">Антишаблон занятости базы данных</span><span class="sxs-lookup"><span data-stu-id="06dc9-103">Busy Database antipattern</span></span>

<span data-ttu-id="06dc9-104">В результате разгрузки обработки на сервер базы данных значительная часть времени будет потрачена на выполнение кода, а не на ответ на запросы для хранения и извлечения данных.</span><span class="sxs-lookup"><span data-stu-id="06dc9-104">Offloading processing to a database server can cause it to spend a significant proportion of time running code, rather than responding to requests to store and retrieve data.</span></span>

## <a name="problem-description"></a><span data-ttu-id="06dc9-105">Описание проблемы</span><span class="sxs-lookup"><span data-stu-id="06dc9-105">Problem description</span></span>

<span data-ttu-id="06dc9-106">Большинство систем баз данных могут выполнять код.</span><span class="sxs-lookup"><span data-stu-id="06dc9-106">Many database systems can run code.</span></span> <span data-ttu-id="06dc9-107">Примеры включают хранимые процедуры и триггеры.</span><span class="sxs-lookup"><span data-stu-id="06dc9-107">Examples include stored procedures and triggers.</span></span> <span data-ttu-id="06dc9-108">Зачастую эффективнее выполнять такую обработку рядом с данными, вместо того чтобы передавать данные в клиентское приложение для обработки.</span><span class="sxs-lookup"><span data-stu-id="06dc9-108">Often, it's more efficient to perform this processing close to the data, rather than transmitting the data to a client application for processing.</span></span> <span data-ttu-id="06dc9-109">Тем не менее чрезмерное использование этих функций может привести к снижению уровня производительности. Причины могут быть следующие:</span><span class="sxs-lookup"><span data-stu-id="06dc9-109">However, overusing these features can hurt performance, for several reasons:</span></span>

- <span data-ttu-id="06dc9-110">Сервер базы данных тратит слишком много времени для обработки, вместо того чтобы принимать новые запросы клиентов и извлекать данные.</span><span class="sxs-lookup"><span data-stu-id="06dc9-110">The database server may spend too much time processing, rather than accepting new client requests and fetching data.</span></span>
- <span data-ttu-id="06dc9-111">База данных представляет собой общий ресурс, поэтому она может стать узким местом во время периодов большой нагрузки.</span><span class="sxs-lookup"><span data-stu-id="06dc9-111">A database is usually a shared resource, so it can become a bottleneck during periods of high use.</span></span>
- <span data-ttu-id="06dc9-112">Затраты на среду выполнения могут быть чрезмерными, если использование хранилища данных тарифицируется по определенным единицам.</span><span class="sxs-lookup"><span data-stu-id="06dc9-112">Runtime costs may be excessive if the data store is metered.</span></span> <span data-ttu-id="06dc9-113">В частности это касается управляемых служб баз данных.</span><span class="sxs-lookup"><span data-stu-id="06dc9-113">That's particularly true of managed database services.</span></span> <span data-ttu-id="06dc9-114">Например, счета за использование базы данных SQL Azure выставляются за [единицу транзакций баз данных][dtu] (DTU).</span><span class="sxs-lookup"><span data-stu-id="06dc9-114">For example, Azure SQL Database charges for [Database Transaction Units][dtu] (DTUs).</span></span>
- <span data-ttu-id="06dc9-115">Базы данных имеют ограничение по масштабируемой емкости, поэтому выполнить горизонтальное масштабирование базы данных непросто.</span><span class="sxs-lookup"><span data-stu-id="06dc9-115">Databases have finite capacity to scale up, and it's not trivial to scale a database horizontally.</span></span> <span data-ttu-id="06dc9-116">Таким образом, лучше переместить обработку в вычислительный ресурс, например виртуальную машину или приложение службы приложений, которые можно легко развернуть.</span><span class="sxs-lookup"><span data-stu-id="06dc9-116">Therefore, it may be better to move processing into a compute resource, such as a VM or App Service app, that can easily scale out.</span></span>

<span data-ttu-id="06dc9-117">Этот антишаблон обычно возникает в следующих случаях:</span><span class="sxs-lookup"><span data-stu-id="06dc9-117">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="06dc9-118">Базы данных рассматриваются как службы, а не репозитории.</span><span class="sxs-lookup"><span data-stu-id="06dc9-118">The database is viewed as a service rather than a repository.</span></span> <span data-ttu-id="06dc9-119">Приложение может использовать сервер базы данных для форматирования данных (например, при преобразовании в XML), управления строковыми данными или выполнения сложных вычислений.</span><span class="sxs-lookup"><span data-stu-id="06dc9-119">An application might use the database server to format data (for example, converting to XML), manipulate string data, or perform complex calculations.</span></span>
- <span data-ttu-id="06dc9-120">Разработчики пытаются написать запросы, результаты которых могут отображаться непосредственно пользователям.</span><span class="sxs-lookup"><span data-stu-id="06dc9-120">Developers try to write queries whose results can be displayed directly to users.</span></span> <span data-ttu-id="06dc9-121">Например, запрос может объединять поля или форматировать даты, время и валюту в соответствии с языковым стандартом.</span><span class="sxs-lookup"><span data-stu-id="06dc9-121">For example a query might combine fields, or format dates, times, and currency according to locale.</span></span>
- <span data-ttu-id="06dc9-122">Разработчики пытаются устранить антишаблон [лишней выборки][ExtraneousFetching] с помощью принудительной отправки вычислений в базу данных.</span><span class="sxs-lookup"><span data-stu-id="06dc9-122">Developers are trying to correct the [Extraneous Fetching][ExtraneousFetching] antipattern by pushing computations to the database.</span></span>
- <span data-ttu-id="06dc9-123">Хранимые процедуры используются для инкапсуляции бизнес-логики, возможно, потому их проще обслуживать и обновлять.</span><span class="sxs-lookup"><span data-stu-id="06dc9-123">Stored procedures are used to encapsulate business logic, perhaps because they are considered easier to maintain and update.</span></span>

<span data-ttu-id="06dc9-124">В следующем примере показано извлечение 20 наиболее ценных заказов для указанной территории продаж, а также форматирование результатов в формате XML.</span><span class="sxs-lookup"><span data-stu-id="06dc9-124">The following example retrieves the 20 most valuable orders for a specified sales territory and formats the results as XML.</span></span>
<span data-ttu-id="06dc9-125">Анализ данных и преобразование результатов в XML выполняются с помощью функций Transact-SQL.</span><span class="sxs-lookup"><span data-stu-id="06dc9-125">It uses Transact-SQL functions to parse the data and convert the results to XML.</span></span> <span data-ttu-id="06dc9-126">Полный пример см. [здесь][sample-app].</span><span class="sxs-lookup"><span data-stu-id="06dc9-126">You can find the complete sample [here][sample-app].</span></span>

```SQL
SELECT TOP 20
  soh.[SalesOrderNumber]  AS '@OrderNumber',
  soh.[Status]            AS '@Status',
  soh.[ShipDate]          AS '@ShipDate',
  YEAR(soh.[OrderDate])   AS '@OrderDateYear',
  MONTH(soh.[OrderDate])  AS '@OrderDateMonth',
  soh.[DueDate]           AS '@DueDate',
  FORMAT(ROUND(soh.[SubTotal],2),'C')
                          AS '@SubTotal',
  FORMAT(ROUND(soh.[TaxAmt],2),'C')
                          AS '@TaxAmt',
  FORMAT(ROUND(soh.[TotalDue],2),'C')
                          AS '@TotalDue',
  CASE WHEN soh.[TotalDue] > 5000 THEN 'Y' ELSE 'N' END
                          AS '@ReviewRequired',
  (
  SELECT
    c.[AccountNumber]     AS '@AccountNumber',
    UPPER(LTRIM(RTRIM(REPLACE(
    CONCAT( p.[Title], ' ', p.[FirstName], ' ', p.[MiddleName], ' ', p.[LastName], ' ', p.[Suffix]),
    '  ', ' '))))         AS '@FullName'
  FROM [Sales].[Customer] c
    INNER JOIN [Person].[Person] p
  ON c.[PersonID] = p.[BusinessEntityID]
  WHERE c.[CustomerID] = soh.[CustomerID]
  FOR XML PATH ('Customer'), TYPE
  ),

  (
  SELECT
    sod.[OrderQty]      AS '@Quantity',
    FORMAT(sod.[UnitPrice],'C')
                        AS '@UnitPrice',
    FORMAT(ROUND(sod.[LineTotal],2),'C')
                        AS '@LineTotal',
    sod.[ProductID]     AS '@ProductId',
    CASE WHEN (sod.[ProductID] >= 710) AND (sod.[ProductID] <= 720) AND (sod.[OrderQty] >= 5) THEN 'Y' ELSE 'N' END
                        AS '@InventoryCheckRequired'

  FROM [Sales].[SalesOrderDetail] sod
  WHERE sod.[SalesOrderID] = soh.[SalesOrderID]
  ORDER BY sod.[SalesOrderDetailID]
  FOR XML PATH ('LineItem'), TYPE, ROOT('OrderLineItems')
  )

FROM [Sales].[SalesOrderHeader] soh
WHERE soh.[TerritoryId] = @TerritoryId
ORDER BY soh.[TotalDue] DESC
FOR XML PATH ('Order'), ROOT('Orders')
```

<span data-ttu-id="06dc9-127">Очевидно, что это сложный запрос.</span><span class="sxs-lookup"><span data-stu-id="06dc9-127">Clearly, this is complex query.</span></span> <span data-ttu-id="06dc9-128">Как будет показано далее, он использует значительные вычислительные ресурсы на сервере базы данных.</span><span class="sxs-lookup"><span data-stu-id="06dc9-128">As we'll see later, it turns out to use significant processing resources on the database server.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="06dc9-129">Как устранить проблему</span><span class="sxs-lookup"><span data-stu-id="06dc9-129">How to fix the problem</span></span>

<span data-ttu-id="06dc9-130">Переместите обработку с сервера базы данных на другие уровни приложения.</span><span class="sxs-lookup"><span data-stu-id="06dc9-130">Move processing from the database server into other application tiers.</span></span> <span data-ttu-id="06dc9-131">В идеале база данных должна выполнять только операции доступа к данным, используя функции, для которых она оптимизирована (например, агрегирование в системе управления реляционной базой данных).</span><span class="sxs-lookup"><span data-stu-id="06dc9-131">Ideally, you should limit the database to performing data access operations, using only the capabilities that the database is optimized for, such as aggregation in an RDBMS.</span></span>

<span data-ttu-id="06dc9-132">Например, вы можете заменить предыдущий код Transact-SQL оператором, который просто получает данные для обработки.</span><span class="sxs-lookup"><span data-stu-id="06dc9-132">For example, the previous Transact-SQL code can be replaced with a statement that simply retrieves the data to be processed.</span></span>

```SQL
SELECT
soh.[SalesOrderNumber]  AS [OrderNumber],
soh.[Status]            AS [Status],
soh.[OrderDate]         AS [OrderDate],
soh.[DueDate]           AS [DueDate],
soh.[ShipDate]          AS [ShipDate],
soh.[SubTotal]          AS [SubTotal],
soh.[TaxAmt]            AS [TaxAmt],
soh.[TotalDue]          AS [TotalDue],
c.[AccountNumber]       AS [AccountNumber],
p.[Title]               AS [CustomerTitle],
p.[FirstName]           AS [CustomerFirstName],
p.[MiddleName]          AS [CustomerMiddleName],
p.[LastName]            AS [CustomerLastName],
p.[Suffix]              AS [CustomerSuffix],
sod.[OrderQty]          AS [Quantity],
sod.[UnitPrice]         AS [UnitPrice],
sod.[LineTotal]         AS [LineTotal],
sod.[ProductID]         AS [ProductId]
FROM [Sales].[SalesOrderHeader] soh
INNER JOIN [Sales].[Customer] c ON soh.[CustomerID] = c.[CustomerID]
INNER JOIN [Person].[Person] p ON c.[PersonID] = p.[BusinessEntityID]
INNER JOIN [Sales].[SalesOrderDetail] sod ON soh.[SalesOrderID] = sod.[SalesOrderID]
WHERE soh.[TerritoryId] = @TerritoryId
AND soh.[SalesOrderId] IN (
    SELECT TOP 20 SalesOrderId
    FROM [Sales].[SalesOrderHeader] soh
    WHERE soh.[TerritoryId] = @TerritoryId
    ORDER BY soh.[TotalDue] DESC)
ORDER BY soh.[TotalDue] DESC, sod.[SalesOrderDetailID]
```

<span data-ttu-id="06dc9-133">Затем приложение использует API-интерфейсы `System.Xml.Linq` .NET Framework для форматирования результатов в XML.</span><span class="sxs-lookup"><span data-stu-id="06dc9-133">The application then uses the .NET Framework `System.Xml.Linq` APIs to format the results as XML.</span></span>

```csharp
// Create a new SqlCommand to run the Transact-SQL query
using (var command = new SqlCommand(...))
{
    command.Parameters.AddWithValue("@TerritoryId", id);

    // Run the query and create the initial XML document
    using (var reader = await command.ExecuteReaderAsync())
    {
        var lastOrderNumber = string.Empty;
        var doc = new XDocument();
        var orders = new XElement("Orders");
        doc.Add(orders);

        XElement lineItems = null;
        // Fetch each row in turn, format the results as XML, and add them to the XML document
        while (await reader.ReadAsync())
        {
            var orderNumber = reader["OrderNumber"].ToString();
            if (orderNumber != lastOrderNumber)
            {
                lastOrderNumber = orderNumber;

                var order = new XElement("Order");
                orders.Add(order);
                var customer = new XElement("Customer");
                lineItems = new XElement("OrderLineItems");
                order.Add(customer, lineItems);

                var orderDate = (DateTime)reader["OrderDate"];
                var totalDue = (Decimal)reader["TotalDue"];
                var reviewRequired = totalDue > 5000 ? 'Y' : 'N';

                order.Add(
                    new XAttribute("OrderNumber", orderNumber),
                    new XAttribute("Status", reader["Status"]),
                    new XAttribute("ShipDate", reader["ShipDate"]),
                    ... // More attributes, not shown.

                    var fullName = string.Join(" ",
                        reader["CustomerTitle"],
                        reader["CustomerFirstName"],
                        reader["CustomerMiddleName"],
                        reader["CustomerLastName"],
                        reader["CustomerSuffix"]
                    )
                   .Replace("  ", " ") //remove double spaces
                   .Trim()
                   .ToUpper();

               customer.Add(
                    new XAttribute("AccountNumber", reader["AccountNumber"]),
                    new XAttribute("FullName", fullName));
            }

            var productId = (int)reader["ProductID"];
            var quantity = (short)reader["Quantity"];
            var inventoryCheckRequired = (productId >= 710 && productId <= 720 && quantity >= 5) ? 'Y' : 'N';

            lineItems.Add(
                new XElement("LineItem",
                    new XAttribute("Quantity", quantity),
                    new XAttribute("UnitPrice", ((Decimal)reader["UnitPrice"]).ToString("C")),
                    new XAttribute("LineTotal", RoundAndFormat(reader["LineTotal"])),
                    new XAttribute("ProductId", productId),
                    new XAttribute("InventoryCheckRequired", inventoryCheckRequired)
                ));
        }
        // Match the exact formatting of the XML returned from SQL
        var xml = doc
            .ToString(SaveOptions.DisableFormatting)
            .Replace(" />", "/>");
    }
}
```

> [!NOTE]
> <span data-ttu-id="06dc9-134">Этот код является довольно сложным.</span><span class="sxs-lookup"><span data-stu-id="06dc9-134">This code is somewhat complex.</span></span> <span data-ttu-id="06dc9-135">Для нового приложения можно использовать библиотеку сериализации.</span><span class="sxs-lookup"><span data-stu-id="06dc9-135">For a new application, you might prefer to use a serialization library.</span></span> <span data-ttu-id="06dc9-136">Однако здесь предполагается, что команда разработчиков выполняет рефакторинг имеющегося приложения. Поэтому метод должен возвращать формат, который совпадает с исходным кодом.</span><span class="sxs-lookup"><span data-stu-id="06dc9-136">However, the assumption here is that the development team is refactoring an existing application, so the method needs to return the exact same format as the original code.</span></span>

## <a name="considerations"></a><span data-ttu-id="06dc9-137">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="06dc9-137">Considerations</span></span>

- <span data-ttu-id="06dc9-138">Многие системы баз данных оптимизированы для выполнения определенных типов обработки данных, например вычисления статистических значений для больших наборов данных.</span><span class="sxs-lookup"><span data-stu-id="06dc9-138">Many database systems are highly optimized to perform certain types of data processing, such as calculating aggregate values over large datasets.</span></span> <span data-ttu-id="06dc9-139">Не перемещайте эти типы обработки из базы данных.</span><span class="sxs-lookup"><span data-stu-id="06dc9-139">Don't move those types of processing out of the database.</span></span>

- <span data-ttu-id="06dc9-140">Не перемещайте обработку, если в результате база данных передает гораздо больше данных по сети.</span><span class="sxs-lookup"><span data-stu-id="06dc9-140">Do not relocate processing if doing so causes the database to transfer far more data over the network.</span></span> <span data-ttu-id="06dc9-141">Дополнительные сведения см. в статье [Антишаблон лишней выборки][ExtraneousFetching].</span><span class="sxs-lookup"><span data-stu-id="06dc9-141">See the [Extraneous Fetching antipattern][ExtraneousFetching].</span></span>

- <span data-ttu-id="06dc9-142">При перемещении обработки на уровень приложения может потребоваться его развернуть, чтобы он мог выполнить дополнительную работу.</span><span class="sxs-lookup"><span data-stu-id="06dc9-142">If you move processing to an application tier, that tier may need to scale out to handle the additional work.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="06dc9-143">Как определить проблему</span><span class="sxs-lookup"><span data-stu-id="06dc9-143">How to detect the problem</span></span>

<span data-ttu-id="06dc9-144">Признаки занятости базы данных включают непропорциональное отклонение в пропускной способности и времени отклика в операциях, которые обращаются к базе данных.</span><span class="sxs-lookup"><span data-stu-id="06dc9-144">Symptoms of a busy database include a disproportionate decline in throughput and response times in operations that access the database.</span></span>

<span data-ttu-id="06dc9-145">Чтобы определить эту проблему, сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="06dc9-145">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="06dc9-146">С помощью мониторинга производительности определите, сколько времени рабочая система тратит на выполнение операций с базой данных.</span><span class="sxs-lookup"><span data-stu-id="06dc9-146">Use performance monitoring to identify how much time the production system spends performing database activity.</span></span>

2. <span data-ttu-id="06dc9-147">Проверьте работу, выполняемую базой данных в эти периоды.</span><span class="sxs-lookup"><span data-stu-id="06dc9-147">Examine the work performed by the database during these periods.</span></span>

3. <span data-ttu-id="06dc9-148">Если есть подозрение, что конкретные операции могут вызывать слишком высокую активность базы данных, выполните нагрузочное тестирование в управляемой среде.</span><span class="sxs-lookup"><span data-stu-id="06dc9-148">If you suspect that particular operations might cause too much database activity, perform load testing in a controlled environment.</span></span> <span data-ttu-id="06dc9-149">Каждый тест должен выполнять сочетание подозрительных операций с переменной пользовательской нагрузкой.</span><span class="sxs-lookup"><span data-stu-id="06dc9-149">Each test should run a mixture of the suspect operations with a variable user load.</span></span> <span data-ttu-id="06dc9-150">Проанализируйте, как используется база данных, изучив данные телеметрии из нагрузочных тестов.</span><span class="sxs-lookup"><span data-stu-id="06dc9-150">Examine the telemetry from the load tests to observe how the database is used.</span></span>

4. <span data-ttu-id="06dc9-151">Если активность базы данных показывает высокий уровень обработки, но низкий уровень трафика данных, просмотрите исходный код, чтобы определить, можно ли выполнить обработку в другом месте.</span><span class="sxs-lookup"><span data-stu-id="06dc9-151">If the database activity reveals significant processing but little data traffic, review the source code to determine whether the processing can better be performed elsewhere.</span></span>

<span data-ttu-id="06dc9-152">Если объем активности базы данных низкий, а время отклика достаточно быстрое, занятая база данных вряд ли вызывает проблему с производительностью.</span><span class="sxs-lookup"><span data-stu-id="06dc9-152">If the volume of database activity is low or response times are relatively fast, then a busy database is unlikely to be a performance problem.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="06dc9-153">Пример диагностики</span><span class="sxs-lookup"><span data-stu-id="06dc9-153">Example diagnosis</span></span>

<span data-ttu-id="06dc9-154">В следующих разделах эти шаги применяются к примеру приложения, описанному ранее.</span><span class="sxs-lookup"><span data-stu-id="06dc9-154">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="monitor-the-volume-of-database-activity"></a><span data-ttu-id="06dc9-155">Мониторинг объема активности базы данных</span><span class="sxs-lookup"><span data-stu-id="06dc9-155">Monitor the volume of database activity</span></span>

<span data-ttu-id="06dc9-156">На следующем графике показаны результаты выполнения нагрузочного теста для примера приложения с пошаговой нагрузкой до 50 одновременных пользователей.</span><span class="sxs-lookup"><span data-stu-id="06dc9-156">The following graph shows the results of running a load test against the sample application, using a step load of up to 50 concurrent users.</span></span> <span data-ttu-id="06dc9-157">Объем запросов быстро достигает предела и остается на этом уровне, в то время как среднее время отклика постоянно увеличивается.</span><span class="sxs-lookup"><span data-stu-id="06dc9-157">The volume of requests quickly reaches a limit and stays at that level, while the average response time steadily increases.</span></span> <span data-ttu-id="06dc9-158">Обратите внимание, что эти показатели измеряются по логарифмической шкале.</span><span class="sxs-lookup"><span data-stu-id="06dc9-158">Note that a logarithmic scale is used for those two metrics.</span></span>

![Результаты нагрузочного теста для выполнения обработки в базе данных][ProcessingInDatabaseLoadTest]

<span data-ttu-id="06dc9-160">На следующем графике показано использование ЦП и единиц передачи данных в виде процента от квоты службы.</span><span class="sxs-lookup"><span data-stu-id="06dc9-160">The next graph shows CPU utilization and DTUs as a percentage of service quota.</span></span> <span data-ttu-id="06dc9-161">Единицы передачи данных — это показатель объема обработки, которую выполняет база данных.</span><span class="sxs-lookup"><span data-stu-id="06dc9-161">DTUs provides a measure of how much processing the database performs.</span></span> <span data-ttu-id="06dc9-162">На графике показано, что объем использования ЦП и единиц передачи данных быстро достиг 100 %.</span><span class="sxs-lookup"><span data-stu-id="06dc9-162">The graph shows that CPU and DTU utilization both quickly reached 100%.</span></span>

![Мониторинг базы данных SQL Azure: уровень производительности базы данных во время выполнения обработки][ProcessingInDatabaseMonitor]

### <a name="examine-the-work-performed-by-the-database"></a><span data-ttu-id="06dc9-164">Проверка работы, которую выполняет база данных</span><span class="sxs-lookup"><span data-stu-id="06dc9-164">Examine the work performed by the database</span></span>

<span data-ttu-id="06dc9-165">Задачи, выполняемые в базе данных, могут быть операциями доступа к данным, а не обработкой. Поэтому очень важно ознакомиться с инструкциями SQL, которые выполняются во время работы базы данных.</span><span class="sxs-lookup"><span data-stu-id="06dc9-165">It could be that the tasks performed by the database are genuine data access operations, rather than processing, so it is important to understand the SQL statements being run while the database is busy.</span></span> <span data-ttu-id="06dc9-166">Выполните мониторинг системы, чтобы зафиксировать трафик SQL и сопоставить операции SQL с запросами приложения.</span><span class="sxs-lookup"><span data-stu-id="06dc9-166">Monitor the system to capture the SQL traffic and correlate the SQL operations with application requests.</span></span>

<span data-ttu-id="06dc9-167">Если операции базы данных являются операциями доступа к данным без большого объема обработки, то проблемой может быть [лишняя выборка][ExtraneousFetching].</span><span class="sxs-lookup"><span data-stu-id="06dc9-167">If the database operations are purely data access operations, without a lot of processing, then the problem might be [Extraneous Fetching][ExtraneousFetching].</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="06dc9-168">Реализация решения и проверка результатов</span><span class="sxs-lookup"><span data-stu-id="06dc9-168">Implement the solution and verify the result</span></span>

<span data-ttu-id="06dc9-169">На следующем графике показано выполнение нагрузочного теста с помощью обновленного кода.</span><span class="sxs-lookup"><span data-stu-id="06dc9-169">The following graph shows a load test using the updated code.</span></span> <span data-ttu-id="06dc9-170">Пропускная способность значительно выше — 400 запросов в секунду (по сравнению с 12 запросами до этого).</span><span class="sxs-lookup"><span data-stu-id="06dc9-170">Throughput is significantly higher, over 400 requests per second versus 12 earlier.</span></span> <span data-ttu-id="06dc9-171">Среднее время отклика также намного ниже — примерно 0,1 с (по сравнению с 4 с).</span><span class="sxs-lookup"><span data-stu-id="06dc9-171">The average response time is also much lower, just above 0.1 seconds compared to over 4 seconds.</span></span>

![Результаты нагрузочного теста для выполнения обработки в базе данных][ProcessingInClientApplicationLoadTest]

<span data-ttu-id="06dc9-173">Уровень использования ресурсов ЦП и единиц передачи данных показывает, что системе понадобилось больше времени для переполнения, несмотря на увеличенную пропускную способность.</span><span class="sxs-lookup"><span data-stu-id="06dc9-173">CPU and DTU utilization shows that the system took longer to reach saturation, despite the increased throughput.</span></span>

![Мониторинг базы данных SQL Azure: уровень производительности базы данных во время выполнения обработки в клиентском приложении][ProcessingInClientApplicationMonitor]

## <a name="related-resources"></a><span data-ttu-id="06dc9-175">Связанные ресурсы</span><span class="sxs-lookup"><span data-stu-id="06dc9-175">Related resources</span></span>

- <span data-ttu-id="06dc9-176">[Антишаблон лишней выборки][ExtraneousFetching]</span><span class="sxs-lookup"><span data-stu-id="06dc9-176">[Extraneous Fetching antipattern][ExtraneousFetching]</span></span>

[dtu]: /azure/sql-database/sql-database-service-tiers-dtu
[ExtraneousFetching]: ../extraneous-fetching/index.md
[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/BusyDatabase

[ProcessingInDatabaseLoadTest]: ./_images/ProcessingInDatabaseLoadTest.jpg
[ProcessingInClientApplicationLoadTest]: ./_images/ProcessingInClientApplicationLoadTest.jpg
[ProcessingInDatabaseMonitor]: ./_images/ProcessingInDatabaseMonitor.jpg
[ProcessingInClientApplicationMonitor]: ./_images/ProcessingInClientApplicationMonitor.jpg
