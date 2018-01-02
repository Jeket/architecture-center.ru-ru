---
title: "Техническое руководство. Восстановление после локальных сбоев в Azure"
description: "Эта статья поможет вам понять и освоить процедуру создания надежных, высокодоступных, отказоустойчивых приложений, а также спланировать аварийное восстановление в случае локальных сбоев в Azure."
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: 180eb465e5f82406bb03924a29d5b06d43bbaa24
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
---
[!INCLUDE [header](../_includes/header.md)]
# <a name="azure-resiliency-technical-guidance-recovery-from-local-failures-in-azure"></a>Техническое руководство по обеспечению устойчивости в Azure. Восстановление после локальных сбоев в Azure

На доступность приложения могут повлиять два основных фактора:

* сбой устройств, таких как диски и серверы;
* нехватка критически важных ресурсов, например вычислительных ресурсов при пиковой нагрузке.

Azure позволяет сохранить высокий уровень доступности в этих условиях благодаря сочетанию управления ресурсами, обеспечения эластичности, балансировки нагрузки и секционирования. Некоторые из этих функций выполняются автоматически для всех облачных служб. Но иногда разработчику приложения необходимо выполнить дополнительные действия, чтобы обеспечить применение этих функций.

## <a name="cloud-services"></a>Облачные службы
Все облачные службы представляют собой коллекции экземпляров одной или нескольких веб- либо рабочих ролей. Несколько экземпляров одной роли могут выполняться одновременно. Количество экземпляров зависит от заданной конфигурации. Отслеживание экземпляров роли и управление ими осуществляется с помощью контроллера структуры. Этот компонент автоматически обнаруживает сбои программного обеспечения и оборудования и реагирует на них.

Каждый экземпляр роли выполняется в отдельной виртуальной машине и взаимодействует с контроллером структуры через гостевого агента. Гостевой агент собирает метрики ресурсов и узлов, в том числе показатели использования виртуальных машин и ресурсов, данные журналов, а также сведения о состоянии, исключениях и сбоях. Контроллер структуры отправляет запросы к гостевому агенту через интервалы, заданные в настройках, и перезапускает виртуальную машину, если гостевой агент не отвечает. В случае сбоя оборудования связанный контроллер структуры перемещает все затронутые экземпляры роли на новый аппаратный узел и соответствующим образом изменяет параметры маршрутизации трафика в настройках сети.

Для использования этих возможностей разработчикам необходимо настроить все роли служб таким образом, чтобы они не сохраняли сведения о состоянии в экземплярах роли. Вместо этого все постоянные данные должны храниться в долговременном хранилище, например в службе хранилища Azure или Базе данных SQL Azure. В этом случае запросы могут обрабатываться с помощью любых ролей. Кроме того, если экземпляры роли в любое время выйдут из строя, это не вызовет несоответствия между временным и постоянным состояниями службы.

Необходимость хранить сведения о состоянии вне экземпляров роли сопровождается некоторыми требованиями. К примеру, все связанные изменения в таблице службы хранилища Azure по возможности должны осуществляться в одной транзакции группы сущностей. Конечно, внести все изменения в рамках одной транзакции возможно не всегда. Необходимо также принять специальные меры, чтобы сбои экземпляров роли не вызывали проблем в случае прерывания длительных операций, охватывающих два и более обновлений постоянного состояния службы. Другая роль, которая попытается повторить эту операцию, должна быть готова обрабатывать случаи, когда операция была частично завершена.

Например, если в службе, которая распределяет секции данных между несколькими хранилищами, рабочая роль выйдет из строя при перемещении сегмента, эта операция может не завершиться либо ее может заново начать другая рабочая роль, что может привести к потере или повреждению данных. Чтобы предотвратить возникновение таких проблем, длительные операции должны иметь следующие характеристики.

* *Идемпотентность*. Повторяемые операции без побочных эффектов. Идемпотентная длительная операция должна действовать одинаково, независимо от того, сколько раз она выполняется, даже в случае прерывания во время выполнения.
* *Добавочная возобновляемость*. Операции, способные продолжаться с момента сбоя. Добавочно возобновляемая длительная операция должна состоять из последовательности небольших атомарных операций. Кроме того, во время этой операции в долговременное хранилище должны записываться сведения о ходе ее выполнения, чтобы при каждом последующем вызове ее выполнение продолжалось с момента остановки.

