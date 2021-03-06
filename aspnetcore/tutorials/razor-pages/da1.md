---
title: Parte 5, atualizar as páginas geradas
author: rick-anderson
description: Parte 5 da série de tutoriais em Razor páginas.
ms.author: riande
ms.date: 09/20/2020
ms.custom: contperf-fy21q2
no-loc:
- Index
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
uid: tutorials/razor-pages/da1
ms.openlocfilehash: 46fbfb50afd03f918f9e02bcc8c1dbde9a080ca4
ms.sourcegitcommit: 6299f08aed5b7f0496001d093aae617559d73240
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97485934"
---
# <a name="part-5-update-the-generated-pages-in-an-aspnet-core-app"></a>Parte 5, atualizar as páginas geradas em um aplicativo ASP.NET Core

De [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

O aplicativo de filme gerado por scaffolding tem um bom começo, mas a apresentação não é ideal. **Liberado** deve ser de duas palavras, **data de lançamento**.

![Aplicativo de filme aberto no Chrome](sql/_static/5/m55.png)

## <a name="update-the-generated-code"></a>Atualizar o código gerado

Abra o arquivo *Models/Movie.cs* e adicione as linhas realçadas mostradas no seguinte código:

[!code-csharp[Main](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Models/MovieDateFixed.cs?name=snippet_1&highlight=3,12,17)]

No código anterior:

* A anotação de dados `[Column(TypeName = "decimal(18, 2)")]` permite que o Entity Framework Core mapeie o `Price` corretamente para a moeda no banco de dados. Para obter mais informações, veja [Tipos de Dados](/ef/core/modeling/relational/data-types).
* O atributo [[DISPLAY]](xref:Microsoft.AspNetCore.Mvc.ModelBinding.Metadata.DisplayMetadata) especifica o nome de exibição de um campo. No código anterior, "data de lançamento" em vez de "liberado".
* O atributo [[DataType]](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) especifica o tipo dos dados ( `Date` ). As informações de hora armazenadas no campo não são exibidas.

[DataAnnotations](/aspnet/mvc/overview/older-versions/mvc-music-store/mvc-music-store-part-6) é abordado no próximo tutorial.

Navegue até *páginas/filmes* e passe o mouse sobre um link de **edição** para ver a URL de destino.

![São mostrados uma janela do navegador com o mouse sobre o link de edição e um link da URL https://localhost:1234/Movies/Edit/5](~/tutorials/razor-pages/da1/edit7.png)

Os links **Editar**, **detalhes** e **excluir** são gerados pelo auxiliar de [marca de âncora](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper) no arquivo *pages/Movies/ Index . cshtml* .

[!code-cshtml[](~/tutorials/razor-pages/razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Index.cshtml?highlight=16-18&range=32-)]

Os [auxiliares de marca](xref:mvc/views/tag-helpers/intro) permitem que o código do servidor participe da criação e renderização de elementos HTML em Razor arquivos.

No código anterior, o [auxiliar de marca de âncora](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper) gera dinamicamente o `href` valor do atributo HTML da Razor página (a rota é relativa), o `asp-page` e o identificador de rota ( `asp-route-id` ). Para obter mais informações, consulte [geração de URL para páginas](xref:razor-pages/index#url-generation-for-pages).

Use o **modo de exibição de origem** de um navegador para examinar a marcação gerada. Uma parte do HTML gerado é mostrada abaixo:

```html
<td>
  <a href="/Movies/Edit?id=1">Edit</a> |
  <a href="/Movies/Details?id=1">Details</a> |
  <a href="/Movies/Delete?id=1">Delete</a>
</td>
```

   Os links gerados dinamicamente passam a ID do filme com uma cadeia de caracteres de consulta. Por exemplo, o `?id=1` no `https://localhost:5001/Movies/Details?id=1` .

### <a name="add-route-template"></a>Adicionar modelo de rota

Atualize as páginas editar, detalhes e excluir Razor para usar o `{id:int}` modelo de rota. Altere a diretiva de página de cada uma dessas páginas de `@page` para `@page "{id:int}"`. Execute o aplicativo e, em seguida, exiba o código-fonte.

O HTML gerado adiciona a ID à parte do caminho da URL:

```html
<td>
  <a href="/Movies/Edit/1">Edit</a> |
  <a href="/Movies/Details/1">Details</a> |
  <a href="/Movies/Delete/1">Delete</a>
</td>
```

Uma solicitação para a página com o `{id:int}` modelo de rota que **não** inclui o inteiro retornará um erro http 404 (não encontrado). Por exemplo, `https://localhost:5001/Movies/Details` retornará um erro 404. Para tornar a ID opcional, acrescente `?` à restrição de rota:

```cshtml
@page "{id:int?}"
```

Testar o comportamento de `@page "{id:int?}"` :

1. defina a diretiva de página em *Pages/Movies/Details.cshtml* como `@page "{id:int?}"`.
1. Defina um ponto de interrupção no `public async Task<IActionResult> OnGetAsync(int? id)` , em *pages/Movies/details. cshtml. cs*.
1. Navegue até `https://localhost:5001/Movies/Details/`.

Com a diretiva `@page "{id:int}"`, o ponto de interrupção nunca é atingido. O mecanismo de roteamento retorna HTTP 404. Usando `@page "{id:int?}"` , o `OnGetAsync` método retorna `NotFound` (http 404):

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie50/Pages/Movies/Details.cshtml.cs?name=snippet1&highlight=10-13)]

### <a name="review-concurrency-exception-handling"></a>Examinar o tratamento de exceção de simultaneidade

Examine o método `OnPostAsync` no arquivo *Pages/Movies/Edit.cshtml.cs*:

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Edit.cshtml.cs?name=snippet)]

