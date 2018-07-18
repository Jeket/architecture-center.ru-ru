---
title: Интеллектуальные приложения обработки изображений в Azure
description: Проверенное решение для создания обработки изображений в приложениях Azure.
author: david-stanford
ms.date: 07/05/2018
ms.openlocfilehash: c5bfb9a929ddddda4336e1cbc8665a0b4d3bbe2c
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/06/2018
ms.locfileid: "37891387"
---
# <a name="insurance-claim-image-classification-on-azure"></a><span data-ttu-id="12b54-103">Классификация изображений в Azure для страховых требований</span><span class="sxs-lookup"><span data-stu-id="12b54-103">Insurance claim image classification on Azure</span></span>

<span data-ttu-id="12b54-104">Этот пример сценария применим для компаний, которым необходимо обрабатывать изображения.</span><span class="sxs-lookup"><span data-stu-id="12b54-104">This example scenario is applicable for businesses that need to process images.</span></span>

<span data-ttu-id="12b54-105">Потенциальные приложения включают классификацию изображений для веб-сайтов о моде, анализ текста и изображений для страховых компаний или распознавание телеметрических данных со скриншотов игр.</span><span class="sxs-lookup"><span data-stu-id="12b54-105">Potential applications include classifying images for a fashion website, analyzing text and images for insurance claims, or understanding telemetry data from game screenshots.</span></span> <span data-ttu-id="12b54-106">Обычно компаниям было необходимо приобретать специальные знания о моделях машинного обучения, обучать эти модели и, наконец, пропускать изображения через индивидуальный процесс, чтобы получить выходные данные.</span><span class="sxs-lookup"><span data-stu-id="12b54-106">Traditionally, companies would need to develop expertise in machine learning models, train the models, and finally run the images through their custom process to get the data out of the images.</span></span>

<span data-ttu-id="12b54-107">Используя службы Azure, такие как API компьютерного зрения и Функции Azure, компании могут избавиться от необходимости управлять отдельными серверами, одновременно сокращая затраты и используя интерфейс, который корпорация Майкрософт уже разработала для обработки изображений с помощью служб Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="12b54-107">By using Azure services such as the Computer Vision API and Azure Functions, companies can eliminate the need to manage individual servers, while reducing costs and leveraging the expertise that Microsoft has already developed around processing images with Cognitive services.</span></span> <span data-ttu-id="12b54-108">Этот сценарий обращается конкретно к сценарию обработки изображений.</span><span class="sxs-lookup"><span data-stu-id="12b54-108">This scenario specifically addresses an image processing scenario.</span></span> <span data-ttu-id="12b54-109">При наличии различных потребностей в искусственном интеллекте, можно использовать полный набор [Cognitive Services][cognitive-docs].</span><span class="sxs-lookup"><span data-stu-id="12b54-109">If you have different AI needs, consider the full suite of [Cognitive Services][cognitive-docs].</span></span>

## <a name="potential-use-cases"></a><span data-ttu-id="12b54-110">Потенциальные варианты использования</span><span class="sxs-lookup"><span data-stu-id="12b54-110">Potential use cases</span></span>

<span data-ttu-id="12b54-111">Рассмотрите это решение для следующих вариантов использования.</span><span class="sxs-lookup"><span data-stu-id="12b54-111">Consider this solution for the following use cases:</span></span>

* <span data-ttu-id="12b54-112">Классификация изображений на веб-сайте о моде.</span><span class="sxs-lookup"><span data-stu-id="12b54-112">Classify images on a fashion website.</span></span>
* <span data-ttu-id="12b54-113">Классификация изображений для страховых требований.</span><span class="sxs-lookup"><span data-stu-id="12b54-113">Classify images for insurance claims</span></span>
* <span data-ttu-id="12b54-114">Классификация телеметрических данных со скриншотов игр.</span><span class="sxs-lookup"><span data-stu-id="12b54-114">Classify telemetry data from screenshots of games.</span></span>