И наконец, все длительные операции должны вызываться несколько раз, пока они не будут полностью завершены. Например, операция подготовки, помещенная в очередь Azure, будет удалена из нее экземпляром рабочей роли только после успешного завершения. Для очистки данных, созданных в результате прерванных операций, может потребоваться сборка мусора.

### <a name="elasticity"></a>Эластичность
Начальное количество выполняющихся экземпляров для каждой роли определено в конфигурации соответствующей роли. Администраторы должны изначально настроить для каждой роли выполнение двух или более экземпляров с учетом ожидаемой нагрузки. Тем не менее число экземпляров роли можно с легкостью увеличить или уменьшить по мере изменения шаблонов использования. Эту операцию можно выполнить вручную на портале Azure или автоматизировать с помощью Windows PowerShell, API управления службами или сторонних инструментов. Дополнительные сведения см. в статье [Автомасштабирование облачной службы](/azure/cloud-services/cloud-services-how-to-scale/).

### <a name="partitioning"></a>Секционирование
Контроллер структуры Azure использует два компонента для секционирования:

* *Домен обновления* используется для обновления экземпляров роли службы в группах. Служба Azure развертывает экземпляры службы в нескольких доменах обновления. При обновлении на месте контроллер структуры завершает работу всех экземпляров в одном домене обновления, обновляет их и перезапускает, а затем переходит к следующему домену обновления. Этот подход позволяет предотвратить недоступность всей службы во время процесса обновления.
* *Домен сбоя* используется для изоляции компонентов оборудования или сети, которые являются потенциальными точками отказа. Контроллер структуры распределяет экземпляры ролей с несколькими экземплярами между несколькими доменами сбоя, чтобы избежать прерывания работы службы из-за сбоев отдельного оборудования. С помощью доменов сбоя в Azure регулируются все риски, связанные со сбоями серверных и кластерных компонентов.

