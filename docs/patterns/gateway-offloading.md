---
title: "Схема разгрузки шлюза"
description: "Перенесение общих или специализированных функций служб на прокси-сервер шлюза."
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 6b3e4541aae77349ca91c18c788ddb508912361d
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
---
# <a name="gateway-offloading-pattern"></a><span data-ttu-id="2c9fb-103">Схема разгрузки шлюза</span><span class="sxs-lookup"><span data-stu-id="2c9fb-103">Gateway Offloading pattern</span></span>

<span data-ttu-id="2c9fb-104">Перенесение общих или специализированных функций служб на прокси-сервер шлюза.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-104">Offload shared or specialized service functionality to a gateway proxy.</span></span> <span data-ttu-id="2c9fb-105">Эта схема позволяет упростить разработку приложений, перемещая общие функции служб (например, использование SSL-сертификатов) из разных частей приложения в шлюз.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-105">This pattern can simplify application development by moving shared service functionality, such as the use of SSL certificates, from other parts of the application into the gateway.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="2c9fb-106">Контекст и проблема</span><span class="sxs-lookup"><span data-stu-id="2c9fb-106">Context and problem</span></span>

<span data-ttu-id="2c9fb-107">Некоторые компоненты традиционно используются сразу в нескольких службах. При этом для каждого из них нужно выполнять настройку, управление и обслуживание.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-107">Some features are commonly used across multiple services, and these features require configuration, management, and maintenance.</span></span> <span data-ttu-id="2c9fb-108">Общая или специализированная служба, которая распространяется вместе с каждым развертыванием приложения, увеличивает административные издержки и повышает вероятность ошибки при развертывании.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-108">A shared or specialized service that is distributed with every application deployment increases the administrative overhead and increases the likelihood of deployment error.</span></span> <span data-ttu-id="2c9fb-109">Все обновления общего компонента нужно развертывать во всех службах, в которых он используется.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-109">Any updates to a shared feature must be deployed across all services that share that feature.</span></span>

<span data-ttu-id="2c9fb-110">Правильное решение проблем безопасности (проверка маркера, шифрование, управление SSL-сертификатами) и других сложных задач требует высокой квалификации разработчиков.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-110">Properly handling security issues (token validation, encryption, SSL certificate management) and other complex tasks can require team members to have highly specialized skills.</span></span> <span data-ttu-id="2c9fb-111">Предположим, что необходимый для приложения сертификат нужно настраивать и развертывать во всех экземплярах приложения.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-111">For example, a certificate needed by an application must be configured and deployed on all application instances.</span></span> <span data-ttu-id="2c9fb-112">Приходится следить за каждым новым развертыванием, чтобы контролировать срок действия сертификата.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-112">With each new deployment, the certificate must be managed to ensure that it does not expire.</span></span> <span data-ttu-id="2c9fb-113">По истечении срока действия общего сертификата его нужно обновлять, проверять и утверждать отдельно в каждом развертывании приложения.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-113">Any common certificate that is due to expire must be updated, tested, and verified on every application deployment.</span></span>

<span data-ttu-id="2c9fb-114">Для большого количества развертываний достаточно сложно развертывать и поддерживать и другие распространенные службы, например отвечающие за аутентификацию, авторизацию, ведение журналов, мониторинг или [регулирование](./throttling.md).</span><span class="sxs-lookup"><span data-stu-id="2c9fb-114">Other common services such as authentication, authorization, logging, monitoring, or [throttling](./throttling.md) can be difficult to implement and manage across a large number of deployments.</span></span> <span data-ttu-id="2c9fb-115">Иногда такие компоненты следует консолидировать, чтобы снизить издержки и вероятность ошибок.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-115">It may be better to consolidate this type of functionality, in order to reduce overhead and the chance of errors.</span></span>

## <a name="solution"></a><span data-ttu-id="2c9fb-116">Решение</span><span class="sxs-lookup"><span data-stu-id="2c9fb-116">Solution</span></span>

