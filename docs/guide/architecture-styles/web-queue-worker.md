---
title: Стиль архитектуры "Интерфейс — очередь — рабочая роль"
description: Описание преимуществ и сложностей архитектуры "Интерфейс — очередь — рабочая роль" в Azure, а также рекомендации по ее разработке
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: 0ebcf49c08c74cec3f1820da2d6f30ba95256e81
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/31/2018
ms.locfileid: "43325357"
---
# <a name="web-queue-worker-architecture-style"></a><span data-ttu-id="9e66b-103">Стиль архитектуры "Интерфейс — очередь — рабочая роль"</span><span class="sxs-lookup"><span data-stu-id="9e66b-103">Web-Queue-Worker architecture style</span></span>

<span data-ttu-id="9e66b-104">Основными компонентами этой архитектуры являются **веб-интерфейс** для обслуживания клиентских запросов и **рабочая роль** для выполнения ресурсоемких задач, долгосрочных рабочих процессов или пакетных заданий.</span><span class="sxs-lookup"><span data-stu-id="9e66b-104">The core components of this architecture are a **web front end** that serves client requests, and a **worker** that performs resource-intensive tasks, long-running workflows, or batch jobs.</span></span>  <span data-ttu-id="9e66b-105">Веб-интерфейс взаимодействует с рабочей ролью через **очередь сообщений**.</span><span class="sxs-lookup"><span data-stu-id="9e66b-105">The web front end communicates with the worker through a **message queue**.</span></span>  

![](./images/web-queue-worker-logical.svg)

<span data-ttu-id="9e66b-106">Также в эту архитектуру часто включают другие компоненты, в том числе:</span><span class="sxs-lookup"><span data-stu-id="9e66b-106">Other components that are commonly incorporated into this architecture include:</span></span>

- <span data-ttu-id="9e66b-107">одну или несколько баз данных;</span><span class="sxs-lookup"><span data-stu-id="9e66b-107">One or more databases.</span></span> 
- <span data-ttu-id="9e66b-108">кэш для хранения значений из базы данных для быстрого чтения;</span><span class="sxs-lookup"><span data-stu-id="9e66b-108">A cache to store values from the database for quick reads.</span></span>
- <span data-ttu-id="9e66b-109">сеть доставки содержимого, которая обслуживает статическое содержимое;</span><span class="sxs-lookup"><span data-stu-id="9e66b-109">CDN to serve static content</span></span>
- <span data-ttu-id="9e66b-110">удаленные службы, например служба электронной почты или текстовых сообщений</span><span class="sxs-lookup"><span data-stu-id="9e66b-110">Remote services, such as email or SMS service.</span></span> <span data-ttu-id="9e66b-111">(часто это службы сторонних производителей);</span><span class="sxs-lookup"><span data-stu-id="9e66b-111">Often these are provided by third parties.</span></span>
- <span data-ttu-id="9e66b-112">поставщик удостоверений для аутентификации.</span><span class="sxs-lookup"><span data-stu-id="9e66b-112">Identity provider for authentication.</span></span>

<span data-ttu-id="9e66b-113">Веб-интерфейс и рабочая роль реализованы без отслеживания состояния.</span><span class="sxs-lookup"><span data-stu-id="9e66b-113">The web and worker are both stateless.</span></span> <span data-ttu-id="9e66b-114">Сведения о состоянии сеанса можно сохранять в распределенном кэше.</span><span class="sxs-lookup"><span data-stu-id="9e66b-114">Session state can be stored in a distributed cache.</span></span> <span data-ttu-id="9e66b-115">Любые длительные задачи выполняются рабочей ролью в асинхронном режиме.</span><span class="sxs-lookup"><span data-stu-id="9e66b-115">Any long-running work is done asynchronously by the worker.</span></span> <span data-ttu-id="9e66b-116">Рабочая роль запускается при помещении сообщений в очередь или по расписанию для пакетной обработки.</span><span class="sxs-lookup"><span data-stu-id="9e66b-116">The worker can be triggered by messages on the queue, or run on a schedule for batch processing.</span></span> <span data-ttu-id="9e66b-117">Рабочая роль используется не всегда.</span><span class="sxs-lookup"><span data-stu-id="9e66b-117">The worker is an optional component.</span></span> <span data-ttu-id="9e66b-118">Если длительные операции не требуются, ее можно не вызывать.</span><span class="sxs-lookup"><span data-stu-id="9e66b-118">If there are no long-running operations, the worker can be omitted.</span></span>  

