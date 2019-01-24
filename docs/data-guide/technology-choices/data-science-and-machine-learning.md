---
title: Выбор технологии машинного обучения
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: a435286781281a43864561f98362675219bb1477
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486579"
---
# <a name="choosing-a-machine-learning-technology-in-azure"></a><span data-ttu-id="d76a7-102">Выбор технологии машинного обучения в Azure</span><span class="sxs-lookup"><span data-stu-id="d76a7-102">Choosing a machine learning technology in Azure</span></span>

<span data-ttu-id="d76a7-103">Обработка и анализ данных и машинное обучение — это рабочая нагрузка, которую обычно выполняют специалисты по обработке и анализу данных.</span><span class="sxs-lookup"><span data-stu-id="d76a7-103">Data science and machine learning is a workload that is usually undertaken by data scientists.</span></span> <span data-ttu-id="d76a7-104">Для этого требуются конкретные инструменты, многие из которых специально разработаны для задач интерактивного исследования и моделирования данных.</span><span class="sxs-lookup"><span data-stu-id="d76a7-104">It requires specialist tools, many of which are designed specifically for the type of interactive data exploration and modeling tasks that a data scientist must perform.</span></span>

<span data-ttu-id="d76a7-105">Решения машинного обучения предусматривают итеративную работу и имеют две различные фазы:</span><span class="sxs-lookup"><span data-stu-id="d76a7-105">Machine learning solutions are built iteratively, and have two distinct phases:</span></span>

- <span data-ttu-id="d76a7-106">подготовка и моделирование данных;</span><span class="sxs-lookup"><span data-stu-id="d76a7-106">Data preparation and modeling.</span></span>
- <span data-ttu-id="d76a7-107">развертывание и использование прогнозных служб.</span><span class="sxs-lookup"><span data-stu-id="d76a7-107">Deployment and consumption of predictive services.</span></span>

## <a name="tools-and-services-for-data-preparation-and-modeling"></a><span data-ttu-id="d76a7-108">Средства и службы для подготовки данных и моделирования</span><span class="sxs-lookup"><span data-stu-id="d76a7-108">Tools and services for data preparation and modeling</span></span>

<span data-ttu-id="d76a7-109">Специалисты по анализу и обработке данных предпочитают работать с данными с использованием пользовательского кода, написанного на Python или R. Как правило, этот код запускается в интерактивном режиме, а специалисты используют его для запроса и изучения данных, создания визуализаций и формирования статистики с целью определения связей в них.</span><span class="sxs-lookup"><span data-stu-id="d76a7-109">Data scientists typically prefer to work with data using custom code written in Python or R. This code is generally run interactively, with the data scientists using it to query and explore the data, generating visualizations and statistics to help determine the relationships with it.</span></span> <span data-ttu-id="d76a7-110">Существует много интерактивных сред для R и Python.</span><span class="sxs-lookup"><span data-stu-id="d76a7-110">There are many interactive environments for R and Python that data scientists can use.</span></span> <span data-ttu-id="d76a7-111">Самая популярная — **записные книжки Jupyter**. Эта браузерная оболочка, которая позволяет специалистам создавать файлы *записной книжки*, содержащие код R или Python и отформатированный текст.</span><span class="sxs-lookup"><span data-stu-id="d76a7-111">A particular favorite is **Jupyter Notebooks** that provides a browser-based shell that enables data scientists to create *notebook* files that contain R or Python code and markdown text.</span></span> <span data-ttu-id="d76a7-112">Это эффективный способ совместной работы путем обмена кодом и результатами и фиксирования их в одном документе.</span><span class="sxs-lookup"><span data-stu-id="d76a7-112">This is an effective way to collaborate by sharing and documenting code and results in a single document.</span></span>

<span data-ttu-id="d76a7-113">Ниже приведены другие широко используемые инструменты.</span><span class="sxs-lookup"><span data-stu-id="d76a7-113">Other commonly used tools include:</span></span>

