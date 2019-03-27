---
title: CAF. Управление доступом к ресурсам в Azure
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Основные сведения о средствах управления доступом к ресурсам в Azure Диспетчер ресурсов Azure, подписки, группы ресурсов и ресурсы
author: petertaylor9999
ms.openlocfilehash: b98cdc94d6d3a37c1e65da1d4de35d5d9520d6eb
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243205"
---
# <a name="resource-access-management-in-azure"></a><span data-ttu-id="0adde-103">Управление доступом к ресурсам в Azure</span><span class="sxs-lookup"><span data-stu-id="0adde-103">Resource access management in Azure</span></span>

<span data-ttu-id="0adde-104">[Система управления облаком](../overview.md) описывает пять дисциплин, включая "Управление ресурсами".</span><span class="sxs-lookup"><span data-stu-id="0adde-104">[Cloud Governance](../overview.md) outlines the five disciplines of Cloud Governance, which includes Resource Management.</span></span>  <span data-ttu-id="0adde-105">Статья [CAF. Общие сведения о дисциплине "Согласованность ресурсов"](overview.md) объясняет, как управление доступом к ресурсам вписывается в дисциплину управления ресурсами.</span><span class="sxs-lookup"><span data-stu-id="0adde-105">[What is resource access governance](overview.md) furthers explains how resource access management fits into the resource management discipline.</span></span> <span data-ttu-id="0adde-106">Прежде чем переходить к изучению принципов проектирования модели системы управления, важно разобраться с элементами управления доступом к ресурсам в Azure.</span><span class="sxs-lookup"><span data-stu-id="0adde-106">Before you move on to learn how to design a governance model, it's important to understand the resource access management controls in Azure.</span></span> <span data-ttu-id="0adde-107">Конфигурация этих элементов управления доступом к ресурсами — это основа модели системы управления.</span><span class="sxs-lookup"><span data-stu-id="0adde-107">The configuration of these resource access management controls forms the basis of your governance model.</span></span>

<span data-ttu-id="0adde-108">Давайте рассмотрим принцип развертывания ресурсов в Azure.</span><span class="sxs-lookup"><span data-stu-id="0adde-108">Begin by taking a closer look at how resources are deployed in Azure.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="what-is-an-azure-resource"></a><span data-ttu-id="0adde-109">Ресурсы Azure</span><span class="sxs-lookup"><span data-stu-id="0adde-109">What is an Azure resource?</span></span>

<span data-ttu-id="0adde-110">В Azure **ресурс** — это сущность под управлением Azure.</span><span class="sxs-lookup"><span data-stu-id="0adde-110">In Azure, the term **resource** refers to an entity managed by Azure.</span></span> <span data-ttu-id="0adde-111">Например, к ресурсам относятся виртуальные машины, виртуальные сети и учетные записи хранения.</span><span class="sxs-lookup"><span data-stu-id="0adde-111">For example, virtual machines, virtual networks, and storage accounts are all referred to as Azure resources.</span></span>

<span data-ttu-id="0adde-112">![Схема ресурса](../../_images/governance-1-9.png)
*Рис. 1. Ресурс.*</span><span class="sxs-lookup"><span data-stu-id="0adde-112">![Diagram of a resource](../../_images/governance-1-9.png)
*Figure 1. A resource.*</span></span>

## <a name="what-is-an-azure-resource-group"></a><span data-ttu-id="0adde-113">Что такое группа ресурсов Azure?</span><span class="sxs-lookup"><span data-stu-id="0adde-113">What is an Azure resource group?</span></span>

