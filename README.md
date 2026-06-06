# Group-5----NetAd
# 🎥 WatchmeWhip

> A web-based CCTV surveillance monitoring system with user authentication, live video streaming, and activity logging.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Environment Variables](#environment-variables)
  - [Running Locally](#running-locally)
- [Usage](#usage)
- [API Endpoints](#api-endpoints)
- [Database Models](#database-models)
- [Deployment](#deployment)
- [Security Notes](#security-notes)

---

## Overview

**WatchmeWhip** is a Flask-based surveillance web application designed for monitoring CCTV camera feeds in real-time. It features a secure login system, live video streaming, session-based access control, and a full monitoring log dashboard that tracks user activity, login attempts, and intrusion warnings.

---

## Features

- 🔐 **Secure Login** — Email/password authentication with bcrypt hashing
- 📹 **Live Video Feed** — Real-time MJPEG stream from IP camera via OpenCV
- 🛡️ **Intrusion Detection Logging** — Logs failed login attempts with device/browser info and IP address
- 📊 **Monitoring Logs Dashboard** — Full table view of all user sessions and actions
- 🕵️ **Activity Tracking** — Logs every user action (pause feed, resume feed, motion simulation, etc.)
- 🌐 **Real IP Detection** — Resolves public IP via headers (`X-Forwarded-For`, `CF-Connecting-IP`) and frontend fallback
- 📱 **Responsive UI** — Clean, minimal dashboard built with vanilla HTML/CSS/JS

---

## Tech Stack

| Layer      | Technology                        |
|------------|-----------------------------------|
| Backend    | Python, Flask                     |
| Database   | PostgreSQL + Flask-SQLAlchemy     |
| Auth       | Flask-Bcrypt, Flask Sessions      |
| Video      | OpenCV (`opencv-python-headless`) |
| Frontend   | HTML5, CSS3, Vanilla JavaScript   |
| Deployment | Gunicorn, Heroku (via Procfile)   |

---

## Project Structure

```
watchmewhip/
├── app.py                  # Main Flask application
├── requirements.txt        # Python dependencies
├── Procfile                # Gunicorn process config (Heroku)
├── templates/
│   └── index.html          # Single-page app (login + dashboard + logs)
├── static/
│   ├── login.css           # All styles
│   └── login.js            # Client-side logic
└── README.md
```

---

## Getting Started

### Prerequisites

- Python 3.8+
- PostgreSQL
- pip

### Installation

1. **Clone the repository**

```bash
git clone https://github.com/your-username/watchmewhip.git
cd watchmewhip
```

2. **Create and activate a virtual environment**

```bash
python -m venv venv
source venv/bin/activate       # On Windows: venv\Scripts\activate
```

3. **Install dependencies**

```bash
pip install -r requirements.txt
```

### Environment Variables

Create a `.env` file or set the following environment variables:

| Variable       | Description                              | Default                                                    |
|----------------|------------------------------------------|------------------------------------------------------------|
| `DATABASE_URL` | PostgreSQL connection string             | `postgresql://postgres:group5@localhost:5432/watchmewhip`  |
| `STREAM_URL`   | IP camera MJPEG stream URL               | `http://admin:password@192.168.x.x:8080/stream/getvideo`  |
| `PORT`         | Port to run the Flask app on             | `5000`                                                     |

> ⚠️ **Important:** Do not commit real credentials to version control. Use environment variables or a secrets manager.

### Running Locally

1. **Create the PostgreSQL database**

```sql
CREATE DATABASE watchmewhip;
```

2. **Start the application**

```bash
python app.py
```

3. **Visit** `http://localhost:5000`

> The database tables are auto-created on first run via `db.create_all()`.

---

## Usage

1. **Login** — Enter your registered email and password on the login screen.
2. **Dashboard** — View the live camera feed, simulate motion alerts, and pause/resume the stream.
3. **Monitoring Logs** — Navigate to the logs page to see all session activity, including login times, actions performed, IP addresses, and intrusion attempts.
4. **Logout** — Ends the session and logs the logout time.

### Adding Users (via DB)

Users must be added directly to the database with a bcrypt-hashed password:

```python
from app import app, db, bcrypt
from app import User

with app.app_context():
    hashed = bcrypt.generate_password_hash("yourpassword").decode('utf-8')
    user = User(username="admin@example.com", password_hash=hashed, role="admin")
    db.session.add(user)
    db.session.commit()
```

---

## API Endpoints

| Method | Endpoint                  | Auth Required | Description                            |
|--------|---------------------------|---------------|----------------------------------------|
| `GET`  | `/`                       | No            | Serves the main HTML page              |
| `POST` | `/api/auth/login`         | No            | Authenticates user, creates session    |
| `POST` | `/api/auth/logout`        | Yes           | Clears session, logs logout time       |
| `POST` | `/api/auth/track_action`  | Yes           | Logs a specific user action            |
| `GET`  | `/api/logs`               | Yes           | Returns all monitoring logs            |
| `GET`  | `/api/stream/verify`      | Yes           | Verifies session before loading stream |
| `GET`  | `/video_feed`             | Yes           | Streams MJPEG video frames             |

---

## Database Models

### `users`
| Column          | Type    | Description              |
|-----------------|---------|--------------------------|
| `id`            | Integer | Primary key              |
| `username`      | String  | User email/username      |
| `password_hash` | Text    | Bcrypt-hashed password   |
| `role`          | String  | `user` or `admin`        |

### `logs`
| Column       | Type    | Description                            |
|--------------|---------|----------------------------------------|
| `id`         | Integer | Primary key                            |
| `user_email` | String  | Email of the acting user               |
| `time_in`    | String  | Time of action/login                   |
| `time_out`   | String  | Time of logout (if applicable)         |
| `date`       | String  | Date of action                         |
| `ip_address` | String  | IP address of the client               |
| `action`     | Text    | Description of the action performed    |

---

## Deployment

This project includes a `Procfile` for Heroku deployment:

```
web: gunicorn app:app
```

**Steps:**

```bash
heroku create
heroku addons:create heroku-postgresql:mini
heroku config:set STREAM_URL="your_camera_stream_url"
git push heroku main
```

---

## Security Notes

> This project was built for academic/group purposes. Before deploying in a production environment, consider the following:

- 🔑 Replace the hardcoded `secret_key` in `app.py` with a randomly generated secret stored in an environment variable.
- 🔒 Move camera credentials out of the default `STREAM_URL` fallback — use `STREAM_URL` env var exclusively.
- 🛑 Add rate limiting on `/api/auth/login` to prevent brute-force attacks.
- 🔐 Use HTTPS in production (e.g., via Heroku, Nginx, or Cloudflare).
- 👤 Implement a proper user registration/admin panel instead of manual DB inserts.

---

## 👥 Authors

**Group 5** — Network Administration Project

---

> Built with Flask, PostgreSQL, and OpenCV.
