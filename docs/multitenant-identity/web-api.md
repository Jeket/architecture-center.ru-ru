---
title: Защита серверного веб-API в мультитенантном приложении
description: Как защитить веб-API серверной части.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: authorize
pnp.series.next: token-cache
ms.openlocfilehash: fd0ac254604470ba51ea00537490cfb22b224e80
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640181"
---
# <a name="secure-a-backend-web-api"></a><span data-ttu-id="6630a-103">Защита серверного веб-API</span><span class="sxs-lookup"><span data-stu-id="6630a-103">Secure a backend web API</span></span>

<span data-ttu-id="6630a-104">[![GitHub](../_images/github.png) Пример кода][sample application]</span><span class="sxs-lookup"><span data-stu-id="6630a-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="6630a-105">Приложение [Tailspin Surveys] использует серверный веб-API для управления операциями CRUD в опросах.</span><span class="sxs-lookup"><span data-stu-id="6630a-105">The [Tailspin Surveys] application uses a backend web API to manage CRUD operations on surveys.</span></span> <span data-ttu-id="6630a-106">Например, когда пользователь щелкает "Мои опросы", веб-приложение отправляет HTTP-запрос в веб-API.</span><span class="sxs-lookup"><span data-stu-id="6630a-106">For example, when a user clicks "My Surveys", the web application sends an HTTP request to the web API:</span></span>

```http
GET /users/{userId}/surveys
```

<span data-ttu-id="6630a-107">Веб-API возвращает объект JSON.</span><span class="sxs-lookup"><span data-stu-id="6630a-107">The web API returns a JSON object:</span></span>

```http
{
  "Published":[],
  "Own":[
    {"Id":1,"Title":"Survey 1"},
    {"Id":3,"Title":"Survey 3"},
    ],
  "Contribute": [{"Id":8,"Title":"My survey"}]
}
```

<span data-ttu-id="6630a-108">Веб-интерфейс API не допускает выполнение анонимных запросов, поэтому веб-приложение должно пройти проверку подлинности с помощью токенов носителя OAuth 2.</span><span class="sxs-lookup"><span data-stu-id="6630a-108">The web API does not allow anonymous requests, so the web app must authenticate itself using OAuth 2 bearer tokens.</span></span>

> [!NOTE]
> <span data-ttu-id="6630a-109">Это сценарий взаимодействия между серверами.</span><span class="sxs-lookup"><span data-stu-id="6630a-109">This is a server-to-server scenario.</span></span> <span data-ttu-id="6630a-110">Приложение не осуществляет AJAX-вызовы API из клиента браузера.</span><span class="sxs-lookup"><span data-stu-id="6630a-110">The application does not make any AJAX calls to the API from the browser client.</span></span>

<span data-ttu-id="6630a-111">Доступны два основных подхода:</span><span class="sxs-lookup"><span data-stu-id="6630a-111">There are two main approaches you can take:</span></span>

* <span data-ttu-id="6630a-112">Удостоверение делегированного пользователя.</span><span class="sxs-lookup"><span data-stu-id="6630a-112">Delegated user identity.</span></span> <span data-ttu-id="6630a-113">Веб-приложение выполняет проверку подлинности с помощью удостоверения пользователя.</span><span class="sxs-lookup"><span data-stu-id="6630a-113">The web application authenticates with the user's identity.</span></span>
* <span data-ttu-id="6630a-114">Удостоверение приложения.</span><span class="sxs-lookup"><span data-stu-id="6630a-114">Application identity.</span></span> <span data-ttu-id="6630a-115">Веб-приложение выполняет проверку подлинности с помощью идентификатора клиента, используя поток учетных данных клиента OAuth 2.</span><span class="sxs-lookup"><span data-stu-id="6630a-115">The web application authenticates with its client ID, using OAuth 2 client credential flow.</span></span>

