# Openshift s2i behind Corporate Proxy (.NET Core from Developer Catalog)

## Prerequisites
- Openshift Cluster behind proxy
- Internal Image registry (to access the images you built in cluster)

## Overview
When behind proxy, .NET Corebuild from **Developer Catalog(e.g .NET Core)** will give error that nuget packages can not download because there are **no secure connection to repo**. 
- To solve this problem you can make **your own builder image** putting your company's proxy certificate in it, but in the end you will not get to use **Red Hat's official images** and updates to those images. You will have to update your own images always . 

### Initial Script to trust company's CA certificates
In this explanation you will have chance to use **Red Hat's own images**. These images have **s2i scripts** in it. **Red Hat gives option to change the s2i scripts**. 
- The idea is to put an initial script first to **inject those company or other certificates to image** before anything actually starts. With that **.NET Core** will securely connect through proxy to nuget repos and building operation will be success.

> In this explanation .NET Core from Developer Catalog will be used.
 
 In this example build will be done in **example project**(namespace).
 - Switch to **Developer Tab** and click to **From Catalog**
 
![1](https://user-images.githubusercontent.com/59168275/91820707-fa519700-ec3e-11ea-95e1-e1836736c3bd.png)
 - Select **.NET Core** and then **Instantiate Template**
 
![2](https://user-images.githubusercontent.com/59168275/91820737-fcb3f100-ec3e-11ea-81f8-cf48e02a39e2.png)
![3](https://user-images.githubusercontent.com/59168275/91820777-ff164b00-ec3e-11ea-83a0-4dcfd0656556.png)
 - As an example [https://github.com/OktaySavdi/chat_example](https://github.com/OktaySavdi/chat_example) project will be built.
 
![4](https://user-images.githubusercontent.com/59168275/91820795-00477800-ec3f-11ea-8bea-37c14ce93711.png)
- Scroll down and uncheck **Launch the first build when the build configuration is created** because the first one would fail and put environment variable as in the picture.

```bash
SSL_CERT_DIR=/opt/app-root/ssl_dir
```
![5](https://user-images.githubusercontent.com/59168275/91820806-00e00e80-ec3f-11ea-923f-f54317291db4.png)
- In the building stage image will call for the script in this path **/usr/libexec/s2i/assemble**. 
  - What will we do is creating our own script and name it as **assemble**
  - In this script **/opt/app-root/ssl_dir** directory will be created and extra CA's to be trusted will be added here.
  - **Custom CA certs will be placed in /opt/app-root/ssl_dir directory** and with **SSL_CERT_DIR=/opt/app-root/ssl_dir** environment variable will make .NET Core trust our own CA's.
- The **assemble** script will look like this

```bash
#!/bin/bash

mkdir -p $DOTNET_SSL_CERT_DIR

cat <<EOF > /tmp/cert1.crt
-----BEGIN CERTIFICATE-----
................................................................
................................................................
-----END CERTIFICATE-----
EOF

cat <<EOF > /tmp/cert2.crt
-----BEGIN CERTIFICATE-----
................................................................
................................................................
-----END CERTIFICATE-----
EOF


/usr/libexec/s2i/assemble
```
- The newly created **/opt/app-root/ssl_dir** directory will have our **CA certs** in it.
- **The assemble script** should be put somewhere reachable from your Cluster. In this example **the assemble script** can be accessed via URL. Here is the official link for [s2i scripts and accessing them](https://docs.openshift.com/container-platform/4.5/builds/build-strategies.html#images-create-s2i-scripts_build-strategies)
- To put the link select **Build tab** and click the **build** that created.

![6](https://user-images.githubusercontent.com/59168275/91820838-02a9d200-ec3f-11ea-9673-71a87abf04a0.png)
- Select **YAML tab** and put the link of the directory contains the script to **spec.strategy.sourceStrategy.scripts** part.
 
![7](https://user-images.githubusercontent.com/59168275/91820867-04739580-ec3f-11ea-9d9e-703cb51718c6.png)
- After all those steps click **Start Build** and in the opened page select **Logs** and watch. You will see that .NET Core won't give error and build process will be finished with success after that.
 
![8](https://user-images.githubusercontent.com/59168275/91820901-063d5900-ec3f-11ea-811b-0b36272b1c5a.png)
![9](https://user-images.githubusercontent.com/59168275/91820919-08071c80-ec3f-11ea-8d8f-982d7d772980.png)


Using this method you get to use Red Hat's updated and supported image always. There is no need create a custom image for certificate and updating it always.

---
##### Resources
1. [Build Strategies](https://docs.openshift.com/container-platform/4.5/builds/build-strategies.html)