<span data-ttu-id="9e66b-119">Внешний интерфейс может быть реализован как веб-API.</span><span class="sxs-lookup"><span data-stu-id="9e66b-119">The front end might consist of a web API.</span></span> <span data-ttu-id="9e66b-120">На стороне клиента веб-API представлен в виде одностраничного приложения, в котором выполняются вызовы через AJAX, или собственного клиентского приложения.</span><span class="sxs-lookup"><span data-stu-id="9e66b-120">On the client side, the web API can be consumed by a single-page application that makes AJAX calls, or by a native client application.</span></span>

## <a name="when-to-use-this-architecture"></a><span data-ttu-id="9e66b-121">Когда следует использовать эту архитектуру</span><span class="sxs-lookup"><span data-stu-id="9e66b-121">When to use this architecture</span></span>

<span data-ttu-id="9e66b-122">Архитектура "Интерфейс — очередь — рабочая роль" обычно реализуется на основе управляемых вычислительных служб, таких как служба приложений Azure или облачные службы Azure.</span><span class="sxs-lookup"><span data-stu-id="9e66b-122">The Web-Queue-Worker architecture is typically implemented using managed compute services, either Azure App Service or Azure Cloud Services.</span></span> 

<span data-ttu-id="9e66b-123">Используйте эту архитектуру для следующих сценариев:</span><span class="sxs-lookup"><span data-stu-id="9e66b-123">Consider this architecture style for:</span></span>

- <span data-ttu-id="9e66b-124">приложения с относительно несложными функциями;</span><span class="sxs-lookup"><span data-stu-id="9e66b-124">Applications with a relatively simple domain.</span></span>
- <span data-ttu-id="9e66b-125">приложения, в которых выполняются долгосрочные процессы или пакетные операции;</span><span class="sxs-lookup"><span data-stu-id="9e66b-125">Applications with some long-running workflows or batch operations.</span></span>
- <span data-ttu-id="9e66b-126">если нужно применить управляемые службы вместо решений IaaS (инфраструктура как услуга).</span><span class="sxs-lookup"><span data-stu-id="9e66b-126">When you want to use managed services, rather than infrastructure as a service (IaaS).</span></span>

## <a name="benefits"></a><span data-ttu-id="9e66b-127">Преимущества</span><span class="sxs-lookup"><span data-stu-id="9e66b-127">Benefits</span></span>

- <span data-ttu-id="9e66b-128">Это несложная и удобная архитектура.</span><span class="sxs-lookup"><span data-stu-id="9e66b-128">Relatively simple architecture that is easy to understand.</span></span>
- <span data-ttu-id="9e66b-129">Она проста в развертывании и управлении.</span><span class="sxs-lookup"><span data-stu-id="9e66b-129">Easy to deploy and manage.</span></span>
- <span data-ttu-id="9e66b-130">Четко разделены зоны ответственности.</span><span class="sxs-lookup"><span data-stu-id="9e66b-130">Clear separation of concerns.</span></span>
- <span data-ttu-id="9e66b-131">Внешний интерфейс отделен от рабочей роли благодаря механизму асинхронного обмена сообщениями.</span><span class="sxs-lookup"><span data-stu-id="9e66b-131">The front end is decoupled from the worker using asynchronous messaging.</span></span>
- <span data-ttu-id="9e66b-132">Внешний интерфейс и рабочую роль можно масштабировать независимо друг от друга.</span><span class="sxs-lookup"><span data-stu-id="9e66b-132">The front end and the worker can be scaled independently.</span></span>