<span data-ttu-id="0adde-114">Каждый ресурс в Azure должен принадлежать [группе ресурсов](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span><span class="sxs-lookup"><span data-stu-id="0adde-114">Each resource in Azure must belong to a [resource group](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span></span> <span data-ttu-id="0adde-115">Группа ресурсов — это логическая конструкция, в которой сгруппированы несколько ресурсов, что позволяет управлять ими как единой сущностью.</span><span class="sxs-lookup"><span data-stu-id="0adde-115">A resource group is simply a logical construct that groups multiple resources together so they can be managed as a single entity.</span></span> <span data-ttu-id="0adde-116">Например, ресурсы с аналогичным жизненным циклом, например ресурсы [n-уровневых приложений](/azure/architecture/guide/architecture-styles/n-tier), можно создавать или удалять как группу.</span><span class="sxs-lookup"><span data-stu-id="0adde-116">For example, resources that share a similar lifecycle, such as the resources for an [n-tier application](/azure/architecture/guide/architecture-styles/n-tier) may be created or deleted as a group.</span></span>

<span data-ttu-id="0adde-117">![Схема группы ресурсов, содержащей ресурс](../../_images/governance-1-10.png)
*Рис. 2. Группа ресурсов, которая содержит ресурс.*</span><span class="sxs-lookup"><span data-stu-id="0adde-117">![Diagram of a resource group containing a resource](../../_images/governance-1-10.png)
*Figure 2. A resource group contains a resource.*</span></span>

<span data-ttu-id="0adde-118">Группы и ресурсы в них связаны с **подпиской** Azure.</span><span class="sxs-lookup"><span data-stu-id="0adde-118">Resource groups and the resources they contain are associated with an Azure **subscription**.</span></span>

## <a name="what-is-an-azure-subscription"></a><span data-ttu-id="0adde-119">Подписка Azure</span><span class="sxs-lookup"><span data-stu-id="0adde-119">What is an Azure subscription?</span></span>

<span data-ttu-id="0adde-120">Подписка Azure, как и группа ресурсов, — это логическая конструкция, в которой сгруппированы группы и ресурсы в них.</span><span class="sxs-lookup"><span data-stu-id="0adde-120">An Azure subscription is similar to a resource group in that it's a logical construct that groups together resource groups and their resources.</span></span> <span data-ttu-id="0adde-121">Тем не менее подписка Azure также связана с элементами управления, используемыми Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="0adde-121">However, an Azure subscription is also associated with the controls used by Azure Resource Manager.</span></span> <span data-ttu-id="0adde-122">Что это означает?</span><span class="sxs-lookup"><span data-stu-id="0adde-122">What does this mean?</span></span> <span data-ttu-id="0adde-123">Рассмотрим более подробно Azure Resource Manager, чтобы больше узнать о связи между ним и подпиской Azure.</span><span class="sxs-lookup"><span data-stu-id="0adde-123">Take a closer look at Azure Resource Manager to learn about the relationship between it and an Azure subscription.</span></span>

<span data-ttu-id="0adde-124">![Схема подписки Azure](../../_images/governance-1-11.png)
*Рис. 3. Подписка Azure.*</span><span class="sxs-lookup"><span data-stu-id="0adde-124">![Diagram of an Azure subscription](../../_images/governance-1-11.png)
*Figure 3. An Azure subscription.*</span></span>

## <a name="what-is-azure-resource-manager"></a><span data-ttu-id="0adde-125">Azure Resource Manager</span><span class="sxs-lookup"><span data-stu-id="0adde-125">What is Azure Resource Manager?</span></span>

<span data-ttu-id="0adde-126">Из статьи [Пояснения. Принцип работы Azure](../../getting-started/what-is-azure.md) вы узнали, что Azure включает внешний интерфейс со многими службами, которые оркестрируют работу всех функции Azure.</span><span class="sxs-lookup"><span data-stu-id="0adde-126">In [how does Azure work?](../../getting-started/what-is-azure.md) you learned that Azure includes a "front end" with many services that orchestrate all the functions of Azure.</span></span> <span data-ttu-id="0adde-127">К этим службам относится [Azure Resource Manager](/azure/azure-resource-manager/). В этой службе размещен интерфейс REST API, с помощью которого клиенты управляют ресурсами.</span><span class="sxs-lookup"><span data-stu-id="0adde-127">One of these services is [Azure Resource Manager](/azure/azure-resource-manager/), and this service hosts the RESTful API used by clients to manage resources.</span></span>

<span data-ttu-id="0adde-128">![Схема Azure Resource Manager](../../_images/governance-1-12.png)
*Рис. 4. Azure Resource Manager*.</span><span class="sxs-lookup"><span data-stu-id="0adde-128">![Diagram of Azure Resource Manager](../../_images/governance-1-12.png)
*Figure 4. Azure Resource Manager.*</span></span>

<span data-ttu-id="0adde-129">На следующем рисунке показано три клиента: [PowerShell](/powershell/azure/overview), [портал Azure](https://portal.azure.com) и [интерфейс командной строки Azure (CLI)](/cli/azure).</span><span class="sxs-lookup"><span data-stu-id="0adde-129">The following figure shows three clients: [PowerShell](/powershell/azure/overview), the [Azure portal](https://portal.azure.com), and the [Azure command-line interface (CLI)](/cli/azure):</span></span>

<span data-ttu-id="0adde-130">![Схема подключения клиентов Azure к REST API Azure Resource Manager](../../_images/governance-1-13.png)
*Рис. 5. Клиенты Azure подключаются к RESTful API Azure Resource Manager.*</span><span class="sxs-lookup"><span data-stu-id="0adde-130">![Diagram of Azure clients connecting to the Azure Resource Manager API](../../_images/governance-1-13.png)
*Figure 5. Azure clients connect to the Azure Resource Manager RESTful API.*</span></span>

<span data-ttu-id="0adde-131">Эти клиенты подключаются к Azure Resource Manager через RESTful API. Однако Azure Resource Manager не содержит функциональность, чтобы управлять ресурсами напрямую.</span><span class="sxs-lookup"><span data-stu-id="0adde-131">While these clients connect to Azure Resource Manager using the RESTful API, Azure Resource Manager does not include functionality to manage resources directly.</span></span> <span data-ttu-id="0adde-132">Вместо этого большинство типов ресурсов в Azure имеют собственный [**поставщик ресурсов**](/azure/azure-resource-manager/resource-group-overview#terminology).</span><span class="sxs-lookup"><span data-stu-id="0adde-132">Rather, most resource types in Azure have their own [**resource provider**](/azure/azure-resource-manager/resource-group-overview#terminology).</span></span>

<span data-ttu-id="0adde-133">![Поставщики ресурсов Azure](../../_images/governance-1-14.png)
*Рис. 6. Поставщики ресурсов Azure.*</span><span class="sxs-lookup"><span data-stu-id="0adde-133">![Azure resource providers](../../_images/governance-1-14.png)
*Figure 6. Azure resource providers.*</span></span>

<span data-ttu-id="0adde-134">Когда клиент отправляет запрос на управление определенным ресурсом, Azure Resource Manager подключается к поставщику ресурсов этого типа ресурса, чтобы выполнить запрос.</span><span class="sxs-lookup"><span data-stu-id="0adde-134">When a client makes a request to manage a specific resource, Azure Resource Manager connects to the resource provider for that resource type to complete the request.</span></span> <span data-ttu-id="0adde-135">Например, если клиент отправляет запрос на управление ресурсами виртуальной машины, Azure Resource Manager подключается к поставщику ресурсов **Microsoft.compute**.</span><span class="sxs-lookup"><span data-stu-id="0adde-135">For example, if a client makes a request to manage a virtual machine resource, Azure Resource Manager connects to the **Microsoft.Compute** resource provider.</span></span>

<span data-ttu-id="0adde-136">![Подключение Azure Resource Manager к поставщику ресурсов Microsoft.Compute](../../_images/governance-1-15.png)
*Рис. 7. Azure Resource Manager подключается к поставщику ресурсов **Microsoft.compute**, чтобы управлять ресурсом, указанным в запросе клиента*.</span><span class="sxs-lookup"><span data-stu-id="0adde-136">![Azure Resource Manager connecting to the Microsoft.Compute resource provider](../../_images/governance-1-15.png)
*Figure 7. Azure Resource Manager connects to the **Microsoft.Compute** resource provider to manage the resource specified in the client request.*</span></span>

<span data-ttu-id="0adde-137">Azure Resource Manager требует, чтобы клиент указал идентификатор подписки и группы ресурсов и тем самым разрешил управлять ресурсами виртуальной машины.</span><span class="sxs-lookup"><span data-stu-id="0adde-137">Azure Resource Manager requires the client to specify an identifier for both the subscription and the resource group in order to manage the virtual machine resource.</span></span>

<span data-ttu-id="0adde-138">Теперь, когда вы имеете представление о принципе работы Azure Resource Manager, рассмотрим, как используемые этой службой элементы управления связаны с подпиской Azure.</span><span class="sxs-lookup"><span data-stu-id="0adde-138">Now that you have an understanding of how Azure Resource Manager works, return to the discussion of how an Azure subscription is associated with the controls used by Azure Resource Manager.</span></span> <span data-ttu-id="0adde-139">Перед выполнением Azure Resource Manager любого запроса на управление ресурсом проверяется набор элементов управления.</span><span class="sxs-lookup"><span data-stu-id="0adde-139">Before any resource management request can be executed by Azure Resource Manager, a set of controls are checked.</span></span>

<span data-ttu-id="0adde-140">Сначала проверяется, выполняется ли запрос проверенным пользователем. Azure Resource Manager имеет отношения доверия с [Azure Active Directory (Azure AD)](/azure/active-directory/), что позволяет обеспечить механизм удостоверения пользователя.</span><span class="sxs-lookup"><span data-stu-id="0adde-140">The first control is that a request must be made by a validated user, and Azure Resource Manager has a trusted relationship with [Azure Active Directory (Azure AD)](/azure/active-directory/) to provide user identity functionality.</span></span>

<span data-ttu-id="0adde-141">![Azure Active Directory](../../_images/governance-1-16.png)
*Рис. 8. Azure Active Directory*.</span><span class="sxs-lookup"><span data-stu-id="0adde-141">![Azure Active Directory](../../_images/governance-1-16.png)
*Figure 8. Azure Active Directory.*</span></span>

<span data-ttu-id="0adde-142">В Azure AD данные пользователей сегментируются по **клиентам**.</span><span class="sxs-lookup"><span data-stu-id="0adde-142">In Azure AD, users are segmented into **tenants**.</span></span> <span data-ttu-id="0adde-143">Клиент — это логическая структура, представляющая собой защищенный выделенный экземпляр Azure AD, который, как правило, связан с организацией.</span><span class="sxs-lookup"><span data-stu-id="0adde-143">A tenant is a logical construct that represents a secure, dedicated instance of Azure AD typically associated with an organization.</span></span> <span data-ttu-id="0adde-144">Каждая подписка связана с клиентом Azure AD.</span><span class="sxs-lookup"><span data-stu-id="0adde-144">Each subscription is associated with an Azure AD tenant.</span></span>

<span data-ttu-id="0adde-145">![Клиент Azure AD, связанный с подпиской](../../_images/governance-1-17.png)
*Рис. 9. Клиент Azure AD, связанный с подпиской.*</span><span class="sxs-lookup"><span data-stu-id="0adde-145">![An Azure AD tenant associated with a subscription](../../_images/governance-1-17.png)
*Figure 9. An Azure AD tenant associated with a subscription.*</span></span>

<span data-ttu-id="0adde-146">Каждый запрос клиента на управление ресурсом в определенной подписке требует наличия у пользователя учетной записи в связанном клиенте Azure AD.</span><span class="sxs-lookup"><span data-stu-id="0adde-146">Each client request to manage a resource in a particular subscription requires that the user has an account in the associated Azure AD tenant.</span></span>

<span data-ttu-id="0adde-147">Следующим проверяется, имеет ли пользователь необходимые права на выполнение запроса.</span><span class="sxs-lookup"><span data-stu-id="0adde-147">The next control is a check that the user has sufficient permission to make the request.</span></span> <span data-ttu-id="0adde-148">Разрешения присваиваются пользователям с использованием механизма [управления доступом на основе ролей (RBAC)](/azure/role-based-access-control/).</span><span class="sxs-lookup"><span data-stu-id="0adde-148">Permissions are assigned to users using [role-based access control (RBAC)](/azure/role-based-access-control/).</span></span>

<span data-ttu-id="0adde-149">![Пользователи, назначенные ролям RBAC](../../_images/governance-1-18.png)
*Рис. 10. Каждому пользователю в клиенте назначается одна или несколько ролей RBAC.*</span><span class="sxs-lookup"><span data-stu-id="0adde-149">![Users assigned to RBAC roles](../../_images/governance-1-18.png)
*Figure 10. Each user in the tenant is assigned one or more RBAC roles.*</span></span>

<span data-ttu-id="0adde-150">Роль RBAC указывает набор разрешений, которые пользователь имеет в определенном ресурсе.</span><span class="sxs-lookup"><span data-stu-id="0adde-150">An RBAC role specifies a set of permissions a user may take on a specific resource.</span></span> <span data-ttu-id="0adde-151">Эти разрешения применяются после назначения роли пользователю.</span><span class="sxs-lookup"><span data-stu-id="0adde-151">When the role is assigned to the user, those permissions are applied.</span></span> <span data-ttu-id="0adde-152">Например, пользователь со [встроенной ролью **владельца**](/azure/role-based-access-control/built-in-roles#owner) может выполнять любые действия в ресурсе.</span><span class="sxs-lookup"><span data-stu-id="0adde-152">For example, the [built-in **owner** role](/azure/role-based-access-control/built-in-roles#owner) allows a user to perform any action on a resource.</span></span>

<span data-ttu-id="0adde-153">На следующем этапе проверяется, разрешен ли запрос согласно параметрам, заданным в [политике ресурсов Azure](/azure/governance/policy/).</span><span class="sxs-lookup"><span data-stu-id="0adde-153">The next control is a check that the request is allowed under the settings specified for [Azure resource policy](/azure/governance/policy/).</span></span> <span data-ttu-id="0adde-154">Политики ресурсов Azure указывают операции, разрешенные в конкретном ресурсе.</span><span class="sxs-lookup"><span data-stu-id="0adde-154">Azure resource policies specify the operations allowed for a specific resource.</span></span> <span data-ttu-id="0adde-155">Например, в соответствии с политикой ресурсов Azure пользователи смогут развертывать только определенные типы виртуальной машины.</span><span class="sxs-lookup"><span data-stu-id="0adde-155">For example, an Azure resource policy can specify that users are only allowed to deploy a specific type of virtual machine.</span></span>

<span data-ttu-id="0adde-156">![Политика ресурсов Azure](../../_images/governance-1-19.png)
*Рис. 11. Политика ресурсов Azure.*</span><span class="sxs-lookup"><span data-stu-id="0adde-156">![Azure resource policy](../../_images/governance-1-19.png)
*Figure 11. Azure resource policy.*</span></span>

<span data-ttu-id="0adde-157">На следующем этапе проверяется, не превышает ли запрос [ограничения подписки Azure](/azure/azure-subscription-service-limits).</span><span class="sxs-lookup"><span data-stu-id="0adde-157">The next control is a check that the request does not exceed an [Azure subscription limit](/azure/azure-subscription-service-limits).</span></span> <span data-ttu-id="0adde-158">Например, каждая подписка может максимально содержать 980 групп ресурсов.</span><span class="sxs-lookup"><span data-stu-id="0adde-158">For example, each subscription has a limit of 980 resource groups per subscription.</span></span> <span data-ttu-id="0adde-159">Если получен запрос на развертывание дополнительной группы ресурсов после превышения этого ограничения, такой запрос будет отклонен.</span><span class="sxs-lookup"><span data-stu-id="0adde-159">If a request is received to deploy an additional resource group once the limit has been reached, it is denied.</span></span>

<span data-ttu-id="0adde-160">![Ограничения ресурсов Azure](../../_images/governance-1-20.png)
*Рис. 12. Ограничения ресурсов Azure.*</span><span class="sxs-lookup"><span data-stu-id="0adde-160">![Azure resource limits](../../_images/governance-1-20.png)
*Figure 12. Azure resource limits.*</span></span>

<span data-ttu-id="0adde-161">В последнюю очередь проверяется, связан ли запрос в рамках финансовых обязательств с подпиской.</span><span class="sxs-lookup"><span data-stu-id="0adde-161">The final control is a check that the request is within the financial commitment associated with the subscription.</span></span> <span data-ttu-id="0adde-162">Например, если отправляется запрос на развертывание виртуальной машины, Azure Resource Manager проверяет платежную информацию в подписке.</span><span class="sxs-lookup"><span data-stu-id="0adde-162">For example, if the request is to deploy a virtual machine, Azure Resource Manager verifies that the subscription has sufficient payment information.</span></span>

<span data-ttu-id="0adde-163">![Финансовое обязательство, связанное с подпиской](../../_images/governance-1-21.png)
*Рис. 13. Финансовое обязательство, связанное с подпиской.*</span><span class="sxs-lookup"><span data-stu-id="0adde-163">![A financial commitment associated with a subscription](../../_images/governance-1-21.png)
*Figure 13. A financial commitment is associated with a subscription.*</span></span>

## <a name="summary"></a><span data-ttu-id="0adde-164">Сводка</span><span class="sxs-lookup"><span data-stu-id="0adde-164">Summary</span></span>

<span data-ttu-id="0adde-165">Из этой статьи вы узнали об управлении доступом к ресурсам в Azure с помощью Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="0adde-165">In this article, you learned about how resource access is managed in Azure using Azure Resource Manager.</span></span>

## <a name="next-steps"></a><span data-ttu-id="0adde-166">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="0adde-166">Next steps</span></span>

<span data-ttu-id="0adde-167">Теперь, когда вы узнали об управлении доступом к ресурсам в Azure, узнайте о том, как разработать модель управления [для простой рабочей нагрузки](governance-simple-workload.md) или [для нескольких команд](governance-multiple-teams.md).</span><span class="sxs-lookup"><span data-stu-id="0adde-167">Now that you understand how to manage resource access in Azure, move on to learn how to design a governance model [for a simple workload](governance-simple-workload.md) or for [multiple teams](governance-multiple-teams.md) using these services.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="0adde-168">Общие сведения о системе управления</span><span class="sxs-lookup"><span data-stu-id="0adde-168">An overview of governance</span></span>](../overview.md)