## <a name="architecture"></a><span data-ttu-id="12b54-115">Архитектура</span><span class="sxs-lookup"><span data-stu-id="12b54-115">Architecture</span></span>

![Архитектура интеллектуального приложения компьютерного зрения][architecture-computer-vision]

<span data-ttu-id="12b54-117">Это решение охватывает серверные компоненты Интернета или мобильного приложения.</span><span class="sxs-lookup"><span data-stu-id="12b54-117">This solution covers the back-end components of a web or mobile application.</span></span> <span data-ttu-id="12b54-118">Поток данных проходит через решение следующим образом.</span><span class="sxs-lookup"><span data-stu-id="12b54-118">Data flows through the solution as follows:</span></span>

1. <span data-ttu-id="12b54-119">Функции Azure выступают в роли уровня API.</span><span class="sxs-lookup"><span data-stu-id="12b54-119">Azure Functions acts as the API layer.</span></span> <span data-ttu-id="12b54-120">Эти API позволяют приложению передавать изображения и получать данные из Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="12b54-120">These APIs enable the application to upload images and retrieve data from Cosmos DB.</span></span>

2. <span data-ttu-id="12b54-121">После загрузки изображения через вызов API, оно сохраняется в хранилище больших двоичных объектов.</span><span class="sxs-lookup"><span data-stu-id="12b54-121">When an image is uploaded via an API call, it's stored in Blob storage.</span></span>

3. <span data-ttu-id="12b54-122">Добавление новых файлов в хранилище больших двоичных объектов активирует отправку уведомления EventGrid в функции Azure.</span><span class="sxs-lookup"><span data-stu-id="12b54-122">Adding new files to Blob storage triggers an EventGrid notification to be sent to an Azure Function.</span></span>

4. <span data-ttu-id="12b54-123">Функции Azure отправляют API компьютерного зрения ссылку на новый загруженный файл для анализа.</span><span class="sxs-lookup"><span data-stu-id="12b54-123">Azure Functions sends a link to the newly uploaded file to the Computer Vision API to analyze.</span></span>

5. <span data-ttu-id="12b54-124">После того как данные из API компьютерного зрения были возвращены, Функции Azure создают запись в базе данных Cosmos DB, чтобы сохранить результаты анализа вместе с метаданными изображения.</span><span class="sxs-lookup"><span data-stu-id="12b54-124">Once the data has been returned from the Computer Vision API, Azure Functions makes an entry in Cosmos DB to persist the results of the analysis alongside the image metadata.</span></span>

### <a name="components"></a><span data-ttu-id="12b54-125">Компоненты</span><span class="sxs-lookup"><span data-stu-id="12b54-125">Components</span></span>

* <span data-ttu-id="12b54-126">[API компьютерного зрения][computer-vision-docs] является частью пакета Cognitive Services и используется для получения сведений о каждом изображении.</span><span class="sxs-lookup"><span data-stu-id="12b54-126">[Computer Vision API][computer-vision-docs] is part of the Cognitive Services suite and is used to retrieve information about each image.</span></span>

* <span data-ttu-id="12b54-127">[Функции Azure][functions-docs] предоставляют серверный API для веб-приложения, а также обработку событий для переданных изображений.</span><span class="sxs-lookup"><span data-stu-id="12b54-127">[Azure Functions][functions-docs] provides the backend API for the web application, as well as the event processing for uploaded images.</span></span>

* <span data-ttu-id="12b54-128">[Сетка событий][eventgrid-docs] активирует событие при отправке нового изображения в хранилище больших двоичных объектов.</span><span class="sxs-lookup"><span data-stu-id="12b54-128">[Event Grid][eventgrid-docs] triggers an event when a new image is uploaded to blob storage.</span></span> <span data-ttu-id="12b54-129">Затем изображение обрабатывается с помощью функций Azure.</span><span class="sxs-lookup"><span data-stu-id="12b54-129">The image is then processed with Azure functions.</span></span>

