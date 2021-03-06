---
title: Hospedar e implantar ASP.NET Core Blazor
author: guardrex
description: Descubra como hospedar e implantar Blazor aplicativos.
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/15/2020
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
uid: blazor/host-and-deploy/index
ms.openlocfilehash: a23bee120611ee603305a88dabac76566481fa4a
ms.sourcegitcommit: 6299f08aed5b7f0496001d093aae617559d73240
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97485882"
---
# <a name="host-and-deploy-aspnet-core-no-locblazor"></a>Hospedar e implantar ASP.NET Core Blazor

Por [Luke Latham](https://github.com/guardrex), [Rainer Stropek](https://www.timecockpit.com) e [Daniel Roth](https://github.com/danroth27)

## <a name="publish-the-app"></a>Publicar o aplicativo

Os aplicativos são publicados para implantação na configuração de versão.

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. Selecione **Compilar**  >  **publicar {Application}** na barra de navegação.
1. Selecione o botão *destino de publicação*. Para publicar localmente, selecione **Pasta**.
1. Aceite o local padrão no campo **Escolher uma pasta** ou especifique um local diferente. Selecione o botão **`Publish`**.

# <a name="visual-studio-for-mac"></a>[Visual Studio para Mac](#tab/visual-studio-mac)

1. Selecione **criar**  >  **publicar na pasta**.
1. Confirme a pasta para receber os ativos publicados e selecione **`Publish`** .

# <a name="net-core-cli"></a>[CLI do .NET Core](#tab/netcore-cli)

Use o [`dotnet publish`](/dotnet/core/tools/dotnet-publish) comando para publicar o aplicativo com uma configuração de versão:

```dotnetcli
dotnet publish -c Release
```

---

Publicar o aplicativo dispara uma [restauração](/dotnet/core/tools/dotnet-restore) das dependências do projeto e [compila](/dotnet/core/tools/dotnet-build) o projeto antes de criar os ativos para implantação. Como parte do processo de build, os assemblies e métodos não usados são removidos para reduzir o tamanho de download do aplicativo e os tempos de carregamento.

Locais de publicação:

* Blazor WebAssembly
  * Autônomo: o aplicativo é publicado na `/bin/Release/{TARGET FRAMEWORK}/publish/wwwroot` pasta. Para implantar o aplicativo como um site estático, copie o conteúdo da `wwwroot` pasta para o host do site estático.
  * Hospedado: o Blazor WebAssembly aplicativo cliente é publicado na `/bin/Release/{TARGET FRAMEWORK}/publish/wwwroot` pasta do aplicativo de servidor, juntamente com quaisquer outros ativos da Web estáticos do aplicativo de servidor. Implante o conteúdo da `publish` pasta no host.
* Blazor Server: O aplicativo é publicado na `/bin/Release/{TARGET FRAMEWORK}/publish` pasta. Implante o conteúdo da `publish` pasta no host.

Os ativos na pasta são implantados no servidor Web. A implantação pode ser um processo manual ou automatizado, dependendo das ferramentas de desenvolvimento em uso.

## <a name="app-base-path"></a>Caminho base do aplicativo

O *caminho base do aplicativo* é o caminho da URL raiz do aplicativo. Considere o seguinte aplicativo ASP.NET Core e o Blazor subaplicativo:

* O aplicativo ASP.NET Core é denominado `MyApp` :
  * O aplicativo reside fisicamente em `d:/MyApp` .
  * As solicitações são recebidas em `https://www.contoso.com/{MYAPP RESOURCE}` .
* Um Blazor aplicativo chamado `CoolApp` é um subaplicativo de `MyApp` :
  * O subaplicativo reside fisicamente em `d:/MyApp/CoolApp` .
  * As solicitações são recebidas em `https://www.contoso.com/CoolApp/{COOLAPP RESOURCE}` .

Sem especificar a configuração adicional para `CoolApp` o, o subaplicativo nesse cenário não tem conhecimento de onde ele reside no servidor. Por exemplo, o aplicativo não pode construir URLs relativas corretas para seus recursos sem saber que ela reside no caminho de URL relativo `/CoolApp/` .

Para fornecer a configuração para o Blazor caminho base do aplicativo `https://www.contoso.com/CoolApp/` , o `<base>` atributo da marca `href` é definido como o caminho raiz relativo no `wwwroot/index.html` arquivo ( Blazor WebAssembly ) ou `Pages/_Host.cshtml` arquivo ( Blazor Server ).

Blazor WebAssembly (`wwwroot/index.html`):

```html
<base href="/CoolApp/">
```

Blazor Server (`Pages/_Host.cshtml`):

```html
<base href="~/CoolApp/">
```

Blazor Server os aplicativos também definem o caminho base do servidor chamando <xref:Microsoft.AspNetCore.Builder.UsePathBaseExtensions.UsePathBase*> no pipeline de solicitação do aplicativo de `Startup.Configure` :

```csharp
app.UsePathBase("/CoolApp");
```

Ao fornecer o caminho de URL relativo, um componente que não está no diretório raiz pode construir URLs relativas ao caminho raiz do aplicativo. Os componentes em diferentes níveis da estrutura de diretório podem criar links para outros recursos em locais em todo o aplicativo. O caminho base do aplicativo também é usado para interceptar hiperlinks selecionados onde o `href` destino do link está dentro do espaço de URI do caminho base do aplicativo. O Blazor roteador manipula a navegação interna.

Em muitos cenários de hospedagem, o caminho de URL relativo para o aplicativo é a raiz do aplicativo. Nesses casos, o caminho base da URL relativa do aplicativo é uma barra ( `<base href="/" />` ), que é a configuração padrão para um Blazor aplicativo. Em outros cenários de hospedagem, como páginas do GitHub e subaplicativos do IIS, o caminho base do aplicativo deve ser definido como o caminho de URL relativo do servidor do aplicativo.

Para definir o caminho base do aplicativo, atualize a `<base>` marca dentro dos `<head>` elementos de marca do `Pages/_Host.cshtml` arquivo ( Blazor Server ) ou `wwwroot/index.html` arquivo ( Blazor WebAssembly ). Defina o `href` valor do atributo como `/{RELATIVE URL PATH}/` ( Blazor WebAssembly ) ou `~/{RELATIVE URL PATH}/` ( Blazor Server ). **A barra à direita é necessária.** O espaço reservado `{RELATIVE URL PATH}` é o caminho de URL relativo completo do aplicativo.

Para um Blazor WebAssembly aplicativo com um caminho de URL não raiz relativo (por exemplo, `<base href="/CoolApp/">` ), o aplicativo não consegue localizar seus recursos *quando executado localmente*. Para superar esse problema durante o desenvolvimento e os testes locais, você pode fornecer um argumento *base de caminho* que corresponde ao valor de `href` da tag `<base>` no runtime. **Não inclua uma barra à direita.** Para passar o argumento de base Path ao executar o aplicativo localmente, execute o `dotnet run` comando no diretório do aplicativo com a `--pathbase` opção:

```dotnetcli
dotnet run --pathbase=/{RELATIVE URL PATH (no trailing slash)}
```

Para um Blazor WebAssembly aplicativo com um caminho de URL relativo de `/CoolApp/` ( `<base href="/CoolApp/">` ), o comando é:

```dotnetcli
dotnet run --pathbase=/CoolApp
```

O Blazor WebAssembly aplicativo responde localmente em `http://localhost:port/CoolApp` .

**Blazor Server`MapFallbackToPage`configuração do**

Passe o seguinte caminho para <xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapFallbackToPage%2A> em `Startup.Configure` :

```csharp
endpoints.MapFallbackToPage("/{RELATIVE PATH}/{**path:nonfile}");
```

O espaço reservado `{RELATIVE PATH}` é o caminho não raiz no servidor. Por exemplo, `CoolApp` é o segmento de espaço reservado se a URL não raiz para o aplicativo `https://{HOST}:{PORT}/CoolApp/` for):

```csharp
endpoints.MapFallbackToPage("/CoolApp/{**path:nonfile}");
```

**Hospedar vários Blazor WebAssembly aplicativos**

Para obter mais informações sobre como hospedar vários Blazor WebAssembly aplicativos em uma solução hospedada Blazor , consulte <xref:blazor/host-and-deploy/webassembly#hosted-deployment-with-multiple-blazor-webassembly-apps> .

## <a name="deployment"></a>Implantação

Confira orientações de implantação nos tópicos a seguir:

* <xref:blazor/host-and-deploy/webassembly>
* <xref:blazor/host-and-deploy/server>
