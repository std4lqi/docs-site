# Onboard your service with the Zowe API Meditation Layer

Use this guide to onboard your REST API service into the Zowe API Mediation Layer. This article outlines a step-by-step process to make your API service available in the API Mediation Layer.

## Communication between the application client and application server
The API ML Discovery Service uses [Netflix/Eureka](https://github.com/Netflix/eureka) as a communication technology. Eureka is a REST (Representational State Transfer) based service that is primarily used to locate services.

Eureka has [endpoints](https://github.com/Netflix/eureka/wiki/Eureka-REST-operations) to onboard (register) your service to the API ML Discovery Service. Use Eureka endpoints to register your service with the Discovery Service and send a periodic heartbeat to the Discovery Service. 

### Service registration
Registration requires that the following list of parameters are defined by Eureka. These parameters are sent to the server at the time of registration. 

When your application starts, call the following API with the POST method in the following format:

 ```{host}/eureka/apps/{serviceId}```

The following code block shows the format of the parameters in your POST call:

```xml
<?xml version="1.0" ?>
<instance>
  <app>{serviceId}</app>
  <ipAddr>{ipAddress}</ipAddr>
  <port enabled="false">{port}</port>
  <securePort enabled="true">{port}</securePort>
  <hostName>{hostname}</hostName>
  <vipAddress>{serviceId}</vipAddress>
  <secureVipAddress>{serviceId}</secureVipAddress>
  <instanceId>{instanceId}</instanceId>
  <dataCenterInfo>
    <name>MyOwn</name>
  </dataCenterInfo>
  <metadata>
        ...
   </metadata>
</instance>
```
where:

* **app** is the service id
* **ipAddr** is the ip address of instance
* **hostname** is the hostname of instance
* **port** is the port of instance when you use http, set `enabled="true"`
* **securePort** is the port of instance when you use https, set `enabled="true"`
* **vipAddress** is the service id when you use http
* **secureVipAddress** is the service id when you use https
* **instanceId** is a unique id for an instance. You define a unique value for the instanceId in the following format: ```{hostname}:{serviceId}:{port}```
* **metadata** is the set of parameters described in the following topic [the metadata of service]()


### Sending heartbeat to APIML Discovery
After registration, a service must send a heartbeat periodically to the Discovery Service to indicate that the service is available. If the Discovery Service does not receive a heartbeat, the service instance is deleted in the Discovery Service.

**Note:** We recommend that the interval for the heartbeat be no more than 30 seconds.

Use the PUT method in the following format to tell the Discovery Service that your service is available:

```{host}/eureka/apps/{serviceId}/{instanceId}```

## APIML Service Onboarding Metadata
At registration time, provide metadata in the following format which include the following parameters:

```xml
<metadata>
  	       <apiml.catalog.tile.id>demoapp</apiml.catalog.tile.id>
            <apiml.catalog.tile.title>Sample API Mediation Layer Applications</apiml.catalog.tile.title>
            <apiml.catalog.tile.description>Applications which demonstrate how to make a service integrated to the API Mediation Layer ecosystem</apiml.catalog.tile.description>
            <apiml.catalog.tile.version>1.0.1</apiml.catalog.tile.version>
            <apiml.service.title>Sample Service ©</apiml.service.title>
            <apiml.service.description>Sample API service showing how to onboard the service</apiml.service.description>
            <apiml.routes.api__v1.gatewayUrl>api/v1</apiml.routes.api__v1.gatewayUrl>
            <apiml.routes.api__v1.serviceUrl>/sampleclient/api/v1</apiml.routes.api__v1.serviceUrl>
            <apiml.routes.ui__v1.serviceUrl>/sampleclient</apiml.routes.ui__v1.serviceUrl>
            <apiml.routes.ui__v1.gatewayUrl>ui/v1</apiml.routes.ui__v1.gatewayUrl>
            <apiml.routes.ws__v1.gatewayUrl>ws/v1</apiml.routes.ws__v1.gatewayUrl>
            <apiml.routes.ws__v1.serviceUrl>/sampleclient/ws</apiml.routes.ws__v1.serviceUrl>
            <apiml.apiInfo.0.apiId>org.zowe.sampleclient</apiml.apiInfo.0.apiId>
            <apiml.apiInfo.0.swaggerUrl>https://hostname/sampleclient/api-doc</apiml.apiInfo.0.swaggerUrl>
            <apiml.apiInfo.0.gatewayUrl>api/v1</apiml.apiInfo.0.gatewayUrl>
            <apiml.apiInfo.0.documentationUrl>https://www.zowe.org</apiml.apiInfo.0.documentationUrl>
 </metadata>
```
The following list describes the parameters contained in the metadata:

### Catalog parameters - apiml.catalog.tile.*

These parameters are used to populate the API Catalog. The API Catalog contains information about every registered API service. The Catalog also groups related APIs. Each API group has its own name and description. Catalog groups are constructed in real-time based on information that is provided by the API services. Each group is displayed as a tile in the API Catalog UI dashboard.

   * **apiml.catalog.tile.id**

       Specifies the unique identifier for the API services product family. This is the grouping value used by the API ML to group multiple API services together into "tiles". Each unique identifier represents a single API Catalog UI dashboard tile. Specify a value that does not interfere with API services from other products.

   * **apiml.catalog.tile.title**

       Specifies the title of the API services product family. This value is displayed in the API Catalog UI dashboard as the tile title

   * **apiml.catalog.tile.description**

       Specifies the detailed description of the API services product family. This value is displayed in the API Catalog UI dashboard as the tile description

   * **apiml.catalog.tile.version**

       Specifies the semantic version of this API Catalog tile. Increase the version when you introduce new changes to the API services product family details (title and description). 

### Service parameters - apiml.service.*
Define service information for API Catalog

   * **apiml.service.title**

       Specifies the human readable name of the API service instance (for example, "Endevor Prod" or "Sysview LPAR1"). This value is displayed in the API Catalog when a specific API service instance is selected. 

       **Tip:** We recommend that you provide a good default value or give good naming examples to the customers.

   * **apiml.service.description**

       Specifies a short description of the API service.

       **Example:** "CA Endevor SCM - Production Instance" or "CA SYSVIEW running on LPAR1".
         This value is displayed in the API Catalog when a specific API service instance is selected. 

        **Tip:** We recommend that you provide a good default value or give good naming examples to the customers. Describe the service so that the end user knows the function of the service.
    
### Routing parameters - apiml.routes
Define routing informations for APIML Gateway.
<!--Add more info for routing-->

   * **apiml.routes._prefix_.gatewayUrl**

       Both gateway-url and service-url parameters specify how the API service endpoints are mapped to the API gateway endpoints. The gateway-url parameter sets the target endpoint on the gateway.
       
   * **apiml.routes._prefix_.serviceUrl**

       Both gateway-url and service-url parameters specify how the API service endpoints are mapped to the API gateway endpoints. The service-url parameter points to the target endpoint on the gateway.


### API Info Parameters - apiml.apiInfo.*
Define api parameters for API(Swagger) documentation

   * **apiml.apiInfo._prefix_.apiId**
      
       Specifies the API identifier that is registered in the API Mediation Layer installation. The API ID uniquely identifies the API in the API Mediation Layer. The same API can be provided by multiple services. The API ID can be used to locate the same APIs that are provided by different services. The creator of the API defines this ID. The API ID needs to be a string of up to 64 characters that uses lowercase alphanumeric characters and a dot: `.`. We recommend that you use your organization as the prefix.
       
   * **apiml.apiInfo._prefix_.gatewayUrl**
      
       The base path at the API gateway where the API is available. Ensure that it is the same path as the _gatewayUrl_ value in the _routes_ sections.
          
   * **apiml.apiInfo._prefix_.documentationUrl**
      
       (Optional) Link to external documentation, if needed. The link to the external documentation can be included along with the Swagger documentation.
       
   * **apiml.apiInfo._prefix_.swaggerUrl**
      
       (Optional) Specifies the HTTP or HTTPS address where the Swagger JSON document is available.             
     
     **Important!** Ensure that each of the values for gatewayUrl parameter are unique in the configuration. Duplicate gatewayUrl values may cause requests to be routed to the wrong service URL.

The process of onboarding depends on the method that is used to develop the API service.

 - Rest API developed based on non-Java(such as Python, NodeJS)
      
   Required parameters should be defined and sent at registration time. For acting as a eureka client, there're eureka-client libraries depends on language. (for example, [python-eureka-client](https://pypi.org/project/py-eureka-client/), [eureka-js-client](https://www.npmjs.com/package/eureka-js-client))

 - [Rest API developed based on Java](https://www.zowe.org/docs-site/latest/extend/extend-apiml/api-mediation-onboard-overview.html#sample-rest-api-service)

### Verify if your service is onboard to API Meditation Layer
After the onboarding process, you can check whether your service is onboard to API Meditation Layer. For this, check your service metadata in APIML Discovery. Use the **GET** method in the following format to ask from the Discovery Service that your service is onboard:

```{host}/eureka/apps/{serviceId}```

You receive a response like this:

```xml
<application>
    <name>{serviceId}</name>
    <instanceId>{hostname}:{serviceId}:{port}</instanceId>
    <hostName>{hostname}</hostName>
    <app>{serviceId}</app>
    <ipAddr>{ipAddress}</ipAddr>
    <status>UP</status>
    <port enabled="false">{port}</port>
    <securePort enabled="true">{port}</securePort>
    <vipAddress>{serviceId}</vipAddress>
    <secureVipAddress>{serviceId}</secureVipAddress>
    <metadata>
            <apiml.service.description>Sample API service showing how to onboard the service</apiml.service.description>
            <apiml.routes.api__v1.gatewayUrl>api/v1</apiml.routes.api__v1.gatewayUrl>
            <apiml.catalog.tile.version>1.0.1</apiml.catalog.tile.version>
            <apiml.routes.ws__v1.serviceUrl>/sampleclient/ws</apiml.routes.ws__v1.serviceUrl>
            <apiml.routes.ws__v1.gatewayUrl>ws/v1</apiml.routes.ws__v1.gatewayUrl>
            <apiml.catalog.tile.description>Applications which demonstrate how to make a service integrated to the API Mediation Layer ecosystem</apiml.catalog.tile.description>
            <apiml.service.title>Sample Service ©</apiml.service.title>
            <apiml.routes.ui__v1.gatewayUrl>ui/v1</apiml.routes.ui__v1.gatewayUrl>
            <apiml.apiInfo.0.apiId>org.zowe.sampleclient</apiml.apiInfo.0.apiId>
            <apiml.apiInfo.0.gatewayUrl>api/v1</apiml.apiInfo.0.gatewayUrl>
            <apiml.apiInfo.0.documentationUrl>https://www.zowe.org</apiml.apiInfo.0.documentationUrl>
            <apiml.catalog.tile.id>demoapp</apiml.catalog.tile.id>
            <apiml.routes.ui__v1.serviceUrl>/sampleclient</apiml.routes.ui__v1.serviceUrl>
            <apiml.routes.api__v1.serviceUrl>/sampleclient/api/v1</apiml.routes.api__v1.serviceUrl>
            <apiml.apiInfo.0.swaggerUrl>https://hostname/sampleclient/api-doc</apiml.apiInfo.0.swaggerUrl>
            <apiml.catalog.tile.title>Sample API Mediation Layer Applications</apiml.catalog.tile.title>
    </metadata>
</application>
```

If API call response shows the parameters as you sent it means your service is onboard.

### External Resources
- https://blog.asarkar.org/technical/netflix-eureka/
- https://medium.com/@fahimfarookme/the-mystery-of-eureka-health-monitoring-5305e3beb6e9
- https://github.com/Netflix/eureka/wiki/Eureka-REST-operations



