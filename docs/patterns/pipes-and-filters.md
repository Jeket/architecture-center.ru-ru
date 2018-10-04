---
title: Каналы и фильтры
description: Задачу, которая требует сложной обработки, можно разбить на ряд отдельных элементов для повторного использования при необходимости.
keywords: Конструктивный шаблон
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- messaging
ms.openlocfilehash: fd616676f9487bdfe1bf23b3d0fec6c65b97a8f4
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429576"
---
# <a name="pipes-and-filters-pattern"></a><span data-ttu-id="8a8f6-104">Шаблон каналов и фильтров</span><span class="sxs-lookup"><span data-stu-id="8a8f6-104">Pipes and Filters pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="8a8f6-105">Разбейте задачу, которая выполняет сложную обработку, на ряд отдельных элементов для повторного использования при необходимости.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-105">Decompose a task that performs complex processing into a series of separate elements that can be reused.</span></span> <span data-ttu-id="8a8f6-106">Так вы сможете повысить производительность, масштабируемость и возможность многократного использования, позволяя развертывать и масштабировать элементы задачи, которые выполняют обработку, независимо друг от друга.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-106">This can improve performance, scalability, and reusability by allowing task elements that perform the processing to be deployed and scaled independently.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="8a8f6-107">Контекст и проблема</span><span class="sxs-lookup"><span data-stu-id="8a8f6-107">Context and problem</span></span>

<span data-ttu-id="8a8f6-108">Приложение должно выполнять различные задачи определенной сложности с информацией, которую оно обрабатывает.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-108">An application is required to perform a variety of tasks of varying complexity on the information that it processes.</span></span> <span data-ttu-id="8a8f6-109">Для осуществления такой обработки в виде неделимого модуля применяется простой, но гибкий способ реализации такого приложения.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-109">A straightforward but inflexible approach to implementing an application is to perform this processing as a monolithic module.</span></span> <span data-ttu-id="8a8f6-110">Однако этот метод, вероятно, сократит возможности для рефакторинга кода, его оптимизации или повторного использования, если части одной и той же обработки потребуются в другом месте приложения.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-110">However, this approach is likely to reduce the opportunities for refactoring the code, optimizing it, or reusing it if parts of the same processing are required elsewhere within the application.</span></span>

<span data-ttu-id="8a8f6-111">На рисунке приведена схема обработки данных с помощью монолитного подхода и сопутствующие проблемы.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-111">The figure illustrates the issues with processing data using the monolithic approach.</span></span> <span data-ttu-id="8a8f6-112">Приложение получает и обрабатывает данные из двух источников.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-112">An application receives and processes data from two sources.</span></span> <span data-ttu-id="8a8f6-113">Данные из каждого источника обрабатываются отдельным модулем, который выполняет ряд задач для преобразования этих данных, прежде чем передавать результат в бизнес-логику приложения.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-113">The data from each source is processed by a separate module that performs a series of tasks to transform this data, before passing the result to the business logic of the application.</span></span>

![Рис. 1. Решение, реализованное с помощью неделимых модулей](./_images/pipes-and-filters-modules.png)

<span data-ttu-id="8a8f6-115">Функции некоторых задач, выполняемых неделимыми модулями, очень похожи, но модули разрабатываются отдельно.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-115">Some of the tasks that the monolithic modules perform are functionally very similar, but the modules have been designed separately.</span></span> <span data-ttu-id="8a8f6-116">Код, который реализует задачи, тесно связан в модуле. Он незначительно или вообще не учитывает задачи масштабируемости и многократного использования.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-116">The code that implements the tasks is closely coupled in a module, and has been developed with little or no thought given to reuse or scalability.</span></span>

<span data-ttu-id="8a8f6-117">Тем не менее, задачи обработки, выполняемые каждым модулем, или требования к развертыванию для каждой задачи, могут изменяться в соответствии с бизнес-требованиями.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-117">However, the processing tasks performed by each module, or the deployment requirements for each task, could change as business requirements are updated.</span></span> <span data-ttu-id="8a8f6-118">Некоторые задачи могут быть ресурсоемкими и более эффективными на мощном оборудовании. В то время как для других задач такие дорогостоящие ресурсы могут не требоваться.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-118">Some tasks might be compute intensive and could benefit from running on powerful hardware, while others might not require such expensive resources.</span></span> <span data-ttu-id="8a8f6-119">Кроме того, в будущем может предполагаться дополнительная обработка или может измениться порядок выполнения задач по обработке.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-119">Also, additional processing might be required in the future, or the order in which the tasks performed by the processing could change.</span></span> <span data-ttu-id="8a8f6-120">Требуется такое решение, которое устранит эти проблемы и увеличит возможность многократного использования кода.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-120">A solution is required that addresses these issues, and increases the possibilities for code reuse.</span></span>

