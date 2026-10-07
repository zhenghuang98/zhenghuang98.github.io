---
title: Aurora Forecast Dashboard
date: 2026-03-21
layout: post
category: Jekyll
permalink: /pages/aurora-forecast/
---

<style>
  /* Styled to sit inside the GitBook theme: flat sections under the theme's own
     headings and tables, and no fixed text or background colours, so the
     White, Sepia and Night reading themes all stay legible. */
  #aurora-app .au-muted { font-size: .85em; opacity: .7; }
  #aurora-app .au-error { font-style: italic; }
  #aurora-app .au-link {
    font: inherit; color: inherit; background: none; border: 0; padding: 0;
    text-decoration: underline; cursor: pointer;
  }

  #aurora-app .au-map {
    display: block; width: 100%; max-width: 560px; height: auto; aspect-ratio: 1 / 1;
  }

  /* Kp levels, used as fills only (bars and swatches), never as text colour */
  #aurora-app .au-k0 { background: #4a9d5d; }
  #aurora-app .au-k4 { background: #d1a012; }
  #aurora-app .au-k5 { background: #ea7a1e; }
  #aurora-app .au-k6 { background: #d9432b; }
  #aurora-app .au-k7 { background: #9c2a8e; }
  #aurora-app .au-dot { display: inline-block; width: .65em; height: .65em; margin-right: .45em; }

  /* Kp timeline */
  #aurora-app .au-chart { margin: 0 0 .5em; padding-left: 1.5em; }
  #aurora-app .au-plot {
    position: relative; display: flex; height: 150px;
    border-left: 1px solid rgba(128, 128, 128, .55);
    border-bottom: 1px solid rgba(128, 128, 128, .55);
  }
  #aurora-app .au-tick {
    position: absolute; left: 0; right: 0; height: 0;
    border-top: 1px dashed rgba(128, 128, 128, .35);
  }
  #aurora-app .au-tick span {
    position: absolute; right: 100%; top: -.8em; margin-right: .4em;
    font-size: .75em; line-height: 1.6; opacity: .7;
  }
  #aurora-app .au-day {
    position: relative; z-index: 1; flex: 1 1 0; min-width: 0; display: flex; padding: 0 2px;
  }
  #aurora-app .au-day + .au-day { border-left: 1px solid rgba(128, 128, 128, .35); }
  #aurora-app .au-slot {
    flex: 1 1 0; min-width: 0; display: flex; align-items: flex-end; padding: 0 1px;
  }
  #aurora-app .au-slot.is-now { background: rgba(128, 128, 128, .22); }
  #aurora-app .au-bar { width: 100%; min-height: 2px; }
  #aurora-app .au-bar.is-forecast { opacity: .5; }
  #aurora-app .au-xaxis { display: flex; font-size: .8em; }
  #aurora-app .au-xaxis span { flex: 1 1 0; min-width: 0; text-align: center; opacity: .7; }
  #aurora-app .au-xaxis .is-today { font-weight: 700; opacity: 1; }
  #aurora-app .au-legend span { display: inline-block; margin-right: 1.1em; white-space: nowrap; }

  /* Tables keep the theme's borders and row striping */
  #aurora-app .au-scroll { overflow-x: auto; }
  #aurora-app .au-kp-table th, #aurora-app .au-kp-table td { text-align: center; }
  #aurora-app .au-kp-table .is-now {
    font-weight: 700; outline: 2px solid currentColor; outline-offset: -2px;
  }
  #aurora-app .au-nowrap { white-space: nowrap; }
  #aurora-app .au-num { text-align: right; font-variant-numeric: tabular-nums; }
  #aurora-app .au-cover { display: flex; align-items: center; gap: .6em; }
  #aurora-app .au-meter {
    flex: 1 1 auto; min-width: 3em; max-width: 14em; height: .7em;
    border: 1px solid rgba(128, 128, 128, .55);
  }
  #aurora-app .au-meter i { display: block; height: 100%; background: currentColor; opacity: .55; }
  #aurora-app .au-pct { min-width: 2.6em; text-align: right; font-variant-numeric: tabular-nums; }

  /* Location form */
  #aurora-app .au-form { display: flex; flex-wrap: wrap; gap: .5em; margin: 0 0 .85em; }
  #aurora-app .au-input, #aurora-app .au-button {
    font: inherit; color: inherit; background: transparent;
    border: 1px solid rgba(128, 128, 128, .55); border-radius: 2px; padding: .3em .7em;
  }
  #aurora-app .au-input { flex: 1 1 14em; min-width: 0; max-width: 24em; }
  #aurora-app .au-button { cursor: pointer; }
  #aurora-app .au-button:hover, #aurora-app .au-input:focus { border-color: currentColor; }

  @media (max-width: 600px) {
    #aurora-app table th, #aurora-app table td { padding: 4px 7px; }
    #aurora-app .au-plot { height: 120px; }
    #aurora-app .au-weekday { display: block; }
    #aurora-app .au-cover { gap: .4em; }
    #aurora-app .au-meter { min-width: 1.8em; }
  }
