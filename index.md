<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Plotly Weighted Regression with Error Bars</title>
  <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      padding: 20px;
    }
    .input-grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      max-width: 600px;
      margin-bottom: 20px;
    }
    input[type="text"], button {
      padding: 8px;
      font-size: 14px;
      width: 100%;
    }
    button {
      margin-top: 10px;
    }
  </style>
</head>
<body>
  <h2>Plot Data with Weighted Regression and Error Bars 3</h2>
  <div class="input-grid">
    <div>
      <label for="xLabel">X Axis Label</label>
      <input type="text" id="xLabel" placeholder="X Axis Label" />

      <label for="xValues">X Values</label>
      <input type="text" id="xValues" placeholder="space-separated" />

      <label for="xErrors">X Errors</label>
      <input type="text" id="xErrors" placeholder="space-separated" />
    </div>
    <div>
      <label for="yLabel">Y Axis Label</label>
      <input type="text" id="yLabel" placeholder="Y Axis Label" />

      <label for="yValues">Y Values</label>
      <input type="text" id="yValues" placeholder="space-separated" />

      <label for="yErrors">Y Errors</label>
      <input type="text" id="yErrors" placeholder="space-separated (optional)" />
    </div>
  </div>
  <button onclick="plotData()">Plot</button>
  <div id="plot" style="width: 100%; height: 600px;"></div>

  <script>
    function parseInput(id) {
      return document.getElementById(id).value.trim().split(/\s+/).map(Number);
    }

    function weightedLinearRegression(x, y, weights) {
      const sum = arr => arr.reduce((a, b) => a + b, 0);
      const sumw = sum(weights);
      const sumwx = sum(x.map((v, i) => v * weights[i]));
      const sumwy = sum(y.map((v, i) => v * weights[i]));
      const sumwx2 = sum(x.map((v, i) => v * v * weights[i]));
      const sumwxy = sum(x.map((v, i) => v * y[i] * weights[i]));

      const xbar = sumwx / sumw;
      const ybar = sumwy / sumw;

      const slope = (sumwxy - sumwx * ybar) / (sumwx2 - sumwx * xbar);
      const intercept = ybar - slope * xbar;

      return { slope, intercept };
    }

    function plotData() {
      const x = parseInput("xValues");
      const y = parseInput("yValues");
      const xErr = parseInput("xErrors");
      const yErr = parseInput("yErrors");

      const xLabel = document.getElementById("xLabel").value || "X";
      const yLabel = document.getElementById("yLabel").value || "Y";

      if (x.length !== y.length ||
          (xErr.length && xErr.length !== x.length) ||
          (yErr.length && yErr.length !== y.length)) {
        alert("Lengths of values and errors must match.");
        return;
      }

      const weights = y.map((_, i) => yErr[i] ? 1 / (yErr[i] ** 2) : 1);
      const { slope, intercept } = weightedLinearRegression(x, y, weights);

      const fitLineX = [Math.min(...x), Math.max(...x)];
      const fitLineY = fitLineX.map(xVal => slope * xVal + intercept);

      const trace1 = {
        x: x,
        y: y,
        error_x: {
          type: "data",
          array: xErr,
          visible: true
        },
        error_y: {
          type: "data",
          array: yErr,
          visible: true
        },
        mode: "markers",
        type: "scatter",
        marker: { color: "blue" },
        showlegend: false
      };

      const trace2 = {
        x: fitLineX,
        y: fitLineY,
        mode: "lines",
        type: "scatter",
        line: { color: "red" },
        showlegend: false
      };

      const equationText = `y = ${slope.toFixed(3)}x + ${intercept.toFixed(3)}`;
      const annotation = {
        x: 0.05,
        y: 0.95,
        xref: "paper",
        yref: "paper",
        text: equationText,
        showarrow: false,
        font: {
          size: 16,
          color: "black"
        }
      };

      const layout = {
        xaxis: { title: xLabel },
        yaxis: { title: yLabel },
        annotations: [annotation],
        showlegend: false
      };

      Plotly.newPlot("plot", [trace1, trace2], layout);
    }
  </script>
</body>
</html>
