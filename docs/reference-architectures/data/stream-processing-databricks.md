---
title: Обработка потоков данных с помощью Azure Databricks
titleSuffix: Azure Reference Architectures
description: Создание сквозного конвейера обработки потоков данных в Azure с помощью Azure Databricks.
author: petertaylor9999
ms.date: 11/30/2018
ms.custom: seodec18
ms.openlocfilehash: 822a3c448dcc2bdd4ae77ef2a2b7a9ffad633440
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/08/2018
ms.locfileid: "53120328"
---
# <a name="create-a-stream-processing-pipeline-with-azure-databricks"></a><span data-ttu-id="135a4-103">Создание конвейера обработки потоков данных с помощью Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="135a4-103">Create a stream processing pipeline with Azure Databricks</span></span>

<span data-ttu-id="135a4-104">На схеме эталонной архитектуры представлен сквозной конвейер [обработки потоков данных](/azure/architecture/data-guide/big-data/real-time-processing).</span><span class="sxs-lookup"><span data-stu-id="135a4-104">This reference architecture shows an end-to-end [stream processing](/azure/architecture/data-guide/big-data/real-time-processing) pipeline.</span></span> <span data-ttu-id="135a4-105">Конвейер такого типа состоит из четырех этапов: прием, обработка, сохранение, анализ и создание отчета.</span><span class="sxs-lookup"><span data-stu-id="135a4-105">This type of pipeline has four stages: ingest, process, store, and analysis and reporting.</span></span> <span data-ttu-id="135a4-106">В этой эталонной архитектуре конвейер принимает данные из двух источников, объединяет связанные записи из каждого потока, дополняет результат и вычисляет среднее значение в реальном времени.</span><span class="sxs-lookup"><span data-stu-id="135a4-106">For this reference architecture, the pipeline ingests data from two sources, performs a join on related records from each stream, enriches the result, and calculates an average in real time.</span></span> <span data-ttu-id="135a4-107">Результаты сохраняются для дальнейшего анализа.</span><span class="sxs-lookup"><span data-stu-id="135a4-107">The results are stored for further analysis.</span></span> <span data-ttu-id="135a4-108">[**Разверните это решение**](#deploy-the-solution).</span><span class="sxs-lookup"><span data-stu-id="135a4-108">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Эталонная архитектура для потоковой обработки с помощью Azure Databricks](./images/stream-processing-databricks.png)

<span data-ttu-id="135a4-110">**Сценарий.** Компания, предоставляющая услуги такси, собирает данные о каждой поездке.</span><span class="sxs-lookup"><span data-stu-id="135a4-110">**Scenario**: A taxi company collects data about each taxi trip.</span></span> <span data-ttu-id="135a4-111">В этом сценарии предполагается, что данные отправляются с двух отдельных устройств.</span><span class="sxs-lookup"><span data-stu-id="135a4-111">For this scenario, we assume there are two separate devices sending data.</span></span> <span data-ttu-id="135a4-112">В такси установлен счетчик, который отправляет данные о каждой поездке, включая сведения о продолжительности, расстоянии, а также местах посадки и высадки.</span><span class="sxs-lookup"><span data-stu-id="135a4-112">The taxi has a meter that sends information about each ride &mdash; the duration, distance, and pickup and dropoff locations.</span></span> <span data-ttu-id="135a4-113">Отдельное устройство принимает платежи от клиентов и отправляет данные о тарифах.</span><span class="sxs-lookup"><span data-stu-id="135a4-113">A separate device accepts payments from customers and sends data about fares.</span></span> <span data-ttu-id="135a4-114">Чтобы определить тенденции роста пассажиропотока, компании нужно для каждого округа вычислить среднюю сумму чаевых на милю в реальном времени.</span><span class="sxs-lookup"><span data-stu-id="135a4-114">To spot ridership trends, the taxi company wants to calculate the average tip per mile driven, in real time, for each neighborhood.</span></span>

## <a name="architecture"></a><span data-ttu-id="135a4-115">Архитектура</span><span class="sxs-lookup"><span data-stu-id="135a4-115">Architecture</span></span>

<span data-ttu-id="135a4-116">Архитектура состоит из следующих компонентов:</span><span class="sxs-lookup"><span data-stu-id="135a4-116">The architecture consists of the following components.</span></span>

<span data-ttu-id="135a4-117">**Источники данных.**</span><span class="sxs-lookup"><span data-stu-id="135a4-117">**Data sources**.</span></span> <span data-ttu-id="135a4-118">В этой архитектуре существует два источника данных, которые создают потоки данных в реальном времени.</span><span class="sxs-lookup"><span data-stu-id="135a4-118">In this architecture, there are two data sources that generate data streams in real time.</span></span> <span data-ttu-id="135a4-119">Первый поток содержит сведения о поездке, а второй — о тарифах.</span><span class="sxs-lookup"><span data-stu-id="135a4-119">The first stream contains ride information, and the second contains fare information.</span></span> <span data-ttu-id="135a4-120">В эталонной архитектуре есть имитированный генератор данных, который считывает данные из набора статических файлов и отправляет данные в Центры событий.</span><span class="sxs-lookup"><span data-stu-id="135a4-120">The reference architecture includes a simulated data generator that reads from a set of static files and pushes the data to Event Hubs.</span></span> <span data-ttu-id="135a4-121">В реальном приложении источниками данных будут устройства, установленные в такси.</span><span class="sxs-lookup"><span data-stu-id="135a4-121">The data sources in a real application would be devices installed in the taxi cabs.</span></span>

<span data-ttu-id="135a4-122">**Центры событий Azure**.</span><span class="sxs-lookup"><span data-stu-id="135a4-122">**Azure Event Hubs**.</span></span> <span data-ttu-id="135a4-123">[Центры событий](/azure/event-hubs/) — это служба приема событий.</span><span class="sxs-lookup"><span data-stu-id="135a4-123">[Event Hubs](/azure/event-hubs/) is an event ingestion service.</span></span> <span data-ttu-id="135a4-124">В этой архитектуре используется два экземпляра службы — по одной на каждый источник данных.</span><span class="sxs-lookup"><span data-stu-id="135a4-124">This architecture uses two event hub instances, one for each data source.</span></span> <span data-ttu-id="135a4-125">Каждый источник данных отправляет поток данных в соответствующую службу.</span><span class="sxs-lookup"><span data-stu-id="135a4-125">Each data source sends a stream of data to the associated event hub.</span></span>

<span data-ttu-id="135a4-126">**Azure Databricks**.</span><span class="sxs-lookup"><span data-stu-id="135a4-126">**Azure Databricks**.</span></span> <span data-ttu-id="135a4-127">[Databricks](/azure/azure-databricks/) — это платформа аналитики на основе Apache Spark, оптимизированная для платформы облачных служб Microsoft Azure.</span><span class="sxs-lookup"><span data-stu-id="135a4-127">[Databricks](/azure/azure-databricks/) is an Apache Spark-based analytics platform optimized for the Microsoft Azure cloud services platform.</span></span> <span data-ttu-id="135a4-128">Databricks используется для корреляции данных о поездке на такси и тарифах, а также для дополнения коррелированных данных сведениями об округе, хранящимися в файловой системе Databricks.</span><span class="sxs-lookup"><span data-stu-id="135a4-128">Databricks is used to correlate of the taxi ride and fare data, and also to enrich the correlated data with neighborhood data stored in the Databricks file system.</span></span>

<span data-ttu-id="135a4-129">**Cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="135a4-129">**Cosmos DB**.</span></span> <span data-ttu-id="135a4-130">Выходные данные задания Azure Databricks будут представлять собой последовательность записей, которые вносятся в [Cosmos DB](/azure/cosmos-db/) с помощью API Cassandra.</span><span class="sxs-lookup"><span data-stu-id="135a4-130">The output from Azure Databricks job is a series of records, which are written to [Cosmos DB](/azure/cosmos-db/) using the Cassandra API.</span></span> <span data-ttu-id="135a4-131">Мы используем API Cassandra, потому что этот интерфейс поддерживает моделирование данных временных рядов.</span><span class="sxs-lookup"><span data-stu-id="135a4-131">The Cassandra API is used because it supports time series data modeling.</span></span>

<span data-ttu-id="135a4-132">**Azure Log Analytics**.</span><span class="sxs-lookup"><span data-stu-id="135a4-132">**Azure Log Analytics**.</span></span> <span data-ttu-id="135a4-133">Данные журнала приложений, собранные [Azure Monitor](/azure/monitoring-and-diagnostics/), хранятся в [рабочей области Log Analytics](/azure/log-analytics).</span><span class="sxs-lookup"><span data-stu-id="135a4-133">Application log data collected by [Azure Monitor](/azure/monitoring-and-diagnostics/) is stored in a [Log Analytics workspace](/azure/log-analytics).</span></span> <span data-ttu-id="135a4-134">Вы можете использовать запросы Log Analytics для анализа и визуализации метрик, а также просмотра сообщений журнала, чтобы выявить проблемы в приложении.</span><span class="sxs-lookup"><span data-stu-id="135a4-134">Log Analytics queries can be used to analyze and visualize metrics and inspect log messages to identify issues within the application.</span></span>

## <a name="data-ingestion"></a><span data-ttu-id="135a4-135">Прием данных</span><span class="sxs-lookup"><span data-stu-id="135a4-135">Data ingestion</span></span>

<span data-ttu-id="135a4-136">Для имитации источника данных в этой эталонной архитектуре используется набор данных<sup>[[1]](#note1)</sup> [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) (Данные о поездках в такси в Нью-Йорке).</span><span class="sxs-lookup"><span data-stu-id="135a4-136">To simulate a data source, this reference architecture uses the [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) dataset<sup>[[1]](#note1)</sup>.</span></span> <span data-ttu-id="135a4-137">Этот набор содержит данные о поездках в такси в Нью-Йорке за 4 года (2010&ndash;2013).</span><span class="sxs-lookup"><span data-stu-id="135a4-137">This dataset contains data about taxi trips in New York City over a four-year period (2010 &ndash; 2013).</span></span> <span data-ttu-id="135a4-138">Он содержит два типа записей: данные о поездке и данные о тарифе.</span><span class="sxs-lookup"><span data-stu-id="135a4-138">It contains two types of record: Ride data and fare data.</span></span> <span data-ttu-id="135a4-139">Данные о поездках включают сведения о продолжительности поездки, расстоянии, а также местах посадки и высадки.</span><span class="sxs-lookup"><span data-stu-id="135a4-139">Ride data includes trip duration, trip distance, and pickup and dropoff location.</span></span> <span data-ttu-id="135a4-140">Данные о тарифах включают сведения о тарифе, налоге и сумме чаевых.</span><span class="sxs-lookup"><span data-stu-id="135a4-140">Fare data includes fare, tax, and tip amounts.</span></span> <span data-ttu-id="135a4-141">В обоих типах записей есть стандартные поля: номер медальона, лицензия на право вождения и код организации.</span><span class="sxs-lookup"><span data-stu-id="135a4-141">Common fields in both record types include medallion number, hack license, and vendor ID.</span></span> <span data-ttu-id="135a4-142">Вместе эти три поля позволяют уникально идентифицировать такси и водителя.</span><span class="sxs-lookup"><span data-stu-id="135a4-142">Together these three fields uniquely identify a taxi plus a driver.</span></span> <span data-ttu-id="135a4-143">Данные хранятся в формате CSV.</span><span class="sxs-lookup"><span data-stu-id="135a4-143">The data is stored in CSV format.</span></span>

<span data-ttu-id="135a4-144">Генератор данных — это приложение .NET Core, которое считывает записи и отправляет их в Центры событий Azure.</span><span class="sxs-lookup"><span data-stu-id="135a4-144">The data generator is a .NET Core application that reads the records and sends them to Azure Event Hubs.</span></span> <span data-ttu-id="135a4-145">Генератор отправляет данные о поездке в формате JSON, а данные о тарифах — в формате CSV.</span><span class="sxs-lookup"><span data-stu-id="135a4-145">The generator sends ride data in JSON format and fare data in CSV format.</span></span>

<span data-ttu-id="135a4-146">Для сегментации данных Центры событий используют [секции](/azure/event-hubs/event-hubs-features#partitions).</span><span class="sxs-lookup"><span data-stu-id="135a4-146">Event Hubs uses [partitions](/azure/event-hubs/event-hubs-features#partitions) to segment the data.</span></span> <span data-ttu-id="135a4-147">Они позволяют объекту-получателю считывать данные каждой секции параллельно.</span><span class="sxs-lookup"><span data-stu-id="135a4-147">Partitions allow a consumer to read each partition in parallel.</span></span> <span data-ttu-id="135a4-148">При отправке данных в Центры событий можно явно указать ключ секции.</span><span class="sxs-lookup"><span data-stu-id="135a4-148">When you send data to Event Hubs, you can specify the partition key explicitly.</span></span> <span data-ttu-id="135a4-149">В противном случае записи назначаются секциям методом циклического перебора.</span><span class="sxs-lookup"><span data-stu-id="135a4-149">Otherwise, records are assigned to partitions in round-robin fashion.</span></span>

<span data-ttu-id="135a4-150">В этом примере данные о поездках и тарифах должны в итоге иметь одинаковый идентификатор секции для определенного такси.</span><span class="sxs-lookup"><span data-stu-id="135a4-150">In this scenario, ride data and fare data should end up with the same partition ID for a given taxi cab.</span></span> <span data-ttu-id="135a4-151">Это позволит Databricks применить определенную степень параллелизма при корреляции двух потоков.</span><span class="sxs-lookup"><span data-stu-id="135a4-151">This enables Databricks to apply a degree of parallelism when it correlates the two streams.</span></span> <span data-ttu-id="135a4-152">Запись в секции *n* с данными о поездке будет соответствовать записи в секции *n* с данными о тарифах.</span><span class="sxs-lookup"><span data-stu-id="135a4-152">A record in partition *n* of the ride data will match a record in partition *n* of the fare data.</span></span>

![Схема потоковой обработки с помощью Azure Databricks и Центров событий Azure](./images/stream-processing-databricks-eh.png)

<span data-ttu-id="135a4-154">В генераторе данных общая модель данных для обоих типов записей имеет свойство `PartitionKey`, в котором объединены `Medallion`, `HackLicense` и `VendorId`.</span><span class="sxs-lookup"><span data-stu-id="135a4-154">In the data generator, the common data model for both record types has a `PartitionKey` property that is the concatenation of `Medallion`, `HackLicense`, and `VendorId`.</span></span>

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

<span data-ttu-id="135a4-155">Это свойство используется для явного предоставления ключа секции при отправке в Центры событий:</span><span class="sxs-lookup"><span data-stu-id="135a4-155">This property is used to provide an explicit partition key when sending to Event Hubs:</span></span>

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

### <a name="event-hubs"></a><span data-ttu-id="135a4-156">Центры событий;</span><span class="sxs-lookup"><span data-stu-id="135a4-156">Event Hubs</span></span>

<span data-ttu-id="135a4-157">Пропускная способность Центров событий вычисляется в [единицах пропускной способности](/azure/event-hubs/event-hubs-features#throughput-units).</span><span class="sxs-lookup"><span data-stu-id="135a4-157">The throughput capacity of Event Hubs is measured in [throughput units](/azure/event-hubs/event-hubs-features#throughput-units).</span></span> <span data-ttu-id="135a4-158">Вы можете автоматически масштабировать концентратор событий, включив [автоматическое расширение](/azure/event-hubs/event-hubs-auto-inflate). Это позволит автоматически масштабировать единицы пропускной способности в зависимости от трафика вплоть до заданного максимума.</span><span class="sxs-lookup"><span data-stu-id="135a4-158">You can autoscale an event hub by enabling [auto-inflate](/azure/event-hubs/event-hubs-auto-inflate), which automatically scales the throughput units based on traffic, up to a configured maximum.</span></span>

## <a name="stream-processing"></a><span data-ttu-id="135a4-159">Потоковая обработка</span><span class="sxs-lookup"><span data-stu-id="135a4-159">Stream processing</span></span>

<span data-ttu-id="135a4-160">В Azure Databricks обработка данных осуществляется с помощью задания.</span><span class="sxs-lookup"><span data-stu-id="135a4-160">In Azure Databricks, data processing is performed by a job.</span></span> <span data-ttu-id="135a4-161">Задание назначается определенному кластеру и выполняется в нем.</span><span class="sxs-lookup"><span data-stu-id="135a4-161">The job is assigned to and runs on a cluster.</span></span> <span data-ttu-id="135a4-162">Задание может представлять собой пользовательский код на Java или [записную книжку](https://docs.databricks.com/user-guide/notebooks/index.html) Spark.</span><span class="sxs-lookup"><span data-stu-id="135a4-162">The job can either be custom code written in Java, or a Spark [notebook](https://docs.databricks.com/user-guide/notebooks/index.html).</span></span>

<span data-ttu-id="135a4-163">В этой эталонной архитектуре задание представляет собой архив Java с классами, написанными на Java и Scala.</span><span class="sxs-lookup"><span data-stu-id="135a4-163">In this reference architecture, the job is a Java archive with classes written in both Java and Scala.</span></span> <span data-ttu-id="135a4-164">Указывая архив Java для задания Databricks, нужно выбрать класс для выполнения в кластере Databricks.</span><span class="sxs-lookup"><span data-stu-id="135a4-164">When specifying the Java archive for a Databricks job, the class is specified for execution by the Databricks cluster.</span></span> <span data-ttu-id="135a4-165">В этом примере метод **main** класса **com.microsoft.pnp.TaxiCabReader** содержит логику обработки данных.</span><span class="sxs-lookup"><span data-stu-id="135a4-165">Here, the **main** method of the **com.microsoft.pnp.TaxiCabReader** class contains the data processing logic.</span></span>

### <a name="reading-the-stream-from-the-two-event-hub-instances"></a><span data-ttu-id="135a4-166">Считывание потоков данных из двух экземпляров концентратора событий</span><span class="sxs-lookup"><span data-stu-id="135a4-166">Reading the stream from the two event hub instances</span></span>

<span data-ttu-id="135a4-167">Для считывания данных из двух экземпляров концентратора событий Azure в логике обработки данных используется [Spark Structured Streaming](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html).</span><span class="sxs-lookup"><span data-stu-id="135a4-167">The data processing logic uses [Spark structured streaming](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html) to read from the two Azure event hub instances:</span></span>

```scala
val rideEventHubOptions = EventHubsConf(rideEventHubConnectionString)
      .setConsumerGroup(conf.taxiRideConsumerGroup())
      .setStartingPosition(EventPosition.fromStartOfStream)
    val rideEvents = spark.readStream
      .format("eventhubs")
      .options(rideEventHubOptions.toMap)
      .load

    val fareEventHubOptions = EventHubsConf(fareEventHubConnectionString)
      .setConsumerGroup(conf.taxiFareConsumerGroup())
      .setStartingPosition(EventPosition.fromStartOfStream)
    val fareEvents = spark.readStream
      .format("eventhubs")
      .options(fareEventHubOptions.toMap)
      .load
```

### <a name="enriching-the-data-with-the-neighborhood-information"></a><span data-ttu-id="135a4-168">Дополнение данных сведениями об округе</span><span class="sxs-lookup"><span data-stu-id="135a4-168">Enriching the data with the neighborhood information</span></span>

<span data-ttu-id="135a4-169">Данные о поездке включают координаты широты и долготы и сведения о местах посадки и высадки.</span><span class="sxs-lookup"><span data-stu-id="135a4-169">The ride data includes the latitude and longitude coordinates of the pick up and drop off locations.</span></span> <span data-ttu-id="135a4-170">Хотя эти координаты и полезны, их неудобно использовать для анализа.</span><span class="sxs-lookup"><span data-stu-id="135a4-170">While these coordinates are useful, they are not easily consumed for analysis.</span></span> <span data-ttu-id="135a4-171">Таким образом, эти данные дополняются сведениями об округе, считанными из [файла фигуры](https://en.wikipedia.org/wiki/Shapefile).</span><span class="sxs-lookup"><span data-stu-id="135a4-171">Therefore, this data is enriched with neighborhood data that is read from a [shapefile](https://en.wikipedia.org/wiki/Shapefile).</span></span>

<span data-ttu-id="135a4-172">Файл фигуры имеет двоичный формат, что усложняет анализ. Но в библиотеке [GeoTools](http://geotools.org/) представлены средства для работы с геопространственными данными в формате файла фигуры.</span><span class="sxs-lookup"><span data-stu-id="135a4-172">The shapefile format is binary and not easily parsed, but the [GeoTools](http://geotools.org/) library provides tools for geospatial data that use the shapefile format.</span></span> <span data-ttu-id="135a4-173">Эта библиотека используется в классе **com.microsoft.pnp.GeoFinder**, чтобы определить название округа на основе координат мест посадки и высадки.</span><span class="sxs-lookup"><span data-stu-id="135a4-173">This library is used in the **com.microsoft.pnp.GeoFinder** class to determine the neighborhood name based on the pick up and drop off coordinates.</span></span>

```scala
val neighborhoodFinder = (lon: Double, lat: Double) => {
      NeighborhoodFinder.getNeighborhood(lon, lat).get()
    }
```

### <a name="joining-the-ride-and-fare-data"></a><span data-ttu-id="135a4-174">Объединение данных о поездках и тарифах</span><span class="sxs-lookup"><span data-stu-id="135a4-174">Joining the ride and fare data</span></span>

<span data-ttu-id="135a4-175">Сначала выполняется преобразование данных о поездках и тарифах:</span><span class="sxs-lookup"><span data-stu-id="135a4-175">First the ride and fare data is transformed:</span></span>

```scala
    val rides = transformedRides
      .filter(r => {
        if (r.isNullAt(r.fieldIndex("errorMessage"))) {
          true
        }
        else {
          malformedRides.add(1)
          false
        }
      })
      .select(
        $"ride.*",
        to_neighborhood($"ride.pickupLon", $"ride.pickupLat")
          .as("pickupNeighborhood"),
        to_neighborhood($"ride.dropoffLon", $"ride.dropoffLat")
          .as("dropoffNeighborhood")
      )
      .withWatermark("pickupTime", conf.taxiRideWatermarkInterval())

    val fares = transformedFares
      .filter(r => {
        if (r.isNullAt(r.fieldIndex("errorMessage"))) {
          true
        }
        else {
          malformedFares.add(1)
          false
        }
      })
      .select(
        $"fare.*",
        $"pickupTime"
      )
      .withWatermark("pickupTime", conf.taxiFareWatermarkInterval())
```

<span data-ttu-id="135a4-176">После этого происходит объединение данных о поездке с данными о тарифах:</span><span class="sxs-lookup"><span data-stu-id="135a4-176">And then the ride data is joined with the fare data:</span></span>

```scala
val mergedTaxiTrip = rides.join(fares, Seq("medallion", "hackLicense", "vendorId", "pickupTime"))
```

### <a name="processing-the-data-and-inserting-into-cosmos-db"></a><span data-ttu-id="135a4-177">Обработка и вставка данных в Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="135a4-177">Processing the data and inserting into Cosmos DB</span></span>

<span data-ttu-id="135a4-178">Вычисляется средний тариф для каждого округа за определенный интервал времени:</span><span class="sxs-lookup"><span data-stu-id="135a4-178">The average fare amount for each neighborhood is calculated for a given time interval:</span></span>

```scala
val maxAvgFarePerNeighborhood = mergedTaxiTrip.selectExpr("medallion", "hackLicense", "vendorId", "pickupTime", "rateCode", "storeAndForwardFlag", "dropoffTime", "passengerCount", "tripTimeInSeconds", "tripDistanceInMiles", "pickupLon", "pickupLat", "dropoffLon", "dropoffLat", "paymentType", "fareAmount", "surcharge", "mtaTax", "tipAmount", "tollsAmount", "totalAmount", "pickupNeighborhood", "dropoffNeighborhood")
      .groupBy(window($"pickupTime", conf.windowInterval()), $"pickupNeighborhood")
      .agg(
        count("*").as("rideCount"),
        sum($"fareAmount").as("totalFareAmount"),
        sum($"tipAmount").as("totalTipAmount")
      )
      .select($"window.start", $"window.end", $"pickupNeighborhood", $"rideCount", $"totalFareAmount", $"totalTipAmount")
```

<span data-ttu-id="135a4-179">Затем результат вставляется в Cosmos DB:</span><span class="sxs-lookup"><span data-stu-id="135a4-179">Which is then inserted into Cosmos DB:</span></span>

```scala
maxAvgFarePerNeighborhood
      .writeStream
      .queryName("maxAvgFarePerNeighborhood_cassandra_insert")
      .outputMode(OutputMode.Append())
      .foreach(new CassandraSinkForeach(connector))
      .start()
      .awaitTermination()
```

## <a name="security-considerations"></a><span data-ttu-id="135a4-180">Вопросы безопасности</span><span class="sxs-lookup"><span data-stu-id="135a4-180">Security considerations</span></span>

<span data-ttu-id="135a4-181">Управление доступом к рабочей области базы данных Azure осуществляется с помощью [консоли администрирования](https://docs.databricks.com/administration-guide/admin-settings/index.html).</span><span class="sxs-lookup"><span data-stu-id="135a4-181">Access to the Azure Database workspace is controlled using the [administrator console](https://docs.databricks.com/administration-guide/admin-settings/index.html).</span></span> <span data-ttu-id="135a4-182">В консоли администрирования предусмотрены функции для добавления пользователей, управления их разрешениями и настройки единого входа.</span><span class="sxs-lookup"><span data-stu-id="135a4-182">The administrator console includes functionality to add users, manage user permissions, and set up single sign-on.</span></span> <span data-ttu-id="135a4-183">Также в ней можно настроить управление доступом для рабочих областей, кластеров, заданий и таблиц.</span><span class="sxs-lookup"><span data-stu-id="135a4-183">Access control for workspaces, clusters, jobs, and tables can also be set through the administrator console.</span></span>

### <a name="managing-secrets"></a><span data-ttu-id="135a4-184">управление секретами;</span><span class="sxs-lookup"><span data-stu-id="135a4-184">Managing secrets</span></span>

<span data-ttu-id="135a4-185">В Azure Databricks предусмотрено [хранилище секретов](https://docs.azuredatabricks.net/user-guide/secrets/index.html), которое используется для хранения не только секретов, но и строк подключения, ключей доступа, имен пользователей и паролей.</span><span class="sxs-lookup"><span data-stu-id="135a4-185">Azure Databricks includes a [secret store](https://docs.azuredatabricks.net/user-guide/secrets/index.html) that is used to store secrets, including connection strings, access keys, user names, and passwords.</span></span> <span data-ttu-id="135a4-186">Секреты в хранилище секретов Azure Databricks секционируются по **областям**:</span><span class="sxs-lookup"><span data-stu-id="135a4-186">Secrets within the Azure Databricks secret store are partitioned by **scopes**:</span></span>

```bash
databricks secrets create-scope --scope "azure-databricks-job"
```

<span data-ttu-id="135a4-187">Секреты добавляются на уровне области:</span><span class="sxs-lookup"><span data-stu-id="135a4-187">Secrets are added at the scope level:</span></span>

```bash
databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
```

> [!NOTE]
> <span data-ttu-id="135a4-188">Вы можете использовать области, поддерживаемые Azure Key Vault, вместо стандартных областей Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="135a4-188">An Azure Key Vault-backed scope can be used instead of the native Azure Databricks scope.</span></span> <span data-ttu-id="135a4-189">Дополнительные сведения см. в документации по [областям, поддерживаемым Azure Key Vault](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).</span><span class="sxs-lookup"><span data-stu-id="135a4-189">To learn more, see [Azure Key Vault-backed scopes](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).</span></span>

<span data-ttu-id="135a4-190">В коде для получения доступа к секретам используются [соответствующие служебные программы](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities) Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="135a4-190">In code, secrets are accessed via the Azure Databricks [secrets utilities](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities).</span></span>

## <a name="monitoring-considerations"></a><span data-ttu-id="135a4-191">Рекомендации по мониторингу</span><span class="sxs-lookup"><span data-stu-id="135a4-191">Monitoring considerations</span></span>

<span data-ttu-id="135a4-192">Платформа Azure Databricks создана на основе Apache Spark и тоже использует [log4j](https://logging.apache.org/log4j/2.x/) в качестве стандартной библиотеки для ведения журналов.</span><span class="sxs-lookup"><span data-stu-id="135a4-192">Azure Databricks is based on Apache Spark, and both use [log4j](https://logging.apache.org/log4j/2.x/) as the standard library for logging.</span></span> <span data-ttu-id="135a4-193">Кроме использования стандартных возможностей ведения журнала, предоставляемых Apache Spark, в этой эталонной архитектуре предусматривается отправка журналов и метрик в [Azure Log Analytics](/azure/log-analytics/).</span><span class="sxs-lookup"><span data-stu-id="135a4-193">In addition to the default logging provided by Apache Spark, this reference architecture sends logs and metrics to [Azure Log Analytics](/azure/log-analytics/).</span></span>

<span data-ttu-id="135a4-194">Используя класс **com.microsoft.pnp.TaxiCabReader** и значения в файле **log4j.properties** вы можете настроить систему ведения журналов Apache Spark для отправки журналов в Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="135a4-194">The **com.microsoft.pnp.TaxiCabReader** class configures the Apache Spark logging system to send its logs to Azure Log Analytics using the values in the **log4j.properties** file.</span></span> <span data-ttu-id="135a4-195">Хотя сообщения средства ведения журнала Apache Spark представлены в строковом формате, для Azure Log Analytics требуется, чтобы эти сообщения были в формате JSON.</span><span class="sxs-lookup"><span data-stu-id="135a4-195">While the Apache Spark logger messages are strings, Azure Log Analytics requires log messages to be formatted as JSON.</span></span> <span data-ttu-id="135a4-196">Класс **com.microsoft.pnp.log4j.LogAnalyticsAppender** позволяет преобразовать такие сообщения в формат JSON:</span><span class="sxs-lookup"><span data-stu-id="135a4-196">The **com.microsoft.pnp.log4j.LogAnalyticsAppender** class transforms these messages to JSON:</span></span>

```scala

    @Override
    protected void append(LoggingEvent loggingEvent) {
        if (this.layout == null) {
            this.setLayout(new JSONLayout());
        }

        String json = this.getLayout().format(loggingEvent);
        try {
            this.client.send(json, this.logType);
        } catch(IOException ioe) {
            LogLog.warn("Error sending LoggingEvent to Log Analytics", ioe);
        }
    }

```

<span data-ttu-id="135a4-197">В результате обработки сообщений о поездках и тарифах с помощью класса **com.microsoft.pnp.TaxiCabReader** некоторые сообщения могу иметь неправильный формат и поэтому быть недопустимыми.</span><span class="sxs-lookup"><span data-stu-id="135a4-197">As the **com.microsoft.pnp.TaxiCabReader** class processes ride and fare messages, it's possible that either one may be malformed and therefore not valid.</span></span> <span data-ttu-id="135a4-198">В рабочей среде нужно проанализировать такие сообщения, чтобы выявить проблему с источниками данных и быстро решить ее для предотвращения потери данных.</span><span class="sxs-lookup"><span data-stu-id="135a4-198">In a production environment, it's important to analyze these malformed messages to identify a problem with the data sources so it can be fixed quickly to prevent data loss.</span></span> <span data-ttu-id="135a4-199">Класс **com.microsoft.pnp.TaxiCabReader** позволяет зарегистрировать накопитель Apache Spark, который отслеживает количество записей о поездках и тарифах в неправильном формате.</span><span class="sxs-lookup"><span data-stu-id="135a4-199">The **com.microsoft.pnp.TaxiCabReader** class registers an Apache Spark Accumulator that keeps track of the number of malformed fare and ride records:</span></span>

```scala
    @transient val appMetrics = new AppMetrics(spark.sparkContext)
    appMetrics.registerGauge("metrics.malformedrides", AppAccumulators.getRideInstance(spark.sparkContext))
    appMetrics.registerGauge("metrics.malformedfares", AppAccumulators.getFareInstance(spark.sparkContext))
    SparkEnv.get.metricsSystem.registerSource(appMetrics)
```

<span data-ttu-id="135a4-200">Для отправки метрик Apache Spark использует библиотеку Dropwizard. Но некоторые стандартные поля метрик Dropwizard несовместимы с Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="135a4-200">Apache Spark uses the Dropwizard library to send metrics, and some of the native Dropwizard metrics fields are incompatible with Azure Log Analytics.</span></span> <span data-ttu-id="135a4-201">Поэтому в эталонной архитектуре предусмотрены пользовательские приемник и средство формирования отчетов Dropwizard.</span><span class="sxs-lookup"><span data-stu-id="135a4-201">Therefore, this reference architecture includes a custom Dropwizard sink and reporter.</span></span> <span data-ttu-id="135a4-202">Это позволяет преобразовать метрики в формат, требуемый для Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="135a4-202">It formats the metrics in the format expected by Azure Log Analytics.</span></span> <span data-ttu-id="135a4-203">Apache Spark передает метрики вместе с пользовательскими метриками для данных о поездках и тарифах неправильного формата.</span><span class="sxs-lookup"><span data-stu-id="135a4-203">When Apache Spark reports metrics, the custom metrics for the malformed ride and fare data are also sent.</span></span>

<span data-ttu-id="135a4-204">Запись последней метрики в журнал рабочей области Azure Log Analytics состоит из нескольких процессов в рамках выполняемого задания Spark Structured Streaming.</span><span class="sxs-lookup"><span data-stu-id="135a4-204">The last metric to be logged to the Azure Log Analytics workspace is the cumulative progress of the Spark Structured Streaming job progress.</span></span> <span data-ttu-id="135a4-205">Эта задача выполняется с помощью прослушивателя, реализованного в виде класса **com.microsoft.pnp.StreamingMetricsListener**.</span><span class="sxs-lookup"><span data-stu-id="135a4-205">This is done using a custom StreamingQuery listener implemented in the **com.microsoft.pnp.StreamingMetricsListener** class.</span></span> <span data-ttu-id="135a4-206">Этот класс регистрируется в сеансе Apache Spark при запуске задания:</span><span class="sxs-lookup"><span data-stu-id="135a4-206">This class is registered to the Apache Spark Session when the job runs:</span></span>

```scala
spark.streams.addListener(new StreamingMetricsListener())
```

<span data-ttu-id="135a4-207">В среде выполнения Apache Spark методы в StreamingMetricsListener вызываются при каждом событии структурированной потоковой передачи. При этом также отправляются сообщения и метрики журнала в рабочую область Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="135a4-207">The methods in the StreamingMetricsListener are called by the Apache Spark runtime whenever a structured steaming event occurs, sending log messages and metrics to the Azure Log Analytics workspace.</span></span> <span data-ttu-id="135a4-208">Для мониторинга приложения вы можете использовать следующие запросы в своей рабочей области:</span><span class="sxs-lookup"><span data-stu-id="135a4-208">You can use the following queries in your workspace to monitor the application:</span></span>

### <a name="latency-and-throughput-for-streaming-queries"></a><span data-ttu-id="135a4-209">Задержка и пропускная способность для запросов потоковой передачи</span><span class="sxs-lookup"><span data-stu-id="135a4-209">Latency and throughput for streaming queries</span></span>

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| project  mdc_inputRowsPerSecond_d, mdc_durationms_triggerExecution_d
| render timechart
```

### <a name="exceptions-logged-during-stream-query-execution"></a><span data-ttu-id="135a4-210">Исключения, зарегистрированные при выполнении запроса потоковой передачи</span><span class="sxs-lookup"><span data-stu-id="135a4-210">Exceptions logged during stream query execution</span></span>

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| where Level contains "Error"
```

### <a name="accumulation-of-malformed-fare-and-ride-data"></a><span data-ttu-id="135a4-211">Сбор данных неправильного формата о поездках и тарифах</span><span class="sxs-lookup"><span data-stu-id="135a4-211">Accumulation of malformed fare and ride data</span></span>

```shell
SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "metrics.malformedrides"

SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "metrics.malformedfares"
```

### <a name="job-execution-to-trace-resiliency"></a><span data-ttu-id="135a4-212">Данные о выполнении задания для трассировки устойчивости</span><span class="sxs-lookup"><span data-stu-id="135a4-212">Job execution to trace resiliency</span></span>

```shell
SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "driver.DAGScheduler.job.allJobs"
```

## <a name="deploy-the-solution"></a><span data-ttu-id="135a4-213">Развертывание решения</span><span class="sxs-lookup"><span data-stu-id="135a4-213">Deploy the solution</span></span>

<span data-ttu-id="135a4-214">Пример развертывания для этой архитектуры можно найти на портале [GitHub](https://github.com/mspnp/azure-databricks-streaming-analytics).</span><span class="sxs-lookup"><span data-stu-id="135a4-214">A deployment for this reference architecture is available on [GitHub](https://github.com/mspnp/azure-databricks-streaming-analytics).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="135a4-215">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="135a4-215">Prerequisites</span></span>

1. <span data-ttu-id="135a4-216">Клонируйте или загрузите репозиторий GitHub [Stream processing with Azure Databricks](https://github.com/mspnp/azure-databricks-streaming-analytics) (Обработка потоков данных с помощью Azure Databricks) либо создайте его вилку.</span><span class="sxs-lookup"><span data-stu-id="135a4-216">Clone, fork, or download the [stream processing with Azure Databricks](https://github.com/mspnp/azure-databricks-streaming-analytics) GitHub repository.</span></span>

2. <span data-ttu-id="135a4-217">Установите [Docker](https://www.docker.com/), чтобы запустить генератор данных.</span><span class="sxs-lookup"><span data-stu-id="135a4-217">Install [Docker](https://www.docker.com/) to run the data generator.</span></span>

3. <span data-ttu-id="135a4-218">Установите [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span><span class="sxs-lookup"><span data-stu-id="135a4-218">Install [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span></span>

4. <span data-ttu-id="135a4-219">Установите [интерфейс командной строки Databricks](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html).</span><span class="sxs-lookup"><span data-stu-id="135a4-219">Install [Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html).</span></span>

5. <span data-ttu-id="135a4-220">Из командной строки, строки bash или строки PowerShell войдите в свою учетную запись Azure, как показано ниже:</span><span class="sxs-lookup"><span data-stu-id="135a4-220">From a command prompt, bash prompt, or PowerShell prompt, sign into your Azure account as follows:</span></span>
    ```shell
    az login
    ```
6. <span data-ttu-id="135a4-221">Установите интегрированную среду разработки Java со следующими ресурсами:</span><span class="sxs-lookup"><span data-stu-id="135a4-221">Install a Java IDE, with the following resources:</span></span>
    - <span data-ttu-id="135a4-222">JDK 1.8;</span><span class="sxs-lookup"><span data-stu-id="135a4-222">JDK 1.8</span></span>
    - <span data-ttu-id="135a4-223">пакет SDK для Scala 2.11;</span><span class="sxs-lookup"><span data-stu-id="135a4-223">Scala SDK 2.11</span></span>
    - <span data-ttu-id="135a4-224">Maven 3.5.4.</span><span class="sxs-lookup"><span data-stu-id="135a4-224">Maven 3.5.4</span></span>

### <a name="download-the-new-york-city-taxi-and-neighborhood-data-files"></a><span data-ttu-id="135a4-225">Скачивание файлов с данными о поездках на такси в Нью-Йорке и сведениями об округах</span><span class="sxs-lookup"><span data-stu-id="135a4-225">Download the New York City taxi and neighborhood data files</span></span>

1. <span data-ttu-id="135a4-226">Создайте каталог с именем `DataFile` в корневом каталоге клонированного репозитория Github в своей локальной файловой системе.</span><span class="sxs-lookup"><span data-stu-id="135a4-226">Create a directory named `DataFile` in the root of the cloned Github repository in your local file system.</span></span>

2. <span data-ttu-id="135a4-227">Откройте браузер и перейдите по ссылке https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.</span><span class="sxs-lookup"><span data-stu-id="135a4-227">Open a web browser and navigate to https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.</span></span>

3. <span data-ttu-id="135a4-228">На этой странице нажмите кнопку **Скачать**, чтобы скачать ZIP-файл со всеми данные о поездках в такси за этот год.</span><span class="sxs-lookup"><span data-stu-id="135a4-228">Click the **Download** button on this page to download a zip file of all the taxi data for that year.</span></span>

4. <span data-ttu-id="135a4-229">Извлеките ZIP-файл в каталог `DataFile`.</span><span class="sxs-lookup"><span data-stu-id="135a4-229">Extract the zip file to the `DataFile` directory.</span></span>

    > [!NOTE]
    > <span data-ttu-id="135a4-230">Этот ZIP-файл содержит другие ZIP-файлы.</span><span class="sxs-lookup"><span data-stu-id="135a4-230">This zip file contains other zip files.</span></span> <span data-ttu-id="135a4-231">Не извлекайте дочерние ZIP-файлы.</span><span class="sxs-lookup"><span data-stu-id="135a4-231">Don't extract the child zip files.</span></span>

    <span data-ttu-id="135a4-232">Структура каталога должна выглядеть так:</span><span class="sxs-lookup"><span data-stu-id="135a4-232">The directory structure must look like the following:</span></span>

    ```shell
    /DataFile
        /FOIL2013
            trip_data_1.zip
            trip_data_2.zip
            trip_data_3.zip
            ...
    ```

5. <span data-ttu-id="135a4-233">Откройте браузер и перейдите по ссылке https://www.zillow.com/howto/api/neighborhood-boundaries.htm.</span><span class="sxs-lookup"><span data-stu-id="135a4-233">Open a web browser and navigate to https://www.zillow.com/howto/api/neighborhood-boundaries.htm.</span></span>

6. <span data-ttu-id="135a4-234">Щелкните **New York Neighborhood Boundaries** (Границы округов штата Нью-Йорк), чтобы скачать файл.</span><span class="sxs-lookup"><span data-stu-id="135a4-234">Click on **New York Neighborhood Boundaries** to download the file.</span></span>

7. <span data-ttu-id="135a4-235">Скопируйте файл **ZillowNeighborhoods-NY.zip** из каталога **загрузок** браузера в каталог `DataFile`.</span><span class="sxs-lookup"><span data-stu-id="135a4-235">Copy the **ZillowNeighborhoods-NY.zip** file from your browser's **downloads** directory to the `DataFile` directory.</span></span>

### <a name="deploy-the-azure-resources"></a><span data-ttu-id="135a4-236">Развертывание ресурсов Azure</span><span class="sxs-lookup"><span data-stu-id="135a4-236">Deploy the Azure resources</span></span>

1. <span data-ttu-id="135a4-237">В оболочке или командной строке Windows выполните следующую команду и следуйте указаниям при запросе на вход:</span><span class="sxs-lookup"><span data-stu-id="135a4-237">From a shell or Windows Command Prompt, run the following command and follow the sign-in prompt:</span></span>

    ```bash
    az login
    ```

2. <span data-ttu-id="135a4-238">Перейдите к папке с именем `azure` в репозитории GitHub.</span><span class="sxs-lookup"><span data-stu-id="135a4-238">Navigate to the folder named `azure` in the GitHub repository:</span></span>

    ```bash
    cd azure
    ```

3. <span data-ttu-id="135a4-239">Выполните следующие команды, чтобы развернуть ресурсы Azure:</span><span class="sxs-lookup"><span data-stu-id="135a4-239">Run the following commands to deploy the Azure resources:</span></span>

    ```bash
    export resourceGroup='[Resource group name]'
    export resourceLocation='[Region]'
    export eventHubNamespace='[Event Hubs namespace name]'
    export databricksWorkspaceName='[Azure Databricks workspace name]'
    export cosmosDatabaseAccount='[Cosmos DB database name]'
    export logAnalyticsWorkspaceName='[Log Analytics workspace name]'
    export logAnalyticsWorkspaceRegion='[Log Analytics region]'

    # Create a resource group
    az group create --name $resourceGroup --location $resourceLocation

    # Deploy resources
    az group deployment create --resource-group $resourceGroup \
        --template-file deployresources.json --parameters \
        eventHubNamespace=$eventHubNamespace \
        databricksWorkspaceName=$databricksWorkspaceName \
        cosmosDatabaseAccount=$cosmosDatabaseAccount \
        logAnalyticsWorkspaceName=$logAnalyticsWorkspaceName \
        logAnalyticsWorkspaceRegion=$logAnalyticsWorkspaceRegion
    ```

4. <span data-ttu-id="135a4-240">По завершении развертывания выходные данные выводятся в консоль.</span><span class="sxs-lookup"><span data-stu-id="135a4-240">The output of the deployment is written to the console once complete.</span></span> <span data-ttu-id="135a4-241">Найдите в выходных данных следующий код JSON:</span><span class="sxs-lookup"><span data-stu-id="135a4-241">Search the output for the following JSON:</span></span>

```json
"outputs": {
        "cosmosDb": {
          "type": "Object",
          "value": {
            "hostName": <value>,
            "secret": <value>,
            "username": <value>
          }
        },
        "eventHubs": {
          "type": "Object",
          "value": {
            "taxi-fare-eh": <value>,
            "taxi-ride-eh": <value>
          }
        },
        "logAnalytics": {
          "type": "Object",
          "value": {
            "secret": <value>,
            "workspaceId": <value>
          }
        }
},
```

<span data-ttu-id="135a4-242">Эти значения являются секретами, которые вы добавите к секретам Databricks в следующих разделах.</span><span class="sxs-lookup"><span data-stu-id="135a4-242">These values are the secrets that will be added to Databricks secrets in upcoming sections.</span></span> <span data-ttu-id="135a4-243">Для этого сохраните их в надежном расположении.</span><span class="sxs-lookup"><span data-stu-id="135a4-243">Keep them secure until you add them in those sections.</span></span>

### <a name="add-a-cassandra-table-to-the-cosmos-db-account"></a><span data-ttu-id="135a4-244">Добавление таблицы Cassandra в учетную запись Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="135a4-244">Add a Cassandra table to the Cosmos DB Account</span></span>

1. <span data-ttu-id="135a4-245">На портале Azure перейдите к группе ресурсов, созданной при работе с предыдущим разделом о **развертывании ресурсов Azure**.</span><span class="sxs-lookup"><span data-stu-id="135a4-245">In the Azure portal, navigate to the resource group created in the **deploy the Azure resources** section above.</span></span> <span data-ttu-id="135a4-246">Щелкните **Учетная запись Azure Cosmos DB**.</span><span class="sxs-lookup"><span data-stu-id="135a4-246">Click on **Azure Cosmos DB Account**.</span></span> <span data-ttu-id="135a4-247">Создайте таблицу с помощью API Cassandra.</span><span class="sxs-lookup"><span data-stu-id="135a4-247">Create a table with the Cassandra API.</span></span>

2. <span data-ttu-id="135a4-248">В колонке **Обзор** щелкните **Добавить таблицу**.</span><span class="sxs-lookup"><span data-stu-id="135a4-248">In the **overview** blade, click **add table**.</span></span>

3. <span data-ttu-id="135a4-249">Когда откроется колонка **Добавление таблицы** введите `newyorktaxi` в текстовое поле **Имя пространства ключей**.</span><span class="sxs-lookup"><span data-stu-id="135a4-249">When the **add table** blade opens, enter `newyorktaxi` in the **Keyspace name** text box.</span></span>

4. <span data-ttu-id="135a4-250">В разделе **ввода команды CQL для создания таблицы** введите `neighborhoodstats` в текстовое поле рядом с `newyorktaxi`.</span><span class="sxs-lookup"><span data-stu-id="135a4-250">In the **enter CQL command to create the table** section, enter `neighborhoodstats` in the text box beside `newyorktaxi`.</span></span>

5. <span data-ttu-id="135a4-251">В текстовом поле ниже введите следующее:</span><span class="sxs-lookup"><span data-stu-id="135a4-251">In the text box below, enter the following:</span></span>
    ```shell
    (neighborhood text, window_end timestamp, number_of_rides bigint,total_fare_amount double, primary key(neighborhood, window_end))
    ```
6. <span data-ttu-id="135a4-252">В текстовое поле **Пропускная способность (1000–1 000 000 единиц запросов в секунду)** введите значение `4000`.</span><span class="sxs-lookup"><span data-stu-id="135a4-252">In the **Throughput (1,000 - 1,000,000 RU/s)** text box enter the value `4000`.</span></span>

7. <span data-ttu-id="135a4-253">Последовательно выберите **ОК**.</span><span class="sxs-lookup"><span data-stu-id="135a4-253">Click **OK**.</span></span>

### <a name="add-the-databricks-secrets-using-the-databricks-cli"></a><span data-ttu-id="135a4-254">Добавление секретов Databricks с помощью интерфейса командной строки Databricks</span><span class="sxs-lookup"><span data-stu-id="135a4-254">Add the Databricks secrets using the Databricks CLI</span></span>

<span data-ttu-id="135a4-255">Сначала введите секреты для концентратора событий.</span><span class="sxs-lookup"><span data-stu-id="135a4-255">First, enter the secrets for EventHub:</span></span>

1. <span data-ttu-id="135a4-256">С помощью **интерфейса командной строки Azure Databricks**, установленном на шаге 2 в разделе с предварительными требованиями, создайте область секретов Azure Databricks:</span><span class="sxs-lookup"><span data-stu-id="135a4-256">Using the **Azure Databricks CLI** installed in step 2 of the prerequisites, create the Azure Databricks secret scope:</span></span>
    ```shell
    databricks secrets create-scope --scope "azure-databricks-job"
    ```
2. <span data-ttu-id="135a4-257">Добавьте секрет для концентратора событий с данными о поездках на такси:</span><span class="sxs-lookup"><span data-stu-id="135a4-257">Add the secret for the taxi ride EventHub:</span></span>
    ```shell
    databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
    ```
    <span data-ttu-id="135a4-258">После завершения эта команда открывает редактор vi.</span><span class="sxs-lookup"><span data-stu-id="135a4-258">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="135a4-259">Введите значение **taxi-ride-eh** из раздела выходных данных **eventHubs**, приведенном на шаге 4 в разделе о *развертывании ресурсов Azure*.</span><span class="sxs-lookup"><span data-stu-id="135a4-259">Enter the **taxi-ride-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="135a4-260">Сохраните файл и закройте редактор vi.</span><span class="sxs-lookup"><span data-stu-id="135a4-260">Save and exit vi.</span></span>

3. <span data-ttu-id="135a4-261">Добавьте секрет для концентратора событий с данными о тарифах на такси:</span><span class="sxs-lookup"><span data-stu-id="135a4-261">Add the secret for the taxi fare EventHub:</span></span>
    ```shell
    databricks secrets put --scope "azure-databricks-job" --key "taxi-fare"
    ```
    <span data-ttu-id="135a4-262">После завершения эта команда открывает редактор vi.</span><span class="sxs-lookup"><span data-stu-id="135a4-262">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="135a4-263">Введите значение **taxi-fare-eh** из раздела выходных данных **eventHubs**, приведенном на шаге 4 в разделе о *развертывании ресурсов Azure*.</span><span class="sxs-lookup"><span data-stu-id="135a4-263">Enter the **taxi-fare-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="135a4-264">Сохраните файл и закройте редактор vi.</span><span class="sxs-lookup"><span data-stu-id="135a4-264">Save and exit vi.</span></span>

<span data-ttu-id="135a4-265">Теперь введите секреты для Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="135a4-265">Next, enter the secrets for Cosmos DB:</span></span>

1. <span data-ttu-id="135a4-266">Откройте портал Azure и перейдите к группе ресурсов, указанной на шаге 3 в разделе о **развертывании ресурсов Azure**.</span><span class="sxs-lookup"><span data-stu-id="135a4-266">Open the Azure portal, and navigate to the resource group specified in step 3 of the **deploy the Azure resources** section.</span></span> <span data-ttu-id="135a4-267">Щелкните "Учетная запись Azure Cosmos DB".</span><span class="sxs-lookup"><span data-stu-id="135a4-267">Click on the Azure Cosmos DB Account.</span></span>

2. <span data-ttu-id="135a4-268">С помощью **интерфейса командной строки Azure Databricks** добавьте секрет для имени пользователя Cosmos DB:</span><span class="sxs-lookup"><span data-stu-id="135a4-268">Using the **Azure Databricks CLI**, add the secret for the Cosmos DB user name:</span></span>
    ```shell
    databricks secrets put --scope azure-databricks-job --key "cassandra-username"
    ```
<span data-ttu-id="135a4-269">После завершения эта команда открывает редактор vi.</span><span class="sxs-lookup"><span data-stu-id="135a4-269">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="135a4-270">Введите значение **username** из раздела выходных данных **CosmosDB**, приведенном на шаге 4 в разделе о *развертывании ресурсов Azure*.</span><span class="sxs-lookup"><span data-stu-id="135a4-270">Enter the **username** value from the **CosmosDb** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="135a4-271">Сохраните файл и закройте редактор vi.</span><span class="sxs-lookup"><span data-stu-id="135a4-271">Save and exit vi.</span></span>

3. <span data-ttu-id="135a4-272">Затем добавьте секрет для пароля Cosmos DB:</span><span class="sxs-lookup"><span data-stu-id="135a4-272">Next, add the secret for the Cosmos DB password:</span></span>
    ```shell
    databricks secrets put --scope azure-databricks-job --key "cassandra-password"
    ```

<span data-ttu-id="135a4-273">После завершения эта команда открывает редактор vi.</span><span class="sxs-lookup"><span data-stu-id="135a4-273">Once executed, this command opens the vi editor.</span></span> <span data-ttu-id="135a4-274">Введите значение **secret** из раздела выходных данных **CosmosDb**, приведенном на шаге 4 в разделе о *развертывании ресурсов Azure*.</span><span class="sxs-lookup"><span data-stu-id="135a4-274">Enter the **secret** value from the **CosmosDb** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="135a4-275">Сохраните файл и закройте редактор vi.</span><span class="sxs-lookup"><span data-stu-id="135a4-275">Save and exit vi.</span></span>

> [!NOTE]
> <span data-ttu-id="135a4-276">Используемая [область секретов, поддерживаемая Azure Key Vault](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes), должна иметь имя **azure-databricks-job**, а имена секретов должны быть точно такими же, как приведенные выше.</span><span class="sxs-lookup"><span data-stu-id="135a4-276">If using an [Azure Key Vault-backed secret scope](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes), the scope must be named **azure-databricks-job** and the secrets must have the exact same names as those above.</span></span>

### <a name="add-the-zillow-neighborhoods-data-file-to-the-databricks-file-system"></a><span data-ttu-id="135a4-277">Добавление файла Zillow с данными об округах в файловую систему Databricks</span><span class="sxs-lookup"><span data-stu-id="135a4-277">Add the Zillow Neighborhoods data file to the Databricks file system</span></span>

1. <span data-ttu-id="135a4-278">Создайте каталог в файловой системе Databricks:</span><span class="sxs-lookup"><span data-stu-id="135a4-278">Create a directory in the Databricks file system:</span></span>
    ```bash
    dbfs mkdirs dbfs:/azure-databricks-jobs
    ```

2. <span data-ttu-id="135a4-279">Перейдите к каталогу `DataFile` и введите следующее:</span><span class="sxs-lookup"><span data-stu-id="135a4-279">Navigate to the `DataFile` directory and enter the following:</span></span>
    ```bash
    dbfs cp ZillowNeighborhoods-NY.zip dbfs:/azure-databricks-jobs
    ```

### <a name="add-the-azure-log-analytics-workspace-id-and-primary-key-to-configuration-files"></a><span data-ttu-id="135a4-280">Добавление идентификатора и первичного ключа рабочей области Azure Log Analytics в файлы конфигурации</span><span class="sxs-lookup"><span data-stu-id="135a4-280">Add the Azure Log Analytics workspace ID and primary key to configuration files</span></span>

<span data-ttu-id="135a4-281">Для выполнения инструкций из этого раздела вам потребуется идентификатор и первичный ключ рабочей области Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="135a4-281">For this section, you require the Log Analytics workspace ID and primary key.</span></span> <span data-ttu-id="135a4-282">Идентификатор рабочей области указан в качестве значения для **workspaceId** в разделе выходных данных **logAnalytics**, приведенном на шаге 4 в разделе о *развертывании ресурсов Azure*.</span><span class="sxs-lookup"><span data-stu-id="135a4-282">The workspace ID is the **workspaceId** value from the **logAnalytics** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="135a4-283">Первичный ключ — это значение для **secret** в разделе выходных данных.</span><span class="sxs-lookup"><span data-stu-id="135a4-283">The primary key is the **secret** from the output section.</span></span>

1. <span data-ttu-id="135a4-284">Чтобы настроить ведение журнала log4j, откройте `\azure\AzureDataBricksJob\src\main\resources\com\microsoft\pnp\azuredatabricksjob\log4j.properties`.</span><span class="sxs-lookup"><span data-stu-id="135a4-284">To configure log4j logging, open `\azure\AzureDataBricksJob\src\main\resources\com\microsoft\pnp\azuredatabricksjob\log4j.properties`.</span></span> <span data-ttu-id="135a4-285">Измените следующие два значения:</span><span class="sxs-lookup"><span data-stu-id="135a4-285">Edit the following two values:</span></span>
    ```shell
    log4j.appender.A1.workspaceId=<Log Analytics workspace ID>
    log4j.appender.A1.secret=<Log Analytics primary key>
    ```

2. <span data-ttu-id="135a4-286">Чтобы настроить пользовательскую систему ведения журналов, откройте `\azure\azure-databricks-monitoring\scripts\metrics.properties`.</span><span class="sxs-lookup"><span data-stu-id="135a4-286">To configure custom logging, open `\azure\azure-databricks-monitoring\scripts\metrics.properties`.</span></span> <span data-ttu-id="135a4-287">Измените следующие два значения:</span><span class="sxs-lookup"><span data-stu-id="135a4-287">Edit the following two values:</span></span>
    ```shell
    *.sink.loganalytics.workspaceId=<Log Analytics workspace ID>
    *.sink.loganalytics.secret=<Log Analytics primary key>
    ```

### <a name="build-the-jar-files-for-the-databricks-job-and-databricks-monitoring"></a><span data-ttu-id="135a4-288">Создание JAR-файлов для задания и мониторинга Databricks</span><span class="sxs-lookup"><span data-stu-id="135a4-288">Build the .jar files for the Databricks job and Databricks monitoring</span></span>

1. <span data-ttu-id="135a4-289">Используйте интегрированную среду разработки Java для импорта файла проекта Maven с именем **pom.xml**, расположенного в корневом каталоге.</span><span class="sxs-lookup"><span data-stu-id="135a4-289">Use your Java IDE to import the Maven project file named **pom.xml** located in the root directory.</span></span>

2. <span data-ttu-id="135a4-290">Выполните чистую сборку.</span><span class="sxs-lookup"><span data-stu-id="135a4-290">Perform a clean build.</span></span> <span data-ttu-id="135a4-291">Результатом этой сборки будет файл **azure-databricks-job-1.0-SNAPSHOT.jar** и **azure-databricks-monitoring-0.9.jar**.</span><span class="sxs-lookup"><span data-stu-id="135a4-291">The output of this build is files named **azure-databricks-job-1.0-SNAPSHOT.jar** and **azure-databricks-monitoring-0.9.jar**.</span></span>

### <a name="configure-custom-logging-for-the-databricks-job"></a><span data-ttu-id="135a4-292">Настройка пользовательской системы ведения журналов для задания Databricks</span><span class="sxs-lookup"><span data-stu-id="135a4-292">Configure custom logging for the Databricks job</span></span>

1. <span data-ttu-id="135a4-293">Скопируйте файл **azure-databricks-monitoring-0.9.jar** в файловую систему Databricks. Для этого введите следующую команду в **интерфейсе командной строки Databricks**:</span><span class="sxs-lookup"><span data-stu-id="135a4-293">Copy the **azure-databricks-monitoring-0.9.jar** file to the Databricks file system by entering the following command in the **Databricks CLI**:</span></span>
    ```shell
    databricks fs cp --overwrite azure-databricks-monitoring-0.9.jar dbfs:/azure-databricks-job/azure-databricks-monitoring-0.9.jar
    ```

2. <span data-ttu-id="135a4-294">Скопируйте свойства пользовательской системы ведения журналов из `\azure\azure-databricks-monitoring\scripts\metrics.properties` в файловую систему Databricks. Для этого введите следующую команду:</span><span class="sxs-lookup"><span data-stu-id="135a4-294">Copy the custom logging properties from `\azure\azure-databricks-monitoring\scripts\metrics.properties` to the Databricks file system by entering the following command:</span></span>
    ```shell
    databricks fs cp --overwrite metrics.properties dbfs:/azure-databricks-job/metrics.properties
    ```

3. <span data-ttu-id="135a4-295">Если вы до этого не определились с именем для кластера Databricks, выберите его сейчас.</span><span class="sxs-lookup"><span data-stu-id="135a4-295">While you haven't yet decided on a name for your Databricks cluster, select one now.</span></span> <span data-ttu-id="135a4-296">Вам потребуется указать его в приведенном ниже пути к кластеру в файловой системе Databricks.</span><span class="sxs-lookup"><span data-stu-id="135a4-296">You'll enter the name below in the Databricks file system path for your cluster.</span></span> <span data-ttu-id="135a4-297">Скопируйте скрипт инициализации из `\azure\azure-databricks-monitoring\scripts\spark.metrics` в файловую систему Databricks. Для этого введите следующую команду:</span><span class="sxs-lookup"><span data-stu-id="135a4-297">Copy the initialization script from `\azure\azure-databricks-monitoring\scripts\spark.metrics` to the Databricks file system by entering the following command:</span></span>
    ```shell
    databricks fs cp --overwrite spark-metrics.sh dbfs:/databricks/init/<cluster-name>/spark-metrics.sh
    ```

### <a name="create-a-databricks-cluster"></a><span data-ttu-id="135a4-298">Создание кластера Databricks</span><span class="sxs-lookup"><span data-stu-id="135a4-298">Create a Databricks cluster</span></span>

1. <span data-ttu-id="135a4-299">В рабочей области Databricks щелкните Clusters (Кластеры) и выберите Create cluster (Создать кластер).</span><span class="sxs-lookup"><span data-stu-id="135a4-299">In the Databricks workspace, click "Clusters", then click "create cluster".</span></span> <span data-ttu-id="135a4-300">Введите имя кластера, созданного на шаге 3 в предыдущем разделе о **настройке пользовательской системы ведения журналов для задания Databricks**.</span><span class="sxs-lookup"><span data-stu-id="135a4-300">Enter the cluster name you created in step 3 of the **configure custom logging for the Databricks job** section above.</span></span>

2. <span data-ttu-id="135a4-301">Выберите **стандартный** режим кластера.</span><span class="sxs-lookup"><span data-stu-id="135a4-301">Select a **standard** cluster mode.</span></span>

3. <span data-ttu-id="135a4-302">Для **версии среды выполнения Databricks** укажите **4.3 (содержит Apache Spark 2.3.1, Scala 2.11)**.</span><span class="sxs-lookup"><span data-stu-id="135a4-302">Set **Databricks runtime version** to **4.3 (includes Apache Spark 2.3.1, Scala 2.11)**</span></span>

4. <span data-ttu-id="135a4-303">Для **версии Python** укажите **2**.</span><span class="sxs-lookup"><span data-stu-id="135a4-303">Set **Python version** to **2**.</span></span>

5. <span data-ttu-id="135a4-304">Укажите для **типа драйвера** значение **Same as worker** (Как для рабочего узла).</span><span class="sxs-lookup"><span data-stu-id="135a4-304">Set **Driver Type** to **Same as worker**</span></span>

6. <span data-ttu-id="135a4-305">Для **Worker Type** (Тип рабочего узла) укажите значение **Standard_DS3_v2**.</span><span class="sxs-lookup"><span data-stu-id="135a4-305">Set **Worker Type** to **Standard_DS3_v2**.</span></span>

7. <span data-ttu-id="135a4-306">Для параметра **Min Workers** (Минимальное количество рабочих узлов) задайте значение **2**.</span><span class="sxs-lookup"><span data-stu-id="135a4-306">Set **Min Workers** to **2**.</span></span>

8. <span data-ttu-id="135a4-307">Снимите флажок **Enable autoscaling** (Включить автомасштабирование).</span><span class="sxs-lookup"><span data-stu-id="135a4-307">Deselect **Enable autoscaling**.</span></span>

9. <span data-ttu-id="135a4-308">В диалоговом окне **Auto Termination** (Автоматическое завершение работы) щелкните **Init Scripts** (Скрипты инициализации).</span><span class="sxs-lookup"><span data-stu-id="135a4-308">Below the **Auto Termination** dialog box, click on **Init Scripts**.</span></span>

10. <span data-ttu-id="135a4-309">Введите **dbfs:/databricks/init/<cluster-name>/spark-metrics.sh** и вместо <cluster-name> укажите имя кластера, созданного на шаге 1.</span><span class="sxs-lookup"><span data-stu-id="135a4-309">Enter **dbfs:/databricks/init/<cluster-name>/spark-metrics.sh**, substituting the cluster name created in step 1 for <cluster-name>.</span></span>

11. <span data-ttu-id="135a4-310">Нажмите кнопку **Add** (Добавить).</span><span class="sxs-lookup"><span data-stu-id="135a4-310">Click the **Add** button.</span></span>

12. <span data-ttu-id="135a4-311">Нажмите кнопку **Create Cluster** (Создать кластер).</span><span class="sxs-lookup"><span data-stu-id="135a4-311">Click the **Create Cluster** button.</span></span>

### <a name="create-a-databricks-job"></a><span data-ttu-id="135a4-312">Создание задания Databricks</span><span class="sxs-lookup"><span data-stu-id="135a4-312">Create a Databricks job</span></span>

1. <span data-ttu-id="135a4-313">В рабочей области Databricks выберите Jobs (Задания) и Create job (Создать задание).</span><span class="sxs-lookup"><span data-stu-id="135a4-313">In the Databricks workspace, click "Jobs", "create job".</span></span>

2. <span data-ttu-id="135a4-314">Введите имя задания.</span><span class="sxs-lookup"><span data-stu-id="135a4-314">Enter a job name.</span></span>

3. <span data-ttu-id="135a4-315">Если щелкнуть set jar (Указать JAR-файл), откроется диалоговое окно Upload JAR to Run (Отправить JAR-файл для запуска).</span><span class="sxs-lookup"><span data-stu-id="135a4-315">Click "set jar", this opens the "Upload JAR to Run" dialog box.</span></span>

4. <span data-ttu-id="135a4-316">Перетащите файл **azure-databricks-job-1.0-SNAPSHOT.jar**, который вы создали при работе с разделом о **создании JAR-файлов для задания и мониторинга Databricks**, в поле **Drop JAR here to upload** (Перетащите сюда JAR-файл для отправки).</span><span class="sxs-lookup"><span data-stu-id="135a4-316">Drag the **azure-databricks-job-1.0-SNAPSHOT.jar** file you created in the **build the .jar for the Databricks job** section to the **Drop JAR here to upload** box.</span></span>

5. <span data-ttu-id="135a4-317">Укажите **com.microsoft.pnp.TaxiCabReader** в поле **Main Class** (Класс Main).</span><span class="sxs-lookup"><span data-stu-id="135a4-317">Enter **com.microsoft.pnp.TaxiCabReader** in the **Main Class** field.</span></span>

6. <span data-ttu-id="135a4-318">В поле аргументов введите следующий код:</span><span class="sxs-lookup"><span data-stu-id="135a4-318">In the arguments field, enter the following:</span></span>
    ```shell
    -n jar:file:/dbfs/azure-databricks-jobs/ZillowNeighborhoods-NY.zip!/ZillowNeighborhoods-NY.shp --taxi-ride-consumer-group taxi-ride-eh-cg --taxi-fare-consumer-group taxi-fare-eh-cg --window-interval "1 minute" --cassandra-host <Cosmos DB Cassandra host name from above>
    ```

7. <span data-ttu-id="135a4-319">Установите зависимые библиотеки, выполнив следующие действия:</span><span class="sxs-lookup"><span data-stu-id="135a4-319">Install the dependent libraries by following these steps:</span></span>

    1. <span data-ttu-id="135a4-320">В пользовательском интерфейсе Databricks нажмите кнопку **Home** (Главная).</span><span class="sxs-lookup"><span data-stu-id="135a4-320">In the Databricks user interface, click on the **home** button.</span></span>

    2. <span data-ttu-id="135a4-321">В раскрывающемся списке **Users** (Пользователи) щелкните имя учетной записи пользователя, чтобы открыть настройки ее рабочей области.</span><span class="sxs-lookup"><span data-stu-id="135a4-321">In the **Users** drop-down, click on your user account name to open your account workspace settings.</span></span>

    3. <span data-ttu-id="135a4-322">Щелкните стрелку раскрывающегося списка рядом с именем вашей учетной записи, выберите **Create** (Создать) и **Library** (Библиотека), чтобы открыть диалоговое окно **New Library** (Новая библиотека).</span><span class="sxs-lookup"><span data-stu-id="135a4-322">Click on the drop-down arrow beside your account name, click on **create**, and click on **Library** to open the **New Library** dialog.</span></span>

    4. <span data-ttu-id="135a4-323">В раскрывающемся элементе управления **Source** (Источник) выберите **Maven Coordinate** (Координаты Maven).</span><span class="sxs-lookup"><span data-stu-id="135a4-323">In the **Source** drop-down control, select **Maven Coordinate**.</span></span>

    5. <span data-ttu-id="135a4-324">В разделе **Install Maven Artifacts** (Установка артефактов Maven) введите `com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.5` в текстовое поле **Coordinate** (Координаты).</span><span class="sxs-lookup"><span data-stu-id="135a4-324">Under the **Install Maven Artifacts** heading, enter `com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.5` in the **Coordinate** text box.</span></span>

    6. <span data-ttu-id="135a4-325">Щелкните **Create Library** (Создать библиотеку), чтобы открыть окно **артефактов**.</span><span class="sxs-lookup"><span data-stu-id="135a4-325">Click on **Create Library** to open the **Artifacts** window.</span></span>

    7. <span data-ttu-id="135a4-326">В разделе **Status on running clusters** (Состояние работающих кластеров) установите флажок **Attach automatically to all clusters** (Автоматически подключать ко всем кластерам).</span><span class="sxs-lookup"><span data-stu-id="135a4-326">Under **Status on running clusters** check the **Attach automatically to all clusters** checkbox.</span></span>

    8. <span data-ttu-id="135a4-327">Повторите шаги 1–7 для координат Maven `com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0`.</span><span class="sxs-lookup"><span data-stu-id="135a4-327">Repeat steps 1 - 7 for the `com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0` Maven coordinate.</span></span>

    9. <span data-ttu-id="135a4-328">Повторите шаги 1–6 для координат Maven `org.geotools:gt-shapefile:19.2`.</span><span class="sxs-lookup"><span data-stu-id="135a4-328">Repeat steps 1 - 6 for the `org.geotools:gt-shapefile:19.2` Maven coordinate.</span></span>

    10. <span data-ttu-id="135a4-329">Щелкните **Advanced Options** (Дополнительные параметры).</span><span class="sxs-lookup"><span data-stu-id="135a4-329">Click on **Advanced Options**.</span></span>

    11. <span data-ttu-id="135a4-330">Введите `http://download.osgeo.org/webdav/geotools/` в текстовое поле для **репозитория**.</span><span class="sxs-lookup"><span data-stu-id="135a4-330">Enter `http://download.osgeo.org/webdav/geotools/` in the **Repository** text box.</span></span>

    12. <span data-ttu-id="135a4-331">Щелкните **Create Library** (Создать библиотеку), чтобы открыть окно **артефактов**.</span><span class="sxs-lookup"><span data-stu-id="135a4-331">Click **Create Library** to open the **Artifacts** window.</span></span> 

    13. <span data-ttu-id="135a4-332">В разделе **Status on running clusters** (Состояние работающих кластеров) установите флажок **Attach automatically to all clusters** (Автоматически подключать ко всем кластерам).</span><span class="sxs-lookup"><span data-stu-id="135a4-332">Under **Status on running clusters** check the **Attach automatically to all clusters** checkbox.</span></span>

8. <span data-ttu-id="135a4-333">Добавьте зависимые библиотеки, добавленные на шаге 7, в задание, которое вы создали в конце шага 6:</span><span class="sxs-lookup"><span data-stu-id="135a4-333">Add the dependent libraries added in step 7 to the job created at the end of step 6:</span></span>

    1. <span data-ttu-id="135a4-334">В рабочей области Azure Databricks выберите **Jobs** (Задания).</span><span class="sxs-lookup"><span data-stu-id="135a4-334">In the Azure Databricks workspace, click on **Jobs**.</span></span>

    2. <span data-ttu-id="135a4-335">Щелкните имя задания, созданного на шаге 2 в разделе о **создании задания Databricks**.</span><span class="sxs-lookup"><span data-stu-id="135a4-335">Click on the job name created in step 2 of the **create a Databricks job** section.</span></span>

    3. <span data-ttu-id="135a4-336">В разделе **зависимых библиотек** щелкните **Add** (Добавить), чтобы открыть диалоговое окно **Add Dependent Library** (Добавление зависимой библиотеки).</span><span class="sxs-lookup"><span data-stu-id="135a4-336">Beside the **Dependent Libraries** section, click on **Add** to open the **Add Dependent Library** dialog.</span></span>

    4. <span data-ttu-id="135a4-337">В разделе **Library From** (Источник библиотеки) выберите **Workspace** (Рабочая область).</span><span class="sxs-lookup"><span data-stu-id="135a4-337">Under **Library From** select **Workspace**.</span></span>

    5. <span data-ttu-id="135a4-338">Щелкните **Users** (Пользователи), выберите свое имя пользователя и щелкните `azure-eventhubs-spark_2.11:2.3.5`.</span><span class="sxs-lookup"><span data-stu-id="135a4-338">Click on **users**, then your username, then click on `azure-eventhubs-spark_2.11:2.3.5`.</span></span>

    6. <span data-ttu-id="135a4-339">Последовательно выберите **ОК**.</span><span class="sxs-lookup"><span data-stu-id="135a4-339">Click **OK**.</span></span>

    7. <span data-ttu-id="135a4-340">Повторите шаги 1–6 для `spark-cassandra-connector_2.11:2.3.1` и `gt-shapefile:19.2`.</span><span class="sxs-lookup"><span data-stu-id="135a4-340">Repeat steps 1 - 6 for `spark-cassandra-connector_2.11:2.3.1` and `gt-shapefile:19.2`.</span></span>

9. <span data-ttu-id="135a4-341">Рядом с полем **Cluster:** (Кластер) щелкните **Edit** (Изменить).</span><span class="sxs-lookup"><span data-stu-id="135a4-341">Beside **Cluster:**, click on **Edit**.</span></span> <span data-ttu-id="135a4-342">Откроется диалоговое окно **настройки кластера**.</span><span class="sxs-lookup"><span data-stu-id="135a4-342">This opens the **Configure Cluster** dialog.</span></span> <span data-ttu-id="135a4-343">В раскрывающемся списке **Cluster Type** (Тип кластера) выберите **Existing Cluster** (Существующий кластер).</span><span class="sxs-lookup"><span data-stu-id="135a4-343">In the **Cluster Type** drop-down, select **Existing Cluster**.</span></span> <span data-ttu-id="135a4-344">В раскрывающемся списке **Select Cluster** (Выбрать кластер) выберите кластер, созданный при работе с разделом **Создание кластера Databricks**.</span><span class="sxs-lookup"><span data-stu-id="135a4-344">In the **Select Cluster** drop-down, select the cluster created the **create a Databricks cluster** section.</span></span> <span data-ttu-id="135a4-345">Щелкните **Confirm** (Подтвердить).</span><span class="sxs-lookup"><span data-stu-id="135a4-345">Click **confirm**.</span></span>

10. <span data-ttu-id="135a4-346">Щелкните **Run now** (Запустить сейчас).</span><span class="sxs-lookup"><span data-stu-id="135a4-346">Click **run now**.</span></span>

### <a name="run-the-data-generator"></a><span data-ttu-id="135a4-347">Запуск генератора данных</span><span class="sxs-lookup"><span data-stu-id="135a4-347">Run the data generator</span></span>

1. <span data-ttu-id="135a4-348">Перейдите к каталогу с именем `onprem` в репозитории GitHub.</span><span class="sxs-lookup"><span data-stu-id="135a4-348">Navigate to the directory named `onprem` in the GitHub repository.</span></span>

2. <span data-ttu-id="135a4-349">Обновите значения в файле **main.env** следующим образом:</span><span class="sxs-lookup"><span data-stu-id="135a4-349">Update the values in the file **main.env** as follows:</span></span>

    ```shell
    RIDE_EVENT_HUB=[Connection string for the taxi-ride event hub]
    FARE_EVENT_HUB=[Connection string for the taxi-fare event hub]
    RIDE_DATA_FILE_PATH=/DataFile/FOIL2013
    MINUTES_TO_LEAD=0
    PUSH_RIDE_DATA_FIRST=false
    ```
    <span data-ttu-id="135a4-350">Строка подключения к концентратору событий с данными о поездках на такси указана в качестве значение **taxi-ride-eh** из раздела выходных данных **eventHubs**, приведенном на шаге 4 в разделе о *развертывании ресурсов Azure*.</span><span class="sxs-lookup"><span data-stu-id="135a4-350">The connection string for the taxi-ride event hub is the **taxi-ride-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span> <span data-ttu-id="135a4-351">Строка подключения к концентратору событий с данными о тарифах на такси указана в качестве значение **taxi-fare-eh** из раздела выходных данных **eventHubs**, приведенном на шаге 4 в разделе о *развертывании ресурсов Azure*.</span><span class="sxs-lookup"><span data-stu-id="135a4-351">The connection string for the taxi-fare event hub the **taxi-fare-eh** value from the **eventHubs** output section in step 4 of the *deploy the Azure resources* section.</span></span>

3. <span data-ttu-id="135a4-352">Чтобы создать образ Docker, выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="135a4-352">Run the following command to build the Docker image.</span></span>

    ```bash
    docker build --no-cache -t dataloader .
    ```

4. <span data-ttu-id="135a4-353">Вернитесь к родительскому каталогу.</span><span class="sxs-lookup"><span data-stu-id="135a4-353">Navigate back to the parent directory.</span></span>

    ```bash
    cd ..
    ```

5. <span data-ttu-id="135a4-354">Чтобы запустить образ Docker, выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="135a4-354">Run the following command to run the Docker image.</span></span>

    ```bash
    docker run -v `pwd`/DataFile:/DataFile --env-file=onprem/main.env dataloader:latest
    ```

<span data-ttu-id="135a4-355">Полученный результат должен выглядеть примерно так:</span><span class="sxs-lookup"><span data-stu-id="135a4-355">The output should look like the following:</span></span>

```
Created 10000 records for TaxiFare
Created 10000 records for TaxiRide
Created 20000 records for TaxiFare
Created 20000 records for TaxiRide
Created 30000 records for TaxiFare
...
```

<span data-ttu-id="135a4-356">Чтобы проверить, правильно ли выполняется задание Databricks, откройте портал Azure и перейдите к базе данных Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="135a4-356">To verify the Databricks job is running correctly, open the Azure portal and navigate to the Cosmos DB database.</span></span> <span data-ttu-id="135a4-357">Откройте колонку **Обозреватель данных** и просмотрите данные в таблице **taxi records**.</span><span class="sxs-lookup"><span data-stu-id="135a4-357">Open the **Data Explorer** blade and examine the data in the **taxi records** table.</span></span> 

<span data-ttu-id="135a4-358">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013) (Брайан Донован, Дэн Уорк, 2016. Данные о поездках в такси по Нью-Йорку за 2010–2013 гг.).</span><span class="sxs-lookup"><span data-stu-id="135a4-358">[1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013).</span></span> <span data-ttu-id="135a4-359">Иллинойсский университет в Урбане-Шампейне.</span><span class="sxs-lookup"><span data-stu-id="135a4-359">University of Illinois at Urbana-Champaign.</span></span> <span data-ttu-id="135a4-360">https://doi.org/10.13012/J8PN93H8</span><span class="sxs-lookup"><span data-stu-id="135a4-360">https://doi.org/10.13012/J8PN93H8</span></span>
