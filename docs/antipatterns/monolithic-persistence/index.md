---
title: "Антишаблон монолитной сохраняемости"
description: "Хранение всех данных приложения в одном хранилище может привести к снижению уровня производительности."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 7f04b9f0805c281068b6b2edaf040683773e6f6e
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
---
# <a name="monolithic-persistence-antipattern"></a><span data-ttu-id="d41e6-103">Антишаблон монолитной сохраняемости</span><span class="sxs-lookup"><span data-stu-id="d41e6-103">Monolithic Persistence antipattern</span></span>

<span data-ttu-id="d41e6-104">Хранение всех данных приложения в одном хранилище может привести к снижению уровня производительности. Это связано с возможным конфликтом ресурсов или с тем, что это хранилище не подходит некоторым типам данных.</span><span class="sxs-lookup"><span data-stu-id="d41e6-104">Putting all of an application's data into a single data store can hurt performance, either because it leads to resource contention, or because the data store is not a good fit for some of the data.</span></span>

## <a name="problem-description"></a><span data-ttu-id="d41e6-105">Описание проблемы</span><span class="sxs-lookup"><span data-stu-id="d41e6-105">Problem description</span></span>

<span data-ttu-id="d41e6-106">Часто все приложения используют одно хранилище, независимо от типов данных, которые нужно в нем хранить.</span><span class="sxs-lookup"><span data-stu-id="d41e6-106">Historically, applications have often used a single data store, regardless of the different types of data that the application might need to store.</span></span> <span data-ttu-id="d41e6-107">Обычно это позволяет упростить разработку приложений или же это по каким-то причинам требуется группе разработчиков.</span><span class="sxs-lookup"><span data-stu-id="d41e6-107">Usually this was done to simplify the application design, or else to match the existing skill set of the development team.</span></span> 

<span data-ttu-id="d41e6-108">Современные облачные системы часто имеют дополнительные функциональные и нефункциональные требования. В них хранится большое количество разнородных типов данных, таких как документы, изображения, кэшированные данные, сообщения в очереди, журналы приложений и данные телеметрии.</span><span class="sxs-lookup"><span data-stu-id="d41e6-108">Modern cloud-based systems often have additional functional and nonfunctional requirements, and need to store many heterogenous types of data, such as documents, images, cached data, queued messages, application logs, and telemetry.</span></span> <span data-ttu-id="d41e6-109">Если следовать традиционному подходу и поместить их в одно хранилище данных, это может снизить производительность. Две основные причины этого заключаются в следующем:</span><span class="sxs-lookup"><span data-stu-id="d41e6-109">Following the traditional approach and putting all of this information into the same data store can hurt performance, for two main reasons:</span></span>

- <span data-ttu-id="d41e6-110">Хранение больших объемов несвязанных данных в том же хранилище и их извлечение может вызывать конфликты, что, в свою очередь, приводит к снижению времени отклика и ошибкам подключения.</span><span class="sxs-lookup"><span data-stu-id="d41e6-110">Storing and retrieving large amounts of unrelated data in the same data store can cause contention, which in turn leads to slow response times and connection failures.</span></span>
- <span data-ttu-id="d41e6-111">Выбранное хранилище может не подходить всем типам данных, или же оно не оптимизировано для выполняемых приложением операций.</span><span class="sxs-lookup"><span data-stu-id="d41e6-111">Whichever data store is chosen, it might not be the best fit for all of the different types of data, or it might not be optimized for the operations that the application performs.</span></span> 

<span data-ttu-id="d41e6-112">Ниже приведен пример контроллера веб-API ASP.NET, который добавляет новую запись в базу данных, а также записывает результаты в журнал.</span><span class="sxs-lookup"><span data-stu-id="d41e6-112">The following example shows an ASP.NET Web API controller that adds a new record to a database and also records the result to a log.</span></span> <span data-ttu-id="d41e6-113">Этот журнал хранится в одной базе данных с бизнес-данными.</span><span class="sxs-lookup"><span data-stu-id="d41e6-113">The log is held in the same database as the business data.</span></span> <span data-ttu-id="d41e6-114">Полный пример см. [здесь][sample-app].</span><span class="sxs-lookup"><span data-stu-id="d41e6-114">You can find the complete sample [here][sample-app].</span></span>

```csharp
public class MonoController : ApiController
{
    private static readonly string ProductionDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        await DataAccess.LogAsync(ProductionDb, LogTableName);
        return Ok();
    }
}
```

