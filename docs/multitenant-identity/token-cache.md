---
title: "Кэширование маркеров доступа в мультитенантном приложении"
description: "Кэширование маркеров доступа, используемых для вызова серверного веб-API"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: web-api
pnp.series.next: adfs
ms.openlocfilehash: cffc15686ef9d77fafb40982efdbcd4a79f5aaf2
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/08/2017
---
# <a name="cache-access-tokens"></a><span data-ttu-id="5b809-103">Кэширование маркеров доступа</span><span class="sxs-lookup"><span data-stu-id="5b809-103">Cache access tokens</span></span>

<span data-ttu-id="5b809-104">[![GitHub](../_images/github.png) Пример кода][sample application]</span><span class="sxs-lookup"><span data-stu-id="5b809-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="5b809-105">Получение маркера доступа OAuth является относительно ресурсоемким процессом, поскольку для этого требуется отправить HTTP-запрос в конечную точку маркера.</span><span class="sxs-lookup"><span data-stu-id="5b809-105">It's relatively expensive to get an OAuth access token, because it requires an HTTP request to the token endpoint.</span></span> <span data-ttu-id="5b809-106">Поэтому рекомендуется по возможности кэшировать маркеры.</span><span class="sxs-lookup"><span data-stu-id="5b809-106">Therefore, it's good to cache tokens whenever possible.</span></span> <span data-ttu-id="5b809-107">[Библиотека аутентификации Azure AD] [ ADAL] (ADAL) автоматически кэширует маркеры, полученные из Azure AD, в том числе маркеры обновления.</span><span class="sxs-lookup"><span data-stu-id="5b809-107">The [Azure AD Authentication Library][ADAL] (ADAL)  automatically caches tokens obtained from Azure AD, including refresh tokens.</span></span>

<span data-ttu-id="5b809-108">ADAL обеспечивает реализацию кэша маркеров по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="5b809-108">ADAL provides a default token cache implementation.</span></span> <span data-ttu-id="5b809-109">Однако этот кэш маркеров предназначен только для собственных клиентских приложений, и **не подходит** для веб-приложений по следующим причинам:</span><span class="sxs-lookup"><span data-stu-id="5b809-109">However, this token cache is intended for native client apps, and is **not** suitable for web apps:</span></span>

* <span data-ttu-id="5b809-110">это статический экземпляр, который не является потокобезопасным;</span><span class="sxs-lookup"><span data-stu-id="5b809-110">It is a static instance, and not thread safe.</span></span>
* <span data-ttu-id="5b809-111">он не масштабируется для большого количества пользователей, так как маркеры всех пользователей попадают в один словарь;</span><span class="sxs-lookup"><span data-stu-id="5b809-111">It doesn't scale to large numbers of users, because tokens from all users go into the same dictionary.</span></span>
* <span data-ttu-id="5b809-112">он не может быть общим для веб-серверов в ферме.</span><span class="sxs-lookup"><span data-stu-id="5b809-112">It can't be shared across web servers in a farm.</span></span>

<span data-ttu-id="5b809-113">Вместо этого вам нужно реализовать собственный кэш маркеров, производный от класса ADAL `TokenCache`. Этот кэш должен соответствовать серверной среде и обеспечивать нужный уровень изоляции между маркерами разных пользователей.</span><span class="sxs-lookup"><span data-stu-id="5b809-113">Instead, you should implement a custom token cache that derives from the ADAL `TokenCache` class but is suitable for a server environment and provides the desirable level of isolation between tokens for different users.</span></span>

<span data-ttu-id="5b809-114">Класс `TokenCache` хранит словарь маркеров, индексированных по издателю, ресурсу, пользователю и идентификатору клиента.</span><span class="sxs-lookup"><span data-stu-id="5b809-114">The `TokenCache` class stores a dictionary of tokens, indexed by issuer, resource, client ID, and user.</span></span> <span data-ttu-id="5b809-115">Настраиваемый кэш маркеров должен записывать этот словарь в резервное хранилище, например кэш Redis.</span><span class="sxs-lookup"><span data-stu-id="5b809-115">A custom token cache should write this dictionary to a backing store, such as a Redis cache.</span></span>

<span data-ttu-id="5b809-116">В приложении Tailspin кэш маркеров реализуется классом `DistributedTokenCache` .</span><span class="sxs-lookup"><span data-stu-id="5b809-116">In the Tailspin Surveys application, the `DistributedTokenCache` class implements the token cache.</span></span> <span data-ttu-id="5b809-117">Эта реализация использует абстракцию [IDistributedCache][distributed-cache] из ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="5b809-117">This implementation uses the [IDistributedCache][distributed-cache] abstraction from ASP.NET Core.</span></span> <span data-ttu-id="5b809-118">Таким образом, резервным хранилищем может быть любая реализация `IDistributedCache` .</span><span class="sxs-lookup"><span data-stu-id="5b809-118">That way, any `IDistributedCache` implementation can be used as a backing store.</span></span>

