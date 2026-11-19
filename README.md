# Wind-Farm Layout Optimisation with a Genetic Algorithm (Shell.ai Hackathon 2020)

A genetic algorithm that places 50 wind turbines on a 4 km x 4 km offshore farm to maximise Annual Energy Production (AEP) under wake losses. Built for the Shell.ai Hackathon for Sustainable and Affordable Energy. Finished 32nd, in the top 10% of non-zero-scoring teams.

## The problem

Place 50 turbines (100 m rotor diameter, 100 m hub height) on a square 4 km x 4 km farm so that yearly energy output is as high as possible.

A turbine slows the wind behind it (its wake), so a turbine standing in another's wake produces less. Layouts are scored with the Jensen (PARK) wake model, using a turbine power curve and a year of hourly wind direction and speed data. A poor layout can lose 5-15% of AEP to wake effects.

A layout is valid only if:

- Perimeter: every turbine is inside the farm and at least 50 m from the boundary.
- Proximity: every pair of turbines is at least 400 m apart (4 rotor diameters).

The search space is high-dimensional (100 coordinates), multimodal and discontinuous, which is why a population-based search suits it. Official statement: [problem-statement.pdf](docs/problem-statement.pdf). Evaluator background: [aep-algorithm.pdf](docs/aep-algorithm.pdf), [wake-model.pdf](docs/wake-model.pdf), [data-interpretation.pdf](docs/data-interpretation.pdf).

## Approach

The notebook [notebooks/01_genetic_algorithm.ipynb](notebooks/01_genetic_algorithm.ipynb) walks through the method; [windfarm/genetic_algorithm.py](windfarm/genetic_algorithm.py) is the same code as a script. What the code does:

- Gene / chromosome: one gene is an (x, y) turbine position; a chromosome is a 50-row table of positions (the whole layout).
- Initial population ([windfarm/chromosome.py](windfarm/chromosome.py)): each layout starts with 20 points sampled from a fixed ring of 36 boundary positions (8 per side, spaced 433 m, plus the 4 corners, all 50 m from the edge). The other 30 are random interior points, each accepted only if it is more than 400 m from every point already placed.
- Fitness: AEP in GWh from `get_AEP` in [windfarm/aep_calculator.py](windfarm/aep_calculator.py) (the organisers' vectorised evaluator: 36 wind directions x 15 speed bins, Jensen wake model, root-sum-square combination of wakes). It reads the 2007 wind data and the power curve.
- Selection (elitism): each generation the 4 layouts with the highest AEP survive.
- Crossover: pool all points from the 4 elite layouts, shuffle, then greedily keep a point if it is at least 400 m from every point already kept, until 50 are collected.
- Mutation: with probability `mutation_probability` (0.8 initially, divided by the child counter after each mutation, so it drops quickly) the child is a fresh random layout from `fn_chromosome`. Otherwise the child is a crossover.
- Loop: each generation adds 15 children to the 4 elites, scores everything and keeps the top 4. The best layout is returned by `fn_main` and written to `layouts/submissions/submission.csv`.

The seed layouts in [layouts/seed/](layouts/seed) and the `submission*.csv` files in [layouts/submissions/](layouts/submissions) are layouts produced during the competition.

## Results

- Leaderboard: 32nd place, top 10% of non-zero-scoring competitors ([HackerEarth leaderboard](https://www.hackerearth.com/challenges/competitive/shell-hackathon/leaderboard/)).
- Re-scored with the included evaluator (numpy 2.4 / pandas 3.1, 2007 wind data): the seed layouts `layouts/seed/seed_1.csv` to `seed_4.csv` give 536.37, 535.41, 536.37 and 536.94 GWh.
- No result plots are included in the repo. `fn_plot` in `windfarm/chromosome.py` draws a layout as a scatter plot.

## Project structure

```
.
├── windfarm/                  # Python package
│   ├── aep_calculator.py      # organisers' AEP evaluator (Jensen wake model)
│   ├── chromosome.py          # layout generation and plotting
│   └── genetic_algorithm.py   # selection, crossover, mutation, main loop
├── notebooks/
│   └── 01_genetic_algorithm.ipynb
├── data/
│   ├── README.md              # what the data is and where it came from
│   └── shell_hackathon/       # power curve, wind data, sample turbine layout
├── layouts/
│   ├── seed/                  # seed layouts seed_1.csv .. seed_4.csv
│   └── submissions/           # layouts produced during the competition
├── docs/                      # problem statement, evaluator notes, solution summary
├── requirements.txt
└── README.md
```

## Tech stack

Python 3.11, NumPy, pandas, Shapely (constraint checks), Matplotlib (plots); developed in Jupyter / Google Colab.

## Quickstart

The data needed to evaluate layouts is already in the repo: `data/shell_hackathon/` holds `power_curve.csv`, `turbine_loc_test.csv` and `wind_data/wind_data_20XX.csv` (see [data/README.md](data/README.md)). Run the commands below from the repo root.

```
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# score a layout
python -c "import pandas as pd; from windfarm.aep_calculator import get_AEP; get_AEP(pd.read_csv('layouts/seed/seed_4.csv'))"

# run the genetic algorithm (writes layouts/submissions/submission.csv, overwriting the committed one)
python -m windfarm.genetic_algorithm
```

Or open `notebooks/01_genetic_algorithm.ipynb`. Tune `mutation_probability`, `number_of_chromos_in_population` and `number_of_iterations` near the top of the script/notebook. A full run is slow because every layout is scored with the full wake model.

The original data comes from the challenge page: https://www.hackerearth.com/challenges/competitive/shell-hackathon/. Layouts are CSVs with columns `x,y` (50 rows).

## Competition context

Shell.ai Hackathon for Sustainable and Affordable Energy (2020), challenge "Windfarm Layout Optimisation", hosted on HackerEarth: https://www.hackerearth.com/challenges/competitive/shell-hackathon/

## License

No license file is currently included. `windfarm/aep_calculator.py` is the organisers' evaluator.
