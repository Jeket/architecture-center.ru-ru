---
title: Модели обмена сообщениями
description: Из-за распределенного характера облачных приложений требуется инфраструктура обмена сообщениями, которая соединяет компоненты и службы. В идеале нужно реализовать слабую связь, чтобы максимизировать масштабируемость. Асинхронный обмен сообщениями широко используется и предоставляет множество преимуществ. Но он тоже не лишен проблем, например с упорядочением сообщений, управлением сообщениями о сбое, идемпотентностью и т. д.
keywords: Конструктивный шаблон
author: dragon119
ms.date: 12/07/2018
pnp.series.title: Cloud Design Patterns
ms.openlocfilehash: 754e9a7dcad20dc1c4471af00f3f142f18022d62
ms.sourcegitcommit: a0a9981e7586bed8d876a54e055dea1e392118f8
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/11/2018
ms.locfileid: "53233886"
---
# <a name="messaging-patterns"></a><span data-ttu-id="5fb55-105">Модели обмена сообщениями</span><span class="sxs-lookup"><span data-stu-id="5fb55-105">Messaging patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="5fb55-106">Из-за распределенного характера облачных приложений требуется инфраструктура обмена сообщениями, которая соединяет компоненты и службы. В идеале нужно реализовать слабую связь, чтобы максимизировать масштабируемость.</span><span class="sxs-lookup"><span data-stu-id="5fb55-106">The distributed nature of cloud applications requires a messaging infrastructure that connects the components and services, ideally in a loosely coupled manner in order to maximize scalability.</span></span> <span data-ttu-id="5fb55-107">Асинхронный обмен сообщениями широко используется и предоставляет множество преимуществ. Но он тоже не лишен проблем, например с упорядочением сообщений, управлением сообщениями о сбое, идемпотентностью и т. д.</span><span class="sxs-lookup"><span data-stu-id="5fb55-107">Asynchronous messaging is widely used, and provides many benefits, but also brings challenges such as the ordering of messages, poison message management, idempotency, and more.</span></span>

| <span data-ttu-id="5fb55-108">Модель</span><span class="sxs-lookup"><span data-stu-id="5fb55-108">Pattern</span></span> | <span data-ttu-id="5fb55-109">Сводка</span><span class="sxs-lookup"><span data-stu-id="5fb55-109">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="5fb55-110">Конкурирующие потребители</span><span class="sxs-lookup"><span data-stu-id="5fb55-110">Competing Consumers</span></span>](../competing-consumers.md) | <span data-ttu-id="5fb55-111">Несколько конкурирующих потребителей обрабатывают сообщения, полученные в одном канале обмена сообщениями.</span><span class="sxs-lookup"><span data-stu-id="5fb55-111">Enable multiple concurrent consumers to process messages received on the same messaging channel.</span></span> |
| [<span data-ttu-id="5fb55-112">Каналы и фильтры</span><span class="sxs-lookup"><span data-stu-id="5fb55-112">Pipes and Filters</span></span>](../pipes-and-filters.md) | <span data-ttu-id="5fb55-113">Задача, которая требует сложной обработки, разбивается на ряд отдельных элементов, которые можно использовать повторно.</span><span class="sxs-lookup"><span data-stu-id="5fb55-113">Break down a task that performs complex processing into a series of separate elements that can be reused.</span></span> |
| [<span data-ttu-id="5fb55-114">Очередь с приоритетом</span><span class="sxs-lookup"><span data-stu-id="5fb55-114">Priority Queue</span></span>](../priority-queue.md) | <span data-ttu-id="5fb55-115">Запросы, отправляемые в службу, получают разные приоритеты. При этом запросы с высоким приоритетом принимаются и обрабатываются быстрее, чем запросы с низким приоритетом.</span><span class="sxs-lookup"><span data-stu-id="5fb55-115">Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority.</span></span> |
| [<span data-ttu-id="5fb55-116">Издатель и подписчик</span><span class="sxs-lookup"><span data-stu-id="5fb55-116">Publisher-Subscriber</span></span>](../publisher-subscriber.md) | <span data-ttu-id="5fb55-117">Настройка в приложении возможности объявлять событие для нескольких объектов-получателей одновременно без взаимозависимости между отправителями и получателями.</span><span class="sxs-lookup"><span data-stu-id="5fb55-117">Enable an application to announce events to multiple interested consumers aynchronously, without coupling the senders to the receivers.</span></span> |
| [<span data-ttu-id="5fb55-118">Выравнивание нагрузки на основе очередей</span><span class="sxs-lookup"><span data-stu-id="5fb55-118">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md) | <span data-ttu-id="5fb55-119">Очередь выполняет роль буфера между задачей и службой, которую она вызывает, позволяя сгладить кратковременные всплески нагрузки.</span><span class="sxs-lookup"><span data-stu-id="5fb55-119">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span> |
| [<span data-ttu-id="5fb55-120">Планировщик, агент, контролер</span><span class="sxs-lookup"><span data-stu-id="5fb55-120">Scheduler Agent Supervisor</span></span>](../scheduler-agent-supervisor.md) | <span data-ttu-id="5fb55-121">Координируйте ряд действий в распределенном наборе служб и других удаленных ресурсов.</span><span class="sxs-lookup"><span data-stu-id="5fb55-121">Coordinate a set of actions across a distributed set of services and other remote resources.</span></span> |
