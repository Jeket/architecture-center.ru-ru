---
title: Антишаблон синхронных операций ввода-вывода
titleSuffix: Performance antipatterns for cloud apps
description: Блокировка вызывающего потока до завершения операций ввода-вывода может привести к снижению производительности. Кроме того, она влияет на вертикальное масштабирование.
author: dragon119
ms.date: 06/05/2017
ms.topic: article
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 1b53806b2939a7c44a8b48c9146d5e86c84d9e2e
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58343566"
---
# <a name="synchronous-io-antipattern"></a><span data-ttu-id="a8bfc-103">Антишаблон синхронных операций ввода-вывода</span><span class="sxs-lookup"><span data-stu-id="a8bfc-103">Synchronous I/O antipattern</span></span>

<span data-ttu-id="a8bfc-104">Блокировка вызывающего потока до завершения операций ввода-вывода может привести к снижению производительности. Кроме того, она влияет на вертикальное масштабирование.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-104">Blocking the calling thread while I/O completes can reduce performance and affect vertical scalability.</span></span>

## <a name="problem-description"></a><span data-ttu-id="a8bfc-105">Описание проблемы</span><span class="sxs-lookup"><span data-stu-id="a8bfc-105">Problem description</span></span>

<span data-ttu-id="a8bfc-106">Синхронные операции ввода-вывода блокируют вызывающий поток, пока не завершится операция ввода-вывода.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-106">A synchronous I/O operation blocks the calling thread while the I/O completes.</span></span> <span data-ttu-id="a8bfc-107">Вызывающий поток переходит в состояние ожидания и не может выполнить нужные действия во время этого интервала, растрачивая ресурсы обработки.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-107">The calling thread enters a wait state and is unable to perform useful work during this interval, wasting processing resources.</span></span>

<span data-ttu-id="a8bfc-108">Ниже приведены распространенные примеры операций ввода-вывода:</span><span class="sxs-lookup"><span data-stu-id="a8bfc-108">Common examples of I/O include:</span></span>

- <span data-ttu-id="a8bfc-109">Получение или сохранение данных в базу данных или любое постоянное хранилище.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-109">Retrieving or persisting data to a database or any type of persistent storage.</span></span>
- <span data-ttu-id="a8bfc-110">Отправка запроса к веб-службе.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-110">Sending a request to a web service.</span></span>
- <span data-ttu-id="a8bfc-111">Публикация сообщения или получение сообщения из очереди.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-111">Posting a message or retrieving a message from a queue.</span></span>
- <span data-ttu-id="a8bfc-112">Запись в локальный файл или считывание данных из него.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-112">Writing to or reading from a local file.</span></span>

<span data-ttu-id="a8bfc-113">Этот антишаблон обычно возникает в следующих случаях:</span><span class="sxs-lookup"><span data-stu-id="a8bfc-113">This antipattern typically occurs because:</span></span>

- <span data-ttu-id="a8bfc-114">Это самый простой способ выполнения операции.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-114">It appears to be the most intuitive way to perform an operation.</span></span>
- <span data-ttu-id="a8bfc-115">Приложению требуется ответ на запрос.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-115">The application requires a response from a request.</span></span>
- <span data-ttu-id="a8bfc-116">Приложение использует библиотеку, которая предоставляет только синхронные методы для операций ввода-вывода.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-116">The application uses a library that only provides synchronous methods for I/O.</span></span>
- <span data-ttu-id="a8bfc-117">Внешняя библиотека выполняет синхронные операции ввода-вывода изнутри.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-117">An external library performs synchronous I/O operations internally.</span></span> <span data-ttu-id="a8bfc-118">Один вызов синхронных операций ввода-вывода может блокировать всю цепочку вызовов.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-118">A single synchronous I/O call can block an entire call chain.</span></span>

