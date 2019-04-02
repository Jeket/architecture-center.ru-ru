---
title: Массовый прием и анализ пакетов новостей в Azure
description: Создайте конвейер для приема и анализа текста, изображений, тональности и других данных из RSS-пакетов новостей, используя только службы Azure, такие как Azure Cosmos DB и Azure Cognitive Services.
author: njray
ms.date: 2/1/2019
ms.custom: azcat-ai, AI
social_image_url: /azure/architecture/example-scenario/ai/media/mass-ingestion-newsfeeds-architecture.png
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.openlocfilehash: e1bc636103753d474b545a6e3e9118b2ef302b34
ms.sourcegitcommit: ea97ac004c38c6b456794c1a8eef29f8d2b77d50
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2019
ms.locfileid: "58489266"
---
# <a name="mass-ingestion-and-analysis-of-news-feeds-on-azure"></a><span data-ttu-id="5418a-103">Массовый прием и анализ пакетов новостей в Azure</span><span class="sxs-lookup"><span data-stu-id="5418a-103">Mass ingestion and analysis of news feeds on Azure</span></span>

<span data-ttu-id="5418a-104">В этом примере сценария описывается конвейер для массового приема и практически в реальном времени анализа документов с помощью общедоступных веб-каналов новостей RSS.</span><span class="sxs-lookup"><span data-stu-id="5418a-104">This example scenario describes a pipeline for mass ingestion and near real-time analysis of documents using public RSS news feeds.</span></span>  <span data-ttu-id="5418a-105">Она использует Azure Cognitive Services для полезных сообщают, включая перевод текста, распознавание лиц и определения мнений.</span><span class="sxs-lookup"><span data-stu-id="5418a-105">It uses Azure Cognitive Services to offer useful insights including text translation, facial recognition, and sentiment detection.</span></span>

<span data-ttu-id="5418a-106">Этот сценарий содержит примеры для [Английский][english], [русский][russian], и [немецкий] [ german] новостей веб-каналы, но вы можете легко перенести его на другие RSS-каналы.</span><span class="sxs-lookup"><span data-stu-id="5418a-106">This scenario contains examples for [English][english], [Russian][russian], and [German][german] news feeds, but you can easily extend it to other RSS feeds.</span></span> <span data-ttu-id="5418a-107">Для простоты развертывания сбор данных, обработки и анализа основаны полностью на службы Azure.</span><span class="sxs-lookup"><span data-stu-id="5418a-107">For ease of deployment, the data collection, processing, and analysis are based entirely on Azure services.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="5418a-108">Варианты соответствующего использования</span><span class="sxs-lookup"><span data-stu-id="5418a-108">Relevant use cases</span></span>

<span data-ttu-id="5418a-109">Хотя этот сценарий основан на обработке RSS-каналы, это относится к любой документ, веб-сайта или статье, где необходимо:</span><span class="sxs-lookup"><span data-stu-id="5418a-109">While this scenario is based on processing of RSS feeds, it's relevant to any document, website, or article where you would need to:</span></span>

* <span data-ttu-id="5418a-110">Перевод любой текст на выбранном языке.</span><span class="sxs-lookup"><span data-stu-id="5418a-110">Translate any text to the language of choice.</span></span>
* <span data-ttu-id="5418a-111">Найти ключевые фразы, сущности и мнений пользователей в цифрового содержимого.</span><span class="sxs-lookup"><span data-stu-id="5418a-111">Find key phrases, entities, and user sentiment in digital content.</span></span>
* <span data-ttu-id="5418a-112">Обнаруживать объекты "," текст "и" ориентиры в изображений, связанных с цифровой статьи.</span><span class="sxs-lookup"><span data-stu-id="5418a-112">Detect objects, text, and landmarks in images associated with a digital article.</span></span>
* <span data-ttu-id="5418a-113">Обнаруживать пользователей по их пол и возраст в любое другое изображение, связанное с цифровым содержимым.</span><span class="sxs-lookup"><span data-stu-id="5418a-113">Detect people by their gender and age in any image associated with digital content.</span></span>

