---
title: Виртуальный собеседник Azure для резервирования отелей
description: Создание чат-ботов для коммерческих приложений с помощью службы Azure Bot.
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: 826aa36da5f30a871abd90fd8ab2b202ffdf93f0
ms.sourcegitcommit: b2a4eb132857afa70201e28d662f18458865a48e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/05/2018
ms.locfileid: "48819654"
---
# <a name="conversational-chatbot-for-hotel-reservations-on-azure"></a><span data-ttu-id="d0c61-103">Виртуальный собеседник Azure для резервирования отелей</span><span class="sxs-lookup"><span data-stu-id="d0c61-103">Conversational chatbot for hotel reservations on Azure</span></span>

<span data-ttu-id="d0c61-104">Этот пример сценария применим к предприятиям, которым нужно интегрировать виртуальный чат-бот в приложение.</span><span class="sxs-lookup"><span data-stu-id="d0c61-104">This example scenario is applicable to businesses that need to integrate a conversational chatbot into applications.</span></span> <span data-ttu-id="d0c61-105">В этом сценарии чат-бот C#, который позволяет клиентам проверить доступность и зарезервировать жилье через веб-приложение или мобильное приложение, используется для сети отелей.</span><span class="sxs-lookup"><span data-stu-id="d0c61-105">In this scenario, a C# chatbot is used for a hotel chain that allows customers to check availability and book accommodation through a web or mobile application.</span></span>

<span data-ttu-id="d0c61-106">Варианты использования включают предоставление клиентам возможности просматривать удобства отеля и бронировать номера, просматривать в ресторане меню на вынос и заказывать еду, а также искать и заказывать распечатку фотографий.</span><span class="sxs-lookup"><span data-stu-id="d0c61-106">Potential uses include providing a way for customers to view hotel availability and book rooms, review a restaurant take-out menu and place a food order, or search for and order prints of photographs.</span></span> <span data-ttu-id="d0c61-107">Чтобы реагировать на такие запросы клиентов, обычно компаниям нужно было нанимать и обучать специалистов службы поддержки, а клиентам приходилось ждать, пока представитель освободится и сможет оказать помощь.</span><span class="sxs-lookup"><span data-stu-id="d0c61-107">Traditionally, businesses would need to hire and train customer service agents to respond to these customer requests, and customers would have to wait until a representative is available to provide assistance.</span></span>

<span data-ttu-id="d0c61-108">Используя службы Azure, например службу Azure Bot и службы "Распознавание речи" или SAPI, компании могут помогать клиентам и обрабатывать заказы или резервирования с помощью автоматизированных масштабируемых ботов.</span><span class="sxs-lookup"><span data-stu-id="d0c61-108">By using Azure services such as the Bot Service and Language Understanding or Speech API services, companies can assist customers and process orders or reservations with automated, scalable bots.</span></span>

## <a name="relevant-use-cases"></a><span data-ttu-id="d0c61-109">Варианты соответствующего использования</span><span class="sxs-lookup"><span data-stu-id="d0c61-109">Relevant use cases</span></span>

<span data-ttu-id="d0c61-110">Рассмотрите этот сценарий для следующих вариантов использования:</span><span class="sxs-lookup"><span data-stu-id="d0c61-110">Consider this scenario for the following use cases:</span></span>

* <span data-ttu-id="d0c61-111">Просмотрите в этом ресторане меню на вынос и сделайте заказ</span><span class="sxs-lookup"><span data-stu-id="d0c61-111">View restaurant take-out menu and order food</span></span>
* <span data-ttu-id="d0c61-112">Проверьте удобства отеля и забронируйте комнату</span><span class="sxs-lookup"><span data-stu-id="d0c61-112">Check hotel availability and reserve a room</span></span>
* <span data-ttu-id="d0c61-113">Найдите доступные фотографии и распечатайте их</span><span class="sxs-lookup"><span data-stu-id="d0c61-113">Search available photos and order prints</span></span>

