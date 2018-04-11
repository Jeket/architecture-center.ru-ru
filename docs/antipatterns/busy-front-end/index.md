---
title: Антишаблон загруженности внешнего интерфейса
description: Выполнение асинхронных операций в большом количестве фоновых потоков может замедлить выполнение других задач переднего плана ресурсов.
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: cedb80ddac5ceb1eb901455df3165993fd28a138
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
---
# <a name="busy-front-end-antipattern"></a><span data-ttu-id="83875-103">Антишаблон загруженности внешнего интерфейса</span><span class="sxs-lookup"><span data-stu-id="83875-103">Busy Front End antipattern</span></span>

<span data-ttu-id="83875-104">Выполнение асинхронных операций в большом количестве фоновых потоков может замедлить выполнение других задач переднего плана ресурсов, что приводит к уменьшению времени отклика до недопустимого уровня.</span><span class="sxs-lookup"><span data-stu-id="83875-104">Performing asynchronous work on a large number of background threads can starve other concurrent foreground tasks of resources, decreasing response times to unacceptable levels.</span></span>

## <a name="problem-description"></a><span data-ttu-id="83875-105">Описание проблемы</span><span class="sxs-lookup"><span data-stu-id="83875-105">Problem description</span></span>

<span data-ttu-id="83875-106">Ресурсоемкие задачи могут увеличить время отклика на запросы пользователей и привести к высокой задержке.</span><span class="sxs-lookup"><span data-stu-id="83875-106">Resource-intensive tasks can increase the response times for user requests and cause high latency.</span></span> <span data-ttu-id="83875-107">Один из способов уменьшения времени отклика заключается в разгрузке ресурсоемких задач в отдельный поток.</span><span class="sxs-lookup"><span data-stu-id="83875-107">One way to improve response times is to offload a resource-intensive task to a separate thread.</span></span> <span data-ttu-id="83875-108">Этот подход позволяет приложению реагировать во время обработки в фоновом режиме.</span><span class="sxs-lookup"><span data-stu-id="83875-108">This approach lets the application stay responsive while processing happens in the background.</span></span> <span data-ttu-id="83875-109">Тем не менее задачи, выполняемые в фоновом потоке, по-прежнему потребляют ресурсы.</span><span class="sxs-lookup"><span data-stu-id="83875-109">However, tasks that run on a background thread still consume resources.</span></span> <span data-ttu-id="83875-110">Большое количество таких задач может замедлить выполнение потоков, обрабатывающих запросы.</span><span class="sxs-lookup"><span data-stu-id="83875-110">If there are too many of them, they can starve the threads that are handling requests.</span></span>

> [!NOTE]
> <span data-ttu-id="83875-111">Термин *ресурс* может охватывать множество понятий, таких как использование ЦП, заполнение памяти и операции ввода-вывода сети или диска.</span><span class="sxs-lookup"><span data-stu-id="83875-111">The term *resource* can encompass many things, such as CPU utilization, memory occupancy, and network or disk I/O.</span></span>

<span data-ttu-id="83875-112">Эта проблема обычно возникает, когда приложение разработано как монолитный фрагмент кода со всеми бизнес-логиками, объединенными на одном уровне с уровнем представления данных.</span><span class="sxs-lookup"><span data-stu-id="83875-112">This problem typically occurs when an application is developed as monolithic piece of code, with all of the business logic combined into a single tier shared with the presentation layer.</span></span>

<span data-ttu-id="83875-113">Ниже приведен пример кода ASP.NET, который демонстрирует эту проблему.</span><span class="sxs-lookup"><span data-stu-id="83875-113">Here’s an example using ASP.NET that demonstrates the problem.</span></span> <span data-ttu-id="83875-114">Полный пример см. [здесь][code-sample].</span><span class="sxs-lookup"><span data-stu-id="83875-114">You can find the complete sample [here][code-sample].</span></span>

```csharp
public class WorkInFrontEndController : ApiController
{
    [HttpPost]
    [Route("api/workinfrontend")]
    public HttpResponseMessage Post()
    {
        new Thread(() =>
        {
            //Simulate processing
            Thread.SpinWait(Int32.MaxValue / 100);
        }).Start();

        return Request.CreateResponse(HttpStatusCode.Accepted);
    }
}

public class UserProfileController : ApiController
{
    [HttpGet]
    [Route("api/userprofile/{id}")]
    public UserProfile Get(int id)
    {
        //Simulate processing
        return new UserProfile() { FirstName = "Alton", LastName = "Hudgens" };
    }
}
```