## <a name="architecture"></a><span data-ttu-id="5418a-114">Архитектура</span><span class="sxs-lookup"><span data-stu-id="5418a-114">Architecture</span></span>

![][architecture]

<span data-ttu-id="5418a-115">Поток данных проходит через решение следующим образом.</span><span class="sxs-lookup"><span data-stu-id="5418a-115">The data flows through the solution as follows:</span></span>

1. <span data-ttu-id="5418a-116">Новости RSS-канал действует как генератор, который получает данные из документа или статье.</span><span class="sxs-lookup"><span data-stu-id="5418a-116">An RSS news feed acts as the generator that obtains data from a document or article.</span></span> <span data-ttu-id="5418a-117">Например статьи, данные обычно включают в себя заголовок, сводку исходный текст элемента новостей и иногда изображений.</span><span class="sxs-lookup"><span data-stu-id="5418a-117">For example, with an article, data typically includes a title, a summary of the original body of the news item, and sometimes images.</span></span>

2. <span data-ttu-id="5418a-118">Генератор или приема процесс вставляет в этой статье и все связанные образы в Azure Cosmos DB [коллекции][collection].</span><span class="sxs-lookup"><span data-stu-id="5418a-118">A generator or ingestion process inserts the article and any associated images into an Azure Cosmos DB [Collection][collection].</span></span>

3. <span data-ttu-id="5418a-119">Уведомление активирует функцию приема в функциях Azure, который хранит текста статьи в CosmosDB и изображений (если таковые имеются) в хранилище BLOB-объектов.</span><span class="sxs-lookup"><span data-stu-id="5418a-119">A notification triggers an ingest function in Azure Functions that stores the article text in CosmosDB and the article images (if any) in Azure Blob Storage.</span></span>  <span data-ttu-id="5418a-120">Статья, затем передается на следующую очередь.</span><span class="sxs-lookup"><span data-stu-id="5418a-120">The article is then passed to the next queue.</span></span>

4. <span data-ttu-id="5418a-121">Функция translate инициируется событие очереди.</span><span class="sxs-lookup"><span data-stu-id="5418a-121">A translate function is triggered by the queue event.</span></span> <span data-ttu-id="5418a-122">Она использует [API перевода текстов] [ translate-text] Azure Cognitive Services для определения языка, перевод, при необходимости, а сбор тело и заголовок тональности, ключевых фраз и сущностей.</span><span class="sxs-lookup"><span data-stu-id="5418a-122">It uses the [Translate Text API][translate-text] of Azure Cognitive Services to detect the language, translate if necessary, and collect the sentiment, key phrases, and entities from the body and the title.</span></span> <span data-ttu-id="5418a-123">Затем он передает статьи на следующую очередь.</span><span class="sxs-lookup"><span data-stu-id="5418a-123">Then it passes the article to the next queue.</span></span>

5. <span data-ttu-id="5418a-124">Функция обнаружения запускается из статьи в очереди.</span><span class="sxs-lookup"><span data-stu-id="5418a-124">A detect function is triggered from the queued article.</span></span> <span data-ttu-id="5418a-125">Она использует [компьютерного] [ vision] служба обнаружения объектов, ориентиров и письменные слов в нужный образ, затем передает статьи на следующую очередь.</span><span class="sxs-lookup"><span data-stu-id="5418a-125">It uses the [Computer Vision][vision] service to detect objects, landmarks, and written words in the associated image, then passes the article to the next queue.</span></span>

6. <span data-ttu-id="5418a-126">Функция распознавания лиц запускается активируется из статьи в очереди.</span><span class="sxs-lookup"><span data-stu-id="5418a-126">A face function is triggered is triggered from the queued article.</span></span> <span data-ttu-id="5418a-127">Она использует [API распознавания лиц Azure] [ face] служба обнаружения лиц для Пол и возраст в нужный образ, а затем передает статьи на следующую очередь.</span><span class="sxs-lookup"><span data-stu-id="5418a-127">It uses the [Azure Face API][face] service to detect faces for gender and age in the associated image, then passes the article to the next queue.</span></span>

