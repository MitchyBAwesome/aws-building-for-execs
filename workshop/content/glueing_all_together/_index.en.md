+++
title = "Gluing it all Together"
chapter = true
weight = 6
+++

So we’ve analysed our data, built a Machine Learning model, and built the basic constructs of our Chatbot. But how does this all glue together? We need to pass the name collected from our Chatbot to our machine learning model to provide us with a personalised recommendation. We then need to pass that information to the drones for delivery. We need some glue to put all this business logic together.

Traditionally, applications would be developed and run on servers in a data centre to facilitate these processes and communication. With AWS Lambda, we’re able to take a more event driven approach, where we execute code on demand that represents business logic, and only pay a fraction of a cent for the seconds that code executes. Let’s create our Lambda function that ties all this together.