<span data-ttu-id="6630a-116">Приложение Tailspin реализует удостоверение делегированного пользователя.</span><span class="sxs-lookup"><span data-stu-id="6630a-116">The Tailspin application implements delegated user identity.</span></span> <span data-ttu-id="6630a-117">Их основные различия приведены ниже.</span><span class="sxs-lookup"><span data-stu-id="6630a-117">Here are the main differences:</span></span>

<span data-ttu-id="6630a-118">**Удостоверение делегированного пользователя**.</span><span class="sxs-lookup"><span data-stu-id="6630a-118">**Delegated user identity:**</span></span>

* <span data-ttu-id="6630a-119">Токен носителя, отправляемый в веб-API, содержит удостоверение пользователя.</span><span class="sxs-lookup"><span data-stu-id="6630a-119">The bearer token sent to the web API contains the user identity.</span></span>
* <span data-ttu-id="6630a-120">Веб-API принимает решения об авторизации на основе удостоверения пользователя.</span><span class="sxs-lookup"><span data-stu-id="6630a-120">The web API makes authorization decisions based on the user identity.</span></span>
* <span data-ttu-id="6630a-121">Веб-приложение должно обрабатывать ошибки 403 ("Запрещено") из веб-API, если пользователь не авторизован для выполнения действия.</span><span class="sxs-lookup"><span data-stu-id="6630a-121">The web application needs to handle 403 (Forbidden) errors from the web API, if the user is not authorized to perform an action.</span></span>
* <span data-ttu-id="6630a-122">Как правило, веб-приложение все еще принимает некоторые решения относительно авторизации, затрагивающие пользовательский интерфейс (например отображение или скрытие элементов пользовательского интерфейса).</span><span class="sxs-lookup"><span data-stu-id="6630a-122">Typically, the web application still makes some authorization decisions that affect UI, such as showing or hiding UI elements).</span></span>
* <span data-ttu-id="6630a-123">Веб-API может использоваться ненадежными клиентами, такими как приложение JavaScript или классическое клиентское приложение.</span><span class="sxs-lookup"><span data-stu-id="6630a-123">The web API can potentially be used by untrusted clients, such as a JavaScript application or a native client application.</span></span>

<span data-ttu-id="6630a-124">**Удостоверение приложения**.</span><span class="sxs-lookup"><span data-stu-id="6630a-124">**Application identity:**</span></span>

* <span data-ttu-id="6630a-125">Веб-API не получает сведения о пользователе.</span><span class="sxs-lookup"><span data-stu-id="6630a-125">The web API does not get information about the user.</span></span>
* <span data-ttu-id="6630a-126">Веб-API не выполняет авторизацию на основе удостоверения пользователя.</span><span class="sxs-lookup"><span data-stu-id="6630a-126">The web API cannot perform any authorization based on the user identity.</span></span> <span data-ttu-id="6630a-127">Все решения об авторизации принимает веб-приложение.</span><span class="sxs-lookup"><span data-stu-id="6630a-127">All authorization decisions are made by the web application.</span></span>  
* <span data-ttu-id="6630a-128">Веб-API нельзя использовать в ненадежном клиенте (в приложении JavaScript или собственном клиентском приложении).</span><span class="sxs-lookup"><span data-stu-id="6630a-128">The web API cannot be used by an untrusted client (JavaScript or native client application).</span></span>
* <span data-ttu-id="6630a-129">Этот подход проще реализовать, так как в веб-API отсутствует логика авторизации.</span><span class="sxs-lookup"><span data-stu-id="6630a-129">This approach may be somewhat simpler to implement, because there is no authorization logic in the Web API.</span></span>

<span data-ttu-id="6630a-130">Независимо от подхода веб-приложение должно получить маркер доступа, представляющий учетные данные, необходимые для вызова веб-API.</span><span class="sxs-lookup"><span data-stu-id="6630a-130">In either approach, the web application must get an access token, which is the credential needed to call the web API.</span></span>

