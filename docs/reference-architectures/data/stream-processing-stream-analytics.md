---
title: Обработка потоков данных с помощью Azure Stream Analytics
titleSuffix: Azure Reference Architectures
description: Создание сквозного конвейера обработки потоков данных в Azure.
author: MikeWasson
ms.date: 11/06/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18
ms.openlocfilehash: af56a010e90fb176b13c1110ad4000dcababe009
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243875"
---
# <a name="create-a-stream-processing-pipeline-with-azure-stream-analytics"></a><span data-ttu-id="4d821-103">Создание конвейера обработки потоков данных с помощью Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="4d821-103">Create a stream processing pipeline with Azure Stream Analytics</span></span>

<span data-ttu-id="4d821-104">На схеме эталонной архитектуры представлен сквозной конвейер [обработки потоков данных](/azure/architecture/data-guide/big-data/real-time-processing).</span><span class="sxs-lookup"><span data-stu-id="4d821-104">This reference architecture shows an end-to-end [stream processing](/azure/architecture/data-guide/big-data/real-time-processing) pipeline.</span></span> <span data-ttu-id="4d821-105">Конвейер принимает данные из двух источников, сопоставляет записи в двух потоках и вычисляет скользящее среднее за определенный промежуток времени.</span><span class="sxs-lookup"><span data-stu-id="4d821-105">The pipeline ingests data from two sources, correlates records in the two streams, and calculates a rolling average across a time window.</span></span> <span data-ttu-id="4d821-106">Результаты сохраняются для дальнейшего анализа.</span><span class="sxs-lookup"><span data-stu-id="4d821-106">The results are stored for further analysis.</span></span>

