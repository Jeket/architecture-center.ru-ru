---
title: Авторизация в мультитенантных приложениях
description: Выполнение авторизации в мультитенантном приложении
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: app-roles
pnp.series.next: web-api
ms.openlocfilehash: 03c4d5fa10c75437a7b066534619ba9a123c350c
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2018
---
# <a name="role-based-and-resource-based-authorization"></a><span data-ttu-id="d029d-103">Авторизация на основе ролей и ресурсов</span><span class="sxs-lookup"><span data-stu-id="d029d-103">Role-based and resource-based authorization</span></span>

<span data-ttu-id="d029d-104">[![GitHub](../_images/github.png) Пример кода][sample application]</span><span class="sxs-lookup"><span data-stu-id="d029d-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="d029d-105">[Эталонная реализация] является приложением ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="d029d-105">Our [reference implementation] is an ASP.NET Core application.</span></span> <span data-ttu-id="d029d-106">В этой статье мы рассмотрим два стандартных подхода к авторизации на основе API авторизации, доступным в ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="d029d-106">In this article we'll look at two general approaches to authorization, using the authorization APIs provided in ASP.NET Core.</span></span>

* <span data-ttu-id="d029d-107">**Авторизация на основе ролей**.</span><span class="sxs-lookup"><span data-stu-id="d029d-107">**Role-based authorization**.</span></span> <span data-ttu-id="d029d-108">Авторизация действия на основе ролей, назначенных пользователю.</span><span class="sxs-lookup"><span data-stu-id="d029d-108">Authorizing an action based on the roles assigned to a user.</span></span> <span data-ttu-id="d029d-109">Например, для выполнения некоторых действий требуется роль администратора.</span><span class="sxs-lookup"><span data-stu-id="d029d-109">For example, some actions require an administrator role.</span></span>
* <span data-ttu-id="d029d-110">**Авторизация на основе ресурсов**.</span><span class="sxs-lookup"><span data-stu-id="d029d-110">**Resource-based authorization**.</span></span> <span data-ttu-id="d029d-111">Авторизация действия на основе конкретного ресурса.</span><span class="sxs-lookup"><span data-stu-id="d029d-111">Authorizing an action based on a particular resource.</span></span> <span data-ttu-id="d029d-112">Например, каждый ресурс имеет владельца.</span><span class="sxs-lookup"><span data-stu-id="d029d-112">For example, every resource has an owner.</span></span> <span data-ttu-id="d029d-113">Владелец может удалить ресурс, а другие пользователи — нет.</span><span class="sxs-lookup"><span data-stu-id="d029d-113">The owner can delete the resource; other users cannot.</span></span>

<span data-ttu-id="d029d-114">В стандартном приложении используется сочетание обоих вариантов.</span><span class="sxs-lookup"><span data-stu-id="d029d-114">A typical app will employ a mix of both.</span></span> <span data-ttu-id="d029d-115">Например, чтобы удалить ресурс, пользователь должен быть владельцем этого ресурса *или* администратором.</span><span class="sxs-lookup"><span data-stu-id="d029d-115">For example, to delete a resource, the user must be the resource owner *or* an admin.</span></span>

## <a name="role-based-authorization"></a><span data-ttu-id="d029d-116">Авторизация на основе ролей</span><span class="sxs-lookup"><span data-stu-id="d029d-116">Role-Based Authorization</span></span>
<span data-ttu-id="d029d-117">Приложение [Tailspin Surveys][Tailspin] определяет следующие роли.</span><span class="sxs-lookup"><span data-stu-id="d029d-117">The [Tailspin Surveys][Tailspin] application defines the following roles:</span></span>

