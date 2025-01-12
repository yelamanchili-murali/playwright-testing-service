# Microsoft Playwright Testing preview

Microsoft Playwright Testing is a fully managed service that uses the cloud to enable you to run Playwright tests with much higher parallelization across different operating system-browser combinations simultaneously. This means faster test runs with broader scenario coverage, which helps speed up delivery of features without sacrificing quality.  

Ready to get started? Jump into our [quickstart guide](#get-started)!

https://github.com/microsoft/playwright-testing-service/assets/12104064/29b969ec-7106-4407-a34a-4f04756d16f7

## Useful Links
- [Quickstart: Run end-to-end tests at scale](https://aka.ms/mpt/quickstart)
- [Quickstart: Set up continuous end-to-end testing across different browsers and operating systems](https://aka.ms/mpt/ci)
- [Explore features and benefits](https://aka.ms/mpt/about)
- [Documentation](https://aka.ms/mpt/docs) 
- [Pricing](https://aka.ms/mpt/pricing)
- [Share feedback](https://aka.ms/mpt/feedback)

## Get Started
Follow these steps to run your existing Playwright test suite with the service.

### Prerequisites

- An Azure account with an active subscription. If you don't have an Azure subscription, [create a free account](https://aka.ms/mpt/create-azure-subscription) before you begin.
- Your Azure account must be assigned the [Owner](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#owner), [Contributor](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#contributor), or one of the [classic administrator roles](https://learn.microsoft.com/en-us/azure/role-based-access-control/rbac-and-directory-admin-roles#classic-subscription-administrator-roles).

### Create a Workspace

1. Sign in to the [Playwright portal](https://aka.ms/mpt/portal) with your Azure account.

1. Create the Workspace.

    ![Create new workspace](https://github.com/microsoft/playwright-testing-service/assets/12104064/d571e86b-9d43-48ac-a2b7-63afb9bb86a8)

    |Field  |Description  |
    |---------|---------|
    |**Workspace Name** | A unique name to identify your workspace.<BR>The name can't contain special characters or whitespace. |
    |**Azure Subscription** | Select an Azure subscription where you want to create the workspace. |
    |**Region** | This is where test run data will be stored for your workspace. |

  > [!NOTE]
  > If you don't see this screen, select an existing workspace and go to the next section.

### Generate Access Token

1. In the [Playwright portal](https://aka.ms/mpt/portal), select **Generate token** to create the access token.

    ![Generate access token](https://github.com/microsoft/playwright-testing-service/assets/12104064/2368ad52-d919-4c8a-b916-bdfddbd7a396)

1. Copy the access token.

### Obtain region endpoint

1. In the [Playwright portal](https://aka.ms/mpt/portal), copy the command under **Add region endpoint in your set up**.

    ![Set workspace endpoint](https://github.com/microsoft/playwright-testing-service/assets/12104064/d81ca629-2b23-4d34-8b70-67b6f7061a83)

    The endpoint URL corresponds to the workspace region. You might see a different endpoint URL in the Playwright portal, depending on the region you selected when creating the workspace.

### Set up environment

Ensure that the `PLAYWRIGHT_SERVICE_ACCESS_TOKEN` and `PLAYWRIGHT_SERVICE_URL` that you obtained in previous steps are available in your environment.

We recommend using `dotenv` module to manage your environment. With `dotenv` you'll be using the `.env` file to define your environment variables.

> [!IMPORTANT]
> Don't forget to add `.env` file to your `.gitignore` file in order to not leak your secrets.

```sh
npm i --save-dev dotenv
```

`.env` file
```
PLAYWRIGHT_SERVICE_ACCESS_TOKEN=eyJh...
PLAYWRIGHT_SERVICE_URL=wss://westus3.api.playwright-int.io/api/authorize/connectSession
```

### Add service configuration

Add the service configuration to your project in the same location as your existing Playwright config file. Use [playwright.service.config.ts](https://aka.ms/mpt/service-config) as a starting point:

```js
// playwright.service.config.ts

import { defineConfig } from '@playwright/test';
import config from './playwright.config';
import dotenv from 'dotenv';

// Define environment on the dev box in .env file:
//  .env:
//    PLAYWRIGHT_SERVICE_ACCESS_KEY=XXX
//    PLAYWRIGHT_SERVICE_URL=XXX

// Define environment in your GitHub workflow spec.
//  env:
//    PLAYWRIGHT_SERVICE_ACCESS_KEY: ${{ secrets.PLAYWRIGHT_SERVICE_ACCESS_KEY }}
//    PLAYWRIGHT_SERVICE_URL: ${{ secrets.PLAYWRIGHT_SERVICE_URL }}
//    PLAYWRIGHT_SERVICE_RUN_ID: ${{ github.run_id }}-${{ github.run_attempt }}-${{ github.sha }}

dotenv.config();

// Name the test run if it's not named yet.
process.env.PLAYWRIGHT_SERVICE_RUN_ID = process.env.PLAYWRIGHT_SERVICE_RUN_ID || new Date().toISOString();

// Can be 'linux' or 'windows'.
const os = process.env.PLAYWRIGHT_SERVICE_OS || 'linux';

export default defineConfig(config, {
  // Define more generous timeout for the service operation if necessary.
  // timeout: 60000,
  // expect: {
  //   timeout: 10000,
  // },
  workers: 20,

  // Enable screenshot testing and configure directory with expectations.
  // https://learn.microsoft.com/azure/playwright-testing/how-to-configure-visual-comparisons
  ignoreSnapshots: false,
  snapshotPathTemplate: `{testDir}/__screenshots__/{testFilePath}/${os}/{arg}{ext}`,
  
  use: {
    // Specify the service endpoint.
    connectOptions: {
      wsEndpoint: `${process.env.PLAYWRIGHT_SERVICE_URL}?cap=${JSON.stringify({
        // Can be 'linux' or 'windows'.
        os,
        runId: process.env.PLAYWRIGHT_SERVICE_RUN_ID
      })}`,
      timeout: 30000,
      headers: {
        'x-mpt-access-key': process.env.PLAYWRIGHT_SERVICE_ACCESS_TOKEN!
      },
      // Allow service to access the localhost.
      exposeNetwork: '<loopback>'
    }
  }
});
```

The service configuration serves to:
- Direct and authenticate Playwright to the Microsoft Playwright Testing service.
- Override timeouts for service operations, if needed.

> [!NOTE]
> Make sure your project uses @playwright/test version 1.37 or above.

### Run the tests

Run Playwright tests against browsers managed by the service using the configuration you created above.

```sh
npx playwright test --config=playwright.service.config.ts --workers=20
```

## Next steps
- Run tests in a [CI/CD pipeline.](https://aka.ms/mpt/configure-pipeline)

- Learn how to [manage access](https://aka.ms/mpt/manage-access) to the created workspace.

- Experiment with different number of workers to [determine the optimal configuration of your test suite](https://aka.ms/mpt/parallelism).


## Contributing

This project welcomes contributions and suggestions. Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft
trademarks or logos is subject to and must follow
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos is subject to those third-party's policies.