## <a name="challenges"></a><span data-ttu-id="9e66b-133">Сложности</span><span class="sxs-lookup"><span data-stu-id="9e66b-133">Challenges</span></span>

- <span data-ttu-id="9e66b-134">К разработке нужно подходить внимательно, иначе внешний интерфейс и (или) рабочая роль могут стать громоздкими и монолитными компонентами, которые трудно обслуживать и обновлять.</span><span class="sxs-lookup"><span data-stu-id="9e66b-134">Without careful design, the front end and the worker can become large, monolithic components that are difficult to maintain and update.</span></span>
- <span data-ttu-id="9e66b-135">Если во внешнем интерфейсе и рабочей роли используются общие схемы данных или модули, могут возникнуть скрытые зависимости.</span><span class="sxs-lookup"><span data-stu-id="9e66b-135">There may be hidden dependencies, if the front end and worker share data schemas or code modules.</span></span> 

## <a name="best-practices"></a><span data-ttu-id="9e66b-136">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="9e66b-136">Best practices</span></span>

- <span data-ttu-id="9e66b-137">Предоставлять клиенту хорошо спроектированный API-интерфейс.</span><span class="sxs-lookup"><span data-stu-id="9e66b-137">Expose a well-designed API to the client.</span></span> <span data-ttu-id="9e66b-138">См. [рекомендации по проектированию API][api-design].</span><span class="sxs-lookup"><span data-stu-id="9e66b-138">See [API design best practices][api-design].</span></span>
- <span data-ttu-id="9e66b-139">Автоматически масштабировать систему с учетом изменений нагрузки.</span><span class="sxs-lookup"><span data-stu-id="9e66b-139">Autoscale to handle changes in load.</span></span> <span data-ttu-id="9e66b-140">См. [рекомендации по автомасштабированию][autoscaling].</span><span class="sxs-lookup"><span data-stu-id="9e66b-140">See [Autoscaling best practices][autoscaling].</span></span>
- <span data-ttu-id="9e66b-141">Кэшировать данные с признаками статических.</span><span class="sxs-lookup"><span data-stu-id="9e66b-141">Cache semi-static data.</span></span> <span data-ttu-id="9e66b-142">См. [рекомендации по кэшированию][caching].</span><span class="sxs-lookup"><span data-stu-id="9e66b-142">See [Caching best practices][caching].</span></span>
- <span data-ttu-id="9e66b-143">Использовать сеть доставки содержимого для размещения статического содержимого.</span><span class="sxs-lookup"><span data-stu-id="9e66b-143">Use a CDN to host static content.</span></span> <span data-ttu-id="9e66b-144">См. [рекомендации по использованию CDN][cdn].</span><span class="sxs-lookup"><span data-stu-id="9e66b-144">See [CDN best practices][cdn].</span></span>
- <span data-ttu-id="9e66b-145">При возможности использовать Polyglot Persistence.</span><span class="sxs-lookup"><span data-stu-id="9e66b-145">Use polyglot persistence when appropriate.</span></span> <span data-ttu-id="9e66b-146">См. статью об [использовании подходящего хранилища данных для задания][polyglot].</span><span class="sxs-lookup"><span data-stu-id="9e66b-146">See [Use the best data store for the job][polyglot].</span></span>
- <span data-ttu-id="9e66b-147">Секционировать данные, чтобы улучшить масштабируемость, уменьшить количество конфликтов и оптимизировать производительность.</span><span class="sxs-lookup"><span data-stu-id="9e66b-147">Partition data to improve scalability, reduce contention, and optimize performance.</span></span> <span data-ttu-id="9e66b-148">См. [рекомендации по секционированию данных][data-partition].</span><span class="sxs-lookup"><span data-stu-id="9e66b-148">See [Data partitioning best practices][data-partition].</span></span>


## <a name="web-queue-worker-on-azure-app-service"></a><span data-ttu-id="9e66b-149">Модель "Интерфейс — очередь — рабочая роль" в службе приложений Azure</span><span class="sxs-lookup"><span data-stu-id="9e66b-149">Web-Queue-Worker on Azure App Service</span></span>

