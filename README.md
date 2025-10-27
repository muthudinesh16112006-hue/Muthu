from flask import Flask, render_template, request, redirect, url_for
import sqlite3
from datetime import datetime

app = Flask(__name__)

# Initialize database
def init_db():
    conn = sqlite3.connect('clinic.db')
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS doctors(
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    name TEXT, specialty TEXT)''')
    c.execute('''CREATE TABLE IF NOT EXISTS appointments(
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    patient_name TEXT,
                    doctor_id INTEGER,
                    date TEXT,
                    time TEXT,
                    FOREIGN KEY(doctor_id) REFERENCES doctors(id))''')
    conn.commit()
    conn.close()

@app.route('/')
def index():
    conn = sqlite3.connect('clinic.db')
    c = conn.cursor()
    c.execute("SELECT * FROM doctors")
    doctors = c.fetchall()
    conn.close()
    return render_template('index.html', doctors=doctors)

@app.route('/add_doctor', methods=['POST'])
def add_doctor():
    name = request.form['name']
    specialty = request.form['specialty']
    conn = sqlite3.connect('clinic.db')
    c = conn.cursor()
    c.execute("INSERT INTO doctors (name, specialty) VALUES (?, ?)", (name, specialty))
    conn.commit()
    conn.close()
    return redirect(url_for('index'))

@app.route('/book', methods=['POST'])
def book():
    patient_name = request.form['patient_name']
    doctor_id = request.form['doctor_id']
    date = request.form['date']
    time = request.form['time']

    conn = sqlite3.connect('clinic.db')
    c = conn.cursor()
    c.execute('''INSERT INTO appointments (patient_name, doctor_id, date, time)
                 VALUES (?, ?, ?, ?)''', (patient_name, doctor_id, date, time))
    conn.commit()
    conn.close()
    return redirect(url_for('appointments'))

@app.route('/appointments')
def appointments():
    conn = sqlite3.connect('clinic.db')
    c = conn.cursor()
    c.execute('''SELECT a
