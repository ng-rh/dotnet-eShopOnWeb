# Openshift Deployment

   We have to deploy 3 components for deployment of Eshop on web in Openshift.

      a) SQL server
      b) Public API
      c) Web App


   1) Clone this repo and change directory to repo cloned directory.

        ``` 
            git clone  https://github.com/arunhari82/dotnet-eShopOnWeb.git
            cd <<Repo directory Name>>  
        ```    

   2) Create a namespace for deploying the eshop application

            oc new-project <<namespace>>

   3) set the namespace as env variable

            export namespace=<<namespace>>        

## Prerequisite

## 1) SQL Server
   
   ### Deploying the sql server

   ### Execute following commands for deploying SQL server. 
   
   All these commands are available under `openshift/commands.txt` file

   Create secret for database credtionials

        oc create secret generic mssql --from-literal=SA_PASSWORD="@someThingComplicated1234" -n $namespace

   Create Service account for database server and provide privileges     
   
        oc create serviceaccount sqlserver-sa -n $namespace
        oc adm policy add-scc-to-user anyuid -z sqlserver-sa -n $namespace

   Create PVC and Deployment to deploy SQL server 

        oc create -f openshift/sql-server/pvc.yaml -n $namespace
        oc create -f openshift/sql-server/Deployment.yaml -n $namespace

   Create Service for SQL server

        oc expose deployment/sqlserver -n $namespace

                   --- or ---

        oc apply -f openshift/sql-server/Service.yaml   


 ## S2i Demo

 ## Public API.

 ### Create Configmap

        oc create -f openshift/publicApi/configmap.yaml

          ------- or --------

        oc create cm  appsettings-cm  --from-file=appsettings.json=openshift/publicApi/assets/appsettings.json

### import image as image stream
       
        oc import-image dotnet:7.0-ubi8 --from=registry.redhat.io/rhel8/dotnet-70:7.0-12 --confirm

### Deploy new app

Since we are building this project from repo home directory we need to specify s2i process to build specific project this can be achieved via build-env variable `DOTNET_STARTUP_PROJECT` . This variable should point to `.csproj` extension file. Please refer the following [documentation](https://github.com/redhat-developer/s2i-dotnetcore/tree/main/7.0/build#environment-variables) for more references.

        oc new-app dotnet:7.0-ubi8~https://github.com/arunhari82/dotnet-eShopOnWeb.git --name public-api --build-env DOTNET_STARTUP_PROJECT=src/PublicApi/PublicApi.csproj -e ASPNETCORE_URLS='http://+:8080' --strategy=source

### Wait for the build to complete by watching logs.

       oc logs -f bc/public-api


        oc set volume dc/public-api --add --name appsettings-vol --mount-path /opt/app-root/app/appsettings.json --configmap-name=appsettings-cm --sub-path=appsettings.json

  ### Web App. 

      oc new-app dotnet:7.0-ubi8~https://github.com/arunhari82/dotnet-eShopOnWeb.git --name web-app --build-env DOTNET_STARTUP_PROJECT=src/Web/Web.csproj -e ASPNETCORE_URLS='http://+:8080' --strategy=source
      
      oc set volume dc/web-app --add --name appsettings-vol --mount-path /opt/app-root/app/appsettings.json --configmap-name=appsettings-cm --sub-path=appsettings.json
      
      oc create route edge --service=web-app
          
 ## Docker Demo

 Install SQL Server which is a Prerequisite

 ### Public API :

 Use Import from Git from Console. Select the docker file as `src/PublicApi/Dockerfile`. Refer Screenshot below

 ![](/openshift/publicApi/assets/public-api-docker-statergy.png) 



        
    