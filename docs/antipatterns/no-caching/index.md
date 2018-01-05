---
title: "Антишаблон без кэширования"
description: "Многократная выборка тех же данных может снизить производительность и масштабируемость."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: 8a2bc3b473a30536cc1bef9e1dcad87acb46c4a9
ms.sourcegitcommit: c93f1b210b3deff17cc969fb66133bc6399cfd10
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/05/2018
---
# <a name="no-caching-antipattern"></a><span data-ttu-id="fb130-103">Антишаблон без кэширования</span><span class="sxs-lookup"><span data-stu-id="fb130-103">No Caching antipattern</span></span>

<span data-ttu-id="fb130-104">В облачном приложении, которое обрабатывает много одновременных запросов, многократная выборка тех же данных может снизить производительность и масштабируемость.</span><span class="sxs-lookup"><span data-stu-id="fb130-104">In a cloud application that handles many concurrent requests, repeatedly fetching the same data can reduce performance and scalability.</span></span> 

## <a name="problem-description"></a><span data-ttu-id="fb130-105">Описание проблемы</span><span class="sxs-lookup"><span data-stu-id="fb130-105">Problem description</span></span>

<span data-ttu-id="fb130-106">Если данные не кэшированные, это может привести к ряду нежелательных реакций на события, включая:</span><span class="sxs-lookup"><span data-stu-id="fb130-106">When data is not cached, it can cause a number of undesirable behaviors, including:</span></span>

- <span data-ttu-id="fb130-107">многократную выборку тех же данных из ресурса, доступ к которому является дорогостоящим с точки зрения дополнительных операций ввода-вывода или увеличенной задержки;</span><span class="sxs-lookup"><span data-stu-id="fb130-107">Repeatedly fetching the same information from a resource that is expensive to access, in terms of I/O overhead or latency.</span></span>
- <span data-ttu-id="fb130-108">многократное создание тех же объектов или структур данных для нескольких запросов;</span><span class="sxs-lookup"><span data-stu-id="fb130-108">Repeatedly constructing the same objects or data structures for multiple requests.</span></span>
- <span data-ttu-id="fb130-109">отправка множественных вызовов к удаленной службе, имеющей квоту службы и регулирующей количество клиентов при превышении лимита.</span><span class="sxs-lookup"><span data-stu-id="fb130-109">Making excessive calls to a remote service that has a service quota and throttles clients past a certain limit.</span></span>

<span data-ttu-id="fb130-110">В свою очередь, эти проблемы могут привести к высокому времени отклика, увеличению конфликтов в хранилище данных и низкой масштабируемости.</span><span class="sxs-lookup"><span data-stu-id="fb130-110">In turn, these problems can lead to poor response times, increased contention in the data store, and poor scalability.</span></span>

<span data-ttu-id="fb130-111">В следующем примере для подключения к базе данных используется платформа Entity Framework.</span><span class="sxs-lookup"><span data-stu-id="fb130-111">The following example uses Entity Framework to connect to a database.</span></span> <span data-ttu-id="fb130-112">Каждый запрос клиента приводит к вызову базы данных, даже если несколько запросов получают одинаковые данные.</span><span class="sxs-lookup"><span data-stu-id="fb130-112">Every client request results in a call to the database, even if multiple requests are fetching exactly the same data.</span></span> <span data-ttu-id="fb130-113">Затраты на повторные запросы могут быстро накапливаться в связи с множественными запросами ввода-вывода и расходами на доступ к данным.</span><span class="sxs-lookup"><span data-stu-id="fb130-113">The cost of repeated requests, in terms of I/O overhead and data access charges, can accumulate quickly.</span></span>

```csharp
public class PersonRepository : IPersonRepository
{
    public async Task<Person> GetAsync(int id)
    {
        using (var context = new AdventureWorksContext())
        {
            return await context.People
                .Where(p => p.Id == id)
                .FirstOrDefaultAsync()
                .ConfigureAwait(false);
        }
    }
}
```

<span data-ttu-id="fb130-114">Полный пример см. [здесь][sample-app].</span><span class="sxs-lookup"><span data-stu-id="fb130-114">You can find the complete sample [here][sample-app].</span></span>

