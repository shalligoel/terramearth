# terramearth - implementation of terramearth case study

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
