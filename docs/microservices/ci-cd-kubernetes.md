---
title: Создание конвейера CI/CD для микрослужб в Kubernetes
description: Описывается конвейер CI/CD с пример развертывания микрослужб для службы Azure Kubernetes (AKS).
author: MikeWasson
ms.date: 04/11/2019
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: microservices
ms.openlocfilehash: e7da8a1b4111fb7856b919b40d033833a4e475e0
ms.sourcegitcommit: d58e6b2b891c9c99e951c59f15fce71addcb96b1
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/12/2019
ms.locfileid: "59533225"
---
<!-- markdownlint-disable MD040 -->

# <a name="building-a-cicd-pipeline-for-microservices-on-kubernetes"></a><span data-ttu-id="f4591-103">Создание конвейера CI/CD для микрослужб в Kubernetes</span><span class="sxs-lookup"><span data-stu-id="f4591-103">Building a CI/CD pipeline for microservices on Kubernetes</span></span>

<span data-ttu-id="f4591-104">Он может оказаться сложной задачей для создания надежного процесса CI/CD для архитектуры микрослужб.</span><span class="sxs-lookup"><span data-stu-id="f4591-104">It can be challenging to create a reliable CI/CD process for a microservices architecture.</span></span> <span data-ttu-id="f4591-105">Отдельные команды должен иметь возможность выпуска службы быстро и надежно, без нарушения работы других команд или нарушать стабильной работы приложения в целом.</span><span class="sxs-lookup"><span data-stu-id="f4591-105">Individual teams must be able to release services quickly and reliably, without disrupting other teams or destabilizing the application as a whole.</span></span>

<span data-ttu-id="f4591-106">В этой статье описывается пример конвейера CI/CD, для развертывания микрослужб для службы Azure Kubernetes (AKS).</span><span class="sxs-lookup"><span data-stu-id="f4591-106">This article describes an example CI/CD pipeline for deploying microservices to Azure Kubernetes Service (AKS).</span></span> <span data-ttu-id="f4591-107">Каждой команды и проекта отличается, поэтому в этой статье не принимают в качестве набора hard-and-fast правил.</span><span class="sxs-lookup"><span data-stu-id="f4591-107">Every team and project is different, so don't take this article as a set of hard-and-fast rules.</span></span> <span data-ttu-id="f4591-108">Вместо этого она создана для стать отправной точкой для разработки собственного процесса непрерывной Интеграции и Развертывания.</span><span class="sxs-lookup"><span data-stu-id="f4591-108">Instead, it's meant to be a starting point for designing your own CI/CD process.</span></span>

