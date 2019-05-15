---
title: CAF. Руководство по принятию решений касательно согласованности ресурсов
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Сведения о согласованности ресурсов при планировании миграции в Azure.
author: rotycenh
ms.openlocfilehash: 3159e4b7aeddfdd99261c0f68591998d741f3359
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640058"
---
# <a name="caf-resource-consistency-decision-guide"></a><span data-ttu-id="2f5f5-103">CAF. Руководство по принятию решений касательно согласованности ресурсов</span><span class="sxs-lookup"><span data-stu-id="2f5f5-103">CAF: Resource consistency decision guide</span></span>

<span data-ttu-id="2f5f5-104">Azure [разработка подписки](../subscriptions/overview.md) определяет, как организовать облачных ресурсов по отношению к организации структуры учета рекомендации и требования рабочей нагрузки.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-104">Azure [subscription design](../subscriptions/overview.md) defines how you organize your cloud assets in relation to your organization's structure, accounting practices, and workload requirements.</span></span> <span data-ttu-id="2f5f5-105">Помимо этого уровня структуры адресации политики вашей организации нормативные требования по недвижимости вашего облака требуется возможность согласованно организации, развертывания и управления ресурсами в подписке.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-105">In addition to this level of structure, addressing your organizational governance policy requirements across your cloud estate requires the ability to consistently organize, deploy, and manage resources within a subscription.</span></span>

![Схема вариантов обеспечения согласованности ресурсов, отсортированных в порядке увеличения сложности, со ссылками для быстрого перехода.](../../_images/discovery-guides/discovery-guide-resource-consistency.png)

