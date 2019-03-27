---
title: CAF. Сбор данных инвентаризации для цифровых активов
titleSuffix: Microsoft Cloud Adoption Framework for Azure
description: Сбор данных инвентаризации для цифровых активов.
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 0c68ff1e5ff51395698d37fb9b59c7a76c9479b7
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58242555"
---
# <a name="gather-inventory-data-for-a-digital-estate"></a><span data-ttu-id="b5338-103">Сбор данных инвентаризации для цифровых активов</span><span class="sxs-lookup"><span data-stu-id="b5338-103">Gather inventory data for a digital estate</span></span>

<span data-ttu-id="b5338-104">Разработка инвентаризации — это первый шаг в [планировании цифровых активов](overview.md).</span><span class="sxs-lookup"><span data-stu-id="b5338-104">Developing an inventory is the first step in [Digital Estate Planning](overview.md).</span></span> <span data-ttu-id="b5338-105">В рамках этого процесса составляется список ИТ-активов, которые поддерживают конкретные бизнес-функции, для дальнейшего анализа и рационализации.</span><span class="sxs-lookup"><span data-stu-id="b5338-105">In this process, a list of IT assets that support specific business functions would be collected for later analysis and rationalization.</span></span> <span data-ttu-id="b5338-106">В этой статье предполагается, что восходящий анализ лучше всего подходит для потребностей планирования.</span><span class="sxs-lookup"><span data-stu-id="b5338-106">This article assumes that a bottom-up approach to analysis is most appropriate for planning needs.</span></span> <span data-ttu-id="b5338-107">Дополнительные сведения см. в статье о [подходах к планированию цифровых активов](./approach.md).</span><span class="sxs-lookup"><span data-stu-id="b5338-107">For more information, see [Approaches to digital estate planning](./approach.md).</span></span>

## <a name="take-inventory-of-a-digital-estate"></a><span data-ttu-id="b5338-108">Инвентаризация цифровых активов</span><span class="sxs-lookup"><span data-stu-id="b5338-108">Take inventory of a digital estate</span></span>

<span data-ttu-id="b5338-109">Инвентарь, поддерживающий цифровые активы, изменяется в зависимости от желаемой цифровой трансформации и от самого процесса трансформации.</span><span class="sxs-lookup"><span data-stu-id="b5338-109">The inventory supporting a digital estate changes depending on the desired digital transformation and corresponding transformation journey.</span></span>

- <span data-ttu-id="b5338-110">**Операционное преобразование**.</span><span class="sxs-lookup"><span data-stu-id="b5338-110">**Operational transformation**.</span></span> <span data-ttu-id="b5338-111">Во время операционного преобразования часто рекомендуется, чтобы для сбора данных об инвентаре использовались инструменты проверки, которые могут создавать централизованный список всех виртуальных машин и серверов.</span><span class="sxs-lookup"><span data-stu-id="b5338-111">During an operational transformation, it is often advised that the inventory be collected from scanning tools which can create a centralized list of all VMs and servers.</span></span> <span data-ttu-id="b5338-112">Некоторые средства также могут создавать сопоставления и зависимости сети, которые помогут определить выравнивание рабочей нагрузки.</span><span class="sxs-lookup"><span data-stu-id="b5338-112">Some tools can also create network mappings and dependencies, which will help define workload alignment.</span></span>

- <span data-ttu-id="b5338-113">**Инкрементное преобразование.**</span><span class="sxs-lookup"><span data-stu-id="b5338-113">**Incremental transformation.**</span></span> <span data-ttu-id="b5338-114">Инвентарь при инкрементном преобразовании начинается с клиента.</span><span class="sxs-lookup"><span data-stu-id="b5338-114">Inventory for an incremental transformation begins with the customer.</span></span> <span data-ttu-id="b5338-115">Следует начать с сопоставления начального и конечного опыта пользователей.</span><span class="sxs-lookup"><span data-stu-id="b5338-115">Mapping the customer experience from start to finish is a good place to begin.</span></span> <span data-ttu-id="b5338-116">Выравнивание, которое сопоставляет приложения, API-интерфейсы, данные и другие ресурсы, создает детальный список инвентаря для анализа.</span><span class="sxs-lookup"><span data-stu-id="b5338-116">Aligning that map to applications, APIs, data, and other assets will create a detailed inventory for analysis.</span></span>

- <span data-ttu-id="b5338-117">**Прорывное преобразование.**</span><span class="sxs-lookup"><span data-stu-id="b5338-117">**Disruptive transformation.**</span></span> <span data-ttu-id="b5338-118">Прорывное преобразование фокусируется на продуктах или сервисах.</span><span class="sxs-lookup"><span data-stu-id="b5338-118">Disruptive transformation focuses on the product or service.</span></span> <span data-ttu-id="b5338-119">В таком случае инвентарь включает сопоставление возможностей, которые вносят коренные перемены на рынке, и необходимых возможностей.</span><span class="sxs-lookup"><span data-stu-id="b5338-119">From there an inventory would include a mapping of the opportunities to disrupt the market and the capabilities needed.</span></span>

## <a name="accuracy-and-completeness-of-an-inventory"></a><span data-ttu-id="b5338-120">Точность и полнота инвентаризации</span><span class="sxs-lookup"><span data-stu-id="b5338-120">Accuracy and completeness of an inventory</span></span>

<span data-ttu-id="b5338-121">Инвентаризация редко завершается на первой итерации.</span><span class="sxs-lookup"><span data-stu-id="b5338-121">An inventory is seldom fully complete in its first iteration.</span></span> <span data-ttu-id="b5338-122">Настоятельно рекомендуется, чтобы различные члены команды по облачной стратегии согласовали требования заинтересованных лиц и опытных пользователей для проверки инвентаря.</span><span class="sxs-lookup"><span data-stu-id="b5338-122">It is highly advised that various members of the Cloud Strategy team align stakeholders and power users to validate the inventory.</span></span> <span data-ttu-id="b5338-123">Когда это возможно, дополнительные инструменты, такие как анализ сети или анализ зависимостей, могут использоваться для идентификации ресурсов, которые отправляют трафик, но не принадлежат к инвентарю.</span><span class="sxs-lookup"><span data-stu-id="b5338-123">When possible, additional tools like network and dependency analysis can be used to identify assets that are being sent traffic, but are not in the inventory.</span></span>

## <a name="next-steps"></a><span data-ttu-id="b5338-124">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="b5338-124">Next steps</span></span>

<span data-ttu-id="b5338-125">Когда инвентарь будет собран и проверен, его можно рационализировать.</span><span class="sxs-lookup"><span data-stu-id="b5338-125">Once an inventory is compiled and validated, it can rationalized.</span></span> <span data-ttu-id="b5338-126">Рационализация инвентаря — это следующий шаг в планировании цифровых активов.</span><span class="sxs-lookup"><span data-stu-id="b5338-126">Inventory Rationalization is the next step to digital estate planning.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="b5338-127">Рационализация цифровых активов</span><span class="sxs-lookup"><span data-stu-id="b5338-127">Rationalize the digital estate</span></span>](rationalize.md)