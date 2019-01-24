---
title: Модели обмена сообщениями
titleSuffix: Cloud Design Patterns
description: Из-за распределенного характера облачных приложений требуется инфраструктура обмена сообщениями, которая соединяет компоненты и службы. В идеале нужно реализовать слабую связь, чтобы максимизировать масштабируемость. Асинхронный обмен сообщениями широко используется и предоставляет множество преимуществ. Но он тоже не лишен проблем, например с упорядочением сообщений, управлением сообщениями о сбое, идемпотентностью и т. д.
keywords: Конструктивный шаблон
author: dragon119
ms.date: 12/07/2018
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 2f8a34afc5e6c427bf5635b7a3be316719211112
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485168"
---
# <a name="messaging-patterns"></a><span data-ttu-id="46ed4-105">Модели обмена сообщениями</span><span class="sxs-lookup"><span data-stu-id="46ed4-105">Messaging patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="46ed4-106">Из-за распределенного характера облачных приложений требуется инфраструктура обмена сообщениями, которая соединяет компоненты и службы. В идеале нужно реализовать слабую связь, чтобы максимизировать масштабируемость.</span><span class="sxs-lookup"><span data-stu-id="46ed4-106">The distributed nature of cloud applications requires a messaging infrastructure that connects the components and services, ideally in a loosely coupled manner in order to maximize scalability.</span></span> <span data-ttu-id="46ed4-107">Асинхронный обмен сообщениями широко используется и предоставляет множество преимуществ. Но он тоже не лишен проблем, например с упорядочением сообщений, управлением сообщениями о сбое, идемпотентностью и т. д.</span><span class="sxs-lookup"><span data-stu-id="46ed4-107">Asynchronous messaging is widely used, and provides many benefits, but also brings challenges such as the ordering of messages, poison message management, idempotency, and more.</span></span>

| <span data-ttu-id="46ed4-108">Модель</span><span class="sxs-lookup"><span data-stu-id="46ed4-108">Pattern</span></span> | <span data-ttu-id="46ed4-109">Сводка</span><span class="sxs-lookup"><span data-stu-id="46ed4-109">Summary</span></span> |
| ------- | ------- |
| [<span data-ttu-id="46ed4-110">Конкурирующие потребители</span><span class="sxs-lookup"><span data-stu-id="46ed4-110">Competing Consumers</span></span>](../competing-consumers.md) | <span data-ttu-id="46ed4-111">Несколько конкурирующих потребителей обрабатывают сообщения, полученные в одном канале обмена сообщениями.</span><span class="sxs-lookup"><span data-stu-id="46ed4-111">Enable multiple concurrent consumers to process messages received on the same messaging channel.</span></span> |
| [<span data-ttu-id="46ed4-112">Каналы и фильтры</span><span class="sxs-lookup"><span data-stu-id="46ed4-112">Pipes and Filters</span></span>](../pipes-and-filters.md) | <span data-ttu-id="46ed4-113">Задача, которая требует сложной обработки, разбивается на ряд отдельных элементов, которые можно использовать повторно.</span><span class="sxs-lookup"><span data-stu-id="46ed4-113">Break down a task that performs complex processing into a series of separate elements that can be reused.</span></span> |
| [<span data-ttu-id="46ed4-114">Очередь с приоритетом</span><span class="sxs-lookup"><span data-stu-id="46ed4-114">Priority Queue</span></span>](../priority-queue.md) | <span data-ttu-id="46ed4-115">Запросы, отправляемые в службу, получают разные приоритеты. При этом запросы с высоким приоритетом принимаются и обрабатываются быстрее, чем запросы с низким приоритетом.</span><span class="sxs-lookup"><span data-stu-id="46ed4-115">Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority.</span></span> |
| [<span data-ttu-id="46ed4-116">Издатель и подписчик</span><span class="sxs-lookup"><span data-stu-id="46ed4-116">Publisher-Subscriber</span></span>](../publisher-subscriber.md) | <span data-ttu-id="46ed4-117">Настройка в приложении возможности объявлять событие для нескольких объектов-получателей одновременно без взаимозависимости между отправителями и получателями.</span><span class="sxs-lookup"><span data-stu-id="46ed4-117">Enable an application to announce events to multiple interested consumers aynchronously, without coupling the senders to the receivers.</span></span> |
| [<span data-ttu-id="46ed4-118">Выравнивание нагрузки на основе очередей</span><span class="sxs-lookup"><span data-stu-id="46ed4-118">Queue-Based Load Leveling</span></span>](../queue-based-load-leveling.md) | <span data-ttu-id="46ed4-119">Очередь выполняет роль буфера между задачей и службой, которую она вызывает, позволяя сгладить кратковременные всплески нагрузки.</span><span class="sxs-lookup"><span data-stu-id="46ed4-119">Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads.</span></span> |
| [<span data-ttu-id="46ed4-120">Планировщик, агент, контролер</span><span class="sxs-lookup"><span data-stu-id="46ed4-120">Scheduler Agent Supervisor</span></span>](../scheduler-agent-supervisor.md) | <span data-ttu-id="46ed4-121">Координируйте ряд действий в распределенном наборе служб и других удаленных ресурсов.</span><span class="sxs-lookup"><span data-stu-id="46ed4-121">Coordinate a set of actions across a distributed set of services and other remote resources.</span></span> |
