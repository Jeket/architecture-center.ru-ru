---
title: Разработка конвейера CI/CD с использованием Azure DevOps
titleSuffix: Azure Example Scenarios
description: Создание и выпуск приложения .NET в службе "Веб-приложения Azure" с помощью Azure DevOps.
author: christianreddington
ms.date: 12/06/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
ms.custom: fasttrack, seodec18
social_image_url: /azure/architecture/example-scenario/apps/media/architecture-devops-dotnet-webapp.svg
ms.openlocfilehash: f999b2ffdf234161f668887d5b2327ecf50f0e55
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/18/2019
ms.locfileid: "59740414"
---
# <a name="design-a-cicd-pipeline-using-azure-devops"></a><span data-ttu-id="a70c5-103">Разработка конвейера CI/CD с использованием Azure DevOps</span><span class="sxs-lookup"><span data-stu-id="a70c5-103">Design a CI/CD pipeline using Azure DevOps</span></span>

<span data-ttu-id="a70c5-104">В этом сценарии предоставляется руководство по архитектуре и разработке для создания конвейера непрерывной интеграции (CI) и непрерывного развертывания (CD).</span><span class="sxs-lookup"><span data-stu-id="a70c5-104">This scenario provides architecture and design guidance for building a continuous integration (CI) and continuous deployment (CD) pipeline.</span></span> <span data-ttu-id="a70c5-105">В этом примере конвейер CI/CD развертывает двухуровневое веб-приложение .NET в Службе приложений Azure.</span><span class="sxs-lookup"><span data-stu-id="a70c5-105">In this example, the CI/CD pipeline deploys a two-tier .NET web application to the Azure App Service.</span></span>

<span data-ttu-id="a70c5-106">Переход на современные процессы CI/CD предоставляет множество преимуществ для создания, развертывания, тестирования и мониторинга приложения.</span><span class="sxs-lookup"><span data-stu-id="a70c5-106">Migrating to modern CI/CD processes provides many benefits for application builds, deployments, testing, and monitoring.</span></span> <span data-ttu-id="a70c5-107">С помощью Azure DevOps вместе с такими службами, как Служба приложений, организации могут уделить больше внимания разработке приложений, а не управлению поддержкой инфраструктуры.</span><span class="sxs-lookup"><span data-stu-id="a70c5-107">By utilizing Azure DevOps along with other services such as App Service, organizations can focus on the development of their apps rather than the management of the supporting infrastructure.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="a70c5-108">Варианты соответствующего использования</span><span class="sxs-lookup"><span data-stu-id="a70c5-108">Relevant use cases</span></span>

<span data-ttu-id="a70c5-109">Рассмотрите использование Azure DevOps и процессов CI/CD для:</span><span class="sxs-lookup"><span data-stu-id="a70c5-109">Consider Azure DevOps and CI/CD processes for:</span></span>

- <span data-ttu-id="a70c5-110">Ускорение разработки приложений и жизненных циклов развертывания.</span><span class="sxs-lookup"><span data-stu-id="a70c5-110">Accelerating application development and development life cycles</span></span>
- <span data-ttu-id="a70c5-111">Внедрение механизмов проверки качества и согласованности в автоматизированный процесс сборки и выпуска.</span><span class="sxs-lookup"><span data-stu-id="a70c5-111">Building quality and consistency into an automated build and release process</span></span>
- <span data-ttu-id="a70c5-112">повышения стабильности и времени доступности приложения.</span><span class="sxs-lookup"><span data-stu-id="a70c5-112">Increasing application stability and uptime</span></span>

## <a name="architecture"></a><span data-ttu-id="a70c5-113">Архитектура</span><span class="sxs-lookup"><span data-stu-id="a70c5-113">Architecture</span></span>

![Схема архитектуры компонентов Azure, участвующих в сценарии DevOps, с использованием Azure DevOps и Службы приложений Azure][architecture]

