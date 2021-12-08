---
title: 打包 Pulsar Function
id: function-package
category: user-guides
---

对 Pulsar Function 进行开发和测试后，需要将其打包，以便将其提交到 Pulsar 集群。本文介绍如何打包 Java、Python 和 Go 函数。

# 前提条件

- Apache Pulsar 2.8.0 或更高版本

- Function Mesh v0.1.3 或更高版本

# 打包 Java 函数

可以将 Java 函数打包到 NAR/JAR 包或者 Docker 镜像。

## 构建 Java 函数包

按照如下步骤操作，构建 Java 中的函数打包。 

1. 使用 `pom.xml` 文件创建新的 Maven 项目。下面的代码示例中，`mainClass` 的值是你打包的包名。

    ```Java
    &lt;?xml version="1.0" encoding="UTF-8"?>
    &lt;project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        &lt;modelVersion>4.0.0&lt;/modelVersion>
        &lt;groupId>java-function&lt;/groupId>
        &lt;artifactId>java-function&lt;/artifactId>
        &lt;version>1.0-SNAPSHOT&lt;/version>
        &lt;dependencies>
            &lt;dependency>
                &lt;groupId>org.apache.pulsar&lt;/groupId>
                &lt;artifactId>pulsar-functions-api&lt;/artifactId>
                &lt;version>2.6.0&lt;/version>
            &lt;/dependency>
        &lt;/dependencies>
        &lt;build>
            &lt;plugins>
                &lt;plugin>
                    &lt;artifactId>maven-assembly-plugin&lt;/artifactId>
                    &lt;configuration>
                        &lt;appendAssemblyId>false&lt;/appendAssemblyId>
                        &lt;descriptorRefs>
                            &lt;descriptorRef>jar-with-dependencies&lt;/descriptorRef>
                        &lt;/descriptorRefs>
                        &lt;archive>
                        &lt;manifest>
                            &lt;mainClass>org.example.test.ExclamationFunction&lt;/mainClass>
                        &lt;/manifest>
                    &lt;/archive>
                    &lt;/configuration>
                    &lt;executions>
                        &lt;execution>
                            &lt;id>make-assembly&lt;/id>
                            &lt;phase>package&lt;/phase>
                            &lt;goals>
                                &lt;goal>assembly&lt;/goal>
                            &lt;/goals>
                        &lt;/execution>
                    &lt;/executions>
                &lt;/plugin>
                &lt;plugin>
                    &lt;groupId>org.apache.maven.plugins&lt;/groupId>
                    &lt;artifactId>maven-compiler-plugin&lt;/artifactId>
                    &lt;configuration>
                        &lt;source>8&lt;/source>
                        &lt;target>8&lt;/target>
                    &lt;/configuration>
                &lt;/plugin>
            &lt;/plugins>
        &lt;/build>
    &lt;/project>
    ```

2. 写入 Java 函数。

    ```java
    package org.example.test;
    import java.util.function.Function;
    public class ExclamationFunction implements Function&lt;String, String> {
        @Override
        public String apply(String s) {
            return "This is my function!";
        }
    }
    ```

3. 打包 Java 函数。

    ```bash
    mvn package
    ```

    Java 函数打包后，会自动创建一个 `target`  目录。打开 `target` 目录，查看是否有类似 `java-function-1.0-SNAPSHOT.jar` 的 JAR 包。

然后可以将包推送到包管理系统中，通过指定上传的 Function 包来定义 Function CRD，并将这些 Function 提交到 Pulsar 集群。

## 构建 Docker 镜像

按照如下步骤创建 Docker 镜像。 