- <span data-ttu-id="83875-115">Метод `Post` в контроллере `WorkInFrontEnd` реализует операцию HTTP POST.</span><span class="sxs-lookup"><span data-stu-id="83875-115">The `Post` method in the `WorkInFrontEnd` controller implements an HTTP POST operation.</span></span> <span data-ttu-id="83875-116">Эта операция моделирует продолжительную задачу с интенсивной нагрузкой ЦП.</span><span class="sxs-lookup"><span data-stu-id="83875-116">This operation simulates a long-running, CPU-intensive task.</span></span> <span data-ttu-id="83875-117">Задача выполняется в отдельном потоке. Это позволяет выполнить операцию POST быстрее.</span><span class="sxs-lookup"><span data-stu-id="83875-117">The work is performed on a separate thread, in an attempt to enable the POST operation to complete quickly.</span></span>

- <span data-ttu-id="83875-118">Метод `Get` в контроллере `UserProfile` реализует операцию HTTP GET.</span><span class="sxs-lookup"><span data-stu-id="83875-118">The `Get` method in the `UserProfile` controller implements an HTTP GET operation.</span></span> <span data-ttu-id="83875-119">Этот метод требует меньше ресурсов ЦП.</span><span class="sxs-lookup"><span data-stu-id="83875-119">This method is much less CPU intensive.</span></span>

<span data-ttu-id="83875-120">Особое внимание следует уделить требованиям к ресурсам метода `Post`.</span><span class="sxs-lookup"><span data-stu-id="83875-120">The primary concern is the resource requirements of the `Post` method.</span></span> <span data-ttu-id="83875-121">Несмотря на то что задачи выполняются в фоновом потоке, они по-прежнему могут потреблять значительный объем ресурсов ЦП.</span><span class="sxs-lookup"><span data-stu-id="83875-121">Although it puts the work onto a background thread, the work can still consume considerable CPU resources.</span></span> <span data-ttu-id="83875-122">Эти ресурсы используются и другими операциями, выполняемыми другими параллельно работающими пользователями.</span><span class="sxs-lookup"><span data-stu-id="83875-122">These resources are shared with other operations being performed by other concurrent users.</span></span> <span data-ttu-id="83875-123">Если умеренное количество пользователей одновременно отправит этот запрос, это с большей вероятностью отрицательно повлияет на общую производительность, что приведет к снижению скорости выполнения всех операций.</span><span class="sxs-lookup"><span data-stu-id="83875-123">If a moderate number of users send this request at the same time, overall performance is likely to suffer, slowing down all operations.</span></span> <span data-ttu-id="83875-124">У пользователей, например, могут возникнуть значительные задержки в методе `Get`.</span><span class="sxs-lookup"><span data-stu-id="83875-124">Users might experience significant latency in the `Get` method, for example.</span></span>

## <a name="how-to-fix-the-problem"></a><span data-ttu-id="83875-125">Как устранить проблему</span><span class="sxs-lookup"><span data-stu-id="83875-125">How to fix the problem</span></span>

<span data-ttu-id="83875-126">Переместите процессы, требующие значительные ресурсы, в отдельную серверную часть.</span><span class="sxs-lookup"><span data-stu-id="83875-126">Move processes that consume significant resources to a separate back end.</span></span> 

<span data-ttu-id="83875-127">При этом подходе внешний интерфейс помещает ресурсоемкие задачи в очередь сообщений.</span><span class="sxs-lookup"><span data-stu-id="83875-127">With this approach, the front end puts resource-intensive tasks onto a message queue.</span></span> <span data-ttu-id="83875-128">Серверная часть выбирает эти задачи и асинхронно обрабатывает их.</span><span class="sxs-lookup"><span data-stu-id="83875-128">The back end picks up the tasks for asynchronous processing.</span></span> <span data-ttu-id="83875-129">Очередь также позволяет выровнять нагрузку, выполняя буферизацию запросов серверной части.</span><span class="sxs-lookup"><span data-stu-id="83875-129">The queue also acts as a load leveler, buffering requests for the back end.</span></span> <span data-ttu-id="83875-130">Если очередь становится слишком длинной, вы можете настроить автоматическое масштабирование серверной части.</span><span class="sxs-lookup"><span data-stu-id="83875-130">If the queue length becomes too long, you can configure autoscaling to scale out the back end.</span></span>

<span data-ttu-id="83875-131">Ниже приведена исправленная версия предыдущего примера кода.</span><span class="sxs-lookup"><span data-stu-id="83875-131">Here is a revised version of the previous code.</span></span> <span data-ttu-id="83875-132">В этой версии метод `Post` помещает сообщение в очередь служебной шины.</span><span class="sxs-lookup"><span data-stu-id="83875-132">In this version, the `Post` method puts a message on a Service Bus queue.</span></span> 

