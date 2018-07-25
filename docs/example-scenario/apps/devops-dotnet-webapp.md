---
title: Конвейер CI/CD c VSTS
description: Пример сборки и запуска приложения .NET в веб-приложениях Azure
author: christianreddington
ms.date: 07/11/18
ms.openlocfilehash: ae4ac5fc02cc841fc39b3cbef46124fe9da75e9b
ms.sourcegitcommit: 71cbef121c40ef36e2d6e3a088cb85c4260599b9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/14/2018
ms.locfileid: "39061015"
---
# <a name="cicd-pipeline-with-vsts"></a>Конвейер CI/CD c VSTS

DevOps — это интеграция операций разработки, контроля качества и ИТ-операций. DevOps требует единой среды и набора параметров для предоставления программного обеспечения.

В этом примере сценария показано, как команды разработчиков могут использовать Visual Studio Team Services для развертывания двухуровневого веб-приложения .NET в Службе приложений Azure. Веб-приложение зависит от подчиненных служб платформы как услуга (PaaS) Azure. В этом документе также указываются важные особенности, которые нужно отметить при разработке такого сценария с помощью платформы как услуги (PaaS) Azure.

Внедрение современного подхода к разработке приложений с помощью непрерывной интеграции (CI) и непрерывного развертывания (CD) поможет вам быстрее предоставлять ценные решения пользователям благодаря надежной сборке, тестированию, развертыванию и мониторингу. С помощью платформы, такой как служба Visual Studio Team Services, в дополнение к таким службам Azure, как Служба приложений, организации могут уделить больше внимания разработке сценария, а не управлению инфраструктурой для его реализации.

## <a name="related-use-cases"></a>Связанные варианты использования

Применяйте DevOps в следующих вариантах использования:

* Ускорение разработки приложений и жизненных циклов развертывания.
* Внедрение механизмов проверки качества и согласованности в автоматизированный процесс сборки и выпуска.

## <a name="architecture"></a>Архитектура

![Обзор архитектуры компонентов Azure, участвующих в сценарии DevOps, с помощью Visual Studio Team Services и Службы приложений Azure][architecture]

В этом сценарии рассматривается конвейер DevOps для веб-приложения .NET с использованием Visual Studio Team Services (VSTS). Данные передаются в сценарии следующим образом:

1. Изменение исходного кода приложения.
2. Фиксация кода приложения и файла web.config веб-приложений.
3. Непрерывная интеграция активирует сборку приложений и модульные тесты.
4. Триггер непрерывного развертывания управляет развертыванием артефактов приложения с использованием *параметризованных значений конфигурации для конкретной среды*.
5. Развертывание в Службе приложений Azure.
6. Служба Azure Application Insights собирает и анализирует данные о работоспособности, производительности и использовании ресурсов.
7. Просмотр сведений о работоспособности, производительности и использовании ресурсов.

### <a name="components"></a>Компоненты

* [Группы ресурсов][resource-groups] — это логические контейнеры для ресурсов Azure, обеспечивающие ограничение управления доступом в плоскости управления. Рассматривайте группу ресурсов как представление единицы развертывания.
* [Visual Studio Team Services (VSTS)][vsts] — это служба, которая позволяет управлять жизненным циклом разработки от начала и до конца, начиная с планирования и управления проектом до управления кодом, сборки и выпуска.
* [Веб-приложения Azure][web-apps] — это служба модели "платформа как услуга" (PaaS) для размещения веб-приложений, интерфейсов REST API и серверной части мобильных решений. Хотя в этой статье основное внимание уделяется .NET, есть несколько дополнительных поддерживаемых вариантов платформ разработки.
* [Application Insights][application-insights] — это расширяемая служба управления производительностью приложений (APM), получающая данные напрямую из источника и предназначенная для веб-разработчиков на нескольких платформах.

### <a name="alternative-devops-tooling-options"></a>Альтернативные варианты инструментария DevOps

Хотя эта статья посвящена Visual Studio Team Services, [Team Foundation Server][team-foundation-server] может использоваться в качестве локального средства для замены. Кроме того, можно также найти набор технологий, используемый вместе с конвейером разработки с открытым кодом [Jenkins][jenkins-on-azure].

