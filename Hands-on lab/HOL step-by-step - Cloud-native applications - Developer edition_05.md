## Exercise 1: Create and run a Docker application

**Duration**: 40 minutes

In this exercise, you will take the starter files and run the node.js application as a Docker application. You will create a Dockerfile, build Docker images, and run containers to execute the application.

### Task 1: Test the application

The purpose of this task is to make sure you can run the application successfully before applying changes to run it as a Docker application.

1. From Azure Cloud Shell, connect to your build agent if you are not already connected. (If you need to reconnect, please review the instructions on previous page 4 "Before the HOL" document.)

   ![This screenshot of the console window shows the output from connecting to mongo.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1-step1.png?raw=true)

2. Type the following command to create a Docker network named `fabmedical`:

   ```
   docker network create fabmedical
   
   ```

3. Run an instance of mongodb to use for local testing.

   ```
   docker container run --name mongo --net fabmedical -p 27017:27017 -d mongo:4.0
   
   ```

4. Confirm that the mongo container is running and ready.

   ```
   docker container list
   docker container logs mongo
   
   ```

   ![In this screenshot of the console window, docker container list has been typed and run at the command prompt, and the “api” container is in the list. Below this the log output is shown.](media/Ex1-Task1.4.png "Docker container mongo logs")

5. Connect to the mongo instance using the mongo shell and test some basic commands:

   ```
   mongo
   show dbs
   quit()
   
   ```

   ![This screenshot of the console window shows the output from connecting to mongo.](media/Ex1-Task1.5.png "Connect to mongodb")

6. To initialize the local database with test content, first navigate to the content-init directory and run npm install.

      ```
      cd ~/Fabmedical/content-init
      npm install

      ```
  
   ![This screenshot of the console window shows the output from connecting to mongo.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1-stp6.png?raw=true "Connect to mongodb")

7. Initialize the database.

   ```
   nodejs server.js
   
   ```

   ![This screenshot of the console window shows output from running the database initialization.](media/Ex1-Task1.7.png "Run nodejs server.js")

8. Confirm that the database now contains test data.

   ```
   mongo
   show dbs
   use contentdb
   show collections
   db.speakers.find()
   db.sessions.find()
   quit()
   
   ```

   This should produce output similar to the following:

   ![This screenshot of the console window shows the data output.](media/Ex1-Task1.8.png "Show database records")

9. Now navigate to the `content-api` directory and run npm install.

   ```
   cd ../content-api
   npm install
   
   ```

10. Start the API as a background process. Press `ENTER` again to get to a command prompt for the next step.

    ```
    nodejs ./server.js &
    
    ```

    ![In this screenshot, nodejs ./server.js & has been typed and run at the command prompt, which starts the API as a background process.](media/image47.png "Start the mongodb in background")

11. Test the API using curl. You will request the speaker's content, and this will return a JSON result.

    ```
    curl http://localhost:3001/speakers
    
    ```

    ![In this screenshot, made a curl request to view speakers.](media/image47_1.png "Display speaker data")

12. Navigate to the web application directory, run `npm install` and `ng build`.

    ```
    cd ../content-web
    npm install
    ng build
    
    ```

    ![In this screenshot, after navigating to the web application directory, nodejs ./server.js & has been typed and run at the command prompt, which runs the application as a background process as well.](media/image48.png "Running web server")

13. You will need **build agent vm** public IP address, which you can get from **Environment Details** tab.

    ![Edit the app.js file in vim in the build machine to update the API URL.](media/copyip.png)   

14. From the cloud shell in the build machine edit the `app.js` file using vim.

       ```
       vim app.js

       ```
       
    Then press **_i_** to get into the edit mode, after that replace localhost with the **build machine public IP address**.

    ![Edit the app.js file in vim in the build machine to update the API URL.](media/image27.png "Edit the app.js")

    Then press **_ESC_**, write **_:wq_** to save you changes and close the file.

15. Now run the content-web application in the background.

    ```
    node ./app.js &
    
    ```

    Press `ENTER` again to get a command prompt for the next step.

