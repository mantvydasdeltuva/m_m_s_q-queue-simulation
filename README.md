<div align="center">
  <img src="assets/banner-dark.png#gh-dark-mode-only" alt="Banner" style="width: 600px; height: auto;">
  <img src="assets/banner-light.png#gh-light-mode-only" alt="Banner" style="width: 600px; height: auto;">
</div>

---

### Overview

This repository implements a simulation of an  **M/M/S/Q queueing model**, a classic queueing system used to study scenarios with a finite number of servers and a finite queue capacity. Calls that cannot be immediately served join the queue if there is space, otherwise, they are dropped.

The M/M/S/Q model is defined by:
- **M** (Markovian Inter-Arrival Times): The time between successive arrivals follows an exponential distribution.
- **M** (Markovian Service Times): The service times are exponentially distributed.
- **S** (Finite Servers): A fixed number of servers are available to serve arriving calls.
- **Q** (Finite Queue Capacity): The queue has a finite capacity; calls that exceed this capacity are dropped.

This model is widely applicable in areas such as telecommunications, server networks, and customer service systems where queuing space is limited.

---

### Events in the Simulation

The simulation revolves around two types of **events**:
1. **Arrival Event:** A new call enters the system. If all servers are busy but the queue has capacity, the call joins the queue. Otherwise, the call is dropped. Each arrival event generates the next arrival time (`next_arrival_time`) by sampling from the exponential inter-arrival time distribution.
2. **Departure Event:** A call completes its service and leaves the system. If there are calls in the queue, the next one starts service.

Each event updates the system state, including the number of active calls (`num_calls`), number of calls in queue (`num_queue_calls`), number of delayed calls (`num_delayed_calls`), number of dropped calls (`num_dropped_calls`) and the time (`current_time`).

---

### Event Determination Process

At any point, the simulation determines which event (arrival or departure) occurs next:
  - **Arrival Time:** Calculated as the current time plus a random inter-arrival time (`exprnd(1 / lambda)`). This is updated every time an arrival event occurs.
  - **Departure Time:** The earliest scheduled departure time from the scheduled departures (`scheduled_departure_times`) list.

The simulation compares the next arrival time (`next_arrival_time`) and the earliest departure time (`min(scheduled_departure_times)`) to decide:
- If the arrival time is sooner, an **arrival event** occurs:
  - If servers are available (`num_calls < S`), the arriving call is served, the number of active calls (`num_calls`) increases, and a departure is scheduled.
  - If servers are full (`num_calls >= S`) and the queue has space (`num_queue_calls < Q`), the call joins the queue (`num_queue_calls`), and the delayed calls counter (`num_delayed_calls`) is incremented.
  - If the queue is also full (`num_queue_calls >= Q`), the call is dropped and dropped calls counter (`num_dropped_calls`) is incremented.
  - Determines the next arrival time by sampling a new inter-arrival time.
- Otherwise, a **departure event** occurs:
  - If there are calls in the queue (`num_queue_calls > 0`), one is moved from the queue to service, and a new departure is scheduled.
  - If no one is in the queue, the number of active calls (`num_calls`) decreases.
  - Removes the completed departure from the schedule.

---

### Simulation Parameters

- **Arrival Rate (λ):** The average number of arrivals per unit time.
- **Service Rate (μ):** The average number of services completed per unit time.
- **Number of Servers (S):** The total number of servers. This is an array for server utilization and delay probability statistics.
- **Queue Capacity (Q):** The maximum number of calls that can wait in the queue.
- **Total Time:** The duration of the simulation.

```matlab
lambda = 30      % arrivals / unit time
mu = 2           % services / unit time
S_range = 1:1:20 % number of servers
Q = 3            % maximum number of calls in queue
total_time = 100 % units
```

---

### Simulation Flow

1. Start the simulation clock at zero (`current_time = 0`).
2. Generate the first arrival event and initialize the system state:
   - Sample the time until the first arrival from the exponential distribution.
   - Log the initial system state (time and number of calls).
3. Iteratively process events by determining whether the next event is an **arrival** or **departure**:
   - Compare the time of the next arrival with the earliest departure.
   - Update the system state based on the type of event:
     - **Arrival Event:**
       - If space is available, serve the call, increase the number of active calls, schedule a departure and generate the next arrival time.
       - If servers are full but queue space is available, add a call to the queue, increment the delayed calls counter (`num_delayed_calls`), and generate the next arrival time.
       - If both servers and the queue are full, increment the dropped calls counter (`num_dropped_calls`) and generate the next arrival time.
     - **Departure Event:**
       - If there are calls in the queue (`num_queue_calls > 0`), one is moved from the queue to service, and a new departure is scheduled.
       - If no one is in the queue, the number of active calls (`num_calls`) decreases.
       - Remove the completed departure from the schedule.
   - Log the system state after each event (time and number of calls).