<span data-ttu-id="fb130-115">Этот антишаблон обычно возникает в следующих случаях:</span><span class="sxs-lookup"><span data-stu-id="fb130-115">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="fb130-116">Не использовать кэш проще и это не влияет на работу при небольших нагрузках.</span><span class="sxs-lookup"><span data-stu-id="fb130-116">Not using a cache is simpler to implement, and it works fine under low loads.</span></span> <span data-ttu-id="fb130-117">Кэширование усложняет код.</span><span class="sxs-lookup"><span data-stu-id="fb130-117">Caching makes the code more complicated.</span></span> 
- <span data-ttu-id="fb130-118">Преимущества и недостатки использования кэша неочевидны.</span><span class="sxs-lookup"><span data-stu-id="fb130-118">The benefits and drawbacks of using a cache are not clearly understood.</span></span>
- <span data-ttu-id="fb130-119">Имеется опасение по поводу дополнительных затрат на поддержание точности и актуальности кэшированных данных.</span><span class="sxs-lookup"><span data-stu-id="fb130-119">There is concern about the overhead of maintaining the accuracy and freshness of cached data.</span></span>
- <span data-ttu-id="fb130-120">Приложение было перенесено из локальной системы, где задержка сети не являлась проблемой и система работала на затратном высокопроизводительном оборудовании, из-за чего кэширование не было учтено в исходной разработке.</span><span class="sxs-lookup"><span data-stu-id="fb130-120">An application was migrated from an on-premises system, where network latency was not an issue, and the system ran on expensive high-performance hardware, so caching wasn't considered in the original design.</span></span>
- <span data-ttu-id="fb130-121">Разработчики не знают о возможности выполнения кэширования в этом сценарии.</span><span class="sxs-lookup"><span data-stu-id="fb130-121">Developers aren't aware that caching is a possibility in a given scenario.</span></span> <span data-ttu-id="fb130-122">Например, разработчики могут не подумать об использовании ETag (тегов сущности) при реализации веб-интерфейса API.</span><span class="sxs-lookup"><span data-stu-id="fb130-122">For example, developers may not think of using ETags when implementing a web API.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="fb130-123">Как устранить проблему</span><span class="sxs-lookup"><span data-stu-id="fb130-123">How to fix the problem</span></span>

<span data-ttu-id="fb130-124">Самые распространенные стратегии кэширования: *по запросу* или *кэш на стороне*.</span><span class="sxs-lookup"><span data-stu-id="fb130-124">The most popular caching strategy is the *on-demand* or *cache-aside* strategy.</span></span>

- <span data-ttu-id="fb130-125">В процессе чтения приложение пытается считать данные из кэша.</span><span class="sxs-lookup"><span data-stu-id="fb130-125">On read, the application tries to read the data from the cache.</span></span> <span data-ttu-id="fb130-126">Если данные находятся не в кэше, приложение извлекает их из источника данных и добавляет в кэш.</span><span class="sxs-lookup"><span data-stu-id="fb130-126">If the data isn't in the cache, the application retrieves it from the data source and adds it to the cache.</span></span>
- <span data-ttu-id="fb130-127">В процессе записи приложение записывает измененные данные непосредственно в источник данных и удаляет старые значения из кэша.</span><span class="sxs-lookup"><span data-stu-id="fb130-127">On write, the application writes the change directly to the data source and removes the old value from the cache.</span></span> <span data-ttu-id="fb130-128">Они будут извлечены и добавлены в кэш в следующий раз (когда понадобится).</span><span class="sxs-lookup"><span data-stu-id="fb130-128">It will be retrieved and added to the cache the next time it is required.</span></span>

<span data-ttu-id="fb130-129">Этот подход используется для данных, которые часто изменяются.</span><span class="sxs-lookup"><span data-stu-id="fb130-129">This approach is suitable for data that changes frequently.</span></span> <span data-ttu-id="fb130-130">Ниже приведен предыдущий пример, однако теперь в нем используется шаблон "кэш на стороне".</span><span class="sxs-lookup"><span data-stu-id="fb130-130">Here is the previous example updated to use the [Cache-Aside][cache-aside] pattern.</span></span>  

