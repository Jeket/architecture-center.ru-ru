---
title: CAF. Рациональное использование цифровых активов
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: Оцените цифровые ресурсы, чтобы определить лучший способ для их размещения в облаке.
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.openlocfilehash: 02189c9edcbfea0a55fe69a53bf610e85470a4d0
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55897207"
---
# <a name="rationalize-the-digital-estate"></a>Рациональное использование цифровых активов

Рационализация облака — это процесс оценки ресурсов, который определяет наилучший подход к их размещению в облаке. Сразу после определения [подхода](approach.md) и вычисления [данных инвентаризации](inventory.md) можно приступать к рационализации облака. В статье с [5 принципами рационализации](5-rs-of-rationalization.md) описываются наиболее распространенные варианты рационализации.

## <a name="traditional-view-of-rationalization"></a>Традиционное представление о рационализации

Рационализацию можно легко понять, если визуализировать традиционный процесс рационализации в виде сложного дерева принятия решений. Каждый ресурс в цифровом активе проходит через процесс, который приводит к одному из пяти ответов (5 принципов). Для небольших активов этот процесс работает хорошо. Для активов большего размера он не очень эффективен и может привести к значительным задержкам. Давайте рассмотрим процесс, чтобы узнать о причинах этого. Затем мы предоставим более эффективную модель.

**Инвентаризация**. Для выполнения полной рационализации с помощью традиционных моделей требуется тщательная инвентаризация ресурсов, включая приложения, программное обеспечение, аппаратное обеспечение, операционные системы и метрики производительности системы.

**Количественный анализ**. В дереве принятия решений количественные вопросы составляют первый слой решений. Часто задаваемые вопросы: используется ли ресурс сегодня; если да, оптимизирован ли он и имеет ли подходящий размер; какие зависимости существуют между ресурсами. Эти вопросы очень важны для классификации данных инвентаризации.

**Качественный анализ**. Для следующего набора решений требуется проведение качественного анализа со стороны человека. Часто эти вопросы являются уникальными для решения, и на них могут ответить только заинтересованные лица и опытные пользователи. Эти решения принимаются там, где процесс обычно задерживается, значительно замедляя ход выполнения. На этот анализ обычно уходит 40&ndash;80 часов ЭПЗ на каждое приложение. Инструкции по созданию списка вопросов качественного анализа см. в статье о [подходе к планированию цифровых активов](approach.md).

**Решение рационализации**. На основе качественных и количественных данных опытные команды по рационализации создают четкие решения. К сожалению, команды с большим опытом в области рационализации обходятся дорого, или на их обучение уходит много месяцев.

## <a name="rationalization-at-enterprise-scale"></a>Рационализация в корпоративном масштабе

Если эти трудозатраты занимают много времени и звучат пугающе для цифрового актива с 50 виртуальными машинами, представьте себе, сколько усилий необходимо приложить для преобразования бизнеса ускоренными темпами в среде с тысячами виртуальных машин и сотнями приложений. Человеческие трудозатраты могут легко превысить 1500 часов ЭПС и 9 месяцев планирования.

Несмотря на то что полная рационализация — это конечное состояние и отличное направление, в котором необходимо двигаться, она редко обеспечивает высокий коэффициент рентабельности инвестиций относительно времени и усилий.

Если рационализация имеет важное значение для принятия финансовых решений, стоит рассмотреть возможность получения профессиональных услуг от организации, которая специализируется на рационализации облака, чтобы ускорить процесс. Даже в таком случае для полной рационализации может потребоваться много ресурсов и трудозатрат, что может привести к задержке преобразования или отсрочить получение бизнес-результатов.

В оставшейся части этой статьи описывается альтернативный способ, известный как добавочная рационализация.

## <a name="incremental-rationalization"></a>Добавочная рационализация

Полная рационализация большого цифрового актива сопряжена с рисками, и ее сложность может вызвать задержки. При использовании подхода добавочной рационализации отложенные решения поэтапно распределяют нагрузку на бизнес, чтобы снизить риск препятствий. Со временем при таком подходе создается органическая модель для разработки процессов и получения опыта, необходимого для более эффективного принятия решений рационализации надлежащего качества.

