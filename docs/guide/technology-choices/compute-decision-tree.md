---
title: Дерево принятия решений для вычислительных служб Azure
description: Блок-схема для выбора службы вычислений
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: 3dcfbd156d4fced863a56bcc8bb74483aa665f9f
ms.sourcegitcommit: 7ced70ebc11aa0df0dc0104092d3cc6ad5c28bd6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/11/2018
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="0b6ef-103">Дерево принятия решений для вычислительных служб Azure</span><span class="sxs-lookup"><span data-stu-id="0b6ef-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="0b6ef-104">В Azure доступно несколько способов размещения кода приложения.</span><span class="sxs-lookup"><span data-stu-id="0b6ef-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="0b6ef-105">Термин *вычислительная служба* означает модель размещения вычислительных ресурсов, которые используются для выполнения приложения.</span><span class="sxs-lookup"><span data-stu-id="0b6ef-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="0b6ef-106">С помощью следующей блок-схемы можно выбрать службы вычислений для приложения.</span><span class="sxs-lookup"><span data-stu-id="0b6ef-106">The following flowchart will help you to choose a compute service for your application.</span></span> <span data-ttu-id="0b6ef-107">В блок-схеме содержится набор ключевых критериев принятия решения, которые соответствуют рекомендациям.</span><span class="sxs-lookup"><span data-stu-id="0b6ef-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> 

<span data-ttu-id="0b6ef-108">**Используйте эту блок-схему в качестве отправной точки.**</span><span class="sxs-lookup"><span data-stu-id="0b6ef-108">**Treat this flowchart as a starting point.**</span></span> <span data-ttu-id="0b6ef-109">К каждому приложению есть свои требования, поэтому в качестве отправной точки используйте рекомендации.</span><span class="sxs-lookup"><span data-stu-id="0b6ef-109">Every application has unique requirements, so use the recommendation as a starting point.</span></span> <span data-ttu-id="0b6ef-110">Затем выполните более глубокую очценку, используя такие аспекты как:</span><span class="sxs-lookup"><span data-stu-id="0b6ef-110">Then perform a more detailed evaluation, looking at aspects such as:</span></span>
 
- <span data-ttu-id="0b6ef-111">Набор возможностей</span><span class="sxs-lookup"><span data-stu-id="0b6ef-111">Feature set</span></span>
- [<span data-ttu-id="0b6ef-112">Ограничения служб</span><span class="sxs-lookup"><span data-stu-id="0b6ef-112">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="0b6ef-113">Стоимость</span><span class="sxs-lookup"><span data-stu-id="0b6ef-113">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="0b6ef-114">Соглашение об уровне обслуживания</span><span class="sxs-lookup"><span data-stu-id="0b6ef-114">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="0b6ef-115">Доступность по регионам</span><span class="sxs-lookup"><span data-stu-id="0b6ef-115">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="0b6ef-116">Разработка экосистемы и навыков команды</span><span class="sxs-lookup"><span data-stu-id="0b6ef-116">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="0b6ef-117">Таблицы сравнения вычислений</span><span class="sxs-lookup"><span data-stu-id="0b6ef-117">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="0b6ef-118">Если ваше приложение состоит из нескольких рабочих нагрузок, оцените каждую из них отдельно.</span><span class="sxs-lookup"><span data-stu-id="0b6ef-118">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="0b6ef-119">Полное решение можно внедрить в две или больше службы вычислений.</span><span class="sxs-lookup"><span data-stu-id="0b6ef-119">A complete solution may incorporate two or more compute services.</span></span>

![](../images/compute-decision-tree.svg)

