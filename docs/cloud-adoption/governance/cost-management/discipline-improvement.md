---
title: CAF. Улучшения по дисциплине управления затратами
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Улучшения по дисциплине управления затратами
author: BrianBlanchard
ms.openlocfilehash: 34975d195a95b1a85ada96efe8c76a6138385ec1
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55902572"
---
# <a name="cost-management-discipline-improvement"></a>Улучшения по дисциплине управления затратами

Дисциплина "Управление затратами" нацелена на анализ ключевых бизнес-рисков, связанных с расходами на размещение облачных рабочих нагрузок. В структуре пяти дисциплин управления облаком управление затратами отвечает за контроль затрат и использование облачных ресурсов с целью создания и сопровождения планового цикла расходов.

В этой статье описаны задачи, которые компания может выполнить для развития и совершенствования дисциплины управления затратами. Эти задачи можно разделить на этапы планирования, создания, внедрения и реализации облачного решения. Итеративное выполнение этих этапов позволяет разработать [поэтапный подход к системе управления облаком](../journeys/overview.md#an-incremental-approach-to-cloud-governance).

![Четыре этапа внедрения](../../_images/adoption-phases.png)

*Рис. 1. Этапы внедрения инкрементального подхода к системе управления облаком.*

Невозможно учесть в одном документе все требования любых компаний. Поэтому в этой статье описываются минимально требуемые и возможные действия для каждого этапа в процессе совершенствования системы управления. Начальной целью этих действий является создание [MVP политики](../journeys/overview.md#an-incremental-approach-to-cloud-governance) и разработка платформы для инкрементального развития этой политики. Команде управления облаком необходимо решить, какие потребуются инвестиции в эти действия, чтобы улучшить возможности системы управления затратами.

> [!CAUTION]
> Описанные в этой статье минимальные или возможные действия не согласованы с какими-либо корпоративными политиками или нормативными требованиями третьих сторон. Это руководство призвано облегчить обсуждения по согласованию модели управления облаком с обоими требованиями.

## <a name="planning-and-readiness"></a>Планирование и готовность

На этом этапе развития системы управления ликвидируется разрыв между бизнес-показателями и стратегиями практических действий. В ходе этого процесса руководство определяет конкретные метрики, сопоставляет их с цифровыми активами и начинает планировать общий процесс миграции.

**Минимальные рекомендуемые действия:**

* Оцените возможности для создания [цепочки инструментов управления затратами](toolchain.md).
* Подготовьте черновой вариант рекомендаций по архитектуре и передайте его основным заинтересованным лицам.
* Проведите обучение и привлеките к участию людей и команды, на работу которых повлияет создание рекомендаций по архитектуре.

**Возможные действия:**

* Примите решения по бюджету, которые поддержат коммерческое обоснование вашей облачной стратегии.
* Проверьте метрики обучения, которые вы используете для отчетов об успешном распределении финансирования.
* Определитесь с целевой моделью учета в облаке, которая влияет на методы учета затрат на облачные решения.
* Ознакомьтесь с планом цифровых ресурсов и проверьте точные прогнозы по затратам.
* Оцените варианты приобретения, например выберите "оплату по мере использования" или предварительную оплату по Соглашению Enterprise.
* Сопоставьте бизнес-цели с выделенными бюджетами и скорректируйте бюджетные планы при необходимости.
* Разработайте механизм отчетности по целям и бюджетам, чтобы передавать информацию заинтересованным лицам в технических и производственных подразделениях в конце каждого цикла затрат.

## <a name="build-and-pre-deployment"></a>Сборка и предварительное развертывание

Для успешной миграции среды нужно выполнить ряд технических и нетехнических требований. Этот процесс нацелен на решения, подготовленность и базовую инфраструктуру, которые должны быть готовы до миграции.

**Минимальные рекомендуемые действия:**

* Создайте [цепочку инструментов управления затратами](toolchain.md), развернув ее в среде подготовки к развертыванию.
* Подготовьте обновленный вариант рекомендаций по архитектуре и передайте его ключевым заинтересованным лицам.
* Разработайте обучающие материалы и документы, информационные сообщения, поощрения и другие программы, которые помогут пользователям освоиться.
* Определите, соответствуют ли требования к закупкам выделенным бюджетам и поставленным целям.

**Возможные действия:**

* Согласуйте план бюджета со [стратегией подписки](../../decision-guides/subscriptions/overview.md), которая определяет базовую модель ответственности.
* Примените [стратегию согласованности ресурсов](../../decision-guides/resource-consistency/overview.md) для постепенного внедрения рекомендаций по архитектуре и затратам.
* Определите наличие аномально высоких затрат, которые могут повлиять на планы внедрения и миграции.

## <a name="adopt-and-migrate"></a>Внедрение и миграция

Миграция представляет собой последовательный процесс перемещения, тестирования и внедрения приложений или рабочих нагрузок в существующей цифровой инфраструктуре.

**Минимальные рекомендуемые действия:**

* Перенесите ваши [цепочки инструментов управления затратами](toolchain.md) из среды подготовки развертывания в рабочую среду.
* Подготовьте обновленный вариант рекомендаций по архитектуре и передайте его ключевым заинтересованным лицам.
* Разработайте обучающие материалы и документы, информационные сообщения, поощрения и другие программы, которые помогут пользователям освоиться.

**Возможные действия:**

* Внедрите модель учета в облаке.
* Убедитесь, что все бюджеты правильно отражают фактические расходы на каждый этап выпуска, а при необходимости внесите изменения.
* Отслеживайте изменения в планах бюджета и согласуйте с заинтересованными лицами дополнительные одобрения.
* Внесите в документ рекомендаций по архитектуре изменения, отражающие фактические расходы.

## <a name="operate-and-post-implementation"></a>Эксплуатация и действия после реализации

Когда преобразование завершится, системы управления и эксплуатации должны сохраняться для поддержки естественного жизненного цикла приложений или рабочих нагрузок. Этот этап развития системы управления посвящен действиям, которые обычно следуют за внедрением решения, когда цикл трансформации становится более стабильным.

**Минимальные рекомендуемые действия:**

* Измените параметры [цепочки инструментов управления затратами](toolchain.md) с учетом изменений требований к управлению затратами в организации.
* Постарайтесь внедрить автоматизацию уведомлений и отчетов по фактическим расходам.
* Скорректируйте рекомендации по архитектуре, чтобы управлять дальнейшими процессами внедрения.
* Периодически проводите обучение для задействованных команд, чтобы обеспечить соблюдение рекомендаций по архитектуре.

**Возможные действия:**

* Ежеквартально выполняйте проверку облачного бизнеса, чтобы получить данные о созданной пользе для бизнеса и соответствующих затратах.
* Ежеквартально корректируйте планы с учетом изменений в фактических расходах.
* Оцените соответствие финансовых показателей отчету о доходах и расходах по подпискам для бизнес-единиц.
* Ежемесячно оценивайте методы информирования заинтересованных лиц о ценности и затратах.
* Принимайте меры по простаивающим ресурсам и принимайте решения о продолжении их поддержки.
* Выявляйте несогласованность между плановыми и фактическими расходами, а также любые аномалии.
* Помогите командам внедрения облака и команде разработчиков облачной стратегии разобраться в этих аномалиях и устранить их.

## <a name="next-steps"></a>Дополнительная информация

Теперь, когда вы понимаете концепцию стратегии управления удостоверениями в облаке, изучите [цепочки инструментов управления затратами](toolchain.md), чтобы выбрать наиболее подходящие средства и функции Azure, которые вам потребуются при развитии дисциплины управления затратами на платформе Azure.

> [!div class="nextstepaction"]
> [Цепочка инструментов управления затратами для Azure](toolchain.md)