---
title: CAF. Улучшения по дисциплине управления затратами
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Улучшения по дисциплине управления затратами
author: BrianBlanchard
ms.openlocfilehash: 34975d195a95b1a85ada96efe8c76a6138385ec1
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55902572"
---
# <a name="cost-management-discipline-improvement"></a><span data-ttu-id="1fc34-103">Улучшения по дисциплине управления затратами</span><span class="sxs-lookup"><span data-stu-id="1fc34-103">Cost Management discipline improvement</span></span>

<span data-ttu-id="1fc34-104">Дисциплина "Управление затратами" нацелена на анализ ключевых бизнес-рисков, связанных с расходами на размещение облачных рабочих нагрузок.</span><span class="sxs-lookup"><span data-stu-id="1fc34-104">The Cost Management discipline attempts to address core business risks related to expenses incurred when hosting cloud-based workloads.</span></span> <span data-ttu-id="1fc34-105">В структуре пяти дисциплин управления облаком управление затратами отвечает за контроль затрат и использование облачных ресурсов с целью создания и сопровождения планового цикла расходов.</span><span class="sxs-lookup"><span data-stu-id="1fc34-105">Within the five disciplines of Cloud Governance, Cost Management is involved in controlling cost and usage of cloud resources with the goal of creating and maintaining a planned cost cycle.</span></span>

