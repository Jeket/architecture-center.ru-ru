---
title: Антишаблон неправильного создания экземпляров
titleSuffix: Performance antipatterns for cloud apps
description: Не следует постоянно создавать экземпляры объекта, который нужно создать раз, а затем использовать совместно.
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: a2e42e35ae1b56b61c8f9f9ecb21ee104cd3222e
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58346354"
---
# <a name="improper-instantiation-antipattern"></a><span data-ttu-id="2b8cb-103">Антишаблон неправильного создания экземпляров</span><span class="sxs-lookup"><span data-stu-id="2b8cb-103">Improper Instantiation antipattern</span></span>

<span data-ttu-id="2b8cb-104">Антишаблон неправильного создания экземпляров может снизить производительность из-за непрерывного создания экземпляров объекта, которые нужно создать раз, а затем использовать совместно.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-104">It can hurt performance to continually create new instances of an object that is meant to be created once and then shared.</span></span>

## <a name="problem-description"></a><span data-ttu-id="2b8cb-105">Описание проблемы</span><span class="sxs-lookup"><span data-stu-id="2b8cb-105">Problem description</span></span>

<span data-ttu-id="2b8cb-106">Во многих библиотеках представлены абстракции внешних ресурсов.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-106">Many libraries provide abstractions of external resources.</span></span> <span data-ttu-id="2b8cb-107">На внутреннем уровне эти классы обычно управляют собственными подключениями к ресурсу и выступают в качестве брокеров, которые клиенты могут использовать для доступа к ресурсу.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-107">Internally, these classes typically manage their own connections to the resource, acting as brokers that clients can use to access the resource.</span></span> <span data-ttu-id="2b8cb-108">Ниже приведены некоторые примеры классов брокера, относящиеся к приложениям Azure.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-108">Here are some examples of broker classes that are relevant to Azure applications:</span></span>

- <span data-ttu-id="2b8cb-109">`System.Net.Http.HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-109">`System.Net.Http.HttpClient`.</span></span> <span data-ttu-id="2b8cb-110">Взаимодействует с веб-службой, используя протокол HTTP.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-110">Communicates with a web service using HTTP.</span></span>
- <span data-ttu-id="2b8cb-111">`Microsoft.ServiceBus.Messaging.QueueClient`.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-111">`Microsoft.ServiceBus.Messaging.QueueClient`.</span></span> <span data-ttu-id="2b8cb-112">Публикует и получает сообщения в очереди служебной шины.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-112">Posts and receives messages to a Service Bus queue.</span></span>
- <span data-ttu-id="2b8cb-113">`Microsoft.Azure.Documents.Client.DocumentClient`.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-113">`Microsoft.Azure.Documents.Client.DocumentClient`.</span></span> <span data-ttu-id="2b8cb-114">Подключается к экземпляру Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-114">Connects to a Cosmos DB instance</span></span>
- <span data-ttu-id="2b8cb-115">`StackExchange.Redis.ConnectionMultiplexer`.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-115">`StackExchange.Redis.ConnectionMultiplexer`.</span></span> <span data-ttu-id="2b8cb-116">Подключается к Redis, включая кэш Redis для Azure.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-116">Connects to Redis, including Azure Redis Cache.</span></span>

<span data-ttu-id="2b8cb-117">Для этих классов экземпляры создаются единожды, а затем используются на протяжении всего времени существования приложения.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-117">These classes are intended to be instantiated once and reused throughout the lifetime of an application.</span></span> <span data-ttu-id="2b8cb-118">Однако распространенное заблуждение заключается в том, что эти классы нужно получать только по мере необходимости и быстро освобождать.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-118">However, it's a common misunderstanding that these classes should be acquired only as necessary and released quickly.</span></span> <span data-ttu-id="2b8cb-119">(Здесь перечислены библиотеки .NET, но шаблон не является уникальным для .NET.) В следующем примере ASP.NET создается экземпляр `HttpClient` для взаимодействия с удаленной службой.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-119">(The ones listed here happen to be .NET libraries, but the pattern is not unique to .NET.) The following ASP.NET example creates an instance of `HttpClient` to communicate with a remote service.</span></span> <span data-ttu-id="2b8cb-120">Полный пример см. [здесь][sample-app].</span><span class="sxs-lookup"><span data-stu-id="2b8cb-120">You can find the complete sample [here][sample-app].</span></span>

```csharp
public class NewHttpClientInstancePerRequestController : ApiController
{
    // This method creates a new instance of HttpClient and disposes it for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        using (var httpClient = new HttpClient())
        {
            var hostName = HttpContext.Current.Request.Url.Host;
            var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
            return new Product { Name = result };
        }
    }
}
```

<span data-ttu-id="2b8cb-121">В веб-приложении этот метод не является масштабируемым.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-121">In a web application, this technique is not scalable.</span></span> <span data-ttu-id="2b8cb-122">Объект `HttpClient` создается для каждого пользовательского запроса.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-122">A new `HttpClient` object is created for each user request.</span></span> <span data-ttu-id="2b8cb-123">В условиях большой нагрузки веб-сервер может исчерпать количество доступных сокетов, что приведет к ошибкам `SocketException`.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-123">Under heavy load, the web server may exhaust the number of available sockets, resulting in `SocketException` errors.</span></span>

<span data-ttu-id="2b8cb-124">Эта проблема не ограничивается классом `HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-124">This problem is not restricted to the `HttpClient` class.</span></span> <span data-ttu-id="2b8cb-125">Другие классы (затратные при создании или создающие программу-оболочку) могут вызывать аналогичные проблемы.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-125">Other classes that wrap resources or are expensive to create might cause similar issues.</span></span> <span data-ttu-id="2b8cb-126">В следующем примере создаются экземпляры класса `ExpensiveToCreateService`.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-126">The following example creates an instances of the `ExpensiveToCreateService` class.</span></span> <span data-ttu-id="2b8cb-127">Здесь проблема заключается не в исчерпании сокетов, а в том, сколько времени занимает создание каждого экземпляра.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-127">Here the issue is not necessarily socket exhaustion, but simply how long it takes to create each instance.</span></span> <span data-ttu-id="2b8cb-128">Непрерывное создание и уничтожение экземпляров этого класса может неблагоприятно повлиять на масштабируемость системы.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-128">Continually creating and destroying instances of this class might adversely affect the scalability of the system.</span></span>

