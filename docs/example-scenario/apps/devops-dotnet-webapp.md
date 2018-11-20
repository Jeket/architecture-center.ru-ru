---
title: Использование конвейера CI/CD с помощью Azure DevOps
description: Создание и выпуск приложения .NET в службе "Веб-приложения Azure" с помощью Azure DevOps.
author: christianreddington
ms.date: 07/11/18
ms.openlocfilehash: 97f16b2d3d9c15bc6f5db6fad4c9d8097243ad3d
ms.sourcegitcommit: 0a31fad9b68d54e2858314ca5fe6cba6c6b95ae4
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/13/2018
ms.locfileid: "51610793"
---
# <a name="cicd-pipeline-with-azure-devops"></a>Использование конвейера CI/CD с помощью Azure DevOps

Технология DevOps предусматривает интеграцию функций разработки, контроля качества и ИТ-операций. DevOps требует единой среды и набора параметров для предоставления программного обеспечения.

В этом примере сценария показано, как команды разработчиков могут использовать Azure DevOps для развертывания двухуровневого веб-приложения .NET в Службе приложений Azure. Такое веб-приложение будет зависеть от подчиненных служб платформы Azure (PaaS). В этом документе также указываются важные особенности, которые нужно отметить при разработке такого сценария с помощью PaaS Azure.

Внедрение современного подхода к разработке приложений на базе непрерывной интеграции (CI) и непрерывного развертывания (CD) ускоряет создание решений для пользователей благодаря использованию надежной службы, объединяющей функции сборки, тестирования, развертывания и мониторинга. С помощью платформы, такой как служба Azure DevOps, в дополнение к таким службам Azure, как Служба приложений, организации могут уделить больше внимания разработке сценария, а не управлению инфраструктурой для его реализации.

## <a name="relevant-use-cases"></a>Варианты соответствующего использования

Применяйте DevOps в следующих вариантах использования:

* Ускорение разработки приложений и жизненных циклов развертывания.
* Внедрение механизмов проверки качества и согласованности в автоматизированный процесс сборки и выпуска.

## <a name="architecture"></a>Архитектура

![Обзор архитектуры компонентов Azure, участвующих в сценарии DevOps, с помощью Azure DevOps и Службы приложений Azure][architecture]

Этот сценарий берет во внимание конвейер CI/CD для веб-приложения .NET с помощью Azure DevOps. Данные передаются в сценарии следующим образом:

1. Изменение исходного кода приложения.
2. Фиксация кода приложения и файла web.config веб-приложений.
3. Непрерывная интеграция активирует сборку приложений и модульные тесты.
4. Триггер непрерывного развертывания управляет развертыванием артефактов приложения с использованием *параметризованных значений конфигурации для конкретной среды*.
5. Развертывание в Службе приложений Azure.
6. Служба Azure Application Insights собирает и анализирует данные о работоспособности, производительности и использовании ресурсов.
7. Просмотр сведений о работоспособности, производительности и использовании ресурсов.

### <a name="components"></a>Компоненты

* [Azure DevOps][vsts] — это служба, которая позволяет управлять жизненным циклом разработки от начала и до конца &mdash; начиная с планирования и управления проектом до управления кодом, сборки и выпуска.
* [Веб-приложения Azure][web-apps] — это служба для размещения веб-приложений, интерфейсов REST API и серверной части мобильных решений. Хотя в этой статье основное внимание уделяется .NET, есть несколько дополнительных поддерживаемых вариантов платформ разработки.
* [Application Insights][application-insights] — это расширяемая служба управления производительностью приложений (APM), получающая данные напрямую из источника и предназначенная для веб-разработчиков на нескольких платформах.

### <a name="alternative-devops-tooling-options"></a>Альтернативные варианты инструментария DevOps

Хотя эта статья посвящена Azure DevOps, [Team Foundation Server][team-foundation-server] может использоваться в качестве локального средства для замены. Кроме того, также можно использовать набор технологий для конвейера разработки с открытым исходным кодом с помощью [Jenkins][jenkins-on-azure].

