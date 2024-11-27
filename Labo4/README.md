# Lab 04: Develop a web app on a Platform-as-a-Service (Google App Engine)

#### Organisation

- Duration of this lab is 4 periods.

#### Pedagogical objectives

- Develop and deploy a custom web application on a PaaS infrastructure (Google App Engine)
    
- Use a data persistence service (NoSQL database)
    
- Conduct performance tests and observe the auto-scaling mechanisms of the platform
    
- Assess how quickly resources (quotas) are used up
    

#### Tasks

In this lab you will perform a number of tasks and document your progress in a lab report. Each task specifies one or more deliverables to be produced. Collect all the deliverables in your lab report. Give the lab report a structure that mimics the structure of this document.

Prerequisites:

- A Google account with Google Cloud Platform enabled
- A billing account with a credit (obtained by redeeming a Google for Education coupon)

## Task 0 Set up the development environment on your machine

The setup of the development environment consists of the steps detailed below. If you run into trouble, please consult the documentation [Building a Java app on App Engine](https://cloud.google.com/appengine/docs/standard/java-gen2/building-app).

If you haven’t already, install the following software on your local machine:

- [Java Development Kit (JDK) 17 or later](https://adoptium.net/)
    
- [Maven 3.8 or later](http://maven.apache.org/). To view the version of your installed Maven, type `mvn -v`.
    

To work with Google Cloud, you will need to install

- [The gcloud CLI](https://cloud.google.com/sdk/docs/install)

## Task 1: Deployment of a simple web application

In this task you will create a simple web application. You will first test it on your local machine, then you will deploy it on Google App Engine (GAE).

### Overview

Here’s an overview, detailed instructions follow below.

- In the management console create a new Google Cloud project to hold all cloud resources related to your app. Connect the project to your billing account.
- Initialize the CLI to connect it to your Google Cloud project.
- Enable cloud APIs that will be needed for the app:
    - App Engine API
    - App Engine Reporting API
    - Cloud Build API
    - Compute API
- In the management console, in the project, create a new App Engine application. Select a region for the app’s computing resources.
- On your local machine, create a code project containing your app’s code.
    - Create a minimal Spring Boot project with a “Hello, World!” application.
    - Run the helloworld application on your local machine for verification.
    - You can use IntelliJ IDEA to develop.
- Deploy your app on App Engine
    - Add an `app.yaml` file to the code base to pass configuration info to App Engine.
    - Use the CLI to build and deploy the app on App Engine.
- View your app on App Engine.

### Detailed instructions

#### Create Google Cloud project

1. Log into the management console [https://console.cloud.google.com/](https://console.cloud.google.com/). Create a new project called `cldlabgae`. Make sure that the project is linked to your billing account. After it has been created, select the project in the drop-down project selector in the top menu bar.
    
2. Enable the App Engine Admin API (`appengine.googleapis.com`):
    
    - Enable: **APIs & Services** > **+ ENABLE APIS AND SERVICES** > **App Engine Admin API** > **ENABLE**.
3. Enable and configure the Cloud Build API (`cloudbuild.googleapis.com`):
    
    - Enable: **APIs & Services** > **+ ENABLE APIS AND SERVICES** > **Cloud Build API** > **ENABLE**.
    - Configure: **Cloud Build** > **Settings** > **App Engine - App Engine Admin** > **ENABLED**.
4. Initialize the Google Cloud CLI to connect to your Google account and your Google Cloud project:
    
    ```
     gcloud init
    ```
    
5. In the management console, create an App Engine application: **Products & solutions** > **App Engine** > **CREATE APPLICATION**.
    
    - To choose a region, use the [Google Cloud Region Picker](https://cloud.withgoogle.com/region-picker/). Under **Product availability** select **App Engine**.
    - Leave the **Identity and API access** at default values.

#### Create an initial code base for a minimal web app on your local machine

We are going to use the most popular Java framework for web app back-ends, [Spring Boot](https://spring.io/), to create a minimal web app.

Spring Boot has a web site, Spring Initializr, where you can create the files for a minimal Spring Boot project.

1. We will generate a standard Maven-managed Java project with a `pom.xml` build file that includes necessary Spring Boot libraries as dependencies. Go to Spring Initializr with the following link: [https://start.spring.io/#!type=maven-project&language=java&packaging=jar&jvmVersion=17&groupId=com.example.appengine&artifactId=springboot&packageName=com.example.appengine.springboot&dependencies=web](https://start.spring.io/#!type=maven-project&language=java&packaging=jar&jvmVersion=17&groupId=com.example.appengine&artifactId=springboot&packageName=com.example.appengine.springboot&dependencies=web). Click the **Generate** button to generate and download an archive of the project files.
    
    You can open the project in IntelliJ or Visual Studio Code or another IDE.
    
2. The demo application generated by Initializr starts the web application server, but doesn’t process any HTTP requests. We add a simple controller that responds to an HTTP GET request at the root path (`/`) by displaying “Hello, world!”. Modify the file `src/main/java/com/example/appengine/springboot/DemoApplication.java` to include the following code:
    

```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@RestController
public class DemoApplication {

  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }

  @GetMapping("/")
  public String hello() {
    return "Hello, world!";
  }

}
```

3. Test the app by running it locally. Build and run the code with
    
    ```
     mvn spring-boot:run
    ```
    
    then in your web browser, enter the address [http://localhost:8080](http://localhost:8080/). You should see the message “Hello, world!”.
    

#### Deploy the app on Google Cloud

1. App Engine needs to know some things about the app in order to prepare the software in the instance the app will run on. For example it needs to know which Java version the app requires. To specify settings for your app in the App Engine runtime environment, create an `app.yaml` file in the directory `src/main/appengine/` with the line
    
    ```
     runtime: java17
    ```
    
2. Now we use the CLI to deploy the app on App Engine. Google does some heavy lifting in this step, and it takes some time. First, the CLI uploads the app’s source files into Google Cloud Storage. Then, the Cloud Build service builds and packages the app. It stores the resulting binaries in Google Cloud Storage. This creates a new _version_ of the app (App Engine keeps previous versions of the app). Then App Engine allocates a domain name for the app and configures the correct routing in the front end. Finally, when you send an HTTP request to the app, App Engine downloads the app’s code onto an app instance to serve the request.
    
    You can optionally divide your app into _services_ that you can deploy individually (think of microservices). We did not define any service, that is why App Engine creates a service called `default`.
    
    On your local machine, in the terminal `cd` into the root directory of the project and run
    
    ```
     gcloud app deploy
    ```
    
    You should see output similar to the following:
    
    ```
     Services to deploy:
    
     descriptor:                  [/Users/marcel.graf/tech/google_appengine_2024/springboot/pom.xml]
     source:                      [/Users/marcel.graf/tech/google_appengine_2024/springboot]
     target project:              [cld-gae-2024]
     target service:              [default]
     target version:              [20240324t153115]
     target url:                  [https://cld-gae-2024.ey.r.appspot.com]
     target service account:      [cld-gae-2024@appspot.gserviceaccount.com]
    
    
     Do you want to continue (Y/n)?  y
    ```
    
    Then you have to wait while App Engine does its work. After successful deployment, you should see a line similar to the following that indicates the URL of your newly deployed app:
    
    ```
     Deployed service [default] to [https://cld-gae-2024.ey.r.appspot.com]
    ```
    
3. Open the URL in your web browser. You should see the message `Hello, world!`.
    
4. In the App Engine console, verify that you see
    
    - the `default` service that was created: **App Engine** > **Services**,
    - the first version of the `default` service and the URL of the deployed version: **Versions**,
    - the instance that App Engine has created to respond to the HTTP request: **Instances**.
5. Navigate to the Cloud Build console. Select the region in which you created your app. You should see the recent build listed. Click on the build. You should see the build log displayed. What is the Maven command that Cloud Build used to build your app (starting with `mvnw`)?
    
6. Navigate to the Cloud Storage console. You should see two buckets, one called `staging.*`. Browse the files in the bucket. There is a file with type `application/java-archive`. This is the jar file that is the result of the build.
    

#### Deliverables:

- Copy the Maven command to the report.

## Task 2: Add a controller that writes to the Datastore

In this task you are going to add a controller to the app that writes data to the Datastore.

Overview:

- Enable the cloud API for Datastore
- In the project’s Maven build file `pom.xml` add Google’s Datastore client library as dependency.
- In your Java source file, import classes from the Datastore client library and add code to write data to the Datastore.

Detailed instructions:

1. To use the Datastore, you need to enable the cloud API for Datastore.
    
2. Google provides Java client libraries for accessing Datastore. We need to add them as dependencies to the Maven build file `pom.xml`.
    
    In the `pom.xml` file, before the section `<dependencies>`, add a new section `<dependencyManagement>` with the following settings:
    

```
<!--  Using libraries-bom to manage versions.
See https://github.com/GoogleCloudPlatform/cloud-opensource-java/wiki/The-Google-Cloud-Platform-Libraries-BOM -->
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>com.google.cloud</groupId>
      <artifactId>libraries-bom</artifactId>
      <version>26.28.0</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

```
In the existing section `<dependencies>`, add the following dependency:
```

```
<dependency>
  <groupId>com.google.cloud</groupId>
  <artifactId>google-cloud-datastore</artifactId>
</dependency>
```

3. Add the following code with a new controller to the app.

```
    private final Datastore datastore = DatastoreOptions.getDefaultInstance().getService();

    @GetMapping("/dswritesimple")
    public String writeEntityToDatastoreSimple(@RequestParam Map<String, String> queryParameters) {
        StringBuilder message = new StringBuilder();

        KeyFactory keyFactory = datastore.newKeyFactory().setKind("book");
        Key key = datastore.allocateId(keyFactory.newKey());
        Entity entity = Entity.newBuilder(key)
                .set("title", "The grapes of wrath")
                .set("author", "John Steinbeck")
                .build();
        message.append("Writing entity to Datastore\n");
        datastore.put(entity);
        Entity retrievedEntity = datastore.get(key);
        message.append("Entity retrieved from Datastore: "
                + retrievedEntity.getString("title")
                + " " + retrievedEntity.getString("author")
                + "\n");
        return message.toString();
    }
```

4. Build and deploy a new version of the app to App Engine.
    
5. In your web browser, invoke the new controller.
    
6. In the management console, view the entity written to the Datastore: Navigate to **Datastore**. You will see a list of databases with an entry `(default)`. Click on it. You will see a database editor called **Datastore Studio**. You should see the entity written by the controller.
    

#### Deliverables:

Copy a screenshot of Datastore Studio with the written entity into the report.

## Task 3: Develop a controller to write arbitrary entities into the Datastore

In this task you will write a controller that writes arbitrary data to the Datastore by pulling data from the URL query parameters in the HTTP request. You will use it later to test the write performance of the Datastore.

The controller shall work as follows:

- The controller method will be called **writeEntityToDatastore** and will respond to HTTP requests that are sent to the URL path **/dswrite**.
    
- For each HTTP request it receives, the controller shall write an entity to the Datastore.
    
- The controller will receive the data for the entity in the query part of the URL of the HTTP request.
    
    The query part of a URL starts after the question mark ‘?’ and continues until the end. It contains a number of field-value pairs of the form
    
    ```
      field1=value1&field2=value2&field3=value3
    ```
    
    Example: To add a new entity describing a book to the datastore one would send the following URL in the HTTP request:
    
    ```
      https://20200406t150557-dot-labgae.appspot.com/dswrite?_kind=book&_key=837094&author=John%20Steinbeck&title=The%20Grapes%20of%20Wrath
    ```
    
    In Spring Boot, you can obtain the field-value pairs by writing the Java method as follows:
    
    ```
      public String writeEntityToDatastore(@RequestParam Map<String, String> queryParameters) {
          ...
      }
    ```
    
- The controller should be able to deal with arbitrary field-value pairs. Each field-value pair shall become a property of the entity. The field name becomes the property name and the field value the property value.
    
- The controller should treat the fields named _**kind** and ___key **specially. The field** _kind **shall indicate the kind of the entity. It is mandatory. The field** _key__ shall contain the key name of the entity. It is optional. If not present, the controller shall let the Datastore generate the key.
    

Example: When the controller is invoked with the URL given above, it should write an entity of kind **book** with key name **837094** and the properties **author: John Steinbeck** and **title: The Grapes of Wrath** to the datastore.

Hints:

- To debug your code, you can write messages to `System.out`. When App Engine executes your app, it captures standard input (and standard error) and displays it in the management console. To view the logs, navigate to **App Engine** > **Versions** > **Logs**.
    
- Use Datastore Studio to view the entities you have written to the Datastore.
    

#### Deliverables:

Copy a code listing of your app into the report.

## Task 4: Test the performance of Datastore writes

In this task you will performance test the App Engine platform with a load generator. You will compare the performance of normal request processing and request processing that involves Datastore write operations.

As your app is deployed with Automatic Scaling, there is a danger of consuming a lot of resources while testing, and burning through a lot of money. Google gave you a coupon with some money that was put into a billing account. To see how much money you have spent, navigate to **Billing**. Click on **Credits** to see how much credit is left. When doing performance tests, check from time to time to see how much money is consumed.

Conduct the performance tests as follows:

1. Using Vegeta test the performance of normal request processing using the helloworld controller.
    
    - Tell Vegeta to invoke the helloworld controller.
        
    - In the App Engine console open the **Instances** view.
        
    - Run a test.
        
    - In the instances view observe the graph of the incoming requests, the number of instances, the number of requests per second (QPS) each instance received, the latency with which each instance responds to the requests.
        
2. Test the performance of the controller that writes to the Datastore.
    
    - Tell Vegeta to invoke the the Datastore controller and construct a query string with three properties. To avoid writing always the same entity do not specify the key name so that the Datastore generates a new ID each time.
        
    - Repeat the test.
        
3. At the end of the tests observe in detail how much resources were used. In the console click on **Quotas**.
    

Deliverables:

- For each performance test include a graph of the load testing tool and copy three screenshots of the App Engine instances view (graph of requests by type, graph of number of instances, graph of latency) into the report.
    
- What average response times do you observe in the test tool for each controller?
    
- Compare the response times shown by the test tool and the App Engine console. Explain the difference.
    
- ~~
    
    How much resources have you used running these tests? From the **Quota Details** view of the console determine the non-zero resource quotas (**Daily quota** different from 0%). Explain each with a sentence. To get a sense of everything that is measured click on **Show resources not in use**.
    
    ~~
- Let’s suppose you become suspicious that the algorithm for the automatic scaling of instances is not working correctly. Imagine a way in which the algorithm could be broken. Which measures shown in the console would you use to detect this failure?