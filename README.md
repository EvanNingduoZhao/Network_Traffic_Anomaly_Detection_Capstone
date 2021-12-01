# Network_Anomaly_Detection_Capstone
This repository contains the code files for both the data preprocessing stage and model training stage of the Network Anomaly Detection Capstone project. This README documents the usage guide for both stages. As a high-level overview of the entire project, in the preprocessing stage, the pcap file parser (which operates on an AWS EC2 instance in our setup)reads in raw pcap files, groups individual packets into flows and labels each flow based on its sender and receiver IP addresses. Then it performs train-test-validation split on the labeled flow data, outputs the splitted data as .hdf5 files and upload them to an AWS S3 bucket. Then, in the model training stage, a notebook operating on an AWS Sagemaker instance reads in the .hdf5 files from the S3 bucket, trains and validates the LUCID model using the data.
## Data Preprocessing
The data preprocessing stage is consisted of two steps:
1. parse pcap files into python native data structure and output a pickle file
2. perform train-test-validation split on output of step 1

### Initial Setup procedures:
To help you simplify the setup process, we have already setup the entire development environment of the project on an AWS EC2 instance and created an AMI (Amazon Machine Image) of the environment. Please use the AMI (ami-074b9810e5373c69c) to create your own EC2 instance. As a referrence, the type of EC2 instance that we used in our setup was c5n.9xlarge.

### Step1:
Specifically, in step 1, the pcap parser reads in unzipped pcap files to group packets into flows and label each flow based on its sender and receiver ip address.

After connecting to an AWS EC2 instance:
create a directory to store raw pcap data
```
cd pcap_parser
mkdir raw_data
cd raw_data
```
download zipped pcap file(s) into the created directory from the CIC site
(Note: the download command below is only for downloading one of the zipped pcap file from the CIC dataset, you can swap the file name at the end of the url to download other pcap files from the CIC dataset)
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
rm raw_data/*.pcap
```
After executing the command above, the raw_data folder should only contain the output of step 1, which is a file with .data extension.
### Step2:
Specifically, step2 reads in the pickle file output by step1, performs train-test-validation split on the labeled flow data and output the results as .hdf5 files. Note: the --preprocess_folder parameter has to be the folder that stores the result from step 1.
```
python3 lucid_dataset_parser.py --preprocess_folder /home/ubuntu/pcap_parser/raw_data
```
Uploading the data preprocessing results to an AWS
aws s3 cp raw_data s3://[S3 bucket Name]/[Folder name inside the bucket] --recursive
## Model Training and Validation
The code along with detailed comments about the specific function of each code segment for the model training and validation stage is in the LUCID_model.ipynb file. To execute LUCID_model.ipynb, we recommend deploying an AWS SageMaker instance and execute the python notebook file on the instance. (Note: Please remember to change the S3 bucket path at the top of the notebook to the path to the S3 bucket that you are using)