* <span data-ttu-id="d029d-118">Администратор.</span><span class="sxs-lookup"><span data-stu-id="d029d-118">Administrator.</span></span> <span data-ttu-id="d029d-119">Может выполнять все операции CRUD в любом опросе, принадлежащем этому клиенту.</span><span class="sxs-lookup"><span data-stu-id="d029d-119">Can perform all CRUD operations on any survey that belongs to that tenant.</span></span>
* <span data-ttu-id="d029d-120">Создатель.</span><span class="sxs-lookup"><span data-stu-id="d029d-120">Creator.</span></span> <span data-ttu-id="d029d-121">Может создавать опросы.</span><span class="sxs-lookup"><span data-stu-id="d029d-121">Can create new surveys</span></span>
* <span data-ttu-id="d029d-122">Читатель.</span><span class="sxs-lookup"><span data-stu-id="d029d-122">Reader.</span></span> <span data-ttu-id="d029d-123">Может читать все опросы, принадлежащие этому клиенту.</span><span class="sxs-lookup"><span data-stu-id="d029d-123">Can read any surveys that belong to that tenant</span></span>

<span data-ttu-id="d029d-124">Роли применяются к *пользователям* приложения.</span><span class="sxs-lookup"><span data-stu-id="d029d-124">Roles apply to *users* of the application.</span></span> <span data-ttu-id="d029d-125">В приложении Surveys пользователь является или администратором, или создателем, или читателем.</span><span class="sxs-lookup"><span data-stu-id="d029d-125">In the Surveys application, a user is either an administrator, creator, or reader.</span></span>

<span data-ttu-id="d029d-126">Инструкции по определению ролей и управлению ими см. в статье о [ролях приложения].</span><span class="sxs-lookup"><span data-stu-id="d029d-126">For a discussion of how to define and manage roles, see [Application roles].</span></span>

<span data-ttu-id="d029d-127">Код авторизации не зависит от способа управления ролями, поэтому он не будет иметь существенных отличий.</span><span class="sxs-lookup"><span data-stu-id="d029d-127">Regardless of how you manage the roles, your authorization code will look similar.</span></span> <span data-ttu-id="d029d-128">В ASP.NET Core есть абстракция, которая называется [политики авторизации][policies].</span><span class="sxs-lookup"><span data-stu-id="d029d-128">ASP.NET Core has an abstraction called [authorization policies][policies].</span></span> <span data-ttu-id="d029d-129">С помощью этой функции пользователь определяет политики авторизации в коде, а затем применяет их к действиям контроллера.</span><span class="sxs-lookup"><span data-stu-id="d029d-129">With this feature, you define authorization policies in code, and then apply those policies to controller actions.</span></span> <span data-ttu-id="d029d-130">Политика не связана с контроллером.</span><span class="sxs-lookup"><span data-stu-id="d029d-130">The policy is decoupled from the controller.</span></span>

### <a name="create-policies"></a><span data-ttu-id="d029d-131">Создайте политики.</span><span class="sxs-lookup"><span data-stu-id="d029d-131">Create policies</span></span>
<span data-ttu-id="d029d-132">Чтобы определить политику, сначала следует создать класс, реализующий `IAuthorizationRequirement`.</span><span class="sxs-lookup"><span data-stu-id="d029d-132">To define a policy, first create a class that implements `IAuthorizationRequirement`.</span></span> <span data-ttu-id="d029d-133">Проще всего извлечь его из `AuthorizationHandler`.</span><span class="sxs-lookup"><span data-stu-id="d029d-133">It's easiest to derive from `AuthorizationHandler`.</span></span> <span data-ttu-id="d029d-134">В методе `Handle` проверьте соответствующие утверждения.</span><span class="sxs-lookup"><span data-stu-id="d029d-134">In the `Handle` method, examine the relevant claim(s).</span></span>

<span data-ttu-id="d029d-135">Ниже приведен пример из приложения Tailspin Surveys.</span><span class="sxs-lookup"><span data-stu-id="d029d-135">Here is an example from the Tailspin Surveys application:</span></span>

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

