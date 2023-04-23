# Tadenex-Forex-capital-
from flask import Flask, render_template, request, redirect, url_for
import pandas as pd
import numpy as np

app = Flask(__name__)
app.secret_key = 'secret'

# Load data
df = pd.read_csv('investment_data.csv')

# Define routes
@app.route('/')
def home():
    return render_template('home.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']

        if username == 'admin' and password == 'admin':
            session['logged_in'] = True
            return redirect(url_for('recommend'))
        else:
            return render_template('login.html', error=True)

    return render_template('login.html', error=False)

@app.route('/logout')
def logout():
    session['logged_in'] = False
    return redirect(url_for('home'))

@app.route('/recommend', methods=['GET', 'POST'])
def recommend():
    if not session.get('logged_in'):
        return redirect(url_for('login'))

    # Get user input
    age = int(request.form['age'])
    income = float(request.form['income'])
    risk_tolerance = request.form['risk_tolerance']

    # Filter data based on user input
    filtered_df = df[(df['Age'] <= age) & (df['Income'] <= income) & (df['Risk Tolerance'] == risk_tolerance)]

    # Calculate recommended investment based on filtered data
    recommended_investment = filtered_df.groupby('Investment Type')['Expected Return'].max().idxmax()

    # Render recommendation template with recommended investment
    return render_template('recommendation.html', investment=recommended_investment)

if __name__ == '__main__':
    app.run(debug=True)
{% extends "base.html" %}

{% block content %}
  <h1>Login</h1>

  {% if error %}
    <p class="error">Incorrect username or password</p>
  {% endif %}

  <form method="post" action="{{ url_for('login') }}">
    <label for="username">Username:</label>
    <input type="text" name="username" id="username" required>

    <label for="password">Password:</label>
    <input type="password" name="password" id="password" required>

    <input type="submit" value="Login">
  </form>
{% endblock %}
{% extends "base.html" %}

{% block content %}
  <h1>Tadenex Forex</h1>
  <p>Invest with confidence and get high returns on your investments.</p>

  <h2>Capital Plus Investment Packages</h2>
  <p>Our investment packages start from $100 and offer high returns with minimal risk. Choose a package that suits your investment goals and budget.</p>

  <ul>
    <li>Basic Package - $100 to $999</li>
    <li>Silver Package - $1,000 to $4,999</li>
    <li>Gold Package - $5,000 to $9,999</li>
    <li>Platinum Package - $10,000 and above</li>
  </ul>

  <a href="{{ url_for('login') }}">Login</a>
{% endblock %}
