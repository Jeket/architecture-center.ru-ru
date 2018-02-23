---
title: "Выбор хранилища данных OLAP"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: f3041b95696c9408a2c9ab747fe1ec3041db0743
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-an-olap-data-store-in-azure"></a><span data-ttu-id="4b41f-102">Выбор хранилища данных OLAP в Azure</span><span class="sxs-lookup"><span data-stu-id="4b41f-102">Choosing an OLAP data store in Azure</span></span>

<span data-ttu-id="4b41f-103">Оперативная аналитическая обработка (OLAP) — это технология, которая упорядочивает большие коммерческие базы данных и поддерживает сложный анализ.</span><span class="sxs-lookup"><span data-stu-id="4b41f-103">Online analytical processing (OLAP) is a technology that organizes large business databases and supports complex analysis.</span></span> <span data-ttu-id="4b41f-104">В этой статье сравниваются варианты решений OLAP в Azure.</span><span class="sxs-lookup"><span data-stu-id="4b41f-104">This topic compares the options for OLAP solutions in Azure.</span></span>

> [!NOTE]
> <span data-ttu-id="4b41f-105">Дополнительные сведения об использовании хранилища данных OLAP см. в статье [об оперативной аналитической обработке (OLAP)](../scenarios/online-analytical-processing.md).</span><span class="sxs-lookup"><span data-stu-id="4b41f-105">For more information about when to use an OLAP data store, see [Online analytical processing](../scenarios/online-analytical-processing.md).</span></span>

## <a name="what-are-your-options-when-choosing-an-olap-data-store"></a><span data-ttu-id="4b41f-106">Варианты при выборе хранилища данных OLAP</span><span class="sxs-lookup"><span data-stu-id="4b41f-106">What are your options when choosing an OLAP data store?</span></span>

<span data-ttu-id="4b41f-107">Все следующие хранилища данных в Azure будут соответствовать основным требованиям для OLAP:</span><span class="sxs-lookup"><span data-stu-id="4b41f-107">In Azure, all of the following data stores will meet the core requirements for OLAP:</span></span>

- <span data-ttu-id="4b41f-108">[SQL Server с индексами columnstore](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics);</span><span class="sxs-lookup"><span data-stu-id="4b41f-108">[SQL Server with Columnstore indexes](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)</span></span>
- <span data-ttu-id="4b41f-109">[Azure Analysis Services](/azure/analysis-services/analysis-services-overview);</span><span class="sxs-lookup"><span data-stu-id="4b41f-109">[Azure Analysis Services](/azure/analysis-services/analysis-services-overview)</span></span>
- [<span data-ttu-id="4b41f-110">SQL Server Analysis Services (SSAS)</span><span class="sxs-lookup"><span data-stu-id="4b41f-110">SQL Server Analysis Services (SSAS)</span></span>](/sql/analysis-services/analysis-services)

<span data-ttu-id="4b41f-111">В службах SQL Server Analysis Services (SSAS) предлагаются возможности OLAP и интеллектуального анализа данных для приложений бизнес-аналитики.</span><span class="sxs-lookup"><span data-stu-id="4b41f-111">SQL Server Analysis Services (SSAS) offers OLAP and data mining functionality for business intelligence applications.</span></span> <span data-ttu-id="4b41f-112">Вы можете установить службы SSAS на локальных серверах или разместить их на виртуальной машине в Azure.</span><span class="sxs-lookup"><span data-stu-id="4b41f-112">You can either install SSAS on local servers, or host within a virtual machine in Azure.</span></span> <span data-ttu-id="4b41f-113">Azure Analysis Services — это полностью управляемая служба, которая предоставляет те же основные функции, что и SSAS.</span><span class="sxs-lookup"><span data-stu-id="4b41f-113">Azure Analysis Services is a fully managed service that provides the same major features as SSAS.</span></span> <span data-ttu-id="4b41f-114">Службы Azure Analysis Services поддерживают подключение к различным облачным и локальным корпоративным [источникам данных](/azure/analysis-services/analysis-services-datasource).</span><span class="sxs-lookup"><span data-stu-id="4b41f-114">Azure Analysis Services supports connecting to [various data sources](/azure/analysis-services/analysis-services-datasource) in the cloud and on-premises in your organization.</span></span>

