# Optimizing a Two-Tier Autonomous Delivery Network

This repository contains a Mixed-Integer Linear Programming (MILP) model designed to optimize an urban delivery network using a combination of carrier vans and autonomous ground rovers.

The core challenge addresses range anxiety in last-mile autonomous delivery. While rovers are efficient for local drop-offs, they have strict battery limits. Vans, which have no range restrictions, act as mobile micro-hubs, transporting packages, carrying multiple rovers, and deploying/picking them at optimal locations. The model uses a time-space network to synchronize these vehicle routes, package handoffs, and battery constraints.

## Spatial and Temporal Architecture
The optimization framework discretizes both geography and time to track the precise state of every asset.
- **Geography:** The service region is partitioned into an interconnected grid of discrete zones (`S0`, `S1`, ...). A central hub (`S0`) serves as the origin and terminus for the fleet.
- **Time:** The operational day is divided into uniform time steps (`t = 0` to `20`). Moving from one zone to an adjacent zone consumes one time period, and the solver ensures active vehicles return to the central hub by the end of the horizon with no remaining cargo.

## Fleet Mechanics and Capacity Constraints
The model controls two distinct vehicle types, each governed by independent operational rules.
- **Vans:** Operate without energy limitations and require one full time step to move between zones or execute loading/unloading operations. Vans possess two independent capacity limits: a maximum package capacity and a maximum rover-carrying capacity.
- **Rovers:** Operate autonomously on the ground but are restricted by a finite battery limit. Operating independently consumes energy for zone transitions and service actions (dropping off or collecting a package), whereas riding aboard a van replenishes battery state for subsequent sorties. Rovers also have a localized throughput limit on service actions per time step.

## Progressive Optimization Phases
The model is formulated in three progressive phases to handle escalating operational complexities.

### Phase 1: Baseline Drop-off Network
Focuses strictly on outbound logistics. All packages originate at the central hub and possess specific deadline constraints. Vans transport rovers into the field, and packages can be handed off from a van to a rover provided they are co-located in the same zone during the exact same time period. Rovers have a strict, non-renewable energy budget for the entire planning horizon and cannot recharge.

### Phase 2: Excursion-Based Recharging
Introduces dynamic energy replenishment. The rigid lifetime battery limit is replaced by an excursion-based constraint. Whenever a depleted rover docks back into a van and rides as cargo, it is considered fully recharged. This allows the solver to formulate multi-sortie routes, where a single rover can be deployed, retrieved, recharged, and redeployed multiple times across different sectors of the city.

### Phase 3: Integrated Collections and Relay Handoffs
Upgrades the system to a full Pickup and Delivery Problem (PDP). It introduces collection tasks bounded by strict release times, requiring rovers or vans to pick up field-generated packages and transport them back to the hub. This phase unlocks maximum operational flexibility by enabling direct rover-to-rover package transfers in the field (in addition to van-to-rover transfers), allowing for advanced, multi-leg relay routing.

## Objective Function
The solver seeks to satisfy all delivery deadlines and collection time windows while minimizing the total transportation cost. The objective function minimizes:
1. A fixed cost for deploying each van from the hub.
2. A variable cost applied to van movement (and conditionally to van idling, depending on the network graph).
3. A variable cost tied directly to the total energy consumed by the rover fleet during ground operations.

## Results & Visualizations - Pick-Up and Delivery with Recharge included

In the sample medium PDP run shown below (**Objective Value: `105.0`**, **Solve Time: `3600.06s`**, 20 time steps, 8 drop-off packages `D:0–D:7`, and 5 collection packages `C:0–C:4`), **Van 1** coordinates with **Rovers 1–4** across sectors `S0–S11`.

### 1. Optimized Fleet Schedules (Gantt Chart)
![Optimized Fleet Schedules Gantt Chart](https://github.com/user-attachments/assets/bf50a92e-f9e3-47bc-89c5-a18f5491a69d)

- **What this chart shows:** Each row tracks one asset (`Van 1` and `Rovers 1–4`) across time steps `0` to `20`, color-coded by activity (`MOVE`, `DRIVE`, `RIDE`, `DROPOFF`, `COLLECTION`, and `WAIT`).
- **Key takeaways:** Rovers frequently piggyback on `Van 1` (`RIDE`, in orange—such as Rovers 3 and 4 riding to `S3` at `t = 1->2`, or Rovers 1, 3, and 4 returning from `S8 -> S2 -> S0` at `t = 16->18`) to conserve battery and reposition across zones, while executing local deliveries (`DROPOFF`) and pickups (`COLLECTION`) independently (`DRIVE`) before returning to the hub (`S0`).

### 2. Package Service Timeline (Drop-Offs & Collections)
![Package Service Timeline](https://github.com/user-attachments/assets/23f07382-28e7-4c4d-bdfb-c71d08a9a510)

- **What this chart shows:** The lower section tracks outbound drop-offs (`Drop 0` through `Drop 7`) from creation at the hub (`t = 0`) to completion (green circle) relative to each package's deadline (red `X`). The upper section tracks inbound collections (`Coll 0` through `Coll 4`) from pickup/release through transit to completion.
- **Key takeaways:** All eight drop-offs complete before their deadlines (for example, `Drop 0` finishes at `t = 7` ahead of its deadline at `t = 8`, `Drop 2` at `t = 4` ahead of `t = 6`, and `Drop 4` right on its deadline at `t = 10`). Collection tasks complete by `t = 18` for `Coll 0, 1, 2, 4` and at `t = 20` for `Coll 3`.

### 3. Unified Time-Space Service Trajectory
![Unified PDP Service Solution](https://github.com/user-attachments/assets/d5d95662-52fa-489f-b7b1-d5167f11bbbf)

- **What this chart shows:** Plots each vehicle's movement across **Sector ID** (vertical axis) over **Time Step** (horizontal axis), annotated with active package payloads (`[D:...]` for drop-offs and `[C:...]` for collections).
- **Key takeaways:** Highlights synchronization points where `Van 1` and the rovers meet or travel together—such as consolidating drop-offs and collections around `S10` and `S11` mid-horizon before converging back toward sector `S0` by the end of the schedule.

## Technologies Used
- **Python** for core logic and data structures.
- **Gurobi** for mixed-integer linear programming and exact optimization.
- **NetworkX** for time-space multigraph generation and spatial routing constraints.
- **Matplotlib** for spatial-temporal visualizations and Gantt chart scheduling.

## How to Run

1. Clone this repository:
   ```bash
   git clone [https://github.com/ivaylo-iliev-31/two-tier-delivery-network.git](https://github.com/yourusername/two-tier-delivery-network.git)
