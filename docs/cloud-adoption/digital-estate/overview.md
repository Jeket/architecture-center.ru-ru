---
title: CAF. Что такое цифровые активы
titleSuffix: Microsoft Cloud Adoption Framework for Azure
description: Что такое цифровые активы
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: d23bdbdd98226e8b9cdb9bbb5fefa5a918a498d8
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55898431"
---
<!-- markdownlint-disable MD026 -->

# <a name="caf-what-is-a-digital-estate"></a><span data-ttu-id="d4303-103">CAF. Что такое цифровые активы</span><span class="sxs-lookup"><span data-stu-id="d4303-103">CAF: What is a digital estate?</span></span>

<span data-ttu-id="d4303-104">У каждой современной компании есть своя форма цифровых активов.</span><span class="sxs-lookup"><span data-stu-id="d4303-104">Every modern company has some form of digital estate.</span></span> <span data-ttu-id="d4303-105">Так же как и физические активы, цифровые активы — это абстрактные ссылки на количество материальных активов, находящихся в собственности.</span><span class="sxs-lookup"><span data-stu-id="d4303-105">Much like a physical estate, a digital estate is an abstract reference to a number of tangible, owned assets.</span></span> <span data-ttu-id="d4303-106">Цифровыми активами являются виртуальные машины, серверы, приложения, данные и так далее.</span><span class="sxs-lookup"><span data-stu-id="d4303-106">In a digital estate, those assets are comprised of virtual machines (VMs), servers, applications, data, and so on.</span></span> <span data-ttu-id="d4303-107">По сути цифровые активы — это коллекция ИТ-активов, которые обеспечивают выполнение бизнес-процессов и вспомогательных операций.</span><span class="sxs-lookup"><span data-stu-id="d4303-107">Essentially, a digital estate is the collection of IT assets that power business processes and supporting operations.</span></span>

<span data-ttu-id="d4303-108">Важность цифровых активов наиболее очевидна при планировании и выполнении цифровой трансформации.</span><span class="sxs-lookup"><span data-stu-id="d4303-108">The importance of a digital estate is most obvious during planning and execution of Digital Transformation efforts.</span></span> <span data-ttu-id="d4303-109">Во время процесса трансформации цифровые активы представляют, как команды по облачной стратегии сопоставляют бизнес-результаты с планами выпуска и техническими трудозатратами.</span><span class="sxs-lookup"><span data-stu-id="d4303-109">During transformation journeys, the digital estate is how Cloud Strategy teams map the business outcomes to release plans and technical efforts.</span></span> <span data-ttu-id="d4303-110">Все начинается с инвентаризации и измерения цифровых активов организации.</span><span class="sxs-lookup"><span data-stu-id="d4303-110">That all starts with an inventory and measurement of the digital assets the organization owns today.</span></span>

## <a name="how-can-a-digital-estate-be-measured"></a><span data-ttu-id="d4303-111">Пути измерения цифровых активов</span><span class="sxs-lookup"><span data-stu-id="d4303-111">How can a digital estate be measured?</span></span>

<span data-ttu-id="d4303-112">Измерение цифровых активов варьируется в зависимости от требуемых бизнес-результатов.</span><span class="sxs-lookup"><span data-stu-id="d4303-112">The measurement of a digital estate changes depending on the desired business outcomes.</span></span>

- <span data-ttu-id="d4303-113">**Миграция инфраструктуры**.</span><span class="sxs-lookup"><span data-stu-id="d4303-113">**Infrastructure migrations**.</span></span> <span data-ttu-id="d4303-114">Если организация является самоподдерживающейся и ищет пути оптимизировать расходы, операционные процессы, гибкость или другие аспекты оптимизации операций, цифровые активы фокусируются на виртуальных машинах, серверах и вспомогательных рабочих нагрузках.</span><span class="sxs-lookup"><span data-stu-id="d4303-114">When an organization is inward facing and seeks to optimize cost, operational processes, agility, or other aspects of optimizing operations, the digital estate focuses on VMs, servers, and supporting workloads.</span></span>

- <span data-ttu-id="d4303-115">**Инновация приложений**.</span><span class="sxs-lookup"><span data-stu-id="d4303-115">**Application innovation**.</span></span> <span data-ttu-id="d4303-116">При ориентировании исключительно на клиентов концепция трансформации немного отличается.</span><span class="sxs-lookup"><span data-stu-id="d4303-116">For customer-obsessed transformations, the lens is a bit different.</span></span> <span data-ttu-id="d4303-117">Нужно фокусироваться на приложениях, API-интерфейсах и данных о транзакциях, в которых нуждаются клиенты.</span><span class="sxs-lookup"><span data-stu-id="d4303-117">The focus should be placed in the applications, APIs, and transactional data that support the customers.</span></span> <span data-ttu-id="d4303-118">Виртуальные машины и сетевые устройства зачастую получают меньше внимания.</span><span class="sxs-lookup"><span data-stu-id="d4303-118">VMs and network appliances are often of less focus.</span></span>

