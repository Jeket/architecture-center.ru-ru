---
title: Шаблон подавления
titleSuffix: Cloud Design Patterns
description: Пошаговая миграция устаревшей системы с постепенной заменой определенных компонентов новыми приложениями и службами.
keywords: Конструктивный шаблон
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: f139d368c98256c0190753930983a47df81a5134
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54480576"
---
# <a name="strangler-pattern"></a><span data-ttu-id="ab2ed-104">Шаблон подавления</span><span class="sxs-lookup"><span data-stu-id="ab2ed-104">Strangler pattern</span></span>

<span data-ttu-id="ab2ed-105">Пошаговая миграция устаревшей системы с постепенной заменой определенных компонентов новыми приложениями и службами.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-105">Incrementally migrate a legacy system by gradually replacing specific pieces of functionality with new applications and services.</span></span> <span data-ttu-id="ab2ed-106">Компоненты устаревшей системы постепенно удаляются, и со временем новая система полностью возьмет на себя все ее функции, что позволяет вывести старую систему из эксплуатации.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-106">As features from the legacy system are replaced, the new system eventually replaces all of the old system's features, strangling the old system and allowing you to decommission it.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="ab2ed-107">Контекст и проблема</span><span class="sxs-lookup"><span data-stu-id="ab2ed-107">Context and problem</span></span>

<span data-ttu-id="ab2ed-108">Средства разработки, технологии размещения и архитектуры, на основе которых создается любая система, часто устаревают с течением времени.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-108">As systems age, the development tools, hosting technology, and even system architectures they were built on can become increasingly obsolete.</span></span> <span data-ttu-id="ab2ed-109">По мере добавления новых функций и возможностей неуклонно растет сложность старых приложений, что еще больше затрудняет обслуживание и добавление новых компонентов.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-109">As new features and functionality are added, the complexity of these applications can increase dramatically, making them harder to maintain or add new features to.</span></span>

<span data-ttu-id="ab2ed-110">Но полная замена сложной системы часто сопряжена с огромными трудностями.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-110">Completely replacing a complex system can be a huge undertaking.</span></span> <span data-ttu-id="ab2ed-111">Во многих случаях предпочтительнее переходить на новую систему постепенно, сохраняя старую систему для обработки тех возможностей, которые еще не реализованы в новой.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-111">Often, you will need a gradual migration to a new system, while keeping the old system to handle features that haven't been migrated yet.</span></span> <span data-ttu-id="ab2ed-112">Выполнение двух разных версий одного приложения означает, что клиенты должны знать, в какой их них находятся нужные компоненты.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-112">However, running two separate versions of an application means that clients have to know where particular features are located.</span></span> <span data-ttu-id="ab2ed-113">И эти сведения нужно обновлять во всех клиентах каждый раз, когда вы переносите очередной компонент или службу.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-113">Every time a feature or service is migrated, clients need to be updated to point to the new location.</span></span>

## <a name="solution"></a><span data-ttu-id="ab2ed-114">Решение</span><span class="sxs-lookup"><span data-stu-id="ab2ed-114">Solution</span></span>

<span data-ttu-id="ab2ed-115">Поэтапно заменяйте компоненты новыми приложениями и службами.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-115">Incrementally replace specific pieces of functionality with new applications and services.</span></span> <span data-ttu-id="ab2ed-116">Создайте оболочку, которая перехватывает все запросы к устаревшим системам серверной части.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-116">Create a façade that intercepts requests going to the backend legacy system.</span></span> <span data-ttu-id="ab2ed-117">Эта оболочка будет выбирать, куда направить такие запросы: к устаревшему приложению или к новой службе.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-117">The façade routes these requests either to the legacy application or the new services.</span></span> <span data-ttu-id="ab2ed-118">Так вы сможете перенести компоненты в новую систему постепенно, не изменяя интерфейс для пользователей, которые даже не заметят, что происходит поэтапная миграция.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-118">Existing features can be migrated to the new system gradually, and consumers can continue using the same interface, unaware that any migration has taken place.</span></span>

![Шаблон подавления](./_images/strangler.png)