* <span data-ttu-id="6630a-131">В случае с удостоверением делегированного пользователя маркер должен поступать от поставщика удостоверений, который выдает маркер от имени пользователя.</span><span class="sxs-lookup"><span data-stu-id="6630a-131">For delegated user identity, the token has to come from the IDP, which can issue a token on behalf of the user.</span></span>
* <span data-ttu-id="6630a-132">Приложение может использовать в качестве учетных данных маркеры, получаемые от IDP, или разместить собственный сервер маркеров.</span><span class="sxs-lookup"><span data-stu-id="6630a-132">For client credentials, an application might get the token from the IDP or host its own token server.</span></span> <span data-ttu-id="6630a-133">Но нет необходимости писать сервер маркеров с нуля, можно применить уже проверенные платформы, например [IdentityServer4]. Если аутентификация выполняется через Azure AD, настоятельно рекомендуем получать маркер доступа из Azure AD, даже если вы реализуете собственный поток учетных данных.</span><span class="sxs-lookup"><span data-stu-id="6630a-133">(But don't write a token server from scratch; use a well-tested framework like [IdentityServer4].) If you authenticate with Azure AD, it's strongly recommended to get the access token from Azure AD, even with client credential flow.</span></span>

<span data-ttu-id="6630a-134">В оставшейся части этой статьи предполагается, что приложение проходит проверку подлинности в Azure AD.</span><span class="sxs-lookup"><span data-stu-id="6630a-134">The rest of this article assumes the application is authenticating with Azure AD.</span></span>

![Получение маркера доступа](./images/access-token.png)

## <a name="register-the-web-api-in-azure-ad"></a><span data-ttu-id="6630a-136">Регистрация веб-API в Azure AD</span><span class="sxs-lookup"><span data-stu-id="6630a-136">Register the web API in Azure AD</span></span>

<span data-ttu-id="6630a-137">Чтобы получить токен носителя для веб-API из Azure AD, необходимо выполнять ряд настроек в Azure AD.</span><span class="sxs-lookup"><span data-stu-id="6630a-137">In order for Azure AD to issue a bearer token for the web API, you need to configure some things in Azure AD.</span></span>

1. <span data-ttu-id="6630a-138">Зарегистрируйте веб-API в Azure AD.</span><span class="sxs-lookup"><span data-stu-id="6630a-138">Register the web API in Azure AD.</span></span>

2. <span data-ttu-id="6630a-139">Укажите идентификатор клиента для веб-приложения в свойстве `knownClientApplications` манифеста приложения веб-API.</span><span class="sxs-lookup"><span data-stu-id="6630a-139">Add the client ID of the web app to the web API application manifest, in the `knownClientApplications` property.</span></span> <span data-ttu-id="6630a-140">См. раздел об [обновлении манифестов приложения].</span><span class="sxs-lookup"><span data-stu-id="6630a-140">See [Update the application manifests].</span></span>

3. <span data-ttu-id="6630a-141">Предоставьте веб-приложению разрешение вызывать веб-API.</span><span class="sxs-lookup"><span data-stu-id="6630a-141">Give the web application permission to call the web API.</span></span> <span data-ttu-id="6630a-142">На портале управления Azure можно задать два типа разрешений: "Разрешения приложения" для удостоверения приложения (поток учетных данных клиента) или "Делегированные разрешения" для удостоверения делегированного пользователя.</span><span class="sxs-lookup"><span data-stu-id="6630a-142">In the Azure Management Portal, you can set two types of permissions: "Application Permissions" for application identity (client credential flow), or "Delegated Permissions" for delegated user identity.</span></span>

   ![Делегированные разрешения](./images/delegated-permissions.png)

## <a name="getting-an-access-token"></a><span data-ttu-id="6630a-144">Получение маркера доступа</span><span class="sxs-lookup"><span data-stu-id="6630a-144">Getting an access token</span></span>

<span data-ttu-id="6630a-145">Перед вызовом веб-API веб-приложение получает маркер доступа из Azure AD.</span><span class="sxs-lookup"><span data-stu-id="6630a-145">Before calling the web API, the web application gets an access token from Azure AD.</span></span> <span data-ttu-id="6630a-146">В приложении .NET используйте [Библиотеку аутентификации Azure AD (ADAL) для .NET][ADAL].</span><span class="sxs-lookup"><span data-stu-id="6630a-146">In a .NET application, use the [Azure AD Authentication Library (ADAL) for .NET][ADAL].</span></span>

<span data-ttu-id="6630a-147">В потоке кода авторизации OAuth 2 приложение обменивается кодом авторизации для маркера доступа.</span><span class="sxs-lookup"><span data-stu-id="6630a-147">In the OAuth 2 authorization code flow, the application exchanges an authorization code for an access token.</span></span> <span data-ttu-id="6630a-148">Для получения маркера доступа приведенный ниже код использует ADAL.</span><span class="sxs-lookup"><span data-stu-id="6630a-148">The following code uses ADAL to get the access token.</span></span> <span data-ttu-id="6630a-149">Этот код вызывается во время события `AuthorizationCodeReceived` .</span><span class="sxs-lookup"><span data-stu-id="6630a-149">This code is called during the `AuthorizationCodeReceived` event.</span></span>

```csharp
// The OpenID Connect middleware sends this event when it gets the authorization code.
public override async Task AuthorizationCodeReceived(AuthorizationCodeReceivedContext context)
{
    string authorizationCode = context.ProtocolMessage.Code;
    string authority = "https://login.microsoftonline.com/" + tenantID
    string resourceID = "https://tailspin.onmicrosoft.com/surveys.webapi" // App ID URI
    ClientCredential credential = new ClientCredential(clientId, clientSecret);

    AuthenticationContext authContext = new AuthenticationContext(authority, tokenCache);
    AuthenticationResult authResult = await authContext.AcquireTokenByAuthorizationCodeAsync(
        authorizationCode, new Uri(redirectUri), credential, resourceID);

    // If successful, the token is in authResult.AccessToken
}
```

<span data-ttu-id="6630a-150">Ниже указаны различные необходимые параметры.</span><span class="sxs-lookup"><span data-stu-id="6630a-150">Here are the various parameters that are needed:</span></span>

* <span data-ttu-id="6630a-151">`authority`.</span><span class="sxs-lookup"><span data-stu-id="6630a-151">`authority`.</span></span> <span data-ttu-id="6630a-152">Вычисляется по идентификатору клиента для пользователя, выполнившего вход.</span><span class="sxs-lookup"><span data-stu-id="6630a-152">Derived from the tenant ID of the signed in user.</span></span> <span data-ttu-id="6630a-153">Идентификатор клиента поставщика службы SaaS не используется.</span><span class="sxs-lookup"><span data-stu-id="6630a-153">(Not the tenant ID of the SaaS provider)</span></span>  
* <span data-ttu-id="6630a-154">`authorizationCode`.</span><span class="sxs-lookup"><span data-stu-id="6630a-154">`authorizationCode`.</span></span> <span data-ttu-id="6630a-155">Код проверки подлинности, полученный от поставщика удостоверений.</span><span class="sxs-lookup"><span data-stu-id="6630a-155">the auth code that you got back from the IDP.</span></span>
* <span data-ttu-id="6630a-156">`clientId`.</span><span class="sxs-lookup"><span data-stu-id="6630a-156">`clientId`.</span></span> <span data-ttu-id="6630a-157">Идентификатор клиента веб-приложения.</span><span class="sxs-lookup"><span data-stu-id="6630a-157">The web application's client ID.</span></span>
* <span data-ttu-id="6630a-158">`clientSecret`.</span><span class="sxs-lookup"><span data-stu-id="6630a-158">`clientSecret`.</span></span> <span data-ttu-id="6630a-159">Секрет клиента веб-приложения.</span><span class="sxs-lookup"><span data-stu-id="6630a-159">The web application's client secret.</span></span>
* <span data-ttu-id="6630a-160">`redirectUri`.</span><span class="sxs-lookup"><span data-stu-id="6630a-160">`redirectUri`.</span></span> <span data-ttu-id="6630a-161">URI, заданный для OpenID Connect перенаправления.</span><span class="sxs-lookup"><span data-stu-id="6630a-161">The redirect URI that you set for OpenID Connect.</span></span> <span data-ttu-id="6630a-162">Здесь поставщик удостоверений выполняет обратный вызов маркера.</span><span class="sxs-lookup"><span data-stu-id="6630a-162">This is where the IDP calls back with the token.</span></span>
* <span data-ttu-id="6630a-163">`resourceID`.</span><span class="sxs-lookup"><span data-stu-id="6630a-163">`resourceID`.</span></span> <span data-ttu-id="6630a-164">URI идентификатора приложения для веб-API, который был создан при регистрации веб-API в Azure AD.</span><span class="sxs-lookup"><span data-stu-id="6630a-164">The App ID URI of the web API, which you created when you registered the web API in Azure AD</span></span>
* <span data-ttu-id="6630a-165">`tokenCache`.</span><span class="sxs-lookup"><span data-stu-id="6630a-165">`tokenCache`.</span></span> <span data-ttu-id="6630a-166">Объект, который кэширует маркеры доступа.</span><span class="sxs-lookup"><span data-stu-id="6630a-166">An object that caches the access tokens.</span></span> <span data-ttu-id="6630a-167">См. статью о [кэшировании маркеров].</span><span class="sxs-lookup"><span data-stu-id="6630a-167">See [Token caching].</span></span>

<span data-ttu-id="6630a-168">Если `AcquireTokenByAuthorizationCodeAsync` завершается успешно, ADAL кэширует маркер.</span><span class="sxs-lookup"><span data-stu-id="6630a-168">If `AcquireTokenByAuthorizationCodeAsync` succeeds, ADAL caches the token.</span></span> <span data-ttu-id="6630a-169">Чтобы получить маркер из кэша позднее, вызовите AcquireTokenSilentAsync:</span><span class="sxs-lookup"><span data-stu-id="6630a-169">Later, you can get the token from the cache by calling AcquireTokenSilentAsync:</span></span>

```csharp
AuthenticationContext authContext = new AuthenticationContext(authority, tokenCache);
var result = await authContext.AcquireTokenSilentAsync(resourceID, credential, new UserIdentifier(userId, UserIdentifierType.UniqueId));
```

<span data-ttu-id="6630a-170">где `userId` обозначает идентификатор объекта пользователя, который указан в утверждении`http://schemas.microsoft.com/identity/claims/objectidentifier`.</span><span class="sxs-lookup"><span data-stu-id="6630a-170">where `userId` is the user's object ID, which is found in the `http://schemas.microsoft.com/identity/claims/objectidentifier` claim.</span></span>

## <a name="using-the-access-token-to-call-the-web-api"></a><span data-ttu-id="6630a-171">Использование маркера доступа для вызова веб-API</span><span class="sxs-lookup"><span data-stu-id="6630a-171">Using the access token to call the web API</span></span>

<span data-ttu-id="6630a-172">Получив маркер, отправьте его в веб-API в заголовке авторизации HTTP-запроса.</span><span class="sxs-lookup"><span data-stu-id="6630a-172">Once you have the token, send it in the Authorization header of the HTTP requests to the web API.</span></span>

```http
Authorization: Bearer xxxxxxxxxx
```

<span data-ttu-id="6630a-173">Приведенный далее метод расширения из приложения Surveys задает заголовок авторизации в HTTP-запросе с помощью класса **HttpClient** .</span><span class="sxs-lookup"><span data-stu-id="6630a-173">The following extension method from the Surveys application sets the Authorization header on an HTTP request, using the **HttpClient** class.</span></span>

```csharp
public static async Task<HttpResponseMessage> SendRequestWithBearerTokenAsync(this HttpClient httpClient, HttpMethod method, string path, object requestBody, string accessToken, CancellationToken ct)
{
    var request = new HttpRequestMessage(method, path);
    if (requestBody != null)
    {
        var json = JsonConvert.SerializeObject(requestBody, Formatting.None);
        var content = new StringContent(json, Encoding.UTF8, "application/json");
        request.Content = content;
    }

    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
    request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));

    var response = await httpClient.SendAsync(request, ct);
    return response;
}
```

## <a name="authenticating-in-the-web-api"></a><span data-ttu-id="6630a-174">Проверка подлинности в веб-API</span><span class="sxs-lookup"><span data-stu-id="6630a-174">Authenticating in the web API</span></span>

<span data-ttu-id="6630a-175">Веб-API должен проверить подлинность токена носителя.</span><span class="sxs-lookup"><span data-stu-id="6630a-175">The web API has to authenticate the bearer token.</span></span> <span data-ttu-id="6630a-176">В ASP.NET Core вы можете использовать пакет [Microsoft.AspNet.Authentication.JwtBearer][JwtBearer].</span><span class="sxs-lookup"><span data-stu-id="6630a-176">In ASP.NET Core, you can use the [Microsoft.AspNet.Authentication.JwtBearer][JwtBearer] package.</span></span> <span data-ttu-id="6630a-177">Этот пакет содержит ПО промежуточного слоя, который позволяет приложению получать токены носителя OpenID Connect.</span><span class="sxs-lookup"><span data-stu-id="6630a-177">This package provides middleware that enables the application to receive OpenID Connect bearer tokens.</span></span>

<span data-ttu-id="6630a-178">Зарегистрируйте ПО промежуточного слоя в классе `Startup` веб-API.</span><span class="sxs-lookup"><span data-stu-id="6630a-178">Register the middleware in your web API `Startup` class.</span></span>

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ApplicationDbContext dbContext, ILoggerFactory loggerFactory)
{
    // ...

    app.UseJwtBearerAuthentication(new JwtBearerOptions {
        Audience = configOptions.AzureAd.WebApiResourceId,
        Authority = Constants.AuthEndpointPrefix,
        TokenValidationParameters = new TokenValidationParameters {
            ValidateIssuer = false
        },
        Events= new SurveysJwtBearerEvents(loggerFactory.CreateLogger<SurveysJwtBearerEvents>())
    });

    // ...
}
```

* <span data-ttu-id="6630a-179">**Audience**.</span><span class="sxs-lookup"><span data-stu-id="6630a-179">**Audience**.</span></span> <span data-ttu-id="6630a-180">Задайте в качестве значения URL-адрес идентификатора приложения для веб-API, который был создан при регистрации веб-API в Azure AD.</span><span class="sxs-lookup"><span data-stu-id="6630a-180">Set this to the App ID URL for the web API, which you created when you registered the web API with Azure AD.</span></span>
* <span data-ttu-id="6630a-181">**Authority**.</span><span class="sxs-lookup"><span data-stu-id="6630a-181">**Authority**.</span></span> <span data-ttu-id="6630a-182">Для мультитенантного приложения установите значение `https://login.microsoftonline.com/common/`.</span><span class="sxs-lookup"><span data-stu-id="6630a-182">For a multitenant application, set this to `https://login.microsoftonline.com/common/`.</span></span>
* <span data-ttu-id="6630a-183">**TokenValidationParameters**.</span><span class="sxs-lookup"><span data-stu-id="6630a-183">**TokenValidationParameters**.</span></span> <span data-ttu-id="6630a-184">Для мультитенантного приложения установите значение false для параметра **ValidateIssuer**.</span><span class="sxs-lookup"><span data-stu-id="6630a-184">For a multitenant application, set **ValidateIssuer** to false.</span></span> <span data-ttu-id="6630a-185">Это означает, что приложение будет самостоятельно проверять издателя.</span><span class="sxs-lookup"><span data-stu-id="6630a-185">That means the application will validate the issuer.</span></span>
* <span data-ttu-id="6630a-186">Класс **Events** наследуется от **JwtBearerEvents**.</span><span class="sxs-lookup"><span data-stu-id="6630a-186">**Events** is a class that derives from **JwtBearerEvents**.</span></span>

