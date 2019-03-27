---
title: CAF. Сопоставление руководств по проектированию с политикой.
description: Как сопоставляются руководства по проектированию с политикой?
author: BrianBlanchard
ms.date: 01/04/2019
ms.openlocfilehash: 77a35597585e5f58967ea79d3c7b0fa17b6ab80e
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241595"
---
<!---
I've established policies. How to help developers adopt these policies?
Draft an architecture design guide.

[Aspirational statement] If you're using Azure, you can use one of ours as a starting point. The choose one of the following 6 as a starting point and mold it to fit your policies.
--->

<!-- markdownlint-disable MD026 -->

# <a name="how-do-you-align-design-guides-with-policy"></a><span data-ttu-id="b75dc-103">Как сопоставляются руководства по проектированию с политикой?</span><span class="sxs-lookup"><span data-stu-id="b75dc-103">How do you align design guides with policy?</span></span>

<span data-ttu-id="b75dc-104">После [определения облачных политик](define-policy.md) на основе [определенных рисков](understanding-business-risk.md) вам потребуется создать практические рекомендации, соответствующие этим политикам, для ИТ-специалистов и разработчиков.</span><span class="sxs-lookup"><span data-stu-id="b75dc-104">After you've [defined cloud policies](define-policy.md) based on your [identified risks](understanding-business-risk.md), you'll need to generate actionable guidance that aligns with these policies for your IT staff and developers to refer to.</span></span> <span data-ttu-id="b75dc-105">Разработка руководства по проектированию облачной системы управления позволяет указать определенные структурные, технологические варианты и варианты процесса на основе инструкций политики, созданных для каждой из пяти дисциплин системы управления.</span><span class="sxs-lookup"><span data-stu-id="b75dc-105">Drafting a cloud governance design guide allows you to specify specific structural, technological, and process choices based on the policy statements you generated for each of the five governance disciplines.</span></span>

<span data-ttu-id="b75dc-106">Руководство по проектированию облачной системы управления должно определить варианты архитектуры и конструктивные шаблоны для каждого из основных компонентов инфраструктуры облачных развертываний, которые наилучшим образом соответствуют требованиям политики.</span><span class="sxs-lookup"><span data-stu-id="b75dc-106">A cloud governance design guide should establish the architecture choices and design patterns for each of the core infrastructure components of cloud deployments that best meet your policy requirements.</span></span> <span data-ttu-id="b75dc-107">Вместе с ними необходимо предоставить основное объяснение технологий, инструментов и процессов, которые будут поддерживать каждое проектное решение.</span><span class="sxs-lookup"><span data-stu-id="b75dc-107">Alongside these you should provide a high-level explanation of the technology, tools, and processes that will support each of these design decisions.</span></span>

<span data-ttu-id="b75dc-108">Несмотря на то, что анализ рисков и правила политики могут в определенной степени быть независимыми от облачной платформы, руководство по проектированию должно предоставить сведения о реализации, относящейся к конкретной платформе, относительно ИТ и разработки.</span><span class="sxs-lookup"><span data-stu-id="b75dc-108">Although your risk analysis and policy statements may, to some degree, be cloud platform agnostic, your design guide should provide platform-specific implementation details that your IT and dev.</span></span> <span data-ttu-id="b75dc-109">Сосредоточьтесь на архитектуре, средствах и функциях выбранной платформы при выборе проектного решения и предоставлении руководства.</span><span class="sxs-lookup"><span data-stu-id="b75dc-109">Focus on the architecture, tools, and features of your chosen platform when making design decision and providing guidance.</span></span>

<span data-ttu-id="b75dc-110">Хотя руководства по проектированию облачной системы должны принимать во внимание некоторые технические сведения, связанные с каждым компонентом инфраструктуры, они не являются обширными техническими документами или спецификацией.</span><span class="sxs-lookup"><span data-stu-id="b75dc-110">While cloud design guides should take into account some of the technical details associated with each infrastructure component, they are not meant to be extensive technical documents or specifications.</span></span> <span data-ttu-id="b75dc-111">Убедитесь, что в руководствах четко определены все правила политики и четко указаны проектные решения в простом для персонала виде для понимания и ознакомления.</span><span class="sxs-lookup"><span data-stu-id="b75dc-111">Make sure your guides address all of your policy statements and clearly state design decisions in a format easy for staff to understand and reference.</span></span>

<!-- markdownlint-enable MD033 -->

## <a name="using-the-actionable-governance-journeys"></a><span data-ttu-id="b75dc-112">Использование стратегий действенного управления</span><span class="sxs-lookup"><span data-stu-id="b75dc-112">Using the actionable governance journeys</span></span>

<span data-ttu-id="b75dc-113">Если вы планируете использовать платформу Azure для внедрения облака, CAF предоставляет [стратегии управления](../journeys/overview.md), иллюстрирующие инкрементальный подход к модели системы управления CAF.</span><span class="sxs-lookup"><span data-stu-id="b75dc-113">If you're planning to use the Azure platform for your cloud adoption, the CAF provides [governance journeys](../journeys/overview.md) illustrating the incremental approach of the CAF governance model.</span></span> <span data-ttu-id="b75dc-114">В них представлен широкий спектр распространенных сценариев внедрения, включая бизнес-риски, требования устойчивости и правила политики, примененные при создании продукта с минимальной функциональностью (MVP) системы управления.</span><span class="sxs-lookup"><span data-stu-id="b75dc-114">These narrative journeys cover a range of common adoption scenarios, including the business risks, tolerance requirements, and policy statements that went into creating a governance minimum viable product (MVP).</span></span> <span data-ttu-id="b75dc-115">Эти стратегии представляют синтез реальных пользовательских возможностей процесса внедрения облака в Azure.</span><span class="sxs-lookup"><span data-stu-id="b75dc-115">These journeys represent a synthesis of real-world customer experience of the cloud adoption process in Azure.</span></span>

<span data-ttu-id="b75dc-116">Хотя каждое внедрение облака имеет уникальные цели, приоритеты и трудности, эти примеры должны предоставлять хороший шаблон по превращению политики в руководство.</span><span class="sxs-lookup"><span data-stu-id="b75dc-116">While every cloud adoption has unique goals, priorities, and challenges, these samples should provide a good template for converting your policy into guidance.</span></span> <span data-ttu-id="b75dc-117">Выберите самый подходящий в вашей ситуации сценарий в качестве отправной точки и примените его в соответствии с потребностями конкретной политики.</span><span class="sxs-lookup"><span data-stu-id="b75dc-117">Pick the closest scenario to your situation as a starting point, and mold it to fit your specific policy needs.</span></span>

## <a name="next-steps"></a><span data-ttu-id="b75dc-118">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="b75dc-118">Next steps</span></span>

<span data-ttu-id="b75dc-119">Сформировав руководство по проектированию, установите процессы, обеспечивающие соблюдение политики, для обеспечения соответствия ее требованиям.</span><span class="sxs-lookup"><span data-stu-id="b75dc-119">With design guidance in place, establish policy adherence processes to ensure policy compliance.</span></span>

> [!div class="nextstepaction"]
> <span data-ttu-id="b75dc-120">[What processes can help ensure policy adherence?](processes.md) (Какие процессы могут помочь обеспечить соблюдение политики?).</span><span class="sxs-lookup"><span data-stu-id="b75dc-120">[Policy adherence processes](processes.md)</span></span>