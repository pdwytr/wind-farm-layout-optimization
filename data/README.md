# Data

`shell_hackathon/` is the dataset supplied by the organisers of the Shell.ai Hackathon for Sustainable and Affordable Energy (2020), hosted on HackerEarth: https://www.hackerearth.com/challenges/competitive/shell-hackathon/

It is small, so it is committed to the repo and no download step is needed.

| Path | Contents |
|---|---|
| `shell_hackathon/power_curve.csv` | Turbine power output by wind speed |
| `shell_hackathon/turbine_loc_test.csv` | Sample turbine layout from the organisers |
| `shell_hackathon/wind_data/wind_data_20XX.csv` | Hourly wind direction and speed for the years 2007, 2008, 2009, 2013, 2014, 2015 and 2017 |

`windfarm/aep_calculator.py` reads the power curve and `wind_data_2007.csv`; it finds this folder relative to its own location, so it works from any working directory.
