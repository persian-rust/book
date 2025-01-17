## نصب

اولین قدم نصب راست است. ما راست را از طریق `rustup` دانلود می‌کنیم، ابزاری خط فرمان برای مدیریت نسخه‌های راست و ابزارهای مربوطه. برای دانلود به اتصال اینترنتی نیاز دارید.

> توجه: اگر به هر دلیلی ترجیح می‌دهید از `rustup` استفاده نکنید، لطفاً صفحه [روش‌های نصب دیگر راست][otherinstall] را برای گزینه‌های بیشتر مشاهده کنید.

مراحل زیر نسخه پایدار جدیدترین کامپایلر راست را نصب می‌کنند. تضمین‌های پایداری راست اطمینان می‌دهند که تمام مثال‌های کتاب که کامپایل می‌شوند، با نسخه‌های جدیدتر راست نیز کامپایل خواهند شد. خروجی ممکن است کمی متفاوت باشد، زیرا راست به طور مرتب پیغام‌های خطا و هشدارها را بهبود می‌بخشد. به عبارت دیگر، هر نسخه پایدار جدیدی که با این مراحل نصب کنید، باید با محتوای این کتاب به درستی کار کند.

> ### یادداشت دستورات خط فرمان
>
> در این فصل و throughout the book، ما برخی از دستورات استفاده شده در ترمینال را نمایش خواهیم داد. خطوطی که باید در ترمینال وارد کنید، همگی با `$` شروع می‌شوند. شما نیازی به وارد کردن نماد `$` ندارید؛ این نماد نشان‌دهنده شروع هر دستور است. خطوطی که با `$` شروع نمی‌شوند معمولاً خروجی دستور قبلی را نشان می‌دهند. علاوه بر این، مثال‌های خاص PowerShell از `>` به جای `$` استفاده می‌کنند.

### نصب `rustup` در لینوکس یا macOS

اگر از لینوکس یا macOS استفاده می‌کنید، یک ترمینال باز کرده و دستور زیر را وارد کنید:

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

این دستور یک اسکریپت دانلود کرده و نصب ابزار `rustup` را آغاز می‌کند که نسخه پایدار جدید راست را نصب می‌کند. ممکن است از شما خواسته شود تا رمز عبور خود را وارد کنید. اگر نصب موفقیت‌آمیز بود، خط زیر ظاهر می‌شود:

```text
Rust is installed now. Great!
```

همچنین به یک _لینکر_ نیاز خواهید داشت که برنامه‌ای است که راست از آن برای ترکیب خروجی‌های کامپایل شده خود به یک فایل استفاده می‌کند. احتمالاً شما یک لینکر دارید. اگر با ارورهای لینکر روبه‌رو شدید، باید یک کامپایلر C نصب کنید که معمولاً لینکر را نیز شامل می‌شود. یک کامپایلر C همچنین مفید است زیرا برخی از پکیج‌های رایج راست به کد C وابسته‌اند و به یک کامپایلر C نیاز دارند.

برای نصب کامپایلر C در macOS، دستور زیر را اجرا کنید:

```console
$ xcode-select --install
```

کاربران لینوکس معمولاً باید GCC یا Clang را طبق مستندات توزیع خود نصب کنند. برای مثال، اگر از اوبونتو استفاده می‌کنید، می‌توانید پکیج `build-essential` را نصب کنید.

### نصب `rustup` در ویندوز

در ویندوز، به [https://www.rust-lang.org/tools/install][install] بروید و دستورالعمل‌های نصب راست را دنبال کنید. در یک مرحله از نصب، از شما خواسته می‌شود تا Visual Studio را نصب کنید. این ابزار یک لینکر و کتابخانه‌های بومی لازم برای کامپایل برنامه‌ها را فراهم می‌کند. اگر به کمک بیشتری نیاز دارید، این صفحه را مشاهده کنید [https://rust-lang.github.io/rustup/installation/windows-msvc.html][msvc]

بقیه کتاب از دستورات استفاده شده در _cmd.exe_ و PowerShell استفاده می‌کند. اگر تفاوت‌های خاصی وجود داشته باشد، توضیح خواهیم داد که کدام را باید استفاده کنید.

### عیب‌یابی

برای بررسی اینکه راست به درستی نصب شده است یا خیر، یک شل باز کرده و این دستور را وارد کنید:

```console
$ rustc --version
```

باید شماره نسخه، هش کمیّت و تاریخ کمیّت برای جدیدترین نسخه پایدار منتشر شده را به صورت زیر ببینید:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

اگر این اطلاعات را مشاهده کردید، راست به درستی نصب شده است! اگر این اطلاعات را مشاهده نکردید، بررسی کنید که راست در متغیر سیستم `%PATH%` شما قرار دارد.

در CMD ویندوز، از دستور زیر استفاده کنید:

```console
> echo %PATH%
```

در PowerShell، از دستور زیر استفاده کنید:

```powershell
> echo $env:Path
```

در لینوکس و macOS، از دستور زیر استفاده کنید:

```console
$ echo $PATH
```

اگر همه چیز درست باشد و راست همچنان کار نکند، منابع زیادی برای کمک وجود دارد. برای تماس با سایر راست‌نویسان (لقب خنده‌داری که خودمان به کار می‌بریم)، به صفحه [اجتماع][community] مراجعه کنید.

### بروزرسانی و حذف نصب

بعد از نصب راست از طریق `rustup`، بروزرسانی به نسخه جدید بسیار آسان است. از شل خود دستور زیر را اجرا کنید:

```console
$ rustup update
```

برای حذف نصب راست و `rustup`، اسکریپت حذف زیر را از شل خود اجرا کنید:

```console
$ rustup self uninstall
```

### مستندات محلی

نصب راست همچنین شامل یک نسخه محلی از مستندات است تا بتوانید آن را به صورت آفلاین مطالعه کنید. برای باز کردن مستندات محلی در مرورگر خود، دستور `rustup doc` را اجرا کنید.

هر زمان که از یک نوع یا تابع ارائه‌شده توسط کتابخانه استاندارد استفاده می‌کنید و مطمئن نیستید که چه کار می‌کند یا چگونه از آن استفاده کنید، از مستندات رابط برنامه‌نویسی (API) برای یافتن آن استفاده کنید!

### ویرایشگرهای متن و محیط‌های توسعه یکپارچه

این کتاب هیچ فرضی درباره ابزارهایی که برای نوشتن کد راست استفاده می‌کنید، ندارد. تقریباً هر ویرایشگر متنی کار را انجام می‌دهد! با این حال، بسیاری از ویرایشگرها و محیط‌های توسعه یکپارچه (IDE) پشتیبانی داخلی برای راست دارند. همیشه می‌توانید فهرست نسبتاً جدیدی از بسیاری از ویرایشگرها و IDEها را در [صفحه ابزارها][tools] در وب‌سایت راست پیدا کنید.

[otherinstall]: https://forge.rust-lang.org/infra/other-installation-methods.html  
[install]: https://www.rust-lang.org/tools/install  
[msvc]: https://rust-lang.github.io/rustup/installation/windows-msvc.html  
[community]: https://www.rust-lang.org/community  
[tools]: https://www.rust-lang.org/tools
