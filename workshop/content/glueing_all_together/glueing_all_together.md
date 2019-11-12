+++
title = "Step by Step"
weight = 2
+++

1. Navigate to the AWS Lambda service in the console

2. Click Create function and specify the following:

    - Function name: orderLambda-team#

    - Runtime: Python 3.7 – This is the language we’ll be writing our business logic in

    - Execution role: Use an existing role: LambdaOrdersRole – This provides our Lambda function programmatic access to our machine learning model and IoT drones for delivery

3. Select **Create function**.

4. We can now specify our business logic in the form of Python code! Copy and paste the below code into the Function code:

        import boto3
        import json

        ##############################
        # Look up users name in DynamoDB to retrieve UserID and Mobile Number
        ##############################
        def ddb_get_user_data(name):

            dynamodb = boto3.resource('dynamodb')
            table = dynamodb.Table('user')
            
            response = table.get_item(Key={'name': name})
            
            item = response['Item']
            return item

        ##############################
        # IoT call to drones with product colour and mobile number
        ##############################
        def iot_Publish(iot_Payload):
            iot_client = boto3.client('iot-data')

            iot_response = iot_client.publish(
                topic='droneIn',
                payload = json.dumps(iot_Payload)
            )
            return iot_response

        ##############################
        # Builders
        ##############################

        def build_PlainSpeech(body):
            speech = {}
            speech['type'] = 'PlainText'
            speech['text'] = body
            return speech

        def build_response(message, session_attributes={}):
            response = {}
            response['version'] = '1.0'
            response['sessionAttributes'] = session_attributes
            response['response'] = message
            return response

        def build_SimpleCard(title, body):
            card = {}
            card['type'] = 'Simple'
            card['title'] = title
            card['content'] = body
            return card

        ##############################
        # Responses
        ##############################

        def statement(title, body):
            speechlet = {}
            speechlet['outputSpeech'] = build_PlainSpeech(body)
            speechlet['card'] = build_SimpleCard(title, body)
            speechlet['shouldEndSession'] = False
            return build_response(speechlet)

        def endStatement(title, body):
            speechlet = {}
            speechlet['outputSpeech'] = build_PlainSpeech(body)
            speechlet['card'] = build_SimpleCard(title, body)
            speechlet['shouldEndSession'] = True
            return build_response(speechlet)

        def continue_dialog():
            message = {}
            message['shouldEndSession'] = False
            message['directives'] = [{'type': 'Dialog.Delegate'}]
            return build_response(message)

        ##############################
        # Custom Intents
        ##############################

        def surprise_me_intent(event, context):
            print(event)
            dialog_state = event['request']['dialogState']

            if dialog_state in ("STARTED", "IN_PROGRESS"):
                return continue_dialog()

            elif dialog_state == "COMPLETED":
                
                #Step 1: Extract First Name from Alexa slot
                first_Name = (event['request']['intent']['slots']['name']['value']).lower()
                print(first_Name)
                
                #Step 2: Use the First Name to get the users Mobile number and UserID from DynamoDB
                get_ddb_info = ddb_get_user_data(first_Name)
                ddb_user_id = get_ddb_info['userid']
                print(ddb_user_id)
                mobile = get_ddb_info['mobile']
                
                #Step 3: Call the Personalize API. 
                
                #Offer Option1: Most Ordered
                offer = ['24852','44632','5077','9076','25890','31506','27156','8021']

                personalizeRt = boto3.client('personalize-runtime')
                response = personalizeRt.get_personalized_ranking(
                    campaignArn = "arn:aws:personalize:us-east-1:649537638751:campaign/PersonalizedRankingCampaign",
                    userId = ddb_user_id,
                    inputList = offer) #banana, strawberries, organic milk
                topProduct = response['personalizedRanking'][0]['itemId']
                print(topProduct)
                
                
                #Step 4: Convert Product into Colour for Drones LED
                droneColours = ["blue","red","green","yellow","purple","white","orange","aqua"]
                moduloProduct = int(offer.index(topProduct))
                print(moduloProduct)
                productColour = droneColours[moduloProduct]
                print(productColour)
            
                #Step 5: Publish IoT message to the topic the Drones are subscribed to
                #iot_Payload = recommendation[][][] #analyse response...
                iot_Payload = {
                        "colour": productColour,
                        "mobile": mobile,
                        "product": topProduct
                }
                iot_Publish(iot_Payload)
                
                #Step 5: Return a message back to Alexa
                return_statement = "Thanks " + first_Name + " Your "+productColour+" surprise is on its way, I will text you for your feedback"

                return endStatement("surprise_me_intent", return_statement)

            else:
                return statement("surprise_me_intent", "No dialog")

        ##############################
        # Required Intents
        ##############################

        def cancel_intent():
            return statement("CancelIntent", "You want to cancel")  #don't use CancelIntent as title it causes code reference error during certification 

        def help_intent():
            return statement("CancelIntent", "You want help")       #same here don't use CancelIntent

        def stop_intent():
            return statement("StopIntent", "You want to stop")      #here also don't use StopIntent

        ##############################
        # On Launch
        ##############################

        def on_launch(event, context):
            return statement("title", "Welcome to Daily Deal, you can ask me to surprise you")

        ##############################
        # Routing
        ##############################

        def intent_router(event, context):
            intent = event['request']['intent']['name']

            # Custom Intents
            
            if intent == "SurpriseMeIntent":
                return surprise_me_intent(event, context)

            # Required Intents

            if intent == "AMAZON.CancelIntent":
                return cancel_intent()

            if intent == "AMAZON.HelpIntent":
                return help_intent()

            if intent == "AMAZON.StopIntent":
                return stop_intent()

        ##############################
        # Program Entry
        ##############################

        def lambda_handler(event, context):
            if event['request']['type'] == "LaunchRequest":
                return on_launch(event, context)

            elif event['request']['type'] == "IntentRequest":
                return intent_router(event, context)

    There’s a fair bit of code here, that’s doing the following:

    - Collecting the user’s name from Alexa and finding their mobile number in our database

    - Sending their name as well as our top 8 products to our machine learning model, to determine which of the 8 products they would be most interested in

    - Sending that information to the drones for delivery of the recommended product to the customer

    {{% expand "Expand for an Added Challenge!" %}}
*See if you can spot the location in the code where we feed in the top 8 products. If you had the chance in Activity 1 to determine the least ordered products, replace the list of items with the bottom 8 products*
    {{% /expand %}}

5. The final thing left to do is add a required Python library, and ensure the Alexa skill invokes the Lambda function once it’s collected the user’s details. To do this, expand the Designer window, click on Layers and add the Boto3layer. Select version 3 and click Add. Click Add trigger, the select Alexa Skills Kit. Select Disable for the Skill ID verification and select Add. Click Save. Copy the ARN which is a resource identifier for the Lambda function in the top right hand corner.

6. Go back to your Alexa developer console and select Endpoint, then choose AWS Lambda ARN, and paste the ARN copied in the previous step, in Default Region. Click Save Endpoints. Select the Invocation tab, then select Build Model.
