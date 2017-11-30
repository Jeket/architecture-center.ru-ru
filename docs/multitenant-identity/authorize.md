---
title: "Авторизация в мультитенантных приложениях"
description: "Выполнение авторизации в мультитенантном приложении"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
ms.openlocfilehash: 86c308d21f19bb3ac2a4a2240a9a03a504de5cf4
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2017
---
# <a name="role-based-and-resource-based-authorization"></a>Авторизация на основе ролей и ресурсов

[![GitHub](../_images/github.png) Пример кода][sample application]

[Эталонная реализация] является приложением ASP.NET Core. В этой статье мы рассмотрим два стандартных подхода к авторизации на основе API авторизации, доступным в ASP.NET Core.

* **Авторизация на основе ролей**. Авторизация действия на основе ролей, назначенных пользователю. Например, для выполнения некоторых действий требуется роль администратора.
* **Авторизация на основе ресурсов**. Авторизация действия на основе конкретного ресурса. Например, каждый ресурс имеет владельца. Владелец может удалить ресурс, а другие пользователи — нет.

В стандартном приложении используется сочетание обоих вариантов. Например, чтобы удалить ресурс, пользователь должен быть владельцем этого ресурса *или* администратором.

## <a name="role-based-authorization"></a>Авторизация на основе ролей
Приложение [Tailspin Surveys][Tailspin] определяет следующие роли.

* Администратор. Может выполнять все операции CRUD в любом опросе, принадлежащем этому клиенту.
* Создатель. Может создавать опросы.
* Читатель. Может читать все опросы, принадлежащие этому клиенту.

Роли применяются к *пользователям* приложения. В приложении Surveys пользователь является или администратором, или создателем, или читателем.

Инструкции по определению ролей и управлению ими см. в статье о [ролях приложения].

Код авторизации не зависит от способа управления ролями, поэтому он не будет иметь существенных отличий. В ASP.NET Core есть абстракция, которая называется [политики авторизации][policies]. С помощью этой функции пользователь определяет политики авторизации в коде, а затем применяет их к действиям контроллера. Политика не связана с контроллером.

### <a name="create-policies"></a>Создайте политики.
Чтобы определить политику, сначала следует создать класс, реализующий `IAuthorizationRequirement`. Проще всего извлечь его из `AuthorizationHandler`. В методе `Handle` проверьте соответствующие утверждения.

Ниже приведен пример из приложения Tailspin Surveys.

```csharp
public class SurveyCreatorRequirement : AuthorizationHandler<SurveyCreatorRequirement>, IAuthorizationRequirement
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, SurveyCreatorRequirement requirement)
    {
        if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin) || 
            context.User.HasClaim(ClaimTypes.Role, Roles.SurveyCreator))
        {
            context.Succeed(requirement);
        }
        return Task.FromResult(0);
    }
}
```

Этот класс определяет требование для пользователя по созданию опроса. Пользователю должна быть назначена роль SurveyAdmin или SurveyCreator.

В классе запуска определите именованную политику, содержащую одно или несколько требований. Если существует несколько требований, пользователь должен выполнить *каждое*, чтобы пройти авторизацию. Следующий код определяет две политики:

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy(PolicyNames.RequireSurveyCreator,
        policy =>
        {
            policy.AddRequirements(new SurveyCreatorRequirement());
            policy.RequireAuthenticatedUser(); // Adds DenyAnonymousAuthorizationRequirement 
            // By adding the CookieAuthenticationDefaults.AuthenticationScheme, if an authenticated
            // user is not in the appropriate role, they will be redirected to a "forbidden" page.
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });

    options.AddPolicy(PolicyNames.RequireSurveyAdmin,
        policy =>
        {
            policy.AddRequirements(new SurveyAdminRequirement());
            policy.RequireAuthenticatedUser();  
            policy.AddAuthenticationSchemes(CookieAuthenticationDefaults.AuthenticationScheme);
        });
});
```

Этот код создает схему аутентификации. Эта схема указывает ПО промежуточного слоя, которое ASP.NET будет выполнять в случае неудачной авторизации. В этом примере указано ПО промежуточного слоя для аутентификации по файлам cookie, поскольку оно может перенаправлять пользователей на страницу "Запрещено". Расположение страницы "Запрещено" задается в параметре `AccessDeniedPath` для ПО промежуточного слоя. Сведения об этом вы найдете в разделе о [настройке ПО промежуточного слоя для аутентификации].

### <a name="authorize-controller-actions"></a>Авторизация действий контроллера
И, наконец, чтобы авторизовать действие в контроллере MVC, можно задать политику в атрибуте `Authorize` , как показано ниже.

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

В более ранних версиях ASP.NET необходимо задать свойство **Roles** для атрибута, как показано ниже.

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]

```

Эта возможность все еще поддерживается в ASP.NET Core, но она имеет определенные недостатки по сравнению с политиками авторизации.

* Она предполагает использование определенного типа утверждения. Политики могут проверять наличие любого типа утверждения. Роли являются лишь типом утверждения.
* Имя роли жестко задается в атрибуте. Благодаря политикам логика авторизации реализуется в одном месте, что упрощает обновление или даже скачивание из параметров конфигурации.
* Политики позволяют принимать более сложные решения по авторизации (например, возраст 21 год и старше), которые не могут быть выражены простым членством в роли.

