---
title: Классификация изображений в Azure для страховых требований
description: Проверенный сценарий для создания обработки изображений в приложениях Azure.
author: david-stanford
ms.date: 07/05/2018
ms.openlocfilehash: 0ca0b46e83219afc5e22c2ac6467bf4be945c97a
ms.sourcegitcommit: c49aeef818d7dfe271bc4128b230cfc676f05230
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/11/2018
ms.locfileid: "44389168"
---
# <a name="image-classification-for-insurance-claims-on-azure"></a><span data-ttu-id="88975-103">Классификация изображений в Azure для страховых требований</span><span class="sxs-lookup"><span data-stu-id="88975-103">Image classification for insurance claims on Azure</span></span>

<span data-ttu-id="88975-104">Этот пример сценария применим для компаний, которым необходимо обрабатывать изображения.</span><span class="sxs-lookup"><span data-stu-id="88975-104">This example scenario is applicable for businesses that need to process images.</span></span>

<span data-ttu-id="88975-105">Потенциальные приложения включают классификацию изображений для веб-сайтов о моде, анализ текста и изображений для страховых компаний или распознавание телеметрических данных со скриншотов игр.</span><span class="sxs-lookup"><span data-stu-id="88975-105">Potential applications include classifying images for a fashion website, analyzing text and images for insurance claims, or understanding telemetry data from game screenshots.</span></span> <span data-ttu-id="88975-106">Обычно компаниям было необходимо приобретать специальные знания о моделях машинного обучения, обучать эти модели и, наконец, пропускать изображения через индивидуальный процесс, чтобы получить выходные данные.</span><span class="sxs-lookup"><span data-stu-id="88975-106">Traditionally, companies would need to develop expertise in machine learning models, train the models, and finally run the images through their custom process to get the data out of the images.</span></span>

<span data-ttu-id="88975-107">Используя службы Azure, такие как API компьютерного зрения и Функции Azure, компании могут избавиться от необходимости управлять отдельными серверами, одновременно сокращая затраты и используя интерфейс, который корпорация Майкрософт уже разработала для обработки изображений с помощью служб Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="88975-107">By using Azure services such as the Computer Vision API and Azure Functions, companies can eliminate the need to manage individual servers, while reducing costs and leveraging the expertise that Microsoft has already developed around processing images with Cognitive services.</span></span> <span data-ttu-id="88975-108">Этот пример сценария предусматривает использование обработки изображений.</span><span class="sxs-lookup"><span data-stu-id="88975-108">This example scenario specifically addresses an image-processing use case.</span></span> <span data-ttu-id="88975-109">При наличии различных потребностей в искусственном интеллекте, можно использовать полный набор [Cognitive Services][cognitive-docs].</span><span class="sxs-lookup"><span data-stu-id="88975-109">If you have different AI needs, consider the full suite of [Cognitive Services][cognitive-docs].</span></span>

## <a name="related-use-cases"></a><span data-ttu-id="88975-110">Связанные варианты использования</span><span class="sxs-lookup"><span data-stu-id="88975-110">Related use cases</span></span>

<span data-ttu-id="88975-111">Рассмотрите этот сценарий для следующих вариантов использования:</span><span class="sxs-lookup"><span data-stu-id="88975-111">Consider this scenario for the following use cases:</span></span>

* <span data-ttu-id="88975-112">Классификация изображений на веб-сайте о моде.</span><span class="sxs-lookup"><span data-stu-id="88975-112">Classify images on a fashion website.</span></span>
* <span data-ttu-id="88975-113">Классификация телеметрических данных со скриншотов игр.</span><span class="sxs-lookup"><span data-stu-id="88975-113">Classify telemetry data from screenshots of games.</span></span>

## <a name="architecture"></a><span data-ttu-id="88975-114">Архитектура</span><span class="sxs-lookup"><span data-stu-id="88975-114">Architecture</span></span>

![Архитектура интеллектуального приложения компьютерного зрения][architecture-computer-vision]

<span data-ttu-id="88975-116">Этот сценарий охватывает серверные компоненты веб-приложения или мобильного приложения.</span><span class="sxs-lookup"><span data-stu-id="88975-116">This scenario covers the back-end components of a web or mobile application.</span></span> <span data-ttu-id="88975-117">Данные передаются в сценарии следующим образом:</span><span class="sxs-lookup"><span data-stu-id="88975-117">Data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="88975-118">Функции Azure выступают в роли уровня API.</span><span class="sxs-lookup"><span data-stu-id="88975-118">Azure Functions acts as the API layer.</span></span> <span data-ttu-id="88975-119">Эти API позволяют приложению передавать изображения и получать данные из Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="88975-119">These APIs enable the application to upload images and retrieve data from Cosmos DB.</span></span>