<span data-ttu-id="4b41f-115">Кластеризованные индексы columnstore доступны в SQL Server 2014 и более поздних версий, а также в Базе данных SQL Azure и отлично подходят для рабочих нагрузок OLAP.</span><span class="sxs-lookup"><span data-stu-id="4b41f-115">Clustered Columnstore indexes are available in SQL Server 2014 and above, as well as Azure SQL Database, and are ideal for OLAP workloads.</span></span> <span data-ttu-id="4b41f-116">Но начиная с версии SQL Server 2016 (включая Базу данных SQL Azure) вы можете воспользоваться гибридной транзакционно-аналитической обработкой (HTAP) благодаря обновляемым некластеризованным индексам columnstore.</span><span class="sxs-lookup"><span data-stu-id="4b41f-116">However, beginning with SQL Server 2016 (including Azure SQL Database), you can take advantage of hybrid transactional/analytics processing (HTAP) through the use of updateable nonclustered columnstore indexes.</span></span> <span data-ttu-id="4b41f-117">HTAP позволяет выполнять задачи обработки OLTP и OLAP на одной платформе, что избавляет от необходимости хранить несколько копий данных и использовать отдельные системы OLTP и OLAP.</span><span class="sxs-lookup"><span data-stu-id="4b41f-117">HTAP enables you to perform OLTP and OLAP processing on the same platform, which removes the need to store multiple copies of your data, and eliminates the need for distinct OLTP and OLAP systems.</span></span> <span data-ttu-id="4b41f-118">Дополнительные сведения см. в статье [Начало работы с Columnstore для получения операционной аналитики в реальном времени](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics).</span><span class="sxs-lookup"><span data-stu-id="4b41f-118">For more information, see [Get started with Columnstore for real-time operational analytics](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics).</span></span>

## <a name="key-selection-criteria"></a><span data-ttu-id="4b41f-119">Основные критерии выбора</span><span class="sxs-lookup"><span data-stu-id="4b41f-119">Key selection criteria</span></span>

<span data-ttu-id="4b41f-120">Чтобы ограничить количество вариантов, сначала ответьте на следующие вопросы:</span><span class="sxs-lookup"><span data-stu-id="4b41f-120">To narrow the choices, start by answering these questions:</span></span>

- <span data-ttu-id="4b41f-121">Вы хотите использовать управляемую службу, а не управлять собственными серверами?</span><span class="sxs-lookup"><span data-stu-id="4b41f-121">Do you want a managed service rather than managing your own servers?</span></span>

- <span data-ttu-id="4b41f-122">Требуется ли безопасная аутентификация с использованием Azure Active Directory (Azure AD)?</span><span class="sxs-lookup"><span data-stu-id="4b41f-122">Do you require secure authentication using Azure Active Directory (Azure AD)?</span></span>

- <span data-ttu-id="4b41f-123">Вам нужно проводить анализ в реальном времени?</span><span class="sxs-lookup"><span data-stu-id="4b41f-123">Do you want to conduct real-time analytics?</span></span> <span data-ttu-id="4b41f-124">Если да, оставьте только те варианты, которые поддерживают аналитику в реальном времени.</span><span class="sxs-lookup"><span data-stu-id="4b41f-124">If so, narrow your options to those that support real-time analytics.</span></span> 

    <span data-ttu-id="4b41f-125">*Аналитика в реальном времени* в этом контексте применяется к одному источнику данных, например к приложению для управления ресурсами предприятия (ERP), в котором будут выполняться операционная и аналитическая рабочие нагрузки.</span><span class="sxs-lookup"><span data-stu-id="4b41f-125">*Real-time analytics* in this context applies to a single data source, such as an enterprise resource planning (ERP) application, that will run both an operational and an analytics workload.</span></span> <span data-ttu-id="4b41f-126">Если требуется интегрировать данные из нескольких источников или обеспечить максимальную производительность для анализа с помощью предварительно вычисленных данных, таких как кубы, вам может потребоваться отдельное хранилище данных.</span><span class="sxs-lookup"><span data-stu-id="4b41f-126">If you need to integrate data from multiple sources, or require extreme analytics performance by using pre-aggregated data such as cubes, you might still require a separate data warehouse.</span></span>