<span data-ttu-id="f4591-109">Этот пример конвейера, описанных здесь была создана для реализации микрослужб вызывается приложения доставки с помощью Дронов, который можно найти на [GitHub][ri].</span><span class="sxs-lookup"><span data-stu-id="f4591-109">The example pipeline described here was created for a microservices reference implementation called the Drone Delivery application, which you can find on [GitHub][ri].</span></span> <span data-ttu-id="f4591-110">Сценарий приложения описан [здесь](./design/index.md#reference-implementation).</span><span class="sxs-lookup"><span data-stu-id="f4591-110">The application scenario is described [here](./design/index.md#reference-implementation).</span></span>

<span data-ttu-id="f4591-111">Цели конвейер можно интерпретировать следующим образом:</span><span class="sxs-lookup"><span data-stu-id="f4591-111">The goals of the pipeline can be summarized as follows:</span></span>

- <span data-ttu-id="f4591-112">Команды можно создавать и развертывать свои службы независимо друг от друга.</span><span class="sxs-lookup"><span data-stu-id="f4591-112">Teams can build and deploy their services independently.</span></span>
- <span data-ttu-id="f4591-113">Изменения кода, прошедшие процесс непрерывной Интеграции, автоматически развертываются в рабочей среде.</span><span class="sxs-lookup"><span data-stu-id="f4591-113">Code changes that pass the CI process are automatically deployed to a production-like environment.</span></span>
- <span data-ttu-id="f4591-114">Системы контроля качества применяются на каждом этапе конвейера.</span><span class="sxs-lookup"><span data-stu-id="f4591-114">Quality gates are enforced at each stage of the pipeline.</span></span>
- <span data-ttu-id="f4591-115">Новую версию службы можно развернуть параллельно с предыдущей версии.</span><span class="sxs-lookup"><span data-stu-id="f4591-115">A new version of a service can be deployed side by side with the previous version.</span></span>

<span data-ttu-id="f4591-116">Дополнительные сведения см. в разделе [CI/CD для архитектур микрослужб](./ci-cd.md).</span><span class="sxs-lookup"><span data-stu-id="f4591-116">For more background, see [CI/CD for microservices architectures](./ci-cd.md).</span></span>

## <a name="assumptions"></a><span data-ttu-id="f4591-117">Предположения</span><span class="sxs-lookup"><span data-stu-id="f4591-117">Assumptions</span></span>

<span data-ttu-id="f4591-118">Для целей этого примера ниже приведены некоторые предположения о группе разработчиков и код базового.</span><span class="sxs-lookup"><span data-stu-id="f4591-118">For purposes of this example, here are some assumptions about the development team and the code base:</span></span>

- <span data-ttu-id="f4591-119">Репозиторий кода — monorepo, с папками микрослужб.</span><span class="sxs-lookup"><span data-stu-id="f4591-119">The code repository is a monorepo, with folders organized by microservice.</span></span>
- <span data-ttu-id="f4591-120">Стратегия создания ветви команды основывается на [разработке на основе магистрали](https://trunkbaseddevelopment.com/).</span><span class="sxs-lookup"><span data-stu-id="f4591-120">The team's branching strategy is based on [trunk-based development](https://trunkbaseddevelopment.com/).</span></span>
- <span data-ttu-id="f4591-121">Команды используют [ветвей выпуска](/azure/devops/repos/git/git-branching-guidance?view=azure-devops#manage-releases) к управлению выпусками.</span><span class="sxs-lookup"><span data-stu-id="f4591-121">The team uses [release branches](/azure/devops/repos/git/git-branching-guidance?view=azure-devops#manage-releases) to manage releases.</span></span> <span data-ttu-id="f4591-122">Для каждой микрослужбы создании отдельных выпусков.</span><span class="sxs-lookup"><span data-stu-id="f4591-122">Separate releases are created for each microservice.</span></span>
- <span data-ttu-id="f4591-123">Процесс Непрерывной интеграции и использует [конвейеры Azure](/azure/devops/pipelines/?view=azure-devops) для создания, тестирования и развертывания микрослужб для AKS.</span><span class="sxs-lookup"><span data-stu-id="f4591-123">The CI/CD process uses [Azure Pipelines](/azure/devops/pipelines/?view=azure-devops) to build, test, and deploy the microservices to AKS.</span></span>
- <span data-ttu-id="f4591-124">Образы контейнеров для каждой микрослужбы, хранятся в [реестр контейнеров Azure](/azure/container-registry/).</span><span class="sxs-lookup"><span data-stu-id="f4591-124">The container images for each microservice are stored in [Azure Container Registry](/azure/container-registry/).</span></span>
- <span data-ttu-id="f4591-125">Группа использует диаграмм Helm для упаковки каждой микрослужбы.</span><span class="sxs-lookup"><span data-stu-id="f4591-125">The team uses Helm charts to package each microservice.</span></span>

<span data-ttu-id="f4591-126">Эти предположения диска многие из конкретных особенностей конвейера CI/CD.</span><span class="sxs-lookup"><span data-stu-id="f4591-126">These assumptions drive many of the specific details of the CI/CD pipeline.</span></span> <span data-ttu-id="f4591-127">Однако базовый подход, описанный здесь быть адаптированы для других процессов, средств и служб, таких как Jenkins или Docker Hub.</span><span class="sxs-lookup"><span data-stu-id="f4591-127">However, the basic approach described here be adapted for other processes, tools, and services, such as Jenkins or Docker Hub.</span></span>

## <a name="validation-builds"></a><span data-ttu-id="f4591-128">Проверка сборок</span><span class="sxs-lookup"><span data-stu-id="f4591-128">Validation builds</span></span>

<span data-ttu-id="f4591-129">Предположим, что разработчик работает в микрослужбе, вызывает службу доставки.</span><span class="sxs-lookup"><span data-stu-id="f4591-129">Suppose that a developer is working on a microservice called the Delivery Service.</span></span> <span data-ttu-id="f4591-130">При разработке новых компонентов разработчик проверяет код в ветви компонентов.</span><span class="sxs-lookup"><span data-stu-id="f4591-130">While developing a new feature, the developer checks code into a feature branch.</span></span> <span data-ttu-id="f4591-131">По соглашению компоненты ветвей носят имя `feature/*`.</span><span class="sxs-lookup"><span data-stu-id="f4591-131">By convention, feature branches are named `feature/*`.</span></span>

![Рабочий процесс CI/CD](./images/aks-cicd-1.png)

<span data-ttu-id="f4591-133">Файл определения сборки содержит триггер, который фильтрует, укажите имя ветви и исходный путь:</span><span class="sxs-lookup"><span data-stu-id="f4591-133">The build definition file includes a trigger that filters by the branch name and the source path:</span></span>

```yaml
trigger:
  batch: true
  branches:
    include:
    - master
    - feature/*
    - topic/*

    exclude:
    - feature/experimental/*
    - topic/experimental/*

  paths:
     include:
     - /src/shipping/delivery/
```

<span data-ttu-id="f4591-134">&#11162;См. в разделе [исходный файл](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-validation.yml).</span><span class="sxs-lookup"><span data-stu-id="f4591-134">&#11162; See the [source file](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-validation.yml).</span></span>

<span data-ttu-id="f4591-135">Благодаря этому подходу каждая команда может иметь свой собственный конвейер сборки.</span><span class="sxs-lookup"><span data-stu-id="f4591-135">Using this approach, each team can have its own build pipeline.</span></span> <span data-ttu-id="f4591-136">Только код, который возвращается в `/src/shipping/delivery` папки активирует сборку службы доставки.</span><span class="sxs-lookup"><span data-stu-id="f4591-136">Only code that is checked into the `/src/shipping/delivery` folder triggers a build of the Delivery Service.</span></span> <span data-ttu-id="f4591-137">Отправив фиксации в ветви, соответствующие критерию фильтра активирует сборку непрерывной Интеграции.</span><span class="sxs-lookup"><span data-stu-id="f4591-137">Pushing commits to a branch that matches the filter triggers a CI build.</span></span> <span data-ttu-id="f4591-138">На этом этапе в рабочем процессе сборка CI выполняет некоторые минимальные проверки кода.</span><span class="sxs-lookup"><span data-stu-id="f4591-138">At this point in the workflow, the CI build runs some minimal code verification:</span></span>

1. <span data-ttu-id="f4591-139">Создавайте код.</span><span class="sxs-lookup"><span data-stu-id="f4591-139">Build the code.</span></span>
1. <span data-ttu-id="f4591-140">Выполнение модульных тестов.</span><span class="sxs-lookup"><span data-stu-id="f4591-140">Run unit tests.</span></span>

<span data-ttu-id="f4591-141">Целью является для сохранения времени построения короткий, поэтому разработчик может получить быструю обратную связь.</span><span class="sxs-lookup"><span data-stu-id="f4591-141">The goal is to keep build times short, so the developer can get quick feedback.</span></span> <span data-ttu-id="f4591-142">Когда компонент готов выполнить слияние в главную ветвь, разработчик Открывает запрос на Вытягивание</span><span class="sxs-lookup"><span data-stu-id="f4591-142">Once the feature is ready to merge into master, the developer opens a PR.</span></span> <span data-ttu-id="f4591-143">Этим активируется другая сборка CI, которая выполняет некоторые дополнительные проверки.</span><span class="sxs-lookup"><span data-stu-id="f4591-143">This triggers another CI build that performs some additional checks:</span></span>

1. <span data-ttu-id="f4591-144">Создавайте код.</span><span class="sxs-lookup"><span data-stu-id="f4591-144">Build the code.</span></span>
1. <span data-ttu-id="f4591-145">Выполнение модульных тестов.</span><span class="sxs-lookup"><span data-stu-id="f4591-145">Run unit tests.</span></span>
1. <span data-ttu-id="f4591-146">Создание образа среды выполнения контейнера.</span><span class="sxs-lookup"><span data-stu-id="f4591-146">Build the runtime container image.</span></span>
1. <span data-ttu-id="f4591-147">Выполните сканирование уязвимостей в образе.</span><span class="sxs-lookup"><span data-stu-id="f4591-147">Run vulnerability scans on the image.</span></span>

![Рабочий процесс CI/CD](./images/aks-cicd-2.png)

> [!NOTE]
> <span data-ttu-id="f4591-149">В репозитории Azure DevOps, вы можете определить [политики](/azure/devops/repos/git/branch-policies) для защиты ветвей.</span><span class="sxs-lookup"><span data-stu-id="f4591-149">In Azure DevOps Repos, you can define [policies](/azure/devops/repos/git/branch-policies) to protect branches.</span></span> <span data-ttu-id="f4591-150">Например, для слияния в главную ветвь политика может требовать успешную сборку CI, а также утверждение от утверждающего.</span><span class="sxs-lookup"><span data-stu-id="f4591-150">For example, the policy could require a successful CI build plus a sign-off from an approver in order to merge into master.</span></span>

## <a name="full-cicd-build"></a><span data-ttu-id="f4591-151">Полная сборка CI/CD</span><span class="sxs-lookup"><span data-stu-id="f4591-151">Full CI/CD build</span></span>

<span data-ttu-id="f4591-152">На некотором этапе группа готова к развертыванию новой версии службы доставки.</span><span class="sxs-lookup"><span data-stu-id="f4591-152">At some point, the team is ready to deploy a new version of the Delivery service.</span></span> <span data-ttu-id="f4591-153">Руководитель выпуска создается ветвь с основного сервера с такому шаблону именования: `release/<microservice name>/<semver>`.</span><span class="sxs-lookup"><span data-stu-id="f4591-153">The release manager creates a branch from master with this naming pattern: `release/<microservice name>/<semver>`.</span></span> <span data-ttu-id="f4591-154">Например, `release/delivery/v1.0.2`.</span><span class="sxs-lookup"><span data-stu-id="f4591-154">For example, `release/delivery/v1.0.2`.</span></span>

![Рабочий процесс CI/CD](./images/aks-cicd-3.png)

<span data-ttu-id="f4591-156">Создание этой ветви триггеров полная сборка непрерывной Интеграции, выполняет все предыдущие шаги плюс:</span><span class="sxs-lookup"><span data-stu-id="f4591-156">Creation of this branch triggers a full CI build that runs all of the previous steps plus:</span></span>

1. <span data-ttu-id="f4591-157">Отправьте образ контейнера в реестр контейнеров Azure.</span><span class="sxs-lookup"><span data-stu-id="f4591-157">Push the container image to Azure Container Registry.</span></span> <span data-ttu-id="f4591-158">Образ отмечен номером версии, взятым из имени ветви.</span><span class="sxs-lookup"><span data-stu-id="f4591-158">The image is tagged with the version number taken from the branch name.</span></span>
2. <span data-ttu-id="f4591-159">Запустите `helm package` упаковка чарта Helm для службы.</span><span class="sxs-lookup"><span data-stu-id="f4591-159">Run `helm package` to package the Helm chart for the service.</span></span> <span data-ttu-id="f4591-160">Диаграммы также по значку с номером версии.</span><span class="sxs-lookup"><span data-stu-id="f4591-160">The chart is also tagged with a version number.</span></span>
3. <span data-ttu-id="f4591-161">Передача пакета Helm в реестр контейнеров.</span><span class="sxs-lookup"><span data-stu-id="f4591-161">Push the Helm package to Container Registry.</span></span>

<span data-ttu-id="f4591-162">При условии, что эта сборка завершается успешно, запускает процесс развертывания (CD) с помощью конвейеров Azure [конвейер выпуска](/azure/devops/pipelines/release/what-is-release-management).</span><span class="sxs-lookup"><span data-stu-id="f4591-162">Assuming this build succeeds, it triggers a deployment (CD) process using an Azure Pipelines [release pipeline](/azure/devops/pipelines/release/what-is-release-management).</span></span> <span data-ttu-id="f4591-163">Этот конвейер содержит следующие действия:</span><span class="sxs-lookup"><span data-stu-id="f4591-163">This pipeline has the following steps:</span></span>

1. <span data-ttu-id="f4591-164">Развертывание чарта Helm в среде контроля Качества.</span><span class="sxs-lookup"><span data-stu-id="f4591-164">Deploy the Helm chart to a QA environment.</span></span>
1. <span data-ttu-id="f4591-165">Утверждающий утверждает, прежде чем пакет переместится в рабочую среду.</span><span class="sxs-lookup"><span data-stu-id="f4591-165">An approver signs off before the package moves to production.</span></span> <span data-ttu-id="f4591-166">Дополнительные сведения см. в статье [Release deployment control using approvals](/azure/devops/pipelines/release/approvals/approvals) (Управление развертыванием выпуска с помощью утверждений).</span><span class="sxs-lookup"><span data-stu-id="f4591-166">See [Release deployment control using approvals](/azure/devops/pipelines/release/approvals/approvals).</span></span>
1. <span data-ttu-id="f4591-167">Повторной образа Docker для пространства имен production в реестре контейнеров Azure.</span><span class="sxs-lookup"><span data-stu-id="f4591-167">Retag the Docker image for the production namespace in Azure Container Registry.</span></span> <span data-ttu-id="f4591-168">Например, если текущий маркер – `myrepo.azurecr.io/delivery:v1.0.2`, рабочим маркером будет `myrepo.azurecr.io/prod/delivery:v1.0.2`.</span><span class="sxs-lookup"><span data-stu-id="f4591-168">For example, if the current tag is `myrepo.azurecr.io/delivery:v1.0.2`, the production tag is `myrepo.azurecr.io/prod/delivery:v1.0.2`.</span></span>
1. <span data-ttu-id="f4591-169">Развертывание чарта Helm в рабочую среду.</span><span class="sxs-lookup"><span data-stu-id="f4591-169">Deploy the Helm chart to the production environment.</span></span>

<span data-ttu-id="f4591-170">Даже в monorepo эти задачи могут быть распространены на отдельные микрослужбы, таким образом, чтобы команды можно развернуть с высокой скоростью.</span><span class="sxs-lookup"><span data-stu-id="f4591-170">Even in a monorepo, these tasks can be scoped to individual microservices, so that teams can deploy with high velocity.</span></span> <span data-ttu-id="f4591-171">Процесс состоит из ряда операций вручную. утверждение PR, создание ветви выпуска и утверждение развертывания в рабочем кластере.</span><span class="sxs-lookup"><span data-stu-id="f4591-171">The process has some manual steps: Approving PRs, creating release branches, and approving deployments into the production cluster.</span></span> <span data-ttu-id="f4591-172">Эти действия выполняются вручную политикой &mdash; они можно автоматизировать, если является предпочтительным для организации.</span><span class="sxs-lookup"><span data-stu-id="f4591-172">These steps are manual by policy &mdash; they could be automated if the organization prefers.</span></span>

## <a name="isolation-of-environments"></a><span data-ttu-id="f4591-173">Изоляция сред</span><span class="sxs-lookup"><span data-stu-id="f4591-173">Isolation of environments</span></span>

<span data-ttu-id="f4591-174">У вас будет несколько сред, в которых вы развертываете службы, включая среды для разработки, тестирования проверки сборки, интеграционного и нагрузочного тестирования и, наконец, рабочую среду.</span><span class="sxs-lookup"><span data-stu-id="f4591-174">You will have multiple environments where you deploy services, including environments for development, smoke testing, integration testing, load testing, and finally production.</span></span> <span data-ttu-id="f4591-175">Эти среды нуждаются в некоторой степени изоляции.</span><span class="sxs-lookup"><span data-stu-id="f4591-175">These environments need some level of isolation.</span></span> <span data-ttu-id="f4591-176">В Kubernetes у вас есть выбор между физической и логической изоляцией.</span><span class="sxs-lookup"><span data-stu-id="f4591-176">In Kubernetes, you have a choice between physical isolation and logical isolation.</span></span> <span data-ttu-id="f4591-177">Физическая изоляция означает развертывание в отдельных кластерах.</span><span class="sxs-lookup"><span data-stu-id="f4591-177">Physical isolation means deploying to separate clusters.</span></span> <span data-ttu-id="f4591-178">При логической изоляции используются пространства имен и политики, как описано выше.</span><span class="sxs-lookup"><span data-stu-id="f4591-178">Logical isolation makes use of namespaces and policies, as described earlier.</span></span>

<span data-ttu-id="f4591-179">Рекомендуется создать выделенный рабочий кластер вместе с отдельным кластером для сред разработки и тестирования.</span><span class="sxs-lookup"><span data-stu-id="f4591-179">Our recommendation is to create a dedicated production cluster along with a separate cluster for your dev/test environments.</span></span> <span data-ttu-id="f4591-180">Используйте логическую изоляцию для разделения сред в кластере разработки и тестирования.</span><span class="sxs-lookup"><span data-stu-id="f4591-180">Use logical isolation to separate environments within the dev/test cluster.</span></span> <span data-ttu-id="f4591-181">У служб, развернутых в кластере разработки и тестирования, никогда не должно быть доступа к хранилищам данных, в которых хранятся бизнес-данные.</span><span class="sxs-lookup"><span data-stu-id="f4591-181">Services deployed to the dev/test cluster should never have access to data stores that hold business data.</span></span>

## <a name="build-process"></a><span data-ttu-id="f4591-182">Процесс сборки</span><span class="sxs-lookup"><span data-stu-id="f4591-182">Build process</span></span>

<span data-ttu-id="f4591-183">Если это возможно, упакуйте процесс сборки в контейнер Docker.</span><span class="sxs-lookup"><span data-stu-id="f4591-183">When possible, package your build process into a Docker container.</span></span> <span data-ttu-id="f4591-184">Который позволяет создавать артефакты кода, с помощью Docker, без обязательной настройки среды сборки на каждом компьютере сборки.</span><span class="sxs-lookup"><span data-stu-id="f4591-184">That allows you to build your code artifacts using Docker, without needing to configure the build environment on each build machine.</span></span> <span data-ttu-id="f4591-185">Процесс построения контейнерных упрощает масштабное развертывание конвейера непрерывной Интеграции путем добавления новых агентов сборки.</span><span class="sxs-lookup"><span data-stu-id="f4591-185">A containerized build process makes it easy to scale out the CI pipeline by adding new build agents.</span></span> <span data-ttu-id="f4591-186">Кроме того любой разработчик в группе можно создавать код, просто запустив контейнер сборки.</span><span class="sxs-lookup"><span data-stu-id="f4591-186">Also, any developer on the team can build the code simply by running the build container.</span></span>

<span data-ttu-id="f4591-187">С помощью многоэтапная сборка в Docker, можно определить среды сборки и образ среды выполнения в одном файле Dockerfile.</span><span class="sxs-lookup"><span data-stu-id="f4591-187">By using multi-stage builds in Docker, you can define the build environment and the runtime image in a single Dockerfile.</span></span> <span data-ttu-id="f4591-188">Например ниже приведен файл Dockerfile, который выполняет построение приложения ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="f4591-188">For example, here's a Dockerfile that builds an ASP.NET Core application:</span></span>

```
FROM microsoft/dotnet:2.2-runtime AS base
WORKDIR /app

FROM microsoft/dotnet:2.2-sdk AS build
WORKDIR /src/Fabrikam.Workflow.Service

COPY Fabrikam.Workflow.Service/Fabrikam.Workflow.Service.csproj .
RUN dotnet restore Fabrikam.Workflow.Service.csproj

COPY Fabrikam.Workflow.Service/. .
RUN dotnet build Fabrikam.Workflow.Service.csproj -c Release -o /app

FROM build AS publish
RUN dotnet publish Fabrikam.Workflow.Service.csproj -c Release -o /app

FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "Fabrikam.Workflow.Service.dll"]
```

<span data-ttu-id="f4591-189">&#11162;См. в разделе [исходный файл](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/workflow/Dockerfile).</span><span class="sxs-lookup"><span data-stu-id="f4591-189">&#11162; See the [source file](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/workflow/Dockerfile).</span></span>

<span data-ttu-id="f4591-190">Этот файл Dockerfile определяет несколько этапов сборки.</span><span class="sxs-lookup"><span data-stu-id="f4591-190">This Dockerfile defines several build stages.</span></span> <span data-ttu-id="f4591-191">Обратите внимание, что рабочей области с именем `base` использует среда выполнения ASP.NET Core, в рабочей области с именем `build` использует полный пакет SDK для ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="f4591-191">Notice that the stage named `base` uses the ASP.NET Core runtime, while the stage named `build` uses the full ASP.NET Core SDK.</span></span> <span data-ttu-id="f4591-192">`build` Этап служит для построения проекта ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="f4591-192">The `build` stage is used to build the ASP.NET Core project.</span></span> <span data-ttu-id="f4591-193">Но создается контейнер окончательной среды выполнения из `base`, который содержит только среду выполнения и значительно меньше, чем полный образ пакета SDK.</span><span class="sxs-lookup"><span data-stu-id="f4591-193">But the final runtime container is built from `base`, which contains just the runtime and is significantly smaller than the full SDK image.</span></span>

### <a name="building-a-test-runner"></a><span data-ttu-id="f4591-194">Создание тестов</span><span class="sxs-lookup"><span data-stu-id="f4591-194">Building a test runner</span></span>

<span data-ttu-id="f4591-195">Еще один полезный прием — для запуска модульных тестов в контейнере.</span><span class="sxs-lookup"><span data-stu-id="f4591-195">Another good practice is to run unit tests in the container.</span></span> <span data-ttu-id="f4591-196">Например ниже приведен в файле Docker, который создает средство выполнения тестов:</span><span class="sxs-lookup"><span data-stu-id="f4591-196">For example, here is part of a Docker file that builds a test runner:</span></span>

```
FROM build AS testrunner
WORKDIR /src/tests

COPY Fabrikam.Workflow.Service.Tests/*.csproj .
RUN dotnet restore Fabrikam.Workflow.Service.Tests.csproj

COPY Fabrikam.Workflow.Service.Tests/. .
ENTRYPOINT ["dotnet", "test", "--logger:trx"]
```

<span data-ttu-id="f4591-197">Разработчик может использовать этот файл Docker для локального запуска тестов:</span><span class="sxs-lookup"><span data-stu-id="f4591-197">A developer can use this Docker file to run the tests locally:</span></span>

```bash
docker build . -t delivery-test:1 --target=testrunner
docker run -p 8080:8080 delivery-test:1
```

<span data-ttu-id="f4591-198">Конвейер непрерывной Интеграции также следует выполнить тесты как часть этапа проверки построения.</span><span class="sxs-lookup"><span data-stu-id="f4591-198">The CI pipeline should also run the tests as part of the build verification step.</span></span>

<span data-ttu-id="f4591-199">Обратите внимание, что этот файл используется Docker `ENTRYPOINT` команду, чтобы запустить тесты, не Docker `RUN` команды.</span><span class="sxs-lookup"><span data-stu-id="f4591-199">Note that this file uses the Docker `ENTRYPOINT` command to run the tests, not the Docker `RUN` command.</span></span>

- <span data-ttu-id="f4591-200">Если вы используете `RUN` команды выполнения тестов при сборке образа.</span><span class="sxs-lookup"><span data-stu-id="f4591-200">If you use the `RUN` command, the tests run every time you build the image.</span></span> <span data-ttu-id="f4591-201">С помощью `ENTRYPOINT`, тесты входят в.</span><span class="sxs-lookup"><span data-stu-id="f4591-201">By using `ENTRYPOINT`, the tests are opt-in.</span></span> <span data-ttu-id="f4591-202">Они выполняются только в том случае, если явно целью `testrunner` рабочей области.</span><span class="sxs-lookup"><span data-stu-id="f4591-202">They run only when you explicitly target the `testrunner` stage.</span></span>
- <span data-ttu-id="f4591-203">Непройденного теста не вызывает Docker `build` к сбою команды.</span><span class="sxs-lookup"><span data-stu-id="f4591-203">A failing test doesn't cause the Docker `build` command to fail.</span></span> <span data-ttu-id="f4591-204">Таким образом, можно отличить контейнер сборки ошибки от сбоев тестов.</span><span class="sxs-lookup"><span data-stu-id="f4591-204">That way you can distinguish container build failures from test failures.</span></span>
- <span data-ttu-id="f4591-205">На подключенный том можно сохранить результаты теста.</span><span class="sxs-lookup"><span data-stu-id="f4591-205">Test results can be saved to a mounted volume.</span></span>

### <a name="container-best-practices"></a><span data-ttu-id="f4591-206">Рекомендации по обеспечению контейнера</span><span class="sxs-lookup"><span data-stu-id="f4591-206">Container best practices</span></span>

<span data-ttu-id="f4591-207">Ниже приведены некоторые другие рекомендации для контейнеров.</span><span class="sxs-lookup"><span data-stu-id="f4591-207">Here are some other best practices to consider for containers:</span></span>

- <span data-ttu-id="f4591-208">Определите действующие для всей организации соглашения для тегов контейнеров, управления версиями и соглашения об именовании ресурсов, развернутых в кластере (модули, службы и т. д.).</span><span class="sxs-lookup"><span data-stu-id="f4591-208">Define organization-wide conventions for container tags, versioning, and naming conventions for resources deployed to the cluster (pods, services, and so on).</span></span> <span data-ttu-id="f4591-209">В результате этого будет легче диагностировать проблемы с развертыванием.</span><span class="sxs-lookup"><span data-stu-id="f4591-209">That can make it easier to diagnose deployment issues.</span></span>

- <span data-ttu-id="f4591-210">Во время цикла разработки и тестирования процесс непрерывной интеграции и поставки включает в себя сборку многих образов контейнера.</span><span class="sxs-lookup"><span data-stu-id="f4591-210">During the development and test cycle, the CI/CD process will build many container images.</span></span> <span data-ttu-id="f4591-211">Только некоторые из этих образов являются кандидатами на выпуск, а затем только некоторые из этих релиз-кандидаты будет получить повышаются до рабочей среды.</span><span class="sxs-lookup"><span data-stu-id="f4591-211">Only some of those images are candidates for release, and then only some of those release candidates will get promoted to production.</span></span> <span data-ttu-id="f4591-212">Иметь четкую стратегию управления версиями, вы узнаете, какие образы сейчас развернуты в рабочей среде и выполнить откат до предыдущей версии при необходимости.</span><span class="sxs-lookup"><span data-stu-id="f4591-212">Have a clear versioning strategy, so that you know which images are currently deployed to production, and can roll back to a previous version if necessary.</span></span>

- <span data-ttu-id="f4591-213">Всегда следует развертывать как теги контейнеров для конкретной версии, не `latest`.</span><span class="sxs-lookup"><span data-stu-id="f4591-213">Always deploy specific container version tags, not `latest`.</span></span>

- <span data-ttu-id="f4591-214">Используйте [пространства имен](/azure/container-registry/container-registry-best-practices#repository-namespaces) в реестре контейнеров Azure, чтобы изолировать образов, утвержденные для рабочей среды на основе образов, по-прежнему тестируемые.</span><span class="sxs-lookup"><span data-stu-id="f4591-214">Use [namespaces](/azure/container-registry/container-registry-best-practices#repository-namespaces) in Azure Container Registry to isolate images that are approved for production from images that are still being tested.</span></span> <span data-ttu-id="f4591-215">Не перемещайте изображение в рабочей среде пространство имен, пока не будете готовы к его развертыванию в рабочей среде.</span><span class="sxs-lookup"><span data-stu-id="f4591-215">Don't move an image into the production namespace until you're ready to deploy it into production.</span></span> <span data-ttu-id="f4591-216">При использовании такого подхода с семантическим управлением версиями образов контейнеров можно уменьшить вероятность случайного развертывания версии, не утвержденной для выпуска.</span><span class="sxs-lookup"><span data-stu-id="f4591-216">If you combine this practice with semantic versioning of container images, it can reduce the chance of accidentally deploying a version that wasn't approved for release.</span></span>

- <span data-ttu-id="f4591-217">Следуйте принципу наименьших прав доступа, выполнив контейнеры как со стороны непривилегированных пользователей.</span><span class="sxs-lookup"><span data-stu-id="f4591-217">Follow the principle of least privilege by running containers as a nonprivileged user.</span></span> <span data-ttu-id="f4591-218">В Kubernetes, можно создать политику безопасности pod, который предотвращает запуск в качестве контейнеров *корневой*.</span><span class="sxs-lookup"><span data-stu-id="f4591-218">In Kubernetes, you can create a pod security policy that prevents containers from running as *root*.</span></span> <span data-ttu-id="f4591-219">См. в разделе [предотвратить запуск с правами привилегированного пользователя модулей](https://docs.bitnami.com/kubernetes/how-to/secure-kubernetes-cluster-psp/).</span><span class="sxs-lookup"><span data-stu-id="f4591-219">See [Prevent Pods From Running With Root Privileges](https://docs.bitnami.com/kubernetes/how-to/secure-kubernetes-cluster-psp/).</span></span>

## <a name="helm-charts"></a><span data-ttu-id="f4591-220">Чарты Helm</span><span class="sxs-lookup"><span data-stu-id="f4591-220">Helm charts</span></span>

<span data-ttu-id="f4591-221">С помощью Helm можно управлять созданием и развертыванием служб.</span><span class="sxs-lookup"><span data-stu-id="f4591-221">Consider using Helm to manage building and deploying services.</span></span> <span data-ttu-id="f4591-222">Вот некоторые возможности Helm, которые помогут в Непрерывной интеграции и:</span><span class="sxs-lookup"><span data-stu-id="f4591-222">Here are some of the features of Helm that help with CI/CD:</span></span>

- <span data-ttu-id="f4591-223">Часто одна микрослужба определяется несколько объектов Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="f4591-223">Often a single microservice is defined by multiple Kubernetes objects.</span></span> <span data-ttu-id="f4591-224">Helm позволяет этим объектам быть упакована в одной диаграмме Helm.</span><span class="sxs-lookup"><span data-stu-id="f4591-224">Helm allows these objects to be packaged into a single Helm chart.</span></span>
- <span data-ttu-id="f4591-225">Диаграмму можно развернуть с помощью одной команды Helm, а не ряд команд kubectl.</span><span class="sxs-lookup"><span data-stu-id="f4591-225">A chart can be deployed with a single Helm command, rather than a series of kubectl commands.</span></span>
- <span data-ttu-id="f4591-226">Диаграммы, явным образом.</span><span class="sxs-lookup"><span data-stu-id="f4591-226">Charts are explicitly versioned.</span></span> <span data-ttu-id="f4591-227">Используйте Helm для выпустить версию и просмотреть выпуски откат к предыдущей версии.</span><span class="sxs-lookup"><span data-stu-id="f4591-227">Use Helm to release a version, view releases, and roll back to a previous version.</span></span> <span data-ttu-id="f4591-228">Отслеживание обновлений и исправлений с использованием семантических версий, а также возможность отката к предыдущей версии.</span><span class="sxs-lookup"><span data-stu-id="f4591-228">Tracking updates and revisions, using semantic versioning, along with the ability to roll back to a previous version.</span></span>
- <span data-ttu-id="f4591-229">Чарты helm используйте шаблоны, чтобы избежать дублирующейся информации, таких как метки и селекторы, в нескольких файлах.</span><span class="sxs-lookup"><span data-stu-id="f4591-229">Helm charts use templates to avoid duplicating information, such as labels and selectors, across many files.</span></span>
- <span data-ttu-id="f4591-230">Helm можно управлять зависимостями между диаграммы.</span><span class="sxs-lookup"><span data-stu-id="f4591-230">Helm can manage dependencies between charts.</span></span>
- <span data-ttu-id="f4591-231">Диаграммы можно хранить в репозитории Helm, например реестр контейнеров Azure и интегрировать в конвейер сборки.</span><span class="sxs-lookup"><span data-stu-id="f4591-231">Charts can be stored in a Helm repository, such as Azure Container Registry, and integrated into the build pipeline.</span></span>

<span data-ttu-id="f4591-232">Дополнительные сведения об использовании Реестра контейнеров Helm см. в статье [Использование Реестра контейнеров Azure в качестве репозитория Helm для диаграмм приложения](/azure/container-registry/container-registry-helm-repos).</span><span class="sxs-lookup"><span data-stu-id="f4591-232">For more information about using Container Registry as a Helm repository, see [Use Azure Container Registry as a Helm repository for your application charts](/azure/container-registry/container-registry-helm-repos).</span></span>

<span data-ttu-id="f4591-233">Одна микрослужба может включать несколько файлов конфигурации Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="f4591-233">A single microservice may involve multiple Kubernetes configuration files.</span></span> <span data-ttu-id="f4591-234">Обновление службы может означать касается всех этих файлов для обновления селекторы, метки и теги изображений.</span><span class="sxs-lookup"><span data-stu-id="f4591-234">Updating a service can mean touching all of these files to update selectors, labels, and image tags.</span></span> <span data-ttu-id="f4591-235">Helm обрабатывает их как единый пакет диаграмму и дает возможность легко обновить файлы yaml-ФАЙЛ с помощью переменных.</span><span class="sxs-lookup"><span data-stu-id="f4591-235">Helm treats these as a single package called a chart and allows you to easily update the YAML files by using variables.</span></span> <span data-ttu-id="f4591-236">Helm использует язык шаблона (на основе Go шаблонов) позволяет создавать параметризованные файлы конфигурации YAML.</span><span class="sxs-lookup"><span data-stu-id="f4591-236">Helm uses a template language (based on Go templates) to let you write parameterized YAML configuration files.</span></span>

<span data-ttu-id="f4591-237">Например Вот часть yaml-файл, который определяет развертывания:</span><span class="sxs-lookup"><span data-stu-id="f4591-237">For example, here's part of a YAML file that defines a deployment:</span></span>

```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: {{ include "package.fullname" . | replace "." "" }}
  labels:
    app.kubernetes.io/name: {{ include "package.name" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
  annotations:
    kubernetes.io/change-cause: {{ .Values.reason }}

...

  spec:
      containers:
      - name: &package-container_name fabrikam-package
        image: {{ .Values.dockerregistry }}/{{ .Values.image.repository }}:{{ .Values.image.tag }}
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        env:
        - name: LOG_LEVEL
          value: {{ .Values.log.level }}
```

<span data-ttu-id="f4591-238">&#11162;См. в разделе [исходный файл](https://github.com/mspnp/microservices-reference-implementation/blob/master/charts/package/templates/package-deploy.yaml).</span><span class="sxs-lookup"><span data-stu-id="f4591-238">&#11162; See the [source file](https://github.com/mspnp/microservices-reference-implementation/blob/master/charts/package/templates/package-deploy.yaml).</span></span>

<span data-ttu-id="f4591-239">Вы увидите, что имя развертывания, метки и контейнер по спецификациям все параметры шаблона использования, которые предоставляются во время развертывания.</span><span class="sxs-lookup"><span data-stu-id="f4591-239">You can see that the deployment name, labels, and container spec all use template parameters, which are provided at deployment time.</span></span> <span data-ttu-id="f4591-240">Например из командной строки:</span><span class="sxs-lookup"><span data-stu-id="f4591-240">For example, from the command line:</span></span>

```bash
helm install $HELM_CHARTS/package/ \
     --set image.tag=0.1.0 \
     --set image.repository=package \
     --set dockerregistry=$ACR_SERVER \
     --namespace backend \
     --name package-v0.1.0
```

<span data-ttu-id="f4591-241">Несмотря на то, что ваш конвейер CI/CD можно установить диаграммы непосредственно в Kubernetes, рекомендуется создать в архиве диаграммы (TGZ-файла) и помещает диаграммы в репозиторий Helm, такие как реестр контейнеров Azure.</span><span class="sxs-lookup"><span data-stu-id="f4591-241">Although your CI/CD pipeline could install a chart directly to Kubernetes, we recommend creating a chart archive (.tgz file) and pushing the chart to a Helm repository such as Azure Container Registry.</span></span> <span data-ttu-id="f4591-242">Дополнительные сведения см. в разделе [приложения на основе Docker пакета в чарты Helm в Azure конвейерах](/azure/devops/pipelines/languages/helm?view=azure-devops).</span><span class="sxs-lookup"><span data-stu-id="f4591-242">For more information, see [Package Docker-based apps in Helm charts in Azure Pipelines](/azure/devops/pipelines/languages/helm?view=azure-devops).</span></span>

<span data-ttu-id="f4591-243">Рассмотрите возможность развертывания Helm в своем собственном пространстве имен и с помощью управления доступом на основе ролей (RBAC) для ограничения пространства имен которого его можно развернуть.</span><span class="sxs-lookup"><span data-stu-id="f4591-243">Consider deploying Helm to its own namespace and using role-based access control (RBAC) to restrict which namespaces it can deploy to.</span></span> <span data-ttu-id="f4591-244">Дополнительные сведения см. в разделе [управления доступом на основе роли](https://helm.sh/docs/using_helm/#helm-and-role-based-access-control) в документации по Helm.</span><span class="sxs-lookup"><span data-stu-id="f4591-244">For more information, see [Role-based Access Control](https://helm.sh/docs/using_helm/#helm-and-role-based-access-control) in the Helm documentation.</span></span>

### <a name="revisions"></a><span data-ttu-id="f4591-245">Исправления</span><span class="sxs-lookup"><span data-stu-id="f4591-245">Revisions</span></span>

<span data-ttu-id="f4591-246">Диаграммы helm всегда имеют номер версии, который необходимо использовать [семантического управления версиями](https://semver.org/).</span><span class="sxs-lookup"><span data-stu-id="f4591-246">Helm charts always have a version number, which must use [semantic versioning](https://semver.org/).</span></span> <span data-ttu-id="f4591-247">Диаграмма может также иметь `appVersion`.</span><span class="sxs-lookup"><span data-stu-id="f4591-247">A chart can also have an `appVersion`.</span></span> <span data-ttu-id="f4591-248">Это поле является необязательным и не должен быть связан с версии диаграммы.</span><span class="sxs-lookup"><span data-stu-id="f4591-248">This field is optional, and doesn't have to be related to the chart version.</span></span> <span data-ttu-id="f4591-249">Некоторые команды может возникнуть необходимость версии приложения отдельно от обновлений к диаграммам.</span><span class="sxs-lookup"><span data-stu-id="f4591-249">Some teams might want to application versions separately from updates to the charts.</span></span> <span data-ttu-id="f4591-250">Но более простой подход использовать один номер версии, поэтому отношение 1:1 между диаграммы версии и версии приложения.</span><span class="sxs-lookup"><span data-stu-id="f4591-250">But a simpler approach is to use one version number, so there's a 1:1 relation between chart version and application version.</span></span> <span data-ttu-id="f4591-251">Таким образом, можно хранить одну диаграмму по выпускам продукта и легко развернуть нужного выпуска:</span><span class="sxs-lookup"><span data-stu-id="f4591-251">That way, you can store one chart per release and easily deploy the desired release:</span></span>

```bash
helm install <package-chart-name> --version <desiredVersion>
```

<span data-ttu-id="f4591-252">Рекомендуется также для предоставления аннотацию причина изменения в шаблон развертывания:</span><span class="sxs-lookup"><span data-stu-id="f4591-252">Another good practice is to provide a change-cause annotation in the deployment template:</span></span>

```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: {{ include "delivery.fullname" . | replace "." "" }}
  labels:
     ...
  annotations:
    kubernetes.io/change-cause: {{ .Values.reason }}
```

<span data-ttu-id="f4591-253">Это позволяет просматривать поле Причина изменений для каждой версии с помощью `kubectl rollout history` команды.</span><span class="sxs-lookup"><span data-stu-id="f4591-253">This lets you view the change-cause field for each revision, using the `kubectl rollout history` command.</span></span> <span data-ttu-id="f4591-254">В предыдущем примере причину изменения предоставляется как параметр диаграмма Helm.</span><span class="sxs-lookup"><span data-stu-id="f4591-254">In the previous example, the change-cause is provided as a Helm chart parameter.</span></span>

```bash
$ kubectl rollout history deployments/delivery-v010 -n backend
deployment.extensions/delivery-v010
REVISION  CHANGE-CAUSE
1         Initial deployment
```

<span data-ttu-id="f4591-255">Можно также использовать `helm list` команду, чтобы просмотреть журнал изменений:</span><span class="sxs-lookup"><span data-stu-id="f4591-255">You can also use the `helm list` command to view the revision history:</span></span>

```bash
~$ helm list
NAME            REVISION    UPDATED                     STATUS        CHART            APP VERSION     NAMESPACE
delivery-v0.1.0 1           Sun Apr  7 00:25:30 2019    DEPLOYED      delivery-v0.1.0  v0.1.0          backend
```

> [!TIP]
> <span data-ttu-id="f4591-256">Используйте `--history-max` флаг при инициализации Helm.</span><span class="sxs-lookup"><span data-stu-id="f4591-256">Use the `--history-max` flag when initializing Helm.</span></span> <span data-ttu-id="f4591-257">Этот параметр ограничивает число исправлений, которые Tiller сохраняет в журнале.</span><span class="sxs-lookup"><span data-stu-id="f4591-257">This setting limits the number of revisions that Tiller saves in its history.</span></span> <span data-ttu-id="f4591-258">Tiller сохраняет журнал редакций в configmaps.</span><span class="sxs-lookup"><span data-stu-id="f4591-258">Tiller stores revision history in configmaps.</span></span> <span data-ttu-id="f4591-259">Если часто выпускаем обновления, если не ограничить размер журнала configmaps может стать очень большим.</span><span class="sxs-lookup"><span data-stu-id="f4591-259">If you're releasing updates frequently, the configmaps can grow very large unless you limit the history size.</span></span>

## <a name="azure-devops-pipeline"></a><span data-ttu-id="f4591-260">Azure конвейера DevOps</span><span class="sxs-lookup"><span data-stu-id="f4591-260">Azure DevOps Pipeline</span></span>

<span data-ttu-id="f4591-261">В конвейерах Azure делятся на конвейеры *создавать конвейеры* и *конвейеры выпуска*.</span><span class="sxs-lookup"><span data-stu-id="f4591-261">In Azure Pipelines, pipelines are divided into *build pipelines* and *release pipelines*.</span></span> <span data-ttu-id="f4591-262">Конвейер сборки выполняется процесс непрерывной Интеграции и создает артефакты сборки.</span><span class="sxs-lookup"><span data-stu-id="f4591-262">The build pipeline runs the CI process and creates build artifacts.</span></span> <span data-ttu-id="f4591-263">Для архитектуры микрослужб в Kubernetes эти артефакты — это образы контейнеров и чарты Helm, которые определяют каждой микрослужбы.</span><span class="sxs-lookup"><span data-stu-id="f4591-263">For a microservices architecture on Kubernetes, these artifacts are the container images and Helm charts that define each microservice.</span></span> <span data-ttu-id="f4591-264">Этот процесс компакт-диска, который развертывает микрослужбы в кластере выполнение конвейера выпуска.</span><span class="sxs-lookup"><span data-stu-id="f4591-264">The release pipeline runs that CD process that deploys a microservice into a cluster.</span></span>

<span data-ttu-id="f4591-265">Исходя из потока непрерывной Интеграции, описанные ранее в этой статье, конвейер сборки может состоять из следующих задач:</span><span class="sxs-lookup"><span data-stu-id="f4591-265">Based on the CI flow described earlier in this article, a build pipeline might consist of the following tasks:</span></span>

1. <span data-ttu-id="f4591-266">Создание контейнера средство выполнения тестов.</span><span class="sxs-lookup"><span data-stu-id="f4591-266">Build the test runner container.</span></span>

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        arguments: '--pull --target testrunner'
        dockerFile: $(System.DefaultWorkingDirectory)/$(dockerFileName)
        imageName: '$(imageName)-test'
    ```

1. <span data-ttu-id="f4591-267">Выполните тесты, путем вызова в контейнере средство выполнения тестов "run" docker.</span><span class="sxs-lookup"><span data-stu-id="f4591-267">Run the tests, by invoking docker run against the test runner container.</span></span>

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        command: 'run'
        containerName: testrunner
        volumes: '$(System.DefaultWorkingDirectory)/TestResults:/app/tests/TestResults'
        imageName: '$(imageName)-test'
        runInBackground: false
    ```

1. <span data-ttu-id="f4591-268">Публиковать результаты тестов.</span><span class="sxs-lookup"><span data-stu-id="f4591-268">Publish the test results.</span></span> <span data-ttu-id="f4591-269">См. в разделе [интеграции сборки и задачи тестирования](/azure/devops/pipelines/languages/docker?view=azure-devops&tabs=yaml#integrate-build-and-test-tasks).</span><span class="sxs-lookup"><span data-stu-id="f4591-269">See [Integrate build and test tasks](/azure/devops/pipelines/languages/docker?view=azure-devops&tabs=yaml#integrate-build-and-test-tasks).</span></span>

    ```yaml
    - task: PublishTestResults@2
      inputs:
        testResultsFormat: 'VSTest'
        testResultsFiles: 'TestResults/*.trx'
        searchFolder: '$(System.DefaultWorkingDirectory)'
        publishRunAttachments: true
    ```

1. <span data-ttu-id="f4591-270">Сборки среды выполнения контейнера.</span><span class="sxs-lookup"><span data-stu-id="f4591-270">Build the runtime container.</span></span>

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        dockerFile: $(System.DefaultWorkingDirectory)/$(dockerFileName)
        includeLatestTag: false
        imageName: '$(imageName)'
    ```

1. <span data-ttu-id="f4591-271">Отправьте контейнер для реестра контейнеров Azure (или других реестра контейнеров).</span><span class="sxs-lookup"><span data-stu-id="f4591-271">Push to the container to Azure Container Registry (or other container registry).</span></span>

    ```yaml
    - task: Docker@1
      inputs:
        azureSubscriptionEndpoint: $(AzureSubscription)
        azureContainerRegistry: $(AzureContainerRegistry)
        command: 'Push an image'
        imageName: '$(imageName)'
        includeSourceTags: false
    ```

1. <span data-ttu-id="f4591-272">Пакет чарта Helm.</span><span class="sxs-lookup"><span data-stu-id="f4591-272">Package the Helm chart.</span></span>

    ```yaml
    - task: HelmDeploy@0
      inputs:
        command: package
        chartPath: $(chartPath)
        chartVersion: $(Build.SourceBranchName)
        arguments: '--app-version $(Build.SourceBranchName)'
    ```

1. <span data-ttu-id="f4591-273">Отправьте пакет Helm в реестр контейнеров Azure (или другой репозиторий Helm).</span><span class="sxs-lookup"><span data-stu-id="f4591-273">Push the Helm package to Azure Container Registry (or other Helm repository).</span></span>

    ```yaml
    task: AzureCLI@1
      inputs:
        azureSubscription: $(AzureSubscription)
        scriptLocation: inlineScript
        inlineScript: |
        az acr helm push $(System.ArtifactsDirectory)/$(repositoryName)-$(Build.SourceBranchName).tgz --name $(AzureContainerRegistry);
    ```

<span data-ttu-id="f4591-274">&#11162;См. в разделе [исходный файл](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-ci.yml).</span><span class="sxs-lookup"><span data-stu-id="f4591-274">&#11162; See the [source file](https://github.com/mspnp/microservices-reference-implementation/blob/master/src/shipping/delivery/azure-pipelines-ci.yml).</span></span>

<span data-ttu-id="f4591-275">Выходные данные из конвейера непрерывной Интеграции — это образ контейнера готовый к выпуску и обновленного чарта Helm для микрослужбы.</span><span class="sxs-lookup"><span data-stu-id="f4591-275">The output from the CI pipeline is a production-ready container image and an updated Helm chart for the microservice.</span></span> <span data-ttu-id="f4591-276">На этом этапе конвейера выпуска можно перехватить.</span><span class="sxs-lookup"><span data-stu-id="f4591-276">At this point, the release pipeline can take over.</span></span> <span data-ttu-id="f4591-277">Он выполняет следующие действия:</span><span class="sxs-lookup"><span data-stu-id="f4591-277">It performs the following steps:</span></span>

- <span data-ttu-id="f4591-278">Развертывание в средах разработки, контроля Качества и промежуточной.</span><span class="sxs-lookup"><span data-stu-id="f4591-278">Deploy to dev/QA/staging environments.</span></span>
- <span data-ttu-id="f4591-279">Дождитесь утверждающему утвердить или отклонить развертывание.</span><span class="sxs-lookup"><span data-stu-id="f4591-279">Wait for an approver to approve or reject the deployment.</span></span>
- <span data-ttu-id="f4591-280">Повторной образ контейнера для выпуска</span><span class="sxs-lookup"><span data-stu-id="f4591-280">Retag the container image for release</span></span>
- <span data-ttu-id="f4591-281">Передача тегом выпуска в реестр контейнеров.</span><span class="sxs-lookup"><span data-stu-id="f4591-281">Push the release tag to the container registry.</span></span>
- <span data-ttu-id="f4591-282">Обновите диаграмму Helm в рабочем кластере.</span><span class="sxs-lookup"><span data-stu-id="f4591-282">Upgrade the Helm chart in the production cluster.</span></span>

<span data-ttu-id="f4591-283">Дополнительные сведения о создании конвейера выпуска, см. в разделе [конвейеры выпуска черновик выпусков и вариантов выпуска](/azure/devops/pipelines/release/?view=azure-devops).</span><span class="sxs-lookup"><span data-stu-id="f4591-283">For more information about creating a release pipeline, see [Release pipelines, draft releases, and release options](/azure/devops/pipelines/release/?view=azure-devops).</span></span>

<span data-ttu-id="f4591-284">В примере ниже показан процесс Непрерывной интеграции и end-to-end, описанный в этой статье:</span><span class="sxs-lookup"><span data-stu-id="f4591-284">The following diagram shows the end-to-end CI/CD process described in this article:</span></span>

![CD/Непрерывной поставки](./images/aks-cicd-flow.png)

## <a name="next-steps"></a><span data-ttu-id="f4591-286">Дальнейшие действия</span><span class="sxs-lookup"><span data-stu-id="f4591-286">Next steps</span></span>

<span data-ttu-id="f4591-287">В этой статье был основан на реализации, которую можно найти на [GitHub][ri].</span><span class="sxs-lookup"><span data-stu-id="f4591-287">This article was based on a reference implementation that you can find on [GitHub][ri].</span></span>

[ri]: https://github.com/mspnp/microservices-reference-implementation