<span data-ttu-id="a70c5-115">Данные передаются в сценарии следующим образом:</span><span class="sxs-lookup"><span data-stu-id="a70c5-115">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="a70c5-116">Разработчик изменяет исходный код приложения.</span><span class="sxs-lookup"><span data-stu-id="a70c5-116">A developer changes application source code.</span></span>
2. <span data-ttu-id="a70c5-117">Код приложения вместе с файлом конфигурации web.config фиксируется в репозитории исходного кода в Azure Repos.</span><span class="sxs-lookup"><span data-stu-id="a70c5-117">Application code including the web.config file is committed to the source code repository in Azure Repos.</span></span>
3. <span data-ttu-id="a70c5-118">Непрерывная интеграция активирует сборку приложения и модульные тесты, используя Azure Test Plans.</span><span class="sxs-lookup"><span data-stu-id="a70c5-118">Continuous integration triggers application build and unit tests using Azure Test Plans.</span></span>
4. <span data-ttu-id="a70c5-119">Непрерывное развертывание в Azure Pipelines активирует автоматическое развертывание артефактов приложения *со значениями конфигурации для конкретной среды*.</span><span class="sxs-lookup"><span data-stu-id="a70c5-119">Continuous deployment within Azure Pipelines triggers an automated deployment of application artifacts *with environment-specific configuration values*.</span></span>
5. <span data-ttu-id="a70c5-120">Артефакты развертываются в Службу приложений Azure.</span><span class="sxs-lookup"><span data-stu-id="a70c5-120">The artifacts are deployed to Azure App Service.</span></span>
6. <span data-ttu-id="a70c5-121">Служба Azure Application Insights собирает и анализирует данные о работоспособности, производительности и использовании ресурсов.</span><span class="sxs-lookup"><span data-stu-id="a70c5-121">Azure Application Insights collects and analyzes health, performance, and usage data.</span></span>
7. <span data-ttu-id="a70c5-122">Разработчики отслеживают работоспособность, производительность и сведения об использовании и управляют этими аспектами.</span><span class="sxs-lookup"><span data-stu-id="a70c5-122">Developers monitor and mange health, performance, and usage information.</span></span>
8. <span data-ttu-id="a70c5-123">Информация о невыполненной работе используется для назначения приоритетов новым функциям и исправлениям ошибок с помощью Azure Boards.</span><span class="sxs-lookup"><span data-stu-id="a70c5-123">Backlog information is used to prioritize new features and bug fixes using Azure Boards.</span></span>

### <a name="components"></a><span data-ttu-id="a70c5-124">Компоненты</span><span class="sxs-lookup"><span data-stu-id="a70c5-124">Components</span></span>

- <span data-ttu-id="a70c5-125">[Azure DevOps][vsts] — это служба, которая позволяет управлять жизненным циклом разработки от начала и до конца &mdash; начиная с планирования и управления проектом до управления кодом, сборки и выпуска.</span><span class="sxs-lookup"><span data-stu-id="a70c5-125">[Azure DevOps][vsts] is a service for managing your development life cycle end-to-end &mdash; from planning and project management, to code management, and continuing to build and release.</span></span>

- <span data-ttu-id="a70c5-126">[Веб-приложения Azure][web-apps] — это служба для размещения веб-приложений, интерфейсов REST API и серверной части мобильных решений.</span><span class="sxs-lookup"><span data-stu-id="a70c5-126">[Azure Web Apps][web-apps] is a PaaS service for hosting web applications, REST APIs, and mobile back ends.</span></span> <span data-ttu-id="a70c5-127">Хотя в этой статье основное внимание уделяется .NET, есть несколько дополнительных поддерживаемых вариантов платформ разработки.</span><span class="sxs-lookup"><span data-stu-id="a70c5-127">While this article focuses on .NET, there are several additional development platform options supported.</span></span>

