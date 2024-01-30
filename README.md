# Chat with your SAP Data (or any data)

Integrate a ChatGPT-like experience directly into Microsoft Teams, enabling users to ask any question about their SAP data. With all data and interactions securely contained within your private Power Platform environment and Azure Subscription, engage in confidential conversations using your business data, ensuring privacy and security at every step.

Get quickly questions solved and even the result presented as a chart.
Examples: 
- What is my current stock of a specific product from a specific vendor in my factories for a certain product?
- How are sales progressing over the last 6 weeks for a specific product, model or service?
- I want to verify data quality: Please show me sold products that have incomplete owners data?
- How many empty seats did we have for for economy, business and first class in September 2023?
- A machine operator is having problems and needs help. The bot knows all the extensive documentation and assists. If he still cannot solve the problem, the bot can create the repair order in SAP.
- Ask for real time financial status of the company from within teams, per region, country, product, etc.
- and much more ...

# Architecture

![High Level Process](images/high-level-process.jpg)


# Chat Examples

![Greetings](images/greetings1.jpg)
![Greetings](images/greetings2.jpg)
![Greetings](images/greetings3.jpg)
![Greetings](images/greetings4.jpg)
![Greetings](images/greetings5.jpg)
![Greetings](images/greetings6.jpg)

# Prerequesites

1. Power Platform 
    - Power Virtual Agent (PVA), to create the bot https://web.powerva.microsoft.com/
    - Power Automate Flows: https://make.powerautomate.com/
    - Recommended for productive use is separated DEV and PROD environments
    - Trial license: https://learn.microsoft.com/en-us/power-apps/maker/signup-for-powerapps
1. Azure Subscription
    - Azure Trial: https://azure.microsoft.com/en-us/free
1. Azure OpenAI Service 
    - Deployment Guide: https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/create-resource?pivots=web-portal
    - Request Access to OpenAI as described. Request only Text & code models.
1. Function App (Premium) 
    1. To run SQL code against the HANA DB
    1. To convert the SQL Output in CSV into other formats like JSON, MARKDOWN or HTML
1. Generate the SAP Flight Demo Data or try directly on SAP business data
    - https://help.sap.com/docs/SUPPORT_CONTENT/abap/3353524003.html
1. A DB user with read access to the required tables


# Setup the Solution

## Azure OpenAI Service 

- Follow this documentation: https://learn.microsoft.com/en-us/azure/ai-services/openai/how-to/create-resource?pivots=web-portal <br> 
- Once the OpenAI Service is deployed go to Models and deploy the gpt-35-turbo or gpt-4 model. <br>
- Retrieve your endpoint URL and Key in "Chat Playground" -> "View code".

## Azure function to execute SQL on the DB