* <span data-ttu-id="12b54-130">[Хранилище больших двоичных объектов][storage-docs] хранит все файлы изображений, которые отправляются в веб-приложение, а также все статические файлы, которые использует веб-приложение.</span><span class="sxs-lookup"><span data-stu-id="12b54-130">[Blob Storage][storage-docs] stores all of the image files that are uploaded into the web application, as well any static files that the web application consumes.</span></span>

* <span data-ttu-id="12b54-131">[Cosmos DB][cosmos-docs] хранит метаданные о каждом переданном изображении, включая результаты обработки API компьютерного зрения.</span><span class="sxs-lookup"><span data-stu-id="12b54-131">[Cosmos DB][cosmos-docs] stores metadata about each image that is uploaded, including the results of the processing from Computer Vision API.</span></span>

## <a name="alternatives"></a><span data-ttu-id="12b54-132">Альтернативные варианты</span><span class="sxs-lookup"><span data-stu-id="12b54-132">Alternatives</span></span>

* <span data-ttu-id="12b54-133">[Пользовательская служба визуального распознавания][custom-vision-docs].</span><span class="sxs-lookup"><span data-stu-id="12b54-133">[Custom Vision Service][custom-vision-docs].</span></span> <span data-ttu-id="12b54-134">API компьютерного зрения возвращает набор [таксономических категорий][cv-categories].</span><span class="sxs-lookup"><span data-stu-id="12b54-134">The Computer Vision API returns a set of [taxonomy-based categories][cv-categories].</span></span> <span data-ttu-id="12b54-135">Если вам необходимо обработать информацию, которую API компьютерного зрения не возвращает, рекомендуется использовать службу пользовательского визуального распознавания, которая позволяет создавать индивидуальные классификаторы изображений.</span><span class="sxs-lookup"><span data-stu-id="12b54-135">If you need to process information that isn't returned by the Computer Vision API, consider the Custom Vision Service, which lets you build custom image classifiers.</span></span>

* <span data-ttu-id="12b54-136">[Поиск Azure][azure-search-docs].</span><span class="sxs-lookup"><span data-stu-id="12b54-136">[Azure Search][azure-search-docs].</span></span> <span data-ttu-id="12b54-137">Если ваш вариант использования включает запросы метаданных для поиска изображений, соответствующих заданным критериям, рекомендуется использовать поиск Azure.</span><span class="sxs-lookup"><span data-stu-id="12b54-137">If your use case involves querying the metadata to find images that meet specific criteria, consider using Azure Search.</span></span> <span data-ttu-id="12b54-138">[Когнитивный поиск][cognitive-search] в предварительной версии сейчас эффективно интегрирует этот рабочий процесс.</span><span class="sxs-lookup"><span data-stu-id="12b54-138">Currently in preview, [Cognitive search][cognitive-search] seamlessly integrates this workflow.</span></span>

## <a name="considerations"></a><span data-ttu-id="12b54-139">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="12b54-139">Considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="12b54-140">Масштабируемость</span><span class="sxs-lookup"><span data-stu-id="12b54-140">Scalability</span></span>

<span data-ttu-id="12b54-141">В большинстве случаев все компоненты этого решения являются управляемыми службами, которые будут автоматически масштабироваться.</span><span class="sxs-lookup"><span data-stu-id="12b54-141">For the most part all of the components of this solution are managed services that will automatically scale.</span></span> <span data-ttu-id="12b54-142">Несколько важных исключений: функции Azure имеют ограничение до 200 экземпляров максимум.</span><span class="sxs-lookup"><span data-stu-id="12b54-142">A couple notable exceptions: Azure Functions has a limit of a maximum of 200 instances.</span></span> <span data-ttu-id="12b54-143">Если вам нужно масштабировать за этими пределами, используйте несколько регионов или планов приложений.</span><span class="sxs-lookup"><span data-stu-id="12b54-143">If you need to scale beyond, consider multiple regions or app plans.</span></span>

