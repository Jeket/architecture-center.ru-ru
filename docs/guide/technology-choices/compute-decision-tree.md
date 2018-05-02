---
title: Дерево принятия решений для вычислительных служб Azure
description: Блок-схема для выбора службы вычислений
author: MikeWasson
ms.date: 04/21/2018
ms.openlocfilehash: f5703f4906ca2ea6f825b383710eb4bd335f5043
ms.sourcegitcommit: d702b4d27e96e7a5a248dc4f2f0e25cf6e82c134
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/23/2018
---
# <a name="decision-tree-for-azure-compute-services"></a><span data-ttu-id="16d1d-103">Дерево принятия решений для вычислительных служб Azure</span><span class="sxs-lookup"><span data-stu-id="16d1d-103">Decision tree for Azure compute services</span></span>

<span data-ttu-id="16d1d-104">В Azure доступно несколько способов размещения кода приложения.</span><span class="sxs-lookup"><span data-stu-id="16d1d-104">Azure offers a number of ways to host your application code.</span></span> <span data-ttu-id="16d1d-105">Термин *вычислительная служба* означает модель размещения вычислительных ресурсов, которые используются для выполнения приложения.</span><span class="sxs-lookup"><span data-stu-id="16d1d-105">The term *compute* refers to the hosting model for the computing resources that your application runs on.</span></span> <span data-ttu-id="16d1d-106">С помощью следующей блок-схемы можно выбрать службы вычислений для приложения.</span><span class="sxs-lookup"><span data-stu-id="16d1d-106">The following flowchart will help you to choose a compute service for your application.</span></span>
 
![](../images/compute-decision-tree.svg)

<span data-ttu-id="16d1d-107">В блок-схеме содержится набор ключевых критериев принятия решения, которые соответствуют рекомендациям.</span><span class="sxs-lookup"><span data-stu-id="16d1d-107">The flowchart guides you through a set of key decision criteria to reach a recommendation.</span></span> <span data-ttu-id="16d1d-108">К каждому приложению есть свои требования, поэтому в качестве отправной точки необходимо использовать рекомендации.</span><span class="sxs-lookup"><span data-stu-id="16d1d-108">Every application has unique requirements, so you should treat the recommendation as a starting point.</span></span> <span data-ttu-id="16d1d-109">Затем выполните более глубокий анализ, используя такие аспекты как:</span><span class="sxs-lookup"><span data-stu-id="16d1d-109">Then perform a more detailed analysis, looking at aspects such as:</span></span>
 
- <span data-ttu-id="16d1d-110">Набор возможностей</span><span class="sxs-lookup"><span data-stu-id="16d1d-110">Feature set</span></span>
- [<span data-ttu-id="16d1d-111">Ограничения служб</span><span class="sxs-lookup"><span data-stu-id="16d1d-111">Service limits</span></span>](/azure/azure-subscription-service-limits)
- [<span data-ttu-id="16d1d-112">Стоимость</span><span class="sxs-lookup"><span data-stu-id="16d1d-112">Cost</span></span>](https://azure.microsoft.com/pricing/)
- [<span data-ttu-id="16d1d-113">Соглашение об уровне обслуживания</span><span class="sxs-lookup"><span data-stu-id="16d1d-113">SLA</span></span>](https://azure.microsoft.com/support/legal/sla/)
- [<span data-ttu-id="16d1d-114">Доступность по регионам</span><span class="sxs-lookup"><span data-stu-id="16d1d-114">Regional availability</span></span>](https://azure.microsoft.com/global-infrastructure/services/)
- <span data-ttu-id="16d1d-115">Разработка экосистемы и навыков команды</span><span class="sxs-lookup"><span data-stu-id="16d1d-115">Developer ecosystem and team skills</span></span>
- [<span data-ttu-id="16d1d-116">Таблицы сравнения вычислений</span><span class="sxs-lookup"><span data-stu-id="16d1d-116">Compute comparison tables</span></span>](./compute-comparison.md)

<span data-ttu-id="16d1d-117">Если ваше приложение состоит из нескольких рабочих нагрузок, оцените каждую из них отдельно.</span><span class="sxs-lookup"><span data-stu-id="16d1d-117">If your application consists of multiple workloads, evaluate each workload separately.</span></span> <span data-ttu-id="16d1d-118">Полное решение можно внедрить в две или больше службы вычислений.</span><span class="sxs-lookup"><span data-stu-id="16d1d-118">A complete solution may incorporate two or more compute services.</span></span>

