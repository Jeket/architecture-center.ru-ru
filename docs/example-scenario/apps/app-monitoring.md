---
title: Мониторинг веб-приложений в Azure
description: Мониторинг веб-приложения, размещенного в Службе приложений Azure.
author: adamboeglin
ms.date: 09/12/2018
ms.openlocfilehash: b1beb5cf5e29ab1ceb760bf95eab85d819b69342
ms.sourcegitcommit: 62945777e519d650159f0f963a2489b6bb6ce094
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/09/2018
ms.locfileid: "48876857"
---
# <a name="web-application-monitoring-on-azure"></a>Мониторинг веб-приложений в Azure

Платформа как услуга (PaaS) в Azure используется для управления вычислительными ресурсами и в некотором роде меняет способы мониторинга развертываний. Azure включает в себя несколько служб мониторинга, каждая из которых выполняет определенную роль. Вместе эти службы предоставляют комплексное решение для сбора, анализа и использования телеметрии из используемых приложений и ресурсов Azure.

В этом сценарии рассматриваются службы мониторинга, которые можно использовать, и описывается модель потока данных, используемая с несколькими источниками данных. Что касается мониторинга, то с развертыванием Azure могут работать многие средства и службы. В этом сценарии выбираются готовые службы, так как их легко использовать. Остальные параметры мониторинга рассматриваются далее.

## <a name="relevant-use-cases"></a>Варианты соответствующего использования

Рассмотрите этот сценарий для следующих вариантов использования:

- инструментирование веб-приложения для мониторинга телеметрии;
- сбор телеметрии интерфейса и сервера для приложения, развернутого в Azure;
- мониторинг метрик и квот, связанных со службами в Azure.

## <a name="architecture"></a>Архитектура

![Схема архитектуры мониторинга приложений][architecture]

В этом сценарии для размещения приложения и уровня данных используется управляемая среда Azure. Данные передаются в сценарии следующим образом:

1. пользователь взаимодействует с приложением;
2. с помощью браузера и службы приложений выполняется отправка телеметрии;
3. служба Application Insights собирает и анализирует данные о работоспособности приложения, его производительности и использовании;
4. разработчики и администраторы могут просматривать данные о работоспособности, производительности и использовании;
5. при помощи базы данных SQL Azure выполняется отправка телеметрии;
6. Azure Monitor собирает и анализирует метрики инфраструктуры и квоты;
7. служба Log Analytics собирает и анализирует журналы и метрики;
8. разработчики и администраторы могут просматривать данные о работоспособности, производительности и использовании.

### <a name="components"></a>Компоненты

- [Служба приложений Azure](/azure/app-service/) — это служба PaaS (платформа как услуга), используемая для создания и размещения приложений в управляемых виртуальных машинах. Вы сами управляете базовой вычислительной инфраструктурой, в которой работают приложения. Служба приложений выполняет мониторинг квот по использованию ресурсов и приложений метрики, журналов сведений о диагностике и оповещений, основанных на метриках. Более того, Application Insights можно использовать для создания [тестов доступности][availability-tests], с помощью которых можно проверять приложения из разных регионов.
- [Application Insights][application-insights] — это расширяемая служба управления производительностью приложений (APM) для разработчиков, поддерживаемая на нескольких платформах. Она используется для отслеживания приложений, обнаружения таких отклонений в работе приложений, как низкая производительность и ошибки, а также для отправки телеметрии на портал Azure. Также Application Insights можно использовать для ведения журнала, распределенной трассировки и метрик пользовательского приложения.
- [Azure Monitor][azure-monitor] предоставляет [метрики и журналы][metrics] инфраструктуры базового уровня для большинства служб в Azure. Пользователь может взаимодействовать с метриками несколькими способами, включая создание диаграмм метрик на портале Azure, доступ к метрикам через REST API или запрос метрик с помощью PowerShell или интерфейса командной строки. Также Azure Monitor предлагает использовать данные непосредственно в [Сбор журналов и метрик для служб Azure для использования в Log Analytics], в которых пользователь может запрашивать и комбинировать их с данными из других источников локально или в облаке.
- [Log Analytics][log-analytics] используется для сопоставления данных об использовании и производительности, собранных Application Insights, с данными о конфигурации и производительности ресурсов Azure, которые поддерживают работу приложения. Чтобы передать журналы аудита SQL Server в Log Analytics в этом сценарии используется [агент Azure Log Analytics][Azure Log Analytics agent]. В колонке Log Analytics на портале Azure можно записывать запросы и просматривать данные.

