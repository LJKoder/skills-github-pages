<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Debug Plot with Error Bars</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      margin: 40px;
    }
    #chart-container {
      width: 100%;
      height: 70vh;
    }
    canvas {
      width: 100% !important;
      height: 100% !important;
    }
  </style>
  
  <!-- Chart.js -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  
  <!-- Error Bars Plugin -->
  <script src="https://cdn.jsdelivr.net/npm/chartjs-chart-error-bars@3.0.0/dist/chartjs-chart-error-bars.umd.js"></script>

</head>
<body>
  <h1>Debug: Plot with Error Bars</h1>

  <div id="chart-container">
    <canvas id="myChart"></canvas>
  </div>

  <script>
    // Register error bar chart types from the plugin
    if (window.ChartErrorBars) {
      Chart.register(
        window.ChartErrorBars.ScatterWithErrorBarsController,
        window.ChartErrorBars.ErrorBarElement
      );
      console.log("✅ Error bars plugin registered.");
    } else {
      console.warn("❌ Error bars plugin not available.");
    }

    const ctx = document.getElementById('myChart').getContext('2d');

    const testData = [
      { x: 1, y: 2, xMin: 0.8, xMax: 1.2, yMin: 1.8, yMax: 2.2 },
      { x: 2, y: 3, xMin: 1.9, xMax: 2.1, yMin: 2.7, yMax: 3.3 },
      { x: 3, y: 4, xMin: 2.7, xMax: 3.3, yMin: 3.8, yMax: 4.2 }
    ];

    let chart;
    try {
      chart = new Chart(ctx, {
        type: 'scatterWithErrorBars',
        data: {
          datasets: [{
            label: 'Test Data with Error Bars',
            data: testData,
            backgroundColor: 'rgba(75, 192, 192, 0.6)',
            borderColor: 'rgba(75, 192, 192, 1)',
            pointRadius: 5,
            errorBarWhiskerColor: 'black',
            errorBarWhiskerLineWidth: 1.5,
          }]
        },
        options: {
          responsive: true,
          plugins: {
            legend: { display: true },
            tooltip: { mode: 'nearest', intersect: false }
          },
          scales: {
            x: {
              title: {
                display: true,
                text: 'X Axis'
              }
            },
            y: {
              title: {
                display: true,
                text: 'Y Axis'
              }
            }
          }
        }
      });
      console.log("✅ Chart rendered with error bars.");
    } catch (error) {
      console.error("❌ Chart rendering failed:", error);
      alert("Chart rendering failed. Check the console for details.");
    }
  </script>
</body>
</html>
