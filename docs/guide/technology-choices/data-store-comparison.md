---
title: Критерии выбора хранилища данных
titleSuffix: Azure Application Architecture Guide
description: Общие сведения о вычислительных возможностях в Azure.
author: MikeWasson
ms.date: 06/01/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 3fd3badd66edbe561bea88576bb80d9fc3e0bb79
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59068928"
---
# <a name="criteria-for-choosing-a-data-store"></a><span data-ttu-id="37acd-103">Критерии выбора хранилища данных</span><span class="sxs-lookup"><span data-stu-id="37acd-103">Criteria for choosing a data store</span></span>

<span data-ttu-id="37acd-104">Azure поддерживает различные типы решений для хранения данных с различными функциями и возможностями.</span><span class="sxs-lookup"><span data-stu-id="37acd-104">Azure supports many types of data storage solutions, each providing different features and capabilities.</span></span> <span data-ttu-id="37acd-105">В этой статье описываются критерии сравнения, которые следует применять при оценке хранилища данных.</span><span class="sxs-lookup"><span data-stu-id="37acd-105">This article describes the comparison criteria you should use when evaluating a data store.</span></span> <span data-ttu-id="37acd-106">Эти сведения помогут выбрать оптимальное хранилище данных в соответствии с требованиями решения.</span><span class="sxs-lookup"><span data-stu-id="37acd-106">The goal is to help you determine which data storage types can meet your solution's requirements.</span></span>

## <a name="general-considerations"></a><span data-ttu-id="37acd-107">Общие рекомендации</span><span class="sxs-lookup"><span data-stu-id="37acd-107">General Considerations</span></span>

<span data-ttu-id="37acd-108">Для начала соберите как можно больше сведений о требованиях к данным.</span><span class="sxs-lookup"><span data-stu-id="37acd-108">To start your comparison, gather as much of the following information as you can about your data needs.</span></span> <span data-ttu-id="37acd-109">Эти сведения помогут определить тип хранилища данных в соответствии с вашими требованиями.</span><span class="sxs-lookup"><span data-stu-id="37acd-109">This information will help you to determine which data storage types will meet your needs.</span></span>

### <a name="functional-requirements"></a><span data-ttu-id="37acd-110">Функциональные требования</span><span class="sxs-lookup"><span data-stu-id="37acd-110">Functional requirements</span></span>

- <span data-ttu-id="37acd-111">**Формат данных.**</span><span class="sxs-lookup"><span data-stu-id="37acd-111">**Data format**.</span></span> <span data-ttu-id="37acd-112">Данные какого типа требуется хранить?</span><span class="sxs-lookup"><span data-stu-id="37acd-112">What type of data are you intending to store?</span></span> <span data-ttu-id="37acd-113">Распространенные типы: данные о транзакциях, объекты JSON, телеметрия, индексы поиска или неструктурированные файлы.</span><span class="sxs-lookup"><span data-stu-id="37acd-113">Common types include transactional data, JSON objects, telemetry, search indexes, or flat files.</span></span>

- <span data-ttu-id="37acd-114">**Размер данных.**</span><span class="sxs-lookup"><span data-stu-id="37acd-114">**Data size**.</span></span> <span data-ttu-id="37acd-115">Каков размер сущностей, которые нужно хранить?</span><span class="sxs-lookup"><span data-stu-id="37acd-115">How large are the entities you need to store?</span></span> <span data-ttu-id="37acd-116">Эти сущности следует использовать как один документ или их можно разбить на несколько документов, таблиц, коллекций и т. д.?</span><span class="sxs-lookup"><span data-stu-id="37acd-116">Will these entities need to be maintained as a single document, or can they be split across multiple documents, tables, collections, and so forth?</span></span>

- <span data-ttu-id="37acd-117">**Масштаб и структура.**</span><span class="sxs-lookup"><span data-stu-id="37acd-117">**Scale and structure**.</span></span> <span data-ttu-id="37acd-118">Какова требуемая емкость хранилища?</span><span class="sxs-lookup"><span data-stu-id="37acd-118">What is the overall amount of storage capacity you need?</span></span> <span data-ttu-id="37acd-119">Планируется ли секционирование данных?</span><span class="sxs-lookup"><span data-stu-id="37acd-119">Do you anticipate partitioning your data?</span></span>

- <span data-ttu-id="37acd-120">**Связи между данными.**</span><span class="sxs-lookup"><span data-stu-id="37acd-120">**Data relationships**.</span></span> <span data-ttu-id="37acd-121">Какой тип связи требуется реализовать в данных: "один ко многим" или "многие ко многим"?</span><span class="sxs-lookup"><span data-stu-id="37acd-121">Will your data need to support one-to-many or many-to-many relationships?</span></span> <span data-ttu-id="37acd-122">Сами связи являются важной частью данных?</span><span class="sxs-lookup"><span data-stu-id="37acd-122">Are relationships themselves an important part of the data?</span></span> <span data-ttu-id="37acd-123">Вам понадобится объединить данные из одного набора данных или из внешних наборов данных?</span><span class="sxs-lookup"><span data-stu-id="37acd-123">Will you need to join or otherwise combine data from within the same dataset, or from external datasets?</span></span>

- <span data-ttu-id="37acd-124">**Модель согласованности.**</span><span class="sxs-lookup"><span data-stu-id="37acd-124">**Consistency model**.</span></span> <span data-ttu-id="37acd-125">Насколько важно, чтобы обновления одного узла отображались на других узлах перед дальнейшим внесением изменений?</span><span class="sxs-lookup"><span data-stu-id="37acd-125">How important is it for updates made in one node to appear in other nodes, before further changes can be made?</span></span> <span data-ttu-id="37acd-126">Возможно ли принять итоговую согласованность?</span><span class="sxs-lookup"><span data-stu-id="37acd-126">Can you accept eventual consistency?</span></span> <span data-ttu-id="37acd-127">Требуются ли гарантии ACID для транзакций?</span><span class="sxs-lookup"><span data-stu-id="37acd-127">Do you need ACID guarantees for transactions?</span></span>

- <span data-ttu-id="37acd-128">**Гибкость схемы.**</span><span class="sxs-lookup"><span data-stu-id="37acd-128">**Schema flexibility**.</span></span> <span data-ttu-id="37acd-129">Какие схемы будут применяться к данным?</span><span class="sxs-lookup"><span data-stu-id="37acd-129">What kind of schemas will you apply to your data?</span></span> <span data-ttu-id="37acd-130">Какой подход будет применяться: фиксированная схема, схема по записи или схема по чтению?</span><span class="sxs-lookup"><span data-stu-id="37acd-130">Will you use a fixed schema, a schema-on-write approach, or a schema-on-read approach?</span></span>

