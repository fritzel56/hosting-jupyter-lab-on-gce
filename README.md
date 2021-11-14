# Hosting a JupyterLab on a Google Cloud Compute Engine

## Motivation

I was taking part in a mentoring program and wanted to find a way that we could share a compute environment. This would allow us to simplify the troubleshooting process as both parties are using the same environment. I decided that JupyterLab hosted in a cloud environment would work well.

## Thanks to

There were a number of online guides but worked quite as written. A big thank you to nyghtowl's article [here](https://medium.com/@nyghtowl/setup-jupyter-notebook-access-on-google-compute-engine-with-https-ad69297f438b) and Alara Dirik's article [here](https://towardsdatascience.com/deploying-a-custom-ml-prediction-service-on-google-cloud-ae3be7e6d38f) for providing good starting points.

## Setup

1. Head over to Google's cloud compute engine page [here](https://console.cloud.google.com/compute)
2. Click `CREATE INSTANCE`
3. Key things to change:
   -  Machine configuration: Machine family: Series: E2 (I assume it should work with other series too but only tested with E2 series)
   - Machine configuration: Machine family: Machine Type: any (tested with a variety and worked with all)
   - Firewall: select both `Allow HTTP traffic` and `Allow HTTPS traffic`
4. Head to bottom and click CREATE
5. Back on the compute page, see your instance and, under Connect, click SSH (note: this uses popups which your browser may block so check for a notification about blocked popups). This should open a new tab with a box asking: `Do you want to initiate an SSH connection to VM instance 'large-conda-e24'?`. Click `Connect`. This will log you into a terminal connected to your instance.
6. Update your instance and install useful packages:
```
sudo apt-get update
sudo apt-get install bzip2 libxml2-dev libsm6 libxrender1 libfontconfig1 wget git
```
7. Setup miniconda (say yes to the prompts):
```
wget https://repo.anaconda.com/miniconda/Miniconda3-4.7.10-Linux-x86_64.sh
bash Miniconda3-4.7.10-Linux-x86_64.sh
```
8. Export the new location to path and remove the download. Make sure you update the first command to replace `<your name here>` with your google account name (if uncertain what this is, see the command prompt in your terminal which should be of the form `<your name>@<instance name>`.
```
export PATH=/home/<your name here>/miniconda3/bin:$PATH
rm Miniconda3-4.7.10-Linux-x86_64.sh
```
9. Check that you have conda installed with:
```
which conda
```
10. This should return something like `/home/<your name here>/yes/bin/conda`. If not, I'm sorry and google is your best friend.
11. Restart the terminal:
```
exec bash
```
12. Set up your environment and install jupyter:
   1. run: `conda create -n myenv`
   2. run: `conda activate myenv`
   3. run: `conda install jupyter jupyterlab`
   4. install any other python packages you want: `conda install package1, package2 etc.`
13. Set up jupyterlab. The second line will prompt you to create a password which will protect your server from random internet people.
```
jupyter lab --generate-config
jupyter lab password
```
14. Get yourself a static IP address so you can consistently interact with your server. Head over to the Google's cloud console's VPC Network: External IP Addresses (direct link [here](https://console.cloud.google.com/networking/addresses/list?project=stalwart-kite-170323). If this is your only instance, there should only be a single line. If there are multiple, match the External Address shown for the instance shown on the Compute page. Select the IP Address and click `RESERVE STATIC ADDRESS`.
15. Start your server:
```
jupyter lab --ip=0.0.0.0 --port=8888 --no-browser &
```
16. Navitagate to your server by putting this into your address bar in a new tab:
```
http://<your external ip address>:8888/lab/
```
17. Enter your password and welcome to your JupyterLab hosted on Google cloud compute!