2. <span data-ttu-id="88975-120">После загрузки изображения через вызов API, оно сохраняется в хранилище больших двоичных объектов.</span><span class="sxs-lookup"><span data-stu-id="88975-120">When an image is uploaded via an API call, it's stored in Blob storage.</span></span>

3. <span data-ttu-id="88975-121">Добавление новых файлов в хранилище больших двоичных объектов активирует отправку уведомления EventGrid в функции Azure.</span><span class="sxs-lookup"><span data-stu-id="88975-121">Adding new files to Blob storage triggers an EventGrid notification to be sent to an Azure Function.</span></span>

4. <span data-ttu-id="88975-122">Функции Azure отправляют API компьютерного зрения ссылку на новый загруженный файл для анализа.</span><span class="sxs-lookup"><span data-stu-id="88975-122">Azure Functions sends a link to the newly uploaded file to the Computer Vision API to analyze.</span></span>

5. <span data-ttu-id="88975-123">После того как данные из API компьютерного зрения были возвращены, Функции Azure создают запись в базе данных Cosmos DB, чтобы сохранить результаты анализа вместе с метаданными изображения.</span><span class="sxs-lookup"><span data-stu-id="88975-123">Once the data has been returned from the Computer Vision API, Azure Functions makes an entry in Cosmos DB to persist the results of the analysis alongside the image metadata.</span></span>

### <a name="components"></a><span data-ttu-id="88975-124">Компоненты</span><span class="sxs-lookup"><span data-stu-id="88975-124">Components</span></span>

* <span data-ttu-id="88975-125">[API компьютерного зрения][computer-vision-docs] является частью пакета Cognitive Services и используется для получения сведений о каждом изображении.</span><span class="sxs-lookup"><span data-stu-id="88975-125">[Computer Vision API][computer-vision-docs] is part of the Cognitive Services suite and is used to retrieve information about each image.</span></span>

* <span data-ttu-id="88975-126">[Функции Azure][functions-docs] предоставляют серверный API для веб-приложения, а также обработку событий для переданных изображений.</span><span class="sxs-lookup"><span data-stu-id="88975-126">[Azure Functions][functions-docs] provides the backend API for the web application, as well as the event processing for uploaded images.</span></span>

* <span data-ttu-id="88975-127">[Сетка событий][eventgrid-docs] активирует событие при отправке нового изображения в хранилище больших двоичных объектов.</span><span class="sxs-lookup"><span data-stu-id="88975-127">[Event Grid][eventgrid-docs] triggers an event when a new image is uploaded to blob storage.</span></span> <span data-ttu-id="88975-128">Затем изображение обрабатывается с помощью функций Azure.</span><span class="sxs-lookup"><span data-stu-id="88975-128">The image is then processed with Azure functions.</span></span>

* <span data-ttu-id="88975-129">[Хранилище больших двоичных объектов][storage-docs] хранит все файлы изображений, которые отправляются в веб-приложение, а также все статические файлы, которые использует веб-приложение.</span><span class="sxs-lookup"><span data-stu-id="88975-129">[Blob Storage][storage-docs] stores all of the image files that are uploaded into the web application, as well any static files that the web application consumes.</span></span>

* <span data-ttu-id="88975-130">[Cosmos DB][cosmos-docs] хранит метаданные о каждом переданном изображении, включая результаты обработки API компьютерного зрения.</span><span class="sxs-lookup"><span data-stu-id="88975-130">[Cosmos DB][cosmos-docs] stores metadata about each image that is uploaded, including the results of the processing from Computer Vision API.</span></span>

## <a name="alternatives"></a><span data-ttu-id="88975-131">Альтернативные варианты</span><span class="sxs-lookup"><span data-stu-id="88975-131">Alternatives</span></span>

* <span data-ttu-id="88975-132">[Пользовательская служба визуального распознавания][custom-vision-docs].</span><span class="sxs-lookup"><span data-stu-id="88975-132">[Custom Vision Service][custom-vision-docs].</span></span> <span data-ttu-id="88975-133">API компьютерного зрения возвращает набор [таксономических категорий][cv-categories].</span><span class="sxs-lookup"><span data-stu-id="88975-133">The Computer Vision API returns a set of [taxonomy-based categories][cv-categories].</span></span> <span data-ttu-id="88975-134">Если вам необходимо обработать информацию, которую API компьютерного зрения не возвращает, рекомендуется использовать службу пользовательского визуального распознавания, которая позволяет создавать индивидуальные классификаторы изображений.</span><span class="sxs-lookup"><span data-stu-id="88975-134">If you need to process information that isn't returned by the Computer Vision API, consider the Custom Vision Service, which lets you build custom image classifiers.</span></span>

