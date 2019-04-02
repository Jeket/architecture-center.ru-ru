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
# <a name="send-azure-databricks-application-logs-to-azure-monitor"></a>Отправка журналов приложения Azure Databricks в Azure Monitor

В этой статье показано, как отправлять журналы приложения и метрики из Azure Databricks для [рабочей области Log Analytics](/azure/azure-monitor/platform/manage-access). Она использует [библиотека мониторинга Azure Databricks](https://github.com/mspnp/spark-monitoring), которая доступна на сайте GitHub.

## <a name="prerequisites"></a>Технические условия

Настройка кластера Azure Databricks для использования мониторинга библиотеки, как описано в разделе [настроить Azure Databricks для отправки метрик в Azure Monitor][config-cluster].

> [!NOTE]
> Библиотека отслеживания выполняет потоковую передачу событий на уровне Apache Spark и структурированной потоковой передачи Spark метрики из заданий в Azure Monitor. Не нужно вносить изменения в код приложения для этих событий и метрик.

## <a name="send-application-metrics-using-dropwizard"></a>Отправка метрики приложения с помощью Dropwizard

Spark используется настраиваемая система метрик на основе библиотеки Dropwizard метрики. Дополнительные сведения см. в разделе [метрики](https://spark.apache.org/docs/latest/monitoring.html#metrics) в документации по Spark.

Для отправки метрик приложения из кода приложения в Azure Databricks в Azure Monitor, выполните следующие действия.

1. Построение **spark прослушивателей loganalytics-1.0-SNAPSHOT.jar** JAR-файл, как описано в разделе [сборку библиотеки мониторинга Azure Databricks][build-lib].

1. Создание Dropwizard [датчики или счетчики](https://metrics.dropwizard.io/4.0.0/manual/core.html) в коде приложения. Можно использовать `UserMetricsSystem` класс, определенный в библиотеке мониторинга. В следующем примере создается счетчик с именем `counter1`.

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

    Библиотека отслеживания включает [пример приложения] [ sample-app] показано, как использовать `UserMetricsSystem` класса.

## <a name="send-application-logs-using-log4j"></a>Отправка журналов приложений с помощью Log4j

Чтобы отправить журналы приложения Azure Databricks Azure Log Analytics с помощью [аппендера Log4j](https://logging.apache.org/log4j/2.x/manual/appenders.html) в библиотеке, выполните следующие действия:

1. Построение **spark прослушивателей loganalytics-1.0-SNAPSHOT.jar** JAR-файл, как описано в разделе [сборку библиотеки мониторинга Azure Databricks][build-lib].

1. Создание **log4j.properties** [файл конфигурации](https://logging.apache.org/log4j/2.x/manual/configuration.html) для вашего приложения. Включать следующие свойства конфигурации. Замените имя пакета приложения и журналов на уровне, как показано:

    ```YAML
    log4j.appender.A1=com.microsoft.pnp.logging.loganalytics.LogAnalyticsAppender
    log4j.appender.A1.layout=com.microsoft.pnp.logging.JSONLayout
    log4j.appender.A1.layout.LocationInfo=false
    log4j.additivity.<your application package name>=false
    log4j.logger.<your application package name>=<log level>, A1
    ```

    Пример файла конфигурации можно найти [здесь][log4j.properties].

1. В коде приложения включают **spark прослушивателей loganalytics** проекта, а также импортировать `com.microsoft.pnp.logging.Log4jconfiguration` в код приложения.

    ```Scala
    import com.microsoft.pnp.logging.Log4jConfiguration
    ```

1. Настройка с помощью Log4j **log4j.properties** файл, созданный на шаге 3:

    ```Scala
    getClass.getResourceAsStream("<path to file in your JAR file>/log4j.properties")) {
          stream => {
            Log4jConfiguration.configure(stream)
          }
    }
    ```

1. Добавление сообщений журнала Apache Spark на соответствующем уровне в коде при необходимости. Например, использовать `logDebug` метод для отправки meesage журнала отладки. Дополнительные сведения см. в разделе [ведение журнала] [ spark-logging] в документации по Spark.

    ```Scala
    logTrace("Trace message")
    logDebug("Debug message")
    logInfo("Info message")
    logWarning("Warning message")
    logError("Error message")
    ```

## <a name="run-the-sample-application"></a>Запуск примера приложения

Библиотека отслеживания включает [пример приложения] [ sample-app] показано, как отправлять метрики приложения и журналы приложений в Azure Monitor. Запуск примера

1. Построение **заданий spark** проект в библиотеке мониторинга, как описано в разделе [сборку библиотеки мониторинга Azure Databricks][build-lib].

1. Перейдите к рабочей области Databricks и создайте новое задание, как описано [здесь](https://docs.azuredatabricks.net/user-guide/jobs.html#create-a-job).

1. На странице сведений задания выберите **задать JAR**.

1. Передача JAR-файл из `/src/spark-jobs/target/spark-jobs-1.0-SNAPSHOT.jar`.

1. Для **класс Main**, введите `com.microsoft.pnp.samplejob.StreamingQueryListenerSampleJob`.

1. Выберите кластер, который уже настроен для использования мониторинга библиотеки. См. в разделе [настроить Azure Databricks для отправки метрик в Azure Monitor][config-cluster].

При запуске задания, можно просмотреть журналы приложения и метрики в рабочей области Log Analytics.

Журналы приложений отображаются в разделе SparkLoggingEvent_CL:

```Kusto
SparkLoggingEvent_CL | where logger_name_s contains "com.microsoft.pnp"
```

Метрики приложения отображаются в разделе SparkMetric_CL:

```Kusto
SparkMetric_CL | where name_s contains "rowcounter" | limit 50
```

## <a name="next-steps"></a>Дальнейшие действия

Разверните панель мониторинга, прилагаемый к этой библиотеке кода для устранения проблем с производительностью в ваших рабочих нагрузок Azure Databricks производительность.

> [!div class="nextstepaction"]
> [Использование панелей мониторинга для визуализации метрик Azure Databricks](./dashboards.md)

<!-- links -->

[build-lib]: ./configure-cluster.md##build-the-azure-databricks-monitoring-library
[config-cluster]: ./configure-cluster.md
[log4j.properties]: https://github.com/mspnp/spark-monitoring/blob/master/src/spark-jobs/src/main/resources/com/microsoft/pnp/samplejob/log4j.properties
[sample-app]: https://github.com/mspnp/spark-monitoring/tree/master/src/spark-jobs
[spark-logging]: https://spark.apache.org/docs/2.3.0/api/java/org/apache/spark/internal/Logging.html