## <a name="architecture"></a><span data-ttu-id="d0c61-114">Архитектура</span><span class="sxs-lookup"><span data-stu-id="d0c61-114">Architecture</span></span>

![Обзор архитектуры компонентов Azure в системе виртуального собеседника][architecture]

<span data-ttu-id="d0c61-116">Этот сценарий включает виртуального собеседника, который исполняет функции консьержа в отеле.</span><span class="sxs-lookup"><span data-stu-id="d0c61-116">This scenario covers a conversational bot that functions as a concierge for a hotel.</span></span> <span data-ttu-id="d0c61-117">Данные передаются в сценарии следующим образом:</span><span class="sxs-lookup"><span data-stu-id="d0c61-117">The data flows through the scenario as follows:</span></span>

1. <span data-ttu-id="d0c61-118">Клиент может обращаться с чат-ботом с помощью мобильного телефона или веб-приложения.</span><span class="sxs-lookup"><span data-stu-id="d0c61-118">The customer accesses the chatbot with a mobile or web app.</span></span>
2. <span data-ttu-id="d0c61-119">Пользователь аутентифицируется с помощью Azure Active Directory B2C (бизнес для клиента).</span><span class="sxs-lookup"><span data-stu-id="d0c61-119">Using Azure Active Directory B2C (Business 2 Customer), the user is authenticated.</span></span>
3. <span data-ttu-id="d0c61-120">Взаимодействуя со службой программ-роботов, пользователь может запросить информацию об удобствах отеля.</span><span class="sxs-lookup"><span data-stu-id="d0c61-120">Interacting with the Bot Service, the user requests information about hotel availability.</span></span>
4. <span data-ttu-id="d0c61-121">Cognitive Services обрабатывает запросы естественного языка, чтобы понять о чем говорит клиент.</span><span class="sxs-lookup"><span data-stu-id="d0c61-121">Cognitive Services processes the natural language request to understand the customer communication.</span></span>
5. <span data-ttu-id="d0c61-122">Когда пользователь удовлетворен результатом, приложение-бот обновляет резервирование клиента в базе данных SQL Azure.</span><span class="sxs-lookup"><span data-stu-id="d0c61-122">After the user is happy with the results, the bot adds or updates the customer’s reservation in a SQL Database.</span></span>
6. <span data-ttu-id="d0c61-123">Служба Application Insights собирает данные телеметрии во время процесса, чтобы дать команде DevOps сведения о производительности бота и его использовании.</span><span class="sxs-lookup"><span data-stu-id="d0c61-123">Application Insights gathers runtime telemetry throughout the process to help the DevOps team with bot performance and usage.</span></span>

### <a name="components"></a><span data-ttu-id="d0c61-124">Компоненты</span><span class="sxs-lookup"><span data-stu-id="d0c61-124">Components</span></span>

