<h2 id="publications" style="margin: 2px 0px 8px;">Selected Papers</h2>

<div class="pub-filter" role="group" aria-label="Filter publications">
  <button type="button" class="active" data-filter="all">All</button>
  <button type="button" data-filter="conf">Conference</button>
  <button type="button" data-filter="jour">Journal</button>
  <button type="button" data-filter="preprint">Preprint</button>
</div>

<p style="color: #1f2937; margin: 0 0 8px 0;"><em>* Co-First Authors, † Corresponding Authors</em></p>

<div class="publications" data-filter="all">
<ol class="bibliography">

{% assign groups = "preprint,conf,jour" | split: "," %}
{% for group in groups %}

<h3 data-type="{{ group }}" style="margin:8px 10px 8px; color: #C2410C; font-size: 1rem; font-weight: 600;">
{%- case group -%}
{%- when "preprint" -%}Preprints
{%- when "conf" -%}Conference Papers
{%- when "jour" -%}Journal Papers
{%- endcase -%}
</h3>

{% for link in site.data.publications[group] %}

<li data-type="{{ group }}">
<div class="pub-row">
  <div class="col-sm-3 abbr" style="position: relative;padding-right: 15px;padding-left: 15px;">
    <img src="{{ link.image }}" class="teaser img-fluid z-depth-1" style="width=100;height=40%">
            <abbr class="badge">{{ link.conference_short }}</abbr>
  </div>
  <div class="col-sm-9" style="position: relative;padding-right: 15px;padding-left: 20px;">
      <div class="title"><a href="{{ link.pdf }}">{{ link.title }}</a></div>
      <div class="author">{{ link.authors }}</div>
      <div class="periodical"><em>{{ link.conference }}</em>
      </div>
    <div class="links">
      {% if link.pdf %} 
      <a href="{{ link.pdf }}" class="btn btn-sm z-depth-0" role="button" target="_blank" style="font-size:12px;">PDF</a>
      {% endif %}
      {% if link.code %} 
      <a href="{{ link.code }}" class="btn btn-sm z-depth-0" role="button" target="_blank" style="font-size:12px;">Code</a>
      {% endif %}
      {% if link.page %} 
      <a href="{{ link.page }}" class="btn btn-sm z-depth-0" role="button" target="_blank" style="font-size:12px;">Project Page</a>
      {% endif %}
      {% if link.bibtex %} 
      <a href="{{ link.bibtex }}" class="btn btn-sm z-depth-0" role="button" target="_blank" style="font-size:12px;">BibTex</a>
      {% endif %}
      {% if link.notes %} 
      <strong> <i style="color:#C2410C">{{ link.notes }}</i></strong>
      {% endif %}
      {% if link.others %} 
      {{ link.others }}
      {% endif %}
    </div>
  </div>
</div>
<br>
</li>

{% endfor %}
{% endfor %}

</ol>
</div>

<script>
(function () {
  var pubs = document.querySelector('.publications');
  var buttons = document.querySelectorAll('.pub-filter button');
  buttons.forEach(function (btn) {
    btn.addEventListener('click', function () {
      pubs.setAttribute('data-filter', btn.getAttribute('data-filter'));
      buttons.forEach(function (b) { b.classList.toggle('active', b === btn); });
    });
  });
})();
</script>