- <span data-ttu-id="d76a7-114">**Spyder**. Интерактивная среда разработки (IDE) для Python в составе дистрибутива Anaconda Python.</span><span class="sxs-lookup"><span data-stu-id="d76a7-114">**Spyder**: The interactive development environment (IDE) for Python provided with the Anaconda Python distribution.</span></span>
- <span data-ttu-id="d76a7-115">**R Studio**. Интерактивная среда разработки для языка R.</span><span class="sxs-lookup"><span data-stu-id="d76a7-115">**R Studio**: An IDE for the R programming language.</span></span>
- <span data-ttu-id="d76a7-116">**Visual Studio Code**. Упрощенная кроссплатформенная среда программирования, которая поддерживает Python, а также часто используемые платформы для машинного обучения и разработки ИИ.</span><span class="sxs-lookup"><span data-stu-id="d76a7-116">**Visual Studio Code**: A lightweight, cross-platform coding environment that supports Python as well as commonly used frameworks for machine learning and AI development.</span></span>

<span data-ttu-id="d76a7-117">В дополнение к этим средствам специалисты по обработке и анализу данных могут использовать службы Azure для упрощения управления кодом и моделью.</span><span class="sxs-lookup"><span data-stu-id="d76a7-117">In addition to these tools, data scientists can leverage Azure services to simplify code and model management.</span></span>

### <a name="azure-notebooks"></a><span data-ttu-id="d76a7-118">Azure Notebook</span><span class="sxs-lookup"><span data-stu-id="d76a7-118">Azure Notebooks</span></span>

<span data-ttu-id="d76a7-119">Azure Notebooks — это сетевая служба записных книжек Jupyter, которая позволяет специалистам по обработке и анализу данных создавать, запускать и совместно использовать записные книжки Jupyter в облачных библиотеках.</span><span class="sxs-lookup"><span data-stu-id="d76a7-119">Azure Notebooks is an online Jupyter Notebooks service that enables data scientists to create, run, and share Jupyter Notebooks in cloud-based libraries.</span></span>

<span data-ttu-id="d76a7-120">Основные преимущества:</span><span class="sxs-lookup"><span data-stu-id="d76a7-120">Key benefits:</span></span>

- <span data-ttu-id="d76a7-121">Бесплатная служба, для которой не требуется подписка Azure.</span><span class="sxs-lookup"><span data-stu-id="d76a7-121">Free service&mdash;no Azure subscription required.</span></span>
- <span data-ttu-id="d76a7-122">Нет необходимости устанавливать Jupyter и дополнительные дистрибутивы R или Python локально. Все работает через браузер.</span><span class="sxs-lookup"><span data-stu-id="d76a7-122">No need to install Jupyter and the supporting R or Python distributions locally&mdash;just use a browser.</span></span>
- <span data-ttu-id="d76a7-123">Управление собственными сетевыми библиотеками и получение доступа к ним с любого устройства.</span><span class="sxs-lookup"><span data-stu-id="d76a7-123">Manage your own online libraries and access them from any device.</span></span>
- <span data-ttu-id="d76a7-124">Предоставление общего доступа к записным книжкам участникам совместной работы.</span><span class="sxs-lookup"><span data-stu-id="d76a7-124">Share your notebooks with collaborators.</span></span>

<span data-ttu-id="d76a7-125">Рекомендации:</span><span class="sxs-lookup"><span data-stu-id="d76a7-125">Considerations:</span></span>

- <span data-ttu-id="d76a7-126">Вы не сможете получить доступ к записным книжкам в автономном режиме.</span><span class="sxs-lookup"><span data-stu-id="d76a7-126">You will be unable to access your notebooks when offline.</span></span>
- <span data-ttu-id="d76a7-127">Ограниченных возможностей обработки в бесплатной службе записных книжек может быть недостаточно для обучения больших или сложных моделей.</span><span class="sxs-lookup"><span data-stu-id="d76a7-127">Limited processing capabilities of the free notebook service may not be enough to train large or complex models.</span></span>

### <a name="data-science-virtual-machine"></a><span data-ttu-id="d76a7-128">Виртуальная машина для обработки и анализа данных</span><span class="sxs-lookup"><span data-stu-id="d76a7-128">Data science virtual machine</span></span>

