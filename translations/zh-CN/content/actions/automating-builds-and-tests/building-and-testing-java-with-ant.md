---
title: 使用 Ant 构建和测试 Java
intro: 您可以在 GitHub Actions 中创建持续集成 (CI) 工作流程，以使用 Ant 构建和测试 Java 项目。
redirect_from:
  - /actions/language-and-framework-guides/building-and-testing-java-with-ant
  - /actions/guides/building-and-testing-java-with-ant
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
type: tutorial
topics:
  - CI
  - Java
  - Ant
shortTitle: 构建和测试 Java & Ant
---

{% data reusables.actions.enterprise-beta %}
{% data reusables.actions.enterprise-github-hosted-runners %}

## 简介

本指南介绍如何使用 Ant 构建系统为 Java 项目创建执行持续集成 (CI) 的工作流程。 您创建的工作流程将允许您查看拉取请求提交何时会在默认分支上导致构建或测试失败； 这个方法可帮助确保您的代码始终是健康的。 您可以扩展 CI 工作流程以从工作流程运行上传构件。

{% ifversion ghae %}
{% data reusables.actions.self-hosted-runners-software %}
{% else %}
{% data variables.product.prodname_dotcom %} 托管的运行器有工具缓存预安装的软件，包括 Java Development Kits (JDKs) 和 Ant。 有关软件以及 JDK 和 Ant 预安装版本的列表，请参阅 [{% data variables.product.prodname_dotcom %} 托管的运行器的规格](/actions/reference/specifications-for-github-hosted-runners/#supported-software)。
{% endif %}

## 基本要求

您应该熟悉 YAML 和 {% data variables.product.prodname_actions %} 的语法。 更多信息请参阅：
- "[{% data variables.product.prodname_actions %} 的工作流程语法](/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions)"
- "[了解 {% data variables.product.prodname_actions %}](/actions/learn-github-actions)"

建议您对 Java 和 Ant 框架有个基本的了解。 更多信息请参阅“[Apache Ant 手册](https://ant.apache.org/manual/)”。

{% data reusables.actions.enterprise-setup-prereq %}

## Using the Ant starter workflow

{% data variables.product.prodname_dotcom %} provides an Ant starter workflow that will work for most Ant-based Java projects. For more information, see the [Ant starter workflow](https://github.com/actions/starter-workflows/blob/main/ci/ant.yml).

To get started quickly, you can choose the preconfigured Ant starter workflow when you create a new workflow. 更多信息请参阅“[{% data variables.product.prodname_actions %} 快速入门](/actions/quickstart)”。

您也可以通过在仓库的 `.github/workflow` 目录中创建新文件来手动添加此工作流程。

{% raw %}
```yaml{:copy}
name: Java CI

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Set up JDK 11
        uses: actions/setup-java@v2
        with:
          java-version: '11'
          distribution: 'adopt'
      - name: Build with Ant
        run: ant -noinput -buildfile build.xml
```
{% endraw %}

此工作流程执行以下步骤：

1. `checkout` 步骤在运行器上下载仓库的副本。
2. `setup-java` 步骤配置 Adoptium 的 Java 11 JDK。
3. “使用 Ant 构建”步骤以非交互模式运行 `build.xml` 中的默认目标。

The default starter workflows are excellent starting points when creating your build and test workflow, and you can customize the starter workflow to suit your project’s needs.

{% data reusables.actions.example-github-runner %}

{% data reusables.actions.java-jvm-architecture %}

## 构建和测试代码

您可以使用与本地相同的命令来构建和测试代码。

初学者工作流程将运行 _build.xml_ 文件中指定的默认目标。  默认目标通常设置为将类、运行测试和包类设置为其可分发格式，例如 JAR 文件。

如果使用不同的命令来构建项目，或者想要运行不同的目标，则可以指定这些命令。 For example, you may want to run the `jar` target that's configured in your `_build-ci.xml_` file.

{% raw %}
```yaml{:copy}
steps:
  - uses: actions/checkout@v2
  - uses: actions/setup-java@v2
    with:
      java-version: '11'
      distribution: 'adopt'
  - name: Run the Ant jar target
    run: ant -noinput -buildfile build-ci.xml jar
```
{% endraw %}

## 将工作流数据打包为构件

构建成功且测试通过后，您可能想要上传生成的 Java 包作为构件。 这会将构建的包存储为工作流程运行的一部分，并允许您下载它们。 构件可帮助您在拉取请求合并之前在本地环境中测试并调试它们。 更多信息请参阅“[使用构件持久化工作流程](/actions/automating-your-workflow-with-github-actions/persisting-workflow-data-using-artifacts)”。

Ant 通常会在 `build/jar` 目录中创建 JAR、EAR 或 WAR 等输出文件。 您可以使用 `upload-artifact` 操作上传该目录的内容。

{% raw %}
```yaml{:copy}
steps:
  - uses: actions/checkout@v2
  - uses: actions/setup-java@v2
    with:
      java-version: '11'
      distribution: 'adopt'

  - run: ant -noinput -buildfile build.xml
  - uses: actions/upload-artifact@v3
    with:
      name: Package
      path: build/jar
```
{% endraw %}
