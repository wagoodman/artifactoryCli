# Artifactory CLI
A dumb and simple script for searching, downloading, and verifying artifacts
from Artifactory.

To use this first open up the artifactory python script and change the `ARTIFACTORY_URL = "repo.jfrog.org/artifactory"`
line to the artifactory URL that you'd like to search on a regular basis.

## Usage by Example
Search for artifacts:
```
$ ./artifactory  search "gradle-0.9.1-2011011" --info
Search results:
   distributions: 5e76aa7e09138942d648b00e301f362e43791e4b /org.gradle/gradle/zips/gradle-0.9.1-20110113063454-bin.zip
   distributions: 363fab80e1d4b85611a58cc1b5c05dfdedcfe2b1 /org.gradle/gradle/0.9.1-20110113073831/gradle-0.9.1-20110113073831-src.zip
   distributions: 0edc819707a0b4ddfac94cfd275de6d36aeb9208 /org.gradle/gradle/zips/gradle-0.9.1-20110113063454-all.zip
   distributions: 56b25fcdbcd216d3ca18a5ff9e854d7e7b88322c /org.gradle/gradle/zips/gradle-0.9.1-20110112155741-bin.zip
...
```

Keep artifact versions for your project in a simple json file:
```
$ cat versions.json
{
    "gradle" : "gradle-0.9.1-20110112144717-bin.zip",
    "tomcat" : "apache-tomcat-7.0.14.tar.gz",
    "quartz" : "camel-quartz-1.6.0.jar.asc"
}
```

Now you can download your artifacts to a single destination using your version config:
```
$ ./artifactory download -c ./versions.json -d ./test
Fetching repository info: [3/3] Complete!
Downloading to './test'
   ✔ camel-quartz-1.6.0.jar.asc                      [100%] 194.0 B   [1/3]
   ✔ apache-tomcat-7.0.14.tar.gz                     [100%] 6.9 MiB   [2/3]
   ✔ gradle-0.9.1-20110112144717-bin.zip             [100%] 34.3 MiB  [3/3]
Download complete!

$ tree ./test
./test
├── org
│   └── apache
│       ├── camel
│       │   └── camel-quartz
│       │       └── 1.6.0
│       │           └── camel-quartz-1.6.0.jar.asc
│       └── tomcat
│           └── apache-tomcat-7.0.14.tar.gz
└── org.gradle
    └── gradle
        └── zips
            └── gradle-0.9.1-20110112144717-bin.zip
```

Or download artifacts to a single destination:
```
$ ./artifactory download -c ./versions.json -d ./test --flatten
...
[wagoodman@kiwi artifactoryCli]$ tree ./test
./test
├── apache-tomcat-7.0.14.tar.gz
├── camel-quartz-1.6.0.jar.asc
└── gradle-0.9.1-20110112144717-bin.zip
```

Verify your artifacts integrity offline:
```
$ ./artifactory  verify -d ./test
Verifying artifacts...
   ✔ camel-quartz-1.6.0.jar.asc                      ff2bf6333cc0e8f634bec9bc6e37857b5c4dd5bb
   ✔ apache-tomcat-7.0.14.tar.gz                     bd6d0cbdbb9d5a693e9f4603ceb24e1c983c9075
   ✔ gradle-0.9.1-20110112144717-bin.zip             ae4255a754535e19be3f8ec8fb8da3de6b37ed6b
Verification complete!
```

A demo of a failed verification:
```
$ echo "ohhnoozzz" >> ./test/apache-tomcat-7.0.14.tar.gz
$ ./artifactory  verify -d ./test
Verifying artifacts...
   ✔ camel-quartz-1.6.0.jar.asc                       ff2bf6333cc0e8f634bec9bc6e37857b5c4dd5bb
   ✘ apache-tomcat-7.0.14.tar.gz
        expected sha1: bd6d0cbdbb9d5a693e9f4603ceb24e1c983c9075
        got sha1:      44818d0adfe7818cc322e02b079c6dfcb784ea6c
        uri: http://repo.jfrog.org/artifactory/distributions/org/apache/tomcat/apache-tomcat-7.0.14.tar.gz
   ✔ gradle-0.9.1-20110112144717-bin.zip              ae4255a754535e19be3f8ec8fb8da3de6b37ed6b
Verification failed!
```

# General Usage
```
usage: artifactory <command> [<args>]

Commands:

   download     Download and verify one or more modules.
   verify       Verify content of all packages from a local sha1 cache.
   search       Show packages from repositories which match a given name.

DOWNLOAD: Download and verify one or more modules.
    usage: artifactory download -d DESTINATION [-c CONFIG] [-r REPO] [-f]

    optional arguments:
      -h, --help            show this help message and exit
      -d DESTINATION, --destination DESTINATION
                            Directory to store the downloaded files.
      -c CONFIG, --config CONFIG
                            Modules are specified in a configuration.
      -r REPO, --repo REPO  Only download from the given repository.
      -f, --flatten         Flatten directory tree (no sub directories will be
                            created).

VERIFY: Verify content of all packages from a local sha1 cache.
    usage: artifactory verify -d DESTINATION

    optional arguments:
      -h, --help            show this help message and exit
      -d DESTINATION, --destination DESTINATION
                            Directory with the sha1 cache and downloaded files.

SEARCH: Show available versions of an artifact.

    usage: artifactory search [-r REPO] [-i] name

    positional arguments:
      name                  A package name (or partial name) to search for.

    optional arguments:
      -h, --help            show this help message and exit
      -r REPO, --repo REPO  Only search the given repository.
      -i, --info            Get package details.
```