```csharp
public class NewServiceInstancePerRequestController : ApiController
{
    public async Task<Product> GetProductAsync(string id)
    {
        var expensiveToCreateService = new ExpensiveToCreateService();
        return await expensiveToCreateService.GetProductByIdAsync(id);
    }
}

public class ExpensiveToCreateService
{
    public ExpensiveToCreateService()
    {
        // Simulate delay due to setup and configuration of ExpensiveToCreateService
        Thread.SpinWait(Int32.MaxValue / 100);
    }
    ...
}
```

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="2b8cb-129">Как устранить проблему</span><span class="sxs-lookup"><span data-stu-id="2b8cb-129">How to fix the problem</span></span>

<span data-ttu-id="2b8cb-130">Если класс, который выступает оболочкой для внешних ресурсов, потокобезопасный и предназначен для совместного использования, создайте общедоступный одноэлементный экземпляр или пул повторном используемых экземпляров этого класса.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-130">If the class that wraps the external resource is shareable and thread-safe, create a shared singleton instance or a pool of reusable instances of the class.</span></span>

<span data-ttu-id="2b8cb-131">В следующем примере используется статический экземпляр `HttpClient`, что позволяет запросам совместно использовать подключение.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-131">The following example uses a static `HttpClient` instance, thus sharing the connection across all requests.</span></span>

```csharp
public class SingleHttpClientInstanceController : ApiController
{
    private static readonly HttpClient httpClient;

    static SingleHttpClientInstanceController()
    {
        httpClient = new HttpClient();
    }

    // This method uses the shared instance of HttpClient for every call to GetProductAsync.
    public async Task<Product> GetProductAsync(string id)
    {
        var hostName = HttpContext.Current.Request.Url.Host;
        var result = await httpClient.GetStringAsync(string.Format("http://{0}:8080/api/...", hostName));
        return new Product { Name = result };
    }
}
```

## <a name="considerations"></a><span data-ttu-id="2b8cb-132">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="2b8cb-132">Considerations</span></span>

