---
title: Руководство по разработке на R в Azure. Программирование на языке R | Документация Майкрософт
description: Статья содержит общие сведения о различных способах, с помощью которых специалисты по обработке и анализу данных могут использовать имеющиеся навыки работы с языком R в Azure. Azure предлагает множество служб, разработчики R можно использовать для расширения своих рабочих нагрузок обработки и анализа данных в облаке.
services: machine-learning
author: AnalyticJeremy
ms.service: machine-learning
ms.workload: data-services
ms.devlang: R
ms.topic: article
ms.date: 03/19/2019
ms.author: jepeach
ms.openlocfilehash: a14c8ce76f78baa7274f22b939eb28cb025ef87e
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2019
ms.locfileid: "58888119"
---
# <a name="r-developers-guide-to-azure"></a>Руководство по разработке на R в Azure

<img src="../images/logo_r.svg" alt="R logo" align="right" width="200" />

Многие специалисты по обработке и анализу данных, работающие с растущими объемами данных, ищут способы использования потенциала облачных вычислений для анализа.  Статья содержит общие сведения о различных способах, с помощью которых специалисты по обработке и анализу данных могут использовать имеющиеся навыки работы с [языком R](https://www.r-project.org) в Azure.

Корпорация Майкрософт полностью внедрила язык R как первоклассное средство для специалистов по обработке и анализу данных.  Предоставляя разработчикам R множество различных способов запуска кода в Azure, компания позволяет специалистам по обработке и анализу данных расширить свои рабочие нагрузки обработки и анализа данных в облако при работе над крупномасштабными проектами.

Рассмотрим различные варианты и наиболее привлекательные сценарии для каждого из них.

## <a name="azure-services-with-r-language-support"></a>Службы Azure с поддержкой языка R

В этой статье рассматриваются следующие службы Azure, поддерживающие язык R:

|Service                                                          |ОПИСАНИЕ                                                                       |
|-----------------------------------------------------------------|----------------------------------------------------------------------------------|
|[Виртуальная машина для обработки и анализа данных](#data-science-virtual-machine)    |Настраиваемая виртуальная машина, которую можно использовать в качестве рабочей станции обработки и анализа данных или настраиваемого целевого объекта вычислений|
|[Службы машинного обучения в HDInsight](#ml-services-on-hdinsight)            |Система на основе кластера для выполнения анализа R в больших наборах данных на нескольких узлах   |
|[Azure Databricks](#azure-databricks)                            |Среда Spark для совместной работы, поддерживающая язык R и другие языки               |
|[Студия машинного обучения Azure](#azure-machine-learning-studio)  |Выполнение пользовательских R-сценариев в экспериментах машинного обучения Azure                      |
|[Пакетная служба Azure](#azure-batch)                                      |Предоставляет различные варианты для выполнения кода R на многих узлах в кластере без лишних затрат|
|[Azure Notebook](#azure-notebooks)                              |Бесплатная облачная версия записных книжек Jupyter                  |
|[Базы данных SQL Azure](#azure-sql-database)                        |Выполнение R-сценариев в ядре СУБД SQL Server                            |

## <a name="data-science-virtual-machine"></a>Виртуальная машина для обработки и анализа данных

[Виртуальная машина для обработки и анализа данных](https://docs.microsoft.com/azure/machine-learning/data-science-virtual-machine/overview) — это образ виртуальной машины на облачной платформе Microsoft Azure, специально созданный и настроенный для обработки и анализа данных. В нем есть множество популярных средств обработки и анализа данных, включая следующие:

* [Microsoft R Open](https://mran.microsoft.com/open/)
* [Сервер машинного обучения Майкрософт.](https://docs.microsoft.com/machine-learning-server/what-is-machine-learning-server)
* [RStudio Desktop](https://www.rstudio.com/products/rstudio/#Desktop)
* [RStudio Server](https://www.rstudio.com/products/rstudio/#Server)

Виртуальную машину для обработки и анализа данных (DSVM) можно подготавливать с операционной системой Windows или Linux.  DSVM можно использовать двумя способами: как интерактивную рабочую станцию или как вычислительную платформу для пользовательского кластера.

### <a name="as-a-workstation"></a>Как рабочая станция

Если вы хотите начать работу с R в облаке быстро и легко, это наилучшее решение.  Среда знакома всем, кто работал с R на локальной рабочей станции.  Тем не менее, вместо использования локальных ресурсов среда R выполняется на виртуальной машине в облаке.  Если ваши данные уже хранятся в Azure, это дополнительное преимущество, так как ваши R-сценарии будут выполняться "ближе к данным". Вместо передачи данных через Интернет доступ к ним можно получить через внутреннюю сеть Azure, что уменьшает время доступа.

DSVM может быть особенно удобна для небольших команд разработчиков R.  Вместо того чтобы вкладывать средства в мощные рабочие станции для каждого разработчика и требовать от участников команды синхронизировать версии различных пакетов программного обеспечения, которые они будут использовать, каждый разработчик может запустить экземпляр DSVM, когда это необходимо.

### <a name="as-a-compute-platform"></a>Как вычислительная платформа

Помимо использования в качестве рабочей станции DSVM также используется как эластично масштабируемая вычислительная платформа для проектов R.  С помощью [ `AzureDSVM` ](https://github.com/Azure/AzureDSVM) пакет R, можно программно управлять Создание и удаление экземпляров DSVM.  Вы можете организовать экземпляры в кластер и развернуть распределенный анализ таким образом, чтобы он выполнялся в облаке.  Всем процессом можно управлять с помощью кода R, выполняемого на локальной рабочей станции.

Дополнительные сведения о DSVM, см. в разделе [введение в Azure виртуальную машину для обработки данных для Linux и Windows](https://docs.microsoft.com/azure/machine-learning/data-science-virtual-machine/overview).

## <a name="ml-services-on-hdinsight"></a>Службы машинного обучения в HDInsight

[Службы машинного обучения Microsoft](https://docs.microsoft.com/azure/hdinsight/r-server/r-server-overview) позволяют специалистам по обработке и анализу данных, статистикам и R-программистам получать доступ к масштабируемым, распределенным методам аналитики в HDInsight по требованию.  Это решение предоставляет новейшие возможности для анализа наборов данных практически любого размера, загруженных в хранилище BLOB-объектов Azure или Data Lake Storage, на базе языка R.

Это решение корпоративного уровня, которое позволяет масштабировать код R в кластере.  За счет использования функций в корпорации Майкрософт [`RevoScaleR`](https://docs.microsoft.com/machine-learning-server/r-reference/revoscaler/revoscaler)
пакет, скриптов на языке R в HDInsight можно запустить функций обработки данных в параллельном режиме на множество узлов в кластере.  Это позволяет R обрабатывать данные в гораздо больших масштабах, чем с помощью однопоточного R, запущенного на рабочей станции.

Благодаря этой возможности масштабирования службы машинного обучения в HDInsight являются отличным вариантом для разработчиков R, работающих с большими наборами данных.  Они предоставляют гибкую и масштабируемую платформу для выполнения R-сценариев в облаке.

Пошаговые инструкции по созданию кластера служб машинного Обучения, см. в разделе [приступить к работе со службами машинного Обучения в Azure HDInsight](https://docs.microsoft.com/azure/hdinsight/r-server/r-server-get-started).

## <a name="azure-databricks"></a>Azure Databricks

[Azure Databricks](https://azure.microsoft.com/services/databricks/) — это платформа аналитики на основе Apache Spark, оптимизированная для платформы облачных служб Microsoft Azure.  Разработанная с помощью основателей Apache Spark платформа Databricks интегрируется с Azure, предоставляя установку одним щелчком, упрощенные рабочие процессы и интерактивную рабочую область, которая обеспечивает совместную работу специалистов в области обработки и анализа данных, инжиниринга данных и бизнес-аналитики.

Совместная работа в Databricks обеспечивается системой записных книжек платформы.  Пользователи могут создавать, совместно использовать и редактировать записные книжки совместно с другими пользователями системы.  Эти записные книжки позволяют пользователям писать код, который выполняется в кластерах Spark, управляемых в среде Databricks.  Эти записные книжки полностью поддерживают R и предоставляют пользователям доступ к Spark с помощью пакетов `SparkR` и `sparklyr`.

Так как служба Databricks создана на основе Spark и в ней серьезное внимание уделяется совместной работе, платформу часто используют команды специалистов по обработке и анализу данных, совместно работающие над сложным анализом крупных наборов данных.  Так как записные книжки в Databricks поддерживают другие языки помимо R, эта служба будет особенно удобной для команд, в которых аналитики используют разные языки для основной работы.

Статья [What is Azure Databricks?](https://docs.microsoft.com/azure/azure-databricks/what-is-azure-databricks) может предоставить дополнительные сведения о платформе и помочь вам начать работу.

## <a name="azure-machine-learning-studio"></a>Студия машинного обучения Azure

[Студия машинного обучения Azure](https://azure.microsoft.com/services/machine-learning-studio/) — это инструмент для совместной работы, поддерживающий функцию перетаскивания объектов и предназначенный для создания, тестирования и развертывания решений для прогнозной аналитики в облаке.  Он позволяет новым специалистам по обработке и анализу данных создавать и развертывать модели машинного обучения без необходимости написания кода большого объема.

Студия машинного обучения Azure поддерживает R и Python.  R со студией машинного обучения Azure можно использовать двумя способами.

### <a name="custom-r-scripts-in-your-experiments"></a>Пользовательские R-сценарии в экспериментах

Сначала можно расширить возможности обработки данных и машинного обучения Студии машинного обучения путем написания пользовательских R-сценариев.
Несмотря на то что Студия машинного обучения включает широкий набор модулей для подготовки и анализа данных, она не может сопоставить возможности такого современного языка, как R. Таким образом служба позволяет вводить собственные пользовательские R-сценарии в случаях, когда встроенные модули не соответствуют вашим потребностям.

Чтобы воспользоваться этой возможностью, перетащите модуль "Выполнить сценарий R" в свой эксперимент.  Затем с помощью редактора кода в области "Свойства" напишите новый R-сценарий или вставьте имеющийся.  В сценарии можно ссылаться на внешние пакеты R.  Скрипт можно использовать для управления данными или обучения сложных моделей машинного Обучения, которые не являются частью стандартной библиотеки модели студии машинного обучения Azure.

Подробное введение по использованию R в эксперименты в студии машинного Обучения, ознакомьтесь с [Приступая к работе с R, языке программирования в студии машинного обучения Azure](https://docs.microsoft.com/azure/machine-learning/studio/r-quickstart).

### <a name="create-manage-and-deploy-experiments-from-your-local-r-environment"></a>Создание, развертывание экспериментов и управление ими из локальной среды R

Во-вторых, можно использовать R со студией машинного обучения Azure является использование [ `AzureML` ](https://cran.r-project.org/web/packages/AzureML/vignettes/getting_started.html) пакета, отслеживать и контролировать процесс службы "Экспериментирование" с помощью R среду программирования.  Этот пакет, который поддерживается корпорацией Майкрософт, позволяет отправлять и загружать наборы данных и из студии машинного обучения Azure, может запросить эксперименты, публикация R работает как веб-служб, и для запуска R данных через существующие веб-службы и получить выходные данные.

Этот пакет позволяет гораздо проще в использовании студии машинного обучения Azure в качестве платформы для масштабируемого развертывания для кода R.  Вместо того чтобы щелкать и перетаскивать элементы в пользовательском интерфейсе, весь процесс развертывания можно автоматизировать с помощью уже знакомых инструментов.

## <a name="azure-batch"></a>Пакетная служба Azure

Для крупномасштабных заданий R можно использовать [пакетную службу Azure](https://azure.microsoft.com/services/batch/).  Эта служба обеспечивает планирование заданий и управление вычислениями в масштабе облака, поэтому рабочую нагрузку R можно масштабировать в десятках, сотнях или тысячах виртуальных машин.  Так как это универсальная вычислительная платформа, есть несколько способов выполнения заданий R в пакетной службе Azure.

Один вариант — использовать корпорации Майкрософт [ `doAzureParallel` ](https://github.com/Azure/doAzureParallel) пакета.  Этот пакет R является параллельной серверной частью для пакета `foreach`.  Он обеспечивает выполнение каждой итерации цикла `foreach` в параллельном режиме на узле в кластере пакетной службы Azure.  Введение в пакет, см. в записи блога [doAzureParallel: Воспользоваться преимуществами гибкой вычислений Azure непосредственно из сеанса r.](https://azure.microsoft.com/blog/doazureparallel/).

Другой способ выполнения R-сценария в пакетной службе Azure — объединение кода с использованием файла RScript.exe в качестве приложения пакетной службы на портале Azure.  Подробное пошаговое руководство, обратитесь к [R рабочих нагрузок в пакетной службе Azure](https://azure.microsoft.com/blog/r-workloads-on-azure-batch/).

Третий способ — использовать [набор средств для инжиниринга распределенных данных Azure](https://github.com/Azure/aztk) (AZTK), что позволяет подготовить кластеры Spark по запросу с помощью контейнеров Docker в пакетной службе Azure.  Это экономичный способ выполнения заданий Spark в Azure.  При использовании [SparklyR с AZTK](https://github.com/Azure/aztk/wiki/SparklyR-on-Azure-with-AZTK) R-сценарии можно развернуть в облаке легко и без лишних затрат.

## <a name="azure-notebooks"></a>Azure Notebook

[Записные книжки Azure](https://notebooks.azure.com) — это экономичный и эффективный метод для разработчиков R, которые предпочитают работать с записными книжками, чтобы перенести свой код в Azure.  Это бесплатная служба для всех, кто разрабатывает и выполняет код в браузере с помощью [Jupyter](https://jupyter.org/), проекта с открытым исходным кодом, позволяющим объединять текст разметки, исполняемый код и графику на одном холсте.

Бесплатный уровень обслуживания Записных книжек Azure приемлемый для небольших проектов, так как он ограничивает процесс каждой записной книжки 4 ГБ памяти и 1 ГБ наборов данных. Однако если вам требуется большая вычислительная мощность и количество данных, вы можете запускать записные книжки в экземпляре Виртуальной машины для обработки и анализа данных. Дополнительные сведения см. в статье [Manage and configure Azure Notebooks projects — Compute tier](/azure/notebooks/configure-manage-azure-notebooks-projects#compute-tier) (Управление и настройка проектов Записных книжек Azure — уровень вычислений).

## <a name="azure-sql-database"></a>Базы данных SQL Azure

[База данных SQL Azure](https://azure.microsoft.com/services/sql-database/) — это интеллектуальная, полностью управляемая служба реляционной облачной базы данных Microsoft.  Она позволяет использовать все возможности SQL Server без каких-либо сложностей с настройкой инфраструктуры.  Сюда входят [служб машинного обучения в SQL Server](https://docs.microsoft.com/sql/advanced-analytics/what-is-sql-server-machine-learning?view=sql-server-2017), которая является одной из последних дополнений к SQL.

Эта возможность предоставляет внедренную, прогнозную аналитику и модуль обработки и анализа данных, который может выполнять код R в базе данных SQL Server в качестве хранимых процедур, сценариев T-SQL, содержащих инструкции R, или R-кода, содержащего инструкции T-SQL.  Вместо извлечения данных из базы данных и их загрузки в среду R вы загружаете код R непосредственно в базу данных и разрешаете его выполнение вместе с данными.

Хотя Службы машинного обучения являются частью локального сервера SQL Server с 2016 года, они сравнительно новые для Базы данных SQL Azure.  В данный момент они находятся на этапе [ограниченной предварительной версии](https://docs.microsoft.com/sql/advanced-analytics/what-s-new-in-sql-server-machine-learning-services?view=sql-server-2017#azure-sql-database-roadmap), но продолжат свое развитие.


### <a name="next-steps"></a>Дальнейшие действия

* [Выполнение R в Azure с помощью mrsdeploy](https://blog.revolutionanalytics.com/2017/03/running-your-r-code-azure.html)
* [Machine Learning Server в облаке](https://docs.microsoft.com/machine-learning-server/install/machine-learning-server-in-the-cloud)
* [Дополнительные ресурсы для сервера машинного обучения и Microsoft R](https://docs.microsoft.com/machine-learning-server/resources-more)
* [R в Azure](https://github.com/yueguoguo/r-on-azure) — общие сведения о пакетах, средствах и примеры использования языка R в Azure.

---

<sub>Эмблема R- &copy; R Foundation 2016 и используется в соответствии с условиями [лицензии Creative Commons Attribution-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-sa/4.0/).</sub>