# Perth Morning Ride Check

A simple web page that tells you whether the next 7 mornings in Perth, WA are good for riding, walking, or staying home for a coffee.

**Live page:** https://YOUR-USERNAME.github.io/YOUR-REPO-NAME/

## What it does

For each of the next 7 days it looks at the forecast between 6:00 AM and 10:00 AM (Perth time) and shows:

- Weather summary and average temperature
- Peak chance of rain
- Average wind speed and the strongest gust
- A verdict for the morning

## How the verdict works

| Verdict | Peak rain chance | Average wind |
| --- | --- | --- |
| Ride | 20% or less | 25 km/h or less |
| Walk | 35% or less | 30 km/h or less |
| Cafe | Anything worse | Anything worse |

## Using it on your phone

Open the live page in Chrome on Android, tap the menu (three dots), then choose **Add to Home screen**. It will sit on your home screen like an app.

## Changing the settings

Everything is in `index.html`. Near the top of the `<script>` section you'll find:

- `LAT`, `LON`, `TZ`: the location and timezone
- `CYCLE_RAIN_MAX`, `CYCLE_WIND_MAX`: the riding thresholds
- `WALK_RAIN_MAX`, `WALK_WIND_MAX`: the walking thresholds
- `ICONS`: what is shown for each verdict (emoji for now, can be swapped for image files)

## Data

Forecast data comes from [Open-Meteo](https://open-meteo.com/), which is free for non-commercial use. The page fetches the data directly in your browser each time it is opened, so there is no server and no API key.