7. <span data-ttu-id="5418a-128">После выполнения всех функций активации функции уведомления.</span><span class="sxs-lookup"><span data-stu-id="5418a-128">When all functions are complete, the notify function is triggered.</span></span> <span data-ttu-id="5418a-129">Он загружает обработанные записи для статьи и ищет все нужные результаты.</span><span class="sxs-lookup"><span data-stu-id="5418a-129">It loads the processed records for the article and scans them for any results you want.</span></span> <span data-ttu-id="5418a-130">Если найден, содержимое помечен и уведомление отправляется в систему по своему усмотрению.</span><span class="sxs-lookup"><span data-stu-id="5418a-130">If found, the content is flagged and a notification is sent to the system of your choice.</span></span>

<span data-ttu-id="5418a-131">На каждом этапе обработки функция записывает результаты в Azure Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="5418a-131">At each processing step, the function writes the results to Azure Cosmos DB.</span></span> <span data-ttu-id="5418a-132">В конечном счете, данные могут использоваться необходимости.</span><span class="sxs-lookup"><span data-stu-id="5418a-132">Ultimately, the data can be used as desired.</span></span> <span data-ttu-id="5418a-133">Например его можно использовать для улучшения бизнес-процессов, найти новых клиентов или выявить проблемы удовлетворенность клиента.</span><span class="sxs-lookup"><span data-stu-id="5418a-133">For example, you can use it to enhance business processes, locate new customers, or identify customer satisfaction issues.</span></span>

### <a name="components"></a><span data-ttu-id="5418a-134">Компоненты</span><span class="sxs-lookup"><span data-stu-id="5418a-134">Components</span></span>

<span data-ttu-id="5418a-135">В этом примере используется следующий список компонентов Azure.</span><span class="sxs-lookup"><span data-stu-id="5418a-135">The following list of Azure components is used in this example.</span></span>

* <span data-ttu-id="5418a-136">[Служба хранилища Azure] [ storage] используется для хранения изображений RAW-формата и видео файлов, связанных с статьи.</span><span class="sxs-lookup"><span data-stu-id="5418a-136">[Azure Storage][storage] is used to hold raw image and video files associated with an article.</span></span> <span data-ttu-id="5418a-137">Дополнительную учетную запись хранения создается с помощью службы приложений Azure и используется для размещения кода функции Azure и журналы.</span><span class="sxs-lookup"><span data-stu-id="5418a-137">A secondary storage account is created with Azure App Service and is used to host the Azure Function code and logs.</span></span>

* <span data-ttu-id="5418a-138">[Azure Cosmos DB] [ cosmos-db] содержит статьи, текст, изображения и видео, сведения об отслеживании.</span><span class="sxs-lookup"><span data-stu-id="5418a-138">[Azure Cosmos DB][cosmos-db] holds article text, image, and video tracking information.</span></span> <span data-ttu-id="5418a-139">Результаты действия Cognitive Services, здесь также хранятся.</span><span class="sxs-lookup"><span data-stu-id="5418a-139">The results of the Cognitive Services steps are also stored here.</span></span>

* <span data-ttu-id="5418a-140">[Функции Azure] [ functions] выполняет код функции реагировать очереди сообщений и преобразования входящего содержимого.</span><span class="sxs-lookup"><span data-stu-id="5418a-140">[Azure Functions][functions] executes the function code used to respond to queue messages and transform the incoming content.</span></span> <span data-ttu-id="5418a-141">[Служба приложений Azure] [ aas] размещает код функции и последовательно обрабатывает записи.</span><span class="sxs-lookup"><span data-stu-id="5418a-141">[Azure App Service][aas] hosts the function code and processes the records serially.</span></span> <span data-ttu-id="5418a-142">Этот сценарий включает пять функций: Прием, преобразования, обнаружить объект, лицо и уведомления.</span><span class="sxs-lookup"><span data-stu-id="5418a-142">This scenario includes five functions: Ingest, Transform, Detect Object, Face, and Notify.</span></span>

