---
title: Стиль архитектуры "Интерфейс — очередь — рабочая роль"
titleSuffix: Azure Application Architecture Guide
description: В статье описаны преимущества и сложности архитектуры "Интерфейс — очередь — рабочая роль" в Azure, а также приведены рекомендации по ее разработке.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: b471d270af09df7ffd58dfdd49e7d03d05bfe582
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58244595"
---
# <a name="web-queue-worker-architecture-style"></a><span data-ttu-id="82d33-103">Стиль архитектуры "Интерфейс — очередь — рабочая роль"</span><span class="sxs-lookup"><span data-stu-id="82d33-103">Web-Queue-Worker architecture style</span></span>

<span data-ttu-id="82d33-104">Основными компонентами этой архитектуры являются **веб-интерфейс** для обслуживания клиентских запросов и **рабочая роль** для выполнения ресурсоемких задач, долгосрочных рабочих процессов или пакетных заданий.</span><span class="sxs-lookup"><span data-stu-id="82d33-104">The core components of this architecture are a **web front end** that serves client requests, and a **worker** that performs resource-intensive tasks, long-running workflows, or batch jobs.</span></span>  <span data-ttu-id="82d33-105">Веб-интерфейс взаимодействует с рабочей ролью через **очередь сообщений**.</span><span class="sxs-lookup"><span data-stu-id="82d33-105">The web front end communicates with the worker through a **message queue**.</span></span>

![Логическая схема архитектуры "Интерфейс — очередь — рабочая роль"](./images/web-queue-worker-logical.svg)

<span data-ttu-id="82d33-107">Также в эту архитектуру часто включают другие компоненты, в том числе:</span><span class="sxs-lookup"><span data-stu-id="82d33-107">Other components that are commonly incorporated into this architecture include:</span></span>

- <span data-ttu-id="82d33-108">одну или несколько баз данных;</span><span class="sxs-lookup"><span data-stu-id="82d33-108">One or more databases.</span></span>
- <span data-ttu-id="82d33-109">кэш для хранения значений из базы данных для быстрого чтения;</span><span class="sxs-lookup"><span data-stu-id="82d33-109">A cache to store values from the database for quick reads.</span></span>
- <span data-ttu-id="82d33-110">сеть доставки содержимого, которая обслуживает статическое содержимое;</span><span class="sxs-lookup"><span data-stu-id="82d33-110">CDN to serve static content</span></span>
- <span data-ttu-id="82d33-111">удаленные службы, например служба электронной почты или текстовых сообщений</span><span class="sxs-lookup"><span data-stu-id="82d33-111">Remote services, such as email or SMS service.</span></span> <span data-ttu-id="82d33-112">(часто это службы сторонних производителей);</span><span class="sxs-lookup"><span data-stu-id="82d33-112">Often these are provided by third parties.</span></span>
- <span data-ttu-id="82d33-113">поставщик удостоверений для аутентификации.</span><span class="sxs-lookup"><span data-stu-id="82d33-113">Identity provider for authentication.</span></span>

<span data-ttu-id="82d33-114">Веб-интерфейс и рабочая роль реализованы без отслеживания состояния.</span><span class="sxs-lookup"><span data-stu-id="82d33-114">The web and worker are both stateless.</span></span> <span data-ttu-id="82d33-115">Сведения о состоянии сеанса можно сохранять в распределенном кэше.</span><span class="sxs-lookup"><span data-stu-id="82d33-115">Session state can be stored in a distributed cache.</span></span> <span data-ttu-id="82d33-116">Любые длительные задачи выполняются рабочей ролью в асинхронном режиме.</span><span class="sxs-lookup"><span data-stu-id="82d33-116">Any long-running work is done asynchronously by the worker.</span></span> <span data-ttu-id="82d33-117">Рабочая роль запускается при помещении сообщений в очередь или по расписанию для пакетной обработки.</span><span class="sxs-lookup"><span data-stu-id="82d33-117">The worker can be triggered by messages on the queue, or run on a schedule for batch processing.</span></span> <span data-ttu-id="82d33-118">Рабочая роль используется не всегда.</span><span class="sxs-lookup"><span data-stu-id="82d33-118">The worker is an optional component.</span></span> <span data-ttu-id="82d33-119">Если длительные операции не требуются, ее можно не вызывать.</span><span class="sxs-lookup"><span data-stu-id="82d33-119">If there are no long-running operations, the worker can be omitted.</span></span>