С точки зрения инфраструктуры как кода [шаблоны Azure Resource Manager (ARM)][arm-templates] входят в проект Azure DevOps, но можно рассмотреть возможность использования [Terraform][terraform] или [Chef][chef], если вы вложили в них какие-то средства. Если вы предпочитаете развертывания по модели "инфраструктура как услуга" (IaaS) и вам требуется управления конфигурацией, то можно использовать [Azure Desired State Configuration][desired-state-configuration], [Ansible][ansible] или [Chef][chef].

### <a name="alternatives-to-web-app-hosting"></a>Альтернативы размещению в веб-приложениях

Альтернативы размещению в веб-приложениях Azure следующие:

* [Виртуальная машина][compare-vm-hosting] предназначена для рабочих нагрузок, требующих высокого уровня управления, или от компонентов операционной системы или службы, которые невозможно использовать в веб-приложениях (например, Windows GAC или COM).
* [Размещение в контейнере][azure-containers] — если есть зависимости операционной системы и требуется определенный уровень мобильности или плотности при размещении.
* [Service Fabric][service-fabric] — это хороший вариант, если архитектура рабочей нагрузки предназначена для распределенных компонентов, которые получают преимущества от развертывания и выполнения в кластере с высокой степенью управления. Service Fabric также может использоваться для размещения контейнеров.
* [Бессерверные функции Azure][azure-functions] — хороший вариант, если архитектура рабочей нагрузки сосредоточена на распределенных компонентах с высокой степенью детализации, требующих минимальных зависимостей, в которых отдельные компоненты должны выполняться только по требованию (не непрерывно), а оркестрация компонентов не требуется.

### <a name="devops"></a>DevOps

**[Непрерывная интеграция (CI)][continuous-integration]** должна быть направлена ​​на демонстрацию стабильной сборки, при которой несколько отдельных разработчиков или команда постоянно совершают небольшие, но частые изменения в общей базе кода.
В рамках конвейера непрерывной интеграции необходимо следующее:

* Регулярно записывать небольшие фрагменты кода после изменения (избегайте пакетной обработки больших и сложных изменений, так как их будет труднее объединить).
* Выполнить модульный тест компонентов приложения с достаточным объемом протестированного кода (включая пути с недостаточным качеством передачи данных).
* Обеспечить выполнение сборки в общей основной (или внешней) ветви. Эта ветвь должна работать стабильно и сохраняться как готовая к развертыванию. Неполные или выполняемые изменения следует изолировать в отдельной ветви с частыми объединениями прямой интеграции, чтобы избежать конфликтов позже.

**[Непрерывная поставка (CD)][continuous-delivery]** должна демонстрировать не только стабильность сборки, но и стабильность развертывания. Это затрудняет реализацию CD, так как требуется конфигурация, предназначенная для конкретной среды, и механизм правильной установки значений.

Кроме того, нужно провести достаточно тестов интеграции, чтобы обеспечить конфигурацию различных компонентов и их корректное взаимодействие.

Это также может потребовать настройки и сброса данных конкретной среды и управления версиями схемы базы данных.

Непрерывная поставка может также расширится в среду нагрузочного тестирования и пользовательского приемочного тестирования.

Непрерывная поставка получает преимущество от непрерывного мониторинга, в идеале — во всех средах.
Согласованность и надежность развертываний и тестирования интеграции в средах проще обеспечить с помощью сценариев создания и конфигурации или инфраструктуры узла (что значительно проще для облачных рабочих нагрузок; см. раздел "Инфраструктура Azure как код"). Это также называется методом [инфраструктура как код][infra-as-code].

* Начните непрерывную доставку как можно раньше в жизненном цикле проекта. Чем позже вы это сделаете, тем труднее будет.
* Интеграция и модульные тесты должны иметь такой же приоритет, что функции проекта.
* Используйте независимые от среды пакеты развертывания и управляйте зависящими от среды конфигурациями с помощью процесса выпуска.
* Защищайте конфиденциальные конфигурации в инструментарии управления выпусками или путем вызова HSM или [Key Vault][azure-key-vault] во время процесса выпуска. Не храните конфиденциальные данные в системе управления версиями.

**Непрерывное обучение**. Самый эффективный мониторинг среды CD обеспечивается средствами наблюдения за производительностью приложений (APM для краткости), например [Application Insights][application-insights] корпорации Майкрософт. Достаточная глубина мониторинга рабочей нагрузки приложения имеет решающее значение для понимания ошибок и производительности под нагрузкой. [App Insights можно интегрировать с VSTS, чтобы включить непрерывный мониторинг конвейера CD][app-insights-cd-monitoring]. Это можно использовать для автоматического перехода на следующий этап без вмешательства человека или для отката при обнаружении предупреждения.