## <a name="solution"></a><span data-ttu-id="8a8f6-121">Решение</span><span class="sxs-lookup"><span data-stu-id="8a8f6-121">Solution</span></span>

<span data-ttu-id="8a8f6-122">Разбейте процесс обработки, требуемый для каждого потока, на ряд отдельных компонентов (или фильтров), каждый из которых выполняет определенную задачу.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-122">Break down the processing required for each stream into a set of separate components (or filters), each performing a single task.</span></span> <span data-ttu-id="8a8f6-123">За счет стандартизации формата данных, которые получает и отправляет каждый компонент, эти фильтры можно объединить в конвейер.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-123">By standardizing the format of the data that each component receives and sends, these filters can be combined together into a pipeline.</span></span> <span data-ttu-id="8a8f6-124">Так вы предотвратите дублирование кода и упростите удаление, замену или интеграцию дополнительных компонентов при изменении требований к обработке.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-124">This helps to avoid duplicating code, and makes it easy to remove, replace, or integrate additional components if the processing requirements change.</span></span> <span data-ttu-id="8a8f6-125">На следующем рисунке показано решение, реализованное с использованием каналов и фильтров.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-125">The next figure shows a solution implemented using pipes and filters.</span></span>

![Рис. 2. Решение, реализованное с использованием каналов и фильтров](./_images/pipes-and-filters-solution.png)


<span data-ttu-id="8a8f6-127">Время, необходимое для обработки одного запроса, зависит от скорости самого медленного фильтра в конвейере.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-127">The time it takes to process a single request depends on the speed of the slowest filter in the pipeline.</span></span> <span data-ttu-id="8a8f6-128">Один или несколько фильтров могут быть сдерживающим фактором, особенно если в потоке из определенного источника данных содержится много запросов.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-128">One or more filters could be a bottleneck, especially if a large number of requests appear in a stream from a particular data source.</span></span> <span data-ttu-id="8a8f6-129">Ключевое преимущество конвейерной структуры заключается в том, что в ней можно параллельно выполнять экземпляры медленных фильтров, позволяя распределить нагрузку и повысить пропускную способность в системе.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-129">A key advantage of the pipeline structure is that it provides opportunities for running parallel instances of slow filters, enabling the system to spread the load and improve throughput.</span></span>

<span data-ttu-id="8a8f6-130">Составляющие конвейер фильтры могут выполняться на разных компьютерах, что позволяет масштабировать их независимо друг от друга и использовать возможности эластичности, которые предоставляются в большинстве облачных сред.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-130">The filters that make up a pipeline can run on different machines, enabling them to be scaled independently and take advantage of the elasticity that many cloud environments provide.</span></span> <span data-ttu-id="8a8f6-131">Ресурсоемкие фильтры могут выполняться на оборудовании с высоким уровнем производительности, а другие, менее ресурсоемкие, могут размещаться на более дешевом стандартном оборудовании.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-131">A filter that is computationally intensive can run on high performance hardware, while other less demanding filters can be hosted on less expensive commodity hardware.</span></span> <span data-ttu-id="8a8f6-132">Фильтры не должны находиться в одном центре обработки данных или географическом расположении. Это позволит каждому элементу в конвейере работать в среде, близкой к необходимым ресурсам.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-132">The filters don't even have to be in the same data center or geographical location, which allows each element in a pipeline to run in an environment that is close to the resources it requires.</span></span>  <span data-ttu-id="8a8f6-133">На следующем рисунке показан пример конвейера для данных из источника 1.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-133">The next figure shows an example applied to the pipeline for the data from Source 1.</span></span>

![Рис. 3. Пример конвейера для данных из источника 1](./_images/pipes-and-filters-load-balancing.png)

<span data-ttu-id="8a8f6-135">Если входные и выходные данные фильтра структурированы в виде потока, обработка для каждого фильтра может выполняться в параллельном режиме.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-135">If the input and output of a filter are structured as a stream, it's possible to perform the processing for each filter in parallel.</span></span> <span data-ttu-id="8a8f6-136">Первый фильтр в конвейере может начать свою работу и вернуть результаты, которые непосредственно передаются следующему фильтру в последовательности, прежде чем первый фильтр завершит свою работу.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-136">The first filter in the pipeline can start its work and output its results, which are passed directly on to the next filter in the sequence before the first filter has completed its work.</span></span>

