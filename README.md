
### Description of the bug

When defining custom repositories in the maven ```pom.xml``` whose urls use a nested property as shown below, Renovate will skip all dependencies, giving the ```recursive-placeholder```-reason as root cause. See the debug logs below.

Expected behaviour: it should be able to just resolve the urls as they are not infinitely recursive.

Occurence: I first noticed this issue in a self-hosted instance of Renovate (using latest at time of writing). I was able to reproduce the issue with the Mend Renovate host too. Here an excerpt from the  ```pom.xml``` from this project:
```xml
  <properties>
    <url.repo.base>https://my.repo.com</url.repo.base>
    <url.repo.releases>${url.repo.base}/content/repositories/java-releases/</url.repo.releases>
  </properties>

  <repositories>
    <repository>
      <id>repo-dependencies</id>
      <url>${url.repo.base}/content/groups/java-group/</url>
    </repository>
    <repository>
      <id>repo-releases</id>
      <url>${url.repo.releases}</url>
    </repository>
  </repositories>
```
Note that here the property ```${url.repo.releases}``` uses the property ```${url.repo.base}```, I assume that this dependency (which is not an infinite recursion) is causing the issue.

### Relevant debug logs

<details><summary>Logs</summary>
Log from renovate run #690915187 

```
DEBUG: packageFiles with updates
{
  "config": {
    "maven": [
      {
        "datasource": "maven",
        "packageFile": "pom.xml",
        "deps": [
          {
            "datasource": "maven",
            "depName": "org.apache.tika:tika-core",
            "currentValue": "1.20",
            "fileReplacePosition": 848,
            "registryUrls": [
              "https://repo.maven.apache.org/maven2",
              "${url.repo.base}/content/groups/java-group/",
              "${url.repo.releases}"
            ],
            "depType": "compile",
            "skipReason": "recursive-placeholder",
            "depIndex": 0,
            "updates": []
          }
        ],
        "packageFileVersion": "1.0-SNAPSHOT"
      }
    ]
  }
}
```

</details>