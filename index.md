<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Plot with Error Bars and Regression</title>
  <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 40px;
      text-align: center;
    }
    .input-grid {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 10px;
      max-width: 600px;
      margin: 0 auto 20px;
    }
    input[type="text"] {
      width: 100%;
      padding: 8px;
    }
    button {
      padding: 10px 20px;
      margin-top: 10px;
      font-size: 16px;
    }
  </style>
</head>
<body>
  <h1>Plot Data with Error Bars and Regression 2</h1>

  <div class="input-grid">
    <input id="xValues" placeholder="X Values (space-separated)">
    <input id="yValues" placeholder="Y Values">
    <input id="yErrors" placeholder="Y Errors">
    <input id="xLabel" placeholder="X Axis Label">
    <input id="yLabel" placeholder="Y Axis Label">
  </div>

  <button onclick="plot()">Plot Graph</button>
  <div id="chart" style="width: 100%; height: 600px;"></div>

  <script>
    function parseInput(id) {
      return document.getElementById(id).value.trim().split(/\s+/).map(Number);
    }

    function weightedLinearRegression(x, y, errors) {
      const w = errors.map(e => 1 / (e * e));
      const sum = arr => arr.reduce((a, b) => a + b, 0);
      const S = sum(w);
      const Sx = sum(x.map((xi, i) => xi * w[i]));
      const Sy = sum(y.map((yi, i) => yi * w[i]));
      const Sxx = sum(x.map((xi, i) => xi * xi * w[i]));
      const Sxy = sum(x.map((xi, i) => xi * y[i] * w[i]));

      const denom = S * Sxx - Sx * Sx;
      const slope = (S * Sxy - Sx * Sy) / denom;
      const intercept = (Sxx * Sy - Sx * Sxy) / denom;

      const slopeError = Math.sqrt(S / denom);
      const interceptError = Math.sqrt(Sxx / denom);

      return { slope, intercept, slopeError, interceptError };
    }

    function plot() {
      const x = parseInput('xValues');
      const y = parseInput('yValues');
      const yErr = parseInput('yErrors');

      if (x.length !== y.length || y.length !== yErr.length) {
        alert("X, Y, and Y Errors must have the same length.");
        return;
      }

      const xLabel = document.getElementById("xLabel").value || "X Axis";
      const yLabel = document.getElementById("yLabel").value || "Y Axis";

      const { slope, intercept, slopeError, interceptError } =
        weightedLinearRegression(x, y, yErr);

      const regressionY = x.map(xi => slope * xi + intercept);

      const traceData = {
        x: x,
        y: y,
        error_y: {
          type: 'data',
          array: yErr,
          visible: true
        },
        mode: 'markers',
        name: 'Data',
        type: 'scatter',
        marker: { color: 'blue' }
      };

      const traceLine = {
        x: x,
        y: regressionY,
        mode: 'lines',
        line: { color: 'red' },
        type: 'scatter',
        showlegend: false
      };

      const equationText = `y = ${slope.toFixed(2)} ± ${slopeError.toFixed(2)}x + ${intercept.toFixed(2)} ± ${interceptError.toFixed(2)}`;

      const layout = {
        title: `Weighted Linear Regression`,
        xaxis: { title: xLabel },
        yaxis: { title: yLabel },
        showlegend: false,
        annotations: [{
          x: Math.min(...x),
          y: Math.max(...y),
          xanchor: 'left',
          yanchor: 'top',
          text: equationText,
          showarrow: false,
          font: {
            size: 14,
            color: 'black'
          },
          bgcolor: 'rgba(255,255,255,0.7)',
          bordercolor: 'black',
          borderwidth: 1
        }]
      };

      Plotly.newPlot('chart', [traceData, traceLine], layout);
    }
  </script>
</body>
</html>