<span data-ttu-id="12b54-144">Cosmos DB не проводит автоматическое масштабирование в единицах запросов RUs.</span><span class="sxs-lookup"><span data-stu-id="12b54-144">Cosmos DB doesn’t auto-scale in terms of provisioned request units (RUs).</span></span>  <span data-ttu-id="12b54-145">Дополнительные сведения в руководстве по оценке требований см. в разделе [Единицы запроса в базе данных Azure Cosmos DB][request-units] нашей документации.</span><span class="sxs-lookup"><span data-stu-id="12b54-145">For guidance on estimating your requirements see [request units][request-units] in our documentation.</span></span> <span data-ttu-id="12b54-146">Чтобы полностью воспользоваться преимуществами масштабирования в Cosmos DB также следует рассмотреть [ключи раздела][partition-key].</span><span class="sxs-lookup"><span data-stu-id="12b54-146">To fully take advantage of the scaling in Cosmos DB you should also take a look at [partition keys][partition-key].</span></span>

<span data-ttu-id="12b54-147">Базы данных NoSQL часто поддерживают целостность (как теорема CAP) для доступности, масштабируемости и разделов.</span><span class="sxs-lookup"><span data-stu-id="12b54-147">NoSQL databases frequently trade consistency (in the sense of the CAP theorem) for availability, scalability and partition.</span></span>  <span data-ttu-id="12b54-148">Однако в случае моделей данных с ключевыми значениями, которые используются в этом сценарии, согласованность транзакций требуется редко, поскольку большинство операций по определению являются атомарными.</span><span class="sxs-lookup"><span data-stu-id="12b54-148">However, in the case of key-value data models which is used in this scenario, transaction consistency is rarely needed as most operations are by definition atomic.</span></span> <span data-ttu-id="12b54-149">Дополнительные рекомендации для [выбора правильного хранилища данных](../../guide/technology-choices/data-store-overview.md) доступны в центре архитектуры.</span><span class="sxs-lookup"><span data-stu-id="12b54-149">Additional guidance to [Choose the right data store](../../guide/technology-choices/data-store-overview.md) is available in the architecture center.</span></span>

<span data-ttu-id="12b54-150">Общие рекомендации по разработке масштабируемых решений см. в разделе [Контрольный список для обеспечения масштабируемости][scalability] в центре архитектуры Azure.</span><span class="sxs-lookup"><span data-stu-id="12b54-150">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="12b54-151">Безопасность</span><span class="sxs-lookup"><span data-stu-id="12b54-151">Security</span></span>

<span data-ttu-id="12b54-152">[Управляемое удостоверение службы][msi] (MSI) используется для предоставления доступа к другим ресурсам, внутренним вашей учетной записи, а затем присваивается функциям Azure.</span><span class="sxs-lookup"><span data-stu-id="12b54-152">[Managed service identities][msi] (MSI) are used to provide access to other resources internal to your account and then assigned to your Azure Functions.</span></span> <span data-ttu-id="12b54-153">В этих удостоверениях разрешайте доступ только к необходимым ресурсам, чтобы гарантировать, что ничто другое не подвергается воздействию ваших функций (и, возможно, ваших клиентов).</span><span class="sxs-lookup"><span data-stu-id="12b54-153">Only allow access to the requisite resources in those identities to ensure that nothing extra is exposed to your functions (and potentially to your customers).</span></span>  

<span data-ttu-id="12b54-154">Общие рекомендации по разработке безопасных решений см. в разделе [Azure Security Documentation][security].</span><span class="sxs-lookup"><span data-stu-id="12b54-154">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="12b54-155">Устойчивость</span><span class="sxs-lookup"><span data-stu-id="12b54-155">Resiliency</span></span>

<span data-ttu-id="12b54-156">Все компоненты в этом решении управляются, поэтому на региональном уровне все они автоматически являются устойчивыми.</span><span class="sxs-lookup"><span data-stu-id="12b54-156">All of the components in this solution are managed, so at a regional level they are all resilient automatically.</span></span> 

