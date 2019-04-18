---
title: Конкретные рекомендации по использованию механизма повторов
titleSuffix: Best practices for cloud applications
description: Конкретные рекомендации по настройке механизма повторов.
author: dragon119
ms.date: 08/13/2018
ms.topic: best-practice
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 170a38f6b8a6c107670561e63f236e43af948d7d
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59641048"
---
# <a name="retry-guidance-for-specific-services"></a><span data-ttu-id="c6d75-103">Руководство по использованию механизма повторов для отдельных служб</span><span class="sxs-lookup"><span data-stu-id="c6d75-103">Retry guidance for specific services</span></span>

<span data-ttu-id="c6d75-104">Большинство служб Azure и клиентских пакетов SDK содержат механизм повтора.</span><span class="sxs-lookup"><span data-stu-id="c6d75-104">Most Azure services and client SDKs include a retry mechanism.</span></span> <span data-ttu-id="c6d75-105">В то же время они отличаются, поскольку каждая служба имеет разные характеристики и требования, поэтому каждый механизм повтора настроен для конкретной службы.</span><span class="sxs-lookup"><span data-stu-id="c6d75-105">However, these differ because each service has different characteristics and requirements, and so each retry mechanism is tuned to a specific service.</span></span> <span data-ttu-id="c6d75-106">В этом руководстве представлены возможности механизма повтора для большинства служб Azure, а также сведения, помогающие использовать, адаптировать или расширять механизм повтора для такой службы.</span><span class="sxs-lookup"><span data-stu-id="c6d75-106">This guide summarizes the retry mechanism features for the majority of Azure services, and includes information to help you use, adapt, or extend the retry mechanism for that service.</span></span>

<span data-ttu-id="c6d75-107">Для получения общих рекомендаций по обработке временных сбоев и выполнения повторных попыток подключения и операций со службами и ресурсами обратитесь к разделу [Руководство по повторам](./transient-faults.md).</span><span class="sxs-lookup"><span data-stu-id="c6d75-107">For general guidance on handling transient faults, and retrying connections and operations against services and resources, see [Retry guidance](./transient-faults.md).</span></span>

<span data-ttu-id="c6d75-108">В следующей таблице перечислены функции механизма повтора для служб Azure, описанных в этом руководстве.</span><span class="sxs-lookup"><span data-stu-id="c6d75-108">The following table summarizes the retry features for the Azure services described in this guidance.</span></span>