- <span data-ttu-id="37acd-131">**Параллелизм.**</span><span class="sxs-lookup"><span data-stu-id="37acd-131">**Concurrency**.</span></span> <span data-ttu-id="37acd-132">Какой механизм параллелизма следует использовать при обновлении и синхронизации данных?</span><span class="sxs-lookup"><span data-stu-id="37acd-132">What kind of concurrency mechanism do you want to use when updating and synchronizing data?</span></span> <span data-ttu-id="37acd-133">Будет ли в приложении выполняться большое количество обновлений, которые потенциально могут конфликтовать?</span><span class="sxs-lookup"><span data-stu-id="37acd-133">Will the application perform many updates that could potentially conflict.</span></span> <span data-ttu-id="37acd-134">В этом случае может потребоваться блокировка записей и контроль пессимистичной блокировки.</span><span class="sxs-lookup"><span data-stu-id="37acd-134">If so, you may require record locking and pessimistic concurrency control.</span></span> <span data-ttu-id="37acd-135">Кроме того, поддерживает ли система элементы управления оптимистичной блокировкой?</span><span class="sxs-lookup"><span data-stu-id="37acd-135">Alternatively, can you support optimistic concurrency controls?</span></span> <span data-ttu-id="37acd-136">Если ответ положительный, достаточно ли простого контроля параллелизма на основе меток времени или требуются дополнительные возможности по контролю параллелизма в нескольких версиях?</span><span class="sxs-lookup"><span data-stu-id="37acd-136">If so, is simple timestamp-based concurrency control enough, or do you need the added functionality of multi-version concurrency control?</span></span>

- <span data-ttu-id="37acd-137">**Перемещение данных.**</span><span class="sxs-lookup"><span data-stu-id="37acd-137">**Data movement**.</span></span> <span data-ttu-id="37acd-138">Понадобится ли решению выполнять задачи ETL для переноса данных в другие хранилища данных?</span><span class="sxs-lookup"><span data-stu-id="37acd-138">Will your solution need to perform ETL tasks to move data to other stores or data warehouses?</span></span>

- <span data-ttu-id="37acd-139">**Жизненный цикл данных.**</span><span class="sxs-lookup"><span data-stu-id="37acd-139">**Data lifecycle**.</span></span> <span data-ttu-id="37acd-140">Принадлежат ли данные к типу "однократная запись, многократное чтение"?</span><span class="sxs-lookup"><span data-stu-id="37acd-140">Is the data write-once, read-many?</span></span> <span data-ttu-id="37acd-141">Можно ли их переместить в "горячее" или "холодное" хранилище?</span><span class="sxs-lookup"><span data-stu-id="37acd-141">Can it be moved into cool or cold storage?</span></span>

- <span data-ttu-id="37acd-142">**Другие поддерживаемые функции.**</span><span class="sxs-lookup"><span data-stu-id="37acd-142">**Other supported features**.</span></span> <span data-ttu-id="37acd-143">Требуются ли другие специальные функции, например проверка схемы, агрегирование, индексирование, полнотекстовый поиск, MapReduce или другие возможности запросов?</span><span class="sxs-lookup"><span data-stu-id="37acd-143">Do you need any other specific features, such as schema validation, aggregation, indexing, full-text search, MapReduce, or other query capabilities?</span></span>

### <a name="non-functional-requirements"></a><span data-ttu-id="37acd-144">Нефункциональные требования</span><span class="sxs-lookup"><span data-stu-id="37acd-144">Non-functional requirements</span></span>

- <span data-ttu-id="37acd-145">**Производительность и масштабирование.**</span><span class="sxs-lookup"><span data-stu-id="37acd-145">**Performance and scalability**.</span></span> <span data-ttu-id="37acd-146">Каковы требования к производительности данных?</span><span class="sxs-lookup"><span data-stu-id="37acd-146">What are your data performance requirements?</span></span> <span data-ttu-id="37acd-147">Имеются ли специальные требования к приему и скорости обработки данных?</span><span class="sxs-lookup"><span data-stu-id="37acd-147">Do you have specific requirements for data ingestion rates and data processing rates?</span></span> <span data-ttu-id="37acd-148">Каково приемлемое время отклика для обращения к данным и их статистической обработки после приема?</span><span class="sxs-lookup"><span data-stu-id="37acd-148">What are the acceptable response times for querying and aggregation of data once ingested?</span></span> <span data-ttu-id="37acd-149">На сколько требуется увеличить масштаб хранилища данных?</span><span class="sxs-lookup"><span data-stu-id="37acd-149">How large will you need the data store to scale up?</span></span> <span data-ttu-id="37acd-150">Каких операций больше в рабочей нагрузке: чтения или записи?</span><span class="sxs-lookup"><span data-stu-id="37acd-150">Is your workload more read-heavy or write-heavy?</span></span>

- <span data-ttu-id="37acd-151">**Надежность.**</span><span class="sxs-lookup"><span data-stu-id="37acd-151">**Reliability**.</span></span> <span data-ttu-id="37acd-152">Какие общие Соглашения об уровне обслуживания требуется поддерживать?</span><span class="sxs-lookup"><span data-stu-id="37acd-152">What overall SLA do you need to support?</span></span> <span data-ttu-id="37acd-153">Какой уровень отказоустойчивости необходимо предоставить для потребителей данных?</span><span class="sxs-lookup"><span data-stu-id="37acd-153">What level of fault-tolerance do you need to provide for data consumers?</span></span> <span data-ttu-id="37acd-154">Какие возможности резервного копирования и восстановления требуются?</span><span class="sxs-lookup"><span data-stu-id="37acd-154">What kind of backup and restore capabilities do you need?</span></span>

- <span data-ttu-id="37acd-155">**Репликация.**</span><span class="sxs-lookup"><span data-stu-id="37acd-155">**Replication**.</span></span> <span data-ttu-id="37acd-156">Понадобится ли распределить данные по нескольким репликам или регионам?</span><span class="sxs-lookup"><span data-stu-id="37acd-156">Will your data need to be distributed among multiple replicas or regions?</span></span> <span data-ttu-id="37acd-157">Какие возможности репликации данных требуются?</span><span class="sxs-lookup"><span data-stu-id="37acd-157">What kind of data replication capabilities do you require?</span></span>

- <span data-ttu-id="37acd-158">**Ограничения.**</span><span class="sxs-lookup"><span data-stu-id="37acd-158">**Limits**.</span></span> <span data-ttu-id="37acd-159">Будут ли ограничения конкретного хранилища данных поддерживать ваши требования к масштабированию, числу подключений и пропускной способности?</span><span class="sxs-lookup"><span data-stu-id="37acd-159">Will the limits of a particular data store support your requirements for scale, number of connections, and throughput?</span></span>

### <a name="management-and-cost"></a><span data-ttu-id="37acd-160">Управление и затраты</span><span class="sxs-lookup"><span data-stu-id="37acd-160">Management and cost</span></span>

