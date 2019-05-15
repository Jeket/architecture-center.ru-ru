---
title: CAF. Примеры правил политики управления затратами
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 1/4/2019
description: Примеры правил политики управления затратами
author: BrianBlanchard
ms.openlocfilehash: 1c8b94ae5285fa26cdf9760892beaced2487af8b
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55902520"
---
# <a name="cost-management-sample-policy-statements"></a><span data-ttu-id="cca6e-103">Примеры правил политики управления затратами</span><span class="sxs-lookup"><span data-stu-id="cca6e-103">Cost Management sample policy statements</span></span>

<span data-ttu-id="cca6e-104">Отдельные правила облачной политики служат рекомендациями по устранению конкретных рисков, выявленных в процессе оценки рисков.</span><span class="sxs-lookup"><span data-stu-id="cca6e-104">Individual cloud policy statements are guidelines for addressing specific risks identified during your risk assessment process.</span></span> <span data-ttu-id="cca6e-105">В этих правилах должна предоставляться краткая сводка по рискам и планам их устранения.</span><span class="sxs-lookup"><span data-stu-id="cca6e-105">These statements should provide a concise summary of risks and plans to deal with them.</span></span> <span data-ttu-id="cca6e-106">Каждое определение правила должно включать следующие сведения:</span><span class="sxs-lookup"><span data-stu-id="cca6e-106">Each statement definition should include these pieces of information:</span></span>

- <span data-ttu-id="cca6e-107">Бизнес-риск. Сводная информация о риске, которому посвящена эта политика.</span><span class="sxs-lookup"><span data-stu-id="cca6e-107">Business risk - A summary of the risk this policy will address.</span></span>
- <span data-ttu-id="cca6e-108">Правила политики. Четкое описание требований политики.</span><span class="sxs-lookup"><span data-stu-id="cca6e-108">Policy statement - A clear summary explanation of the policy requirements.</span></span>
- <span data-ttu-id="cca6e-109">Варианты архитектуры. Практические рекомендации, спецификации или другие руководства, которые позволят ИТ-специалистам и разработчикам реализовать эту политику.</span><span class="sxs-lookup"><span data-stu-id="cca6e-109">Design options - Actionable recommendations, specifications, or other guidance that IT teams and developers can use when implementing the policy.</span></span>

<span data-ttu-id="cca6e-110">Следующие примеры правил политики нацелены на несколько распространенных бизнес-рисков, связанных с затратами. Вы можете использовать их как основу для разработки реальных правил политики по конкретным потребностям организации.</span><span class="sxs-lookup"><span data-stu-id="cca6e-110">The following sample policy statements address a number of common cost-related business risks, and are provided as examples for you to reference when drafting actual policy statements addressing your own organization's needs.</span></span> <span data-ttu-id="cca6e-111">Не следует рассматривать эти примеры как строгие рекомендации, ведь почти для каждого выявленного риска существует несколько возможных вариантов политик.</span><span class="sxs-lookup"><span data-stu-id="cca6e-111">These examples are not meant to be proscriptive, and there are potentially several policy options for dealing with any single identified risk.</span></span> <span data-ttu-id="cca6e-112">Лучшие решения для политик по конкретным рискам следует искать в тесном сотрудничестве с подразделениями организации и ИТ-службами.</span><span class="sxs-lookup"><span data-stu-id="cca6e-112">Work closely with business and IT teams to identify the best policy solutions for your particular cost-related risks.</span></span>  

## <a name="future-proofing"></a><span data-ttu-id="cca6e-113">Перспективность</span><span class="sxs-lookup"><span data-stu-id="cca6e-113">Future-proofing</span></span>

<span data-ttu-id="cca6e-114">**Бизнес-риск**.</span><span class="sxs-lookup"><span data-stu-id="cca6e-114">**Business risk**.</span></span> <span data-ttu-id="cca6e-115">Текущие условия не гарантируют внимания команды управления к дисциплине управления затратами.</span><span class="sxs-lookup"><span data-stu-id="cca6e-115">Current criteria that don't warrant an investment in a Cost Management discipline from the governance team.</span></span> <span data-ttu-id="cca6e-116">Но вы полагаете, что в будущем такое внимание появится.</span><span class="sxs-lookup"><span data-stu-id="cca6e-116">However, you anticipate such an investment in the future.</span></span>

<span data-ttu-id="cca6e-117">**Правило политики**.</span><span class="sxs-lookup"><span data-stu-id="cca6e-117">**Policy statement**.</span></span> <span data-ttu-id="cca6e-118">Все развернутые в облаке ресурсы следует согласовать с отделом выставления счетов, а также сопоставить с приложениями или рабочими нагрузками и проверить соблюдение стандартов именования.</span><span class="sxs-lookup"><span data-stu-id="cca6e-118">You should associate all assets deployed to the cloud with a billing unit, application/workload, and meet naming standards.</span></span> <span data-ttu-id="cca6e-119">Эта политика гарантирует эффективность будущих усилий по управлению затратами.</span><span class="sxs-lookup"><span data-stu-id="cca6e-119">This policy will ensure that future Cost Management efforts will be effective.</span></span>

