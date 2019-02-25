---
title: Создание API рекомендаций в режиме реального времени в Azure
description: Использование машинного обучения для автоматизации рекомендаций с помощью Azure Databricks и виртуальных машин для обработки и анализа данных Azure для обучения модели в Azure.
author: njray
ms.date: 12/12/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: azcat-ai
ms.openlocfilehash: c4bfd6e92fc9c770a03a63355fc922d19ef27b7b
ms.sourcegitcommit: f4ed242dff8b204cfd8ebebb7778f356a19f5923
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/13/2019
ms.locfileid: "56224170"
---
# <a name="build-a-real-time-recommendation-api-on-azure"></a><span data-ttu-id="51cc4-103">Создание API рекомендаций в режиме реального времени в Azure</span><span class="sxs-lookup"><span data-stu-id="51cc4-103">Build a real-time recommendation API on Azure</span></span>

<span data-ttu-id="51cc4-104">В этой эталонной архитектуре показано, как обучить модель рекомендаций с помощью Azure Databricks и развернуть ее как API с помощью Azure Cosmos DB, Машинного обучения Azure и Службы Azure Kubernetes (AKS).</span><span class="sxs-lookup"><span data-stu-id="51cc4-104">This reference architecture shows how to train a recommendation model using Azure Databricks and deploy it as an API by using Azure Cosmos DB, Azure Machine Learning, and Azure Kubernetes Service (AKS).</span></span> <span data-ttu-id="51cc4-105">Эту архитектуру можно подготовить к использованию для большинства сценариев механизма рекомендаций, включая рекомендации для продуктов, фильмов и новостей.</span><span class="sxs-lookup"><span data-stu-id="51cc4-105">This architecture can be generalized for most recommendation engine scenarios, including recommendations for products, movies, and news.</span></span>

