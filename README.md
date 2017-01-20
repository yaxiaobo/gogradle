# Gogradle - a full-featured gradle plugin for building golang
[![Build Status](https://travis-ci.org/blindpirate/gogradle.svg?branch=master)](https://travis-ci.org/blindpirate/gogradle)
[![Coverage Status](https://coveralls.io/repos/github/blindpirate/gogradle/badge.svg?branch=master)](https://coveralls.io/github/blindpirate/gogradle?branch=master)
[![Java 8+](https://img.shields.io/badge/java-8+-4c7e9f.svg)](http://java.oracle.com)
[![Apache License 2](https://img.shields.io/badge/license-APL2-blue.svg)](http://www.apache.org/licenses/LICENSE-2.0.txt)

Don't use me. I'm under development.
My final goal is building complicated golang system such as Kubernetes.

# Golang workspace convention

By default, gogradle will use project directory as workspace instead of standard golang workspace described in
[the documentation](https://golang.org/doc/code.html#Workspaces).
 
It will:

- not touch files in `$GOPATH` at all.
- put the build result into `${projectRoot}/build/gopath/pkg` (library) or `${projectRoot}/build/gopath/bin` (executable).
 
However, you can turn it on easily, with configuration as follows:

```
golang {
    buildTags=['appengine','xxx']
    useGlobalGopath = true
    globalGopath = '/the/GOPATH'  // Only needed if GOPATH environment variable isn't set 
}
```

In this case, gogradle will use your global workspace specified by `$GOPATH`. It will:

- update the `src` directory when necessary as `go get` does.
- put the build result into `pkg` (library) or `bin`(executable). 

# Specify Go version

This is in the plan but not implemented yet.

# Dependency Management

Gogradle provides nearly native gradle DSLs for dependency management.

For example, you can declare your dependencies with following statements.

```
repositories{
    git {
        url 'github.com/user/project'
        credentials {
            username ''
            password ''
        }
    }
    git {
        url {it->it.startsWith('github.com')}
        credentials {
            privateKeyFile '/path/to/private/key'
        }
    }
    git {
        url ~/github\.com.*/
        credentials {
            privateKeyFile '/path/to/private/key'
        }
    }
}

dependencies {
    build 'github.com/user/project'
    build 'github.com/user/project@1.0.0-RELEASE'
    build 'github.com/user/project#d3fbe10ecf7294331763e5c219bb5aa3a6a86e80'
    
    build name: 'github.com/user/project', version: '2.5'
    build name: 'github.com/user/project', tag: 'v1.0.0'
    build name: 'github.com/user/project', commit: 'commitId'
    
    build name: 'github.com/user/project', url:'https://github.com/user/project.git', tag:'v1.0.0', vcs:'git'
    build name: 'github.com/user/project', url:'git@github.com:user/project.git', tag:'v1.0.0', vcs:'git'
    
    build 'github.com/user/project@1.0.0',
            'github.com/user/anotherproject#commitId'
             
    build(
        [name: 'github.com/user/project', version: '2.5'],
        [name: 'github.com/user/project', commit: 'commitId']
    )
    build('github.com/user/project') {
        transitive = true
    }
    build name: 'github.com/user/project', tag: 'v1.0.0', transitive: true
    
    build(name: 'github.com/user/project', tag: 'v1.0.0') {
        transitive = true
        excludeVendor = true
        stategy = 
        exclude module: 'github.com/user/anotherproject'
    }
    
    // build 'github.com/a/b' withDir '${GOPATH}/a/b'
    build dir('${GOPATH}/a/b') asPackage('github.com/a/b') {
    
    }
      
    // This is equivalent to 
    // build(dir('${GOPATH}/a/b')).asPackage('github.com/a/b',{})
}
```

## About Version
A GolangDependency:

- group: rootPath
- name: path
- version: unique identifier to locate a version of code, e.g. Git's commit

The default produce strategy is:

- If external module detected (e.g gopm/govendor/glide, including gogradle itself), it will be used.
- Otherwise, if vendor code exists (there is at least one .go file in the `vendor` directory tree), it will be used.
- Otherwise, an import-scan occurs.

# Special support for Chinese users

This paragraph is for Chinese users who are behind the [GFW](https://en.wikipedia.org/wiki/Great_Firewall) only.

Gogradle有一个特殊的设定，你可以简单地在`golang`块中加入设置`fuckGfw=true`来开启该设定，如下所示：

```
golang {
    // ...其他设置
    fuckGfw = true
}
```

该设定的功能是：Gogradle将会**尽最大努力**绕开GFW（例如下载golang的二进制包）。
注意，这并非自动翻墙。对于依赖包，Gogradle仍然会按照正常的方式访问网络进行import，这一过程中可能受到GFW的干扰。
在这种情况下，Gogradle能够对慢速连接给出警告信息：

```
Access to http://a-very-slow-url.com/package is very slow (0.1KB/s), are you in China?
```



# Why we want to isolate a golang build process

Many (possibly most) people don't want to put all there code together in GOPATH/src. 
They often want to put their own package on `/home/blindpirate/work/` or somewhere else,
and put libraries on other places. Gogradle is made to do this. 


TODOLIST:

- --offline
- how to handle lock file
- exception handle
- log

# Settings
```
golang {
    mode=Develop // Repoducible
    packagePath='github.com/user/project'
    goVersion='1.7'
}
```

# Deal with settings.gradle

// Auto-generated by gogradle. You should never modify this file manually.
ext{
    build name:'github.com/a/b',url:'https://github.com/a/b/git',vcs:'git',commit:'commitId',hash:'xxxx'
} 

# Gogradle tasks

## build

`gradlew build` is equivalent to `go build ${projectRoot}`. The [docmentation](https://golang.org/cmd/go/) says the following:

> When compiling a single main package, build writes the resulting 
executable to an output file named after the first source file 
('go build ed.go rx.go' writes 'ed' or 'ed.exe') or the source code directory
 ('go build unix/sam' writes 'sam' or 'sam.exe'). 
 The '.exe' name is added when writing a Windows executable.
...
When compiling multiple packages or a single non-main package,
 build compiles the packages but discards the resulting object,
  serving only as a check that the packages can be built.
  
Note: in case that main package locates in the `${projectRoot}`, the result executable
  will be installed to `${projectRoot}/build/`

## install

`gradlew install` is equivalent to `go install ${projectRoot}`. The [documentation](https://golang.org/cmd/go/) says:

> Install compiles and installs the packages named by the import paths, along with their dependencies.

The only difference is that the result will be put into `${projectRoot}/build/gopath`.
For example, you have a gogradle-managed package named `example.com/foo/bar`, after `gradlew install`, the result will
be located in `${projectRoot}/build/gopath/bin/` (main package) or `${projectRoot}/build/gopath/pkg` (non-main package).

## clean

## test

## check


# Migrate from other package management tools

If you're using an external package management tool, don't worry. Migration to Gogradle is very simple.  
All you need to do is checking out [this project] to your project root then running `./gradlew migrate`.


# Dependency Lock

When dependency resolution completes, Gogradle will try to lock the dependencies. 
It will flatten the dependency tree and write all dependencies to the end of `${projectRoot}/settings.gradle`.
See [gradle documentation](https://docs.gradle.org/current/userguide/build_lifecycle.html#sec:settings_file) for more details.

You should never modify these dependencies manually.

When a build process start, gogradle will check if locked dependencies exist.
In `Develop` mode, dependency declarations in `build.gradle` have priority over those in `settings.gradle` 

```
// Auto-generated by gogradle. You should NEVER modify it manually and keep it at the end of this file.
gradle.ext.lock=[
[:],
[:]
]
```
# Roadmap

## Multi VCS support
## Full platform support
## Without go installed
## IDE intergration 

# TODO LIST

- how to handle GOPATH? - ignore global GOPATH totally
- lockfile use yaml
- exceptions & logs
- unresolvable packages and its vendor path
- same package in vendor tree
- Develop/Reproducible switch
- exclude and transitive

# problems met

- some ssl cert can only be loaded in newest jdk





