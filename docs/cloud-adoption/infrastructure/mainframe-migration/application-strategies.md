---
title: Миграция с мейнфреймов. Перенос приложений с мейнфреймов
description: Перенос приложений из сред мейнфреймов в Azure — проверенную, высокодоступную и масштабируемую инфраструктуру для систем, работающих на мейнфреймах.
author: njray
ms.date: 12/26/2018
ms.openlocfilehash: dcae5077e26ab8ba9b08e0da71a5e69d0d9f62e3
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901427"
---
# <a name="mainframe-application-migration"></a>Перенос приложений с мейнфреймов

Многие команды прагматично подходят к переносу приложений из сред мейнфреймов в Azure: код при возможности используется повторно, а затем, если приложения необходимо переписать или заменить, они развертываются поэтапно.

Для переноса приложений обычно применяется одна или несколько из следующих стратегий:

- Повторное размещение. Переместите существующий код, программы и приложения с мейнфрейма. Затем повторно скомпилируйте код, чтобы запустить его в эмуляторе мейнфреймов, который размещен в экземпляре облака. Как правило, при этом подходе сначала приложения перемещаются в облачный эмулятор, а затем базы данных переносятся в облачную базу данных. По мере преобразования данных и файлов потребуется немного изменить код и выполнить рефакторинг.

    Кроме того, вы можете повторно разместить приложение у своего поставщика услуг размещения. Одним из основных преимуществ облака является аутсорсинг управления инфраструктурой. Вы можете найти центр обработки данных и разместить в нем рабочие нагрузки для своих мейнфреймов. Эта модель позволяет выиграть время, снизить зависимость от поставщика и сэкономить средства за определенный период.

- Прекращение поддержки приложений. Перед миграцией удалите все приложения, которые больше не будут использоваться.

- Повторная сборка. Некоторые организации принимают решение полностью переписать программы с использованием современных методик. Учитывая дополнительные затраты и сложность этого подхода, он применяется реже, чем lift-and-shift. Если вы также выбрали этот подход, рекомендуем заменить модули и код с помощью механизмов преобразования кода после миграции.

- Замена. Этот подход заключается в замене функциональных возможностей мейнфреймов на аналогичные функции в облаке. Программное обеспечение как услуга (SaaS) — это один из вариантов, который используется в корпоративных решениях для отдела кадров, финансов, производства или планирования ресурсов предприятия. Кроме того, многие приложения, разработанные с учетом специфики отрасли, позволяют вам решать проблемы, которые ранее решались с помощью мейнфреймов.

Начните с планирования рабочих нагрузок, которые необходимо перенести в первую очередь. Затем перенесите приложения, устаревшие базы кода и базы данных, связанные с этими нагрузками, с учетом установленных требований.

## <a name="mainframe-emulation-in-azure"></a>Эмуляция мейнфрейма в Azure

Облачные службы Azure позволяют эмулировать традиционные среды мейнфреймов, чтобы повторно использовать существующий код мейнфреймов и приложений. К общим компонентам сервера, которые можно эмулировать, относятся системы онлайн-обработки транзакций (OLTP), пакетной обработки и приема данных.

### <a name="oltp-systems"></a>Системы OLTP

Во многих мейнфреймах используются системы OLTP, обрабатывающие тысячи или миллионы обновлений для огромного числа пользователей. В этих приложениях часто применяется программное обеспечение для обработки транзакций и экранных форм, например системы управления информацией клиента (CICS), системы управления информацией (IMS) и интерфейсный процессор терминала (TIP).

При переносе приложений OLTP в Azure вы можете запускать эмуляторы для обработки транзакций мейнфреймов (TP) в виде IaaS (инфраструктура как услуга) на виртуальных машинах в Azure. Функциональные возможности для работы с формами и экраном также реализуются с помощью веб-серверов. Для доступа к данным и транзакциям вы можете применять этот подход совместно с API-интерфейсами баз данных, такими как Объекты данных ActiveX (ADO), ODBC и Java Database Connectivity (JDBC).

