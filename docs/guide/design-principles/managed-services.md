---
title: Использование управляемых служб
titleSuffix: Azure Application Architecture Guide
description: При возможности выбирайте службы PaaS (платформа как услуга) вместо служб IaaS (инфраструктура как услуга).
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: f08914e1eacc4f02fdb16093fe590f46249e3da3
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54485814"
---
# <a name="use-managed-services"></a><span data-ttu-id="09ed6-103">Использование управляемых служб</span><span class="sxs-lookup"><span data-stu-id="09ed6-103">Use managed services</span></span>

## <a name="when-possible-use-platform-as-a-service-paas-rather-than-infrastructure-as-a-service-iaas"></a><span data-ttu-id="09ed6-104">При возможности выбирайте службы PaaS (платформа как услуга), а не IaaS (инфраструктура как услуга).</span><span class="sxs-lookup"><span data-stu-id="09ed6-104">When possible, use platform as a service (PaaS) rather than infrastructure as a service (IaaS)</span></span>

<span data-ttu-id="09ed6-105">IaaS предоставляет лишь набор компонентов.</span><span class="sxs-lookup"><span data-stu-id="09ed6-105">IaaS is like having a box of parts.</span></span> <span data-ttu-id="09ed6-106">Вы можете собрать из них что угодно, но придется делать это самостоятельно.</span><span class="sxs-lookup"><span data-stu-id="09ed6-106">You can build anything, but you have to assemble it yourself.</span></span> <span data-ttu-id="09ed6-107">Управляемые службы намного удобнее в настройке и администрировании.</span><span class="sxs-lookup"><span data-stu-id="09ed6-107">Managed services are easier to configure and administer.</span></span> <span data-ttu-id="09ed6-108">Вам не придется подготавливать виртуальные машины, настраивать виртуальные сети, управлять исправлениями и обновлениями, а также беспокоиться о множестве других проблем, связанных с работой программного обеспечения на виртуальной машине.</span><span class="sxs-lookup"><span data-stu-id="09ed6-108">You don't need to provision VMs, set up VNets, manage patches and updates, and all of the other overhead associated with running software on a VM.</span></span>

<span data-ttu-id="09ed6-109">Предположим, что в вашем приложении нужна очередь сообщений.</span><span class="sxs-lookup"><span data-stu-id="09ed6-109">For example, suppose your application needs a message queue.</span></span> <span data-ttu-id="09ed6-110">Чтобы настроить собственную службу обмена сообщениями на виртуальной машине,вам потребуется установить, например, RabbitMQ.</span><span class="sxs-lookup"><span data-stu-id="09ed6-110">You could set up your own messaging service on a VM, using something like RabbitMQ.</span></span> <span data-ttu-id="09ed6-111">А ведь служебная шина Azure уже предоставляет надежную службу для обмена сообщениями, и ее гораздо проще настроить.</span><span class="sxs-lookup"><span data-stu-id="09ed6-111">But Azure Service Bus already provides reliable messaging as service, and it's simpler to set up.</span></span> <span data-ttu-id="09ed6-112">Достаточно создать пространство имен служебной шины (например, прямо в скрипте развертывания) и обратиться к ней через клиентский пакет SDK.</span><span class="sxs-lookup"><span data-stu-id="09ed6-112">Just create a Service Bus namespace (which can be done as part of a deployment script) and then call Service Bus using the client SDK.</span></span>

<span data-ttu-id="09ed6-113">Конечно, у вашего приложения могут быть особые требования, для которых модель IaaS подходит лучше.</span><span class="sxs-lookup"><span data-stu-id="09ed6-113">Of course, your application may have specific requirements that make an IaaS approach more suitable.</span></span> <span data-ttu-id="09ed6-114">Но даже для приложений на основе службы IaaS можно найти возможность естественного применения управляемых служб.</span><span class="sxs-lookup"><span data-stu-id="09ed6-114">However, even if your application is based on IaaS, look for places where it may be natural to incorporate managed services.</span></span> <span data-ttu-id="09ed6-115">Например, служб кэширования, управления очередями и хранилища данных.</span><span class="sxs-lookup"><span data-stu-id="09ed6-115">These include cache, queues, and data storage.</span></span>

| <span data-ttu-id="09ed6-116">Вместо службы...</span><span class="sxs-lookup"><span data-stu-id="09ed6-116">Instead of running...</span></span> | <span data-ttu-id="09ed6-117">рекомендуем использовать...</span><span class="sxs-lookup"><span data-stu-id="09ed6-117">Consider using...</span></span> |
|-----------------------|-------------|
| <span data-ttu-id="09ed6-118">Active Directory</span><span class="sxs-lookup"><span data-stu-id="09ed6-118">Active Directory</span></span> | <span data-ttu-id="09ed6-119">Доменные службы Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="09ed6-119">Azure Active Directory Domain Services</span></span> |
| <span data-ttu-id="09ed6-120">Elasticsearch</span><span class="sxs-lookup"><span data-stu-id="09ed6-120">Elasticsearch</span></span> | <span data-ttu-id="09ed6-121">Поиск Azure</span><span class="sxs-lookup"><span data-stu-id="09ed6-121">Azure Search</span></span> |
| <span data-ttu-id="09ed6-122">Hadoop</span><span class="sxs-lookup"><span data-stu-id="09ed6-122">Hadoop</span></span> | <span data-ttu-id="09ed6-123">HDInsight</span><span class="sxs-lookup"><span data-stu-id="09ed6-123">HDInsight</span></span> |
| <span data-ttu-id="09ed6-124">IIS</span><span class="sxs-lookup"><span data-stu-id="09ed6-124">IIS</span></span> | <span data-ttu-id="09ed6-125">Служба приложений</span><span class="sxs-lookup"><span data-stu-id="09ed6-125">App Service</span></span> |
| <span data-ttu-id="09ed6-126">MongoDB</span><span class="sxs-lookup"><span data-stu-id="09ed6-126">MongoDB</span></span> | <span data-ttu-id="09ed6-127">База данных Cosmos</span><span class="sxs-lookup"><span data-stu-id="09ed6-127">Cosmos DB</span></span> |
| <span data-ttu-id="09ed6-128">Redis</span><span class="sxs-lookup"><span data-stu-id="09ed6-128">Redis</span></span> | <span data-ttu-id="09ed6-129">кэш Azure Redis</span><span class="sxs-lookup"><span data-stu-id="09ed6-129">Azure Redis Cache</span></span> |
| <span data-ttu-id="09ed6-130">SQL Server;</span><span class="sxs-lookup"><span data-stu-id="09ed6-130">SQL Server</span></span> | <span data-ttu-id="09ed6-131">Базы данных SQL Azure</span><span class="sxs-lookup"><span data-stu-id="09ed6-131">Azure SQL Database</span></span> |
