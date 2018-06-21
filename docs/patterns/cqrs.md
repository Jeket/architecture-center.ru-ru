---
title: CQRS
description: Вы можете разделить интерфейсы для операций считывания и записи данных.
keywords: Конструктивный шаблон
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- data-management
- design-implementation
- performance-scalability
ms.openlocfilehash: c2832aa806909c6f0aab8b6345ffb8162eb59903
ms.sourcegitcommit: d08f6ee27e1e8a623aeee32d298e616bc9bb87ff
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/07/2018
ms.locfileid: "33811055"
---
# <a name="command-and-query-responsibility-segregation-cqrs-pattern"></a><span data-ttu-id="15235-104">Шаблон CQRS</span><span class="sxs-lookup"><span data-stu-id="15235-104">Command and Query Responsibility Segregation (CQRS) pattern</span></span>

[!INCLUDE [header](../_includes/header.md)]

<span data-ttu-id="15235-105">Вы можете разделить интерфейсы для операций считывания и записи данных.</span><span class="sxs-lookup"><span data-stu-id="15235-105">Segregate operations that read data from operations that update data by using separate interfaces.</span></span> <span data-ttu-id="15235-106">Так вы повысите производительность, масштабируемость и безопасность.</span><span class="sxs-lookup"><span data-stu-id="15235-106">This can maximize performance, scalability, and security.</span></span> <span data-ttu-id="15235-107">Также это упростит процессы, связанные с развитием системы, благодаря дополнительной гибкости, и позволит избежать конфликтов слияния на уровне домена, вызываемых командами обновления.</span><span class="sxs-lookup"><span data-stu-id="15235-107">Supports the evolution of the system over time through higher flexibility, and prevents update commands from causing merge conflicts at the domain level.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="15235-108">Контекст и проблема</span><span class="sxs-lookup"><span data-stu-id="15235-108">Context and problem</span></span>

<span data-ttu-id="15235-109">В привычных системах управления данными обновление данных (команды) и получение данных (запросы) выполняются с одним набором сущностей в едином хранилище.</span><span class="sxs-lookup"><span data-stu-id="15235-109">In traditional data management systems, both commands (updates to the data) and queries (requests for data) are executed against the same set of entities in a single data repository.</span></span> <span data-ttu-id="15235-110">Эти сущности могут быть представлены набором строк в одной или нескольких таблицах в реляционной базе данных, например SQL Server.</span><span class="sxs-lookup"><span data-stu-id="15235-110">These entities can be a subset of the rows in one or more tables in a relational database such as SQL Server.</span></span>

<span data-ttu-id="15235-111">Обычно в таких системах все операции создания, чтения, обновления и удаления (CRUD) применяются к одному и тому же представлению этой сущности.</span><span class="sxs-lookup"><span data-stu-id="15235-111">Typically in these systems, all create, read, update, and delete (CRUD) operations are applied to the same representation of the entity.</span></span> <span data-ttu-id="15235-112">Например, объект передачи данных (DTO), представляющий клиента, извлекается из хранилища данных на уровне доступа к данным (DAL) и отображается на экране.</span><span class="sxs-lookup"><span data-stu-id="15235-112">For example, a data transfer object (DTO) representing a customer is retrieved from the data store by the data access layer (DAL) and displayed on the screen.</span></span> <span data-ttu-id="15235-113">Пользователь обновляет несколько полей в DTO (возможно, с использованием привязки данных), и DAL помещает этот DTO обратно в хранилище данных.</span><span class="sxs-lookup"><span data-stu-id="15235-113">A user updates some fields of the DTO (perhaps through data binding) and the DTO is then saved back in the data store by the DAL.</span></span> <span data-ttu-id="15235-114">Один и тот же объект DTO используется для операций чтения и записи.</span><span class="sxs-lookup"><span data-stu-id="15235-114">The same DTO is used for both the read and write operations.</span></span> <span data-ttu-id="15235-115">На этой схеме представлена стандартная архитектура CRUD.</span><span class="sxs-lookup"><span data-stu-id="15235-115">The figure illustrates a traditional CRUD architecture.</span></span>

![Стандартная архитектура CRUD](./_images/command-and-query-responsibility-segregation-cqrs-tradition-crud.png)

<span data-ttu-id="15235-117">Традиционные схемы реализации CRUD хорошо работают, только когда к операциям с данными применяется ограниченная бизнес-логика.</span><span class="sxs-lookup"><span data-stu-id="15235-117">Traditional CRUD designs work well when only limited business logic is applied to the data operations.</span></span> <span data-ttu-id="15235-118">Механизмы шаблонов, реализованные в средствах разработки, позволяют быстро создать код для доступа к данным, дорабатывая его по мере необходимости.</span><span class="sxs-lookup"><span data-stu-id="15235-118">Scaffold mechanisms provided by development tools can create data access code very quickly, which can then be customized as required.</span></span>

<span data-ttu-id="15235-119">Но такой подход имеет и некоторые недостатки.</span><span class="sxs-lookup"><span data-stu-id="15235-119">However, the traditional CRUD approach has some disadvantages:</span></span>

