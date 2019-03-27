---
title: Шаблоны доступности
titleSuffix: Cloud Design Patterns
description: Доступность определяет долю времени, на протяжении которого система работает надлежащим образом. На нее влияют системные ошибки, проблемы инфраструктуры, вредоносные атаки и нагрузка системы. Обычно этот показатель измеряется в процентах от времени непрерывной работы. Облачные приложения обычно предоставляются пользователям вместе с соглашением об уровне обслуживания, поэтому приложения следует разрабатывать и реализовать таким образом, чтобы обеспечить их максимальную доступность.
keywords: Конструктивный шаблон
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 897bc3c87beb23770220ef0fc3c5c4394d192f2a
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243045"
---
# <a name="availability-patterns"></a><span data-ttu-id="099b0-107">Шаблоны доступности</span><span class="sxs-lookup"><span data-stu-id="099b0-107">Availability patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="099b0-108">Доступность определяет долю времени, на протяжении которого система работает надлежащим образом.</span><span class="sxs-lookup"><span data-stu-id="099b0-108">Availability defines the proportion of time that the system is functional and working.</span></span> <span data-ttu-id="099b0-109">На нее влияют системные ошибки, проблемы инфраструктуры, вредоносные атаки и нагрузка системы.</span><span class="sxs-lookup"><span data-stu-id="099b0-109">It will be affected by system errors, infrastructure problems, malicious attacks, and system load.</span></span> <span data-ttu-id="099b0-110">Обычно этот показатель измеряется в процентах от времени непрерывной работы.</span><span class="sxs-lookup"><span data-stu-id="099b0-110">It is usually measured as a percentage of uptime.</span></span> <span data-ttu-id="099b0-111">Облачные приложения обычно предоставляются пользователям вместе с соглашением об уровне обслуживания, поэтому приложения следует разрабатывать и реализовать таким образом, чтобы обеспечить их максимальную доступность.</span><span class="sxs-lookup"><span data-stu-id="099b0-111">Cloud applications typically provide users with a service level agreement (SLA), which means that applications must be designed and implemented in a way that maximizes availability.</span></span>

|                            <span data-ttu-id="099b0-112">Модель</span><span class="sxs-lookup"><span data-stu-id="099b0-112">Pattern</span></span>                             |                                                           <span data-ttu-id="099b0-113">Сводка</span><span class="sxs-lookup"><span data-stu-id="099b0-113">Summary</span></span>                                                            |
|----------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| [<span data-ttu-id="099b0-114">Мониторинг конечных точек работоспособности</span><span class="sxs-lookup"><span data-stu-id="099b0-114">Health Endpoint Monitoring</span></span>](../health-endpoint-monitoring.md) | <span data-ttu-id="099b0-115">Внедрение в приложение функциональных проверок, к которым внешние средства могут регулярно получать доступ через предоставленные конечные точки.</span><span class="sxs-lookup"><span data-stu-id="099b0-115">Implement functional checks in an application that external tools can access through exposed endpoints at regular intervals.</span></span> |
|  [<span data-ttu-id="099b0-116">Выравнивание нагрузки на основе очередей</span><span class="sxs-lookup"><span data-stu-id="099b0-116">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md)  | <span data-ttu-id="099b0-117">Очередь выполняет роль буфера между задачей и службой, которую она вызывает, позволяя сгладить кратковременные всплески нагрузки.</span><span class="sxs-lookup"><span data-stu-id="099b0-117">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span>  |
|                 [<span data-ttu-id="099b0-118">Регулирование</span><span class="sxs-lookup"><span data-stu-id="099b0-118">Throttling</span></span>](../throttling.md)                 |   <span data-ttu-id="099b0-119">Контролируйте потребление ресурсов, используемых экземпляром приложения, отдельным клиентом или всей службой.</span><span class="sxs-lookup"><span data-stu-id="099b0-119">Control the consumption of resources used by an instance of an application, an individual tenant, or an entire service.</span></span>    |
