---
title: "Конкретные рекомендации по использованию механизма повторов"
description: "Конкретные рекомендации по настройке механизма повторов."
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 6bb623bd8be89573178f250570407bf83d62c098
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/23/2018
---
# <a name="retry-guidance-for-specific-services"></a><span data-ttu-id="76904-103">Руководство по использованию механизма повторов для отдельных служб</span><span class="sxs-lookup"><span data-stu-id="76904-103">Retry guidance for specific services</span></span>

<span data-ttu-id="76904-104">Большинство служб Azure и клиентских пакетов SDK содержат механизм повтора.</span><span class="sxs-lookup"><span data-stu-id="76904-104">Most Azure services and client SDKs include a retry mechanism.</span></span> <span data-ttu-id="76904-105">В то же время они отличаются, поскольку каждая служба имеет разные характеристики и требования, поэтому каждый механизм повтора настроен для конкретной службы.</span><span class="sxs-lookup"><span data-stu-id="76904-105">However, these differ because each service has different characteristics and requirements, and so each retry mechanism is tuned to a specific service.</span></span> <span data-ttu-id="76904-106">В этом руководстве представлены возможности механизма повтора для большинства служб Azure, а также сведения, помогающие использовать, адаптировать или расширять механизм повтора для такой службы.</span><span class="sxs-lookup"><span data-stu-id="76904-106">This guide summarizes the retry mechanism features for the majority of Azure services, and includes information to help you use, adapt, or extend the retry mechanism for that service.</span></span>

<span data-ttu-id="76904-107">Для получения общих рекомендаций по обработке временных сбоев и выполнения повторных попыток подключения и операций со службами и ресурсами обратитесь к разделу [Руководство по повторам](./transient-faults.md).</span><span class="sxs-lookup"><span data-stu-id="76904-107">For general guidance on handling transient faults, and retrying connections and operations against services and resources, see [Retry guidance](./transient-faults.md).</span></span>

<span data-ttu-id="76904-108">В следующей таблице перечислены функции механизма повтора для служб Azure, описанных в этом руководстве.</span><span class="sxs-lookup"><span data-stu-id="76904-108">The following table summarizes the retry features for the Azure services described in this guidance.</span></span>