<span data-ttu-id="d029d-136">Этот класс определяет требование для пользователя по созданию опроса.</span><span class="sxs-lookup"><span data-stu-id="d029d-136">This class defines the requirement for a user to create a new survey.</span></span> <span data-ttu-id="d029d-137">Пользователю должна быть назначена роль SurveyAdmin или SurveyCreator.</span><span class="sxs-lookup"><span data-stu-id="d029d-137">The user must be in the SurveyAdmin or SurveyCreator role.</span></span>

<span data-ttu-id="d029d-138">В классе запуска определите именованную политику, содержащую одно или несколько требований.</span><span class="sxs-lookup"><span data-stu-id="d029d-138">In your startup class, define a named policy that includes one or more requirements.</span></span> <span data-ttu-id="d029d-139">Если существует несколько требований, пользователь должен выполнить *каждое*, чтобы пройти авторизацию.</span><span class="sxs-lookup"><span data-stu-id="d029d-139">If there are multiple requirements, the user must meet *every* requirement to be authorized.</span></span> <span data-ttu-id="d029d-140">Следующий код определяет две политики:</span><span class="sxs-lookup"><span data-stu-id="d029d-140">The following code defines two policies:</span></span>

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

<span data-ttu-id="d029d-141">Этот код создает схему аутентификации. Эта схема указывает ПО промежуточного слоя, которое ASP.NET будет выполнять в случае неудачной авторизации.</span><span class="sxs-lookup"><span data-stu-id="d029d-141">This code also sets the authentication scheme, which tells ASP.NET which authentication middleware should run if authorization fails.</span></span> <span data-ttu-id="d029d-142">В этом примере указано ПО промежуточного слоя для аутентификации по файлам cookie, поскольку оно может перенаправлять пользователей на страницу "Запрещено".</span><span class="sxs-lookup"><span data-stu-id="d029d-142">In this case, we specify the cookie authentication middleware, because the cookie authentication middleware can redirect the user to a "Forbidden" page.</span></span> <span data-ttu-id="d029d-143">Расположение страницы "Запрещено" задается в параметре `AccessDeniedPath` для ПО промежуточного слоя. Сведения об этом вы найдете в разделе о [настройке ПО промежуточного слоя для аутентификации].</span><span class="sxs-lookup"><span data-stu-id="d029d-143">The location of the Forbidden page is set in the `AccessDeniedPath` option for the cookie middleware; see [Configuring the authentication middleware].</span></span>

### <a name="authorize-controller-actions"></a><span data-ttu-id="d029d-144">Авторизация действий контроллера</span><span class="sxs-lookup"><span data-stu-id="d029d-144">Authorize controller actions</span></span>
<span data-ttu-id="d029d-145">И, наконец, чтобы авторизовать действие в контроллере MVC, можно задать политику в атрибуте `Authorize` , как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="d029d-145">Finally, to authorize an action in an MVC controller, set the policy in the `Authorize` attribute:</span></span>

```csharp
[Authorize(Policy = PolicyNames.RequireSurveyCreator)]
public IActionResult Create()
{
    var survey = new SurveyDTO();
    return View(survey);
}
```

<span data-ttu-id="d029d-146">В более ранних версиях ASP.NET необходимо задать свойство **Roles** для атрибута, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="d029d-146">In earlier versions of ASP.NET, you would set the **Roles** property on the attribute:</span></span>

```csharp
// old way
[Authorize(Roles = "SurveyCreator")]
```

<span data-ttu-id="d029d-147">Эта возможность все еще поддерживается в ASP.NET Core, но она имеет определенные недостатки по сравнению с политиками авторизации.</span><span class="sxs-lookup"><span data-stu-id="d029d-147">This is still supported in ASP.NET Core, but it has some drawbacks compared with authorization policies:</span></span>

