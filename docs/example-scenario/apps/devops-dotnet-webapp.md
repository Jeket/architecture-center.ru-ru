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
# <a name="design-a-cicd-pipeline-using-azure-devops"></a>Разработка конвейера CI/CD с использованием Azure DevOps

В этом сценарии предоставляется руководство по архитектуре и разработке для создания конвейера непрерывной интеграции (CI) и непрерывного развертывания (CD). В этом примере конвейер CI/CD развертывает двухуровневое веб-приложение .NET в Службе приложений Azure.

Переход на современные процессы CI/CD предоставляет множество преимуществ для создания, развертывания, тестирования и мониторинга приложения. С помощью Azure DevOps вместе с такими службами, как Служба приложений, организации могут уделить больше внимания разработке приложений, а не управлению поддержкой инфраструктуры.

## <a name="relevant-use-cases"></a>Варианты соответствующего использования

Рассмотрите использование Azure DevOps и процессов CI/CD для:

- Ускорение разработки приложений и жизненных циклов развертывания.
- Внедрение механизмов проверки качества и согласованности в автоматизированный процесс сборки и выпуска.
- повышения стабильности и времени доступности приложения.

## <a name="architecture"></a>Архитектура

![Схема архитектуры компонентов Azure, участвующих в сценарии DevOps, с использованием Azure DevOps и Службы приложений Azure][architecture]

Данные передаются в сценарии следующим образом:

1. Разработчик изменяет исходный код приложения.
2. Код приложения вместе с файлом конфигурации web.config фиксируется в репозитории исходного кода в Azure Repos.
3. Непрерывная интеграция активирует сборку приложения и модульные тесты, используя Azure Test Plans.
4. Непрерывное развертывание в Azure Pipelines активирует автоматическое развертывание артефактов приложения *со значениями конфигурации для конкретной среды*.
5. Артефакты развертываются в Службу приложений Azure.
6. Служба Azure Application Insights собирает и анализирует данные о работоспособности, производительности и использовании ресурсов.
7. Разработчики отслеживают работоспособность, производительность и сведения об использовании и управляют этими аспектами.
8. Информация о невыполненной работе используется для назначения приоритетов новым функциям и исправлениям ошибок с помощью Azure Boards.

### <a name="components"></a>Компоненты

- [Azure DevOps][vsts] — это служба, которая позволяет управлять жизненным циклом разработки от начала и до конца &mdash; начиная с планирования и управления проектом до управления кодом, сборки и выпуска.

- [Веб-приложения Azure][web-apps] — это служба для размещения веб-приложений, интерфейсов REST API и серверной части мобильных решений. Хотя в этой статье основное внимание уделяется .NET, есть несколько дополнительных поддерживаемых вариантов платформ разработки.

- [Application Insights][application-insights] — это расширяемая служба управления производительностью приложений (APM), получающая данные напрямую из источника и предназначенная для веб-разработчиков на нескольких платформах.

### <a name="alternatives"></a>Альтернативные варианты

Хотя эта статья посвящена Azure DevOps, [Azure DevOps Server][azure-devops-server] (ранее известный как Team Foundation Server) может использоваться в качестве локального средства для замены. Кроме того, также можно использовать набор технологий для разработки конвейера с открытым исходным кодом с помощью [Jenkins][jenkins-on-azure].

В контексте инфраструктуры как кода [шаблоны Resource Manager][arm-templates] использовались как часть проекта Azure DevOps, но вы можете использовать и другие технологии управления, например [Terraform][terraform] или [Chef][chef]. Если вы предпочитаете развертывание по схеме "инфраструктура как услуга" (IaaS) и вам требуется управления конфигурацией, можно использовать службу [Настройка состояния службы автоматизации Azure][desired-state-configuration], [Ansible][ansible] или [Chef][chef].

Можно использовать следующие альтернативы для размещения веб-приложений Azure.

- [Виртуальные машины Azure][compare-vm-hosting] справляются с рабочими нагрузками, требующими высокой степени управляемости или зависящими от компонентов операционной системы и служб, что невозможно реализовать в веб-приложениях (например, Windows GAC или модель COM).

- [Service Fabric][service-fabric] — это хороший вариант, если архитектура рабочей нагрузки предназначена для распределенных компонентов, которые получают преимущества от развертывания и выполнения в кластере с высокой степенью управления. Service Fabric также может использоваться для размещения контейнеров.

