# Rasa DB Demo with External HayStack



This repo is a bare-bones chat bot project using [Rasa](https://rasa.com/) in combination with [Haystack](https://github.com/deepset-ai/haystack) along with SqliteDB. 
While Rasa is used for the whole flow of the dialogue and intent management, 
Haystack is used to answer the long tail of "knowledge queries" that can be answered by searching an answer in a document corpus. 
This example repo sketches how to use the _fallback intent_ or an dedicated _knowledge base intent_ to offload such queries to haystack.

#  Installing and checking dependencies. 

## To check the docker version

<code>docker -v && docker-compose -v</code>

**If this returns an error do the following steps else jump to [Getting Stared](https://github.com/athenasaurav/Rasa_Haystack_DB/blob/main/README.md#get-started)**

Docker :

```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```
Docker Compose :
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose 
sudo chmod +x /usr/local/bin/docker-compose
```
<hr>

# Get Started

- Start the Haystack REST API and a demo DocumentStore along with RASA connected to custome tracker store using SQLiteDB via Docker:
```
git clone https://github.com/athenasaurav/RASA_EXT_HayStack.git
cd RASA_EXT_HayStack
docker-compose build
docker-compose up -d
``` 
- This will automatically train a model in this repository with `rasa train`  

# Interacting with RASA

- To Interact with rasa we can use either of the following endpoints.

## Using Default REST

- Send POST request to ```http://ip:5005/webhooks/rest/webhook```
- Request format should be like :
```
{
    "message" : "hi",
    "sender": "sender"
}
```
- You will receive response like this:
```
{
    "message" : "Hi! How can i help you?",
    "sender": "sender"
}
```

## Using custom REST channel named senderid

- Send POST request to ```http://ip:5005/webhooks/senderid/webhook```
- Request format should be like :
```
{
    "message" : "hi",
    "sender": "sender"
}
```
- You will receive response like this:
```
{
    "message" : "Hi! How can i help you?",
    "sender": "sender"
}
```
## Customizing Custom REST Url

To customize the rasa custom url we need to make following changes in Three files. 

Step 1) Customize the rest url name in ```MyIO.py``` in ```actions/connector```

```
nano actions/connector/MyIO.py
```

Step 2) Make these following changes in the ```MyIO.py``` file. For Ex: If you want the custom REST url should be ```/webhooks/example/webhook``` instead of ```/webhooks/senderid/webhook```

- Change Line no. 12, Class name from ```class senderidInput(InputChannel):``` to ```class exampleInput(InputChannel):```
- Change Line no. 15, Return name from ```return "senderid"``` to ```return "example"```
- Change Line no. 104, collector form ```collector = senderidOutput()``` to ```collector = exampleOutput()```
- Change Line no. 130, Class name from ```class senderidOutput(CollectingOutputChannel):``` to ```class exampleOutput(CollectingOutputChannel):```
- Change Line no. 133, Return name from ```return "senderid"``` to ```return "example"```

Save this changes. 

NOTE : Please Check your doing it all from Root Directory.

Step 4) Copy the Newly edited ```MyIO.py``` from ```actions/connector``` to ```backend/connector```

```
rm backend/connector/MyIO.py

cp actions/connector/MyIO.py backend/connector/MyIO.py

```
Step 5) Update the name in the ```backend/credentials.yml```
```
nano backend/connector/credentials.yml
```
Change the Following : Line no 9 : ```connector.MyIO.senderidInput:``` to ```connector.MyIO.exampleInput:```

Step 6) Save and Exit. Your Custom REST API Endpoint is ready to be deployed. 

Step 7) Rebuild and Restart the docker

```
docker-compose down
docker-compose build
docker-compose up -d
```
Step 8) Now you can send POST request to ```http://ip:5005/webhooks/example/webhook```

## Customizing Custom Tracker Store

To Customize Tracker Store we can set it in ```backend/endpoints.yml```

```
tracker_store:
    type: SQL
    dialect: "sqlite"  # the dialect used to interact with the db
    url: "sqlite:///./rasa.db"  # (optional) host of the sql db, e.g. "localhost"
    db: "rasa.db"  # path to your db
    username:  # username used for authentication
    password:  # password used for authentication
    query: # optional dictionary to be added as a query string to the connection URL
      driver: my-driver
```

Set your Databse url, DB name and DB path accordingly. Incase if we dont change anything it will create a default rasa.db in docker which will only be accessiable inside docker. 


# RUN RASA WITH EXTERNAL HAYSTACK

To run RASA with SQLite DB Tracker Store and Haystack Included. Please visit [RASA_without_Haystack](https://github.com/athenasaurav/Rasa_Haystack_DB.git)
