---
layout: post
title: Checklist for solving certificate issues with web services
subtitle: How to get rid of "could not establish trust relationship for the ssl/tls secure channel" error
# image: /img/hello_world.jpeg
---
A common error message I've encountered when writing code to communicate with web services is *"could not establish trust relationship for the ssl/tls secure channel"*

Having seen this error message numerous times whilst writing code to work with web services, here are the steps I follow to investigate and get to a solution.

First, I ask myself if the web service I'm connecting to require a client certificate to be sent with each request?

If it does, check that under Certificates -> Local Computer in Microsoft Management Console and verify that the client certificate to be sent is installed to the Personal store. If it is and the web service call isn't working, right-click the certificate and go to All Tasks -> Manage Private Keys. Ensure that the account used by the application making the web service call has Read permissions on the certificate.

If no client certificates are being sent, then the error suggests an issue with trusting the SSL certificate used by the web service. 
To investigate, first browse to the web service url in a modern browser such as Google Chrome and click the padlock icon to view the certificate.
![Padlock icon](..\img\cert-1.png)

Next, click on the Certification Path and see if the certificate and the authority that created it are trusted. If the path is highlighted red, then the solution for the error would be to add the root certificate to the Trusted Root Certification Authorities on your machine.
![Certification Path](..\img\cert-2.png)

If the root authority for the certificate is trusted and you're still seeing the error *"could not establish trust relationship for the ssl/tls secure channel"*, then this could be an issue with the transport protocol used by the SSL certificate.
In .NET, the property *ServicePointManager.SecurityProtocol* determines which SSL/TLS protocols are supported. If your code is running on an older version of the .NET Framework, it's possible that this property only has enum values for SSL3 and TLS. Adding the following line of code will resolve the issue.
```
ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12;
```
Alternatively, I've seen examples where the web service being called is running on an older server where the only way to make the web service call succeed is to add the following line of code.
```
ServicePointManager.SecurityProtocol = SecurityProtocolType.Ssl3;
```
Ssl3 is no longer considered secure, so this isn't an ideal fix. If the server being connected to is within your control, then enabling a newer TLS version on the server is a better solution.