# Logging

## Logging objects

This logging utility is based on three objects:

- Component Loggers: Objects which log messages for an individual component of the environment, such as a REST API for an App or to log user access.
- Destinations: Objects which are called when a component logger requests a message to be logged. Destinations determine how something is logged, for example, to a file or to a console, and what formatting is applied.
- Logger: Central logging object which can spawn component loggers and attach destinations

## Logger IDs

Because Zowe Apps have unique identifiers, both data services and App's web content are provided with a component logger that knows this unique ID such that messages logged can be prefixed with that ID. With the association of logging to IDs, verbosity of logs can be controlled by setting log verbosity by ID.

## Accessing logger objects

Logger

The core logger object is attached as a global for low-level access.

App Server

NodeJS uses `global` as its global object, so the logger is attached to: `global.COM_RS_COMMON_LOGGER`

Web

Browsers use `window` as the global object, so the logger is attached to:`window.COM_RS_COMMON_LOGGER`

## Component logger

Component loggers are created from the core logger object, but when working with an App, you should allow the App framework to create these loggers for you. An App's component logger is presented to data services or web content as follows

App Server

[Dataservice Context Object](https://github.com/zowe/zlux/wiki/ZLUX-Dataservices#router-data service-context)

Web

[Angular App Instance Injectible](https://github.com/zowe/zlux/wiki/Virtual-Desktop-&-Window-Management#logger)

## Logger API

The following constants and functions are available on the central logging object

| Attribute                   | Type     | Description                                                  | Arguments                   |
| --------------------------- | -------- | ------------------------------------------------------------ | --------------------------- |
| makeComponentLogger         | function | Creates a component logger - Automatically done by the App framework for data services and web content | componentIDString           |
| setLogLevelForComponentName | function | Sets the verbosity of an existing component logger           | componentIDString, logLevel |

## Component logger API

The following constants and functions are available to each component logger

| Attribute     | Type     | Description                                                  | Arguments               |
| ------------- | -------- | ------------------------------------------------------------ | ----------------------- |
| SEVERE        | const    | Is a const for logLevel                                      |                         |
| WARNING       | const    | Is a const for logLevel                                      |                         |
| INFO          | const    | Is a const for logLevel                                      |                         |
| FINE          | const    | Is a const for logLevel                                      |                         |
| FINER         | const    | Is a const for logLevel                                      |                         |
| FINEST        | const    | Is a const for logLevel                                      |                         |
| log           | function | Used to write a log, specifying the log level                | logLevel, messageString |
| severe        | function | Used to write a SEVERE log.                                  | messageString           |
| warn          | function | Used to write a WARNING log.                                 | messageString           |
| info          | function | Used to write an INFO log.                                   | messageString           |
| debug         | function | Used to write a FINE log.                                    | messageString           |
| makeSublogger | function | Creates a new component logger with an ID appended by the string given | componentNameSuffix     |

## Log levels

An enum, `LogLevel`, exists for specifying the verbosity level of a logger. The mapping is:

| Level   | Number |
| ------- | ------ |
| SEVERE  | 0      |
| WARNING | 1      |
| INFO    | 2      |
| FINE    | 3      |
| FINER   | 4      |
| FINEST  | 5      |

**NOTE:** The default log level for any logger is **INFO**.

## Log verbosity

Using the component logger API above, loggers can dictate at which level of verbosity a log message should be visible. The end user can configure the server or client to show more or less verbose messages by using the [core logger's API objects](https://github.com/zowe/zlux/wiki/Logging#logger-api).

Example: You want to set the verbosity of the org.zowe.foo App's data service, bar to show debugging info. `logger.setLogLevelForComponentName('org.zowe.foo.bar',LogLevel.DEBUG)`

**Configuring logging verbosity**

The App framework provides ways to specify what component loggers you would like to set default verbosity for, such that you can easily turn logging on or off as an end-user.

## Server startup logging configuration

[The server configuration file](https://github.com/zowe/zlux/wiki/Configuration-for-zLUX-App-Server-&-ZSS), allows for specification of default log levels, as a top-level attribute **logLevels**, which takes key-value pairs where the key is a regex pattern for component IDs, and the value is an integer for the log levels.

Example:

```
"logLevels": {
    "org.zowe.configjs.data.access": 2,
    //the string given is a regex pattern string, so .* at the end here will cover that service and its subloggers.
    "org.zowe.myplugin.myservice.*": 4
    //
    // '_' char reserved, and '_' at beginning reserved for server. Just as we reserve
    // '_internal' for plugin config data for config service.
    // _unp = universal node proxy core logging
    //"_unp.dsauth": 2
  },
```