<span data-ttu-id="8a8f6-137">Еще одно преимущество, предоставляемое этой моделью, — устойчивость.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-137">Another benefit is the resiliency that this model can provide.</span></span> <span data-ttu-id="8a8f6-138">Если произойдет сбой фильтра или компьютер, на котором он выполняется, станет недоступен, конвейер позволит перепланировать операции, выполняемые фильтром и передать их другому экземпляру компонента.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-138">If a filter fails or the machine it's running on is no longer available, the pipeline can reschedule the work that the filter was performing and direct this work to another instance of the component.</span></span> <span data-ttu-id="8a8f6-139">Сбой одного фильтра не обязательно приводит к сбою всего конвейера.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-139">Failure of a single filter doesn't necessarily result in failure of the entire pipeline.</span></span>

<span data-ttu-id="8a8f6-140">Альтернативный подход к реализации распределенных транзакций — использовать шаблон каналов и фильтров в сочетании с [шаблоном компенсирующих транзакций](compensating-transaction.md).</span><span class="sxs-lookup"><span data-stu-id="8a8f6-140">Using the Pipes and Filters pattern in conjunction with the [Compensating Transaction pattern](compensating-transaction.md) is an alternative approach to implementing distributed transactions.</span></span> <span data-ttu-id="8a8f6-141">Распределенные транзакции можно разделить на отдельные компенсируемые задачи, каждую из которых можно реализовать с помощью фильтра, также реализуемого с помощью шаблона компенсирующих транзакций.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-141">A distributed transaction can be broken down into separate, compensable tasks, each of which can be implemented by using a filter that also implements the Compensating Transaction pattern.</span></span> <span data-ttu-id="8a8f6-142">Фильтры в конвейере можно реализовать в виде отдельно размещенных задач, выполняемых в непосредственной близости к данным, которыми они управляют.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-142">The filters in a pipeline can be implemented as separate hosted tasks running close to the data that they maintain.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="8a8f6-143">Проблемы и рекомендации</span><span class="sxs-lookup"><span data-stu-id="8a8f6-143">Issues and considerations</span></span>

<span data-ttu-id="8a8f6-144">При выборе схемы реализации этого шаблона следует учитывать следующие моменты.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-144">You should consider the following points when deciding how to implement this pattern:</span></span>
- <span data-ttu-id="8a8f6-145">**Сложность.**</span><span class="sxs-lookup"><span data-stu-id="8a8f6-145">**Complexity**.</span></span> <span data-ttu-id="8a8f6-146">Повышенная гибкость, которую обеспечивает этот шаблон, может также вызвать сложности, особенно если фильтры в конвейере распределяются между разными серверами.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-146">The increased flexibility that this pattern provides can also introduce complexity, especially if the filters in a pipeline are distributed across different servers.</span></span>

- <span data-ttu-id="8a8f6-147">**Надежность.**</span><span class="sxs-lookup"><span data-stu-id="8a8f6-147">**Reliability**.</span></span> <span data-ttu-id="8a8f6-148">Используйте инфраструктуру, которая гарантирует сохранность данных, передаваемых между фильтрами в конвейере.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-148">Use an infrastructure that ensures that data flowing between filters in a pipeline won't be lost.</span></span>

- <span data-ttu-id="8a8f6-149">**Идемпотентность**.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-149">**Idempotency**.</span></span> <span data-ttu-id="8a8f6-150">Если после получения сообщения происходит сбой фильтра в конвейере и операция переносится на другой экземпляр фильтра, часть операции может быть уже выполнена.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-150">If a filter in a pipeline fails after receiving a message and the work is rescheduled to another instance of the filter, part of the work might have already been completed.</span></span> <span data-ttu-id="8a8f6-151">Если эта операция обновляет некоторые аспекты глобального состояния (например, сведения, хранящиеся в базе данных), это обновление можно повторить.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-151">If this work updates some aspect of the global state (such as information stored in a database), the same update could be repeated.</span></span> <span data-ttu-id="8a8f6-152">Такая же проблема может произойти после передачи результатов фильтра следующему фильтру в конвейере, но до того, как появится сообщение об успешном завершении работы фильтра.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-152">A similar issue might occur if a filter fails after posting its results to the next filter in the pipeline, but before indicating that it's completed its work successfully.</span></span> <span data-ttu-id="8a8f6-153">В этих случаях другой экземпляр фильтра может повторить эти операции, что приведет к тому, что одни и те же результаты будут переданы дважды.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-153">In these cases, the same work could be repeated by another instance of the filter, causing the same results to be posted twice.</span></span> <span data-ttu-id="8a8f6-154">В результате следующие фильтры в конвейере могут дважды обработать одни и те же данные.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-154">This could result in subsequent filters in the pipeline processing the same data twice.</span></span> <span data-ttu-id="8a8f6-155">Поэтому фильтры в конвейере должны разрабатываться идемпотентными.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-155">Therefore filters in a pipeline should be designed to be idempotent.</span></span> <span data-ttu-id="8a8f6-156">Дополнительные сведения см. в описании [шаблонов идемпотентности](https://blog.jonathanoliver.com/idempotency-patterns/) в блоге Джонатана Оливера (Jonathan Oliver).</span><span class="sxs-lookup"><span data-stu-id="8a8f6-156">For more information see [Idempotency Patterns](https://blog.jonathanoliver.com/idempotency-patterns/) on Jonathan Oliver’s blog.</span></span>