```csharp
public class CachedPersonRepository : IPersonRepository
{
    private readonly PersonRepository _innerRepository;

    public CachedPersonRepository(PersonRepository innerRepository)
    {
        _innerRepository = innerRepository;
    }

    public async Task<Person> GetAsync(int id)
    {
        return await CacheService.GetAsync<Person>("p:" + id, () => _innerRepository.GetAsync(id)).ConfigureAwait(false);
    }
}

public class CacheService
{
    private static ConnectionMultiplexer _connection;

    public static async Task<T> GetAsync<T>(string key, Func<Task<T>> loadCache, double expirationTimeInMinutes)
    {
        IDatabase cache = Connection.GetDatabase();
        T value = await GetAsync<T>(cache, key).ConfigureAwait(false);
        if (value == null)
        {
            // Value was not found in the cache. Call the lambda to get the value from the database.
            value = await loadCache().ConfigureAwait(false);
            if (value != null)
            {
                // Add the value to the cache.
                await SetAsync(cache, key, value, expirationTimeInMinutes).ConfigureAwait(false);
            }
        }
        return value;
    }
}
```

<span data-ttu-id="fb130-131">Обратите внимание, что теперь метод `GetAsync` вызывает класс `CacheService` вместо непосредственного вызова базы данных.</span><span class="sxs-lookup"><span data-stu-id="fb130-131">Notice that the `GetAsync` method now calls the `CacheService` class, rather than calling the database directly.</span></span> <span data-ttu-id="fb130-132">Класс `CacheService` сначала пытается получить элемент из кэша Redis для Azure.</span><span class="sxs-lookup"><span data-stu-id="fb130-132">The `CacheService` class first tries to get the item from Azure Redis Cache.</span></span> <span data-ttu-id="fb130-133">Если значение не найдено в кэше Redis, `CacheService` вызывает лямбда-функцию, полученную от вызывающей стороны.</span><span class="sxs-lookup"><span data-stu-id="fb130-133">If the value isn't found in Redis Cache, the `CacheService` invokes a lambda function that was passed to it by the caller.</span></span> <span data-ttu-id="fb130-134">Лямбда-функция отвечает за получение данных из базы данных.</span><span class="sxs-lookup"><span data-stu-id="fb130-134">The lambda function is responsible for fetching the data from the database.</span></span> <span data-ttu-id="fb130-135">Эта реализация отделяет репозиторий от конкретного решения кэширования и разрывает связь между `CacheService` и базой данных.</span><span class="sxs-lookup"><span data-stu-id="fb130-135">This implementation decouples the repository from the particular caching solution, and decouples the `CacheService` from the database.</span></span> 

## <a name="considerations"></a><span data-ttu-id="fb130-136">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="fb130-136">Considerations</span></span>

- <span data-ttu-id="fb130-137">Кэш может быть недоступным из-за временного сбоя. В таком случае не возвращайте ошибку клиенту.</span><span class="sxs-lookup"><span data-stu-id="fb130-137">If the cache is unavailable, perhaps because of a transient failure, don't return an error to the client.</span></span> <span data-ttu-id="fb130-138">Вместо этого получите данные из исходного источника данных.</span><span class="sxs-lookup"><span data-stu-id="fb130-138">Instead, fetch the data from the original data source.</span></span> <span data-ttu-id="fb130-139">Имейте в виду, что пока кэш восстанавливается, исходное хранилище данных может быть загружено множеством запросов, что приведет к увеличению времени ожидания и к сбоям соединения.</span><span class="sxs-lookup"><span data-stu-id="fb130-139">However, be aware that while the cache is being recovered, the original data store could be swamped with requests, resulting in timeouts and failed connections.</span></span> <span data-ttu-id="fb130-140">(В конечном итоге, это и есть одна из основных причин использования кэша.) Чтобы избежать перегрузки источника данных, используйте [шаблон прерывателя][circuit-breaker].</span><span class="sxs-lookup"><span data-stu-id="fb130-140">(After all, this is one of the motivations for using a cache in the first place.) Use a technique such as the [Circuit Breaker pattern][circuit-breaker] to avoid overwhelming the data source.</span></span>

- <span data-ttu-id="fb130-141">Приложения, которые кэшируют нестатические данные, должны поддерживать итоговую согласованность.</span><span class="sxs-lookup"><span data-stu-id="fb130-141">Applications that cache nonstatic data should be designed to support eventual consistency.</span></span>

