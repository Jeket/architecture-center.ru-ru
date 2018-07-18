---
title: Конкретные рекомендации по использованию механизма повторов
description: Конкретные рекомендации по настройке механизма повторов.
author: dragon119
ms.date: 07/13/2016
pnp.series.title: Best Practices
ms.openlocfilehash: 39d342dc96e3d0d923ce159c392d9427359a4639
ms.sourcegitcommit: f7fa67e3bdbc57d368edb67bac0e1fdec63695d2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/05/2018
ms.locfileid: "37843632"
---
# <a name="retry-guidance-for-specific-services"></a><span data-ttu-id="e7c8f-103">Руководство по использованию механизма повторов для отдельных служб</span><span class="sxs-lookup"><span data-stu-id="e7c8f-103">Retry guidance for specific services</span></span>

<span data-ttu-id="e7c8f-104">Большинство служб Azure и клиентских пакетов SDK содержат механизм повтора.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-104">Most Azure services and client SDKs include a retry mechanism.</span></span> <span data-ttu-id="e7c8f-105">В то же время они отличаются, поскольку каждая служба имеет разные характеристики и требования, поэтому каждый механизм повтора настроен для конкретной службы.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-105">However, these differ because each service has different characteristics and requirements, and so each retry mechanism is tuned to a specific service.</span></span> <span data-ttu-id="e7c8f-106">В этом руководстве представлены возможности механизма повтора для большинства служб Azure, а также сведения, помогающие использовать, адаптировать или расширять механизм повтора для такой службы.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-106">This guide summarizes the retry mechanism features for the majority of Azure services, and includes information to help you use, adapt, or extend the retry mechanism for that service.</span></span>

<span data-ttu-id="e7c8f-107">Для получения общих рекомендаций по обработке временных сбоев и выполнения повторных попыток подключения и операций со службами и ресурсами обратитесь к разделу [Руководство по повторам](./transient-faults.md).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-107">For general guidance on handling transient faults, and retrying connections and operations against services and resources, see [Retry guidance](./transient-faults.md).</span></span>

<span data-ttu-id="e7c8f-108">В следующей таблице перечислены функции механизма повтора для служб Azure, описанных в этом руководстве.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-108">The following table summarizes the retry features for the Azure services described in this guidance.</span></span>