- <span data-ttu-id="37acd-161">**Управляемая служба.**</span><span class="sxs-lookup"><span data-stu-id="37acd-161">**Managed service**.</span></span> <span data-ttu-id="37acd-162">По возможности используйте управляемую службу данных, если вам не требуются конкретные возможности, доступные только в хранилище данных, размещенном в IaaS.</span><span class="sxs-lookup"><span data-stu-id="37acd-162">When possible, use a managed data service, unless you require specific capabilities that can only be found in an IaaS-hosted data store.</span></span>

- <span data-ttu-id="37acd-163">**Доступность по регионам.**</span><span class="sxs-lookup"><span data-stu-id="37acd-163">**Region availability**.</span></span> <span data-ttu-id="37acd-164">Доступна ли управляемая служба во всех регионах Azure?</span><span class="sxs-lookup"><span data-stu-id="37acd-164">For managed services, is the service available in all Azure regions?</span></span> <span data-ttu-id="37acd-165">Нужно ли разместить решение в определенных регионах Azure?</span><span class="sxs-lookup"><span data-stu-id="37acd-165">Does your solution need to be hosted in certain Azure regions?</span></span>

- <span data-ttu-id="37acd-166">**Мобильность.**</span><span class="sxs-lookup"><span data-stu-id="37acd-166">**Portability**.</span></span> <span data-ttu-id="37acd-167">Данных потребуется перенести на предприятии, внешние центры обработки данных или другие облачные среды размещения?</span><span class="sxs-lookup"><span data-stu-id="37acd-167">Will your data need to be migrated to on-premises, external datacenters, or other cloud hosting environments?</span></span>

- <span data-ttu-id="37acd-168">**Лицензирование.**</span><span class="sxs-lookup"><span data-stu-id="37acd-168">**Licensing**.</span></span> <span data-ttu-id="37acd-169">Что вы предпочитаете: частную лицензию или лицензию OSS?</span><span class="sxs-lookup"><span data-stu-id="37acd-169">Do you have a preference of a proprietary versus OSS license type?</span></span> <span data-ttu-id="37acd-170">Существуют ли другие внешние ограничения по типу лицензии, который следует использовать?</span><span class="sxs-lookup"><span data-stu-id="37acd-170">Are there any other external restrictions on what type of license you can use?</span></span>

- <span data-ttu-id="37acd-171">**Общая стоимость.**</span><span class="sxs-lookup"><span data-stu-id="37acd-171">**Overall cost**.</span></span> <span data-ttu-id="37acd-172">Какова общая стоимость использования службы в решении?</span><span class="sxs-lookup"><span data-stu-id="37acd-172">What is the overall cost of using the service within your solution?</span></span> <span data-ttu-id="37acd-173">Сколько экземпляров потребуется в соответствии с требованиями к бесперебойной работе и пропускной способности?</span><span class="sxs-lookup"><span data-stu-id="37acd-173">How many instances will need to run, to support your uptime and throughput requirements?</span></span> <span data-ttu-id="37acd-174">Рассчитывая это, учтите эксплуатационные расходы.</span><span class="sxs-lookup"><span data-stu-id="37acd-174">Consider operations costs in this calculation.</span></span> <span data-ttu-id="37acd-175">Одна из причин для выбора управляемых служб — низкие эксплуатационные расходы.</span><span class="sxs-lookup"><span data-stu-id="37acd-175">One reason to prefer managed services is the reduced operational cost.</span></span>

- <span data-ttu-id="37acd-176">**Экономичность.**</span><span class="sxs-lookup"><span data-stu-id="37acd-176">**Cost effectiveness**.</span></span> <span data-ttu-id="37acd-177">Можно ли секционировать данные для более экономичного хранения?</span><span class="sxs-lookup"><span data-stu-id="37acd-177">Can you partition your data, to store it more cost effectively?</span></span> <span data-ttu-id="37acd-178">Например, можно ли переместить большие объекты из дорогой реляционной базы данных в хранилище объектов?</span><span class="sxs-lookup"><span data-stu-id="37acd-178">For example, can you move large objects out of an expensive relational database into an object store?</span></span>

### <a name="security"></a><span data-ttu-id="37acd-179">Безопасность</span><span class="sxs-lookup"><span data-stu-id="37acd-179">Security</span></span>

- <span data-ttu-id="37acd-180">**Безопасность**.</span><span class="sxs-lookup"><span data-stu-id="37acd-180">**Security**.</span></span> <span data-ttu-id="37acd-181">Какой тип шифрования нужен?</span><span class="sxs-lookup"><span data-stu-id="37acd-181">What type of encryption do you require?</span></span> <span data-ttu-id="37acd-182">Требуется ли шифрование неактивных данных?</span><span class="sxs-lookup"><span data-stu-id="37acd-182">Do you need encryption at rest?</span></span> <span data-ttu-id="37acd-183">Какой метод аутентификации нужно использовать для подключения к данным?</span><span class="sxs-lookup"><span data-stu-id="37acd-183">What authentication mechanism do you want to use to connect to your data?</span></span>

- <span data-ttu-id="37acd-184">**Аудит.**</span><span class="sxs-lookup"><span data-stu-id="37acd-184">**Auditing**.</span></span> <span data-ttu-id="37acd-185">Журнал какого типа аудита необходимо создать?</span><span class="sxs-lookup"><span data-stu-id="37acd-185">What kind of audit log do you need to generate?</span></span>

- <span data-ttu-id="37acd-186">**Требования к сети.**</span><span class="sxs-lookup"><span data-stu-id="37acd-186">**Networking requirements**.</span></span> <span data-ttu-id="37acd-187">Необходимо ли ограничить доступ к данным из других сетевых ресурсов или управлять доступом иным образом?</span><span class="sxs-lookup"><span data-stu-id="37acd-187">Do you need to restrict or otherwise manage access to your data from other network resources?</span></span> <span data-ttu-id="37acd-188">Данные должны быть доступны только в среде Azure?</span><span class="sxs-lookup"><span data-stu-id="37acd-188">Does data need to be accessible only from inside the Azure environment?</span></span> <span data-ttu-id="37acd-189">Требуется ли использовать конкретный IP-адрес или подсеть для доступа к данным?</span><span class="sxs-lookup"><span data-stu-id="37acd-189">Does the data need to be accessible from specific IP addresses or subnets?</span></span> <span data-ttu-id="37acd-190">Нужно ли предоставить доступ к данным из приложений или служб, размещенных локально или в других внешних центрах обработки данных?</span><span class="sxs-lookup"><span data-stu-id="37acd-190">Does it need to be accessible from applications or services hosted on-premises or in other external datacenters?</span></span>

### <a name="devops"></a><span data-ttu-id="37acd-191">DevOps</span><span class="sxs-lookup"><span data-stu-id="37acd-191">DevOps</span></span>

- <span data-ttu-id="37acd-192">**Навыки.**</span><span class="sxs-lookup"><span data-stu-id="37acd-192">**Skill set**.</span></span> <span data-ttu-id="37acd-193">Освоила ли ваша команда определенные языки программирования, работу с операционными системами или с другими технологиями?</span><span class="sxs-lookup"><span data-stu-id="37acd-193">Are there particular programming languages, operating systems, or other technology that your team is particularly adept at using?</span></span> <span data-ttu-id="37acd-194">При работе с какими технологиями и системами у вашей команды возникают проблемы?</span><span class="sxs-lookup"><span data-stu-id="37acd-194">Are there others that would be difficult for your team to work with?</span></span>