<span data-ttu-id="51cc4-106">Эталонную реализацию для этой архитектуры можно найти на сайте [GitHub](https://github.com/Microsoft/Recommenders/blob/master/notebooks/05_operationalize/als_movie_o16n.ipynb).</span><span class="sxs-lookup"><span data-stu-id="51cc4-106">A reference implementation for this architecture is available on [GitHub](https://github.com/Microsoft/Recommenders/blob/master/notebooks/05_operationalize/als_movie_o16n.ipynb).</span></span>

![Архитектура модели машинного обучения для создания рекомендаций к фильмам](./_images/recommenders-architecture.png)

<span data-ttu-id="51cc4-108">**Сценарий**. Вещательная организация хочет предоставить своим пользователям рекомендации к фильмам или видео.</span><span class="sxs-lookup"><span data-stu-id="51cc4-108">**Scenario**: A media organization wants to provide movie or video recommendations to its users.</span></span> <span data-ttu-id="51cc4-109">Предоставляя персональные рекомендации, организация достигает несколько бизнес-целей, включая повышение коэффициента переходов, рост популярности сайта и повышение удовлетворенности пользователей.</span><span class="sxs-lookup"><span data-stu-id="51cc4-109">By providing personalized recommendations, the organization meets several business goals, including increased click-through rates, increased engagement on site, and higher user satisfaction.</span></span>

<span data-ttu-id="51cc4-110">Эта эталонная архитектура предназначена для обучения и развертывания API службы рекомендаций в режиме реального времени, которая может предоставить 10 лучших рекомендаций по фильмам для данного пользователя.</span><span class="sxs-lookup"><span data-stu-id="51cc4-110">This reference architecture is for training and deploying a real-time recommender service API that can provide the top 10 movie recommendations for a given user.</span></span>

<span data-ttu-id="51cc4-111">Поток данных для этой модели рекомендаций выглядит следующим образом.</span><span class="sxs-lookup"><span data-stu-id="51cc4-111">The data flow for this recommendation model is as follows:</span></span>

1. <span data-ttu-id="51cc4-112">Отслеживание поведения пользователя.</span><span class="sxs-lookup"><span data-stu-id="51cc4-112">Track user behaviors.</span></span> <span data-ttu-id="51cc4-113">Например, серверная служба может регистрировать, когда пользователь оценивает фильм, щелкает продукт или новостную статью.</span><span class="sxs-lookup"><span data-stu-id="51cc4-113">For example, a backend service might log when a user rates a movie or clicks a product or news article.</span></span>

2. <span data-ttu-id="51cc4-114">Загрузка данных в Azure Databricks из доступного [источника данных][data-source].</span><span class="sxs-lookup"><span data-stu-id="51cc4-114">Load the data into Azure Databricks from an available [data source][data-source].</span></span>

3. <span data-ttu-id="51cc4-115">Подготовка данных и их разделение на наборы для обучения и тестирования модели.</span><span class="sxs-lookup"><span data-stu-id="51cc4-115">Prepare the data and split it into training and testing sets to train the model.</span></span> <span data-ttu-id="51cc4-116">(В [этом руководстве ][guide] описаны параметры для разделения данных).</span><span class="sxs-lookup"><span data-stu-id="51cc4-116">([This guide][guide] describes options for splitting data.)</span></span>

4. <span data-ttu-id="51cc4-117">Настройте модель [Spark Collaborative Filtering][als] согласно данным.</span><span class="sxs-lookup"><span data-stu-id="51cc4-117">Fit the [Spark Collaborative Filtering][als] model to the data.</span></span>

5. <span data-ttu-id="51cc4-118">Оцените качество модели, используя рейтинг и метрики ранжирования.</span><span class="sxs-lookup"><span data-stu-id="51cc4-118">Evaluate the quality of the model using rating and ranking metrics.</span></span> <span data-ttu-id="51cc4-119">(В [этом руководстве][eval-guide] содержатся сведения о метриках, по которым вы можете оценивать рекомендации).</span><span class="sxs-lookup"><span data-stu-id="51cc4-119">([This guide][eval-guide] provides details about the metrics you can evaluate your recommender on.)</span></span>

6. <span data-ttu-id="51cc4-120">Предварительно вычислите 10 рекомендаций для каждого пользователя и сохраните их в виде кэша в Azure Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="51cc4-120">Precompute the top 10 recommendations per user and store as a cache in Azure Cosmos DB.</span></span>

7. <span data-ttu-id="51cc4-121">Разверните службу API в AKS с помощью API Машинного обучения Azure для контейнеризации и развертывания API.</span><span class="sxs-lookup"><span data-stu-id="51cc4-121">Deploy an API service to AKS using the Azure Machine Learning APIs to containerize and deploy the API.</span></span>

8. <span data-ttu-id="51cc4-122">Когда серверная служба получит запрос от пользователя, вызовите API рекомендаций, размещенный в AKS, чтобы получить 10 лучших рекомендаций и отобразить их пользователю.</span><span class="sxs-lookup"><span data-stu-id="51cc4-122">When the backend service gets a request from a user, call the recommendations API hosted in AKS to get the top 10 recommendations and display them to the user.</span></span>

## <a name="architecture"></a><span data-ttu-id="51cc4-123">Архитектура</span><span class="sxs-lookup"><span data-stu-id="51cc4-123">Architecture</span></span>

<span data-ttu-id="51cc4-124">Эта архитектура состоит из следующих компонентов.</span><span class="sxs-lookup"><span data-stu-id="51cc4-124">This architecture consists of the following components:</span></span>

<span data-ttu-id="51cc4-125">[Azure Databricks][databricks].</span><span class="sxs-lookup"><span data-stu-id="51cc4-125">[Azure Databricks][databricks].</span></span> <span data-ttu-id="51cc4-126">Databricks – это среда разработки, используемая для подготовки входных данных и обучения модели рекомендаций в кластере Spark.</span><span class="sxs-lookup"><span data-stu-id="51cc4-126">Databricks is a development environment used to prepare input data and train the recommender model on a Spark cluster.</span></span> <span data-ttu-id="51cc4-127">Azure Databricks также предоставляет интерактивную рабочую область для запуска и совместной работы над записными книжками для любых задач обработки данных или машинного обучения.</span><span class="sxs-lookup"><span data-stu-id="51cc4-127">Azure Databricks also provides an interactive workspace to run and collaborate on notebooks for any data processing or machine learning tasks.</span></span>

<span data-ttu-id="51cc4-128">[Служба Azure Kubernetes][aks] (AKS).</span><span class="sxs-lookup"><span data-stu-id="51cc4-128">[Azure Kubernetes Service][aks] (AKS).</span></span> <span data-ttu-id="51cc4-129">AKS используется для развертывания и эксплуатации API службы модели машинного обучения в кластере Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="51cc4-129">AKS is used to deploy and operationalize a machine learning model service API on a Kubernetes cluster.</span></span> <span data-ttu-id="51cc4-130">AKS поддерживает контейнерную модель, обеспечивая масштабируемость, соответствующую вашим требованиям к пропускной способности, управлению идентификацией и доступом, а также ведению журналов и мониторингу работоспособности.</span><span class="sxs-lookup"><span data-stu-id="51cc4-130">AKS hosts the containerized model, providing scalability that meets your throughput requirements, identity and access management, and logging and health monitoring.</span></span>

<span data-ttu-id="51cc4-131">[Azure Cosmos DB][cosmosdb].</span><span class="sxs-lookup"><span data-stu-id="51cc4-131">[Azure Cosmos DB][cosmosdb].</span></span> <span data-ttu-id="51cc4-132">Cosmos DB – это глобально распределенная служба базы данных, используемая для хранения 10 лучших рекомендуемых фильмов для каждого пользователя.</span><span class="sxs-lookup"><span data-stu-id="51cc4-132">Cosmos DB is a globally distributed database service used to store the top 10 recommended movies for each user.</span></span> <span data-ttu-id="51cc4-133">Azure Cosmos DB хорошо подходит для этого сценария, поскольку обеспечивает низкую задержку (10 мс при 99-м процентиле) для чтения 10 рекомендуемых элементов для данного пользователя.</span><span class="sxs-lookup"><span data-stu-id="51cc4-133">Azure Cosmos DB is well-suited for this scenario, because it provides low latency (10 ms at 99th percentile) to read the top recommended items for a given user.</span></span>

<span data-ttu-id="51cc4-134">[Служба машинного обучения Azure][mls].</span><span class="sxs-lookup"><span data-stu-id="51cc4-134">[Azure Machine Learning Service][mls].</span></span> <span data-ttu-id="51cc4-135">Эта служба используется для отслеживания и управления моделями машинного обучения, а затем для упаковки и развертывания этих моделей в масштабируемой среде AKS.</span><span class="sxs-lookup"><span data-stu-id="51cc4-135">This service is used to track and manage machine learning models, and then package and deploy these models to a scalable AKS environment.</span></span>

<span data-ttu-id="51cc4-136">[Microsoft Recommenders][github].</span><span class="sxs-lookup"><span data-stu-id="51cc4-136">[Microsoft Recommenders][github].</span></span> <span data-ttu-id="51cc4-137">Этот репозиторий с открытым исходным кодом содержит служебный код и примеры, которые помогут пользователям начать создавать, оценивать и вводить в действие систему рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="51cc4-137">This open-source repository contains utility code and samples to help users get started in building, evaluating, and operationalizing a recommender system.</span></span>

## <a name="performance-considerations"></a><span data-ttu-id="51cc4-138">Рекомендации по производительности</span><span class="sxs-lookup"><span data-stu-id="51cc4-138">Performance considerations</span></span>

<span data-ttu-id="51cc4-139">Производительность является основным фактором для рекомендаций в режиме реального времени, потому что рекомендации обычно находятся на критическом пути запроса, который пользователь делает на вашем сайте.</span><span class="sxs-lookup"><span data-stu-id="51cc4-139">Performance is a primary consideration for real-time recommendations, because recommendations usually fall in the critical path of the request a user makes on your site.</span></span>

<span data-ttu-id="51cc4-140">Сочетание AKS и Azure Cosmos DB позволяет этой архитектуре формировать хорошую отправную точку для рекомендаций для рабочей нагрузки среднего размера с минимальными издержками.</span><span class="sxs-lookup"><span data-stu-id="51cc4-140">The combination of AKS and Azure Cosmos DB enables this architecture to provide a good starting point to provide recommendations for a medium-sized workload with minimal overhead.</span></span> <span data-ttu-id="51cc4-141">При нагрузочном тесте с 200 одновременными пользователями эта архитектура предоставляет рекомендации со средней задержкой около 60 мс и пропускной способностью 180 запросов в секунду.</span><span class="sxs-lookup"><span data-stu-id="51cc4-141">Under a load test with 200 concurrent users, this architecture provides recommendations at a median latency of about 60 ms and performs at a throughput of 180 requests per second.</span></span> <span data-ttu-id="51cc4-142">Нагрузочный тест был выполнен с использованием конфигурации развертывания по умолчанию (кластер 3x AKS D3 v2 с 12 виртуальными ЦП, 42 ГБ памяти и 11 000 [единиц запросов (RU) в секунду][ru], выделенных для Azure Cosmos DB).</span><span class="sxs-lookup"><span data-stu-id="51cc4-142">The load test was run against the default deployment configuration (a 3x D3 v2 AKS cluster with 12 vCPUs, 42 GB of memory, and 11,000 [Request Units (RUs) per second][ru] provisioned for Azure Cosmos DB).</span></span>

![График производительности](./_images/recommenders-performance.png)

![График пропускной способности](./_images/recommenders-throughput.png)

<span data-ttu-id="51cc4-145">Azure Cosmos DB рекомендуется из-за ее удобства и возможности глобального распределения при удовлетворении любых требований к базе данных, которые есть у вашего приложения.</span><span class="sxs-lookup"><span data-stu-id="51cc4-145">Azure Cosmos DB is recommended for its turnkey global distribution and usefulness in meeting any database requirements your app has.</span></span> <span data-ttu-id="51cc4-146">Для незначительного [сокращения задержки][latency] попробуйте для обслуживания поиска использовать [кэш Redis для Azure][redis] вместо Azure Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="51cc4-146">For slightly [faster latency][latency], consider using [Azure Redis Cache][redis] instead of Azure Cosmos DB to serve lookups.</span></span> <span data-ttu-id="51cc4-147">Кэш Redis может повысить производительность систем, которые сильно зависят от данных в серверных хранилищах.</span><span class="sxs-lookup"><span data-stu-id="51cc4-147">Redis Cache can improve performance of systems that rely highly on data in back-end stores.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="51cc4-148">Вопросы масштабируемости</span><span class="sxs-lookup"><span data-stu-id="51cc4-148">Scalability considerations</span></span>

<span data-ttu-id="51cc4-149">Если вы не планируете использовать Spark или у вас меньшая рабочая нагрузка, где не нужно распределение, рекомендуем использовать [виртуальную машину для обработки и анализа данных][dsvm] (DSVM) вместо Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="51cc4-149">If you don't plan to use Spark, or you have a smaller workload where you don't need distribution, consider using [Data Science Virtual Machine][dsvm] (DSVM) instead of Azure Databricks.</span></span> <span data-ttu-id="51cc4-150">DSVM – это виртуальная машина Azure с платформами для глубинного обучения и средствами для машинного обучения, обработки и анализа данных.</span><span class="sxs-lookup"><span data-stu-id="51cc4-150">DSVM is an Azure virtual machine with deep learning frameworks and tools for machine learning and data science.</span></span> <span data-ttu-id="51cc4-151">Как и в случае с Azure Databricks, любую модель, созданную в DSVM, можно использовать в качестве службы в AKS с помощью Машинного обучения Azure.</span><span class="sxs-lookup"><span data-stu-id="51cc4-151">As with Azure Databricks, any model you create in a DSVM can be operationalized as a service on AKS via Azure Machine Learning.</span></span>

<span data-ttu-id="51cc4-152">Во время обучения выделите больший кластер Spark фиксированного размера в Azure Databricks или настройте [автомасштабирование][autoscaling].</span><span class="sxs-lookup"><span data-stu-id="51cc4-152">During training, provision a larger fixed-size Spark cluster in Azure Databricks or configure [autoscaling][autoscaling].</span></span> <span data-ttu-id="51cc4-153">Если автомасштабирование включено, Databricks отслеживает нагрузку на ваш кластер и при необходимости масштабирует его.</span><span class="sxs-lookup"><span data-stu-id="51cc4-153">When autoscaling is enabled, Databricks monitors the load on your cluster and scales up and downs when required.</span></span> <span data-ttu-id="51cc4-154">Подготовьте или масштабируйте более крупный кластер, если у вас большой объем данных и вы хотите сократить время, необходимое для подготовки данных или моделирования задач.</span><span class="sxs-lookup"><span data-stu-id="51cc4-154">Provision or scale out a larger cluster if you have a large data size and you want to reduce the amount of time it takes for data preparation or modeling tasks.</span></span>

<span data-ttu-id="51cc4-155">Масштабируйте кластер AKS в соответствии с вашими требованиями к производительности и пропускной способности.</span><span class="sxs-lookup"><span data-stu-id="51cc4-155">Scale the AKS cluster to meet your performance and throughput requirements.</span></span> <span data-ttu-id="51cc4-156">Будьте внимательны при увеличении количества [модулей][scale], чтобы полностью использовать кластер, и при масштабировании [узлов][nodes] кластера, чтобы соответствовать требованиям вашей службы.</span><span class="sxs-lookup"><span data-stu-id="51cc4-156">Take care to scale up the number of [pods][scale] to fully utilize the cluster, and to scale the [nodes][nodes] of the cluster to meet the demand of your service.</span></span> <span data-ttu-id="51cc4-157">Дополнительные сведения о том, как масштабировать кластер в соответствии с требованиями к производительности и пропускной способности службы рекомендации, см. в статье [Масштабирование кластеров службы контейнеров Azure][blog].</span><span class="sxs-lookup"><span data-stu-id="51cc4-157">For more information on how to scale your cluster to meet the performance and throughput requirements of your recommender service, see [Scaling Azure Container Service Clusters][blog].</span></span>

<span data-ttu-id="51cc4-158">Чтобы управлять производительностью Azure Cosmos DB, оцените необходимое количество операций чтения в секунду и укажите количество [EЗ в секунду][ru] (пропускная способность).</span><span class="sxs-lookup"><span data-stu-id="51cc4-158">To manage Azure Cosmos DB performance, estimate the number of reads required per second, and provision the number of [RUs per second][ru] (throughput) needed.</span></span> <span data-ttu-id="51cc4-159">Следуйте рекомендациям для [секционирования и горизонтального масштабирования][partition-data].</span><span class="sxs-lookup"><span data-stu-id="51cc4-159">Use best practices for [partitioning and horizontal scaling][partition-data].</span></span>

## <a name="cost-considerations"></a><span data-ttu-id="51cc4-160">Рекомендации по стоимости</span><span class="sxs-lookup"><span data-stu-id="51cc4-160">Cost considerations</span></span>

<span data-ttu-id="51cc4-161">Основными аргументами стоимости в этом сценарии являются:</span><span class="sxs-lookup"><span data-stu-id="51cc4-161">The main drivers of cost in this scenario are:</span></span>

- <span data-ttu-id="51cc4-162">размер кластера Azure Databricks, необходимый для обучения;</span><span class="sxs-lookup"><span data-stu-id="51cc4-162">The Azure Databricks cluster size required for training.</span></span>
- <span data-ttu-id="51cc4-163">размер кластера AKS, необходимый для выполнения требований к производительности;</span><span class="sxs-lookup"><span data-stu-id="51cc4-163">The AKS cluster size required to meet your performance requirements.</span></span>
- <span data-ttu-id="51cc4-164">ЕЗ Azure Cosmos DB предоставлены в соответствии с вашими требованиями к производительности.</span><span class="sxs-lookup"><span data-stu-id="51cc4-164">Azure Cosmos DB RUs provisioned to meet your performance requirements.</span></span>

<span data-ttu-id="51cc4-165">Управляйте стоимостью Azure Databricks, реже проходя переподготовку и отключая кластер Spark, когда он не используется.</span><span class="sxs-lookup"><span data-stu-id="51cc4-165">Manage the Azure Databricks costs by retraining less frequently and turning off the Spark cluster when not in use.</span></span> <span data-ttu-id="51cc4-166">Стоимость AKS и Azure Cosmos DB привязана к пропускной способности и производительности, которые требуются вашему сайту. Она будет увеличиваться и уменьшаться в зависимости от объема трафика на вашем сайте.</span><span class="sxs-lookup"><span data-stu-id="51cc4-166">The AKS and Azure Cosmos DB costs are tied to the throughput and performance required by your site and will scale up and down depending on the volume of traffic to your site.</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="51cc4-167">Развертывание решения</span><span class="sxs-lookup"><span data-stu-id="51cc4-167">Deploy the solution</span></span>

<span data-ttu-id="51cc4-168">Чтобы развернуть эту архитектуру, сначала создайте среду Azure Databricks для подготовки данных и обучения модели рекомендации.</span><span class="sxs-lookup"><span data-stu-id="51cc4-168">To deploy this architecture, first create an Azure Databricks environment to prepare data and train a recommender model:</span></span>

1. <span data-ttu-id="51cc4-169">Создайте [рабочую область Azure Databricks][workspace].</span><span class="sxs-lookup"><span data-stu-id="51cc4-169">Create an [Azure Databricks workspace][workspace].</span></span>

2. <span data-ttu-id="51cc4-170">Создайте новый кластер Spark в Azure Databricks.</span><span class="sxs-lookup"><span data-stu-id="51cc4-170">Create a new cluster in Azure Databricks.</span></span> <span data-ttu-id="51cc4-171">Требуются следующие значения конфигурации.</span><span class="sxs-lookup"><span data-stu-id="51cc4-171">The following configuration is required:</span></span>

    - <span data-ttu-id="51cc4-172">Режим кластера: Стандартная</span><span class="sxs-lookup"><span data-stu-id="51cc4-172">Cluster mode: Standard</span></span>
    - <span data-ttu-id="51cc4-173">Версия среды выполнения Databricks: 4.1 (содержит Apache Spark 2.3.0, Scala 2.11)</span><span class="sxs-lookup"><span data-stu-id="51cc4-173">Databricks Runtime Version: 4.1 (includes Apache Spark 2.3.0, Scala 2.11)</span></span>
    - <span data-ttu-id="51cc4-174">Версия Python: 3</span><span class="sxs-lookup"><span data-stu-id="51cc4-174">Python Version: 3</span></span>
    - <span data-ttu-id="51cc4-175">Тип драйвера: Стандартный \_DS3\_версии 2</span><span class="sxs-lookup"><span data-stu-id="51cc4-175">Driver Type: Standard\_DS3\_v2</span></span>
    - <span data-ttu-id="51cc4-176">Тип рабочего процесса: Стандартный\_DS3\_версии 2 (min и max, при необходимости)</span><span class="sxs-lookup"><span data-stu-id="51cc4-176">Worker Type: Standard\_DS3\_v2 (min and max as required)</span></span>
    - <span data-ttu-id="51cc4-177">Автоматическое завершение работы: при необходимости</span><span class="sxs-lookup"><span data-stu-id="51cc4-177">Auto Termination: (as required)</span></span>
    - <span data-ttu-id="51cc4-178">Конфигурация Spark: при необходимости</span><span class="sxs-lookup"><span data-stu-id="51cc4-178">Spark Config: (as required)</span></span>
    - <span data-ttu-id="51cc4-179">Переменные среды: при необходимости</span><span class="sxs-lookup"><span data-stu-id="51cc4-179">Environment Variables: (as required)</span></span>

3. <span data-ttu-id="51cc4-180">Клонируйте репозиторий[Microsoft Recommenders][github] на локальном компьютере.</span><span class="sxs-lookup"><span data-stu-id="51cc4-180">Clone the [Microsoft Recommenders][github] repository on your local computer.</span></span>

4. <span data-ttu-id="51cc4-181">Заархивируйте содержимое папки "Рекомендации".</span><span class="sxs-lookup"><span data-stu-id="51cc4-181">Zip the content inside the Recommenders folder:</span></span>

    ```console
    cd Recommenders
    zip -r Recommenders.zip
    ```

5. <span data-ttu-id="51cc4-182">Подключите библиотеку "Рекомендации" к своему кластеру следующим образом.</span><span class="sxs-lookup"><span data-stu-id="51cc4-182">Attach the Recommenders library to your cluster as follows:</span></span>

    1. <span data-ttu-id="51cc4-183">В следующем меню используйте параметр для импорта библиотеки ("Чтобы импортировать библиотеку, например, jar или egg, щелкните здесь") и **щелкните здесь**.</span><span class="sxs-lookup"><span data-stu-id="51cc4-183">In the next menu, use the option to import a library ("To import a library, such as a jar or egg, click here") and press **click here**.</span></span>

    2. <span data-ttu-id="51cc4-184">В первом раскрывающемся меню выберите параметр **Передать пакет Python PyPI или Python egg**.</span><span class="sxs-lookup"><span data-stu-id="51cc4-184">At the first drop-down menu, select the **Upload Python egg or PyPI** option.</span></span>

    3. <span data-ttu-id="51cc4-185">Выберите команду **Перетащить сюда библиотеку egg для передачи**, а затем выберите недавно созданный файл Recommenders.zip.</span><span class="sxs-lookup"><span data-stu-id="51cc4-185">Select **Drop library egg here to upload** and select the Recommenders.zip file you just created.</span></span>

    4. <span data-ttu-id="51cc4-186">Выберите **Создать библиотеку**, чтобы передать ZIP-файл и сделать его доступным в рабочей области.</span><span class="sxs-lookup"><span data-stu-id="51cc4-186">Select **Create library** to upload the .zip file and make it available in your workspace.</span></span>

    5. <span data-ttu-id="51cc4-187">В следующем меню подключите библиотеку к своему кластеру.</span><span class="sxs-lookup"><span data-stu-id="51cc4-187">In the next menu, attach the library to your cluster.</span></span>

6. <span data-ttu-id="51cc4-188">В своей рабочей области импортируйте [пример ALS Movie Operationalization][als-example].</span><span class="sxs-lookup"><span data-stu-id="51cc4-188">In your workspace, import the [ALS Movie Operationalization example][als-example].</span></span>

7. <span data-ttu-id="51cc4-189">Запустите записную книжку ALS Movie Operationalization, чтобы создать ресурсы, необходимые для создания API рекомендаций, который предоставляет 10 рекомендаций по фильмам для данного пользователя.</span><span class="sxs-lookup"><span data-stu-id="51cc4-189">Run the ALS Movie Operationalization notebook to create the resources required to create a recommendation API that provides the top-10 movie recommendations for a given user.</span></span>

## <a name="related-architectures"></a><span data-ttu-id="51cc4-190">Связанные архитектуры</span><span class="sxs-lookup"><span data-stu-id="51cc4-190">Related architectures</span></span>

<span data-ttu-id="51cc4-191">Мы также создали эталонную архитектуру, которая использует Spark и Azure Databricks для выполнения запланированных [процессов оценки пакетов][batch-scoring].</span><span class="sxs-lookup"><span data-stu-id="51cc4-191">We have also built a reference architecture that uses Spark and Azure Databricks to execute scheduled [batch-scoring processes][batch-scoring].</span></span> <span data-ttu-id="51cc4-192">Посмотрите эту эталонную архитектуру, чтобы понять рекомендуемый подход для регулярного создания новых рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="51cc4-192">See that reference architecture to understand a recommended approach for generating new recommendations routinely.</span></span>

<!-- links -->
[aci]: /azure/container-instances/container-instances-overview
[aad]: /azure/active-directory-b2c/active-directory-b2c-overview
[aks]: /azure/aks/intro-kubernetes
[als]: https://spark.apache.org/docs/latest/ml-collaborative-filtering.html
[als-example]: https://github.com/Microsoft/Recommenders/blob/master/notebooks/04_operationalize/als_movie_o16n.ipynb
[autoscaling]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html
[autoscale]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html#autoscaling
[availability]: /azure/architecture/checklist/availability
[batch-scoring]: /azure/architecture/reference-architectures/ai/batch-scoring-databricks
[blob]: /azure/storage/blobs/storage-blobs-introduction
[blog]: https://blogs.technet.microsoft.com/machinelearning/2018/03/20/scaling-azure-container-service-cluster/
[clusters]: https://docs.azuredatabricks.net/user-guide/clusters/configure.html
[cosmosdb]: /azure/cosmos-db/introduction
[data-source]: https://docs.azuredatabricks.net/spark/latest/data-sources/index.html
[databricks]: /azure/azure-databricks/what-is-azure-databricks
[dsvm]: /azure/machine-learning/data-science-virtual-machine/overview
[dsvm-ubuntu]: /azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro
[eval-guide]: https://github.com/Microsoft/Recommenders/blob/master/notebooks/03_evaluate/evaluation.ipynb
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[github]: https://github.com/Microsoft/Recommenders
[guide]: https://github.com/Microsoft/Recommenders/blob/master/notebooks/01_prepare_data/data_split.ipynb
[latency]: https://github.com/jessebenson/azure-performance
[mls]: /azure/machine-learning/service/
[n-tier]: /azure/architecture/reference-architectures/n-tier/n-tier-cassandra
[ndcg]: https://en.wikipedia.org/wiki/Discounted_cumulative_gain
[nodes]: /azure/aks/scale-cluster
[notebook]: https://github.com/Microsoft/Recommenders/notebooks/00_quick_start/als_pyspark_movielens.ipynb
[partition-data]: /azure/cosmos-db/partition-data
[redis]: /azure/redis-cache/cache-overview
[regions]: https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines&regions=all
[resiliency]: /azure/architecture/resiliency/
[ru]: /azure/cosmos-db/request-units
[sec-docs]: /azure/security/
[setup]: https://github.com/Microsoft/Recommenders/blob/master/SETUP.md%60
[scale]: /azure/aks/tutorial-kubernetes-scale
[sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_8/
[vm-size]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
[workspace]: https://docs.azuredatabricks.net/getting-started/index.html