<span data-ttu-id="cca6e-120">**Варианты архитектуры.**</span><span class="sxs-lookup"><span data-stu-id="cca6e-120">**Design options**.</span></span> <span data-ttu-id="cca6e-121">Сведения о создании перспективной основы можно получить в разделе обсуждений по созданию MVP управления, которые включены в инструкции по CAF в виде [практических руководств по разработке](../journeys/overview.md).</span><span class="sxs-lookup"><span data-stu-id="cca6e-121">For information on establishing a future-proof foundation, see the discussions related to creating a governance MVP in the [actionable design guides](../journeys/overview.md) included as part of the CAF guidance.</span></span>

## <a name="budget-overruns"></a><span data-ttu-id="cca6e-122">Превышения бюджета</span><span class="sxs-lookup"><span data-stu-id="cca6e-122">Budget overruns</span></span>

<span data-ttu-id="cca6e-123">**Бизнес-риск**. Самостоятельное развертывание влечет за собой риск перерасхода.</span><span class="sxs-lookup"><span data-stu-id="cca6e-123">**Business risk:** Self-service deployment creates a risk of overspending.</span></span>

<span data-ttu-id="cca6e-124">**Правило политики**. Учет всех облачных развертываний должен быть закреплен за отделом выставления счетов, имеющим утвержденный бюджет и механизм применения бюджетных ограничений.</span><span class="sxs-lookup"><span data-stu-id="cca6e-124">**Policy statement:** Any cloud deployment must be allocated to a billing unit with approved budget and a mechanism for budgetary limits.</span></span>

<span data-ttu-id="cca6e-125">**Варианты архитектуры.** В Azure управление бюджетом можно реализовать с помощью службы [Управление затратами Azure](/azure/cost-management/manage-budgets).</span><span class="sxs-lookup"><span data-stu-id="cca6e-125">**Design options:** In Azure, budget can be controlled with [Azure Cost Management](/azure/cost-management/manage-budgets)</span></span>

## <a name="underutilization"></a><span data-ttu-id="cca6e-126">Недостаточно эффективное использование</span><span class="sxs-lookup"><span data-stu-id="cca6e-126">Underutilization</span></span>

<span data-ttu-id="cca6e-127">**Бизнес-риск**. Компания заранее оплачивает облачные службы или выделила определенную сумму на год.</span><span class="sxs-lookup"><span data-stu-id="cca6e-127">**Business risk:** The company has prepaid for cloud services or has made an annual commitment to spend a specific amount.</span></span> <span data-ttu-id="cca6e-128">Есть риск, что согласованный объем ресурсов не будет использован полностью, что означает потерю инвестиций.</span><span class="sxs-lookup"><span data-stu-id="cca6e-128">There is a risk that the agreed upon amount won't be used, resulting in a lost investment.</span></span>

<span data-ttu-id="cca6e-129">**Правило политики**. Каждый отдел выставления счетов с выделенным бюджетом на облачные службы будет проводить совещания ежегодно для определения бюджетов, ежеквартально для корректировки бюджетов и ежемесячно для планирования действий по сравнению бюджетных и фактических расходов.</span><span class="sxs-lookup"><span data-stu-id="cca6e-129">**Policy statement:** Each billing unit with an allocated cloud budget will meet annually to set budgets, quarterly to adjust budgets, and monthly to allocate time for reviewing planned versus actual spending.</span></span> <span data-ttu-id="cca6e-130">Ежемесячно обсуждайте любые отклонения свыше 20 % с руководителем отдела выставления счетов.</span><span class="sxs-lookup"><span data-stu-id="cca6e-130">Discuss any deviations greater than 20% with the billing unit leader monthly.</span></span> <span data-ttu-id="cca6e-131">В целях учета передавайте данные о всех ресурсах отделу выставления счетов.</span><span class="sxs-lookup"><span data-stu-id="cca6e-131">For tracking purposes, assign all assets to a billing unit.</span></span>

<span data-ttu-id="cca6e-132">**Варианты архитектуры.**</span><span class="sxs-lookup"><span data-stu-id="cca6e-132">**Design options:**</span></span>

