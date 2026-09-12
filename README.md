# Barna Air Quality Improvement 🌫️

![Python Badge](https://img.shields.io/badge/Code-Python-8f7891?style=flat&logo=python&logoColor=white)
![Pandas Badge](https://img.shields.io/badge/Data-pandas-8f7891?style=flat&logo=pandas&logoColor=white)
![Jupyter Badge](https://img.shields.io/badge/Data-Jupyter-8f7891?style=flat&logo=jupyter&logoColor=white)
![Tableau Badge](https://img.shields.io/badge/Data-Tableau-8f7891?style=flat)

<a href="https://public.tableau.com/app/profile/rory.patrick.sheridan/viz/BarcelonaAirQualityImprovement/Dashboard1">
  <img align="left" width="42%" src="https://github.com/user-attachments/assets/7981855f-7c14-459e-aacb-e88fb3213de1" alt="Barcelona Air Quality Improvement dashboard">
</a>

<a href="https://public.tableau.com/app/profile/rory.patrick.sheridan/viz/BarcelonaAirQualityImprovement/Dashboard1">
  <img align="right" width="42%" src="https://github.com/user-attachments/assets/f1e25b7c-05f1-4755-8783-d73c4af62b9b" alt="Barcelona Air Quality Improvement dashboard">
</a>

<br clear="all">

## Data Pipeline Showcase

[Barcelona Air Quality Improvement][1] is a Tableau dashboard showing how Barcelona's policies have improved its air quality over a number of years. I built it using a dataset I chose myself: nitrogen dioxide (NO₂) readings from the city's monitoring stations, 2018 to 2024. Previously Barcelona had spent years above EU limits for nitrogen dioxide. I lived in Barcelona over a number of years, and was truly inspired by the effort and management invested into the improvement of the city's environment. In 2020 the city brought in a low emission zone, improved public transport and started pedestrianising the centre. This dashboard was built as a testament to that work and its successes.

Across the city, average NO₂ fell from 33 µg/m³ in 2019 to 21 µg/m³ in 2024. Levels dropped sharply in 2020 during lockdown and rose again in 2022, but by 2023 they had fallen below even the lockdown year, which points to a lasting change due to implemented policies.

This repo is the data side of the project. Barcelona changed how it publishes air quality data partway through, so the notebook here rebuilds two incompatible formats into one clean daily dataset that Tableau can use.

## The Dashboard

Two views, linked by the button in the top right. One focusing on when and where, the other on Improvements over time.

**Where and When Is Air Quality Worst?** — the spatial and seasonal picture, filtered by year and quarter. Uses a divergent colour scale keyed to the [WHO][12] and [EU][13] reference levels for NO₂: 10, 25 and 40 µg/m³. Blue is below, red is above, so severity reads at a glance without checking an axis.

- A map of average NO₂ by neighbourhood. Click an area to filter everything else.
- A density plot of daily averages per month, showing how a handful of extreme days can drag an average upwards.
- A bar chart of the share of days above 40 µg/m³, ranking the worst areas.
- A heat map of neighbourhood by month, which makes the winter spike obvious.

**Tracking Improvements over Time** — the same data as a trend, with a switch between a city overview and neighbourhood detail.

- A quarterly line chart, where the low emission zone, the pandemic and the 2022 rebound are all visible.
- Percentage change against 2019 as a baseline, per year.
- Average NO₂ against its standard deviation, showing that falling levels come with falling variability.
- Box plots by year: the interquartile range narrows and the outliers thin out.

## The Data

All data comes from [Open Data BCN][2], Barcelona City Council's open data portal:

- [Air quality data from the measure stations][3]: monthly CSV files of hourly pollutant readings.
- [Air quality measure stations][4]: station numbers, codes, names and coordinates.

The pipeline keeps NO₂ only, from eight monitoring stations: Ciutadella, Eixample, Gràcia, Observatori Fabra, Palau Reial, Poblenou, Sants and Vall d'Hebron. Readings run from 11 June 2018 to 31 December 2024, with a break in February and March 2019 while the city switched data source.

The output, `pollution_dataset.csv`, has one row per station per day:

| Column | Description |
| --- | --- |
| `date` | Date of the readings |
| `year`, `month`, `day` | Date parts, for filtering and grouping in Tableau |
| `estacio` | Station number |
| `station_name` | Station name |
| `lat`, `lon` | Station coordinates |
| `no2_daily_avg` | Average of that day's NO₂ readings, in µg/m³ |

## The Pipeline

Everything runs in [`dataset_convert.ipynb`](dataset_convert.ipynb).

In April 2019 the data source moved to Barcelona's Public Health Agency (ASPB), and the files changed structure. Older and newer files have different columns, station codes and layouts, so each format gets its own steps before the two are combined.

**Current format (April 2019 onwards):** one row per station, pollutant and day, with 24 hourly readings and a validation flag for each.

1. Filter to NO₂ (pollutant code `8`).
2. Discard hourly readings flagged as not validated.
3. Average the remaining hours into a daily value.
4. Build a date from the year, month and day columns.
5. Join station names and coordinates from the stations dataset.

**Older format (June 2018 to March 2019):** snapshot rows with values stored as text, such as `86 µg/m³`.

1. Strip the units and convert the values to numbers.
2. Parse each snapshot's timestamp into a date.
3. Map the old station codes to station numbers, including the one code (`IZ`) shared by Palau Reial and Observatori Fabra.
4. Average each station's readings into a daily value.
5. Rename and reorder the columns to match the current format.

Both sets are then combined, sorted by date and station, and saved as `pollution_dataset.csv`, ready for Tableau.

## Technologies Used

- [Python][5]
  - The language used for the whole pipeline.
- [pandas][6]
  - Loading, cleaning, reshaping and merging the CSV files.
- [Jupyter][7]
  - Notebook for building and running the pipeline step by step.
- [Tableau Public][8]
  - Creating and publishing the dashboard.

The raw CSVs are already included in `datasets/`. New months in the current format can be added to `datasets/air_quality/2020s/` and picked up by running the notebook again.

## Credit and Contact

### Data

Air quality and station data from [Open Data BCN][2], Barcelona City Council, used under [CC BY 4.0][10].

### Contact

Please feel free to contact me on [LinkedIn][11].

[1]: https://public.tableau.com/app/profile/rory.patrick.sheridan/viz/BarcelonaAirQualityImprovement/Dashboard1
[2]: https://opendata-ajuntament.barcelona.cat/
[3]: https://opendata-ajuntament.barcelona.cat/data/en/dataset/qualitat-aire-detall-bcn
[4]: https://opendata-ajuntament.barcelona.cat/data/en/dataset/qualitat-aire-estacions-bcn
[5]: https://www.python.org/
[6]: https://pandas.pydata.org/
[7]: https://jupyter.org/
[8]: https://public.tableau.com/
[10]: https://creativecommons.org/licenses/by/4.0/
[11]: https://www.linkedin.com/in/rp-sheridan/
[12]: https://apps.who.int/iris/bitstream/handle/10665/345329/9789240034228-eng.pdf
[13]: https://environment.ec.europa.eu/topics/air/air-quality/eu-air-quality-standards_en
