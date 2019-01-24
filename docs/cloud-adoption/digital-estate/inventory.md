---
title: Сбор данных инвентаризации для цифровых активов
titleSuffix: Enterprise Cloud Adoption
description: Сведения о создании инвентаризации для цифровых активов
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.openlocfilehash: 5c2f270cf8de81c8a94d1f924f51611e657ed0ed
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54481802"
---
# <a name="enterprise-cloud-adoption-gather-inventory-data-for-a-digital-estate"></a><span data-ttu-id="4dfc3-103">Внедрение облачных решений в организации. Сбор данных инвентаризации для цифровых активов</span><span class="sxs-lookup"><span data-stu-id="4dfc3-103">Enterprise Cloud Adoption: Gather inventory data for a digital estate</span></span>

<span data-ttu-id="4dfc3-104">Разработка инвентаризации — это первый шаг в [планировании цифровых активов](overview.md).</span><span class="sxs-lookup"><span data-stu-id="4dfc3-104">Developing an inventory is the first step in [Digital Estate Planning](overview.md).</span></span> <span data-ttu-id="4dfc3-105">В рамках этого процесса составляется список ИТ-активов, которые поддерживают конкретные бизнес-функции, для дальнейшего анализа и рационализации.</span><span class="sxs-lookup"><span data-stu-id="4dfc3-105">In this process, a list of IT assets that support specific business functions would be collected for later analysis and rationalization.</span></span> <span data-ttu-id="4dfc3-106">В этой статье предполагается, что восходящий анализ лучше всего подходит для потребностей планирования.</span><span class="sxs-lookup"><span data-stu-id="4dfc3-106">This article assumes that a bottom-up approach to analysis is most appropriate for planning needs.</span></span> <span data-ttu-id="4dfc3-107">Дополнительные сведения см. в статье о [подходах к планированию цифровых активов](./approach.md).</span><span class="sxs-lookup"><span data-stu-id="4dfc3-107">For more information, see [Approaches to digital estate planning](./approach.md).</span></span>

## <a name="how-can-a-digital-estate-be-inventoried"></a><span data-ttu-id="4dfc3-108">Пути инвентаризации цифровых активов</span><span class="sxs-lookup"><span data-stu-id="4dfc3-108">How can a digital estate be inventoried?</span></span>

<span data-ttu-id="4dfc3-109">Инвентарь, поддерживающий цифровые активы, изменяется в зависимости от желаемой цифровой трансформации и от самого процесса трансформации.</span><span class="sxs-lookup"><span data-stu-id="4dfc3-109">The inventory supporting a digital estate changes depending on the desired Digital Transformation and corresponding Transformation Journey.</span></span>

- <span data-ttu-id="4dfc3-110">Операционное преобразование. Во время операционного преобразования часто рекомендуется, чтобы для сбора данных об инвентаре использовались инструменты проверки, которые могут создавать централизованный список всех виртуальных машин и серверов.</span><span class="sxs-lookup"><span data-stu-id="4dfc3-110">Operational Transformation: During an operational transformation, it is often advised that the inventory be collected from scanning tools which can create a centralized list of all VMs and servers.</span></span> <span data-ttu-id="4dfc3-111">Некоторые средства также могут создавать сопоставления и зависимости сети, которые помогут определить выравнивание рабочей нагрузки.</span><span class="sxs-lookup"><span data-stu-id="4dfc3-111">Some tools can also create network mappings and dependencies, which will help define workload alignment.</span></span>

- <span data-ttu-id="4dfc3-112">Инкрементное преобразование. Инвентарь при инкрементном преобразовании начинается с клиента.</span><span class="sxs-lookup"><span data-stu-id="4dfc3-112">Incremental Transformation: Inventory for an incremental transformation begins with the customer.</span></span> <span data-ttu-id="4dfc3-113">Следует начать с сопоставления начального и конечного опыта пользователей.</span><span class="sxs-lookup"><span data-stu-id="4dfc3-113">Mapping the customer experience from start to finish is a good place to begin.</span></span> <span data-ttu-id="4dfc3-114">Выравнивание, которое сопоставляет приложения, API-интерфейсы, данные и другие ресурсы, создает детальный список инвентаря для анализа.</span><span class="sxs-lookup"><span data-stu-id="4dfc3-114">Aligning that map to applications, APIs, data, and other assets will create a detailed inventory for analysis.</span></span>

- <span data-ttu-id="4dfc3-115">Прорывное преобразование. Прорывное преобразование фокусируется на продуктах или сервисах.</span><span class="sxs-lookup"><span data-stu-id="4dfc3-115">Disruptive Transformation: Disruptive transformation focuses on the product or service.</span></span> <span data-ttu-id="4dfc3-116">В таком случае инвентарь включает сопоставление возможностей, которые вносят коренные перемены на рынке, и необходимых возможностей.</span><span class="sxs-lookup"><span data-stu-id="4dfc3-116">From there an inventory would include a mapping of the opportunities to disrupt the market and the capabilities needed.</span></span>

## <a name="accuracy-and-completeness-of-an-inventory"></a><span data-ttu-id="4dfc3-117">Точность и полнота инвентаризации</span><span class="sxs-lookup"><span data-stu-id="4dfc3-117">Accuracy and completeness of an inventory</span></span>

<span data-ttu-id="4dfc3-118">Инвентаризация редко завершается на первой итерации.</span><span class="sxs-lookup"><span data-stu-id="4dfc3-118">An inventory is seldom fully complete in its first iteration.</span></span> <span data-ttu-id="4dfc3-119">Настоятельно рекомендуется, чтобы различные члены команды по облачной стратегии согласовали требования заинтересованных лиц и опытных пользователей для проверки инвентаря.</span><span class="sxs-lookup"><span data-stu-id="4dfc3-119">It is highly advised that various members of the Cloud Strategy team align stakeholders and power users to validate the inventory.</span></span> <span data-ttu-id="4dfc3-120">Когда это возможно, дополнительные инструменты, такие как анализ сети или анализ зависимостей, могут использоваться для идентификации ресурсов, которые отправляют трафик, но не принадлежат к инвентарю.</span><span class="sxs-lookup"><span data-stu-id="4dfc3-120">When possible, additional tools like network and dependency analysis can be used to identify assets that are being sent traffic, but are not in the inventory.</span></span>

## <a name="next-steps"></a><span data-ttu-id="4dfc3-121">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="4dfc3-121">Next steps</span></span>

<span data-ttu-id="4dfc3-122">Когда инвентарь будет собран и проверен, его можно рационализировать.</span><span class="sxs-lookup"><span data-stu-id="4dfc3-122">Once an inventory is compiled and validated, it can rationalized.</span></span> <span data-ttu-id="4dfc3-123">Рационализация инвентаря — это следующий шаг в планировании цифровых активов.</span><span class="sxs-lookup"><span data-stu-id="4dfc3-123">Inventory Rationalization is the next step to digital estate planning.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="4dfc3-124">Рационализация цифровых активов</span><span class="sxs-lookup"><span data-stu-id="4dfc3-124">Rationalize the digital estate</span></span>](rationalize.md)