<span data-ttu-id="9e66b-150">В этом разделе описана рекомендуемая архитектура "Интерфейс — очередь — рабочая роль" для службы приложений Azure.</span><span class="sxs-lookup"><span data-stu-id="9e66b-150">This section describes a recommended Web-Queue-Worker architecture that uses Azure App Service.</span></span> 

![](./images/web-queue-worker-physical.png)

<span data-ttu-id="9e66b-151">Внешний интерфейс реализуется в виде веб-приложения службы приложений Azure, а рабочая роль — как веб-задание.</span><span class="sxs-lookup"><span data-stu-id="9e66b-151">The front end is implemented as an Azure App Service web app, and the worker is implemented as a WebJob.</span></span> <span data-ttu-id="9e66b-152">Эти веб-приложение и веб-задание сопоставляются с планом службы приложений, который предоставляет экземпляры виртуальных машин.</span><span class="sxs-lookup"><span data-stu-id="9e66b-152">The web app and the WebJob are both associated with an App Service plan that provides the VM instances.</span></span> 

<span data-ttu-id="9e66b-153">В качестве очереди сообщений можно применить очередь служебной шины Azure или очередь службы хранилища Azure.</span><span class="sxs-lookup"><span data-stu-id="9e66b-153">You can use either Azure Service Bus or Azure Storage queues for the message queue.</span></span> <span data-ttu-id="9e66b-154">(На схеме представлена очередь службы хранилища Azure.)</span><span class="sxs-lookup"><span data-stu-id="9e66b-154">(The diagram shows an Azure Storage queue.)</span></span>

<span data-ttu-id="9e66b-155">В кэше Redis для Azure хранятся сведения о состоянии сеанса и другие данные, к которым нужен доступ с низкой задержкой.</span><span class="sxs-lookup"><span data-stu-id="9e66b-155">Azure Redis Cache stores session state and other data that needs low latency access.</span></span>

<span data-ttu-id="9e66b-156">Azure CDN кэширует статическое содержимое, например изображения, документы HTML и CSS.</span><span class="sxs-lookup"><span data-stu-id="9e66b-156">Azure CDN is used to cache static content such as images, CSS, or HTML.</span></span>

<span data-ttu-id="9e66b-157">Для хранения данных вы можете выбрать любую технологию, соответствующую требованиям приложения.</span><span class="sxs-lookup"><span data-stu-id="9e66b-157">For storage, choose the storage technologies that best fit the needs of the application.</span></span> <span data-ttu-id="9e66b-158">Можно одновременно использовать несколько технологий хранения данных (Polyglot Persistence).</span><span class="sxs-lookup"><span data-stu-id="9e66b-158">You might use multiple storage technologies (polyglot persistence).</span></span> <span data-ttu-id="9e66b-159">Эта концепция отражена на схеме в виде блоков службы "База данных SQL Azure" и Azure Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="9e66b-159">To illustrate this idea, the diagram shows Azure SQL Database and Azure Cosmos DB.</span></span>  

<span data-ttu-id="9e66b-160">Дополнительные сведения см. в [описании базовой архитектуры для веб-приложения в службе приложений][scalable-web-app].</span><span class="sxs-lookup"><span data-stu-id="9e66b-160">For more details, see [App Service web application reference architecture][scalable-web-app].</span></span>

### <a name="additional-considerations"></a><span data-ttu-id="9e66b-161">Дополнительные замечания</span><span class="sxs-lookup"><span data-stu-id="9e66b-161">Additional considerations</span></span>

