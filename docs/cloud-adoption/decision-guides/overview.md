---
title: CAF. Руководства по принятию архитектурных решений
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Сведения о руководствах по архитектурным решениям на платформе внедрения облака.
author: rotycenh
ms.openlocfilehash: b43047b5fc5b636bc84b9f28ec24846730e63672
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55902471"
---
# <a name="architectural-decision-guides"></a><span data-ttu-id="2ac8d-103">Руководства по принятию архитектурных решений</span><span class="sxs-lookup"><span data-stu-id="2ac8d-103">Architectural decision guides</span></span>

<span data-ttu-id="2ac8d-104">В руководствах по архитектурным решениям на платформе внедрения облака описаны примеры и модели, которые помогут при создании руководства по проектированию системы управления облаком.</span><span class="sxs-lookup"><span data-stu-id="2ac8d-104">The architectural decision guides in the Cloud Adoption Framework describe patterns and models that help when creating cloud governance design guidance.</span></span> <span data-ttu-id="2ac8d-105">В каждом руководстве по принятию решений описывается один основной компонент инфраструктуры облачных развертываний и представлены возможные шаблоны или модели, предназначенные для поддержки определенных сценариев облачных развертываний.</span><span class="sxs-lookup"><span data-stu-id="2ac8d-105">Each decision guide focuses on one core infrastructure component of cloud deployments and lists potential patterns or models intended to support specific cloud deployment scenarios.</span></span>

<span data-ttu-id="2ac8d-106">При определении системы управления облаком для вашей организации стратегии действенного управления позволяют получить стратегию базовых показателей.</span><span class="sxs-lookup"><span data-stu-id="2ac8d-106">When you begin to establish cloud governance for your organization,  actionable governance journeys provide a baseline roadmap.</span></span> <span data-ttu-id="2ac8d-107">Тем не менее в этих стратегиях предполагаются требования и приоритеты, которые могут не отражать потребности вашей организации.</span><span class="sxs-lookup"><span data-stu-id="2ac8d-107">However, these journeys make assumptions about requirements and priorities that may not reflect those of your organization.</span></span>
<span data-ttu-id="2ac8d-108">Эти руководства по принятию решений дополняют примеры стратегий управления, предоставляя альтернативные шаблоны и модели, которые помогают скорректировать варианты проектирования архитектуры, выбранные в примере руководства по проектированию, в соответствии с вашими требованиями.</span><span class="sxs-lookup"><span data-stu-id="2ac8d-108">These decision guides supplement the sample governance journeys by providing alternative patterns and models that help you align the architectural design choices made in the example design guidance with your own requirements.</span></span>

## <a name="design-guidance-categories"></a><span data-ttu-id="2ac8d-109">Категории руководств по проектированию</span><span class="sxs-lookup"><span data-stu-id="2ac8d-109">Design guidance categories</span></span>

<span data-ttu-id="2ac8d-110">В каждой из следующих категорий представлена основополагающая технология всех облачных развертываний.</span><span class="sxs-lookup"><span data-stu-id="2ac8d-110">Each of the following categories represents a foundational technology of all cloud deployments.</span></span> <span data-ttu-id="2ac8d-111">Изучите варианты в соответствующем разделе ниже, чтобы для каждого значения в примерах стратегий проектирования, которое не соответствуют вашим требованиям, выбрать шаблон или модель, который наиболее полно соответствует вашим потребностям.</span><span class="sxs-lookup"><span data-stu-id="2ac8d-111">For each choice in the example design journeys that doesn’t match your requirements, examine the options in the relevant section below to choose a pattern or model better suited to your needs.</span></span>

<span data-ttu-id="2ac8d-112">[Подписки](./subscriptions/overview.md). Запланируйте разработку подписки и структуру учетной записи облачного развертывания в соответствии с возможностями владения, выставления счетов и управления вашей организации.</span><span class="sxs-lookup"><span data-stu-id="2ac8d-112">[Subscriptions](./subscriptions/overview.md): Plan your cloud deployment's subscription design and account structure to match your organization's ownership, billing, and management capabilities.</span></span>

<span data-ttu-id="2ac8d-113">[Идентификация](./identity/overview.md). Интегрируйте облачные службы идентификации с имеющимися ресурсами идентификации для поддержки управления доступом в ИТ-среде.</span><span class="sxs-lookup"><span data-stu-id="2ac8d-113">[Identity](./identity/overview.md): Integrate cloud-based identity services with your existing identity resources to support access control within your IT environment.</span></span>

