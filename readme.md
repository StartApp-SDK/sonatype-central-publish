# Sonatype Central Publish

Publish your Android AAR libraries into Maven Central Sonatype repository

## Intro

In March 2025 [Sonatype.org announced](https://central.sonatype.org/news/20250326_ossrh_sunset/) that OSSRH service will be shutdown:

> The OSSRH service will reach end-of-life on June 30th, 2025.

### Gradle & Android

[Sonatype documentation says](https://central.sonatype.org/publish/publish-portal-gradle/):

> Currently, there is no official Gradle plugin for publishing to Maven Central via the Central Publishing Portal.

Many Android developers were using a plugin file `publish-mavencentral.gradle`, which could be found on many websites and repositories.

Nowadays (May 2025), it still can be found here:

- [github.com/eegets/UploadApkPlugin](https://github.com/eegets/UploadApkPlugin/blob/6d37367277b93295b393aa78f8c8fc4416e25fcf/publish-mavencentral.gradle)
- [github.com/lopspower/RxAnimation](https://github.com/lopspower/RxAnimation/blob/ccb2e0e8fde1c8daebd5911791a6b6c1fb91820c/gradle/publish-mavencentral.gradle)
- [github.com/bimromatic/MVVM_Practical](https://github.com/bimromatic/MVVM_Practical/blob/bfbe1dfb0d0366f647f43723598ca317f04f19f7/publish-mavencentral.gradle)

As a temporary solution, Sonatype documentation suggests using [The Portal OSSRH Staging API](https://central.sonatype.org/publish/publish-portal-ossrh-staging-api/), until the most popular plugins will adopt the Sonatype Publishing Portal API.

### Goal

This plugin is aiming to provide a quick fix for teams which are still using `publish-mavencentral.gradle`.

### Implementation

This plugin implements two extra tasks which make HTTP requests to the OSSRH Staging API:

- Manual Search Repository
- Manual Upload Default Repository

These two calls fill the gap in currently available tools. Rest of the process stays very similar to what we had with OSSRH Nexus Repository Manager.

## Usage

In the very end of module-level `build.gradle` add the following line:

```groovy
apply from: 'https://raw.githubusercontent.com/StartApp-SDK/sonatype-central-publish/refs/heads/main/sonatype-central-publish.gradle'
```

Alternatively, you can save file [`sonatype-central-publish.gradle`](sonatype-central-publish.gradle) into the root dir of your project:

```groovy
apply from: "$rootDir/sonatype-central-publish.gradle"
```

In project-level `gradle.properties` add the following lines:

```properties
LIB_DEVELOPER_ID=your_developer_id
LIB_DEVELOPER_NAME=Your Developer Name
LIB_DEVELOPER_EMAIL=email@example.com
LIB_SITE=https://example.com
LIB_LICENSE_NAME=Your License Name
LIB_LICENSE_URL=https://github.com/your_account/your_repo/blob/master/LICENSE
LIB_SCM_CONNECTION=scm:git:github.com/your_account/your_repo.git
LIB_SCM_DEVELOPER_CONNECTION=scm:git:ssh://github.com/your_account/your_repo.git
LIB_SCM_URL=https://github.com/your_account/your_repo
LIB_DESCRIPTION=Your Project Description
```

In module-level `gradle.properties` add the following lines:

```properties
groupId=com.example
artifactName=your_project_name
versionName=0.0.1
```

In root directory of project create or edit `local.properties`, add the following lines: 

```properties
signing.keyId=YOUR_KEY_ID
signing.password=your_password
signing.secretKeyRingFile=/path/to/key.gpg
ossrhUsername=generated_username
ossrhPassword=generated_token
```

**Important!** You must generate new user token for your [Sonatype account](https://central.sonatype.com/account) for making requests via the [Publisher Api](https://central.sonatype.org/publish/publish-portal-api/).

Then execute Gradle command:

```bash
./gradlew build publishReleasePublicationToSonatypeRepository
```

After successful execution you need to complete the upload process on the [Sonatype Publishing Portal](https://central.sonatype.com/publishing/deployments).

## Resolving errors

Sometimes you may get into situation, when some artifacts were uploaded to repository, but were not uploaded to portal. In this case you'll get an error from this plugin saying:

```
Sonatype namespace com.example is not empty.
If you understand what you do and want to avoid this check, you can add a parameter -PsonatypeSkipNamespaceCheck=true
```

You may use parameter `-PsonatypeSkipNamespaceCheck=true` to skip the check, uploading artifacts as is, but then you must go portal and double check your files. In most cases it's better to drop the deployment.
