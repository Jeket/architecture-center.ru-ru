---
title: Обработка потоков данных с помощью Azure Stream Analytics
description: Создание сквозного конвейера обработки потоков данных в Azure
author: MikeWasson
ms.date: 08/09/2018
ms.openlocfilehash: 82887bdd45f811ac733ead18c1f256098e575253
ms.sourcegitcommit: c4106b58ad08f490e170e461009a4693578294ea
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/10/2018
ms.locfileid: "40025508"
---
# <a name="stream-processing-with-azure-stream-analytics"></a><span data-ttu-id="d0e6a-103">Обработка потоков данных с помощью Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="d0e6a-103">Stream processing with Azure Stream Analytics</span></span>

<span data-ttu-id="d0e6a-104">На схеме эталонной архитектуры представлен сквозной конвейер потоков данных.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-104">This reference architecture shows an end-to-end stream processing pipeline.</span></span> <span data-ttu-id="d0e6a-105">Конвейер принимает данные из двух источников, сопоставляет записи в двух потоках и вычисляет скользящее среднее за определенный промежуток времени.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-105">The pipeline ingests data from two sources, correlates records in the two streams, and calculates a rolling average across a time window.</span></span> <span data-ttu-id="d0e6a-106">Результаты сохраняются для дальнейшего анализа.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-106">The results are stored for further analysis.</span></span> [<span data-ttu-id="d0e6a-107">**Разверните это решение**.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-107">**Deploy this solution**.</span></span>](#deploy-the-solution)

![](./images/stream-processing-asa/stream-processing-asa.png)

<span data-ttu-id="d0e6a-108">**Сценарий**. Компания, предоставляющая услуги такси, собирает данные о каждой поездке в такси.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-108">**Scenario**: A taxi company collects data about each taxi trip.</span></span> <span data-ttu-id="d0e6a-109">В этом сценарии предполагается, что данные отправляются с двух отдельных устройств.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-109">For this scenario, we assume there are two separate devices sending data.</span></span> <span data-ttu-id="d0e6a-110">В такси установлен счетчик, который отправляет данные о каждой поездке, включая сведения о продолжительности, расстоянии, а также местах посадки и высадки.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-110">The taxi has a meter that sends information about each ride &mdash; the duration, distance, and pickup and dropoff locations.</span></span> <span data-ttu-id="d0e6a-111">Отдельное устройство принимает платежи от клиентов и отправляет данные о тарифах.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-111">A separate device accepts payments from customers and sends data about fares.</span></span> <span data-ttu-id="d0e6a-112">Компании необходимо вычислить среднюю сумму чаевых на милю в реальном времени, чтобы определить тенденции.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-112">The taxi company wants to calculate the average tip per mile driven, in real time, in order to spot trends.</span></span>

## <a name="architecture"></a><span data-ttu-id="d0e6a-113">Архитектура</span><span class="sxs-lookup"><span data-stu-id="d0e6a-113">Architecture</span></span>

<span data-ttu-id="d0e6a-114">Архитектура состоит из следующих компонентов:</span><span class="sxs-lookup"><span data-stu-id="d0e6a-114">The architecture consists of the following components.</span></span>

<span data-ttu-id="d0e6a-115">**Источники данных.**</span><span class="sxs-lookup"><span data-stu-id="d0e6a-115">**Data sources**.</span></span> <span data-ttu-id="d0e6a-116">В этой архитектуре существует два источника данных, которые создают потоки данных в реальном времени.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-116">In this architecture, there are two data sources that generate data streams in real time.</span></span> <span data-ttu-id="d0e6a-117">Первый поток содержит сведения о поездке, а второй — о тарифах.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-117">The first stream contains ride information, and the second contains fare information.</span></span> <span data-ttu-id="d0e6a-118">В эталонной архитектуре есть имитированный генератор данных, который считывает данные из набора статических файлов и отправляет данные в Центры событий.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-118">The reference architecture includes a simulated data generator that reads from a set of static files and pushes the data to Event Hubs.</span></span> <span data-ttu-id="d0e6a-119">В реальном приложении источниками данных будут устройства, установленные в такси.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-119">In a real application, the data sources would be devices installed in the taxi cabs.</span></span>

<span data-ttu-id="d0e6a-120">**Концентраторы событий Azure**.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-120">**Azure Event Hubs**.</span></span> <span data-ttu-id="d0e6a-121">[Центры событий](/azure/event-hubs/) — это служба приема событий.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-121">[Event Hubs](/azure/event-hubs/) is an event ingestion service.</span></span> <span data-ttu-id="d0e6a-122">В этой архитектуре используется два экземпляра службы — по одной на каждый источник данных.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-122">This architecture uses two event hub instances, one for each data source.</span></span> <span data-ttu-id="d0e6a-123">Каждый источник данных отправляет поток данных в соответствующую службу.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-123">Each data source sends a stream of data to the associated event hub.</span></span>

<span data-ttu-id="d0e6a-124">**Azure Stream Analytics**.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-124">**Azure Stream Analytics**.</span></span> <span data-ttu-id="d0e6a-125">[Stream Analytics](/azure/stream-analytics/) — это модуль обработки событий.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-125">[Stream Analytics](/azure/stream-analytics/) is an event-processing engine.</span></span> <span data-ttu-id="d0e6a-126">Задание Stream Analytics считывает потоки данных из двух экземпляров и обрабатывает эти потоки.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-126">A Stream Analytics job reads the data streams from the two event hubs and performs stream processing.</span></span>

<span data-ttu-id="d0e6a-127">**Cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-127">**Cosmos DB**.</span></span> <span data-ttu-id="d0e6a-128">Выходные данные задания Stream Analytics — это набор записей, которые регистрируются в виде документов JSON в базе данных документов Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-128">The output from the Stream Analytics job is a series of records, which are written as JSON documents to a Cosmos DB document database.</span></span>

<span data-ttu-id="d0e6a-129">**Microsoft Power BI**.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-129">**Microsoft Power BI**.</span></span> <span data-ttu-id="d0e6a-130">Power BI — набор средств бизнес-аналитики для анализа информации о бизнесе.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-130">Power BI is a suite of business analytics tools to analyze data for business insights.</span></span> <span data-ttu-id="d0e6a-131">В нашей архитектуре данные загружаются из Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-131">In this architecture, it loads the data from Cosmos DB.</span></span> <span data-ttu-id="d0e6a-132">Благодаря этому пользователи могут выполнять анализ полного набора собранных исторических данных.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-132">This allows users to analyze the complete set of historical data that's been collected.</span></span> <span data-ttu-id="d0e6a-133">Также результаты можно передать в виде потока прямо из Stream Analytics в Power BI, чтобы просмотреть данные в реальном времени.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-133">You could also stream the results directly from Stream Analytics to Power BI for a real-time view of the data.</span></span> <span data-ttu-id="d0e6a-134">Дополнительные сведения см. в статье [Потоковая передача в реальном времени в Power BI](/power-bi/service-real-time-streaming).</span><span class="sxs-lookup"><span data-stu-id="d0e6a-134">For more information, see [Real-time streaming in Power BI](/power-bi/service-real-time-streaming).</span></span>

<span data-ttu-id="d0e6a-135">**Azure Monitor**.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-135">**Azure Monitor**.</span></span> <span data-ttu-id="d0e6a-136">[Azure Monitor](/azure/monitoring-and-diagnostics/) собирает метрики производительности о службах Azure, развернутых в решении.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-136">[Azure Monitor](/azure/monitoring-and-diagnostics/) collects performance metrics about the Azure services deployed in the solution.</span></span> <span data-ttu-id="d0e6a-137">Отобразив эти данные в визуализации на панели мониторинга, можно получить полезные сведения о работоспособности решения.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-137">By visualizing these in a dashboard, you can get insights into the health of the solution.</span></span> 

## <a name="data-ingestion"></a><span data-ttu-id="d0e6a-138">Прием данных</span><span class="sxs-lookup"><span data-stu-id="d0e6a-138">Data ingestion</span></span>

<span data-ttu-id="d0e6a-139">Для имитации источника данных в этой эталонной архитектуре используется набор данных<sup>[[1]](#note1)</sup> [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) (Данные о поездках в такси в Нью-Йорке).</span><span class="sxs-lookup"><span data-stu-id="d0e6a-139">To simulate a data source, this reference architecture uses the [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) dataset<sup>[[1]](#note1)</sup>.</span></span> <span data-ttu-id="d0e6a-140">Этот набор содержит данные о поездках в такси в Нью-Йорке за 4 года (2010&ndash;2013).</span><span class="sxs-lookup"><span data-stu-id="d0e6a-140">This dataset contains data about taxi trips in New York City over a 4-year period (2010 &ndash; 2013).</span></span> <span data-ttu-id="d0e6a-141">В нем представлены два типа записей: данные о поездках и тарифах.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-141">It contains two types of record: Ride data and fare data.</span></span> <span data-ttu-id="d0e6a-142">Данные о поездках включают сведения о продолжительности поездки, расстоянии, а также местах посадки и высадки.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-142">Ride data includes trip duration, trip distance, and pickup and dropoff location.</span></span> <span data-ttu-id="d0e6a-143">Данные о тарифах включают сведения о тарифе, налоге и сумме чаевых.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-143">Fare data includes fare, tax, and tip amounts.</span></span> <span data-ttu-id="d0e6a-144">В обоих типах записей есть стандартные поля: номер медальона, лицензия на право вождения и код организации.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-144">Common fields in both record types include medallion number, hack license, and vendor ID.</span></span> <span data-ttu-id="d0e6a-145">Вместе эти три поля позволяют уникально идентифицировать такси и водителя.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-145">Together these three fields uniquely identify a taxi plus a driver.</span></span> <span data-ttu-id="d0e6a-146">Данные хранятся в формате CSV.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-146">The data is stored in CSV format.</span></span> 

<span data-ttu-id="d0e6a-147">Генератор данных — это приложение .NET Core, которое считывает записи и отправляет их в Центры событий Azure.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-147">The data generator is a .NET Core application that reads the records and sends them to Azure Event Hubs.</span></span> <span data-ttu-id="d0e6a-148">Генератор отправляет данные о поездке в формате JSON, а данные о тарифах — в формате CSV.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-148">The generator sends ride data in JSON format and fare data in CSV format.</span></span> 

<span data-ttu-id="d0e6a-149">Для сегментации данных Центры событий используют [секции](/azure/event-hubs/event-hubs-features#partitions).</span><span class="sxs-lookup"><span data-stu-id="d0e6a-149">Event Hubs uses [partitions](/azure/event-hubs/event-hubs-features#partitions) to segment the data.</span></span> <span data-ttu-id="d0e6a-150">Они позволяют объекту-получателю считывать данные каждой секции параллельно.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-150">Partitions allow a consumer to read each partition in parallel.</span></span> <span data-ttu-id="d0e6a-151">При отправке данных в Центры событий можно явно указать ключ секции.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-151">When you send data to Event Hubs, you can specify the partition key explicitly.</span></span> <span data-ttu-id="d0e6a-152">В противном случае записи назначаются секциям методом циклического перебора.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-152">Otherwise, records are assigned to partitions in round-robin fashion.</span></span> 

<span data-ttu-id="d0e6a-153">В нашем примере данные о поездках и тарифах должны в итоге иметь одинаковый идентификатор секции для определенного такси.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-153">In this particular scenario, ride data and fare data should end up with the same partition ID for a given taxi cab.</span></span> <span data-ttu-id="d0e6a-154">Это позволит Stream Analytics применить определенную степень параллелизма при сопоставлении двух потоков.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-154">This enables Stream Analytics to apply a degree of parallelism when it correlates the two streams.</span></span> <span data-ttu-id="d0e6a-155">Запись в секции *n* с данными о поездке будет соответствовать записи в секции *n* с данными о тарифах.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-155">A record in partition *n* of the ride data will match a record in partition *n* of the fare data.</span></span>

![](./images/stream-processing-asa/stream-processing-eh.png)

<span data-ttu-id="d0e6a-156">В генераторе данных общая модель данных для обоих типов записей имеет свойство `PartitionKey`, в котором объединены `Medallion`, `HackLicense` и `VendorId`.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-156">In the data generator, the common data model for both record types has a `PartitionKey` property which is the concatenation of `Medallion`, `HackLicense`, and `VendorId`.</span></span>

```csharp
public abstract class TaxiData
{
    public TaxiData()
    {
    }

    [JsonProperty]
    public long Medallion { get; set; }

    [JsonProperty]
    public long HackLicense { get; set; }

    [JsonProperty]
    public string VendorId { get; set; }

    [JsonProperty]
    public DateTimeOffset PickupTime { get; set; }

    [JsonIgnore]
    public string PartitionKey
    {
        get => $"{Medallion}_{HackLicense}_{VendorId}";
    }
```

<span data-ttu-id="d0e6a-157">Это свойство используется для явного предоставления ключа секции при отправке в Центры событий:</span><span class="sxs-lookup"><span data-stu-id="d0e6a-157">This property is used to provide an explicit partition key when sending to Event Hubs:</span></span>

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

## <a name="stream-processing"></a><span data-ttu-id="d0e6a-158">Потоковая обработка</span><span class="sxs-lookup"><span data-stu-id="d0e6a-158">Stream processing</span></span>

<span data-ttu-id="d0e6a-159">Задание обработки потока определяется с помощью SQL-запроса в несколько отдельных шагов.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-159">The stream processing job is defined using a SQL query with several distinct steps.</span></span> <span data-ttu-id="d0e6a-160">На первых двух шагах просто выбираются записи из двух входных потоков.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-160">The first two steps simply select records from the two input streams.</span></span>

```sql
WITH
Step1 AS (
    SELECT PartitionId,
           TRY_CAST(Medallion AS nvarchar(max)) AS Medallion,
           TRY_CAST(HackLicense AS nvarchar(max)) AS HackLicense,
           VendorId,
           TRY_CAST(PickupTime AS datetime) AS PickupTime,
           TripDistanceInMiles
    FROM [TaxiRide] PARTITION BY PartitionId
),
Step2 AS (
    SELECT PartitionId,
           medallion AS Medallion,
           hack_license AS HackLicense,
           vendor_id AS VendorId,
           TRY_CAST(pickup_datetime AS datetime) AS PickupTime,
           tip_amount AS TipAmount
    FROM [TaxiFare] PARTITION BY PartitionId
),
```

<span data-ttu-id="d0e6a-161">На следующем шаге эти два входных потока объединяются для выбора совпадающих записей из каждого потока.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-161">The next step joins the two input streams to select matching records from each stream.</span></span>

```sql
Step3 AS (
  SELECT
         tr.Medallion,
         tr.HackLicense,
         tr.VendorId,
         tr.PickupTime,
         tr.TripDistanceInMiles,
         tf.TipAmount
    FROM [Step1] tr
    PARTITION BY PartitionId
    JOIN [Step2] tf PARTITION BY PartitionId
      ON tr.Medallion = tf.Medallion
     AND tr.HackLicense = tf.HackLicense
     AND tr.VendorId = tf.VendorId
     AND tr.PickupTime = tf.PickupTime
     AND tr.PartitionId = tf.PartitionId
     AND DATEDIFF(minute, tr, tf) BETWEEN 0 AND 15
)
```

<span data-ttu-id="d0e6a-162">Этот запрос объединяет записи в набор полей, которые уникально идентифицируют совпадающие записи (Medallion, HackLicense, VendorId и PickupTime).</span><span class="sxs-lookup"><span data-stu-id="d0e6a-162">This query joins records on a set of fields that uniquely identify matching records (Medallion, HackLicense, VendorId, and PickupTime).</span></span> <span data-ttu-id="d0e6a-163">Инструкция `JOIN` также содержит идентификатор секции.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-163">The `JOIN` statement also includes the partition ID.</span></span> <span data-ttu-id="d0e6a-164">Как уже упоминалось, преимущество состоит в том, что совпадающие записи всегда имеют одинаковый идентификатор секции в этом сценарии.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-164">As mentioned, this takes advantage of the fact that matching records always have the same partition ID in this scenario.</span></span>

<span data-ttu-id="d0e6a-165">В Stream Analytics объединения являются *темпоральными*, то есть записи объединяются в пределах определенного временного окна.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-165">In Stream Analytics, joins are *temporal*, meaning records are joined within a particular window of time.</span></span> <span data-ttu-id="d0e6a-166">В противном случае задание может бесконечно ожидать сопоставления.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-166">Otherwise, the job might need to wait indefinitely for a match.</span></span> <span data-ttu-id="d0e6a-167">Функция [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/join-azure-stream-analytics) указывает, насколько две совпадающих записи могут быть разделены по времени для сопоставления.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-167">The [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/join-azure-stream-analytics) function specifies how far two matching records can be separated in time for a match.</span></span> 

<span data-ttu-id="d0e6a-168">На последнем шаге задания вычисляется средняя сумма чаевых на милю, сгруппированных по "прыгающему" окну в 5 минут.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-168">The last step in the job computes the average tip per mile, grouped by a hopping window of 5 minutes.</span></span>

```sql
SELECT System.Timestamp AS WindowTime,
       SUM(tr.TipAmount) / SUM(tr.TripDistanceInMiles) AS AverageTipPerMile
  INTO [TaxiDrain]
  FROM [Step3] tr
  GROUP BY HoppingWindow(Duration(minute, 5), Hop(minute, 1))
```

<span data-ttu-id="d0e6a-169">Stream Analytics предоставляет несколько [функций обработки методом окна](/azure/stream-analytics/stream-analytics-window-functions).</span><span class="sxs-lookup"><span data-stu-id="d0e6a-169">Stream Analytics provides several [windowing functions](/azure/stream-analytics/stream-analytics-window-functions).</span></span> <span data-ttu-id="d0e6a-170">"Прыгающее" окно перемещается вперед во времени в рамках фиксированного периода, в нашем примере — на 1 минуту за прыжок.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-170">A hopping window moves forward in time by a fixed period, in this case 1 minute per hop.</span></span> <span data-ttu-id="d0e6a-171">Результатом будет вычисление скользящего среднего за последние 5 минут.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-171">The result is to calculate a moving average over the past 5 minutes.</span></span>

<span data-ttu-id="d0e6a-172">В показанной здесь архитектуре в Cosmos DB сохраняются только результаты задания Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-172">In the architecture shown here, only the results of the Stream Analytics job are saved to Cosmos DB.</span></span> <span data-ttu-id="d0e6a-173">Для сценария с большими данными используйте также функцию [Сбор](/azure/event-hubs/event-hubs-capture-overview) в Центрах событий, чтобы сохранять необработанные данные о событиях в хранилище BLOB-объектов Azure.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-173">For a big data scenario, consider also using [Event Hubs Capture](/azure/event-hubs/event-hubs-capture-overview) to save the raw event data into Azure Blob storage.</span></span> <span data-ttu-id="d0e6a-174">Сохранение необработанных данных позволит вам в дальнейшем выполнять пакетные запросы к данным журнала, чтобы получить новые полезные сведения из этих данных.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-174">Keeping the raw data will allow you to run batch queries over your historical data at later time, in order to derive new insights from the data.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="d0e6a-175">Вопросы масштабируемости</span><span class="sxs-lookup"><span data-stu-id="d0e6a-175">Scalability considerations</span></span>

### <a name="event-hubs"></a><span data-ttu-id="d0e6a-176">концентраторы событий;</span><span class="sxs-lookup"><span data-stu-id="d0e6a-176">Event Hubs</span></span>

<span data-ttu-id="d0e6a-177">Пропускная способность Центров событий вычисляется в [единицах пропускной способности](/azure/event-hubs/event-hubs-features#throughput-units).</span><span class="sxs-lookup"><span data-stu-id="d0e6a-177">The throughput capacity of Event Hubs is measured in [throughput units](/azure/event-hubs/event-hubs-features#throughput-units).</span></span> <span data-ttu-id="d0e6a-178">Вы можете автоматически масштабировать концентратор событий, включив [автоматическое расширение](/azure/event-hubs/event-hubs-auto-inflate). Это позволит автоматически масштабировать единицы пропускной способности в зависимости от трафика вплоть до заданного максимума.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-178">You can autoscale an event hub by enabling [auto-inflate](/azure/event-hubs/event-hubs-auto-inflate), which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span> 

### <a name="stream-analytics"></a><span data-ttu-id="d0e6a-179">Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="d0e6a-179">Stream Analytics</span></span>

<span data-ttu-id="d0e6a-180">В Stream Analytics вычислительные ресурсы, выделенные для задания, измеряются в единицах потоковой передачи.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-180">For Stream Analytics, the computing resources allocated to a job are measured in Streaming Units.</span></span> <span data-ttu-id="d0e6a-181">Задания Stream Analytics лучше всего масштабировать, если оно может выполняться параллельно.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-181">Stream Analytics jobs scale best if the job can be parallelized.</span></span> <span data-ttu-id="d0e6a-182">Таким образом, Stream Analytics сможет распределять задания между несколькими вычислительными узлами.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-182">That way, Stream Analytics can distribute the job across multiple compute nodes.</span></span>

<span data-ttu-id="d0e6a-183">Для входных данных Центров событий используйте ключевое слово `PARTITION BY`, чтобы секционировать задание Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-183">For Event Hubs input, use the `PARTITION BY` keyword to partition the Stream Analytics job.</span></span> <span data-ttu-id="d0e6a-184">Данные будут разделены на подмножества на основе секций Центров событий.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-184">The data will be divided into subsets based on the Event Hubs partitions.</span></span> 

<span data-ttu-id="d0e6a-185">Для функций обработки методом окна и временных объединений требуются дополнительные единицы потоковой передачи.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-185">Windowing functions and temporal joins require additional SU.</span></span> <span data-ttu-id="d0e6a-186">По возможности используйте `PARTITION BY` так, чтобы каждая секция обрабатывалась отдельно.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-186">When possible, use `PARTITION BY` so that each partition is processed separately.</span></span> <span data-ttu-id="d0e6a-187">Дополнительные сведения см. в статье [Обзор и настройка единиц потоковой передачи](/azure/stream-analytics/stream-analytics-streaming-unit-consumption#windowed-aggregates).</span><span class="sxs-lookup"><span data-stu-id="d0e6a-187">For more information, see [Understand and adjust Streaming Units](/azure/stream-analytics/stream-analytics-streaming-unit-consumption#windowed-aggregates).</span></span>

<span data-ttu-id="d0e6a-188">Если невозможно параллелизовать все задание Stream Analytics, попробуйте разбить его на несколько шагов, начиная с одного или нескольких параллельных шагов.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-188">If it's not possible to parallelize the entire Stream Analytics job, try to break the job into multiple steps, starting with one or more parallel steps.</span></span> <span data-ttu-id="d0e6a-189">Так первые шаги можно будет выполнить параллельно.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-189">That way, the first steps can run in parallel.</span></span> <span data-ttu-id="d0e6a-190">Например, как в этой эталонной архитектуре:</span><span class="sxs-lookup"><span data-stu-id="d0e6a-190">For example, in this reference architecture:</span></span>

- <span data-ttu-id="d0e6a-191">Шаги 1 и 2 — это простые инструкции `SELECT`, которые выбирают записи из одной секции.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-191">Steps 1 and 2 are simple `SELECT` statements that select records within a single partition.</span></span> 
- <span data-ttu-id="d0e6a-192">На шаге 3 выполняется секционированное объединение двух входных потоков.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-192">Step 3 performs a partitioned join across two input streams.</span></span> <span data-ttu-id="d0e6a-193">На этом шаге используется преимущество, заключающееся в том, что совпадающие записи имеют один и тот же ключ секции. Поэтому у них всегда будет одинаковый идентификатор секции в каждом входном потоке.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-193">This step takes advantage of the fact that matching records share the same partition key, and so are guaranteed to have the same partition ID in each input stream.</span></span>
- <span data-ttu-id="d0e6a-194">На шаге 4 выполняется статистическое вычисление для всех секций.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-194">Step 4 aggregates across all of the partitions.</span></span> <span data-ttu-id="d0e6a-195">Этот шаг нельзя выполнить параллельно.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-195">This step cannot be parallelized.</span></span>

<span data-ttu-id="d0e6a-196">Используйте [схему заданий](/azure/stream-analytics/stream-analytics-job-diagram-with-metrics) в Stream Analytics, чтобы просмотреть, сколько секций назначено каждому шагу в задании.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-196">Use the Stream Analytics [job diagram](/azure/stream-analytics/stream-analytics-job-diagram-with-metrics) to see how many partitions are assigned to each step in the job.</span></span> <span data-ttu-id="d0e6a-197">Ниже показана схема заданий для этой эталонной архитектуры:</span><span class="sxs-lookup"><span data-stu-id="d0e6a-197">The following diagram shows the job diagram for this reference architecture:</span></span>

![](./images/stream-processing-asa/job-diagram.png)

### <a name="cosmos-db"></a><span data-ttu-id="d0e6a-198">База данных Cosmos</span><span class="sxs-lookup"><span data-stu-id="d0e6a-198">Cosmos DB</span></span>

<span data-ttu-id="d0e6a-199">Пропускная способность для Cosmos DB измеряется в [единицах запроса](/azure/cosmos-db/request-units) (RU).</span><span class="sxs-lookup"><span data-stu-id="d0e6a-199">Throughput capacity for Cosmos DB is measured in [Request Units](/azure/cosmos-db/request-units) (RU).</span></span> <span data-ttu-id="d0e6a-200">Чтобы масштабировать контейнер Cosmos DB на более чем 10 000 единиц запросов, необходимо указать [ключ секции](/azure/cosmos-db/partition-data) при создании контейнера и добавить этот ключ в каждый документ.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-200">In order to scale a Cosmos DB container past 10,000 RU, you must specify a [partition key](/azure/cosmos-db/partition-data) when you create the container, and include the partition key in every document.</span></span> 

<span data-ttu-id="d0e6a-201">В этой эталонной архитектуре новые документы создаются только раз в минуту (интервал "прыгающего" окна), поэтому требования к пропускной способности довольно низкие.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-201">In this reference architecture, new documents are created only once per minute (the hopping window interval), so the throughput requirements are quite low.</span></span> <span data-ttu-id="d0e6a-202">По этой причине нет необходимости назначать ключ секции в этом сценарии.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-202">For that reason, there's no need to assign a partition key in this scenario.</span></span>

## <a name="monitoring-considerations"></a><span data-ttu-id="d0e6a-203">Рекомендации по мониторингу</span><span class="sxs-lookup"><span data-stu-id="d0e6a-203">Monitoring considerations</span></span>

<span data-ttu-id="d0e6a-204">Для любого решения обработки потоков данных очень важно отслеживать производительность и работоспособность системы.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-204">With any stream processing solution, it's important to monitor the performance and health of the system.</span></span> <span data-ttu-id="d0e6a-205">[Azure Monitor](/azure/monitoring-and-diagnostics/) собирает журналы метрик и диагностики для служб Azure, используемых в этой архитектуре.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-205">[Azure Monitor](/azure/monitoring-and-diagnostics/) collects metrics and diagnostics logs for the Azure services used in the architecture.</span></span> <span data-ttu-id="d0e6a-206">Служба Azure Monitor встроена в платформу Azure и не требует написания дополнительного кода в приложении.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-206">Azure Monitor is built into the Azure platform and does not require any additional code in your application.</span></span>

<span data-ttu-id="d0e6a-207">Любой из следующих сигналов предупреждения указывает на то, что нужно масштабировать соответствующий ресурс Azure:</span><span class="sxs-lookup"><span data-stu-id="d0e6a-207">Any of the following warning signals indicate that you should scale out the relevant Azure resource:</span></span>

- <span data-ttu-id="d0e6a-208">Служба "Центры событий" регулирует запросы или указывает на то, что показатели близки к превышению дневной квоты сообщений.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-208">Event Hubs throttles requests or is close to the daily message quota.</span></span>
- <span data-ttu-id="d0e6a-209">Задание Stream Analytics постоянно использует более 80 % выделенных единиц потоковой передачи.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-209">The Stream Analytics job consistently uses more than 80% of allocated Streaming Units (SU).</span></span>
- <span data-ttu-id="d0e6a-210">Cosmos DB начинает регулировать запросы.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-210">Cosmos DB begins to throttle requests.</span></span>

<span data-ttu-id="d0e6a-211">Эталонная архитектура включает пользовательскую панель мониторинга, которая развертывается на портале Azure.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-211">The reference architecture includes a custom dashboard, which is deployed to the Azure portal.</span></span> <span data-ttu-id="d0e6a-212">Когда архитектура будет развернута, вы сможете просмотреть панель мониторинга, открыв [портал Azure](https://portal.azure.com) и выбрав `TaxiRidesDashboard` в списке панелей мониторинга.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-212">After you deploy the architecture, you can view the dashboard by opening the [Azure Portal](https://portal.azure.com) and selecting `TaxiRidesDashboard` from list of dashboards.</span></span> <span data-ttu-id="d0e6a-213">Дополнительные сведения о создании и развертывании пользовательских панелей мониторинга см. в статье [Создание панелей мониторинга Azure программными средствами](/azure/azure-portal/azure-portal-dashboards-create-programmatically).</span><span class="sxs-lookup"><span data-stu-id="d0e6a-213">For more information about creating and deploying custom dashboards in the Azure portal, see [Programmatically create Azure Dashboards](/azure/azure-portal/azure-portal-dashboards-create-programmatically).</span></span>

<span data-ttu-id="d0e6a-214">На следующем изображении показана панель мониторинга приблизительно через час после запуска задания Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-214">The following image shows the dashboard after the Stream Analytics job ran for about an hour.</span></span>

![](./images/stream-processing-asa/asa-dashboard.png)

<span data-ttu-id="d0e6a-215">На внизу слева показано, что использование единиц потоковой передачи для задания Stream Analytics повышается в течение первых 15 минут и затем снижается.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-215">The panel on the lower left shows that the SU consumption for the Stream Analytics job climbs during the first 15 minutes and then levels off.</span></span> <span data-ttu-id="d0e6a-216">Это стандартное развитие событий, так как задание достигает устойчивого состояния.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-216">This is a typical pattern as the job reaches a steady state.</span></span> 

<span data-ttu-id="d0e6a-217">Обратите внимание на то, что служба "Центры событий" регулирует запросы, как показано вверху справа.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-217">Notice that Event Hubs is throttling requests, shown in the upper right panel.</span></span> <span data-ttu-id="d0e6a-218">Случайно отрегулированный запрос не является проблемой, так как клиентский пакет SDK службы "Центры событий" автоматически осуществляет новую попытку при получении ошибки регулирования.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-218">An occasional throttled request is not a problem, because the Event Hubs client SDK automatically retries when it receives a throttling error.</span></span> <span data-ttu-id="d0e6a-219">Но если ошибки регулирования повторяются, значит службе требуются дополнительные единицы пропускной способности.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-219">However, if you see consistent throttling errors, it means the event hub needs more throughput units.</span></span> <span data-ttu-id="d0e6a-220">На следующей диаграмме показан тестовый запуск с использованием функции автоматического расширения в службе "Центры событий", которая автоматически масштабирует единицы пропускной способности при необходимости.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-220">The following graph shows a test run using the Event Hubs auto-inflate feature, which automatically scales out the throughput units as needed.</span></span> 

![](./images/stream-processing-asa/stream-processing-eh-autoscale.png)

<span data-ttu-id="d0e6a-221">Функция автоматического расширения включена приблизительно в 06:35.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-221">Auto-inflate was enabled at about the 06:35 mark.</span></span> <span data-ttu-id="d0e6a-222">Вы можете заметить снижение числа пакетов в регулируемых запросах, так как служба "Центры событий" автоматически масштабируется до трех единиц пропускной способности.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-222">You can see the p drop in throttled requests, as Event Hubs automatically scaled up to 3 throughput units.</span></span>

<span data-ttu-id="d0e6a-223">Интересно, что это побочный эффект на увеличение использования единиц SU в задании Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-223">Interestingly, this had the side effect of increasing the SU utilization in the Stream Analytics job.</span></span> <span data-ttu-id="d0e6a-224">При регулировании в службе "Центры событий" искусственно снизилась скорость приема данных для задания Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-224">By throttling, Event Hubs was artificially reducing the ingestion rate for the Stream Analytics job.</span></span> <span data-ttu-id="d0e6a-225">Довольно часто бывает так, что при устранении одной проблемы с производительностью возникает другая.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-225">It's actually common that resolving one performance bottleneck reveals another.</span></span> <span data-ttu-id="d0e6a-226">В таком случае проблему можно решить, выделив дополнительные единицы потоковой передачи для задания Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-226">In this case, allocating additional SU for the Stream Analytics job resolved the issue.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="d0e6a-227">Развертывание решения</span><span class="sxs-lookup"><span data-stu-id="d0e6a-227">Deploy the solution</span></span>

<span data-ttu-id="d0e6a-228">Пример развертывания для этой архитектуры можно найти на портале [GitHub](https://github.com/mspnp/reference-architectures/tree/master/data).</span><span class="sxs-lookup"><span data-stu-id="d0e6a-228">A deployment for this reference architecture is available on [GitHub](https://github.com/mspnp/reference-architectures/tree/master/data).</span></span> 

### <a name="prerequisites"></a><span data-ttu-id="d0e6a-229">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="d0e6a-229">Prerequisites</span></span>

1. <span data-ttu-id="d0e6a-230">Клонируйте или скачайте ZIP-файл [с эталонными архитектурами](https://github.com/mspnp/reference-architectures) в репозитории GitHub либо создайте для него вилку.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-230">Clone, fork, or download the zip file for the [reference architectures](https://github.com/mspnp/reference-architectures) GitHub repository.</span></span>

2. <span data-ttu-id="d0e6a-231">Установите [Docker](https://www.docker.com/), чтобы запустить генератор данных.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-231">Install [Docker](https://www.docker.com/) to run the data generator.</span></span>

3. <span data-ttu-id="d0e6a-232">Установите [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span><span class="sxs-lookup"><span data-stu-id="d0e6a-232">Install [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span></span>

4. <span data-ttu-id="d0e6a-233">Из командной строки, строки bash или строки PowerShell войдите в свою учетную запись Azure, как показано ниже:</span><span class="sxs-lookup"><span data-stu-id="d0e6a-233">From a command prompt, bash prompt, or PowerShell prompt, sign into your Azure account as follows:</span></span>

    ```
    az login
    ```

### <a name="download-the-source-data-files"></a><span data-ttu-id="d0e6a-234">Скачивание файлов с исходными данными</span><span class="sxs-lookup"><span data-stu-id="d0e6a-234">Download the source data files</span></span>

1. <span data-ttu-id="d0e6a-235">Создайте каталог с именем `DataFile` в каталоге `data/streaming_asa` репозитория GitHub.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-235">Create a directory named `DataFile` under the `data/streaming_asa` directory in the GitHub repo.</span></span>

2. <span data-ttu-id="d0e6a-236">Откройте браузер и перейдите по ссылке https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-236">Open a web browser and navigate to https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.</span></span>

3. <span data-ttu-id="d0e6a-237">На этой странице нажмите кнопку **Скачать**, чтобы скачать ZIP-файл со всеми данные о поездках в такси за этот год.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-237">Click the **Download** button on this page to download a zip file of all the taxi data for that year.</span></span>

4. <span data-ttu-id="d0e6a-238">Извлеките ZIP-файл в каталог `DataFile`.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-238">Extract the zip file to the `DataFile` directory.</span></span>

    > [!NOTE]
    > <span data-ttu-id="d0e6a-239">Этот ZIP-файл содержит другие ZIP-файлы.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-239">This zip file contains other zip files.</span></span> <span data-ttu-id="d0e6a-240">Не извлекайте дочерние ZIP-файлы.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-240">Don't extract the child zip files.</span></span>

<span data-ttu-id="d0e6a-241">Структура каталогов должна выглядеть так:</span><span class="sxs-lookup"><span data-stu-id="d0e6a-241">The directory structure should look like the following:</span></span>

```
/data
    /streaming_asa
        /DataFile
            /FOIL2013
                trip_data_1.zip
                trip_data_2.zip
                trip_data_3.zip
                ...
```

### <a name="deploy-the-azure-resources"></a><span data-ttu-id="d0e6a-242">Развертывание ресурсов Azure</span><span class="sxs-lookup"><span data-stu-id="d0e6a-242">Deploy the Azure resources</span></span>

1. <span data-ttu-id="d0e6a-243">В оболочке или командной строке Windows выполните следующую команду и следуйте указаниям при запросе на вход:</span><span class="sxs-lookup"><span data-stu-id="d0e6a-243">From a shell or Windows Command Prompt, run the following command and follow the sign-in prompt:</span></span>

    ```bash
    az login
    ```

2. <span data-ttu-id="d0e6a-244">Перейдите в папку `data/streaming_asa` в репозитории GitHub.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-244">Navigate to the folder `data/streaming_asa` in the GitHub repository</span></span>

    ```bash
    cd data/streaming_asa
    ```

2. <span data-ttu-id="d0e6a-245">Выполните следующие команды, чтобы развернуть ресурсы Azure:</span><span class="sxs-lookup"><span data-stu-id="d0e6a-245">Run the following commands to deploy the Azure resources:</span></span>

    ```bash
    export resourceGroup='[Resource group name]'
    export resourceLocation='[Location]'
    export cosmosDatabaseAccount='[Cosmos DB account name]'
    export cosmosDatabase='[Cosmod DB database name]'
    export cosmosDataBaseCollection='[Cosmos DB collection name]'
    export eventHubNamespace='[Event Hubs namespace name]'

    # Create a resource group
    az group create --name $resourceGroup --location $resourceLocation

    # Deploy resources
    az group deployment create --resource-group $resourceGroup \
      --template-file ./azure/deployresources.json --parameters \
      eventHubNamespace=$eventHubNamespace \
      outputCosmosDatabaseAccount=$cosmosDatabaseAccount \
      outputCosmosDatabase=$cosmosDatabase \
      outputCosmosDatabaseCollection=$cosmosDataBaseCollection

    # Create a database 
    az cosmosdb database create --name $cosmosDatabaseAccount \
        --db-name $cosmosDatabase --resource-group $resourceGroup

    # Create a collection
    az cosmosdb collection create --collection-name $cosmosDataBaseCollection \
        --name $cosmosDatabaseAccount --db-name $cosmosDatabase \
        --resource-group $resourceGroup
    ```

3. <span data-ttu-id="d0e6a-246">На портале Azure перейдите к созданной группе ресурсов.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-246">In the Azure portal, navigate to the resource group that was created.</span></span>

4. <span data-ttu-id="d0e6a-247">Откройте колонку для задания Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-247">Open the blade for the Stream Analytics job.</span></span>

5. <span data-ttu-id="d0e6a-248">Щелкните **Пуск**, чтобы начать задание.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-248">Click **Start** to start the job.</span></span> <span data-ttu-id="d0e6a-249">Выберите **Сейчас** в качестве времени начала создания выходных данных.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-249">Select **Now** as the output start time.</span></span> <span data-ttu-id="d0e6a-250">Дождитесь запуска задания.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-250">Wait for the job to start.</span></span>

### <a name="run-the-data-generator"></a><span data-ttu-id="d0e6a-251">Запуск генератора данных</span><span class="sxs-lookup"><span data-stu-id="d0e6a-251">Run the data generator</span></span>

1. <span data-ttu-id="d0e6a-252">Получите строки подключения для концентратора событий.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-252">Get the Event Hub connection strings.</span></span> <span data-ttu-id="d0e6a-253">Их можно получить на портале Azure или с помощью следующих команд CLI:</span><span class="sxs-lookup"><span data-stu-id="d0e6a-253">You can get these from the Azure portal, or by running the following CLI commands:</span></span>

    ```bash
    # RIDE_EVENT_HUB
    az eventhubs eventhub authorization-rule keys list \
        --eventhub-name taxi-ride \
        --name taxi-ride-asa-access-policy \
        --namespace-name $eventHubNamespace \
        --resource-group $resourceGroup \
        --query primaryConnectionString

    # FARE_EVENT_HUB
    az eventhubs eventhub authorization-rule keys list \
        --eventhub-name taxi-fare \
        --name taxi-fare-asa-access-policy \
        --namespace-name $eventHubNamespace \
        --resource-group $resourceGroup \
        --query primaryConnectionString
    ```

2. <span data-ttu-id="d0e6a-254">Перейдите в каталог `data/streaming_asa/onprem` в репозитории GitHub.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-254">Navigate to the directory `data/streaming_asa/onprem` in the GitHub repository</span></span>

3. <span data-ttu-id="d0e6a-255">Обновите значения в файле `main.env` следующим образом:</span><span class="sxs-lookup"><span data-stu-id="d0e6a-255">Update the values in the file `main.env` as follows:</span></span>

    ```
    RIDE_EVENT_HUB=[Connection string for taxi-ride event hub]
    FARE_EVENT_HUB=[Connection string for taxi-fare event hub]
    RIDE_DATA_FILE_PATH=/DataFile/FOIL2013
    MINUTES_TO_LEAD=0
    PUSH_RIDE_DATA_FIRST=false
    ```

4. <span data-ttu-id="d0e6a-256">Чтобы создать образ Docker, выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="d0e6a-256">Run the following command to build the Docker image.</span></span>

    ```bash
    docker build --no-cache -t dataloader .
    ```

5. <span data-ttu-id="d0e6a-257">Вернитесь к родительскому каталогу `data/stream_asa`.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-257">Navigate back to the parent directory, `data/stream_asa`.</span></span>

    ```bash
    cd ..
    ```

6. <span data-ttu-id="d0e6a-258">Чтобы запустить образ Docker, выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="d0e6a-258">Run the following command to run the Docker image.</span></span>

    ```bash
    docker run -v `pwd`/DataFile:/DataFile --env-file=onprem/main.env dataloader:latest
    ```

<span data-ttu-id="d0e6a-259">Полученный результат должен выглядеть примерно так:</span><span class="sxs-lookup"><span data-stu-id="d0e6a-259">The output should look like the following:</span></span>

```
Created 10000 records for TaxiFare
Created 10000 records for TaxiRide
Created 20000 records for TaxiFare
Created 20000 records for TaxiRide
Created 30000 records for TaxiFare
...
```

<span data-ttu-id="d0e6a-260">Оставьте программу запущенной по крайней мере на пять минут. Это — окно, определенное в запросе Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-260">Let the program run for at least 5 minutes, which is the window defined in the Stream Analytics query.</span></span> <span data-ttu-id="d0e6a-261">Чтобы проверить, правильно ли выполняется задание Stream Analytics, откройте портал Azure и перейдите к базе данных Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-261">To verify the Stream Analytics job is running correctly, open the Azure portal and navigate to the Cosmos DB database.</span></span> <span data-ttu-id="d0e6a-262">Откройте колонку **Обозреватель данных** и просмотрите документы.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-262">Open the **Data Explorer** blade and view the documents.</span></span> 

<span data-ttu-id="d0e6a-263">[1] <span id="note1">Брайан Донован (Brian Donovan); Дэн Уорк (Dan Work), 2016 г.: New York City Taxi Trip Data (2010–2013) (Данные о поездках в такси в Нью-Йорке).</span><span class="sxs-lookup"><span data-stu-id="d0e6a-263">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span></span> <span data-ttu-id="d0e6a-264">Иллинойсский университет в Урбане-Шампейне.</span><span class="sxs-lookup"><span data-stu-id="d0e6a-264">University of Illinois at Urbana-Champaign.</span></span> <span data-ttu-id="d0e6a-265">https://doi.org/10.13012/J8PN93H8</span><span class="sxs-lookup"><span data-stu-id="d0e6a-265">https://doi.org/10.13012/J8PN93H8</span></span>
