# What is it?

This repository will provision an environment that may be used as a Hackathon to build an end to end scenario that does the following:

*	Query a Contact List API for a customer
*	Query our legacy on-prem Ticket system to get the customer’s last feedback
*	Perform sentiment analysis on this feedback
*	Generate a digital discount coupon if they were dissatisfied
*	Mail them the coupon

# What does it showcase?

This solution brings together Infrastructure as a Service (IaaS), Platform as a Service (PaaS), Software as a Service (saaS) and Serverless components on Microsoft Azure to build a realistic end to end scenario common in retail. Furthermore, the democratization of AI is tied in nicely by incorporating Cognitive Services to perform text analysis and determine the sentiment of a customer’s feedback.

The solution aims to show the following:

*	How legacy lift and shift applications on IaaS can be incorporated into modern solutions to quickly derive value from higher value services in the cloud.
*	How existing investments can be modernized without having to rebuild everything to drive customer value
*	The ease with which On-premise, public and private components may be brought together to build workloads that bring business value
*	The meshing of IaaS, PaaS, SaaS, Serverless and AI with tools that are accessible to non-developers
*	OSS workloads running on Azure

# Technology used

The following technology components are used in this solution:

*	Swagger enabled Node.js APIs running on Azure App Services (PaaS)
*	Ubuntu with a custom extension template to rapidly provision and deploy a custom image with an running solution (IaaS)
*	Azure networking to isolate legacy workloads (IaaS)
*	API Management to govern APIs and to bridge publicly accessible APIs with isolated APIs (SaaS) (IaaS)
*	Azure functions to run dynamic ‘pay-as-you-go’ compute (Serverless) [Thanks Christof Claasens and Katrien de Graeve](https://github.com/xstof/Quiz) 
*	Azure logic apps to provide serverless integration that is accessible to non-developers (Serveless)
*	Azure Resource Manager templates to automate the provisioning and inflation of a full environment
*	The Azure CLI 2.0

# Solution flow

![alt text](https://github.com/shanepeckham/MiniCADHackathon/blob/master/Typology.jpg)

# The Hackathon component

This solution will install and configure all of the components required to build the end to end Loyalty scenario. The Hackathon attendees just need to wire everything together in a Logic App. 

# Preparing for the solution

For this Hackathon you will require:
* A cognitive services trial account key, get it here - https://www.microsoft.com/cognitive-services/en-us/sign-up
* A Gmail account for sending emails, get it here - https://accounts.google.com/SignUp?service=mail&continue=http%3A%2F%2Fmail.google.com%2Fmail%2Fe-11-14a952576b5f2ffe90ec0dd9823744e0-46a962fbb7947b68059374ddde7f29b5490a6b4d
* Install Postman, get it here - https://www.getpostman.com

# How to install the solution

1. Provisioning the components: Select Deploy to Azure to deploy to your Azure instance that you are currently logged in to.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-oms-extension-ubuntu-vm%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>
<a href="http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-oms-extension-ubuntu-vm%2Fazuredeploy.json" target="_blank">
    <img src="http://armviz.io/visualizebutton.png"/>
</a>

The only parameter you need to change is the Deployment Name - give it any name as it will be used to generate a hash to ensure your site names are unique, see the image below:

![alt text](https://github.com/shanepeckham/CADHackathon_Loyalty/blob/master/Images/DeploymentName.jpg)

This will take roughly 30 minutes as this will provision:

* Two VNETs
* An Ubuntu VM and place in inside the VNET isolated with NSGs
* An API Management instance (Developer Tier) and place it inside a subnet within the VNET
* An App Service API app (Contact List) and deploy a node.js Swagger enabled API to it
* An App Service serverless function with dynamic scaling and pricing
* Storage accounts to house the VM VHD, the Function logging and the App Service API logging

2. Checking the Contact List API App: Once deployment is complete, navigate to your App Service API App, its default name will be CADAPIMasterSite[hash] and click on the URI in the overview blade, see below:

![alt text](https://github.com/shanepeckham/CADHackathon_Loyalty/blob/master/Images/API%20URI.jpg)

This will navigate you to a URL that will display the following message if it was provisioned correctly:
```
Cannot GET / 
```
Now type ``` /docs ``` after the azurewebsites.net part of the url and you should see the Swagger editor open with a test harness to test the API:

![alt text](https://github.com/shanepeckham/CADHackathon_Loyalty/blob/master/Images/SwaggerHarness.jpg)

You should now be able to test a few methods of the API to check if how it works. It will query 3 contacts and methods exist to query all, query by Id, query associated Case Number by Id and query contact Email by Id.

Now copy the URL without the ``` /docs ``` component and paste is somewhere for retrieval later on.

3. Now we will import the Contact List API into the API Management solution

Navigate to API Management component provisioned within Azure, its name will be generated by default with the following format cadapim[hash].

Click on the overview blade and select 'Publisher Portal' (note we will use the old portal while the new portal is still in preview), see below:

![alt text](https://github.com/shanepeckham/CADHackathon_Loyalty/blob/master/Images/APIMPublisherPortal.jpg)

This will navigate you to the Publisher portal, select Import API, see below:

![alt text](https://github.com/shanepeckham/CADHackathon_Loyalty/blob/master/Images/ImportAPI.jpg)

Now enter the following values:

* Select "From Url"
* Paste your API URL from step 2 and add ``` /swagger ``` on the end e.g. http://cadapimastersitervhyzok7zv4gw.azurewebsites.net/swagger
* Select "Swagger" as the specification format
* Select New API
* Type "Contacts" into the Web API URL Suffix field
* Click Products and select "Unlimited"
* Click Save

See below:

![alt text](https://github.com/shanepeckham/CADHackathon_Loyalty/blob/master/Images/ImportAPIDetails.jpg)

You will now have imported an API that will now be accessible from the API Gateway. You can test it by clicking on Developer Portal.







# The Logic App solution

Create a HTTP Request Step, click save - you will receive an endpoint upon save. 

![alt text](https://github.com/shanepeckham/CADHackathon_Loyalty/blob/master/Images/HTTP%20Request.jpg)

You can now invoke your logic app with Postman - add the URL and select POST. Ensure you have set the Header "Content-Type" with value "application/json". Select body, select "raw" and enter the follow value for your body content:
```
{
  "APIMKey": "[Your APIM Key goes here]",
  "id": 1
}
```
![alt text](https://github.com/shanepeckham/CADHackathon_Loyalty/blob/master/Images/PostManHeaders.jpg)
![alt text](https://github.com/shanepeckham/CADHackathon_Loyalty/blob/master/Images/PostManBody.jpg)

Now add a step to include an API Management API - select your API "Contact List API"

You will need to navigate to the code view to be able to select the json fields that will be posted as part of the body. Your code view should look like this:
```
"Query_Contacts_by_Id": {
                "inputs": {
                    "api": {
                        "id": "/subscriptions/[subscription id]/resourceGroups/MiniCAD/providers/Microsoft.ApiManagement/service/minicad123api/apis/[api id]"
                    },
                    "method": "get",
                    "pathTemplate": {
                        "parameters": {
                            "id": "@{encodeURIComponent(int(triggerBody()['id']))}"
                        },
                        "template": "/Contacts/contacts/{id}"
                    },
                    "subscriptionKey": "@{encodeURIComponent(triggerBody()['APIMKey'])}"
                },
                "runAfter": {},
                "type": "ApiManagement"
            }

```
Now add a For Each loop as we want to iterate through the resultset, so select the Body as the output from your previous request.

[!alt text](https://github.com/shanepeckham/CADHackathon_Loyalty/blob/master/Images/ForEach.jpg)

Now you want to add a step to query the Legacy Ticket API which is inside the isolated network, add an API Management API step and once again query the Id, which in this case is the casenum output from the previous step and add the API Management subscription key.

![alt text](https://github.com/shanepeckham/CADHackathon_Loyalty/blob/master/Images/QueryCasesFeedback.jpg)

Here is what your code view should look like:
```
"Query_Cases_Feedback": {
                        "inputs": {
                            "api": {
                                "id": "/subscriptions/de019774-dddc-40a9-9515-51f9df268c95/resourceGroups/[Your Resource Group]/providers/Microsoft.ApiManagement/service/minicad123api/apis/58af3fded9e0430e784c6b9d"
                            },
                            "method": "get",
                            "pathTemplate": {
                                "parameters": {
                                    "id": "@{encodeURIComponent(item()?['caseNum'])}"
                                },
                                "template": "/LegacyAPI/contacts/{id}"
                            },
                            "subscriptionKey": "@{encodeURIComponent(triggerBody()['APIMKey'])}"
                        },
                        "runAfter": {},
                        "type": "ApiManagement"
                    },
```

Now we want to add our Cognitive Services 'Detect Sentiment' step so that we can analyse the sentiment of the Ticket Feedback, your step should look like this:

![alt text](https://github.com/shanepeckham/CADHackathon_Loyalty/blob/master/Images/DetectSentiment.jpg)

Your code view should look like this:
```
"Detect_Sentiment": {
                        "inputs": {
                            "body": {
                                "text": "@body('Query_Cases_Feedback')['last_feedback']"
                            },
                            "host": {
                                "api": {
                                    "runtimeUrl": "https://logic-apis-northeurope.azure-apim.net/apim/[YourCognitiveServicesConnectionName]"
                                },
                                "connection": {
                                    "name": "@parameters('$connections')[YourCognitiveServicesConnectionName]['connectionId']"
                                }
                            },
                            "method": "post",
                            "path": "/sentiment"
                        },
                        "runAfter": {
                            "Query_Cases_Feedback": [
                                "Succeeded"
                            ]
                        },
                        "type": "ApiConnection"
                    },
```

Now we want to add a condition to check the sentiment, if the probability outcome is less than 0.5, then it negative sentiment and therefore qualifies for our discount coupon.

Your condition should look like this:

![alt text](https://github.com/shanepeckham/CADHackathon_Loyalty/blob/master/Images/Condition.jpg)

With the following in code view:
```
"expression": "@less(body('Detect_Sentiment')?['score'], 0.5)",
                        "runAfter": {
                            "Detect_Sentiment": [
                                "Succeeded"
                            ]
                        },
                        "type": "If"

```

Now you can call the GenerateCoupon function if the condition is met, pass in the name of the user that you want to generate the digital coupon for:

![alt text](https://github.com/shanepeckham/CADHackathon_Loyalty/blob/master/Images/IfCondition.jpg)

With the following in the code view:
```
"GenerateCoupon": {
                                "inputs": {
                                    "body": {
                                        "name": "@item()?['name']"
                                    },
                                    "function": {
                                        "id": "/subscriptions/de019774-dddc-40a9-9515-51f9df268c95/resourceGroups/[Your Resource Group]/providers/Microsoft.Web/sites/MiniCADFunctionApp4pb56ec3fmsgg/functions/GenerateCoupon"
                                    },
                                    "method": "POST"
                                },
                                "runAfter": {},
                                "type": "Function"
                            }
```
Now we want to send an email to every receipient to inform them that they can download a digital coupon which we have generated for them. Your email step should look like this:

![alt text](https://github.com/shanepeckham/CADHackathon_Loyalty/blob/master/Images/Email.jpg)

With the following code view:
```
"Send_email": {
                        "inputs": {
                            "body": {
                                "Body": "Please get your coupon here: @{body('GenerateCoupon')}",
                                "Subject": "Your Coupon has been generated",
                                "To": "@{item()?['email']}"
                            },
                            "host": {
                                "api": {
                                    "runtimeUrl": "https://logic-apis-northeurope.azure-apim.net/apim/gmail"
                                },
                                "connection": {
                                    "name": "@parameters('$connections')['gmail']['connectionId']"
                                }
                            },
                            "method": "post",
                            "path": "/Mail"
                        },
                        "runAfter": {
                            "Condition": [
                                "Succeeded"
                            ]
                        },
                        "type": "ApiConnection"
                    }
```

The full code solution view looks like this:
```
{
    "$connections": {
        "value": {
            "cognitiveservicestextanalytics": {
                "connectionId": "/subscriptions/de019774-dddc-40a9-9515-51f9df268c95/resourceGroups/MiniCAD/providers/Microsoft.Web/connections/cognitiveservicestextanalytics",
                "connectionName": "cognitiveservicestextanalytics",
                "id": "/subscriptions/de019774-dddc-40a9-9515-51f9df268c95/providers/Microsoft.Web/locations/northeurope/managedApis/cognitiveservicestextanalytics"
            },
            "gmail": {
                "connectionId": "/subscriptions/de019774-dddc-40a9-9515-51f9df268c95/resourceGroups/MiniCAD/providers/Microsoft.Web/connections/gmail",
                "connectionName": "gmail",
                "id": "/subscriptions/de019774-dddc-40a9-9515-51f9df268c95/providers/Microsoft.Web/locations/northeurope/managedApis/gmail"
            }
        }
    },
    "definition": {
        "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
        "actions": {
            "For_each": {
                "actions": {
                    "Condition": {
                        "actions": {
                            "GenerateCoupon": {
                                "inputs": {
                                    "body": {
                                        "name": "@item()?['name']"
                                    },
                                    "function": {
                                        "id": "/subscriptions/de019774-dddc-40a9-9515-51f9df268c95/resourceGroups/MiniCADFunction/providers/Microsoft.Web/sites/MiniCADFunctionApp4pb56ec3fmsgg/functions/GenerateCoupon"
                                    },
                                    "method": "POST"
                                },
                                "runAfter": {},
                                "type": "Function"
                            }
                        },
                        "expression": "@less(body('Detect_Sentiment')?['score'], 0.5)",
                        "runAfter": {
                            "Detect_Sentiment": [
                                "Succeeded"
                            ]
                        },
                        "type": "If"
                    },
                    "Detect_Sentiment": {
                        "inputs": {
                            "body": {
                                "text": "@body('Query_Cases_Feedback')['last_feedback']"
                            },
                            "host": {
                                "api": {
                                    "runtimeUrl": "https://logic-apis-northeurope.azure-apim.net/apim/cognitiveservicestextanalytics"
                                },
                                "connection": {
                                    "name": "@parameters('$connections')['cognitiveservicestextanalytics']['connectionId']"
                                }
                            },
                            "method": "post",
                            "path": "/sentiment"
                        },
                        "runAfter": {
                            "Query_Cases_Feedback": [
                                "Succeeded"
                            ]
                        },
                        "type": "ApiConnection"
                    },
                    "Query_Cases_Feedback": {
                        "inputs": {
                            "api": {
                                "id": "/subscriptions/de019774-dddc-40a9-9515-51f9df268c95/resourceGroups/MiniCAD/providers/Microsoft.ApiManagement/service/minicad123api/apis/58af3fded9e0430e784c6b9d"
                            },
                            "method": "get",
                            "pathTemplate": {
                                "parameters": {
                                    "id": "@{encodeURIComponent(item()?['caseNum'])}"
                                },
                                "template": "/LegacyAPI/contacts/{id}"
                            },
                            "subscriptionKey": "@{encodeURIComponent(triggerBody()['APIMKey'])}"
                        },
                        "runAfter": {},
                        "type": "ApiManagement"
                    },
                    "Send_email": {
                        "inputs": {
                            "body": {
                                "Body": "Please get your coupon here: @{body('GenerateCoupon')}",
                                "Subject": "Your Coupon has been generated",
                                "To": "@{item()?['email']}"
                            },
                            "host": {
                                "api": {
                                    "runtimeUrl": "https://logic-apis-northeurope.azure-apim.net/apim/gmail"
                                },
                                "connection": {
                                    "name": "@parameters('$connections')['gmail']['connectionId']"
                                }
                            },
                            "method": "post",
                            "path": "/Mail"
                        },
                        "runAfter": {
                            "Condition": [
                                "Succeeded"
                            ]
                        },
                        "type": "ApiConnection"
                    }
                },
                "foreach": "@body('Query_Contacts_by_Id')",
                "runAfter": {
                    "Query_Contacts_by_Id": [
                        "Succeeded"
                    ]
                },
                "type": "Foreach"
            },
            "Query_Contacts_by_Id": {
                "inputs": {
                    "api": {
                        "id": "/subscriptions/de019774-dddc-40a9-9515-51f9df268c95/resourceGroups/MiniCAD/providers/Microsoft.ApiManagement/service/minicad123api/apis/58af405bd9e0430e784c6b9f"
                    },
                    "method": "get",
                    "pathTemplate": {
                        "parameters": {
                            "id": "@{encodeURIComponent(int(triggerBody()['id']))}"
                        },
                        "template": "/Contacts/contacts/{id}"
                    },
                    "subscriptionKey": "@{encodeURIComponent(triggerBody()['APIMKey'])}"
                },
                "runAfter": {},
                "type": "ApiManagement"
            }
        },
        "contentVersion": "1.0.0.0",
        "outputs": {},
        "parameters": {
            "$connections": {
                "defaultValue": {},
                "type": "Object"
            }
        },
        "triggers": {
            "manual": {
                "inputs": {
                    "schema": {}
                },
                "kind": "Http",
                "type": "Request"
            }
        }
    }
}

```