<span data-ttu-id="2c9fb-117">Перенесите в шлюз API часть компонентов, в частности те, которые широко используются в приложении, например отвечают за управление сертификатами, аутентификацию, оконечную точку SSL, мониторинг, преобразование протоколов и регулирование.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-117">Offload some features into an API gateway, particularly cross-cutting concerns such as certificate management, authentication, SSL termination, monitoring, protocol translation, or throttling.</span></span> 

<span data-ttu-id="2c9fb-118">На следующей схеме представлен шлюз API, служащий оконечной точкой для входящих SSL-подключений.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-118">The following diagram shows an API gateway that terminates inbound SSL connections.</span></span> <span data-ttu-id="2c9fb-119">Он обращается от имени исходной запрашивающей стороны ко всем HTTP-серверам, размещенным за этим шлюзом API.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-119">It requests data on behalf of the original requestor from any HTTP server upstream of the API gateway.</span></span>

 ![](./_images/gateway-offload.png)
 
<span data-ttu-id="2c9fb-120">Такая схема предоставляет следующие преимущества:</span><span class="sxs-lookup"><span data-stu-id="2c9fb-120">Benefits of this pattern include:</span></span>

- <span data-ttu-id="2c9fb-121">Упрощает разработку служб, так как не нужно распространять и поддерживать для всех защищенных веб-узлов вспомогательные ресурсы, например сертификаты и настройки веб-сервера.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-121">Simplify the development of services by removing the need to distribute and maintain supporting resources, such as web server certificates and configuration for secure websites.</span></span> <span data-ttu-id="2c9fb-122">Упрощает настройку и управление, а также повышает масштабируемость и удобство обновления служб.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-122">Simpler configuration results in easier management and scalability and makes service upgrades simpler.</span></span>

- <span data-ttu-id="2c9fb-123">Позволяет поручить отдельным командам реализацию функций, требующих специальных навыков (например, в сфере систем безопасности).</span><span class="sxs-lookup"><span data-stu-id="2c9fb-123">Allow dedicated teams to implement features that require specialized expertise, such as security.</span></span> <span data-ttu-id="2c9fb-124">Благодаря этому основная команда разработчиков может сосредоточиться на важных функциях приложения, переложив сквозные специализированные задачи на экспертов в соответствующей области.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-124">This allows your core team to focus on the application functionality, leaving these specialized but cross-cutting concerns to the relevant experts.</span></span>

- <span data-ttu-id="2c9fb-125">Повышает согласованность при ведении журналов, а также при мониторинге запросов и ответов.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-125">Provide some consistency for request and response logging and monitoring.</span></span> <span data-ttu-id="2c9fb-126">Даже если служба не располагает необходимыми инструментами, минимальный уровень мониторинга и ведения журнала можно настроить прямо в шлюзе.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-126">Even if a service is not correctly instrumented, the gateway can be configured to ensure a minimum level of monitoring and logging.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="2c9fb-127">Проблемы и рекомендации</span><span class="sxs-lookup"><span data-stu-id="2c9fb-127">Issues and considerations</span></span>

