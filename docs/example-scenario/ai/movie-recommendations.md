---
title: Рекомендации фильмов в Azure
description: Использование машинного обучения для автоматизированного предоставления рекомендаций по фильмам, товарам и т. п. путем обучения модели в Azure с помощью машинного обучения и виртуальных машин для обработки и анализа данных (DSVM) Azure.
author: njray
ms.date: 01/09/2019
ms.custom: azcat-ai, AI
social_image_url: /azure/architecture/example-scenario/ai/media/architecture-movie-recommender.png
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.openlocfilehash: be7959c1201e3a2aaf92c192d73a0ca47a990934
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58249649"
---
# <a name="movie-recommendations-on-azure"></a><span data-ttu-id="fe5f9-103">Рекомендации фильмов в Azure</span><span class="sxs-lookup"><span data-stu-id="fe5f9-103">Movie recommendations on Azure</span></span>

<span data-ttu-id="fe5f9-104">Здесь рассматривается пример сценария, в котором предприятие применяет машинное обучение для автоматизации рекомендаций товаров своим клиентам.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-104">This example scenario shows how a business can use machine learning to automate product recommendations for their customers.</span></span> <span data-ttu-id="fe5f9-105">Виртуальная машина для обработки и анализа данных (DSVM) Azure используется для обучения в Azure модели, которая рекомендует пользователям фильмы на основе оценок, присвоенных этим фильмам.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-105">An Azure Data Science Virtual Machine (DSVM) is used to train a model on Azure that recommends movies to users based on ratings that have been given to movies.</span></span>

<span data-ttu-id="fe5f9-106">Такие рекомендации будут полезны в разных отраслях — от розничной торговли до новостей и СМИ.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-106">Recommendations can be useful in various industries from retail to news to media.</span></span> <span data-ttu-id="fe5f9-107">Например, вы можете рекомендовать товары в виртуальном магазине, новостные статьи или записи в блогах, а также музыкальные композиции.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-107">Potential applications include providing  product recommendations in a virtual store, providing news or post recommendations, or providing music recommendations.</span></span> <span data-ttu-id="fe5f9-108">Раньше предприятиям приходилось нанимать и обучать консультантов для того, чтобы предоставлять клиентам персонализированные рекомендации.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-108">Traditionally, businesses had to hire and train assistants to make personalized recommendations to customers.</span></span> <span data-ttu-id="fe5f9-109">Но теперь мы можем осуществлять такие действия в широком масштабе, используя Azure для обучения моделей, учитывающих предпочтения клиентов.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-109">Today, we can  provide customized recommendations at scale by utilizing Azure to train models to understand customer preferences.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="fe5f9-110">Варианты соответствующего использования</span><span class="sxs-lookup"><span data-stu-id="fe5f9-110">Relevant use cases</span></span>

<span data-ttu-id="fe5f9-111">Рассмотрите этот сценарий для следующих вариантов использования:</span><span class="sxs-lookup"><span data-stu-id="fe5f9-111">Consider this scenario for the following use cases:</span></span>

* <span data-ttu-id="fe5f9-112">рекомендации фильмов на веб-сайте;</span><span class="sxs-lookup"><span data-stu-id="fe5f9-112">Movie recommendations on a website.</span></span>
* <span data-ttu-id="fe5f9-113">рекомендации по потребительским товарам в мобильном приложении;</span><span class="sxs-lookup"><span data-stu-id="fe5f9-113">Consumer product recommendations in a mobile app.</span></span>
* <span data-ttu-id="fe5f9-114">рекомендации новостей в вещательных сетях.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-114">News recommendations on streaming media.</span></span>

## <a name="architecture"></a><span data-ttu-id="fe5f9-115">Архитектура</span><span class="sxs-lookup"><span data-stu-id="fe5f9-115">Architecture</span></span>

![Архитектура модели машинного обучения для создания рекомендаций к фильмам][architecture]

<span data-ttu-id="fe5f9-117">Этот сценарий посвящен обучению и оценке модели машинного обучения с помощью алгоритма Spark [чередующихся наименьших квадратов][als] (ALS) по набору данных с оценками фильмов.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-117">This scenario covers the training and evaluating of the machine learning model using the Spark [alternating least squares][als] (ALS) algorithm on a dataset of movie ratings.</span></span> <span data-ttu-id="fe5f9-118">В этом сценарии применяются следующие действия.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-118">The steps for this scenario are as following:</span></span>