<span data-ttu-id="82d33-120">Внешний интерфейс может быть реализован как веб-API.</span><span class="sxs-lookup"><span data-stu-id="82d33-120">The front end might consist of a web API.</span></span> <span data-ttu-id="82d33-121">На стороне клиента веб-API представлен в виде одностраничного приложения, в котором выполняются вызовы через AJAX, или собственного клиентского приложения.</span><span class="sxs-lookup"><span data-stu-id="82d33-121">On the client side, the web API can be consumed by a single-page application that makes AJAX calls, or by a native client application.</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="82d33-122">Когда следует использовать эту архитектуру</span><span class="sxs-lookup"><span data-stu-id="82d33-122">When to use this architecture</span></span>

<span data-ttu-id="82d33-123">Архитектура "Интерфейс — очередь — рабочая роль" обычно реализуется на основе управляемых вычислительных служб, таких как служба приложений Azure или облачные службы Azure.</span><span class="sxs-lookup"><span data-stu-id="82d33-123">The Web-Queue-Worker architecture is typically implemented using managed compute services, either Azure App Service or Azure Cloud Services.</span></span>

<span data-ttu-id="82d33-124">Используйте эту архитектуру для следующих сценариев:</span><span class="sxs-lookup"><span data-stu-id="82d33-124">Consider this architecture style for:</span></span>

- <span data-ttu-id="82d33-125">приложения с относительно несложными функциями;</span><span class="sxs-lookup"><span data-stu-id="82d33-125">Applications with a relatively simple domain.</span></span>
- <span data-ttu-id="82d33-126">приложения, в которых выполняются долгосрочные процессы или пакетные операции;</span><span class="sxs-lookup"><span data-stu-id="82d33-126">Applications with some long-running workflows or batch operations.</span></span>
- <span data-ttu-id="82d33-127">если нужно применить управляемые службы вместо решений IaaS (инфраструктура как услуга).</span><span class="sxs-lookup"><span data-stu-id="82d33-127">When you want to use managed services, rather than infrastructure as a service (IaaS).</span></span>

## <a name="benefits"></a><span data-ttu-id="82d33-128">Преимущества</span><span class="sxs-lookup"><span data-stu-id="82d33-128">Benefits</span></span>

- <span data-ttu-id="82d33-129">Это несложная и удобная архитектура.</span><span class="sxs-lookup"><span data-stu-id="82d33-129">Relatively simple architecture that is easy to understand.</span></span>
- <span data-ttu-id="82d33-130">Она проста в развертывании и управлении.</span><span class="sxs-lookup"><span data-stu-id="82d33-130">Easy to deploy and manage.</span></span>
- <span data-ttu-id="82d33-131">Четко разделены зоны ответственности.</span><span class="sxs-lookup"><span data-stu-id="82d33-131">Clear separation of concerns.</span></span>
- <span data-ttu-id="82d33-132">Внешний интерфейс отделен от рабочей роли благодаря механизму асинхронного обмена сообщениями.</span><span class="sxs-lookup"><span data-stu-id="82d33-132">The front end is decoupled from the worker using asynchronous messaging.</span></span>
- <span data-ttu-id="82d33-133">Внешний интерфейс и рабочую роль можно масштабировать независимо друг от друга.</span><span class="sxs-lookup"><span data-stu-id="82d33-133">The front end and the worker can be scaled independently.</span></span>