### <a name="inventory-reduce-discovery-data-points"></a>Инвентаризация: уменьшение точек данных обнаружения

Очень немногие организации вкладывают средства, время и тратят энергию на то, чтобы поддерживать точную инвентаризацию полного цифрового актива в реальном времени. Потери, кражи, циклы обновления и адаптация сотрудников часто оправдывают подробное отслеживание ресурсов устройств пользователей. Тем не менее рентабельность инвестиций в поддержание правильной инвентаризации серверов и приложений в традиционном локальном центре обработки данных часто находится на низком уровне. Большинству ИТ-организаций нужно решать другие более важные вопросы, чем отслеживание использования фиксированных ресурсов в центре обработки данных.

В преобразовании в облако инвентаризация непосредственно сопоставляется с операционными расходами. Для правильного планирования необходимы точные данные инвентаризации. К сожалению, текущие варианты сканирования среды могут отложить решения на недели или месяцы для сканирования и каталогизации полной инвентаризации. К счастью, есть несколько приемов, ускоряющих сбор данных.

Сканирование на основе агента — это наиболее часто встречающаяся задержка. Надежные данные, необходимые для традиционной рационализации, часто зависят от данных, которые можно собрать только с помощью агента, работающего в каждом ресурсе. Эта зависимость от агентов часто замедляет ход выполнения, так как могут потребоваться отзывы о функциях безопасности, эксплуатации и администрирования.

В процессе добавочной рационализации для начального обнаружения может использоваться решение без агентов, чтобы ускорить ранние решения. В зависимости от уровня сложности в среде решение на основе агентов все равно может потребоваться, но его можно удалить из критического пути к изменениям в бизнесе.

### <a name="quantitative-analysis-streamline-decisions"></a>Количественный анализ: упрощение решений

Независимо от подхода к обнаружению инвентаризации количественный анализ может предоставить ряд начальных решений и предположений. Это особенно верно, когда организация пытается определить первую рабочую нагрузку или если целью рационализации является сравнение затрат высокого уровня. В процессе добавочной рационализации команды облачной стратегии и внедрения облака ограничивают [5 принципов рационализации](5-rs-of-rationalization.md) до двух кратких решений и применяют только эти количественные факторы, упрощая анализ и уменьшая объем начальных данных, необходимых для изменения.

Например, если организация находится на середине пути миграции IaaS в облако, можно предположить, что большинство рабочих нагрузок будут удалены или перемещены.

### <a name="qualitative-analysis-temporary-assumptions"></a>Качественный анализ: временные предположения

Благодаря уменьшению числа возможных результатов проще достичь первоначального решения о будущем состоянии ресурса. При сокращении числа вариантов также сокращается число вопросов о бизнесе на этой ранней стадии.

В продолжение приведенного выше примера, если варианты ограничены до повторного размещения или прекращения использования, на самом деле есть только один вопрос к бизнесу во время начальной рационализации, а именно: следует ли прекратить использование.

"Анализ предполагает, что нет пользователей, которые активно используют этот ресурс. Это верно, или мы пропустили что-нибудь?" Для такого двоичного вопроса, как правило, гораздо проще выполнить качественный анализ.

Этот упрощенный подход формирует базовые показатели, финансовые планы, стратегию и направление. В последующих действиях каждый ресурс будет проходить через дальнейшую рационализацию и качественный анализ, чтобы оценить другие варианты. Все предположения, сделанные в этой начальной рационализации, будут протестированы перед реализацией.

## <a name="challenging-assumptions"></a>Проверка предположений на прочность

Результат предыдущего раздела — приблизительная рационализация с предположениями. Теперь пришла пора проверить на прочность некоторые из этих предположений.

### <a name="retiring-assets"></a>Прекращение использования активов