- <span data-ttu-id="37acd-195">**Клиенты.** Предоставляется ли поддержка клиентов для ваших языков разработки на должном уровне?</span><span class="sxs-lookup"><span data-stu-id="37acd-195">**Clients** Is there good client support for your development languages?</span></span>

<span data-ttu-id="37acd-196">В следующих разделах сравниваются различные модели хранилища данных с точки зрения профиля рабочей нагрузки, типов данных и примеров использования.</span><span class="sxs-lookup"><span data-stu-id="37acd-196">The following sections compare various data store models in terms of workload profile, data types, and example use cases.</span></span>

## <a name="relational-database-management-systems-rdbms"></a><span data-ttu-id="37acd-197">Реляционные СУБД</span><span class="sxs-lookup"><span data-stu-id="37acd-197">Relational database management systems (RDBMS)</span></span>

<!-- markdownlint-disable MD033 -->

<table>
<tr><td><span data-ttu-id="37acd-198"><strong>Рабочая нагрузка</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-198"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-199">Записи создаются и имеющиеся данные обновляются регулярно.</span><span class="sxs-lookup"><span data-stu-id="37acd-199">Both the creation of new records and updates to existing data happen regularly.</span></span></li>
            <li><span data-ttu-id="37acd-200">Несколько операций должны выполниться в одной транзакции.</span><span class="sxs-lookup"><span data-stu-id="37acd-200">Multiple operations have to be completed in a single transaction.</span></span></li>
            <li><span data-ttu-id="37acd-201">Требуются функции агрегирования для выполнения перекрестной табуляции.</span><span class="sxs-lookup"><span data-stu-id="37acd-201">Requires aggregation functions to perform cross-tabulation.</span></span></li>
            <li><span data-ttu-id="37acd-202">Требуется тесная интеграция с инструментами создания отчетов.</span><span class="sxs-lookup"><span data-stu-id="37acd-202">Strong integration with reporting tools is required.</span></span></li>
            <li><span data-ttu-id="37acd-203">Связи задаются по ограничениям базы данных.</span><span class="sxs-lookup"><span data-stu-id="37acd-203">Relationships are enforced using database constraints.</span></span></li>
            <li><span data-ttu-id="37acd-204">Индексы используются для оптимизации производительности запросов.</span><span class="sxs-lookup"><span data-stu-id="37acd-204">Indexes are used to optimize query performance.</span></span></li>
            <li><span data-ttu-id="37acd-205">Разрешается доступ к определенному подмножеству данных.</span><span class="sxs-lookup"><span data-stu-id="37acd-205">Allows access to specific subsets of data.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-206"><strong>Тип данных</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-206"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-207">Данные имеют высокую степень нормализации.</span><span class="sxs-lookup"><span data-stu-id="37acd-207">Data is highly normalized.</span></span></li>
            <li><span data-ttu-id="37acd-208">Схемы базы данных требуются и принудительно применяются.</span><span class="sxs-lookup"><span data-stu-id="37acd-208">Database schemas are required and enforced.</span></span></li>
            <li><span data-ttu-id="37acd-209">Для данных в базе данных настроена связь "многие ко многим".</span><span class="sxs-lookup"><span data-stu-id="37acd-209">Many-to-many relationships between data entities in the database.</span></span></li>
            <li><span data-ttu-id="37acd-210">Ограничения определяются в схеме и применяются к любым данным в базе данных.</span><span class="sxs-lookup"><span data-stu-id="37acd-210">Constraints are defined in the schema and imposed on any data in the database.</span></span></li>
            <li><span data-ttu-id="37acd-211">Данные требуют высокого уровня целостности.</span><span class="sxs-lookup"><span data-stu-id="37acd-211">Data requires high integrity.</span></span> <span data-ttu-id="37acd-212">Индексы и связи требуют осторожного использования.</span><span class="sxs-lookup"><span data-stu-id="37acd-212">Indexes and relationships need to be maintained accurately.</span></span></li>
            <li><span data-ttu-id="37acd-213">Данные требуют строгой согласованности.</span><span class="sxs-lookup"><span data-stu-id="37acd-213">Data requires strong consistency.</span></span> <span data-ttu-id="37acd-214">Транзакции осуществляются таким образом, который гарантирует, что все данные полностью согласованы для всех пользователей и процессов</span><span class="sxs-lookup"><span data-stu-id="37acd-214">Transactions operate in a way that ensures all data are 100% consistent for all users and processes.</span></span></li>
            <li><span data-ttu-id="37acd-215">Отдельные записи представлены в малом и среднем размере.</span><span class="sxs-lookup"><span data-stu-id="37acd-215">Size of individual data entries is intended to be small to medium-sized.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-216"><strong>Примеры</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-216"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-217">Бизнес-приложения (управление человеческими ресурсами, управление взаимоотношениями с клиентами, планирование ресурсов предприятия)</span><span class="sxs-lookup"><span data-stu-id="37acd-217">Line of business  (human capital management, customer relationship management, enterprise resource planning)</span></span></li>
            <li><span data-ttu-id="37acd-218">Управление запасами</span><span class="sxs-lookup"><span data-stu-id="37acd-218">Inventory management</span></span></li>
            <li><span data-ttu-id="37acd-219">База данных отчетов</span><span class="sxs-lookup"><span data-stu-id="37acd-219">Reporting database</span></span></li>
            <li><span data-ttu-id="37acd-220">Учет</span><span class="sxs-lookup"><span data-stu-id="37acd-220">Accounting</span></span></li>
            <li><span data-ttu-id="37acd-221">Управление ресурсами</span><span class="sxs-lookup"><span data-stu-id="37acd-221">Asset management</span></span></li>
            <li><span data-ttu-id="37acd-222">Управление средствами</span><span class="sxs-lookup"><span data-stu-id="37acd-222">Fund management</span></span></li>
            <li><span data-ttu-id="37acd-223">Управление заказами</span><span class="sxs-lookup"><span data-stu-id="37acd-223">Order management</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="document-databases"></a><span data-ttu-id="37acd-224">Базы данных документов</span><span class="sxs-lookup"><span data-stu-id="37acd-224">Document databases</span></span>

