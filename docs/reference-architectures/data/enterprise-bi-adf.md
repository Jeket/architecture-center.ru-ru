---
title: Автоматизированная корпоративная бизнес-аналитика с использованием Хранилища данных SQL и Фабрики данных Azure
description: Сведения о том, как автоматизировать рабочий процесс ELT в Azure с помощью Фабрики данных Azure
author: MikeWasson
ms.date: 07/01/2018
ms.openlocfilehash: f004c02da93335e74b07b9720236832ad7f744db
ms.sourcegitcommit: 62945777e519d650159f0f963a2489b6bb6ce094
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/09/2018
ms.locfileid: "48876908"
---
# <a name="automated-enterprise-bi-with-sql-data-warehouse-and-azure-data-factory"></a><span data-ttu-id="228e4-103">Автоматизированная корпоративная бизнес-аналитика с использованием Хранилища данных SQL и Фабрики данных Azure</span><span class="sxs-lookup"><span data-stu-id="228e4-103">Automated enterprise BI with SQL Data Warehouse and Azure Data Factory</span></span>

<span data-ttu-id="228e4-104">На примере этой эталонной архитектуры показано, как выполнять добавочную нагрузку в конвейере [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (извлечение, загрузка и преобразование).</span><span class="sxs-lookup"><span data-stu-id="228e4-104">This reference architecture shows how to perform incremental loading in an [ELT](../../data-guide/relational-data/etl.md#extract-load-and-transform-elt) (extract-load-transform) pipeline.</span></span> <span data-ttu-id="228e4-105">Для автоматизации этого конвейера используется Фабрика данных Azure.</span><span class="sxs-lookup"><span data-stu-id="228e4-105">It uses Azure Data Factory to automate the ELT pipeline.</span></span> <span data-ttu-id="228e4-106">Конвейер поэтапно перемещает последние данные OLTP из локальной базы данных SQL Server в Хранилище данных SQL.</span><span class="sxs-lookup"><span data-stu-id="228e4-106">The pipeline incrementally moves the latest OLTP data from an on-premises SQL Server database into SQL Data Warehouse.</span></span> <span data-ttu-id="228e4-107">Данные о транзакциях преобразуются в табличную модель для анализа.</span><span class="sxs-lookup"><span data-stu-id="228e4-107">Transactional data is transformed into a tabular model for analysis.</span></span> [<span data-ttu-id="228e4-108">**Разверните это решение**.</span><span class="sxs-lookup"><span data-stu-id="228e4-108">**Deploy this solution**.</span></span>](#deploy-the-solution)

![](./images/enterprise-bi-sqldw-adf.png)

<span data-ttu-id="228e4-109">Эта архитектура создана на основе архитектуры, описанной в статье [Корпоративная бизнес-аналитика с использованием хранилища данных SQL](./enterprise-bi-sqldw.md), но с некоторыми дополнительными функциями, требуемыми для хранения корпоративных данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-109">This architecture builds on the one shown in [Enterprise BI with SQL Data Warehouse](./enterprise-bi-sqldw.md), but adds some features that are important for enterprise data warehousing scenarios.</span></span>

-   <span data-ttu-id="228e4-110">Автоматизация конвейера с помощью Фабрики данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-110">Automation of the pipeline using Data Factory.</span></span>
-   <span data-ttu-id="228e4-111">Добавочная загрузка.</span><span class="sxs-lookup"><span data-stu-id="228e4-111">Incremental loading.</span></span>
-   <span data-ttu-id="228e4-112">Интеграция нескольких источников данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-112">Integrating multiple data sources.</span></span>
-   <span data-ttu-id="228e4-113">Загрузка таких двоичных данных, как геопространственные данные и изображения.</span><span class="sxs-lookup"><span data-stu-id="228e4-113">Loading binary data such as geospatial data and images.</span></span>

## <a name="architecture"></a><span data-ttu-id="228e4-114">Архитектура</span><span class="sxs-lookup"><span data-stu-id="228e4-114">Architecture</span></span>

<span data-ttu-id="228e4-115">Архитектура состоит из следующих компонентов:</span><span class="sxs-lookup"><span data-stu-id="228e4-115">The architecture consists of the following components.</span></span>

### <a name="data-sources"></a><span data-ttu-id="228e4-116">Источники данных</span><span class="sxs-lookup"><span data-stu-id="228e4-116">Data sources</span></span>

<span data-ttu-id="228e4-117">**Локальный сервер SQL Server**.</span><span class="sxs-lookup"><span data-stu-id="228e4-117">**On-premises SQL Server**.</span></span> <span data-ttu-id="228e4-118">Исходные данные размещаются локально в базе данных SQL Server.</span><span class="sxs-lookup"><span data-stu-id="228e4-118">The source data is located in a SQL Server database on premises.</span></span> <span data-ttu-id="228e4-119">Чтобы имитировать локальную среду, сценарии развертывания для этой архитектуры предоставляют виртуальную машину в Azure с установленным SQL Server.</span><span class="sxs-lookup"><span data-stu-id="228e4-119">To simulate the on-premises environment, the deployment scripts for this architecture provision a virtual machine in Azure with SQL Server installed.</span></span> <span data-ttu-id="228e4-120">В качестве базы данных-источника используется [пример базы данных OLTP Wide World Importers][WWI].</span><span class="sxs-lookup"><span data-stu-id="228e4-120">The [Wide World Importers OLTP sample database][wwi] is used as the source database.</span></span>

<span data-ttu-id="228e4-121">**Внешние данные**.</span><span class="sxs-lookup"><span data-stu-id="228e4-121">**External data**.</span></span> <span data-ttu-id="228e4-122">Распространенный сценарий для хранилищ данных — выполнение интеграции нескольких источников данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-122">A common scenario for data warehouses is to integrate multiple data sources.</span></span> <span data-ttu-id="228e4-123">Для этой эталонной архитектуры загружается набор внешних данных о численности населения города по годам, интегрируемый с данными из базы данных OLTP.</span><span class="sxs-lookup"><span data-stu-id="228e4-123">This reference architecture loads an external data set that contains city populations by year, and integrates it with the data from the OLTP database.</span></span> <span data-ttu-id="228e4-124">Эта данные можно использовать для получения полезных сведений. Например, чтобы узнать, соответствуют ли показатели роста продаж в каждом регионе показателям роста населения или превышают их.</span><span class="sxs-lookup"><span data-stu-id="228e4-124">You can use this data for insights such as: "Does sales growth in each region match or exceed population growth?"</span></span>

### <a name="ingestion-and-data-storage"></a><span data-ttu-id="228e4-125">Прием и хранение данных</span><span class="sxs-lookup"><span data-stu-id="228e4-125">Ingestion and data storage</span></span>

<span data-ttu-id="228e4-126">**Хранилище больших двоичных объектов**.</span><span class="sxs-lookup"><span data-stu-id="228e4-126">**Blob Storage**.</span></span> <span data-ttu-id="228e4-127">Хранилище больших двоичных объектов используется в качестве промежуточной области для исходных данных перед их загрузкой в Хранилище данных SQL.</span><span class="sxs-lookup"><span data-stu-id="228e4-127">Blob storage is used as a staging area for the source data before loading it into SQL Data Warehouse.</span></span>

<span data-ttu-id="228e4-128">**Хранилище данных Azure SQL.**</span><span class="sxs-lookup"><span data-stu-id="228e4-128">**Azure SQL Data Warehouse**.</span></span> <span data-ttu-id="228e4-129">[Хранилище данных SQL](/azure/sql-data-warehouse/) — распределенная система, предназначенная для анализа больших объемов данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-129">[SQL Data Warehouse](/azure/sql-data-warehouse/) is a distributed system designed to perform analytics on large data.</span></span> <span data-ttu-id="228e4-130">Она поддерживает массовую параллельную обработку (MPP), что делает ее пригодной для запуска высокопроизводительной аналитики.</span><span class="sxs-lookup"><span data-stu-id="228e4-130">It supports massive parallel processing (MPP), which makes it suitable for running high-performance analytics.</span></span> 

<span data-ttu-id="228e4-131">**Фабрика данных Azure**.</span><span class="sxs-lookup"><span data-stu-id="228e4-131">**Azure Data Factory**.</span></span> <span data-ttu-id="228e4-132">[Фабрика данных][ADF] — это управляемая служба, которая координирует и автоматизирует перемещение и преобразование данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-132">[Data Factory][adf] is a managed service that orchestrates and automates data movement and data transformation.</span></span> <span data-ttu-id="228e4-133">В этой архитектуре она координирует разные этапы процесса ELT.</span><span class="sxs-lookup"><span data-stu-id="228e4-133">In this architecture, it coordinates the various stages of the ELT process.</span></span>

### <a name="analysis-and-reporting"></a><span data-ttu-id="228e4-134">Анализ и создание отчетов</span><span class="sxs-lookup"><span data-stu-id="228e4-134">Analysis and reporting</span></span>

<span data-ttu-id="228e4-135">**Службы Azure Analysis Services**.</span><span class="sxs-lookup"><span data-stu-id="228e4-135">**Azure Analysis Services**.</span></span> <span data-ttu-id="228e4-136">[Analysis Services](/azure/analysis-services/) — полностью управляемая служба, которая предоставляет возможности моделирования данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-136">[Analysis Services](/azure/analysis-services/) is a fully managed service that provides data modeling capabilities.</span></span> <span data-ttu-id="228e4-137">Семантическая модель загружается в службы Analysis Services.</span><span class="sxs-lookup"><span data-stu-id="228e4-137">The semantic model is loaded into Analysis Services.</span></span>

<span data-ttu-id="228e4-138">**Power BI**.</span><span class="sxs-lookup"><span data-stu-id="228e4-138">**Power BI**.</span></span> <span data-ttu-id="228e4-139">Power BI — набор средств бизнес-аналитики для анализа информации о бизнесе.</span><span class="sxs-lookup"><span data-stu-id="228e4-139">Power BI is a suite of business analytics tools to analyze data for business insights.</span></span> <span data-ttu-id="228e4-140">В этой архитектуре он запрашивает семантическую модель, хранящуюся в службе Analysis Services.</span><span class="sxs-lookup"><span data-stu-id="228e4-140">In this architecture, it queries the semantic model stored in Analysis Services.</span></span>

### <a name="authentication"></a><span data-ttu-id="228e4-141">Authentication</span><span class="sxs-lookup"><span data-stu-id="228e4-141">Authentication</span></span>

<span data-ttu-id="228e4-142">**Azure Active Directory** (Azure AD) аутентифицирует пользователей, которые подключаются к серверу Analysis Services через Power BI.</span><span class="sxs-lookup"><span data-stu-id="228e4-142">**Azure Active Directory** (Azure AD) authenticates users who connect to the Analysis Services server through Power BI.</span></span>

<span data-ttu-id="228e4-143">В Фабрике данных также можно использовать Azure AD для аутентификации в хранилище данных SQL с использованием субъекта-службы или Управляемого удостоверения службы (MSI).</span><span class="sxs-lookup"><span data-stu-id="228e4-143">Data Factory can use also use Azure AD to authenticate to SQL Data Warehouse, by using a service principal or Managed Service Identity (MSI).</span></span> <span data-ttu-id="228e4-144">Для простоты в примере развертывания используется аутентификация SQL Server.</span><span class="sxs-lookup"><span data-stu-id="228e4-144">For simplicity, the example deployment uses SQL Server authentication.</span></span>

## <a name="data-pipeline"></a><span data-ttu-id="228e4-145">Конвейер данных</span><span class="sxs-lookup"><span data-stu-id="228e4-145">Data pipeline</span></span>

<span data-ttu-id="228e4-146">Конвейер в [Фабрике данных Azure][ADF] — это логическая группа действий, используемых для координации задачи &mdash; в нашем примере с загрузкой и преобразованием данных в Хранилище данных SQL.</span><span class="sxs-lookup"><span data-stu-id="228e4-146">In [Azure Data Factory][adf], a pipeline is a logical grouping of activities used to coordinate a task &mdash; in this case, loading and transforming data into SQL Data Warehouse.</span></span> 

<span data-ttu-id="228e4-147">В этой эталонной архитектуре определяется основной конвейер, который запускает последовательность дочерних конвейеров.</span><span class="sxs-lookup"><span data-stu-id="228e4-147">This reference architecture defines a master pipeline that runs a sequence of child pipelines.</span></span> <span data-ttu-id="228e4-148">Каждый дочерний конвейер загружает данные в одну или несколько таблиц в хранилище данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-148">Each child pipeline loads data into one or more data warehouse tables.</span></span>

![](./images/adf-pipeline.png)

## <a name="incremental-loading"></a><span data-ttu-id="228e4-149">Добавочная загрузка</span><span class="sxs-lookup"><span data-stu-id="228e4-149">Incremental loading</span></span>

<span data-ttu-id="228e4-150">При выполнении автоматизированного процесса ETL или ELT гораздо эффективнее загружать только те данные, которые изменились с момента предыдущего выполнения.</span><span class="sxs-lookup"><span data-stu-id="228e4-150">When you run an automated ETL or ELT process, it's most efficient to load only the data that changed since the previous run.</span></span> <span data-ttu-id="228e4-151">Это называется *добавочной загрузкой*. Этот процесс отличается от полной загрузки, при которой загружаются все данные.</span><span class="sxs-lookup"><span data-stu-id="228e4-151">This is called an *incremental load*, as opposed to a full load that loads all of the data.</span></span> <span data-ttu-id="228e4-152">Чтобы выполнить добавочную загрузку, нужно выбрать метод определения измененных данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-152">To perform an incremental load, you need a way to identify which data has changed.</span></span> <span data-ttu-id="228e4-153">Для этого чаще всего используется значение *верхнего предела*. То есть отслеживается последнее значение в некотором столбце исходной таблицы, например по столбцу даты и времени или по столбцу уникальных целых чисел.</span><span class="sxs-lookup"><span data-stu-id="228e4-153">The most common approach is to use a *high water mark* value, which means tracking the latest value of some column in the source table, either a datetime column or a unique integer column.</span></span> 

<span data-ttu-id="228e4-154">Начиная с версии SQL Server 2016, вы можете использовать [темпоральные таблицы](/sql/relational-databases/tables/temporal-tables).</span><span class="sxs-lookup"><span data-stu-id="228e4-154">Starting with SQL Server 2016, you can use [temporal tables](/sql/relational-databases/tables/temporal-tables).</span></span> <span data-ttu-id="228e4-155">Это таблицы с системным управлением версиями, в которых хранится полный журнал изменения данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-155">These are system-versioned tables that keep a full history of data changes.</span></span> <span data-ttu-id="228e4-156">Ядро СУБД автоматически записывает каждое изменение в отдельную таблицу журнала.</span><span class="sxs-lookup"><span data-stu-id="228e4-156">The database engine automatically records the history of every change in a separate history table.</span></span> <span data-ttu-id="228e4-157">Чтобы запросить данные журнала, добавьте в запрос предложение FOR SYSTEM_TIME.</span><span class="sxs-lookup"><span data-stu-id="228e4-157">You can query the historical data by adding a FOR SYSTEM_TIME clause to a query.</span></span> <span data-ttu-id="228e4-158">На внутреннем уровне ядро СУБД отправляет запрос к таблице журнала, но это происходит незаметно для приложения.</span><span class="sxs-lookup"><span data-stu-id="228e4-158">Internally, the database engine queries the history table, but this is transparent to the application.</span></span> 

> [!NOTE]
> <span data-ttu-id="228e4-159">Для более ранних версий SQL Server можно использовать функцию [отслеживания измененных данных](/sql/relational-databases/track-changes/about-change-data-capture-sql-server) (CDC).</span><span class="sxs-lookup"><span data-stu-id="228e4-159">For earlier versions of SQL Server, you can use [Change Data Capture](/sql/relational-databases/track-changes/about-change-data-capture-sql-server) (CDC).</span></span> <span data-ttu-id="228e4-160">Этот метод не такой удобный, как темпоральные таблицы, так как вам нужно запросить отдельную таблицу изменений, и изменения отслеживаются по регистрационному номеру транзакции (LSN) в журнале, а не по метке времени.</span><span class="sxs-lookup"><span data-stu-id="228e4-160">This approach is less convenient than temporal tables, because you have to query a separate change table, and changes are tracked by a log sequence number, rather than a timestamp.</span></span> 

<span data-ttu-id="228e4-161">Темпоральные таблицы удобно использовать для данных измерений, которые могут изменяться со временем.</span><span class="sxs-lookup"><span data-stu-id="228e4-161">Temporal tables are useful for dimension data, which can change over time.</span></span> <span data-ttu-id="228e4-162">В таблице фактов обычно представлены неизменяемые транзакции, например при продаже. В этом случае ведение журнала версий системы не имеет смысла.</span><span class="sxs-lookup"><span data-stu-id="228e4-162">Fact tables usually represent an immutable transaction such as a sale, in which case keeping the system version history doesn't make sense.</span></span> <span data-ttu-id="228e4-163">Вместо этого для транзакций обычно присутствует столбец, в котором представлена дата транзакции, что может использоваться в качестве значения верхнего предела.</span><span class="sxs-lookup"><span data-stu-id="228e4-163">Instead, transactions usually have a column that represents the transaction date, which can be used as the watermark value.</span></span> <span data-ttu-id="228e4-164">Например, в базе данных OLTP Wide World Importers таблицы Sales.Invoices и Sales.InvoiceLines имеют поле `LastEditedWhen` со значением по умолчанию `sysdatetime()`.</span><span class="sxs-lookup"><span data-stu-id="228e4-164">For example, in the Wide World Importers OLTP database, the Sales.Invoices and Sales.InvoiceLines tables have a `LastEditedWhen` field that defaults to `sysdatetime()`.</span></span> 

<span data-ttu-id="228e4-165">Ниже приведена стандартная последовательность действий для конвейера ELT:</span><span class="sxs-lookup"><span data-stu-id="228e4-165">Here is the general flow for the ELT pipeline:</span></span>

1. <span data-ttu-id="228e4-166">Для каждой таблицы базы данных-источника отслеживается пороговое значение времени, когда запускается последнее задание ELT.</span><span class="sxs-lookup"><span data-stu-id="228e4-166">For each table in the source database, track the cutoff time when the last ELT job ran.</span></span> <span data-ttu-id="228e4-167">Сохраните эту информацию в хранилище данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-167">Store this information in the data warehouse.</span></span> <span data-ttu-id="228e4-168">(При начальной настройке для всех значений времени указано 1-1-1900.)</span><span class="sxs-lookup"><span data-stu-id="228e4-168">(On initial setup, all times are set to '1-1-1900'.)</span></span>

2. <span data-ttu-id="228e4-169">На этапе экспорта данных пороговое значение времени передается в качестве параметра в набор хранимых процедур в базе данных-источника.</span><span class="sxs-lookup"><span data-stu-id="228e4-169">During the data export step, the cutoff time is passed as a parameter to a set of stored procedures in the source database.</span></span> <span data-ttu-id="228e4-170">Эти хранимые процедуры запрашивают любые записи, которые были изменены или созданы после этого значения времени.</span><span class="sxs-lookup"><span data-stu-id="228e4-170">These stored procedures query for any records that were changed or created after the cutoff time.</span></span> <span data-ttu-id="228e4-171">Для таблицы фактов Sales используется столбец `LastEditedWhen`.</span><span class="sxs-lookup"><span data-stu-id="228e4-171">For the Sales fact table, the `LastEditedWhen` column is used.</span></span> <span data-ttu-id="228e4-172">Для данных измерения используются темпоральные таблицы с системным управлением версиями.</span><span class="sxs-lookup"><span data-stu-id="228e4-172">For the dimension data, system-versioned temporal tables are used.</span></span>

3. <span data-ttu-id="228e4-173">Когда перенос данных завершится, обновите таблицу, в которой хранятся пороговые значения времени.</span><span class="sxs-lookup"><span data-stu-id="228e4-173">When the data migration is complete, update the table that stores the cutoff times.</span></span>

<span data-ttu-id="228e4-174">Также полезно вести *журнал преобразований* для каждого выполнения ELT.</span><span class="sxs-lookup"><span data-stu-id="228e4-174">It's also useful to record a *lineage* for each ELT run.</span></span> <span data-ttu-id="228e4-175">В таком журнале определенная запись связывается с выполнением ELT, при котором создаются данные.</span><span class="sxs-lookup"><span data-stu-id="228e4-175">For a given record, the lineage associates that record with the ELT run that produced the data.</span></span> <span data-ttu-id="228e4-176">При каждом выполнении ETL создается запись журнала преобразований для каждой таблицы. В этой записи указано время начала и завершения загрузки.</span><span class="sxs-lookup"><span data-stu-id="228e4-176">For each ETL run, a new lineage record is created for every table, showing the starting and ending load times.</span></span> <span data-ttu-id="228e4-177">Ключи для каждой записи журнала преобразований хранятся в таблицах фактов и измерений.</span><span class="sxs-lookup"><span data-stu-id="228e4-177">The lineage keys for each record are stored in the dimension and fact tables.</span></span>

![](./images/city-dimension-table.png)

<span data-ttu-id="228e4-178">Когда новый пакет данных загрузится в хранилище, обновите табличную модель служб Analysis Services.</span><span class="sxs-lookup"><span data-stu-id="228e4-178">After a new batch of data is loaded into the warehouse, refresh the Analysis Services tabular model.</span></span> <span data-ttu-id="228e4-179">Дополнительные сведения см. в статье [Асинхронное обновление с помощью REST API](/azure/analysis-services/analysis-services-async-refresh).</span><span class="sxs-lookup"><span data-stu-id="228e4-179">See [Asynchronous refresh with the REST API](/azure/analysis-services/analysis-services-async-refresh).</span></span>

## <a name="data-cleansing"></a><span data-ttu-id="228e4-180">Очистка данных</span><span class="sxs-lookup"><span data-stu-id="228e4-180">Data cleansing</span></span>

<span data-ttu-id="228e4-181">Очистка данных должна выполняться в рамках процесса ELT.</span><span class="sxs-lookup"><span data-stu-id="228e4-181">Data cleansing should be part of the ELT process.</span></span> <span data-ttu-id="228e4-182">В этой эталонной архитектуре есть один источник с поврежденными данными — таблица численности населения по городам, где некоторые города имеют нулевое значение, возможно, из-за недоступности данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-182">In this reference architecture, one source of bad data is the city population table, where some cities have zero population, perhaps because no data was available.</span></span> <span data-ttu-id="228e4-183">Во время обработки в конвейере ELT такие города удаляются из таблицы численности населения по городам.</span><span class="sxs-lookup"><span data-stu-id="228e4-183">During processing, the ELT pipeline removes those cities from the city population table.</span></span> <span data-ttu-id="228e4-184">Очистку данных следует выполнять с промежуточными таблицами, а не внешними.</span><span class="sxs-lookup"><span data-stu-id="228e4-184">Perform data cleansing on staging tables, rather than external tables.</span></span>

<span data-ttu-id="228e4-185">Ниже приводится хранимая процедура, которая удаляет города с нулевым значением численности населения из соответствующей таблицы.</span><span class="sxs-lookup"><span data-stu-id="228e4-185">Here is the stored procedure that removes the cities with zero population from the City Population table.</span></span> <span data-ttu-id="228e4-186">(Исходный файл можно найти [здесь](https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/citypopulation/%5BIntegration%5D.%5BMigrateExternalCityPopulationData%5D.sql).)</span><span class="sxs-lookup"><span data-stu-id="228e4-186">(You can find the source file [here](https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/citypopulation/%5BIntegration%5D.%5BMigrateExternalCityPopulationData%5D.sql).)</span></span> 

```sql
DELETE FROM [Integration].[CityPopulation_Staging]
WHERE RowNumber in (SELECT DISTINCT RowNumber
FROM [Integration].[CityPopulation_Staging]
WHERE POPULATION = 0
GROUP BY RowNumber
HAVING COUNT(RowNumber) = 4)
```

## <a name="external-data-sources"></a><span data-ttu-id="228e4-187">Внешние источники данных</span><span class="sxs-lookup"><span data-stu-id="228e4-187">External data sources</span></span>

<span data-ttu-id="228e4-188">В хранилищах часто объединяются данные из нескольких источников.</span><span class="sxs-lookup"><span data-stu-id="228e4-188">Data warehouses often consolidate data from multiple sources.</span></span> <span data-ttu-id="228e4-189">Эта эталонная архитектура позволяет загрузить внешний источник данных, содержащий демографические данные.</span><span class="sxs-lookup"><span data-stu-id="228e4-189">This reference architecture loads an external data source that contains demographics data.</span></span> <span data-ttu-id="228e4-190">Этот набор данных доступен в хранилище BLOB-объектов Azure как часть примера [WorldWideImportersDW](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/wide-world-importers/sample-scripts/polybase).</span><span class="sxs-lookup"><span data-stu-id="228e4-190">This dataset is available in Azure blob storage as part of the [WorldWideImportersDW](https://github.com/Microsoft/sql-server-samples/tree/master/samples/databases/wide-world-importers/sample-scripts/polybase) sample.</span></span>

<span data-ttu-id="228e4-191">В Фабрике данных Azure можно копировать данные прямо из хранилища BLOB-объектов с помощью [соответствующего соединителя](/azure/data-factory/connector-azure-blob-storage).</span><span class="sxs-lookup"><span data-stu-id="228e4-191">Azure Data Factory can copy directly from blob storage, using the [blob storage connector](/azure/data-factory/connector-azure-blob-storage).</span></span> <span data-ttu-id="228e4-192">Но соединителю требуется строка подключения или подписанный URL-адрес, поэтому вы не сможете использовать соединитель для копирования BLOB-объекта с общим доступом на чтение.</span><span class="sxs-lookup"><span data-stu-id="228e4-192">However, the connector requires a connection string or a shared access signature, so it can't be used to copy a blob with public read access.</span></span> <span data-ttu-id="228e4-193">В качестве решения с помощью PolyBase создайте внешнюю таблицу в хранилище BLOB-объектов и скопируйте внешние таблицы в Хранилище данных SQL.</span><span class="sxs-lookup"><span data-stu-id="228e4-193">As a workaround, you can use PolyBase to create an external table over Blob storage and then copy the external tables into SQL Data Warehouse.</span></span> 

## <a name="handling-large-binary-data"></a><span data-ttu-id="228e4-194">Обработка больших двоичных данных</span><span class="sxs-lookup"><span data-stu-id="228e4-194">Handling large binary data</span></span> 

<span data-ttu-id="228e4-195">В базе данных-источника таблица Cities содержит столбец Location, в котором представлены данные пространственного типа — [geography](/sql/t-sql/spatial-geography/spatial-types-geography).</span><span class="sxs-lookup"><span data-stu-id="228e4-195">In the source database, the Cities table has a Location column that holds a [geography](/sql/t-sql/spatial-geography/spatial-types-geography) spatial data type.</span></span> <span data-ttu-id="228e4-196">Хранилище данных SQL изначально не поддерживает тип данных **geography**, поэтому при загрузке такие поля преобразуются в тип **varbinary**.</span><span class="sxs-lookup"><span data-stu-id="228e4-196">SQL Data Warehouse doesn't support the **geography** type natively, so this field is converted to a **varbinary** type during loading.</span></span> <span data-ttu-id="228e4-197">Дополнительные сведения см. в разделе [Обходные решения для неподдерживаемых типов данных](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types#unsupported-data-types).</span><span class="sxs-lookup"><span data-stu-id="228e4-197">(See [Workarounds for unsupported data types](/azure/sql-data-warehouse/sql-data-warehouse-tables-data-types#unsupported-data-types).)</span></span>

<span data-ttu-id="228e4-198">Но в PolyBase размер столбца не должен превышать значение `varbinary(8000)`. Это означает, что некоторые данные могут быть усечены.</span><span class="sxs-lookup"><span data-stu-id="228e4-198">However, PolyBase supports a maximum column size of `varbinary(8000)`, which means some data could be truncated.</span></span> <span data-ttu-id="228e4-199">Чтобы решить эту проблему, вы можете разбить данные на блоки во время экспорта, а затем воссоединить их, как описано ниже:</span><span class="sxs-lookup"><span data-stu-id="228e4-199">A workaround for this problem is to break the data up into chunks during export, and then reassemble the chunks, as follows:</span></span>

1. <span data-ttu-id="228e4-200">Создайте временную промежуточную таблицу для столбца Location.</span><span class="sxs-lookup"><span data-stu-id="228e4-200">Create a temporary staging table for the Location column.</span></span>

2. <span data-ttu-id="228e4-201">Для каждого города разбейте данные расположения на блоки объемом 8000 байт. В результате для каждого города будет создано 1 &ndash; N строк.</span><span class="sxs-lookup"><span data-stu-id="228e4-201">For each city, split the location data into 8000-byte chunks, resulting in 1 &ndash; N rows for each city.</span></span>

3. <span data-ttu-id="228e4-202">Чтобы воссоединить эти блоки, с помощью оператора T-SQL [PIVOT](/sql/t-sql/queries/from-using-pivot-and-unpivot) преобразуйте строки в столбцы, а затем сцепите значения столбцов для каждого города.</span><span class="sxs-lookup"><span data-stu-id="228e4-202">To reassemble the chunks, use the T-SQL [PIVOT](/sql/t-sql/queries/from-using-pivot-and-unpivot) operator to convert rows into columns and then concatenate the column values for each city.</span></span>

<span data-ttu-id="228e4-203">Сложность состоит в том, что для каждого города будет разное количество строк, в зависимости от размера географических данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-203">The challenge is that each city will be split into a different number of rows, depending on the size of geography data.</span></span> <span data-ttu-id="228e4-204">Для использования оператора PIVOT необходимо, чтобы у каждого города было одинаковое количество строк.</span><span class="sxs-lookup"><span data-stu-id="228e4-204">For the PIVOT operator to work, every city must have the same number of rows.</span></span> <span data-ttu-id="228e4-205">Для этого запрос T-SQL (сведения о нем см. [здесь][MergeLocation]) выполняет некоторые приемы, чтобы включить строки с пустыми значениями, обеспечивая одинаковое количество столбцов для каждого города после сведения.</span><span class="sxs-lookup"><span data-stu-id="228e4-205">To make this work, the T-SQL query (which you can view [here][MergeLocation]) does some tricks to pad out the rows with blank values, so that every city has the same number of columns after the pivot.</span></span> <span data-ttu-id="228e4-206">Результирующий запрос выполняется намного быстрее, чем последовательный циклический перебор каждой строки.</span><span class="sxs-lookup"><span data-stu-id="228e4-206">The resulting query turns out to be much faster than looping through the rows one at a time.</span></span>

<span data-ttu-id="228e4-207">Такой же подход используется для данных изображений.</span><span class="sxs-lookup"><span data-stu-id="228e4-207">The same approach is used for image data.</span></span>

## <a name="slowly-changing-dimensions"></a><span data-ttu-id="228e4-208">Медленно изменяющиеся измерения</span><span class="sxs-lookup"><span data-stu-id="228e4-208">Slowly changing dimensions</span></span>

<span data-ttu-id="228e4-209">Данные измерений являются относительно статичными, но могут изменяться.</span><span class="sxs-lookup"><span data-stu-id="228e4-209">Dimension data is relatively static, but it can change.</span></span> <span data-ttu-id="228e4-210">Например, продукт может быть переназначен другой категории продуктов.</span><span class="sxs-lookup"><span data-stu-id="228e4-210">For example, a product might get reassigned to a different product category.</span></span> <span data-ttu-id="228e4-211">Медленно изменяющиеся измерения можно обрабатывать несколькими способами.</span><span class="sxs-lookup"><span data-stu-id="228e4-211">There are several approaches to handling slowly changing dimensions.</span></span> <span data-ttu-id="228e4-212">Для этого чаще всего используется [тип 2](https://wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row), который позволяет добавлять новую запись при каждом изменении измерения.</span><span class="sxs-lookup"><span data-stu-id="228e4-212">A common technique, called [Type 2](https://wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row), is to add a new record whenever a dimension changes.</span></span> 

<span data-ttu-id="228e4-213">Чтобы реализовать такой метод, в таблицу измерений нужно добавить столбцы, которые определяют фактический диапазон дат для определенной записи.</span><span class="sxs-lookup"><span data-stu-id="228e4-213">In order to implement the Type 2 approach, dimension tables need additional columns that specify the effective date range for a given record.</span></span> <span data-ttu-id="228e4-214">Кроме того, нужно продублировать первичные ключи из базы данных-источника, чтобы таблица измерений содержала смоделированные первичные ключи.</span><span class="sxs-lookup"><span data-stu-id="228e4-214">Also, primary keys from the source database will be duplicated, so the dimension table must have an artificial primary key.</span></span>

<span data-ttu-id="228e4-215">На изображении ниже показана таблица Dimension.City.</span><span class="sxs-lookup"><span data-stu-id="228e4-215">The following image shows the Dimension.City table.</span></span> <span data-ttu-id="228e4-216">В столбце `WWI City ID` представлены первичные ключи из базы данных-источника.</span><span class="sxs-lookup"><span data-stu-id="228e4-216">The `WWI City ID` column is the primary key from the source database.</span></span> <span data-ttu-id="228e4-217">А в столбце `City Key` содержатся смоделированные ключи, созданные при выполнении конвейера ETL.</span><span class="sxs-lookup"><span data-stu-id="228e4-217">The `City Key` column is an artificial key generated during the ETL pipeline.</span></span> <span data-ttu-id="228e4-218">Также обратите внимание, что в таблице есть столбцы `Valid From` и `Valid To`, которые определяют период, в который строка была актуальна.</span><span class="sxs-lookup"><span data-stu-id="228e4-218">Also notice that the table has `Valid From` and `Valid To` columns, which define the range when each row was valid.</span></span> <span data-ttu-id="228e4-219">Текущие значения в столбце `Valid To` равны 9999-12-31.</span><span class="sxs-lookup"><span data-stu-id="228e4-219">Current values have a `Valid To` equal to '9999-12-31'.</span></span>

![](./images/city-dimension-table.png)

<span data-ttu-id="228e4-220">Преимущество такого подхода заключается в том, что он позволяет хранить данные журнала, которые могут пригодиться для анализа.</span><span class="sxs-lookup"><span data-stu-id="228e4-220">The advantage of this approach is that it preserves historical data, which can be valuable for analysis.</span></span> <span data-ttu-id="228e4-221">Но это также означает, что у одной сущности будет несколько строк.</span><span class="sxs-lookup"><span data-stu-id="228e4-221">However, it also means there will be multiple rows for the same entity.</span></span> <span data-ttu-id="228e4-222">Например, ниже показаны записи, для которых в столбце `WWI City ID` указано значение 28561:</span><span class="sxs-lookup"><span data-stu-id="228e4-222">For example, here are the records that match `WWI City ID` = 28561:</span></span>

![](./images/city-dimension-table-2.png)

<span data-ttu-id="228e4-223">Каждый факт Sales необходимо связать с одной строкой в таблице измерения City, соответствующей дате выставления счета.</span><span class="sxs-lookup"><span data-stu-id="228e4-223">For each Sales fact, you want to associate that fact with a single row in City dimension table, corresponding to the invoice date.</span></span> <span data-ttu-id="228e4-224">В рамках процесса ETL создайте дополнительный столбец.</span><span class="sxs-lookup"><span data-stu-id="228e4-224">As part of the ETL process, create an additional column that</span></span> 

<span data-ttu-id="228e4-225">Следующий запрос T-SQL создает временную таблицу, связывающую каждый счет с правильным ключом в столбце City Key таблицы измерений City.</span><span class="sxs-lookup"><span data-stu-id="228e4-225">The following T-SQL query creates a temporary table that associates each invoice with the correct City Key from the City dimension table.</span></span>

```sql
CREATE TABLE CityHolder
WITH (HEAP , DISTRIBUTION = HASH([WWI Invoice ID]))
AS
SELECT DISTINCT s1.[WWI Invoice ID] AS [WWI Invoice ID],
                c.[City Key] AS [City Key]
    FROM [Integration].[Sale_Staging] s1
    CROSS APPLY (
                SELECT TOP 1 [City Key]
                    FROM [Dimension].[City]
                WHERE [WWI City ID] = s1.[WWI City ID]
                    AND s1.[Last Modified When] > [Valid From]
                    AND s1.[Last Modified When] <= [Valid To]
                ORDER BY [Valid From], [City Key] DESC
                ) c

```

<span data-ttu-id="228e4-226">Эта таблица используется для заполнения столбца в таблице фактов Sales:</span><span class="sxs-lookup"><span data-stu-id="228e4-226">This table is used to populate a column in the Sales fact table:</span></span>

```sql
UPDATE [Integration].[Sale_Staging]
SET [Integration].[Sale_Staging].[WWI Customer ID] =  CustomerHolder.[WWI Customer ID]
```

<span data-ttu-id="228e4-227">Этот столбец позволяет с помощью запроса Power BI найти нужную запись в таблице City для заданного счета на продажу.</span><span class="sxs-lookup"><span data-stu-id="228e4-227">This column enables a Power BI query to find the correct City record for a given sales invoice.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="228e4-228">Вопросы безопасности</span><span class="sxs-lookup"><span data-stu-id="228e4-228">Security considerations</span></span>

<span data-ttu-id="228e4-229">Чтобы обеспечить дополнительную защиту ресурсов служб Azure только в своей виртуальной сети, используйте [конечные точки виртуальной сети](/azure/virtual-network/virtual-network-service-endpoints-overview).</span><span class="sxs-lookup"><span data-stu-id="228e4-229">For additional security, you can use [Virtual Network service endpoints](/azure/virtual-network/virtual-network-service-endpoints-overview) to secure Azure service resources to only your virtual network.</span></span> <span data-ttu-id="228e4-230">Это позволит полностью исключить открытый доступ из Интернета к этим ресурсам и разрешить трафик только из виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="228e4-230">This fully removes public Internet access to those resources, allowing traffic only from your virtual network.</span></span>

<span data-ttu-id="228e4-231">В этом случае вам нужно создать виртуальную сеть в Azure и частные конечные точки для служб Azure.</span><span class="sxs-lookup"><span data-stu-id="228e4-231">With this approach, you create a VNet in Azure and then create private service endpoints for Azure services.</span></span> <span data-ttu-id="228e4-232">В такие службы трафик будет поступать только из виртуальной сети.</span><span class="sxs-lookup"><span data-stu-id="228e4-232">Those services are then restricted to traffic from that virtual network.</span></span> <span data-ttu-id="228e4-233">Доступ к таким службам можно также получить в локальной сети через шлюз.</span><span class="sxs-lookup"><span data-stu-id="228e4-233">You can also reach them from your on-premises network through a gateway.</span></span>

<span data-ttu-id="228e4-234">Следует учитывать следующие ограничения.</span><span class="sxs-lookup"><span data-stu-id="228e4-234">Be aware of the following limitations:</span></span>

- <span data-ttu-id="228e4-235">На момент создания этой эталонной архитектуры конечные точки службы поддерживались для службы хранилища Azure и Хранилища данных SQL Azure, но не для служб Azure Analysis Services.</span><span class="sxs-lookup"><span data-stu-id="228e4-235">At the time this reference architecture was created, VNet service endpoints are supported for Azure Storage and Azure SQL Data Warehouse, but not for Azure Analysis Service.</span></span> <span data-ttu-id="228e4-236">Последние сведения о состоянии поддержки см. [здесь](https://azure.microsoft.com/updates/?product=virtual-network).</span><span class="sxs-lookup"><span data-stu-id="228e4-236">Check the latest status [here](https://azure.microsoft.com/updates/?product=virtual-network).</span></span> 

- <span data-ttu-id="228e4-237">Если для службы хранилища Azure включены конечные точки службы, PolyBase не сможет скопировать данные из службы хранилища в хранилище данных SQL.</span><span class="sxs-lookup"><span data-stu-id="228e4-237">If service endpoints are enabled for Azure Storage, PolyBase cannot copy data from Storage into SQL Data Warehouse.</span></span> <span data-ttu-id="228e4-238">Эту проблему можно устранить.</span><span class="sxs-lookup"><span data-stu-id="228e4-238">There is a mitigation for this issue.</span></span> <span data-ttu-id="228e4-239">Дополнительные сведения см. в разделе [Влияние использования конечных точек службы виртуальной сети со службой хранилища Azure](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=%2fazure%2fvirtual-network%2ftoc.json#impact-of-using-vnet-service-endpoints-with-azure-storage).</span><span class="sxs-lookup"><span data-stu-id="228e4-239">For more information, see [Impact of using VNet Service Endpoints with Azure storage](/azure/sql-database/sql-database-vnet-service-endpoint-rule-overview?toc=%2fazure%2fvirtual-network%2ftoc.json#impact-of-using-vnet-service-endpoints-with-azure-storage).</span></span> 

- <span data-ttu-id="228e4-240">Чтобы переместить данные из локального расположения в службу хранилища Azure, нужно добавить в список разрешений общедоступные IP-адреса из локальной среды или ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="228e4-240">To move data from on-premises into Azure Storage, you will need to whitelist public IP addresses from your on-premises or ExpressRoute.</span></span> <span data-ttu-id="228e4-241">Дополнительные сведения см. в разделе [Защита служб Azure в виртуальных сетях](/azure/virtual-network/virtual-network-service-endpoints-overview#securing-azure-services-to-virtual-networks).</span><span class="sxs-lookup"><span data-stu-id="228e4-241">For details, see [Securing Azure services to virtual networks](/azure/virtual-network/virtual-network-service-endpoints-overview#securing-azure-services-to-virtual-networks).</span></span>

- <span data-ttu-id="228e4-242">Чтобы службы Analysis Services могли считывать данные из хранилища данных SQL, разверните виртуальные машины Windows в виртуальной сети, которая содержит конечную точку службы "Хранилище данных SQL".</span><span class="sxs-lookup"><span data-stu-id="228e4-242">To enable Analysis Services to read data from SQL Data Warehouse, deploy a Windows VM to the virtual network that contains the SQL Data Warehouse service endpoint.</span></span> <span data-ttu-id="228e4-243">Установите в этой виртуальной машине [локальный шлюз данных Azure](/azure/analysis-services/analysis-services-gateway).</span><span class="sxs-lookup"><span data-stu-id="228e4-243">Install [Azure On-premises Data Gateway](/azure/analysis-services/analysis-services-gateway) on this VM.</span></span> <span data-ttu-id="228e4-244">Затем подключите службы Azure Analysis Services к этому шлюзу данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-244">Then connect your Azure Analysis service to the data gateway.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="228e4-245">Развертывание решения</span><span class="sxs-lookup"><span data-stu-id="228e4-245">Deploy the solution</span></span>

<span data-ttu-id="228e4-246">Пример развертывания для этой архитектуры можно найти на портале [GitHub][ref-arch-repo-folder].</span><span class="sxs-lookup"><span data-stu-id="228e4-246">A deployment for this reference architecture is available on [GitHub][ref-arch-repo-folder].</span></span> <span data-ttu-id="228e4-247">Он позволяет развернуть следующее:</span><span class="sxs-lookup"><span data-stu-id="228e4-247">It deploys the following:</span></span>

  * <span data-ttu-id="228e4-248">Виртуальную машину Windows для имитации локального сервера базы данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-248">A Windows VM to simulate an on-premises database server.</span></span> <span data-ttu-id="228e4-249">Она включает SQL Server 2017 и связанные с ним инструменты, а также Power BI Desktop.</span><span class="sxs-lookup"><span data-stu-id="228e4-249">It includes SQL Server 2017 and related tools, along with Power BI Desktop.</span></span>
  * <span data-ttu-id="228e4-250">Учетная запись хранения Azure, которая обеспечивает хранилище больших двоичных объектов для хранения данных, экспортированных из базы данных SQL Server.</span><span class="sxs-lookup"><span data-stu-id="228e4-250">An Azure storage account that provides Blob storage to hold data exported from the SQL Server database.</span></span>
  * <span data-ttu-id="228e4-251">Экземпляр хранилища данных SQL Azure.</span><span class="sxs-lookup"><span data-stu-id="228e4-251">An Azure SQL Data Warehouse instance.</span></span>
  * <span data-ttu-id="228e4-252">Экземпляр службы Azure Analysis Services.</span><span class="sxs-lookup"><span data-stu-id="228e4-252">An Azure Analysis Services instance.</span></span>
  * <span data-ttu-id="228e4-253">Фабрику данных Azure и конвейер фабрики данных для задания ELT.</span><span class="sxs-lookup"><span data-stu-id="228e4-253">Azure Data Factory and the Data Factory pipeline for the ELT job.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="228e4-254">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="228e4-254">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="variables"></a><span data-ttu-id="228e4-255">Переменные</span><span class="sxs-lookup"><span data-stu-id="228e4-255">Variables</span></span>

<span data-ttu-id="228e4-256">В следующих шагах используются некоторые определяемые пользователем переменные.</span><span class="sxs-lookup"><span data-stu-id="228e4-256">The steps that follow include some user-defined variables.</span></span> <span data-ttu-id="228e4-257">Вам нужно заменить их значения собственными значениями.</span><span class="sxs-lookup"><span data-stu-id="228e4-257">You will need to replace these with values that you define.</span></span>

- <span data-ttu-id="228e4-258">`<data_factory_name>`.</span><span class="sxs-lookup"><span data-stu-id="228e4-258">`<data_factory_name>`.</span></span> <span data-ttu-id="228e4-259">Имя фабрики данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-259">Data Factory name.</span></span>
- <span data-ttu-id="228e4-260">`<analysis_server_name>`.</span><span class="sxs-lookup"><span data-stu-id="228e4-260">`<analysis_server_name>`.</span></span> <span data-ttu-id="228e4-261">Имя сервера служб Analysis Services.</span><span class="sxs-lookup"><span data-stu-id="228e4-261">Analysis Services server name.</span></span>
- <span data-ttu-id="228e4-262">`<active_directory_upn>`.</span><span class="sxs-lookup"><span data-stu-id="228e4-262">`<active_directory_upn>`.</span></span> <span data-ttu-id="228e4-263">Имя участника-пользователя Azure Active Directory (UPN).</span><span class="sxs-lookup"><span data-stu-id="228e4-263">Your Azure Active Directory user principal name (UPN).</span></span> <span data-ttu-id="228e4-264">Например, `user@contoso.com`.</span><span class="sxs-lookup"><span data-stu-id="228e4-264">For example, `user@contoso.com`.</span></span>
- <span data-ttu-id="228e4-265">`<data_warehouse_server_name>`.</span><span class="sxs-lookup"><span data-stu-id="228e4-265">`<data_warehouse_server_name>`.</span></span> <span data-ttu-id="228e4-266">Имя сервера Хранилища данных SQL.</span><span class="sxs-lookup"><span data-stu-id="228e4-266">SQL Data Warehouse server name.</span></span>
- <span data-ttu-id="228e4-267">`<data_warehouse_password>`.</span><span class="sxs-lookup"><span data-stu-id="228e4-267">`<data_warehouse_password>`.</span></span> <span data-ttu-id="228e4-268">Пароль администратора Хранилища данных SQL.</span><span class="sxs-lookup"><span data-stu-id="228e4-268">SQL Data Warehouse administrator password.</span></span>
- <span data-ttu-id="228e4-269">`<resource_group_name>`.</span><span class="sxs-lookup"><span data-stu-id="228e4-269">`<resource_group_name>`.</span></span> <span data-ttu-id="228e4-270">Имя группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="228e4-270">The name of the resource group.</span></span>
- <span data-ttu-id="228e4-271">`<region>`.</span><span class="sxs-lookup"><span data-stu-id="228e4-271">`<region>`.</span></span> <span data-ttu-id="228e4-272">Регион Azure, в котором будут развертываться ресурсы.</span><span class="sxs-lookup"><span data-stu-id="228e4-272">The Azure region where the resources will be deployed.</span></span>
- <span data-ttu-id="228e4-273">`<storage_account_name>`.</span><span class="sxs-lookup"><span data-stu-id="228e4-273">`<storage_account_name>`.</span></span> <span data-ttu-id="228e4-274">имя учетной записи хранения;</span><span class="sxs-lookup"><span data-stu-id="228e4-274">Storage account name.</span></span> <span data-ttu-id="228e4-275">которое должно соответствовать [правилам именования](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) для учетных записей хранения.</span><span class="sxs-lookup"><span data-stu-id="228e4-275">Must follow the [naming rules](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) for Storage accounts.</span></span>
- <span data-ttu-id="228e4-276">`<sql-db-password>`.</span><span class="sxs-lookup"><span data-stu-id="228e4-276">`<sql-db-password>`.</span></span> <span data-ttu-id="228e4-277">Пароль для входа на сервер SQL Server.</span><span class="sxs-lookup"><span data-stu-id="228e4-277">SQL Server login password.</span></span>

### <a name="deploy-azure-data-factory"></a><span data-ttu-id="228e4-278">Развертывание Фабрики данных Azure</span><span class="sxs-lookup"><span data-stu-id="228e4-278">Deploy Azure Data Factory</span></span>

1. <span data-ttu-id="228e4-279">Перейдите в папку `data\enterprise_bi_sqldw_advanced\azure\templates` в [репозитории GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="228e4-279">Navigate to the `data\enterprise_bi_sqldw_advanced\azure\templates` folder of the [GitHub repository][ref-arch-repo].</span></span>

2. <span data-ttu-id="228e4-280">Выполните следующую команду в Azure CLI, чтобы создать группу ресурсов.</span><span class="sxs-lookup"><span data-stu-id="228e4-280">Run the following Azure CLI command to create a resource group.</span></span>  

    ```bash
    az group create --name <resource_group_name> --location <region>  
    ```

    <span data-ttu-id="228e4-281">Укажите регион, в котором поддерживается Хранилище данных SQL Azure, Azure Analysis Services и Фабрика данных версии 2.</span><span class="sxs-lookup"><span data-stu-id="228e4-281">Specify a region that supports SQL Data Warehouse, Azure Analysis Services, and Data Factory v2.</span></span> <span data-ttu-id="228e4-282">Ознакомьтесь со статьей [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/).</span><span class="sxs-lookup"><span data-stu-id="228e4-282">See [Azure Products by Region](https://azure.microsoft.com/global-infrastructure/services/)</span></span>

3. <span data-ttu-id="228e4-283">Выполните следующую команду</span><span class="sxs-lookup"><span data-stu-id="228e4-283">Run the following command</span></span>

    ```
    az group deployment create --resource-group <resource_group_name> \
        --template-file adf-create-deploy.json \
        --parameters factoryName=<data_factory_name> location=<location>
    ```

<span data-ttu-id="228e4-284">Затем на портале Azure получите ключ аутентификации для [среды выполнения интеграции](/azure/data-factory/concepts-integration-runtime) Фабрики данных Azure, как показано ниже:</span><span class="sxs-lookup"><span data-stu-id="228e4-284">Next, use the Azure Portal to get the authentication key for the Azure Data Factory [integration runtime](/azure/data-factory/concepts-integration-runtime), as follows:</span></span>

1. <span data-ttu-id="228e4-285">На [портале Azure](https://portal.azure.com/) перейдите к экземпляру Фабрики данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-285">In the [Azure Portal](https://portal.azure.com/), navigate to the Data Factory instance.</span></span>

2. <span data-ttu-id="228e4-286">В колонке "Фабрика данных" щелкните **Создание и мониторинг**.</span><span class="sxs-lookup"><span data-stu-id="228e4-286">In the Data Factory blade, click **Author & Monitor**.</span></span> <span data-ttu-id="228e4-287">После этого в другом окне браузера откроется портал Фабрики данных Azure.</span><span class="sxs-lookup"><span data-stu-id="228e4-287">This opens the Azure Data Factory portal in another browser window.</span></span>

    ![](./images/adf-blade.png)

3. <span data-ttu-id="228e4-288">На портале Фабрики данных Azure щелкните значок карандаша ("Создать").</span><span class="sxs-lookup"><span data-stu-id="228e4-288">In the Azure Data Factory portal, select the pencil icon ("Author").</span></span> 

4. <span data-ttu-id="228e4-289">Выберите **Подключения** и **Среды выполнения интеграции**.</span><span class="sxs-lookup"><span data-stu-id="228e4-289">Click **Connections**, and then select **Integration Runtimes**.</span></span>

5. <span data-ttu-id="228e4-290">Рядом с **sourceIntegrationRuntime** щелкните значок карандаша ("Изменить").</span><span class="sxs-lookup"><span data-stu-id="228e4-290">Under **sourceIntegrationRuntime**, click the pencil icon ("Edit").</span></span>

    > [!NOTE]
    > <span data-ttu-id="228e4-291">На портале отобразится состояние "Недоступно".</span><span class="sxs-lookup"><span data-stu-id="228e4-291">The portal will show the status as "unavailable".</span></span> <span data-ttu-id="228e4-292">Это ожидаемое поведение, так как вы еще не развернули сервер в локальной среде.</span><span class="sxs-lookup"><span data-stu-id="228e4-292">This is expected until you deploy the on-premises server.</span></span>

6. <span data-ttu-id="228e4-293">Найдите поле **Key1** и скопируйте значение ключа аутентификации.</span><span class="sxs-lookup"><span data-stu-id="228e4-293">Find **Key1** and copy the value of the authentication key.</span></span>

<span data-ttu-id="228e4-294">Этот ключ аутентификации понадобится вам на следующем шаге.</span><span class="sxs-lookup"><span data-stu-id="228e4-294">You will need the authentication key for the next step.</span></span>

### <a name="deploy-the-simulated-on-premises-server"></a><span data-ttu-id="228e4-295">Развертывание имитации локального сервера</span><span class="sxs-lookup"><span data-stu-id="228e4-295">Deploy the simulated on-premises server</span></span>

<span data-ttu-id="228e4-296">На этом шаге вы развернете виртуальную машину в качестве имитируемого локального сервера, который включает SQL Server 2017 и связанные инструменты.</span><span class="sxs-lookup"><span data-stu-id="228e4-296">This step deploys a VM as a simulated on-premises server, which includes SQL Server 2017 and related tools.</span></span> <span data-ttu-id="228e4-297">Также вы загрузите [базу данных OLTP Wide World Importers][WWI] в SQL Server.</span><span class="sxs-lookup"><span data-stu-id="228e4-297">It also loads the [Wide World Importers OLTP database][wwi] into SQL Server.</span></span>

1. <span data-ttu-id="228e4-298">Перейдите в папку `data\enterprise_bi_sqldw_advanced\onprem\templates` в репозитории.</span><span class="sxs-lookup"><span data-stu-id="228e4-298">Navigate to the `data\enterprise_bi_sqldw_advanced\onprem\templates` folder of the repository.</span></span>

2. <span data-ttu-id="228e4-299">В файле `onprem.parameters.json` найдите `adminPassword`.</span><span class="sxs-lookup"><span data-stu-id="228e4-299">In the `onprem.parameters.json` file, search for `adminPassword`.</span></span> <span data-ttu-id="228e4-300">Это пароль для входа в виртуальную машину SQL Server.</span><span class="sxs-lookup"><span data-stu-id="228e4-300">This is the password to log into the SQL Server VM.</span></span> <span data-ttu-id="228e4-301">Замените это значение другим паролем.</span><span class="sxs-lookup"><span data-stu-id="228e4-301">Replace the value with another password.</span></span>

3. <span data-ttu-id="228e4-302">В этом же файле найдите `SqlUserCredentials`.</span><span class="sxs-lookup"><span data-stu-id="228e4-302">In the same file, search for `SqlUserCredentials`.</span></span> <span data-ttu-id="228e4-303">Это свойство определяет учетные данные SQL Server.</span><span class="sxs-lookup"><span data-stu-id="228e4-303">This property specifies the SQL Server account credentials.</span></span> <span data-ttu-id="228e4-304">Замените пароль другим значением.</span><span class="sxs-lookup"><span data-stu-id="228e4-304">Replace the password with a different value.</span></span>

4. <span data-ttu-id="228e4-305">В том же файле вставьте в параметр `IntegrationRuntimeGatewayKey` ключ аутентификации для среды выполнения интеграции, как показано ниже:</span><span class="sxs-lookup"><span data-stu-id="228e4-305">In the same file, paste the Integration Runtime authentication key into the `IntegrationRuntimeGatewayKey` parameter, as shown below:</span></span>

    ```json
    "protectedSettings": {
        "configurationArguments": {
            "SqlUserCredentials": {
                "userName": ".\\adminUser",
                "password": "<sql-db-password>"
            },
            "IntegrationRuntimeGatewayKey": "<authentication key>"
        }
    ```

5. <span data-ttu-id="228e4-306">Выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="228e4-306">Run the following command.</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.parameters.json --deploy
    ```

<span data-ttu-id="228e4-307">Выполнение этого шага может занять 20 или 30 минут.</span><span class="sxs-lookup"><span data-stu-id="228e4-307">This step may take 20 to 30 minutes to complete.</span></span> <span data-ttu-id="228e4-308">В ходе этого процесса выполняется скрипт [DSC](/powershell/dsc/overview), чтобы установить средства и восстановить базу данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-308">It includes running a [DSC](/powershell/dsc/overview) script to install the tools and restore the database.</span></span> 

### <a name="deploy-azure-resources"></a><span data-ttu-id="228e4-309">Развертывание ресурсов Azure</span><span class="sxs-lookup"><span data-stu-id="228e4-309">Deploy Azure resources</span></span>

<span data-ttu-id="228e4-310">На этом этапе вы подготовите Хранилище данных SQL Azure, Azure Analysis Services и Фабрику данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-310">This step provisions SQL Data Warehouse, Azure Analysis Services, and Data Factory.</span></span>

1. <span data-ttu-id="228e4-311">Перейдите в папку `data\enterprise_bi_sqldw_advanced\azure\templates` в [репозитории GitHub][ref-arch-repo].</span><span class="sxs-lookup"><span data-stu-id="228e4-311">Navigate to the `data\enterprise_bi_sqldw_advanced\azure\templates` folder of the [GitHub repository][ref-arch-repo].</span></span>

2. <span data-ttu-id="228e4-312">Выполните следующую команду CLI Azure.</span><span class="sxs-lookup"><span data-stu-id="228e4-312">Run the following Azure CLI command.</span></span> <span data-ttu-id="228e4-313">Замените значения параметров, отображаемые в угловых скобках.</span><span class="sxs-lookup"><span data-stu-id="228e4-313">Replace the parameter values shown in angle brackets.</span></span>

    ```bash
    az group deployment create --resource-group <resource_group_name> \
     --template-file azure-resources-deploy.json \
     --parameters "dwServerName"="<data_warehouse_server_name>" \
     "dwAdminLogin"="adminuser" "dwAdminPassword"="<data_warehouse_password>" \ 
     "storageAccountName"="<storage_account_name>" \
     "analysisServerName"="<analysis_server_name>" \
     "analysisServerAdmin"="<user@contoso.com>"
    ```

    - <span data-ttu-id="228e4-314">Параметр `storageAccountName` должен следовать [правилам именования](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) для учетных записей хранения.</span><span class="sxs-lookup"><span data-stu-id="228e4-314">The `storageAccountName` parameter must follow the [naming rules](../../best-practices/naming-conventions.md#naming-rules-and-restrictions) for Storage accounts.</span></span> 
    - <span data-ttu-id="228e4-315">Для параметра `analysisServerAdmin` используйте имя участника-пользователя Azure Active Directory (UPN).</span><span class="sxs-lookup"><span data-stu-id="228e4-315">For the `analysisServerAdmin` parameter, use your Azure Active Directory user principal name (UPN).</span></span>

3. <span data-ttu-id="228e4-316">Чтобы получить ключ доступа для учетной записи хранения, выполните следующую команду Azure CLI.</span><span class="sxs-lookup"><span data-stu-id="228e4-316">Run the following Azure CLI command to get the access key for the storage account.</span></span> <span data-ttu-id="228e4-317">Этот ключ понадобится вам на следующем шаге.</span><span class="sxs-lookup"><span data-stu-id="228e4-317">You will use this key in the next step.</span></span>

    ```bash
    az storage account keys list -n <storage_account_name> -g <resource_group_name> --query [0].value
    ```

4. <span data-ttu-id="228e4-318">Выполните следующую команду CLI Azure.</span><span class="sxs-lookup"><span data-stu-id="228e4-318">Run the following Azure CLI command.</span></span> <span data-ttu-id="228e4-319">Замените значения параметров, отображаемые в угловых скобках.</span><span class="sxs-lookup"><span data-stu-id="228e4-319">Replace the parameter values shown in angle brackets.</span></span> 

    ```bash
    az group deployment create --resource-group <resource_group_name> \
    --template-file adf-pipeline-deploy.json \
    --parameters "factoryName"="<data_factory_name>" \
    "sinkDWConnectionString"="Server=tcp:<data_warehouse_server_name>.database.windows.net,1433;Initial Catalog=wwi;Persist Security Info=False;User ID=adminuser;Password=<data_warehouse_password>;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" \
    "blobConnectionString"="DefaultEndpointsProtocol=https;AccountName=<storage_account_name>;AccountKey=<storage_account_key>;EndpointSuffix=core.windows.net" \
    "sourceDBConnectionString"="Server=sql1;Database=WideWorldImporters;User Id=adminuser;Password=<sql-db-password>;Trusted_Connection=True;"
    ```

    <span data-ttu-id="228e4-320">В строках подключения есть вложенные строки, указанные в угловых скобках. Вам нужно заменить эти вложенные строки.</span><span class="sxs-lookup"><span data-stu-id="228e4-320">The connection strings have substrings shown in angle brackets that must be replaced.</span></span> <span data-ttu-id="228e4-321">Для `<storage_account_key>` укажите ключ, полученный на предыдущем шаге.</span><span class="sxs-lookup"><span data-stu-id="228e4-321">For `<storage_account_key>`, use the key that you got in the previous step.</span></span> <span data-ttu-id="228e4-322">Для `<sql-db-password>` задайте пароль учетной записи SQL Server, который вы указали в файле `onprem.parameters.json` ранее.</span><span class="sxs-lookup"><span data-stu-id="228e4-322">For `<sql-db-password>`, use the SQL Server account password that you specified in the `onprem.parameters.json` file previously.</span></span>

### <a name="run-the-data-warehouse-scripts"></a><span data-ttu-id="228e4-323">Выполнение скриптов хранилища данных</span><span class="sxs-lookup"><span data-stu-id="228e4-323">Run the data warehouse scripts</span></span>

1. <span data-ttu-id="228e4-324">На [портале Azure](https://portal.azure.com/) найдите локальную виртуальную машину с именем `sql-vm1`.</span><span class="sxs-lookup"><span data-stu-id="228e4-324">In the [Azure Portal](https://portal.azure.com/), find the on-premises VM, which is named `sql-vm1`.</span></span> <span data-ttu-id="228e4-325">Имя пользователя и пароль для виртуальной машины указаны в файле `onprem.parameters.json`.</span><span class="sxs-lookup"><span data-stu-id="228e4-325">The user name and password for the VM are specified in the `onprem.parameters.json` file.</span></span>

2. <span data-ttu-id="228e4-326">Нажмите кнопку **Подключиться** и используйте удаленный рабочий стол, чтобы подключиться к виртуальной машине.</span><span class="sxs-lookup"><span data-stu-id="228e4-326">Click **Connect** and use Remote Desktop to connect to the VM.</span></span>

3. <span data-ttu-id="228e4-327">В сеансе удаленного рабочего стола откройте окно командной строки и перейдите к следующей папке на виртуальной машине:</span><span class="sxs-lookup"><span data-stu-id="228e4-327">From your Remote Desktop session, open a command prompt and navigate to the following folder on the VM:</span></span>

    ```
    cd C:\SampleDataFiles\reference-architectures\data\enterprise_bi_sqldw_advanced\azure\sqldw_scripts
    ```

4. <span data-ttu-id="228e4-328">Выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="228e4-328">Run the following command:</span></span>

    ```
    deploy_database.cmd -S <data_warehouse_server_name>.database.windows.net -d wwi -U adminuser -P <data_warehouse_password> -N -I
    ```

    <span data-ttu-id="228e4-329">Для `<data_warehouse_server_name>` и `<data_warehouse_password>` укажите имя сервера хранилища данных и пароль, которые вы задали ранее.</span><span class="sxs-lookup"><span data-stu-id="228e4-329">For `<data_warehouse_server_name>` and `<data_warehouse_password>`, use the data warehouse server name and password from earlier.</span></span>

<span data-ttu-id="228e4-330">Для проверки подключитесь к базе данных Хранилища данных SQL с помощью SQL Server Management Studio (SSMS).</span><span class="sxs-lookup"><span data-stu-id="228e4-330">To verify this step, you can use SQL Server Management Studio (SSMS) to connect to the SQL Data Warehouse database.</span></span> <span data-ttu-id="228e4-331">Вы должны увидеть схемы таблиц базы данных.</span><span class="sxs-lookup"><span data-stu-id="228e4-331">You should see the database table schemas.</span></span>

### <a name="run-the-data-factory-pipeline"></a><span data-ttu-id="228e4-332">Запуск конвейера Фабрики данных</span><span class="sxs-lookup"><span data-stu-id="228e4-332">Run the Data Factory pipeline</span></span>

1. <span data-ttu-id="228e4-333">В том же сеансе удаленного рабочего стола откройте окно PowerShell.</span><span class="sxs-lookup"><span data-stu-id="228e4-333">From the same Remote Desktop session, open a PowerShell window.</span></span>

2. <span data-ttu-id="228e4-334">Выполните следующую команду PowerShell.</span><span class="sxs-lookup"><span data-stu-id="228e4-334">Run the following PowerShell command.</span></span> <span data-ttu-id="228e4-335">Выберите **Да** при появлении запроса.</span><span class="sxs-lookup"><span data-stu-id="228e4-335">Choose **Yes** when prompted.</span></span>

    ```powershell
    Install-Module -Name AzureRM -AllowClobber
    ```

3. <span data-ttu-id="228e4-336">Выполните следующую команду PowerShell.</span><span class="sxs-lookup"><span data-stu-id="228e4-336">Run the following PowerShell command.</span></span> <span data-ttu-id="228e4-337">При появлении запроса введите свои учетные данные Azure.</span><span class="sxs-lookup"><span data-stu-id="228e4-337">Enter your Azure credentials when prompted.</span></span>

    ```powershell
    Connect-AzureRmAccount 
    ```

4. <span data-ttu-id="228e4-338">Выполните приведенные ниже команды PowerShell.</span><span class="sxs-lookup"><span data-stu-id="228e4-338">Run the following PowerShell commands.</span></span> <span data-ttu-id="228e4-339">Замените значения в угловых скобках.</span><span class="sxs-lookup"><span data-stu-id="228e4-339">Replace the values in angle brackets.</span></span>

    ```powershell
    Set-AzureRmContext -SubscriptionId <subscription id>

    Invoke-AzureRmDataFactoryV2Pipeline -DataFactory <data-factory-name> -PipelineName "MasterPipeline" -ResourceGroupName <resource_group_name>

5. In the Azure Portal, navigate to the Data Factory instance that was created earlier.

6. In the Data Factory blade, click **Author & Monitor**. This opens the Azure Data Factory portal in another browser window.

    ![](./images/adf-blade.png)

7. In the Azure Data Factory portal, click the **Monitor** icon. 

8. Verify that the pipeline completes successfully. It can take a few minutes.

    ![](./images/adf-pipeline-progress.png)


## Build the Analysis Services model

In this step, you will create a tabular model that imports data from the data warehouse. Then you will deploy the model to Azure Analysis Services.

**Create a new tabular project**

1. From your Remote Desktop session, launch SQL Server Data Tools 2015.

2. Select **File** > **New** > **Project**.

3. In the **New Project** dialog, under **Templates**, select  **Business Intelligence** > **Analysis Services** > **Analysis Services Tabular Project**. 

4. Name the project and click **OK**.

5. In the **Tabular model designer** dialog, select **Integrated workspace**  and set **Compatibility level** to `SQL Server 2017 / Azure Analysis Services (1400)`. 

6. Click **OK**.


**Import data**

1. In the **Tabular Model Explorer** window, right-click the project and select **Import from Data Source**.

2. Select **Azure SQL Data Warehouse** and click **Connect**.

3. For **Server**, enter the fully qualified name of your Azure SQL Data Warehouse server. You can get this value from the Azure Portal. For **Database**, enter `wwi`. Click **OK**.

4. In the next dialog, choose **Database** authentication and enter your Azure SQL Data Warehouse user name and password, and click **OK**.

5. In the **Navigator** dialog, select the checkboxes for the **Fact.\*** and **Dimension.\*** tables.

    ![](./images/analysis-services-import-2.png)

6. Click **Load**. When processing is complete, click **Close**. You should now see a tabular view of the data.

**Create measures**

1. In the model designer, select the **Fact Sale** table.

2. Click a cell in the the measure grid. By default, the measure grid is displayed below the table. 

    ![](./images/tabular-model-measures.png)

3. In the formula bar, enter the following and press ENTER:

    ```
    <span data-ttu-id="228e4-340">Total Sales:=SUM('Fact Sale'[Total Including Tax])</span><span class="sxs-lookup"><span data-stu-id="228e4-340">Total Sales:=SUM('Fact Sale'[Total Including Tax])</span></span>
    ```

4. Repeat these steps to create the following measures:

    ```
    <span data-ttu-id="228e4-341">Number of Years:=(MAX('Fact CityPopulation'[YearNumber])-MIN('Fact CityPopulation'[YearNumber]))+1</span><span class="sxs-lookup"><span data-stu-id="228e4-341">Number of Years:=(MAX('Fact CityPopulation'[YearNumber])-MIN('Fact CityPopulation'[YearNumber]))+1</span></span>
    
    <span data-ttu-id="228e4-342">Beginning Population:=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[YearNumber]=MIN('Fact CityPopulation'[YearNumber])))</span><span class="sxs-lookup"><span data-stu-id="228e4-342">Beginning Population:=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[YearNumber]=MIN('Fact CityPopulation'[YearNumber])))</span></span>
    
    <span data-ttu-id="228e4-343">Ending Population:=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[YearNumber]=MAX('Fact CityPopulation'[YearNumber])))</span><span class="sxs-lookup"><span data-stu-id="228e4-343">Ending Population:=CALCULATE(SUM('Fact CityPopulation'[Population]),FILTER('Fact CityPopulation','Fact CityPopulation'[YearNumber]=MAX('Fact CityPopulation'[YearNumber])))</span></span>
    
    <span data-ttu-id="228e4-344">CAGR:=IFERROR((([Ending Population]/[Beginning Population])^(1/[Number of Years]))-1,0)</span><span class="sxs-lookup"><span data-stu-id="228e4-344">CAGR:=IFERROR((([Ending Population]/[Beginning Population])^(1/[Number of Years]))-1,0)</span></span>
    ```

    ![](./images/analysis-services-measures.png)

For more information about creating measures in SQL Server Data Tools, see [Measures](/sql/analysis-services/tabular-models/measures-ssas-tabular).

**Create relationships**

1. In the **Tabular Model Explorer** window, right-click the project and select **Model View** > **Diagram View**.

2. Drag the **[Fact Sale].[City Key]** field to the **[Dimension City].[City Key]** field to create a relationship.  

3. Drag the **[Face CityPopulation].[City Key]** field to the **[Dimension City].[City Key]** field.  

    ![](./images/analysis-services-relations-2.png)

**Deploy the model**

1. From the **File** menu, choose **Save All**.

2. In **Solution Explorer**, right-click the project and select **Properties**. 

3. Under **Server**, enter the URL of your Azure Analysis Services instance. You can get this value from the Azure Portal. In the portal, select the Analysis Services resource, click the Overview pane, and look for the **Server Name** property. It will be similar to `asazure://westus.asazure.windows.net/contoso`. Click **OK**.

    ![](./images/analysis-services-properties.png)

4. In **Solution Explorer**, right-click the project and select **Deploy**. Sign into Azure if prompted. When processing is complete, click **Close**.

5. In the Azure portal, view the details for your Azure Analysis Services instance. Verify that your model appears in the list of models.

    ![](./images/analysis-services-models.png)

## Analyze the data in Power BI Desktop

In this step, you will use Power BI to create a report from the data in Analysis Services.

1. From your Remote Desktop session, launch Power BI Desktop.

2. In the Welcome Scren, click **Get Data**.

3. Select **Azure** > **Azure Analysis Services database**. Click **Connect**

    ![](./images/power-bi-get-data.png)

4. Enter the URL of your Analysis Services instance, then click **OK**. Sign into Azure if prompted.

5. In the **Navigator** dialog, expand the tabular project, select the model, and click **OK**.

2. In the **Visualizations** pane, select the **Table** icon. In the Report view, resize the visualization to make it larger.

6. In the **Fields** pane, expand **Dimension City**.

7. From **Dimension City**, drag **City** and **State Province** to the **Values** well.

9. In the **Fields** pane, expand **Fact Sale**.

10. From **Fact Sale**, drag **CAGR**, **Ending Population**,  and **Total Sales** to the **Value** well.

11. Under **Visual Level Filters**, select **Ending Population**. Set the filter to "is greater than 100000" and click **Apply filter**.

12. Under **Visual Level Filters**, select **Total Sales**. Set the filter to "is 0" and click **Apply filter**.

![](./images/power-bi-report-2.png)

The table now shows cities with population greater than 100,000 and zero sales. CAGR  stands for Compounded Annual Growth Rate and measures the rate of population growth per city. You could use this value to find cities with high growth rates, for example. However, note that the values for CAGR in the model aren't accurate, because they are derived from sample data.

To learn more about Power BI Desktop, see [Getting started with Power BI Desktop](/power-bi/desktop-getting-started).


[adf]: //azure/data-factory
[azure-cli-2]: //azure/install-azure-cli
[azbb-repo]: https://github.com/mspnp/template-building-blocks
[azbb-wiki]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[MergeLocation]: https://github.com/mspnp/reference-architectures/blob/master/data/enterprise_bi_sqldw_advanced/azure/sqldw_scripts/city/%5BIntegration%5D.%5BMergeLocation%5D.sql
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[ref-arch-repo-folder]: https://github.com/mspnp/reference-architectures/tree/master/data/enterprise_bi_sqldw_advanced
[wwi]: //sql/sample/world-wide-importers/wide-world-importers-oltp-database
