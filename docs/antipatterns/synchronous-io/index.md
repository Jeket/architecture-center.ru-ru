---
title: "Антишаблон синхронных операций ввода-вывода"
description: "Блокировка вызывающего потока до завершения операций ввода-вывода может привести к снижению производительности. Кроме того, она влияет на вертикальное масштабирование."
author: dragon119
ms.date: 06/05/2017
ms.openlocfilehash: d5b3635565c6b71ef7716f54ee8cccc76093c3a3
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
---
# <a name="synchronous-io-antipattern"></a>Антишаблон синхронных операций ввода-вывода

Блокировка вызывающего потока до завершения операций ввода-вывода может привести к снижению производительности. Кроме того, она влияет на вертикальное масштабирование.

## <a name="problem-description"></a>Описание проблемы

Синхронные операции ввода-вывода блокируют вызывающий поток, пока не завершится операция ввода-вывода. Вызывающий поток переходит в состояние ожидания и не может выполнить нужные действия во время этого интервала, растрачивая ресурсы обработки.

Ниже приведены распространенные примеры операций ввода-вывода:

- Получение или сохранение данных в базу данных или любое постоянное хранилище.
- Отправка запроса к веб-службе.
- Публикация сообщения или получение сообщения из очереди.
- Запись в локальный файл или считывание данных из него.

Этот антишаблон обычно возникает в следующих случаях:

- Это самый простой способ выполнения операции. 
- Приложению требуется ответ на запрос.
- Приложение использует библиотеку, которая предоставляет только синхронные методы для операций ввода-вывода. 
- Внешняя библиотека выполняет синхронные операции ввода-вывода изнутри. Один вызов синхронных операций ввода-вывода может блокировать всю цепочку вызовов.

Приведенный ниже код передает файл в хранилище BLOB-объектов Azure. Существует два места, где код выполняет блокировку, ожидая выполнения синхронных операций ввода-вывода: метод `CreateIfNotExists` и `UploadFromStream`.

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

container.CreateIfNotExists();
var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    blockBlob.UploadFromStream(fileStream);
}
```

Ниже приведен пример ожидания ответа от внешней службы. Метод `GetUserProfile` вызывает удаленную службу, которая возвращает `UserProfile`.

```csharp
public interface IUserProfileService
{
    UserProfile GetUserProfile();
}

public class SyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public SyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is a synchronous method that calls the synchronous GetUserProfile method.
    public UserProfile GetUserProfile()
    {
        return _userProfileService.GetUserProfile();
    }
}
```

Полный код для этих примеров можно найти [здесь][sample-app].

## <a name="how-to-fix-the-problem"></a>Как устранить проблему

Замените синхронные операции ввода-вывода асинхронными. Они не заблокируют текущий поток, а освободят его для продолжения выполнения работы, а также улучшат использование вычислительных ресурсов. Выполнение асинхронных операций ввода-вывода особенно эффективно для обработки непредвиденного всплеска запросов от клиентских приложений. 

Многие библиотеки предоставляют синхронные и асинхронные версии методов. По возможности используйте асинхронные. Ниже приведена асинхронная версия предыдущего примера, которая отправляет файл в хранилище BLOB-объектов Azure.

```csharp
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("uploadedfiles");

await container.CreateIfNotExistsAsync();

var blockBlob = container.GetBlockBlobReference("myblob");

// Create or overwrite the "myblob" blob with contents from a local file.
using (var fileStream = File.OpenRead(HostingEnvironment.MapPath("~/FileToUpload.txt")))
{
    await blockBlob.UploadFromStreamAsync(fileStream);
}
```

Во время выполнения асинхронной операции оператор `await` возвращает управление вызывающей среде. После этого код выступает в качестве кода продолжения, который будет выполняться по завершении асинхронной операции.

Хорошо спроектированные службы также должны предоставлять асинхронные операции. Ниже приведен пример асинхронной версии веб-службы, которая возвращает профили пользователей. Метод `GetUserProfileAsync` зависит от наличия асинхронной версии службы профилей пользователей.

```csharp
public interface IUserProfileService
{
    Task<UserProfile> GetUserProfileAsync();
}

public class AsyncController : ApiController
{
    private readonly IUserProfileService _userProfileService;

    public AsyncController()
    {
        _userProfileService = new FakeUserProfileService();
    }