<span data-ttu-id="d76a7-129">Виртуальная машина для обработки и анализа данных является образом виртуальной машины Azure, содержащим средства и платформы, которые часто используются специалистами, включая R, Python, записные книжки Jupyter, Visual Studio Code и библиотеки для моделирования машинного обучения, такие как Microsoft Cognitive Toolkit.</span><span class="sxs-lookup"><span data-stu-id="d76a7-129">The data science virtual machine is an Azure virtual machine image that includes the tools and frameworks commonly used by data scientists, including R, Python, Jupyter Notebooks, Visual Studio Code, and libraries for machine learning modeling such as the Microsoft Cognitive Toolkit.</span></span> <span data-ttu-id="d76a7-130">Установка этих средств может быть сложной и отнимать много времени. К тому же они могут содержать много взаимозависимостей, которые часто приводят к проблемам управления версиями.</span><span class="sxs-lookup"><span data-stu-id="d76a7-130">These tools can be complex and time consuming to install, and contain many interdependencies that often lead to version management issues.</span></span> <span data-ttu-id="d76a7-131">Наличие предустановленного образа может сократить время, которое специалисты по обработке и анализу данных тратят на устранение неполадок, и они могут сосредоточиться на задачах изучения и моделирования данных.</span><span class="sxs-lookup"><span data-stu-id="d76a7-131">Having a preinstalled image can reduce the time data scientists spend troubleshooting environment issues, allowing them to focus on the data exploration and modeling tasks they need to perform.</span></span>

<span data-ttu-id="d76a7-132">Основные преимущества:</span><span class="sxs-lookup"><span data-stu-id="d76a7-132">Key benefits:</span></span>

- <span data-ttu-id="d76a7-133">Сокращение времени на установку, управление и устранение неполадок в средствах и ​​платформах для обработки и анализа данных.</span><span class="sxs-lookup"><span data-stu-id="d76a7-133">Reduced time to install, manage, and troubleshoot data science tools and frameworks.</span></span>
- <span data-ttu-id="d76a7-134">Включены последние версии всех широко используемых средств и ​​платформ.</span><span class="sxs-lookup"><span data-stu-id="d76a7-134">The latest versions of all commonly used tools and frameworks are included.</span></span>
- <span data-ttu-id="d76a7-135">Варианты виртуальных машин включают в себя высоко масштабируемые образы с поддержкой GPU для интенсивного моделирования данных.</span><span class="sxs-lookup"><span data-stu-id="d76a7-135">Virtual machine options include highly scalable images with GPU capabilities for intensive data modeling.</span></span>

<span data-ttu-id="d76a7-136">Рекомендации:</span><span class="sxs-lookup"><span data-stu-id="d76a7-136">Considerations:</span></span>

- <span data-ttu-id="d76a7-137">К виртуальной машине невозможно получить доступ в автономном режиме.</span><span class="sxs-lookup"><span data-stu-id="d76a7-137">The virtual machine cannot be accessed when offline.</span></span>
- <span data-ttu-id="d76a7-138">За использование виртуальной машины взимается плата в Azure, поэтому следует следить за тем, чтобы она работала только при необходимости.</span><span class="sxs-lookup"><span data-stu-id="d76a7-138">Running a virtual machine incurs Azure charges, so you must be careful to have it running only when required.</span></span>

### <a name="azure-machine-learning"></a><span data-ttu-id="d76a7-139">Машинное обучение Azure</span><span class="sxs-lookup"><span data-stu-id="d76a7-139">Azure Machine Learning</span></span>

<span data-ttu-id="d76a7-140">Машинное обучение Azure является облачной службой для управления экспериментами и моделями машинного обучения.</span><span class="sxs-lookup"><span data-stu-id="d76a7-140">Azure Machine Learning is a cloud-based service for managing machine learning experiments and models.</span></span> <span data-ttu-id="d76a7-141">Она включает в себя службу экспериментов, которая отслеживает подготовку данных, моделирование скриптов обучения и управление историей всех выполнений, чтобы вы могли сравнивать производительность модели во всех итерациях.</span><span class="sxs-lookup"><span data-stu-id="d76a7-141">It includes an experimentation service that tracks data preparation and modeling training scripts, maintaining a history of all executions so you can compare model performance across iterations.</span></span> <span data-ttu-id="d76a7-142">Специалисты по обработке и анализу данных могут создавать скрипты в привычных средствах, таких как записные книжки Jupyter или Visual Studio Code, а затем выполнять развертывание в разные [вычислительные ресурсы](/azure/machine-learning/service/how-to-set-up-training-targets) в Azure.</span><span class="sxs-lookup"><span data-stu-id="d76a7-142">Data scientists can create scripts in their tool of choice, such as Jupyter Notebooks or Visual Studio Code, and then deploy to a variety of different [compute resources](/azure/machine-learning/service/how-to-set-up-training-targets) in Azure.</span></span>

