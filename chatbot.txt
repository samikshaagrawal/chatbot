# Import necessary modules
from rasa_nlu.training_data import load_data
from rasa_nlu import config
from rasa_nlu.model import Trainer
import spacy
nlp=spacy.load('en')
bot_template = "BOT : {0}"
user_template = "USER : {0}"
# Create args dictionary
args = {"pipeline":"spacy_sklearn"}

# Create a configuration and trainer
training_data = load_data("C:/rasa-nlu-trainer/src/state/testData.json")
trainer = Trainer(config.load("C:/Users/lenovo/rasa_nlu/sample_configs/config_spacy.yml"))
trainer.train(training_data)

# Create an interpreter by training the model
interpreter = trainer.train(training_data)

import sqlite3
connection=sqlite3.connect('project.db')
cursor=connection.cursor()
params={'Symptoms':[]}

def find_diseases(params):
    cursor.execute("SELECT symptoms FROM DISEASES")
    symptoms=list(cursor.fetchall())
    sym=[]
    for s in symptoms:
        flag=0
        check=str(s)
        listy=list(params.values())
        for value in listy[0]:
            if(check.find(value.lstrip())==-1):
                flag=1
        if flag==0:
            sym.append(check[2:-3])
    query="select disease from diseases where symptoms = ?"
    cursor.execute(query,sym)    
    return cursor.fetchall()

def respond(message,params): 
    responses=["I'm sorry :( I could not identify the disease","you might have {}","you might have {} or {}","I'm confused between {} , {} and {}"]
    data=interpreter.parse(message)
    intent=data["intent"]["name"]
    if intent=="affirm":
        return "hello! I'm an automatic symptom checker ",params
    elif intent=="goodbye":
        return "Good-bye!",params
    elif intent=="deny":
        results=find_diseases(params)
        names=[r[0] for r in results]
        n=min(len(results),3)
        return responses[n].format(*names),params 
    elif intent=="greet":
        return "yeah! go on. I'm listening",params
    elif intent=="not well":
        return "I am here to help! please specify your symptoms.",params
    elif intent=="Symptom Identification":

        for ent in data["entities"]:
            params['Symptoms'].append(ent["value"])
        return "Do you have any other symptoms?",params
    else:
        return"I'm sorry - I'm not sure how to help you",params


def send_message(message):
    # Print user_template including the user_message
    print(user_template.format(message))
    # Get the bot's response to the message
    response,param = respond(message,params)
    # Print the bot template including the bot's response.
    print(bot_template.format(response))
messages=["hey","i'm not feeling well","I have abdominal pain","yes","I have fever","yes","i'm bloated","yes","i had vomit","yes","i lost weight","yes","i lost my appetite","noi","bye"]
for message in messages:
    send_message(message)
