---
title: Схема разгрузки шлюза
titleSuffix: Cloud Design Patterns
description: Перенесение общих или специализированных функций служб на прокси-сервер шлюза.
keywords: Конструктивный шаблон
author: dragon119
ms.date: 06/23/2017
ms.custom: seodec18
ms.openlocfilehash: 50af3d8593279986ed6efee55667187424c18e56
ms.sourcegitcommit: 680c9cef945dff6fee5e66b38e24f07804510fa9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/04/2019
ms.locfileid: "54010223"
---
# <a name="gateway-offloading-pattern"></a><span data-ttu-id="cfd77-104">Схема разгрузки шлюза</span><span class="sxs-lookup"><span data-stu-id="cfd77-104">Gateway Offloading pattern</span></span>

<span data-ttu-id="cfd77-105">Перенесение общих или специализированных функций служб на прокси-сервер шлюза.</span><span class="sxs-lookup"><span data-stu-id="cfd77-105">Offload shared or specialized service functionality to a gateway proxy.</span></span> <span data-ttu-id="cfd77-106">Эта схема позволяет упростить разработку приложений, перемещая общие функции служб (например, использование SSL-сертификатов) из разных частей приложения в шлюз.</span><span class="sxs-lookup"><span data-stu-id="cfd77-106">This pattern can simplify application development by moving shared service functionality, such as the use of SSL certificates, from other parts of the application into the gateway.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="cfd77-107">Контекст и проблема</span><span class="sxs-lookup"><span data-stu-id="cfd77-107">Context and problem</span></span>

<span data-ttu-id="cfd77-108">Некоторые компоненты традиционно используются сразу в нескольких службах. При этом для каждого из них нужно выполнять настройку, управление и обслуживание.</span><span class="sxs-lookup"><span data-stu-id="cfd77-108">Some features are commonly used across multiple services, and these features require configuration, management, and maintenance.</span></span> <span data-ttu-id="cfd77-109">Общая или специализированная служба, которая распространяется вместе с каждым развертыванием приложения, увеличивает административные издержки и повышает вероятность ошибки при развертывании.</span><span class="sxs-lookup"><span data-stu-id="cfd77-109">A shared or specialized service that is distributed with every application deployment increases the administrative overhead and increases the likelihood of deployment error.</span></span> <span data-ttu-id="cfd77-110">Все обновления общего компонента нужно развертывать во всех службах, в которых он используется.</span><span class="sxs-lookup"><span data-stu-id="cfd77-110">Any updates to a shared feature must be deployed across all services that share that feature.</span></span>

<span data-ttu-id="cfd77-111">Правильное решение проблем безопасности (проверка маркера, шифрование, управление SSL-сертификатами) и других сложных задач требует высокой квалификации разработчиков.</span><span class="sxs-lookup"><span data-stu-id="cfd77-111">Properly handling security issues (token validation, encryption, SSL certificate management) and other complex tasks can require team members to have highly specialized skills.</span></span> <span data-ttu-id="cfd77-112">Предположим, что необходимый для приложения сертификат нужно настраивать и развертывать во всех экземплярах приложения.</span><span class="sxs-lookup"><span data-stu-id="cfd77-112">For example, a certificate needed by an application must be configured and deployed on all application instances.</span></span> <span data-ttu-id="cfd77-113">Приходится следить за каждым новым развертыванием, чтобы контролировать срок действия сертификата.</span><span class="sxs-lookup"><span data-stu-id="cfd77-113">With each new deployment, the certificate must be managed to ensure that it does not expire.</span></span> <span data-ttu-id="cfd77-114">По истечении срока действия общего сертификата его нужно обновлять, проверять и утверждать отдельно в каждом развертывании приложения.</span><span class="sxs-lookup"><span data-stu-id="cfd77-114">Any common certificate that is due to expire must be updated, tested, and verified on every application deployment.</span></span>

<span data-ttu-id="cfd77-115">Для большого количества развертываний достаточно сложно развертывать и поддерживать и другие распространенные службы, например отвечающие за аутентификацию, авторизацию, ведение журналов, мониторинг или [регулирование](./throttling.md).</span><span class="sxs-lookup"><span data-stu-id="cfd77-115">Other common services such as authentication, authorization, logging, monitoring, or [throttling](./throttling.md) can be difficult to implement and manage across a large number of deployments.</span></span> <span data-ttu-id="cfd77-116">Иногда такие компоненты следует консолидировать, чтобы снизить издержки и вероятность ошибок.</span><span class="sxs-lookup"><span data-stu-id="cfd77-116">It may be better to consolidate this type of functionality, in order to reduce overhead and the chance of errors.</span></span>

