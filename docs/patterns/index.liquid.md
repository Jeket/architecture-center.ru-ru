---
title: "Конструктивные шаблоны облачных решений"
description: "Конструктивные шаблоны облачных решений для Microsoft Azure"
keywords: "Таблицы Azure"
ms.openlocfilehash: 264b8296a428f9c1b87314b782efcabc89cf010f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
---
# <a name="cloud-design-patterns"></a><span data-ttu-id="7ae74-104">Конструктивные шаблоны облачных решений</span><span class="sxs-lookup"><span data-stu-id="7ae74-104">Cloud Design Patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="7ae74-105">Эти конструктивные шаблоны полезны для создания надежных, масштабируемых, безопасных приложений в облаке.</span><span class="sxs-lookup"><span data-stu-id="7ae74-105">These design patterns are useful for building reliable, scalable, secure applications in the cloud.</span></span>

<span data-ttu-id="7ae74-106">В каждом из шаблонов описывается проблема, которую он устраняет, приведены рекомендации для применения шаблона и пример, основанный на Microsoft Azure.</span><span class="sxs-lookup"><span data-stu-id="7ae74-106">Each pattern describes the problem that the pattern addresses, considerations for applying the pattern, and an example based on Microsoft Azure.</span></span> <span data-ttu-id="7ae74-107">Большинство шаблонов содержат примеры или фрагменты кода, в которых показано, как реализовать шаблон в Azure.</span><span class="sxs-lookup"><span data-stu-id="7ae74-107">Most of the patterns include code samples or snippets that show how to implement the pattern on Azure.</span></span> <span data-ttu-id="7ae74-108">Однако многие шаблоны относятся к любой распределенной системе, размещенной в Azure или на других облачных платформах.</span><span class="sxs-lookup"><span data-stu-id="7ae74-108">However, most of the patterns are relevant to any distributed system, whether hosted on Azure or on other cloud platforms.</span></span>

## <a name="problem-areas-in-the-cloud"></a><span data-ttu-id="7ae74-109">Проблемные области в облаке</span><span class="sxs-lookup"><span data-stu-id="7ae74-109">Problem areas in the cloud</span></span>

<ul id="categories" class="panel">
<span data-ttu-id="7ae74-110">{%- for category in categories %}</span><span class="sxs-lookup"><span data-stu-id="7ae74-110">{%- for category in categories %}</span></span>
    <li>
    <span data-ttu-id="7ae74-111">{% include 'pattern-category-card' %}</span><span class="sxs-lookup"><span data-stu-id="7ae74-111">{% include 'pattern-category-card' %}</span></span>
    </li>
<span data-ttu-id="7ae74-112">{%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="7ae74-112">{%- endfor %}</span></span>
</ul>

## <a name="catalog-of-patterns"></a><span data-ttu-id="7ae74-113">Каталог шаблонов</span><span class="sxs-lookup"><span data-stu-id="7ae74-113">Catalog of patterns</span></span>

| <span data-ttu-id="7ae74-114">Модель</span><span class="sxs-lookup"><span data-stu-id="7ae74-114">Pattern</span></span> | <span data-ttu-id="7ae74-115">Сводка</span><span class="sxs-lookup"><span data-stu-id="7ae74-115">Summary</span></span> |
| ------- | ------- |
<span data-ttu-id="7ae74-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="7ae74-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span></span>