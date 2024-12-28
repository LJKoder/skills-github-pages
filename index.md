<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Plot X and Y Data</title>
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
    <h1>Plot X and Y Data</h1>
    <p>Enter X and Y values (tab or newline-separated) to plot on the graph.</p>

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
    </div>

    <button onclick="plotGraph()">Plot</button>

    <div id="chart-container">
        <canvas id="myChart"></canvas>
    </div>

    <script>
        let chart; // To store the Chart.js instance

        function plotGraph() {
    const xInput = document.getElementById('xValues').value.trim();
    const yInput = document.getElementById('yValues').value.trim();

    if (!xInput || !yInput) {
        alert('Please enter both X and Y values.');
        return;
    }

    // Split by tabs or newlines to handle multiline Excel input
    const xValues = xInput.split(/\s+/).map(val => parseFloat(val.trim()));
    const yValues = yInput.split(/\s+/).map(val => parseFloat(val.trim()));

    if (xValues.some(isNaN) || yValues.some(isNaN)) {
        alert('Please ensure all inputs are valid numbers.');
        return;
    }

    if (xValues.length !== yValues.length) {
        alert('X and Y values must have the same length.');
        return;
    }

    // Get the context for the chart
    const ctx = document.getElementById('myChart').getContext('2d');

    // Destroy the previous chart instance if it exists
    if (chart) {
        chart.destroy();
    }

    // Create the new chart
    chart = new Chart(ctx, {
        type: 'scatter',
        data: {
            datasets: [{
                label: 'X vs Y Plot',
                data: xValues.map((x, i) => ({ x: x, y: yValues[i] })),
                borderColor: 'rgba(54, 162, 235, 1)',  // More neutral color
                backgroundColor: 'rgba(54, 162, 235, 0.2)',  // Slightly muted background
                borderWidth: 2,
                showLine: true, // Connect points with a line
                tension: 0.3 // A slight curve for the line
            }]
        },
        options: {
            responsive: true,
            plugins: {
                legend: {
                    display: true,
                    position: 'top',  // Legend at the top for a cleaner look
                    labels: {
                        font: {
                            family: 'Arial',  // Use a cleaner, scientific font
                            size: 14,  // Adjust the font size for clarity
                            weight: 'bold'  // Make the font a bit bolder for readability
                        }
                    }
                }
            },
            scales: {
                x: {
                    title: {
                        display: true,
                        text: 'X Values',
                        font: {
                            family: 'Arial',
                            size: 16,  // Slightly larger title
                            weight: 'bold'
                        },
                    },
                    grid: {
                        color: 'rgba(200, 200, 200, 0.5)',  // Lighter gridlines
                        lineWidth: 0.5  // Thinner gridlines for a less distracting effect
                    },
                    ticks: {
                        font: {
                            family: 'Arial',
                            size: 12
                        },
                        color: 'rgba(0, 0, 0, 0.7)'  // Darker tick marks for visibility
                    }
                },
                y: {
                    title: {
                        display: true,
                        text: 'Y Values',
                        font: {
                            family: 'Arial',
                            size: 16,
                            weight: 'bold'
                        },
                    },
                    grid: {
                        color: 'rgba(200, 200, 200, 0.5)',
                        lineWidth: 0.5
                    },
                    ticks: {
                        font: {
                            family: 'Arial',
                            size: 12
                        },
                        color: 'rgba(0, 0, 0, 0.7)'
                    }
                }
            },
            layout: {
                padding: 20  // Add some padding for clarity
            },
            elements: {
                point: {
                    radius: 5,  // Smaller point radius for a more professional look
                    hoverRadius: 7,  // Slightly larger on hover
                    borderWidth: 1  // Thin border around the points
                }
            },
            backgroundColor: 'white'  // Clean, scientific background color
        }
    });
}

        function clearInput(inputId) {
            const inputBox = document.getElementById(inputId);
            inputBox.value = '';
        }
    </script>
</body>
</html>
