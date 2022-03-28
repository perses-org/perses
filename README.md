
# Perses

Perses framework for Distributed Mobile Applications Performance Analysis.

There are two different ways to use Perses: (i) An standalone uasge in the development environment, manually run by an operator and (ii) integrate Perses in the CI/CD Pipeline with Github Actions so it will be automatically launched. The following two sections deteail each of those kinds of usage.

# Standalone usage

## Prerequisites

1. Download and install git -> https://git-scm.com/book/en/v2/Getting-Started-Installing-Git

2. Download and install nodejs-> https://nodejs.org/en/download/

3. Download and install terraform -> https://www.terraform.io/downloads.html

4. Amazon Web Service (AWS) account -> https://aws.amazon.com/

5. [Verify the possibility of .metal instance creation](MetalVerification.md)

6. Download [Perses Engine](https://github.com/perses-org/perses-engine) or sending an email to slasom (at) unex.es

7. Into perses engine folder:
- `npm install`



## Configuration File *test/configExample.yaml*
Perses needs a configuration file where the characteristics of the virtual scenario, the tests, and the desired QoS are defined.


![](Perse-config file.png)


**Author:** name of the operator. Used as a prefix to avoid name collision.

**Scenario:** to deploy the virtual scenario, it is necessary to define the infrastructure characteristics. These characteristics are defined in the following parameters.
_Instance Type_, this property indicates the AWS EC2 instance type to host the virtual scenarios. Currently, Perses uses AWS as a cloud infrastructure provider to deploy virtual scenarios, but it can be easily extended to other providers. Please note that to be able to emulate Android mobile devices, the cloud instance requires [KVM virtualization](https://developer.android.com/studio/run/emulator-acceleration). Only instances with high capacity provide this characteristic. For example, the.metal of AWS EC2 (for example, _c5.metal_).
_Region_, acronym for the region where the AWS resources are to be used, e.g., _eu-west-1_.
_AMI_ID_, this parameter specifies the Amazon Machine Image (AMI) to be used. Perses is prepared to use Ubuntu Server 18.04. Please note that depending on the region used, the AMI ID changes even if it is the same OS.

**Application Id:** this parameter specifies the Id of the mobile application. Each Android app has a unique application Id (e.g., _com.example.myapp_). Perses must know which application to run for the evaluation.

**Devices:** these parameters define the set of virtual devices to be deployed, their operating system, and their hardware characteristics. It can define several sets of devices with different characteristics to simulate a virtual scenario with heterogeneous devices. The most important properties are as follows: _Id_ is the name identifying the set of devices. _Type_, _'mobile'_ by default (for future work, it is intended to deploy other types of devices). _Devices_ is the number of devices to be deployed. _Android Version_ is the Android OS version number. Currently, Perses supports the Android versions from 6 to 11. _Hardware_, this parameter contains the hardware characteristics that the devices shall have. _CPU_, the maximum amount of available CPU resources (from the AWS instance) each device should use. Please note that a minimum of _1.5_ is recommended, and smaller values have a negative impact on the performance of virtual devices. On the other hand, _RAM_ indicates the maximum amount of memory each device can use. A minimum of _3g_ (3GB) is recommended. Smaller values cause virtual devices to slow down.


**Time Wait:** to ensure that all virtual devices are available simultaneously, it is necessary to wait a timeout period for them to be deployed. This parameter specifies the lag (1m '1 minute', 1h '1 hour') required to deploy the devices previously specified.



**Network:** Mobile devices communicate with the cloud or with other devices via a network infrastructure. In order to emulate the communication between devices, it is necessary to emulate this infrastructure. One of the most important properties is network latency. The latency is the time it takes for a packet to be transmitted within the network. Therefore, we have added the latency parameter that allows the developer to indicate the latency of the device to the other devices, nodes, etc. with which it communicates.


Please note that we have only indicated the latency between mobile and cloud for simplicity, but Kathara allows developers to define also the latency between devices. Likewise, we have only considered connections through WIFI and LTE interfaces because these are the ones currently supported by Kathara and the Android virtual image. For example, the Bluetooth connection is not possible to simulate due to the limitations of Android emulators.

Regarding the network topology, currently, a router is deployed where all the virtual mobile devices are connected. Note that Kathara allows one to define different network topologies. Future releases of Perses will support the definition of more complex infrastructures.

Therefore, the _Latency_ property indicates the additional average latency between the virtual mobile devices and the cloud node due to the emulation of the WIFI/LTE network (1s ’1 second’, 100ms ’100 milliseconds’).


Tests: this parameter defines the set of tests to evaluate the application (E2E and user interface tests). Each test set consists of the following parameters. _Id_ is the name identifying the test. _Type_ can be _apipecker_ or _espresso_. The _apipecker_ test is composed of the _config_ parameter to define the characteristics of the E2E test. _Iterations_ is the number of test repetitions. _Concurrent Users_ allow simulating concurrent users running the functionality to be evaluated and thus check how much load it can support. _Delay_ is the waiting time (in ms) between iterations. _Url_ is the API URL where the request to execute a functionality to start the E2E test is sent. _Expect_ allows different mathematical methods to be defined to analyze the results. Currently, it allows developers to use the _mean_, i.e., Perses will calculate the average of the results and compare it with the indicated value in the _Under_ field to determine whether the result is below the expected result.

**Log Tags:** the logs of devices offer much information about the system, services, etc. This parameter allows indicating keywords to filter the logs of the virtual devices and obtain important information. These keywords must be included in the implementation of the application to appear in the log. For example, we can indicate in the log via the keyword _"ExecutionTime"_, the processing time of the mobile phones. _Custom_ indicates the name of the file containing the custom function to analyze the results obtained from the log tag (e.g., function to calculate the mean _Execution Time_).


## Credentials Configuration

Configure *test/credentialExample.yaml*

- AWS Access Key
- AWS Secret Key
- AWS Key '.pem' absolute path
- AWS Key name


  

## Command line usage

Usage: `node index -a action -g <config.yaml> -c <credentials.yaml> -n projectName`

**Where : action = setup | launch | tests | getLogs | destroy** 

### Setup project
` node index.js -a setup -g test/configExample.yaml -c test/credentialsExample.yaml -n projectName `


### Launch project

`node index.js -a launch -n projectName`

### Run tests

`node index.js -a tests -n projectName`

### Get Logs

`node index.js -a getLogs -n projectName`
  
### Destroy project

`node index.js -a destroy -n projectName`

# Devops automation with GHA
Perses can be integrated with a devops cycle with the help of [GHA](https://github.com/features/actions)

## Prerequistes
 - Graddle configured to be run into the project (Having the script gradlew / graddle.bat [with execution rights configured propperly](https://stackoverflow.com/questions/17668265/gradlew-permission-denied) in the root of the repo)
 - APK build is configured to be generated at ``/app/build/outputs/apk/debug/app-debug.apk``
 - APK for Espresso tests build is configured to be generated at ``/app/build/outputs/apk/androidTest/debug/app-debug-androidTest.apk``
 
## Configuration
1. Copy [GHA workflow](https://github.com/perses-org/gha/blob/master/workflow/perses-workflow.yml) into your repo ``.github/workflow/perses.yml`` 
2. Copy perses [configuration template](https://github.com/perses-org/gha/blob/master/template/.perses.yml) ``.perses.yml`` to the root of your repo and fill it in with your preferences.
3. Include the following secrets with your AWS credentials:
   - AWS_ACCESS_KEY
   - AWS_SECRET_KEY
   - KEY_NAME
   - KEY_PEM

# Recommendations
Based on our experience, depending on the number of devices that compose the experiment configuration, we propose the folloing values for the ``time_wait`` parameter in the config (We assume a **EC2 - C5.metal: (1.5 CPU and 3G RAM)** as the exprimental host where Perses is running):
- 3 min for 2 devices
- 3 min for 4 devices
- 4 min for 8 devices
- 6 min for 16 devices
- 9 min for 32 devices
- 18 min for 50 devices
- 25 min for 60 devices (max devices)