```csharp
public class WorkInBackgroundController : ApiController
{
    private static readonly QueueClient QueueClient;
    private static readonly string QueueName;
    private static readonly ServiceBusQueueHandler ServiceBusQueueHandler;

    public WorkInBackgroundController()
    {
        var serviceBusConnectionString = ...;
        QueueName = ...;
        ServiceBusQueueHandler = new ServiceBusQueueHandler(serviceBusConnectionString);
        QueueClient = ServiceBusQueueHandler.GetQueueClientAsync(QueueName).Result;
    }

    [HttpPost]
    [Route("api/workinbackground")]
    public async Task<long> Post()
    {
        return await ServiceBusQueuehandler.AddWorkLoadToQueueAsync(QueueClient, QueueName, 0);
    }
}
```

<span data-ttu-id="83875-133">Серверная часть извлекает эти сообщения и обрабатывает их.</span><span class="sxs-lookup"><span data-stu-id="83875-133">The back end pulls messages from the Service Bus queue and does the processing.</span></span>

```csharp
public async Task RunAsync(CancellationToken cancellationToken)
{
    this._queueClient.OnMessageAsync(
        // This lambda is invoked for each message received.
        async (receivedMessage) =>
        {
            try
            {
                // Simulate processing of message
                Thread.SpinWait(Int32.Maxvalue / 1000);

                await receivedMessage.CompleteAsync();
            }
            catch
            {
                receivedMessage.Abandon();
            }
        });
}
```

## <a name="considerations"></a><span data-ttu-id="83875-134">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="83875-134">Considerations</span></span>

- <span data-ttu-id="83875-135">Этот подход усложняет архитектуру приложения.</span><span class="sxs-lookup"><span data-stu-id="83875-135">This approach adds some additional complexity to the application.</span></span> <span data-ttu-id="83875-136">При обработке помещения сообщений в очередь и удаления их из нее следует соблюдать осторожность, чтобы не потерять запросы в случае сбоя.</span><span class="sxs-lookup"><span data-stu-id="83875-136">You must handle queuing and dequeuing safely to avoid losing requests in the event of a failure.</span></span>
- <span data-ttu-id="83875-137">Приложение зависит от дополнительной службы обработки очереди сообщений.</span><span class="sxs-lookup"><span data-stu-id="83875-137">The application takes a dependency on an additional service for the message queue.</span></span>
- <span data-ttu-id="83875-138">Среда обработки должна обладать достаточной масштабируемостью, чтобы обработать ожидаемую рабочую нагрузку и обеспечить соответствие целевым показателям пропускной способности.</span><span class="sxs-lookup"><span data-stu-id="83875-138">The processing environment must be sufficiently scalable to handle the expected workload and meet the required throughput targets.</span></span>
- <span data-ttu-id="83875-139">Хотя такой подход должен повысить общую скорость реагирования, выполнение задач, перемещаемых в серверную часть, может занять больше времени.</span><span class="sxs-lookup"><span data-stu-id="83875-139">While this approach should improve overall responsiveness, the tasks that are moved to the back end may take longer to complete.</span></span> 

## <a name="how-to-detect-the-problem"></a><span data-ttu-id="83875-140">Как определить проблему</span><span class="sxs-lookup"><span data-stu-id="83875-140">How to detect the problem</span></span>

<span data-ttu-id="83875-141">Если внешний интерфейс загружен, при выполнении ресурсоемких задач возникают большие задержки.</span><span class="sxs-lookup"><span data-stu-id="83875-141">Symptoms of a busy front end include high latency when resource-intensive tasks are being performed.</span></span> <span data-ttu-id="83875-142">Пользователи могут сообщить об увеличении времени отклика или ошибках, вызванных завершением времени ожидания служб. Эти сбои также могут возвращать ошибки HTTP 500 (внутренняя ошибка сервера) или HTTP 503 (служба недоступна).</span><span class="sxs-lookup"><span data-stu-id="83875-142">End users are likely to report extended response times or failures caused by services timing out. These failures could also return HTTP 500 (Internal Server) errors or HTTP 503 (Service Unavailable) errors.</span></span> <span data-ttu-id="83875-143">Изучите журналы событий веб-сервера, которые, скорее всего, содержат более подробные сведения о причинах и обстоятельствах возникновения ошибок.</span><span class="sxs-lookup"><span data-stu-id="83875-143">Examine the event logs for the web server, which are likely to contain more detailed information about the causes and circumstances of the errors.</span></span>