1. <span data-ttu-id="fe5f9-119">Интерфейсный веб-сайт или служба приложений собирает исторические данные о взаимодействиях пользователей с фильмами и размещает их в таблице с кортежами пользователя, элемента и числовой оценки.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-119">The front-end website or app service collects historical data of user-movie interactions, which are represented in a table of user, item, and numerical rating tuples.</span></span>

2. <span data-ttu-id="fe5f9-120">Собранные исторические данные сохраняются в хранилище BLOB-объектов.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-120">The collected historical data is stored in a blob storage.</span></span>

3. <span data-ttu-id="fe5f9-121">DSVM часто используется для экспериментов с моделями рекомендаций на основе алгоритма Spark ALS и публикации готовых моделей.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-121">A DSVM is often used to experiment with or productize a Spark ALS recommender model.</span></span> <span data-ttu-id="fe5f9-122">Модель ALS обучается с использованием учебного набора данных, который извлекается из общего набора данных путем применения наиболее подходящей стратегии разбиения данных.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-122">The ALS model is trained using a training dataset, which is produced from the overall dataset by applying the appropriate data splitting strategy.</span></span> <span data-ttu-id="fe5f9-123">Например, набор данных можно разделить на несколько наборов случайным образом, в хронологическом порядке или по определенному условию, в зависимости от конкретных бизнес-требований.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-123">For example, the dataset can be split into sets randomly, chronologically, or stratified, depending on the business requirement.</span></span> <span data-ttu-id="fe5f9-124">Как и для других задач машинного обучения, система рекомендаций проверяется по метрикам оценки (например, precision\@*k*, recall\@*k*, [MAP][map], [nDCG\@k][ndcg]).</span><span class="sxs-lookup"><span data-stu-id="fe5f9-124">Similar to other machine learning tasks, a recommender is validated by using evaluation metrics (for example, precision\@*k*, recall\@*k*, [MAP][map], [nDCG\@k][ndcg]).</span></span>

4. <span data-ttu-id="fe5f9-125">Служба машинного обучения Azure берет на себя координирование экспериментов, в том числе перебор гиперпараметров и управление моделями.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-125">Azure Machine Learning service is used for coordinating the experimentation, such as hyperparameter sweeping and model management.</span></span>

5. <span data-ttu-id="fe5f9-126">Обученная модель сохраняется в Azure Cosmos DB. Ее можно применить для получения *k* лучших фильмов, которые можно рекомендовать конкретному пользователю.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-126">A trained model is preserved on Azure Cosmos DB, which can then be applied for recommending the top *k* movies for a given user.</span></span>

6. <span data-ttu-id="fe5f9-127">Затем эта модель развертывается на сайте или в службе приложений с использованием экземпляров контейнеров Azure или службы Azure Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-127">The model is then deployed onto a web or app service by using Azure Container Instances or Azure Kubernetes Service.</span></span>

<span data-ttu-id="fe5f9-128">Подробное руководство по разработке и масштабированию службы рекомендаций вы найдете в статье [Создание API рекомендаций в режиме реального времени в Azure][ref-arch].</span><span class="sxs-lookup"><span data-stu-id="fe5f9-128">For an in-depth guide to building and scaling a recommender service, see [Build a real-time recommendation API on Azure][ref-arch].</span></span>

### <a name="components"></a><span data-ttu-id="fe5f9-129">Компоненты</span><span class="sxs-lookup"><span data-stu-id="fe5f9-129">Components</span></span>

* <span data-ttu-id="fe5f9-130">[Виртуальная машина для обработки и анализа данных][dsvm] (DSVM) — это виртуальная машина Azure с платформами для глубокого обучения и средствами для машинного обучения, а также обработки и анализа данных.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-130">[Data Science Virtual Machine][dsvm] (DSVM) is an Azure virtual machine with deep learning frameworks and tools for machine learning and data science.</span></span> <span data-ttu-id="fe5f9-131">DSVM комплектуется изолированной средой Spark, в которой можно выполнять ALS.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-131">The DSVM has a standalone Spark environment that can be used to run ALS.</span></span>

