---
title: Welcome to my blog
---
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Number Input with Chart</title>
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
        }

        canvas {
            max-width: 600px;
            margin: 0 auto;
        }
    </style>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
    <h1>Enter Numbers and Plot</h1>
    <p>Paste numbers in the boxes below and click "Submit" to process and plot the graph.</p>

    <div class="input-section">
        <div>
            <input type="text" id="numberInput1" placeholder="Input 1 (comma-separated)">
            <br>
            <button onclick="plotGraph()">Plot</button>
            <button class="clear-button" onclick="clearInput('numberInput1', 'output1')">Clear</button>
        </div>
    </div>

    <div id="chart-container">
        <canvas id="myChart"></canvas>
    </div>

    <script>
        let chart; // To store the Chart.js instance

        function plotGraph() {
            const inputBox = document.getElementById('numberInput1');
            const inputValue = inputBox.value.trim();
            if (!inputValue) {
                alert('Please enter some numbers.');
                return;
            }

            // Parse input into an array of numbers
            const numbers = inputValue.split(',').map(num => parseFloat(num.trim()));
            if (numbers.some(isNaN)) {
                alert('Please ensure all inputs are valid numbers.');
                return;
            }

            // Generate labels for the graph (e.g., 1, 2, 3...)
            const labels = numbers.map((_, index) => index + 1);

            // Destroy the previous chart instance if it exists
            if (chart) {
                chart.destroy();
            }

            // Create the chart
            const ctx = document.getElementById('myChart').getContext('2d');
            chart = new Chart(ctx, {
                type: 'line', // You can change this to 'bar', 'scatter', etc.
                data: {
                    labels: labels,
                    datasets: [{
                        label: 'Input Data',
                        data: numbers,
                        borderColor: 'rgba(75, 192, 192, 1)',
                        backgroundColor: 'rgba(75, 192, 192, 0.2)',
                        borderWidth: 2,
                        fill: true,
                        tension: 0.4, // Smooth the curve
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            display: true
                        }
                    },
                    scales: {
                        x: {
                            title: {
                                display: true,
                                text: 'Index'
                            }
                        },
                        y: {
                            title: {
                                display: true,
                                text: 'Value'
                            }
                        }
                    }
                }
            });
        }

        function clearInput(inputId, outputId) {
            const inputBox = document.getElementById(inputId);
            inputBox.value = '';
        }
    </script>
</body>
</html>
