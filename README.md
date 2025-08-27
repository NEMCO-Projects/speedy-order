# speedy-order


Here’s the cleaned-up version:

---

# 🚀 Deploying **Speedy Order** Application on IIS (Windows Server)

This guide explains step by step how to set up **IIS**, clone the project from GitHub, configure permissions, compile, and run the **Speedy Order** website on Windows Server.
Designed for **beginners / slow learners** ✅

---

## 1️⃣ Open Firewall for HTTP (Port 80)

Open **PowerShell** as Administrator and run:

```powershell
netsh advfirewall firewall add rule name="IIS HTTP" dir=in action=allow protocol=TCP localport=80
```

---

## 2️⃣ Install IIS and Required Features

Run in **PowerShell**:

```powershell
Install-WindowsFeature Web-Server,Web-Common-Http,Web-Default-Doc,Web-Static-Content,Web-Http-Errors,Web-Http-Logging,Web-Request-Monitor,Web-Filtering,Web-Mgmt-Console,Web-App-Dev,Web-Net-Ext45,Web-Asp-Net45,Web-ISAPI-Ext,Web-ISAPI-Filter -IncludeManagementTools
```

✅ This installs IIS with all features required to run ASP.NET applications.

---

## 3️⃣ Install Git

1. Download Git from [https://git-scm.com/downloads](https://git-scm.com/downloads)
2. Install it with default options.

---

## 4️⃣ Clone the Project

Open **Command Prompt (CMD)** and run:

**Create sites folder in C:\***
```cmd
mkdir C:\sites
```

```cmd
cd C:\sites
git clone https://github.com/NEMCO-Projects/speedy-order.git
```

---

## 5️⃣ Set IIS Permissions

Grant IIS access to project folders:

```cmd
icacls "C:\sites\speedy-order" /grant "IIS_IUSRS:(OI)(CI)(RX)" /T
icacls "C:\sites\speedy-order\imgs" /grant "IIS_IUSRS:(OI)(CI)(M)" /T
```

---

## 6️⃣ Create `bin` Folder & Copy Required DLL

Run in **PowerShell**:

```powershell
New-Item "C:\sites\speedy-order\bin" -ItemType Directory -Force | Out-Null

Copy-Item "C:\sites\speedy-order\packages\Microsoft.CodeDom.Providers.DotNetCompilerPlatform.2.0.1\lib\net45\Microsoft.CodeDom.Providers.DotNetCompilerPlatform.dll" "C:\sites\speedy-order\bin" -Force
```

---

## 7️⃣ Create IIS App Pool & Site

Run in **CMD**:

```cmd
%windir%\system32\inetsrv\appcmd add apppool /name:"speedy-order-app" /managedRuntimeVersion:"v4.0" /managedPipelineMode:"Integrated"

%windir%\system32\inetsrv\appcmd add site /name:"speedy-order" /bindings:"http/*:80:" /physicalPath:"C:\sites\speedy-order"

%windir%\system32\inetsrv\appcmd set app "speedy-order/" /applicationPool:"speedy-order-app"
```

---

## 8️⃣ Configure Default Document

Run in **CMD**:

```cmd
%windir%\system32\inetsrv\appcmd set config "speedy-order" /section:defaultDocument /+files.[value='userlogin.aspx']
%windir%\system32\inetsrv\appcmd set config "speedy-order" /section:defaultDocument /+files.[value='WebForm1.aspx']
```

---

## 9️⃣ Install Visual Studio Build Tools

1. Go to [Visual Studio Downloads](https://visualstudio.microsoft.com/downloads/)
2. Download **Build Tools for Visual Studio 2022**
3. During installation, select:

   * ✅ **.NET Framework build tools**
   * ✅ **Web development build tools**

---
### Note : Download link
https://aka.ms/vs/17/release/vs_BuildTools.exe

## 🔟 Build the Database Project

Run in **CMD**:

```cmd
"C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\MSBuild\Current\Bin\MSBuild.exe" "C:\sites\speedy-order\DatabaseProject.csproj" /p:Configuration=Release
```

---

## 1️⃣1️⃣ Precompile ASP.NET Application

Run in **PowerShell**:

```powershell
& $env:WINDIR\Microsoft.NET\Framework64\v4.0.30319\aspnet_compiler.exe `
  -p "C:\sites\speedy-order" `
  -v / `
  -fixednames -u -errorstack -f `
  "C:\sites\speedy-order_precompiled"
```

---

## 1️⃣2️⃣ Configure IIS Website

1. Press **Win + R**, type `inetmgr`, and hit Enter → IIS Manager opens.
2. Stop or remove the **Default Web Site** (otherwise IIS will show it on port 80).

   * Right-click → **Stop** (or **Delete**).
3. Start the **Speedy Order** website:

   * Right-click **speedy-order** → **Start**.

---

## ✅ Final Step: Test Website

Open your browser and go to:

👉 [http://172.185.169.14](http://172.185.169.14)

You should now see the **Speedy Order Application** 🎉

---

⚡ Done! Your app is now running on IIS.

---

---

**SpeedyOrder Setup Guide (Step-by-Step)**

---

**✅ Prerequisites (Install These First)**

Before starting, make sure you have the following installed on your computer:

1. **SQL Server Express** and **SQL Server Management Studio (SSMS)**
2. **Visual Studio 2022 Build Tools**
3. **IIS (Internet Information Services)**
4. **PowerShell** (already included in Windows)
5. **.NET Framework version 4.0.30319**

---

**Step 1: Create the Database**

1. Open **SQL Server Management Studio (SSMS)**.
2. Connect to your local server (usually named something like `.\SQLEXPRESS`).
3. In the query window, paste the following and execute:

```
CREATE DATABASE SpeedyOrder;
GO
```

This will create the database needed for the project.

---

**Step 2: Load the Database Schema**

1. Find the file called **Database.sql** inside the project repo.
2. Open that file in SSMS.
3. Make sure you're connected to the **SpeedyOrder** database.
4. Run the script to create tables and insert necessary data.

---

**Step 3: Create a SQL Login and User**

Run the following SQL code in SSMS to create a user that the application will use:

```
CREATE LOGIN speedyuser WITH PASSWORD = 'ChangeThisStrong#P@ss1';
USE SpeedyOrder;
CREATE USER speedyuser FOR LOGIN speedyuser;
EXEC sp_addrolemember 'db_owner', 'speedyuser';
```

This allows the app to connect securely to the database.

---

**Step 4: Update the Web.config File**

1. Open the file located at `C:\sites\speedy-order\Web.config`
2. Look for the `<connectionStrings>` section.
3. Replace it with the following:

```
<connectionStrings>
  <add name="MyConnectionString"
       connectionString="Server=.\SQLSEVER;Database=SpeedyOrder;User ID=speedyuser;Password=ChangeThisStrong#P@ss1;MultipleActiveResultSets=true"
       providerName="System.Data.SqlClient" />
</connectionStrings>
```

<!-- Web.Config Configuration File -->

### Check this also in web.config
```
<configuration>
    <system.web>
        <customErrors mode="Off"/>
    </system.web>
</configuration>

```       
Make sure the server name (`.\SQLSEVER`) is correct. You might need to change it to `.\SQLEXPRESS` or your actual SQL instance name.

---

**Step 5: Enable SQL Server Network Access**

This step ensures IIS can connect to SQL Server.

1. Open **SQL Server Configuration Manager**.
2. Go to:

   * SQL Server Network Configuration → Protocols for `SQLEXPRESS`
3. Enable:

   * **TCP/IP**
   * **Named Pipes**
4. Restart the **SQL Server (SQLEXPRESS)** service.

---

**Step 6: Allow SQL Server Through Windows Firewall**

Open **PowerShell as Administrator** and run:

```
netsh advfirewall firewall add rule name="SQL Server" dir=in action=allow protocol=TCP localport=1433
```

This allows SQL traffic on port 1433 through the firewall.

---

**Step 7: Restart IIS**

Still in PowerShell, run this command:

```
iisreset
```

This restarts IIS to apply changes.

---

**Step 8: Build the Database Project**

1. Open **Command Prompt** (not PowerShell).
2. Run the following command to build the project:

```
"C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\MSBuild\Current\Bin\MSBuild.exe" "C:\sites\speedy-order\DatabaseProject.csproj" /p:Configuration=Release
```

Make sure the file path matches where your project is located.

---

**Step 9: Precompile the ASP.NET Website**

In **PowerShell**, run the following command:

```
& $env:WINDIR\Microsoft.NET\Framework64\v4.0.30319\aspnet_compiler.exe `
  -p "C:\sites\speedy-order" `
  -v / `
  -fixednames -u -errorstack -f `
  "C:\sites\speedy-order_precompiled"
```

This precompiles the website and prepares it for deployment.

---

**🎉 You're Done!**

Your SpeedyOrder project should now be set up and ready to use. If you run into issues:

* Double-check that SQL Server is running and accepting connections
* Make sure IIS is started
* Verify your Web.config connection string
* Ensure the firewall rule was added correctly

Let me know if you need help with any specific step.
