---
title: Разработка с учетом эксплуатации
titleSuffix: Azure Application Architecture Guide
description: Разработайте приложение так, чтобы группа эксплуатации получила все необходимые средства.
author: MikeWasson
ms.date: 08/30/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seojan19
ms.openlocfilehash: 75eaa7f8e322c66a83d2b43d180780a2fdcd745b
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58245635"
---
# <a name="design-for-operations"></a>Разработка с учетом эксплуатации

## <a name="design-an-application-so-that-the-operations-team-has-the-tools-they-need"></a>Разработайте приложение так, чтобы группа эксплуатации получила все необходимые средства.

Облако значительно изменило функции специалистов из группы эксплуатации. Теперь им больше не нужно заниматься оборудованием и инфраструктурой для размещения приложения.  С другой стороны, этап эксплуатации остается критически важной частью успешной работы облачного приложения. Ниже перечислены некоторые основные задачи группы эксплуатации:

- Развертывание
- Мониторинг
- Escalation (Эскалация);
- реагирование на инциденты.
- Аудит безопасности.

Для облачных приложений особенно важны надежные механизмы ведения журнала и трассировки. Взаимодействуйте с группой эксплуатации на этапах планирования и разработки, чтобы приложение могло предоставить им нужные данные и рекомендации для успешной работы.  <!-- to do: Link to DevOps checklist -->

## <a name="recommendations"></a>Рекомендации

**Обеспечьте полную отслеживаемость.** Когда решение развернуто и работает, основными инструментами для анализа системы становятся журналы и трассировки. *Трассировка* позволяет изучать пути процессов в системе и выявлять узкие места, проблемы с производительностью и точки сбоя. *Ведение журнала* обеспечивает сведения об отдельных событиях, например об изменении состояния приложения, ошибках и исключениях. Обязательно ведите журнал в рабочей среде, иначе у вас не будет важных сведений в тот момент, когда они больше всего нужны.

**Обеспечьте средство мониторинга.** Мониторинг позволяет понять, насколько хорошо (или плохо) работает приложение с точки зрения доступности, производительности и работоспособности. Например, мониторинг дает возможность проверить соблюдение соглашений об уровне обслуживания. Мониторинг выполняется во время обычной работы системы. Он должен быть максимально приближен к режиму реального времени, чтобы группа эксплуатации могла быстро реагировать на любые проблемы. Идеально реализованный мониторинг позволит предотвратить проблемы до того, как они приведут к критическому сбою. Дополнительные сведения см. в статье [Monitoring and diagnostics][monitoring] (Мониторинг и диагностика).

**Обеспечьте средство анализа первопричин.** Анализ первопричин — это процесс поиска основной причины ошибки. Он выполняется уже после того, как произошел сбой.

**Используйте распределенную трассировку.** Примените систему распределенной трассировки, охватывающую параллелизм, асинхронность и облачные масштабы. При трассировке должен использоваться идентификатор корреляции, единый для всех служб приложения. Одна операция может содержать множество вызовов к разным службам приложения. В случае сбоя такой операции идентификатор корреляции поможет найти причину сбоя.

**Стандартизируйте журналы и метрики.** Группой эксплуатации будут собираться журналы из нескольких служб, развернутых в решении. Если в каждой службе используется индивидуальный формат журнала, будет сложно или даже невозможно получить полезные сведения из таких журналов. Создайте общую схему и описания полей, например идентификатор корреляции, имя события, IP-адрес отправителя и т. д. Для отдельных служб можно создавать пользовательские схемы, которые наследуют формат базовой схемы и содержат дополнительные поля.

**Автоматизируйте задачи управления**, в том числе процессы подготовки, развертывания и мониторинга. Автоматизация задач позволяет легко повторять их с меньшей вероятностью ошибок оператора.

**Считайте конфигурацию собственным кодом.** Включите файлы конфигурации в систему управления версиями, чтобы отслеживать изменения в них, вести учет версий и отменять изменения при необходимости.

<!-- links -->

[monitoring]: ../../best-practices/monitoring.md
