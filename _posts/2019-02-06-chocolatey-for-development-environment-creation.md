---
layout: post
title: Chocolatey for Development Environments
subtitle: Automating developer tool installation
---

I'm currently experimenting with using [Chocolatey](https://chocolatey.org/) as a way to automate installation of developer tools that I commonly use. The main benefit I envision for this approach would be speeding up creation of development environments and therefore improving the on-boarding process for new developers joining a team that need to have a common set of development tools installed.

Having worked in teams that use local virtual machines for development and where the base virtual machine image isn't kept up to date with the latest versions of software installed, I feel this offers an improvement to that scenario as well. This can be achieved by creating a base virtual machine containing just a clean OS and an install of Chocolatey with a packages.config file containing the software that needs to be installed. Then, any new developer can simply create a copy of the virtual machine and run 'choco install packages.config -y' in the same directory as packages.config, and they'll end up with the latest versions of the tools installed successfully.

Below is a sample packages.config file containing some common tools that I rely on for software development.

```
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="visualstudio2017enterprise" />
  <package id="sql-server-management-studio" />
  <package id="ssdt17" />
  <package id="vscode" />
  <package id="azure-data-studio" />
  <package id="beyondcompare" version="3.3.11.18371" />
  <package id="filezilla" />
  <package id="postman" />
  <package id="nugetpackageexplorer" />
  <package id="queueexplorer-professional" />
</packages>
```