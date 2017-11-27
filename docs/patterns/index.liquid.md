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
# <a name="cloud-design-patterns"></a>Конструктивные шаблоны облачных решений

[!INCLUDE [header](../../_includes/header.md)]

Эти конструктивные шаблоны полезны для создания надежных, масштабируемых, безопасных приложений в облаке.

В каждом из шаблонов описывается проблема, которую он устраняет, приведены рекомендации для применения шаблона и пример, основанный на Microsoft Azure. Большинство шаблонов содержат примеры или фрагменты кода, в которых показано, как реализовать шаблон в Azure. Однако многие шаблоны относятся к любой распределенной системе, размещенной в Azure или на других облачных платформах.

## <a name="problem-areas-in-the-cloud"></a>Проблемные области в облаке

<ul id="categories" class="panel">
{%- for category in categories %}
    <li>
    {% include 'pattern-category-card' %}
    </li>
{%- endfor %}
</ul>

## <a name="catalog-of-patterns"></a>Каталог шаблонов

| Модель | Сводка |
| ------- | ------- |
{%- for pattern in patterns %} | [{{ pattern.title }}](./{{ pattern.file }}) | {{ pattern.description }} | {%- endfor %}