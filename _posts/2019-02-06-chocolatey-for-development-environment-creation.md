---
layout: post
title: Chocolatey for Development Environments
subtitle: Automating developer tool installation
---

I'm currently experimenting with using Chocolatey as tool to automate installation of developer tools that I commonly use. 

Below is a sample packages.config file I use to install the most common tools that I rely on for software development.

```
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="visualstudio2017enterprise" />
  <package id="sql-server-management-studio" />
  <package id="vscode" />
  <package id="azure-data-studio" />
  <package id="beyondcompare" version="3.3.11.18371" />
  <package id="filezilla" />
  <package id="postman" />
  <package id="nugetpackageexplorer" />
  <package id="queueexplorer-professional" />
</packages>
```