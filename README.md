# docker-android-sdk

Contains Dockerfiles to create docker images to build android projects.

# Documentation
## Prerequisites
In order to use these images correctly, you must install the following packages:
- [Docker](https://docs.docker.com/engine/install/ubuntu/)
- [Gitlab-runner](https://docs.gitlab.com/runner/install/linux-manually.html)

## Setting up gitlab-runner

### Registering a runner

Go to your gitlab project > settings > CI CD, expand "runners" to get your registration token.

In your server where gitlab-runner and docker are installed, run :
```bash
gitlab-runner register
```
Then follow instructions. Don't forget to give a custom tag (for example "android", it will be used in gitlab-ci to specify the runner to use). Specify docker as executor, with alpine as default image.

Once your runner config is done, edit the file /etc/gitlab-runner/config.toml :
```bash
vim /etc/gitlab-runner/config.toml
```
Find the configuration of your runner, for example, if you set the name as "android-sdk", search for the line : 
name = "android-sdk".
Now, find the "volumes" line and edit it
from
```bash
volumes = ["/cache"]
```
to
```bash
volumes = ["/cache","/home/alpine/android"]
```
Let's explain how this works.

1. The first time you execute a gitlab-ci job with this runner, an anonymous docker volume will be created at /var/lib/docker/volumes. This volume will be the same for each job of the runner.
    * This volume will be bound to the /home/alpine/android container directory, which is where all the android tools are installed.

2. When your build your project with gradle, gradle will use the env variable 
$ANDROID_SDK_ROOT (which is equals to /home/alpine/android in our docker images) to search for all the tools he needs. 
    * If the tools are not installed, gradle will automatically install them. As example, if gradle need the 28.0.2 version of the the build-tools, it will download them and put them in /home/alpine/android/build-tools/. Then, it will launch the build process.
    * If the tools are already installed, gradle will launch the build process without downloading them again. 

3. When your project build is successful, all the tools needed are in the home/alpine/android directory, which is bound to an anonymous docker volume. So every time your gitlab-runner runs a new job, it will bind all the content of the anonymous docker volume to /home/alpine/android to prevent gradle from re downloading again the tools.

In short, each runner has his own cache located in an anonymous docker volume located in the server where the gitlab-runner runs, and all the tools are only downloaded the first time the job is executed by the runner.

## gitlab-ci configuration

here is an example of a gitlab-ci which use a custom docker image to build an android project with gradle :
```yaml
image: android:28
stages:
 - build
before_script:
 - export GRADLE_USER_HOME=`pwd`/.gradle
cache:
 paths:
 - .gradle/wrapper
 - .gradle/caches
build:
 tags:
 - android #replace with the tag that you gave to your runner
 stage: build
 script:
 - git clone <your_project>
 - cd <your_project>
 - ./gradlew build
```
## Images configurations
There are 4 Dockerfiles in the repository. 
Each Dockerfile contains all the environment to compile an android project with a few  changes between them.
- The "28-" Dockerfile use openjdk8 to compile android projects. It is used to compile android projects that use the level 28 of the Android API.
- The 28-ndk is the same Dockerfile as the previous, but when the container is created, the ndk are installed if this is the first time the gitlab-runner runs this container.

- The "29+" Dockerfile use openjdk11 to compile android projects. It is used to compile android projects that use the level 29 of the Android API.
- The 29+ndk is the same Dockerfile as the previous, but when the container is created, the ndk are installed if this is the first time the gitlab-runner runs this container.

