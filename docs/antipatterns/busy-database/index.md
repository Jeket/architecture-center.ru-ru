---
title: Антишаблон занятости базы данных
description: Разгрузка обработки на сервер базы данных может привести к проблемам с производительностью и масштабируемостью.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 9fdbde0731a1be570ef611894a9d23a1be87f4e7
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24538798"
---
# <a name="busy-database-antipattern"></a>Антишаблон занятости базы данных

В результате разгрузки обработки на сервер базы данных значительная часть времени будет потрачена на выполнение кода, а не на ответ на запросы для хранения и извлечения данных. 

## <a name="problem-description"></a>Описание проблемы

Большинство систем баз данных могут выполнять код. Примеры включают хранимые процедуры и триггеры. Зачастую эффективнее выполнять такую обработку рядом с данными, вместо того чтобы передавать данные в клиентское приложение для обработки. Тем не менее чрезмерное использование этих функций может привести к снижению уровня производительности. Причины могут быть следующие:

- Сервер базы данных тратит слишком много времени для обработки, вместо того чтобы принимать новые запросы клиентов и извлекать данные.
- База данных представляет собой общий ресурс, поэтому она может стать узким местом во время периодов большой нагрузки.
- Затраты на среду выполнения могут быть чрезмерными, если использование хранилища данных тарифицируется по определенным единицам. В частности это касается управляемых служб баз данных. Например, счета за использование базы данных SQL Azure выставляются за [единицу транзакций баз данных][dtu] (DTU).
- Базы данных имеют ограничение по масштабируемой емкости, поэтому выполнить горизонтальное масштабирование базы данных непросто. Таким образом, лучше переместить обработку в вычислительный ресурс, например виртуальную машину или приложение службы приложений, которые можно легко развернуть.

Этот антишаблон обычно возникает в следующих случаях:

- Базы данных рассматриваются как службы, а не репозитории. Приложение может использовать сервер базы данных для форматирования данных (например, при преобразовании в XML), управления строковыми данными или выполнения сложных вычислений.
- Разработчики пытаются написать запросы, результаты которых могут отображаться непосредственно пользователям. Например, запрос может объединять поля или форматировать даты, время и валюту в соответствии с языковым стандартом.
- Разработчики пытаются устранить антишаблон [лишней выборки][ExtraneousFetching] с помощью принудительной отправки вычислений в базу данных.
- Хранимые процедуры используются для инкапсуляции бизнес-логики, возможно, потому их проще обслуживать и обновлять.

В следующем примере показано извлечение 20 наиболее ценных заказов для указанной территории продаж, а также форматирование результатов в формате XML.
Анализ данных и преобразование результатов в XML выполняются с помощью функций Transact-SQL. Полный пример см. [здесь][sample-app].

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

Очевидно, что это сложный запрос. Как будет показано далее, он использует значительные вычислительные ресурсы на сервере базы данных.

## <a name="how-to-fix-the-problem"></a>Как устранить проблему

Переместите обработку с сервера базы данных на другие уровни приложения. В идеале база данных должна выполнять только операции доступа к данным, используя функции, для которых она оптимизирована (например, агрегирование в системе управления реляционной базой данных).

Например, вы можете заменить предыдущий код Transact-SQL оператором, который просто получает данные для обработки.

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

Затем приложение использует API-интерфейсы `System.Xml.Linq` .NET Framework для форматирования результатов в XML.

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
> Этот код является довольно сложным. Для нового приложения можно использовать библиотеку сериализации. Однако здесь предполагается, что команда разработчиков выполняет рефакторинг имеющегося приложения. Поэтому метод должен возвращать формат, который совпадает с исходным кодом.

## <a name="considerations"></a>Рекомендации

- Многие системы баз данных оптимизированы для выполнения определенных типов обработки данных, например вычисления статистических значений для больших наборов данных. Не перемещайте эти типы обработки из базы данных.

- Не перемещайте обработку, если в результате база данных передает гораздо больше данных по сети. Дополнительные сведения см. в статье [Антишаблон лишней выборки][ExtraneousFetching].

- При перемещении обработки на уровень приложения может потребоваться его развернуть, чтобы он мог выполнить дополнительную работу.

## <a name="how-to-detect-the-problem"></a>Как определить проблему

