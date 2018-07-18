---
title: Виртуальный собеседник Azure для резервирования отелей
description: Проверенное решение для создания виртуального собеседника для коммерческих приложений со службой Azure Bot, Cognitive Services и LUIS, базой данных Azure SQL и Application Insights.
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: 85bdc3194961bbbd8d89db34e5c56e4baa8d8599
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/06/2018
ms.locfileid: "37891384"
---
# <a name="conversational-azure-chatbot-for-hotel-reservations"></a><span data-ttu-id="58078-103">Виртуальный собеседник Azure для резервирования отелей</span><span class="sxs-lookup"><span data-stu-id="58078-103">Conversational Azure chatbot for hotel reservations</span></span>

<span data-ttu-id="58078-104">Этот пример сценария применим для предприятий, которым необходимо интегрировать виртуального собеседника в приложение.</span><span class="sxs-lookup"><span data-stu-id="58078-104">This example scenario is applicable to businesses that need integrate a conversational chatbot into applications.</span></span> <span data-ttu-id="58078-105">В этом решении чат-бот C# используется для сети отелей, чтобы клиенты могли проверить доступность и зарезервировать жилье через веб-узел и мобильное приложение.</span><span class="sxs-lookup"><span data-stu-id="58078-105">In this solution, a C# chatbot is used for a hotel chain that allows customers to check availability and book accommodation through a web or mobile application.</span></span>

<span data-ttu-id="58078-106">Примеры сценариев включают предоставление клиентам возможности просматривать удобства отеля и бронировать номера, просматривать в ресторане меню на вынос и заказывать еду, а также искать и заказывать распечатку фотографий.</span><span class="sxs-lookup"><span data-stu-id="58078-106">Example scenarios include providing a way for customers to view hotel availability and book rooms, review a restaurant take-out menu and place a food order, or search for and order prints of photographs.</span></span> <span data-ttu-id="58078-107">Чтобы реагировать на такие запросы клиентов, обычно компаниям нужно было нанимать и обучать специалистов службы поддержки, а клиентам приходилось ждать, пока представитель освободится и сможет оказать помощь.</span><span class="sxs-lookup"><span data-stu-id="58078-107">Traditionally, businesses would need to hire and train customer service agents to respond to these customer requests, and customers would have to wait until a representative is available to provide assistance.</span></span>

<span data-ttu-id="58078-108">Используя службы Azure, например службы программ-роботов и распознавания речи или SAPI, компании могут помогать клиентам и обрабатывать заказы или резервирования с помощью автоматизированных масштабируемых ботов.</span><span class="sxs-lookup"><span data-stu-id="58078-108">By using Azure services such as the Bot Service and Language Understanding or Speech API services, companies can assist customers and process orders or reservations with automated, scalable bots.</span></span>

## <a name="potential-use-cases"></a><span data-ttu-id="58078-109">Потенциальные варианты использования</span><span class="sxs-lookup"><span data-stu-id="58078-109">Potential use cases</span></span>

<span data-ttu-id="58078-110">Рассмотрите это решение для следующих вариантов использования.</span><span class="sxs-lookup"><span data-stu-id="58078-110">Consider this solution for the following use cases:</span></span>

* <span data-ttu-id="58078-111">Просмотрите в этом ресторане меню на вынос и сделайте заказ</span><span class="sxs-lookup"><span data-stu-id="58078-111">View restaurant take-out menu and order food</span></span>
* <span data-ttu-id="58078-112">Проверьте удобства отеля и забронируйте комнату</span><span class="sxs-lookup"><span data-stu-id="58078-112">Check hotel availability and reserve a room</span></span>
* <span data-ttu-id="58078-113">Найдите доступные фотографии и распечатайте их</span><span class="sxs-lookup"><span data-stu-id="58078-113">Search available photos and order prints</span></span>

## <a name="architecture"></a><span data-ttu-id="58078-114">Архитектура</span><span class="sxs-lookup"><span data-stu-id="58078-114">Architecture</span></span>

![Обзор архитектуры компонентов Azure в системе виртуального собеседника][architecture]

