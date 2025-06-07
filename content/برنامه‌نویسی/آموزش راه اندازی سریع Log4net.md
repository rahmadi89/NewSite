این آموزش تلاشش اینه که خیلی مینیمال باشه چون log4net تنظیمات زیادی داره و من نمیخوام درگیرشون بشم.
مراحل کار اینه:
اول کتابخونه log4net رو از طریق این منو برای هر پروژه‌ای که میخوای توش لاگ کنی نصب کن:
![[Pasted image 20240818092738.png]]
بعدش از سربرگ Browse سرچ می‌کنیم Log4net و بعد انتخاب کتابخونه مربوط به Apache، تیک پروژه یا پروژه‌هایی که میخوایم از Log4.net داخلشون استفاده کنیم رو میزنیم و در آخر روی Install کلیک می‌کنیم تا نصب بشه (طبیعتا اینترنت باید وصل باشه برای این فرآیند). 
![[Pasted image 20240818095735.png]]

خب اول فایل کانفیگش رو ایجاد می کنیم که من یدونه مثال میارم اینجا:
```xml
<configuration>
  <log4net>
    <root>
      <level value="ALL" />
      <appender-ref ref="txtfile" />
    </root>

    <appender name="txtfile" type="log4net.Appender.FileAppender">
      <file value="D:\testLog4net.txt" />
      <appendToFile value="true" />
    
      <Layout type="log4net.Layout.PatternLayout">
        <conversionPattern value="%date %level %logger.%method [%line] - %message%newline" />
    
      </Layout>

    </appender>
  </log4net>
</configuration>
```

بعدش هم موارد موردنیاز رو از توی این کد استخراج می‌کنیم و توی کد خودمون وارد می‌کنیم:
```CS
using log4net;
using log4net.Config;
using System;
using System.IO;
using System.Reflection;

namespace TestLog4NetMessageEncryptor
{
    class Program
    {
        private static readonly ILog log = LogManager.GetLogger(MethodBase.GetCurrentMethod().DeclaringType);
        static void Main(string[] args)
        {
            var logRepository = LogManager.GetRepository(Assembly.GetEntryAssembly());
            XmlConfigurator.Configure(logRepository, new FileInfo("log4net.config"));
            log.Info("Hello World");
        }
    }
}

```
البته برای اینکه این کد کار کنه باید حتما فایل log4net.config با محتوایی که بالاتر گفتم کنار فایل اجرایی برنامه مون وجود داشته باشه. بنابراین اگه از طریق Visual Studio این فایل رو ایجاد کردیم باید یادمون باشه تو تنظیماتش Copy always رو بزنیم:
![[Pasted image 20240818105425.png]]
همین، این حالت ساده ش بود اینم نتیجه کار:
![[Pasted image 20240818105535.png]]
دیگه اینکه فرمت لاگ چطوری باشه، یا مثلا هم توی Console بزنه هم توی فایل یا مثلا سطح یا همون Level لاگ چی باشه و باقی تنظیمات دیگه بستگی به نیاز داره که اینجا نمیخوام بازگوش کنم.