## <a name="considerations"></a>Рекомендации

Application Insights рекомендуется добавить в код при разработке с помощью [пакетов SDK для Application Insights][Application Insights SDKs] и настроить его для каждого приложения. Пакеты SDK с открытым исходным кодом доступны для большинства исполняющих сред. Чтобы расширить и контролировать собранные данные, включите пакеты SDK для тестирования и рабочего развертывания в процесс разработки. Основным требованием для приложений — это наличие прямой или непрямой связи с конечной точкой приема Applications Insights, размещенной с помощью адреса, ориентированного на Интернет. После этого можно добавить телеметрию или обогатить существующую коллекцию телеметрии.

Еще один быстрый способ приступить к работе — мониторинг сред выполнения. Управление собранными данными телеметрии должно выполнятся с помощью файлов конфигурации. Например, можно включить методы среды выполнения, в которые включены такие инструменты как [монитор состояний Application Insights][Application Insights Status Monitor], предназначенный для развертывания пакетов SDK в нужную папку, и добавить нужные конфигурации для начала мониторинга.

Также, как и Application Insights, Log Analytics предоставляет средства для [анализа данных из разных источников][analyzing data across sources], создания сложных запросов и [отправки упреждающих оповещений][sending proactive alerts] на основе указанных условий. Просматривать телеметрию можно также на [портале Azure][the Azure portal]. Log Analytics используется для добавления значения к таким существующим службам мониторинга как [Azure Monitor][Azure Monitor], а также для мониторинга локальных сред.

