---
title: Дерево принятия решений для вычислительных служб Azure
description: Блок-схема для выбора службы вычислений
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: e601dcb653ed1809ea3f9bbda8db8b40efb460a5
ms.sourcegitcommit: 3846a0ab2b2b2552202a3c9c21af0097a145ffc6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/29/2018
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="e5244-103">Дерево принятия решений для вычислительных служб Azure</span><span class="sxs-lookup"><span data-stu-id="e5244-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="e5244-104">В Azure доступно несколько способов размещения кода приложения.</span><span class="sxs-lookup"><span data-stu-id="e5244-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="e5244-105">Термин *вычислительная служба* означает модель размещения вычислительных ресурсов, которые используются для выполнения приложения.</span><span class="sxs-lookup"><span data-stu-id="e5244-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="e5244-106">С помощью следующей блок-схемы можно выбрать службы вычислений для приложения.</span><span class="sxs-lookup"><span data-stu-id="e5244-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="e5244-107">В блок-схеме содержится набор ключевых критериев принятия решения, которые соответствуют рекомендациям.</span><span class="sxs-lookup"><span data-stu-id="e5244-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> 

<span data-ttu-id="e5244-108">**Используйте эту блок-схему в качестве отправной точки.**</span><span class="sxs-lookup"><span data-stu-id="e5244-108">**Treat this flowchart as a stating point.**</span></span> <span data-ttu-id="e5244-109">К каждому приложению есть свои требования, поэтому в качестве отправной точки используйте рекомендации.</span><span class="sxs-lookup"><span data-stu-id="e5244-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="e5244-110">Затем выполните более глубокую очценку, используя такие аспекты как:</span><span class="sxs-lookup"><span data-stu-id="e5244-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>
 
- <span data-ttu-id="e5244-111">Набор возможностей</span><span class="sxs-lookup"><span data-stu-id="e5244-111">Feature set</span></span>
- [<span data-ttu-id="e5244-112">Ограничения служб</span><span class="sxs-lookup"><span data-stu-id="e5244-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="e5244-113">Стоимость</span><span class="sxs-lookup"><span data-stu-id="e5244-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="e5244-114">Соглашение об уровне обслуживания</span><span class="sxs-lookup"><span data-stu-id="e5244-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="e5244-115">Доступность по регионам</span><span class="sxs-lookup"><span data-stu-id="e5244-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="e5244-116">Разработка экосистемы и навыков команды</span><span class="sxs-lookup"><span data-stu-id="e5244-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="e5244-117">Таблицы сравнения вычислений</span><span class="sxs-lookup"><span data-stu-id="e5244-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="e5244-118">Если ваше приложение состоит из нескольких рабочих нагрузок, оцените каждую из них отдельно.</span><span class="sxs-lookup"><span data-stu-id="e5244-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="e5244-119">Полное решение можно внедрить в две или больше службы вычислений.</span><span class="sxs-lookup"><span data-stu-id="e5244-119">A complete solution may incorporate two or more compute services.</span></span>

![](../images/compute-decision-tree.svg)