- <span data-ttu-id="8a8f6-157">**Повторяющиеся сообщения**.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-157">**Repeated messages**.</span></span> <span data-ttu-id="8a8f6-158">Если сбой фильтра в конвейере происходит после публикации сообщения для следующего этапа конвейера, может запуститься другой экземпляр фильтра и опубликовать копию этого сообщения в конвейере.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-158">If a filter in a pipeline fails after posting a message to the next stage of the pipeline, another instance of the filter might be run, and it'll post a copy of the same message to the pipeline.</span></span> <span data-ttu-id="8a8f6-159">Это может привести к передаче следующему фильтру двух экземпляров одного сообщения.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-159">This could cause two instances of the same message to be passed to the next filter.</span></span> <span data-ttu-id="8a8f6-160">Чтобы избежать этого, конвейер должен обнаруживать и удалять дублирующие сообщения.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-160">To avoid this, the pipeline should detect and eliminate duplicate messages.</span></span>

    >  <span data-ttu-id="8a8f6-161">Если вы реализуете конвейер с использованием очередей сообщений (например, очередей служебной шины Microsoft Azure), в инфраструктуре очередей сообщений может быть предусмотрено автоматическое обнаружение и удаление дубликатов сообщений.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-161">If you're implementing the pipeline by using message queues (such as Microsoft Azure Service Bus queues), the message queuing infrastructure might provide automatic duplicate message detection and removal.</span></span>

- <span data-ttu-id="8a8f6-162">**Контекст и состояние**.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-162">**Context and state**.</span></span> <span data-ttu-id="8a8f6-163">В конвейере каждый фильтр по сути выполняется изолированно. Способ его вызова не должен вызывать предположения.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-163">In a pipeline, each filter essentially runs in isolation and shouldn't make any assumptions about how it was invoked.</span></span> <span data-ttu-id="8a8f6-164">Это означает, что каждому фильтру нужно предоставить достаточный контекст для выполнения операций.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-164">This means that each filter should be provided with sufficient context to perform its work.</span></span> <span data-ttu-id="8a8f6-165">Этот контекст может содержать много информации о состоянии.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-165">This context could include a large amount of state information.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="8a8f6-166">Когда следует использовать этот шаблон</span><span class="sxs-lookup"><span data-stu-id="8a8f6-166">When to use this pattern</span></span>

<span data-ttu-id="8a8f6-167">Используйте этот шаблон в следующих случаях:</span><span class="sxs-lookup"><span data-stu-id="8a8f6-167">Use this pattern when:</span></span>
- <span data-ttu-id="8a8f6-168">Процесс обработки, требуемый приложением, можно легко разделить на ряд независимых этапов.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-168">The processing required by an application can easily be broken down into a set of independent steps.</span></span>

- <span data-ttu-id="8a8f6-169">У этапов обработки, которые выполняются приложением, разные требования к масштабируемости.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-169">The processing steps performed by an application have different scalability requirements.</span></span>

    >  <span data-ttu-id="8a8f6-170">Вы можете сгруппировать фильтры, которые должны масштабироваться в рамках одного процесса.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-170">It's possible to group filters that should scale together in the same process.</span></span> <span data-ttu-id="8a8f6-171">Дополнительные сведения см. в описании [шаблона консолидации вычислительных ресурсов](compute-resource-consolidation.md).</span><span class="sxs-lookup"><span data-stu-id="8a8f6-171">For more information, see the [Compute Resource Consolidation pattern](compute-resource-consolidation.md).</span></span>

- <span data-ttu-id="8a8f6-172">Требуется определенная гибкость, чтобы изменять порядок шагов обработки, выполняемых приложением, или добавлять и удалять шаги.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-172">Flexibility is required to allow reordering of the processing steps performed by an application, or the capability to add and remove steps.</span></span>

- <span data-ttu-id="8a8f6-173">Система будет работать эффективнее, если распределить шаги обработки между несколькими серверами.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-173">The system can benefit from distributing the processing for steps across different servers.</span></span>

- <span data-ttu-id="8a8f6-174">Чтобы свести к минимуму последствия сбоя на шаге при обработке данных, требуется надежное решение.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-174">A reliable solution is required that minimizes the effects of failure in a step while data is being processed.</span></span>

