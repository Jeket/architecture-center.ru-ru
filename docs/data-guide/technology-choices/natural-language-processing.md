---
title: Выбор технологии обработки естественных языков
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: b1cb019164285d16b6e9d34eae220801785adab9
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/05/2018
ms.locfileid: "52902314"
---
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a>Выбор технологии обработки естественных языков в Azure

Обработка свободного текста выполняется для документов, содержащих абзацы, как правило, чтобы обеспечить поиск, а также другие задачи обработки естественного языка (NLP), например анализа тональности, распознавания тем, распознавания языка, извлечения ключевых фраз и классификация документов. Эта статья посвящена выбору технологий, которые служат для поддержки задач NLP.

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a>Варианты выбора служб NLP

В Azure функции NLP предоставляются в следующих службах:

- [Azure HDInsight со Spark и Spark MLlib](/azure/hdinsight/spark/apache-spark-overview);
- [Azure Databricks](/azure/azure-databricks/what-is-azure-databricks)
- [Microsoft Cognitive Services](/azure/cognitive-services/welcome).

## <a name="key-selection-criteria"></a>Основные критерии выбора

Чтобы ограничить количество вариантов, сначала ответьте на следующие вопросы:

- Нужно использовать предварительно созданные модели? Если да, используйте API-интерфейсы, предлагаемые в службах Microsoft Cognitive Services.

- Нужно ли обучать пользовательские модели на основе большой совокупности текстовых данных? Если да, используйте Azure HDInsight со Spark MLlib и Spark NLP.

- Требуются ли низкоуровневые возможности NLP, например разметка, выделение корней слов, лемматизация и частота условия или инверсная частота в документе? Если да, используйте Azure HDInsight со Spark MLlib и Spark NLP.

- Требуются ли простые, высокоуровневые возможности NLP, например распознавание сущностей и намерений, распознавание темы, проверка орфографии и анализ тональности. Если да, используйте API-интерфейсы, предлагаемые в службах Microsoft Cognitive Services.

## <a name="capability-matrix"></a>Матрица возможностей

В следующих таблицах перечислены основные различия в возможностях.  

### <a name="general-capabilities"></a>Общие возможности

| | Azure HDInsight | Microsoft Cognitive Services |
| --- | --- | --- |
| Предоставление предварительно обученных моделей как услуги | Нет  | Yes |
| REST API | Yes | Yes |
| Программируемость | Python, Scala, Java | C#, Java, Node.js, Python, PHP, Ruby |
| Поддержка обработки больших наборов данных и больших документов | Yes | Нет  |

### <a name="low-level-natural-language-processing-capabilities"></a>Низкоуровневые возможности обработки естественного языка

| | Azure HDInsight | Microsoft Cognitive Services |  
| --- | --- | --- | 
| Создатель маркеров | Да (Spark NLP) | Да (API лингвистического анализа) |
| Парадигматический модуль | Да (Spark NLP) | Нет  |
| Лемматический модуль | Да (Spark NLP) | Нет  |
| Добавление тегов к частям речи | Да (Spark NLP) | Да (API лингвистического анализа) |
| Частота условия или инверсная частота в документе (TF/IDF) | Да (Spark MLlib) | Нет  |
| Семантические сходства строк &mdash;вычисление расстояния редактирования | Да (Spark MLlib) | Нет  |
| Вычисление N-грамм | Да (Spark MLlib) | Нет  |
| Удаление стоп-слов | Да (Spark MLlib) | Нет  |

### <a name="high-level-natural-language-processing-capabilities"></a>Высокоуровневые возможности обработки естественного языка

| | Azure HDInsight | Microsoft Cognitive Services |
| --- | --- | --- | 
| Распознавание и извлечение сущностей и намерений | Нет  | Да (API распознавания речи (LUIS)) |    
| Определение темы | Да (Spark NLP) | Да (API анализа текста) |
| Проверка орфографии | Да (Spark NLP) | Да (API Bing для проверки орфографии) |
| Анализ мнений | Да (Spark NLP) | Да (API анализа текста) |
| Определение языка | Нет  | Да (API анализа текста) |
| Поддержка нескольких языков, кроме английского | Нет  | Да (зависит от API) |

## <a name="see-also"></a>См. также

[Обработка естественного языка](../scenarios/natural-language-processing.md)
