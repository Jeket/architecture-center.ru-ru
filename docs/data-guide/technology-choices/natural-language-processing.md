---
title: Выбор технологии обработки естественных языков
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.topic: guide
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.openlocfilehash: 20dc51e661befcc09dd1751b031d445ff2b9fa1a
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54486494"
---
# <a name="choosing-a-natural-language-processing-technology-in-azure"></a><span data-ttu-id="b17c2-102">Выбор технологии обработки естественных языков в Azure</span><span class="sxs-lookup"><span data-stu-id="b17c2-102">Choosing a natural language processing technology in Azure</span></span>

<span data-ttu-id="b17c2-103">Обработка свободного текста выполняется для документов, содержащих абзацы, как правило, чтобы обеспечить поиск, а также другие задачи обработки естественного языка (NLP), например анализа тональности, распознавания тем, распознавания языка, извлечения ключевых фраз и классификация документов.</span><span class="sxs-lookup"><span data-stu-id="b17c2-103">Free-form text processing is performed against documents containing paragraphs of text, typically for the purpose of supporting search, but is also used to perform other natural language processing (NLP) tasks such as sentiment analysis, topic detection, language detection, key phrase extraction, and document categorization.</span></span> <span data-ttu-id="b17c2-104">Эта статья посвящена выбору технологий, которые служат для поддержки задач NLP.</span><span class="sxs-lookup"><span data-stu-id="b17c2-104">This article focuses on the technology choices that act in support of the NLP tasks.</span></span>

<!-- markdownlint-disable MD026 -->

## <a name="what-are-your-options-when-choosing-an-nlp-service"></a><span data-ttu-id="b17c2-105">Варианты выбора служб NLP</span><span class="sxs-lookup"><span data-stu-id="b17c2-105">What are your options when choosing an NLP service?</span></span>

<!-- markdownlint-enable MD026 -->

<span data-ttu-id="b17c2-106">В Azure функции NLP предоставляются в следующих службах:</span><span class="sxs-lookup"><span data-stu-id="b17c2-106">In Azure, the following services provide natural language processing (NLP) capabilities:</span></span>

- <span data-ttu-id="b17c2-107">[Azure HDInsight со Spark и Spark MLlib](/azure/hdinsight/spark/apache-spark-overview);</span><span class="sxs-lookup"><span data-stu-id="b17c2-107">[Azure HDInsight with Spark and Spark MLlib](/azure/hdinsight/spark/apache-spark-overview)</span></span>
- [<span data-ttu-id="b17c2-108">Azure Databricks</span><span class="sxs-lookup"><span data-stu-id="b17c2-108">Azure Databricks</span></span>](/azure/azure-databricks/what-is-azure-databricks)
- <span data-ttu-id="b17c2-109">[Microsoft Cognitive Services](/azure/cognitive-services/welcome).</span><span class="sxs-lookup"><span data-stu-id="b17c2-109">[Microsoft Cognitive Services](/azure/cognitive-services/welcome)</span></span>

## <a name="key-selection-criteria"></a><span data-ttu-id="b17c2-110">Основные критерии выбора</span><span class="sxs-lookup"><span data-stu-id="b17c2-110">Key selection criteria</span></span>

<span data-ttu-id="b17c2-111">Чтобы ограничить количество вариантов, сначала ответьте на следующие вопросы:</span><span class="sxs-lookup"><span data-stu-id="b17c2-111">To narrow the choices, start by answering these questions:</span></span>

- <span data-ttu-id="b17c2-112">Нужно использовать предварительно созданные модели?</span><span class="sxs-lookup"><span data-stu-id="b17c2-112">Do you want to use prebuilt models?</span></span> <span data-ttu-id="b17c2-113">Если да, используйте API-интерфейсы, предлагаемые в службах Microsoft Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="b17c2-113">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