В контексте инфраструктуры как кода [шаблоны Azure Resource Manager][arm-templates] являются частью проекта Azure DevOps, но вы также можете использовать [Terraform][terraform] или [Chef][chef]. Если вы предпочитаете развертывание по схеме "инфраструктура как услуга" (IaaS) и вам требуется управления конфигурацией, можно использовать службу [Настройка состояния службы автоматизации Azure][desired-state-configuration], [Ansible][ansible] или [Chef][chef].

### <a name="alternatives-to-azure-web-apps"></a>Альтернативы веб-приложений Azure

Можно использовать следующие альтернативы для размещения веб-приложений Azure.

* [Виртуальные машины Azure][compare-vm-hosting] &mdash; — используется для рабочих нагрузок, требующих высокой степени управляемости или, зависимых от компонентов, операционной системы или служб, что невозможно реализовать в веб-приложениях Azure (например Windows GAC или COM).
* [Service Fabric][service-fabric] &mdash;— это хороший вариант, если архитектура рабочей нагрузки предназначена для распределенных компонентов, которые получают преимущества от развертывания и выполнения в кластере с высокой степенью управления. Service Fabric также может использоваться для размещения контейнеров.
* [Функции Azure][azure-functions] — хороший вариант, если архитектура рабочей нагрузки сосредоточена на распределенных компонентах с высокой степенью детализации, требующих минимальных зависимостей, в которых отдельные компоненты должны выполняться только по требованию (не непрерывно), а оркестрация компонентов не требуется.

Это [дерево принятия решений](/azure/architecture/guide/technology-choices/compute-decision-tree) может помочь при выборе правильного пути для миграции.

### <a name="devops"></a>DevOps

**[Непрерывная интеграция (CI)][continuous-integration]** поддерживает стабильную сборку, при этом несколько разработчиков регулярно совершают небольшие частые изменения в общей кодовой базе. В рамках конвейера непрерывной интеграции необходимо следующее:
* Часто фиксировать изменения кода меньшего размера. Избегать дозирования больших или более сложных изменений, которые труднее всего успешно объединить.
* Выполнить модульный тест компонентов приложения с достаточным объемом протестированного кода (включая пути с недостаточным качеством передачи данных).
* Обеспечить выполнение сборки в общей основной (или внешней) ветви. Эта ветвь должна работать стабильно и сохраняться как готовая к развертыванию. Неполные или выполняемые изменения следует изолировать в отдельной ветви с частыми объединениями прямой интеграции, чтобы избежать конфликтов позже.

**[Непрерывная поставка (CD)][continuous-delivery]** должна демонстрировать не только стабильность сборки, но и стабильность развертывания. Это затрудняет реализацию CD, так как требуется конфигурация, предназначенная для конкретной среды, и механизм правильной установки их значений. Ниже приведены дополнительные рекомендации для CD.
* Нужно провести достаточно тестов интеграции, чтобы обеспечить конфигурацию различных компонентов и их корректное взаимодействие.
* CD может потребовать настройку и сброс данных конкретной среды и управление версиями схемы базы данных.
* Непрерывная поставка может также расширится в среду нагрузочного тестирования и пользовательского приемочного тестирования.
* Непрерывная поставка получает преимущество от непрерывного мониторинга, в идеале — во всех средах.
* Согласованность и надежность развертывания и тестирования в средах интеграции упрощается с помощью сценариев создания и конфигурации инфраструктуры размещения. Это весьма легко для облачных рабочих нагрузок. Дополнительные сведения см. в разделе [Инфраструктура как код][infra-as-code].
* Начните непрерывную доставку как можно раньше в жизненном цикле проекта. Чем позже вы это сделаете, тем труднее будет.
* Интеграция и модульные тесты должны иметь такой же приоритет, как и функции проекта.
* Используйте независимые от среды пакеты развертывания и управляйте зависящими от среды конфигурациями в ходе выпуска.
* Защищайте конфиденциальные конфигурации в инструментарии управления выпусками или путем вызова HSM или [Azure Key Vault][azure-key-vault] во время процесса выпуска. Не храните конфиденциальные данные в системе управления версиями.