## <a name="considerations"></a>Рекомендации

### <a name="availability"></a>Доступность

Рассмотрите возможность использования [распространенных конструктивных шаблонов для обеспечения доступности][design-patterns-availability] при создании облачного приложения.

Ознакомьтесь с вопросами доступности в соответствующем [описании эталонной архитектуры для веб-приложения в службе приложений][app-service-reference-architecture].

Дополнительные сведения по другим вопросам масштабируемости см. в разделе [с контрольным списком для обеспечения доступности][availability] в Центре архитектуры Azure.

### <a name="scalability"></a>Масштабируемость

При создании облачного приложения следует учитывать [распространенные конструктивные шаблоны для обеспечения масштабируемости][design-patterns-scalability].

Ознакомьтесь с вопросами масштабируемости в соответствующем [описании эталонной архитектуры для веб-приложения в службе приложений][app-service-reference-architecture].

Другие вопросы по масштабируемости см. в [контрольном списке для обеспечения масштабируемости][scalability] в Центре архитектуры Azure.

### <a name="security"></a>Безопасность

Рассмотрите использование [распространенных конструктивных шаблонов для обеспечения безопасности][design-patterns-security] там, где уместно.

Ознакомьтесь с вопросами безопасности в соответствующем [описании эталонной архитектуры для веб-приложения в службе приложений][app-service-reference-architecture].

Общие рекомендации по разработке безопасных решений см. в разделе [Azure Security Documentation][security].

### <a name="resiliency"></a>Устойчивость

Ознакомьтесь с [распространенными конструктивными шаблонами для обеспечения устойчивости][design-patterns-resiliency] и реализуйте их, если это уместно.

Вы найдете ряд [рекомендаций по обеспечению устойчивости для Службы приложений][resiliency-app-service] в центре архитектуры.

Общее руководство по проектированию устойчивых решений см. в разделе [Проектирование устойчивых приложений для Azure][resiliency].

## <a name="deploy-the-scenario"></a>Развертывание сценария

### <a name="prerequisites"></a>Предварительные требования

* Необходимо иметь учетную запись Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись][azure-free-account], прежде чем начинать работу.
* Необходима учетная запись Visual Studio Team Services (VSTS). Дополнительные сведения о создании учетной записи Visual Studio Team Services (VSTS) см. [в этой статье][vsts-account-create].

### <a name="walk-through"></a>Пошаговое руководство

В этом сценарии для создания конвейера CI/CD используется проект DevOps Azure.

Он развернет план Службы приложений, Службу приложений и ресурс App Insights, а также настроит проект Visual Studio Team Services.

После того как проект DevOps и сборка будут завершены, просмотрите связанные изменения кода, рабочие элементы и результаты тестов. Вы заметите, что результаты теста не отображаются, так как в коде нет никаких тестов для выполнения.

Просмотрите определения выпуска. Обратите внимание, что был настроен конвейер выпуска, который выпустил наше приложение в среду разработки. Обратите внимание на наличие **триггера непрерывного развертывания**, установленного из артефакта сборки **Drop** с помощью автоматических выпусков в средах разработки. В рамках процесса непрерывного развертывания можно увидеть разброс выпусков в нескольких средах. Выпуск может охватывать обе инфраструктуры (используя такие методы, как инфраструктура как код), а также развертывать необходимые пакеты приложений и все выполняемые после настройки задачи.

**Дополнительные замечания**

* Рассмотрите возможность использования одной из [задач разметки][vsts-tokenization], которые доступны в VSTS marketplace.
* Рассмотрите возможность использования задачи VSTS [Развертывание: Azure Key Vault][download-keyvault-secrets], чтобы скачать секреты из Azure Key Vault в свой выпуск. Затем эти секреты можно использовать в качестве переменных для своего определения выпуска. Их не следует хранить в системе управления версиями.
* Рассмотрите возможность использования [переменных выпуска][vsts-release-variables] в определениях выпуска для изменения конфигурации среды. Переменные выпуска могут охватывать как весь выпуск, так и конкретную среду. При использовании переменных для секретной информации убедитесь, что выбран значок с изображением замка.
* Рассмотрите возможность использования [шлюзов развертывания][vsts-deployment-gates] в конвейере выпуска. Это позволяет использовать данные мониторинга для внешних систем (например, системы инцидентов или дополнительные системы, подходящие по требованиям), чтобы определить, следует ли повысить уровень выпуска.
* Когда требуется вмешательство в конвейер выпуска, рассмотрите возможность использования функции [утверждений][vsts-approvals].
* При выпуске конвейера рассмотрите возможность использования [Application Insights][application-insights] и дополнительных средств мониторинга как можно раньше. Большинство организаций начинают мониторинг только в своей рабочей среде, хоть выявить возможные ошибки можно на более раннем этапе и предотвратить их воздействие на пользователей рабочей среды.

