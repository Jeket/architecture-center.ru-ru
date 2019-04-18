---
title: CAF. Enterprise мелких и средних — начальное корпоративной политики за стратегии управления
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Enterprise мелких и средних — начальное корпоративной политики за стратегии управления
author: BrianBlanchard
ms.openlocfilehash: 2133145c9933ad450ea3cc55ecd68b8a667df783
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640028"
---
# <a name="small-to-medium-enterprise-initial-corporate-policy-behind-the-governance-strategy"></a><span data-ttu-id="ad544-103">Малые и средние предприятия. Начальная корпоративная политика, лежащая в основе стратегии управления</span><span class="sxs-lookup"><span data-stu-id="ad544-103">Small-to-medium enterprise: Initial corporate policy behind the governance strategy</span></span>

<span data-ttu-id="ad544-104">Следующая корпоративная политика определяет начальную позицию системы управления, которая является отправной точкой для этого пути взаимодействия.</span><span class="sxs-lookup"><span data-stu-id="ad544-104">The following corporate policy defines an initial governance position, which is the starting point for this journey.</span></span> <span data-ttu-id="ad544-105">В этой статье определяются риски на ранних стадиях, первоначальные правила политики и ранние процессы для принудительного применения правил политики.</span><span class="sxs-lookup"><span data-stu-id="ad544-105">This article defines early-stage risks, initial policy statements, and early processes to enforce policy statements.</span></span>

> [!NOTE]
><span data-ttu-id="ad544-106">Корпоративная политика не является техническим документом, но она управляет многими техническими решениями.</span><span class="sxs-lookup"><span data-stu-id="ad544-106">The corporate policy is not a technical document, but it drives many technical decisions.</span></span> <span data-ttu-id="ad544-107">MVP системы управления, описанный в [обзоре](./overview.md), в конечном счете является производным от этой политики.</span><span class="sxs-lookup"><span data-stu-id="ad544-107">The governance MVP described in the [overview](./overview.md) ultimately derives from this policy.</span></span> <span data-ttu-id="ad544-108">Перед реализацией MVP системы управления ваша организация должна разработать корпоративную политику на основе собственных целей и бизнес-рисков.</span><span class="sxs-lookup"><span data-stu-id="ad544-108">Before implementing a governance MVP, your organization should develop a corporate policy based on your own objectives and business risks.</span></span>

## <a name="cloud-governance-team"></a><span data-ttu-id="ad544-109">Команда управления облачными решениями</span><span class="sxs-lookup"><span data-stu-id="ad544-109">Cloud Governance team</span></span>

<span data-ttu-id="ad544-110">В нашем примере команда системы управления облаком состоит из двух системных администраторов, которые осознали важность системы управления.</span><span class="sxs-lookup"><span data-stu-id="ad544-110">In this narrative, the Cloud Governance team is comprised of two systems administrators who have recognized the need for governance.</span></span> <span data-ttu-id="ad544-111">В течение нескольких следующих месяцев они начнут выполнять работу по очистке системы управления облаком от присутствия компании, что принесет им прозвище "Хранители облака".</span><span class="sxs-lookup"><span data-stu-id="ad544-111">Over the next several months, they will inherit the job of cleaning up the governance of the company’s cloud presence, earning them the title of Cloud Custodians.</span></span> <span data-ttu-id="ad544-112">Скорее всего, в процессе эволюции это прозвище будет изменено.</span><span class="sxs-lookup"><span data-stu-id="ad544-112">In subsequent evolutions, this title will likely change.</span></span>

[!INCLUDE [business-risk](../../../../../includes/cloud-adoption/governance/business-risks.md)]

## <a name="tolerance-indicators"></a><span data-ttu-id="ad544-113">Индикаторы устойчивости</span><span class="sxs-lookup"><span data-stu-id="ad544-113">Tolerance indicators</span></span>

<span data-ttu-id="ad544-114">При низком желании инвестирования в систему управления уровень текущей устойчивости к риску остается высоким.</span><span class="sxs-lookup"><span data-stu-id="ad544-114">The current tolerance for risk is high and the appetite for investing in cloud governance is low.</span></span> <span data-ttu-id="ad544-115">Таким образом, индикаторы устойчивости выступают в качестве системы раннего предупреждения, которая позволит вложить больше времени и энергии.</span><span class="sxs-lookup"><span data-stu-id="ad544-115">As such, the tolerance indicators act as an early warning system to trigger more investment of time and energy.</span></span> <span data-ttu-id="ad544-116">Если будут наблюдаться следующие индикаторы, следует развить стратегию управления.</span><span class="sxs-lookup"><span data-stu-id="ad544-116">If and when the following indicators are observed, you should evolve the governance strategy.</span></span>

- <span data-ttu-id="ad544-117">Управление затратами. Масштаб развертывания превышает 100 ресурсов в облаке, или расходы превышают 1000 долларов США в месяц.</span><span class="sxs-lookup"><span data-stu-id="ad544-117">Cost Management: The scale of deployment exceeds 100 assets to the cloud, or monthly spending exceeds $1,000 USD per month.</span></span>
- <span data-ttu-id="ad544-118">Основные способы защиты. Включение защищенных данных в определенные планы по внедрению облака.</span><span class="sxs-lookup"><span data-stu-id="ad544-118">Security Baseline: Inclusion of protected data in defined cloud adoption plans.</span></span>
- <span data-ttu-id="ad544-119">Согласованность ресурсов. Включение любых критически важных приложений в определенные планы внедрения облака.</span><span class="sxs-lookup"><span data-stu-id="ad544-119">Resource Consistency: Inclusion of any mission-critical applications in defined cloud adoption plans.</span></span>

[!INCLUDE [policy-statements](../../../../../includes/cloud-adoption/governance/policy-statements.md)]

## <a name="next-steps"></a><span data-ttu-id="ad544-120">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="ad544-120">Next steps</span></span>

<span data-ttu-id="ad544-121">Эта корпоративная политика подготавливает команду управления облачными решениями к реализации MVP системы управления, который станет основой для внедрения.</span><span class="sxs-lookup"><span data-stu-id="ad544-121">This corporate policy prepares the Cloud Governance team to implement the governance MVP, which will be the foundation for adoption.</span></span> <span data-ttu-id="ad544-122">Следующим шагом является реализация MVP.</span><span class="sxs-lookup"><span data-stu-id="ad544-122">The next step is to implement this MVP.</span></span>

> [!div class="nextstepaction"]
> <span data-ttu-id="ad544-123">[Small-to-medium enterprise: Best practice explained](./best-practice-explained.md) (Малые и средние предприятия. Описание лучших методик)</span><span class="sxs-lookup"><span data-stu-id="ad544-123">[Best practice explained](./best-practice-explained.md)</span></span>