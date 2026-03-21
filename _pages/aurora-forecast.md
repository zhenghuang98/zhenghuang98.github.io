---
title: Aurora Forecast Dashboard
date: 2026-03-21
layout: post
category: Jekyll
---

<style>
  .aurora-dashboard {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    color: #e0e0e0;
  }
  .aurora-dashboard h2 {
    border-bottom: 1px solid #444;
    padding-bottom: 6px;
  }
  .dashboard-grid {
    display: grid;
    grid-template-columns: 1fr;
    gap: 20px;
    margin: 16px 0;
  }
  .card {
    background: #1a1a2e;
    border: 1px solid #333;
    border-radius: 10px;
    padding: 20px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.3);
  }
  .card h3 {
    margin-top: 0;
    color: #7ec8e3;
    font-size: 1.1em;
  }
  .card img {
    width: 100%;
    max-width: 600px;
    border-radius: 6px;
    display: block;
    margin: 0 auto;
  }
  .kp-table {
    width: 100%;
    border-collapse: collapse;
    margin: 10px 0;
    font-size: 0.9em;
  }
  .kp-table th, .kp-table td {
    padding: 8px 10px;
    border: 1px solid #333;
    text-align: center;
  }
  .kp-table th {
    background: #16213e;
    color: #7ec8e3;
  }
  .kp-table td {
    background: #1a1a2e;
  }
  .kp-0, .kp-1 { color: #4caf50; }
  .kp-2, .kp-3 { color: #8bc34a; }
  .kp-4 { color: #ffeb3b; }
  .kp-5 { color: #ff9800; font-weight: bold; }
  .kp-6 { color: #ff5722; font-weight: bold; }
  .kp-7 { color: #f44336; font-weight: bold; }
  .kp-8, .kp-9 { color: #e91e63; font-weight: bold; }
  .cloud-bar-container {
    display: flex;
    align-items: center;
    gap: 8px;
  }
  .cloud-bar-bg {
    flex: 1;
    background: #16213e;
    border-radius: 4px;
    height: 18px;
    overflow: hidden;
  }
  .cloud-bar-fill {
    height: 100%;
    border-radius: 4px;
    transition: width 0.3s;
  }
  .cloud-bar-label {
    min-width: 40px;
    text-align: right;
    font-size: 0.85em;
  }
  .location-info {
    background: #16213e;
    padding: 10px 14px;
    border-radius: 6px;
    margin-bottom: 14px;
    font-size: 0.9em;
    color: #aaa;
  }
  .legend {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    margin: 10px 0;
    font-size: 0.8em;
  }
  .legend-item {
    display: flex;
    align-items: center;
    gap: 4px;
  }
  .legend-color {
    width: 14px;
    height: 14px;
    border-radius: 3px;
    display: inline-block;
  }
  .refresh-btn {
    background: #16213e;
    color: #7ec8e3;
    border: 1px solid #7ec8e3;
    padding: 6px 16px;
    border-radius: 6px;
    cursor: pointer;
    font-size: 0.9em;
    margin-bottom: 16px;
  }
  .refresh-btn:hover {
    background: #7ec8e3;
    color: #1a1a2e;
  }
  .loading {
    color: #888;
    font-style: italic;
  }
  .error-msg {
    color: #f44336;
    font-style: italic;
  }
  .timestamp {
    font-size: 0.8em;
    color: #666;
    margin-top: 8px;
  }
  .two-col {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 20px;
  }
  @media (max-width: 768px) {
    .two-col { grid-template-columns: 1fr; }
  }
</style>

<div class="aurora-dashboard">

<button class="refresh-btn" onclick="refreshAll()">↻ Refresh All Data</button>

<div class="dashboard-grid">

<!-- Aurora Map -->
<div class="card">
  <h3>🌌 NOAA Aurora Forecast — Northern Hemisphere</h3>
  <p style="font-size:0.85em;color:#999;">OVATION Prime model — 30-minute forecast updated every 5 min.</p>
  <img id="aurora-map" alt="Aurora Forecast Map" />
  <div class="legend">
    <div class="legend-item"><span class="legend-color" style="background:#1a6b1a;"></span> Low</div>
    <div class="legend-item"><span class="legend-color" style="background:#4caf50;"></span> Moderate</div>
    <div class="legend-item"><span class="legend-color" style="background:#ffeb3b;"></span> High</div>
    <div class="legend-item"><span class="legend-color" style="background:#ff5722;"></span> Very High</div>
  </div>
  <div id="aurora-map-ts" class="timestamp"></div>
</div>

<!-- KP Index -->
<div class="card">
  <h3>📊 Planetary Kp Index — 3-Day Forecast</h3>
  <p style="font-size:0.85em;color:#999;">NOAA SWPC estimated Kp values in 3-hour intervals.</p>
  <div id="kp-forecast-content"><span class="loading">Loading KP forecast…</span></div>
  <div class="legend" style="margin-top:12px;">
    <div class="legend-item"><span class="legend-color" style="background:#4caf50;"></span> Kp 0–1 Quiet</div>
    <div class="legend-item"><span class="legend-color" style="background:#8bc34a;"></span> Kp 2–3 Unsettled</div>
    <div class="legend-item"><span class="legend-color" style="background:#ffeb3b;"></span> Kp 4 Active</div>
    <div class="legend-item"><span class="legend-color" style="background:#ff9800;"></span> Kp 5 Minor Storm</div>
    <div class="legend-item"><span class="legend-color" style="background:#f44336;"></span> Kp 6–7 Major Storm</div>
    <div class="legend-item"><span class="legend-color" style="background:#e91e63;"></span> Kp 8–9 Severe</div>
  </div>
</div>

<!-- KP Recent Observed -->
<div class="card">
  <h3>📈 Recent Observed Kp Index</h3>
  <p style="font-size:0.85em;color:#999;">Most recent observed planetary Kp values.</p>
  <canvas id="kp-chart" height="180"></canvas>
  <div id="kp-observed-content"><span class="loading">Loading observed Kp…</span></div>
</div>

<!-- Cloud Coverage -->
<div class="card">
  <h3>☁️ Cloud Coverage Forecast</h3>
  <p style="font-size:0.85em;color:#999;">Based on your approximate IP location via <a href="https://open-meteo.com/" style="color:#7ec8e3;">Open-Meteo</a>.</p>
  <div id="location-info" class="location-info">Detecting your location…</div>

  <div class="two-col">
    <div>
      <h4 style="color:#7ec8e3;margin-top:0;">Next 5 Hours (Hourly)</h4>
      <div id="cloud-hourly"><span class="loading">Loading…</span></div>
    </div>
    <div>
      <h4 style="color:#7ec8e3;margin-top:0;">3-Day Daily Forecast</h4>
      <div id="cloud-daily"><span class="loading">Loading…</span></div>
    </div>
  </div>
</div>

</div><!-- end dashboard-grid -->

<p style="font-size:0.75em;color:#555;margin-top:20px;">
  Data sources: <a href="https://www.swpc.noaa.gov/" style="color:#555;">NOAA Space Weather Prediction Center</a>,
  <a href="https://open-meteo.com/" style="color:#555;">Open-Meteo</a>,
  <a href="https://ipapi.co/" style="color:#555;">ipapi.co</a>.
  All data refreshed on page load. Aurora map imagery © NOAA.
</p>

</div><!-- end aurora-dashboard -->

<script>
(function() {
  // ── Aurora Map ──
  function loadAuroraMap() {
    var img = document.getElementById('aurora-map');
    // NOAA provides a static latest image for the northern hemisphere ovation model
    var ts = new Date().getTime();
    img.src = 'https://services.swpc.noaa.gov/images/aurora-forecast-northern-hemisphere.jpg?' + ts;
    document.getElementById('aurora-map-ts').textContent = 'Map loaded at ' + new Date().toLocaleString();
  }

  // ── KP Forecast (3-day) ──
  function loadKpForecast() {
    var container = document.getElementById('kp-forecast-content');
    fetch('https://services.swpc.noaa.gov/products/noaa-planetary-k-index-forecast.json')
      .then(function(r) { return r.json(); })
      .then(function(data) {
        // data is array of arrays: [time_tag, Kp, observed/estimated/predicted, noaa_scale]
        // skip header row if present
        var rows = data;
        if (rows.length > 0 && rows[0][0] === 'time_tag') rows = rows.slice(1);

        // group by date
        var byDate = {};
        rows.forEach(function(row) {
          var dt = row[0]; // e.g. "2026-03-21 03:00:00"
          var dateStr = dt.split(' ')[0];
          if (!byDate[dateStr]) byDate[dateStr] = [];
          byDate[dateStr].push({ time: dt.split(' ')[1].substring(0,5), kp: parseFloat(row[1]), type: row[2] });
        });

        var html = '<table class="kp-table"><thead><tr><th>Date</th><th>Time (UTC)</th><th>Kp</th><th>Type</th></tr></thead><tbody>';
        var dates = Object.keys(byDate).sort();
        dates.forEach(function(date) {
          byDate[date].forEach(function(entry, idx) {
            var kpClass = 'kp-' + Math.min(9, Math.floor(entry.kp));
            html += '<tr>';
            if (idx === 0) html += '<td rowspan="' + byDate[date].length + '">' + date + '</td>';
            html += '<td>' + entry.time + '</td>';
            html += '<td class="' + kpClass + '">' + entry.kp.toFixed(2) + '</td>';
            html += '<td style="font-size:0.8em;color:#888;">' + entry.type + '</td>';
            html += '</tr>';
          });
        });
        html += '</tbody></table>';
        container.innerHTML = html;
      })
      .catch(function(err) {
        container.innerHTML = '<span class="error-msg">Failed to load KP forecast: ' + err.message + '</span>';
      });
  }

  // ── KP Observed (chart) ──
  function loadKpObserved() {
    var container = document.getElementById('kp-observed-content');
    fetch('https://services.swpc.noaa.gov/products/noaa-planetary-k-index.json')
      .then(function(r) { return r.json(); })
      .then(function(data) {
        var rows = data;
        if (rows.length > 0 && rows[0][0] === 'time_tag') rows = rows.slice(1);

        // take last 24 entries
        var recent = rows.slice(-24);

        // draw simple bar chart on canvas
        var canvas = document.getElementById('kp-chart');
        if (!canvas || !canvas.getContext) { container.innerHTML = ''; return; }
        var ctx = canvas.getContext('2d');
        var W = canvas.width = canvas.parentElement.clientWidth - 40;
        var H = canvas.height = 180;
        ctx.clearRect(0, 0, W, H);

        var barW = Math.floor((W - 40) / recent.length) - 2;
        var maxKp = 9;
        var originX = 35, originY = H - 25;
        var chartH = originY - 10;

        // axis
        ctx.strokeStyle = '#444';
        ctx.lineWidth = 1;
        ctx.beginPath(); ctx.moveTo(originX, 5); ctx.lineTo(originX, originY); ctx.lineTo(W, originY); ctx.stroke();

        // y labels
        ctx.fillStyle = '#888';
        ctx.font = '10px sans-serif';
        ctx.textAlign = 'right';
        for (var k = 0; k <= 9; k += 3) {
          var y = originY - (k / maxKp) * chartH;
          ctx.fillText(k, originX - 4, y + 3);
          ctx.strokeStyle = '#2a2a2a'; ctx.beginPath(); ctx.moveTo(originX, y); ctx.lineTo(W, y); ctx.stroke();
        }

        // bars
        recent.forEach(function(row, i) {
          var kp = parseFloat(row[1]);
          var h = (kp / maxKp) * chartH;
          var x = originX + 4 + i * (barW + 2);
          var color = kp < 2 ? '#4caf50' : kp < 4 ? '#8bc34a' : kp < 5 ? '#ffeb3b' : kp < 6 ? '#ff9800' : kp < 8 ? '#f44336' : '#e91e63';
          ctx.fillStyle = color;
          ctx.fillRect(x, originY - h, barW, h);

          // x label every 4th bar
          if (i % 4 === 0) {
            ctx.fillStyle = '#666';
            ctx.font = '9px sans-serif';
            ctx.textAlign = 'center';
            var lbl = row[0].split(' ')[1];
            if (lbl) ctx.fillText(lbl.substring(0,5), x + barW / 2, originY + 12);
          }
        });

        container.innerHTML = '<div class="timestamp">Showing last ' + recent.length + ' observed 3-hour intervals</div>';
      })
      .catch(function(err) {
        container.innerHTML = '<span class="error-msg">Failed to load observed Kp: ' + err.message + '</span>';
      });
  }

  // ── Cloud Coverage ──
  function loadCloudCoverage() {
    var locEl = document.getElementById('location-info');
    var hourlyEl = document.getElementById('cloud-hourly');
    var dailyEl = document.getElementById('cloud-daily');

    // 1. Get location from IP
    fetch('https://ipapi.co/json/')
      .then(function(r) { return r.json(); })
      .then(function(geo) {
        var lat = geo.latitude;
        var lon = geo.longitude;
        locEl.innerHTML = '📍 <strong>' + (geo.city || 'Unknown') + ', ' + (geo.region || '') + ', ' + (geo.country_name || geo.country || '') + '</strong> (' + lat.toFixed(2) + '°, ' + lon.toFixed(2) + '°)';

        // 2. Fetch cloud data from Open-Meteo
        var url = 'https://api.open-meteo.com/v1/forecast?latitude=' + lat + '&longitude=' + lon +
          '&hourly=cloud_cover,cloud_cover_low,cloud_cover_mid,cloud_cover_high' +
          '&daily=weather_code&timezone=auto&forecast_days=3';
        return fetch(url);
      })
      .then(function(r) { return r.json(); })
      .then(function(weather) {
        // Hourly — find current hour index, show next 5 hours
        var now = new Date();
        var times = weather.hourly.time; // ISO strings
        var clouds = weather.hourly.cloud_cover;
        var cloudsLow = weather.hourly.cloud_cover_low;
        var cloudsMid = weather.hourly.cloud_cover_mid;
        var cloudsHigh = weather.hourly.cloud_cover_high;

        // find the index closest to now
        var startIdx = 0;
        for (var i = 0; i < times.length; i++) {
          if (new Date(times[i]) >= now) { startIdx = i; break; }
        }

        var html = '';
        for (var j = startIdx; j < Math.min(startIdx + 5, times.length); j++) {
          var t = new Date(times[j]);
          var hr = t.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
          var pct = clouds[j];
          var color = pct < 25 ? '#4caf50' : pct < 50 ? '#8bc34a' : pct < 75 ? '#ffeb3b' : '#ff9800';
          html += '<div style="margin-bottom:8px;">';
          html += '<div style="font-size:0.85em;margin-bottom:2px;">' + hr + '</div>';
          html += '<div class="cloud-bar-container">';
          html += '<div class="cloud-bar-bg"><div class="cloud-bar-fill" style="width:' + pct + '%;background:' + color + ';"></div></div>';
          html += '<div class="cloud-bar-label">' + pct + '%</div>';
          html += '</div>';
          html += '<div style="font-size:0.75em;color:#666;">Low ' + (cloudsLow[j] || 0) + '% · Mid ' + (cloudsMid[j] || 0) + '% · High ' + (cloudsHigh[j] || 0) + '%</div>';
          html += '</div>';
        }
        hourlyEl.innerHTML = html;

        // Daily — compute average cloud cover per day from hourly data
        var dailyMap = {};
        for (var d = 0; d < times.length; d++) {
          var dayKey = times[d].split('T')[0];
          if (!dailyMap[dayKey]) dailyMap[dayKey] = { sum: 0, count: 0 };
          dailyMap[dayKey].sum += clouds[d];
          dailyMap[dayKey].count += 1;
        }

        var dhtml = '';
        var dayKeys = Object.keys(dailyMap).sort().slice(0, 3);
        var weatherCodes = {
          0: 'Clear sky', 1: 'Mainly clear', 2: 'Partly cloudy', 3: 'Overcast',
          45: 'Fog', 48: 'Rime fog', 51: 'Light drizzle', 53: 'Moderate drizzle',
          55: 'Dense drizzle', 61: 'Slight rain', 63: 'Moderate rain', 65: 'Heavy rain',
          71: 'Slight snow', 73: 'Moderate snow', 75: 'Heavy snow', 80: 'Slight showers',
          81: 'Moderate showers', 82: 'Violent showers', 95: 'Thunderstorm'
        };

        dayKeys.forEach(function(dk, idx) {
          var avg = Math.round(dailyMap[dk].sum / dailyMap[dk].count);
          var color = avg < 25 ? '#4caf50' : avg < 50 ? '#8bc34a' : avg < 75 ? '#ffeb3b' : '#ff9800';
          var wCode = weather.daily && weather.daily.weather_code ? weather.daily.weather_code[idx] : null;
          var desc = wCode !== null && weatherCodes[wCode] ? weatherCodes[wCode] : '';
          var dayDate = new Date(dk + 'T12:00:00');
          var dayLabel = dayDate.toLocaleDateString(undefined, { weekday: 'short', month: 'short', day: 'numeric' });

          dhtml += '<div style="margin-bottom:10px;">';
          dhtml += '<div style="font-size:0.85em;margin-bottom:2px;">' + dayLabel + (desc ? ' — ' + desc : '') + '</div>';
          dhtml += '<div class="cloud-bar-container">';
          dhtml += '<div class="cloud-bar-bg"><div class="cloud-bar-fill" style="width:' + avg + '%;background:' + color + ';"></div></div>';
          dhtml += '<div class="cloud-bar-label">' + avg + '%</div>';
          dhtml += '</div>';
          dhtml += '</div>';
        });
        dailyEl.innerHTML = dhtml;
      })
      .catch(function(err) {
        locEl.innerHTML = '<span class="error-msg">Could not detect location.</span>';
        hourlyEl.innerHTML = '<span class="error-msg">' + err.message + '</span>';
        dailyEl.innerHTML = '';
      });
  }

  // ── Init ──
  function refreshAll() {
    loadAuroraMap();
    loadKpForecast();
    loadKpObserved();
    loadCloudCoverage();
  }
  window.refreshAll = refreshAll;

  // Run on page load
  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', refreshAll);
  } else {
    refreshAll();
  }
})();
</script>
