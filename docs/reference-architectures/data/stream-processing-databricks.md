---
title: Обработка потоков данных с помощью Azure Databricks
description: Создание сквозного конвейера обработки потоков данных в Azure с помощью Azure Databricks
author: petertaylor9999
ms.date: 11/30/2018
ms.openlocfilehash: 0640e900c212d2b75cc9cdd5bec3a4f7c050490d
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902839"
---
# <a name="stream-processing-with-azure-databricks"></a>Обработка потоков данных с помощью Azure Databricks

На схеме эталонной архитектуры представлен сквозной конвейер [обработки потоков данных](/azure/architecture/data-guide/big-data/real-time-processing). Конвейер такого типа состоит из четырех этапов: прием, обработка, сохранение, анализ и создание отчета. В этой эталонной архитектуре конвейер принимает данные из двух источников, объединяет связанные записи из каждого потока, дополняет результат и вычисляет среднее значение в реальном времени. Результаты сохраняются для дальнейшего анализа. [**Разверните это решение**.](#deploy-the-solution)

![](./images/stream-processing-databricks.png)

**Сценарий**. Компания, предоставляющая услуги такси, собирает данные о каждой поездке в такси. В этом сценарии предполагается, что данные отправляются с двух отдельных устройств. В такси установлен счетчик, который отправляет данные о каждой поездке, включая сведения о продолжительности, расстоянии, а также местах посадки и высадки. Отдельное устройство принимает платежи от клиентов и отправляет данные о тарифах. Чтобы определить тенденции роста пассажиропотока, компании нужно для каждого округа вычислить среднюю сумму чаевых на милю в реальном времени.

## <a name="architecture"></a>Архитектура

Архитектура состоит из следующих компонентов:

**Источники данных.** В этой архитектуре существует два источника данных, которые создают потоки данных в реальном времени. Первый поток содержит сведения о поездке, а второй — о тарифах. В эталонной архитектуре есть имитированный генератор данных, который считывает данные из набора статических файлов и отправляет данные в Центры событий. В реальном приложении источниками данных будут устройства, установленные в такси.

**Центры событий Azure**. [Центры событий](/azure/event-hubs/) — это служба приема событий. В этой архитектуре используется два экземпляра службы — по одной на каждый источник данных. Каждый источник данных отправляет поток данных в соответствующую службу.

**Azure Databricks**. [Databricks](/azure/azure-databricks/) — это платформа аналитики на основе Apache Spark, оптимизированная для платформы облачных служб Microsoft Azure. Databricks используется для корреляции данных о поездке на такси и тарифах, а также для дополнения коррелированных данных сведениями об округе, хранящимися в файловой системе Databricks.

**Cosmos DB**. Выходные данные задания Azure Databricks будут представлять собой последовательность записей, которые вносятся в [Cosmos DB](/azure/cosmos-db/) с помощью API Cassandra. Мы используем API Cassandra, потому что этот интерфейс поддерживает моделирование данных временных рядов.

**Azure Log Analytics**. Данные журнала приложений, собранные [Azure Monitor](/azure/monitoring-and-diagnostics/), хранятся в [рабочей области Log Analytics](/azure/log-analytics). Вы можете использовать запросы Log Analytics для анализа и визуализации метрик, а также просмотра сообщений журнала, чтобы выявить проблемы в приложении.

## <a name="data-ingestion"></a>Прием данных

Для имитации источника данных в этой эталонной архитектуре используется набор данных<sup>[[1]](#note1)</sup> [New York City Taxi Data](https://uofi.app.box.com/v/NYCtaxidata/folder/2332218797) (Данные о поездках в такси в Нью-Йорке). Этот набор содержит данные о поездках в такси в Нью-Йорке за 4 года (2010&ndash;2013). В нем представлены два типа записей: данные о поездках и тарифах. Данные о поездках включают сведения о продолжительности поездки, расстоянии, а также местах посадки и высадки. Данные о тарифах включают сведения о тарифе, налоге и сумме чаевых. В обоих типах записей есть стандартные поля: номер медальона, лицензия на право вождения и код организации. Вместе эти три поля позволяют уникально идентифицировать такси и водителя. Данные хранятся в формате CSV. 

Генератор данных — это приложение .NET Core, которое считывает записи и отправляет их в Центры событий Azure. Генератор отправляет данные о поездке в формате JSON, а данные о тарифах — в формате CSV. 

Для сегментации данных Центры событий используют [секции](/azure/event-hubs/event-hubs-features#partitions). Они позволяют объекту-получателю считывать данные каждой секции параллельно. При отправке данных в Центры событий можно явно указать ключ секции. В противном случае записи назначаются секциям методом циклического перебора. 

В этом примере данные о поездках и тарифах должны в итоге иметь одинаковый идентификатор секции для определенного такси. Это позволит Databricks применить определенную степень параллелизма при корреляции двух потоков. Запись в секции *n* с данными о поездке будет соответствовать записи в секции *n* с данными о тарифах.

![](./images/stream-processing-databricks-eh.png)

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

Пример развертывания для этой архитектуры можно найти на портале [GitHub](https://github.com/mspnp/azure-databricks-streaming-analytics). 

### <a name="prerequisites"></a>Предварительные требования

1. Клонируйте или загрузите репозиторий GitHub [Stream processing with Azure Databricks](https://github.com/mspnp/azure-databricks-streaming-analytics) (Обработка потоков данных с помощью Azure Databricks) либо создайте его вилку.

2. Установите [Docker](https://www.docker.com/), чтобы запустить генератор данных.

3. Установите [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).

4. Установите [интерфейс командной строки Databricks](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html).

5. Из командной строки, строки bash или строки PowerShell войдите в свою учетную запись Azure, как показано ниже:
    ```shell
    az login
    ```
6. Установите интегрированную среду разработки Java со следующими ресурсами:
    - JDK 1.8;
    - пакет SDK для Scala 2.11;
    - Maven 3.5.4.

### <a name="download-the-new-york-city-taxi-and-neighborhood-data-files"></a>Скачивание файлов с данными о поездках на такси в Нью-Йорке и сведениями об округах

1. Создайте каталог с именем `DataFile` в корневом каталоге клонированного репозитория Github в своей локальной файловой системе.

2. Откройте браузер и перейдите по ссылке https://uofi.app.box.com/v/NYCtaxidata/folder/2332219935.

3. На этой странице нажмите кнопку **Скачать**, чтобы скачать ZIP-файл со всеми данные о поездках в такси за этот год.

4. Извлеките ZIP-файл в каталог `DataFile`.

    > [!NOTE]
    > Этот ZIP-файл содержит другие ZIP-файлы. Не извлекайте дочерние ZIP-файлы.

    Структура каталога должна выглядеть так:

    ```shell
    /DataFile
        /FOIL2013
            trip_data_1.zip
            trip_data_2.zip
            trip_data_3.zip
            ...
    ```

5. Откройте браузер и перейдите по ссылке https://www.zillow.com/howto/api/neighborhood-boundaries.htm. 

6. Щелкните **New York Neighborhood Boundaries** (Границы округов штата Нью-Йорк), чтобы скачать файл.

7. Скопируйте файл **ZillowNeighborhoods-NY.zip** из каталога **загрузок** браузера в каталог `DataFile`.

### <a name="deploy-the-azure-resources"></a>Развертывание ресурсов Azure

1. В оболочке или командной строке Windows выполните следующую команду и следуйте указаниям при запросе на вход:

    ```bash
    az login
    ```

2. Перейдите к папке с именем `azure` в репозитории GitHub.

    ```bash
    cd azure
    ```

3. Выполните следующие команды, чтобы развернуть ресурсы Azure:

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

4. По завершении развертывания выходные данные выводятся в консоль. Найдите в выходных данных следующий код JSON:

```JSON
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
Эти значения являются секретами, которые вы добавите к секретам Databricks в следующих разделах. Для этого сохраните их в надежном расположении.

### <a name="add-a-cassandra-table-to-the-cosmos-db-account"></a>Добавление таблицы Cassandra в учетную запись Cosmos DB

1. На портале Azure перейдите к группе ресурсов, созданной при работе с предыдущим разделом о **развертывании ресурсов Azure**. Щелкните **Учетная запись Azure Cosmos DB**. Создайте таблицу с помощью API Cassandra.

2. В колонке **Обзор** щелкните **Добавить таблицу**.

3. Когда откроется колонка **Добавление таблицы** введите `newyorktaxi` в текстовое поле **Имя пространства ключей**. 

4. В разделе **ввода команды CQL для создания таблицы** введите `neighborhoodstats` в текстовое поле рядом с `newyorktaxi`.

5. В текстовое поле ниже введите следующее:
```shell
(neighborhood text, window_end timestamp, number_of_rides bigint,total_fare_amount double, primary key(neighborhood, window_end))
```
6. В текстовое поле **Пропускная способность (1000–1 000 000 единиц запросов в секунду)** введите значение `4000`.

7. Последовательно выберите **ОК**.

### <a name="add-the-databricks-secrets-using-the-databricks-cli"></a>Добавление секретов Databricks с помощью интерфейса командной строки Databricks

Сначала введите секреты для концентратора событий.

1. С помощью **интерфейса командной строки Azure Databricks**, установленном на шаге 2 в разделе с предварительными требованиями, создайте область секретов Azure Databricks:
    ```shell
    databricks secrets create-scope --scope "azure-databricks-job"
    ```
2. Добавьте секрет для концентратора событий с данными о поездках на такси:
    ```shell
    databricks secrets put --scope "azure-databricks-job" --key "taxi-ride"
    ```
    После завершения эта команда открывает редактор vi. Введите значение **taxi-ride-eh** из раздела выходных данных **eventHubs**, приведенном на шаге 4 в разделе о *развертывании ресурсов Azure*. Сохраните файл и закройте редактор vi.

3. Добавьте секрет для концентратора событий с данными о тарифах на такси:
    ```shell
    databricks secrets put --scope "azure-databricks-job" --key "taxi-fare"
    ```
    После завершения эта команда открывает редактор vi. Введите значение **taxi-fare-eh** из раздела выходных данных **eventHubs**, приведенном на шаге 4 в разделе о *развертывании ресурсов Azure*. Сохраните файл и закройте редактор vi.

Теперь введите секреты для Cosmos DB.

1. Откройте портал Azure и перейдите к группе ресурсов, указанной на шаге 3 в разделе о **развертывании ресурсов Azure**. Щелкните "Учетная запись Azure Cosmos DB".

2. С помощью **интерфейса командной строки Azure Databricks** добавьте секрет для имени пользователя Cosmos DB:
    ```shell
    databricks secrets put --scope azure-databricks-job --key "cassandra-username"
    ```
После завершения эта команда открывает редактор vi. Введите значение **username** из раздела выходных данных **CosmosDB**, приведенном на шаге 4 в разделе о *развертывании ресурсов Azure*. Сохраните файл и закройте редактор vi.

3. Затем добавьте секрет для пароля Cosmos DB:
    ```shell
    databricks secrets put --scope azure-databricks-job --key "cassandra-password"
    ```

После завершения эта команда открывает редактор vi. Введите значение **secret** из раздела выходных данных **CosmosDb**, приведенном на шаге 4 в разделе о *развертывании ресурсов Azure*. Сохраните файл и закройте редактор vi.

> [!NOTE]
> Используемая [область секретов, поддерживаемая Azure Key Vault](https://docs.azuredatabricks.net/user-guide/secrets/secret-scopes.html#azure-key-vault-backed-scopes), должна иметь имя **azure-databricks-job**, а имена секретов должны быть точно такими же, как приведенные выше.

### <a name="add-the-zillow-neighborhoods-data-file-to-the-databricks-file-system"></a>Добавление файла Zillow с данными об округах в файловую систему Databricks

1. Создайте каталог в файловой системе Databricks:
    ```bash
    dbfs mkdirs dbfs:/azure-databricks-jobs
    ```

2. Перейдите к каталогу `DataFile` и введите следующее:
    ```bash
    dbfs cp ZillowNeighborhoods-NY.zip dbfs:/azure-databricks-jobs
    ```

### <a name="add-the-azure-log-analytics-workspace-id-and-primary-key-to-configuration-files"></a>Добавление идентификатора и первичного ключа рабочей области Azure Log Analytics в файлы конфигурации

Для выполнения инструкций из этого раздела вам потребуется идентификатор и первичный ключ рабочей области Log Analytics. Идентификатор рабочей области указан в качестве значения для **workspaceId** в разделе выходных данных **logAnalytics**, приведенном на шаге 4 в разделе о *развертывании ресурсов Azure*. Первичный ключ — это значение для **secret** в разделе выходных данных. 

1. Чтобы настроить ведение журнала log4j, откройте `\azure\AzureDataBricksJob\src\main\resources\com\microsoft\pnp\azuredatabricksjob\log4j.properties`. Измените следующие два значения:
    ```shell
    log4j.appender.A1.workspaceId=<Log Analytics workspace ID>
    log4j.appender.A1.secret=<Log Analytics primary key>
    ```

2. Чтобы настроить пользовательскую систему ведения журналов, откройте `\azure\azure-databricks-monitoring\scripts\metrics.properties`. Измените следующие два значения:
    ```shell
    *.sink.loganalytics.workspaceId=<Log Analytics workspace ID>
    *.sink.loganalytics.secret=<Log Analytics primary key>
    ```

### <a name="build-the-jar-files-for-the-databricks-job-and-databricks-monitoring"></a>Создание JAR-файлов для задания и мониторинга Databricks

1. Используйте интегрированную среду разработки Java для импорта файла проекта Maven с именем **pom.xml**, расположенного в корневом каталоге. 

2. Выполните чистую сборку. Результатом этой сборки будет файл **azure-databricks-job-1.0-SNAPSHOT.jar** и **azure-databricks-monitoring-0.9.jar**. 

### <a name="configure-custom-logging-for-the-databricks-job"></a>Настройка пользовательской системы ведения журналов для задания Databricks

1. Скопируйте файл **azure-databricks-monitoring-0.9.jar** в файловую систему Databricks. Для этого введите следующую команду в **интерфейсе командной строки Databricks**:
    ```shell
    databricks fs cp --overwrite azure-databricks-monitoring-0.9.jar dbfs:/azure-databricks-job/azure-databricks-monitoring-0.9.jar
    ```

2. Скопируйте свойства пользовательской системы ведения журналов из `\azure\azure-databricks-monitoring\scripts\metrics.properties` в файловую систему Databricks. Для этого введите следующую команду:
    ```shell
    databricks fs cp --overwrite metrics.properties dbfs:/azure-databricks-job/metrics.properties
    ```

3. Если вы до этого не определились с именем для кластера Databricks, выберите его сейчас. Вам потребуется указать его в приведенном ниже пути к кластеру в файловой системе Databricks. Скопируйте скрипт инициализации из `\azure\azure-databricks-monitoring\scripts\spark.metrics` в файловую систему Databricks. Для этого введите следующую команду:
    ```
    databricks fs cp --overwrite spark-metrics.sh dbfs:/databricks/init/<cluster-name>/spark-metrics.sh
    ```

### <a name="create-a-databricks-cluster"></a>Создание кластера Databricks

1. В рабочей области Databricks щелкните Clusters (Кластеры) и выберите Create cluster (Создать кластер). Введите имя кластера, созданного на шаге 3 в предыдущем разделе о **настройке пользовательской системы ведения журналов для задания Databricks**. 

2. Выберите **стандартный** режим кластера.

3. Для **версии среды выполнения Databricks** укажите **4.3 (содержит Apache Spark 2.3.1, Scala 2.11)**.

4. Для **версии Python** укажите **2**.

5. Укажите для **типа драйвера** значение **Same as worker** (Как для рабочего узла).

6. Для **Worker Type** (Тип рабочего узла) укажите значение **Standard_DS3_v2**.

7. Для параметра **Min Workers** (Минимальное количество рабочих узлов) задайте значение **2**.

8. Снимите флажок **Enable autoscaling** (Включить автомасштабирование). 

9. В диалоговом окне **Auto Termination** (Автоматическое завершение работы) щелкните **Init Scripts** (Скрипты инициализации). 

10. Введите **dbfs:/databricks/init/<cluster-name>/spark-metrics.sh** и вместо <cluster-name> укажите имя кластера, созданного на шаге 1.

11. Нажмите кнопку **Add** (Добавить).

12. Нажмите кнопку **Create Cluster** (Создать кластер).

### <a name="create-a-databricks-job"></a>Создание задания Databricks

1. В рабочей области Databricks выберите Jobs (Задания) и Create job (Создать задание).

2. Введите имя задания.

3. Если щелкнуть set jar (Указать JAR-файл), откроется диалоговое окно Upload JAR to Run (Отправить JAR-файл для запуска).

4. Перетащите файл **azure-databricks-job-1.0-SNAPSHOT.jar**, который вы создали при работе с разделом о **создании JAR-файлов для задания и мониторинга Databricks**, в поле **Drop JAR here to upload** (Перетащите сюда JAR-файл для отправки).

5. Укажите **com.microsoft.pnp.TaxiCabReader** в поле **Main Class** (Класс Main).

6. В поле аргументов введите следующий код:
    ```shell
    -n jar:file:/dbfs/azure-databricks-jobs/ZillowNeighborhoods-NY.zip!/ZillowNeighborhoods-NY.shp --taxi-ride-consumer-group taxi-ride-eh-cg --taxi-fare-consumer-group taxi-fare-eh-cg --window-interval "1 minute" --cassandra-host <Cosmos DB Cassandra host name from above> 
    ``` 

7. Установите зависимые библиотеки, выполнив следующие действия:
    
    1. В пользовательском интерфейсе Databricks нажмите кнопку **Home** (Главная).
    
    2. В раскрывающемся списке **Users** (Пользователи) щелкните имя учетной записи пользователя, чтобы открыть настройки ее рабочей области.
    
    3. Щелкните стрелку раскрывающегося списка рядом с именем вашей учетной записи, выберите **Create** (Создать) и **Library** (Библиотека), чтобы открыть диалоговое окно **New Library** (Новая библиотека).
    
    4. В раскрывающемся элементе управления **Source** (Источник) выберите **Maven Coordinate** (Координаты Maven).
    
    5. В разделе **Install Maven Artifacts** (Установка артефактов Maven) введите `com.microsoft.azure:azure-eventhubs-spark_2.11:2.3.5` в текстовое поле **Coordinate** (Координаты). 
    
    6. Щелкните **Create Library** (Создать библиотеку), чтобы открыть окно **артефактов**.
    
    7. В разделе **Status on running clusters** (Состояние работающих кластеров) установите флажок **Attach automatically to all clusters** (Автоматически подключать ко всем кластерам).
    
    8. Повторите шаги 1–7 для координат Maven `com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.0.0`.
    
    9. Повторите шаги 1–6 для координат Maven `org.geotools:gt-shapefile:19.2`.
    
    10. Щелкните **Advanced Options** (Дополнительные параметры).
    
    11. Введите `http://download.osgeo.org/webdav/geotools/` в текстовое поле для **репозитория**. 
    
    12. Щелкните **Create Library** (Создать библиотеку), чтобы открыть окно **артефактов**. 
    
    13. В разделе **Status on running clusters** (Состояние работающих кластеров) установите флажок **Attach automatically to all clusters** (Автоматически подключать ко всем кластерам).

8. Добавьте зависимые библиотеки, добавленные на шаге 7, в задание, которое вы создали в конце шага 6:
    1. В рабочей области Azure Databricks выберите **Jobs** (Задания).

    2. Щелкните имя задания, созданного на шаге 2 в разделе о **создании задания Databricks**. 
    
    3. В разделе **зависимых библиотек** щелкните **Add** (Добавить), чтобы открыть диалоговое окно **Add Dependent Library** (Добавление зависимой библиотеки). 
    
    4. В разделе **Library From** (Источник библиотеки) выберите **Workspace** (Рабочая область).
    
    5. Щелкните **Users** (Пользователи), выберите свое имя пользователя и щелкните `azure-eventhubs-spark_2.11:2.3.5`. 
    
    6. Последовательно выберите **ОК**.
    
    7. Повторите шаги 1–6 для `spark-cassandra-connector_2.11:2.3.1` и `gt-shapefile:19.2`.

9. Рядом с полем **Cluster:** (Кластер) щелкните **Edit** (Изменить). Откроется диалоговое окно **настройки кластера**. В раскрывающемся списке **Cluster Type** (Тип кластера) выберите **Existing Cluster** (Существующий кластер). В раскрывающемся списке **Select Cluster** (Выбрать кластер) выберите кластер, созданный при работе с разделом **Создание кластера Databricks**. Щелкните **Confirm** (Подтвердить).

10. Щелкните **Run now** (Запустить сейчас).

### <a name="run-the-data-generator"></a>Запуск генератора данных

1. Перейдите к каталогу с именем `onprem` в репозитории GitHub.

2. Обновите значения в файле **main.env** следующим образом:

    ```shell
    RIDE_EVENT_HUB=[Connection string for the taxi-ride event hub]
    FARE_EVENT_HUB=[Connection string for the taxi-fare event hub]
    RIDE_DATA_FILE_PATH=/DataFile/FOIL2013
    MINUTES_TO_LEAD=0
    PUSH_RIDE_DATA_FIRST=false
    ```
    Строка подключения к концентратору событий с данными о поездках на такси указана в качестве значение **taxi-ride-eh** из раздела выходных данных **eventHubs**, приведенном на шаге 4 в разделе о *развертывании ресурсов Azure*. Строка подключения к концентратору событий с данными о тарифах на такси указана в качестве значение **taxi-fare-eh** из раздела выходных данных **eventHubs**, приведенном на шаге 4 в разделе о *развертывании ресурсов Azure*.

3. Чтобы создать образ Docker, выполните следующую команду:

    ```bash
    docker build --no-cache -t dataloader .
    ```

4. Вернитесь к родительскому каталогу.

    ```bash
    cd ..
    ```

5. Чтобы запустить образ Docker, выполните следующую команду:

    ```bash
    docker run -v `pwd`/DataFile:/DataFile --env-file=onprem/main.env dataloader:latest
    ```

Полученный результат должен выглядеть примерно так:

```
Created 10000 records for TaxiFare
Created 10000 records for TaxiRide
Created 20000 records for TaxiFare
Created 20000 records for TaxiRide
Created 30000 records for TaxiFare
...
```

Чтобы проверить, правильно ли выполняется задание Databricks, откройте портал Azure и перейдите к базе данных Cosmos DB. Откройте колонку **Обозреватель данных** и просмотрите данные в таблице **taxi records**. 

[1] <span id="note1">Брайан Донован (Brian Donovan); Дэн Уорк (Dan Work), 2016 г.: New York City Taxi Trip Data (2010–2013) (Данные о поездках в такси в Нью-Йорке). Иллинойсский университет в Урбане-Шампейне. https://doi.org/10.13012/J8PN93H8