<span data-ttu-id="12b54-157">Общее руководство по проектированию устойчивых решений см. в разделе [Проектирование устойчивых приложений для Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="12b54-157">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="12b54-158">Цены</span><span class="sxs-lookup"><span data-stu-id="12b54-158">Pricing</span></span>

<span data-ttu-id="12b54-159">Чтобы изучить стоимость выполнения этого решения, все услуги были предварительно сконфигурированы в калькуляторе стоимости.</span><span class="sxs-lookup"><span data-stu-id="12b54-159">To explore the cost of running this solution, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="12b54-160">Чтобы узнать, как изменится цена для вашего конкретного варианта использования, измените соответствующие переменные в соответствии с ожидаемым трафиком.</span><span class="sxs-lookup"><span data-stu-id="12b54-160">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="12b54-161">Вам предоставлено три примера профиля затрат в зависимости от объема трафика (размер каждого изображения примерно 100 Кбайт).</span><span class="sxs-lookup"><span data-stu-id="12b54-161">We have provided three sample cost profiles based on amount of traffic (we assume all images are 100kb in size):</span></span>

* <span data-ttu-id="12b54-162">[Небольшой][pricing]. Коррелируется с обработкой &lt; 5000 изображений в месяц.</span><span class="sxs-lookup"><span data-stu-id="12b54-162">[Small][pricing]: this correlates to processing &lt; 5000 images a month.</span></span>
* <span data-ttu-id="12b54-163">[Средний][medium-pricing]. Коррелируется с обработкой 500 000 изображений в месяц.</span><span class="sxs-lookup"><span data-stu-id="12b54-163">[Medium][medium-pricing]: this correlates to processing 500,000 images a month.</span></span>
* <span data-ttu-id="12b54-164">[Большой][large-pricing]. Коррелируется с обработкой 50 млн изображений в месяц.</span><span class="sxs-lookup"><span data-stu-id="12b54-164">[Large][large-pricing]: this correlates to processing 50 million images a month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="12b54-165">Связанные ресурсы</span><span class="sxs-lookup"><span data-stu-id="12b54-165">Related Resources</span></span>

<span data-ttu-id="12b54-166">Пошаговую схему обучения этого решения см. в разделе [Создание бессерверных приложений в Azure][serverless].</span><span class="sxs-lookup"><span data-stu-id="12b54-166">For a guided learning path of this solution, see [Build a serverless web app in Azure][serverless].</span></span>  

<span data-ttu-id="12b54-167">Прежде чем поместить это в рабочей среде, ознакомьтесь с [рекомендациями][functions-best-practices] относительно функций Azure.</span><span class="sxs-lookup"><span data-stu-id="12b54-167">Before putting this in a production environment, review the Azure Functions [best practices][functions-best-practices].</span></span>

<!-- links -->
[pricing]: https://azure.com/e/f9b59d238b43423683db73f4a31dc380
[medium-pricing]: https://azure.com/e/7c7fc474db344b87aae93bc29ae27108
[large-pricing]: https://azure.com/e/cbadbca30f8640d6a061f8457a74ba7d
[functions-docs]: /azure/azure-functions/
[computer-vision-docs]: /azure/cognitive-services/computer-vision/home
[storage-docs]: /azure/storage/
[azure-search-docs]: /azure/search/
[cognitive-search]: /azure/search/cognitive-search-concept-intro
[architecture-computer-vision]: ./media/architecture-computer-vision.png
[serverless]: /azure/functions/tutorial-static-website-serverless-api-with-database
[cosmos-docs]: /azure/cosmos-db/
[eventgrid-docs]: /azure/event-grid/
[cognitive-docs]: /azure/#pivot=products&panel=ai
[custom-vision-docs]: /azure/cognitive-services/Custom-Vision-Service/home
[cv-categories]: /azure/cognitive-services/computer-vision/home#the-86-category-concept
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[functions-best-practices]: /azure/azure-functions/functions-best-practices
[msi]: /azure/app-service/app-service-managed-service-identity
[request-units]: /azure/cosmos-db/request-units
[partition-key]: /azure/cosmos-db/partition-data