- <span data-ttu-id="a70c5-128">[Application Insights][application-insights] — это расширяемая служба управления производительностью приложений (APM), получающая данные напрямую из источника и предназначенная для веб-разработчиков на нескольких платформах.</span><span class="sxs-lookup"><span data-stu-id="a70c5-128">[Application Insights][application-insights] is a first-party, extensible Application Performance Management (APM) service for web developers on multiple platforms.</span></span>

### <a name="alternatives"></a><span data-ttu-id="a70c5-129">Альтернативные варианты</span><span class="sxs-lookup"><span data-stu-id="a70c5-129">Alternatives</span></span>

<span data-ttu-id="a70c5-130">Хотя эта статья посвящена Azure DevOps, [Azure DevOps Server][azure-devops-server] (ранее известный как Team Foundation Server) может использоваться в качестве локального средства для замены.</span><span class="sxs-lookup"><span data-stu-id="a70c5-130">While this article focuses on Azure DevOps, [Azure DevOps Server][azure-devops-server] (previously known as Team Foundation Server) could be used as an on-premises substitute.</span></span> <span data-ttu-id="a70c5-131">Кроме того, также можно использовать набор технологий для разработки конвейера с открытым исходным кодом с помощью [Jenkins][jenkins-on-azure].</span><span class="sxs-lookup"><span data-stu-id="a70c5-131">Alternatively, you could also use a set of technologies for an open-source development pipeline using [Jenkins][jenkins-on-azure].</span></span>

<span data-ttu-id="a70c5-132">В контексте инфраструктуры как кода [шаблоны Resource Manager][arm-templates] использовались как часть проекта Azure DevOps, но вы можете использовать и другие технологии управления, например [Terraform][terraform] или [Chef][chef].</span><span class="sxs-lookup"><span data-stu-id="a70c5-132">From an infrastructure-as-code perspective, [Resource Manager templates][arm-templates] were used as part of the Azure DevOps project, but you could consider other management technologies such as [Terraform][terraform] or [Chef][chef].</span></span> <span data-ttu-id="a70c5-133">Если вы предпочитаете развертывание по схеме "инфраструктура как услуга" (IaaS) и вам требуется управления конфигурацией, можно использовать службу [Настройка состояния службы автоматизации Azure][desired-state-configuration], [Ansible][ansible] или [Chef][chef].</span><span class="sxs-lookup"><span data-stu-id="a70c5-133">If you prefer an infrastructure-as-a-service (IaaS)-based deployment and require configuration management, you could consider either [Azure Automation State Configuration][desired-state-configuration], [Ansible][ansible], or [Chef][chef].</span></span>

<span data-ttu-id="a70c5-134">Можно использовать следующие альтернативы для размещения веб-приложений Azure.</span><span class="sxs-lookup"><span data-stu-id="a70c5-134">You could consider these alternatives to hosting in Azure Web Apps:</span></span>

- <span data-ttu-id="a70c5-135">[Виртуальные машины Azure][compare-vm-hosting] справляются с рабочими нагрузками, требующими высокой степени управляемости или зависящими от компонентов операционной системы и служб, что невозможно реализовать в веб-приложениях (например, Windows GAC или модель COM).</span><span class="sxs-lookup"><span data-stu-id="a70c5-135">[Azure Virtual Machines][compare-vm-hosting] handles workloads that require a high degree of control, or depend on OS components and services that are not possible with Web Apps (for example, the Windows GAC, or COM).</span></span>

- <span data-ttu-id="a70c5-136">[Service Fabric][service-fabric] — это хороший вариант, если архитектура рабочей нагрузки предназначена для распределенных компонентов, которые получают преимущества от развертывания и выполнения в кластере с высокой степенью управления.</span><span class="sxs-lookup"><span data-stu-id="a70c5-136">[Service Fabric][service-fabric] is a good option if the workload architecture is focused around distributed components that benefit from being deployed and run across a cluster with a high degree of control.</span></span> <span data-ttu-id="a70c5-137">Service Fabric также может использоваться для размещения контейнеров.</span><span class="sxs-lookup"><span data-stu-id="a70c5-137">Service Fabric can also be used to host containers.</span></span>

