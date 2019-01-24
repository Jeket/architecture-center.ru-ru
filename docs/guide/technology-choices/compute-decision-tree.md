---
title: Дерево принятия решений для вычислительных служб Azure
titleSuffix: Azure Application Architecture Guide
description: Блок-схема выбора службы вычислений
author: MikeWasson
ms.date: 11/03/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: e3df1cbdd049e8c40597a85eca29899d8d0d1bc3
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54484658"
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="bcb07-103">Дерево принятия решений для вычислительных служб Azure</span><span class="sxs-lookup"><span data-stu-id="bcb07-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="bcb07-104">В Azure доступно несколько способов размещения кода приложения.</span><span class="sxs-lookup"><span data-stu-id="bcb07-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="bcb07-105">Термин *вычислительная служба* означает модель размещения вычислительных ресурсов, которые используются для выполнения приложения.</span><span class="sxs-lookup"><span data-stu-id="bcb07-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="bcb07-106">С помощью следующей блок-схемы можно выбрать службы вычислений для приложения.</span><span class="sxs-lookup"><span data-stu-id="bcb07-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="bcb07-107">В блок-схеме содержится набор ключевых критериев принятия решения, которые соответствуют рекомендациям.</span><span class="sxs-lookup"><span data-stu-id="bcb07-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span>

<span data-ttu-id="bcb07-108">**Используйте эту блок-схему в качестве отправной точки.**</span><span class="sxs-lookup"><span data-stu-id="bcb07-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="bcb07-109">К каждому приложению есть свои требования, поэтому в качестве отправной точки используйте рекомендации.</span><span class="sxs-lookup"><span data-stu-id="bcb07-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="bcb07-110">Затем выполните более глубокую очценку, используя такие аспекты как:</span><span class="sxs-lookup"><span data-stu-id="bcb07-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>

- <span data-ttu-id="bcb07-111">Набор возможностей</span><span class="sxs-lookup"><span data-stu-id="bcb07-111">Feature set</span></span>
- [<span data-ttu-id="bcb07-112">Ограничения служб</span><span class="sxs-lookup"><span data-stu-id="bcb07-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="bcb07-113">Стоимость</span><span class="sxs-lookup"><span data-stu-id="bcb07-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="bcb07-114">Соглашение об уровне обслуживания</span><span class="sxs-lookup"><span data-stu-id="bcb07-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="bcb07-115">Доступность по регионам</span><span class="sxs-lookup"><span data-stu-id="bcb07-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="bcb07-116">Разработка экосистемы и навыков команды</span><span class="sxs-lookup"><span data-stu-id="bcb07-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="bcb07-117">Таблицы сравнения вычислений</span><span class="sxs-lookup"><span data-stu-id="bcb07-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="bcb07-118">Если ваше приложение состоит из нескольких рабочих нагрузок, оцените каждую из них отдельно.</span><span class="sxs-lookup"><span data-stu-id="bcb07-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="bcb07-119">Полное решение можно внедрить в две или больше службы вычислений.</span><span class="sxs-lookup"><span data-stu-id="bcb07-119">A complete solution may incorporate two or more compute services.</span></span>

<span data-ttu-id="bcb07-120">См. дополнительные сведения о [вариантах размещения контейнеров в Azure](https://azure.microsoft.com/overview/containers/).</span><span class="sxs-lookup"><span data-stu-id="bcb07-120">For more information about your options for hosting containers in Azure, see [Azure Containers](https://azure.microsoft.com/overview/containers/).</span></span>

## <a name="flowchart"></a><span data-ttu-id="bcb07-121">Блок-схема</span><span class="sxs-lookup"><span data-stu-id="bcb07-121">Flowchart</span></span>

![Дерево принятия решений для вычислительных служб Azure](../images/compute-decision-tree.svg)

## <a name="definitions"></a><span data-ttu-id="bcb07-123">Определения</span><span class="sxs-lookup"><span data-stu-id="bcb07-123">Definitions</span></span>

- <span data-ttu-id="bcb07-124">**Lift-and-shift** — это стратегия по переноса рабочей нагрузки в облако без повторного проектирования приложения и внесения изменений в код.</span><span class="sxs-lookup"><span data-stu-id="bcb07-124">**Lift and shift** is a strategy for migrating a workload to the cloud without redesigning the application or making code changes.</span></span> <span data-ttu-id="bcb07-125">Также называется *повторным размещением*.</span><span class="sxs-lookup"><span data-stu-id="bcb07-125">Also called *rehosting*.</span></span> <span data-ttu-id="bcb07-126">См. дополнительные сведения о [Центре миграции Azure](https://azure.microsoft.com/migration/).</span><span class="sxs-lookup"><span data-stu-id="bcb07-126">For more information, see [Azure migration center](https://azure.microsoft.com/migration/).</span></span>

- <span data-ttu-id="bcb07-127">**Оптимизация для облака** — это стратегия переноса в облако путем рефакторинга приложения для использования преимуществ облачных функций и возможностей.</span><span class="sxs-lookup"><span data-stu-id="bcb07-127">**Cloud optimized** is a strategy for migrating to the cloud by refactoring an application to take advantage of cloud-native features and capabilities.</span></span>

## <a name="next-steps"></a><span data-ttu-id="bcb07-128">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="bcb07-128">Next steps</span></span>

<span data-ttu-id="bcb07-129">См. дополнительные сведения о [критериях для выбора вычислительной службы Azure](./compute-comparison.md).</span><span class="sxs-lookup"><span data-stu-id="bcb07-129">For additional criteria to consider, see [Criteria for choosing an Azure compute service](./compute-comparison.md).</span></span>
