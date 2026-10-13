---
title: API explorer
linkTitle: API explorer
weight: 20
description: Swagger UI over fortuna-api's OpenAPI document — every endpoint, its schema, and its authorization.
---

Every endpoint the API exposes, rendered from
[`docs/openapi/fortuna.json`](https://github.com/artur-rios/fortuna-api/blob/main/docs/openapi/fortuna.json)
on fortuna-api's `main` branch — read live, so this page always shows the
released document. If the explorer below stays empty, open
[the raw document](https://raw.githubusercontent.com/artur-rios/fortuna-api/main/docs/openapi/fortuna.json)
directly.

{{% alert title="This explorer does not call anything" color="info" %}}
There is no server behind this page, so **Try it out** and **Authorize** are
removed. To call the endpoints, run the API (see [Installation](../../installation/))
and use its own Swagger UI. For what each endpoint means and who may call it,
see the [API reference](../).
{{% /alert %}}

{{% alert title="The explorer cannot show who may call what" color="warning" %}}
The OpenAPI document marks no operation as anonymous or as secured, so the
explorer's padlocks cannot tell a public route from a protected one. The anonymous routes are listed on the
[API reference]({{< relref "/docs/api#anonymous-routes" >}}), checked against the code.
{{% /alert %}}

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/swagger-ui-dist@5.32.13/swagger-ui.css" integrity="sha384-tRpWwikYYdk1+1Mu0osh0Tz/Ay5xgS+s/Nf2Aa7GVAFtZLFdJlAbozfrq4g+xHBK" crossorigin="anonymous">

<div id="swagger-ui" class="fortuna-swagger"></div>

<script src="https://cdn.jsdelivr.net/npm/swagger-ui-dist@5.32.13/swagger-ui-bundle.js" integrity="sha384-PsJla434CobCNv3y1K4wRavOqkUAvwGEQEfbUmI98CCqqGCJsmuDsgIjM6ZQQODP" crossorigin="anonymous"></script>
<script>
  // Width, dark mode and the inert padlocks: assets/scss/_styles_project.scss, .fortuna-swagger.
  window.addEventListener('load', function () {
    window.SwaggerUIBundle({
      url: 'https://raw.githubusercontent.com/artur-rios/fortuna-api/main/docs/openapi/fortuna.json',
      dom_id: '#swagger-ui',
      supportedSubmitMethods: [],
      docExpansion: 'none',
      defaultModelsExpandDepth: 0,
      deepLinking: true,
      filter: true,
      tryItOutEnabled: false,
      presets: [window.SwaggerUIBundle.presets.apis],
      plugins: [{ wrapComponents: { authorizeBtn: function () { return function () { return null; }; } } }],
      layout: 'BaseLayout'
    });
  });
</script>