<table>
<tr><td><span data-ttu-id="37acd-225"><strong>Рабочая нагрузка</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-225"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-226">Общего назначения.</span><span class="sxs-lookup"><span data-stu-id="37acd-226">General purpose.</span></span></li>
            <li><span data-ttu-id="37acd-227">Операции вставки и обновления являются общими.</span><span class="sxs-lookup"><span data-stu-id="37acd-227">Insert and update operations are common.</span></span> <span data-ttu-id="37acd-228">Записи создаются и имеющиеся данные обновляются регулярно.</span><span class="sxs-lookup"><span data-stu-id="37acd-228">Both the creation of new records and updates to existing data happen regularly.</span></span></li>
            <li><span data-ttu-id="37acd-229">Нет объектно-реляционной несогласованности.</span><span class="sxs-lookup"><span data-stu-id="37acd-229">No object-relational impedance mismatch.</span></span> <span data-ttu-id="37acd-230">Документы можно эффективнее сопоставить со структурами объектов, используемыми в коде приложения.</span><span class="sxs-lookup"><span data-stu-id="37acd-230">Documents can better match the object structures used in application code.</span></span></li>
            <li><span data-ttu-id="37acd-231">Чаще используется оптимистичная блокировка.</span><span class="sxs-lookup"><span data-stu-id="37acd-231">Optimistic concurrency is more commonly used.</span></span></li>
            <li><span data-ttu-id="37acd-232">Данные должно изменять и обрабатывать использующее их приложение.</span><span class="sxs-lookup"><span data-stu-id="37acd-232">Data must be modified and processed by consuming application.</span></span></li>
            <li><span data-ttu-id="37acd-233">Данным требуется индекс по нескольким полям.</span><span class="sxs-lookup"><span data-stu-id="37acd-233">Data requires index on multiple fields.</span></span></li>
            <li><span data-ttu-id="37acd-234">Отдельные документы извлекаются и записываются как единый блок.</span><span class="sxs-lookup"><span data-stu-id="37acd-234">Individual documents are retrieved and written as a single block.</span></span></li>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-235"><strong>Тип данных</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-235"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-236">Данными можно управлять не нормализованно.</span><span class="sxs-lookup"><span data-stu-id="37acd-236">Data can be managed in de-normalized way.</span></span></li>
            <li><span data-ttu-id="37acd-237">Размер отдельных блоков данных документа относительно невелик.</span><span class="sxs-lookup"><span data-stu-id="37acd-237">Size of individual document data is relatively small.</span></span></li>
            <li><span data-ttu-id="37acd-238">Тип каждого документа может использовать собственную схему.</span><span class="sxs-lookup"><span data-stu-id="37acd-238">Each document type can use its own schema.</span></span></li>
            <li><span data-ttu-id="37acd-239">Документы могут содержать дополнительные поля.</span><span class="sxs-lookup"><span data-stu-id="37acd-239">Documents can include optional fields.</span></span></li>
            <li><span data-ttu-id="37acd-240">Данные документа полуструктурированные. Это означает, что типы данных каждого поля не строго определены.</span><span class="sxs-lookup"><span data-stu-id="37acd-240">Document data is semi-structured, meaning that data types of each field are not strictly defined.</span></span></li>
            <li><span data-ttu-id="37acd-241">Объединение данных поддерживается.</span><span class="sxs-lookup"><span data-stu-id="37acd-241">Data aggregation is supported.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-242"><strong>Примеры</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-242"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-243">Каталог продукции</span><span class="sxs-lookup"><span data-stu-id="37acd-243">Product catalog</span></span></li>
            <li><span data-ttu-id="37acd-244">Учетные записи пользователей</span><span class="sxs-lookup"><span data-stu-id="37acd-244">User accounts</span></span></li>
            <li><span data-ttu-id="37acd-245">Спецификация</span><span class="sxs-lookup"><span data-stu-id="37acd-245">Bill of materials</span></span></li>
            <li><span data-ttu-id="37acd-246">Персонализация</span><span class="sxs-lookup"><span data-stu-id="37acd-246">Personalization</span></span></li>
            <li><span data-ttu-id="37acd-247">Управление содержимым</span><span class="sxs-lookup"><span data-stu-id="37acd-247">Content management</span></span></li>
            <li><span data-ttu-id="37acd-248">Рабочие данные</span><span class="sxs-lookup"><span data-stu-id="37acd-248">Operations data</span></span></li>
            <li><span data-ttu-id="37acd-249">Управление запасами</span><span class="sxs-lookup"><span data-stu-id="37acd-249">Inventory management</span></span></li>
            <li><span data-ttu-id="37acd-250">Данные журнала транзакций</span><span class="sxs-lookup"><span data-stu-id="37acd-250">Transaction history data</span></span></li>
            <li><span data-ttu-id="37acd-251">Материализованное представление других хранилищ NoSQL</span><span class="sxs-lookup"><span data-stu-id="37acd-251">Materialized view of other NoSQL stores.</span></span> <span data-ttu-id="37acd-252">Заменяет индексацию файлов или больших двоичных объектов.</span><span class="sxs-lookup"><span data-stu-id="37acd-252">Replaces file/BLOB indexing.</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="keyvalue-stores"></a><span data-ttu-id="37acd-253">Хранилище пар "ключ — значение"</span><span class="sxs-lookup"><span data-stu-id="37acd-253">Key/value stores</span></span>

<table>
<tr><td><span data-ttu-id="37acd-254"><strong>Рабочая нагрузка</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-254"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-255">Данные идентифицируются, и к ним получается доступ с помощью одного ключа идентификатора, например словаря.</span><span class="sxs-lookup"><span data-stu-id="37acd-255">Data is identified and accessed using a single ID key, like a dictionary.</span></span></li>
            <li><span data-ttu-id="37acd-256">Высокая масштабируемость.</span><span class="sxs-lookup"><span data-stu-id="37acd-256">Massively scalable.</span></span></li>
            <li><span data-ttu-id="37acd-257">Соединения, блокировки или объединения не нужны.</span><span class="sxs-lookup"><span data-stu-id="37acd-257">No joins, lock, or unions are required.</span></span></li>
            <li><span data-ttu-id="37acd-258">Механизмы статистической обработки не используются.</span><span class="sxs-lookup"><span data-stu-id="37acd-258">No aggregation mechanisms are used.</span></span></li>
            <li><span data-ttu-id="37acd-259">Как правило, вторичные индексы не используются.</span><span class="sxs-lookup"><span data-stu-id="37acd-259">Secondary indexes are generally not used.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-260"><strong>Тип данных</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-260"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-261">Размер данных, как правило, большой.</span><span class="sxs-lookup"><span data-stu-id="37acd-261">Data size tends to be large.</span></span></li>
            <li><span data-ttu-id="37acd-262">Каждый ключ связан с одним значением, которое является неуправляемым большим двоичным объектом данных.</span><span class="sxs-lookup"><span data-stu-id="37acd-262">Each key is associated with a single value, which is an unmanaged data BLOB.</span></span></li>
            <li><span data-ttu-id="37acd-263">Схема не применяется.</span><span class="sxs-lookup"><span data-stu-id="37acd-263">There is no schema enforcement.</span></span></li>
            <li><span data-ttu-id="37acd-264">Сущности не связаны.</span><span class="sxs-lookup"><span data-stu-id="37acd-264">No relationships between entities.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-265"><strong>Примеры</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-265"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-266">Кэширование данных</span><span class="sxs-lookup"><span data-stu-id="37acd-266">Data caching</span></span></li>
            <li><span data-ttu-id="37acd-267">Управление сеансом</span><span class="sxs-lookup"><span data-stu-id="37acd-267">Session management</span></span></li>
            <li><span data-ttu-id="37acd-268">Управления параметрами и профилями пользователя</span><span class="sxs-lookup"><span data-stu-id="37acd-268">User preference and profile management</span></span></li>
            <li><span data-ttu-id="37acd-269">Рекомендации по продуктам и реклама</span><span class="sxs-lookup"><span data-stu-id="37acd-269">Product recommendation and ad serving</span></span></li>
            <li><span data-ttu-id="37acd-270">Словари</span><span class="sxs-lookup"><span data-stu-id="37acd-270">Dictionaries</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="graph-databases"></a><span data-ttu-id="37acd-271">Базы данных графов</span><span class="sxs-lookup"><span data-stu-id="37acd-271">Graph databases</span></span>