<span data-ttu-id="d76a7-143">Модели можно развертывать как веб-службы в контейнере Docker, Spark в Azure HDInsight, Microsoft Machine Learning Server или SQL Server.</span><span class="sxs-lookup"><span data-stu-id="d76a7-143">Models can be deployed as a web service to a Docker container, Spark on Azure HDinsight, Microsoft Machine Learning Server, or SQL Server.</span></span> <span data-ttu-id="d76a7-144">Затем вы сможете отслеживать развертывания моделей и управлять ими в облаке, на граничных устройствах или на предприятии с помощью службы "Управление моделями Машинного обучения Azure".</span><span class="sxs-lookup"><span data-stu-id="d76a7-144">The Azure Machine Learning Model Management service then enables you to track and manage model deployments in the cloud, on edge devices, or across the enterprise.</span></span>

<span data-ttu-id="d76a7-145">Основные преимущества:</span><span class="sxs-lookup"><span data-stu-id="d76a7-145">Key benefits:</span></span>

- <span data-ttu-id="d76a7-146">Централизованное управление скриптами и журналом выполнения, что упрощает сравнение версий модели.</span><span class="sxs-lookup"><span data-stu-id="d76a7-146">Central management of scripts and run history, making it easy to compare model versions.</span></span>
- <span data-ttu-id="d76a7-147">Интерактивное преобразование данных в визуальном редакторе.</span><span class="sxs-lookup"><span data-stu-id="d76a7-147">Interactive data transformation through a visual editor.</span></span>
- <span data-ttu-id="d76a7-148">Простое развертывание моделей и управление ими в облаке или на граничных устройствах.</span><span class="sxs-lookup"><span data-stu-id="d76a7-148">Easy deployment and management of models to the cloud or edge devices.</span></span>

<span data-ttu-id="d76a7-149">Рекомендации:</span><span class="sxs-lookup"><span data-stu-id="d76a7-149">Considerations:</span></span>

- <span data-ttu-id="d76a7-150">Необходимо ознакомиться с моделью управления моделями.</span><span class="sxs-lookup"><span data-stu-id="d76a7-150">Requires some familiarity with the model management model.</span></span>

### <a name="azure-batch-ai"></a><span data-ttu-id="d76a7-151">Azure Batch AI</span><span class="sxs-lookup"><span data-stu-id="d76a7-151">Azure Batch AI</span></span>

<span data-ttu-id="d76a7-152">Служба Azure Batch AI позволяет запускать эксперименты машинного обучения в параллельном режиме и выполнять масштабное обучение модели в кластере виртуальных машин с GPU.</span><span class="sxs-lookup"><span data-stu-id="d76a7-152">Azure Batch AI enables you to run your machine learning experiments in parallel, and perform model training at scale across a cluster of virtual machines with GPUs.</span></span> <span data-ttu-id="d76a7-153">Вы также можете развернуть задания глубокого обучения в кластере GPU, используя такие платформы, как Cognitive Toolkit, Caffe, Chainer и TensorFlow.</span><span class="sxs-lookup"><span data-stu-id="d76a7-153">Batch AI training enables you to scale out deep learning jobs across clustered GPUs, using frameworks such as Cognitive Toolkit, Caffe, Chainer, and TensorFlow.</span></span>

