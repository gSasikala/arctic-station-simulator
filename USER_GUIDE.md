# Arctic Station Simulator User Guide

Use the notebook to compare a real modeled frequency change with an incorrect FS101 reading. Each run starts at 50.00 Hz and advances simulated time immediately. You do not need to wait 30 real seconds to complete a 30-second run.

## Open the notebook

Follow the installation steps in [README.md](README.md), then open `arctic-station-simulator.ipynb` in JupyterLab. Select the Python 3 kernel that has Matplotlib installed and run all cells once. If you only want to inspect the default plots, they are already saved in the notebook.

## Use the notebook interface

The code cells define the model and plotting functions. The **Run settings** cell is the only one you need to edit during a drill. Change the two values below, then click that cell and press `Shift+Enter`; the summary and three-panel plot are replaced by the new run.

```python
DURATION_S = 30
SCENARIO = "normal"
```

| Setting | What to enter |
| --- | --- |
| `DURATION_S` | A positive whole number of simulated seconds. Use `30` for comparisons, `12` to focus on the first events, or a longer run to observe steady cycling. |
| `SCENARIO` | Exactly one of `"normal"`, `"sensor_fault"`, or `"generator_outage"`. Include quotes. |

Each run is independent: changing settings and rerunning does not carry over the previous frequency or states. After a kernel restart, use **Run → Run All Cells** before editing or rerunning only the settings cell.

## Read the output

- **Summary:** `Load connected` counts the seconds that CB101 was CLOSED and its percentage of the run. It is not a voltage or service-availability measurement. `True frequency` is the physical range calculated by the simulator. The two out-of-band counts separate physical frequency and FS101 samples; both counts include the terminal frequency sample.
- **Top panel, frequency:** The solid teal line joins raw FS101 samples for readability. Dashed lines mark 49.50 and 50.50 Hz; landing exactly on either line is in band. In the fault drill, a dotted line shows the true frequency and an X marks the incorrect reading. In the other two drills the reported and true values coincide, so only FS101 is plotted.
- **Middle panel, G101:** The step is ON or OFF for each one-second interval. OFF in the outage drill is a scheduled scenario event, not a controller command.
- **Bottom panel, CB101:** A step starting at `t` applies until `t+1`. CLOSED connects L101; OPEN disconnects it. Use the same horizontal time scale across all panels to match a sample with the state it produced. There is no extra interval at the right edge.

## Run the drills

| Drill, all at 30 s | True range | True samples out of band | FS101 samples out of band | Load connected |
| --- | --- | ---: | ---: | ---: |
| `normal` | 49.50–50.00 Hz | 0 | 0 | 24 s (80.0%) |
| `sensor_fault` | 49.50–50.00 Hz | 0 | 1 | 24 s (80.0%) |
| `generator_outage` | 49.50–50.00 Hz | 0 | 0 | 20 s (66.7%) |

### Normal operation

Set `SCENARIO = "normal"`. At 10 s the bus reaches 49.50 Hz. The next connected second would reach 49.45 Hz, so CB101 opens over `[10, 11)`. Frequency rises to 49.62 Hz, allowing CB101 to close at 11 s. The later sawtooth is caused by further switching near the lower limit.

### False under-frequency indication

Set `SCENARIO = "sensor_fault"` and run for at least 7 s. At 6 s, FS101 reports 49.10 Hz even though the true bus is at 49.70 Hz. The controller sees only FS101 and opens CB101 over `[6, 7)`. At 7 s, FS101 again matches true frequency (49.82 Hz), and the breaker recloses if the run includes that interval, which requires at least 8 s. Compare the bottom plots around 6 s: the total connected time is the same in both 30-second runs, so that total alone does not reveal the false trip.

### Generator outage and restart

Set `SCENARIO = "generator_outage"`. G101 is OFF over `[5, 11)`. CB101 remains CLOSED during `[5, 6)` because the modeled stored energy allows frequency to go from 49.75 to 49.60 Hz without crossing 49.50 Hz. It opens at 6 s, and frequency stays at 49.60 Hz until the generator returns. At 11 s the generator returns and the breaker closes. Use at least 12 s to include the first restarted interval.

## Troubleshoot and save

- **`NameError`:** Run all cells in order to load the simulator and plotting functions before the settings cell.
- **`ModuleNotFoundError: matplotlib`:** Install the requirements in the same Python environment used by the notebook kernel. Relaunch JupyterLab from that environment if needed.
- **`ValueError`:** Check the exact scenario spelling and use a whole number greater than zero for duration.
- **No event visible:** Use 30 s or the minimum duration specified for the drill. If the plot is too dense in a long run, shorten it to examine individual transitions.
- **Save a plot:** After a run, add a new code cell with `fig.savefig("my-drill.png", dpi=180, bbox_inches="tight")`. Run it, then retrieve the PNG from JupyterLab's file browser.

Use these drills to discuss why CB101 opened, whether the bus really left the band, and what evidence would distinguish a bad measurement from a real fault. Do not use this training model to choose real protection settings, restart procedures, or equipment operating limits.
