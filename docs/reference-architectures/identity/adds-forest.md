---
title: Создание леса ресурсов доменных служб Active Directory в Azure
description: >-
  Инструкции по созданию доверенного домена Active Directory в Azure.

  рекомендации,vpn-шлюз,expressroute,подсистема балансировки нагрузки,виртуальная сеть,active directory
author: telmosampaio
ms.date: 05/02/2018
pnp.series.title: Identity management
pnp.series.prev: adds-extend-domain
pnp.series.next: adfs
cardTitle: Create an AD DS forest in Azure
ms.openlocfilehash: 105ffc4b329331522529161731b1947f99e02280
ms.sourcegitcommit: 58d93e7ac9a6d44d5668a187a6827d7cd4f5a34d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/02/2018
ms.locfileid: "37142205"
---
# <a name="create-an-active-directory-domain-services-ad-ds-resource-forest-in-azure"></a><span data-ttu-id="d6519-104">Создание леса ресурсов доменных служб Active Directory (AD DS) в Azure</span><span class="sxs-lookup"><span data-stu-id="d6519-104">Create an Active Directory Domain Services (AD DS) resource forest in Azure</span></span>

<span data-ttu-id="d6519-105">На схеме эталонной архитектуры представлены данные о создании отдельного домена Active Directory в Azure, который является доверенным для доменов в локальном лесу AD.</span><span class="sxs-lookup"><span data-stu-id="d6519-105">This reference architecture shows how to create a separate Active Directory domain in Azure that is trusted by domains in your on-premises AD forest.</span></span> [<span data-ttu-id="d6519-106">**Разверните это решение**.</span><span class="sxs-lookup"><span data-stu-id="d6519-106">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="d6519-107">[![0]][0]</span><span class="sxs-lookup"><span data-stu-id="d6519-107">[![0]][0]</span></span> 

<span data-ttu-id="d6519-108">*Скачайте [файл Visio][visio-download] этой архитектуры.*</span><span class="sxs-lookup"><span data-stu-id="d6519-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="d6519-109">Доменные службы Active Directory (AD DS) хранят сведения об удостоверении в виде иерархической структуры.</span><span class="sxs-lookup"><span data-stu-id="d6519-109">Active Directory Domain Services (AD DS) stores identity information in a hierarchical structure.</span></span> <span data-ttu-id="d6519-110">Верхний узел в иерархической структуре называется лесом.</span><span class="sxs-lookup"><span data-stu-id="d6519-110">The top node in the hierarchical structure is known as a forest.</span></span> <span data-ttu-id="d6519-111">Лес содержит домены, которые, в свою очередь, содержат объекты других типов.</span><span class="sxs-lookup"><span data-stu-id="d6519-111">A forest contains domains, and domains contain other types of objects.</span></span> <span data-ttu-id="d6519-112">Эталонная архитектура создает лес AD DS в Azure с односторонним исходящим отношением доверия с локальным доменом.</span><span class="sxs-lookup"><span data-stu-id="d6519-112">This reference architecture creates an AD DS forest in Azure with a one-way outgoing trust relationship with an on-premises domain.</span></span> <span data-ttu-id="d6519-113">Лес в Azure содержит домен, отсутствующий в локальной среде.</span><span class="sxs-lookup"><span data-stu-id="d6519-113">The forest in Azure contains a domain that does not exist on-premises.</span></span> <span data-ttu-id="d6519-114">Благодаря отношению доверия операции входа в локальные домены могут быть доверенными и получать доступ к ресурсам в отдельном домене Azure.</span><span class="sxs-lookup"><span data-stu-id="d6519-114">Because of the trust relationship, logons made against on-premises domains can be trusted for access to resources in the separate Azure domain.</span></span> 

<span data-ttu-id="d6519-115">Типичные способы применения этой архитектуры включают поддержку разделения по безопасности для облачных объектов и удостоверений, а также перенос отдельных доменов из локальной среды в облако.</span><span class="sxs-lookup"><span data-stu-id="d6519-115">Typical uses for this architecture include maintaining security separation for objects and identities held in the cloud, and migrating individual domains from on-premises to the cloud.</span></span> 

<span data-ttu-id="d6519-116">Дополнительные рекомендации см. в статье [Выбор решения для интеграции локальной среды Active Directory с Azure][considerations].</span><span class="sxs-lookup"><span data-stu-id="d6519-116">For additional considerations, see [Choose a solution for integrating on-premises Active Directory with Azure][considerations].</span></span> 

## <a name="architecture"></a><span data-ttu-id="d6519-117">Архитектура</span><span class="sxs-lookup"><span data-stu-id="d6519-117">Architecture</span></span>

<span data-ttu-id="d6519-118">Архитектура состоит из следующих компонентов.</span><span class="sxs-lookup"><span data-stu-id="d6519-118">The architecture has the following components.</span></span>

* <span data-ttu-id="d6519-119">**Локальная сеть**.</span><span class="sxs-lookup"><span data-stu-id="d6519-119">**On-premises network**.</span></span> <span data-ttu-id="d6519-120">Локальная сеть содержит собственные лес и домены Active Directory.</span><span class="sxs-lookup"><span data-stu-id="d6519-120">The on-premises network contains its own Active Directory forest and domains.</span></span>
* <span data-ttu-id="d6519-121">**Серверы Active Directory**.</span><span class="sxs-lookup"><span data-stu-id="d6519-121">**Active Directory servers**.</span></span> <span data-ttu-id="d6519-122">Они являются контроллерами домена, которые реализуют службы домена, работающие в облаке в качестве виртуальных машин.</span><span class="sxs-lookup"><span data-stu-id="d6519-122">These are domain controllers implementing domain services running as VMs in the cloud.</span></span> <span data-ttu-id="d6519-123">На этих серверах размещается лес с одним или несколькими доменами, отдельными от локальных.</span><span class="sxs-lookup"><span data-stu-id="d6519-123">These servers host a forest containing one or more domains, separate from those located on-premises.</span></span>
* <span data-ttu-id="d6519-124">**Одностороннее отношение доверия**.</span><span class="sxs-lookup"><span data-stu-id="d6519-124">**One-way trust relationship**.</span></span> <span data-ttu-id="d6519-125">В примере на этой диаграмме показано одностороннее отношение доверия между доменом в Azure и локальным доменом.</span><span class="sxs-lookup"><span data-stu-id="d6519-125">The example in the diagram shows a one-way trust from the domain in Azure to the on-premises domain.</span></span> <span data-ttu-id="d6519-126">Это отношение позволяет локальным пользователям получать доступ к ресурсам в домене в Azure, но не наоборот.</span><span class="sxs-lookup"><span data-stu-id="d6519-126">This relationship enables on-premises users to access resources in the domain in Azure, but not the other way around.</span></span> <span data-ttu-id="d6519-127">Если пользователям облака также требуется доступ к локальным ресурсам, можно создать двустороннее отношение доверия.</span><span class="sxs-lookup"><span data-stu-id="d6519-127">It is possible to create a two-way trust if cloud users also require access to on-premises resources.</span></span>
* <span data-ttu-id="d6519-128">**Подсеть Active Directory**.</span><span class="sxs-lookup"><span data-stu-id="d6519-128">**Active Directory subnet**.</span></span> <span data-ttu-id="d6519-129">Серверы доменных служб Active Directory находятся в отдельной подсети.</span><span class="sxs-lookup"><span data-stu-id="d6519-129">The AD DS servers are hosted in a separate subnet.</span></span> <span data-ttu-id="d6519-130">Правила группы безопасности сети (NSG) защищают серверы доменных служб Active Directory и предоставляют брандмауэр для трафика из неизвестных источников.</span><span class="sxs-lookup"><span data-stu-id="d6519-130">Network security group (NSG) rules protect the AD DS servers and provide a firewall against traffic from unexpected sources.</span></span>
* <span data-ttu-id="d6519-131">**Шлюз Azure**.</span><span class="sxs-lookup"><span data-stu-id="d6519-131">**Azure gateway**.</span></span> <span data-ttu-id="d6519-132">Шлюз Azure обеспечивает соединение между локальной и виртуальной сетями Azure.</span><span class="sxs-lookup"><span data-stu-id="d6519-132">The Azure gateway provides a connection between the on-premises network and the Azure VNet.</span></span> <span data-ttu-id="d6519-133">Это может быть [VPN-подключение][azure-vpn-gateway] или [Azure ExpressRoute][azure-expressroute].</span><span class="sxs-lookup"><span data-stu-id="d6519-133">This can be a [VPN connection][azure-vpn-gateway] or [Azure ExpressRoute][azure-expressroute].</span></span> <span data-ttu-id="d6519-134">Дополнительные сведения см. в статье [DMZ between Azure and your on-premises datacenter] (Сеть периметра между Azure и локальным центром обработки данных) [implementing-a-secure-hybrid-network-architecture].</span><span class="sxs-lookup"><span data-stu-id="d6519-134">For more information, see [Implementing a secure hybrid network architecture in Azure][implementing-a-secure-hybrid-network-architecture].</span></span>

## <a name="recommendations"></a><span data-ttu-id="d6519-135">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="d6519-135">Recommendations</span></span>

<span data-ttu-id="d6519-136">Конкретные рекомендации по реализации Active Directory в Azure см. в следующих статьях:</span><span class="sxs-lookup"><span data-stu-id="d6519-136">For specific recommendations on implementing Active Directory in Azure, see the following articles:</span></span>

- <span data-ttu-id="d6519-137">[Расширение доменных служб Active Directory в Azure][adds-extend-domain].</span><span class="sxs-lookup"><span data-stu-id="d6519-137">[Extending Active Directory Domain Services (AD DS) to Azure][adds-extend-domain].</span></span> 
- <span data-ttu-id="d6519-138">[Руководства по развертыванию Windows Server Active Directory на виртуальных машинах Azure][ad-azure-guidelines].</span><span class="sxs-lookup"><span data-stu-id="d6519-138">[Guidelines for Deploying Windows Server Active Directory on Azure Virtual Machines][ad-azure-guidelines].</span></span>

### <a name="trust"></a><span data-ttu-id="d6519-139">Доверие</span><span class="sxs-lookup"><span data-stu-id="d6519-139">Trust</span></span>

<span data-ttu-id="d6519-140">Локальные домены находятся в другом лесу, отличном от леса с доменами в облаке.</span><span class="sxs-lookup"><span data-stu-id="d6519-140">The on-premises domains are contained within a different forest from the domains in the cloud.</span></span> <span data-ttu-id="d6519-141">Чтобы включить аутентификацию локальных пользователей в облаке, домены в Azure должны доверять домену входа в систему в локальном лесу.</span><span class="sxs-lookup"><span data-stu-id="d6519-141">To enable authentication of on-premises users in the cloud, the domains in Azure must trust the logon domain in the on-premises forest.</span></span> <span data-ttu-id="d6519-142">Аналогично, если облако предоставляет домен входа в систему для внешних пользователей, возможно, понадобится настроить отношение доверия между локальным лесом и облачным доменом.</span><span class="sxs-lookup"><span data-stu-id="d6519-142">Similarly, if the cloud provides a logon domain for external users, it may be necessary for the on-premises forest to trust the cloud domain.</span></span>

<span data-ttu-id="d6519-143">Вы можете установить доверие на уровне леса, [создав отношения доверия лесов][creating-forest-trusts], или на уровне домена, [создав отношения внешнего доверия][creating-external-trusts].</span><span class="sxs-lookup"><span data-stu-id="d6519-143">You can establish trusts at the forest level by [creating forest trusts][creating-forest-trusts], or at the domain level by [creating external trusts][creating-external-trusts].</span></span> <span data-ttu-id="d6519-144">Уровень доверия леса создает связь между всеми доменами в двух лесах.</span><span class="sxs-lookup"><span data-stu-id="d6519-144">A forest level trust creates a relationship between all domains in two forests.</span></span> <span data-ttu-id="d6519-145">Доверительные отношения на уровне внешнего домена создают связь только между двумя указанными доменами.</span><span class="sxs-lookup"><span data-stu-id="d6519-145">An external domain level trust only creates a relationship between two specified domains.</span></span> <span data-ttu-id="d6519-146">Вам необходимо только создать доверительные отношения на уровне внешнего домена между доменами в разных лесах.</span><span class="sxs-lookup"><span data-stu-id="d6519-146">You should only create external domain level trusts between domains in different forests.</span></span>

<span data-ttu-id="d6519-147">Отношения доверия могут быть однонаправленными (односторонними) или двунаправленными (двусторонними):</span><span class="sxs-lookup"><span data-stu-id="d6519-147">Trusts can be unidirectional (one-way) or bidirectional (two-way):</span></span>

* <span data-ttu-id="d6519-148">Одностороннее отношение доверия позволяет пользователям из одного домена или леса (известного как *входящий* домен или лес) получать доступ к ресурсам, которые содержатся в другом (*исходящем* домене или лесу).</span><span class="sxs-lookup"><span data-stu-id="d6519-148">A one-way trust enables users in one domain or forest (known as the *incoming* domain or forest) to access the resources held in another (the *outgoing* domain or forest).</span></span>
* <span data-ttu-id="d6519-149">Двустороннее отношение доверия позволяет пользователям в домене или лесу получать доступ к ресурсам, которые содержатся в другом домене или лесу.</span><span class="sxs-lookup"><span data-stu-id="d6519-149">A two-way trust enables users in either domain or forest to access resources held in the other.</span></span>

<span data-ttu-id="d6519-150">В следующей таблице перечислены конфигурации доверия для некоторых простых сценариев:</span><span class="sxs-lookup"><span data-stu-id="d6519-150">The following table summarizes trust configurations for some simple scenarios:</span></span>

| <span data-ttu-id="d6519-151">Сценарий</span><span class="sxs-lookup"><span data-stu-id="d6519-151">Scenario</span></span> | <span data-ttu-id="d6519-152">Локальное отношение доверия</span><span class="sxs-lookup"><span data-stu-id="d6519-152">On-premises trust</span></span> | <span data-ttu-id="d6519-153">Отношение доверия облака</span><span class="sxs-lookup"><span data-stu-id="d6519-153">Cloud trust</span></span> |
| --- | --- | --- |
| <span data-ttu-id="d6519-154">Локальным пользователям требуется доступ к ресурсам в облаке, но не наоборот</span><span class="sxs-lookup"><span data-stu-id="d6519-154">On-premises users require access to resources in the cloud, but not vice versa</span></span> |<span data-ttu-id="d6519-155">Односторонний, входящий трафик</span><span class="sxs-lookup"><span data-stu-id="d6519-155">One-way, incoming</span></span> |<span data-ttu-id="d6519-156">Односторонний, исходящий трафик</span><span class="sxs-lookup"><span data-stu-id="d6519-156">One-way, outgoing</span></span> |
| <span data-ttu-id="d6519-157">Пользователям в облаке требуется доступ к локальным ресурсам, но не наоборот</span><span class="sxs-lookup"><span data-stu-id="d6519-157">Users in the cloud require access to resources located on-premises, but not vice versa</span></span> |<span data-ttu-id="d6519-158">Односторонний, исходящий трафик</span><span class="sxs-lookup"><span data-stu-id="d6519-158">One-way, outgoing</span></span> |<span data-ttu-id="d6519-159">Односторонний, входящий трафик</span><span class="sxs-lookup"><span data-stu-id="d6519-159">One-way, incoming</span></span> |
| <span data-ttu-id="d6519-160">Пользователям в облаке и локальной среде требуется доступ к ресурсам, хранящимся в облаке и в локальной среде</span><span class="sxs-lookup"><span data-stu-id="d6519-160">Users in the cloud and on-premises both requires access to resources held in the cloud and on-premises</span></span> |<span data-ttu-id="d6519-161">Двусторонний, входящий и исходящий трафик</span><span class="sxs-lookup"><span data-stu-id="d6519-161">Two-way, incoming and outgoing</span></span> |<span data-ttu-id="d6519-162">Двусторонний, входящий и исходящий трафик</span><span class="sxs-lookup"><span data-stu-id="d6519-162">Two-way, incoming and outgoing</span></span> |

## <a name="scalability-considerations"></a><span data-ttu-id="d6519-163">Вопросы масштабируемости</span><span class="sxs-lookup"><span data-stu-id="d6519-163">Scalability considerations</span></span>

<span data-ttu-id="d6519-164">Active Directory автоматически масштабируется для контроллеров, которые являются частью одного домена.</span><span class="sxs-lookup"><span data-stu-id="d6519-164">Active Directory is automatically scalable for domain controllers that are part of the same domain.</span></span> <span data-ttu-id="d6519-165">Запросы распространяются на все контроллеры в домене.</span><span class="sxs-lookup"><span data-stu-id="d6519-165">Requests are distributed across all controllers within a domain.</span></span> <span data-ttu-id="d6519-166">Вы можете добавить еще один контроллер домена, и он автоматически синхронизируется с доменом.</span><span class="sxs-lookup"><span data-stu-id="d6519-166">You can add another domain controller, and it synchronizes automatically with the domain.</span></span> <span data-ttu-id="d6519-167">Не настраивайте отдельную подсистему балансировки нагрузки для направления трафика в контроллерах в домене.</span><span class="sxs-lookup"><span data-stu-id="d6519-167">Do not configure a separate load balancer to direct traffic to controllers within the domain.</span></span> <span data-ttu-id="d6519-168">Убедитесь, что все контроллеры домена имеют достаточно ресурсов памяти и хранилища для обработки базы данных домена.</span><span class="sxs-lookup"><span data-stu-id="d6519-168">Ensure that all domain controllers have sufficient memory and storage resources to handle the domain database.</span></span> <span data-ttu-id="d6519-169">Сделайте все виртуальные машины контроллера домена одного размера.</span><span class="sxs-lookup"><span data-stu-id="d6519-169">Make all domain controller VMs the same size.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="d6519-170">Вопросы доступности</span><span class="sxs-lookup"><span data-stu-id="d6519-170">Availability considerations</span></span>

<span data-ttu-id="d6519-171">Подготовьте по крайней мере два контроллера домена для каждого домена.</span><span class="sxs-lookup"><span data-stu-id="d6519-171">Provision at least two domain controllers for each domain.</span></span> <span data-ttu-id="d6519-172">Это позволяет выполнять репликацию между серверами автоматически.</span><span class="sxs-lookup"><span data-stu-id="d6519-172">This enables automatic replication between servers.</span></span> <span data-ttu-id="d6519-173">Создайте группу доступности для виртуальных машин, действующих в качестве серверов Active Directory, обрабатывающих каждый домен.</span><span class="sxs-lookup"><span data-stu-id="d6519-173">Create an availability set for the VMs acting as Active Directory servers handling each domain.</span></span> <span data-ttu-id="d6519-174">Поместите по крайней мере два сервера в эту группу доступности.</span><span class="sxs-lookup"><span data-stu-id="d6519-174">Put at least two servers in this availability set.</span></span>

<span data-ttu-id="d6519-175">Кроме того, рассмотрите вопрос о назначении одного или нескольких серверов в каждом домене в качестве [владельцев резервных операций][standby-operations-masters] в случае сбоя подключения к серверу, который выступает в качестве владельца роли FSMO.</span><span class="sxs-lookup"><span data-stu-id="d6519-175">Also, consider designating one or more servers in each domain as [standby operations masters][standby-operations-masters] in case connectivity to a server acting as a flexible single master operation (FSMO) role fails.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="d6519-176">Вопросы управляемости</span><span class="sxs-lookup"><span data-stu-id="d6519-176">Manageability considerations</span></span>

<span data-ttu-id="d6519-177">Дополнительные сведения и рекомендации по управлению и мониторингу см. в статье [Расширение доменных служб Active Directory в Azure][adds-extend-domain].</span><span class="sxs-lookup"><span data-stu-id="d6519-177">For information about management and monitoring considerations, see [Extending Active Directory to Azure][adds-extend-domain].</span></span> 
 
<span data-ttu-id="d6519-178">Дополнительные сведения см. в документе [Monitoring Active Directory][monitoring_ad] (Мониторинг Active Directory).</span><span class="sxs-lookup"><span data-stu-id="d6519-178">For additional information, see [Monitoring Active Directory][monitoring_ad].</span></span> <span data-ttu-id="d6519-179">Вы можете установить инструменты (например, [Microsoft Systems Center][microsoft_systems_center]) на сервере мониторинга в подсети управления для выполнения этих задач.</span><span class="sxs-lookup"><span data-stu-id="d6519-179">You can install tools such as [Microsoft Systems Center][microsoft_systems_center] on a monitoring server in the management subnet to help perform these tasks.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="d6519-180">Вопросы безопасности</span><span class="sxs-lookup"><span data-stu-id="d6519-180">Security considerations</span></span>

<span data-ttu-id="d6519-181">Отношения доверия на уровне леса являются транзитивными.</span><span class="sxs-lookup"><span data-stu-id="d6519-181">Forest level trusts are transitive.</span></span> <span data-ttu-id="d6519-182">Если установить отношение доверия на уровне леса между локальным и облачным лесами, оно распространяется на другие новые домены в любом лесу.</span><span class="sxs-lookup"><span data-stu-id="d6519-182">If you establish a forest level trust between an on-premises forest and a forest in the cloud, this trust is extended to other new domains created in either forest.</span></span> <span data-ttu-id="d6519-183">Если вы используете домены для обеспечения разделения в целях безопасности, рекомендуется создать доверительные отношения только на уровне домена.</span><span class="sxs-lookup"><span data-stu-id="d6519-183">If you use domains to provide separation for security purposes, consider creating trusts at the domain level only.</span></span> <span data-ttu-id="d6519-184">Отношения доверия на уровне домена являются нетранзитивными.</span><span class="sxs-lookup"><span data-stu-id="d6519-184">Domain level trusts are non-transitive.</span></span>

<span data-ttu-id="d6519-185">Рекомендации по безопасности, связанные с Active Directory, см. в разделе "Вопросы безопасности" в статье [Расширение доменных служб Active Directory в Azure][adds-extend-domain].</span><span class="sxs-lookup"><span data-stu-id="d6519-185">For Active Directory-specific security considerations, see the security considerations section in [Extending Active Directory to Azure][adds-extend-domain].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="d6519-186">Развертывание решения</span><span class="sxs-lookup"><span data-stu-id="d6519-186">Deploy the solution</span></span>

<span data-ttu-id="d6519-187">Пример развертывания для этой архитектуры можно найти на портале [GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="d6519-187">A deployment for this architecture is available on [GitHub][github].</span></span> <span data-ttu-id="d6519-188">Обратите внимание, что для полного развертывания может потребоваться до двух часов, включая создание VPN-шлюза и запуск скриптов, которые настраивают доменные службы Active Directory.</span><span class="sxs-lookup"><span data-stu-id="d6519-188">Note that the entire deployment can take up to two hours, which includes creating the VPN gateway and running the scripts that configure AD DS.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="d6519-189">предварительным требованиям</span><span class="sxs-lookup"><span data-stu-id="d6519-189">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="d6519-190">Развертывание имитации локального центра обработки данных</span><span class="sxs-lookup"><span data-stu-id="d6519-190">Deploy the simulated on-premises datacenter</span></span>

1. <span data-ttu-id="d6519-191">Перейдите в папку `identity/adds-forest` в репозитории GitHub.</span><span class="sxs-lookup"><span data-stu-id="d6519-191">Navigate to the `identity/adds-forest` folder of the GitHub repository.</span></span>

2. <span data-ttu-id="d6519-192">Откройте файл `onprem.json` .</span><span class="sxs-lookup"><span data-stu-id="d6519-192">Open the `onprem.json` file.</span></span> <span data-ttu-id="d6519-193">Найдите экземпляры `adminPassword` и `Password` и добавьте значения для паролей.</span><span class="sxs-lookup"><span data-stu-id="d6519-193">Search for instances of `adminPassword` and `Password` and add values for the passwords.</span></span>

3. <span data-ttu-id="d6519-194">Выполните следующую команду и дождитесь завершения развертывания:</span><span class="sxs-lookup"><span data-stu-id="d6519-194">Run the following command and wait for the deployment to finish:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onprem.json --deploy
    ```

### <a name="deploy-the-azure-vnet"></a><span data-ttu-id="d6519-195">Развертывание виртуальной сети Azure</span><span class="sxs-lookup"><span data-stu-id="d6519-195">Deploy the Azure VNet</span></span>

1. <span data-ttu-id="d6519-196">Откройте файл `azure.json` .</span><span class="sxs-lookup"><span data-stu-id="d6519-196">Open the `azure.json` file.</span></span> <span data-ttu-id="d6519-197">Найдите экземпляры `adminPassword` и `Password` и добавьте значения для паролей.</span><span class="sxs-lookup"><span data-stu-id="d6519-197">Search for instances of `adminPassword` and `Password` and add values for the passwords.</span></span>

2. <span data-ttu-id="d6519-198">В том же файле найдите экземпляры `sharedKey` и введите общие ключи для VPN-подключения.</span><span class="sxs-lookup"><span data-stu-id="d6519-198">In the same file, search for instances of `sharedKey` and enter shared keys for the VPN connection.</span></span> 

    ```bash
    "sharedKey": "",
    ```

3. <span data-ttu-id="d6519-199">Выполните следующую команду и дождитесь завершения развертывания.</span><span class="sxs-lookup"><span data-stu-id="d6519-199">Run the following command and wait for the deployment to finish.</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onoprem.json --deploy
    ```

   <span data-ttu-id="d6519-200">Разверните ту же группу ресурсов, что и локальная виртуальная сеть.</span><span class="sxs-lookup"><span data-stu-id="d6519-200">Deploy to the same resource group as the on-premises VNet.</span></span>


### <a name="test-the-ad-trust-relation"></a><span data-ttu-id="d6519-201">Проверка отношения доверия AD</span><span class="sxs-lookup"><span data-stu-id="d6519-201">Test the AD trust relation</span></span>

1. <span data-ttu-id="d6519-202">На портале Azure перейдите к созданной группе ресурсов.</span><span class="sxs-lookup"><span data-stu-id="d6519-202">Use the Azure portal, navigate to the resource group that you created.</span></span>

2. <span data-ttu-id="d6519-203">Используйте портал Azure для поиска виртуальной машины с именем `ra-adt-mgmt-vm1`.</span><span class="sxs-lookup"><span data-stu-id="d6519-203">Use the Azure portal to find the VM named `ra-adt-mgmt-vm1`.</span></span>

2. <span data-ttu-id="d6519-204">Нажмите `Connect`, чтобы открыть сеанс удаленного рабочего стола для виртуальной машины.</span><span class="sxs-lookup"><span data-stu-id="d6519-204">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="d6519-205">Имя пользователя — `contoso\testuser`, а пароль — тот, который указан в файле параметров `onprem.json`.</span><span class="sxs-lookup"><span data-stu-id="d6519-205">The username is `contoso\testuser`, and the password is the one that you specified in the `onprem.json` parameter file.</span></span>

3. <span data-ttu-id="d6519-206">Вовремя сеанса удаленного рабочего стола откройте еще один такой сеанс к виртуальной машине `ra-adtrust-onpremise-ad-vm1` с IP-адресом 192.168.0.4.</span><span class="sxs-lookup"><span data-stu-id="d6519-206">From inside your remote desktop session, open another remote desktop session to 192.168.0.4, which is the IP address of the VM named `ra-adtrust-onpremise-ad-vm1`.</span></span> <span data-ttu-id="d6519-207">Имя пользователя — `contoso\testuser`, а пароль — тот, который указан в файле параметров `azure.json`.</span><span class="sxs-lookup"><span data-stu-id="d6519-207">The username is `contoso\testuser`, and the password is the one that you specified in the `azure.json` parameter file.</span></span>

4. <span data-ttu-id="d6519-208">Во время сеанса подключения к удаленному рабочему столу для `ra-adtrust-onpremise-ad-vm1` перейдите к **диспетчеру сервера** и выберите **Средства** > **Active Directory —домены и доверие**.</span><span class="sxs-lookup"><span data-stu-id="d6519-208">From inside the remote desktop session for `ra-adtrust-onpremise-ad-vm1`, go to **Server Manager** and click **Tools** > **Active Directory Domains and Trusts**.</span></span> 

5. <span data-ttu-id="d6519-209">На панели слева щелкните правой кнопкой мыши contoso.com и выберите **Свойства**.</span><span class="sxs-lookup"><span data-stu-id="d6519-209">In the left pane, right-click on the contoso.com and select **Properties**.</span></span>

6. <span data-ttu-id="d6519-210">Щелкните вкладку **Доверие**. В списке должно появиться входящее отношение доверия treyresearch.net.</span><span class="sxs-lookup"><span data-stu-id="d6519-210">Click the **Trusts** tab. You should see treyresearch.net listed as an incoming trust.</span></span>

![](./images/ad-forest-trust.png)


## <a name="next-steps"></a><span data-ttu-id="d6519-211">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="d6519-211">Next steps</span></span>

* <span data-ttu-id="d6519-212">Изучите рекомендации по [расширению локальных доменов доменных служб Active Directory в Azure][adds-extend-domain]</span><span class="sxs-lookup"><span data-stu-id="d6519-212">Learn the best practices for [extending your on-premises AD DS domain to Azure][adds-extend-domain]</span></span>
* <span data-ttu-id="d6519-213">Изучите рекомендации по [созданию инфраструктуры служб федерации Active Directory (AD FS)][adfs] в Azure.</span><span class="sxs-lookup"><span data-stu-id="d6519-213">Learn the best practices for [creating an AD FS infrastructure][adfs] in Azure.</span></span>

<!-- links -->
[adds-extend-domain]: adds-extend-domain.md
[adfs]: adfs.md
[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[running-VMs-for-an-N-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[ad-azure-guidelines]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[azure-expressroute]: https://azure.microsoft.com/documentation/articles/expressroute-introduction/
[azure-vpn-gateway]: https://azure.microsoft.com/documentation/articles/vpn-gateway-about-vpngateways/
[considerations]: ./considerations.md
[creating-external-trusts]: https://technet.microsoft.com/library/cc816837(v=ws.10).aspx
[creating-forest-trusts]: https://technet.microsoft.com/library/cc816810(v=ws.10).aspx
[github]: https://github.com/mspnp/reference-architectures/tree/master/identity/adds-forest
[incoming-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/incoming-trust.ps1
[microsoft_systems_center]: https://www.microsoft.com/server-cloud/products/system-center-2016/
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[solution-script]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/Deploy-ReferenceArchitecture.ps1
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[outgoing-trust]: https://raw.githubusercontent.com/mspnp/reference-architectures/master/identity/adds-forest/extensions/outgoing-trust.ps1
[verify-a-trust]: https://technet.microsoft.com/library/cc753821.aspx
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
[0]: ./images/adds-forest.png "Безопасная гибридная сетевая архитектура с несколькими доменами Active Directory"