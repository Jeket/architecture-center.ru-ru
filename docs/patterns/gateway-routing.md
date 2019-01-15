---
title: Схема маршрутизации шлюза
titleSuffix: Cloud Design Patterns
description: Используйте одну конечную точку при маршрутизации запросов к нескольким службам.
keywords: Конструктивный шаблон
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 4db98038f582e0315a743a55d46013d2eda187e3
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54010483"
---
# <a name="gateway-routing-pattern"></a><span data-ttu-id="62d51-104">Схема маршрутизации шлюза</span><span class="sxs-lookup"><span data-stu-id="62d51-104">Gateway Routing pattern</span></span>

<span data-ttu-id="62d51-105">Используйте одну конечную точку при маршрутизации запросов к нескольким службам.</span><span class="sxs-lookup"><span data-stu-id="62d51-105">Route requests to multiple services using a single endpoint.</span></span> <span data-ttu-id="62d51-106">Такой подход полезен, если нужно предоставить несколько служб через одну конечную точку, направляя разные запросы к разным службам.</span><span class="sxs-lookup"><span data-stu-id="62d51-106">This pattern is useful when you wish to expose multiple services on a single endpoint and route to the appropriate service based on the request.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="62d51-107">Контекст и проблема</span><span class="sxs-lookup"><span data-stu-id="62d51-107">Context and problem</span></span>

<span data-ttu-id="62d51-108">Если клиенту нужно использовать несколько служб, настройка отдельной конечной точки для каждой службы и управление ими могут представлять определенные трудности.</span><span class="sxs-lookup"><span data-stu-id="62d51-108">When a client needs to consume multiple services, setting up a separate endpoint for each service and having the client manage each endpoint can be challenging.</span></span> <span data-ttu-id="62d51-109">Например, приложение электронной торговли предоставляет сразу несколько служб, отвечающих за поиск, обзор, корзину покупок, оплату и историю заказов.</span><span class="sxs-lookup"><span data-stu-id="62d51-109">For example, an e-commerce application might provide services such as search, reviews, cart, checkout, and order history.</span></span> <span data-ttu-id="62d51-110">В каждой службе реализован отдельный API-интерфейс, с которым должны взаимодействовать клиенты. При этом каждый клиент должен иметь информацию обо всех конечных точках для подключения к этим службам.</span><span class="sxs-lookup"><span data-stu-id="62d51-110">Each service has a different API that the client must interact with, and the client must know about each endpoint in order to connect to the services.</span></span> <span data-ttu-id="62d51-111">Если изменяется любой API, приходится обновлять все клиенты.</span><span class="sxs-lookup"><span data-stu-id="62d51-111">If an API changes, the client must be updated as well.</span></span> <span data-ttu-id="62d51-112">Если вы разделяете одну службу на две или несколько служб, приходится изменять код в самой службе и во всех клиентах.</span><span class="sxs-lookup"><span data-stu-id="62d51-112">If you refactor a service into two or more separate services, the code must change in both the service and the client.</span></span>

## <a name="solution"></a><span data-ttu-id="62d51-113">Решение</span><span class="sxs-lookup"><span data-stu-id="62d51-113">Solution</span></span>

<span data-ttu-id="62d51-114">Установите шлюз перед набором приложений, служб и развертываний.</span><span class="sxs-lookup"><span data-stu-id="62d51-114">Place a gateway in front of a set of applications, services, or deployments.</span></span> <span data-ttu-id="62d51-115">Используйте маршрутизацию на уровне приложений (уровень 7) для передачи запросов в соответствующие экземпляры.</span><span class="sxs-lookup"><span data-stu-id="62d51-115">Use application Layer 7 routing to route the request to the appropriate instances.</span></span>

<span data-ttu-id="62d51-116">При такой схеме работы клиентское приложение имеет информацию только об одной конечной точке и взаимодействует только с ней.</span><span class="sxs-lookup"><span data-stu-id="62d51-116">With this pattern, the client application only needs to know about and communicate with a single endpoint.</span></span> <span data-ttu-id="62d51-117">При слияниях и разделениях служб клиент не всегда требуется обновлять.</span><span class="sxs-lookup"><span data-stu-id="62d51-117">If a service is consolidated or decomposed, the client does not necessarily require updating.</span></span> <span data-ttu-id="62d51-118">Он может по-прежнему направлять запросы в шлюз, а изменения затронут только маршрутизацию.</span><span class="sxs-lookup"><span data-stu-id="62d51-118">It can continue making requests to the gateway, and only the routing changes.</span></span>