16. Test the web application using curl. You will see HTML output returned without errors.

    ```
    curl http://localhost:3000
    
    ```
    
    ![Edit the app.js file in vim in the build machine to update the API URL.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1tsk1-step17.png?raw=true "Edit the app.js")

   > **Note**:  If you get no such file or directory error, run the below command and retry step 16
   ```
   ng build
   
   ```
  
17. Leave the application running for the next task.

18. If you received a JSON response to the /speakers content request and an HTML response from the web application, your environment is working as expected.

### Task 2: Browsing to the web application

In this task, you will browse to the web application for testing.

1. Test the web application from a browser. Navigate to the web application using your build agent IP address at port `3000`. Get the Build Agent VM Ip from **Environment Details** tab.

   ```
   http://[BUILDAGENTVMIP]:3000
   ```
   > EXAMPLE: ```http://13.68.113.176:3000```
   
   ![In the Virtual Machine blade, Overview is selected on the left and Public IP address 52.174.141.11 is highlighted on the right.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1tsk2-step4.png?raw=true "VM Public IP Address")

1. Select the Speakers and Sessions links in the header. You will see the pages display the HTML version of the JSON content you curled previously.

   ![In the Virtual Machine blade, Overview is selected on the left and Public IP address 52.174.141.11 is highlighted on the right.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1tsk2-step5.png?raw=true "VM Public IP Address")

1. Once you have verified the application is accessible through a browser, go to your cloud shell window and stop the running node processes.

   ```
   killall nodejs
   killall node
   
   ```
   > **Note**: If cloud shell gets stuck, follow the below steps

  - Open a new Azure Cloud Shell console. You can do this by selecting the Open new session button from the first console, or navigating to ```https://shell.azure.com``` and logging in with the same lab credentials.

  - Connect to build agent vm using the Command to Connect to Build Agent VM, which is given on lab environment details page.

  - Run the following command
   ```
   cd ~/Fabmedical/content-web
   ```

### Task 3: Download a Dockerfile

In this task, you will create a new Dockerfile that will be used to run the API application as a containerized application.

> **Note**: You will be working in a Linux VM without friendly editor tools. You must follow the steps very carefully to work with Vim for a few editing exercises if you are not already familiar with Vim.

1. From cloud shell, navigate to the `content-api` folder. List the files in the folder with this command. The output should look like the screenshot below.

   ```
   cd ../content-api
   ll
   
   ```

   ![In this screenshot of the console window, ll has been typed and run at the command prompt. The files in the folder are listed in the window. At this time, we are unable to capture all of the information in the window. Future versions of this course should address this.](media/image55.png "List the files")




2. Download a file named `Dockerfile` and varify it is downloaded by listing the contents of the folder again.

   ```
   wget https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Cloud-native-applications/fix/Hands-on%20lab/Dockerfile -P ~/Fabmedical/content-api
   ll
   
   ```
   
   ![In this screenshot of the console window, ll has been typed and run at the command prompt. The Dockerfile file is highlighted at the top of list.](media/image58.png "Highlight the Dockerfile")
   
3. Review the `Dockerfile` content.   

   ```
   cat Dockerfile
   ```
   > Content should be similar to following:
   
   ```
   FROM node:alpine AS base
   RUN apk -U add curl
   WORKDIR /usr/src/app
   EXPOSE 3001

   FROM node:argon AS build
   WORKDIR /usr/src/app

   # Install app dependencies
   COPY package.json /usr/src/app/
   RUN npm install

   # Bundle app source
   COPY . /usr/src/app

   FROM base AS final
   WORKDIR /usr/src/app
   COPY --from=build /usr/src/app .
   CMD [ "npm", "start" ]
   ```
   
4. This Dockerfile describes the following:

   - The base stage includes environment setup which we expect to change very rarely, if at all.

     - Creates a new Docker image from the base image node:alpine. This base image has node.js on it and is optimized for small size.

     - Add `curl` to the base image to support Docker health checks.

     - Creates a directory on the image where the application files can be copied.

     - Exposes application port `3001` to the container environment so that the application can be reached at port `3001`.

   - The build stage contains all the tools and intermediate files needed to create the application.

     - Creates a new Docker image from `node:argon`.

     - Creates a directory on the image where the application files can be copied.

     - Copies `package.json` to the working directory.

     - Runs npm install to initialize the node application environment.

     - Copies the source files for the application over to the image.

   - The final stage combines the base image with the build output from the build stage.

     - Sets the working directory to the application file location.

     - Copies the app files from the build stage.

     - Indicates the command to start the node application when the container is run.