* <span data-ttu-id="5b809-119">По умолчанию в приложении Surveys используется кэш Redis.</span><span class="sxs-lookup"><span data-stu-id="5b809-119">By default, the Surveys app uses a Redis cache.</span></span>
* <span data-ttu-id="5b809-120">Для веб-сервера с одним экземпляром можно использовать [кэш в памяти][in-memory-cache] ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="5b809-120">For a single-instance web server, you could use the ASP.NET Core [in-memory cache][in-memory-cache].</span></span> <span data-ttu-id="5b809-121">(Этот вариант также подходит для локального запуска приложения во время разработки.)</span><span class="sxs-lookup"><span data-stu-id="5b809-121">(This is also a good option for running the app locally during development.)</span></span>

<span data-ttu-id="5b809-122">`DistributedTokenCache` хранит данные кэша как пары "ключ-значение" в резервном хранилище.</span><span class="sxs-lookup"><span data-stu-id="5b809-122">`DistributedTokenCache` stores the cache data as key/value pairs in the backing store.</span></span> <span data-ttu-id="5b809-123">Ключом является идентификатор пользователя и идентификатор клиента, поэтому в резервном хранилище имеются отдельные данные кэша для каждого уникального сочетания пользователя и клиента.</span><span class="sxs-lookup"><span data-stu-id="5b809-123">The key is the user ID plus client ID, so the backing store holds separate cache data for each unique combination of user/client.</span></span>

![Кэш маркеров](./images/token-cache.png)

<span data-ttu-id="5b809-125">Резервное хранилище секционируется по пользователям.</span><span class="sxs-lookup"><span data-stu-id="5b809-125">The backing store is partitioned by user.</span></span> <span data-ttu-id="5b809-126">Для каждого HTTP-запроса маркеры, относящиеся к конкретному пользователю, считываются из резервного хранилища и сохраняются в словаре `TokenCache`.</span><span class="sxs-lookup"><span data-stu-id="5b809-126">For each HTTP request, the tokens for that user are read from the backing store and loaded into the `TokenCache` dictionary.</span></span> <span data-ttu-id="5b809-127">Если в качестве резервного хранилища используется Redis, все экземпляры каждого сервера в ферме используют для чтения и записи один и тот же кэш, что позволяет масштабировать систему на большое количество пользователей.</span><span class="sxs-lookup"><span data-stu-id="5b809-127">If Redis is used as the backing store, every server instance in a server farm reads/writes to the same cache, and this approach scales to many users.</span></span>

## <a name="encrypting-cached-tokens"></a><span data-ttu-id="5b809-128">Шифрование кэшированных маркеров</span><span class="sxs-lookup"><span data-stu-id="5b809-128">Encrypting cached tokens</span></span>
<span data-ttu-id="5b809-129">Маркеры считаются конфиденциальными данными, поскольку они предоставляют доступ к ресурсам пользователя.</span><span class="sxs-lookup"><span data-stu-id="5b809-129">Tokens are sensitive data, because they grant access to a user's resources.</span></span> <span data-ttu-id="5b809-130">Кроме того, в отличие от пароля пользователя, их нельзя хранить в виде хэша. Поэтому крайне важно защитить их от компрометации.</span><span class="sxs-lookup"><span data-stu-id="5b809-130">(Moreover, unlike a user's password, you can't just store a hash of the token.) Therefore, it's critical to protect tokens from being compromised.</span></span> <span data-ttu-id="5b809-131">Кэш на основе Redis защищен паролем, но любой обладатель пароля может получить все кэшированные маркеры доступа.</span><span class="sxs-lookup"><span data-stu-id="5b809-131">The Redis-backed cache is protected by a password, but if someone obtains the password, they could get all of the cached access tokens.</span></span> <span data-ttu-id="5b809-132">По этой причине `DistributedTokenCache` шифрует все данные, которые он записывает в резервное хранилище.</span><span class="sxs-lookup"><span data-stu-id="5b809-132">For that reason, the `DistributedTokenCache` encrypts everything that it writes to the backing store.</span></span> <span data-ttu-id="5b809-133">Шифрование выполняется с помощью API-интерфейсов [защиты данных][data-protection], которые предоставляет ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="5b809-133">Encryption is done using the ASP.NET Core [data protection][data-protection] APIs.</span></span>

