---
title: CAF. Пять принципов рационализации
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
description: Просмотрите параметры, доступные для рационализации цифровых активов.
author: BrianBlanchard
ms.date: 12/10/2018
ms.topic: guide
ms.openlocfilehash: ee196487e6f59b1e1b3c63bab9496cbbf805affd
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/08/2019
ms.locfileid: "55897479"
---
# <a name="the-5-rs-of-rationalization"></a><span data-ttu-id="f4d41-103">Пять принципов рационализации</span><span class="sxs-lookup"><span data-stu-id="f4d41-103">The 5 Rs of rationalization</span></span>

<span data-ttu-id="f4d41-104">Рационализация облака — это процесс оценки активов, который определяет наилучший подход к миграции или модернизации каждого ресурса в облаке.</span><span class="sxs-lookup"><span data-stu-id="f4d41-104">Cloud Rationalization is the process of evaluating assets to determine the best approach to migrating or modernizing each asset in the cloud.</span></span> <span data-ttu-id="f4d41-105">Дополнительные сведения о процессе рационализации см. в статье с описанием [цифровых активов](overview.md).</span><span class="sxs-lookup"><span data-stu-id="f4d41-105">For more information about the process of rationalization, see [What is a digital estate?](overview.md)</span></span>

<span data-ttu-id="f4d41-106">Перечисленные здесь пять принципов рационализации описывают наиболее распространенные варианты рационализации.</span><span class="sxs-lookup"><span data-stu-id="f4d41-106">The "5 Rs of rationalization" listed here describe the most common options for rationalization.</span></span>

## <a name="rehost"></a><span data-ttu-id="f4d41-107">Повторное размещение</span><span class="sxs-lookup"><span data-stu-id="f4d41-107">Rehost</span></span>

<span data-ttu-id="f4d41-108">Этот принцип также известен как "lift and shift". При повторном размещении ресурс с текущим состоянием перемещается в выбранный поставщик облачных служб с минимальными изменениями общей архитектуры.</span><span class="sxs-lookup"><span data-stu-id="f4d41-108">Also known as "lift and shift," a rehost effort moves the current state asset to the chosen cloud provider, with minimal change to overall architecture.</span></span>

<span data-ttu-id="f4d41-109">Общие факторы могут включать в себя:</span><span class="sxs-lookup"><span data-stu-id="f4d41-109">Common drivers could include:</span></span>

* <span data-ttu-id="f4d41-110">сокращение капитальных расходов;</span><span class="sxs-lookup"><span data-stu-id="f4d41-110">Reduce CapEx</span></span>
* <span data-ttu-id="f4d41-111">освобождение места в центре обработки данных;</span><span class="sxs-lookup"><span data-stu-id="f4d41-111">Free up datacenter space</span></span>
* <span data-ttu-id="f4d41-112">быстрое получение рентабельности инвестиций в облако.</span><span class="sxs-lookup"><span data-stu-id="f4d41-112">Quick cloud ROI</span></span>

<span data-ttu-id="f4d41-113">Факторы количественного анализа:</span><span class="sxs-lookup"><span data-stu-id="f4d41-113">Quantitative analysis factors:</span></span>

* <span data-ttu-id="f4d41-114">размер ресурсов виртуальной машины (ЦП, память, хранилище);</span><span class="sxs-lookup"><span data-stu-id="f4d41-114">VM size (CPU, memory, storage)</span></span>
* <span data-ttu-id="f4d41-115">зависимости (сетевой трафик);</span><span class="sxs-lookup"><span data-stu-id="f4d41-115">Dependencies (network traffic)</span></span>
* <span data-ttu-id="f4d41-116">совместимость ресурсов.</span><span class="sxs-lookup"><span data-stu-id="f4d41-116">Asset compatibility</span></span>

<span data-ttu-id="f4d41-117">Факторы качественного анализа:</span><span class="sxs-lookup"><span data-stu-id="f4d41-117">Qualitative analysis factors:</span></span>

