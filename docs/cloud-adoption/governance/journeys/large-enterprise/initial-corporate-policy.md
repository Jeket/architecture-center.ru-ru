---
title: 'CAF. Крупное предприятие: начальная корпоративная политика, лежащая в основе стратегии управления'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 2/11/2019
description: 'Крупное предприятие: начальная корпоративная политика, лежащая в основе стратегии управления.'
author: BrianBlanchard
ms.openlocfilehash: d3bc31f95229a82cbc2f6ac6e684a87107783c07
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55902496"
---
# <a name="large-enterprise-initial-corporate-policy-behind-the-governance-strategy"></a><span data-ttu-id="5041d-103">Крупное предприятие. Начальная корпоративная политика, лежащая в основе стратегии управления</span><span class="sxs-lookup"><span data-stu-id="5041d-103">Large enterprise: Initial corporate policy behind the governance strategy</span></span>

<span data-ttu-id="5041d-104">Следующая корпоративная политика определяет начальную позицию управления, которая является отправной точкой для этого пути взаимодействия.</span><span class="sxs-lookup"><span data-stu-id="5041d-104">The following corporate policy defines the initial governance position, which is the starting point for this journey.</span></span> <span data-ttu-id="5041d-105">В этой статье определяются риски на ранних стадиях, первоначальные правила политики и ранние процессы для принудительного применения правил политики.</span><span class="sxs-lookup"><span data-stu-id="5041d-105">This article defines early-stage risks, initial policy statements, and early processes to enforce policy statements.</span></span>

> [!NOTE]
><span data-ttu-id="5041d-106">Корпоративная политика не является техническим документом, но она управляет многими техническими решениями.</span><span class="sxs-lookup"><span data-stu-id="5041d-106">The corporate policy is not a technical document, but it drives many technical decisions.</span></span> <span data-ttu-id="5041d-107">MVP системы управления, описанный в [обзоре](./overview.md), в конечном счете является производным от этой политики.</span><span class="sxs-lookup"><span data-stu-id="5041d-107">The governance MVP described in the [overview](./overview.md) ultimately derives from this policy.</span></span> <span data-ttu-id="5041d-108">Перед реализацией MVP системы управления ваша организация должна разработать корпоративную политику на основе собственных целей и бизнес-рисков.</span><span class="sxs-lookup"><span data-stu-id="5041d-108">Before implementing a governance MVP, your organization should develop a corporate policy based on your own objectives and business risks.</span></span>

## <a name="cloud-governance-team"></a><span data-ttu-id="5041d-109">Команда управления облачными решениями</span><span class="sxs-lookup"><span data-stu-id="5041d-109">Cloud Governance team</span></span>

<span data-ttu-id="5041d-110">ИТ-директор недавно провел совещание с командой ИТ-управления, чтобы разобраться с персональными данными и критически важными политиками, а также определить влияние изменения этих политик.</span><span class="sxs-lookup"><span data-stu-id="5041d-110">The CIO recently held a meeting with the IT Governance team to understand the history of the PII and mission-critical policies and review the impact of changing those policies.</span></span> <span data-ttu-id="5041d-111">Он также обсудил общий потенциал облака для ИТ и компании.</span><span class="sxs-lookup"><span data-stu-id="5041d-111">She also discussed the overall potential of the cloud for IT and the company.</span></span>

<span data-ttu-id="5041d-112">После совещания два члена команды ИТ-управления запросили разрешение на исследование и поддержку усилий по планированию в облаке.</span><span class="sxs-lookup"><span data-stu-id="5041d-112">After the meeting, two members of the IT Governance team requested permission to research and support the cloud planning efforts.</span></span> <span data-ttu-id="5041d-113">Понимая необходимость управления и возможность ограничить теневые ИТ, директор по ИТ-управлению поддержал эту идею.</span><span class="sxs-lookup"><span data-stu-id="5041d-113">Recognizing the need for governance and an opportunity to limit shadow IT, the Director of IT Governance supported this idea.</span></span> <span data-ttu-id="5041d-114">Таким образом была создана команда управления облачными решениями.</span><span class="sxs-lookup"><span data-stu-id="5041d-114">With that, the Cloud Governance team was born.</span></span> <span data-ttu-id="5041d-115">В течение следующих нескольких месяцев им придется устранить множество ошибок, допущенных во время исследования в облаке, с точки зрения управления.</span><span class="sxs-lookup"><span data-stu-id="5041d-115">Over the next several months, they will inherit the cleanup of many mistakes made during exploration in the cloud from a governance perspective.</span></span> <span data-ttu-id="5041d-116">Так они завоюют прозвище "Хранители облака".</span><span class="sxs-lookup"><span data-stu-id="5041d-116">This will earn them the moniker of Cloud Custodians.</span></span> <span data-ttu-id="5041d-117">На следующих этапах развития этот путь взаимодействия покажет, как со временем меняются их роли.</span><span class="sxs-lookup"><span data-stu-id="5041d-117">In later evolutions, this journey will show how their roles change over time.</span></span>

