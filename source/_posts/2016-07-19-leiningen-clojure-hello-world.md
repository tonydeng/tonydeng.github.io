layout: post
title: Leiningen + Clojure Hello World!
published: true
comments: true
sharing: true
footer: true
categories: [Leiningen + Clojure之旅]
tags: [leiningen, clojure]
date: 2016-07-19 09:01:58
keywords: Leiningen + Clojure Hello World!
---

![hello world](/images/blog/hello-world.jpg)

我们之前已经写过[Leiningen + Clojure之旅](/categories/Leiningen-Clojure之旅/)一篇Blog[安装Leiningen](/2015/11/19/install-leiningen/)。

今天我们在看看如何结合Leiningen写一个Clojure的Hello World，毕竟开始一门语言，都是从Hello World开始。

<!-- more -->

## 创建项目

我们还是先用lein创建一个项目，使用标准项目模板。

```bash
➜ lein new app clojure-demo --force
Generating a project called clojure-demo based on the 'app' template.
```

这儿有关于new这个指令的详细使用介绍，包括项目模板的介绍。

```bash
➜ lein help new
Generate scaffolding for a new project based on a template.

If only one argument is passed to the "new" task, the default template
is used and the argument is used as the name of the project.

If two arguments are passed, the first should be the name of a template,
and the second is used as the name of the project, for example:

    lein new $TEMPLATE_NAME $PROJECT_NAME

To generate to a directory different than your project's name use --to-dir:

    lein new $TEMPLATE_NAME $PROJECT_NAME --to-dir $DIR

By default, the "new" task will not write to an existing directory.
Supply the --force option to override this behavior:

    lein new $TEMPLATE_NAME $PROJECT_NAME --force
    lein new $TEMPLATE_NAME $PROJECT_NAME --to-dir $DIR --force

Arguments can be passed to templates by adding them after "new"'s options. Use
`--` to separate arguments to lein new and the actual template you are using:

    lein new $TEMPLATE_NAME $PROJECT_NAME --to-dir $DIR -- template-arg-1 template-arg-2

If you'd like to use an unreleased (ie, SNAPSHOT) template, pass in --snapshot:

    lein new $TEMPLATE_NAME $PROJECT_NAME --snapshot

If you'd rather like to use a specific version of template, specify the version
with --template-version option:

    lein new $TEMPLATE_NAME $PROJECT_NAME --template-version $TEMPLATE_VERSION

If you use the `--snapshot` or `--template-version` argument with template args
you may need to use `--` to prevent template args from being interpreted as
arguments to `lein new`:

    lein new $TEMPLATE_NAME $PROJECT_NAME --snapshot -- template-arg-1 template-arg-2

Third-party templates can be found at https://clojars.org/search?q=lein-template.
When creating a new project from a third-party template, use its group-id
as the template name. Note that there's no need to "install" a given third-
party template --- lein will automatically fetch it for you.

Use `lein new :show $TEMPLATE` to see details about a given template.

To create a new template of your own, see the documentation for the
lein-new Leiningen plug-in.

Subtasks available:
default    A general project template for libraries.
plugin     A leiningen plugin project template.
app        An application project template.
template   A meta-template for 'lein new' templates.

Run `lein help new $SUBTASK` for subtask details.

Arguments: ([project-name] [template project-name [-- & args]])
```

## project.clj说明

`project.clj`是`Leigingen`为项目添加的配置文件，类似于`Maven`的`pom.xml`。我们来先看看都有些什么内容。

```clojure
(defproject clojure-demo "0.1.0-SNAPSHOT"
  :description "FIXME: write description"
  :url "http://example.com/FIXME"
  :license {:name "Eclipse Public License"
            :url "http://www.eclipse.org/legal/epl-v10.html"}
  :dependencies [[org.clojure/clojure "1.8.0"]]
  :main ^:skip-aot clojure-demo.core
  :target-path "target/%s"
  :profiles {:uberjar {:aot :all}})
```

1. 第一行，描述了项目名称以及版本，当前的项目名称就是 `clojure-demo`,版本是`0.1.0-SNAPSHOT`。如果一个项目版本以`SNAPSHOT`结尾，通常表明该项目还处于开发阶段，还没有正式的release。
1. `description`是该项目的简要描述。
1. `url`是可选的网址，可以是你项目将来上线后的实际地址。
2. `license`是该项目使用License，默认给你设置为`Eclipse Public License`。
3. `dependencies`是我们初期最需要关注的内容，也就是我们项目中需要依赖的其他项目及其版本的配置。比如`clojure-demo`这个项目就依赖了clojore-1.8.0的版本。
4. `main`是配置了当前项目执行的文件，比如`clojure-demo.core`这个配置就表明了，执行文件就在`src/clojure-demo/core.clj`。
5. `profiles`是我们自己个性化定制的不同profile的指令，比如，我们现在可以通过`lein uberjar`将项目生成一个jar供其他项目使用。


## 结合IntelliJ

当你用IntelliJ打开`clojure-demo`，发现你根本看不到项目的结构，只能看到最外层的文件，比如`project.clj`等。src,test等目录和实际的代码你根本就看不到。那该怎么办？

可以先执行下面的命令：

```bash
➜ lein pom
```

这样lein帮你生成了一个`pom.xml`，你就可以按照Maven的方式import到IntelliJ中了。

## 执行Hello World!

好，项目也生成了，IDE也可以使用了，我们就开始经典的`Hello World`吧。

我们在`project.clj`中配置了直接运行的方式，那我们现在开始编辑一下`core.clj`。


```clojure
(ns clojure-demo.core
  (:gen-class))

(defn -main
  "I don't do a whole lot ... yet."
  [& args]
  (println "Hello, Clojure World!"))
```

执行一下看看效果

```bash
➜ lein run
Hello, Clojure World!
```

## 执行测试

我们先添加一个简单的方法，然后测试一下。

在`core.clj`中添加`my-plus`方法，简单计算一下两个数的相加

```clojure
(defn my-plus
  "I don't do a whole lot."
  [a b]
  (+ a b))
```

在`core_test.clj`中添加`my-plus-test`的测试方法，测试一下1+1是否等于2

```clojure
(deftest my-plus-test
  (testing "Test my plus."
    (is (= (my-plus 1 1) 2)))
  )
```

```bash
➜ lein test

lein test clojure-demo.core-test

Ran 1 tests containing 1 assertions.
0 failures, 0 errors.
```

如果，你做到这一步的话，欢迎你来到Leiningen + Clojure的世界，Java工程师也可以体验一把Clojure带给你的不同体验吧。