- <span data-ttu-id="d4303-119">**Инновация на основе данных**.</span><span class="sxs-lookup"><span data-stu-id="d4303-119">**Data-driven innovation**.</span></span> <span data-ttu-id="d4303-120">На сегодняшнем цифровом рынке сложно запустить новый продукт или сервис без прочного фундамента данных.</span><span class="sxs-lookup"><span data-stu-id="d4303-120">In today's digitally driven market, it's difficult to launch a new product or service without a strong foundation in data.</span></span> <span data-ttu-id="d4303-121">Во время прорывного преобразования основное внимание уделяется разрозненным данным в организации.</span><span class="sxs-lookup"><span data-stu-id="d4303-121">During disruptive transformation, the focus is more on the silos of data across the organization.</span></span>

<span data-ttu-id="d4303-122">После того как организация определилась с наиболее важной формой трансформации, планирование цифровых активов становится куда легче.</span><span class="sxs-lookup"><span data-stu-id="d4303-122">Once an organization understands the most important form of transformation, digital estate planning becomes much easier to manage.</span></span>

> [!TIP]
> <span data-ttu-id="d4303-123">Каждый тип преобразования может измеряться любым из трех путей.</span><span class="sxs-lookup"><span data-stu-id="d4303-123">Each type of transformation could be measured with any of the three views.</span></span> <span data-ttu-id="d4303-124">Более того, обычной практикой является параллельное выполнение всех трех типов преобразования.</span><span class="sxs-lookup"><span data-stu-id="d4303-124">Further, it is common for companies to execute all three transformations in parallel.</span></span> <span data-ttu-id="d4303-125">Настоятельно рекомендуется, чтобы руководство и команда по облачной стратегии были твердо настроены относительно преобразования. Это самый важный залог успешности бизнеса.</span><span class="sxs-lookup"><span data-stu-id="d4303-125">It is highly suggested that the leadership and Cloud Strategy team be firmly aligned regarding the transformation that is most important for business success.</span></span> <span data-ttu-id="d4303-126">Понимание этого будет основой будущего общего языка и показателей при дальнейших множественных инициативах.</span><span class="sxs-lookup"><span data-stu-id="d4303-126">That understanding will serve as the basis for common language and metrics across multiple initiatives.</span></span>

## <a name="how-can-a-financial-model-be-updated-to-reflect-the-digital-estate"></a><span data-ttu-id="d4303-127">Обновление финансовой модели для отражения цифровых активов</span><span class="sxs-lookup"><span data-stu-id="d4303-127">How can a financial model be updated to reflect the digital estate?</span></span>

<span data-ttu-id="d4303-128">Анализ цифровых активов будет способствовать действиям по внедрению облачных служб.</span><span class="sxs-lookup"><span data-stu-id="d4303-128">An analysis of the digital estate will drive cloud adoption activities.</span></span> <span data-ttu-id="d4303-129">Он также предоставит информацию для финансовых моделей благодаря составлению моделей расходов на облако, что поспособствует рентабельности инвестиций.</span><span class="sxs-lookup"><span data-stu-id="d4303-129">It will also inform financial models by providing cloud costing models, which will in turn drive the return on investment (ROI).</span></span>

<span data-ttu-id="d4303-130">Чтобы завершить анализ цифровых активов, выполните следующие шаги.</span><span class="sxs-lookup"><span data-stu-id="d4303-130">To complete the digital estate analysis, perform the following steps:</span></span>

1. <span data-ttu-id="d4303-131">[Определите подход для анализа](approach.md).</span><span class="sxs-lookup"><span data-stu-id="d4303-131">[Determine analysis approach](approach.md)</span></span>
1. <span data-ttu-id="d4303-132">[Соберите текущие данные инвентаризации](inventory.md).</span><span class="sxs-lookup"><span data-stu-id="d4303-132">[Collect current state inventory](inventory.md)</span></span>
1. <span data-ttu-id="d4303-133">[Рационализируйте ресурсы цифровых активов](rationalize.md).</span><span class="sxs-lookup"><span data-stu-id="d4303-133">[Rationalize the assets in the digital estate](rationalize.md)</span></span>
1. <span data-ttu-id="d4303-134">[Сравните активы с облачными предложениями, чтобы рассчитать стоимость](calculate.md).</span><span class="sxs-lookup"><span data-stu-id="d4303-134">[Align assets to cloud offerings to calculate pricing](calculate.md)</span></span>

<span data-ttu-id="d4303-135">Финансовые модели и невыполненные работы по миграции можно изменить для отражения рационализованных и оцененных активов.</span><span class="sxs-lookup"><span data-stu-id="d4303-135">Financial models and migration backlogs can be modified to reflect the rationalized and priced estate.</span></span>

## <a name="next-steps"></a><span data-ttu-id="d4303-136">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="d4303-136">Next steps</span></span>

<span data-ttu-id="d4303-137">Перед началом планирования цифровых активов необходимо определить оптимальный подход.</span><span class="sxs-lookup"><span data-stu-id="d4303-137">Before digital estate planning can begin, you must determine what approach to use.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="d4303-138">Подходы к планированию цифровых активов</span><span class="sxs-lookup"><span data-stu-id="d4303-138">Approaches to digital estate planning</span></span>](approach.md)