<span data-ttu-id="58078-116">Это решение включает виртуального собеседника, который исполняет функции консьержа в отеле.</span><span class="sxs-lookup"><span data-stu-id="58078-116">This solution covers a conversational bot that functions as a concierge for a hotel.</span></span> <span data-ttu-id="58078-117">Поток данных проходит через решение следующим образом.</span><span class="sxs-lookup"><span data-stu-id="58078-117">The data flows through the solution as follows:</span></span>

1. <span data-ttu-id="58078-118">Клиент может обращаться с чат-ботом с помощью мобильного телефона или веб-приложения.</span><span class="sxs-lookup"><span data-stu-id="58078-118">The customer accesses the chatbot with a mobile or web app.</span></span>
2. <span data-ttu-id="58078-119">Пользователь аутентифицируется с помощью Azure Active Directory B2C (бизнес для клиента).</span><span class="sxs-lookup"><span data-stu-id="58078-119">Using Azure Active Directory B2C (Business 2 Customer), the user is authenticated.</span></span>
3. <span data-ttu-id="58078-120">Взаимодействуя со службой программ-роботов, пользователь может запросить информацию об удобствах отеля.</span><span class="sxs-lookup"><span data-stu-id="58078-120">Interacting with the Bot Service, the user requests information about hotel availability.</span></span>
4. <span data-ttu-id="58078-121">Cognitive Services обрабатывает запросы естественного языка, чтобы понять о чем говорит клиент.</span><span class="sxs-lookup"><span data-stu-id="58078-121">Cognitive Services processes the natural language request to understand the customer communication.</span></span>
5. <span data-ttu-id="58078-122">Когда пользователь удовлетворен результатом, приложение-бот обновляет резервирование клиента в базе данных SQL Azure.</span><span class="sxs-lookup"><span data-stu-id="58078-122">After the user is happy with the results, the bot adds or updates the customer’s reservation in a SQL Database.</span></span>
6. <span data-ttu-id="58078-123">Служба Application Insights собирает данные телеметрии во время процесса, чтобы дать команде DevOps сведения о производительности бота и его использовании.</span><span class="sxs-lookup"><span data-stu-id="58078-123">Application Insights gathers runtime telemetry throughout the process to help the DevOps team with bot performance and usage.</span></span>

### <a name="components"></a><span data-ttu-id="58078-124">Компоненты</span><span class="sxs-lookup"><span data-stu-id="58078-124">Components</span></span>

* <span data-ttu-id="58078-125">[Azure Active Directory][aad-docs] — это мультитенантный облачный каталог и служба управления удостоверениями корпорации Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="58078-125">[Azure Active Directory][aad-docs] is Microsoft’s multi-tenant cloud-based directory and identity management service.</span></span> <span data-ttu-id="58078-126">Azure AD поддерживает соединитель B2C, позволяющий идентифицировать тех, кто использует внешние идентификаторы, например, Google, Facebook или учетную запись Microsoft.</span><span class="sxs-lookup"><span data-stu-id="58078-126">Azure AD supports a B2C connector allowing you to identify individuals using external IDs such as Google, Facebook, or a Microsoft Account.</span></span>
* <span data-ttu-id="58078-127">[Служба приложений Azure][appservice-docs] позволяет создавать и размещать веб-приложения на любых языках программирования без необходимости управлять инфраструктурой.</span><span class="sxs-lookup"><span data-stu-id="58078-127">[App Service][appservice-docs] enables you to build and host web applications in the programming language of your choice without managing infrastructure.</span></span>
* <span data-ttu-id="58078-128">[Служба программ-роботов][botservice-docs] предоставляет средства для создания, тестирования, развертывания и управления интеллектуальными ботами.</span><span class="sxs-lookup"><span data-stu-id="58078-128">[Bot Service][botservice-docs] provides tools to build, test, deploy, and manage intelligent bots.</span></span>
* <span data-ttu-id="58078-129">[Cognitive Services][cognitive-docs] позволяет использовать интеллектуальные алгоритмы, чтобы видеть, слышать, говорить, понимать и интерпретировать потребности клиентов с помощью обычных средств связи.</span><span class="sxs-lookup"><span data-stu-id="58078-129">[Cognitive Services][cognitive-docs] lets you use intelligent algorithms to see, hear, speak, understand and interpret your user needs through natural methods of communication.</span></span>
* <span data-ttu-id="58078-130">[База данных SQL][sqldatabase-docs] — это полностью управляемая облачная реляционная база данных, которая обеспечивает совместимость с ядром SQL Server.</span><span class="sxs-lookup"><span data-stu-id="58078-130">[SQL Database][sqldatabase-docs] is a fully managed relational cloud database service that provides SQL Server engine compatibility.</span></span>
* <span data-ttu-id="58078-131">[Application Insights][appinsights-docs] — это расширяемая служба управления производительностью приложений (APM), которая позволяет контролировать производительность приложений, таких как чат-бот.</span><span class="sxs-lookup"><span data-stu-id="58078-131">[Application Insights][appinsights-docs] is an extensible Application Performance Management (APM) service that lets you monitor the performance of applications, such as your chatbot.</span></span>