* <span data-ttu-id="5418a-143">[Azure Service Bus] [ service-bus] размещает очередей служебной шины Azure, используемые функциями.</span><span class="sxs-lookup"><span data-stu-id="5418a-143">[Azure Service Bus][service-bus] hosts the Azure Service Bus queues used by the functions.</span></span>

* <span data-ttu-id="5418a-144">[Azure Cognitive Services] [ acs] предоставляет искусственного Интеллекта для конвейера, на основе реализации [компьютерного] [ vision] службы, [лиц API][face], и [перевести текст] [ translate-text] служба машинного перевода.</span><span class="sxs-lookup"><span data-stu-id="5418a-144">[Azure Cognitive Services][acs] provides the AI for the pipeline based on implementations of the [Computer Vision][vision] service, [Face API][face], and [Translate Text][translate-text] machine translation service.</span></span>

* <span data-ttu-id="5418a-145">[Azure Application Insights] [ aai] предоставляет analytics которые помогут вам диагностировать проблемы и понять функциональности приложения.</span><span class="sxs-lookup"><span data-stu-id="5418a-145">[Azure Application Insights][aai] provides analytics to help you diagnose issues and to understand functionality of your application.</span></span>

### <a name="alternatives"></a><span data-ttu-id="5418a-146">Альтернативные варианты</span><span class="sxs-lookup"><span data-stu-id="5418a-146">Alternatives</span></span>

* <span data-ttu-id="5418a-147">Вместо того чтобы использовать шаблон, основанный на очереди уведомлений и функций Azure, используйте другой шаблон для этого потока данных.</span><span class="sxs-lookup"><span data-stu-id="5418a-147">Instead of using a pattern based on queue notification and Azure Functions, use another pattern for this data flow.</span></span> <span data-ttu-id="5418a-148">Например [разделы служебной шины Azure] [ topics] может использоваться для процессов, различные части статьи в параллельном режиме в отличие от последовательной обработки в этом примере.</span><span class="sxs-lookup"><span data-stu-id="5418a-148">For example, [Azure Service Bus Topics][topics] can be used to processes the various parts of the article in parallel as opposed to the serial processing done in this example.</span></span> <span data-ttu-id="5418a-149">Дополнительные сведения, сравните [очередей и разделов][queues-topics].</span><span class="sxs-lookup"><span data-stu-id="5418a-149">For more information, compare [queues and topics][queues-topics].</span></span>

* <span data-ttu-id="5418a-150">Используйте [приложения логики Azure] [ logic-app] для реализации кода самой функции и реализации, такие как блокировка на уровне записи [Redlock] [ redlock] (необходимые для параллельного обработку до Azure Cosmos DB поддерживает [обновления частичный документ][partial]).</span><span class="sxs-lookup"><span data-stu-id="5418a-150">Use [Azure Logic App][logic-app] to implement the function code and implement record-level locking such as [Redlock][redlock] (needed for parallel processing until Azure Cosmos DB supports [partial document updates][partial]).</span></span> <span data-ttu-id="5418a-151">Дополнительные сведения [сравнение функций и Logic Apps][compare].</span><span class="sxs-lookup"><span data-stu-id="5418a-151">For more information, [compare Functions and Logic Apps][compare].</span></span>

* <span data-ttu-id="5418a-152">Реализации этой архитектуры с помощью настраиваемых компонентов искусственного Интеллекта, а не на существующие службы Azure.</span><span class="sxs-lookup"><span data-stu-id="5418a-152">Implement this architecture using customized AI components rather than existing Azure services.</span></span> <span data-ttu-id="5418a-153">Например расширение конвейера, с помощью настраиваемых модели, определяющей определенным пользователям изображения число универсальных людей, пол, в отличие от и возраст данных, собранных в этом примере.</span><span class="sxs-lookup"><span data-stu-id="5418a-153">For example, extend the pipeline using a customized model that detects certain people in an image as opposed to the generic people count, gender, and age data collected in this example.</span></span> <span data-ttu-id="5418a-154">Чтобы использовать настраиваемые машинного обучения или модели искусственного Интеллекта с этой архитектурой, создайте модель как конечные точки RESTful, поэтому они могут вызываться из функций Azure.</span><span class="sxs-lookup"><span data-stu-id="5418a-154">To use customized machine learning or AI models with this architecture, build the models as RESTful endpoints so they can be called from Azure Functions.</span></span>

