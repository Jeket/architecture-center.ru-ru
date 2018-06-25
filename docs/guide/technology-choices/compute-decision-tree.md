---
title: Дерево принятия решений для вычислительных служб Azure
description: Блок-схема для выбора службы вычислений
author: MikeWasson
ms.date: 06/13/2018
ms.openlocfilehash: 60bb84d4bf210888d3d43498db043b6e452f6a80
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206780"
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="67750-103">Дерево принятия решений для вычислительных служб Azure</span><span class="sxs-lookup"><span data-stu-id="67750-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="67750-104">В Azure доступно несколько способов размещения кода приложения.</span><span class="sxs-lookup"><span data-stu-id="67750-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="67750-105">Термин *вычислительная служба* означает модель размещения вычислительных ресурсов, которые используются для выполнения приложения.</span><span class="sxs-lookup"><span data-stu-id="67750-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="67750-106">С помощью следующей блок-схемы можно выбрать службы вычислений для приложения.</span><span class="sxs-lookup"><span data-stu-id="67750-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="67750-107">В блок-схеме содержится набор ключевых критериев принятия решения, которые соответствуют рекомендациям.</span><span class="sxs-lookup"><span data-stu-id="67750-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> 

<span data-ttu-id="67750-108">**Используйте эту блок-схему в качестве отправной точки.**</span><span class="sxs-lookup"><span data-stu-id="67750-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="67750-109">К каждому приложению есть свои требования, поэтому в качестве отправной точки используйте рекомендации.</span><span class="sxs-lookup"><span data-stu-id="67750-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="67750-110">Затем выполните более глубокую очценку, используя такие аспекты как:</span><span class="sxs-lookup"><span data-stu-id="67750-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>
 
- <span data-ttu-id="67750-111">Набор возможностей</span><span class="sxs-lookup"><span data-stu-id="67750-111">Feature set</span></span>
- [<span data-ttu-id="67750-112">Ограничения служб</span><span class="sxs-lookup"><span data-stu-id="67750-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="67750-113">Стоимость</span><span class="sxs-lookup"><span data-stu-id="67750-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="67750-114">Соглашение об уровне обслуживания</span><span class="sxs-lookup"><span data-stu-id="67750-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="67750-115">Доступность по регионам</span><span class="sxs-lookup"><span data-stu-id="67750-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="67750-116">Разработка экосистемы и навыков команды</span><span class="sxs-lookup"><span data-stu-id="67750-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="67750-117">Таблицы сравнения вычислений</span><span class="sxs-lookup"><span data-stu-id="67750-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="67750-118">Если ваше приложение состоит из нескольких рабочих нагрузок, оцените каждую из них отдельно.</span><span class="sxs-lookup"><span data-stu-id="67750-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="67750-119">Полное решение можно внедрить в две или больше службы вычислений.</span><span class="sxs-lookup"><span data-stu-id="67750-119">A complete solution may incorporate two or more compute services.</span></span>

## <a name="flowchart"></a><span data-ttu-id="67750-120">Блок-схема</span><span class="sxs-lookup"><span data-stu-id="67750-120">Flowchart</span></span>

![](../images/compute-decision-tree.svg)

## <a name="definitions"></a><span data-ttu-id="67750-121">Определения</span><span class="sxs-lookup"><span data-stu-id="67750-121">Definitions</span></span>

- <span data-ttu-id="67750-122">**Первичный** — проект разработки программного обеспечения, создаваемый с нуля.</span><span class="sxs-lookup"><span data-stu-id="67750-122">**Greenfield** describes a software project that is completely new and built from scratch.</span></span> <span data-ttu-id="67750-123">Он не включает существующий код.</span><span class="sxs-lookup"><span data-stu-id="67750-123">It does not include legacy code.</span></span> 

- <span data-ttu-id="67750-124">**Вторичный** — проект разработки программного обеспечения, созданный на основе существующего приложения.</span><span class="sxs-lookup"><span data-stu-id="67750-124">**Brownfield** describes a software project that builds on an existing application.</span></span> <span data-ttu-id="67750-125">Он может включать устаревший код или платформы предыдущих версий.</span><span class="sxs-lookup"><span data-stu-id="67750-125">It may inherit legacy code or frameworks.</span></span>

- <span data-ttu-id="67750-126">**Lift-and-shift** — это стратегия по переноса рабочей нагрузки в облако без повторного проектирования приложения и внесения изменений в код.</span><span class="sxs-lookup"><span data-stu-id="67750-126">**Lift and shift** is a strategy for migrating a workload to the cloud without redesigning the application or making code changes.</span></span> <span data-ttu-id="67750-127">Также называется *повторным размещением*.</span><span class="sxs-lookup"><span data-stu-id="67750-127">Also called *rehosting*.</span></span> <span data-ttu-id="67750-128">См. дополнительные сведения о [Центре миграции Azure](https://azure.microsoft.com/migration/).</span><span class="sxs-lookup"><span data-stu-id="67750-128">For more information, see [Azure migration center](https://azure.microsoft.com/migration/).</span></span>

- <span data-ttu-id="67750-129">**Оптимизация для облака** — это стратегия переноса в облако путем рефакторинга приложения для использования преимуществ облачных функций и возможностей.</span><span class="sxs-lookup"><span data-stu-id="67750-129">**Cloud optimized** is a strategy for migrating to the cloud by refactoring an application to take advantage of cloud-native features and capabilities.</span></span>

## <a name="next-steps"></a><span data-ttu-id="67750-130">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="67750-130">Next steps</span></span>

<span data-ttu-id="67750-131">См. дополнительные сведения о [критериях для выбора вычислительной службы Azure](./compute-comparison.md).</span><span class="sxs-lookup"><span data-stu-id="67750-131">For additional criteria to consider, see [Criteria for choosing an Azure compute service](./compute-comparison.md).</span></span>