    // This is an synchronous method that calls the Task based GetUserProfileAsync method.
    public Task<UserProfile> GetUserProfileAsync()
    {
        return _userProfileService.GetUserProfileAsync();
    }
}
```

Библиотеки, которые не предоставляют асинхронные версии операций, могут создать асинхронные программы-оболочки вокруг выбранных синхронных методов. Используйте этот подход с осторожностью. Он потребляет больше ресурсов, хотя и может повысить скорость реагирования на поток, который вызывает асинхронную программу-оболочку. Вы можете создать дополнительный поток, однако с синхронизацией работы, выполняемой этим потоком, связаны определенные затраты. В следующей записи блога приведены некоторые компромиссы: [Should I expose asynchronous wrappers for synchronous methods?][async-wrappers] (Следует ли предоставлять асинхронные программы-оболочки для синхронных методов?)

Ниже приведен пример асинхронной программы-оболочки, применяемой к синхронному методу.

```csharp
// Asynchronous wrapper around synchronous library method
private async Task<int> LibraryIOOperationAsync()
{
    return await Task.Run(() => LibraryIOOperation());
}
```

Теперь вызывающий код может ожидать в программе-оболочке:

```csharp
// Invoke the asynchronous wrapper using a task
await LibraryIOOperationAsync();
```

## <a name="considerations"></a>Рекомендации

- Операции ввода-вывода, которые длятся недолго и вряд ли будут вызывать конфликты, могут быть более производительными, чем синхронные операции. Примером может быть чтение небольших файлов на SSD-диске. Дополнительные издержки, связанные с отправкой задачи в другой поток и синхронизацией с этим потоком после завершения задачи, могут перевесить преимущества асинхронного ввода-вывода. Однако такое случается редко, и большинство операций ввода-вывода должны выполняться асинхронно.

- Повышение производительности операций ввода-вывода может стать причиной возникновения узких мест в системе. Например, разблокирование потоков может привести к увеличению объема одновременных запросов к общим ресурсам. В результате возникнет нехватка ресурсов или необходимость регулирования. Если это становится проблемой, нужно увеличить количество веб-серверов или выполнить секционирование хранилищ данных, чтобы уменьшить количество конфликтов.

## <a name="how-to-detect-the-problem"></a>Как определить проблему

Приложение может не отвечать или периодически зависать. Может произойти сбой приложения с исключением времени ожидания. Эти сбои также могут возвращать ошибки HTTP 500 (внутренняя ошибка сервера). На сервере входящие запросы клиентов могут блокироваться до тех пор, пока поток не станет доступным. В результате существенно увеличится очередь запросов и отобразится ошибка HTTP 503 (служба недоступна).

Чтобы определить проблему, сделайте следующее:

1. Выполните мониторинг рабочей системы и определите, ограничивают ли заблокированные рабочие потоки пропускную способность.

2. Если запросы блокируются из-за отсутствия потоков, проверьте приложение, чтобы определить, какие операции выполняют ввод-вывод синхронно.

3. Выполните управляемое нагрузочное тестирование каждой операции, которая выполняет синхронный ввод-вывод, чтобы узнать, влияют ли они на производительность системы.

## <a name="example-diagnosis"></a>Пример диагностики

В следующих разделах эти шаги применяются к примеру приложения, описанному ранее.

### <a name="monitor-web-server-performance"></a>Мониторинг производительности веб-сервера

Мониторинг производительности веб-сервера IIS нужно выполнять для веб-ролей и веб-приложений Azure. В частности, обратите внимание на длину очередей запросов, чтобы узнать, блокируются ли запросы в ожидании доступных потоков во время периодов высокой активности. Эти данные можно собирать, включив систему диагностики Azure. Дополнительные сведения можно найти в разделе 

- [Мониторинг приложений в службе приложений Azure][web-sites-monitor].
- [Создание и использование счетчиков производительности в приложении Azure][performance-counters].

Инструментируйте приложение, чтобы узнать, как обрабатываются принятые запросы. Трассировка последовательности запроса может помочь определить, выполняются ли медленные вызовы и блокируют ли он текущий поток. Профилирование потока также может помочь выделить заблокированные запросы.

### <a name="load-test-the-application"></a>Нагрузочное тестирование приложения

На следующем графике показана производительность синхронного метода `GetUserProfile` (описанного ранее) при различных нагрузках до 4000 одновременных пользователей. В этом примере показано приложение ASP.NET, работающее в веб-роли облачной службы Azure.

![Диаграмма производительности для примера приложения, выполняющего синхронные операции ввода-вывода][sync-performance]

Для синхронной операции жестко задан спящий режим на 2 секунды, чтобы имитировать синхронный ввод-вывод, поэтому минимальное время отклика будет немного превышать 2 секунды. Если нагрузка достигает приблизительно 2500 одновременных пользователей, среднее время отклика достигает пика, несмотря на то что объем запросов продолжает увеличиваться каждую секунду. Обратите внимание, что эти величины измеряются по логарифмической шкале. Количество запросов в секунду увеличивается вдвое между этой точкой и временем завершения теста.

В результате этого изолированного теста непонятно, является ли синхронный ввод-вывод проблемой. При более высокой нагрузке приложение может достичь переломного момента, когда веб-сервер больше не может обрабатывать запросы своевременно. В результате в клиентских приложениях произойдет сбой с исключением времени ожидания.

Входящие запросы помещаются в очередь веб-сервером IIS и передаются в поток, который выполняется в пуле потоков ASP.NET. Поток блокируется до завершения операции, так как каждая операция выполняет ввод-вывод синхронно. При увеличении рабочей нагрузки, в конечном итоге, все потоки ASP.NET в пуле потоков выделяются и блокируются. На этом этапе любые дополнительные входящие запросы должны ожидать в очереди, пока не освободится поток. При увеличении длины очереди время ожидания запросов начинает истекать.

### <a name="implement-the-solution-and-verify-the-result"></a>Реализация решения и проверка результатов

На следующем графике показаны результаты нагрузочных тестов асинхронной версии кода.

![Диаграмма производительности для примера приложения, выполняющего асинхронные операции ввода-вывода][async-performance]

Пропускная способность намного выше. На протяжении того же промежутка времени, что и в предыдущем тесте, система успешно обрабатывает почти десятикратное увеличение пропускной способности, измеренной по количеству запросов в секунду. Кроме того, среднее время отклика является относительной константой и примерно в 25 раз меньше, чем в предыдущем тесте.


[sample-app]: https://github.com/mspnp/performance-optimization/tree/master/SynchronousIO


[async-wrappers]: http://blogs.msdn.com/b/pfxteam/archive/2012/03/24/10287244.aspx
[performance-counters]: /azure/cloud-services/cloud-services-dotnet-diagnostics-performance-counters
[web-sites-monitor]: /azure/app-service-web/web-sites-monitor

[sync-performance]: _images/SyncPerformance.jpg
[async-performance]: _images/AsyncPerformance.jpg