<span data-ttu-id="83875-144">Чтобы определить эту проблему, сделайте следующее:</span><span class="sxs-lookup"><span data-stu-id="83875-144">You can perform the following steps to help identify this problem:</span></span>

1. <span data-ttu-id="83875-145">Выполните мониторинг рабочей системы, чтобы определить точки замедления времени отклика.</span><span class="sxs-lookup"><span data-stu-id="83875-145">Perform process monitoring of the production system, to identify points when response times slow down.</span></span>
2. <span data-ttu-id="83875-146">Проверьте данные телеметрии, собранные в этих точках, чтобы определить, выполняемые операции и используемые ресурсы.</span><span class="sxs-lookup"><span data-stu-id="83875-146">Examine the telemetry data captured at these points to determine the mix of operations being performed and the resources being used.</span></span> 
3. <span data-ttu-id="83875-147">Определите связь между увеличением времени отклика и количеством выполняемых в это время операций, а также их сочетанием.</span><span class="sxs-lookup"><span data-stu-id="83875-147">Find any correlations between poor response times and the volumes and combinations of operations that were happening at those times.</span></span>
4. <span data-ttu-id="83875-148">Выполните нагрузочное тестирование каждой подозрительной операции, чтобы определить, какие из них потребляют ресурсы и замедляют выполнение других операций.</span><span class="sxs-lookup"><span data-stu-id="83875-148">Load test each suspected operation to identify which operations are consuming resources and starving other operations.</span></span> 
5. <span data-ttu-id="83875-149">Просмотрите исходный код этих операций, чтобы определить причину избыточного потребления ресурсов.</span><span class="sxs-lookup"><span data-stu-id="83875-149">Review the source code for those operations to determine why they might cause excessive resource consumption.</span></span>

## <a name="example-diagnosis"></a><span data-ttu-id="83875-150">Пример диагностики</span><span class="sxs-lookup"><span data-stu-id="83875-150">Example diagnosis</span></span> 

<span data-ttu-id="83875-151">В следующих разделах эти шаги применяются к примеру приложения, описанному ранее.</span><span class="sxs-lookup"><span data-stu-id="83875-151">The following sections apply these steps to the sample application described earlier.</span></span>

### <a name="identify-points-of-slowdown"></a><span data-ttu-id="83875-152">Определение точек замедления</span><span class="sxs-lookup"><span data-stu-id="83875-152">Identify points of slowdown</span></span>

<span data-ttu-id="83875-153">Выполните инструментирование каждого метода, чтобы отследить продолжительность каждого запроса и потребляемые ресурсы.</span><span class="sxs-lookup"><span data-stu-id="83875-153">Instrument each method to track the duration and resources consumed by each request.</span></span> <span data-ttu-id="83875-154">Затем выполните мониторинг приложения в рабочей среде.</span><span class="sxs-lookup"><span data-stu-id="83875-154">Then monitor the application in production.</span></span> <span data-ttu-id="83875-155">Это может обеспечить полное представление того, как запросы конкурируют друг с другом.</span><span class="sxs-lookup"><span data-stu-id="83875-155">This can provide an overall view of how requests compete with each other.</span></span> <span data-ttu-id="83875-156">Во время стрессовых периодов медленно выполняемые, нуждающиеся в ресурсах запросы, скорее всего, повлияют на другие операции. Это поведение, а также соответствующие проблемы с производительностью можно предотвратить, выполняя мониторинг системы.</span><span class="sxs-lookup"><span data-stu-id="83875-156">During periods of stress, slow-running resource-hungry requests will likely impact other operations, and this behavior can be observed by monitoring the system and noting the drop off in performance.</span></span>

