# mule-ch2-cicd
A template for trying CICD using Github Actions for Cloudhub 2.0 (CH2) deployments

## Quick Start
There are two background steps required to get this this repo working as it is currently configured:
- Using connected app (client id and client secret). This isn't necessary, but is my recommendation if you have 2 factor authentication on your username and password. Feel free to change the pom.xml and workflow.yml to use your username and password instead. Here is a guide on [how to create a connected app](https://docs.mulesoft.com/access-management/connected-apps-developers). Use these settings
    - App acts on its own behalf (client credentials)
    - With the following scopes
        - Exchange Admin
        - Runtime Manager: create, download, manage, read
        - General: view environment, view organization
        - Design Center: developer
- If you want to use m-unit, you'll need Nexus Enterprise repo access. Nexus is where Mulesoft hosts all their files and access rights are scoped to your username and password. The only way to get access is to ask [Mulesoft support](https://www.mulesoft.com/support-login) for it. Even as a Mulesoft Employee I had to ask support for credentials.

### Steps
Once you have connected app credentials and nexus enterprise repo credentials we can follow these steps to test this repo yourself.
1. Clone or download the repo
2. Modifications needed in the pom.xml file:
    - groupId. Find this from Anypoint Platform >> Access Management >> Organization and clicking the Org link of your choice to bring up a popup that has the Organisation Id
    - version. Feel free to change this to 1.0.0 or 1.0.0-SNAPSHOT for your first try. A super quick background about versions. -SNAPSHOT are supposed to be used as developer-only versions, end users are not meant to see them because they could have lots of breaking changes yet to be made. Non-SNAPSHOT versions are "production" versions for end user usage. Every time you create changes in your Mule app, you should change the version to the appropriate value. See [semver](https://semver.org/) for a guide
    - build>>plugins>>plugin>>cloudhub2Deployment>>
        - environment. I have hardcoded this to "Sandbox", but you can make it a variable (e.g. ${environment}) and reference it from workflow.yml
        - any other variables, e.g. target,muleVersion,vCores,etc
        - refer to this [guide](https://docs.mulesoft.com/mule-runtime/4.4/deploy-to-cloudhub-2) for variable possibilities 
3. Modifications in the .github/workflows/workflow.yml file:
    - build-and-deploy environment variables
      - APP_NAME. Change this to whatever you wish
      - POM_FILE. Only use this if you have different maven build, test and deploy settings you want to try in another pom file. The default is pom.xml, but you can see in this repo I have one called pom-no-munit.xml. This pom file doesn't have m-unit config and therefore doesn't require working Nexus Enterprise repo credentials.
      - MUNIT_CUSTOM_FILE. By default, whenever you run the "mvn" command where the build step is involved, all tests in the src/test/munit folder are run, this is unnecessary and slows down our workflow. So in all relevant maven commands I have the -DskipTests flag. I then have a dedicated test step, where I specifically only use test3-params.xml. The reason is that in test2-semver-mentioned.xml, there are failing tests there on purpose and our workflow won't run if this test is included
4. Enter in your Github Actions secrets
    - MULE_CLIENT_ID
    - MULE_CLIENT_SECRET
    - MULE_NEXUS_USERNAME
    - MULE_NEXUS_PASSWORD
5. Now every change pushed to the main branch, 1 of 3 things happens.
    1. If there's a change in -SNAPSHOT version, your app gets deployed in Anypoint Exchange (you can view versions there) and deployed to CH2
    2. If there's no change in production version, deploys to Exchange and CH2 get skipped
    3. if there's a change in production version, your app gets deployed in Anypoint Exchange (you can view versions there) and deployed to CH2


![Mulesoft Logo](https://images.g2crowd.com/uploads/product/image/social_landscape/social_landscape_08f1e92399dc8ab3acadf4bbc964b6cf/mulesoft-anypoint-platform.png)
