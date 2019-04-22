---
layout: mypost
title: NPM Package Publish
categories: [NodeJS]
---

# NPM Package Publish

## Registry

存放npm package的仓库。

![npm源](/Users/pinyzhang/Desktop/npm-registry.png)

## Scope

Npm上目前有数十万Package，当发布新的package时，可能会和Registry中已存在的Package命名相同，此时需要添加Scope。若发布私有Package时也需要添加Scope。

Scope格式为@somescope，最终发布的Package命名为 @somescope/somepackagename。

Scope的命名必须是当前用户名或者当前用户加入的组织名。

## Publish Package

```sh
// 登录
npm adduser
// npm login

// 初始化项目
npm init

// 发布
npm publish
```

## Publish JD Package

```sh
npm login --registry=http://registry.m.jd.com/ --scope=@jd

// 初始化项目
npm init --scope=@jd

npm publish --registry=http://registry.m.jd.com/
```

## Unpublish

```sh
// npm
npm unpublish [pkg] --force

// JD
// 联系管理员(xiaoshuangjie)
```

## 其他命令

```sh
// 显示当前登录用户
npm whoami

// 登出
npm logout

// 设置scope
npm config set scope @somescope

// 删除scope
npm config delete scope

// 查看名称是否被使用
npm pack [<package>]
```