* <span data-ttu-id="5418a-155">Используйте другой механизм ввода вместо RSS-каналы.</span><span class="sxs-lookup"><span data-stu-id="5418a-155">Use a different input mechanism instead of RSS feeds.</span></span> <span data-ttu-id="5418a-156">Используйте несколько генераторов и процессы приема для веб-канала Azure Cosmos DB и службы хранилища Azure.</span><span class="sxs-lookup"><span data-stu-id="5418a-156">Use multiple generators or ingestion processes to feed Azure Cosmos DB and Azure Storage.</span></span>

## <a name="considerations"></a><span data-ttu-id="5418a-157">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="5418a-157">Considerations</span></span>

<span data-ttu-id="5418a-158">Для простоты в этом примере сценария используется только небольшая часть доступных API-интерфейсов и служб в Azure Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="5418a-158">For simplicity, this example scenario uses only a few of the available APIs and services from Azure Cognitive Services.</span></span> <span data-ttu-id="5418a-159">Например, текст на изображениях можно проанализировать с помощью [API текстовой аналитики][text-analytics].</span><span class="sxs-lookup"><span data-stu-id="5418a-159">For example, text in images can be analyzed using the [Text Analytics API][text-analytics].</span></span> <span data-ttu-id="5418a-160">Целевой язык в этом сценарии предполагается, что на английском языке, но можно изменить входные данные для любого [поддерживаемого языка] [ language] по своему усмотрению.</span><span class="sxs-lookup"><span data-stu-id="5418a-160">The target language in this scenario is assumed to be English, but you can change the input to any [supported language][language] of your choice.</span></span>

### <a name="scalability"></a><span data-ttu-id="5418a-161">Масштабируемость</span><span class="sxs-lookup"><span data-stu-id="5418a-161">Scalability</span></span>

<span data-ttu-id="5418a-162">Масштабирование функций Azure зависит от [план размещения] [ plan] использовании.</span><span class="sxs-lookup"><span data-stu-id="5418a-162">Azure Functions scaling depends on the [hosting plan][plan] you use.</span></span> <span data-ttu-id="5418a-163">В этом решении предполагается [план потребления][plan-c], в которой вычислительное power автоматически выделяется в функции, при необходимости.</span><span class="sxs-lookup"><span data-stu-id="5418a-163">This solution assumes a [Consumption plan][plan-c], in which compute power is automatically allocated to the functions when required.</span></span> <span data-ttu-id="5418a-164">Вы платите только при выполнении функций.</span><span class="sxs-lookup"><span data-stu-id="5418a-164">You pay only when your functions are running.</span></span> <span data-ttu-id="5418a-165">Другой вариант — использовать [службе приложений Azure] [ plan-aas] план, который позволяет масштабировать между уровнями для выделения разного объема ресурсов.</span><span class="sxs-lookup"><span data-stu-id="5418a-165">Another option is to use an [Azure App Service][plan-aas] plan, which allows you to scale between tiers to allocate a different amount of resources.</span></span>

<span data-ttu-id="5418a-166">С помощью Azure Cosmos DB, ключ является распределение рабочей нагрузки примерно поровну между достаточно большим количеством [ключей секций][keys].</span><span class="sxs-lookup"><span data-stu-id="5418a-166">With Azure Cosmos DB, the key is to distribute your workload roughly evenly among a sufficiently large number of [partition keys][keys].</span></span> <span data-ttu-id="5418a-167">Нет ограничений на общий объем данных, который может хранить контейнер или общий объем [пропускной способности] [ throughput] , поддерживаемой контейнером.</span><span class="sxs-lookup"><span data-stu-id="5418a-167">There's no limit to the total amount of data that a container can store or to the total amount of [throughput][throughput] that a container can support.</span></span>