<span data-ttu-id="4d821-107">Эталонную реализацию для этой архитектуры можно найти на сайте [GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="4d821-107">A reference implementation for this architecture is available on [GitHub][github].</span></span>

![Эталонная архитектура для создания конвейера обработки потока с использованием Azure Stream Analytics](./images/stream-processing-asa/stream-processing-asa.png)

<span data-ttu-id="4d821-109">**Сценарий.** Компания, предоставляющая услуги такси, собирает данные о каждой поездке.</span><span class="sxs-lookup"><span data-stu-id="4d821-109">**Scenario**: A taxi company collects data about each taxi trip.</span></span> <span data-ttu-id="4d821-110">В этом сценарии предполагается, что данные отправляются с двух отдельных устройств.</span><span class="sxs-lookup"><span data-stu-id="4d821-110">For this scenario, we assume there are two separate devices sending data.</span></span> <span data-ttu-id="4d821-111">В такси установлен счетчик, который отправляет данные о каждой поездке, включая сведения о продолжительности, расстоянии, а также местах посадки и высадки.</span><span class="sxs-lookup"><span data-stu-id="4d821-111">The taxi has a meter that sends information about each ride &mdash; the duration, distance, and pickup and dropoff locations.</span></span> <span data-ttu-id="4d821-112">Отдельное устройство принимает платежи от клиентов и отправляет данные о тарифах.</span><span class="sxs-lookup"><span data-stu-id="4d821-112">A separate device accepts payments from customers and sends data about fares.</span></span> <span data-ttu-id="4d821-113">Компании необходимо вычислить среднюю сумму чаевых на милю в реальном времени, чтобы определить тенденции.</span><span class="sxs-lookup"><span data-stu-id="4d821-113">The taxi company wants to calculate the average tip per mile driven, in real time, in order to spot trends.</span></span>

## <a name="architecture"></a><span data-ttu-id="4d821-114">Архитектура</span><span class="sxs-lookup"><span data-stu-id="4d821-114">Architecture</span></span>

<span data-ttu-id="4d821-115">Архитектура состоит из следующих компонентов:</span><span class="sxs-lookup"><span data-stu-id="4d821-115">The architecture consists of the following components.</span></span>

<span data-ttu-id="4d821-116">**Источники данных.**</span><span class="sxs-lookup"><span data-stu-id="4d821-116">**Data sources**.</span></span> <span data-ttu-id="4d821-117">В этой архитектуре существует два источника данных, которые создают потоки данных в реальном времени.</span><span class="sxs-lookup"><span data-stu-id="4d821-117">In this architecture, there are two data sources that generate data streams in real time.</span></span> <span data-ttu-id="4d821-118">Первый поток содержит сведения о поездке, а второй — о тарифах.</span><span class="sxs-lookup"><span data-stu-id="4d821-118">The first stream contains ride information, and the second contains fare information.</span></span> <span data-ttu-id="4d821-119">В эталонной архитектуре есть имитированный генератор данных, который считывает данные из набора статических файлов и отправляет данные в Центры событий.</span><span class="sxs-lookup"><span data-stu-id="4d821-119">The reference architecture includes a simulated data generator that reads from a set of static files and pushes the data to Event Hubs.</span></span> <span data-ttu-id="4d821-120">В реальном приложении источниками данных будут устройства, установленные в такси.</span><span class="sxs-lookup"><span data-stu-id="4d821-120">In a real application, the data sources would be devices installed in the taxi cabs.</span></span>

<span data-ttu-id="4d821-121">**Центры событий Azure**.</span><span class="sxs-lookup"><span data-stu-id="4d821-121">**Azure Event Hubs**.</span></span> <span data-ttu-id="4d821-122">[Центры событий](/azure/event-hubs/) — это служба приема событий.</span><span class="sxs-lookup"><span data-stu-id="4d821-122">[Event Hubs](/azure/event-hubs/) is an event ingestion service.</span></span> <span data-ttu-id="4d821-123">В этой архитектуре используется два экземпляра службы — по одной на каждый источник данных.</span><span class="sxs-lookup"><span data-stu-id="4d821-123">This architecture uses two event hub instances, one for each data source.</span></span> <span data-ttu-id="4d821-124">Каждый источник данных отправляет поток данных в соответствующую службу.</span><span class="sxs-lookup"><span data-stu-id="4d821-124">Each data source sends a stream of data to the associated event hub.</span></span>

<span data-ttu-id="4d821-125">**Azure Stream Analytics**.</span><span class="sxs-lookup"><span data-stu-id="4d821-125">**Azure Stream Analytics**.</span></span> <span data-ttu-id="4d821-126">[Stream Analytics](/azure/stream-analytics/) — это модуль обработки событий.</span><span class="sxs-lookup"><span data-stu-id="4d821-126">[Stream Analytics](/azure/stream-analytics/) is an event-processing engine.</span></span> <span data-ttu-id="4d821-127">Задание Stream Analytics считывает потоки данных из двух экземпляров и обрабатывает эти потоки.</span><span class="sxs-lookup"><span data-stu-id="4d821-127">A Stream Analytics job reads the data streams from the two event hubs and performs stream processing.</span></span>

<span data-ttu-id="4d821-128">**Cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="4d821-128">**Cosmos DB**.</span></span> <span data-ttu-id="4d821-129">Выходные данные задания Stream Analytics — это набор записей, которые регистрируются в виде документов JSON в базе данных документов Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="4d821-129">The output from the Stream Analytics job is a series of records, which are written as JSON documents to a Cosmos DB document database.</span></span>

<span data-ttu-id="4d821-130">**Microsoft Power BI**.</span><span class="sxs-lookup"><span data-stu-id="4d821-130">**Microsoft Power BI**.</span></span> <span data-ttu-id="4d821-131">Power BI — набор средств бизнес-аналитики для анализа информации о бизнесе.</span><span class="sxs-lookup"><span data-stu-id="4d821-131">Power BI is a suite of business analytics tools to analyze data for business insights.</span></span> <span data-ttu-id="4d821-132">В нашей архитектуре данные загружаются из Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="4d821-132">In this architecture, it loads the data from Cosmos DB.</span></span> <span data-ttu-id="4d821-133">Благодаря этому пользователи могут выполнять анализ полного набора собранных исторических данных.</span><span class="sxs-lookup"><span data-stu-id="4d821-133">This allows users to analyze the complete set of historical data that's been collected.</span></span> <span data-ttu-id="4d821-134">Также результаты можно передать в виде потока прямо из Stream Analytics в Power BI, чтобы просмотреть данные в реальном времени.</span><span class="sxs-lookup"><span data-stu-id="4d821-134">You could also stream the results directly from Stream Analytics to Power BI for a real-time view of the data.</span></span> <span data-ttu-id="4d821-135">Дополнительные сведения см. в статье [Потоковая передача в реальном времени в Power BI](/power-bi/service-real-time-streaming).</span><span class="sxs-lookup"><span data-stu-id="4d821-135">For more information, see [Real-time streaming in Power BI](/power-bi/service-real-time-streaming).</span></span>

<span data-ttu-id="4d821-136">**Azure Monitor**.</span><span class="sxs-lookup"><span data-stu-id="4d821-136">**Azure Monitor**.</span></span> <span data-ttu-id="4d821-137">[Azure Monitor](/azure/monitoring-and-diagnostics/) собирает метрики производительности о службах Azure, развернутых в решении.</span><span class="sxs-lookup"><span data-stu-id="4d821-137">[Azure Monitor](/azure/monitoring-and-diagnostics/) collects performance metrics about the Azure services deployed in the solution.</span></span> <span data-ttu-id="4d821-138">Отобразив эти данные в визуализации на панели мониторинга, можно получить полезные сведения о работоспособности решения.</span><span class="sxs-lookup"><span data-stu-id="4d821-138">By visualizing these in a dashboard, you can get insights into the health of the solution.</span></span>

## <a name="data-ingestion"></a><span data-ttu-id="4d821-139">Прием данных</span><span class="sxs-lookup"><span data-stu-id="4d821-139">Data ingestion</span></span>

<!-- markdownlint-disable MD033 -->

<span data-ttu-id="4d821-140">Для имитации источника данных в этой эталонной архитектуре используется набор данных<sup>[[1]](#note1)</sup> [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) (Данные о поездках в такси в Нью-Йорке).</span><span class="sxs-lookup"><span data-stu-id="4d821-140">To simulate a data source, this reference architecture uses the [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) dataset<sup>[[1]](#note1)</sup>.</span></span> <span data-ttu-id="4d821-141">Этот набор содержит данные о поездках в такси в Нью-Йорке за 4 года (2010&ndash;2013).</span><span class="sxs-lookup"><span data-stu-id="4d821-141">This dataset contains data about taxi trips in New York City over a 4-year period (2010 &ndash; 2013).</span></span> <span data-ttu-id="4d821-142">Он содержит два типа записей: данные о поездке и данные о тарифе.</span><span class="sxs-lookup"><span data-stu-id="4d821-142">It contains two types of record: Ride data and fare data.</span></span> <span data-ttu-id="4d821-143">Данные о поездках включают сведения о продолжительности поездки, расстоянии, а также местах посадки и высадки.</span><span class="sxs-lookup"><span data-stu-id="4d821-143">Ride data includes trip duration, trip distance, and pickup and dropoff location.</span></span> <span data-ttu-id="4d821-144">Данные о тарифах включают сведения о тарифе, налоге и сумме чаевых.</span><span class="sxs-lookup"><span data-stu-id="4d821-144">Fare data includes fare, tax, and tip amounts.</span></span> <span data-ttu-id="4d821-145">В обоих типах записей есть стандартные поля: номер медальона, лицензия на право вождения и код организации.</span><span class="sxs-lookup"><span data-stu-id="4d821-145">Common fields in both record types include medallion number, hack license, and vendor ID.</span></span> <span data-ttu-id="4d821-146">Вместе эти три поля позволяют уникально идентифицировать такси и водителя.</span><span class="sxs-lookup"><span data-stu-id="4d821-146">Together these three fields uniquely identify a taxi plus a driver.</span></span> <span data-ttu-id="4d821-147">Данные хранятся в формате CSV.</span><span class="sxs-lookup"><span data-stu-id="4d821-147">The data is stored in CSV format.</span></span>

<span data-ttu-id="4d821-148"><span id="note1">[1]</span> Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013) (Брайан Донован, Дэн Уорк, 2016. Данные о поездках в такси по Нью-Йорку за 2010–2013 гг.).</span><span class="sxs-lookup"><span data-stu-id="4d821-148"><span id="note1">[1]</span> Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span></span> <span data-ttu-id="4d821-149">Иллинойсский университет в Урбане-Шампейне.</span><span class="sxs-lookup"><span data-stu-id="4d821-149">University of Illinois at Urbana-Champaign.</span></span> <span data-ttu-id="4d821-150">https://doi.org/10.13012/J8PN93H8</span><span class="sxs-lookup"><span data-stu-id="4d821-150">https://doi.org/10.13012/J8PN93H8</span></span>

<!-- markdownlint-enable MD033 -->

<span data-ttu-id="4d821-151">Генератор данных — это приложение .NET Core, которое считывает записи и отправляет их в Центры событий Azure.</span><span class="sxs-lookup"><span data-stu-id="4d821-151">The data generator is a .NET Core application that reads the records and sends them to Azure Event Hubs.</span></span> <span data-ttu-id="4d821-152">Генератор отправляет данные о поездке в формате JSON, а данные о тарифах — в формате CSV.</span><span class="sxs-lookup"><span data-stu-id="4d821-152">The generator sends ride data in JSON format and fare data in CSV format.</span></span>

<span data-ttu-id="4d821-153">Для сегментации данных Центры событий используют [секции](/azure/event-hubs/event-hubs-features#partitions).</span><span class="sxs-lookup"><span data-stu-id="4d821-153">Event Hubs uses [partitions](/azure/event-hubs/event-hubs-features#partitions) to segment the data.</span></span> <span data-ttu-id="4d821-154">Они позволяют объекту-получателю считывать данные каждой секции параллельно.</span><span class="sxs-lookup"><span data-stu-id="4d821-154">Partitions allow a consumer to read each partition in parallel.</span></span> <span data-ttu-id="4d821-155">При отправке данных в Центры событий можно явно указать ключ секции.</span><span class="sxs-lookup"><span data-stu-id="4d821-155">When you send data to Event Hubs, you can specify the partition key explicitly.</span></span> <span data-ttu-id="4d821-156">В противном случае записи назначаются секциям методом циклического перебора.</span><span class="sxs-lookup"><span data-stu-id="4d821-156">Otherwise, records are assigned to partitions in round-robin fashion.</span></span>

<span data-ttu-id="4d821-157">В нашем примере данные о поездках и тарифах должны в итоге иметь одинаковый идентификатор секции для определенного такси.</span><span class="sxs-lookup"><span data-stu-id="4d821-157">In this particular scenario, ride data and fare data should end up with the same partition ID for a given taxi cab.</span></span> <span data-ttu-id="4d821-158">Это позволит Stream Analytics применить определенную степень параллелизма при сопоставлении двух потоков.</span><span class="sxs-lookup"><span data-stu-id="4d821-158">This enables Stream Analytics to apply a degree of parallelism when it correlates the two streams.</span></span> <span data-ttu-id="4d821-159">Запись в секции *n* с данными о поездке будет соответствовать записи в секции *n* с данными о тарифах.</span><span class="sxs-lookup"><span data-stu-id="4d821-159">A record in partition *n* of the ride data will match a record in partition *n* of the fare data.</span></span>

![Схема потоковой обработки с помощью Azure Stream Analytics и Центров событий Azure](./images/stream-processing-asa/stream-processing-eh.png)

<span data-ttu-id="4d821-161">В генераторе данных общая модель данных для обоих типов записей имеет свойство `PartitionKey`, в котором объединены `Medallion`, `HackLicense` и `VendorId`.</span><span class="sxs-lookup"><span data-stu-id="4d821-161">In the data generator, the common data model for both record types has a `PartitionKey` property which is the concatenation of `Medallion`, `HackLicense`, and `VendorId`.</span></span>

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

<span data-ttu-id="4d821-162">Это свойство используется для явного предоставления ключа секции при отправке в Центры событий:</span><span class="sxs-lookup"><span data-stu-id="4d821-162">This property is used to provide an explicit partition key when sending to Event Hubs:</span></span>

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

## <a name="stream-processing"></a><span data-ttu-id="4d821-163">Потоковая обработка</span><span class="sxs-lookup"><span data-stu-id="4d821-163">Stream processing</span></span>

<span data-ttu-id="4d821-164">Задание обработки потока определяется с помощью SQL-запроса в несколько отдельных шагов.</span><span class="sxs-lookup"><span data-stu-id="4d821-164">The stream processing job is defined using a SQL query with several distinct steps.</span></span> <span data-ttu-id="4d821-165">На первых двух шагах просто выбираются записи из двух входных потоков.</span><span class="sxs-lookup"><span data-stu-id="4d821-165">The first two steps simply select records from the two input streams.</span></span>

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

<span data-ttu-id="4d821-166">На следующем шаге эти два входных потока объединяются для выбора совпадающих записей из каждого потока.</span><span class="sxs-lookup"><span data-stu-id="4d821-166">The next step joins the two input streams to select matching records from each stream.</span></span>

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

<span data-ttu-id="4d821-167">Этот запрос объединяет записи в набор полей, которые уникально идентифицируют совпадающие записи (Medallion, HackLicense, VendorId и PickupTime).</span><span class="sxs-lookup"><span data-stu-id="4d821-167">This query joins records on a set of fields that uniquely identify matching records (Medallion, HackLicense, VendorId, and PickupTime).</span></span> <span data-ttu-id="4d821-168">Инструкция `JOIN` также содержит идентификатор секции.</span><span class="sxs-lookup"><span data-stu-id="4d821-168">The `JOIN` statement also includes the partition ID.</span></span> <span data-ttu-id="4d821-169">Как уже упоминалось, преимущество состоит в том, что совпадающие записи всегда имеют одинаковый идентификатор секции в этом сценарии.</span><span class="sxs-lookup"><span data-stu-id="4d821-169">As mentioned, this takes advantage of the fact that matching records always have the same partition ID in this scenario.</span></span>

<span data-ttu-id="4d821-170">В Stream Analytics объединения являются *темпоральными*, то есть записи объединяются в пределах определенного временного окна.</span><span class="sxs-lookup"><span data-stu-id="4d821-170">In Stream Analytics, joins are *temporal*, meaning records are joined within a particular window of time.</span></span> <span data-ttu-id="4d821-171">В противном случае задание может бесконечно ожидать сопоставления.</span><span class="sxs-lookup"><span data-stu-id="4d821-171">Otherwise, the job might need to wait indefinitely for a match.</span></span> <span data-ttu-id="4d821-172">Функция [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/join-azure-stream-analytics) указывает, насколько две совпадающих записи могут быть разделены по времени для сопоставления.</span><span class="sxs-lookup"><span data-stu-id="4d821-172">The [DATEDIFF](https://msdn.microsoft.com/azure/stream-analytics/reference/join-azure-stream-analytics) function specifies how far two matching records can be separated in time for a match.</span></span>

<span data-ttu-id="4d821-173">На последнем шаге задания вычисляется средняя сумма чаевых на милю, сгруппированных по "прыгающему" окну в 5 минут.</span><span class="sxs-lookup"><span data-stu-id="4d821-173">The last step in the job computes the average tip per mile, grouped by a hopping window of 5 minutes.</span></span>

```sql
SELECT System.Timestamp AS WindowTime,
       SUM(tr.TipAmount) / SUM(tr.TripDistanceInMiles) AS AverageTipPerMile
  INTO [TaxiDrain]
  FROM [Step3] tr
  GROUP BY HoppingWindow(Duration(minute, 5), Hop(minute, 1))
```

<span data-ttu-id="4d821-174">Stream Analytics предоставляет несколько [функций обработки методом окна](/azure/stream-analytics/stream-analytics-window-functions).</span><span class="sxs-lookup"><span data-stu-id="4d821-174">Stream Analytics provides several [windowing functions](/azure/stream-analytics/stream-analytics-window-functions).</span></span> <span data-ttu-id="4d821-175">"Прыгающее" окно перемещается вперед во времени в рамках фиксированного периода, в нашем примере — на 1 минуту за прыжок.</span><span class="sxs-lookup"><span data-stu-id="4d821-175">A hopping window moves forward in time by a fixed period, in this case 1 minute per hop.</span></span> <span data-ttu-id="4d821-176">Результатом будет вычисление скользящего среднего за последние 5 минут.</span><span class="sxs-lookup"><span data-stu-id="4d821-176">The result is to calculate a moving average over the past 5 minutes.</span></span>

<span data-ttu-id="4d821-177">В показанной здесь архитектуре в Cosmos DB сохраняются только результаты задания Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="4d821-177">In the architecture shown here, only the results of the Stream Analytics job are saved to Cosmos DB.</span></span> <span data-ttu-id="4d821-178">Для сценария с большими данными используйте также функцию [Сбор](/azure/event-hubs/event-hubs-capture-overview) в Центрах событий, чтобы сохранять необработанные данные о событиях в хранилище BLOB-объектов Azure.</span><span class="sxs-lookup"><span data-stu-id="4d821-178">For a big data scenario, consider also using [Event Hubs Capture](/azure/event-hubs/event-hubs-capture-overview) to save the raw event data into Azure Blob storage.</span></span> <span data-ttu-id="4d821-179">Сохранение необработанных данных позволит вам в дальнейшем выполнять пакетные запросы к данным журнала, чтобы получить новые полезные сведения из этих данных.</span><span class="sxs-lookup"><span data-stu-id="4d821-179">Keeping the raw data will allow you to run batch queries over your historical data at later time, in order to derive new insights from the data.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="4d821-180">Вопросы масштабируемости</span><span class="sxs-lookup"><span data-stu-id="4d821-180">Scalability considerations</span></span>

### <a name="event-hubs"></a><span data-ttu-id="4d821-181">Центры событий;</span><span class="sxs-lookup"><span data-stu-id="4d821-181">Event Hubs</span></span>

<span data-ttu-id="4d821-182">Пропускная способность Центров событий вычисляется в [единицах пропускной способности](/azure/event-hubs/event-hubs-features#throughput-units).</span><span class="sxs-lookup"><span data-stu-id="4d821-182">The throughput capacity of Event Hubs is measured in [throughput units](/azure/event-hubs/event-hubs-features#throughput-units).</span></span> <span data-ttu-id="4d821-183">Вы можете автоматически масштабировать концентратор событий, включив [автоматическое расширение](/azure/event-hubs/event-hubs-auto-inflate). Это позволит автоматически масштабировать единицы пропускной способности в зависимости от трафика вплоть до заданного максимума.</span><span class="sxs-lookup"><span data-stu-id="4d821-183">You can autoscale an event hub by enabling [auto-inflate](/azure/event-hubs/event-hubs-auto-inflate), which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span>

### <a name="stream-analytics"></a><span data-ttu-id="4d821-184">Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="4d821-184">Stream Analytics</span></span>

<span data-ttu-id="4d821-185">В Stream Analytics вычислительные ресурсы, выделенные для задания, измеряются в единицах потоковой передачи.</span><span class="sxs-lookup"><span data-stu-id="4d821-185">For Stream Analytics, the computing resources allocated to a job are measured in Streaming Units.</span></span> <span data-ttu-id="4d821-186">Задания Stream Analytics лучше всего масштабировать, если оно может выполняться параллельно.</span><span class="sxs-lookup"><span data-stu-id="4d821-186">Stream Analytics jobs scale best if the job can be parallelized.</span></span> <span data-ttu-id="4d821-187">Таким образом, Stream Analytics сможет распределять задания между несколькими вычислительными узлами.</span><span class="sxs-lookup"><span data-stu-id="4d821-187">That way, Stream Analytics can distribute the job across multiple compute nodes.</span></span>

<span data-ttu-id="4d821-188">Для входных данных Центров событий используйте ключевое слово `PARTITION BY`, чтобы секционировать задание Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="4d821-188">For Event Hubs input, use the `PARTITION BY` keyword to partition the Stream Analytics job.</span></span> <span data-ttu-id="4d821-189">Данные будут разделены на подмножества на основе секций Центров событий.</span><span class="sxs-lookup"><span data-stu-id="4d821-189">The data will be divided into subsets based on the Event Hubs partitions.</span></span>

<span data-ttu-id="4d821-190">Для функций обработки методом окна и временных объединений требуются дополнительные единицы потоковой передачи.</span><span class="sxs-lookup"><span data-stu-id="4d821-190">Windowing functions and temporal joins require additional SU.</span></span> <span data-ttu-id="4d821-191">По возможности используйте `PARTITION BY` так, чтобы каждая секция обрабатывалась отдельно.</span><span class="sxs-lookup"><span data-stu-id="4d821-191">When possible, use `PARTITION BY` so that each partition is processed separately.</span></span> <span data-ttu-id="4d821-192">Дополнительные сведения см. в статье [Обзор и настройка единиц потоковой передачи](/azure/stream-analytics/stream-analytics-streaming-unit-consumption#windowed-aggregates).</span><span class="sxs-lookup"><span data-stu-id="4d821-192">For more information, see [Understand and adjust Streaming Units](/azure/stream-analytics/stream-analytics-streaming-unit-consumption#windowed-aggregates).</span></span>

<span data-ttu-id="4d821-193">Если невозможно параллелизовать все задание Stream Analytics, попробуйте разбить его на несколько шагов, начиная с одного или нескольких параллельных шагов.</span><span class="sxs-lookup"><span data-stu-id="4d821-193">If it's not possible to parallelize the entire Stream Analytics job, try to break the job into multiple steps, starting with one or more parallel steps.</span></span> <span data-ttu-id="4d821-194">Так первые шаги можно будет выполнить параллельно.</span><span class="sxs-lookup"><span data-stu-id="4d821-194">That way, the first steps can run in parallel.</span></span> <span data-ttu-id="4d821-195">Например, как в этой эталонной архитектуре:</span><span class="sxs-lookup"><span data-stu-id="4d821-195">For example, in this reference architecture:</span></span>

- <span data-ttu-id="4d821-196">Шаги 1 и 2 — это простые инструкции `SELECT`, которые выбирают записи из одной секции.</span><span class="sxs-lookup"><span data-stu-id="4d821-196">Steps 1 and 2 are simple `SELECT` statements that select records within a single partition.</span></span>
- <span data-ttu-id="4d821-197">На шаге 3 выполняется секционированное объединение двух входных потоков.</span><span class="sxs-lookup"><span data-stu-id="4d821-197">Step 3 performs a partitioned join across two input streams.</span></span> <span data-ttu-id="4d821-198">На этом шаге используется преимущество, заключающееся в том, что совпадающие записи имеют один и тот же ключ секции. Поэтому у них всегда будет одинаковый идентификатор секции в каждом входном потоке.</span><span class="sxs-lookup"><span data-stu-id="4d821-198">This step takes advantage of the fact that matching records share the same partition key, and so are guaranteed to have the same partition ID in each input stream.</span></span>
- <span data-ttu-id="4d821-199">На шаге 4 выполняется статистическое вычисление для всех секций.</span><span class="sxs-lookup"><span data-stu-id="4d821-199">Step 4 aggregates across all of the partitions.</span></span> <span data-ttu-id="4d821-200">Этот шаг нельзя выполнить параллельно.</span><span class="sxs-lookup"><span data-stu-id="4d821-200">This step cannot be parallelized.</span></span>

<span data-ttu-id="4d821-201">Используйте [схему заданий](/azure/stream-analytics/stream-analytics-job-diagram-with-metrics) в Stream Analytics, чтобы просмотреть, сколько секций назначено каждому шагу в задании.</span><span class="sxs-lookup"><span data-stu-id="4d821-201">Use the Stream Analytics [job diagram](/azure/stream-analytics/stream-analytics-job-diagram-with-metrics) to see how many partitions are assigned to each step in the job.</span></span> <span data-ttu-id="4d821-202">Ниже показана схема заданий для этой эталонной архитектуры:</span><span class="sxs-lookup"><span data-stu-id="4d821-202">The following diagram shows the job diagram for this reference architecture:</span></span>

![Схема заданий](./images/stream-processing-asa/job-diagram.png)

### <a name="cosmos-db"></a><span data-ttu-id="4d821-204">База данных Cosmos</span><span class="sxs-lookup"><span data-stu-id="4d821-204">Cosmos DB</span></span>

<span data-ttu-id="4d821-205">Пропускная способность для Cosmos DB измеряется в [единицах запроса](/azure/cosmos-db/request-units) (RU).</span><span class="sxs-lookup"><span data-stu-id="4d821-205">Throughput capacity for Cosmos DB is measured in [Request Units](/azure/cosmos-db/request-units) (RU).</span></span> <span data-ttu-id="4d821-206">Чтобы масштабировать контейнер Cosmos DB на более чем 10 000 единиц запросов, необходимо указать [ключ секции](/azure/cosmos-db/partition-data) при создании контейнера и добавить этот ключ в каждый документ.</span><span class="sxs-lookup"><span data-stu-id="4d821-206">In order to scale a Cosmos DB container past 10,000 RU, you must specify a [partition key](/azure/cosmos-db/partition-data) when you create the container, and include the partition key in every document.</span></span>

<span data-ttu-id="4d821-207">В этой эталонной архитектуре новые документы создаются только раз в минуту (интервал "прыгающего" окна), поэтому требования к пропускной способности довольно низкие.</span><span class="sxs-lookup"><span data-stu-id="4d821-207">In this reference architecture, new documents are created only once per minute (the hopping window interval), so the throughput requirements are quite low.</span></span> <span data-ttu-id="4d821-208">По этой причине нет необходимости назначать ключ секции в этом сценарии.</span><span class="sxs-lookup"><span data-stu-id="4d821-208">For that reason, there's no need to assign a partition key in this scenario.</span></span>

## <a name="monitoring-considerations"></a><span data-ttu-id="4d821-209">Рекомендации по мониторингу</span><span class="sxs-lookup"><span data-stu-id="4d821-209">Monitoring considerations</span></span>

<span data-ttu-id="4d821-210">Для любого решения обработки потоков данных очень важно отслеживать производительность и работоспособность системы.</span><span class="sxs-lookup"><span data-stu-id="4d821-210">With any stream processing solution, it's important to monitor the performance and health of the system.</span></span> <span data-ttu-id="4d821-211">[Azure Monitor](/azure/monitoring-and-diagnostics/) собирает журналы метрик и диагностики для служб Azure, используемых в этой архитектуре.</span><span class="sxs-lookup"><span data-stu-id="4d821-211">[Azure Monitor](/azure/monitoring-and-diagnostics/) collects metrics and diagnostics logs for the Azure services used in the architecture.</span></span> <span data-ttu-id="4d821-212">Служба Azure Monitor встроена в платформу Azure и не требует написания дополнительного кода в приложении.</span><span class="sxs-lookup"><span data-stu-id="4d821-212">Azure Monitor is built into the Azure platform and does not require any additional code in your application.</span></span>

<span data-ttu-id="4d821-213">Любой из следующих сигналов предупреждения указывает на то, что нужно масштабировать соответствующий ресурс Azure:</span><span class="sxs-lookup"><span data-stu-id="4d821-213">Any of the following warning signals indicate that you should scale out the relevant Azure resource:</span></span>

- <span data-ttu-id="4d821-214">Служба "Центры событий" регулирует запросы или указывает на то, что показатели близки к превышению дневной квоты сообщений.</span><span class="sxs-lookup"><span data-stu-id="4d821-214">Event Hubs throttles requests or is close to the daily message quota.</span></span>
- <span data-ttu-id="4d821-215">Задание Stream Analytics постоянно использует более 80 % выделенных единиц потоковой передачи.</span><span class="sxs-lookup"><span data-stu-id="4d821-215">The Stream Analytics job consistently uses more than 80% of allocated Streaming Units (SU).</span></span>
- <span data-ttu-id="4d821-216">Cosmos DB начинает регулировать запросы.</span><span class="sxs-lookup"><span data-stu-id="4d821-216">Cosmos DB begins to throttle requests.</span></span>

<span data-ttu-id="4d821-217">Эталонная архитектура включает пользовательскую панель мониторинга, которая развертывается на портале Azure.</span><span class="sxs-lookup"><span data-stu-id="4d821-217">The reference architecture includes a custom dashboard, which is deployed to the Azure portal.</span></span> <span data-ttu-id="4d821-218">Когда архитектура будет развернута, вы сможете просмотреть панель мониторинга, открыв [портал Azure](https://portal.azure.com) и выбрав `TaxiRidesDashboard` в списке панелей мониторинга.</span><span class="sxs-lookup"><span data-stu-id="4d821-218">After you deploy the architecture, you can view the dashboard by opening the [Azure Portal](https://portal.azure.com) and selecting `TaxiRidesDashboard` from list of dashboards.</span></span> <span data-ttu-id="4d821-219">Дополнительные сведения о создании и развертывании пользовательских панелей мониторинга см. в статье [Создание панелей мониторинга Azure программными средствами](/azure/azure-portal/azure-portal-dashboards-create-programmatically).</span><span class="sxs-lookup"><span data-stu-id="4d821-219">For more information about creating and deploying custom dashboards in the Azure portal, see [Programmatically create Azure Dashboards](/azure/azure-portal/azure-portal-dashboards-create-programmatically).</span></span>

<span data-ttu-id="4d821-220">На следующем изображении показана панель мониторинга приблизительно через час после запуска задания Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="4d821-220">The following image shows the dashboard after the Stream Analytics job ran for about an hour.</span></span>

![Снимок экрана панели мониторинга поездок в такси](./images/stream-processing-asa/asa-dashboard.png)

<span data-ttu-id="4d821-222">На внизу слева показано, что использование единиц потоковой передачи для задания Stream Analytics повышается в течение первых 15 минут и затем снижается.</span><span class="sxs-lookup"><span data-stu-id="4d821-222">The panel on the lower left shows that the SU consumption for the Stream Analytics job climbs during the first 15 minutes and then levels off.</span></span> <span data-ttu-id="4d821-223">Это стандартное развитие событий, так как задание достигает устойчивого состояния.</span><span class="sxs-lookup"><span data-stu-id="4d821-223">This is a typical pattern as the job reaches a steady state.</span></span>

<span data-ttu-id="4d821-224">Обратите внимание на то, что служба "Центры событий" регулирует запросы, как показано вверху справа.</span><span class="sxs-lookup"><span data-stu-id="4d821-224">Notice that Event Hubs is throttling requests, shown in the upper right panel.</span></span> <span data-ttu-id="4d821-225">Случайно отрегулированный запрос не является проблемой, так как клиентский пакет SDK службы "Центры событий" автоматически осуществляет новую попытку при получении ошибки регулирования.</span><span class="sxs-lookup"><span data-stu-id="4d821-225">An occasional throttled request is not a problem, because the Event Hubs client SDK automatically retries when it receives a throttling error.</span></span> <span data-ttu-id="4d821-226">Но если ошибки регулирования повторяются, значит службе требуются дополнительные единицы пропускной способности.</span><span class="sxs-lookup"><span data-stu-id="4d821-226">However, if you see consistent throttling errors, it means the event hub needs more throughput units.</span></span> <span data-ttu-id="4d821-227">На следующей диаграмме показан тестовый запуск с использованием функции автоматического расширения в службе "Центры событий", которая автоматически масштабирует единицы пропускной способности при необходимости.</span><span class="sxs-lookup"><span data-stu-id="4d821-227">The following graph shows a test run using the Event Hubs auto-inflate feature, which automatically scales out the throughput units as needed.</span></span>

![Снимок экрана с окном автоматического масштабирования в Центрах событий](./images/stream-processing-asa/stream-processing-eh-autoscale.png)

<span data-ttu-id="4d821-229">Функция автоматического расширения включена приблизительно в 06:35.</span><span class="sxs-lookup"><span data-stu-id="4d821-229">Auto-inflate was enabled at about the 06:35 mark.</span></span> <span data-ttu-id="4d821-230">Вы можете заметить снижение числа пакетов в регулируемых запросах, так как служба "Центры событий" автоматически масштабируется до трех единиц пропускной способности.</span><span class="sxs-lookup"><span data-stu-id="4d821-230">You can see the p drop in throttled requests, as Event Hubs automatically scaled up to 3 throughput units.</span></span>

<span data-ttu-id="4d821-231">Интересно, что это побочный эффект на увеличение использования единиц SU в задании Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="4d821-231">Interestingly, this had the side effect of increasing the SU utilization in the Stream Analytics job.</span></span> <span data-ttu-id="4d821-232">При регулировании в службе "Центры событий" искусственно снизилась скорость приема данных для задания Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="4d821-232">By throttling, Event Hubs was artificially reducing the ingestion rate for the Stream Analytics job.</span></span> <span data-ttu-id="4d821-233">Довольно часто бывает так, что при устранении одной проблемы с производительностью возникает другая.</span><span class="sxs-lookup"><span data-stu-id="4d821-233">It's actually common that resolving one performance bottleneck reveals another.</span></span> <span data-ttu-id="4d821-234">В таком случае проблему можно решить, выделив дополнительные единицы потоковой передачи для задания Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="4d821-234">In this case, allocating additional SU for the Stream Analytics job resolved the issue.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="4d821-235">Развертывание решения</span><span class="sxs-lookup"><span data-stu-id="4d821-235">Deploy the solution</span></span>

<span data-ttu-id="4d821-236">Чтобы выполнить развертывание и запуск эталонной реализации, выполните действия, описанные в [файле сведений на GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="4d821-236">To the deploy and run the reference implementation, follow the steps in the [GitHub readme][github].</span></span>

## <a name="related-resources"></a><span data-ttu-id="4d821-237">Связанные ресурсы</span><span class="sxs-lookup"><span data-stu-id="4d821-237">Related resources</span></span>

<span data-ttu-id="4d821-238">Вы можете просмотреть следующий [пример сценария Azure](/azure/architecture/example-scenario), в котором описываются конкретные решения, использующие некоторые из этих технологий:</span><span class="sxs-lookup"><span data-stu-id="4d821-238">You may wish to review the following [Azure example scenarios](/azure/architecture/example-scenario) that demonstrate specific solutions using some of the same technologies:</span></span>

- [<span data-ttu-id="4d821-239">Решения Интернета вещей и аналитики данных для строительной отрасли</span><span class="sxs-lookup"><span data-stu-id="4d821-239">IoT and data analytics in the construction industry</span></span>](/azure/architecture/example-scenario/data/big-data-with-iot)
- [<span data-ttu-id="4d821-240">Выявление мошенничества в реальном времени</span><span class="sxs-lookup"><span data-stu-id="4d821-240">Real-time fraud detection</span></span>](/azure/architecture/example-scenario/data/fraud-detection)

<!-- links -->

[github]: https://github.com/mspnp/azure-stream-analytics-data-pipeline