### <a name="alternatives"></a><span data-ttu-id="58078-132">Альтернативные варианты</span><span class="sxs-lookup"><span data-stu-id="58078-132">Alternatives</span></span>

* <span data-ttu-id="58078-133">[API распознавания речи Microsoft][speech-api] может использоваться для изменения взаимодействия клиентов с ботом.</span><span class="sxs-lookup"><span data-stu-id="58078-133">[Microsoft Speech API][speech-api] can be used to change how customers interface with your bot.</span></span>
* <span data-ttu-id="58078-134">[QnA Maker][qna-maker] можно использовать для быстрого добавления бота набора знаний из полуструктурированного содержимого, например, раздела "Вопросы и ответы".</span><span class="sxs-lookup"><span data-stu-id="58078-134">[QnA Maker][qna-maker] can be used as to quickly add knowledge to your bot from semi-structured content like an FAQ.</span></span>
* <span data-ttu-id="58078-135">[Перевод текстов][translator] — это служба, которая может легко добавить для бота многоязычную поддержку.</span><span class="sxs-lookup"><span data-stu-id="58078-135">[Translator Text][translator] is a service that you might consider to easily add multi-lingual support to your bot.</span></span>

## <a name="considerations"></a><span data-ttu-id="58078-136">Рекомендации</span><span class="sxs-lookup"><span data-stu-id="58078-136">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="58078-137">Доступность</span><span class="sxs-lookup"><span data-stu-id="58078-137">Availability</span></span>

<span data-ttu-id="58078-138">Это решение использует базу данных SQL Azure для хранения резервирования клиента.</span><span class="sxs-lookup"><span data-stu-id="58078-138">This solution uses Azure SQL Database for storing customer reservations.</span></span> <span data-ttu-id="58078-139">База данных SQL включает базу данных избыточную в пределах зоны, группы отработки отказа и георепликацию.</span><span class="sxs-lookup"><span data-stu-id="58078-139">SQL Database includes zone redundant databases, failover groups, and geo-replication.</span></span> <span data-ttu-id="58078-140">Дополнительные сведения см. в статье [Azure SQL Database availability capabilities][sqlavailability-docs] (Возможности доступности базы данных Azure SQL).</span><span class="sxs-lookup"><span data-stu-id="58078-140">For more information, see [Azure SQL Database availability capabilities][sqlavailability-docs].</span></span>

<span data-ttu-id="58078-141">Дополнительные сведения по другим вопросам доступности, см. раздел [Checkliste für die Verfügbarkeit][availability] (Контрольный список для обеспечения доступности) в центре архитектуры Azure.</span><span class="sxs-lookup"><span data-stu-id="58078-141">For other scalability topics, see the [availability checklist][availability] in the Azure Architecture Center.</span></span>

### <a name="scalability"></a><span data-ttu-id="58078-142">Масштабируемость</span><span class="sxs-lookup"><span data-stu-id="58078-142">Scalability</span></span>

