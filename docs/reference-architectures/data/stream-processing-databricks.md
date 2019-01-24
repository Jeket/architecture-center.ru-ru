---
title: Обработка потоков данных с помощью Azure Databricks
titleSuffix: Azure Reference Architectures
description: Создание сквозного конвейера обработки потоков данных в Azure с помощью Azure Databricks.
author: petertaylor9999
ms.date: 11/30/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18
ms.openlocfilehash: 748b191aeee931d580dd27b1ad54c4f4bd63e369
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54483706"
---
# <a name="create-a-stream-processing-pipeline-with-azure-databricks"></a>Создание конвейера обработки потоков данных с помощью Azure Databricks

На схеме эталонной архитектуры представлен сквозной конвейер [обработки потоков данных](/azure/architecture/data-guide/big-data/real-time-processing). Конвейер такого типа состоит из четырех этапов: прием, обработка, сохранение, анализ и создание отчета. В этой эталонной архитектуре конвейер принимает данные из двух источников, объединяет связанные записи из каждого потока, дополняет результат и вычисляет среднее значение в реальном времени. Результаты сохраняются для дальнейшего анализа. [**Разверните это решение**](#deploy-the-solution).

![Эталонная архитектура для потоковой обработки с помощью Azure Databricks](./images/stream-processing-databricks.png)

**Сценарий.** Компания, предоставляющая услуги такси, собирает данные о каждой поездке. В этом сценарии предполагается, что данные отправляются с двух отдельных устройств. В такси установлен счетчик, который отправляет данные о каждой поездке, включая сведения о продолжительности, расстоянии, а также местах посадки и высадки. Отдельное устройство принимает платежи от клиентов и отправляет данные о тарифах. Чтобы определить тенденции роста пассажиропотока, компании нужно для каждого округа вычислить среднюю сумму чаевых на милю в реальном времени.

## <a name="architecture"></a>Архитектура

Архитектура состоит из следующих компонентов:

**Источники данных.** В этой архитектуре существует два источника данных, которые создают потоки данных в реальном времени. Первый поток содержит сведения о поездке, а второй — о тарифах. В эталонной архитектуре есть имитированный генератор данных, который считывает данные из набора статических файлов и отправляет данные в Центры событий. В реальном приложении источниками данных будут устройства, установленные в такси.

**Центры событий Azure**. [Центры событий](/azure/event-hubs/) — это служба приема событий. В этой архитектуре используется два экземпляра службы — по одной на каждый источник данных. Каждый источник данных отправляет поток данных в соответствующую службу.

**Azure Databricks**. [Databricks](/azure/azure-databricks/) — это платформа аналитики на основе Apache Spark, оптимизированная для платформы облачных служб Microsoft Azure. Databricks используется для корреляции данных о поездке на такси и тарифах, а также для дополнения коррелированных данных сведениями об округе, хранящимися в файловой системе Databricks.

**Cosmos DB**. Выходные данные задания Azure Databricks будут представлять собой последовательность записей, которые вносятся в [Cosmos DB](/azure/cosmos-db/) с помощью API Cassandra. Мы используем API Cassandra, потому что этот интерфейс поддерживает моделирование данных временных рядов.

**Azure Log Analytics**. Данные журнала приложений, собранные [Azure Monitor](/azure/monitoring-and-diagnostics/), хранятся в [рабочей области Log Analytics](/azure/log-analytics). Вы можете использовать запросы Log Analytics для анализа и визуализации метрик, а также просмотра сообщений журнала, чтобы выявить проблемы в приложении.

## <a name="data-ingestion"></a>Прием данных

<!-- markdownlint-disable MD033 -->

Для имитации источника данных в этой эталонной архитектуре используется набор данных<sup>[[1]](#note1)</sup> [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) (Данные о поездках в такси в Нью-Йорке). Этот набор содержит данные о поездках в такси в Нью-Йорке за 4 года (2010&ndash;2013). Он содержит два типа записей: данные о поездке и данные о тарифе. Данные о поездках включают сведения о продолжительности поездки, расстоянии, а также местах посадки и высадки. Данные о тарифах включают сведения о тарифе, налоге и сумме чаевых. В обоих типах записей есть стандартные поля: номер медальона, лицензия на право вождения и код организации. Вместе эти три поля позволяют уникально идентифицировать такси и водителя. Данные хранятся в формате CSV.

> [1] <span id="note1">Donovan, Brian; Work, Dan (2016): New York City Taxi Trip Data (2010-2013) (Брайан Донован, Дэн Уорк, 2016. Данные о поездках в такси по Нью-Йорку за 2010–2013 гг.). Иллинойсский университет в Урбане-Шампейне. <https://doi.org/10.13012/J8PN93H8>

<!-- markdownlint-enable MD033 -->

Генератор данных — это приложение .NET Core, которое считывает записи и отправляет их в Центры событий Azure. Генератор отправляет данные о поездке в формате JSON, а данные о тарифах — в формате CSV.

Для сегментации данных Центры событий используют [секции](/azure/event-hubs/event-hubs-features#partitions). Они позволяют объекту-получателю считывать данные каждой секции параллельно. При отправке данных в Центры событий можно явно указать ключ секции. В противном случае записи назначаются секциям методом циклического перебора.

В этом примере данные о поездках и тарифах должны в итоге иметь одинаковый идентификатор секции для определенного такси. Это позволит Databricks применить определенную степень параллелизма при корреляции двух потоков. Запись в секции *n* с данными о поездке будет соответствовать записи в секции *n* с данными о тарифах.

![Схема потоковой обработки с помощью Azure Databricks и Центров событий Azure](./images/stream-processing-databricks-eh.png)

В генераторе данных общая модель данных для обоих типов записей имеет свойство `PartitionKey`, в котором объединены `Medallion`, `HackLicense` и `VendorId`.

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

Это свойство используется для явного предоставления ключа секции при отправке в Центры событий:

```csharp
using (var client = pool.GetObject())
{
    return client.Value.SendAsync(new EventData(Encoding.UTF8.GetBytes(
        t.GetData(dataFormat))), t.PartitionKey);
}
```

### <a name="event-hubs"></a>Центры событий;

Пропускная способность Центров событий вычисляется в [единицах пропускной способности](/azure/event-hubs/event-hubs-features#throughput-units). Вы можете автоматически масштабировать концентратор событий, включив [автоматическое расширение](/azure/event-hubs/event-hubs-auto-inflate). Это позволит автоматически масштабировать единицы пропускной способности в зависимости от трафика вплоть до заданного максимума.

## <a name="stream-processing"></a>Потоковая обработка

В Azure Databricks обработка данных осуществляется с помощью задания. Задание назначается определенному кластеру и выполняется в нем. Задание может представлять собой пользовательский код на Java или [записную книжку](https://docs.databricks.com/user-guide/notebooks/index.html) Spark.

В этой эталонной архитектуре задание представляет собой архив Java с классами, написанными на Java и Scala. Указывая архив Java для задания Databricks, нужно выбрать класс для выполнения в кластере Databricks. В этом примере метод **main** класса **com.microsoft.pnp.TaxiCabReader** содержит логику обработки данных.

### <a name="reading-the-stream-from-the-two-event-hub-instances"></a>Считывание потоков данных из двух экземпляров концентратора событий

Для считывания данных из двух экземпляров концентратора событий Azure в логике обработки данных используется [Spark Structured Streaming](https://spark.apache.org/docs/2.1.2/structured-streaming-programming-guide.html).

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

### <a name="enriching-the-data-with-the-neighborhood-information"></a>Дополнение данных сведениями об округе

Данные о поездке включают координаты широты и долготы и сведения о местах посадки и высадки. Хотя эти координаты и полезны, их неудобно использовать для анализа. Таким образом, эти данные дополняются сведениями об округе, считанными из [файла фигуры](https://en.wikipedia.org/wiki/Shapefile).

Файл фигуры имеет двоичный формат, что усложняет анализ. Но в библиотеке [GeoTools](http://geotools.org/) представлены средства для работы с геопространственными данными в формате файла фигуры. Эта библиотека используется в классе **com.microsoft.pnp.GeoFinder**, чтобы определить название округа на основе координат мест посадки и высадки.

```scala
val neighborhoodFinder = (lon: Double, lat: Double) => {
      NeighborhoodFinder.getNeighborhood(lon, lat).get()
    }
```

### <a name="joining-the-ride-and-fare-data"></a>Объединение данных о поездках и тарифах

Сначала выполняется преобразование данных о поездках и тарифах:

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

После этого происходит объединение данных о поездке с данными о тарифах:

```scala
val mergedTaxiTrip = rides.join(fares, Seq("medallion", "hackLicense", "vendorId", "pickupTime"))
```

### <a name="processing-the-data-and-inserting-into-cosmos-db"></a>Обработка и вставка данных в Cosmos DB

Вычисляется средний тариф для каждого округа за определенный интервал времени:

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

Затем результат вставляется в Cosmos DB:

```scala
maxAvgFarePerNeighborhood
      .writeStream
      .queryName("maxAvgFarePerNeighborhood_cassandra_insert")
      .outputMode(OutputMode.Append())
      .foreach(new CassandraSinkForeach(connector))
      .start()
      .awaitTermination()
```

## <a name="security-considerations"></a>Вопросы безопасности

Управление доступом к рабочей области базы данных Azure осуществляется с помощью [консоли администрирования](https://docs.databricks.com/administration-guide/admin-settings/index.html). В консоли администрирования предусмотрены функции для добавления пользователей, управления их разрешениями и настройки единого входа. Также в ней можно настроить управление доступом для рабочих областей, кластеров, заданий и таблиц.

### <a name="managing-secrets"></a>управление секретами;

В Azure Databricks предусмотрено [хранилище секретов](https://docs.azuredatabricks.net/user-guide/secrets/index.html), которое используется для хранения не только секретов, но и строк подключения, ключей доступа, имен пользователей и паролей. Секреты в хранилище секретов Azure Databricks секционируются по **областям**:

```bash
databricks secrets create-scope --scope "azure-databricks-job"
```

Секреты добавляются на уровне области:

```bash
databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
```

> [!NOTE]
> Вы можете использовать области, поддерживаемые Azure Key Vault, вместо стандартных областей Azure Databricks. Дополнительные сведения см. в документации по [областям, поддерживаемым Azure Key Vault](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes).

В коде для получения доступа к секретам используются [соответствующие служебные программы](https://docs.databricks.com/user-guide/dev-tools/dbutils.html#secrets-utilities) Azure Databricks.

## <a name="monitoring-considerations"></a>Рекомендации по мониторингу

Платформа Azure Databricks создана на основе Apache Spark и тоже использует [log4j](https://logging.apache.org/log4j/2.x/) в качестве стандартной библиотеки для ведения журналов. Кроме использования стандартных возможностей ведения журнала, предоставляемых Apache Spark, в этой эталонной архитектуре предусматривается отправка журналов и метрик в [Azure Log Analytics](/azure/log-analytics/).

Используя класс **com.microsoft.pnp.TaxiCabReader** и значения в файле **log4j.properties** вы можете настроить систему ведения журналов Apache Spark для отправки журналов в Azure Log Analytics. Хотя сообщения средства ведения журнала Apache Spark представлены в строковом формате, для Azure Log Analytics требуется, чтобы эти сообщения были в формате JSON. Класс **com.microsoft.pnp.log4j.LogAnalyticsAppender** позволяет преобразовать такие сообщения в формат JSON:

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

В результате обработки сообщений о поездках и тарифах с помощью класса **com.microsoft.pnp.TaxiCabReader** некоторые сообщения могу иметь неправильный формат и поэтому быть недопустимыми. В рабочей среде нужно проанализировать такие сообщения, чтобы выявить проблему с источниками данных и быстро решить ее для предотвращения потери данных. Класс **com.microsoft.pnp.TaxiCabReader** позволяет зарегистрировать накопитель Apache Spark, который отслеживает количество записей о поездках и тарифах в неправильном формате.

```scala
    @transient val appMetrics = new AppMetrics(spark.sparkContext)
    appMetrics.registerGauge("metrics.malformedrides", AppAccumulators.getRideInstance(spark.sparkContext))
    appMetrics.registerGauge("metrics.malformedfares", AppAccumulators.getFareInstance(spark.sparkContext))
    SparkEnv.get.metricsSystem.registerSource(appMetrics)
```

Для отправки метрик Apache Spark использует библиотеку Dropwizard. Но некоторые стандартные поля метрик Dropwizard несовместимы с Azure Log Analytics. Поэтому в эталонной архитектуре предусмотрены пользовательские приемник и средство формирования отчетов Dropwizard. Это позволяет преобразовать метрики в формат, требуемый для Azure Log Analytics. Apache Spark передает метрики вместе с пользовательскими метриками для данных о поездках и тарифах неправильного формата.

Запись последней метрики в журнал рабочей области Azure Log Analytics состоит из нескольких процессов в рамках выполняемого задания Spark Structured Streaming. Эта задача выполняется с помощью прослушивателя, реализованного в виде класса **com.microsoft.pnp.StreamingMetricsListener**. Этот класс регистрируется в сеансе Apache Spark при запуске задания:

```scala
spark.streams.addListener(new StreamingMetricsListener())
```

В среде выполнения Apache Spark методы в StreamingMetricsListener вызываются при каждом событии структурированной потоковой передачи. При этом также отправляются сообщения и метрики журнала в рабочую область Azure Log Analytics. Для мониторинга приложения вы можете использовать следующие запросы в своей рабочей области:

### <a name="latency-and-throughput-for-streaming-queries"></a>Задержка и пропускная способность для запросов потоковой передачи

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| project  mdc_inputRowsPerSecond_d, mdc_durationms_triggerExecution_d
| render timechart
```

### <a name="exceptions-logged-during-stream-query-execution"></a>Исключения, зарегистрированные при выполнении запроса потоковой передачи

```shell
taxijob_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| where Level contains "Error"
```

### <a name="accumulation-of-malformed-fare-and-ride-data"></a>Сбор данных неправильного формата о поездках и тарифах

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

### <a name="job-execution-to-trace-resiliency"></a>Данные о выполнении задания для трассировки устойчивости

```shell
SparkMetric_CL
| where TimeGenerated > startofday(datetime(<date>)) and TimeGenerated < endofday(datetime(<date>))
| render timechart
| where name_s contains "driver.DAGScheduler.job.allJobs"
```

## <a name="deploy-the-solution"></a>Развертывание решения

Чтобы выполнить развертывание и запуск эталонной реализации, выполните действия, описанные в [файле сведений на GitHub](https://github.com/mspnp/azure-databricks-streaming-analytics).
