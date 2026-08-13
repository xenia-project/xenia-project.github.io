---
layout: page
title: Download
icon: file_download
permalink: /download/
---

<h2>Downloads</h2>

Before downloading any version of Xenia below, read the [Quickstart](https://github.com/xenia-canary/xenia-canary/wiki/Quickstart) guide.

{% for platform in site.artifacts %}
  {% if platform.icon == "windows" %}
{{ platform.title }} branches:
  {% else %}
{{ platform.title }} branches:
  {% endif %}
  {% for branch in platform.branches %}
    {% if branch.download_url %}
* [{{ branch.title}}]({{ branch.download_url }})
    {% endif %}
  {% endfor %}
{% endfor %}

[System Requirements](https://github.com/xenia-project/xenia/wiki/Quickstart#system-requirements)

<div>
    <p>MacOS and ARM64 support are currently under development for xenia_edge.</p>

    <p style="background-color:yellow;">Xenia UWP (Xenia on Xbox Series S/X) is not made by or endorsed by the Xenia team.</p>
</div>