* <span data-ttu-id="fe5f9-132">В [хранилище BLOB-объектов][blob] хранится набор данных для рекомендаций фильмов.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-132">[Azure Blob storage][blob] stores the dataset for movie recommendations.</span></span>

* <span data-ttu-id="fe5f9-133">[Служба машинного обучения Azure][mls] используется для ускорения разработки, управления и развертывания моделей машинного обучения.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-133">[Azure Machine Learning service][mls] is used to accelerate the building, managing, and deploying of machine learning models.</span></span>

* <span data-ttu-id="fe5f9-134">[Azure Cosmos DB][cosmosdb] предоставляет многомодельное глобально распределенное хранилище на основе баз данных.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-134">[Azure Cosmos DB][cosmosdb] enables globally distributed and multi-model database storage.</span></span>

* <span data-ttu-id="fe5f9-135">[Экземпляры контейнеров Azure][aci] применяются для развертывания обученных моделей в веб-службах или службах приложений, в том числе с применением [службы Azure Kubernetes][aks].</span><span class="sxs-lookup"><span data-stu-id="fe5f9-135">[Azure Container Instances][aci] is used to deploy the trained models to web or app services, optionally using [Azure Kubernetes Service][aks].</span></span>

### <a name="alternatives"></a><span data-ttu-id="fe5f9-136">Альтернативные варианты</span><span class="sxs-lookup"><span data-stu-id="fe5f9-136">Alternatives</span></span>

<span data-ttu-id="fe5f9-137">[Azure Databricks][databricks] — это управляемый кластер Spark, в котором выполняются обучение и оценка моделей.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-137">[Azure Databricks][databricks] is a managed Spark cluster where model training and evaluating is performed.</span></span> <span data-ttu-id="fe5f9-138">Вы можете настроить управляемую среду Spark всего за несколько минут и [автоматически масштабировать][autoscale] ее, сокращая затраты ресурсов и денежных средств, связанные с ручным масштабированием кластеров.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-138">You can set up a managed Spark environment in minutes, and [autoscale][autoscale] up and down to help reduce the resources and costs associated with scaling clusters manually.</span></span> <span data-ttu-id="fe5f9-139">Еще один способ экономить ресурсы — настроить автоматическое завершение работы неактивных [кластеров][clusters].</span><span class="sxs-lookup"><span data-stu-id="fe5f9-139">Another resource-saving option is to configure inactive [clusters][clusters] to terminate automatically.</span></span>

## <a name="considerations"></a><span data-ttu-id="fe5f9-140">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="fe5f9-140">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="fe5f9-141">Доступность</span><span class="sxs-lookup"><span data-stu-id="fe5f9-141">Availability</span></span>

<span data-ttu-id="fe5f9-142">Приложения на основе машинного обучения содержат два компонента ресурсов: ресурсы для обучения и ресурсы для обслуживания.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-142">Machine-learning-built apps are split into two resource components: resources for training, and resources for serving.</span></span> <span data-ttu-id="fe5f9-143">Ресурсам для обучения обычно не требуется высокий уровень доступности, так как запросы из производственной среды не обращаются к ним напрямую.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-143">Resources required for training generally do not need  high availability, as live production requests do not directly hit these resources.</span></span> <span data-ttu-id="fe5f9-144">Ресурсам для обслуживания важен высокий уровень доступности, что позволяет быстро обслуживать запросы клиентов.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-144">Resources required for serving need to have high availability to serve customer requests.</span></span>

<span data-ttu-id="fe5f9-145">DSVM для обучения доступны в [нескольких регионах][regions] по всему миру, и для них предоставляются гарантии [соглашения об уровне обслуживания][sla] для виртуальных машин.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-145">For training, the DSVM is available in [multiple regions][regions] around the globe and meets the [service level agreement][sla] (SLA) for virtual machines.</span></span> <span data-ttu-id="fe5f9-146">Для целей обслуживания служба Azure Kubernetes предлагает инфраструктуру [с высоким уровнем доступности][ha].</span><span class="sxs-lookup"><span data-stu-id="fe5f9-146">For serving, Azure Kubernetes Service provides a [highly available][ha] infrastructure.</span></span> <span data-ttu-id="fe5f9-147">К узлам агентов также применяется [соглашение об уровне обслуживания][sla-aks] для виртуальных машин.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-147">Agent nodes also follow the [SLA][sla-aks] for virtual machines.</span></span>

