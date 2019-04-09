---
title: CAF. Общее представление о системе управления облачными ресурсами
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.date: 02/11/2019
description: Объяснение системы управления облачными ресурсами в Azure
author: petertaylor9999
ms.openlocfilehash: 0989a5aad8a6290cce07fd71771c690bbd615e0d
ms.sourcegitcommit: 0a8a60d782facc294f7f78ec0e9033e3ee16bf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/08/2019
ms.locfileid: "59068860"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-cloud-resource-governance"></a><span data-ttu-id="fdbe6-103">Общее представление о системе управления облачными ресурсами</span><span class="sxs-lookup"><span data-stu-id="fdbe6-103">What is cloud resource governance?</span></span>

<span data-ttu-id="fdbe6-104">В [как работает Azure?](what-is-azure.md), вы узнали, что Azure — это совокупность серверов и сетевого оборудования, запуска виртуализированного оборудования и программного обеспечения, от имени пользователей.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-104">In [How does Azure work?](what-is-azure.md), you learned that Azure is a collection of servers and networking hardware running virtualized hardware and software on behalf of users.</span></span> <span data-ttu-id="fdbe6-105">Azure позволяет вашей организации в разработке приложений и ИТ-отделы agile, упрощая создание, чтение, обновление и удаление ресурсов, при необходимости.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-105">Azure enables your organization's application development and IT departments to be agile by making it easy to create, read, update, and delete resources as needed.</span></span>

<span data-ttu-id="fdbe6-106">Тем не менее хотя неограниченный доступ к ресурсам мог разработчиков очень гибкой, она может также привести к непредвиденных расходов.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-106">However, while unrestricted access to resources can make developers very agile, it can also lead to unexpected costs.</span></span> <span data-ttu-id="fdbe6-107">Например, команде разработчиков может быть разрешено развернуть набор ресурсов для тестирования, но после завершения команда может забыть их удалить.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-107">For example, a development team might be approved to deploy a set of resources for testing but forget to delete them when testing is complete.</span></span> <span data-ttu-id="fdbe6-108">Эти ресурсы будут продолжать накапливать затраты, несмотря на то, что они больше не утвержденных или не нужны.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-108">These resources will continue to accrue costs even though they are no longer approved or necessary.</span></span>

<span data-ttu-id="fdbe6-109">Решением является управление ресурсами доступа.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-109">The solution is resource access governance.</span></span> <span data-ttu-id="fdbe6-110">**Управление** — непрерывный процесс управления, мониторинг и аудит использования ресурсов Azure в соответствии с требованиями вашей организации.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-110">**Governance** is the ongoing process of managing, monitoring, and auditing the use of Azure resources to meet the requirements of your organization.</span></span>

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ii94]

<!-- markdownlint-enable MD034 -->

<span data-ttu-id="fdbe6-111">Эти требования являются уникальными для каждой организации, поэтому не рекомендуется использовать универсального подхода к управлению.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-111">These requirements are unique to each organization, so a one-size-fits-all approach to governance isn't helpful.</span></span> <span data-ttu-id="fdbe6-112">Вместо этого он работает для каждой организации для разработки их модель управления, использовать два средства основной системы управления Azure: **управления доступом на основе ролей (RBAC)** и **политики ресурсов**.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-112">Instead, it's up to each organization to design their governance model using Azure's two primary governance tools: **role-based access control (RBAC)** and **resource policy**.</span></span>

<span data-ttu-id="fdbe6-113">RBAC определяет роли и роли определяют возможности каждого пользователя, назначенного этой роли.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-113">RBAC defines roles, and roles define the capabilities of each user assigned that role.</span></span> <span data-ttu-id="fdbe6-114">Например **владельца** роль предоставляет все возможности (Создание, чтение, обновление и удаление) для ресурса, хотя **чтения** роль позволяет только возможность чтения.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-114">For example, the **owner** role allows all capabilites (create, read, update, and delete) for a resource, while the  **reader** role allows only the read capability.</span></span> <span data-ttu-id="fdbe6-115">Роли могут определяться с широкой области, к которому применяется множество типов ресурсов или узкой области, к которому применяется несколько.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-115">Roles can be defined with a broad scope that applies to many resource types, or a narrow scope that applies to a few.</span></span>

<span data-ttu-id="fdbe6-116">Правила создания ресурсов определяются политиками ресурсов.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-116">Resource policies define rules for resource creation.</span></span> <span data-ttu-id="fdbe6-117">Например политику ресурсов можно ограничить номер SKU виртуальной машины для предварительно утвержденных определенный размер.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-117">For example, a resource policy can limit the SKU of a virtual machine to a particular pre-approved size.</span></span> <span data-ttu-id="fdbe6-118">Другой политики ресурсов удалось применить приложение тега для центра затрат, назначенный при запросе для создания ресурса.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-118">Another resource policy could enforce the application of a tag for an assigned cost center when the request is made to create the resource.</span></span>

<span data-ttu-id="fdbe6-119">При настройке этих средств, управление и гибкость организации все описанное выше.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-119">When configuring these tools, it is important to balance governance and organizational agility.</span></span> <span data-ttu-id="fdbe6-120">Более строгие политики управления, менее гибкой разработки ваших разработчиков и ИТ-специалисты будут.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-120">The more restrictive your governance policy, the less agile your developers and IT workers will be.</span></span> <span data-ttu-id="fdbe6-121">Политика ограничения управления может требовать дополнительных действий вручную, как требование разработчику заполнить форму или отправить сообщение электронной почты членом команды управления, чтобы вручную создать ресурс.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-121">A restrictive governance policy may require more manual steps like requiring a developer to fill out a form or send an email to a member of the governance team to manually create a resource.</span></span> <span data-ttu-id="fdbe6-122">Группа управления имеет ограничение по масштабируемой емкости и может стать узким местом, приводит к группам разработчиков непродуктивно ожидание их ресурсы были созданный или ненужные ресурсы, накапливаемый затраты, прежде чем они будут удалены.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-122">The governance team has finite capacity and may become a bottleneck, resulting in development teams waiting unproductively for their resources to be created or unneeded resources accruing costs before they are deleted.</span></span>

## <a name="next-steps"></a><span data-ttu-id="fdbe6-123">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="fdbe6-123">Next steps</span></span>

<span data-ttu-id="fdbe6-124">Теперь, когда вы понимаете концепцию облачной управление ресурсами, Дополнительные сведения о том, как осуществляется доступ к ресурсам в Azure.</span><span class="sxs-lookup"><span data-stu-id="fdbe6-124">Now that you understand the concept of cloud resource governance, learn more about how resource access is managed in Azure.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="fdbe6-125">Дополнительные сведения о доступе к ресурсам в Azure</span><span class="sxs-lookup"><span data-stu-id="fdbe6-125">Learn about resource access in Azure</span></span>](azure-resource-access.md)