</style>

<div id="aurora-app">

<blockquote>
  <p id="au-summary">Loading the latest Kp index…</p>
</blockquote>
<p class="au-muted"><span id="au-refreshed"></span><button type="button" class="au-link" id="au-refresh">Refresh</button></p>

<h2 id="aurora-map">Aurora map</h2>
<p><img id="au-map" class="au-map" alt="NOAA OVATION aurora forecast map for the northern hemisphere" /></p>
<p class="au-muted">OVATION model from the <a href="https://www.swpc.noaa.gov/products/aurora-30-minute-forecast" target="_blank" rel="noopener">NOAA Space Weather Prediction Center</a>: probability of aurora at the time printed on the map.<span id="au-map-status"></span></p>

<h2 id="kp-index">Kp index</h2>
<div id="au-kp-chart"><p class="au-muted">Loading…</p></div>

<h3 id="kp-forecast">3-day forecast</h3>
<div id="au-kp-table"><p class="au-muted">Loading…</p></div>

<h2 id="cloud-cover">Cloud cover</h2>
<p id="au-location"></p>
<form id="au-location-form" class="au-form">
  <input id="au-location-input" class="au-input" type="text" autocomplete="off" placeholder="City, or latitude, longitude" aria-label="Location" />
  <button type="submit" class="au-button">Set location</button>
</form>
<div id="au-location-note"></div>

<h3 id="cloud-hourly">Next 8 hours</h3>
<div id="au-cloud-hourly"><p class="au-muted">Loading…</p></div>

<h3 id="cloud-nights">Next 3 nights</h3>
<div id="au-cloud-nights"><p class="au-muted">Loading…</p></div>

<p class="au-muted">Data: <a href="https://www.swpc.noaa.gov/" target="_blank" rel="noopener">NOAA SWPC</a> (Kp index, aurora map) and <a href="https://open-meteo.com/" target="_blank" rel="noopener">Open-Meteo</a> (cloud cover, place search).</p>

</div>