1. 打包 Pulsar function。详细信息参见[打包 Pulsar function](#package-java-functions)。

2. 定义 `Dockerfile`。

    下面的例子显示如何使用 Java 函数的 JAR 包 (`example-function.jar`) 来定义 `Dockerfile`。

    ```dockerfile
    # Use pulsar-functions-java-runner since we pack Java function
    FROM streamnative/pulsar-functions-java-runner:2.7.1
    # Copy function JAR package into /pulsar directory  
    COPY example-function.jar /pulsar/
    ```

然后将 Function Docker 镜像推送到镜像注册中心（如 [Docker Hub](https://hub.docker.com/)，或任意私有的注册中心），并使用Function Docker 镜像来配置并将 Function 提交到 Pulsar 集群。

# 打包 Python 函数 

可以将 Python 函数打包成外部包（One Python 文件或 ZIP 文件）或 Docker 镜像。 

## 打包 Python 函数包

本节描述如何为 Python 函数构建包。 

Python 函数支持 One Python 文件或 ZIP 文件。

- One Python 文件

- ZIP 文件

### One Python 文件

如下示例介绍如何使用**One Python 文件** 打包 Python 函数。 

    ```python
    from pulsar import Function //  import the Function module from Pulsar
    # The classic ExclamationFunction that appends an exclamation at the end
    # of the input
    class ExclamationFunction(Function):
      def __init__(self):
        pass
      def process(self, input, context):
        return input + '!'
    ```
    
    In this example, when you write a Python Functions, you need to inherit the Function class and implement the `process()` method.
    
    The `process()` method mainly has two parameters:
    
    - `input` represents your input.
    
    - `context` represents an interface exposed by the Pulsar Function. You can get the attributes in the Python Functions based on the provided context object.

### ZIP 文件

    To package a Python Functions with the **ZIP file** in Python, you need to prepare a ZIP file. The following is required when packaging the ZIP file of the Python Function.
    
    Assume the zip file is named as `func.zip`, unzip the `func.zip` folder:
    
    ```text
        "func/src"
        "func/requirements.txt"
        "func/deps"
    ```
    
    Take [exclamation.zip](https://github.com/apache/pulsar/tree/master/tests/docker-images/latest-version-image/python-examples) as an example. The internal structure of the example is as follows.
    
    ```text
    .
    ├── deps
    │   └── sh-1.12.14-py2.py3-none-any.whl
    └── src
        └── exclamation.py
    ```

然后将包推送到包管理系统中，通过指定上传的 Function 包来定义 Function CRD，并将这些 Function 提交到 Pulsar 集群。

## 构建 Docker 镜像

按照如下步骤操作构建 Docker 镜像。

1. 打包 Python 函数。细节参见[打包 Pulsar 函数](#package-python-functions)。

2. 定义 `Dockerfile`。

    下面的例子显示如何使用 Python 函数的 JAR 包 (`example-function.jar`) 来定义 `Dockerfile`。

    ```dockerfile
    
    # Use pulsar-functions-python-runner since we pack python function
    
    FROM streamnative/pulsar-functions-python-runner:2.7.1
    
    # Copy function JAR package into /pulsar directory  
    
    COPY example-function.jar /pulsar/
    
    ```

然后将 Function Docker 镜像推送到镜像注册中心（如 [Docker Hub](https://hub.docker.com/)，或任意私有的注册中心），并使用Function Docker 镜像来配置并将 Function 提交到 Pulsar 集群。

# 打包 Go 函数

可以将 Go 函数打包到外部包或者 Docker 镜像。 

## 构建 Go 函数包

按照如下步骤操作在 Go 环境中打包 Go 函数。 

1. 写入 Go 函数。

    目前 Go 函数可以**仅**使用 SDK 实现，并且函数的接口以 SDK 的形式公开。使用 Go 函数前，需要先导入 `github.com/apache/pulsar/pulsar-function-go/pf`。

    ```go
    import (
        "context"
        "fmt"
        "github.com/apache/pulsar/pulsar-function-go/pf"
    )
    func HandleRequest(ctx context.Context, input []byte) error {
        fmt.Println(string(input) + "!")
        return nil
    }
    func main() {
        pf.Start(HandleRequest)
    }
    ```

    写入 Go 函数时需要记住如下几点： 

    - 在 `main()` 中，**只需要**将函数名称注册到 `Start()`。在 `Start()`  中**只**接收函数名。

    - Go 函数使用 Go 反射，根据接收到的函数名来验证参数列表和返回值列表是否正确。参数列表和返回值列表**必须**是以下示例函数之一：

      ```go
       func ()
       func () error
       func (input) error
       func () (output, error)
       func (input) (output, error)
       func (context.Context) error
       func (context.Context, input) error
       func (context.Context) (output, error)
       func (context.Context, input) (output, error)
      ```

    可以使用上下文连接到 Go 函数。 

    ```go
    if fc, ok := pf.FromContext(ctx); ok {
        fmt.Printf("function ID is:%s, ", fc.GetFuncID())
        fmt.Printf("function version is:%s\n", fc.GetFuncVersion())
    }
    ```

2. 构建 Go 函数。

    ```bash
    go build &lt;your Go Function filename>.go 
    ```

然后将包推送到包管理系统中，通过指定上传的 Function 包来定义 Function CRD，并将这些 Function 提交到 Pulsar 集群。 

## 构建 Docker 镜像

按照如下步骤构建 Docker 镜像。

1. 打包 Go 函数。详情参见[打包 Pulsar Function](#package-go-functions)。

2. 定义 `Dockerfile`。

    如下示例介绍如何使用 Go 函数的 JAR 包（`example-function.jar`）定义 `Dockerfile`。

    ```dockerfile
    # Use pulsar-functions-go-runner since we pack Go function
    FROM streamnative/pulsar-functions-go-runner:2.7.1
    # Copy function JAR package into /pulsar directory  
    COPY example-function.jar /pulsar/
    ```

然后将 Function Docker 镜像推送到镜像注册中心（如 [Docker Hub](https://hub.docker.com/)，或任意私有的注册中心），并使用 Function Docker 镜像来配置并将 Function 提交到 Pulsar 集群。
