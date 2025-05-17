<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Weighted Regression with Plotly</title>
  <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 40px;
    }
    .input-grid {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 10px;
      margin-bottom: 20px;
      max-width: 900px;
    }
    .input-grid div {
      display: flex;
      flex-direction: column;
    }
    textarea {
      width: 100%;
      height: 100px;
      font-size: 14px;
      padding: 5px;
    }
    button {
      padding: 10px 20px;
      font-size: 16px;
      margin-right: 10px;
      background-color: #4CAF50;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    #plot {
      width: 100%;
      height: 600px;
    }
  </style>
</head>
<body>
  <h2>Plotly Weighted Linear Regression with Error Bars 4</h2>

  <div class="input-grid">
    <div>
      <label for="xValues">X Values</label>
      <textarea id="xValues" placeholder="e.g. 1 2 3"></textarea>
    </div>
    <div>
      <label for="yValues">Y Values</label>
      <textarea id="yValues" placeholder="e.g. 2.1 2.9 4.2"></textarea>
    </div>
    <div>
      <label for="yErrors">Y Errors</label>
      <textarea id="yErrors" placeholder="e.g. 0.1 0.2 0.15"></textarea>
    </div>
  </div>

  <button onclick="plotData()">Plot Graph</button>
  <div id="plot"></div>

  <script>
    function parseInput(id) {
      return document.getElementById(id).value.trim().split(/\s+/).map(Number);
    }

    function weightedLinearRegression(x, y, yErr) {
      const w = yErr.map(e => 1 / (e * e));
      const sum = arr => arr.reduce((a, b) => a + b, 0);

      const sw = sum(w);
      const swx = sum(x.map((xi, i) => w[i] * xi));
      const swy = sum(y.map((yi, i) => w[i] * yi));
      const swxy = sum(x.map((xi, i) => w[i] * xi * y[i]));
      const swx2 = sum(x.map((xi, i) => w[i] * xi * xi));

      const delta = sw * swx2 - swx * swx;
      const slope = (sw * swxy - swx * swy) / delta;
      const intercept = (swy * swx2 - swx * swxy) / delta;

      // uncertainties
      const slopeErr = Math.sqrt(sw / delta);
      const interceptErr = Math.sqrt(swx2 / delta);

      return { slope, intercept, slopeErr, interceptErr };
    }

    function plotData() {
      const x = parseInput("xValues");
      const y = parseInput("yValues");
      const yErr = parseInput("yErrors");

      if (x.length !== y.length || yErr.length !== y.length) {
        alert("All arrays must be of the same length.");
        return;
      }

      const tracePoints = {
        x: x,
        y: y,
        error_y: {
          type: "data",
          array: yErr,
          visible: true,
          color: "black",
          thickness: 1.5,
          width: 0
        },
        mode: "markers",
        type: "scatter",
        marker: { color: "blue", size: 8 },
        name: "Data"
      };

      const { slope, intercept, slopeErr, interceptErr } = weightedLinearRegression(x, y, yErr);

      const xMin = Math.min(...x);
      const xMax = Math.max(...x);
      const fitX = [xMin, xMax];
      const fitY = fitX.map(xi => slope * xi + intercept);

      const traceFit = {
        x: fitX,
        y: fitY,
        mode: "lines",
        type: "scatter",
        line: { color: "red", width: 2 },
        name: "Fit"
      };

      const equationText = `y = ${slope.toFixed(3)} ± ${slopeErr.toFixed(3)} x + ${intercept.toFixed(3)} ± ${interceptErr.toFixed(3)}`;

      const annotation = {
        x: xMin + 0.05 * (xMax - xMin),
        y: Math.max(...y),
        text: equationText,
        showarrow: false,
        font: {
          family: "Arial",
          size: 16,
          color: "black"
        },
        bgcolor: "#f3f3f3",
        bordercolor: "#c7c7c7",
        borderwidth: 1,
        borderpad: 4
      };

      const layout = {
        margin: { t: 40 },
        showlegend: false,
        xaxis: { title: "X" },
        yaxis: { title: "Y" },
        annotations: [annotation]
      };

      Plotly.newPlot("plot", [tracePoints, traceFit], layout, { responsive: true });
    }
  </script>
</body>
</html>
