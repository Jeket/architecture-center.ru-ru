---
title: Реализация сети периметра между Azure и Интернетом
description: Способы реализации защищенной гибридной сетевой архитектуры с доступом к Интернету в Azure.
author: telmosampaio
ms.date: 10/22/2018
pnp.series.title: Network DMZ
pnp.series.next: nva-ha
pnp.series.prev: secure-vnet-hybrid
cardTitle: DMZ between Azure and the Internet
ms.openlocfilehash: d12c76a08f028d54784de330c62311f294802cb9
ms.sourcegitcommit: dbbf914757b03cdee7a274204f9579fa63d7eed2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916337"
---
# <a name="dmz-between-azure-and-the-internet"></a><span data-ttu-id="18bcc-103">Сеть периметра между Azure и Интернетом</span><span class="sxs-lookup"><span data-stu-id="18bcc-103">DMZ between Azure and the Internet</span></span>

<span data-ttu-id="18bcc-104">Эта эталонная архитектура представляет собой безопасную гибридную сеть, которая расширяет локальную сеть в Azure, а также принимает интернет-трафик.</span><span class="sxs-lookup"><span data-stu-id="18bcc-104">This reference architecture shows a secure hybrid network that extends an on-premises network to Azure and also accepts Internet traffic.</span></span> [<span data-ttu-id="18bcc-105">**Разверните это решение**.</span><span class="sxs-lookup"><span data-stu-id="18bcc-105">**Deploy this solution**.</span></span>](#deploy-the-solution)

<span data-ttu-id="18bcc-106">[![0]][0]</span><span class="sxs-lookup"><span data-stu-id="18bcc-106">[![0]][0]</span></span> 

<span data-ttu-id="18bcc-107">*Скачайте [файл Visio][visio-download] этой архитектуры.*</span><span class="sxs-lookup"><span data-stu-id="18bcc-107">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="18bcc-108">Эта эталонная архитектура расширяет архитектуру, описанную в [DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture] (Сеть периметра между Azure и локальным центром данных).</span><span class="sxs-lookup"><span data-stu-id="18bcc-108">This reference architecture extends the architecture described in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture].</span></span> <span data-ttu-id="18bcc-109">Она добавляет открытую сеть периметра, обрабатывающую интернет-трафик, в дополнение к частной сети периметра, которая обрабатывает трафик из локальной сети.</span><span class="sxs-lookup"><span data-stu-id="18bcc-109">It adds a public DMZ that handles Internet traffic, in addition to the private DMZ that handles traffic from the on-premises network</span></span> 

<span data-ttu-id="18bcc-110">Типичные способы использования этой архитектуры:</span><span class="sxs-lookup"><span data-stu-id="18bcc-110">Typical uses for this architecture include:</span></span>

* <span data-ttu-id="18bcc-111">Гибридные приложения, в которых рабочие нагрузки выполняются частично локально и частично в Azure.</span><span class="sxs-lookup"><span data-stu-id="18bcc-111">Hybrid applications where workloads run partly on-premises and partly in Azure.</span></span>
* <span data-ttu-id="18bcc-112">Инфраструктура Azure, которая направляет входящий трафик из локальной среды и Интернета.</span><span class="sxs-lookup"><span data-stu-id="18bcc-112">Azure infrastructure that routes incoming traffic from on-premises and the Internet.</span></span>

## <a name="architecture"></a><span data-ttu-id="18bcc-113">Архитектура</span><span class="sxs-lookup"><span data-stu-id="18bcc-113">Architecture</span></span>

<span data-ttu-id="18bcc-114">Архитектура состоит из следующих компонентов:</span><span class="sxs-lookup"><span data-stu-id="18bcc-114">The architecture consists of the following components.</span></span>