* <span data-ttu-id="d0c61-125">[Azure Active Directory][aad-docs] — это мультитенантный облачный каталог и служба управления удостоверениями корпорации Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="d0c61-125">[Azure Active Directory][aad-docs] is Microsoft’s multi-tenant cloud-based directory and identity management service.</span></span> <span data-ttu-id="d0c61-126">Azure AD поддерживает соединитель B2C, позволяющий идентифицировать тех, кто использует внешние идентификаторы, например, Google, Facebook или учетную запись Microsoft.</span><span class="sxs-lookup"><span data-stu-id="d0c61-126">Azure AD supports a B2C connector allowing you to identify individuals using external IDs such as Google, Facebook, or a Microsoft Account.</span></span>
* <span data-ttu-id="d0c61-127">[Служба приложений Azure][appservice-docs] позволяет создавать и размещать веб-приложения на любых языках программирования без необходимости управлять инфраструктурой.</span><span class="sxs-lookup"><span data-stu-id="d0c61-127">[App Service][appservice-docs] enables you to build and host web applications in the programming language of your choice without managing infrastructure.</span></span>
* <span data-ttu-id="d0c61-128">[Служба программ-роботов][botservice-docs] предоставляет средства для создания, тестирования, развертывания и управления интеллектуальными ботами.</span><span class="sxs-lookup"><span data-stu-id="d0c61-128">[Bot Service][botservice-docs] provides tools to build, test, deploy, and manage intelligent bots.</span></span>
* <span data-ttu-id="d0c61-129">[Cognitive Services][cognitive-docs] позволяют использовать интеллектуальные алгоритмы, чтобы видеть, слышать, говорить, понимать и интерпретировать потребности клиентов с помощью обычных средств связи.</span><span class="sxs-lookup"><span data-stu-id="d0c61-129">[Cognitive Services][cognitive-docs] lets you use intelligent algorithms to see, hear, speak, understand, and interpret your user needs through natural methods of communication.</span></span>
* <span data-ttu-id="d0c61-130">[База данных SQL][sqldatabase-docs] — это полностью управляемая облачная реляционная база данных, которая обеспечивает совместимость с ядром SQL Server.</span><span class="sxs-lookup"><span data-stu-id="d0c61-130">[SQL Database][sqldatabase-docs] is a fully managed relational cloud database service that provides SQL Server engine compatibility.</span></span>
* <span data-ttu-id="d0c61-131">[Application Insights][appinsights-docs] — это расширяемая служба управления производительностью приложений (APM), которая позволяет контролировать производительность приложений, таких как чат-бот.</span><span class="sxs-lookup"><span data-stu-id="d0c61-131">[Application Insights][appinsights-docs] is an extensible Application Performance Management (APM) service that lets you monitor the performance of applications, such as your chatbot.</span></span>

### <a name="alternatives"></a><span data-ttu-id="d0c61-132">Альтернативные варианты</span><span class="sxs-lookup"><span data-stu-id="d0c61-132">Alternatives</span></span>

* <span data-ttu-id="d0c61-133">[API распознавания речи Microsoft][speech-api] может использоваться для изменения взаимодействия клиентов с ботом.</span><span class="sxs-lookup"><span data-stu-id="d0c61-133">[Microsoft Speech API][speech-api] can be used to change how customers interface with your bot.</span></span>
* <span data-ttu-id="d0c61-134">[QnA Maker][qna-maker] можно использовать для быстрого добавления бота набора знаний из полуструктурированного содержимого, например, раздела "Вопросы и ответы".</span><span class="sxs-lookup"><span data-stu-id="d0c61-134">[QnA Maker][qna-maker] can be used as to quickly add knowledge to your bot from semi-structured content like an FAQ.</span></span>
* <span data-ttu-id="d0c61-135">[Перевод текстов][translator] — это служба, которая может легко добавить для бота многоязычную поддержку.</span><span class="sxs-lookup"><span data-stu-id="d0c61-135">[Translator Text][translator] is a service that you might consider to easily add multi-lingual support to your bot.</span></span>

## <a name="considerations"></a><span data-ttu-id="d0c61-136">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="d0c61-136">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="d0c61-137">Доступность</span><span class="sxs-lookup"><span data-stu-id="d0c61-137">Availability</span></span>

<span data-ttu-id="d0c61-138">Этот сценарий использует Базу данных SQL Azure для хранения данных о резервировании клиента.</span><span class="sxs-lookup"><span data-stu-id="d0c61-138">This scenario uses Azure SQL Database for storing customer reservations.</span></span> <span data-ttu-id="d0c61-139">База данных SQL включает базу данных избыточную в пределах зоны, группы отработки отказа и георепликацию.</span><span class="sxs-lookup"><span data-stu-id="d0c61-139">SQL Database includes zone redundant databases, failover groups, and geo-replication.</span></span> <span data-ttu-id="d0c61-140">Дополнительные сведения см. в статье [Azure SQL Database availability capabilities][sqlavailability-docs] (Возможности доступности базы данных Azure SQL).</span><span class="sxs-lookup"><span data-stu-id="d0c61-140">For more information, see [Azure SQL Database availability capabilities][sqlavailability-docs].</span></span>