- <span data-ttu-id="fb130-142">Поддержку кэширования на стороне клиента для веб-API можно включить, добавив заголовок Cache-Control в сообщения запроса и ответа, а также используя теги eTag для идентификации версий объектов.</span><span class="sxs-lookup"><span data-stu-id="fb130-142">For web APIs, you can support client-side caching by including a Cache-Control header in request and response messages, and using ETags to identify versions of objects.</span></span> <span data-ttu-id="fb130-143">Дополнительные сведения см. в статье [Реализация API][api-implementation].</span><span class="sxs-lookup"><span data-stu-id="fb130-143">For more information, see [API implementation][api-implementation].</span></span>

- <span data-ttu-id="fb130-144">Нет необходимости выполнять кэширование всей сущности.</span><span class="sxs-lookup"><span data-stu-id="fb130-144">You don't have to cache entire entities.</span></span> <span data-ttu-id="fb130-145">Если большая часть сущности статична, а другая ее часть часто изменяется, выполните кэширование статических элементов и получите динамические элементы из источника данных.</span><span class="sxs-lookup"><span data-stu-id="fb130-145">If most of an entity is static but only a small piece changes frequently, cache the static elements and retrieve the dynamic elements from the data source.</span></span> <span data-ttu-id="fb130-146">Такой подход позволяет сократить объем операций ввода-вывода, выполняемых в источнике данных.</span><span class="sxs-lookup"><span data-stu-id="fb130-146">This approach can help to reduce the volume of I/O being performed against the data source.</span></span>

- <span data-ttu-id="fb130-147">При наличии непостоянных кратковременных данных будет целесообразно кэшировать их.</span><span class="sxs-lookup"><span data-stu-id="fb130-147">In some cases, if volatile data is short-lived, it can be useful to cache it.</span></span> <span data-ttu-id="fb130-148">Например, рассмотрим устройство, которое постоянно отправляет обновления состояния.</span><span class="sxs-lookup"><span data-stu-id="fb130-148">For example, consider a device that continually sends status updates.</span></span> <span data-ttu-id="fb130-149">Может быть целесообразно кэшировать эту информацию по мере поступления и не записывать ее в постоянное хранилище.</span><span class="sxs-lookup"><span data-stu-id="fb130-149">It might make sense to cache this information as it arrives, and not write it to a persistent store at all.</span></span>  

- <span data-ttu-id="fb130-150">Чтобы предотвратить устаревание данных, многие решения кэширования поддерживают настраиваемые сроки действия, поэтому данные автоматически удаляются из кэша через указанное время.</span><span class="sxs-lookup"><span data-stu-id="fb130-150">To prevent data from becoming stale, many caching solutions support configurable expiration periods, so that data is automatically removed from the cache after a specified interval.</span></span> <span data-ttu-id="fb130-151">В вашем сценарии может потребоваться настроить время истечения срока действия.</span><span class="sxs-lookup"><span data-stu-id="fb130-151">You may need to tune the expiration time for your scenario.</span></span> <span data-ttu-id="fb130-152">Высокостатические данные могут оставаться в кэше дольше, чем непостоянные данные, которые могут быстро устареть.</span><span class="sxs-lookup"><span data-stu-id="fb130-152">Data that is highly static can stay in the cache for longer periods than volatile data that may become stale quickly.</span></span>

- <span data-ttu-id="fb130-153">Если в решении для кэширования не установлен период истечения срока действия, может потребоваться реализовать фоновый процесс, который периодически выполняет очистку кэша, чтобы предотвратить бесконечное увеличение его объема.</span><span class="sxs-lookup"><span data-stu-id="fb130-153">If the caching solution doesn't provide built-in expiration, you may need to implement a background process that occasionally sweeps the cache, to prevent it from growing without limits.</span></span> 

- <span data-ttu-id="fb130-154">Помимо кэширования данных из внешнего источника данных, можно использовать кэширование для сохранения результатов сложных вычислений.</span><span class="sxs-lookup"><span data-stu-id="fb130-154">Besides caching data from an external data source, you can use caching to save the results of complex computations.</span></span> <span data-ttu-id="fb130-155">Однако, прежде чем делать это, инструментируйте приложение, чтобы определить, зависит ли оно от ЦП.</span><span class="sxs-lookup"><span data-stu-id="fb130-155">Before you do that, however, instrument the application to determine whether the application is really CPU bound.</span></span>

