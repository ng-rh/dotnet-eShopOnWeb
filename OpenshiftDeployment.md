# Openshift Deployment

## prerequisite

## 1) SQL Server
   
   ### Deploying the sql server

   ### Execute following commands for deploying SQL server. All these commands are available under `openshift/commands.txt` file

      ``` 
        oc create secret generic mssql --from-literal=SA_PASSWORD="@someThingComplicated1234" -n dotnet
        oc create serviceaccount sqlserver-sa -n dotnet
        oc adm policy add-scc-to-user anyuid -z sqlserver-sa -n dotnet
        oc create -f openshift/sql-server/pvc.yaml
        oc create -f openshift/sql-server/Deployment.yaml
        oc expose deployment/sqlserver -n dotnet
      ```  
    