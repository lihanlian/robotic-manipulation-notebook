---
title: ACT
parent: Imitation Learning
nav_order: 3
---

# ACT
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Just the Docs uses [lunr.js](https://lunrjs.com) to add a client-side search interface powered by a JSON index that Jekyll generates.
All search results are shown in an auto-complete style interface (there is no search results page).
By default, all generated HTML pages are indexed using the following data points:

- Page title
- Page content
- Page URL

## Note

kl divergence, CVAE test time style variable.

### Search granularity

Pages are split into sections that can be searched individually.
The sections are defined by the headings on the page.
Each section is displayed in a separate search result.

```yaml
Current qpos + demonstrated action chunk
                    ↓
              CVAE encoder
                    ↓
               mu, logvar
                    ↓
              sample latent z
                    ↓
Images + current qpos + z → action decoder → a_hat
```

## Hiding pages from search

Sometimes you might have a page that you don't want to be indexed for the search nor to show up in search results, e.g., a 404 page.
To exclude a page from search, add the `search_exclude: true` parameter to the page's YAML front matter:

{: .no_toc }

```yaml
---
layout: default
title: Page not found
nav_exclude: true
search_exclude: true
---

```

## Generate search index when used as a gem

If you use Just the Docs as a remote theme, you do not need the following steps.

If you use the theme as a gem, you must initialize the search by running this `rake` command that comes with `just-the-docs`:

```bash
$ bundle exec just-the-docs rake search:init
```

This command creates the `assets/js/zzzz-search-data.json` file that Jekyll uses to create your search index.
Alternatively, you can create the file manually with [this content]({{ site.github.repository_url }}/blob/main/assets/js/zzzz-search-data.json).

## Custom content for search index
{: .d-inline-block }

New (v0.4.0)
{: .label .label-green }

Advanced
{: .label .label-yellow }

By default, the search feature indexes a page's `.content`, `.title`, and *some* headers within the `.content`. Other data (e.g. front matter, files in `_data` and `assets`) is not indexed. Users can customize what is indexed.

{: .warning }
> Customizing search indices is an advanced feature that requires Javascript and Liquid knowledge.

1. When Just the Docs is a local or gem theme, ensure `assets/js/zzzz-search-data.json` is up-to-date with [Generate search index when used as a gem](#generate-search-index-when-used-as-a-gem).
2. Add a new file named `_includes/lunr/custom-data.json`. Insert custom Liquid code that reads your data (e.g. the page object at `include.page`) then generates custom Javascript fields that hold the custom data you want to index. Verify these fields in the generated `assets/js/search-data.json`.
3. Add a new file named `_includes/lunr/custom-index.js`. Insert custom Javascript code that reads your custom Javascript fields and inserts them into the search index. You may want to inspect `assets/js/just-the-docs.js` to better understand the code.

### Example: adding custom `usage` and `examples` fields
{: .text-delta }

This example adds front matter `usage` and `examples` fields to the search index.

`_includes/lunr/custom-data.json` custom code reads the page `usage` and `examples` fields, normalizes the text, and writes the text to custom Javascript `myusage` and `myexamples` fields. Javascript fields are similar yet [not the same as JSON](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON#javascript_and_json_differences). `jsonify` will probably work for most scenarios.

{% raw %}
```liquid
{%- capture newline %}
{% endcapture -%}
"myusage": {{ include.page.usage | markdownify | replace:newline,' ' | strip_html | normalize_whitespace | strip | jsonify }},
"myexamples": {{ include.page.examples | markdownify | replace:newline,' ' | strip_html | normalize_whitespace | strip | jsonify }},
```
{% endraw %}

`_includes/lunr/custom-index.js` custom code is inserted into the Javascript loop of `assets/js/just-the-docs.js`. All custom Javascript fields are accessed as fields of `docs[i]` such as `docs[i].myusage`. Finally, append your custom fields on to the already existing `docs[i].content`.

```javascript
const content_to_merge = [docs[i].content, docs[i].myusage, docs[i].myexamples];
docs[i].content = content_to_merge.join(' ');
```

### Reference
1. <i class="fab fa-youtube"></i> [Variational Autoencoder - Model, ELBO, loss function and maths explained easily!](https://www.youtube.com/watch?v=iwEzwTTalbg)

2. <i class="fab fa-youtube"></i> [(Conditional) Variational Autoencoder](https://www.youtube.com/watch?v=liXZwc32Tg8) 


 