<span data-ttu-id="83875-157">На снимке экрана ниже показана панель мониторинга.</span><span class="sxs-lookup"><span data-stu-id="83875-157">The following image shows a monitoring dashboard.</span></span> <span data-ttu-id="83875-158">(При тестировании мы использовали [AppDynamics].) Изначально в системе наблюдается небольшая нагрузка.</span><span class="sxs-lookup"><span data-stu-id="83875-158">(We used [AppDynamics] for our tests.) Initially, the system has light load.</span></span> <span data-ttu-id="83875-159">Затем пользователи запрашивают метод GET `UserProfile`.</span><span class="sxs-lookup"><span data-stu-id="83875-159">Then users start requesting the `UserProfile` GET method.</span></span> <span data-ttu-id="83875-160">Производительность оставалась довольно хорошей, пока другие пользователи не начали отправлять запросы к методу POST `WorkInFrontEnd`.</span><span class="sxs-lookup"><span data-stu-id="83875-160">The performance is reasonably good until other users start issuing requests to the `WorkInFrontEnd` POST method.</span></span> <span data-ttu-id="83875-161">На этом этапе значительно увеличилось время отклика (первая стрелка).</span><span class="sxs-lookup"><span data-stu-id="83875-161">At that point, response times increase dramatically (first arrow).</span></span> <span data-ttu-id="83875-162">Время отклика улучшилось только после снижения количества запросов к контроллеру `WorkInFrontEnd` (вторая стрелка).</span><span class="sxs-lookup"><span data-stu-id="83875-162">Response times only improve after the volume of requests to the `WorkInFrontEnd` controller diminishes (second arrow).</span></span>

![Панель бизнес-транзакций AppDynamics, на которой отображается время отклика всех запросов при использовании контроллера WorkInFrontEnd][AppDynamics-Transactions-Front-End-Requests]

### <a name="examine-telemetry-data-and-find-correlations"></a><span data-ttu-id="83875-164">Изучение данных телеметрии и поиск связи</span><span class="sxs-lookup"><span data-stu-id="83875-164">Examine telemetry data and find correlations</span></span>

<span data-ttu-id="83875-165">На следующем снимке экрана показаны некоторые метрики, собранные для мониторинга использования ресурсов на притяжении того же периода времени.</span><span class="sxs-lookup"><span data-stu-id="83875-165">The next image shows some of the metrics gathered to monitor resource utilization during the same interval.</span></span> <span data-ttu-id="83875-166">Сначала несколько пользователей вошли в систему.</span><span class="sxs-lookup"><span data-stu-id="83875-166">At first, few users are accessing the system.</span></span> <span data-ttu-id="83875-167">Когда к ней подключается больше пользователей, загрузка ЦП становится очень высокой (100 %).</span><span class="sxs-lookup"><span data-stu-id="83875-167">As more users connect, CPU utilization becomes very high (100%).</span></span> <span data-ttu-id="83875-168">Кроме того, обратите внимание, что с увеличением использования ресурсов ЦП сначала скорость сетевых операций ввода-вывода поднимается.</span><span class="sxs-lookup"><span data-stu-id="83875-168">Also notice that the network I/O rate initially goes up as CPU usage rises.</span></span> <span data-ttu-id="83875-169">Но когда использование ЦП достигает пиковых значений, скорость сетевых операций ввода-вывода снижается.</span><span class="sxs-lookup"><span data-stu-id="83875-169">But once CPU usage peaks, network I/O actually goes down.</span></span> <span data-ttu-id="83875-170">Это связано с тем, что система может обрабатывать только относительно небольшое число запросов, когда ЦП работает на полную мощность.</span><span class="sxs-lookup"><span data-stu-id="83875-170">That's because the system can only handle a relatively small number of requests once the CPU is at capacity.</span></span> <span data-ttu-id="83875-171">Когда пользователи отключаются, нагрузка на ЦП уменьшается.</span><span class="sxs-lookup"><span data-stu-id="83875-171">As users disconnect, the CPU load tails off.</span></span>

![Метрики использования ресурсов сети и ЦП в AppDynamics][AppDynamics-Metrics-Front-End-Requests]

<span data-ttu-id="83875-173">На этом этапе, скорее всего, более тщательно следует проанализировать метод `Post` в контроллере `WorkInFrontEnd`.</span><span class="sxs-lookup"><span data-stu-id="83875-173">At this point, it appears the `Post` method in the `WorkInFrontEnd` controller is a prime candidate for closer examination.</span></span> <span data-ttu-id="83875-174">Чтобы подтвердить эту версию, в управляемой среде нужно провести дальнейшую работу.</span><span class="sxs-lookup"><span data-stu-id="83875-174">Further work in a controlled environment is needed to confirm the hypothesis.</span></span>

### <a name="perform-load-testing"></a><span data-ttu-id="83875-175">Выполнение нагрузочного тестирования</span><span class="sxs-lookup"><span data-stu-id="83875-175">Perform load testing</span></span> 

<span data-ttu-id="83875-176">Следующий шаг — провести тестирования в управляемой среде.</span><span class="sxs-lookup"><span data-stu-id="83875-176">The next step is to perform tests in a controlled environment.</span></span> <span data-ttu-id="83875-177">Например, можно выполнить ряд нагрузочных тестов, которые сначала включают каждый запрос, а затем исключают их, чтобы оценить влияние.</span><span class="sxs-lookup"><span data-stu-id="83875-177">For example, run a series of load tests that include and then omit each request in turn to see the effects.</span></span>