- <span data-ttu-id="b17c2-114">Нужно ли обучать пользовательские модели на основе большой совокупности текстовых данных?</span><span class="sxs-lookup"><span data-stu-id="b17c2-114">Do you need to train custom models against a large corpus of text data?</span></span> <span data-ttu-id="b17c2-115">Если да, используйте Azure HDInsight со Spark MLlib и Spark NLP.</span><span class="sxs-lookup"><span data-stu-id="b17c2-115">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="b17c2-116">Требуются ли низкоуровневые возможности NLP, например разметка, выделение корней слов, лемматизация и частота условия или инверсная частота в документе?</span><span class="sxs-lookup"><span data-stu-id="b17c2-116">Do you need low-level NLP capabilities like tokenization, stemming, lemmatization, and term frequency/inverse document frequency (TF/IDF)?</span></span> <span data-ttu-id="b17c2-117">Если да, используйте Azure HDInsight со Spark MLlib и Spark NLP.</span><span class="sxs-lookup"><span data-stu-id="b17c2-117">If yes, consider using Azure HDInsight with Spark MLlib and Spark NLP.</span></span>

- <span data-ttu-id="b17c2-118">Требуются ли простые, высокоуровневые возможности NLP, например распознавание сущностей и намерений, распознавание темы, проверка орфографии и анализ тональности.</span><span class="sxs-lookup"><span data-stu-id="b17c2-118">Do you need simple, high-level NLP capabilities like entity and intent identification, topic detection, spell check, or sentiment analysis?</span></span> <span data-ttu-id="b17c2-119">Если да, используйте API-интерфейсы, предлагаемые в службах Microsoft Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="b17c2-119">If yes, consider using the APIs offered by Microsoft Cognitive Services.</span></span>

## <a name="capability-matrix"></a><span data-ttu-id="b17c2-120">Матрица возможностей</span><span class="sxs-lookup"><span data-stu-id="b17c2-120">Capability matrix</span></span>

<span data-ttu-id="b17c2-121">В следующих таблицах перечислены основные различия в возможностях.</span><span class="sxs-lookup"><span data-stu-id="b17c2-121">The following tables summarize the key differences in capabilities.</span></span>

### <a name="general-capabilities"></a><span data-ttu-id="b17c2-122">Общие возможности</span><span class="sxs-lookup"><span data-stu-id="b17c2-122">General capabilities</span></span>

| | <span data-ttu-id="b17c2-123">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="b17c2-123">Azure HDInsight</span></span> | <span data-ttu-id="b17c2-124">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="b17c2-124">Microsoft Cognitive Services</span></span> |
| --- | --- | --- |
| <span data-ttu-id="b17c2-125">Предоставление предварительно обученных моделей как услуги</span><span class="sxs-lookup"><span data-stu-id="b17c2-125">Provides pretrained models as a service</span></span> | <span data-ttu-id="b17c2-126">Нет </span><span class="sxs-lookup"><span data-stu-id="b17c2-126">No</span></span> | <span data-ttu-id="b17c2-127">Yes</span><span class="sxs-lookup"><span data-stu-id="b17c2-127">Yes</span></span> |
| <span data-ttu-id="b17c2-128">REST API</span><span class="sxs-lookup"><span data-stu-id="b17c2-128">REST API</span></span> | <span data-ttu-id="b17c2-129">Yes</span><span class="sxs-lookup"><span data-stu-id="b17c2-129">Yes</span></span> | <span data-ttu-id="b17c2-130">Yes</span><span class="sxs-lookup"><span data-stu-id="b17c2-130">Yes</span></span> |
| <span data-ttu-id="b17c2-131">Программируемость</span><span class="sxs-lookup"><span data-stu-id="b17c2-131">Programmability</span></span> | <span data-ttu-id="b17c2-132">Python, Scala, Java</span><span class="sxs-lookup"><span data-stu-id="b17c2-132">Python, Scala, Java</span></span> | <span data-ttu-id="b17c2-133">C#, Java, Node.js, Python, PHP, Ruby</span><span class="sxs-lookup"><span data-stu-id="b17c2-133">C#, Java, Node.js, Python, PHP, Ruby</span></span> |
| <span data-ttu-id="b17c2-134">Поддержка обработки больших наборов данных и больших документов</span><span class="sxs-lookup"><span data-stu-id="b17c2-134">Support processing of big data sets and large documents</span></span> | <span data-ttu-id="b17c2-135">Yes</span><span class="sxs-lookup"><span data-stu-id="b17c2-135">Yes</span></span> | <span data-ttu-id="b17c2-136">Нет </span><span class="sxs-lookup"><span data-stu-id="b17c2-136">No</span></span> |

