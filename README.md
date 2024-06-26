# GCP Cloud
Repo for discovering endpoints for **[GKE](#gke)**, **[GCE](#gce)**, connection strings for **[MySQL, PostgreSQL, and SQL Server](#sql)**, and **[Uptime Check](#uptime)**
  
## [GKE](#anchors-gke)

### POM

> ```
> <dependency>
>     <groupId>com.google.cloud</groupId>
>     <artifactId>google-cloud-container</artifactId>
>     <version>2.1.7</version> <!-- Check for the latest version -->
> </dependency>
> <dependency>
>     <groupId>io.kubernetes</groupId>
>     <artifactId>client-java</artifactId>
>     <version>13.0.0</version> 
> </dependency>
> ```

### GKEClientExample

> ```
> import com.google.cloud.container.v1.ClusterManagerClient;
> import com.google.cloud.container.v1.ClusterManagerSettings;
> import com.google.cloud.container.v1.ListClustersRequest;
> import com.google.cloud.container.v1.ListClustersResponse;
> import com.google.container.v1.Cluster;
> 
> public class GKEClientExample {
>     public static void main(String[] args) {
>         // Set your Google Cloud project ID and zone
>         String projectId = "your-project-id"; // Replace with your project ID
>         String zone = "your-zone"; // Replace with your zone, e.g., "us-central1-a"
> 
>         try {
>             // Initialize the GKE client
>             ClusterManagerSettings clusterManagerSettings = ClusterManagerSettings.newBuilder().build();
>             try (ClusterManagerClient clusterManagerClient = ClusterManagerClient.create(clusterManagerSettings)) {
>                 
>                 // List all clusters in the specified project and zone
>                 ListClustersRequest request = ListClustersRequest.newBuilder()
>                         .setParent(String.format("projects/%s/locations/%s", projectId, zone))
>                         .build();
>                 ListClustersResponse response = clusterManagerClient.listClusters(request);
>                 
>                 for (Cluster cluster : response.getClustersList()) {
>                     System.out.println("Cluster name: " + cluster.getName());
>                     System.out.println("Cluster endpoint: " + cluster.getEndpoint());
>                     // Here you can use the cluster details to configure a Kubernetes client
>                     // and interact with the cluster's resources, such as services and ingresses.
>                 }
>             }
>         } catch (Exception e) {
>             e.printStackTrace();
>         }
>     }
> }
> ```

### KubeClientExample

> ```
> import io.kubernetes.client.openapi.ApiClient;
> 
> import io.kubernetes.client.openapi.Configuration;
> import io.kubernetes.client.openapi.apis.CoreV1Api;
> import io.kubernetes.client.openapi.models.V1ServiceList;
> import io.kubernetes.client.util.Config;
> 
> public class KubeClientExample {
>     public static void main(String[] args) {
>         try {
>             // Load the kubeconfig file.
>             ApiClient client = Config.defaultClient();
>             Configuration.setDefaultApiClient(client);
> 
>             CoreV1Api api = new CoreV1Api();
>             String namespace = "default"; // Change this to your namespace
> 
>             // List all Services in the specified namespace
>             V1ServiceList services = api.listNamespacedService(namespace, null, null, null, null, null, null, null, null, null, null);
>             services.getItems().forEach((service) -> {
>                 System.out.println(service.getMetadata().getName());
>             });
>         } catch (Exception e) {
>             e.printStackTrace();
>         }
>     }
> }
> ```

## [GCE](#anchors-gce)

### POM

> ```
> <dependency>
>     <groupId>com.google.cloud</groupId>
>     <artifactId>google-cloud-compute</artifactId>
>     <version>0.119.0-alpha</version> <!-- Make sure to use the latest version -->
> </dependency>
> ```

### ComputeEngineClientExample

> ```
> import com.google.cloud.compute.v1.Instance;
> import com.google.cloud.compute.v1.InstanceClient;
> import com.google.cloud.compute.v1.InstancesClient;
> import com.google.cloud.compute.v1.ProjectZoneName;
> 
> public class ComputeEngineClientExample {
>     public static void main(String[] args) {
>         String projectId = "your-project-id"; // Replace with your project ID
>         String zone = "your-zone"; // Replace with your zone
> 
>         try (InstancesClient instancesClient = InstancesClient.create()) {
>             ProjectZoneName zoneName = ProjectZoneName.of(projectId, zone);
>             for (Instance instance : instancesClient.list(zoneName).iterateAll()) {
>                 System.out.println("Instance name: " + instance.getName());
>                 instance.getNetworkInterfacesList().forEach(networkInterface -> {
>                     // Displaying the internal IP
>                     System.out.println("Internal IP: " + networkInterface.getNetworkIP());
>                     // External IP, if available
>                     networkInterface.getAccessConfigsList().forEach(accessConfig -> {
>                         if (accessConfig.hasNatIP()) {
>                             System.out.println("External IP: " + accessConfig.getNatIP());
>                         }
>                     });
>                 });
>             }
>         } catch (Exception e) {
>             e.printStackTrace();
>         }
>     }
> }
> ```

## [SQL](#anchors-sql)

### POM

> ```
> <dependency>
>     <groupId>com.google.cloud</groupId>
>     <artifactId>google-cloud-sqladmin</artifactId>
>     <version>1.2.2</version> <!-- Check for the latest version -->
> </dependency>
> ```

### CloudSqlClientExample

> ```
> import com.google.api.gax.rpc.NotFoundException;
> import com.google.cloud.sql.v1.SqlInstancesGetRequest;
> import com.google.cloud.sql.v1.DatabaseInstance;
> import com.google.cloud.sqladmin.v1.SqlInstancesServiceClient;
> 
> public class CloudSqlClientExample {
>     public static void main(String[] args) {
>         String projectId = "your-project-id"; // Replace with your project ID
>         String instanceId = "your-instance-id"; // Replace with your Cloud SQL instance ID
> 
>         try (SqlInstancesServiceClient sqlClient = SqlInstancesServiceClient.create()) {
>             SqlInstancesGetRequest request = SqlInstancesGetRequest.newBuilder()
>                     .setProject(projectId)
>                     .setInstance(instanceId)
>                     .build();
> 
>             DatabaseInstance instance = sqlClient.get(request);
>             String ipAddress = instance.getIpAddressesList().get(0).getIpAddress(); // Assuming the first IP is the one you want
> 
>             // Example for a PostgreSQL instance
>             String connectionStr = String.format("jdbc:postgresql://%s/%s", ipAddress, instance.getDatabaseVersion());
>             System.out.println("Connection string: " + connectionStr);
>         } catch (NotFoundException e) {
>             System.err.println("The specified instance does not exist.");
>         } catch (Exception e) {
>             e.printStackTrace();
>         }
>     }
> }
> ```

> [!TIP]
> ### Constructing the Connection String
> The connection string format depends on the database type. Here are examples for MySQL, PostgreSQL, and SQL Server, assuming you have the necessary information like IP address, database name, user, and password:
> 
> * **MySQL:** "jdbc:mysql://[HOSTNAME]:[PORT]/[DATABASE]"
> * **PostgreSQL:** "jdbc:postgresql://[HOSTNAME]:[PORT]/[DATABASE]"
> * **SQL Server:** "jdbc:sqlserver://[HOSTNAME]:[PORT];databaseName=[DATABASE]"
> 
> Replace [HOSTNAME], [PORT], [DATABASE], [USER], and [PASSWORD] with your actual database details. The port is typically standard for each database type (e.g., 5432 for PostgreSQL, 3306 for MySQL, 1433 for SQL Server), unless you've configured it differently.
> 
> Remember, for production environments, it's recommended to use Cloud SQL's private IP for connectivity within the same Google Cloud project or VPC network, and ensure your connection strings and credentials are securely managed, not hardcoded in your > application.


## [Uptime](#anchors-uptime)

### POM

> ```
> <dependencies>
>     <!-- Google Cloud Monitoring client library -->
>     <dependency>
>         <groupId>com.google.cloud</groupId>
>         <artifactId>google-cloud-monitoring</artifactId>
>         <version>3.0.1</version> <!-- Make sure to use the latest version -->
>     </dependency>
> </dependencies>
> ```

### ListUptimeChecks

> ```
> import com.google.cloud.monitoring.v3.UptimeCheckServiceClient;
> import com.google.monitoring.v3.ListUptimeCheckConfigsRequest;
> import com.google.monitoring.v3.ProjectName;
> import com.google.monitoring.v3.UptimeCheckConfig;
> 
> public class ListUptimeChecks {
>     public static void main(String[] args) throws Exception {
>         // Replace "your-project-id" with your actual GCP project ID.
>         String projectId = "your-project-id";
>         listUptimeCheckConfigs(projectId);
>     }
> 
>     public static void listUptimeCheckConfigs(String projectId) throws Exception {
>         // Initialize the client, which will be used to send requests.
>         try (UptimeCheckServiceClient client = UptimeCheckServiceClient.create()) {
>             ProjectName projectName = ProjectName.of(projectId);
> 
>             ListUptimeCheckConfigsRequest request = ListUptimeCheckConfigsRequest.newBuilder()
>                     .setParent(projectName.toString())
>                     .build();
> 
>             // Iterate over all response items
>             for (UptimeCheckConfig config : client.listUptimeCheckConfigs(request).iterateAll()) {
>                 System.out.println(config.getName());
>                 // You can also access other properties of the Uptime Check Config here.
>             }
>         }
>     }
> }
> ```

> [!TIP]
> ### Steps to Set Up an Uptime Check for External Services:
> 
> * Go to the Cloud Console: Navigate to the Cloud Monitoring section in the Google Cloud Console.
> * Create an Uptime Check: Click on "Uptime Checks" in the sidebar menu, then click "Create Uptime Check".
> * Configure the Check: Fill in the details for the Uptime Check, including the Resource Type (URL/IP), Host Name (or IP address), and the Path (if applicable). You can also specify the check frequency and other parameters.
> * Set Up Alerting: Optionally, you can configure an alerting policy directly from the Uptime Check creation process. This allows you to define who gets notified and how in case the check fails.
> * Save and Test: After configuring the check and any alerting policies, save the Uptime Check. You can then test it to ensure it's set up correctly and that it can successfully monitor the target service. 
