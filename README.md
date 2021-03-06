Taverna workbench
=================

_[Taverna](http://www.taverna.org.uk/) Workbench product build and installers_

Released by [myGrid](http://www.mygrid.org.uk/)

(c) Copyright 2005-2014 University of Manchester, UK


Introduction
------------

This repository is a Maven source code trees that do not themselves contain 
any code, but which are used to assemble the distribution, add launchers and
configurations files, and package this as installers. 

You can think of a product as the last build step before having a 
downloadable/executable software distribution. Each product includes their 
required code from the common 
[Taverna source code](http://dev.mygrid.org.uk/wiki/display/developer/Taverna+source+code), 
by retrieved the compiled JARs from the 
[myGrid Maven repositories](http://dev.mygrid.org.uk/wiki/display/developer/Maven+repository) 
unless they have already been built locally with Maven.

This is the build product for the desktop user interface 
[Taverna Workbench](http://www.taverna.org.uk/download/workbench/).

In most cases you do not need to compile Taverna, instead you can 
download the official distribution for your operating system from the
[Taverna download pages](http://www.taverna.org.uk/download/).

For usage of this product, see the 
[Taverna user manual](http://dev.mygrid.org.uk/wiki/display/taverna/).



Editions
--------

This repository have several "edition" branches, one for each domain-specific builds. 
Each domain-specific build extends the core branch by modifying the plugin set 
(and ensuring the repository folder is populated for those plugins), but may also 
modify configurations such as the default service sets.

Branches:

  *  `core` - for any domain - general activities (WSDL, REST, Beanshell, Component, Interaction, etc) - also basis for the other branches (can be considered the 'master' or 'trunk' branch)
  *  `bioinformatics` - formerly maintenance, adds BioMart, BioMoby, Soaplab, WebDAV, corresponding default services and [BioCatalogue](https://www.biocatalogue.org/)
  *  `digitalpreservation` - core + WebDAV
  *  `biodiversity` - core + WebDAV + [BioDiversityCatalogue](https://www.biodiversitycatalogue.org/)
  *  `astronomy` - core + [AstroTaverna](http://wf4ever.github.io/astrotaverna/)
  *  `enterprise` - core with all possible plugins

To check which edition you are building, verify these bits of `pom.xml`:

        <artifactId>taverna-workbench-${edition}</artifactId>
        <name>Taverna Workbench [${edition}]</name>

        <properties>
                <edition>core</edition>


For more information, see http://dev.mygrid.org.uk/wiki/display/developer/Taverna+products


### Creating a new edition

To create a new edition, make a new branch based on `core` and modify the above line(s).
You would then typically also modify `src/main/resources/plugins/plugins.txt`
and add the `<dependency>` sections from the new plugin(s) to the `pom.xml` so that
the new plugin is added to the generated `repository/` folder. 

Note: You MUST use `<scope>test</scope>` in `pom.xml` for the added
`<dependency>` blocks to avoid adding them to the `lib/` folder, which would
bypass the plugin-system and potentially cause conflicts with other parts of
Taverna.

To verify the build, run `executeworkflow.sh`. A new user home directory
for your edition should be created, e.g.
`~/.taverna-astronomy-2.5-snapshot-20140228t1156/` for the `astronomy` edition.

Often when adding plugins, Maven will not download all the same versions
required at runtime by Taverna's plugin system Raven. The first execution 
of `executeworkflow.sh` will download these to the `repository` folder
of the edition-specific user home directory. 

To avoid this download phase for other users and permanently add these
"missing" pieces, try the equivalent of:

	stain@biggie:~$ cd .taverna-astronomy-2.5-snapshot-20140228t1156/repository
	stain@biggie:~/.taverna-astronomy-2.5-snapshot-20140228t1156/repository$ find . -type f > ~/files
	stain@biggie:~/.taverna-astronomy-2.5-snapshot-20140228t1156/repository$ cat ~/files
	./axis/axis-jaxrpc/1.4/axis-jaxrpc-1.4.pom
	./axis/axis-jaxrpc/1.4/axis-jaxrpc-1.4.jar
	./axis/axis-saaj/1.4/axis-saaj-1.4.jar
	./axis/axis-saaj/1.4/axis-saaj-1.4.pom


There is a Python script that will read `files` from the current directory and show the
XML that needs to be added to `pom.xml`:

	stain@biggie:~$ python src/taverna-workbench-product/src/main/python/generateArtifactItems.py 
                                <artifactItem>
                                    <groupId>axis</groupId>
                                    <artifactId>axis-jaxrpc</artifactId>
                                    <version>1.4</version>
                                    <type>pom</type>
                                    <outputDirectory>${project.build.directory}/repository/axis/axis-jaxrpc/1.4</outputDirectory>
                                </artifactItem>
                                <artifactItem>
                                    <groupId>axis</groupId>
                                    <artifactId>axis-jaxrpc</artifactId>
                                    <version>1.4</version>
                                    <type>jar</type>
                                    <outputDirectory>${project.build.directory}/repository/axis/axis-jaxrpc/1.4</outputDirectory>
                                </artifactItem>
                                <artifactItem>
                                    <groupId>axis</groupId>
                                    <artifactId>axis-saaj</artifactId>
                                    <version>1.4</version>
                                    <type>jar</type>
                                    <outputDirectory>${project.build.directory}/repository/axis/axis-saaj/1.4</outputDirectory>
                                </artifactItem>
                                <artifactItem>
                                    <groupId>axis</groupId>
                                    <artifactId>axis-saaj</artifactId>
                                    <version>1.4</version>
                                    <type>pom</type>
                                    <outputDirectory>${project.build.directory}/repository/axis/axis-saaj/1.4</outputDirectory>
                                </artifactItem>
	

Now copy and paste to add those `artifactItem` blocks within the
beginng (or end) of the `<artifactIems>` block in `pom.xml`. 
On next build, the `repository/` folder of the distribution
should now also include these files, and Taverna would no
longer attempt to download them on startup.

To verify, now the repository folder should be empty:

	mvn clean install
	rm -rf ~/.taverna-commandline-*
	cd target
	unzip *.zip
	cd *SNAPSHOT
	sh executeworkflow.sh
	find ~/.taverna-commandline-*/repository


Licence
=======

Taverna is licenced under the GNU Lesser General Public Licence. (LGPL) 2.1.
See the file LICENSE.txt or http://www.gnu.org/licenses/lgpl-2.1.html for
details.

If the source code was not included in this download, you can download it from
http://www.taverna.org.uk/download/workbench/2-5/#download-source or
http://www.taverna.org.uk/download/source-code/

If you installed a OS-specific distribution of Taverna it may come
bundled with a distribution of OpenJDK. OpenJDK is distributed under the
GNU Public License (GPL) 2.0 w/Classpath exception. See jre/LICENSE.txt or
http://hg.openjdk.java.net/jdk7u/jdk7u/raw-file/da55264ff2fb/LICENSE
for details.

Taverna uses various third-party libraries that are included under compatible
open source licences such as the Apache Licence.

  
  
Building
========
First, make sure you have checked out the edition you want to build. 

Build requirements:
 * Java [JDK 7](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html) or OpenJDK 7
 * [Apache Maven 3.x](http://maven.apache.org/download.cgi)
 * [Install4j](http://www.ej-technologies.com/products/install4j/overview.html) (optional)


The below will assume you want to build the `core` edition. 

    git fetch --all          # fetch all branches/editions
    git checkout core        # replace 'core' with your edition
    mvn clean install
    
The packaging will take several minutes. The first time, this might take up to an
hour as several libraries are downloaded from Maven repositories.

    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 5:39.145s
    [INFO] Finished at: Thu Feb 27 01:06:08 GMT 2014
    [INFO] Final Memory: 47M/1200M
    [INFO] ------------------------------------------------------------------------


After a successful build, the file 
`target/taverna-commandline-core-2.5-SNAPSHOT.zip` (or equivalent) contains
the platform-independent distribution of the Taverna Command Line Tool. 


Installers
----------

If you would like to build the platform-specific installers, you will need
an installation of 
[Install4j](http://www.ej-technologies.com/products/install4j/overview.html). 

First configure Maven to find your Install4j installation.

    stain@biggie:~$ cat .m2/settings.xml 
    <settings>
      	<profiles>
      	    <profile>
      		<id>dist</id>
        		<properties>
        		    <install4j.home>/opt/install4j5</install4j.home>
    		    </properties>
  	        </profile>
  	    </profiles>
      </settings>

Start `install4j` to install your license key or run a 30-day trial, then quit the user interface.

To generate the installers, run:

    mvn clean install -Pdist

This process can take anything from 2 minutes to 30 minutes depending on your
hardware. You can speed up the process by adding 
[install4j-maven-plugin](http://sonatype.github.io/install4j-support/install4j-maven-plugin/compile-mojo.html)
parameters, for instance generating only the Windows 64-bit installer with
`-Dinstall4j.buildIds=winx64`. The buildIds are:

  * winx64 (Windows .exe 64-bit)
  * wini386 (Windows .exe 32-bit)
  * osx (Mac OS X)
  * linux_amd64 (Debian .deb x64)
  * linux_x86_64 (Redhat .rpm x64) 
  * standalone (Windows .zip)
  * unix.tar (Unix .tar.gz)

Note that on first execution, this will also download [OpenJDK binaries](http://build.mygrid.org.uk/openjdk/) 
for embedding in OS-specifc installers. OpenJDK is licensed as GPL 2.1 with Classpath exception.

The `media/media` folder will contain installers for different operating systems.

	md5sums
	output.txt
	taverna-workbench-core-2.5-SNAPSHOT-linux_amd64.deb
	taverna-workbench-core-2.5-SNAPSHOT-linux_x86_64.rpm
	taverna-workbench-core-2.5-SNAPSHOT-macos.tgz
	taverna-workbench-core-2.5-SNAPSHOT-unix.tar.gz
	taverna-workbench-core-2.5-SNAPSHOT-windows-x64.exe
	taverna-workbench-core-2.5-SNAPSHOT-windows-x86.exe
	taverna-workbench-core-2.5-SNAPSHOT.zip
	updates.xml


Icons
-----

If you would like to customize the icons and splashscreen used by the generated
installers and launchers, see `src/main/resources/lib` - note that there are
different icon files for different operating systems.

For more information on Taverna installers, 
see http://dev.mygrid.org.uk/wiki/display/developer/Taverna+installers

Support
=======

See http://www.taverna.org.uk/about/contact-us/ for contact details.

You may email support@mygrid.org.uk for any questions on using Taverna
workbench. myGrid's support team should respond to your query within a
week.

