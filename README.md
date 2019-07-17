# Priya
from __future__ import print_function
import requests
import boto3

# --------------- Helpers that build all of the responses ----------------------

def build_speechlet_response(title, output, reprompt_text, should_end_session):
    return {
        'outputSpeech': {
            'type': 'PlainText',
            'text': output
        },
        'card': {
            'type': 'Simple',
            'title': "SessionSpeechlet - " + title,
            'content': "SessionSpeechlet - " + output
        },
        'reprompt': {
            'outputSpeech': {
                'type': 'PlainText',
                'text': reprompt_text
            }
        },
        'shouldEndSession': should_end_session
    }

def build_response(session_attributes, speechlet_response):
    return {
        'version': '1.0',
        'sessionAttributes': session_attributes,
        'response': speechlet_response
    }


# --------------- Functions that control the skill's behavior ------------------
def get_welcome_response():
    """ If we wanted to initialize the session to have some attributes we could
    add those here
    """
    session_attributes = {}
    card_title = "Welcome"
    speech_output = "Welcome to alexa voice for Technology news"
    reprompt_text = "I don't know if you heard me, welcome to your custom alexa application!"
    should_end_session = False
    return build_response(session_attributes, build_speechlet_response(
        card_title, speech_output, reprompt_text, should_end_session))

def get_news_response():
    session_attributes = {}
    card_title = "latestNews"
    
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('news')
    resource = table.scan()
    data = resource['Items']
    
    list = []
    for x in data:
        list = list+[x['headline']]
        
    


    speech_output = "list headlines are" +str(list)
    reprompt_text = "You never responded to the first test message. Sending another one."
    should_end_session = False
    return build_response(session_attributes, build_speechlet_response(
        card_title, speech_output, reprompt_text, should_end_session))
def get_art_response(intent):
    session_attributes = {}
    card_title = "article"
    
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('news')
    resource = table.scan()
    data = resource['Items']
    data2=""
    tech1= intent['slots']['number']['value']
    for x in data:
        if int(x['id'])==int(tech1):
            data2 =data2+ x['article']
    
    speech_output = tech1+", headlines article is   " + data2
    reprompt_text = "You never responded to the first test message. Sending another one."
    should_end_session = False
    return build_response(session_attributes, build_speechlet_response(
        card_title, speech_output, reprompt_text, should_end_session))
        
def get_summ_response(intent):
    session_attributes = {}
    card_title = "summary"
    print(2)
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('news')
    resource = table.scan()
    data = resource['Items']
    data1=""
    tech1= intent['slots']['number']['value']
    print(tech1)
    for x in data:
        
        if int(x['id'])==int(tech1):
            data1 =data1+ x['summary']
            print(data1)
    
    speech_output = tech1+", headlines summary is   " + data1
    reprompt_text = "You never responded to the first test message. Sending another one."
    should_end_session = False
    return build_response(session_attributes, build_speechlet_response(
        card_title, speech_output, reprompt_text, should_end_session))
        
def get_tech_response(intent):
    session_attributes = {}
    card_title = "technologyNews"
    
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('news')
    resource = table.scan()
    data = resource['Items']
    list1 = []
    tech= intent['slots']['type']['value']
    for x in data:
        if x['category']==tech:
            list1 = list1+[x['headline']]
    
    speech_output = tech+"news headlines are" +str(list1)
    reprompt_text = "You never responded to the first test message. Sending another one."
    should_end_session = False
    return build_response(session_attributes, build_speechlet_response(
        card_title, speech_output, reprompt_text, should_end_session))

def get_Student_Response(intent):
    session_attributes = {}
    card_title = "news"
    
    kid = intent['slots']['RollNo']['value']
    print('===>',kid)
    
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('StudentDetails')
    resource = table.scan()
    data = resource['Items']
    
    smarks = 0
    for x in data:
        if str(kid) == x['Student_Id'] :
            sname = x['Student_name']
            slocation = x['Location']
            smarks = x['Marks']
    if  smarks != 0:
        speech_output = str(kid)+" Student details are student name "+ sname + " Location " + slocation + " Marks " +smarks + " %"
        reprompt_text = str(kid)+" Student details are student name "+ sname + " Location " + slocation + " Marks " +smarks + " %"
    
        should_end_session = False
        return build_response(session_attributes, build_speechlet_response(card_title, speech_output, reprompt_text, should_end_session))
       
    else:
        speech_output = str(kid)+" student Doesn't Exist"
        reprompt_text = str(kid)+" student Doesn't Exist"
        should_end_session = False 
        return build_response(session_attributes, build_speechlet_response(card_title, speech_output, reprompt_text, should_end_session))

def handle_session_end_request():
    card_title = "Session Ended"
    speech_output = "Thank you for trying the Alexa Skills Kit sample. " \
                    "Have a nice day! "
    # Setting this to true ends the session and exits the skill.
    should_end_session = True
    return build_response({}, build_speechlet_response(
        card_title, speech_output, None, should_end_session))

# --------------- Events ------------------

def on_session_started(session_started_request, session):
    """ Called when the session starts.
        One possible use of this function is to initialize specific 
        variables from a previous state stored in an external database
    """
    # Add additional code here as needed
    pass

    

def on_launch(launch_request, session):
    """ Called when the user launches the skill without specifying what they
    want
    """
    # Dispatch to your skill's launch message
    return get_welcome_response()


def on_intent(intent_request, session):
    """ Called when the user specifies an intent for this skill """

    intent = intent_request['intent']
    intent_name = intent_request['intent']['name']

    # Dispatch to your skill's intent handlers
    if intent_name == "StudentDetails":
        return get_Student_Response(intent)
    
    elif intent_name == "latestNews":
        return get_news_response()
    
    elif intent_name == "summaryType":
        print(1)
        return get_summ_response(intent)
        
    elif intent_name == "articleType":
        return get_art_response(intent)
        
    elif intent_name == "technologyNews":
        return get_tech_response(intent)
    
    elif intent_name == "AMAZON.HelpIntent":
        return get_welcome_response()
    
    elif intent_name == "AMAZON.CancelIntent" or intent_name == "AMAZON.StopIntent":
        return handle_session_end_request()
    else:
        raise ValueError("Invalid intent")


def on_session_ended(session_ended_request, session):
    """ Called when the user ends the session.
    Is not called when the skill returns should_end_session=true
    """
    print("on_session_ended requestId=" + session_ended_request['requestId'] +
          ", sessionId=" + session['sessionId'])
    # add cleanup logic here


# --------------- Main handler ------------------

def lambda_handler(event, context):
    """ Route the incoming request based on type (LaunchRequest, IntentRequest,
    etc.) The JSON body of the request is provided in the event parameter.
    """
    print("Incoming request...")

    """
    Uncomment this if statement and populate with your skill's application ID to
    prevent someone else from configuring a skill that sends requests to this
    function.
    """
    # if (event['session']['application']['applicationId'] !=
    #         "amzn1.echo-sdk-ams.app.[unique-value-here]"):
    #     raise ValueError("Invalid Application ID")

    if event['session']['new']:
        on_session_started({'requestId': event['request']['requestId']},
                           event['session'])

    if event['request']['type'] == "LaunchRequest":
        return on_launch(event['request'], event['session'])
    elif event['request']['type'] == "IntentRequest":
        return on_intent(event['request'], event['session'])
    elif event['request']['type'] == "SessionEndedRequest":
        return on_session_ended(event['request'], event['session'])
