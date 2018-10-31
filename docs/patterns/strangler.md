---
title: Шаблон подавления
description: Пошаговая миграция устаревшей системы с постепенной заменой определенных компонентов новыми приложениями и службами.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 0bf0b76a69f947419da83edd894a04dbea02371b
ms.sourcegitcommit: 2ae794de13c45cf24ad60d4f4dbb193c25944eff
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/25/2018
ms.locfileid: "50001887"
---
# <a name="strangler-pattern"></a><span data-ttu-id="cbd00-103">Шаблон подавления</span><span class="sxs-lookup"><span data-stu-id="cbd00-103">Strangler pattern</span></span>

<span data-ttu-id="cbd00-104">Пошаговая миграция устаревшей системы с постепенной заменой определенных компонентов новыми приложениями и службами.</span><span class="sxs-lookup"><span data-stu-id="cbd00-104">Incrementally migrate a legacy system by gradually replacing specific pieces of functionality with new applications and services.</span></span> <span data-ttu-id="cbd00-105">Компоненты устаревшей системы постепенно удаляются, и со временем новая система полностью возьмет на себя все ее функции, что позволяет вывести старую систему из эксплуатации.</span><span class="sxs-lookup"><span data-stu-id="cbd00-105">As features from the legacy system are replaced, the new system eventually replaces all of the old system's features, strangling the old system and allowing you to decommission it.</span></span> 

## <a name="context-and-problem"></a><span data-ttu-id="cbd00-106">Контекст и проблема</span><span class="sxs-lookup"><span data-stu-id="cbd00-106">Context and problem</span></span>

<span data-ttu-id="cbd00-107">Средства разработки, технологии размещения и архитектуры, на основе которых создается любая система, часто устаревают с течением времени.</span><span class="sxs-lookup"><span data-stu-id="cbd00-107">As systems age, the development tools, hosting technology, and even system architectures they were built on can become increasingly obsolete.</span></span> <span data-ttu-id="cbd00-108">По мере добавления новых функций и возможностей неуклонно растет сложность старых приложений, что еще больше затрудняет обслуживание и добавление новых компонентов.</span><span class="sxs-lookup"><span data-stu-id="cbd00-108">As new features and functionality are added, the complexity of these applications can increase dramatically, making them harder to maintain or add new features to.</span></span>

<span data-ttu-id="cbd00-109">Но полная замена сложной системы часто сопряжена с огромными трудностями.</span><span class="sxs-lookup"><span data-stu-id="cbd00-109">Completely replacing a complex system can be a huge undertaking.</span></span> <span data-ttu-id="cbd00-110">Во многих случаях предпочтительнее переходить на новую систему постепенно, сохраняя старую систему для обработки тех возможностей, которые еще не реализованы в новой.</span><span class="sxs-lookup"><span data-stu-id="cbd00-110">Often, you will need a gradual migration to a new system, while keeping the old system to handle features that haven't been migrated yet.</span></span> <span data-ttu-id="cbd00-111">Выполнение двух разных версий одного приложения означает, что клиенты должны знать, в какой их них находятся нужные компоненты.</span><span class="sxs-lookup"><span data-stu-id="cbd00-111">However, running two separate versions of an application means that clients have to know where particular features are located.</span></span> <span data-ttu-id="cbd00-112">И эти сведения нужно обновлять во всех клиентах каждый раз, когда вы переносите очередной компонент или службу.</span><span class="sxs-lookup"><span data-stu-id="cbd00-112">Every time a feature or service is migrated, clients need to be updated to point to the new location.</span></span>

## <a name="solution"></a><span data-ttu-id="cbd00-113">Решение</span><span class="sxs-lookup"><span data-stu-id="cbd00-113">Solution</span></span>

<span data-ttu-id="cbd00-114">Поэтапно заменяйте компоненты новыми приложениями и службами.</span><span class="sxs-lookup"><span data-stu-id="cbd00-114">Incrementally replace specific pieces of functionality with new applications and services.</span></span> <span data-ttu-id="cbd00-115">Создайте оболочку, которая перехватывает все запросы к устаревшим системам серверной части.</span><span class="sxs-lookup"><span data-stu-id="cbd00-115">Create a façade that intercepts requests going to the backend legacy system.</span></span> <span data-ttu-id="cbd00-116">Эта оболочка будет выбирать, куда направить такие запросы: к устаревшему приложению или к новой службе.</span><span class="sxs-lookup"><span data-stu-id="cbd00-116">The façade routes these requests either to the legacy application or the new services.</span></span> <span data-ttu-id="cbd00-117">Так вы сможете перенести компоненты в новую систему постепенно, не изменяя интерфейс для пользователей, которые даже не заметят, что происходит поэтапная миграция.</span><span class="sxs-lookup"><span data-stu-id="cbd00-117">Existing features can be migrated to the new system gradually, and consumers can continue using the same interface, unaware that any migration has taken place.</span></span>

![](./_images/strangler.png)  