<span data-ttu-id="d41e6-115">Скорость, с которой создаются записи журнала, скорее всего, повлияет на производительность бизнес-операций.</span><span class="sxs-lookup"><span data-stu-id="d41e6-115">The rate at which log records are generated will probably affect the performance of the business operations.</span></span> <span data-ttu-id="d41e6-116">Кроме того, на это может повлиять и другой компонент, например монитор процессов приложения, который регулярно читает и обрабатывает данные журнала.</span><span class="sxs-lookup"><span data-stu-id="d41e6-116">And if another component, such as an application process monitor, regularly reads and processes the log data, that can also affect the business operations.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="d41e6-117">Как устранить проблему</span><span class="sxs-lookup"><span data-stu-id="d41e6-117">How to fix the problem</span></span>

<span data-ttu-id="d41e6-118">Разделите данные в соответствии с их использованием.</span><span class="sxs-lookup"><span data-stu-id="d41e6-118">Separate data according to its use.</span></span> <span data-ttu-id="d41e6-119">Поместите каждый набор данных в хранилище, которое лучше всего подходит цели его использования.</span><span class="sxs-lookup"><span data-stu-id="d41e6-119">For each data set, select a data store that best matches how that data set will be used.</span></span> <span data-ttu-id="d41e6-120">В предыдущем примере приложение должно выполнять вход в отдельное хранилище из базы данных, в которой хранятся бизнес-данные:</span><span class="sxs-lookup"><span data-stu-id="d41e6-120">In the previous example, the application should be logging to a separate store from the database that holds business data:</span></span> 

```csharp
public class PolyController : ApiController
{
    private static readonly string ProductionDb = ...;
    private static readonly string LogDb = ...;

    public async Task<IHttpActionResult> PostAsync([FromBody]string value)
    {
        await DataAccess.InsertPurchaseOrderHeaderAsync(ProductionDb);
        // Log to a different data store.
        await DataAccess.LogAsync(LogDb, LogTableName);
        return Ok();
    }
}
```

## <a name="considerations"></a><span data-ttu-id="d41e6-121">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="d41e6-121">Considerations</span></span>

- <span data-ttu-id="d41e6-122">Разделите данные в зависимости от их использования и способа доступа.</span><span class="sxs-lookup"><span data-stu-id="d41e6-122">Separate data by the way it is used and how it is accessed.</span></span> <span data-ttu-id="d41e6-123">Например, не храните в том же хранилище данные журнала и бизнес-данные.</span><span class="sxs-lookup"><span data-stu-id="d41e6-123">For example, don't store log information and business data in the same data store.</span></span> <span data-ttu-id="d41e6-124">Эти два типа имеют совсем другие требования и шаблоны доступа.</span><span class="sxs-lookup"><span data-stu-id="d41e6-124">These types of data have significantly different requirements and patterns of access.</span></span> <span data-ttu-id="d41e6-125">Записи журнала по своей сути требуют последовательного доступа, а бизнес-данные чаще всего — произвольного (в большинстве случаев они реляционные).</span><span class="sxs-lookup"><span data-stu-id="d41e6-125">Log records are inherently sequential, while business data is more likely to require random access, and is often relational.</span></span>

- <span data-ttu-id="d41e6-126">Учтите шаблон доступа к каждому типу данных.</span><span class="sxs-lookup"><span data-stu-id="d41e6-126">Consider the data access pattern for each type of data.</span></span> <span data-ttu-id="d41e6-127">Например, храните форматированные отчеты и документы в базе данных документов, например [Cosmos DB][CosmosDB], но кэшируйте временные данные с помощью [кэша Redis для Azure][Azure-cache].</span><span class="sxs-lookup"><span data-stu-id="d41e6-127">For example, store formatted reports and documents in a document database such as [Cosmos DB][CosmosDB], but use [Azure Redis Cache][Azure-cache] to cache temporary data.</span></span>

