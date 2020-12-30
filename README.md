# Overview
This project is to build a simple motion detection gadget that can describe the motion as it is detected by the camera

## Setup
### OS on RaspberryPi
```
cat /etc/os-release
```

Raspian on raspberry pi

PRETTY_NAME = "Raspian GNU/Linux 10 (buster)

VERSION="10(buster)"

ID =raspian

ID_LIKE=debian


### Update RaspberryPi
```
sudo apt-get update
sudo apt full-upgrade
```

## Projects
The whole solution includes the following:
configurations on RaspberryPi and DoorBell.sln that includes following projects:
1. DoorBell.AzureFunction
2. DoorBell.DataContracts
3. DoorBell.Storage
4. DoorBell.Web

### 1. Configuration on RaspberryPi
1. On RaspberryPi, motion library is used to capture and record motion. The updated motion.conf file is included in the repo
2. A custom python (uploadFile.py) file is added to RaspberryPi. This file connects with the IotHub. Once the motion library saves the motion as .mp4 file, it executes the uploadFile.py. This file extracts frames from the .mp4 file, uploads the frames and the original mp4 file to a blob storage using the IoTHub's file upload mechanism and then sends a message to the IotHub that a new event happened including the details.

### 2. DoorBell.AzureFunction 
The azure function gets triggered everytime the IotHub receives a message. The function does the following:
1. For each frame (jpg) file stored (by the RaspberryPi), calls Azure Computer Vision sdk to get different vision features (including the captions) for that image.
2. Store the details received from the Azure Computer Vision service along with the message in the Cosmos DB.

### 3. DoorBell.DataContracts
This is data contracts library that is used across the projects.
### 4. DoorBell.Storage
This is a web api that wraps around the CosmosDB to save and fetch device messages along with the processed information received from the Azure Computer Vision service.

### 5. DoorBell.Web
This is a webApp that shows the events, captions (description) received from Azure Computer Vision services along with the original video (mp4) file recorded by the RaspberryPi

### Project setup on RaspberryPi
use pip3 to use python 3

use --user option when installing python libraries in the user's python install directory

From the project repo run pip3 install to install all the required packages
```
pip3 install -r requirements.txt --user
```

Setup azure credentials for keyvault.

login to shell.azure.com/bash
```
az ad sp create-for-rbac --name http://my-application --skip-assignment
export AZURE_CLIENT_ID="generated app id"
export AZURE_CLIENT_SECRET="random password"
export AZURE_TENANT_ID="tenant id"
```

Authorize the service principal to perform key operations in your Key Vault:
```
az keyvault set-policy --name my-key-vault --spn $AZURE_CLIENT_ID --secret-permissions get set list
```

# References
1. RaspberryPi docs: https://www.raspberrypi.org/documentation/raspbian/
2. Motion library on RaspberryPi (version 4.1.1):https://motion-project.github.io/4.1.1/motion_guide.html 
3. Azure IotHub Python samples: https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-python-python-file-upload
4. CosmosDb Samples: https://docs.microsoft.com/en-us/azure/cosmos-db/sql-api-dotnet-v3sdk-samples
5. Azure Functions: https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-event-iot

## Note:
Sorry, the code repo is still kept private :)

# The output
Here is a screenshot from the DoorBell.Web app of the results:
Events List:

![Alt text](DocImages/HomePage1.png?raw=true "Event list")

Event Description with original video:

![Alt text](DocImages/whiteTruck.png?raw=true "WhiteTruck")