---
title: Welcome to my blog
---

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Number Input Website</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 50px;
        }

        input[type="text"] {
            width: 300px;
            padding: 10px;
            margin: 10px 0;
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
    </style>
</head>
<body>
    <h1>Enter Numbers</h1>
    <p>Paste numbers in the box below and click "Submit" to process.</p>
    <input type="text" id="numberInput" placeholder="Paste your numbers here">
    <br>
    <button onclick="processNumbers()">Submit</button>
    <button class="clear-button" onclick="clearInput()">Clear</button>

    <div id="output" style="margin-top: 20px; font-size: 18px;"></div>

    <script>
        function processNumbers() {
            const inputBox = document.getElementById('numberInput');
            const outputDiv = document.getElementById('output');

            // Get the input value
            const numbers = inputBox.value.trim();

            if (numbers) {
                outputDiv.textContent = `You entered: ${numbers}`;
            } else {
                outputDiv.textContent = 'Please enter some numbers.';
            }
        }

        function clearInput() {
            const inputBox = document.getElementById('numberInput');
            const outputDiv = document.getElementById('output');

            // Clear the input box and output div
            inputBox.value = '';
            outputDiv.textContent = '';
        }
    </script>
</body>
</html>