### Task 4: Create Docker images

In this task, you will create Docker images for the application --- one for the API application and another for the web application. Each image will be created via Docker commands that rely on a Dockerfile.

1. From cloud shell connected to the build agent VM, type the following command to view any Docker images on the VM. The list will only contain the mongodb image downloaded earlier.

   ```bash
   docker image ls

   ```
   
   ![The node image (node) and your container image (content-api) are visible in this screenshot of the console window.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1tsk4-step1.png?raw=true "List Docker images")


2. From the content-api folder containing the API application files and the new Dockerfile you created, type the following command to create a Docker image for the API application. This command does the following:

   - Executes the Docker build command to produce the image

   - Tags the resulting image with the name `content-api` (-t)

   - The final dot (`.`) indicates to use the Dockerfile in this current directory context. By default, this file is expected to have the name `Dockerfile` (case sensitive).

   ```bash
   docker image build -t content-api .

   ```
   
   ![The node image (node) and your container image (content-api) are visible in this screenshot of the console window.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1tsk4-step2.png?raw=true "List Docker images")


3. Once the image is successfully built, run the Docker images listing command again. You will see several new images: the node images and your container image.

   ```bash
   docker image ls

   ```

   Notice the untagged image. This is the build stage which contains all the intermediate files not needed in your final image.

   ![The node image (node) and your container image (content-api) are visible in this screenshot of the console window.](media/image59.png "List Docker images")

4. Commit and push the new Dockerfile before continuing.

   ```bash
   git add .
   git commit -m "Added Dockerfile"
   git push

   ```

   Enter credentials if prompted.

5. Navigate to the content-web folder again and list the files. Note that this folder already has a Dockerfile.

   ```bash
   cd ../content-web
   ll

   ```

6. View the Dockerfile contents -- which are similar to the file you created previously in the API folder. Type the following command:

   ```bash
   cat Dockerfile

   ```

   > Notice that the `content-web` Dockerfile build stage includes additional tools for a front-end Angular application in addition to installing npm packages.

7. Type the following command to create a Docker image for the web application.

   ```bash
   docker image build -t content-web .

   ```
   
8. Navigate to the content-init folder again and list the files. Note that this folder already has a Dockerfile.

   ```bash
   cd ../content-init
   ll

   ```

9. View the Dockerfile contents -- which are similar to the file you created previously in the API folder. Type the following command:

   ```bash
   cat Dockerfile

   ```

10. Type the following command to create a Docker image for the init application.

      ```bash
      docker image build -t content-init .

      ```
      
     ![The node image (node) and your container image (content-api) are visible in this screenshot of the console window.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1tsk4-step10.png?raw=true "List Docker images")


11. When complete, you will see eight images now exist when you run the Docker images command.

   ```bash
   docker image ls

   ```
   ![Three images are now visible in this screenshot of the console window: content-init, content-web, content-api, and node.](media/vm-list-containers.PNG "View content images")

### Task 5: Run a containerized application

The web application container will be calling endpoints exposed by the API application container and the API application container will be communicating with mongodb. In this exercise, you will launch the images you created as containers on the same bridge network you created when starting mongodb.

1. Create and start the API application container with the following command. The command does the following:

   - Names the container `api` for later reference with Docker commands.

   - Instructs the Docker engine to use the `fabmedical` network.

   - Instructs the Docker engine to use port `3001` and map that to the internal container port `3001`.

   - Creates a container from the specified image, by its tag, such as `content-api`.

   ```bash
   docker container run --name api --net fabmedical -p 3001:3001 content-api

   ```
   ![Three images are now visible in this screenshot of the console window: content-init, content-web, content-api, and node.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1task5-step1.png?raw=true "View content images")
   
