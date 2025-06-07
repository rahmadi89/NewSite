گاهی اوقات پیش میاد بخوایم از یه برنامه دیگه داخل برنامه اصلی استفاده کنیم ولی برامون مهمه که فایل اجرایی (exe) برنامه فرعی به صورت جداگانه، در دسترس کاربر نباشه. برای این منظور میتونیم این مراحل رو پیش بریم:
1. پروژه جدید ایجاد کنید
![[Pasted image 20240824104619.png]]
2. از طریق راست کلیک و گزینه Add و سپس Existing Item فایل فرعی رو به برنامه استفاده می کنیم (برای سادگی اینجا از فایل regedt32.exe) استفاده شد. بعد از اضافه کردن حتما از طریق راست کلیک روی فایل مربوط و انتخاب Properties گزینه Build Action روی Embedded Resource قرار بگیره. از این طریق فایل exe داخل برنامه embedd میشه.
![[Pasted image 20240824104749.png]]
3. مرحله آخر اینه که حالا که embedd کردیم چطوری ازش استفاده کنیم. فرمت استفاده از فایل به این صورت میشه:
ApplicationNameSpace.ResourceName
در مثال ما میشه : EmbedExe.Regedt32.exe

4. برای اجرا میشه از این تابع استفاده کرد
```cs
		int ExecuteFile(){
			
			resourceName = "EmbedExe.Regedt32.exe";
            outputPath = Path.Combine(Path.GetTempPath(), "Regedt32.exe");
            // Extract the embedded resource
            using (Stream stream = Assembly.GetExecutingAssembly().GetManifestResourceStream(resourceName))
            using (FileStream fileStream = new FileStream(outputPath, FileMode.Create, FileAccess.Write))
            {
                stream.CopyTo(fileStream);
            }


            // Execute the extracted file and capture the exit code
            Process process = new Process();
            process.StartInfo.FileName = outputPath;
            process.StartInfo.WindowStyle = ProcessWindowStyle.Hidden;
            process.Start();
            process.WaitForExit(); // Wait for the process to exit

            File.Delete(outputPath);

            return process.ExitCode;
        }
```
این تابع میاد فایلمون رو توی مسیر temp ویندوز کپی، اجرا و بعدش پاک میکنه و در نهایت ExitCode رو برمیگردونه.