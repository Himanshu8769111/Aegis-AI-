# Aegis-AI-
from flask import Flask, render_template, request, jsonify
import re
import hashlib
import requests
import secrets
import string

app = Flask(__name__)

with open('aegis/common_passwords.txt', 'r') as f:
    common_passwords = [line.strip().lower() for line in f]
def generate_password():
    alphabet = string.ascii_letters + string.digits + string.punctuation
    while True:
        password = ''.join(secrets.choice(alphabet) for i in range(16))
        if (any(c.islower() for c in password)
                and any(c.isupper() for c in password)
                and any(c.isdigit() for c in password)
                and any(c in string.punctuation for c in password)):
            break
    return password

def is_password_pwned(password):
    """Checks if a password has been pwned using the Have I Been Pwned API."""
    sha1_password = hashlib.sha1(password.encode('utf-8')).hexdigest().upper()
    prefix, suffix = sha1_password[:5], sha1_password[5:]
    url = f'https://api.pwnedpasswords.com/range/{prefix}'
    
    try:
        response = requests.get(url)
        if response.status_code == 200:
            hashes = (line.split(':') for line in response.text.splitlines())
            for h, count in hashes:
                if h == suffix:
                    return int(count)
    except requests.RequestException:
        # If the API call fails, we'll just skip this check
        return 0
    return 0

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/check_password', methods=['POST'])
def check_password():
    password = request.json.get('password')
    feedback = []
    score = 0

    # 0. Common password check
    if password.lower() in common_passwords:
        feedback.append({'message': 'This is a very common password. Please choose something else.', 'type': 'error'})
        return jsonify({'score': 0, 'feedback': feedback})

    # Pwned password check
    pwned_count = is_password_pwned(password)
    if pwned_count > 0:
        feedback.append({'message': f'This password has appeared in data breaches {pwned_count} times. It is not secure.', 'type': 'error'})
        return jsonify({'score': 0, 'feedback': feedback})
    else:
        feedback.append({'message': 'This password has not been found in any known data breaches.', 'type': 'success'})


    # 1. Length check
    if len(password) >= 8:
        score += 1
        feedback.append({'message': 'Good length (8+ characters).', 'type': 'success'})
    else:
        feedback.append({'message': 'Password should be at least 8 characters long.', 'type': 'error'})
        return jsonify({'score': score, 'feedback': feedback})

    if len(password) >= 12:
        score += 1
        feedback.append({'message': 'Excellent length (12+ characters).', 'type': 'success'})


    # 2. Character variety check
    if re.search(r"[a-z]", password) and re.search(r"[A-Z]", password) and re.search(r"\d", password) and re.search(r"[\W_]", password):
        score += 2
        feedback.append({'message': 'Good mix of characters (uppercase, lowercase, numbers, and symbols).', 'type': 'success'})
    elif re.search(r"[a-zA-Z]", password) and re.search(r"\d", password):
        score += 1
        feedback.append({'message': 'Contains letters and numbers.', 'type': 'info'})
    else:
        feedback.append({'message': 'Consider adding a mix of uppercase, lowercase, numbers, and symbols.', 'type': 'error'})


    return jsonify({'score': score, 'feedback': feedback})

@app.route('/generate_password', methods=['GET'])
def generate_password_route():
    password = generate_password()
    return jsonify({'password': password})


if __name__ == '__main__':
    app.run(debug=True)

