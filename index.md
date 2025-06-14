<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Monte Carlo Linear Fit with Error Bars</title>
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
  <h1>Monte Carlo Linear Regression with Error Bars</h1>
  <p>This tool performs a Monte Carlo simulation using both X and Y uncertainties.</p>

  <div class="input-grid">
    <!-- Axis Labels -->
    <div><strong>Axis Labels</strong></div><div></div><div></div>
    <div style="grid-column: span 3;">
      <p>Use <code>&lt;sub&gt;</code> and <code>&lt;sup&gt;</code> tags for formatting (e.g. H&lt;sub&gt;2&lt;/sub&gt;O → H<sub>2</sub>O).</p>
    </div>
    <input type="text" id="xLabel" placeholder="X Axis Label">
    <input type="text" id="yLabel" placeholder="Y Axis Label">
    <div></div>

    <!-- X and Y Values -->
    <div><strong>X Values</strong></div><div></div><div></div>
    <input type="text" id="xValues" placeholder="X Values (space-separated)">
    <input type="text" id="xErrors" placeholder="X Errors (optional)">
    <div></div>

    <div><strong>Y Values</strong></div><div></div><div></div>
    <input type="text" id="yValues" placeholder="Y Values (space-separated)">
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

    function monteCarloLinFit2D(x, y, xErr, yErr, nIter = 10000) {
      const slopes = [];
      const intercepts = [];

      for (let i = 0; i < nIter; i++) {
        const xSim = x.map((xi, j) => xi + randn_bm() * (xErr[j] || 0));
        const ySim = y.map((yi, j) => yi + randn_bm() * (yErr[j] || 0));
        const { slope, intercept } = linregress(xSim, ySim);
        if (isFinite(slope) && isFinite(intercept)) {
          slopes.push(slope);
          intercepts.push(intercept);
        }
      }

      const mean = arr => arr.reduce((a, b) => a + b, 0) / arr.length;
      const std = arr => {
        const m = mean(arr);
        return Math.sqrt(arr.reduce((s, v) => s + (v - m) ** 2, 0) / (arr.length - 1));
      };

      return {
        slope: { mean: mean(slopes), std: std(slopes) },
        intercept: { mean: mean(intercepts), std: std(intercepts) }
      };
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

      const { slope, intercept } = monteCarloLinFit2D(x, y, xErr, yErr);

      const lineX = [Math.min(...x), Math.max(...x)];
      const lineY = lineX.map(xi => slope.mean * xi + intercept.mean);

      const data = [];

      data.push({
        x: x,
        y: y,
        mode: 'markers',
        type: 'scatter',
        name: 'Data',
        marker: {
          color: 'black',
          size: 7,
          symbol: 'x-thin-open'
        },
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

      data.push({
        x: lineX,
        y: lineY,
        mode: 'lines',
        type: 'scatter',
        name: 'MC Fit',
        line: { color: 'red', width: 2 }
      });

      const annotationText = `y = (${slope.mean.toExponential(3)} ± ${slope.std.toExponential(3)})x + (${intercept.mean.toExponential(3)} ± ${intercept.std.toExponential(3)})`;

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
      Plotly.downloadImage('plot', { format: 'png', filename: 'plot_with_mc_regression' });
    }
  </script>
</body>
</html>