### <a name="time-constrained-batch-updates"></a>Пакетные обновления, ограниченные по времени

Во многих системах на базе мейнфреймов миллионы учетных записей обновляются ежемесячно или ежегодно, например в банках, страховых компаниях и государственных организациях. Мейнфреймы обрабатывают такие рабочие нагрузки с помощью систем обработки данных с высокой пропускной способностью. Чтобы повысить производительность, зависящую от скорости обработки (количества операций ввода-вывода в секунду) магистрали мейнфреймов, пакетные задания мейнфреймов обычно выполняются последовательно.

Для повышения производительности в облачных средах для пакетных операций используются параллельные вычисления и высокоскоростные сети. Если вам нужно оптимизировать производительность пакетных операций, Azure предоставляет различные варианты вычислений, хранения данных и сетей.

### <a name="data-ingestion-systems"></a>Системы приема данных

Мейнфреймы принимают для обработки большие пакеты данных из области розничной торговли, финансовых услуг, производства и других решений для обработки. Чтобы скопировать данные в расположение для хранения и из него, вы можете использовать простые средства командной строки Azure, такие как [AzCopy](/azure/storage/common/storage-use-azcopy). Кроме того, служба [Фабрика данных Azure](/azure/data-factory/introduction) позволяет принимать данные из разрозненных хранилищ, чтобы создавать и планировать рабочие процессы на основе этих данных.

Кроме среды эмуляции, Azure предоставляет службы PaaS (платформа как услуга) и аналитики, которые позволяют улучшить существующие среды мейнфреймов.

## <a name="migrate-oltp-workloads-to-azure"></a>Перенос рабочих нагрузок OLTP в Azure

Подход lift-and-shift позволяет быстро перенести существующие приложения в Azure без изменения их кода. Каждое приложение переносится без изменений, чтобы вы могли воспользоваться преимуществами облачной среды без рисков и затрат, связанных с изменением кода. С этой целью для мониторов обработки транзакций (TP) мейнфрейма в Azure применяется эмулятор.

Мониторы обработки транзакций предоставляются различными поставщиками и работают на виртуальных машинах (инфраструктура как услуга (IaaS) в Azure). На следующей схеме показан перенос веб-приложения на основе IBM DB2, системы управления реляционной базой данных (СУБД) на мейнфрейм IBM z/OS (до и после). В DB2 для z/OS файлы метода доступа к виртуальному хранилищу (VSAM) и индексно-последовательный метод доступа (ISAM) применяются для хранения неструктурированных файлов. В этой архитектуре CICS также используется для мониторинга транзакций.

![Перенос среды мейнфрейма в Azure с помощью программного обеспечения для эмуляции (lift-and-shift)](../../_images/mainframe-migration/mainframe-vs-azure.png)

Для запуска диспетчера транзакций и пакетных заданий, использующих JCL, в Azure используется среда эмуляции. На уровне данных DB2 заменяется [Базой данных SQL Azure](/azure/sql-database/sql-database-technical-overview). Кроме того, можно использовать Microsoft SQL Server, DB2 LUW или Oracle Database. Эмулятор поддерживает IMS, VSAM и SEQ. Средства управления системой мейнфрейма заменяются службами Azure и программным обеспечением сторонних поставщиков, которое запускается на виртуальных машинах.

Как правило, функции обработки экрана и форм для ввода данных реализуются с помощью веб-серверов, которые можно комбинировать с API баз данных, такими как ADO, ODBC и JDBC, для транзакций и доступа к данным. Полный перечень IaaS-компонентов Azure зависит от операционной системы, которую вы предпочитаете использовать. Например: 

- Виртуальные машины под управлением Windows. Internet Information Server (IIS) вместе с ASP.NET для обработки экрана и бизнес-логики. Для доступа к данным и транзакций используйте стандарт ADO.NET.

