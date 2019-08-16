# Troubleshooting API ML

As an API Mediation Layer user, you may encounter problems with the functioning of API ML. This article presents known API ML issues and their solutions.

## Enable API ML Debug Mode

Use debug mode to activate the following functions:

- Display additional debug messages for the API ML 
- Enable changing log level for individual code components
    
**Important:** We highly recommend that you enable debug mode only when you want to troubleshoot issues.
Disable debug mode when you are not troubleshooting. Running in debug mode while operating API ML can adversely affect
its performance and create large log files that consume a large volume of disk space.

**Follow these steps:**

1. Locate the following shell script files in the `<Zowe install directory>/api-mediation/scripts` directory:

    - ```api-mediation-start-catalog.sh```
    - ```api-mediation-start-discovery.sh```
    - ```api-mediation-start-gateway.sh```

2. Open a file, for which you want to enable the debug mode.

3. Find the line which contains the `spring.profiles.include` parameter and change the value to `debug`: 
    ```
    -Dspring.profiles.include=debug \
    ```

4. Restart Zowe.

    You have enabled the debug mode.

5. (Optional) Reproduce a bug that causes issues and review debug messages. If you are unable to resolve the issue, contact CA Support.

6. Disable the debug mode. Modify the line which contains the `spring.profiles.include` parameter back to default:
    ```
    -Dspring.profiles.include= \
    ```
7. Restart Zowe.

    You have disabled the debug mode.
 ___
## Change the Log Level of Individual Code Components

You can change the log level of a particular code component of the API ML internal service at run time.

**Follow these steps:**

1. Enable API ML Debug Mode as described in Enable API ML Debug Mode.
This activates the application/loggers endpoints in each API ML internal service (Gateway, Discovery Service, and Catalog).
2. List the available loggers of a service by issuing the GET request for the given service URL:

    ```
    GET scheme://hostname:port/application/loggers
    ```
    
    Where:
    - **scheme**
    
        API ML service scheme (http or https)
    
    - **hostname**
    
        API ML service hostname
    
    - **port**
    
        TCP port where API ML service listens on. The port is defined by the configuration parameter MFS_GW_PORT for the Gateway,
    MFS_DS_PORT for the Discovery Service (by default, set to gateway port + 1), and MFS_AC_PORT for the Catalog 
    (by default, set to gateway port + 2).
    
    **Exception:** For the catalog you will able to get list the available loggers by issuing the GET request for the given service URL:
    ```
    GET [gateway-scheme]://[gateway-hostname]:[gateway-port]/api/v1/apicatalog/application/loggers
    ```

    **Tip:** One way to issue REST calls is to use the http command in the free HTTPie tool: https://httpie.org/.
    
    **Example:**
 
    ```
    HTTPie command:
    http GET https://lpar.ca.com:10000/application/loggers

    Output:
    {"levels":["OFF","ERROR","WARN","INFO","DEBUG","TRACE"],
     "loggers":{
       "ROOT":{"configuredLevel":"INFO","effectiveLevel":"INFO"},
       "com":{"configuredLevel":null,"effectiveLevel":"INFO"},
       "com.ca":{"configuredLevel":null,"effectiveLevel":"INFO"},
       ...
     }
    }
    ```
 
3. Alternatively, you extract the configuration of a specific logger using the extended **GET** request:

    ```
    GET scheme://hostname:port/application/loggers/{name}
    ```
    Where:

    - **{name}**

         is the logger name
    
4. Change the log level of the given component of the API ML internal service. Use the POST request for the given service URL:

    ```
    POST scheme://hostname:port/application/loggers/{name}
    ```
    The POST request requires a new log level parameter value that is provided in the request body:
    ```
    {

    "configuredLevel": "level"

    }
    ```
    Where:

    - **level**
    
        is the new log level: **OFF**, **ERROR**, **WARN**, **INFO**, **DEBUG**, **TRACE**
    
    **Example:**

    ```
    http POST https://hostname:port/application/loggers/com.ca.mfaas.enable.model configuredLevel=WARN
    ```



## Known Issues 

### API ML stops accepting connections after z/OS TCP/IP stack is recycled

**Symptom:**

When z/OS TCP/IP stack is restarted, it is possible that the internal services of API Mediation Layer 
(Gateway, Catalog, and Discovery Service) stop accepting all incoming connections, go into a continuous loop, 
and write a numerous error messages in the log.

