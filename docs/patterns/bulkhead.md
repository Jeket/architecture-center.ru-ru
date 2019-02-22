---
title: Шаблон отсеков
titleSuffix: Cloud Design Patterns
description: Изоляция элементов приложения в пулах, чтобы в случае сбоя одного элемента остальные продолжали функционировать.
keywords: Конструктивный шаблон
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 4ad221ae5e9bd51c6d304ba33b884f71c5081b16
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55898261"
---
# <a name="bulkhead-pattern"></a><span data-ttu-id="29847-104">Шаблон отсеков</span><span class="sxs-lookup"><span data-stu-id="29847-104">Bulkhead pattern</span></span>

<span data-ttu-id="29847-105">Изоляция элементов приложения в пулах, чтобы в случае сбоя одного элемента остальные продолжали функционировать.</span><span class="sxs-lookup"><span data-stu-id="29847-105">Isolate elements of an application into pools so that if one fails, the others will continue to function.</span></span>

<span data-ttu-id="29847-106">В названии шаблона упоминаются *отсеки*, так в его основе лежит идея разделения внутреннего пространства корабля.</span><span class="sxs-lookup"><span data-stu-id="29847-106">This pattern is named *Bulkhead* because it resembles the sectioned partitions of a ship's hull.</span></span> <span data-ttu-id="29847-107">Если корпус корабля будет поврежден, вода заполнит только один поврежденный отсек, и корабль останется на плаву.</span><span class="sxs-lookup"><span data-stu-id="29847-107">If the hull of a ship is compromised, only the damaged section fills with water, which prevents the ship from sinking.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="29847-108">Контекст и проблема</span><span class="sxs-lookup"><span data-stu-id="29847-108">Context and problem</span></span>

<span data-ttu-id="29847-109">Облачные приложения могут содержать несколько служб, у каждой из которых есть один или несколько потребителей.</span><span class="sxs-lookup"><span data-stu-id="29847-109">A cloud-based application may include multiple services, with each service having one or more consumers.</span></span> <span data-ttu-id="29847-110">Избыточная нагрузка на службу или сбой повлияют на всех потребителей этой службы.</span><span class="sxs-lookup"><span data-stu-id="29847-110">Excessive load or failure in a service will impact all consumers of the service.</span></span>

<span data-ttu-id="29847-111">Кроме того, потребители могут отправлять запросы к нескольким службам одновременно, а каждый запрос использует некоторые ресурсы.</span><span class="sxs-lookup"><span data-stu-id="29847-111">Moreover, a consumer may send requests to multiple services simultaneously, using resources for each request.</span></span> <span data-ttu-id="29847-112">Когда пользователь отправляет запрос к неправильно настроенной или неисправной службе, выделенные для этого запроса ресурсы могут не освободиться своевременно.</span><span class="sxs-lookup"><span data-stu-id="29847-112">When the consumer sends a request to a service that is misconfigured or not responding, the resources used by the client's request may not be freed in a timely manner.</span></span> <span data-ttu-id="29847-113">При этом запросы продолжают поступать, что может привести к полному исчерпанию ресурсов.</span><span class="sxs-lookup"><span data-stu-id="29847-113">As requests to the service continue, those resources may be exhausted.</span></span> <span data-ttu-id="29847-114">Например, будут исчерпаны возможности пула клиентских подключений.</span><span class="sxs-lookup"><span data-stu-id="29847-114">For example, the client's connection pool may be exhausted.</span></span> <span data-ttu-id="29847-115">Теперь проблема затрагивает уже и запросы потребителей к другим службам.</span><span class="sxs-lookup"><span data-stu-id="29847-115">At that point, requests by the consumer to other services are affected.</span></span> <span data-ttu-id="29847-116">В конечном итоге потребители не смогут отправлять запросы ко всем службам, а не только к проблемной.</span><span class="sxs-lookup"><span data-stu-id="29847-116">Eventually the consumer can no longer send requests to other services, not just the original unresponsive service.</span></span>

<span data-ttu-id="29847-117">Эта же проблема нехватки ресурсов влияет и на службы с несколькими потребителями.</span><span class="sxs-lookup"><span data-stu-id="29847-117">The same issue of resource exhaustion affects services with multiple consumers.</span></span> <span data-ttu-id="29847-118">Большое количество запросов от одного клиента может исчерпать все доступные службе ресурсы.</span><span class="sxs-lookup"><span data-stu-id="29847-118">A large number of requests originating from one client may exhaust available resources in the service.</span></span> <span data-ttu-id="29847-119">Тогда другие пользователи не смогут использовать службу, что приводит к каскадному сбою.</span><span class="sxs-lookup"><span data-stu-id="29847-119">Other consumers are no longer able to consume the service, causing a cascading failure effect.</span></span>