<span data-ttu-id="d76a7-154">Служба "Управление моделями Машинного обучения Azure" позволяет извлечь модели из пакетного обучения искусственного интеллекта, а также выполнять для них развертывание, управление и мониторинг.</span><span class="sxs-lookup"><span data-stu-id="d76a7-154">Azure Machine Learning Model Management can be used to take models from Batch AI training to deploy, manage, and monitor them.</span></span>

### <a name="azure-machine-learning-studio"></a><span data-ttu-id="d76a7-155">Студия машинного обучения Azure</span><span class="sxs-lookup"><span data-stu-id="d76a7-155">Azure Machine Learning Studio</span></span>

<span data-ttu-id="d76a7-156">Студия машинного обучения представляет собой облачную среду визуальной разработки для создания экспериментов с данными, обучения моделей машинного обучения и их публикации в качестве веб-служб в Azure.</span><span class="sxs-lookup"><span data-stu-id="d76a7-156">Azure Machine Learning Studio is a cloud-based, visual development environment for creating data experiments, training machine learning models, and publishing them as web services in Azure.</span></span> <span data-ttu-id="d76a7-157">В ее визуальном интерфейсе перетаскивания специалисты по обработке и анализу данных и опытные пользователи могут быстро создавать решения машинного обучения. Она поддерживает пользовательскую логику R и Python, широкий набор принятых статистических алгоритмов и методов для задач моделирования машинного обучения и имеет встроенную поддержку записных книжек Jupyter.</span><span class="sxs-lookup"><span data-stu-id="d76a7-157">Its visual drag-and-drop interface lets data scientists and power users create machine learning solutions quickly, while supporting custom R and Python logic, a wide range of established statistical algorithms and techniques for machine learning modeling tasks, and built-in support for Jupyter Notebooks.</span></span>

<span data-ttu-id="d76a7-158">Основные преимущества:</span><span class="sxs-lookup"><span data-stu-id="d76a7-158">Key benefits:</span></span>

- <span data-ttu-id="d76a7-159">Создание моделей машинного обучения с минимальным использованием кода благодаря интерактивному визуальному интерфейсу.</span><span class="sxs-lookup"><span data-stu-id="d76a7-159">Interactive visual interface enables machine learning modeling with minimal code.</span></span>
- <span data-ttu-id="d76a7-160">Встроенные записные книжки Jupyter для исследования данных.</span><span class="sxs-lookup"><span data-stu-id="d76a7-160">Built-in Jupyter Notebooks for data exploration.</span></span>
- <span data-ttu-id="d76a7-161">Прямое развертывание обученных моделей как веб-служб Azure.</span><span class="sxs-lookup"><span data-stu-id="d76a7-161">Direct deployment of trained models as Azure web services.</span></span>

<span data-ttu-id="d76a7-162">Рекомендации:</span><span class="sxs-lookup"><span data-stu-id="d76a7-162">Considerations:</span></span>

- <span data-ttu-id="d76a7-163">Ограниченная масштабируемость.</span><span class="sxs-lookup"><span data-stu-id="d76a7-163">Limited scalability.</span></span> <span data-ttu-id="d76a7-164">Максимальный размер набора данных для обучения составляет 10 ГБ.</span><span class="sxs-lookup"><span data-stu-id="d76a7-164">The maximum size of a training dataset is 10 GB.</span></span>
- <span data-ttu-id="d76a7-165">Служба доступна только в сети.</span><span class="sxs-lookup"><span data-stu-id="d76a7-165">Online only.</span></span> <span data-ttu-id="d76a7-166">Отсутствует автономная среда разработки.</span><span class="sxs-lookup"><span data-stu-id="d76a7-166">No offline development environment.</span></span>

## <a name="tools-and-services-for-deploying-machine-learning-models"></a><span data-ttu-id="d76a7-167">Средства и службы для развертывания моделей машинного обучения</span><span class="sxs-lookup"><span data-stu-id="d76a7-167">Tools and services for deploying machine learning models</span></span>

<span data-ttu-id="d76a7-168">После создания модели машинного обучения специалисту по обработке и анализу данных, как правило, требуется развернуть ее и использовать в приложениях или в других потоках данных.</span><span class="sxs-lookup"><span data-stu-id="d76a7-168">After a data scientist has created a machine learning model, you will typically need to deploy it and consume it from applications or in other data flows.</span></span> <span data-ttu-id="d76a7-169">Существует несколько возможных целевых сред для развертывания моделей машинного обучения.</span><span class="sxs-lookup"><span data-stu-id="d76a7-169">There are a number of potential deployment targets for machine learning models.</span></span>