В традиционной локальной среде размещение небольших неиспользуемых ресурсов редко оказывает значительное влияние на ежегодные затраты. За некоторыми исключениями, экономия денежных средств, связанная с удалением и прекращением использования этих ресурсов, компенсируется трудозатратами ЭПЗ, необходимыми для анализа и прекращения использования фактического ресурса.

Тем не менее при переходе на облачную модель учета прекращение использования ресурсов позволяет существенно сократить годовые операционные расходы и трудозатраты на первоначальную миграцию.

Нередко в организациях по завершении количественного анализа прекращают использовать 20 % или более цифровых активов. Прежде чем принимать решение по этому действию, рекомендуется выполнить дальнейший качественный анализ. После подтверждения прекращение использования этих ресурсов может привести к первой победе в вопросе рентабельности инвестиций при миграции в облако. Во многих случаях это один из самых значительных факторов сокращения затрат. Таким образом предполагается, что команда облачной стратегии контролирует проверку и прекращение использования ресурсов в параллельном режиме, чтобы создать этап процесса миграции и обеспечить раннюю финансовую победу.

### <a name="program-adjustments"></a>Корректировки программы

Компания редко приступает к только одному преобразованию. Выбор между снижением затрат, ростом рынка и новыми потоками прибыли редко является двоичным решением. Таким образом предполагается, что команда облачной стратегии сотрудничает с ИТ, чтобы определить ресурсы в параллельных трудозатратах преобразования, которые находятся за пределами основного пути преобразования.

В примере миграции IaaS, используемом в этой статье:

- Попросите команду DevOps определить ресурсы, которые уже являются частью автоматизации развертывания, и удалить из основного плана миграции.

- Попросите команду по работе с данными и команду по научным исследованиям и разработке определить ресурсы, которые питают новые потоки доходов, и удалить их из основного плана миграции.

Этот ориентированный на программу качественный анализ можно быстро выполнить, и он создает выравнивание по нескольким элементам невыполненной работы по миграции.

Некоторые ресурсы по-прежнему необходимо рассматривать как ресурсы повторного размещения в течение некоторого времени, что влечет за собой поэтапный переход в более позднюю рационализацию после первоначальной миграции.

## <a name="selecting-the-first-workload"></a>Выбор первой рабочей нагрузки

Реализация первой рабочей нагрузки крайне важна для тестирования и обучения. Это первая возможность для демонстрации и создания профессиональной компетенции.

### <a name="business-criteria"></a>Критерии бизнеса

Определите рабочую нагрузку, которую поддерживает член подразделения команды облачной стратегии, чтобы обеспечить прозрачность бизнеса. Желательно выбрать того, в котором команда заинтересована и у кого есть надежная мотивация для перемещения в облако.

### <a name="technical-criteria"></a>Технические критерии

Выберите рабочую нагрузку, у которой минимальное количество зависимостей и которую можно переместить как небольшую группу ресурсов. Для упрощения проверки рекомендуется выбрать рабочую нагрузку с определенным путем тестирования.

Первая рабочая нагрузка часто развертывается в экспериментальной среде без производственной мощности и системы управления. Очень важно выбрать рабочую нагрузку, которая не взаимодействует с защищенными данными.

### <a name="qualitative-analysis"></a>Качественный анализ

Команды по внедрению облака и облачной стратегии могут работать совместно, чтобы проанализировать эту небольшую рабочую нагрузку. При этом создается управляемая возможность создавать и тестировать критерии качественного анализа. Благодаря небольшому размеру нагрузки можно опросить затронутых пользователей, чтобы выполнить подробный качественный анализ за неделю или меньше. Сведения о распространенных факторах качественного анализа см. в разделе о конкретной цели рационализации в статье о [5 принципах рационализации](5-rs-of-rationalization.md).

### <a name="migration"></a>Миграция

Одновременно с дальнейшей рационализацией команда по внедрению облака может начать миграцию небольшой рабочей нагрузки, чтобы расширить обучение в следующих ключевых областях:

