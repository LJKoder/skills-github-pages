<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Plot Data with Error Bars</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 50px;
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
    </style>
    <!-- Chart.js Library -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Chart.js Error Bars Plugin -->
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-error-bars"></script>
</head>
<body>
    <h1>Plot Data with Error Bars</h1>
    <button onclick="plotGraph()">Plot Data with Error Bars</button>

    <div id="chart-container">
        <canvas id="myChart"></canvas>
    </div>

    <script>
        let chart; // To store the Chart.js instance

        // Plot the graph with error bars
        function plotGraph() {
            const canvas = document.getElementById('myChart');
            const ctx = canvas.getContext('2d');

            // Destroy the previous chart instance if it exists
            if (chart) {
                chart.destroy();
            }

            // Hardcoded dataset with error bars
            const dataset = {
                label: 'Dataset with Error Bars',
                data: [
                    { x: 1, y: 2, xMin: 0.5, xMax: 1.5, yMin: 1.5, yMax: 2.5 },
                    { x: 2, y: 3, xMin: 1.8, xMax: 2.2, yMin: 2.8, yMax: 3.2 },
                    { x: 3, y: 4, xMin: 2.7, xMax: 3.3, yMin: 3.5, yMax: 4.5 },
                ],
                backgroundColor: 'rgba(54, 162, 235, 0.5)',
                borderColor: 'rgba(54, 162, 235, 1)',
                errorBarColor: 'rgba(0, 0, 0, 0.8)', // Error bar color
                type: 'scatter', // Ensure we are using scatter chart type
                errorBars: {
                    mode: 'both', // 'both', 'x', 'y'
                    xMinKey: 'xMin',
                    xMaxKey: 'xMax',
                    yMinKey: 'yMin',
                    yMaxKey: 'yMax',
                    color: 'rgba(0, 0, 0, 0.8)',
                },
            };

            // Create the chart with the dataset
            chart = new Chart(ctx, {
                type: 'scatter', // Use scatter chart type for error bars
                data: {
                    datasets: [dataset],
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: { display: true },
                    },
                    scales: {
                        x: {
                            title: { display: true, text: 'X Values' },
                        },
                        y: {
                            title: { display: true, text: 'Y Values' },
                        },
                    },
                },
            });
        }
    </script>
</body>
</html>
