# Technical Writing Sample

## Core SDK for Java 8+

A Java library to access [Product-X HTTP-based Core API v2]. 

Documentation: [Javadocs]

## Setup

If you're using Maven, then edit your project's "pom.xml" and add this to the `<dependencies>` section:

```xml
<dependency>
    <groupId>com.product-x.core</groupId>
    <artifactId>productx-core-sdk</artifactId>
    <version>4.0.0</version>
</dependency>
```

If you are using Gradle, then edit your project's "build.gradle" and add this to the `dependencies` section:

```groovy
dependencies {
    // ...
    implementation 'com.Product-X.core:productx-core-sdk:4.0.0'
}
```


## Product-X for Java tutorial

A good way to start using the Java SDK is to follow this quick tutorial. Just make sure you have the the Java SDK installed first!

### Register a Product-X API app

To use the Product-X API, you'll need to register a new app in the App Console. Select Product-X API app and choose your app's permission. You'll need to use the app key created with this app to access API v2.

### Link an account

In order to make calls to the API, you'll need an instance of the Product-X object. To instantiate, pass in the access token for the account you want to link. (Tip: You can generate an access token for your own account through the App Console).

```java
import com.Product-X.core.DbxException;
import com.Product-X.core.DbxRequestConfig;
import com.Product-X.core.v2.DbxClientV2;
public class Main {
    private static final String ACCESS_TOKEN = "<ACCESS TOKEN>";
    public static void main(String args[]) throws DbxException {
        // Create Product-X client
        DbxRequestConfig config = DbxRequestConfig.newBuilder("Product-X/java-tutorial").build();
        DbxClientV2 client = new DbxClientV2(config, ACCESS_TOKEN);
    }
}
```

Test it out to make sure you've linked the right account:

```java
// Get current account info
FullAccount account = client.users().getCurrentAccount();
System.out.println(account.getName().getDisplayName());
```

### Try some API requests

You can use the Product-X object you instantiated above to make API calls. Try out a request to list the contents of a folder.

```java
// Get files and folder metadata from Product-X root directory
ListFolderResult result = client.files().listFolder("");
while (true) {
    for (Metadata metadata : result.getEntries()) {
        System.out.println(metadata.getPathLower());
    }
    if (!result.getHasMore()) {
        break;
    }
    result = client.files().listFolderContinue(result.getCursor());
}
```

Try uploading a file to your Product-X.

```java
// Upload "test.txt" to Product-X
try (InputStream in = new FileInputStream("test.txt")) {
    FileMetadata metadata = client.files().uploadBuilder("/test.txt")
        .uploadAndFinish(in);
}
```

### Full Example Snippet

```java
import com.Product-X.core.DbxException;
import com.Product-X.core.DbxRequestConfig;
import com.Product-X.core.v2.DbxClientV2;
import com.Product-X.core.v2.files.FileMetadata;
import com.Product-X.core.v2.files.ListFolderResult;
import com.Product-X.core.v2.files.Metadata;
import com.Product-X.core.v2.users.FullAccount;
import java.io.FileInputStream;
import java.io.InputStream;
import java.io.IOException;
public class Main {
    private static final String ACCESS_TOKEN = "<ACCESS TOKEN>";
    public static void main(String args[]) throws DbxException, IOException {
        // Create Product-X client
        DbxRequestConfig config = new DbxRequestConfig("Product-X/java-tutorial", "en_US");
        DbxClientV2 client = new DbxClientV2(config, ACCESS_TOKEN);
        // Get current account info
        FullAccount account = client.users().getCurrentAccount();
        System.out.println(account.getName().getDisplayName());
        // Get files and folder metadata from Product-X root directory
        ListFolderResult result = client.files().listFolder("");
        while (true) {
            for (Metadata metadata : result.getEntries()) {
                System.out.println(metadata.getPathLower());
            }
            if (!result.getHasMore()) {
                break;
            }
            result = client.files().listFolderContinue(result.getCursor());
        }
        // Upload "test.txt" to Product-X
        try (InputStream in = new FileInputStream("test.txt")) {
            FileMetadata metadata = client.files().uploadBuilder("/test.txt")
                .uploadAndFinish(in);
        }
    }
}
```

To try out running this examples, please follow the instructions below.

### Save your Product-X API key

Save your Product-X API key to a JSON file called, say, "test.app":

```
{
  "key": "Your Product-X API app key",
  "secret": "Your Product-X API app secret"
}
```