- <span data-ttu-id="d41e6-128">Если вы следовали этим инструкциям, но по-прежнему достигаете предельных значений, возможно, нужно выполнить масштабирование базы данных.</span><span class="sxs-lookup"><span data-stu-id="d41e6-128">If you follow this guidance but still reach the limits of the database, you may need to scale up the database.</span></span> <span data-ttu-id="d41e6-129">Кроме того, попробуйте выполнить горизонтальное масштабирование и секционирование нагрузки по серверам баз данных.</span><span class="sxs-lookup"><span data-stu-id="d41e6-129">Also consider scaling horizontally and partitioning the load across database servers.</span></span> <span data-ttu-id="d41e6-130">Однако чтобы выполнить секционирование, может потребоваться перепроектировать приложение.</span><span class="sxs-lookup"><span data-stu-id="d41e6-130">However, partitioning may require redesigning the application.</span></span> <span data-ttu-id="d41e6-131">Дополнительные сведения см. в статье [Секционирование данных][DataPartitioningGuidance].</span><span class="sxs-lookup"><span data-stu-id="d41e6-131">For more information, see [Data partitioning][DataPartitioningGuidance].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="d41e6-132">Как определить проблему</span><span class="sxs-lookup"><span data-stu-id="d41e6-132">How to detect the problem</span></span>

<span data-ttu-id="d41e6-133">Система, скорее всего, начнет работать значительно медленнее, и в конечном счете произойдет сбой. Это связано с тем, что системе не хватает ресурсов, таких как подключения к базе данных.</span><span class="sxs-lookup"><span data-stu-id="d41e6-133">The system will likely slow down dramatically and eventually fail, as the system runs out of resources such as database connections.</span></span>

<span data-ttu-id="d41e6-134">Чтобы определить причину, сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="d41e6-134">You can perform the following steps to help identify the cause.</span></span>

1. <span data-ttu-id="d41e6-135">Выполните инструментирование системы, чтобы собрать статистику производительности.</span><span class="sxs-lookup"><span data-stu-id="d41e6-135">Instrument the system to record the key performance statistics.</span></span> <span data-ttu-id="d41e6-136">Зафиксируйте время выполнения каждой операции и точек, в которых приложение читает и записывает данные.</span><span class="sxs-lookup"><span data-stu-id="d41e6-136">Capture timing information for each operation, as well as the points where the application reads and writes data.</span></span>
1. <span data-ttu-id="d41e6-137">Если это возможно, проследите несколько дней за выполнением системы в рабочей среде, чтобы получить реальное представление о ее использовании.</span><span class="sxs-lookup"><span data-stu-id="d41e6-137">If possible, monitor the system running for a few days in a production environment to get a real-world view of how the system is used.</span></span> <span data-ttu-id="d41e6-138">В противном случае, используя скрипты, выполните нагрузочные тесты с правдоподобным количеством виртуальных пользователей, выполняющих типичные операции.</span><span class="sxs-lookup"><span data-stu-id="d41e6-138">If this is not possible, run scripted load tests with a realistic volume of virtual users performing a typical series of operations.</span></span>
2. <span data-ttu-id="d41e6-139">Определите периоды низкой производительности с помощью данных телеметрии.</span><span class="sxs-lookup"><span data-stu-id="d41e6-139">Use the telemetry data to identify periods of poor performance.</span></span>
3. <span data-ttu-id="d41e6-140">Определите, к каким хранилищам данных осуществлялся доступ в течение этих периодов.</span><span class="sxs-lookup"><span data-stu-id="d41e6-140">Identify which data stores were accessed during those periods.</span></span>
4. <span data-ttu-id="d41e6-141">Определите ресурсы хранилища данных, в связи с которыми могут возникать конфликты.</span><span class="sxs-lookup"><span data-stu-id="d41e6-141">Identify data storage resources that might be experiencing contention.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="d41e6-142">Пример диагностики</span><span class="sxs-lookup"><span data-stu-id="d41e6-142">Example diagnosis</span></span>

<span data-ttu-id="d41e6-143">В следующих разделах эти шаги применяются к примеру приложения, описанному ранее.</span><span class="sxs-lookup"><span data-stu-id="d41e6-143">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="instrument-and-monitor-the-system"></a><span data-ttu-id="d41e6-144">Инструментирование и мониторинг системы</span><span class="sxs-lookup"><span data-stu-id="d41e6-144">Instrument and monitor the system</span></span>

<span data-ttu-id="d41e6-145">На графике ниже показаны результаты нагрузочного тестирования примера приложения, описанного ранее.</span><span class="sxs-lookup"><span data-stu-id="d41e6-145">The following graph shows the results of load testing the sample application described earlier.</span></span> <span data-ttu-id="d41e6-146">При тестировании применялась пошаговая нагрузка с не более 1000 параллельно работающих пользователей.</span><span class="sxs-lookup"><span data-stu-id="d41e6-146">The test used a step load of up to 1000 concurrent users.</span></span>

