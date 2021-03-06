# Developers guide for Kafka Functions for Java


### Modify the sample

The sample is already created and configured. However, you might want to change the configuration. For example, the Function App name. It might conflict with our function app. Let's change the config to fit your environment.

### Modify pom.xml (optional)

Go to the sample app directory, build the sample. If you want to change the function App that you want to deploy this app, modify the following section on the `pom.xml` file. The current `pom.xml` file is for windows and deploys windows based function app. If you are using a mac, you can rename the `pom_linux.xml` to `pom.xml.` It deploys a Linux based function app.

_pom.xml_  

```xml
        <functionAppName>kafka-function-20190419163130420</functionAppName>
        <functionAppRegion>westus</functionAppRegion>
        <stagingDirectory>${project.build.directory}/azure-functions/${functionAppName}</stagingDirectory>
        <functionResourceGroup>java-functions-group</functionResourceGroup>
```

### Modify install_extension shell (Optional)

Modify the functionAppName of `install_extension.ps1` (windows) or `install_extension.sh` (mac/bash) if you change the functionAppName.

_isntall_extension.ps1_

```powershell
$FunctionAppName = "kafka-function-20190419163130420"
```

_install_extension.sh_

```bash
FUNCTION_APP_NAME=kafka-function-20190419163130420
```

### Modify TriggerFunction.java (Windows user only)

If you want to run the sample on your Windows with Confluent Cloud and you are not using DevContainer, uncomment the following line. It is the settings of the CA certificate. .NET Core that is azure functions host language can not access the Windows registry, which means it can not access the CA certificate of the Confluent Cloud.

```java
sslCaLocation = "confluent_cloud_cacert.pem",
```

### Build and package the app

```bash
$ mvn clean package
```

## Install the KafkaTriggerExtension

Install extension script for installing the Kafka extension. For windows, It also install `confluent_cloud_cacert.pem`.

_windows_

```powershell
PS > .\isntall_extension.ps1
```

_mac, bash, or DevContainer_

```bash
$ ./install_extension.sh
```

**NOTE**: In case of DevContainer on Windows, the repository is cloned by CRLF, it is mounted on docker container. Please change the CRLF to LF. For the Visual Studio Code, click `install_extension.sh` then find `CRFL` on the right bottom of the Visual Studio Code and change it to `LF`.

Check if there is dll packages under the `target/azure-functions/Kafka-function-(some number)/bin`. If it is success, you will find `Microsoft.Azure.WebJobs.Extensions.Kafka.dll` on it. 

## Run the Azure Functions

## Configuration (Optional)
Copy the `confluent/local.settings.json.example` to `confluent/local.settings.json` and configure it. This configuration requires for `KafkaTrigger-Java-Many` sample that works with the confluent cloud. 

_local.settings.json_
```
{
    "IsEncrypted": false,
    "Values": {
    "BrokerList": "YOUR_BROKER_LIST_HERE",
    "ConfluentCloudUserName": "YOUR_CONFLUENT_USER_NAME_HERE",
    "ConfluentCloudPassword": "YOUR_CONFLUENT_PASSWORD_HERE"
    }
}
```

## Run 

This sample includes two samples. Go to `TriggerFunction.java.` You will find `KafkaTrigger-Java` and `KafkaTrigger-Java-Many.` `KafkaTrigger-Java` is a sample for local Kafka cluster. `KafkaTrigger-Java-Many` is a sample for [Confluent Cloud in Azure](https://github.com/Azure/azure-functions-kafka-extension#connecting-to-confluent-cloud-in-azure) with batch execution. If you execute this sample as-is, one of the functions will cause an error since these are the different configurations. Remove or comment on one of them and enjoy it.

Before running the Kafka extension, you need to configure `LD_LIBRARY_PATH` to the `target/azure-functions/kafka-function-20190419163130420`. For the DevContainer, the configuration resides in the `devontainer.json.` You don't need to configure it. 

```
$ mvn azure-functions:run
```

If you want to run with Debug mode, try this. 

```
$ mvn azure-functions:run -DenableDebug
```
Then you can attach with Debugger. 

_.vscode/launch.json_

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Attach to Java Functions",
            "type": "java",
            "request": "attach",
            "hostName": "127.0.0.1",
            "port": 5005,
        }
    ]
}
```

## Deploy to Azure

If you want to deploy your functions to Azure, modify your `pom.xml.` Find two pieces to configure it. 

### Configuration 

#### FunctionApp name, region, and resource group

Change this info if you already have a function app. 

_pom.xml_

```xml
        <functionAppName>kafka-function-20190419163130420</functionAppName>
        <functionAppRegion>westus</functionAppRegion>
        <stagingDirectory>${project.build.directory}/azure-functions/${functionAppName}</stagingDirectory>
        <functionResourceGroup>java-functions-group</functionResourceGroup>
```

#### os and pricing tier (premium plan)

_pom.xml_

```xml
 <runtime><os>windows</os></runtime>
 <pricingTier>EP1</pricingTier>
```

### deploy to Azure

#### deploy the app
The maven target will create a function app if it does not exist then deploy the application. 

```
$ mvn azure-functions:deploy -e
```

#### Configure AppSettings 

Go to Azure Portal, select the FunctionAPp, then go to Configuration > Application settings. You need to configure these application settings. `BrokerList`, `ConfluentCloudUsername` and `ConfluentCloudPassowrd` are required for the sample. 
`LD_LIBRARY_PATH` is required for Linux based Function App. That is references so library that is included on the Kafka extensions. 

| Name | Description | NOTE |
| BrokerList | Kafka Broker List | e.g. changeme.eastus.azure.confluent.cloud:9092 |
| ConfluentCloudUsername | Username of Confluent Cloud | - |
| ConfluentCloudPassword | Password of Confluent Cloud | - |
| LD_LIBRARY_PATH | /home/site/wwwroot/bin/runtimes/linux-x64/native | Linux only |

#### Send kakfka event

Send Kafka events from a producer. You can call `KafkaInput-Java` for a local cluster, or you can use [ccloud](https://docs.confluent.io/current/cloud/cli/index.html) command for the confluent cloud.

```bash
$ ccloud login
$ ccloud kafka topic produce message
```

For more details, Go to [ccloud](https://docs.confluent.io/current/cloud/cli/command-reference/ccloud.html).

If you want to send an event to the local Kafka cluster, you can use
[kafakacat](https://docs.confluent.io/current/app-development/kafkacat-usage.html) instead.

```bash
$ apt-get update && apt-get install kafkacat
$ kafkacat -b broker:29092 -t users -P
```

# Resource

* [Quickstart: Create a function in Azure that responds to HTTP requests](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-azure-function-azure-cli?pivots=programming-language-java&tabs=bash%2Cbrowser)
* [azure-maven-plugins: AzureFunctions: Configuration Details](https://github.com/microsoft/azure-maven-plugins/wiki/Azure-Functions:-Configuration-Details)
* [Confluent cloud Quick Start](https://docs.confluent.io/current/quickstart/cloud-quickstart/index.html#cloud-quickstart)

