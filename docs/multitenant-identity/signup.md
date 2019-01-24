---
title: Регистрация и адаптация клиентов в мультитенантных приложениях
description: Подключение клиентов в мультитенантном приложении.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: claims
pnp.series.next: app-roles
ms.openlocfilehash: a1ec441b731ba7f2166f9115452b052ec944444f
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488415"
---
# <a name="tenant-sign-up-and-onboarding"></a>Регистрация и адаптации клиента

[![GitHub](../_images/github.png) Пример кода][sample application]

В этой статье описывается, как реализовать процесс *регистрации* в мультитенантном приложении, чтобы клиент смог зарегистрировать свою организацию для работы с приложением.
Существует несколько причин для реализации процесса регистрации:

* Предоставление администраторам AD возможности давать согласие на использование приложения во всей организации клиента.
* Сбор данных об оплате кредитной картой или других сведений о клиенте.
* Выполнение однократной настройки каждого клиента, необходимой приложению.

## <a name="admin-consent-and-azure-ad-permissions"></a>Согласие администратора и разрешения Azure AD

Чтобы проверять подлинность в Azure AD, приложению требуется доступ к каталогу пользователя. Как минимум, приложению требуется разрешение на чтение профиля пользователя. При первом входе пользователя в Azure AD будет отображена страница согласия со списком запрашиваемых разрешений. Нажав кнопку **Принять**, пользователь предоставляет разрешение приложению.

По умолчанию согласие предоставляется на уровне пользователя. Страницу согласия видит каждый пользователь, выполняющий вход. Однако Azure AD также поддерживает *согласие администратора*, позволяющее администраторам AD давать согласие на использование приложения в рамках всей организации.

Когда используется поток согласия администратора, на странице согласия отображается сообщение о том, что администратор AD предоставляет разрешение от имени всего клиента.

![Запрос на согласие администратора](./images/admin-consent.png)

После того как администратор нажмет кнопку **Принять**, другие пользователи в рамках того же клиента смогут выполнять вход, а Azure AD будет пропускать экран согласия.

Предоставлять согласие администратора может только администратор AD, поскольку он дает разрешение от имени всей организации. Если проверку подлинности в потоке согласия пытается пройти пользователь без прав администратора, в Azure AD выводится сообщение об ошибке.

![Ошибка согласия](./images/consent-error.png)

Если приложению позднее потребуются дополнительные разрешения, клиенту будет нужно повторно зарегистрироваться и согласиться с обновленными разрешениями.

## <a name="implementing-tenant-sign-up"></a>Реализация регистрации клиента

Мы определили несколько требований для регистрации в приложении [Tailspin Surveys][Tailspin]:

* Клиент должен пройти регистрацию до того, как пользователи смогут выполнять вход.
* В процессе регистрации используется поток согласия администратора.
* При регистрации клиент пользователя добавляется в базу данных приложения.
* После регистрации клиента в приложении отображается страница адаптации.

В этом разделе мы рассмотрим реализацию процесса регистрации.
Важно понимать, что понятия "регистрация" и "вход" формируют концепцию приложения. Во время потока проверки подлинности службе Azure AD изначально неизвестно, что пользователь находится в процессе регистрации. Отслеживанием контекста занимается приложение.

Когда анонимный пользователь открывает страницу приложения Surveys, он видит две кнопки — одну для входа и одну для регистрации организации (регистрация).

![Страница регистрации в приложении](./images/sign-up-page.png)

Эти кнопки позволяют вызывать действия в классе `AccountController`.

Действие `SignIn` возвращает **ChallegeResult**, в результате чего ПО промежуточного слоя OpenID Connect выполняет перенаправление в конечную точку проверки подлинности. Это заданный по умолчанию способ запуска проверки подлинности в ASP.NET Core.

```csharp
[AllowAnonymous]
public IActionResult SignIn()
{
    return new ChallengeResult(
        OpenIdConnectDefaults.AuthenticationScheme,
        new AuthenticationProperties
        {
            IsPersistent = true,
            RedirectUri = Url.Action("SignInCallback", "Account")
        });
}
```

