---
layout: article
title: Build
subtitle: developer guide
relativePath: ..
---

Building UML Designer update-site
---------------------------------

UML Designer and SysML Designer are both provided as Eclipse plugins but only UML Designer is included in the **UML Designer Product**.

You’ll need Apache Maven 3 to be installed on your computer and you must have [cloned the UML Designer github repository]({{page.relativePath}}/developer-guide/contribute.html#Get_source_code) on your computer.
To launch the build, go to the root of the git repository and type :

`mvn clean package`

Maven will bootstrap itself, download all the dependencies and build UML Designer. The output of the build is a p2 repository (an update site) generated here :

`packaging/org.obeonetwork.dsl.uml2.update/target/repository`

To install it in an Eclipse installation use Help/Install New Software and point to this location.

Launching the test is not harder, just type :

`mvn clean verify`

Maven will launch the tests and give you the result.
Code coverage is captured thanks to Jacoco.

This command builds:

![]({{page.relativePath}}/images/UMLDesignerBuildPlugins.png)

Building UML Designer product
-----------------------------

We have a dedicated command for the product.

`mvn clean package -f releng/org.obeonetwork.dsl.uml2.product.parent/pom.xml`

> You will retrieve product in: **packaging/org.obeonetwork.dsl.uml2.product/target/products/**

This command builds:

![]({{page.relativePath}}/images/UMLDesignerBuildProduct.png)

Maven Tycho
-----------

Technically, we use Maven Tycho to build these Eclipse artifacts.

To reuse a maximum of properties and maven configurations, we have a hierarchy of parents.
All UML designer plugins/features/update-sites refers to a dedicated parent:
![]({{page.relativePath}}/images/UMLDesignerMavenStructure.png)

For example, to add a new plugin org.obeonetwork.dsl.uml2.myplugin, add a
{% highlight XML %}
    <module>${root-path}/plugins/org.obeonetwork.dsl.uml2.myplugin<module>
{% endhighlight %}

in org.obeonetwork.dsl.uml2.parent and add a \`pom.xml\` in your plugin with the following header:
{% highlight XML %}
    <project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
        <modelVersion>4.0.0</modelVersion>
        <parent>
            <groupId>org.obeonetwork.dsl.uml2</groupId>
            <artifactId>parent</artifactId>
            <version>4.0.0-SNAPSHOT</version>
           <relativePath>../../releng/org.obeonetwork.dsl.uml2.parent</relativePath>
        </parent>
        <artifactId>org.obeonetwork.dsl.uml2.myplugin</artifactId>
        <packaging>eclipse-plugin</packaging>
        ...
    </project>
{% endhighlight %}

Continuous integration
----------------------

One continuous integration builds exists to build UML Designer. This build runs on [Travis-ci](https://travis-ci.org/).

### UML-Designer build

#### Building branches

This Travis [build]() https://travis-ci.org/ObeoNetwork/UML-Designer/ is used to :

-   generate the documentation,
-   update the gh-pages (umldesigner.org web site),
-   create the UML Designer update site and products.

This build deploys automatically the build artifacts on [S3](http://aws.amazon.com/fr/s3/) : [http://obeo-umldesigner-nightly.s3-website-eu-west-1.amazonaws.com/](http://obeo-umldesigner-nightly.s3-website-eu-west-1.amazonaws.com)

The build is configured thanks to the [`.travis.yml`](https://github.com/ObeoNetwork/UML-Designer/blob/master/.travis.yml) file.

Each time a commit is integrated on the github repository a nightly build is executed and the resulting update-site and products are deployed on s3:
http://obeo-umldesigner-nightly.s3-website-eu-west-1.amazonaws.com/$BRANCH\_NAME.

The build deploys in S3 `obeo-umldesigner-nightly` bucket:

-   the `repository` folder: the UML Designer nightly update site.
-   the `org.obeonetwork.dsl.uml2.update-nightly.zip` : a zip of the nightly update site.
-   the `bundles`: the nightly products.
-   the `umldesigner-xxx.tpd`: the target platform used to build the update site and products.

The deploy phase is active only for specific branches: [`master`](https://github.com/ObeoNetwork/UML-Designer/tree/master), [`5.0.x`](https://github.com/ObeoNetwork/UML-Designer/tree/5.0.x), [`5.0.x_juno3.8`](https://github.com/ObeoNetwork/UML-Designer/tree/5.0.x_juno3.8).

The [umldesigner.org](http://www.umldesigner.org) web site is also updated and deployed thanks to this build. For more details about the web site have a look to the [documentation section]({{page.relativePath}}/developer-guide/documentation.html) of the developer guide.

#### Pull-requests

When a pull request is sent, a build is automatically launched. In this case, the deploy phase and the web site updates phases are not performed. The pull request build results is directly [visible in the github interface](http://blog.travis-ci.com/2012-09-04-pull-requests-just-got-even-more-awesome/).

### UML-Designer-UI-Tests build

The Travis [UI tests build]() https://travis-ci.org/ObeoNetwork/UML-Designer-UI-Tests is used to launch the UI tests.
This build deploys the tests results on S3:
http://obeo-umldesigner-nightly.s3-website-eu-west-1.amazonaws.com/$BRANCH\_NAME/test-results/$JOB\_NUMBER.

#### Make a release

When a new tag is pushed on the git repository a release is deployed on the S3 : http://obeo-umldesigner-releases.s3-website-eu-west-1.amazonaws.com/$TAG. The same tag must be added on the test project in order that the UI tests are relaunched and the results will be provided in:
http://obeo-umldesigner-releases.s3-website-eu-west-1.amazonaws.com/$TAG/test-results.
