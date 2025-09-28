# Prevent Indexing of Blazor Server Public Web Apps in Lower Environments
Most systems now include a web front end accessible to end users through the Internet. Discoverability and indexability are key characteristics that allow these applications to be found and processed by search engines such as Google or Bing.

In modern software delivery, staging (also called “lower environments”) is a best practice. These environments are replicas of production and are used to validate changes before release.

A typical setup includes three environments: test, acceptance, and production, with acceptance usually being closest to production in terms of data and configuration.

## The Issue
Public-facing web applications in lower environments are often also exposed to the Internet. As a result, search engine crawlers can access and index them. The consequence: search results may include non-production (test or acceptance) versions of your site, causing confusion and possibly exposing incomplete features.

## Solutions
The problem is simple, and so are the mitigations. According to [Google](https://developers.google.com/search/docs/crawling-indexing/robots/intro), you can instruct crawlers not to index a page by adding a `meta` tag to the `<head>` section:

```html
<meta name="robots" content="noindex">
```

Search engines also access a `robots.txt` file located at the site root to determine allowed crawl paths. To block all crawling you can use:

```
User-agent: *
Disallow: /
```

Using both approaches together is recommended in lower environments. 


## Implementation

### Block indexing using the meta tag
In the `App.razor` (root app component), conditionally emit the meta tag when not in production:

```html
<!DOCTYPE html>
<html lang="en">
@inject IWebHostEnvironment env
<head>
    <meta charset="utf-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <base href="/"/>
    <link rel="stylesheet" href="@Assets["lib/bootstrap/dist/css/bootstrap.min.css"]"/>
    <link rel="stylesheet" href="@Assets["app.css"]"/>
    <link rel="stylesheet" href="@Assets["BlazorAppPublic.styles.css"]"/>
    <ImportMap/>
    <link rel="icon" type="image/png" href="favicon.png"/>
    @if (!env.IsProduction())
    {
        <meta name="robots" content="noindex, nofollow">
    }
    <HeadOutlet/>
</head>

<body>
<Routes/>
<script src="_framework/blazor.web.js"></script>
</body>

</html>
```
- `IWebHostEnvironment` determines the current environment.
- The meta tag is rendered only outside production.

### robots.txt Dynamic Generation
Generate `robots.txt` on the fly using Minimal APIs:

```cs
 public static void ConfigureSearchEngine(this WebApplication app, IHostEnvironment env)
    {
        app.Map("/robots.txt", async context =>
        {
            context.Response.ContentType = "text/plain";

            if (env.IsProduction())
            {
                await context.Response.WriteAsync("User-agent: *\nAllow: /");
                
            }
            else
            {
                await context.Response.WriteAsync("User-agent: *\nDisallow: /");
            }
        });
    }
```

## Wrap-Up
By conditionally adding a `noindex` meta tag and dynamically serving an environment-aware `robots.txt`, we prevent lower environments from being indexed while keeping production fully discoverable.