> [!NOTE]
> <span data-ttu-id="5b809-134">Если приложение развернуто на веб-сайтах Azure, ключи шифрования сохраняются в сетевое хранилище и синхронизируются на всех компьютерах. См. статью об [управлении ключами и времени существования ключей][key-management].</span><span class="sxs-lookup"><span data-stu-id="5b809-134">If you deploy to Azure Web Sites, the encryption keys are backed up to network storage and synchronized across all machines (see [Key management and lifetime][key-management]).</span></span> <span data-ttu-id="5b809-135">По умолчанию ключи не шифруются при использовании в службе веб-сайтов Azure, но вы можете [включить шифрование с использованием сертификата X.509][x509-cert-encryption].</span><span class="sxs-lookup"><span data-stu-id="5b809-135">By default, keys are not encrypted when running in Azure Web Sites, but you can [enable encryption using an X.509 certificate][x509-cert-encryption].</span></span>
> 
> 

## <a name="distributedtokencache-implementation"></a><span data-ttu-id="5b809-136">Реализация DistributedTokenCache</span><span class="sxs-lookup"><span data-stu-id="5b809-136">DistributedTokenCache implementation</span></span>
<span data-ttu-id="5b809-137">Класс `DistributedTokenCache` является производным от класса ADAL [TokenCache][tokencache-class].</span><span class="sxs-lookup"><span data-stu-id="5b809-137">The `DistributedTokenCache` class derives from the ADAL [TokenCache][tokencache-class] class.</span></span>

<span data-ttu-id="5b809-138">В конструкторе класс `DistributedTokenCache` создает ключ для текущего пользователя и загружает кэш из резервного хранилища.</span><span class="sxs-lookup"><span data-stu-id="5b809-138">In the constructor, the `DistributedTokenCache` class creates a key for the current user and loads the cache from the backing store:</span></span>

```csharp
public DistributedTokenCache(
    ClaimsPrincipal claimsPrincipal,
    IDistributedCache distributedCache,
    ILoggerFactory loggerFactory,
    IDataProtectionProvider dataProtectionProvider)
    : base()
{
    _claimsPrincipal = claimsPrincipal;
    _cacheKey = BuildCacheKey(_claimsPrincipal);
    _distributedCache = distributedCache;
    _logger = loggerFactory.CreateLogger<DistributedTokenCache>();
    _protector = dataProtectionProvider.CreateProtector(typeof(DistributedTokenCache).FullName);
    AfterAccess = AfterAccessNotification;
    LoadFromCache();
}
```

<span data-ttu-id="5b809-139">Ключ создается путем объединения идентификатора пользователя и идентификатора клиента.</span><span class="sxs-lookup"><span data-stu-id="5b809-139">The key is created by concatenating the user ID and client ID.</span></span> <span data-ttu-id="5b809-140">Оба идентификатора берутся из утверждений в `ClaimsPrincipal`пользователя.</span><span class="sxs-lookup"><span data-stu-id="5b809-140">Both of these are taken from claims found in the user's `ClaimsPrincipal`:</span></span>

```csharp
private static string BuildCacheKey(ClaimsPrincipal claimsPrincipal)
{
    string clientId = claimsPrincipal.FindFirstValue("aud", true);
    return string.Format(
        "UserId:{0}::ClientId:{1}",
        claimsPrincipal.GetObjectIdentifierValue(),
        clientId);
}
```

<span data-ttu-id="5b809-141">Чтобы загрузить данные кэша, прочтите сериализованный BLOB-объект из резервного хранилища и вызовите метод `TokenCache.Deserialize` для преобразования BLOB-объекта в данные кэша.</span><span class="sxs-lookup"><span data-stu-id="5b809-141">To load the cache data, read the serialized blob from the backing store, and call `TokenCache.Deserialize` to convert the blob into cache data.</span></span>

```csharp
private void LoadFromCache()
{
    byte[] cacheData = _distributedCache.Get(_cacheKey);
    if (cacheData != null)
    {
        this.Deserialize(_protector.Unprotect(cacheData));
    }
}
```

<span data-ttu-id="5b809-142">При каждом доступе ADAL к кэшу возникает событие `AfterAccess` .</span><span class="sxs-lookup"><span data-stu-id="5b809-142">Whenever ADAL access the cache, it fires an `AfterAccess` event.</span></span> <span data-ttu-id="5b809-143">Если данные кэша изменились, свойство `HasStateChanged` получает значение "true".</span><span class="sxs-lookup"><span data-stu-id="5b809-143">If the cache data has changed, the `HasStateChanged` property is true.</span></span> <span data-ttu-id="5b809-144">В этом случае обновите резервное хранилище в соответствии с изменениями, а затем задайте свойству `HasStateChanged` значение "false".</span><span class="sxs-lookup"><span data-stu-id="5b809-144">In that case, update the backing store to reflect the change, and then set `HasStateChanged` to false.</span></span>