<span data-ttu-id="62d51-119">Кроме того, шлюз обеспечивает абстракцию серверных служб для клиентов. Это позволяет использовать несложный формат запросов и вносить любые изменения в серверные службы за шлюзом.</span><span class="sxs-lookup"><span data-stu-id="62d51-119">A gateway also lets you abstract backend services from the clients, allowing you to keep client calls simple while enabling changes in the backend services behind the gateway.</span></span> <span data-ttu-id="62d51-120">Запросы от клиента можно направлять для обработки в любую службу или несколько служб в зависимости от ситуации. При этом вы можете свободно добавлять, разделять или реорганизовывать службы, ничего не изменяя в клиенте.</span><span class="sxs-lookup"><span data-stu-id="62d51-120">Client calls can be routed to whatever service or services need to handle the expected client behavior, allowing you to add, split, and reorganize services behind the gateway without changing the client.</span></span>

![Шаблон маршрутизации шлюза](./_images/gateway-routing.png)

<span data-ttu-id="62d51-122">Такая схема удобна и для управления развертыванием, так как дает свободу в применении обновлений для пользователей.</span><span class="sxs-lookup"><span data-stu-id="62d51-122">This pattern can also help with deployment, by allowing you to manage how updates are rolled out to users.</span></span> <span data-ttu-id="62d51-123">Когда вы развертываете новую версию службы, она может работать параллельно с предыдущей.</span><span class="sxs-lookup"><span data-stu-id="62d51-123">When a new version of your service is deployed, it can be deployed in parallel with the existing version.</span></span> <span data-ttu-id="62d51-124">С помощью маршрутизации вы можете предоставлять клиентам разные версии службы, позволяя гибко применять разные стратегии обновления: добавочное, параллельное или полное развертывание выпусков.</span><span class="sxs-lookup"><span data-stu-id="62d51-124">Routing lets you control what version of the service is presented to the clients, giving you the flexibility to use various release strategies, whether incremental, parallel, or complete rollouts of updates.</span></span> <span data-ttu-id="62d51-125">Если после развертывания новой версии службы возникнут проблемы, вы сможете быстро отменить любые изменения с помощью маршрутизации, и эти изменения никак не затронут клиенты.</span><span class="sxs-lookup"><span data-stu-id="62d51-125">Any issues discovered after the new service is deployed can be quickly reverted by making a configuration change at the gateway, without affecting clients.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="62d51-126">Проблемы и рекомендации</span><span class="sxs-lookup"><span data-stu-id="62d51-126">Issues and considerations</span></span>

- <span data-ttu-id="62d51-127">Служба шлюза может стать единой точкой отказа.</span><span class="sxs-lookup"><span data-stu-id="62d51-127">The gateway service may introduce a single point of failure.</span></span> <span data-ttu-id="62d51-128">Важно правильно подойти к ее разработке, чтобы она соответствовала требованиям к доступности.</span><span class="sxs-lookup"><span data-stu-id="62d51-128">Ensure it is properly designed to meet your availability requirements.</span></span> <span data-ttu-id="62d51-129">При реализации оцените ее возможности в плане отказоустойчивости и надежности.</span><span class="sxs-lookup"><span data-stu-id="62d51-129">Consider resiliency and fault tolerance capabilities when implementing.</span></span>
- <span data-ttu-id="62d51-130">Служба шлюза может стать узким местом системы.</span><span class="sxs-lookup"><span data-stu-id="62d51-130">The gateway service may introduce a bottleneck.</span></span> <span data-ttu-id="62d51-131">Убедитесь, что шлюз достаточно производителен для планируемой нагрузки и что его будет легко масштабировать в соответствии с ожидаемым темпом роста.</span><span class="sxs-lookup"><span data-stu-id="62d51-131">Ensure the gateway has adequate performance to handle load and can easily scale in line with your growth expectations.</span></span>
- <span data-ttu-id="62d51-132">Выполните нагрузочное тестирование шлюза, чтобы исключить возможность каскадных сбоев служб.</span><span class="sxs-lookup"><span data-stu-id="62d51-132">Perform load testing against the gateway to ensure you don't introduce cascading failures for services.</span></span>
- <span data-ttu-id="62d51-133">Маршрутизация шлюза выполняется на 7-м уровне.</span><span class="sxs-lookup"><span data-stu-id="62d51-133">Gateway routing is level 7.</span></span> <span data-ttu-id="62d51-134">Для этого можно использовать IP-адреса, порты, заголовки запросов и (или) URL-адреса.</span><span class="sxs-lookup"><span data-stu-id="62d51-134">It can be based on IP, port, header, or URL.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="62d51-135">Когда следует использовать этот шаблон</span><span class="sxs-lookup"><span data-stu-id="62d51-135">When to use this pattern</span></span>