<span data-ttu-id="58078-143">Это решение использует службу приложений Azure.</span><span class="sxs-lookup"><span data-stu-id="58078-143">This solution uses Azure App Service.</span></span> <span data-ttu-id="58078-144">С помощью службы приложений можно автоматически масштабировать количество экземпляров, которые запускает бот.</span><span class="sxs-lookup"><span data-stu-id="58078-144">With App Service, you can automatically scale the number of instances that run your bot.</span></span> <span data-ttu-id="58078-145">Эта функциональность позволяет не отставать от потребительского спроса на веб-приложение и чат-бот.</span><span class="sxs-lookup"><span data-stu-id="58078-145">This functionality lets you keep up with customer demand for your web application and chatbot.</span></span> <span data-ttu-id="58078-146">Дополнительные сведения о автомасштабировании см. в разделе [Automatische Skalierung][autoscaling] (Автоматическое масштабирование) в центре архитектуры Azure.</span><span class="sxs-lookup"><span data-stu-id="58078-146">For more information on autoscale, see [Autoscaling best practices][autoscaling] in the architecture center.</span></span>

<span data-ttu-id="58078-147">Для других вопросов масштабируемости, см. раздел [Контрольный список для обеспечения масштабируемости][scalability] в центре архитектуры Azure.</span><span class="sxs-lookup"><span data-stu-id="58078-147">For other scalability topics, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="58078-148">Безопасность</span><span class="sxs-lookup"><span data-stu-id="58078-148">Security</span></span>

<span data-ttu-id="58078-149">Для аутентификации пользователей это решение использует Azure Active Directory B2C (бизнес для клиента).</span><span class="sxs-lookup"><span data-stu-id="58078-149">This solution uses Azure Active Directory B2C (Business 2 Consumer) to authenticate users.</span></span> <span data-ttu-id="58078-150">С помощью AAD B2C чат-бот не будет хранить конфиденциальной информации учетной записи или учетных данных.</span><span class="sxs-lookup"><span data-stu-id="58078-150">With AAD B2C, your chatbot doesn't store any sensitive customer account information or credentials.</span></span> <span data-ttu-id="58078-151">Дополнительные сведения см. в статье [Azure Active Directory B2C: регистрация и вход пользователей в приложения][aadb2c-docs].</span><span class="sxs-lookup"><span data-stu-id="58078-151">For more information, see [Azure Active Directory B2C overview][aadb2c-docs].</span></span>

<span data-ttu-id="58078-152">Информация, хранящаяся в Azure SQL Database, шифруются при хранении с помощью прозрачного шифрования данных (TDE).</span><span class="sxs-lookup"><span data-stu-id="58078-152">Information stored in Azure SQL Database is encrypted at rest with transparent data encryption (TDE).</span></span> <span data-ttu-id="58078-153">База данных SQL также предлагает функцию Always Encrypted, которая шифрует данные при выполнении и обработке запросов.</span><span class="sxs-lookup"><span data-stu-id="58078-153">SQL Database also offers Always Encrypted which encrypts data during querying and processing.</span></span> <span data-ttu-id="58078-154">Дополнительные сведения о безопасности базы данных SQL, см. в разделе [What is the Azure SQL Database service? Availability capabilities][sqlsecurity-docs] (Обзор службы базы данных SQL Azure. Возможности доступности).</span><span class="sxs-lookup"><span data-stu-id="58078-154">For more information on SQL Database security, see [Azure SQL Database security and compliance][sqlsecurity-docs].</span></span>

<span data-ttu-id="58078-155">Общие рекомендации по разработке безопасных решений см. в разделе [Azure Security Documentation][security].</span><span class="sxs-lookup"><span data-stu-id="58078-155">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="58078-156">Устойчивость</span><span class="sxs-lookup"><span data-stu-id="58078-156">Resiliency</span></span>

<span data-ttu-id="58078-157">Это решение использует базу данных SQL Azure для хранения резервирования клиента.</span><span class="sxs-lookup"><span data-stu-id="58078-157">This solution uses Azure SQL Database for storing customer reservations.</span></span> <span data-ttu-id="58078-158">База данных SQL включает базу данных избыточную в пределах зоны, группы отработки отказа, георепликацию и автоматические резервные копии.</span><span class="sxs-lookup"><span data-stu-id="58078-158">SQL Database includes zone redundant databases, failover groups, geo-replication, and automatic backups.</span></span> <span data-ttu-id="58078-159">Эти функции позволяют приложению продолжать работу, когда происходят события обслуживания или отключения.</span><span class="sxs-lookup"><span data-stu-id="58078-159">These features allow your application to continue running in the event of a maintenance event or outage.</span></span> <span data-ttu-id="58078-160">Дополнительные сведения см. в статье [Azure SQL Database availability capabilities][sqlavailability-docs] (Возможности доступности базы данных Azure SQL).</span><span class="sxs-lookup"><span data-stu-id="58078-160">For more information, see [Azure SQL Database availability capabilities][sqlavailability-docs].</span></span>

