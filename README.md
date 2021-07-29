
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

  

## Configure Test

Configure *test/credentialExample.yaml*

- AWS Access Key
- AWS Secret Key
- AWS Key '.pem' absolute path
- AWS Key name


Configure *test/configExample.yaml*
- Author
- Region (if necessary)
- Apk absolute path (the Apk test is in 'test' folder )
- Log tags
- Tests

  

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