<span data-ttu-id="a8bfc-119">Приведенный ниже код передает файл в хранилище BLOB-объектов Azure.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-119">The following code uploads a file to Azure blob storage.</span></span> <span data-ttu-id="a8bfc-120">Существует два места, где код выполняет блокировку, ожидая выполнения синхронных операций ввода-вывода: метод `CreateIfNotExists` и `UploadFromStream`.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-120">There are two places where the code blocks waiting for synchronous I/O, the `CreateIfNotExists` method and the `UploadFromStream` method.</span></span>

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

container.CreateIfNotExists();
var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    blockBlob.UploadFromStream(fileStream);
}
```

<span data-ttu-id="a8bfc-121">Ниже приведен пример ожидания ответа от внешней службы.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-121">Here's an example of waiting for a response from an external service.</span></span> <span data-ttu-id="a8bfc-122">Метод `GetUserProfile` вызывает удаленную службу, которая возвращает `UserProfile`.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-122">The `GetUserProfile` method calls a remote service that returns a `UserProfile`.</span></span>

```csharp
public interface IUserProfileService
{
    UserProfile GetUserProfile();
}

public class SyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public SyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is a synchronous method that calls the synchronous GetUserProfile method.
    public UserProfile GetUserProfile()
    {
        return _userProfileService.GetUserProfile();
    }
}
```

<span data-ttu-id="a8bfc-123">Полный код для этих примеров можно найти [здесь][sample-app].</span><span class="sxs-lookup"><span data-stu-id="a8bfc-123">You can find the complete code for both of these examples [here][sample-app].</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="a8bfc-124">Как устранить проблему</span><span class="sxs-lookup"><span data-stu-id="a8bfc-124">How to fix the problem</span></span>

<span data-ttu-id="a8bfc-125">Замените синхронные операции ввода-вывода асинхронными.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-125">Replace synchronous I/O operations with asynchronous operations.</span></span> <span data-ttu-id="a8bfc-126">Они не заблокируют текущий поток, а освободят его для продолжения выполнения работы, а также улучшат использование вычислительных ресурсов.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-126">This frees the current thread to continue performing meaningful work rather than blocking, and helps improve the utilization of compute resources.</span></span> <span data-ttu-id="a8bfc-127">Выполнение асинхронных операций ввода-вывода особенно эффективно для обработки непредвиденного всплеска запросов от клиентских приложений.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-127">Performing I/O asynchronously is particularly efficient for handling an unexpected surge in requests from client applications.</span></span>

<span data-ttu-id="a8bfc-128">Многие библиотеки предоставляют синхронные и асинхронные версии методов.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-128">Many libraries provide both synchronous and asynchronous versions of methods.</span></span> <span data-ttu-id="a8bfc-129">По возможности используйте асинхронные.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-129">Whenever possible, use the asynchronous versions.</span></span> <span data-ttu-id="a8bfc-130">Ниже приведена асинхронная версия предыдущего примера, которая отправляет файл в хранилище BLOB-объектов Azure.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-130">Here is the asynchronous version of the previous example that uploads a file to Azure blob storage.</span></span>

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

await container.CreateIfNotExistsAsync();

var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    await blockBlob.UploadFromStreamAsync(fileStream);
}
```

<span data-ttu-id="a8bfc-131">Во время выполнения асинхронной операции оператор `await` возвращает управление вызывающей среде.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-131">The `await` operator returns control to the calling environment while the asynchronous operation is performed.</span></span> <span data-ttu-id="a8bfc-132">После этого код выступает в качестве кода продолжения, который будет выполняться по завершении асинхронной операции.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-132">The code after this statement acts as a continuation that runs when the asynchronous operation has completed.</span></span>

<span data-ttu-id="a8bfc-133">Хорошо спроектированные службы также должны предоставлять асинхронные операции.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-133">A well designed service should also provide asynchronous operations.</span></span> <span data-ttu-id="a8bfc-134">Ниже приведен пример асинхронной версии веб-службы, которая возвращает профили пользователей.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-134">Here is an asynchronous version of the web service that returns user profiles.</span></span> <span data-ttu-id="a8bfc-135">Метод `GetUserProfileAsync` зависит от наличия асинхронной версии службы профилей пользователей.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-135">The `GetUserProfileAsync` method depends on having an asynchronous version of the User Profile service.</span></span>