**Sample message:**

The following message is a typical error message displayed in STDOUT:

```
2018-Sep-12 12:17:22.850. org.apache.tomcat.util.net.NioEndpoint -- Socket accept failed java.io.IOException: EDC5122I Input/output error. 

.at sun.nio.ch.ServerSocketChannelImpl.accept0(Native Method) ~.na:1.8.0.
.at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:478) ~.na:1.8.0.
.at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:287) ~.na:1.8.0.
.at org.apache.tomcat.util.net.NioEndpoint$Acceptor.run(NioEndpoint.java:455) ~.tomcat-coyote-8.5.29.jar!/:8.5.29. 
.at java.lang.Thread.run(Thread.java:811) .na:2.9 (12-15-2017).
```
**Solution:**

Restart API Mediation Layer. 

**Tip:**  To prevent this issue from occurring, it is strongly recommended not to restart TCP/IP stack while the API ML is running.

### Unexpected error when logging into API Catalog

**Symptom:**

Failure to log into API Catalog that is caused by failed z/OSMF authentication.

**Sample message:**

The following message is displayed when attempting to log into API Catalog:

```
Unexpected error, please try again later (SEC0002)
```

**Solution:**

Open ZOWESVR joblog and look for a message containing `ZosmfAuthenticationProvider` to identify the nature of the issue. Use one of the following solutions to fix the issue:

