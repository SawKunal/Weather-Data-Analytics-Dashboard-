# Weather-Data-Analytics-Dashboard-
# Global Weather Dashboard (Power BI)

![Dashboard Overview](assets/overview.png)

An interactive weather analytics dashboard built in **Power BI**, leveraging data from **WeatherAPI.com**. It provides a comprehensive view of current and forecasted weather conditions for major global cities—supporting environmental monitoring, strategic planning, and educational analysis.

---

## 🔗 Quick Links
- **Live PBIX / Releases:** `link-to-releases-or-drive-if-any`
- **Sample Dataset (optional):** `link-if-any`

---

## ✨ Key Features
- **Weather Data Integration:** Data sourced from WeatherAPI for cities such as **Cairo**, **Mumbai**, **Reykjavik**, and more.  
- **Metrics Visualized:** Temperature (°C), Humidity (%), Wind Speed (km/h or mph), Precipitation (mm), and **7-Day Forecast** with trends.  
- **Sunrise & Sunset Tracking:** Visualizes daylight cycles for planning around energy use or outdoor activities.  
- **Rain Probability Modeling:** Custom **DAX** logic to estimate and visualize likelihood of rain for events and logistics.  
- **Time Series Forecasting:** Predictive charts using Power BI’s built-in analytics on historical + forecasted data.

---

## 🧰 Tools & Technologies
- **Power BI Desktop**
- **DAX (Data Analysis Expressions)**
- **Power Query (M)**
- **WeatherAPI.com**
- Time series forecasting techniques (Power BI analytics)

---

## 🗂️ Repo Structure
Global-Weather-Dashboard/
├─ assets/
│ └─ overview.png # ← Add one screenshot here (used in README)
├─ powerbi/
│ └─ GlobalWeatherDashboard.pbix
├─ queries/
│ └─ weather-api-query.m # Power Query (M) examples
├─ measures/
│ └─ dax-measures.txt # DAX snippets used in the model
└─ README.md
🌐 Cities Covered (sample)

Cairo • Mumbai • Reykjavik • London • New York • Tokyo • Sydney • São Paulo
(Extend by adding city names to the parameter list / table.)

🔌 Power Query (M) – API Ingestion (Example)

Create two parameters in Power Query:

WeatherApiKey (Text)

CityList (List) — e.g., {"Cairo","Mumbai","Reykjavik"}

Current Conditions (M):

let
  Cities      = CityList,
  BaseUrl     = "http://api.weatherapi.com/v1/current.json?key=" & WeatherApiKey & "&q=",
  GetCity     = (city as text) =>
      let
        Source   = Json.Document(Web.Contents(BaseUrl & Uri.EscapeDataString(city))),
        Loc      = Source[location],
        Cur      = Source[current],
        Record   = [
          City = Loc[name],
          Country = Loc[country],
          Lat = Loc[lat],
          Lon = Loc[lon],
          LocalTime = Loc[localtime],
          TempC = Cur[temp_c],
          Humidity = Cur[humidity],
          WindKph = Cur[wind_kph],
          PrecipMm = Cur[precip_mm],
          Condition = Cur[condition][text],
          Cloud = Cur[cloud],
          IsDay = Cur[is_day]
        ]
      in
        Record,
  TableOut   = Table.FromRecords(List.Transform(Cities, each GetCity(_)))
in
  TableOut


Forecast (7-Day) (M):

let
  Cities    = CityList,
  Days      = "7",
  BaseUrl   = "http://api.weatherapi.com/v1/forecast.json?key=" & WeatherApiKey & "&days=" & Days & "&aqi=no&alerts=no&q=",
  GetCity   = (city as text) =>
      let
        Source   = Json.Document(Web.Contents(BaseUrl & Uri.EscapeDataString(city))),
        Loc      = Source[location],
        Fcst     = Source[forecast][forecastday],
        Rows     = List.Transform(Fcst, each [
                    City = Loc[name],
                    Country = Loc[country],
                    Date = _[date],
                    MaxTempC = _[day][maxtemp_c],
                    MinTempC = _[day][mintemp_c],
                    AvgTempC = _[day][avgtemp_c],
                    DailyChanceOfRain = _[day][daily_chance_of_rain],
                    TotalPrecipMm = _[day][totalprecip_mm],
                    Condition = _[day][condition][text],
                    Sunrise = _[astro][sunrise],
                    Sunset = _[astro][sunset]
                  ])
      in
        Rows,
  Flatten  = List.Combine(List.Transform(Cities, each GetCity(_))),
  TableOut = Table.FromRecords(Flatten)
in
  TableOut


Note: WeatherAPI’s free tier has request limits—consider incremental refresh / scheduled refresh cadence accordingly.

🧮 DAX Snippets

1) Rain Probability (display as %):

Rain Probability % =
VAR BaseProb =
    AVERAGE('Forecast'[DailyChanceOfRain]) / 100
RETURN
FORMAT(BaseProb, "0.0%")


2) Simple “Likely to Rain” Flag:

Likely To Rain =
IF ( AVERAGE('Forecast'[DailyChanceOfRain]) >= 50, "Yes", "No" )


3) Temperature Trend (7-Day Avg):

Avg Temp (7D) =
AVERAGEX(
    DATESINPERIOD('Calendar'[Date], MAX('Calendar'[Date]), -7, DAY),
    AVERAGE('Forecast'[AvgTempC])
)


(For forecasting, use Power BI’s Analytics pane on line charts to add trend/forecast lines.)

📊 Recommended Visuals

Cards: Current Temp, Humidity, Wind, Precip

Line/Area Charts: 7-Day Temp, Rain Probability, Precipitation Trends

Bar/Column: City comparisons (current conditions)

Sunburst/Donut: Condition distribution (Clear/Cloudy/Rain)

Custom: Sunrise/Sunset (by city) for daylight comparison

🔒 Data & API Notes

Source: WeatherAPI.com (Current + Forecast endpoints)

Respect API usage limits; avoid excessive refreshes.

Consider parameterizing city lists and using incremental refresh for efficiency.

✅ Use Cases

City-level environmental awareness

Logistics & event planning

Weather trend analysis for education

Support for renewable energy and agriculture

🚀 Roadmap

 Add hourly forecasts

 Add severe weather alerts page

 Introduce RLS (if sharing internally)

 Optional: push data to a lakehouse for long-term history

🤝 Contributing

PRs welcome! Please open an issue for feature requests or bugs.

📜 License

MIT — see LICENSE
 for details.