### <a name="scalability"></a><span data-ttu-id="fe5f9-148">Масштабируемость</span><span class="sxs-lookup"><span data-stu-id="fe5f9-148">Scalability</span></span>

<span data-ttu-id="fe5f9-149">Если вы используете данные большого объема, попробуйте масштабировать DSVM для сокращения времени обучения.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-149">If you have a large data size, you can scale your DSVM to shorten training time.</span></span> <span data-ttu-id="fe5f9-150">Для масштабирования виртуальной машины можно [увеличивать и уменьшать][vm-size] ее размер.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-150">You can scale a VM up or down by changing the [VM size][vm-size].</span></span> <span data-ttu-id="fe5f9-151">Выберите объем памяти, достаточный для размещения всего набора данных, и большое количество виртуальных ЦП, чтобы уменьшить количество времени на обучение.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-151">Choose a memory size large enough to fit your dataset in-memory and a higher vCPU count in order to decrease the amount of time that training takes.</span></span>

### <a name="security"></a><span data-ttu-id="fe5f9-152">Безопасность</span><span class="sxs-lookup"><span data-stu-id="fe5f9-152">Security</span></span>

<span data-ttu-id="fe5f9-153">В этом сценарии можно применить Azure Active Directory для аутентификации [доступа пользователей к DSVM][dsvm-id], где хранятся код приложения, модели и данные (в памяти).</span><span class="sxs-lookup"><span data-stu-id="fe5f9-153">This scenario can use Azure Active Directory to authenticate users for [access to the DSVM][dsvm-id], which contains your code, models, and (in-memory) data.</span></span> <span data-ttu-id="fe5f9-154">Данные сохраняются в службе хранилища Azure, а затем передаются на DSVM, где они автоматически шифруются с помощью [шифрования службы хранилища][storage-security].</span><span class="sxs-lookup"><span data-stu-id="fe5f9-154">Data is stored in Azure Storage prior to being loaded on a DSVM, where it is automatically encrypted using [Storage Service Encryption][storage-security].</span></span> <span data-ttu-id="fe5f9-155">Управлять разрешениями можно с помощью аутентификации Azure Active Directory или управления доступом на основе ролей.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-155">Permissions can be managed via Azure Active Directory authentication or role-based access control.</span></span>

## <a name="deploy-this-scenario"></a><span data-ttu-id="fe5f9-156">Развертывание этого сценария</span><span class="sxs-lookup"><span data-stu-id="fe5f9-156">Deploy this scenario</span></span>

<span data-ttu-id="fe5f9-157">**Предварительные требования**: Необходимо иметь учетную запись Azure.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-157">**Prerequisites**: You must have an existing Azure account.</span></span> <span data-ttu-id="fe5f9-158">Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись][free], прежде чем начинать работу.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-158">If you don't have an Azure subscription, create a [free account][free] before you begin.</span></span>

<span data-ttu-id="fe5f9-159">Весь код для этого сценария размещен в [репозитории Microsoft Recommenders][github].</span><span class="sxs-lookup"><span data-stu-id="fe5f9-159">All the code for this scenario is available in the [Microsoft Recommenders repository][github].</span></span>

<span data-ttu-id="fe5f9-160">Выполните следующие действия, чтобы запустить [записную книжку для начала работы с ALS][notebook]:</span><span class="sxs-lookup"><span data-stu-id="fe5f9-160">Follow these steps to run the [ALS quickstart notebook][notebook]:</span></span>

1. <span data-ttu-id="fe5f9-161">[Создайте виртуальную машину для обработки и анализа данных][dsvm-ubuntu] на портале Azure.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-161">[Create a DSVM][dsvm-ubuntu] from the Azure portal.</span></span>