* <span data-ttu-id="88975-135">[Поиск Azure][azure-search-docs].</span><span class="sxs-lookup"><span data-stu-id="88975-135">[Azure Search][azure-search-docs].</span></span> <span data-ttu-id="88975-136">Если ваш вариант использования включает запросы метаданных для поиска изображений, соответствующих заданным критериям, рекомендуется использовать поиск Azure.</span><span class="sxs-lookup"><span data-stu-id="88975-136">If your use case involves querying the metadata to find images that meet specific criteria, consider using Azure Search.</span></span> <span data-ttu-id="88975-137">[Когнитивный поиск][cognitive-search] в предварительной версии сейчас эффективно интегрирует этот рабочий процесс.</span><span class="sxs-lookup"><span data-stu-id="88975-137">Currently in preview, [Cognitive search][cognitive-search] seamlessly integrates this workflow.</span></span>

## <a name="considerations"></a><span data-ttu-id="88975-138">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="88975-138">Considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="88975-139">Масштабируемость</span><span class="sxs-lookup"><span data-stu-id="88975-139">Scalability</span></span>

<span data-ttu-id="88975-140">Большинство компонентов в этом примере сценария являются управляемыми службами, которые будут автоматически масштабироваться.</span><span class="sxs-lookup"><span data-stu-id="88975-140">The majority of the components used in this example scenario are managed services that will automatically scale.</span></span> <span data-ttu-id="88975-141">Несколько важных исключений: функции Azure имеют ограничение до 200 экземпляров максимум.</span><span class="sxs-lookup"><span data-stu-id="88975-141">A couple notable exceptions: Azure Functions has a limit of a maximum of 200 instances.</span></span> <span data-ttu-id="88975-142">Если вам нужно выполнить дополнительное масштабирование, используйте несколько регионов или планов приложений.</span><span class="sxs-lookup"><span data-stu-id="88975-142">If you need to scale beyond this limit, consider multiple regions or app plans.</span></span>

<span data-ttu-id="88975-143">Cosmos DB не проводит автоматическое масштабирование в единицах запросов RUs.</span><span class="sxs-lookup"><span data-stu-id="88975-143">Cosmos DB doesn’t auto-scale in terms of provisioned request units (RUs).</span></span>  <span data-ttu-id="88975-144">Дополнительные сведения в руководстве по оценке требований см. в разделе [Единицы запроса в базе данных Azure Cosmos DB][request-units] нашей документации.</span><span class="sxs-lookup"><span data-stu-id="88975-144">For guidance on estimating your requirements see [request units][request-units] in our documentation.</span></span> <span data-ttu-id="88975-145">Чтобы использовать все преимущества масштабирования в Cosmos DB, см. описание [ключей секций][partition-key].</span><span class="sxs-lookup"><span data-stu-id="88975-145">To fully take advantage of the scaling in Cosmos DB, explore [partition keys][partition-key].</span></span>

<span data-ttu-id="88975-146">Базы данных NoSQL часто поддерживают согласованность (как в теореме CAP) для обеспечения доступности, масштабируемости и секционирования.</span><span class="sxs-lookup"><span data-stu-id="88975-146">NoSQL databases frequently trade consistency (in the sense of the CAP theorem) for availability, scalability, and partitioning.</span></span>  <span data-ttu-id="88975-147">В этом примере сценария используется модель данных с ключевыми значениями, а согласованность транзакций требуется редко, так как большинство операций по определению являются атомарными.</span><span class="sxs-lookup"><span data-stu-id="88975-147">In this example scenario, a key-value data model is used and transaction consistency is rarely needed as most operations are by definition atomic.</span></span> <span data-ttu-id="88975-148">Дополнительные рекомендации по [выбору правильного хранилища данных](../../guide/technology-choices/data-store-overview.md) см. в Центре архитектуры Azure.</span><span class="sxs-lookup"><span data-stu-id="88975-148">Additional guidance to [Choose the right data store](../../guide/technology-choices/data-store-overview.md) is available in the Azure Architecture Center.</span></span>