### <a name="issuer-validation"></a><span data-ttu-id="6630a-187">Проверка издателя</span><span class="sxs-lookup"><span data-stu-id="6630a-187">Issuer validation</span></span>

<span data-ttu-id="6630a-188">Издатель маркера проверяется в событии **JwtBearerEvents.TokenValidated**.</span><span class="sxs-lookup"><span data-stu-id="6630a-188">Validate the token issuer in the **JwtBearerEvents.TokenValidated** event.</span></span> <span data-ttu-id="6630a-189">Издатель отправляется в утверждении "iss".</span><span class="sxs-lookup"><span data-stu-id="6630a-189">The issuer is sent in the "iss" claim.</span></span>

<span data-ttu-id="6630a-190">В приложении Surveys веб-API не обрабатывает [Регистрация клиента].</span><span class="sxs-lookup"><span data-stu-id="6630a-190">In the Surveys application, the web API doesn't handle [tenant sign-up].</span></span> <span data-ttu-id="6630a-191">Таким образом, он только проверяет, находится ли издатель в базе данных приложения.</span><span class="sxs-lookup"><span data-stu-id="6630a-191">Therefore, it just checks if the issuer is already in the application database.</span></span> <span data-ttu-id="6630a-192">Если издатель отсутствует, создается исключение, которое приводит к ошибке проверки подлинности.</span><span class="sxs-lookup"><span data-stu-id="6630a-192">If not, it throws an exception, which causes authentication to fail.</span></span>

