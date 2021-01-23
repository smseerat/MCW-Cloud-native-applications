## Before the hands-on lab

**Duration**: 60 minutes

You should follow all of the steps provided in this section _before_ taking part in the hands-on lab ahead of time as some of these steps take time.

### Task 1: Setup Azure Cloud Shell
1. In the **JumpVM** provided to you on the left side, double click on the **Azure Portal** browser shortcut on the desktop.
 
   ![](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/media/azureportal.png?raw=true "Cloud Shell Bash Window")
   
1. Click on **Get Started** on edge browser.

   ![](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/media/getstarted.png?raw=true)
   
1. Now, click on the cross icon to close **Let's set up your new tab page**.

   ![](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/media/closeletus.png?raw=true)
   
1. Close the current tab by clicking on close tab icon and then **Sign in Microsoft Azure** tab will open.
 
   ![](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/media/closeedgetab.png?raw=true)
 
1. On **Sign in to Microsoft Azure** tab, **Sign in** with following Azure credentials.
 
     * Azure Usename/Email: <inject key="AzureAdUserEmail"></inject> 
 
     * Azure Password: <inject key="AzureAdUserPassword"></inject> 
 
1. If you see the pop-up **Stay Signed in?**, click Yes. 
 
1. If you see the pop-up **You have free Azure Advisor recommendations!**, close the window to continue the lab. 
 
1. If a **Welcome to Microsoft Azure** popup window appears, click **Maybe Later** to skip the tour.

1. In the **Azure portal**, open the **Azure Cloud Shell** by clicking on the cloud shell icon in the top menu bar. Alternatively, you can open cloud shell by navigating to ```https://shell.azure.com```.

   ![The cloud shell icon is highlighted on the menu bar.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/media/b4-image35.png?raw=true "Cloud Shell")

1. After launching the Azure Cloud Shell, select the **Bash** option. Now, on **You have no storage mounted** dialog box click on **Show advanced settings**. Select Create new under Storage account and provide values as below: 
  
      - **Storage account** : **storage{Deployementid}**
      - **File Share** : **blob**
  
    > **Note**: Storage account name should be always unique, you can get the Deployement Id  from the **Environment Details** tab. 

   ![This is a screenshot of the cloud shell opened in a browser window. Bash was selected.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/media/b4-image36.png?raw=true "Cloud Shell Bash Window")

### Task 2: Download Starter Files

In this task, you use `git` to copy the lab content to your cloud shell so that the lab starter files will be available.

1. Copy the following command to clone the lab files inside clouddrive directory and then delete **.git** directory, becouse **.git** directory is not required. Paste in cloudshell and then press `<ENTER>`:

     ```
     cd clouddrive
     git clone https://github.com/microsoft/MCW-Cloud-native-applications.git
     rm -rf MCW-Cloud-native-applications/.git

     ```

   ![The cloud shell icon is highlighted on the menu bar.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/media/clonefiles.png?raw=true "Cloud Shell")

### Task 3: Create a GitHub repository

FabMedical has provided starter files for you. They have taken a copy of the websites for their customer Contoso Neuro and refactored it from a single node.js site into a website with a content API that serves up the speakers and sessions. This refactored code is a starting point to validate the containerization of their websites. Use this to help them complete a POC that validates the development workflow for running the website and API as Docker containers and managing them within the Azure Kubernetes Service environment.

1. In a new browser tab and open ```https://www.github.com``` and Log in with your personal GitHub account.

    > **Note** : You have to use your own GitHub account. If you don't have a GitHub account then navigate to the following link ```https://github.com/join ``` and create one.

1. In the upper-right corner, expand the user drop down menu and select **Your repositories**.

    ![The user menu is expanded with the Your repositories item selected.](media/2020-08-23-18-03-40.png "User menu, your repositories")

1. Next to the search criteria, locate and select the **New** button.

    ![The GitHub Find a repository search criteria is shown with the New button selected.](media/2020-08-23-18-08-02.png "New repository button")

1. On the **Create a new repository** screen, name the repository ```Fabmedical``` and select the **Create repository** button.

    ![Create a new repository page with Repository name field and Create repository button highlighted.](media/2020-08-23-18-11-38.png "Create a new repository")