4. Continue until simulation time is exceeded (`current_time >= total_time`).

This process iterates until simulations are computed with all server capacities.

---

### Visualizations

#### Number of Calls in the System Over Time and Probability Distribution of Calls in the System

This visualization consists of two graphs:

1. **Number of Calls in the System Over Time:** This plot tracks the evolution of the number of active calls during the simulation. The orange line represents the number of active calls (being served and delayed in queue) at each logged event. If the orange graph is above the green dashed line that indicates that the calls over it are in system queue. The cyan dashed line indicates the overall maximum system capacity (servers + queue). The red dashed line shows the time-weighted average number of calls, computed across the simulation duration.
2. **Probability Distribution of Calls in the System:** This histogram displays the probability distribution of the number of active calls (being served and delayed in queue) during the simulation. The distribution reflects the system's overall load.

_This visualization is only created for the last server capacity (S) value. In this case for `S = 20`._

<div align="center">
  <img src="assets/graph1-dark.png#gh-dark-mode-only" alt="Number of Calls in the System Over Time and Probability Distribution of Calls in the System" style="width: 500px; height: auto; border-radius: 4px;">
  <img src="assets/graph1-light.png#gh-light-mode-only" alt="Number of Calls in the System Over Time and Probability Distribution of Calls in the System" style="width: 500px; height: auto; border-radius: 4px;">
</div>

#### Probability of Delay Across Server Capacity and Probability of Loss Across Server Capacity

This plot provides two separate graphs:

1. **Probability of Delay Across Server Capacities:** This plot shows the probability of call delay for different server capacities (S), based on fixed arrival rate (λ) and service rate (μ). The red dashed line represent the trend of simulated delay probabilities across server capacities. The green dashed line indicates simulated system overall quality of service across server capacities. The cyan dashed line represents theoretical delay probability across server capacities. Orange dots are simulated delay probabilities at each server capacity.
2. **Probability of Loss Across Server Capacities:** This plot shows the probability of call loss for different server capacities (S), based on fixed arrival rate (λ), service rate (μ) and maximum queue capacity (Q). The red dashed line represent the trend of simulated loss probabilities across server capacities. The green dashed line indicates simulated system overall quality of service across server capacities. The cyan dashed line represents theoretical loss probability across server capacities. Orange dots are simulated loss probabilities at each server capacity.

<div align="center">
  <img src="assets/graph2-dark.png#gh-dark-mode-only" alt="Probability of Delay Across Server Capacity and Probability of Loss Across Server Capacity" style="width: 500px; height: auto; border-radius: 4px;">
  <img src="assets/graph2-light.png#gh-light-mode-only" alt="Probability of Delay Across Server Capacity and Probability of Loss Across Server Capacity" style="width: 500px; height: auto; border-radius: 4px;">
</div>

#### Server Utilization Across Server Capacity

This plot represents the server utilization for different server capacities (S), based on fixed arrival rate (λ) and service rate (μ). The red dashed line shows the trend of simulated utilization across server capacities. The cyan dashed line represents theoretical utilization across server capacities. Orange dots are simulated utilizations at each server capacity.

<div align="center">
  <img src="assets/graph3-dark.png#gh-dark-mode-only" alt="Server Utilization Across Server Capacity" style="width: 500px; height: auto; border-radius: 4px;">
  <img src="assets/graph3-light.png#gh-light-mode-only" alt="Server Utilization Across Server Capacity" style="width: 500px; height: auto; border-radius: 4px;">
</div>

---

### Running the Simulation

1. Clone the repository:
   ```bash
   git clone https://github.com/mantvydasdeltuva/m_m_s_q-queue-simulation.git
   cd m_m_s_q-queue-simulation
2. Open the MATLAB script in the MATLAB editor.
3. Modify the parameters (`lambda`, `mu`, `S_range`, `Q`, `total_time`) if needed.
4. Run the script to simulate and generate the visualizations.

---

### Dependencies

- MATLAB (Recommended version: R2023a or later).
- Statistics and Machine Learning Toolbox
- Curve Fitting Toolbox

---

### License

This project is licensed under the MIT License. See the LICENSE file for more details.

---

### Contributions

Contributions and suggestions are welcome. Please feel free to submit an issue or a pull request for improvements.