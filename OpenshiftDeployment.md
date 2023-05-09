# Openshift Deployment

   We have to deploy 3 components for deployment of Eshop on web in Openshift.

      a) SQL server
      b) Public API
      c) Web App

## Prerequisite

## 1) SQL Server
   
   ### Deploying the sql server

   ### Execute following commands for deploying SQL server. 
   
   All these commands are available under `openshift/commands.txt` file

      ``` 
        oc create secret generic mssql --from-literal=SA_PASSWORD="@someThingComplicated1234" -n dotnet
        oc create serviceaccount sqlserver-sa -n dotnet
        oc adm policy add-scc-to-user anyuid -z sqlserver-sa -n dotnet
        oc create -f openshift/sql-server/pvc.yaml
        oc create -f openshift/sql-server/Deployment.yaml
        oc expose deployment/sqlserver -n dotnet
      ```  

 ## S2i Demo

 ### Public API.


    ```
        oc new-app dotnet:7.0-ubi8~https://github.com/arunhari82/dotnet-eShopOnWeb.git --name public-api --build-env DOTNET_STARTUP_PROJECT=src/PublicApi/PublicApi.csproj -e ASPNETCORE_URLS='http://+:5200' --strategy=source
        oc set volume dc/public-api --add --name appsettings-vol --mount-path /opt/app-root/app/appsettings.json --configmap-name=appsettings-cm --sub-path=appsettings.json

    ```

        
    