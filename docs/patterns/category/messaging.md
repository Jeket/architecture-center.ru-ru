---
title: Модели обмена сообщениями
titleSuffix: Cloud Design Patterns
description: Из-за распределенного характера облачных приложений требуется инфраструктура обмена сообщениями, которая соединяет компоненты и службы. В идеале нужно реализовать слабую связь, чтобы максимизировать масштабируемость. Асинхронный обмен сообщениями широко используется и предоставляет множество преимуществ. Но он тоже не лишен проблем, например с упорядочением сообщений, управлением сообщениями о сбое, идемпотентностью и т. д.
keywords: Конструктивный шаблон
author: dragon119
ms.date: 03/01/2018
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: f838288e10acf9af5776c93972028b6b878bd7b0
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640674"
---
# <a name="messaging-patterns"></a><span data-ttu-id="06e34-105">Модели обмена сообщениями</span><span class="sxs-lookup"><span data-stu-id="06e34-105">Messaging patterns</span></span>

<span data-ttu-id="06e34-106">Из-за распределенного характера облачных приложений требуется инфраструктура обмена сообщениями, которая соединяет компоненты и службы. В идеале нужно реализовать слабую связь, чтобы максимизировать масштабируемость.</span><span class="sxs-lookup"><span data-stu-id="06e34-106">The distributed nature of cloud applications requires a messaging infrastructure that connects the components and services, ideally in a loosely coupled manner in order to maximize scalability.</span></span> <span data-ttu-id="06e34-107">Асинхронный обмен сообщениями широко используется и предоставляет множество преимуществ. Но он тоже не лишен проблем, например с упорядочением сообщений, управлением сообщениями о сбое, идемпотентностью и т. д.</span><span class="sxs-lookup"><span data-stu-id="06e34-107">Asynchronous messaging is widely used, and provides many benefits, but also brings challenges such as the ordering of messages, poison message management, idempotency, and more.</span></span>

| <span data-ttu-id="06e34-108">Модель</span><span class="sxs-lookup"><span data-stu-id="06e34-108">Pattern</span></span> | <span data-ttu-id="06e34-109">Сводка</span><span class="sxs-lookup"><span data-stu-id="06e34-109">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="06e34-110">Проверка утверждений</span><span class="sxs-lookup"><span data-stu-id="06e34-110">Claim Check</span></span>](../claim-check.md) | <span data-ttu-id="06e34-111">Разделение большого сообщения на схему проверки утверждений и полезную нагрузку, чтобы избежать перегрузки в канале сообщений.</span><span class="sxs-lookup"><span data-stu-id="06e34-111">Split a large message into a claim check and a payload to avoid overwhelming a message bus.</span></span> |
| [<span data-ttu-id="06e34-112">Конкурирующие потребители</span><span class="sxs-lookup"><span data-stu-id="06e34-112">Competing Consumers</span></span>](../competing-consumers.md) | <span data-ttu-id="06e34-113">Несколько конкурирующих потребителей обрабатывают сообщения, полученные в одном канале обмена сообщениями.</span><span class="sxs-lookup"><span data-stu-id="06e34-113">Enable multiple concurrent consumers to process messages received on the same messaging channel.</span></span> |
| [<span data-ttu-id="06e34-114">Каналы и фильтры</span><span class="sxs-lookup"><span data-stu-id="06e34-114">Pipes and Filters</span></span>](../pipes-and-filters.md) | <span data-ttu-id="06e34-115">Задача, которая требует сложной обработки, разбивается на ряд отдельных элементов, которые можно использовать повторно.</span><span class="sxs-lookup"><span data-stu-id="06e34-115">Break down a task that performs complex processing into a series of separate elements that can be reused.</span></span> |
| [<span data-ttu-id="06e34-116">Очередь с приоритетом</span><span class="sxs-lookup"><span data-stu-id="06e34-116">Priority Queue</span></span>](../priority-queue.md) | <span data-ttu-id="06e34-117">Запросы, отправляемые в службу, получают разные приоритеты. При этом запросы с высоким приоритетом принимаются и обрабатываются быстрее, чем запросы с низким приоритетом.</span><span class="sxs-lookup"><span data-stu-id="06e34-117">Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority.</span></span> |
| [<span data-ttu-id="06e34-118">Издатель и подписчик</span><span class="sxs-lookup"><span data-stu-id="06e34-118">Publisher-Subscriber</span></span>](../publisher-subscriber.md) | <span data-ttu-id="06e34-119">Позволяют приложению информировать о событиях, чтобы несколько потребителей заинтересованные асинхронно, без взаимозависимость отправителями получателям.</span><span class="sxs-lookup"><span data-stu-id="06e34-119">Enable an application to announce events to multiple interested consumers asynchronously, without coupling the senders to the receivers.</span></span> |
| [<span data-ttu-id="06e34-120">Выравнивание нагрузки на основе очередей</span><span class="sxs-lookup"><span data-stu-id="06e34-120">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md) | <span data-ttu-id="06e34-121">Очередь выполняет роль буфера между задачей и службой, которую она вызывает, позволяя сгладить кратковременные всплески нагрузки.</span><span class="sxs-lookup"><span data-stu-id="06e34-121">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span> |
| [<span data-ttu-id="06e34-122">Планировщик, агент, контролер</span><span class="sxs-lookup"><span data-stu-id="06e34-122">Scheduler Agent Supervisor</span></span>](../scheduler-agent-supervisor.md) | <span data-ttu-id="06e34-123">Координируйте ряд действий в распределенном наборе служб и других удаленных ресурсов.</span><span class="sxs-lookup"><span data-stu-id="06e34-123">Coordinate a set of actions across a distributed set of services and other remote resources.</span></span> |
