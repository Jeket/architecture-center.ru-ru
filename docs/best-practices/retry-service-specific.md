---
title: Конкретные рекомендации по использованию механизма повторов
titleSuffix: Best practices for cloud applications
description: Конкретные рекомендации по настройке механизма повторов.
author: dragon119
ms.date: 08/13/2018
ms.topic: best-practice
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 170a38f6b8a6c107670561e63f236e43af948d7d
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59641048"
---
# <a name="retry-guidance-for-specific-services"></a>Руководство по использованию механизма повторов для отдельных служб

Большинство служб Azure и клиентских пакетов SDK содержат механизм повтора. В то же время они отличаются, поскольку каждая служба имеет разные характеристики и требования, поэтому каждый механизм повтора настроен для конкретной службы. В этом руководстве представлены возможности механизма повтора для большинства служб Azure, а также сведения, помогающие использовать, адаптировать или расширять механизм повтора для такой службы.

Для получения общих рекомендаций по обработке временных сбоев и выполнения повторных попыток подключения и операций со службами и ресурсами обратитесь к разделу [Руководство по повторам](./transient-faults.md).

В следующей таблице перечислены функции механизма повтора для служб Azure, описанных в этом руководстве.

| **Служба** | **Ключевые возможности** | **Настройки политики** | **Область** | **Функции телеметрии** |
| --- | --- | --- | --- | --- |
| **[Azure Active Directory](#azure-active-directory)** |Машинный код в библиотеке ADAL |Внедренные в библиотеку ADAL |Внутренний |Нет |
| **[Cosmos DB](#cosmos-db)** |Машинный код в службе |Ненастраиваемые |Глобальные |TraceSource |
| **Data Lake Store** |Машинный код в клиенте |Ненастраиваемые |Индивидуальные операции |Нет |
| **[Центры событий](#event-hubs)** |Машинный код в клиенте |Программный |Клиент |Нет |
| **[Центр Интернета вещей](#iot-hub)** |Машинный код в клиентском пакете SDK |Программный |Клиент |Нет |
| **[Кэш Redis](#azure-redis-cache)** |Машинный код в клиенте |Программный |Клиент |TextWriter |
| **[Служба поиска](#azure-search)** |Машинный код в клиенте |Программный |Клиент |Трассировка событий Windows или пользовательская |
| **[Служебная шина](#service-bus)** |Машинный код в клиенте |Программный |Диспетчер пространств имен, фабрика сообщений и клиент |Трассировка событий Windows |
| **[Service Fabric](#service-fabric)** |Машинный код в клиенте |Программный |Клиент |Нет |
| **[База данных SQL с ADO.NET](#sql-database-using-adonet)** |[Polly](#transient-fault-handling-with-polly) |Декларативные и программные |Единые инструкции или блоки кода |Пользовательская |
| **[База данных SQL с Entity Framework](#sql-database-using-entity-framework-6)** |Машинный код в клиенте |Программный |Глобальные каждого домена приложения |Нет |
| **[База данных SQL с Entity Framework Core](#sql-database-using-entity-framework-core)** |Машинный код в клиенте |Программный |Глобальные каждого домена приложения |Нет |
| **[Хранилище](#azure-storage)** |Машинный код в клиенте |Программный |Клиентские и индивидуальные операции |TraceSource |

> [!NOTE]
> Сейчас для большинства встроенных механизмов повтора Azure невозможно применить другую политику повтора для других типов ошибок и исключений. Вам следует настроить политику, которая обеспечит оптимальный средний уровень производительности и доступности. Одним из способов настройки политики является анализ файлов журнала для определения типа возникающих временных сбоев.

<!-- markdownlint-disable MD024 MD033 -->

## <a name="azure-active-directory"></a>Azure Active Directory

Azure Active Directory (Azure AD) — это комплексное решение по управлению идентификацией и доступом к облаку, которое включает службы основных каталогов, функции расширенного управления удостоверениями и доступа к приложениям, а также средства обеспечения безопасности. Azure AD также предлагает разработчикам платформу управления удостоверениями для доставки элементов контроля доступа к приложениям на основе централизованной политики и правил.

> [!NOTE]
> Для получения рекомендаций по удостоверению управляемой службы конечных точек см. статью [Использование управляемого удостоверения службы (MSI) виртуальной машины Azure для получения маркера](/azure/active-directory/managed-service-identity/how-to-use-vm-token#error-handling)

### <a name="retry-mechanism"></a>Механизм повтора

Библиотека аутентификации для Active Directory (ADAL) имеет встроенный механизм повторов для службы Azure Active Directory. Чтобы избежать непредвиденных блокировок, мы рекомендуем разработчикам **не** применять повторные попытки в своих библиотеках и приложениях, а использовать для этого библиотеку ADAL.

### <a name="retry-usage-guidance"></a>Руководство по использованию механизма повторов

При использовании Azure Active Directory придерживайтесь следующих рекомендаций.

- Везде, где это возможно, используйте библиотеку ADAL и ее встроенный механизм повторных попыток.
- Если вы используете REST API для Azure Active Directory и получаете код результата 429 (слишком много запросов) или ошибку в диапазоне 5xx, повторите операцию. Не следует выполнять повтор при возникновении других ошибок.
- В пакетных сценариях с Azure Active Directory рекомендуется использование экспоненциальных политик отсрочки.

Для начала установите следующие параметры для операций повтора. Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.

| **Контекст** | **Максимальная задержка<br />примера целевого E2E** | **Стратегия повторов** | **Параметры** | **Значения** | **Принцип работы** |
| --- | --- | --- | --- | --- | --- |
| Интерактивный, пользовательский интерфейс<br />или передний план |2 с |FixedInterval |Число повторных попыток<br />Интервал попытки<br />Первый быстрый повтор |3<br />500 мс<br />true |Попытка 1 — задержка 0 с<br />Попытка 2 — задержка 500 мс<br />попытка 3 — задержка 500 мс |
| Фоновый<br /> или пакетный |60 с |ExponentialBackoff |Число повторных попыток<br />Минимальная задержка<br />Максимальная задержка<br />Разностная задержка<br />Первый быстрый повтор |5<br />0 с<br />60 с<br />2 с<br />false |Попытка 1 — задержка 0 с<br />Попытка 2 — задержка 2 с<br />Попытка 3 — задержка 6 с<br />Попытка 4 — задержка 14 с<br />Попытка 5 — задержка 30 с |

### <a name="more-information"></a>Дополнительные сведения

- [Библиотеки аутентификации Azure Active Directory][adal]

## <a name="cosmos-db"></a>База данных Cosmos

Cosmos DB — это полностью управляемая база данных, которая поддерживает несколько моделей и данные JSON без схемы данных. Она обеспечивает настраиваемую и надежную производительность, собственную обработку транзакций на основе JavaScript и разработана для облачной службы с гибким масштабированием.

### <a name="retry-mechanism"></a>Механизм повтора

Класс `DocumentClient` автоматически осуществляет новую попытку после сбоя. Чтобы задать количество повторных попыток и максимальное время ожидания, настройте свойство [ConnectionPolicy.RetryOptions]. Исключения клиента находятся за пределами действия политики повтора или не являются временными ошибками.

Если Cosmos DB выполняет для клиента регулирование, возвращается ошибка HTTP 429. Проверьте код состояния в `DocumentClientException`.

### <a name="policy-configuration"></a>Настройки политики

В следующей таблице показаны значения по умолчанию для класса `RetryOptions`.

| Параметр | Значение по умолчанию | ОПИСАНИЕ |
| --- | --- | --- |
| MaxRetryAttemptsOnThrottledRequests |9 |Максимальное число повторных попыток, когда запрос завершается сбоем из-за ограничения частоты запросов для клиента в Cosmos DB. |
| MaxRetryWaitTimeInSeconds |30 |Максимальное время повтора в секундах. |

### <a name="example"></a>Пример

```csharp
DocumentClient client = new DocumentClient(new Uri(endpoint), authKey); ;
var options = client.ConnectionPolicy.RetryOptions;
options.MaxRetryAttemptsOnThrottledRequests = 5;
options.MaxRetryWaitTimeInSeconds = 15;
```

### <a name="telemetry"></a>Телеметрия

Повторные попытки регистрируются как неструктурированные сообщения трассировки в .NET **TraceSource**. Необходимо настроить **TraceListener** для сбора данных о событиях и записи этих данных в журнал с подходящим назначением.

Например, если добавить в файл App.config следующий фрагмент кода, трассировка будет создаваться в текстовом файле в том же расположении, что и исполняемый файл:

```xml
<configuration>
  <system.diagnostics>
    <switches>
      <add name="SourceSwitch" value="Verbose"/>
    </switches>
    <sources>
      <source name="DocDBTrace" switchName="SourceSwitch" switchType="System.Diagnostics.SourceSwitch" >
        <listeners>
          <add name="MyTextListener" type="System.Diagnostics.TextWriterTraceListener" traceOutputOptions="DateTime,ProcessId,ThreadId" initializeData="CosmosDBTrace.txt"></add>
        </listeners>
      </source>
    </sources>
  </system.diagnostics>
</configuration>
```

## <a name="event-hubs"></a>Центры событий;

Центры событий Azure — это служба обработки данных телеметрии с высокой степенью масштабируемости. Она собирает, преобразовывает и хранит миллионы событий.

### <a name="retry-mechanism"></a>Механизм повтора

Механизм повторных попыток в клиентской библиотеке для Центров событий Azure управляется свойством `RetryPolicy` в классе `EventHubClient`. По умолчанию, если концентратор Azure возвращает временную ошибку `EventHubsException` или `OperationCanceledException`, используется политика повторных попыток с экспоненциальным увеличением задержки.

### <a name="example"></a>Пример

```csharp
EventHubClient client = EventHubClient.CreateFromConnectionString("[event_hub_connection_string]");
client.RetryPolicy = RetryPolicy.Default;
```

### <a name="more-information"></a>Дополнительные сведения

[Клиентская библиотека .NET Standard для Центров событий Azure](https://github.com/Azure/azure-event-hubs-dotnet)

## <a name="iot-hub"></a>Центр Интернета вещей

Центр Интернета вещей Azure — это служба подключения, мониторинга и администрирования устройств для разработки приложений Интернета вещей (IoT).

### <a name="retry-mechanism"></a>Механизм повтора

С помощью пакета SDK для устройств Azure IoT можно обнаруживать ошибки в сети, протоколе и приложениях. В зависимости от типа ошибки пакет SDK проверяет необходимость выполнения повторной попытки. Если ошибка является *устранимой*, пакет SDK выполняет повторные попытки, используя настроенную политику повтора.

Политика повтора по умолчанию использует *стратегию экспоненциальной отсрочки со случайным дрожанием*, но эту конфигурацию можно изменить.

### <a name="policy-configuration"></a>Настройки политики

Конфигурации политики отличаются для разных языков. См. дополнительные сведения см. об [конфигурации политики повтора Центра Интернета вещей](/azure/iot-hub/iot-hub-reliability-features-in-sdks#retry-policy-apis).

### <a name="more-information"></a>Дополнительные сведения

- [Политика повтора Центра Интернета вещей](/azure/iot-hub/iot-hub-reliability-features-in-sdks)
- [Устранение неполадок с отключением устройства Центра Интернета вещей](/azure/iot-hub/iot-hub-troubleshoot-connectivity)

## <a name="azure-redis-cache"></a>кэш Azure Redis

Кэш Redis для Azure является средством быстрого доступа к данным и службой кэша с низкой задержкой, созданной на основе кэша с открытым исходным кодом Redis. Он является безопасным, управляется Майкрософт и доступен из любого приложения в Azure.

Рекомендации в этом разделе основаны на использовании клиента StackExchange.Redis для доступа к кэшу. Список других подходящих клиентов можно найти на [веб-сайте Redis](https://redis.io/clients), и они могут применять различные механизмы.

Обратите внимание, что клиент StackExchange.Redis использует мультиплексирование через одно подключение. Рекомендуется создавать экземпляр клиента при запуске приложения и использовать этот экземпляр для всех операций в кэше. По этой причине подключение к кэшу происходит только один раз, и поэтому все рекомендации в этом разделе относится к политике повтора для этого исходного соединения &mdash; , а не для каждой операции, которая обращается к кэшу.

### <a name="retry-mechanism"></a>Механизм повтора

Клиент StackExchange.Redis использует класс диспетчера подключений, для настройки которого доступен ряд параметров, в том числе:

- **ConnectRetry**. Число повторных попыток при сбое подключения к кэшу.
- **ReconnectRetryPolicy**. Используемая стратегия повторных попыток.
- **ConnectTimeout**. Максимальное время ожидания в миллисекундах.

### <a name="policy-configuration"></a>Настройки политики

Политики повтора настраиваются программным способом с помощью параметров клиента перед подключением к кэшу. Для этого нужно создать экземпляр класса **ConfigurationOptions**, заполнить его свойства и передать ему метод **Connect**.

Встроенные классы поддерживают линейные (постоянные) паузы и экспоненциальное увеличение задержки с псевдослучайными интервалами. Также вы можете создать пользовательскую политику повторных попыток, реализовав интерфейс **IReconnectRetryPolicy**.

Код следующего примера настраивает стратегию повторных попыток с экспоненциальным увеличением задержки.

```csharp
var deltaBackOffInMilliseconds = TimeSpan.FromSeconds(5).Milliseconds;
var maxDeltaBackOffInMilliseconds = TimeSpan.FromSeconds(20).Milliseconds;
var options = new ConfigurationOptions
{
    EndPoints = {"localhost"},
    ConnectRetry = 3,
    ReconnectRetryPolicy = new ExponentialRetry(deltaBackOffInMilliseconds, maxDeltaBackOffInMilliseconds),
    ConnectTimeout = 2000
};
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

Кроме того, можно указать параметры в виде строки и передать эти данные методу **Connect** . Обратите внимание, что свойство **ReconnectRetryPolicy** нельзя задать таким способом, а можно только с помощью кода.

```csharp
var options = "localhost,connectRetry=3,connectTimeout=2000";
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

Также вы можете указать параметры непосредственно при подключении к кэшу.

```csharp
var conn = ConnectionMultiplexer.Connect("redis0:6380,redis1:6380,connectRetry=3");
```

Дополнительные сведения см. в [документации по StackExchange.Redis](https://stackexchange.github.io/StackExchange.Redis/Configuration).

Следующая таблица показывает значения по умолчанию для встроенной политики повторов.

| **Контекст** | **Параметр** | **Значение по умолчанию**<br />(версия 1.2.2) | **Значение** |
| --- | --- | --- | --- |
| Параметры конфигурации |ConnectRetry<br /><br />ConnectTimeout<br /><br />SyncTimeout<br /><br />ReconnectRetryPolicy |3<br /><br />Максимально 5000 мс плюс SyncTimeout<br />1000<br /><br />LinearRetry 5000 мс |Количество попыток повторов подключения во время начального подключения.<br />Время ожидания (мс) для операций подключения. Без задержки между повторными попытками.<br />Время (мс) для синхронных операций.<br /><br />Повторы через каждые 5000 мс.|

> [!NOTE]
> Для синхронных операций можно добавить `SyncTimeout` для определения совокупного времени задержки, но слишком низкое значение этого параметра приведет к чрезмерному времени ожидания. См. статью [Способы устранения проблем с кэшем Redis для Azure][redis-cache-troubleshoot]. Обычно следует избегать синхронных операций и использовать асинхронные везде, где это возможно. Дополнительные сведения см. в статье [Конвейеры и мультиплексоры](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/PipelinesMultiplexers.md).

### <a name="retry-usage-guidance"></a>Руководство по использованию механизма повторов

При использовании кэша Redis для Azure, придерживайтесь следующих рекомендаций.

- Клиент StackExchange Redis управляет собственными операциями повтора, но только при установлении соединения с кэшем при первом запуске приложения. Вы можете настроить время ожидания подключения, число повторных попыток подключения и паузы между ними, но эта политика повторов не применяется к операциям с кэшем.
- Вместо использования большого числа повторных попыток рассмотрите возможность возврата к исходному источнику данных.

### <a name="telemetry"></a>Телеметрия

Можно собирать сведения о подключении (но не о других операциях) с помощью **TextWriter**.

```csharp
var writer = new StringWriter();
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);
```

Ниже показан пример генерируемых выходных данных.

```text
localhost:6379,connectTimeout=2000,connectRetry=3
1 unique nodes specified
Requesting tie-break from localhost:6379 > __Booksleeve_TieBreak...
Allowing endpoints 00:00:02 to respond...
localhost:6379 faulted: SocketFailure on PING
localhost:6379 failed to nominate (Faulted)
> UnableToResolvePhysicalConnection on GET
No masters detected
localhost:6379: Standalone v2.0.0, master; keep-alive: 00:01:00; int: Connecting; sub: Connecting; not in use: DidNotRespond
localhost:6379: int ops=0, qu=0, qs=0, qc=1, wr=0, sync=1, socks=2; sub ops=0, qu=0, qs=0, qc=0, wr=0, socks=2
Circular op-count snapshot; int: 0 (0.00 ops/s; spans 10s); sub: 0 (0.00 ops/s; spans 10s)
Sync timeouts: 0; fire and forget: 0; last heartbeat: -1s ago
resetting failing connections to retry...
retrying; attempts left: 2...
...
```

### <a name="examples"></a>Примеры

Следующий пример кода настраивает постоянную (линейную) задержку между повторными попытками при инициализации клиента StackExchange.Redis. В этом примере показано, как настроить конфигурацию с помощью экземпляра **ConfigurationOptions**.

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    var retryTimeInMilliseconds = TimeSpan.FromSeconds(4).Milliseconds; // delay between retries

                    // Using object-based configuration.
                    var options = new ConfigurationOptions
                                        {
                                            EndPoints = { "localhost" },
                                            ConnectRetry = 3,
                                            ReconnectRetryPolicy = new LinearRetry(retryTimeInMilliseconds)
                                        };
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

В следующем примере показано, как настроить конфигурацию, указав параметры в строковом формате. Время ожидания подключения обозначает максимальный период времени для ожидания соединения с кэшем, но не задержку между повторными попытками. Обратите внимание, что свойство **ReconnectRetryPolicy** можно задавать только в коде.

```csharp
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using StackExchange.Redis;

namespace RetryCodeSamples
{
    class CacheRedisCodeSamples
    {
        public async static Task Samples()
        {
            var writer = new StringWriter();
            {
                try
                {
                    // Using string-based configuration.
                    var options = "localhost,connectRetry=3,connectTimeout=2000";
                    ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options, writer);

                    // Store a reference to the multiplexer for use in the application.
                }
                catch
                {
                    Console.WriteLine(writer.ToString());
                    throw;
                }
            }
        }
    }
}
```

Дополнительные примеры см. в разделе [Конфигурация](https://github.com/StackExchange/StackExchange.Redis/blob/master/docs/Configuration.md) на веб-сайте проекта.

### <a name="more-information"></a>Дополнительные сведения

- [Веб-сайт Redis](https://redis.io/)

## <a name="azure-search"></a>Поиск Azure

Поиск Azure является мощным и сложным инструментом поиска для сайта или приложения с возможностями быстрой и простой настройки результатов поиска и построения информативных и точных моделей ранжирования.

### <a name="retry-mechanism"></a>Механизм повтора

Поведение повтора в пакете SDK для Поиска Azure контролируется в методе `SetRetryPolicy` класса [SearchServiceClient] и [SearchIndexClient]. Повтор политики по умолчанию осуществляется с экспоненциальной задержкой, если Поиск Azure возвращает ответ с кодом 5xx или 408 (истекло время ожидания запроса).

### <a name="telemetry"></a>Телеметрия

Используйте трассировку событий Windows или осуществляйте трассировку путем регистрации пользовательского поставщика трассировки. Дополнительные сведения см. в [документации по AutoRest][autorest].

## <a name="service-bus"></a>Служебная шина Azure

Служебная шина является облачной платформой обмена сообщениями, которая предоставляет слабосвязанный обмен сообщениями с улучшенным масштабированием и устойчивостью для компонентов приложения, размещенных в облаке или локально.

### <a name="retry-mechanism"></a>Механизм повтора

Служебная шина реализует повторы с помощью реализации базового класса [RetryPolicy](/dotnet/api/microsoft.servicebus.retrypolicy) . Все клиенты служебной шины предоставляют свойство **RetryPolicy**, которое может быть присвоено одной из реализаций базового класса **RetryPolicy**. Встроенные реализации

- [Класс RetryExponential](/dotnet/api/microsoft.servicebus.retryexponential). Этот класс предоставляет свойства, управляющие интервалом отсрочки, числом попыток и свойством **TerminationTimeBuffer** , используемым для ограничения общего времени завершения операции.

- [Класс NoRetry](/dotnet/api/microsoft.servicebus.noretry). Используется, когда повторы на уровне API служебной шины не требуются, например, если повторные попытки управляются другим процессом в рамках пакета операций либо многоэтапной операции.

Действия служебной шины могут вызвать ряд исключений, как указано в статье [Исключения обмена сообщениями служебной шины](/azure/service-bus-messaging/service-bus-messaging-exceptions). Список предоставляет сведения о том, какие из исключений указывают, что повторение операции является целесообразным. Например **ServerBusyException** указывает, что клиенту следует подождать некоторое время и затем повторить операцию. Появление исключения **ServerBusyException** также вынуждает служебную шину переключиться в другой режим, в котором добавляется дополнительная 10-секундная задержка к вычисляемому времени задержки повтора операции. Этот режим сбрасывается после короткого времени.

Исключения, возвращаемые служебной шиной, предоставляют свойство **IsTransient** , указывающее, что клиент должен повторить операцию. Встроенная политика **RetryExponential** зависит от свойства **IsTransient** класса **MessagingException**, который является базовым классом для всех исключений служебной шины. Если создать пользовательские реализации базового класса **RetryPolicy**, то можно будет использовать сочетание типа исключения и свойство **IsTransient**, чтобы организовать более точное управление повторами. Например, можно обнаружить исключение **QuotaExceededException** и принять меры по очистке очереди, прежде чем повторить отправку сообщения в очередь.

### <a name="policy-configuration"></a>Настройки политики

Политики повтора настраиваются программно и могут быть установлены в качестве политики по умолчанию для **NamespaceManager** и **MessagingFactory** или по отдельности для каждого клиента обмена сообщениями. Чтобы задать политику повтора по умолчанию для сеанса обмена сообщениями, можно установить свойство **RetryPolicy** класса **NamespaceManager**.

```csharp
namespaceManager.Settings.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                                maxBackoff: TimeSpan.FromSeconds(30),
                                                                maxRetryCount: 3);
```

Чтобы задать политику повторов по умолчанию для всех клиентов, созданных из фабрики обмена сообщениями, установите свойство **RetryPolicy** класса **MessagingFactory**.

```csharp
messagingFactory.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                                    maxBackoff: TimeSpan.FromSeconds(30),
                                                    maxRetryCount: 3);
```

Чтобы задать политику повтора для клиента обмена сообщениями или переопределить политику по умолчанию, задайте свойство **RetryPolicy** с помощью экземпляра класса требуемой политики:

```csharp
client.RetryPolicy = new RetryExponential(minBackoff: TimeSpan.FromSeconds(0.1),
                                            maxBackoff: TimeSpan.FromSeconds(30),
                                            maxRetryCount: 3);
```

Нельзя задать политику повтора на уровне отдельных операций. Она применяется для всех операций для клиента обмена сообщениями.
Следующая таблица показывает значения по умолчанию для встроенной политики повторов.

| Параметр | Значение по умолчанию | Значение |
|---------|---------------|---------|
| Политика | Экспоненциальная | Экспоненциальная задержка. |
| MinimalBackoff | 0 | Минимальное время задержки. Добавляется ко всем интервалам повтора, вычисленным на основе значения deltaBackoff. |
| MaximumBackoff | 30 секунд | Максимальное время задержки. MaxBackoff используется, если расчетный интервал повторных попыток превышает значение MaxBackoff. |
| DeltaBackoff | 3 секунды | Интервал отсрочки между повторными попытками. Величина, кратная timespan, будет использоваться для последующих попыток. |
| TimeBuffer | 5 с | Буфер времени завершения, связанный с повторной попыткой. Повторные попытки будут прерваны, если оставшееся время меньше значения TimeBuffer. |
| MaxRetryCount | 10 | Максимальное число попыток. |
| ServerBusyBaseSleepTime | 10 с | Если последнее исключение было **ServerBusyException**, это значение будет добавляться к интервалу вычисляемых значений повторных попыток. Это значение не может быть изменено. |

### <a name="retry-usage-guidance"></a>Руководство по использованию механизма повторов

При использовании служебной шины придерживайтесь следующих рекомендаций.

- При использовании встроенной реализации **RetryExponential** не реализуйте операции резервирования, поскольку политика реагирует на исключения типа «Сервер занят» и автоматически переключается в режим повтора.
- Server Bus поддерживает функцию, называемую Сопряженные пространства имен, которая реализует автоматическое переключение на очередь резервного копирования в отдельном пространстве имен, если очередь в первичном пространстве имен дает сбой. Сообщения из вторичной очереди могут отправляться обратно в основную очередь после ее восстановления. Эта функция позволяет справиться с временными сбоями. Для получения дополнительных сведений обратитесь к разделу [Асинхронные шаблоны обмена сообщениями и высокий уровень доступности](/azure/service-bus-messaging/service-bus-async-messaging).

Начните с использования следующих параметров для операции повтора. Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.

| Context | Пример максимального значения задержки | Политика повтора | Параметры | Принцип работы |
|---------|---------|---------|---------|---------|
| Интерактивный, пользовательский интерфейс или передний план | 2 с*  | Экспоненциальная | MinimumBackoff = 0 <br/> MaximumBackoff = 30 с <br/> DeltaBackoff = 300 мс <br/> TimeBuffer = 300 мс <br/> MaxRetryCount = 2 | Попытка 1: задержка 0 с <br/> Попытка 2: задержка ~300 мс <br/> Попытка 3: задержка ~900 мс |
| Фоновый или пакетный | 30 секунд | Экспоненциальная | MinimumBackoff = 1 <br/> MaximumBackoff = 30 с <br/> DeltaBackoff = 1,75 с <br/> TimeBuffer = 5 с <br/> MaxRetryCount = 3 | Попытка 1: задержка ~1 с <br/> Попытка 2: задержка ~3 с <br/> Попытка 3: задержка ~6 мс <br/> Попытка 4: задержка ~13 мс |

\* Без значения дополнительной задержки, которая добавляется при получении ответа "Сервер занят".

### <a name="telemetry"></a>Телеметрия

Служебная шина регистрирует повторные попытки как события ETW с помощью **EventSource**. Необходимо присоединить службу **EventListener** к источнику события, чтобы зарегистрировать события и просмотреть в средстве просмотра производительности или записать данные о них в журнале с подходящим назначением. События повтора операций отображаются в следующей форме:

```text
Microsoft-ServiceBus-Client/RetryPolicyIteration
ThreadID="14,500"
FormattedMessage="[TrackingId:] RetryExponential: Operation Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05 at iteration 0 is retrying after 00:00:00.1000000 sleep because of Microsoft.ServiceBus.Messaging.MessagingCommunicationException: The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3, TimeStamp:9/5/2014 10:00:13 PM."
trackingId=""
policyType="RetryExponential"
operation="Get:https://retry-tests.servicebus.windows.net/TestQueue/?api-version=2014-05"
iteration="0"
iterationSleep="00:00:00.1000000"
lastExceptionType="Microsoft.ServiceBus.Messaging.MessagingCommunicationException"
exceptionMessage="The remote name could not be resolved: 'retry-tests.servicebus.windows.net'.TrackingId:6a26f99c-dc6d-422e-8565-f89fdd0d4fe3,TimeStamp:9/5/2014 10:00:13 PM"
```

### <a name="examples"></a>Примеры

В следующем примере кода показано, как задать политику повтора для

- Диспетчера пространства имен. Политика применяется ко всем операциям диспетчера и не может быть переопределена для отдельных операций.
- Фабрики сообщений. Политика применяется ко всем клиентам, созданным из этой фабрики, и не может быть переопределена при создании отдельных клиентов.
- Отдельных клиентов обмена сообщениями. После создания клиента можно задать политику повтора для этого клиента. Политика применяется ко всем операциям на этом клиенте.

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.ServiceBus;
using Microsoft.ServiceBus.Messaging;

namespace RetryCodeSamples
{
    class ServiceBusCodeSamples
    {
        private const string connectionString =
            @"Endpoint=sb://[my-namespace].servicebus.windows.net/;
                SharedAccessKeyName=RootManageSharedAccessKey;
                SharedAccessKey=C99..........Mk=";

        public async static Task Samples()
        {
            const string QueueName = "TestQueue";

            ServiceBusEnvironment.SystemConnectivity.Mode = ConnectivityMode.Http;

            var namespaceManager = NamespaceManager.CreateFromConnectionString(connectionString);

            // The namespace manager will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for all operations on the namespace manager.
                namespaceManager.Settings.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                if (!await namespaceManager.QueueExistsAsync(QueueName))
                {
                    await namespaceManager.CreateQueueAsync(QueueName);
                }
            }

            var messagingFactory = MessagingFactory.Create(
                namespaceManager.Address, namespaceManager.Settings.TokenProvider);
            // The messaging factory will have a default exponential policy with 10 retry attempts
            // and a 3 second delay delta.
            // Retry delays will be approximately 0 sec, 3 sec, 9 sec, 25 sec and the fixed 30 sec,
            // with an extra 10 sec added when receiving a ServiceBusyException.

            {
                // Set different values for the retry policy, used for clients created from it.
                messagingFactory.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);


                // Policies cannot be specified on a per-operation basis.
                var session = await messagingFactory.AcceptMessageSessionAsync();
            }

            {
                var client = messagingFactory.CreateQueueClient(QueueName);
                // The client inherits the policy from the factory that created it.


                // Set different values for the retry policy on the client.
                client.RetryPolicy =
                    new RetryExponential(
                        minBackoff: TimeSpan.FromSeconds(0.1),
                        maxBackoff: TimeSpan.FromSeconds(30),
                        maxRetryCount: 3);

                // Policies cannot be specified on a per-operation basis.
                var session = await client.AcceptMessageSessionAsync();
            }
        }
    }
}
```

### <a name="more-information"></a>Дополнительные сведения

- [Шаблоны асинхронного обмена сообщениями и высокий уровень доступности](/azure/service-bus-messaging/service-bus-async-messaging)

## <a name="service-fabric"></a>Service Fabric

Распространение надежных служб в кластере Service Fabric защищает от большинства потенциальных временных сбоев, которые описаны в этой статье. Но некоторые временные сбои все равно могут произойти. Например, если служба именования в момент получения запроса выполняет изменение маршрутизации, она создаст исключение. Если повторить этот же запрос через 100 миллисекунд, он, скорее всего, завершится успешно.

Service Fabric обрабатывает такие временные сбои на внутреннем уровне. Некоторые параметры можно настроить с помощью класса `OperationRetrySettings` при настройке служб. Пример такой настройки приведен в следующем коде. В большинстве случаев это не требуется, и можно использовать параметры по умолчанию.

```csharp
FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
{
    OperationTimeout = TimeSpan.FromSeconds(30)
};

var retrySettings = new OperationRetrySettings(TimeSpan.FromSeconds(15), TimeSpan.FromSeconds(1), 5);

var clientFactory = new FabricTransportServiceRemotingClientFactory(transportSettings);

var serviceProxyFactory = new ServiceProxyFactory((c) => clientFactory, retrySettings);

var client = serviceProxyFactory.CreateServiceProxy<ISomeService>(
    new Uri("fabric:/SomeApp/SomeStatefulReliableService"),
    new ServicePartitionKey(0));
```

### <a name="more-information"></a>Дополнительные сведения

- [Удаленная обработка исключений](/azure/service-fabric/service-fabric-reliable-services-communication-remoting#remoting-exception-handling)

## <a name="sql-database-using-adonet"></a>Использование Базы данных SQL с ADO.NET

База данных SQL является размещенной базой данных SQL различных размеров и является как стандартной (общей), так и премиальной (частной) службой.

### <a name="retry-mechanism"></a>Механизм повтора

База данных SQL не имеет встроенной поддержки выполнения повторных попыток, если доступ осуществляется с помощью ADO.NET. Однако коды возврата от запросов могут быть использованы для определения причины сбоя запроса. Дополнительные сведения о регулировании базы данных SQL см. в статье [Ограничения ресурсов базы данных SQL Azure](/azure/sql-database/sql-database-resource-limits). Список кодов ошибок см. в описании [кодов ошибок SQL для клиентских приложений Базы данных SQL](/azure/sql-database/sql-database-develop-error-messages).

Для реализации повторных попыток при обращении к Базе данных SQL вы можете использовать библиотеку Polly. См. руководство по [обработке временного сбоя с помощью Polly](#transient-fault-handling-with-polly).

### <a name="retry-usage-guidance"></a>Руководство по использованию механизма повторов

При доступе к базе данных SQL с помощью ADO.NET придерживайтесь следующих рекомендаций.

- Выберите соответствующую службу (общую или премиальную). Общий экземпляр может испытывать большие задержки подключения и регулирования чем обычно из-за использования другими клиентами общего сервера. Если требуется более прогнозируемая производительность и выполнение надежных операций с низкой задержкой, попробуйте выбрать премиальный вариант.
- Убедитесь, что выполнение повторов происходит на соответствующем уровне или области во избежание операций, не являющихся идемпотентными, что может вызвать несоответствие в данных. В идеальном случае все операции должны быть идемпотентными таким образом, чтобы их можно было повторить без возникновения несоответствия. Если это не так, повтор следует производить на уровне или в области, которая позволяет отменить все связанные изменения, если одна операция завершится ошибкой. Например, в области транзакций. Для получения дополнительных сведений обратитесь к разделу [Основы облачной службы. Уровень доступа к данным: обработка временных ошибок](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx#Idempotent_Guarantee).
- Не рекомендуется использование стратегии с фиксированным интервалом при работе с базой данных SQL Azure, кроме интерактивных сценариев при наличии нескольких попыток повтора с очень короткими интервалами. Вместо этого рекомендуется использовать стратегию экспоненциальной отсрочки для большинства сценариев.
- Выберите подходящее значение для времени ожидания подключения и команды при определении подключения. Слишком короткое время ожидания может привести к преждевременным сбоям подключений, когда база данных занята. Слишком длинное время ожидания может помешать правильной работе логики повторов из-за слишком долгого ожидания до обнаружения ошибки соединения. Значение времени ожидания — это компонент сквозной задержки; фактически это значение добавляется к величине задержки повтора, указанной в политике повтора для каждой попытки.
- Закройте соединение после определенного числа повторных попыток, даже если используется экспоненциальная логика повторных попыток, и повторите операцию с новым подключением. Повторение одной и той же операции несколько раз для одного соединения может стать фактором возникновения проблем с подключением. Пример использования этой технологии см. в статье [Основы облачной службы. Уровень доступа к данным: обработка временных ошибок](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx).
- Когда используется пул подключений (по умолчанию), существует вероятность того, что будет выбрано то же подключение из пула, даже после закрытия и повторного открытия соединения. Если это так, способом устранения подобной ситуации является вызов метода **ClearPool** класса **SqlConnection**, чтобы отметить подключение как не используемое повторно. Тем не менее это следует делать только после сбоя нескольких попыток соединения и только при обнаружении определенного класса временных ошибок, таких как время ожидания SQL (код ошибки -2), связанных со сбоями в подключениях.
- Если код доступа к данным использует транзакции, инициируемые как экземпляры **TransactionScope** , то логика повторных попыток должна включать повторное открытие подключения и инициацию новой области транзакции. По этой причине блок кода повторной попытки должен охватывать всю область транзакции.

Начните с использования следующих параметров для операции повтора. Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.

| **Контекст** | **Максимальная задержка<br />примера целевого E2E** | **Стратегия повторов** | **Параметры** | **Значения** | **Принцип работы** |
| --- | --- | --- | --- | --- | --- |
| Интерактивный, пользовательский интерфейс<br />или передний план |2 с |FixedInterval |Число повторных попыток<br />Интервал попытки<br />Первый быстрый повтор |3<br />500 мс<br />true |Попытка 1 — задержка 0 с<br />Попытка 2 — задержка 500 мс<br />попытка 3 — задержка 500 мс |
| Фоновый<br />или пакетный |30 с |ExponentialBackoff |Число повторных попыток<br />Минимальная задержка<br />Максимальная задержка<br />Разностная задержка<br />Первый быстрый повтор |5<br />0 с<br />60 с<br />2 с<br />false |Попытка 1 — задержка 0 с<br />Попытка 2 — задержка 2 с<br />Попытка 3 — задержка 6 с<br />Попытка 4 — задержка 14 с<br />Попытка 5 — задержка 30 с |

> [!NOTE]
> Целевые значения сквозной задержки подразумевают время ожидания по умолчанию для подключения к службе. При указании более длинного времени ожидания подключения величина сквозной задержки будет увеличена путем добавления этого дополнительного времени для каждой попытки повтора.

### <a name="examples"></a>Примеры

В этом разделе показано, как с помощью Polly обращаться к Базе данных SQL Azure с использованием набора политик повторных попыток, настроенных в классе `Policy`.

Следующий код демонстрирует метод расширения для класса `SqlCommand`, который вызывает `ExecuteAsync` с экспоненциальным увеличением задержки.

```csharp
public async static Task<SqlDataReader> ExecuteReaderWithRetryAsync(this SqlCommand command)
{
    GuardConnectionIsNotNull(command);

    var policy = Policy.Handle<Exception>().WaitAndRetryAsync(
        retryCount: 3, // Retry 3 times
        sleepDurationProvider: attempt => TimeSpan.FromMilliseconds(200 * Math.Pow(2, attempt - 1)), // Exponential backoff based on an initial 200ms delay.
        onRetry: (exception, attempt) =>
        {
            // Capture some info for logging/telemetry.
            logger.LogWarn($"ExecuteReaderWithRetryAsync: Retry {attempt} due to {exception}.");
        });

    // Retry the following call according to the policy.
    await policy.ExecuteAsync<SqlDataReader>(async token =>
    {
        // This code is executed within the Policy

        if (conn.State != System.Data.ConnectionState.Open) await conn.OpenAsync(token);
        return await command.ExecuteReaderAsync(System.Data.CommandBehavior.Default, token);

    }, cancellationToken);
}
```

Этот асинхронный метод расширения можно использовать следующим образом.

```csharp
var sqlCommand = sqlConnection.CreateCommand();
sqlCommand.CommandText = "[some query]";

using (var reader = await sqlCommand.ExecuteReaderWithRetryAsync())
{
    // Do something with the values
}
```

### <a name="more-information"></a>Дополнительные сведения

- [Основы облачной службы. Уровень доступа к данным: обработка временных ошибок](https://social.technet.microsoft.com/wiki/contents/articles/18665.cloud-service-fundamentals-data-access-layer-transient-fault-handling.aspx)

Общие рекомендации по повышению эффективности Базы данных SQL см. в руководстве по [обеспечению эластичности и производительности Базы данных SQL Azure](https://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx).

## <a name="sql-database-using-entity-framework-6"></a>Использование Базы данных SQL с Entity Framework 6

База данных SQL является размещенной базой данных SQL различных размеров и является как стандартной (общей), так и премиальной (частной) службой. Механизм Entity Framework является модулем объектно-реляционного сопоставления, который позволяет разработчикам .NET работать с реляционными данными с помощью специфических для домена объектов. Это исключает необходимость в большинстве кодов доступа к данным, которые обычно требуется писать разработчикам.

### <a name="retry-mechanism"></a>Механизм повтора

Поддержка повторных попыток при доступе к Базе данных SQL с помощью Entity Framework 6.0 и выше предоставляется через механизм, называемый [устойчивость подключения и логика повторных попыток](/ef/ef6/fundamentals/connection-resiliency/retry-logic). Ниже перечислены основные возможности механизма повтора.

- Интерфейс **IDbExecutionStrategy** является основной абстракцией. Этот интерфейс
  - Определяет синхронные и асинхронные методы **Execute** \*.
  - Определяет классы для непосредственного использования, или которые могут быть настроены на контекст базы данных как стратегию по умолчанию, с сопоставленным именем поставщика или сопоставленным именем поставщика и именем сервера. При настройке на контекст повторы происходят на уровне операций с отдельными базами данных, из которых несколько может соответствовать заданному контексту операции.
  - Определяет, когда и каким образом следует выполнить повтор при ошибке соединения.
- Включает несколько встроенных реализаций интерфейса **IDbExecutionStrategy** .
  - Значение по умолчанию — не повторять.
  - По умолчанию для базы данных SQL (автоматический режим) — не повторять, но проверять исключения и завершать их с предложением использовать стратегию базы данных SQL.
  - По умолчанию для базы данных SQL — экспоненциальный повтор (наследуется от базового класса) плюс логика обнаружения базы данных SQL.
- Это реализует стратегию экспоненциальной отсрочки, которая также включает случайный выбор.
- Встроенные классы повтора включают отслеживание состояния и не являются потокобезопасными. Однако они могут использоваться повторно после завершения текущей операции.
- В случае превышения заданного счетчика повторов результаты помещаются в новое исключение. Это не вызывает перенаправления текущего исключения.

### <a name="policy-configuration"></a>Настройки политики

Поддержка повторных попыток предоставляется при доступе к базе данных SQL с помощью Entity Framework 6.0 и более поздних версий. Политики повтора настраиваются программным способом. Нельзя изменить конфигурацию для каждой операции.

При настройке стратегии в контексте по умолчанию, необходимо указать функцию, которая создает новые стратегии по требованию. В следующем коде показано, как можно создать класс конфигурации повтора, который расширяет базовый класс **DbConfiguration** .

```csharp
public class BloggingContextConfiguration : DbConfiguration
{
  public BlogConfiguration()
  {
    // Set up the execution strategy for SQL Database (exponential) with 5 retries and 4 sec delay
    this.SetExecutionStrategy(
         "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4)));
  }
}
```

Затем можно указать это в качестве стратегии повторных попыток по умолчанию для всех операций с помощью метода **SetConfiguration** экземпляра **DbConfiguration** при запуске приложения. По умолчанию EF будет автоматически обнаруживать и использовать класс конфигурации.

```csharp
DbConfiguration.SetConfiguration(new BloggingContextConfiguration());
```

Класс конфигурации повторных попыток для контекста задается дополнением контекста класса атрибутом **DbConfigurationType** . Однако при наличии только одного класса конфигурации EF будет использовать его без необходимости дополнения контекста.

```csharp
[DbConfigurationType(typeof(BloggingContextConfiguration))]
public class BloggingContext : DbContext
```

Если необходимо использовать различные стратегии для конкретных операций или отключить повторы для определенных операций, то можно создать класс конфигурации, который позволяет приостановить или заменить стратегии, установив флаг **CallContext**. Класс конфигурации может использовать этот флаг для переключения стратегии или отключения стратегии, предоставленной вами, с последующим использованием стратегии по умолчанию. Дополнительные сведения см. в разделе [Приостановка выполнения стратегии](/ef/ef6/fundamentals/connection-resiliency/retry-logic#workaround-suspend-execution-strategy) (для EF6 и выше).

Иным способом использования стратегий повторов, определенных для отдельных операций, является создание экземпляра класса требуемой стратегии и передача ему нужных параметров. Затем можно вызвать его метод **ExecuteAsync** .

```csharp
var executionStrategy = new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(4));
var blogs = await executionStrategy.ExecuteAsync(
    async () =>
    {
        using (var db = new BloggingContext("Blogs"))
        {
            // Acquire some values asynchronously and return them
        }
    },
    new CancellationToken()
);
```

Самый простой способ использовать класс **DbConfiguration** — это обнаружить его в той же сборке, что и класс **DbContext**. Однако этот способ не подходит в случае, когда тот же контекст требуется в различных сценариях, таких как различные интерактивные и фоновые стратегии повторов. Если разные контексты выполняются в отдельных доменах приложений, можно использовать встроенную поддержку для указания классов конфигурации в файле конфигурации или задать ее явным образом с помощью кода. Если различные контексты должны выполняться в том же домене приложения, потребуется создание пользовательского решения.

Дополнительные сведения см. в статье [Конфигурация на основе кода (для EF6 и выше)](/ef/ef6/fundamentals/configuring/code-based).

Следующая таблица показывает значения по умолчанию для встроенной политики повторов при использовании EF6.

| Параметр | Значение по умолчанию | Значение |
|---------|---------------|---------|
| Политика | Экспоненциальная | Экспоненциальная задержка. |
| MaxRetryCount | 5 | Максимальное число попыток. |
| MaxDelay | 30 секунд | Максимальная задержка между повторными попытками. Это значение не влияет на то, как вычисляется значение серии задержек. Оно определяет только верхнюю границу. |
| DefaultCoefficient | 1 с | Коэффициент для вычисления значения экспоненциальной задержки. Это значение не может быть изменено. |
| DefaultRandomFactor | 1,1 | Множитель, который используется для добавления значения случайной задержки для каждой записи. Это значение не может быть изменено. |
| DefaultExponentialBase | 2 | Множитель, который используется для вычисления значения следующей задержки. Это значение не может быть изменено. |

### <a name="retry-usage-guidance"></a>Руководство по использованию механизма повторов

При доступе к базе данных SQL с помощью EF6, придерживайтесь следующих рекомендаций.

- Выберите соответствующую службу (общую или премиальную). Общий экземпляр может испытывать большие задержки подключения и регулирования чем обычно из-за использования другими клиентами общего сервера. Если требуется выполнение надежных операций с низкой задержкой и прогнозируемая производительность, попробуйте выбрать премиальный вариант.

- Стратегия фиксированного интервала не рекомендуется для использования с базой данных SQL Azure. Вместо этого используйте стратегию экспоненциальной отсрочки, так как служба может быть перегружена и большее время задержки обеспечит большее время для ее восстановления.

- Выберите подходящее значение для времени ожидания подключения и команды при определении подключения. Выберите значение времени ожидания на основе логики вашей работы и на основе испытаний. Может потребоваться изменить это значение по мере изменения объемов данных или бизнес-процессов. Слишком короткое время ожидания может привести к преждевременным сбоям подключений, когда база данных занята. Слишком длинное время ожидания может помешать правильной работе логики повторов из-за слишком долгого ожидания до обнаружения ошибки соединения. Значение времени ожидания — это компонент сквозной задержки, несмотря на это вы не можете легко определить, сколько команд будет выполнено при сохранении контекста. Время ожидания по умолчанию можно изменить, задав свойство **CommandTimeout** экземпляра **DbContext**.

- Платформа Entity Framework поддерживает конфигурации повторов, определенные в файлах конфигурации. Однако для максимальной гибкости в Azure необходимо создать конфигурацию программным способом в приложении. Определенные параметры политик повторов, например количество повторных попыток и интервалы повтора, можно сохранить в файле конфигурации службы и использовать во время выполнения для создания соответствующих политик. Это позволяет изменять параметры без перезапуска приложения.

Для начала установите следующие параметры для операций повтора. Нельзя указать задержку между попытками повтора (она является фиксированной и генерируется в виде экспоненциальной последовательности). Максимальные значения можно указать, как показано ниже, если только вы не создаете пользовательскую стратегию. Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.

| **Контекст** | **Максимальная задержка<br />примера целевого E2E** | **Политика повтора** | **Параметры** | **Значения** | **Принцип работы** |
| --- | --- | --- | --- | --- | --- |
| Интерактивный, пользовательский интерфейс<br />или передний план |2 секунды |Экспоненциальная |MaxRetryCount<br />MaxDelay |3<br />750 мс |Попытка 1 — задержка 0 с<br />Попытка 2 — задержка 750 мс<br />Попытка 3 – задержка 750 мс |
| Фоновый<br /> или пакетный |30 секунд |Экспоненциальная |MaxRetryCount<br />MaxDelay |5<br />12 секунд |Попытка 1 — задержка 0 с<br />Попытка 2 — задержка 1 с<br />Попытка 3 — задержка 3 с<br />Попытка 4 — задержка 7 с<br />Попытка 5 — задержка 12 с |

> [!NOTE]
> Целевые значения сквозной задержки подразумевают время ожидания по умолчанию для подключения к службе. При указании более длинного времени ожидания подключения величина сквозной задержки будет увеличена путем добавления этого дополнительного времени для каждой попытки повтора.

### <a name="examples"></a>Примеры

В следующем примере кода определяется простое решение доступа к данным, использующее Entity Framework. Программа определяет конкретную стратегию путем определения класса с именем экземпляра **BlogConfiguration**, который расширяет **DbConfiguration**.

```csharp
using System;
using System.Collections.Generic;
using System.Data.Entity;
using System.Data.Entity.SqlServer;
using System.Threading.Tasks;

namespace RetryCodeSamples
{
    public class BlogConfiguration : DbConfiguration
    {
        public BlogConfiguration()
        {
            // Set up the execution strategy for SQL Database (exponential) with 5 retries and 12 sec delay.
            // These values could be loaded from configuration rather than being hard-coded.
            this.SetExecutionStrategy(
                    "System.Data.SqlClient", () => new SqlAzureExecutionStrategy(5, TimeSpan.FromSeconds(12)));
        }
    }

    // Specify the configuration type if more than one has been defined.
    // [DbConfigurationType(typeof(BlogConfiguration))]
    public class BloggingContext : DbContext
    {
        // Definition of content goes here.
    }

    class EF6CodeSamples
    {
        public async static Task Samples()
        {
            // Execution strategy configured by DbConfiguration subclass, discovered automatically or
            // or explicitly indicated through configuration or with an attribute. Default is no retries.
            using (var db = new BloggingContext("Blogs"))
            {
                // Add, edit, delete blog items here, then:
                await db.SaveChangesAsync();
            }
        }
    }
}
```

Дополнительные примеры использования механизма повтора Entity Framework можно найти в разделе [Устойчивость соединения и логика повтора](/ef/ef6/fundamentals/connection-resiliency/retry-logic).

### <a name="more-information"></a>Дополнительные сведения

- [Руководство по обеспечению эластичности и производительности Базы данных SQL Azure](https://social.technet.microsoft.com/wiki/contents/articles/3507.windows-azure-sql-database-performance-and-elasticity-guide.aspx)

## <a name="sql-database-using-entity-framework-core"></a>Использование Базы данных SQL с Entity Framework Core

Механизм [Entity Framework Core](/ef/core/) отслеживает взаимоотношения между объектами и позволяет разработчикам .NET использовать объекты на уровне доменов при работе с данными. Это исключает необходимость в большинстве кодов доступа к данным, которые обычно требуется писать разработчикам. Эта версия Entity Framework полностью создана с нуля и по умолчанию не наследует возможности EF6.x.

### <a name="retry-mechanism"></a>Механизм повтора

Поддержка повторных попыток при доступе к Базе данных SQL с использованием Entity Framework Core предоставляется через механизм, называемый [устойчивость подключения](/ef/core/miscellaneous/connection-resiliency). Устойчивость подключения впервые появилась в версии EF Core 1.1.0.

Основным объектом абстракции является интерфейс `IExecutionStrategy`. Стратегия выполнения для SQL Server, включая SQL Azure, учитывает типы исключений, для которых возможны повторные попытки. Для нее предусмотрены стандартные рациональные параметры, определяющие максимальное число повторных попыток, пауз между ними и т. д.

### <a name="examples"></a>Примеры

Следующий пример кода использует автоматические повторные попытки при настройке объекта DbContext, который представляет сеанс подключения к базе данных.

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(
            @"Server=(localdb)\mssqllocaldb;Database=EFMiscellanous.ConnectionResiliency;Trusted_Connection=True;",
            options => options.EnableRetryOnFailure());
}
```

Следующий пример кода демонстрирует, как выполнять транзакцию с автоматическими повторными попытками с использованием стратегии выполнения. Транзакция определяется в делегате. Если происходит временный сбой, стратегия выполнения снова вызывает делегат.

```csharp
using (var db = new BloggingContext())
{
    var strategy = db.Database.CreateExecutionStrategy();

    strategy.Execute(() =>
    {
        using (var transaction = db.Database.BeginTransaction())
        {
            db.Blogs.Add(new Blog { Url = "https://blogs.msdn.com/dotnet" });
            db.SaveChanges();

            db.Blogs.Add(new Blog { Url = "https://blogs.msdn.com/visualstudio" });
            db.SaveChanges();

            transaction.Commit();
        }
    });
}
```

## <a name="azure-storage"></a>Хранилище Azure

Службы хранилища Azure включают хранилища таблиц и больших двоичных объектов, файлы и очереди хранилища.

### <a name="retry-mechanism"></a>Механизм повтора

Повторы выполняются на уровне отдельных операции REST и являются неотъемлемой частью реализации API клиента. Клиентский пакет SDK службы хранилища использует классы, реализующие [Интерфейс IExtendedRetryPolicy](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy).

Существуют различные реализации этого интерфейса. Клиенты службы хранилища могут выбирать политики специально для доступа к таблицам, BLOB-объектам и очередям. Каждая реализация использует различные стратегии повтора, фактически определяя интервал повторных попыток и другие параметры.

Встроенные классы обеспечивают поддержку для линейных (постоянная задержка) и экспоненциальных со случайными параметрами интервалов повтора. Здесь также нет политики повторов для случаев, когда другой процесс обрабатывает повторы на более высоком уровне. Тем не менее если имеются особые требования, не предоставляемые встроенными классами, то можно реализовать собственные классы для обработки повторов.

Альтернативные повторы переключаются между первичным и вторичным расположением службы хранилища при использовании геоизбыточного хранилища с доступом на чтение (RA-GRS) и если результатом запроса стала ошибка. Дополнительные сведения см. в статье [Варианты избыточности хранилища Azure](/azure/storage/common/storage-redundancy).

### <a name="policy-configuration"></a>Настройки политики

Политики повтора настраиваются программным способом. Типичная процедура состоит в создании и заполнении экземпляров **TableRequestOptions**, **BlobRequestOptions**, **FileRequestOptions** или **QueueRequestOptions**.

```csharp
TableRequestOptions interactiveRequestOption = new TableRequestOptions()
{
  RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
  // For Read-access geo-redundant storage, use PrimaryThenSecondary.
  // Otherwise set this to PrimaryOnly.
  LocationMode = LocationMode.PrimaryThenSecondary,
  // Maximum execution time based on the business use case.
  MaximumExecutionTime = TimeSpan.FromSeconds(2)
};
```

Затем можно задать параметры экземпляра запроса на клиенте, и все операции с клиентом будут использовать указанные параметры запроса.

```csharp
client.DefaultRequestOptions = interactiveRequestOption;
var stats = await client.GetServiceStatsAsync();
```

Можно переопределить параметры запроса клиента путем передачи заполненного экземпляра класса параметров запроса в качестве параметра метода.

```csharp
var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
```

Можно использовать экземпляр **OperationContext** , чтобы указать код, выполняемый при возникновении повтора и после завершения операции. Этот код может собирать сведения об операции для последующего использования в результатах телеметрии и журналах.

```csharp
// Set up notifications for an operation
var context = new OperationContext();
context.ClientRequestID = "some request id";
context.Retrying += (sender, args) =>
{
    /* Collect retry information */
};
context.RequestCompleted += (sender, args) =>
{
    /* Collect operation completion information */
};
var stats = await client.GetServiceStatsAsync(null, context);
```

Кроме определения, подходит ли сбой для повторных попыток, расширенные политики повтора возвращают объект **RetryContext**, который указывает число повторных попыток, результаты последнего запроса, произойдет ли следующая попытка в первичном или вторичном расположении (дополнительные сведения см. в таблице ниже). Свойства объекта **RetryContext** позволяют принять решение о необходимости и времени выполнения повтора. Дополнительные сведения см. в статье [Метод IExtendedRetryPolicy.Evaluate](/dotnet/api/microsoft.windowsazure.storage.retrypolicies.iextendedretrypolicy.evaluate).

Следующие таблицы содержат значения по умолчанию для встроенной политики повторов.

**Параметры запроса**:

| **Параметр** | **Значение по умолчанию** | **Значение** |
| --- | --- | --- |
| MaximumExecutionTime | Нет | Максимальное время выполнения запроса, включая все потенциальные повторные попытки. Если оно не указано, то количество времени на выполнение запроса будет не ограничено. Другими словами, запрос может зависнуть. |
| ServerTimeOut | Нет | Интервал времени ожидания сервера для запроса (значение округляется до секунд). Если не указан, он будет использовать значение по умолчанию для всех запросов к серверу. Как правило, лучше всего опустить этот параметр, чтобы использовать значение сервера по умолчанию. |
| LocationMode | Нет | Если учетная запись хранения создается с вариантом репликации на географически избыточном хранилище с доступом для чтения (RA-GRS), то можно использовать режим расположения, чтобы указать расположение, где запрос должен быть получен. Например, если указано **PrimaryThenSecondary**, запросы всегда будут отправляться сначала в основное расположение. Если запрос завершается ошибкой, он направляется во вторичное расположение. |
| политика RetryPolicy | ExponentialPolicy | Сведения о каждом параметре смотрите далее. |

**Экспоненциальная политика**:

| **Параметр** | **Значение по умолчанию** | **Значение** |
| --- | --- | --- |
| maxAttempt | 3 | Количество повторных попыток. |
| deltaBackoff | 4 секунды | Интервал отсрочки между повторными попытками. Кратное timespan, включая элемент случая, который будет использоваться для последующих попыток. |
| MinBackoff | 3 секунды | Добавить ко всем интервалам повтора, вычисленным по deltaBackoff. Это значение не может быть изменено.
| MaxBackoff | 120 секунд | MaxBackoff используется в том случае, если расчетный интервал повторных попыток оказывается больше, чем MaxBackoff. Это значение не может быть изменено. |

**Линейная политика**:

| **Параметр** | **Значение по умолчанию** | **Значение** |
| --- | --- | --- |
| maxAttempt | 3 | Количество повторных попыток. |
| deltaBackoff | 30 секунд | Интервал отсрочки между повторными попытками. |

### <a name="retry-usage-guidance"></a>Руководство по использованию механизма повторов

При доступе к службам хранилища Azure с помощью API клиентской службы хранилища, придерживайтесь следующих рекомендаций.

- Используйте встроенные политики повтора из пространства имен Microsoft.WindowsAzure.Storage.RetryPolicies, которые соответствуют требованиям. В большинстве случаев использования этих политик будет достаточно.

- Используйте политику **ExponentialRetry** для пакетных операций, фоновых задач или неинтерактивных сценариев. В этих сценариях обычно имеется больше времени для восстановления службы &mdash; с таким образом повышается вероятность операции.

- Следует указать свойства **MaximumExecutionTime** параметра **RequestOptions** для ограничения общего времени выполнения, а также принять во внимание тип и размер операции при выборе значения времени ожидания.

- Если необходимо реализовать пользовательский повтор, избегайте создания оболочек клиентских классов хранения. Вместо этого используйте возможности для расширения существующих политик с помощью интерфейса **IExtendedRetryPolicy** .

- Если вы используете геоизбыточное хранилище с доступом на чтение (RA-GRS), можно использовать **LocationMode** для указания, что повторные попытки должны будут выполняться для вторичной копии хранилища в случае сбоя доступа к первичному хранилищу. Однако при использовании этого параметра необходимо убедиться, что ваше приложение может успешно работать с данными, которые могут устаревать, если процедура репликации с основным хранилищем еще не завершена.

Начните с использования следующих параметров для операции повтора. Это параметры общего назначения, и вам будет необходимо отслеживать операции и проводить тонкую настройку параметров в соответствии с конкретной ситуацией.

| **Контекст** | **Максимальная задержка<br />примера целевого E2E** | **Политика повтора** | **Параметры** | **Значения** | **Принцип работы** |
| --- | --- | --- | --- | --- | --- |
| Интерактивный, пользовательский интерфейс<br />или передний план |2 секунды |Линейная |maxAttempt<br />deltaBackoff |3<br />500 мс |Попытка 1 — задержка 500 мс<br />Попытка 2 — задержка 500 мс<br />попытка 3 — задержка 500 мс |
| Фоновый<br />или пакетный |30 секунд |Экспоненциальная |maxAttempt<br />deltaBackoff |5<br />4 секунды |Попытка 1 — задержка ~ 3 с<br />Попытка 2 — задержка 7 с<br />Попытка 3 — задержка ~ 15 с |

### <a name="telemetry"></a>Телеметрия

Количество попыток входа регистрируется в **TraceSource**. Необходимо настроить **TraceListener** для сбора данных о событиях и записи этих данных в журнал с подходящим назначением. Можно использовать **TextWriterTraceListener** или **XmlWriterTraceListener** для записи данных в файл журнала, **EventLogTraceListener** для записи в журнал событий Windows или **EventProviderTraceListener** для записи данных трассировки в подсистеме трассировки событий Windows. Можно также настроить автоматическую очистку буфера и уровень детализации регистрируемых событий (например, Ошибка, Предупреждение, Информационное событие и Подробное протоколирование). Дополнительные сведения см. в статье [Вход в клиентскую библиотеку хранилища .NET на стороне клиента](/rest/api/storageservices/Client-side-Logging-with-the-.NET-Storage-Client-Library).

Операции могут получать экземпляр **OperationContext**, предоставляющий событие **Повтор**, которое может использоваться для присоединения телеметрии пользовательской логики. Дополнительные сведения см. в статье [Событие OperationContext.Retrying](/dotnet/api/microsoft.windowsazure.storage.operationcontext.retrying).

### <a name="examples"></a>Примеры

В следующем примере кода показано, как создать два экземпляра **TableRequestOptions** с различными параметрами, один для интерактивных запросов и один для фоновых запросов. Пример затем устанавливает эти две политики повторов на клиентском компьютере, чтобы они применялись для всех запросов, а также задает интерактивную стратегию на конкретный запрос, чтобы он переопределял параметры по умолчанию, применяемые к клиенту.

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.RetryPolicies;
using Microsoft.WindowsAzure.Storage.Table;

namespace RetryCodeSamples
{
    class AzureStorageCodeSamples
    {
        private const string connectionString = "UseDevelopmentStorage=true";

        public async static Task Samples()
        {
            var storageAccount = CloudStorageAccount.Parse(connectionString);

            TableRequestOptions interactiveRequestOption = new TableRequestOptions()
            {
                RetryPolicy = new LinearRetry(TimeSpan.FromMilliseconds(500), 3),
                // For Read-access geo-redundant storage, use PrimaryThenSecondary.
                // Otherwise set this to PrimaryOnly.
                LocationMode = LocationMode.PrimaryThenSecondary,
                // Maximum execution time based on the business use case.
                MaximumExecutionTime = TimeSpan.FromSeconds(2)
            };

            TableRequestOptions backgroundRequestOption = new TableRequestOptions()
            {
                // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
                // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
                MaximumExecutionTime = TimeSpan.FromSeconds(30),
                // PrimaryThenSecondary in case of Read-access geo-redundant storage, else set this to PrimaryOnly
                LocationMode = LocationMode.PrimaryThenSecondary
            };

            var client = storageAccount.CreateCloudTableClient();
            // Client has a default exponential retry policy with 4 sec delay and 3 retry attempts
            // Retry delays will be approximately 3 sec, 7 sec, and 15 sec
            // ServerTimeout and MaximumExecutionTime are not set

            {
                // Set properties for the client (used on all requests unless overridden)
                // Different exponential policy parameters for background scenarios
                client.DefaultRequestOptions = backgroundRequestOption;
                // Linear policy for interactive scenarios
                client.DefaultRequestOptions = interactiveRequestOption;
            }

            {
                // set properties for a specific request
                var stats = await client.GetServiceStatsAsync(interactiveRequestOption, operationContext: null);
            }

            {
                // Set up notifications for an operation
                var context = new OperationContext();
                context.ClientRequestID = "some request id";
                context.Retrying += (sender, args) =>
                {
                    /* Collect retry information */
                };
                context.RequestCompleted += (sender, args) =>
                {
                    /* Collect operation completion information */
                };
                var stats = await client.GetServiceStatsAsync(null, context);
            }
        }
    }
}
```

### <a name="more-information"></a>Дополнительные сведения

- [Рекомендации по настройке политики повтора для клиентской библиотеки службы хранилища Azure](https://azure.microsoft.com/blog/2014/05/22/azure-storage-client-library-retry-policy-recommendations/)

- [Клиентская библиотека службы хранилища 2.0: реализация политики повтора](https://gauravmantri.com/2012/12/30/storage-client-library-2-0-implementing-retry-policies/)

## <a name="general-rest-and-retry-guidelines"></a>Общие рекомендации по использованию REST и механизма повторов

При обращении к службам Azure или службам сторонних производителей учитывайте следующее.

- Используйте систематический подход к управлению механизмом повторов, возможно в виде повторно используемого кода, что таким образом дает возможность применения согласованной методологии ко всем клиентам и всем решениям.

- Если целевая служба или клиент не имеет встроенного механизма повторов, рекомендуется использовать платформы повторов, например [Polly][polly], для управления механизмом повторов. Это поможет реализовать согласованный механизм повтора и может предоставить подходящую стратегию повторов по умолчанию для целевой службы. Тем не менее необходимо создать пользовательский код механизма повторов для служб, имеющих нестандартное поведение, которые не полагаются на исключения для определения временных сбоев, или если вы хотите использовать ответ **Retry-Response** для управления поведением механизма повторов.

- Логика обнаружения временных сбоев будет зависеть от фактического клиентского API, который используется для вызова REST. Некоторые клиенты, например новый класс **HttpClient** , не создают исключения для запросов, завершенных с неуспешным кодом состояния HTTP.

- Код состояния HTTP, возвращаемый службой, позволяет указать, является ли ошибка временной. Необходимо изучить исключения, генерируемые клиентом, или механизм повторов для доступа к коду состояния для определения эквивалентного типа исключения. Следующие коды HTTP обычно показывают, что повтор может быть выполнен.
  
  - 408 — Истекло время ожидания запроса
  - 429 — слишком много запросов
  - 500 — Внутренняя ошибка сервера
  - 502 — Недопустимый шлюз
  - 503 — Служба недоступна
  - 504 — Истекло время ожидания шлюза

- Если логика повторов основана на исключениях, следующие параметры обычно указывают, что произошел временный сбой где не удалось создать подключение.

  - WebExceptionStatus.ConnectionClosed
  - WebExceptionStatus.ConnectFailure
  - WebExceptionStatus.Timeout
  - WebExceptionStatus.RequestCanceled

- Если служба недоступна, она может передать прогнозируемое значение задержки перед следующей повторной попыткой в заголовке ответа **Retry-After** или другом пользовательском заголовке. Службы также отправляют дополнительную информацию в виде пользовательских заголовков или внедренной в содержимое ответа.

- Не выполняйте повторы для кодов состояния, представляющих клиентские ошибки (ошибки в диапазоне 4xx) за исключением ошибки «408 — Истекло время ожидания запроса».

- Тщательно протестируйте свои стратегии и механизмы повторов для различных условий, например, указав другую сеть и с различными нагрузками системы.

### <a name="retry-strategies"></a>Стратегия повторов

Ниже приведены типичные виды интервалов стратегий повтора.

- **Экспоненциальная**. Политика повторов, основанная на выполнении указанного числа повторных попыток и использующая случайный экспоненциальный пассивный метод для определения интервала между повторными попытками. Например: 

    ```csharp
    var random = new Random();

    var delta = (int)((Math.Pow(2.0, currentRetryCount) - 1.0) *
                random.Next((int)(this.deltaBackoff.TotalMilliseconds * 0.8),
                (int)(this.deltaBackoff.TotalMilliseconds * 1.2)));
    var interval = (int)Math.Min(checked(this.minBackoff.TotalMilliseconds + delta),
                    this.maxBackoff.TotalMilliseconds);
    retryInterval = TimeSpan.FromMilliseconds(interval);
    ```

- **Добавочная**. Стратегия повторов с указанным числом повторных попыток и добавочным интервалом между повторными попытками. Например: 

    ```csharp
    retryInterval = TimeSpan.FromMilliseconds(this.initialInterval.TotalMilliseconds +
                    (this.increment.TotalMilliseconds * currentRetryCount));
    ```

- **Линейный повтор**. Политика повторов, выполняющая указанное число повторных попыток и использующая указанный фиксированный интервал времени между повторными попытками. Например: 

    ```csharp
    retryInterval = this.deltaBackoff;
    ```

### <a name="transient-fault-handling-with-polly"></a>Обработка временного сбоя с помощью Polly

[Polly] [ polly] — это библиотека программным образом обрабатывать повторные попытки и [Размыкатель цепи](../patterns/circuit-breaker.md) стратегии. Проект Polly является частью [.NET Foundation][dotnet-foundation]. Применение Polly будет неплохим вариантом при работе со службами, клиент которых не имеет встроенного механизма повторных попыток. Вам не придется самостоятельно создавать код для повторных попыток, который иногда очень трудно реализовать правильно. Polly также поддерживает трассировку возникающих ошибок, что позволяет вести журнал повторных попыток.

### <a name="more-information"></a>Дополнительные сведения

- [Устойчивость подключения](/ef/core/miscellaneous/connection-resiliency)
- [Точки данных для EF Core 1.1](https://msdn.microsoft.com/magazine/mt745093.aspx)

<!-- links -->

[adal]: /azure/active-directory/develop/active-directory-authentication-libraries
[autorest]: https://github.com/Azure/autorest/tree/master/docs
[ConnectionPolicy.RetryOptions]: https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.connectionpolicy.retryoptions.aspx
[dotnet-foundation]: https://dotnetfoundation.org/
[polly]: http://www.thepollyproject.org
[redis-cache-troubleshoot]: /azure/redis-cache/cache-how-to-troubleshoot
[SearchIndexClient]: https://msdn.microsoft.com/library/azure/microsoft.azure.search.searchindexclient.aspx
[SearchServiceClient]: https://msdn.microsoft.com/library/microsoft.azure.search.searchserviceclient.aspx