### <a name="spark-on-azure-hdinsight"></a><span data-ttu-id="d76a7-170">Spark в Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="d76a7-170">Spark on Azure HDInsight</span></span>

<span data-ttu-id="d76a7-171">Apache Spark включает в себя Spark MLlib, платформу и библиотеку для моделей машинного обучения.</span><span class="sxs-lookup"><span data-stu-id="d76a7-171">Apache Spark includes Spark MLlib, a framework and library for machine learning models.</span></span> <span data-ttu-id="d76a7-172">Библиотека машинного обучения Майкрософт для Spark (MMLSpark) также обеспечивает поддержку алгоритмов глубокого обучения для прогнозных моделей в Spark.</span><span class="sxs-lookup"><span data-stu-id="d76a7-172">The Microsoft Machine Learning library for Spark (MMLSpark) also provides deep learning algorithm support for predictive models in Spark.</span></span>

<span data-ttu-id="d76a7-173">Основные преимущества:</span><span class="sxs-lookup"><span data-stu-id="d76a7-173">Key benefits:</span></span>

- <span data-ttu-id="d76a7-174">Spark — это распределенная платформа, которая обеспечивает высокую масштабируемость процессов машинного обучения большого объема.</span><span class="sxs-lookup"><span data-stu-id="d76a7-174">Spark is a distributed platform that offers high scalability for high-volume machine learning processes.</span></span>
- <span data-ttu-id="d76a7-175">Вы можете развернуть модели непосредственно в Spark для HDInsight и управлять ими с помощью службы "Управление моделями Машинного обучения Azure".</span><span class="sxs-lookup"><span data-stu-id="d76a7-175">You can deploy models directly to Spark in HDinsight and manage them using the Azure Machine Learning Model Management service.</span></span>

<span data-ttu-id="d76a7-176">Рекомендации:</span><span class="sxs-lookup"><span data-stu-id="d76a7-176">Considerations:</span></span>

- <span data-ttu-id="d76a7-177">Spark работает в кластере HDInsight, за работу которого взимается плата.</span><span class="sxs-lookup"><span data-stu-id="d76a7-177">Spark runs in an HDinsght cluster that incurs charges the whole time it is running.</span></span> <span data-ttu-id="d76a7-178">Если служба машинного обучения будет использоваться только время от времени, это может привести к необоснованным затратам.</span><span class="sxs-lookup"><span data-stu-id="d76a7-178">If the machine learning service will only be used occasionally, this may result in unnecessary costs.</span></span>

### <a name="azure-databricks"></a><span data-ttu-id="d76a7-179">Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="d76a7-179">Azure Databricks</span></span>

<span data-ttu-id="d76a7-180">[Azure Databricks](/azure/azure-databricks/) — это высокопроизводительная платформа на основе Apache Spark.</span><span class="sxs-lookup"><span data-stu-id="d76a7-180">[Azure Databricks](/azure/azure-databricks/) is an Apache Spark-based analytics platform.</span></span> <span data-ttu-id="d76a7-181">Представляет собой модель "Spark как услуга".</span><span class="sxs-lookup"><span data-stu-id="d76a7-181">You can think of it as "Spark as a service."</span></span> <span data-ttu-id="d76a7-182">Это самый простой способ использования Spark на платформе Azure.</span><span class="sxs-lookup"><span data-stu-id="d76a7-182">It's the easiest way to use Spark on the Azure platform.</span></span> <span data-ttu-id="d76a7-183">Для машинного обучения можно использовать [MLFlow](https://www.mlflow.org/), [ML Databricks Runtime](https://docs.azuredatabricks.net/user-guide/clusters/mlruntime.html), Apache Spark MLlib и т. д.</span><span class="sxs-lookup"><span data-stu-id="d76a7-183">For machine learning, you can use [MLFlow](https://www.mlflow.org/), [Databricks Runtime ML](https://docs.azuredatabricks.net/user-guide/clusters/mlruntime.html), Apache Spark MLlib, and others.</span></span> <span data-ttu-id="d76a7-184">См. дополнительные сведения об [использовании Azure Databricks в Машинном обучении](https://docs.azuredatabricks.net/spark/latest/mllib/index.html).</span><span class="sxs-lookup"><span data-stu-id="d76a7-184">For more information, see [Azure Databricks: Machine Learning](https://docs.azuredatabricks.net/spark/latest/mllib/index.html).</span></span>

