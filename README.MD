# This small project demonstrates an issue with #jsonschema2pojo-maven-plugin when maven build is run with JDK9

## Environment settings

### JDK 1.8

$ java -version
java version "9-ea"
Java(TM) SE Runtime Environment (build 9-ea+131)
Java HotSpot(TM) 64-Bit Server VM (build 9-ea+131, mixed mode)

$ mvn -v
Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-10T18:41:47+02:00)
Maven home: C:\Casino\apache-maven-3.3.9
Java version: 1.8.0_92, vendor: Oracle Corporation
Java home: C:\Casino\jdk1.8.0_92\jre
Default locale: en_US, platform encoding: Cp1257
OS name: "windows 8.1", version: "6.3", arch: "amd64", family: "dos"

$ mvn -e -X clean install

**RESULT:** Pojo class has been successfully generated

### JDK-9 (EA 131)

$ java -version
java version "9-ea"
Java(TM) SE Runtime Environment (build 9-ea+131)
Java HotSpot(TM) 64-Bit Server VM (build 9-ea+131, mixed mode)

$ mvn -v
Apache Maven 3.3.9 (bb52d8502b132ec0a5a3f4c09453c07478323dc5; 2015-11-10T18:41:47+02:00)
Maven home: C:\Casino\apache-maven-3.3.9
Java version: 9-ea, vendor: Oracle Corporation
Java home: C:\Casino\jdk-9
Default locale: en_US, platform encoding: Cp1257
OS name: "windows 8.1", version: "6.3", arch: "amd64", family: "dos"

$ mvn -e -X clean install --activate-profiles jdk9

**RESULT:** Error

-----------------------------------------------------

	at org.apache.maven.plugin.DefaultBuildPluginManager.executeMojo(DefaultBuildPluginManager.java:166)
	... 21 more
Caused by: java.lang.NoClassDefFoundError: javax/annotation/Generated
	at org.jsonschema2pojo.rules.ObjectRule.addGeneratedAnnotation(ObjectRule.java:362)
	at org.jsonschema2pojo.rules.ObjectRule.apply(ObjectRule.java:111)
	at org.jsonschema2pojo.rules.ObjectRule.apply(ObjectRule.java:70)
	at org.jsonschema2pojo.rules.TypeRule.apply(TypeRule.java:90)
	at org.jsonschema2pojo.rules.TypeRule.apply(TypeRule.java:40)
	at org.jsonschema2pojo.rules.SchemaRule.apply(SchemaRule.java:73)
	at org.jsonschema2pojo.rules.SchemaRule.apply(SchemaRule.java:66)
	at org.jsonschema2pojo.rules.SchemaRule.apply(SchemaRule.java:30)
	at org.jsonschema2pojo.SchemaMapper.generate(SchemaMapper.java:92)
	at org.jsonschema2pojo.Jsonschema2Pojo.generateRecursive(Jsonschema2Pojo.java:109)
	at org.jsonschema2pojo.Jsonschema2Pojo.generate(Jsonschema2Pojo.java:73)
	at org.jsonschema2pojo.maven.Jsonschema2PojoMojo.execute(Jsonschema2PojoMojo.java:594)
	at org.apache.maven.plugin.DefaultBuildPluginManager.executeMojo(DefaultBuildPluginManager.java:134)
	... 21 more
Caused by: java.lang.ClassNotFoundException: javax.annotation.Generated
	at org.codehaus.plexus.classworlds.strategy.SelfFirstStrategy.loadClass(SelfFirstStrategy.java:50)
	at org.codehaus.plexus.classworlds.realm.ClassRealm.unsynchronizedLoadClass(ClassRealm.java:271)
	at org.codehaus.plexus.classworlds.realm.ClassRealm.loadClass(ClassRealm.java:247)
	at org.codehaus.plexus.classworlds.realm.ClassRealm.loadClass(ClassRealm.java:239)
	... 34 more
[ERROR] 
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/PluginContainerException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <goals> -rf :pojo

### _Detailed build log can be found here: **jdk9-build.log**_


## Issue analysis

* Java 9 (Jigsaw) has introduced a new MODULARIZATION concept, when all javax.* package becomes optional at compilation and runtime. See good overview [here](http://mail.openjdk.java.net/pipermail/jdk9-dev/2016-May/004309.html)
* The problem with jsonschema2pojo-maven-plugin could be solved if there will be possible to specify additional JVM arguments inside the plugin configuration section. 
** Compile time. See an example of **maven-compiler-plugin** 
      
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>1.9</source>
                <target>1.9</target>
                <fork>true</fork>
                <meminitial>1024m</meminitial>
                <maxmem>4096m</maxmem>
                <compilerArgs>
                    <arg>-addmods</arg>
                    <arg>java.xml.bind,java.annotations.common</arg>
                </compilerArgs>
             </configuration>
      </plugin>
** Runtime: 
    -Djdk.launcher.addmods=java.xml.bind,java.annotations.common