<span data-ttu-id="58078-161">Для мониторинга работоспособности приложения, это решение использует Application Insights.</span><span class="sxs-lookup"><span data-stu-id="58078-161">To monitor the health of your application, this solution uses Application Insights.</span></span> <span data-ttu-id="58078-162">С помощью Application Insights можно создавать оповещения и реагировать на проблемы с производительностью, которые будут влиять на качество обслуживания и доступность чат-бота.</span><span class="sxs-lookup"><span data-stu-id="58078-162">With Application Insights, you can generate alerts and respond to performance issues that would impact the customer experience and availability of the chatbot.</span></span> <span data-ttu-id="58078-163">Для дополнительных сведений см. [Что такое Application Insights?][appinsights-docs]</span><span class="sxs-lookup"><span data-stu-id="58078-163">For more information, see [What is Application Insights?][appinsights-docs]</span></span>

<span data-ttu-id="58078-164">Общее руководство по проектированию устойчивых решений см. в разделе [Проектирование устойчивых приложений для Azure][resiliency].</span><span class="sxs-lookup"><span data-stu-id="58078-164">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="58078-165">Развертывание решения</span><span class="sxs-lookup"><span data-stu-id="58078-165">Deploy the solution</span></span>

<span data-ttu-id="58078-166">Это решение состоит из трех компонентов для изучения областей, на которых вы больше всего сосредоточены.</span><span class="sxs-lookup"><span data-stu-id="58078-166">This solution is divided into three components for you to explore areas that you are most focused on:</span></span>

