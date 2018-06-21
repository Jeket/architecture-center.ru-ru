---
title: Внешнее хранилище конфигураций
description: Переместите сведения о конфигурации из пакета развертывания приложения в централизованное расположение.
keywords: конструктивный шаблон
author: dragon119
ms.date: 06/23/2017
pnp.series.title: Cloud Design Patterns
pnp.pattern.categories:
- design-implementation
- management-monitoring
ms.openlocfilehash: 733ca979903d1526d3a1a6b281a8903893e19fda
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
ms.locfileid: "24542286"
---
# <a name="external-configuration-store-pattern"></a>Шаблон внешнего хранилища конфигураций

[!INCLUDE [header](../_includes/header.md)]

Переместите сведения о конфигурации из пакета развертывания приложения в централизованное расположение. Это может помочь упростить управление данными конфигурации, а также обмен данными конфигурации между приложениями и экземплярами приложений.

## <a name="context-and-problem"></a>Контекст и проблема

В большинстве сред выполнения приложения содержатся сведения о конфигурации, которые хранятся в файлах, развернутых с этим приложением. В некоторых случаях можно изменить эти файлы, чтобы повлиять на поведение приложения после его развертывания. Однако изменения в конфигурации требуют повторного развертывания приложения, что часто приводит к неприемлемому простою и другим административным издержкам.

Локальные файлы конфигурации также ограничивают конфигурацию для одного приложения, но иногда могут быть полезными для совместного использования параметров конфигурации несколькими приложениями. В примерах содержатся строки подключения к базе данных, сведения о теме пользовательского интерфейса, а также URL-адреса очередей и хранилища, используемые связанным набором приложений.

Иногда сложно управлять изменениями для локальных конфигураций на нескольких запущенных экземплярах приложения, особенно при использовании сценария размещения в облаке. Это может привести к тому, что экземпляры используют другие параметры конфигурации во время развертывания обновления.

Кроме того, для обновления приложений и компонентов может потребоваться внести изменения в схемы конфигурации. Во многих системах конфигурации не поддерживаются разные версии сведений о конфигурации.

## <a name="solution"></a>Решение

Храните сведения о конфигурации во внешнем хранилище и предоставьте интерфейс, который можно использовать для быстрого и эффективного чтения и обновления параметров конфигурации. Тип внешнего хранилища зависит от среды выполнения и размещения приложения. В сценарии размещения в облаке обычно используется служба облачного хранения, но может также применяться размещенная база данных или другая система.

В выбранном для информации о конфигурации резервном хранилище должен быть простой в использовании и последовательный интерфейс. Он должен предоставлять сведения в правильно типизированном и структурированном формате. Для реализации может также потребоваться разрешить доступ пользователям, чтобы защитить данные конфигурации и обеспечить достаточную гибкость, чтобы использовать хранилище с несколькими версиями конфигурации (например, среда разработки, промежуточная среда или рабочая среда, а также несколько версий выпуска каждой из них).

