# TonyDeng's Blog

[![Build Status](https://travis-ci.org/tonydeng/tonydeng.github.io.svg?branch=develop)](https://travis-ci.org/tonydeng/tonydeng.github.io)

## 安装NodeJS及NPM

### 安装node.js

```
brew install node
```

### 安装npm

```
brew install npm
```

### 安装cnpm

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

## Hexo安装

```
cnpm install hexo-cli -g
```

## Blog及Theme

```
git clone git@github.com:tonydeng/tonydeng.github.io.git

git checkout develop

cd tonydeng.github.io/themes

git clone git@github.com:tonydeng/hexo-theme-next.git next

cd next

git checkout master
```

## 使用git subtree来管理Blog和Theme

```
git clone git@github.com:tonydeng/tonydeng.github.io.git

git checkout develop

git remote add -f theme-next git@github.com:tonydeng/hexo-theme-next.git

git subtree add -P theme/next theme-next master
```

## Blog构建

```
cnpm install --save

hexo g && hexo s --debug
```
