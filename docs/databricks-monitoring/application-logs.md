---
title: Отправка журналов приложения Azure Databricks в Azure Monitor
description: Как отправлять пользовательские журналы и метрики из Azure Databricks в Azure Monitor
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: 49c631687fb3e3bbd807ffbbb49d9c5f6526bfb4
ms.sourcegitcommit: 9854bd27fb5cf92041bbfb743d43045cd3552a69
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/27/2019
ms.locfileid: "58503434"
---
# <a name="send-azure-databricks-application-logs-to-azure-monitor"></a><span data-ttu-id="cb7a6-103">Отправка журналов приложения Azure Databricks в Azure Monitor</span><span class="sxs-lookup"><span data-stu-id="cb7a6-103">Send Azure Databricks application logs to Azure Monitor</span></span>

<span data-ttu-id="cb7a6-104">В этой статье показано, как отправлять журналы приложения и метрики из Azure Databricks для [рабочей области Log Analytics](/azure/azure-monitor/platform/manage-access).</span><span class="sxs-lookup"><span data-stu-id="cb7a6-104">This article shows how to send application logs and metrics from Azure Databricks to a [Log Analytics workspace](/azure/azure-monitor/platform/manage-access).</span></span> <span data-ttu-id="cb7a6-105">Она использует [библиотека мониторинга Azure Databricks](https://github.com/mspnp/spark-monitoring), которая доступна на сайте GitHub.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-105">It uses the [Azure Databricks Monitoring Library](https://github.com/mspnp/spark-monitoring), which is available on GitHub.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="cb7a6-106">Технические условия</span><span class="sxs-lookup"><span data-stu-id="cb7a6-106">Prerequisites</span></span>

<span data-ttu-id="cb7a6-107">Настройка кластера Azure Databricks для использования мониторинга библиотеки, как описано в разделе [настроить Azure Databricks для отправки метрик в Azure Monitor][config-cluster].</span><span class="sxs-lookup"><span data-stu-id="cb7a6-107">Configure your Azure Databricks cluster to use the monitoring library, as described in [Configure Azure Databricks to send metrics to Azure Monitor][config-cluster].</span></span>

> [!NOTE]
> <span data-ttu-id="cb7a6-108">Библиотека отслеживания выполняет потоковую передачу событий на уровне Apache Spark и структурированной потоковой передачи Spark метрики из заданий в Azure Monitor.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-108">The monitoring library streams Apache Spark level events and Spark Structured Streaming metrics from your jobs to Azure Monitor.</span></span> <span data-ttu-id="cb7a6-109">Не нужно вносить изменения в код приложения для этих событий и метрик.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-109">You don't need to make any changes to your application code for these events and metrics.</span></span>

## <a name="send-application-metrics-using-dropwizard"></a><span data-ttu-id="cb7a6-110">Отправка метрики приложения с помощью Dropwizard</span><span class="sxs-lookup"><span data-stu-id="cb7a6-110">Send application metrics using Dropwizard</span></span>

<span data-ttu-id="cb7a6-111">Spark используется настраиваемая система метрик на основе библиотеки Dropwizard метрики.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-111">Spark uses a configurable metrics system based on the Dropwizard Metrics Library.</span></span> <span data-ttu-id="cb7a6-112">Дополнительные сведения см. в разделе [метрики](https://spark.apache.org/docs/latest/monitoring.html#metrics) в документации по Spark.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-112">For more information, see [Metrics](https://spark.apache.org/docs/latest/monitoring.html#metrics) in the Spark documentation.</span></span>

<span data-ttu-id="cb7a6-113">Для отправки метрик приложения из кода приложения в Azure Databricks в Azure Monitor, выполните следующие действия.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-113">To send application metrics from Azure Databricks application code to Azure Monitor, follow these steps:</span></span>

1. <span data-ttu-id="cb7a6-114">Построение **spark прослушивателей loganalytics-1.0-SNAPSHOT.jar** JAR-файл, как описано в разделе [сборку библиотеки мониторинга Azure Databricks][build-lib].</span><span class="sxs-lookup"><span data-stu-id="cb7a6-114">Build the **spark-listeners-loganalytics-1.0-SNAPSHOT.jar** JAR file as described in [Build the Azure Databricks Monitoring Library][build-lib].</span></span>

1. <span data-ttu-id="cb7a6-115">Создание Dropwizard [датчики или счетчики](https://metrics.dropwizard.io/4.0.0/manual/core.html) в коде приложения.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-115">Create Dropwizard [gauges or counters](https://metrics.dropwizard.io/4.0.0/manual/core.html) in your application code.</span></span> <span data-ttu-id="cb7a6-116">Можно использовать `UserMetricsSystem` класс, определенный в библиотеке мониторинга.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-116">You can use the `UserMetricsSystem` class defined in the monitoring library.</span></span> <span data-ttu-id="cb7a6-117">В следующем примере создается счетчик с именем `counter1`.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-117">The following example creates a counter named `counter1`.</span></span>

    ```Scala
    import org.apache.spark.metrics.UserMetricsSystems
    import org.apache.spark.sql.SparkSession

    object StreamingQueryListenerSampleJob  {

      private final val METRICS_NAMESPACE = "samplejob"
      private final val COUNTER_NAME = "counter1"

      def main(args: Array[String]): Unit = {

        val spark = SparkSession
          .builder
          .getOrCreate

        val driverMetricsSystem = UserMetricsSystems
            .getMetricSystem(METRICS_NAMESPACE, builder => {
              builder.registerCounter(COUNTER_NAME)
            })

        driverMetricsSystem.counter(COUNTER_NAME).inc(5)
      }
    }
    ```

    <span data-ttu-id="cb7a6-118">Библиотека отслеживания включает [пример приложения] [ sample-app] показано, как использовать `UserMetricsSystem` класса.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-118">The monitoring library includes a [sample application][sample-app] that demonstrates how to use the `UserMetricsSystem` class.</span></span>

## <a name="send-application-logs-using-log4j"></a><span data-ttu-id="cb7a6-119">Отправка журналов приложений с помощью Log4j</span><span class="sxs-lookup"><span data-stu-id="cb7a6-119">Send application logs using Log4j</span></span>

<span data-ttu-id="cb7a6-120">Чтобы отправить журналы приложения Azure Databricks Azure Log Analytics с помощью [аппендера Log4j](https://logging.apache.org/log4j/2.x/manual/appenders.html) в библиотеке, выполните следующие действия:</span><span class="sxs-lookup"><span data-stu-id="cb7a6-120">To send your Azure Databricks application logs to Azure Log Analytics using the [Log4j appender](https://logging.apache.org/log4j/2.x/manual/appenders.html) in the library, follow these steps:</span></span>

1. <span data-ttu-id="cb7a6-121">Построение **spark прослушивателей loganalytics-1.0-SNAPSHOT.jar** JAR-файл, как описано в разделе [сборку библиотеки мониторинга Azure Databricks][build-lib].</span><span class="sxs-lookup"><span data-stu-id="cb7a6-121">Build the **spark-listeners-loganalytics-1.0-SNAPSHOT.jar** JAR file as described in [Build the Azure Databricks Monitoring Library][build-lib].</span></span>

1. <span data-ttu-id="cb7a6-122">Создание **log4j.properties** [файл конфигурации](https://logging.apache.org/log4j/2.x/manual/configuration.html) для вашего приложения.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-122">Create a **log4j.properties** [configuration file](https://logging.apache.org/log4j/2.x/manual/configuration.html) for your application.</span></span> <span data-ttu-id="cb7a6-123">Включать следующие свойства конфигурации.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-123">Include the following configuration properties.</span></span> <span data-ttu-id="cb7a6-124">Замените имя пакета приложения и журналов на уровне, как показано:</span><span class="sxs-lookup"><span data-stu-id="cb7a6-124">Substitute your application package name and log level where indicated:</span></span>

    ```YAML
    log4j.appender.A1=com.microsoft.pnp.logging.loganalytics.LogAnalyticsAppender
    log4j.appender.A1.layout=com.microsoft.pnp.logging.JSONLayout
    log4j.appender.A1.layout.LocationInfo=false
    log4j.additivity.<your application package name>=false
    log4j.logger.<your application package name>=<log level>, A1
    ```

    <span data-ttu-id="cb7a6-125">Пример файла конфигурации можно найти [здесь][log4j.properties].</span><span class="sxs-lookup"><span data-stu-id="cb7a6-125">You can find a sample configuration file [here][log4j.properties].</span></span>

1. <span data-ttu-id="cb7a6-126">В коде приложения включают **spark прослушивателей loganalytics** проекта, а также импортировать `com.microsoft.pnp.logging.Log4jconfiguration` в код приложения.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-126">In your application code, include the **spark-listeners-loganalytics** project, and import `com.microsoft.pnp.logging.Log4jconfiguration` to your application code.</span></span>

    ```Scala
    import com.microsoft.pnp.logging.Log4jConfiguration
    ```

1. <span data-ttu-id="cb7a6-127">Настройка с помощью Log4j **log4j.properties** файл, созданный на шаге 3:</span><span class="sxs-lookup"><span data-stu-id="cb7a6-127">Configure Log4j using the **log4j.properties** file you created in step 3:</span></span>

    ```Scala
    getClass.getResourceAsStream("<path to file in your JAR file>/log4j.properties")) {
          stream => {
            Log4jConfiguration.configure(stream)
          }
    }
    ```

1. <span data-ttu-id="cb7a6-128">Добавление сообщений журнала Apache Spark на соответствующем уровне в коде при необходимости.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-128">Add Apache Spark log messages at the appropriate level in your code as required.</span></span> <span data-ttu-id="cb7a6-129">Например, использовать `logDebug` метод для отправки meesage журнала отладки.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-129">For example, use the `logDebug` method to send a debug log meesage.</span></span> <span data-ttu-id="cb7a6-130">Дополнительные сведения см. в разделе [ведение журнала] [ spark-logging] в документации по Spark.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-130">For more information, see [Logging][spark-logging] in the Spark documentation.</span></span>

    ```Scala
    logTrace("Trace message")
    logDebug("Debug message")
    logInfo("Info message")
    logWarning("Warning message")
    logError("Error message")
    ```

## <a name="run-the-sample-application"></a><span data-ttu-id="cb7a6-131">Запуск примера приложения</span><span class="sxs-lookup"><span data-stu-id="cb7a6-131">Run the sample application</span></span>

<span data-ttu-id="cb7a6-132">Библиотека отслеживания включает [пример приложения] [ sample-app] показано, как отправлять метрики приложения и журналы приложений в Azure Monitor.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-132">The monitoring library includes a [sample application][sample-app] that demonstrates how to send both application metrics and application logs to Azure Monitor.</span></span> <span data-ttu-id="cb7a6-133">Запуск примера</span><span class="sxs-lookup"><span data-stu-id="cb7a6-133">To run the sample:</span></span>

1. <span data-ttu-id="cb7a6-134">Построение **заданий spark** проект в библиотеке мониторинга, как описано в разделе [сборку библиотеки мониторинга Azure Databricks][build-lib].</span><span class="sxs-lookup"><span data-stu-id="cb7a6-134">Build the **spark-jobs** project in the monitoring library, as described in [Build the Azure Databricks Monitoring Library][build-lib].</span></span>

1. <span data-ttu-id="cb7a6-135">Перейдите к рабочей области Databricks и создайте новое задание, как описано [здесь](https://docs.azuredatabricks.net/user-guide/jobs.html#create-a-job).</span><span class="sxs-lookup"><span data-stu-id="cb7a6-135">Navigate to your Databricks workspace and create a new job, as described [here](https://docs.azuredatabricks.net/user-guide/jobs.html#create-a-job).</span></span>

1. <span data-ttu-id="cb7a6-136">На странице сведений задания выберите **задать JAR**.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-136">In the job detail page, select **Set JAR**.</span></span>

1. <span data-ttu-id="cb7a6-137">Передача JAR-файл из `/src/spark-jobs/target/spark-jobs-1.0-SNAPSHOT.jar`.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-137">Upload the JAR file from `/src/spark-jobs/target/spark-jobs-1.0-SNAPSHOT.jar`.</span></span>

1. <span data-ttu-id="cb7a6-138">Для **класс Main**, введите `com.microsoft.pnp.samplejob.StreamingQueryListenerSampleJob`.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-138">For **Main class**, enter `com.microsoft.pnp.samplejob.StreamingQueryListenerSampleJob`.</span></span>

1. <span data-ttu-id="cb7a6-139">Выберите кластер, который уже настроен для использования мониторинга библиотеки.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-139">Select a cluster that is already configured to use the monitoring library.</span></span> <span data-ttu-id="cb7a6-140">См. в разделе [настроить Azure Databricks для отправки метрик в Azure Monitor][config-cluster].</span><span class="sxs-lookup"><span data-stu-id="cb7a6-140">See [Configure Azure Databricks to send metrics to Azure Monitor][config-cluster].</span></span>

<span data-ttu-id="cb7a6-141">При запуске задания, можно просмотреть журналы приложения и метрики в рабочей области Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-141">When the job runs, you can view the application logs and metrics in your Log Analytics workspace.</span></span>

<span data-ttu-id="cb7a6-142">Журналы приложений отображаются в разделе SparkLoggingEvent_CL:</span><span class="sxs-lookup"><span data-stu-id="cb7a6-142">Application logs appear under SparkLoggingEvent_CL:</span></span>

```Kusto
SparkLoggingEvent_CL | where logger_name_s contains "com.microsoft.pnp"
```

<span data-ttu-id="cb7a6-143">Метрики приложения отображаются в разделе SparkMetric_CL:</span><span class="sxs-lookup"><span data-stu-id="cb7a6-143">Application metrics appear under SparkMetric_CL:</span></span>

```Kusto
SparkMetric_CL | where name_s contains "rowcounter" | limit 50
```

## <a name="next-steps"></a><span data-ttu-id="cb7a6-144">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="cb7a6-144">Next steps</span></span>

<span data-ttu-id="cb7a6-145">Разверните панель мониторинга, прилагаемый к этой библиотеке кода для устранения проблем с производительностью в ваших рабочих нагрузок Azure Databricks производительность.</span><span class="sxs-lookup"><span data-stu-id="cb7a6-145">Deploy the performance monitoring dashboard that accompanies this code library to troubleshoot performance issues in your production Azure Databricks workloads.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="cb7a6-146">Использование панелей мониторинга для визуализации метрик Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="cb7a6-146">Use dashboards to visualize Azure Databricks metrics</span></span>](./dashboards.md)

<!-- links -->

[build-lib]: ./configure-cluster.md##build-the-azure-databricks-monitoring-library
[config-cluster]: ./configure-cluster.md
[log4j.properties]: https://github.com/mspnp/spark-monitoring/blob/master/src/spark-jobs/src/main/resources/com/microsoft/pnp/samplejob/log4j.properties
[sample-app]: https://github.com/mspnp/spark-monitoring/tree/master/src/spark-jobs
[spark-logging]: https://spark.apache.org/docs/2.3.0/api/java/org/apache/spark/internal/Logging.html