* <span data-ttu-id="f4d41-118">устойчивость к изменениям;</span><span class="sxs-lookup"><span data-stu-id="f4d41-118">Tolerance for change</span></span>
* <span data-ttu-id="f4d41-119">бизнес-приоритеты;</span><span class="sxs-lookup"><span data-stu-id="f4d41-119">Business priorities</span></span>
* <span data-ttu-id="f4d41-120">критические бизнес-события;</span><span class="sxs-lookup"><span data-stu-id="f4d41-120">Critical business events</span></span>
* <span data-ttu-id="f4d41-121">зависимости процессов.</span><span class="sxs-lookup"><span data-stu-id="f4d41-121">Process dependencies</span></span>

## <a name="refactor"></a><span data-ttu-id="f4d41-122">Рефакторинг</span><span class="sxs-lookup"><span data-stu-id="f4d41-122">Refactor</span></span>

<span data-ttu-id="f4d41-123">Платформа как услуга (PaaS) может сокращать операционные расходы, связанные с многими приложениями.</span><span class="sxs-lookup"><span data-stu-id="f4d41-123">Platform as a Service (PaaS) options can reduce operational costs associated with many applications.</span></span> <span data-ttu-id="f4d41-124">Может быть рационально выполнить поверхностный рефакторинг приложения для соответствия модели PaaS.</span><span class="sxs-lookup"><span data-stu-id="f4d41-124">It can be prudent to slightly refactor an application to fit a PaaS based model.</span></span>

<span data-ttu-id="f4d41-125">Рефакторинг также относится к процессу разработки приложения. Рефакторинг кода позволяет добавлять вашему приложению новые бизнес-возможности.</span><span class="sxs-lookup"><span data-stu-id="f4d41-125">Refactor also refers to the application development process of refactoring code to allow an application to deliver on new business opportunities.</span></span>

<span data-ttu-id="f4d41-126">Общие факторы могут включать в себя:</span><span class="sxs-lookup"><span data-stu-id="f4d41-126">Common drivers could include:</span></span>

* <span data-ttu-id="f4d41-127">более частые незначительные обновления;</span><span class="sxs-lookup"><span data-stu-id="f4d41-127">Faster, shorter, updates</span></span>
* <span data-ttu-id="f4d41-128">переносимость кода;</span><span class="sxs-lookup"><span data-stu-id="f4d41-128">Code portability</span></span>
* <span data-ttu-id="f4d41-129">повышение облачной эффективности (ресурсы, скорость, затраты).</span><span class="sxs-lookup"><span data-stu-id="f4d41-129">Greater cloud efficiency (resources, speed, cost)</span></span>

<span data-ttu-id="f4d41-130">Факторы количественного анализа:</span><span class="sxs-lookup"><span data-stu-id="f4d41-130">Quantitative analysis factors:</span></span>

* <span data-ttu-id="f4d41-131">размеры ресурсов приложения (ЦП, память, хранилище);</span><span class="sxs-lookup"><span data-stu-id="f4d41-131">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="f4d41-132">зависимости (сетевой трафик);</span><span class="sxs-lookup"><span data-stu-id="f4d41-132">Dependencies (network traffic)</span></span>
* <span data-ttu-id="f4d41-133">пользовательский трафик (просмотры страниц, время, потраченное на просмотр страниц, время загрузки);</span><span class="sxs-lookup"><span data-stu-id="f4d41-133">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="f4d41-134">платформу разработки (языки, платформа данных, службы среднего уровня).</span><span class="sxs-lookup"><span data-stu-id="f4d41-134">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="f4d41-135">Факторы качественного анализа:</span><span class="sxs-lookup"><span data-stu-id="f4d41-135">Qualitative analysis factors:</span></span>

* <span data-ttu-id="f4d41-136">продолжение инвестиций в бизнес;</span><span class="sxs-lookup"><span data-stu-id="f4d41-136">Continued business investments</span></span>
* <span data-ttu-id="f4d41-137">расширение сроков или функциональности;</span><span class="sxs-lookup"><span data-stu-id="f4d41-137">Bursting options/timelines</span></span>
* <span data-ttu-id="f4d41-138">зависимости бизнес-процессов.</span><span class="sxs-lookup"><span data-stu-id="f4d41-138">Business process dependencies</span></span>

## <a name="rearchitect"></a><span data-ttu-id="f4d41-139">Перепроектирование</span><span class="sxs-lookup"><span data-stu-id="f4d41-139">Rearchitect</span></span>