![Результаты производительности, полученные при нагрузочном тесте с использованием контроллера на основе SQL][MonolithicScenarioLoadTest]

<span data-ttu-id="d41e6-148">Если увеличить нагрузку до 700 пользователей, также увеличивается и пропускная способность.</span><span class="sxs-lookup"><span data-stu-id="d41e6-148">As the load increases to 700 users, so does the throughput.</span></span> <span data-ttu-id="d41e6-149">Но на том этапе пропускная способность выравнивается и система начинает работать с максимальной мощностью.</span><span class="sxs-lookup"><span data-stu-id="d41e6-149">But at that point, throughput levels off, and the system appears to be running at its maximum capacity.</span></span> <span data-ttu-id="d41e6-150">Среднее время ответа постепенно увеличивается с количеством пользователей, показывая, что система не справляется с нагрузкой.</span><span class="sxs-lookup"><span data-stu-id="d41e6-150">The average response gradually increases with user load, showing that the system can't keep up with demand.</span></span>

### <a name="identify-periods-of-poor-performance"></a><span data-ttu-id="d41e6-151">Определение периодов низкой производительности</span><span class="sxs-lookup"><span data-stu-id="d41e6-151">Identify periods of poor performance</span></span>

<span data-ttu-id="d41e6-152">Наблюдая за рабочей системой, вы можете заметить определенные шаблоны поведения.</span><span class="sxs-lookup"><span data-stu-id="d41e6-152">If you are monitoring the production system, you might notice patterns.</span></span> <span data-ttu-id="d41e6-153">Например, время отклика может значительно увеличиваться каждый день в то же время.</span><span class="sxs-lookup"><span data-stu-id="d41e6-153">For example, response times might drop off significantly at the same time each day.</span></span> <span data-ttu-id="d41e6-154">Причина этого может заключаться в регулярной рабочей нагрузке, запланированном пакетном задании или в простом увеличении количества пользователей в системе в определенное время.</span><span class="sxs-lookup"><span data-stu-id="d41e6-154">This could be caused by a regular workload or scheduled batch job, or just because the system has more users at certain times.</span></span> <span data-ttu-id="d41e6-155">В этом случае следует уделить внимание данным телеметрии этих событий.</span><span class="sxs-lookup"><span data-stu-id="d41e6-155">You should focus on the telemetry data for these events.</span></span>

<span data-ttu-id="d41e6-156">Определите зависимости между увеличением времени отклика и нагрузки базы данных или числа операций ввода-вывода в общих ресурсах.</span><span class="sxs-lookup"><span data-stu-id="d41e6-156">Look for correlations between increased response times and increased database activity or I/O to shared resources.</span></span> <span data-ttu-id="d41e6-157">Если такие зависимости существуют, это означает, что база данных может быть узким местом.</span><span class="sxs-lookup"><span data-stu-id="d41e6-157">If there are correlations, it means the database might be a bottleneck.</span></span>

### <a name="identify-which-data-stores-are-accessed-during-those-periods"></a><span data-ttu-id="d41e6-158">Определение хранилищ данных, к которым осуществляется доступ в течение этих периодов</span><span class="sxs-lookup"><span data-stu-id="d41e6-158">Identify which data stores are accessed during those periods</span></span>

<span data-ttu-id="d41e6-159">На следующем графике показано использование единиц пропускной способности базы данных (DTU) во время нагрузочного теста.</span><span class="sxs-lookup"><span data-stu-id="d41e6-159">The next graph shows the utilization of database throughput units (DTU) during the load test.</span></span> <span data-ttu-id="d41e6-160">(DTU — это мера доступной емкости, основанная на смешанном показателе использования ЦП, выделения памяти и скорости ввода-вывода.) Использование единиц DTU быстро достигло показателя 100 %.</span><span class="sxs-lookup"><span data-stu-id="d41e6-160">(A DTU is a measure of available capacity, and is a combination of CPU utilization, memory allocation, I/O rate.) Utilization of DTUs quickly reached 100%.</span></span> <span data-ttu-id="d41e6-161">Это примерно точка, в которой наблюдался скачок пропускной способности на предыдущем графике.</span><span class="sxs-lookup"><span data-stu-id="d41e6-161">This is roughly the point where throughput peaked in the previous graph.</span></span> <span data-ttu-id="d41e6-162">До завершения теста показатель использования базы данных оставался очень высоким.</span><span class="sxs-lookup"><span data-stu-id="d41e6-162">Database utilization remained very high until the test finished.</span></span> <span data-ttu-id="d41e6-163">Под конец произошло небольшое снижение через регулирование количества запросов, снижения количества подключений к базе данных или другие факторы.</span><span class="sxs-lookup"><span data-stu-id="d41e6-163">There is a slight drop toward the end, which could be caused by throttling, competition for database connections, or other factors.</span></span>