## <a name="pricing"></a>Цены

Расчет стоимости Visual Studio Team Services будет зависеть от количества пользователей в организации, которым требуется доступ, в дополнение к таким факторам, как количество параллельных требуемых выпусков и сборок, а также число тестовых пользователей. Это описано [на странице цен VSTS][vsts-pricing-page].

* [Visual Studio Team Services (VSTS)][vsts-pricing-calculator] — это служба, позволяющая вам управлять жизненным циклом разработки. Оплата производится ежемесячно для каждого пользователя. В дополнение к оплате за дополнительных тестовых пользователей или базовые пользовательские лицензии может взиматься плата в зависимости от требуемых параллельных конвейеров.

## <a name="related-resources"></a>Связанные ресурсы

* [What is DevOps?][devops-whatis] (Что такое DevOps?)
* [DevOps at Microsoft][devops-microsoft] (DevOps в корпорации Майкрософт)
* [DevOps with Visual Studio Team Services][devops-with-vsts] (DevOps с использованием Visual Studio Team Services)
* [Создание конвейера CI/CD для .NET с помощью проекта Azure DevOps][devops-project-create]

<!-- links -->
[ansible]: /azure/ansible/
[application-insights]: /azure/application-insights/app-insights-overview
[app-service-reference-architecture]: /azure/architecture/reference-architectures/app-service-web-app/
[azure-free-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
[arm-templates]: /azure/azure-resource-manager/resource-group-overview#template-deployment
[architecture]: ./media/devops-dotnet-webapp/architecture-devops-dotnet-webapp.png
[availability]: /azure/architecture/checklist/availability
[chef]: /azure/chef/
[design-patterns-availability]: /azure/architecture/patterns/category/availability
[design-patterns-resiliency]: /azure/architecture/patterns/category/resiliency
[design-patterns-scalability]: /azure/architecture/patterns/category/performance-scalability
[design-patterns-security]: /azure/architecture/patterns/category/security
[desired-state-configuration]: /azure/automation/automation-dsc-overview
[devops-microsoft]: /azure/devops/devops-at-microsoft/
[devops-with-vsts]: https://almvm.azurewebsites.net/labs/vsts/
[application-insights]: https://azure.microsoft.com/en-gb/services/application-insights/
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
[vsts-account-create]: /vsts/organizations/accounts/create-account-msa-or-work-student?view=vsts
[vsts-approvals]: /vsts/pipelines/release/approvals/approvals?view=vsts
[devops-project]: https://portal.azure.com/?feature.customportal=false#create/Microsoft.AzureProject
[vsts-deployment-gates]: /vsts/pipelines/release/approvals/gates?view=vsts
[vsts-pricing-calculator]: https://azure.com/e/498aa024454445a8a352e75724f900b1
[vsts-pricing-page]: https://azure.microsoft.com/en-us/pricing/details/visual-studio-team-services/
[vsts-release-variables]: /vsts/pipelines/release/variables?view=vsts&tabs=batch
[vsts-tokenization]: https://marketplace.visualstudio.com/search?term=token&target=VSTS&category=All%20categories&sortBy=Relevance
[azure-key-vault]: /azure/key-vault/key-vault-overview
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[team-foundation-server]: https://visualstudio.microsoft.com/tfs/
[infra-as-code]: https://blogs.msdn.microsoft.com/mvpawardprogram/2018/02/13/infrastructure-as-code/
[service-fabric]:/azure/service-fabric/
[azure-functions]:/azure/azure-functions/
[azure-containers]:https://azure.microsoft.com/en-us/overview/containers/
[compare-vm-hosting]:/azure/app-service/choose-web-site-cloud-service-vm
[app-insights-cd-monitoring]:/azure/application-insights/app-insights-vsts-continuous-monitoring
[azure-region-pair-bcdr]:/azure/best-practices-availability-paired-regions
[devops-project-create]: /vsts/pipelines/apps/cd/azure/azure-devops-project-aspnetcore?view=vsts
[security]: /azure/security/