* <span data-ttu-id="58078-167">[Компоненты инфраструктуры](#deploy-infrastructure-components).</span><span class="sxs-lookup"><span data-stu-id="58078-167">[Infrastructure components](#deploy-infrastructure-components).</span></span> <span data-ttu-id="58078-168">Используйте шаблон Azure Resource Manager для развертывания основных компонентов инфраструктуры службы приложений, веб-приложений, Application Insights, учетной записи хранения, SQL Server и базы данных.</span><span class="sxs-lookup"><span data-stu-id="58078-168">Use an Azure Resource Manger template to deploy the core infrastructure components of an App Service, Web App, Application Insights, Storage account, and SQL Server and database.</span></span>
* <span data-ttu-id="58078-169">[Чат-бот веб-приложения](#deploy-web-app-chatbot).</span><span class="sxs-lookup"><span data-stu-id="58078-169">[Web App Chatbot](#deploy-web-app-chatbot).</span></span> <span data-ttu-id="58078-170">Используйте Azure CLI для развертывания бота с помощью службы программ-роботов и приложения интеллектуальной службы распознавания речи (LUIS).</span><span class="sxs-lookup"><span data-stu-id="58078-170">Use the Azure CLI to deploy a bot with the Bot Service and Language Understanding and Intelligent Services (LUIS) app.</span></span>
* <span data-ttu-id="58078-171">[Пример приложения чат-бота C#](#deploy-chatbot-c-application-code).</span><span class="sxs-lookup"><span data-stu-id="58078-171">[Sample C# chatbot application](#deploy-chatbot-c-application-code).</span></span> <span data-ttu-id="58078-172">Используйте Visual Studio, чтобы просмотреть образец кода C# бронирование отеля, и разверните бот в Azure.</span><span class="sxs-lookup"><span data-stu-id="58078-172">Use Visual Studio to review the sample hotel reservation C# application code and deploy to a bot in Azure.</span></span>

<span data-ttu-id="58078-173">**Предварительные требования**</span><span class="sxs-lookup"><span data-stu-id="58078-173">**Prerequisites.**</span></span> <span data-ttu-id="58078-174">Необходимо иметь учетную запись Azure.</span><span class="sxs-lookup"><span data-stu-id="58078-174">You must have an existing Azure account.</span></span> <span data-ttu-id="58078-175">Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.</span><span class="sxs-lookup"><span data-stu-id="58078-175">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>

### <a name="deploy-infrastructure-components"></a><span data-ttu-id="58078-176">Развертывание компонентов инфраструктуры</span><span class="sxs-lookup"><span data-stu-id="58078-176">Deploy infrastructure components</span></span>

<span data-ttu-id="58078-177">Чтобы развернуть компоненты инфраструктуры с помощью шаблона Azure Resource Manager, выполните следующие действия.</span><span class="sxs-lookup"><span data-stu-id="58078-177">To deploy the infrastructure components with an Azure Resource Manager template, perform the following steps.</span></span>

1. <span data-ttu-id="58078-178">Нажмите кнопку **Развертывание в Azure**.</span><span class="sxs-lookup"><span data-stu-id="58078-178">Click the **Deploy to Azure** button:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fcommerce-chatbot.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="58078-179">Подождите, пока откроется развертывание шаблона на портале Azure, а затем выполните следующие шаги.</span><span class="sxs-lookup"><span data-stu-id="58078-179">Wait for the template deployment to open in the Azure portal, then complete the following steps:</span></span>
   * <span data-ttu-id="58078-180">Выберите **Создать** группу ресурсов, а затем укажите в текстовом поле имя, например *myCommerceChatBotInfrastructure*.</span><span class="sxs-lookup"><span data-stu-id="58078-180">Choose to **Create new** resource group, then provide a name such as *myCommerceChatBotInfrastructure* in the text box.</span></span>
   * <span data-ttu-id="58078-181">В раскрывающемся списке **Расположение** выберите регион.</span><span class="sxs-lookup"><span data-stu-id="58078-181">Select a region from the **Location** drop-down box.</span></span>
   * <span data-ttu-id="58078-182">Укажите имя пользователя и надежный пароль для учетной записи администратора SQL Server.</span><span class="sxs-lookup"><span data-stu-id="58078-182">Provide a username and secure password for the SQL Server administrator account.</span></span>
   * <span data-ttu-id="58078-183">Прочтите условия использования и установите флажок **Я принимаю указанные выше условия**.</span><span class="sxs-lookup"><span data-stu-id="58078-183">Review the terms and conditions, then check **I agree to the terms and conditions stated above**.</span></span>
   * <span data-ttu-id="58078-184">Нажмите кнопку **Приобрести**.</span><span class="sxs-lookup"><span data-stu-id="58078-184">Select the **Purchase** button.</span></span>

<span data-ttu-id="58078-185">Подождите несколько минут до завершения развертывания.</span><span class="sxs-lookup"><span data-stu-id="58078-185">It takes a few minutes for the deployment to complete.</span></span>

### <a name="deploy-web-app-chatbot"></a><span data-ttu-id="58078-186">Развертывание чат-бота веб-приложения</span><span class="sxs-lookup"><span data-stu-id="58078-186">Deploy Web App chatbot</span></span>

<span data-ttu-id="58078-187">Чтобы создать чат-бот, используйте Azure CLI.</span><span class="sxs-lookup"><span data-stu-id="58078-187">To create the chatbot, use the Azure CLI.</span></span> <span data-ttu-id="58078-188">В следующем примере показано установку расширения CLI для службы программ-роботов, создание группы ресурсов, а затем развертывание бота, который использует Application Insights.</span><span class="sxs-lookup"><span data-stu-id="58078-188">The following example installs the CLI extension for Bot Service, creates a resource group, then deploys a bot that uses Application Insights.</span></span> <span data-ttu-id="58078-189">При поступлении соответствующего запроса системы, подтвердите подлинность своей учетной записи Microsoft и позвольте боту зарегистрировать себя в службе программ-роботов и в приложении LUIS.</span><span class="sxs-lookup"><span data-stu-id="58078-189">When prompted, authenticate your Microsoft account and allow the bot to register itself with the Bot Service and Language Understanding and Intelligent Services (LUIS) app.</span></span>

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

### <a name="deploy-chatbot-c-application-code"></a><span data-ttu-id="58078-190">Развертывание кода приложения C# чат-бота</span><span class="sxs-lookup"><span data-stu-id="58078-190">Deploy chatbot C# application code</span></span>

<span data-ttu-id="58078-191">Пример приложения C# доступен в GitHub.</span><span class="sxs-lookup"><span data-stu-id="58078-191">A sample C# application is available on GitHub:</span></span> 

* [<span data-ttu-id="58078-192">Пример коммерческого бота C#</span><span class="sxs-lookup"><span data-stu-id="58078-192">Commerce Bot C# sample</span></span>](https://github.com/Microsoft/AzureBotServices-scenarios/tree/master/CSharp/Commerce/src)

<span data-ttu-id="58078-193">Пример приложения включает в себя компоненты аутентификации Azure Active Directory и интеграцию с интеллектуальной службой распознавания речи (LUIS), компонент Cognitive Services.</span><span class="sxs-lookup"><span data-stu-id="58078-193">The sample application includes the Azure Active Directory authentication components and integration with the Language Understanding and Intelligent Services (LUIS) component of Cognitive Services.</span></span> <span data-ttu-id="58078-194">Приложению требуется Visual Studio для создания и развертывания решения.</span><span class="sxs-lookup"><span data-stu-id="58078-194">The application requires Visual Studio to build and deploy the solution.</span></span> <span data-ttu-id="58078-195">Дополнительные сведения о настройке AAD B2C и приложения LUIS можно найти в документации репозитория GitHub.</span><span class="sxs-lookup"><span data-stu-id="58078-195">Additional information on configuring AAD B2C and the LUIS app can be found in the GitHub repo documentation.</span></span>

## <a name="pricing"></a><span data-ttu-id="58078-196">Цены</span><span class="sxs-lookup"><span data-stu-id="58078-196">Pricing</span></span>

<span data-ttu-id="58078-197">Чтобы изучить стоимость выполнения этого решения, все услуги были предварительно сконфигурированы в калькуляторе стоимости.</span><span class="sxs-lookup"><span data-stu-id="58078-197">To explore the cost of running this solution, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="58078-198">Чтобы узнать, как изменится цена для вашего конкретного варианта использования, измените соответствующие переменные в соответствии с ожидаемым трафиком.</span><span class="sxs-lookup"><span data-stu-id="58078-198">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="58078-199">Мы предоставили три примера профилей затрат, в основе которых количество сообщений, которое вы б хотели чтобы ваш бот обрабатывал:</span><span class="sxs-lookup"><span data-stu-id="58078-199">We have provided three sample cost profiles based on the amount of messages you expect your chatbot to process:</span></span>

* <span data-ttu-id="58078-200">[Небольшой][small-pricing]: коррелирует с обработкой < 10 000 сообщений в месяц.</span><span class="sxs-lookup"><span data-stu-id="58078-200">[Small][small-pricing]: this correlates to processing < 10,000 messages per month.</span></span>
* <span data-ttu-id="58078-201">[Средний][medium-pricing]: коррелирует с обработкой < 500 000 сообщений в месяц.</span><span class="sxs-lookup"><span data-stu-id="58078-201">[Medium][medium-pricing]: this correlates to processing < 500,000 messages per month.</span></span>
* <span data-ttu-id="58078-202">[Большой][large-pricing]: коррелирует с обработкой < 10 млн. сообщений в месяц.</span><span class="sxs-lookup"><span data-stu-id="58078-202">[Large][large-pricing]: this correlates to processing < 10 million messages per month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="58078-203">Связанные ресурсы</span><span class="sxs-lookup"><span data-stu-id="58078-203">Related Resources</span></span>

<span data-ttu-id="58078-204">Для набора руководств по использованию службы Azure Bot, см. [учебный узел][botservice-docs] документации.</span><span class="sxs-lookup"><span data-stu-id="58078-204">For a set of guided tutorials on leveraging the Azure Bot Service, see the [tutorial node][botservice-docs] of the documentation.</span></span>

<!-- links -->
[aadb2c-docs]: /azure/active-directory-b2c/active-directory-b2c-overview
[aad-docs]: /azure/active-directory/
[appinsights-docs]: /azure/application-insights/app-insights-overview
[appservice-docs]: /azure/app-service/
[architecture]: ./media/commerce-chatbot/architecture-commerce-chatbot.png
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