```csharp
public override async Task TokenValidated(TokenValidatedContext context)
{
    var principal = context.Ticket.Principal;
    var tenantManager = context.HttpContext.RequestServices.GetService<TenantManager>();
    var userManager = context.HttpContext.RequestServices.GetService<UserManager>();
    var issuerValue = principal.GetIssuerValue();
    var tenant = await tenantManager.FindByIssuerValueAsync(issuerValue);

    if (tenant == null)
    {
        // The caller was not from a trusted issuer. Throw to block the authentication flow.
        throw new SecurityTokenValidationException();
    }

    var identity = principal.Identities.First();

    // Add new claim for survey_userid
    var registeredUser = await userManager.FindByObjectIdentifier(principal.GetObjectIdentifierValue());
    identity.AddClaim(new Claim(SurveyClaimTypes.SurveyUserIdClaimType, registeredUser.Id.ToString()));
    identity.AddClaim(new Claim(SurveyClaimTypes.SurveyTenantIdClaimType, registeredUser.TenantId.ToString()));

    // Add new claim for Email
    var email = principal.FindFirst(ClaimTypes.Upn)?.Value;
    if (!string.IsNullOrWhiteSpace(email))
    {
        identity.AddClaim(new Claim(ClaimTypes.Email, email));
    }
}
```