- <span data-ttu-id="15235-120">Часто он приводит к несоответствию между представлениями данных для чтения и записи. Например, в объектах могут существовать дополнительные столбцы или свойства, которые важно правильно обновлять, даже когда они не участвуют в операции.</span><span class="sxs-lookup"><span data-stu-id="15235-120">It often means that there's a mismatch between the read and write representations of the data, such as additional columns or properties that must be updated correctly even though they aren't required as part of an operation.</span></span>

- <span data-ttu-id="15235-121">Так же он может привести к конфликту блокировки, если несколько субъектов одновременно работают с одним набором данных в хранилище для совместной работы.</span><span class="sxs-lookup"><span data-stu-id="15235-121">It risks data contention when records are locked in the data store in a collaborative domain, where multiple actors operate in parallel on the same set of data.</span></span> <span data-ttu-id="15235-122">Кроме того, могут возникать конфликты обновления, если выполняются одновременные обновления с оптимистической блокировкой.</span><span class="sxs-lookup"><span data-stu-id="15235-122">Or update conflicts caused by concurrent updates when optimistic locking is used.</span></span> <span data-ttu-id="15235-123">Эти риски постоянно увеличиваются с ростом сложности системы и нагрузки на нее.</span><span class="sxs-lookup"><span data-stu-id="15235-123">These risks increase as the complexity and throughput of the system grows.</span></span> <span data-ttu-id="15235-124">Кроме того, традиционный подход может негативно влиять на производительность из-за высокой нагрузки на уровень доступа к данным в хранилище данных и высокой сложности запросов для получения данных.</span><span class="sxs-lookup"><span data-stu-id="15235-124">In addition, the traditional approach can have a negative effect on performance due to load on the data store and data access layer, and the complexity of queries required to retrieve information.</span></span>

- <span data-ttu-id="15235-125">Управление безопасностью и разрешениями — сложные процедуры, так как каждая сущность может выполнять операции чтения и записи, а следовательно есть риск предоставить данные в неправильном контексте.</span><span class="sxs-lookup"><span data-stu-id="15235-125">It can make managing security and permissions more complex because each entity is subject to both read and write operations, which might expose data in the wrong context.</span></span>