<span data-ttu-id="8a8f6-175">Этот шаблон может оказаться неэффективным в следующих случаях:</span><span class="sxs-lookup"><span data-stu-id="8a8f6-175">This pattern might not be useful when:</span></span>
- <span data-ttu-id="8a8f6-176">Шаги обработки, выполняемые приложением, не являются независимыми или должны выполняться вместе в рамках одной транзакции.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-176">The processing steps performed by an application aren't independent, or they have to be performed together as part of the same transaction.</span></span>

- <span data-ttu-id="8a8f6-177">Объем контекста или сведений о состоянии, необходимых для выполнения шага, делают такой подход неэффективным.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-177">The amount of context or state information required by a step makes this approach inefficient.</span></span> <span data-ttu-id="8a8f6-178">Вместо этого можно сохранить информацию о состоянии в базе данных. Но не используйте эту стратегию, если дополнительная нагрузка на базу данных приведет к большому количеству конфликтов.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-178">It might be possible to persist state information to a database instead, but don't use this strategy if the additional load on the database causes excessive contention.</span></span>

## <a name="example"></a><span data-ttu-id="8a8f6-179">Пример</span><span class="sxs-lookup"><span data-stu-id="8a8f6-179">Example</span></span>

<span data-ttu-id="8a8f6-180">Чтобы предоставить инфраструктуру, необходимую для реализации конвейера, можно использовать последовательность очередей сообщений.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-180">You can use a sequence of message queues to provide the infrastructure required to implement a pipeline.</span></span> <span data-ttu-id="8a8f6-181">В начальную очередь поступают необработанные сообщения.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-181">An initial message queue receives unprocessed messages.</span></span> <span data-ttu-id="8a8f6-182">Компонент, реализованный в виде задачи фильтра, ожидает передачи сообщения в очереди, выполняет свою операцию и затем передает преобразованное сообщение в следующую очередь в последовательности.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-182">A component implemented as a filter task listens for a message on this queue, performs its work, and then posts the transformed message to the next queue in the sequence.</span></span> <span data-ttu-id="8a8f6-183">Другая задача фильтра может ожидать передачи сообщения в этой очереди, обрабатывать их и отправлять результаты в другую очередь. И так пока в последнем сообщении в очереди не появятся полностью преобразованные данные.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-183">Another filter task can listen for messages on this queue, process them, post the results to another queue, and so on until the fully transformed data appears in the final message in the queue.</span></span> <span data-ttu-id="8a8f6-184">На следующем рисунке показана реализация конвейера с использованием очередей сообщений.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-184">The next figure illustrates implementing a pipeline using message queues.</span></span>

![Рис. 4. Реализация конвейера с использованием очередей сообщений](./_images/pipes-and-filters-message-queues.png)


<span data-ttu-id="8a8f6-186">Если вы разрабатываете решение в Azure, можно использовать очереди служебной шины, чтобы создать надежный и масштабируемый механизм организации очереди.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-186">If you're building a solution on Azure you can use Service Bus queues to provide a reliable and scalable queuing mechanism.</span></span> <span data-ttu-id="8a8f6-187">В примере класса `ServiceBusPipeFilter` на языке C# ниже показано, как можно реализовать фильтр, который получает входящие сообщения из очереди, обрабатывает их и передает результаты в другую очередь.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-187">The `ServiceBusPipeFilter` class shown below in C# demonstrates how you can implement a filter that receives input messages from a queue, processes these messages, and posts the results to another queue.</span></span>

>  <span data-ttu-id="8a8f6-188">Класс `ServiceBusPipeFilter` определен в проекте PipesAndFilters.Shared, доступном на сайте [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).</span><span class="sxs-lookup"><span data-stu-id="8a8f6-188">The `ServiceBusPipeFilter` class is defined in the PipesAndFilters.Shared project available from [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).</span></span>

```csharp
public class ServiceBusPipeFilter
{
  ...
  private readonly string inQueuePath;
  private readonly string outQueuePath;
  ...
  private QueueClient inQueue;
  private QueueClient outQueue;
  ...

  public ServiceBusPipeFilter(..., string inQueuePath, string outQueuePath = null)
  {
     ...
     this.inQueuePath = inQueuePath;
     this.outQueuePath = outQueuePath;
  }

  public void Start()
  {
    ...
    // Create the outbound filter queue if it doesn't exist.
    ...
    this.outQueue = QueueClient.CreateFromConnectionString(...);

    ...
    // Create the inbound and outbound queue clients.
    this.inQueue = QueueClient.CreateFromConnectionString(...);
  }

  public void OnPipeFilterMessageAsync(
    Func<BrokeredMessage, Task<BrokeredMessage>> asyncFilterTask, ...)
  {
    ...

    this.inQueue.OnMessageAsync(
      async (msg) =>
    {
      ...
      // Process the filter and send the output to the
      // next queue in the pipeline.
      var outMessage = await asyncFilterTask(msg);

      // Send the message from the filter processor
      // to the next queue in the pipeline.
      if (outQueue != null)
      {
        await outQueue.SendAsync(outMessage);
      }

      // Note: There's a chance that the same message could be sent twice
      // or that a message gets processed by an upstream or downstream
      // filter at the same time.
      // This would happen in a situation where processing of a message was
      // completed, it was sent to the next pipe/queue, and then failed
      // to complete when using the PeekLock method.
      // Idempotent message processing and concurrency should be considered
      // in a real-world implementation.
    },
    options);
  }

  public async Task Close(TimeSpan timespan)
  {
    // Pause the processing threads.
    this.pauseProcessingEvent.Reset();

    // There's no clean approach for waiting for the threads to complete
    // the processing. This example simply stops any new processing, waits
    // for the existing thread to complete, then closes the message pump
    // and finally returns.
    Thread.Sleep(timespan);

    this.inQueue.Close();
    ...
  }

  ...
}
```

