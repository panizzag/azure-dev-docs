---
title: Create a Terraform base template in Azure using Yeoman
description: Learn how to create a Terraform base template in Azure using Yeoman.
ms.topic: how-to
ms.date: 08/07/2021
ms.custom: devx-track-terraform
---

# Create a Terraform base template in Azure using Yeoman

In this article, you learn how to use the combination of [Terraform](/azure/terraform/) and [Yeoman](https://yeoman.io/). Terraform is a tool for creating infrastructure on Azure. Yeoman makes it easy to create Terraform modules.

In this article, you learn how to do the following tasks:
> [!div class="checklist"]

> * Create a base Terraform template using the Yeoman module generator.
> * Test the Terraform template using two different methods.
> * Run the Terraform module using a Docker file.
> * Run the Terraform module natively in Azure Cloud Shell.

## 1. Configure your environment

[!INCLUDE [open-source-devops-prereqs-azure-subscription.md](../includes/open-source-devops-prereqs-azure-subscription.md)]

[!INCLUDE [configure-terraform.md](includes/configure-terraform.md)]

- **Visual Studio Code**: [Download Visual Studio Code](https://code.visualstudio.com/download) for your platform.

- **Docker**: [Install Docker](https://www.docker.com/get-started) to run the module created by the Yeoman generator.

- **Go programming language**: [Install Go](https://golang.org/) as Yeoman-generated test cases are code using the Go language.

- **Nodejs:** [Install Node.js](https://nodejs.org/en/download/)

- **Install Yeoman:** Run the following command: `npm install -g yo`.

- **Yeoman template:** Run the following command to install the Yeoman template for Terraform module: `npm install -g generator-az-terra-module`.

## 2. Create directory for Yeoman-generated module

The Yeoman template generates files in the current directory. For this reason, you need to create a directory.

This empty directory is required to be put under $GOPATH/src. For more information about this path, see the article [Setting GOPATH](https://github.com/golang/go/wiki/SettingGOPATH).

1. Navigate to the parent directory from which to create a new directory.

1. Run the following command replacing the placeholder. For this example, a directory name of `GeneratorDocSample` is used.

    ```bash
    mkdir <new-directory-name>
    ```

    ![mkdir](media/create-a-base-template-using-yeoman/ymg-mkdir-GeneratorDocSample.png)

1. Navigate to the new directory:

    ```bash
    cd <new-directory-name>
    ```

    ![Navigate to your new directory](media/create-a-base-template-using-yeoman/ymg-cd-GeneratorDocSample.png)

## 3. Create base module template

1. Run the following command:

    ```bash
    yo az-terra-module
    ```

1. Follow the on-screen instructions to provide the following information:

    - **Terraform module project Name** - A value of `doc-sample-module` is used for the example.

        ![Project name](media/create-a-base-template-using-yeoman/ymg-project-name.png)


    - **Would you like to include the Docker image file?** - Enter `y`. If you enter `n`, the generated module code will support running only in native mode.

        ![Include Docker image file?](media/create-a-base-template-using-yeoman/ymg-include-docker-image-file.png) 

1. List the directory contents to view the resulting files that are created:

    ```bash
    ls
    ```

    ![List created files](media/create-a-base-template-using-yeoman/ymg-ls-GeneratorDocSample-files.png)

## 4. Review the generated module code

1. Launch Visual Studio Code

1. From the menu bar, select **File > Open Folder** and select the folder you created.

    ![Visual Studio Code](media/create-a-base-template-using-yeoman/ymg-open-in-vscode.png)

The following files were created by the Yeoman module generator:

- `main.tf` - Defines a module called `random-shuffle`. The input is a `string_list`. The output is the count of the permutations.
- `variables.tf` - Defines the input and output variables used by the module.
- `outputs.tf` - Defines what the module outputs. Here, it's the value returned by `random_shuffle`, which is a built-in, Terraform module.
- `Rakefile` - Defines the build steps. These steps include:
    - `build` - Validates the formatting of the main.tf file.
    - `unit` -  The generated module skeleton doesn't include code for a unit test. If you want to specify a unit test scenario, you would you add that code here.
    - `e2e` - Runs an end-to-end test of the module.
- `test`
    - Test cases are written in Go.
    - All codes in test are end-to-end tests.
    - End-to-end tests attempt to provision all of the items defined under `fixture`. The results in the `template_output.go` file are compared with the pre-defined expected values.
    - `Gopkg.lock` and `Gopkg.toml`: Defines the dependencies.

For more information about the Yeoman generator for Azure https://github.com/Azure/generator-az-terra-module, see the [Terratest documentation](https://terratest.gruntwork.io/docs/).

## 5. Test the Terraform module using a Docker file

This section shows how to test a Terraform module using a Docker file.

>[!NOTE]
>This example runs the module locally; not on Azure.

### Confirm Docker is installed and running

From a command prompt, enter `docker version`.

![Docker version](media/create-a-base-template-using-yeoman/ymg-docker-version.png)

The resulting output confirms that Docker is installed.

To confirm that Docker is actually running, enter `docker info`.

![Docker info](media/create-a-base-template-using-yeoman/ymg-docker-info.png)

### Set up a Docker container

1. From a command prompt, enter

    `docker build --build-arg BUILD_ARM_SUBSCRIPTION_ID= --build-arg BUILD_ARM_CLIENT_ID= --build-arg BUILD_ARM_CLIENT_SECRET= --build-arg BUILD_ARM_TENANT_ID= -t terra-mod-example .`.

    The message **Successfully built** will be displayed.

    ![Message indicating a successful build](media/create-a-base-template-using-yeoman/ymg-successfully-built.png)

1. From the command prompt, enter `docker image ls` to see your created module `terra-mod-example` listed.

    ![List containing the new module](media/create-a-base-template-using-yeoman/ymg-repository-results.png)

1. Enter `docker run -it terra-mod-example /bin/sh`. After running the `docker run` command, you're in the Docker environment. At that point, you can discover the file by using the `ls` command.

    ![File list in Docker](media/create-a-base-template-using-yeoman/ymg-list-docker-file.png)

### Build the module

1. Run the following command:

    ```bash
    bundle install
    ```

1. Run the following command:

    ```bash
    rake build
    ```

    ![Rake build](media/create-a-base-template-using-yeoman/ymg-rake-build.png)

### Run the end-to-end test

1. Run the following command:

    ```bash
    rake e2e
    ```

1. After a few moments, the **PASS** message will appear.

    ![PASS](media/create-a-base-template-using-yeoman/ymg-pass.png)

1. Enter `exit` to complete the test and exit the Docker environment.

## 6. Use Yeoman generator to create and test a module

In this section, the Yeoman generator is used to create and test a module in Cloud Shell. Using Cloud Shell instead of using a Docker file greatly simplifies the process. Using Cloud Shell, the following products are all pre-installed:

- Node.js
- Yeoman
- Terraform

### Start a Cloud Shell session

1. Start an Azure Cloud Shell session via either the [Azure portal](https://portal.azure.com/), [shell.azure.com](https://shell.azure.com), or the [Azure mobile app](https://azure.microsoft.com/features/azure-portal/mobile-app/).

1. **The Welcome to Azure Cloud Shell** page opens. Select **Bash (Linux)**.

    ![Welcome to Azure Cloud Shell](media/create-a-base-template-using-yeoman/ymg-welcome-to-azure-cloud-shell.png)

1. If you have not already set up an Azure storage account, the following screen appears. Select **Create storage**.

    ![You have no storage mounted](media/create-a-base-template-using-yeoman/ymg-you-have-no-storage-mounted.png)

1. Azure Cloud Shell launches in the shell you previously selected and displays information for the cloud drive it just created for you.

    ![Your cloud drive has been created](media/create-a-base-template-using-yeoman/ymg-your-cloud-drive-has-been-created-in.png)

### Prepare a directory to hold your Terraform module

1. At this point, Cloud Shell will have already configured GOPATH in your environment variables for you. To see the path, enter `go env`.

1. Create the $GOPATH directory, if one doesn't already exist: Enter `mkdir ~/go`.

1. Create a directory within the $GOPATH directory. This directory is used to hold the different project directories created in this example. 

    ```bash
    mkdir ~/go/src
    ```

1. Create a directory to hold your Terraform module replacing the placeholder. For this example, a directory name of `my-module-name` is used.

    ```bash
    mkdir ~/go/src/<your-module-name>
    ```

1. Navigate to your module directory: 

    ```bash
    cd ~/go/src/<your-module-name>
    ```

### Create and test your Terraform module

1. Run the following command and follow the instructions. When asked if you want to create the Docker files, you enter `N`.

    ```bash
    yo az-terra-module
    ```

1. Run the following command to install the dependencies:

    ```bash
    bundle install
    ```

1. Run the following command to build the module:

    ```bash
    rake build
    ```

    ![Rake build](media/create-a-base-template-using-yeoman/ymg-rake-build.png)

1. Run the following command to run the test:

    ```bash
    rake e2e
    ```

    ![Test-pass results](media/create-a-base-template-using-yeoman/ymg-pass.png)

## Troubleshoot Terraform on Azure

[Troubleshoot common problems when using Terraform on Azure](troubleshoot.md)

## Next steps

> [!div class="nextstepaction"]
> [Install and use the Azure Terraform Visual Studio Code extension](/azure/terraform/terraform-vscode-extension).