> <span data-ttu-id="15235-126">Чтобы лучше понять ограничения, связанные с архитектурой CRUD, ознакомьтесь с [этой](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/) записью блога.</span><span class="sxs-lookup"><span data-stu-id="15235-126">For a deeper understanding of the limits of the CRUD approach see [CRUD, Only When You Can Afford It](https://blogs.msdn.microsoft.com/maarten_mullender/2004/07/23/crud-only-when-you-can-afford-it-revisited/).</span></span>

## <a name="solution"></a><span data-ttu-id="15235-127">Решение</span><span class="sxs-lookup"><span data-stu-id="15235-127">Solution</span></span>

<span data-ttu-id="15235-128">Шаблон CQRS разделяет операции считывания данных (запросы) и операции обновления данных (команды), используя разные интерфейсы.</span><span class="sxs-lookup"><span data-stu-id="15235-128">Command and Query Responsibility Segregation (CQRS) is a pattern that segregates the operations that read data (queries) from the operations that update data (commands) by using separate interfaces.</span></span> <span data-ttu-id="15235-129">Это означает, что для запросов и обновлений также используются разные модели данных.</span><span class="sxs-lookup"><span data-stu-id="15235-129">This means that the data models used for querying and updates are different.</span></span> <span data-ttu-id="15235-130">Так вы можете изолировать эти модели, как показано на следующей схеме, хотя это и не обязательно.</span><span class="sxs-lookup"><span data-stu-id="15235-130">The models can then be isolated, as shown in the following figure, although that's not an absolute requirement.</span></span>

![Базовая архитектура CQRS](./_images/command-and-query-responsibility-segregation-cqrs-basic.png)

<span data-ttu-id="15235-132">В отличие от единой модели в системах на базе CRUD, здесь применяются разные модели данных для запросов и обновлений, что упрощает проектирование и реализацию систем CQRS.</span><span class="sxs-lookup"><span data-stu-id="15235-132">Compared to the single data model used in CRUD-based systems, the use of separate query and update models for the data in CQRS-based systems simplifies design and implementation.</span></span> <span data-ttu-id="15235-133">Единственный недостаток этого подхода по сравнению с CRUD заключается в том, что для CQRS невозможно автоматически создать код с помощью встроенного механизма шаблонов.</span><span class="sxs-lookup"><span data-stu-id="15235-133">However, one disadvantage is that unlike CRUD designs, CQRS code can't automatically be generated using scaffold mechanisms.</span></span>

<span data-ttu-id="15235-134">Модель запросов для чтения данных и модель обновления для записи данных могут обращаться к одному физическому хранилищу, например используя представления SQL или оперативно создавая проекции.</span><span class="sxs-lookup"><span data-stu-id="15235-134">The query model for reading data and the update model for writing data can access the same physical store, perhaps by using SQL views or by generating projections on the fly.</span></span> <span data-ttu-id="15235-135">Но чаще эти данные разносятся в разные физические хранилища, как показано на следующем рисунке, что повышает производительность, масштабируемость и безопасность.</span><span class="sxs-lookup"><span data-stu-id="15235-135">However, it's common to separate the data into different physical stores to maximize performance, scalability, and security, as shown in the next figure.</span></span>

![Архитектура CQRS с раздельными хранилищами для чтения и записи](./_images/command-and-query-responsibility-segregation-cqrs-separate-stores.png)

<span data-ttu-id="15235-137">В качестве хранилища для чтения можно использовать реплику хранилища для записи с доступом на чтение. Более того, это могут быть хранилища с совершенно разными структурами данных.</span><span class="sxs-lookup"><span data-stu-id="15235-137">The read store can be a read-only replica of the write store, or the read and write stores can have a different structure altogether.</span></span> <span data-ttu-id="15235-138">Создав несколько реплик хранилища для чтения, вы сможете значительно увеличить производительность обработки запросов и скорость отклика пользовательского интерфейса приложения. Это особенно удобно в распределенных сценариях, где реплики с доступом на чтение можно разместить рядом с экземплярами приложения.</span><span class="sxs-lookup"><span data-stu-id="15235-138">Using multiple read-only replicas of the read store can greatly increase query performance and application UI responsiveness, especially in distributed scenarios where read-only replicas are located close to the application instances.</span></span> <span data-ttu-id="15235-139">Некоторые системы баз данных (например, SQL Server) предоставляют дополнительные возможности для повышения доступности, например реплики для отработки отказа.</span><span class="sxs-lookup"><span data-stu-id="15235-139">Some database systems (SQL Server) provide additional features such as failover replicas to maximize availability.</span></span>

<span data-ttu-id="15235-140">Разделение хранилищ для чтения и записи позволяет правильно масштабировать каждое из них в соответствии с нагрузкой.</span><span class="sxs-lookup"><span data-stu-id="15235-140">Separation of the read and write stores also allows each to be scaled appropriately to match the load.</span></span> <span data-ttu-id="15235-141">Традиционно нагрузка на хранилища для чтения намного выше, чем на хранилища для записи.</span><span class="sxs-lookup"><span data-stu-id="15235-141">For example, read stores typically encounter a much higher load than write stores.</span></span>

<span data-ttu-id="15235-142">Если модель запросов или чтения содержит денормализованные данные (см. описание [шаблона материализованного представления](materialized-view.md)), максимальной производительности чтения можно добиться при считывании в приложении данных для каждого представления или при запросе данных в системе.</span><span class="sxs-lookup"><span data-stu-id="15235-142">When the query/read model contains denormalized data (see [Materialized View pattern](materialized-view.md)), performance is maximized when reading data for each of the views in an application or when querying the data in the system.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="15235-143">Проблемы и рекомендации</span><span class="sxs-lookup"><span data-stu-id="15235-143">Issues and considerations</span></span>

<span data-ttu-id="15235-144">При принятии решения о реализации этого шаблона необходимо учитывать следующие моменты.</span><span class="sxs-lookup"><span data-stu-id="15235-144">Consider the following points when deciding how to implement this pattern:</span></span>

- <span data-ttu-id="15235-145">Разнеся данные для чтения и записи в разные физические хранилища, вы сможете повысить производительность и безопасность системы, но усложните ее с точки зрения обеспечения устойчивости и итоговой согласованности.</span><span class="sxs-lookup"><span data-stu-id="15235-145">Dividing the data store into separate physical stores for read and write operations can increase the performance and security of a system, but it can add complexity in terms of resiliency and eventual consistency.</span></span> <span data-ttu-id="15235-146">Хранилище для модели чтения необходимо обновлять с учетом изменений в хранилище для модели записи. Иногда сложно обнаружить, что запрос пользователя основан на устаревших данных, и в такой ситуации операцию выполнить невозможно.</span><span class="sxs-lookup"><span data-stu-id="15235-146">The read model store must be updated to reflect changes to the write model store, and it can be difficult to detect when a user has issued a request based on stale read data, which means that the operation can't be completed.</span></span>

    > <span data-ttu-id="15235-147">Концепция итоговой согласованности описана в [руководстве по согласованности данных](https://msdn.microsoft.com/library/dn589800.aspx).</span><span class="sxs-lookup"><span data-stu-id="15235-147">For a description of eventual consistency see the [Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span>

- <span data-ttu-id="15235-148">Попробуйте применить подход CQRS в ограниченных разделах системы, где эффект от него будет заметным.</span><span class="sxs-lookup"><span data-stu-id="15235-148">Consider applying CQRS to limited sections of your system where it will be most valuable.</span></span>

- <span data-ttu-id="15235-149">Чтобы обеспечить итоговую согласованность, традиционно шаблон CQRS дополняется шаблоном источников событий. В такой архитектуре модель записи данных сохраняет поток событий, несущий информацию о каждой команде, в хранилище постоянного пополнения.</span><span class="sxs-lookup"><span data-stu-id="15235-149">A typical approach to deploying eventual consistency is to use event sourcing in conjunction with CQRS so that the write model is an append-only stream of events driven by execution of commands.</span></span> <span data-ttu-id="15235-150">Затем на основе этих событий обновляются материализованные представления в модели для чтения.</span><span class="sxs-lookup"><span data-stu-id="15235-150">These events are used to update materialized views that act as the read model.</span></span> <span data-ttu-id="15235-151">Дополнительные сведения см. в разделе о [применении источников событий совместно с CQRS](/azure/architecture/patterns/cqrs#event-sourcing-and-cqrs).</span><span class="sxs-lookup"><span data-stu-id="15235-151">For more information see [Event Sourcing and CQRS](/azure/architecture/patterns/cqrs#event-sourcing-and-cqrs).</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="15235-152">Когда следует использовать этот шаблон</span><span class="sxs-lookup"><span data-stu-id="15235-152">When to use this pattern</span></span>

<span data-ttu-id="15235-153">Используйте этот шаблон в следующих ситуациях:</span><span class="sxs-lookup"><span data-stu-id="15235-153">Use this pattern in the following situations:</span></span>

- <span data-ttu-id="15235-154">В предметных областях с режимом совместной работы, где с одними и теми же данными одновременно выполняется множество операций.</span><span class="sxs-lookup"><span data-stu-id="15235-154">Collaborative domains where multiple operations are performed in parallel on the same data.</span></span> <span data-ttu-id="15235-155">CQRS позволяет определить команды с высокой степенью детализации, чтобы минимизировать риск конфликтов слияния на уровне предметной области (при этом команды должны устранять все возникающие конфликты) даже при обновлении данных одного типа.</span><span class="sxs-lookup"><span data-stu-id="15235-155">CQRS allows you to define commands with enough granularity to minimize merge conflicts at the domain level (any conflicts that do arise can be merged by the command), even when updating what appears to be the same type of data.</span></span>

- <span data-ttu-id="15235-156">В пользовательских интерфейсах на основе задач, где пользователи выполняют сложный процесс в виде последовательности шагов или применяется сложная модель предметной области.</span><span class="sxs-lookup"><span data-stu-id="15235-156">Task-based user interfaces where users are guided through a complex process as a series of steps or with complex domain models.</span></span> <span data-ttu-id="15235-157">Также он будет полезен для команд, которые уже знакомы с методами разработки на основе предметной области (DDD).</span><span class="sxs-lookup"><span data-stu-id="15235-157">Also, useful for teams already familiar with domain-driven design (DDD) techniques.</span></span> <span data-ttu-id="15235-158">Модель записи содержит полный стек обработки команд и реализует бизнес-логику, проверку входных данных и проверку бизнес-требований, чтобы гарантировать постоянную и полную согласованность модели записи по всем агрегатам (так называются кластеры связанных объектов, рассматриваемые как единый блок при изменении данных).</span><span class="sxs-lookup"><span data-stu-id="15235-158">The write model has a full command-processing stack with business logic, input validation, and business validation to ensure that everything is always consistent for each of the aggregates (each cluster of associated objects treated as a unit for data changes) in the write model.</span></span> <span data-ttu-id="15235-159">Модель чтения не использует бизнес-логику или проверку данных, а просто возвращает DTO для модели представления.</span><span class="sxs-lookup"><span data-stu-id="15235-159">The read model has no business logic or validation stack and just returns a DTO for use in a view model.</span></span> <span data-ttu-id="15235-160">Для моделей чтения и записи реализована итоговая согласованность.</span><span class="sxs-lookup"><span data-stu-id="15235-160">The read model is eventually consistent with the write model.</span></span>

- <span data-ttu-id="15235-161">В сценариях, где производительность чтения и записи данных нужно настраивать отдельно, особенно при очень высоком соотношении операций чтения и записи и необходимости горизонтального масштабирования.</span><span class="sxs-lookup"><span data-stu-id="15235-161">Scenarios where performance of data reads must be fine tuned separately from performance of data writes, especially when the read/write ratio is very high, and when horizontal scaling is required.</span></span> <span data-ttu-id="15235-162">Например, во многих системах число операций чтения во много раз превышает число операций записи.</span><span class="sxs-lookup"><span data-stu-id="15235-162">For example, in many systems the number of read operations is many times greater than the number of write operations.</span></span> <span data-ttu-id="15235-163">Чтобы учесть такую разницу, вы можете масштабировать модель чтения, сохраняя небольшое число экземпляров для модели записи.</span><span class="sxs-lookup"><span data-stu-id="15235-163">To accommodate this, consider scaling out the read model, but running the write model on only one or a few instances.</span></span> <span data-ttu-id="15235-164">Ограничение числа экземпляров модели записи помогает снизить риск конфликтов слияния.</span><span class="sxs-lookup"><span data-stu-id="15235-164">A small number of write model instances also helps to minimize the occurrence of merge conflicts.</span></span>

- <span data-ttu-id="15235-165">В сценариях, где отдельная команда разработчиков работает над сложной моделью предметной области в рамках модели записи, позволяя другой команде направить усилия на модель чтения и пользовательские интерфейсы.</span><span class="sxs-lookup"><span data-stu-id="15235-165">Scenarios where one team of developers can focus on the complex domain model that is part of the write model, and another team can focus on the read model and the user interfaces.</span></span>

- <span data-ttu-id="15235-166">Если ожидается, что система будет со временем развиваться с разделением моделей, или в ней будут часто меняться бизнес-правила.</span><span class="sxs-lookup"><span data-stu-id="15235-166">Scenarios where the system is expected to evolve over time and might contain multiple versions of the model, or where business rules change regularly.</span></span>

- <span data-ttu-id="15235-167">При интеграции с другими системами, особенно с применением модели источников событий, что позволит при временном сбое одной из подсистем сохранить доступность остальных.</span><span class="sxs-lookup"><span data-stu-id="15235-167">Integration with other systems, especially in combination with event sourcing, where the temporal failure of one subsystem shouldn't affect the availability of the others.</span></span>

<span data-ttu-id="15235-168">Этот шаблон не следует использовать в следующих сценариях:</span><span class="sxs-lookup"><span data-stu-id="15235-168">This pattern isn't recommended in the following situations:</span></span>

- <span data-ttu-id="15235-169">Если предметная область или бизнес-правила достаточно просты.</span><span class="sxs-lookup"><span data-stu-id="15235-169">Where the domain or the business rules are simple.</span></span>

- <span data-ttu-id="15235-170">Если подход CRUD удовлетворяет всем требованиям к пользовательскому интерфейсу и операциям доступа к данным.</span><span class="sxs-lookup"><span data-stu-id="15235-170">Where a simple CRUD-style user interface and the related data access operations are sufficient.</span></span>

- <span data-ttu-id="15235-171">Для всех операций в пределах системы.</span><span class="sxs-lookup"><span data-stu-id="15235-171">For implementation across the whole system.</span></span> <span data-ttu-id="15235-172">В некоторых компонентах сценария управления данными подход CQRS может быть полезным, но существенно усложнит систему без достаточных для того оснований.</span><span class="sxs-lookup"><span data-stu-id="15235-172">There are specific components of an overall data management scenario where CQRS can be useful, but it can add considerable and unnecessary complexity when it isn't required.</span></span>

## <a name="event-sourcing-and-cqrs"></a><span data-ttu-id="15235-173">Использование источников событий совместно с CQRS</span><span class="sxs-lookup"><span data-stu-id="15235-173">Event Sourcing and CQRS</span></span>

<span data-ttu-id="15235-174">Шаблон CQRS часто используется в сочетании с шаблоном источников событий.</span><span class="sxs-lookup"><span data-stu-id="15235-174">The CQRS pattern is often used along with the Event Sourcing pattern.</span></span> <span data-ttu-id="15235-175">Системы на основе CQRS используют разные модели данных для чтения и записи, каждая из которых оптимизируется для выполнения соответствующих задач, а данные могут размещаться в разных физических хранилищах.</span><span class="sxs-lookup"><span data-stu-id="15235-175">CQRS-based systems use separate read and write data models, each tailored to relevant tasks and often located in physically separate stores.</span></span> <span data-ttu-id="15235-176">Используемая вместе с шаблоном [источников событий](event-sourcing.md), модель записи сохраняет данные о событиях и является официальным источником данных.</span><span class="sxs-lookup"><span data-stu-id="15235-176">When used with the [Event Sourcing](event-sourcing.md) pattern, the store of events is the write model, and is the official source of information.</span></span> <span data-ttu-id="15235-177">Модель чтения в системах на базе CQRS обычно предоставляет материализованные представления данных, часто с высоким уровнем денормализации.</span><span class="sxs-lookup"><span data-stu-id="15235-177">The read model of a CQRS-based system provides materialized views of the data, typically as highly denormalized views.</span></span> <span data-ttu-id="15235-178">Эти представления создаются с учетом конкретных интерфейсов приложения и требований к отображению, что позволяет максимизировать производительность интерфейсов и запросов.</span><span class="sxs-lookup"><span data-stu-id="15235-178">These views are tailored to the interfaces and display requirements of the application, which helps to maximize both display and query performance.</span></span>

<span data-ttu-id="15235-179">Если в качестве хранилища для записи используется поток событий, а не данные на определенный момент времени, можно избежать конфликтов при обновлении агрегатов, максимально повысить производительность и масштабируемость.</span><span class="sxs-lookup"><span data-stu-id="15235-179">Using the stream of events as the write store, rather than the actual data at a point in time, avoids update conflicts on a single aggregate and maximizes performance and scalability.</span></span> <span data-ttu-id="15235-180">Также эти события позволят в асинхронном режиме создавать материализованные представления данных для заполнения хранилища для чтения.</span><span class="sxs-lookup"><span data-stu-id="15235-180">The events can be used to asynchronously generate materialized views of the data that are used to populate the read store.</span></span>

<span data-ttu-id="15235-181">Так как официальным источником информации являются хранилища событий, в случае изменения системы или модели чтения можно просто удалить все материализованные представления и заново воспроизвести предыдущие события, чтобы создать новое представление текущего состояния.</span><span class="sxs-lookup"><span data-stu-id="15235-181">Because the event store is the official source of information, it is possible to delete the materialized views and replay all past events to create a new representation of the current state when the system evolves, or when the read model must change.</span></span> <span data-ttu-id="15235-182">Материализованные представления по сути выполняют роль устойчивого кэша данных с доступом только на чтение.</span><span class="sxs-lookup"><span data-stu-id="15235-182">The materialized views are in effect a durable read-only cache of the data.</span></span>

<span data-ttu-id="15235-183">Если вы используете шаблоны CQRS и источников событий, учитывайте следующее:</span><span class="sxs-lookup"><span data-stu-id="15235-183">When using CQRS combined with the Event Sourcing pattern, consider the following:</span></span>

- <span data-ttu-id="15235-184">Как в любой системе с разделением хранилищ для записи и чтения, здесь возможна только итоговая согласованность.</span><span class="sxs-lookup"><span data-stu-id="15235-184">As with any system where the write and read stores are separate, systems based on this pattern are only eventually consistent.</span></span> <span data-ttu-id="15235-185">Между созданием события и обновлением данных в хранилище всегда будет некоторая задержка.</span><span class="sxs-lookup"><span data-stu-id="15235-185">There will be some delay between the event being generated and the data store being updated.</span></span>

- <span data-ttu-id="15235-186">Этот шаблон повышает сложность системы, так как в нем нужно создать код для запуска и обработки событий, а также постоянно собирать или обновлять все представления и объекты, необходимые для обработки запросов в модели чтения.</span><span class="sxs-lookup"><span data-stu-id="15235-186">The pattern adds complexity because code must be created to initiate and handle events, and assemble or update the appropriate views or objects required by queries or a read model.</span></span> <span data-ttu-id="15235-187">Сложность совместного применения шаблонов источников событий и CQRS затруднит реализацию системы и потребует нового подхода к разработке системы.</span><span class="sxs-lookup"><span data-stu-id="15235-187">The complexity of the CQRS pattern when used with the Event Sourcing pattern can make a successful implementation more difficult, and requires a different approach to designing systems.</span></span> <span data-ttu-id="15235-188">Но зато источники событий упрощают моделирование предметной области, а также позволяют легко изменять представления и создавать новые, сохраняя все изменения в данных.</span><span class="sxs-lookup"><span data-stu-id="15235-188">However, event sourcing can make it easier to model the domain, and makes it easier to rebuild views or create new ones because the intent of the changes in the data is preserved.</span></span>

- <span data-ttu-id="15235-189">Может потребоваться значительное время и большой объем ресурсов для создания материализованных представлений, которые используются в модели чтения или в проекциях, так как этот процесс включает воспроизведение и обработку всех событий для конкретных сущностей или коллекций сущностей.</span><span class="sxs-lookup"><span data-stu-id="15235-189">Generating materialized views for use in the read model or projections of the data by replaying and handling the events for specific entities or collections of entities can require significant processing time and resource usage.</span></span> <span data-ttu-id="15235-190">Это особенно важно при обработке данных за длительные периоды времени, ведь нужно полностью проверить все связанные события.</span><span class="sxs-lookup"><span data-stu-id="15235-190">This is especially true if it requires summation or analysis of values over long periods, because all the associated events might need to be examined.</span></span> <span data-ttu-id="15235-191">Чтобы решить эту проблему, можно создавать моментальные снимки данных через определенные промежутки времени, например сохранять общее число определенных действий за определенный период или текущие состояния сущностей.</span><span class="sxs-lookup"><span data-stu-id="15235-191">Resolve this by implementing snapshots of the data at scheduled intervals, such as a total count of the number of a specific action that have occurred, or the current state of an entity.</span></span>

## <a name="example"></a><span data-ttu-id="15235-192">Пример</span><span class="sxs-lookup"><span data-stu-id="15235-192">Example</span></span>

<span data-ttu-id="15235-193">Следующий фрагмент кода взят из реализации подхода CQRS, в рамках которой применены разные определения для моделей чтения и записи.</span><span class="sxs-lookup"><span data-stu-id="15235-193">The following code shows some extracts from an example of a CQRS implementation that uses different definitions for the read and the write models.</span></span> <span data-ttu-id="15235-194">Интерфейсы модели не налагают ограничения на функции хранилищ данных, и их можно изменять или конфигурировать независимо друг от друга, так как эти интерфейсы разделены.</span><span class="sxs-lookup"><span data-stu-id="15235-194">The model interfaces don't dictate any features of the underlying data stores, and they can evolve and be fine-tuned independently because these interfaces are separated.</span></span>

<span data-ttu-id="15235-195">В следующем коде представлено определение модели чтения.</span><span class="sxs-lookup"><span data-stu-id="15235-195">The following code shows the read model definition.</span></span>

```csharp
// Query interface
namespace ReadModel
{
  public interface ProductsDao
  {
    ProductDisplay FindById(int productId);
    ICollection<ProductDisplay> FindByName(string name);
    ICollection<ProductInventory> FindOutOfStockProducts();
    ICollection<ProductDisplay> FindRelatedProducts(int productId);
  }

  public class ProductDisplay
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal UnitPrice { get; set; }
    public bool IsOutOfStock { get; set; }
    public double UserRating { get; set; }
  }

  public class ProductInventory
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public int CurrentStock { get; set; }
  }
}
```

<span data-ttu-id="15235-196">Эта система позволяет пользователям оценивать товары.</span><span class="sxs-lookup"><span data-stu-id="15235-196">The system allows users to rate products.</span></span> <span data-ttu-id="15235-197">Для этого в коде приложения реализована команда `RateProduct`, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="15235-197">The application code does this using the `RateProduct` command shown in the following code.</span></span>

```csharp
public interface ICommand
{
  Guid Id { get; }
}

public class RateProduct : ICommand
{
  public RateProduct()
  {
    this.Id = Guid.NewGuid();
  }
  public Guid Id { get; set; }
  public int ProductId { get; set; }
  public int Rating { get; set; }
  public int UserId {get; set; }
}
```

<span data-ttu-id="15235-198">Система использует класс `ProductsCommandHandler` для обработки команд, отправляемых приложением.</span><span class="sxs-lookup"><span data-stu-id="15235-198">The system uses the `ProductsCommandHandler` class to handle commands sent by the application.</span></span> <span data-ttu-id="15235-199">Обычно клиенты отправляют команды в предметную область через систему обмена сообщениями, например очередь.</span><span class="sxs-lookup"><span data-stu-id="15235-199">Clients typically send commands to the domain through a messaging system such as a queue.</span></span> <span data-ttu-id="15235-200">Обработчик команд принимает эти команды и вызывает методы интерфейса предметной области.</span><span class="sxs-lookup"><span data-stu-id="15235-200">The command handler accepts these commands and invokes methods of the domain interface.</span></span> <span data-ttu-id="15235-201">Для каждой команды выбирается такая степень детализации, которая позволяет снизить риск конфликта запросов.</span><span class="sxs-lookup"><span data-stu-id="15235-201">The granularity of each command is designed to reduce the chance of conflicting requests.</span></span> <span data-ttu-id="15235-202">В следующем коде представлена упрощенная реализация класса `ProductsCommandHandler`.</span><span class="sxs-lookup"><span data-stu-id="15235-202">The following code shows an outline of the `ProductsCommandHandler` class.</span></span>

```csharp
public class ProductsCommandHandler :
    ICommandHandler<AddNewProduct>,
    ICommandHandler<RateProduct>,
    ICommandHandler<AddToInventory>,
    ICommandHandler<ConfirmItemShipped>,
    ICommandHandler<UpdateStockFromInventoryRecount>
{
  private readonly IRepository<Product> repository;

  public ProductsCommandHandler (IRepository<Product> repository)
  {
    this.repository = repository;
  }

  void Handle (AddNewProduct command)
  {
    ...
  }

  void Handle (RateProduct command)
  {
    var product = repository.Find(command.ProductId);
    if (product != null)
    {
      product.RateProduct(command.UserId, command.Rating);
      repository.Save(product);
    }
  }

  void Handle (AddToInventory command)
  {
    ...
  }

  void Handle (ConfirmItemsShipped command)
  {
    ...
  }

  void Handle (UpdateStockFromInventoryRecount command)
  {
    ...
  }
}
```

<span data-ttu-id="15235-203">В следующем коде реализован интерфейс `IProductsDomain` для модели записи.</span><span class="sxs-lookup"><span data-stu-id="15235-203">The following code shows the `IProductsDomain` interface from the write model.</span></span>

```csharp
public interface IProductsDomain
{
  void AddNewProduct(int id, string name, string description, decimal price);
  void RateProduct(int userId, int rating);
  void AddToInventory(int productId, int quantity);
  void ConfirmItemsShipped(int productId, int quantity);
  void UpdateStockFromInventoryRecount(int productId, int updatedQuantity);
}
```

<span data-ttu-id="15235-204">Обратите внимание, что в интерфейс `IProductsDomain` включены методы, имеющие значение в конкретной предметной области.</span><span class="sxs-lookup"><span data-stu-id="15235-204">Also notice how the `IProductsDomain` interface contains methods that have a meaning in the domain.</span></span> <span data-ttu-id="15235-205">В среде CRUD такие методы обычно имеют универсальные имена, например `Save` или `Update`, и принимают DTO в качестве единственного аргумента.</span><span class="sxs-lookup"><span data-stu-id="15235-205">Typically, in a CRUD environment these methods would have generic names such as `Save` or `Update`, and have a DTO as the only argument.</span></span> <span data-ttu-id="15235-206">Подход CQRS можно адаптировать под потребности конкретных бизнес-процессов и систем управления запасами, которые используются в организации.</span><span class="sxs-lookup"><span data-stu-id="15235-206">The CQRS approach can be designed to meet the needs of this organization's business and inventory management systems.</span></span>

## <a name="related-patterns-and-guidance"></a><span data-ttu-id="15235-207">Связанные шаблоны и рекомендации</span><span class="sxs-lookup"><span data-stu-id="15235-207">Related patterns and guidance</span></span>

<span data-ttu-id="15235-208">При реализации этого шаблона будут полезны следующие шаблоны и рекомендации:</span><span class="sxs-lookup"><span data-stu-id="15235-208">The following patterns and guidance are useful when implementing this pattern:</span></span>

- <span data-ttu-id="15235-209">Сравнение CQRS с другими стилями архитектуры см. в статьях [Стили архитектуры](/azure/architecture/guide/architecture-styles/) и [Стиль архитектуры CQRS](/azure/architecture/guide/architecture-styles/cqrs).</span><span class="sxs-lookup"><span data-stu-id="15235-209">For a comparison of CQRS with other architectural styles, see [Architecture styles](/azure/architecture/guide/architecture-styles/) and [CQRS architecture style](/azure/architecture/guide/architecture-styles/cqrs).</span></span>

- <span data-ttu-id="15235-210">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx) (Руководство по обеспечению согласованности данных).</span><span class="sxs-lookup"><span data-stu-id="15235-210">[Data Consistency Primer](https://msdn.microsoft.com/library/dn589800.aspx).</span></span> <span data-ttu-id="15235-211">Описание проблем, часто возникающих из-за режима итоговой согласованности, который поддерживается в шаблоне между хранилищами для чтения и записи, и способов решения этих проблем.</span><span class="sxs-lookup"><span data-stu-id="15235-211">Explains the issues that are typically encountered due to eventual consistency between the read and write data stores when using the CQRS pattern, and how these issues can be resolved.</span></span>

- <span data-ttu-id="15235-212">[Data Partitioning Guidance](https://msdn.microsoft.com/library/dn589795.aspx) (Руководство по секционированию данных).</span><span class="sxs-lookup"><span data-stu-id="15235-212">[Data Partitioning Guidance](https://msdn.microsoft.com/library/dn589795.aspx).</span></span> <span data-ttu-id="15235-213">Описание процедуры разделения используемых в шаблоне CQRS хранилищ данных для чтения и записи на секции, управление которыми и доступ к которым выполняются раздельно. Это позволяет улучшить масштабируемость, уменьшить число конфликтов и оптимизировать производительность.</span><span class="sxs-lookup"><span data-stu-id="15235-213">Describes how the read and write data stores used in the CQRS pattern can be divided into partitions that can be managed and accessed separately to improve scalability, reduce contention, and optimize performance.</span></span>

- <span data-ttu-id="15235-214">[Шаблон источников событий](event-sourcing.md).</span><span class="sxs-lookup"><span data-stu-id="15235-214">[Event Sourcing Pattern](event-sourcing.md).</span></span> <span data-ttu-id="15235-215">Описание применения источников событий совместно с шаблоном CQRS для упрощения задач в сложных предметных областях при одновременном повышении производительности, масштабируемости и скорости реагирования.</span><span class="sxs-lookup"><span data-stu-id="15235-215">Describes in more detail how Event Sourcing can be used with the CQRS pattern to simplify tasks in complex domains while improving performance, scalability, and responsiveness.</span></span> <span data-ttu-id="15235-216">Кроме того, рассмотрено предоставление поддержки согласованности для транзакционных данных с сохранением полного аудиторского следа и истории, которые позволяют выполнять компенсацию действий.</span><span class="sxs-lookup"><span data-stu-id="15235-216">As well as how to provide consistency for transactional data while maintaining full audit trails and history that can enable compensating actions.</span></span>

- <span data-ttu-id="15235-217">[Шаблон материализованного представления](materialized-view.md).</span><span class="sxs-lookup"><span data-stu-id="15235-217">[Materialized View Pattern](materialized-view.md).</span></span> <span data-ttu-id="15235-218">Модель чтения в реализации CQRS может содержать или самостоятельно создавать материализованные представления данных из модели записи.</span><span class="sxs-lookup"><span data-stu-id="15235-218">The read model of a CQRS implementation can contain materialized views of the write model data, or the read model can be used to generate materialized views.</span></span>

- <span data-ttu-id="15235-219">Руководство по шаблонам и методикам для модели CQRS представлено [здесь](http://aka.ms/cqrs).</span><span class="sxs-lookup"><span data-stu-id="15235-219">The patterns & practices guide [CQRS Journey](http://aka.ms/cqrs).</span></span> <span data-ttu-id="15235-220">В частности, статья [Introducing the Command Query Responsibility Segregation Pattern](https://msdn.microsoft.com/library/jj591573.aspx) (Знакомство с шаблоном разделения ответственности команды и запроса) описывает этот шаблон и его применимость, а статья [Epilogue: Lessons Learned](https://msdn.microsoft.com/library/jj591568.aspx) (Эпилог и обсуждение полученного опыта) поможет разобраться с некоторыми проблемами, которые возникают при использовании этого шаблона.</span><span class="sxs-lookup"><span data-stu-id="15235-220">In particular, [Introducing the Command Query Responsibility Segregation Pattern](https://msdn.microsoft.com/library/jj591573.aspx) explores the pattern and when it's useful, and [Epilogue: Lessons Learned](https://msdn.microsoft.com/library/jj591568.aspx) helps you understand some of the issues that come up when using this pattern.</span></span>

- <span data-ttu-id="15235-221">В записи в блоге Мартина Фоулера (Martin Fowler) о [CQRS](http://martinfowler.com/bliki/CQRS.html) представлен обзор этого шаблона, а также ссылки на полезные ресурсы.</span><span class="sxs-lookup"><span data-stu-id="15235-221">The post [CQRS by Martin Fowler](http://martinfowler.com/bliki/CQRS.html), which explains the basics of the pattern and links to other useful resources.</span></span>

- <span data-ttu-id="15235-222">В [записях блога](http://codebetter.com/gregyoung/) Грега Янга (Greg Young) подробно рассмотрены разные аспекты использования шаблона CQRS.</span><span class="sxs-lookup"><span data-stu-id="15235-222">[Greg Young’s posts](http://codebetter.com/gregyoung/), which explore many aspects of the CQRS pattern.</span></span>
