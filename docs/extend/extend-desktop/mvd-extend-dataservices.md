# Using data services

The following data service object is an example taken from the sample-angular-app pluginDefinition.json file that is provided with the Zowe Application Server:

```
"dataServices": [
    {
      "type": "router",
      "name": "hello",
      "filename": "helloWorld.js",
      "routerFactory": "helloWorldRouter",
      "dependenciesIncluded": true,
      "initializerLookupMethod": "external",
      "version": "1.0.0"
    }
  ],
```

type: Valid values are `router`, `service`, `import`, `external`. 

- Use a router data service to use ExpressJS Routers attach actions to URLs and methods. Router data services run under the Zowe Application Server.
- Use a service data service to use the ZSS API to attach actions to URLs and methods. Service data services run under ZSS.
- Specify "import" to import a data service from another plug-in.
- Specify "external" to configure an endpoint in your plugin namespace as a proxy for an endpoint on another server.