O código anterior detecta exceções de simultaneidade quando um cliente exclui o filme e o outro cliente posta alterações no filme.

Para testar o bloco `catch`:

1. Defina um ponto de interrupção em `catch (DbUpdateConcurrencyException)` .
1. Selecione **Editar** para um filme, faça alterações, mas não insira **Salvar**.
1. Em outra janela do navegador, selecione o link **Excluir** do mesmo filme e, em seguida, exclua o filme.
1. Na janela do navegador anterior, poste as alterações no filme.

O código de produção talvez deseje detectar conflitos de simultaneidade. Confira [Lidar com conflitos de simultaneidade](xref:data/ef-rp/concurrency) para obter mais informações.

### <a name="posting-and-binding-review"></a>Análise de postagem e associação

Examine o arquivo *Pages/Movies/Edit.cshtml.cs*: 

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/SnapShots/Edit.cshtml.cs?name=snippet2)]

Quando uma solicitação HTTP GET é feita na página filmes/editar, por exemplo `https://localhost:5001/Movies/Edit/3` :

* O método `OnGetAsync` busca o filme do banco de dados e retorna o método `Page`.
* O `Page` método renderiza a página *pages/Movies/Edit. cshtml* Razor . O arquivo *pages/Movies/Edit. cshtml* contém a diretiva Model `@model RazorPagesMovie.Pages.Movies.EditModel` , que torna o modelo de filme disponível na página.
* O formulário Editar é exibido com os valores do filme.

Quando a página Movies/Edit é postada:

* Os valores de formulário na página são associados à propriedade `Movie`. O atributo `[BindProperty]` habilita o [Model binding](xref:mvc/models/model-binding).

  ```csharp
  [BindProperty]
  public Movie Movie { get; set; }
  ```

* Se houver erros no estado do modelo, por exemplo, `ReleaseDate` não puder ser convertido em uma data, o formulário será exibido novamente com os valores enviados.
* Se não houver erros do modelo, o filme será salvo.

Os métodos GET HTTP nas Index páginas, criar e excluir Razor seguem um padrão semelhante. O método HTTP POST `OnPostAsync` na página Criar Razor segue um padrão semelhante ao `OnPostAsync` método na Razor página Editar.