* <span data-ttu-id="d029d-148">Она предполагает использование определенного типа утверждения.</span><span class="sxs-lookup"><span data-stu-id="d029d-148">It assumes a particular claim type.</span></span> <span data-ttu-id="d029d-149">Политики могут проверять наличие любого типа утверждения.</span><span class="sxs-lookup"><span data-stu-id="d029d-149">Policies can check for any claim type.</span></span> <span data-ttu-id="d029d-150">Роли являются лишь типом утверждения.</span><span class="sxs-lookup"><span data-stu-id="d029d-150">Roles are just a type of claim.</span></span>
* <span data-ttu-id="d029d-151">Имя роли жестко задается в атрибуте.</span><span class="sxs-lookup"><span data-stu-id="d029d-151">The role name is hard-coded into the attribute.</span></span> <span data-ttu-id="d029d-152">Благодаря политикам логика авторизации реализуется в одном месте, что упрощает обновление или даже скачивание из параметров конфигурации.</span><span class="sxs-lookup"><span data-stu-id="d029d-152">With policies, the authorization logic is all in one place, making it easier to update or even load from configuration settings.</span></span>
* <span data-ttu-id="d029d-153">Политики позволяют принимать более сложные решения по авторизации (например, возраст 21 год и старше), которые не могут быть выражены простым членством в роли.</span><span class="sxs-lookup"><span data-stu-id="d029d-153">Policies enable more complex authorization decisions (e.g., age >= 21) that can't be expressed by simple role membership.</span></span>

## <a name="resource-based-authorization"></a><span data-ttu-id="d029d-154">Авторизация на основе ресурсов</span><span class="sxs-lookup"><span data-stu-id="d029d-154">Resource based authorization</span></span>
<span data-ttu-id="d029d-155">*Авторизация на основе ресурсов* выполняется каждый раз, когда авторизация зависит от конкретного ресурса, затрагиваемого операцией.</span><span class="sxs-lookup"><span data-stu-id="d029d-155">*Resource based authorization* occurs whenever the authorization depends on a specific resource that will be affected by an operation.</span></span> <span data-ttu-id="d029d-156">В приложении Tailspin Surveys для каждого опроса существует владелец и ни одного или несколько участников.</span><span class="sxs-lookup"><span data-stu-id="d029d-156">In the Tailspin Surveys application, every survey has an owner and zero-to-many contributors.</span></span>

* <span data-ttu-id="d029d-157">Владелец может читать, обновлять, удалять, публиковать опрос и отменять его публикацию.</span><span class="sxs-lookup"><span data-stu-id="d029d-157">The owner can read, update, delete, publish, and unpublish the survey.</span></span>
* <span data-ttu-id="d029d-158">Владелец может назначать участников опроса.</span><span class="sxs-lookup"><span data-stu-id="d029d-158">The owner can assign contributors to the survey.</span></span>
* <span data-ttu-id="d029d-159">Участники могут читать и обновлять опрос.</span><span class="sxs-lookup"><span data-stu-id="d029d-159">Contributors can read and update the survey.</span></span>

<span data-ttu-id="d029d-160">Обратите внимание, что "владелец" и "участник" не являются ролями приложения, а сохраняются отдельно для каждого опроса в базе данных приложения.</span><span class="sxs-lookup"><span data-stu-id="d029d-160">Note that "owner" and "contributor" are not application roles; they are stored per survey, in the application database.</span></span> <span data-ttu-id="d029d-161">Чтобы определить, может ли пользователь удалять опрос, приложение проверяет, является ли этот пользователь владельцем этого опроса.</span><span class="sxs-lookup"><span data-stu-id="d029d-161">To check whether a user can delete a survey, for example, the app checks whether the user is the owner for that survey.</span></span>

<span data-ttu-id="d029d-162">В ASP.NET Core авторизация на основе ресурсов наследуется от **AuthorizationHandler** с переопределением метода **Handle**.</span><span class="sxs-lookup"><span data-stu-id="d029d-162">In ASP.NET Core, implement resource-based authorization by deriving from **AuthorizationHandler** and overriding the **Handle** method.</span></span>

