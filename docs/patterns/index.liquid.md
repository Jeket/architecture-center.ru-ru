---
title: Конструктивные шаблоны облачных решений
titleSuffix: Azure Architecture Center
description: Конструктивные шаблоны облачных решений для Microsoft Azure
keywords: Таблицы Azure
author: dragon119
ms.date: 12/10/2018
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 4229531366f1b0c3257384694cf4358da9e63177
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58241485"
---
# <a name="cloud-design-patterns"></a><span data-ttu-id="2fc0c-104">Конструктивные шаблоны облачных решений</span><span class="sxs-lookup"><span data-stu-id="2fc0c-104">Cloud Design Patterns</span></span>

[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="2fc0c-105">Эти конструктивные шаблоны полезны для создания надежных, масштабируемых, безопасных приложений в облаке.</span><span class="sxs-lookup"><span data-stu-id="2fc0c-105">These design patterns are useful for building reliable, scalable, secure applications in the cloud.</span></span>

<span data-ttu-id="2fc0c-106">В каждом из шаблонов описывается проблема, которую он устраняет, приведены рекомендации для применения шаблона и пример, основанный на Microsoft Azure.</span><span class="sxs-lookup"><span data-stu-id="2fc0c-106">Each pattern describes the problem that the pattern addresses, considerations for applying the pattern, and an example based on Microsoft Azure.</span></span> <span data-ttu-id="2fc0c-107">Большинство шаблонов содержат примеры или фрагменты кода, в которых показано, как реализовать шаблон в Azure.</span><span class="sxs-lookup"><span data-stu-id="2fc0c-107">Most of the patterns include code samples or snippets that show how to implement the pattern on Azure.</span></span> <span data-ttu-id="2fc0c-108">Однако многие шаблоны относятся к любой распределенной системе, размещенной в Azure или на других облачных платформах.</span><span class="sxs-lookup"><span data-stu-id="2fc0c-108">However, most of the patterns are relevant to any distributed system, whether hosted on Azure or on other cloud platforms.</span></span>

## <a name="problem-areas-in-the-cloud"></a><span data-ttu-id="2fc0c-109">Проблемные области в облаке</span><span class="sxs-lookup"><span data-stu-id="2fc0c-109">Problem areas in the cloud</span></span>

<!-- markdownlint-disable MD033 -->

<ul id="categories" class="panel">
<span data-ttu-id="2fc0c-110">{%- for category in categories %}</span><span class="sxs-lookup"><span data-stu-id="2fc0c-110">{%- for category in categories %}</span></span>
    <li>
    <span data-ttu-id="2fc0c-111">{% include 'pattern-category-card' %}</span><span class="sxs-lookup"><span data-stu-id="2fc0c-111">{% include 'pattern-category-card' %}</span></span>
    </li>
<span data-ttu-id="2fc0c-112">{%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="2fc0c-112">{%- endfor %}</span></span>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="catalog-of-patterns"></a><span data-ttu-id="2fc0c-113">Каталог шаблонов</span><span class="sxs-lookup"><span data-stu-id="2fc0c-113">Catalog of patterns</span></span>

| <span data-ttu-id="2fc0c-114">Модель</span><span class="sxs-lookup"><span data-stu-id="2fc0c-114">Pattern</span></span> | <span data-ttu-id="2fc0c-115">Сводка</span><span class="sxs-lookup"><span data-stu-id="2fc0c-115">Summary</span></span> |
|---------|---------|
|         |         |

<span data-ttu-id="2fc0c-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span><span class="sxs-lookup"><span data-stu-id="2fc0c-116">{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}</span></span>