2. <span data-ttu-id="fe5f9-162">Клонируйте репозиторий в папку Notebooks.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-162">Clone the repo in the Notebooks folder:</span></span>

    ```shell
    cd notebooks
    git clone https://github.com/Microsoft/Recommenders
    ```

3. <span data-ttu-id="fe5f9-163">Установите зависимости conda, выполнив действия, описанные в файле [SETUP.md][setup].</span><span class="sxs-lookup"><span data-stu-id="fe5f9-163">Install the conda dependencies following the steps described in the [SETUP.md][setup] file.</span></span>

4. <span data-ttu-id="fe5f9-164">Откройте в браузере виртуальную машину jupyterlab и перейдите к `notebooks/00_quick_start/als_pyspark_movielens.ipynb`.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-164">In a browser, go to your jupyterlab VM and navigate to `notebooks/00_quick_start/als_pyspark_movielens.ipynb`.</span></span>

5. <span data-ttu-id="fe5f9-165">Выполните записную книжку.</span><span class="sxs-lookup"><span data-stu-id="fe5f9-165">Execute the notebook.</span></span>

## <a name="related-resources"></a><span data-ttu-id="fe5f9-166">Связанные ресурсы</span><span class="sxs-lookup"><span data-stu-id="fe5f9-166">Related resources</span></span>

<span data-ttu-id="fe5f9-167">Подробное руководство по разработке и масштабированию службы рекомендаций вы найдете в статье [Создание API рекомендаций в режиме реального времени в Azure][ref-arch].</span><span class="sxs-lookup"><span data-stu-id="fe5f9-167">For an in-depth guide to building and scaling a recommender service, see [Build a real-time recommendation API on Azure][ref-arch].</span></span> <span data-ttu-id="fe5f9-168">Руководства и примеры систем рекомендаций можно найти в [репозитории Microsoft Recommenders][github].</span><span class="sxs-lookup"><span data-stu-id="fe5f9-168">For tutorials and examples of recommendation systems, see [Microsoft Recommenders repository][github].</span></span>

[architecture]: ./media/architecture-movie-recommender.png
[aci]: /azure/container-instances/container-instances-overview
[aad]: /azure/active-directory-b2c/active-directory-b2c-overview
[aks]: /azure/aks/intro-kubernetes
[als]: https://spark.apache.org/docs/latest/ml-collaborative-filtering.html
[autoscale]: https://docs.azuredatabricks.net/user-guide/clusters/sizing.html#autoscaling
[blob]: /azure/storage/blobs/storage-blobs-introduction
[clusters]: https://docs.azuredatabricks.net/user-guide/clusters/configure.html
[cosmosdb]: /azure/cosmos-db/introduction
[databricks]: /azure/azure-databricks/what-is-azure-databricks
[dsvm]: /azure/machine-learning/data-science-virtual-machine/overview
[dsvm-id]: /azure/machine-learning/data-science-virtual-machine/dsvm-common-identity
[dsvm-ubuntu]: /azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[github]: https://github.com/Microsoft/Recommenders
[ha]: /azure/aks/container-service-quotas
[map]: https://en.wikipedia.org/wiki/Evaluation_measures_(information_retrieval)
[mls]: /azure/machine-learning/service/
[n-tier]: /azure/architecture/reference-architectures/n-tier/n-tier-cassandra
[ndcg]: https://en.wikipedia.org/wiki/Discounted_cumulative_gain
[notebook]: https://github.com/Microsoft/Recommenders/notebooks/00_quick_start/als_pyspark_movielens.ipynb
[ref-arch]: /azure/architecture/reference-architectures/ai/real-time-recommendation
[regions]: https://azure.microsoft.com/en-us/global-infrastructure/services/?products=virtual-machines&regions=all
[resiliency]: /azure/architecture/resiliency/
[sec-docs]: /azure/security/
[setup]: https://github.com/Microsoft/Recommenders/blob/master/SETUP.md%60
[sla]: https://azure.microsoft.com/en-us/support/legal/sla/virtual-machines/v1_8/
[sla-aks]: https://azure.microsoft.com/en-us/support/legal/sla/kubernetes-service/v1_0/
[storage-security]: /azure/storage/common/storage-service-encryption
[vm-size]: /azure/virtual-machines/virtual-machines-linux-change-vm-size
