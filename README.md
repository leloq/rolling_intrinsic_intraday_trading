
# ⚡ Rolling Intrinsic Intraday Energy Trading Optimization

## Project Overview
The Rolling Intrinsic Intraday Energy Trading Optimization repository models continous intraday trading with the means of discretization and linear optimization. While the results of the underlying paper are based on actual EPEX Spot data (which requires paid access), the open-source repository at hand provides randomly generated data. 

## Project Structure

The notebook 'Create Randomized Intraday Transaction Data' constitutes the first step: random transaction data (continous buy/sell orders) are created. Then, in the folder 'Code/Rolling Intrinsic' the Python scripts 'Rolling Intrinsic H.py' and 'Rolling Intrinsic QH.py' can be executed to model rolling intrinsic trading, respectively on the market for hourly or quarter hourly products. The Python script executes the function simulate_period() with the parameters described below. Finally, the results are then saved in the 'results' folder.

## Parameter Descriptions

1. **threshold**: 
   - **Description**: This is the relative price change threshold as a percentage. It represents the minimum price change required to trigger an action (buy/sell).
   - **Example**: A threshold of 0 means that even the smallest price fluctuation can trigger trading decisions.

2. **threshold_abs_min**: 
   - **Description**: This is the absolute minimum price change threshold. It ensures that there’s a fixed minimum price movement (in absolute terms) that must occur before any trading action is triggered, regardless of percentage changes.
   - **Example**: A value of 0 means no minimum price movement is required.

3. **discount_rate**: 
   - **Description**: This is used for time-based price discounting. It represents the percentage rate at which the future value of cash flows is discounted.
   - **Example**: A discount rate of 0 means no time discounting is applied to prices, treating future prices as if they were happening now.

4. **bucket_size**: 
   - **Description**: This refers to the time interval (in minutes) for grouping or aggregating trades.
   - **Example**: For example, with a bucket size of 15, trades are grouped into 15-minute intervals, helping to control the granularity of the optimization process.

5. **c_rate**: 
   - **Description**: This is the charge rate for the battery storage system. It defines how fast the system can charge or discharge as a fraction of the total storage capacity.
   - **Example**: A value of 0.5 means the battery can charge or discharge up to 50% of its capacity in one unit of time.

6. **roundtrip_eff**: 
   - **Description**: This refers to the roundtrip efficiency of the battery storage system. It represents the overall efficiency of the battery when charging and discharging.
   - **Example**: A roundtrip efficiency of 0.86 means that for every 100 units of energy stored, 86 units are available for use after charging and discharging.

7. **max_cycles**: 
   - **Description**: This defines the maximum number of charge-discharge cycles the battery can perform in a given period (e.g., a year).
   - **Example**: A value of 365 means the battery can undergo one full cycle (charge + discharge) per day on average, limiting the total wear on the battery system.


## 🚀 Getting Started

### Step 1: PostgreSQL Database Setup
First, you need to set up the PostgreSQL database. You can follow these steps:

Open a terminal and run the PostgreSQL command-line interface:

```bash
psql -d postgres
```

#### Create the `intradaydb` Database
Create the database for your project:

```sql
CREATE DATABASE intradaydb;
```

#### Create the `leloq` User
Create a user named `leloq` with the password `123`:

```sql
CREATE USER leloq WITH PASSWORD '123';
```

#### Change ownership to `leloq`
 Change the ownership of `intradaydb` database to the `leloq` user:

```sql
ALTER DATABASE intradaydb OWNER TO leloq;
```

#### Exit the PostgreSQL Shell
Type `\q` to exit the PostgreSQL shell.

```bash
\q
```

### Step 2: Python Environment Setup
Next, set up the Python environment. You'll need Python 3.8 or higher. You can set up a virtual environment and install the required dependencies as follows:

#### Create and activate a virtual environment:

```bash
python3 -m venv venv
source venv/bin/activate
```

#### Install the required dependencies:
Install all required libraries listed in the `requirements.txt` file:

```bash
pip install -r requirements.txt
```

#### Optional -> Install Gurobi:
Gurobi can be used as a solver in this project. Make sure you have a valid Gurobi license. If you haven't installed it yet, you can follow these steps:

Install Gurobi Python bindings:

```bash
pip install gurobipy
```

Set up your Gurobi license (follow the instructions on the Gurobi website).

In the python files `Code/Rolling Intrinsic/Rolling Intrinsic H.py` & `Code/Rolling Intrinsic/Rolling Intrinsic QH.py`
change to the GUROBI solver pulp provides and disable the default solver from pulp:
```python
from pulp import (
    LpProblem,
    LpVariable,
    lpSum,
    LpMaximize,
    GUROBI,
    # PULP_CBC_CMD,
)
...
# m_battery.solve(PULP_CBC_CMD(msg=0))
m_battery.solve(GUROBI(msg=0))
```

### Step 3: Create randomized transaction data (or use actual EPEX Spot data)

Now, you can run the code from the notebook 'Create Randomized Intraday Transaction Data'. Alternatively, you can fill the intradaydb table with actual data from any continous intraday market you want to cover.

### Step 4: Running the Code
Now you're ready to run the optimization code in the "Code/Rolling Intrinsic" folder.
Make sure you execute the file with current working directory being `rolling_intrinsic_intraday_trading/`.
This is necessary for the output landing in the `output/` folder.

### Step 5: Visualize the Results
#### Rolling Intrinsic Behaviour
After having run the optimization code, you can visualize the behaviour of the Algorithm by running the jupyter notebooks
`plotting_trading_behaviour_RIB_*.ipynb`.
The only thing you have to do is to enter the date you want to visualize in the 2nd cell as constants.

Imagine you want to plot December 24th 2022. In that case, you have to change the string in `START_OF_DAY`
to be ```2022-12-24 00:00:00``` and `END_OF_DAY` to ```2022-12-25 00:00:00```
```python
START_OF_DAY = pd.Timestamp('2022-12-24 00:00:00', tz='Europe/Berlin')
END_OF_DAY = pd.Timestamp('2022-12-25 00:00:00', tz='Europe/Berlin')
EXECUTION_TIME_START = (START_OF_DAY - timedelta(days=1)).replace(hour=16, minute=0)
EXECUTION_TIME_END = START_OF_DAY.replace(hour=22, minute=54, second=59)
```
After doing that, just run all cells.
The plots are stored in the path `output/plots/rolling_intrinsic_*.png`

#### Results Plot (Fig. 5) and Table (Tab. 2) in the Paper
Just run the jupyter notebook `revenue_plots_and_results_table.ipynb`.
In case you want to change the timeframe of the visualisation also adapt the two constants in cell 2.

```python
START_DATE = pd.Timestamp('2022-01-01 00:00:00', tz='Europe/Berlin')
END_DATE = pd.Timestamp('2023-01-01 00:00:00', tz='Europe/Berlin')
```


## ⚙️ Dependencies
- **Python**: 3.8+
- **PostgreSQL**: 13+
- **Gurobi**: Optimization solver
- **Libraries**:
  - `psycopg2`
  - `pandas`
  - `numpy`
  - `gurobipy`
  - `sqlalchemy`
  - `pulp`
  - `jupyter`
  - `matplotlib`
  - `loguru`

## 📄 License
This project is licensed under the MIT License - see the `LICENSE` file for details.

## 🛠️ Contributing
Contributions are welcome! Please open an issue or submit a pull request.

## 👤 Contributors
Jannik Dresselhaus, Jan Ludwig, Leo Semmelmann, Kim Miskiw


