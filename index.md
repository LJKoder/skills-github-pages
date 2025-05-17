<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Plotly with Error Bars and Regression</title>
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
      gap: 15px;
      max-width: 800px;
      margin: 0 auto 30px;
    }
    input[type="text"] {
      width: 100%;
      padding: 8px;
      font-size: 14px;
    }
    button {
      padding: 10px 20px;
      margin: 10px;
      font-size: 16px;
      cursor: pointer;
    }
    #plot {
      width: 100%;
      height: 600px;
    }
  </style>
</head>
<body>
  <h1>Plotly: Error Bars and Weighted Regression 5</h1>

  <div class="input-grid">
    <input type="text" id="xValues" placeholder="X values (space-separated)">
    <input type="text" id="yValues" placeholder="Y values">

    <input type="text" id="xErrors" placeholder="X errors (optional)">
    <input type="text" id="yErrors" placeholder="Y errors (used for weighting)">

    <input type="text" id="xLabel" placeholder="X axis label">
    <input type="text" id="yLabel" placeholder="Y axis label">
  </div>

  <button onclick="plotData()">Plot</button>

  <div id="plot"></div>

  <script>
    function parseInput(id) {
      return document.getElementById(id).value.trim().split(/\s+/).map(parseFloat);
    }

    function weightedLinearRegression(x, y, yErrors) {
      const weights = yErrors.map(e => 1 / (e * e));
      const sum = arr => arr.reduce((a, b) => a + b, 0);

      const S = sum(weights);
      const Sx = sum(x.map((xi, i) => weights[i] * xi));
      const Sy = sum(y.map((yi, i) => weights[i] * yi));
      const Sxx = sum(x.map((xi, i) => weights[i] * xi * xi));
      const Sxy = sum(x.map((xi, i) => weights[i] * xi * y[i]));

      const delta = S * Sxx - Sx * Sx;
      const slope = (S * Sxy - Sx * Sy) / delta;
      const intercept = (Sxx * Sy - Sx * Sxy) / delta;

      const slopeError = Math.sqrt(S / delta);
      const interceptError = Math.sqrt(Sxx / delta);

      return { slope, intercept, slopeError, interceptError };
    }

    function plotData() {
      const x = parseInput('xValues');
      const y = parseInput('yValues');
      const xErr = parseInput('xErrors');
      const yErr = parseInput('yErrors');
      const xLabel = document.getElementById('xLabel').value || 'X Axis';
      const yLabel = document.getElementById('yLabel').value || 'Y Axis';

      if (x.length !== y.length || (yErr.length && yErr.length !== y.length)) {
        alert("Lengths of X, Y, and Y errors must match.");
        return;
      }

      // Perform weighted linear regression
      const { slope, intercept, slopeError, interceptError } = weightedLinearRegression(x, y, yErr.length ? yErr : Array(x.length).fill(1));

      // Generate regression line
      const xMin = Math.min(...x);
      const xMax = Math.max(...x);
      const fitX = [xMin, xMax];
      const fitY = fitX.map(val => slope * val + intercept);

      const tracePoints = {
        x: x,
        y: y,
        error_x: xErr.length ? { type: "data", array: xErr, visible: true } : undefined,
        error_y: yErr.length ? { type: "data", array: yErr, visible: true } : undefined,
        mode: 'markers',
        type: 'scatter',
        marker: { color: 'blue' },
        name: 'Data'
      };

      const traceFit = {
        x: fitX,
        y: fitY,
        mode: 'lines',
        type: 'scatter',
        line: { color: 'red' },
        name: 'Fit Line'
      };

      const equationText = `y = ${slope.toFixed(2)}x + ${intercept.toFixed(2)}<br>` +
                           `± ${slopeError.toFixed(2)} , ± ${interceptError.toFixed(2)}`;

      const annotation = {
        x: xMin,
        y: slope * xMin + intercept,
        xanchor: 'left',
        yanchor: 'bottom',
        text: equationText,
        showarrow: false,
        font: { size: 14, color: 'red' },
        align: 'left',
        bgcolor: 'rgba(255,255,255,0.8)',
        bordercolor: 'red',
        borderwidth: 1
      };

      const layout = {
        title: '',
        xaxis: { title: xLabel },
        yaxis: { title: yLabel },
        showlegend: false,
        annotations: [annotation]
      };

      Plotly.newPlot('plot', [tracePoints, traceFit], layout);
    }
  </script>
</body>
</html>