* <span data-ttu-id="18bcc-115">**Общедоступный IP-адрес (PIP).**</span><span class="sxs-lookup"><span data-stu-id="18bcc-115">**Public IP address (PIP)**.</span></span> <span data-ttu-id="18bcc-116">IP-адрес общедоступной конечной точки.</span><span class="sxs-lookup"><span data-stu-id="18bcc-116">The IP address of the public endpoint.</span></span> <span data-ttu-id="18bcc-117">Внешние пользователи, подключенные к Интернету, могут получить доступ к системе через этот адрес.</span><span class="sxs-lookup"><span data-stu-id="18bcc-117">External users connected to the Internet can access the system through this address.</span></span>
* <span data-ttu-id="18bcc-118">**Виртуальный сетевой модуль (NVA).**</span><span class="sxs-lookup"><span data-stu-id="18bcc-118">**Network virtual appliance (NVA)**.</span></span> <span data-ttu-id="18bcc-119">Эта архитектура включает в себя отдельный пул NVA для исходящего трафика в Интернете.</span><span class="sxs-lookup"><span data-stu-id="18bcc-119">This architecture includes a separate pool of NVAs for traffic originating on the Internet.</span></span>
* <span data-ttu-id="18bcc-120">**Azure Load Balancer.**</span><span class="sxs-lookup"><span data-stu-id="18bcc-120">**Azure load balancer**.</span></span> <span data-ttu-id="18bcc-121">Все входящие интернет-запросы проходят через подсистему балансировки нагрузки и распространяются в NVA в общедоступной сети периметра.</span><span class="sxs-lookup"><span data-stu-id="18bcc-121">All incoming requests from the Internet pass through the load balancer and are distributed to the NVAs in the public DMZ.</span></span>
* <span data-ttu-id="18bcc-122">**Подсеть входящего трафика общедоступной сети периметра.**</span><span class="sxs-lookup"><span data-stu-id="18bcc-122">**Public DMZ inbound subnet**.</span></span> <span data-ttu-id="18bcc-123">Эта подсеть принимает запросы от подсистемы балансировки нагрузки Azure.</span><span class="sxs-lookup"><span data-stu-id="18bcc-123">This subnet accepts requests from the Azure load balancer.</span></span> <span data-ttu-id="18bcc-124">Входящие запросы передаются одному из NVA в общедоступной сети периметра.</span><span class="sxs-lookup"><span data-stu-id="18bcc-124">Incoming requests are passed to one of the NVAs in the public DMZ.</span></span>
* <span data-ttu-id="18bcc-125">**Подсеть исходящего трафика общедоступной сети периметра.**</span><span class="sxs-lookup"><span data-stu-id="18bcc-125">**Public DMZ outbound subnet**.</span></span> <span data-ttu-id="18bcc-126">Запросы, утвержденные виртуальным сетевым модулем, проходят через подсеть внутренней подсистемы балансировки нагрузки для веб-уровня.</span><span class="sxs-lookup"><span data-stu-id="18bcc-126">Requests that are approved by the NVA pass through this subnet to the internal load balancer for the web tier.</span></span>

## <a name="recommendations"></a><span data-ttu-id="18bcc-127">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="18bcc-127">Recommendations</span></span>

<span data-ttu-id="18bcc-128">Следующие рекомендации применимы для большинства ситуаций.</span><span class="sxs-lookup"><span data-stu-id="18bcc-128">The following recommendations apply for most scenarios.</span></span> <span data-ttu-id="18bcc-129">Следуйте этим рекомендациям, если они не противоречат особым требованиям для вашего случая.</span><span class="sxs-lookup"><span data-stu-id="18bcc-129">Follow these recommendations unless you have a specific requirement that overrides them.</span></span> 

### <a name="nva-recommendations"></a><span data-ttu-id="18bcc-130">Рекомендации по использованию виртуального сетевого модуля</span><span class="sxs-lookup"><span data-stu-id="18bcc-130">NVA recommendations</span></span>

