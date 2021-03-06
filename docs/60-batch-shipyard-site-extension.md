# Batch Shipyard Site Extension for use with Azure Functions/App Service
The focus of this article is to describe the process for enabling the Batch
Shipyard site extension in Azure App Service (e.g., Azure Function App) to
be used for processing as a result of a trigger.

## Setup and Installation
The following assumes that you are able to interact with the
[Azure Portal](https://portal.azure.com/) for creating and setting up your
Azure Function App with the Batch Shipyard site extension.

### Step 1: Create an Azure Function App
The first step is to create an Azure Function App. The leftmost navigation
bar, you will see a plus sign. Select `Compute >` and then
`Function App`.

![60-site-extension-step1-0.png](https://azurebatchshipyard.blob.core.windows.net/github/60-site-extension-step1-0.png)

In the create blade, fill in the appropriate properties for your Function App.
You will need to ensure that the OS is set to `Windows`. This restriction
is only for Batch Shipyard to execute, and has no bearing on the actual
container workload invoked by Batch Shipyard (as that runs in the Batch
service). Additionally, ensure that your select a Hosting Plan that is
appropriate for your workload. Please see documentation
[here](https://docs.microsoft.com/azure/azure-functions/functions-scale)
regarding default and maximum timeouts (and how to modify them). It is
recommended to use an `App Service Plan` rather than a Consumption plan
for pre-warmed instances and other benefits. Use the `App Service plan/Location`
to create a new App Service plan (if applicable). Very basic and Consumption
plans may lead to startup delay in Batch Shipyard. You can ignore the
`Runtime Stack` language selection; you can keep it populated as .NET Core.
It has no bearing on the functionality of this Azure Function App for Batch
Shipyard as we will be installing Python as a site extension (in a later step).

![60-site-extension-step1-1.png](https://azurebatchshipyard.blob.core.windows.net/github/60-site-extension-step1-1.png)

### Step 2: Install Python 3.X x64 Site Extension
Now we'll install Python 3.x as a site extension as a pre-requisite for
Batch Shipyard. To do this, navigate to your newly created Function App.
Select your function app and then select `Platform features` on the right.

![60-site-extension-step2-0.png](https://azurebatchshipyard.blob.core.windows.net/github/60-site-extension-step2-0.png)

In the next blade, select `Advanced tools (Kudu)`. Do not use the
`Extensions` option as the amount of time given for an extension to
install through this method is too short.

![60-site-extension-step2-1.png](https://azurebatchshipyard.blob.core.windows.net/github/60-site-extension-step2-1.png)

Click on `Extensions` on the top navigation bar and then click on
`Gallery` below. This will load all of the available site extensions from
[nuget.org](https://www.nuget.org/). For the curious, the Batch Shipyard
site extension nuget page can be found
[here](https://www.nuget.org/packages/BatchShipyard).

![60-site-extension-step2-2.png](https://azurebatchshipyard.blob.core.windows.net/github/60-site-extension-step2-2.png)

You should select the latest version of Python 3.X x64 that is available.
You may have to search for it using the search box under the `Gallery` tab.
Click the `+` icon to install this site extension to your Azure Function App
environment.

![60-site-extension-step2-3.png](https://azurebatchshipyard.blob.core.windows.net/github/60-site-extension-step2-3.png)

### Step 3: Install Batch Shipyard Site Extension
After Python 3.X x64 installs successfully, find the `Batch Shipyard`
site extension in the same `Gallery` area. You may have to search for it
using the search box under the `Gallery` tab. Click the `+` icon to install
the Batch Shipyard site extension to your Azure Function app environment.

![60-site-extension-step3-0.png](https://azurebatchshipyard.blob.core.windows.net/github/60-site-extension-step3-0.png)

Please be patient during this step, as installation may take a while depending
upon the App Service plan you have selected. Once the Batch Shipyard site
extension installation completes, hit the `Restart Site` button at the top
right.

![60-site-extension-step3-1.png](https://azurebatchshipyard.blob.core.windows.net/github/60-site-extension-step3-1.png)

You are now ready to run Batch Shipyard in your Azure Function App
environment.

**Note about installation failures:** If you receive a message such as
`Failed to install` then it's possible that the installation did not complete
in the time allotted (which can happen for App Service Plans which are
underpowered). To continue the installation, follow these steps:

1. Load Kudu, go to `Debug console` and select `CMD`
2. Navigate to `D:\home\SiteExtensions\BatchShipyard`
3. Run the command `install.cmd`. Let this command run to completion
(it may take a long time).
4. Restart your site.
5. To verify: Re-load Kudu, go back to the `Debug console`, select `CMD` and
then run `%BATCH_SHIPYARD_CMD% --version`. There will be a long delay, but you
should see the version in the output.

## Invoking Batch Shipyard in an Azure Function App
The Batch Shipyard site extension automatically attempts to find the version
of Python installed via the site extension and links the version found with
the Batch Shipyard invocation. As part of the site extension, a global
application environment variable is defined, `BATCH_SHIPYARD_CMD`, which
can be used in your `run.py` (or other function language trigger script).
If this environment variable is not available, then you may have skipped
the `Restart Site` step in Step 3 above.

For example, a python `run.py` trigger script to invoke would look
something like:

```python
import os
import subprocess

cmd = os.environ['BATCH_SHIPYARD_CMD']
stdout = subprocess.check_output(cmd)
print(stdout)
```

This, of course, does nothing but invoking `shipyard` via the
`BATCH_SHIPYARD_CMD` environment variable without any commands and parameters.
The `stdout` variable would contain only the help text.

You can also define Azure App Settings that populate environment variables
such as `SHIPYARD_CONFIGDIR`, `SHIPYARD_AAD_DIRECTORY_ID`, etc. such that
these variables are automatically populated for you when your trigger
executes.