2. The `docker container run` command has failed because it is configured to connect to mongodb using a localhost URL. However, now that content-api is isolated in a separate container, it cannot access mongodb via localhost even when running on the same docker host. Instead, the API must use the bridge network to connect to mongodb.

   ```text
   > content-api@0.0.0 start
   > node ./server.js

   Listening on port 3001
   Could not connect to MongoDB!
   MongooseServerSelectionError: connect ECONNREFUSED 127.0.0.1:27017
   npm notice
   npm notice New patch version of npm available! 7.0.8 -> 7.0.13
   npm notice Changelog: <https://github.com/npm/cli/releases/tag/v7.0.13>
   npm notice Run `npm install -g npm@7.0.13` to update!
   npm notice
   npm ERR! code 255
   npm ERR! path /usr/src/app
   npm ERR! command failed
   npm ERR! command sh -c node ./server.js

   npm ERR! A complete log of this run can be found in:
   npm ERR!     /root/.npm/_logs/2020-11-23T03_04_12_948Z-debug.log
   ```

3. The content-api application allows an environment variable to configure the mongodb connection string. Remove the existing container, and then instruct the docker engine to set the environment variable by adding the `-e` switch to the `docker container run` command. Also, use the `-d` switch to run the api as a daemon.

   ```bash
   docker container rm api
   docker container run --name api --net fabmedical -p 3001:3001 -e MONGODB_CONNECTION=mongodb://mongo:27017/contentdb -d content-api

   ```
   ![In this screenshot of the console window, docker container ls has again been typed and run at the command prompt. 0.0.0.0:32768->3000/tcp is highlighted under Ports.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1task5-step3.png?raw=true "List Docker containers")

4. Enter the command to show running containers. You will observe that the `api` container is in the list. Use the docker logs command to see that the API application has connected to mongodb.

   ```bash
   docker container ls
   docker container logs api

   ```

   ![In this screenshot of the console window, docker container ls has been typed and run at the command prompt, and the "api" container is in the list with the following values for Container ID, Image, Command, Created, Status, Ports, and Names: 458d47f2aaf1, content-api, "docker-entrypoint.s...", 37 seconds ago, Up 36 seconds, 0.0.0.0:3001->3001/tcp, and api.](media/image61v2.png "List Docker containers")

5. Test the API by curling the URL. You will see JSON output as you did when testing previously.

   ```bash
   curl http://localhost:3001/speakers

   ```
   
6. Create and start the web application container with a similar `docker container run` command -- instruct the docker engine to use any port with the `-P` command.

   ```bash
   docker container run --name web --net fabmedical -P -d content-web

   ```
   ![In this screenshot of the console window, docker container ls has again been typed and run at the command prompt. 0.0.0.0:32768->3000/tcp is highlighted under Ports.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1task5-step6.png?raw=true "List Docker containers")

7. Enter the command to show running containers again, and you will observe that both the API and web containers are in the list. The web container shows a dynamically assigned port mapping to its internal container port `3000`.

   ```bash
   docker container ls

   ```
   ![In this screenshot of the console window, docker container ls has again been typed and run at the command prompt. 0.0.0.0:32768->3000/tcp is highlighted under Ports.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1task5-step7.png?raw=true "List Docker containers")

8. Test the web application by fetching the URL with curl. For the port, use the dynamically assigned port, which you can find in the output from the previous command. You will see HTML output, as you did when testing previously.

   ```bash
   curl http://localhost:[PORT]/speakers.html
   ```
    ![In this screenshot of the console window, docker container ls has again been typed and run at the command prompt. 0.0.0.0:32768->3000/tcp is highlighted under Ports.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1task5-step8.png?raw=true "List Docker containers")

### Task 6: Setup environment variables

In this task, you will configure the `web` container to communicate with the API container using an environment variable, similar to the way the mongodb connection string is provided to the api.

1. From cloud shell connected to the build agent VM, stop and remove the web container using the following commands.

   ```bash
   docker container stop web
   docker container rm web

   ```

2. Validate that the web container is no longer running or present by using the `-a` flag as shown in this command. You will see that the `web` container is no longer listed.

   ```bash
   docker container ls -a

   ```

