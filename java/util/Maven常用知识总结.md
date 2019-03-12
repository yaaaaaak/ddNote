1. clean install整个parent的话，如果子模块很多，要打很久，而且大多数对于你的开发并没有什么用处。因此可以考虑只打基础依赖。具体含义可以查阅相关api，这里不做赘述。

   ```shell
   mvn clean install -pl base -am -DskipTests -f pom.xml
   ```

   

2. 仓库分为本地仓库和远程仓库，本地仓库不赘述，远程仓库分为私服、中央仓库等。操作时，优先级为本地-远程私服-远程中央，达到省资源的目的。

   - nexus只是远程仓库私服的一种
   - 公司内部的包、插件等，大多放在私服

3. 使用distributionManagement配置snapshots和releases仓库，deploy时可按实际需要发布到不同的仓库

   ```shell
   #发布到releases
   mvn deploy -P release
   #默认发布到snapshots
   mvn deploy
   ```

   