<span data-ttu-id="83875-178">На графике ниже показаны результаты нагрузочного теста, выполненного в такой же облачной службе, что и в предыдущих тестах.</span><span class="sxs-lookup"><span data-stu-id="83875-178">The graph below shows the results of a load test performed against an identical deployment of the cloud service used in the previous tests.</span></span> <span data-ttu-id="83875-179">При тестировании применялась постоянная нагрузка с 500 пользователями, выполняющими операцию `Get` в контроллере `UserProfile`, а также пошаговая нагрузка с пользователями, выполняющими операцию `Post` в контроллере `WorkInFrontEnd`.</span><span class="sxs-lookup"><span data-stu-id="83875-179">The test used a constant load of 500 users performing the `Get` operation in the `UserProfile` controller, along with a step load of users performing the `Post` operation in the `WorkInFrontEnd` controller.</span></span> 

![Начальные результаты нагрузочного теста контроллера WorkInFrontEnd][Initial-Load-Test-Results-Front-End]

<span data-ttu-id="83875-181">Изначально пошаговая нагрузка равна нулю, поэтому только активные пользователи выполняют запросы `UserProfile`.</span><span class="sxs-lookup"><span data-stu-id="83875-181">Initially, the step load is 0, so the only active users are performing the `UserProfile` requests.</span></span> <span data-ttu-id="83875-182">Система может отвечать примерно на 500 запросов в секунду.</span><span class="sxs-lookup"><span data-stu-id="83875-182">The system is able to respond to approximately 500 requests per second.</span></span> <span data-ttu-id="83875-183">Через 60 секунд нагрузку увеличили на 100 дополнительных пользователей, которые начинают отправлять запросы POST к контроллеру `WorkInFrontEnd`.</span><span class="sxs-lookup"><span data-stu-id="83875-183">After 60 seconds, a load of 100 additional users starts sending POST requests to the `WorkInFrontEnd` controller.</span></span> <span data-ttu-id="83875-184">Почти мгновенно рабочая нагрузка, отправленная к контроллеру `UserProfile`, снижается до около 150 запросов в секунду.</span><span class="sxs-lookup"><span data-stu-id="83875-184">Almost immediately, the workload sent to the `UserProfile` controller drops to about 150 requests per second.</span></span> <span data-ttu-id="83875-185">Это связано с функционированием средства выполнения нагрузочных тестов.</span><span class="sxs-lookup"><span data-stu-id="83875-185">This is due to the way the load-test runner functions.</span></span> <span data-ttu-id="83875-186">Он ожидает ответ перед отправкой следующего запроса, поэтому чем больше время ответа, тем ниже частота запросов.</span><span class="sxs-lookup"><span data-stu-id="83875-186">It waits for a response before sending the next request, so the longer it takes to receive a response, the lower the request rate.</span></span>

<span data-ttu-id="83875-187">При увеличении количества пользователей, отправляющих запросы POST к контроллеру `WorkInFrontEnd`, скорость ответа контроллера `UserProfile` продолжает снижаться.</span><span class="sxs-lookup"><span data-stu-id="83875-187">As more users send POST requests to the `WorkInFrontEnd` controller, the response rate of the `UserProfile` controller continues to drop.</span></span> <span data-ttu-id="83875-188">Но обратите внимание, что количество запросов, обрабатываемых контроллером `WorkInFrontEnd`, остается относительно постоянным.</span><span class="sxs-lookup"><span data-stu-id="83875-188">But note that the volume of requests handled by the `WorkInFrontEnd`controller remains relatively constant.</span></span> <span data-ttu-id="83875-189">Насыщенность системы становится очевидной, так как общее число обоих запросов становится постоянным, но имеет минимальный уровень.</span><span class="sxs-lookup"><span data-stu-id="83875-189">The saturation of the system becomes apparent as the overall rate of both requests tends towards a steady but low limit.</span></span>


### <a name="review-the-source-code"></a><span data-ttu-id="83875-190">Просмотр исходного кода</span><span class="sxs-lookup"><span data-stu-id="83875-190">Review the source code</span></span>