| <span data-ttu-id="c6d75-109">**Служба**</span><span class="sxs-lookup"><span data-stu-id="c6d75-109">**Service**</span></span> | <span data-ttu-id="c6d75-110">**Ключевые возможности**</span><span class="sxs-lookup"><span data-stu-id="c6d75-110">**Retry capabilities**</span></span> | <span data-ttu-id="c6d75-111">**Настройки политики**</span><span class="sxs-lookup"><span data-stu-id="c6d75-111">**Policy configuration**</span></span> | <span data-ttu-id="c6d75-112">**Область**</span><span class="sxs-lookup"><span data-stu-id="c6d75-112">**Scope**</span></span> | <span data-ttu-id="c6d75-113">**Функции телеметрии**</span><span class="sxs-lookup"><span data-stu-id="c6d75-113">**Telemetry features**</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="c6d75-114">**[Azure Active Directory](#azure-active-directory)**</span><span class="sxs-lookup"><span data-stu-id="c6d75-114">**[Azure Active Directory](#azure-active-directory)**</span></span> |<span data-ttu-id="c6d75-115">Машинный код в библиотеке ADAL</span><span class="sxs-lookup"><span data-stu-id="c6d75-115">Native in ADAL library</span></span> |<span data-ttu-id="c6d75-116">Внедренные в библиотеку ADAL</span><span class="sxs-lookup"><span data-stu-id="c6d75-116">Embedded into ADAL library</span></span> |<span data-ttu-id="c6d75-117">Внутренний</span><span class="sxs-lookup"><span data-stu-id="c6d75-117">Internal</span></span> |<span data-ttu-id="c6d75-118">Нет</span><span class="sxs-lookup"><span data-stu-id="c6d75-118">None</span></span> |
| <span data-ttu-id="c6d75-119">**[Cosmos DB](#cosmos-db)**</span><span class="sxs-lookup"><span data-stu-id="c6d75-119">**[Cosmos DB](#cosmos-db)**</span></span> |<span data-ttu-id="c6d75-120">Машинный код в службе</span><span class="sxs-lookup"><span data-stu-id="c6d75-120">Native in service</span></span> |<span data-ttu-id="c6d75-121">Ненастраиваемые</span><span class="sxs-lookup"><span data-stu-id="c6d75-121">Non-configurable</span></span> |<span data-ttu-id="c6d75-122">Глобальные</span><span class="sxs-lookup"><span data-stu-id="c6d75-122">Global</span></span> |<span data-ttu-id="c6d75-123">TraceSource</span><span class="sxs-lookup"><span data-stu-id="c6d75-123">TraceSource</span></span> |
| <span data-ttu-id="c6d75-124">**Data Lake Store**</span><span class="sxs-lookup"><span data-stu-id="c6d75-124">**Data Lake Store**</span></span> |<span data-ttu-id="c6d75-125">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="c6d75-125">Native in client</span></span> |<span data-ttu-id="c6d75-126">Ненастраиваемые</span><span class="sxs-lookup"><span data-stu-id="c6d75-126">Non-configurable</span></span> |<span data-ttu-id="c6d75-127">Индивидуальные операции</span><span class="sxs-lookup"><span data-stu-id="c6d75-127">Individual operations</span></span> |<span data-ttu-id="c6d75-128">Нет</span><span class="sxs-lookup"><span data-stu-id="c6d75-128">None</span></span> |
| <span data-ttu-id="c6d75-129">**[Центры событий](#event-hubs)**</span><span class="sxs-lookup"><span data-stu-id="c6d75-129">**[Event Hubs](#event-hubs)**</span></span> |<span data-ttu-id="c6d75-130">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="c6d75-130">Native in client</span></span> |<span data-ttu-id="c6d75-131">Программный</span><span class="sxs-lookup"><span data-stu-id="c6d75-131">Programmatic</span></span> |<span data-ttu-id="c6d75-132">Клиент</span><span class="sxs-lookup"><span data-stu-id="c6d75-132">Client</span></span> |<span data-ttu-id="c6d75-133">Нет</span><span class="sxs-lookup"><span data-stu-id="c6d75-133">None</span></span> |
| <span data-ttu-id="c6d75-134">**[Центр Интернета вещей](#iot-hub)**</span><span class="sxs-lookup"><span data-stu-id="c6d75-134">**[IoT Hub](#iot-hub)**</span></span> |<span data-ttu-id="c6d75-135">Машинный код в клиентском пакете SDK</span><span class="sxs-lookup"><span data-stu-id="c6d75-135">Native in client SDK</span></span> |<span data-ttu-id="c6d75-136">Программный</span><span class="sxs-lookup"><span data-stu-id="c6d75-136">Programmatic</span></span> |<span data-ttu-id="c6d75-137">Клиент</span><span class="sxs-lookup"><span data-stu-id="c6d75-137">Client</span></span> |<span data-ttu-id="c6d75-138">Нет</span><span class="sxs-lookup"><span data-stu-id="c6d75-138">None</span></span> |
| <span data-ttu-id="c6d75-139">**[Кэш Redis](#azure-redis-cache)**</span><span class="sxs-lookup"><span data-stu-id="c6d75-139">**[Redis Cache](#azure-redis-cache)**</span></span> |<span data-ttu-id="c6d75-140">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="c6d75-140">Native in client</span></span> |<span data-ttu-id="c6d75-141">Программный</span><span class="sxs-lookup"><span data-stu-id="c6d75-141">Programmatic</span></span> |<span data-ttu-id="c6d75-142">Клиент</span><span class="sxs-lookup"><span data-stu-id="c6d75-142">Client</span></span> |<span data-ttu-id="c6d75-143">TextWriter</span><span class="sxs-lookup"><span data-stu-id="c6d75-143">TextWriter</span></span> |
| <span data-ttu-id="c6d75-144">**[Служба поиска](#azure-search)**</span><span class="sxs-lookup"><span data-stu-id="c6d75-144">**[Search](#azure-search)**</span></span> |<span data-ttu-id="c6d75-145">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="c6d75-145">Native in client</span></span> |<span data-ttu-id="c6d75-146">Программный</span><span class="sxs-lookup"><span data-stu-id="c6d75-146">Programmatic</span></span> |<span data-ttu-id="c6d75-147">Клиент</span><span class="sxs-lookup"><span data-stu-id="c6d75-147">Client</span></span> |<span data-ttu-id="c6d75-148">Трассировка событий Windows или пользовательская</span><span class="sxs-lookup"><span data-stu-id="c6d75-148">ETW or Custom</span></span> |
| <span data-ttu-id="c6d75-149">**[Служебная шина](#service-bus)**</span><span class="sxs-lookup"><span data-stu-id="c6d75-149">**[Service Bus](#service-bus)**</span></span> |<span data-ttu-id="c6d75-150">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="c6d75-150">Native in client</span></span> |<span data-ttu-id="c6d75-151">Программный</span><span class="sxs-lookup"><span data-stu-id="c6d75-151">Programmatic</span></span> |<span data-ttu-id="c6d75-152">Диспетчер пространств имен, фабрика сообщений и клиент</span><span class="sxs-lookup"><span data-stu-id="c6d75-152">Namespace Manager, Messaging Factory, and Client</span></span> |<span data-ttu-id="c6d75-153">Трассировка событий Windows</span><span class="sxs-lookup"><span data-stu-id="c6d75-153">ETW</span></span> |
| <span data-ttu-id="c6d75-154">**[Service Fabric](#service-fabric)**</span><span class="sxs-lookup"><span data-stu-id="c6d75-154">**[Service Fabric](#service-fabric)**</span></span> |<span data-ttu-id="c6d75-155">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="c6d75-155">Native in client</span></span> |<span data-ttu-id="c6d75-156">Программный</span><span class="sxs-lookup"><span data-stu-id="c6d75-156">Programmatic</span></span> |<span data-ttu-id="c6d75-157">Клиент</span><span class="sxs-lookup"><span data-stu-id="c6d75-157">Client</span></span> |<span data-ttu-id="c6d75-158">Нет</span><span class="sxs-lookup"><span data-stu-id="c6d75-158">None</span></span> |
| <span data-ttu-id="c6d75-159">**[База данных SQL с ADO.NET](#sql-database-using-adonet)**</span><span class="sxs-lookup"><span data-stu-id="c6d75-159">**[SQL Database with ADO.NET](#sql-database-using-adonet)**</span></span> |[<span data-ttu-id="c6d75-160">Polly</span><span class="sxs-lookup"><span data-stu-id="c6d75-160">Polly</span></span>](#transient-fault-handling-with-polly) |<span data-ttu-id="c6d75-161">Декларативные и программные</span><span class="sxs-lookup"><span data-stu-id="c6d75-161">Declarative and programmatic</span></span> |<span data-ttu-id="c6d75-162">Единые инструкции или блоки кода</span><span class="sxs-lookup"><span data-stu-id="c6d75-162">Single statements or blocks of code</span></span> |<span data-ttu-id="c6d75-163">Пользовательская</span><span class="sxs-lookup"><span data-stu-id="c6d75-163">Custom</span></span> |
| <span data-ttu-id="c6d75-164">**[База данных SQL с Entity Framework](#sql-database-using-entity-framework-6)**</span><span class="sxs-lookup"><span data-stu-id="c6d75-164">**[SQL Database with Entity Framework](#sql-database-using-entity-framework-6)**</span></span> |<span data-ttu-id="c6d75-165">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="c6d75-165">Native in client</span></span> |<span data-ttu-id="c6d75-166">Программный</span><span class="sxs-lookup"><span data-stu-id="c6d75-166">Programmatic</span></span> |<span data-ttu-id="c6d75-167">Глобальные каждого домена приложения</span><span class="sxs-lookup"><span data-stu-id="c6d75-167">Global per AppDomain</span></span> |<span data-ttu-id="c6d75-168">Нет</span><span class="sxs-lookup"><span data-stu-id="c6d75-168">None</span></span> |
| <span data-ttu-id="c6d75-169">**[База данных SQL с Entity Framework Core](#sql-database-using-entity-framework-core)**</span><span class="sxs-lookup"><span data-stu-id="c6d75-169">**[SQL Database with Entity Framework Core](#sql-database-using-entity-framework-core)**</span></span> |<span data-ttu-id="c6d75-170">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="c6d75-170">Native in client</span></span> |<span data-ttu-id="c6d75-171">Программный</span><span class="sxs-lookup"><span data-stu-id="c6d75-171">Programmatic</span></span> |<span data-ttu-id="c6d75-172">Глобальные каждого домена приложения</span><span class="sxs-lookup"><span data-stu-id="c6d75-172">Global per AppDomain</span></span> |<span data-ttu-id="c6d75-173">Нет</span><span class="sxs-lookup"><span data-stu-id="c6d75-173">None</span></span> |
| <span data-ttu-id="c6d75-174">**[Хранилище](#azure-storage)**</span><span class="sxs-lookup"><span data-stu-id="c6d75-174">**[Storage](#azure-storage)**</span></span> |<span data-ttu-id="c6d75-175">Машинный код в клиенте</span><span class="sxs-lookup"><span data-stu-id="c6d75-175">Native in client</span></span> |<span data-ttu-id="c6d75-176">Программный</span><span class="sxs-lookup"><span data-stu-id="c6d75-176">Programmatic</span></span> |<span data-ttu-id="c6d75-177">Клиентские и индивидуальные операции</span><span class="sxs-lookup"><span data-stu-id="c6d75-177">Client and individual operations</span></span> |<span data-ttu-id="c6d75-178">TraceSource</span><span class="sxs-lookup"><span data-stu-id="c6d75-178">TraceSource</span></span> |

> [!NOTE]
> <span data-ttu-id="c6d75-179">Сейчас для большинства встроенных механизмов повтора Azure невозможно применить другую политику повтора для других типов ошибок и исключений.</span><span class="sxs-lookup"><span data-stu-id="c6d75-179">For most of the Azure built-in retry mechanisms, there is currently no way apply a different retry policy for different types of error or exception.</span></span> <span data-ttu-id="c6d75-180">Вам следует настроить политику, которая обеспечит оптимальный средний уровень производительности и доступности.</span><span class="sxs-lookup"><span data-stu-id="c6d75-180">You should configure a policy that provides the optimum average performance and availability.</span></span> <span data-ttu-id="c6d75-181">Одним из способов настройки политики является анализ файлов журнала для определения типа возникающих временных сбоев.</span><span class="sxs-lookup"><span data-stu-id="c6d75-181">One way to fine-tune the policy is to analyze log files to determine the type of transient faults that are occurring.</span></span>

<!-- markdownlint-disable MD024 MD033 -->

## <a name="azure-active-directory"></a><span data-ttu-id="c6d75-182">Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="c6d75-182">Azure Active Directory</span></span>

<span data-ttu-id="c6d75-183">Azure Active Directory (Azure AD) — это комплексное решение по управлению идентификацией и доступом к облаку, которое включает службы основных каталогов, функции расширенного управления удостоверениями и доступа к приложениям, а также средства обеспечения безопасности.</span><span class="sxs-lookup"><span data-stu-id="c6d75-183">Azure Active Directory (Azure AD) is a comprehensive identity and access management cloud solution that combines core directory services, advanced identity governance, security, and application access management.</span></span> <span data-ttu-id="c6d75-184">Azure AD также предлагает разработчикам платформу управления удостоверениями для доставки элементов контроля доступа к приложениям на основе централизованной политики и правил.</span><span class="sxs-lookup"><span data-stu-id="c6d75-184">Azure AD also offers developers an identity management platform to deliver access control to their applications, based on centralized policy and rules.</span></span>

> [!NOTE]
> <span data-ttu-id="c6d75-185">Для получения рекомендаций по удостоверению управляемой службы конечных точек см. статью [Использование управляемого удостоверения службы (MSI) виртуальной машины Azure для получения маркера](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling)</span><span class="sxs-lookup"><span data-stu-id="c6d75-185">For retry guidance on Managed Service Identity endpoints, see [How to use an Azure VM Managed Service Identity (MSI) for token acquisition](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling).</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="c6d75-186">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="c6d75-186">Retry mechanism</span></span>

<span data-ttu-id="c6d75-187">Библиотека аутентификации для Active Directory (ADAL) имеет встроенный механизм повторов для службы Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="c6d75-187">There is a built-in retry mechanism for Azure Active Directory in the Active Directory Authentication Library (ADAL).</span></span> <span data-ttu-id="c6d75-188">Чтобы избежать непредвиденных блокировок, мы рекомендуем разработчикам **не** применять повторные попытки в своих библиотеках и приложениях, а использовать для этого библиотеку ADAL.</span><span class="sxs-lookup"><span data-stu-id="c6d75-188">To avoid unexpected lockouts, we recommend that third party libraries and application code do **not** retry failed connections, but allow ADAL to handle retries.</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="c6d75-189">Руководство по использованию механизма повторов</span><span class="sxs-lookup"><span data-stu-id="c6d75-189">Retry usage guidance</span></span>

<span data-ttu-id="c6d75-190">При использовании Azure Active Directory придерживайтесь следующих рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="c6d75-190">Consider the following guidelines when using Azure Active Directory:</span></span>

- <span data-ttu-id="c6d75-191">Везде, где это возможно, используйте библиотеку ADAL и ее встроенный механизм повторных попыток.</span><span class="sxs-lookup"><span data-stu-id="c6d75-191">When possible, use the ADAL library and the built-in support for retries.</span></span>
- <span data-ttu-id="c6d75-192">Если вы используете REST API для Azure Active Directory и получаете код результата 429 (слишком много запросов) или ошибку в диапазоне 5xx, повторите операцию.</span><span class="sxs-lookup"><span data-stu-id="c6d75-192">If you are using the REST API for Azure Active Directory, retry the operation if the result code is 429 (Too Many Requests) or an error in the 5xx range.</span></span> <span data-ttu-id="c6d75-193">Не следует выполнять повтор при возникновении других ошибок.</span><span class="sxs-lookup"><span data-stu-id="c6d75-193">Do not retry for any other errors.</span></span>
- <span data-ttu-id="c6d75-194">В пакетных сценариях с Azure Active Directory рекомендуется использование экспоненциальных политик отсрочки.</span><span class="sxs-lookup"><span data-stu-id="c6d75-194">An exponential back-off policy is recommended for use in batch scenarios with Azure Active Directory.</span></span>

<span data-ttu-id="c6d75-195">Для начала установите следующие параметры для операций повтора.</span><span class="sxs-lookup"><span data-stu-id="c6d75-195">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="c6d75-196">Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.</span><span class="sxs-lookup"><span data-stu-id="c6d75-196">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="c6d75-197">**Контекст**</span><span class="sxs-lookup"><span data-stu-id="c6d75-197">**Context**</span></span> | <span data-ttu-id="c6d75-198">**Максимальная задержка<br />примера целевого E2E**</span><span class="sxs-lookup"><span data-stu-id="c6d75-198">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="c6d75-199">**Стратегия повторов**</span><span class="sxs-lookup"><span data-stu-id="c6d75-199">**Retry strategy**</span></span> | <span data-ttu-id="c6d75-200">**Параметры**</span><span class="sxs-lookup"><span data-stu-id="c6d75-200">**Settings**</span></span> | <span data-ttu-id="c6d75-201">**Значения**</span><span class="sxs-lookup"><span data-stu-id="c6d75-201">**Values**</span></span> | <span data-ttu-id="c6d75-202">**Принцип работы**</span><span class="sxs-lookup"><span data-stu-id="c6d75-202">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="c6d75-203">Интерактивный, пользовательский интерфейс</span><span class="sxs-lookup"><span data-stu-id="c6d75-203">Interactive, UI,</span></span><br /><span data-ttu-id="c6d75-204">или передний план</span><span class="sxs-lookup"><span data-stu-id="c6d75-204">or foreground</span></span> |<span data-ttu-id="c6d75-205">2 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-205">2 sec</span></span> |<span data-ttu-id="c6d75-206">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="c6d75-206">FixedInterval</span></span> |<span data-ttu-id="c6d75-207">Число повторных попыток</span><span class="sxs-lookup"><span data-stu-id="c6d75-207">Retry count</span></span><br /><span data-ttu-id="c6d75-208">Интервал попытки</span><span class="sxs-lookup"><span data-stu-id="c6d75-208">Retry interval</span></span><br /><span data-ttu-id="c6d75-209">Первый быстрый повтор</span><span class="sxs-lookup"><span data-stu-id="c6d75-209">First fast retry</span></span> |<span data-ttu-id="c6d75-210">3</span><span class="sxs-lookup"><span data-stu-id="c6d75-210">3</span></span><br /><span data-ttu-id="c6d75-211">500 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-211">500 ms</span></span><br /><span data-ttu-id="c6d75-212">true</span><span class="sxs-lookup"><span data-stu-id="c6d75-212">true</span></span> |<span data-ttu-id="c6d75-213">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-213">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="c6d75-214">Попытка 2 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-214">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="c6d75-215">попытка 3 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-215">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="c6d75-216">Фоновый</span><span class="sxs-lookup"><span data-stu-id="c6d75-216">Background or</span></span><br /><span data-ttu-id="c6d75-217"> или пакетный</span><span class="sxs-lookup"><span data-stu-id="c6d75-217">batch</span></span> |<span data-ttu-id="c6d75-218">60 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-218">60 sec</span></span> |<span data-ttu-id="c6d75-219">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="c6d75-219">ExponentialBackoff</span></span> |<span data-ttu-id="c6d75-220">Число повторных попыток</span><span class="sxs-lookup"><span data-stu-id="c6d75-220">Retry count</span></span><br /><span data-ttu-id="c6d75-221">Минимальная задержка</span><span class="sxs-lookup"><span data-stu-id="c6d75-221">Min back-off</span></span><br /><span data-ttu-id="c6d75-222">Максимальная задержка</span><span class="sxs-lookup"><span data-stu-id="c6d75-222">Max back-off</span></span><br /><span data-ttu-id="c6d75-223">Разностная задержка</span><span class="sxs-lookup"><span data-stu-id="c6d75-223">Delta back-off</span></span><br /><span data-ttu-id="c6d75-224">Первый быстрый повтор</span><span class="sxs-lookup"><span data-stu-id="c6d75-224">First fast retry</span></span> |<span data-ttu-id="c6d75-225">5</span><span class="sxs-lookup"><span data-stu-id="c6d75-225">5</span></span><br /><span data-ttu-id="c6d75-226">0 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-226">0 sec</span></span><br /><span data-ttu-id="c6d75-227">60 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-227">60 sec</span></span><br /><span data-ttu-id="c6d75-228">2 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-228">2 sec</span></span><br /><span data-ttu-id="c6d75-229">false</span><span class="sxs-lookup"><span data-stu-id="c6d75-229">false</span></span> |<span data-ttu-id="c6d75-230">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-230">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="c6d75-231">Попытка 2 — задержка 2 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-231">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="c6d75-232">Попытка 3 — задержка 6 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-232">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="c6d75-233">Попытка 4 — задержка 14 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-233">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="c6d75-234">Попытка 5 — задержка 30 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-234">Attempt 5 - delay ~30 sec</span></span> |

### <a name="more-information"></a><span data-ttu-id="c6d75-235">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="c6d75-235">More information</span></span>

- <span data-ttu-id="c6d75-236">[Библиотеки аутентификации Azure Active Directory][adal]</span><span class="sxs-lookup"><span data-stu-id="c6d75-236">[Azure Active Directory Authentication Libraries][adal]</span></span>

## <a name="cosmos-db"></a><span data-ttu-id="c6d75-237">База данных Cosmos</span><span class="sxs-lookup"><span data-stu-id="c6d75-237">Cosmos DB</span></span>

<span data-ttu-id="c6d75-238">Cosmos DB — это полностью управляемая база данных, которая поддерживает несколько моделей и данные JSON без схемы данных.</span><span class="sxs-lookup"><span data-stu-id="c6d75-238">Cosmos DB is a fully-managed multi-model database that supports schema-less JSON data.</span></span> <span data-ttu-id="c6d75-239">Она обеспечивает настраиваемую и надежную производительность, собственную обработку транзакций на основе JavaScript и разработана для облачной службы с гибким масштабированием.</span><span class="sxs-lookup"><span data-stu-id="c6d75-239">It offers configurable and reliable performance, native JavaScript transactional processing, and is built for the cloud with elastic scale.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="c6d75-240">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="c6d75-240">Retry mechanism</span></span>

<span data-ttu-id="c6d75-241">Класс `DocumentClient` автоматически осуществляет новую попытку после сбоя.</span><span class="sxs-lookup"><span data-stu-id="c6d75-241">The `DocumentClient` class automatically retries failed attempts.</span></span> <span data-ttu-id="c6d75-242">Чтобы задать количество повторных попыток и максимальное время ожидания, настройте свойство [ConnectionPolicy.RetryOptions].</span><span class="sxs-lookup"><span data-stu-id="c6d75-242">To set the number of retries and the maximum wait time, configure [ConnectionPolicy.RetryOptions].</span></span> <span data-ttu-id="c6d75-243">Исключения клиента находятся за пределами действия политики повтора или не являются временными ошибками.</span><span class="sxs-lookup"><span data-stu-id="c6d75-243">Exceptions that the client raises are either beyond the retry policy or are not transient errors.</span></span>

<span data-ttu-id="c6d75-244">Если Cosmos DB выполняет для клиента регулирование, возвращается ошибка HTTP 429.</span><span class="sxs-lookup"><span data-stu-id="c6d75-244">If Cosmos DB throttles the client, it returns an HTTP 429 error.</span></span> <span data-ttu-id="c6d75-245">Проверьте код состояния в `DocumentClientException`.</span><span class="sxs-lookup"><span data-stu-id="c6d75-245">Check the status code in the `DocumentClientException`.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="c6d75-246">Настройки политики</span><span class="sxs-lookup"><span data-stu-id="c6d75-246">Policy configuration</span></span>

<span data-ttu-id="c6d75-247">В следующей таблице показаны значения по умолчанию для класса `RetryOptions`.</span><span class="sxs-lookup"><span data-stu-id="c6d75-247">The following table shows the default settings for the `RetryOptions` class.</span></span>

| <span data-ttu-id="c6d75-248">Параметр</span><span class="sxs-lookup"><span data-stu-id="c6d75-248">Setting</span></span> | <span data-ttu-id="c6d75-249">Значение по умолчанию</span><span class="sxs-lookup"><span data-stu-id="c6d75-249">Default value</span></span> | <span data-ttu-id="c6d75-250">ОПИСАНИЕ</span><span class="sxs-lookup"><span data-stu-id="c6d75-250">Description</span></span> |
| --- | --- | --- |
| <span data-ttu-id="c6d75-251">MaxRetryAttemptsOnThrottledRequests</span><span class="sxs-lookup"><span data-stu-id="c6d75-251">MaxRetryAttemptsOnThrottledRequests</span></span> |<span data-ttu-id="c6d75-252">9</span><span class="sxs-lookup"><span data-stu-id="c6d75-252">9</span></span> |<span data-ttu-id="c6d75-253">Максимальное число повторных попыток, когда запрос завершается сбоем из-за ограничения частоты запросов для клиента в Cosmos DB.</span><span class="sxs-lookup"><span data-stu-id="c6d75-253">The maximum number of retries if the request fails because Cosmos DB applied rate limiting on the client.</span></span> |
| <span data-ttu-id="c6d75-254">MaxRetryWaitTimeInSeconds</span><span class="sxs-lookup"><span data-stu-id="c6d75-254">MaxRetryWaitTimeInSeconds</span></span> |<span data-ttu-id="c6d75-255">30</span><span class="sxs-lookup"><span data-stu-id="c6d75-255">30</span></span> |<span data-ttu-id="c6d75-256">Максимальное время повтора в секундах.</span><span class="sxs-lookup"><span data-stu-id="c6d75-256">The maximum retry time in seconds.</span></span> |

### <a name="example"></a><span data-ttu-id="c6d75-257">Пример</span><span class="sxs-lookup"><span data-stu-id="c6d75-257">Example</span></span>

```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a><span data-ttu-id="c6d75-258">Телеметрия</span><span class="sxs-lookup"><span data-stu-id="c6d75-258">Telemetry</span></span>

<span data-ttu-id="c6d75-259">Повторные попытки регистрируются как неструктурированные сообщения трассировки в .NET **TraceSource**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-259">Retry attempts are logged as unstructured trace messages through a .NET **TraceSource**.</span></span> <span data-ttu-id="c6d75-260">Необходимо настроить **TraceListener** для сбора данных о событиях и записи этих данных в журнал с подходящим назначением.</span><span class="sxs-lookup"><span data-stu-id="c6d75-260">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span>

<span data-ttu-id="c6d75-261">Например, если добавить в файл App.config следующий фрагмент кода, трассировка будет создаваться в текстовом файле в том же расположении, что и исполняемый файл:</span><span class="sxs-lookup"><span data-stu-id="c6d75-261">For example, if you add the following to your App.config file, traces will be generated in a text file in the same location as the executable:</span></span>

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

## <a name="event-hubs"></a><span data-ttu-id="c6d75-262">Центры событий;</span><span class="sxs-lookup"><span data-stu-id="c6d75-262">Event Hubs</span></span>

<span data-ttu-id="c6d75-263">Центры событий Azure — это служба обработки данных телеметрии с высокой степенью масштабируемости. Она собирает, преобразовывает и хранит миллионы событий.</span><span class="sxs-lookup"><span data-stu-id="c6d75-263">Azure Event Hubs is a hyper-scale telemetry ingestion service that collects, transforms, and stores millions of events.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="c6d75-264">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="c6d75-264">Retry mechanism</span></span>

<span data-ttu-id="c6d75-265">Механизм повторных попыток в клиентской библиотеке для Центров событий Azure управляется свойством `RetryPolicy` в классе `EventHubClient`.</span><span class="sxs-lookup"><span data-stu-id="c6d75-265">Retry behavior in the Azure Event Hubs Client Library is controlled by the `RetryPolicy` property on the `EventHubClient` class.</span></span> <span data-ttu-id="c6d75-266">По умолчанию, если концентратор Azure возвращает временную ошибку `EventHubsException` или `OperationCanceledException`, используется политика повторных попыток с экспоненциальным увеличением задержки.</span><span class="sxs-lookup"><span data-stu-id="c6d75-266">The default policy retries with exponential backoff when Azure Event Hub returns a transient `EventHubsException` or an `OperationCanceledException`.</span></span>

### <a name="example"></a><span data-ttu-id="c6d75-267">Пример</span><span class="sxs-lookup"><span data-stu-id="c6d75-267">Example</span></span>

```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a><span data-ttu-id="c6d75-268">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="c6d75-268">More information</span></span>

[<span data-ttu-id="c6d75-269">Клиентская библиотека .NET Standard для Центров событий Azure</span><span class="sxs-lookup"><span data-stu-id="c6d75-269">.NET Standard client library for Azure Event Hubs</span></span>](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="iot-hub"></a><span data-ttu-id="c6d75-270">Центр Интернета вещей</span><span class="sxs-lookup"><span data-stu-id="c6d75-270">IoT Hub</span></span>

<span data-ttu-id="c6d75-271">Центр Интернета вещей Azure — это служба подключения, мониторинга и администрирования устройств для разработки приложений Интернета вещей (IoT).</span><span class="sxs-lookup"><span data-stu-id="c6d75-271">Azure IoT Hub is a service for connecting, monitoring, and managing devices to develop Internet of Things (IoT) applications.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="c6d75-272">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="c6d75-272">Retry mechanism</span></span>

<span data-ttu-id="c6d75-273">С помощью пакета SDK для устройств Azure IoT можно обнаруживать ошибки в сети, протоколе и приложениях.</span><span class="sxs-lookup"><span data-stu-id="c6d75-273">The Azure IoT device SDK can detect errors in the network, protocol, or application.</span></span> <span data-ttu-id="c6d75-274">В зависимости от типа ошибки пакет SDK проверяет необходимость выполнения повторной попытки.</span><span class="sxs-lookup"><span data-stu-id="c6d75-274">Based on the error type, the SDK checks whether a retry needs to be performed.</span></span> <span data-ttu-id="c6d75-275">Если ошибка является *устранимой*, пакет SDK выполняет повторные попытки, используя настроенную политику повтора.</span><span class="sxs-lookup"><span data-stu-id="c6d75-275">If the error is *recoverable*, the SDK begins to retry using the configured retry policy.</span></span>

<span data-ttu-id="c6d75-276">Политика повтора по умолчанию использует *стратегию экспоненциальной отсрочки со случайным дрожанием*, но эту конфигурацию можно изменить.</span><span class="sxs-lookup"><span data-stu-id="c6d75-276">The default retry policy is *exponential back-off with random jitter*, but it can be configured.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="c6d75-277">Настройки политики</span><span class="sxs-lookup"><span data-stu-id="c6d75-277">Policy configuration</span></span>

<span data-ttu-id="c6d75-278">Конфигурации политики отличаются для разных языков.</span><span class="sxs-lookup"><span data-stu-id="c6d75-278">Policy configuration differs by language.</span></span> <span data-ttu-id="c6d75-279">См. дополнительные сведения см. об [конфигурации политики повтора Центра Интернета вещей](/azure/iot-hub/iot-hub-reliability-features-in-sdks#retry-policy-apis).</span><span class="sxs-lookup"><span data-stu-id="c6d75-279">For more details, see [IoT Hub retry policy configuration](/azure/iot-hub/iot-hub-reliability-features-in-sdks#retry-policy-apis).</span></span>

### <a name="more-information"></a><span data-ttu-id="c6d75-280">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="c6d75-280">More information</span></span>

- [<span data-ttu-id="c6d75-281">Политика повтора Центра Интернета вещей</span><span class="sxs-lookup"><span data-stu-id="c6d75-281">IoT Hub retry policy</span></span>](/azure/iot-hub/iot-hub-reliability-features-in-sdks)
- [<span data-ttu-id="c6d75-282">Устранение неполадок с отключением устройства Центра Интернета вещей</span><span class="sxs-lookup"><span data-stu-id="c6d75-282">Troubleshoot IoT Hub device disconnection</span></span>](/azure/iot-hub/iot-hub-troubleshoot-connectivity)

## <a name="azure-redis-cache"></a><span data-ttu-id="c6d75-283">кэш Azure Redis</span><span class="sxs-lookup"><span data-stu-id="c6d75-283">Azure Redis Cache</span></span>

<span data-ttu-id="c6d75-284">Кэш Redis для Azure является средством быстрого доступа к данным и службой кэша с низкой задержкой, созданной на основе кэша с открытым исходным кодом Redis.</span><span class="sxs-lookup"><span data-stu-id="c6d75-284">Azure Redis Cache is a fast data access and low latency cache service based on the popular open source Redis Cache.</span></span> <span data-ttu-id="c6d75-285">Он является безопасным, управляется Майкрософт и доступен из любого приложения в Azure.</span><span class="sxs-lookup"><span data-stu-id="c6d75-285">It is secure, managed by Microsoft, and is accessible from any application in Azure.</span></span>

<span data-ttu-id="c6d75-286">Рекомендации в этом разделе основаны на использовании клиента StackExchange.Redis для доступа к кэшу.</span><span class="sxs-lookup"><span data-stu-id="c6d75-286">The guidance in this section is based on using the StackExchange.Redis client to access the cache.</span></span> <span data-ttu-id="c6d75-287">Список других подходящих клиентов можно найти на [веб-сайте Redis](https://redis.io/clients), и они могут применять различные механизмы.</span><span class="sxs-lookup"><span data-stu-id="c6d75-287">A list of other suitable clients can be found on the [Redis website](https://redis.io/clients), and these may have different retry mechanisms.</span></span>

<span data-ttu-id="c6d75-288">Обратите внимание, что клиент StackExchange.Redis использует мультиплексирование через одно подключение.</span><span class="sxs-lookup"><span data-stu-id="c6d75-288">Note that the StackExchange.Redis client uses multiplexing through a single connection.</span></span> <span data-ttu-id="c6d75-289">Рекомендуется создавать экземпляр клиента при запуске приложения и использовать этот экземпляр для всех операций в кэше.</span><span class="sxs-lookup"><span data-stu-id="c6d75-289">The recommended usage is to create an instance of the client at application startup and use this instance for all operations against the cache.</span></span> <span data-ttu-id="c6d75-290">По этой причине подключение к кэшу происходит только один раз, и поэтому все рекомендации в этом разделе относится к политике повтора для этого исходного соединения &mdash; , а не для каждой операции, которая обращается к кэшу.</span><span class="sxs-lookup"><span data-stu-id="c6d75-290">For this reason, the connection to the cache is made only once, and so all of the guidance in this section is related to the retry policy for this initial connection &mdash; and not for each operation that accesses the cache.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="c6d75-291">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="c6d75-291">Retry mechanism</span></span>

<span data-ttu-id="c6d75-292">Клиент StackExchange.Redis использует класс диспетчера подключений, для настройки которого доступен ряд параметров, в том числе:</span><span class="sxs-lookup"><span data-stu-id="c6d75-292">The StackExchange.Redis client uses a connection manager class that is configured through a set of options, including:</span></span>

- <span data-ttu-id="c6d75-293">**ConnectRetry**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-293">**ConnectRetry**.</span></span> <span data-ttu-id="c6d75-294">Число повторных попыток при сбое подключения к кэшу.</span><span class="sxs-lookup"><span data-stu-id="c6d75-294">The number of times a failed connection to the cache will be retried.</span></span>
- <span data-ttu-id="c6d75-295">**ReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-295">**ReconnectRetryPolicy**.</span></span> <span data-ttu-id="c6d75-296">Используемая стратегия повторных попыток.</span><span class="sxs-lookup"><span data-stu-id="c6d75-296">The retry strategy to use.</span></span>
- <span data-ttu-id="c6d75-297">**ConnectTimeout**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-297">**ConnectTimeout**.</span></span> <span data-ttu-id="c6d75-298">Максимальное время ожидания в миллисекундах.</span><span class="sxs-lookup"><span data-stu-id="c6d75-298">The maximum waiting time in milliseconds.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="c6d75-299">Настройки политики</span><span class="sxs-lookup"><span data-stu-id="c6d75-299">Policy configuration</span></span>

<span data-ttu-id="c6d75-300">Политики повтора настраиваются программным способом с помощью параметров клиента перед подключением к кэшу.</span><span class="sxs-lookup"><span data-stu-id="c6d75-300">Retry policies are configured programmatically by setting the options for the client before connecting to the cache.</span></span> <span data-ttu-id="c6d75-301">Для этого нужно создать экземпляр класса **ConfigurationOptions**, заполнить его свойства и передать ему метод **Connect**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-301">This can be done by creating an instance of the **ConfigurationOptions** class, populating its properties, and passing it to the **Connect** method.</span></span>

<span data-ttu-id="c6d75-302">Встроенные классы поддерживают линейные (постоянные) паузы и экспоненциальное увеличение задержки с псевдослучайными интервалами.</span><span class="sxs-lookup"><span data-stu-id="c6d75-302">The built-in classes support linear (constant) delay and exponential backoff with randomized retry intervals.</span></span> <span data-ttu-id="c6d75-303">Также вы можете создать пользовательскую политику повторных попыток, реализовав интерфейс **IReconnectRetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-303">You can also create a custom retry policy by implementing the **IReconnectRetryPolicy** interface.</span></span>

<span data-ttu-id="c6d75-304">Код следующего примера настраивает стратегию повторных попыток с экспоненциальным увеличением задержки.</span><span class="sxs-lookup"><span data-stu-id="c6d75-304">The following example configures a retry strategy using exponential backoff.</span></span>

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

<span data-ttu-id="c6d75-305">Кроме того, можно указать параметры в виде строки и передать эти данные методу **Connect** .</span><span class="sxs-lookup"><span data-stu-id="c6d75-305">Alternatively, you can specify the options as a string, and pass this to the **Connect** method.</span></span> <span data-ttu-id="c6d75-306">Обратите внимание, что свойство **ReconnectRetryPolicy** нельзя задать таким способом, а можно только с помощью кода.</span><span class="sxs-lookup"><span data-stu-id="c6d75-306">Note that the **ReconnectRetryPolicy** property cannot be set this way, only through code.</span></span>

```csharp
var options = "localhost,connectRetry=3,connectTimeout=2000";
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="c6d75-307">Также вы можете указать параметры непосредственно при подключении к кэшу.</span><span class="sxs-lookup"><span data-stu-id="c6d75-307">You can also specify options directly when you connect to the cache.</span></span>

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

<span data-ttu-id="c6d75-308">Дополнительные сведения см. в [документации по StackExchange.Redis](https://stackexchange.github.io/StackExchange.Redis/Configuration).</span><span class="sxs-lookup"><span data-stu-id="c6d75-308">For more information, see [Stack Exchange Redis Configuration](https://stackexchange.github.io/StackExchange.Redis/Configuration) in the StackExchange.Redis documentation.</span></span>

<span data-ttu-id="c6d75-309">Следующая таблица показывает значения по умолчанию для встроенной политики повторов.</span><span class="sxs-lookup"><span data-stu-id="c6d75-309">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="c6d75-310">**Контекст**</span><span class="sxs-lookup"><span data-stu-id="c6d75-310">**Context**</span></span> | <span data-ttu-id="c6d75-311">**Параметр**</span><span class="sxs-lookup"><span data-stu-id="c6d75-311">**Setting**</span></span> | <span data-ttu-id="c6d75-312">**Значение по умолчанию**</span><span class="sxs-lookup"><span data-stu-id="c6d75-312">**Default value**</span></span><br /><span data-ttu-id="c6d75-313">(версия 1.2.2)</span><span class="sxs-lookup"><span data-stu-id="c6d75-313">(v 1.2.2)</span></span> | <span data-ttu-id="c6d75-314">**Значение**</span><span class="sxs-lookup"><span data-stu-id="c6d75-314">**Meaning**</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="c6d75-315">Параметры конфигурации</span><span class="sxs-lookup"><span data-stu-id="c6d75-315">ConfigurationOptions</span></span> |<span data-ttu-id="c6d75-316">ConnectRetry</span><span class="sxs-lookup"><span data-stu-id="c6d75-316">ConnectRetry</span></span><br /><br /><span data-ttu-id="c6d75-317">ConnectTimeout</span><span class="sxs-lookup"><span data-stu-id="c6d75-317">ConnectTimeout</span></span><br /><br /><span data-ttu-id="c6d75-318">SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="c6d75-318">SyncTimeout</span></span><br /><br /><span data-ttu-id="c6d75-319">ReconnectRetryPolicy</span><span class="sxs-lookup"><span data-stu-id="c6d75-319">ReconnectRetryPolicy</span></span> |<span data-ttu-id="c6d75-320">3</span><span class="sxs-lookup"><span data-stu-id="c6d75-320">3</span></span><br /><br /><span data-ttu-id="c6d75-321">Максимально 5000 мс плюс SyncTimeout</span><span class="sxs-lookup"><span data-stu-id="c6d75-321">Maximum 5000 ms plus SyncTimeout</span></span><br /><span data-ttu-id="c6d75-322">1000</span><span class="sxs-lookup"><span data-stu-id="c6d75-322">1000</span></span><br /><br /><span data-ttu-id="c6d75-323">LinearRetry 5000 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-323">LinearRetry 5000 ms</span></span> |<span data-ttu-id="c6d75-324">Количество попыток повторов подключения во время начального подключения.</span><span class="sxs-lookup"><span data-stu-id="c6d75-324">The number of times to repeat connect attempts during the initial connection operation.</span></span><br /><span data-ttu-id="c6d75-325">Время ожидания (мс) для операций подключения.</span><span class="sxs-lookup"><span data-stu-id="c6d75-325">Timeout (ms) for connect operations.</span></span> <span data-ttu-id="c6d75-326">Без задержки между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="c6d75-326">Not a delay between retry attempts.</span></span><br /><span data-ttu-id="c6d75-327">Время (мс) для синхронных операций.</span><span class="sxs-lookup"><span data-stu-id="c6d75-327">Time (ms) to allow for synchronous operations.</span></span><br /><br /><span data-ttu-id="c6d75-328">Повторы через каждые 5000 мс.</span><span class="sxs-lookup"><span data-stu-id="c6d75-328">Retry every 5000 ms.</span></span>|

> [!NOTE]
> <span data-ttu-id="c6d75-329">Для синхронных операций можно добавить `SyncTimeout` для определения совокупного времени задержки, но слишком низкое значение этого параметра приведет к чрезмерному времени ожидания.</span><span class="sxs-lookup"><span data-stu-id="c6d75-329">For synchronous operations, `SyncTimeout` can add to the end-to-end latency, but setting the value too low can cause excessive timeouts.</span></span> <span data-ttu-id="c6d75-330">См. статью [Способы устранения проблем с кэшем Redis для Azure][redis-cache-troubleshoot].</span><span class="sxs-lookup"><span data-stu-id="c6d75-330">See [How to troubleshoot Azure Redis Cache][redis-cache-troubleshoot].</span></span> <span data-ttu-id="c6d75-331">Обычно следует избегать синхронных операций и использовать асинхронные везде, где это возможно.</span><span class="sxs-lookup"><span data-stu-id="c6d75-331">In general, avoid using synchronous operations, and use asynchronous operations instead.</span></span> <span data-ttu-id="c6d75-332">Дополнительные сведения см. в статье [Конвейеры и мультиплексоры](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/PipelinesMultiplexers.md).</span><span class="sxs-lookup"><span data-stu-id="c6d75-332">For more information see [Pipelines and Multiplexers](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/PipelinesMultiplexers.md).</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="c6d75-333">Руководство по использованию механизма повторов</span><span class="sxs-lookup"><span data-stu-id="c6d75-333">Retry usage guidance</span></span>

<span data-ttu-id="c6d75-334">При использовании кэша Redis для Azure, придерживайтесь следующих рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="c6d75-334">Consider the following guidelines when using Azure Redis Cache:</span></span>

- <span data-ttu-id="c6d75-335">Клиент StackExchange Redis управляет собственными операциями повтора, но только при установлении соединения с кэшем при первом запуске приложения.</span><span class="sxs-lookup"><span data-stu-id="c6d75-335">The StackExchange Redis client manages its own retries, but only when establishing a connection to the cache when the application first starts.</span></span> <span data-ttu-id="c6d75-336">Вы можете настроить время ожидания подключения, число повторных попыток подключения и паузы между ними, но эта политика повторов не применяется к операциям с кэшем.</span><span class="sxs-lookup"><span data-stu-id="c6d75-336">You can configure the connection timeout, the number of retry attempts, and the time between retries to establish this connection, but the retry policy does not apply to operations against the cache.</span></span>
- <span data-ttu-id="c6d75-337">Вместо использования большого числа повторных попыток рассмотрите возможность возврата к исходному источнику данных.</span><span class="sxs-lookup"><span data-stu-id="c6d75-337">Instead of using a large number of retry attempts, consider falling back by accessing the original data source instead.</span></span>

### <a name="telemetry"></a><span data-ttu-id="c6d75-338">Телеметрия</span><span class="sxs-lookup"><span data-stu-id="c6d75-338">Telemetry</span></span>

<span data-ttu-id="c6d75-339">Можно собирать сведения о подключении (но не о других операциях) с помощью **TextWriter**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-339">You can collect information about connections (but not other operations) using a **TextWriter**.</span></span>

```csharp
var writer = new StringWriter();
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

<span data-ttu-id="c6d75-340">Ниже показан пример генерируемых выходных данных.</span><span class="sxs-lookup"><span data-stu-id="c6d75-340">An example of the output this generates is shown below.</span></span>

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

### <a name="examples"></a><span data-ttu-id="c6d75-341">Примеры</span><span class="sxs-lookup"><span data-stu-id="c6d75-341">Examples</span></span>

<span data-ttu-id="c6d75-342">Следующий пример кода настраивает постоянную (линейную) задержку между повторными попытками при инициализации клиента StackExchange.Redis.</span><span class="sxs-lookup"><span data-stu-id="c6d75-342">The following code example configures a constant (linear) delay between retries when initializing the StackExchange.Redis client.</span></span> <span data-ttu-id="c6d75-343">В этом примере показано, как настроить конфигурацию с помощью экземпляра **ConfigurationOptions**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-343">This example shows how to set the configuration using a **ConfigurationOptions** instance.</span></span>

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

<span data-ttu-id="c6d75-344">В следующем примере показано, как настроить конфигурацию, указав параметры в строковом формате.</span><span class="sxs-lookup"><span data-stu-id="c6d75-344">The next example sets the configuration by specifying the options as a string.</span></span> <span data-ttu-id="c6d75-345">Время ожидания подключения обозначает максимальный период времени для ожидания соединения с кэшем, но не задержку между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="c6d75-345">The connection timeout is the maximum period of time to wait for a connection to the cache, not the delay between retry attempts.</span></span> <span data-ttu-id="c6d75-346">Обратите внимание, что свойство **ReconnectRetryPolicy** можно задавать только в коде.</span><span class="sxs-lookup"><span data-stu-id="c6d75-346">Note that the **ReconnectRetryPolicy** property can only be set by code.</span></span>

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

<span data-ttu-id="c6d75-347">Дополнительные примеры см. в разделе [Конфигурация](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/Configuration.md) на веб-сайте проекта.</span><span class="sxs-lookup"><span data-stu-id="c6d75-347">For more examples, see [Configuration](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/Configuration.md) on the project website.</span></span>

### <a name="more-information"></a><span data-ttu-id="c6d75-348">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="c6d75-348">More information</span></span>

- [<span data-ttu-id="c6d75-349">Веб-сайт Redis</span><span class="sxs-lookup"><span data-stu-id="c6d75-349">Redis website</span></span>](https://redis.io/)

## <a name="azure-search"></a><span data-ttu-id="c6d75-350">Поиск Azure</span><span class="sxs-lookup"><span data-stu-id="c6d75-350">Azure Search</span></span>

<span data-ttu-id="c6d75-351">Поиск Azure является мощным и сложным инструментом поиска для сайта или приложения с возможностями быстрой и простой настройки результатов поиска и построения информативных и точных моделей ранжирования.</span><span class="sxs-lookup"><span data-stu-id="c6d75-351">Azure Search can be used to add powerful and sophisticated search capabilities to a website or application, quickly and easily tune search results, and construct rich and fine-tuned ranking models.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="c6d75-352">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="c6d75-352">Retry mechanism</span></span>

<span data-ttu-id="c6d75-353">Поведение повтора в пакете SDK для Поиска Azure контролируется в методе `SetRetryPolicy` класса [SearchServiceClient] и [SearchIndexClient].</span><span class="sxs-lookup"><span data-stu-id="c6d75-353">Retry behavior in the Azure Search SDK is controlled by the `SetRetryPolicy` method on the [SearchServiceClient] and [SearchIndexClient] classes.</span></span> <span data-ttu-id="c6d75-354">Повтор политики по умолчанию осуществляется с экспоненциальной задержкой, если Поиск Azure возвращает ответ с кодом 5xx или 408 (истекло время ожидания запроса).</span><span class="sxs-lookup"><span data-stu-id="c6d75-354">The default policy retries with exponential backoff when Azure Search returns a 5xx or 408 (Request Timeout) response.</span></span>

### <a name="telemetry"></a><span data-ttu-id="c6d75-355">Телеметрия</span><span class="sxs-lookup"><span data-stu-id="c6d75-355">Telemetry</span></span>

<span data-ttu-id="c6d75-356">Используйте трассировку событий Windows или осуществляйте трассировку путем регистрации пользовательского поставщика трассировки.</span><span class="sxs-lookup"><span data-stu-id="c6d75-356">Trace with ETW or by registering a custom trace provider.</span></span> <span data-ttu-id="c6d75-357">Дополнительные сведения см. в [документации по AutoRest][autorest].</span><span class="sxs-lookup"><span data-stu-id="c6d75-357">For more information, see the [AutoRest documentation][autorest].</span></span>

## <a name="service-bus"></a><span data-ttu-id="c6d75-358">Служебная шина Azure</span><span class="sxs-lookup"><span data-stu-id="c6d75-358">Service Bus</span></span>

<span data-ttu-id="c6d75-359">Служебная шина является облачной платформой обмена сообщениями, которая предоставляет слабосвязанный обмен сообщениями с улучшенным масштабированием и устойчивостью для компонентов приложения, размещенных в облаке или локально.</span><span class="sxs-lookup"><span data-stu-id="c6d75-359">Service Bus is a cloud messaging platform that provides loosely coupled message exchange with improved scale and resiliency for components of an application, whether hosted in the cloud or on-premises.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="c6d75-360">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="c6d75-360">Retry mechanism</span></span>

<span data-ttu-id="c6d75-361">Служебная шина реализует повторы с помощью реализации базового класса [RetryPolicy](/dotnet/api/microsoft.servicebus.retrypolicy) .</span><span class="sxs-lookup"><span data-stu-id="c6d75-361">Service Bus implements retries using implementations of the [RetryPolicy](/dotnet/api/microsoft.servicebus.retrypolicy) base class.</span></span> <span data-ttu-id="c6d75-362">Все клиенты служебной шины предоставляют свойство **RetryPolicy**, которое может быть присвоено одной из реализаций базового класса **RetryPolicy**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-362">All of the Service Bus clients expose a **RetryPolicy** property that can be set to one of the implementations of the **RetryPolicy** base class.</span></span> <span data-ttu-id="c6d75-363">Встроенные реализации</span><span class="sxs-lookup"><span data-stu-id="c6d75-363">The built-in implementations are:</span></span>

- <span data-ttu-id="c6d75-364">[Класс RetryExponential](/dotnet/api/microsoft.servicebus.retryexponential).</span><span class="sxs-lookup"><span data-stu-id="c6d75-364">The [RetryExponential class](/dotnet/api/microsoft.servicebus.retryexponential).</span></span> <span data-ttu-id="c6d75-365">Этот класс предоставляет свойства, управляющие интервалом отсрочки, числом попыток и свойством **TerminationTimeBuffer** , используемым для ограничения общего времени завершения операции.</span><span class="sxs-lookup"><span data-stu-id="c6d75-365">This exposes properties that control the back-off interval, the retry count, and the **TerminationTimeBuffer** property that is used to limit the total time for the operation to complete.</span></span>

- <span data-ttu-id="c6d75-366">[Класс NoRetry](/dotnet/api/microsoft.servicebus.noretry).</span><span class="sxs-lookup"><span data-stu-id="c6d75-366">The [NoRetry class](/dotnet/api/microsoft.servicebus.noretry).</span></span> <span data-ttu-id="c6d75-367">Используется, когда повторы на уровне API служебной шины не требуются, например, если повторные попытки управляются другим процессом в рамках пакета операций либо многоэтапной операции.</span><span class="sxs-lookup"><span data-stu-id="c6d75-367">This is used when retries at the Service Bus API level are not required, such as when retries are managed by another process as part of a batch or multiple step operation.</span></span>

<span data-ttu-id="c6d75-368">Действия служебной шины могут вызвать ряд исключений, как указано в статье [Исключения обмена сообщениями служебной шины](/azure/service-bus-messaging/service-bus-messaging-exceptions).</span><span class="sxs-lookup"><span data-stu-id="c6d75-368">Service Bus actions can return a range of exceptions, as listed in [Service Bus messaging exceptions](/azure/service-bus-messaging/service-bus-messaging-exceptions).</span></span> <span data-ttu-id="c6d75-369">Список предоставляет сведения о том, какие из исключений указывают, что повторение операции является целесообразным.</span><span class="sxs-lookup"><span data-stu-id="c6d75-369">The list provides information about which if these indicate that retrying the operation is appropriate.</span></span> <span data-ttu-id="c6d75-370">Например **ServerBusyException** указывает, что клиенту следует подождать некоторое время и затем повторить операцию.</span><span class="sxs-lookup"><span data-stu-id="c6d75-370">For example, a **ServerBusyException** indicates that the client should wait for a period of time, then retry the operation.</span></span> <span data-ttu-id="c6d75-371">Появление исключения **ServerBusyException** также вынуждает служебную шину переключиться в другой режим, в котором добавляется дополнительная 10-секундная задержка к вычисляемому времени задержки повтора операции.</span><span class="sxs-lookup"><span data-stu-id="c6d75-371">The occurrence of a **ServerBusyException** also causes Service Bus to switch to a different mode, in which an extra 10-second delay is added to the computed retry delays.</span></span> <span data-ttu-id="c6d75-372">Этот режим сбрасывается после короткого времени.</span><span class="sxs-lookup"><span data-stu-id="c6d75-372">This mode is reset after a short period.</span></span>

<span data-ttu-id="c6d75-373">Исключения, возвращаемые служебной шиной, предоставляют свойство **IsTransient** , указывающее, что клиент должен повторить операцию.</span><span class="sxs-lookup"><span data-stu-id="c6d75-373">The exceptions returned from Service Bus expose the **IsTransient** property that indicates if the client should retry the operation.</span></span> <span data-ttu-id="c6d75-374">Встроенная политика **RetryExponential** зависит от свойства **IsTransient** класса **MessagingException**, который является базовым классом для всех исключений служебной шины.</span><span class="sxs-lookup"><span data-stu-id="c6d75-374">The built-in **RetryExponential** policy relies on the **IsTransient** property in the **MessagingException** class, which is the base class for all Service Bus exceptions.</span></span> <span data-ttu-id="c6d75-375">Если создать пользовательские реализации базового класса **RetryPolicy**, то можно будет использовать сочетание типа исключения и свойство **IsTransient**, чтобы организовать более точное управление повторами.</span><span class="sxs-lookup"><span data-stu-id="c6d75-375">If you create custom implementations of the **RetryPolicy** base class you could use a combination of the exception type and the **IsTransient** property to provide more fine-grained control over retry actions.</span></span> <span data-ttu-id="c6d75-376">Например, можно обнаружить исключение **QuotaExceededException** и принять меры по очистке очереди, прежде чем повторить отправку сообщения в очередь.</span><span class="sxs-lookup"><span data-stu-id="c6d75-376">For example, you could detect a **QuotaExceededException** and take action to drain the queue before retrying sending a message to it.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="c6d75-377">Настройки политики</span><span class="sxs-lookup"><span data-stu-id="c6d75-377">Policy configuration</span></span>

<span data-ttu-id="c6d75-378">Политики повтора настраиваются программно и могут быть установлены в качестве политики по умолчанию для **NamespaceManager** и **MessagingFactory** или по отдельности для каждого клиента обмена сообщениями.</span><span class="sxs-lookup"><span data-stu-id="c6d75-378">Retry policies are set programmatically, and can be set as a default policy for a **NamespaceManager** and for a **MessagingFactory**, or individually for each messaging client.</span></span> <span data-ttu-id="c6d75-379">Чтобы задать политику повтора по умолчанию для сеанса обмена сообщениями, можно установить свойство **RetryPolicy** класса **NamespaceManager**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-379">To set the default retry policy for a messaging session you set the **RetryPolicy** of the **NamespaceManager**.</span></span>

```csharp
namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                maxBackoff: TimeSpan.FromSeconds(30),
                                                                maxRetryCount: 3);
```

<span data-ttu-id="c6d75-380">Чтобы задать политику повторов по умолчанию для всех клиентов, созданных из фабрики обмена сообщениями, установите свойство **RetryPolicy** класса **MessagingFactory**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-380">To set the default retry policy for all clients created from a messaging factory, you set the **RetryPolicy** of the **MessagingFactory**.</span></span>

```csharp
messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                    maxBackoff: TimeSpan.FromSeconds(30),
                                                    maxRetryCount: 3);
```

<span data-ttu-id="c6d75-381">Чтобы задать политику повтора для клиента обмена сообщениями или переопределить политику по умолчанию, задайте свойство **RetryPolicy** с помощью экземпляра класса требуемой политики:</span><span class="sxs-lookup"><span data-stu-id="c6d75-381">To set the retry policy for a messaging client, or to override its default policy, you set its **RetryPolicy** property using an instance of the required policy class:</span></span>

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

<span data-ttu-id="c6d75-382">Нельзя задать политику повтора на уровне отдельных операций.</span><span class="sxs-lookup"><span data-stu-id="c6d75-382">The retry policy cannot be set at the individual operation level.</span></span> <span data-ttu-id="c6d75-383">Она применяется для всех операций для клиента обмена сообщениями.</span><span class="sxs-lookup"><span data-stu-id="c6d75-383">It applies to all operations for the messaging client.</span></span>
<span data-ttu-id="c6d75-384">Следующая таблица показывает значения по умолчанию для встроенной политики повторов.</span><span class="sxs-lookup"><span data-stu-id="c6d75-384">The following table shows the default settings for the built-in retry policy.</span></span>

| <span data-ttu-id="c6d75-385">Параметр</span><span class="sxs-lookup"><span data-stu-id="c6d75-385">Setting</span></span> | <span data-ttu-id="c6d75-386">Значение по умолчанию</span><span class="sxs-lookup"><span data-stu-id="c6d75-386">Default value</span></span> | <span data-ttu-id="c6d75-387">Значение</span><span class="sxs-lookup"><span data-stu-id="c6d75-387">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="c6d75-388">Политика</span><span class="sxs-lookup"><span data-stu-id="c6d75-388">Policy</span></span> | <span data-ttu-id="c6d75-389">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="c6d75-389">Exponential</span></span> | <span data-ttu-id="c6d75-390">Экспоненциальная задержка.</span><span class="sxs-lookup"><span data-stu-id="c6d75-390">Exponential back-off.</span></span> |
| <span data-ttu-id="c6d75-391">MinimalBackoff</span><span class="sxs-lookup"><span data-stu-id="c6d75-391">MinimalBackoff</span></span> | <span data-ttu-id="c6d75-392">0</span><span class="sxs-lookup"><span data-stu-id="c6d75-392">0</span></span> | <span data-ttu-id="c6d75-393">Минимальное время задержки.</span><span class="sxs-lookup"><span data-stu-id="c6d75-393">Minimum back-off interval.</span></span> <span data-ttu-id="c6d75-394">Добавляется ко всем интервалам повтора, вычисленным на основе значения deltaBackoff.</span><span class="sxs-lookup"><span data-stu-id="c6d75-394">This is added to the retry interval computed from deltaBackoff.</span></span> |
| <span data-ttu-id="c6d75-395">MaximumBackoff</span><span class="sxs-lookup"><span data-stu-id="c6d75-395">MaximumBackoff</span></span> | <span data-ttu-id="c6d75-396">30 секунд</span><span class="sxs-lookup"><span data-stu-id="c6d75-396">30 seconds</span></span> | <span data-ttu-id="c6d75-397">Максимальное время задержки.</span><span class="sxs-lookup"><span data-stu-id="c6d75-397">Maximum back-off interval.</span></span> <span data-ttu-id="c6d75-398">MaxBackoff используется, если расчетный интервал повторных попыток превышает значение MaxBackoff.</span><span class="sxs-lookup"><span data-stu-id="c6d75-398">MaximumBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> |
| <span data-ttu-id="c6d75-399">DeltaBackoff</span><span class="sxs-lookup"><span data-stu-id="c6d75-399">DeltaBackoff</span></span> | <span data-ttu-id="c6d75-400">3 секунды</span><span class="sxs-lookup"><span data-stu-id="c6d75-400">3 seconds</span></span> | <span data-ttu-id="c6d75-401">Интервал отсрочки между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="c6d75-401">Back-off interval between retries.</span></span> <span data-ttu-id="c6d75-402">Величина, кратная timespan, будет использоваться для последующих попыток.</span><span class="sxs-lookup"><span data-stu-id="c6d75-402">Multiples of this timespan will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="c6d75-403">TimeBuffer</span><span class="sxs-lookup"><span data-stu-id="c6d75-403">TimeBuffer</span></span> | <span data-ttu-id="c6d75-404">5 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-404">5 seconds</span></span> | <span data-ttu-id="c6d75-405">Буфер времени завершения, связанный с повторной попыткой.</span><span class="sxs-lookup"><span data-stu-id="c6d75-405">The termination time buffer associated with the retry.</span></span> <span data-ttu-id="c6d75-406">Повторные попытки будут прерваны, если оставшееся время меньше значения TimeBuffer.</span><span class="sxs-lookup"><span data-stu-id="c6d75-406">Retry attempts will be abandoned if the remaining time is less than TimeBuffer.</span></span> |
| <span data-ttu-id="c6d75-407">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="c6d75-407">MaxRetryCount</span></span> | <span data-ttu-id="c6d75-408">10</span><span class="sxs-lookup"><span data-stu-id="c6d75-408">10</span></span> | <span data-ttu-id="c6d75-409">Максимальное число попыток.</span><span class="sxs-lookup"><span data-stu-id="c6d75-409">The maximum number of retries.</span></span> |
| <span data-ttu-id="c6d75-410">ServerBusyBaseSleepTime</span><span class="sxs-lookup"><span data-stu-id="c6d75-410">ServerBusyBaseSleepTime</span></span> | <span data-ttu-id="c6d75-411">10 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-411">10 seconds</span></span> | <span data-ttu-id="c6d75-412">Если последнее исключение было **ServerBusyException**, это значение будет добавляться к интервалу вычисляемых значений повторных попыток.</span><span class="sxs-lookup"><span data-stu-id="c6d75-412">If the last exception encountered was **ServerBusyException**, this value will be added to the computed retry interval.</span></span> <span data-ttu-id="c6d75-413">Это значение не может быть изменено.</span><span class="sxs-lookup"><span data-stu-id="c6d75-413">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="c6d75-414">Руководство по использованию механизма повторов</span><span class="sxs-lookup"><span data-stu-id="c6d75-414">Retry usage guidance</span></span>

<span data-ttu-id="c6d75-415">При использовании служебной шины придерживайтесь следующих рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="c6d75-415">Consider the following guidelines when using Service Bus:</span></span>

- <span data-ttu-id="c6d75-416">При использовании встроенной реализации **RetryExponential** не реализуйте операции резервирования, поскольку политика реагирует на исключения типа «Сервер занят» и автоматически переключается в режим повтора.</span><span class="sxs-lookup"><span data-stu-id="c6d75-416">When using the built-in **RetryExponential** implementation, do not implement a fallback operation as the policy reacts to Server Busy exceptions and automatically switches to an appropriate retry mode.</span></span>
- <span data-ttu-id="c6d75-417">Server Bus поддерживает функцию, называемую Сопряженные пространства имен, которая реализует автоматическое переключение на очередь резервного копирования в отдельном пространстве имен, если очередь в первичном пространстве имен дает сбой.</span><span class="sxs-lookup"><span data-stu-id="c6d75-417">Service Bus supports a feature called Paired Namespaces, which implements automatic failover to a backup queue in a separate namespace if the queue in the primary namespace fails.</span></span> <span data-ttu-id="c6d75-418">Сообщения из вторичной очереди могут отправляться обратно в основную очередь после ее восстановления.</span><span class="sxs-lookup"><span data-stu-id="c6d75-418">Messages from the secondary queue can be sent back to the primary queue when it recovers.</span></span> <span data-ttu-id="c6d75-419">Эта функция позволяет справиться с временными сбоями.</span><span class="sxs-lookup"><span data-stu-id="c6d75-419">This feature helps to address transient failures.</span></span> <span data-ttu-id="c6d75-420">Для получения дополнительных сведений обратитесь к разделу [Асинхронные шаблоны обмена сообщениями и высокий уровень доступности](/azure/service-bus-messaging/service-bus-async-messaging).</span><span class="sxs-lookup"><span data-stu-id="c6d75-420">For more information, see [Asynchronous Messaging Patterns and High Availability](/azure/service-bus-messaging/service-bus-async-messaging).</span></span>

<span data-ttu-id="c6d75-421">Начните с использования следующих параметров для операции повтора.</span><span class="sxs-lookup"><span data-stu-id="c6d75-421">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="c6d75-422">Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.</span><span class="sxs-lookup"><span data-stu-id="c6d75-422">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="c6d75-423">Context</span><span class="sxs-lookup"><span data-stu-id="c6d75-423">Context</span></span> | <span data-ttu-id="c6d75-424">Пример максимального значения задержки</span><span class="sxs-lookup"><span data-stu-id="c6d75-424">Example maximum latency</span></span> | <span data-ttu-id="c6d75-425">Политика повтора</span><span class="sxs-lookup"><span data-stu-id="c6d75-425">Retry policy</span></span> | <span data-ttu-id="c6d75-426">Параметры</span><span class="sxs-lookup"><span data-stu-id="c6d75-426">Settings</span></span> | <span data-ttu-id="c6d75-427">Принцип работы</span><span class="sxs-lookup"><span data-stu-id="c6d75-427">How it works</span></span> |
|---------|---------|---------|---------|---------|
| <span data-ttu-id="c6d75-428">Интерактивный, пользовательский интерфейс или передний план</span><span class="sxs-lookup"><span data-stu-id="c6d75-428">Interactive, UI, or foreground</span></span> | <span data-ttu-id="c6d75-429">2 с\*</span><span class="sxs-lookup"><span data-stu-id="c6d75-429">2 seconds\*</span></span>  | <span data-ttu-id="c6d75-430">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="c6d75-430">Exponential</span></span> | <span data-ttu-id="c6d75-431">MinimumBackoff = 0</span><span class="sxs-lookup"><span data-stu-id="c6d75-431">MinimumBackoff = 0</span></span> <br/> <span data-ttu-id="c6d75-432">MaximumBackoff = 30 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-432">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="c6d75-433">DeltaBackoff = 300 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-433">DeltaBackoff = 300 msec.</span></span> <br/> <span data-ttu-id="c6d75-434">TimeBuffer = 300 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-434">TimeBuffer = 300 msec.</span></span> <br/> <span data-ttu-id="c6d75-435">MaxRetryCount = 2</span><span class="sxs-lookup"><span data-stu-id="c6d75-435">MaxRetryCount = 2</span></span> | <span data-ttu-id="c6d75-436">Попытка 1: задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-436">Attempt 1: Delay 0 sec.</span></span> <br/> <span data-ttu-id="c6d75-437">Попытка 2: задержка ~300 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-437">Attempt 2: Delay ~300 msec.</span></span> <br/> <span data-ttu-id="c6d75-438">Попытка 3: задержка ~900 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-438">Attempt 3: Delay ~900 msec.</span></span> |
| <span data-ttu-id="c6d75-439">Фоновый или пакетный</span><span class="sxs-lookup"><span data-stu-id="c6d75-439">Background or batch</span></span> | <span data-ttu-id="c6d75-440">30 секунд</span><span class="sxs-lookup"><span data-stu-id="c6d75-440">30 seconds</span></span> | <span data-ttu-id="c6d75-441">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="c6d75-441">Exponential</span></span> | <span data-ttu-id="c6d75-442">MinimumBackoff = 1</span><span class="sxs-lookup"><span data-stu-id="c6d75-442">MinimumBackoff = 1</span></span> <br/> <span data-ttu-id="c6d75-443">MaximumBackoff = 30 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-443">MaximumBackoff = 30 sec.</span></span> <br/> <span data-ttu-id="c6d75-444">DeltaBackoff = 1,75 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-444">DeltaBackoff = 1.75 sec.</span></span> <br/> <span data-ttu-id="c6d75-445">TimeBuffer = 5 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-445">TimeBuffer = 5 sec.</span></span> <br/> <span data-ttu-id="c6d75-446">MaxRetryCount = 3</span><span class="sxs-lookup"><span data-stu-id="c6d75-446">MaxRetryCount = 3</span></span> | <span data-ttu-id="c6d75-447">Попытка 1: задержка ~1 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-447">Attempt 1: Delay ~1 sec.</span></span> <br/> <span data-ttu-id="c6d75-448">Попытка 2: задержка ~3 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-448">Attempt 2: Delay ~3 sec.</span></span> <br/> <span data-ttu-id="c6d75-449">Попытка 3: задержка ~6 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-449">Attempt 3: Delay ~6 msec.</span></span> <br/> <span data-ttu-id="c6d75-450">Попытка 4: задержка ~13 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-450">Attempt 4: Delay ~13 msec.</span></span> |

<span data-ttu-id="c6d75-451">\* Без значения дополнительной задержки, которая добавляется при получении ответа "Сервер занят".</span><span class="sxs-lookup"><span data-stu-id="c6d75-451">\* Not including additional delay that is added if a Server Busy response is received.</span></span>

### <a name="telemetry"></a><span data-ttu-id="c6d75-452">Телеметрия</span><span class="sxs-lookup"><span data-stu-id="c6d75-452">Telemetry</span></span>

<span data-ttu-id="c6d75-453">Служебная шина регистрирует повторные попытки как события ETW с помощью **EventSource**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-453">Service Bus logs retries as ETW events using an **EventSource**.</span></span> <span data-ttu-id="c6d75-454">Необходимо присоединить службу **EventListener** к источнику события, чтобы зарегистрировать события и просмотреть в средстве просмотра производительности или записать данные о них в журнале с подходящим назначением.</span><span class="sxs-lookup"><span data-stu-id="c6d75-454">You must attach an **EventListener** to the event source to capture the events and view them in Performance Viewer, or write them to a suitable destination log.</span></span> <span data-ttu-id="c6d75-455">События повтора операций отображаются в следующей форме:</span><span class="sxs-lookup"><span data-stu-id="c6d75-455">The retry events are of the following form:</span></span>

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

### <a name="examples"></a><span data-ttu-id="c6d75-456">Примеры</span><span class="sxs-lookup"><span data-stu-id="c6d75-456">Examples</span></span>

<span data-ttu-id="c6d75-457">В следующем примере кода показано, как задать политику повтора для</span><span class="sxs-lookup"><span data-stu-id="c6d75-457">The following code example shows how to set the retry policy for:</span></span>

- <span data-ttu-id="c6d75-458">Диспетчера пространства имен.</span><span class="sxs-lookup"><span data-stu-id="c6d75-458">A namespace manager.</span></span> <span data-ttu-id="c6d75-459">Политика применяется ко всем операциям диспетчера и не может быть переопределена для отдельных операций.</span><span class="sxs-lookup"><span data-stu-id="c6d75-459">The policy applies to all operations on that manager, and cannot be overridden for individual operations.</span></span>
- <span data-ttu-id="c6d75-460">Фабрики сообщений.</span><span class="sxs-lookup"><span data-stu-id="c6d75-460">A messaging factory.</span></span> <span data-ttu-id="c6d75-461">Политика применяется ко всем клиентам, созданным из этой фабрики, и не может быть переопределена при создании отдельных клиентов.</span><span class="sxs-lookup"><span data-stu-id="c6d75-461">The policy applies to all clients created from that factory, and cannot be overridden when creating individual clients.</span></span>
- <span data-ttu-id="c6d75-462">Отдельных клиентов обмена сообщениями.</span><span class="sxs-lookup"><span data-stu-id="c6d75-462">An individual messaging client.</span></span> <span data-ttu-id="c6d75-463">После создания клиента можно задать политику повтора для этого клиента.</span><span class="sxs-lookup"><span data-stu-id="c6d75-463">After a client has been created, you can set the retry policy for that client.</span></span> <span data-ttu-id="c6d75-464">Политика применяется ко всем операциям на этом клиенте.</span><span class="sxs-lookup"><span data-stu-id="c6d75-464">The policy applies to all operations on that client.</span></span>

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

### <a name="more-information"></a><span data-ttu-id="c6d75-465">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="c6d75-465">More information</span></span>

- [<span data-ttu-id="c6d75-466">Шаблоны асинхронного обмена сообщениями и высокий уровень доступности</span><span class="sxs-lookup"><span data-stu-id="c6d75-466">Asynchronous messaging patterns and high availability</span></span>](/azure/service-bus-messaging/service-bus-async-messaging)

## <a name="service-fabric"></a><span data-ttu-id="c6d75-467">Service Fabric</span><span class="sxs-lookup"><span data-stu-id="c6d75-467">Service Fabric</span></span>

<span data-ttu-id="c6d75-468">Распространение надежных служб в кластере Service Fabric защищает от большинства потенциальных временных сбоев, которые описаны в этой статье.</span><span class="sxs-lookup"><span data-stu-id="c6d75-468">Distributing reliable services in a Service Fabric cluster guards against most of the potential transient faults discussed in this article.</span></span> <span data-ttu-id="c6d75-469">Но некоторые временные сбои все равно могут произойти.</span><span class="sxs-lookup"><span data-stu-id="c6d75-469">Some transient faults are still possible, however.</span></span> <span data-ttu-id="c6d75-470">Например, если служба именования в момент получения запроса выполняет изменение маршрутизации, она создаст исключение.</span><span class="sxs-lookup"><span data-stu-id="c6d75-470">For example, the naming service might be in the middle of a routing change when it gets a request, causing it to throw an exception.</span></span> <span data-ttu-id="c6d75-471">Если повторить этот же запрос через 100 миллисекунд, он, скорее всего, завершится успешно.</span><span class="sxs-lookup"><span data-stu-id="c6d75-471">If the same request comes 100 milliseconds later, it will probably succeed.</span></span>

<span data-ttu-id="c6d75-472">Service Fabric обрабатывает такие временные сбои на внутреннем уровне.</span><span class="sxs-lookup"><span data-stu-id="c6d75-472">Internally, Service Fabric manages this kind of transient fault.</span></span> <span data-ttu-id="c6d75-473">Некоторые параметры можно настроить с помощью класса `OperationRetrySettings` при настройке служб.</span><span class="sxs-lookup"><span data-stu-id="c6d75-473">You can configure some settings by using the `OperationRetrySettings` class while setting up your services.</span></span> <span data-ttu-id="c6d75-474">Пример такой настройки приведен в следующем коде.</span><span class="sxs-lookup"><span data-stu-id="c6d75-474">The following code shows an example.</span></span> <span data-ttu-id="c6d75-475">В большинстве случаев это не требуется, и можно использовать параметры по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="c6d75-475">In most cases, this should not be necessary, and the default settings will be fine.</span></span>

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

### <a name="more-information"></a><span data-ttu-id="c6d75-476">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="c6d75-476">More information</span></span>

- [<span data-ttu-id="c6d75-477">Удаленная обработка исключений</span><span class="sxs-lookup"><span data-stu-id="c6d75-477">Remote exception handling</span></span>](/azure/service-fabric/service-fabric-reliable-services-communication-remoting#remoting-exception-handling)

## <a name="sql-database-using-adonet"></a><span data-ttu-id="c6d75-478">Использование Базы данных SQL с ADO.NET</span><span class="sxs-lookup"><span data-stu-id="c6d75-478">SQL Database using ADO.NET</span></span>

<span data-ttu-id="c6d75-479">База данных SQL является размещенной базой данных SQL различных размеров и является как стандартной (общей), так и премиальной (частной) службой.</span><span class="sxs-lookup"><span data-stu-id="c6d75-479">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="c6d75-480">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="c6d75-480">Retry mechanism</span></span>

<span data-ttu-id="c6d75-481">База данных SQL не имеет встроенной поддержки выполнения повторных попыток, если доступ осуществляется с помощью ADO.NET.</span><span class="sxs-lookup"><span data-stu-id="c6d75-481">SQL Database has no built-in support for retries when accessed using ADO.NET.</span></span> <span data-ttu-id="c6d75-482">Однако коды возврата от запросов могут быть использованы для определения причины сбоя запроса.</span><span class="sxs-lookup"><span data-stu-id="c6d75-482">However, the return codes from requests can be used to determine why a request failed.</span></span> <span data-ttu-id="c6d75-483">Дополнительные сведения о регулировании базы данных SQL см. в статье [Ограничения ресурсов базы данных SQL Azure](/azure/sql-database/sql-database-resource-limits).</span><span class="sxs-lookup"><span data-stu-id="c6d75-483">For more information about SQL Database throttling, see [Azure SQL Database resource limits](/azure/sql-database/sql-database-resource-limits).</span></span> <span data-ttu-id="c6d75-484">Список кодов ошибок см. в описании [кодов ошибок SQL для клиентских приложений Базы данных SQL](/azure/sql-database/sql-database-develop-error-messages).</span><span class="sxs-lookup"><span data-stu-id="c6d75-484">For a list of relevant error codes, see [SQL error codes for SQL Database client applications](/azure/sql-database/sql-database-develop-error-messages).</span></span>

<span data-ttu-id="c6d75-485">Для реализации повторных попыток при обращении к Базе данных SQL вы можете использовать библиотеку Polly.</span><span class="sxs-lookup"><span data-stu-id="c6d75-485">You can use the Polly library to implement retries for SQL Database.</span></span> <span data-ttu-id="c6d75-486">См. руководство по [обработке временного сбоя с помощью Polly](#transient-fault-handling-with-polly).</span><span class="sxs-lookup"><span data-stu-id="c6d75-486">See [Transient fault handling with Polly](#transient-fault-handling-with-polly).</span></span>

### <a name="retry-usage-guidance"></a><span data-ttu-id="c6d75-487">Руководство по использованию механизма повторов</span><span class="sxs-lookup"><span data-stu-id="c6d75-487">Retry usage guidance</span></span>

<span data-ttu-id="c6d75-488">При доступе к базе данных SQL с помощью ADO.NET придерживайтесь следующих рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="c6d75-488">Consider the following guidelines when accessing SQL Database using ADO.NET:</span></span>

- <span data-ttu-id="c6d75-489">Выберите соответствующую службу (общую или премиальную).</span><span class="sxs-lookup"><span data-stu-id="c6d75-489">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="c6d75-490">Общий экземпляр может испытывать большие задержки подключения и регулирования чем обычно из-за использования другими клиентами общего сервера.</span><span class="sxs-lookup"><span data-stu-id="c6d75-490">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="c6d75-491">Если требуется более прогнозируемая производительность и выполнение надежных операций с низкой задержкой, попробуйте выбрать премиальный вариант.</span><span class="sxs-lookup"><span data-stu-id="c6d75-491">If more predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>
- <span data-ttu-id="c6d75-492">Убедитесь, что выполнение повторов происходит на соответствующем уровне или области во избежание операций, не являющихся идемпотентными, что может вызвать несоответствие в данных.</span><span class="sxs-lookup"><span data-stu-id="c6d75-492">Ensure that you perform retries at the appropriate level or scope to avoid non-idempotent operations causing inconsistency in the data.</span></span> <span data-ttu-id="c6d75-493">В идеальном случае все операции должны быть идемпотентными таким образом, чтобы их можно было повторить без возникновения несоответствия.</span><span class="sxs-lookup"><span data-stu-id="c6d75-493">Ideally, all operations should be idempotent so that they can be repeated without causing inconsistency.</span></span> <span data-ttu-id="c6d75-494">Если это не так, повтор следует производить на уровне или в области, которая позволяет отменить все связанные изменения, если одна операция завершится ошибкой. Например, в области транзакций.</span><span class="sxs-lookup"><span data-stu-id="c6d75-494">Where this is not the case, the retry should be performed at a level or scope that allows all related changes to be undone if one operation fails; for example, from within a transactional scope.</span></span> <span data-ttu-id="c6d75-495">Для получения дополнительных сведений обратитесь к разделу [Основы облачной службы. Уровень доступа к данным: обработка временных ошибок](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span><span class="sxs-lookup"><span data-stu-id="c6d75-495">For more information, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).</span></span>
- <span data-ttu-id="c6d75-496">Не рекомендуется использование стратегии с фиксированным интервалом при работе с базой данных SQL Azure, кроме интерактивных сценариев при наличии нескольких попыток повтора с очень короткими интервалами.</span><span class="sxs-lookup"><span data-stu-id="c6d75-496">A fixed interval strategy is not recommended for use with Azure SQL Database except for interactive scenarios where there are only a few retries at very short intervals.</span></span> <span data-ttu-id="c6d75-497">Вместо этого рекомендуется использовать стратегию экспоненциальной отсрочки для большинства сценариев.</span><span class="sxs-lookup"><span data-stu-id="c6d75-497">Instead, consider using an exponential back-off strategy for the majority of scenarios.</span></span>
- <span data-ttu-id="c6d75-498">Выберите подходящее значение для времени ожидания подключения и команды при определении подключения.</span><span class="sxs-lookup"><span data-stu-id="c6d75-498">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="c6d75-499">Слишком короткое время ожидания может привести к преждевременным сбоям подключений, когда база данных занята.</span><span class="sxs-lookup"><span data-stu-id="c6d75-499">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="c6d75-500">Слишком длинное время ожидания может помешать правильной работе логики повторов из-за слишком долгого ожидания до обнаружения ошибки соединения.</span><span class="sxs-lookup"><span data-stu-id="c6d75-500">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="c6d75-501">Значение времени ожидания — это компонент сквозной задержки; фактически это значение добавляется к величине задержки повтора, указанной в политике повтора для каждой попытки.</span><span class="sxs-lookup"><span data-stu-id="c6d75-501">The value of the timeout is a component of the end-to-end latency; it is effectively added to the retry delay specified in the retry policy for every retry attempt.</span></span>
- <span data-ttu-id="c6d75-502">Закройте соединение после определенного числа повторных попыток, даже если используется экспоненциальная логика повторных попыток, и повторите операцию с новым подключением.</span><span class="sxs-lookup"><span data-stu-id="c6d75-502">Close the connection after a certain number of retries, even when using an exponential back off retry logic, and retry the operation on a new connection.</span></span> <span data-ttu-id="c6d75-503">Повторение одной и той же операции несколько раз для одного соединения может стать фактором возникновения проблем с подключением.</span><span class="sxs-lookup"><span data-stu-id="c6d75-503">Retrying the same operation multiple times on the same connection can be a factor that contributes to connection problems.</span></span> <span data-ttu-id="c6d75-504">Пример использования этой технологии см. в статье [Основы облачной службы. Уровень доступа к данным: обработка временных ошибок](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span><span class="sxs-lookup"><span data-stu-id="c6d75-504">For an example of this technique, see [Cloud Service Fundamentals Data Access Layer – Transient Fault Handling](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).</span></span>
- <span data-ttu-id="c6d75-505">Когда используется пул подключений (по умолчанию), существует вероятность того, что будет выбрано то же подключение из пула, даже после закрытия и повторного открытия соединения.</span><span class="sxs-lookup"><span data-stu-id="c6d75-505">When connection pooling is in use (the default) there is a chance that the same connection will be chosen from the pool, even after closing and reopening a connection.</span></span> <span data-ttu-id="c6d75-506">Если это так, способом устранения подобной ситуации является вызов метода **ClearPool** класса **SqlConnection**, чтобы отметить подключение как не используемое повторно.</span><span class="sxs-lookup"><span data-stu-id="c6d75-506">If this is the case, a technique to resolve it is to call the **ClearPool** method of the **SqlConnection** class to mark the connection as not reusable.</span></span> <span data-ttu-id="c6d75-507">Тем не менее это следует делать только после сбоя нескольких попыток соединения и только при обнаружении определенного класса временных ошибок, таких как время ожидания SQL (код ошибки -2), связанных со сбоями в подключениях.</span><span class="sxs-lookup"><span data-stu-id="c6d75-507">However, you should do this only after several connection attempts have failed, and only when encountering the specific class of transient failures such as SQL timeouts (error code -2) related to faulty connections.</span></span>
- <span data-ttu-id="c6d75-508">Если код доступа к данным использует транзакции, инициируемые как экземпляры **TransactionScope** , то логика повторных попыток должна включать повторное открытие подключения и инициацию новой области транзакции.</span><span class="sxs-lookup"><span data-stu-id="c6d75-508">If the data access code uses transactions initiated as **TransactionScope** instances, the retry logic should reopen the connection and initiate a new transaction scope.</span></span> <span data-ttu-id="c6d75-509">По этой причине блок кода повторной попытки должен охватывать всю область транзакции.</span><span class="sxs-lookup"><span data-stu-id="c6d75-509">For this reason, the retryable code block should encompass the entire scope of the transaction.</span></span>

<span data-ttu-id="c6d75-510">Начните с использования следующих параметров для операции повтора.</span><span class="sxs-lookup"><span data-stu-id="c6d75-510">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="c6d75-511">Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.</span><span class="sxs-lookup"><span data-stu-id="c6d75-511">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="c6d75-512">**Контекст**</span><span class="sxs-lookup"><span data-stu-id="c6d75-512">**Context**</span></span> | <span data-ttu-id="c6d75-513">**Максимальная задержка<br />примера целевого E2E**</span><span class="sxs-lookup"><span data-stu-id="c6d75-513">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="c6d75-514">**Стратегия повторов**</span><span class="sxs-lookup"><span data-stu-id="c6d75-514">**Retry strategy**</span></span> | <span data-ttu-id="c6d75-515">**Параметры**</span><span class="sxs-lookup"><span data-stu-id="c6d75-515">**Settings**</span></span> | <span data-ttu-id="c6d75-516">**Значения**</span><span class="sxs-lookup"><span data-stu-id="c6d75-516">**Values**</span></span> | <span data-ttu-id="c6d75-517">**Принцип работы**</span><span class="sxs-lookup"><span data-stu-id="c6d75-517">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="c6d75-518">Интерактивный, пользовательский интерфейс</span><span class="sxs-lookup"><span data-stu-id="c6d75-518">Interactive, UI,</span></span><br /><span data-ttu-id="c6d75-519">или передний план</span><span class="sxs-lookup"><span data-stu-id="c6d75-519">or foreground</span></span> |<span data-ttu-id="c6d75-520">2 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-520">2 sec</span></span> |<span data-ttu-id="c6d75-521">FixedInterval</span><span class="sxs-lookup"><span data-stu-id="c6d75-521">FixedInterval</span></span> |<span data-ttu-id="c6d75-522">Число повторных попыток</span><span class="sxs-lookup"><span data-stu-id="c6d75-522">Retry count</span></span><br /><span data-ttu-id="c6d75-523">Интервал попытки</span><span class="sxs-lookup"><span data-stu-id="c6d75-523">Retry interval</span></span><br /><span data-ttu-id="c6d75-524">Первый быстрый повтор</span><span class="sxs-lookup"><span data-stu-id="c6d75-524">First fast retry</span></span> |<span data-ttu-id="c6d75-525">3</span><span class="sxs-lookup"><span data-stu-id="c6d75-525">3</span></span><br /><span data-ttu-id="c6d75-526">500 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-526">500 ms</span></span><br /><span data-ttu-id="c6d75-527">true</span><span class="sxs-lookup"><span data-stu-id="c6d75-527">true</span></span> |<span data-ttu-id="c6d75-528">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-528">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="c6d75-529">Попытка 2 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-529">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="c6d75-530">попытка 3 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-530">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="c6d75-531">Фоновый</span><span class="sxs-lookup"><span data-stu-id="c6d75-531">Background</span></span><br /><span data-ttu-id="c6d75-532">или пакетный</span><span class="sxs-lookup"><span data-stu-id="c6d75-532">or batch</span></span> |<span data-ttu-id="c6d75-533">30 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-533">30 sec</span></span> |<span data-ttu-id="c6d75-534">ExponentialBackoff</span><span class="sxs-lookup"><span data-stu-id="c6d75-534">ExponentialBackoff</span></span> |<span data-ttu-id="c6d75-535">Число повторных попыток</span><span class="sxs-lookup"><span data-stu-id="c6d75-535">Retry count</span></span><br /><span data-ttu-id="c6d75-536">Минимальная задержка</span><span class="sxs-lookup"><span data-stu-id="c6d75-536">Min back-off</span></span><br /><span data-ttu-id="c6d75-537">Максимальная задержка</span><span class="sxs-lookup"><span data-stu-id="c6d75-537">Max back-off</span></span><br /><span data-ttu-id="c6d75-538">Разностная задержка</span><span class="sxs-lookup"><span data-stu-id="c6d75-538">Delta back-off</span></span><br /><span data-ttu-id="c6d75-539">Первый быстрый повтор</span><span class="sxs-lookup"><span data-stu-id="c6d75-539">First fast retry</span></span> |<span data-ttu-id="c6d75-540">5</span><span class="sxs-lookup"><span data-stu-id="c6d75-540">5</span></span><br /><span data-ttu-id="c6d75-541">0 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-541">0 sec</span></span><br /><span data-ttu-id="c6d75-542">60 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-542">60 sec</span></span><br /><span data-ttu-id="c6d75-543">2 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-543">2 sec</span></span><br /><span data-ttu-id="c6d75-544">false</span><span class="sxs-lookup"><span data-stu-id="c6d75-544">false</span></span> |<span data-ttu-id="c6d75-545">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-545">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="c6d75-546">Попытка 2 — задержка 2 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-546">Attempt 2 - delay ~2 sec</span></span><br /><span data-ttu-id="c6d75-547">Попытка 3 — задержка 6 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-547">Attempt 3 - delay ~6 sec</span></span><br /><span data-ttu-id="c6d75-548">Попытка 4 — задержка 14 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-548">Attempt 4 - delay ~14 sec</span></span><br /><span data-ttu-id="c6d75-549">Попытка 5 — задержка 30 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-549">Attempt 5 - delay ~30 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="c6d75-550">Целевые значения сквозной задержки подразумевают время ожидания по умолчанию для подключения к службе.</span><span class="sxs-lookup"><span data-stu-id="c6d75-550">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="c6d75-551">При указании более длинного времени ожидания подключения величина сквозной задержки будет увеличена путем добавления этого дополнительного времени для каждой попытки повтора.</span><span class="sxs-lookup"><span data-stu-id="c6d75-551">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>

### <a name="examples"></a><span data-ttu-id="c6d75-552">Примеры</span><span class="sxs-lookup"><span data-stu-id="c6d75-552">Examples</span></span>

<span data-ttu-id="c6d75-553">В этом разделе показано, как с помощью Polly обращаться к Базе данных SQL Azure с использованием набора политик повторных попыток, настроенных в классе `Policy`.</span><span class="sxs-lookup"><span data-stu-id="c6d75-553">This section shows how you can use Polly to access Azure SQL Database using a set of retry policies configured in the `Policy` class.</span></span>

<span data-ttu-id="c6d75-554">Следующий код демонстрирует метод расширения для класса `SqlCommand`, который вызывает `ExecuteAsync` с экспоненциальным увеличением задержки.</span><span class="sxs-lookup"><span data-stu-id="c6d75-554">The following code shows an extension method on the `SqlCommand` class that calls `ExecuteAsync` with exponential backoff.</span></span>

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

<span data-ttu-id="c6d75-555">Этот асинхронный метод расширения можно использовать следующим образом.</span><span class="sxs-lookup"><span data-stu-id="c6d75-555">This asynchronous extension method can be used as follows.</span></span>

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a><span data-ttu-id="c6d75-556">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="c6d75-556">More information</span></span>

- [<span data-ttu-id="c6d75-557">Основы облачной службы. Уровень доступа к данным: обработка временных ошибок</span><span class="sxs-lookup"><span data-stu-id="c6d75-557">Cloud Service Fundamentals Data Access Layer – Transient Fault Handling</span></span>](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

<span data-ttu-id="c6d75-558">Общие рекомендации по повышению эффективности Базы данных SQL см. в руководстве по [обеспечению эластичности и производительности Базы данных SQL Azure](https://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span><span class="sxs-lookup"><span data-stu-id="c6d75-558">For general guidance on getting the most from SQL Database, see [Azure SQL Database performance and elasticity guide](https://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).</span></span>

## <a name="sql-database-using-entity-framework-6"></a><span data-ttu-id="c6d75-559">Использование Базы данных SQL с Entity Framework 6</span><span class="sxs-lookup"><span data-stu-id="c6d75-559">SQL Database using Entity Framework 6</span></span>

<span data-ttu-id="c6d75-560">База данных SQL является размещенной базой данных SQL различных размеров и является как стандартной (общей), так и премиальной (частной) службой.</span><span class="sxs-lookup"><span data-stu-id="c6d75-560">SQL Database is a hosted SQL database available in a range of sizes and as both a standard (shared) and premium (non-shared) service.</span></span> <span data-ttu-id="c6d75-561">Механизм Entity Framework является модулем объектно-реляционного сопоставления, который позволяет разработчикам .NET работать с реляционными данными с помощью специфических для домена объектов.</span><span class="sxs-lookup"><span data-stu-id="c6d75-561">Entity Framework is an object-relational mapper that enables .NET developers to work with relational data using domain-specific objects.</span></span> <span data-ttu-id="c6d75-562">Это исключает необходимость в большинстве кодов доступа к данным, которые обычно требуется писать разработчикам.</span><span class="sxs-lookup"><span data-stu-id="c6d75-562">It eliminates the need for most of the data-access code that developers usually need to write.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="c6d75-563">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="c6d75-563">Retry mechanism</span></span>

<span data-ttu-id="c6d75-564">Поддержка повторных попыток при доступе к Базе данных SQL с помощью Entity Framework 6.0 и выше предоставляется через механизм, называемый [устойчивость подключения и логика повторных попыток](/ef/ef6/fundamentals/connection-resiliency/retry-logic).</span><span class="sxs-lookup"><span data-stu-id="c6d75-564">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher through a mechanism called [Connection resiliency / retry logic](/ef/ef6/fundamentals/connection-resiliency/retry-logic).</span></span> <span data-ttu-id="c6d75-565">Ниже перечислены основные возможности механизма повтора.</span><span class="sxs-lookup"><span data-stu-id="c6d75-565">The main features of the retry mechanism are:</span></span>

- <span data-ttu-id="c6d75-566">Интерфейс **IDbExecutionStrategy** является основной абстракцией.</span><span class="sxs-lookup"><span data-stu-id="c6d75-566">The primary abstraction is the **IDbExecutionStrategy** interface.</span></span> <span data-ttu-id="c6d75-567">Этот интерфейс</span><span class="sxs-lookup"><span data-stu-id="c6d75-567">This interface:</span></span>
  - <span data-ttu-id="c6d75-568">Определяет синхронные и асинхронные методы **Execute** \*.</span><span class="sxs-lookup"><span data-stu-id="c6d75-568">Defines synchronous and asynchronous **Execute**\* methods.</span></span>
  - <span data-ttu-id="c6d75-569">Определяет классы для непосредственного использования, или которые могут быть настроены на контекст базы данных как стратегию по умолчанию, с сопоставленным именем поставщика или сопоставленным именем поставщика и именем сервера.</span><span class="sxs-lookup"><span data-stu-id="c6d75-569">Defines classes that can be used directly or can be configured on a database context as a default strategy, mapped to provider name, or mapped to a provider name and server name.</span></span> <span data-ttu-id="c6d75-570">При настройке на контекст повторы происходят на уровне операций с отдельными базами данных, из которых несколько может соответствовать заданному контексту операции.</span><span class="sxs-lookup"><span data-stu-id="c6d75-570">When configured on a context, retries occur at the level of individual database operations, of which there might be several for a given context operation.</span></span>
  - <span data-ttu-id="c6d75-571">Определяет, когда и каким образом следует выполнить повтор при ошибке соединения.</span><span class="sxs-lookup"><span data-stu-id="c6d75-571">Defines when to retry a failed connection, and how.</span></span>
- <span data-ttu-id="c6d75-572">Включает несколько встроенных реализаций интерфейса **IDbExecutionStrategy** .</span><span class="sxs-lookup"><span data-stu-id="c6d75-572">It includes several built-in implementations of the **IDbExecutionStrategy** interface:</span></span>
  - <span data-ttu-id="c6d75-573">Значение по умолчанию — не повторять.</span><span class="sxs-lookup"><span data-stu-id="c6d75-573">Default - no retrying.</span></span>
  - <span data-ttu-id="c6d75-574">По умолчанию для базы данных SQL (автоматический режим) — не повторять, но проверять исключения и завершать их с предложением использовать стратегию базы данных SQL.</span><span class="sxs-lookup"><span data-stu-id="c6d75-574">Default for SQL Database (automatic) - no retrying, but inspects exceptions and wraps them with suggestion to use the SQL Database strategy.</span></span>
  - <span data-ttu-id="c6d75-575">По умолчанию для базы данных SQL — экспоненциальный повтор (наследуется от базового класса) плюс логика обнаружения базы данных SQL.</span><span class="sxs-lookup"><span data-stu-id="c6d75-575">Default for SQL Database - exponential (inherited from base class) plus SQL Database detection logic.</span></span>
- <span data-ttu-id="c6d75-576">Это реализует стратегию экспоненциальной отсрочки, которая также включает случайный выбор.</span><span class="sxs-lookup"><span data-stu-id="c6d75-576">It implements an exponential back-off strategy that includes randomization.</span></span>
- <span data-ttu-id="c6d75-577">Встроенные классы повтора включают отслеживание состояния и не являются потокобезопасными.</span><span class="sxs-lookup"><span data-stu-id="c6d75-577">The built-in retry classes are stateful and are not thread safe.</span></span> <span data-ttu-id="c6d75-578">Однако они могут использоваться повторно после завершения текущей операции.</span><span class="sxs-lookup"><span data-stu-id="c6d75-578">However, they can be reused after the current operation is completed.</span></span>
- <span data-ttu-id="c6d75-579">В случае превышения заданного счетчика повторов результаты помещаются в новое исключение.</span><span class="sxs-lookup"><span data-stu-id="c6d75-579">If the specified retry count is exceeded, the results are wrapped in a new exception.</span></span> <span data-ttu-id="c6d75-580">Это не вызывает перенаправления текущего исключения.</span><span class="sxs-lookup"><span data-stu-id="c6d75-580">It does not bubble up the current exception.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="c6d75-581">Настройки политики</span><span class="sxs-lookup"><span data-stu-id="c6d75-581">Policy configuration</span></span>

<span data-ttu-id="c6d75-582">Поддержка повторных попыток предоставляется при доступе к базе данных SQL с помощью Entity Framework 6.0 и более поздних версий.</span><span class="sxs-lookup"><span data-stu-id="c6d75-582">Retry support is provided when accessing SQL Database using Entity Framework 6.0 and higher.</span></span> <span data-ttu-id="c6d75-583">Политики повтора настраиваются программным способом.</span><span class="sxs-lookup"><span data-stu-id="c6d75-583">Retry policies are configured programmatically.</span></span> <span data-ttu-id="c6d75-584">Нельзя изменить конфигурацию для каждой операции.</span><span class="sxs-lookup"><span data-stu-id="c6d75-584">The configuration cannot be changed on a per-operation basis.</span></span>

<span data-ttu-id="c6d75-585">При настройке стратегии в контексте по умолчанию, необходимо указать функцию, которая создает новые стратегии по требованию.</span><span class="sxs-lookup"><span data-stu-id="c6d75-585">When configuring a strategy on the context as the default, you specify a function that creates a new strategy on demand.</span></span> <span data-ttu-id="c6d75-586">В следующем коде показано, как можно создать класс конфигурации повтора, который расширяет базовый класс **DbConfiguration** .</span><span class="sxs-lookup"><span data-stu-id="c6d75-586">The following code shows how you can create a retry configuration class that extends the **DbConfiguration** base class.</span></span>

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

<span data-ttu-id="c6d75-587">Затем можно указать это в качестве стратегии повторных попыток по умолчанию для всех операций с помощью метода **SetConfiguration** экземпляра **DbConfiguration** при запуске приложения.</span><span class="sxs-lookup"><span data-stu-id="c6d75-587">You can then specify this as the default retry strategy for all operations using the **SetConfiguration** method of the **DbConfiguration** instance when the application starts.</span></span> <span data-ttu-id="c6d75-588">По умолчанию EF будет автоматически обнаруживать и использовать класс конфигурации.</span><span class="sxs-lookup"><span data-stu-id="c6d75-588">By default, EF will automatically discover and use the configuration class.</span></span>

```csharp
DbConfiguration.SetConfiguration(new BloggingContextConfiguration());
```

<span data-ttu-id="c6d75-589">Класс конфигурации повторных попыток для контекста задается дополнением контекста класса атрибутом **DbConfigurationType** .</span><span class="sxs-lookup"><span data-stu-id="c6d75-589">You can specify the retry configuration class for a context by annotating the context class with a **DbConfigurationType** attribute.</span></span> <span data-ttu-id="c6d75-590">Однако при наличии только одного класса конфигурации EF будет использовать его без необходимости дополнения контекста.</span><span class="sxs-lookup"><span data-stu-id="c6d75-590">However, if you have only one configuration class, EF will use it without the need to annotate the context.</span></span>

```csharp
[DbConfigurationType(typeof(BloggingContextConfiguration))]
public class BloggingContext : DbContext
```

<span data-ttu-id="c6d75-591">Если необходимо использовать различные стратегии для конкретных операций или отключить повторы для определенных операций, то можно создать класс конфигурации, который позволяет приостановить или заменить стратегии, установив флаг **CallContext**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-591">If you need to use different retry strategies for specific operations, or disable retries for specific operations, you can create a configuration class that allows you to suspend or swap strategies by setting a flag in the **CallContext**.</span></span> <span data-ttu-id="c6d75-592">Класс конфигурации может использовать этот флаг для переключения стратегии или отключения стратегии, предоставленной вами, с последующим использованием стратегии по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="c6d75-592">The configuration class can use this flag to switch strategies, or disable the strategy you provide and use a default strategy.</span></span> <span data-ttu-id="c6d75-593">Дополнительные сведения см. в разделе [Приостановка выполнения стратегии](/ef/ef6/fundamentals/connection-resiliency/retry-logic#workaround-suspend-execution-strategy) (для EF6 и выше).</span><span class="sxs-lookup"><span data-stu-id="c6d75-593">For more information, see [Suspend Execution Strategy](/ef/ef6/fundamentals/connection-resiliency/retry-logic#workaround-suspend-execution-strategy) (EF6 onwards).</span></span>

<span data-ttu-id="c6d75-594">Иным способом использования стратегий повторов, определенных для отдельных операций, является создание экземпляра класса требуемой стратегии и передача ему нужных параметров.</span><span class="sxs-lookup"><span data-stu-id="c6d75-594">Another technique for using specific retry strategies for individual operations is to create an instance of the required strategy class and supply the desired settings through parameters.</span></span> <span data-ttu-id="c6d75-595">Затем можно вызвать его метод **ExecuteAsync** .</span><span class="sxs-lookup"><span data-stu-id="c6d75-595">You then invoke its **ExecuteAsync** method.</span></span>

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

<span data-ttu-id="c6d75-596">Самый простой способ использовать класс **DbConfiguration** — это обнаружить его в той же сборке, что и класс **DbContext**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-596">The simplest way to use a **DbConfiguration** class is to locate it in the same assembly as the **DbContext** class.</span></span> <span data-ttu-id="c6d75-597">Однако этот способ не подходит в случае, когда тот же контекст требуется в различных сценариях, таких как различные интерактивные и фоновые стратегии повторов.</span><span class="sxs-lookup"><span data-stu-id="c6d75-597">However, this is not appropriate when the same context is required in different scenarios, such as different interactive and background retry strategies.</span></span> <span data-ttu-id="c6d75-598">Если разные контексты выполняются в отдельных доменах приложений, можно использовать встроенную поддержку для указания классов конфигурации в файле конфигурации или задать ее явным образом с помощью кода.</span><span class="sxs-lookup"><span data-stu-id="c6d75-598">If the different contexts execute in separate AppDomains, you can use the built-in support for specifying configuration classes in the configuration file or set it explicitly using code.</span></span> <span data-ttu-id="c6d75-599">Если различные контексты должны выполняться в том же домене приложения, потребуется создание пользовательского решения.</span><span class="sxs-lookup"><span data-stu-id="c6d75-599">If the different contexts must execute in the same AppDomain, a custom solution will be required.</span></span>

<span data-ttu-id="c6d75-600">Дополнительные сведения см. в статье [Конфигурация на основе кода (для EF6 и выше)](/ef/ef6/fundamentals/configuring/code-based).</span><span class="sxs-lookup"><span data-stu-id="c6d75-600">For more information, see [Code-Based Configuration](/ef/ef6/fundamentals/configuring/code-based) (EF6 onwards).</span></span>

<span data-ttu-id="c6d75-601">Следующая таблица показывает значения по умолчанию для встроенной политики повторов при использовании EF6.</span><span class="sxs-lookup"><span data-stu-id="c6d75-601">The following table shows the default settings for the built-in retry policy when using EF6.</span></span>

| <span data-ttu-id="c6d75-602">Параметр</span><span class="sxs-lookup"><span data-stu-id="c6d75-602">Setting</span></span> | <span data-ttu-id="c6d75-603">Значение по умолчанию</span><span class="sxs-lookup"><span data-stu-id="c6d75-603">Default value</span></span> | <span data-ttu-id="c6d75-604">Значение</span><span class="sxs-lookup"><span data-stu-id="c6d75-604">Meaning</span></span> |
|---------|---------------|---------|
| <span data-ttu-id="c6d75-605">Политика</span><span class="sxs-lookup"><span data-stu-id="c6d75-605">Policy</span></span> | <span data-ttu-id="c6d75-606">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="c6d75-606">Exponential</span></span> | <span data-ttu-id="c6d75-607">Экспоненциальная задержка.</span><span class="sxs-lookup"><span data-stu-id="c6d75-607">Exponential back-off.</span></span> |
| <span data-ttu-id="c6d75-608">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="c6d75-608">MaxRetryCount</span></span> | <span data-ttu-id="c6d75-609">5</span><span class="sxs-lookup"><span data-stu-id="c6d75-609">5</span></span> | <span data-ttu-id="c6d75-610">Максимальное число попыток.</span><span class="sxs-lookup"><span data-stu-id="c6d75-610">The maximum number of retries.</span></span> |
| <span data-ttu-id="c6d75-611">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="c6d75-611">MaxDelay</span></span> | <span data-ttu-id="c6d75-612">30 секунд</span><span class="sxs-lookup"><span data-stu-id="c6d75-612">30 seconds</span></span> | <span data-ttu-id="c6d75-613">Максимальная задержка между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="c6d75-613">The maximum delay between retries.</span></span> <span data-ttu-id="c6d75-614">Это значение не влияет на то, как вычисляется значение серии задержек.</span><span class="sxs-lookup"><span data-stu-id="c6d75-614">This value does not affect how the series of delays are computed.</span></span> <span data-ttu-id="c6d75-615">Оно определяет только верхнюю границу.</span><span class="sxs-lookup"><span data-stu-id="c6d75-615">It only defines an upper bound.</span></span> |
| <span data-ttu-id="c6d75-616">DefaultCoefficient</span><span class="sxs-lookup"><span data-stu-id="c6d75-616">DefaultCoefficient</span></span> | <span data-ttu-id="c6d75-617">1 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-617">1 second</span></span> | <span data-ttu-id="c6d75-618">Коэффициент для вычисления значения экспоненциальной задержки.</span><span class="sxs-lookup"><span data-stu-id="c6d75-618">The coefficient for the exponential back-off computation.</span></span> <span data-ttu-id="c6d75-619">Это значение не может быть изменено.</span><span class="sxs-lookup"><span data-stu-id="c6d75-619">This value cannot be changed.</span></span> |
| <span data-ttu-id="c6d75-620">DefaultRandomFactor</span><span class="sxs-lookup"><span data-stu-id="c6d75-620">DefaultRandomFactor</span></span> | <span data-ttu-id="c6d75-621">1,1</span><span class="sxs-lookup"><span data-stu-id="c6d75-621">1.1</span></span> | <span data-ttu-id="c6d75-622">Множитель, который используется для добавления значения случайной задержки для каждой записи.</span><span class="sxs-lookup"><span data-stu-id="c6d75-622">The multiplier used to add a random delay for each entry.</span></span> <span data-ttu-id="c6d75-623">Это значение не может быть изменено.</span><span class="sxs-lookup"><span data-stu-id="c6d75-623">This value cannot be changed.</span></span> |
| <span data-ttu-id="c6d75-624">DefaultExponentialBase</span><span class="sxs-lookup"><span data-stu-id="c6d75-624">DefaultExponentialBase</span></span> | <span data-ttu-id="c6d75-625">2</span><span class="sxs-lookup"><span data-stu-id="c6d75-625">2</span></span> | <span data-ttu-id="c6d75-626">Множитель, который используется для вычисления значения следующей задержки.</span><span class="sxs-lookup"><span data-stu-id="c6d75-626">The multiplier used to calculate the next delay.</span></span> <span data-ttu-id="c6d75-627">Это значение не может быть изменено.</span><span class="sxs-lookup"><span data-stu-id="c6d75-627">This value cannot be changed.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="c6d75-628">Руководство по использованию механизма повторов</span><span class="sxs-lookup"><span data-stu-id="c6d75-628">Retry usage guidance</span></span>

<span data-ttu-id="c6d75-629">При доступе к базе данных SQL с помощью EF6, придерживайтесь следующих рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="c6d75-629">Consider the following guidelines when accessing SQL Database using EF6:</span></span>

- <span data-ttu-id="c6d75-630">Выберите соответствующую службу (общую или премиальную).</span><span class="sxs-lookup"><span data-stu-id="c6d75-630">Choose the appropriate service option (shared or premium).</span></span> <span data-ttu-id="c6d75-631">Общий экземпляр может испытывать большие задержки подключения и регулирования чем обычно из-за использования другими клиентами общего сервера.</span><span class="sxs-lookup"><span data-stu-id="c6d75-631">A shared instance may suffer longer than usual connection delays and throttling due to the usage by other tenants of the shared server.</span></span> <span data-ttu-id="c6d75-632">Если требуется выполнение надежных операций с низкой задержкой и прогнозируемая производительность, попробуйте выбрать премиальный вариант.</span><span class="sxs-lookup"><span data-stu-id="c6d75-632">If predictable performance and reliable low latency operations are required, consider choosing the premium option.</span></span>

- <span data-ttu-id="c6d75-633">Стратегия фиксированного интервала не рекомендуется для использования с базой данных SQL Azure.</span><span class="sxs-lookup"><span data-stu-id="c6d75-633">A fixed interval strategy is not recommended for use with Azure SQL Database.</span></span> <span data-ttu-id="c6d75-634">Вместо этого используйте стратегию экспоненциальной отсрочки, так как служба может быть перегружена и большее время задержки обеспечит большее время для ее восстановления.</span><span class="sxs-lookup"><span data-stu-id="c6d75-634">Instead, use an exponential back-off strategy because the service may be overloaded, and longer delays allow more time for it to recover.</span></span>

- <span data-ttu-id="c6d75-635">Выберите подходящее значение для времени ожидания подключения и команды при определении подключения.</span><span class="sxs-lookup"><span data-stu-id="c6d75-635">Choose a suitable value for the connection and command timeouts when defining connections.</span></span> <span data-ttu-id="c6d75-636">Выберите значение времени ожидания на основе логики вашей работы и на основе испытаний.</span><span class="sxs-lookup"><span data-stu-id="c6d75-636">Base the timeout on both your business logic design and through testing.</span></span> <span data-ttu-id="c6d75-637">Может потребоваться изменить это значение по мере изменения объемов данных или бизнес-процессов.</span><span class="sxs-lookup"><span data-stu-id="c6d75-637">You may need to modify this value over time as the volumes of data or the business processes change.</span></span> <span data-ttu-id="c6d75-638">Слишком короткое время ожидания может привести к преждевременным сбоям подключений, когда база данных занята.</span><span class="sxs-lookup"><span data-stu-id="c6d75-638">Too short a timeout may result in premature failures of connections when the database is busy.</span></span> <span data-ttu-id="c6d75-639">Слишком длинное время ожидания может помешать правильной работе логики повторов из-за слишком долгого ожидания до обнаружения ошибки соединения.</span><span class="sxs-lookup"><span data-stu-id="c6d75-639">Too long a timeout may prevent the retry logic working correctly by waiting too long before detecting a failed connection.</span></span> <span data-ttu-id="c6d75-640">Значение времени ожидания — это компонент сквозной задержки, несмотря на это вы не можете легко определить, сколько команд будет выполнено при сохранении контекста.</span><span class="sxs-lookup"><span data-stu-id="c6d75-640">The value of the timeout is a component of the end-to-end latency, although you cannot easily determine how many commands will execute when saving the context.</span></span> <span data-ttu-id="c6d75-641">Время ожидания по умолчанию можно изменить, задав свойство **CommandTimeout** экземпляра **DbContext**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-641">You can change the default timeout by setting the **CommandTimeout** property of the **DbContext** instance.</span></span>

- <span data-ttu-id="c6d75-642">Платформа Entity Framework поддерживает конфигурации повторов, определенные в файлах конфигурации.</span><span class="sxs-lookup"><span data-stu-id="c6d75-642">Entity Framework supports retry configurations defined in configuration files.</span></span> <span data-ttu-id="c6d75-643">Однако для максимальной гибкости в Azure необходимо создать конфигурацию программным способом в приложении.</span><span class="sxs-lookup"><span data-stu-id="c6d75-643">However, for maximum flexibility on Azure you should consider creating the configuration programmatically within the application.</span></span> <span data-ttu-id="c6d75-644">Определенные параметры политик повторов, например количество повторных попыток и интервалы повтора, можно сохранить в файле конфигурации службы и использовать во время выполнения для создания соответствующих политик.</span><span class="sxs-lookup"><span data-stu-id="c6d75-644">The specific parameters for the retry policies, such as the number of retries and the retry intervals, can be stored in the service configuration file and used at runtime to create the appropriate policies.</span></span> <span data-ttu-id="c6d75-645">Это позволяет изменять параметры без перезапуска приложения.</span><span class="sxs-lookup"><span data-stu-id="c6d75-645">This allows the settings to be changed without requiring the application to be restarted.</span></span>

<span data-ttu-id="c6d75-646">Для начала установите следующие параметры для операций повтора.</span><span class="sxs-lookup"><span data-stu-id="c6d75-646">Consider starting with the following settings for retrying operations.</span></span> <span data-ttu-id="c6d75-647">Нельзя указать задержку между попытками повтора (она является фиксированной и генерируется в виде экспоненциальной последовательности).</span><span class="sxs-lookup"><span data-stu-id="c6d75-647">You cannot specify the delay between retry attempts (it is fixed and generated as an exponential sequence).</span></span> <span data-ttu-id="c6d75-648">Максимальные значения можно указать, как показано ниже, если только вы не создаете пользовательскую стратегию.</span><span class="sxs-lookup"><span data-stu-id="c6d75-648">You can specify only the maximum values, as shown here; unless you create a custom retry strategy.</span></span> <span data-ttu-id="c6d75-649">Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.</span><span class="sxs-lookup"><span data-stu-id="c6d75-649">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="c6d75-650">**Контекст**</span><span class="sxs-lookup"><span data-stu-id="c6d75-650">**Context**</span></span> | <span data-ttu-id="c6d75-651">**Максимальная задержка<br />примера целевого E2E**</span><span class="sxs-lookup"><span data-stu-id="c6d75-651">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="c6d75-652">**Политика повтора**</span><span class="sxs-lookup"><span data-stu-id="c6d75-652">**Retry policy**</span></span> | <span data-ttu-id="c6d75-653">**Параметры**</span><span class="sxs-lookup"><span data-stu-id="c6d75-653">**Settings**</span></span> | <span data-ttu-id="c6d75-654">**Значения**</span><span class="sxs-lookup"><span data-stu-id="c6d75-654">**Values**</span></span> | <span data-ttu-id="c6d75-655">**Принцип работы**</span><span class="sxs-lookup"><span data-stu-id="c6d75-655">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="c6d75-656">Интерактивный, пользовательский интерфейс</span><span class="sxs-lookup"><span data-stu-id="c6d75-656">Interactive, UI,</span></span><br /><span data-ttu-id="c6d75-657">или передний план</span><span class="sxs-lookup"><span data-stu-id="c6d75-657">or foreground</span></span> |<span data-ttu-id="c6d75-658">2 секунды</span><span class="sxs-lookup"><span data-stu-id="c6d75-658">2 seconds</span></span> |<span data-ttu-id="c6d75-659">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="c6d75-659">Exponential</span></span> |<span data-ttu-id="c6d75-660">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="c6d75-660">MaxRetryCount</span></span><br /><span data-ttu-id="c6d75-661">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="c6d75-661">MaxDelay</span></span> |<span data-ttu-id="c6d75-662">3</span><span class="sxs-lookup"><span data-stu-id="c6d75-662">3</span></span><br /><span data-ttu-id="c6d75-663">750 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-663">750 ms</span></span> |<span data-ttu-id="c6d75-664">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-664">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="c6d75-665">Попытка 2 — задержка 750 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-665">Attempt 2 - delay 750 ms</span></span><br /><span data-ttu-id="c6d75-666">Попытка 3 – задержка 750 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-666">Attempt 3 – delay 750 ms</span></span> |
| <span data-ttu-id="c6d75-667">Фоновый</span><span class="sxs-lookup"><span data-stu-id="c6d75-667">Background</span></span><br /> <span data-ttu-id="c6d75-668">или пакетный</span><span class="sxs-lookup"><span data-stu-id="c6d75-668">or batch</span></span> |<span data-ttu-id="c6d75-669">30 секунд</span><span class="sxs-lookup"><span data-stu-id="c6d75-669">30 seconds</span></span> |<span data-ttu-id="c6d75-670">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="c6d75-670">Exponential</span></span> |<span data-ttu-id="c6d75-671">MaxRetryCount</span><span class="sxs-lookup"><span data-stu-id="c6d75-671">MaxRetryCount</span></span><br /><span data-ttu-id="c6d75-672">MaxDelay</span><span class="sxs-lookup"><span data-stu-id="c6d75-672">MaxDelay</span></span> |<span data-ttu-id="c6d75-673">5</span><span class="sxs-lookup"><span data-stu-id="c6d75-673">5</span></span><br /><span data-ttu-id="c6d75-674">12 секунд</span><span class="sxs-lookup"><span data-stu-id="c6d75-674">12 seconds</span></span> |<span data-ttu-id="c6d75-675">Попытка 1 — задержка 0 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-675">Attempt 1 - delay 0 sec</span></span><br /><span data-ttu-id="c6d75-676">Попытка 2 — задержка 1 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-676">Attempt 2 - delay ~1 sec</span></span><br /><span data-ttu-id="c6d75-677">Попытка 3 — задержка 3 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-677">Attempt 3 - delay ~3 sec</span></span><br /><span data-ttu-id="c6d75-678">Попытка 4 — задержка 7 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-678">Attempt 4 - delay ~7 sec</span></span><br /><span data-ttu-id="c6d75-679">Попытка 5 — задержка 12 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-679">Attempt 5 - delay 12 sec</span></span> |

> [!NOTE]
> <span data-ttu-id="c6d75-680">Целевые значения сквозной задержки подразумевают время ожидания по умолчанию для подключения к службе.</span><span class="sxs-lookup"><span data-stu-id="c6d75-680">The end-to-end latency targets assume the default timeout for connections to the service.</span></span> <span data-ttu-id="c6d75-681">При указании более длинного времени ожидания подключения величина сквозной задержки будет увеличена путем добавления этого дополнительного времени для каждой попытки повтора.</span><span class="sxs-lookup"><span data-stu-id="c6d75-681">If you specify longer connection timeouts, the end-to-end latency will be extended by this additional time for every retry attempt.</span></span>

### <a name="examples"></a><span data-ttu-id="c6d75-682">Примеры</span><span class="sxs-lookup"><span data-stu-id="c6d75-682">Examples</span></span>

<span data-ttu-id="c6d75-683">В следующем примере кода определяется простое решение доступа к данным, использующее Entity Framework.</span><span class="sxs-lookup"><span data-stu-id="c6d75-683">The following code example defines a simple data access solution that uses Entity Framework.</span></span> <span data-ttu-id="c6d75-684">Программа определяет конкретную стратегию путем определения класса с именем экземпляра **BlogConfiguration**, который расширяет **DbConfiguration**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-684">It sets a specific retry strategy by defining an instance of a class named **BlogConfiguration** that extends **DbConfiguration**.</span></span>

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

<span data-ttu-id="c6d75-685">Дополнительные примеры использования механизма повтора Entity Framework можно найти в разделе [Устойчивость соединения и логика повтора](/ef/ef6/fundamentals/connection-resiliency/retry-logic).</span><span class="sxs-lookup"><span data-stu-id="c6d75-685">More examples of using the Entity Framework retry mechanism can be found in [Connection Resiliency / Retry Logic](/ef/ef6/fundamentals/connection-resiliency/retry-logic).</span></span>

### <a name="more-information"></a><span data-ttu-id="c6d75-686">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="c6d75-686">More information</span></span>

- [<span data-ttu-id="c6d75-687">Руководство по обеспечению эластичности и производительности Базы данных SQL Azure</span><span class="sxs-lookup"><span data-stu-id="c6d75-687">Azure SQL Database performance and elasticity guide</span></span>](https://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core"></a><span data-ttu-id="c6d75-688">Использование Базы данных SQL с Entity Framework Core</span><span class="sxs-lookup"><span data-stu-id="c6d75-688">SQL Database using Entity Framework Core</span></span>

<span data-ttu-id="c6d75-689">Механизм [Entity Framework Core](/ef/core/) отслеживает взаимоотношения между объектами и позволяет разработчикам .NET использовать объекты на уровне доменов при работе с данными.</span><span class="sxs-lookup"><span data-stu-id="c6d75-689">[Entity Framework Core](/ef/core/) is an object-relational mapper that enables .NET Core developers to work with data using domain-specific objects.</span></span> <span data-ttu-id="c6d75-690">Это исключает необходимость в большинстве кодов доступа к данным, которые обычно требуется писать разработчикам.</span><span class="sxs-lookup"><span data-stu-id="c6d75-690">It eliminates the need for most of the data-access code that developers usually need to write.</span></span> <span data-ttu-id="c6d75-691">Эта версия Entity Framework полностью создана с нуля и по умолчанию не наследует возможности EF6.x.</span><span class="sxs-lookup"><span data-stu-id="c6d75-691">This version of Entity Framework was written from the ground up, and doesn't automatically inherit all the features from EF6.x.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="c6d75-692">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="c6d75-692">Retry mechanism</span></span>

<span data-ttu-id="c6d75-693">Поддержка повторных попыток при доступе к Базе данных SQL с использованием Entity Framework Core предоставляется через механизм, называемый [устойчивость подключения](/ef/core/miscellaneous/connection-resiliency).</span><span class="sxs-lookup"><span data-stu-id="c6d75-693">Retry support is provided when accessing SQL Database using Entity Framework Core through a mechanism called [connection resiliency](/ef/core/miscellaneous/connection-resiliency).</span></span> <span data-ttu-id="c6d75-694">Устойчивость подключения впервые появилась в версии EF Core 1.1.0.</span><span class="sxs-lookup"><span data-stu-id="c6d75-694">Connection resiliency was introduced in EF Core 1.1.0.</span></span>

<span data-ttu-id="c6d75-695">Основным объектом абстракции является интерфейс `IExecutionStrategy`.</span><span class="sxs-lookup"><span data-stu-id="c6d75-695">The primary abstraction is the `IExecutionStrategy` interface.</span></span> <span data-ttu-id="c6d75-696">Стратегия выполнения для SQL Server, включая SQL Azure, учитывает типы исключений, для которых возможны повторные попытки. Для нее предусмотрены стандартные рациональные параметры, определяющие максимальное число повторных попыток, пауз между ними и т. д.</span><span class="sxs-lookup"><span data-stu-id="c6d75-696">The execution strategy for SQL Server, including SQL Azure, is aware of the exception types that can be retried and has sensible defaults for maximum retries, delay between retries, and so on.</span></span>

### <a name="examples"></a><span data-ttu-id="c6d75-697">Примеры</span><span class="sxs-lookup"><span data-stu-id="c6d75-697">Examples</span></span>

<span data-ttu-id="c6d75-698">Следующий пример кода использует автоматические повторные попытки при настройке объекта DbContext, который представляет сеанс подключения к базе данных.</span><span class="sxs-lookup"><span data-stu-id="c6d75-698">The following code enables automatic retries when configuring the DbContext object, which represents a session with the database.</span></span>

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

<span data-ttu-id="c6d75-699">Следующий пример кода демонстрирует, как выполнять транзакцию с автоматическими повторными попытками с использованием стратегии выполнения.</span><span class="sxs-lookup"><span data-stu-id="c6d75-699">The following code shows how to execute a transaction with automatic retries, by using an execution strategy.</span></span> <span data-ttu-id="c6d75-700">Транзакция определяется в делегате.</span><span class="sxs-lookup"><span data-stu-id="c6d75-700">The transaction is defined in a delegate.</span></span> <span data-ttu-id="c6d75-701">Если происходит временный сбой, стратегия выполнения снова вызывает делегат.</span><span class="sxs-lookup"><span data-stu-id="c6d75-701">If a transient failure occurs, the execution strategy will invoke the delegate again.</span></span>

```csharp
using (var db = new BloggingContext())
{
    var strategy = db.Database.CreateExecutionStrategy();

    strategy.Execute(() =>
    {
        using (var transaction = db.Database.BeginTransaction())
        {
            db.Blogs.Add(new Blog { Url = "https://blogs.msdn.com/dotnet" });
            db.SaveChanges();

            db.Blogs.Add(new Blog { Url = "https://blogs.msdn.com/visualstudio" });
            db.SaveChanges();

            transaction.Commit();
        }
    });
}
```

## <a name="azure-storage"></a><span data-ttu-id="c6d75-702">Хранилище Azure</span><span class="sxs-lookup"><span data-stu-id="c6d75-702">Azure Storage</span></span>

<span data-ttu-id="c6d75-703">Службы хранилища Azure включают хранилища таблиц и больших двоичных объектов, файлы и очереди хранилища.</span><span class="sxs-lookup"><span data-stu-id="c6d75-703">Azure Storage services include table and blob storage, files, and storage queues.</span></span>

### <a name="retry-mechanism"></a><span data-ttu-id="c6d75-704">Механизм повтора</span><span class="sxs-lookup"><span data-stu-id="c6d75-704">Retry mechanism</span></span>

<span data-ttu-id="c6d75-705">Повторы выполняются на уровне отдельных операции REST и являются неотъемлемой частью реализации API клиента.</span><span class="sxs-lookup"><span data-stu-id="c6d75-705">Retries occur at the individual REST operation level and are an integral part of the client API implementation.</span></span> <span data-ttu-id="c6d75-706">Клиентский пакет SDK службы хранилища использует классы, реализующие [Интерфейс IExtendedRetryPolicy](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy).</span><span class="sxs-lookup"><span data-stu-id="c6d75-706">The client storage SDK uses classes that implement the [IExtendedRetryPolicy Interface](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy).</span></span>

<span data-ttu-id="c6d75-707">Существуют различные реализации этого интерфейса.</span><span class="sxs-lookup"><span data-stu-id="c6d75-707">There are different implementations of the interface.</span></span> <span data-ttu-id="c6d75-708">Клиенты службы хранилища могут выбирать политики специально для доступа к таблицам, BLOB-объектам и очередям.</span><span class="sxs-lookup"><span data-stu-id="c6d75-708">Storage clients can choose from policies specifically designed for accessing tables, blobs, and queues.</span></span> <span data-ttu-id="c6d75-709">Каждая реализация использует различные стратегии повтора, фактически определяя интервал повторных попыток и другие параметры.</span><span class="sxs-lookup"><span data-stu-id="c6d75-709">Each implementation uses a different retry strategy that essentially defines the retry interval and other details.</span></span>

<span data-ttu-id="c6d75-710">Встроенные классы обеспечивают поддержку для линейных (постоянная задержка) и экспоненциальных со случайными параметрами интервалов повтора.</span><span class="sxs-lookup"><span data-stu-id="c6d75-710">The built-in classes provide support for linear (constant delay) and exponential with randomization retry intervals.</span></span> <span data-ttu-id="c6d75-711">Здесь также нет политики повторов для случаев, когда другой процесс обрабатывает повторы на более высоком уровне.</span><span class="sxs-lookup"><span data-stu-id="c6d75-711">There is also a no retry policy for use when another process is handling retries at a higher level.</span></span> <span data-ttu-id="c6d75-712">Тем не менее если имеются особые требования, не предоставляемые встроенными классами, то можно реализовать собственные классы для обработки повторов.</span><span class="sxs-lookup"><span data-stu-id="c6d75-712">However, you can implement your own retry classes if you have specific requirements not provided by the built-in classes.</span></span>

<span data-ttu-id="c6d75-713">Альтернативные повторы переключаются между первичным и вторичным расположением службы хранилища при использовании геоизбыточного хранилища с доступом на чтение (RA-GRS) и если результатом запроса стала ошибка.</span><span class="sxs-lookup"><span data-stu-id="c6d75-713">Alternate retries switch between primary and secondary storage service location if you are using read access geo-redundant storage (RA-GRS) and the result of the request is a retryable error.</span></span> <span data-ttu-id="c6d75-714">Дополнительные сведения см. в статье [Варианты избыточности хранилища Azure](/azure/storage/common/storage-redundancy).</span><span class="sxs-lookup"><span data-stu-id="c6d75-714">See [Azure Storage Redundancy Options](/azure/storage/common/storage-redundancy) for more information.</span></span>

### <a name="policy-configuration"></a><span data-ttu-id="c6d75-715">Настройки политики</span><span class="sxs-lookup"><span data-stu-id="c6d75-715">Policy configuration</span></span>

<span data-ttu-id="c6d75-716">Политики повтора настраиваются программным способом.</span><span class="sxs-lookup"><span data-stu-id="c6d75-716">Retry policies are configured programmatically.</span></span> <span data-ttu-id="c6d75-717">Типичная процедура состоит в создании и заполнении экземпляров **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions** или **QueueRequestOptions**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-717">A typical procedure is to create and populate a **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions**, or **QueueRequestOptions** instance.</span></span>

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

<span data-ttu-id="c6d75-718">Затем можно задать параметры экземпляра запроса на клиенте, и все операции с клиентом будут использовать указанные параметры запроса.</span><span class="sxs-lookup"><span data-stu-id="c6d75-718">The request options instance can then be set on the client, and all operations with the client will use the specified request options.</span></span>

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

<span data-ttu-id="c6d75-719">Можно переопределить параметры запроса клиента путем передачи заполненного экземпляра класса параметров запроса в качестве параметра метода.</span><span class="sxs-lookup"><span data-stu-id="c6d75-719">You can override the client request options by passing a populated instance of the request options class as a parameter to operation methods.</span></span>

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

<span data-ttu-id="c6d75-720">Можно использовать экземпляр **OperationContext** , чтобы указать код, выполняемый при возникновении повтора и после завершения операции.</span><span class="sxs-lookup"><span data-stu-id="c6d75-720">You use an **OperationContext** instance to specify the code to execute when a retry occurs and when an operation has completed.</span></span> <span data-ttu-id="c6d75-721">Этот код может собирать сведения об операции для последующего использования в результатах телеметрии и журналах.</span><span class="sxs-lookup"><span data-stu-id="c6d75-721">This code can collect information about the operation for use in logs and telemetry.</span></span>

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

<span data-ttu-id="c6d75-722">Кроме определения, подходит ли сбой для повторных попыток, расширенные политики повтора возвращают объект **RetryContext**, который указывает число повторных попыток, результаты последнего запроса, произойдет ли следующая попытка в первичном или вторичном расположении (дополнительные сведения см. в таблице ниже).</span><span class="sxs-lookup"><span data-stu-id="c6d75-722">In addition to indicating whether a failure is suitable for retry, the extended retry policies return a **RetryContext** object that indicates the number of retries, the results of the last request, whether the next retry will happen in the primary or secondary location (see table below for details).</span></span> <span data-ttu-id="c6d75-723">Свойства объекта **RetryContext** позволяют принять решение о необходимости и времени выполнения повтора.</span><span class="sxs-lookup"><span data-stu-id="c6d75-723">The properties of the **RetryContext** object can be used to decide if and when to attempt a retry.</span></span> <span data-ttu-id="c6d75-724">Дополнительные сведения см. в статье [Метод IExtendedRetryPolicy.Evaluate](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate).</span><span class="sxs-lookup"><span data-stu-id="c6d75-724">For more details, see [IExtendedRetryPolicy.Evaluate Method](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate).</span></span>

<span data-ttu-id="c6d75-725">Следующие таблицы содержат значения по умолчанию для встроенной политики повторов.</span><span class="sxs-lookup"><span data-stu-id="c6d75-725">The following tables show the default settings for the built-in retry policies.</span></span>

<span data-ttu-id="c6d75-726">**Параметры запроса**:</span><span class="sxs-lookup"><span data-stu-id="c6d75-726">**Request options:**</span></span>

| <span data-ttu-id="c6d75-727">**Параметр**</span><span class="sxs-lookup"><span data-stu-id="c6d75-727">**Setting**</span></span> | <span data-ttu-id="c6d75-728">**Значение по умолчанию**</span><span class="sxs-lookup"><span data-stu-id="c6d75-728">**Default value**</span></span> | <span data-ttu-id="c6d75-729">**Значение**</span><span class="sxs-lookup"><span data-stu-id="c6d75-729">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="c6d75-730">MaximumExecutionTime</span><span class="sxs-lookup"><span data-stu-id="c6d75-730">MaximumExecutionTime</span></span> | <span data-ttu-id="c6d75-731">Нет</span><span class="sxs-lookup"><span data-stu-id="c6d75-731">None</span></span> | <span data-ttu-id="c6d75-732">Максимальное время выполнения запроса, включая все потенциальные повторные попытки.</span><span class="sxs-lookup"><span data-stu-id="c6d75-732">Maximum execution time for the request, including all potential retry attempts.</span></span> <span data-ttu-id="c6d75-733">Если оно не указано, то количество времени на выполнение запроса будет не ограничено.</span><span class="sxs-lookup"><span data-stu-id="c6d75-733">If it is not specified, then the amount of time that a request is permitted to take is unlimited.</span></span> <span data-ttu-id="c6d75-734">Другими словами, запрос может зависнуть.</span><span class="sxs-lookup"><span data-stu-id="c6d75-734">In other words, the request might hang.</span></span> |
| <span data-ttu-id="c6d75-735">ServerTimeOut</span><span class="sxs-lookup"><span data-stu-id="c6d75-735">ServerTimeout</span></span> | <span data-ttu-id="c6d75-736">Нет</span><span class="sxs-lookup"><span data-stu-id="c6d75-736">None</span></span> | <span data-ttu-id="c6d75-737">Интервал времени ожидания сервера для запроса (значение округляется до секунд).</span><span class="sxs-lookup"><span data-stu-id="c6d75-737">Server timeout interval for the request (value is rounded to seconds).</span></span> <span data-ttu-id="c6d75-738">Если не указан, он будет использовать значение по умолчанию для всех запросов к серверу.</span><span class="sxs-lookup"><span data-stu-id="c6d75-738">If not specified, it will use the default value for all requests to the server.</span></span> <span data-ttu-id="c6d75-739">Как правило, лучше всего опустить этот параметр, чтобы использовать значение сервера по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="c6d75-739">Usually, the best option is to omit this setting so that the server default is used.</span></span> |
| <span data-ttu-id="c6d75-740">LocationMode</span><span class="sxs-lookup"><span data-stu-id="c6d75-740">LocationMode</span></span> | <span data-ttu-id="c6d75-741">Нет</span><span class="sxs-lookup"><span data-stu-id="c6d75-741">None</span></span> | <span data-ttu-id="c6d75-742">Если учетная запись хранения создается с вариантом репликации на географически избыточном хранилище с доступом для чтения (RA-GRS), то можно использовать режим расположения, чтобы указать расположение, где запрос должен быть получен.</span><span class="sxs-lookup"><span data-stu-id="c6d75-742">If the storage account is created with the Read access geo-redundant storage (RA-GRS) replication option, you can use the location mode to indicate which location should receive the request.</span></span> <span data-ttu-id="c6d75-743">Например, если указано **PrimaryThenSecondary**, запросы всегда будут отправляться сначала в основное расположение.</span><span class="sxs-lookup"><span data-stu-id="c6d75-743">For example, if **PrimaryThenSecondary** is specified, requests are always sent to the primary location first.</span></span> <span data-ttu-id="c6d75-744">Если запрос завершается ошибкой, он направляется во вторичное расположение.</span><span class="sxs-lookup"><span data-stu-id="c6d75-744">If a request fails, it is sent to the secondary location.</span></span> |
| <span data-ttu-id="c6d75-745">политика RetryPolicy</span><span class="sxs-lookup"><span data-stu-id="c6d75-745">RetryPolicy</span></span> | <span data-ttu-id="c6d75-746">ExponentialPolicy</span><span class="sxs-lookup"><span data-stu-id="c6d75-746">ExponentialPolicy</span></span> | <span data-ttu-id="c6d75-747">Сведения о каждом параметре смотрите далее.</span><span class="sxs-lookup"><span data-stu-id="c6d75-747">See below for details of each option.</span></span> |

<span data-ttu-id="c6d75-748">**Экспоненциальная политика**:</span><span class="sxs-lookup"><span data-stu-id="c6d75-748">**Exponential policy:**</span></span>

| <span data-ttu-id="c6d75-749">**Параметр**</span><span class="sxs-lookup"><span data-stu-id="c6d75-749">**Setting**</span></span> | <span data-ttu-id="c6d75-750">**Значение по умолчанию**</span><span class="sxs-lookup"><span data-stu-id="c6d75-750">**Default value**</span></span> | <span data-ttu-id="c6d75-751">**Значение**</span><span class="sxs-lookup"><span data-stu-id="c6d75-751">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="c6d75-752">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="c6d75-752">maxAttempt</span></span> | <span data-ttu-id="c6d75-753">3</span><span class="sxs-lookup"><span data-stu-id="c6d75-753">3</span></span> | <span data-ttu-id="c6d75-754">Количество повторных попыток.</span><span class="sxs-lookup"><span data-stu-id="c6d75-754">Number of retry attempts.</span></span> |
| <span data-ttu-id="c6d75-755">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="c6d75-755">deltaBackoff</span></span> | <span data-ttu-id="c6d75-756">4 секунды</span><span class="sxs-lookup"><span data-stu-id="c6d75-756">4 seconds</span></span> | <span data-ttu-id="c6d75-757">Интервал отсрочки между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="c6d75-757">Back-off interval between retries.</span></span> <span data-ttu-id="c6d75-758">Кратное timespan, включая элемент случая, который будет использоваться для последующих попыток.</span><span class="sxs-lookup"><span data-stu-id="c6d75-758">Multiples of this timespan, including a random element, will be used for subsequent retry attempts.</span></span> |
| <span data-ttu-id="c6d75-759">MinBackoff</span><span class="sxs-lookup"><span data-stu-id="c6d75-759">MinBackoff</span></span> | <span data-ttu-id="c6d75-760">3 секунды</span><span class="sxs-lookup"><span data-stu-id="c6d75-760">3 seconds</span></span> | <span data-ttu-id="c6d75-761">Добавить ко всем интервалам повтора, вычисленным по deltaBackoff.</span><span class="sxs-lookup"><span data-stu-id="c6d75-761">Added to all retry intervals computed from deltaBackoff.</span></span> <span data-ttu-id="c6d75-762">Это значение не может быть изменено.</span><span class="sxs-lookup"><span data-stu-id="c6d75-762">This value cannot be changed.</span></span>
| <span data-ttu-id="c6d75-763">MaxBackoff</span><span class="sxs-lookup"><span data-stu-id="c6d75-763">MaxBackoff</span></span> | <span data-ttu-id="c6d75-764">120 секунд</span><span class="sxs-lookup"><span data-stu-id="c6d75-764">120 seconds</span></span> | <span data-ttu-id="c6d75-765">MaxBackoff используется в том случае, если расчетный интервал повторных попыток оказывается больше, чем MaxBackoff.</span><span class="sxs-lookup"><span data-stu-id="c6d75-765">MaxBackoff is used if the computed retry interval is greater than MaxBackoff.</span></span> <span data-ttu-id="c6d75-766">Это значение не может быть изменено.</span><span class="sxs-lookup"><span data-stu-id="c6d75-766">This value cannot be changed.</span></span> |

<span data-ttu-id="c6d75-767">**Линейная политика**:</span><span class="sxs-lookup"><span data-stu-id="c6d75-767">**Linear policy:**</span></span>

| <span data-ttu-id="c6d75-768">**Параметр**</span><span class="sxs-lookup"><span data-stu-id="c6d75-768">**Setting**</span></span> | <span data-ttu-id="c6d75-769">**Значение по умолчанию**</span><span class="sxs-lookup"><span data-stu-id="c6d75-769">**Default value**</span></span> | <span data-ttu-id="c6d75-770">**Значение**</span><span class="sxs-lookup"><span data-stu-id="c6d75-770">**Meaning**</span></span> |
| --- | --- | --- |
| <span data-ttu-id="c6d75-771">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="c6d75-771">maxAttempt</span></span> | <span data-ttu-id="c6d75-772">3</span><span class="sxs-lookup"><span data-stu-id="c6d75-772">3</span></span> | <span data-ttu-id="c6d75-773">Количество повторных попыток.</span><span class="sxs-lookup"><span data-stu-id="c6d75-773">Number of retry attempts.</span></span> |
| <span data-ttu-id="c6d75-774">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="c6d75-774">deltaBackoff</span></span> | <span data-ttu-id="c6d75-775">30 секунд</span><span class="sxs-lookup"><span data-stu-id="c6d75-775">30 seconds</span></span> | <span data-ttu-id="c6d75-776">Интервал отсрочки между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="c6d75-776">Back-off interval between retries.</span></span> |

### <a name="retry-usage-guidance"></a><span data-ttu-id="c6d75-777">Руководство по использованию механизма повторов</span><span class="sxs-lookup"><span data-stu-id="c6d75-777">Retry usage guidance</span></span>

<span data-ttu-id="c6d75-778">При доступе к службам хранилища Azure с помощью API клиентской службы хранилища, придерживайтесь следующих рекомендаций.</span><span class="sxs-lookup"><span data-stu-id="c6d75-778">Consider the following guidelines when accessing Azure storage services using the storage client API:</span></span>

- <span data-ttu-id="c6d75-779">Используйте встроенные политики повтора из пространства имен Microsoft.WindowsAzure.Storage.RetryPolicies, которые соответствуют требованиям.</span><span class="sxs-lookup"><span data-stu-id="c6d75-779">Use the built-in retry policies from the Microsoft.WindowsAzure.Storage.RetryPolicies namespace where they are appropriate for your requirements.</span></span> <span data-ttu-id="c6d75-780">В большинстве случаев использования этих политик будет достаточно.</span><span class="sxs-lookup"><span data-stu-id="c6d75-780">In most cases, these policies will be sufficient.</span></span>

- <span data-ttu-id="c6d75-781">Используйте политику **ExponentialRetry** для пакетных операций, фоновых задач или неинтерактивных сценариев.</span><span class="sxs-lookup"><span data-stu-id="c6d75-781">Use the **ExponentialRetry** policy in batch operations, background tasks, or non-interactive scenarios.</span></span> <span data-ttu-id="c6d75-782">В этих сценариях обычно имеется больше времени для восстановления службы &mdash; с таким образом повышается вероятность операции.</span><span class="sxs-lookup"><span data-stu-id="c6d75-782">In these scenarios, you can typically allow more time for the service to recover &mdash; with a consequently increased chance of the operation eventually succeeding.</span></span>

- <span data-ttu-id="c6d75-783">Следует указать свойства **MaximumExecutionTime** параметра **RequestOptions** для ограничения общего времени выполнения, а также принять во внимание тип и размер операции при выборе значения времени ожидания.</span><span class="sxs-lookup"><span data-stu-id="c6d75-783">Consider specifying the **MaximumExecutionTime** property of the **RequestOptions** parameter to limit the total execution time, but take into account the type and size of the operation when choosing a timeout value.</span></span>

- <span data-ttu-id="c6d75-784">Если необходимо реализовать пользовательский повтор, избегайте создания оболочек клиентских классов хранения.</span><span class="sxs-lookup"><span data-stu-id="c6d75-784">If you need to implement a custom retry, avoid creating wrappers around the storage client classes.</span></span> <span data-ttu-id="c6d75-785">Вместо этого используйте возможности для расширения существующих политик с помощью интерфейса **IExtendedRetryPolicy** .</span><span class="sxs-lookup"><span data-stu-id="c6d75-785">Instead, use the capabilities to extend the existing policies through the **IExtendedRetryPolicy** interface.</span></span>

- <span data-ttu-id="c6d75-786">Если вы используете геоизбыточное хранилище с доступом на чтение (RA-GRS), можно использовать **LocationMode** для указания, что повторные попытки должны будут выполняться для вторичной копии хранилища в случае сбоя доступа к первичному хранилищу.</span><span class="sxs-lookup"><span data-stu-id="c6d75-786">If you are using read access geo-redundant storage (RA-GRS) you can use the **LocationMode** to specify that retry attempts will access the secondary read-only copy of the store should the primary access fail.</span></span> <span data-ttu-id="c6d75-787">Однако при использовании этого параметра необходимо убедиться, что ваше приложение может успешно работать с данными, которые могут устаревать, если процедура репликации с основным хранилищем еще не завершена.</span><span class="sxs-lookup"><span data-stu-id="c6d75-787">However, when using this option you must ensure that your application can work successfully with data that may be stale if the replication from the primary store has not yet completed.</span></span>

<span data-ttu-id="c6d75-788">Начните с использования следующих параметров для операции повтора.</span><span class="sxs-lookup"><span data-stu-id="c6d75-788">Consider starting with following settings for retrying operations.</span></span> <span data-ttu-id="c6d75-789">Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.</span><span class="sxs-lookup"><span data-stu-id="c6d75-789">These are general purpose settings, and you should monitor the operations and fine tune the values to suit your own scenario.</span></span>

| <span data-ttu-id="c6d75-790">**Контекст**</span><span class="sxs-lookup"><span data-stu-id="c6d75-790">**Context**</span></span> | <span data-ttu-id="c6d75-791">**Максимальная задержка<br />примера целевого E2E**</span><span class="sxs-lookup"><span data-stu-id="c6d75-791">**Sample target E2E<br />max latency**</span></span> | <span data-ttu-id="c6d75-792">**Политика повтора**</span><span class="sxs-lookup"><span data-stu-id="c6d75-792">**Retry policy**</span></span> | <span data-ttu-id="c6d75-793">**Параметры**</span><span class="sxs-lookup"><span data-stu-id="c6d75-793">**Settings**</span></span> | <span data-ttu-id="c6d75-794">**Значения**</span><span class="sxs-lookup"><span data-stu-id="c6d75-794">**Values**</span></span> | <span data-ttu-id="c6d75-795">**Принцип работы**</span><span class="sxs-lookup"><span data-stu-id="c6d75-795">**How it works**</span></span> |
| --- | --- | --- | --- | --- | --- |
| <span data-ttu-id="c6d75-796">Интерактивный, пользовательский интерфейс</span><span class="sxs-lookup"><span data-stu-id="c6d75-796">Interactive, UI,</span></span><br /><span data-ttu-id="c6d75-797">или передний план</span><span class="sxs-lookup"><span data-stu-id="c6d75-797">or foreground</span></span> |<span data-ttu-id="c6d75-798">2 секунды</span><span class="sxs-lookup"><span data-stu-id="c6d75-798">2 seconds</span></span> |<span data-ttu-id="c6d75-799">Линейная</span><span class="sxs-lookup"><span data-stu-id="c6d75-799">Linear</span></span> |<span data-ttu-id="c6d75-800">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="c6d75-800">maxAttempt</span></span><br /><span data-ttu-id="c6d75-801">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="c6d75-801">deltaBackoff</span></span> |<span data-ttu-id="c6d75-802">3</span><span class="sxs-lookup"><span data-stu-id="c6d75-802">3</span></span><br /><span data-ttu-id="c6d75-803">500 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-803">500 ms</span></span> |<span data-ttu-id="c6d75-804">Попытка 1 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-804">Attempt 1 - delay 500 ms</span></span><br /><span data-ttu-id="c6d75-805">Попытка 2 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-805">Attempt 2 - delay 500 ms</span></span><br /><span data-ttu-id="c6d75-806">попытка 3 — задержка 500 мс</span><span class="sxs-lookup"><span data-stu-id="c6d75-806">Attempt 3 - delay 500 ms</span></span> |
| <span data-ttu-id="c6d75-807">Фоновый</span><span class="sxs-lookup"><span data-stu-id="c6d75-807">Background</span></span><br /><span data-ttu-id="c6d75-808">или пакетный</span><span class="sxs-lookup"><span data-stu-id="c6d75-808">or batch</span></span> |<span data-ttu-id="c6d75-809">30 секунд</span><span class="sxs-lookup"><span data-stu-id="c6d75-809">30 seconds</span></span> |<span data-ttu-id="c6d75-810">Экспоненциальная</span><span class="sxs-lookup"><span data-stu-id="c6d75-810">Exponential</span></span> |<span data-ttu-id="c6d75-811">maxAttempt</span><span class="sxs-lookup"><span data-stu-id="c6d75-811">maxAttempt</span></span><br /><span data-ttu-id="c6d75-812">deltaBackoff</span><span class="sxs-lookup"><span data-stu-id="c6d75-812">deltaBackoff</span></span> |<span data-ttu-id="c6d75-813">5</span><span class="sxs-lookup"><span data-stu-id="c6d75-813">5</span></span><br /><span data-ttu-id="c6d75-814">4 секунды</span><span class="sxs-lookup"><span data-stu-id="c6d75-814">4 seconds</span></span> |<span data-ttu-id="c6d75-815">Попытка 1 — задержка ~ 3 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-815">Attempt 1 - delay ~3 sec</span></span><br /><span data-ttu-id="c6d75-816">Попытка 2 — задержка 7 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-816">Attempt 2 - delay ~7 sec</span></span><br /><span data-ttu-id="c6d75-817">Попытка 3 — задержка ~ 15 с</span><span class="sxs-lookup"><span data-stu-id="c6d75-817">Attempt 3 - delay ~15 sec</span></span> |

### <a name="telemetry"></a><span data-ttu-id="c6d75-818">Телеметрия</span><span class="sxs-lookup"><span data-stu-id="c6d75-818">Telemetry</span></span>

<span data-ttu-id="c6d75-819">Количество попыток входа регистрируется в **TraceSource**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-819">Retry attempts are logged to a **TraceSource**.</span></span> <span data-ttu-id="c6d75-820">Необходимо настроить **TraceListener** для сбора данных о событиях и записи этих данных в журнал с подходящим назначением.</span><span class="sxs-lookup"><span data-stu-id="c6d75-820">You must configure a **TraceListener** to capture the events and write them to a suitable destination log.</span></span> <span data-ttu-id="c6d75-821">Можно использовать **TextWriterTraceListener** или **XmlWriterTraceListener** для записи данных в файл журнала, **EventLogTraceListener** для записи в журнал событий Windows или **EventProviderTraceListener** для записи данных трассировки в подсистеме трассировки событий Windows.</span><span class="sxs-lookup"><span data-stu-id="c6d75-821">You can use the **TextWriterTraceListener** or **XmlWriterTraceListener** to write the data to a log file, the **EventLogTraceListener** to write to the Windows Event Log, or the **EventProviderTraceListener** to write trace data to the ETW subsystem.</span></span> <span data-ttu-id="c6d75-822">Можно также настроить автоматическую очистку буфера и уровень детализации регистрируемых событий (например, Ошибка, Предупреждение, Информационное событие и Подробное протоколирование).</span><span class="sxs-lookup"><span data-stu-id="c6d75-822">You can also configure auto-flushing of the buffer, and the verbosity of events that will be logged (for example, Error, Warning, Informational, and Verbose).</span></span> <span data-ttu-id="c6d75-823">Дополнительные сведения см. в статье [Вход в клиентскую библиотеку хранилища .NET на стороне клиента](/rest/api/storageservices/Client-side-Logging-with-the-.NET-Storage-Client-Library).</span><span class="sxs-lookup"><span data-stu-id="c6d75-823">For more information, see [Client-side Logging with the .NET Storage Client Library](/rest/api/storageservices/Client-side-Logging-with-the-.NET-Storage-Client-Library).</span></span>

<span data-ttu-id="c6d75-824">Операции могут получать экземпляр **OperationContext**, предоставляющий событие **Повтор**, которое может использоваться для присоединения телеметрии пользовательской логики.</span><span class="sxs-lookup"><span data-stu-id="c6d75-824">Operations can receive an **OperationContext** instance, which exposes a **Retrying** event that can be used to attach custom telemetry logic.</span></span> <span data-ttu-id="c6d75-825">Дополнительные сведения см. в статье [Событие OperationContext.Retrying](/dotnet/api/microsoft.windowsazure.storage.operationcontext.retrying).</span><span class="sxs-lookup"><span data-stu-id="c6d75-825">For more information, see [OperationContext.Retrying Event](/dotnet/api/microsoft.windowsazure.storage.operationcontext.retrying).</span></span>

### <a name="examples"></a><span data-ttu-id="c6d75-826">Примеры</span><span class="sxs-lookup"><span data-stu-id="c6d75-826">Examples</span></span>

<span data-ttu-id="c6d75-827">В следующем примере кода показано, как создать два экземпляра **TableRequestOptions** с различными параметрами, один для интерактивных запросов и один для фоновых запросов.</span><span class="sxs-lookup"><span data-stu-id="c6d75-827">The following code example shows how to create two **TableRequestOptions** instances with different retry settings; one for interactive requests and one for background requests.</span></span> <span data-ttu-id="c6d75-828">Пример затем устанавливает эти две политики повторов на клиентском компьютере, чтобы они применялись для всех запросов, а также задает интерактивную стратегию на конкретный запрос, чтобы он переопределял параметры по умолчанию, применяемые к клиенту.</span><span class="sxs-lookup"><span data-stu-id="c6d75-828">The example then sets these two retry policies on the client so that they apply for all requests, and also sets the interactive strategy on a specific request so that it overrides the default settings applied to the client.</span></span>

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

### <a name="more-information"></a><span data-ttu-id="c6d75-829">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="c6d75-829">More information</span></span>

- [<span data-ttu-id="c6d75-830">Рекомендации по настройке политики повтора для клиентской библиотеки службы хранилища Azure</span><span class="sxs-lookup"><span data-stu-id="c6d75-830">Azure Storage client Library retry policy recommendations</span></span>](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)

- [<span data-ttu-id="c6d75-831">Клиентская библиотека службы хранилища 2.0: реализация политики повтора</span><span class="sxs-lookup"><span data-stu-id="c6d75-831">Storage Client Library 2.0 – Implementing retry policies</span></span>](https://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="general-rest-and-retry-guidelines"></a><span data-ttu-id="c6d75-832">Общие рекомендации по использованию REST и механизма повторов</span><span class="sxs-lookup"><span data-stu-id="c6d75-832">General REST and retry guidelines</span></span>

<span data-ttu-id="c6d75-833">При обращении к службам Azure или службам сторонних производителей учитывайте следующее.</span><span class="sxs-lookup"><span data-stu-id="c6d75-833">Consider the following when accessing Azure or third party services:</span></span>

- <span data-ttu-id="c6d75-834">Используйте систематический подход к управлению механизмом повторов, возможно в виде повторно используемого кода, что таким образом дает возможность применения согласованной методологии ко всем клиентам и всем решениям.</span><span class="sxs-lookup"><span data-stu-id="c6d75-834">Use a systematic approach to managing retries, perhaps as reusable code, so that you can apply a consistent methodology across all clients and all solutions.</span></span>

- <span data-ttu-id="c6d75-835">Если целевая служба или клиент не имеет встроенного механизма повторов, рекомендуется использовать платформы повторов, например [Polly][polly], для управления механизмом повторов.</span><span class="sxs-lookup"><span data-stu-id="c6d75-835">Consider using a retry framework such as [Polly][polly] to manage retries if the target service or client has no built-in retry mechanism.</span></span> <span data-ttu-id="c6d75-836">Это поможет реализовать согласованный механизм повтора и может предоставить подходящую стратегию повторов по умолчанию для целевой службы.</span><span class="sxs-lookup"><span data-stu-id="c6d75-836">This will help you implement a consistent retry behavior, and it may provide a suitable default retry strategy for the target service.</span></span> <span data-ttu-id="c6d75-837">Тем не менее необходимо создать пользовательский код механизма повторов для служб, имеющих нестандартное поведение, которые не полагаются на исключения для определения временных сбоев, или если вы хотите использовать ответ **Retry-Response** для управления поведением механизма повторов.</span><span class="sxs-lookup"><span data-stu-id="c6d75-837">However, you may need to create custom retry code for services that have non-standard behavior, that do not rely on exceptions to indicate transient failures, or if you want to use a **Retry-Response** reply to manage retry behavior.</span></span>

- <span data-ttu-id="c6d75-838">Логика обнаружения временных сбоев будет зависеть от фактического клиентского API, который используется для вызова REST.</span><span class="sxs-lookup"><span data-stu-id="c6d75-838">The transient detection logic will depend on the actual client API you use to invoke the REST calls.</span></span> <span data-ttu-id="c6d75-839">Некоторые клиенты, например новый класс **HttpClient** , не создают исключения для запросов, завершенных с неуспешным кодом состояния HTTP.</span><span class="sxs-lookup"><span data-stu-id="c6d75-839">Some clients, such as the newer **HttpClient** class, will not throw exceptions for completed requests with a non-success HTTP status code.</span></span>

- <span data-ttu-id="c6d75-840">Код состояния HTTP, возвращаемый службой, позволяет указать, является ли ошибка временной.</span><span class="sxs-lookup"><span data-stu-id="c6d75-840">The HTTP status code returned from the service can help to indicate whether the failure is transient.</span></span> <span data-ttu-id="c6d75-841">Необходимо изучить исключения, генерируемые клиентом, или механизм повторов для доступа к коду состояния для определения эквивалентного типа исключения.</span><span class="sxs-lookup"><span data-stu-id="c6d75-841">You may need to examine the exceptions generated by a client or the retry framework to access the status code or to determine the equivalent exception type.</span></span> <span data-ttu-id="c6d75-842">Следующие коды HTTP обычно показывают, что повтор может быть выполнен.</span><span class="sxs-lookup"><span data-stu-id="c6d75-842">The following HTTP codes typically indicate that a retry is appropriate:</span></span>
  
  - <span data-ttu-id="c6d75-843">408 — Истекло время ожидания запроса</span><span class="sxs-lookup"><span data-stu-id="c6d75-843">408 Request Timeout</span></span>
  - <span data-ttu-id="c6d75-844">429 — слишком много запросов</span><span class="sxs-lookup"><span data-stu-id="c6d75-844">429 Too Many Requests</span></span>
  - <span data-ttu-id="c6d75-845">500 — Внутренняя ошибка сервера</span><span class="sxs-lookup"><span data-stu-id="c6d75-845">500 Internal Server Error</span></span>
  - <span data-ttu-id="c6d75-846">502 — Недопустимый шлюз</span><span class="sxs-lookup"><span data-stu-id="c6d75-846">502 Bad Gateway</span></span>
  - <span data-ttu-id="c6d75-847">503 — Служба недоступна</span><span class="sxs-lookup"><span data-stu-id="c6d75-847">503 Service Unavailable</span></span>
  - <span data-ttu-id="c6d75-848">504 — Истекло время ожидания шлюза</span><span class="sxs-lookup"><span data-stu-id="c6d75-848">504 Gateway Timeout</span></span>

- <span data-ttu-id="c6d75-849">Если логика повторов основана на исключениях, следующие параметры обычно указывают, что произошел временный сбой где не удалось создать подключение.</span><span class="sxs-lookup"><span data-stu-id="c6d75-849">If you base your retry logic on exceptions, the following typically indicate a transient failure where no connection could be established:</span></span>

  - <span data-ttu-id="c6d75-850">WebExceptionStatus.ConnectionClosed</span><span class="sxs-lookup"><span data-stu-id="c6d75-850">WebExceptionStatus.ConnectionClosed</span></span>
  - <span data-ttu-id="c6d75-851">WebExceptionStatus.ConnectFailure</span><span class="sxs-lookup"><span data-stu-id="c6d75-851">WebExceptionStatus.ConnectFailure</span></span>
  - <span data-ttu-id="c6d75-852">WebExceptionStatus.Timeout</span><span class="sxs-lookup"><span data-stu-id="c6d75-852">WebExceptionStatus.Timeout</span></span>
  - <span data-ttu-id="c6d75-853">WebExceptionStatus.RequestCanceled</span><span class="sxs-lookup"><span data-stu-id="c6d75-853">WebExceptionStatus.RequestCanceled</span></span>

- <span data-ttu-id="c6d75-854">Если служба недоступна, она может передать прогнозируемое значение задержки перед следующей повторной попыткой в заголовке ответа **Retry-After** или другом пользовательском заголовке.</span><span class="sxs-lookup"><span data-stu-id="c6d75-854">In the case of a service unavailable status, the service might indicate the appropriate delay before retrying in the **Retry-After** response header or a different custom header.</span></span> <span data-ttu-id="c6d75-855">Службы также отправляют дополнительную информацию в виде пользовательских заголовков или внедренной в содержимое ответа.</span><span class="sxs-lookup"><span data-stu-id="c6d75-855">Services might also send additional information as custom headers, or embedded in the content of the response.</span></span>

- <span data-ttu-id="c6d75-856">Не выполняйте повторы для кодов состояния, представляющих клиентские ошибки (ошибки в диапазоне 4xx) за исключением ошибки «408 — Истекло время ожидания запроса».</span><span class="sxs-lookup"><span data-stu-id="c6d75-856">Do not retry for status codes representing client errors (errors in the 4xx range) except for a 408 Request Timeout.</span></span>

- <span data-ttu-id="c6d75-857">Тщательно протестируйте свои стратегии и механизмы повторов для различных условий, например, указав другую сеть и с различными нагрузками системы.</span><span class="sxs-lookup"><span data-stu-id="c6d75-857">Thoroughly test your retry strategies and mechanisms under a range of conditions, such as different network states and varying system loadings.</span></span>

### <a name="retry-strategies"></a><span data-ttu-id="c6d75-858">Стратегия повторов</span><span class="sxs-lookup"><span data-stu-id="c6d75-858">Retry strategies</span></span>

<span data-ttu-id="c6d75-859">Ниже приведены типичные виды интервалов стратегий повтора.</span><span class="sxs-lookup"><span data-stu-id="c6d75-859">The following are the typical types of retry strategy intervals:</span></span>

- <span data-ttu-id="c6d75-860">**Экспоненциальная**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-860">**Exponential**.</span></span> <span data-ttu-id="c6d75-861">Политика повторов, основанная на выполнении указанного числа повторных попыток и использующая случайный экспоненциальный пассивный метод для определения интервала между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="c6d75-861">A retry policy that performs a specified number of retries, using a randomized exponential back off approach to determine the interval between retries.</span></span> <span data-ttu-id="c6d75-862">Например: </span><span class="sxs-lookup"><span data-stu-id="c6d75-862">For example:</span></span>

    ```csharp
    var random = new Random();

    var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
    var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                    this.maxBackoff.TotalMilliseconds);
    retryInterval = TimeSpan.FromMilliseconds(interval);
    ```

- <span data-ttu-id="c6d75-863">**Добавочная**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-863">**Incremental**.</span></span> <span data-ttu-id="c6d75-864">Стратегия повторов с указанным числом повторных попыток и добавочным интервалом между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="c6d75-864">A retry strategy with a specified number of retry attempts and an incremental time interval between retries.</span></span> <span data-ttu-id="c6d75-865">Например: </span><span class="sxs-lookup"><span data-stu-id="c6d75-865">For example:</span></span>

    ```csharp
    retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                    (this.increment.TotalMilliseconds * currentRetryCount));
    ```

- <span data-ttu-id="c6d75-866">**Линейный повтор**.</span><span class="sxs-lookup"><span data-stu-id="c6d75-866">**LinearRetry**.</span></span> <span data-ttu-id="c6d75-867">Политика повторов, выполняющая указанное число повторных попыток и использующая указанный фиксированный интервал времени между повторными попытками.</span><span class="sxs-lookup"><span data-stu-id="c6d75-867">A retry policy that performs a specified number of retries, using a specified fixed time interval between retries.</span></span> <span data-ttu-id="c6d75-868">Например: </span><span class="sxs-lookup"><span data-stu-id="c6d75-868">For example:</span></span>

    ```csharp
    retryInterval = this.deltaBackoff;
    ```

### <a name="transient-fault-handling-with-polly"></a><span data-ttu-id="c6d75-869">Обработка временного сбоя с помощью Polly</span><span class="sxs-lookup"><span data-stu-id="c6d75-869">Transient fault handling with Polly</span></span>

<span data-ttu-id="c6d75-870">[Polly] [ polly] — это библиотека программным образом обрабатывать повторные попытки и [Размыкатель цепи](../patterns/circuit-breaker.md) стратегии.</span><span class="sxs-lookup"><span data-stu-id="c6d75-870">[Polly][polly] is a library to programmatically handle retries and [circuit breaker](../patterns/circuit-breaker.md) strategies.</span></span> <span data-ttu-id="c6d75-871">Проект Polly является частью [.NET Foundation][dotnet-foundation].</span><span class="sxs-lookup"><span data-stu-id="c6d75-871">The Polly project is a member of the [.NET Foundation][dotnet-foundation].</span></span> <span data-ttu-id="c6d75-872">Применение Polly будет неплохим вариантом при работе со службами, клиент которых не имеет встроенного механизма повторных попыток. Вам не придется самостоятельно создавать код для повторных попыток, который иногда очень трудно реализовать правильно.</span><span class="sxs-lookup"><span data-stu-id="c6d75-872">For services where the client does not natively support retries, Polly is a valid alternative and avoids the need to write custom retry code, which can be hard to implement correctly.</span></span> <span data-ttu-id="c6d75-873">Polly также поддерживает трассировку возникающих ошибок, что позволяет вести журнал повторных попыток.</span><span class="sxs-lookup"><span data-stu-id="c6d75-873">Polly also provides a way to trace errors when they occur, so that you can log retries.</span></span>

### <a name="more-information"></a><span data-ttu-id="c6d75-874">Дополнительные сведения</span><span class="sxs-lookup"><span data-stu-id="c6d75-874">More information</span></span>

- [<span data-ttu-id="c6d75-875">Устойчивость подключения</span><span class="sxs-lookup"><span data-stu-id="c6d75-875">Connection resiliency</span></span>](/ef/core/miscellaneous/connection-resiliency)
- [<span data-ttu-id="c6d75-876">Точки данных для EF Core 1.1</span><span class="sxs-lookup"><span data-stu-id="c6d75-876">Data Points - EF Core 1.1</span></span>](https://msdn.microsoft.com/magazine/mt745093.aspx)

<!-- links -->

[adal]: /azure/active-directory/develop/active-directory-authentication-libraries
[autorest]: https://github.com/Azure/autorest/tree/master/docs
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[dotnet-foundation]: https://dotnetfoundation.org/
[polly]: http://www.thepollyproject.org
[redis-cache-troubleshoot]: /azure/redis-cache/cache-how-to-troubleshoot
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx
