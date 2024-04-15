# Zoho CRM Streaming Agent CodeLib

##  **Introduction**

The Zoho CRM Streaming Agent CodeLib offers a secure mechanism to transmit Zoho CRM data to a local machine, whether it's a private server or the user's workstation. Utilizing socket communication with HTTP long-polling to maintain connection longevity, each socket connection is treated as a client. This CodeLib exclusively relies on socket.io library. For further insights into socket.io, refer [here](https://socket.io/docs/v4/).

**Note**

The Zoho CRM Streaming Agent CodeLib converts Zoho CRM data into text and then stores them in Catalyst. Each piece of Zoho CRM data is referred to as a **message**

## **Components Used in Catalyst**

- Catalyst Appsail
    - zoho-crm-bridge - This serves as a connecting server linking a local machine with data produced in Zoho CRM.

- Catalyst Serverless Functions
    - zoho_crm_event_listener - This is an event function, which gathers the data generated by Zoho CRM into the Catalyst datastore.
    - message_cleanup - This is a cron function used to clean up the datastore in the catalyst.
    - message_service - This is an advancedio function used to read the messages.

- Catalyst Cloud Scale Data Store
    - Message - This table is used to store the data produced in Zoho CRM

- Catalyst Cloud Scale Cron
    - Message_Cleanup - This cron is scheduled daily to clean up the Zoho CRM data stored in Catalyst.

- Catalyst Cloud Scale Event Listener
    - You need to configure an event rule manually.

## **How to use**

### **Configuring Catalyst**

Before installing any Catalyst CodeLib solution, please make sure to login to the Catalyst CLI using your Catalyst account by following the steps mentioned [here](https://docs.catalyst.zoho.com/en/cli/v1/cli-command-reference/).

Follow all the steps mentioned below to install, configure and execute the Zoho CRM Streaming Agent CodeLib.

#### **Step 1: Install the CodeLib solution**

If you are installing the CodeLib solution for an already existing Catalyst project, navigate to its root directory from your terminal and directly proceed with installing the CodeLib.

You can also initialize a new project following the steps mentioned in this page. After you initialize the project, proceed to navigate to its root directory and continue with the installation.

Execute the command below in the terminal to install the Zoho CRM Streaming Agent CodeLib solution:

```bash
catalyst codelib:install https://github.com/akilavan-jayakumar/codelib-zoho-crm-tunnelling-agent
```

Upon installation, the pre-configured Catalyst resources of the CodeLib solution will be automatically deployed to the Catalyst console.

#### **Step 2: Configure Functions Component**

1. Open the `functions` directory of your Catalyst project in your local system.
2. In the `catalyst-config.json` file of the `message_service` function, configure the values given below for the key `env_variables`:

    -   CODELIB_SECRET_KEY
3.  In the `catalyst-config.json` file of the `message_cleanup` function, add the values given below for the key `env_variables`:

    -   MESSAGES_MAX_RETENTION_DAYS

By default, the messages retention period would be `7` days. This step is optional.

**Note**

Every time you attempt to access any endpoints of the `message_service` function, you must include the `CODELIB_SECRET_KEY` in the request header with name `catalyst-codelib-secret-key`. This key allows you to access the Catalyst resources securely.

#### **Step 3: Configure Appsail Component**

1.  Create a directory named `server` in the root directory of your project.

2.  Execute the command below in the terminal to clone a github repository into the `server` directory.

 ```bash 
 git clone https://github.com/akilavan-jayakumar/appsail-socket-io-server ./server
 ```

3.  Once the repository has been cloned, initialize the server directory as an appsail project by following the documentation provided  [here](https://docs.catalyst.zoho.com/en/cli/v1/initialize-resources/initialize-appsail/). During the initialization of appsail, specify the `build` directory as the build path, `zoho-crm-bridge` as the name and `NodeJS 18` as the stack.

4.  Navigate to the `app-config.json` file located in the `server` directory, 
    1.  Configure the values given below for the key `env_variables`

        -  CODELIB_SECRET_KEY
        -  PROJECT_DOMAIN
    2.  Copy and paste the code given below for the key `scripts`

        ```bash
        {
            "preserve": "npm run build",
            "predeploy": "npm run build",
            "postserve": "rm -r build/",
            "postdeploy": "rm -r build/"
        }
        ```
    3. Copy and paste the code given below for the key `command`
    
        ```bash
        npm run start
        ```

After completing the configuration changes as instructed, proceed to deploy the CodeLib solution again from your local terminal. This ensures that the modifications are applied and visible in the remote console.

#### **Step 4 : Creating Event Listener**

Create a custom event listener with type `zoho events` within the Catalyst console. Then, set up a rule in the same with `zoho_crm_event_listener` as the target function, and provide the necessary values accordingly.

### **Configuring Local Machine**

1.  Create a directory separate from the project's root directory
2.  Execute the command below in the terminal to clone a github repository
```bash
 git clone https://github.com/akilavan-jayakumar/codesample-socket-io-client
```
3.  After successfully cloning the repository, navigate to the directory where it was cloned. Locate and open the `.env` file within this directory. Within the file, update the values for the keys:

    -  APPSAIL_DOMAIN
    -  CODELIB_SECRET_KEY

To obtain the `APPSAIL_DOMAIN`, navigate to the Catalyst Console, select the relevant project, then proceed to Serverless Service -\> AppSail -\> Click on `zoho-crm-bridge` appsail. In the overview section, you will find the appsail domain.

4.  To install the necessary packages and initiate the program, follow these commands in the specified order. Ensure your terminal's current directory is the root of the cloned repository.

    1.  Install Packages
    ```bash
    npm install
    ```
    2.  Start the program
    ```bash
    npm run start
    ```

**Note**

If you need to read a message from a particular ID, include the message ID in the socket.io client query using the name last_read_message. To know more about socket-io client you can refer [here](https://socket.io/docs/v4/client-initialization/). The local machine's program should remain active to receive data from Zoho CRM consistently.

Once the configuration has been completed, each time an event is generated in the Catalyst event listener, event data is transformed as a `text` message and stored in catalyst, and then the message will be transmitted to the local machine.
