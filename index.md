---
title: Develop AI-powered information extraction solutions in Azure
permalink: index.html
layout: default
---

<div class="course-intro"><p>These exercises provide hands-on experience with common developer tasks for building AI-powered information extraction solutions in Microsoft Azure.</p></div>

<div class="course-note"><strong>Note:</strong> To complete the exercises, you'll need an Azure subscription with sufficient permissions and quota to provision the required Azure resources and generative AI models. You can sign up for an <a href="https://azure.microsoft.com/free">Azure account</a>.</div>

## Exercises {#exercises}

<div class="course-grid">
  {%- assign exercises = site.pages | where_exp: "page", "page.path contains 'Instructions/Exercises/'" | sort: "path" -%}
  {%- for activity in exercises -%}
    {%- if activity.lab.title -%}
    <article class="course-card"><h3><a href="{{ activity.url | relative_url }}">{{ activity.lab.title }}</a></h3><div class="course-meta">{%- if activity.lab.level -%}<span class="course-badge">Level {{ activity.lab.level }}</span>{%- endif -%}{%- if activity.lab.duration -%}<span class="course-badge">{{ activity.lab.duration }} minutes</span>{%- endif -%}</div>{%- if activity.lab.description -%}<p>{{ activity.lab.description }}</p>{%- endif -%}</article>
    {%- endif -%}
  {%- endfor -%}
</div>

> **More learning:** Explore the [Microsoft Learn training](https://learn.microsoft.com/training/) resources for Azure AI and information extraction.