<script>
(function () {
  'use strict';

  if (!document.getElementById('aurora-app')) return;

  var KP_URL = 'https://services.swpc.noaa.gov/products/noaa-planetary-k-index-forecast.json';
  var MAP_URL = 'https://services.swpc.noaa.gov/images/aurora-forecast-northern-hemisphere.jpg';
  var METEO_URL = 'https://api.open-meteo.com/v1/forecast';
  var GEOCODE_URL = 'https://geocoding-api.open-meteo.com/v1/search';
  var STORE_KEY = 'aurora-forecast.location';
  var DEFAULT_LOCATION = { lat: 41.0814, lon: -81.519, name: 'Akron, Ohio, US' };

  var HOUR = 3600000;
  var SLOT = 3 * HOUR; // NOAA reports Kp in 3-hour UTC windows
  var DAY = 24 * HOUR;
  var WEEKDAYS = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
  var MONTHS = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];
  var STORMS = ['', 'minor', 'moderate', 'strong', 'severe', 'extreme'];

  // Sequence counters let a slow, superseded request be ignored when it lands.
  var state = { loc: readStoredLocation() || DEFAULT_LOCATION, kpSeq: 0, cloudSeq: 0, placeSeq: 0 };

  // ── DOM helpers ──
  function byId(id) { return document.getElementById(id); }

  function append(node, children) {
    (children || []).forEach(function (child) {
      node.appendChild(typeof child === 'string' ? document.createTextNode(child) : child);
    });
    return node;
  }

  // Strings always become text nodes, so API and user text is never parsed as HTML.
  function el(tag, attrs, children) {
    var node = document.createElement(tag);
    Object.keys(attrs || {}).forEach(function (key) {
      if (key === 'className') node.className = attrs[key];
      else node.setAttribute(key, attrs[key]);
    });
    return append(node, children);
  }

  function fill(node, children) {
    node.textContent = '';
    return append(node, children);
  }

  function linkButton(label, onClick) {
    var button = el('button', { type: 'button', className: 'au-link' }, [label]);
    button.addEventListener('click', onClick);
    return button;
  }

  function showError(node, what, err, retry) {
    fill(node, [el('p', { className: 'au-error' }, [
      'Could not load ' + what + ' (' + ((err && err.message) || 'unknown error') + '). ',
      linkButton('Try again', retry)
    ])]);
  }

  function fetchJson(url) {
    var controller = typeof AbortController === 'function' ? new AbortController() : null;
    var timer = setTimeout(function () { if (controller) controller.abort(); }, 20000);
    return fetch(url, controller ? { signal: controller.signal } : {})
      .then(function (res) {
        return res.json().then(null, function () { return null; }).then(function (body) {
          if (!res.ok) throw new Error((body && body.reason) || 'HTTP ' + res.status);
          if (body === null) throw new Error('unreadable response');
          return body;
        });
      })
      .then(function (body) {
        clearTimeout(timer);
        return body;
      }, function (err) {
        clearTimeout(timer);
        throw new Error(err && err.name === 'AbortError' ? 'timed out' : (err && err.message) || 'network error');
      });
  }

  // ── Time formatting ──
  function pad(n) { return (n < 10 ? '0' : '') + n; }

  // NOAA time tags are UTC ISO strings with no zone suffix (older feeds used a
  // space instead of the "T").
  function parseUtc(tag) {
    var m = /^(\d{4})-(\d\d)-(\d\d)[T ](\d\d):(\d\d)/.exec(tag || '');
    return m ? Date.UTC(+m[1], +m[2] - 1, +m[3], +m[4], +m[5]) : NaN;
  }

  function utcDate(ms) {
    var d = new Date(ms);
    return WEEKDAYS[d.getUTCDay()] + ', ' + MONTHS[d.getUTCMonth()] + ' ' + d.getUTCDate();
  }

  function utcSlot(ms) {
    var hour = new Date(ms).getUTCHours();
    return pad(hour) + '–' + pad((hour + 3) % 24);
  }

  function localClock(ms) {
    var d = new Date(ms);
    return pad(d.getHours()) + ':' + pad(d.getMinutes());
  }

  // The same 3-hour window in the viewer's own time zone, e.g. "Fri 05:00–08:00".
  function localSlot(ms) {
    return WEEKDAYS[new Date(ms).getDay()] + ' ' + localClock(ms) + '–' + localClock(ms + SLOT);
  }

  // ── Kp index ──
  // The feed is an array of objects ({time_tag, kp, observed, noaa_scale}); it used
  // to be an array of arrays with a header row. Accept both, drop anything unparseable.
  function normalizeKp(data) {
    var rows = [];
    (Array.isArray(data) ? data : []).forEach(function (raw) {
      var row = Array.isArray(raw)
        ? { time_tag: raw[0], kp: raw[1], observed: raw[2], noaa_scale: raw[3] }
        : (raw || {});
      var start = parseUtc(row.time_tag);
      var kp = parseFloat(row.kp !== undefined ? row.kp : row.Kp);
      if (isNaN(start) || isNaN(kp)) return;
      rows.push({ start: start, kp: kp, type: row.observed || '', scale: row.noaa_scale || '' });
    });
    rows.sort(function (a, b) { return a.start - b.start; });
    return rows;
  }

  // Kp comes in thirds. NOAA's G storm scale uses the nearest whole Kp, except
  // that only a full Kp 9 is G5. Prefer the scale NOAA sends when it is there.
  function describeKp(row) {
    var rounded = Math.round(row.kp);
    var match = /^G([1-5])$/.exec(row.scale);
    var storm = match ? +match[1] : (row.kp >= 9 ? 5 : rounded >= 5 ? Math.min(4, rounded - 4) : 0);
    if (storm) {
      return { cls: 'au-k' + Math.min(7, storm + 4), label: 'G' + storm + ' ' + STORMS[storm] + ' storm' };
    }
    if (rounded === 4) return { cls: 'au-k4', label: 'active' };
    return { cls: 'au-k0', label: rounded >= 2 ? 'unsettled' : 'quiet' };
  }

  function slotTitle(row) {
    return utcDate(row.start) + ' ' + utcSlot(row.start) + ' UTC (' + localSlot(row.start) + ' your time): Kp ' +
      row.kp.toFixed(2) + ', ' + describeKp(row).label + (row.type ? ', ' + row.type : '');
  }

  function renderSummary(rows, now) {
    var current = null, latest = null, peak = null;
    rows.forEach(function (row) {
      if (row.start <= now) latest = row;
      if (row.start <= now && now < row.start + SLOT) current = row;
      if (row.start + SLOT > now && (!peak || row.kp > peak.kp)) peak = row;
    });
    var shown = current || latest;
    var parts = [];
    if (shown) {
      parts.push(el('strong', {}, ['Kp ' + shown.kp.toFixed(2)]));
      parts.push((current ? ' now' : ' at the last report') + ' — ' + describeKp(shown).label + '. ');
    }
    if (peak && peak !== current) {
      parts.push('Forecast peak: ');
      parts.push(el('strong', {}, ['Kp ' + peak.kp.toFixed(2)]));
      parts.push(' (' + describeKp(peak).label + ') on ' + utcDate(peak.start) + ', ' + utcSlot(peak.start) +
        ' UTC — ' + localSlot(peak.start) + ' your time.');
    } else if (peak) {
      parts.push('Nothing higher is forecast through ' + utcDate(rows[rows.length - 1].start) + '.');
    }
    fill(byId('au-summary'), parts);
  }

  // Two days of history plus the three forecast days, one bar per 3-hour window.
  function renderChart(bySlot, today, now) {
    var plot = el('div', { className: 'au-plot' });
    var axis = el('div', { className: 'au-xaxis' });
    [3, 6, 9].forEach(function (kp) {
      plot.appendChild(el('div', { className: 'au-tick', style: 'bottom:' + (kp / 9 * 100).toFixed(2) + '%' },
        [el('span', {}, [String(kp)])]));
    });
    for (var d = -2; d <= 2; d++) {
      var dayStart = today + d * DAY;
      var day = el('div', { className: 'au-day' });
      for (var s = 0; s < 8; s++) {
        var start = dayStart + s * SLOT;
        var row = bySlot[start];
        var slot = el('div', { className: 'au-slot' + (start <= now && now < start + SLOT ? ' is-now' : '') });
        if (row) {
          slot.title = slotTitle(row);
          slot.appendChild(el('div', {
            className: 'au-bar ' + describeKp(row).cls + (row.type === 'predicted' ? ' is-forecast' : ''),
            style: 'height:' + (Math.min(9, Math.max(0, row.kp)) / 9 * 100).toFixed(1) + '%'
          }));
        }
        day.appendChild(slot);
      }
      plot.appendChild(day);
      var date = new Date(dayStart);
      axis.appendChild(el('span', d === 0 ? { className: 'is-today' } : {},
        [MONTHS[date.getUTCMonth()] + ' ' + date.getUTCDate()]));
    }

    var legend = el('p', { className: 'au-muted au-legend' });
    [['au-k0', 'Kp 0–3 quiet to unsettled'], ['au-k4', '4 active'], ['au-k5', '5 G1'], ['au-k6', '6 G2'], ['au-k7', '7+ G3 or stronger']]
      .forEach(function (item) {
        legend.appendChild(el('span', {}, [el('i', { className: 'au-dot ' + item[0] }), item[1]]));
      });
    legend.appendChild(document.createTextNode('Solid bars are observed, faded bars are forecast; the shaded column is now. Days are UTC.'));

    fill(byId('au-kp-chart'), [
      el('div', { className: 'au-chart', role: 'img', 'aria-label': 'Planetary Kp index for the past two days and the three-day forecast' }, [plot, axis]),
      legend
    ]);
  }

  // NOAA's own layout: one column per UTC day, one row per 3-hour window.
  function renderTable(bySlot, today, now) {
    var headRow = el('tr', {}, [el('th', { scope: 'col' }, ['UTC'])]);
    var body = el('tbody');
    for (var d = 0; d < 3; d++) {
      var date = new Date(today + d * DAY);
      headRow.appendChild(el('th', { scope: 'col' }, [
        el('span', { className: 'au-weekday' }, [WEEKDAYS[date.getUTCDay()]]),
        ' ' + MONTHS[date.getUTCMonth()] + ' ' + date.getUTCDate()
      ]));
    }
    for (var s = 0; s < 8; s++) {
      var tr = el('tr', {}, [el('th', { scope: 'row', className: 'au-nowrap' }, [utcSlot(today + s * SLOT)])]);
      for (var c = 0; c < 3; c++) {
        var start = today + c * DAY + s * SLOT;
        var row = bySlot[start];
        var td = el('td', start <= now && now < start + SLOT ? { className: 'is-now' } : {});
        if (row) {
          td.title = slotTitle(row);
          append(td, [el('i', { className: 'au-dot ' + describeKp(row).cls }), row.kp.toFixed(2)]);
        } else {
          td.appendChild(document.createTextNode('–'));
        }
        tr.appendChild(td);
      }
      body.appendChild(tr);
    }
    fill(byId('au-kp-table'), [
      el('div', { className: 'au-scroll' }, [
        el('table', { className: 'au-kp-table' }, [el('thead', {}, [headRow]), body])
      ]),
      el('p', { className: 'au-muted' }, ['Outlined cell: the current window. Hover a value for your local time.'])
    ]);
  }

  function loadKp() {
    var seq = ++state.kpSeq;
    fetchJson(KP_URL).then(function (data) {
      if (seq !== state.kpSeq) return;
      var rows = normalizeKp(data);
      if (!rows.length) throw new Error('no usable rows in the response');
      var now = Date.now();
      var today = Math.floor(now / DAY) * DAY;
      var bySlot = {};
      rows.forEach(function (row) { bySlot[row.start] = row; });
      renderSummary(rows, now);
      renderChart(bySlot, today, now);
      renderTable(bySlot, today, now);
    }).catch(function (err) {
      if (seq !== state.kpSeq) return;
      fill(byId('au-summary'), ['The Kp index is unavailable right now.']);
      showError(byId('au-kp-chart'), 'the Kp index', err, loadKp);
      fill(byId('au-kp-table'), []);
    });
  }

  // ── Aurora map ──
  function loadMap() {
    byId('au-map').src = MAP_URL + '?t=' + Date.now();
  }

  // ── Location ──
  function validLocation(loc) {
    return !!loc && typeof loc.lat === 'number' && typeof loc.lon === 'number' &&
      isFinite(loc.lat) && isFinite(loc.lon) && Math.abs(loc.lat) <= 90 && Math.abs(loc.lon) <= 180;
  }

  function readStoredLocation() {
    try {
      var loc = JSON.parse(window.localStorage.getItem(STORE_KEY));
      if (validLocation(loc)) {
        return { lat: loc.lat, lon: loc.lon, name: typeof loc.name === 'string' ? loc.name.slice(0, 80) : '' };
      }
    } catch (e) { /* storage blocked or corrupt: use the default */ }
    return null;
  }

  function formatCoords(loc) {
    return Math.abs(loc.lat).toFixed(2) + '°' + (loc.lat < 0 ? 'S' : 'N') + ', ' +
      Math.abs(loc.lon).toFixed(2) + '°' + (loc.lon < 0 ? 'W' : 'E');
  }

  function renderLocation(zone) {
    var loc = state.loc;
    var parts = loc.name
      ? [el('strong', {}, [loc.name]), ' (' + formatCoords(loc) + ')']
      : [el('strong', {}, [formatCoords(loc)])];
    if (zone) parts.push(' · times below are local to this place (' + zone + ')');
    fill(byId('au-location'), parts);
  }

  function setLocation(loc) {
    state.loc = loc;
    try { window.localStorage.setItem(STORE_KEY, JSON.stringify(loc)); } catch (e) { /* not persisted */ }
    byId('au-location-input').value = '';
    loadCloud();
  }

  function noteLocation(children) {
    fill(byId('au-location-note'), children.length ? [el('p', { className: 'au-muted' }, children)] : []);
  }

  function searchPlace(query) {
    var seq = ++state.placeSeq;
    // Open-Meteo only matches names in the requested language.
    var language = /[㐀-鿿]/.test(query) ? 'zh' : 'en';
    noteLocation(['Searching…']);
    fetchJson(GEOCODE_URL + '?name=' + encodeURIComponent(query) + '&count=5&format=json&language=' + language)
      .then(function (data) {
        if (seq !== state.placeSeq) return;
        var places = ((data && data.results) || []).map(function (r) {
          return { lat: r.latitude, lon: r.longitude, name: [r.name, r.admin1, r.country_code].filter(Boolean).join(', ') };
        }).filter(validLocation);
        if (!places.length) {
          noteLocation(['No place found for “' + query + '”. Try another spelling, or enter coordinates such as 64.84, -147.72.']);
          return;
        }
        setLocation(places[0]);
        var others = [];
        places.slice(1).forEach(function (place) {
          others.push(others.length ? ' · ' : 'Other matches: ');
          others.push(linkButton(place.name, function () {
            noteLocation([]);
            setLocation(place);
          }));
        });
        noteLocation(others);
      })
      .catch(function (err) {
        if (seq !== state.placeSeq) return;
        noteLocation(['Place search failed (' + err.message + '). You can still enter coordinates such as 64.84, -147.72.']);
      });
  }

  function onLocationSubmit(event) {
    event.preventDefault();
    var text = byId('au-location-input').value.trim();
    if (!text) return;
    var coords = /^(-?\d+(?:\.\d+)?)\s*[,;\s]\s*(-?\d+(?:\.\d+)?)$/.exec(text);
    if (!coords) {
      searchPlace(text);
      return;
    }
    state.placeSeq++; // cancel any place search still in flight
    var loc = { lat: parseFloat(coords[1]), lon: parseFloat(coords[2]), name: '' };
    if (!validLocation(loc)) {
      noteLocation(['Latitude must be between −90 and 90, and longitude between −180 and 180.']);
      return;
    }
    noteLocation([]);
    setLocation(loc);
  }

  // ── Cloud cover ──
  function percent(value) {
    return typeof value === 'number' ? Math.round(value) + '%' : '–';
  }

  function coverCell(value) {
    var known = typeof value === 'number';
    var width = known ? Math.min(100, Math.max(0, value)) : 0;
    return el('td', {}, [el('div', { className: 'au-cover' }, [
      el('span', { className: 'au-meter' }, [el('i', { style: 'width:' + width + '%' })]),
      el('span', { className: 'au-pct' }, [percent(value)])
    ])]);
  }

  function skyText(cover) {
    return cover <= 20 ? 'Clear' : cover <= 50 ? 'Partly cloudy' : cover <= 80 ? 'Mostly cloudy' : 'Overcast';
  }

  function table(headings, rows) {
    return el('div', { className: 'au-scroll' }, [el('table', {}, [
      el('thead', {}, [el('tr', {}, headings.map(function (heading) {
        return el('th', { scope: 'col' }, [heading]);
      }))]),
      el('tbody', {}, rows)
    ])]);
  }

  function renderHourly(hourly, start) {
    var rows = [];
    for (var i = start; i < Math.min(start + 8, hourly.time.length); i++) {
      rows.push(el('tr', {}, [
        el('td', {}, [i === start ? 'Now' : hourly.time[i].slice(11, 16)]),
        coverCell(hourly.cloud_cover[i]),
        el('td', { className: 'au-num' }, [percent((hourly.cloud_cover_low || [])[i])]),
        el('td', { className: 'au-num' }, [percent((hourly.cloud_cover_mid || [])[i])]),
        el('td', { className: 'au-num' }, [percent((hourly.cloud_cover_high || [])[i])])
      ]));
    }
    fill(byId('au-cloud-hourly'), [table(['Time', 'Cloud cover', 'Low', 'Mid', 'High'], rows)]);
  }

  // A night is a run of hours with the sun down. Runs are also split at local
  // noon so that polar night still yields one row per day. Before dawn, what is
  // left of the current night is listed in addition to the three nights ahead.
  function findNights(hourly, start) {
    var nights = [];
    var full = 0;
    var run = null;
    for (var i = start; i < hourly.time.length; i++) {
      var hour = +hourly.time[i].slice(11, 13);
      if (hourly.is_day[i] !== 0) {
        run = null;
        continue;
      }
      if (!run || hour === 12) {
        if (full === 3) break;
        run = { from: i, to: i, sum: 0, count: 0, remnant: i === start && hour < 12 };
        if (!run.remnant) full++;
        nights.push(run);
      }
      run.to = i;
      if (typeof hourly.cloud_cover[i] === 'number') {
        run.sum += hourly.cloud_cover[i];
        run.count++;
      }
    }
    return nights;
  }

  function nightName(hourly, night, index, start) {
    if (night.remnant) return 'Rest of tonight';
    var date = hourly.time[night.from].slice(0, 10);
    if (index === 0 && (night.from === start || date === hourly.time[start].slice(0, 10))) return 'Tonight';
    var parts = date.split('-');
    return WEEKDAYS[new Date(Date.UTC(+parts[0], +parts[1] - 1, +parts[2])).getUTCDay()] + ' night';
  }

  function renderNights(hourly, start) {
    var target = byId('au-cloud-nights');
    if (!Array.isArray(hourly.is_day)) {
      fill(target, [el('p', { className: 'au-muted' }, ['Night-time data is unavailable for this place.'])]);
      return;
    }
    var nights = findNights(hourly, start);
    if (!nights.length) {
      fill(target, [el('p', { className: 'au-muted' }, ['The sun does not set here in the next few days.'])]);
      return;
    }
    var rows = nights.map(function (night, index) {
      var mean = night.count ? night.sum / night.count : null;
      var from = night.from === start ? 'now' : hourly.time[night.from].slice(11, 16);
      var hours = night.from === night.to ? from : from + '–' + hourly.time[night.to].slice(11, 16);
      return el('tr', {}, [
        el('td', {}, [nightName(hourly, night, index, start)]),
        el('td', { className: 'au-nowrap' }, [hours]),
        coverCell(mean),
        el('td', {}, [mean === null ? '–' : skyText(mean)])
      ]);
    });
    fill(target, [
      table(['Night', 'Dark hours', 'Mean cloud cover', 'Sky'], rows),
      el('p', { className: 'au-muted' }, ['Dark hours run from sunset to sunrise; twilight is included.'])
    ]);
  }

  function loadCloud() {
    var seq = ++state.cloudSeq;
    var loc = state.loc;
    renderLocation('');
    fetchJson(METEO_URL + '?latitude=' + loc.lat + '&longitude=' + loc.lon +
      '&hourly=cloud_cover,cloud_cover_low,cloud_cover_mid,cloud_cover_high,is_day&timezone=auto&forecast_days=4')
      .then(function (data) {
        if (seq !== state.cloudSeq) return;
        var hourly = data && data.hourly;
        if (!hourly || !Array.isArray(hourly.time) || !Array.isArray(hourly.cloud_cover) || !hourly.time.length) {
          throw new Error('unexpected response');
        }
        // Hourly times are wall-clock times at the location, so compare them with
        // "now" shifted into that zone; ISO strings sort chronologically.
        var offset = (data.utc_offset_seconds || 0) * 1000;
        var nowHour = new Date(Date.now() + offset).toISOString().slice(0, 13);
        var start = 0;
        for (var i = 0; i < hourly.time.length; i++) {
          if (hourly.time[i].slice(0, 13) <= nowHour) start = i;
          else break;
        }
        var zone = data.timezone || '';
        if (zone && data.timezone_abbreviation && data.timezone_abbreviation !== zone) {
          zone += ', ' + data.timezone_abbreviation;
        }
        renderLocation(zone);
        renderHourly(hourly, start);
        renderNights(hourly, start);
      })
      .catch(function (err) {
        if (seq !== state.cloudSeq) return;
        showError(byId('au-cloud-hourly'), 'cloud cover', err, loadCloud);
        fill(byId('au-cloud-nights'), []);
      });
  }

  // ── Init ──
  function refreshAll() {
    fill(byId('au-refreshed'), ['Refreshed at ' + localClock(Date.now()) + ' · ']);
    loadMap();
    loadKp();
    loadCloud();
  }

  byId('au-map').addEventListener('load', function () { fill(byId('au-map-status'), []); });
  byId('au-map').addEventListener('error', function () {
    fill(byId('au-map-status'), [' The map image could not be loaded right now.']);
  });
  byId('au-refresh').addEventListener('click', refreshAll);
  byId('au-location-form').addEventListener('submit', onLocationSubmit);

  refreshAll();
})();
</script>