<span data-ttu-id="62d51-136">Используйте этот шаблон в следующих случаях:</span><span class="sxs-lookup"><span data-stu-id="62d51-136">Use this pattern when:</span></span>

- <span data-ttu-id="62d51-137">если клиенту нужно использовать несколько служб, доступ к которым возможен через шлюз;</span><span class="sxs-lookup"><span data-stu-id="62d51-137">A client needs to consume multiple services that can be accessed behind a gateway.</span></span>
- <span data-ttu-id="62d51-138">если вам нужно упростить клиентские приложения, чтобы они взаимодействовали только с одной конечной точкой;</span><span class="sxs-lookup"><span data-stu-id="62d51-138">You wish to simplify client applications by using a single endpoint.</span></span>
- <span data-ttu-id="62d51-139">если вам нужно перенаправлять на виртуальные внутренние конечные точки запросы, поступающие на конечные точки, которые доступны из внешних сетей (например, чтобы предоставить доступ к портам виртуальной машины через IP-адреса виртуального кластера).</span><span class="sxs-lookup"><span data-stu-id="62d51-139">You need to route requests from externally addressable endpoints to internal virtual endpoints, such as exposing ports on a VM to cluster virtual IP addresses.</span></span>

<span data-ttu-id="62d51-140">Такая схема не очень удобна, если вы используете простое приложение для доступа всего к одной или двум службам.</span><span class="sxs-lookup"><span data-stu-id="62d51-140">This pattern may not be suitable when you have a simple application that uses only one or two services.</span></span>

## <a name="example"></a><span data-ttu-id="62d51-141">Пример</span><span class="sxs-lookup"><span data-stu-id="62d51-141">Example</span></span>

<span data-ttu-id="62d51-142">Ниже приведен простой пример файла конфигурации для сервера, на котором Nginx выполняет функции маршрутизатора и передает запросы к приложениям, размещенным в разных виртуальных каталогах на разных компьютерах серверной части.</span><span class="sxs-lookup"><span data-stu-id="62d51-142">Using Nginx as the router, the following is a simple example configuration file for a server that routes requests for applications residing on different virtual directories to different machines at the back end.</span></span>

```console
server {
    listen 80;
    server_name domain.com;

    location /app1 {
        proxy_pass http://10.0.3.10:80;
    }

    location /app2 {
        proxy_pass http://10.0.3.20:80;
    }

    location /app3 {
        proxy_pass http://10.0.3.30:80;
    }
}
```

## <a name="related-guidance"></a><span data-ttu-id="62d51-143">Связанные руководства</span><span class="sxs-lookup"><span data-stu-id="62d51-143">Related guidance</span></span>

- [<span data-ttu-id="62d51-144">Схема отдельных серверных частей для каждого интерфейса</span><span class="sxs-lookup"><span data-stu-id="62d51-144">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- [<span data-ttu-id="62d51-145">Схема агрегирования на шлюзе</span><span class="sxs-lookup"><span data-stu-id="62d51-145">Gateway Aggregation pattern</span></span>](./gateway-aggregation.md)
- [<span data-ttu-id="62d51-146">Схема разгрузки шлюза</span><span class="sxs-lookup"><span data-stu-id="62d51-146">Gateway Offloading pattern</span></span>](./gateway-offloading.md)