## <a name="solution"></a><span data-ttu-id="29847-120">Решение</span><span class="sxs-lookup"><span data-stu-id="29847-120">Solution</span></span>

<span data-ttu-id="29847-121">Разделите экземпляры службы на несколько групп в соответствии с характером нагрузки от пользователей и требованиями доступности.</span><span class="sxs-lookup"><span data-stu-id="29847-121">Partition service instances into different groups, based on consumer load and availability requirements.</span></span> <span data-ttu-id="29847-122">Такой подход позволяет изолировать сбои, сохраняя работоспособность служб хотя бы для некоторых пользователей даже во время сбоя.</span><span class="sxs-lookup"><span data-stu-id="29847-122">This design helps to isolate failures, and allows you to sustain service functionality for some consumers, even during a failure.</span></span>

<span data-ttu-id="29847-123">Аналогично можно разделить ресурсы для потребителя, чтобы вызовы к одной службе не влияли на доступность ресурсов для вызова других служб.</span><span class="sxs-lookup"><span data-stu-id="29847-123">A consumer can also partition resources, to ensure that resources used to call one service don't affect the resources used to call another service.</span></span> <span data-ttu-id="29847-124">Например, если потребитель должен обращаться к нескольким службам, ему можно выделить отдельный пул подключений для каждой службы.</span><span class="sxs-lookup"><span data-stu-id="29847-124">For example, a consumer that calls multiple services may be assigned a connection pool for each service.</span></span> <span data-ttu-id="29847-125">Теперь, если одна из служб работает плохо, это повлияет только на пул подключений, назначенный для этой службы, что позволяет потребителю продолжать использовать другие службы.</span><span class="sxs-lookup"><span data-stu-id="29847-125">If a service begins to fail, it only affects the connection pool assigned for that service, allowing the consumer to continue using the other services.</span></span>

<span data-ttu-id="29847-126">Такая схема предоставляет следующие преимущества:</span><span class="sxs-lookup"><span data-stu-id="29847-126">The benefits of this pattern include:</span></span>

- <span data-ttu-id="29847-127">Изоляция потребителей и служб от каскадных сбоев.</span><span class="sxs-lookup"><span data-stu-id="29847-127">Isolates consumers and services from cascading failures.</span></span> <span data-ttu-id="29847-128">Проблема, затрагивающая потребителя или службу, будет изолирована в одном конкретном отсеке, а не обрушит все решение полностью.</span><span class="sxs-lookup"><span data-stu-id="29847-128">An issue affecting a consumer or service can be isolated within its own bulkhead, preventing the entire solution from failing.</span></span>
- <span data-ttu-id="29847-129">Вы сможете сохранить некоторую функциональность даже в случае сбоя службы.</span><span class="sxs-lookup"><span data-stu-id="29847-129">Allows you to preserve some functionality in the event of a service failure.</span></span> <span data-ttu-id="29847-130">Другие службы и функции приложения продолжат работать.</span><span class="sxs-lookup"><span data-stu-id="29847-130">Other services and features of the application will continue to work.</span></span>
- <span data-ttu-id="29847-131">Вы сможете развертывать для приложений службы с разным качеством обслуживания.</span><span class="sxs-lookup"><span data-stu-id="29847-131">Allows you to deploy services that offer a different quality of service for consuming applications.</span></span> <span data-ttu-id="29847-132">Например, пул высокоприоритетных потребителей будет использовать службы с высоким приоритетом.</span><span class="sxs-lookup"><span data-stu-id="29847-132">A high-priority consumer pool can be configured to use high-priority services.</span></span>

<span data-ttu-id="29847-133">На следующей схеме показаны отсеки, созданные для пулов подключений, которые обращаются к отдельным службам.</span><span class="sxs-lookup"><span data-stu-id="29847-133">The following diagram shows bulkheads structured around connection pools that call individual services.</span></span> <span data-ttu-id="29847-134">При сбое или проблемах в службе A пул подключений к ней изолируется, и такой сбой влияет только на те рабочие нагрузки, которые используют пул потоков этой службы A.</span><span class="sxs-lookup"><span data-stu-id="29847-134">If Service A fails or causes some other issue, the connection pool is isolated, so only workloads using the thread pool assigned to Service A are affected.</span></span> <span data-ttu-id="29847-135">Рабочие нагрузки, использующие службы B и C, не затрагиваются и могут продолжать работу без перебоев.</span><span class="sxs-lookup"><span data-stu-id="29847-135">Workloads that use Service B and C are not affected and can continue working without interruption.</span></span>

