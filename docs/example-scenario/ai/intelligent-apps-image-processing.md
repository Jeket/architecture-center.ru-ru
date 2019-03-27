---
title: Классификация изображений для страховых требований
titleSuffix: Azure Example Scenarios
description: Создание сценария обработки изображений в приложениях Azure.
author: david-stanford
ms.date: 07/05/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
social_image_url: /azure/architecture/example-scenario/ai/media/architecture-intelligent-apps-image-processing.png
ms.openlocfilehash: 03ef9d15ec9bf64dc743657e9d1e7a7e275c5a8e
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58244805"
---
# <a name="image-classification-for-insurance-claims-on-azure"></a><span data-ttu-id="ff91a-103">Классификация изображений в Azure для страховых требований</span><span class="sxs-lookup"><span data-stu-id="ff91a-103">Image classification for insurance claims on Azure</span></span>

<span data-ttu-id="ff91a-104">Этот сценарий подходит для корпораций, которым нужно обрабатывать изображения.</span><span class="sxs-lookup"><span data-stu-id="ff91a-104">This scenario is relevant for businesses that need to process images.</span></span>

<span data-ttu-id="ff91a-105">Потенциальные приложения включают классификацию изображений для веб-сайтов о моде, анализ текста и изображений для страховых компаний или распознавание телеметрических данных со скриншотов игр.</span><span class="sxs-lookup"><span data-stu-id="ff91a-105">Potential applications include classifying images for a fashion website, analyzing text and images for insurance claims, or understanding telemetry data from game screenshots.</span></span> <span data-ttu-id="ff91a-106">Обычно компаниям было необходимо приобретать специальные знания о моделях машинного обучения, обучать эти модели и, наконец, пропускать изображения через индивидуальный процесс, чтобы получить выходные данные.</span><span class="sxs-lookup"><span data-stu-id="ff91a-106">Traditionally, companies would need to develop expertise in machine learning models, train the models, and finally run the images through their custom process to get the data out of the images.</span></span>

