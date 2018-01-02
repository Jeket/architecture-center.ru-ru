---
title: "Запуск виртуальных машин с балансировкой нагрузки в Azure для обеспечения масштабируемости и доступности"
description: "Узнайте, как запустить несколько виртуальных машин Windows в Azure для обеспечения масштабируемости и доступности."
author: telmosampaio
ms.date: 11/16/2017
pnp.series.title: Windows VM workloads
pnp.series.next: n-tier
pnp.series.prev: single-vm
ms.openlocfilehash: c9b1e52044d38348ecf1bd29cb24b3c20d1d6a45
ms.sourcegitcommit: 115db7ee008a0b1f2b0be50a26471050742ddb04
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/17/2017
---
# <a name="run-load-balanced-vms-for-scalability-and-availability"></a>Запуск виртуальных машин с балансировкой нагрузки для обеспечения масштабируемости и доступности

На этой схеме эталонной архитектуры представлены утвержденные способы запуска нескольких виртуальных машин Linux в масштабируемом наборе за подсистемой балансировки нагрузки для повышения доступности и масштабируемости. Эта архитектура может использоваться для любой рабочей нагрузки без учета состояния, например веб-сервера. Она является строительным блоком при развертывании n-уровневых приложений. [**Разверните это решение**.](#deploy-the-solution)

![[0]][0]

*Скачайте [файл Visio][visio-download] этой архитектуры.*

## <a name="architecture"></a>Архитектура

Эта архитектура основана на [эталонной архитектуре, включающей одну виртуальную машину][single-vm]. Соответствующие рекомендации применимы и к этой архитектуре.

В этой архитектуре рабочая нагрузка распределяется между несколькими экземплярами виртуальных машин. Здесь приведен один общедоступный IP-адрес, а интернет-трафик распределяется между виртуальными машинами с помощью подсистемы балансировки нагрузки. Эта архитектура может использоваться для одноуровневых приложений, например веб-приложения без сохранения состояния.

Архитектура состоит из следующих компонентов:

* **Группа ресурсов.** [Группы ресурсов][resource-manager-overview] используются для группирования ресурсов по времени существования, владельцу и другим критериям для управления.
* **Виртуальная сеть и подсеть.** Каждая виртуальная машина Azure развертывается в виртуальной сети, которую можно разделить на несколько подсетей.
* **Azure Load Balancer.** [Подсистема балансировки нагрузки][load-balancer] распределяет входящие запросы из Интернета между экземплярами виртуальных машин. 
* **Общедоступный IP-адрес.** Общедоступный IP-адрес необходим подсистеме балансировки нагрузки для приема интернет-трафика.
* **Масштабируемый набор виртуальных машин.** [Масштабируемый набор виртуальных машин][vm-scaleset] — это набор идентичных виртуальных машин, используемых для размещения рабочей нагрузки. Масштабируемые наборы позволяют масштабировать определенное число виртуальных машин вручную или автоматически на основе предопределенных правил.
* **Группа доступности**. [Группа доступности][availability-set] содержит виртуальные машины, что позволяет им использовать [соглашение об уровне обслуживания][vm-sla] более высокого уровня. Для применения соглашений об уровне обслуживания более высокого уровня группа доступности должна включать как минимум две виртуальные машины. Группы доступности неявны в масштабируемых наборах. Если создать виртуальную машину за пределами масштабируемого набора, необходимо отдельно создать группу доступности.
* **Управляемые диски.** Управляемые диски Azure управляют файлами VHD виртуальных машин. 
* **Хранилище.** Создайте учетную запись хранения Azure, чтобы хранить журналы диагностики виртуальных машин.

## <a name="recommendations"></a>Рекомендации

Ваши требования могут не полностью соответствовать описываемой здесь архитектуре. Воспользуйтесь этими рекомендациями в качестве отправной точки. 

### <a name="availability-and-scalability-recommendations"></a>Рекомендации по доступности и масштабируемости

Для обеспечения высокой доступности и масштабируемости можно использовать [масштабируемый набор виртуальных машин][vmss]. Масштабируемые наборы виртуальных машин позволяют развернуть набор идентичных виртуальных машин и управлять им. Они также поддерживают автоматическое масштабирование на основе метрик производительности. По мере увеличения нагрузки на виртуальные машины в подсистему балансировки нагрузки автоматически добавляются дополнительные виртуальные машины. Подумайте об использовании масштабируемых наборов, если необходимо быстро развернуть виртуальные машины или выполнить автомасштабирование.

По умолчанию масштабируемые наборы используют избыточную подготовку, что означает, что масштабируемый набор изначально подготавливает виртуальных машин больше, чем необходимо, а затем удаляет лишние. Это увеличивает общее количество успешных попыток при подготовке виртуальных машин. Если вы не используете [управляемые диски](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-managed-disks), рекомендуется размещать не более 20 виртуальных машин в одной учетной записи хранения при включенной избыточной подготовке и не более 40 виртуальных машин при отключенной.

Настройку виртуальных машин, развернутых в масштабируемый набор, можно выполнить двумя основными способами:

- Использовать расширения для настройки виртуальной машины после ее подготовки. В этом случае новые экземпляры виртуальной машины могут дольше запускаться, чем виртуальные машины без расширений.

- Развернуть [управляемый диск](/azure/storage/storage-managed-disks-overview) с помощью пользовательского образа диска. При этом развертывание выполняется быстрее. Однако требуется обновление образа.

Дополнительные сведения см. в статье [Рекомендации по проектированию масштабируемых наборов][vmss-design].

> [!TIP]
> При использовании любого решения автоматического масштабирования заблаговременно протестируйте его с рабочими нагрузками производственного уровня.

Рассмотрите возможность использования группы доступности, если масштабируемый набор не используется. Создайте по крайней мере две виртуальные машины в группе доступности для реализации [соглашения об уровне обслуживания, гарантирующем доступность виртуальных машин Azure][vm-sla]. Для подсистемы балансировки нагрузки Azure также требуется, чтобы виртуальные машины с балансировкой нагрузки входили в одну группу доступности.

В каждой подписке Azure настроены ограничения по умолчанию, включая максимальное количество виртуальных машин в каждом регионе. Их лимит можно увеличить, создав запрос на поддержку. Дополнительные сведения см. в статье [Подписка Azure, границы, квоты и ограничения службы][subscription-limits].

### <a name="network-recommendations"></a>Рекомендации по сети

Разверните виртуальные машины в одной подсети. Не подключайте виртуальные машины напрямую к Интернету. Вместо этого предоставьте каждой из них частный IP-адрес. Клиенты подключаются с помощью общедоступного IP-адреса подсистемы балансировки нагрузки.

Если вам нужно войти в виртуальные машины за подсистемой балансировки нагрузки, попробуйте добавить одну виртуальную машину в качестве jumpbox (узла-бастиона) с общедоступным IP-адресом, предназначенным для входа. Затем войдите в виртуальные машины за подсистемой балансировки нагрузки из jumpbox. Или настройте правила преобразования сетевых адресов (NAT) для входящего трафика SSH подсистемы балансировки нагрузки. Тем не менее использовать jumpbox предпочтительнее, если вы размещаете n-уровневые рабочие нагрузки или несколько рабочих нагрузок.

### <a name="load-balancer-recommendations"></a>Рекомендации по использованию подсистемы балансировки нагрузки

Добавьте все виртуальные машины в группе доступности в серверный пул адресов подсистемы балансировки нагрузки.

Определите правила подсистемы балансировки нагрузки для направления трафика к виртуальным машинам. Например, чтобы включить трафик HTTP, создайте правило, которое сопоставляет порт 80 интерфейсной конфигурации с портом 80 серверного пула адресов. Когда клиент отправляет HTTP-запрос на порт 80, подсистема балансировки нагрузки выбирает IP-адрес серверной части с помощью [хэш-алгоритма][load-balancer-hashing], который включает исходный IP-адрес. Таким образом клиентские запросы распределяются по всем виртуальным машинам.

Для передачи трафика в определенную виртуальную машину используйте правила NAT. Например, чтобы включить подключение к виртуальным машинам по RDP, создайте отдельное правило NAT для каждой виртуальной машины. Каждое правило должно быть сопоставлено с уникальным номером порта 3389 (порт по умолчанию для RDP). Например, используйте порт 50001 для VM1, порт 50002 для VM2 и т. д. Назначьте правила NAT для сетевых адаптеров в виртуальных машинах.

### <a name="storage-account-recommendations"></a>Рекомендации для учетной записи хранения

Мы рекомендуем использовать [управляемые диски](/azure/storage/storage-managed-disks-overview) с [хранилищем класса Premium][premium]. Управляемым дискам не требуется учетная запись хранения. Просто укажите размер и тип диска, и он будет развернут как высокодоступный ресурс.

Если вы используете неуправляемые диски, создайте отдельные учетные записи хранения Azure для виртуальных жестких дисков каждой виртуальной машины, чтобы не превышать [ограничения по операциям ввода-вывода в секунду][vm-disk-limits] для учетных записей хранения.

Создайте одну учетную запись хранения для журналов диагностики. Эта учетная запись может совместно использоваться всеми виртуальными машинами. Это может быть неуправляемая учетная запись хранения, использующая стандартные диски.

## <a name="availability-considerations"></a>Вопросы доступности

Группа доступности делает приложение более устойчивым к запланированным и внеплановым событиям обслуживания.

* *Плановое техническое обслуживание* возникает, когда корпорация Майкрософт обновляет базовую платформу, что иногда приводит к перезапуску виртуальной машины. Azure гарантирует, что виртуальные машины в группе доступности не перезагружаются одновременно. По крайней мере одна из них остается включенной, пока другие перезагружаются.
* *Внеплановое обслуживание* происходит в случае сбоя оборудования. Azure гарантирует, что виртуальные машины в группе доступности подготавливаются на нескольких серверных стойках. Это помогает снизить влияние сбоев оборудования, сети, электропитания и т. д.

Дополнительные сведения см. в статье [Управление доступностью виртуальных машин Windows в Azure][availability-set]. Дополнительные сведения о группах доступности см. в видеоруководстве по [настройке групп доступности для масштабирования виртуальных машин][availability-set-ch9].

> [!WARNING]
> Не забудьте настроить группу доступности при подготовке виртуальной машины. В настоящее время после подготовки виртуальную машину диспетчера ресурсов невозможно добавить в группу доступности.

Подсистема балансировки нагрузки выполняет [проверку работоспособности][health-probes], чтобы отслеживать доступность экземпляров виртуальных машин. Если проверка не выполняется в экземпляре в течение времени ожидания, подсистема балансировки нагрузки перестает отправлять трафик на эту виртуальную машину. Тем не менее подсистема балансировки нагрузки продолжит выполнять проверку, и если виртуальная машина станет снова доступна, отправка трафика к этой виртуальной машине будет возобновлена.

Вот несколько рекомендаций по проверке работоспособности подсистемы балансировки нагрузки:

* При проверке может тестироваться HTTP или TCP. Если на виртуальных машинах запущен HTTP-сервер, создайте проверку HTTP. В противном случае создайте проверку TCP.
* В проверке HTTP укажите путь к конечной точке HTTP. При этом проверяется код ответа HTTP 200 с этого пути. Это может быть путь к корневому каталогу ("/") или конечная точка мониторинга работоспособности, которая реализует определенную пользовательскую логику для проверки работоспособности приложения. Конечная точка должна разрешать анонимные HTTP-запросы.
* Проверка отправляется с [известного][health-probe-ip] IP-адреса — 168.63.129.16. Убедитесь, что вы не блокируете входящий и исходящий трафик для этого IP-адреса в правилах брандмауэра или группе безопасности сети (NSG).
* Используйте [журналы проверки работоспособности][health-probe-log] для просмотра состояния проверки работоспособности. Включите ведение журнала на портале Azure для каждой подсистемы балансировки нагрузки. Журналы записываются в хранилище BLOB-объектов Azure. В журналах показано, сколько виртуальных машин на серверной стороне не получают сетевой трафик из-за неудачных ответов проверки.

## <a name="manageability-considerations"></a>Вопросы управляемости

При наличии нескольких виртуальных машин важно автоматизировать процессы, чтобы они были надежными и повторяемыми. Для автоматизации развертывания можно использовать [службу автоматизации Azure][azure-automation], исправления ОС и другие задачи. Вы можете использовать для этих целей [службу автоматизации Azure][azure-automation] на базе PowerShell. Пример скриптов службы автоматизации можно просмотреть в [коллекции Runbook][runbook-gallery].

## <a name="security-considerations"></a>Вопросы безопасности

Виртуальные сети являются границей, изолирующей трафик в Azure. Виртуальные машины в пределах одной виртуальной сети не могут напрямую обмениваться данными с виртуальными машинами из другой виртуальной сети. Виртуальные машины в одной виртуальной сети могут обмениваться данными, если только не созданы [группы безопасности сети][nsg], ограничивающие этот трафик. Дополнительные сведения см. в статье [Облачные службы Microsoft Cloud и сетевая безопасность][network-security].

Для входящего интернет-трафика правила подсистемы балансировки нагрузки определяют, какой трафик может достичь серверной части. Однако правила подсистемы балансировки нагрузки не поддерживают списки безопасных IP-адресов, поэтому если вы хотите добавить некоторые общедоступные IP-адреса в список надежных, добавьте группу безопасности сети в подсеть.

## <a name="deploy-the-solution"></a>Развертывание решения

Пример развертывания для этой архитектуры можно найти на портале [GitHub][github-folder]. Он позволяет развернуть следующее:

  * виртуальную сеть с одной подсетью с именем **web** для размещения виртуальных машин;
  * масштабируемый набор виртуальных машин, который содержит последнюю версию Windows Server 2016 Datacenter Edition (автомасштабирование включено);
  * подсистему балансировки нагрузки, которая находится перед масштабируемым набором виртуальных машин;
  * группу безопасности сети с правилами для входящего трафика, позволяющими передавать HTTP-трафик в масштабируемый набор виртуальных машин.

### <a name="prerequisites"></a>Предварительные требования

Перед развертыванием эталонной архитектуры в собственной подписке необходимо выполнить следующие действия.

1. Клонируйте и скачайте ZIP-файл [с эталонными архитектурами AzureCAT][ref-arch-repo] в репозитории GitHub, а также выполните его разветвление.

2. Убедитесь, что Azure CLI 2.0 установлен на компьютере. См. инструкции по [установке Azure CLI 2.0][azure-cli-2].

3. Установите пакет npm [стандартных блоков Azure][azbb].

4. В командной строке, строке bash или строке PowerShell войдите в свою учетную запись Azure с помощью одной из приведенных ниже команд и следуйте инструкциям.

  ```bash
  az login
  ```

### <a name="deploy-the-solution-using-azbb"></a>Развертывание решения с помощью azbb

Чтобы развернуть пример одной рабочей нагрузки виртуальной машины, сделайте следующее:

1. Перейдите в папку `virtual-machines\multi-vm\parameters\windows` репозитория, скачанного на предыдущем шаге.

2. Откройте файл `multi-vm-v2.json` и введите имя пользователя и пароль в кавычках, как показано ниже, а затем сохраните файл.

  ```bash
  "adminUsername": "",
  "adminPassword": "",
  ```

3. Запустите `azbb`, чтобы развернуть виртуальные машины, как показано ниже.

  ```bash
  azbb -s <subscription_id> -g <resource_group_name> -l <location> -p multi-vm-v2.json --deploy
  ```

Дополнительные сведения о развертывании этого примера эталонной архитектуры см. в [нашем репозитории GitHub][git].

<!-- links -->

[availability-set]: /azure/virtual-machines/virtual-machines-windows-manage-availability
[availability-set-ch9]: https://channel9.msdn.com/Series/Microsoft-Azure-Fundamentals-Virtual-Machines/08
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks
[azure-automation]: /azure/automation/automation-intro
[azure-cli]: /azure/virtual-machines-command-line-tools
[azure-cli-2]: /azure/install-azure-cli?view=azure-cli-latest
[git]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[github-folder]: https://github.com/mspnp/reference-architectures/tree/master/virtual-machines/multi-vm
[health-probe-log]: /azure/load-balancer/load-balancer-monitor-log
[health-probes]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[health-probe-ip]: /azure/virtual-network/virtual-networks-nsg#special-rules
[load-balancer]: /azure/load-balancer/load-balancer-get-started-internet-arm-cli
[load-balancer-hashing]: /azure/load-balancer/load-balancer-overview#load-balancer-features
[naming-conventions]: ../../best-practices/naming-conventions.md
[network-security]: /azure/best-practices-network-security
[nsg]: /azure/virtual-network/virtual-networks-nsg
[premium]: /azure/storage/common/storage-premium-storage
[ref-arch-repo]: https://github.com/mspnp/reference-architectures
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview 
[runbook-gallery]: /azure/automation/automation-runbook-gallery#runbooks-in-runbook-gallery
[single-vm]: single-vm.md
[subscription-limits]: /azure/azure-subscription-service-limits
[visio-download]: https://archcenter.azureedge.net/cdn/vm-reference-architectures.vsdx
[vm-disk-limits]: /azure/azure-subscription-service-limits#virtual-machine-disk-limits
[vm-scaleset]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vm-sizes]: https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_2/
[vmss]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-overview
[vmss-design]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-design-overview
[vmss-quickstart]: https://azure.microsoft.com/documentation/templates/?term=scale+set
[0]: ./images/multi-vm-diagram.png "Архитектура решения с несколькими виртуальными машинами в Azure, включающая группу доступности с двумя виртуальными машинами и подсистемой балансировки нагрузки"