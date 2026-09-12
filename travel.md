---
layout: homepage
---

<a href="./">← Back to Home</a>

## Travel

{% assign n_cities = 0 %}{% for c in site.data.travel %}{% assign n_cities = n_cities | plus: c.cities.size %}{% endfor %}
<p class="travel-stats"><strong>{{ site.data.travel | size }}</strong> countries · <strong>{{ n_cities }}</strong> cities</p>

<div class="travel-map">
  <svg id="travel-svg" viewBox="0 0 960 470" role="img" aria-label="World map of places I have visited"></svg>
  <div class="map-tip" hidden></div>
</div>

<div class="travel-legend">
  <span><i class="sw born"></i>Born</span>
  <span><i class="sw"></i>Visited</span>
  <span><i class="dot home"></i>Living now</span>
  <span><i class="dot"></i>City visited</span>
</div>
<p class="travel-hint">Drag to pan · ⌘/Ctrl + scroll or double-click to zoom · <a href="#" id="map-reset">Reset view</a></p>

<ul class="travel-list">
{% for c in site.data.travel %}
  <li><strong>{{ c.flag }} {{ c.country }}</strong>{% if c.status == "born" %} <em>born</em>{% endif %}{% if c.cities.size > 0 %} <span class="cities">· {% for city in c.cities %}{{ city.name }}{% if city.status == "home" %} <em>living</em>{% endif %}{% unless forloop.last %}, {% endunless %}{% endfor %}</span>{% endif %}</li>
{% endfor %}
</ul>

<script src="https://cdn.jsdelivr.net/npm/d3@7.9.0/dist/d3.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/topojson-client@3.1.0/dist/topojson-client.min.js"></script>
<script>
(function () {
  var trips = {{ site.data.travel | jsonify }};
  var W = 960, H = 470;
  var svg = d3.select('#travel-svg');
  var g = svg.append('g');
  var wrap = document.querySelector('.travel-map');
  var tip = document.querySelector('.map-tip');
  var k = 1;

  var status = {};
  var cities = [];
  trips.forEach(function (c) {
    status[String(c.id).padStart(3, '0')] = c.status || 'visited';
    (c.cities || []).forEach(function (city) {
      cities.push({ name: city.name, country: c.country, lat: city.lat, lon: city.lon, home: city.status === 'home' });
    });
  });

  function showTip(event, text) {
    var r = wrap.getBoundingClientRect();
    tip.textContent = text;
    tip.style.left = (event.clientX - r.left) + 'px';
    tip.style.top = (event.clientY - r.top) + 'px';
    tip.hidden = false;
  }
  function hideTip() { tip.hidden = true; }

  var projection = d3.geoNaturalEarth1();
  var path = d3.geoPath(projection);

  d3.json('./assets/data/countries-110m.json').then(function (world) {
    var features = topojson.feature(world, world.objects.countries).features
      .filter(function (f) { return f.id !== '010'; });
    var land = { type: 'FeatureCollection', features: features };
    projection.fitWidth(W, land);
    H = Math.ceil(path.bounds(land)[1][1]);
    svg.attr('viewBox', '0 0 ' + W + ' ' + H);

    g.append('g').selectAll('path')
      .data(features).join('path')
      .attr('class', function (f) { return 'country ' + (status[f.id] || ''); })
      .attr('d', path)
      .filter(function (f) { return status[f.id]; })
      .on('mousemove', function (event, f) { showTip(event, f.properties.name); })
      .on('mouseleave', hideTip);

    var dots = g.append('g').selectAll('g')
      .data(cities).join('g')
      .attr('class', function (d) { return 'city' + (d.home ? ' home' : ''); })
      .attr('transform', function (d) { var p = projection([d.lon, d.lat]); return 'translate(' + p[0] + ',' + p[1] + ')'; })
      .on('mousemove', function (event, d) { showTip(event, d.name + ', ' + d.country); })
      .on('mouseleave', hideTip);
    dots.filter(function (d) { return d.home; }).append('circle').attr('class', 'ring');
    dots.append('circle').attr('class', 'pin');

    function rescale() {
      g.selectAll('.country').attr('stroke-width', 0.5 / k);
      g.selectAll('.pin').attr('r', function (d) { return (d.home ? 6.5 : 5) / k; }).attr('stroke-width', 1.2 / k);
      g.selectAll('.ring').attr('r', 13 / k).attr('stroke-width', 1.5 / k);
    }
    rescale();

    var zoom = d3.zoom()
      .scaleExtent([1, 8])
      .translateExtent([[0, 0], [W, H]])
      .filter(function (event) {
        if (event.type === 'wheel') return event.ctrlKey || event.metaKey;
        return !event.button;
      })
      .on('zoom', function (event) {
        k = event.transform.k;
        g.attr('transform', event.transform);
        rescale();
        hideTip();
      });
    svg.call(zoom);
    document.getElementById('map-reset').addEventListener('click', function (e) {
      e.preventDefault();
      svg.transition().duration(500).call(zoom.transform, d3.zoomIdentity);
    });
  });
})();
</script>
