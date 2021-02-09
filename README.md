# Terramearth - implementation of terramearth case study

# Phase - 1

1. Open Cloud-shell.

2. Make your project as current project.

    gcloud config set project project-id

3. Create a cloud pub/sub topic.

   gcloud pubsub topics create terramearth-topic

4. Create a subscription to receive messages from pub/sub topic.

   gcloud pubsub subscriptions create terramearth-sub --topic terramearth-topic

5. Send some messages on the topic.

   gcloud pubsub topics publish terramearth-topic --message hello-world

6. Pull messages from the topic.

   gcloud pubsub subscriptions pull terramearth-sub --auto-ack


Now your pub/sub topic is set.



# Phase - 2

1. Create a public and private key for cloud iot core.
   
   mkdir ca

   cd ca

   openssl req -x509 -newkey rsa:2048 -keyout rsa_private.pem -nodes -out rsa_cert.pem -subj "/CN=unused"

   private key-file: rsa_private.pem

   public key-file: rsa_cert.pem

2. Display contents of public key-file
 
   cat rsa_cert.pem

3. Copy the all contents.

4. Go to cloud core console.

5. Create a new registry with following settings:
   
    Registry ID - terramearth-registry

    Region - asia-east1

    cloud pub/sub topic - terramearth-topic

    Click on the advanced option and keep default settings. 

    CA certificate - insert public key-file contents

    Click on create.

    Now your registry is ready.

6. Click on Device. Create a new device with following settings:

    Device ID - dragline-100

    publicKey Format - RS256_X509

    leave default settings and clock on create.

7.  Now, Install the required software on cloud-shell.

    apt install git

    curl -sL https://deb.nodesource.com/setup_14.x -o nodesource_setup.sh

    bash nodesource_setup.sh

    apt install -y nodejs

6.  Download application code.
     
     git clone https://github.com/shalligoel/terramearth

7. Make terramearth as current folder.

   ls

   npm install

8. Copy private-file from ca folder to terramearh folder

   cp ~/ca/rsa_private.pem ~/terramearth

9.  send messages on cloud iot core by the following command.


   node terramearth.js mqttDeviceDemo --projectId=$DEVSHELL_PROJECT_ID --cloudRegion=asia-east1 --reg
istryId=terramearth-registry --deviceId=dragline-100 --privateKeyFile=rsa_private.pem --numMessages=2 --algorithm=RS256


10. Retrieve messages through pub/sub scbscriber.
    
    gcloud pubsub subscriptions pull terramearth-sub --auto-ack


11. If you can retrieve messages, means everything is set and ready for phase-3.



# Phase -3 

1. Go to Bigquery console.

2. Select your project and create a new dataset with following settings.
   
   Dataset Name - terramearh_dataset
   
   Default Location - asia-southeast1
   
   Default table expiration - None
   
   click on create.

3. Select the dataset and create a new table with following settings.

   Source - Empty table
   
   Table Name - terramearth_table
   
   Schema:
   
   registry:STRING,deviceID:STRING,deviceName:STRING,pressure:FLOAT,speed:INTEGER,engineUpTime:INTEGER,oilLevel:INTEGER,timestamp:TIMESTAMP

   Leave Default setting and click on create.

4. Now your Bigquery dataset and table is ready to get messages.


5. Now create dataflow job from template. Goto Dataflow console.

6. Click on Create job from template with following settings.

Job name - terramearth-job

Regional endpoint - asia-southeast1

Dataflow template - pub/sub topic to BigQuery

input pub/sub topic - projects/project-id/topics/terramearth-topic

BigQuery output table - project-id:terramearth_dataset.terramearth_table


Temporary location - gs://bucket-id/temp

Max workers - 2

Number of workers - 1

Worker region - asia-southeast1

Machine type - n1-standar-1

Network - custom VPC 

subnetwork - https://www.googleapis.com/compute/v1/projects/project-id/regions/asia-southeast1/subnetworks/subnet-id

Note: Make sure you have subnet in asia-southeast1 otherwise job will fail.

click on Run Job.


5. Make sure your job is running successfully.

6.  Send messages from cloud-shell.

   node terramearth.js mqttDeviceDemo --projectId=$DEVSHELL_PROJECT_ID --cloudRegion=asia-east1 --registryId=terramearth-registry --deviceId=dragline-100 --privateKeyFile=rsa_private.pem --numMessages=10 --algorithm=RS256

7. Wait for some time.

8. Go to Bigquery and refresh.

9. Check your table to see messages.

10. You got messages from cloud iot core to Bigquery through Dataflow job successfully.

You are ready for next phase now.


