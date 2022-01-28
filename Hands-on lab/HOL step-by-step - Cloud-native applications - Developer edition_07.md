## Exercise 2: Deploy the solution to Azure Kubernetes Service

**Duration**: 60 minutes

In this exercise, you will connect to the Azure Kubernetes Service cluster you created before the hands-on lab and deploy the Docker application to the cluster using Kubernetes.

### Task 1: Tunnel into the Azure Kubernetes Service cluster

In this task, you will gather the information you need about your Azure Kubernetes Service cluster to connect to the cluster and execute commands to connect to the Kubernetes management dashboard from cloud shell.

> **Note**: The following tasks should be executed in cloud shell and not the build machine, so disconnect from build machine if still connected. Please open a **new Azure cloud shell** and perform the steps as **odl_user**.

   ![](media/cna15.png)

1. Verify that you are connected to the correct subscription with the following command to show your default subscription:

   ```bash
   az account show
   ```

   - If you are not connected to the correct subscription, list your subscriptions and then set the subscription by its id with the following commands (similar to what you did in cloud shell before the lab):

   ```bash
   az account list
   az account set --subscription {id}
   ```

2. Configure kubectl to connect to the Kubernetes cluster:

   ```bash
   az aks get-credentials -a --name fabmedical-SUFFIX --resource-group fabmedical-SUFFIX
   ```

3. Test that the configuration is correct by running a simple kubectl command to produce a list of nodes:

   ```bash
   kubectl get nodes
   ```

   ![In this screenshot of the console, kubectl get nodes has been typed and run at the command prompt, which produces a list of nodes.](media/image75.png "kubectl get nodes")

### Task 2: Deploy a service using the Azure Portal

In this task, you will deploy the API application to the Azure Kubernetes Service cluster using the Azure Portal.

1. From the Azure Portal, select the resource group named fabmedical-DeploymentId, and then select your Kubernetes Service Azure resource.

   ![In this screenshot, the resource group was previously selected and the AKS cluster is selected.](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Cloud-native-applications/fix/Hands-on%20lab/local/ex3tsk7-step1.png "Select fabmedical resource group")

2. We first need to define a Service for our API so that the application is accessible within the cluster. In the AKS blade select **Services and ingresses** and on the Services tab select **+ Add**.

    ![This is a screenshot of the Azure Portal for AKS showing adding a Service.](media/2021-03-25-17-04-04.png "Add a Service")

3. In the **Add with YAML** screen, paste following YAML and choose **Add**.

   ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        app: api
      name: api
    spec:
      ports:
        - name: api-traffic
          port: 3001
          protocol: TCP
          targetPort: 3001
      selector:
        app: api
      sessionAffinity: None
      type: ClusterIP
   ```

4. Now select **Workloads** under the **Kubernetes resources** section in the left navigation.

    ![Select workloads under Kubernetes resources.](media/2021-03-25-17-04-35.png "Select workloads under Kubernetes resources")

5. From the Workloads view, with **Deployments** selected (the default) then select **+ Add**.

   ![Selecting + Add to create a deployment.](media/2021-03-25-17-05-05.png "Selecing + Add to create a deployment")

6. In the **Add with YAML** screen that loads paste the following YAML and update the `[LOGINSERVER]` placeholder with the name of the ACR instance.

   ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: api
      name: api
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: api
      strategy:
        rollingUpdate:
        maxSurge: 1
        maxUnavailable: 1
        type: RollingUpdate
      template:
        metadata:
          labels:
            app: api
            name: api
        spec:
          containers:
            - name: api
              image: [LOGINSERVER].azurecr.io/content-api
              imagePullPolicy: Always
              livenessProbe:
                httpGet:
                  path: /
                  port: 3001
                initialDelaySeconds: 30
                periodSeconds: 20
                timeoutSeconds: 10
                failureThreshold: 3
              ports:
                - containerPort: 3001
                  hostPort: 3001
                  protocol: TCP
              resources:
                requests:
                  cpu: 1
                  memory: 128Mi
              securityContext:
                privileged: false
                terminationMessagePath: /dev/termination-log
                terminationMessagePolicy: File
                dnsPolicy: ClusterFirst
                restartPolicy: Always
                schedulerName: default-scheduler
                securityContext: {}
                terminationGracePeriodSeconds: 30
   ```