- <span data-ttu-id="fb130-156">Возможно, целесообразно очистить кэш при запуске приложения.</span><span class="sxs-lookup"><span data-stu-id="fb130-156">It might be useful to prime the cache when the application starts.</span></span> <span data-ttu-id="fb130-157">Заполните кэш данными, которые с большой вероятностью будут использоваться.</span><span class="sxs-lookup"><span data-stu-id="fb130-157">Populate the cache with the data that is most likely to be used.</span></span>

- <span data-ttu-id="fb130-158">Всегда включайте инструментирование, которое обнаруживает попадания и промахи в кэше.</span><span class="sxs-lookup"><span data-stu-id="fb130-158">Always include instrumentation that detects cache hits and cache misses.</span></span> <span data-ttu-id="fb130-159">Используйте эти сведения для настройки политик кэширования (например, данные для кэширования и период хранения данных в кэше до истечения их срока действия).</span><span class="sxs-lookup"><span data-stu-id="fb130-159">Use this information to tune caching policies, such what data to cache, and how long to hold data in the cache before it expires.</span></span>

- <span data-ttu-id="fb130-160">Если отсутствие кэширования является узким местом, то добавление кэширования может увеличить объем запросов настолько, что веб-интерфейс будет перегружен.</span><span class="sxs-lookup"><span data-stu-id="fb130-160">If the lack of caching is a bottleneck, then adding caching may increase the volume of requests so much that the web front end becomes overloaded.</span></span> <span data-ttu-id="fb130-161">Клиенты могут начать получать ошибки HTTP 503 (служба недоступна).</span><span class="sxs-lookup"><span data-stu-id="fb130-161">Clients may start to receive HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="fb130-162">Это означает, что нужно развернуть внешний интерфейс.</span><span class="sxs-lookup"><span data-stu-id="fb130-162">These are an indication that you should scale out the front end.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="fb130-163">Как определить проблему</span><span class="sxs-lookup"><span data-stu-id="fb130-163">How to detect the problem</span></span>

<span data-ttu-id="fb130-164">Чтобы определить, вызывает ли отсутствие кэширования проблему с производительностью, сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="fb130-164">You can perform the following steps to help identify whether lack of caching is causing performance problems:</span></span>

1. <span data-ttu-id="fb130-165">Просмотрите проект приложения.</span><span class="sxs-lookup"><span data-stu-id="fb130-165">Review the application design.</span></span> <span data-ttu-id="fb130-166">Проведите инвентаризацию всех хранилищ данных, используемых приложением.</span><span class="sxs-lookup"><span data-stu-id="fb130-166">Take an inventory of all the data stores that the application uses.</span></span> <span data-ttu-id="fb130-167">Для каждого из них определите, использует ли приложение кэш.</span><span class="sxs-lookup"><span data-stu-id="fb130-167">For each, determine whether the application is using a cache.</span></span> <span data-ttu-id="fb130-168">Если возможно, определите, как часто изменяются данные.</span><span class="sxs-lookup"><span data-stu-id="fb130-168">If possible, determine how frequently the data changes.</span></span> <span data-ttu-id="fb130-169">Подходящие исходные кандидаты для кэширования включают в себя данные, которые медленно меняются, и статические эталонные данные, которые часто считаваются.</span><span class="sxs-lookup"><span data-stu-id="fb130-169">Good initial candidates for caching include data that changes slowly, and static reference data that is read frequently.</span></span> 

2. <span data-ttu-id="fb130-170">Инструментируйте приложение и выполните мониторинг активной системы, чтобы узнать, как часто приложение получает данные или вычисляет сведения.</span><span class="sxs-lookup"><span data-stu-id="fb130-170">Instrument the application and monitor the live system to find out how frequently the application retrieves data or calculates information.</span></span>

3. <span data-ttu-id="fb130-171">Выполните профилирование приложения в тестовой среде, чтобы собрать низкоуровневые метрики о накладных расходах, связанных с операциями доступа к данным или другими часто выполняемыми вычислениями.</span><span class="sxs-lookup"><span data-stu-id="fb130-171">Profile the application in a test environment to capture low-level metrics about the overhead associated with data access operations or other frequently performed calculations.</span></span>

4. <span data-ttu-id="fb130-172">Выполните нагрузочное тестирование в тестовой среде, чтобы определить, как система реагирует при обычных и интенсивных рабочих нагрузках.</span><span class="sxs-lookup"><span data-stu-id="fb130-172">Perform load testing in a test environment to identify how the system responds under a normal workload and under heavy load.</span></span> <span data-ttu-id="fb130-173">Нагрузочное тестирование должно имитировать шаблон доступа к данным в рабочей среде, используя реалистичные рабочие нагрузки.</span><span class="sxs-lookup"><span data-stu-id="fb130-173">Load testing should simulate the pattern of data access observed in the production environment using realistic workloads.</span></span> 

5. <span data-ttu-id="fb130-174">Проанализируйте статистику доступа к данным для базовых хранилищ данных и проверьте, как часто повторяются те же запросы данных.</span><span class="sxs-lookup"><span data-stu-id="fb130-174">Examine the data access statistics for the underlying data stores and review how often the same data requests are repeated.</span></span> 


## <a name="example-diagnosis"></a><span data-ttu-id="fb130-175">Пример диагностики</span><span class="sxs-lookup"><span data-stu-id="fb130-175">Example diagnosis</span></span>

<span data-ttu-id="fb130-176">В следующих разделах эти шаги применяются к примеру приложения, описанному ранее.</span><span class="sxs-lookup"><span data-stu-id="fb130-176">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="instrument-the-application-and-monitor-the-live-system"></a><span data-ttu-id="fb130-177">Инструментирование приложения и мониторинг активной системы</span><span class="sxs-lookup"><span data-stu-id="fb130-177">Instrument the application and monitor the live system</span></span>

<span data-ttu-id="fb130-178">Инструментирование и мониторинг приложения позволяют получить сведения о конкретных запросах, выполняемых пользователями, пока приложение находится в рабочей среде.</span><span class="sxs-lookup"><span data-stu-id="fb130-178">Instrument the application and monitor it to get information about the specific requests that users make while the application is in production.</span></span> 

<span data-ttu-id="fb130-179">На следующем рисунке показан мониторинг данных, полученных с помощью [New Relic][NewRelic] во время нагрузочного теста.</span><span class="sxs-lookup"><span data-stu-id="fb130-179">The following image shows monitoring data captured by [New Relic][NewRelic] during a load test.</span></span> <span data-ttu-id="fb130-180">В этом случае единственной выполняемой операцией HTTP GET является `Person/GetAsync`.</span><span class="sxs-lookup"><span data-stu-id="fb130-180">In this case, the only HTTP GET operation performed is `Person/GetAsync`.</span></span> <span data-ttu-id="fb130-181">Но в активной рабочей среде на основе относительной частоты выполнения каждого запроса можно понять, какие ресурсы необходимо кэшировать.</span><span class="sxs-lookup"><span data-stu-id="fb130-181">But in a live production environment, knowing the relative frequency that each request is performed can give you insight into which resources should be cached.</span></span>

![Запросы сервера для приложения CachingDemo при использовании New Relic][NewRelic-server-requests]

<span data-ttu-id="fb130-183">Если требуется более глубокий анализ, можно использовать профилировщик для сбора данных о производительности нижнего уровня в тестовой среде (не в рабочей системе).</span><span class="sxs-lookup"><span data-stu-id="fb130-183">If you need a deeper analysis, you can use a profiler to capture low-level performance data in a test environment (not the production system).</span></span> <span data-ttu-id="fb130-184">Проверьте метрики (например, показатели частоты запросов ввода-вывода, использование памяти и ЦП).</span><span class="sxs-lookup"><span data-stu-id="fb130-184">Look at metrics such as I/O request rates, memory usage, and CPU utilization.</span></span> <span data-ttu-id="fb130-185">Эти метрики могут показать большое количество запросов в хранилище данных или службу, а также многократную обработку, которая выполняет один и тот же расчет.</span><span class="sxs-lookup"><span data-stu-id="fb130-185">These metrics may show a large number of requests to a data store or service, or repeated processing that performs the same calculation.</span></span> 

### <a name="load-test-the-application"></a><span data-ttu-id="fb130-186">Нагрузочное тестирование приложения</span><span class="sxs-lookup"><span data-stu-id="fb130-186">Load test the application</span></span>