<span data-ttu-id="cbd00-118">Эта модель позволяет снизить риски миграции и распределить усилия разработчиков по времени.</span><span class="sxs-lookup"><span data-stu-id="cbd00-118">This pattern helps to minimize risk from the migration, and spread the development effort over time.</span></span> <span data-ttu-id="cbd00-119">Пока оболочка безопасно направляет пользователей к правильному приложению, вы можете добавлять компоненты в новую систему в комфортном темпе, сохраняя при этом работоспособность старого приложения.</span><span class="sxs-lookup"><span data-stu-id="cbd00-119">With the façade safely routing users to the correct application, you can add functionality to the new system at whatever pace you like, while ensuring the legacy application continues to function.</span></span> <span data-ttu-id="cbd00-120">Через некоторое время все функции будут перенесены в новую систему и устаревшая система больше не будет нужна.</span><span class="sxs-lookup"><span data-stu-id="cbd00-120">Over time, as features are migrated to the new system, the legacy system is eventually "strangled" and is no longer necessary.</span></span> <span data-ttu-id="cbd00-121">Когда этот процесс завершится, вы сможете отключить и удалить старую систему.</span><span class="sxs-lookup"><span data-stu-id="cbd00-121">Once this process is complete, the legacy system can safely be retired.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="cbd00-122">Проблемы и рекомендации</span><span class="sxs-lookup"><span data-stu-id="cbd00-122">Issues and considerations</span></span>

- <span data-ttu-id="cbd00-123">Отдельного внимания потребуют службы и хранилища данных, которые одновременно используются в новой и устаревшей системах.</span><span class="sxs-lookup"><span data-stu-id="cbd00-123">Consider how to handle services and data stores that are potentially used by both new and legacy systems.</span></span> <span data-ttu-id="cbd00-124">Следите за тем, чтобы обе системы могли получать доступ к этим ресурсам.</span><span class="sxs-lookup"><span data-stu-id="cbd00-124">Make sure both can access these resources side-by-side.</span></span>
- <span data-ttu-id="cbd00-125">Организуйте новые приложения и службы так, чтобы их можно было легко отключить и заменить во время следующей миграции.</span><span class="sxs-lookup"><span data-stu-id="cbd00-125">Structure new applications and services in a way that they can easily be intercepted and replaced in future strangler migrations.</span></span>
- <span data-ttu-id="cbd00-126">В определенный момент, когда миграция завершится, созданную для этого шаблона оболочку нужно убрать или преобразовать в адаптер для клиентов устаревших версий.</span><span class="sxs-lookup"><span data-stu-id="cbd00-126">At some point, when the migration is complete, the strangler façade will either go away or evolve into an adaptor for legacy clients.</span></span>
- <span data-ttu-id="cbd00-127">Следите за тем, чтобы оболочка обновлялась параллельно с перенесенной версией.</span><span class="sxs-lookup"><span data-stu-id="cbd00-127">Make sure the façade keeps up with the migration.</span></span>
- <span data-ttu-id="cbd00-128">Следите за тем, чтобы оболочка не стала единой точкой отказа или узким местом производительности.</span><span class="sxs-lookup"><span data-stu-id="cbd00-128">Make sure the façade doesn't become a single point of failure or a performance bottleneck.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="cbd00-129">Когда следует использовать этот шаблон</span><span class="sxs-lookup"><span data-stu-id="cbd00-129">When to use this pattern</span></span>

<span data-ttu-id="cbd00-130">Используйте этот шаблон при поэтапной миграции серверной части приложения на новую архитектуру.</span><span class="sxs-lookup"><span data-stu-id="cbd00-130">Use this pattern when gradually migrating a back-end application to a new architecture.</span></span>

<span data-ttu-id="cbd00-131">Эту схему не стоит применять в следующих случаях:</span><span class="sxs-lookup"><span data-stu-id="cbd00-131">This pattern may not be suitable:</span></span>

- <span data-ttu-id="cbd00-132">если запросы к системам серверной части невозможно задерживать;</span><span class="sxs-lookup"><span data-stu-id="cbd00-132">When requests to the back-end system cannot be intercepted.</span></span>
- <span data-ttu-id="cbd00-133">если система относительно невелика и ее легко полностью заменить.</span><span class="sxs-lookup"><span data-stu-id="cbd00-133">For smaller systems where the complexity of wholesale replacement is low.</span></span>

## <a name="related-guidance"></a><span data-ttu-id="cbd00-134">Связанные руководства</span><span class="sxs-lookup"><span data-stu-id="cbd00-134">Related guidance</span></span>

- <span data-ttu-id="cbd00-135">Запись блога [StranglerApplication](https://www.martinfowler.com/bliki/StranglerApplication.html) (автор — Мартин Фаулер)</span><span class="sxs-lookup"><span data-stu-id="cbd00-135">Martin Fowler's blog post on [StranglerApplication](https://www.martinfowler.com/bliki/StranglerApplication.html)</span></span>
- <span data-ttu-id="cbd00-136">[Anti-Corruption Layer pattern](./anti-corruption-layer.md) (Шаблон уровня защиты от повреждения)</span><span class="sxs-lookup"><span data-stu-id="cbd00-136">[Anti-Corruption Layer pattern](./anti-corruption-layer.md)</span></span>
- [<span data-ttu-id="cbd00-137">Схема маршрутизации шлюза</span><span class="sxs-lookup"><span data-stu-id="cbd00-137">Gateway Routing pattern</span></span>](./gateway-routing.md)


 