### <a name="management-and-logging"></a><span data-ttu-id="5418a-168">Управление и регистрация</span><span class="sxs-lookup"><span data-stu-id="5418a-168">Management and logging</span></span>

<span data-ttu-id="5418a-169">Это решение использует [Application Insights] [ aai] для сбора данных производительности и сведения о ведении журнала.</span><span class="sxs-lookup"><span data-stu-id="5418a-169">This solution uses [Application Insights][aai] to collect performance and logging information.</span></span> <span data-ttu-id="5418a-170">Экземпляр Application Insights создается с помощью развертывания в той же группе ресурсов, что другие службы, необходимые для этого развертывания.</span><span class="sxs-lookup"><span data-stu-id="5418a-170">An instance of Application Insights is created with the deployment in the same resource group as the other services needed for this deployment.</span></span>

<span data-ttu-id="5418a-171">Чтобы просмотреть журналы, созданные решения:</span><span class="sxs-lookup"><span data-stu-id="5418a-171">To view the logs generated by the solution:</span></span>

1. <span data-ttu-id="5418a-172">Перейдите к [портала Azure] [ portal] и перейдите к группе ресурсов, созданной для развертывания.</span><span class="sxs-lookup"><span data-stu-id="5418a-172">Go to [Azure portal][portal] and navigate to the resource group created for the deployment.</span></span>

2. <span data-ttu-id="5418a-173">Нажмите кнопку **Application Insights** экземпляра.</span><span class="sxs-lookup"><span data-stu-id="5418a-173">Click the **Application Insights** instance.</span></span>

3. <span data-ttu-id="5418a-174">Из **Application Insights** разделе, перейдите в каталог **исследовать\\поиска** и поиск данных.</span><span class="sxs-lookup"><span data-stu-id="5418a-174">From the **Application Insights** section, navigate to **Investigate\\Search** and search the data.</span></span>

### <a name="security"></a><span data-ttu-id="5418a-175">Безопасность</span><span class="sxs-lookup"><span data-stu-id="5418a-175">Security</span></span>

<span data-ttu-id="5418a-176">Azure Cosmos DB использует безопасное подключение и подписи общего доступа через C\# SDK, предоставляемые корпорацией Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="5418a-176">Azure Cosmos DB uses a secured connection and shared access signature through the C\# SDK provided by Microsoft.</span></span> <span data-ttu-id="5418a-177">Существуют нет других внешний контактные зоны.</span><span class="sxs-lookup"><span data-stu-id="5418a-177">There are no other externally facing surface areas.</span></span> <span data-ttu-id="5418a-178">Дополнительные сведения о безопасности [рекомендации] [ db-practices] для Azure Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="5418a-178">Learn more about security [best practices][db-practices] for Azure Cosmos DB.</span></span>

## <a name="pricing"></a><span data-ttu-id="5418a-179">Цены</span><span class="sxs-lookup"><span data-stu-id="5418a-179">Pricing</span></span>

<span data-ttu-id="5418a-180">Расчетная стоимость ежедневно для сохранения этого развертывания доступны составляет приблизительно \$20 США без данных, перемещение по системе.</span><span class="sxs-lookup"><span data-stu-id="5418a-180">The estimated daily cost to keep this deployment available is approximately \$20 U.S. with no data moving through the system.</span></span>

<span data-ttu-id="5418a-181">Azure Cosmos DB обладает широкими возможностями, но влечет за собой наиболее [стоимость] [ db-cost] в этом развертывании.</span><span class="sxs-lookup"><span data-stu-id="5418a-181">Azure Cosmos DB is powerful but incurs the greatest [cost][db-cost] in this deployment.</span></span> <span data-ttu-id="5418a-182">Можно использовать другое решение хранения рефакторингом кода функций Azure, предоставленного.</span><span class="sxs-lookup"><span data-stu-id="5418a-182">You can use another storage solution by refactoring the Azure Functions code provided.</span></span>