### <a name="web-service-in-a-container"></a><span data-ttu-id="d76a7-185">Веб-служба в контейнере</span><span class="sxs-lookup"><span data-stu-id="d76a7-185">Web service in a container</span></span>

<span data-ttu-id="d76a7-186">Модель машинного обучения можно развернуть как веб-службу Python в контейнере Docker.</span><span class="sxs-lookup"><span data-stu-id="d76a7-186">You can deploy a machine learning model as a Python web service in a Docker container.</span></span> <span data-ttu-id="d76a7-187">Вы можете развернуть модель в Azure или на граничном устройстве, где она может использоваться локально с данными, с которыми она работает.</span><span class="sxs-lookup"><span data-stu-id="d76a7-187">You can deploy the model to Azure or to an edge device, where it can be used locally with the data on which it operates.</span></span>

<span data-ttu-id="d76a7-188">Основные преимущества:</span><span class="sxs-lookup"><span data-stu-id="d76a7-188">Key Benefits:</span></span>

- <span data-ttu-id="d76a7-189">Контейнеры — это легкий и экономически эффективный способ упаковки и развертывания служб.</span><span class="sxs-lookup"><span data-stu-id="d76a7-189">Containers are a lightweight and generally cost effective way to package and deploy services.</span></span>
- <span data-ttu-id="d76a7-190">Возможность развертывания на граничном устройстве позволяет перемещать прогнозную логику ближе к данным.</span><span class="sxs-lookup"><span data-stu-id="d76a7-190">The ability to deploy to an edge device enables you to move your predictive logic closer to the data.</span></span>

<span data-ttu-id="d76a7-191">Рекомендации:</span><span class="sxs-lookup"><span data-stu-id="d76a7-191">Considerations:</span></span>

- <span data-ttu-id="d76a7-192">Эта модель развертывания основана на контейнерах Docker, поэтому вам следует ознакомиться с этой технологией, прежде чем развертывать веб-службу таким образом.</span><span class="sxs-lookup"><span data-stu-id="d76a7-192">This deployment model is based on Docker containers, so you should be familiar with this technology before deploying a web service this way.</span></span>

### <a name="microsoft-machine-learning-server"></a><span data-ttu-id="d76a7-193">Сервер машинного обучения Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="d76a7-193">Microsoft Machine Learning Server</span></span>

<span data-ttu-id="d76a7-194">Machine Learning Server (ранее — Microsoft R Server) — это масштабируемая платформа для кода R и Python, специально разработанная для сценариев машинного обучения.</span><span class="sxs-lookup"><span data-stu-id="d76a7-194">Machine Learning Server (formerly Microsoft R Server) is a scalable platform for R and Python code, specifically designed for machine learning scenarios.</span></span>

<span data-ttu-id="d76a7-195">Основные преимущества:</span><span class="sxs-lookup"><span data-stu-id="d76a7-195">Key benefits:</span></span>

- <span data-ttu-id="d76a7-196">Высокий уровень масштабируемости.</span><span class="sxs-lookup"><span data-stu-id="d76a7-196">High scalability.</span></span>

<span data-ttu-id="d76a7-197">Рекомендации:</span><span class="sxs-lookup"><span data-stu-id="d76a7-197">Considerations:</span></span>

- <span data-ttu-id="d76a7-198">Требуется развернуть Machine Learning Server и управлять им в вашей организации.</span><span class="sxs-lookup"><span data-stu-id="d76a7-198">You need to deploy and manage Machine Learning Server in your enterprise.</span></span>