<span data-ttu-id="1fc34-106">В этой статье описаны задачи, которые компания может выполнить для развития и совершенствования дисциплины управления затратами.</span><span class="sxs-lookup"><span data-stu-id="1fc34-106">This article outlines potential tasks your company perform to develop and mature your Cost Management discipline.</span></span> <span data-ttu-id="1fc34-107">Эти задачи можно разделить на этапы планирования, создания, внедрения и реализации облачного решения. Итеративное выполнение этих этапов позволяет разработать [поэтапный подход к системе управления облаком](../journeys/overview.md#an-incremental-approach-to-cloud-governance).</span><span class="sxs-lookup"><span data-stu-id="1fc34-107">These tasks can be broken down into planning, building, adopting, and operating phases of implementing a cloud solution, which are then iterated on allowing the development of an [incremental approach to cloud governance](../journeys/overview.md#an-incremental-approach-to-cloud-governance).</span></span>

![Четыре этапа внедрения](../../_images/adoption-phases.png)

<span data-ttu-id="1fc34-109">*Рис. 1. Этапы внедрения инкрементального подхода к системе управления облаком.*</span><span class="sxs-lookup"><span data-stu-id="1fc34-109">*Figure 1. Adoption phases of the incremental approach to cloud governance.*</span></span>

<span data-ttu-id="1fc34-110">Невозможно учесть в одном документе все требования любых компаний.</span><span class="sxs-lookup"><span data-stu-id="1fc34-110">No single document can account for the requirements of all businesses.</span></span> <span data-ttu-id="1fc34-111">Поэтому в этой статье описываются минимально требуемые и возможные действия для каждого этапа в процессе совершенствования системы управления.</span><span class="sxs-lookup"><span data-stu-id="1fc34-111">As such, this article outlines suggested minimum and potential example activities for each phase of the governance maturation process.</span></span> <span data-ttu-id="1fc34-112">Начальной целью этих действий является создание [MVP политики](../journeys/overview.md#an-incremental-approach-to-cloud-governance) и разработка платформы для инкрементального развития этой политики.</span><span class="sxs-lookup"><span data-stu-id="1fc34-112">The initial objective of these activities is to help you build a [Policy MVP](../journeys/overview.md#an-incremental-approach-to-cloud-governance) and establish a framework for incremental policy evolution.</span></span> <span data-ttu-id="1fc34-113">Команде управления облаком необходимо решить, какие потребуются инвестиции в эти действия, чтобы улучшить возможности системы управления затратами.</span><span class="sxs-lookup"><span data-stu-id="1fc34-113">Your Cloud Governance team will need to decide how much to invest in these activities to improve your Cost Management governance capabilities.</span></span>

> [!CAUTION]
> <span data-ttu-id="1fc34-114">Описанные в этой статье минимальные или возможные действия не согласованы с какими-либо корпоративными политиками или нормативными требованиями третьих сторон.</span><span class="sxs-lookup"><span data-stu-id="1fc34-114">Neither the minimum or potential activities outlined in this article are aligned to specific corporate policies or third party compliance requirements.</span></span> <span data-ttu-id="1fc34-115">Это руководство призвано облегчить обсуждения по согласованию модели управления облаком с обоими требованиями.</span><span class="sxs-lookup"><span data-stu-id="1fc34-115">This guidance is designed to help facilitate the conversations that will lead to alignment of both requirements with a cloud governance model.</span></span>

## <a name="planning-and-readiness"></a><span data-ttu-id="1fc34-116">Планирование и готовность</span><span class="sxs-lookup"><span data-stu-id="1fc34-116">Planning and readiness</span></span>

<span data-ttu-id="1fc34-117">На этом этапе развития системы управления ликвидируется разрыв между бизнес-показателями и стратегиями практических действий.</span><span class="sxs-lookup"><span data-stu-id="1fc34-117">This phase of governance maturity bridges the divide between business outcomes and actionable strategies.</span></span> <span data-ttu-id="1fc34-118">В ходе этого процесса руководство определяет конкретные метрики, сопоставляет их с цифровыми активами и начинает планировать общий процесс миграции.</span><span class="sxs-lookup"><span data-stu-id="1fc34-118">During this process, the leadership team defines specific metrics, maps those metrics to the digital estate, and begins planning the overall migration effort.</span></span>

<span data-ttu-id="1fc34-119">**Минимальные рекомендуемые действия:**</span><span class="sxs-lookup"><span data-stu-id="1fc34-119">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="1fc34-120">Оцените возможности для создания [цепочки инструментов управления затратами](toolchain.md).</span><span class="sxs-lookup"><span data-stu-id="1fc34-120">Evaluate your [Cost Management toolchain](toolchain.md) options.</span></span>
* <span data-ttu-id="1fc34-121">Подготовьте черновой вариант рекомендаций по архитектуре и передайте его основным заинтересованным лицам.</span><span class="sxs-lookup"><span data-stu-id="1fc34-121">Develop a draft Architecture Guidelines document and distribute to key stakeholders.</span></span>
* <span data-ttu-id="1fc34-122">Проведите обучение и привлеките к участию людей и команды, на работу которых повлияет создание рекомендаций по архитектуре.</span><span class="sxs-lookup"><span data-stu-id="1fc34-122">Educate and involve the people and teams affected by the development of Architecture Guidelines.</span></span>

<span data-ttu-id="1fc34-123">**Возможные действия:**</span><span class="sxs-lookup"><span data-stu-id="1fc34-123">**Potential activities:**</span></span>

* <span data-ttu-id="1fc34-124">Примите решения по бюджету, которые поддержат коммерческое обоснование вашей облачной стратегии.</span><span class="sxs-lookup"><span data-stu-id="1fc34-124">Ensure budgetary decisions that support the business justification for your cloud strategy.</span></span>
* <span data-ttu-id="1fc34-125">Проверьте метрики обучения, которые вы используете для отчетов об успешном распределении финансирования.</span><span class="sxs-lookup"><span data-stu-id="1fc34-125">Validate learning metrics that you use to report on the successful allocation of funding.</span></span>
* <span data-ttu-id="1fc34-126">Определитесь с целевой моделью учета в облаке, которая влияет на методы учета затрат на облачные решения.</span><span class="sxs-lookup"><span data-stu-id="1fc34-126">Understand the desired cloud accounting model that affects how cloud costs should be accounted for.</span></span>
* <span data-ttu-id="1fc34-127">Ознакомьтесь с планом цифровых ресурсов и проверьте точные прогнозы по затратам.</span><span class="sxs-lookup"><span data-stu-id="1fc34-127">Become familiar with the digital estate plan and validate accurate costing expectations.</span></span>
* <span data-ttu-id="1fc34-128">Оцените варианты приобретения, например выберите "оплату по мере использования" или предварительную оплату по Соглашению Enterprise.</span><span class="sxs-lookup"><span data-stu-id="1fc34-128">Evaluate buying options to determine if it's better to "pay as you go" or to make a precommitment by purchasing an Enterprise Agreement.</span></span>
* <span data-ttu-id="1fc34-129">Сопоставьте бизнес-цели с выделенными бюджетами и скорректируйте бюджетные планы при необходимости.</span><span class="sxs-lookup"><span data-stu-id="1fc34-129">Align business goals with planned budgets and adjust budgetary plans as necessary.</span></span>
* <span data-ttu-id="1fc34-130">Разработайте механизм отчетности по целям и бюджетам, чтобы передавать информацию заинтересованным лицам в технических и производственных подразделениях в конце каждого цикла затрат.</span><span class="sxs-lookup"><span data-stu-id="1fc34-130">Develop a goals and budget reporting mechanism to notify technical and business stakeholders at the end of each cost cycle.</span></span>

## <a name="build-and-pre-deployment"></a><span data-ttu-id="1fc34-131">Сборка и предварительное развертывание</span><span class="sxs-lookup"><span data-stu-id="1fc34-131">Build and pre-deployment</span></span>

<span data-ttu-id="1fc34-132">Для успешной миграции среды нужно выполнить ряд технических и нетехнических требований.</span><span class="sxs-lookup"><span data-stu-id="1fc34-132">A number of technical and nontechnical prerequisites are required to successfully migrate an environment.</span></span> <span data-ttu-id="1fc34-133">Этот процесс нацелен на решения, подготовленность и базовую инфраструктуру, которые должны быть готовы до миграции.</span><span class="sxs-lookup"><span data-stu-id="1fc34-133">This process focuses on the decisions, readiness, and core infrastructure that proceeds a migration.</span></span>

<span data-ttu-id="1fc34-134">**Минимальные рекомендуемые действия:**</span><span class="sxs-lookup"><span data-stu-id="1fc34-134">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="1fc34-135">Создайте [цепочку инструментов управления затратами](toolchain.md), развернув ее в среде подготовки к развертыванию.</span><span class="sxs-lookup"><span data-stu-id="1fc34-135">Implement your [Cost Management toolchain](toolchain.md) by rolling out in a pre-deployment phase.</span></span>
* <span data-ttu-id="1fc34-136">Подготовьте обновленный вариант рекомендаций по архитектуре и передайте его ключевым заинтересованным лицам.</span><span class="sxs-lookup"><span data-stu-id="1fc34-136">Update the Architecture Guidelines document and distribute to key stakeholders.</span></span>
* <span data-ttu-id="1fc34-137">Разработайте обучающие материалы и документы, информационные сообщения, поощрения и другие программы, которые помогут пользователям освоиться.</span><span class="sxs-lookup"><span data-stu-id="1fc34-137">Develop educational materials and documentation, awareness communications, incentives, and other programs to help drive user adoption.</span></span>
* <span data-ttu-id="1fc34-138">Определите, соответствуют ли требования к закупкам выделенным бюджетам и поставленным целям.</span><span class="sxs-lookup"><span data-stu-id="1fc34-138">Determine if your purchase requirements align with your budgets and goals.</span></span>

<span data-ttu-id="1fc34-139">**Возможные действия:**</span><span class="sxs-lookup"><span data-stu-id="1fc34-139">**Potential activities:**</span></span>

* <span data-ttu-id="1fc34-140">Согласуйте план бюджета со [стратегией подписки](../../decision-guides/subscriptions/overview.md), которая определяет базовую модель ответственности.</span><span class="sxs-lookup"><span data-stu-id="1fc34-140">Align your budgetary plans with the [Subscription Strategy](../../decision-guides/subscriptions/overview.md) that defines your core ownership model.</span></span>
* <span data-ttu-id="1fc34-141">Примените [стратегию согласованности ресурсов](../../decision-guides/resource-consistency/overview.md) для постепенного внедрения рекомендаций по архитектуре и затратам.</span><span class="sxs-lookup"><span data-stu-id="1fc34-141">Leverage the [Resource Consistency Strategy](../../decision-guides/resource-consistency/overview.md) to enforce architecture and cost guidelines over time.</span></span>
* <span data-ttu-id="1fc34-142">Определите наличие аномально высоких затрат, которые могут повлиять на планы внедрения и миграции.</span><span class="sxs-lookup"><span data-stu-id="1fc34-142">Determine if there are any cost anomalies that affect your adoption and migration plans.</span></span>

## <a name="adopt-and-migrate"></a><span data-ttu-id="1fc34-143">Внедрение и миграция</span><span class="sxs-lookup"><span data-stu-id="1fc34-143">Adopt and migrate</span></span>

<span data-ttu-id="1fc34-144">Миграция представляет собой последовательный процесс перемещения, тестирования и внедрения приложений или рабочих нагрузок в существующей цифровой инфраструктуре.</span><span class="sxs-lookup"><span data-stu-id="1fc34-144">Migration is an incremental process that focuses on the movement, testing, and adoption of applications or workloads in an existing digital estate.</span></span>

<span data-ttu-id="1fc34-145">**Минимальные рекомендуемые действия:**</span><span class="sxs-lookup"><span data-stu-id="1fc34-145">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="1fc34-146">Перенесите ваши [цепочки инструментов управления затратами](toolchain.md) из среды подготовки развертывания в рабочую среду.</span><span class="sxs-lookup"><span data-stu-id="1fc34-146">Migrate your [Cost Management toolchain](toolchain.md) from pre-deployment to production.</span></span>
* <span data-ttu-id="1fc34-147">Подготовьте обновленный вариант рекомендаций по архитектуре и передайте его ключевым заинтересованным лицам.</span><span class="sxs-lookup"><span data-stu-id="1fc34-147">Update the Architecture Guidelines document and distribute to key stakeholders.</span></span>
* <span data-ttu-id="1fc34-148">Разработайте обучающие материалы и документы, информационные сообщения, поощрения и другие программы, которые помогут пользователям освоиться.</span><span class="sxs-lookup"><span data-stu-id="1fc34-148">Develop educational materials and documentation, awareness communications, incentives, and other programs to help drive user adoption.</span></span>

<span data-ttu-id="1fc34-149">**Возможные действия:**</span><span class="sxs-lookup"><span data-stu-id="1fc34-149">**Potential activities:**</span></span>

* <span data-ttu-id="1fc34-150">Внедрите модель учета в облаке.</span><span class="sxs-lookup"><span data-stu-id="1fc34-150">Implement your Cloud Accounting Model.</span></span>
* <span data-ttu-id="1fc34-151">Убедитесь, что все бюджеты правильно отражают фактические расходы на каждый этап выпуска, а при необходимости внесите изменения.</span><span class="sxs-lookup"><span data-stu-id="1fc34-151">Ensure that your budgets reflect your actual spending during each release and adjust as necessary.</span></span>
* <span data-ttu-id="1fc34-152">Отслеживайте изменения в планах бюджета и согласуйте с заинтересованными лицами дополнительные одобрения.</span><span class="sxs-lookup"><span data-stu-id="1fc34-152">Monitor changes in budgetary plans and validate with stakeholders if additional sign-offs are needed.</span></span>
* <span data-ttu-id="1fc34-153">Внесите в документ рекомендаций по архитектуре изменения, отражающие фактические расходы.</span><span class="sxs-lookup"><span data-stu-id="1fc34-153">Update changes to the Architecture Guidelines document to reflect actual costs.</span></span>

## <a name="operate-and-post-implementation"></a><span data-ttu-id="1fc34-154">Эксплуатация и действия после реализации</span><span class="sxs-lookup"><span data-stu-id="1fc34-154">Operate and post-implementation</span></span>

<span data-ttu-id="1fc34-155">Когда преобразование завершится, системы управления и эксплуатации должны сохраняться для поддержки естественного жизненного цикла приложений или рабочих нагрузок.</span><span class="sxs-lookup"><span data-stu-id="1fc34-155">Once the transformation is complete, governance and operations must live on for the natural lifecycle of an application or workload.</span></span> <span data-ttu-id="1fc34-156">Этот этап развития системы управления посвящен действиям, которые обычно следуют за внедрением решения, когда цикл трансформации становится более стабильным.</span><span class="sxs-lookup"><span data-stu-id="1fc34-156">This phase of governance maturity focuses on the activities that commonly come after the solution is implemented and the transformation cycle begins to stabilize.</span></span>

<span data-ttu-id="1fc34-157">**Минимальные рекомендуемые действия:**</span><span class="sxs-lookup"><span data-stu-id="1fc34-157">**Minimum suggested activities:**</span></span>

* <span data-ttu-id="1fc34-158">Измените параметры [цепочки инструментов управления затратами](toolchain.md) с учетом изменений требований к управлению затратами в организации.</span><span class="sxs-lookup"><span data-stu-id="1fc34-158">Customize your [Cost Management toolchain](toolchain.md) based on changes in your organization’s cost management needs.</span></span>
* <span data-ttu-id="1fc34-159">Постарайтесь внедрить автоматизацию уведомлений и отчетов по фактическим расходам.</span><span class="sxs-lookup"><span data-stu-id="1fc34-159">Consider automating any notifications and reports to reflect actual spending.</span></span>
* <span data-ttu-id="1fc34-160">Скорректируйте рекомендации по архитектуре, чтобы управлять дальнейшими процессами внедрения.</span><span class="sxs-lookup"><span data-stu-id="1fc34-160">Refine Architecture Guidelines to guide future adoption processes.</span></span>
* <span data-ttu-id="1fc34-161">Периодически проводите обучение для задействованных команд, чтобы обеспечить соблюдение рекомендаций по архитектуре.</span><span class="sxs-lookup"><span data-stu-id="1fc34-161">Educate affected teams on a periodic basis to ensure ongoing adherence to the Architecture Guidelines.</span></span>

<span data-ttu-id="1fc34-162">**Возможные действия:**</span><span class="sxs-lookup"><span data-stu-id="1fc34-162">**Potential activities:**</span></span>

* <span data-ttu-id="1fc34-163">Ежеквартально выполняйте проверку облачного бизнеса, чтобы получить данные о созданной пользе для бизнеса и соответствующих затратах.</span><span class="sxs-lookup"><span data-stu-id="1fc34-163">Execute a quarterly cloud business review to communicate value delivered to the business and associated costs.</span></span>
* <span data-ttu-id="1fc34-164">Ежеквартально корректируйте планы с учетом изменений в фактических расходах.</span><span class="sxs-lookup"><span data-stu-id="1fc34-164">Adjust plans quarterly to reflect changes to actual spending.</span></span>
* <span data-ttu-id="1fc34-165">Оцените соответствие финансовых показателей отчету о доходах и расходах по подпискам для бизнес-единиц.</span><span class="sxs-lookup"><span data-stu-id="1fc34-165">Determine financial alignment to P&Ls for business unit subscriptions.</span></span>
* <span data-ttu-id="1fc34-166">Ежемесячно оценивайте методы информирования заинтересованных лиц о ценности и затратах.</span><span class="sxs-lookup"><span data-stu-id="1fc34-166">Analyze stakeholder value and cost reporting methods on a monthly basis.</span></span>
* <span data-ttu-id="1fc34-167">Принимайте меры по простаивающим ресурсам и принимайте решения о продолжении их поддержки.</span><span class="sxs-lookup"><span data-stu-id="1fc34-167">Remediate underused assets and determine if they're worth continuing.</span></span>
* <span data-ttu-id="1fc34-168">Выявляйте несогласованность между плановыми и фактическими расходами, а также любые аномалии.</span><span class="sxs-lookup"><span data-stu-id="1fc34-168">Detect misalignments and anomalies between the plan and actual spending.</span></span>
* <span data-ttu-id="1fc34-169">Помогите командам внедрения облака и команде разработчиков облачной стратегии разобраться в этих аномалиях и устранить их.</span><span class="sxs-lookup"><span data-stu-id="1fc34-169">Assist the cloud adoption teams and the Cloud Strategy team with understanding and resolving these anomalies.</span></span>

## <a name="next-steps"></a><span data-ttu-id="1fc34-170">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="1fc34-170">Next steps</span></span>

<span data-ttu-id="1fc34-171">Теперь, когда вы понимаете концепцию стратегии управления удостоверениями в облаке, изучите [цепочки инструментов управления затратами](toolchain.md), чтобы выбрать наиболее подходящие средства и функции Azure, которые вам потребуются при развитии дисциплины управления затратами на платформе Azure.</span><span class="sxs-lookup"><span data-stu-id="1fc34-171">Now that you understand the concept of cloud identity governance, examine the [Cost Management toolchain](toolchain.md) to identify Azure tools and features that you'll need when developing the Cost Management governance discipline on the Azure platform.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="1fc34-172">Цепочка инструментов управления затратами для Azure</span><span class="sxs-lookup"><span data-stu-id="1fc34-172">Cost Management toolchain for Azure</span></span>](toolchain.md)