## <a name="resource-based-authorization"></a>Авторизация на основе ресурсов
*Авторизация на основе ресурсов* выполняется каждый раз, когда авторизация зависит от конкретного ресурса, затрагиваемого операцией. В приложении Tailspin Surveys для каждого опроса существует владелец и ни одного или несколько участников.

* Владелец может читать, обновлять, удалять, публиковать опрос и отменять его публикацию.
* Владелец может назначать участников опроса.
* Участники могут читать и обновлять опрос.

Обратите внимание, что "владелец" и "участник" не являются ролями приложения, а сохраняются отдельно для каждого опроса в базе данных приложения. Чтобы определить, может ли пользователь удалять опрос, приложение проверяет, является ли этот пользователь владельцем этого опроса.

В ASP.NET Core авторизация на основе ресурсов наследуется от **AuthorizationHandler** с переопределением метода **Handle**.

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

Обратите внимание, что этот класс является строго типизированным для объектов опроса.  Зарегистрируйте класс для DI при запуске, как показано ниже.

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

Для проверок авторизации используйте интерфейс **IAuthorizationService** , который можно внедрить в контроллеры. Следующий код проверяет, может ли пользователь читать опрос:

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

Мы передаем объект `Survey`. Этот вызов приведет к выполнению `SurveyAuthorizationHandler`.

В коде авторизации целесообразно объединить все разрешения пользователя на основе ролей и ресурсов, а затем проверить обобщенный набор на возможность выполнения нужной операции.
Ниже приведен пример из приложения Surveys. Приложение определяет несколько типов разрешений:

* Администратор
* Участник
* создатель;
* Владелец
* читатель.

Приложение также определяет набор возможных операций с опросами:

* Создание
* чтение
* Блокировка изменений
* Удалить
* Опубликовать
* Отмена публикации

Приведенный ниже код создает список разрешений для конкретного пользователя и опроса. Обратите внимание, что этот код просматривает роли приложения пользователя и поля владельца или участника в опросе.

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement requirement, Survey resource)
    {
        var permissions = new List<UserPermissionType>();
        int surveyTenantId = context.User.GetSurveyTenantIdValue();
        int userId = context.User.GetSurveyUserIdValue();
        string user = context.User.GetUserName();

        if (resource.TenantId == surveyTenantId)
        {
            // Admin can do anything, as long as the resource belongs to the admin's tenant.
            if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyAdmin))
            {
                context.Succeed(requirement);
                return Task.FromResult(0);
            }

            if (context.User.HasClaim(ClaimTypes.Role, Roles.SurveyCreator))
            {
                permissions.Add(UserPermissionType.Creator);
            }
            else
            {
                permissions.Add(UserPermissionType.Reader);
            }

            if (resource.OwnerId == userId)
            {
                permissions.Add(UserPermissionType.Owner);
            }
        }
        if (resource.Contributors != null && resource.Contributors.Any(x => x.UserId == userId))
        {
            permissions.Add(UserPermissionType.Contributor);
        }

        if (ValidateUserPermissions[requirement](permissions))
        {
            context.Succeed(requirement);
        }
        return Task.FromResult(0);
    }
}
```

В мультитенантном приложении важно следить за тем, чтобы разрешения не затрагивали данные других клиентов. В приложении Surveys допускается разрешение "Участник" для других клиентов, то есть вы можете назначить пользователя другого клиента в качестве участника. Другие типы разрешений допустимы только для ресурсов, которые принадлежат тому же клиенту. Чтобы это требование соблюдалось, код проверяет идентификатор клиента, прежде чем предоставить ему разрешение. (Поле `TenantId` , назначенное при создании опроса.)

Следующий шаг — проверка разрешений для конкретной операции (чтение, обновление, удаление и т. д). Приложение Surveys реализует этот шаг с помощью таблицы подстановки функций.

```csharp
static readonly Dictionary<OperationAuthorizationRequirement, Func<List<UserPermissionType>, bool>> ValidateUserPermissions
    = new Dictionary<OperationAuthorizationRequirement, Func<List<UserPermissionType>, bool>>

    {
        { Operations.Create, x => x.Contains(UserPermissionType.Creator) },

        { Operations.Read, x => x.Contains(UserPermissionType.Creator) ||
                                x.Contains(UserPermissionType.Reader) ||
                                x.Contains(UserPermissionType.Contributor) ||
                                x.Contains(UserPermissionType.Owner) },

        { Operations.Update, x => x.Contains(UserPermissionType.Contributor) ||
                                x.Contains(UserPermissionType.Owner) },

        { Operations.Delete, x => x.Contains(UserPermissionType.Owner) },

        { Operations.Publish, x => x.Contains(UserPermissionType.Owner) },

        { Operations.UnPublish, x => x.Contains(UserPermissionType.Owner) }
    };
```

[**Далее**][web-api]

<!-- Links -->
[Tailspin]: tailspin.md

[ролях приложения]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[Эталонная реализация]: tailspin.md
[настройке ПО промежуточного слоя для аутентификации]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