- <span data-ttu-id="9e66b-162">Не все транзакции передаются в хранилище через очередь и рабочую роль.</span><span class="sxs-lookup"><span data-stu-id="9e66b-162">Not every transaction has to go through the queue and worker to storage.</span></span> <span data-ttu-id="9e66b-163">Простые операции чтения и (или) записи веб-интерфейс может выполнять напрямую.</span><span class="sxs-lookup"><span data-stu-id="9e66b-163">The web front end can perform simple read/write operations directly.</span></span> <span data-ttu-id="9e66b-164">Рабочие роли предназначены только для ресурсоемких задач или долго выполняющихся рабочих процессов.</span><span class="sxs-lookup"><span data-stu-id="9e66b-164">Workers are designed for resource-intensive tasks or long-running workflows.</span></span> <span data-ttu-id="9e66b-165">В некоторых сценариях рабочая роль не нужна.</span><span class="sxs-lookup"><span data-stu-id="9e66b-165">In some cases, you might not need a worker at all.</span></span>

- <span data-ttu-id="9e66b-166">Встроенная в службу приложений функция автоматического масштабирования позволяет изменять количество экземпляров виртуальных машин.</span><span class="sxs-lookup"><span data-stu-id="9e66b-166">Use the built-in autoscale feature of App Service to scale out the number of VM instances.</span></span> <span data-ttu-id="9e66b-167">При четко прогнозируемой нагрузке на приложение используйте автоматическое масштабирование на основе расписания.</span><span class="sxs-lookup"><span data-stu-id="9e66b-167">If the load on the application follows predictable patterns, use schedule-based autoscale.</span></span> <span data-ttu-id="9e66b-168">Если нагрузка непредсказуема, используйте правила автоматического масштабирования на основе метрик.</span><span class="sxs-lookup"><span data-stu-id="9e66b-168">If the load is unpredictable, use metrics-based autoscaling rules.</span></span>      

- <span data-ttu-id="9e66b-169">Возможно, веб-приложение и веб-задание стоит разместить в разных планах службы приложений.</span><span class="sxs-lookup"><span data-stu-id="9e66b-169">Consider putting the web app and the WebJob into separate App Service plans.</span></span> <span data-ttu-id="9e66b-170">Тогда они будут выполняться на разных экземплярах виртуальных машин и вы сможете масштабировать их независимо друг от друга.</span><span class="sxs-lookup"><span data-stu-id="9e66b-170">That way, they are hosted on separate VM instances and can be scaled independently.</span></span> 

- <span data-ttu-id="9e66b-171">Используйте разные планы службы приложений для рабочей и тестовой сред.</span><span class="sxs-lookup"><span data-stu-id="9e66b-171">Use separate App Service plans for production and testing.</span></span> <span data-ttu-id="9e66b-172">Если у вас будет общий план для рабочей среды и тестирования, все задачи тестирования будут выполняться прямо на виртуальных машинах рабочей среды.</span><span class="sxs-lookup"><span data-stu-id="9e66b-172">Otherwise, if you use the same plan for production and testing, it means your tests are running on your production VMs.</span></span>

- <span data-ttu-id="9e66b-173">Для управления развертываниями используйте слоты развертывания.</span><span class="sxs-lookup"><span data-stu-id="9e66b-173">Use deployment slots to manage deployments.</span></span> <span data-ttu-id="9e66b-174">Это позволит развернуть обновленную версию в промежуточном слоте, а затем переключить на нее рабочий слот.</span><span class="sxs-lookup"><span data-stu-id="9e66b-174">This lets you to deploy an updated version to a staging slot, then swap over to the new version.</span></span> <span data-ttu-id="9e66b-175">Кроме того, вы сможете легко вернуться к предыдущей версии, если возникнет проблема с обновлением.</span><span class="sxs-lookup"><span data-stu-id="9e66b-175">It also lets you swap back to the previous version, if there was a problem with the update.</span></span>

<!-- links -->

[api-design]: ../../best-practices/api-design.md
[autoscaling]: ../../best-practices/auto-scaling.md
[caching]: ../../best-practices/caching.md
[cdn]: ../../best-practices/cdn.md
[data-partition]: ../../best-practices/data-partitioning.md
[polyglot]: ../design-principles/use-the-best-data-store.md
[scalable-web-app]: ../../reference-architectures/app-service-web-app/scalable-web-app.md