```csharp
public interface IUserProfileService
{
    Task<UserProfile> GetUserProfileAsync();
}

public class AsyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public AsyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is an synchronous method that calls the Task based GetUserProfileAsync method.
    public Task<UserProfile> GetUserProfileAsync()
    {
        return _userProfileService.GetUserProfileAsync();
    }
}
```

<span data-ttu-id="a8bfc-136">Библиотеки, которые не предоставляют асинхронные версии операций, могут создать асинхронные программы-оболочки вокруг выбранных синхронных методов.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-136">For libraries that don't provide asynchronous versions of operations, it may be possible to create asynchronous wrappers around selected synchronous methods.</span></span> <span data-ttu-id="a8bfc-137">Используйте этот подход с осторожностью.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-137">Follow this approach with caution.</span></span> <span data-ttu-id="a8bfc-138">Он потребляет больше ресурсов, хотя и может повысить скорость реагирования на поток, который вызывает асинхронную программу-оболочку.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-138">While it may improve responsiveness on the thread that invokes the asynchronous wrapper, it actually consumes more resources.</span></span> <span data-ttu-id="a8bfc-139">Вы можете создать дополнительный поток, однако с синхронизацией работы, выполняемой этим потоком, связаны определенные затраты.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-139">An extra thread may be created, and there is overhead associated with synchronizing the work done by this thread.</span></span> <span data-ttu-id="a8bfc-140">В следующей записи блога рассматриваются некоторые компромиссы: [Should I expose asynchronous wrappers for synchronous methods?][async-wrappers] (Следует ли предоставлять асинхронные программы-оболочки для синхронных методов?)</span><span class="sxs-lookup"><span data-stu-id="a8bfc-140">Some tradeoffs are discussed in this blog post: [Should I expose asynchronous wrappers for synchronous methods?][async-wrappers]</span></span>

<span data-ttu-id="a8bfc-141">Ниже приведен пример асинхронной программы-оболочки, применяемой к синхронному методу.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-141">Here is an example of an asynchronous wrapper around a synchronous method.</span></span>

```csharp
// Asynchronous wrapper around synchronous library method
private async Task<int> LibraryIOOperationAsync()
{
    return await Task.Run(() => LibraryIOOperation());
}
```

<span data-ttu-id="a8bfc-142">Теперь вызывающий код может ожидать в программе-оболочке:</span><span class="sxs-lookup"><span data-stu-id="a8bfc-142">Now the calling code can await on the wrapper:</span></span>

```csharp
// Invoke the asynchronous wrapper using a task
await LibraryIOOperationAsync();
```

## <a name="considerations"></a><span data-ttu-id="a8bfc-143">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="a8bfc-143">Considerations</span></span>

- <span data-ttu-id="a8bfc-144">Операции ввода-вывода, которые длятся недолго и вряд ли будут вызывать конфликты, могут быть более производительными, чем синхронные операции.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-144">I/O operations that are expected to be very short lived and are unlikely to cause contention might be more performant as synchronous operations.</span></span> <span data-ttu-id="a8bfc-145">Примером может быть чтение небольших файлов на SSD-диске.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-145">An example might be reading small files on an SSD drive.</span></span> <span data-ttu-id="a8bfc-146">Дополнительные издержки, связанные с отправкой задачи в другой поток и синхронизацией с этим потоком после завершения задачи, могут перевесить преимущества асинхронного ввода-вывода.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-146">The overhead of dispatching a task to another thread, and synchronizing with that thread when the task completes, might outweigh the benefits of asynchronous I/O.</span></span> <span data-ttu-id="a8bfc-147">Однако такое случается редко, и большинство операций ввода-вывода должны выполняться асинхронно.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-147">However, these cases are relatively rare, and most I/O operations should be done asynchronously.</span></span>