| <span data-ttu-id="e7c8f-109">**Служба**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-109">**Service**</span></span> | <span data-ttu-id="e7c8f-110">**Ключевые возможности**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-110">**Retry capabilities**</span></span> | <span data-ttu-id="e7c8f-111">**Настройки политики**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-111">**Policy configuration**</span></span> | <span data-ttu-id="e7c8f-112">**Область**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-112">**Scope**</span></span> | <span data-ttu-id="e7c8f-113">**Функции телеметрии**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-113">**Telemetry features**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="e7c8f-114">**[Azure Active Directory](#azure-active-directory)**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-114">**[Azure Active Directory](#azure-active-directory)**</span></span> |<span data-ttu-id="e7c8f-115">Машинный код в библиотеке ADAL</span><span class="sxs-lookup"><span data-stu-id="e7c8f-115">Native in ADAL library</span></span> |<span data-ttu-id="e7c8f-116">Встроена в библиотеку ADAL</span><span class="sxs-lookup"><span data-stu-id="e7c8f-116">Embeded into ADAL library</span></span> |<span data-ttu-id="e7c8f-117">Внутренний</span><span class="sxs-lookup"><span data-stu-id="e7c8f-117">Internal</span></span> |<span data-ttu-id="e7c8f-118">None</span><span class="sxs-lookup"><span data-stu-id="e7c8f-118">None</span></span> |
| <span data-ttu-id="e7c8f-119">**[Cosmos DB](#cosmos-db)**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-119">**[Cosmos DB](#cosmos-db)**</span></span> |<span data-ttu-id="e7c8f-120">Машинный код в службе</span><span class="sxs-lookup"><span data-stu-id="e7c8f-120">Native in service</span></span> |<span data-ttu-id="e7c8f-121">Ненастраиваемые</span><span class="sxs-lookup"><span data-stu-id="e7c8f-121">Non-configurable</span></span> |<span data-ttu-id="e7c8f-122">Глобальные</span><span class="sxs-lookup"><span data-stu-id="e7c8f-122">Global</span></span> |<span data-ttu-id="e7c8f-123">TraceSource</span><span class="sxs-lookup"><span data-stu-id="e7c8f-123">TraceSource</span></span> |
| <span data-ttu-id="e7c8f-124">**[Концентраторы событий](#azure-event-hubs)**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-124">**[Event Hubs](#azure-event-hubs)**</span></span> |<span data-ttu-id="e7c8f-125">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="e7c8f-125">Native in client</span></span> |<span data-ttu-id="e7c8f-126">Программный</span><span class="sxs-lookup"><span data-stu-id="e7c8f-126">Programmatic</span></span> |<span data-ttu-id="e7c8f-127">Клиент</span><span class="sxs-lookup"><span data-stu-id="e7c8f-127">Client</span></span> |<span data-ttu-id="e7c8f-128">None</span><span class="sxs-lookup"><span data-stu-id="e7c8f-128">None</span></span> |
| <span data-ttu-id="e7c8f-129">**[Кэш Redis](#azure-redis-cache)**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-129">**[Redis Cache](#azure-redis-cache)**</span></span> |<span data-ttu-id="e7c8f-130">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="e7c8f-130">Native in client</span></span> |<span data-ttu-id="e7c8f-131">Программный</span><span class="sxs-lookup"><span data-stu-id="e7c8f-131">Programmatic</span></span> |<span data-ttu-id="e7c8f-132">Клиент</span><span class="sxs-lookup"><span data-stu-id="e7c8f-132">Client</span></span> |<span data-ttu-id="e7c8f-133">TextWriter</span><span class="sxs-lookup"><span data-stu-id="e7c8f-133">TextWriter</span></span> |
| <span data-ttu-id="e7c8f-134">**[Служба поиска](#azure-search)**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-134">**[Search](#azure-search)**</span></span> |<span data-ttu-id="e7c8f-135">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="e7c8f-135">Native in client</span></span> |<span data-ttu-id="e7c8f-136">Программный</span><span class="sxs-lookup"><span data-stu-id="e7c8f-136">Programmatic</span></span> |<span data-ttu-id="e7c8f-137">Клиент</span><span class="sxs-lookup"><span data-stu-id="e7c8f-137">Client</span></span> |<span data-ttu-id="e7c8f-138">Трассировка событий Windows или пользовательская</span><span class="sxs-lookup"><span data-stu-id="e7c8f-138">ETW or Custom</span></span> |
| <span data-ttu-id="e7c8f-139">**[Служебная шина](#service-bus)**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-139">**[Service Bus](#service-bus)**</span></span> |<span data-ttu-id="e7c8f-140">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="e7c8f-140">Native in client</span></span> |<span data-ttu-id="e7c8f-141">Программный</span><span class="sxs-lookup"><span data-stu-id="e7c8f-141">Programmatic</span></span> |<span data-ttu-id="e7c8f-142">Диспетчер пространств имен, фабрика сообщений и клиент</span><span class="sxs-lookup"><span data-stu-id="e7c8f-142">Namespace Manager, Messaging Factory, and Client</span></span> |<span data-ttu-id="e7c8f-143">Трассировка событий Windows</span><span class="sxs-lookup"><span data-stu-id="e7c8f-143">ETW</span></span> |
| <span data-ttu-id="e7c8f-144">**[Service Fabric](#service-fabric)**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-144">**[Service Fabric](#service-fabric)**</span></span> |<span data-ttu-id="e7c8f-145">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="e7c8f-145">Native in client</span></span> |<span data-ttu-id="e7c8f-146">Программный</span><span class="sxs-lookup"><span data-stu-id="e7c8f-146">Programmatic</span></span> |<span data-ttu-id="e7c8f-147">Клиент</span><span class="sxs-lookup"><span data-stu-id="e7c8f-147">Client</span></span> |<span data-ttu-id="e7c8f-148">None</span><span class="sxs-lookup"><span data-stu-id="e7c8f-148">None</span></span> | 
| <span data-ttu-id="e7c8f-149">**[База данных SQL с ADO.NET](#sql-database-using-adonet)**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-149">**[SQL Database with ADO.NET](#sql-database-using-adonet)**</span></span> |[<span data-ttu-id="e7c8f-150">Polly</span><span class="sxs-lookup"><span data-stu-id="e7c8f-150">Polly</span></span>](#transient-fault-handling-with-polly) |<span data-ttu-id="e7c8f-151">Декларативные и программные</span><span class="sxs-lookup"><span data-stu-id="e7c8f-151">Declarative and programmatic</span></span> |<span data-ttu-id="e7c8f-152">Единые инструкции или блоки кода</span><span class="sxs-lookup"><span data-stu-id="e7c8f-152">Single statements or blocks of code</span></span> |<span data-ttu-id="e7c8f-153">Пользовательская</span><span class="sxs-lookup"><span data-stu-id="e7c8f-153">Custom</span></span> |
| <span data-ttu-id="e7c8f-154">**[База данных SQL с Entity Framework](#sql-database-using-entity-framework-6)**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-154">**[SQL Database with Entity Framework](#sql-database-using-entity-framework-6)**</span></span> |<span data-ttu-id="e7c8f-155">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="e7c8f-155">Native in client</span></span> |<span data-ttu-id="e7c8f-156">Программный</span><span class="sxs-lookup"><span data-stu-id="e7c8f-156">Programmatic</span></span> |<span data-ttu-id="e7c8f-157">Глобальные каждого домена приложения</span><span class="sxs-lookup"><span data-stu-id="e7c8f-157">Global per AppDomain</span></span> |<span data-ttu-id="e7c8f-158">None</span><span class="sxs-lookup"><span data-stu-id="e7c8f-158">None</span></span> |
| <span data-ttu-id="e7c8f-159">**[База данных SQL с Entity Framework Core](#sql-database-using-entity-framework-core)**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-159">**[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core)**</span></span> |<span data-ttu-id="e7c8f-160">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="e7c8f-160">Native in client</span></span> |<span data-ttu-id="e7c8f-161">Программный</span><span class="sxs-lookup"><span data-stu-id="e7c8f-161">Programmatic</span></span> |<span data-ttu-id="e7c8f-162">Глобальные каждого домена приложения</span><span class="sxs-lookup"><span data-stu-id="e7c8f-162">Global per AppDomain</span></span> |<span data-ttu-id="e7c8f-163">None</span><span class="sxs-lookup"><span data-stu-id="e7c8f-163">None</span></span> |
| <span data-ttu-id="e7c8f-164">**[Хранилище](#azure-storage)**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-164">**[Storage](#azure-storage)**</span></span> |<span data-ttu-id="e7c8f-165">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="e7c8f-165">Native in client</span></span> |<span data-ttu-id="e7c8f-166">Программный</span><span class="sxs-lookup"><span data-stu-id="e7c8f-166">Programmatic</span></span> |<span data-ttu-id="e7c8f-167">Клиентские и индивидуальные операции</span><span class="sxs-lookup"><span data-stu-id="e7c8f-167">Client and individual operations</span></span> |<span data-ttu-id="e7c8f-168">TraceSource</span><span class="sxs-lookup"><span data-stu-id="e7c8f-168">TraceSource</span></span> |

> [!NOTE]
> <span data-ttu-id="e7c8f-169">Сейчас для большинства встроенных механизмов повтора Azure невозможно применить другую политику повтора для других типов ошибок и исключений.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-169">For most of the Azure built-in retry mechanisms, there is currently no way apply a different retry policy for different types of error or exception.</span></span> <span data-ttu-id="e7c8f-170">Вам следует настроить политику, которая обеспечит оптимальный средний уровень производительности и доступности.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-170">You should configure a policy that provides the optimum average performance and availability.</span></span> <span data-ttu-id="e7c8f-171">Одним из способов настройки политики является анализ файлов журнала для определения типа возникающих временных сбоев.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-171">One way to fine-tune the policy is to analyze log files to determine the type of transient faults that are occurring.</span></span> 

## <a name="azure-active-directory"></a><span data-ttu-id="e7c8f-172">Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="e7c8f-172">Azure Active Directory</span></span>
<span data-ttu-id="e7c8f-173">Azure Active Directory (Azure AD) — это комплексное решение по управлению идентификацией и доступом к облаку, которое включает службы основных каталогов, функции расширенного управления удостоверениями и доступа к приложениям, а также средства обеспечения безопасности.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-173">Azure Active Directory (Azure AD) is a comprehensive identity and access management cloud solution that combines core directory services, advanced identity governance, security, and application access management.</span></span> <span data-ttu-id="e7c8f-174">Azure AD также предлагает разработчикам платформу управления удостоверениями для доставки элементов контроля доступа к приложениям на основе централизованной политики и правил.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-174">Azure AD also offers developers an identity management platform to deliver access control to their applications, based on centralized policy and rules.</span></span>

> [!NOTE]
> <span data-ttu-id="e7c8f-175">Для получения рекомендаций по удостоверению управляемой службы конечных точек см. статью [Использование управляемого удостоверения службы (MSI) виртуальной машины Azure для получения маркера](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling)</span><span class="sxs-lookup"><span data-stu-id="e7c8f-175">For retry guidance on Managed Service Identity endpoints, see [How to use an Azure VM Managed Service Identity (MSI) for token acquisition](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling).</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="e7c8f-176">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="e7c8f-176">Retry mechanism</span></span>
<span data-ttu-id="e7c8f-177">Библиотека аутентификации для Active Directory (ADAL) имеет встроенный механизм повторов для службы Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-177">There is a built-in retry mechanism for Azure Active Directory in the Active Directory Authentication Library (ADAL).</span></span> <span data-ttu-id="e7c8f-178">Чтобы избежать непредвиденных блокировок, мы рекомендуем разработчикам **не** применять повторные попытки в своих библиотеках и приложениях, а использовать для этого библиотеку ADAL.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-178">To avoid unexpected lockouts, we recommend that third party libraries and application code do **not** retry failed connections, but allow ADAL to handle retries.</span></span> 

### <a name="retry-usage-guidance"></a><span data-ttu-id="e7c8f-179">Руководство по использованию механизма повторов</span><span class="sxs-lookup"><span data-stu-id="e7c8f-179">Retry usage guidance</span></span>
<span data-ttu-id="e7c8f-180">При использовании Azure Active Directory придерживайтесь следующих рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-180">Consider the following guidelines when using Azure Active Directory:</span></span>

* <span data-ttu-id="e7c8f-181">Везде, где это возможно, используйте библиотеку ADAL и ее встроенный механизм повторных попыток.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-181">When possible, use the ADAL library and the built-in support for retries.</span></span>
* <span data-ttu-id="e7c8f-182">Если вы используете REST API для Azure Active Directory и получаете код результата 429 (слишком много запросов) или ошибку в диапазоне 5xx, повторите операцию.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-182">If you are using the REST API for Azure Active Directory, retry the operation if the result code is 429 (Too Many Requests) or an error in the 5xx range.</span></span> <span data-ttu-id="e7c8f-183">Не следует выполнять повтор при возникновении других ошибок.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-183">Do not retry for any other errors.</span></span>
* <span data-ttu-id="e7c8f-184">В пакетных сценариях с Azure Active Directory рекомендуется использование экспоненциальных политик отсрочки.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-184">An exponential back-off policy is recommended for use in batch scenarios with Azure Active Directory.</span></span>

<span data-ttu-id="e7c8f-185">Для начала установите следующие параметры для операций повтора.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-185">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="e7c8f-186">Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-186">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="e7c8f-187">**Контекст**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-187">**Context**</span></span> | <span data-ttu-id="e7c8f-188">**Максимальная задержка<br />примера целевого E2E**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-188">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="e7c8f-189">**Стратегия повторов**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-189">**Retry strategy**</span></span> | <span data-ttu-id="e7c8f-190">**Параметры**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-190">**Settings**</span></span> | <span data-ttu-id="e7c8f-191">**Значения**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-191">**Values**</span></span> | <span data-ttu-id="e7c8f-192">**Принцип работы**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-192">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="e7c8f-193">Интерактивный, пользовательский интерфейс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-193">Interactive, UI,</span></span><br /><span data-ttu-id="e7c8f-194">или передний план</span><span class="sxs-lookup"><span data-stu-id="e7c8f-194">or foreground</span></span> |<span data-ttu-id="e7c8f-195">2 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-195">2 sec</span></span> |<span data-ttu-id="e7c8f-196">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="e7c8f-196">FixedInterval</span></span> |<span data-ttu-id="e7c8f-197">Число повторных попыток</span><span class="sxs-lookup"><span data-stu-id="e7c8f-197">Retry count</span></span><br /><span data-ttu-id="e7c8f-198">Интервал попытки</span><span class="sxs-lookup"><span data-stu-id="e7c8f-198">Retry interval</span></span><br /><span data-ttu-id="e7c8f-199">Первый быстрый повтор</span><span class="sxs-lookup"><span data-stu-id="e7c8f-199">First fast retry</span></span> |<span data-ttu-id="e7c8f-200">3</span><span class="sxs-lookup"><span data-stu-id="e7c8f-200">3</span></span><br /><span data-ttu-id="e7c8f-201">500 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-201">500 ms</span></span><br /><span data-ttu-id="e7c8f-202">true</span><span class="sxs-lookup"><span data-stu-id="e7c8f-202">true</span></span> |<span data-ttu-id="e7c8f-203">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-203">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="e7c8f-204">Попытка 2 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-204">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="e7c8f-205">попытка 3 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-205">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="e7c8f-206">Фоновый</span><span class="sxs-lookup"><span data-stu-id="e7c8f-206">Background or</span></span><br /><span data-ttu-id="e7c8f-207">или пакетный</span><span class="sxs-lookup"><span data-stu-id="e7c8f-207">batch</span></span> |<span data-ttu-id="e7c8f-208">60 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-208">60 sec</span></span> |<span data-ttu-id="e7c8f-209">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="e7c8f-209">ExponentialBackoff</span></span> |<span data-ttu-id="e7c8f-210">Число повторных попыток</span><span class="sxs-lookup"><span data-stu-id="e7c8f-210">Retry count</span></span><br /><span data-ttu-id="e7c8f-211">Минимальная задержка</span><span class="sxs-lookup"><span data-stu-id="e7c8f-211">Min back-off</span></span><br /><span data-ttu-id="e7c8f-212">Максимальная задержка</span><span class="sxs-lookup"><span data-stu-id="e7c8f-212">Max back-off</span></span><br /><span data-ttu-id="e7c8f-213">Разностная задержка</span><span class="sxs-lookup"><span data-stu-id="e7c8f-213">Delta back-off</span></span><br /><span data-ttu-id="e7c8f-214">Первый быстрый повтор</span><span class="sxs-lookup"><span data-stu-id="e7c8f-214">First fast retry</span></span> |<span data-ttu-id="e7c8f-215">5</span><span class="sxs-lookup"><span data-stu-id="e7c8f-215">5</span></span><br /><span data-ttu-id="e7c8f-216">0 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-216">0 sec</span></span><br /><span data-ttu-id="e7c8f-217">60 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-217">60 sec</span></span><br /><span data-ttu-id="e7c8f-218">2 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-218">2 sec</span></span><br /><span data-ttu-id="e7c8f-219">false</span><span class="sxs-lookup"><span data-stu-id="e7c8f-219">false</span></span> |<span data-ttu-id="e7c8f-220">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-220">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="e7c8f-221">Попытка 2 — задержка 2 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-221">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="e7c8f-222">Попытка 3 — задержка 6 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-222">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="e7c8f-223">Попытка 4 — задержка 14 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-223">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="e7c8f-224">Попытка 5 — задержка 30 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-224">Attempt 5 - delay ~30 sec</span></span> |

### <a name="more-information"></a><span data-ttu-id="e7c8f-225">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="e7c8f-225">More information</span></span>
* <span data-ttu-id="e7c8f-226">[Библиотеки аутентификации Azure Active Directory][adal]</span><span class="sxs-lookup"><span data-stu-id="e7c8f-226">[Azure Active Directory Authentication Libraries][adal]</span></span>

## <a name="cosmos-db"></a><span data-ttu-id="e7c8f-227">База данных Cosmos</span><span class="sxs-lookup"><span data-stu-id="e7c8f-227">Cosmos DB</span></span>

<span data-ttu-id="e7c8f-228">Cosmos DB — это полностью управляемая база данных, которая поддерживает несколько моделей и данные JSON без схемы данных.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-228">Cosmos DB is a fully-managed multi-model database that supports schema-less JSON data.</span></span> <span data-ttu-id="e7c8f-229">Она обеспечивает настраиваемую и надежную производительность, собственную обработку транзакций на основе JavaScript и разработана для облачной службы с гибким масштабированием.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-229">It offers configurable and reliable performance, native JavaScript transactional processing, and is built for the cloud with elastic scale.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="e7c8f-230">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="e7c8f-230">Retry mechanism</span></span>
<span data-ttu-id="e7c8f-231">Класс `DocumentClient` автоматически осуществляет новую попытку после сбоя.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-231">The `DocumentClient` class automatically retries failed attempts.</span></span> <span data-ttu-id="e7c8f-232">Чтобы задать количество повторных попыток и максимальное время ожидания, настройте свойство [ConnectionPolicy.RetryOptions].</span><span class="sxs-lookup"><span data-stu-id="e7c8f-232">To set the number of retries and the maximum wait time, configure [ConnectionPolicy.RetryOptions].</span></span> <span data-ttu-id="e7c8f-233">Исключения клиента находятся за пределами действия политики повтора или не являются временными ошибками.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-233">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>

<span data-ttu-id="e7c8f-234">Если Cosmos DB выполняет для клиента регулирование, возвращается ошибка HTTP 429.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-234">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="e7c8f-235">Проверьте код состояния в `DocumentClientException`.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-235">Check the status code in the `DocumentClientException`.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="e7c8f-236">Настройки политики</span><span class="sxs-lookup"><span data-stu-id="e7c8f-236">Policy configuration</span></span>
<span data-ttu-id="e7c8f-237">В следующей таблице показаны значения по умолчанию для класса `RetryOptions`.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-237">The following table shows the default settings for the `RetryOptions` class.</span></span>

| <span data-ttu-id="e7c8f-238">Параметр</span><span class="sxs-lookup"><span data-stu-id="e7c8f-238">Setting</span></span> | <span data-ttu-id="e7c8f-239">Значение по умолчанию</span><span class="sxs-lookup"><span data-stu-id="e7c8f-239">Default value</span></span> | <span data-ttu-id="e7c8f-240">ОПИСАНИЕ</span><span class="sxs-lookup"><span data-stu-id="e7c8f-240">Description</span></span> |
| --- | --- | --- |
| <span data-ttu-id="e7c8f-241">MaxRetryAttemptsOnThrottledRequests</span><span class="sxs-lookup"><span data-stu-id="e7c8f-241">MaxRetryAttemptsOnThrottledRequests</span></span> |<span data-ttu-id="e7c8f-242">9</span><span class="sxs-lookup"><span data-stu-id="e7c8f-242">9</span></span> |<span data-ttu-id="e7c8f-243">Максимальное число повторных попыток, когда запрос завершается сбоем из-за ограничения частоты запросов для клиента в Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-243">The maximum number of retries if the request fails because Cosmos DB applied rate limiting on the client.</span></span> |
| <span data-ttu-id="e7c8f-244">MaxRetryWaitTimeInSeconds</span><span class="sxs-lookup"><span data-stu-id="e7c8f-244">MaxRetryWaitTimeInSeconds</span></span> |<span data-ttu-id="e7c8f-245">30</span><span class="sxs-lookup"><span data-stu-id="e7c8f-245">30</span></span> |<span data-ttu-id="e7c8f-246">Максимальное время повтора в секундах.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-246">The maximum retry time in seconds.</span></span> |

### <a name="example"></a><span data-ttu-id="e7c8f-247">Пример</span><span class="sxs-lookup"><span data-stu-id="e7c8f-247">Example</span></span>
```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a><span data-ttu-id="e7c8f-248">Телеметрия</span><span class="sxs-lookup"><span data-stu-id="e7c8f-248">Telemetry</span></span>
<span data-ttu-id="e7c8f-249">Повторные попытки регистрируются как неструктурированные сообщения трассировки в .NET **TraceSource**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-249">Retry attempts are logged as unstructured trace messages through a .NET **TraceSource**.</span></span> <span data-ttu-id="e7c8f-250">Необходимо настроить **TraceListener** для сбора данных о событиях и записи этих данных в журнал с подходящим назначением.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-250">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span>

<span data-ttu-id="e7c8f-251">Например, если добавить в файл App.config следующий фрагмент кода, трассировка будет создаваться в текстовом файле в том же расположении, что и исполняемый файл:</span><span class="sxs-lookup"><span data-stu-id="e7c8f-251">For example, if you add the following to your App.config file, traces will be generated in a text file in the same location as the executable:</span></span>

```xml
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

## <a name="event-hubs"></a><span data-ttu-id="e7c8f-252">Концентраторы событий</span><span class="sxs-lookup"><span data-stu-id="e7c8f-252">Event Hubs</span></span>

<span data-ttu-id="e7c8f-253">Концентраторы событий Azure — это служба обработки данных телеметрии с высокой степенью масштабируемости. Она собирает, преобразовывает и хранит миллионы событий.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-253">Azure Event Hubs is a hyper-scale telemetry ingestion service that collects, transforms, and stores millions of events.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="e7c8f-254">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="e7c8f-254">Retry mechanism</span></span>
<span data-ttu-id="e7c8f-255">Механизм повторных попыток в клиентской библиотеке для концентраторов событий Azure управляется свойством `RetryPolicy` в классе `EventHubClient`.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-255">Retry behavior in the Azure Event Hubs Client Library is controlled by the `RetryPolicy` property on the `EventHubClient` class.</span></span> <span data-ttu-id="e7c8f-256">По умолчанию, если концентратор Azure возвращает временную ошибку `EventHubsException` или `OperationCanceledException`, используется политика повторных попыток с экспоненциальным увеличением задержки.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-256">The default policy retries with exponential backoff when Azure Event Hub returns a transient `EventHubsException` or an `OperationCanceledException`.</span></span>

### <a name="example"></a><span data-ttu-id="e7c8f-257">Пример</span><span class="sxs-lookup"><span data-stu-id="e7c8f-257">Example</span></span>
```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a><span data-ttu-id="e7c8f-258">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="e7c8f-258">More information</span></span>
[<span data-ttu-id="e7c8f-259">Клиентская библиотека .NET Standard для концентраторов событий Azure</span><span class="sxs-lookup"><span data-stu-id="e7c8f-259"> .NET Standard client library for Azure Event Hubs</span></span>](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="azure-redis-cache"></a><span data-ttu-id="e7c8f-260">кэш Azure Redis</span><span class="sxs-lookup"><span data-stu-id="e7c8f-260">Azure Redis Cache</span></span>
<span data-ttu-id="e7c8f-261">Кэш Redis для Azure является средством быстрого доступа к данным и службой кэша с низкой задержкой, созданной на основе кэша с открытым исходным кодом Redis.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-261">Azure Redis Cache is a fast data access and low latency cache service based on the popular open source Redis Cache.</span></span> <span data-ttu-id="e7c8f-262">Он является безопасным, управляется Майкрософт и доступен из любого приложения в Azure.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-262">It is secure, managed by Microsoft, and is accessible from any application in Azure.</span></span>

<span data-ttu-id="e7c8f-263">Рекомендации в этом разделе основаны на использовании клиента StackExchange.Redis для доступа к кэшу.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-263">The guidance in this section is based on using the StackExchange.Redis client to access the cache.</span></span> <span data-ttu-id="e7c8f-264">Список других подходящих клиентов можно найти на [веб-сайте Redis](http://redis.io/clients), и они могут применять различные механизмы.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-264">A list of other suitable clients can be found on the [Redis website](http://redis.io/clients), and these may have different retry mechanisms.</span></span>

<span data-ttu-id="e7c8f-265">Обратите внимание, что клиент StackExchange.Redis использует мультиплексирование через одно подключение.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-265">Note that the StackExchange.Redis client uses multiplexing through a single connection.</span></span> <span data-ttu-id="e7c8f-266">Рекомендуется создавать экземпляр клиента при запуске приложения и использовать этот экземпляр для всех операций в кэше.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-266">The recommended usage is to create an instance of the client at application startup and use this instance for all operations against the cache.</span></span> <span data-ttu-id="e7c8f-267">По этой причине подключение к кэшу происходит только один раз, и поэтому все инструкции в этом разделе относятся к политике повтора для этого исходного соединения, а не для каждой операции, которая обращается к кэшу.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-267">For this reason, the connection to the cache is made only once, and so all of the guidance in this section is related to the retry policy for this initial connection—and not for each operation that accesses the cache.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="e7c8f-268">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="e7c8f-268">Retry mechanism</span></span>
<span data-ttu-id="e7c8f-269">Клиент StackExchange.Redis использует класс диспетчера подключений, для настройки которого доступен ряд параметров:</span><span class="sxs-lookup"><span data-stu-id="e7c8f-269">The StackExchange.Redis client uses a connection manager class that is configured through a set of options, incuding:</span></span>

- <span data-ttu-id="e7c8f-270">**ConnectRetry**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-270">**ConnectRetry**.</span></span> <span data-ttu-id="e7c8f-271">Число повторных попыток при сбое подключения к кэшу.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-271">The number of times a failed connection to the cache will be retried.</span></span>
- <span data-ttu-id="e7c8f-272">**ReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-272">**ReconnectRetryPolicy**.</span></span> <span data-ttu-id="e7c8f-273">Используемая стратегия повторных попыток.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-273">The retry strategy to use.</span></span>
- <span data-ttu-id="e7c8f-274">**ConnectTimeout**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-274">**ConnectTimeout**.</span></span> <span data-ttu-id="e7c8f-275">Максимальное время ожидания в миллисекундах.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-275">The maximum waiting time in milliseconds.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="e7c8f-276">Настройки политики</span><span class="sxs-lookup"><span data-stu-id="e7c8f-276">Policy configuration</span></span>
<span data-ttu-id="e7c8f-277">Политики повтора настраиваются программным способом с помощью параметров клиента перед подключением к кэшу.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-277">Retry policies are configured programmatically by setting the options for the client before connecting to the cache.</span></span> <span data-ttu-id="e7c8f-278">Для этого нужно создать экземпляр класса **ConfigurationOptions**, заполнить его свойства и передать ему метод **Connect**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-278">This can be done by creating an instance of the **ConfigurationOptions** class, populating its properties, and passing it to the **Connect** method.</span></span>

<span data-ttu-id="e7c8f-279">Встроенные классы поддерживают линейные (постоянные) паузы и экспоненциальное увеличение задержки с псевдослучайными интервалами.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-279">The built-in classes support linear (constant) delay and exponential backoff with randomized retry intervals.</span></span> <span data-ttu-id="e7c8f-280">Также вы можете создать пользовательскую политику повторных попыток, реализовав интерфейс **IReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-280">You can also create a custom retry policy by implementing the **IReconnectRetryPolicy** interface.</span></span>

<span data-ttu-id="e7c8f-281">Код следующего примера настраивает стратегию повторных попыток с экспоненциальным увеличением задержки.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-281">The following example configures a retry strategy using exponential backoff.</span></span>

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

<span data-ttu-id="e7c8f-282">Кроме того, можно указать параметры в виде строки и передать эти данные методу **Connect** .</span><span class="sxs-lookup"><span data-stu-id="e7c8f-282">Alternatively, you can specify the options as a string, and pass this to the **Connect** method.</span></span> <span data-ttu-id="e7c8f-283">Обратите внимание, что свойство **ReconnectRetryPolicy** нельзя задать таким способом, а можно только с помощью кода.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-283">Note that the **ReconnectRetryPolicy** property cannot be set this way, only through code.</span></span>

```csharp
var options = "localhost,connectRetry=3,connectTimeout=2000";
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="e7c8f-284">Также вы можете указать параметры непосредственно при подключении к кэшу.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-284">You can also specify options directly when you connect to the cache.</span></span>

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

<span data-ttu-id="e7c8f-285">Дополнительные сведения см. в [документации по StackExchange.Redis](https://stackexchange.github.io/StackExchange.Redis/Configuration).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-285">For more information, see [Stack Exchange Redis Configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration) in the StackExchange.Redis documentation.</span></span>

<span data-ttu-id="e7c8f-286">Следующая таблица показывает значения по умолчанию для встроенной политики повторов.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-286">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="e7c8f-287">**Контекст**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-287">**Context**</span></span> | <span data-ttu-id="e7c8f-288">**Параметр**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-288">**Setting**</span></span> | <span data-ttu-id="e7c8f-289">**Значение по умолчанию**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-289">**Default value**</span></span><br /><span data-ttu-id="e7c8f-290">(версия 1.2.2)</span><span class="sxs-lookup"><span data-stu-id="e7c8f-290">(v 1.2.2)</span></span> | <span data-ttu-id="e7c8f-291">**Значение**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-291">**Meaning**</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="e7c8f-292">Параметры конфигурации</span><span class="sxs-lookup"><span data-stu-id="e7c8f-292">ConfigurationOptions</span></span> |<span data-ttu-id="e7c8f-293">ConnectRetry</span><span class="sxs-lookup"><span data-stu-id="e7c8f-293">ConnectRetry</span></span><br /><br /><span data-ttu-id="e7c8f-294">ConnectTimeout</span><span class="sxs-lookup"><span data-stu-id="e7c8f-294">ConnectTimeout</span></span><br /><br /><span data-ttu-id="e7c8f-295">SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="e7c8f-295">SyncTimeout</span></span><br /><br /><span data-ttu-id="e7c8f-296">ReconnectRetryPolicy</span><span class="sxs-lookup"><span data-stu-id="e7c8f-296">ReconnectRetryPolicy</span></span> |<span data-ttu-id="e7c8f-297">3</span><span class="sxs-lookup"><span data-stu-id="e7c8f-297">3</span></span><br /><br /><span data-ttu-id="e7c8f-298">Максимально 5000 мс плюс SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="e7c8f-298">Maximum 5000 ms plus SyncTimeout</span></span><br /><span data-ttu-id="e7c8f-299">1000</span><span class="sxs-lookup"><span data-stu-id="e7c8f-299">1000</span></span><br /><br /><span data-ttu-id="e7c8f-300">LinearRetry 5000 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-300">LinearRetry 5000 ms</span></span> |<span data-ttu-id="e7c8f-301">Количество попыток повторов подключения во время начального подключения.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-301">The number of times to repeat connect attempts during the initial connection operation.</span></span><br /><span data-ttu-id="e7c8f-302">Время ожидания (мс) для операций подключения.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-302">Timeout (ms) for connect operations.</span></span> <span data-ttu-id="e7c8f-303">Без задержки между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-303">Not a delay between retry attempts.</span></span><br /><span data-ttu-id="e7c8f-304">Время (мс) для синхронных операций.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-304">Time (ms) to allow for synchronous operations.</span></span><br /><br /><span data-ttu-id="e7c8f-305">Повторы через каждые 5000 мс.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-305">Retry every 5000 ms.</span></span>|

> [!NOTE]
> <span data-ttu-id="e7c8f-306">Для синхронных операций можно добавить `SyncTimeout` для определения совокупного времени задержки, но слишком низкое значение этого параметра приведет к чрезмерному времени ожидания.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-306">For synchronous operations, `SyncTimeout` can add to the end-to-end latency, but setting the value too low can cause excessive timeouts.</span></span> <span data-ttu-id="e7c8f-307">См. статью [Способы устранения проблем с кэшем Redis для Azure][redis-cache-troubleshoot].</span><span class="sxs-lookup"><span data-stu-id="e7c8f-307">See [How to troubleshoot Azure Redis Cache][redis-cache-troubleshoot].</span></span> <span data-ttu-id="e7c8f-308">Обычно следует избегать синхронных операций и использовать асинхронные везде, где это возможно.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-308">In general, avoid using synchronous operations, and use asynchronous operations instead.</span></span> <span data-ttu-id="e7c8f-309">Дополнительные сведения см. в статье [Конвейеры и мультиплексоры](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-309">For more information see [Pipelines and Multiplexers](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/PipelinesMultiplexers.md).</span></span>
>
>

### <a name="retry-usage-guidance"></a><span data-ttu-id="e7c8f-310">Руководство по использованию механизма повторов</span><span class="sxs-lookup"><span data-stu-id="e7c8f-310">Retry usage guidance</span></span>
<span data-ttu-id="e7c8f-311">При использовании кэша Redis для Azure, придерживайтесь следующих рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-311">Consider the following guidelines when using Azure Redis Cache:</span></span>

* <span data-ttu-id="e7c8f-312">Клиент StackExchange Redis управляет собственными операциями повтора, но только при установлении соединения с кэшем при первом запуске приложения.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-312">The StackExchange Redis client manages its own retries, but only when establishing a connection to the cache when the application first starts.</span></span> <span data-ttu-id="e7c8f-313">Вы можете настроить время ожидания подключения, число повторных попыток подключения и паузы между ними, но эта политика повторов не применяется к операциям с кэшем.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-313">You can configure the connection timeout, the number of retry attempts, and the time between retries to establish this connection, but the retry policy does not apply to operations against the cache.</span></span>
* <span data-ttu-id="e7c8f-314">Вместо использования большого числа повторных попыток рассмотрите возможность возврата к исходному источнику данных.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-314">Instead of using a large number of retry attempts, consider falling back by accessing the original data source instead.</span></span>

### <a name="telemetry"></a><span data-ttu-id="e7c8f-315">Телеметрия</span><span class="sxs-lookup"><span data-stu-id="e7c8f-315">Telemetry</span></span>
<span data-ttu-id="e7c8f-316">Можно собирать сведения о подключении (но не о других операциях) с помощью **TextWriter**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-316">You can collect information about connections (but not other operations) using a **TextWriter**.</span></span>

```csharp
var writer = new StringWriter();
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="e7c8f-317">Ниже показан пример генерируемых выходных данных.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-317">An example of the output this generates is shown below.</span></span>

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

### <a name="examples"></a><span data-ttu-id="e7c8f-318">Примеры</span><span class="sxs-lookup"><span data-stu-id="e7c8f-318">Examples</span></span>
<span data-ttu-id="e7c8f-319">Следующий пример кода настраивает постоянную (линейную) задержку между повторными попытками при инициализации клиента StackExchange.Redis.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-319">The following code example configures a constant (linear) delay between retries when initializing the StackExchange.Redis client.</span></span> <span data-ttu-id="e7c8f-320">В этом примере показано, как настроить конфигурацию с помощью экземпляра **ConfigurationOptions**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-320">This example shows how to set the configuration using a **ConfigurationOptions** instance.</span></span>

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

<span data-ttu-id="e7c8f-321">В следующем примере показано, как настроить конфигурацию, указав параметры в строковом формате.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-321">The next example sets the configuration by specifying the options as a string.</span></span> <span data-ttu-id="e7c8f-322">Время ожидания подключения обозначает максимальный период времени для ожидания соединения с кэшем, но не задержку между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-322">The connection timeout is the maximum period of time to wait for a connection to the cache, not the delay between retry attempts.</span></span> <span data-ttu-id="e7c8f-323">Обратите внимание, что свойство **ReconnectRetryPolicy** можно задавать только в коде.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-323">Note that the **ReconnectRetryPolicy** property can only be set by code.</span></span>

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

<span data-ttu-id="e7c8f-324">Дополнительные примеры см. в разделе [Конфигурация](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) на веб-сайте проекта.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-324">For more examples, see [Configuration](http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md#configuration) on the project website.</span></span>

### <a name="more-information"></a><span data-ttu-id="e7c8f-325">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="e7c8f-325">More information</span></span>
* [<span data-ttu-id="e7c8f-326">Веб-сайт Redis</span><span class="sxs-lookup"><span data-stu-id="e7c8f-326">Redis website</span></span>](http://redis.io/)

## <a name="azure-search"></a><span data-ttu-id="e7c8f-327">Поиск Azure</span><span class="sxs-lookup"><span data-stu-id="e7c8f-327">Azure Search</span></span>
<span data-ttu-id="e7c8f-328">Поиск Azure является мощным и сложным инструментом поиска для сайта или приложения с возможностями быстрой и простой настройки результатов поиска и построения информативных и точных моделей ранжирования.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-328">Azure Search can be used to add powerful and sophisticated search capabilities to a website or application, quickly and easily tune search results, and construct rich and fine-tuned ranking models.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="e7c8f-329">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="e7c8f-329">Retry mechanism</span></span>
<span data-ttu-id="e7c8f-330">Поведение повтора в пакете SDK для Поиска Azure контролируется в методе `SetRetryPolicy` класса [SearchServiceClient] и [SearchIndexClient].</span><span class="sxs-lookup"><span data-stu-id="e7c8f-330">Retry behavior in the Azure Search SDK is controlled by the `SetRetryPolicy` method on the [SearchServiceClient] and [SearchIndexClient] classes.</span></span> <span data-ttu-id="e7c8f-331">Повтор политики по умолчанию осуществляется с экспоненциальной задержкой, если Поиск Azure возвращает ответ с кодом 5xx или 408 (истекло время ожидания запроса).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-331">The default policy retries with exponential backoff when Azure Search returns a 5xx or 408 (Request Timeout) response.</span></span>

### <a name="telemetry"></a><span data-ttu-id="e7c8f-332">Телеметрия</span><span class="sxs-lookup"><span data-stu-id="e7c8f-332">Telemetry</span></span>
<span data-ttu-id="e7c8f-333">Используйте трассировку событий Windows или осуществляйте трассировку путем регистрации пользовательского поставщика трассировки.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-333">Trace with ETW or by registering a custom trace provider.</span></span> <span data-ttu-id="e7c8f-334">Дополнительные сведения см. в [документации по AutoRest][autorest].</span><span class="sxs-lookup"><span data-stu-id="e7c8f-334">For more information, see the [AutoRest documentation][autorest].</span></span>

## <a name="service-bus"></a><span data-ttu-id="e7c8f-335">Служебная шина Azure</span><span class="sxs-lookup"><span data-stu-id="e7c8f-335">Service Bus</span></span>
<span data-ttu-id="e7c8f-336">Служебная шина является облачной платформой обмена сообщениями, которая предоставляет слабосвязанный обмен сообщениями с улучшенным масштабированием и устойчивостью для компонентов приложения, размещенных в облаке или локально.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-336">Service Bus is a cloud messaging platform that provides loosely coupled message exchange with improved scale and resiliency for components of an application, whether hosted in the cloud or on-premises.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="e7c8f-337">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="e7c8f-337">Retry mechanism</span></span>
<span data-ttu-id="e7c8f-338">Служебная шина реализует повторы с помощью реализации базового класса [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) .</span><span class="sxs-lookup"><span data-stu-id="e7c8f-338">Service Bus implements retries using implementations of the [RetryPolicy](http://msdn.microsoft.com/library/microsoft.servicebus.retrypolicy.aspx) base class.</span></span> <span data-ttu-id="e7c8f-339">Все клиенты служебной шины предоставляют свойство **RetryPolicy**, которое может быть присвоено одной из реализаций базового класса **RetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-339">All of the Service Bus clients expose a **RetryPolicy** property that can be set to one of the implementations of the **RetryPolicy** base class.</span></span> <span data-ttu-id="e7c8f-340">Встроенные реализации</span><span class="sxs-lookup"><span data-stu-id="e7c8f-340">The built-in implementations are:</span></span>

* <span data-ttu-id="e7c8f-341">[Класс RetryExponential](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-341">The [RetryExponential Class](http://msdn.microsoft.com/library/microsoft.servicebus.retryexponential.aspx).</span></span> <span data-ttu-id="e7c8f-342">Этот класс предоставляет свойства, управляющие интервалом отсрочки, числом попыток и свойством **TerminationTimeBuffer** , используемым для ограничения общего времени завершения операции.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-342">This exposes properties that control the back-off interval, the retry count, and the **TerminationTimeBuffer** property that is used to limit the total time for the operation to complete.</span></span>
* <span data-ttu-id="e7c8f-343">[Класс NoRetry](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-343">The [NoRetry Class](http://msdn.microsoft.com/library/microsoft.servicebus.noretry.aspx).</span></span> <span data-ttu-id="e7c8f-344">Используется, когда повторы на уровне API служебной шины не требуются, например, если повторные попытки управляются другим процессом в рамках пакета операций либо многоэтапной операции.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-344">This is used when retries at the Service Bus API level are not required, such as when retries are managed by another process as part of a batch or multiple step operation.</span></span>

<span data-ttu-id="e7c8f-345">Действия служебной шины могут вызвать ряд исключений, как указано в статье [Исключения обмена сообщениями служебной шины](/azure/service-bus-messaging/service-bus-messaging-exceptions).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-345">Service Bus actions can return a range of exceptions, as listed in [Service Bus messaging exceptions](/azure/service-bus-messaging/service-bus-messaging-exceptions).</span></span> <span data-ttu-id="e7c8f-346">Список предоставляет сведения о том, какие из исключений указывают, что повторение операции является целесообразным.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-346">The list provides information about which if these indicate that retrying the operation is appropriate.</span></span> <span data-ttu-id="e7c8f-347">Например **ServerBusyException** указывает, что клиенту следует подождать некоторое время и затем повторить операцию.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-347">For example, a **ServerBusyException** indicates that the client should wait for a period of time, then retry the operation.</span></span> <span data-ttu-id="e7c8f-348">Появление исключения **ServerBusyException** также вынуждает служебную шину переключиться в другой режим, в котором добавляется дополнительная 10-секундная задержка к вычисляемому времени задержки повтора операции.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-348">The occurrence of a **ServerBusyException** also causes Service Bus to switch to a different mode, in which an extra 10-second delay is added to the computed retry delays.</span></span> <span data-ttu-id="e7c8f-349">Этот режим сбрасывается после короткого времени.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-349">This mode is reset after a short period.</span></span>

<span data-ttu-id="e7c8f-350">Исключения, возвращаемые служебной шиной, предоставляют свойство **IsTransient** , указывающее, что клиент должен повторить операцию.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-350">The exceptions returned from Service Bus expose the **IsTransient** property that indicates if the client should retry the operation.</span></span> <span data-ttu-id="e7c8f-351">Встроенная политика **RetryExponential** зависит от свойства **IsTransient** класса **MessagingException**, который является базовым классом для всех исключений служебной шины.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-351">The built-in **RetryExponential** policy relies on the **IsTransient** property in the **MessagingException** class, which is the base class for all Service Bus exceptions.</span></span> <span data-ttu-id="e7c8f-352">Если создать пользовательские реализации базового класса **RetryPolicy**, то можно будет использовать сочетание типа исключения и свойство **IsTransient**, чтобы организовать более точное управление повторами.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-352">If you create custom implementations of the **RetryPolicy** base class you could use a combination of the exception type and the **IsTransient** property to provide more fine-grained control over retry actions.</span></span> <span data-ttu-id="e7c8f-353">Например, можно обнаружить исключение **QuotaExceededException** и принять меры по очистке очереди, прежде чем повторить отправку сообщения в очередь.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-353">For example, you could detect a **QuotaExceededException** and take action to drain the queue before retrying sending a message to it.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="e7c8f-354">Настройки политики</span><span class="sxs-lookup"><span data-stu-id="e7c8f-354">Policy configuration</span></span>
<span data-ttu-id="e7c8f-355">Политики повтора настраиваются программно и могут быть установлены в качестве политики по умолчанию для **NamespaceManager** и **MessagingFactory** или по отдельности для каждого клиента обмена сообщениями.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-355">Retry policies are set programmatically, and can be set as a default policy for a **NamespaceManager** and for a **MessagingFactory**, or individually for each messaging client.</span></span> <span data-ttu-id="e7c8f-356">Чтобы задать политику повтора по умолчанию для сеанса обмена сообщениями, можно установить свойство **RetryPolicy** класса **NamespaceManager**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-356">To set the default retry policy for a messaging session you set the **RetryPolicy** of the **NamespaceManager**.</span></span>

```csharp
namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                maxBackoff: TimeSpan.FromSeconds(30),
                                                                maxRetryCount: 3);
```

<span data-ttu-id="e7c8f-357">Чтобы задать политику повторов по умолчанию для всех клиентов, созданных из фабрики обмена сообщениями, установите свойство **RetryPolicy** класса **MessagingFactory**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-357">To set the default retry policy for all clients created from a messaging factory, you set the **RetryPolicy** of the **MessagingFactory**.</span></span>

```csharp
messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                    maxBackoff: TimeSpan.FromSeconds(30),
                                                    maxRetryCount: 3);
```

<span data-ttu-id="e7c8f-358">Чтобы задать политику повтора для клиента обмена сообщениями или переопределить политику по умолчанию, задайте свойство **RetryPolicy** с помощью экземпляра класса требуемой политики:</span><span class="sxs-lookup"><span data-stu-id="e7c8f-358">To set the retry policy for a messaging client, or to override its default policy, you set its **RetryPolicy** property using an instance of the required policy class:</span></span>

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

<span data-ttu-id="e7c8f-359">Нельзя задать политику повтора на уровне отдельных операций.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-359">The retry policy cannot be set at the individual operation level.</span></span> <span data-ttu-id="e7c8f-360">Она применяется для всех операций для клиента обмена сообщениями.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-360">It applies to all operations for the messaging client.</span></span>
<span data-ttu-id="e7c8f-361">Следующая таблица показывает значения по умолчанию для встроенной политики повторов.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-361">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="e7c8f-362">Параметр</span><span class="sxs-lookup"><span data-stu-id="e7c8f-362">Setting</span></span> | <span data-ttu-id="e7c8f-363">Значение по умолчанию</span><span class="sxs-lookup"><span data-stu-id="e7c8f-363">Default value</span></span> | <span data-ttu-id="e7c8f-364">Значение</span><span class="sxs-lookup"><span data-stu-id="e7c8f-364">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="e7c8f-365">Политика</span><span class="sxs-lookup"><span data-stu-id="e7c8f-365">Policy</span></span> | <span data-ttu-id="e7c8f-366">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="e7c8f-366">Exponential</span></span> | <span data-ttu-id="e7c8f-367">Экспоненциальная задержка.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-367">Exponential back-off.</span></span> |
| <span data-ttu-id="e7c8f-368">MinimalBackoff</span><span class="sxs-lookup"><span data-stu-id="e7c8f-368">MinimalBackoff</span></span> | <span data-ttu-id="e7c8f-369">0</span><span class="sxs-lookup"><span data-stu-id="e7c8f-369">0</span></span> | <span data-ttu-id="e7c8f-370">Минимальное время задержки.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-370">Minimum back-off interval.</span></span> <span data-ttu-id="e7c8f-371">Добавляется ко всем интервалам повтора, вычисленным на основе значения deltaBackoff.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-371">This is added to the retry interval computed from deltaBackoff.</span></span> |
| <span data-ttu-id="e7c8f-372">MaximumBackoff</span><span class="sxs-lookup"><span data-stu-id="e7c8f-372">MaximumBackoff</span></span> | <span data-ttu-id="e7c8f-373">30 секунд</span><span class="sxs-lookup"><span data-stu-id="e7c8f-373">30 seconds</span></span> | <span data-ttu-id="e7c8f-374">Максимальное время задержки.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-374">Maximum back-off interval.</span></span> <span data-ttu-id="e7c8f-375">MaxBackoff используется, если расчетный интервал повторных попыток превышает значение MaxBackoff.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-375">MaximumBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> |
| <span data-ttu-id="e7c8f-376">DeltaBackoff</span><span class="sxs-lookup"><span data-stu-id="e7c8f-376">DeltaBackoff</span></span> | <span data-ttu-id="e7c8f-377">3 секунды</span><span class="sxs-lookup"><span data-stu-id="e7c8f-377">3 seconds</span></span> | <span data-ttu-id="e7c8f-378">Интервал отсрочки между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-378">Back-off interval between retries.</span></span> <span data-ttu-id="e7c8f-379">Величина, кратная timespan, будет использоваться для последующих попыток.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-379">Multiples of this timespan will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="e7c8f-380">TimeBuffer</span><span class="sxs-lookup"><span data-stu-id="e7c8f-380">TimeBuffer</span></span> | <span data-ttu-id="e7c8f-381">5 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-381">5 seconds</span></span> | <span data-ttu-id="e7c8f-382">Буфер времени завершения, связанный с повторной попыткой.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-382">The termination time buffer associated with the retry.</span></span> <span data-ttu-id="e7c8f-383">Повторные попытки будут прерваны, если оставшееся время меньше значения TimeBuffer.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-383">Retry attempts will be abandoned if the remaining time is less than TimeBuffer.</span></span> |
| <span data-ttu-id="e7c8f-384">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="e7c8f-384">MaxRetryCount</span></span> | <span data-ttu-id="e7c8f-385">10</span><span class="sxs-lookup"><span data-stu-id="e7c8f-385">10</span></span> | <span data-ttu-id="e7c8f-386">Максимальное число попыток.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-386">The maximum number of retries.</span></span> |
| <span data-ttu-id="e7c8f-387">ServerBusyBaseSleepTime</span><span class="sxs-lookup"><span data-stu-id="e7c8f-387">ServerBusyBaseSleepTime</span></span> | <span data-ttu-id="e7c8f-388">10 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-388">10 seconds</span></span> | <span data-ttu-id="e7c8f-389">Если последнее исключение было **ServerBusyException**, это значение будет добавляться к интервалу вычисляемых значений повторных попыток.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-389">If the last exception encountered was **ServerBusyException**, this value will be added to the computed retry interval.</span></span> <span data-ttu-id="e7c8f-390">Это значение не может быть изменено.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-390">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="e7c8f-391">Руководство по использованию механизма повторов</span><span class="sxs-lookup"><span data-stu-id="e7c8f-391">Retry usage guidance</span></span>
<span data-ttu-id="e7c8f-392">При использовании служебной шины придерживайтесь следующих рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-392">Consider the following guidelines when using Service Bus:</span></span>

* <span data-ttu-id="e7c8f-393">При использовании встроенной реализации **RetryExponential** не реализуйте операции резервирования, поскольку политика реагирует на исключения типа «Сервер занят» и автоматически переключается в режим повтора.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-393">When using the built-in **RetryExponential** implementation, do not implement a fallback operation as the policy reacts to Server Busy exceptions and automatically switches to an appropriate retry mode.</span></span>
* <span data-ttu-id="e7c8f-394">Server Bus поддерживает функцию, называемую Сопряженные пространства имен, которая реализует автоматическое переключение на очередь резервного копирования в отдельном пространстве имен, если очередь в первичном пространстве имен дает сбой.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-394">Service Bus supports a feature called Paired Namespaces, which implements automatic failover to a backup queue in a separate namespace if the queue in the primary namespace fails.</span></span> <span data-ttu-id="e7c8f-395">Сообщения из вторичной очереди могут отправляться обратно в основную очередь после ее восстановления.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-395">Messages from the secondary queue can be sent back to the primary queue when it recovers.</span></span> <span data-ttu-id="e7c8f-396">Эта функция позволяет справиться с временными сбоями.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-396">This feature helps to address transient failures.</span></span> <span data-ttu-id="e7c8f-397">Для получения дополнительных сведений обратитесь к разделу [Асинхронные шаблоны обмена сообщениями и высокий уровень доступности](http://msdn.microsoft.com/library/azure/dn292562.aspx).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-397">For more information, see [Asynchronous Messaging Patterns and High Availability](http://msdn.microsoft.com/library/azure/dn292562.aspx).</span></span>

<span data-ttu-id="e7c8f-398">Начните с использования следующих параметров для операции повтора.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-398">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="e7c8f-399">Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-399">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="e7c8f-400">Context</span><span class="sxs-lookup"><span data-stu-id="e7c8f-400">Context</span></span> | <span data-ttu-id="e7c8f-401">Пример максимального значения задержки</span><span class="sxs-lookup"><span data-stu-id="e7c8f-401">Example maximum latency</span></span> | <span data-ttu-id="e7c8f-402">Политика повтора</span><span class="sxs-lookup"><span data-stu-id="e7c8f-402">Retry policy</span></span> | <span data-ttu-id="e7c8f-403">Параметры</span><span class="sxs-lookup"><span data-stu-id="e7c8f-403">Settings</span></span> | <span data-ttu-id="e7c8f-404">Принцип работы</span><span class="sxs-lookup"><span data-stu-id="e7c8f-404">How it works</span></span> |
|---------|---------|---------|---------|---------|
| <span data-ttu-id="e7c8f-405">Интерактивный, пользовательский интерфейс или передний план</span><span class="sxs-lookup"><span data-stu-id="e7c8f-405">Interactive, UI, or foreground</span></span> | <span data-ttu-id="e7c8f-406">2 с\*</span><span class="sxs-lookup"><span data-stu-id="e7c8f-406">2 seconds\*</span></span>  | <span data-ttu-id="e7c8f-407">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="e7c8f-407">Exponential</span></span> | <span data-ttu-id="e7c8f-408">MinimumBackoff = 0</span><span class="sxs-lookup"><span data-stu-id="e7c8f-408">MinimumBackoff = 0</span></span> <br/> <span data-ttu-id="e7c8f-409">MaximumBackoff = 30 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-409">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="e7c8f-410">DeltaBackoff = 300 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-410">DeltaBackoff = 300 msec.</span></span> <br/> <span data-ttu-id="e7c8f-411">TimeBuffer = 300 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-411">TimeBuffer = 300 msec.</span></span> <br/> <span data-ttu-id="e7c8f-412">MaxRetryCount = 2</span><span class="sxs-lookup"><span data-stu-id="e7c8f-412">MaxRetryCount = 2</span></span> | <span data-ttu-id="e7c8f-413">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-413">Attempt 1: Delay 0 sec.</span></span> <br/> <span data-ttu-id="e7c8f-414">Попытка 2 — задержка ~ 300 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-414">Attempt 2: Delay ~300 msec.</span></span> <br/> <span data-ttu-id="e7c8f-415">Попытка 3 — задержка ~ 900 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-415">Attempt 3: Delay ~900 msec.</span></span> |
| <span data-ttu-id="e7c8f-416">Фоновый или пакетный</span><span class="sxs-lookup"><span data-stu-id="e7c8f-416">Background or batch</span></span> | <span data-ttu-id="e7c8f-417">30 секунд</span><span class="sxs-lookup"><span data-stu-id="e7c8f-417">30 seconds</span></span> | <span data-ttu-id="e7c8f-418">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="e7c8f-418">Exponential</span></span> | <span data-ttu-id="e7c8f-419">MinimumBackoff = 1</span><span class="sxs-lookup"><span data-stu-id="e7c8f-419">MinimumBackoff = 1</span></span> <br/> <span data-ttu-id="e7c8f-420">MaximumBackoff = 30 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-420">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="e7c8f-421">DeltaBackoff = 1,75 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-421">DeltaBackoff = 1.75 sec.</span></span> <br/> <span data-ttu-id="e7c8f-422">TimeBuffer = 5 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-422">TimeBuffer = 5 sec.</span></span> <br/> <span data-ttu-id="e7c8f-423">MaxRetryCount = 3</span><span class="sxs-lookup"><span data-stu-id="e7c8f-423">MaxRetryCount = 3</span></span> | <span data-ttu-id="e7c8f-424">Попытка 1 — задержка ~ 1 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-424">Attempt 1: Delay ~1 sec.</span></span> <br/> <span data-ttu-id="e7c8f-425">Попытка 2 — задержка ~ 3 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-425">Attempt 2: Delay ~3 sec.</span></span> <br/> <span data-ttu-id="e7c8f-426">Попытка 3 — задержка ~ 6 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-426">Attempt 3: Delay ~6 msec.</span></span> <br/> <span data-ttu-id="e7c8f-427">Попытка 4 — задержка ~ 13 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-427">Attempt 4: Delay ~13 msec.</span></span> |

<span data-ttu-id="e7c8f-428">\* Без значения дополнительной задержки, которая добавляется при получении ответа "Сервер занят".</span><span class="sxs-lookup"><span data-stu-id="e7c8f-428">\* Not including additional delay that is added if a Server Busy response is received.</span></span>

### <a name="telemetry"></a><span data-ttu-id="e7c8f-429">Телеметрия</span><span class="sxs-lookup"><span data-stu-id="e7c8f-429">Telemetry</span></span>
<span data-ttu-id="e7c8f-430">Служебная шина регистрирует повторные попытки как события ETW с помощью **EventSource**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-430">Service Bus logs retries as ETW events using an **EventSource**.</span></span> <span data-ttu-id="e7c8f-431">Необходимо присоединить службу **EventListener** к источнику события, чтобы зарегистрировать события и просмотреть в средстве просмотра производительности или записать данные о них в журнале с подходящим назначением.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-431">You must attach an **EventListener** to the event source to capture the events and view them in Performance Viewer, or write them to a suitable destination log.</span></span> <span data-ttu-id="e7c8f-432">События повтора операций отображаются в следующей форме:</span><span class="sxs-lookup"><span data-stu-id="e7c8f-432">The retry events are of the following form:</span></span>

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

### <a name="examples"></a><span data-ttu-id="e7c8f-433">Примеры</span><span class="sxs-lookup"><span data-stu-id="e7c8f-433">Examples</span></span>
<span data-ttu-id="e7c8f-434">В следующем примере кода показано, как задать политику повтора для</span><span class="sxs-lookup"><span data-stu-id="e7c8f-434">The following code example shows how to set the retry policy for:</span></span>

* <span data-ttu-id="e7c8f-435">Диспетчера пространства имен.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-435">A namespace manager.</span></span> <span data-ttu-id="e7c8f-436">Политика применяется ко всем операциям диспетчера и не может быть переопределена для отдельных операций.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-436">The policy applies to all operations on that manager, and cannot be overridden for individual operations.</span></span>
* <span data-ttu-id="e7c8f-437">Фабрики сообщений.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-437">A messaging factory.</span></span> <span data-ttu-id="e7c8f-438">Политика применяется ко всем клиентам, созданным из этой фабрики, и не может быть переопределена при создании отдельных клиентов.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-438">The policy applies to all clients created from that factory, and cannot be overridden when creating individual clients.</span></span>
* <span data-ttu-id="e7c8f-439">Отдельных клиентов обмена сообщениями.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-439">An individual messaging client.</span></span> <span data-ttu-id="e7c8f-440">После создания клиента можно задать политику повтора для этого клиента.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-440">After a client has been created, you can set the retry policy for that client.</span></span> <span data-ttu-id="e7c8f-441">Политика применяется ко всем операциям на этом клиенте.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-441">The policy applies to all operations on that client.</span></span>

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

### <a name="more-information"></a><span data-ttu-id="e7c8f-442">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="e7c8f-442">More information</span></span>
* [<span data-ttu-id="e7c8f-443">Шаблоны асинхронного обмена сообщениями и высокий уровень доступности</span><span class="sxs-lookup"><span data-stu-id="e7c8f-443">Asynchronous Messaging Patterns and High Availability</span></span>](http://msdn.microsoft.com/library/azure/dn292562.aspx)

## <a name="service-fabric"></a><span data-ttu-id="e7c8f-444">Service Fabric</span><span class="sxs-lookup"><span data-stu-id="e7c8f-444">Service Fabric</span></span>

<span data-ttu-id="e7c8f-445">Распространение надежных служб в кластере Service Fabric защищает от большинства потенциальных временных сбоев, которые описаны в этой статье.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-445">Distributing reliable services in a Service Fabric cluster guards against most of the potential transient faults discussed in this article.</span></span> <span data-ttu-id="e7c8f-446">Но некоторые временные сбои все равно могут произойти.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-446">Some transient faults are still possible, however.</span></span> <span data-ttu-id="e7c8f-447">Например, если служба именования в момент получения запроса выполняет изменение маршрутизации, она создаст исключение.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-447">For example, the naming service might be in the middle of a routing change when it gets a request, causing it to throw an exception.</span></span> <span data-ttu-id="e7c8f-448">Если повторить этот же запрос через 100 миллисекунд, он, скорее всего, завершится успешно.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-448">If the same request comes 100 milliseconds later, it will probably succeed.</span></span>

<span data-ttu-id="e7c8f-449">Service Fabric обрабатывает такие временные сбои на внутреннем уровне.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-449">Internally, Service Fabric manages this kind of transient fault.</span></span> <span data-ttu-id="e7c8f-450">Некоторые параметры можно настроить с помощью класса `OperationRetrySettings` при настройке служб.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-450">You can configure some settings by using the `OperationRetrySettings` class while setting up your services.</span></span>  <span data-ttu-id="e7c8f-451">Пример такой настройки приведен в следующем коде.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-451">The following code shows an example.</span></span> <span data-ttu-id="e7c8f-452">В большинстве случаев это не требуется, и можно использовать параметры по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-452">In most cases, this should not be necessary, and the default settings will be fine.</span></span>

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

### <a name="more-information"></a><span data-ttu-id="e7c8f-453">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="e7c8f-453">More information</span></span>

* [<span data-ttu-id="e7c8f-454">Удаленная обработка исключений</span><span class="sxs-lookup"><span data-stu-id="e7c8f-454">Remote Exception Handling</span></span>](https://github.com/Microsoft/azure-docs/blob/master/articles/service-fabric/service-fabric-reliable-services-communication-remoting.md#remoting-exception-handling)

## <a name="sql-database-using-adonet"></a><span data-ttu-id="e7c8f-455">Использование Базы данных SQL с ADO.NET</span><span class="sxs-lookup"><span data-stu-id="e7c8f-455">SQL Database using ADO.NET</span></span>
<span data-ttu-id="e7c8f-456">База данных SQL является размещенной базой данных SQL различных размеров и является как стандартной (общей), так и премиальной (частной) службой.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-456">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="e7c8f-457">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="e7c8f-457">Retry mechanism</span></span>
<span data-ttu-id="e7c8f-458">База данных SQL не имеет встроенной поддержки выполнения повторных попыток, если доступ осуществляется с помощью ADO.NET.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-458">SQL Database has no built-in support for retries when accessed using ADO.NET.</span></span> <span data-ttu-id="e7c8f-459">Однако коды возврата от запросов могут быть использованы для определения причины сбоя запроса.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-459">However, the return codes from requests can be used to determine why a request failed.</span></span> <span data-ttu-id="e7c8f-460">Дополнительные сведения о регулировании базы данных SQL см. в статье [Ограничения ресурсов базы данных SQL Azure](/azure/sql-database/sql-database-resource-limits).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-460">For more information about SQL Database throttling, see [Azure SQL Database resource limits](/azure/sql-database/sql-database-resource-limits).</span></span> <span data-ttu-id="e7c8f-461">Список кодов ошибок см. в описании [кодов ошибок SQL для клиентских приложений Базы данных SQL](/azure/sql-database/sql-database-develop-error-messages).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-461">For a list of relevant error codes, see [SQL error codes for SQL Database client applications](/azure/sql-database/sql-database-develop-error-messages).</span></span>

<span data-ttu-id="e7c8f-462">Для реализации повторных попыток при обращении к Базе данных SQL вы можете использовать библиотеку Polly.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-462">You can use the Polly library to implement retries for SQL Database.</span></span> <span data-ttu-id="e7c8f-463">См. руководство по [обработке временного сбоя с помощью Polly](#transient-fault-handling-with-polly).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-463">See [Transient fault handling with Polly](#transient-fault-handling-with-polly).</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="e7c8f-464">Руководство по использованию механизма повторов</span><span class="sxs-lookup"><span data-stu-id="e7c8f-464">Retry usage guidance</span></span>
<span data-ttu-id="e7c8f-465">При доступе к базе данных SQL с помощью ADO.NET придерживайтесь следующих рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-465">Consider the following guidelines when accessing SQL Database using ADO.NET:</span></span>

* <span data-ttu-id="e7c8f-466">Выберите соответствующую службу (общую или премиальную).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-466">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="e7c8f-467">Общий экземпляр может испытывать большие задержки подключения и регулирования чем обычно из-за использования другими клиентами общего сервера.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-467">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="e7c8f-468">Если требуется более прогнозируемая производительность и выполнение надежных операций с низкой задержкой, попробуйте выбрать премиальный вариант.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-468">If more predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="e7c8f-469">Убедитесь, что выполнение повторов происходит на соответствующем уровне или области во избежание операций, не являющихся идемпотентными, что может вызвать несоответствие в данных.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-469">Ensure that you perform retries at the appropriate level or scope to avoid non-idempotent operations causing inconsistency in the data.</span></span> <span data-ttu-id="e7c8f-470">В идеальном случае все операции должны быть идемпотентными таким образом, чтобы их можно было повторить без возникновения несоответствия.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-470">Ideally, all operations should be idempotent so that they can be repeated without causing inconsistency.</span></span> <span data-ttu-id="e7c8f-471">Если это не так, повтор следует производить на уровне или в области, которая позволяет отменить все связанные изменения, если одна операция завершится ошибкой. Например, в области транзакций.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-471">Where this is not the case, the retry should be performed at a level or scope that allows all related changes to be undone if one operation fails; for example, from within a transactional scope.</span></span> <span data-ttu-id="e7c8f-472">Для получения дополнительных сведений обратитесь к разделу [Основы облачной службы. Уровень доступа к данным: обработка временных ошибок](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-472">For more information, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span></span>
* <span data-ttu-id="e7c8f-473">Не рекомендуется использование стратегии с фиксированным интервалом при работе с базой данных SQL Azure, кроме интерактивных сценариев при наличии нескольких попыток повтора с очень короткими интервалами.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-473">A fixed interval strategy is not recommended for use with Azure SQL Database except for interactive scenarios where there are only a few retries at very short intervals.</span></span> <span data-ttu-id="e7c8f-474">Вместо этого рекомендуется использовать стратегию экспоненциальной отсрочки для большинства сценариев.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-474">Instead, consider using an exponential back-off strategy for the majority of scenarios.</span></span>
* <span data-ttu-id="e7c8f-475">Выберите подходящее значение для времени ожидания подключения и команды при определении подключения.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-475">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="e7c8f-476">Слишком короткое время ожидания может привести к преждевременным сбоям подключений, когда база данных занята.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-476">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="e7c8f-477">Слишком длинное время ожидания может помешать правильной работе логики повторов из-за слишком долгого ожидания до обнаружения ошибки соединения.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-477">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="e7c8f-478">Значение времени ожидания — это компонент сквозной задержки; фактически это значение добавляется к величине задержки повтора, указанной в политике повтора для каждой попытки.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-478">The value of the timeout is a component of the end-to-end latency; it is effectively added to the retry delay specified in the retry policy for every retry attempt.</span></span>
* <span data-ttu-id="e7c8f-479">Закройте соединение после определенного числа повторных попыток, даже если используется экспоненциальная логика повторных попыток, и повторите операцию с новым подключением.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-479">Close the connection after a certain number of retries, even when using an exponential back off retry logic, and retry the operation on a new connection.</span></span> <span data-ttu-id="e7c8f-480">Повторение одной и той же операции несколько раз для одного соединения может стать фактором возникновения проблем с подключением.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-480">Retrying the same operation multiple times on the same connection can be a factor that contributes to connection problems.</span></span> <span data-ttu-id="e7c8f-481">Пример использования этой технологии см. в статье [Основы облачной службы. Уровень доступа к данным: обработка временных ошибок](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-481">For an example of this technique, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span></span>
* <span data-ttu-id="e7c8f-482">Когда используется пул подключений (по умолчанию), существует вероятность того, что будет выбрано то же подключение из пула, даже после закрытия и повторного открытия соединения.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-482">When connection pooling is in use (the default) there is a chance that the same connection will be chosen from the pool, even after closing and reopening a connection.</span></span> <span data-ttu-id="e7c8f-483">Если это так, способом устранения подобной ситуации является вызов метода **ClearPool** класса **SqlConnection**, чтобы отметить подключение как не используемое повторно.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-483">If this is the case, a technique to resolve it is to call the **ClearPool** method of the **SqlConnection** class to mark the connection as not reusable.</span></span> <span data-ttu-id="e7c8f-484">Тем не менее это следует делать только после сбоя нескольких попыток соединения и только при обнаружении определенного класса временных ошибок, таких как время ожидания SQL (код ошибки -2), связанных со сбоями в подключениях.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-484">However, you should do this only after several connection attempts have failed, and only when encountering the specific class of transient failures such as SQL timeouts (error code -2) related to faulty connections.</span></span>
* <span data-ttu-id="e7c8f-485">Если код доступа к данным использует транзакции, инициируемые как экземпляры **TransactionScope** , то логика повторных попыток должна включать повторное открытие подключения и инициацию новой области транзакции.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-485">If the data access code uses transactions initiated as **TransactionScope** instances, the retry logic should reopen the connection and initiate a new transaction scope.</span></span> <span data-ttu-id="e7c8f-486">По этой причине блок кода повторной попытки должен охватывать всю область транзакции.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-486">For this reason, the retryable code block should encompass the entire scope of the transaction.</span></span>

<span data-ttu-id="e7c8f-487">Начните с использования следующих параметров для операции повтора.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-487">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="e7c8f-488">Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-488">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="e7c8f-489">**Контекст**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-489">**Context**</span></span> | <span data-ttu-id="e7c8f-490">**Максимальная задержка<br />примера целевого E2E**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-490">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="e7c8f-491">**Стратегия повторов**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-491">**Retry strategy**</span></span> | <span data-ttu-id="e7c8f-492">**Параметры**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-492">**Settings**</span></span> | <span data-ttu-id="e7c8f-493">**Значения**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-493">**Values**</span></span> | <span data-ttu-id="e7c8f-494">**Принцип работы**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-494">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="e7c8f-495">Интерактивный, пользовательский интерфейс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-495">Interactive, UI,</span></span><br /><span data-ttu-id="e7c8f-496">или передний план</span><span class="sxs-lookup"><span data-stu-id="e7c8f-496">or foreground</span></span> |<span data-ttu-id="e7c8f-497">2 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-497">2 sec</span></span> |<span data-ttu-id="e7c8f-498">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="e7c8f-498">FixedInterval</span></span> |<span data-ttu-id="e7c8f-499">Число повторных попыток</span><span class="sxs-lookup"><span data-stu-id="e7c8f-499">Retry count</span></span><br /><span data-ttu-id="e7c8f-500">Интервал попытки</span><span class="sxs-lookup"><span data-stu-id="e7c8f-500">Retry interval</span></span><br /><span data-ttu-id="e7c8f-501">Первый быстрый повтор</span><span class="sxs-lookup"><span data-stu-id="e7c8f-501">First fast retry</span></span> |<span data-ttu-id="e7c8f-502">3</span><span class="sxs-lookup"><span data-stu-id="e7c8f-502">3</span></span><br /><span data-ttu-id="e7c8f-503">500 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-503">500 ms</span></span><br /><span data-ttu-id="e7c8f-504">true</span><span class="sxs-lookup"><span data-stu-id="e7c8f-504">true</span></span> |<span data-ttu-id="e7c8f-505">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-505">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="e7c8f-506">Попытка 2 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-506">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="e7c8f-507">попытка 3 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-507">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="e7c8f-508">Фоновый</span><span class="sxs-lookup"><span data-stu-id="e7c8f-508">Background</span></span><br /><span data-ttu-id="e7c8f-509">или пакетный</span><span class="sxs-lookup"><span data-stu-id="e7c8f-509">or batch</span></span> |<span data-ttu-id="e7c8f-510">30 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-510">30 sec</span></span> |<span data-ttu-id="e7c8f-511">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="e7c8f-511">ExponentialBackoff</span></span> |<span data-ttu-id="e7c8f-512">Число повторных попыток</span><span class="sxs-lookup"><span data-stu-id="e7c8f-512">Retry count</span></span><br /><span data-ttu-id="e7c8f-513">Минимальная задержка</span><span class="sxs-lookup"><span data-stu-id="e7c8f-513">Min back-off</span></span><br /><span data-ttu-id="e7c8f-514">Максимальная задержка</span><span class="sxs-lookup"><span data-stu-id="e7c8f-514">Max back-off</span></span><br /><span data-ttu-id="e7c8f-515">Разностная задержка</span><span class="sxs-lookup"><span data-stu-id="e7c8f-515">Delta back-off</span></span><br /><span data-ttu-id="e7c8f-516">Первый быстрый повтор</span><span class="sxs-lookup"><span data-stu-id="e7c8f-516">First fast retry</span></span> |<span data-ttu-id="e7c8f-517">5</span><span class="sxs-lookup"><span data-stu-id="e7c8f-517">5</span></span><br /><span data-ttu-id="e7c8f-518">0 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-518">0 sec</span></span><br /><span data-ttu-id="e7c8f-519">60 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-519">60 sec</span></span><br /><span data-ttu-id="e7c8f-520">2 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-520">2 sec</span></span><br /><span data-ttu-id="e7c8f-521">false</span><span class="sxs-lookup"><span data-stu-id="e7c8f-521">false</span></span> |<span data-ttu-id="e7c8f-522">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-522">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="e7c8f-523">Попытка 2 — задержка 2 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-523">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="e7c8f-524">Попытка 3 — задержка 6 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-524">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="e7c8f-525">Попытка 4 — задержка 14 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-525">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="e7c8f-526">Попытка 5 — задержка 30 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-526">Attempt 5 - delay ~30 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="e7c8f-527">Целевые значения сквозной задержки подразумевают время ожидания по умолчанию для подключения к службе.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-527">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="e7c8f-528">При указании более длинного времени ожидания подключения величина сквозной задержки будет увеличена путем добавления этого дополнительного времени для каждой попытки повтора.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-528">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="e7c8f-529">Примеры</span><span class="sxs-lookup"><span data-stu-id="e7c8f-529">Examples</span></span>
<span data-ttu-id="e7c8f-530">В этом разделе показано, как с помощью Polly обращаться к Базе данных SQL Azure с использованием набора политик повторных попыток, настроенных в классе `Policy`.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-530">This section shows how you can use Polly to access Azure SQL Database using a set of retry policies configured in the `Policy` class.</span></span>

<span data-ttu-id="e7c8f-531">Следующий код демонстрирует метод расширения для класса `SqlCommand`, который вызывает `ExecuteAsync` с экспоненциальным увеличением задержки.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-531">The following code shows an extension method on the `SqlCommand` class that calls `ExecuteAsync` with exponential backoff.</span></span>

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

<span data-ttu-id="e7c8f-532">Этот асинхронный метод расширения можно использовать следующим образом.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-532">This asynchronous extension method can be used as follows.</span></span>

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a><span data-ttu-id="e7c8f-533">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="e7c8f-533">More information</span></span>
* [<span data-ttu-id="e7c8f-534">Основы облачной службы. Уровень доступа к данным: обработка временных ошибок</span><span class="sxs-lookup"><span data-stu-id="e7c8f-534">Cloud Service Fundamentals Data Access Layer – Transient Fault Handling</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

<span data-ttu-id="e7c8f-535">Общие рекомендации по повышению эффективности Базы данных SQL см. в статье [Руководство по эластичности и производительности базы данных SQL Azure](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)</span><span class="sxs-lookup"><span data-stu-id="e7c8f-535">For general guidance on getting the most from SQL Database, see [Azure SQL Database Performance and Elasticity Guide](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span></span>

## <a name="sql-database-using-entity-framework-6"></a><span data-ttu-id="e7c8f-536">Использование Базы данных SQL с Entity Framework 6</span><span class="sxs-lookup"><span data-stu-id="e7c8f-536">SQL Database using Entity Framework 6</span></span>
<span data-ttu-id="e7c8f-537">База данных SQL является размещенной базой данных SQL различных размеров и является как стандартной (общей), так и премиальной (частной) службой.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-537">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span> <span data-ttu-id="e7c8f-538">Механизм Entity Framework является модулем объектно-реляционного сопоставления, который позволяет разработчикам .NET работать с реляционными данными с помощью специфических для домена объектов.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-538">Entity Framework is an object-relational mapper that enables .NET developers to work with relational data using domain-specific objects.</span></span> <span data-ttu-id="e7c8f-539">Это исключает необходимость в большинстве кодов доступа к данным, которые обычно требуется писать разработчикам.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-539">It eliminates the need for most of the data-access code that developers usually need to write.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="e7c8f-540">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="e7c8f-540">Retry mechanism</span></span>
<span data-ttu-id="e7c8f-541">Поддержка повторных попыток при доступе к базе данных SQL с помощью Entity Framework 6.0 и выше предоставляется через механизм, называемый [Устойчивость соединения и логика повтора](http://msdn.microsoft.com/data/dn456835.aspx).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-541">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher through a mechanism called [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span> <span data-ttu-id="e7c8f-542">Ниже перечислены основные возможности механизма повтора.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-542">The main features of the retry mechanism are:</span></span>

* <span data-ttu-id="e7c8f-543">Интерфейс **IDbExecutionStrategy** является основной абстракцией.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-543">The primary abstraction is the **IDbExecutionStrategy** interface.</span></span> <span data-ttu-id="e7c8f-544">Этот интерфейс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-544">This interface:</span></span>
  * <span data-ttu-id="e7c8f-545">Определяет синхронные и асинхронные методы **Execute** \*.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-545">Defines synchronous and asynchronous **Execute**\* methods.</span></span>
  * <span data-ttu-id="e7c8f-546">Определяет классы для непосредственного использования, или которые могут быть настроены на контекст базы данных как стратегию по умолчанию, с сопоставленным именем поставщика или сопоставленным именем поставщика и именем сервера.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-546">Defines classes that can be used directly or can be configured on a database context as a default strategy, mapped to provider name, or mapped to a provider name and server name.</span></span> <span data-ttu-id="e7c8f-547">При настройке на контекст повторы происходят на уровне операций с отдельными базами данных, из которых несколько может соответствовать заданному контексту операции.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-547">When configured on a context, retries occur at the level of individual database operations, of which there might be several for a given context operation.</span></span>
  * <span data-ttu-id="e7c8f-548">Определяет, когда и каким образом следует выполнить повтор при ошибке соединения.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-548">Defines when to retry a failed connection, and how.</span></span>
* <span data-ttu-id="e7c8f-549">Включает несколько встроенных реализаций интерфейса **IDbExecutionStrategy** .</span><span class="sxs-lookup"><span data-stu-id="e7c8f-549">It includes several built-in implementations of the **IDbExecutionStrategy** interface:</span></span>
  * <span data-ttu-id="e7c8f-550">Значение по умолчанию — не повторять.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-550">Default - no retrying.</span></span>
  * <span data-ttu-id="e7c8f-551">По умолчанию для базы данных SQL (автоматический режим) — не повторять, но проверять исключения и завершать их с предложением использовать стратегию базы данных SQL.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-551">Default for SQL Database (automatic) - no retrying, but inspects exceptions and wraps them with suggestion to use the SQL Database strategy.</span></span>
  * <span data-ttu-id="e7c8f-552">По умолчанию для базы данных SQL — экспоненциальный повтор (наследуется от базового класса) плюс логика обнаружения базы данных SQL.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-552">Default for SQL Database - exponential (inherited from base class) plus SQL Database detection logic.</span></span>
* <span data-ttu-id="e7c8f-553">Это реализует стратегию экспоненциальной отсрочки, которая также включает случайный выбор.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-553">It implements an exponential back-off strategy that includes randomization.</span></span>
* <span data-ttu-id="e7c8f-554">Встроенные классы повтора включают отслеживание состояния и не являются потокобезопасными.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-554">The built-in retry classes are stateful and are not thread safe.</span></span> <span data-ttu-id="e7c8f-555">Однако они могут использоваться повторно после завершения текущей операции.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-555">However, they can be reused after the current operation is completed.</span></span>
* <span data-ttu-id="e7c8f-556">В случае превышения заданного счетчика повторов результаты помещаются в новое исключение.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-556">If the specified retry count is exceeded, the results are wrapped in a new exception.</span></span> <span data-ttu-id="e7c8f-557">Это не вызывает перенаправления текущего исключения.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-557">It does not bubble up the current exception.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="e7c8f-558">Настройки политики</span><span class="sxs-lookup"><span data-stu-id="e7c8f-558">Policy configuration</span></span>
<span data-ttu-id="e7c8f-559">Поддержка повторных попыток предоставляется при доступе к базе данных SQL с помощью Entity Framework 6.0 и более поздних версий.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-559">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher.</span></span> <span data-ttu-id="e7c8f-560">Политики повтора настраиваются программным способом.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-560">Retry policies are configured programmatically.</span></span> <span data-ttu-id="e7c8f-561">Нельзя изменить конфигурацию для каждой операции.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-561">The configuration cannot be changed on a per-operation basis.</span></span>

<span data-ttu-id="e7c8f-562">При настройке стратегии в контексте по умолчанию, необходимо указать функцию, которая создает новые стратегии по требованию.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-562">When configuring a strategy on the context as the default, you specify a function that creates a new strategy on demand.</span></span> <span data-ttu-id="e7c8f-563">В следующем коде показано, как можно создать класс конфигурации повтора, который расширяет базовый класс **DbConfiguration** .</span><span class="sxs-lookup"><span data-stu-id="e7c8f-563">The following code shows how you can create a retry configuration class that extends the **DbConfiguration** base class.</span></span>

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

<span data-ttu-id="e7c8f-564">Затем можно указать это в качестве стратегии повторных попыток по умолчанию для всех операций с помощью метода **SetConfiguration** экземпляра **DbConfiguration** при запуске приложения.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-564">You can then specify this as the default retry strategy for all operations using the **SetConfiguration** method of the **DbConfiguration** instance when the application starts.</span></span> <span data-ttu-id="e7c8f-565">По умолчанию EF будет автоматически обнаруживать и использовать класс конфигурации.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-565">By default, EF will automatically discover and use the configuration class.</span></span>

```csharp
DbConfiguration.SetConfiguration(new BloggingContextConfiguration());
```

<span data-ttu-id="e7c8f-566">Класс конфигурации повторных попыток для контекста задается дополнением контекста класса атрибутом **DbConfigurationType** .</span><span class="sxs-lookup"><span data-stu-id="e7c8f-566">You can specify the retry configuration class for a context by annotating the context class with a **DbConfigurationType** attribute.</span></span> <span data-ttu-id="e7c8f-567">Однако при наличии только одного класса конфигурации EF будет использовать его без необходимости дополнения контекста.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-567">However, if you have only one configuration class, EF will use it without the need to annotate the context.</span></span>

```csharp
[DbConfigurationType(typeof(BloggingContextConfiguration))]
public class BloggingContext : DbContext
```

<span data-ttu-id="e7c8f-568">Если необходимо использовать различные стратегии для конкретных операций или отключить повторы для определенных операций, то можно создать класс конфигурации, который позволяет приостановить или заменить стратегии, установив флаг **CallContext**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-568">If you need to use different retry strategies for specific operations, or disable retries for specific operations, you can create a configuration class that allows you to suspend or swap strategies by setting a flag in the **CallContext**.</span></span> <span data-ttu-id="e7c8f-569">Класс конфигурации может использовать этот флаг для переключения стратегии или отключения стратегии, предоставленной вами, с последующим использованием стратегии по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-569">The configuration class can use this flag to switch strategies, or disable the strategy you provide and use a default strategy.</span></span> <span data-ttu-id="e7c8f-570">Дополнительные сведения см. в разделе [Приостановка выполнения стратегии](http://msdn.microsoft.com/dn307226#transactions_workarounds) на странице "Ограничения стратегий выполнения повторов (для EF6 и выше)".</span><span class="sxs-lookup"><span data-stu-id="e7c8f-570">For more information, see [Suspend Execution Strategy](http://msdn.microsoft.com/dn307226#transactions_workarounds) in the page Limitations with Retrying Execution Strategies (EF6 onwards).</span></span>

<span data-ttu-id="e7c8f-571">Иным способом использования стратегий повторов, определенных для отдельных операций, является создание экземпляра класса требуемой стратегии и передача ему нужных параметров.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-571">Another technique for using specific retry strategies for individual operations is to create an instance of the required strategy class and supply the desired settings through parameters.</span></span> <span data-ttu-id="e7c8f-572">Затем можно вызвать его метод **ExecuteAsync** .</span><span class="sxs-lookup"><span data-stu-id="e7c8f-572">You then invoke its **ExecuteAsync** method.</span></span>

```csharp
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
```

<span data-ttu-id="e7c8f-573">Самый простой способ использовать класс **DbConfiguration** — это обнаружить его в той же сборке, что и класс **DbContext**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-573">The simplest way to use a **DbConfiguration** class is to locate it in the same assembly as the **DbContext** class.</span></span> <span data-ttu-id="e7c8f-574">Однако этот способ не подходит в случае, когда тот же контекст требуется в различных сценариях, таких как различные интерактивные и фоновые стратегии повторов.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-574">However, this is not appropriate when the same context is required in different scenarios, such as different interactive and background retry strategies.</span></span> <span data-ttu-id="e7c8f-575">Если разные контексты выполняются в отдельных доменах приложений, можно использовать встроенную поддержку для указания классов конфигурации в файле конфигурации или задать ее явным образом с помощью кода.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-575">If the different contexts execute in separate AppDomains, you can use the built-in support for specifying configuration classes in the configuration file or set it explicitly using code.</span></span> <span data-ttu-id="e7c8f-576">Если различные контексты должны выполняться в том же домене приложения, потребуется создание пользовательского решения.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-576">If the different contexts must execute in the same AppDomain, a custom solution will be required.</span></span>

<span data-ttu-id="e7c8f-577">Дополнительные сведения см. в статье [Конфигурация на основе кода (для EF6 и выше)](http://msdn.microsoft.com/data/jj680699.aspx).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-577">For more information, see [Code-Based Configuration (EF6 onwards)](http://msdn.microsoft.com/data/jj680699.aspx).</span></span>

<span data-ttu-id="e7c8f-578">Следующая таблица показывает значения по умолчанию для встроенной политики повторов при использовании EF6.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-578">The following table shows the default settings for the built-in retry policy when using EF6.</span></span>

| <span data-ttu-id="e7c8f-579">Параметр</span><span class="sxs-lookup"><span data-stu-id="e7c8f-579">Setting</span></span> | <span data-ttu-id="e7c8f-580">Значение по умолчанию</span><span class="sxs-lookup"><span data-stu-id="e7c8f-580">Default value</span></span> | <span data-ttu-id="e7c8f-581">Значение</span><span class="sxs-lookup"><span data-stu-id="e7c8f-581">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="e7c8f-582">Политика</span><span class="sxs-lookup"><span data-stu-id="e7c8f-582">Policy</span></span> | <span data-ttu-id="e7c8f-583">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="e7c8f-583">Exponential</span></span> | <span data-ttu-id="e7c8f-584">Экспоненциальная задержка.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-584">Exponential back-off.</span></span> |
| <span data-ttu-id="e7c8f-585">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="e7c8f-585">MaxRetryCount</span></span> | <span data-ttu-id="e7c8f-586">5</span><span class="sxs-lookup"><span data-stu-id="e7c8f-586">5</span></span> | <span data-ttu-id="e7c8f-587">Максимальное число попыток.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-587">The maximum number of retries.</span></span> |
| <span data-ttu-id="e7c8f-588">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="e7c8f-588">MaxDelay</span></span> | <span data-ttu-id="e7c8f-589">30 секунд</span><span class="sxs-lookup"><span data-stu-id="e7c8f-589">30 seconds</span></span> | <span data-ttu-id="e7c8f-590">Максимальная задержка между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-590">The maximum delay between retries.</span></span> <span data-ttu-id="e7c8f-591">Это значение не влияет на то, как вычисляется значение серии задержек.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-591">This value does not affect how the series of delays are computed.</span></span> <span data-ttu-id="e7c8f-592">Оно определяет только верхнюю границу.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-592">It only defines an upper bound.</span></span> |
| <span data-ttu-id="e7c8f-593">DefaultCoefficient</span><span class="sxs-lookup"><span data-stu-id="e7c8f-593">DefaultCoefficient</span></span> | <span data-ttu-id="e7c8f-594">1 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-594">1 second</span></span> | <span data-ttu-id="e7c8f-595">Коэффициент для вычисления значения экспоненциальной задержки.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-595">The coefficient for the exponential back-off computation.</span></span> <span data-ttu-id="e7c8f-596">Это значение не может быть изменено.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-596">This value cannot be changed.</span></span> |
| <span data-ttu-id="e7c8f-597">DefaultRandomFactor</span><span class="sxs-lookup"><span data-stu-id="e7c8f-597">DefaultRandomFactor</span></span> | <span data-ttu-id="e7c8f-598">1,1</span><span class="sxs-lookup"><span data-stu-id="e7c8f-598">1.1</span></span> | <span data-ttu-id="e7c8f-599">Множитель, который используется для добавления значения случайной задержки для каждой записи.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-599">The multiplier used to add a random delay for each entry.</span></span> <span data-ttu-id="e7c8f-600">Это значение не может быть изменено.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-600">This value cannot be changed.</span></span> |
| <span data-ttu-id="e7c8f-601">DefaultExponentialBase</span><span class="sxs-lookup"><span data-stu-id="e7c8f-601">DefaultExponentialBase</span></span> | <span data-ttu-id="e7c8f-602">2</span><span class="sxs-lookup"><span data-stu-id="e7c8f-602">2</span></span> | <span data-ttu-id="e7c8f-603">Множитель, который используется для вычисления значения следующей задержки.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-603">The multiplier used to calculate the next delay.</span></span> <span data-ttu-id="e7c8f-604">Это значение не может быть изменено.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-604">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="e7c8f-605">Руководство по использованию механизма повторов</span><span class="sxs-lookup"><span data-stu-id="e7c8f-605">Retry usage guidance</span></span>
<span data-ttu-id="e7c8f-606">При доступе к базе данных SQL с помощью EF6, придерживайтесь следующих рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-606">Consider the following guidelines when accessing SQL Database using EF6:</span></span>

* <span data-ttu-id="e7c8f-607">Выберите соответствующую службу (общую или премиальную).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-607">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="e7c8f-608">Общий экземпляр может испытывать большие задержки подключения и регулирования чем обычно из-за использования другими клиентами общего сервера.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-608">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="e7c8f-609">Если требуется выполнение надежных операций с низкой задержкой и прогнозируемая производительность, попробуйте выбрать премиальный вариант.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-609">If predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
* <span data-ttu-id="e7c8f-610">Стратегия фиксированного интервала не рекомендуется для использования с базой данных SQL Azure.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-610">A fixed interval strategy is not recommended for use with Azure SQL Database.</span></span> <span data-ttu-id="e7c8f-611">Вместо этого используйте стратегию экспоненциальной отсрочки, так как служба может быть перегружена и большее время задержки обеспечит большее время для ее восстановления.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-611">Instead, use an exponential back-off strategy because the service may be overloaded, and longer delays allow more time for it to recover.</span></span>
* <span data-ttu-id="e7c8f-612">Выберите подходящее значение для времени ожидания подключения и команды при определении подключения.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-612">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="e7c8f-613">Выберите значение времени ожидания на основе логики вашей работы и на основе испытаний.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-613">Base the timeout on both your business logic design and through testing.</span></span> <span data-ttu-id="e7c8f-614">Может потребоваться изменить это значение по мере изменения объемов данных или бизнес-процессов.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-614">You may need to modify this value over time as the volumes of data or the business processes change.</span></span> <span data-ttu-id="e7c8f-615">Слишком короткое время ожидания может привести к преждевременным сбоям подключений, когда база данных занята.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-615">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="e7c8f-616">Слишком длинное время ожидания может помешать правильной работе логики повторов из-за слишком долгого ожидания до обнаружения ошибки соединения.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-616">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="e7c8f-617">Значение времени ожидания — это компонент сквозной задержки, несмотря на это вы не можете легко определить, сколько команд будет выполнено при сохранении контекста.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-617">The value of the timeout is a component of the end-to-end latency, although you cannot easily determine how many commands will execute when saving the context.</span></span> <span data-ttu-id="e7c8f-618">Время ожидания по умолчанию можно изменить, задав свойство **CommandTimeout** экземпляра **DbContext**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-618">You can change the default timeout by setting the **CommandTimeout** property of the **DbContext** instance.</span></span>
* <span data-ttu-id="e7c8f-619">Платформа Entity Framework поддерживает конфигурации повторов, определенные в файлах конфигурации.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-619">Entity Framework supports retry configurations defined in configuration files.</span></span> <span data-ttu-id="e7c8f-620">Однако для максимальной гибкости в Azure необходимо создать конфигурацию программным способом в приложении.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-620">However, for maximum flexibility on Azure you should consider creating the configuration programmatically within the application.</span></span> <span data-ttu-id="e7c8f-621">Определенные параметры политик повторов, например количество повторных попыток и интервалы повтора, можно сохранить в файле конфигурации службы и использовать во время выполнения для создания соответствующих политик.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-621">The specific parameters for the retry policies, such as the number of retries and the retry intervals, can be stored in the service configuration file and used at runtime to create the appropriate policies.</span></span> <span data-ttu-id="e7c8f-622">Это позволяет изменять параметры без перезапуска приложения.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-622">This allows the settings to be changed without requiring the application to be restarted.</span></span>

<span data-ttu-id="e7c8f-623">Для начала установите следующие параметры для операций повтора.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-623">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="e7c8f-624">Нельзя указать задержку между попытками повтора (она является фиксированной и генерируется в виде экспоненциальной последовательности).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-624">You cannot specify the delay between retry attempts (it is fixed and generated as an exponential sequence).</span></span> <span data-ttu-id="e7c8f-625">Максимальные значения можно указать, как показано ниже, если только вы не создаете пользовательскую стратегию.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-625">You can specify only the maximum values, as shown here; unless you create a custom retry strategy.</span></span> <span data-ttu-id="e7c8f-626">Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-626">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="e7c8f-627">**Контекст**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-627">**Context**</span></span> | <span data-ttu-id="e7c8f-628">**Максимальная задержка<br />примера целевого E2E**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-628">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="e7c8f-629">**Политика повтора**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-629">**Retry policy**</span></span> | <span data-ttu-id="e7c8f-630">**Параметры**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-630">**Settings**</span></span> | <span data-ttu-id="e7c8f-631">**Значения**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-631">**Values**</span></span> | <span data-ttu-id="e7c8f-632">**Принцип работы**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-632">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="e7c8f-633">Интерактивный, пользовательский интерфейс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-633">Interactive, UI,</span></span><br /><span data-ttu-id="e7c8f-634">или передний план</span><span class="sxs-lookup"><span data-stu-id="e7c8f-634">or foreground</span></span> |<span data-ttu-id="e7c8f-635">2 секунды</span><span class="sxs-lookup"><span data-stu-id="e7c8f-635">2 seconds</span></span> |<span data-ttu-id="e7c8f-636">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="e7c8f-636">Exponential</span></span> |<span data-ttu-id="e7c8f-637">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="e7c8f-637">MaxRetryCount</span></span><br /><span data-ttu-id="e7c8f-638">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="e7c8f-638">MaxDelay</span></span> |<span data-ttu-id="e7c8f-639">3</span><span class="sxs-lookup"><span data-stu-id="e7c8f-639">3</span></span><br /><span data-ttu-id="e7c8f-640">750 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-640">750 ms</span></span> |<span data-ttu-id="e7c8f-641">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-641">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="e7c8f-642">Попытка 2 — задержка 750 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-642">Attempt 2 - delay 750 ms</span></span><br /><span data-ttu-id="e7c8f-643">Попытка 3 – задержка 750 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-643">Attempt 3 – delay 750 ms</span></span> |
| <span data-ttu-id="e7c8f-644">Фоновый</span><span class="sxs-lookup"><span data-stu-id="e7c8f-644">Background</span></span><br /> <span data-ttu-id="e7c8f-645">или пакетный</span><span class="sxs-lookup"><span data-stu-id="e7c8f-645">or batch</span></span> |<span data-ttu-id="e7c8f-646">30 секунд</span><span class="sxs-lookup"><span data-stu-id="e7c8f-646">30 seconds</span></span> |<span data-ttu-id="e7c8f-647">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="e7c8f-647">Exponential</span></span> |<span data-ttu-id="e7c8f-648">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="e7c8f-648">MaxRetryCount</span></span><br /><span data-ttu-id="e7c8f-649">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="e7c8f-649">MaxDelay</span></span> |<span data-ttu-id="e7c8f-650">5</span><span class="sxs-lookup"><span data-stu-id="e7c8f-650">5</span></span><br /><span data-ttu-id="e7c8f-651">12 секунд</span><span class="sxs-lookup"><span data-stu-id="e7c8f-651">12 seconds</span></span> |<span data-ttu-id="e7c8f-652">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-652">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="e7c8f-653">Попытка 2 — задержка 1 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-653">Attempt 2 - delay ~1 sec</span></span><br /><span data-ttu-id="e7c8f-654">Попытка 3 — задержка 3 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-654">Attempt 3 - delay ~3 sec</span></span><br /><span data-ttu-id="e7c8f-655">Попытка 4 — задержка 7 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-655">Attempt 4 - delay ~7 sec</span></span><br /><span data-ttu-id="e7c8f-656">Попытка 5 — задержка 12 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-656">Attempt 5 - delay 12 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="e7c8f-657">Целевые значения сквозной задержки подразумевают время ожидания по умолчанию для подключения к службе.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-657">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="e7c8f-658">При указании более длинного времени ожидания подключения величина сквозной задержки будет увеличена путем добавления этого дополнительного времени для каждой попытки повтора.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-658">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>
>
>

### <a name="examples"></a><span data-ttu-id="e7c8f-659">Примеры</span><span class="sxs-lookup"><span data-stu-id="e7c8f-659">Examples</span></span>
<span data-ttu-id="e7c8f-660">В следующем примере кода определяется простое решение доступа к данным, использующее Entity Framework.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-660">The following code example defines a simple data access solution that uses Entity Framework.</span></span> <span data-ttu-id="e7c8f-661">Программа определяет конкретную стратегию путем определения класса с именем экземпляра **BlogConfiguration**, который расширяет **DbConfiguration**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-661">It sets a specific retry strategy by defining an instance of a class named **BlogConfiguration** that extends **DbConfiguration**.</span></span>

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

<span data-ttu-id="e7c8f-662">Дополнительные примеры использования механизма повтора Entity Framework можно найти в разделе [Устойчивость соединения и логика повтора](http://msdn.microsoft.com/data/dn456835.aspx).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-662">More examples of using the Entity Framework retry mechanism can be found in [Connection Resiliency / Retry Logic](http://msdn.microsoft.com/data/dn456835.aspx).</span></span>

### <a name="more-information"></a><span data-ttu-id="e7c8f-663">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="e7c8f-663">More information</span></span>
* [<span data-ttu-id="e7c8f-664">Руководство по эластичности и производительности базы данных SQL Azure</span><span class="sxs-lookup"><span data-stu-id="e7c8f-664">Azure SQL Database Performance and Elasticity Guide</span></span>](http://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core"></a><span data-ttu-id="e7c8f-665">Использование Базы данных SQL с Entity Framework Core</span><span class="sxs-lookup"><span data-stu-id="e7c8f-665">SQL Database using Entity Framework Core</span></span>
<span data-ttu-id="e7c8f-666">Механизм [Entity Framework Core](/ef/core/) отслеживает взаимоотношения между объектами и позволяет разработчикам .NET использовать объекты на уровне доменов при работе с данными.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-666">[Entity Framework Core](/ef/core/) is an object-relational mapper that enables .NET Core developers to work with data using domain-specific objects.</span></span> <span data-ttu-id="e7c8f-667">Это исключает необходимость в большинстве кодов доступа к данным, которые обычно требуется писать разработчикам.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-667">It eliminates the need for most of the data-access code that developers usually need to write.</span></span> <span data-ttu-id="e7c8f-668">Эта версия Entity Framework полностью создана с нуля и по умолчанию не наследует возможности EF6.x.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-668">This version of Entity Framework was written from the ground up, and doesn't automatically inherit all the features from EF6.x.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="e7c8f-669">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="e7c8f-669">Retry mechanism</span></span>
<span data-ttu-id="e7c8f-670">Поддержка повторных попыток при доступе к базе данных SQL с использованием Entity Framework Core предоставляется через механизм, называемый [Устойчивость подключения](/ef/core/miscellaneous/connection-resiliency).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-670">Retry support is provided when accessing SQL Database using Entity Framework Core through a mechanism called [Connection Resiliency](/ef/core/miscellaneous/connection-resiliency).</span></span> <span data-ttu-id="e7c8f-671">Устойчивость подключения впервые появилась в версии EF Core 1.1.0.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-671">Connection resiliency was introduced in EF Core 1.1.0.</span></span>

<span data-ttu-id="e7c8f-672">Основным объектом абстракции является интерфейс `IExecutionStrategy`.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-672">The primary abstraction is the `IExecutionStrategy` interface.</span></span> <span data-ttu-id="e7c8f-673">Стратегия выполнения для SQL Server, включая SQL Azure, учитывает типы исключений, для которых возможны повторные попытки. Для нее предусмотрены стандартные рациональные параметры, определяющие максимальное число повторных попыток, пауз между ними и т. д.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-673">The execution strategy for SQL Server, including SQL Azure, is aware of the exception types that can be retried and has sensible defaults for maximum retries, delay between retries, and so on.</span></span>

### <a name="examples"></a><span data-ttu-id="e7c8f-674">Примеры</span><span class="sxs-lookup"><span data-stu-id="e7c8f-674">Examples</span></span>

<span data-ttu-id="e7c8f-675">Следующий пример кода использует автоматические повторные попытки при настройке объекта DbContext, который представляет сеанс подключения к базе данных.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-675">The following code enables automatic retries when configuring the DbContext object, which represents a session with the database.</span></span> 

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

<span data-ttu-id="e7c8f-676">Следующий пример кода демонстрирует, как выполнять транзакцию с автоматическими повторными попытками с использованием стратегии выполнения.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-676">The following code shows how to execute a transaction with automatic retries, by using an execution strategy.</span></span> <span data-ttu-id="e7c8f-677">Транзакция определяется в делегате.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-677">The transaction is defined in a delegate.</span></span> <span data-ttu-id="e7c8f-678">Если происходит временный сбой, стратегия выполнения снова вызывает делегат.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-678">If a transient failure occurs, the execution strategy will invoke the delegate again.</span></span>

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

## <a name="azure-storage"></a><span data-ttu-id="e7c8f-679">Хранилище Azure</span><span class="sxs-lookup"><span data-stu-id="e7c8f-679">Azure Storage</span></span>
<span data-ttu-id="e7c8f-680">Службы хранилища Azure включают таблицы и BLOB-хранилища, файлы и очереди хранилища.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-680">Azure storage services include table and blob storage, files, and storage queues.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="e7c8f-681">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="e7c8f-681">Retry mechanism</span></span>
<span data-ttu-id="e7c8f-682">Повторы выполняются на уровне отдельных операции REST и являются неотъемлемой частью реализации API клиента.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-682">Retries occur at the individual REST operation level and are an integral part of the client API implementation.</span></span> <span data-ttu-id="e7c8f-683">Клиентский пакет SDK службы хранилища использует классы, реализующие [Интерфейс IExtendedRetryPolicy](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-683">The client storage SDK uses classes that implement the [IExtendedRetryPolicy Interface](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.aspx).</span></span>

<span data-ttu-id="e7c8f-684">Существуют различные реализации этого интерфейса.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-684">There are different implementations of the interface.</span></span> <span data-ttu-id="e7c8f-685">Клиенты службы хранилища могут выбирать политики специально для доступа к таблицам, BLOB-объектам и очередям.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-685">Storage clients can choose from policies specifically designed for accessing tables, blobs, and queues.</span></span> <span data-ttu-id="e7c8f-686">Каждая реализация использует различные стратегии повтора, фактически определяя интервал повторных попыток и другие параметры.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-686">Each implementation uses a different retry strategy that essentially defines the retry interval and other details.</span></span>

<span data-ttu-id="e7c8f-687">Встроенные классы обеспечивают поддержку для линейных (постоянная задержка) и экспоненциальных со случайными параметрами интервалов повтора.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-687">The built-in classes provide support for linear (constant delay) and exponential with randomization retry intervals.</span></span> <span data-ttu-id="e7c8f-688">Здесь также нет политики повторов для случаев, когда другой процесс обрабатывает повторы на более высоком уровне.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-688">There is also a no retry policy for use when another process is handling retries at a higher level.</span></span> <span data-ttu-id="e7c8f-689">Тем не менее если имеются особые требования, не предоставляемые встроенными классами, то можно реализовать собственные классы для обработки повторов.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-689">However, you can implement your own retry classes if you have specific requirements not provided by the built-in classes.</span></span>

<span data-ttu-id="e7c8f-690">Альтернативные повторы переключаются между первичным и вторичным расположением службы хранилища при использовании геоизбыточного хранилища с доступом на чтение (RA-GRS) и если результатом запроса стала ошибка.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-690">Alternate retries switch between primary and secondary storage service location if you are using read access geo-redundant storage (RA-GRS) and the result of the request is a retryable error.</span></span> <span data-ttu-id="e7c8f-691">Дополнительные сведения см. в статье [Варианты избыточности хранилища Azure](http://msdn.microsoft.com/library/azure/dn727290.aspx).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-691">See [Azure Storage Redundancy Options](http://msdn.microsoft.com/library/azure/dn727290.aspx) for more information.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="e7c8f-692">Настройки политики</span><span class="sxs-lookup"><span data-stu-id="e7c8f-692">Policy configuration</span></span>
<span data-ttu-id="e7c8f-693">Политики повтора настраиваются программным способом.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-693">Retry policies are configured programmatically.</span></span> <span data-ttu-id="e7c8f-694">Типичная процедура состоит в создании и заполнении экземпляров **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions** или **QueueRequestOptions**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-694">A typical procedure is to create and populate a **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, or **QueueRequestOptions** instance.</span></span>

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

<span data-ttu-id="e7c8f-695">Затем можно задать параметры экземпляра запроса на клиенте, и все операции с клиентом будут использовать указанные параметры запроса.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-695">The request options instance can then be set on the client, and all operations with the client will use the specified request options.</span></span>

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

<span data-ttu-id="e7c8f-696">Можно переопределить параметры запроса клиента путем передачи заполненного экземпляра класса параметров запроса в качестве параметра метода.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-696">You can override the client request options by passing a populated instance of the request options class as a parameter to operation methods.</span></span>

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

<span data-ttu-id="e7c8f-697">Можно использовать экземпляр **OperationContext** , чтобы указать код, выполняемый при возникновении повтора и после завершения операции.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-697">You use an **OperationContext** instance to specify the code to execute when a retry occurs and when an operation has completed.</span></span> <span data-ttu-id="e7c8f-698">Этот код может собирать сведения об операции для последующего использования в результатах телеметрии и журналах.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-698">This code can collect information about the operation for use in logs and telemetry.</span></span>

```csharp
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
```

<span data-ttu-id="e7c8f-699">Кроме определения, подходит ли сбой для повторных попыток, расширенные политики повтора возвращают объект **RetryContext**, который указывает число повторных попыток, результаты последнего запроса, произойдет ли следующая попытка в первичном или вторичном расположении (дополнительные сведения см. в таблице ниже).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-699">In addition to indicating whether a failure is suitable for retry, the extended retry policies return a **RetryContext** object that indicates the number of retries, the results of the last request, whether the next retry will happen in the primary or secondary location (see table below for details).</span></span> <span data-ttu-id="e7c8f-700">Свойства объекта **RetryContext** позволяют принять решение о необходимости и времени выполнения повтора.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-700">The properties of the **RetryContext** object can be used to decide if and when to attempt a retry.</span></span> <span data-ttu-id="e7c8f-701">Дополнительные сведения см. в статье [Метод IExtendedRetryPolicy.Evaluate](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-701">For more details, see [IExtendedRetryPolicy.Evaluate Method](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate.aspx).</span></span>

<span data-ttu-id="e7c8f-702">Следующие таблицы содержат значения по умолчанию для встроенной политики повторов.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-702">The following tables show the default settings for the built-in retry policies.</span></span>

<span data-ttu-id="e7c8f-703">**Параметры запроса**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-703">**Request options**</span></span>

| <span data-ttu-id="e7c8f-704">**Параметр**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-704">**Setting**</span></span> | <span data-ttu-id="e7c8f-705">**Значение по умолчанию**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-705">**Default value**</span></span> | <span data-ttu-id="e7c8f-706">**Значение**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-706">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="e7c8f-707">MaximumExecutionTime</span><span class="sxs-lookup"><span data-stu-id="e7c8f-707">MaximumExecutionTime</span></span> | <span data-ttu-id="e7c8f-708">None</span><span class="sxs-lookup"><span data-stu-id="e7c8f-708">None</span></span> | <span data-ttu-id="e7c8f-709">Максимальное время выполнения запроса, включая все потенциальные повторные попытки.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-709">Maximum execution time for the request, including all potential retry attempts.</span></span> <span data-ttu-id="e7c8f-710">Если оно не указано, то количество времени на выполнение запроса будет не ограничено.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-710">If it is not specified, then the amount of time that a request is permitted to take is unlimited.</span></span> <span data-ttu-id="e7c8f-711">Другими словами, запрос может зависнуть.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-711">In other words, the request might hang.</span></span> |
| <span data-ttu-id="e7c8f-712">ServerTimeOut</span><span class="sxs-lookup"><span data-stu-id="e7c8f-712">ServerTimeout</span></span> | <span data-ttu-id="e7c8f-713">None</span><span class="sxs-lookup"><span data-stu-id="e7c8f-713">None</span></span> | <span data-ttu-id="e7c8f-714">Интервал времени ожидания сервера для запроса (значение округляется до секунд).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-714">Server timeout interval for the request (value is rounded to seconds).</span></span> <span data-ttu-id="e7c8f-715">Если не указан, он будет использовать значение по умолчанию для всех запросов к серверу.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-715">If not specified, it will use the default value for all requests to the server.</span></span> <span data-ttu-id="e7c8f-716">Как правило, лучше всего опустить этот параметр, чтобы использовать значение сервера по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-716">Usually, the best option is to omit this setting so that the server default is used.</span></span> | 
| <span data-ttu-id="e7c8f-717">LocationMode</span><span class="sxs-lookup"><span data-stu-id="e7c8f-717">LocationMode</span></span> | <span data-ttu-id="e7c8f-718">None</span><span class="sxs-lookup"><span data-stu-id="e7c8f-718">None</span></span> | <span data-ttu-id="e7c8f-719">Если учетная запись хранения создается с вариантом репликации на географически избыточном хранилище с доступом для чтения (RA-GRS), то можно использовать режим расположения, чтобы указать расположение, где запрос должен быть получен.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-719">If the storage account is created with the Read access geo-redundant storage (RA-GRS) replication option, you can use the location mode to indicate which location should receive the request.</span></span> <span data-ttu-id="e7c8f-720">Например, если указано **PrimaryThenSecondary**, запросы всегда будут отправляться сначала в основное расположение.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-720">For example, if **PrimaryThenSecondary** is specified, requests are always sent to the primary location first.</span></span> <span data-ttu-id="e7c8f-721">Если запрос завершается ошибкой, он направляется во вторичное расположение.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-721">If a request fails, it is sent to the secondary location.</span></span> |
| <span data-ttu-id="e7c8f-722">политика RetryPolicy</span><span class="sxs-lookup"><span data-stu-id="e7c8f-722">RetryPolicy</span></span> | <span data-ttu-id="e7c8f-723">ExponentialPolicy</span><span class="sxs-lookup"><span data-stu-id="e7c8f-723">ExponentialPolicy</span></span> | <span data-ttu-id="e7c8f-724">Сведения о каждом параметре смотрите далее.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-724">See below for details of each option.</span></span> |

<span data-ttu-id="e7c8f-725">**Экспоненциальная политика**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-725">**Exponential policy**</span></span> 

| <span data-ttu-id="e7c8f-726">**Параметр**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-726">**Setting**</span></span> | <span data-ttu-id="e7c8f-727">**Значение по умолчанию**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-727">**Default value**</span></span> | <span data-ttu-id="e7c8f-728">**Значение**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-728">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="e7c8f-729">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="e7c8f-729">maxAttempt</span></span> | <span data-ttu-id="e7c8f-730">3</span><span class="sxs-lookup"><span data-stu-id="e7c8f-730">3</span></span> | <span data-ttu-id="e7c8f-731">Количество повторных попыток.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-731">Number of retry attempts.</span></span> |
| <span data-ttu-id="e7c8f-732">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="e7c8f-732">deltaBackoff</span></span> | <span data-ttu-id="e7c8f-733">4 секунды</span><span class="sxs-lookup"><span data-stu-id="e7c8f-733">4 seconds</span></span> | <span data-ttu-id="e7c8f-734">Интервал отсрочки между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-734">Back-off interval between retries.</span></span> <span data-ttu-id="e7c8f-735">Кратное timespan, включая элемент случая, который будет использоваться для последующих попыток.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-735">Multiples of this timespan, including a random element, will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="e7c8f-736">MinBackoff</span><span class="sxs-lookup"><span data-stu-id="e7c8f-736">MinBackoff</span></span> | <span data-ttu-id="e7c8f-737">3 секунды</span><span class="sxs-lookup"><span data-stu-id="e7c8f-737">3 seconds</span></span> | <span data-ttu-id="e7c8f-738">Добавить ко всем интервалам повтора, вычисленным по deltaBackoff.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-738">Added to all retry intervals computed from deltaBackoff.</span></span> <span data-ttu-id="e7c8f-739">Это значение не может быть изменено.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-739">This value cannot be changed.</span></span>
| <span data-ttu-id="e7c8f-740">MaxBackoff</span><span class="sxs-lookup"><span data-stu-id="e7c8f-740">MaxBackoff</span></span> | <span data-ttu-id="e7c8f-741">120 секунд</span><span class="sxs-lookup"><span data-stu-id="e7c8f-741">120 seconds</span></span> | <span data-ttu-id="e7c8f-742">MaxBackoff используется в том случае, если расчетный интервал повторных попыток оказывается больше, чем MaxBackoff.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-742">MaxBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> <span data-ttu-id="e7c8f-743">Это значение не может быть изменено.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-743">This value cannot be changed.</span></span> |

<span data-ttu-id="e7c8f-744">**Линейная политика**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-744">**Linear policy**</span></span>

| <span data-ttu-id="e7c8f-745">**Параметр**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-745">**Setting**</span></span> | <span data-ttu-id="e7c8f-746">**Значение по умолчанию**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-746">**Default value**</span></span> | <span data-ttu-id="e7c8f-747">**Значение**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-747">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="e7c8f-748">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="e7c8f-748">maxAttempt</span></span> | <span data-ttu-id="e7c8f-749">3</span><span class="sxs-lookup"><span data-stu-id="e7c8f-749">3</span></span> | <span data-ttu-id="e7c8f-750">Количество повторных попыток.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-750">Number of retry attempts.</span></span> |
| <span data-ttu-id="e7c8f-751">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="e7c8f-751">deltaBackoff</span></span> | <span data-ttu-id="e7c8f-752">30 секунд</span><span class="sxs-lookup"><span data-stu-id="e7c8f-752">30 seconds</span></span> | <span data-ttu-id="e7c8f-753">Интервал отсрочки между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-753">Back-off interval between retries.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="e7c8f-754">Руководство по использованию механизма повторов</span><span class="sxs-lookup"><span data-stu-id="e7c8f-754">Retry usage guidance</span></span>
<span data-ttu-id="e7c8f-755">При доступе к службам хранилища Azure с помощью API клиентской службы хранилища, придерживайтесь следующих рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-755">Consider the following guidelines when accessing Azure storage services using the storage client API:</span></span>

* <span data-ttu-id="e7c8f-756">Используйте встроенные политики повтора из пространства имен Microsoft.WindowsAzure.Storage.RetryPolicies, которые соответствуют требованиям.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-756">Use the built-in retry policies from the Microsoft.WindowsAzure.Storage.RetryPolicies namespace where they are appropriate for your requirements.</span></span> <span data-ttu-id="e7c8f-757">В большинстве случаев использования этих политик будет достаточно.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-757">In most cases, these policies will be sufficient.</span></span>
* <span data-ttu-id="e7c8f-758">Используйте политику **ExponentialRetry** для пакетных операций, фоновых задач или неинтерактивных сценариев.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-758">Use the **ExponentialRetry** policy in batch operations, background tasks, or non-interactive scenarios.</span></span> <span data-ttu-id="e7c8f-759">В этих сценариях обычно имеется больше времени для восстановления службы — таким образом повышается успешного выполнения операции.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-759">In these scenarios, you can typically allow more time for the service to recover—with a consequently increased chance of the operation eventually succeeding.</span></span>
* <span data-ttu-id="e7c8f-760">Следует указать свойства **MaximumExecutionTime** параметра **RequestOptions** для ограничения общего времени выполнения, а также принять во внимание тип и размер операции при выборе значения времени ожидания.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-760">Consider specifying the **MaximumExecutionTime** property of the **RequestOptions** parameter to limit the total execution time, but take into account the type and size of the operation when choosing a timeout value.</span></span>
* <span data-ttu-id="e7c8f-761">Если необходимо реализовать пользовательский повтор, избегайте создания оболочек клиентских классов хранения.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-761">If you need to implement a custom retry, avoid creating wrappers around the storage client classes.</span></span> <span data-ttu-id="e7c8f-762">Вместо этого используйте возможности для расширения существующих политик с помощью интерфейса **IExtendedRetryPolicy** .</span><span class="sxs-lookup"><span data-stu-id="e7c8f-762">Instead, use the capabilities to extend the existing policies through the **IExtendedRetryPolicy** interface.</span></span>
* <span data-ttu-id="e7c8f-763">Если вы используете геоизбыточное хранилище с доступом на чтение (RA-GRS), можно использовать **LocationMode** для указания, что повторные попытки должны будут выполняться для вторичной копии хранилища в случае сбоя доступа к первичному хранилищу.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-763">If you are using read access geo-redundant storage (RA-GRS) you can use the **LocationMode** to specify that retry attempts will access the secondary read-only copy of the store should the primary access fail.</span></span> <span data-ttu-id="e7c8f-764">Однако при использовании этого параметра необходимо убедиться, что ваше приложение может успешно работать с данными, которые могут устаревать, если процедура репликации с основным хранилищем еще не завершена.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-764">However, when using this option you must ensure that your application can work successfully with data that may be stale if the replication from the primary store has not yet completed.</span></span>

<span data-ttu-id="e7c8f-765">Начните с использования следующих параметров для операции повтора.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-765">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="e7c8f-766">Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-766">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>  

| <span data-ttu-id="e7c8f-767">**Контекст**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-767">**Context**</span></span> | <span data-ttu-id="e7c8f-768">**Максимальная задержка<br />примера целевого E2E**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-768">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="e7c8f-769">**Политика повтора**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-769">**Retry policy**</span></span> | <span data-ttu-id="e7c8f-770">**Параметры**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-770">**Settings**</span></span> | <span data-ttu-id="e7c8f-771">**Значения**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-771">**Values**</span></span> | <span data-ttu-id="e7c8f-772">**Принцип работы**</span><span class="sxs-lookup"><span data-stu-id="e7c8f-772">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="e7c8f-773">Интерактивный, пользовательский интерфейс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-773">Interactive, UI,</span></span><br /><span data-ttu-id="e7c8f-774">или передний план</span><span class="sxs-lookup"><span data-stu-id="e7c8f-774">or foreground</span></span> |<span data-ttu-id="e7c8f-775">2 секунды</span><span class="sxs-lookup"><span data-stu-id="e7c8f-775">2 seconds</span></span> |<span data-ttu-id="e7c8f-776">Линейная</span><span class="sxs-lookup"><span data-stu-id="e7c8f-776">Linear</span></span> |<span data-ttu-id="e7c8f-777">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="e7c8f-777">maxAttempt</span></span><br /><span data-ttu-id="e7c8f-778">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="e7c8f-778">deltaBackoff</span></span> |<span data-ttu-id="e7c8f-779">3</span><span class="sxs-lookup"><span data-stu-id="e7c8f-779">3</span></span><br /><span data-ttu-id="e7c8f-780">500 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-780">500 ms</span></span> |<span data-ttu-id="e7c8f-781">Попытка 1 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-781">Attempt 1 - delay 500 ms</span></span><br /><span data-ttu-id="e7c8f-782">Попытка 2 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-782">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="e7c8f-783">попытка 3 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="e7c8f-783">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="e7c8f-784">Фоновый</span><span class="sxs-lookup"><span data-stu-id="e7c8f-784">Background</span></span><br /><span data-ttu-id="e7c8f-785">или пакетный</span><span class="sxs-lookup"><span data-stu-id="e7c8f-785">or batch</span></span> |<span data-ttu-id="e7c8f-786">30 секунд</span><span class="sxs-lookup"><span data-stu-id="e7c8f-786">30 seconds</span></span> |<span data-ttu-id="e7c8f-787">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="e7c8f-787">Exponential</span></span> |<span data-ttu-id="e7c8f-788">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="e7c8f-788">maxAttempt</span></span><br /><span data-ttu-id="e7c8f-789">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="e7c8f-789">deltaBackoff</span></span> |<span data-ttu-id="e7c8f-790">5</span><span class="sxs-lookup"><span data-stu-id="e7c8f-790">5</span></span><br /><span data-ttu-id="e7c8f-791">4 секунды</span><span class="sxs-lookup"><span data-stu-id="e7c8f-791">4 seconds</span></span> |<span data-ttu-id="e7c8f-792">Попытка 1 — задержка ~ 3 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-792">Attempt 1 - delay ~3 sec</span></span><br /><span data-ttu-id="e7c8f-793">Попытка 2 — задержка 7 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-793">Attempt 2 - delay ~7 sec</span></span><br /><span data-ttu-id="e7c8f-794">Попытка 3 — задержка ~ 15 с</span><span class="sxs-lookup"><span data-stu-id="e7c8f-794">Attempt 3 - delay ~15 sec</span></span> |

### <a name="telemetry"></a><span data-ttu-id="e7c8f-795">Телеметрия</span><span class="sxs-lookup"><span data-stu-id="e7c8f-795">Telemetry</span></span>
<span data-ttu-id="e7c8f-796">Количество попыток входа регистрируется в **TraceSource**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-796">Retry attempts are logged to a **TraceSource**.</span></span> <span data-ttu-id="e7c8f-797">Необходимо настроить **TraceListener** для сбора данных о событиях и записи этих данных в журнал с подходящим назначением.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-797">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span> <span data-ttu-id="e7c8f-798">Можно использовать **TextWriterTraceListener** или **XmlWriterTraceListener** для записи данных в файл журнала, **EventLogTraceListener** для записи в журнал событий Windows или **EventProviderTraceListener** для записи данных трассировки в подсистеме трассировки событий Windows.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-798">You can use the **TextWriterTraceListener** or **XmlWriterTraceListener** to write the data to a log file, the **EventLogTraceListener** to write to the Windows Event Log, or the **EventProviderTraceListener** to write trace data to the ETW subsystem.</span></span> <span data-ttu-id="e7c8f-799">Можно также настроить автоматическую очистку буфера и уровень детализации регистрируемых событий (например, Ошибка, Предупреждение, Информационное событие и Подробное протоколирование).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-799">You can also configure auto-flushing of the buffer, and the verbosity of events that will be logged (for example, Error, Warning, Informational, and Verbose).</span></span> <span data-ttu-id="e7c8f-800">Дополнительные сведения см. в статье [Вход в клиентскую библиотеку хранилища .NET на стороне клиента](http://msdn.microsoft.com/library/azure/dn782839.aspx).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-800">For more information, see [Client-side Logging with the .NET Storage Client Library](http://msdn.microsoft.com/library/azure/dn782839.aspx).</span></span>

<span data-ttu-id="e7c8f-801">Операции могут получать экземпляр **OperationContext**, предоставляющий событие **Повтор**, которое может использоваться для присоединения телеметрии пользовательской логики.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-801">Operations can receive an **OperationContext** instance, which exposes a **Retrying** event that can be used to attach custom telemetry logic.</span></span> <span data-ttu-id="e7c8f-802">Дополнительные сведения см. в статье [Событие OperationContext.Retrying](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span><span class="sxs-lookup"><span data-stu-id="e7c8f-802">For more information, see [OperationContext.Retrying Event](http://msdn.microsoft.com/library/microsoft.windowsazure.storage.operationcontext.retrying.aspx).</span></span>

### <a name="examples"></a><span data-ttu-id="e7c8f-803">Примеры</span><span class="sxs-lookup"><span data-stu-id="e7c8f-803">Examples</span></span>
<span data-ttu-id="e7c8f-804">В следующем примере кода показано, как создать два экземпляра **TableRequestOptions** с различными параметрами, один для интерактивных запросов и один для фоновых запросов.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-804">The following code example shows how to create two **TableRequestOptions** instances with different retry settings; one for interactive requests and one for background requests.</span></span> <span data-ttu-id="e7c8f-805">Пример затем устанавливает эти две политики повторов на клиентском компьютере, чтобы они применялись для всех запросов, а также задает интерактивную стратегию на конкретный запрос, чтобы он переопределял параметры по умолчанию, применяемые к клиенту.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-805">The example then sets these two retry policies on the client so that they apply for all requests, and also sets the interactive strategy on a specific request so that it overrides the default settings applied to the client.</span></span>

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

### <a name="more-information"></a><span data-ttu-id="e7c8f-806">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="e7c8f-806">More information</span></span>
* [<span data-ttu-id="e7c8f-807">Рекомендации по настройке политики повтора для клиентской библиотеки службы хранилища Azure</span><span class="sxs-lookup"><span data-stu-id="e7c8f-807">Azure Storage Client Library Retry Policy Recommendations</span></span>](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)
* [<span data-ttu-id="e7c8f-808">Клиентская библиотека службы хранилища 2.0: реализация политики повтора</span><span class="sxs-lookup"><span data-stu-id="e7c8f-808">Storage Client Library 2.0 – Implementing Retry Policies</span></span>](http://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="general-rest-and-retry-guidelines"></a><span data-ttu-id="e7c8f-809">Общие рекомендации по использованию REST и механизма повторов</span><span class="sxs-lookup"><span data-stu-id="e7c8f-809">General REST and retry guidelines</span></span>
<span data-ttu-id="e7c8f-810">При обращении к службам Azure или службам сторонних производителей учитывайте следующее.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-810">Consider the following when accessing Azure or third party services:</span></span>

* <span data-ttu-id="e7c8f-811">Используйте систематический подход к управлению механизмом повторов, возможно в виде повторно используемого кода, что таким образом дает возможность применения согласованной методологии ко всем клиентам и всем решениям.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-811">Use a systematic approach to managing retries, perhaps as reusable code, so that you can apply a consistent methodology across all clients and all solutions.</span></span>
* <span data-ttu-id="e7c8f-812">Если целевая служба или клиент не имеет встроенного механизма повторов, рекомендуется использовать платформы повторов, например [Polly][polly], для управления механизмом повторов.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-812">Consider using a retry framework such as [Polly][polly] to manage retries if the target service or client has no built-in retry mechanism.</span></span> <span data-ttu-id="e7c8f-813">Это поможет реализовать согласованный механизм повтора и может предоставить подходящую стратегию повторов по умолчанию для целевой службы.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-813">This will help you implement a consistent retry behavior, and it may provide a suitable default retry strategy for the target service.</span></span> <span data-ttu-id="e7c8f-814">Тем не менее необходимо создать пользовательский код механизма повторов для служб, имеющих нестандартное поведение, которые не полагаются на исключения для определения временных сбоев, или если вы хотите использовать ответ **Retry-Response** для управления поведением механизма повторов.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-814">However, you may need to create custom retry code for services that have non-standard behavior, that do not rely on exceptions to indicate transient failures, or if you want to use a **Retry-Response** reply to manage retry behavior.</span></span>
* <span data-ttu-id="e7c8f-815">Логика обнаружения временных сбоев будет зависеть от фактического клиентского API, который используется для вызова REST.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-815">The transient detection logic will depend on the actual client API you use to invoke the REST calls.</span></span> <span data-ttu-id="e7c8f-816">Некоторые клиенты, например новый класс **HttpClient** , не создают исключения для запросов, завершенных с неуспешным кодом состояния HTTP.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-816">Some clients, such as the newer **HttpClient** class, will not throw exceptions for completed requests with a non-success HTTP status code.</span></span> 
* <span data-ttu-id="e7c8f-817">Код состояния HTTP, возвращаемый службой, позволяет указать, является ли ошибка временной.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-817">The HTTP status code returned from the service can help to indicate whether the failure is transient.</span></span> <span data-ttu-id="e7c8f-818">Необходимо изучить исключения, генерируемые клиентом, или механизм повторов для доступа к коду состояния для определения эквивалентного типа исключения.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-818">You may need to examine the exceptions generated by a client or the retry framework to access the status code or to determine the equivalent exception type.</span></span> <span data-ttu-id="e7c8f-819">Следующие коды HTTP обычно показывают, что повтор может быть выполнен.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-819">The following HTTP codes typically indicate that a retry is appropriate:</span></span>
  * <span data-ttu-id="e7c8f-820">408 — Истекло время ожидания запроса</span><span class="sxs-lookup"><span data-stu-id="e7c8f-820">408 Request Timeout</span></span>
  * <span data-ttu-id="e7c8f-821">429 — слишком много запросов</span><span class="sxs-lookup"><span data-stu-id="e7c8f-821">429 Too Many Requests</span></span>
  * <span data-ttu-id="e7c8f-822">500 — Внутренняя ошибка сервера</span><span class="sxs-lookup"><span data-stu-id="e7c8f-822">500 Internal Server Error</span></span>
  * <span data-ttu-id="e7c8f-823">502 — Недопустимый шлюз</span><span class="sxs-lookup"><span data-stu-id="e7c8f-823">502 Bad Gateway</span></span>
  * <span data-ttu-id="e7c8f-824">503 — Служба недоступна</span><span class="sxs-lookup"><span data-stu-id="e7c8f-824">503 Service Unavailable</span></span>
  * <span data-ttu-id="e7c8f-825">504 — Истекло время ожидания шлюза</span><span class="sxs-lookup"><span data-stu-id="e7c8f-825">504 Gateway Timeout</span></span>
* <span data-ttu-id="e7c8f-826">Если логика повторов основана на исключениях, следующие параметры обычно указывают, что произошел временный сбой где не удалось создать подключение.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-826">If you base your retry logic on exceptions, the following typically indicate a transient failure where no connection could be established:</span></span>
  * <span data-ttu-id="e7c8f-827">WebExceptionStatus.ConnectionClosed</span><span class="sxs-lookup"><span data-stu-id="e7c8f-827">WebExceptionStatus.ConnectionClosed</span></span>
  * <span data-ttu-id="e7c8f-828">WebExceptionStatus.ConnectFailure</span><span class="sxs-lookup"><span data-stu-id="e7c8f-828">WebExceptionStatus.ConnectFailure</span></span>
  * <span data-ttu-id="e7c8f-829">WebExceptionStatus.Timeout</span><span class="sxs-lookup"><span data-stu-id="e7c8f-829">WebExceptionStatus.Timeout</span></span>
  * <span data-ttu-id="e7c8f-830">WebExceptionStatus.RequestCanceled</span><span class="sxs-lookup"><span data-stu-id="e7c8f-830">WebExceptionStatus.RequestCanceled</span></span>
* <span data-ttu-id="e7c8f-831">Если служба недоступна, она может передать прогнозируемое значение задержки перед следующей повторной попыткой в заголовке ответа **Retry-After** или другом пользовательском заголовке.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-831">In the case of a service unavailable status, the service might indicate the appropriate delay before retrying in the **Retry-After** response header or a different custom header.</span></span> <span data-ttu-id="e7c8f-832">Службы также отправляют дополнительную информацию в виде пользовательских заголовков или внедренной в содержимое ответа.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-832">Services might also send additional information as custom headers, or embedded in the content of the response.</span></span> 
* <span data-ttu-id="e7c8f-833">Не выполняйте повторы для кодов состояния, представляющих клиентские ошибки (ошибки в диапазоне 4xx) за исключением ошибки «408 — Истекло время ожидания запроса».</span><span class="sxs-lookup"><span data-stu-id="e7c8f-833">Do not retry for status codes representing client errors (errors in the 4xx range) except for a 408 Request Timeout.</span></span>
* <span data-ttu-id="e7c8f-834">Тщательно протестируйте свои стратегии и механизмы повторов для различных условий, например, указав другую сеть и с различными нагрузками системы.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-834">Thoroughly test your retry strategies and mechanisms under a range of conditions, such as different network states and varying system loadings.</span></span>

### <a name="retry-strategies"></a><span data-ttu-id="e7c8f-835">Стратегия повторов</span><span class="sxs-lookup"><span data-stu-id="e7c8f-835">Retry strategies</span></span>
<span data-ttu-id="e7c8f-836">Ниже приведены типичные виды интервалов стратегий повтора.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-836">The following are the typical types of retry strategy intervals:</span></span>

* <span data-ttu-id="e7c8f-837">**Экспоненциальная**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-837">**Exponential**.</span></span> <span data-ttu-id="e7c8f-838">Политика повторов, основанная на выполнении указанного числа повторных попыток и использующая случайный экспоненциальный пассивный метод для определения интервала между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-838">A retry policy that performs a specified number of retries, using a randomized exponential back off approach to determine the interval between retries.</span></span> <span data-ttu-id="e7c8f-839">Например: </span><span class="sxs-lookup"><span data-stu-id="e7c8f-839">For example:</span></span>

    ```csharp
    var random = new Random();

    var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
    var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                    this.maxBackoff.TotalMilliseconds);
    retryInterval = TimeSpan.FromMilliseconds(interval);
    ```

* <span data-ttu-id="e7c8f-840">**Добавочная**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-840">**Incremental**.</span></span> <span data-ttu-id="e7c8f-841">Стратегия повторов с указанным числом повторных попыток и добавочным интервалом между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-841">A retry strategy with a specified number of retry attempts and an incremental time interval between retries.</span></span> <span data-ttu-id="e7c8f-842">Например: </span><span class="sxs-lookup"><span data-stu-id="e7c8f-842">For example:</span></span>

    ```csharp
    retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                    (this.increment.TotalMilliseconds * currentRetryCount));
    ```

* <span data-ttu-id="e7c8f-843">**Линейный повтор**.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-843">**LinearRetry**.</span></span> <span data-ttu-id="e7c8f-844">Политика повторов, выполняющая указанное число повторных попыток и использующая указанный фиксированный интервал времени между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-844">A retry policy that performs a specified number of retries, using a specified fixed time interval between retries.</span></span> <span data-ttu-id="e7c8f-845">Например: </span><span class="sxs-lookup"><span data-stu-id="e7c8f-845">For example:</span></span>

    ```csharp
    retryInterval = this.deltaBackoff;
    ```

### <a name="transient-fault-handling-with-polly"></a><span data-ttu-id="e7c8f-846">Обработка временного сбоя с помощью Polly</span><span class="sxs-lookup"><span data-stu-id="e7c8f-846">Transient fault handling with Polly</span></span>
<span data-ttu-id="e7c8f-847">Библиотека [Polly][polly] предназначена для программной обработки механизма повторных попыток и [стратегий автоматического прерывания][circuit-breaker].</span><span class="sxs-lookup"><span data-stu-id="e7c8f-847">[Polly][polly] is a library to programatically handle retries and [circuit breaker][circuit-breaker] strategies.</span></span> <span data-ttu-id="e7c8f-848">Проект Polly является частью [.NET Foundation][dotnet-foundation].</span><span class="sxs-lookup"><span data-stu-id="e7c8f-848">The Polly project is a member of the [.NET Foundation][dotnet-foundation].</span></span> <span data-ttu-id="e7c8f-849">Применение Polly будет неплохим вариантом при работе со службами, клиент которых не имеет встроенного механизма повторных попыток. Вам не придется самостоятельно создавать код для повторных попыток, который иногда очень трудно реализовать правильно.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-849">For services where the client does not natively support retries, Polly is a valid alternative and avoids the need to write custom retry code, which can be hard to implement correctly.</span></span> <span data-ttu-id="e7c8f-850">Polly также поддерживает трассировку возникающих ошибок, что позволяет вести журнал повторных попыток.</span><span class="sxs-lookup"><span data-stu-id="e7c8f-850">Polly also provides a way to trace errors when they occur, so that you can log retries.</span></span>


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


### <a name="more-information"></a><span data-ttu-id="e7c8f-854">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="e7c8f-854">More information</span></span>
* [<span data-ttu-id="e7c8f-855">Устойчивость подключения</span><span class="sxs-lookup"><span data-stu-id="e7c8f-855">Connection Resiliency</span></span>](/ef/core/miscellaneous/connection-resiliency)
* [<span data-ttu-id="e7c8f-856">Точки данных для EF Core 1.1</span><span class="sxs-lookup"><span data-stu-id="e7c8f-856">Data Points - EF Core 1.1</span></span>](https://msdn.microsoft.com/magazine/mt745093.aspx)


