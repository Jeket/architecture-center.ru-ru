---
title: CAF. Классификация данных
description: Классификация данных
author: BrianBlanchard
ms.date: 2/11/2019
ms.openlocfilehash: 07268e7242d92ac2581bf28b378a3c43d166620c
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901627"
---
<!-- markdownlint-disable MD026 -->

# <a name="what-is-data-classification"></a><span data-ttu-id="dd17c-103">Классификация данных</span><span class="sxs-lookup"><span data-stu-id="dd17c-103">What is data classification?</span></span>

<span data-ttu-id="dd17c-104">Это вводная статья на общую тему классификации данных.</span><span class="sxs-lookup"><span data-stu-id="dd17c-104">This is an introductory article on the general topic of Data Classification.</span></span> <span data-ttu-id="dd17c-105">Классификация данных является очень распространенной отправной точкой для всего управления.</span><span class="sxs-lookup"><span data-stu-id="dd17c-105">Data classification is a very common starting point for all governance.</span></span>

## <a name="business-risks-and-governance"></a><span data-ttu-id="dd17c-106">Бизнес-риски и управление</span><span class="sxs-lookup"><span data-stu-id="dd17c-106">Business risks and governance</span></span>

<span data-ttu-id="dd17c-107">В большинстве организаций основные причины инвестирования в управление могут быть сведены к трем бизнес-рискам:</span><span class="sxs-lookup"><span data-stu-id="dd17c-107">In most organizations, the primary reasons for investing in governance can be reduced to three business risks:</span></span>

* <span data-ttu-id="dd17c-108">ответственность за нарушение безопасности данных;</span><span class="sxs-lookup"><span data-stu-id="dd17c-108">Liability associated with data breaches</span></span>
* <span data-ttu-id="dd17c-109">прерывание бизнес-деятельности после сбоев;</span><span class="sxs-lookup"><span data-stu-id="dd17c-109">Interruption to the business from outages</span></span>
* <span data-ttu-id="dd17c-110">незапланированные или непредвиденные расходы.</span><span class="sxs-lookup"><span data-stu-id="dd17c-110">Unplanned or unexpected spending</span></span>

<span data-ttu-id="dd17c-111">Существует множество вариантов этих трех бизнес-рисков.</span><span class="sxs-lookup"><span data-stu-id="dd17c-111">There are many variants of these three business risks.</span></span> <span data-ttu-id="dd17c-112">Тем не менее они, как правило, самые распространенные.</span><span class="sxs-lookup"><span data-stu-id="dd17c-112">However, the tend to be the most common.</span></span>

## <a name="understand-then-mitigate"></a><span data-ttu-id="dd17c-113">Распознавание и устранение</span><span class="sxs-lookup"><span data-stu-id="dd17c-113">Understand then mitigate</span></span>

<span data-ttu-id="dd17c-114">Прежде, чем устранить любой риск, его необходимо распознать.</span><span class="sxs-lookup"><span data-stu-id="dd17c-114">Before any risk can be mitigated, it must be understood.</span></span> <span data-ttu-id="dd17c-115">В случае ответственности за нарушение безопасности данных такое распознавание начинается с классификации данных.</span><span class="sxs-lookup"><span data-stu-id="dd17c-115">In the case of data breach liability, that understanding starts with data classification.</span></span> <span data-ttu-id="dd17c-116">Классификация данных — это процесс связывания характеристики метаданных с каждым цифровым ресурсом, который определяет связанный с ним тип данных.</span><span class="sxs-lookup"><span data-stu-id="dd17c-116">Data classification is the process of associating a meta data characteristic to every asset in a digital estate, which identifies the type of data associated with that asset.</span></span>

<span data-ttu-id="dd17c-117">Корпорация Майкрософт предполагает, что любой ресурс, который был определен в качестве потенциального кандидата для миграции или развертывания в облаке, должен иметь документированные метаданные для записи классификации данных, важности для бизнеса и ответственности за выставление счетов.</span><span class="sxs-lookup"><span data-stu-id="dd17c-117">Microsoft suggests that any asset which has been identified as a potential candidate for migration or deployment to the cloud should have documented meta data to record the data classification, business criticality, and billing responsibility.</span></span> <span data-ttu-id="dd17c-118">Эти три точки классификации могут иметь большое значение для распознавания и устранения рисков.</span><span class="sxs-lookup"><span data-stu-id="dd17c-118">These three points of classification can go a long way to understanding and mitigating risks.</span></span>

## <a name="microsofts-data-classification"></a><span data-ttu-id="dd17c-119">Классификация данных корпорации Майкрософт</span><span class="sxs-lookup"><span data-stu-id="dd17c-119">Microsoft's data classification</span></span>

