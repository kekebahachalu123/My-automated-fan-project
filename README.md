# Fan Placement Optimization for a Lecture Hall

## What this project does

A lecture hall is warmer in some places than others. Two fans can help, but only if they are in the right place and pointed the right way. If they are placed badly, they overcool the seats next to them and the hall gets more uneven.

This project treats fan placement as an **optimization problem**. It measures the hall's temperature, builds a model of that temperature, models how a fan cools nearby seats, and then searches for the fan positions and directions that make all seats as close in temperature as possible.

## Main result

The gap is **J = hottest seat minus coolest seat**. A smaller J means a more even hall.

| Scenario | J (°C) | Hottest seat (°C) | Coolest seat (°C) | Change vs. no fans |
|---|---|---|---|---|
| No fans | 2.73 | 22.93 | 20.19 | baseline |
| Unoptimized (corner fans) | 2.94 | 22.88 | 19.93 | 7.6% worse |
| Optimized, West wall, aimed East | 2.01 | 21.83 | 19.82 | 26.5% better |
| Optimized, free position and aim | 1.58 | 21.72 | 20.14 | 42.1% better |

The same two fans can make the hall less even or much more even. The only difference is where they are and where they point.

## Why the hall is uneven

- The West side is warmer than the East side. Temperature drops about 0.095 °C for every meter moving East.
- The ceiling is 8.0 m high at the front and drops to 5.18 m at the rear, so warm air collects over the back rows.

## How it works

1. **Measure.** Ten temperature readings with the fans off, across the 21 m x 32 m hall (`data/baseline_measurements.csv`).
2. **Fit a temperature model.** A least-squares fit gives the temperature at any point:

   `T(x, y) = 19.99 + 2.00 * (1 - x/21) + 25.17 * (1/h(y) - 1/8)`

   Here `x` runs West to East, `y` runs front to rear, and `h(y)` is the ceiling height at `y`. The fit matches the measurements with R² = 0.9997.
3. **Model a fan.** A fan cools a seat by

   `Cmax / (1 + (d / reach)^2) * max(0, cos(angle))^1.5`

   - `d` is the distance from the fan to the seat.
   - `Cmax = 4.0 °C` is the strongest possible cooling.
   - `reach = 3.5 m` controls how quickly cooling fades with distance.
   - The `cos(angle)` part means cooling is strongest straight ahead of the fan and zero behind it.
   - Where both fans reach a seat, the stronger one is used.
4. **Optimize.** SciPy's differential evolution searches for the fan positions and aims that give the smallest J over all seats. The random seed is fixed, so results are the same on every run.

## Best fan placements found

Positions are in meters. The aim angle is measured from East, so 0° points East.

| Scenario | Fan 1 (x, y, aim) | Fan 2 (x, y, aim) |
|---|---|---|
| Unoptimized | (0.0, 0.0, 45°) | (0.0, 21.0, -45°) |
| West wall, aimed East | (0.0, 16.75, 0°) | (0.0, 23.49, 0°) |
| Free position and aim | (0.86, 23.40, -80.1°) | (0.76, 27.04, -26.4°) |

Both optimized layouts put the fans on the warm West side, in the back half of the hall where the ceiling is low and the seats are hottest.

## How to run

```
git clone https://github.com/kekebahachalu123/fan-placement-optimization.git
cd fan-placement-optimization
pip install -r requirements.txt
python run.py
```

The results are saved in the `results/` folder:

- `summary.txt` and `results.json`: the fitted model and the scenario table
- `temperature_maps.png`: temperature maps for three scenarios
- `seat_gap_comparison.png`: bar chart of J for all four scenarios
- `fan_placement_animation.gif`: fans ramping up, unoptimized vs. optimized

## Files

- `model.py`: hall geometry, temperature fit, fan model, seat grid
- `run.py`: fitting, optimization, tables, figures, animation
- `data/baseline_measurements.csv`: the ten fans-off readings
- `requirements.txt`: Python packages needed (numpy, scipy, matplotlib, pillow)

The hall size, seat blocks, `Cmax`, `reach`, and the exponent are constants at the top of `model.py`. Change them to test a different room.

## Limitations

- The temperature model is fitted to only 10 readings with 3 parameters. The high R² shows it matches those readings, not that it is right at every seat.
- `Cmax`, `reach`, and the 1.5 exponent are modeling choices, not measurements of a real fan, and the best placements depend on them.
- The model is a flat 2D snapshot. It ignores air circulation, cooling time, seat height, and people in the room.
- The placements are tested in simulation only, not with real fans in the hall.

## Author

Kekeba Bekele - [github.com/kekebahachalu123](https://github.com/kekebahachalu123)

