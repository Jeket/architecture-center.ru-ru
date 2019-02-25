---
title: CAF. Управление доступом к ресурсам в Azure
description: 'Сведения о компонентах управления доступом к ресурсам в Azure: Azure Resource Manager, подписках, ресурсах и их группах.'
author: petertaylor9999
ms.date: 2/11/2019
ms.openlocfilehash: f23540a03c82fbc46872645ac0fd82d574db353a
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55898125"
---
# <a name="resource-access-management-in-azure"></a><span data-ttu-id="cbd9c-103">Управление доступом к ресурсам в Azure</span><span class="sxs-lookup"><span data-stu-id="cbd9c-103">Resource access management in Azure</span></span>

<span data-ttu-id="cbd9c-104">Из статьи [Знакомство с системой управления облачными ресурсами](what-is-governance.md) вы узнали, что система управления ссылается на текущий процесс управления, мониторинга и аудита используемых ресурсов Azure для достижения целей и требований организации.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-104">In [what is resource governance?](what-is-governance.md), you learned that governance refers to the ongoing process of managing, monitoring, and auditing the use of Azure resources to meet the goals and requirements of your organization.</span></span> <span data-ttu-id="cbd9c-105">Прежде чем переходить к изучению принципов проектирования модели системы управления, важно разобраться с элементами управления доступом к ресурсам в Azure.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-105">Before you move on to learn how to design a governance model, it's important to understand the resource access management controls in Azure.</span></span> <span data-ttu-id="cbd9c-106">Конфигурация этих элементов управления доступом к ресурсами — это основа модели системы управления.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-106">The configuration of these resource access management controls forms the basis of your governance model.</span></span>

<span data-ttu-id="cbd9c-107">Сначала давайте рассмотрим принцип развертывания ресурсов в Azure.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-107">Let's begin by taking a closer look at how resources are deployed in Azure.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="what-is-an-azure-resource"></a><span data-ttu-id="cbd9c-108">Ресурсы Azure</span><span class="sxs-lookup"><span data-stu-id="cbd9c-108">What is an Azure resource?</span></span>

<span data-ttu-id="cbd9c-109">В Azure **ресурс** — это сущность под управлением Azure.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-109">In Azure, the term **resource** refers to an entity managed by Azure.</span></span> <span data-ttu-id="cbd9c-110">Например, к ресурсам относятся виртуальные машины, виртуальные сети и учетные записи хранения.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-110">For example, virtual machines, virtual networks, and storage accounts are all referred to as Azure resources.</span></span>

<span data-ttu-id="cbd9c-111">![](../_images/governance-1-9.png)
*Рис. 1. Ресурс.*</span><span class="sxs-lookup"><span data-stu-id="cbd9c-111">![](../_images/governance-1-9.png)
*Figure 1. A resource.*</span></span>

## <a name="what-is-an-azure-resource-group"></a><span data-ttu-id="cbd9c-112">Что такое группа ресурсов Azure?</span><span class="sxs-lookup"><span data-stu-id="cbd9c-112">What is an Azure resource group?</span></span>