- <span data-ttu-id="a8bfc-148">Повышение производительности операций ввода-вывода может стать причиной возникновения узких мест в системе.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-148">Improving I/O performance may cause other parts of the system to become bottlenecks.</span></span> <span data-ttu-id="a8bfc-149">Например, разблокирование потоков может привести к увеличению объема одновременных запросов к общим ресурсам. В результате возникнет нехватка ресурсов или необходимость регулирования.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-149">For example, unblocking threads might result in a higher volume of concurrent requests to shared resources, leading in turn to resource starvation or throttling.</span></span> <span data-ttu-id="a8bfc-150">Если это становится проблемой, нужно увеличить количество веб-серверов или выполнить секционирование хранилищ данных, чтобы уменьшить количество конфликтов.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-150">If that becomes a problem, you might need to scale out the number of web servers or partition data stores to reduce contention.</span></span>

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="a8bfc-151">Как определить проблему</span><span class="sxs-lookup"><span data-stu-id="a8bfc-151">How to detect the problem</span></span>

<span data-ttu-id="a8bfc-152">Приложение может не отвечать или периодически зависать.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-152">For users, the application may seem unresponsive or appear to hang periodically.</span></span> <span data-ttu-id="a8bfc-153">Может произойти сбой приложения с исключением времени ожидания.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-153">The application might fail with timeout exceptions.</span></span> <span data-ttu-id="a8bfc-154">Эти сбои также могут возвращать ошибки HTTP 500 (внутренняя ошибка сервера).</span><span class="sxs-lookup"><span data-stu-id="a8bfc-154">These failures could also return HTTP 500 (Internal Server) errors.</span></span> <span data-ttu-id="a8bfc-155">На сервере входящие запросы клиентов могут блокироваться до тех пор, пока поток не станет доступным. В результате существенно увеличится очередь запросов и отобразится ошибка HTTP 503 (служба недоступна).</span><span class="sxs-lookup"><span data-stu-id="a8bfc-155">On the server, incoming client requests might be blocked until a thread becomes available, resulting in excessive request queue lengths, manifested as HTTP 503 (Service Unavailable) errors.</span></span>

<span data-ttu-id="a8bfc-156">Чтобы определить проблему, сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="a8bfc-156">You can perform the following steps to help identify the problem:</span></span>

1. <span data-ttu-id="a8bfc-157">Выполните мониторинг рабочей системы и определите, ограничивают ли заблокированные рабочие потоки пропускную способность.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-157">Monitor the production system and determine whether blocked worker threads are constraining throughput.</span></span>

2. <span data-ttu-id="a8bfc-158">Если запросы блокируются из-за отсутствия потоков, проверьте приложение, чтобы определить, какие операции выполняют ввод-вывод синхронно.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-158">If requests are being blocked due to lack of threads, review the application to determine which operations may be performing I/O synchronously.</span></span>

3. <span data-ttu-id="a8bfc-159">Выполните управляемое нагрузочное тестирование каждой операции, которая выполняет синхронный ввод-вывод, чтобы узнать, влияют ли они на производительность системы.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-159">Perform controlled load testing of each operation that is performing synchronous I/O, to find out whether those operations are affecting system performance.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="a8bfc-160">Пример диагностики</span><span class="sxs-lookup"><span data-stu-id="a8bfc-160">Example diagnosis</span></span>

<span data-ttu-id="a8bfc-161">В следующих разделах эти шаги применяются к примеру приложения, описанному ранее.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-161">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="monitor-web-server-performance"></a><span data-ttu-id="a8bfc-162">Мониторинг производительности веб-сервера</span><span class="sxs-lookup"><span data-stu-id="a8bfc-162">Monitor web server performance</span></span>

