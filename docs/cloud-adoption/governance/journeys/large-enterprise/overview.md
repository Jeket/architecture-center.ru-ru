---
title: CAF. Стратегия развития системы управления для крупных предприятий
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Стратегия развития системы управления для крупных предприятий
author: BrianBlanchard
ms.openlocfilehash: 689b600524c3be6c505ade8c5aaa447d772c6e35
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901332"
---
# <a name="caf-large-enterprise-governance-journey"></a><span data-ttu-id="f7a8e-103">CAF. Стратегия развития системы управления для крупных предприятий</span><span class="sxs-lookup"><span data-stu-id="f7a8e-103">CAF: Large enterprise governance journey</span></span>

## <a name="best-practice-overview"></a><span data-ttu-id="f7a8e-104">Обзор рекомендаций</span><span class="sxs-lookup"><span data-stu-id="f7a8e-104">Best practice overview</span></span>

<span data-ttu-id="f7a8e-105">Путь развития показан на примере вымышленной компании на различных этапах развития системы управления.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-105">This governance journey follows the experiences of a fictional company through various stages of governance maturity.</span></span> <span data-ttu-id="f7a8e-106">Он основан на реальных путях взаимодействия с клиентом.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-106">It is based on real customer journeys.</span></span> <span data-ttu-id="f7a8e-107">Предлагаемые рекомендации основаны на ограничениях и потребностях вымышленной компании.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-107">The suggested best practices are based on the constraints and needs of the fictional company.</span></span>

<span data-ttu-id="f7a8e-108">Чтобы предоставить вам быструю отправную точку, в этом обзоре определен минимально жизнеспособный продукт (MVP) системы управления, основанный на рекомендациях.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-108">As a quick starting point, this overview defines a minimum viable product (MVP) for governance based on best practices.</span></span> <span data-ttu-id="f7a8e-109">Здесь также содержатся ссылки на данные о развитии системы управления, которые позволяют получить рекомендации по мере возникновения новых бизнес-рисков или технических рисков.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-109">It also provides links to some governance evolutions that add further best practices as new business or technical risks emerge.</span></span>

> [!WARNING]
> <span data-ttu-id="f7a8e-110">Этот MVP — это базовая начальная точка, для которой учитываются определенные допущения.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-110">This MVP is a baseline starting point, based on a set of assumptions.</span></span> <span data-ttu-id="f7a8e-111">Даже этот минимальный набор рекомендаций основывается на корпоративных политиках, созданных с учетом уникальных бизнес-рисков и допустимых рисков.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-111">Even this minimal set of best practices is based on corporate policies driven by unique business risks and risk tolerances.</span></span> <span data-ttu-id="f7a8e-112">Чтобы понять, относятся ли эти предположения к вам, ознакомьтесь с [подробным описанием](./narrative.md), относящимся к этой статье.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-112">To see if these assumptions apply to you, read the [longer narrative](./narrative.md) that follows this article.</span></span>

### <a name="governance-best-practice"></a><span data-ttu-id="f7a8e-113">Рекомендация по системе управления</span><span class="sxs-lookup"><span data-stu-id="f7a8e-113">Governance best practice</span></span>

<span data-ttu-id="f7a8e-114">Эта рекомендации служат в качестве основы, которую организация может использовать, чтобы быстро и согласованно добавлять защитные структуры системы управления в несколько подписок Azure.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-114">This best practice serves as a foundation that an organization can use to quickly and consistently add governance guardrails across multiple Azure subscriptions.</span></span>

### <a name="resource-organization"></a><span data-ttu-id="f7a8e-115">Организация ресурсов</span><span class="sxs-lookup"><span data-stu-id="f7a8e-115">Resource organization</span></span>

<span data-ttu-id="f7a8e-116">В примере ниже показана иерархия MVP системы управления для организации ресурсов.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-116">The following diagram shows the governance MVP hierarchy for organizing resources.</span></span>

![Схема организации ресурсов](../../../_images/governance/resource-organization.png)

