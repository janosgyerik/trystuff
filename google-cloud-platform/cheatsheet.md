https://codelabs.developers.google.com/devoxx

Google Cloud Shell is a browser-based command line tool to access Google Cloud Platform resources.
Cloud Shell makes it really easy to manage your Cloud Platform Console projects and resources
without having to install the Google Cloud SDK and other tools on your system.


- Free trial budget of $300, during 1 year
- Don't forget to clean up after trials to stop consuming the budget and incur costs
- As of 2018/04/21 -> You have €264.02 in credit and 74 days left in your free trial.


1. Create application
2. Start cloud shell

The shell gives a persistent 5GB home directory.

Command examples:

    gcloud auth list  # shows active account you@gmail.com

    gcloud config list project  # shows current project

    gcloud config set project ID  # switch project

    gcloud -h
    gcloud config --help

    gcloud config list
    gcloud config list --all
    
    # install Spring boot
    curl -s "https://get.sdkman.io" | bash
    source "$HOME/.sdkman/bin/sdkman-init.sh"
    sdk install springboot
    
    # create a spring boot app
    spring init --dependencies=web helloworld
    
    # edit DemoApplication.java to add a GET route
    # run the app
    ./mvnw -DskipTests spring-boot:run
    
    gcloud config list compute/zone
    gcloud config set compute/zone us-central1-f
    
    mvn archetype:generate -DgroupId=com.example.grpc -DartifactId=grpc-hello-server -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    
Open Eclipse Orion by clicking Launch code editor from the Cloud Shell menu.

In `src/main/proto/GreetingService.proto`:

    syntax = "proto3";
    package com.example.grpc;

    // Request payload
    message HelloRequest {
      // Each message attribute is strongly typed.
      // You also must assign a "tag" number.
      // Each tag number is unique within the message.
      string name = 1;

      // This defines a strongly typed list of String
      repeated string hobbies = 2;

      // There are many more basics types, like Enum, Map
      // See https://developers.google.com/protocol-buffers/docs/proto3
      // for more information.
    }

    message HelloResponse {
      string greeting = 1;
    }

    // Defining a Service, a Service can have multiple RPC operations
    service GreetingService {
      // Define a RPC operation
      rpc greeting(HelloRequest) returns (HelloResponse);
    }

Dependencies:

    <project>
      ...
      <dependencies>
        <dependency>
          <groupId>io.grpc</groupId>
          <artifactId>grpc-netty</artifactId>
          <version>1.7.0</version>
        </dependency>
        <dependency>
          <groupId>io.grpc</groupId>
          <artifactId>grpc-protobuf</artifactId>
          <version>1.7.0</version>
        </dependency>
        <dependency>
          <groupId>io.grpc</groupId>
          <artifactId>grpc-stub</artifactId>
          <version>1.7.0</version>
        </dependency>
        ...
      </dependencies>
      ...
    </project>

Add gRPC plugin:

    <project>
      ...
      <dependencies>
        ...
      </dependencies>
      <build>
        <extensions>
          <extension>
            <groupId>kr.motd.maven</groupId>
            <artifactId>os-maven-plugin</artifactId>
            <version>1.5.0.Final</version>
          </extension>
        </extensions>
        <plugins>
          <plugin>
            <groupId>org.xolstice.maven.plugins</groupId>
            <artifactId>protobuf-maven-plugin</artifactId>
            <version>0.5.0</version>
            <configuration>
              <protocArtifact>com.google.protobuf:protoc:3.4.0:exe:${os.detected.classifier}</protocArtifact>
              <pluginId>grpc-java</pluginId>
              <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.7.0:exe:${os.detected.classifier}</pluginArtifact>
            </configuration>
            <executions>
              <execution>
                <goals>
                  <goal>compile</goal>
                  <goal>compile-custom</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
        </plugins>
      </build>
    </project>