## <a name="additional-resources"></a>Recursos adicionais

> [!div class="step-by-step"]
> [Anterior: trabalhar com um banco de dados](xref:tutorials/razor-pages/sql) 
>  [Próximo: Adicionar pesquisa](xref:tutorials/razor-pages/search)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

O aplicativo de filme gerado por scaffolding tem um bom começo, mas a apresentação não é ideal. **Liberado** deve ser de duas palavras, **data de lançamento**.

![Aplicativo de filme aberto no Chrome](sql/_static/m55https.png)

## <a name="update-the-generated-code"></a>Atualizar o código gerado

Abra o arquivo *Models/Movie.cs* e adicione as linhas realçadas mostradas no seguinte código:

[!code-csharp[Main](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/MovieDateFixed.cs?name=snippet_1&highlight=3,12,17)]

A anotação de dados `[Column(TypeName = "decimal(18, 2)")]` permite que o Entity Framework Core mapeie o `Price` corretamente para a moeda no banco de dados. Para obter mais informações, veja [Tipos de Dados](/ef/core/modeling/relational/data-types).

[DataAnnotations](/aspnet/mvc/overview/older-versions/mvc-music-store/mvc-music-store-part-6) é abordado no próximo tutorial. O atributo [[DISPLAY]](xref:Microsoft.AspNetCore.Mvc.ModelBinding.Metadata.DisplayMetadata) especifica o que exibir para o nome de um campo, neste caso, "data de lançamento" em vez de "liberado". O atributo [DataType](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) especifica o tipo dos dados ( `Date` ), portanto, as informações de hora armazenadas no campo não são exibidas.

Procure Pages/Movies e focalize um link **Editar** para ver a URL de destino.

![São mostrados uma janela do navegador com o mouse sobre o link de edição e um link da URL http://localhost:1234/Movies/Edit/5](~/tutorials/razor-pages/da1/edit7.png)

Os links **Editar**, **detalhes** e **excluir** são gerados pelo auxiliar de [marca de âncora](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper) no arquivo *pages/Movies/ Index . cshtml* .

[!code-cshtml[](~/tutorials/razor-pages/razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Index.cshtml?highlight=16-18&range=32-)]

Os [auxiliares de marca](xref:mvc/views/tag-helpers/intro) permitem que o código do servidor participe da criação e renderização de elementos HTML em Razor arquivos. No código anterior, o `AnchorTagHelper` gera dinamicamente o valor do `href` atributo HTML da Razor página (a rota é relativa), a `asp-page` e a ID da rota ( `asp-route-id` ). Consulte [Geração de URL para Páginas](xref:razor-pages/index#url-generation-for-pages) para obter mais informações.

Use o **modo de exibição de origem** de um navegador para examinar a marcação gerada. Uma parte do HTML gerado é mostrada abaixo:

```html
<td>
  <a href="/Movies/Edit?id=1">Edit</a> |
  <a href="/Movies/Details?id=1">Details</a> |
  <a href="/Movies/Delete?id=1">Delete</a>
</td>
```

Os links gerados dinamicamente passam a ID do filme com uma cadeia de caracteres de consulta. Por exemplo, o `?id=1` no  `https://localhost:5001/Movies/Details?id=1` .

Atualize as páginas editar, detalhes e excluir Razor para usar o modelo de rota "{ID: int}". Altere a diretiva de página de cada uma dessas páginas de `@page` para `@page "{id:int}"`. Execute o aplicativo e, em seguida, exiba o código-fonte. O HTML gerado adiciona a ID à parte do caminho da URL:

```html
<td>
  <a href="/Movies/Edit/1">Edit</a> |
  <a href="/Movies/Details/1">Details</a> |
  <a href="/Movies/Delete/1">Delete</a>
</td>
```

Uma solicitação para a página com o modelo de rota “{id:int}” que **não** inclui o inteiro retornará um erro HTTP 404 (não encontrado). Por exemplo, `https://localhost:5001/Movies/Details` retornará um erro 404. Para tornar a ID opcional, acrescente `?` à restrição de rota:

 ```cshtml
@page "{id:int?}"
```

Para testar o comportamento de `@page "{id:int?}"`:

* defina a diretiva de página em *Pages/Movies/Details.cshtml* como `@page "{id:int?}"`.
* Defina um ponto de interrupção no `public async Task<IActionResult> OnGetAsync(int? id)` , em *pages/Movies/details. cshtml. cs*.
* Navegue até `https://localhost:5001/Movies/Details/`.

Com a diretiva `@page "{id:int}"`, o ponto de interrupção nunca é atingido. O mecanismo de roteamento retorna HTTP 404. Usando `@page "{id:int?}"` , o `OnGetAsync` método retorna `NotFound` (http 404):

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Pages/Movies/Details.cshtml.cs?name=snippet1&highlight=10-13)]