<span data-ttu-id="f4d41-140">Некоторые устаревшие приложения несовместимы с поставщиками облачных служб из-за архитектурных решений, принятых на этапе создания приложения.</span><span class="sxs-lookup"><span data-stu-id="f4d41-140">Some aging applications aren't compatible with cloud providers because of the architectural decisions made when the application was built.</span></span> <span data-ttu-id="f4d41-141">В таких случаях приложение нужно перепроектировать для соответствия облачным службам.</span><span class="sxs-lookup"><span data-stu-id="f4d41-141">In these cases, the application may need to be rearchitected prior to transformation.</span></span>

<span data-ttu-id="f4d41-142">В других случаях, если приложения совместимы с облачными службами, но не имеют преимуществ полностью облачных решений, перепроектирование приложения в полностью облачное может оптимизировать расходы и повысить операционную эффективность.</span><span class="sxs-lookup"><span data-stu-id="f4d41-142">In other cases, applications that are cloud compatible, but not cloud native benefits, may produce cost efficiencies and operational efficiencies by rearchitecting the solution to be a cloud native application.</span></span>

<span data-ttu-id="f4d41-143">Общие факторы могут включать в себя:</span><span class="sxs-lookup"><span data-stu-id="f4d41-143">Common drivers could include:</span></span>

* <span data-ttu-id="f4d41-144">масштабирование и гибкость приложения;</span><span class="sxs-lookup"><span data-stu-id="f4d41-144">Application scale and agility</span></span>
* <span data-ttu-id="f4d41-145">упрощение внедрения новых облачных возможностей;</span><span class="sxs-lookup"><span data-stu-id="f4d41-145">Easier adoption of new cloud capabilities</span></span>
* <span data-ttu-id="f4d41-146">сочетание стеков технологий.</span><span class="sxs-lookup"><span data-stu-id="f4d41-146">Mix of technology stacks</span></span>

<span data-ttu-id="f4d41-147">Факторы количественного анализа:</span><span class="sxs-lookup"><span data-stu-id="f4d41-147">Quantitative analysis factors:</span></span>

* <span data-ttu-id="f4d41-148">размеры ресурсов приложения (ЦП, память, хранилище);</span><span class="sxs-lookup"><span data-stu-id="f4d41-148">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="f4d41-149">зависимости (сетевой трафик);</span><span class="sxs-lookup"><span data-stu-id="f4d41-149">Dependencies (network traffic)</span></span>
* <span data-ttu-id="f4d41-150">пользовательский трафик (просмотры страниц, время, потраченное на просмотр страниц, время загрузки);</span><span class="sxs-lookup"><span data-stu-id="f4d41-150">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="f4d41-151">платформу разработки (языки, платформа данных, службы среднего уровня).</span><span class="sxs-lookup"><span data-stu-id="f4d41-151">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="f4d41-152">Факторы качественного анализа:</span><span class="sxs-lookup"><span data-stu-id="f4d41-152">Qualitative analysis factors:</span></span>

* <span data-ttu-id="f4d41-153">прирост инвестиций в бизнес;</span><span class="sxs-lookup"><span data-stu-id="f4d41-153">Growing business investments</span></span>
* <span data-ttu-id="f4d41-154">операционные расходы;</span><span class="sxs-lookup"><span data-stu-id="f4d41-154">Operational costs</span></span>
* <span data-ttu-id="f4d41-155">потенциальные циклы обратной связи и инвестиции в DevOps.</span><span class="sxs-lookup"><span data-stu-id="f4d41-155">Potential feedback loops and DevOps investments</span></span>

## <a name="rebuild"></a><span data-ttu-id="f4d41-156">перестроение;</span><span class="sxs-lookup"><span data-stu-id="f4d41-156">Rebuild</span></span>