## <a name="challenges"></a><span data-ttu-id="82d33-134">Сложности</span><span class="sxs-lookup"><span data-stu-id="82d33-134">Challenges</span></span>

- <span data-ttu-id="82d33-135">К разработке нужно подходить внимательно, иначе внешний интерфейс и (или) рабочая роль могут стать громоздкими и монолитными компонентами, которые трудно обслуживать и обновлять.</span><span class="sxs-lookup"><span data-stu-id="82d33-135">Without careful design, the front end and the worker can become large, monolithic components that are difficult to maintain and update.</span></span>
- <span data-ttu-id="82d33-136">Если во внешнем интерфейсе и рабочей роли используются общие схемы данных или модули, могут возникнуть скрытые зависимости.</span><span class="sxs-lookup"><span data-stu-id="82d33-136">There may be hidden dependencies, if the front end and worker share data schemas or code modules.</span></span>

## <a name="best-practices"></a><span data-ttu-id="82d33-137">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="82d33-137">Best practices</span></span>

- <span data-ttu-id="82d33-138">Предоставлять клиенту хорошо спроектированный API-интерфейс.</span><span class="sxs-lookup"><span data-stu-id="82d33-138">Expose a well-designed API to the client.</span></span> <span data-ttu-id="82d33-139">См. [рекомендации по проектированию API][api-design].</span><span class="sxs-lookup"><span data-stu-id="82d33-139">See [API design best practices][api-design].</span></span>
- <span data-ttu-id="82d33-140">Автоматически масштабировать систему с учетом изменений нагрузки.</span><span class="sxs-lookup"><span data-stu-id="82d33-140">Autoscale to handle changes in load.</span></span> <span data-ttu-id="82d33-141">См. [рекомендации по автомасштабированию][autoscaling].</span><span class="sxs-lookup"><span data-stu-id="82d33-141">See [Autoscaling best practices][autoscaling].</span></span>
- <span data-ttu-id="82d33-142">Кэшировать данные с признаками статических.</span><span class="sxs-lookup"><span data-stu-id="82d33-142">Cache semi-static data.</span></span> <span data-ttu-id="82d33-143">См. [рекомендации по кэшированию][caching].</span><span class="sxs-lookup"><span data-stu-id="82d33-143">See [Caching best practices][caching].</span></span>
- <span data-ttu-id="82d33-144">Использовать сеть доставки содержимого для размещения статического содержимого.</span><span class="sxs-lookup"><span data-stu-id="82d33-144">Use a CDN to host static content.</span></span> <span data-ttu-id="82d33-145">См. [рекомендации по использованию CDN][cdn].</span><span class="sxs-lookup"><span data-stu-id="82d33-145">See [CDN best practices][cdn].</span></span>
- <span data-ttu-id="82d33-146">При возможности использовать Polyglot Persistence.</span><span class="sxs-lookup"><span data-stu-id="82d33-146">Use polyglot persistence when appropriate.</span></span> <span data-ttu-id="82d33-147">См. статью об [использовании подходящего хранилища данных для задания][polyglot].</span><span class="sxs-lookup"><span data-stu-id="82d33-147">See [Use the best data store for the job][polyglot].</span></span>
- <span data-ttu-id="82d33-148">Секционировать данные, чтобы улучшить масштабируемость, уменьшить количество конфликтов и оптимизировать производительность.</span><span class="sxs-lookup"><span data-stu-id="82d33-148">Partition data to improve scalability, reduce contention, and optimize performance.</span></span> <span data-ttu-id="82d33-149">См. [рекомендации по секционированию данных][data-partition].</span><span class="sxs-lookup"><span data-stu-id="82d33-149">See [Data partitioning best practices][data-partition].</span></span>

## <a name="web-queue-worker-on-azure-app-service"></a><span data-ttu-id="82d33-150">Модель "Интерфейс — очередь — рабочая роль" в службе приложений Azure</span><span class="sxs-lookup"><span data-stu-id="82d33-150">Web-Queue-Worker on Azure App Service</span></span>