В соответствии с [соглашением об уровне обслуживания Azure](https://azure.microsoft.com/support/legal/sla/) корпорация Майкрософт гарантирует, что два или несколько экземпляров веб-роли, развернутых в разных доменах сбоя и обновления, будут иметь возможность внешнего подключения по крайней мере 99,95 % времени. В отличие от доменов обновления количеством доменов сбоя управлять невозможно. Служба Azure автоматически выделяет домены сбоя и распределяет между ними экземпляры роли. По крайней мере два первых экземпляра каждой роли размещаются в разных доменах сбоя и обновления, чтобы гарантировать, что каждая роль с минимум двумя экземплярами будет удовлетворять условиям соглашения об уровне обслуживания. Это показано на схеме ниже.

![Упрощенное представление изоляции доменов сбоя](./images/technical-guidance-recovery-local-failures/partitioning-1.png)

### <a name="load-balancing"></a>Балансировка нагрузки.
Весь входящий трафик веб-роли проходит через балансировщик нагрузки без отслеживания состояния, который распределяет запросы клиентов между экземплярами роли. У отдельных экземпляров роли нет общедоступных IP-адресов, и они недоступны напрямую через Интернет. Веб-роли не отслеживают состояние, поэтому любой запрос клиента может быть направлен в любой экземпляр роли. Каждые 15 секунд возникает событие [StatusCheck](https://msdn.microsoft.com/library/microsoft.windowsazure.serviceruntime.roleenvironment.statuscheck.aspx). Оно позволяет узнать, готова ли роль к приему трафика. Если роль занята, она перестанет использоваться в балансировщике нагрузки.

## <a name="virtual-machines"></a>Виртуальные машины
С точки зрения высокой доступности виртуальные машины Azure отличаются от вычислительных ролей модели "платформа как услуга" (PaaS) в нескольких аспектах. В некоторых случаях для обеспечения высокой доступности требуются дополнительные действия.

### <a name="disk-durability"></a>Устойчивость диска
В отличие от экземпляров роли PaaS данные, хранящиеся на дисках виртуальной машины, остаются постоянными даже при перемещении виртуальной машины. Виртуальные машины Azure используют диски виртуальной машины, которые представлены в качестве больших двоичных объектов в службе хранилища Azure. Благодаря возможностям доступности службы хранилища Azure данные, хранящиеся на дисках виртуальной машины, также имеют высокий уровень доступности.

Однако это не относится к диску D (в виртуальных машинах Windows). Диск D представляет собой физическое хранилище на стоечном сервере, где размещена виртуальная машина. При перезапуске этого диска хранящиеся на нем данные будут потеряны. Таким образом, диск D предназначен только для временного хранения. В Linux Azure обычно (но не всегда) предоставляет в качестве локального временного диска блочное устройство /dev/sdb. Зачастую он подключается агентом Linux для Azure. Точками подключения выступают /mnt/resource и /mnt (настраиваемые в файле конфигурации /etc/waagent.conf).

### <a name="partitioning"></a>Секционирование
Служба Azure изначально распознает уровни приложения PaaS (веб-роль и рабочая роль) и, соответственно, может должным образом распределить их по доменам сбоя и обновления. Уровни в приложениях IaaS, напротив, следует определить вручную с помощью групп доступности. Группы доступности требуются в рамках соглашения об уровне обслуживания для систем IaaS.

![Группы доступности для виртуальных машин Azure](./images/technical-guidance-recovery-local-failures/partitioning-2.png)

На схеме выше уровень Internet Information Server (IIS), который выступает в качестве уровня веб-приложения, и уровень SQL, который выступает в качестве уровня данных, назначены разным группам доступности. Это позволяет гарантировать аппаратную избыточность всех экземпляров каждого уровня за счет распределения виртуальных машин по доменам сбоя и поддержания уровней работы во время обновления.

### <a name="load-balancing"></a>Балансировка нагрузки.
Если требуется распределять трафик между виртуальными машинами, необходимо сгруппировать их в приложении и распределить нагрузку в определенной конечной точке TCP или UDP. Дополнительные сведения см. в статье [Балансировка нагрузки для служб инфраструктуры Azure](/azure/virtual-machines/virtual-machines-linux-load-balance/?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). Если виртуальные машины получают входящие данные из другого источника (например, механизма очередей), то балансировщик нагрузки не требуется. Балансировщик нагрузки использует обычную проверку работоспособности, чтобы определить, следует ли отправлять трафик на узел. Можно также настроить собственные зонды, которые помогут определить, следует ли отправлять трафик в виртуальную машину, на основе метрик работоспособности приложения.

## <a name="storage"></a>Хранилище
Служба хранилища Azure — это базовая служба долговременного хранения данных Azure, которая предоставляет хранилища BLOB-объектов, таблиц, очередей и дисков виртуальных машин. Она обеспечивает высокую доступность в пределах одного центра обработки данных благодаря сочетанию репликации и управления ресурсами. Соглашение об уровне обслуживания по доступности службы хранилища Azure гарантирует, что по крайней мере в 99,9 % случаев:

* правильно отформатированные запросы на добавление, обновление, чтение и удаление данных будут должным образом обработаны;
* учетные записи хранения будут иметь возможность подключения к шлюзу Интернета.

### <a name="replication"></a>Репликация
Надежность данных в службе хранилища Azure обеспечивается за счет хранения нескольких копий всех данных на разных дисках, расположенных в полностью независимых физических подсистемах хранения в регионе. Данные реплицируются синхронно, и перед подтверждением записи все копии фиксируются. Служба хранилища Azure строго согласованная. Это означает, что операции чтения гарантировано отражают последние записанные данные. Кроме того, копии данных постоянно сканируются на наличие ухудшения производительности и нарушения целостности (чему часто не уделяют должного внимания). В случае обнаружения эти проблемы устраняются.

Служба хранилища Azure обеспечивает репликацию служб. Разработчику службы не нужно предпринимать какие-либо дополнительные действия для восстановления после локального сбоя.

### <a name="resource-management"></a>Управление ресурсами
Учетные записи хранения, созданные после мая 2014 года, могут предоставлять до 500 ТБ емкости (предыдущее ограничение — 200 ТБ). Если этого пространства недостаточно, приложения необходимо спроектировать таким образом, чтобы они поддерживали использование нескольких учетных записей хранения.

### <a name="virtual-machine-disks"></a>Диски виртуальной машины
Диск виртуальной машины хранится в качестве страничного BLOB-объекта в службе хранилища Azure, благодаря чему он получает те же свойства надежности и масштабируемости, что и хранилище BLOB-объектов. Таким образом, данные на диске виртуальной машины не могут быть утеряны, даже если на сервере, где запущена виртуальная машина, происходит сбой и ее необходимо перезапустить на другом сервере.

## <a name="database"></a>База данных
### <a name="sql-database"></a>База данных SQL
База данных SQL Azure предоставляется как услуга. Благодаря этому приложения могут быстро подготавливать реляционные базы данных, вставлять в них данные и отправлять к ним запросы. Эта база данных содержит много привычных возможностей и функций SQL Server, а все, что касается оборудования, конфигурации, исправлений и устойчивости, представлено в ней на абстрактном уровне.

> [!NOTE]
> База данных SQL Azure не предоставляет все точно такие же функции, как SQL Server. Она предназначена для выполнения других требований, соответствующих облачным приложениям (эластичное масштабирование, предоставление модели "база данных как услуга" для уменьшения расходов на обслуживание и т. д.). Дополнительные сведения см. в [этой статье](/azure/sql-database/sql-database-paas-vs-sql-server-iaas/).
> 
> 

#### <a name="replication"></a>Репликация
База данных SQL Azure обеспечивает высокий уровень устойчивости к сбоям на уровне узла. Все операции записи в базу данных автоматически реплицируются на два или более фоновых узла с помощью метода фиксации кворума. При этом транзакция будет считаться завершенной и вернется, только если первичная реплика и по крайней мере одна вторичная реплика подтвердят, что действие записано в журнал транзакций. В случае сбоя узла база данных автоматически переносится на одну из вторичных реплик. При этом временное подключение для клиентских приложений прерывается. По этой причине все клиенты Базы данных SQL Azure должны выполнять определенный вид обработки временных подключений. Дополнительные сведения см. в статье [Конкретные рекомендации по использованию механизма повторов](/azure/best-practices-retry-service-specific/).

#### <a name="resource-management"></a>Управление ресурсами
Во время создания для каждой базы данных задается предельное значение размера. В настоящее время максимальный размер составляет 1 ТБ. Ограничения размера зависят от уровня службы. См. сведения об [уровнях служб и производительности баз данных SQL Azure](/azure/sql-database/sql-database-resource-limits/#service-tiers-and-performance-levels). Если база данных достигает максимального размера, последующие команды вставки и обновления отклоняются (запрос и удаление данных по-прежнему возможны).

Для управления ресурсами в Базе данных SQL Azure используется структура. Однако вместо контроллера структуры для обнаружения сбоев применяется кольцевая топология. Каждая реплика в кластере имеет два соседних экземпляра и отвечает за обнаружение, когда они выходят из строя. Если реплика выходит из строя, ее соседи активируют агент перенастройки, чтобы повторно создать эту реплику на другом компьютере. Чтобы логический сервер не использовал слишком много ресурсов на виртуальной машине и не превышал физические ограничения, осуществляется регулирование ресурсов ядра.

### <a name="elasticity"></a>Эластичность
Если приложению требуется более 1 ТБ емкости для базы данных, необходимо применить масштабирование. Масштабирование при использовании Базы данных SQL Azure можно выполнить путем секционирования данных по нескольким базам данных SQL вручную (также известно как сегментирование). Такой подход гарантирует практически линейный рост затрат в соответствии с масштабом. Гибкий рост или предоставление емкости по запросу могут сопровождаться добавочными затратами, так как счета за базы данных выставляются на основе средней фактической используемой емкости за день, а не максимально возможного размера.

## <a name="sql-server-on-virtual-machines"></a>SQL Server на виртуальных машинах
Установив SQL Server (версии 2014 или более поздней) на виртуальных машинах Azure, вы получаете стандартные функции обеспечения доступности SQL Server, такие как группы доступности AlwaysOn и зеркальное отображение базы данных. Обратите внимание, что виртуальные машины, хранилища и сети Azure имеют рабочие характеристики, отличные от характеристик локальной, невиртуализированной ИТ-инфраструктуры. Чтобы успешно реализовать решение по обеспечению высокой доступности и аварийному восстановлению для SQL Server в Azure, вы должны понимать эти различия и учитывать их при разработке решения.

### <a name="high-availability-nodes-in-an-availability-set"></a>Узлы с высоким уровнем доступности в группе доступности
При реализации решения для обеспечения высокой доступности в Azure группа доступности позволяет помещать узлы с высоким уровнем доступности в отдельные домены сбоя и обновления. Следует уточнить, что группа доступности — это понятие Azure. Это рекомендуемый подход, которому необходимо следовать, чтобы обеспечить высокую доступность баз данных при использовании групп доступности AlwaysOn, зеркального отображения базы данных или других методов. В противном случае вы можете думать, что в вашей системе обеспечена высокая доступность, но на самом деле все узлы могут одновременно выйти из строя, так как они размещены в одном домене сбоя в регионе Azure.

Этот подход неприменим при использовании доставки журналов, так как это решение предназначено для обеспечения аварийного восстановления, и, следовательно, серверы должны находиться в разных регионах Azure. По определению эти регионы представляют собой отдельные домены сбоя.

Чтобы развертываемые на классическом портале виртуальные машины облачных служб Azure находились в одной группе доступности, их необходимо развернуть в одной облачной службе. Что же касается виртуальных машин, развертываемых с помощью Azure Resource Manager (на текущем портале), такого ограничения нет. При развертывании виртуальных машин в облачной службе Azure через классический портал в одной группе доступности можно разместить только узлы одной облачной службы. Кроме того, виртуальные машины облачных служб должны находиться в одной и той же виртуальной сети, чтобы их IP-адреса не менялись даже после восстановления службы. Это позволяет избежать сбоев при обновлении DNS.

### <a name="azure-only-high-availability-solutions"></a>Только в Azure: решения для обеспечения высокой доступности
У вас может быть решение высокого уровня доступности для баз данных SQL Server в Azure, использующее группы доступности AlwaysOn или зеркальное отображение базы данных.

На схеме ниже показана архитектура групп доступности AlwaysOn в виртуальных машинах Azure. Эта схема взята из подробной статьи на эту тему: [Высокий уровень доступности и аварийное восстановление для SQL Server на виртуальных машинах Azure](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

![Группы доступности AlwaysOn в Microsoft Azure](./images/technical-guidance-recovery-local-failures/high_availability_solutions-1.png)

С помощью шаблона AlwaysOn на портале Azure можно автоматически подготовить готовое развертывание групп доступности AlwaysOn в виртуальных машинах Azure. Дополнительные сведения см. в записи блога [SQL Server AlwaysOn Offering in Microsoft Azure Portal Gallery](https://blogs.technet.microsoft.com/dataplatforminsider/2014/08/25/sql-server-alwayson-offering-in-microsoft-azure-portal-gallery/) (Предложения AlwaysOn SQL Server в коллекции портала Microsoft Azure).

На схеме ниже показано использование зеркального отображения базы данных в виртуальных машинах Azure. Эта схема взята из той же подробной статьи: [Высокий уровень доступности и аварийное восстановление для SQL Server на виртуальных машинах Azure](/azure/virtual-machines/windows/sql/virtual-machines-windows-sql-high-availability-dr/).

![Зеркальное отображение базы данных в Microsoft Azure](./images/technical-guidance-recovery-local-failures/high_availability_solutions-2.png)

> [!NOTE]
> В обеих архитектурах требуется контроллер домена. Однако при применении зеркального отображения базы данных можно использовать сертификаты сервера, и тогда необходимость в контроллере домена отпадет.
> 
> 

## <a name="other-azure-platform-services"></a>Другие службы платформы Azure
Приложения, созданные на основе Azure, поддерживают возможности платформы для восстановления после локальных сбоев. В некоторых случаях можно предпринять определенные действия, чтобы повысить доступность для конкретного сценария.

### <a name="service-bus"></a>Service Bus
Чтобы снизить риск временного сбоя служебной шины Azure, можно создать долгосрочную очередь на стороне клиента. Это временный альтернативный метод, позволяющий локально хранить сообщения, которые не могут быть добавлены в очередь служебной шины. Эти временно хранимые сообщения будут обработаны в приложении после восстановления работы службы. Дополнительные сведения см. в статье [Рекомендации по повышению производительности с помощью обмена сообщениями через брокер в служебной шине](/azure/service-bus-messaging/service-bus-performance-improvements/) и в разделе о [служебной шине в рамках решения по аварийному восстановлению](recovery-loss-azure-region.md#other-azure-platform-services).

### <a name="hdinsight"></a>HDInsight
Данные, связанные с Azure HDInsight, по умолчанию сохраняются в хранилище BLOB-объектов Azure. Свойства высокой доступности и устойчивости для хранилища BLOB-объектов задаются в службе хранилища Azure. Во временной распределенной файловой системе Hadoop (HDFS), которую при необходимости подготавливает служба HDInsight, выполняется многоузловая обработка в рамках заданий Hadoop MapReduce. Результаты задания MapReduce также по умолчанию сохраняются в хранилище BLOB-объектов Azure. Таким образом, обработанные данные устойчивы и имеют высокий уровень доступности после отзыва кластера Hadoop. Дополнительные сведения см. в разделе об [HDInsight в рамках решения по аварийному восстановлению](recovery-loss-azure-region.md#other-azure-platform-services).

## <a name="checklists-for-local-failures"></a>Контрольные списки для локальных сбоев
### <a name="cloud-services"></a>Облачные службы
1. Просмотрите раздел "Облачные службы" этого документа.
2. Настройте по крайней мере два экземпляра для каждой роли.
3. Сохраняйте сведения о состоянии в долговременном хранилище, а не в экземплярах роли.
4. Настройте правильную обработку события StatusCheck.
5. По возможности объединяйте связанные изменения в транзакциях.
6. Убедитесь, что задачи рабочей роли идемпотентные и перезапускаемые.
7. Настройте выполнение вызова неудачных операций до тех пор, пока они не будут завершены.
8. Рассмотрите возможность применения стратегий автоматического масштабирования.

### <a name="virtual-machines"></a>Виртуальные машины
1. Просмотрите раздел "Виртуальные машины" этого документа.
2. Не используйте диск D для постоянного хранения.
3. Объединяйте виртуальные машины на уровне службы в группу доступности.
4. Настройте балансировку нагрузки и при необходимости зонды.

### <a name="storage"></a>Хранилище
1. Просмотрите раздел "Хранилище" этого документа.
2. Используйте несколько учетных записей хранения, если объем данных или пропускной способности превышает квоту.

### <a name="sql-database"></a>База данных SQL
1. Просмотрите раздел "База данных SQL" этого документа.
2. Реализуйте политику повтора для обработки временных ошибок.
3. Используйте секционирование или сегментирование в качестве стратегии масштабирования.

### <a name="sql-server-on-virtual-machines"></a>SQL Server на виртуальных машинах
1. Просмотрите раздел "SQL Server на виртуальных машинах" этого документа.
2. Следуйте приведенным выше рекомендациям для виртуальных машин.
3. Используйте функции для обеспечения высокой доступности SQL Server, например AlwaysOn.

### <a name="service-bus"></a>Service Bus
1. Просмотрите раздел "Служебная шина" этого документа.
2. Рассмотрите возможность создания долгосрочной очереди на стороне клиента для хранения избыточных сообщений.

### <a name="hdinsight"></a>HDInsight
1. Просмотрите раздел "HDInsight" этого документа.
2. Дополнительные действия, касающиеся обеспечения высокой доступности, в случае локальных сбоев для этого компонента отсутствуют.
