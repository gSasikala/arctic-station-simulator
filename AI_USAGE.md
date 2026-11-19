# AI Usage

## Perplexity Computer: primary implementation

**Tool and access.** GPT-6.2 Sol was used through the hosted Perplexity Computer task interface. It authored the Python implementation, tests, notebook, diagrams, code PDF, user documentation, and presentation.

**Prompt.** The user provided this request and the attached brief reproduced at the end of this file.

```text
Build a simple and efficient code to solve the problem as in attached file. Provide the deliverables in downloadable file format
```

**Response.** The complete response is the delivered package, not just the final chat summary. `simulator.py` and `arctic-station-simulator.ipynb` contain the generated implementation; the notebook includes the executed normal result. This is the breaker decision function from the generated code:

```python
def should_close(generator_on: bool, reported: int) -> bool:
    """Use only FS101 and the known generator state to decide for this second."""
    next_if_closed = reported + RATE[generator_on, True]
    return _in_band(reported) and _in_band(next_if_closed)
```

## Local model reviews

Both GGUF models were downloaded and run on the task sandbox CPU using `node-llama-cpp` 3.22.3. The reviews used a fresh chat context, two CPU threads, a 1,024-token context, a 170-token response limit, temperature 0, and seed 42. No hosted inference API was used for these reviews, and neither model is required to run the simulator.

### SmolLM2-135M-Instruct

