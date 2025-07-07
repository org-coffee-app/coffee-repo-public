[[_TOC_]]

**You can run [Devbox Setup Script](https://o365exchange.visualstudio.com/O365%20Core/_wiki/wikis/O365%20Core.wiki/519808/Devbox-Setup-Script) to do the environment setup which has automated the environment setup experience.**

# Pre-Requisites

### 1. Git for windows - https://git-scm.com/download/win
### 2. Visual Studio 2022 latest stable version ([VS Installer](https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=enterprise&rel=17)) 
-  Install Enterprise 2022 version with ASP.NET and web development.
-  Select **.NET desktop development** and **Azure development** in Workloads tab.

### 3. NET Core SDK
-  Check the installed version of .NET Core SDK by running **`dotnet --list-sdks`** in powershell
-  If it's not **8.0.xxx**, download from [.NET SDK](https://dotnet.microsoft.com/en-us/download)

### 4. Service Fabric SDK and Runtime
-  To build and run Azure Service Fabric applications on your Windows development machine, install the Service Fabric Runtime and SDK by following [Set Up Windows Dev Environment - Azure Service Fabric](https://learn.microsoft.com/en-us/azure/service-fabric/service-fabric-get-started#install-the-sdk-and-tools) 
-  Service Fabric uses Windows PowerShell scripts for creating a local development cluster and for deploying applications from Visual Studio. By default, Windows blocks these scripts from running. 
To enable them, execute **`Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Force -Scope CurrentUser`** by running powershell as administrator

### 5. Download Latest AAD Certificate
- For accessing Azure Resources in **Pre-Production Environment (PPE)** like SQL Db, Cosmos Db, Storage Accounts etc., this certificate is required
- This certificate has an expiry and hence is required to be re-installed after some time 
- Steps to install the latest Certificate: 
  - Go to [AAD Cert](https://ms.portal.azure.com/?l=en.en-us#@microsoft.onmicrosoft.com/asset/Microsoft_Azure_KeyVault/Certificate/https://namgcsppe.vault.azure.net/certificates/AADCert). For accessing this certificate, the user should be a part of either
    - SG: [Search Connector Engineering Team](https://o365exchange.visualstudio.com/O365%20Core/_settings/permissions?subjectDescriptor=aadgp.Uy0xLTktMTU1MTM3NDI0NS0xMjA0NDAwOTY5LTI0MDI5ODY0MTMtMjE3OTQwODYxNi0zLTMwNDQxMDkwMi0zNjg5NDAzMjExLTI3MTQ0MTUxMjEtNTY2MTgxNDc1) - Only for the directs of Krishan Nagpal - Core Connectors and Framework team in IDC
    - SG: GraphConnectorPartnerDevs (gcpartnerdevs@microsoft.com) - **For the Partner Engineering teams like STCA, Power Platform, etc. You can request access from [IDWeb](https://idweb.microsoft.com/IdentityManagement/default.aspx)**.
  - Download the latest version in **PFX/PEM format**
  - Install the downloaded certificate. Choose **Local Machine** when prompted for store location. When asked for password, **leave it Empty** and click Next. Wait till a pop-up appears as **Import Successful**

### 6. Connect AzAccount

- Run `Connect-AzAccount -DeviceCode` in PowerShell
- Though this step is included in `init.cmd`, it does not work well in some rare case. Thus, it is good to run it in advance to make the environment setup smooth.

# Setup Repo in Local
```cmd
git clone https://o365exchange.visualstudio.com/DefaultCollection/O365%20Core/_git/GraphConnectors
```

## Setup Enlistment
Once you clone the repo, open `Developer Command Prompt for VS 2022`, run `init.cmd` in the root folder.
```cmd
> cd GraphConnectors
> init.cmd
```

`init.cmd` will install credential scanner and quickbuild, as well as create a shortcut on your desktop `CMD-GraphConnectors`. Right click on the shortcut -> Properties -> Shortcut -> Advanced -> Check Run as Administrator.

#If you have .NET 9.X.X Installed
You will need to modify the value of `rollForward` property in the `global.json` file of the project root. See example below:

`{
    "msbuild-sdks": {
      "Microsoft.Build.Traversal": "4.1.0",
      "Microsoft.Build.NoTargets": "3.7.56"
    },
    "sdk": {
      "version": "8.0.304",
      "rollForward": "latestMinor"
    }
  }` 
# Build GCS in Local
- In the shortcut `CMD-GraphConnectors` from `init.cmd`, type `gcsvs` to launch the Model D GCS.sln in Visual Studio. 
  -  Alternatively, navigate to **\GraphConnectors\sources\dev\GCS\Services** inside cloned location
  -  Open **GCS.sln** in Visual Studio with Administrator Permissions
-  Right click on the sln -> Build to build GCS solution.
-  If you are getting build errors mentioning .net framework targets, you need to update the global.json. Ensure rollForward is set to "rollForward": "latestMinor"

# Deploy GCS in Local
-  Start Service Fabric **1Node** Local Cluster from quick start

   ![image.png](/.attachments/image-8d051aab-7413-4274-acfe-f690d8e711bb.png)
   ![image.png](/.attachments/image-77eb00c0-51cf-42e8-9496-0e0c73fa7edb.png)

- Remove Mock Service Overrides
  - Comment out the `<!--Mock service overrides -->` section in files: local.1node.xml and local.5node.xml
    - Directory: `GraphConnectors\sources\dev\GCS\Services\GCSApp\ApplicationParameters\`
    - Specifically these lines:
  ```xml
  <!--Mock service overrides -->
  <!-- <Parameter Name="MockAadTokenProvider" Value="true" /> 
  <Parameter Name="StiTokenIssuerApiHostName" Value=" http://localhost:9000" />
  <Parameter Name="SubstrateBaseUrl" Value=" http://localhost:9000" />
  <Parameter Name="SubstrateAudienceUrl" Value=" http://localhost:9000/ssms" />
  <Parameter Name="MicrosoftGraphBaseUrl" Value=" http://localhost:9000" />
  <Parameter Name="IngestionB2UrlPrefix" Value=" http://localhost:9000" /> -->
  ```

-  Deploy GCS

   ![image.png](/.attachments/image-66a475b9-7feb-4f7d-a003-b3d7e1c7e4df.png)
   ![image.png](/.attachments/image-d6647453-77ff-43b0-9127-018cdf66ff38.png)
   Connection Endpoint should be **Local Cluster**
-  After the Deployment is done, open Service Fabric Explorer and validate whether all the services are Green
   
   ![image.png](/.attachments/image-5cf48af8-68e7-451f-84ea-c820d65aaf90.png)
   ![image.png](/.attachments/image-ff97c873-7e84-442b-af0d-b1213207cda5.png)
- Hit the **Health Check** endpoint to validate the availability
**`https://localhost/v1.0/admin/AdminDataSetCrawl/healthcheck?version=true`**

  ![image.png](/.attachments/image-600b89bb-364c-4c67-95b5-dff530f65a1b.png)

# Troubleshooting
-  Deploy GCS in **Debug mode** from Visual Studio

   ![image.png](/.attachments/image-2b291c40-88d1-4c3b-9f47-51da51a501c6.png)
- If there is build error related to **long name** for SourceAliasingService, follow these steps:
  **Enable Long Paths** in Windows 10/11 by modifying the registry:
  - Open the Registry Editor (regedit)
  - Navigate to HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem.
  - Create a new DWORD (32-bit) value named LongPathsEnabled.
  - Set its value to 1.
  - Restart your computer.

- If you are getting the below error while installing Service Fabric Runtime

  ![image.png](/.attachments/image-afd7796a-c4ea-4ca5-a91f-d79ecb643215.png)
  Make sure you followed the steps to install the runtime mentioned here [Install the runtime](https://learn.microsoft.com/en-us/azure/service-fabric/service-fabric-get-started#install-the-runtime)

- If the GCS App deployment succeeded in Debug mode and your debug points are not getting hit, make sure the service which you want to debug are attached.
To attach processes to the debugger, follow these steps:
  - Go to Debug > Attach to Process
  - Check the Show Processes for all users checkbox
  - Search for Microsoft.Graph.Connectors
  - Select the service processes which you want to debug
  - Click Attach

  ![image.png](/.attachments/image-9b6300cc-44a7-40cc-a533-c6b57b4b2aed.png)
 
- If you are getting below error while building GCS

  ![image.png](/.attachments/image-4aafbde4-cd67-49ff-ac73-37d82f91b936.png)
  File is locked as it is being used by another process
  It may be possible that the GCS is deployed locally in release mode, you can verify that by checking the Service Fabric Explorer.
  To resolve this, you can **Reset the service fabric cluster**

  ![image.png](/.attachments/image-69ad7d3a-66a2-4225-b770-cd6751088d41.png)

- Unable to fetch secret from key vault.
  ![image.png](/.attachments/image-55cf0e87-90cb-446f-9158-d6e4b34ab9f0.png)

- - Ensure you are connected to MSFT-AzVPN-Manual
- - Ensure you have the latest cert installed from here [AAD Cert](https://ms.portal.azure.com/?l=en.en-us#@microsoft.onmicrosoft.com/asset/Microsoft_Azure_KeyVault/Certificate/https://namgcsppe.vault.azure.net/certificates/AADCert)
- - If you are still facing the issue after checking first two, check if your vpn is connected over IPv6. VPN is supported when using MSFT-AzVPN-Manual (Azure VPN) over IPv4. If you disable IPv6, then AzureVPN will use IPv4.
To disable IPv6, go to Start->Run->ncpa.cpl. Right click your Network connection (e.g. Wi-Fi) -> Properties-> uncheck IPv6
![image.png](/.attachments/image-2d85ba95-6d65-4833-89b3-9ce6f17dc885.png)