<span data-ttu-id="dd17c-120">Ниже приведен список классификаций, используемых корпорацией Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="dd17c-120">The following is a list of classifications Microsoft uses.</span></span> <span data-ttu-id="dd17c-121">В зависимости от отрасли или имеющихся требований безопасности стандарты классификации данных могут уже существовать в вашей организации.</span><span class="sxs-lookup"><span data-stu-id="dd17c-121">Depending on your industry or existing security requirements, data classifications standards may already exist within your organization.</span></span> <span data-ttu-id="dd17c-122">Если стандарты не введены, мы советуем использовать этот пример классификации, который поможет лучше понять ваши цифровые ресурсы и профиль риска.</span><span class="sxs-lookup"><span data-stu-id="dd17c-122">If no standard exists, we welcome you to use this sample classification, to help you better understand your digital estate and risk profile.</span></span>  

* <span data-ttu-id="dd17c-123">**Не бизнес-данные**. Данные из личной жизни, которые не принадлежат корпорации Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="dd17c-123">**Non-Business:** Data from your personal life that does not belong to Microsoft</span></span>
* <span data-ttu-id="dd17c-124">**Общедоступные**. Бизнес-данные, которые являются общедоступными и утвержденными для предоставления широкой публике.</span><span class="sxs-lookup"><span data-stu-id="dd17c-124">**Public:** Business data that is freely available and approved for public consumption</span></span>
* <span data-ttu-id="dd17c-125">**Общие сведения**. Бизнес-данные, не предназначенные для широкой аудитории.</span><span class="sxs-lookup"><span data-stu-id="dd17c-125">**General:** Business data that is not meant for a public audience</span></span>
* <span data-ttu-id="dd17c-126">**Конфиденциальные**. Бизнес-данные, которые могут нанести вред корпорации Майкрософт, если чрезмерно ими делиться.</span><span class="sxs-lookup"><span data-stu-id="dd17c-126">**Confidential:** Business data that could cause harm to Microsoft if over-shared</span></span>
* <span data-ttu-id="dd17c-127">**Строго конфиденциальные**. Бизнес-данные, которые могут привести к серьезному ущербу для корпорации Майкрософт, если чрезмерно ими делиться.</span><span class="sxs-lookup"><span data-stu-id="dd17c-127">**Highly Confidential:** Business data that would cause extensive harm to Microsoft if over-shared</span></span>

## <a name="tagging-data-classification-in-azure"></a><span data-ttu-id="dd17c-128">Добавление тегов к классификации данных в Azure</span><span class="sxs-lookup"><span data-stu-id="dd17c-128">Tagging data classification in Azure</span></span>

<span data-ttu-id="dd17c-129">Каждый поставщик облачных служб должен предлагать механизм записи метаданных о любом ресурсе.</span><span class="sxs-lookup"><span data-stu-id="dd17c-129">Every cloud provider should offer a mechanism for recording metadata about any asset.</span></span> <span data-ttu-id="dd17c-130">Метаданные являются крайне важными для управления ресурсами в облаке.</span><span class="sxs-lookup"><span data-stu-id="dd17c-130">Metadata is vital to managing assets in the cloud.</span></span> <span data-ttu-id="dd17c-131">В Azure теги ресурсов являются рекомендуемым подходом для хранения метаданных.</span><span class="sxs-lookup"><span data-stu-id="dd17c-131">In the case of Azure, resource tags are the suggested approach for metadata storage.</span></span> <span data-ttu-id="dd17c-132">Дополнительные сведения о тегах ресурсов см. в статье [Использование тегов для организации ресурсов в Azure](/azure/azure-resource-manager/resource-group-using-tags).</span><span class="sxs-lookup"><span data-stu-id="dd17c-132">For additional information on resource tagging in Azure, see the article on [Using tags to organize your Azure resources](/azure/azure-resource-manager/resource-group-using-tags).</span></span>

## <a name="next-steps"></a><span data-ttu-id="dd17c-133">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="dd17c-133">Next steps</span></span>

<span data-ttu-id="dd17c-134">Примените классификации данных в одной из стратегий действенного управления.</span><span class="sxs-lookup"><span data-stu-id="dd17c-134">Apply data classifications during one of the actionable governance journeys.</span></span>

> [!div class="nextstepaction"]
> <span data-ttu-id="dd17c-135">[Actionable governance journeys](../journeys/overview.md) (Стратегии действенного управления)</span><span class="sxs-lookup"><span data-stu-id="dd17c-135">[Begin an Actionable Governance Journeys](../journeys/overview.md)</span></span>