<table>
<tr><td><span data-ttu-id="37acd-272"><strong>Рабочая нагрузка</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-272"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-273">Связи между элементами данных очень сложные и включают много переходов между связанными элементами данных.</span><span class="sxs-lookup"><span data-stu-id="37acd-273">The relationships between data items are very complex, involving many hops between related data items.</span></span></li>
            <li><span data-ttu-id="37acd-274">Связи между элементами данных динамические и изменяются со временем.</span><span class="sxs-lookup"><span data-stu-id="37acd-274">The relationship between data items are dynamic and change over time.</span></span></li>
            <li><span data-ttu-id="37acd-275">Отношения между объектами являются привилегированными. Для обхода не требуются внешние ключи и соединения.</span><span class="sxs-lookup"><span data-stu-id="37acd-275">Relationships between objects are first-class citizens, without requiring foreign-keys and joins to traverse.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-276"><strong>Тип данных</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-276"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-277">Данные состоят из узлов и связей.</span><span class="sxs-lookup"><span data-stu-id="37acd-277">Data is comprised of nodes and relationships.</span></span></li>
            <li><span data-ttu-id="37acd-278">Узлы похожи на строки таблицы или документы JSON.</span><span class="sxs-lookup"><span data-stu-id="37acd-278">Nodes are similar to table rows or JSON documents.</span></span></li>
            <li><span data-ttu-id="37acd-279">Связи так же важны, как и узлы, и явно предоставляются на языке запросов.</span><span class="sxs-lookup"><span data-stu-id="37acd-279">Relationships are just as important as nodes, and are exposed directly in the query language.</span></span></li>
            <li><span data-ttu-id="37acd-280">Составные объекты, такие как пользователь с несколькими телефонными номерами, как правило, разделяются на несколько отдельных небольших узлов в сочетании с переходными связями.</span><span class="sxs-lookup"><span data-stu-id="37acd-280">Composite objects, such as a person with multiple phone numbers, tend to be broken into separate, smaller nodes, combined with traversable relationships</span></span> </li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-281"><strong>Примеры</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-281"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-282">Организационные диаграммы</span><span class="sxs-lookup"><span data-stu-id="37acd-282">Organization charts</span></span></li>
            <li><span data-ttu-id="37acd-283">Графы социальных сетей</span><span class="sxs-lookup"><span data-stu-id="37acd-283">Social graphs</span></span></li>
            <li><span data-ttu-id="37acd-284">Обнаружение мошенничества.</span><span class="sxs-lookup"><span data-stu-id="37acd-284">Fraud detection</span></span></li>
            <li><span data-ttu-id="37acd-285">Analytics</span><span class="sxs-lookup"><span data-stu-id="37acd-285">Analytics</span></span></li>
            <li><span data-ttu-id="37acd-286">Системы рекомендаций</span><span class="sxs-lookup"><span data-stu-id="37acd-286">Recommendation engines</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="column-family-databases"></a><span data-ttu-id="37acd-287">Базы данных столбцов</span><span class="sxs-lookup"><span data-stu-id="37acd-287">Column-family databases</span></span>

<table>
<tr><td><span data-ttu-id="37acd-288"><strong>Рабочая нагрузка</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-288"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-289">В большинстве баз данных столбцов операции записи выполняются очень быстро.</span><span class="sxs-lookup"><span data-stu-id="37acd-289">Most column-family databases perform write operations extremely quickly.</span></span></li>
            <li><span data-ttu-id="37acd-290">Операции обновления и удаления выполняются редко.</span><span class="sxs-lookup"><span data-stu-id="37acd-290">Update and delete operations are rare.</span></span></li>
            <li><span data-ttu-id="37acd-291">Предназначены для обеспечения доступа с высокой пропускной способностью и малой задержкой.</span><span class="sxs-lookup"><span data-stu-id="37acd-291">Designed to provide high throughput and low-latency access.</span></span></li>
            <li><span data-ttu-id="37acd-292">Поддерживают простой доступ с выполнением запроса к конкретному набору полей в большой записи.</span><span class="sxs-lookup"><span data-stu-id="37acd-292">Supports easy query access to a particular set of fields within a much larger record.</span></span></li>
            <li><span data-ttu-id="37acd-293">Высокая масштабируемость.</span><span class="sxs-lookup"><span data-stu-id="37acd-293">Massively scalable.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-294"><strong>Тип данных</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-294"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-295">Данные хранятся в таблицах, состоящих из ключевого столбца и одного или нескольких наборов столбцов.</span><span class="sxs-lookup"><span data-stu-id="37acd-295">Data is stored in tables consisting of a key column and one or more column families.</span></span></li>
            <li><span data-ttu-id="37acd-296">Определенные столбцы изменяются в зависимости от отдельных строк.</span><span class="sxs-lookup"><span data-stu-id="37acd-296">Specific columns can vary by individual rows.</span></span></li>
            <li><span data-ttu-id="37acd-297">Доступ к отдельным ячейкам осуществляется с использованием команд GET и PUT.</span><span class="sxs-lookup"><span data-stu-id="37acd-297">Individual cells are accessed via get and put commands</span></span></li>
            <li><span data-ttu-id="37acd-298">Несколько строк возвращаются с использованием команды проверки.</span><span class="sxs-lookup"><span data-stu-id="37acd-298">Multiple rows are returned using a scan command.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-299"><strong>Примеры</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-299"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-300">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="37acd-300">Recommendations</span></span></li>
            <li><span data-ttu-id="37acd-301">Персонализация</span><span class="sxs-lookup"><span data-stu-id="37acd-301">Personalization</span></span></li>
            <li><span data-ttu-id="37acd-302">Данные от датчиков</span><span class="sxs-lookup"><span data-stu-id="37acd-302">Sensor data</span></span></li>
            <li><span data-ttu-id="37acd-303">Телеметрия</span><span class="sxs-lookup"><span data-stu-id="37acd-303">Telemetry</span></span></li>
            <li><span data-ttu-id="37acd-304">Обмен сообщениями</span><span class="sxs-lookup"><span data-stu-id="37acd-304">Messaging</span></span></li>
            <li><span data-ttu-id="37acd-305">Анализ социальных сетей</span><span class="sxs-lookup"><span data-stu-id="37acd-305">Social media analytics</span></span></li>
            <li><span data-ttu-id="37acd-306">Веб-аналитика</span><span class="sxs-lookup"><span data-stu-id="37acd-306">Web analytics</span></span></li>
            <li><span data-ttu-id="37acd-307">Мониторинг активности</span><span class="sxs-lookup"><span data-stu-id="37acd-307">Activity monitoring</span></span></li>
            <li><span data-ttu-id="37acd-308">Прогнозные данные и другие данные временных рядов</span><span class="sxs-lookup"><span data-stu-id="37acd-308">Weather and other time-series data</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="search-engine-databases"></a><span data-ttu-id="37acd-309">Базы данных поисковой системы</span><span class="sxs-lookup"><span data-stu-id="37acd-309">Search engine databases</span></span>

<table>
<tr><td><span data-ttu-id="37acd-310"><strong>Рабочая нагрузка</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-310"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-311">Индексирование данных из нескольких источников и служб.</span><span class="sxs-lookup"><span data-stu-id="37acd-311">Indexing data from multiple sources and services.</span></span></li>
            <li><span data-ttu-id="37acd-312">Запросы являются специализированными и могут быть сложными.</span><span class="sxs-lookup"><span data-stu-id="37acd-312">Queries are ad-hoc and can be complex.</span></span></li>
            <li><span data-ttu-id="37acd-313">Требуется статистическая обработка.</span><span class="sxs-lookup"><span data-stu-id="37acd-313">Requires aggregation.</span></span></li>
            <li><span data-ttu-id="37acd-314">Требуется полнотекстовый поиск.</span><span class="sxs-lookup"><span data-stu-id="37acd-314">Full text search is required.</span></span></li>
            <li><span data-ttu-id="37acd-315">Требуется специализированный запрос на самообслуживание.</span><span class="sxs-lookup"><span data-stu-id="37acd-315">Ad hoc self-service query is required.</span></span></li>
            <li><span data-ttu-id="37acd-316">Требуется анализ данных с индексом по всем полям.</span><span class="sxs-lookup"><span data-stu-id="37acd-316">Data analysis with index on all fields is required.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-317"><strong>Тип данных</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-317"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-318">Полуструктурированные и неструктурированные данные</span><span class="sxs-lookup"><span data-stu-id="37acd-318">Semi-structured or unstructured</span></span></li>
            <li><span data-ttu-id="37acd-319">текст</span><span class="sxs-lookup"><span data-stu-id="37acd-319">Text</span></span></li>
            <li><span data-ttu-id="37acd-320">Текст со ссылкой на структурированные данные</span><span class="sxs-lookup"><span data-stu-id="37acd-320">Text with reference to structured data</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-321"><strong>Примеры</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-321"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-322">Каталоги продуктов</span><span class="sxs-lookup"><span data-stu-id="37acd-322">Product catalogs</span></span></li>
            <li><span data-ttu-id="37acd-323">Поиск на сайте</span><span class="sxs-lookup"><span data-stu-id="37acd-323">Site search</span></span></li>
            <li><span data-ttu-id="37acd-324">Ведение журналов</span><span class="sxs-lookup"><span data-stu-id="37acd-324">Logging</span></span></li>
            <li><span data-ttu-id="37acd-325">Analytics</span><span class="sxs-lookup"><span data-stu-id="37acd-325">Analytics</span></span></li>
            <li><span data-ttu-id="37acd-326">Сайты покупок</span><span class="sxs-lookup"><span data-stu-id="37acd-326">Shopping sites</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="data-warehouse"></a><span data-ttu-id="37acd-327">Хранилище данных</span><span class="sxs-lookup"><span data-stu-id="37acd-327">Data warehouse</span></span>

<table>
<tr><td><span data-ttu-id="37acd-328"><strong>Рабочая нагрузка</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-328"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-329">Аналитика данных</span><span class="sxs-lookup"><span data-stu-id="37acd-329">Data analytics</span></span></li>
            <li><span data-ttu-id="37acd-330">Корпоративная бизнес-аналитика</span><span class="sxs-lookup"><span data-stu-id="37acd-330">Enterprise BI</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-331"><strong>Тип данных</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-331"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-332">Исторические данные из нескольких источников.</span><span class="sxs-lookup"><span data-stu-id="37acd-332">Historical data from multiple sources.</span></span></li>
            <li><span data-ttu-id="37acd-333">Обычно денормализовано в схеме типа &quot;звезда&quot; или &quot;снежинка&quot;, состоящей из таблиц фактов и измерений.</span><span class="sxs-lookup"><span data-stu-id="37acd-333">Usually denormalized in a &quot;star&quot; or &quot;snowflake&quot; schema, consisting of fact and dimension tables.</span></span></li>
            <li><span data-ttu-id="37acd-334">Как правило, новые данные загружаются по расписанию.</span><span class="sxs-lookup"><span data-stu-id="37acd-334">Usually loaded with new data on a scheduled basis.</span></span></li>
            <li><span data-ttu-id="37acd-335">Таблицы измерений часто включают в себя несколько исторических версий сущности, называемых <em>медленно изменяющимися измерениями</em>.</span><span class="sxs-lookup"><span data-stu-id="37acd-335">Dimension tables often include multiple historic versions of an entity, referred to as a <em>slowly changing dimension</em>.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-336"><strong>Примеры</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-336"><strong>Examples</strong></span></span></td>
    <td><span data-ttu-id="37acd-337">Корпоративное хранилище данных, предоставляющее данные для аналитических моделей, отчетов и панелей мониторинга.</span><span class="sxs-lookup"><span data-stu-id="37acd-337">An enterprise data warehouse that provides data for analytical models, reports, and dashboards.</span></span>
    </td>
</tr>
</table>

## <a name="time-series-databases"></a><span data-ttu-id="37acd-338">Базы данных временных рядов</span><span class="sxs-lookup"><span data-stu-id="37acd-338">Time series databases</span></span>