<span data-ttu-id="18bcc-131">Используйте разные наборы NVA для трафика в Интернете и трафика, создаваемого локальными системами.</span><span class="sxs-lookup"><span data-stu-id="18bcc-131">Use one set of NVAs for traffic originating on the Internet, and another for traffic originating on-premises.</span></span> <span data-ttu-id="18bcc-132">Использование только одного набора NVA в обоих случаях представляет угрозу безопасности, так как между двумя наборами сетевого трафика отсутствует периметр безопасности.</span><span class="sxs-lookup"><span data-stu-id="18bcc-132">Using only one set of NVAs for both is a security risk, because it provides no security perimeter between the two sets of network traffic.</span></span> <span data-ttu-id="18bcc-133">Использование отдельных NVA упрощает проверку правил безопасности, а также позволяет понять, какие правила соответствуют определенным входящим сетевым запросам.</span><span class="sxs-lookup"><span data-stu-id="18bcc-133">Using separate NVAs reduces the complexity of checking security rules, and makes it clear which rules correspond to each incoming network request.</span></span> <span data-ttu-id="18bcc-134">Один набор NVA реализует правила для интернет-трафика, а другой — правила для локального трафика.</span><span class="sxs-lookup"><span data-stu-id="18bcc-134">One set of NVAs implements rules for Internet traffic only, while another set of NVAs implement rules for on-premises traffic only.</span></span>

<span data-ttu-id="18bcc-135">Включите NVA уровня 7, чтобы завершать соединения приложений на уровне NVA и обеспечить поддержку совместимости с серверными уровнями.</span><span class="sxs-lookup"><span data-stu-id="18bcc-135">Include a layer-7 NVA to terminate application connections at the NVA level and maintain compatibility with the backend tiers.</span></span> <span data-ttu-id="18bcc-136">Это гарантирует симметричное подключение, при котором ответный трафик из серверных уровней возвращается через NVA.</span><span class="sxs-lookup"><span data-stu-id="18bcc-136">This guarantees symmetric connectivity where response traffic from the backend tiers returns through the NVA.</span></span>  

### <a name="public-load-balancer-recommendations"></a><span data-ttu-id="18bcc-137">Рекомендации по использованию общедоступной подсистемы балансировки нагрузки</span><span class="sxs-lookup"><span data-stu-id="18bcc-137">Public load balancer recommendations</span></span>

<span data-ttu-id="18bcc-138">Для обеспечения масштабируемости и доступности развертывайте общедоступную сеть периметра NVA в [группе доступности][availability-set], а также распределяйте интернет-запросы по NVA в группе доступности с помощью [балансировщика нагрузки для Интернета][load-balancer].</span><span class="sxs-lookup"><span data-stu-id="18bcc-138">For scalability and availability, deploy the public DMZ NVAs in an [availability set][availability-set] and use an [Internet facing load balancer][load-balancer] to distribute Internet requests across the NVAs in the availability set.</span></span>  

<span data-ttu-id="18bcc-139">Настройте подсистему балансировки нагрузки для принятия запросов только на порты, необходимые для интернет-трафика.</span><span class="sxs-lookup"><span data-stu-id="18bcc-139">Configure the load balancer to accept requests only on the ports necessary for Internet traffic.</span></span> <span data-ttu-id="18bcc-140">Например, ограничьте входящие HTTP-запросы на порт 80 и 443.</span><span class="sxs-lookup"><span data-stu-id="18bcc-140">For example, restrict inbound HTTP requests to port 80 and inbound HTTPS requests to port 443.</span></span>

## <a name="scalability-considerations"></a><span data-ttu-id="18bcc-141">Вопросы масштабируемости</span><span class="sxs-lookup"><span data-stu-id="18bcc-141">Scalability considerations</span></span>

