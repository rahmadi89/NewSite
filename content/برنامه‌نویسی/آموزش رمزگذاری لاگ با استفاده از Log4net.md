بعد از [[آموزش راه اندازی سریع Log4net]] حالا کار دیگه‌ای که میتونیم بکنیم اینه که قسمت‌های حساس لاگ رو رمزگذاری کنیم. البته یه راهی که این روزهای زیاد هم استفاده میشه اینه که کل لاگ رو رمزگذاری می‌کنن که خب این راه می‌تونه خوندن لاگ رو سخت‌تر کنه و من ترجیح میدم فقط قسمت‌های حساس رمزگذاری بشن. برای این موضوع یک راه اینه که قبل اینکه متنی رو لاگ کنیم اول رمزگذاریش کنیم و بعد متن رمزنگاری شده رو لاگ کنیم. راه بدی هم نیست. ولی من ترجیح میدم راه دوم رو برم. راه دوم استفاده از کتابخونه [Log4NetMessageEncryptor](https://github.com/ArtisanCode/Log4NetMessageEncryptor) هست که خیلی جذاب نوشته شده. برای استفاده از این کتابخونه لازم نیست تغییری توی قسمت‌های مختلف نرم‌افزار که لاگ کردیم بدیم. فقط با چند خط کانفیگ و همون لاگرهای قبلی که نوشته شده میتونیم اطلاعات حساس رو رمزنگاری کنیم. واسه همین توی پروژه‌‌هایی که از قبل نوشته شدن و می‌خوایم تغییری توشون بدیم عالیه.
بریم سراغش
اول باید کتابخونه شو نصب کنیم:
![[Pasted image 20240818123947.png]]
از اونجا که با نصب این کتابخونه، کتابخونه Log4Net مرتبط با همین نسخه نصب میشه، شاید بهتر باشه نسخه قبلی (log4net) که نصب هست رو پاک کنیم که به مشکل نخوریم.
بعدش باید بخش مربوط به configSections رو به فایل app.config اضافه کنیم:
```xml
<configsections>
  <section name="Log4NetMessageEncryption" type="ArtisanCode.Log4NetMessageEncryptor.Log4NetMessageEncryptorConfiguration, ArtisanCode.Log4NetMessageEncryptor" />
  <section name="log4net" type="log4net.Config.Log4NetConfigurationSectionHandler, log4net" />
  <!-- Additional sections may be defined but not duplicated -->
</configsections>
```
بعد از این هم لازمه بخش مربوط به کلید رو به فایل app.config اضافه کنیم که اینه:
```xml
<Log4NetMessageEncryption>
  <EncryptionKey KeySize="256" Key="3q2+796tvu/erb7v3q2+796rvu/erb7v3q2+796tvu8="/>
</Log4NetMessageEncryption>
```
==نکته مهم: به هیچ عنوان از کلید نمونه این آموزش استفاده نشه==
نکته: اینطوری کلید ما در دسترس همه قرار میگیره و هر کسی با داشتن این کلید میتونه اقدام به رمزگشایی کنه. برای اینکه این مشکل رفع بشه میتونیم از روش‌هایی مثل
[[آموزش رمزگذاری فایل‌های کانفیگ در Dot Net Framework]] استفاده کنیم.

در نهایت باید بخش مربوط به log4net رو اضافه کنیم. اینجا برای اینکه فایل Log4net.config رو جدا کنم اول این خط و میزنم توی app.config:
```xml
  <log4net configSource="log4net.config" />
```

یک نمونه فایل کامل app.config:
```xml
<?xml version="1.0"?>
<configuration>
  <configSections>
    <section name="Log4NetMessageEncryption" type="ArtisanCode.Log4NetMessageEncryptor.Log4NetMessageEncryptorConfiguration, ArtisanCode.Log4NetMessageEncryptor"/>
    <section name="log4net" type="log4net.Config.Log4NetConfigurationSectionHandler, log4net"/>
  </configSections>

  <startup>
    <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.0"/>
  </startup>

  <Log4NetMessageEncryption CipherMode="CBC" Padding="ISO10126">
    <EncryptionKey KeySize="256" Key="3q2+796tvu/erb7v3q2+796tvu/erb7v3q2+796tvu8="/>
  </Log4NetMessageEncryption>

  <log4net configSource="log4net.config" />
  
</configuration>

```


اینم فایل log4net.config:
```xml
<log4net>

  <appender name="RollingFileAppender" type="log4net.Appender.RollingFileAppender">
    <file value="log.txt"/>
    <appendToFile value="true"/>
    <rollingStyle value="Size"/>
    <maxSizeRollBackups value="10"/>
    <maximumFileSize value="100KB"/>
    <staticLogFileName value="true"/>
    <layout type="log4net.Layout.PatternLayout">
      <conversionPattern value="%date [%thread] %-5level %logger - %message%newline"/>
    </layout>
  </appender>

  <appender name="RollingXmlFileAppender" type="log4net.Appender.RollingFileAppender">
    <file value="log.xml"/>
    <appendToFile value="true"/>
    <rollingStyle value="Size"/>
    <maxSizeRollBackups value="10"/>
    <maximumFileSize value="100KB"/>
    <staticLogFileName value="true"/>
    <layout type="log4net.Layout.XMLLayout"/>
  </appender>

  <appender name="ColoredConsoleAppender" type="log4net.Appender.ColoredConsoleAppender">
    <mapping>
      <level value="FATAL"/>
      <foreColor value="White, HighIntensity"/>
      <backColor value="Red, HighIntensity"/>
    </mapping>
    <mapping>
      <level value="ERROR"/>
      <foreColor value="Red, HighIntensity"/>
    </mapping>
    <mapping>
      <level value="WARN"/>
      <foreColor value="Yellow, HighIntensity"/>
    </mapping>
    <mapping>
      <level value="INFO"/>
      <foreColor value="Cyan"/>
    </mapping>
    <mapping>
      <level value="DEBUG"/>
      <foreColor value="Green"/>
    </mapping>
    <layout type="log4net.Layout.PatternLayout">
      <conversionPattern value="%date [%thread] %-5level %logger - %message%newline"/>
    </layout>
  </appender>

  <appender name="MessageEncryptingAppender" type="ArtisanCode.Log4NetMessageEncryptor.MessageEncryptingForwardingAppender, ArtisanCode.Log4NetMessageEncryptor">
    <!-- Add other appenders here and the log messages will be sent to every listed appender with the encrypted messages -->
    <appender-ref ref="RollingFileAppender"/>
    <!-- Link to another Log4Net appender -->
    <appender-ref ref="ColoredConsoleAppender"/>
    <!-- Link to another Log4Net appender -->
    <appender-ref ref="RollingXmlFileAppender"/>
    <!-- Link to another Log4Net appender -->
  </appender>

  <root>
    <!-- WARNING: Any other appenders added here will NOT receive encrypted messages. All the messages will appear in plain text. -->
    <appender-ref ref="MessageEncryptingAppender"/>
  </root>

</log4net>
```

این مثال که مال خود گیت‌هاب پروژه ست، اومده 3 تا Appender رو رمزگذاری کرده. ولی اگه بخوای یه Appender به صورت عادی و رمزگذاری نشده باشه کافیه توی قسمت root مستقیم بهش اشاره کنی
اینم یه نمونه کد:
```cs
using ArtisanCode.Log4NetMessageEncryptor;
using log4net;
using log4net.Config;
using System;
using System.Configuration;

namespace ArtisanCode.MessageEncryptorExample
{
    class Program
    {
        private static ILog log = LogManager.GetLogger("MessageEncryptorExample");

        static void Main(string[] args)
        {
            // Ensure log4net is configured
            XmlConfigurator.Configure();

            Log4NetMessageEncryptorConfiguration config = (Log4NetMessageEncryptorConfiguration)ConfigurationManager.GetSection("Log4NetMessageEncryption");

            Console.WriteLine("Configuration found:");
            Console.WriteLine("Configuration.Padding - {0}", config.Padding);
            Console.WriteLine("Configuration.CipherMode - {0}", config.CipherMode);
            Console.WriteLine("Configuration.EncryptionKey.KeySize - {0}", config.EncryptionKey.KeySize);
            Console.WriteLine("Configuration.EncryptionKey.Key - {0}", config.EncryptionKey.Key);

            log.Debug("Debug message 1");
            log.DebugFormat("Debug message {0}a", 1);
            log.Debug("Debug Exception message", new ApplicationException("Debug message log"));

            log.Info("Info message 1");
            log.InfoFormat("Info message {0}a", 1);
            log.Info("Info Exception message", new ApplicationException("Info message log"));

            log.Warn("Warning message 1");
            log.WarnFormat("Warning message {0}a", 1);
            log.Warn("Warning Exception message", new ApplicationException("Debug message log"));

            log.Error("Error message 1");
            log.ErrorFormat("Error message {0}a", 1);
            log.Error("Error Exception message", new MemberAccessException("Error message log"));

            log.Fatal("Fatal message 1");
            log.FatalFormat("Fatal message {0}a", 1);
            log.Fatal("Fatal Exception message", new StackOverflowException("Fatal message log", new OutOfMemoryException("Out of memory inner exception")));

            Console.WriteLine("Please press any key to close...");
            Console.ReadKey();
        }
    }
}
```
و در نهایت نتیجه:
 
همین اتفاق توی لاگ‌های Text و XML هم میفته.


برای رمزگشایی کردن هم میشه از پروژه موجود در این مسیر استفاده کرد: [LogDecrypt](https://github.com/ArtisanCode/Log4NetMessageEncryptor/tree/master/src/Tools/LogDecrypt)
همچنین برای تولید کلید هم پروژه ساخته شده: [KeyGen](https://github.com/ArtisanCode/Log4NetMessageEncryptor/tree/master/src/Tools/KeyGen)
