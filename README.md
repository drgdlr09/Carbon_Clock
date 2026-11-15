# Carbon Clock
**Track:** AI + Climate
**Team:** Dilan Meswani
**Demo Video:** https://youtu.be/kga1mEwQqTU?si=XsbIYaXkTQgH0r4Q

## How to run the notebook
The notebook's results are already saved in `Carbon_Clock.ipynb`. To rerun it yourself:
1. Download the PJM file from the EIA Hourly Electric Grid Monitor: https://www.eia.gov/electricity/gridmonitor/dashboard/electric_overview/US48/US48 --> Download Data  --> Balancing Authority/Regional Files --> PJM.
2. Rename it `PJM.xlsx` and put it in the same folder as the notebook.
3. Run all cells. (Or skip the first data cell: `pjm_carbon.csv`, the 2-column extract it creates, is already included in this repo.)

## The problem
As a new driver who drives an electric vehicle, I wanted to know if when I charge my car changes how much pollution the charge causes. It turns out that the power grid's mix of energy sources changes every hour, so the same charge could be cleaner or dirtier depending on timing. The power I use to charge my car could come from clean sources, like solar, wind, or nuclear power, or dirty sources, like fossil-fuels such as coal and natural gas depending on what time I choose to charge. The big problem is that drivers have no easy way to know when the grid will be cleanest.

## What I built
Carbon Clock forecasts the hourly carbon intensity of the PJM grid (which serves my home state, New Jersey) and recommends when to charge. 
**Inputs:** the time a driver plugs in, how long the car needs to charge, and what time the driver needs the car by (deadline)
**Outputs:** the time recommended by the model that a driver should schedule a charging session for their car to maximize CO2 savings

## What I added
Carbon Clock was my own idea. Beyond forecasting, I tested Carbon Clock as a decision tool by simulating charging on every night of the test period and compared it with charging immediately, a fixed rule that relies on the average best time, and ideal results, and tested how savings change with more flexible deadlines.

## How it works
**Data:** hourly CO2 emissions intensity of consumed electricity for the PJM power grid, from the U.S. Energy Information Administration's Hourly Electric Grid Monitor, July 2018 - November 2026 (~71,000 hours of data). I checked for duplicates (none), implausible values (none), and dropped hours that were missing data (about 3% of hours). The full Excel file from the EIA is 110 MB, so the repo includes the 2-column extract ('pjm_carbon.csv') used for the project.

**Model:** linear regression using hour of day, day of week, weekend, month, and carbon intensity at the same hour yesterday (explained further in results section). Random forest was also tested. I trained on data before August 2026 and tested on August-November 2026.

## Results
**Forecast (on unseen test data):** the persistence baseline ("same as yesterday") scored a mean absolute error of 15.9 g CO2/kWh. My linear regression scored 14.5 (about 9% better), and the random forest scored 15.0.

**Scheduling test:** I plug in at 6:00PM and need 2 hours of charging (~50 miles on a 7.2 kW Level 2 home charger). Each strategy chooses its charging hours using only the information it would really have, but all are graded on the real carbon intensity of the hours they chose.

**Charging decisions (96 nights):**
| Strategy | CO₂ saved vs. charging at 6 PM (done by 7 AM) | (done by 6 PM next day) |
|---|---|---|
| Fixed rule (average pattern) | 2.0% | 2.0% |
| **Carbon Clock forecast** | **2.4%** | **8.7%** |
| Ideal results (best possible) | 3.6% | 10.2% |

- With a full day of flexibility, Carbon Clock captures about 85% of the best possible savings. With an overnight deadline, Carbon Clock is able to capture 65% of all possible savings vs. the fixed rule's 54%.
- For one driver charging 250 times per year, the overnight scenario saves about 29 kg of CO2 per year. For 100 drivers each charging 250 times per year, about 2.9 metric tons of CO2 are saved.
- **Where it fails:** the model leans heavily on yesterday's carbon intensity values, so it reacts to sudden changes a day late. On October 11, its worst day in the test period, it recommended starting at 6:00PM (5,122 g CO2) while the best starting time was 4:00AM (4,446 g CO2).
- **How it compares to the fixed rule (based on average):** With a 1 AM deadline, the fixed rule actually increased emissions by 3.8%, while Carbon Clock never did worse than charging immediately.
- **Takeaways:** on PJM, carbon intensity varies only a few percent across a night, so timing alone only gives modest savings. Flexibility matters far more.

**Limitations:** forecasts only work up to 24 hours ahead; if many drivers charged at the same "cleanest" hour, the extra demand could make the hour dirtier.

## What I'd do next
- Connect Carbon Clock to EV apps and smart plugs so scheduling can be done automatically.
- Use the PJM's day-ahead demand forecast so the model doesn't just repeat yesterday's pattern.

## Sources
- Data: U.S. Energy Information Administration (EIA), Hourly Electric Grid Monitor (PJM file): https://www.eia.gov/electricity/gridmonitor/dashboard/electric_overview/US48/US48
- EV charger power and range: U.S. Department of Transportation (https://www.transportation.gov/rural/ev/toolkit/ev-basics/charging-speeds) and U.S. Environmental Protection Agency (https://www.epa.gov/greenvehicles/plug-electric-vehicle-charging-basics)
- Tutorials I used to learn ML: Kaggle Learn: Intro to Machine Learning (https://www.kaggle.com/learn/intro-to-machine-learning), 
Simple Machine Learning Code Tutorial for Beginners with Sklearn Scikit-Learn by Python Simplified (https://www.youtube.com/watch?v=-IvNzmrcyUM&t=609s), Python Machine Learning Tutorial (Data Science) by Programming with Mosh (https://www.youtube.com/watch?v=7eh4d6sabA0&t=891s), and 
Jupyter Notebook Complete Beginner Guide - From Jupyter to Jupyterlab, Google Colab and Kaggle! by Rob Mulla (http://youtube.com/watch?v=5pf0_bpNbkw&t=588s)
- Libraries: pandas, NumPy, scikit-learn, matplotlib
- AI tools: Claude (Anthropic) for learning concepts, debugging, and suggesting code. All analysis choices were reviewed and understood by me, and I wrote most of the code by hand myself.

**This project is a prototype for educational purposes.**