- <span data-ttu-id="a70c5-138">[Функции Azure][azure-functions] предоставляют эффективный бессерверный подход, если архитектура рабочей нагрузки сосредоточена на распределенных компонентах с высокой степенью детализации, требующих минимальных зависимостей, в которых отдельные компоненты должны выполняться только по требованию (не непрерывно), а оркестрация компонентов не требуется.</span><span class="sxs-lookup"><span data-stu-id="a70c5-138">[Azure Functions][azure-functions] provides an effective serverless approach if the workload architecture is centered around fine grained distributed components, requiring minimal dependencies, where individual components are only required to run on demand (not continuously) and orchestration of components is not required.</span></span>

<span data-ttu-id="a70c5-139">Это [дерево принятия решений для вычислительных служб Azure](/azure/architecture/guide/technology-choices/compute-decision-tree) может помочь при выборе правильного пути для миграции.</span><span class="sxs-lookup"><span data-stu-id="a70c5-139">This [decision tree for Azure compute services](/azure/architecture/guide/technology-choices/compute-decision-tree) may help when choosing the right path to take for a migration.</span></span>

## <a name="management-and-security-considerations"></a><span data-ttu-id="a70c5-140">Управление и безопасность</span><span class="sxs-lookup"><span data-stu-id="a70c5-140">Management and Security Considerations</span></span>

- <span data-ttu-id="a70c5-141">Рассмотрите возможность использования одной из [задач разметки][vsts-tokenization], которые доступны в VSTS marketplace.</span><span class="sxs-lookup"><span data-stu-id="a70c5-141">Consider leveraging one of the [tokenization tasks][vsts-tokenization] available in the VSTS marketplace.</span></span>

- <span data-ttu-id="a70c5-142">Задачи [Azure Key Vault][download-keyvault-secrets] могут загружать секреты из Azure Key Vault в ваш выпуск.</span><span class="sxs-lookup"><span data-stu-id="a70c5-142">[Azure Key Vault][download-keyvault-secrets] tasks can download secrets from an Azure Key Vault into your release.</span></span> <span data-ttu-id="a70c5-143">Затем эти секреты можно использовать в качестве переменных в своем определении выпуска, что позволит избежать их хранения в системе управления версиями.</span><span class="sxs-lookup"><span data-stu-id="a70c5-143">You can then use those secrets as variables in your release definition, which avoids storing them in source control.</span></span>

- <span data-ttu-id="a70c5-144">Используйте [переменные выпуска][vsts-release-variables] в определениях выпуска для изменения конфигурации среды.</span><span class="sxs-lookup"><span data-stu-id="a70c5-144">Use [release variables][vsts-release-variables] in your release definitions to drive configuration changes of your environments.</span></span> <span data-ttu-id="a70c5-145">Переменные выпуска могут охватывать как весь выпуск, так и конкретную среду.</span><span class="sxs-lookup"><span data-stu-id="a70c5-145">Release variables can be scoped to an entire release or a given environment.</span></span> <span data-ttu-id="a70c5-146">При использовании переменных для секретной информации убедитесь, что выбран значок с изображением замка.</span><span class="sxs-lookup"><span data-stu-id="a70c5-146">When using variables for secret information, ensure that you select the padlock icon.</span></span>

- <span data-ttu-id="a70c5-147">Следует использовать [шлюзы развертывания][vsts-deployment-gates] в конвейере выпуска.</span><span class="sxs-lookup"><span data-stu-id="a70c5-147">[Deployment gates][vsts-deployment-gates] should be used in your release pipeline.</span></span> <span data-ttu-id="a70c5-148">Это позволяет использовать данные мониторинга во внешних системах (например системах управления инцидентами или дополнительных пользовательских системах), чтобы определить, следует ли повысить уровень выпуска.</span><span class="sxs-lookup"><span data-stu-id="a70c5-148">This lets you leverage monitoring data in association with external systems (for example, incident management or additional bespoke systems) to determine whether a release should be promoted.</span></span>