3. Review the `app.js` file.

   ```bash
   cd ../content-web
   cat app.js
   
   ```

4. Observe the output of above command: Observe that the `contentApiUrl` variable can be set with an environment variable.

   ```javascript
   const contentApiUrl = process.env.CONTENT_API_URL || "http://Build-Machine-IP:3001";
   ```
   ![In this screenshot of Dockerfile, the CONTENT_API_URL code appears above the next Dockerfile line, which reads EXPOSE 3000.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1task6-step4.png?raw=true "Set ENV variable")
  
5. Open the Dockerfile for editing using Vim and press the `i` key to go into edit mode.

   ```bash
   vi Dockerfile
   <i>
   ```

6. Locate the `EXPOSE` line shown below and add a line above it that sets the default value for the environment variable, as shown in the screenshot.

   ```Dockerfile
   ENV CONTENT_API_URL http://localhost:3001
   ```
   ![In this screenshot of Dockerfile, the CONTENT_API_URL code appears above the next Dockerfile line, which reads EXPOSE 3000.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1task6-step6.png?raw=true "Set ENV variable")

7. Press the Escape key and type `:wq` and then press the Enter key to save and close the file.

   ```text
   <Esc>
   :wq
   <Enter>
   ```

8. Rebuild the web application Docker image using the same command as you did previously.

   ```bash
   docker image build -t content-web .

   ```

9. Create and start the image passing the correct URI to the API container as an environment variable. This variable will address the API application using its container name over the Docker network you created. After running the container, check to see the container is running and note the dynamic port assignment for the next step.

   ```bash
   docker container run --name web --net fabmedical -P -d -e CONTENT_API_URL=http://api:3001 content-web
   docker container ls

   ```
   
   ![In this screenshot of Dockerfile, the CONTENT_API_URL code appears above the next Dockerfile line, which reads EXPOSE 3000.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1task6-step9.png?raw=true "Set ENV variable")

10. Curl the speakers path again, using the port assigned to the web container. Again, you will see HTML returned, but because curl does not process javascript, you cannot determine if the web application is communicating with the api application. You must verify this connection in a browser.

    ```bash
    curl http://localhost:[PORT]/speakers.html
    ```
    
    ![In this screenshot of Dockerfile, the CONTENT_API_URL code appears above the next Dockerfile line, which reads EXPOSE 3000.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1task6-step10.png?raw=true "Set ENV variable")
   
11. You will not be able to browse to the web application on the ephemeral port because the VM only exposes a limited port range. Now you will stop the web container and restart it using port `3000` to test in the browser. Type the following commands to stop the container, remove it, and run it again using explicit settings for the port.

    ```bash
    docker container stop web
    docker container rm web
    docker container run --name web --net fabmedical -p 3000:3000 -d -e CONTENT_API_URL=http://api:3001 content-web

    ```

12. Curl the speaker path again, using port `3000`. You will see the same HTML returned.

    ```bash
    curl http://localhost:3000/speakers.html

    ```

13. You can now use a web browser to navigate to the website and successfully view the application at port `3000`. Replace `[BUILDAGENTIP]` with the **IP address** you used previously.

    ```bash
    http://[BUILDAGENTIP]:3000

    EXAMPLE: http://13.68.113.176:3000
    ```

14. Commit your changes and push to the repository.

    ```bash
    git add .
    git commit -m "Setup Environment Variables"
    git push

    ```

    Enter credentials if prompted.

### Task 7: Push images to Azure Container Registry

To run containers in a remote environment, you will typically push images to a Docker registry, where you can store and distribute images. Each service will have a repository that can be pushed to and pulled from with Docker commands. Azure Container Registry (ACR) is a managed private Docker registry service based on Docker Registry v2.

In this task, you will push images to your ACR account, version images with tagging, and setup continuous integration (CI) to build future versions of your containers and push them to ACR automatically.

1. In the Azure Portal `(https://portal.azure.com/)`, under navigate select **Resource groups** and click on **fabmedical-{DeploymentID}** resource group. Now in the resource group page select Azure container registry 
 
   ![This is a screenshot of the selecting resource group.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/media/resourcegroup.png?raw=true "Cloud Shell Bash Window") 
 
   ![This is a screenshot of acr.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/media/acr.png?raw=true "Cloud Shell Bash Window") 
 

