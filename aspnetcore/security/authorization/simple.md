---
title: Autorização simples no ASP.NET Core
author: rick-anderson
description: Saiba como usar o atributo Authorize para restringir o acesso a ASP.NET Core de controladores e ações.
ms.author: riande
ms.date: 10/14/2016
no-loc:
- appsettings.json
- ASP.NET Core Identity
- cookie
- Cookie
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: security/authorization/simple
ms.openlocfilehash: ae8fb47e58924d559f1c2c4ed7c9545c37141209
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93061334"
---
# <a name="simple-authorization-in-aspnet-core"></a>Autorização simples no ASP.NET Core

<a name="security-authorization-simple"></a>

A autorização no ASP.NET Core é controlada com <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute> e seus vários parâmetros. Em sua forma mais simples, aplicar o `[Authorize]` atributo a um controlador, uma ação ou uma Razor página, limita o acesso a esse componente a qualquer usuário autenticado.

Por exemplo, o código a seguir limita o acesso ao `AccountController` para qualquer usuário autenticado.

```csharp
[Authorize]
public class AccountController : Controller
{
    public ActionResult Login()
    {
    }

    public ActionResult Logout()
    {
    }
}
```

Se você quiser aplicar a autorização a uma ação em vez de ao controlador, aplique o `AuthorizeAttribute` atributo à própria ação:

```csharp
public class AccountController : Controller
{
   public ActionResult Login()
   {
   }

   [Authorize]
   public ActionResult Logout()
   {
   }
}
```

Agora, somente usuários autenticados podem acessar a `Logout` função.

Você também pode usar o `AllowAnonymous` atributo para permitir o acesso por usuários não autenticados a ações individuais. Por exemplo:

```csharp
[Authorize]
public class AccountController : Controller
{
    [AllowAnonymous]
    public ActionResult Login()
    {
    }

    public ActionResult Logout()
    {
    }
}
```

Isso permitiria apenas usuários autenticados para o `AccountController` , exceto para a `Login` ação, que é acessível por todos, independentemente de seu status autenticado ou não autenticado/anônimo.

> [!WARNING]
> `[AllowAnonymous]` ignora todas as instruções de autorização. Se você combinar `[AllowAnonymous]` e qualquer `[Authorize]` atributo, os `[Authorize]` atributos serão ignorados. Por exemplo, se você aplicar `[AllowAnonymous]` no nível do controlador, todos os `[Authorize]` atributos no mesmo controlador (ou em qualquer ação dentro dele) serão ignorados.

[!INCLUDE[](~/includes/requireAuth.md)]

<a name="aarp"></a>

## <a name="authorize-attribute-and-no-locrazor-pages"></a>Autorizar atributo e Razor páginas

O <xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute> pode * **não** ser aplicado a Razor manipuladores de página. Por exemplo, `[Authorize]` não pode ser aplicado a `OnGet` , `OnPost` ou a qualquer outro manipulador de página. Considere o uso de um controlador MVC ASP.NET Core para páginas com diferentes requisitos de autorização para diferentes manipuladores.

As duas abordagens a seguir podem ser usadas para aplicar a autorização aos Razor métodos do manipulador de página:

_ Use páginas separadas para manipuladores de página que exigem autorização diferente. Moveu o conteúdo compartilhado em uma ou mais [exibições parciais](xref:mvc/views/partial). Quando possível, essa é a abordagem recomendada.
* Para o conteúdo que deve compartilhar uma página comum, escreva um filtro que execute a autorização como parte do [IAsyncPageFilter. OnPageHandlerSelectionAsync](xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncPageFilter.OnPageHandlerSelectionAsync%2A). O projeto GitHub [PageHandlerAuth](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/simple/samples/3.1/PageHandlerAuth) demonstra essa abordagem:
  * O [AuthorizeIndexPageHandlerFilter](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/security/authorization/simple/samples/3.1/PageHandlerAuth/AuthorizeIndexPageHandlerFilter.cs) implementa o filtro de autorização: [!code-csharp[](~/security/authorization/simple/samples/3.1/PageHandlerAuth/Pages/Index.cshtml.cs?name=snippet)]

  * O atributo [[AuthorizePageHandler]](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/simple/samples/3.1/PageHandlerAuth/Pages/Index.cshtml.cs#L16) é aplicado ao `OnGet` manipulador de página: [!code-csharp[](~/security/authorization/simple/samples/3.1/PageHandlerAuth/AuthorizeIndexPageHandlerFilter.cs?name=snippet)]

> [!WARNING]
> A abordagem de exemplo [PageHandlerAuth](https://github.com/pranavkm/PageHandlerAuth) não **é * _: _** compor com atributos de autorização aplicados à página, ao modelo de página ou globalmente. A composição de atributos de autorização resulta em autenticação e autorização em execução várias vezes quando você tem mais `AuthorizeAttribute` ou `AuthorizeFilter` instâncias também aplicadas à página.
> * Trabalhe em conjunto com o restante de ASP.NET Core sistema de autenticação e autorização. Você deve verificar se o uso dessa abordagem funciona corretamente para seu aplicativo.

Não há planos para dar suporte aos `AuthorizeAttribute` Razor manipuladores de página. 