Признаки занятости базы данных включают непропорциональное отклонение в пропускной способности и времени отклика в операциях, которые обращаются к базе данных. 

Чтобы определить эту проблему, сделайте следующее: 

1. С помощью мониторинга производительности определите, сколько времени рабочая система тратит на выполнение операций с базой данных.

2. Проверьте работу, выполняемую базой данных в эти периоды.

3. Если есть подозрение, что конкретные операции могут вызывать слишком высокую активность базы данных, выполните нагрузочное тестирование в управляемой среде. Каждый тест должен выполнять сочетание подозрительных операций с переменной пользовательской нагрузкой. Проанализируйте, как используется база данных, изучив данные телеметрии из нагрузочных тестов.

4. Если активность базы данных показывает высокий уровень обработки, но низкий уровень трафика данных, просмотрите исходный код, чтобы определить, можно ли выполнить обработку в другом месте.

Если объем активности базы данных низкий, а время отклика достаточно быстрое, занятая база данных вряд ли вызывает проблему с производительностью.

## <a name="example-diagnosis"></a>Пример диагностики

В следующих разделах эти шаги применяются к примеру приложения, описанному ранее.

### <a name="monitor-the-volume-of-database-activity"></a>Мониторинг объема активности базы данных

На следующем графике показаны результаты выполнения нагрузочного теста для примера приложения с пошаговой нагрузкой до 50 одновременных пользователей. Объем запросов быстро достигает предела и остается на этом уровне, в то время как среднее время отклика постоянно увеличивается. Обратите внимание, что эти показатели измеряются по логарифмической шкале.

![Результаты нагрузочного теста для выполнения обработки в базе данных][ProcessingInDatabaseLoadTest]

На следующем графике показано использование ЦП и единиц передачи данных в виде процента от квоты службы. Единицы передачи данных — это показатель объема обработки, которую выполняет база данных. На графике показано, что объем использования ЦП и единиц передачи данных быстро достиг 100 %.

![Мониторинг базы данных SQL Azure: уровень производительности базы данных во время выполнения обработки][ProcessingInDatabaseMonitor]

### <a name="examine-the-work-performed-by-the-database"></a>Проверка работы, которую выполняет база данных

Задачи, выполняемые в базе данных, могут быть операциями доступа к данным, а не обработкой. Поэтому очень важно ознакомиться с инструкциями SQL, которые выполняются во время работы базы данных. Выполните мониторинг системы, чтобы зафиксировать трафик SQL и сопоставить операции SQL с запросами приложения.

Если операции базы данных являются операциями доступа к данным без большого объема обработки, то проблемой может быть [лишняя выборка][ExtraneousFetching].

### <a name="implement-the-solution-and-verify-the-result"></a>Реализация решения и проверка результатов

На следующем графике показано выполнение нагрузочного теста с помощью обновленного кода. Пропускная способность значительно выше — 400 запросов в секунду (по сравнению с 12 запросами до этого). Среднее время отклика также намного ниже — примерно 0,1 с (по сравнению с 4 с).

![Результаты нагрузочного теста для выполнения обработки в базе данных][ProcessingInClientApplicationLoadTest]

Уровень использования ресурсов ЦП и единиц передачи данных показывает, что системе понадобилось больше времени для переполнения, несмотря на увеличенную пропускную способность.

![Мониторинг базы данных SQL Azure: уровень производительности базы данных во время выполнения обработки в клиентском приложении][ProcessingInClientApplicationMonitor]

## <a name="related-resources"></a>Связанные ресурсы 

- [Антишаблон лишней выборки][ExtraneousFetching]


[dtu]: /sql-database/sql-database-what-is-a-dtu
[ExtraneousFetching]: ../extraneous-fetching/index.md
[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/BusyDatabase

[ProcessingInDatabaseLoadTest]: ./_images/ProcessingInDatabaseLoadTest.jpg
[ProcessingInClientApplicationLoadTest]: ./_images/ProcessingInClientApplicationLoadTest.jpg
[ProcessingInDatabaseMonitor]: ./_images/ProcessingInDatabaseMonitor.jpg
[ProcessingInClientApplicationMonitor]: ./_images/ProcessingInClientApplicationMonitor.jpg