## <a name="solution"></a><span data-ttu-id="cfd77-117">Решение</span><span class="sxs-lookup"><span data-stu-id="cfd77-117">Solution</span></span>

<span data-ttu-id="cfd77-118">Перенесите в шлюз API часть компонентов, в частности те, которые широко используются в приложении, например отвечают за управление сертификатами, аутентификацию, оконечную точку SSL, мониторинг, преобразование протоколов и регулирование.</span><span class="sxs-lookup"><span data-stu-id="cfd77-118">Offload some features into an API gateway, particularly cross-cutting concerns such as certificate management, authentication, SSL termination, monitoring, protocol translation, or throttling.</span></span>

<span data-ttu-id="cfd77-119">На следующей схеме представлен шлюз API, служащий оконечной точкой для входящих SSL-подключений.</span><span class="sxs-lookup"><span data-stu-id="cfd77-119">The following diagram shows an API gateway that terminates inbound SSL connections.</span></span> <span data-ttu-id="cfd77-120">Он обращается от имени исходной запрашивающей стороны ко всем HTTP-серверам, размещенным за этим шлюзом API.</span><span class="sxs-lookup"><span data-stu-id="cfd77-120">It requests data on behalf of the original requestor from any HTTP server upstream of the API gateway.</span></span>

 ![Схема разгрузки шлюза](./_images/gateway-offload.png)

<span data-ttu-id="cfd77-122">Такая схема предоставляет следующие преимущества:</span><span class="sxs-lookup"><span data-stu-id="cfd77-122">Benefits of this pattern include:</span></span>

- <span data-ttu-id="cfd77-123">Упрощает разработку служб, так как не нужно распространять и поддерживать для всех защищенных веб-узлов вспомогательные ресурсы, например сертификаты и настройки веб-сервера.</span><span class="sxs-lookup"><span data-stu-id="cfd77-123">Simplify the development of services by removing the need to distribute and maintain supporting resources, such as web server certificates and configuration for secure websites.</span></span> <span data-ttu-id="cfd77-124">Упрощает настройку и управление, а также повышает масштабируемость и удобство обновления служб.</span><span class="sxs-lookup"><span data-stu-id="cfd77-124">Simpler configuration results in easier management and scalability and makes service upgrades simpler.</span></span>

- <span data-ttu-id="cfd77-125">Позволяет поручить отдельным командам реализацию функций, требующих специальных навыков (например, в сфере систем безопасности).</span><span class="sxs-lookup"><span data-stu-id="cfd77-125">Allow dedicated teams to implement features that require specialized expertise, such as security.</span></span> <span data-ttu-id="cfd77-126">Благодаря этому основная команда разработчиков может сосредоточиться на важных функциях приложения, переложив сквозные специализированные задачи на экспертов в соответствующей области.</span><span class="sxs-lookup"><span data-stu-id="cfd77-126">This allows your core team to focus on the application functionality, leaving these specialized but cross-cutting concerns to the relevant experts.</span></span>

- <span data-ttu-id="cfd77-127">Повышает согласованность при ведении журналов, а также при мониторинге запросов и ответов.</span><span class="sxs-lookup"><span data-stu-id="cfd77-127">Provide some consistency for request and response logging and monitoring.</span></span> <span data-ttu-id="cfd77-128">Даже если служба не располагает необходимыми инструментами, минимальный уровень мониторинга и ведения журнала можно настроить прямо в шлюзе.</span><span class="sxs-lookup"><span data-stu-id="cfd77-128">Even if a service is not correctly instrumented, the gateway can be configured to ensure a minimum level of monitoring and logging.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="cfd77-129">Проблемы и рекомендации</span><span class="sxs-lookup"><span data-stu-id="cfd77-129">Issues and considerations</span></span>