<span data-ttu-id="18bcc-142">Даже если первоначально для архитектуры требуется один NVA в общедоступной сети периметра, мы рекомендуем сначала разместить подсистему балансировки нагрузки перед общедоступной сетью периметра.</span><span class="sxs-lookup"><span data-stu-id="18bcc-142">Even if your architecture initially requires a single NVA in the public DMZ, we recommend putting a load balancer in front of the public DMZ from the beginning.</span></span> <span data-ttu-id="18bcc-143">Это поможет упростить выполнение масштабирования для нескольких NVA в будущем.</span><span class="sxs-lookup"><span data-stu-id="18bcc-143">That will make it easier to scale to multiple NVAs in the future, if needed.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="18bcc-144">Вопросы доступности</span><span class="sxs-lookup"><span data-stu-id="18bcc-144">Availability considerations</span></span>

<span data-ttu-id="18bcc-145">Балансировщик нагрузки для Интернета требует, чтобы каждый NVA в подсети входящего трафика общедоступной сети периметра прошел [проверку работоспособности][lb-probe].</span><span class="sxs-lookup"><span data-stu-id="18bcc-145">The Internet facing load balancer requires each NVA in the public DMZ inbound subnet to implement a [health probe][lb-probe].</span></span> <span data-ttu-id="18bcc-146">Проверка работоспособности, которая не отвечает на конечной точке, считается недоступной, из-за чего подсистема балансировки нагрузки будет направлять запросы к другим NVA в той же группе доступности.</span><span class="sxs-lookup"><span data-stu-id="18bcc-146">A health probe that fails to respond on this endpoint is considered to be unavailable, and the load balancer will direct requests to other NVAs in the same availability set.</span></span> <span data-ttu-id="18bcc-147">Обратите внимание, что если все NVA не отвечают, приложение не будет работать. Поэтому важно, чтобы были настроены уведомления на DevOps, когда число исправных экземпляров NVA падает ниже заданного порогового значения.</span><span class="sxs-lookup"><span data-stu-id="18bcc-147">Note that if all NVAs fail to respond, your application will fail, so it's important to have monitoring configured to alert DevOps when the number of healthy NVA instances falls below a defined threshold.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="18bcc-148">Вопросы управляемости</span><span class="sxs-lookup"><span data-stu-id="18bcc-148">Manageability considerations</span></span>

<span data-ttu-id="18bcc-149">Все операции мониторинга и управления для NVA в общедоступной сети периметра должны выполняться в подсети управления с помощью jumpbox.</span><span class="sxs-lookup"><span data-stu-id="18bcc-149">All monitoring and management for the NVAs in the public DMZ should be performed by the jumpbox in the management subnet.</span></span> <span data-ttu-id="18bcc-150">Определите один сетевой маршрут из локальной сети через шлюз к jumpbox, чтобы ограничить доступ, как было сказано в статье [DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture] (Сеть периметра между Azure и локальным центром данных).</span><span class="sxs-lookup"><span data-stu-id="18bcc-150">As discussed in [Implementing a DMZ between Azure and your on-premises datacenter][implementing-a-secure-hybrid-network-architecture], define a single network route from the on-premises network through the gateway to the jumpbox, in order to restrict access.</span></span>

<span data-ttu-id="18bcc-151">При разрыве подключения шлюза из локальной сети в Azure можно по-прежнему подключиться к jumpbox путем развертывания общедоступного IP-адреса, добавив его в jumpbox и выполнив вход из Интернета.</span><span class="sxs-lookup"><span data-stu-id="18bcc-151">If gateway connectivity from your on-premises network to Azure is down, you can still reach the jumpbox by deploying a public IP address, adding it to the jumpbox, and logging in from the Internet.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="18bcc-152">Вопросы безопасности</span><span class="sxs-lookup"><span data-stu-id="18bcc-152">Security considerations</span></span>

<span data-ttu-id="18bcc-153">Эта эталонная архитектура реализует несколько уровней безопасности:</span><span class="sxs-lookup"><span data-stu-id="18bcc-153">This reference architecture implements multiple levels of security:</span></span>