- [Change z/OSMF configuration](#change-z/osmf-configuration)
- [Fix the z/OSMF certificate](#fix-the-z/osmf-certificate)
- [Fix a DNS name in z/OSMF certificate](#fix-a-dns-name-in-z/osmf-certificate)

#### Change z/OSMF configuration

Change z/OSMF configuration.

**Follow these steps:**

1. Ensure that z/OSMF listens on localhost 127.0.0.1.
2. Identify z/OSMF PARMLIB member IZUPRMxx in SYS1.PARMLIB(IZUPRM00).
3. Change `HOSTNAME('10.175.81.28')` to `HOSTNAME('*')`.
<!-- TODO. We might need more context here -->
For more information, see [Syntax rules for IZUPRMxx](https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.3.0/com.ibm.zos.v2r3.izua300/izuconfig_IZUPRMxx.htm).
<!-- TODO. Check why this link was provided in the first place. Maybe we should expand the procedure to include all necessary steps that will fix the issue -->

#### Fix the z/OSMF certificate

Fix the invalid certificate for z/OSMF.
<!-- TODO. There are two ways of fixing the issue, but I'd rather pick one (most suitable) -->
**Follow these steps:**

1. Obtain a valid certificate for z/OSMF and place it into the z/OSMF keyring. For more information, see [Configure the z/OSMF Keyring and Certificate](https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.3.0/com.ibm.zos.v2r3.izua300/izuconfig_KeyringAndCertificate.htm).
2. Navigate to `$ZOWE_RUNTIME/api-mediation` and run the following command:
    ```
    scripts/apiml_cm.sh --action trust-zosmf 
    ```
3. (Optional) If you do not use the default z/OSMF userid (IZUSVR) and keyring (IZUKeyring.IZUDFLT), issue the following command:
    ```
    scripts/apiml_cm.sh --action trust-zosmf --zosmf-userid **ZOSMF_USER** --zosmf-keyring **ZOSMF_KEYRING**
    ```
    The `--zosmf-keyring` and `--zosmf-userid` options override the default userid and keyring.
<!-- TODO. Probably we should not add the insecure way of fixing this issue. -->

#### Fix a DNS name in z/OSMF certificate

Use one of the following options to fix a DNS name in the z/OSMF certificate:

- Request a new certificate that contains the existing DNS name.
- Change `ZOWE_EXPLORER_HOST` to a DNS name from subject alternative names of the z/OSMF certificate in `.zowe_profile` in home directory of user who runs Zowe installation. 
<!-- TODO. It's not "runs", it's smth else instead-->



### SEC0002 error when logging into API Catalog

**Symptom:**

Failure to log into API Catalog that is caused by incorrect z/OSMF configuration. 

**Sample message:**

The following message is displayed when attempting to log into API Catalog:

```
Connect to MVSDE25.lvn.broadcom.net:1443 .MVSDE25.lvn.broadcom.net/127.0.0.1. failed: EDC8128I Connection refused.; nested exception is org.apache.http.conn.HttpHostConnectException: Connect to MVSDE25.lvn.broadcom.net:1443 .MVSDE25.lvn.broadcom.net/127.0.0.1. failed: EDC8128I Connection refused
```
<!-- Make the text in the codeblock more readable -->
**Solution:**

Change z/OSMF configuration.

**Follow these steps:**

1. Ensure that z/OSMF listens on localhost 127.0.0.1.
2. Identify z/OSMF PARMLIB member IZUPRMxx in SYS1.PARMLIB(IZUPRM00).
3. Change `HOSTNAME('10.175.81.28')` to `HOSTNAME('*')`.
<!-- TODO. We might need more context here -->
For more information, see [Syntax rules for IZUPRMxx](https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.3.0/com.ibm.zos.v2r3.izua300/izuconfig_IZUPRMxx.htm).
<!-- TODO. Check why this link was provided in the first place. Maybe we should expand the procedure to include all necessary steps that will fix the issue -->

### SEC0002 error when logging into API Catalog

**Symptom:**

Failure to log into API Catalog that is caused by invalid z/OSMF certificate.

**Sample message:**

The following message is displayed when attempting to log into API Catalog:

```
Certificate for <MVSDE25.lvn.broadcom.net> doesn't match any of the subject alternative names: ..; nested exception is javax.net.ssl.SSLPeerUnverifiedException: Certificate for <MVSDE25.lvn.broadcom.net> doesn't match any of the subject alternative names: ..
```

**Solution:**

Fix the invalid certificate for z/OSMF using the following methods:

- [Suitable for production fix](#suitable-for-production-fix)
- [Suitable for development purposes fix](#suitable-for-development-purposes-fix)

#### Suitable for production fix

1. Obtain a valid certificate for z/OSMF and place it into the z/OSMF keyring. For more information, see [Configure the z/OSMF Keyring and Certificate](https://www.ibm.com/support/knowledgecenter/en/SSLTBW_2.3.0/com.ibm.zos.v2r3.izua300/izuconfig_KeyringAndCertificate.htm).
2. Navigate to `$ZOWE_RUNTIME/api-mediation` and run the following command:
    ```
    scripts/apiml_cm.sh --action trust-zosmf 
    ```
3. (Optional) If you do not use the default z/OSMF userid (IZUSVR) and keyring (IZUKeyring.IZUDFLT), issue the following command:
    ```
    scripts/apiml_cm.sh --action trust-zosmf --zosmf-userid **ZOSMF_USER** --zosmf-keyring **ZOSMF_KEYRING**
    ```
    The `--zosmf-keyring` and `--zosmf-userid` options override the default userid and keyring.
<!-- TODO. Probably we should not add the insecure way of fixing this issue. -->

#### Suitable for development purposes fix

1. Reinstall Zowe
2. Set the `verifyCertificatesOfServices` property to `false` in `zowe-install.yaml` to disable verification of certificates in Zowe.

**Important!** This method is non-secure way, only for development purposes.
<!-- What will hapen if I employ this fix -->


### SEC0002 error when logging into API Catalog

**Symptom:**

Failure to log into API Catalog that is caused by a DNS name not listed in the z/OSMF certificate.

**Sample message:**

The following message is displayed when attempting to log into API Catalog:

```
Certificate for <USILCA32.lvn.broadcom.net> doesn't match any of the subject alternative names: [ca32.ca.com, ca32, localhost, ca32-virt, ca32-virt.ca.com, ca32-virt1, ca32-virt1.ca.com, ca32-virt2, ca32-virt2.ca.com, usilca32, usilca32.ca.com]; nested exception is javax.net.ssl.SSLPeerUnverifiedException: Certificate for <USILCA32.lvn.broadcom.net> doesn't match any of the subject alternative names: [ca32.ca.com, ca32, localhost, ca32-virt, ca32-virt.ca.com, ca32-virt1, ca32-virt1.ca.com, ca32-virt2, ca32-virt2.ca.com, usilca32, usilca32.ca.com]
```

**Solution:**

Request a new certificate that contains the existing DNS name. Alternatively, you can change `ZOWE_EXPLORER_HOST` to a DNS name from subject alternative names of the z/OSMF certificate in `.zowe_profile` in home directory of user who runs Zowe installation. 
<!-- TODO. It's not "runs", it's smth else instead-->