<span data-ttu-id="2ac8d-114">[Принудительное применение политики](./policy-enforcement/overview.md). Определите и примените правила политики организации для ресурсов и рабочих нагрузок, которые можно развернуть в облаке.</span><span class="sxs-lookup"><span data-stu-id="2ac8d-114">[Policy Enforcement](./policy-enforcement/overview.md): Define and enforce organizational policy rules for resources and workloads that you deploy to the cloud.</span></span>

<span data-ttu-id="2ac8d-115">[Согласованность ресурсов](./resource-consistency/overview.md). Убедитесь, что развертывание и организация облачных ресурсов выровнены для принудительного применения требований к управлению ресурсами и политикам.</span><span class="sxs-lookup"><span data-stu-id="2ac8d-115">[Resource Consistency](./resource-consistency/overview.md): Ensure that deployment and organization of your cloud-based resources align to enforce resource management and policy requirements.</span></span>

<span data-ttu-id="2ac8d-116">[Добавление тегов к ресурсам](./resource-tagging/overview.md). Упорядочьте облачные ресурсы для поддержки моделей выставления счетов, подходов к учету облака, управления, контроля доступа и эффективности работы.</span><span class="sxs-lookup"><span data-stu-id="2ac8d-116">[Resource Tagging](./resource-tagging/overview.md): Organize your cloud-based resources to support billing models, cloud accounting approaches, management, access control, and operational efficiency.</span></span> <span data-ttu-id="2ac8d-117">Для добавления тегов к ресурсам требуется согласованное и хорошо организованное именование и схема метаданных.</span><span class="sxs-lookup"><span data-stu-id="2ac8d-117">Resource tagging requires a consistent and well-organized naming and metadata scheme.</span></span>

<span data-ttu-id="2ac8d-118">[Программно-определяемые сети](./software-defined-network/overview.md). Разверните защищенные рабочие нагрузки в облако с помощью быстрого развертывания и модификации возможностей виртуализированной работы с сетью.</span><span class="sxs-lookup"><span data-stu-id="2ac8d-118">[Software Defined Networks](./software-defined-network/overview.md): Deploy secure workloads to the cloud using rapid deployment and modification of virtualized networking capabilities.</span></span> <span data-ttu-id="2ac8d-119">Программно-определяемые сети (SDN) могут поддерживать гибкие рабочие процессы, изолировать ресурсы и интегрировать облачные системы с имеющейся ИТ-инфраструктурой.</span><span class="sxs-lookup"><span data-stu-id="2ac8d-119">Software-defined networks (SDNs) can support agile workflows, isolate resources, and integrate cloud-based systems with your existing IT infrastructure.</span></span>

<span data-ttu-id="2ac8d-120">[Шифрование](./encryption/overview.md). Защитите конфиденциальные данные с использованием шифрования — важным аспектом безопасности в рамках облачного развертывания.</span><span class="sxs-lookup"><span data-stu-id="2ac8d-120">[Encryption](./encryption/overview.md): Secure your sensitive data using encryption, an important aspect of security within a cloud deployment.</span></span>

<span data-ttu-id="2ac8d-121">[Журналы и отчеты](./log-and-report/overview.md). Отслеживайте данные журнала, создаваемые облачными ресурсами.</span><span class="sxs-lookup"><span data-stu-id="2ac8d-121">[Logs and Reporting](./log-and-report/overview.md): Monitor log data generated by cloud-based resources.</span></span> <span data-ttu-id="2ac8d-122">Анализ данных предоставляет сведения об операциях, обслуживании и состоянии применения политик в рабочих нагрузках.</span><span class="sxs-lookup"><span data-stu-id="2ac8d-122">Analyzing data provides health-related insights into the operations, maintenance, and policy enforcement status of workloads.</span></span>

## <a name="next-steps"></a><span data-ttu-id="2ac8d-123">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="2ac8d-123">Next steps</span></span>

<span data-ttu-id="2ac8d-124">Узнайте, каким образом подписки и учетные записи используются в качестве базы для облачного развертывания.</span><span class="sxs-lookup"><span data-stu-id="2ac8d-124">Learn how subscriptions and accounts serve as the base of a cloud deployment.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="2ac8d-125">Проектирование подписок</span><span class="sxs-lookup"><span data-stu-id="2ac8d-125">Subscriptions design</span></span>](subscriptions/overview.md)