- <span data-ttu-id="cca6e-133">В Azure управление плановыми и фактическими расходами можно осуществлять с помощью [Управления затратами Azure](/azure/cost-management/quick-acm-cost-analysis).</span><span class="sxs-lookup"><span data-stu-id="cca6e-133">In Azure, planned versus actual spending can be managed via [Azure Cost Management](/azure/cost-management/quick-acm-cost-analysis)</span></span>
- <span data-ttu-id="cca6e-134">Существует несколько вариантов группировки ресурсов отделом выставления счетов.</span><span class="sxs-lookup"><span data-stu-id="cca6e-134">There are several options for grouping resources by billing unit.</span></span> <span data-ttu-id="cca6e-135">Совместно с командой управления следует выбрать для Azure [модель согласованности ресурсов](../../decision-guides/resource-consistency/overview.md) и применить ее ко всем ресурсам.</span><span class="sxs-lookup"><span data-stu-id="cca6e-135">In Azure, a [resource consistency model](../../decision-guides/resource-consistency/overview.md) should be chosen in conjunction with the governance team and applied to all assets.</span></span>

## <a name="overprovisioned-assets"></a><span data-ttu-id="cca6e-136">Избыточная подготовка ресурсов</span><span class="sxs-lookup"><span data-stu-id="cca6e-136">Overprovisioned assets</span></span>

<span data-ttu-id="cca6e-137">**Бизнес-риск**. В традиционных локальных центрах обработки данных часто применяется практика избыточного развертывания ресурсов с расчетом на расширение в отдаленном будущем.</span><span class="sxs-lookup"><span data-stu-id="cca6e-137">**Business risk:** In traditional on-premises datacenters, it is common practice to deploy assets with extra capacity planning for growth in the distant future.</span></span> <span data-ttu-id="cca6e-138">Облако масштабируется намного быстрее, чем обычное оборудование.</span><span class="sxs-lookup"><span data-stu-id="cca6e-138">The cloud can scale more quickly than traditional equipment.</span></span> <span data-ttu-id="cca6e-139">Кроме того, ресурсы в облаке оплачиваются пропорционально их технической мощности.</span><span class="sxs-lookup"><span data-stu-id="cca6e-139">Assets in the cloud are also priced based on the technical capacity.</span></span> <span data-ttu-id="cca6e-140">Существует риск того, что прежний подход, характерный для локальной инфраструктуры, искусственно раздует затраты на облако.</span><span class="sxs-lookup"><span data-stu-id="cca6e-140">There is a risk of the old on-premises practice artificially inflating cloud spending.</span></span>

<span data-ttu-id="cca6e-141">**Правило политики**. Любой развернутый в облаке ресурс должен быть зарегистрирован в программе, позволяющей отслеживать использование и информировать о всех мощностях, использование которых превышает 50 %.</span><span class="sxs-lookup"><span data-stu-id="cca6e-141">**Policy statement:** Any asset deployed to the cloud must be enrolled in a program that can monitor utilization and report any capacity in excess of 50% of utilization.</span></span> <span data-ttu-id="cca6e-142">Любому развернутому в облаке ресурсу следует назначить группу или тег с логичным значением, чтобы команда управления всегда могла связаться с владельцем рабочей нагрузки для обсуждения оптимизации при избыточной подготовке ресурсов.</span><span class="sxs-lookup"><span data-stu-id="cca6e-142">Any asset deployed to the cloud must be grouped or tagged in a logical manner, so governance team members can engage the workload owner regarding any optimization of overprovisioned assets.</span></span>

<span data-ttu-id="cca6e-143">**Варианты архитектуры.**</span><span class="sxs-lookup"><span data-stu-id="cca6e-143">**Design options:**</span></span>

- <span data-ttu-id="cca6e-144">В Azure рекомендации по оптимизации можно получать от [Помощника по Azure](/azure/advisor/advisor-cost-recommendations).</span><span class="sxs-lookup"><span data-stu-id="cca6e-144">In Azure, [Azure Advisor](/azure/advisor/advisor-cost-recommendations) can provide optimization recommendations.</span></span>
- <span data-ttu-id="cca6e-145">Существует несколько вариантов группировки ресурсов отделом выставления счетов.</span><span class="sxs-lookup"><span data-stu-id="cca6e-145">There are several options for grouping resources by billing unit.</span></span> <span data-ttu-id="cca6e-146">Совместно с командой управления следует выбрать для Azure [модель согласованности ресурсов](../../decision-guides/resource-consistency/overview.md) и применить ее ко всем ресурсам.</span><span class="sxs-lookup"><span data-stu-id="cca6e-146">In Azure, a [resource consistency model](../../decision-guides/resource-consistency/overview.md) should be chosen in conjunction with the governance team and applied to all assets.</span></span>

## <a name="overoptimization"></a><span data-ttu-id="cca6e-147">Избыточная оптимизация</span><span class="sxs-lookup"><span data-stu-id="cca6e-147">Overoptimization</span></span>