Теперь сравните действие `SignUp` :

```csharp
[AllowAnonymous]
public IActionResult SignUp()
{
    var state = new Dictionary<string, string> { { "signup", "true" }};
    return new ChallengeResult(
        OpenIdConnectDefaults.AuthenticationScheme,
        new AuthenticationProperties(state)
        {
            RedirectUri = Url.Action(nameof(SignUpCallback), "Account")
        });
}
```

Как и `SignIn`, действие `SignUp` также возвращает `ChallengeResult`. Но на этот раз мы добавляем в `AuthenticationProperties` in the `ChallengeResult`фрагмент сведений о состоянии:

* signup: логический флаг, указывающий на то, что пользователь начал процесс регистрации.

Сведения о состоянии в `AuthenticationProperties` добавляются в параметр [state] OpenID Connect для круговых обходов во время потока проверки подлинности.

![Параметр состояния](./images/state-parameter.png)

После того как пользователь пройдет проверку подлинности в Azure AD и будет перенаправлен обратно в приложение, в билете проверки подлинности будет указано состояние. Мы используем этот факт, чтобы гарантировать, что значение "signup" сохраняется во всем потоке проверки подлинности.

## <a name="adding-the-admin-consent-prompt"></a>Добавление запроса на согласие администратора

В Azure AD поток согласия администратора запускается путем добавления параметра "prompt" в строку запроса в запросе на проверку подлинности.

<!-- markdownlint-disable MD040 -->

```
/authorize?prompt=admin_consent&...
```

<!-- markdownlint-enable MD040 -->

Приложение Surveys добавляет запрос во время события `RedirectToAuthenticationEndpoint` . Это событие вызывается сразу перед тем, как ПО промежуточного слоя выполнит перенаправление в конечную точку проверки подлинности.

```csharp
public override Task RedirectToAuthenticationEndpoint(RedirectContext context)
{
    if (context.IsSigningUp())
    {
        context.ProtocolMessage.Prompt = "admin_consent";
    }

    _logger.RedirectToIdentityProvider();
    return Task.FromResult(0);
}
```

При настройке `ProtocolMessage.Prompt` ПО промежуточного слоя добавляет параметр prompt в запрос аутентификации.

Обратите внимание, что запрос необходим только во время регистрации. Он не требуется для обычного входа. Чтобы различить их, проверим наличие значения `signup` в состоянии проверки подлинности. Для проверки этого условия используется следующий метод расширения:

```csharp
internal static bool IsSigningUp(this BaseControlContext context)
{
    Guard.ArgumentNotNull(context, nameof(context));

    string signupValue;
    // Check the HTTP context and convert to string
    if ((context.Ticket == null) ||
        (!context.Ticket.Properties.Items.TryGetValue("signup", out signupValue)))
    {
        return false;
    }

    // We have found the value, so see if it's valid
    bool isSigningUp;
    if (!bool.TryParse(signupValue, out isSigningUp))
    {
        // The value for signup is not a valid boolean, throw

        throw new InvalidOperationException($"'{signupValue}' is an invalid boolean value");
    }

    return isSigningUp;
}
```

## <a name="registering-a-tenant"></a>Регистрация клиента

Приложение Surveys хранит сведения о каждом клиенте и пользователе в базе данных приложения.

![Таблица клиентов](./images/tenant-table.png)

В таблице Tenant IssuerValue является значением утверждения издателя для клиента. Для Azure AD это `https://sts.windows.net/<tentantID>` с уникальным значением для каждого клиента.

При регистрации нового клиента приложение Surveys вносит запись клиента в базу данных. Это происходит внутри события `AuthenticationValidated` . (Не выполняйте это действие до события, так как маркер идентификатора пока не прошел проверку и значениям утверждения доверять нельзя.) См. статью [Проверка подлинности].

Ниже приведен соответствующий код из приложения Surveys:

