Maven in SF uses **Nexus Repository Manager** to source dependencies when building a project. This is a pre-requisite before running maven build anywhere.

Note: If moving from SF, can delete this page, or move to the /Salesforce folder.
## Nexus Credentials

Generate nexus credentials to download repos for maven [here](https://nexus-dev.data.sfdc.net/nexus/)

Click Profile, generate creds if not already.

Clone `https://git.soma.salesforce.com/service-cloud-realtime/scrt2-pod`

cd into and run 

`make maven`

This will setup your ~/.m2/settings.xml and import certs for you
## Config

```
vi ~/.m2/settings.xml
```

TODO: update below to proper latest nexus settings

```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                              http://maven.apache.org/xsd/settings-1.0.0.xsd">
<servers>
    <server>
        <id>central</id>
        <username>2YR6ZS3I</username>
        <password>iwBcee-Rz-oeXzXyoJdhg4ijS_kbMOyvirOLxk75y3Ax</password>
    </server>
    <server>
        <id>sfdc.sharedlibs</id>
        <username>2YR6ZS3I</username>
        <password>iwBcee-Rz-oeXzXyoJdhg4ijS_kbMOyvirOLxk75y3Ax</password>
    </server>
    <server>
        <id>core</id>
        <username>2YR6ZS3I</username>
        <password>iwBcee-Rz-oeXzXyoJdhg4ijS_kbMOyvirOLxk75y3Ax</password>
</server>
    <server>
        <id>nexus</id>
        <username>2YR6ZS3I</username>
        <password>iwBcee-Rz-oeXzXyoJdhg4ijS_kbMOyvirOLxk75y3Ax</password>
    </server>
</servers>
  <profiles>
    <profile>
      <id>sfdc</id>
      <!--
       | By defining repositories having the ID "central", we essentially mirror Maven Central and save
       | some HTTP round-trips. Our internal Nexus repository proxies Maven Central, so you will find everything
       | there that you'd find by letting Maven hit Maven Central directly.
       |-->
      <repositories>
        <repository>
          <id>central</id>
          <url>https://nexus-dev.data.sfdc.net/nexus/content/groups/public/</url>
        </repository>
      </repositories>
      <pluginRepositories>
        <pluginRepository>
          <id>central</id>
          <url>https://nexus-dev.data.sfdc.net/nexus/content/groups/public/</url>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>

  <activeProfiles>
    <activeProfile>sfdc</activeProfile>
  </activeProfiles>

  <mirrors>
    <mirror>
      <id>nexus</id>
      <mirrorOf>*,!core,!staging,!snapshots,!sfdc.sharedlibs</mirrorOf>
      <url>https://nexus-proxy.repo.local.sfdc.net/nexus/content/groups/public/</url>
      <!-- <url>https://nexus-dev.data.sfdc.net/nexus/content/groups/public/</url> -->
    </mirror>
  </mirrors>
</settings>
```