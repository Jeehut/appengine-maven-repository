# Installation

### Create a new Project
First of all, you'll need to go to your [Google Cloud console](https://console.cloud.google.com/projectselector/appengine/create?lang=java&st=true) to create a new App Engine application: 
Make sure the project Id is something meaningful. It should be the company name for example. This will be the maven URL used to download the dependencies.

![](https://i.imgur.com/SD1WwP3.png)

As soon as your project is created, a default [Google Cloud storage bucket](https://console.cloud.google.com/storage/browser) has been automatically created for you which provides the first 5GB of storage for free.

### Setup the Google Cloud SDK

Follow the [official documentation](https://cloud.google.com/sdk/docs/) to install the latest Google Cloud SDK. As a shorthand, you'll find below the Ubuntu/Debian instructions:


```bash
$ export CLOUD_SDK_REPO="cloud-sdk-$(lsb_release -c -s)"
$ echo "deb http://packages.cloud.google.com/apt $CLOUD_SDK_REPO main" | sudo tee -a /etc/apt/sources.list.d/google-cloud-sdk.list
$ curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ sudo apt-get update && sudo apt-get install google-cloud-sdk
```

As a last step, configure the `gcloud` command line environment and select your newly created App Engine project when requested to do so:

```bash
$ gcloud init
```

## Configuration

Clone the project

Update [`WEB-INF/users.txt`](src/main/webapp/WEB-INF/users.txt) to declare users, passwords and permissions:

```ini
# That file declares your users - using basic authentication.
# Minimalistic access control is provided through the following permissions: write, read, or list.
# Syntax is:
# <username>:<password>:<permission>
# (use '*' as username and password to declare anonymous users)
*:*:read
admin:l33t:write
john:j123:read
donald:coolpw:read
guest:guest:list
```
> The `list` permission allows users to list the content of the repository but prohibits downloads. The `read` permission allows downloads (and implies `list`). The `write` permission allows uploads (and implies `read`).

> Anonymous users are supported by using "*" for both username and password. For example, `*:*:read` will allow anonymous read access. 

We can keep `*:*:read` to allow all the developers to download the dependencies Or maybe specify one account for reading.

## Deployment

Replace `GCLOUD_CONFIG` in the `build.gradle` file to the project Id created before.
```
appengine {
    deploy {
        projectId = 'GCLOUD_CONFIG'
        version = 'v1'
    }
}
```
Once you're ready to go live, just push the application to Google App-Engine:

```bash
$ cd appengine-maven-repository
$ ./gradlew appengineDeploy
```
Or you can run the gradle task `appengineDeploy` from Android Studio. 

And voilà! Your private Maven repository is hosted and running at the following address:

`https://company-android-libraries.appspot.com`

# Publish a library

Make sure to build the project with the release build type before publishing.
An example publishing artifacts using the maven plugin for Gradle:

```gradle
apply plugin: 'maven-publish'

...

publishing {
    publications {
        maven(MavenPublication) {
            groupId 'com.company-android-libraries'
            artifactId 'utility-android'
            version '1.0.0'
            artifact("$buildDir/outputs/aar/utility-debug.aar") // here the file that will be uploaded.
        }
    }
    repositories {
        maven {
            url "https://company-android-libraries.appspot.com"
            credentials {
                username 'username' // here add the user that has the write permission.
                password 'password'
            }
        }
    }
}
```

Using the above plugin, publishing artifacts to your repository is as simple as:

```bash
$ ./gradlew publish
```
Or you can run the gradle task `publish` from the Android Studio after syncing the project.

In the other way, accessing password protected Maven repositories using Gradle only requires you to specify the `credentials` closure:

```gradle
repositories {
    maven {
        credentials {
            username 'user' // user that has a read access or maybe you can remove the credentials block if you allow the artifact to be accessible without authentication.
            password 'password'
        }
        url "https://company-android-libraries.appspot.com"
    }
}

dependencies {
    implementation "com.company-android-libraries:utility-android:1.0.0"
}
```


# Limitations

Google App-Engine HTTP requests are limited to 32MB - and thus, any artifacts above that limit can't be hosted.