```csharp
public void AfterAccessNotification(TokenCacheNotificationArgs args)
{
    if (this.HasStateChanged)
    {
        try
        {
            if (this.Count > 0)
            {
                _distributedCache.Set(_cacheKey, _protector.Protect(this.Serialize()));
            }
            else
            {
                // There are no tokens for this user/client, so remove the item from the cache.
                _distributedCache.Remove(_cacheKey);
            }
            this.HasStateChanged = false;
        }
        catch (Exception exp)
        {
            _logger.WriteToCacheFailed(exp);
            throw;
        }
    }
}
```

<span data-ttu-id="5b809-145">TokenCache отправляет два следующих события:</span><span class="sxs-lookup"><span data-stu-id="5b809-145">TokenCache sends two other events:</span></span>

* <span data-ttu-id="5b809-146">`BeforeWrite`.</span><span class="sxs-lookup"><span data-stu-id="5b809-146">`BeforeWrite`.</span></span> <span data-ttu-id="5b809-147">Вызывается непосредственно перед тем, как ADAL выполнит запись в кэш.</span><span class="sxs-lookup"><span data-stu-id="5b809-147">Called immediately before ADAL writes to the cache.</span></span> <span data-ttu-id="5b809-148">Его можно использовать для реализации стратегии параллелизма.</span><span class="sxs-lookup"><span data-stu-id="5b809-148">You can use this to implement a concurrency strategy</span></span>
* <span data-ttu-id="5b809-149">`BeforeAccess`.</span><span class="sxs-lookup"><span data-stu-id="5b809-149">`BeforeAccess`.</span></span> <span data-ttu-id="5b809-150">Вызывается непосредственно перед тем, как ADAL выполнит чтение из кэша.</span><span class="sxs-lookup"><span data-stu-id="5b809-150">Called immediately before ADAL reads from the cache.</span></span> <span data-ttu-id="5b809-151">В этом случае можно перезагрузить кэш, чтобы получить его последнюю версию.</span><span class="sxs-lookup"><span data-stu-id="5b809-151">Here you can reload the cache to get the latest version.</span></span>

<span data-ttu-id="5b809-152">Мы решили не обрабатывать эти два события.</span><span class="sxs-lookup"><span data-stu-id="5b809-152">In our case, we decided not to handle these two events.</span></span>

* <span data-ttu-id="5b809-153">В случае параллелизма приоритет имеет последняя запись.</span><span class="sxs-lookup"><span data-stu-id="5b809-153">For concurrency, last write wins.</span></span> <span data-ttu-id="5b809-154">Это нормально, поскольку маркеры хранятся независимо для каждого пользователя и клиента, поэтому конфликт может произойти только в случае, если один пользователь открыл два сеанса входа одновременно.</span><span class="sxs-lookup"><span data-stu-id="5b809-154">That's OK, because tokens are stored independently for each user + client, so a conflict would only happen if the same user had two concurrent login sessions.</span></span>
* <span data-ttu-id="5b809-155">Для операции чтения кэш будет загружаться при каждом запросе.</span><span class="sxs-lookup"><span data-stu-id="5b809-155">For reading, we load the cache on every request.</span></span> <span data-ttu-id="5b809-156">Запросы являются кратковременными.</span><span class="sxs-lookup"><span data-stu-id="5b809-156">Requests are short lived.</span></span> <span data-ttu-id="5b809-157">Если в это время кэш будет изменен, следующий запрос получит новое значение.</span><span class="sxs-lookup"><span data-stu-id="5b809-157">If the cache gets modified in that time, the next request will pick up the new value.</span></span>

<span data-ttu-id="5b809-158">[**Далее**][client-assertion]</span><span class="sxs-lookup"><span data-stu-id="5b809-158">[**Next**][client-assertion]</span></span>

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[client-assertion]: ./client-assertion.md
[data-protection]: /aspnet/core/security/data-protection/
[distributed-cache]: /aspnet/core/performance/caching/distributed
[key-management]: /aspnet/core/security/data-protection/configuration/default-settings
[in-memory-cache]: /aspnet/core/performance/caching/memory
[tokencache-class]: https://msdn.microsoft.com/library/azure/microsoft.identitymodel.clients.activedirectory.tokencache.aspx
[x509-cert-encryption]: /aspnet/core/security/data-protection/implementation/key-encryption-at-rest#x509-certificate
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