- <span data-ttu-id="2b8cb-133">Ключевым элементом этого антишаблона является многократное создание и уничтожение экземпляров объекта *совместного использования*.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-133">The key element of this antipattern is repeatedly creating and destroying instances of a *shareable* object.</span></span> <span data-ttu-id="2b8cb-134">Если класс не предназначен для общего использования (не потокобезопасен), то этот антишаблон не применяется.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-134">If a class is not shareable (not thread-safe), then this antipattern does not apply.</span></span>

- <span data-ttu-id="2b8cb-135">Тип общего ресурса может определять, следует ли использовать одноэлементный экземпляр или создавать пул.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-135">The type of shared resource might dictate whether you should use a singleton or create a pool.</span></span> <span data-ttu-id="2b8cb-136">Класс `HttpClient` предназначен для общего использования, а не применения в составе пула.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-136">The `HttpClient` class is designed to be shared rather than pooled.</span></span> <span data-ttu-id="2b8cb-137">Другие объекты могут поддерживать пулы, позволяя системе распределять рабочую нагрузку между несколькими экземплярами.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-137">Other objects might support pooling, enabling the system to spread the workload across multiple instances.</span></span>

- <span data-ttu-id="2b8cb-138">Объекты общего использования для нескольких запросов *должны* быть потокобезопасными.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-138">Objects that you share across multiple requests *must* be thread-safe.</span></span> <span data-ttu-id="2b8cb-139">Класс `HttpClient` предназначен для использования подобным образом, но другие классы могут не поддерживать одновременные запросы, поэтому нужно ознакомиться с доступной документацией.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-139">The `HttpClient` class is designed to be used in this manner, but other classes might not support concurrent requests, so check the available documentation.</span></span>

- <span data-ttu-id="2b8cb-140">Будьте осторожны, выбирая свойства для совместно используемых объектов, так как это может привести к состоянию гонки.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-140">Be careful about setting properties on shared objects, as this can lead to race conditions.</span></span> <span data-ttu-id="2b8cb-141">Например, настройка `DefaultRequestHeaders` для класса `HttpClient` перед каждым запросом может привести к состоянию гонки.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-141">For example, setting `DefaultRequestHeaders` on the `HttpClient` class before each request can create a race condition.</span></span> <span data-ttu-id="2b8cb-142">Настраивать такие параметры следует один раз (например, во время запуска). Создавайте отдельные экземпляры, если вам нужно настроить разные параметры.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-142">Set such properties once (for example, during startup), and create separate instances if you need to configure different settings.</span></span>

- <span data-ttu-id="2b8cb-143">Некоторые типы ресурсов могут быть дефицитными, и их не нужно ставить на удержание.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-143">Some resource types are scarce and should not be held onto.</span></span> <span data-ttu-id="2b8cb-144">Примером выступают подключения к базе данных.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-144">Database connections are an example.</span></span> <span data-ttu-id="2b8cb-145">Удерживание ненужного открытого подключения к базе данных может привести к тому, что другие параллельные пользователи не смогут получить доступ к базе данных.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-145">Holding an open database connection that is not required may prevent other concurrent users from gaining access to the database.</span></span>

- <span data-ttu-id="2b8cb-146">На платформе .NET Framework многие объекты, устанавливающие подключения ко внешним ресурсам, созданы путем использования статических фабричных методов других классов, которые управляют этими подключениями.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-146">In the .NET Framework, many objects that establish connections to external resources are created by using static factory methods of other classes that manage these connections.</span></span> <span data-ttu-id="2b8cb-147">Эти объекты должны храниться и использоваться повторно, а не удаляться и создаваться заново.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-147">These objects are intended to be saved and reused, rather than disposed and recreated.</span></span> <span data-ttu-id="2b8cb-148">Например, в служебной шине Azure объект `QueueClient` создается через объект `MessagingFactory`.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-148">For example, in Azure Service Bus, the `QueueClient` object is created through a `MessagingFactory` object.</span></span> <span data-ttu-id="2b8cb-149">На внутреннем уровне `MessagingFactory` управляет подключениями.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-149">Internally, the `MessagingFactory` manages connections.</span></span> <span data-ttu-id="2b8cb-150">Дополнительные сведения см. в статье [Рекомендации по повышению производительности с помощью обмена сообщениями через служебную шину][service-bus-messaging].</span><span class="sxs-lookup"><span data-stu-id="2b8cb-150">For more information, see [Best Practices for performance improvements using Service Bus Messaging][service-bus-messaging].</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="2b8cb-151">Как определить проблему</span><span class="sxs-lookup"><span data-stu-id="2b8cb-151">How to detect the problem</span></span>