> Многие встроенные системы конфигурации считывают данные при запуске приложения и выполняют кэширование данных в память для быстрого доступа к ним и минимизации влияния на производительность приложения. В зависимости от типа используемого резервного хранилища и его задержки может быть полезно реализовать механизм кэширования во внешнем хранилище конфигураций. Дополнительные сведения см. в [руководстве по кэшированию](https://msdn.microsoft.com/library/dn589802.aspx). На этом рисунке показаны общие сведения о шаблоне внешнего хранилища конфигураций с необязательным локальным кэшем.

![Общие сведения о шаблоне внешнего хранилища конфигураций с необязательным локальным кэшем](./_images/external-configuration-store-overview.png)


## <a name="issues-and-considerations"></a>Проблемы и рекомендации

При принятии решения о реализации этого шаблона необходимо учитывать следующие моменты.

Выберите резервное хранилище, обеспечивающее приемлемую производительность, высокий уровень доступности, надежность и для которого можно создать резервную копию при обслуживании и администрировании приложения. Для удовлетворения этих требований в приложении, размещенном в облаке, обычно применяется механизм облачного хранилища.

Разработайте схему резервного хранилища, чтобы обеспечивать гибкость типов хранимых данных. Убедитесь, что предусмотрены все требования конфигурации (например, введенные данные, коллекции настроек, несколько версий параметров и любые другие компоненты, требуемые для использования приложения). Схема должна легко расширяться для поддержки дополнительных параметров по мере изменения требований.

Учитывайте физические возможности резервного хранилища, как оно связано со способом хранения сведений о конфигурации, а также его влияние на производительность. Например, для хранения XML-документа, в котором содержатся сведения о конфигурации, потребуется интерфейс конфигурации или приложение для синтаксического анализа документа, чтобы выполнить чтение отдельных параметров. Это обновление параметра слегка усложнится, хотя кэширование параметров может помочь компенсировать медленную скорость чтения.

Учтите то, каким образом интерфейс конфигурации будет разрешать управлять областью и наследованием настроек конфигурации. Например, требования к настройкам конфигурации области на уровне организации, приложения и компьютера. Эти требования могут понадобиться, чтобы поддержать делегирование контроля над доступом к разным областям, а также чтобы запретить или разрешить отдельным приложениям переопределять параметры.

Убедитесь, что интерфейс конфигурации может предоставлять данные конфигурации в требуемых форматах (например, типизированные значения, коллекции, пары "ключ — значение" или контейнеры свойств).

Посмотрите, как будет работать интерфейс хранилища конфигурации, когда настройки содержат ошибки или не существуют в резервном хранилище. Возможно, потребуется восстановить параметры по умолчанию и зарегистрировать ошибки. Кроме того, обратите внимание на учет регистра в ключах или именах параметров конфигурации, хранение и обработку двоичных данных и способы обработки пустых или нулевых значений.

Рассмотрите возможность защиты данных конфигурации, чтобы предоставить доступ только соответствующим пользователям и приложениям. Это, скорее всего, возможность интерфейса хранилища конфигурации, но необходимо убедиться, что к данным в резервном хранилище невозможно получить доступ напрямую без соответствующего разрешения. Обеспечьте строгое разделение между разрешениями, необходимыми для чтения и записи данных конфигурации. Учтите также необходимость шифрования некоторых или всех параметров конфигурации и способ их реализации в интерфейсе хранилища конфигурации.

Централизованно хранимые конфигурации, которые влияют на поведение приложения при выполнении, являются критически важными, и их необходимо развернуть, обновить и управлять ими с помощью механизмов, используемых для развертывания кода приложения. Например, изменения, которые могут повлиять на несколько приложений, необходимо выполнять с использованием подхода полного тестирования и промежуточного развертывания. Это позволит убедиться, что изменение подходит для всех приложений, использующих эту конфигурацию. Если администратор изменяет параметр, чтобы обновить одно приложение, это может отрицательно повлиять на другие приложения, использующие тот же параметр.

Если приложение кэширует сведения о конфигурации, ему необходимо получать предупреждения об изменениях конфигурации. Чтобы время от времени эти сведения автоматически обновлялись и были учтены все изменения (и обрабатывались), для кэшированных данных конфигурации можно реализовать политику срока действия.

## <a name="when-to-use-this-pattern"></a>Когда следует использовать этот шаблон

Этот шаблон полезен для следующих сценариев:

- если параметры конфигурации являются общими для нескольких приложений и экземпляров приложения, или если для нескольких приложений и экземпляров приложений применяется стандартная конфигурация;

- если стандартная система конфигураций не поддерживает все необходимые параметры конфигурации (например, сохранение образов или сложных типов данных);

- если дополнительное хранилище для некоторых параметров приложений позволяет приложениям переопределять некоторые или все централизованно хранимые параметры;

- если нужно упростить администрирование нескольких приложений и при необходимости проводить мониторинг использования параметров конфигурации путем ведения журнала для некоторых или всех типов доступа к хранилищу конфигурации.

## <a name="example"></a>Пример

Для внешнего хранения информации размещаемого приложения Microsoft Azure обычно используется служба хранилища Azure. Эта служба устойчива, обеспечивает высокую производительность, предоставляет высокий уровень доступности и реплицируется три раза с помощью автоматической отработки отказа. Хранилище таблиц Azure предоставляет хранилище ключей и значений с возможностью использования гибкой схемы для значений. Хранилище BLOB-объектов Azure предоставляет иерархическое, основанное на контейнере хранилище, которое может содержать любой тип данных в отдельно именованных больших двоичных объектах.

В следующем примере показано, как хранилище конфигурации можно реализовать в хранилище BLOB-объектов для хранения и предоставления сведений о конфигурации. Класс `BlobSettingsStore` абстрагирует хранилище BLOB-объектов для хранения сведений о конфигурации, а также реализует интерфейс `ISettingsStore`, показанный в следующем коде.

> Этот код предоставляется в проекте _ExternalConfigurationStore.Cloud_ в решении _ExternalConfigurationStore_, доступном на портале [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).

```csharp
public interface ISettingsStore
{
    Task<string> GetVersionAsync();

    Task<Dictionary<string, string>> FindAllAsync();
}
```

Этот интерфейс определяет методы для получения и обновления параметров конфигурации, которые содержатся в хранилище конфигураций, и включает в себя номер версии, который можно использовать, чтобы определить, изменялись ли в последнее время параметры конфигурации. Класс `BlobSettingsStore` использует свойство `ETag` большого двоичного объекта для реализации управления версиями. Свойство `ETag` обновляется автоматически при каждой записи большого двоичного объекта.

> Исходя из структуры, это простое решение предоставляет все параметры конфигурации в виде строковых, а не типизированных значений.

Класс `ExternalConfigurationManager` предоставляет программу-оболочку для объекта `BlobSettingsStore`. Приложение может использовать этот класс для хранения и извлечения сведений о конфигурации. Этот класс использует библиотеку [реактивных расширений](https://msdn.microsoft.com/library/hh242985.aspx) Майкрософт, чтобы предоставить любые изменения, внесенные в конфигурацию, через реализацию интерфейса `IObservable`. Если параметр изменяется путем вызова метода `SetAppSetting`, об этом будут уведомлены событие `Changed` и все подписчики этого события.

Обратите внимание, что для быстрого получения доступа все параметры также кэшируются в объекте `Dictionary` в классе `ExternalConfigurationManager`. Метод `GetSetting`, используемый для получения параметров конфигурации, считывает данные из кэша. Если параметр не найден в кэше, он извлекается из объекта `BlobSettingsStore`.

Метод `GetSettings` вызывает метод `CheckForConfigurationChanges`, чтобы узнать, изменились ли сведения о конфигурации в хранилище BLOB-объектов. Это делается путем проверки и сравнения номера версии с текущим номером версии, хранимым в объекте `ExternalConfigurationManager`. Если имеется одно или несколько изменений, возникает событие `Changed` и обновляются параметры конфигурации, кэшированные в объекте `Dictionary`. Это приложение шаблона [Кэш на стороне](cache-aside.md).

В следующем примере кода показано, как реализуются событие `Changed`, метод `GetSettings` и `CheckForConfigurationChanges`.

```csharp
public class ExternalConfigurationManager : IDisposable
{
  // An abstraction of the configuration store.
  private readonly ISettingsStore settings;
  private readonly ISubject<KeyValuePair<string, string>> changed;
  ...
  private readonly ReaderWriterLockSlim settingsCacheLock = new ReaderWriterLockSlim();
  private readonly SemaphoreSlim syncCacheSemaphore = new SemaphoreSlim(1);  
  ...
  private Dictionary<string, string> settingsCache;
  private string currentVersion;
  ...
  public ExternalConfigurationManager(ISettingsStore settings, ...)
  {
    this.settings = settings;
    ...
  }
  ...
  public IObservable<KeyValuePair<string, string>> Changed => this.changed.AsObservable();
  ...

  public string GetAppSetting(string key)
  {
    ...
    // Try to get the value from the settings cache. 
    // If there's a cache miss, get the setting from the settings store and refresh the settings cache.

    string value;
    try
    {
        this.settingsCacheLock.EnterReadLock();

        this.settingsCache.TryGetValue(key, out value);
    }
    finally
    {
        this.settingsCacheLock.ExitReadLock();
    }

    return value;
  }
  ...
  private void CheckForConfigurationChanges()
  {
    try
    {
        // It is assumed that updates are infrequent.
        // To avoid race conditions in refreshing the cache, synchronize access to the in-memory cache.
        await this.syncCacheSemaphore.WaitAsync();

        var latestVersion = await this.settings.GetVersionAsync();

        // If the versions are the same, nothing has changed in the configuration.
        if (this.currentVersion == latestVersion) return;

        // Get the latest settings from the settings store and publish changes.
        var latestSettings = await this.settings.FindAllAsync();

        // Refresh the settings cache.
        try
        {
            this.settingsCacheLock.EnterWriteLock();

            if (this.settingsCache != null)
            {
                //Notify settings changed
                latestSettings.Except(this.settingsCache).ToList().ForEach(kv => this.changed.OnNext(kv));
            }
            this.settingsCache = latestSettings;
        }
        finally
        {
            this.settingsCacheLock.ExitWriteLock();
        }

        // Update the current version.
        this.currentVersion = latestVersion;
    }
    catch (Exception ex)
    {
        this.changed.OnError(ex);
    }
    finally
    {
        this.syncCacheSemaphore.Release();
    }
  }
}
```

> Класс `ExternalConfigurationManager` также предоставляет свойство с именем `Environment`. Это свойство поддерживает различные конфигурации для приложения, работающего в разных средах (например, в промежуточной и рабочей).

Объект `ExternalConfigurationManager` может также периодически выполнять запрос по изменениям объекта `BlobSettingsStore`. В следующем коде метод `StartMonitor` вызывает `CheckForConfigurationChanges` с интервалом, чтобы обнаружить все изменения, и инициирует событие `Changed`, как описано выше.

```csharp
public class ExternalConfigurationManager : IDisposable
{
  ...
  private readonly ISubject<KeyValuePair<string, string>> changed;
  private Dictionary<string, string> settingsCache;
  private readonly CancellationTokenSource cts = new CancellationTokenSource();
  private Task monitoringTask;
  private readonly TimeSpan interval;

  private readonly SemaphoreSlim timerSemaphore = new SemaphoreSlim(1);
  ...
  public ExternalConfigurationManager(string environment) : this(new BlobSettingsStore(environment), TimeSpan.FromSeconds(15), environment)
  {
  }
  
  public ExternalConfigurationManager(ISettingsStore settings, TimeSpan interval, string environment)
  {
      this.settings = settings;
      this.interval = interval;
      this.CheckForConfigurationChangesAsync().Wait();
      this.changed = new Subject<KeyValuePair<string, string>>();
      this.Environment = environment;
  }
  ...
  /// <summary>
  /// Check to see if the current instance is monitoring for changes
  /// </summary>
  public bool IsMonitoring => this.monitoringTask != null && !this.monitoringTask.IsCompleted;

  /// <summary>
  /// Start the background monitoring for configuration changes in the central store
  /// </summary>
  public void StartMonitor()
  {
      if (this.IsMonitoring)
          return;

      try
      {
          this.timerSemaphore.Wait();

          // Check again to make sure we are not already running.
          if (this.IsMonitoring)
              return;

          // Start running our task loop.
          this.monitoringTask = ConfigChangeMonitor();
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  /// <summary>
  /// Loop that monitors for configuration changes
  /// </summary>
  /// <returns></returns>
  public async Task ConfigChangeMonitor()
  {
      while (!cts.Token.IsCancellationRequested)
      {
          await this.CheckForConfigurationChangesAsync();
          await Task.Delay(this.interval, cts.Token);
      }
  }

  /// <summary>
  /// Stop monitoring for configuration changes
  /// </summary>
  public void StopMonitor()
  {
      try
      {
          this.timerSemaphore.Wait();

          // Signal the task to stop.
          this.cts.Cancel();

          // Wait for the loop to stop.
          this.monitoringTask.Wait();

          this.monitoringTask = null;
      }
      finally
      {
          this.timerSemaphore.Release();
      }
  }

  public void Dispose()
  {
      this.cts.Cancel();
  }
  ...
}
```

Класс `ExternalConfigurationManager` создан в качестве одноэлементного экземпляра с помощью класса `ExternalConfiguration`, показанного ниже.

```csharp
public static class ExternalConfiguration
{
    private static readonly Lazy<ExternalConfigurationManager> configuredInstance = new Lazy<ExternalConfigurationManager>(
        () =>
        {
            var environment = CloudConfigurationManager.GetSetting("environment");
            return new ExternalConfigurationManager(environment);
        });

    public static ExternalConfigurationManager Instance => configuredInstance.Value;
}
```

Приведенный ниже код взят из класса `WorkerRole` в проекте _ExternalConfigurationStore.Cloud_. В нем показано, как приложение использует класс `ExternalConfiguration` для чтения параметра.

```csharp
public override void Run()
{
  // Start monitoring configuration changes.
  ExternalConfiguration.Instance.StartMonitor();

  // Get a setting.
  var setting = ExternalConfiguration.Instance.GetAppSetting("setting1");
  Trace.TraceInformation("Worker Role: Get setting1, value: " + setting);

  this.completeEvent.WaitOne();
}
```

Следующий код также взят из класса `WorkerRole`, и в нем показывается, как приложение подписывается на события конфигурации.

```csharp
public override bool OnStart()
{
  ...
  // Subscribe to the event.
  ExternalConfiguration.Instance.Changed.Subscribe(
     m => Trace.TraceInformation("Configuration has changed. Key:{0} Value:{1}",
          m.Key, m.Value),
     ex => Trace.TraceError("Error detected: " + ex.Message));
  ...
}
```

## <a name="related-patterns-and-guidance"></a>Связанные шаблоны и рекомендации

- Пример, демонстрирующий этот шаблон, можно найти на [GitHub](https://github.com/mspnp/cloud-design-patterns/tree/master/external-configuration-store).