```csharp
public class SurveyAuthorizationHandler : AuthorizationHandler<OperationAuthorizationRequirement, Survey>
{
    protected override void HandleRequirementAsync(AuthorizationHandlerContext context, OperationAuthorizationRequirement operation, Survey resource)
    {
    }
}
```

<span data-ttu-id="d029d-163">Обратите внимание, что этот класс является строго типизированным для объектов опроса.</span><span class="sxs-lookup"><span data-stu-id="d029d-163">Notice that this class is strongly typed for Survey objects.</span></span>  <span data-ttu-id="d029d-164">Зарегистрируйте класс для DI при запуске, как показано ниже.</span><span class="sxs-lookup"><span data-stu-id="d029d-164">Register the class for DI on startup:</span></span>

```csharp
services.AddSingleton<IAuthorizationHandler>(factory =>
{
    return new SurveyAuthorizationHandler();
});
```

<span data-ttu-id="d029d-165">Для проверок авторизации используйте интерфейс **IAuthorizationService** , который можно внедрить в контроллеры.</span><span class="sxs-lookup"><span data-stu-id="d029d-165">To perform authorization checks, use the **IAuthorizationService** interface, which you can inject into your controllers.</span></span> <span data-ttu-id="d029d-166">Следующий код проверяет, может ли пользователь читать опрос:</span><span class="sxs-lookup"><span data-stu-id="d029d-166">The following code checks whether a user can read a survey:</span></span>

```csharp
if (await _authorizationService.AuthorizeAsync(User, survey, Operations.Read) == false)
{
    return StatusCode(403);
}
```

<span data-ttu-id="d029d-167">Мы передаем объект `Survey`. Этот вызов приведет к выполнению `SurveyAuthorizationHandler`.</span><span class="sxs-lookup"><span data-stu-id="d029d-167">Because we pass in a `Survey` object, this call will invoke the `SurveyAuthorizationHandler`.</span></span>

<span data-ttu-id="d029d-168">В коде авторизации целесообразно объединить все разрешения пользователя на основе ролей и ресурсов, а затем проверить обобщенный набор на возможность выполнения нужной операции.</span><span class="sxs-lookup"><span data-stu-id="d029d-168">In your authorization code, a good approach is to aggregate all of the user's role-based and resource-based permissions, then check the aggregate set against the desired operation.</span></span>
<span data-ttu-id="d029d-169">Ниже приведен пример из приложения Surveys.</span><span class="sxs-lookup"><span data-stu-id="d029d-169">Here is an example from the Surveys app.</span></span> <span data-ttu-id="d029d-170">Приложение определяет несколько типов разрешений:</span><span class="sxs-lookup"><span data-stu-id="d029d-170">The application defines several permission types:</span></span>

* <span data-ttu-id="d029d-171">Администратор</span><span class="sxs-lookup"><span data-stu-id="d029d-171">Admin</span></span>
* <span data-ttu-id="d029d-172">участник;</span><span class="sxs-lookup"><span data-stu-id="d029d-172">Contributor</span></span>
* <span data-ttu-id="d029d-173">создатель;</span><span class="sxs-lookup"><span data-stu-id="d029d-173">Creator</span></span>
* <span data-ttu-id="d029d-174">Владелец.</span><span class="sxs-lookup"><span data-stu-id="d029d-174">Owner</span></span>
* <span data-ttu-id="d029d-175">Читатель</span><span class="sxs-lookup"><span data-stu-id="d029d-175">Reader</span></span>

<span data-ttu-id="d029d-176">Приложение также определяет набор возможных операций с опросами:</span><span class="sxs-lookup"><span data-stu-id="d029d-176">The application also defines a set of possible operations on surveys:</span></span>