<span data-ttu-id="f4d41-157">В некоторых сценариях разница в затратах, которую необходимо понести при переносе приложения, может быть слишком большой, чтобы оправдать дальнейшие инвестиции.</span><span class="sxs-lookup"><span data-stu-id="f4d41-157">In some scenarios, the delta that must be overcome to carry forward an application can be too large to justify further investment.</span></span> <span data-ttu-id="f4d41-158">Это особо касается приложений, которые отвечали требованиям бизнеса, но теперь не поддерживаются или не соответствуют современным бизнес-процессам.</span><span class="sxs-lookup"><span data-stu-id="f4d41-158">This is especially true for applications that used to meet the needs of the business, but are now unsupported or misaligned with how the business processes are executed today.</span></span> <span data-ttu-id="f4d41-159">В таком случае создается новая база кода, которая соответствует [облачному](https://azure.microsoft.com/overview/cloudnative/) подходу.</span><span class="sxs-lookup"><span data-stu-id="f4d41-159">In this case, a new code base is created to align with a [cloud native](https://azure.microsoft.com/overview/cloudnative/) approach.</span></span>

<span data-ttu-id="f4d41-160">Общие факторы могут включать в себя:</span><span class="sxs-lookup"><span data-stu-id="f4d41-160">Common drivers could include:</span></span>

* <span data-ttu-id="f4d41-161">ускорение внедрения инноваций;</span><span class="sxs-lookup"><span data-stu-id="f4d41-161">Accelerate innovation</span></span>
* <span data-ttu-id="f4d41-162">ускорение создания приложений;</span><span class="sxs-lookup"><span data-stu-id="f4d41-162">Build apps faster</span></span>
* <span data-ttu-id="f4d41-163">сокращение операционных расходов.</span><span class="sxs-lookup"><span data-stu-id="f4d41-163">Reduce operational cost</span></span>

<span data-ttu-id="f4d41-164">Факторы количественного анализа:</span><span class="sxs-lookup"><span data-stu-id="f4d41-164">Quantitative analysis factors:</span></span>

* <span data-ttu-id="f4d41-165">размеры ресурсов приложения (ЦП, память, хранилище);</span><span class="sxs-lookup"><span data-stu-id="f4d41-165">Application asset size (CPU, memory, storage)</span></span>
* <span data-ttu-id="f4d41-166">зависимости (сетевой трафик);</span><span class="sxs-lookup"><span data-stu-id="f4d41-166">Dependencies (network traffic)</span></span>
* <span data-ttu-id="f4d41-167">пользовательский трафик (просмотры страниц, время, потраченное на просмотр страниц, время загрузки);</span><span class="sxs-lookup"><span data-stu-id="f4d41-167">User traffic (page views, time on page, load time)</span></span>
* <span data-ttu-id="f4d41-168">платформу разработки (языки, платформа данных, службы среднего уровня).</span><span class="sxs-lookup"><span data-stu-id="f4d41-168">Development platform (languages, data platform, middle tier services)</span></span>

<span data-ttu-id="f4d41-169">Факторы качественного анализа:</span><span class="sxs-lookup"><span data-stu-id="f4d41-169">Qualitative analysis Factors:</span></span>

* <span data-ttu-id="f4d41-170">спад удовлетворенности пользователей;</span><span class="sxs-lookup"><span data-stu-id="f4d41-170">Declining end user satisfaction</span></span>
* <span data-ttu-id="f4d41-171">ограниченность функциональности бизнес-процессов;</span><span class="sxs-lookup"><span data-stu-id="f4d41-171">Business processes limited by functionality</span></span>
* <span data-ttu-id="f4d41-172">потенциальные затраты, расширение опыта и рост доходов.</span><span class="sxs-lookup"><span data-stu-id="f4d41-172">Potential cost, experience, or revenue gains</span></span>

## <a name="replace"></a><span data-ttu-id="f4d41-173">Замените</span><span class="sxs-lookup"><span data-stu-id="f4d41-173">Replace</span></span>

<span data-ttu-id="f4d41-174">Обычно решения реализуются с использованием наилучшей технологии и подхода, доступных в конкретное время.</span><span class="sxs-lookup"><span data-stu-id="f4d41-174">Solutions are generally implemented using the best technology and approach available at the time.</span></span> <span data-ttu-id="f4d41-175">В некоторых случаях SaaS-приложения могут обеспечивать все функциональные возможности, требуемые размещенному приложению.</span><span class="sxs-lookup"><span data-stu-id="f4d41-175">In some cases, Software as a Service (SaaS) applications can meet all of the functionality required of the hosted application.</span></span> <span data-ttu-id="f4d41-176">В этих сценариях рабочую нагрузку можно оставить для будущих замен, фактически удаляя ее из списка трудозатрат на преобразование.</span><span class="sxs-lookup"><span data-stu-id="f4d41-176">In these scenarios, a workload could be slated for future replacement, effectively removing it from the transformation effort.</span></span>

<span data-ttu-id="f4d41-177">Общие факторы могут включать в себя:</span><span class="sxs-lookup"><span data-stu-id="f4d41-177">Common drivers could include:</span></span>

* <span data-ttu-id="f4d41-178">стандартизацию рекомендаций для отрасли;</span><span class="sxs-lookup"><span data-stu-id="f4d41-178">Standardize around industry-best practices</span></span>
* <span data-ttu-id="f4d41-179">ускорение внедрения подходов на основе бизнес-процессов;</span><span class="sxs-lookup"><span data-stu-id="f4d41-179">Accelerate adoption of business process driven approaches</span></span>
* <span data-ttu-id="f4d41-180">перераспределение инвестиций в разработку приложений, которые создают конкурентоспособные различия и преимущества.</span><span class="sxs-lookup"><span data-stu-id="f4d41-180">Reallocate development investments into applications that create competitive differentiation or advantages</span></span>

<span data-ttu-id="f4d41-181">Факторы количественного анализа:</span><span class="sxs-lookup"><span data-stu-id="f4d41-181">Quantitative analysis factors:</span></span>

* <span data-ttu-id="f4d41-182">сокращение общих операционных расходов;</span><span class="sxs-lookup"><span data-stu-id="f4d41-182">General operating cost reductions</span></span>
* <span data-ttu-id="f4d41-183">размер ресурсов виртуальной машины (ЦП, память, хранилище);</span><span class="sxs-lookup"><span data-stu-id="f4d41-183">VM size (CPU, memory, storage)</span></span>
* <span data-ttu-id="f4d41-184">зависимости (сетевой трафик);</span><span class="sxs-lookup"><span data-stu-id="f4d41-184">Dependencies (network traffic)</span></span>
* <span data-ttu-id="f4d41-185">активы, которые выводятся из эксплуатации.</span><span class="sxs-lookup"><span data-stu-id="f4d41-185">Assets to be retired</span></span>

<span data-ttu-id="f4d41-186">Факторы качественного анализа:</span><span class="sxs-lookup"><span data-stu-id="f4d41-186">Qualitative analysis factors:</span></span>

* <span data-ttu-id="f4d41-187">анализ экономической выгоды текущей архитектуры в сравнении с решением SaaS;</span><span class="sxs-lookup"><span data-stu-id="f4d41-187">Cost benefit analysis of the current architecture versus a SaaS solution</span></span>
* <span data-ttu-id="f4d41-188">сопоставление бизнес-процессов;</span><span class="sxs-lookup"><span data-stu-id="f4d41-188">Business process maps</span></span>
* <span data-ttu-id="f4d41-189">схемы данных;</span><span class="sxs-lookup"><span data-stu-id="f4d41-189">Data schemas</span></span>
* <span data-ttu-id="f4d41-190">настраиваемые или автоматизированные процессы.</span><span class="sxs-lookup"><span data-stu-id="f4d41-190">Custom or automated processes</span></span>

## <a name="next-steps"></a><span data-ttu-id="f4d41-191">Дополнительная информация</span><span class="sxs-lookup"><span data-stu-id="f4d41-191">Next steps</span></span>

<span data-ttu-id="f4d41-192">В совокупности эти 5 принципов рационализации могут применяться для цифровых активов, чтобы принимать рационализаторские решения относительно будущего состояния каждого приложения.</span><span class="sxs-lookup"><span data-stu-id="f4d41-192">Collectively, these 5 Rs of Rationalization can be applied to a digital estate to make rationalization decisions regarding the future state of each application.</span></span>

> [!div class="nextstepaction"]
> [<span data-ttu-id="f4d41-193">Внедрение облачных решений в организации. Что такое цифровые активы</span><span class="sxs-lookup"><span data-stu-id="f4d41-193">What is a digital estate?</span></span>](overview.md)