**Непрерывное обучение**. Самый эффективный мониторинг среды CD обеспечивается средствами наблюдения за производительностью приложений (APM для краткости), например [Application Insights][application-insights]. Достаточная глубина мониторинга рабочей нагрузки приложения имеет решающее значение для понимания ошибок и производительности под нагрузкой. Чтобы включить [непрерывный мониторинг конвейера CD][app-insights-cd-monitoring], Application Insights можно интегрировать с VSTS. Это можно использовать для автоматического перехода на следующий этап без вмешательства человека или для отката при обнаружении предупреждения.

## <a name="considerations"></a>Рекомендации

### <a name="availability"></a>Доступность

Рассмотрите возможность использования [распространенных конструктивных шаблонов для обеспечения доступности][design-patterns-availability] при создании облачного приложения.

Ознакомьтесь с вопросами доступности в соответствующем [описании эталонной архитектуры для веб-приложения в службе приложений][app-service-reference-architecture].

Дополнительные сведения по другим вопросам доступности см. в статье [с контрольным списком для обеспечения доступности][availability] в Центре архитектуры Azure.

### <a name="scalability"></a>Масштабируемость

При создании облачного приложения следует учитывать [распространенные конструктивные шаблоны для обеспечения масштабируемости][design-patterns-scalability].

Ознакомьтесь с вопросами масштабируемости в соответствующем [описании эталонной архитектуры для веб-приложения в службе приложений][app-service-reference-architecture].

Для других вопросов масштабируемости, см. раздел [Контрольный список для обеспечения масштабируемости][scalability] в центре архитектуры Azure.

### <a name="security"></a>Безопасность

Рассмотрите использование [распространенных конструктивных шаблонов для обеспечения безопасности][design-patterns-security] там, где уместно.

Ознакомьтесь с вопросами безопасности в соответствующем [описании эталонной архитектуры для веб-приложения в службе приложений][app-service-reference-architecture].

Общие рекомендации по разработке безопасных решений см. в разделе [Azure Security Documentation][security].

### <a name="resiliency"></a>Устойчивость

При необходимости рассмотрите возможность реализации [распространенных конструктивных шаблонов для обеспечения устойчивости][design-patterns-resiliency].

[Рекомендации по работе со Службой приложений Azure][resiliency-app-service] см. в Центре архитектуры Azure.

Общее руководство по проектированию устойчивых решений см. в разделе [Проектирование устойчивых приложений для Azure][resiliency].

## <a name="deploy-the-scenario"></a>Развертывание сценария

### <a name="prerequisites"></a>Предварительные требования

* Необходимо иметь учетную запись Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись][azure-free-account], прежде чем начинать работу.
* Вам нужно зарегистрироваться в организации Azure DevOps. Дополнительные сведения см. в [этом кратком руководстве][vsts-account-create].

### <a name="walk-through"></a>Пошаговое руководство

В этом сценарии для создания конвейера CI/CD используется проект Azure DevOps.

Проект развертывает план Службы приложений Azure, Службу приложений Azure и ресурс App Insights, а также настраивает проект Azure DevOps.

Когда вы развернете проект Azure DevOps и завершите сборку, просмотрите связанные изменения кода, рабочие элементы и результаты тестов. Вы заметите, что результаты теста не отображаются, так как в коде нет никаких тестов для выполнения.

Просмотрите определения выпуска. Обратите внимание, что был настроен конвейер выпуска, который выпустил наше приложение в среду разработки. Обратите внимание на наличие **триггера непрерывного развертывания**, установленного из артефакта сборки **Drop** с помощью автоматических выпусков в средах разработки. В рамках процесса непрерывного развертывания можно увидеть разброс выпусков в нескольких средах. Выпуск может охватывать обе инфраструктуры (используя такие методы, как инфраструктура как код), а также развертывать необходимые пакеты приложений и все выполняемые после настройки задачи.