![Монитор на классическом портале Azure с показателями использования ресурсов базы данных][MonolithicDatabaseUtilization]

### <a name="examine-the-telemetry-for-the-data-stores"></a><span data-ttu-id="d41e6-165">Анализ данных телеметрии хранилищ</span><span class="sxs-lookup"><span data-stu-id="d41e6-165">Examine the telemetry for the data stores</span></span>

<span data-ttu-id="d41e6-166">Выполните инструментирование хранилищ данных, чтобы собрать низкоуровневые сведения о действии.</span><span class="sxs-lookup"><span data-stu-id="d41e6-166">Instrument the data stores to capture the low-level details of the activity.</span></span> <span data-ttu-id="d41e6-167">В статистике доступа к данным примера приложения показано большое количество операций вставки в таблице `PurchaseOrderHeader` и `MonoLog`.</span><span class="sxs-lookup"><span data-stu-id="d41e6-167">In the sample application, the data access statistics showed a high volume of insert operations performed against both the `PurchaseOrderHeader` table and the `MonoLog` table.</span></span> 

![Статистика доступа к данным примера приложения][MonolithicDataAccessStats]

### <a name="identify-resource-contention"></a><span data-ttu-id="d41e6-169">Определение конфликта ресурсов</span><span class="sxs-lookup"><span data-stu-id="d41e6-169">Identify resource contention</span></span>

<span data-ttu-id="d41e6-170">На этом этапе вы можете просмотреть исходный код и сосредоточить свое внимание на точках, где приложение осуществляло доступ к нужным ресурсам.</span><span class="sxs-lookup"><span data-stu-id="d41e6-170">At this point, you can review the source code, focusing on the points where contended resources are accessed by the application.</span></span> <span data-ttu-id="d41e6-171">Например, проверьте следующие ситуации:</span><span class="sxs-lookup"><span data-stu-id="d41e6-171">Look for situations such as:</span></span>

- <span data-ttu-id="d41e6-172">Логически разделенные данные записаны в одно хранилище.</span><span class="sxs-lookup"><span data-stu-id="d41e6-172">Data that is logically separate being written to the same store.</span></span> <span data-ttu-id="d41e6-173">Данные, такие как журналы, отчеты и сообщения в очереди, не должны храниться в той же базе данных, что и бизнес-данные.</span><span class="sxs-lookup"><span data-stu-id="d41e6-173">Data such as logs, reports, and queued messages should not be held in the same database as business information.</span></span>
- <span data-ttu-id="d41e6-174">Несоответствие между выбранным хранилищем и типом данных, например большие двоичные объекты или XML-документы в реляционной базе данных.</span><span class="sxs-lookup"><span data-stu-id="d41e6-174">A mismatch between the choice of data store and the type of data, such as large blobs or XML documents in a relational database.</span></span>
- <span data-ttu-id="d41e6-175">Данные, шаблон использования которых значительно отличается, не должны храниться в одном хранилище, например данные с высокой активностью записи и низкой активностью чтения не должны храниться с данными с низкой активностью записи и высокой активностью чтения.</span><span class="sxs-lookup"><span data-stu-id="d41e6-175">Data with significantly different usage patterns that share the same store, such as high-write/low-read data being stored with low-write/high-read data.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="d41e6-176">Реализация решения и проверка результатов</span><span class="sxs-lookup"><span data-stu-id="d41e6-176">Implement the solution and verify the result</span></span>

<span data-ttu-id="d41e6-177">После изменения приложение записывает журналы в отдельное хранилище данных.</span><span class="sxs-lookup"><span data-stu-id="d41e6-177">The application was changed to write logs to a separate data store.</span></span> <span data-ttu-id="d41e6-178">Ниже приведены результаты нагрузочного теста.</span><span class="sxs-lookup"><span data-stu-id="d41e6-178">Here are the load test results:</span></span>

![Результаты производительности, полученные при нагрузочном тесте с использованием контроллера Polyglot][PolyglotScenarioLoadTest]