<span data-ttu-id="83875-191">Последний шаг — просмотреть исходный код.</span><span class="sxs-lookup"><span data-stu-id="83875-191">The final step is to look at the source code.</span></span> <span data-ttu-id="83875-192">Группе разработчиков стало известно, что выполнение метода `Post` может занять значительное время, поэтому при исходной реализации используется отдельный поток.</span><span class="sxs-lookup"><span data-stu-id="83875-192">The development team was aware that the `Post` method could take a considerable amount of time, which is why the original implementation used a separate thread.</span></span> <span data-ttu-id="83875-193">Это позволило решить насущную проблему, так как метод `Post` не блокирует выполнение длительных задач.</span><span class="sxs-lookup"><span data-stu-id="83875-193">That solved the immediate problem, because the `Post` method did not block waiting for a long-running task to complete.</span></span>

<span data-ttu-id="83875-194">Однако операция, выполняемая этим методом, по-прежнему зависит от ЦП, памяти и других ресурсов.</span><span class="sxs-lookup"><span data-stu-id="83875-194">However, the work performed by this method still consumes CPU, memory, and other resources.</span></span> <span data-ttu-id="83875-195">Если выполнить этот процесс асинхронно, это может фактически снизить производительность, так как пользователи смогут одновременно активировать большое количество этих операций без какого-либо контроля.</span><span class="sxs-lookup"><span data-stu-id="83875-195">Enabling this process to run asynchronously might actually damage performance, as users can trigger a large number of these operations simultaneously, in an uncontrolled manner.</span></span> <span data-ttu-id="83875-196">Число потоков, которое можно запустить на сервере, ограничено.</span><span class="sxs-lookup"><span data-stu-id="83875-196">There is a limit to the number of threads that a server can run.</span></span> <span data-ttu-id="83875-197">При превышении этого предела, когда приложение попытается создать поток, скорее всего, возникнет исключение.</span><span class="sxs-lookup"><span data-stu-id="83875-197">Past this limit, the application is likely to get an exception when it tries to start a new thread.</span></span>

> [!NOTE]
> <span data-ttu-id="83875-198">Это не значит, что следует избегать асинхронных операций.</span><span class="sxs-lookup"><span data-stu-id="83875-198">This doesn't mean you should avoid asynchronous operations.</span></span> <span data-ttu-id="83875-199">В сетевых вызовах рекомендуется применять асинхронные методы с использованием оператора await</span><span class="sxs-lookup"><span data-stu-id="83875-199">Performing an asynchronous await on a network call is a recommended practice.</span></span> <span data-ttu-id="83875-200">(см. антишаблон [синхронных операций ввода-вывода][sync-io]). Проблема здесь в том, что нагружающая ЦП задача может возникнуть в другом потоке.</span><span class="sxs-lookup"><span data-stu-id="83875-200">(See the [Synchronous I/O][sync-io] antipattern.) The problem here is that CPU-intensive work was spawned on another thread.</span></span> 

### <a name="implement-the-solution-and-verify-the-result"></a><span data-ttu-id="83875-201">Реализация решения и проверка результатов</span><span class="sxs-lookup"><span data-stu-id="83875-201">Implement the solution and verify the result</span></span>

<span data-ttu-id="83875-202">На снимке экрана ниже приведены показатели производительности после реализации решения.</span><span class="sxs-lookup"><span data-stu-id="83875-202">The following image shows performance monitoring after the solution was implemented.</span></span> <span data-ttu-id="83875-203">Нагрузка совпадает с приведенной выше, но теперь время ответа контроллера `UserProfile` значительно выше.</span><span class="sxs-lookup"><span data-stu-id="83875-203">The load was similar to that shown earlier, but the response times for the `UserProfile` controller are now much faster.</span></span> <span data-ttu-id="83875-204">За аналогичный период количество запросов выросло с 2759 до 23 565.</span><span class="sxs-lookup"><span data-stu-id="83875-204">The volume of requests increased over the same duration, from 2,759 to 23,565.</span></span> 

![Панель бизнес-транзакций AppDynamics, на которой отображается время отклика всех запросов при использовании контроллера WorkInBackground][AppDynamics-Transactions-Background-Requests]

<span data-ttu-id="83875-206">Обратите внимание, что контроллер `WorkInBackground` также обработал гораздо большее количество запросов.</span><span class="sxs-lookup"><span data-stu-id="83875-206">Note that the `WorkInBackground` controller also handled a much larger volume of requests.</span></span> <span data-ttu-id="83875-207">Однако прямое сравнение в этом случае невозможно сделать, так как операции, выполняемые в этом контроллере, сильно отличаются от исходного кода.</span><span class="sxs-lookup"><span data-stu-id="83875-207">However, you can't make a direct comparison in this case, because the work being performed in this controller is very different from the original code.</span></span> <span data-ttu-id="83875-208">Новая версия просто добавляет запрос в очередь, а не выполняет продолжительное вычисление.</span><span class="sxs-lookup"><span data-stu-id="83875-208">The new version simply queues a request, rather than performing a time consuming calculation.</span></span> <span data-ttu-id="83875-209">Самое главное, что этот метод больше не влияет на производительность всей системы под нагрузкой.</span><span class="sxs-lookup"><span data-stu-id="83875-209">The main point is that this method no longer drags down the entire system under load.</span></span>