* <span data-ttu-id="18bcc-154">Балансировщик нагрузки для Интернета направляет запросы виртуальному сетевому модулю в подсеть входящего трафика общедоступной сети периметра и на порты, необходимые для приложения.</span><span class="sxs-lookup"><span data-stu-id="18bcc-154">The Internet facing load balancer directs requests to the NVAs in the inbound public DMZ subnet, and only on the ports necessary for the application.</span></span>
* <span data-ttu-id="18bcc-155">Правила NSG для входящего и исходящего трафика подсетей общедоступной сети периметра предотвращают компрометацию NVA путем блокировки запросов, которые выходят за пределы правил NSG.</span><span class="sxs-lookup"><span data-stu-id="18bcc-155">The NSG rules for the inbound and outbound public DMZ subnets prevent the NVAs from being compromised, by blocking requests that fall outside of the NSG rules.</span></span>
* <span data-ttu-id="18bcc-156">Конфигурация маршрутизации NAT для NVA направляет входящие запросы к порту 80 и 443 в подсистему балансировки нагрузки веб-уровня. Однако запросы на остальных портах игнорируются.</span><span class="sxs-lookup"><span data-stu-id="18bcc-156">The NAT routing configuration for the NVAs directs incoming requests on port 80 and port 443 to the web tier load balancer, but ignores requests on all other ports.</span></span>

<span data-ttu-id="18bcc-157">Необходимо записывать входящие запросы на всех портах.</span><span class="sxs-lookup"><span data-stu-id="18bcc-157">You should log all incoming requests on all ports.</span></span> <span data-ttu-id="18bcc-158">Регулярно выполняйте аудит журналов, обращая внимание на запросы, которые выходят за пределы необходимых параметров, так как их появление может свидетельствовать о попытках вторжения.</span><span class="sxs-lookup"><span data-stu-id="18bcc-158">Regularly audit the logs, paying attention to requests that fall outside of expected parameters, as these may indicate intrusion attempts.</span></span>


## <a name="deploy-the-solution"></a><span data-ttu-id="18bcc-159">Развертывание решения</span><span class="sxs-lookup"><span data-stu-id="18bcc-159">Deploy the solution</span></span>

<span data-ttu-id="18bcc-160">В репозитории [GitHub][github-folder] есть шаблон развертывания эталонной архитектуры, для которого реализованы эти рекомендации.</span><span class="sxs-lookup"><span data-stu-id="18bcc-160">A deployment for a reference architecture that implements these recommendations is available on [GitHub][github-folder].</span></span> 

### <a name="prerequisites"></a><span data-ttu-id="18bcc-161">Предварительные требования</span><span class="sxs-lookup"><span data-stu-id="18bcc-161">Prerequisites</span></span>

[!INCLUDE [ref-arch-prerequisites.md](../../../includes/ref-arch-prerequisites.md)]

### <a name="deploy-resources"></a><span data-ttu-id="18bcc-162">Развертывание ресурсов</span><span class="sxs-lookup"><span data-stu-id="18bcc-162">Deploy resources</span></span>

1. <span data-ttu-id="18bcc-163">Перейдите в папку `/dmz/secure-vnet-hybrid` в репозитории эталонных архитектур на сайте GitHub.</span><span class="sxs-lookup"><span data-stu-id="18bcc-163">Navigate to the `/dmz/secure-vnet-hybrid` folder of the reference architectures GitHub repository.</span></span>

2. <span data-ttu-id="18bcc-164">Выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="18bcc-164">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p onprem.json --deploy
    ```

3. <span data-ttu-id="18bcc-165">Выполните следующую команду:</span><span class="sxs-lookup"><span data-stu-id="18bcc-165">Run the following command:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource_group_name> -l <region> -p secure-vnet-hybrid.json --deploy
    ```

### <a name="connect-the-on-premises-and-azure-gateways"></a><span data-ttu-id="18bcc-166">Подключение локальных шлюзов и шлюзов Azure</span><span class="sxs-lookup"><span data-stu-id="18bcc-166">Connect the on-premises and Azure gateways</span></span>

