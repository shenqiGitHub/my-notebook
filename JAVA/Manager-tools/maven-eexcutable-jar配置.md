自定义

```xml
<plugin>  

    <groupId>org.springframework.boot</groupId>  

    <artifactId>spring-boot-maven-plugin</artifactId>  

    <version>${spring-boot.version}</version>  

    <configuration>  

        <executable>true</executable>  

        <embeddedLaunchScriptProperties>  

            <inlinedConfScript>${basedir}/inlined-conf.script</inlinedConfScript>  

        </embeddedLaunchScriptProperties>  

        <embeddedLaunchScript>${basedir}/launch.script</embeddedLaunchScript>  

    </configuration>  

    <executions>  

        <execution>  

            <goals>  

                <goal>repackage</goal>  

            </goals>  

        </execution>  

    </executions>  

</plugin>
```

对于具体的Springboot相关部分详见

springboot reference   的3.2.3. Customizing the Startup Script章节。里面有对inlinedConfScript脚本属性的相关说明