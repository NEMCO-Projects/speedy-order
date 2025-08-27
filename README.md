# speedy-order


Got it 👍
I’ll turn your steps into a **well-structured Markdown (MD) guide**, written clearly and simply so even slow learners can follow along step by step.

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

Would you like me to also create a **visual flow diagram (steps + commands)** in the same MD so slow learners can see both text + diagram?