2. Select **Access keys** under **Settings** on the left-hand menu.

   ![In this screenshot of the left-hand menu, Access keys is highlighted below Settings.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1task7-step2.png?raw=true "Access keys")

3. The Access keys blade displays the Login server, username, and password that will be required for the next step. Make note of this values, you will be using it in the next part of lab

   > **Note**: If the username and password do not appear, select Enable on the Admin user option.

   ![This is a screenshot of acr.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1task7-step3.png?raw=true "Cloud Shell Bash Window")
   
4. From the cloud shell session connected to your build VM, login to your ACR account by typing the following command. Follow the instructions to complete the login.

   ```bash
   docker login [LOGINSERVER] -u [USERNAME] -p [PASSWORD]
   ```

   For example:

   ```bash
   docker login acr289069.azurecr.io -u acr289069 -p +W/j=l+Fcze=n07SchxvGSlvsLRh/7ga
   ```

   ![In this screenshot of the console window, the following has been typed and run at the command prompt: docker login fabmedicalsoll.azurecr.io](media/image65.png "Docker log into container")

   > **Tip**: Make sure to specify the fully qualified registry login server (all lowercase).

5. Run the following commands to properly tag your images to match your ACR account name.

   ```bash
   docker image tag content-web [LOGINSERVER]/content-web
   docker image tag content-api [LOGINSERVER]/content-api
   ```

   > **Note**: Be sure to replace the `[LOGINSERVER]` of your ACR instance.

6. List your docker images and look at the repository and tag. Note that the repository is prefixed with your ACR login server name, such as the sample shown in the screenshot below.

   ```bash
   docker image ls

   ```

   ![This is a screenshot of a docker images list example.](media/vm-docker-images-list.PNG "Docker image list")

7. Push the images to your ACR account with the following command:

   ```bash
   docker image push [LOGINSERVER]/content-web
   docker image push [LOGINSERVER]/content-api
   ```

   ![In this screenshot of the console window, an example of images being pushed to an ACR account results from typing and running the following at the command prompt: docker push [LOGINSERVER]/content-web.](media/image67.png "Push image to ACR")

8. In the Azure Portal, navigate to your ACR account, and select **Repositories** under **Services** on the left-hand menu. You will now see two, one for each image.

   ![In this screenshot, content-api and content-web each appear on their own lines below Repositories.](media/image68spk.png "Search for repositories")

9. Select `content-api`. You will see the latest tag is assigned.

   ![In this screenshot, content-api is selected under Repositories, and the Tags blade appears on the right.](media/image69spk.png "View latest repo tags")

10. From the cloud shell session attached to the VM, assign the `v1` tag to each image with the following commands. Then list the Docker images to note that there are now two entries for each image: showing the `latest` tag and the `v1` tag. Also note that the image ID is the same for the two entries, as there is only one copy of the image.

    ```bash
    docker image tag [LOGINSERVER]/content-web:latest [LOGINSERVER]/content-web:v1
    docker image tag [LOGINSERVER]/content-api:latest [LOGINSERVER]/content-api:v1
    docker image ls
    ```

    ![In this screenshot of the console window is an example of tags being added and displayed.](media/image70.png "View latest image by tag")

11. Push the images to your ACR account with the following command:

    ```bash
    docker image push [LOGINSERVER]/content-web:v1
    docker image push [LOGINSERVER]/content-api:v1
    ```

12. Refresh one of the repositories to see the two versions of the image now appear.

    ![In this screenshot, content-api is selected under Repositories, and the Tags blade appears on the right. In the Tags blade, latest and v1 appear under Tags.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/ex1task7-step12.png?raw=true "View two versions of image")

13. Run the following commands to pull an image from the repository. Note that the default behavior is to pull images tagged with `latest`. You can pull a specific version using the version tag. Also, note that since the images already exist on the build agent, nothing is downloaded.

    ```bash
    docker image pull [LOGINSERVER]/content-web
    docker image pull [LOGINSERVER]/content-web:v1
    ```

