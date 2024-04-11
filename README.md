# OS Fingerprinting

Data processing and machine learning application of PCAP (packet capture, .pcap) data to passively identify operating systems.

# Installation on Debian Linux:

`chmod +x configure/configure.sh`

`sudo configure/configure.sh`

OSirisML is currently only available for installation on Debian Linux because it is reccomended to run on a server with high `RAM` capacity. Through basic testing, a PCAP file of about `8` gb is reccomended to have at least `128` gb of RAM.

See `/configure/installation_instructions.txt` for more information on how to install dependencies.

# Overview of workflow

[Workflow Diagram PDF](OSirisML.pdf)

This open-source tool is built off the work on passive OS detection nprint and nprintML.

https://arxiv.org/pdf/2008.02695.pdf

Given a PCAP file and identifying source IPs, a script is run to call `tcpdump` on the PCAP file for each source IP, so the model is given classiciation labels for each element.

https://www.tcpdump.org/

Using nprint, the open-source PCAP preprocessing tool, the PCAP data is transformed into `.npt` data.

https://github.com/nprint/nprint

Note: OSirisML is configured to work with `nprint-1.2.1`. To use a newer version, see nprint's github for installation instructions and replace the `tar` file in `/configure`

These `.npt` files are combined to a single `CSV` file using a custom script in /preprocessing

This CSV file is split into `X_train`, `X_test`, `Y_train`, and `Y_test` data, where X is the `960` attributes of tabular data, and Y is the corresponding operating system classification. This is done with `Pandas`, an open-source data manipulation tool.

https://pandas.pydata.org/

The payload and source IP bytes of the packet are dropped from the dataframe and not considered in the model.

This data is trained using `XGBoost`, an open-source machine learning tool that uses gradient boosting. By default, a test size of `0.2` is used, unless specified as an additional sys argument. See usage for `model/xgboostmodel.py`

https://xgboost.readthedocs.io/en/stable/

# Results with CIC-IDS2017 Dataset

Replicating section 5.2 with xgboost saw an Accuracy score of 74.38% with an F1 Score of 75.48%

To create the appropriate labels use tcpdump to separate the source IP addresses into 13 separate pcap files, then run the label generation script provided in the paper above.

Here is the table provided by University of New Brunswick:
- Web server 16 Public: 192.168.10.50, 205.174.165.68
- Ubuntu server 12 Public: 192.168.10.51, 205.174.165.66
- Ubuntu 14.4, 32B: 192.168.10.19
- Ubuntu 14.4, 64B: 192.168.10.17
- Ubuntu 16.4, 32B: 192.168.10.16
- Ubuntu 16.4, 64B: 192.168.10.12
- Win 7 Pro, 64B: 192.168.10.9
- Win 8.1, 64B: 192.168.10.5
- Win Vista, 64B: 192.168.10.8
- Win 10, pro 32B: 192.168.10.14
- Win 10, 64B: 192.168.10.15
- MAC: 192.168.10.25
- Kali: 205.174.165.73


# Implementation with PCAP data

To use OSirisML with any dataset, the network data needs to be sorted by source IP. This is done best in a controlled environment, where each source IP is a unique OS.

Modify the /preprocessing/tcp_dump.sh script to label each source IP with the corresponding operating system.


# TODO - Ordered by priority tier

Optimizing model:

- Optimize model with max_depth and eta params
- Retrain model on other datasets

Creating command line tool for generic use:

- Have single command to execute workflow with params for: csv file for number and type of OSes, decimal value of training set size (pass in to xgboost.py, left to 0.2 by default if none given),
- option to remove data files after each step of workflow (remove the .pcaps after they are all converted to npt, then removed the .npts after they are all csv)
- Create test directory with small test pcap file (see nprint's repo)

Extra:

- Create markdown README with documentation and command line prints for errors and usage