## <a name="additional-considerations"></a>Дополнительные замечания

* Рассмотрите возможность использования одной из [задач разметки][vsts-tokenization], которые доступны в VSTS marketplace.
* Рассмотрите возможность использования задачи VSTS [Развертывание: Azure Key Vault][download-keyvault-secrets], чтобы скачать секреты из Azure Key Vault в свой выпуск. Затем эти секреты можно использовать в качестве переменных для своего определения выпуска. Их можно не хранить в системе управления версиями.
* Рассмотрите возможность использования [переменных выпуска][vsts-release-variables] в определениях выпуска для изменения конфигурации среды. Переменные выпуска могут охватывать как весь выпуск, так и конкретную среду. При использовании переменных для секретной информации убедитесь, что выбран значок с изображением замка.
* Рассмотрите возможность использования [шлюзов развертывания][vsts-deployment-gates] в конвейере выпуска. Это позволяет использовать данные мониторинга во внешних системах (например системах управления инцидентами или дополнительных пользовательских системах), чтобы определить, следует ли повысить уровень выпуска.
* Когда требуется вмешательство в конвейер выпуска, рассмотрите возможность использования функции [утверждений][vsts-approvals].
* При выпуске конвейера рассмотрите возможность использования [Application Insights][application-insights] и дополнительных средств мониторинга как можно раньше. Многие организации только начинают мониторинг в своей производственной среде. Отслеживая другие среды, можно определить ошибки ранее в процессе разработки и избежать проблем в вашей рабочей среде.

## <a name="pricing"></a>Цены

Расчет стоимости Azure DevOps будет зависеть от количества пользователей в организации, которым требуется доступ, в дополнение к таким факторам, как количество параллельных требуемых выпусков и сборок, а также число тестовых пользователей. Дополнительные сведения см. [Цены на Azure DevOps][vsts-pricing-page].

* [Azure DevOps][vsts-pricing-calculator] — служба, которая позволяет управлять жизненным циклом разработки. Оплата происходит на основе данных пользователя в месяц. В дополнение к оплате за дополнительных тестовых пользователей или базовые пользовательские лицензии может взиматься плата в зависимости от требуемых параллельных конвейеров.

## <a name="related-resources"></a>Связанные ресурсы

* [What is DevOps?][devops-whatis] (Что такое DevOps?)
* [DevOps в Майкрософт. Работа с Azure DevOps][devops-microsoft]
* [Пошаговые учебники: DevOps с использованием Azure DevOps][devops-with-vsts]
* [Контрольный список DevOps][devops-checklist]
* [Создание конвейера CI/CD для .NET с помощью проекта Azure DevOps][devops-project-create]

<!-- links -->
[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: ../../reference-architectures/app-service-web-app/basic-web-app.md
[azure-free-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/architecture-devops-dotnet-webapp.png
[availability]: /azure/architecture/checklist/availability
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
[resiliency]: /azure/architecture/checklist/resiliency
[scalability]: /azure/architecture/checklist/scalability
[vsts]: /vsts/?view=vsts#pivot=services
[continuous-integration]: /azure/devops/what-is-continuous-integration
[continuous-delivery]: /azure/devops/what-is-continuous-delivery
[web-apps]: /azure/app-service/app-service-web-overview
[terraform]: /azure/terraform/
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
[team-foundation-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]: /azure/service-fabric/
[azure-functions]: /azure/azure-functions/
[azure-containers]: https://azure.microsoft.com/overview/containers/
[compare-vm-hosting]: /azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]: /azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]: /azure/best-practices-availability-paired-regions
[devops-project-create]: /azure/devops-project/azure-devops-project-aspnet-core
[security]: /azure/security/