- [Функции Azure][azure-functions] предоставляют эффективный бессерверный подход, если архитектура рабочей нагрузки сосредоточена на распределенных компонентах с высокой степенью детализации, требующих минимальных зависимостей, в которых отдельные компоненты должны выполняться только по требованию (не непрерывно), а оркестрация компонентов не требуется.

Это [дерево принятия решений для вычислительных служб Azure](/azure/architecture/guide/technology-choices/compute-decision-tree) может помочь при выборе правильного пути для миграции.

## <a name="management-and-security-considerations"></a>Управление и безопасность

- Рассмотрите возможность использования одной из [задач разметки][vsts-tokenization], которые доступны в VSTS marketplace.

- Задачи [Azure Key Vault][download-keyvault-secrets] могут загружать секреты из Azure Key Vault в ваш выпуск. Затем эти секреты можно использовать в качестве переменных в своем определении выпуска, что позволит избежать их хранения в системе управления версиями.

- Используйте [переменные выпуска][vsts-release-variables] в определениях выпуска для изменения конфигурации среды. Переменные выпуска могут охватывать как весь выпуск, так и конкретную среду. При использовании переменных для секретной информации убедитесь, что выбран значок с изображением замка.

- Следует использовать [шлюзы развертывания][vsts-deployment-gates] в конвейере выпуска. Это позволяет использовать данные мониторинга во внешних системах (например системах управления инцидентами или дополнительных пользовательских системах), чтобы определить, следует ли повысить уровень выпуска.

- Если требуется вмешательство в конвейер выпуска, используйте функциональность [утверждений][vsts-approvals].

- При выпуске конвейера рассмотрите возможность использования [Application Insights][application-insights] и дополнительных средств мониторинга как можно раньше. Множество организаций начинают проводить мониторинг только в своей рабочей среде. Отслеживая другие среды, вы можете определять ошибки на ранних этапах разработки и избегать проблем в рабочей среде.

## <a name="deploy-the-scenario"></a>Развертывание сценария

### <a name="prerequisites"></a>Технические условия

- Необходимо иметь учетную запись Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

- Вам нужно зарегистрироваться в организации Azure DevOps. Дополнительные сведения см. в [кратком руководстве по созданию собственной организации][vsts-account-create].

### <a name="walk-through"></a>Пошаговое руководство

[Проекты Azure DevOps](/azure/devops-project/azure-devops-project-github) будет развернуть план службы приложений службы приложений и ресурса App Insights для вас, а также настроить конвейер конвейеры Azure для вас.

После того как вы настроили конвейер с помощью проектов Azure DevOps и сборка завершена, просмотрите связанные изменения кода, рабочие элементы и результаты тестирования. Вы заметите, что результаты теста не отображаются, так как в коде нет никаких тестов для выполнения.

Конвейер создает определение выпуска и триггера непрерывного развертывания, развертывание наше приложение в среде разработки. В рамках процесса непрерывного развертывания можно увидеть разброс выпусков в нескольких средах. Выпуск может охватывать обе инфраструктуры (используя такие методы, как инфраструктура как код), а также развертывать необходимые пакеты приложений и все выполняемые после настройки задачи.

## <a name="pricing"></a>Цены

Расчет стоимости Azure DevOps будет зависеть от количества пользователей в организации, которым требуется доступ, в дополнение к таким факторам, как количество параллельных требуемых выпусков и сборок, а также число тестовых пользователей. Дополнительные сведения см. [Цены на Azure DevOps][vsts-pricing-page].

[Калькулятор цен][vsts-pricing-calculator] предоставляет оценку запуска Azure DevOps с 20 пользователями.

Azure DevOps оплачивается в расчете на каждого пользователя за месяц. В дополнение к оплате за дополнительных тестовых пользователей или базовые пользовательские лицензии может взиматься плата в зависимости от требуемых параллельных конвейеров.

## <a name="related-resources"></a>Связанные ресурсы

Для получения дополнительных сведений о CI/CD и Azure DevOps ознакомьтесь со следующими ресурсами.

- [What is DevOps?][devops-whatis] (Что такое DevOps?)
- [DevOps в Майкрософт. Работа с Azure DevOps][devops-microsoft]
- [Пошаговые руководства по использованию DevOps вместе с Azure DevOps][devops-with-vsts]
- [Контрольный список DevOps][devops-checklist]
- [Создание конвейера CI/CD для .NET с помощью проектов Azure DevOps][devops-project-create]

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