<span data-ttu-id="83875-210">Показатели использования ресурсов ЦП и сети также улучшились.</span><span class="sxs-lookup"><span data-stu-id="83875-210">CPU and network utilization also show the improved performance.</span></span> <span data-ttu-id="83875-211">Показатель использования ЦП никогда не достигал 100 %, а количество обработанных сетевых запросов значительно выросло и не снижалось до падения рабочей нагрузки.</span><span class="sxs-lookup"><span data-stu-id="83875-211">The CPU utilization never reached 100%, and the volume of handled network requests was far greater than earlier, and did not tail off until the workload dropped.</span></span>

![Метрики использования ресурсов сети и ЦП контроллера WorkInBackground в AppDynamics][AppDynamics-Metrics-Background-Requests]

<span data-ttu-id="83875-213">На графике ниже показаны результаты нагрузочного теста.</span><span class="sxs-lookup"><span data-stu-id="83875-213">The following graph shows the results of a load test.</span></span> <span data-ttu-id="83875-214">Общее количество обработанных запросов значительно увеличилось по сравнению с предыдущими тестами.</span><span class="sxs-lookup"><span data-stu-id="83875-214">The overall volume of requests serviced is greatly improved compared to the the earlier tests.</span></span>

![Результаты нагрузочного теста контроллера BackgroundImageProcessing][Load-Test-Results-Background]

## <a name="related-guidance"></a><span data-ttu-id="83875-216">Связанные руководства</span><span class="sxs-lookup"><span data-stu-id="83875-216">Related guidance</span></span>

- <span data-ttu-id="83875-217">[Автомасштабирование][autoscaling]</span><span class="sxs-lookup"><span data-stu-id="83875-217">[Autoscaling best practices][autoscaling]</span></span>
- <span data-ttu-id="83875-218">[Фоновые задания][background-jobs]</span><span class="sxs-lookup"><span data-stu-id="83875-218">[Background jobs best practices][background-jobs]</span></span>
- <span data-ttu-id="83875-219">[Шаблон балансировки нагрузки на основе очередей][load-leveling]</span><span class="sxs-lookup"><span data-stu-id="83875-219">[Queue-Based Load Leveling pattern][load-leveling]</span></span>
- <span data-ttu-id="83875-220">[Web Queue Worker architecture style][web-queue-worker] (Стиль архитектуры с применением веб-интерфейса, рабочего потока и очереди)</span><span class="sxs-lookup"><span data-stu-id="83875-220">[Web Queue Worker architecture style][web-queue-worker]</span></span>

[AppDyanamics]: https://www.appdynamics.com/
[autoscaling]: /azure/architecture/best-practices/auto-scaling
[background-jobs]: /azure/architecture/best-practices/background-jobs
[code-sample]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[fullDemonstrationOfSolution]: https://github.com/mspnp/performance-optimization/tree/master/BusyFrontEnd
[load-leveling]: /azure/architecture/patterns/queue-based-load-leveling
[sync-io]: ../synchronous-io/index.md
[web-queue-worker]: /azure/architecture/guide/architecture-styles/web-queue-worker

[WebJobs]: http://www.hanselman.com/blog/IntroducingWindowsAzureWebJobs.aspx
[ComputePartitioning]: https://msdn.microsoft.com/library/dn589773.aspx
[ServiceBusQueues]: https://msdn.microsoft.com/library/azure/hh367516.aspx
[AppDynamics-Transactions-Front-End-Requests]: ./_images/AppDynamicsPerformanceStats.jpg
[AppDynamics-Metrics-Front-End-Requests]: ./_images/AppDynamicsFrontEndMetrics.jpg
[Initial-Load-Test-Results-Front-End]: ./_images/InitialLoadTestResultsFrontEnd.jpg
[AppDynamics-Transactions-Background-Requests]: ./_images/AppDynamicsBackgroundPerformanceStats.jpg
[AppDynamics-Metrics-Background-Requests]: ./_images/AppDynamicsBackgroundMetrics.jpg
[Load-Test-Results-Background]: ./_images/LoadTestResultsBackground.jpg