<span data-ttu-id="a8bfc-163">Мониторинг производительности веб-сервера IIS нужно выполнять для веб-ролей и веб-приложений Azure.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-163">For Azure web applications and web roles, it's worth monitoring the performance of the IIS web server.</span></span> <span data-ttu-id="a8bfc-164">В частности, обратите внимание на длину очередей запросов, чтобы узнать, блокируются ли запросы в ожидании доступных потоков во время периодов высокой активности.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-164">In particular, pay attention to the request queue length to establish whether requests are being blocked waiting for available threads during periods of high activity.</span></span> <span data-ttu-id="a8bfc-165">Эти данные можно собирать, включив систему диагностики Azure.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-165">You can gather this information by enabling Azure diagnostics.</span></span> <span data-ttu-id="a8bfc-166">Дополнительные сведения можно найти в разделе </span><span class="sxs-lookup"><span data-stu-id="a8bfc-166">For more information, see:</span></span>

- <span data-ttu-id="a8bfc-167">[Мониторинг приложений в службе приложений Azure][web-sites-monitor].</span><span class="sxs-lookup"><span data-stu-id="a8bfc-167">[Monitor Apps in Azure App Service][web-sites-monitor]</span></span>
- <span data-ttu-id="a8bfc-168">[Создание и использование счетчиков производительности в приложении Azure][performance-counters].</span><span class="sxs-lookup"><span data-stu-id="a8bfc-168">[Create and use performance counters in an Azure application][performance-counters]</span></span>

<span data-ttu-id="a8bfc-169">Инструментируйте приложение, чтобы узнать, как обрабатываются принятые запросы.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-169">Instrument the application to see how requests are handled once they have been accepted.</span></span> <span data-ttu-id="a8bfc-170">Трассировка последовательности запроса может помочь определить, выполняются ли медленные вызовы и блокируют ли он текущий поток.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-170">Tracing the flow of a request can help to identify whether it is performing slow-running calls and blocking the current thread.</span></span> <span data-ttu-id="a8bfc-171">Профилирование потока также может помочь выделить заблокированные запросы.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-171">Thread profiling can also highlight requests that are being blocked.</span></span>

### <a name="load-test-the-application"></a><span data-ttu-id="a8bfc-172">Нагрузочное тестирование приложения</span><span class="sxs-lookup"><span data-stu-id="a8bfc-172">Load test the application</span></span>

<span data-ttu-id="a8bfc-173">На следующем графике показана производительность синхронного метода `GetUserProfile` (описанного ранее) при различных нагрузках до 4000 одновременных пользователей.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-173">The following graph shows the performance of the synchronous `GetUserProfile` method shown earlier, under varying loads of up to 4000 concurrent users.</span></span> <span data-ttu-id="a8bfc-174">В этом примере показано приложение ASP.NET, работающее в веб-роли облачной службы Azure.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-174">The application is an ASP.NET application running in an Azure Cloud Service web role.</span></span>

![Диаграмма производительности для примера приложения, выполняющего синхронные операции ввода-вывода][sync-performance]

<span data-ttu-id="a8bfc-176">Для синхронной операции жестко задан спящий режим на 2 секунды, чтобы имитировать синхронный ввод-вывод, поэтому минимальное время отклика будет немного превышать 2 секунды.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-176">The synchronous operation is hard-coded to sleep for 2 seconds, to simulate synchronous I/O, so the minimum response time is slightly over 2 seconds.</span></span> <span data-ttu-id="a8bfc-177">Если нагрузка достигает приблизительно 2500 одновременных пользователей, среднее время отклика достигает пика, несмотря на то что объем запросов продолжает увеличиваться каждую секунду.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-177">When the load reaches approximately 2500 concurrent users, the average response time reaches a plateau, although the volume of requests per second continues to increase.</span></span> <span data-ttu-id="a8bfc-178">Обратите внимание, что эти величины измеряются по логарифмической шкале.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-178">Note that the scale for these two measures is logarithmic.</span></span> <span data-ttu-id="a8bfc-179">Количество запросов в секунду увеличивается вдвое между этой точкой и временем завершения теста.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-179">The number of requests per second doubles between this point and the end of the test.</span></span>