<span data-ttu-id="88975-149">Общие рекомендации по разработке масштабируемых решений см. в разделе [Контрольный список для обеспечения масштабируемости][scalability] в центре архитектуры Azure.</span><span class="sxs-lookup"><span data-stu-id="88975-149">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="88975-150">Безопасность</span><span class="sxs-lookup"><span data-stu-id="88975-150">Security</span></span>

<span data-ttu-id="88975-151">[Управляемое удостоверение службы][msi] (MSI) используется для предоставления доступа к другим ресурсам, внутренним вашей учетной записи, а затем присваивается функциям Azure.</span><span class="sxs-lookup"><span data-stu-id="88975-151">[Managed service identities][msi] (MSI) are used to provide access to other resources internal to your account and then assigned to your Azure Functions.</span></span> <span data-ttu-id="88975-152">В этих удостоверениях разрешайте доступ только к необходимым ресурсам, чтобы гарантировать, что ничто другое не подвергается воздействию ваших функций (и, возможно, ваших клиентов).</span><span class="sxs-lookup"><span data-stu-id="88975-152">Only allow access to the requisite resources in those identities to ensure that nothing extra is exposed to your functions (and potentially to your customers).</span></span>  

<span data-ttu-id="88975-153">Общие рекомендации по разработке безопасных решений см. в разделе [Azure Security Documentation][security].</span><span class="sxs-lookup"><span data-stu-id="88975-153">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="88975-154">Устойчивость</span><span class="sxs-lookup"><span data-stu-id="88975-154">Resiliency</span></span>

<span data-ttu-id="88975-155">Все компоненты этого сценария управляются, поэтому на региональном уровне все они автоматически являются устойчивыми.</span><span class="sxs-lookup"><span data-stu-id="88975-155">All of the components in this scenario are managed, so at a regional level they are all resilient automatically.</span></span>

<span data-ttu-id="88975-156">Общее руководство по проектированию устойчивых решений см. в разделе [Проектирование устойчивых приложений для Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="88975-156">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="88975-157">Цены</span><span class="sxs-lookup"><span data-stu-id="88975-157">Pricing</span></span>

<span data-ttu-id="88975-158">Чтобы изучить стоимость выполнения этого сценария, все услуги были предварительно сконфигурированы в калькуляторе стоимости.</span><span class="sxs-lookup"><span data-stu-id="88975-158">To explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="88975-159">Чтобы узнать, как изменится цена для вашего конкретного варианта использования, измените соответствующие переменные в соответствии с ожидаемым трафиком.</span><span class="sxs-lookup"><span data-stu-id="88975-159">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="88975-160">Доступны три примера профиля затрат на основе объема трафика (размер каждого изображения примерно 100 КБ).</span><span class="sxs-lookup"><span data-stu-id="88975-160">We have provided three sample cost profiles based on amount of traffic (we assume all images are 100 kb in size):</span></span>

* <span data-ttu-id="88975-161">[Небольшой][pricing]: этот пример тарификации предполагает обработку &lt; 5000 изображений в месяц.</span><span class="sxs-lookup"><span data-stu-id="88975-161">[Small][pricing]: this pricing example correlates to processing &lt; 5000 images a month.</span></span>
* <span data-ttu-id="88975-162">[Средний][medium-pricing]: этот пример тарификации предполагает обработку 500 000 изображений в месяц.</span><span class="sxs-lookup"><span data-stu-id="88975-162">[Medium][medium-pricing]: this pricing example correlates to processing 500,000 images a month.</span></span>
* <span data-ttu-id="88975-163">[Большой][large-pricing]: этот пример тарификации предполагает обработку 50 миллионов изображений в месяц.</span><span class="sxs-lookup"><span data-stu-id="88975-163">[Large][large-pricing]: this pricing example correlates to processing 50 million images a month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="88975-164">Связанные ресурсы</span><span class="sxs-lookup"><span data-stu-id="88975-164">Related Resources</span></span>

<span data-ttu-id="88975-165">Пошаговую схему обучения этого сценария см. в статье [Build a serverless web app in Azure][serverless] (Создание бессерверных приложений в Azure).</span><span class="sxs-lookup"><span data-stu-id="88975-165">For a guided learning path of this scenario, see [Build a serverless web app in Azure][serverless].</span></span>  

<span data-ttu-id="88975-166">Прежде чем поместить этот пример сценария в рабочую среду, ознакомьтесь с [рекомендациями][functions-best-practices] по использованию Функций Azure.</span><span class="sxs-lookup"><span data-stu-id="88975-166">Before deploying this example scenario in a production environment, review the Azure Functions [best practices][functions-best-practices].</span></span>

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