```csharp
public override async Task TokenValidated(TokenValidatedContext context)
{
    var principal = context.AuthenticationTicket.Principal;
    var userId = principal.GetObjectIdentifierValue();
    var tenantManager = context.HttpContext.RequestServices.GetService<TenantManager>();
    var userManager = context.HttpContext.RequestServices.GetService<UserManager>();
    var issuerValue = principal.GetIssuerValue();
    _logger.AuthenticationValidated(userId, issuerValue);

    // Normalize the claims first.
    NormalizeClaims(principal);
    var tenant = await tenantManager.FindByIssuerValueAsync(issuerValue)
        .ConfigureAwait(false);

    if (context.IsSigningUp())
    {
        if (tenant == null)
        {
            tenant = await SignUpTenantAsync(context, tenantManager)
                .ConfigureAwait(false);
        }

        // In this case, we need to go ahead and set up the user signing us up.
        await CreateOrUpdateUserAsync(context.Ticket, userManager, tenant)
            .ConfigureAwait(false);
    }
    else
    {
        if (tenant == null)
        {
            _logger.UnregisteredUserSignInAttempted(userId, issuerValue);
            throw new SecurityTokenValidationException($"Tenant {issuerValue} is not registered");
        }

        await CreateOrUpdateUserAsync(context.Ticket, userManager, tenant)
            .ConfigureAwait(false);
    }
}
```

Этот код делает следующее:

1. Проверяет наличие значения издателя клиента в базе данных. Если клиент не зарегистрирован, `FindByIssuerValueAsync` возвращает значение NULL.
2. Если пользователь проходит регистрацию:
   1. Добавляет клиента в базу данных (`SignUpTenantAsync`).
   2. Добавляет прошедшего проверку подлинности пользователя в базу данных (`CreateOrUpdateUserAsync`).
3. В противном случае выполняется обычный поток входа:
   1. Если издатель клиента не найден в базе данных, это значит, что клиент не зарегистрирован, и заказчику нужно зарегистрироваться. В этом случае создает исключение для сбоя проверки подлинности.
   2. В противном случае создает запись в базе данных для этого пользователя, если ее еще нет (`CreateOrUpdateUserAsync`).

Далее приведен метод `SignUpTenantAsync`, добавляющий клиент в базу данных.

```csharp
private async Task<Tenant> SignUpTenantAsync(BaseControlContext context, TenantManager tenantManager)
{
    Guard.ArgumentNotNull(context, nameof(context));
    Guard.ArgumentNotNull(tenantManager, nameof(tenantManager));

    var principal = context.Ticket.Principal;
    var issuerValue = principal.GetIssuerValue();
    var tenant = new Tenant
    {
        IssuerValue = issuerValue,
        Created = DateTimeOffset.UtcNow
    };

    try
    {
        await tenantManager.CreateAsync(tenant)
            .ConfigureAwait(false);
    }
    catch(Exception ex)
    {
        _logger.SignUpTenantFailed(principal.GetObjectIdentifierValue(), issuerValue, ex);
        throw;
    }

    return tenant;
}
```

Далее приведено краткое описание всего потока регистрации в приложении Surveys:

1. Пользователь нажимает кнопку **Регистрация** .
2. Действие `AccountController.SignUp` возвращает ChallengeResult.  Состояние проверки подлинности содержит значение "signup".
3. Добавьте строки `admin_consent` в событие `RedirectToAuthenticationEndpoint`.
4. ПО промежуточного слоя OpenID Connect осуществляет перенаправление в Azure AD, и пользователь проходит проверку подлинности.
5. В событии `AuthenticationValidated` выполняется поиск состояния "signup".
6. Клиент добавляется в базу данных.

[**Далее**][app roles]

<!-- links -->

[app roles]: app-roles.md
[Tailspin]: tailspin.md

[state]: https://openid.net/specs/openid-connect-core-1_0.html#AuthRequest
[Проверка подлинности]: authenticate.md
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