<span data-ttu-id="82d33-151">В этом разделе описана рекомендуемая архитектура "Интерфейс — очередь — рабочая роль" для службы приложений Azure.</span><span class="sxs-lookup"><span data-stu-id="82d33-151">This section describes a recommended Web-Queue-Worker architecture that uses Azure App Service.</span></span>

![Физическая схема архитектуры "Интерфейс — очередь — рабочая роль"](./images/web-queue-worker-physical.png)

<span data-ttu-id="82d33-153">Внешний интерфейс реализуется в виде веб-приложения службы приложений Azure, а рабочая роль — как веб-задание.</span><span class="sxs-lookup"><span data-stu-id="82d33-153">The front end is implemented as an Azure App Service web app, and the worker is implemented as a WebJob.</span></span> <span data-ttu-id="82d33-154">Эти веб-приложение и веб-задание сопоставляются с планом службы приложений, который предоставляет экземпляры виртуальных машин.</span><span class="sxs-lookup"><span data-stu-id="82d33-154">The web app and the WebJob are both associated with an App Service plan that provides the VM instances.</span></span>

<span data-ttu-id="82d33-155">В качестве очереди сообщений можно применить очередь служебной шины Azure или очередь службы хранилища Azure.</span><span class="sxs-lookup"><span data-stu-id="82d33-155">You can use either Azure Service Bus or Azure Storage queues for the message queue.</span></span> <span data-ttu-id="82d33-156">(На схеме представлена очередь службы хранилища Azure.)</span><span class="sxs-lookup"><span data-stu-id="82d33-156">(The diagram shows an Azure Storage queue.)</span></span>

<span data-ttu-id="82d33-157">В кэше Redis для Azure хранятся сведения о состоянии сеанса и другие данные, к которым нужен доступ с низкой задержкой.</span><span class="sxs-lookup"><span data-stu-id="82d33-157">Azure Redis Cache stores session state and other data that needs low latency access.</span></span>

<span data-ttu-id="82d33-158">Azure CDN кэширует статическое содержимое, например изображения, документы HTML и CSS.</span><span class="sxs-lookup"><span data-stu-id="82d33-158">Azure CDN is used to cache static content such as images, CSS, or HTML.</span></span>

<span data-ttu-id="82d33-159">Для хранения данных вы можете выбрать любую технологию, соответствующую требованиям приложения.</span><span class="sxs-lookup"><span data-stu-id="82d33-159">For storage, choose the storage technologies that best fit the needs of the application.</span></span> <span data-ttu-id="82d33-160">Можно одновременно использовать несколько технологий хранения данных (Polyglot Persistence).</span><span class="sxs-lookup"><span data-stu-id="82d33-160">You might use multiple storage technologies (polyglot persistence).</span></span> <span data-ttu-id="82d33-161">Эта концепция отражена на схеме в виде блоков службы "База данных SQL Azure" и Azure Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="82d33-161">To illustrate this idea, the diagram shows Azure SQL Database and Azure Cosmos DB.</span></span>

<span data-ttu-id="82d33-162">Дополнительные сведения см. в [описании базовой архитектуры для веб-приложения в службе приложений][scalable-web-app].</span><span class="sxs-lookup"><span data-stu-id="82d33-162">For more details, see [App Service web application reference architecture][scalable-web-app].</span></span>

### <a name="additional-considerations"></a><span data-ttu-id="82d33-163">Дополнительные замечания</span><span class="sxs-lookup"><span data-stu-id="82d33-163">Additional considerations</span></span>

