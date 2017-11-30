---
title: "Шаблоны доступности"
description: "Доступность определяет долю времени, на протяжении которого система работает надлежащим образом. На нее влияют системные ошибки, проблемы инфраструктуры, вредоносные атаки и нагрузка системы. Обычно этот показатель измеряется в процентах от времени непрерывной работы. Облачные приложения обычно предоставляются пользователям вместе с соглашением об уровне обслуживания, поэтому приложения следует разрабатывать и реализовать таким образом, чтобы обеспечить их максимальную доступность."
keywords: "конструктивный шаблон"
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: f7eb6b0df388b2f1dab83e64ab540cc22f368e19
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
---
# <a name="availability-patterns"></a><span data-ttu-id="5770c-107">Шаблоны доступности</span><span class="sxs-lookup"><span data-stu-id="5770c-107">Availability patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="5770c-108">Доступность определяет долю времени, на протяжении которого система работает надлежащим образом.</span><span class="sxs-lookup"><span data-stu-id="5770c-108">Availability defines the proportion of time that the system is functional and working.</span></span> <span data-ttu-id="5770c-109">На нее влияют системные ошибки, проблемы инфраструктуры, вредоносные атаки и нагрузка системы.</span><span class="sxs-lookup"><span data-stu-id="5770c-109">It will be affected by system errors, infrastructure problems, malicious attacks, and system load.</span></span> <span data-ttu-id="5770c-110">Обычно этот показатель измеряется в процентах от времени непрерывной работы.</span><span class="sxs-lookup"><span data-stu-id="5770c-110">It is usually measured as a percentage of uptime.</span></span> <span data-ttu-id="5770c-111">Облачные приложения обычно предоставляются пользователям вместе с соглашением об уровне обслуживания, поэтому приложения следует разрабатывать и реализовать таким образом, чтобы обеспечить их максимальную доступность.</span><span class="sxs-lookup"><span data-stu-id="5770c-111">Cloud applications typically provide users with a service level agreement (SLA), which means that applications must be designed and implemented in a way that maximizes availability.</span></span>

| <span data-ttu-id="5770c-112">Модель</span><span class="sxs-lookup"><span data-stu-id="5770c-112">Pattern</span></span> | <span data-ttu-id="5770c-113">Сводка</span><span class="sxs-lookup"><span data-stu-id="5770c-113">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="5770c-114">Мониторинг конечных точек работоспособности</span><span class="sxs-lookup"><span data-stu-id="5770c-114">Health Endpoint Monitoring</span></span>](../health-endpoint-monitoring.md) | <span data-ttu-id="5770c-115">Реализация функциональных проверок в приложении, к которому внешние средства могут регулярно получать доступ через предоставленные конечные точки.</span><span class="sxs-lookup"><span data-stu-id="5770c-115">Implement functional checks in an application that external tools can access through exposed endpoints at regular intervals.</span></span> |
| [<span data-ttu-id="5770c-116">Выравнивание нагрузки на основе очередей</span><span class="sxs-lookup"><span data-stu-id="5770c-116">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md) | <span data-ttu-id="5770c-117">Очередь выполняет роль буфера между задачей и службой, которую она вызывает, позволяя сгладить кратковременные всплески нагрузки.</span><span class="sxs-lookup"><span data-stu-id="5770c-117">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span> |
| [<span data-ttu-id="5770c-118">Регулирование</span><span class="sxs-lookup"><span data-stu-id="5770c-118">Throttling</span></span>](../throttling.md) | <span data-ttu-id="5770c-119">Контролируйте потребление ресурсов, используемых экземпляром приложения, отдельным клиентом или всей службой.</span><span class="sxs-lookup"><span data-stu-id="5770c-119">Control the consumption of resources used by an instance of an application, an individual tenant, or an entire service.</span></span> |