<span data-ttu-id="6630a-193">Как показано в этом примере, вы можете изменять утверждения с помощью события **TokenValidated**.</span><span class="sxs-lookup"><span data-stu-id="6630a-193">As this example shows, you can also use the **TokenValidated** event to modify the claims.</span></span> <span data-ttu-id="6630a-194">Как вы помните, эти утверждения поступают из Azure AD.</span><span class="sxs-lookup"><span data-stu-id="6630a-194">Remember that the claims come directly from Azure AD.</span></span> <span data-ttu-id="6630a-195">Если веб-приложение изменяет полученные утверждения, эти изменения не влияют на маркер носителя, который передается в веб-API.</span><span class="sxs-lookup"><span data-stu-id="6630a-195">If the web application modifies the claims that it gets, those changes won't show up in the bearer token that the web API receives.</span></span> <span data-ttu-id="6630a-196">Дополнительные сведения см. в разделе о [преобразовании утверждений][claims-transformation].</span><span class="sxs-lookup"><span data-stu-id="6630a-196">For more information, see [Claims transformations][claims-transformation].</span></span>

## <a name="authorization"></a><span data-ttu-id="6630a-197">Авторизация</span><span class="sxs-lookup"><span data-stu-id="6630a-197">Authorization</span></span>

<span data-ttu-id="6630a-198">Общие сведения об авторизации см. в статье об [авторизации на основе ролей и ресурсов][Authorization].</span><span class="sxs-lookup"><span data-stu-id="6630a-198">For a general discussion of authorization, see [Role-based and resource-based authorization][Authorization].</span></span>

