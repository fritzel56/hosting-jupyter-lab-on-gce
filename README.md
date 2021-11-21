# Hosting a JupyterLab on a Google Cloud Compute Engine

## Motivation

I was taking part in a mentoring program and wanted to find a way that I could share a compute environment with my mentee. Having a hosted JupyterLab instance would allow us to simplify the troubleshooting process as both parties are using the same environment.

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
8. Export the new location to path. Make sure you update the first command to replace `<your name here>` with your google account name (if uncertain what this is, see the command prompt in your terminal which should be of the form `<your name>@<instance name>`.
```
export PATH=/home/<your name here>/miniconda3/bin:$PATH
```
9. Remove the download:
```
rm Miniconda3-4.7.10-Linux-x86_64.sh
```
10. Check that you have conda installed with:
```
which conda
```
11. This should return something like `/home/<your name here>/yes/bin/conda`. If not, I'm sorry and google is your best friend.
12. Restart the terminal:
```
exec bash
```
13. Set up your environment and install jupyter:
   1. run:
```conda create -n myenv```
   2. run:
```conda activate myenv```
   3. run:
```conda install jupyter jupyterlab```
   4. install any other python packages you want: `conda install package1, package2 etc.`
14. Set up jupyterlab. The second line will prompt you to create a password which will protect your server from random internet people.
```
jupyter lab --generate-config
jupyter lab password
```
15. Get yourself a static IP address so you can consistently interact with your server. Head over to the Google's cloud console's VPC Network: External IP Addresses (direct link [here](https://console.cloud.google.com/networking/addresses/list?project=stalwart-kite-170323). If this is your only instance, there should only be a single line. If there are multiple, match the External Address shown for the instance shown on the Compute page. Select `RESERVE` on the left.
16. Start your server:
```
jupyter lab --ip=0.0.0.0 --port=8888 --no-browser &
```
17. Navitagate to your server by putting this into your address bar in a new tab:
```
http://<your external ip address>:8888/lab/
```
18. Enter your password and welcome to your JupyterLab hosted on Google cloud compute!

### HTTP vs HTTPS

The article by nyghtowl [here](https://medium.com/@nyghtowl/setup-jupyter-notebook-access-on-google-compute-engine-with-https-ad69297f438b) walks you through hosting an HTTPS server. I opted not to go this way because, with the self-signed SSL certificate, the browser redirects you to a warning page, making it harder to login and more concerning to people that I share the server with. With HTTP, Firefox shows a warning in the address bar but takes you directly to the server.


# Using Plotly Express with JupyterLabs and GCE

I was expecting that, having followed the above steps, I could then install plotly express into my virutal environment with the following:
```
conda install -c plotly plotly_express
```
and then run plotly express in my jupyter notebook; however, when I then ran plotly's example code from [here](https://plotly.com/python/getting-started/):
```python
import plotly.graph_objects as go
fig = go.Figure(data=go.Bar(y=[2, 3, 1]))
fig.show()
```
the cell runs but no graph is rendered - just a large empty white space (see example below):

![Missing Graph](https://raw.githubusercontent.com/fritzel56/hosting-jupyter-lab-on-gce/master/images/missing-plotly-graph.PNG)

Again, there are a number of sites discussing how to remedy this but none quite worked for me. In the end, I tracked it down to nodejs being missing. It appears that plotly specifically needs nodejs version 12. I tried several installation methods which installed older versions. What ended up working for me was inspired by [this article](https://computingforgeeks.com/how-to-install-nodejs-on-ubuntu-debian-linux-mint/):

```
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install -y nodejs
```

You can then test that you have the correct version installed by running:
```
node -v
```
which should return something along the lines of `v12.xx.x`

Having done this and refreshing my connection, my graph now renders:

![Rendered Graph](https://raw.githubusercontent.com/fritzel56/hosting-jupyter-lab-on-gce/master/images/plotly-graph-rendered.PNG)