- <span data-ttu-id="cfd77-130">Следите за тем, чтобы для шлюза API поддерживалась высокая доступность и устойчивость к сбоям.</span><span class="sxs-lookup"><span data-stu-id="cfd77-130">Ensure the API gateway is highly available and resilient to failure.</span></span> <span data-ttu-id="cfd77-131">Избавляйтесь от единых точек отказа, создавая несколько экземпляров шлюза API.</span><span class="sxs-lookup"><span data-stu-id="cfd77-131">Avoid single points of failure by running multiple instances of your API gateway.</span></span>
- <span data-ttu-id="cfd77-132">Обеспечьте для шлюза достаточную емкость и возможность масштабирования в соответствии с потребностями приложений и конечных точек.</span><span class="sxs-lookup"><span data-stu-id="cfd77-132">Ensure the gateway is designed for the capacity and scaling requirements of your application and endpoints.</span></span> <span data-ttu-id="cfd77-133">Убедитесь, что шлюз не станет узким местом для приложения и его можно масштабировать до необходимого уровня.</span><span class="sxs-lookup"><span data-stu-id="cfd77-133">Make sure the gateway does not become a bottleneck for the application and is sufficiently scalable.</span></span>
- <span data-ttu-id="cfd77-134">Переносите только те компоненты, которые используются во всех частях приложения, такие как функции обеспечения безопасности и (или) передачи данных.</span><span class="sxs-lookup"><span data-stu-id="cfd77-134">Only offload features that are used by the entire application, such as security or data transfer.</span></span>
- <span data-ttu-id="cfd77-135">Ни в коем случае не переносите в шлюз API компоненты бизнес-логики.</span><span class="sxs-lookup"><span data-stu-id="cfd77-135">Business logic should never be offloaded to the API gateway.</span></span>
- <span data-ttu-id="cfd77-136">Чтобы отслеживать транзакции, рекомендуем внедрить идентификаторы корреляции для ведения журнала.</span><span class="sxs-lookup"><span data-stu-id="cfd77-136">If you need to track transactions, consider generating correlation IDs for logging purposes.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="cfd77-137">Когда следует использовать этот шаблон</span><span class="sxs-lookup"><span data-stu-id="cfd77-137">When to use this pattern</span></span>

<span data-ttu-id="cfd77-138">Используйте этот шаблон в следующих случаях:</span><span class="sxs-lookup"><span data-stu-id="cfd77-138">Use this pattern when:</span></span>

- <span data-ttu-id="cfd77-139">в развертывании приложения есть общий компонент, например сертификаты SSL или шифрование;</span><span class="sxs-lookup"><span data-stu-id="cfd77-139">An application deployment has a shared concern such as SSL certificates or encryption.</span></span>
- <span data-ttu-id="cfd77-140">есть общий для нескольких развертываний компонент, который может иметь разные требования к ресурсам, например к памяти, хранилищу или сетевым подключениям;</span><span class="sxs-lookup"><span data-stu-id="cfd77-140">A feature that is common across application deployments that may have different resource requirements, such as memory resources, storage capacity or network connections.</span></span>
- <span data-ttu-id="cfd77-141">нужно передать специализированной команде определенные задачи, например обеспечение сетевой безопасности, регулирование или другие проблемы границ сети.</span><span class="sxs-lookup"><span data-stu-id="cfd77-141">You wish to move the responsibility for issues such as network security, throttling, or other network boundary concerns to a more specialized team.</span></span>

<span data-ttu-id="cfd77-142">Такая схема может не подойти, если она добавляет новые взаимозависимости между службами.</span><span class="sxs-lookup"><span data-stu-id="cfd77-142">This pattern may not be suitable if it introduces coupling across services.</span></span>

## <a name="example"></a><span data-ttu-id="cfd77-143">Пример</span><span class="sxs-lookup"><span data-stu-id="cfd77-143">Example</span></span>

<span data-ttu-id="cfd77-144">В следующей конфигурации как устройство для разгрузки SSL используется Nginx. Он завершает входящее подключение SSL и распределяет запросы между тремя HTTP-серверами, размещенными дальше в сети.</span><span class="sxs-lookup"><span data-stu-id="cfd77-144">Using Nginx as the SSL offload appliance, the following configuration terminates an inbound SSL connection and distributes the connection to one of three upstream HTTP servers.</span></span>

```console
upstream iis {
        server  10.3.0.10    max_fails=3    fail_timeout=15s;
        server  10.3.0.20    max_fails=3    fail_timeout=15s;
        server  10.3.0.30    max_fails=3    fail_timeout=15s;
}

server {
        listen 443;
        ssl on;
        ssl_certificate /etc/nginx/ssl/domain.cer;
        ssl_certificate_key /etc/nginx/ssl/domain.key;

        location / {
                set $targ iis;
                proxy_pass http://$targ;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto https;
proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header Host $host;
        }
}
```

## <a name="related-guidance"></a><span data-ttu-id="cfd77-145">Связанные руководства</span><span class="sxs-lookup"><span data-stu-id="cfd77-145">Related guidance</span></span>

- [<span data-ttu-id="cfd77-146">Схема отдельных серверных частей для каждого интерфейса</span><span class="sxs-lookup"><span data-stu-id="cfd77-146">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- [<span data-ttu-id="cfd77-147">Схема агрегирования на шлюзе</span><span class="sxs-lookup"><span data-stu-id="cfd77-147">Gateway Aggregation pattern</span></span>](./gateway-aggregation.md)
- [<span data-ttu-id="cfd77-148">Схема маршрутизации шлюза</span><span class="sxs-lookup"><span data-stu-id="cfd77-148">Gateway Routing pattern</span></span>](./gateway-routing.md)