<span data-ttu-id="ab2ed-120">Эта модель позволяет снизить риски миграции и распределить усилия разработчиков по времени.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-120">This pattern helps to minimize risk from the migration, and spread the development effort over time.</span></span> <span data-ttu-id="ab2ed-121">Пока оболочка безопасно направляет пользователей к правильному приложению, вы можете добавлять компоненты в новую систему в комфортном темпе, сохраняя при этом работоспособность старого приложения.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-121">With the façade safely routing users to the correct application, you can add functionality to the new system at whatever pace you like, while ensuring the legacy application continues to function.</span></span> <span data-ttu-id="ab2ed-122">Через некоторое время все функции будут перенесены в новую систему и устаревшая система больше не будет нужна.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-122">Over time, as features are migrated to the new system, the legacy system is eventually "strangled" and is no longer necessary.</span></span> <span data-ttu-id="ab2ed-123">Когда этот процесс завершится, вы сможете отключить и удалить старую систему.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-123">Once this process is complete, the legacy system can safely be retired.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="ab2ed-124">Проблемы и рекомендации</span><span class="sxs-lookup"><span data-stu-id="ab2ed-124">Issues and considerations</span></span>

- <span data-ttu-id="ab2ed-125">Отдельного внимания потребуют службы и хранилища данных, которые одновременно используются в новой и устаревшей системах.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-125">Consider how to handle services and data stores that are potentially used by both new and legacy systems.</span></span> <span data-ttu-id="ab2ed-126">Следите за тем, чтобы обе системы могли получать доступ к этим ресурсам.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-126">Make sure both can access these resources side-by-side.</span></span>
- <span data-ttu-id="ab2ed-127">Организуйте новые приложения и службы так, чтобы их можно было легко отключить и заменить во время следующей миграции.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-127">Structure new applications and services in a way that they can easily be intercepted and replaced in future strangler migrations.</span></span>
- <span data-ttu-id="ab2ed-128">В определенный момент, когда миграция завершится, созданную для этого шаблона оболочку нужно убрать или преобразовать в адаптер для клиентов устаревших версий.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-128">At some point, when the migration is complete, the strangler façade will either go away or evolve into an adaptor for legacy clients.</span></span>
- <span data-ttu-id="ab2ed-129">Следите за тем, чтобы оболочка обновлялась параллельно с перенесенной версией.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-129">Make sure the façade keeps up with the migration.</span></span>
- <span data-ttu-id="ab2ed-130">Следите за тем, чтобы оболочка не стала единой точкой отказа или узким местом производительности.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-130">Make sure the façade doesn't become a single point of failure or a performance bottleneck.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="ab2ed-131">Когда следует использовать этот шаблон</span><span class="sxs-lookup"><span data-stu-id="ab2ed-131">When to use this pattern</span></span>

<span data-ttu-id="ab2ed-132">Используйте этот шаблон при поэтапной миграции серверной части приложения на новую архитектуру.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-132">Use this pattern when gradually migrating a back-end application to a new architecture.</span></span>

<span data-ttu-id="ab2ed-133">Эту схему не стоит применять в следующих случаях:</span><span class="sxs-lookup"><span data-stu-id="ab2ed-133">This pattern may not be suitable:</span></span>

- <span data-ttu-id="ab2ed-134">если запросы к системам серверной части невозможно задерживать;</span><span class="sxs-lookup"><span data-stu-id="ab2ed-134">When requests to the back-end system cannot be intercepted.</span></span>
- <span data-ttu-id="ab2ed-135">если система относительно невелика и ее легко полностью заменить.</span><span class="sxs-lookup"><span data-stu-id="ab2ed-135">For smaller systems where the complexity of wholesale replacement is low.</span></span>

## <a name="related-guidance"></a><span data-ttu-id="ab2ed-136">Связанные руководства</span><span class="sxs-lookup"><span data-stu-id="ab2ed-136">Related guidance</span></span>

- <span data-ttu-id="ab2ed-137">Запись блога [StranglerApplication](https://www.martinfowler.com/bliki/StranglerApplication.html) (автор — Мартин Фаулер)</span><span class="sxs-lookup"><span data-stu-id="ab2ed-137">Martin Fowler's blog post on [StranglerApplication](https://www.martinfowler.com/bliki/StranglerApplication.html)</span></span>
