# Statistical-Analysis-for-Solaredge-PV-Fault-Detection
This Github repository provides Python code for the automated retrieval of Solarledge module-level power generation data, and the subsequent application of statistical analysis to detect faults in PV modules.

# 1. Overview
[Solaredge monitoring platform](https://www.solaredge.com/en/products/software-tools/monitoring-platform) is a web-based platform that allows users to monitor and optimize the performance of their solar PV systems in real-time. [Solaredge Monitoring API](https://developers.solaredge.com/) currently only provides the ability to download Site and Inverter level data. In order to obtain Module level data, one must manually download it from the Solaredge Monitoring Platform. This process is time-consuming and cannot meet the needs of periodic automatic batch downloading and analysis of Module level power generation data.

We have developed a Python code that enables Solaredge users to automatically retrieve module level power generation data for a single PV station or for all PV stations under their account. The prerequisite for this work is:
1) PV station is equipped with module optimizers (which are the smallest measuring units for PV power generation);
2) The measurement data of the module optimizers is publicly available on the Solaredge Monitoring Platform (related to user settings).

