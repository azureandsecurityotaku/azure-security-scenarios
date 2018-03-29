﻿# Objective of the POC
Showcase a SQL injection attack and mitigation on a Web Application (Web App + SQL DB)

# Prerequisites
Access to Azure subscription to deploy following resources 
1. Application gateway (WAF enabled)
2. App Service (Web App)
3. SQL Database 
4. OMS (Monitoring)

# Deploy

1. Go to Edge Browser and Open [Azure Cloud Shell](https://shell.azure.com/)
1. Change directory to CloudDrive directory 

    `cd $Home\CloudDrive `

1. Clone Azure-Security-Scenarios repos to CloudDrive.

    `git clone https://github.com/AvyanConsultingCorp/azure-security-scenarios.git`

1. Change directory to azure-security-scenarios
 
    `cd .\azure-security-scenarios\`

1. Run below to get list of supported scenarios

    `.\deploy-azuresecurityscenarios.ps1 -Help`

1. If you are using Cloud Shell you can simply pass 2 parameters to run the deployment.

    `.\deploy-azuresecurityscenarios.ps1 -Scenario "sql-injection-attack-on-webapp" -Command Deploy  -Verbose`

1. However, if you are running on a local machine pass additional parameters to connect to subscription and run the deployment.

    `.\deploy-azuresecurityscenarios.ps1 -SubscriptionId <subscriptionId> -UserName <username> -Password <securePassword> -Scenario "sql-injection-attack-on-webapp" -Command Deploy   -Verbose`

# Attack
Attack on web app with
* Application gateway - WAF - Detection mode 
* SQL server and database with Threat Detection disabled. 

1. Go to Azure Portal --> Select Resource Groups services --> Select Resource Group - <prefix> "-sql-injection-attack-on-webapp"

2. Select Application Gateway with name 'appgw-detection-' as prefix.

    ![Setup Subscription](images/sql-inj-appgateway-det-location.png)

3. Application Gateway WAF enabled and Firewall in Detection mode as shown below.

    ![Setup Subscription](images/sql-inj-appgateway-waf-det.png)

4. On Overview Page --> Copy Frontend public IP address as
    ![Setup Subscription](images/sql-inj-appgateway-det-ip.png)

5. Open Internet Explorer with above details as shown below  
    ![Setup Subscription](images/sql-inj-webapp-contoso-landingpage.png)

4. Click on Patient link it will display list of details 

    ![Setup Subscription](images/sql-inj-webapp-contoso-patients-defpage.png)

4. Perform SQL Injection attack by copying " **'order by SSN--** " in search box and click on "Search". Application will show sorted data based on SSN.

    ![Setup Subscription](images/sql-inj-webapp-contoso-patients-attack-page.png)    
    'order by SSN--
    
# Detect
To detect the attack execute following query in Azure Log Analytics

AzureDiagnostics | where Message  contains "Injection" and action_s contains "detected"
    ![Setup Subscription](images/sql-inj-log-analytics-det.png) 
    
    
# Prevention

  * Update Web application firewall mode to Prevention for application gateway. This will take 5-10 mins. Hence we will connect the application using Application Gateway (WAF- Prevention mode) 

    ![Setup Subscription](images/sql-inj-appgateway-waf-prev.png)    
    
  

## Detection after Prevention

* Execute the step 7 to perform SQL Injection attack, Application Gateway will prevent access

    ![Setup Subscription](images/403-forbidden-access-denied.png)  

 
* To detect the prevention of attack execute following query in Azure Log Analytics


    AzureDiagnostics | where Message  contains "injection" and action_s contains "blocked"
    
    ![Setup Subscription](images/sql-inj-log-analytics-blocked.png)  


You will notice events related to Quarantined items. It might take few minutes for OMS to pull logs from virtual machine, so if you don't get any search results, please try again after sometime.

References -

 