<span data-ttu-id="a8bfc-180">В результате этого изолированного теста непонятно, является ли синхронный ввод-вывод проблемой.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-180">In isolation, it's not necessarily clear from this test whether the synchronous I/O is a problem.</span></span> <span data-ttu-id="a8bfc-181">При более высокой нагрузке приложение может достичь переломного момента, когда веб-сервер больше не может обрабатывать запросы своевременно. В результате в клиентских приложениях произойдет сбой с исключением времени ожидания.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-181">Under heavier load, the application may reach a tipping point where the web server can no longer process requests in a timely manner, causing client applications to receive time-out exceptions.</span></span>

<span data-ttu-id="a8bfc-182">Входящие запросы помещаются в очередь веб-сервером IIS и передаются в поток, который выполняется в пуле потоков ASP.NET.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-182">Incoming requests are queued by the IIS web server and handed to a thread running in the ASP.NET thread pool.</span></span> <span data-ttu-id="a8bfc-183">Поток блокируется до завершения операции, так как каждая операция выполняет ввод-вывод синхронно.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-183">Because each operation performs I/O synchronously, the thread is blocked until the operation completes.</span></span> <span data-ttu-id="a8bfc-184">При увеличении рабочей нагрузки, в конечном итоге, все потоки ASP.NET в пуле потоков выделяются и блокируются.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-184">As the workload increases, eventually all of the ASP.NET threads in the thread pool are allocated and blocked.</span></span> <span data-ttu-id="a8bfc-185">На этом этапе любые дополнительные входящие запросы должны ожидать в очереди, пока не освободится поток.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-185">At that point, any further incoming requests must wait in the queue for an available thread.</span></span> <span data-ttu-id="a8bfc-186">При увеличении длины очереди время ожидания запросов начинает истекать.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-186">As the queue length grows, requests start to time out.</span></span>

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="a8bfc-187">Реализация решения и проверка результатов</span><span class="sxs-lookup"><span data-stu-id="a8bfc-187">Implement the solution and verify the result</span></span>

<span data-ttu-id="a8bfc-188">На следующем графике показаны результаты нагрузочных тестов асинхронной версии кода.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-188">The next graph shows the results from load testing the asynchronous version of the code.</span></span>

![Диаграмма производительности для примера приложения, выполняющего асинхронные операции ввода-вывода][async-performance]

<span data-ttu-id="a8bfc-190">Пропускная способность намного выше.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-190">Throughput is far higher.</span></span> <span data-ttu-id="a8bfc-191">На протяжении того же промежутка времени, что и в предыдущем тесте, система успешно обрабатывает почти десятикратное увеличение пропускной способности, измеренной по количеству запросов в секунду.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-191">Over the same duration as the previous test, the system successfully handles a nearly tenfold increase in throughput, as measured in requests per second.</span></span> <span data-ttu-id="a8bfc-192">Кроме того, среднее время отклика является относительной константой и примерно в 25 раз меньше, чем в предыдущем тесте.</span><span class="sxs-lookup"><span data-stu-id="a8bfc-192">Moreover, the average response time is relatively constant and remains approximately 25 times smaller than the previous test.</span></span>

[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/SynchronousIO
[async-wrappers]: https://blogs.msdn.microsoft.com/pfxteam/2012/03/24/should-i-expose-asynchronous-wrappers-for-synchronous-methods/
[performance-counters]: /azure/cloud-services/cloud-services-dotnet-diagnostics-performance-counters
[web-sites-monitor]: /azure/app-service-web/web-sites-monitor

[sync-performance]: _images/SyncPerformance.jpg
[async-performance]: _images/AsyncPerformance.jpg