<span data-ttu-id="8a8f6-189">Метод `Start` в классе `ServiceBusPipeFilter` позволяет подключиться к паре очередей ввода и вывода, а метод `Close` — отключиться от очереди ввода.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-189">The `Start` method in the `ServiceBusPipeFilter` class connects to a pair of input and output queues, and the `Close` method disconnects from the input queue.</span></span> <span data-ttu-id="8a8f6-190">Метод `OnPipeFilterMessageAsync` выполняет фактическую обработку сообщений, а параметр `asyncFilterTask` для этого метода указывает, какую именно обработку следует выполнить.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-190">The `OnPipeFilterMessageAsync` method performs the actual processing of messages, the `asyncFilterTask` parameter to this method specifies the processing to be performed.</span></span> <span data-ttu-id="8a8f6-191">Метод `OnPipeFilterMessageAsync` ожидает поступления сообщений в очередь ввода, выполняет код, определяемый параметром `asyncFilterTask` для каждого сообщения по мере их поступления, и передает результаты в очередь вывода.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-191">The `OnPipeFilterMessageAsync` method waits for incoming messages on the input queue, runs the code specified by the `asyncFilterTask` parameter over each message as it arrives, and posts the results to the output queue.</span></span> <span data-ttu-id="8a8f6-192">Сами очереди определяются с помощью конструктора.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-192">The queues themselves are specified by the constructor.</span></span>

<span data-ttu-id="8a8f6-193">В примере решения фильтры реализуются в наборе рабочих ролей.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-193">The sample solution implements filters in a set of worker roles.</span></span> <span data-ttu-id="8a8f6-194">Каждую рабочую роль можно масштабировать автономно в зависимости от сложности обработки коммерческих данных или ресурсов, необходимых для обработки.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-194">Each worker role can be scaled independently, depending on the complexity of the business processing that it performs or the resources required for processing.</span></span> <span data-ttu-id="8a8f6-195">Кроме того, можно параллельно выполнять несколько экземпляров каждой рабочей роли, чтобы повысить пропускную способность.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-195">Additionally, multiple instances of each worker role can be run in parallel to improve throughput.</span></span>

<span data-ttu-id="8a8f6-196">В следующем коде показана рабочая роль Azure с именем `PipeFilterARoleEntry`, определенная в проекте PipeFilterA в примере решения.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-196">The following code shows an Azure worker role named `PipeFilterARoleEntry`, defined in the PipeFilterA project in the sample solution.</span></span>

```csharp
public class PipeFilterARoleEntry : RoleEntryPoint
{
  ...
  private ServiceBusPipeFilter pipeFilterA;

  public override bool OnStart()
  {
    ...
    this.pipeFilterA = new ServiceBusPipeFilter(
      ...,
      Constants.QueueAPath,
      Constants.QueueBPath);

    this.pipeFilterA.Start();
    ...
  }

  public override void Run()
  {
    this.pipeFilterA.OnPipeFilterMessageAsync(async (msg) =>
    {
      // Clone the message and update it.
      // Properties set by the broker (Deliver count, enqueue time, ...)
      // aren't cloned and must be copied over if required.
      var newMsg = msg.Clone();

      await Task.Delay(500); // DOING WORK

      Trace.TraceInformation("Filter A processed message:{0} at {1}",
        msg.MessageId, DateTime.UtcNow);

      newMsg.Properties.Add(Constants.FilterAMessageKey, "Complete");

      return newMsg;
    });

    ...
  }

  ...
}
```