<span data-ttu-id="cbd9c-113">Каждый ресурс в Azure должен принадлежать [группе ресурсов](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span><span class="sxs-lookup"><span data-stu-id="cbd9c-113">Each resource in Azure must belong to a [resource group](/azure/azure-resource-manager/resource-group-overview#resource-groups).</span></span> <span data-ttu-id="cbd9c-114">Группа ресурсов — это логическая конструкция, в которой сгруппированы несколько ресурсов, что позволяет управлять ими как единой сущностью.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-114">A resource group is simply a logical construct that groups multiple resources together so they can be managed as a single entity.</span></span> <span data-ttu-id="cbd9c-115">Например, ресурсы с аналогичным жизненным циклом, например ресурсы [n-уровневых приложений](/azure/architecture/guide/architecture-styles/n-tier), можно создавать или удалять как группу.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-115">For example, resources that share a similar lifecycle, such as the resources for an [n-tier application](/azure/architecture/guide/architecture-styles/n-tier) may be created or deleted as a group.</span></span>

<span data-ttu-id="cbd9c-116">![](../_images/governance-1-10.png)
*Рис. 2. Группа ресурсов, которая содержит ресурс.*</span><span class="sxs-lookup"><span data-stu-id="cbd9c-116">![](../_images/governance-1-10.png)
*Figure 2. A resource group contains a resource.*</span></span>

<span data-ttu-id="cbd9c-117">Группы и ресурсы в них связаны с **подпиской** Azure.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-117">Resource groups and the resources they contain are associated with an Azure **subscription**.</span></span>

## <a name="what-is-an-azure-subscription"></a><span data-ttu-id="cbd9c-118">Подписка Azure</span><span class="sxs-lookup"><span data-stu-id="cbd9c-118">What is an Azure subscription?</span></span>

<span data-ttu-id="cbd9c-119">Подписка Azure, как и группа ресурсов, — это логическая конструкция, в которой сгруппированы группы и ресурсы в них.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-119">An Azure subscription is similar to a resource group in that it's a logical construct that groups together resource groups and their resources.</span></span> <span data-ttu-id="cbd9c-120">Тем не менее подписка Azure также связана с элементами управления, используемыми Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-120">However, an Azure subscription is also associated with the controls used by Azure Resource Manager.</span></span> <span data-ttu-id="cbd9c-121">Что это означает?</span><span class="sxs-lookup"><span data-stu-id="cbd9c-121">What does this mean?</span></span> <span data-ttu-id="cbd9c-122">Давайте рассмотрим более подробно Resource Manager, чтобы больше узнать о связи между ним и подпиской Azure.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-122">Let's take a closer look at Resource Manager to learn about the relationship between it and an Azure subscription.</span></span>

<span data-ttu-id="cbd9c-123">![](../_images/governance-1-11.png)
*Рис. 3. Подписка Azure.*</span><span class="sxs-lookup"><span data-stu-id="cbd9c-123">![](../_images/governance-1-11.png)
*Figure 3. An Azure subscription.*</span></span>

## <a name="what-is-azure-resource-manager"></a><span data-ttu-id="cbd9c-124">Azure Resource Manager</span><span class="sxs-lookup"><span data-stu-id="cbd9c-124">What is Azure Resource Manager?</span></span>

<span data-ttu-id="cbd9c-125">Из статьи [Пояснения. Принцип работы Azure](what-is-azure.md) вы узнали, что Azure включает внешний интерфейс со многими службами, которые оркестрируют работу всех функции Azure.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-125">In [how does Azure work?](what-is-azure.md) you learned that Azure includes a "front end" with many services that orchestrate all the functions of Azure.</span></span> <span data-ttu-id="cbd9c-126">К этим службам относится [Resource Manager](/azure/azure-resource-manager/). В этой службе размещен интерфейс RESTful API, с помощью которого клиенты управляют ресурсами.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-126">One of these services is [Resource Manager](/azure/azure-resource-manager/), and this service hosts the RESTful API used by clients to manage resources.</span></span>

<span data-ttu-id="cbd9c-127">![](../_images/governance-1-12.png)
*Рис. 4. Azure Resource Manager.*</span><span class="sxs-lookup"><span data-stu-id="cbd9c-127">![](../_images/governance-1-12.png)
*Figure 4. Azure resource manager.*</span></span>

<span data-ttu-id="cbd9c-128">На следующем рисунке показано три клиента: [PowerShell](/powershell/azure/overview), [портал Azure](https://portal.azure.com) и [интерфейс командной строки (CLI) Azure](/cli/azure).</span><span class="sxs-lookup"><span data-stu-id="cbd9c-128">The following figure shows three clients: [PowerShell](/powershell/azure/overview), the [Azure portal](https://portal.azure.com), and the [Azure command line interface (CLI)](/cli/azure):</span></span>

<span data-ttu-id="cbd9c-129">![](../_images/governance-1-13.png)
*Рис. 5. Клиенты Azure подключаются к RESTful API Resource Manager.*</span><span class="sxs-lookup"><span data-stu-id="cbd9c-129">![](../_images/governance-1-13.png)
*Figure 5. Azure clients connect to the Resource Manager RESTful API.*</span></span>

<span data-ttu-id="cbd9c-130">Эти клиенты подключаются к Resource Manager через RESTful API. Однако Resource Manager не содержит функциональность, чтобы управлять ресурсами напрямую.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-130">While these clients connect to Resource Manager using the RESTful API, Resource Manager does not include functionality to manage resources directly.</span></span> <span data-ttu-id="cbd9c-131">Вместо этого большинство типов ресурсов в Azure имеют собственный [**поставщик ресурсов**](/azure/azure-resource-manager/resource-group-overview#terminology).</span><span class="sxs-lookup"><span data-stu-id="cbd9c-131">Rather, most resource types in Azure have their own [**resource provider**](/azure/azure-resource-manager/resource-group-overview#terminology).</span></span>

<span data-ttu-id="cbd9c-132">\*
*Рис. 6. Поставщики ресурсов Azure.*</span><span class="sxs-lookup"><span data-stu-id="cbd9c-132">![](../_images/governance-1-14.png)
*Figure 6. Azure resource providers.*</span></span>

<span data-ttu-id="cbd9c-133">Когда клиент отправляет запрос на управление определенным ресурсом, Resource Manager подключается к поставщику ресурсов этого типа ресурса, чтобы выполнить запрос.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-133">When a client makes a request to manage a specific resource, Resource Manager connects to the resource provider for that resource type to complete the request.</span></span> <span data-ttu-id="cbd9c-134">Например, если клиент отправляет запрос на управление ресурсами виртуальной машины, Resource Manager подключается к поставщику ресурсов **Microsoft.compute**.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-134">For example, if a client makes a request to manage a virtual machine resource, Resource Manager connects to the **Microsoft.Compute** resource provider.</span></span>

<span data-ttu-id="cbd9c-135">![](../_images/governance-1-15.png)
*Рис. 7. Resource Manager подключается к поставщику ресурсов **Microsoft.Compute**, чтобы управлять ресурсом, указанным в запросе клиента.*</span><span class="sxs-lookup"><span data-stu-id="cbd9c-135">![](../_images/governance-1-15.png)
*Figure 7. Resource Manager connects to the **Microsoft.Compute** resource provider to manage the resource specified in the client request.*</span></span>

<span data-ttu-id="cbd9c-136">В Resource Manager требуется, чтобы клиент указал идентификатор подписки и группы ресурсов и тем самым разрешил управлять ресурсами виртуальной машины.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-136">Resource Manager requires the client to specify an identifier for both the subscription and the resource group in order to manage the virtual machine resource.</span></span>

<span data-ttu-id="cbd9c-137">Теперь, когда вы имеете представление о принципе работы Resource Manager, давайте рассмотрим, как используемые этой службой элементы управления связаны с подпиской Azure.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-137">Now that you have an understanding of how Resource Manager works, let's return to our discussion of how an Azure subscription is associated with the controls used by Azure resource manager.</span></span> <span data-ttu-id="cbd9c-138">Перед выполнением Resource Manager любого запроса на управление ресурсом проверяется набор элементов управления.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-138">Before any resource management request can be executed by Resource Manager, a set of controls are checked.</span></span>

<span data-ttu-id="cbd9c-139">Сначала проверяется, выполняется ли запрос проверенным пользователем. Resource Manager имеет отношения доверия с [Azure Active Directory](/azure/active-directory/) (Azure AD), что позволяет обеспечить механизм удостоверения пользователя.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-139">The first control is that a request must be made by a validated user, and Resource Manager has a trusted relationship with [Azure Active Directory](/azure/active-directory/) (Azure AD) to provide user identity functionality.</span></span>

<span data-ttu-id="cbd9c-140">![](../_images/governance-1-16.png)
*Рис. 8. Azure Active Directory.*</span><span class="sxs-lookup"><span data-stu-id="cbd9c-140">![](../_images/governance-1-16.png)
*Figure 8. Azure Active Directory.*</span></span>

<span data-ttu-id="cbd9c-141">В Azure AD данные пользователей сегментируются по **клиентам**.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-141">In Azure AD, users are segmented into **tenants**.</span></span> <span data-ttu-id="cbd9c-142">Клиент — это логическая структура, представляющая собой защищенный выделенный экземпляр Azure AD, который, как правило, связан с организацией.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-142">A tenant is a logical construct that represents a secure, dedicated instance of Azure AD typically associated with an organization.</span></span> <span data-ttu-id="cbd9c-143">Каждая подписка связана с клиентом Azure AD.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-143">Each subscription is associated with an Azure AD tenant.</span></span>

<span data-ttu-id="cbd9c-144">![](../_images/governance-1-17.png)
*Рис. 9. Клиент Azure AD, связанный с подпиской.*</span><span class="sxs-lookup"><span data-stu-id="cbd9c-144">![](../_images/governance-1-17.png)
*Figure 9. An Azure AD tenant associated with a subscription.*</span></span>

<span data-ttu-id="cbd9c-145">Каждый запрос клиента на управление ресурсом в определенной подписке требует наличия у пользователя учетной записи в связанном клиенте Azure AD.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-145">Each client request to manage a resource in a particular subscription requires that the user has an account in the associated Azure AD tenant.</span></span>

<span data-ttu-id="cbd9c-146">Следующим проверяется, имеет ли пользователь необходимые права на выполнение запроса.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-146">The next control is a check that the user has sufficient permission to make the request.</span></span> <span data-ttu-id="cbd9c-147">Разрешения присваиваются пользователям с использованием механизма [управления доступом на основе ролей (RBAC)](/azure/role-based-access-control/).</span><span class="sxs-lookup"><span data-stu-id="cbd9c-147">Permissions are assigned to users using [role-based access control (RBAC)](/azure/role-based-access-control/).</span></span>

<span data-ttu-id="cbd9c-148">![](../_images/governance-1-18.png)
*Рис. 10. Каждому пользователю в клиенте назначается одна или несколько ролей RBAC.*</span><span class="sxs-lookup"><span data-stu-id="cbd9c-148">![](../_images/governance-1-18.png)
*Figure 10. Each user in the tenant is assigned one or more RBAC roles.*</span></span>

<span data-ttu-id="cbd9c-149">Роль RBAC указывает набор разрешений, которые пользователь имеет в определенном ресурсе.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-149">An RBAC role specifies a set of permissions a user may take on a specific resource.</span></span> <span data-ttu-id="cbd9c-150">Эти разрешения применяются после назначения роли пользователю.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-150">When the role is assigned to the user, those permissions are applied.</span></span> <span data-ttu-id="cbd9c-151">Например, пользователь со [встроенной ролью **владельца**](/azure/role-based-access-control/built-in-roles#owner) может выполнять любые действия в ресурсе.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-151">For example, the [built-in **owner** role](/azure/role-based-access-control/built-in-roles#owner) allows a user to perform any action on a resource.</span></span>

<span data-ttu-id="cbd9c-152">На следующем этапе проверяется, разрешен ли запрос согласно параметрам, заданным в [политике ресурсов Azure](/azure/governance/policy/).</span><span class="sxs-lookup"><span data-stu-id="cbd9c-152">The next control is a check that the request is allowed under the settings specified for [Azure resource policy](/azure/governance/policy/).</span></span> <span data-ttu-id="cbd9c-153">Политики ресурсов Azure указывают операции, разрешенные в конкретном ресурсе.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-153">Azure resource policies specify the operations allowed for a specific resource.</span></span> <span data-ttu-id="cbd9c-154">Например, в соответствии с политикой ресурсов Azure пользователи смогут развертывать только определенные типы виртуальной машины.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-154">For example, an Azure resource policy can specify that users are only allowed to deploy a specific type of virtual machine.</span></span>

<span data-ttu-id="cbd9c-155">![](../_images/governance-1-19.png)
*Рис. 11. Политика ресурсов Azure.*</span><span class="sxs-lookup"><span data-stu-id="cbd9c-155">![](../_images/governance-1-19.png)
*Figure 11. Azure resource policy.*</span></span>

<span data-ttu-id="cbd9c-156">На следующем этапе проверяется, не превышает ли запрос [ограничения подписки Azure](/azure/azure-subscription-service-limits).</span><span class="sxs-lookup"><span data-stu-id="cbd9c-156">The next control is a check that the request does not exceed an [Azure subscription limit](/azure/azure-subscription-service-limits).</span></span> <span data-ttu-id="cbd9c-157">Например, каждая подписка может максимально содержать 980 групп ресурсов.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-157">For example, each subscription has a limit of 980 resource groups per subscription.</span></span> <span data-ttu-id="cbd9c-158">Если получен запрос на развертывание дополнительной группы ресурсов после превышения этого ограничения, такой запрос будет отклонен.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-158">If a request is received to deploy an additional resource group once the limit has been reached, it is denied.</span></span>

<span data-ttu-id="cbd9c-159">![](../_images/governance-1-20.png)
*Рис. 12. Ограничения ресурсов Azure.*</span><span class="sxs-lookup"><span data-stu-id="cbd9c-159">![](../_images/governance-1-20.png)
*Figure 12. Azure resource limits.*</span></span>

<span data-ttu-id="cbd9c-160">В последнюю очередь проверяется, связан ли запрос в рамках финансовых обязательств с подпиской.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-160">The final control is a check that the request is within the financial commitment associated with the subscription.</span></span> <span data-ttu-id="cbd9c-161">Например, если отправляется запрос на развертывание виртуальной машины, Resource Manager проверяет платежную информацию в подписке.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-161">For example, if the request is to deploy a virtual machine, Resource Manager verifies that the subscription has sufficient payment information.</span></span>

<span data-ttu-id="cbd9c-162">![](../_images/governance-1-21.png)
*Рис. 13. Финансовое обязательство, связанное с подпиской.*</span><span class="sxs-lookup"><span data-stu-id="cbd9c-162">![](../_images/governance-1-21.png)
*Figure 13. A financial commitment is associated with a subscription.*</span></span>

## <a name="summary"></a><span data-ttu-id="cbd9c-163">Сводка</span><span class="sxs-lookup"><span data-stu-id="cbd9c-163">Summary</span></span>

<span data-ttu-id="cbd9c-164">Из этой статьи вы узнали об управлении доступом к ресурсам в Azure с помощью Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-164">In this article, you learned about how resource access is managed in Azure using Azure Resource Manager.</span></span>

## <a name="next-steps"></a><span data-ttu-id="cbd9c-165">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="cbd9c-165">Next steps</span></span>

<span data-ttu-id="cbd9c-166">После того как вы узнали об управлении доступом к ресурсам в Azure, ознакомьтесь со сведениями о проектировании модели системы управления.</span><span class="sxs-lookup"><span data-stu-id="cbd9c-166">Now that you understand how resource access is managed in Azure, move on to learn how to design a governance model.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="cbd9c-167">Внедрение облачных решений в организации. Обзор системы управления</span><span class="sxs-lookup"><span data-stu-id="cbd9c-167">Cloud governance</span></span>](../governance/overview.md)