![Первая схема шаблона отсеков](./_images/bulkhead-1.png)

<span data-ttu-id="29847-137">На следующей схеме показано несколько клиентов, которые вызывают одну службу.</span><span class="sxs-lookup"><span data-stu-id="29847-137">The next diagram shows multiple clients calling a single service.</span></span> <span data-ttu-id="29847-138">Каждому клиенту назначен отдельный экземпляр этой службы.</span><span class="sxs-lookup"><span data-stu-id="29847-138">Each client is assigned a separate service instance.</span></span> <span data-ttu-id="29847-139">Здесь клиент 1 создал слишком много запросов и перегрузил свой экземпляр.</span><span class="sxs-lookup"><span data-stu-id="29847-139">Client 1 has made too many requests and overwhelmed its instance.</span></span> <span data-ttu-id="29847-140">Так как каждый экземпляр службы изолирован от других, остальные клиенты спокойно продолжают работать с этой службой.</span><span class="sxs-lookup"><span data-stu-id="29847-140">Because each service instance is isolated from the others, the other clients can continue making calls.</span></span>

![Первая схема шаблона отсеков](./_images/bulkhead-2.png)

## <a name="issues-and-considerations"></a><span data-ttu-id="29847-142">Проблемы и рекомендации</span><span class="sxs-lookup"><span data-stu-id="29847-142">Issues and considerations</span></span>

- <span data-ttu-id="29847-143">Проектируйте отсеки с учетом коммерческих и технических требований приложения.</span><span class="sxs-lookup"><span data-stu-id="29847-143">Define partitions around the business and technical requirements of the application.</span></span>
- <span data-ttu-id="29847-144">Выбирая технологию разделения служб или потребителей, учитывайте доступный уровень изоляции и влияние этой технологии на стоимость решения, производительность и управляемость.</span><span class="sxs-lookup"><span data-stu-id="29847-144">When partitioning services or consumers into bulkheads, consider the level of isolation offered by the technology as well as the overhead in terms of cost, performance and manageability.</span></span>
- <span data-ttu-id="29847-145">Рассмотрите возможность объединить шаблон отсеков с шаблонами автоматического отключения и (или) регулирования, чтобы создать более сложный механизм обработки сбоев.</span><span class="sxs-lookup"><span data-stu-id="29847-145">Consider combining bulkheads with retry, circuit breaker, and throttling patterns to provide more sophisticated fault handling.</span></span>
- <span data-ttu-id="29847-146">Для распределения потребителей между отсеками попробуйте применить процессы, пулы потоков и семафоры.</span><span class="sxs-lookup"><span data-stu-id="29847-146">When partitioning consumers into bulkheads, consider using processes, thread pools, and semaphores.</span></span> <span data-ttu-id="29847-147">Некоторые проекты, например [Netflix Hystrix][hystrix] и [Polly][polly] предлагают клиентские платформы для разделения отсеков.</span><span class="sxs-lookup"><span data-stu-id="29847-147">Projects like [Netflix Hystrix][hystrix] and [Polly][polly] offer a framework for creating consumer bulkheads.</span></span>
- <span data-ttu-id="29847-148">Для распределения служб между отсеками попробуйте применить отдельные виртуальные машины, контейнеры или процессы.</span><span class="sxs-lookup"><span data-stu-id="29847-148">When partitioning services into bulkheads, consider deploying them into separate virtual machines, containers, or processes.</span></span> <span data-ttu-id="29847-149">Контейнеры — это компромиссное решение, обеспечивающее изоляцию ресурсов и низкие издержки.</span><span class="sxs-lookup"><span data-stu-id="29847-149">Containers offer a good balance of resource isolation with fairly low overhead.</span></span>
- <span data-ttu-id="29847-150">Службы, которые взаимодействуют с помощью асинхронных сообщений, можно разделить с помощью разных наборов очередей.</span><span class="sxs-lookup"><span data-stu-id="29847-150">Services that communicate using asynchronous messages can be isolated through different sets of queues.</span></span> <span data-ttu-id="29847-151">Можно выделить для каждой очереди свой набор экземпляров, которые обрабатывают сообщения из нее, или определить специальный алгоритм для выборки и распределения сообщений в одной группе экземпляров.</span><span class="sxs-lookup"><span data-stu-id="29847-151">Each queue can have a dedicated set of instances processing messages on the queue, or a single group of instances using an algorithm to dequeue and dispatch processing.</span></span>
- <span data-ttu-id="29847-152">Определите уровень детализации для отсеков.</span><span class="sxs-lookup"><span data-stu-id="29847-152">Determine the level of granularity for the bulkheads.</span></span> <span data-ttu-id="29847-153">Например, если вы хотите распределить клиенты между несколькими секциями, можно создать по одной секции для каждого клиента или поместить в одну секцию несколько клиентов.</span><span class="sxs-lookup"><span data-stu-id="29847-153">For example, if you want to distribute tenants across partitions, you could place each tenant into a separate partition, or put several tenants into one partition.</span></span>
- <span data-ttu-id="29847-154">Отслеживайте производительность и соглашение об уровне обслуживания отдельно для каждой секции.</span><span class="sxs-lookup"><span data-stu-id="29847-154">Monitor each partition’s performance and SLA.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="29847-155">Когда следует использовать этот шаблон</span><span class="sxs-lookup"><span data-stu-id="29847-155">When to use this pattern</span></span>