Implement the service:

    package com.example.grpc;

    import io.grpc.stub.StreamObserver;

    public class GreetingServiceImpl extends GreetingServiceGrpc.GreetingServiceImplBase {
      @Override
      public void greeting(GreetingServiceOuterClass.HelloRequest request,
            StreamObserver<GreetingServiceOuterClass.HelloResponse> responseObserver) {
      // HelloRequest has toString auto-generated.
        System.out.println(request);

        // You must use a builder to construct a new Protobuffer object
        GreetingServiceOuterClass.HelloResponse response = GreetingServiceOuterClass.HelloResponse.newBuilder()
          .setGreeting("Hello there, " + request.getName())
          .build();

        // Use responseObserver to send a single response back
        responseObserver.onNext(response);

        // When you are done, you must call onCompleted.
        responseObserver.onCompleted();
      }
    }

Start server:

    package com.example.grpc;

    import io.grpc.*;

    public class App
    {
        public static void main( String[] args ) throws Exception
        {
          // Create a new server to listen on port 8080
          Server server = ServerBuilder.forPort(8080)
            .addService(new GreetingServiceImpl())
            .build();

          // Start the server
          server.start();

          // Server threads are running in the background.
          System.out.println("Server started");
          // Don't exit the main thread. Wait until server is terminated.
          server.awaitTermination();
        }
    }

Run it:

    mvn package exec:java -Dexec.mainClass=com.example.grpc.App

Implement a client:

    package com.example.grpc;

    import io.grpc.*;

    public class Client
    {
        public static void main( String[] args ) throws Exception
        {
          // Channel is the abstraction to connect to a service endpoint
          // Let's use plaintext communication because we don't have certs
          final ManagedChannel channel = ManagedChannelBuilder.forTarget("localhost:8080")
            .usePlaintext(true)
            .build();

          // It is up to the client to determine whether to block the call
          // Here we create a blocking stub, but an async stub,
          // or an async stub with Future are always possible.
          GreetingServiceGrpc.GreetingServiceBlockingStub stub = GreetingServiceGrpc.newBlockingStub(channel);
          GreetingServiceOuterClass.HelloRequest request =
            GreetingServiceOuterClass.HelloRequest.newBuilder()
              .setName("Ray")
              .build();

          // Finally, make the call using the stub
          GreetingServiceOuterClass.HelloResponse response = 
            stub.greeting(request);

          System.out.println(response);

          // A Channel should be shutdown before stopping the process.
          channel.shutdownNow();
        }
    }

Run the client:

    mvn package exec:java -Dexec.mainClass=com.example.grpc.Client

To make the service streaming, modify the schema:

    // Defining a Service, a Service can have multiple RPC operations
    service GreetingService {
      // Define a RPC operation
      rpc greeting(HelloRequest) returns (stream HelloResponse);
    }

While the server implementation doesn't need to change (because it's already using an asynchronous implementation),
the client must use an asynchronous stub instead of the blocking stub. Update the client code making sure
to update the stub type to GreetingServiceStub:

    // It is up to the client to determine whether to block the call
    // Here we create a blocking stub, but an async stub,
    // or an async stub with Future are always possible.
    GreetingServiceGrpc.GreetingServiceStub stub = GreetingServiceGrpc.newStub(channel);
    GreetingServiceOuterClass.HelloRequest request =
      GreetingServiceOuterClass.HelloRequest.newBuilder()
        .setName("Jack")
        .build();

    // Make an Asynchronous call. Listen to responses w/ StreamObserver
    stub.greeting(request, new StreamObserver<GreetingServiceOuterClass.HelloResponse>() {
      public void onNext(GreetingServiceOuterClass.HelloResponse response) {
        System.out.println(response);
      }
      public void onError(Throwable t) {
      }
      public void onCompleted() {
        // Typically you'll shutdown the channel somewhere else.
        // But for the purpose of the lab, we are only making a single
        // request. We'll shutdown as soon as this request is done.
        channel.shutdownNow();
      }
    });

## SQL

    # codelab-0 is the name of the MySQL database instance created
    gcloud beta sql connect codelab-0 --user=root
    gcloud beta sql instances delete codelab-0

## Useful links

- https://cloud.google.com/shell/docs/
- https://cloud.google.com/sdk/gcloud/
- https://cloud.google.com/java/
- https://cloud.google.com/java/samples
- https://github.com/grpc/grpc-java
- https://github.com/saturnism/grpc-java-by-example
- https://github.com/saturnism/grpc-java-by-example/tree/master/streaming-example
- https://cloud.google.com/endpoints/docs/grpc/transcoding
- https://cloud.google.com/sql/docs/