- <span data-ttu-id="82d33-164">Не все транзакции передаются в хранилище через очередь и рабочую роль.</span><span class="sxs-lookup"><span data-stu-id="82d33-164">Not every transaction has to go through the queue and worker to storage.</span></span> <span data-ttu-id="82d33-165">Простые операции чтения и (или) записи веб-интерфейс может выполнять напрямую.</span><span class="sxs-lookup"><span data-stu-id="82d33-165">The web front end can perform simple read/write operations directly.</span></span> <span data-ttu-id="82d33-166">Рабочие роли предназначены только для ресурсоемких задач или долго выполняющихся рабочих процессов.</span><span class="sxs-lookup"><span data-stu-id="82d33-166">Workers are designed for resource-intensive tasks or long-running workflows.</span></span> <span data-ttu-id="82d33-167">В некоторых сценариях рабочая роль не нужна.</span><span class="sxs-lookup"><span data-stu-id="82d33-167">In some cases, you might not need a worker at all.</span></span>

- <span data-ttu-id="82d33-168">Встроенная в службу приложений функция автоматического масштабирования позволяет изменять количество экземпляров виртуальных машин.</span><span class="sxs-lookup"><span data-stu-id="82d33-168">Use the built-in autoscale feature of App Service to scale out the number of VM instances.</span></span> <span data-ttu-id="82d33-169">При четко прогнозируемой нагрузке на приложение используйте автоматическое масштабирование на основе расписания.</span><span class="sxs-lookup"><span data-stu-id="82d33-169">If the load on the application follows predictable patterns, use schedule-based autoscale.</span></span> <span data-ttu-id="82d33-170">Если нагрузка непредсказуема, используйте правила автоматического масштабирования на основе метрик.</span><span class="sxs-lookup"><span data-stu-id="82d33-170">If the load is unpredictable, use metrics-based autoscaling rules.</span></span>

- <span data-ttu-id="82d33-171">Возможно, веб-приложение и веб-задание стоит разместить в разных планах службы приложений.</span><span class="sxs-lookup"><span data-stu-id="82d33-171">Consider putting the web app and the WebJob into separate App Service plans.</span></span> <span data-ttu-id="82d33-172">Тогда они будут выполняться на разных экземплярах виртуальных машин и вы сможете масштабировать их независимо друг от друга.</span><span class="sxs-lookup"><span data-stu-id="82d33-172">That way, they are hosted on separate VM instances and can be scaled independently.</span></span>

- <span data-ttu-id="82d33-173">Используйте разные планы службы приложений для рабочей и тестовой сред.</span><span class="sxs-lookup"><span data-stu-id="82d33-173">Use separate App Service plans for production and testing.</span></span> <span data-ttu-id="82d33-174">Если у вас будет общий план для рабочей среды и тестирования, все задачи тестирования будут выполняться прямо на виртуальных машинах рабочей среды.</span><span class="sxs-lookup"><span data-stu-id="82d33-174">Otherwise, if you use the same plan for production and testing, it means your tests are running on your production VMs.</span></span>

- <span data-ttu-id="82d33-175">Для управления развертываниями используйте слоты развертывания.</span><span class="sxs-lookup"><span data-stu-id="82d33-175">Use deployment slots to manage deployments.</span></span> <span data-ttu-id="82d33-176">Это позволит развернуть обновленную версию в промежуточном слоте, а затем переключить на нее рабочий слот.</span><span class="sxs-lookup"><span data-stu-id="82d33-176">This lets you to deploy an updated version to a staging slot, then swap over to the new version.</span></span> <span data-ttu-id="82d33-177">Кроме того, вы сможете легко вернуться к предыдущей версии, если возникнет проблема с обновлением.</span><span class="sxs-lookup"><span data-stu-id="82d33-177">It also lets you swap back to the previous version, if there was a problem with the update.</span></span>

<!-- links -->

[api-design]: ../../best-practices/api-design.md
[autoscaling]: ../../best-practices/auto-scaling.md
[caching]: ../../best-practices/caching.md
[cdn]: ../../best-practices/cdn.md
[data-partition]: ../../best-practices/data-partitioning.md
[polyglot]: ../design-principles/use-the-best-data-store.md
[scalable-web-app]: ../../reference-architectures/app-service-web-app/scalable-web-app.md