<span data-ttu-id="f7a8e-118">Каждое приложение необходимо развернуть в соответствующей области иерархии группы управления, подписки и группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-118">Every application should be deployed in the proper area of the management group, subscription, and resource group hierarchy.</span></span> <span data-ttu-id="f7a8e-119">Во время планирования развертывания команда системы управления облачными решениями создаст в иерархии необходимые узлы для работы команд по внедрению облака.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-119">During deployment planning, the Cloud Governance team will create the necessary nodes in the hierarchy to empower the cloud adoption teams.</span></span>

1. <span data-ttu-id="f7a8e-120">Группа управления для каждого бизнес-подразделения с детальной иерархией, которая отражает географический регион, а затем тип среды (рабочая среда или нерабочая среда).</span><span class="sxs-lookup"><span data-stu-id="f7a8e-120">A management group for each business unit with a detailed hierarchy that reflects geography then environment type (Production, Non-Production).</span></span>
2. <span data-ttu-id="f7a8e-121">Подписка для каждого уникального сочетания подразделения, географического региона, среды и классификации приложения.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-121">A subscription for each unique combination of business unit, geography, environment, and "Application Categorization."</span></span>
3. <span data-ttu-id="f7a8e-122">Отдельная группа ресурсов для каждого приложения.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-122">A separate resource group for each application.</span></span>
4. <span data-ttu-id="f7a8e-123">На каждом уровне иерархии группирования должна применяться согласованная номенклатура.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-123">Consistent nomenclature should be applied at each level of this grouping hierarchy.</span></span>

![Схема организации ресурсов на крупных предприятиях](../../../_images/governance/large-enterprise-resource-organization.png)

<span data-ttu-id="f7a8e-125">Эти шаблоны предоставляют пространство для роста без лишних усложнений иерархии.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-125">These patterns provide room for growth without complicating the hierarchy unnecessarily.</span></span>

[!INCLUDE [governance-of-resources](../../../../../includes/cloud-adoption/governance/governance-of-resources.md)]

## <a name="governance-evolutions"></a><span data-ttu-id="f7a8e-126">Развитие процесса управления</span><span class="sxs-lookup"><span data-stu-id="f7a8e-126">Governance evolutions</span></span>

<span data-ttu-id="f7a8e-127">После развертывания MVP в среду можно быстро интегрировать дополнительные уровни управления.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-127">Once this MVP has been deployed, additional layers of governance can be quickly incorporated into the environment.</span></span> <span data-ttu-id="f7a8e-128">Ниже приведены некоторые способы развития MVP в соответствии с потребностями конкретной организации.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-128">Here are some ways to evolve the MVP to meet specific business needs:</span></span>

- [<span data-ttu-id="f7a8e-129">Базовая система безопасности — защищенные данные</span><span class="sxs-lookup"><span data-stu-id="f7a8e-129">Security Baseline for protected data</span></span>](./security-baseline-evolution.md)
- [<span data-ttu-id="f7a8e-130">Настройки ресурсов для критически важных приложений</span><span class="sxs-lookup"><span data-stu-id="f7a8e-130">Resource configurations for mission-critical applications</span></span>](./resource-consistency-evolution.md)
- [<span data-ttu-id="f7a8e-131">Элементы управления для Управления затратами</span><span class="sxs-lookup"><span data-stu-id="f7a8e-131">Controls for Cost Management</span></span>](./cost-management-evolution.md)
- [<span data-ttu-id="f7a8e-132">Элементы управления для развития использования нескольких облаков</span><span class="sxs-lookup"><span data-stu-id="f7a8e-132">Controls for multi-cloud evolution</span></span>](./multi-cloud-evolution.md)

<!-- markdownlint-disable MD026 -->

## <a name="what-does-this-best-practice-do"></a><span data-ttu-id="f7a8e-133">Возможности, которые предоставляет эта рекомендация</span><span class="sxs-lookup"><span data-stu-id="f7a8e-133">What does this best practice do?</span></span>