- Виртуальные машины под управлением Linux. Серверы Java-приложений, такие как Apache Tomcat, для обработки экрана и бизнес-функций на платформе Java. Для доступа к данным и транзакций используйте стандарт JDBC.

## <a name="migrate-batch-workloads-to-azure"></a>Перенос пакетных рабочих нагрузок в Azure

Пакетные операции в Azure отличаются от выполняемых в обычной среде для пакетных задач на мейнфреймах. Чтобы повысить производительность, зависящую от количества операций ввода-вывода в секунду в магистрали мейнфреймов, пакетные задания мейнфреймов обычно выполняются последовательно. Для повышения производительности пакетных операций в облачных средах используются параллельные вычисления и высокоскоростные сети.

Чтобы оптимизировать производительность пакетных операций с помощью Azure, учитывайте возможности [вычислительных ресурсов](/azure/virtual-machines/windows/overview), [хранилища](/azure/storage/blobs/storage-blobs-introduction), [сети](https://azure.microsoft.com/blog/maximize-your-vm-s-performance-with-accelerated-networking-now-generally-available-for-both-windows-and-linux/) и [мониторинга](/azure/azure-monitor/overview), как описано ниже.

### <a name="compute"></a>Службы вычислений

Используйте:

- Виртуальные машины с высокой тактовой частотой процессора. Часто приложения для мейнфреймов с высокой тактовой частотой ЦП бывают однопоточными.

- Виртуальные машины с большим объемом памяти для кэширования данных и рабочих областей приложений.

- Виртуальные машины с виртуальными ЦП более высокой плотности, позволяющие использовать преимущества многопоточной обработки, если приложение поддерживает многопоточность.

- Параллельную обработку. Azure легко масштабируется для параллельной обработки, обеспечивая большую вычислительную мощность для выполнения пакетных операций.

### <a name="storage"></a>Хранилище

Используйте:

- [SSD (цен. категория "Премиум") Azure](/azure/virtual-machines/windows/premium-storage) или [SSD (цен. категория "Ультра") Azure](/azure/virtual-machines/windows/disks-ultra-ssd), чтобы обеспечить максимальное количество операций ввода-вывода в секунду.

- Чередование нескольких дисков, чтобы увеличить отношение количества операций ввода-вывода в секунду к объему хранилища.

- Секционирование хранилища для распределения операций ввода-вывода между несколькими устройствами хранения в Azure.

### <a name="networking"></a>Сеть

- Используйте [ускоренную сеть Azure](/azure/virtual-network/create-vm-accelerated-networking-powershell), чтобы минимизировать задержку.

### <a name="monitoring"></a>Мониторинг

- Администраторы могут отслеживать избыточную производительность выполнения пакетов и устранять узкие места с помощью таких средств мониторинга, как [Azure Monitor](/azure/azure-monitor/overview), [Azure Application Insights](/azure/application-insights/app-insights-overview) и даже журналы Azure.

## <a name="migrate-development-environments"></a>Перенос сред разработки

Для разработки облачной распределенной архитектуры применяется другой набор средств разработки, обеспечивающий преимущества современных методик и языков. Чтобы облегчить переход, используйте среду разработки с другими средствами, предназначенными для эмуляции сред IBM z/OS. Ниже приведен список вариантов, предлагаемых корпорацией Майкрософт и другими поставщиками:

| Компонент        | Варианты Azure                                                                                                                                  |
|------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| z/OS             | Windows, Linux или UNIX                                                                                                                      |
| CICS             | Службы Azure, предлагаемые Micro Focus, Oracle, GT Software (Fujitsu), TmaxSoft, Raincode и NTT Data. Также можно переписать решение с использованием Kubernetes. |
| IMS              | Службы Azure, предлагаемые Micro Focus и Oracle                                                                                  |
| Assembler        | Службы Azure, предлагаемые Raincode и TmaxSoft; COBOL, C или Java; сопоставление с функциями операционной системы               |
| JCL              | JCL, PowerShell или другие средства для работы со скриптами                                                                                                   |
| COBOL            | COBOL, C или Java                                                                                                                            |
| Natural          | Natural, COBOL, C или Java                                                                                                                  |
| FORTRAN и PL/I | FORTRAN, PL/I, COBOL, C или Java                                                                                                           |
| REXX и PL/I    | REXX, PowerShell или другие средства для работы со скриптами                                                                                                  |

## <a name="migrate-databases-and-data"></a>Перенос баз данных и данных

Как правило при переносе приложений требуется повторное размещение приложения на уровне данных. Вы можете перенести базы данных SQL Server, базы данных с открытым кодом и другие реляционные базы данных в полностью управляемые решения Azure, например [Управляемый экземпляр Базы данных SQL Azure](/azure/sql-database/sql-database-managed-instance), [Базу данных Azure для PostgreSQL](/azure/postgresql/overview) и [Базу данных Azure для MySQL](/azure/mysql/overview), с помощью [Azure Database Migration Service](/azure/dms/dms-overview).

К примеру, вы можете перенести приложение, если на уровне данных мейнфрейма используются:

- База данных IBM DB2 или IMS. Воспользуйтесь базой данных Azure SQL, SQL Server, DB2 LUW или Oracle Database в Azure.

- VSAM и другие неструктурированные файлы. Воспользуйтесь неструктурированными файлами с индексно-последовательным методом доступа (ISAM) для Azure SQL, SQL Server, DB2 LUW или Oracle.

- Группы даты создания (GDG). Перенесите в Azure файлы, для которых используется соглашение об именовании и расширение имени файла, обеспечивающие аналогичные функциональные возможности для GDG.

На уровне данных IBM есть несколько основных компонентов, которые также необходимо перенести. Например, при переносе базы данных переносится и коллекция c данными в пулах, каждый из которых содержит "dbextents" — наборы данных VSAM z/OS. При миграции нужно указать каталог, определяющий расположения данных в пулах носителей. Кроме того, в плане миграции необходимо учесть журнал базы данных с записями операций, выполняемых в базе данных. База данных может иметь один, два (двойных или альтернативных) или четыре (двойных и альтернативных) журнала.

При переносе базы данных необходимо также перенести следующие компоненты:

- Диспетчер базы данных. Предоставляет доступ к данным в базе данных. Диспетчер базы данных запускается в отдельном разделе в среде z/OS.

- Обработчик запросов приложения. Принимает запросы от приложений перед их передачей на сервер приложений.

- Адаптер подключенного ресурса. Включает компоненты обработчика запросов приложения для использования в транзакциях CICS.

- Адаптер ресурса пакетной службы. Реализует компоненты обработчика запросов приложения для пакетных приложений z/OS.

- Интерактивный SQL (ISQL). Запускается как приложение и интерфейс CICS, позволяя пользователям вводить инструкции или команды оператора SQL.

- Приложение CICS. Выполняется под управлением CICS с использованием доступных ресурсов и источников данных в CICS.

- Приложение пакетной службы. Запускает логику процесса без интерактивного взаимодействия с пользователями, например, для создания массовых обновлений данных или отчетов из базы данных.

## <a name="optimize-scale-and-throughput-for-azure"></a>Оптимизация масштаба и пропускной способности для Azure

В общем случае, при расширении облака масштаб мейнфреймов увеличивается. Чтобы оптимизировать масштаб и пропускную способность приложений с использованием мейнфреймов, запускаемых в Azure, важно понимать, как мейнфреймы позволяют разделять и изолировать приложения. Чтобы изолировать ресурсы определенного приложения и управлять ими в одном экземпляре, в мейнфрейме z/OS используется функция логических разделов (LPARS).

Например, в мейнфрейме один логический раздел (LPAR) используется для региона CICS со связанными программами COBOL, а второй — для DB2. Дополнительные логические разделы часто используются для сред разработки, тестирования и промежуточных сред.

В Azure для этой цели принято использовать отдельные виртуальные машины. В архитектурах Azure на уровне приложений обычно развертываются виртуальные машины: один набор виртуальных машин применяется на уровне данных, и другой набор используется для разработки и подобных целей. Каждый уровень обработки можно оптимизировать с использованием наиболее подходящего типа виртуальных машин и компонентов для этой среды.

Кроме того, на каждом уровне можно применять соответствующие службы аварийного восстановления. Например, для виртуальных машин, используемых в рабочей среде, и виртуальных машин с базами данных, возможно, потребуется "горячее" или "теплое" восстановление. А виртуальные машины для разработки и тестирования поддерживают "холодное" восстановление.

На следующем рисунке показан возможный процесс развертывания в Azure с помощью основного и дополнительного веб-сайта. На основном сайте виртуальные машины для производственной, подготовительной и тестовой версии приложения развертываются с высоким уровнем доступности. Дополнительный сайт используется для резервного копирования и аварийного восстановления.

![Возможный процесс развертывания в Azure с помощью основного и дополнительного сайта](../../_images/mainframe-migration/migration-backup-DR.png)

## <a name="perform-a-staged-mainframe-to-azure"></a>Поэтапный перенос приложения с мейнфрейма в Azure

Возможно, перенести решения с мейнфрейма в Azure потребуется *поэтапно*. При этом одни приложения переносятся в первую очередь, а другие временно или навсегда остаются на мейнфрейме. Как правило, для реализации этого подхода необходимы системы, которые позволяют приложениям и базам данных взаимодействовать между мейнфреймом и Azure.

Распространенный сценарий — перенос приложения в Azure с сохранением данных, используемых приложением, на мейнфрейме. Чтобы предоставить приложениям в Azure доступ к данным с мэйнфрейма, используется специальное программное обеспечение. К счастью, широкий выбор решений обеспечивает интеграцию между Azure и существующими средами мейнфреймов, поддержку гибридных сценариев и поэтапную миграцию. Партнеры корпорации Майкрософт, независимые поставщики программного обеспечения и системные интеграторы, смогут помочь вам реализовать такое взаимодействие.

Один из вариантов — [Microsoft Host Integration Server](https://docs.microsoft.com/host-integration-server/) (HIS). Это решение, которое предоставляет архитектуру распределенной реляционной базы данных (DRDA), необходимую для приложений в Azure, чтобы обеспечить доступ к данным в базе DB2, которая остается на мейнфрейме. Для других вариантов интеграции мэйнфреймов в Azure используются решения IBM, Attunity, Codit и других поставщиков, а также решения с открытым исходным кодом.

## <a name="partner-solutions"></a>Решения партнеров

Если вы планируете миграцию с мэйнфреймов, вам поможет партнерская экосистема.

Azure предоставляет проверенную, высокодоступную и масштабируемую инфраструктуру для систем, которые сейчас работают на мейнфреймах. Некоторые рабочие нагрузки можно перенести без особых трудностей. Другие рабочие нагрузки, зависящие от программного обеспечения устаревших систем, такие как CICS и IMS, можно со временем перенести в Azure с помощью решений партнеров. Какой бы из вариантов вы не выбрали, корпорация Майкрософт и ее партнеры готовы помочь вам оптимизировать решение для Azure, сохранив функциональные возможности программного обеспечения системы мейнфрейма.

См. дополнительные сведение о критериях выбора решений от партнеров в статье [Platform Modernization Alliance](https://www.platformmodernization.org/pages/mainframe.aspx) (Альянс модернизации платформы).

## <a name="learn-more"></a>Подробнее

Для получения дополнительных сведений см. следующие ресурсы:

- [Приступая к работе с Azure](/azure)

- [Альянс по модернизации платформы: перенос решений для мейнфреймов](https://www.platformmodernization.org/pages/mainframe.aspx)

- [Развертывание IBM DB2 pureScale в Azure](https://azure.microsoft.com/resources/deploy-ibm-db2-purescale-on-azure)

- [Документация по Host Integration Server (HIS)](https://docs.microsoft.com/host-integration-server/)