- <span data-ttu-id="4b41f-127">Вам нужно использовать предварительно вычисленные данные, например, чтобы предоставлять семантические модели, которые делают анализ более удобным для организаций?</span><span class="sxs-lookup"><span data-stu-id="4b41f-127">Do you need to use pre-aggregated data, for example to provide semantic models that make analytics more business user friendly?</span></span> <span data-ttu-id="4b41f-128">Если да, выберите вариант, который поддерживает многомерные кубы или табличные семантические модели.</span><span class="sxs-lookup"><span data-stu-id="4b41f-128">If yes, choose an option that supports multidimensional cubes or tabular semantic models.</span></span> 

    <span data-ttu-id="4b41f-129">Благодаря статистическим выражениям пользователи могут последовательно выполнять статистическое вычисление данных.</span><span class="sxs-lookup"><span data-stu-id="4b41f-129">Providing aggregates can help users consistently calculate data aggregates.</span></span> <span data-ttu-id="4b41f-130">Предварительно вычисленные данные также позволяют значительно повысить производительность при работе с несколькими столбцами с множеством строк.</span><span class="sxs-lookup"><span data-stu-id="4b41f-130">Pre-aggregated data can also provide a large performance boost when dealing with several columns across many rows.</span></span> <span data-ttu-id="4b41f-131">Предварительно вычисленные данные могут быть представлены в виде многомерного куба или табличной семантической модели.</span><span class="sxs-lookup"><span data-stu-id="4b41f-131">Data can be pre-aggregated in multidimensional cubes or tabular semantic models.</span></span>

- <span data-ttu-id="4b41f-132">Нужно ли интегрировать данные из нескольких источников за пределами хранилища данных OLTP?</span><span class="sxs-lookup"><span data-stu-id="4b41f-132">Do you need to integrate data from several sources, beyond your OLTP data store?</span></span> <span data-ttu-id="4b41f-133">Если да, рассмотрите варианты, которые позволяют легко интегрировать несколько источников данных.</span><span class="sxs-lookup"><span data-stu-id="4b41f-133">If so, consider options that easily integrate multiple data sources.</span></span>

## <a name="capability-matrix"></a><span data-ttu-id="4b41f-134">Матрица возможностей</span><span class="sxs-lookup"><span data-stu-id="4b41f-134">Capability matrix</span></span>

<span data-ttu-id="4b41f-135">В следующих таблицах перечислены основные различия в возможностях.</span><span class="sxs-lookup"><span data-stu-id="4b41f-135">The following tables summarize the key differences in capabilities.</span></span>

### <a name="general-capabilities"></a><span data-ttu-id="4b41f-136">Общие возможности</span><span class="sxs-lookup"><span data-stu-id="4b41f-136">General capabilities</span></span>