<span data-ttu-id="fb130-187">На следующем графике показаны результаты нагрузочного тестирования примера приложения.</span><span class="sxs-lookup"><span data-stu-id="fb130-187">The following graph shows the results of load testing the sample application.</span></span> <span data-ttu-id="fb130-188">Нагрузочный тест имитирует пошаговую нагрузку до 800 пользователей, выполняющих типичную последовательность операций.</span><span class="sxs-lookup"><span data-stu-id="fb130-188">The load test simulates a step load of up to 800 users performing a typical series of operations.</span></span> 

![Результаты тестовой нагрузки производительности для сценария без кэширования][Performance-Load-Test-Results-Uncached]

<span data-ttu-id="fb130-190">Количество успешных тестов, выполняемых каждую секунду, достигает плато, в результате чего дополнительные запросы замедляются.</span><span class="sxs-lookup"><span data-stu-id="fb130-190">The number of successful tests performed each second reaches a plateau, and additional requests are slowed as a result.</span></span> <span data-ttu-id="fb130-191">Среднее время тестирования постоянно увеличивается вместе с рабочей нагрузкой.</span><span class="sxs-lookup"><span data-stu-id="fb130-191">The average test time steadily increases with the workload.</span></span> <span data-ttu-id="fb130-192">Время отклика увеличивается во время пиковой пользовательской нагрузки.</span><span class="sxs-lookup"><span data-stu-id="fb130-192">The response time levels off once the user load peaks.</span></span>

### <a name="examine-data-access-statistics"></a><span data-ttu-id="fb130-193">Обзор статистики получения доступа к данным</span><span class="sxs-lookup"><span data-stu-id="fb130-193">Examine data access statistics</span></span>

<span data-ttu-id="fb130-194">Статистика получения доступа к данным и другая информация, предоставляемая хранилищем данных, могут содержать полезные сведения (например, какие запросы наиболее часто повторяются).</span><span class="sxs-lookup"><span data-stu-id="fb130-194">Data access statistics and other information provided by a data store can give useful information, such as which queries are repeated most frequently.</span></span> <span data-ttu-id="fb130-195">Например, в Microsoft SQL Server административное представление `sys.dm_exec_query_stats` содержит статистические данные о недавно выполненных запросах.</span><span class="sxs-lookup"><span data-stu-id="fb130-195">For example, in Microsoft SQL Server, the `sys.dm_exec_query_stats` management view has statistical information for recently executed queries.</span></span> <span data-ttu-id="fb130-196">Текст каждого запроса доступен в представлении `sys.dm_exec-query_plan`.</span><span class="sxs-lookup"><span data-stu-id="fb130-196">The text for each query is available in the `sys.dm_exec-query_plan` view.</span></span> <span data-ttu-id="fb130-197">Чтобы запустить следующий SQL-запрос и определить, как часто выполняются запросы, можно использовать средство SQL Server Management Studio.</span><span class="sxs-lookup"><span data-stu-id="fb130-197">You can use a tool such as SQL Server Management Studio to run the following SQL query and determine how frequently queries are performed.</span></span>

```SQL
SELECT UseCounts, Text, Query_Plan
FROM sys.dm_exec_cached_plans
CROSS APPLY sys.dm_exec_sql_text(plan_handle)
CROSS APPLY sys.dm_exec_query_plan(plan_handle)
```

<span data-ttu-id="fb130-198">Столбец `UseCount` в результатах показывает частоту выполнения каждого запроса.</span><span class="sxs-lookup"><span data-stu-id="fb130-198">The `UseCount` column in the results indicates how frequently each query is run.</span></span> <span data-ttu-id="fb130-199">На следующем рисунке показано, что третий запрос выполнялся более 250 000 раз (гораздо больше, чем любой другой запрос).</span><span class="sxs-lookup"><span data-stu-id="fb130-199">The following image shows that the third query was run more than 250,000 times, significantly more than any other query.</span></span>

![Результаты запроса динамических административных представлений на сервере управления SQL Server][Dynamic-Management-Views]

<span data-ttu-id="fb130-201">Ниже показан SQL-запрос, который является причиной такого большого количества запросов к базе данных.</span><span class="sxs-lookup"><span data-stu-id="fb130-201">Here is the SQL query that is causing so many database requests:</span></span> 