<span data-ttu-id="2f5f5-107">Перейти к разделу: [Простое группирование](#basic-grouping) | [Согласованность развертывания](#deployment-consistency) | [Согласованность политики](#policy-consistency) | [Иерархическая согласованность](#hierarchical-consistency) | [Автоматическая согласованность](#automated-consistency)</span><span class="sxs-lookup"><span data-stu-id="2f5f5-107">Jump to: [Basic grouping](#basic-grouping) | [Deployment consistency](#deployment-consistency) | [Policy consistency](#policy-consistency) | [Hierarchical consistency](#hierarchical-consistency) | [Automated consistency](#automated-consistency)</span></span>

<span data-ttu-id="2f5f5-108">Решения по поводу уровнем требований к согласованности ресурсов риэлтерской компании облака в основном обусловлено следующими факторами: размер цифровых недвижимости после миграции, business или требования к среде, которые не удается отнести из существующей подписки подхода к проектированию, или нужно применить управление временем, после развертывания ресурсов.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-108">Decisions regarding the level of your cloud estate's resource consistency requirements are primarily driven by these factors: post-migration digital estate size, business or environmental requirements that don't fit neatly within your existing subscription design approaches, or the need to enforce governance over time after resources have been deployed.</span></span> 

<span data-ttu-id="2f5f5-109">Как эти факторы увеличение важности, преимущества обеспечения согласованного развертывания, группирование и управление ресурсами на основе облака становится более важным.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-109">As these factors increase in importance, the benefits of ensuring consistent deployment, grouping, and management of cloud-based resources becomes more important.</span></span> <span data-ttu-id="2f5f5-110">Достижение более высокий уровень согласованности ресурсов в соответствии с требованиями увеличение требуются дополнительные усилия, потраченные на автоматизации, средств и внедрения согласованности и это приводит к больше времени, затраченное на управление изменениями и отслеживания.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-110">Achieving more advanced levels of resource consistency to meet increasing requirements requires more effort spent in automation, tooling, and consistency enforcement, and this results in an more time spent on change management and tracking.</span></span>

## <a name="basic-grouping"></a><span data-ttu-id="2f5f5-111">Простое группирование</span><span class="sxs-lookup"><span data-stu-id="2f5f5-111">Basic grouping</span></span>

<span data-ttu-id="2f5f5-112">В Azure [группы ресурсов](/azure/azure-resource-manager/resource-group-overview#resource-groups) представляют собой базовый механизм организации ресурсов, позволяющий логически группировать ресурсы в подписке.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-112">In Azure, [resource groups](/azure/azure-resource-manager/resource-group-overview#resource-groups) are a core resource organization mechanism to logically group resources within a subscription.</span></span>

<span data-ttu-id="2f5f5-113">Это контейнеры для ресурсов с общим жизненным циклом или общими ограничениями управления, такими как требования управления доступом на основе ролей (RBAC) или требования политики.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-113">Resource groups act as containers for resources with a common lifecycle or shared management constraints such as policy or role-based access control (RBAC) requirements.</span></span> <span data-ttu-id="2f5f5-114">Группы ресурсов не могут быть вложенными, и ресурсы могут принадлежать только одной группе ресурсов.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-114">Resource groups can't be nested, and resources can only belong to one resource group.</span></span> <span data-ttu-id="2f5f5-115">Определенные действия могут применяться для всех ресурсов в группе ресурсов.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-115">Some actions can act on all resources in a resource group.</span></span> <span data-ttu-id="2f5f5-116">Например, при удалении группы ресурсов удаляются все входящие в нее ресурсы.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-116">For example, deleting a resource group removes all resources within that group.</span></span> <span data-ttu-id="2f5f5-117">Существуют распространенные шаблоны создания групп ресурсов, которые разделены на две общие категории:</span><span class="sxs-lookup"><span data-stu-id="2f5f5-117">There are common patterns when creating resource groups, commonly divided into two categories:</span></span>

- <span data-ttu-id="2f5f5-118">Традиционные рабочие нагрузки ИТ. Чаще всего группируются по элементам в рамках одного жизненного цикла, например приложения.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-118">Traditional IT workloads: Most often grouped by items within the same lifecycle, such as an application.</span></span> <span data-ttu-id="2f5f5-119">Группирование по приложениям позволяет управлять отдельными приложениями.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-119">Grouping by application allows for individual application management.</span></span>
- <span data-ttu-id="2f5f5-120">Гибкие рабочие нагрузки ИТ. Сосредотачиваются на облачных приложениях для внешних клиентов.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-120">Agile IT workloads: Focus on external customer-facing cloud applications.</span></span> <span data-ttu-id="2f5f5-121">Эти группы ресурсов часто отражают функциональные уровни развертывания (например, уровень Интернета или уровень приложений) и управления.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-121">These resource groups often reflect the functional layers of deployment (such as web tier or app tier) and management.</span></span>

## <a name="deployment-consistency"></a><span data-ttu-id="2f5f5-122">Согласованность развертывания</span><span class="sxs-lookup"><span data-stu-id="2f5f5-122">Deployment consistency</span></span>

<span data-ttu-id="2f5f5-123">Основываясь на основе базового ресурса, механизм группирования, платформа Azure предоставляет систему использовать шаблоны для развертывания ресурсов в облачной среде.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-123">Building on top of the base resource grouping mechanism, the Azure platform provides a system for using templates to deploy your resources to the cloud environment.</span></span> <span data-ttu-id="2f5f5-124">Вы можете использовать шаблоны для создания согласованной структуры и соглашений об именовании при развертывании рабочих нагрузок, применяя эти аспекты развертывания и контроля ресурсов.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-124">You can use templates to create consistent organization and naming conventions when deploying workloads, enforcing those aspects of your resource deployment and management design.</span></span>

<span data-ttu-id="2f5f5-125">[Шаблоны Azure Resource Manager](/azure/azure-resource-manager/resource-group-overview#template-deployment) позволяют многократно развертывать ресурсы в согласованном состоянии с использованием предопределенной структуры группы ресурсов и конфигурации.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-125">[Azure Resource Manager templates](/azure/azure-resource-manager/resource-group-overview#template-deployment) allow you to repeatedly deploy your resources in a consistent state using a predetermined configuration and resource group structure.</span></span> <span data-ttu-id="2f5f5-126">Эти шаблоны помогают задать набор стандартов в качестве основы для ваших развертываний.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-126">Resource Manager templates help you define a set of standards as a basis for your deployments.</span></span>

<span data-ttu-id="2f5f5-127">Например может иметь стандартный шаблон для развертывания рабочей нагрузки сервера web, которая содержит две виртуальные машины в качестве веб-серверов, в сочетании с подсистемой балансировки нагрузки для распределения трафика между серверами.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-127">For example, you can have a standard template for deploying a web server workload that contains two virtual machines as web servers combined with a load balancer to distribute traffic between the servers.</span></span> <span data-ttu-id="2f5f5-128">Затем можно повторно использовать этот шаблон для создания структурно идентичных виртуальных машин и балансировщик нагрузки, при необходимости этого типа рабочей нагрузки, участвует только изменения, имя развертывания и IP-адреса.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-128">You can then reuse this template to create structurally identical set of virtual machines and load balancer whenever this type of workload is needed, only changing the deployment name and IP addresses involved.</span></span>

<span data-ttu-id="2f5f5-129">Обратите внимание, что можно также развертывать эти шаблоны программным способом и интегрировать их в свои системы CI/CD.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-129">Note that you can also programmatically deploy these templates and integrate them with your CI/CD systems.</span></span>

## <a name="policy-consistency"></a><span data-ttu-id="2f5f5-130">Согласованность политик</span><span class="sxs-lookup"><span data-stu-id="2f5f5-130">Policy consistency</span></span>

<span data-ttu-id="2f5f5-131">Чтобы обеспечить применение политик управления во время создания ресурсов, как часть проекта группирования ресурсов при развертывании ресурсов необходимо использовать типичную конфигурацию.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-131">To ensure that governance policies are applied when resources are created, part of resource grouping design involves using a common configuration when deploying resources.</span></span>

<span data-ttu-id="2f5f5-132">Объединяя группы ресурсов и стандартизированные шаблоны Resource Manager, можно применять стандарты для параметров, необходимых в развертывании, и правил [Политики Azure](/azure/governance/policy/overview), применяемых к каждой группе ресурсов или ресурсу.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-132">By combining resource groups and standardized Resource Manager templates, you can enforce standards for what settings are required in a deployment and what [Azure Policy](/azure/governance/policy/overview) rules are applied to each resource group or resource.</span></span>

<span data-ttu-id="2f5f5-133">Например, может существовать требование подключения всех виртуальных машин, развернутых в рамках вашей подписки, к общей подсети, управляемой центральным ИТ-отделом.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-133">For example, you may have a requirement that all virtual machines deployed within your subscription connect to a common subnet managed by your central IT team.</span></span> <span data-ttu-id="2f5f5-134">Вы можете создать стандартный шаблон для развертывания виртуальных машин рабочей нагрузки, который будет создавать отдельную группу ресурсов для рабочей нагрузки и развертывать в ней виртуальные машины.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-134">You can create a standard template for deploying workload VMs which would create a separate resource group for the workload and deploy the required VMs there.</span></span> <span data-ttu-id="2f5f5-135">У этой группы ресурсов будет правило политики, которое разрешает присоединение к общей подсети только тех сетевых интерфейсов, которые находятся внутри группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-135">This resource group would have a policy rule to only allow network interfaces within the resource group to be joined to the shared subnet.</span></span>

<span data-ttu-id="2f5f5-136">Более подробное рассмотрение принудительного применения решений политики в рамках облачного развертывания см. в [этой статье](../policy-enforcement/overview.md).</span><span class="sxs-lookup"><span data-stu-id="2f5f5-136">For a more in-depth discussion of enforcing your policy decisions within a cloud deployment, see [Policy enforcement](../policy-enforcement/overview.md).</span></span>

## <a name="hierarchical-consistency"></a><span data-ttu-id="2f5f5-137">Иерархическая согласованность</span><span class="sxs-lookup"><span data-stu-id="2f5f5-137">Hierarchical consistency</span></span>

<span data-ttu-id="2f5f5-138">Группы ресурсов позволяет поддерживать дополнительных уровней иерархии в вашей организации в рамках подписки, применение правил политики Azure и доступа к элементам управления на уровне группы ресурсов.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-138">Resource groups allows you to support additional levels of hierarchy within your organization within the subscription, applying Azure Policy rules and access controls at a resource group level.</span></span> <span data-ttu-id="2f5f5-139">Тем не менее по мере роста размера вашей облачной недвижимости, может потребоваться поддержка сложнее между подписками нормативные требования к чем может поддерживаться с помощью иерархии соглашения Enterprise Azure Enterprise, отдел или учетную запись и подписки.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-139">However, As the size of your cloud estate grows, you may need to support more complicated cross-subscription governance requirements than can be supported using the Azure Enterprise Agreement's Enterprise/Department/Account/Subscription hierarchy.</span></span> 

<span data-ttu-id="2f5f5-140">[Группы управления Azure](../subscriptions/overview.md#management-groups) позволяет организации подписок в более сложные структуры организации путем группирования подписок в иерархии альтернативные для, установленного с структура соглашения enterprise.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-140">[Azure management groups](../subscriptions/overview.md#management-groups) allows you to organization subscriptions into more sophisticated organizational structures by grouping subscriptions in an alternative hierarchy to that established by your enterprise agreement's structure.</span></span> <span data-ttu-id="2f5f5-141">Этот альтернативный иерархия позволяет применить механизмы принудительного применения элемента управления и политики доступа в нескольких подписках и ресурсы, которые они содержат.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-141">This alternate hierarchy allows you to apply access control and policy enforcement mechanisms across multiple subscriptions and the resources they contain.</span></span> <span data-ttu-id="2f5f5-142">Иерархии группы управления могут использоваться для сопоставления риэлтерской компании облачных подписок с помощью операции или нормативные требования бизнеса.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-142">Management group hierarchies can be used to match your cloud estate's subscriptions with operations or business governance requirements.</span></span> 

## <a name="automated-consistency"></a><span data-ttu-id="2f5f5-143">Автоматическая согласованность</span><span class="sxs-lookup"><span data-stu-id="2f5f5-143">Automated consistency</span></span>

<span data-ttu-id="2f5f5-144">Для больших облачных развертываний глобальное управление становится более важным и более сложным.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-144">For large cloud deployments, global governance becomes both more important and more complex.</span></span> <span data-ttu-id="2f5f5-145">Очень важно автоматически принудительно применять нормативные требования при развертывании ресурсов, а также выполнять обновленные требования для существующих развертываний.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-145">It is crucial to automatically apply and enforce governance requirements when deploying resources, as well as meet updated requirements for existing deployments.</span></span>

<span data-ttu-id="2f5f5-146">Служба [Azure Blueprints](/azure/governance/blueprints/overview) позволяет организациям поддерживать глобальное управление большими облачными инфраструктурами в Azure.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-146">[Azure Blueprints](/azure/governance/blueprints/overview) enable organizations to support global governance of large cloud estates in Azure.</span></span> <span data-ttu-id="2f5f5-147">Схемы выходят за рамки возможностей, предоставляемых стандартными шаблонами Azure Resource Manager. Они позволяют создавать полные оркестрации развертываний, поддерживающие правила развертывания ресурсов и применение правил политики.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-147">Blueprints move beyond the capabilities provided by standard Azure Resource Manager templates to create complete deployment orchestrations capable of deploying resources and applying policy rules.</span></span> <span data-ttu-id="2f5f5-148">Схемы поддерживают управление версиями, возможность применения обновлений для всех подписок, где использовалась схема, и возможность блокировки развернутых подписок во избежание несанкционированного создания и изменения ресурсов.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-148">Blueprints supports versioning, the ability to make apply updates to all subscriptions where the blueprint was used, and the ability to lock down deployed subscriptions to avoid the unauthorized creation and modification of resources.</span></span>

<span data-ttu-id="2f5f5-149">Эти пакеты развертывания позволяют ИТ-специалистам и командам разработчиков быстро развертывать новые рабочие нагрузки и сетевые ресурсы, которые соответствуют меняющимся требованиям политики организации.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-149">These deployment packages allow IT and development teams to rapidly deploy new workloads and networking assets that comply with changing organizational policy requirements.</span></span> <span data-ttu-id="2f5f5-150">Схемы также можно интегрировать в конвейеры CI/CD для применения пересмотренных стандартов управления к развертываниям по мере их обновления.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-150">Blueprints can also be integrated into CI/CD pipelines to apply revised governance standards to deployments as they are updated.</span></span>

## <a name="next-steps"></a><span data-ttu-id="2f5f5-151">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="2f5f5-151">Next steps</span></span>

<span data-ttu-id="2f5f5-152">Ознакомьтесь со сведениями об именовании ресурсов и добавлении к ним тегов для дальнейшей организации облачных ресурсов и управления ими.</span><span class="sxs-lookup"><span data-stu-id="2f5f5-152">Learn how resource naming and tagging are used to further organize and manage your cloud resources.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="2f5f5-153">Именование ресурсов и добавление к ним тегов</span><span class="sxs-lookup"><span data-stu-id="2f5f5-153">Resource naming and tagging</span></span>](../resource-tagging/overview.md)