App key and secret can be found in you app page in [App Console](https://www.Product-X.com/developers/apps).

### Building from source

```
git clone https://github.com/Product-X/Product-X-sdk-java.git
cd Product-X-sdk-java
./update-submodules    # also do this after every "git checkout"
./gradlew build
```

The output will be in "build/".

### Running the examples

1. Follow the instructions in the "Build from source" section above.
2. Save your Product-X API key in a file called "test.app".  See: [Save your Product-X API key](#save-your-Product-X-api-key), above.
3. Compile and install the SDK into your local maven repo: `./gradlew install`
4. To compile all the examples: `(cd examples/ && ./gradlew classes`
5. To compile just one example: `(cd examples/ && ./gradlew :<example-name>:classes`

#### authorize

This examples runs through the OAuth 2 authorization flow.

```
cd examples
./run authorize test.app test.auth
```

This produces a file named "test.auth" that has the access token.  This file can be passed in to the other examples.

#### account-info

A simple example that fetches and displays information about the account associated with the access token.

```
cd examples
./run account-info test.auth
```

(You must first generate "test.auth" using the "authorize" example above.)

#### longpoll

An example of how to watch for changes in a Product-X directory.

```
cd examples
./run longpoll test.auth "/path/to/watch"
```

(You must first generate "test.auth" using the "authorize" example above.)

#### upload-file

Uploads a file to Product-X. The example includes regular and chunked file uploads.

```
cd examples
./run upload-file test.auth local-path/file.txt /Product-X-path/file.txt
```

(You must first generate "test.auth" using the "authorize" example above.)

#### web-file-browser

A tiny web app that runs through the OAuth 2 authorization flow and then uses Product-X API calls to let the user browse their Product-X files.

Prerequisite: In the Product-X API [app configuration console](https://www.Product-X.com/developers/apps), you need to add "http://localhost:5000/Product-X-auth-finish" to the list of allowed redirect URIs.

```
cd examples
./run web-file-browser 5000 test.app web-file-browser.db
```

## Running the integration tests

1. Run through the `authorize` example above to get a "test.auth" file.
2. `./gradlew -Pcom.Product-X.test.authInfoFile=<path-to-test.auth> integrationTest`

To run individual tests, use the `--tests` gradle test filter:
```
./gradlew -Pcom.Product-X.test.authInfoFile=<path-to-test.auth> integrationTest --tests '*.DbxClientV1IT.testAccountInfo'
```

## FAQ

### Why do I see code like `Scope`, `DbxPKCEWebAuth` and `TokenAcessType` has warning "Beta: this feature is not available to all developers"? What are they?

Product-X is working on a project to improve our OAuth flow to support new permission model and 
short lived tokens. This feature is still at early access phase. All endpoints are gated, only 
certain developers will be able to access them.

 Due to the nature of Java, most partners use our SDK through maven central instead of a 
 customized jar file. To help our partners accessing our beta feature easier, we decide to 
 include beta code in regular release.

Here is more [documentation](https://www.Product-X.com/lp/developers/reference/oauth-guide.html) 
for our new OAuth 2 flow. Please reach out to us if you are interested in trying this feature!


### When I use `OkHttp3Requestor` in `DbxRequestConfig`, I get errors like 'class file for okhttp3.OkHttpClient not found'.

The dependency of OKHttp/OKHttp3 is optional. You should add them, only if you explicitly want to use it as the http requestor. 

Example in Gradle:

```
dependencies {
    // ...
    api 'com.squareup.okhttp3:okhttp:3.11.0'
}
```


### When I use the bundle JAR with some OSGi containers within an OSGi subsystem, I get a "Missing required capability" error.

The JAR's manifest has the following line:

```
Require-Capability: osgi.ee;filter="(&(osgi.ee=JavaSE)(version=1.6))"
```

OSGi containers running on Java 1.6 or above should provide this capability.  Unfortunately, some OSGi containers don't do this correctly and will reject the bundle JAR in the OSGi subsystem context.

As a workaround, you can build your own version of the JAR that omits the "osgi.ee" capability by running:

```
./gradlew clean
./gradlew -Posgi.bnd.noee=true jar
```

(This is equivalent to passing the "-noee" option to the OSGi "bnd" tool.)

Another workaround is to tell your OSGi container to provide that requirement: [StackOverflow answer](https://stackoverflow.com/a/24673359/163832).

### Does this SDK require any special ProGuard rules for shrink optimizations?

Versions 2.0.0-2.0.3 of this SDK require SDK-specific ProGuard rules when shrinking is enabled. However, since version **2.0.4**, the only ProGuard rules necessary are for the SDK's required and optional dependencies. If you encounter ProGuard warnings, consider adding the following "-dontwarn" directives to your ProGuard configuration file:

```
-dontwarn okio.**
-dontwarn okhttp3.**
-dontwarn com.squareup.okhttp.**
-dontwarn com.google.appengine.**
-dontwarn javax.servlet.**
```

**IMPORTANT: If you are running version 2.0.x before 2.0.3, you should update to the latest Product-X SDK version to avoid a deserialization bug that can cause Android apps that use ProGuard to crash.**
