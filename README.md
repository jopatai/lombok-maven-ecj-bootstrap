# Lombok Maven+ECJ bootstrap agent

This is a simple Java agent that makes building a project with Lombok via Maven using 
the ECJ compiler simpler. Since Lombok must run as a Java agent in order to hook 
into ECJ properly, it must be present in the command line that starts Maven. This 
requires a project to either include lombok.jar in the source repository, or specify 
a non-portable or multi-step build process that is not wholly contained within the 
pom.xml.

Use of the bootstrap agent keeps lombok.jar as a dependency declaration in the pom.xml 
instead of in source control, and does not require multiple Maven executions to build 
a project.

## Usage

1. Maven 3.3.1+ allows JVM customization per project by adding a file named `.mvn/jvm.config` 
   to the project. So, create this file, and add the bootstrap agent along side it (yes, 
   this is still a binary in source control, but it is less than 4KB and unlikely to 
   need to be updated). 
   ```none
      ├── project-root
      │   ├── .mvn
      │   │   ├── jvm.config
      │   │   └── lombok-bootstrap.jar
      │   ├── src
   ```
2. The contents of jvm.config is a single line: `-javaagent:.mvn/lombok-bootstrap.jar`.
3. Modify your pom.xml to configure Lombok (demonstrated here using Tycho, but Plexus 
   would work as well):
   ```xml
      <dependencies>
        <dependency>
          <groupId>org.projectlombok</groupId>
          <artifactId>lombok</artifactId>
          <scope>provided</scope>
        </dependency>
      </dependencies>
      <build>
        <plugins>
          <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
              <compilerId>jdt</compilerId>
            </configuration>
            <dependencies>
              <dependency>
                <groupId>org.eclipse.tycho</groupId>
                <artifactId>tycho-compiler-jdt</artifactId>
                <version>${tycho-version}</version>
              </dependency>
              <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok-version}</version>
              </dependency>
            </dependencies>
          </plugin>
        </plugins>
      </build>
   ```
   Note especially that lombok must be declared as a dependency to the compiler plugin 
   and not as an annotation processor.

## Explanation

At runtime, the bootstrap agent simply watches for the maven-compiler-plugin to be 
loaded. When this happens, it finds the declared dependency to lombok and loads it 
as a Java agent as well.