<table>
<tr><td><span data-ttu-id="37acd-339"><strong>Рабочая нагрузка</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-339"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-340">Большая часть операций (95–99 %) — это операции записи.</span><span class="sxs-lookup"><span data-stu-id="37acd-340">An overwhelming proportion of operations (95-99%) are writes.</span></span></li>
            <li><span data-ttu-id="37acd-341">Записи обычно добавляются последовательно по времени.</span><span class="sxs-lookup"><span data-stu-id="37acd-341">Records are generally appended sequentially in time order.</span></span></li>
            <li><span data-ttu-id="37acd-342">Обновления происходят редко.</span><span class="sxs-lookup"><span data-stu-id="37acd-342">Updates are rare.</span></span></li>
            <li><span data-ttu-id="37acd-343">Удаление выполняется в пакетном режиме. Удаляются смежные блоки или записи.</span><span class="sxs-lookup"><span data-stu-id="37acd-343">Deletes occur in bulk, and are made to contiguous blocks or records.</span></span></li>
            <li><span data-ttu-id="37acd-344">Размер запросов на чтение может превышать доступный объем памяти.</span><span class="sxs-lookup"><span data-stu-id="37acd-344">Read requests can be larger than available memory.</span></span></li>
            <li><span data-ttu-id="37acd-345">Довольно часто несколько операций чтения выполняются одновременно.</span><span class="sxs-lookup"><span data-stu-id="37acd-345">It&#39;s common for multiple reads to occur simultaneously.</span></span></li>
            <li><span data-ttu-id="37acd-346">Данные считываются последовательно в порядке возрастания или убывания по времени.</span><span class="sxs-lookup"><span data-stu-id="37acd-346">Data is read sequentially in either ascending or descending time order.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-347"><strong>Тип данных</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-347"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-348">Метка времени используется в качестве первичного ключа и механизма сортировки.</span><span class="sxs-lookup"><span data-stu-id="37acd-348">A time stamp that is used as the primary key and sorting mechanism.</span></span></li>
            <li><span data-ttu-id="37acd-349">Измерения из записи или описания записи.</span><span class="sxs-lookup"><span data-stu-id="37acd-349">Measurements from the entry or descriptions of what the entry represents.</span></span></li>
            <li><span data-ttu-id="37acd-350">Теги, определяющие дополнительные сведения о типе, источнике и другие данные о записи.</span><span class="sxs-lookup"><span data-stu-id="37acd-350">Tags that define additional information about the type, origin, and other information about the entry.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-351"><strong>Примеры</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-351"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-352">Мониторинг и телеметрия событий.</span><span class="sxs-lookup"><span data-stu-id="37acd-352">Monitoring and event telemetry.</span></span></li>
            <li><span data-ttu-id="37acd-353">Данные датчиков или другие данные Интернета вещей.</span><span class="sxs-lookup"><span data-stu-id="37acd-353">Sensor or other IoT data.</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="object-storage"></a><span data-ttu-id="37acd-354">Хранилище объектов</span><span class="sxs-lookup"><span data-stu-id="37acd-354">Object storage</span></span>

<table>
<tr><td><span data-ttu-id="37acd-355"><strong>Рабочая нагрузка</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-355"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-356">Определяется ключом.</span><span class="sxs-lookup"><span data-stu-id="37acd-356">Identified by key.</span></span></li>
            <li><span data-ttu-id="37acd-357">Объекты могут быть общедоступными или частными.</span><span class="sxs-lookup"><span data-stu-id="37acd-357">Objects may be publicly or privately accessible.</span></span></li>
            <li><span data-ttu-id="37acd-358">Содержимое обычно является ресурсом, например электронная таблица, изображение или видеофайл.</span><span class="sxs-lookup"><span data-stu-id="37acd-358">Content is typically an asset such as a spreadsheet, image, or video file.</span></span></li>
            <li><span data-ttu-id="37acd-359">Содержимое должно быть устойчивыми (постоянным) и внешним для любого уровня приложения или виртуальной машины.</span><span class="sxs-lookup"><span data-stu-id="37acd-359">Content must be durable (persistent), and external to any application tier or virtual machine.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-360"><strong>Тип данных</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-360"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-361">Данные большого размера.</span><span class="sxs-lookup"><span data-stu-id="37acd-361">Data size is large.</span></span></li>
            <li><span data-ttu-id="37acd-362">Данные большого двоичного объекта.</span><span class="sxs-lookup"><span data-stu-id="37acd-362">Blob data.</span></span></li>
            <li><span data-ttu-id="37acd-363">Значение является непрозрачным.</span><span class="sxs-lookup"><span data-stu-id="37acd-363">Value is opaque.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-364"><strong>Примеры</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-364"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-365">Изображения, видео, офисные документы, PDF-файлы</span><span class="sxs-lookup"><span data-stu-id="37acd-365">Images, videos, office documents, PDFs</span></span></li>
            <li><span data-ttu-id="37acd-366">CSS, сценарии, CSV</span><span class="sxs-lookup"><span data-stu-id="37acd-366">CSS, Scripts, CSV</span></span></li>
            <li><span data-ttu-id="37acd-367">Статический HTML, JSON</span><span class="sxs-lookup"><span data-stu-id="37acd-367">Static HTML, JSON</span></span></li>
            <li><span data-ttu-id="37acd-368">Файлы журнала и аудита</span><span class="sxs-lookup"><span data-stu-id="37acd-368">Log and audit files</span></span></li>
            <li><span data-ttu-id="37acd-369">Резервные копии базы данных</span><span class="sxs-lookup"><span data-stu-id="37acd-369">Database backups</span></span></li>
        </ul>
    </td>
</tr>
</table>

## <a name="shared-files"></a><span data-ttu-id="37acd-370">Общие файлы</span><span class="sxs-lookup"><span data-stu-id="37acd-370">Shared files</span></span>

<table>
<tr><td><span data-ttu-id="37acd-371"><strong>Рабочая нагрузка</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-371"><strong>Workload</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-372">Миграция из имеющихся приложений, взаимодействующих с файловой системой.</span><span class="sxs-lookup"><span data-stu-id="37acd-372">Migration from existing apps that interact with the file system.</span></span></li>
            <li><span data-ttu-id="37acd-373">Требуется интерфейс SMB.</span><span class="sxs-lookup"><span data-stu-id="37acd-373">Requires SMB interface.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-374"><strong>Тип данных</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-374"><strong>Data type</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-375">Файлы в наборе папок иерархической структуры.</span><span class="sxs-lookup"><span data-stu-id="37acd-375">Files in a hierarchical set of folders.</span></span></li>
            <li><span data-ttu-id="37acd-376">Доступны в стандартных библиотеках ввода-вывода.</span><span class="sxs-lookup"><span data-stu-id="37acd-376">Accessible with standard I/O libraries.</span></span></li>
        </ul>
    </td>
</tr>
<tr><td><span data-ttu-id="37acd-377"><strong>Примеры</strong></span><span class="sxs-lookup"><span data-stu-id="37acd-377"><strong>Examples</strong></span></span></td>
    <td>
        <ul>
            <li><span data-ttu-id="37acd-378">Устаревшие файлы</span><span class="sxs-lookup"><span data-stu-id="37acd-378">Legacy files</span></span></li>
            <li><span data-ttu-id="37acd-379">Общее содержимое доступно в нескольких виртуальных машинах или экземплярах приложения</span><span class="sxs-lookup"><span data-stu-id="37acd-379">Shared content accessible among a number of VMs or app instances</span></span></li>
        </ul>
    </td>
</tr>
</table>

<!-- markdownlint-enable MD033 -->