[!INCLUDE [business-risk](../../../../../includes/cloud-adoption/governance/business-risks.md)]

## <a name="tolerance-indicators"></a><span data-ttu-id="5041d-118">Индикаторы устойчивости</span><span class="sxs-lookup"><span data-stu-id="5041d-118">Tolerance indicators</span></span>

<span data-ttu-id="5041d-119">Текущая устойчивость к риску высока, а желание инвестировать в систему управления низкое.</span><span class="sxs-lookup"><span data-stu-id="5041d-119">The current risk tolerance is high and the appetite for investing in cloud governance is low.</span></span> <span data-ttu-id="5041d-120">Таким образом, индикаторы допустимости выступают в качестве системы раннего предупреждения и позволяют начать вкладывать время и энергию.</span><span class="sxs-lookup"><span data-stu-id="5041d-120">As such, the tolerance indicators act as an early warning system to trigger the investment of time and energy.</span></span> <span data-ttu-id="5041d-121">Если будут наблюдаться следующие индикаторы, было бы разумно развить стратегию управления.</span><span class="sxs-lookup"><span data-stu-id="5041d-121">If the following indicators are observed, it would be wise to evolve the governance strategy.</span></span>

- <span data-ttu-id="5041d-122">Управление затратами. Масштаб развертывания превышает 1000 ресурсов в облаке или расходы превышают 10 000 долларов США в месяц.</span><span class="sxs-lookup"><span data-stu-id="5041d-122">Cost Management: Scale of deployment exceeds 1,000 assets to the cloud, or monthly spending exceeds $10,000 USD per month.</span></span>
- <span data-ttu-id="5041d-123">Основные способы идентификации. Включение приложений с требованиями устаревшей или сторонней многофакторной идентификации (MFA).</span><span class="sxs-lookup"><span data-stu-id="5041d-123">Identity Baseline: Inclusion of applications with legacy or third-party multifactor authentication (MFA) requirements.</span></span>
- <span data-ttu-id="5041d-124">Основные способы защиты. Включение защищенных данных в определенные планы внедрения облака.</span><span class="sxs-lookup"><span data-stu-id="5041d-124">Security Baseline: Inclusion of protected data in defined cloud adoption plans.</span></span>
- <span data-ttu-id="5041d-125">Согласованность ресурсов. Включение любых критически важных приложений в определенные планы внедрения облака.</span><span class="sxs-lookup"><span data-stu-id="5041d-125">Resource Consistency: Inclusion of any mission-critical applications in defined cloud adoption plans.</span></span>

[!INCLUDE [policy-statements](../../../../../includes/cloud-adoption/governance/policy-statements.md)]

## <a name="next-steps"></a><span data-ttu-id="5041d-126">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="5041d-126">Next steps</span></span>

<span data-ttu-id="5041d-127">Эта корпоративная политика подготавливает команду управления облачными решениями к реализации MVP системы управления, который станет основой для внедрения.</span><span class="sxs-lookup"><span data-stu-id="5041d-127">This corporate policy prepares the Cloud Governance team to implement the governance MVP, which will be the foundation for adoption.</span></span> <span data-ttu-id="5041d-128">Следующим шагом является реализация MVP.</span><span class="sxs-lookup"><span data-stu-id="5041d-128">The next step is to implement this MVP.</span></span>

> [!div class="nextstepaction"]
> <span data-ttu-id="5041d-129">[Small-to-medium enterprise: Best practice explained](./best-practice-explained.md) (Малые и средние предприятия. Описание лучших методик)</span><span class="sxs-lookup"><span data-stu-id="5041d-129">[Best practice explained](./best-practice-explained.md)</span></span>