- <span data-ttu-id="2c9fb-128">Следите за тем, чтобы для шлюза API поддерживалась высокая доступность и устойчивость к сбоям.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-128">Ensure the API gateway is highly available and resilient to failure.</span></span> <span data-ttu-id="2c9fb-129">Избавляйтесь от единых точек отказа, создавая несколько экземпляров шлюза API.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-129">Avoid single points of failure by running multiple instances of your API gateway.</span></span> 
- <span data-ttu-id="2c9fb-130">Обеспечьте для шлюза достаточную емкость и возможность масштабирования в соответствии с потребностями приложений и конечных точек.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-130">Ensure the gateway is designed for the capacity and scaling requirements of your application and endpoints.</span></span> <span data-ttu-id="2c9fb-131">Убедитесь, что шлюз не станет узким местом для приложения и его можно масштабировать до необходимого уровня.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-131">Make sure the gateway does not become a bottleneck for the application and is sufficiently scalable.</span></span>
- <span data-ttu-id="2c9fb-132">Переносите только те компоненты, которые используются во всех частях приложения, такие как функции обеспечения безопасности и (или) передачи данных.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-132">Only offload features that are used by the entire application, such as security or data transfer.</span></span>
- <span data-ttu-id="2c9fb-133">Ни в коем случае не переносите в шлюз API компоненты бизнес-логики.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-133">Business logic should never be offloaded to the API gateway.</span></span> 
- <span data-ttu-id="2c9fb-134">Чтобы отслеживать транзакции, рекомендуем внедрить идентификаторы корреляции для ведения журнала.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-134">If you need to track transactions, consider generating correlation IDs for logging purposes.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="2c9fb-135">Когда следует использовать этот шаблон</span><span class="sxs-lookup"><span data-stu-id="2c9fb-135">When to use this pattern</span></span>

<span data-ttu-id="2c9fb-136">Используйте эту схему в следующих случаях:</span><span class="sxs-lookup"><span data-stu-id="2c9fb-136">Use this pattern when:</span></span>

- <span data-ttu-id="2c9fb-137">в развертывании приложения есть общий компонент, например сертификаты SSL или шифрование;</span><span class="sxs-lookup"><span data-stu-id="2c9fb-137">An application deployment has a shared concern such as SSL certificates or encryption.</span></span>
- <span data-ttu-id="2c9fb-138">есть общий для нескольких развертываний компонент, который может иметь разные требования к ресурсам, например к памяти, хранилищу или сетевым подключениям;</span><span class="sxs-lookup"><span data-stu-id="2c9fb-138">A feature that is common across application deployments that may have different resource requirements, such as memory resources, storage capacity or network connections.</span></span>
- <span data-ttu-id="2c9fb-139">нужно передать специализированной команде определенные задачи, например обеспечение сетевой безопасности, регулирование или другие проблемы границ сети.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-139">You wish to move the responsibility for issues such as network security, throttling, or other network boundary concerns to a more specialized team.</span></span>

<span data-ttu-id="2c9fb-140">Такая схема может не подойти, если она добавляет новые взаимозависимости между службами.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-140">This pattern may not be suitable if it introduces coupling across services.</span></span>

## <a name="example"></a><span data-ttu-id="2c9fb-141">Пример</span><span class="sxs-lookup"><span data-stu-id="2c9fb-141">Example</span></span>

<span data-ttu-id="2c9fb-142">В следующей конфигурации как устройство для разгрузки SSL используется Nginx. Он завершает входящее подключение SSL и распределяет запросы между тремя HTTP-серверами, размещенными дальше в сети.</span><span class="sxs-lookup"><span data-stu-id="2c9fb-142">Using Nginx as the SSL offload appliance, the following configuration terminates an inbound SSL connection and distributes the connection to one of three upstream HTTP servers.</span></span>

```
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

## <a name="related-guidance"></a><span data-ttu-id="2c9fb-143">Связанные руководства</span><span class="sxs-lookup"><span data-stu-id="2c9fb-143">Related guidance</span></span>

- [<span data-ttu-id="2c9fb-144">Схема отдельных серверных частей для каждого интерфейса</span><span class="sxs-lookup"><span data-stu-id="2c9fb-144">Backends for Frontends pattern</span></span>](./backends-for-frontends.md)
- [<span data-ttu-id="2c9fb-145">Схема агрегирования на шлюзе</span><span class="sxs-lookup"><span data-stu-id="2c9fb-145">Gateway Aggregation pattern</span></span>](./gateway-aggregation.md)
- [<span data-ttu-id="2c9fb-146">Схема маршрутизации шлюза</span><span class="sxs-lookup"><span data-stu-id="2c9fb-146">Gateway Routing pattern</span></span>](./gateway-routing.md)