<span data-ttu-id="d0c61-141">Дополнительные сведения по другим вопросам доступности см. в статье [с контрольным списком для обеспечения доступности][availability] в Центре архитектуры Azure.</span><span class="sxs-lookup"><span data-stu-id="d0c61-141">For other availability topics, see the [availability checklist][availability] in the Azure Architecture Center.</span></span>

### <a name="scalability"></a><span data-ttu-id="d0c61-142">Масштабируемость</span><span class="sxs-lookup"><span data-stu-id="d0c61-142">Scalability</span></span>

<span data-ttu-id="d0c61-143">Этот сценарий использует Службу приложений Azure.</span><span class="sxs-lookup"><span data-stu-id="d0c61-143">This scenario uses Azure App Service.</span></span> <span data-ttu-id="d0c61-144">С помощью службы приложений можно автоматически масштабировать количество экземпляров, которые запускает бот.</span><span class="sxs-lookup"><span data-stu-id="d0c61-144">With App Service, you can automatically scale the number of instances that run your bot.</span></span> <span data-ttu-id="d0c61-145">Эта функциональность позволяет не отставать от потребительского спроса на веб-приложение и чат-бот.</span><span class="sxs-lookup"><span data-stu-id="d0c61-145">This functionality lets you keep up with customer demand for your web application and chatbot.</span></span> <span data-ttu-id="d0c61-146">См. дополнительные сведения об [автоматическом масштабирование][autoscaling] в центре архитектуры Azure.</span><span class="sxs-lookup"><span data-stu-id="d0c61-146">For more information on autoscale, see [Autoscaling best practices][autoscaling] in the Azure Architecture Center.</span></span>

<span data-ttu-id="d0c61-147">Для других вопросов масштабируемости, см. раздел [Контрольный список для обеспечения масштабируемости][scalability] в центре архитектуры Azure.</span><span class="sxs-lookup"><span data-stu-id="d0c61-147">For other scalability topics, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="d0c61-148">Безопасность</span><span class="sxs-lookup"><span data-stu-id="d0c61-148">Security</span></span>

<span data-ttu-id="d0c61-149">Для проверки подлинности пользователей в этом сценарии используется Azure Active Directory B2C (бизнес для клиента).</span><span class="sxs-lookup"><span data-stu-id="d0c61-149">This scenario uses Azure Active Directory B2C (Business 2 Consumer) to authenticate users.</span></span> <span data-ttu-id="d0c61-150">С помощью AAD B2C чат-бот не будет хранить конфиденциальной информации учетной записи или учетных данных.</span><span class="sxs-lookup"><span data-stu-id="d0c61-150">With AAD B2C, your chatbot doesn't store any sensitive customer account information or credentials.</span></span> <span data-ttu-id="d0c61-151">Дополнительные сведения см. в статье [Azure Active Directory B2C: регистрация и вход пользователей в приложения][aadb2c-docs].</span><span class="sxs-lookup"><span data-stu-id="d0c61-151">For more information, see [Azure Active Directory B2C overview][aadb2c-docs].</span></span>

<span data-ttu-id="d0c61-152">Информация, хранящаяся в Azure SQL Database, шифруются при хранении с помощью прозрачного шифрования данных (TDE).</span><span class="sxs-lookup"><span data-stu-id="d0c61-152">Information stored in Azure SQL Database is encrypted at rest with transparent data encryption (TDE).</span></span> <span data-ttu-id="d0c61-153">База данных SQL также предлагает функцию Always Encrypted, которая шифрует данные при выполнении и обработке запросов.</span><span class="sxs-lookup"><span data-stu-id="d0c61-153">SQL Database also offers Always Encrypted which encrypts data during querying and processing.</span></span> <span data-ttu-id="d0c61-154">Дополнительные сведения о безопасности базы данных SQL, см. в разделе [What is the Azure SQL Database service? Availability capabilities][sqlsecurity-docs] (Обзор службы базы данных SQL Azure. Возможности доступности).</span><span class="sxs-lookup"><span data-stu-id="d0c61-154">For more information on SQL Database security, see [Azure SQL Database security and compliance][sqlsecurity-docs].</span></span>

