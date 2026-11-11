# HR Onboarding Risk Automation Bot

A Python bot that decides whether new hires should onboard **in person or remotely** at each of 5 US office locations, based on live weather and air-quality forecasts. It runs without any manual input or API keys.

**Stack:** Python · REST APIs (Open-Meteo, JSONPlaceholder) · pytest · HTML reporting

---

## The problem

An HR team runs onboarding sessions in **Minneapolis, Chicago, New York, Austin, and Denver**. Every week someone checks the forecast for each city by hand and decides whether incoming employees should commute in. This bot automates that decision.

## What it does

```
[Fetch]                    →  [Transform]                 →  [Output]
7-day weather forecast        °C → °F conversion             onboarding_weather_data.csv
7-day air quality (UV, PM10)  flag each day HIGH/LOW risk    onboarding_hire_report.csv
new-hire list                 7-day risk score per city      onboarding_report.html
                              rank cities, assign hires
```

1. **Fetch** live data from 3 public APIs for every city.
2. **Transform** it with business rules: a day is **HIGH RISK** if *any one* threshold is exceeded:

   | Condition | Threshold |
   |---|---|
   | Too cold (daily high) | below 20°F |
   | Too hot | above 95°F |
   | Heavy precipitation | over 20 mm |
   | High wind | over 50 km/h |
   | High UV index | above 8 |
   | Poor air quality (PM10) | above 50 µg/m³ |

   A city's **risk score** is the % of the next 7 days that are high risk. Scores **above 40%** trigger a **REMOTE** recommendation.
3. **Output** three ready-to-share files:
   - `onboarding_weather_data.csv`: 35 rows (5 cities × 7 days) with a risk flag per day
   - `onboarding_hire_report.csv`: one row per new hire with an assigned city and a recommendation
   - `onboarding_report.html`: a self-contained executive report with an auto-written summary, city risk rankings, and per-hire decisions

## Engineering details

- **Fault tolerant:** every API call has a 10-second timeout and response-key validation. If a call fails, the pipeline continues with safe fallback values and logs `[FAILED]` rather than crashing.
- **Configurable:** all thresholds live as constants at the top of the script, so HR can change policy without touching logic.
- **Tested:** **32 unit tests** cover unit conversion, risk flagging (including exact boundary values), city scoring, hire assignment, data merging, and API success and failure paths using mocked responses.

## Run it

```bash
pip install requests pytest
python onboarding_weather_bot.py      # generates the 3 output files
python -m pytest tests/ -v            # runs the 32 unit tests
```

## Project structure

```
├── onboarding_weather_bot.py      # fetch → transform → output pipeline
├── tests/
│   ├── test_transforms.py         # business-logic tests (24)
│   └── test_fetch.py              # API fetch tests with mocks (8)
├── onboarding_weather_data.csv    # sample output
├── onboarding_hire_report.csv     # sample output
└── onboarding_report.html         # sample output
```

## What I'd add next

- **Anomaly detection** on rolling weather history to catch unusual conditions that fixed thresholds miss
- **Longer-range planning** using historical weather data, so HR can schedule onboarding cohorts weeks ahead
- **A scheduler** (e.g., GitHub Actions cron) to run the bot weekly and email the HTML report automatically
