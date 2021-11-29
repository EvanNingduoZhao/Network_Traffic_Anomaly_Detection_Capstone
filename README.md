# Network_Anomaly_Detection_Capstone
This repository contains the code files for both the data preprocessing stage and model training stage of the Network Anomaly Detection Capstone project. This README documents the usage guide for both stages. As a high-level overview of the entire project, in the preprocessing stage, the pcap file parser (which operates on an AWS EC2 instance in our setup)reads in raw pcap files, groups individual packets into flows and labels each flow based on its sender and receiver IP addresses. Then it performs train-test-validation split on the labeled flow data, outputs the splitted data as .hdf5 files and upload them to an AWS S3 bucket. Then, in the model training stage, a notebook operating on an AWS Sagemaker instance reads in the .hdf5 files from the S3 bucket, trains and validated the LUCID model using the data.
## Data Preprocessing
The data preprocessing stage is consisted of two steps:
1. parse pcap files into python native data structure and output pickle
2. perform train-test-validation split on output of step 1
### Step1:
Specifically, in step 1, the pcap parser reads in unzipped pcap files to group packets into flows and label each flow based on its sender and receiver ip address.

After connecting to an AWS EC2 instance and cloning the repository to the instance:
create a directory to store raw data
```
mkdir raw_data
cd raw_data
```
download zipped pcap file into the created directory from the CIC site
```
curl -o ./PCAP-01-12_0-0249.zip http://205.174.165.80/CICDataset/CICDDoS2019/Dataset/PCAPs/01-12/PCAP-01-12_0-0249.zip
```
unzip the downloaded pcap file
```
unzip PCAP-01-12_0-0249.zip
```
delete the zip file
```
rm *.zip
```
add .pcap file extension to each of the unzipped files and go back to the parent directory
```
for FILENAME in *; do mv $FILENAME $FILENAME.pcap; done
cd ..
```
perform step 1 of pcap files parsing, which groups packets into flows and labels each flow, store labeled flow data into a python native data structure and output the result as a pickle file.
```
python3 lucid_dataset_parser.py --dataset_type DDOS2019 --dataset_folder /home/ubuntu/pcap_parser/raw_data --packets_per_flow 10 --dataset_id DDOS2019 --traffic_type all --time_window 10
```
The following parameters can be specified when using lucid_dataset_parser.py:
- --dataset_folder: Folder that contains the .pcap files
- --output_folder : Folder where the scripts saves the output. The dataset folder is used when this option is not used
- --traffic_type : Type of flow to process (all, benign, ddos)
- --preprocess_folder : Folder containing the intermediate files *.data (this option should only be used when performing step2 of the preprocessing)
- --dataset_type : please stick with using DDOS2019 since the parser code is cutomized for CIC DDOS2019 dataset
- --packets_per_flow : Maximum number of packets in a sample
- --time_window : Length of the time window (in seconds)
- --dataset_id : String to append to the names of output files

remove the .pcap files from the raw_data folder to prepare for step 2:
```
rm whole_data/*.pcap
```
### Step2:
Specifically, step2 reads in the pickle file output by step1, performs train-test-validation split on the labeled flow data and output the results as hdf5 files
```
python3 lucid_dataset_parser.py --preprocess_folder [absolute path to the directory that contains the result of step1]
```