<span data-ttu-id="29847-156">Используйте этот шаблон в следующих случаях:</span><span class="sxs-lookup"><span data-stu-id="29847-156">Use this pattern to:</span></span>

- <span data-ttu-id="29847-157">Для изоляции ресурсов, используемых набором серверных служб, особенно если приложение способно поддерживать некоторый уровень функциональности даже при неработоспобности одной из этих служб.</span><span class="sxs-lookup"><span data-stu-id="29847-157">Isolate resources used to consume a set of backend services, especially if the application can provide some level of functionality even when one of the services is not responding.</span></span>
- <span data-ttu-id="29847-158">Для изоляции критически важных потребителей от стандартных потребителей.</span><span class="sxs-lookup"><span data-stu-id="29847-158">Isolate critical consumers from standard consumers.</span></span>
- <span data-ttu-id="29847-159">Для защиты приложения от каскадных сбоев.</span><span class="sxs-lookup"><span data-stu-id="29847-159">Protect the application from cascading failures.</span></span>

<span data-ttu-id="29847-160">Эту схему не стоит применять в следующих случаях:</span><span class="sxs-lookup"><span data-stu-id="29847-160">This pattern may not be suitable when:</span></span>

- <span data-ttu-id="29847-161">Если для проекта нельзя неэффективно использовать ресурсы.</span><span class="sxs-lookup"><span data-stu-id="29847-161">Less efficient use of resources may not be acceptable in the project.</span></span>
- <span data-ttu-id="29847-162">Если дополнительная сложность ничем не оправдана.</span><span class="sxs-lookup"><span data-stu-id="29847-162">The added complexity is not necessary</span></span>

## <a name="example"></a><span data-ttu-id="29847-163">Пример</span><span class="sxs-lookup"><span data-stu-id="29847-163">Example</span></span>

<span data-ttu-id="29847-164">В следующем файле конфигурации Kubernetes создается изолированный контейнер для выполнения одной службы с отдельными ограничениями и ресурсами ЦП и памяти.</span><span class="sxs-lookup"><span data-stu-id="29847-164">The following Kubernetes configuration file creates an isolated container to run a single service, with its own CPU and memory resources and limits.</span></span>

```yml
apiVersion: v1
kind: Pod
metadata:
  name: drone-management
spec:
  containers:
  - name: drone-management-container
    image: drone-service
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "1"
```

## <a name="related-guidance"></a><span data-ttu-id="29847-165">Связанные руководства</span><span class="sxs-lookup"><span data-stu-id="29847-165">Related guidance</span></span>

- [<span data-ttu-id="29847-166">Проектирование устойчивых приложений для Azure</span><span class="sxs-lookup"><span data-stu-id="29847-166">Designing resilient applications for Azure</span></span>](../resiliency/index.md)
- [<span data-ttu-id="29847-167">Шаблон автоматического выключения</span><span class="sxs-lookup"><span data-stu-id="29847-167">Circuit Breaker pattern</span></span>](./circuit-breaker.md)
- [<span data-ttu-id="29847-168">Шаблон повторов</span><span class="sxs-lookup"><span data-stu-id="29847-168">Retry pattern</span></span>](./retry.md)
- [<span data-ttu-id="29847-169">Шаблон регулирования</span><span class="sxs-lookup"><span data-stu-id="29847-169">Throttling pattern</span></span>](./throttling.md)

<!-- links -->

[hystrix]: https://github.com/Netflix/Hystrix
[polly]: https://github.com/App-vNext/Polly
