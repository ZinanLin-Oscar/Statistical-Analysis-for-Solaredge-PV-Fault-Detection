# Statistical-Analysis-for-Solaredge-PV-Fault-Detection
This Github repository provides Python code for the automated retrieval of Solarledge module-level power generation data, and the subsequent application of statistical analysis to detect faults in PV modules.

# 1. Overview
[Solaredge monitoring platform](https://www.solaredge.com/en/products/software-tools/monitoring-platform) is a web-based platform that allows users to monitor and optimize the performance of their solar PV systems in real-time. [Solaredge Monitoring API](https://developers.solaredge.com/) currently only provides the ability to download Site and Inverter level data. In order to obtain Module level data, one must manually download it from the Solaredge Monitoring Platform. This process is time-consuming and cannot meet the needs of periodic automatic batch downloading and analysis of Module level power generation data.

We have developed a Python code that enables Solaredge users to automatically retrieve module level power generation data for a single PV station or for all PV stations under their account. The prerequisite for this work is:
1) PV station is equipped with module optimizers (which are the smallest measuring units for PV power generation);
2) The measurement data of the module optimizers is publicly available on the Solaredge Monitoring Platform (related to user settings).

# 2. Retrieval of module level data
- **Logic：** Crawling data from "Lay out -- Show playback" section on Solaredge Monitoring Platform. The example illustrated in the image below.
- **Data crawling result：** The power generation data (in unit of W) for each module will be collected at 15-minute intervals and exported as a CSV file. Each column in the CSV file will be labeled with a 9-digit code that corresponds to each module.
- **Scheduled automatic data crawling：**  We used ```schedule.every().day.at("23:50").do(job)``` to automatically execute the code at 11:50 PM every day and save the obtained CSV data in a local folder since the data crawling method cannot obtain historical module level power generation data.
- **Data curation:** By using the above method, the data for string level and site level in the "Lay out -- Show playback" section were mistakenly captured. To remove the data for these two levels, we used the following method: ```if df[col].max() > 800: df = df.drop(col, axis=1)```.
![image](https://github.com/ZinanLin-Oscar/Statistical-Analysis-for-Solaredge-PV-Fault-Detection/assets/113269274/c6492e13-de38-4d58-907d-63e1da9b45f8)

