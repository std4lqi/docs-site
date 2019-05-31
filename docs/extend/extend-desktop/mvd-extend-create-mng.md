# Creating and managing applications
To extend Zowe, you can create application plug-ins. Application plug-ins are installable sets of files that provide resources that can be either a web-based user interface and a set of RESTful services, a just a set of RESTful services.

## Setting environment variables
Before you build an application plug-in, you must set the environment variables that support the plug-in environment. To set up the environment, the node must be accessible on the PATH. To determine if the node is already on the PATH, issue the following command from the command line:

```text
node --version
```

If the version is returned, the node is already on the PATH. If nothing is returned, you can set the PATH using the *NODE_HOME* variable. The *NODE_HOME* variable must be set to the directory of the node install. You can use the `export` command to set the directory. For example:

```text
export NODE_HOME=node_installation_directory
```

Using this directory, the node will be included on the PATH in `nodeCluster.sh`. (`nodeCluster.sh` is located in `zlux-app-server/bin`).


## Using the sample application
You can experiment with the provided `sample-app` sample application plug-in.

Before you can build the sample application, you must add Node.js and npm to the PATH. Then use the `npm run build` or `npm start` commands to build the sample application. These commands are configured in `package.json`.

**Note:**

- If you change the source code for the sample application, you must rebuild it.
- If you want to modify `sample-app`, you must run `npm install` in the Zowe Desktop and the `sample-app/webClient`. Then, you can run `npm run build` in `sample-app/webClient`.
- Ensure that you set the `MVD_DESKTOP_DIR` system variable to the Zowe Desktop plug-in location. For example: `<ZLUX_CAP>/zlux-app-manager/virtual-desktop`.



1. Add an item to `sample-app`. The following figure shows an excerpt from `app.component.ts`:

   ```text
     export class AppComponent {
       items = ['a', 'b', 'c', 'd']
       title = 'app';
       helloText: string;
       serverResponseMessage: string;
   ```

2. Save the changes to `app.component.ts`.

3. Issue one of the following commands:

   - To rebuild the application plug-in, issue the following command:

   ```text
      npm run build
   ```

   - To rebuild the application plug-in and wait for additional changes to `app.component.ts`, issue the following command:

   ```text
     npm start
   ```

4. Reload the web page.

5. If you make changes to the sample application source code, follow these steps to rebuild the application:

   1. Navigate to the `sample-app` subdirectory where you made the source code changes.

   2. Issue the following command:

      ```text
       npm run build
      ```

   3. Reload the web page.

## Application structure
Some text

## Creating data services
Some text

## Accessing resources with the URI broker 
Some text

## Communicating with other applications
Some text

## Reporting errors
Some text