To connect to the HANA DB we start hdbsql from an Azure function. <br> 
Implement this Azure Function to call hdbsql and execute the SQL code in the HANA DB: [Azure Function to call hdbsql](https://github.com/mimergel/saphanasqlfunction) <br> 

I haven't had the opportunity to explore alternative database vendors. I'm sure this concept can be applied to any SAP database systems, such as Oracle, DB2, MS SQL, MAXDB, and Sybase. <br> 

Similarly, I'm sure that it can also be effectively utilized with Non-SAP databases. <br> 


## Azure function to convert SQL Output in CSV into JSON and MARKDOWN

Implement this Azure Function for the conversion: [Azure Function to convert CSV](https://github.com/mimergel/sqloutputcsv_to_json_html_markdown) <br> 


## Setup the Power Automate Flow (Flow)

- Login to: https://make.powerautomate.com/.
- Go to Solutions and import the Power Automate Flow.
- Here you can download the flow as zip file: [Download the Power Automate Flow](https://github.com/mimergel/chat_with_your_sap_data/blob/main/flow/FlightDemoBotV2_1_0_0_6.zip)

    ![Import the Flow](images/import-flow.jpg) <br>

<br>

**Flow Screenshots**

![Chat with my SAP Data Flow Part 1-2](images/flow1-2.jpg) <br>

![Chat with my SAP Data Flow Part 3](images/flow3.jpg) <br>

![Chat with my SAP Data Flow Part 4](images/flow4.jpg) <br>


## Adapt the HTTP Connectors in the Flow

- Get your Azure OpenAI URL and API-Key from the Azure Open AI Service Chat playground.

    ![Get URI and KEY](images/get-uri-key.jpg) <br>

- URI example: https://[your endpoint].openai.azure.com/openai/deployments/[deployed model name]]/chat/completions?api-version=2023-07-01-preview

- Edit the Flow
    - Enter the URI and KEY in the respective fields of the HTTP Connectors for Azure OpenAI. There are 2 HTTP Connectors using Azure OpenAI: 
        - HTTP Azure OpenAI
        - HTTP create chart.js data
    - Adapt: 
        - URIs &
        - api-keys

    ![Change HTTP Connector](images/http-openai.jpg) <br>

For now you can enter URI and key in clear text. For later production use it's recommended to configure these as secrets and put references to the secrets like seen in the screen shot. <br>


- Change in the same way the following 3 HTTP Connectors:
    - HTTP hdbsql
    - HTTP csv to markdown
    - HTTP csv to json

    ![Change HTTP Connector](images/http-csv2json.jpg) <br>

<br>

Note that the key is here part of queries section and is called "code".

## Setup the Bot

- Create a new bot in https://web.powerva.microsoft.com/ <br>

    ![New Bot](images/pva-new-bot.jpg) <br>
     
     <br>

    ![New Bot 2. Step](images/pva-new-bot2.jpg) <br> 

- In the fallback topic paste the code from the provided fallback.yaml file.

- Get the code for the fallback topic: [fallback.yaml](pva/fallback.yaml) <br>

- Open the fallback topic in your bot

    ![Open Fallback Topic](images/pva-fallback1.jpg) <br> 

- Open the code editor and paste the code.

    ![Code Editor](images/pva-fallback2.jpg) <br> 

 <br> 


## Test & Publish the Bot

- Test the Bot in: https://web.powerva.microsoft.com

    ![Test 1](images/test-pva1.jpg)

    | ![Test 2](images/test-pva2.jpg) | ![Test 3](images/test-pva3.jpg) |

    ![Test 4](images/test-pva4.jpg)


- When everything works fine publish the bot under `Settings` -> `Channels` -> `Microsoft Teams`

    ![Publish](images/publish.jpg)

- Activate and Open the Bot in Teams

    ![Publish](images/open-bot-in-teams.jpg) <br>

- Open the Bot in Teams and test: <br>

    ![Publish](images/test-teams1.jpg) 

    ![Publish](images/test-teams2.jpg) 

    ![Publish](images/test-teams3.jpg) 



## Troubleshooting

- In case of errors most likely the flow failed.
    Example here with a wrong API key: 

    ![Troubleshooting 1](images/troubleshooting1.jpg) <br>

    - Check here: https://make.powerautomate.com/ in <br>
        - "My Flows"   <br>
        - "28-days run history"  <br>

    ![Troubleshooting 2](images/troubleshooting2.jpg) <br>

- The logs of the last flow run will provide helpful error messages.  <br>
- Got to `Edit` and fix the issue:  

    ![Troubleshooting 1](images/troubleshooting3.jpg) <br>


# Adapt to other business scenarios

Steps to adapt the solution to another business scenario:

- Identify the relevant SAP tables with the data that you want to ask questions about.
- Grant the HANA DB user read access to these tables.
- Adapt the Initialize variable "System Prompt" step in the flow:

    ![Adapt Flow](images/adapt-flow.jpg) <br>

- Enter the table(s) definition for your scenario

## Contributing

Contributions to enhance the capabilities are welcome.

## License

This project is licensed under the MIT License.

## Disclaimer

THE CONTENT OF THIS REPOSITORY IS PROVIDED AS IS WITHOUT WARRANTY OF ANY KIND, EITHER EXPRESS OR IMPLIED, INCLUDING ANY IMPLIED WARRANTIES OF FITNESS FOR A PARTICULAR PURPOSE, MERCHANTABILITY, OR NON-INFRINGEMENT.