### <a name="low-level-natural-language-processing-capabilities"></a><span data-ttu-id="b17c2-137">Низкоуровневые возможности обработки естественного языка</span><span class="sxs-lookup"><span data-stu-id="b17c2-137">Low-level natural language processing capabilities</span></span>

| | <span data-ttu-id="b17c2-138">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="b17c2-138">Azure HDInsight</span></span> | <span data-ttu-id="b17c2-139">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="b17c2-139">Microsoft Cognitive Services</span></span> |  
| --- | --- | --- |
| <span data-ttu-id="b17c2-140">Создатель маркеров</span><span class="sxs-lookup"><span data-stu-id="b17c2-140">Tokenizer</span></span> | <span data-ttu-id="b17c2-141">Да (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="b17c2-141">Yes (Spark NLP)</span></span> | <span data-ttu-id="b17c2-142">Да (API лингвистического анализа)</span><span class="sxs-lookup"><span data-stu-id="b17c2-142">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="b17c2-143">Парадигматический модуль</span><span class="sxs-lookup"><span data-stu-id="b17c2-143">Stemmer</span></span> | <span data-ttu-id="b17c2-144">Да (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="b17c2-144">Yes (Spark NLP)</span></span> | <span data-ttu-id="b17c2-145">Нет </span><span class="sxs-lookup"><span data-stu-id="b17c2-145">No</span></span> |
| <span data-ttu-id="b17c2-146">Лемматический модуль</span><span class="sxs-lookup"><span data-stu-id="b17c2-146">Lemmatizer</span></span> | <span data-ttu-id="b17c2-147">Да (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="b17c2-147">Yes (Spark NLP)</span></span> | <span data-ttu-id="b17c2-148">Нет </span><span class="sxs-lookup"><span data-stu-id="b17c2-148">No</span></span> |
| <span data-ttu-id="b17c2-149">Добавление тегов к частям речи</span><span class="sxs-lookup"><span data-stu-id="b17c2-149">Part of speech tagging</span></span> | <span data-ttu-id="b17c2-150">Да (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="b17c2-150">Yes (Spark NLP)</span></span> | <span data-ttu-id="b17c2-151">Да (API лингвистического анализа)</span><span class="sxs-lookup"><span data-stu-id="b17c2-151">Yes (Linguistic Analysis API)</span></span> |
| <span data-ttu-id="b17c2-152">Частота условия или инверсная частота в документе (TF/IDF)</span><span class="sxs-lookup"><span data-stu-id="b17c2-152">Term frequency/inverse-document frequency (TF/IDF)</span></span> | <span data-ttu-id="b17c2-153">Да (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="b17c2-153">Yes (Spark MLlib)</span></span> | <span data-ttu-id="b17c2-154">Нет </span><span class="sxs-lookup"><span data-stu-id="b17c2-154">No</span></span> |
| <span data-ttu-id="b17c2-155">Семантические сходства строк &mdash;вычисление расстояния редактирования</span><span class="sxs-lookup"><span data-stu-id="b17c2-155">String similarity&mdash;edit distance calculation</span></span> | <span data-ttu-id="b17c2-156">Да (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="b17c2-156">Yes (Spark MLlib)</span></span> | <span data-ttu-id="b17c2-157">Нет </span><span class="sxs-lookup"><span data-stu-id="b17c2-157">No</span></span> |
| <span data-ttu-id="b17c2-158">Вычисление N-грамм</span><span class="sxs-lookup"><span data-stu-id="b17c2-158">N-gram calculation</span></span> | <span data-ttu-id="b17c2-159">Да (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="b17c2-159">Yes (Spark MLlib)</span></span> | <span data-ttu-id="b17c2-160">Нет </span><span class="sxs-lookup"><span data-stu-id="b17c2-160">No</span></span> |
| <span data-ttu-id="b17c2-161">Удаление стоп-слов</span><span class="sxs-lookup"><span data-stu-id="b17c2-161">Stop word removal</span></span> | <span data-ttu-id="b17c2-162">Да (Spark MLlib)</span><span class="sxs-lookup"><span data-stu-id="b17c2-162">Yes (Spark MLlib)</span></span> | <span data-ttu-id="b17c2-163">Нет </span><span class="sxs-lookup"><span data-stu-id="b17c2-163">No</span></span> |

### <a name="high-level-natural-language-processing-capabilities"></a><span data-ttu-id="b17c2-164">Высокоуровневые возможности обработки естественного языка</span><span class="sxs-lookup"><span data-stu-id="b17c2-164">High-level natural language processing capabilities</span></span>

| | <span data-ttu-id="b17c2-165">Azure HDInsight</span><span class="sxs-lookup"><span data-stu-id="b17c2-165">Azure HDInsight</span></span> | <span data-ttu-id="b17c2-166">Microsoft Cognitive Services</span><span class="sxs-lookup"><span data-stu-id="b17c2-166">Microsoft Cognitive Services</span></span> |
| --- | --- | --- |
| <span data-ttu-id="b17c2-167">Распознавание и извлечение сущностей и намерений</span><span class="sxs-lookup"><span data-stu-id="b17c2-167">Entity/intent identification & extraction</span></span> | <span data-ttu-id="b17c2-168">Нет </span><span class="sxs-lookup"><span data-stu-id="b17c2-168">No</span></span> | <span data-ttu-id="b17c2-169">Да (API распознавания речи (LUIS))</span><span class="sxs-lookup"><span data-stu-id="b17c2-169">Yes (Language Understanding Intelligent Service (LUIS) API)</span></span> |
| <span data-ttu-id="b17c2-170">Определение темы</span><span class="sxs-lookup"><span data-stu-id="b17c2-170">Topic detection</span></span> | <span data-ttu-id="b17c2-171">Да (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="b17c2-171">Yes (Spark NLP)</span></span> | <span data-ttu-id="b17c2-172">Да (API анализа текста)</span><span class="sxs-lookup"><span data-stu-id="b17c2-172">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="b17c2-173">Проверка орфографии</span><span class="sxs-lookup"><span data-stu-id="b17c2-173">Spell checking</span></span> | <span data-ttu-id="b17c2-174">Да (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="b17c2-174">Yes (Spark NLP)</span></span> | <span data-ttu-id="b17c2-175">Да (API Bing для проверки орфографии)</span><span class="sxs-lookup"><span data-stu-id="b17c2-175">Yes (Bing Spell Check API)</span></span> |
| <span data-ttu-id="b17c2-176">Анализ мнений</span><span class="sxs-lookup"><span data-stu-id="b17c2-176">Sentiment analysis</span></span> | <span data-ttu-id="b17c2-177">Да (Spark NLP)</span><span class="sxs-lookup"><span data-stu-id="b17c2-177">Yes (Spark NLP)</span></span> | <span data-ttu-id="b17c2-178">Да (API анализа текста)</span><span class="sxs-lookup"><span data-stu-id="b17c2-178">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="b17c2-179">Определение языка</span><span class="sxs-lookup"><span data-stu-id="b17c2-179">Language detection</span></span> | <span data-ttu-id="b17c2-180">Нет </span><span class="sxs-lookup"><span data-stu-id="b17c2-180">No</span></span> | <span data-ttu-id="b17c2-181">Да (API анализа текста)</span><span class="sxs-lookup"><span data-stu-id="b17c2-181">Yes (Text Analytics API)</span></span> |
| <span data-ttu-id="b17c2-182">Поддержка нескольких языков, кроме английского</span><span class="sxs-lookup"><span data-stu-id="b17c2-182">Supports multiple languages besides English</span></span> | <span data-ttu-id="b17c2-183">Нет </span><span class="sxs-lookup"><span data-stu-id="b17c2-183">No</span></span> | <span data-ttu-id="b17c2-184">Да (зависит от API)</span><span class="sxs-lookup"><span data-stu-id="b17c2-184">Yes (varies by API)</span></span> |

## <a name="see-also"></a><span data-ttu-id="b17c2-185">См. также</span><span class="sxs-lookup"><span data-stu-id="b17c2-185">See also</span></span>

[<span data-ttu-id="b17c2-186">Обработка естественного языка</span><span class="sxs-lookup"><span data-stu-id="b17c2-186">Natural language processing</span></span>](../scenarios/natural-language-processing.md)