7. Select **Add** to initiate the deployment. This can take a few minutes after which you will see the deployment listed.

   ![Service is showing as unhealthy](media/2021-03-25-17-05-36.png "Service is showing as unhealthy")

8. Select the **api** deployment to open the Deployment, select **Live logs** and then a Pod from the drop-down. After a few moments, the live logs should appear.

   ![Service is showing as unhealthy](media/2021-03-25-17-06-09.png "Service is showing as unhealthy")

   > **Note:** if the logs don't display it may be the Pod no longer exists. You can use the **View in Log Analytics** to view historical logs regardless of Pod.

9. If you scroll through the log you can see it indicates that the content-api application is once again failing because it cannot find a MongoDB api to communicate with. You will resolve this issue by connecting to Cosmos DB.

   ![This screenshot of the Kubernetes management dashboard shows logs output for the api container.](media/2021-03-25-17-07-13.png "MongoDB communication error")

10. In the Azure Portal navigate to your resource group and find your Azure Cosmos DB API for MongoDB account. Select the Cosmos DB resource to view details.

    ![This is a screenshot of the Azure Portal showing the Cosmos DB among existing resources.](media/cna17.png "Select CosmosDB resource from list")

11. Under **Quick Start** select the **Node.js** tab and copy the **Node.js 3.0 connection string**.

    ![This is a screenshot of the Azure Portal showing the quick start for setting up Cosmos DB with MongoDB API. The copy button is highlighted.](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Cloud-native-applications/fix/Hands-on%20lab/local/ex3tsk2-step8.png "Capture CosmosDB connection string")