<span data-ttu-id="2b8cb-152">Признаки этой проблемы включают в себя снижение пропускной способности, повышенную частоту ошибок, а также несколько следующих признаков:</span><span class="sxs-lookup"><span data-stu-id="2b8cb-152">Symptoms of this problem include a drop in throughput or an increased error rate, along with one or more of the following:</span></span>

- <span data-ttu-id="2b8cb-153">увеличение исключений, которые указывают на нехватку системных ресурсов (например, сокетов, подключений к базе данных, дескрипторов файлов и т. д.);</span><span class="sxs-lookup"><span data-stu-id="2b8cb-153">An increase in exceptions that indicate exhaustion of resources such as sockets, database connections, file handles, and so on.</span></span>
- <span data-ttu-id="2b8cb-154">повышение потребления памяти и сборки мусора;</span><span class="sxs-lookup"><span data-stu-id="2b8cb-154">Increased memory use and garbage collection.</span></span>
- <span data-ttu-id="2b8cb-155">увеличение активности базы данных, диска или сети.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-155">An increase in network, disk, or database activity.</span></span>

<span data-ttu-id="2b8cb-156">Чтобы определить эту проблему, сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="2b8cb-156">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="2b8cb-157">Выполните мониторинг обработки в рабочей системе, чтобы определить точки, когда время отклика замедляется или происходит сбой системы из-за нехватки ресурсов.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-157">Performing process monitoring of the production system, to identify points when response times slow down or the system fails due to lack of resources.</span></span>
2. <span data-ttu-id="2b8cb-158">Проанализируйте данные телеметрии, полученные в этих точках, чтобы определить, какие операции могут создавать и уничтожать ресурсоемкие объекты.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-158">Examine the telemetry data captured at these points to determine which operations might be creating and destroying resource-consuming objects.</span></span>
3. <span data-ttu-id="2b8cb-159">Проведите нагрузочный тест для каждой подозрительной операции в управляемой тестовой среде, а не в рабочей системе.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-159">Load test each suspected operation, in a controlled test environment rather than the production system.</span></span>
4. <span data-ttu-id="2b8cb-160">Просмотрите исходный код и определите способ управления объектами брокера.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-160">Review the source code and examine the how broker objects are managed.</span></span>

<span data-ttu-id="2b8cb-161">Просмотрите в трассировках стека, нет ли операций, которые выполняются медленно или создают исключения, когда система находится под нагрузкой.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-161">Look at stack traces for operations that are slow-running or that generate exceptions when the system is under load.</span></span> <span data-ttu-id="2b8cb-162">Эти сведения помогут вам определить, как эти операции используют ресурсы.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-162">This information can help to identify how these operations are using resources.</span></span> <span data-ttu-id="2b8cb-163">С помощью этих исключений можно узнать, стало ли причиной возникновения ошибок исчерпание общих ресурсов.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-163">Exceptions can help to determine whether errors are caused by shared resources being exhausted.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="2b8cb-164">Пример диагностики</span><span class="sxs-lookup"><span data-stu-id="2b8cb-164">Example diagnosis</span></span>

<span data-ttu-id="2b8cb-165">В следующих разделах эти шаги применяются к примеру приложения, описанному ранее.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-165">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="identify-points-of-slow-down-or-failure"></a><span data-ttu-id="2b8cb-166">Определение точек замедления или сбоя</span><span class="sxs-lookup"><span data-stu-id="2b8cb-166">Identify points of slow down or failure</span></span>

<span data-ttu-id="2b8cb-167">На следующем рисунке показаны результаты, сформированные с помощью [New Relic APM][new-relic], в которых отображены операции, имеющие высокое время отклика.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-167">The following image shows results generated using [New Relic APM][new-relic], showing operations that have a poor response time.</span></span> <span data-ttu-id="2b8cb-168">В этом случае нужно дополнительно изучить метод `GetProductAsync` в контроллере `NewHttpClientInstancePerRequest`.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-168">In this case, the `GetProductAsync` method in the `NewHttpClientInstancePerRequest` controller is worth investigating further.</span></span> <span data-ttu-id="2b8cb-169">Обратите внимание, что частота ошибок также увеличивается при выполнении операций.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-169">Notice that the error rate also increases when these operations are running.</span></span>