<span data-ttu-id="18bcc-167">На этом этапе вы подключите два шлюза локальной сети.</span><span class="sxs-lookup"><span data-stu-id="18bcc-167">In this step, you will connect the two local network gateways.</span></span>

1. <span data-ttu-id="18bcc-168">На портале Azure перейдите к созданной группе ресурсов.</span><span class="sxs-lookup"><span data-stu-id="18bcc-168">In the Azure Portal, navigate to the resource group that you created.</span></span> 

2. <span data-ttu-id="18bcc-169">Найдите ресурс с именем `ra-vpn-vgw-pip` и скопируйте IP-адрес, который отображается в колонке **Обзор**.</span><span class="sxs-lookup"><span data-stu-id="18bcc-169">Find the resource named `ra-vpn-vgw-pip` and copy the IP address shown in the **Overview** blade.</span></span>

3. <span data-ttu-id="18bcc-170">Найдите ресурс с именем `onprem-vpn-lgw`.</span><span class="sxs-lookup"><span data-stu-id="18bcc-170">Find the resource named `onprem-vpn-lgw`.</span></span>

4. <span data-ttu-id="18bcc-171">Перейдите к колонке **Конфигурация**.</span><span class="sxs-lookup"><span data-stu-id="18bcc-171">Click the **Configuration** blade.</span></span> <span data-ttu-id="18bcc-172">В поле **IP-адрес** вставьте IP-адрес из шага 2.</span><span class="sxs-lookup"><span data-stu-id="18bcc-172">Under **IP address**, paste in the IP address from step 2.</span></span>

    ![](./images/local-net-gw.png)

5. <span data-ttu-id="18bcc-173">Нажмите кнопку **Сохранить** и дождитесь завершения операции.</span><span class="sxs-lookup"><span data-stu-id="18bcc-173">Click **Save** and wait for the operation to complete.</span></span> <span data-ttu-id="18bcc-174">Это может занять около 5 минут.</span><span class="sxs-lookup"><span data-stu-id="18bcc-174">It can take about 5 minutes.</span></span>

6. <span data-ttu-id="18bcc-175">Найдите ресурс с именем `onprem-vpn-gateway1-pip`.</span><span class="sxs-lookup"><span data-stu-id="18bcc-175">Find the resource named `onprem-vpn-gateway1-pip`.</span></span> <span data-ttu-id="18bcc-176">Скопируйте IP-адрес, который отображается в колонке **Обзор**.</span><span class="sxs-lookup"><span data-stu-id="18bcc-176">Copy the IP address shown in the **Overview** blade.</span></span>

7. <span data-ttu-id="18bcc-177">Найдите ресурс с именем `ra-vpn-lgw`.</span><span class="sxs-lookup"><span data-stu-id="18bcc-177">Find the resource named `ra-vpn-lgw`.</span></span> 

8. <span data-ttu-id="18bcc-178">Перейдите к колонке **Конфигурация**.</span><span class="sxs-lookup"><span data-stu-id="18bcc-178">Click the **Configuration** blade.</span></span> <span data-ttu-id="18bcc-179">В поле **IP-адрес** вставьте IP-адрес из шага 6.</span><span class="sxs-lookup"><span data-stu-id="18bcc-179">Under **IP address**, paste in the IP address from step 6.</span></span>

9. <span data-ttu-id="18bcc-180">Нажмите кнопку **Сохранить** и дождитесь завершения операции.</span><span class="sxs-lookup"><span data-stu-id="18bcc-180">Click **Save** and wait for the operation to complete.</span></span>

10. <span data-ttu-id="18bcc-181">Чтобы проверить подключение, перейдите к колонке **Подключения** для каждого шлюза.</span><span class="sxs-lookup"><span data-stu-id="18bcc-181">To verify the connection, go to the **Connections** blade for each gateway.</span></span> <span data-ttu-id="18bcc-182">Должно отображаться состояние **Подключено**.</span><span class="sxs-lookup"><span data-stu-id="18bcc-182">The status should be **Connected**.</span></span>