Model file: [SmolLM2-135M-Instruct-Q4_0.gguf](https://huggingface.co/bartowski/SmolLM2-135M-Instruct-GGUF/resolve/main/SmolLM2-135M-Instruct-Q4_0.gguf).

**Prompt sent**

```text
Review this offline power simulator rule with exact numbers.
Safe band [49.50, 50.50] Hz. G101 is ON. CLOSED changes true frequency by -0.05 Hz per second; OPEN changes it by +0.12 Hz per second. The breaker closes only if the reported sensor frequency and reported frequency minus 0.05 are both in band. At t=6 the true frequency is 49.70 Hz, but the sensor falsely reports 49.10 Hz. The sensor is correct again at t=7.
Give the breaker state at t=6, the true frequency at t=7, and the breaker state at t=7. Explain why a test must compare the true and reported frequencies. Keep the answer under 90 words.
```

**Raw response returned**

```text
The breaker state at t=6 is 49.70 Hz. The breaker state at t=7 is 49.10 Hz. The reason for the difference is that the sensor is reporting the true frequency at t=7, but the breaker is reporting the false frequency at t=6. The difference is 0.05 Hz. The breaker state at t=7 is 49.10 Hz. The reason for the difference is that the sensor is reporting the true frequency at t=7, but the breaker is reporting the false frequency at t=6. The difference is 0.05 Hz. The breaker state at t=7 is 49.10 Hz. The reason for the difference is that the sensor is reporting the true frequency at t=7, but the breaker
```

**Disposition.** Rejected. It confused breaker states with frequency values and repeated itself. In the verified trace, CB101 is OPEN at 6 s, the true bus reaches 49.82 Hz at 7 s, and CB101 is CLOSED at 7 s. The regression test explicitly checks separate true and reported samples.

### Qwen2.5-0.5B-Instruct

Model file: [qwen2.5-0.5b-instruct-q4_k_m.gguf](https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct-GGUF/resolve/main/qwen2.5-0.5b-instruct-q4_k_m.gguf).

**Prompt sent**

```text
Review this Python protection rule for an offline simulator. The values are integer hundredths of a hertz.

def should_close(on, reported):
    next_if_closed = reported - (5 if on else 15)
    return 4950 <= reported <= 5050 and 4950 <= next_if_closed <= 5050

What return values should tests expect for these four calls in order: (True, 4954), (True, 4955), (False, 4964), (False, 4965)? Explain why one value on each side of each threshold is important. Use no more than 80 words.
```

**Raw response returned**

```text
The return values should expect (True, 4954), (True, 4955), (False, 4964), (False, 4965). The first value on each side of the threshold is important because it ensures the simulation is closed, while the second value on each side ensures the simulation is not closed.
```

**Disposition.** Rejected as numeric validation. The response repeated the input pairs instead of giving the four return values and misstated which side closes. The correct results are `False, True, False, True`; they are covered by the threshold regression test.

## Changes and verification

- **Timing:** The notebook and code distinguish the (n+1) frequency samples from the (n) state intervals. The test at exactly 6 s confirms that a fault in the terminal sample is counted but does not create an extra breaker action.
- **Plot annotation:** The reported glitch is labeled directly, with right-edge-aware placement so the numeric label stays readable even if the simulation ends at the faulty sample. The true trace remains separate from the FS101 input.
- **Code PDF:** Long summary print statements were split across lines to avoid truncation in the PDF. The notebook and PDF were regenerated from the same module and settings code.
- **Authority:** The local outputs above were kept for auditability, not used as ground truth. Direct arithmetic, eight passing tests, and execution of all three notebook presets were used to validate the delivered behavior.

## Attached implementation prompt

The text below reproduces the user-supplied `Prompt.txt`.

```text
Background
You're the systems engineer at a remote Arctic research station, 800km from the nearest town. The station runs on a single generator powering life support, heating, satellite communications, and lab equipment for 40 researchers. Outside, it's minus 35 degrees. If the generator overloads and the breaker trips, heating shuts down. At these temperatures, you have roughly 90 minutes before indoor pipes freeze and burst. Last winter, a faulty frequency reading caused an unnecessary breaker trip at 2am. By the time the on-call engineer woke up and reset the system, three labs had lost samples worth two years of data. Your job is to build a simulator that models this power system so the team can train, test, and rehearse exactly these scenarios before the next winter season.

Objective
Build a simulator for the station's single-bus power distribution system that models the physical process, implements the control logic, and displays the system state graphically.

System Components
•	Generator G101: the station's sole diesel generator
•	Circuit breaker CB101: connects or disconnects the station load from the bus
•	Station load L101: everything the station needs to stay alive (heating, life support, labs, comms)
•	Frequency sensor FS101: measures the bus frequency

Design Assumptions
1.	The system operates at a nominal frequency of 50.00 Hz. Frequency above 50.50 Hz is considered an over-frequency condition and below 49.50 Hz is considered an under-frequency condition.
2.	Under normal operation, the system should never enter an over-frequency or under-frequency condition.
3.	CB101 should never be CLOSED (load connected) when the frequency is below 49.50 Hz.
4.	The station always needs power. Therefore, CB101 should be CLOSED whenever feasible.
5.	Frequency variation rates are as follows:
a. G101 = ON, CB101 = CLOSED: frequency drops at 0.05 Hz/second (station load drawing power)
b. G101 = ON, CB101 = OPEN: frequency rises at 0.12 Hz/second (no load, generator spinning freely)
c. G101 = OFF, CB101 = CLOSED: frequency drops at 0.15 Hz/second (load draining stored energy)
d. G101 = OFF, CB101 = OPEN: no change in frequency

Functional Requirements
1.	Implement the simulator with control logic that maintains the bus frequency within safe operating bounds (between 49.50 Hz and 50.50 Hz). The simulator should run in discrete time steps (per second) over a configurable duration.
2.	Display the following as graphs over the simulation duration:
a. Bus frequency as reported by FS101, with the upper and lower limits clearly indicated
b. State of generator G101 (ON/OFF) over time
c. State of circuit breaker CB101 (OPEN/CLOSED) over time


Technical Requirements
•	You may use Python programming language.
•	Organize your project in a clear and maintainable manner.
•	Document any assumptions made during development

AI-Assisted Development
use of open-source models running locally of your system (such as Qwen, GLM, DeepSeek) would be viewed favorably.
Please include an AI_USAGE.md file containing:
1.	The model or tool used and how it was accessed
2.	The prompts provided to the model
3.	The corresponding responses generated by the model
4.	Any modifications you made to the AI-generated output and why


Deliverables 
1. system architecture as a png file
2. Generate simple,efficient and easy to understand python code in downloadable jupyter notebook and pdf files.
3. README.md with project overview, installation instructions, and steps to run the simulator
4. AI_USAGE.md Document your use of AI models for code assistance.
5. Create a user guide that provides detailed instructions on using the simulator, including a description of the UI and how to interpret the results.
6. Create a flowchart illustrating the control logic and processes involved in the simulator. Use a flowchart tool and save it as an image file.
7. Develop a presentation outlining the project, including its motivation, architecture, functionality, and results. Use speaker notes to include narration that explains each slide.
```