* <span data-ttu-id="d029d-177">Создание</span><span class="sxs-lookup"><span data-stu-id="d029d-177">Create</span></span>
* <span data-ttu-id="d029d-178">чтение</span><span class="sxs-lookup"><span data-stu-id="d029d-178">Read</span></span>
* <span data-ttu-id="d029d-179">Блокировка изменений</span><span class="sxs-lookup"><span data-stu-id="d029d-179">Update</span></span>
* <span data-ttu-id="d029d-180">Delete</span><span class="sxs-lookup"><span data-stu-id="d029d-180">Delete</span></span>
* <span data-ttu-id="d029d-181">Опубликовать</span><span class="sxs-lookup"><span data-stu-id="d029d-181">Publish</span></span>
* <span data-ttu-id="d029d-182">Отмена публикации</span><span class="sxs-lookup"><span data-stu-id="d029d-182">Unpublsh</span></span>

<span data-ttu-id="d029d-183">Приведенный ниже код создает список разрешений для конкретного пользователя и опроса.</span><span class="sxs-lookup"><span data-stu-id="d029d-183">The following code creates a list of permissions for a particular user and survey.</span></span> <span data-ttu-id="d029d-184">Обратите внимание, что этот код просматривает роли приложения пользователя и поля владельца или участника в опросе.</span><span class="sxs-lookup"><span data-stu-id="d029d-184">Notice that this code looks at both the user's app roles, and the owner/contributor fields in the survey.</span></span>

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

<span data-ttu-id="d029d-185">В мультитенантном приложении важно следить за тем, чтобы разрешения не затрагивали данные других клиентов.</span><span class="sxs-lookup"><span data-stu-id="d029d-185">In a multi-tenant application, you must ensure that permissions don't "leak" to another tenant's data.</span></span> <span data-ttu-id="d029d-186">В приложении Surveys допускается разрешение "Участник" для других клиентов, то есть вы можете назначить пользователя другого клиента в качестве участника.</span><span class="sxs-lookup"><span data-stu-id="d029d-186">In the Surveys app, the Contributor permission is allowed across tenants &mdash; you can assign someone from another tenant as a contriubutor.</span></span> <span data-ttu-id="d029d-187">Другие типы разрешений допустимы только для ресурсов, которые принадлежат тому же клиенту.</span><span class="sxs-lookup"><span data-stu-id="d029d-187">The other permission types are restricted to resources that belong to that user's tenant.</span></span> <span data-ttu-id="d029d-188">Чтобы это требование соблюдалось, код проверяет идентификатор клиента, прежде чем предоставить ему разрешение.</span><span class="sxs-lookup"><span data-stu-id="d029d-188">To enforce this requirement, the code checks the tenant ID before granting the permission.</span></span> <span data-ttu-id="d029d-189">(Поле `TenantId` , назначенное при создании опроса.)</span><span class="sxs-lookup"><span data-stu-id="d029d-189">(The `TenantId` field as assigned when the survey is created.)</span></span>

<span data-ttu-id="d029d-190">Следующий шаг — проверка разрешений для конкретной операции (чтение, обновление, удаление и т. д).</span><span class="sxs-lookup"><span data-stu-id="d029d-190">The next step is to check the operation (read, update, delete, etc) against the permissions.</span></span> <span data-ttu-id="d029d-191">Приложение Surveys реализует этот шаг с помощью таблицы подстановки функций.</span><span class="sxs-lookup"><span data-stu-id="d029d-191">The Surveys app implements this step by using a lookup table of functions:</span></span>

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

<span data-ttu-id="d029d-192">[**Далее**][web-api]</span><span class="sxs-lookup"><span data-stu-id="d029d-192">[**Next**][web-api]</span></span>

<!-- Links -->
[Tailspin]: tailspin.md

[ролях приложения]: app-roles.md
[Application roles]: app-roles.md
[policies]: /aspnet/core/security/authorization/policies
[Эталонная реализация]: tailspin.md
[reference implementation]: tailspin.md
[настройке ПО промежуточного слоя для аутентификации]: authenticate.md#configure-the-auth-middleware
[Configuring the authentication middleware]: authenticate.md#configure-the-auth-middleware
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[web-api]: web-api.md