![Панель мониторинга New Relic, где показан пример приложения, создающий экземпляр объекта HttpClient для каждого запроса][dashboard-new-HTTPClient-instance]

### <a name="examine-telemetry-data-and-find-correlations"></a><span data-ttu-id="2b8cb-171">Изучение данных телеметрии и поиск связи</span><span class="sxs-lookup"><span data-stu-id="2b8cb-171">Examine telemetry data and find correlations</span></span>

<span data-ttu-id="2b8cb-172">На следующем рисунке показаны данные, полученные с помощью профилирования потока, за период времени, соответствующие предыдущему рисунку.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-172">The next image shows data captured using thread profiling, over the same period corresponding as the previous image.</span></span> <span data-ttu-id="2b8cb-173">Система тратит немало времени, открывая подключения к сокетам, и еще больше времени, закрывая их и обрабатывая исключения.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-173">The system spends a significant time opening socket connections, and even more time closing them and handling socket exceptions.</span></span>

![Профилировщик потоков New Relic: пример приложения, создающего экземпляр объекта HttpClient для каждого запроса][thread-profiler-new-HTTPClient-instance]

### <a name="performing-load-testing"></a><span data-ttu-id="2b8cb-175">Выполнение нагрузочного тестирования</span><span class="sxs-lookup"><span data-stu-id="2b8cb-175">Performing load testing</span></span>

<span data-ttu-id="2b8cb-176">Нагрузочное тестирование следует использовать для имитации стандартных операций, которые могут выполнять пользователи.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-176">Use load testing to simulate the typical operations that users might perform.</span></span> <span data-ttu-id="2b8cb-177">Оно позволяет определить, какие части системы пострадали из-за исчерпания ресурсов под различными нагрузками.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-177">This can help to identify which parts of a system suffer from resource exhaustion under varying loads.</span></span> <span data-ttu-id="2b8cb-178">Выполняйте эти тесты в управляемой среде, а не в рабочей системе.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-178">Perform these tests in a controlled environment rather than the production system.</span></span> <span data-ttu-id="2b8cb-179">На следующем графике показана пропускная способность запросов, обрабатываемых контроллером `NewHttpClientInstancePerRequest`, при увеличении пользовательской нагрузки до 100 одновременных пользователей.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-179">The following graph shows the throughput of requests handled by the `NewHttpClientInstancePerRequest` controller as the user load increases to 100 concurrent users.</span></span>

![Пропускная способность примера приложения, создающего экземпляр объекта HttpClient для каждого запроса][throughput-new-HTTPClient-instance]

<span data-ttu-id="2b8cb-181">Сначала объем выполняемых в секунду запросов возрастает по мере увеличения рабочей нагрузки.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-181">At first, the volume of requests handled per second increases as the workload increases.</span></span> <span data-ttu-id="2b8cb-182">Однако, когда имеется примерно 30 пользователей, объем успешных запросов достигает предела и система начинает создавать исключения.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-182">At about 30 users, however, the volume of successful requests reaches a limit, and the system starts to generate exceptions.</span></span> <span data-ttu-id="2b8cb-183">После этого объем исключений постепенно возрастает вместе с пользовательской нагрузкой.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-183">From then on, the volume of exceptions gradually increases with the user load.</span></span>

<span data-ttu-id="2b8cb-184">В результатах нагрузочного тестирования эти сбои указаны как ошибки HTTP 500 (внутренний сервер).</span><span class="sxs-lookup"><span data-stu-id="2b8cb-184">The load test reported these failures as HTTP 500 (Internal Server) errors.</span></span> <span data-ttu-id="2b8cb-185">Анализ телеметрии показал, что эти ошибки были вызваны тем, что система исчерпала ресурсы сокетов, поскольку создавалось все больше объектов `HttpClient`.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-185">Reviewing the telemetry showed that these errors were caused by the system running out of socket resources, as more and more `HttpClient` objects were created.</span></span>

<span data-ttu-id="2b8cb-186">На следующем графике показан подобный тест для контроллера, который создает настраиваемый объект `ExpensiveToCreateService`.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-186">The next graph shows a similar test for a controller that creates the custom `ExpensiveToCreateService` object.</span></span>