| | <span data-ttu-id="4b41f-137">Службы Azure Analysis Services</span><span class="sxs-lookup"><span data-stu-id="4b41f-137">Azure Analysis Services</span></span> | <span data-ttu-id="4b41f-138">SQL Server Analysis Services</span><span class="sxs-lookup"><span data-stu-id="4b41f-138">SQL Server Analysis Services</span></span> | <span data-ttu-id="4b41f-139">SQL Server с индексами columnstore</span><span class="sxs-lookup"><span data-stu-id="4b41f-139">SQL Server with Columnstore Indexes</span></span> | <span data-ttu-id="4b41f-140">База данных SQL Azure с индексами columnstore</span><span class="sxs-lookup"><span data-stu-id="4b41f-140">Azure SQL Database with Columnstore Indexes</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="4b41f-141">Управляемая служба</span><span class="sxs-lookup"><span data-stu-id="4b41f-141">Is managed service</span></span> | <span data-ttu-id="4b41f-142">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-142">Yes</span></span> | <span data-ttu-id="4b41f-143">Нет </span><span class="sxs-lookup"><span data-stu-id="4b41f-143">No</span></span> | <span data-ttu-id="4b41f-144">Нет </span><span class="sxs-lookup"><span data-stu-id="4b41f-144">No</span></span> | <span data-ttu-id="4b41f-145">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-145">Yes</span></span> |
| <span data-ttu-id="4b41f-146">Поддержка многомерных кубов</span><span class="sxs-lookup"><span data-stu-id="4b41f-146">Supports multidimensional cubes</span></span> | <span data-ttu-id="4b41f-147">Нет </span><span class="sxs-lookup"><span data-stu-id="4b41f-147">No</span></span> | <span data-ttu-id="4b41f-148">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-148">Yes</span></span> | <span data-ttu-id="4b41f-149">Нет </span><span class="sxs-lookup"><span data-stu-id="4b41f-149">No</span></span> | <span data-ttu-id="4b41f-150">Нет </span><span class="sxs-lookup"><span data-stu-id="4b41f-150">No</span></span> |
| <span data-ttu-id="4b41f-151">Поддержка табличных семантических моделей</span><span class="sxs-lookup"><span data-stu-id="4b41f-151">Supports tabular semantic models</span></span> | <span data-ttu-id="4b41f-152">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-152">Yes</span></span> | <span data-ttu-id="4b41f-153">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-153">Yes</span></span> | <span data-ttu-id="4b41f-154">Нет </span><span class="sxs-lookup"><span data-stu-id="4b41f-154">No</span></span> | <span data-ttu-id="4b41f-155">Нет </span><span class="sxs-lookup"><span data-stu-id="4b41f-155">No</span></span> |
| <span data-ttu-id="4b41f-156">Простая интеграция нескольких источников данных</span><span class="sxs-lookup"><span data-stu-id="4b41f-156">Easily integrate multiple data sources</span></span> | <span data-ttu-id="4b41f-157">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-157">Yes</span></span> | <span data-ttu-id="4b41f-158">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-158">Yes</span></span> | <span data-ttu-id="4b41f-159">Нет <sup>1</sup></span><span class="sxs-lookup"><span data-stu-id="4b41f-159">No <sup>1</sup></span></span> | <span data-ttu-id="4b41f-160">Нет <sup>1</sup></span><span class="sxs-lookup"><span data-stu-id="4b41f-160">No <sup>1</sup></span></span> |
| <span data-ttu-id="4b41f-161">Поддержка аналитики в режиме реального времени</span><span class="sxs-lookup"><span data-stu-id="4b41f-161">Supports real-time analytics</span></span> | <span data-ttu-id="4b41f-162">Нет </span><span class="sxs-lookup"><span data-stu-id="4b41f-162">No</span></span> | <span data-ttu-id="4b41f-163">Нет </span><span class="sxs-lookup"><span data-stu-id="4b41f-163">No</span></span> | <span data-ttu-id="4b41f-164">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-164">Yes</span></span> | <span data-ttu-id="4b41f-165">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-165">Yes</span></span> |
| <span data-ttu-id="4b41f-166">Необходимость обработки данных для их копирования из источников</span><span class="sxs-lookup"><span data-stu-id="4b41f-166">Requires process to copy data from source(s)</span></span> | <span data-ttu-id="4b41f-167">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-167">Yes</span></span> | <span data-ttu-id="4b41f-168">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-168">Yes</span></span> | <span data-ttu-id="4b41f-169">Нет </span><span class="sxs-lookup"><span data-stu-id="4b41f-169">No</span></span> | <span data-ttu-id="4b41f-170">Нет </span><span class="sxs-lookup"><span data-stu-id="4b41f-170">No</span></span> |
| <span data-ttu-id="4b41f-171">Интеграция с Azure AD</span><span class="sxs-lookup"><span data-stu-id="4b41f-171">Azure AD integration</span></span> | <span data-ttu-id="4b41f-172">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-172">Yes</span></span> | <span data-ttu-id="4b41f-173">Нет </span><span class="sxs-lookup"><span data-stu-id="4b41f-173">No</span></span> | <span data-ttu-id="4b41f-174">Нет <sup>2</sup></span><span class="sxs-lookup"><span data-stu-id="4b41f-174">No <sup>2</sup></span></span> | <span data-ttu-id="4b41f-175">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-175">Yes</span></span> |

<span data-ttu-id="4b41f-176">[1] Хотя SQL Server и Базу данных SQL Azure нельзя использовать для отправки запросов и интеграции нескольких внешних источников данных, можно создать конвейер для этих задач с помощью [SSIS](/sql/integration-services/sql-server-integration-services) или [фабрики данных Azure](/azure/data-factory/).</span><span class="sxs-lookup"><span data-stu-id="4b41f-176">[1] Although SQL Server and Azure SQL Database cannot be used to query from and integrate multiple external data sources, you can still build a pipeline that does this for you using [SSIS](/sql/integration-services/sql-server-integration-services) or [Azure Data Factory](/azure/data-factory/).</span></span> <span data-ttu-id="4b41f-177">Сервер SQL Server, размещенный на виртуальной машине Azure, предоставляет дополнительные варианты, например связанные серверы и [PolyBase](/sql/relational-databases/polybase/polybase-guide).</span><span class="sxs-lookup"><span data-stu-id="4b41f-177">SQL Server hosted in an Azure VM has additional options, such as linked servers and [PolyBase](/sql/relational-databases/polybase/polybase-guide).</span></span> <span data-ttu-id="4b41f-178">Дополнительные сведения см. в статье [Choosing a data pipeline orchestration technology in Azure](../technology-choices/pipeline-orchestration-data-movement.md) (Выбор технологии оркестрации конвейера данных в Azure).</span><span class="sxs-lookup"><span data-stu-id="4b41f-178">For more information, see [Pipeline orchestration, control flow, and data movement](../technology-choices/pipeline-orchestration-data-movement.md).</span></span>

