# Aegis-AI-
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Aegis - Intelligent Password Security Suite</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>
    <div class="container">
        <h1>Aegis</h1>
        <p>Your Intelligent Password Security Suite</p>
        <div class="password-container">
            <input type="password" id="password-input" placeholder="Enter a password to check">
            <button id="check-btn">Check Strength</button>
        </div>
        <div id="score-container">
            <div id="score-circle">
                <span id="score-text">0</span>
            </div>
        </div>
        <div id="results-container">
            
        </div>
        <div class="generation-container">
            <h2>Generate a Secure Password</h2>
            <div class="password-container">
                <input type="text" id="generated-password" readonly>
                <button id="generate-btn">Generate</button>
                <button id="copy-btn">Copy</button>
            </div>
        </div>
    </div>
    <script src="{{ url_for('static', filename='script.js') }}"></script>
</body>
</html>