![Пропускная способность примера приложения, создающего экземпляр объекта ExpensiveToCreateService для каждого запроса][throughput-new-ExpensiveToCreateService-instance]

<span data-ttu-id="2b8cb-188">На этот раз контроллер не создает никаких исключений, но пропускная способность все еще достигает плато, тогда как среднее время отклика увеличивается в 20 раз.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-188">This time, the controller does not generate any exceptions, but throughput still reaches a plateau, while the average response time increases by a factor of 20.</span></span> <span data-ttu-id="2b8cb-189">(На этом графике используется логарифмическая шкала для времени отклика и пропускной способности.) В данных телеметрии показано, что создание экземпляров `ExpensiveToCreateService` было главной причиной возникновения проблемы.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-189">(The graph uses a logarithmic scale for response time and throughput.) Telemetry showed that creating new instances of the `ExpensiveToCreateService` was the main cause of the problem.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="2b8cb-190">Реализация решения и проверка результатов</span><span class="sxs-lookup"><span data-stu-id="2b8cb-190">Implement the solution and verify the result</span></span>

<span data-ttu-id="2b8cb-191">После переключения метода `GetProductAsync` для совместного использования одного экземпляра `HttpClient` результаты второго нагрузочного тестирования показали повышение производительности.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-191">After switching the `GetProductAsync` method to share a single `HttpClient` instance, a second load test showed improved performance.</span></span> <span data-ttu-id="2b8cb-192">Ошибки не найдены, и система смогла обработать увеличение нагрузки до 500 запросов в секунду.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-192">No errors were reported, and the system was able to handle an increasing load of up to 500 requests per second.</span></span> <span data-ttu-id="2b8cb-193">Среднее время отклика было сокращено вдвое по сравнению с предыдущим тестом.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-193">The average response time was cut in half, compared with the previous test.</span></span>

![Пропускная способность примера приложения, которое повторно использует один и тот же экземпляр объекта HttpClient для каждого запроса][throughput-single-HTTPClient-instance]

<span data-ttu-id="2b8cb-195">Для сравнения на следующем рисунке показана телеметрия трассировки стека.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-195">For comparison, the following image shows the stack trace telemetry.</span></span> <span data-ttu-id="2b8cb-196">На этот раз система тратит большую часть времени на выполнение реальной работы, а не открывает и закрывает сокеты.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-196">This time, the system spends most of its time performing real work, rather than opening and closing sockets.</span></span>

![Профилировщик потоков New Relic: пример приложения, создающего один экземпляр объекта HttpClient для всех запросов][thread-profiler-single-HTTPClient-instance]

<span data-ttu-id="2b8cb-198">На следующем графике показано такое же нагрузочное тестирование с использованием общего экземпляра объекта `ExpensiveToCreateService`.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-198">The next graph shows a similar load test using a shared instance of the `ExpensiveToCreateService` object.</span></span> <span data-ttu-id="2b8cb-199">Объем обработанных запросов увеличивается в соответствии с пользовательской нагрузкой, тогда как среднее время отклика остается низким.</span><span class="sxs-lookup"><span data-stu-id="2b8cb-199">Again, the volume of handled requests increases in line with the user load, while the average response time remains low.</span></span>

![Пропускная способность примера приложения, которое повторно использует один и тот же экземпляр объекта HttpClient для каждого запроса][throughput-single-ExpensiveToCreateService-instance]

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/ImproperInstantiation
[service-bus-messaging]: /azure/service-bus-messaging/service-bus-performance-improvements
[new-relic]: https://newrelic.com/application-monitoring
[throughput-new-HTTPClient-instance]: _images/HttpClientInstancePerRequest.jpg
[dashboard-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestWebTransactions.jpg
[thread-profiler-new-HTTPClient-instance]: _images/HttpClientInstancePerRequestThreadProfile.jpg
[throughput-new-ExpensiveToCreateService-instance]: _images/ServiceInstancePerRequest.jpg
[throughput-single-HTTPClient-instance]: _images/SingleHttpClientInstance.jpg
[throughput-single-ExpensiveToCreateService-instance]: _images/SingleServiceInstance.jpg
[thread-profiler-single-HTTPClient-instance]: _images/SingleHttpClientInstanceThreadProfile.jpg