### <a name="microsoft-sql-server"></a><span data-ttu-id="d76a7-199">Microsoft SQL Server</span><span class="sxs-lookup"><span data-stu-id="d76a7-199">Microsoft SQL Server</span></span>

<span data-ttu-id="d76a7-200">Microsoft SQL Server поддерживает R и Python изначально, позволяя инкапсулировать модели машинного обучения, созданные на этих языках, в качестве функций Transact-SQL в базе данных.</span><span class="sxs-lookup"><span data-stu-id="d76a7-200">Microsoft SQL Server supports R and Python natively, enabling you to encapsulate machine learning models built in these languages as Transact-SQL functions in a database.</span></span>

<span data-ttu-id="d76a7-201">Основные преимущества:</span><span class="sxs-lookup"><span data-stu-id="d76a7-201">Key benefits:</span></span>

- <span data-ttu-id="d76a7-202">Возможность инкапсулирования прогнозной логики в функции базы данных, что позволяет легко включать ее в логику уровня данных.</span><span class="sxs-lookup"><span data-stu-id="d76a7-202">Encapsulate predictive logic in a database function, making it easy to include in data-tier logic.</span></span>

<span data-ttu-id="d76a7-203">Рекомендации:</span><span class="sxs-lookup"><span data-stu-id="d76a7-203">Considerations:</span></span>

- <span data-ttu-id="d76a7-204">Предполагается наличие базы данных SQL Server в качестве уровня данных в вашем приложении.</span><span class="sxs-lookup"><span data-stu-id="d76a7-204">Assumes a SQL Server database as the data tier for your application.</span></span>

### <a name="azure-machine-learning-web-service"></a><span data-ttu-id="d76a7-205">Веб-служба машинного обучения Azure</span><span class="sxs-lookup"><span data-stu-id="d76a7-205">Azure Machine Learning web service</span></span>

<span data-ttu-id="d76a7-206">Модель машинного обучения, создаваемую в Студии машинного обучения Azure, можно развернуть как веб-службу.</span><span class="sxs-lookup"><span data-stu-id="d76a7-206">When you create a machine learning model using Azure Machine Learning Studio, you can deploy it as a web service.</span></span> <span data-ttu-id="d76a7-207">Затем ее можно использовать с помощью интерфейса REST из любых клиентских приложений, поддерживающих обмен данными по протоколу HTTP.</span><span class="sxs-lookup"><span data-stu-id="d76a7-207">This can then be consumed through a REST interface from any client applications capable of communicating by HTTP.</span></span>

<span data-ttu-id="d76a7-208">Основные преимущества:</span><span class="sxs-lookup"><span data-stu-id="d76a7-208">Key benefits:</span></span>

- <span data-ttu-id="d76a7-209">Простота разработки и развертывания.</span><span class="sxs-lookup"><span data-stu-id="d76a7-209">Ease of development and deployment.</span></span>
- <span data-ttu-id="d76a7-210">Портал управления веб-службами с базовыми метриками мониторинга.</span><span class="sxs-lookup"><span data-stu-id="d76a7-210">Web service management portal with basic monitoring metrics.</span></span>
- <span data-ttu-id="d76a7-211">Встроенная поддержка вызова веб-служб машинного обучения из Azure Data Lake Analytics, фабрики данных Azure и Azure Stream Analytics.</span><span class="sxs-lookup"><span data-stu-id="d76a7-211">Built-in support for calling Azure Machine Learning web services from Azure Data Lake Analytics, Azure Data Factory, and Azure Stream Analytics.</span></span>

<span data-ttu-id="d76a7-212">Рекомендации:</span><span class="sxs-lookup"><span data-stu-id="d76a7-212">Considerations:</span></span>

- <span data-ttu-id="d76a7-213">Доступно только для моделей, созданных с помощью Студии машинного обучения Azure.</span><span class="sxs-lookup"><span data-stu-id="d76a7-213">Only available for models built using Azure Machine Learning Studio.</span></span>
- <span data-ttu-id="d76a7-214">Только веб-доступ. Обученные модели не работают в локальной среде или в автономном режиме.</span><span class="sxs-lookup"><span data-stu-id="d76a7-214">Web-based access only, trained models cannot run on-premises or offline.</span></span>
