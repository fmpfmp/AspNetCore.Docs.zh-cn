---
title: 使用 AzureBlazor活动目录保护ASP.NET核心 Web 组件托管应用
author: guardrex
description: ''
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 04/08/2020
no-loc:
- Blazor
- SignalR
uid: security/blazor/webassembly/hosted-with-azure-active-directory
ms.openlocfilehash: a80be8d145b7c58be35e2c353a448db7e234e20b
ms.sourcegitcommit: 5547d920f322e5a823575c031529e4755ab119de
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/21/2020
ms.locfileid: "81661835"
---
# <a name="secure-an-aspnet-core-opno-locblazor-webassembly-hosted-app-with-azure-active-directory"></a>使用 AzureBlazor活动目录保护ASP.NET核心 Web 组件托管应用

哈维尔[·卡尔瓦罗·纳尔逊](https://github.com/javiercn)和[卢克·莱瑟姆](https://github.com/guardrex)

[!INCLUDE[](~/includes/blazorwasm-preview-notice.md)]

[!INCLUDE[](~/includes/blazorwasm-3.2-template-article-notice.md)]

本文介绍如何创建使用[Azure 活动目录 （AAD）](https://azure.microsoft.com/services/active-directory/)进行身份验证的[BlazorWebAssembly 托管应用](xref:blazor/hosting-models#blazor-webassembly)。

## <a name="register-apps-in-aad-b2c-and-create-solution"></a>在 AAD B2C 中注册应用并创建解决方案

### <a name="create-a-tenant"></a>创建租户

按照["快速入门"中的指南：设置租户](/azure/active-directory/develop/quickstart-create-new-tenant)以在 AAD 中创建租户。

### <a name="register-a-server-api-app"></a>注册服务器 API 应用

按照["快速入门"中的指南操作：将应用程序注册到 Microsoft 标识平台](/azure/active-directory/develop/quickstart-register-app)和后续 Azure AAD 主题，以在 Azure 门户的**Azure 活动目录** > **应用注册**区域注册*服务器 API 应用*的 AAD 应用：

1. 选择“新注册”。 
1. 为应用提供**名称**（例如，**Blazor服务器 AAD**）。
1. 选择**受支持的帐户类型**。 对于此体验，您只能选择**此组织目录中的帐户**（单个租户）。
1. 在这种情况下，*服务器 API 应用*不需要重定向**URI，** 因此请将下拉列表设置为**Web，** 并且不输入重定向 URI。
1. 禁用 **"权限** > **授予管理员集中打开"和offline_access权限**复选框。
1. 选择“注册”  。

在**API 权限**中，删除 Microsoft**图形** > **用户.读取**权限，因为应用不需要登录或访问配置文件。

在**公开 API 中**：

1. 选择 **"添加范围**"。
1. 选择“保存并继续”。****
1. 提供**范围名称**（例如， `API.Access`
1. 提供**管理员同意显示名称**（例如， `Access API`
1. 提供**管理员同意说明**（例如， `Allows the app to access server app API endpoints.`
1. 确认**状态**设置为**启用**。
1. 选择 **"添加范围**"。

记录以下信息：

* *服务器 API 应用*应用程序 ID（客户端 ID）（例如`11111111-1111-1111-1111-111111111111`）
* 应用 ID URI（例如`https://contoso.onmicrosoft.com/11111111-1111-1111-1111-111111111111`，`api://11111111-1111-1111-1111-111111111111`或您提供的自定义值）
* 目录 ID（租户 ID）（例如`222222222-2222-2222-2222-222222222222`）
* AAD 租户域（例如） `contoso.onmicrosoft.com`
* 默认作用域（例如）， `API.Access`

### <a name="register-a-client-app"></a>注册客户端应用

按照["快速入门"中的指南操作：将应用程序注册到 Microsoft 标识平台](/azure/active-directory/develop/quickstart-register-app)和后续 Azure AAD 主题，以在 Azure 门户的**Azure 活动目录** > **应用注册**区域注册*客户端应用*的 AAD 应用：

1. 选择“新注册”。 
1. 为应用提供**名称**（例如，**Blazor客户端 AAD**）。
1. 选择**受支持的帐户类型**。 对于此体验，您只能选择**此组织目录中的帐户**（单个租户）。
1. 将**重定向 URI**下拉列表设置为**Web，** 并提供 重定向 URI。 `https://localhost:5001/authentication/login-callback`
1. 禁用 **"权限** > **授予管理员集中打开"和offline_access权限**复选框。
1. 选择“注册”  。

在**身份验证** > **平台配置中** > **，Web**：

1. 确认存在 重定向`https://localhost:5001/authentication/login-callback` **URI。**
1. 对于**隐式授予**，选择 Access**令牌**和**ID 令牌的**复选框。
1. 此体验可以接受应用的剩余默认值。
1. 选择"**保存**"按钮。

在**API 权限**中：

1. 确认应用具有**Microsoft 图形** > **用户.读取**权限。
1. 选择 **"添加"** 后跟**我的 API 的权限**。
1. 从 **"名称"** 列中选择*服务器 API 应用*（例如，**Blazor服务器 AAD**）。
1. 打开**API**列表。
1. 启用对 API 的访问（例如， `API.Access`
1. 选择“添加权限”  。
1. 选择 **[TENANT 名称] 按钮的授予管理员内容**。 请选择“是”以确认。****

记录*客户端应用*应用程序 ID（客户端 ID）（例如。 `33333333-3333-3333-3333-333333333333`

### <a name="create-the-app"></a>创建应用

将以下命令中的占位符替换为前面记录的信息，并在命令 shell 中执行该命令：

```dotnetcli
dotnet new blazorwasm -au SingleOrg --api-client-id "{SERVER API APP CLIENT ID}" --app-id-uri "{SERVER API APP ID URI}" --client-id "{CLIENT APP CLIENT ID}" --default-scope "{DEFAULT SCOPE}" --domain "{DOMAIN}" -ho --tenant-id "{TENANT ID}"
```

要指定输出位置（如果不存在，则创建项目文件夹）请在命令中包含具有路径的输出选项（例如。 `-o BlazorSample` 文件夹名称也将成为项目名称的一部分。

> [!NOTE]
> 将应用 ID URI`app-id-uri`传递给该选项，但请注意客户端应用中可能需要更改配置，这在[Access 令牌作用域](#access-token-scopes)部分中对此进行了说明。

## <a name="server-app-configuration"></a>服务器应用配置

*本节涉及解决方案的 **"服务器"** 应用。*

### <a name="authentication-package"></a>身份验证包

对验证和授权调用ASP.NET核心 Web API 的支持由： `Microsoft.AspNetCore.Authentication.AzureAD.UI`

```xml
<PackageReference Include="Microsoft.AspNetCore.Authentication.AzureAD.UI" 
    Version="3.1.0" />
```

### <a name="authentication-service-support"></a>身份验证服务支持

该方法`AddAuthentication`在应用中设置身份验证服务，并将 JWT 承载处理程序配置为默认身份验证方法。 该方法`AddAzureADBearer`在 JWT 承载处理程序中设置验证 Azure 活动目录发出的令牌所需的特定参数：

```csharp
services.AddAuthentication(AzureADDefaults.BearerAuthenticationScheme)
    .AddAzureADBearer(options => Configuration.Bind("AzureAd", options));
```

`UseAuthentication`并确保`UseAuthorization`：

* 应用尝试对传入请求解析和验证令牌。
* 尝试在没有适当凭据的情况下访问受保护资源的任何请求都失败。

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

### <a name="useridentityname"></a>User.Identity.Name

默认情况下，服务器应用 API`User.Identity.Name`使用`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name`声明类型中的值填充（例如， `2d64b3da-d9d5-42c6-9352-53d8df33d770@contoso.onmicrosoft.com`。

要将应用配置为从`name`声明类型接收值，请配置<xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerOptions>中的`Startup.ConfigureServices`[令牌验证参数。](xref:Microsoft.IdentityModel.Tokens.TokenValidationParameters.NameClaimType)

```csharp
services.Configure<JwtBearerOptions>(
    AzureADDefaults.JwtBearerAuthenticationScheme, options =>
    {
        options.TokenValidationParameters.NameClaimType = "name";
    });
```

### <a name="app-settings"></a>应用设置

*appsettings.json*文件包含用于配置用于验证访问令牌的 JWT 承载处理程序的选项。

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "{DOMAIN}",
    "TenantId": "{TENANT ID}",
    "ClientId": "{API CLIENT ID}",
  }
}
```

### <a name="weatherforecast-controller"></a>天气预报控制器

天气预报控制器 （*控制器/天气预报控制器.cs*） 公开受保护的 API，`[Authorize]`该属性应用于控制器。 **请务必**了解：

* 此`[Authorize]`API 控制器中的属性是保护此 API 免受未经授权的访问的唯一原因。
* Blazor WebAssembly 应用中使用的`[Authorize]`属性仅用作应用的提示，即应授权用户使应用正常工作。

```csharp
[Authorize]
[ApiController]
[Route("[controller]")]
public class WeatherForecastController : ControllerBase
{
    [HttpGet]
    public IEnumerable<WeatherForecast> Get()
    {
        ...
    }
}
```

## <a name="client-app-configuration"></a>客户端应用配置

*本节涉及解决方案的**客户端**应用。*

### <a name="authentication-package"></a>身份验证包

当创建应用以使用工作帐户或学校帐户 （）`SingleOrg`时，应用会自动接收 Microsoft[身份验证库](/azure/active-directory/develop/msal-overview)（）`Microsoft.Authentication.WebAssembly.Msal`的包引用 。 该包提供一组基元，可帮助应用对用户进行身份验证，并获取令牌来调用受保护的 API。

如果向应用添加身份验证，则手动将包添加到应用的项目文件中：

```xml
<PackageReference Include="Microsoft.Authentication.WebAssembly.Msal" 
    Version="{VERSION}" />
```

在前面`{VERSION}`的包引用中替换为`Microsoft.AspNetCore.Blazor.Templates`<xref:blazor/get-started>本文中显示的包版本。

包`Microsoft.Authentication.WebAssembly.Msal`会临时将`Microsoft.AspNetCore.Components.WebAssembly.Authentication`包添加到应用。

### <a name="authentication-service-support"></a>身份验证服务支持

使用`AddMsalAuthentication``Microsoft.Authentication.WebAssembly.Msal`包提供的扩展方法在服务容器中注册对用户进行身份验证的支持。 此方法设置应用与标识提供程序 （IP） 交互所需的所有服务。

Program.cs  :

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    var authentication = options.ProviderOptions.Authentication;
    authentication.Authority = "https://login.microsoftonline.com/{TENANT ID}";
    authentication.ClientId = "{CLIENT ID}";
    options.ProviderOptions.DefaultAccessTokenScopes.Add("{SCOPE URI}");
});
```

该方法`AddMsalAuthentication`接受回调以配置验证应用所需的参数。 注册应用时，可以从 Azure 门户 AAD 配置中获取配置应用所需的值。

### <a name="access-token-scopes"></a>访问令牌作用域

默认访问令牌作用域表示访问令牌作用域的列表，这些作用域是：

* 默认情况下包含在登录请求中。
* 用于在身份验证后立即预配访问令牌。

根据 Azure 活动目录规则，所有作用域必须属于同一应用。 根据需要为其他 API 应用添加其他作用域：

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...
    options.ProviderOptions.DefaultAccessTokenScopes.Add("{SCOPE URI}");
});
```

> [!NOTE]
> 如果 Azure 门户提供作用域 URI，并且应用在收到来自 API 的*401 未授权*响应时**引发未处理的异常**，请尝试使用不包括方案和主机的范围 URI。 例如，Azure 门户可以提供以下作用域 URI 格式之一：
>
> * `https://{ORGANIZATION}.onmicrosoft.com/{API CLIENT ID OR CUSTOM VALUE}/{SCOPE NAME}`
> * `api://{API CLIENT ID OR CUSTOM VALUE}/{SCOPE NAME}`
>
> 提供无方案和主机的范围 URI：
>
> ```csharp
> options.ProviderOptions.DefaultAccessTokenScopes.Add(
>     "{API CLIENT ID OR CUSTOM VALUE}/{SCOPE NAME}");
> ```

