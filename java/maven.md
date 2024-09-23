# Maven 构建优化

在 Maven 构建期间优化 Java 编译器是一个很重要的问题，因为它可以显著提高构建速度和性能。以下是一些建议和最佳实践，以帮助您在 Maven 构建期间优化 Java 编译器：

## 依赖管理

在 `pom.xml` 文件中，尽量避免引入过多不必要的依赖。只引入项目需要的依赖，可以减少项目构建和运行时的资源消耗。定期检查项目依赖的版本，保持依赖库的最新版本，以获得最新的性能优化和 bug 修复。

## 编译优化

1. 使用并行构建：通过使用 `-T` 选项，您可以在 Maven 构建期间使用多个线程并行编译Java代码。例如，使用 `-T 4` 可以使用4个线程并行编译。
2. 使用增量编译：通过使用 Incremental Build，您可以仅编译自上次构建以来已更改的源代码文件。这可以显著减少构建时间。要启用 Incremental Build，请在 `pom.xml` 文件中添加以下配置： 

```xml
 <plugins>
   <plugin>
     <groupId>org.apache.maven.plugins</groupId>
     <artifactId>maven-compiler-plugin</artifactId>
     <version>3.8.1</version>
     <configuration>
       <source>1.8</source>
       <target>1.8</target>
        <useIncrementalCompilation>true</useIncrementalCompilation>
      </configuration>
    </plugin>
  </plugins>
</build>
```

默认情况下，`mvn compile` 命令已经是增量的，它只编译那些自上次编译以来已经变更的源文件。

如果需要强制进行全编译，可以使用 `-U` 参数，强制 Maven 更新依赖信息，并且使用 `-o` 参数，进入离线模式，不去远程仓库下载依赖。

Maven 缺省的就是增量编译。Java 的项目通常正式的 build 不能用增量编译, 原因很简单, Maven 和 Ant 都不支持减量编译：如果删除 .java 文件，编译结果 .class 文件将不会被删除, 而 Java 支持运行期动态加载，这样被删除的文件的 class 也可能在运行时被使用, 结果可能是灾难性的。所以我们的 build 都是用 `mvn clean install`， 先清除再编译。不过如果确定没有删除文件或者被删除文件的 .class 文件不会被使用，我个人觉得可以用增量编译`mvn install`。

如果想加快编译, 可以考虑这几个方面：

1. 忽略 Maven 生命周期中的某些阶段，比如：`mvn install -Dmaven.test.skip=true` 跳过 TestCase 检验，否则在 install 时会运行 TestCase 测试。
2. 修改 pom 文件来删减 maven plugin 的执行。 比如：从 pom 中去掉打 source jar 的 plugin, 这样最后 build 的结果不包含源代码。
   3) 冒险用增量编译。 过去一年里我们项目影响增量编译的删掉的文件大概10多个，主要源于需求变化和代码重构。和一年365天相比，这个概率还是比较低的。

## JVM 参数优化

1. 调整堆内存大小： 根据项目的需求和实际情况，调整 JVM 的堆内存大小，避免内存溢出或者过大的内存占用。可以在启动命令中添加参数 `-Xms512m -Xmx1024m`。
2. GC 调优：根据项目的特点，选择合适的垃圾回收算法和参数进行调优，以减少 GC 对程序性能的影响。

## 缓存优化

1. Maven 本地仓库：合理配置 Maven 本地仓库，避免重复下载依赖，减少网络开销和构建时间。
2. 使用插件如 `maven-compiler-plugin` 的增量编译功能，避免每次都重新编译整个项目。

# 参考链接

[1]: https://blog.csdn.net/nalanxiaoxiao2011/article/details/130349372	"Maven 常用插件"
[2]: https://springdoc.cn/maven-fast-build/	"如何提高 Maven 构建速度"

