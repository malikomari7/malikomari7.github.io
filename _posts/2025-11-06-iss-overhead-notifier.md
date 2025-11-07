---
layout: single
title: "Day 33 – ISS Overhead Notifier"
subtitle: "A real-time Python script that looks to the skies"
date: 2025-11-06
author_profile: true
tags: [Python, API, Automation, 100DaysOfCode, Space, Notification]
---

### Overview

For **Day 33** of my #100DaysOfCode challenge, I built a project that literally looks to the skies.

Using Python, I created a **real-time ISS Overhead Notifier** — a script that checks when the International Space Station (ISS) is flying overhead and alerts me if it’s visible in the night sky.

This project combines **API requests**, **geolocation**, and **email automation**, giving me practical experience in event-driven programming and asynchronous checks.

---

### Code

```python
import requests
from datetime import datetime
import smtplib
import time

MY_EMAIL = "your_email@example.com"
MY_PASSWORD = "your_password"
MY_LAT = 51.507351     # Example: London latitude
MY_LONG = -0.127758    # Example: London longitude

def is_iss_overhead():
    """Return True if the ISS is within ±5° of your position."""
    response = requests.get(url="http://api.open-notify.org/iss-now.json")
    response.raise_for_status()
    data = response.json()

    iss_latitude = float(data["iss_position"]["latitude"])
    iss_longitude = float(data["iss_position"]["longitude"])

    return (MY_LAT - 5 <= iss_latitude <= MY_LAT + 5) and \
           (MY_LONG - 5 <= iss_longitude <= MY_LONG + 5)

def is_night():
    """Return True if it’s currently nighttime at your location."""
    parameters = {"lat": MY_LAT, "lng": MY_LONG, "formatted": 0}
    response = requests.get("https://api.sunrise-sunset.org/json", params=parameters)
    response.raise_for_status()
    data = response.json()

    sunrise = int(data["results"]["sunrise"].split("T")[1].split(":")[0])
    sunset = int(data["results"]["sunset"].split("T")[1].split(":")[0])

    time_now = datetime.utcnow().hour
    return time_now >= sunset or time_now <= sunrise

while True:
    time.sleep(60)  # Check every minute
    if is_iss_overhead() and is_night():
        with smtplib.SMTP("smtp.gmail.com", port=587) as connection:
            connection.starttls()
            connection.login(MY_EMAIL, MY_PASSWORD)
            connection.sendmail(
                from_addr=MY_EMAIL,
                to_addrs=MY_EMAIL,
                msg="Subject:Look Up 👆\n\nThe ISS is above you in the sky!"
            )
```
How It Works

ISS API — Fetches the current latitude and longitude of the ISS.

Sunrise-Sunset API — Determines if it’s nighttime at your coordinates.

Email Notification — Sends an alert when both conditions are true.

Loop & Delay — Runs every 60 seconds, checking for visibility.

Lessons Learned

Handling multiple APIs efficiently and parsing JSON data.

Using datetime and UTC conversions to align time zones.

Writing modular functions to separate logic (visibility vs. notification).

Automating periodic checks with time.sleep().

Next Steps

Integrate SMS notifications via Twilio.

Use scheduling (schedule or cron) instead of infinite loops.

Containerize it with Docker for consistent background execution.