В Application Insights и Log Analytics используется [язык запросов Azure Log Analytics][Azure Log Analytics Query Language]. Также для анализа телеметрии, собранной Application Insights и Log Analytics, с помощью единого запроса можно использовать [запросы между ресурсами](https://azure.microsoft.com/blog/query-across-resources).

Azure Monitor, Application Insights и Log Analytics используются для отправки [оповещений](/azure/monitoring-and-diagnostics/monitoring-overview-alerts). Например, к оповещениям метрики Azure Monitor уровня платформы можно отнести загрузку ЦП, а к оповещениям метрик Application Insights уровня приложения — время отклика сервера. Azure Monitor уведомляет о новых событиях в журнале действий Azure, в то время как служба Log Analytics может уведомлять о метриках или данных о событиях для служб, настроенных для ее использования. [Функция "Унифицированные оповещения" в Azure Monitor](/azure/monitoring-and-diagnostics/monitoring-overview-unified-alerts) представляет собой новый, единый интерфейс оповещений в Azure, для которого используется разная классификация.

### <a name="alternatives"></a>Альтернативные варианты

С помощью популярных функций в этой статье описываются доступные параметры мониторинга. При этом у пользователя остается богатый выбор, включая возможность создания собственных механизмов ведения журнала. Добавлять службы мониторинга рекомендуется при создании уровней в решении. Ниже приведены некоторые расширения и альтернативные варианты.

- Объединение метрик Azure Monitor и Application Insights в Grafana с помощью [источника данных Azure Monitor для Grafana][Azure Monitor Data Source For Grafana].
- Служба [Data Dog][data-dog] включает в себя соединитель для Azure Monitor.
- Использование [службы автоматизации Azure][Azure Automation] для автоматизации функций мониторинга.
- Добавление обмена данными с помощью [решений ITSM][ITSM solutions].
- Расширение Log Analytics с помощью [решения по управлению][management solution].

### <a name="scalability-and-availability"></a>Доступность и масштабируемость

По большей части в данном сценарии основное внимание уделяется решениям PaaS для мониторинга, так как их удобно использовать для обеспечения доступности и масштабируемости, а также они поддерживаются соглашениями об уровне обслуживания (SLA). Например, использование службы приложений обеспечивает гарантированную [SLA][SLA] для доступности.

У Application Insights есть [ограничения][app-insights-limits] на количество обрабатываемых запросов в секунду. Если превысить лимит, то возникнет регулирование количества запросов сообщений. Чтобы этого избежать, необходимо выполнить [фильтрацию][message-filtering] или [выборку][message-sampling], что позволит уменьшить интенсивность передачи данных.

Тем не менее, за рекомендации по высокой доступности, предназначенные для запущенного приложения, ответственность несет разработчик. Например, дополнительные сведения о масштабе см. в разделе [Вопросы масштабируемости](#scalability-considerations) в эталонной архитектуре базового веб-приложения. После развертывания приложения можно использовать Application Insights и настроить тесты для [мониторинга доступности][monitor its availability].

### <a name="security"></a>Безопасность

Информация с ограниченным доступом и требования соответствия влияют на сбор, сохранение и хранение данных. Узнайте больше об обработке телеметрии в [Application Insights][application-insights] и [Log Analytics][log-analytics].

Следующие вопросы безопасности также могут быть применены.

- Разработка плана обработки личной информации в случае, когда разработчикам можно собирать собственные данные или обогащать существующую телеметрию.
- Рассмотрение сохранения данных. Например, Application Insights хранит данные телеметрии на протяжении 90 дней. Для доступа к архивным данным, которые хранятся в течение длительного времени, используется Microsoft Power BI, непрерывный экспорт или REST API. Применимы ставки за использование хранилища.
- Ограничение доступа к ресурсам Azure. Это позволит управлять доступом к данным и тем, кто может просматривать телеметрию из определенного приложения. Чтобы получить дополнительные сведения о блокировке доступа к мониторингу телеметрии, см. статью [Ресурсы, роли и контроль доступа в Application Insights][Resources, roles, and access control in Application Insights].
- Рассмотрение необходимости управлять доступом на чтение или запись в коде приложения, чтобы избежать добавления пользователями маркеров версий или тегов, которые ограничивают прием данных из приложения. При использовании Application Insights контроль над отдельными элементами данных после их отправки на ресурс отсутствует, поэтому если у пользователя есть доступ к каким-либо данным, он получает доступ ко всем данным отдельного ресурса.
- Добавление механизма [системы управления](/azure/security/governance-in-azure) для применения управления политикой или затратами в ресурсах Azure (при необходимости). В качестве примера можно использовать Log Analytics для связанного с безопасностью мониторинга, например, политики и управления доступом на основе ролей, или использовать [политику Azure](/azure/azure-policy/azure-policy-introduction) для создания и назначения определений политики, а также управления ими.
- Для отслеживания потенциальных проблем с безопасностью и получения полного представления о состоянии безопасности ресурсов Azure используйте [Центр безопасности Azure](/azure/security-center/security-center-intro).

## <a name="pricing"></a>Цены

Плата за мониторинг может быстро расти, поэтому рекомендуется рассмотреть ценообразование, получить общие сведения о контролируемых ресурсах и проверить соответствующую комиссию за каждую услугу. Azure Monitor бесплатно предоставляет [базовые метрики][basic metrics], в то время как стоимость мониторинга для [Application Insights][application-insights-pricing] и [Log Analytics][log-analytics] основывается на количестве принятых данных и числе выполненных тестов.

Для начала рекомендуется оценить затраты с помощью [калькулятора цен][pricing]. Чтобы узнать, как изменится цена для конкретного варианта использования, измените различные параметры в соответствии с ожидаемым развертыванием.

Данные телеметрии из Application Insights оправляются на портал Azure как во время отладки, так и после публикации приложения. В целях проверки и предотвращения расходов инструментируется ограниченный объем данных телеметрии. Чтобы добавить индикаторы, можно расширить ограничения телеметрии. Дополнительные сведения см. в статье [Выборка в Application Insights][Sampling in Application Insights].

После развертывания можно просмотреть показатели эффективности [Live Metrics Stream][Live Metrics Stream]. Эти данные не сохраняются. Пользователь просматривает метрики в реальном времени. Однако телеметрию можно сохранить для последующего анализа. Плата за данные Live Stream не взимается.

Плата за использование Log Analytics начисляется за каждый гигабайт данных, полученных службой. Каждый месяц первые 5 ГБ данных, получаемых службой Azure Log Analytics, предоставляются бесплатно. Также эти данные бесплатно сохраняются в течение первых 31 дней в рабочем пространстве Log Analytics.

## <a name="next-steps"></a>Дополнительная информация

Ознакомьтесь с материалами, которые помогут приступить к работе с помощью собственного решения для мониторинга.

[Базовое веб-приложение][Basic web application reference architecture]

[Запуск мониторинга веб-приложения ASP.NET][Start monitoring your ASP.NET Web Application]

[Сбор данных о виртуальных машинах Azure][Collect data about Azure Virtual Machines]

## <a name="related-resources"></a>Связанные ресурсы

[Общие сведения о службе Azure Monitor][Monitoring Azure applications and resources]

[Поиск и диагностика исключений во время выполнения с помощью Azure Application Insights][Find and diagnose run-time exceptions with Azure Application Insights]

<!-- links -->
[architecture]: ./media/architecture-app-monitoring.png
[availability-tests]: /azure/application-insights/app-insights-monitor-web-app-availability
[application-insights]: /azure/application-insights/app-insights-overview
[azure-monitor]: /azure/monitoring-and-diagnostics/monitoring-overview-azure-monitor
[metrics]: /azure/monitoring-and-diagnostics/monitoring-supported-metrics
[Сбор журналов и метрик для служб Azure для использования в Log Analytics]: /azure/log-analytics/log-analytics-azure-storage
[log-analytics]: /azure/log-analytics/log-analytics-overview
[Azure Log Analytics agent]: https://blogs.msdn.microsoft.com/sqlsecurity/2017/12/28/azure-log-analytics-oms-agent-now-collects-sql-server-audit-logs/
[application-insights-pricing]: https://azure.microsoft.com/pricing/details/application-insights/
[Application Insights SDKs]: /azure/application-insights/app-insights-asp-net
[Application Insights Status Monitor]: https://azure.microsoft.com/updates/application-insights-status-monitor-and-sdk-updated/
[analyzing data across sources]: /azure/log-analytics/log-analytics-dashboards
[sending proactive alerts]: /azure/log-analytics/log-analytics-alerts
[the Azure portal]: /azure/log-analytics/log-analytics-tutorial-dashboards
[Azure Log Analytics Query Language]: https://docs.loganalytics.io/docs/Learn
[cross-resource queries]: https://azure.microsoft.com/blog/query-across-resources/
[alerts]: /azure/monitoring-and-diagnostics/monitoring-overview-alerts
[Alerts (Preview)]: /azure/monitoring-and-diagnostics/monitoring-overview-unified-alerts
[Azure Monitor Data Source For Grafana]: https://grafana.com/plugins/grafana-azure-monitor-datasource
[Azure Automation]: /azure/automation/automation-intro
[ITSM solutions]: https://azure.microsoft.com/blog/itsm-connector-for-azure-is-now-generally-available/
[management solution]: /azure/monitoring/monitoring-solutions
[SLA]: https://azure.microsoft.com/support/legal/sla/app-service/v1_4/
[monitor its availability]: /azure/application-insights/app-insights-monitor-web-app-availability
[Resources, roles, and access control in Application Insights]: /azure/application-insights/app-insights-resources-roles-access-control
[basic metrics]: /azure/monitoring-and-diagnostics/monitoring-supported-metrics
[pricing]: https://azure.microsoft.com/pricing/calculator/#log-analyticsc126d8c1-ec9c-4e5b-9b51-4db95d06a9b1
[Sampling in Application Insights]: /azure/application-insights/app-insights-sampling
[Live Metrics Stream]: /azure/application-insights/app-insights-live-stream
[Basic web application reference architecture]: /azure/architecture/reference-architectures/app-service-web-app/basic-web-app#scalability-considerations
[Start monitoring your ASP.NET Web Application]: /azure/application-insights/quick-monitor-portal
[Collect data about Azure Virtual Machines]: /azure/log-analytics/log-analytics-quick-collect-azurevm
[Monitoring Azure applications and resources]: /azure/monitoring-and-diagnostics/monitoring-overview
[Find and diagnose run-time exceptions with Azure Application Insights]: /azure/application-insights/app-insights-tutorial-runtime-exceptions
[data-dog]: https://www.datadoghq.com/blog/azure-monitoring-enhancements/
[app-insights-limits]: /azure/azure-subscription-service-limits#application-insights-limits
[message-filtering]: /azure/application-insights/app-insights-api-filtering-sampling
[message-sampling]: /azure/application-insights/app-insights-sampling