<span data-ttu-id="cca6e-148">**Бизнес-риск**. Эффективное управление затратами иногда создает новые риски.</span><span class="sxs-lookup"><span data-stu-id="cca6e-148">**Business risk:** Effective cost management can actually create new risks.</span></span> <span data-ttu-id="cca6e-149">Оптимизация расходов напрямую противоречит производительности системы.</span><span class="sxs-lookup"><span data-stu-id="cca6e-149">Optimization of spending is inverse to system performance.</span></span> <span data-ttu-id="cca6e-150">Снижение затрат может приводить к ухудшению взаимодействий с пользователями.</span><span class="sxs-lookup"><span data-stu-id="cca6e-150">When reducing costs, there is a risk of overtightening spending and producing poor user experiences.</span></span>

<span data-ttu-id="cca6e-151">**Правило политики**. Следует выделить с помощью групп или тегов все ресурсы, которые непосредственно влияют на взаимодействие с клиентом.</span><span class="sxs-lookup"><span data-stu-id="cca6e-151">**Policy statement:** Any asset that directly affects customer experiences must be identified through grouping or tagging.</span></span> <span data-ttu-id="cca6e-152">Прежде чем выполнять оптимизацию для любого ресурса, который влияет на взаимодействие с клиентом, команде управления облаком следует принять во внимание тенденции его использования не менее чем за 90 дней.</span><span class="sxs-lookup"><span data-stu-id="cca6e-152">Before optimizing any asset that affects customer experience, the Cloud Governance team must adjust optimization based on no fewer than 90 days of utilization trends.</span></span> <span data-ttu-id="cca6e-153">Документируйте любые пики нагрузки, связанные с сезонными изменениями или конкретными событиями, которые нужно учитывать при оптимизации.</span><span class="sxs-lookup"><span data-stu-id="cca6e-153">Document any seasonal or event driven bursts considered when optimizing assets.</span></span>

<span data-ttu-id="cca6e-154">**Варианты архитектуры.**</span><span class="sxs-lookup"><span data-stu-id="cca6e-154">**Design options:**</span></span>

- <span data-ttu-id="cca6e-155">В Azure есть [аналитические функции Azure Monitor](/azure/azure-monitor/insights/vminsights-performance), которые помогут в анализе использования системы.</span><span class="sxs-lookup"><span data-stu-id="cca6e-155">In Azure, [Azure Monitor's insights features](/azure/azure-monitor/insights/vminsights-performance) can help with analysis of system utilization.</span></span>
- <span data-ttu-id="cca6e-156">Также существует несколько методов группировки и маркировки ресурсов с учетом их ролей.</span><span class="sxs-lookup"><span data-stu-id="cca6e-156">There are several options for grouping and tagging resources based on roles.</span></span> <span data-ttu-id="cca6e-157">Совместно с группой управления следует выбрать для Azure [модель согласованности ресурсов](../../decision-guides/resource-consistency/overview.md) и применить ее ко всем ресурсам.</span><span class="sxs-lookup"><span data-stu-id="cca6e-157">In Azure, you should choose a [resource consistency model](../../decision-guides/resource-consistency/overview.md) in conjunction with the governance team and apply this to all assets.</span></span>

## <a name="next-steps"></a><span data-ttu-id="cca6e-158">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="cca6e-158">Next steps</span></span>

<span data-ttu-id="cca6e-159">Используйте примеры, приведенные в этой статье, в качестве отправной точки для разработки политик по конкретным бизнес-рискам, соответствующих вашим планам внедрения облачных систем.</span><span class="sxs-lookup"><span data-stu-id="cca6e-159">Use the samples mentioned in this article as a starting point to develop policies that address specific business risks that align with your cloud adoption plans.</span></span>

<span data-ttu-id="cca6e-160">Чтобы начать создание собственных правил пользовательской политики, имеющих отношение к управлению затратами, скачайте [шаблон управления затратами](template.md).</span><span class="sxs-lookup"><span data-stu-id="cca6e-160">To begin developing your own custom policy statements related to Cost Management, download the [Cost Management template](template.md).</span></span>

<span data-ttu-id="cca6e-161">Чтобы ускорить внедрение этой дисциплины, выберите [практические рекомендации по управлению](../journeys/overview.md), которые больше всего подходят вашей среде.</span><span class="sxs-lookup"><span data-stu-id="cca6e-161">To accelerate adoption of this discipline, choose the [Actionable Governance Journey](../journeys/overview.md) that most closely aligns with your environment.</span></span> <span data-ttu-id="cca6e-162">Затем измените архитектуру с учетом принятых решений по корпоративной политике.</span><span class="sxs-lookup"><span data-stu-id="cca6e-162">Then modify the design to incorporate your specific corporate policy decisions.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="cca6e-163">Стратегии действенного управления</span><span class="sxs-lookup"><span data-stu-id="cca6e-163">Actionable Governance Journeys</span></span>](../journeys/overview.md)