<span data-ttu-id="4b41f-179">[2] Подключение к SQL Server на виртуальной машине Azure с помощью учетной записи Azure AD не поддерживается.</span><span class="sxs-lookup"><span data-stu-id="4b41f-179">[2] Connecting to SQL Server running on an Azure Virtual Machine is not supported using an Azure AD account.</span></span> <span data-ttu-id="4b41f-180">Вместо этого используйте учетную запись домена Active Directory.</span><span class="sxs-lookup"><span data-stu-id="4b41f-180">Use a domain Active Directory account instead.</span></span>

### <a name="scalability-capabilities"></a><span data-ttu-id="4b41f-181">Масштабируемость</span><span class="sxs-lookup"><span data-stu-id="4b41f-181">Scalability Capabilities</span></span>

| | <span data-ttu-id="4b41f-182">Службы Azure Analysis Services</span><span class="sxs-lookup"><span data-stu-id="4b41f-182">Azure Analysis Services</span></span> | <span data-ttu-id="4b41f-183">SQL Server Analysis Services</span><span class="sxs-lookup"><span data-stu-id="4b41f-183">SQL Server Analysis Services</span></span> | <span data-ttu-id="4b41f-184">SQL Server с индексами columnstore</span><span class="sxs-lookup"><span data-stu-id="4b41f-184">SQL Server with Columnstore Indexes</span></span> | <span data-ttu-id="4b41f-185">База данных SQL Azure с индексами columnstore</span><span class="sxs-lookup"><span data-stu-id="4b41f-185">Azure SQL Database with Columnstore Indexes</span></span> |
| --- | --- | --- | --- | --- |
| <span data-ttu-id="4b41f-186">Избыточные региональные серверы для высокого уровня доступности</span><span class="sxs-lookup"><span data-stu-id="4b41f-186">Redundant regional servers for high availability</span></span>  | <span data-ttu-id="4b41f-187">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-187">Yes</span></span> | <span data-ttu-id="4b41f-188">Нет </span><span class="sxs-lookup"><span data-stu-id="4b41f-188">No</span></span> | <span data-ttu-id="4b41f-189">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-189">Yes</span></span> | <span data-ttu-id="4b41f-190">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-190">Yes</span></span> |
| <span data-ttu-id="4b41f-191">Поддержка масштабирования запросов</span><span class="sxs-lookup"><span data-stu-id="4b41f-191">Supports query scale out</span></span>  | <span data-ttu-id="4b41f-192">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-192">Yes</span></span> | <span data-ttu-id="4b41f-193">Нет </span><span class="sxs-lookup"><span data-stu-id="4b41f-193">No</span></span> | <span data-ttu-id="4b41f-194">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-194">Yes</span></span> | <span data-ttu-id="4b41f-195">Нет </span><span class="sxs-lookup"><span data-stu-id="4b41f-195">No</span></span> |
| <span data-ttu-id="4b41f-196">Динамическая масштабируемость (увеличение масштаба)</span><span class="sxs-lookup"><span data-stu-id="4b41f-196">Dynamic scalability (scale up)</span></span>  | <span data-ttu-id="4b41f-197">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-197">Yes</span></span> | <span data-ttu-id="4b41f-198">Нет </span><span class="sxs-lookup"><span data-stu-id="4b41f-198">No</span></span> | <span data-ttu-id="4b41f-199">Yes</span><span class="sxs-lookup"><span data-stu-id="4b41f-199">Yes</span></span> | <span data-ttu-id="4b41f-200">Нет </span><span class="sxs-lookup"><span data-stu-id="4b41f-200">No</span></span> |