### <a name="verify-that-network-traffic-reaches-the-web-tier"></a><span data-ttu-id="18bcc-183">Проверка того, поступает ли трафик на веб-уровень</span><span class="sxs-lookup"><span data-stu-id="18bcc-183">Verify that network traffic reaches the web tier</span></span>

1. <span data-ttu-id="18bcc-184">На портале Azure перейдите к созданной группе ресурсов.</span><span class="sxs-lookup"><span data-stu-id="18bcc-184">In the Azure Portal, navigate to the resource group that you created.</span></span> 

2. <span data-ttu-id="18bcc-185">Найдите ресурс с именем `pub-dmz-lb`. Это подсистема балансировки нагрузки, размещенная перед общедоступной промежуточной подсетью.</span><span class="sxs-lookup"><span data-stu-id="18bcc-185">Find the resource named `pub-dmz-lb`, which is the load balancer in front of the public DMZ.</span></span> 

3. <span data-ttu-id="18bcc-186">Скопируйте общедоступный IP-адрес в колонке **Обзор** и откройте его в веб-браузере.</span><span class="sxs-lookup"><span data-stu-id="18bcc-186">Copy the public IP addess from the **Overview** blade and open this address in a web browser.</span></span> <span data-ttu-id="18bcc-187">Вы увидите домашнюю страницу сервера Apache2 по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="18bcc-187">You should see the default Apache2 server home page.</span></span>

4. <span data-ttu-id="18bcc-188">Найдите ресурс с именем `int-dmz-lb`. Это подсистема балансировки нагрузки, размещенная перед частной промежуточной подсетью.</span><span class="sxs-lookup"><span data-stu-id="18bcc-188">Find the resource named `int-dmz-lb`, which is the load balancer in front of the private DMZ.</span></span> <span data-ttu-id="18bcc-189">Скопируйте частный IP-адрес в колонке **Обзор**.</span><span class="sxs-lookup"><span data-stu-id="18bcc-189">Copy the private IP address from the **Overview** blade.</span></span>

5. <span data-ttu-id="18bcc-190">Найдите виртуальную машину с именем `jb-vm1`.</span><span class="sxs-lookup"><span data-stu-id="18bcc-190">Find the VM named `jb-vm1`.</span></span> <span data-ttu-id="18bcc-191">Нажмите кнопку **Подключиться** и используйте удаленный рабочий стол, чтобы подключиться к виртуальной машине.</span><span class="sxs-lookup"><span data-stu-id="18bcc-191">Click **Connect** and use Remote Desktop to connect to the VM.</span></span> <span data-ttu-id="18bcc-192">Имя пользователя и пароль указываются в файле onprem.json.</span><span class="sxs-lookup"><span data-stu-id="18bcc-192">The user name and password are specified in the onprem.json file.</span></span>

6. <span data-ttu-id="18bcc-193">В сеансе удаленного рабочего стола откройте браузер и перейдите по IP-адресу из шага 4.</span><span class="sxs-lookup"><span data-stu-id="18bcc-193">From the Remote Desktop Session, open a web browser and navigate to the IP address from step 4.</span></span> <span data-ttu-id="18bcc-194">Вы увидите домашнюю страницу сервера Apache2 по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="18bcc-194">You should see the default Apache2 server home page.</span></span>

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/dmz/secure-vnet-dmz

[implementing-a-secure-hybrid-network-architecture]: ./secure-vnet-hybrid.md
[iptables]: https://help.ubuntu.com/community/IptablesHowTo
[lb-probe]: /azure/load-balancer/load-balancer-custom-probe-overview
[load-balancer]: /azure/load-balancer/load-balancer-Internet-overview
[network-security-group]: /azure/virtual-network/virtual-networks-nsg

[visio-download]: https://archcenter.blob.core.windows.net/cdn/dmz-reference-architectures.vsdx


[0]: ./images/dmz-public.png "Архитектура защищенной гибридной сети"