```SQL
(@p__linq__0 int)SELECT TOP (2)
[Extent1].[BusinessEntityId] AS [BusinessEntityId],
[Extent1].[FirstName] AS [FirstName],
[Extent1].[LastName] AS [LastName]
FROM [Person].[Person] AS [Extent1]
WHERE [Extent1].[BusinessEntityId] = @p__linq__0
```

<span data-ttu-id="fb130-202">Это запрос, который генерирует платформа Entity Framework в методе `GetByIdAsync`, представленном выше.</span><span class="sxs-lookup"><span data-stu-id="fb130-202">This is the query that Entity Framework generates in `GetByIdAsync` method shown earlier.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="fb130-203">Реализация решения и проверка результатов</span><span class="sxs-lookup"><span data-stu-id="fb130-203">Implement the solution and verify the result</span></span>

<span data-ttu-id="fb130-204">После внедрения кэша повторите нагрузочные тесты и сравните эти результаты с результатами предыдущих тестов без кэша.</span><span class="sxs-lookup"><span data-stu-id="fb130-204">After you incorporate a cache, repeat the load tests and compare the results to the earlier load tests without a cache.</span></span> <span data-ttu-id="fb130-205">Ниже приведены результаты нагрузочного теста после добавления кэша в пример приложения.</span><span class="sxs-lookup"><span data-stu-id="fb130-205">Here are the load test results after adding a cache to the sample application.</span></span>

![Результаты нагрузочного теста производительности для сценария с кэшированием][Performance-Load-Test-Results-Cached]

<span data-ttu-id="fb130-207">Количество успешных тестов по-прежнему достигает плато, но при более высокой пользовательской нагрузке.</span><span class="sxs-lookup"><span data-stu-id="fb130-207">The volume of successful tests still reaches a plateau, but at a higher user load.</span></span> <span data-ttu-id="fb130-208">Частота запросов при этой нагрузке значительно больше, чем ранее.</span><span class="sxs-lookup"><span data-stu-id="fb130-208">The request rate at this load is significantly higher than earlier.</span></span> <span data-ttu-id="fb130-209">Среднее время выполнения теста по-прежнему возрастает по мере увеличения нагрузки, но максимальное время отклика составляет 0,05 мс (по сравнению с 1 мс, что в &mdash; 20&times; раз лучше).</span><span class="sxs-lookup"><span data-stu-id="fb130-209">Average test time still increases with load, but the maximum response time is 0.05 ms, compared with 1ms earlier &mdash; a 20&times; improvement.</span></span> 

## <a name="related-resources"></a><span data-ttu-id="fb130-210">Связанные ресурсы</span><span class="sxs-lookup"><span data-stu-id="fb130-210">Related resources</span></span>

- <span data-ttu-id="fb130-211">[Рекомендации по реализации API][api-implementation]</span><span class="sxs-lookup"><span data-stu-id="fb130-211">[API implementation best practices][api-implementation]</span></span>
- <span data-ttu-id="fb130-212">[Шаблон "Кэш на стороне"][cache-aside-pattern]</span><span class="sxs-lookup"><span data-stu-id="fb130-212">[Cache-Aside Pattern][cache-aside-pattern]</span></span>
- <span data-ttu-id="fb130-213">[Рекомендации по кэшированию][caching-guidance]</span><span class="sxs-lookup"><span data-stu-id="fb130-213">[Caching best practices][caching-guidance]</span></span>
- <span data-ttu-id="fb130-214">[Шаблон прерывателя][circuit-breaker]</span><span class="sxs-lookup"><span data-stu-id="fb130-214">[Circuit Breaker pattern][circuit-breaker]</span></span>

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/NoCaching
[cache-aside-pattern]: /azure/architecture/patterns/cache-aside
[caching-guidance]: ../../best-practices/caching.md
[circuit-breaker]: ../../patterns/circuit-breaker.md
[api-implementation]: ../../best-practices/api-implementation.md#optimizing-client-side-data-access
[NewRelic]: http://newrelic.com/azure
[NewRelic-server-requests]: _images/New-Relic.jpg
[Performance-Load-Test-Results-Uncached]:_images/InitialLoadTestResults.jpg
[Dynamic-Management-Views]: _images/SQLServerManagementStudio.jpg
[Performance-Load-Test-Results-Cached]: _images/CachedLoadTestResults.jpg