<span data-ttu-id="5418a-183">Цены для функций Azure отличается в зависимости от [план] [ function-plan] она выполняется.</span><span class="sxs-lookup"><span data-stu-id="5418a-183">Pricing for Azure Functions varies depending on the [plan][function-plan] it runs in.</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="5418a-184">Развертывание сценария</span><span class="sxs-lookup"><span data-stu-id="5418a-184">Deploy the scenario</span></span>

> [!NOTE]
> <span data-ttu-id="5418a-185">Необходимо иметь учетную запись Azure.</span><span class="sxs-lookup"><span data-stu-id="5418a-185">You must have an existing Azure account.</span></span> <span data-ttu-id="5418a-186">Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись][free], прежде чем начинать работу.</span><span class="sxs-lookup"><span data-stu-id="5418a-186">If you don't have an Azure subscription, create a [free account][free] before you begin.</span></span>

<span data-ttu-id="5418a-187">Весь код для этого сценария доступно в [GitHub] [ github] репозитория.</span><span class="sxs-lookup"><span data-stu-id="5418a-187">All the code for this scenario is available in the [GitHub][github] repository.</span></span> <span data-ttu-id="5418a-188">Этот репозиторий содержит исходный код, используемый для создания приложения генератора с веб-каналов в конвейер для этой демонстрации.</span><span class="sxs-lookup"><span data-stu-id="5418a-188">This repository contains the source code used to build the generator application that feeds the pipeline for this demo.</span></span>

[architecture]: ./media/mass-ingestion-newsfeeds-architecture.png
[aai]: /azure/azure-monitor/app/app-insights-overview
[aas]: https://azure.microsoft.com/try/app-service/
[acs]: https://azure.microsoft.com/services/cognitive-services/directory/
[collection]: /rest/api/cosmos-db/collections
[compare]: /azure/azure-functions/functions-compare-logic-apps-ms-flow-webjobs#compare-azure-functions-and-azure-logic-apps
[cosmos-db]: /azure/cosmos-db/introduction
[db-cost]: https://azure.microsoft.com/pricing/details/cosmos-db/
[db-practices]: /azure/cosmos-db/database-security
[db-collection]: /azure/cosmos-db/databases-containers-items
[english]: https://www.nasa.gov/rss/dyn/breaking_news.rss
[face]: /azure/cognitive-services/face/overview
[free]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[functions]: /azure/azure-functions/functions-overview
[function-plan]: /azure/azure-functions/functions-scale
[german]: http://www.bamf.de/SiteGlobals/Functions/RSS/DE/Feed/RSSNewsfeed_Meldungen
[github]: https://github.com/Azure/cognitive-services
[keys]: /azure/cosmos-db/partition-data
[language]: /azure/cognitive-services/translator/reference/v3-0-languages
[logic-app]: /azure/logic-apps/logic-apps-overview
[queues-topics]: /azure/service-bus-messaging/service-bus-queues-topics-subscriptions
[partial]: https://feedback.azure.com/forums/263030-azure-cosmos-db/suggestions/6693091-be-able-to-do-partial-updates-on-document
[plan]: /azure/azure-functions/functions-scale
[plan-aas]: /azure/azure-functions/functions-scale#app-service-plan
[plan-c]: /azure/azure-functions/functions-scale#consumption-plan
[portal]: http://portal.azure.com
[redlock]: https://redis.io/topics/distlock
[russian]: http://government.ru/all/rss/
[service-bus]: /azure/service-bus-messaging/
[storage]: /azure/storage/common/storage-account-overview 
[throughput]: /azure/cosmos-db/scaling-throughput
[topics]: /azure/service-bus-messaging/service-bus-dotnet-how-to-use-topics-subscriptions
[text-analytics]: /azure/cognitive-services/text-analytics/
[translate-text]: /azure/cognitive-services/translator/translator-info-overview
[vision]: /azure/cognitive-services/computer-vision/home
