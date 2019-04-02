---
title: Настройка Azure Databricks для отправки метрик в Azure Monitor
description: Это библиотека scala, чтобы включить мониторинг метрик и ведения журнала данных в Azure Log Analytics
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: af6b6433f87964ac60c179ecf498e54129344126
ms.sourcegitcommit: 9854bd27fb5cf92041bbfb743d43045cd3552a69
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/27/2019
ms.locfileid: "58503439"
---
<!-- markdownlint-disable MD040 -->

# <a name="configure-azure-databricks-to-send-metrics-to-azure-monitor"></a>Настройка Azure Databricks для отправки метрик в Azure Monitor

В этой статье показано, как настроить кластер Azure Databricks для отправки метрик для [рабочей области Log Analytics](/azure/azure-monitor/platform/manage-access). Она использует [библиотека мониторинга Azure Databricks](https://github.com/mspnp/spark-monitoring), которая доступна на сайте GitHub. Основные сведения о Java, Scala и Maven рекомендуется в качестве prerequisistes.

## <a name="about-the-azure-databricks-monitoring-library"></a>Об Azure Databricks, библиотека мониторинга

[Репозиторий GitHub](https://github.com/mspnp/spark-monitoring) для библиотеки мониторинга Azure Databricks имеет следующие структуру каталогов:

```
/src  
    /spark-jobs  
    /spark-listeners-loganalytics  
    /spark-listeners  
    /pom.xml  
```

**Заданий spark** каталог — пример приложения Spark, демонстрирующий, как реализовать счетчик метрик приложения Spark.

**Spark прослушиватели** каталог включает функциональность, позволяющую Databricks Azure для отправки событий Apache Spark на уровне службы в рабочую область Azure Log Analytics. Azure Databricks — это служба, на основе Apache Spark, который включает набор структурированных API для пакетной обработки данных с помощью наборов данных, таблицы данных и SQL. С помощью Apache Spark 2.0 была добавлена поддержка [структурированной потоковой передачи](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html), поток данных обработки API, созданный на обработку интерфейсы API пакета Spark.

**Spark прослушивателей loganalytics** каталог включает приемника для Spark прослушиватели приемника для DropWizard и клиента для рабочей области Azure Log Analytics. Этот каталог также Аппендера log4j для журналов приложений Apache Spark.

**Spark прослушивателей loganalytics** и **spark прослушиватели** каталоги содержат код для создания двух JAR-файлы, которые будут развернуты в кластере Databricks. **Spark прослушиватели** каталог включает **сценариев** каталог, содержащий сценарий инициализации узла кластера для копирования JAR-ФАЙЛ файлы из промежуточного каталога в файловой системе Azure Databricks для узлы выполнения.

**Pom.xml** это основной файл сборки Maven для всего проекта.

## <a name="prerequisites"></a>Технические условия

Чтобы приступить к работе, разверните следующие ресурсы Azure:

- В рабочей области Azure Databricks. См. [Краткое руководство. Запуск задания Spark в Azure Databricks с помощью портала Azure](/azure/azure-databricks/quickstart-create-databricks-workspace-portal).
- Рабочая область Log Analytics. См. в разделе [создать рабочую область Log Analytics на портале Azure](/azure/azure-monitor/learn/quick-create-workspace).

Затем установите [интерфейса командной строки Azure Databricks](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html#install-the-cli). Azure Databricks личный маркер доступа необходим для использования интерфейса командной строки. Инструкции см. в разделе [создать маркер](https://docs.azuredatabricks.net/api/latest/authentication.html#token-management).

Найти свой идентификатор рабочей области Log Analytics и ключ. Инструкции см. в разделе [получить идентификатор и ключ](/azure/azure-monitor/platform/agent-windows#obtain-workspace-id-and-key). Эти значения потребуются при настройке кластера Databricks.

Чтобы построить библиотеку мониторинга, потребуется это интегрированная среда разработки Java со следующими ресурсами:

- [Java Development Kit (JDK) версии 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
- [Язык Scala SDK 2.11](https://www.scala-lang.org/download/)
- [Apache Maven 3.5.4](http://maven.apache.org/download.cgi)

## <a name="build-the-azure-databricks-monitoring-library"></a>Создание Azure Databricks, библиотека мониторинга

Чтобы выполнить сборку библиотеки мониторинга Azure Databricks, выполните следующие действия.

1. Клонировать, разветвление или загрузить [репозиторий GitHub](https://github.com/mspnp/spark-monitoring).

1. Импорт файла модели объекта проекта Maven, _pom.xml_, который находится в **/src** папки в проект. Будут импортированы три проекта:

    - заданий Spark
    - Spark прослушиватели
    - Spark прослушивателей loganalytics

1. Выполнение Maven **пакета** этапа построения в интегрированной среде разработки на Java для создания JAR-файлы для каждого из этих проектов:

    |Project| JAR-файл|
    |-------|---------|
    |заданий Spark|Spark задания-1.0-SNAPSHOT.jar|
    |Spark прослушиватели|Spark прослушивателей-1.0-SNAPSHOT.jar|
    |Spark прослушивателей loganalytics|Spark прослушивателей loganalytics-1.0-SNAPSHOT.jar|

1. Использование интерфейса командной строки Databricks Azure создать каталог с именем **dbfs: / databricks, мониторинг-промежуточное хранение**:  

    ```bash
    dbfs mkdirs dbfs:/databricks/monitoring-staging
    ```

1. Использование интерфейса командной строки Databricks Azure для копирования **/src/spark-listeners/scripts/listeners.sh** ранее в каталог:

    ```bash
    dbfs cp ./src/spark-listeners/scripts/listeners.sh dbfs:/databricks/monitoring-staging/listeners.sh
    ```

1. Использование интерфейса командной строки Databricks Azure для копирования **/src/spark-listeners/scripts/metrics.properties** в каталог, созданный ранее:

    ```bash
    dbfs cp <local path to metrics.properties> dbfs:/databricks/monitoring-staging/metrics.properties
    ```

1. Использование интерфейса командной строки Databricks Azure для копирования **spark прослушивателей-1.0-SNAPSHOT.jar** и **spark прослушивателей loganalytics-1.0-SNAPSHOT.jar** в каталог, созданный ранее:

    ```bash
    dbfs cp ./src/spark-listeners/target/spark-listeners-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-1.0-SNAPSHOT.jar
    dbfs cp ./src/spark-listeners-loganalytics/target/spark-listeners-loganalytics-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-loganalytics-1.0-SNAPSHOT.jar
    ```

## <a name="create-and-configure-an-azure-databricks-cluster"></a>Создать и настроить кластер Azure Databricks

Чтобы создать и настроить кластер Azure Databricks, выполните следующие действия.

1. Перейдите к рабочей области Azure Databricks на портале Azure.
1. На домашней странице, щелкните **новый кластер**.
1. Введите имя для своего кластера на **имя кластера** текстовое поле.
1. В **версией среды выполнения Databricks** раскрывающемся списке выберите **4.3** или более поздней версии.
1. В разделе **Дополнительно**, щелкните **Spark** вкладки. Введите следующие пары имя значение в **конфигурации Spark** текстового поля:

    ```
    spark.extraListeners com.databricks.backend.daemon.driver.DBCEventLoggingListener,org.apache.spark.listeners.UnifiedSparkListener
    spark.unifiedListener.sink org.apache.spark.listeners.sink.loganalytics.LogAnalyticsListenerSink
    spark.unifiedListener.logBlockUpdates false
    ```

1. Хотя по-прежнему в разделе **Spark** введите следующие значения в **переменные среды** текстового поля:

    ```
    LOG_ANALYTICS_WORKSPACE_ID=[your Azure Log Analytics workspace ID]
    LOG_ANALYTICS_WORKSPACE_KEY=[your Azure Log Analytics workspace key]
    ```

    ![Снимок экрана пользовательского интерфейса Databricks](./_images/create-cluster1.png)

1. Хотя по-прежнему в разделе **Дополнительно** щелкните **скрипты Init** вкладки.
1. В **назначения** раскрывающемся списке выберите **DBFS**.
1. В разделе **путь к скрипту Init**, введите `dbfs:/databricks/monitoring-staging/listeners.sh`.
1. Щелкните **Добавить**.

    ![Снимок экрана пользовательского интерфейса Databricks](./_images/create-cluster2.png)

1. Нажмите кнопку **создания кластеров** для создания кластера.

## <a name="view-metrics"></a>Просмотр метрик

После выполнения этих действий кластера Databricks выполняет потоковую передачу некоторые данные метрик, о самого кластера в Azure Monitor. Данные журналов доступен в рабочей области Azure Log Analytics в разделе «Active | Пользовательские журналы | Схема SparkMetric_CL». Дополнительные сведения о типах метрик, которые вошли см. в разделе [Core метрики](https://metrics.dropwizard.io/4.0.0/manual/core.html) в документации по Dropwizard. Тип метрики, например датчиков, счетчика или гистограммы, записывается в поле «тип».

Кроме того библиотека выполняет потоковую передачу, события уровня Apache Spark и структурированной потоковой передачи Spark метрики из заданий в Azure Monitor. Не нужно вносить изменения в код приложения для этих событий и метрик. Данные журнала структурированной потоковой передачи Spark доступен в разделе «Active | Пользовательские журналы | Схема SparkListenerEvent_CL».

![Снимок экрана с рабочей областью Log Analytics](./_images/workspace.png)

Дополнительные сведения о просмотре журналов см. в разделе [Просмотр и анализ данных журнала в Azure Monitor](/azure/azure-monitor/log-query/portals).

## <a name="next-steps"></a>Дальнейшие действия

В этой статье описано, как настроить кластер для отправки метрик в Azure Monitor. Следующая статья показано, как использовать библиотеку Databricks мониторинга Azure для отправки приложения метрики и журналы.

> [!div class="nextstepaction"]
> [Отправка журналов приложения в Azure Monitor](./application-logs.md)