<span data-ttu-id="8a8f6-197">Эта роль содержит объект `ServiceBusPipeFilter`.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-197">This role contains a `ServiceBusPipeFilter` object.</span></span> <span data-ttu-id="8a8f6-198">Метод `OnStart` в роли позволяет подключиться к очередям для получения входных сообщений и передачи выходных сообщений (имена очередей определены в классе `Constants`).</span><span class="sxs-lookup"><span data-stu-id="8a8f6-198">The `OnStart` method in the role connects to the queues for receiving input messages and posting output messages (the names of the queues are defined in the `Constants` class).</span></span> <span data-ttu-id="8a8f6-199">Метод `Run` вызывает метод `OnPipeFilterMessagesAsync` для обработки каждого полученного сообщения. (В этом примере обработка имитируется ожиданием в течение короткого периода времени.)</span><span class="sxs-lookup"><span data-stu-id="8a8f6-199">The `Run` method invokes the `OnPipeFilterMessagesAsync` method to perform some processing on each message that's received (in this example, the processing is simulated by waiting for a short period of time).</span></span> <span data-ttu-id="8a8f6-200">По завершении обработки создается новое сообщение, содержащее результаты (в этом случае во входящее сообщение добавлено пользовательское свойство), и это сообщение передается в очередь вывода.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-200">When processing is complete, a new message is constructed containing the results (in this case, the input message has a custom property added), and this message is posted to the output queue.</span></span>

<span data-ttu-id="8a8f6-201">Пример кода содержит еще одну рабочую роль с именем `PipeFilterBRoleEntry` в проекте PipeFilterB.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-201">The sample code contains another worker role named `PipeFilterBRoleEntry` in the PipeFilterB project.</span></span> <span data-ttu-id="8a8f6-202">Эта роль аналогична роли `PipeFilterARoleEntry` за исключением того, что она выполняет другие операции обработки в методе `Run`.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-202">This role is similar to `PipeFilterARoleEntry` except that it performs different processing in the `Run` method.</span></span> <span data-ttu-id="8a8f6-203">В примере решения эти две роли объединяются для создания конвейера, где очередь вывода для роли `PipeFilterARoleEntry` является очередью ввода для роли `PipeFilterBRoleEntry`.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-203">In the example solution, these two roles are combined to construct a pipeline, the output queue for the `PipeFilterARoleEntry` role is the input queue for the `PipeFilterBRoleEntry` role.</span></span>

<span data-ttu-id="8a8f6-204">В примере решения также предоставлены две дополнительные роли с именами `InitialSenderRoleEntry` (в проекте InitialSender) и `FinalReceiverRoleEntry` (в проекте FinalReceiver).</span><span class="sxs-lookup"><span data-stu-id="8a8f6-204">The sample solution also provides two additional roles named `InitialSenderRoleEntry` (in the InitialSender project) and `FinalReceiverRoleEntry` (in the FinalReceiver project).</span></span> <span data-ttu-id="8a8f6-205">Роль `InitialSenderRoleEntry` предоставляет начальное сообщение в конвейере.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-205">The `InitialSenderRoleEntry` role provides the initial message in the pipeline.</span></span> <span data-ttu-id="8a8f6-206">Метод `OnStart` позволяет подключиться к одной очереди, а метод `Run` передает метод в эту очередь.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-206">The `OnStart` method connects to a single queue and the `Run` method posts a method to this queue.</span></span> <span data-ttu-id="8a8f6-207">Эта очередь является очередью ввода, которая используется ролью `PipeFilterARoleEntry`, поэтому при передаче сообщения в нее это сообщение принимается и обрабатывается ролью `PipeFilterARoleEntry`.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-207">This queue is the input queue used by the `PipeFilterARoleEntry` role, so posting a message to it causes the message to be received and processed by the `PipeFilterARoleEntry` role.</span></span> <span data-ttu-id="8a8f6-208">Затем обработанное сообщение передается через роль `PipeFilterBRoleEntry`.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-208">The processed message then passes through the `PipeFilterBRoleEntry` role.</span></span>

<span data-ttu-id="8a8f6-209">Очередь ввода для роли `FinalReceiveRoleEntry` является очередью вывода для роли `PipeFilterBRoleEntry`.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-209">The input queue for the `FinalReceiveRoleEntry` role is the output queue for the `PipeFilterBRoleEntry` role.</span></span> <span data-ttu-id="8a8f6-210">Метод `Run` в роли `FinalReceiveRoleEntry`, показанный ниже, получает сообщение и выполняет некоторые задачи окончательной обработки.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-210">The `Run` method in the `FinalReceiveRoleEntry` role, shown below, receives the message and performs some final processing.</span></span> <span data-ttu-id="8a8f6-211">Затем он записывает значения пользовательских свойств, добавленных фильтрами в конвейере, чтобы выполнить трассировку выходных данных.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-211">Then it writes the values of the custom properties added by the filters in the pipeline to the trace output.</span></span>

