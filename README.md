Absolutely — here's your entire **Speedy Order IIS deployment guide converted to PowerShell**, step-by-step. This version eliminates CMD-specific syntax and ensures everything is **PowerShell-compatible**, including escaping and formatting. ✅

---

# 🚀 Full PowerShell Deployment Script for Speedy Order on IIS

This PowerShell version is ready to paste into a terminal or save as a `.ps1` script.

---

## 1️⃣ Open Firewall for HTTP (Port 80)

```powershell
netsh advfirewall firewall add rule name="IIS HTTP" dir=in action=allow protocol=TCP localport=80
```

---

## 2️⃣ Install IIS and Required Features

```powershell
Install-WindowsFeature Web-Server,Web-Common-Http,Web-Default-Doc,Web-Static-Content,Web-Http-Errors,Web-Http-Logging,Web-Request-Monitor,Web-Filtering,Web-Mgmt-Console,Web-App-Dev,Web-Net-Ext45,Web-Asp-Net45,Web-ISAPI-Ext,Web-ISAPI-Filter -IncludeManagementTools
```

---

## 3️⃣ Install Git (Manual Step)

Download and install from: [https://git-scm.com/downloads](https://git-scm.com/downloads)

after git installation restart power shell

---

## 4️⃣ Clone the Project

```powershell
New-Item -ItemType Directory -Force -Path "C:\sites"
Set-Location "C:\sites"
git clone https://github.com/NEMCO-Projects/speedy-order.git
```

---

## 5️⃣ Set IIS Permissions

```powershell
icacls "C:\sites\speedy-order" /grant "IIS_IUSRS:(OI)(CI)(RX)" /T
icacls "C:\sites\speedy-order\imgs" /grant "IIS_IUSRS:(OI)(CI)(M)" /T
```

---

## 6️⃣ Create `bin` Folder & Copy Required DLL

```powershell
New-Item "C:\sites\speedy-order\bin" -ItemType Directory -Force | Out-Null

Copy-Item "C:\sites\speedy-order\packages\Microsoft.CodeDom.Providers.DotNetCompilerPlatform.2.0.1\lib\net45\Microsoft.CodeDom.Providers.DotNetCompilerPlatform.dll" `
  -Destination "C:\sites\speedy-order\bin" -Force
```

---

## 7️⃣ Create IIS App Pool & Site

```powershell
$appcmd = "$env:windir\system32\inetsrv\appcmd.exe"

& $appcmd add apppool /name:"speedy-order-app" /managedRuntimeVersion:"v4.0" /managedPipelineMode:"Integrated"
& $appcmd add site /name:"speedy-order" /bindings:"http/*:80:" /physicalPath:"C:\sites\speedy-order"
& $appcmd set app "speedy-order/" /applicationPool:"speedy-order-app"
```

---

## 8️⃣ Configure Default Documents

```powershell
& $appcmd set config "speedy-order" /section:defaultDocument /+files.[value='userlogin.aspx']
& $appcmd set config "speedy-order" /section:defaultDocument /+files.[value='WebForm1.aspx']
```

---

## 9️⃣ Install Visual Studio Build Tools (Manual Step)

Download from:
[https://aka.ms/vs/17/release/vs\_BuildTools.exe](https://aka.ms/vs/17/release/vs_BuildTools.exe)

During install, select:

* ✅ .NET Framework build tools
* ✅ Web development build tools

---

## 🔟 Build the Database Project

```powershell
& "C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\MSBuild\Current\Bin\MSBuild.exe" `
  "C:\sites\speedy-order\DatabaseProject.csproj" /p:Configuration=Release
```

---

## 1️⃣1️⃣ Precompile ASP.NET Application

```powershell
& "$env:WINDIR\Microsoft.NET\Framework64\v4.0.30319\aspnet_compiler.exe" `
  -p "C:\sites\speedy-order" `
  -v / `
  -fixednames -u -errorstack -f `
  "C:\sites\speedy-order_precompiled"
```

---

## 1️⃣2️⃣ Configure IIS in GUI (Manual Step)

1. Press **Win + R**, type `inetmgr`, hit **Enter**
2. Stop or delete **Default Web Site**
3. Start your site: Right-click on `speedy-order` → **Start**

---

## ✅ Test the Website

Open your browser and go to:

```
http://<your-server-ip>
```

Example:

```
http://172.185.169.14
```

---

⚡ Done! Your app is now running on IIS.

---




Got it! Here's the **step-by-step guide** for setting up the SpeedyOrder project — **without Markdown formatting** — and explained clearly from start to finish, assuming you're new to this or moving slowly. Each section builds on the previous one.

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