### <a name="review-concurrency-exception-handling"></a>Examinar o tratamento de exceção de simultaneidade

Examine o método `OnPostAsync` no arquivo *Pages/Movies/Edit.cshtml.cs*:

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Edit.cshtml.cs?name=snippet)]

O código anterior detecta exceções de simultaneidade quando um cliente exclui o filme e o outro cliente posta alterações no filme.

Para testar o bloco `catch`:

* Definir um ponto de interrupção em `catch (DbUpdateConcurrencyException)`
* Selecione **Editar** para um filme, faça alterações, mas não insira **Salvar**.
* Em outra janela do navegador, selecione o link **Excluir** do mesmo filme e, em seguida, exclua o filme.
* Na janela do navegador anterior, poste as alterações no filme.

O código de produção talvez deseje detectar conflitos de simultaneidade. Confira [Lidar com conflitos de simultaneidade](xref:data/ef-rp/concurrency) para obter mais informações.

### <a name="posting-and-binding-review"></a>Análise de postagem e associação

Examine o arquivo *Pages/Movies/Edit.cshtml.cs*: 

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/snapshot_sample/RazorPagesMovie/Pages/Movies/Edit21.cshtml.cs?name=snippet2)]

Quando uma solicitação HTTP GET é feita na página filmes/editar, por exemplo `https://localhost:5001/Movies/Edit/3` :

* O método `OnGetAsync` busca o filme do banco de dados e retorna o método `Page`. 
* O `Page` método renderiza a página *pages/Movies/Edit. cshtml* Razor . O arquivo *pages/Movies/Edit. cshtml* contém a diretiva Model `@model RazorPagesMovie.Pages.Movies.EditModel` , que torna o modelo de filme disponível na página.
* O formulário Editar é exibido com os valores do filme.

Quando a página Movies/Edit é postada:

* Os valores de formulário na página são associados à propriedade `Movie`. O atributo `[BindProperty]` habilita o [Model binding](xref:mvc/models/model-binding).

  ```csharp
  [BindProperty]
  public Movie Movie { get; set; }
  ```

* Se houver erros no estado do modelo, por exemplo, `ReleaseDate` não puder ser convertido em uma data, o formulário será exibido com os valores enviados.
* Se não houver erros do modelo, o filme será salvo.

Os métodos GET HTTP nas Index páginas, criar e excluir Razor seguem um padrão semelhante. O método HTTP POST `OnPostAsync` na página Criar Razor segue um padrão semelhante ao `OnPostAsync` método na Razor página Editar.

A pesquisa é adicionada no próximo tutorial.

## <a name="additional-resources"></a>Recursos adicionais

* [Versão do YouTube deste tutorial](https://youtu.be/yLnnleREMtQ)

> [!div class="step-by-step"]
> [Anterior: trabalhar com um banco de dados](xref:tutorials/razor-pages/sql) 
>  [Próximo: Adicionar pesquisa](xref:tutorials/razor-pages/search)

::: moniker-end
