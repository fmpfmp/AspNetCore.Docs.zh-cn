---
title: ASP.NET Core Web API 中的控制器操作返回类型
author: scottaddie
description: 了解在 ASP.NET Core Web API 中使用各种控制器操作方法返回类型的相关信息。
ms.author: scaddie
ms.custom: mvc
ms.date: 03/21/2018
uid: web-api/action-return-types
ms.openlocfilehash: 422db97da222fb5e742e1d8e6ae410edc90dbc18
ms.sourcegitcommit: a1afd04758e663d7062a5bfa8a0d4dca38f42afc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/20/2018
ms.locfileid: "36273551"
---
# <a name="controller-action-return-types-in-aspnet-core-web-api"></a><span data-ttu-id="9a344-103">ASP.NET Core Web API 中的控制器操作返回类型</span><span class="sxs-lookup"><span data-stu-id="9a344-103">Controller action return types in ASP.NET Core Web API</span></span>

<span data-ttu-id="9a344-104">作者：[Scott Addie](https://github.com/scottaddie)</span><span class="sxs-lookup"><span data-stu-id="9a344-104">By [Scott Addie](https://github.com/scottaddie)</span></span>

<span data-ttu-id="9a344-105">[查看或下载示例代码](https://github.com/aspnet/Docs/tree/master/aspnetcore/web-api/action-return-types/samples)（[如何下载](xref:tutorials/index#how-to-download-a-sample)）</span><span class="sxs-lookup"><span data-stu-id="9a344-105">[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/web-api/action-return-types/samples) ([how to download](xref:tutorials/index#how-to-download-a-sample))</span></span>

<span data-ttu-id="9a344-106">ASP.NET Core 提供以下 Web API 控制器操作返回类型选项：</span><span class="sxs-lookup"><span data-stu-id="9a344-106">ASP.NET Core offers the following options for Web API controller action return types:</span></span>

::: moniker range="<= aspnetcore-2.0"
* [<span data-ttu-id="9a344-107">特定类型</span><span class="sxs-lookup"><span data-stu-id="9a344-107">Specific type</span></span>](#specific-type)
* [<span data-ttu-id="9a344-108">IActionResult</span><span class="sxs-lookup"><span data-stu-id="9a344-108">IActionResult</span></span>](#iactionresult-type)
::: moniker-end
::: moniker range=">= aspnetcore-2.1"
* [<span data-ttu-id="9a344-109">特定类型</span><span class="sxs-lookup"><span data-stu-id="9a344-109">Specific type</span></span>](#specific-type)
* [<span data-ttu-id="9a344-110">IActionResult</span><span class="sxs-lookup"><span data-stu-id="9a344-110">IActionResult</span></span>](#iactionresult-type)
* [<span data-ttu-id="9a344-111">ActionResult\<T></span><span class="sxs-lookup"><span data-stu-id="9a344-111">ActionResult\<T></span></span>](#actionresultt-type)
::: moniker-end

<span data-ttu-id="9a344-112">本文档说明每种返回类型的最佳适用情况。</span><span class="sxs-lookup"><span data-stu-id="9a344-112">This document explains when it's most appropriate to use each return type.</span></span>

## <a name="specific-type"></a><span data-ttu-id="9a344-113">特定类型</span><span class="sxs-lookup"><span data-stu-id="9a344-113">Specific type</span></span>

<span data-ttu-id="9a344-114">最简单的操作返回基元或复杂数据类型（如 `string` 或自定义对象类型）。</span><span class="sxs-lookup"><span data-stu-id="9a344-114">The simplest action returns a primitive or complex data type (for example, `string` or a custom object type).</span></span> <span data-ttu-id="9a344-115">请参考以下操作，该操作返回自定义 `Product` 对象的集合：</span><span class="sxs-lookup"><span data-stu-id="9a344-115">Consider the following action, which returns a collection of custom `Product` objects:</span></span>

[!code-csharp[](../web-api/action-return-types/samples/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_Get)]

<span data-ttu-id="9a344-116">在执行操作期间无需防范已知条件，返回特定类型即可满足要求。</span><span class="sxs-lookup"><span data-stu-id="9a344-116">Without known conditions to safeguard against during action execution, returning a specific type could suffice.</span></span> <span data-ttu-id="9a344-117">上述操作不接受任何参数，因此不需要参数约束验证。</span><span class="sxs-lookup"><span data-stu-id="9a344-117">The preceding action accepts no parameters, so parameter constraints validation isn't needed.</span></span>

<span data-ttu-id="9a344-118">当在操作中需要考虑已知条件时，将引入多个返回路径。</span><span class="sxs-lookup"><span data-stu-id="9a344-118">When known conditions need to be accounted for in an action, multiple return paths are introduced.</span></span> <span data-ttu-id="9a344-119">在此类情况下，通常会将 [ActionResult](/dotnet/api/microsoft.aspnetcore.mvc.actionresult) 返回类型和基元或复杂返回类型混合。</span><span class="sxs-lookup"><span data-stu-id="9a344-119">In such a case, it's common to mix an [ActionResult](/dotnet/api/microsoft.aspnetcore.mvc.actionresult) return type with the primitive or complex return type.</span></span> <span data-ttu-id="9a344-120">要支持此类操作，必须使用 [IActionResult](#iactionresult-type) 或 [ActionResult\<T>](#actionresultt-type)。</span><span class="sxs-lookup"><span data-stu-id="9a344-120">Either [IActionResult](#iactionresult-type) or [ActionResult\<T>](#actionresultt-type) are necessary to accommodate this type of action.</span></span>

## <a name="iactionresult-type"></a><span data-ttu-id="9a344-121">IActionResult 类型</span><span class="sxs-lookup"><span data-stu-id="9a344-121">IActionResult type</span></span>

<span data-ttu-id="9a344-122">当操作中可能有多个 [ActionResult](/dotnet/api/microsoft.aspnetcore.mvc.actionresult) 返回类型时，适合使用 [IActionResult](/dotnet/api/microsoft.aspnetcore.mvc.iactionresult) 返回类型。</span><span class="sxs-lookup"><span data-stu-id="9a344-122">The [IActionResult](/dotnet/api/microsoft.aspnetcore.mvc.iactionresult) return type is appropriate when multiple [ActionResult](/dotnet/api/microsoft.aspnetcore.mvc.actionresult) return types are possible in an action.</span></span> <span data-ttu-id="9a344-123">`ActionResult` 类型表示多种 HTTP 状态代码。</span><span class="sxs-lookup"><span data-stu-id="9a344-123">The `ActionResult` types represent various HTTP status codes.</span></span> <span data-ttu-id="9a344-124">属于此类别的一些常见返回类型包括：[BadRequestResult](/dotnet/api/microsoft.aspnetcore.mvc.badrequestresult) (400)、[NotFoundResult](/dotnet/api/microsoft.aspnetcore.mvc.notfoundresult) (404) 和 [OkObjectResult](/dotnet/api/microsoft.aspnetcore.mvc.okobjectresult) (200)。</span><span class="sxs-lookup"><span data-stu-id="9a344-124">Some common return types falling into this category are [BadRequestResult](/dotnet/api/microsoft.aspnetcore.mvc.badrequestresult) (400), [NotFoundResult](/dotnet/api/microsoft.aspnetcore.mvc.notfoundresult) (404), and [OkObjectResult](/dotnet/api/microsoft.aspnetcore.mvc.okobjectresult) (200).</span></span>

<span data-ttu-id="9a344-125">由于操作中有多个返回类型和路径，因此必须自由使用 [[ProducesResponseType]](/dotnet/api/microsoft.aspnetcore.mvc.producesresponsetypeattribute.-ctor) 特性。</span><span class="sxs-lookup"><span data-stu-id="9a344-125">Because there are multiple return types and paths in the action, liberal use of the [[ProducesResponseType]](/dotnet/api/microsoft.aspnetcore.mvc.producesresponsetypeattribute.-ctor) attribute is necessary.</span></span> <span data-ttu-id="9a344-126">此特性可针对 [Swagger](/aspnet/core/tutorials/web-api-help-pages-using-swagger) 等工具生成的 API 帮助页生成更多描述性响应详细信息。</span><span class="sxs-lookup"><span data-stu-id="9a344-126">This attribute produces more descriptive response details for API help pages generated by tools like [Swagger](/aspnet/core/tutorials/web-api-help-pages-using-swagger).</span></span> <span data-ttu-id="9a344-127">`[ProducesResponseType]` 指示操作将返回的已知类型和 HTTP 状态代码。</span><span class="sxs-lookup"><span data-stu-id="9a344-127">`[ProducesResponseType]` indicates the known types and HTTP status codes to be returned by the action.</span></span>

### <a name="synchronous-action"></a><span data-ttu-id="9a344-128">同步操作</span><span class="sxs-lookup"><span data-stu-id="9a344-128">Synchronous action</span></span>

<span data-ttu-id="9a344-129">请参考以下同步操作，其中有两种可能的返回类型：</span><span class="sxs-lookup"><span data-stu-id="9a344-129">Consider the following synchronous action in which there are two possible return types:</span></span>

[!code-csharp[](../web-api/action-return-types/samples/WebApiSample.Api.Pre21/Controllers/ProductsController.cs?name=snippet_GetById&highlight=8,11)]

<span data-ttu-id="9a344-130">在上述操作中，当 `id` 代表的产品不在基础数据存储中时，则返回 404 状态代码。</span><span class="sxs-lookup"><span data-stu-id="9a344-130">In the preceding action, a 404 status code is returned when the product represented by `id` doesn't exist in the underlying data store.</span></span> <span data-ttu-id="9a344-131">调用 [NotFound](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.notfound) 帮助程序方法作为 `return new NotFoundResult();` 的快捷方式。</span><span class="sxs-lookup"><span data-stu-id="9a344-131">The [NotFound](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.notfound) helper method is invoked as a shortcut to `return new NotFoundResult();`.</span></span> <span data-ttu-id="9a344-132">如果产品确实存在，则返回代表有效负载的 `Product` 对象和状态代码 200。</span><span class="sxs-lookup"><span data-stu-id="9a344-132">If the product does exist, a `Product` object representing the payload is returned with a 200 status code.</span></span> <span data-ttu-id="9a344-133">调用 [Ok](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.ok) 帮助程序方法作为 `return new OkObjectResult(product);` 的快捷方式。</span><span class="sxs-lookup"><span data-stu-id="9a344-133">The [Ok](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.ok) helper method is invoked as the shorthand form of `return new OkObjectResult(product);`.</span></span>

### <a name="asynchronous-action"></a><span data-ttu-id="9a344-134">异步操作</span><span class="sxs-lookup"><span data-stu-id="9a344-134">Asynchronous action</span></span>

<span data-ttu-id="9a344-135">请参考以下异步操作，其中有两种可能的返回类型：</span><span class="sxs-lookup"><span data-stu-id="9a344-135">Consider the following asynchronous action in which there are two possible return types:</span></span>

[!code-csharp[](../web-api/action-return-types/samples/WebApiSample.Api.Pre21/Controllers/ProductsController.cs?name=snippet_CreateAsync&highlight=8,13)]

<span data-ttu-id="9a344-136">在上述操作中，模型验证失败时返回 400 状态代码，并且调用了 [BadRequest](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.badrequest) 帮助程序方法。</span><span class="sxs-lookup"><span data-stu-id="9a344-136">In the preceding action, a 400 status code is returned when model validation fails and the [BadRequest](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.badrequest) helper method is invoked.</span></span> <span data-ttu-id="9a344-137">例如，以下模型指示请求必须提供 `Name` 属性和值。</span><span class="sxs-lookup"><span data-stu-id="9a344-137">For example, the following model indicates that requests must provide the `Name` property and a value.</span></span> <span data-ttu-id="9a344-138">因此，若请求中未能提供合适的 `Name`，则会导致模型验证失败。</span><span class="sxs-lookup"><span data-stu-id="9a344-138">Therefore, failure to provide a proper `Name` in the request causes model validation to fail.</span></span>

[!code-csharp[](../web-api/action-return-types/samples/WebApiSample.DataAccess/Models/Product.cs?name=snippet_ProductClass&highlight=5-6)]

<span data-ttu-id="9a344-139">上述操作的其他已知返回代码是 201，由 [CreatedAtAction](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.createdataction) 帮助程序方法生成。</span><span class="sxs-lookup"><span data-stu-id="9a344-139">The preceding action's other known return code is a 201, which is generated by the [CreatedAtAction](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.createdataction) helper method.</span></span> <span data-ttu-id="9a344-140">在此路径中，返回了 `Product` 对象。</span><span class="sxs-lookup"><span data-stu-id="9a344-140">In this path, the `Product` object is returned.</span></span>

::: moniker range=">= aspnetcore-2.1"
## <a name="actionresultt-type"></a><span data-ttu-id="9a344-141">ActionResult\<T> 类型</span><span class="sxs-lookup"><span data-stu-id="9a344-141">ActionResult\<T> type</span></span>

<span data-ttu-id="9a344-142">ASP.NET Core 2.1 引入了面向 Web API 控制器操作的 [ActionResult\<T>](/dotnet/api/microsoft.aspnetcore.mvc.actionresult-1) 返回类型。</span><span class="sxs-lookup"><span data-stu-id="9a344-142">ASP.NET Core 2.1 introduces the [ActionResult\<T>](/dotnet/api/microsoft.aspnetcore.mvc.actionresult-1) return type for Web API controller actions.</span></span> <span data-ttu-id="9a344-143">它支持返回从 [ActionResult](/dotnet/api/microsoft.aspnetcore.mvc.actionresult) 派生的类型或返回[特定类型](#specific-type)。</span><span class="sxs-lookup"><span data-stu-id="9a344-143">It enables you to return a type deriving from [ActionResult](/dotnet/api/microsoft.aspnetcore.mvc.actionresult) or return a [specific type](#specific-type).</span></span> <span data-ttu-id="9a344-144">`ActionResult<T>` 通过 [IActionResult 类型](#iactionresult-type)可提供以下优势：</span><span class="sxs-lookup"><span data-stu-id="9a344-144">`ActionResult<T>` offers the following benefits over the [IActionResult type](#iactionresult-type):</span></span>

* <span data-ttu-id="9a344-145">可排除 [[ProducesResponseType]](/dotnet/api/microsoft.aspnetcore.mvc.producesresponsetypeattribute) 特性的 `Type` 属性。</span><span class="sxs-lookup"><span data-stu-id="9a344-145">The [[ProducesResponseType]](/dotnet/api/microsoft.aspnetcore.mvc.producesresponsetypeattribute) attribute's `Type` property can be excluded.</span></span>
* <span data-ttu-id="9a344-146">[隐式强制转换运算符](/dotnet/csharp/language-reference/keywords/implicit)支持将 `T` 和 `ActionResult` 均转换为 `ActionResult<T>`。</span><span class="sxs-lookup"><span data-stu-id="9a344-146">[Implicit cast operators](/dotnet/csharp/language-reference/keywords/implicit) support the conversion of both `T` and `ActionResult` to `ActionResult<T>`.</span></span> <span data-ttu-id="9a344-147">将 `T` 转换为 [ObjectResult](/dotnet/api/microsoft.aspnetcore.mvc.objectresult)，也就是将 `return new ObjectResult(T);` 简化为 `return T;`。</span><span class="sxs-lookup"><span data-stu-id="9a344-147">`T` converts to [ObjectResult](/dotnet/api/microsoft.aspnetcore.mvc.objectresult), which means `return new ObjectResult(T);` is simplified to `return T;`.</span></span>

<span data-ttu-id="9a344-148">大多数操作具有特定返回类型。</span><span class="sxs-lookup"><span data-stu-id="9a344-148">Most actions have a specific return type.</span></span> <span data-ttu-id="9a344-149">执行操作期间可能出现意外情况，不返回特定类型就是其中之一。</span><span class="sxs-lookup"><span data-stu-id="9a344-149">Unexpected conditions can occur during action execution, in which case the specific type isn't returned.</span></span> <span data-ttu-id="9a344-150">例如，操作的输入参数可能无法通过模型验证。</span><span class="sxs-lookup"><span data-stu-id="9a344-150">For example, an action's input parameter may fail model validation.</span></span> <span data-ttu-id="9a344-151">在此情况下，通常会返回相应的 `ActionResult` 类型，而不是特定类型。</span><span class="sxs-lookup"><span data-stu-id="9a344-151">In such a case, it's common to return the appropriate `ActionResult` type instead of the specific type.</span></span>

### <a name="synchronous-action"></a><span data-ttu-id="9a344-152">同步操作</span><span class="sxs-lookup"><span data-stu-id="9a344-152">Synchronous action</span></span>

<span data-ttu-id="9a344-153">请参考以下同步操作，其中有两种可能的返回类型：</span><span class="sxs-lookup"><span data-stu-id="9a344-153">Consider a synchronous action in which there are two possible return types:</span></span>

<span data-ttu-id="9a344-154">[!code-csharp[](../web-api/action-return-types/samples/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_GetById&highlight=8,11)]</span><span class="sxs-lookup"><span data-stu-id="9a344-154">[!code-csharp[](../web-api/action-return-types/samples/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_GetById&highlight=8,11)]</span></span>

<span data-ttu-id="9a344-155">在上述代码中，当产品不在数据库中时则返回状态代码 404。</span><span class="sxs-lookup"><span data-stu-id="9a344-155">In the preceding code, a 404 status code is returned when the product doesn't exist in the database.</span></span> <span data-ttu-id="9a344-156">如果产品确实存在，则返回相应的 `Product` 对象。</span><span class="sxs-lookup"><span data-stu-id="9a344-156">If the product does exist, the corresponding `Product` object is returned.</span></span> <span data-ttu-id="9a344-157">ASP.NET Core 2.1 之前，`return product;` 行是 `return Ok(product);`。</span><span class="sxs-lookup"><span data-stu-id="9a344-157">Before ASP.NET Core 2.1, the `return product;` line would have been `return Ok(product);`.</span></span>

> [!TIP]
> <span data-ttu-id="9a344-158">从 ASP.NET Core 2.1 开始，使用 `[ApiController]` 特性修饰控制器类时，将启用操作参数绑定源推理。</span><span class="sxs-lookup"><span data-stu-id="9a344-158">As of ASP.NET Core 2.1, action parameter binding source inference is enabled when a controller class is decorated with the `[ApiController]` attribute.</span></span> <span data-ttu-id="9a344-159">与路由模板中的名称相匹配的参数名称将通过请求路由数据自动绑定。</span><span class="sxs-lookup"><span data-stu-id="9a344-159">A parameter name matching a name in the route template is automatically bound using the request route data.</span></span> <span data-ttu-id="9a344-160">因此，不会使用 [[FromRoute]](/dotnet/api/microsoft.aspnetcore.mvc.fromrouteattribute) 特性对上述操作中的 `id` 参数进行显示批注。</span><span class="sxs-lookup"><span data-stu-id="9a344-160">Consequently, the preceding action's `id` parameter isn't explicitly annotated with the [[FromRoute]](/dotnet/api/microsoft.aspnetcore.mvc.fromrouteattribute) attribute.</span></span>

### <a name="asynchronous-action"></a><span data-ttu-id="9a344-161">异步操作</span><span class="sxs-lookup"><span data-stu-id="9a344-161">Asynchronous action</span></span>

<span data-ttu-id="9a344-162">请参考以下异步操作，其中有两种可能的返回类型：</span><span class="sxs-lookup"><span data-stu-id="9a344-162">Consider an asynchronous action in which there are two possible return types:</span></span>

<span data-ttu-id="9a344-163">[!code-csharp[](../web-api/action-return-types/samples/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_CreateAsync&highlight=8,13)]</span><span class="sxs-lookup"><span data-stu-id="9a344-163">[!code-csharp[](../web-api/action-return-types/samples/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_CreateAsync&highlight=8,13)]</span></span>

<span data-ttu-id="9a344-164">如果模型验证失败，则调用 [BadRequest](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.badrequest#Microsoft_AspNetCore_Mvc_ControllerBase_BadRequest_Microsoft_AspNetCore_Mvc_ModelBinding_ModelStateDictionary_) 方法以返回 400 状态代码。</span><span class="sxs-lookup"><span data-stu-id="9a344-164">If model validation fails, the [BadRequest](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.badrequest#Microsoft_AspNetCore_Mvc_ControllerBase_BadRequest_Microsoft_AspNetCore_Mvc_ModelBinding_ModelStateDictionary_) method is invoked to return a 400 status code.</span></span> <span data-ttu-id="9a344-165">向它传递包含特定验证错误的 [ModelState](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.modelstate) 属性。</span><span class="sxs-lookup"><span data-stu-id="9a344-165">The [ModelState](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.modelstate) property containing the specific validation errors is passed to it.</span></span> <span data-ttu-id="9a344-166">如果模型验证成功，则在数据库中创建产品。</span><span class="sxs-lookup"><span data-stu-id="9a344-166">If model validation succeeds, the product is created in the database.</span></span> <span data-ttu-id="9a344-167">返回 201 状态代码。</span><span class="sxs-lookup"><span data-stu-id="9a344-167">A 201 status code is returned.</span></span>

> [!TIP]
> <span data-ttu-id="9a344-168">从 ASP.NET Core 2.1 开始，使用 `[ApiController]` 特性修饰控制器类时，将启用操作参数绑定源推理。</span><span class="sxs-lookup"><span data-stu-id="9a344-168">As of ASP.NET Core 2.1, action parameter binding source inference is enabled when a controller class is decorated with the `[ApiController]` attribute.</span></span> <span data-ttu-id="9a344-169">复杂类型参数通过请求正文自动绑定。</span><span class="sxs-lookup"><span data-stu-id="9a344-169">Complex type parameters are automatically bound using the request body.</span></span> <span data-ttu-id="9a344-170">因此，不会使用 [[FromBody]](/dotnet/api/microsoft.aspnetcore.mvc.frombodyattribute) 特性对前面操作中的 `product` 参数进行显示批注。</span><span class="sxs-lookup"><span data-stu-id="9a344-170">Consequently, the preceding action's `product` parameter isn't explicitly annotated with the [[FromBody]](/dotnet/api/microsoft.aspnetcore.mvc.frombodyattribute) attribute.</span></span>
::: moniker-end

## <a name="additional-resources"></a><span data-ttu-id="9a344-171">其他资源</span><span class="sxs-lookup"><span data-stu-id="9a344-171">Additional resources</span></span>

* [<span data-ttu-id="9a344-172">控制器操作</span><span class="sxs-lookup"><span data-stu-id="9a344-172">Controller actions</span></span>](xref:mvc/controllers/actions)
* [<span data-ttu-id="9a344-173">模型验证</span><span class="sxs-lookup"><span data-stu-id="9a344-173">Model validation</span></span>](xref:mvc/models/validation)
* [<span data-ttu-id="9a344-174">使用 Swagger 的 Web API 帮助页</span><span class="sxs-lookup"><span data-stu-id="9a344-174">Web API help pages using Swagger</span></span>](xref:tutorials/web-api-help-pages-using-swagger)