```csharp
public class FinalReceiverRoleEntry : RoleEntryPoint
{
  ...
  // Final queue/pipe in the pipeline to process data from.
  private ServiceBusPipeFilter queueFinal;

  public override bool OnStart()
  {
    ...
    // Set up the queue.
    this.queueFinal = new ServiceBusPipeFilter(...,Constants.QueueFinalPath);
    this.queueFinal.Start();
    ...
  }

  public override void Run()
  {
    this.queueFinal.OnPipeFilterMessageAsync(
      async (msg) =>
      {
        await Task.Delay(500); // DOING WORK

        // The pipeline message was received.
        Trace.TraceInformation(
          "Pipeline Message Complete - FilterA:{0} FilterB:{1}",
          msg.Properties[Constants.FilterAMessageKey],
          msg.Properties[Constants.FilterBMessageKey]);

        return null;
      });
    ...
  }

  ...
}
```

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="8a8f6-212">Связанные шаблоны и рекомендации</span><span class="sxs-lookup"><span data-stu-id="8a8f6-212">Related patterns and guidance</span></span>

<span data-ttu-id="8a8f6-213">При реализации этого шаблона следует принять во внимание следующие шаблоны и рекомендации.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-213">The following patterns and guidance might also be relevant when implementing this pattern:</span></span>
- <span data-ttu-id="8a8f6-214">Пример, демонстрирующий этот шаблон, можно найти на сайте [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).</span><span class="sxs-lookup"><span data-stu-id="8a8f6-214">A sample that demonstrates this pattern is available on [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/pipes-and-filters).</span></span>
- <span data-ttu-id="8a8f6-215">[Шаблон конкурирующих потребителей](competing-consumers.md).</span><span class="sxs-lookup"><span data-stu-id="8a8f6-215">[Competing Consumers pattern](competing-consumers.md).</span></span> <span data-ttu-id="8a8f6-216">Конвейер может содержать несколько экземпляров одного или нескольких фильтров.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-216">A pipeline can contain multiple instances of one or more filters.</span></span> <span data-ttu-id="8a8f6-217">Этот метод подходит, чтобы параллельно выполнять экземпляры медленных фильтров, позволяя распределить нагрузку и повысить пропускную способность в системе.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-217">This approach is useful for running parallel instances of slow filters, enabling the system to spread the load and improve throughput.</span></span> <span data-ttu-id="8a8f6-218">Каждый экземпляр фильтра будет конкурировать за входные данные с другими экземплярами. Два экземпляра фильтра не должны обрабатывать одни и те же данные.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-218">Each instance of a filter will compete for input with the other instances, two instances of a filter shouldn't be able to process the same data.</span></span> <span data-ttu-id="8a8f6-219">По ссылке выше предоставляется объяснение этого подхода.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-219">Provides an explanation of this approach.</span></span>
- <span data-ttu-id="8a8f6-220">[Шаблон консолидации вычислительных ресурсов](compute-resource-consolidation.md).</span><span class="sxs-lookup"><span data-stu-id="8a8f6-220">[Compute Resource Consolidation pattern](compute-resource-consolidation.md).</span></span> <span data-ttu-id="8a8f6-221">Можно сгруппировать фильтры, которые должны масштабироваться вместе в рамках одного процесса.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-221">It might be possible to group filters that should scale together into the same process.</span></span> <span data-ttu-id="8a8f6-222">По ссылке выше предоставляются дополнительные сведения о преимуществах и недостатках реализации этой стратегии.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-222">Provides more information about the benefits and tradeoffs of this strategy.</span></span>
- <span data-ttu-id="8a8f6-223">[Шаблон компенсирующих транзакций](compensating-transaction.md).</span><span class="sxs-lookup"><span data-stu-id="8a8f6-223">[Compensating Transaction pattern](compensating-transaction.md).</span></span> <span data-ttu-id="8a8f6-224">Фильтр можно реализовать в виде операции с возможностью отмены или с компенсирующей операцией, которая восстанавливает предыдущее состояние в случае сбоя.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-224">A filter can be implemented as an operation that can be reversed, or that has a compensating operation that restores the state to a previous version in the event of a failure.</span></span> <span data-ttu-id="8a8f6-225">По ссылке выше объясняется, как реализовать этот шаблон, чтобы обеспечить итоговую согласованность или добиться ее.</span><span class="sxs-lookup"><span data-stu-id="8a8f6-225">Explains how this can be implemented to maintain or achieve eventual consistency.</span></span>
- <span data-ttu-id="8a8f6-226">[Шаблоны идемпотентности](https://blog.jonathanoliver.com/idempotency-patterns/) в блоге Джонатана Оливера (Jonathan Oliver).</span><span class="sxs-lookup"><span data-stu-id="8a8f6-226">[Idempotency Patterns](https://blog.jonathanoliver.com/idempotency-patterns/) on Jonathan Oliver’s blog.</span></span>