有关详细信息，请参阅 <xref:security/blazor/webassembly/additional-scenarios#request-additional-access-tokens>。

### <a name="imports-file"></a>导入文件

[!INCLUDE[](~/includes/blazor-security/imports-file-hosted.md)]

### <a name="index-page"></a>索引页面

[!INCLUDE[](~/includes/blazor-security/index-page-msal.md)]

### <a name="app-component"></a>应用组件

[!INCLUDE[](~/includes/blazor-security/app-component.md)]

### <a name="redirecttologin-component"></a>重定向到登录组件

[!INCLUDE[](~/includes/blazor-security/redirecttologin-component.md)]

### <a name="logindisplay-component"></a>登录显示组件

[!INCLUDE[](~/includes/blazor-security/logindisplay-component.md)]

### <a name="authentication-component"></a>身份验证组件

[!INCLUDE[](~/includes/blazor-security/authentication-component.md)]

### <a name="fetchdata-component"></a>提取数据组件

[!INCLUDE[](~/includes/blazor-security/fetchdata-component.md)]

## <a name="run-the-app"></a>运行应用

从"服务器"项目运行应用。 使用 Visual Studio 时，在**解决方案资源管理器**中选择"服务器"项目，然后选择工具栏中的 **"运行"** 按钮，或从 **"调试"** 菜单启动应用。

<!-- HOLD
[!INCLUDE[](~/includes/blazor-security/usermanager-signinmanager.md)]
-->

[!INCLUDE[](~/includes/blazor-security/troubleshoot.md)]

## <a name="additional-resources"></a>其他资源

* [请求其他访问令牌](xref:security/blazor/webassembly/additional-scenarios#request-additional-access-tokens)
* <xref:security/authentication/azure-active-directory/index>
* [Microsoft 标识平台文档](/azure/active-directory/develop/)