### Task 8: Setup CI Pipeline to Push Images

In this task, you will use YAML to define a GitHub Actions workflow that builds your Docker
image and pushes it to your ACR instance automatically.

1. In GitHub, return to the **Fabmedical** repository screen, and select the **Settings** tab.

2. From the left menu, select **Secrets**.

3. Select the **New repository secret** button.

    ![Settings link, Secrets link, and New secret button are highlighted.](media/2020-08-24-21-45-42.png "GitHub Repository secrets")

4. In the **New secret** form, enter the name `ACR_USERNAME` and for the value, paste in the Azure Container Registry **Username** that was copied previously. Select **Add secret**.

    ![New secret screen with values are entered.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/media/2020-08-24-21-48-54.png?raw=true "New secret screen")

5. Add another Secret, by entering the name `ACR_PASSWORD` and for the value, paste in the Azure Container Registry **Password** that was copied previously.

    ![Secrets screen with both the ACR_USERNAME and ACR_PASSWORD secrets created.](media/2020-08-24-21-51-24.png "Secrets screen")

6. In your Azure Cloud Shell session connected to the build agent VM, navigate to the `~/Fabmedical` directory:

   ```bash
   cd ~/Fabmedical

   ```

7. Before the GitHub Actions workflows can be setup, the `.github/workflows` directory needs to be created again and download workflow yml files. Do this by running the following commands:

    ```
    rm -rf ~/Fabmedical/.github/workflows/
    mkdir ~/Fabmedical/.github/workflows/
    cd ~/Fabmedical/.github/workflows/
    wget http://bit.ly/hol-content-web -O content-web.yml
    wget http://bit.ly/hol-content-api -O content-api.yml
    wget http://bit.ly/hol-content-init -O content-init.yml

    ```

9. Next edit the workflow YAML file.

    ```dotnetcli
    vi content-web.yml

    ```

   - replace `[DeploymentID]` with your DeploymentID value given on Environment details page

       ```
       # Environment variables are defined so that they can be used throughout the job definitions.
       env:
        imageRepository: 'content-web'
        resourceGroupName: 'fabmedical-[DeploymentID]'
        containerRegistryName: 'acr[DeploymentID]'
        containerRegistry: 'acr[DeploymentID].azurecr.io'
        dockerfilePath: './content-web'
        tag: '${{ github.run_id  }}'

       ```
 
    ![The content-web Action is shown with the Actions, content-web, and Run workflow links highlighted.](media/update-web-yml.png "content-web workflow")

10. Save the file and exit VI by pressing `<Esc>` then `:wq`.

11. Save the pipeline YAML, then commit and push it to the Git repository:

    ```bash
    git add .
    git commit -m "Added workflow YAML"
    git push

    ```

12. In GitHub, return to the **Fabmedical** repository screen, and select the **Actions** tab.

13. On the **Actions** page, select the **content-web** workflow.

14. On the **content-web** workflow, select **Run workflow** and manually trigger the workflow to execute.

    ![The content-web Action is shown with the Actions, content-web, and Run workflow links highlighted.](media/2020-08-25-15-38-06.png "content-web workflow")

15. After a second, the newly triggered workflow execution will display in the list. Select the new **content-web** execution to view its status.

16. Selecting the **Build and Push Docker Image** job of the workflow will display its execution status.

    ![Build and Push Docker Image job.](media/2020-08-25-15-42-11.png "Build and Push Docker Image job")

17. Next, setup the `content-api` workflow. 

    ```
    cd ~/Fabmedical/.github/workflows
    vi content-api.yml
    
    ```

18. Edit the `resourceGroupName` by replacing the `[DeploymentID]` with your ```DeploymentID``` value from ```Environment Details``` tab.

    ![The screenshot shows the content-api.yml with the environment variables highlighted.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/2020-08-25-15-59-56-1.png?raw=true "content-api.yml environment variables highlighted")

19. Save the file, then commit and push it to the Git repository:
    ```bash
    git add .
    git commit -m "Updated workflow YAML"
    git push
    
    ```
20. Now navigate to the repositories in GitHub, select Actions, and then manually run the content-api workflow.
