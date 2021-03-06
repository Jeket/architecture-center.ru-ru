---
title: Разработка с учетом горизонтального масштабирования
titleSuffix: Azure Application Architecture Guide
description: Архитектура облачных приложений должна обеспечивать возможность горизонтального масштабирования.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 32ea1e7dc732c819783ad2fc06dbcffd75685d23
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243085"
---
# <a name="design-to-scale-out"></a>Разработка с учетом горизонтального масштабирования

## <a name="design-your-application-so-that-it-can-scale-horizontally"></a>Разработка приложения с учетом горизонтального масштабирования

Основное преимущество облака &mdash; это возможность эластичного масштабирования. Оно позволяет использовать строго необходимый объем ресурсов. При увеличении нагрузки приложение развертывается, а когда дополнительная мощность не требуется, возвращается к исходному масштабу. Учитывайте горизонтальное масштабирование при разработке приложения, чтобы можно было добавлять и удалять его экземпляры в зависимости от нагрузки.

## <a name="recommendations"></a>Рекомендации

**Избегайте закрепления экземпляров.** Закрепление экземпляров, или *сходство сеансов*, означает ситуацию, в которой запросы от конкретного клиента всегда направляются к одному и тому же серверу. Закрепление ограничивает возможность горизонтального масштабирования приложения. Например, трафик от наиболее активных пользователей не удастся распределить между несколькими экземплярами. Закрепление возникает по разным причинам, например при сохранении в памяти сведений о состоянии сеанса или при шифровании с использованием ключей конкретного компьютера. Следите за тем, чтобы любой экземпляр приложения всегда мог обработать любой запрос.

**Выявляйте узкие места.** Горизонтальное масштабирование не станет панацеей от любых проблем с производительностью. Например, если узким местом является серверная база данных, дополнительные веб-серверы никак не улучшат ситуацию. Сначала найдите и устраните все узкие места в системе, и лишь затем пытайтесь решить проблему, добавляя новые экземпляры. Возникновение узких мест наиболее вероятно в компонентах, которые работают с отслеживанием состояния.

**Разделите рабочие нагрузки на части в соответствии с требованиями к масштабируемости.**  В приложениях часто выполняется несколько рабочих нагрузок с разными требованиями к масштабированию. Например, в приложении может использоваться общедоступный веб-сайт и отдельный веб-интерфейс администрирования. На общедоступном сайте часто возникают непредвиденные всплески трафика, а нагрузка на сайт администрирования намного ниже и более предсказуема.

**Разгружайте ресурсоемкие задачи.** Задачи, которые требуют значительных ресурсов процессора или ввода-вывода, при возможности следует перемещать в [фоновые задания][background-jobs], чтобы снизить нагрузку на внешний интерфейс, в котором обрабатываются запросы пользователей.

**Используйте встроенные возможности автомасштабирования.** В многие вычислительные службы Azure встроена поддержка автомасштабирования. Если рабочая нагрузка приложения хорошо прогнозируется и регулярно изменяется, используйте масштабирование по расписанию. Например, развертывайте приложение в рабочее время. Если же рабочая нагрузка плохо прогнозируется, используйте для автомасштабирования триггеры по метрикам производительности, таким как загрузка процессора и длина очереди. Также изучите наши [рекомендации по автомасштабированию][autoscaling].

**Используйте агрессивное автомасштабирование для критически важных рабочих нагрузок.** Для критически важных рабочих нагрузок следует предвосхищать повышение интенсивности. Например, в условиях значительной нагрузки можно быстро добавить сразу несколько экземпляров для ускоренной обработки трафика, а затем постепенно уменьшать их количество.

**Учитывайте возможность свертывания при разработке приложения.**  Не забывайте, что при гибком масштабировании нужно предусмотреть и периоды свертывания, когда удаляются некоторые экземпляры приложения. Приложение должно корректно обрабатывать удаление экземпляров. Вот несколько возможных способов для обработки свертывания:

- Прослушивайте сообщения о событиях завершения работы (если они доступны) и завершайте работу надлежащим образом.
- Клиенты (потребители) службы должны поддерживать обработку временных сбоев и выполнять повторные попытки.
- Длительно выполняемые задачи можно разделять на части, использовать для них контрольные точки или реализовать шаблон [каналов и фильтров][pipes-filters-pattern].
- Передавайте рабочие элементы в очередь, из которой они смогут перейти в другой экземпляр, если текущий будет удален в процессе обработки.

<!-- links -->

[autoscaling]: ../../best-practices/auto-scaling.md
[background-jobs]: ../../best-practices/background-jobs.md
[pipes-filters-pattern]: ../../patterns/pipes-and-filters.md
