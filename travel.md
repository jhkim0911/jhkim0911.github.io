---
layout: homepage
updated: 2026-09-12
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

<p class="travel-updated">Updated on {{ page.updated | date: "%b %d, %Y" }}</p>

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
    var cid = String(c.id).padStart(3, '0');
    status[cid] = c.status || 'visited';
    (c.cities || []).forEach(function (city) {
      cities.push({ name: city.name, country: c.country, cid: cid, lat: city.lat, lon: city.lon, home: city.status === 'home' });
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

  // State/province a city belongs to: the one containing it, else the nearest one
  // within 50 km (coastal city centres can sit just outside the simplified shoreline).
  function regionOf(regions, city) {
    var p = [city.lon, city.lat];
    var own = regions.filter(function (r) { return r.properties.cid === city.cid; });
    for (var i = 0; i < own.length; i++) if (d3.geoContains(own[i], p)) return own[i];
    var best = null, bestD = 50 / 6371;
    own.forEach(function (r) {
      d3.geoStream(r, {
        point: function (x, y) { var d = d3.geoDistance(p, [x, y]); if (d < bestD) { bestD = d; best = r; } },
        lineStart: function () {}, lineEnd: function () {}, polygonStart: function () {}, polygonEnd: function () {}, sphere: function () {}
      });
    });
    return best;
  }

  var projection = d3.geoNaturalEarth1();
  var path = d3.geoPath(projection);

  Promise.all([
    d3.json('./assets/data/countries-110m.json'),
    d3.json('./assets/data/states-50m.json')
  ]).then(function (res) {
    var regions = topojson.feature(res[1], res[1].objects.states).features;
    var split = {};
    regions.forEach(function (r) { split[r.properties.cid] = true; });
    var countries = topojson.feature(res[0], res[0].objects.countries).features
      .filter(function (f) { return f.id !== '010' && !split[f.id]; });
    var land = { type: 'FeatureCollection', features: countries.concat(regions) };
    projection.fitWidth(W, land);
    H = Math.ceil(path.bounds(land)[1][1]);
    svg.attr('viewBox', '0 0 ' + W + ' ' + H);

    cities.forEach(function (c) {
      var r = split[c.cid] && regionOf(regions, c);
      if (r) r.properties.visited = true;
    });

    g.append('g').selectAll('path')
      .data(countries).join('path')
      .attr('class', function (f) { return 'country ' + (status[f.id] || ''); })
      .attr('d', path)
      .filter(function (f) { return status[f.id]; })
      .on('mousemove', function (event, f) { showTip(event, f.properties.name); })
      .on('mouseleave', hideTip);

    g.append('g').selectAll('path')
      .data(regions).join('path')
      .attr('class', function (r) { return 'country region' + (r.properties.visited ? ' visited' : ''); })
      .attr('d', path)
      .filter(function (r) { return r.properties.visited; })
      .on('mousemove', function (event, r) { showTip(event, r.properties.name + ', ' + r.properties.country); })
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