- <span data-ttu-id="a70c5-149">Если требуется вмешательство в конвейер выпуска, используйте функциональность [утверждений][vsts-approvals].</span><span class="sxs-lookup"><span data-stu-id="a70c5-149">Where manual intervention in a release pipeline is required, use the [approvals][vsts-approvals] functionality.</span></span>

- <span data-ttu-id="a70c5-150">При выпуске конвейера рассмотрите возможность использования [Application Insights][application-insights] и дополнительных средств мониторинга как можно раньше.</span><span class="sxs-lookup"><span data-stu-id="a70c5-150">Consider using [Application Insights][application-insights] and additional monitoring tools as early as possible in your release pipeline.</span></span> <span data-ttu-id="a70c5-151">Множество организаций начинают проводить мониторинг только в своей рабочей среде.</span><span class="sxs-lookup"><span data-stu-id="a70c5-151">Many organizations only begin monitoring in their production environment.</span></span> <span data-ttu-id="a70c5-152">Отслеживая другие среды, вы можете определять ошибки на ранних этапах разработки и избегать проблем в рабочей среде.</span><span class="sxs-lookup"><span data-stu-id="a70c5-152">By monitoring your other environments, you can identify bugs earlier in the development process and avoid issues in your production environment.</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="a70c5-153">Развертывание сценария</span><span class="sxs-lookup"><span data-stu-id="a70c5-153">Deploy the scenario</span></span>

### <a name="prerequisites"></a><span data-ttu-id="a70c5-154">Технические условия</span><span class="sxs-lookup"><span data-stu-id="a70c5-154">Prerequisites</span></span>

- <span data-ttu-id="a70c5-155">Необходимо иметь учетную запись Azure.</span><span class="sxs-lookup"><span data-stu-id="a70c5-155">You must have an existing Azure account.</span></span> <span data-ttu-id="a70c5-156">Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.</span><span class="sxs-lookup"><span data-stu-id="a70c5-156">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>

- <span data-ttu-id="a70c5-157">Вам нужно зарегистрироваться в организации Azure DevOps.</span><span class="sxs-lookup"><span data-stu-id="a70c5-157">You must sign up for an Azure DevOps organization.</span></span> <span data-ttu-id="a70c5-158">Дополнительные сведения см. в [кратком руководстве по созданию собственной организации][vsts-account-create].</span><span class="sxs-lookup"><span data-stu-id="a70c5-158">For more information, see [Quickstart: Create your organization][vsts-account-create].</span></span>

### <a name="walk-through"></a><span data-ttu-id="a70c5-159">Пошаговое руководство</span><span class="sxs-lookup"><span data-stu-id="a70c5-159">Walk-through</span></span>

<span data-ttu-id="a70c5-160">[Проекты Azure DevOps](/azure/devops-project/azure-devops-project-github) будет развернуть план службы приложений службы приложений и ресурса App Insights для вас, а также настроить конвейер конвейеры Azure для вас.</span><span class="sxs-lookup"><span data-stu-id="a70c5-160">[Azure DevOps Projects](/azure/devops-project/azure-devops-project-github) will deploy an App Service Plan, App Service, and an App Insights resource for you, as well as configure an Azure Pipelines pipeline for you.</span></span>

<span data-ttu-id="a70c5-161">После того как вы настроили конвейер с помощью проектов Azure DevOps и сборка завершена, просмотрите связанные изменения кода, рабочие элементы и результаты тестирования.</span><span class="sxs-lookup"><span data-stu-id="a70c5-161">Once you've configure a pipeline with Azure DevOps Projects and the build is completed, review the associated code changes, work items, and test results.</span></span> <span data-ttu-id="a70c5-162">Вы заметите, что результаты теста не отображаются, так как в коде нет никаких тестов для выполнения.</span><span class="sxs-lookup"><span data-stu-id="a70c5-162">You will notice that no test results are displayed, because the code does not contain any tests to run.</span></span>