<span data-ttu-id="d41e6-180">Шаблон пропускной способности аналогичен приведенному на предыдущем графике, но точка, в которой наблюдается скачок производительности, выше примерно на 500 запросов в секунду.</span><span class="sxs-lookup"><span data-stu-id="d41e6-180">The pattern of throughput is similar to the earlier graph, but the point at which performance peaks is approximately 500 requests per second higher.</span></span> <span data-ttu-id="d41e6-181">Среднее время ответа немного меньше.</span><span class="sxs-lookup"><span data-stu-id="d41e6-181">The average response time is marginally lower.</span></span> <span data-ttu-id="d41e6-182">Тем не менее эти статистические данные описывают полную картину.</span><span class="sxs-lookup"><span data-stu-id="d41e6-182">However, these statistics don't tell the full story.</span></span> <span data-ttu-id="d41e6-183">На основе данных телеметрии рабочей базы данных можно увидеть, что скачок использования единиц DTU происходит на показателе примерно 75 %, а не 100 %.</span><span class="sxs-lookup"><span data-stu-id="d41e6-183">Telemetry for the business database shows that DTU utilization peaks at around 75%, rather than 100%.</span></span>

![Монитор на классическом портале Azure с показателями использования ресурсов базы данных в сценарии Polyglot][PolyglotDatabaseUtilization]

<span data-ttu-id="d41e6-185">Аналогичным образом максимальное использование единиц DTU базы данных журналов происходит на показателе примерно 70 %.</span><span class="sxs-lookup"><span data-stu-id="d41e6-185">Similarly, the maximum DTU utilization of the log database only reaches about 70%.</span></span> <span data-ttu-id="d41e6-186">Производительность системы больше не зависит от баз данных.</span><span class="sxs-lookup"><span data-stu-id="d41e6-186">The databases are no longer the limiting factor in the performance of the system.</span></span>

![Монитор на классическом портале Azure с показателями использования ресурсов базы данных журналов в сценарии Polyglot][LogDatabaseUtilization]


## <a name="related-resources"></a><span data-ttu-id="d41e6-188">Связанные ресурсы</span><span class="sxs-lookup"><span data-stu-id="d41e6-188">Related resources</span></span>

- <span data-ttu-id="d41e6-189">[Choose the right data store][data-store-overview] (Выбор правильного хранилища данных)</span><span class="sxs-lookup"><span data-stu-id="d41e6-189">[Choose the right data store][data-store-overview]</span></span>
- <span data-ttu-id="d41e6-190">[Criteria for choosing a data store][data-store-comparison] (Критерии выбора хранилища данных)</span><span class="sxs-lookup"><span data-stu-id="d41e6-190">[Criteria for choosing a data store][data-store-comparison]</span></span>
- <span data-ttu-id="d41e6-191">[Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence][Data-Access-Guide] (Доступ к данным масштабируемых решений с помощью SQL, NoSQL и Polyglot Persistence)</span><span class="sxs-lookup"><span data-stu-id="d41e6-191">[Data Access for Highly-Scalable Solutions: Using SQL, NoSQL, and Polyglot Persistence][Data-Access-Guide]</span></span>
- <span data-ttu-id="d41e6-192">[Секционирование данных][DataPartitioningGuidance]</span><span class="sxs-lookup"><span data-stu-id="d41e6-192">[Data partitioning][DataPartitioningGuidance]</span></span>

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/MonolithicPersistence
[CosmosDB]: http://azure.microsoft.com/services/cosmos-db/
[Azure-cache]: /azure/redis-cache/
[Data-Access-Guide]: https://msdn.microsoft.com/library/dn271399.aspx
[DataPartitioningGuidance]: ../../best-practices/data-partitioning.md
[data-store-overview]: ../../guide/technology-choices/data-store-overview.md
[data-store-comparison]: ../../guide/technology-choices/data-store-comparison.md

[MonolithicScenarioLoadTest]: _images/MonolithicScenarioLoadTest.jpg
[MonolithicDatabaseUtilization]: _images/MonolithicDatabaseUtilization.jpg
[MonolithicDataAccessStats]: _images/MonolithicDataAccessStats.jpg
[PolyglotScenarioLoadTest]: _images/PolyglotScenarioLoadTest.jpg
[PolyglotDatabaseUtilization]: _images/PolyglotDatabaseUtilization.jpg
[LogDatabaseUtilization]: _images/LogDatabaseUtilization.jpg