<span data-ttu-id="6630a-199">ПО промежуточного слоя JwtBearer обрабатывает ответы на запросы авторизации.</span><span class="sxs-lookup"><span data-stu-id="6630a-199">The JwtBearer middleware handles the authorization responses.</span></span> <span data-ttu-id="6630a-200">Например, чтобы ограничить действие контроллера пользователям, прошедшим проверку, используйте **[Authorize]** и укажите **JwtBearerDefaults.AuthenticationScheme** схемы проверки подлинности:</span><span class="sxs-lookup"><span data-stu-id="6630a-200">For example, to restrict a controller action to authenticated users, use the **[Authorize]** attribute and specify **JwtBearerDefaults.AuthenticationScheme** as the authentication scheme:</span></span>

```csharp
[Authorize(ActiveAuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)]
```

<span data-ttu-id="6630a-201">Возвращает код состояния 401, если пользователь не прошел проверку подлинности.</span><span class="sxs-lookup"><span data-stu-id="6630a-201">This returns a 401 status code if the user is not authenticated.</span></span>

<span data-ttu-id="6630a-202">Чтобы ограничить действие контроллера политикой авторизации, укажите имя политики в атрибуте **[Authorize]**:</span><span class="sxs-lookup"><span data-stu-id="6630a-202">To restrict a controller action by authorizaton policy, specify the policy name in the **[Authorize]** attribute:</span></span>

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
```

<span data-ttu-id="6630a-203">Возвращает код состояния 401, если пользователь не прошел проверку подлинности, и 403, если пользователь прошел проверку подлинности, но не авторизован.</span><span class="sxs-lookup"><span data-stu-id="6630a-203">This returns a 401 status code if the user is not authenticated, and 403 if the user is authenticated but not authorized.</span></span> <span data-ttu-id="6630a-204">Зарегистрируйте политику при запуске:</span><span class="sxs-lookup"><span data-stu-id="6630a-204">Register the policy on startup:</span></span>

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthorization(options =>
    {
        options.AddPolicy(PolicyNames.RequireSurveyCreator,
            policy =>
            {
                policy.AddRequirements(new SurveyCreatorRequirement());
                policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement
                policy.AddAuthenticationSchemes(JwtBearerDefaults.AuthenticationScheme);
            });
        options.AddPolicy(PolicyNames.RequireSurveyAdmin,
            policy =>
            {
                policy.AddRequirements(new SurveyAdminRequirement());
                policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement
                policy.AddAuthenticationSchemes(JwtBearerDefaults.AuthenticationScheme);
            });
    });

    // ...
}
```

<span data-ttu-id="6630a-205">[**Далее**][token cache]</span><span class="sxs-lookup"><span data-stu-id="6630a-205">[**Next**][token cache]</span></span>

<!-- links -->
[ADAL]: https://msdn.microsoft.com/library/azure/jj573266.aspx
[JwtBearer]: https://www.nuget.org/packages/Microsoft.AspNet.Authentication.JwtBearer

[Tailspin Surveys]: tailspin.md
[IdentityServer4]: https://github.com/IdentityServer/IdentityServer4
[обновлении манифестов приложения]: ./run-the-app.md#update-the-application-manifests
[Update the application manifests]: ./run-the-app.md#update-the-application-manifests
[кэшировании маркеров]: token-cache.md
[Token caching]: token-cache.md
[Регистрация клиента]: signup.md
[tenant sign-up]: signup.md
[claims-transformation]: claims.md#claims-transformations
[Authorization]: authorize.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[token cache]: token-cache.md
