---
title: Принципы разработки для приложений Azure
description: Принципы разработки для приложений Azure
author: MikeWasson
ms.openlocfilehash: 462896098c668c0775464ca498925266cd73c6e1
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206810"
---
# <a name="design-principles-for-azure-applications"></a><span data-ttu-id="9e890-103">Принципы разработки для приложений Azure</span><span class="sxs-lookup"><span data-stu-id="9e890-103">Design principles for Azure applications</span></span>

<span data-ttu-id="9e890-104">Следуйте приведенным здесь принципам проектирования, чтобы сделать приложение более масштабируемым, отказоустойчивым и управляемым.</span><span class="sxs-lookup"><span data-stu-id="9e890-104">Follow these design principles to make your application more scalable, resilient, and manageable.</span></span> 

<span data-ttu-id="9e890-105">**[Разработка с учетом возможности самовосстановления](self-healing.md)**.</span><span class="sxs-lookup"><span data-stu-id="9e890-105">**[Design for self healing](self-healing.md)**.</span></span> <span data-ttu-id="9e890-106">В распределенной системе иногда происходят сбои.</span><span class="sxs-lookup"><span data-stu-id="9e890-106">In a distributed system, failures happen.</span></span> <span data-ttu-id="9e890-107">Реализуйте в приложении возможность самовосстановления при сбоях.</span><span class="sxs-lookup"><span data-stu-id="9e890-107">Design your application to be self healing when failures occur.</span></span>

<span data-ttu-id="9e890-108">**[Обеспечение избыточности всех компонентов](redundancy.md)**.</span><span class="sxs-lookup"><span data-stu-id="9e890-108">**[Make all things redundant](redundancy.md)**.</span></span> <span data-ttu-id="9e890-109">Обеспечьте избыточность приложения, чтобы избежать наличия единых точек отказа.</span><span class="sxs-lookup"><span data-stu-id="9e890-109">Build redundancy into your application, to avoid having single points of failure.</span></span>
 
<span data-ttu-id="9e890-110">**[Уменьшение координации](minimize-coordination.md)**.</span><span class="sxs-lookup"><span data-stu-id="9e890-110">**[Minimize coordination](minimize-coordination.md)**.</span></span> <span data-ttu-id="9e890-111">Сведите к минимуму координацию между службами приложений, чтобы обеспечить масштабируемость.</span><span class="sxs-lookup"><span data-stu-id="9e890-111">Minimize coordination between application services to achieve scalability.</span></span>
 
<span data-ttu-id="9e890-112">**[Разработка для обеспечения горизонтального масштабирования](scale-out.md)**. Разработайте приложение таким образом, чтобы оно могло выполнять горизонтальное масштабирование, добавляя или удаляя новые экземпляры (в зависимости требований).</span><span class="sxs-lookup"><span data-stu-id="9e890-112">**[Design to scale out](scale-out.md)**. Design your application so that it can scale horizontally, adding or removing new instances as demand requires.</span></span>

<span data-ttu-id="9e890-113">**[Секционирование для обхода ограничений](partition.md)**.</span><span class="sxs-lookup"><span data-stu-id="9e890-113">**[Partition around limits](partition.md)**.</span></span> <span data-ttu-id="9e890-114">Применяйте секционирование для обхода ограничений базы данных, сети и вычислений.</span><span class="sxs-lookup"><span data-stu-id="9e890-114">Use partitioning to work around database, network, and compute limits.</span></span>

<span data-ttu-id="9e890-115">**[Внедрение необходимых средств для работы](design-for-operations.md)**.</span><span class="sxs-lookup"><span data-stu-id="9e890-115">**[Design for operations](design-for-operations.md)**.</span></span> <span data-ttu-id="9e890-116">Разработайте приложение таким образом, чтобы у рабочей группы были необходимые средства.</span><span class="sxs-lookup"><span data-stu-id="9e890-116">Design your application so that the operations team has the tools they need.</span></span>

<span data-ttu-id="9e890-117">**[Использование управляемых служб](managed-services.md)**.</span><span class="sxs-lookup"><span data-stu-id="9e890-117">**[Use managed services](managed-services.md)**.</span></span> <span data-ttu-id="9e890-118">По возможности используйте платформу как услугу (PaaS) вместо инфраструктуры как услуги (IaaS).</span><span class="sxs-lookup"><span data-stu-id="9e890-118">When possible, use platform as a service (PaaS) rather than infrastructure as a service (IaaS).</span></span>

<span data-ttu-id="9e890-119">**[Использование подходящего хранилища данных для задания](use-the-best-data-store.md)**.</span><span class="sxs-lookup"><span data-stu-id="9e890-119">**[Use the best data store for the job](use-the-best-data-store.md)**.</span></span> <span data-ttu-id="9e890-120">Выберите подходящую технологию хранения для используемых данных и способ ее использования.</span><span class="sxs-lookup"><span data-stu-id="9e890-120">Pick the storage technology that is the best fit for your data and how it will be used.</span></span> 
 
<span data-ttu-id="9e890-121">**[Разработка с учетом будущих изменений](design-for-evolution.md)**.</span><span class="sxs-lookup"><span data-stu-id="9e890-121">**[Design for evolution](design-for-evolution.md)**.</span></span> <span data-ttu-id="9e890-122">Все успешные приложения изменяются со временем.</span><span class="sxs-lookup"><span data-stu-id="9e890-122">All successful applications change over time.</span></span> <span data-ttu-id="9e890-123">Эволюционное проектирование является ключом для непрерывных инноваций.</span><span class="sxs-lookup"><span data-stu-id="9e890-123">An evolutionary design is key for continuous innovation.</span></span>

<span data-ttu-id="9e890-124">**[Разработка с учетом бизнес-потребностей](build-for-business.md)**.</span><span class="sxs-lookup"><span data-stu-id="9e890-124">**[Build for the needs of business](build-for-business.md)**.</span></span> <span data-ttu-id="9e890-125">Каждое решение по разработке должно соответствовать бизнес-требованиям.</span><span class="sxs-lookup"><span data-stu-id="9e890-125">Every design decision must be justified by a business requirement.</span></span>