<span data-ttu-id="a70c5-163">Конвейер создает определение выпуска и триггера непрерывного развертывания, развертывание наше приложение в среде разработки.</span><span class="sxs-lookup"><span data-stu-id="a70c5-163">The pipeline creates a release definition and a continuous deployment trigger, deploying our application into the Dev environment.</span></span> <span data-ttu-id="a70c5-164">В рамках процесса непрерывного развертывания можно увидеть разброс выпусков в нескольких средах.</span><span class="sxs-lookup"><span data-stu-id="a70c5-164">As part of a continuous deployment process, you may see releases that span multiple environments.</span></span> <span data-ttu-id="a70c5-165">Выпуск может охватывать обе инфраструктуры (используя такие методы, как инфраструктура как код), а также развертывать необходимые пакеты приложений и все выполняемые после настройки задачи.</span><span class="sxs-lookup"><span data-stu-id="a70c5-165">A release can span both infrastructure (using techniques such as infrastructure-as-code), and can also deploy the application packages required along with any post-configuration tasks.</span></span>

## <a name="pricing"></a><span data-ttu-id="a70c5-166">Цены</span><span class="sxs-lookup"><span data-stu-id="a70c5-166">Pricing</span></span>

<span data-ttu-id="a70c5-167">Расчет стоимости Azure DevOps будет зависеть от количества пользователей в организации, которым требуется доступ, в дополнение к таким факторам, как количество параллельных требуемых выпусков и сборок, а также число тестовых пользователей.</span><span class="sxs-lookup"><span data-stu-id="a70c5-167">Azure DevOps costs depend on the number of users in your organization that require access, along with other factors like the number of concurrent build/releases required and number of test users.</span></span> <span data-ttu-id="a70c5-168">Дополнительные сведения см. [Цены на Azure DevOps][vsts-pricing-page].</span><span class="sxs-lookup"><span data-stu-id="a70c5-168">For more information, see [Azure DevOps pricing][vsts-pricing-page].</span></span>

<span data-ttu-id="a70c5-169">[Калькулятор цен][vsts-pricing-calculator] предоставляет оценку запуска Azure DevOps с 20 пользователями.</span><span class="sxs-lookup"><span data-stu-id="a70c5-169">This [pricing calculator][vsts-pricing-calculator] provides an estimate for running Azure DevOps with 20 users.</span></span>

<span data-ttu-id="a70c5-170">Azure DevOps оплачивается в расчете на каждого пользователя за месяц.</span><span class="sxs-lookup"><span data-stu-id="a70c5-170">Azure DevOps is billed on a per-user per-month basis.</span></span> <span data-ttu-id="a70c5-171">В дополнение к оплате за дополнительных тестовых пользователей или базовые пользовательские лицензии может взиматься плата в зависимости от требуемых параллельных конвейеров.</span><span class="sxs-lookup"><span data-stu-id="a70c5-171">There may be additional charges dependent upon concurrent pipelines needed, in addition to any additional test users or user basic licenses.</span></span>

## <a name="related-resources"></a><span data-ttu-id="a70c5-172">Связанные ресурсы</span><span class="sxs-lookup"><span data-stu-id="a70c5-172">Related resources</span></span>

<span data-ttu-id="a70c5-173">Для получения дополнительных сведений о CI/CD и Azure DevOps ознакомьтесь со следующими ресурсами.</span><span class="sxs-lookup"><span data-stu-id="a70c5-173">Review the following resources to learn more about CI/CD and Azure DevOps:</span></span>

