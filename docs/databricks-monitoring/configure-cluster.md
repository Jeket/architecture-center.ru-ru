---
title: Настройка Azure Databricks для отправки метрик в Azure Monitor
description: Это библиотека scala, чтобы включить мониторинг метрик и ведения журнала данных в Azure Log Analytics
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: f2fc1fd19da661b74ddf032dd1d5153ce575345c
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59639892"
---
<!-- markdownlint-disable MD040 -->

# <a name="configure-azure-databricks-to-send-metrics-to-azure-monitor"></a><span data-ttu-id="997db-103">Настройка Azure Databricks для отправки метрик в Azure Monitor</span><span class="sxs-lookup"><span data-stu-id="997db-103">Configure Azure Databricks to send metrics to Azure Monitor</span></span>

<span data-ttu-id="997db-104">В этой статье показано, как настроить кластер Azure Databricks для отправки метрик для [рабочей области Log Analytics](/azure/azure-monitor/platform/manage-access).</span><span class="sxs-lookup"><span data-stu-id="997db-104">This article shows how to configure an Azure Databricks cluster to send metrics to a [Log Analytics workspace](/azure/azure-monitor/platform/manage-access).</span></span> <span data-ttu-id="997db-105">Она использует [библиотека мониторинга Azure Databricks](https://github.com/mspnp/spark-monitoring), которая доступна на сайте GitHub.</span><span class="sxs-lookup"><span data-stu-id="997db-105">It uses the [Azure Databricks Monitoring Library](https://github.com/mspnp/spark-monitoring), which is available on GitHub.</span></span> <span data-ttu-id="997db-106">Основные сведения о Java, Scala и Maven рекомендуется в качестве необходимых компонентов.</span><span class="sxs-lookup"><span data-stu-id="997db-106">Understanding of Java, Scala, and Maven are recommended as prerequisites.</span></span>

## <a name="about-the-azure-databricks-monitoring-library"></a><span data-ttu-id="997db-107">Об Azure Databricks, библиотека мониторинга</span><span class="sxs-lookup"><span data-stu-id="997db-107">About the Azure Databricks Monitoring Library</span></span>

<span data-ttu-id="997db-108">[Репозиторий GitHub](https://github.com/mspnp/spark-monitoring) для библиотеки мониторинга Azure Databricks имеет следующие структуру каталогов:</span><span class="sxs-lookup"><span data-stu-id="997db-108">The [GitHub repo](https://github.com/mspnp/spark-monitoring) for the Azure Databricks Monitoring Library has following directory structure:</span></span>

```
/src  
    /spark-jobs  
    /spark-listeners-loganalytics  
    /spark-listeners  
    /pom.xml  
```

<span data-ttu-id="997db-109">**Заданий spark** каталог — пример приложения Spark, демонстрирующий, как реализовать счетчик метрик приложения Spark.</span><span class="sxs-lookup"><span data-stu-id="997db-109">The **spark-jobs** directory is a sample Spark application demonstrating how to implement a Spark application metric counter.</span></span>

<span data-ttu-id="997db-110">**Spark прослушиватели** каталог включает функциональность, позволяющую Databricks Azure для отправки событий Apache Spark на уровне службы в рабочую область Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="997db-110">The **spark-listeners** directory includes functionality that enables Azure Databricks to send Apache Spark events at the service level to an Azure Log Analytics workspace.</span></span> <span data-ttu-id="997db-111">Azure Databricks — это служба, на основе Apache Spark, который включает набор структурированных API для пакетной обработки данных с помощью наборов данных, таблицы данных и SQL.</span><span class="sxs-lookup"><span data-stu-id="997db-111">Azure Databricks is a service based on Apache Spark, which includes a set of structured APIs for batch processing data using Datasets, DataFrames, and SQL.</span></span> <span data-ttu-id="997db-112">С помощью Apache Spark 2.0 была добавлена поддержка [структурированной потоковой передачи](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html), поток данных обработки API, созданный на обработку интерфейсы API пакета Spark.</span><span class="sxs-lookup"><span data-stu-id="997db-112">With Apache Spark 2.0, support was added for [Structured Streaming](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html), a data stream processing API built on Spark's batch processing APIs.</span></span>

<span data-ttu-id="997db-113">**Spark прослушивателей loganalytics** каталог включает приемника для Spark прослушиватели приемника для DropWizard и клиента для рабочей области Azure Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="997db-113">The **spark-listeners-loganalytics** directory includes a sink for Spark listeners, a sink for DropWizard, and a client for an Azure Log Analytics Workspace.</span></span> <span data-ttu-id="997db-114">Этот каталог также Аппендера log4j для журналов приложений Apache Spark.</span><span class="sxs-lookup"><span data-stu-id="997db-114">This directory also includes a log4j Appender for your Apache Spark application logs.</span></span>

<span data-ttu-id="997db-115">**Spark прослушивателей loganalytics** и **spark прослушиватели** каталоги содержат код для создания двух JAR-файлы, которые будут развернуты в кластере Databricks.</span><span class="sxs-lookup"><span data-stu-id="997db-115">The **spark-listeners-loganalytics** and **spark-listeners** directories contain the code for building the two JAR files that are deployed to the Databricks cluster.</span></span> <span data-ttu-id="997db-116">**Spark прослушиватели** каталог включает **сценариев** каталог, содержащий сценарий инициализации узла кластера для копирования JAR-ФАЙЛ файлы из промежуточного каталога в файловой системе Azure Databricks для узлы выполнения.</span><span class="sxs-lookup"><span data-stu-id="997db-116">The **spark-listeners** directory includes a **scripts** directory that contains a cluster node initialization script to copy the JAR files from a staging directory in the Azure Databricks file system to execution nodes.</span></span>

<span data-ttu-id="997db-117">**Pom.xml** это основной файл сборки Maven для всего проекта.</span><span class="sxs-lookup"><span data-stu-id="997db-117">The **pom.xml** file is the main Maven build file for the entire project.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="997db-118">Технические условия</span><span class="sxs-lookup"><span data-stu-id="997db-118">Prerequisites</span></span>

<span data-ttu-id="997db-119">Чтобы приступить к работе, разверните следующие ресурсы Azure:</span><span class="sxs-lookup"><span data-stu-id="997db-119">To get started, deploy the following Azure resources:</span></span>

- <span data-ttu-id="997db-120">В рабочей области Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="997db-120">An Azure Databricks workspace.</span></span> <span data-ttu-id="997db-121">См. [Краткое руководство. Запуск задания Spark в Azure Databricks с помощью портала Azure](/azure/azure-databricks/quickstart-create-databricks-workspace-portal).</span><span class="sxs-lookup"><span data-stu-id="997db-121">See [Quickstart: Run a Spark job on Azure Databricks using the Azure portal](/azure/azure-databricks/quickstart-create-databricks-workspace-portal).</span></span>
- <span data-ttu-id="997db-122">Рабочая область Log Analytics.</span><span class="sxs-lookup"><span data-stu-id="997db-122">A Log Analytics workspace.</span></span> <span data-ttu-id="997db-123">См. в разделе [создать рабочую область Log Analytics на портале Azure](/azure/azure-monitor/learn/quick-create-workspace).</span><span class="sxs-lookup"><span data-stu-id="997db-123">See [Create a Log Analytics workspace in the Azure portal](/azure/azure-monitor/learn/quick-create-workspace).</span></span>

<span data-ttu-id="997db-124">Затем установите [интерфейса командной строки Azure Databricks](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html#install-the-cli).</span><span class="sxs-lookup"><span data-stu-id="997db-124">Next, install the [Azure Databricks CLI](https://docs.databricks.com/user-guide/dev-tools/databricks-cli.html#install-the-cli).</span></span> <span data-ttu-id="997db-125">Azure Databricks личный маркер доступа необходим для использования интерфейса командной строки.</span><span class="sxs-lookup"><span data-stu-id="997db-125">An Azure Databricks personal access token is required to use the CLI.</span></span> <span data-ttu-id="997db-126">Инструкции см. в разделе [создать маркер](https://docs.azuredatabricks.net/api/latest/authentication.html#token-management).</span><span class="sxs-lookup"><span data-stu-id="997db-126">For instructions, see [Generate a token](https://docs.azuredatabricks.net/api/latest/authentication.html#token-management).</span></span>

<span data-ttu-id="997db-127">Найти свой идентификатор рабочей области Log Analytics и ключ.</span><span class="sxs-lookup"><span data-stu-id="997db-127">Find your Log Analytics workspace ID and key.</span></span> <span data-ttu-id="997db-128">Инструкции см. в разделе [получить идентификатор и ключ](/azure/azure-monitor/platform/agent-windows#obtain-workspace-id-and-key).</span><span class="sxs-lookup"><span data-stu-id="997db-128">For instructions, see [Obtain workspace ID and key](/azure/azure-monitor/platform/agent-windows#obtain-workspace-id-and-key).</span></span> <span data-ttu-id="997db-129">Эти значения потребуются при настройке кластера Databricks.</span><span class="sxs-lookup"><span data-stu-id="997db-129">You will need these values when you configure the Databricks cluster.</span></span>

<span data-ttu-id="997db-130">Чтобы построить библиотеку мониторинга, потребуется это интегрированная среда разработки Java со следующими ресурсами:</span><span class="sxs-lookup"><span data-stu-id="997db-130">To build the monitoring library, you will need a Java IDE with the following resources:</span></span>

- [<span data-ttu-id="997db-131">Java Development Kit (JDK) версии 1.8</span><span class="sxs-lookup"><span data-stu-id="997db-131">Java Development Kit (JDK) version 1.8</span></span>](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
- [<span data-ttu-id="997db-132">Язык Scala SDK 2.11</span><span class="sxs-lookup"><span data-stu-id="997db-132">Scala language SDK 2.11</span></span>](https://www.scala-lang.org/download/)
- [<span data-ttu-id="997db-133">Apache Maven 3.5.4</span><span class="sxs-lookup"><span data-stu-id="997db-133">Apache Maven 3.5.4</span></span>](http://maven.apache.org/download.cgi)

## <a name="build-the-azure-databricks-monitoring-library"></a><span data-ttu-id="997db-134">Создание Azure Databricks, библиотека мониторинга</span><span class="sxs-lookup"><span data-stu-id="997db-134">Build the Azure Databricks Monitoring Library</span></span>

<span data-ttu-id="997db-135">Чтобы выполнить сборку библиотеки мониторинга Azure Databricks, выполните следующие действия.</span><span class="sxs-lookup"><span data-stu-id="997db-135">To build the Azure Databricks Monitoring Library, perform the following steps:</span></span>

1. <span data-ttu-id="997db-136">Клонировать, разветвление или загрузить [репозиторий GitHub](https://github.com/mspnp/spark-monitoring).</span><span class="sxs-lookup"><span data-stu-id="997db-136">Clone, fork, or download the [GitHub repository](https://github.com/mspnp/spark-monitoring).</span></span>

1. <span data-ttu-id="997db-137">Импорт файла модели объекта проекта Maven, _pom.xml_, который находится в **/src** папки в проект.</span><span class="sxs-lookup"><span data-stu-id="997db-137">Import the Maven project object model file, _pom.xml_, located in the **/src** folder into your project.</span></span> <span data-ttu-id="997db-138">Будут импортированы три проекта:</span><span class="sxs-lookup"><span data-stu-id="997db-138">This will import three projects:</span></span>

    - <span data-ttu-id="997db-139">заданий Spark</span><span class="sxs-lookup"><span data-stu-id="997db-139">spark-jobs</span></span>
    - <span data-ttu-id="997db-140">Spark прослушиватели</span><span class="sxs-lookup"><span data-stu-id="997db-140">spark-listeners</span></span>
    - <span data-ttu-id="997db-141">Spark прослушивателей loganalytics</span><span class="sxs-lookup"><span data-stu-id="997db-141">spark-listeners-loganalytics</span></span>

1. <span data-ttu-id="997db-142">Выполнение Maven **пакета** этапа построения в интегрированной среде разработки на Java для создания JAR-файлы для каждого из этих проектов:</span><span class="sxs-lookup"><span data-stu-id="997db-142">Execute the Maven **package** build phase in your Java IDE to build the JAR files for each of these projects:</span></span>

    |<span data-ttu-id="997db-143">Project</span><span class="sxs-lookup"><span data-stu-id="997db-143">Project</span></span>| <span data-ttu-id="997db-144">JAR-файл</span><span class="sxs-lookup"><span data-stu-id="997db-144">JAR file</span></span>|
    |-------|---------|
    |<span data-ttu-id="997db-145">заданий Spark</span><span class="sxs-lookup"><span data-stu-id="997db-145">spark-jobs</span></span>|<span data-ttu-id="997db-146">Spark задания-1.0-SNAPSHOT.jar</span><span class="sxs-lookup"><span data-stu-id="997db-146">spark-jobs-1.0-SNAPSHOT.jar</span></span>|
    |<span data-ttu-id="997db-147">Spark прослушиватели</span><span class="sxs-lookup"><span data-stu-id="997db-147">spark-listeners</span></span>|<span data-ttu-id="997db-148">Spark прослушивателей-1.0-SNAPSHOT.jar</span><span class="sxs-lookup"><span data-stu-id="997db-148">spark-listeners-1.0-SNAPSHOT.jar</span></span>|
    |<span data-ttu-id="997db-149">Spark прослушивателей loganalytics</span><span class="sxs-lookup"><span data-stu-id="997db-149">spark-listeners-loganalytics</span></span>|<span data-ttu-id="997db-150">Spark прослушивателей loganalytics-1.0-SNAPSHOT.jar</span><span class="sxs-lookup"><span data-stu-id="997db-150">spark-listeners-loganalytics-1.0-SNAPSHOT.jar</span></span>|

1. <span data-ttu-id="997db-151">Использование интерфейса командной строки Databricks Azure создать каталог с именем **dbfs: / databricks, мониторинг-промежуточное хранение**:</span><span class="sxs-lookup"><span data-stu-id="997db-151">Use the Azure Databricks CLI to create a directory named **dbfs:/databricks/monitoring-staging**:</span></span>  

    ```bash
    dbfs mkdirs dbfs:/databricks/monitoring-staging
    ```

1. <span data-ttu-id="997db-152">Использование интерфейса командной строки Databricks Azure для копирования **/src/spark-listeners/scripts/listeners.sh** ранее в каталог:</span><span class="sxs-lookup"><span data-stu-id="997db-152">Use the Azure Databricks CLI to copy **/src/spark-listeners/scripts/listeners.sh** to the directory previously:</span></span>

    ```bash
    dbfs cp ./src/spark-listeners/scripts/listeners.sh dbfs:/databricks/monitoring-staging/listeners.sh
    ```

1. <span data-ttu-id="997db-153">Использование интерфейса командной строки Databricks Azure для копирования **/src/spark-listeners/scripts/metrics.properties** в каталог, созданный ранее:</span><span class="sxs-lookup"><span data-stu-id="997db-153">Use the Azure Databricks CLI to copy **/src/spark-listeners/scripts/metrics.properties** to the directory created previously:</span></span>

    ```bash
    dbfs cp <local path to metrics.properties> dbfs:/databricks/monitoring-staging/metrics.properties
    ```

1. <span data-ttu-id="997db-154">Использование интерфейса командной строки Databricks Azure для копирования **spark прослушивателей-1.0-SNAPSHOT.jar** и **spark прослушивателей loganalytics-1.0-SNAPSHOT.jar** в каталог, созданный ранее:</span><span class="sxs-lookup"><span data-stu-id="997db-154">Use the Azure Databricks CLI to copy **spark-listeners-1.0-SNAPSHOT.jar** and **spark-listeners-loganalytics-1.0-SNAPSHOT.jar** to the directory created previously:</span></span>

    ```bash
    dbfs cp ./src/spark-listeners/target/spark-listeners-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-1.0-SNAPSHOT.jar
    dbfs cp ./src/spark-listeners-loganalytics/target/spark-listeners-loganalytics-1.0-SNAPSHOT.jar dbfs:/databricks/monitoring-staging/spark-listeners-loganalytics-1.0-SNAPSHOT.jar
    ```

## <a name="create-and-configure-an-azure-databricks-cluster"></a><span data-ttu-id="997db-155">Создать и настроить кластер Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="997db-155">Create and configure an Azure Databricks cluster</span></span>

<span data-ttu-id="997db-156">Чтобы создать и настроить кластер Azure Databricks, выполните следующие действия.</span><span class="sxs-lookup"><span data-stu-id="997db-156">To create and configure an Azure Databricks cluster, follow these steps:</span></span>

1. <span data-ttu-id="997db-157">Перейдите к рабочей области Azure Databricks на портале Azure.</span><span class="sxs-lookup"><span data-stu-id="997db-157">Navigate to your Azure Databricks workspace in the Azure portal.</span></span>
1. <span data-ttu-id="997db-158">На домашней странице, щелкните **новый кластер**.</span><span class="sxs-lookup"><span data-stu-id="997db-158">On the home page, click **New Cluster**.</span></span>
1. <span data-ttu-id="997db-159">Введите имя для своего кластера на **имя кластера** текстовое поле.</span><span class="sxs-lookup"><span data-stu-id="997db-159">Enter a name for your cluster in the **Cluster Name** text box.</span></span>
1. <span data-ttu-id="997db-160">В **версией среды выполнения Databricks** раскрывающемся списке выберите **4.3** или более поздней версии.</span><span class="sxs-lookup"><span data-stu-id="997db-160">In the **Databricks Runtime Version** dropdown, select **4.3** or greater.</span></span>
1. <span data-ttu-id="997db-161">В разделе **Дополнительно**, щелкните **Spark** вкладки. Введите следующие пары имя значение в **конфигурации Spark** текстового поля:</span><span class="sxs-lookup"><span data-stu-id="997db-161">Under **Advanced Options**, click on the **Spark** tab. Enter the following name-value pairs in the **Spark Config** text box:</span></span>

    ```
    spark.extraListeners com.databricks.backend.daemon.driver.DBCEventLoggingListener,org.apache.spark.listeners.UnifiedSparkListener
    spark.unifiedListener.sink org.apache.spark.listeners.sink.loganalytics.LogAnalyticsListenerSink
    spark.unifiedListener.logBlockUpdates false
    ```

1. <span data-ttu-id="997db-162">Хотя по-прежнему в разделе **Spark** введите следующие значения в **переменные среды** текстового поля:</span><span class="sxs-lookup"><span data-stu-id="997db-162">While still under the **Spark** tab, enter the following values in the **Environment Variables** text box:</span></span>

    ```
    LOG_ANALYTICS_WORKSPACE_ID=[your Azure Log Analytics workspace ID]
    LOG_ANALYTICS_WORKSPACE_KEY=[your Azure Log Analytics workspace key]
    ```

    ![Снимок экрана пользовательского интерфейса Databricks](./_images/create-cluster1.png)

1. <span data-ttu-id="997db-164">Хотя по-прежнему в разделе **Дополнительно** щелкните **скрипты Init** вкладки.</span><span class="sxs-lookup"><span data-stu-id="997db-164">While still under the **Advanced Options** section, click on the **Init Scripts** tab.</span></span>
1. <span data-ttu-id="997db-165">В **назначения** раскрывающемся списке выберите **DBFS**.</span><span class="sxs-lookup"><span data-stu-id="997db-165">In the **Destination** dropdown, select **DBFS**.</span></span>
1. <span data-ttu-id="997db-166">В разделе **путь к скрипту Init**, введите `dbfs:/databricks/monitoring-staging/listeners.sh`.</span><span class="sxs-lookup"><span data-stu-id="997db-166">Under **Init Script Path**, enter `dbfs:/databricks/monitoring-staging/listeners.sh`.</span></span>
1. <span data-ttu-id="997db-167">Щелкните **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="997db-167">Click **Add**.</span></span>

    ![Снимок экрана пользовательского интерфейса Databricks](./_images/create-cluster2.png)

1. <span data-ttu-id="997db-169">Нажмите кнопку **создания кластеров** для создания кластера.</span><span class="sxs-lookup"><span data-stu-id="997db-169">Click **Create Cluster** to create the cluster.</span></span>

## <a name="view-metrics"></a><span data-ttu-id="997db-170">Просмотр метрик</span><span class="sxs-lookup"><span data-stu-id="997db-170">View metrics</span></span>

<span data-ttu-id="997db-171">После выполнения этих действий кластера Databricks выполняет потоковую передачу некоторые данные метрик, о самого кластера в Azure Monitor.</span><span class="sxs-lookup"><span data-stu-id="997db-171">After you complete these steps, your Databricks cluster streams some metric data about the cluster itself to Azure Monitor.</span></span> <span data-ttu-id="997db-172">Данные журналов доступен в рабочей области Azure Log Analytics в разделе «Active | Пользовательские журналы | Схема SparkMetric_CL».</span><span class="sxs-lookup"><span data-stu-id="997db-172">This log data is available in your Azure Log Analytics workspace under the "Active | Custom Logs | SparkMetric_CL" schema.</span></span> <span data-ttu-id="997db-173">Дополнительные сведения о типах метрик, которые вошли см. в разделе [Core метрики](https://metrics.dropwizard.io/4.0.0/manual/core.html) в документации по Dropwizard.</span><span class="sxs-lookup"><span data-stu-id="997db-173">For more information about the types of metrics that are logged, see [Metrics Core](https://metrics.dropwizard.io/4.0.0/manual/core.html) in the Dropwizard documentation.</span></span> <span data-ttu-id="997db-174">Тип метрики, например датчиков, счетчика или гистограммы, записывается в поле «тип».</span><span class="sxs-lookup"><span data-stu-id="997db-174">The metric type, such as gauge, counter, or histogram, is written to the Type field.</span></span>

<span data-ttu-id="997db-175">Кроме того библиотека выполняет потоковую передачу, события уровня Apache Spark и структурированной потоковой передачи Spark метрики из заданий в Azure Monitor.</span><span class="sxs-lookup"><span data-stu-id="997db-175">In addition, the library streams Apache Spark level events and Spark Structured Streaming metrics from your jobs to Azure Monitor.</span></span> <span data-ttu-id="997db-176">Не нужно вносить изменения в код приложения для этих событий и метрик.</span><span class="sxs-lookup"><span data-stu-id="997db-176">You don't need to make any changes to your application code for these events and metrics.</span></span> <span data-ttu-id="997db-177">Данные журнала структурированной потоковой передачи Spark доступен в разделе «Active | Пользовательские журналы | Схема SparkListenerEvent_CL».</span><span class="sxs-lookup"><span data-stu-id="997db-177">Spark Structured Streaming log data is available under the "Active | Custom Logs | SparkListenerEvent_CL" schema.</span></span>

![Снимок экрана с рабочей областью Log Analytics](./_images/workspace.png)

<span data-ttu-id="997db-179">Дополнительные сведения о просмотре журналов см. в разделе [Просмотр и анализ данных журнала в Azure Monitor](/azure/azure-monitor/log-query/portals).</span><span class="sxs-lookup"><span data-stu-id="997db-179">For more information about viewing logs, see [Viewing and analyzing log data in Azure Monitor](/azure/azure-monitor/log-query/portals).</span></span>

## <a name="next-steps"></a><span data-ttu-id="997db-180">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="997db-180">Next steps</span></span>

<span data-ttu-id="997db-181">В этой статье описано, как настроить кластер для отправки метрик в Azure Monitor.</span><span class="sxs-lookup"><span data-stu-id="997db-181">This article described how to configure your cluster to send metrics to Azure Monitor.</span></span> <span data-ttu-id="997db-182">Следующая статья показано, как использовать библиотеку Databricks мониторинга Azure для отправки приложения метрики и журналы.</span><span class="sxs-lookup"><span data-stu-id="997db-182">The next article shows how to use the Azure Databricks Monitoring Library to send application metrics and logs.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="997db-183">Отправка журналов приложения в Azure Monitor</span><span class="sxs-lookup"><span data-stu-id="997db-183">Send application logs to Azure Monitor</span></span>](./application-logs.md)
