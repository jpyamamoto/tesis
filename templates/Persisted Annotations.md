Title: {{title}}
Tags: {{hashTags}}

Author: {{authors}}
Zotero Link: {{pdfZoteroLink}}

> [!NOTE]- Abstract
> {{abstractNote}}

{% persist "annotations" %}
{% set newAnnotations = annotations | filterby("date","dateafter", lastImportDate) %}
{% if newAnnotations.length > 0 %}

### Imported: {{importDate | format("YYYY-MM-DD")}}

{% for a in newAnnotations %}
> {% if a.annotatedText %}{{a.annotatedText}}{% endif %}{{a.colorCategory}} {{a.type}} [Page{{a.page}}]({{a.desktopURI}})
> {% if a.hashTags %}{{a.hashTags}}{% endif %}
> {% if a.comment %}{{a.comment}}{% endif %}
{% endfor %}

{% endif %}
{% endpersist %}