<span data-ttu-id="d0c61-155">Общие рекомендации по разработке безопасных решений см. в разделе [Azure Security Documentation][security].</span><span class="sxs-lookup"><span data-stu-id="d0c61-155">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="d0c61-156">Устойчивость</span><span class="sxs-lookup"><span data-stu-id="d0c61-156">Resiliency</span></span>

<span data-ttu-id="d0c61-157">Этот сценарий использует Базу данных SQL Azure для хранения данных о резервировании клиента.</span><span class="sxs-lookup"><span data-stu-id="d0c61-157">This scenario uses Azure SQL Database for storing customer reservations.</span></span> <span data-ttu-id="d0c61-158">База данных SQL включает базу данных избыточную в пределах зоны, группы отработки отказа, георепликацию и автоматические резервные копии.</span><span class="sxs-lookup"><span data-stu-id="d0c61-158">SQL Database includes zone redundant databases, failover groups, geo-replication, and automatic backups.</span></span> <span data-ttu-id="d0c61-159">Эти функции позволяют приложению продолжать работу, когда происходят события обслуживания или отключения.</span><span class="sxs-lookup"><span data-stu-id="d0c61-159">These features allow your application to continue running if there is a maintenance event or outage.</span></span> <span data-ttu-id="d0c61-160">Дополнительные сведения см. в статье [Azure SQL Database availability capabilities][sqlavailability-docs] (Возможности доступности базы данных Azure SQL).</span><span class="sxs-lookup"><span data-stu-id="d0c61-160">For more information, see [Azure SQL Database availability capabilities][sqlavailability-docs].</span></span>

<span data-ttu-id="d0c61-161">Для мониторинга работоспособности приложения этот сценарий использует Application Insights.</span><span class="sxs-lookup"><span data-stu-id="d0c61-161">To monitor the health of your application, this scenario uses Application Insights.</span></span> <span data-ttu-id="d0c61-162">С помощью Application Insights можно создавать оповещения и реагировать на проблемы с производительностью, которые будут влиять на качество обслуживания и доступность чат-бота.</span><span class="sxs-lookup"><span data-stu-id="d0c61-162">With Application Insights, you can generate alerts and respond to performance issues that would impact the customer experience and availability of the chatbot.</span></span> <span data-ttu-id="d0c61-163">Для дополнительных сведений см. [Что такое Application Insights?][appinsights-docs]</span><span class="sxs-lookup"><span data-stu-id="d0c61-163">For more information, see [What is Application Insights?][appinsights-docs]</span></span>