12. Modify the copied connection string by adding the database `contentdb` to the URL, along with a replicaSet of `globaldb`. The resulting connection string should look like the below sample. Note that you may need to modify the endpoint URL.

    ```text
    mongodb://<USERNAME>:<PASSWORD>@fabmedical-<SUFFIX>.mongo.cosmos.azure.com:10255/contentdb?ssl=true&replicaSet=globaldb
    ```
    
    ![This is a screenshot of the Azure cloud shell window showing the command to create the base64 encoded secret.  The output to copy is highlighted.](https://github.com/CloudLabs-MCW/MCW-Cloud-native-applications/blob/fix/Hands-on%20lab/local/connnectionstring.png?raw=true "Show encoded secret")

13. You will setup a Kubernetes secret to store the connection string and configure the `content-api` application to access the secret. First, you must base64 encode the secret value. Open your Azure Cloud Shell window and use the following command to encode the connection string and then, copy the output.

    > **Note**: Double quote marks surrounding the connection string are required to successfully produce the required output.

    ```bash
    echo -n "[CONNECTION STRING VALUE]" | base64 -w 0 - | echo $(</dev/stdin)
    ```

    ![This is a screenshot of the Azure cloud shell window showing the command to create the base64 encoded secret.  The output to copy is highlighted.](media/hol-2019-10-18_07-12-13.png "Show encoded secret")

14. Return to the AKS blade in the Azure Portal and select **Configuration** under the **Kubernetes resources** section. Select **Secrets** and choose **+ Add**.

15. In the **Add with YAML** screen, paste following YAML and replace the placeholder with the encoded connection string from your clipboard and choose **Add**. Note that YAML is position sensitive so you must ensure indentation is correct when typing or pasting.

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: cosmosdb
    type: Opaque
    data:
      db: <base64 encoded value>
    ```

    ![This is a screenshot of the Azure Portal for AKS howing the YAML file for creating a deployment.](media/2021-03-25-17-08-06.png "Upload YAML data")

16. Sort the Secrets list by name and you should now see your new secret displayed.

    ![This is a screenshot of the Azure Portal for AKS showing secrets.](media/find-cosmosdb-secret.png "Manage Kubernetes secrets")

17. View the details for the **cosmosdb** secret by selected it in the list.

    ![This is a screenshot of the Azure Portal for AKS showing the value of a secret.](media/2021-03-25-17-08-54.png "View cosmosdb secret")

18. Next, download the api deployment configuration using the following command in your Azure Cloud Shell window:

    ```bash
    kubectl get -o=yaml deployment api > api.deployment.yml
    ```

19. Edit the downloaded file using cloud shell code editor:

    ```bash
    code api.deployment.yml
    ```

    Add the following environment configuration to the container spec, below the `image` property:

    ```yaml
      env:
      - name: MONGODB_CONNECTION
        valueFrom:
          secretKeyRef:
            name: cosmosdb
            key: db
    ```

    ![This is a screenshot of the Kubernetes management dashboard showing part of the deployment file.](media/Ex2-Task1.17.png "Edit the api.deployment.yml file")

20. Save your changes and close the editor.

    ![This is a screenshot of the code editor save and close actions.](media/Ex2-Task1.17.1.png "Code editor configuration update")

21. Update the api deployment by using `kubectl` to deploy the API.

    ```bash
    kubectl delete deployment api
    kubectl create -f api.deployment.yml
    ```

22. In the Azure Portal return to Live logs (see Step 5). The last log should show as connected to MongoDB.

    ![This is a screenshot of the Kubernetes management dashboard showing logs output.](media/2021-03-25-17-09-24.png "API Logs")

### Task 3: Deploy a service using kubectl

In this task, deploy the web service using `kubectl`.

1. Open a **new** Azure Cloud Shell console.

2. Create a text file called `web.deployment.yml` using the Azure Cloud Shell
   Editor.

   ```bash
   code web.deployment.yml
   ```

3. Copy and paste the following text into the editor:

   > **Note**: Be sure to copy and paste only the contents of the code block carefully to avoid introducing any special characters.

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     labels:
         app: web
     name: web
   spec:
     replicas: 1
     selector:
         matchLabels:
           app: web
     strategy:
         rollingUpdate:
           maxSurge: 1
           maxUnavailable: 1
         type: RollingUpdate
     template:
         metadata:
           labels:
               app: web
           name: web
         spec:
           containers:
           - image: [LOGINSERVER].azurecr.io/content-web
             env:
               - name: CONTENT_API_URL
                 value: http://api:3001
             livenessProbe:
               httpGet:
                   path: /
                   port: 3000
               initialDelaySeconds: 30
               periodSeconds: 20
               timeoutSeconds: 10
               failureThreshold: 3
             imagePullPolicy: Always
             name: web
             ports:
               - containerPort: 3000
                 hostPort: 80
                 protocol: TCP
             resources:
               requests:
                   cpu: 1000m
                   memory: 128Mi
             securityContext:
               privileged: false
             terminationMessagePath: /dev/termination-log
             terminationMessagePolicy: File
           dnsPolicy: ClusterFirst
           restartPolicy: Always
           schedulerName: default-scheduler
           securityContext: {}
           terminationGracePeriodSeconds: 30
   ```

4. Update the `[LOGINSERVER]` entry to match the name of your ACR Login Server.

5. Select the **...** button and choose **Save**.

   ![In this screenshot of an Azure Cloud Shell editor window, the ... button has been selected and the Save option is highlighted.](media/b4-image62.png "Save Azure Cloud Shell changes")

6. Select the **...** button again and choose **Close Editor**.

   ![In this screenshot of the Azure Cloud Shell editor window, the ... button has been selected and the Close Editor option is highlighted.](media/b4-image63.png "Close Azure Cloud Editor")

7. Create a text file called `web.service.yml` using the Azure Cloud Shell
   Editor.

   ```bash
   code web.service.yml
   ```

8. Copy and paste the following text into the editor:

   > **Note**: Be sure to copy and paste only the contents of the code block carefully to avoid introducing any special characters.

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     labels:
       app: web
     name: web
   spec:
     ports:
       - name: web-traffic
         port: 80
         protocol: TCP
         targetPort: 3000
     selector:
       app: web
     sessionAffinity: None
     type: LoadBalancer
   ```

9. Save changes and close the editor.

10. Type the following command to deploy the application described by the YAML files. You will receive a message indicating the items kubectl has created a web deployment and a web service.

    ```bash
    kubectl create --save-config=true -f web.deployment.yml -f web.service.yml
    ```

    ![In this screenshot of the console, kubectl apply -f kubernetes-web.yaml has been typed and run at the command prompt. Messages about web deployment and web service creation appear below.](media/image93.png "kubectl create application")

11. Return to the AKS blade in the Azure Portal. From the navigation menu, under **Kubernetes resources**, select the **Services and ingresses** view. You should be able to access the website via an external endpoint.

    ![AKS services and ingresses shown with External IP highlighted](media/aks-resources-services-ingresses-view.png "AKS services and ingresses shown with External IP highlighted")

12. In the top navigation, select the `speakers` and `sessions` links.

    ![A screenshot of the web site showing no data displayed.](media/Ex2-Task3.11.png "Web site home page")

### Task 4: Review Azure Monitor for Containers

In this task, you will access and review the various logs and dashboards made available by Azure Monitor for Containers.

1. From the Azure Portal, select the resource group you created named `fabmedical-SUFFIX`, and then select your `Kubernetes Service` Azure resource.

   ![In this screenshot, the resource group was previously selected and the AKS cluster is selected.](https://raw.githubusercontent.com/CloudLabs-MCW/MCW-Cloud-native-applications/fix/Hands-on%20lab/local/ex3tsk7-step1.png "Select fabmedical resource group")

2. From the Monitoring blade, select **Insights**.

   ![In the Monitoring blade, Insights is highlighted.](media/Ex2-Task8.2.png "Select Insights link")

3. Review the various available dashboards and a deeper look at the various metrics and logs available on the Cluster, Nodes, Controllers, and deployed Containers.

   ![In this screenshot, the dashboards and blades are shown. Cluster metrics can be reviewed.](media/Ex2-Task8.3.png "Review the dashboard metrics")

4. To review the Containers dashboards and see more detailed information about each container, select the **Containers** tab.

   ![In this screenshot, the various containers information is shown.](media/monitor_1.png "View containers data")

5. Now filter by container name and search for the **web** containers, you will see all the containers created in the Kubernetes cluster with the pod names. 

   ![In this screenshot, the containers are filtered by container named web.](media/monitor_3.png "Filter data by container and web")

6. By default, the CPU Usage metric will be selected displaying all CPU information for the selected container, to switch to another metric open the metric dropdown list and select a different metric.

   ![In this screenshot, the various metric options are shown.](media/monitor_2.png "Filter by CPU usage")

7. Upon selecting any pod, all the information related to the selected metric will be displayed on the right panel, and that would be the case when selecting any other metric, the details will be displayed on the right panel for the selected pod.

   ![In this screenshot, the pod CPU usage details are shown.](media/cna13.png "POD CPU details")

8. To display the logs for any container simply select it and view the right panel and you will find "View container logs" option which will list all logs for this specific container.

   ![In the View in Analytics dropdown, the View container logs item is selected.](media/cna14.png "View container logs menu option")

   ![The container logs are displayed based on a query entered in the query window.](media/monitor_6.png "Container logs")

9. For each log entry you can display more information by expanding the log entry to view the below details.

   ![The container log query results are displayed, one log entry is expanded in the results view with its details shown.](media/monitor_7.png "Expand the results")
