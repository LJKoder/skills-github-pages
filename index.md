<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Monte Carlo Linear Fit with Confidence Band</title>
  <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 40px;
      text-align: center;
    }
    .input-grid {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 15px;
      max-width: 900px;
      margin: 0 auto 30px;
    }
    .input-grid strong {
      font-size: 16px;
      grid-column: span 3;
      text-align: left;
      padding-top: 10px;
    }
    input[type="text"] {
      padding: 8px;
      font-size: 14px;
      width: 100%;
    }
    button {
      padding: 10px 20px;
      margin: 10px;
      font-size: 16px;
      background-color: #4CAF50;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    button:hover {
      background-color: #45a049;
    }
    #plot {
      width: 100%;
      height: 75vh;
    }
  </style>
</head>
<body>
  <h1>Monte Carlo Linear Regression with Confidence Band</h1>
  <p>Paste your data from Excel into the boxes below. Includes both x and y errors.</p>

  <div class="input-grid">
    <div><strong>Axis Labels</strong></div><div></div><div></div>
    <div style="grid-column: span 3;">
      <p>You can use HTML like <code>&lt;sub&gt;</code> and <code>&lt;sup&gt;</code> (e.g., H&lt;sub&gt;2&lt;/sub&gt;O → H<sub>2</sub>O).</p>
    </div>
    <input type="text" id="xLabel" placeholder="X Axis Label">
    <input type="text" id="yLabel" placeholder="Y Axis Label">
    <div></div>

    <div><strong>X Values</strong></div><div></div><div></div>
    <input type="text" id="xValues" placeholder="X Values (space-separated)">
    <input type="text" id="xErrors" placeholder="X Errors (optional)">
    <div></div>

    <div><strong>Y Values</strong></div><div></div><div></div>
    <input type="text" id="yValues" placeholder="Y Values">
    <input type="text" id="yErrors" placeholder="Y Errors (optional)">
    <div></div>
  </div>

  <button onclick="plotData()">Plot</button>
  <button onclick="downloadPlot()">Download Chart</button>
  <div id="plot"></div>

  <script>
    function parseValues(id) {
      const val = document.getElementById(id).value.trim();
      return val === "" ? [] : val.split(/\s+/).map(Number);
    }

    function plotData() {
      const x = parseValues('xValues');
      const y = parseValues('yValues');
      const xErr = parseValues('xErrors');
      const yErr = parseValues('yErrors');
      const xLabel = document.getElementById('xLabel').value || 'X Axis';
      const yLabel = document.getElementById('yLabel').value || 'Y Axis';

      if (x.length !== y.length) {
        alert('X and Y values must be the same length.');
        return;
      }

      const N = 10000;
      const slopes = [], intercepts = [];

      function randn_bm() {
        let u = 0, v = 0;
        while (u === 0) u = Math.random();
        while (v === 0) v = Math.random();
        return Math.sqrt(-2.0 * Math.log(u)) * Math.cos(2.0 * Math.PI * v);
      }

      function linregress(xs, ys) {
        const n = xs.length;
        const xMean = xs.reduce((a, b) => a + b, 0) / n;
        const yMean = ys.reduce((a, b) => a + b, 0) / n;
        let num = 0, den = 0;
        for (let i = 0; i < n; i++) {
          num += (xs[i] - xMean) * (ys[i] - yMean);
          den += (xs[i] - xMean) ** 2;
        }
        const slope = num / den;
        const intercept = yMean - slope * xMean;
        return { slope, intercept };
      }

      for (let i = 0; i < N; i++) {
        const xSim = x.map((xi, j) => xi + randn_bm() * (xErr[j] || 0));
        const ySim = y.map((yi, j) => yi + randn_bm() * (yErr[j] || 0));
        const { slope, intercept } = linregress(xSim, ySim);
        if (isFinite(slope) && isFinite(intercept)) {
          slopes.push(slope);
          intercepts.push(intercept);
        }
      }

      const xRange = [];
      const xmin = Math.min(...x), xmax = Math.max(...x);
      const steps = 100;
      const step = (xmax - xmin) / steps;
      for (let i = 0; i <= steps; i++) {
        xRange.push(xmin + i * step);
      }

      const lower = [], upper = [], meanLine = [];
      for (let xi of xRange) {
        const ySamples = slopes.map((m, i) => m * xi + intercepts[i]);
        ySamples.sort((a, b) => a - b);
        const lo = ySamples[Math.floor(0.025 * ySamples.length)];
        const hi = ySamples[Math.floor(0.975 * ySamples.length)];
        const avg = ySamples.reduce((a, b) => a + b, 0) / ySamples.length;
        lower.push(lo);
        upper.push(hi);
        meanLine.push(avg);
      }

      const mean = arr => arr.reduce((a, b) => a + b, 0) / arr.length;
      const std = arr => {
        const m = mean(arr);
        return Math.sqrt(arr.reduce((s, v) => s + (v - m) ** 2, 0) / (arr.length - 1));
      };
      const slopeStat = { mean: mean(slopes), std: std(slopes) };
      const interceptStat = { mean: mean(intercepts), std: std(intercepts) };

      const annotationText = `y = (${slopeStat.mean.toExponential(3)} ± ${slopeStat.std.toExponential(3)})x + (${interceptStat.mean.toExponential(3)} ± ${interceptStat.std.toExponential(3)})`;

      const data = [];

      // Original data points with error bars
      data.push({
        x: x,
        y: y,
        mode: 'markers',
        type: 'scatter',
        name: 'Data',
        marker: { color: 'black', size: 7, symbol: 'x-thin-open' },
        error_x: xErr.length === x.length ? {
          type: 'data',
          array: xErr,
          visible: true
        } : undefined,
        error_y: yErr.length === y.length ? {
          type: 'data',
          array: yErr,
          visible: true
        } : undefined
      });

      // Confidence band
      data.push({
        x: [...xRange, ...xRange.slice().reverse()],
        y: [...upper, ...lower.slice().reverse()],
        fill: 'toself',
        fillcolor: 'rgba(255, 0, 0, 0.2)',
        line: { color: 'transparent' },
        name: '95% Confidence Band',
        type: 'scatter',
        showlegend: false
      });

      // Mean regression line
      data.push({
        x: xRange,
        y: meanLine,
        mode: 'lines',
        type: 'scatter',
        name: 'Mean Fit',
        line: { color: 'red', width: 2 }
      });

      const layout = {
        title: '',
        xaxis: { title: xLabel },
        yaxis: { title: yLabel },
        showlegend: false,
        annotations: [{
          x: 0.05,
          y: 0.95,
          xref: 'paper',
          yref: 'paper',
          text: annotationText,
          showarrow: false,
          font: { color: 'black', size: 14 }
        }]
      };

      Plotly.newPlot('plot', data, layout);
    }

    function downloadPlot() {
      Plotly.downloadImage('plot', { format: 'png', filename: 'monte_carlo_regression' });
    }
  </script>
</body>
</html>