<span data-ttu-id="f7a8e-134">В MVP устанавливаются рекомендации и средства дисциплины [ускорения развертывания](../../deployment-acceleration/overview.md), чтобы быстро применять корпоративную политику.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-134">In the MVP, practices and tools from the [Deployment Acceleration](../../deployment-acceleration/overview.md) discipline are established to quickly apply corporate policy.</span></span> <span data-ttu-id="f7a8e-135">В частности MVP использует Azure Blueprints, Политику Azure и группы управления Azure для применения нескольких основных корпоративных политик, как определено в описании этой вымышленной компании.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-135">In particular, the MVP uses Azure Blueprints, Azure Policy, and Azure management groups to apply a few basic corporate policies, as defined in the narrative for this fictional company.</span></span> <span data-ttu-id="f7a8e-136">Эти корпоративные политики применяются с использованием шаблонов Azure Resource Manager и политик Azure для установления очень малых базовых показателей идентификации и безопасности.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-136">Those corporate policies are applied using Azure Resource Manager templates and Azure policies to establish a very small baseline for identity and security.</span></span>

![Пример минимально жизнеспособного продукта для системы управления инкрементальной модели](../../../_images/governance/governance-mvp.png)

## <a name="evolving-the-best-practice"></a><span data-ttu-id="f7a8e-138">Расширение рекомендаций</span><span class="sxs-lookup"><span data-stu-id="f7a8e-138">Evolving the best practice</span></span>

<span data-ttu-id="f7a8e-139">Со временем этот MVP системы управления будет использоваться для расширения рекомендаций по управлению.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-139">Over time, this governance MVP will be used to evolve the governance practices.</span></span> <span data-ttu-id="f7a8e-140">По мере внедрения растут риски для бизнеса.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-140">As adoption advances, business risk grows.</span></span> <span data-ttu-id="f7a8e-141">Для снижения этих рисков модель системы управления CAF будет развиваться.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-141">Various disciplines within the CAF governance model will evolve to mitigate those risks.</span></span> <span data-ttu-id="f7a8e-142">В более поздних статьях этой серии рассматривается развитие корпоративной политики, влияющей на вымышленную компанию.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-142">Later articles in this series discuss the evolution of corporate policy affecting the fictional company.</span></span> <span data-ttu-id="f7a8e-143">Это развитие возникает в трех различных дисциплинах:</span><span class="sxs-lookup"><span data-stu-id="f7a8e-143">These evolutions happen across three disciplines:</span></span>

- <span data-ttu-id="f7a8e-144">"Базовая система идентификации" по мере развития зависимостей миграции в описании концепции.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-144">Identity Baseline, as migration dependencies evolve in the narrative</span></span>
- <span data-ttu-id="f7a8e-145">"Управление затратами" по мере масштабирования внедрения.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-145">Cost Management, as adoption scales.</span></span>
- <span data-ttu-id="f7a8e-146">"Базовая система безопасности" по мере развертывания защищенных данных.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-146">Security Baseline, as protected data is deployed.</span></span>
- <span data-ttu-id="f7a8e-147">"Согласованность ресурсов" по мере поддержки критически важных рабочих нагрузок отделом ИТ-операций.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-147">Resource Consistency, as IT Operations begins supporting mission-critical workloads.</span></span>

![Пример минимально жизнеспособного продукта для системы управления инкрементальной модели](../../../_images/governance/governance-evolution-large.png)

## <a name="next-steps"></a><span data-ttu-id="f7a8e-149">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="f7a8e-149">Next steps</span></span>

<span data-ttu-id="f7a8e-150">Теперь, когда вы знакомы с MVP системы управления и знаете о развитии системы управления, ознакомьтесь с дополнительной статьей для контекста.</span><span class="sxs-lookup"><span data-stu-id="f7a8e-150">Now that you’re familiar with the governance MVP and have an idea of the governance evolutions to follow, read the supporting narrative for additional context.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="f7a8e-151">Читать вспомогательное описание</span><span class="sxs-lookup"><span data-stu-id="f7a8e-151">Read the supporting narrative</span></span>](./narrative.md)
