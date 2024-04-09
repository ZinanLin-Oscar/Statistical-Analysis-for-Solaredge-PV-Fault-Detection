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
- **Data curation:** By using the above method, data from String and Site level in the "Lay out -- Show playback" section were mistakenly captured. To remove the data from these two levels, we used the following method: ```if df[col].max() > 900: df = df.drop(col, axis=1)```. Threshold 900 is defined as the rated power of each PV module falls between 370-415W. Two PV modules are connected to one optimizer at our sites, therefore, it is believed that the power generation of module-level data will not exceed 900 W at peak times. This parameter can be modified based on actual conditions.

![image](https://github.com/ZinanLin-Oscar/Statistical-Analysis-for-Solaredge-PV-Fault-Detection/assets/113269274/c6492e13-de38-4d58-907d-63e1da9b45f8)

| Parameter | Description | 
| ------- | ------- | 
| login_url | Solaredge monitoring platform entrance | 
| panels_url | Solaredge monitoring platform "Lay out -- Show playback" section | 
| SOLAREDGE_USER | Your Solaredge monitoring platform account | 
| SOLAREDGE_PASS | Your Solaredge monitoring platform account password | 
| SOLAREDGE_SITE_ID | Site 7-digit ID number | 

| Code | Description | 
| ------- | ------- | 
| ```Daily crawling for one PV station.ipynb``` | 1. Crawl all the module-level data for a single power station in a single day and generate a dataframe | 
| ```Automated daily crawling for all PV stations.ipynb``` | 1. Crawl all the module-level data for all power stations in a single day<br>2. Save all the data for each power station in a single day to a CSV file<br>3. Save all the CSV files in a single day to a local folder<br> 4. Automatically schedule code execution at 11:50 pm everyday | 


# 3. 3σ-rule based PV panel fault detection (under same operation period)
- **Logic：** The 3σ rule, also known as the three-sigma rule, is a statistical rule that states that nearly all values (about 99.7%) lie within three standard deviations of the mean in a normal distribution. It is commonly used to identify outliers or unusual values in a dataset. We first integrate the generated power data (in unit of W) collected every 15 minutes as described above to calculate the energy output (in unit of kWh) of a specific module during a certain time period. We then compare the energy output of this module with that of the other PV panels in the same power station. If the energy output of this module is lower than a certain threshold, we consider it as a faulty module.
In a normal distribution:

| Range | Probability | 
| ------- | ------- | 
| x < u-σ | 15.87% |
| x < u-2σ | 2.28% |
| x < u-3σ | 0.13% | 

<img src="https://github.com/ZinanLin-Oscar/Statistical-Analysis-for-Solaredge-PV-Fault-Detection/assets/113269274/7e944cb2-bc14-4ed4-99f6-823c41d0cd6f" alt="image" style="width: 500px;"/>

- **Prerequisite：** The historical power generation data has been crawled and saved in a local folder using the method described in Section 2. The accurate hierarchical structure of data folders should be as follows:
```
- crawl_data_folder
  - 2024-01-01
    - site1_name.csv
    - site2_name.csv
    - site3_name.csv
    - ...
  - 2024-01-02
    - site1_name.csv
    - site2_name.csv
    - site3_name.csv
    - ...
  - ...
```

| Parameter | Description | 
| ------- | ------- | 
| crawl_data_folder | The local folder path for storing historical data of crawled module levels | 
| target_station | The name of the PV station conducting fault detection | 
| start_date | The start date of the fault detection cycle (yyyy-mm-dd) | 
| end_date | The end date of the fault detection cycle (yyyy-mm-dd) | 
| threshold | The multiple of standard deviation subtracted for fault detection | 
| site_list | A list of the names of all power stations associated with your SolarEdge account | 

| Section | Description | 
| ------- | ------- | 
| ``` def calculate_daily_energy(file_path) ``` | A function to calculate daily energy generation for PV panels in a single station | 
| ``` def sum_daily_energy(target_station, start_date, end_date) ``` | A function to sum the daily energy by module within a specific operation period | 
| ``` def find_low_energy_pv(target_station, start_date, end_date,threshold) ``` | A function to find the PV panels with low energy | 

| Detection range | Parameter settings | Result example | 
| ------- | ------- | ------- | 
| A specific station | target_station='UST Shaw Auditorium'<br>start_date='2024-04-06'<br> end_date='2024-04-08<br>threshold=0.3| ```The following PV panels at UST Shaw Auditorium generated less energy than 0.3 standard deviations below the mean:['241559551', '241559563', '241559575', '241559620', '241559634', '241559641']```|
| All stations | start_date='2024-04-06'<br> end_date='2024-04-08'<br>threshold=0.5 | ```The following PV panels at UST SQ37_48 generated less energy than 0.5 standard deviations below the mean:<br>['172430269', '172430272', '172430287', '172430297']<br>The following PV panels at UST SQ25_36 generated less energy than 0.5 standard deviations below the mean:<br>['172845718', '172845720', '172845733', '172845738']<br>The following PV panels at UST Shaw Auditorium generated less energy than 0.5 standard deviations below the mean:<br>[]```|


# Feedback
Feel free to send any questions/feedback to [Zinan LIN](zlinby@connect.ust.hk).