<span data-ttu-id="ff91a-107">Используя такие службы Azure, как API компьютерного зрения и Функции Azure, компании могут избавиться от необходимости управлять отдельными серверами, одновременно сокращая затраты и используя опыт корпорации Майкрософт в обработке изображений с помощью служб Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="ff91a-107">By using Azure services such as the Computer Vision API and Azure Functions, companies can eliminate the need to manage individual servers, while reducing costs and leveraging the expertise that Microsoft has already developed around processing images with Cognitive Services.</span></span> <span data-ttu-id="ff91a-108">Этот пример сценария предусматривает использование обработки изображений.</span><span class="sxs-lookup"><span data-stu-id="ff91a-108">This example scenario specifically addresses an image-processing use case.</span></span> <span data-ttu-id="ff91a-109">При наличии иных потребностей в искусственном интеллекте можно использовать полный набор [Cognitive Services](/azure/#pivot=products&panel=ai).</span><span class="sxs-lookup"><span data-stu-id="ff91a-109">If you have different AI needs, consider the full suite of [Cognitive Services](/azure/#pivot=products&panel=ai).</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="ff91a-110">Варианты соответствующего использования</span><span class="sxs-lookup"><span data-stu-id="ff91a-110">Relevant use cases</span></span>

<span data-ttu-id="ff91a-111">Другие варианты использования:</span><span class="sxs-lookup"><span data-stu-id="ff91a-111">Other relevant use cases include:</span></span>

- <span data-ttu-id="ff91a-112">Классификация изображений на веб-сайте о моде.</span><span class="sxs-lookup"><span data-stu-id="ff91a-112">Classifying images on a fashion website.</span></span>
- <span data-ttu-id="ff91a-113">Классификация телеметрических данных со скриншотов игр.</span><span class="sxs-lookup"><span data-stu-id="ff91a-113">Classifying telemetry data from screenshots of games.</span></span>

## <a name="architecture"></a><span data-ttu-id="ff91a-114">Архитектура</span><span class="sxs-lookup"><span data-stu-id="ff91a-114">Architecture</span></span>

![Архитектура для классификации изображений][architecture]

<span data-ttu-id="ff91a-116">Этот сценарий охватывает серверные компоненты веб-приложения или мобильного приложения.</span><span class="sxs-lookup"><span data-stu-id="ff91a-116">This scenario covers the back-end components of a web or mobile application.</span></span> <span data-ttu-id="ff91a-117">Данные передаются в сценарии следующим образом:</span><span class="sxs-lookup"><span data-stu-id="ff91a-117">Data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="ff91a-118">Слой API создается с помощью Функций Azure.</span><span class="sxs-lookup"><span data-stu-id="ff91a-118">The API layer is built using Azure Functions.</span></span> <span data-ttu-id="ff91a-119">Эти API позволяют приложению передавать изображения и получать данные из Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="ff91a-119">These APIs enable the application to upload images and retrieve data from Cosmos DB.</span></span>
2. <span data-ttu-id="ff91a-120">После загрузки изображения через вызов API, оно сохраняется в хранилище больших двоичных объектов.</span><span class="sxs-lookup"><span data-stu-id="ff91a-120">When an image is uploaded via an API call, it's stored in Blob storage.</span></span>
3. <span data-ttu-id="ff91a-121">Добавление новых файлов в хранилище больших двоичных объектов активирует отправку уведомления Сетки событий в Функцию Azure.</span><span class="sxs-lookup"><span data-stu-id="ff91a-121">Adding new files to Blob storage triggers an Event Grid notification to be sent to an Azure Function.</span></span>
4. <span data-ttu-id="ff91a-122">Функции Azure отправляют API компьютерного зрения ссылку на новый загруженный файл для анализа.</span><span class="sxs-lookup"><span data-stu-id="ff91a-122">Azure Functions sends a link to the newly uploaded file to the Computer Vision API to analyze.</span></span>
5. <span data-ttu-id="ff91a-123">После возвращения данных из API компьютерного зрения Функции Azure создают запись в базе данных Cosmos DB, чтобы сохранить результаты анализа вместе с метаданными изображения.</span><span class="sxs-lookup"><span data-stu-id="ff91a-123">Once the data has been returned from the Computer Vision API, Azure Functions makes an entry in Cosmos DB to persist the results of the analysis along with the image metadata.</span></span>

### <a name="components"></a><span data-ttu-id="ff91a-124">Компоненты</span><span class="sxs-lookup"><span data-stu-id="ff91a-124">Components</span></span>

- <span data-ttu-id="ff91a-125">[API компьютерного зрения](/azure/cognitive-services/computer-vision/home) — это часть набора Cognitive Services, которая используется для получения сведений о каждом изображении.</span><span class="sxs-lookup"><span data-stu-id="ff91a-125">[Computer Vision API](/azure/cognitive-services/computer-vision/home) is part of the Cognitive Services suite and is used to retrieve information about each image.</span></span>
- <span data-ttu-id="ff91a-126">[Функции Azure](/azure/azure-functions/functions-overview) предоставляют серверный API для веб-приложения, а также обработку событий для переданных изображений.</span><span class="sxs-lookup"><span data-stu-id="ff91a-126">[Azure Functions](/azure/azure-functions/functions-overview) provides the back-end API for the web application, as well as the event processing for uploaded images.</span></span>
- <span data-ttu-id="ff91a-127">[Сетка событий](/azure/event-grid/overview) активирует событие при отправке нового изображения в хранилище больших двоичных объектов.</span><span class="sxs-lookup"><span data-stu-id="ff91a-127">[Event Grid](/azure/event-grid/overview) triggers an event when a new image is uploaded to blob storage.</span></span> <span data-ttu-id="ff91a-128">Затем изображение обрабатывается с помощью функций Azure.</span><span class="sxs-lookup"><span data-stu-id="ff91a-128">The image is then processed with Azure functions.</span></span>
- <span data-ttu-id="ff91a-129">[Хранилище больших двоичных объектов](/azure/storage/blobs/storage-blobs-introduction) хранит все файлы изображений, которые отправляются в веб-приложение, а также все статические файлы, которые используются веб-приложением.</span><span class="sxs-lookup"><span data-stu-id="ff91a-129">[Blob storage](/azure/storage/blobs/storage-blobs-introduction) stores all of the image files that are uploaded into the web application, as well any static files that the web application consumes.</span></span>
- <span data-ttu-id="ff91a-130">[Cosmos DB](/azure/cosmos-db/introduction) хранит метаданные о каждом переданном изображении, включая результаты обработки API компьютерного зрения.</span><span class="sxs-lookup"><span data-stu-id="ff91a-130">[Cosmos DB](/azure/cosmos-db/introduction) stores metadata about each image that is uploaded, including the results of the processing from Computer Vision API.</span></span>

## <a name="alternatives"></a><span data-ttu-id="ff91a-131">Альтернативные варианты</span><span class="sxs-lookup"><span data-stu-id="ff91a-131">Alternatives</span></span>

- <span data-ttu-id="ff91a-132">[Пользовательская служба визуального распознавания](/azure/cognitive-services/custom-vision-service/home).</span><span class="sxs-lookup"><span data-stu-id="ff91a-132">[Custom Vision Service](/azure/cognitive-services/custom-vision-service/home).</span></span> <span data-ttu-id="ff91a-133">API компьютерного зрения возвращает набор [таксономических категорий][cv-categories].</span><span class="sxs-lookup"><span data-stu-id="ff91a-133">The Computer Vision API returns a set of [taxonomy-based categories][cv-categories].</span></span> <span data-ttu-id="ff91a-134">Если вам необходимо обработать информацию, которую API компьютерного зрения не возвращает, рекомендуется использовать службу пользовательского визуального распознавания, которая позволяет создавать индивидуальные классификаторы изображений.</span><span class="sxs-lookup"><span data-stu-id="ff91a-134">If you need to process information that isn't returned by the Computer Vision API, consider the Custom Vision Service, which lets you build custom image classifiers.</span></span>
- <span data-ttu-id="ff91a-135">[Поиск Azure.](/azure/search/search-what-is-azure-search)</span><span class="sxs-lookup"><span data-stu-id="ff91a-135">[Azure Search](/azure/search/search-what-is-azure-search).</span></span> <span data-ttu-id="ff91a-136">Если ваш вариант использования включает запросы метаданных для поиска изображений, соответствующих заданным критериям, рекомендуется использовать поиск Azure.</span><span class="sxs-lookup"><span data-stu-id="ff91a-136">If your use case involves querying the metadata to find images that meet specific criteria, consider using Azure Search.</span></span> <span data-ttu-id="ff91a-137">[Когнитивный поиск](/azure/search/cognitive-search-concept-intro), находящийся в предварительной версии, эффективно интегрирует этот рабочий процесс.</span><span class="sxs-lookup"><span data-stu-id="ff91a-137">Currently in preview, [Cognitive search](/azure/search/cognitive-search-concept-intro) seamlessly integrates this workflow.</span></span>

## <a name="considerations"></a><span data-ttu-id="ff91a-138">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="ff91a-138">Considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="ff91a-139">Масштабируемость</span><span class="sxs-lookup"><span data-stu-id="ff91a-139">Scalability</span></span>

<span data-ttu-id="ff91a-140">Большинство компонентов в этом примере сценария являются управляемыми службами, которые будут автоматически масштабироваться.</span><span class="sxs-lookup"><span data-stu-id="ff91a-140">The majority of the components used in this example scenario are managed services that will automatically scale.</span></span> <span data-ttu-id="ff91a-141">Несколько важных исключений: Функции Azure имеют ограничение до 200 экземпляров максимум.</span><span class="sxs-lookup"><span data-stu-id="ff91a-141">A couple notable exceptions: Azure Functions has a limit of a maximum of 200 instances.</span></span> <span data-ttu-id="ff91a-142">Если вам нужно выполнить дополнительное масштабирование, используйте несколько регионов или планов приложений.</span><span class="sxs-lookup"><span data-stu-id="ff91a-142">If you need to scale beyond this limit, consider multiple regions or app plans.</span></span>

<span data-ttu-id="ff91a-143">Cosmos DB не поддерживает автоматическое масштабирование на основе единиц запросов.</span><span class="sxs-lookup"><span data-stu-id="ff91a-143">Cosmos DB doesn’t autoscale in terms of provisioned request units (RUs).</span></span> <span data-ttu-id="ff91a-144">Дополнительные сведения об оценке требований см. в разделе о [единицах запроса](/azure/cosmos-db/request-units) в нашей документации.</span><span class="sxs-lookup"><span data-stu-id="ff91a-144">For guidance on estimating your requirements see [request units](/azure/cosmos-db/request-units) in our documentation.</span></span> <span data-ttu-id="ff91a-145">Чтобы полностью воспользоваться преимуществами масштабирования в Cosmos DB, ознакомьтесь с принципами работы [ключей секции](/azure/cosmos-db/partition-data) в CosmosDB.</span><span class="sxs-lookup"><span data-stu-id="ff91a-145">To fully take advantage of the scaling in Cosmos DB, understand how [partition keys](/azure/cosmos-db/partition-data) work in CosmosDB.</span></span>

<span data-ttu-id="ff91a-146">Базы данных NoSQL часто поддерживают согласованность (как в теореме CAP) для обеспечения доступности, масштабируемости и секционирования.</span><span class="sxs-lookup"><span data-stu-id="ff91a-146">NoSQL databases frequently trade consistency (in the sense of the CAP theorem) for availability, scalability, and partitioning.</span></span> <span data-ttu-id="ff91a-147">В этом примере сценария используется модель данных с ключевыми значениями, а согласованность транзакций требуется редко, так как большинство операций по определению являются атомарными.</span><span class="sxs-lookup"><span data-stu-id="ff91a-147">In this example scenario, a key-value data model is used and transaction consistency is rarely needed as most operations are by definition atomic.</span></span> <span data-ttu-id="ff91a-148">Дополнительные рекомендации по [выбору правильного хранилища данных](../../guide/technology-choices/data-store-overview.md) см. в Центре архитектуры Azure.</span><span class="sxs-lookup"><span data-stu-id="ff91a-148">Additional guidance to [Choose the right data store](../../guide/technology-choices/data-store-overview.md) is available in the Azure Architecture Center.</span></span> <span data-ttu-id="ff91a-149">Если реализация требует высокого уровня согласованности, в Cosmos DB можно [выбрать собственный уровень согласованности](/azure/cosmos-db/consistency-levels).</span><span class="sxs-lookup"><span data-stu-id="ff91a-149">If your implementation requires high consistency, you can [choose your consistency level](/azure/cosmos-db/consistency-levels) in CosmosDB.</span></span>

<span data-ttu-id="ff91a-150">Общие рекомендации по разработке масштабируемых решений см. в разделе [Контрольный список для обеспечения масштабируемости][scalability] в центре архитектуры Azure.</span><span class="sxs-lookup"><span data-stu-id="ff91a-150">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="ff91a-151">Безопасность</span><span class="sxs-lookup"><span data-stu-id="ff91a-151">Security</span></span>

<span data-ttu-id="ff91a-152">[Управляемые удостоверения Служб Azure][msi] используются для предоставления доступа к другим ресурсам, находящимся внутри учетной записи, а затем присвоенным Функциям Azure.</span><span class="sxs-lookup"><span data-stu-id="ff91a-152">[Managed identities for Azure resources][msi] are used to provide access to other resources internal to your account and then assigned to your Azure Functions.</span></span> <span data-ttu-id="ff91a-153">В этих удостоверениях разрешайте доступ только к необходимым ресурсам, чтобы гарантировать, что ничто другое не подвергается воздействию ваших функций (и, возможно, ваших клиентов).</span><span class="sxs-lookup"><span data-stu-id="ff91a-153">Only allow access to the requisite resources in those identities to ensure that nothing extra is exposed to your functions (and potentially to your customers).</span></span>

<span data-ttu-id="ff91a-154">Общие рекомендации по разработке безопасных решений см. в разделе [Azure Security Documentation][security].</span><span class="sxs-lookup"><span data-stu-id="ff91a-154">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="ff91a-155">Устойчивость</span><span class="sxs-lookup"><span data-stu-id="ff91a-155">Resiliency</span></span>

<span data-ttu-id="ff91a-156">Все компоненты этого сценария управляются, поэтому на региональном уровне все они автоматически являются устойчивыми.</span><span class="sxs-lookup"><span data-stu-id="ff91a-156">All of the components in this scenario are managed, so at a regional level they are all resilient automatically.</span></span>

<span data-ttu-id="ff91a-157">Общее руководство по проектированию устойчивых решений см. в разделе [Проектирование устойчивых приложений для Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="ff91a-157">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="ff91a-158">Цены</span><span class="sxs-lookup"><span data-stu-id="ff91a-158">Pricing</span></span>

<span data-ttu-id="ff91a-159">Чтобы изучить стоимость выполнения этого сценария, все услуги были предварительно сконфигурированы в калькуляторе стоимости.</span><span class="sxs-lookup"><span data-stu-id="ff91a-159">To explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="ff91a-160">Чтобы узнать, как изменится цена для вашего конкретного варианта использования, измените соответствующие переменные в соответствии с ожидаемым трафиком.</span><span class="sxs-lookup"><span data-stu-id="ff91a-160">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="ff91a-161">Доступны три примера профиля затрат на основе объема трафика (размер каждого изображения примерно 100 КБ).</span><span class="sxs-lookup"><span data-stu-id="ff91a-161">We have provided three sample cost profiles based on amount of traffic (we assume all images are 100 kb in size):</span></span>

- <span data-ttu-id="ff91a-162">[Небольшой][small-pricing]: этот пример тарификации предполагает обработку &lt; 5000 изображений в месяц.</span><span class="sxs-lookup"><span data-stu-id="ff91a-162">[Small][small-pricing]: this pricing example correlates to processing &lt; 5000 images a month.</span></span>
- <span data-ttu-id="ff91a-163">[Средний][medium-pricing]: этот пример тарификации предполагает обработку 500 000 изображений в месяц.</span><span class="sxs-lookup"><span data-stu-id="ff91a-163">[Medium][medium-pricing]: this pricing example correlates to processing 500,000 images a month.</span></span>
- <span data-ttu-id="ff91a-164">[Большой][large-pricing]: этот пример тарификации предполагает обработку 50 миллионов изображений в месяц.</span><span class="sxs-lookup"><span data-stu-id="ff91a-164">[Large][large-pricing]: this pricing example correlates to processing 50 million images a month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="ff91a-165">Связанные ресурсы</span><span class="sxs-lookup"><span data-stu-id="ff91a-165">Related resources</span></span>

<span data-ttu-id="ff91a-166">Пошаговую схему обучения см. в руководстве по [созданию бессерверных приложений в Azure][serverless].</span><span class="sxs-lookup"><span data-stu-id="ff91a-166">For a guided learning path, see [Build a serverless web app in Azure][serverless].</span></span>

<span data-ttu-id="ff91a-167">Прежде чем поместить этот пример сценария в рабочую среду, ознакомьтесь с рекомендациями по [оптимизации производительности и надежности Функций Azure][functions-best-practices].</span><span class="sxs-lookup"><span data-stu-id="ff91a-167">Before deploying this example scenario in a production environment, review recommended practices for [optimizing the performance and reliability of Azure Functions][functions-best-practices].</span></span>

<!-- links -->
[architecture]: ./media/architecture-intelligent-apps-image-processing.png
[small-pricing]: https://azure.com/e/f9b59d238b43423683db73f4a31dc380
[medium-pricing]: https://azure.com/e/7c7fc474db344b87aae93bc29ae27108
[large-pricing]: https://azure.com/e/cbadbca30f8640d6a061f8457a74ba7d
[cognitive-search]: /azure/search/cognitive-search-concept-intro
[serverless]: /azure/functions/tutorial-static-website-serverless-api-with-database
[cv-categories]: /azure/cognitive-services/computer-vision/home#the-86-category-concept
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[functions-best-practices]: /azure/azure-functions/functions-best-practices
[msi]: /azure/app-service/app-service-managed-service-identity
