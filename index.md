<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Plot X, Y Data with Error Bars</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 50px;
        }

        .input-section {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin: 20px 0;
        }

        input[type="text"] {
            width: 200px;
            padding: 10px;
            font-size: 16px;
        }

        button {
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
        }

        button:hover {
            background-color: #45a049;
        }

        .clear-button {
            background-color: #f44336;
            margin-left: 10px;
        }

        .clear-button:hover {
            background-color: #d32f2f;
        }

        #chart-container {
            margin-top: 50px;
            width: 100%;
            height: 80vh;
        }

        canvas {
            width: 100% !important;
            height: 100% !important;
        }
    </style>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-errorbars"></script>
</head>
<body>
    <h1>Plot X, Y Data with Error Bars</h1>
    <p>Enter X, Y values and their uncertainties (tab or newline-separated) to plot with error bars.</p>

    <div class="input-section">
        <div>
            <input type="text" id="xValues" placeholder="X Values">
            <br>
            <button class="clear-button" onclick="clearInput('xValues')">Clear</button>
        </div>
        <div>
            <input type="text" id="yValues" placeholder="Y Values">
            <br>
            <button class="clear-button" onclick="clearInput('yValues')">Clear</button>
        </div>
        <div>
            <input type="text" id="xUncertainty" placeholder="X Uncertainty">
            <br>
            <button class="clear-button" onclick="clearInput('xUncertainty')">Clear</button>
        </div>
        <div>
            <input type="text" id="yUncertainty" placeholder="Y Uncertainty">
            <br>
            <button class="clear-button" onclick="clearInput('yUncertainty')">Clear</button>
        </div>
    </div>

    <button onclick="plotGraph()">Plot Data</button>

    <div id="chart-container">
        <canvas id="myChart"></canvas>
    </div>

    <script>
        let chart; // To store the Chart.js instance

        // Register the error bar plugin
        Chart.register(ChartErrorBars);

     function plotGraph() {
    const canvas = document.getElementById('myChart');
    const ctx = canvas.getContext('2d');

    // Destroy the previous chart instance if it exists
    if (chart) {
        chart.destroy();
    }

    // Hardcoded dataset for testing
    const dataset = {
        data: [
            { x: 1, y: 2, xMin: 0.5, xMax: 1.5, yMin: 1.5, yMax: 2.5 },
            { x: 2, y: 3, xMin: 1.8, xMax: 2.2, yMin: 2.8, yMax: 3.2 },
            { x: 3, y: 4, xMin: 2.7, xMax: 3.3, yMin: 3.5, yMax: 4.5 },
        ],
        backgroundColor: 'rgba(54, 162, 235, 0.5)',
        borderColor: 'rgba(54, 162, 235, 1)',
        errorBarColor: 'rgba(0, 0, 0, 0.8)',
    };

    console.log('Dataset for Testing:', dataset);

    // Initialize the chart with the hardcoded dataset
    chart = new Chart(ctx, {
        type: 'scatter',
        data: {
            datasets: [dataset],
        },
        options: {
            responsive: true,
            plugins: {
                legend: { display: true },
            },
            scales: {
                x: { title: { display: true, text: 'X Values' } },
                y: { title: { display: true, text: 'Y Values' } },
            },
        },
    });
}


        function clearInput(inputId) {
            document.getElementById(inputId).value = '';
        }
    </script>
</body>
</html>