<span data-ttu-id="d0c61-164">Общее руководство по проектированию устойчивых решений см. в разделе [Проектирование устойчивых приложений для Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="d0c61-164">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="deploy-the-scenario"></a><span data-ttu-id="d0c61-165">Развертывание сценария</span><span class="sxs-lookup"><span data-stu-id="d0c61-165">Deploy the scenario</span></span>

<span data-ttu-id="d0c61-166">Этот сценарий состоит из трех компонентов для изучения областей, на которых вы сосредоточены больше всего.</span><span class="sxs-lookup"><span data-stu-id="d0c61-166">This scenario is divided into three components for you to explore areas that you are most focused on:</span></span>

* <span data-ttu-id="d0c61-167">[Компоненты инфраструктуры](#deploy-infrastructure-components).</span><span class="sxs-lookup"><span data-stu-id="d0c61-167">[Infrastructure components](#deploy-infrastructure-components).</span></span> <span data-ttu-id="d0c61-168">Используйте шаблон Azure Resource Manager для развертывания основных компонентов инфраструктуры службы приложений, веб-приложений, Application Insights, учетной записи хранения, SQL Server и базы данных.</span><span class="sxs-lookup"><span data-stu-id="d0c61-168">Use an Azure Resource Manger template to deploy the core infrastructure components of an App Service, Web App, Application Insights, Storage account, and SQL Server and database.</span></span>
* <span data-ttu-id="d0c61-169">[Чат-бот веб-приложения](#deploy-web-app-chatbot).</span><span class="sxs-lookup"><span data-stu-id="d0c61-169">[Web App Chatbot](#deploy-web-app-chatbot).</span></span> <span data-ttu-id="d0c61-170">Используйте Azure CLI для развертывания бота с помощью службы Azure Bot и приложения Интеллектуальной службы распознавания речи (LUIS).</span><span class="sxs-lookup"><span data-stu-id="d0c61-170">Use the Azure CLI to deploy a bot with the Bot Service and Language Understanding and Intelligent Services (LUIS) app.</span></span>
* <span data-ttu-id="d0c61-171">[Пример приложения чат-бота C#](#deploy-chatbot-c-application-code).</span><span class="sxs-lookup"><span data-stu-id="d0c61-171">[Sample C# chatbot application](#deploy-chatbot-c-application-code).</span></span> <span data-ttu-id="d0c61-172">Используйте Visual Studio, чтобы просмотреть образец кода C# бронирование отеля, и разверните бот в Azure.</span><span class="sxs-lookup"><span data-stu-id="d0c61-172">Use Visual Studio to review the sample hotel reservation C# application code and deploy to a bot in Azure.</span></span>

<span data-ttu-id="d0c61-173">**Предварительные требования**</span><span class="sxs-lookup"><span data-stu-id="d0c61-173">**Prerequisites.**</span></span> <span data-ttu-id="d0c61-174">Необходимо иметь учетную запись Azure.</span><span class="sxs-lookup"><span data-stu-id="d0c61-174">You must have an existing Azure account.</span></span> <span data-ttu-id="d0c61-175">Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.</span><span class="sxs-lookup"><span data-stu-id="d0c61-175">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>

### <a name="deploy-infrastructure-components"></a><span data-ttu-id="d0c61-176">Развертывание компонентов инфраструктуры</span><span class="sxs-lookup"><span data-stu-id="d0c61-176">Deploy infrastructure components</span></span>

<span data-ttu-id="d0c61-177">Чтобы развернуть компоненты инфраструктуры с помощью шаблона Azure Resource Manager, выполните следующие действия.</span><span class="sxs-lookup"><span data-stu-id="d0c61-177">To deploy the infrastructure components with an Azure Resource Manager template, perform the following steps.</span></span>

1. <span data-ttu-id="d0c61-178">Нажмите кнопку **Развертывание в Azure**.</span><span class="sxs-lookup"><span data-stu-id="d0c61-178">Click the **Deploy to Azure** button:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fcommerce-chatbot.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="d0c61-179">Подождите, пока откроется развертывание шаблона на портале Azure, а затем выполните следующие шаги.</span><span class="sxs-lookup"><span data-stu-id="d0c61-179">Wait for the template deployment to open in the Azure portal, then complete the following steps:</span></span>
   * <span data-ttu-id="d0c61-180">Выберите **Создать** группу ресурсов, а затем укажите в текстовом поле имя, например *myCommerceChatBotInfrastructure*.</span><span class="sxs-lookup"><span data-stu-id="d0c61-180">Choose to **Create new** resource group, then provide a name such as *myCommerceChatBotInfrastructure* in the text box.</span></span>
   * <span data-ttu-id="d0c61-181">В раскрывающемся списке **Расположение** выберите регион.</span><span class="sxs-lookup"><span data-stu-id="d0c61-181">Select a region from the **Location** drop-down box.</span></span>
   * <span data-ttu-id="d0c61-182">Укажите имя пользователя и надежный пароль для учетной записи администратора SQL Server.</span><span class="sxs-lookup"><span data-stu-id="d0c61-182">Provide a username and secure password for the SQL Server administrator account.</span></span>
   * <span data-ttu-id="d0c61-183">Прочтите условия использования и установите флажок **Я принимаю указанные выше условия**.</span><span class="sxs-lookup"><span data-stu-id="d0c61-183">Review the terms and conditions, then check **I agree to the terms and conditions stated above**.</span></span>
   * <span data-ttu-id="d0c61-184">Нажмите кнопку **Приобрести**.</span><span class="sxs-lookup"><span data-stu-id="d0c61-184">Select the **Purchase** button.</span></span>

<span data-ttu-id="d0c61-185">Подождите несколько минут до завершения развертывания.</span><span class="sxs-lookup"><span data-stu-id="d0c61-185">It takes a few minutes for the deployment to complete.</span></span>

### <a name="deploy-web-app-chatbot"></a><span data-ttu-id="d0c61-186">Развертывание чат-бота веб-приложения</span><span class="sxs-lookup"><span data-stu-id="d0c61-186">Deploy Web App chatbot</span></span>

<span data-ttu-id="d0c61-187">Чтобы создать чат-бот, используйте Azure CLI.</span><span class="sxs-lookup"><span data-stu-id="d0c61-187">To create the chatbot, use the Azure CLI.</span></span> <span data-ttu-id="d0c61-188">В следующем примере показано установку расширения CLI для службы программ-роботов, создание группы ресурсов, а затем развертывание бота, который использует Application Insights.</span><span class="sxs-lookup"><span data-stu-id="d0c61-188">The following example installs the CLI extension for Bot Service, creates a resource group, then deploys a bot that uses Application Insights.</span></span> <span data-ttu-id="d0c61-189">При поступлении соответствующего запроса системы, подтвердите подлинность своей учетной записи Microsoft и разрешите боту зарегистрировать себя в службе Azure Bot и приложении Интеллектуальной службы распознавания речи (LUIS).</span><span class="sxs-lookup"><span data-stu-id="d0c61-189">When prompted, authenticate your Microsoft account and allow the bot to register itself with the Bot Service and Language Understanding and Intelligent Services (LUIS) app.</span></span>

```azurecli-interactive
# Install the Azure CLI extension for the Bot Service
az extension add --name botservice --yes

# Create a resource group
az group create --name myCommerceChatbot --location eastus

# Create a Web App Chatbot that uses Application Insights
az bot create \
    --resource-group myCommerceChatbot \
    --name commerceChatbot \
    --location eastus \
    --kind webapp \
    --sku S1 \
    --insights eastus
```

### <a name="deploy-chatbot-c-application-code"></a><span data-ttu-id="d0c61-190">Развертывание кода приложения C# чат-бота</span><span class="sxs-lookup"><span data-stu-id="d0c61-190">Deploy chatbot C# application code</span></span>

<span data-ttu-id="d0c61-191">Пример приложения C# доступен в GitHub.</span><span class="sxs-lookup"><span data-stu-id="d0c61-191">A sample C# application is available on GitHub:</span></span> 

* [<span data-ttu-id="d0c61-192">Пример коммерческого бота C#</span><span class="sxs-lookup"><span data-stu-id="d0c61-192">Commerce Bot C# sample</span></span>](https://github.com/Microsoft/AzureBotServices-scenarios/tree/master/CSharp/Commerce/src)

<span data-ttu-id="d0c61-193">Пример приложения включает в себя компоненты аутентификации Azure Active Directory и интеграцию с Интеллектуальной службой распознавания речи (LUIS), компонентом Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="d0c61-193">The sample application includes the Azure Active Directory authentication components and integration with the Language Understanding and Intelligent Services (LUIS) component of Cognitive Services.</span></span> <span data-ttu-id="d0c61-194">Приложению требуется Visual Studio для создания и развертывания сценария.</span><span class="sxs-lookup"><span data-stu-id="d0c61-194">The application requires Visual Studio to build and deploy the scenario.</span></span> <span data-ttu-id="d0c61-195">Дополнительные сведения о настройке AAD B2C и приложения LUIS можно найти в документации репозитория GitHub.</span><span class="sxs-lookup"><span data-stu-id="d0c61-195">Additional information on configuring AAD B2C and the LUIS app can be found in the GitHub repo documentation.</span></span>

## <a name="pricing"></a><span data-ttu-id="d0c61-196">Цены</span><span class="sxs-lookup"><span data-stu-id="d0c61-196">Pricing</span></span>

<span data-ttu-id="d0c61-197">Чтобы изучить стоимость выполнения этого сценария, все услуги были предварительно сконфигурированы в калькуляторе стоимости.</span><span class="sxs-lookup"><span data-stu-id="d0c61-197">To explore the cost of running this scenario, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="d0c61-198">Чтобы узнать, как изменится цена для вашего конкретного варианта использования, измените соответствующие переменные в соответствии с ожидаемым трафиком.</span><span class="sxs-lookup"><span data-stu-id="d0c61-198">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="d0c61-199">Доступны три примера профилей затрат на основе количества сообщений, которое должен обрабатывать чат-бот:</span><span class="sxs-lookup"><span data-stu-id="d0c61-199">We have provided three sample cost profiles based on the number of messages you expect your chatbot to process:</span></span>

* <span data-ttu-id="d0c61-200">[Небольшой][small-pricing]: этот пример тарификации предполагает обработку до 10 000 сообщений в месяц.</span><span class="sxs-lookup"><span data-stu-id="d0c61-200">[Small][small-pricing]: this pricing example correlates to processing < 10,000 messages per month.</span></span>
* <span data-ttu-id="d0c61-201">[Средний][medium-pricing]: этот пример тарификации предполагает обработку до 500 000 сообщений в месяц.</span><span class="sxs-lookup"><span data-stu-id="d0c61-201">[Medium][medium-pricing]: this pricing example correlates to processing < 500,000 messages per month.</span></span>
* <span data-ttu-id="d0c61-202">[Большой][large-pricing]: этот пример тарификации предполагает обработку до 10 млн сообщений в месяц.</span><span class="sxs-lookup"><span data-stu-id="d0c61-202">[Large][large-pricing]: this pricing example correlates to processing < 10 million messages per month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="d0c61-203">Связанные ресурсы</span><span class="sxs-lookup"><span data-stu-id="d0c61-203">Related resources</span></span>

<span data-ttu-id="d0c61-204">Для набора руководств по использованию службы Azure Bot, см. [учебный узел][botservice-docs] документации.</span><span class="sxs-lookup"><span data-stu-id="d0c61-204">For a set of guided tutorials on leveraging the Azure Bot Service, see the [tutorial node][botservice-docs] of the documentation.</span></span>

<!-- links -->
[aadb2c-docs]: /azure/active-directory-b2c/active-directory-b2c-overview
[aad-docs]: /azure/active-directory/
[appinsights-docs]: /azure/application-insights/app-insights-overview
[appservice-docs]: /azure/app-service/
[architecture]: ./media/architecture-commerce-chatbot.png
[autoscaling]: ../../best-practices/auto-scaling.md
[availability]: ../../checklist/availability.md
[botservice-docs]: /azure/bot-service/
[cognitive-docs]: /azure/cognitive-services/
[resiliency]: ../../resiliency/index.md
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[scalability]: ../../checklist/scalability.md
[sqlavailability-docs]: /azure/sql-database/sql-database-technical-overview#availability-capabilities
[sqldatabase-docs]: /azure/sql-database/
[sqlsecurity-docs]: /azure/sql-database/sql-database-technical-overview#advanced-security-and-compliance
[qna-maker]: /azure/cognitive-services/QnAMaker/Overview/overview
[speech-api]: /azure/cognitive-services/speech/home
[translator]: /azure/cognitive-services/translator/translator-info-overview

[small-pricing]: https://azure.com/e/dce05b6184904c50b38e1a8654f726b6
[medium-pricing]: https://azure.com/e/304d17106afc480dbc414f9726078a03
[large-pricing]: https://azure.com/e/8319dd5e5e3d4f118f9029e32a80e887