1. On the **Quick setup** screen, copy the **HTTPS** GitHub URL for your new repository, paste this in notepad for future use.

    ![Quick setup screen is displayed with the copy button next to the GitHub URL textbox selected.](media/2020-08-23-18-15-45.png "Quick setup screen")

1. Now, go back to the cloudshell window, if you have closed that open a **new** Azure Cloud Shell console.  You can do this by selecting the **Open new session** button from the first console, or navigating to ```https://shell.azure.com``` and logging in with the same lab credentials.

1. Navigate to the **FabMedical** source code folder and list the contents.

    ```
    cd ~/clouddrive/MCW-Cloud-native-applications/Hands-on\ lab/lab-files/developer/
    ls

    ```

1. You'll see the listing includes three folders, one for the web site, another for the content API and one to initialize API data:

    ```
    content-api/
    content-init/
    content-web/
    ```

1. Set your username and email, which git uses for commits, run below commands
  
    ```
    git config --global user.email "you@example.com"
    git config --global user.name "Your Name"
    ```

1. Using the Cloud Shell, initialize a new git repository:

    ```
    git init
    git add .
    git commit -m "Initial Commit"
    
    ```

11. Set the remote origin to the GitHub URL by issuing the following command, replace ```<your_github_username>``` with your Github username.

    ```
    git remote add origin https://github.com/<your_github_username>/Fabmedical
    
    ```

12. Configure git CLI to cache your credentials, so that you don't have to keep re-typing them.

    ```
    git config --global --unset credential.helper
    git config --global credential.helper store
    
    ```

13. Push to the master branch by issuing the following command:

    ```
    git push -u origin master
    
    ```

    > **Note**: If you have multi-factor authentication, you will need to create a personal access token when using the cloud shell. Reference the following link for help with setting up a GitHub personal access token to use for authenticating `git` with your GitHub account: ```https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token ```.

    > **Note**: Once you have your personal access token, retry the above command, use your token as the password.

14. Refresh your GitHub repository, you should now see the code published.

### Task 4: Connect securely to the build agent

In this section, you validate that you can connect to the new build agent VM.

1. Connect to build agent vm using the **Command to Connect to Build Agent VM**, which is given on lab environment details page.

2. When asked to confirm if you want to continue connecting, type `yes`.

3. When asked for the password, enter **Build Agent VM Password** given below.
    ```
    Password.1!!
    ```

4. SSH connects to the VM and displays a command prompt such as the following. Keep this cloud shell window open for the next step:

   `adminfabmedical@fabmedical-SUFFIX:~$`

   ![In this screenshot of a Cloud Shell window, ssh -i .ssh/fabmedical adminfabmedical@52.174.141.11 has been typed and run at the command prompt. The information detailed above appears in the window.](media/b4-image27.png "Azure Cloud Shell Connect to Host")


### Task 5: Clone Repositories to the Build Agent

In this task, you clone your repositories from GitHub so you can work with them on the build agent.

1. As you previously did in cloud shell, set your username and email which are used for git commits.

   ```bash
   git config --global user.email "you@example.com"
   git config --global user.name "Your Name"
   ```

   > **Note**: In some cases, the `root` user owns your user's `.config` folder. If this happens, run the following command to return ownership to `adminfabmedical` and then try the `git` command again:
   >
   > ```bash
   > sudo chown -R $USER:$(id -gn $USER) /home/adminfabmedical/.config
   > ```

2. Configure git CLI to cache your credentials, so that you don't have to keep
   re-typing them.

   ```bash
   git config --global credential.helper cache
   ```

   > **Note**: In some cases, the `root` user owns your user's `.config` folder. If this happens, run the following command to return ownership to `adminfabmedical` and then try the `git` command again:
   >
   > ```bash
   > sudo chown -R $USER:$(id -gn $USER) /home/adminfabmedical/.config
   > ```

3. Use the GitHub URL to clone the repository code to your build agent machine.

   ```bash
   git clone <GITHUB_REPOSITORY_URL>
   ```

   > **Note**: In some cases, the `root` user owns your user's `.config` folder. If this happens, run the following command to return ownership to `adminfabmedical` and then try the `git` command again:
   >
   > ```bash
   > sudo chown -R $USER:$(id -gn $USER) /home/adminfabmedical/.config
   > ```


