# Config_Setup
# Running the Application (Industrial-Visual-Analysis)
Follow these steps to setup and run the application. The steps are described in detail below.

## Steps
1. [Watson Visual Recognition Setup](#1-Watson-Visual-Recognition-Setup)
2. [Cloudant NoSQL DB Setup](#2-Cloudant-NoSQL-DB-Setup)
3. [IBM Cloud Functions Setup](#3-IBM-Cloud-Functions-Setup)
4. [Run Web Application](#4-Run-Web-Application)

## 1. Watson Visual Recognition Setup

Create the [Watson Visual Recognition](https://www.ibm.com/watson/services/visual-recognition/) service in IBM Cloud.  You will need the ``API Key``.

* Open a command line interface (CLI) on your desktop and clone this repo:
```
git clone https://github.com/IBM/Predictive-Industrial-Visual-Analysis
```

* Go to the folder where the images are placed
```
cd Predictive-Industrial-Visual-Analysis/vr-image-data
```

Here we will create a classifier using the zipped images to train the Watson Visual-Recognition service. The images in each zipped folder are used to make the Watson VR service become familiar with the images that relate to the different categories (Corrosion, Leak, etc.). Run the following command to submit all 6 sets of images to the Watson service classifier:

```
curl -X POST -u "apikey:{INSERT-YOUR-IAM-APIKEY-HERE}" -F "Bursted_Pipe_positive_examples=@Burst_Images.zip" -F "Corroded_Pipe_positive_examples=@Corrosion_Images.zip" -F "Damaged_Coating_positive_examples=@Damaged_Coating_Images.zip" -F "Joint_Failure_positive_examples=@Joint_Failure_Images.zip" -F "Pipe_Leak_positive_examples=@Leak_Images.zip" -F "Normal_Condition_positive_examples=@Normal_Condition.zip" -F "name=OilPipeCondition" "https://gateway.watsonplatform.net/visual-recognition/api/v3/classifiers?version=2018-03-19"
```

The response from above will provide you with a status on the submission and will give you a `CLASSIFIER_ID`. Please copy this for future use as well. After executing the above command, you can view the status of your Watson service and whether it has finished training on the images you submitted. You can check the status like this:

```
curl -X GET -u "apikey:{INSERT-YOUR-IAM-APIKEY-HERE}"  "https://gateway.watsonplatform.net/visual-recognition/api/v3/classifiers/{INSERT-CLASSIFIER-ID-HERE}?api_key={INSERT-API-KEY-HERE}&version=2018-03-19"
```

You can find more information on working with your classifier [here](https://console.bluemix.net/docs/services/visual-recognition/tutorial-custom-classifier.html#creating-a-custom-classifier)

## 2. Cloudant NoSQL DB Setup

Create the [Cloudant NoSQL](https://www.ibm.com/analytics/us/en/technology/cloud-data-services/cloudant/) service in IBM Cloud.

Create a new database in Cloudant called <strong>image_db</strong>

<p align="center">
  <img width="600"  src="readme_images\cloudant_db.png">
</p>


Next, create a view on the database with the design name ``image_db_images``, index name ``image_db.images``, and use the following map function:
```
function (doc) {
if ( doc.type == 'image_db.image' ) {
  emit(doc);
}
}
```

<p align="center">
  <img width="600"  src="readme_images\cloudant_view.png">
</p>


## 3. IBM Cloud Functions Setup

We will now set up the IBM Cloud Functions (OpenWhisk) using Bluemix CLI.

#### [Setup and download the Bluemix CLI](https://console.bluemix.net/docs/cli/reference/bluemix_cli/download_cli.html#download_install)

* Install the Cloud Functions Plugin
```
bx plugin install Cloud-Functions -r Bluemix
```

* Log in to IBM Cloud, and target a Region (i.e api.ng.bluemix.net), Organization (i.e Raheel.Zubairy) and Space (i.e dev).
```
bx login -a {INSERT REGION} -o {INSERT ORGANIZATION} -s {INSERT SPACE}
```

#### API Authentication and Host

We will need the API authentication key and host.

* Command to retrieve API host:
```
bx wsk property get --apihost
```

* Command to retrieve API authentication key:
```
bx wsk property get --auth
```

__N.B:__ make sure what plan (Lite, etc.) you are associating when creating this service.


#### Configure .env file

You will need to provide credentials to your Cloudant NoSQL database and Watson Visual Recognition service, and Cloud Functions Host/Auth information retrieved in the previous step, into a `.env file`. Copy the sample `.env.example` file using the following command:

```
cp .env.example .env
```

and fill in your credentials and your VR Classifier name.

```
#From cloudant NoSQL database
CLOUDANT_USERNAME=
CLOUDANT_PASSWORD=
CLOUDANT_HOST=
CLOUDANT_URL=
CLOUDANT_DB=image_db
#From Watson Visual Recognition Service
VR_KEY=
VR_URL=
VR_CLASSIFIERS=OilPipeCondition_1063693116
#From OpenWhisk Functions Service in IBM Cloud
FUNCTIONS_APIHOST=
FUNCTIONS_AUTHORIZATION=
```

#### Run setup_functions.sh

We will now run the ``setup_functions.sh`` file to set up the microservice which triggers the Visual Recognition analysis as an image is added to the Cloudant database.

```
chmod +x setup_functions.sh
./setup_functions.sh --install
```
The above command will setup the OpenWhisk actions for you, there should be no need to do anything else if you see an Install Complete message with green OK signs in the CLI.


#### Explore IBM Cloud Functions

In IBM Cloud, look for ``Functions`` in ``Catalog``

There you will see a UI to ``Manage`` and ``Monitor`` the service. In addition, it has information for ``Getting Started`` and even ``Develop`` actions.

<p align="center">
  <img width="800"  src="readme_images\cloud_functions_scrnshot.png">
</p>


## 4. Run Web Application

#### Run locally

To run the app, go to the ```Industrial-Visual-Analysis``` folder and run the following commands.

* Install the dependencies you application need:

```
npm install
```

* Start the application locally:

```
npm start
```

Test your application by going to: [http://localhost:3000/](http://localhost:3000/)
