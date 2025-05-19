Here is a **well-organized technical document** that explains how to install, configure, and manage **Apache Tomcat** step-by-step.

---

# 📘 Apache Tomcat Setup and Configuration Guide

## 🧾 What is Tomcat?

**Apache Tomcat** is an open-source web server and servlet container developed by the Apache Software Foundation. It:

* Hosts Java-based web applications.
* Implements **Java Servlet** and **JavaServer Pages (JSP)** technologies.
* Is commonly used in development and production for Java apps.

---

## 🛠️ 1. Tomcat Installation

### 🔹 Download Tomcat

```bash
wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.86/bin/apache-tomcat-9.0.86.tar.gz
```

### 🔹 Extract the Archive

```bash
tar -xvzf apache-tomcat-9.0.86.tar.gz
```

### 🔹 (Optional) Rename the Directory

```bash
mv apache-tomcat-9.0.86 tomcat
```

---

## 🚀 2. Start and Stop Tomcat

Navigate to the `bin` directory inside the Tomcat folder:

```bash
cd tomcat/bin
```

* Start Tomcat:

  ```bash
  sh startup.sh
  ```

* Stop Tomcat:

  ```bash
  sh shutdown.sh
  ```

📝 **Default port**: `8080`
To change the port, edit the following file:

```bash
vi tomcat/conf/server.xml
```

Search for `<Connector port="8080"` and modify the port number.

---

## 📦 3. Deploy a WAR File

Copy your `.war` file to:

```bash
tomcat/webapps/
```

Tomcat will auto-deploy it if the service is running.

---

## 🔐 4. Enable Remote Access to Manager GUI

By default, the **Manager App** is only accessible from `localhost`.

### 🔹 Locate the Context XML Files

Find the files:

```bash
find / -name context.xml
```

Typical paths:

* `/tomcat/webapps/host-manager/META-INF/context.xml`
* `/tomcat/webapps/manager/META-INF/context.xml`

### 🔹 Modify Access Restrictions

Edit both files and **comment out** the following `<Valve>` entry:

```xml
<!--
<Valve className="org.apache.catalina.valves.RemoteAddrValve"
       allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
-->
```

This allows access from any IP. You can customize it to allow specific IPs instead.

---

## 👤 5. Configure Manager App Credentials

Tomcat Manager requires login credentials.

### 🔹 Edit `tomcat-users.xml`

```bash
vi tomcat/conf/tomcat-users.xml
```

Add the following content to define roles and users:

```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>
<role rolename="manager-status"/>

<user username="admin" password="admin" roles="manager-gui,manager-script,manager-jmx,manager-status"/>
<user username="deployer" password="deployer" roles="manager-script"/>
<user username="tomcat" password="s3cret" roles="manager-gui"/>
```

Save the file and restart Tomcat to apply changes:

```bash
sh tomcat/bin/shutdown.sh
sh tomcat/bin/startup.sh
```

Now, you can access the Tomcat manager from your browser at:

```
http://<your-server-ip>:8080/manager/html
```

Login with the credentials added above.

---

## ✅ Summary

| Task                         | Command/Path                      |
| ---------------------------- | --------------------------------- |
| Download Tomcat              | `wget ...`                        |
| Extract Tomcat               | `tar -xvzf`                       |
| Start/Stop Tomcat            | `sh startup.sh`, `sh shutdown.sh` |
| Change Port                  | `conf/server.xml`                 |
| Deploy WAR File              | `webapps/`                        |
| Enable Remote Manager Access | `META-INF/context.xml`            |
| Add Manager Login            | `conf/tomcat-users.xml`           |

---

Would you like this exported as a **PDF**, **Markdown file**, or included in a server automation script?
