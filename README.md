# Arctic Station Power Simulator
A one-second simulator for a station with one generator and one breaker.
<br><br>
This offline Python simulator models a diesel generator (G101), one bus, a circuit breaker (CB101), station load (L101), and the frequency sensor (FS101). It is a training tool for comparing normal breaker cycling with a false under-frequency reading and a generator outage. It does not connect to equipment or configure real protection.

## Getting Started
<br>
### Prerequisites

Before running this project, ensure you have the following installed refer ([requirements.txt](./requirements.txt)):
* **Python:** Version 3.10 or higher ([Download Python](https://python.org))
* **Jupyter:** Jupyter Notebook or JupyterLab (if running `.ipynb` files) and chart dependencies
  
## A. Install and run

1. Extract `arctic-station-simulator.zip` and open a terminal in the extracted `arctic-station-simulator` folder.
2. Open `arctic-station-simulator.ipynb`, select a Python 3 kernel if prompted, and choose **Run → Run All Cells**.  
3. In the notebook's **Run settings** cell, change `DURATION_S` or `SCENARIO` and rerun that cell. See [the user guide](USER_GUIDE.md) for expected timings and graph interpretation.

## B. System Architecture
<img width="3986" height="2076" alt="system-architecture" src="https://github.com/user-attachments/assets/05ccb0a3-bf89-4f80-9806-909b1d203968" />

## C. Model assumptions

| G101 | CB101 | Change per second |
| --- | --- | ---: |
| ON | CLOSED | −0.05 Hz |
| ON | OPEN | +0.12 Hz |
| OFF | CLOSED | −0.15 Hz |
| OFF | OPEN | 0.00 Hz |

- **Limits:** Simulations start at 50.00 Hz. The 49.50–50.50 Hz band is inclusive. Calculations use integer hundredths of a hertz, so reaching 49.50 Hz exactly does not trigger an accidental floating-point comparison error.
- **Timing:** At integer time `t`, the scenario selects G101's state, FS101 takes a sample, and the controller chooses CB101 for `[t, t+1)`. The true frequency changes at the specified constant rate during the interval. The final frequency sample at `t = DURATION_S` is recorded without another control decision.
- **Breaker policy:** Close whenever the current FS101 reading and the predicted reading after one closed second are both in band. With G101 ON this requires at least 49.55 Hz; with G101 OFF it requires at least 49.65 Hz. The controller has no access to true frequency. G101 stays ON except for the scheduled outage; the breaker is the only control action.
- **Fault behavior:** The `sensor_fault` preset injects one low reading at 6 s and can cause an unnecessary one-second opening. The model does not validate arbitrary sensor errors, simulate sensor redundancy, or claim safe operation under faults beyond these presets.
- **Scope:** The exercise allows instantaneous switching and automatic reclosing, with no breaker wear, protection delays, governor dynamics, voltage behavior, or thermal modeling. CB101 CLOSED measures load connection, not proof of usable voltage or sustained power. The OFF/CLOSED rate is modeled stored-energy drawdown only.

Normal frequency cannot remain at 50.00 Hz while the load stays connected: the modeled connected rate is always negative. The resulting short open intervals are an intended consequence of the specified rates, not a claim about actual diesel-generator operation.

## D. Flowchart - Model control and logic.
<img width="3726" height="3457" alt="control-logic-flowchart" src="https://github.com/user-attachments/assets/9a0c67b5-ad48-499d-8b9a-22360b162152" />

## E. Project files

| File | Purpose |
| --- | --- |
| `arctic-station-simulator.ipynb` | Self-contained, runnable notebook with the saved 30-second normal result |
| `requirements.txt` | Notebook and plotting dependencies |
| `system-architecture.png` | Power path, feedback and notebook output |
| `control-logic-flowchart.png` | One-second simulation and protection decision |
| `USER_GUIDE.md` | Notebook interface, exercises and troubleshooting |
| `AI_USAGE.md` | AI tools, prompts, responses and engineering changes |
| `Results/*.png` | Simulation results |

## F. Sample Results
<br>
1. Arctic Station - Normal 
<br><br>
<img width="1820" height="1136" alt="my-drill-normal" src="https://github.com/user-attachments/assets/74479827-bdae-45b1-973b-c9932f93defc" />
<br><br>
2. Arctic Station - Sensor Fault
<br><br>
<img width="1820" height="1136" alt="my-drill-sensor_fault" src="https://github.com/user-attachments/assets/eefebaef-1539-4284-8ef4-54a4e513fc46" />

<br><br>
3. Arctic Station - Generator Outage
<br><br>
<img width="1820" height="1136" alt="my-drill-generator_outage" src="https://github.com/user-attachments/assets/0578198a-2248-4ba7-a470-29dba5aaf787" />

<br><br>