- <span data-ttu-id="a70c5-174">[What is DevOps?][devops-whatis] (Что такое DevOps?)</span><span class="sxs-lookup"><span data-stu-id="a70c5-174">[What is DevOps?][devops-whatis]</span></span>
- <span data-ttu-id="a70c5-175">[DevOps в Майкрософт. Работа с Azure DevOps][devops-microsoft]</span><span class="sxs-lookup"><span data-stu-id="a70c5-175">[DevOps at Microsoft - How we work with Azure DevOps][devops-microsoft]</span></span>
- <span data-ttu-id="a70c5-176">[Пошаговые руководства по использованию DevOps вместе с Azure DevOps][devops-with-vsts]</span><span class="sxs-lookup"><span data-stu-id="a70c5-176">[Step-by-step Tutorials: DevOps with Azure DevOps][devops-with-vsts]</span></span>
- <span data-ttu-id="a70c5-177">[Контрольный список DevOps][devops-checklist]</span><span class="sxs-lookup"><span data-stu-id="a70c5-177">[DevOps Checklist][devops-checklist]</span></span>
- <span data-ttu-id="a70c5-178">[Создание конвейера CI/CD для .NET с помощью проектов Azure DevOps][devops-project-create]</span><span class="sxs-lookup"><span data-stu-id="a70c5-178">[Create a CI/CD pipeline for .NET with Azure DevOps Projects][devops-project-create]</span></span>

<!-- links -->

[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/architecture-devops-dotnet-webapp.svg
[chef]: /azure/chef/
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[desired-state-configuration]: /azure/automation/automation-dsc-overview
[devops-microsoft]: /azure/devops/devops-at-microsoft/
[devops-with-vsts]: https://almvm.azurewebsites.net/labs/vsts/
[devops-checklist]: /azure/architecture/checklist/dev-ops
[application-insights]: https://azure.microsoft.com/services/application-insights/
[cloud-based-load-testing]: https://visualstudio.microsoft.com/team-services/cloud-load-testing/
[cloud-based-load-testing-on-premises]: /vsts/test/load-test/clt-with-private-machines?view=vsts
[jenkins-on-azure]: /azure/jenkins/
[devops-whatis]: /azure/devops/what-is-devops
[download-keyvault-secrets]: /vsts/pipelines/tasks/deploy/azure-key-vault?view=vsts
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[resiliency-app-service]: /azure/architecture/checklist/resiliency-per-service#app-service
[vsts]: /vsts/?view=vsts#pivot=services
[continuous-integration]: /azure/devops/what-is-continuous-integration
[continuous-delivery]: /azure/devops/what-is-continuous-delivery
[web-apps]: /azure/app-service/app-service-web-overview
[vsts-account-create]: /azure/devops/organizations/accounts/create-organization-msa-or-work-student?view=vsts
[vsts-approvals]: /vsts/pipelines/release/approvals/approvals?view=vsts
[devops-project]: https://portal.azure.com/?feature.customportal=false#create/Microsoft.AzureProject
[vsts-deployment-gates]: /vsts/pipelines/release/approvals/gates?view=vsts
[vsts-pricing-calculator]: https://azure.com/e/498aa024454445a8a352e75724f900b1
[vsts-pricing-page]: https://azure.microsoft.com/pricing/details/visual-studio-team-services/
[vsts-release-variables]: /vsts/pipelines/release/variables?view=vsts&tabs=batch
[vsts-tokenization]: https://marketplace.visualstudio.com/search?term=token&target=VSTS&category=All%20categories&sortBy=Relevance
[azure-key-vault]: /azure/key-vault/key-vault-overview
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[azure-devops-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]: /azure/service-fabric/
[azure-functions]: /azure/azure-functions/
[azure-containers]: https://azure.microsoft.com/overview/containers/
[compare-vm-hosting]: /azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]: /azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]: /azure/best-practices-availability-paired-regions
[devops-project-create]: /azure/devops-project/azure-devops-project-aspnet-core
[terraform]: /azure/terraform/
