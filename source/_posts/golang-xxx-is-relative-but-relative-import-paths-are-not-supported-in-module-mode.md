---
title: >-
  golang  xxx is relative, but relative import paths are not supported in module
  mode
date: 2023-06-19 20:26:18
tags:
---
# Golang package management

Go has its own package management system named "go modules". It was introduced in Go 1.11as an official solution for golang's package management. With go modules mode, we no longer need to place our code in **$GOPATH** directory. 

# Go init

In golang modules mode, we cant import our package in our project directly ( like what we do in python). Before we import our package, we need make it become a golang package first. And we should use `go mod init` command to make become a golang package in our project. 

Usage:

```
go mod init [module-path]
```

Example:

```
# run it in "mypackage" folder
go mod init example.com/m
```

If we have a package named "test". Now we can import it from the path **"example.com/m/mypackage/test"**. 

And after we run init command we can get a file named `go.mod` at the root of the project. It defines the module's name, the Go version compatibility, and the required dependencies. The `go.mod` file is automatically created and updated as you add or remove dependencies.

# Solve this problem

The answer is clear now. We shouldn't use relative path to import our package if we want to use module mode. Just create new module and import it.
