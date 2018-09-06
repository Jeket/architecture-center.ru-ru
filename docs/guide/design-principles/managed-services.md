---
title: Использование управляемых служб
description: При возможности выбирайте службы PaaS (платформа как услуга) вместо служб IaaS (инфраструктура как услуга).
author: MikeWasson
ms.date: 08/30/2018
ms.openlocfilehash: f6777a19e126a8a7f64be05dfad9bc503d27b1c3
ms.sourcegitcommit: ae8a1de6f4af7a89a66a8339879843d945201f85
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/31/2018
ms.locfileid: "43325780"
---
# <a name="use-managed-services"></a><span data-ttu-id="309ed-103">Использование управляемых служб</span><span class="sxs-lookup"><span data-stu-id="309ed-103">Use managed services</span></span>

## <a name="when-possible-use-platform-as-a-service-paas-rather-than-infrastructure-as-a-service-iaas"></a><span data-ttu-id="309ed-104">При возможности выбирайте службы PaaS (платформа как услуга), а не IaaS (инфраструктура как услуга).</span><span class="sxs-lookup"><span data-stu-id="309ed-104">When possible, use platform as a service (PaaS) rather than infrastructure as a service (IaaS)</span></span>

<span data-ttu-id="309ed-105">IaaS предоставляет лишь набор компонентов.</span><span class="sxs-lookup"><span data-stu-id="309ed-105">IaaS is like having a box of parts.</span></span> <span data-ttu-id="309ed-106">Вы можете собрать из них что угодно, но придется делать это самостоятельно.</span><span class="sxs-lookup"><span data-stu-id="309ed-106">You can build anything, but you have to assemble it yourself.</span></span> <span data-ttu-id="309ed-107">Управляемые службы намного удобнее в настройке и администрировании.</span><span class="sxs-lookup"><span data-stu-id="309ed-107">Managed services are easier to configure and administer.</span></span> <span data-ttu-id="309ed-108">Вам не придется подготавливать виртуальные машины, настраивать виртуальные сети, управлять исправлениями и обновлениями, а также беспокоиться о множестве других проблем, связанных с работой программного обеспечения на виртуальной машине.</span><span class="sxs-lookup"><span data-stu-id="309ed-108">You don't need to provision VMs, set up VNets, manage patches and updates, and all of the other overhead associated with running software on a VM.</span></span>

<span data-ttu-id="309ed-109">Предположим, что в вашем приложении нужна очередь сообщений.</span><span class="sxs-lookup"><span data-stu-id="309ed-109">For example, suppose your application needs a message queue.</span></span> <span data-ttu-id="309ed-110">Чтобы настроить собственную службу обмена сообщениями на виртуальной машине,вам потребуется установить, например, RabbitMQ.</span><span class="sxs-lookup"><span data-stu-id="309ed-110">You could set up your own messaging service on a VM, using something like RabbitMQ.</span></span> <span data-ttu-id="309ed-111">А ведь служебная шина Azure уже предоставляет надежную службу для обмена сообщениями, и ее гораздо проще настроить.</span><span class="sxs-lookup"><span data-stu-id="309ed-111">But Azure Service Bus already provides reliable messaging as service, and it's simpler to set up.</span></span> <span data-ttu-id="309ed-112">Достаточно создать пространство имен служебной шины (например, прямо в скрипте развертывания) и обратиться к ней через клиентский пакет SDK.</span><span class="sxs-lookup"><span data-stu-id="309ed-112">Just create a Service Bus namespace (which can be done as part of a deployment script) and then call Service Bus using the client SDK.</span></span> 

<span data-ttu-id="309ed-113">Конечно, у вашего приложения могут быть особые требования, для которых модель IaaS подходит лучше.</span><span class="sxs-lookup"><span data-stu-id="309ed-113">Of course, your application may have specific requirements that make an IaaS approach more suitable.</span></span> <span data-ttu-id="309ed-114">Но даже для приложений на основе службы IaaS можно найти возможность естественного применения управляемых служб.</span><span class="sxs-lookup"><span data-stu-id="309ed-114">However, even if your application is based on IaaS, look for places where it may be natural to incorporate managed services.</span></span> <span data-ttu-id="309ed-115">Например, служб кэширования, управления очередями и хранилища данных.</span><span class="sxs-lookup"><span data-stu-id="309ed-115">These include cache, queues, and data storage.</span></span>

| <span data-ttu-id="309ed-116">Вместо службы...</span><span class="sxs-lookup"><span data-stu-id="309ed-116">Instead of running...</span></span> | <span data-ttu-id="309ed-117">рекомендуем использовать...</span><span class="sxs-lookup"><span data-stu-id="309ed-117">Consider using...</span></span> |
|-----------------------|-------------|
| <span data-ttu-id="309ed-118">Active Directory</span><span class="sxs-lookup"><span data-stu-id="309ed-118">Active Directory</span></span> | <span data-ttu-id="309ed-119">Доменные службы Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="309ed-119">Azure Active Directory Domain Services</span></span> |
| <span data-ttu-id="309ed-120">Elasticsearch</span><span class="sxs-lookup"><span data-stu-id="309ed-120">Elasticsearch</span></span> | <span data-ttu-id="309ed-121">Поиск Azure</span><span class="sxs-lookup"><span data-stu-id="309ed-121">Azure Search</span></span> |
| <span data-ttu-id="309ed-122">Hadoop</span><span class="sxs-lookup"><span data-stu-id="309ed-122">Hadoop</span></span> | <span data-ttu-id="309ed-123">HDInsight</span><span class="sxs-lookup"><span data-stu-id="309ed-123">HDInsight</span></span> |
| <span data-ttu-id="309ed-124">IIS</span><span class="sxs-lookup"><span data-stu-id="309ed-124">IIS</span></span> | <span data-ttu-id="309ed-125">Служба приложений</span><span class="sxs-lookup"><span data-stu-id="309ed-125">App Service</span></span> |
| <span data-ttu-id="309ed-126">MongoDB</span><span class="sxs-lookup"><span data-stu-id="309ed-126">MongoDB</span></span> | <span data-ttu-id="309ed-127">База данных Cosmos</span><span class="sxs-lookup"><span data-stu-id="309ed-127">Cosmos DB</span></span> |
| <span data-ttu-id="309ed-128">Redis</span><span class="sxs-lookup"><span data-stu-id="309ed-128">Redis</span></span> | <span data-ttu-id="309ed-129">кэш Azure Redis</span><span class="sxs-lookup"><span data-stu-id="309ed-129">Azure Redis Cache</span></span> |
| <span data-ttu-id="309ed-130">SQL Server;</span><span class="sxs-lookup"><span data-stu-id="309ed-130">SQL Server</span></span> | <span data-ttu-id="309ed-131">Базы данных SQL Azure</span><span class="sxs-lookup"><span data-stu-id="309ed-131">Azure SQL Database</span></span> |