- Совершенствование навыков работы с платформой поставщика облачных служб.
- Определение основных служб (и стандартов Azure) в соответствии с долгосрочной целью.
- Получение лучшего представления о необходимости дальнейшего развития производственных процессов на пути преобразования.
- Изучение всех рисков, присущих бизнесу, и допустимости отклонения для этих рисков.
- Установка базового или минимально приемлемого продукта для управления на основе допустимых рисков для бизнеса.

## <a name="release-planning"></a>Планирование выпуска

Когда команда по внедрению облака выполняет миграцию или реализацию первой рабочей нагрузки, команда облачной стратегии может начать расстановку приоритетов для оставшихся приложений и рабочих нагрузок.

### <a name="power-of-ten"></a>Метод "Сила десяти"

Традиционный подход к рационализации пытается объять необъятное. К счастью, для начала пути преобразования план для каждого приложения часто не требуется. В добавочной модели метод "Сила десяти" предоставляет хорошую отправную точку. В этой модели команда по облачной стратегии выбирает первые десять приложений, которые необходимо перенести. Эти десять рабочих нагрузок должны представлять собой смесь простых и сложных рабочих нагрузок.

### <a name="building-the-first-backlogs"></a>Создание первых невыполненных работ

Команды по внедрению облака и облачной стратегии могут совместно работать над качественным анализом для первых десяти рабочих нагрузок. При этом создается первая приоритетная невыполненная работа по миграции и приоритетная невыполненная работа по выпускам. Такой подход позволяет командам выполнить итерацию подхода и предоставляет достаточно времени для создания соответствующего процесса для качественного анализа.

### <a name="maturing-the-process"></a>Совершенствование процесса

Как только две команды разработчиков согласуют критерии качественного анализа, оценка может стать задачей, выполняемой в каждой итерации. Для достижения консенсуса по критериям оценки обычно требуется 2&ndash;3 выпуска.

После перемещения оценки в процессы добавочного выполнения миграции команда по внедрению облака может выполнять итерацию оценки и архитектуры быстрее. На этом этапе команда по облачной стратегии также абстрагируется, снижая трату времени. Это также позволяет этой команде сосредоточиться на определении приоритетности приложений, которые еще не находятся в определенном выпуске, тем самым обеспечивая точное соответствие изменяющимся рыночным условиям.

Не все приоритетные приложения будут готовы к миграции. Последовательность, скорее всего, изменится, так как команда выполняет более глубокий качественный анализ и обнаруживает бизнес-события и зависимости, которые инициируют повторное определение приоритетности невыполненных работ. Некоторые выпуски могут группировать небольшое количество рабочих нагрузок. Другие могут просто содержать одну рабочую нагрузку.

Команда по внедрению облака, скорее всего, будет выполнять итерации, которые не производят полную миграцию рабочей нагрузки. Чем меньше рабочая нагрузки и чем меньше число зависимостей, тем выше вероятность того, что рабочая нагрузка поместится в один спринт или итерацию. По этой причине рекомендуется, чтобы первые несколько приложений в списке невыполненных работ выпуска были небольшими и содержали небольшое число внешних зависимостей.

## <a name="end-state"></a>Конечное состояние

Со временем объединение результатов работы команд по внедрению облака и облачной стратегии завершит полную рационализацию инвентаризации. Тем не менее этот добавочный подход позволяет командам постоянно ускорять процесс рационализации. Кроме того, благодаря ему преобразование может помочь быстрее достичь ощутимых бизнес-результатов без больших трудозатрат на предварительный анализ.

В некоторых случаях финансовая модель может быть слишком маленькой для принятия решения без дополнительной рационализации. В таких случаях может потребоваться более традиционный подход к рационализации.

## <a name="next-steps"></a>Дополнительная информация

Результат трудозатрат по рационализации — это приоритетные невыполненные работы всех ресурсов, на которые повлияет выбранное преобразование. Эта невыполненная работа теперь может выступать в качестве основы для модели затрат облачных служб.

> [!div class="nextstepaction"]
> [Сопоставление моделей затрат на цифровые активы](calculate.md)