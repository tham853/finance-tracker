# finance-tracker
Create a personal finance tracker that helps users manage their income, expenses, and savings. The application should allow users to add, edit, and delete transactions, categorize them, and generate monthly reports.
Setup the Project

Initialize a new Python project
Set up a virtual environment
Install necessary libraries (e.g., Flask, SQLAlchemy, matplotlib)
User Authentication

python
Copy code
from flask import Flask, render_template, redirect, url_for, request, session
from flask_sqlalchemy import SQLAlchemy
from werkzeug.security import generate_password_hash, check_password_hash

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your_secret_key'
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///finance.db'
db = SQLAlchemy(app)

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(150), unique=True, nullable=False)
    password = db.Column(db.String(150), nullable=False)

@app.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'POST':
        username = request.form['username']
        password = generate_password_hash(request.form['password'], method='sha256')
        new_user = User(username=username, password=password)
        db.session.add(new_user)
        db.session.commit()
        return redirect(url_for('login'))
    return render_template('register.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        user = User.query.filter_by(username=username).first()
        if user and check_password_hash(user.password, password):
            session['user_id'] = user.id
            return redirect(url_for('dashboard'))
    return render_template('login.html')

if __name__ == '__main__':
    db.create_all()
    app.run(debug=True)
Transaction Management

python
Copy code
class Transaction(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    date = db.Column(db.Date, nullable=False)
    category = db.Column(db.String(50), nullable=False)
    amount = db.Column(db.Float, nullable=False)
    description = db.Column(db.String(200))

@app.route('/add_transaction', methods=['GET', 'POST'])
def add_transaction():
    if request.method == 'POST':
        date = request.form['date']
        category = request.form['category']
        amount = request.form['amount']
        description = request.form['description']
        new_transaction = Transaction(user_id=session['user_id'], date=date, category=category, amount=amount, description=description)
        db.session.add(new_transaction)
        db.session.commit()
        return redirect(url_for('dashboard'))
    return render_template('add_transaction.html')

@app.route('/dashboard')
def dashboard():
    user_id = session['user_id']
    transactions = Transaction.query.filter_by(user_id=user_id).all()
    return render_template('dashboard.html', transactions=transactions)
Generate Reports and Visualization

python
Copy code
import matplotlib.pyplot as plt
from io import BytesIO
import base64

@app.route('/report')
def report():
    user_id = session['user_id']
    transactions = Transaction.query.filter_by(user_id=user_id).all()
    categories = {}
    for transaction in transactions:
        if transaction.category in categories:
            categories[transaction.category] += transaction.amount
        else:
            categories[transaction.category] = transaction.amount

            register.html
html
Copy code
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Register</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <h2>Register</h2>
    <form action="/register" method="post">
        <label for="username">Username:</label>
        <input type="text" id="username" name="username" required>
        <label for="password">Password:</label>
        <input type="password" id="password" name="password" required>
        <button type="submit">Register</button>
    </form>
    <p>Already have an account? <a href="/login">Login here</a></p>
</body>
</html>
login.html
html
Copy code
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <h2>Login</h2>
    <form action="/login" method="post">
        <label for="username">Username:</label>
        <input type="text" id="username" name="username" required>
        <label for="password">Password:</label>
        <input type="password" id="password" name="password" required>
        <button type="submit">Login</button>
    </form>
    <p>Don't have an account? <a href="/register">Register here</a></p>
</body>
</html>
add_transaction.html
html
Copy code
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Add Transaction</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <h2>Add Transaction</h2>
    <form action="/add_transaction" method="post">
        <label for="date">Date:</label>
        <input type="date" id="date" name="date" required>
        <label for="category">Category:</label>
        <input type="text" id="category" name="category" required>
        <label for="amount">Amount:</label>
        <input type="number" id="amount" name="amount" step="0.01" required>
        <label for="description">Description:</label>
        <input type="text" id="description" name="description">
        <button type="submit">Add Transaction</button>
    </form>
    <p><a href="/dashboard">Back to Dashboard</a></p>
</body>
</html>
dashboard.html
html
Copy code
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <h2>Dashboard</h2>
    <p>Welcome, {{ user.username }}</p>
    <p><a href="/add_transaction">Add New Transaction</a> | <a href="/report">View Report</a> | <a href="/logout">Logout</a></p>
    <table>
        <thead>
            <tr>
                <th>Date</th>
                <th>Category</th>
                <th>Amount</th>
                <th>Description</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            {% for transaction in transactions %}
            <tr>
                <td>{{ transaction.date }}</td>
                <td>{{ transaction.category }}</td>
                <td>{{ transaction.amount }}</td>
                <td>{{ transaction.description }}</td>
                <td>
                    <a href="/edit_transaction/{{ transaction.id }}">Edit</a>
                    <a href="/delete_transaction/{{ transaction.id }}" onclick="return confirm('Are you sure you want to delete this transaction?');">Delete</a>
                </td>
            </tr>
            {% endfor %}
        </tbody>
    </table>
</body>
</html>
report.html
html
Copy code
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Monthly Report</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <h2>Monthly Report</h2>
    <p><a href="/dashboard">Back to Dashboard</a></p>
    <div>
        <img src="data:image/png;base64,{{ image }}" alt="Expense Report">
    </div>
</body>
</html>
These templates use a simple and consistent design. You may want to add some CSS to styles.css for better styling. Here’s a basic example:

styles.css
css
Copy code
body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
    background-color: #f4f4f4;
}

h2 {
    color: #333;
}

form {
    max-width: 400px;
    margin: 50px auto;
    padding: 20px;
    background: #fff;
    border-radius: 8px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

form label {
    display: block;
    margin-bottom: 8px;
}

form input {
    width: 100%;
    padding: 8px;
    margin-bottom: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
}

form button {
    width: 100%;
    padding: 10px;
    background-color: #007bff;
    color: #fff;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

form button:hover {
    background-color: #0056b3;
}

table {
    width: 80%;
    margin: 20px auto;
    border-collapse: collapse;
}

table, th, td {
    border: 1px solid #ddd;
}

th, td {
    padding: 8px;
    text-align: left;
}

th {
    background-color: #f2f2f2;
}

a {
    color: #007bff;
    text-decoration: none;
}

a:hover {
    text-decoration: underline;
}

    # Generate Pie Chart
    fig, ax = plt.subplots()
    ax.pie(categories.values(), labels=categories.keys(), autopct='%1.1f%%')
    buf = BytesIO()
    plt.savefig(buf, format='png')
    buf.seek(0)
    image = base64.b64encode(buf.getvalue()).decode('utf-8')

    return render_template('report.html', image=image)