| <span data-ttu-id="76904-109">**Служба**</span><span class="sxs-lookup"><span data-stu-id="76904-109">**Service**</span></span> | <span data-ttu-id="76904-110">**Ключевые возможности**</span><span class="sxs-lookup"><span data-stu-id="76904-110">**Retry capabilities**</span></span> | <span data-ttu-id="76904-111">**Настройки политики**</span><span class="sxs-lookup"><span data-stu-id="76904-111">**Policy configuration**</span></span> | <span data-ttu-id="76904-112">**Область**</span><span class="sxs-lookup"><span data-stu-id="76904-112">**Scope**</span></span> | <span data-ttu-id="76904-113">**Функции телеметрии**</span><span class="sxs-lookup"><span data-stu-id="76904-113">**Telemetry features**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="76904-114">**[Служба хранилища Azure](#azure-storage-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="76904-114">**[Azure Storage](#azure-storage-retry-guidelines)**</span></span> |<span data-ttu-id="76904-115">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="76904-115">Native in client</span></span> |<span data-ttu-id="76904-116">Программный</span><span class="sxs-lookup"><span data-stu-id="76904-116">Programmatic</span></span> |<span data-ttu-id="76904-117">Клиентские и индивидуальные операции</span><span class="sxs-lookup"><span data-stu-id="76904-117">Client and individual operations</span></span> |<span data-ttu-id="76904-118">TraceSource</span><span class="sxs-lookup"><span data-stu-id="76904-118">TraceSource</span></span> |
| <span data-ttu-id="76904-119">**[База данных SQL с Entity Framework](#sql-database-using-entity-framework-6-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="76904-119">**[SQL Database with Entity Framework](#sql-database-using-entity-framework-6-retry-guidelines)**</span></span> |<span data-ttu-id="76904-120">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="76904-120">Native in client</span></span> |<span data-ttu-id="76904-121">Программный</span><span class="sxs-lookup"><span data-stu-id="76904-121">Programmatic</span></span> |<span data-ttu-id="76904-122">Глобальные каждого домена приложения</span><span class="sxs-lookup"><span data-stu-id="76904-122">Global per AppDomain</span></span> |<span data-ttu-id="76904-123">Нет</span><span class="sxs-lookup"><span data-stu-id="76904-123">None</span></span> |
| <span data-ttu-id="76904-124">**[База данных SQL с Entity Framework Core](#sql-database-using-entity-framework-core-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="76904-124">**[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core-retry-guidelines)**</span></span> |<span data-ttu-id="76904-125">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="76904-125">Native in client</span></span> |<span data-ttu-id="76904-126">Программный</span><span class="sxs-lookup"><span data-stu-id="76904-126">Programmatic</span></span> |<span data-ttu-id="76904-127">Глобальные каждого домена приложения</span><span class="sxs-lookup"><span data-stu-id="76904-127">Global per AppDomain</span></span> |<span data-ttu-id="76904-128">Нет</span><span class="sxs-lookup"><span data-stu-id="76904-128">None</span></span> |
| <span data-ttu-id="76904-129">**[База данных SQL с ADO.NET](#sql-database-using-adonet-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="76904-129">**[SQL Database with ADO.NET](#sql-database-using-adonet-retry-guidelines)**</span></span> |[<span data-ttu-id="76904-130">Polly</span><span class="sxs-lookup"><span data-stu-id="76904-130">Polly</span></span>](#transient-fault-handling-with-polly) |<span data-ttu-id="76904-131">Декларативные и программные</span><span class="sxs-lookup"><span data-stu-id="76904-131">Declarative and programmatic</span></span> |<span data-ttu-id="76904-132">Единые инструкции или блоки кода</span><span class="sxs-lookup"><span data-stu-id="76904-132">Single statements or blocks of code</span></span> |<span data-ttu-id="76904-133">Пользовательская</span><span class="sxs-lookup"><span data-stu-id="76904-133">Custom</span></span> |
| <span data-ttu-id="76904-134">**[Служебная шина](#service-bus-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="76904-134">**[Service Bus](#service-bus-retry-guidelines)**</span></span> |<span data-ttu-id="76904-135">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="76904-135">Native in client</span></span> |<span data-ttu-id="76904-136">Программный</span><span class="sxs-lookup"><span data-stu-id="76904-136">Programmatic</span></span> |<span data-ttu-id="76904-137">Диспетчер пространств имен, фабрика сообщений и клиент</span><span class="sxs-lookup"><span data-stu-id="76904-137">Namespace Manager, Messaging Factory, and Client</span></span> |<span data-ttu-id="76904-138">Трассировка событий Windows</span><span class="sxs-lookup"><span data-stu-id="76904-138">ETW</span></span> |
| <span data-ttu-id="76904-139">**[Кэш Redis для Azure](#azure-redis-cache-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="76904-139">**[Azure Redis Cache](#azure-redis-cache-retry-guidelines)**</span></span> |<span data-ttu-id="76904-140">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="76904-140">Native in client</span></span> |<span data-ttu-id="76904-141">Программный</span><span class="sxs-lookup"><span data-stu-id="76904-141">Programmatic</span></span> |<span data-ttu-id="76904-142">Клиент</span><span class="sxs-lookup"><span data-stu-id="76904-142">Client</span></span> |<span data-ttu-id="76904-143">TextWriter</span><span class="sxs-lookup"><span data-stu-id="76904-143">TextWriter</span></span> |
| <span data-ttu-id="76904-144">**[Cosmos DB](#cosmos-db-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="76904-144">**[Cosmos DB](#cosmos-db-retry-guidelines)**</span></span> |<span data-ttu-id="76904-145">Машинный код в службе</span><span class="sxs-lookup"><span data-stu-id="76904-145">Native in service</span></span> |<span data-ttu-id="76904-146">Ненастраиваемые</span><span class="sxs-lookup"><span data-stu-id="76904-146">Non-configurable</span></span> |<span data-ttu-id="76904-147">Глобальные</span><span class="sxs-lookup"><span data-stu-id="76904-147">Global</span></span> |<span data-ttu-id="76904-148">TraceSource</span><span class="sxs-lookup"><span data-stu-id="76904-148">TraceSource</span></span> |
| <span data-ttu-id="76904-149">**[Поиск Azure](#azure-storage-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="76904-149">**[Azure Search](#azure-storage-retry-guidelines)**</span></span> |<span data-ttu-id="76904-150">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="76904-150">Native in client</span></span> |<span data-ttu-id="76904-151">Программный</span><span class="sxs-lookup"><span data-stu-id="76904-151">Programmatic</span></span> |<span data-ttu-id="76904-152">Клиент</span><span class="sxs-lookup"><span data-stu-id="76904-152">Client</span></span> |<span data-ttu-id="76904-153">Трассировка событий Windows или пользовательская</span><span class="sxs-lookup"><span data-stu-id="76904-153">ETW or Custom</span></span> |
| <span data-ttu-id="76904-154">**[Azure Active Directory](#azure-active-directory-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="76904-154">**[Azure Active Directory](#azure-active-directory-retry-guidelines)**</span></span> |<span data-ttu-id="76904-155">Машинный код в библиотеке ADAL</span><span class="sxs-lookup"><span data-stu-id="76904-155">Native in ADAL library</span></span> |<span data-ttu-id="76904-156">Встроена в библиотеку ADAL</span><span class="sxs-lookup"><span data-stu-id="76904-156">Embeded into ADAL library</span></span> |<span data-ttu-id="76904-157">Внутренний</span><span class="sxs-lookup"><span data-stu-id="76904-157">Internal</span></span> |<span data-ttu-id="76904-158">None</span><span class="sxs-lookup"><span data-stu-id="76904-158">None</span></span> |
| <span data-ttu-id="76904-159">**[Service Fabric](#service-fabric-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="76904-159">**[Service Fabric](#service-fabric-retry-guidelines)**</span></span> |<span data-ttu-id="76904-160">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="76904-160">Native in client</span></span> |<span data-ttu-id="76904-161">Программный</span><span class="sxs-lookup"><span data-stu-id="76904-161">Programmatic</span></span> |<span data-ttu-id="76904-162">Клиент</span><span class="sxs-lookup"><span data-stu-id="76904-162">Client</span></span> |<span data-ttu-id="76904-163">None</span><span class="sxs-lookup"><span data-stu-id="76904-163">None</span></span> | 
| <span data-ttu-id="76904-164">**[Концентраторы событий Azure](#azure-event-hubs-retry-guidelines)**</span><span class="sxs-lookup"><span data-stu-id="76904-164">**[Azure Event Hubs](#azure-event-hubs-retry-guidelines)**</span></span> |<span data-ttu-id="76904-165">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="76904-165">Native in client</span></span> |<span data-ttu-id="76904-166">Программный</span><span class="sxs-lookup"><span data-stu-id="76904-166">Programmatic</span></span> |<span data-ttu-id="76904-167">Клиент</span><span class="sxs-lookup"><span data-stu-id="76904-167">Client</span></span> |<span data-ttu-id="76904-168">None</span><span class="sxs-lookup"><span data-stu-id="76904-168">None</span></span> |

> [!NOTE]
> <span data-ttu-id="76904-169">Для большинства встроенных механизмов повтора Azure в данный момент невозможно применить различную политику повтора для различных типов ошибок или исключений за пределами функционала, включенного в политику повтора.</span><span class="sxs-lookup"><span data-stu-id="76904-169">For most of the Azure built-in retry mechanisms, there is currently no way apply a different retry policy for different types of error or exception beyond the functionality include in the retry policy.</span></span> <span data-ttu-id="76904-170">Таким образом наилучшей рекомендацией на момент написания статьи будет являться настройка политики, которая обеспечит оптимальный средний уровень производительности и доступности.</span><span class="sxs-lookup"><span data-stu-id="76904-170">Therefore, the best guidance available at the time of writing is to configure a policy that provides the optimum average performance and availability.</span></span> <span data-ttu-id="76904-171">Одним из способов настройки политики является анализ файлов журнала для определения типа возникающих временных сбоев.</span><span class="sxs-lookup"><span data-stu-id="76904-171">One way to fine-tune the policy is to analyze log files to determine the type of transient faults that are occurring.</span></span> <span data-ttu-id="76904-172">Например, если большая часть ошибок связана с проблемами с сетевым подключением, вы можете попробовать настроить немедленный повтор вместо ожидания перед первой попыткой.</span><span class="sxs-lookup"><span data-stu-id="76904-172">For example, if the majority of errors are related to network connectivity issues, you might attempt an immediate retry rather than wait a long time for the first retry.</span></span>
>
>

## <a name="azure-storage-retry-guidelines"></a><span data-ttu-id="76904-173">Рекомендации по использованию механизма повторов для службы хранилища Azure</span><span class="sxs-lookup"><span data-stu-id="76904-173">Azure Storage retry guidelines</span></span>
<span data-ttu-id="76904-174">Службы хранилища Azure включают таблицы и BLOB-хранилища, файлы и очереди хранилища.</span><span class="sxs-lookup"><span data-stu-id="76904-174">Azure storage services include table and blob storage, files, and storage queues.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="76904-175">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="76904-175">Retry mechanism</span></span>
<span data-ttu-id="76904-176">Повторы выполняются на уровне отдельных операции REST и являются неотъемлемой частью реализации API клиента.</span><span class="sxs-lookup"><span data-stu-id="76904-176">Retries occur at the individual REST operation level and are an integral part of the client API implementation.</span></span> <span data-ttu-id="76904-177">Клиентский пакет SDK службы хранилища использует классы, реализующие [Интерфейс IExtendedRetryPolicy](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).</span><span class="sxs-lookup"><span data-stu-id="76904-177">The client storage SDK uses classes that implement the [IExtendedRetryPolicy Interface](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).</span></span>

<span data-ttu-id="76904-178">Существуют различные реализации этого интерфейса.</span><span class="sxs-lookup"><span data-stu-id="76904-178">There are different implementations of the interface.</span></span> <span data-ttu-id="76904-179">Клиенты службы хранилища могут выбирать политики специально для доступа к таблицам, BLOB-объектам и очередям.</span><span class="sxs-lookup"><span data-stu-id="76904-179">Storage clients can choose from policies specifically designed for accessing tables, blobs, and queues.</span></span> <span data-ttu-id="76904-180">Каждая реализация использует различные стратегии повтора, фактически определяя интервал повторных попыток и другие параметры.</span><span class="sxs-lookup"><span data-stu-id="76904-180">Each implementation uses a different retry strategy that essentially defines the retry interval and other details.</span></span>

<span data-ttu-id="76904-181">Встроенные классы обеспечивают поддержку для линейных (постоянная задержка) и экспоненциальных со случайными параметрами интервалов повтора.</span><span class="sxs-lookup"><span data-stu-id="76904-181">The built-in classes provide support for linear (constant delay) and exponential with randomization retry intervals.</span></span> <span data-ttu-id="76904-182">Здесь также нет политики повторов для случаев, когда другой процесс обрабатывает повторы на более высоком уровне.</span><span class="sxs-lookup"><span data-stu-id="76904-182">There is also a no retry policy for use when another process is handling retries at a higher level.</span></span> <span data-ttu-id="76904-183">Тем не менее если имеются особые требования, не предоставляемые встроенными классами, то можно реализовать собственные классы для обработки повторов.</span><span class="sxs-lookup"><span data-stu-id="76904-183">However, you can implement your own retry classes if you have specific requirements not provided by the built-in classes.</span></span>

<span data-ttu-id="76904-184">Альтернативные повторы переключаются между первичным и вторичным расположением службы хранилища при использовании геоизбыточного хранилища с доступом на чтение (RA-GRS) и если результатом запроса стала ошибка.</span><span class="sxs-lookup"><span data-stu-id="76904-184">Alternate retries switch between primary and secondary storage service location if you are using read access geo-redundant storage (RA-GRS) and the result of the request is a retryable error.</span></span> <span data-ttu-id="76904-185">Дополнительные сведения см. в статье [Варианты избыточности хранилища Azure](http://msdn.microsoft.com/library/azure/dn727290.aspx).</span><span class="sxs-lookup"><span data-stu-id="76904-185">See [Azure Storage Redundancy Options](http://msdn.microsoft.com/library/azure/dn727290.aspx) for more information.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="76904-186">Настройки политики</span><span class="sxs-lookup"><span data-stu-id="76904-186">Policy configuration</span></span>
<span data-ttu-id="76904-187">Политики повтора настраиваются программным способом.</span><span class="sxs-lookup"><span data-stu-id="76904-187">Retry policies are configured programmatically.</span></span> <span data-ttu-id="76904-188">Типичная процедура состоит в создании и заполнении экземпляров **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions** или **QueueRequestOptions**.</span><span class="sxs-lookup"><span data-stu-id="76904-188">A typical procedure is to create and populate a **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, or **QueueRequestOptions** instance.</span></span>

```csharp
TableRequestOptions interactiveRequestOption = new TableRequestOptions()
{
  RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
  // For Read-access geo-redundant storage, use PrimaryThenSecondary.
  // Otherwise set this to PrimaryOnly.
  LocationMode = LocationMode.PrimaryThenSecondary,
  // Maximum execution time based on the business use case. 
  MaximumExecutionTime = TimeSpan.FromSeconds(2)
};
```

<span data-ttu-id="76904-189">Затем можно задать параметры экземпляра запроса на клиенте, и все операции с клиентом будут использовать указанные параметры запроса.</span><span class="sxs-lookup"><span data-stu-id="76904-189">The request options instance can then be set on the client, and all operations with the client will use the specified request options.</span></span>

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

<span data-ttu-id="76904-190">Можно переопределить параметры запроса клиента путем передачи заполненного экземпляра класса параметров запроса в качестве параметра метода.</span><span class="sxs-lookup"><span data-stu-id="76904-190">You can override the client request options by passing a populated instance of the request options class as a parameter to operation methods.</span></span>

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

<span data-ttu-id="76904-191">Можно использовать экземпляр **OperationContext** , чтобы указать код, выполняемый при возникновении повтора и после завершения операции.</span><span class="sxs-lookup"><span data-stu-id="76904-191">You use an **OperationContext** instance to specify the code to execute when a retry occurs and when an operation has completed.</span></span> <span data-ttu-id="76904-192">Этот код может собирать сведения об операции для последующего использования в результатах телеметрии и журналах.</span><span class="sxs-lookup"><span data-stu-id="76904-192">This code can collect information about the operation for use in logs and telemetry.</span></span>

    // Set up notifications for an operation
    var context = new OperationContext();
    context.ClientRequestID = "some request id";
    context.Retrying += (sender, args) =>
    {
      /* Collect retry information */
    };
    context.RequestCompleted += (sender, args) =>
    {
      /* Collect operation completion information */
    };
    var stats = await client.GetServiceStatsAsync(null, context);

<span data-ttu-id="76904-193">Кроме определения, подходит ли сбой для повторных попыток, расширенные политики повтора возвращают объект **RetryContext**, который указывает число повторных попыток, результаты последнего запроса, произойдет ли следующая попытка в первичном или вторичном расположении (дополнительные сведения см. в таблице ниже).</span><span class="sxs-lookup"><span data-stu-id="76904-193">In addition to indicating whether a failure is suitable for retry, the extended retry policies return a **RetryContext** object that indicates the number of retries, the results of the last request, whether the next retry will happen in the primary or secondary location (see table below for details).</span></span> <span data-ttu-id="76904-194">Свойства объекта **RetryContext** позволяют принять решение о необходимости и времени выполнения повтора.</span><span class="sxs-lookup"><span data-stu-id="76904-194">The properties of the **RetryContext** object can be used to decide if and when to attempt a retry.</span></span> <span data-ttu-id="76904-195">Дополнительные сведения см. в статье [Метод IExtendedRetryPolicy.Evaluate](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).</span><span class="sxs-lookup"><span data-stu-id="76904-195">For more details, see [IExtendedRetryPolicy.Evaluate Method](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).</span></span>

<span data-ttu-id="76904-196">Следующие таблицы содержат значения по умолчанию для встроенной политики повторов.</span><span class="sxs-lookup"><span data-stu-id="76904-196">The following tables show the default settings for the built-in retry policies.</span></span>

<span data-ttu-id="76904-197">**Параметры запроса**</span><span class="sxs-lookup"><span data-stu-id="76904-197">**Request options**</span></span>

| <span data-ttu-id="76904-198">**Параметр**</span><span class="sxs-lookup"><span data-stu-id="76904-198">**Setting**</span></span> | <span data-ttu-id="76904-199">**Значение по умолчанию**</span><span class="sxs-lookup"><span data-stu-id="76904-199">**Default value**</span></span> | <span data-ttu-id="76904-200">**Значение**</span><span class="sxs-lookup"><span data-stu-id="76904-200">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="76904-201">MaximumExecutionTime</span><span class="sxs-lookup"><span data-stu-id="76904-201">MaximumExecutionTime</span></span> | <span data-ttu-id="76904-202">120 секунд</span><span class="sxs-lookup"><span data-stu-id="76904-202">120 seconds</span></span> | <span data-ttu-id="76904-203">Максимальное время выполнения запроса, включая все потенциальные повторные попытки.</span><span class="sxs-lookup"><span data-stu-id="76904-203">Maximum execution time for the request, including all potential retry attempts.</span></span> |
| <span data-ttu-id="76904-204">ServerTimeOut</span><span class="sxs-lookup"><span data-stu-id="76904-204">ServerTimeout</span></span> | <span data-ttu-id="76904-205">None</span><span class="sxs-lookup"><span data-stu-id="76904-205">None</span></span> | <span data-ttu-id="76904-206">Интервал времени ожидания сервера для запроса (значение округляется до секунд).</span><span class="sxs-lookup"><span data-stu-id="76904-206">Server timeout interval for the request (value is rounded to seconds).</span></span> <span data-ttu-id="76904-207">Если не указан, он будет использовать значение по умолчанию для всех запросов к серверу.</span><span class="sxs-lookup"><span data-stu-id="76904-207">If not specified, it will use the default value for all requests to the server.</span></span> <span data-ttu-id="76904-208">Как правило, лучше всего опустить этот параметр, чтобы использовать значение сервера по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="76904-208">Usually, the best option is to omit this setting so that the server default is used.</span></span> | 
| <span data-ttu-id="76904-209">LocationMode</span><span class="sxs-lookup"><span data-stu-id="76904-209">LocationMode</span></span> | <span data-ttu-id="76904-210">Нет</span><span class="sxs-lookup"><span data-stu-id="76904-210">None</span></span> | <span data-ttu-id="76904-211">Если учетная запись хранения создается с вариантом репликации на географически избыточном хранилище с доступом для чтения (RA-GRS), то можно использовать режим расположения, чтобы указать расположение, где запрос должен быть получен.</span><span class="sxs-lookup"><span data-stu-id="76904-211">If the storage account is created with the Read access geo-redundant storage (RA-GRS) replication option, you can use the location mode to indicate which location should receive the request.</span></span> <span data-ttu-id="76904-212">Например, если указано **PrimaryThenSecondary**, запросы всегда будут отправляться сначала в основное расположение.</span><span class="sxs-lookup"><span data-stu-id="76904-212">For example, if **PrimaryThenSecondary** is specified, requests are always sent to the primary location first.</span></span> <span data-ttu-id="76904-213">Если запрос завершается ошибкой, он направляется во вторичное расположение.</span><span class="sxs-lookup"><span data-stu-id="76904-213">If a request fails, it is sent to the secondary location.</span></span> |
| <span data-ttu-id="76904-214">политика RetryPolicy</span><span class="sxs-lookup"><span data-stu-id="76904-214">RetryPolicy</span></span> | <span data-ttu-id="76904-215">ExponentialPolicy</span><span class="sxs-lookup"><span data-stu-id="76904-215">ExponentialPolicy</span></span> | <span data-ttu-id="76904-216">Сведения о каждом параметре смотрите далее.</span><span class="sxs-lookup"><span data-stu-id="76904-216">See below for details of each option.</span></span> |

<span data-ttu-id="76904-217">**Экспоненциальная политика**</span><span class="sxs-lookup"><span data-stu-id="76904-217">**Exponential policy**</span></span> 

| <span data-ttu-id="76904-218">**Параметр**</span><span class="sxs-lookup"><span data-stu-id="76904-218">**Setting**</span></span> | <span data-ttu-id="76904-219">**Значение по умолчанию**</span><span class="sxs-lookup"><span data-stu-id="76904-219">**Default value**</span></span> | <span data-ttu-id="76904-220">**Значение**</span><span class="sxs-lookup"><span data-stu-id="76904-220">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="76904-221">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="76904-221">maxAttempt</span></span> | <span data-ttu-id="76904-222">3</span><span class="sxs-lookup"><span data-stu-id="76904-222">3</span></span> | <span data-ttu-id="76904-223">Количество повторных попыток.</span><span class="sxs-lookup"><span data-stu-id="76904-223">Number of retry attempts.</span></span> |
| <span data-ttu-id="76904-224">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="76904-224">deltaBackoff</span></span> | <span data-ttu-id="76904-225">4 секунды</span><span class="sxs-lookup"><span data-stu-id="76904-225">4 seconds</span></span> | <span data-ttu-id="76904-226">Интервал отсрочки между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="76904-226">Back-off interval between retries.</span></span> <span data-ttu-id="76904-227">Кратное timespan, включая элемент случая, который будет использоваться для последующих попыток.</span><span class="sxs-lookup"><span data-stu-id="76904-227">Multiples of this timespan, including a random element, will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="76904-228">MinBackoff</span><span class="sxs-lookup"><span data-stu-id="76904-228">MinBackoff</span></span> | <span data-ttu-id="76904-229">3 секунды</span><span class="sxs-lookup"><span data-stu-id="76904-229">3 seconds</span></span> | <span data-ttu-id="76904-230">Добавить ко всем интервалам повтора, вычисленным по deltaBackoff.</span><span class="sxs-lookup"><span data-stu-id="76904-230">Added to all retry intervals computed from deltaBackoff.</span></span> <span data-ttu-id="76904-231">Это значение не может быть изменено.</span><span class="sxs-lookup"><span data-stu-id="76904-231">This value cannot be changed.</span></span>
| <span data-ttu-id="76904-232">MaxBackoff</span><span class="sxs-lookup"><span data-stu-id="76904-232">MaxBackoff</span></span> | <span data-ttu-id="76904-233">120 секунд</span><span class="sxs-lookup"><span data-stu-id="76904-233">120 seconds</span></span> | <span data-ttu-id="76904-234">MaxBackoff используется в том случае, если расчетный интервал повторных попыток оказывается больше, чем MaxBackoff.</span><span class="sxs-lookup"><span data-stu-id="76904-234">MaxBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> <span data-ttu-id="76904-235">Это значение не может быть изменено.</span><span class="sxs-lookup"><span data-stu-id="76904-235">This value cannot be changed.</span></span> |

<span data-ttu-id="76904-236">**Линейная политика**</span><span class="sxs-lookup"><span data-stu-id="76904-236">**Linear policy**</span></span>

| <span data-ttu-id="76904-237">**Параметр**</span><span class="sxs-lookup"><span data-stu-id="76904-237">**Setting**</span></span> | <span data-ttu-id="76904-238">**Значение по умолчанию**</span><span class="sxs-lookup"><span data-stu-id="76904-238">**Default value**</span></span> | <span data-ttu-id="76904-239">**Значение**</span><span class="sxs-lookup"><span data-stu-id="76904-239">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="76904-240">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="76904-240">maxAttempt</span></span> | <span data-ttu-id="76904-241">3</span><span class="sxs-lookup"><span data-stu-id="76904-241">3</span></span> | <span data-ttu-id="76904-242">Количество повторных попыток.</span><span class="sxs-lookup"><span data-stu-id="76904-242">Number of retry attempts.</span></span> |
| <span data-ttu-id="76904-243">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="76904-243">deltaBackoff</span></span> | <span data-ttu-id="76904-244">30 секунд</span><span class="sxs-lookup"><span data-stu-id="76904-244">30 seconds</span></span> | <span data-ttu-id="76904-245">Интервал отсрочки между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="76904-245">Back-off interval between retries.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="76904-246">Руководство по использованию механизма повторов</span><span class="sxs-lookup"><span data-stu-id="76904-246">Retry usage guidance</span></span>
<span data-ttu-id="76904-247">При доступе к службам хранилища Azure с помощью API клиентской службы хранилища, придерживайтесь следующих рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="76904-247">Consider the following guidelines when accessing Azure storage services using the storage client API:</span></span>

* <span data-ttu-id="76904-248">Используйте встроенные политики повтора из пространства имен Microsoft.WindowsAzure.Storage.RetryPolicies, которые соответствуют требованиям.</span><span class="sxs-lookup"><span data-stu-id="76904-248">Use the built-in retry policies from the Microsoft.WindowsAzure.Storage.RetryPolicies namespace where they are appropriate for your requirements.</span></span> <span data-ttu-id="76904-249">В большинстве случаев использования этих политик будет достаточно.</span><span class="sxs-lookup"><span data-stu-id="76904-249">In most cases, these policies will be sufficient.</span></span>
* <span data-ttu-id="76904-250">Используйте политику **ExponentialRetry** для пакетных операций, фоновых задач или неинтерактивных сценариев.</span><span class="sxs-lookup"><span data-stu-id="76904-250">Use the **ExponentialRetry** policy in batch operations, background tasks, or non-interactive scenarios.</span></span> <span data-ttu-id="76904-251">В этих сценариях обычно имеется больше времени для восстановления службы — таким образом повышается успешного выполнения операции.</span><span class="sxs-lookup"><span data-stu-id="76904-251">In these scenarios, you can typically allow more time for the service to recover—with a consequently increased chance of the operation eventually succeeding.</span></span>
* <span data-ttu-id="76904-252">Следует указать свойства **MaximumExecutionTime** параметра **RequestOptions** для ограничения общего времени выполнения, а также принять во внимание тип и размер операции при выборе значения времени ожидания.</span><span class="sxs-lookup"><span data-stu-id="76904-252">Consider specifying the **MaximumExecutionTime** property of the **RequestOptions** parameter to limit the total execution time, but take into account the type and size of the operation when choosing a timeout value.</span></span>
* <span data-ttu-id="76904-253">Если необходимо реализовать пользовательский повтор, избегайте создания оболочек клиентских классов хранения.</span><span class="sxs-lookup"><span data-stu-id="76904-253">If you need to implement a custom retry, avoid creating wrappers around the storage client classes.</span></span> <span data-ttu-id="76904-254">Вместо этого используйте возможности для расширения существующих политик с помощью интерфейса **IExtendedRetryPolicy** .</span><span class="sxs-lookup"><span data-stu-id="76904-254">Instead, use the capabilities to extend the existing policies through the **IExtendedRetryPolicy** interface.</span></span>
* <span data-ttu-id="76904-255">Если вы используете геоизбыточное хранилище с доступом на чтение (RA-GRS), можно использовать **LocationMode** для указания, что повторные попытки должны будут выполняться для вторичной копии хранилища в случае сбоя доступа к первичному хранилищу.</span><span class="sxs-lookup"><span data-stu-id="76904-255">If you are using read access geo-redundant storage (RA-GRS) you can use the **LocationMode** to specify that retry attempts will access the secondary read-only copy of the store should the primary access fail.</span></span> <span data-ttu-id="76904-256">Однако при использовании этого параметра необходимо убедиться, что ваше приложение может успешно работать с данными, которые могут устаревать, если процедура репликации с основным хранилищем еще не завершена.</span><span class="sxs-lookup"><span data-stu-id="76904-256">However, when using this option you must ensure that your application can work successfully with data that may be stale if the replication from the primary store has not yet completed.</span></span>

<span data-ttu-id="76904-257">Начните с использования следующих параметров для операции повтора.</span><span class="sxs-lookup"><span data-stu-id="76904-257">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="76904-258">Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.</span><span class="sxs-lookup"><span data-stu-id="76904-258">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>  

| <span data-ttu-id="76904-259">**Контекст**</span><span class="sxs-lookup"><span data-stu-id="76904-259">**Context**</span></span> | <span data-ttu-id="76904-260">**Максимальная задержка<br />примера целевого E2E**</span><span class="sxs-lookup"><span data-stu-id="76904-260">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="76904-261">**Политика повтора**</span><span class="sxs-lookup"><span data-stu-id="76904-261">**Retry policy**</span></span> | <span data-ttu-id="76904-262">**Параметры**</span><span class="sxs-lookup"><span data-stu-id="76904-262">**Settings**</span></span> | <span data-ttu-id="76904-263">**Значения**</span><span class="sxs-lookup"><span data-stu-id="76904-263">**Values**</span></span> | <span data-ttu-id="76904-264">**Принцип работы**</span><span class="sxs-lookup"><span data-stu-id="76904-264">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="76904-265">Интерактивный, пользовательский интерфейс</span><span class="sxs-lookup"><span data-stu-id="76904-265">Interactive, UI,</span></span><br /><span data-ttu-id="76904-266">или передний план</span><span class="sxs-lookup"><span data-stu-id="76904-266">or foreground</span></span> |<span data-ttu-id="76904-267">2 секунды</span><span class="sxs-lookup"><span data-stu-id="76904-267">2 seconds</span></span> |<span data-ttu-id="76904-268">Линейная</span><span class="sxs-lookup"><span data-stu-id="76904-268">Linear</span></span> |<span data-ttu-id="76904-269">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="76904-269">maxAttempt</span></span><br /><span data-ttu-id="76904-270">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="76904-270">deltaBackoff</span></span> |<span data-ttu-id="76904-271">3</span><span class="sxs-lookup"><span data-stu-id="76904-271">3</span></span><br /><span data-ttu-id="76904-272">500 мс</span><span class="sxs-lookup"><span data-stu-id="76904-272">500 ms</span></span> |<span data-ttu-id="76904-273">Попытка 1 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="76904-273">Attempt 1 - delay 500 ms</span></span><br /><span data-ttu-id="76904-274">Попытка 2 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="76904-274">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="76904-275">попытка 3 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="76904-275">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="76904-276">Фоновый</span><span class="sxs-lookup"><span data-stu-id="76904-276">Background</span></span><br /><span data-ttu-id="76904-277">или пакетный</span><span class="sxs-lookup"><span data-stu-id="76904-277">or batch</span></span> |<span data-ttu-id="76904-278">30 секунд</span><span class="sxs-lookup"><span data-stu-id="76904-278">30 seconds</span></span> |<span data-ttu-id="76904-279">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="76904-279">Exponential</span></span> |<span data-ttu-id="76904-280">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="76904-280">maxAttempt</span></span><br /><span data-ttu-id="76904-281">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="76904-281">deltaBackoff</span></span> |<span data-ttu-id="76904-282">5</span><span class="sxs-lookup"><span data-stu-id="76904-282">5</span></span><br /><span data-ttu-id="76904-283">4 секунды</span><span class="sxs-lookup"><span data-stu-id="76904-283">4 seconds</span></span> |<span data-ttu-id="76904-284">Попытка 1 — задержка ~ 3 с</span><span class="sxs-lookup"><span data-stu-id="76904-284">Attempt 1 - delay ~3 sec</span></span><br /><span data-ttu-id="76904-285">Попытка 2 — задержка 7 с</span><span class="sxs-lookup"><span data-stu-id="76904-285">Attempt 2 - delay ~7 sec</span></span><br /><span data-ttu-id="76904-286">Попытка 3 — задержка ~ 15 с</span><span class="sxs-lookup"><span data-stu-id="76904-286">Attempt 3 - delay ~15 sec</span></span> |

### <a name="telemetry"></a><span data-ttu-id="76904-287">Телеметрия</span><span class="sxs-lookup"><span data-stu-id="76904-287">Telemetry</span></span>
<span data-ttu-id="76904-288">Количество попыток входа регистрируется в **TraceSource**.</span><span class="sxs-lookup"><span data-stu-id="76904-288">Retry attempts are logged to a **TraceSource**.</span></span> <span data-ttu-id="76904-289">Необходимо настроить **TraceListener** для сбора данных о событиях и записи этих данных в журнал с подходящим назначением.</span><span class="sxs-lookup"><span data-stu-id="76904-289">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span> <span data-ttu-id="76904-290">Можно использовать **TextWriterTraceListener** или **XmlWriterTraceListener** для записи данных в файл журнала, **EventLogTraceListener** для записи в журнал событий Windows или **EventProviderTraceListener** для записи данных трассировки в подсистеме трассировки событий Windows.</span><span class="sxs-lookup"><span data-stu-id="76904-290">You can use the **TextWriterTraceListener** or **XmlWriterTraceListener** to write the data to a log file, the **EventLogTraceListener** to write to the Windows Event Log, or the **EventProviderTraceListener** to write trace data to the ETW subsystem.</span></span> <span data-ttu-id="76904-291">Можно также настроить автоматическую очистку буфера и уровень детализации регистрируемых событий (например, Ошибка, Предупреждение, Информационное событие и Подробное протоколирование).</span><span class="sxs-lookup"><span data-stu-id="76904-291">You can also configure auto-flushing of the buffer, and the verbosity of events that will be logged (for example, Error, Warning, Informational, and Verbose).</span></span> <span data-ttu-id="76904-292">Дополнительные сведения см. в статье [Вход в клиентскую библиотеку хранилища .NET на стороне клиента](http://msdn.microsoft.com/library/azure/dn782839.aspx).</span><span class="sxs-lookup"><span data-stu-id="76904-292">For more information, see [Client-side Logging with the .NET Storage Client Library](http://msdn.microsoft.com/library/azure/dn782839.aspx).</span></span>

<span data-ttu-id="76904-293">Операции могут получать экземпляр **OperationContext**, предоставляющий событие **Повтор**, которое может использоваться для присоединения телеметрии пользовательской логики.</span><span class="sxs-lookup"><span data-stu-id="76904-293">Operations can receive an **OperationContext** instance, which exposes a **Retrying** event that can be used to attach custom telemetry logic.</span></span> <span data-ttu-id="76904-294">Дополнительные сведения см. в статье [Событие OperationContext.Retrying](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span><span class="sxs-lookup"><span data-stu-id="76904-294">For more information, see [OperationContext.Retrying Event](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span></span>

### <a name="examples"></a><span data-ttu-id="76904-295">Примеры</span><span class="sxs-lookup"><span data-stu-id="76904-295">Examples</span></span>
<span data-ttu-id="76904-296">В следующем примере кода показано, как создать два экземпляра **TableRequestOptions** с различными параметрами, один для интерактивных запросов и один для фоновых запросов.</span><span class="sxs-lookup"><span data-stu-id="76904-296">The following code example shows how to create two **TableRequestOptions** instances with different retry settings; one for interactive requests and one for background requests.</span></span> <span data-ttu-id="76904-297">Пример затем устанавливает эти две политики повторов на клиентском компьютере, чтобы они применялись для всех запросов, а также задает интерактивную стратегию на конкретный запрос, чтобы он переопределял параметры по умолчанию, применяемые к клиенту.</span><span class="sxs-lookup"><span data-stu-id="76904-297">The example then sets these two retry policies on the client so that they apply for all requests, and also sets the interactive strategy on a specific request so that it overrides the default settings applied to the client.</span></span>

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.RetryPolicies;
using Microsoft.WindowsAzure.Storage.Table;

namespace RetryCodeSamples
{
    class AzureStorageCodeSamples
    {
        private const string connectionString = "UseDevelopmentStorage=true";

        public async static Task Samples()
        {
            var storageAccount = CloudStorageAccount.Parse(connectionString);

            TableRequestOptions interactiveRequestOption = new TableRequestOptions()
            {
                RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
                // For Read-access geo-redundant storage, use PrimaryThenSecondary.
                // Otherwise set this to PrimaryOnly.
                LocationMode = LocationMode.PrimaryThenSecondary,
                // Maximum execution time based on the business use case. 
                MaximumExecutionTime = TimeSpan.FromSeconds(2)
            };

            TableRequestOptions backgroundRequestOption = new TableRequestOptions()
            {
                // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
                // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
                MaximumExecutionTime = TimeSpan.FromSeconds(30),
                // PrimaryThenSecondary in case of Read-access geo-redundant storage, else set this to PrimaryOnly
                LocationMode = LocationMode.PrimaryThenSecondary
            };

            var client = storageAccount.CreateCloudTableClient();
            // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
            // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
            // ServerTimeout and MaximumExecutionTime are not set

            {
                // Set properties for the client (used on all requests unless overridden)
                // Different exponential policy parameters for background scenarios
                client.DefaultRequestOptions = backgroundRequestOption;
                // Linear policy for interactive scenarios
                client.DefaultRequestOptions = interactiveRequestOption;
            }

            {
                // set properties for a specific request
                var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
            }

            {
                // Set up notifications for an operation
                var context = new OperationContext();
                context.ClientRequestID = "some request id";
                context.Retrying += (sender, args) =>
                {
                    /* Collect retry information */
                };
                context.RequestCompleted += (sender, args) =>
                {
                    /* Collect operation completion information */
                };
                var stats = await client.GetServiceStatsAsync(null, context);
            }
        }
    }
}
```

### <a name="more-information"></a><span data-ttu-id="76904-298">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="76904-298">More information</span></span>
* [<span data-ttu-id="76904-299">Рекомендации по настройке политики повтора для клиентской библиотеки службы хранилища Azure</span><span class="sxs-lookup"><span data-stu-id="76904-299">Azure Storage Client Library Retry Policy Recommendations</span></span>](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [<span data-ttu-id="76904-300">Клиентская библиотека службы хранилища 2.0: реализация политики повтора</span><span class="sxs-lookup"><span data-stu-id="76904-300">Storage Client Library 2.0 – Implementing Retry Policies</span></span>](http://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="sql-database-using-entity-framework-6-retry-guidelines"></a><span data-ttu-id="76904-301">Руководство по использованию механизма повторов для базы данных SQL с помощью инструкций Entity Framework 6</span><span class="sxs-lookup"><span data-stu-id="76904-301">SQL Database using Entity Framework 6 retry guidelines</span></span>
<span data-ttu-id="76904-302">База данных SQL является размещенной базой данных SQL различных размеров и является как стандартной (общей), так и премиальной (частной) службой.</span><span class="sxs-lookup"><span data-stu-id="76904-302">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span> <span data-ttu-id="76904-303">Механизм Entity Framework является модулем объектно-реляционного сопоставления, который позволяет разработчикам .NET работать с реляционными данными с помощью специфических для домена объектов.</span><span class="sxs-lookup"><span data-stu-id="76904-303">Entity Framework is an object-relational mapper that enables .NET developers to work with relational data using domain-specific objects.</span></span> <span data-ttu-id="76904-304">Это исключает необходимость в большинстве кодов доступа к данным, которые обычно требуется писать разработчикам.</span><span class="sxs-lookup"><span data-stu-id="76904-304">It eliminates the need for most of the data-access code that developers usually need to write.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="76904-305">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="76904-305">Retry mechanism</span></span>
<span data-ttu-id="76904-306">Поддержка повторных попыток при доступе к базе данных SQL с помощью Entity Framework 6.0 и выше предоставляется через механизм, называемый [Устойчивость соединения и логика повтора](http://msdn.microsoft.com/data/dn456835.aspx).</span><span class="sxs-lookup"><span data-stu-id="76904-306">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher through a mechanism called [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span> <span data-ttu-id="76904-307">Ниже перечислены основные возможности механизма повтора.</span><span class="sxs-lookup"><span data-stu-id="76904-307">The main features of the retry mechanism are:</span></span>

* <span data-ttu-id="76904-308">Интерфейс **IDbExecutionStrategy** является основной абстракцией.</span><span class="sxs-lookup"><span data-stu-id="76904-308">The primary abstraction is the **IDbExecutionStrategy** interface.</span></span> <span data-ttu-id="76904-309">Этот интерфейс</span><span class="sxs-lookup"><span data-stu-id="76904-309">This interface:</span></span>
  * <span data-ttu-id="76904-310">Определяет синхронные и асинхронные методы \*\*Execute\*\*\*.</span><span class="sxs-lookup"><span data-stu-id="76904-310">Defines synchronous and asynchronous **Execute**\* methods.</span></span>
  * <span data-ttu-id="76904-311">Определяет классы для непосредственного использования, или которые могут быть настроены на контекст базы данных как стратегию по умолчанию, с сопоставленным именем поставщика или сопоставленным именем поставщика и именем сервера.</span><span class="sxs-lookup"><span data-stu-id="76904-311">Defines classes that can be used directly or can be configured on a database context as a default strategy, mapped to provider name, or mapped to a provider name and server name.</span></span> <span data-ttu-id="76904-312">При настройке на контекст повторы происходят на уровне операций с отдельными базами данных, из которых несколько может соответствовать заданному контексту операции.</span><span class="sxs-lookup"><span data-stu-id="76904-312">When configured on a context, retries occur at the level of individual database operations, of which there might be several for a given context operation.</span></span>
  * <span data-ttu-id="76904-313">Определяет, когда и каким образом следует выполнить повтор при ошибке соединения.</span><span class="sxs-lookup"><span data-stu-id="76904-313">Defines when to retry a failed connection, and how.</span></span>
* <span data-ttu-id="76904-314">Включает несколько встроенных реализаций интерфейса **IDbExecutionStrategy** .</span><span class="sxs-lookup"><span data-stu-id="76904-314">It includes several built-in implementations of the **IDbExecutionStrategy** interface:</span></span>
  * <span data-ttu-id="76904-315">Значение по умолчанию — не повторять.</span><span class="sxs-lookup"><span data-stu-id="76904-315">Default - no retrying.</span></span>
  * <span data-ttu-id="76904-316">По умолчанию для базы данных SQL (автоматический режим) — не повторять, но проверять исключения и завершать их с предложением использовать стратегию базы данных SQL.</span><span class="sxs-lookup"><span data-stu-id="76904-316">Default for SQL Database (automatic) - no retrying, but inspects exceptions and wraps them with suggestion to use the SQL Database strategy.</span></span>
  * <span data-ttu-id="76904-317">По умолчанию для базы данных SQL — экспоненциальный повтор (наследуется от базового класса) плюс логика обнаружения базы данных SQL.</span><span class="sxs-lookup"><span data-stu-id="76904-317">Default for SQL Database - exponential (inherited from base class) plus SQL Database detection logic.</span></span>
* <span data-ttu-id="76904-318">Это реализует стратегию экспоненциальной отсрочки, которая также включает случайный выбор.</span><span class="sxs-lookup"><span data-stu-id="76904-318">It implements an exponential back-off strategy that includes randomization.</span></span>
* <span data-ttu-id="76904-319">Встроенные классы повтора включают отслеживание состояния и не являются потокобезопасными.</span><span class="sxs-lookup"><span data-stu-id="76904-319">The built-in retry classes are stateful and are not thread safe.</span></span> <span data-ttu-id="76904-320">Однако они могут использоваться повторно после завершения текущей операции.</span><span class="sxs-lookup"><span data-stu-id="76904-320">However, they can be reused after the current operation is completed.</span></span>
* <span data-ttu-id="76904-321">В случае превышения заданного счетчика повторов результаты помещаются в новое исключение.</span><span class="sxs-lookup"><span data-stu-id="76904-321">If the specified retry count is exceeded, the results are wrapped in a new exception.</span></span> <span data-ttu-id="76904-322">Это не вызывает перенаправления текущего исключения.</span><span class="sxs-lookup"><span data-stu-id="76904-322">It does not bubble up the current exception.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="76904-323">Настройки политики</span><span class="sxs-lookup"><span data-stu-id="76904-323">Policy configuration</span></span>
<span data-ttu-id="76904-324">Поддержка повторных попыток предоставляется при доступе к базе данных SQL с помощью Entity Framework 6.0 и более поздних версий.</span><span class="sxs-lookup"><span data-stu-id="76904-324">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher.</span></span> <span data-ttu-id="76904-325">Политики повтора настраиваются программным способом.</span><span class="sxs-lookup"><span data-stu-id="76904-325">Retry policies are configured programmatically.</span></span> <span data-ttu-id="76904-326">Нельзя изменить конфигурацию для каждой операции.</span><span class="sxs-lookup"><span data-stu-id="76904-326">The configuration cannot be changed on a per-operation basis.</span></span>

<span data-ttu-id="76904-327">При настройке стратегии в контексте по умолчанию, необходимо указать функцию, которая создает новые стратегии по требованию.</span><span class="sxs-lookup"><span data-stu-id="76904-327">When configuring a strategy on the context as the default, you specify a function that creates a new strategy on demand.</span></span> <span data-ttu-id="76904-328">В следующем коде показано, как можно создать класс конфигурации повтора, который расширяет базовый класс **DbConfiguration** .</span><span class="sxs-lookup"><span data-stu-id="76904-328">The following code shows how you can create a retry configuration class that extends the **DbConfiguration** base class.</span></span>

```csharp
public class BloggingContextConfiguration : DbConfiguration
{
  public BlogConfiguration()
  {
    // Set up the execution strategy for SQL Database (exponential) with 5 retries and 4 sec delay
    this.SetExecutionStrategy(
         "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4)));
  }
}
```

<span data-ttu-id="76904-329">Затем можно указать это в качестве стратегии повторных попыток по умолчанию для всех операций с помощью метода **SetConfiguration** экземпляра **DbConfiguration** при запуске приложения.</span><span class="sxs-lookup"><span data-stu-id="76904-329">You can then specify this as the default retry strategy for all operations using the **SetConfiguration** method of the **DbConfiguration** instance when the application starts.</span></span> <span data-ttu-id="76904-330">По умолчанию EF будет автоматически обнаруживать и использовать класс конфигурации.</span><span class="sxs-lookup"><span data-stu-id="76904-330">By default, EF will automatically discover and use the configuration class.</span></span>

    DbConfiguration.SetConfiguration(new BloggingContextConfiguration());

<span data-ttu-id="76904-331">Класс конфигурации повторных попыток для контекста задается дополнением контекста класса атрибутом **DbConfigurationType** .</span><span class="sxs-lookup"><span data-stu-id="76904-331">You can specify the retry configuration class for a context by annotating the context class with a **DbConfigurationType** attribute.</span></span> <span data-ttu-id="76904-332">Однако при наличии только одного класса конфигурации EF будет использовать его без необходимости дополнения контекста.</span><span class="sxs-lookup"><span data-stu-id="76904-332">However, if you have only one configuration class, EF will use it without the need to annotate the context.</span></span>

    [DbConfigurationType(typeof(BloggingContextConfiguration))]
    public class BloggingContext : DbContext
    { ...

<span data-ttu-id="76904-333">Если необходимо использовать различные стратегии для конкретных операций или отключить повторы для определенных операций, то можно создать класс конфигурации, который позволяет приостановить или заменить стратегии, установив флаг **CallContext**.</span><span class="sxs-lookup"><span data-stu-id="76904-333">If you need to use different retry strategies for specific operations, or disable retries for specific operations, you can create a configuration class that allows you to suspend or swap strategies by setting a flag in the **CallContext**.</span></span> <span data-ttu-id="76904-334">Класс конфигурации может использовать этот флаг для переключения стратегии или отключения стратегии, предоставленной вами, с последующим использованием стратегии по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="76904-334">The configuration class can use this flag to switch strategies, or disable the strategy you provide and use a default strategy.</span></span> <span data-ttu-id="76904-335">Дополнительные сведения см. в разделе [Приостановка выполнения стратегии](http://msdn.microsoft.com/dn307226#transactions_workarounds) на странице "Ограничения стратегий выполнения повторов (для EF6 и выше)".</span><span class="sxs-lookup"><span data-stu-id="76904-335">For more information, see [Suspend Execution Strategy](http://msdn.microsoft.com/dn307226#transactions_workarounds) in the page Limitations with Retrying Execution Strategies (EF6 onwards).</span></span>

<span data-ttu-id="76904-336">Иным способом использования стратегий повторов, определенных для отдельных операций, является создание экземпляра класса требуемой стратегии и передача ему нужных параметров.</span><span class="sxs-lookup"><span data-stu-id="76904-336">Another technique for using specific retry strategies for individual operations is to create an instance of the required strategy class and supply the desired settings through parameters.</span></span> <span data-ttu-id="76904-337">Затем можно вызвать его метод **ExecuteAsync** .</span><span class="sxs-lookup"><span data-stu-id="76904-337">You then invoke its **ExecuteAsync** method.</span></span>

    var executionStrategy = new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4));
    var blogs = await executionStrategy.ExecuteAsync(
        async () =>
        {
            using (var db = new BloggingContext("Blogs"))
            {
                // Acquire some values asynchronously and return them
            }
        },
        new CancellationToken()
    );

<span data-ttu-id="76904-338">Самый простой способ использовать класс **DbConfiguration** — это обнаружить его в той же сборке, что и класс **DbContext**.</span><span class="sxs-lookup"><span data-stu-id="76904-338">The simplest way to use a **DbConfiguration** class is to locate it in the same assembly as the **DbContext** class.</span></span> <span data-ttu-id="76904-339">Однако этот способ не подходит в случае, когда тот же контекст требуется в различных сценариях, таких как различные интерактивные и фоновые стратегии повторов.</span><span class="sxs-lookup"><span data-stu-id="76904-339">However, this is not appropriate when the same context is required in different scenarios, such as different interactive and background retry strategies.</span></span> <span data-ttu-id="76904-340">Если разные контексты выполняются в отдельных доменах приложений, можно использовать встроенную поддержку для указания классов конфигурации в файле конфигурации или задать ее явным образом с помощью кода.</span><span class="sxs-lookup"><span data-stu-id="76904-340">If the different contexts execute in separate AppDomains, you can use the built-in support for specifying configuration classes in the configuration file or set it explicitly using code.</span></span> <span data-ttu-id="76904-341">Если различные контексты должны выполняться в том же домене приложения, потребуется создание пользовательского решения.</span><span class="sxs-lookup"><span data-stu-id="76904-341">If the different contexts must execute in the same AppDomain, a custom solution will be required.</span></span>

<span data-ttu-id="76904-342">Дополнительные сведения см. в статье [Конфигурация на основе кода (для EF6 и выше)](http://msdn.microsoft.com/data/jj680699.aspx).</span><span class="sxs-lookup"><span data-stu-id="76904-342">For more information, see [Code-Based Configuration (EF6 onwards)](http://msdn.microsoft.com/data/jj680699.aspx).</span></span>

<span data-ttu-id="76904-343">Следующая таблица показывает значения по умолчанию для встроенной политики повторов при использовании EF6.</span><span class="sxs-lookup"><span data-stu-id="76904-343">The following table shows the default settings for the built-in retry policy when using EF6.</span></span>

| <span data-ttu-id="76904-344">Параметр</span><span class="sxs-lookup"><span data-stu-id="76904-344">Setting</span></span> | <span data-ttu-id="76904-345">Значение по умолчанию</span><span class="sxs-lookup"><span data-stu-id="76904-345">Default value</span></span> | <span data-ttu-id="76904-346">Значение</span><span class="sxs-lookup"><span data-stu-id="76904-346">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="76904-347">Политика</span><span class="sxs-lookup"><span data-stu-id="76904-347">Policy</span></span> | <span data-ttu-id="76904-348">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="76904-348">Exponential</span></span> | <span data-ttu-id="76904-349">Экспоненциальная задержка.</span><span class="sxs-lookup"><span data-stu-id="76904-349">Exponential back-off.</span></span> |
| <span data-ttu-id="76904-350">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="76904-350">MaxRetryCount</span></span> | <span data-ttu-id="76904-351">5</span><span class="sxs-lookup"><span data-stu-id="76904-351">5</span></span> | <span data-ttu-id="76904-352">Максимальное число попыток.</span><span class="sxs-lookup"><span data-stu-id="76904-352">The maximum number of retries.</span></span> |
| <span data-ttu-id="76904-353">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="76904-353">MaxDelay</span></span> | <span data-ttu-id="76904-354">30 секунд</span><span class="sxs-lookup"><span data-stu-id="76904-354">30 seconds</span></span> | <span data-ttu-id="76904-355">Максимальная задержка между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="76904-355">The maximum delay between retries.</span></span> <span data-ttu-id="76904-356">Это значение не влияет на то, как вычисляется значение серии задержек.</span><span class="sxs-lookup"><span data-stu-id="76904-356">This value does not affect how the series of delays are computed.</span></span> <span data-ttu-id="76904-357">Оно определяет только верхнюю границу.</span><span class="sxs-lookup"><span data-stu-id="76904-357">It only defines an upper bound.</span></span> |
| <span data-ttu-id="76904-358">DefaultCoefficient</span><span class="sxs-lookup"><span data-stu-id="76904-358">DefaultCoefficient</span></span> | <span data-ttu-id="76904-359">1 с</span><span class="sxs-lookup"><span data-stu-id="76904-359">1 second</span></span> | <span data-ttu-id="76904-360">Коэффициент для вычисления значения экспоненциальной задержки.</span><span class="sxs-lookup"><span data-stu-id="76904-360">The coefficient for the exponential back-off computation.</span></span> <span data-ttu-id="76904-361">Это значение не может быть изменено.</span><span class="sxs-lookup"><span data-stu-id="76904-361">This value cannot be changed.</span></span> |
| <span data-ttu-id="76904-362">DefaultRandomFactor</span><span class="sxs-lookup"><span data-stu-id="76904-362">DefaultRandomFactor</span></span> | <span data-ttu-id="76904-363">1,1</span><span class="sxs-lookup"><span data-stu-id="76904-363">1.1</span></span> | <span data-ttu-id="76904-364">Множитель, который используется для добавления значения случайной задержки для каждой записи.</span><span class="sxs-lookup"><span data-stu-id="76904-364">The multiplier used to add a random delay for each entry.</span></span> <span data-ttu-id="76904-365">Это значение не может быть изменено.</span><span class="sxs-lookup"><span data-stu-id="76904-365">This value cannot be changed.</span></span> |
| <span data-ttu-id="76904-366">DefaultExponentialBase</span><span class="sxs-lookup"><span data-stu-id="76904-366">DefaultExponentialBase</span></span> | <span data-ttu-id="76904-367">2</span><span class="sxs-lookup"><span data-stu-id="76904-367">2</span></span> | <span data-ttu-id="76904-368">Множитель, который используется для вычисления значения следующей задержки.</span><span class="sxs-lookup"><span data-stu-id="76904-368">The multiplier used to calculate the next delay.</span></span> <span data-ttu-id="76904-369">Это значение не может быть изменено.</span><span class="sxs-lookup"><span data-stu-id="76904-369">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="76904-370">Руководство по использованию механизма повторов</span><span class="sxs-lookup"><span data-stu-id="76904-370">Retry usage guidance</span></span>
<span data-ttu-id="76904-371">При доступе к базе данных SQL с помощью EF6, придерживайтесь следующих рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="76904-371">Consider the following guidelines when accessing SQL Database using EF6:</span></span>

* <span data-ttu-id="76904-372">Выберите соответствующую службу (общую или премиальную).</span><span class="sxs-lookup"><span data-stu-id="76904-372">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="76904-373">Общий экземпляр может испытывать большие задержки подключения и регулирования чем обычно из-за использования другими клиентами общего сервера.</span><span class="sxs-lookup"><span data-stu-id="76904-373">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="76904-374">Если требуется выполнение надежных операций с низкой задержкой и прогнозируемая производительность, попробуйте выбрать премиальный вариант.</span><span class="sxs-lookup"><span data-stu-id="76904-374">If predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="76904-375">Стратегия фиксированного интервала не рекомендуется для использования с базой данных SQL Azure.</span><span class="sxs-lookup"><span data-stu-id="76904-375">A fixed interval strategy is not recommended for use with Azure SQL Database.</span></span> <span data-ttu-id="76904-376">Вместо этого используйте стратегию экспоненциальной отсрочки, так как служба может быть перегружена и большее время задержки обеспечит большее время для ее восстановления.</span><span class="sxs-lookup"><span data-stu-id="76904-376">Instead, use an exponential back-off strategy because the service may be overloaded, and longer delays allow more time for it to recover.</span></span>
* <span data-ttu-id="76904-377">Выберите подходящее значение для времени ожидания подключения и команды при определении подключения.</span><span class="sxs-lookup"><span data-stu-id="76904-377">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="76904-378">Выберите значение времени ожидания на основе логики вашей работы и на основе испытаний.</span><span class="sxs-lookup"><span data-stu-id="76904-378">Base the timeout on both your business logic design and through testing.</span></span> <span data-ttu-id="76904-379">Может потребоваться изменить это значение по мере изменения объемов данных или бизнес-процессов.</span><span class="sxs-lookup"><span data-stu-id="76904-379">You may need to modify this value over time as the volumes of data or the business processes change.</span></span> <span data-ttu-id="76904-380">Слишком короткое время ожидания может привести к преждевременным сбоям подключений, когда база данных занята.</span><span class="sxs-lookup"><span data-stu-id="76904-380">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="76904-381">Слишком длинное время ожидания может помешать правильной работе логики повторов из-за слишком долгого ожидания до обнаружения ошибки соединения.</span><span class="sxs-lookup"><span data-stu-id="76904-381">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="76904-382">Значение времени ожидания — это компонент сквозной задержки, несмотря на это вы не можете легко определить, сколько команд будет выполнено при сохранении контекста.</span><span class="sxs-lookup"><span data-stu-id="76904-382">The value of the timeout is a component of the end-to-end latency, although you cannot easily determine how many commands will execute when saving the context.</span></span> <span data-ttu-id="76904-383">Время ожидания по умолчанию можно изменить, задав свойство **CommandTimeout** экземпляра **DbContext**.</span><span class="sxs-lookup"><span data-stu-id="76904-383">You can change the default timeout by setting the **CommandTimeout** property of the **DbContext** instance.</span></span>
* <span data-ttu-id="76904-384">Платформа Entity Framework поддерживает конфигурации повторов, определенные в файлах конфигурации.</span><span class="sxs-lookup"><span data-stu-id="76904-384">Entity Framework supports retry configurations defined in configuration files.</span></span> <span data-ttu-id="76904-385">Однако для максимальной гибкости в Azure необходимо создать конфигурацию программным способом в приложении.</span><span class="sxs-lookup"><span data-stu-id="76904-385">However, for maximum flexibility on Azure you should consider creating the configuration programmatically within the application.</span></span> <span data-ttu-id="76904-386">Определенные параметры политик повторов, например количество повторных попыток и интервалы повтора, можно сохранить в файле конфигурации службы и использовать во время выполнения для создания соответствующих политик.</span><span class="sxs-lookup"><span data-stu-id="76904-386">The specific parameters for the retry policies, such as the number of retries and the retry intervals, can be stored in the service configuration file and used at runtime to create the appropriate policies.</span></span> <span data-ttu-id="76904-387">Это позволяет изменять параметры без перезапуска приложения.</span><span class="sxs-lookup"><span data-stu-id="76904-387">This allows the settings to be changed without requiring the application to be restarted.</span></span>

<span data-ttu-id="76904-388">Для начала установите следующие параметры для операций повтора.</span><span class="sxs-lookup"><span data-stu-id="76904-388">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="76904-389">Нельзя указать задержку между попытками повтора (она является фиксированной и генерируется в виде экспоненциальной последовательности).</span><span class="sxs-lookup"><span data-stu-id="76904-389">You cannot specify the delay between retry attempts (it is fixed and generated as an exponential sequence).</span></span> <span data-ttu-id="76904-390">Максимальные значения можно указать, как показано ниже, если только вы не создаете пользовательскую стратегию.</span><span class="sxs-lookup"><span data-stu-id="76904-390">You can specify only the maximum values, as shown here; unless you create a custom retry strategy.</span></span> <span data-ttu-id="76904-391">Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.</span><span class="sxs-lookup"><span data-stu-id="76904-391">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="76904-392">**Контекст**</span><span class="sxs-lookup"><span data-stu-id="76904-392">**Context**</span></span> | <span data-ttu-id="76904-393">**Максимальная задержка<br />примера целевого E2E**</span><span class="sxs-lookup"><span data-stu-id="76904-393">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="76904-394">**Политика повтора**</span><span class="sxs-lookup"><span data-stu-id="76904-394">**Retry policy**</span></span> | <span data-ttu-id="76904-395">**Параметры**</span><span class="sxs-lookup"><span data-stu-id="76904-395">**Settings**</span></span> | <span data-ttu-id="76904-396">**Значения**</span><span class="sxs-lookup"><span data-stu-id="76904-396">**Values**</span></span> | <span data-ttu-id="76904-397">**Принцип работы**</span><span class="sxs-lookup"><span data-stu-id="76904-397">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="76904-398">Интерактивный, пользовательский интерфейс</span><span class="sxs-lookup"><span data-stu-id="76904-398">Interactive, UI,</span></span><br /><span data-ttu-id="76904-399">или передний план</span><span class="sxs-lookup"><span data-stu-id="76904-399">or foreground</span></span> |<span data-ttu-id="76904-400">2 секунды</span><span class="sxs-lookup"><span data-stu-id="76904-400">2 seconds</span></span> |<span data-ttu-id="76904-401">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="76904-401">Exponential</span></span> |<span data-ttu-id="76904-402">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="76904-402">MaxRetryCount</span></span><br /><span data-ttu-id="76904-403">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="76904-403">MaxDelay</span></span> |<span data-ttu-id="76904-404">3</span><span class="sxs-lookup"><span data-stu-id="76904-404">3</span></span><br /><span data-ttu-id="76904-405">750 мс</span><span class="sxs-lookup"><span data-stu-id="76904-405">750 ms</span></span> |<span data-ttu-id="76904-406">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="76904-406">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="76904-407">Попытка 2 — задержка 750 мс</span><span class="sxs-lookup"><span data-stu-id="76904-407">Attempt 2 - delay 750 ms</span></span><br /><span data-ttu-id="76904-408">Попытка 3 – задержка 750 мс</span><span class="sxs-lookup"><span data-stu-id="76904-408">Attempt 3 – delay 750 ms</span></span> |
| <span data-ttu-id="76904-409">Фоновый</span><span class="sxs-lookup"><span data-stu-id="76904-409">Background</span></span><br /> <span data-ttu-id="76904-410">или пакетный</span><span class="sxs-lookup"><span data-stu-id="76904-410">or batch</span></span> |<span data-ttu-id="76904-411">30 секунд</span><span class="sxs-lookup"><span data-stu-id="76904-411">30 seconds</span></span> |<span data-ttu-id="76904-412">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="76904-412">Exponential</span></span> |<span data-ttu-id="76904-413">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="76904-413">MaxRetryCount</span></span><br /><span data-ttu-id="76904-414">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="76904-414">MaxDelay</span></span> |<span data-ttu-id="76904-415">5</span><span class="sxs-lookup"><span data-stu-id="76904-415">5</span></span><br /><span data-ttu-id="76904-416">12 секунд</span><span class="sxs-lookup"><span data-stu-id="76904-416">12 seconds</span></span> |<span data-ttu-id="76904-417">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="76904-417">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="76904-418">Попытка 2 — задержка 1 с</span><span class="sxs-lookup"><span data-stu-id="76904-418">Attempt 2 - delay ~1 sec</span></span><br /><span data-ttu-id="76904-419">Попытка 3 — задержка 3 с</span><span class="sxs-lookup"><span data-stu-id="76904-419">Attempt 3 - delay ~3 sec</span></span><br /><span data-ttu-id="76904-420">Попытка 4 — задержка 7 с</span><span class="sxs-lookup"><span data-stu-id="76904-420">Attempt 4 - delay ~7 sec</span></span><br /><span data-ttu-id="76904-421">Попытка 5 — задержка 12 с</span><span class="sxs-lookup"><span data-stu-id="76904-421">Attempt 5 - delay 12 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="76904-422">Целевые значения сквозной задержки подразумевают время ожидания по умолчанию для подключения к службе.</span><span class="sxs-lookup"><span data-stu-id="76904-422">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="76904-423">При указании более длинного времени ожидания подключения величина сквозной задержки будет увеличена путем добавления этого дополнительного времени для каждой попытки повтора.</span><span class="sxs-lookup"><span data-stu-id="76904-423">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="76904-424">Примеры</span><span class="sxs-lookup"><span data-stu-id="76904-424">Examples</span></span>
<span data-ttu-id="76904-425">В следующем примере кода определяется простое решение доступа к данным, использующее Entity Framework.</span><span class="sxs-lookup"><span data-stu-id="76904-425">The following code example defines a simple data access solution that uses Entity Framework.</span></span> <span data-ttu-id="76904-426">Программа определяет конкретную стратегию путем определения класса с именем экземпляра **BlogConfiguration**, который расширяет **DbConfiguration**.</span><span class="sxs-lookup"><span data-stu-id="76904-426">It sets a specific retry strategy by defining an instance of a class named **BlogConfiguration** that extends **DbConfiguration**.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.Entity.SqlServer;
using System.Threading.Tasks;

namespace RetryCodeSamples
{
    public class BlogConfiguration : DbConfiguration
    {
        public BlogConfiguration()
        {
            // Set up the execution strategy for SQL Database (exponential) with 5 retries and 12 sec delay.
            // These values could be loaded from configuration rather than being hard-coded.
            this.SetExecutionStrategy(
                    "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(12)));
        }
    }

    // Specify the configuration type if more than one has been defined.
    // [DbConfigurationType(typeof(BlogConfiguration))]
    public class BloggingContext : DbContext
    {
        // Definition of content goes here.
    }

    class EF6CodeSamples
    {
        public async static Task Samples()
        {
            // Execution strategy configured by DbConfiguration subclass, discovered automatically or
            // or explicitly indicated through configuration or with an attribute. Default is no retries.
            using (var db = new BloggingContext("Blogs"))
            {
                // Add, edit, delete blog items here, then:
                await db.SaveChangesAsync();
            }
        }
    }
}
```

<span data-ttu-id="76904-427">Дополнительные примеры использования механизма повтора Entity Framework можно найти в разделе [Устойчивость соединения и логика повтора](http://msdn.microsoft.com/data/dn456835.aspx).</span><span class="sxs-lookup"><span data-stu-id="76904-427">More examples of using the Entity Framework retry mechanism can be found in [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span>

### <a name="more-information"></a><span data-ttu-id="76904-428">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="76904-428">More information</span></span>
* [<span data-ttu-id="76904-429">Руководство по эластичности и производительности базы данных SQL Azure</span><span class="sxs-lookup"><span data-stu-id="76904-429">Azure SQL Database Performance and Elasticity Guide</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core-retry-guidelines"></a><span data-ttu-id="76904-430">Руководство по использованию механизма повторов для базы данных SQL с помощью Entity Framework Core</span><span class="sxs-lookup"><span data-stu-id="76904-430">SQL Database using Entity Framework Core retry guidelines</span></span>
<span data-ttu-id="76904-431">Механизм [Entity Framework Core](/ef/core/) отслеживает взаимоотношения между объектами и позволяет разработчикам .NET использовать объекты на уровне доменов при работе с данными.</span><span class="sxs-lookup"><span data-stu-id="76904-431">[Entity Framework Core](/ef/core/) is an object-relational mapper that enables .NET Core developers to work with data using domain-specific objects.</span></span> <span data-ttu-id="76904-432">Это исключает необходимость в большинстве кодов доступа к данным, которые обычно требуется писать разработчикам.</span><span class="sxs-lookup"><span data-stu-id="76904-432">It eliminates the need for most of the data-access code that developers usually need to write.</span></span> <span data-ttu-id="76904-433">Эта версия Entity Framework полностью создана с нуля и по умолчанию не наследует возможности EF6.x.</span><span class="sxs-lookup"><span data-stu-id="76904-433">This version of Entity Framework was written from the ground up, and doesn't automatically inherit all the features from EF6.x.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="76904-434">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="76904-434">Retry mechanism</span></span>
<span data-ttu-id="76904-435">Поддержка повторных попыток при доступе к базе данных SQL с использованием Entity Framework Core предоставляется через механизм, называемый [Устойчивость подключения](/ef/core/miscellaneous/connection-resiliency).</span><span class="sxs-lookup"><span data-stu-id="76904-435">Retry support is provided when accessing SQL Database using Entity Framework Core through a mechanism called [Connection Resiliency](/ef/core/miscellaneous/connection-resiliency).</span></span> <span data-ttu-id="76904-436">Устойчивость подключения впервые появилась в версии EF Core 1.1.0.</span><span class="sxs-lookup"><span data-stu-id="76904-436">Connection resiliency was introduced in EF Core 1.1.0.</span></span>

<span data-ttu-id="76904-437">Основным объектом абстракции является интерфейс `IExecutionStrategy`.</span><span class="sxs-lookup"><span data-stu-id="76904-437">The primary abstraction is the `IExecutionStrategy` interface.</span></span> <span data-ttu-id="76904-438">Стратегия выполнения для SQL Server, включая SQL Azure, учитывает типы исключений, для которых возможны повторные попытки. Для нее предусмотрены стандартные рациональные параметры, определяющие максимальное число повторных попыток, пауз между ними и т. д.</span><span class="sxs-lookup"><span data-stu-id="76904-438">The execution strategy for SQL Server, including SQL Azure, is aware of the exception types that can be retried and has sensible defaults for maximum retries, delay between retries, and so on.</span></span>

### <a name="examples"></a><span data-ttu-id="76904-439">Примеры</span><span class="sxs-lookup"><span data-stu-id="76904-439">Examples</span></span>

<span data-ttu-id="76904-440">Следующий пример кода использует автоматические повторные попытки при настройке объекта DbContext, который представляет сеанс подключения к базе данных.</span><span class="sxs-lookup"><span data-stu-id="76904-440">The following code enables automatic retries when configuring the DbContext object, which represents a session with the database.</span></span> 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

<span data-ttu-id="76904-441">Следующий пример кода демонстрирует, как выполнять транзакцию с автоматическими повторными попытками с использованием стратегии выполнения.</span><span class="sxs-lookup"><span data-stu-id="76904-441">The following code shows how to execute a transaction with automatic retries, by using an execution strategy.</span></span> <span data-ttu-id="76904-442">Транзакция определяется в делегате.</span><span class="sxs-lookup"><span data-stu-id="76904-442">The transaction is defined in a delegate.</span></span> <span data-ttu-id="76904-443">Если происходит временный сбой, стратегия выполнения снова вызывает делегат.</span><span class="sxs-lookup"><span data-stu-id="76904-443">If a transient failure occurs, the execution strategy will invoke the delegate again.</span></span>

```csharp
using (var db = new BloggingContext())
{
    var strategy = db.Database.CreateExecutionStrategy();

    strategy.Execute(() =>
    {
        using (var transaction = db.Database.BeginTransaction())
        {
            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/dotnet" });
            db.SaveChanges();

            db.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/visualstudio" });
            db.SaveChanges();

            transaction.Commit();
        }
    });
}
```

### <a name="more-information"></a><span data-ttu-id="76904-444">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="76904-444">More information</span></span>
* [<span data-ttu-id="76904-445">Устойчивость подключения</span><span class="sxs-lookup"><span data-stu-id="76904-445">Connection Resiliency</span></span>](/ef/core/miscellaneous/connection-resiliency)
* [<span data-ttu-id="76904-446">Точки данных для EF Core 1.1</span><span class="sxs-lookup"><span data-stu-id="76904-446">Data Points - EF Core 1.1</span></span>](https://msdn.microsoft.com/magazine/mt745093.aspx)

## <a name="sql-database-using-adonet-retry-guidelines"></a><span data-ttu-id="76904-447">Руководство по использованию механизма повторов с базой данных SQL с использованием ADO.NET</span><span class="sxs-lookup"><span data-stu-id="76904-447">SQL Database using ADO.NET retry guidelines</span></span>
<span data-ttu-id="76904-448">База данных SQL является размещенной базой данных SQL различных размеров и является как стандартной (общей), так и премиальной (частной) службой.</span><span class="sxs-lookup"><span data-stu-id="76904-448">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="76904-449">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="76904-449">Retry mechanism</span></span>
<span data-ttu-id="76904-450">База данных SQL не имеет встроенной поддержки выполнения повторных попыток, если доступ осуществляется с помощью ADO.NET.</span><span class="sxs-lookup"><span data-stu-id="76904-450">SQL Database has no built-in support for retries when accessed using ADO.NET.</span></span> <span data-ttu-id="76904-451">Однако коды возврата от запросов могут быть использованы для определения причины сбоя запроса.</span><span class="sxs-lookup"><span data-stu-id="76904-451">However, the return codes from requests can be used to determine why a request failed.</span></span> <span data-ttu-id="76904-452">Дополнительные сведения о регулировании базы данных SQL см. в статье [Ограничения ресурсов базы данных SQL Azure](/azure/sql-database/sql-database-resource-limits).</span><span class="sxs-lookup"><span data-stu-id="76904-452">For more information about SQL Database throttling, see [Azure SQL Database resource limits](/azure/sql-database/sql-database-resource-limits).</span></span> <span data-ttu-id="76904-453">Список кодов ошибок см. в описании [кодов ошибок SQL для клиентских приложений Базы данных SQL](/azure/sql-database/sql-database-develop-error-messages).</span><span class="sxs-lookup"><span data-stu-id="76904-453">For a list of relevant error codes, see [SQL error codes for SQL Database client applications](/azure/sql-database/sql-database-develop-error-messages).</span></span>

<span data-ttu-id="76904-454">Для реализации повторных попыток при обращении к Базе данных SQL вы можете использовать библиотеку Polly.</span><span class="sxs-lookup"><span data-stu-id="76904-454">You can use the Polly library to implement retries for SQL Database.</span></span> <span data-ttu-id="76904-455">См. руководство по [обработке временного сбоя с помощью Polly](#transient-fault-handling-with-polly).</span><span class="sxs-lookup"><span data-stu-id="76904-455">See [Transient fault handling with Polly](#transient-fault-handling-with-polly).</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="76904-456">Руководство по использованию механизма повторов</span><span class="sxs-lookup"><span data-stu-id="76904-456">Retry usage guidance</span></span>
<span data-ttu-id="76904-457">При доступе к базе данных SQL с помощью ADO.NET придерживайтесь следующих рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="76904-457">Consider the following guidelines when accessing SQL Database using ADO.NET:</span></span>

* <span data-ttu-id="76904-458">Выберите соответствующую службу (общую или премиальную).</span><span class="sxs-lookup"><span data-stu-id="76904-458">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="76904-459">Общий экземпляр может испытывать большие задержки подключения и регулирования чем обычно из-за использования другими клиентами общего сервера.</span><span class="sxs-lookup"><span data-stu-id="76904-459">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="76904-460">Если требуется более прогнозируемая производительность и выполнение надежных операций с низкой задержкой, попробуйте выбрать премиальный вариант.</span><span class="sxs-lookup"><span data-stu-id="76904-460">If more predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="76904-461">Убедитесь, что выполнение повторов происходит на соответствующем уровне или области во избежание операций, не являющихся идемпотентными, что может вызвать несоответствие в данных.</span><span class="sxs-lookup"><span data-stu-id="76904-461">Ensure that you perform retries at the appropriate level or scope to avoid non-idempotent operations causing inconsistency in the data.</span></span> <span data-ttu-id="76904-462">В идеальном случае все операции должны быть идемпотентными таким образом, чтобы их можно было повторить без возникновения несоответствия.</span><span class="sxs-lookup"><span data-stu-id="76904-462">Ideally, all operations should be idempotent so that they can be repeated without causing inconsistency.</span></span> <span data-ttu-id="76904-463">Если это не так, повтор следует производить на уровне или в области, которая позволяет отменить все связанные изменения, если одна операция завершится ошибкой. Например, в области транзакций.</span><span class="sxs-lookup"><span data-stu-id="76904-463">Where this is not the case, the retry should be performed at a level or scope that allows all related changes to be undone if one operation fails; for example, from within a transactional scope.</span></span> <span data-ttu-id="76904-464">Для получения дополнительных сведений обратитесь к разделу [Основы облачной службы. Уровень доступа к данным: обработка временных ошибок](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span><span class="sxs-lookup"><span data-stu-id="76904-464">For more information, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span></span>
* <span data-ttu-id="76904-465">Не рекомендуется использование стратегии с фиксированным интервалом при работе с базой данных SQL Azure, кроме интерактивных сценариев при наличии нескольких попыток повтора с очень короткими интервалами.</span><span class="sxs-lookup"><span data-stu-id="76904-465">A fixed interval strategy is not recommended for use with Azure SQL Database except for interactive scenarios where there are only a few retries at very short intervals.</span></span> <span data-ttu-id="76904-466">Вместо этого рекомендуется использовать стратегию экспоненциальной отсрочки для большинства сценариев.</span><span class="sxs-lookup"><span data-stu-id="76904-466">Instead, consider using an exponential back-off strategy for the majority of scenarios.</span></span>
* <span data-ttu-id="76904-467">Выберите подходящее значение для времени ожидания подключения и команды при определении подключения.</span><span class="sxs-lookup"><span data-stu-id="76904-467">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="76904-468">Слишком короткое время ожидания может привести к преждевременным сбоям подключений, когда база данных занята.</span><span class="sxs-lookup"><span data-stu-id="76904-468">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="76904-469">Слишком длинное время ожидания может помешать правильной работе логики повторов из-за слишком долгого ожидания до обнаружения ошибки соединения.</span><span class="sxs-lookup"><span data-stu-id="76904-469">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="76904-470">Значение времени ожидания — это компонент сквозной задержки; фактически это значение добавляется к величине задержки повтора, указанной в политике повтора для каждой попытки.</span><span class="sxs-lookup"><span data-stu-id="76904-470">The value of the timeout is a component of the end-to-end latency; it is effectively added to the retry delay specified in the retry policy for every retry attempt.</span></span>
* <span data-ttu-id="76904-471">Закройте соединение после определенного числа повторных попыток, даже если используется экспоненциальная логика повторных попыток, и повторите операцию с новым подключением.</span><span class="sxs-lookup"><span data-stu-id="76904-471">Close the connection after a certain number of retries, even when using an exponential back off retry logic, and retry the operation on a new connection.</span></span> <span data-ttu-id="76904-472">Повторение одной и той же операции несколько раз для одного соединения может стать фактором возникновения проблем с подключением.</span><span class="sxs-lookup"><span data-stu-id="76904-472">Retrying the same operation multiple times on the same connection can be a factor that contributes to connection problems.</span></span> <span data-ttu-id="76904-473">Пример использования этой технологии см. в статье [Основы облачной службы. Уровень доступа к данным: обработка временных ошибок](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span><span class="sxs-lookup"><span data-stu-id="76904-473">For an example of this technique, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span></span>
* <span data-ttu-id="76904-474">Когда используется пул подключений (по умолчанию), существует вероятность того, что будет выбрано то же подключение из пула, даже после закрытия и повторного открытия соединения.</span><span class="sxs-lookup"><span data-stu-id="76904-474">When connection pooling is in use (the default) there is a chance that the same connection will be chosen from the pool, even after closing and reopening a connection.</span></span> <span data-ttu-id="76904-475">Если это так, способом устранения подобной ситуации является вызов метода **ClearPool** класса **SqlConnection**, чтобы отметить подключение как не используемое повторно.</span><span class="sxs-lookup"><span data-stu-id="76904-475">If this is the case, a technique to resolve it is to call the **ClearPool** method of the **SqlConnection** class to mark the connection as not reusable.</span></span> <span data-ttu-id="76904-476">Тем не менее это следует делать только после сбоя нескольких попыток соединения и только при обнаружении определенного класса временных ошибок, таких как время ожидания SQL (код ошибки -2), связанных со сбоями в подключениях.</span><span class="sxs-lookup"><span data-stu-id="76904-476">However, you should do this only after several connection attempts have failed, and only when encountering the specific class of transient failures such as SQL timeouts (error code -2) related to faulty connections.</span></span>
* <span data-ttu-id="76904-477">Если код доступа к данным использует транзакции, инициируемые как экземпляры **TransactionScope** , то логика повторных попыток должна включать повторное открытие подключения и инициацию новой области транзакции.</span><span class="sxs-lookup"><span data-stu-id="76904-477">If the data access code uses transactions initiated as **TransactionScope** instances, the retry logic should reopen the connection and initiate a new transaction scope.</span></span> <span data-ttu-id="76904-478">По этой причине блок кода повторной попытки должен охватывать всю область транзакции.</span><span class="sxs-lookup"><span data-stu-id="76904-478">For this reason, the retryable code block should encompass the entire scope of the transaction.</span></span>

<span data-ttu-id="76904-479">Начните с использования следующих параметров для операции повтора.</span><span class="sxs-lookup"><span data-stu-id="76904-479">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="76904-480">Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.</span><span class="sxs-lookup"><span data-stu-id="76904-480">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="76904-481">**Контекст**</span><span class="sxs-lookup"><span data-stu-id="76904-481">**Context**</span></span> | <span data-ttu-id="76904-482">**Максимальная задержка<br />примера целевого E2E**</span><span class="sxs-lookup"><span data-stu-id="76904-482">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="76904-483">**Стратегия повторов**</span><span class="sxs-lookup"><span data-stu-id="76904-483">**Retry strategy**</span></span> | <span data-ttu-id="76904-484">**Параметры**</span><span class="sxs-lookup"><span data-stu-id="76904-484">**Settings**</span></span> | <span data-ttu-id="76904-485">**Значения**</span><span class="sxs-lookup"><span data-stu-id="76904-485">**Values**</span></span> | <span data-ttu-id="76904-486">**Принцип работы**</span><span class="sxs-lookup"><span data-stu-id="76904-486">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="76904-487">Интерактивный, пользовательский интерфейс</span><span class="sxs-lookup"><span data-stu-id="76904-487">Interactive, UI,</span></span><br /><span data-ttu-id="76904-488">или передний план</span><span class="sxs-lookup"><span data-stu-id="76904-488">or foreground</span></span> |<span data-ttu-id="76904-489">2 с</span><span class="sxs-lookup"><span data-stu-id="76904-489">2 sec</span></span> |<span data-ttu-id="76904-490">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="76904-490">FixedInterval</span></span> |<span data-ttu-id="76904-491">Число повторных попыток</span><span class="sxs-lookup"><span data-stu-id="76904-491">Retry count</span></span><br /><span data-ttu-id="76904-492">Интервал попытки</span><span class="sxs-lookup"><span data-stu-id="76904-492">Retry interval</span></span><br /><span data-ttu-id="76904-493">Первый быстрый повтор</span><span class="sxs-lookup"><span data-stu-id="76904-493">First fast retry</span></span> |<span data-ttu-id="76904-494">3</span><span class="sxs-lookup"><span data-stu-id="76904-494">3</span></span><br /><span data-ttu-id="76904-495">500 мс</span><span class="sxs-lookup"><span data-stu-id="76904-495">500 ms</span></span><br /><span data-ttu-id="76904-496">true</span><span class="sxs-lookup"><span data-stu-id="76904-496">true</span></span> |<span data-ttu-id="76904-497">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="76904-497">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="76904-498">Попытка 2 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="76904-498">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="76904-499">попытка 3 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="76904-499">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="76904-500">Фоновый</span><span class="sxs-lookup"><span data-stu-id="76904-500">Background</span></span><br /><span data-ttu-id="76904-501">или пакетный</span><span class="sxs-lookup"><span data-stu-id="76904-501">or batch</span></span> |<span data-ttu-id="76904-502">30 с</span><span class="sxs-lookup"><span data-stu-id="76904-502">30 sec</span></span> |<span data-ttu-id="76904-503">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="76904-503">ExponentialBackoff</span></span> |<span data-ttu-id="76904-504">Число повторных попыток</span><span class="sxs-lookup"><span data-stu-id="76904-504">Retry count</span></span><br /><span data-ttu-id="76904-505">Минимальная задержка</span><span class="sxs-lookup"><span data-stu-id="76904-505">Min back-off</span></span><br /><span data-ttu-id="76904-506">Максимальная задержка</span><span class="sxs-lookup"><span data-stu-id="76904-506">Max back-off</span></span><br /><span data-ttu-id="76904-507">Разностная задержка</span><span class="sxs-lookup"><span data-stu-id="76904-507">Delta back-off</span></span><br /><span data-ttu-id="76904-508">Первый быстрый повтор</span><span class="sxs-lookup"><span data-stu-id="76904-508">First fast retry</span></span> |<span data-ttu-id="76904-509">5</span><span class="sxs-lookup"><span data-stu-id="76904-509">5</span></span><br /><span data-ttu-id="76904-510">0 с</span><span class="sxs-lookup"><span data-stu-id="76904-510">0 sec</span></span><br /><span data-ttu-id="76904-511">60 с</span><span class="sxs-lookup"><span data-stu-id="76904-511">60 sec</span></span><br /><span data-ttu-id="76904-512">2 с</span><span class="sxs-lookup"><span data-stu-id="76904-512">2 sec</span></span><br /><span data-ttu-id="76904-513">false</span><span class="sxs-lookup"><span data-stu-id="76904-513">false</span></span> |<span data-ttu-id="76904-514">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="76904-514">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="76904-515">Попытка 2 — задержка 2 с</span><span class="sxs-lookup"><span data-stu-id="76904-515">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="76904-516">Попытка 3 — задержка 6 с</span><span class="sxs-lookup"><span data-stu-id="76904-516">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="76904-517">Попытка 4 — задержка 14 с</span><span class="sxs-lookup"><span data-stu-id="76904-517">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="76904-518">Попытка 5 — задержка 30 с</span><span class="sxs-lookup"><span data-stu-id="76904-518">Attempt 5 - delay ~30 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="76904-519">Целевые значения сквозной задержки подразумевают время ожидания по умолчанию для подключения к службе.</span><span class="sxs-lookup"><span data-stu-id="76904-519">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="76904-520">При указании более длинного времени ожидания подключения величина сквозной задержки будет увеличена путем добавления этого дополнительного времени для каждой попытки повтора.</span><span class="sxs-lookup"><span data-stu-id="76904-520">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="76904-521">Примеры</span><span class="sxs-lookup"><span data-stu-id="76904-521">Examples</span></span>
<span data-ttu-id="76904-522">В этом разделе показано, как с помощью Polly обращаться к Базе данных SQL Azure с использованием набора политик повторных попыток, настроенных в классе `Policy`.</span><span class="sxs-lookup"><span data-stu-id="76904-522">This section shows how you can use Polly to access Azure SQL Database using a set of retry policies configured in the `Policy` class.</span></span>

<span data-ttu-id="76904-523">Следующий код демонстрирует метод расширения для класса `SqlCommand`, который вызывает `ExecuteAsync` с экспоненциальным увеличением задержки.</span><span class="sxs-lookup"><span data-stu-id="76904-523">The following code shows an extension method on the `SqlCommand` class that calls `ExecuteAsync` with exponential backoff.</span></span>

```csharp
public async static Task<SqlDataReader> ExecuteReaderWithRetryAsync(this SqlCommand command)
{
    GuardConnectionIsNotNull(command);

    var policy = Policy.Handle<Exception>().WaitAndRetryAsync(
        retryCount: 3, // Retry 3 times
        sleepDurationProvider: attempt => TimeSpan.FromMilliseconds(200 * Math.Pow(2, attempt - 1)), // Exponential backoff based on an initial 200ms delay.
        onRetry: (exception, attempt) => 
        {
            // Capture some info for logging/telemetry.  
            logger.LogWarn($"ExecuteReaderWithRetryAsync: Retry {attempt} due to {exception}.");
        });

    // Retry the following call according to the policy.
    await policy.ExecuteAsync<SqlDataReader>(async token =>
    {
        // This code is executed within the Policy 

        if (conn.State != System.Data.ConnectionState.Open) await conn.OpenAsync(token);
        return await command.ExecuteReaderAsync(System.Data.CommandBehavior.Default, token);

    }, cancellationToken);
}

```

<span data-ttu-id="76904-524">Этот асинхронный метод расширения можно использовать следующим образом.</span><span class="sxs-lookup"><span data-stu-id="76904-524">This asynchronous extension method can be used as follows.</span></span>

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a><span data-ttu-id="76904-525">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="76904-525">More information</span></span>
* [<span data-ttu-id="76904-526">Основы облачной службы. Уровень доступа к данным: обработка временных ошибок</span><span class="sxs-lookup"><span data-stu-id="76904-526">Cloud Service Fundamentals Data Access Layer – Transient Fault Handling</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

<span data-ttu-id="76904-527">Общие рекомендации по повышению эффективности Базы данных SQL см. в статье [Руководство по эластичности и производительности базы данных SQL Azure](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)</span><span class="sxs-lookup"><span data-stu-id="76904-527">For general guidance on getting the most from SQL Database, see [Azure SQL Database Performance and Elasticity Guide](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span></span>


## <a name="service-bus-retry-guidelines"></a><span data-ttu-id="76904-528">Руководство по использованию механизма повторов для служебной шины</span><span class="sxs-lookup"><span data-stu-id="76904-528">Service Bus retry guidelines</span></span>
<span data-ttu-id="76904-529">Служебная шина является облачной платформой обмена сообщениями, которая предоставляет слабосвязанный обмен сообщениями с улучшенным масштабированием и устойчивостью для компонентов приложения, размещенных в облаке или локально.</span><span class="sxs-lookup"><span data-stu-id="76904-529">Service Bus is a cloud messaging platform that provides loosely coupled message exchange with improved scale and resiliency for components of an application, whether hosted in the cloud or on-premises.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="76904-530">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="76904-530">Retry mechanism</span></span>
<span data-ttu-id="76904-531">Служебная шина реализует повторы с помощью реализации базового класса [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) .</span><span class="sxs-lookup"><span data-stu-id="76904-531">Service Bus implements retries using implementations of the [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) base class.</span></span> <span data-ttu-id="76904-532">Все клиенты служебной шины предоставляют свойство **RetryPolicy**, которое может быть присвоено одной из реализаций базового класса **RetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="76904-532">All of the Service Bus clients expose a **RetryPolicy** property that can be set to one of the implementations of the **RetryPolicy** base class.</span></span> <span data-ttu-id="76904-533">Встроенные реализации</span><span class="sxs-lookup"><span data-stu-id="76904-533">The built-in implementations are:</span></span>

* <span data-ttu-id="76904-534">[Класс RetryExponential](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span><span class="sxs-lookup"><span data-stu-id="76904-534">The [RetryExponential Class](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span></span> <span data-ttu-id="76904-535">Этот класс предоставляет свойства, управляющие интервалом отсрочки, числом попыток и свойством **TerminationTimeBuffer** , используемым для ограничения общего времени завершения операции.</span><span class="sxs-lookup"><span data-stu-id="76904-535">This exposes properties that control the back-off interval, the retry count, and the **TerminationTimeBuffer** property that is used to limit the total time for the operation to complete.</span></span>
* <span data-ttu-id="76904-536">[Класс NoRetry](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span><span class="sxs-lookup"><span data-stu-id="76904-536">The [NoRetry Class](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span></span> <span data-ttu-id="76904-537">Используется, когда повторы на уровне API служебной шины не требуются, например, если повторные попытки управляются другим процессом в рамках пакета операций либо многоэтапной операции.</span><span class="sxs-lookup"><span data-stu-id="76904-537">This is used when retries at the Service Bus API level are not required, such as when retries are managed by another process as part of a batch or multiple step operation.</span></span>

<span data-ttu-id="76904-538">Действия служебной шины могут вызвать ряд исключений, как указано в документе [Приложение: исключения при передаче сообщений](http://msdn.microsoft.com/library/hh418082.aspx).</span><span class="sxs-lookup"><span data-stu-id="76904-538">Service Bus actions can return a range of exceptions, as listed in [Appendix: Messaging Exceptions](http://msdn.microsoft.com/library/hh418082.aspx).</span></span> <span data-ttu-id="76904-539">Список предоставляет сведения о том, какие из исключений указывают, что повторение операции является целесообразным.</span><span class="sxs-lookup"><span data-stu-id="76904-539">The list provides information about which if these indicate that retrying the operation is appropriate.</span></span> <span data-ttu-id="76904-540">Например [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) указывает, что клиенту следует подождать некоторое время и затем повторить операцию.</span><span class="sxs-lookup"><span data-stu-id="76904-540">For example, a [ServerBusyException](http://msdn.microsoft.com/library/microsoft.servicebus.messaging.serverbusyexception.aspx) indicates that the client should wait for a period of time, then retry the operation.</span></span> <span data-ttu-id="76904-541">Появление исключения **ServerBusyException** также вынуждает служебную шину переключиться в другой режим, в котором добавляется дополнительная 10-секундная задержка к вычисляемому времени задержки повтора операции.</span><span class="sxs-lookup"><span data-stu-id="76904-541">The occurrence of a **ServerBusyException** also causes Service Bus to switch to a different mode, in which an extra 10-second delay is added to the computed retry delays.</span></span> <span data-ttu-id="76904-542">Этот режим сбрасывается после короткого времени.</span><span class="sxs-lookup"><span data-stu-id="76904-542">This mode is reset after a short period.</span></span>

<span data-ttu-id="76904-543">Исключения, возвращаемые служебной шиной, предоставляют свойство **IsTransient** , указывающее, что клиент должен повторить операцию.</span><span class="sxs-lookup"><span data-stu-id="76904-543">The exceptions returned from Service Bus expose the **IsTransient** property that indicates if the client should retry the operation.</span></span> <span data-ttu-id="76904-544">Встроенная политика **RetryExponential** зависит от свойства **IsTransient** класса **MessagingException**, который является базовым классом для всех исключений служебной шины.</span><span class="sxs-lookup"><span data-stu-id="76904-544">The built-in **RetryExponential** policy relies on the **IsTransient** property in the **MessagingException** class, which is the base class for all Service Bus exceptions.</span></span> <span data-ttu-id="76904-545">Если создать пользовательские реализации базового класса **RetryPolicy**, то можно будет использовать сочетание типа исключения и свойство **IsTransient**, чтобы организовать более точное управление повторами.</span><span class="sxs-lookup"><span data-stu-id="76904-545">If you create custom implementations of the **RetryPolicy** base class you could use a combination of the exception type and the **IsTransient** property to provide more fine-grained control over retry actions.</span></span> <span data-ttu-id="76904-546">Например, можно обнаружить исключение **QuotaExceededException** и принять меры по очистке очереди, прежде чем повторить отправку сообщения в очередь.</span><span class="sxs-lookup"><span data-stu-id="76904-546">For example, you could detect a **QuotaExceededException** and take action to drain the queue before retrying sending a message to it.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="76904-547">Настройки политики</span><span class="sxs-lookup"><span data-stu-id="76904-547">Policy configuration</span></span>
<span data-ttu-id="76904-548">Политики повтора настраиваются программно и могут быть установлены в качестве политики по умолчанию для **NamespaceManager** и **MessagingFactory** или по отдельности для каждого клиента обмена сообщениями.</span><span class="sxs-lookup"><span data-stu-id="76904-548">Retry policies are set programmatically, and can be set as a default policy for a **NamespaceManager** and for a **MessagingFactory**, or individually for each messaging client.</span></span> <span data-ttu-id="76904-549">Чтобы задать политику повтора по умолчанию для сеанса обмена сообщениями, можно установить свойство **RetryPolicy** класса **NamespaceManager**.</span><span class="sxs-lookup"><span data-stu-id="76904-549">To set the default retry policy for a messaging session you set the **RetryPolicy** of the **NamespaceManager**.</span></span>

    namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                 maxBackoff: TimeSpan.FromSeconds(30),
                                                                 maxRetryCount: 3);

<span data-ttu-id="76904-550">Чтобы задать политику повторов по умолчанию для всех клиентов, созданных из фабрики обмена сообщениями, установите свойство **RetryPolicy** класса **MessagingFactory**.</span><span class="sxs-lookup"><span data-stu-id="76904-550">To set the default retry policy for all clients created from a messaging factory, you set the **RetryPolicy** of the **MessagingFactory**.</span></span>

    messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                        maxBackoff: TimeSpan.FromSeconds(30),
                                                        maxRetryCount: 3);

<span data-ttu-id="76904-551">Чтобы задать политику повтора для клиента обмена сообщениями или переопределить политику по умолчанию, задайте свойство **RetryPolicy** с помощью экземпляра класса требуемой политики:</span><span class="sxs-lookup"><span data-stu-id="76904-551">To set the retry policy for a messaging client, or to override its default policy, you set its **RetryPolicy** property using an instance of the required policy class:</span></span>

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

<span data-ttu-id="76904-552">Нельзя задать политику повтора на уровне отдельных операций.</span><span class="sxs-lookup"><span data-stu-id="76904-552">The retry policy cannot be set at the individual operation level.</span></span> <span data-ttu-id="76904-553">Она применяется для всех операций для клиента обмена сообщениями.</span><span class="sxs-lookup"><span data-stu-id="76904-553">It applies to all operations for the messaging client.</span></span>
<span data-ttu-id="76904-554">Следующая таблица показывает значения по умолчанию для встроенной политики повторов.</span><span class="sxs-lookup"><span data-stu-id="76904-554">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="76904-555">Параметр</span><span class="sxs-lookup"><span data-stu-id="76904-555">Setting</span></span> | <span data-ttu-id="76904-556">Значение по умолчанию</span><span class="sxs-lookup"><span data-stu-id="76904-556">Default value</span></span> | <span data-ttu-id="76904-557">Значение</span><span class="sxs-lookup"><span data-stu-id="76904-557">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="76904-558">Политика</span><span class="sxs-lookup"><span data-stu-id="76904-558">Policy</span></span> | <span data-ttu-id="76904-559">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="76904-559">Exponential</span></span> | <span data-ttu-id="76904-560">Экспоненциальная задержка.</span><span class="sxs-lookup"><span data-stu-id="76904-560">Exponential back-off.</span></span> |
| <span data-ttu-id="76904-561">MinimalBackoff</span><span class="sxs-lookup"><span data-stu-id="76904-561">MinimalBackoff</span></span> | <span data-ttu-id="76904-562">0</span><span class="sxs-lookup"><span data-stu-id="76904-562">0</span></span> | <span data-ttu-id="76904-563">Минимальное время задержки.</span><span class="sxs-lookup"><span data-stu-id="76904-563">Minimum back-off interval.</span></span> <span data-ttu-id="76904-564">Добавляется ко всем интервалам повтора, вычисленным на основе значения deltaBackoff.</span><span class="sxs-lookup"><span data-stu-id="76904-564">This is added to the retry interval computed from deltaBackoff.</span></span> |
| <span data-ttu-id="76904-565">MaximumBackoff</span><span class="sxs-lookup"><span data-stu-id="76904-565">MaximumBackoff</span></span> | <span data-ttu-id="76904-566">30 секунд</span><span class="sxs-lookup"><span data-stu-id="76904-566">30 seconds</span></span> | <span data-ttu-id="76904-567">Максимальное время задержки.</span><span class="sxs-lookup"><span data-stu-id="76904-567">Maximum back-off interval.</span></span> <span data-ttu-id="76904-568">MaxBackoff используется, если расчетный интервал повторных попыток превышает значение MaxBackoff.</span><span class="sxs-lookup"><span data-stu-id="76904-568">MaximumBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> |
| <span data-ttu-id="76904-569">DeltaBackoff</span><span class="sxs-lookup"><span data-stu-id="76904-569">DeltaBackoff</span></span> | <span data-ttu-id="76904-570">3 секунды</span><span class="sxs-lookup"><span data-stu-id="76904-570">3 seconds</span></span> | <span data-ttu-id="76904-571">Интервал отсрочки между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="76904-571">Back-off interval between retries.</span></span> <span data-ttu-id="76904-572">Величина, кратная timespan, будет использоваться для последующих попыток.</span><span class="sxs-lookup"><span data-stu-id="76904-572">Multiples of this timespan will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="76904-573">TimeBuffer</span><span class="sxs-lookup"><span data-stu-id="76904-573">TimeBuffer</span></span> | <span data-ttu-id="76904-574">5 с</span><span class="sxs-lookup"><span data-stu-id="76904-574">5 seconds</span></span> | <span data-ttu-id="76904-575">Буфер времени завершения, связанный с повторной попыткой.</span><span class="sxs-lookup"><span data-stu-id="76904-575">The termination time buffer associated with the retry.</span></span> <span data-ttu-id="76904-576">Повторные попытки будут прерваны, если оставшееся время меньше значения TimeBuffer.</span><span class="sxs-lookup"><span data-stu-id="76904-576">Retry attempts will be abandoned if the remaining time is less than TimeBuffer.</span></span> |
| <span data-ttu-id="76904-577">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="76904-577">MaxRetryCount</span></span> | <span data-ttu-id="76904-578">10</span><span class="sxs-lookup"><span data-stu-id="76904-578">10</span></span> | <span data-ttu-id="76904-579">Максимальное число попыток.</span><span class="sxs-lookup"><span data-stu-id="76904-579">The maximum number of retries.</span></span> |
| <span data-ttu-id="76904-580">ServerBusyBaseSleepTime</span><span class="sxs-lookup"><span data-stu-id="76904-580">ServerBusyBaseSleepTime</span></span> | <span data-ttu-id="76904-581">10 с</span><span class="sxs-lookup"><span data-stu-id="76904-581">10 seconds</span></span> | <span data-ttu-id="76904-582">Если последнее исключение было **ServerBusyException**, это значение будет добавляться к интервалу вычисляемых значений повторных попыток.</span><span class="sxs-lookup"><span data-stu-id="76904-582">If the last exception encountered was **ServerBusyException**, this value will be added to the computed retry interval.</span></span> <span data-ttu-id="76904-583">Это значение не может быть изменено.</span><span class="sxs-lookup"><span data-stu-id="76904-583">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="76904-584">Руководство по использованию механизма повторов</span><span class="sxs-lookup"><span data-stu-id="76904-584">Retry usage guidance</span></span>
<span data-ttu-id="76904-585">При использовании служебной шины придерживайтесь следующих рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="76904-585">Consider the following guidelines when using Service Bus:</span></span>

* <span data-ttu-id="76904-586">При использовании встроенной реализации **RetryExponential** не реализуйте операции резервирования, поскольку политика реагирует на исключения типа «Сервер занят» и автоматически переключается в режим повтора.</span><span class="sxs-lookup"><span data-stu-id="76904-586">When using the built-in **RetryExponential** implementation, do not implement a fallback operation as the policy reacts to Server Busy exceptions and automatically switches to an appropriate retry mode.</span></span>
* <span data-ttu-id="76904-587">Server Bus поддерживает функцию, называемую Сопряженные пространства имен, которая реализует автоматическое переключение на очередь резервного копирования в отдельном пространстве имен, если очередь в первичном пространстве имен дает сбой.</span><span class="sxs-lookup"><span data-stu-id="76904-587">Service Bus supports a feature called Paired Namespaces, which implements automatic failover to a backup queue in a separate namespace if the queue in the primary namespace fails.</span></span> <span data-ttu-id="76904-588">Сообщения из вторичной очереди могут отправляться обратно в основную очередь после ее восстановления.</span><span class="sxs-lookup"><span data-stu-id="76904-588">Messages from the secondary queue can be sent back to the primary queue when it recovers.</span></span> <span data-ttu-id="76904-589">Эта функция позволяет справиться с временными сбоями.</span><span class="sxs-lookup"><span data-stu-id="76904-589">This feature helps to address transient failures.</span></span> <span data-ttu-id="76904-590">Для получения дополнительных сведений обратитесь к разделу [Асинхронные шаблоны обмена сообщениями и высокий уровень доступности](http://msdn.microsoft.com/library/azure/dn292562.aspx).</span><span class="sxs-lookup"><span data-stu-id="76904-590">For more information, see [Asynchronous Messaging Patterns and High Availability](http://msdn.microsoft.com/library/azure/dn292562.aspx).</span></span>

<span data-ttu-id="76904-591">Начните с использования следующих параметров для операции повтора.</span><span class="sxs-lookup"><span data-stu-id="76904-591">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="76904-592">Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.</span><span class="sxs-lookup"><span data-stu-id="76904-592">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="76904-593">Context</span><span class="sxs-lookup"><span data-stu-id="76904-593">Context</span></span> | <span data-ttu-id="76904-594">Пример максимального значения задержки</span><span class="sxs-lookup"><span data-stu-id="76904-594">Example maximum latency</span></span> | <span data-ttu-id="76904-595">Политика повтора</span><span class="sxs-lookup"><span data-stu-id="76904-595">Retry policy</span></span> | <span data-ttu-id="76904-596">Параметры</span><span class="sxs-lookup"><span data-stu-id="76904-596">Settings</span></span> | <span data-ttu-id="76904-597">Принцип работы</span><span class="sxs-lookup"><span data-stu-id="76904-597">How it works</span></span> |
|---------|---------|---------|---------|---------|
| <span data-ttu-id="76904-598">Интерактивный, пользовательский интерфейс или передний план</span><span class="sxs-lookup"><span data-stu-id="76904-598">Interactive, UI, or foreground</span></span> | <span data-ttu-id="76904-599">2 с\*</span><span class="sxs-lookup"><span data-stu-id="76904-599">2 seconds\*</span></span>  | <span data-ttu-id="76904-600">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="76904-600">Exponential</span></span> | <span data-ttu-id="76904-601">MinimumBackoff = 0</span><span class="sxs-lookup"><span data-stu-id="76904-601">MinimumBackoff = 0</span></span> <br/> <span data-ttu-id="76904-602">MaximumBackoff = 30 с</span><span class="sxs-lookup"><span data-stu-id="76904-602">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="76904-603">DeltaBackoff = 300 мс</span><span class="sxs-lookup"><span data-stu-id="76904-603">DeltaBackoff = 300 msec.</span></span> <br/> <span data-ttu-id="76904-604">TimeBuffer = 300 мс</span><span class="sxs-lookup"><span data-stu-id="76904-604">TimeBuffer = 300 msec.</span></span> <br/> <span data-ttu-id="76904-605">MaxRetryCount = 2</span><span class="sxs-lookup"><span data-stu-id="76904-605">MaxRetryCount = 2</span></span> | <span data-ttu-id="76904-606">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="76904-606">Attempt 1: Delay 0 sec.</span></span> <br/> <span data-ttu-id="76904-607">Попытка 2 — задержка ~ 300 мс</span><span class="sxs-lookup"><span data-stu-id="76904-607">Attempt 2: Delay ~300 msec.</span></span> <br/> <span data-ttu-id="76904-608">Попытка 3 — задержка ~ 900 мс</span><span class="sxs-lookup"><span data-stu-id="76904-608">Attempt 3: Delay ~900 msec.</span></span> |
| <span data-ttu-id="76904-609">Фоновый или пакетный</span><span class="sxs-lookup"><span data-stu-id="76904-609">Background or batch</span></span> | <span data-ttu-id="76904-610">30 секунд</span><span class="sxs-lookup"><span data-stu-id="76904-610">30 seconds</span></span> | <span data-ttu-id="76904-611">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="76904-611">Exponential</span></span> | <span data-ttu-id="76904-612">MinimumBackoff = 1</span><span class="sxs-lookup"><span data-stu-id="76904-612">MinimumBackoff = 1</span></span> <br/> <span data-ttu-id="76904-613">MaximumBackoff = 30 с</span><span class="sxs-lookup"><span data-stu-id="76904-613">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="76904-614">DeltaBackoff = 1,75 с</span><span class="sxs-lookup"><span data-stu-id="76904-614">DeltaBackoff = 1.75 sec.</span></span> <br/> <span data-ttu-id="76904-615">TimeBuffer = 5 с</span><span class="sxs-lookup"><span data-stu-id="76904-615">TimeBuffer = 5 sec.</span></span> <br/> <span data-ttu-id="76904-616">MaxRetryCount = 3</span><span class="sxs-lookup"><span data-stu-id="76904-616">MaxRetryCount = 3</span></span> | <span data-ttu-id="76904-617">Попытка 1 — задержка ~ 1 с</span><span class="sxs-lookup"><span data-stu-id="76904-617">Attempt 1: Delay ~1 sec.</span></span> <br/> <span data-ttu-id="76904-618">Попытка 2 — задержка ~ 3 с</span><span class="sxs-lookup"><span data-stu-id="76904-618">Attempt 2: Delay ~3 sec.</span></span> <br/> <span data-ttu-id="76904-619">Попытка 3 — задержка ~ 6 мс</span><span class="sxs-lookup"><span data-stu-id="76904-619">Attempt 3: Delay ~6 msec.</span></span> <br/> <span data-ttu-id="76904-620">Попытка 4 — задержка ~ 13 мс</span><span class="sxs-lookup"><span data-stu-id="76904-620">Attempt 4: Delay ~13 msec.</span></span> |

<span data-ttu-id="76904-621">\* Без значения дополнительной задержки, которая добавляется при получении ответа "Сервер занят".</span><span class="sxs-lookup"><span data-stu-id="76904-621">\* Not including additional delay that is added if a Server Busy response is received.</span></span>

### <a name="telemetry"></a><span data-ttu-id="76904-622">Телеметрия</span><span class="sxs-lookup"><span data-stu-id="76904-622">Telemetry</span></span>
<span data-ttu-id="76904-623">Служебная шина регистрирует повторные попытки как события ETW с помощью **EventSource**.</span><span class="sxs-lookup"><span data-stu-id="76904-623">Service Bus logs retries as ETW events using an **EventSource**.</span></span> <span data-ttu-id="76904-624">Необходимо присоединить службу **EventListener** к источнику события, чтобы зарегистрировать события и просмотреть в средстве просмотра производительности или записать данные о них в журнале с подходящим назначением.</span><span class="sxs-lookup"><span data-stu-id="76904-624">You must attach an **EventListener** to the event source to capture the events and view them in Performance Viewer, or write them to a suitable destination log.</span></span> <span data-ttu-id="76904-625">Чтобы это сделать, можно использовать [Блок приложения семантического ведения журнала](http://msdn.microsoft.com/library/dn775006.aspx) .</span><span class="sxs-lookup"><span data-stu-id="76904-625">You could use the [Semantic Logging Application Block](http://msdn.microsoft.com/library/dn775006.aspx) to do this.</span></span> <span data-ttu-id="76904-626">События повтора операций отображаются в следующей форме:</span><span class="sxs-lookup"><span data-stu-id="76904-626">The retry events are of the following form:</span></span>

```text
Microsoft-ServiceBus-Client/RetryPolicyIteration
ThreadID="14,500"
FormattedMessage="[TrackingId:] RetryExponential: Operation Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05 at iteration 0 is retrying after 00:00:00.1000000 sleep because of Microsoft.ServiceBus.Messaging.MessagingCommunicationException: The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3, TimeStamp:9/5/2014 10:00:13 PM."
trackingId=""
policyType="RetryExponential"
operation="Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05"
iteration="0"
iterationSleep="00:00:00.1000000"
lastExceptionType="Microsoft.ServiceBus.Messaging.MessagingCommunicationException"
exceptionMessage="The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3,TimeStamp:9/5/2014 10:00:13 PM"
```

### <a name="examples"></a><span data-ttu-id="76904-627">Примеры</span><span class="sxs-lookup"><span data-stu-id="76904-627">Examples</span></span>
<span data-ttu-id="76904-628">В следующем примере кода показано, как задать политику повтора для</span><span class="sxs-lookup"><span data-stu-id="76904-628">The following code example shows how to set the retry policy for:</span></span>

* <span data-ttu-id="76904-629">Диспетчера пространства имен.</span><span class="sxs-lookup"><span data-stu-id="76904-629">A namespace manager.</span></span> <span data-ttu-id="76904-630">Политика применяется ко всем операциям диспетчера и не может быть переопределена для отдельных операций.</span><span class="sxs-lookup"><span data-stu-id="76904-630">The policy applies to all operations on that manager, and cannot be overridden for individual operations.</span></span>
* <span data-ttu-id="76904-631">Фабрики сообщений.</span><span class="sxs-lookup"><span data-stu-id="76904-631">A messaging factory.</span></span> <span data-ttu-id="76904-632">Политика применяется ко всем клиентам, созданным из этой фабрики, и не может быть переопределена при создании отдельных клиентов.</span><span class="sxs-lookup"><span data-stu-id="76904-632">The policy applies to all clients created from that factory, and cannot be overridden when creating individual clients.</span></span>
* <span data-ttu-id="76904-633">Отдельных клиентов обмена сообщениями.</span><span class="sxs-lookup"><span data-stu-id="76904-633">An individual messaging client.</span></span> <span data-ttu-id="76904-634">После создания клиента можно задать политику повтора для этого клиента.</span><span class="sxs-lookup"><span data-stu-id="76904-634">After a client has been created, you can set the retry policy for that client.</span></span> <span data-ttu-id="76904-635">Политика применяется ко всем операциям на этом клиенте.</span><span class="sxs-lookup"><span data-stu-id="76904-635">The policy applies to all operations on that client.</span></span>

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Messaging;

namespace RetryCodeSamples
{
    class ServiceBusCodeSamples
    {
        private const string connectionString =
            @"Endpoint=sb://[my-namespace].servicebus.windows.net/;
                SharedAccessKeyName=RootManageSharedAccessKey;
                SharedAccessKey=C99..........Mk=";

        public async static Task Samples()
        {
            const string QueueName = "TestQueue";

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.Http;

            var namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);

            // The namespace manager will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for all operations on the namespace manager.
                namespaceManager.Settings.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                if (!await namespaceManager.QueueExistsAsync(QueueName))
                {
                    await namespaceManager.CreateQueueAsync(QueueName);
                }
            }


            var messagingFactory = MessagingFactory.Create(
                namespaceManager.Address, namespaceManager.Settings.TokenProvider);
            // The messaging factory will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for clients created from it.
                messagingFactory.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await messagingFactory.AcceptMessageSessionAsync();
            }


            {
                var client = messagingFactory.CreateQueueClient(QueueName);
                // The client inherits the policy from the factory that created it.


                // Set different values for the retry policy on the client.
                client.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0.1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await client.AcceptMessageSessionAsync();
            }
        }
    }
}
```

### <a name="more-information"></a><span data-ttu-id="76904-636">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="76904-636">More information</span></span>
* [<span data-ttu-id="76904-637">Шаблоны асинхронного обмена сообщениями и высокий уровень доступности</span><span class="sxs-lookup"><span data-stu-id="76904-637">Asynchronous Messaging Patterns and High Availability</span></span>](http://msdn.microsoft.com/library/azure/dn292562.aspx)

## <a name="azure-redis-cache-retry-guidelines"></a><span data-ttu-id="76904-638">Руководство по использованию механизма повторов для кэша Redis для Azure</span><span class="sxs-lookup"><span data-stu-id="76904-638">Azure Redis Cache retry guidelines</span></span>
<span data-ttu-id="76904-639">Кэш Redis для Azure является средством быстрого доступа к данным и службой кэша с низкой задержкой, созданной на основе кэша с открытым исходным кодом Redis.</span><span class="sxs-lookup"><span data-stu-id="76904-639">Azure Redis Cache is a fast data access and low latency cache service based on the popular open source Redis Cache.</span></span> <span data-ttu-id="76904-640">Он является безопасным, управляется Майкрософт и доступен из любого приложения в Azure.</span><span class="sxs-lookup"><span data-stu-id="76904-640">It is secure, managed by Microsoft, and is accessible from any application in Azure.</span></span>

<span data-ttu-id="76904-641">Рекомендации в этом разделе основаны на использовании клиента StackExchange.Redis для доступа к кэшу.</span><span class="sxs-lookup"><span data-stu-id="76904-641">The guidance in this section is based on using the StackExchange.Redis client to access the cache.</span></span> <span data-ttu-id="76904-642">Список других подходящих клиентов можно найти на [веб-сайте Redis](http://redis.io/clients), и они могут применять различные механизмы.</span><span class="sxs-lookup"><span data-stu-id="76904-642">A list of other suitable clients can be found on the [Redis website](http://redis.io/clients), and these may have different retry mechanisms.</span></span>

<span data-ttu-id="76904-643">Обратите внимание, что клиент StackExchange.Redis использует мультиплексирование через одно подключение.</span><span class="sxs-lookup"><span data-stu-id="76904-643">Note that the StackExchange.Redis client uses multiplexing through a single connection.</span></span> <span data-ttu-id="76904-644">Рекомендуется создавать экземпляр клиента при запуске приложения и использовать этот экземпляр для всех операций в кэше.</span><span class="sxs-lookup"><span data-stu-id="76904-644">The recommended usage is to create an instance of the client at application startup and use this instance for all operations against the cache.</span></span> <span data-ttu-id="76904-645">По этой причине подключение к кэшу происходит только один раз, и поэтому все инструкции в этом разделе относятся к политике повтора для этого исходного соединения, а не для каждой операции, которая обращается к кэшу.</span><span class="sxs-lookup"><span data-stu-id="76904-645">For this reason, the connection to the cache is made only once, and so all of the guidance in this section is related to the retry policy for this initial connection—and not for each operation that accesses the cache.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="76904-646">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="76904-646">Retry mechanism</span></span>
<span data-ttu-id="76904-647">Клиент StackExchange.Redis использует класс диспетчера подключений, для настройки которого доступен ряд параметров:</span><span class="sxs-lookup"><span data-stu-id="76904-647">The StackExchange.Redis client uses a connection manager class that is configured through a set of options, incuding:</span></span>

- <span data-ttu-id="76904-648">**ConnectRetry**.</span><span class="sxs-lookup"><span data-stu-id="76904-648">**ConnectRetry**.</span></span> <span data-ttu-id="76904-649">Число повторных попыток при сбое подключения к кэшу.</span><span class="sxs-lookup"><span data-stu-id="76904-649">The number of times a failed connection to the cache will be retried.</span></span>
- <span data-ttu-id="76904-650">**ReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="76904-650">**ReconnectRetryPolicy**.</span></span> <span data-ttu-id="76904-651">Используемая стратегия повторных попыток.</span><span class="sxs-lookup"><span data-stu-id="76904-651">The retry strategy to use.</span></span>
- <span data-ttu-id="76904-652">**ConnectTimeout**.</span><span class="sxs-lookup"><span data-stu-id="76904-652">**ConnectTimeout**.</span></span> <span data-ttu-id="76904-653">Максимальное время ожидания в миллисекундах.</span><span class="sxs-lookup"><span data-stu-id="76904-653">The maximum waiting time in milliseconds.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="76904-654">Настройки политики</span><span class="sxs-lookup"><span data-stu-id="76904-654">Policy configuration</span></span>
<span data-ttu-id="76904-655">Политики повтора настраиваются программным способом с помощью параметров клиента перед подключением к кэшу.</span><span class="sxs-lookup"><span data-stu-id="76904-655">Retry policies are configured programmatically by setting the options for the client before connecting to the cache.</span></span> <span data-ttu-id="76904-656">Для этого нужно создать экземпляр класса **ConfigurationOptions**, заполнить его свойства и передать ему метод **Connect**.</span><span class="sxs-lookup"><span data-stu-id="76904-656">This can be done by creating an instance of the **ConfigurationOptions** class, populating its properties, and passing it to the **Connect** method.</span></span>

<span data-ttu-id="76904-657">Встроенные классы поддерживают линейные (постоянные) паузы и экспоненциальное увеличение задержки с псевдослучайными интервалами.</span><span class="sxs-lookup"><span data-stu-id="76904-657">The built-in classes support linear (constant) delay and exponential backoff with randomized retry intervals.</span></span> <span data-ttu-id="76904-658">Также вы можете создать пользовательскую политику повторных попыток, реализовав интерфейс **IReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="76904-658">You can also create a custom retry policy by implementing the **IReconnectRetryPolicy** interface.</span></span>

<span data-ttu-id="76904-659">Код следующего примера настраивает стратегию повторных попыток с экспоненциальным увеличением задержки.</span><span class="sxs-lookup"><span data-stu-id="76904-659">The following example configures a retry strategy using exponential backoff.</span></span>

```csharp
var deltaBackOffInMilliseconds = TimeSpan.FromSeconds(5).Milliseconds;
var maxDeltaBackOffInMilliseconds = TimeSpan.FromSeconds(20).Milliseconds;
var options = new ConfigurationOptions
{
    EndPoints = {"localhost"},
    ConnectRetry = 3,
    ReconnectRetryPolicy = new ExponentialRetry(deltaBackOffInMilliseconds, maxDeltaBackOffInMilliseconds),
    ConnectTimeout = 2000
};
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="76904-660">Кроме того, можно указать параметры в виде строки и передать эти данные методу **Connect** .</span><span class="sxs-lookup"><span data-stu-id="76904-660">Alternatively, you can specify the options as a string, and pass this to the **Connect** method.</span></span> <span data-ttu-id="76904-661">Обратите внимание, что свойство **ReconnectRetryPolicy** нельзя задать таким способом, а можно только с помощью кода.</span><span class="sxs-lookup"><span data-stu-id="76904-661">Note that the **ReconnectRetryPolicy** property cannot be set this way, only through code.</span></span>

```csharp
    var options = "localhost,connectRetry=3,connectTimeout=2000";
    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="76904-662">Также вы можете указать параметры непосредственно при подключении к кэшу.</span><span class="sxs-lookup"><span data-stu-id="76904-662">You can also specify options directly when you connect to the cache.</span></span>

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

<span data-ttu-id="76904-663">Дополнительные сведения см. в [документации по StackExchange.Redis](https://stackexchange.github.io/StackExchange.Redis/Configuration).</span><span class="sxs-lookup"><span data-stu-id="76904-663">For more information, see [Stack Exchange Redis Configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration) in the StackExchange.Redis documentation.</span></span>

<span data-ttu-id="76904-664">Следующая таблица показывает значения по умолчанию для встроенной политики повторов.</span><span class="sxs-lookup"><span data-stu-id="76904-664">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="76904-665">**Контекст**</span><span class="sxs-lookup"><span data-stu-id="76904-665">**Context**</span></span> | <span data-ttu-id="76904-666">**Параметр**</span><span class="sxs-lookup"><span data-stu-id="76904-666">**Setting**</span></span> | <span data-ttu-id="76904-667">**Значение по умолчанию**</span><span class="sxs-lookup"><span data-stu-id="76904-667">**Default value**</span></span><br /><span data-ttu-id="76904-668">(версия 1.2.2)</span><span class="sxs-lookup"><span data-stu-id="76904-668">(v 1.2.2)</span></span> | <span data-ttu-id="76904-669">**Значение**</span><span class="sxs-lookup"><span data-stu-id="76904-669">**Meaning**</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="76904-670">Параметры конфигурации</span><span class="sxs-lookup"><span data-stu-id="76904-670">ConfigurationOptions</span></span> |<span data-ttu-id="76904-671">ConnectRetry</span><span class="sxs-lookup"><span data-stu-id="76904-671">ConnectRetry</span></span><br /><br /><span data-ttu-id="76904-672">ConnectTimeout</span><span class="sxs-lookup"><span data-stu-id="76904-672">ConnectTimeout</span></span><br /><br /><span data-ttu-id="76904-673">SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="76904-673">SyncTimeout</span></span><br /><br /><span data-ttu-id="76904-674">ReconnectRetryPolicy</span><span class="sxs-lookup"><span data-stu-id="76904-674">ReconnectRetryPolicy</span></span> |<span data-ttu-id="76904-675">3</span><span class="sxs-lookup"><span data-stu-id="76904-675">3</span></span><br /><br /><span data-ttu-id="76904-676">Максимально 5000 мс плюс SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="76904-676">Maximum 5000 ms plus SyncTimeout</span></span><br /><span data-ttu-id="76904-677">1000</span><span class="sxs-lookup"><span data-stu-id="76904-677">1000</span></span><br /><br /><span data-ttu-id="76904-678">LinearRetry 5000 мс</span><span class="sxs-lookup"><span data-stu-id="76904-678">LinearRetry 5000 ms</span></span> |<span data-ttu-id="76904-679">Количество попыток повторов подключения во время начального подключения.</span><span class="sxs-lookup"><span data-stu-id="76904-679">The number of times to repeat connect attempts during the initial connection operation.</span></span><br /><span data-ttu-id="76904-680">Время ожидания (мс) для операций подключения.</span><span class="sxs-lookup"><span data-stu-id="76904-680">Timeout (ms) for connect operations.</span></span> <span data-ttu-id="76904-681">Без задержки между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="76904-681">Not a delay between retry attempts.</span></span><br /><span data-ttu-id="76904-682">Время (мс) для синхронных операций.</span><span class="sxs-lookup"><span data-stu-id="76904-682">Time (ms) to allow for synchronous operations.</span></span><br /><br /><span data-ttu-id="76904-683">Повторы через каждые 5000 мс.</span><span class="sxs-lookup"><span data-stu-id="76904-683">Retry every 5000 ms.</span></span>|

> [!NOTE]
> <span data-ttu-id="76904-684">Для синхронных операций можно добавить `SyncTimeout` для определения совокупного времени задержки, но слишком низкое значение этого параметра приведет к чрезмерному времени ожидания.</span><span class="sxs-lookup"><span data-stu-id="76904-684">For synchronous operations, `SyncTimeout` can add to the end-to-end latency, but setting the value too low can cause excessive timeouts.</span></span> <span data-ttu-id="76904-685">См. статью [Способы устранения проблем с кэшем Redis для Azure][redis-cache-troubleshoot].</span><span class="sxs-lookup"><span data-stu-id="76904-685">See [How to troubleshoot Azure Redis Cache][redis-cache-troubleshoot].</span></span> <span data-ttu-id="76904-686">Обычно следует избегать синхронных операций и использовать асинхронные везде, где это возможно.</span><span class="sxs-lookup"><span data-stu-id="76904-686">In general, avoid using synchronous operations, and use asynchronous operations instead.</span></span> <span data-ttu-id="76904-687">Дополнительные сведения см. в статье [Конвейеры и мультиплексоры](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span><span class="sxs-lookup"><span data-stu-id="76904-687">For more information see [Pipelines and Multiplexers](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span></span>
>
>

### <a name="retry-usage-guidance"></a><span data-ttu-id="76904-688">Руководство по использованию механизма повторов</span><span class="sxs-lookup"><span data-stu-id="76904-688">Retry usage guidance</span></span>
<span data-ttu-id="76904-689">При использовании кэша Redis для Azure, придерживайтесь следующих рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="76904-689">Consider the following guidelines when using Azure Redis Cache:</span></span>

* <span data-ttu-id="76904-690">Клиент StackExchange Redis управляет собственными операциями повтора, но только при установлении соединения с кэшем при первом запуске приложения.</span><span class="sxs-lookup"><span data-stu-id="76904-690">The StackExchange Redis client manages its own retries, but only when establishing a connection to the cache when the application first starts.</span></span> <span data-ttu-id="76904-691">Вы можете настроить время ожидания подключения, число повторных попыток подключения и паузы между ними, но эта политика повторов не применяется к операциям с кэшем.</span><span class="sxs-lookup"><span data-stu-id="76904-691">You can configure the connection timeout, the number of retry attempts, and the time between retries to establish this connection, but the retry policy does not apply to operations against the cache.</span></span>
* <span data-ttu-id="76904-692">Вместо использования большого числа повторных попыток рассмотрите возможность возврата к исходному источнику данных.</span><span class="sxs-lookup"><span data-stu-id="76904-692">Instead of using a large number of retry attempts, consider falling back by accessing the original data source instead.</span></span>

### <a name="telemetry"></a><span data-ttu-id="76904-693">Телеметрия</span><span class="sxs-lookup"><span data-stu-id="76904-693">Telemetry</span></span>
<span data-ttu-id="76904-694">Можно собирать сведения о подключении (но не о других операциях) с помощью **TextWriter**.</span><span class="sxs-lookup"><span data-stu-id="76904-694">You can collect information about connections (but not other operations) using a **TextWriter**.</span></span>

```csharp
var writer = new StringWriter();
...
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="76904-695">Ниже показан пример генерируемых выходных данных.</span><span class="sxs-lookup"><span data-stu-id="76904-695">An example of the output this generates is shown below.</span></span>

```text
localhost:6379,connectTimeout=2000,connectRetry=3
1 unique nodes specified
Requesting tie-break from localhost:6379 > __Booksleeve_TieBreak...
Allowing endpoints 00:00:02 to respond...
localhost:6379 faulted: SocketFailure on PING
localhost:6379 failed to nominate (Faulted)
> UnableToResolvePhysicalConnection on GET
No masters detected
localhost:6379: Standalone v2.0.0, master; keep-alive: 00:01:00; int: Connecting; sub: Connecting; not in use: DidNotRespond
localhost:6379: int ops=0, qu=0, qs=0, qc=1, wr=0, sync=1, socks=2; sub ops=0, qu=0, qs=0, qc=0, wr=0, socks=2
Circular op-count snapshot; int: 0 (0.00 ops/s; spans 10s); sub: 0 (0.00 ops/s; spans 10s)
Sync timeouts: 0; fire and forget: 0; last heartbeat: -1s ago
resetting failing connections to retry...
retrying; attempts left: 2...
...
```

### <a name="examples"></a><span data-ttu-id="76904-696">Примеры</span><span class="sxs-lookup"><span data-stu-id="76904-696">Examples</span></span>
<span data-ttu-id="76904-697">Следующий пример кода настраивает постоянную (линейную) задержку между повторными попытками при инициализации клиента StackExchange.Redis.</span><span class="sxs-lookup"><span data-stu-id="76904-697">The following code example configures a constant (linear) delay between retries when initializing the StackExchange.Redis client.</span></span> <span data-ttu-id="76904-698">В этом примере показано, как настроить конфигурацию с помощью экземпляра **ConfigurationOptions**.</span><span class="sxs-lookup"><span data-stu-id="76904-698">This example shows how to set the configuration using a **ConfigurationOptions** instance.</span></span>

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();

            {
                try
                {
                    var retryTimeInMilliseconds = TimeSpan.FromSeconds(4).Milliseconds; // delay between retries
                    
                    // Using object-based configuration.
                    var options = new ConfigurationOptions
                                        {
                                            EndPoints = { "localhost" },
                                            ConnectRetry = 3,
                                            ReconnectRetryPolicy = new LinearRetry(retryTimeInMilliseconds)
                                        };
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

<span data-ttu-id="76904-699">В следующем примере показано, как настроить конфигурацию, указав параметры в строковом формате.</span><span class="sxs-lookup"><span data-stu-id="76904-699">The next example sets the configuration by specifying the options as a string.</span></span> <span data-ttu-id="76904-700">Время ожидания подключения обозначает максимальный период времени для ожидания соединения с кэшем, но не задержку между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="76904-700">The connection timeout is the maximum period of time to wait for a connection to the cache, not the delay between retry attempts.</span></span> <span data-ttu-id="76904-701">Обратите внимание, что свойство **ReconnectRetryPolicy** можно задавать только в коде.</span><span class="sxs-lookup"><span data-stu-id="76904-701">Note that the **ReconnectRetryPolicy** property can only be set by code.</span></span>

```csharp
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();

            {
                try
                {
                    // Using string-based configuration.
                    var options = "localhost,connectRetry=3,connectTimeout=2000";
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

<span data-ttu-id="76904-702">Дополнительные примеры см. в разделе [Конфигурация](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) на веб-сайте проекта.</span><span class="sxs-lookup"><span data-stu-id="76904-702">For more examples, see [Configuration](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) on the project website.</span></span>

### <a name="more-information"></a><span data-ttu-id="76904-703">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="76904-703">More information</span></span>
* [<span data-ttu-id="76904-704">Веб-сайт Redis</span><span class="sxs-lookup"><span data-stu-id="76904-704">Redis website</span></span>](http://redis.io/)

## <a name="cosmos-db-retry-guidelines"></a><span data-ttu-id="76904-705">Рекомендации по использованию механизма повторных попыток для Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="76904-705">Cosmos DB retry guidelines</span></span>

<span data-ttu-id="76904-706">Cosmos DB — это полностью управляемая база данных, которая поддерживает несколько моделей и данные JSON без схемы данных.</span><span class="sxs-lookup"><span data-stu-id="76904-706">Cosmos DB is a fully-managed multi-model database that supports schema-less JSON data.</span></span> <span data-ttu-id="76904-707">Она обеспечивает настраиваемую и надежную производительность, собственную обработку транзакций на основе JavaScript и разработана для облачной службы с гибким масштабированием.</span><span class="sxs-lookup"><span data-stu-id="76904-707">It offers configurable and reliable performance, native JavaScript transactional processing, and is built for the cloud with elastic scale.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="76904-708">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="76904-708">Retry mechanism</span></span>
<span data-ttu-id="76904-709">Класс `DocumentClient` автоматически осуществляет новую попытку после сбоя.</span><span class="sxs-lookup"><span data-stu-id="76904-709">The `DocumentClient` class automatically retries failed attempts.</span></span> <span data-ttu-id="76904-710">Чтобы задать количество повторных попыток и максимальное время ожидания, настройте свойство [ConnectionPolicy.RetryOptions].</span><span class="sxs-lookup"><span data-stu-id="76904-710">To set the number of retries and the maximum wait time, configure [ConnectionPolicy.RetryOptions].</span></span> <span data-ttu-id="76904-711">Исключения клиента находятся за пределами действия политики повтора или не являются временными ошибками.</span><span class="sxs-lookup"><span data-stu-id="76904-711">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>

<span data-ttu-id="76904-712">Если Cosmos DB выполняет для клиента регулирование, возвращается ошибка HTTP 429.</span><span class="sxs-lookup"><span data-stu-id="76904-712">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="76904-713">Проверьте код состояния в `DocumentClientException`.</span><span class="sxs-lookup"><span data-stu-id="76904-713">Check the status code in the `DocumentClientException`.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="76904-714">Настройки политики</span><span class="sxs-lookup"><span data-stu-id="76904-714">Policy configuration</span></span>
<span data-ttu-id="76904-715">В следующей таблице показаны значения по умолчанию для класса `RetryOptions`.</span><span class="sxs-lookup"><span data-stu-id="76904-715">The following table shows the default settings for the `RetryOptions` class.</span></span>

| <span data-ttu-id="76904-716">Параметр</span><span class="sxs-lookup"><span data-stu-id="76904-716">Setting</span></span> | <span data-ttu-id="76904-717">Значение по умолчанию</span><span class="sxs-lookup"><span data-stu-id="76904-717">Default value</span></span> | <span data-ttu-id="76904-718">ОПИСАНИЕ</span><span class="sxs-lookup"><span data-stu-id="76904-718">Description</span></span> |
| --- | --- | --- |
| <span data-ttu-id="76904-719">MaxRetryAttemptsOnThrottledRequests</span><span class="sxs-lookup"><span data-stu-id="76904-719">MaxRetryAttemptsOnThrottledRequests</span></span> |<span data-ttu-id="76904-720">9</span><span class="sxs-lookup"><span data-stu-id="76904-720">9</span></span> |<span data-ttu-id="76904-721">Максимальное число повторных попыток, когда запрос завершается сбоем из-за ограничения частоты запросов для клиента в Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="76904-721">The maximum number of retries if the request fails because Cosmos DB applied rate limiting on the client.</span></span> |
| <span data-ttu-id="76904-722">MaxRetryWaitTimeInSeconds</span><span class="sxs-lookup"><span data-stu-id="76904-722">MaxRetryWaitTimeInSeconds</span></span> |<span data-ttu-id="76904-723">30</span><span class="sxs-lookup"><span data-stu-id="76904-723">30</span></span> |<span data-ttu-id="76904-724">Максимальное время повтора в секундах.</span><span class="sxs-lookup"><span data-stu-id="76904-724">The maximum retry time in seconds.</span></span> |

### <a name="example"></a><span data-ttu-id="76904-725">Пример</span><span class="sxs-lookup"><span data-stu-id="76904-725">Example</span></span>
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a><span data-ttu-id="76904-726">Телеметрия</span><span class="sxs-lookup"><span data-stu-id="76904-726">Telemetry</span></span>
<span data-ttu-id="76904-727">Повторные попытки регистрируются как неструктурированные сообщения трассировки в .NET **TraceSource**.</span><span class="sxs-lookup"><span data-stu-id="76904-727">Retry attempts are logged as unstructured trace messages through a .NET **TraceSource**.</span></span> <span data-ttu-id="76904-728">Необходимо настроить **TraceListener** для сбора данных о событиях и записи этих данных в журнал с подходящим назначением.</span><span class="sxs-lookup"><span data-stu-id="76904-728">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span>

<span data-ttu-id="76904-729">Например, если добавить в файл App.config следующий фрагмент кода, трассировка будет создаваться в текстовом файле в том же расположении, что и исполняемый файл:</span><span class="sxs-lookup"><span data-stu-id="76904-729">For example, if you add the following to your App.config file, traces will be generated in a text file in the same location as the executable:</span></span>

```
<configuration>
  <system.diagnostics>
    <switches>
      <add name="SourceSwitch" value="Verbose"/>
    </switches>
    <sources>
      <source name="DocDBTrace" switchName="SourceSwitch" switchType="System.Diagnostics.SourceSwitch" >
        <listeners>
          <add name="MyTextListener" type="System.Diagnostics.TextWriterTraceListener" traceOutputOptions="DateTime,ProcessId,ThreadId" initializeData="CosmosDBTrace.txt"></add>
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>
```


## <a name="azure-search-retry-guidelines"></a><span data-ttu-id="76904-730">Рекомендации по использованию механизма повторов для Поиска Azure</span><span class="sxs-lookup"><span data-stu-id="76904-730">Azure Search retry guidelines</span></span>
<span data-ttu-id="76904-731">Поиск Azure является мощным и сложным инструментом поиска для сайта или приложения с возможностями быстрой и простой настройки результатов поиска и построения информативных и точных моделей ранжирования.</span><span class="sxs-lookup"><span data-stu-id="76904-731">Azure Search can be used to add powerful and sophisticated search capabilities to a website or application, quickly and easily tune search results, and construct rich and fine-tuned ranking models.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="76904-732">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="76904-732">Retry mechanism</span></span>
<span data-ttu-id="76904-733">Поведение повтора в пакете SDK для Поиска Azure контролируется в методе `SetRetryPolicy` класса [SearchServiceClient] и [SearchIndexClient].</span><span class="sxs-lookup"><span data-stu-id="76904-733">Retry behavior in the Azure Search SDK is controlled by the `SetRetryPolicy` method on the [SearchServiceClient] and [SearchIndexClient] classes.</span></span> <span data-ttu-id="76904-734">Повтор политики по умолчанию осуществляется с экспоненциальной задержкой, если Поиск Azure возвращает ответ с кодом 5xx или 408 (истекло время ожидания запроса).</span><span class="sxs-lookup"><span data-stu-id="76904-734">The default policy retries with exponential backoff when Azure Search returns a 5xx or 408 (Request Timeout) response.</span></span>

### <a name="telemetry"></a><span data-ttu-id="76904-735">Телеметрия</span><span class="sxs-lookup"><span data-stu-id="76904-735">Telemetry</span></span>
<span data-ttu-id="76904-736">Используйте трассировку событий Windows или осуществляйте трассировку путем регистрации пользовательского поставщика трассировки.</span><span class="sxs-lookup"><span data-stu-id="76904-736">Trace with ETW or by registering a custom trace provider.</span></span> <span data-ttu-id="76904-737">Дополнительные сведения см. в [документации по AutoRest][autorest].</span><span class="sxs-lookup"><span data-stu-id="76904-737">For more information, see the [AutoRest documentation][autorest].</span></span>

## <a name="azure-active-directory-retry-guidelines"></a><span data-ttu-id="76904-738">Руководство по использованию механизма повторов для Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="76904-738">Azure Active Directory retry guidelines</span></span>
<span data-ttu-id="76904-739">Azure Active Directory (Azure AD) — это комплексное решение по управлению идентификацией и доступом к облаку, которое включает службы основных каталогов, функции расширенного управления удостоверениями и доступа к приложениям, а также средства обеспечения безопасности.</span><span class="sxs-lookup"><span data-stu-id="76904-739">Azure Active Directory (Azure AD) is a comprehensive identity and access management cloud solution that combines core directory services, advanced identity governance, security, and application access management.</span></span> <span data-ttu-id="76904-740">Azure AD также предлагает разработчикам платформу управления удостоверениями для доставки элементов контроля доступа к приложениям на основе централизованной политики и правил.</span><span class="sxs-lookup"><span data-stu-id="76904-740">Azure AD also offers developers an identity management platform to deliver access control to their applications, based on centralized policy and rules.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="76904-741">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="76904-741">Retry mechanism</span></span>
<span data-ttu-id="76904-742">Библиотека аутентификации для Active Directory (ADAL) имеет встроенный механизм повторов для службы Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="76904-742">There is a built-in retry mechanism for Azure Active Directory in the Active Directory Authentication Library (ADAL).</span></span> <span data-ttu-id="76904-743">Чтобы избежать непредвиденных блокировок, мы рекомендуем разработчикам **не** применять повторные попытки в своих библиотеках и приложениях, а использовать для этого библиотеку ADAL.</span><span class="sxs-lookup"><span data-stu-id="76904-743">To avoid unexpected lockouts, we recommend that third party libraries and application code do **not** retry failed connections, but allow ADAL to handle retries.</span></span> 

### <a name="retry-usage-guidance"></a><span data-ttu-id="76904-744">Руководство по использованию механизма повторов</span><span class="sxs-lookup"><span data-stu-id="76904-744">Retry usage guidance</span></span>
<span data-ttu-id="76904-745">При использовании Azure Active Directory придерживайтесь следующих рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="76904-745">Consider the following guidelines when using Azure Active Directory:</span></span>

* <span data-ttu-id="76904-746">Везде, где это возможно, используйте библиотеку ADAL и ее встроенный механизм повторных попыток.</span><span class="sxs-lookup"><span data-stu-id="76904-746">When possible, use the ADAL library and the built-in support for retries.</span></span>
* <span data-ttu-id="76904-747">При использовании интерфейса API REST для Azure Active Directory следует повторить операцию только в том случае, если происходит ошибка в диапазоне 5xx (500 — Внутренняя ошибка сервера, 502 — Ошибочный шлюз, 503 — Служба недоступна и 504 — Истекло время ожидания шлюза).</span><span class="sxs-lookup"><span data-stu-id="76904-747">If you are using the REST API for Azure Active Directory, you should retry the operation only if the result is an error in the 5xx range (such as 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable, and 504 Gateway Timeout).</span></span> <span data-ttu-id="76904-748">Не следует выполнять повтор при возникновении других ошибок.</span><span class="sxs-lookup"><span data-stu-id="76904-748">Do not retry for any other errors.</span></span>
* <span data-ttu-id="76904-749">В пакетных сценариях с Azure Active Directory рекомендуется использование экспоненциальных политик отсрочки.</span><span class="sxs-lookup"><span data-stu-id="76904-749">An exponential back-off policy is recommended for use in batch scenarios with Azure Active Directory.</span></span>

<span data-ttu-id="76904-750">Для начала установите следующие параметры для операций повтора.</span><span class="sxs-lookup"><span data-stu-id="76904-750">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="76904-751">Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.</span><span class="sxs-lookup"><span data-stu-id="76904-751">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="76904-752">**Контекст**</span><span class="sxs-lookup"><span data-stu-id="76904-752">**Context**</span></span> | <span data-ttu-id="76904-753">**Максимальная задержка<br />примера целевого E2E**</span><span class="sxs-lookup"><span data-stu-id="76904-753">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="76904-754">**Стратегия повторов**</span><span class="sxs-lookup"><span data-stu-id="76904-754">**Retry strategy**</span></span> | <span data-ttu-id="76904-755">**Параметры**</span><span class="sxs-lookup"><span data-stu-id="76904-755">**Settings**</span></span> | <span data-ttu-id="76904-756">**Значения**</span><span class="sxs-lookup"><span data-stu-id="76904-756">**Values**</span></span> | <span data-ttu-id="76904-757">**Принцип работы**</span><span class="sxs-lookup"><span data-stu-id="76904-757">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="76904-758">Интерактивный, пользовательский интерфейс</span><span class="sxs-lookup"><span data-stu-id="76904-758">Interactive, UI,</span></span><br /><span data-ttu-id="76904-759">или передний план</span><span class="sxs-lookup"><span data-stu-id="76904-759">or foreground</span></span> |<span data-ttu-id="76904-760">2 с</span><span class="sxs-lookup"><span data-stu-id="76904-760">2 sec</span></span> |<span data-ttu-id="76904-761">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="76904-761">FixedInterval</span></span> |<span data-ttu-id="76904-762">Число повторных попыток</span><span class="sxs-lookup"><span data-stu-id="76904-762">Retry count</span></span><br /><span data-ttu-id="76904-763">Интервал попытки</span><span class="sxs-lookup"><span data-stu-id="76904-763">Retry interval</span></span><br /><span data-ttu-id="76904-764">Первый быстрый повтор</span><span class="sxs-lookup"><span data-stu-id="76904-764">First fast retry</span></span> |<span data-ttu-id="76904-765">3</span><span class="sxs-lookup"><span data-stu-id="76904-765">3</span></span><br /><span data-ttu-id="76904-766">500 мс</span><span class="sxs-lookup"><span data-stu-id="76904-766">500 ms</span></span><br /><span data-ttu-id="76904-767">true</span><span class="sxs-lookup"><span data-stu-id="76904-767">true</span></span> |<span data-ttu-id="76904-768">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="76904-768">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="76904-769">Попытка 2 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="76904-769">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="76904-770">попытка 3 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="76904-770">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="76904-771">Фоновый</span><span class="sxs-lookup"><span data-stu-id="76904-771">Background or</span></span><br /><span data-ttu-id="76904-772">или пакетный</span><span class="sxs-lookup"><span data-stu-id="76904-772">batch</span></span> |<span data-ttu-id="76904-773">60 с</span><span class="sxs-lookup"><span data-stu-id="76904-773">60 sec</span></span> |<span data-ttu-id="76904-774">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="76904-774">ExponentialBackoff</span></span> |<span data-ttu-id="76904-775">Число повторных попыток</span><span class="sxs-lookup"><span data-stu-id="76904-775">Retry count</span></span><br /><span data-ttu-id="76904-776">Минимальная задержка</span><span class="sxs-lookup"><span data-stu-id="76904-776">Min back-off</span></span><br /><span data-ttu-id="76904-777">Максимальная задержка</span><span class="sxs-lookup"><span data-stu-id="76904-777">Max back-off</span></span><br /><span data-ttu-id="76904-778">Разностная задержка</span><span class="sxs-lookup"><span data-stu-id="76904-778">Delta back-off</span></span><br /><span data-ttu-id="76904-779">Первый быстрый повтор</span><span class="sxs-lookup"><span data-stu-id="76904-779">First fast retry</span></span> |<span data-ttu-id="76904-780">5</span><span class="sxs-lookup"><span data-stu-id="76904-780">5</span></span><br /><span data-ttu-id="76904-781">0 с</span><span class="sxs-lookup"><span data-stu-id="76904-781">0 sec</span></span><br /><span data-ttu-id="76904-782">60 с</span><span class="sxs-lookup"><span data-stu-id="76904-782">60 sec</span></span><br /><span data-ttu-id="76904-783">2 с</span><span class="sxs-lookup"><span data-stu-id="76904-783">2 sec</span></span><br /><span data-ttu-id="76904-784">false</span><span class="sxs-lookup"><span data-stu-id="76904-784">false</span></span> |<span data-ttu-id="76904-785">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="76904-785">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="76904-786">Попытка 2 — задержка 2 с</span><span class="sxs-lookup"><span data-stu-id="76904-786">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="76904-787">Попытка 3 — задержка 6 с</span><span class="sxs-lookup"><span data-stu-id="76904-787">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="76904-788">Попытка 4 — задержка 14 с</span><span class="sxs-lookup"><span data-stu-id="76904-788">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="76904-789">Попытка 5 — задержка 30 с</span><span class="sxs-lookup"><span data-stu-id="76904-789">Attempt 5 - delay ~30 sec</span></span> |

### <a name="more-information"></a><span data-ttu-id="76904-790">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="76904-790">More information</span></span>
* <span data-ttu-id="76904-791">[Библиотеки аутентификации Azure Active Directory][adal]</span><span class="sxs-lookup"><span data-stu-id="76904-791">[Azure Active Directory Authentication Libraries][adal]</span></span>

## <a name="service-fabric-retry-guidelines"></a><span data-ttu-id="76904-792">Руководство по использованию механизма повторов для Service Fabric</span><span class="sxs-lookup"><span data-stu-id="76904-792">Service Fabric retry guidelines</span></span>

<span data-ttu-id="76904-793">Распространение надежных служб в кластере Service Fabric защищает от большинства потенциальных временных сбоев, которые описаны в этой статье.</span><span class="sxs-lookup"><span data-stu-id="76904-793">Distributing reliable services in a Service Fabric cluster guards against most of the potential transient faults discussed in this article.</span></span> <span data-ttu-id="76904-794">Но некоторые временные сбои все равно могут произойти.</span><span class="sxs-lookup"><span data-stu-id="76904-794">Some transient faults are still possible, however.</span></span> <span data-ttu-id="76904-795">Например, если служба именования в момент получения запроса выполняет изменение маршрутизации, она создаст исключение.</span><span class="sxs-lookup"><span data-stu-id="76904-795">For example, the naming service might be in the middle of a routing change when it gets a request, causing it to throw an exception.</span></span> <span data-ttu-id="76904-796">Если повторить этот же запрос через 100 миллисекунд, он, скорее всего, завершится успешно.</span><span class="sxs-lookup"><span data-stu-id="76904-796">If the same request comes 100 milliseconds later, it will probably succeed.</span></span>

<span data-ttu-id="76904-797">Service Fabric обрабатывает такие временные сбои на внутреннем уровне.</span><span class="sxs-lookup"><span data-stu-id="76904-797">Internally, Service Fabric manages this kind of transient fault.</span></span> <span data-ttu-id="76904-798">Некоторые параметры можно настроить с помощью класса `OperationRetrySettings` при настройке служб.</span><span class="sxs-lookup"><span data-stu-id="76904-798">You can configure some settings by using the `OperationRetrySettings` class while setting up your services.</span></span>  <span data-ttu-id="76904-799">Пример такой настройки приведен в следующем коде.</span><span class="sxs-lookup"><span data-stu-id="76904-799">The following code shows an example.</span></span> <span data-ttu-id="76904-800">В большинстве случаев это не требуется, и можно использовать параметры по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="76904-800">In most cases, this should not be necessary, and the default settings will be fine.</span></span>

```csharp
    FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
    {
        OperationTimeout = TimeSpan.FromSeconds(30)
    };

    var retrySettings = new OperationRetrySettings(TimeSpan.FromSeconds(15), TimeSpan.FromSeconds(1), 5);

    var clientFactory = new FabricTransportServiceRemotingClientFactory(transportSettings);

    var serviceProxyFactory = new ServiceProxyFactory((c) => clientFactory, retrySettings);

    var client = serviceProxyFactory.CreateServiceProxy<ISomeService>(
        new Uri("fabric:/SomeApp/SomeStatefulReliableService"),
        new ServicePartitionKey(0));
```

## <a name="more-information"></a><span data-ttu-id="76904-801">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="76904-801">More information</span></span>

* [<span data-ttu-id="76904-802">Удаленная обработка исключений</span><span class="sxs-lookup"><span data-stu-id="76904-802">Remote Exception Handling</span></span>](https://github.com/Microsoft/azure-docs/blob/master/articles/service-fabric/service-fabric-reliable-services-communication-remoting.md#remoting-exception-handling)


## <a name="azure-event-hubs-retry-guidelines"></a><span data-ttu-id="76904-803">Рекомендации по использованию механизма повторных попыток для концентраторов событий Azure</span><span class="sxs-lookup"><span data-stu-id="76904-803">Azure Event Hubs retry guidelines</span></span>

<span data-ttu-id="76904-804">Концентраторы событий Azure — это служба обработки данных телеметрии с высокой степенью масштабируемости. Она собирает, преобразовывает и хранит миллионы событий.</span><span class="sxs-lookup"><span data-stu-id="76904-804">Azure Event Hubs is a hyper-scale telemetry ingestion service that collects, transforms, and stores millions of events.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="76904-805">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="76904-805">Retry mechanism</span></span>
<span data-ttu-id="76904-806">Механизм повторных попыток в клиентской библиотеке для концентраторов событий Azure управляется свойством `RetryPolicy` в классе `EventHubClient`.</span><span class="sxs-lookup"><span data-stu-id="76904-806">Retry behavior in the Azure Event Hubs Client Library is controlled by the `RetryPolicy` property on the `EventHubClient` class.</span></span> <span data-ttu-id="76904-807">По умолчанию, если концентратор Azure возвращает временную ошибку `EventHubsException` или `OperationCanceledException`, используется политика повторных попыток с экспоненциальным увеличением задержки.</span><span class="sxs-lookup"><span data-stu-id="76904-807">The default policy retries with exponential backoff when Azure Event Hub returns a transient `EventHubsException` or an `OperationCanceledException`.</span></span>

### <a name="example"></a><span data-ttu-id="76904-808">Пример</span><span class="sxs-lookup"><span data-stu-id="76904-808">Example</span></span>
```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a><span data-ttu-id="76904-809">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="76904-809">More information</span></span>
[<span data-ttu-id="76904-810">Клиентская библиотека .NET Standard для концентраторов событий Azure</span><span class="sxs-lookup"><span data-stu-id="76904-810"> .NET Standard client library for Azure Event Hubs</span></span>](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="general-rest-and-retry-guidelines"></a><span data-ttu-id="76904-811">Общие рекомендации по использованию REST и механизма повторов</span><span class="sxs-lookup"><span data-stu-id="76904-811">General REST and retry guidelines</span></span>
<span data-ttu-id="76904-812">При обращении к службам Azure или службам сторонних производителей учитывайте следующее.</span><span class="sxs-lookup"><span data-stu-id="76904-812">Consider the following when accessing Azure or third party services:</span></span>

* <span data-ttu-id="76904-813">Используйте систематический подход к управлению механизмом повторов, возможно в виде повторно используемого кода, что таким образом дает возможность применения согласованной методологии ко всем клиентам и всем решениям.</span><span class="sxs-lookup"><span data-stu-id="76904-813">Use a systematic approach to managing retries, perhaps as reusable code, so that you can apply a consistent methodology across all clients and all solutions.</span></span>
* <span data-ttu-id="76904-814">Рассмотрите возможность использования платформы повторов, например блока приложения для обработки временной ошибки, для управления механизмом повторов, если целевая служба или клиент не имеет встроенного механизма повторов.</span><span class="sxs-lookup"><span data-stu-id="76904-814">Consider using a retry framework such as the Transient Fault Handling Application Block to manage retries if the target service or client has no built-in retry mechanism.</span></span> <span data-ttu-id="76904-815">Это поможет реализовать согласованный механизм повтора и может предоставить подходящую стратегию повторов по умолчанию для целевой службы.</span><span class="sxs-lookup"><span data-stu-id="76904-815">This will help you implement a consistent retry behavior, and it may provide a suitable default retry strategy for the target service.</span></span> <span data-ttu-id="76904-816">Тем не менее необходимо создать пользовательский код механизма повторов для служб, имеющих нестандартное поведение, которые не полагаются на исключения для определения временных сбоев, или если вы хотите использовать ответ **Retry-Response** для управления поведением механизма повторов.</span><span class="sxs-lookup"><span data-stu-id="76904-816">However, you may need to create custom retry code for services that have non-standard behavior, that do not rely on exceptions to indicate transient failures, or if you want to use a **Retry-Response** reply to manage retry behavior.</span></span>
* <span data-ttu-id="76904-817">Логика обнаружения временных сбоев будет зависеть от фактического клиентского API, который используется для вызова REST.</span><span class="sxs-lookup"><span data-stu-id="76904-817">The transient detection logic will depend on the actual client API you use to invoke the REST calls.</span></span> <span data-ttu-id="76904-818">Некоторые клиенты, например новый класс **HttpClient** , не создают исключения для запросов, завершенных с неуспешным кодом состояния HTTP.</span><span class="sxs-lookup"><span data-stu-id="76904-818">Some clients, such as the newer **HttpClient** class, will not throw exceptions for completed requests with a non-success HTTP status code.</span></span> <span data-ttu-id="76904-819">Это улучшает производительность, но предотвращает использование блока приложения для обработки временной ошибки.</span><span class="sxs-lookup"><span data-stu-id="76904-819">This improves performance but prevents the use of the Transient Fault Handling Application Block.</span></span> <span data-ttu-id="76904-820">В этом случае можно поместить вызов API REST с кодом, который создает исключения для неуспешных кодов состояния HTTP, которые затем могут быть обработаны в блоке.</span><span class="sxs-lookup"><span data-stu-id="76904-820">In this case you could wrap the call to the REST API with code that produces exceptions for non-success HTTP status codes, which can then be processed by the block.</span></span> <span data-ttu-id="76904-821">Также можно использовать другой механизм для запуска повторов.</span><span class="sxs-lookup"><span data-stu-id="76904-821">Alternatively, you can use a different mechanism to drive the retries.</span></span>
* <span data-ttu-id="76904-822">Код состояния HTTP, возвращаемый службой, позволяет указать, является ли ошибка временной.</span><span class="sxs-lookup"><span data-stu-id="76904-822">The HTTP status code returned from the service can help to indicate whether the failure is transient.</span></span> <span data-ttu-id="76904-823">Необходимо изучить исключения, генерируемые клиентом, или механизм повторов для доступа к коду состояния для определения эквивалентного типа исключения.</span><span class="sxs-lookup"><span data-stu-id="76904-823">You may need to examine the exceptions generated by a client or the retry framework to access the status code or to determine the equivalent exception type.</span></span> <span data-ttu-id="76904-824">Следующие коды HTTP обычно показывают, что повтор может быть выполнен.</span><span class="sxs-lookup"><span data-stu-id="76904-824">The following HTTP codes typically indicate that a retry is appropriate:</span></span>
  * <span data-ttu-id="76904-825">408 — Истекло время ожидания запроса</span><span class="sxs-lookup"><span data-stu-id="76904-825">408 Request Timeout</span></span>
  * <span data-ttu-id="76904-826">500 — Внутренняя ошибка сервера</span><span class="sxs-lookup"><span data-stu-id="76904-826">500 Internal Server Error</span></span>
  * <span data-ttu-id="76904-827">502 — Недопустимый шлюз</span><span class="sxs-lookup"><span data-stu-id="76904-827">502 Bad Gateway</span></span>
  * <span data-ttu-id="76904-828">503 — Служба недоступна</span><span class="sxs-lookup"><span data-stu-id="76904-828">503 Service Unavailable</span></span>
  * <span data-ttu-id="76904-829">504 — Истекло время ожидания шлюза</span><span class="sxs-lookup"><span data-stu-id="76904-829">504 Gateway Timeout</span></span>
* <span data-ttu-id="76904-830">Если логика повторов основана на исключениях, следующие параметры обычно указывают, что произошел временный сбой где не удалось создать подключение.</span><span class="sxs-lookup"><span data-stu-id="76904-830">If you base your retry logic on exceptions, the following typically indicate a transient failure where no connection could be established:</span></span>
  * <span data-ttu-id="76904-831">WebExceptionStatus.ConnectionClosed</span><span class="sxs-lookup"><span data-stu-id="76904-831">WebExceptionStatus.ConnectionClosed</span></span>
  * <span data-ttu-id="76904-832">WebExceptionStatus.ConnectFailure</span><span class="sxs-lookup"><span data-stu-id="76904-832">WebExceptionStatus.ConnectFailure</span></span>
  * <span data-ttu-id="76904-833">WebExceptionStatus.Timeout</span><span class="sxs-lookup"><span data-stu-id="76904-833">WebExceptionStatus.Timeout</span></span>
  * <span data-ttu-id="76904-834">WebExceptionStatus.RequestCanceled</span><span class="sxs-lookup"><span data-stu-id="76904-834">WebExceptionStatus.RequestCanceled</span></span>
* <span data-ttu-id="76904-835">Если служба недоступна, она может передать прогнозируемое значение задержки перед следующей повторной попыткой в заголовке ответа **Retry-After** или другом пользовательском заголовке.</span><span class="sxs-lookup"><span data-stu-id="76904-835">In the case of a service unavailable status, the service might indicate the appropriate delay before retrying in the **Retry-After** response header or a different custom header.</span></span> <span data-ttu-id="76904-836">Службы также отправляют дополнительную информацию в виде пользовательских заголовков или внедренной в содержимое ответа.</span><span class="sxs-lookup"><span data-stu-id="76904-836">Services might also send additional information as custom headers, or embedded in the content of the response.</span></span> <span data-ttu-id="76904-837">Блок приложения для обработки временной ошибки не может использовать стандартные или пользовательские заголовки retry-after.</span><span class="sxs-lookup"><span data-stu-id="76904-837">The Transient Fault Handling Application Block cannot use the standard or any custom “retry-after” headers.</span></span>
* <span data-ttu-id="76904-838">Не выполняйте повторы для кодов состояния, представляющих клиентские ошибки (ошибки в диапазоне 4xx) за исключением ошибки «408 — Истекло время ожидания запроса».</span><span class="sxs-lookup"><span data-stu-id="76904-838">Do not retry for status codes representing client errors (errors in the 4xx range) except for a 408 Request Timeout.</span></span>
* <span data-ttu-id="76904-839">Тщательно протестируйте свои стратегии и механизмы повторов для различных условий, например, указав другую сеть и с различными нагрузками системы.</span><span class="sxs-lookup"><span data-stu-id="76904-839">Thoroughly test your retry strategies and mechanisms under a range of conditions, such as different network states and varying system loadings.</span></span>

### <a name="retry-strategies"></a><span data-ttu-id="76904-840">Стратегия повторов</span><span class="sxs-lookup"><span data-stu-id="76904-840">Retry strategies</span></span>
<span data-ttu-id="76904-841">Ниже приведены типичные виды интервалов стратегий повтора.</span><span class="sxs-lookup"><span data-stu-id="76904-841">The following are the typical types of retry strategy intervals:</span></span>

* <span data-ttu-id="76904-842">**Экспоненциальный**: политика повторов, основанная на выполнении указанного числа повторных попыток, используя случайный экспоненциальный пассивный метод для определения интервала между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="76904-842">**Exponential**: A retry policy that performs a specified number of retries, using a randomized exponential back off approach to determine the interval between retries.</span></span> <span data-ttu-id="76904-843">Например: </span><span class="sxs-lookup"><span data-stu-id="76904-843">For example:</span></span>

        var random = new Random();

        var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                    random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                    (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
        var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                       this.maxBackoff.TotalMilliseconds);
        retryInterval = TimeSpan.FromMilliseconds(interval);
* <span data-ttu-id="76904-844">**Добавочный**: стратегия повторов с указанным количеством повторных попыток и добавочным интервалом между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="76904-844">**Incremental**: A retry strategy with a specified number of retry attempts and an incremental time interval between retries.</span></span> <span data-ttu-id="76904-845">Например: </span><span class="sxs-lookup"><span data-stu-id="76904-845">For example:</span></span>

        retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                       (this.increment.TotalMilliseconds * currentRetryCount));
* <span data-ttu-id="76904-846">**Линейный повтор**: политика повторов, которая выполняет указанное число повторных попыток, используя указанный фиксированный интервал времени между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="76904-846">**LinearRetry**: A retry policy that performs a specified number of retries, using a specified fixed time interval between retries.</span></span> <span data-ttu-id="76904-847">Например: </span><span class="sxs-lookup"><span data-stu-id="76904-847">For example:</span></span>

        retryInterval = this.deltaBackoff;

### <a name="more-information"></a><span data-ttu-id="76904-848">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="76904-848">More information</span></span>
* [<span data-ttu-id="76904-849">Стратегии выключателя</span><span class="sxs-lookup"><span data-stu-id="76904-849">Circuit breaker strategies</span></span>](http://msdn.microsoft.com/library/dn589784.aspx)

## <a name="transient-fault-handling-with-polly"></a><span data-ttu-id="76904-850">Обработка временного сбоя с помощью Polly</span><span class="sxs-lookup"><span data-stu-id="76904-850">Transient fault handling with Polly</span></span>
<span data-ttu-id="76904-851">Библиотека [Polly](http://www.thepollyproject.org) предназначена для программной обработки механизма повторных попыток и стратегий [автоматического прерывания][circuit-breaker].</span><span class="sxs-lookup"><span data-stu-id="76904-851">[Polly](http://www.thepollyproject.org) is a library to programatically handle retries and [circuit breaker][circuit-breaker] strategies.</span></span> <span data-ttu-id="76904-852">Проект Polly является частью [.NET Foundation][dotnet-foundation].</span><span class="sxs-lookup"><span data-stu-id="76904-852">The Polly project is a member of the [.NET Foundation][dotnet-foundation].</span></span> <span data-ttu-id="76904-853">Применение Polly будет неплохим вариантом при работе со службами, клиент которых не имеет встроенного механизма повторных попыток. Вам не придется самостоятельно создавать код для повторных попыток, который иногда очень трудно реализовать правильно.</span><span class="sxs-lookup"><span data-stu-id="76904-853">For services where the client does not natively support retries, Polly is a valid alternative and avoids the need to write custom retry code, which can be hard to implement correctly.</span></span> <span data-ttu-id="76904-854">Polly также поддерживает трассировку возникающих ошибок, что позволяет вести журнал повторных попыток.</span><span class="sxs-lookup"><span data-stu-id="76904-854">Polly also provides a way to trace errors when they occur, so that you can log retries.</span></span>

<!-- links -->

[adal]: /azure/active-directory/develop/active-directory-authentication-libraries
[autorest]: https://github.com/Azure/autorest/tree/master/docs
[circuit-breaker]: ../patterns/circuit-breaker.md
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[dotnet-foundation]: https://dotnetfoundation.org/
[polly]: http://www.thepollyproject.org
[redis-cache-troubleshoot]: /azure/redis-cache/cache-how-to-troubleshoot
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx
