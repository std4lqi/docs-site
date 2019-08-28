# Onboard you service with API Meditation Layer

As an API developer, use this guide to onboard your REST API service into the Zowe API Mediation Layer. This article outlines a step-by-step process to make your API service available in the API Mediation Layer.

## Communication between the application client and application server
APIML Discovery uses [Netflix/Eureka](https://github.com/Netflix/eureka) as a communication technology. Eureka is a REST (Representational State Transfer) based service that is primarily used for locating services.

Eureka has [endpoints](https://github.com/Netflix/eureka/wiki/Eureka-REST-operations) to onboards(register) your service to APIML Discovery. Your service should register itself and send heartbeat periodically to APIML Discovery. For registering there are minimum require params defined by Eureka which should be sent to the server at the registration time.

```
```

* **appname** the service id
* **hostname** the hostname of instance
* **port** the port of instance when you choose http
* **nonSecurePortEnabled** true or false
* **securePort** the port of instance when you choose https
* **securePortEnabled** true or false
* **ipAddress** the ip address of instance
* **instanceId** the id of instance; hostname:serviceId:port
* **virtualHostname** the virtualHostname
* **secureVirtualHostname** the virtualHostname
* **metadata** the metadata

## APIML Service Onboarding Metadatas
At registration time, metadata section is provided by the parameters below:

```xml
<metadata>
  	        <apiml.catalog.tile.id>cademoapps</apiml.catalog.tile.id>
            <apiml.catalog.tile.title>Sample API Mediation Layer Applications</apiml.catalog.tile.title>
            <apiml.catalog.tile.description>Applications which demonstrate how to make a service integrated to the API Mediation Layer ecosystem</apiml.catalog.tile.description>
            <apiml.catalog.tile.version>1.0.1</apiml.catalog.tile.version>
            <apiml.service.title>Service Integration Enabler V2 Sample Application (Spring Boot 2.x) ©</apiml.service.title>
            <apiml.service.description>Sample API service showing how to integrate a Spring Boot v2.x application</apiml.service.description>
            <apiml.routes.api__v1.gatewayUrl>api/v1</apiml.routes.api__v1.gatewayUrl>
            <apiml.routes.api__v1.serviceUrl>/discoverableclient/api/v1</apiml.routes.api__v1.serviceUrl>
            <apiml.routes.ui__v1.serviceUrl>/discoverableclient</apiml.routes.ui__v1.serviceUrl>
            <apiml.routes.ui__v1.gatewayUrl>ui/v1</apiml.routes.ui__v1.gatewayUrl>
            <apiml.routes.ws__v1.gatewayUrl>ws/v1</apiml.routes.ws__v1.gatewayUrl>
            <apiml.routes.ws__v1.serviceUrl>/discoverableclient/ws</apiml.routes.ws__v1.serviceUrl>
            <apiml.apiInfo.0.apiId>org.zowe.discoverableclient</apiml.apiInfo.0.apiId>
            <apiml.apiInfo.0.swaggerUrl>https://localhost:10012/discoverableclient/api-doc</apiml.apiInfo.0.swaggerUrl>
            <apiml.apiInfo.0.gatewayUrl>api/v1</apiml.apiInfo.0.gatewayUrl>
            <apiml.apiInfo.0.documentationUrl>https://www.zowe.org</apiml.apiInfo.0.documentationUrl>
 </metadata>
```

### Catalog parameters - apiml.catalog.tile.*

These parameters are used to populate the API Catalog. The API Catalog contains information about every registered API service. The Catalog also groups related APIs. Each API group has its own name and description. Catalog groups are constructed in real-time based on information that is provided by the API services. Each group is displayed as a tile in the API Catalog UI dashboard.

      * `apiml.catalog.tile.id`

        Specifies the unique identifier for the API services product family. This is the grouping value used by the API ML to group multiple API services together into "tiles". Each unique identifier represents a single API Catalog UI dashboard tile. Specify a value that does not interfere with API services from other products.

      * `apiml.catalog.tile.title`

        Specifies the title of the API services product family. This value is displayed in the API Catalog UI dashboard as the tile title

      * `apiml.catalog.tile.description`

        Specifies the detailed description of the API services product family. This value is displayed in the API Catalog UI dashboard as the tile description

      * `apiml.catalog.tile.version`

        Specifies the semantic version of this API Catalog tile. Increase the version when you introduce new changes to the API services product family details (title and description). 

### Service parameters - apiml.service.*
Say something about services

      * `apiml.service.title`

        Specifies the human readable name of the API service instance (for example, "Endevor Prod" or "Sysview LPAR1"). This value is displayed in the API Catalog when a specific API service instance is selected. 

         **Tip:** We recommend that you provide a good default value or give good naming examples to the customers.

      * `apiml.service.description`

          Specifies a short description of the API service.

          **Example:** "CA Endevor SCM - Production Instance" or "CA SYSVIEW running on LPAR1".
          This value is displayed in the API Catalog when a specific API service instance is selected. 

        **Tip:** We recommend that you provide a good default value or give good naming examples to the customers. Describe the service so that the end user knows the function of the service.
    
### Routing parameters - apiml.routes
Say something about routing

      * `apiml.routes.<prefix>.gatewayUrl`

           Both gateway-url and service-url parameters specify how the API service endpoints are mapped to the API gateway endpoints. The gateway-url parameter sets the target endpoint on the gateway.
      * `apiml.routes.<prefix>.serviceUrl`

          Both gateway-url and service-url parameters specify how the API service endpoints are mapped to the API gateway endpoints. The service-url parameter points to the target endpoint on the gateway.


### API Info Parameters - apiml.apiInfo.*
Say something about api info
       * `apiml.apiInfo.<prefix>.apiId`
      
          Specifies the API identifier that is registered in the API Mediation Layer installation. The API ID uniquely identifies the API in the API Mediation Layer.
          The same API can be provided by multiple services. The API ID can be used to locate the same APIs that are provided by different services. The creator of the API defines this ID.
          The API ID needs to be a string of up to 64 characters that uses lowercase alphanumeric characters and a dot: `.`. We recommend that you use your organization as the prefix.
      * `apiml.apiInfo.<prefix>.gatewayUrl`
      
          The base path at the API gateway where the API is available. Ensure that it is the same path as the _gatewayUrl_ value in the _routes_ sections.
          
      * `apiml.apiInfo.<prefix>.documentationUrl`
      
          (Optional) Link to external documentation, if needed. The link to the external documentation can be included along with the Swagger documentation.
      * `apiml.apiInfo.<prefix>.swaggerUrl`
      
          (Optional) Specifies the HTTP or HTTPS address where the Swagger JSON document is available.             
        **Important!** Ensure that each of the values for gatewayUrl parameter are unique in the configuration. Duplicate gatewayUrl values may cause requests to be routed to the wrong service URL.


The process of onboarding depends on the method that is used to develop the API service.

 - Rest API developed based on non-Java(such as Python, NodeJS)
      
        Required parameters should be defined and sent at registration time. For acting as a eureka client, there're eureka-client libraries depends on language. (for example, [python-eureka-client](https://pypi.org/project/py-eureka-client/), [eureka-js-client](https://www.npmjs.com/package/eureka-js-client))


 - [Rest API developed based on Java](https://www.zowe.org/docs-site/latest/extend/extend-apiml/api-mediation-onboard-overview.html#sample-rest-api-service)



