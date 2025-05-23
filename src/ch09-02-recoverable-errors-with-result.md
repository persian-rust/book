## خطاهای قابل بازیابی با `Result`

بیشتر خطاها به اندازه‌ای جدی نیستند که نیاز به توقف کامل برنامه داشته باشند. گاهی اوقات وقتی
یک تابع با شکست مواجه می‌شود، دلیلی وجود دارد که می‌توانید آن را به راحتی تفسیر کرده و به آن
پاسخ دهید. برای مثال، اگر بخواهید یک فایل را باز کنید و این عملیات به دلیل وجود نداشتن فایل شکست
بخورد، ممکن است بخواهید فایل را ایجاد کنید به جای اینکه فرآیند را متوقف کنید.

به یاد بیاورید از بخش [“Handling Potential Failure with `Result`”][handle_failure]<!-- ignore -->
در فصل ۲ که `Result` به صورت یک enum تعریف شده که دو حالت دارد، `Ok` و `Err`، به صورت زیر:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` و `E` پارامترهای نوع جنریک هستند: ما درباره جنریک‌ها به طور کامل‌تر در فصل ۱۰ صحبت خواهیم کرد.
چیزی که اکنون باید بدانید این است که `T` نمایانگر نوع مقداری است که در حالت موفقیت در داخل `Ok`
بازگردانده می‌شود، و `E` نمایانگر نوع خطایی است که در حالت شکست در داخل `Err` بازگردانده می‌شود.
زیرا `Result` این پارامترهای نوع جنریک را دارد، می‌توانیم نوع `Result` و توابع تعریف شده روی آن را
در بسیاری از شرایط مختلف که مقادیر موفقیت و خطا ممکن است متفاوت باشند، استفاده کنیم.

بیایید تابعی را فراخوانی کنیم که یک مقدار `Result` را بازمی‌گرداند زیرا این تابع ممکن است با شکست
مواجه شود. در لیست ۹-۳ سعی می‌کنیم یک فایل را باز کنیم.

<Listing number="9-3" file-name="src/main.rs" caption="باز کردن یک فایل">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-03/src/main.rs}}
```

</Listing>

نوع بازگشتی `File::open` یک `Result<T, E>` است. پارامتر نوع جنریک `T` توسط پیاده‌سازی
`File::open` با نوع مقدار موفقیت، یعنی `std::fs::File`، که یک فایل هندل است، مقداردهی
شده است. نوع `E` استفاده شده در مقدار خطا `std::io::Error` است. این نوع بازگشتی به این معنی
است که فراخوانی `File::open` ممکن است موفقیت‌آمیز باشد و یک فایل هندل بازگرداند که می‌توانیم از
آن برای خواندن یا نوشتن استفاده کنیم. همچنین ممکن است این فراخوانی با شکست مواجه شود: برای مثال،
فایل ممکن است وجود نداشته باشد یا ممکن است مجوز دسترسی به فایل را نداشته باشیم. تابع `File::open`
باید روشی داشته باشد تا به ما بگوید که آیا موفقیت‌آمیز بود یا شکست خورد و در عین حال فایل هندل یا
اطلاعات خطا را به ما بدهد. این اطلاعات دقیقاً همان چیزی است که enum `Result` منتقل می‌کند.

در حالتی که `File::open` موفقیت‌آمیز باشد، مقدار در متغیر `greeting_file_result` یک نمونه از `Ok`
خواهد بود که یک فایل هندل را شامل می‌شود. در حالتی که با شکست مواجه شود، مقدار در
`greeting_file_result` یک نمونه از `Err` خواهد بود که اطلاعات بیشتری در مورد نوع خطایی که رخ
داده است را شامل می‌شود.

باید به کد در لیست ۹-۳ اضافه کنیم تا اقدامات متفاوتی بسته به مقداری که `File::open` بازمی‌گرداند
انجام دهیم. لیست ۹-۴ یک روش برای مدیریت `Result` با استفاده از یک ابزار پایه، یعنی عبارت `match`
که در فصل ۶ مورد بحث قرار گرفت، نشان می‌دهد.

<Listing number="9-4" file-name="src/main.rs" caption="استفاده از عبارت `match` برای مدیریت حالت‌های `Result` که ممکن است بازگردانده شود">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-04/src/main.rs}}
```

</Listing>

توجه داشته باشید که مانند enum `Option`، enum `Result` و حالات آن به وسیله prelude به محدوده آورده شده‌اند، بنابراین نیازی نیست قبل از حالات `Ok` و `Err` در بازوهای `match` از `Result::` استفاده کنیم.

وقتی نتیجه `Ok` باشد، این کد مقدار داخلی `file` را از حالت `Ok` بازمی‌گرداند و سپس آن مقدار فایل هندل را به متغیر `greeting_file` اختصاص می‌دهیم. بعد از `match`، می‌توانیم از فایل هندل برای خواندن یا نوشتن استفاده کنیم.

بازوی دیگر `match` حالت زمانی را مدیریت می‌کند که از `File::open` یک مقدار `Err` دریافت می‌کنیم. در این مثال، تصمیم گرفته‌ایم ماکروی `panic!` را فراخوانی کنیم. اگر فایل _hello.txt_ در دایرکتوری فعلی ما وجود نداشته باشد و این کد را اجرا کنیم، خروجی زیر را از ماکروی `panic!` خواهیم دید:

```console
{{#include ../listings/ch09-error-handling/listing-09-04/output.txt}}
```

مثل همیشه، این خروجی دقیقاً به ما می‌گوید چه اشتباهی رخ داده است.

### مطابقت بر اساس خطاهای مختلف

کد در لیست ۹-۴ در هر صورتی که `File::open` با شکست مواجه شود، ماکروی `panic!` را فراخوانی می‌کند. با این حال، ما می‌خواهیم اقدامات متفاوتی برای دلایل مختلف شکست انجام دهیم. اگر `File::open` به دلیل وجود نداشتن فایل شکست بخورد، می‌خواهیم فایل را ایجاد کنیم و هندل فایل جدید را بازگردانیم. اگر `File::open` به دلایل دیگری شکست بخورد—برای مثال، به دلیل نداشتن مجوز باز کردن فایل—همچنان می‌خواهیم کد مانند لیست ۹-۴ `panic!` کند. برای این کار، یک عبارت `match` داخلی اضافه می‌کنیم که در لیست ۹-۵ نشان داده شده است.

<Listing number="9-5" file-name="src/main.rs" caption="مدیریت انواع مختلف خطاها به روش‌های مختلف">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-05/src/main.rs}}
```

</Listing>

نوع مقداری که `File::open` درون حالت `Err` بازمی‌گرداند، `io::Error` است که یک ساختار داده ارائه شده توسط کتابخانه استاندارد است. این ساختار دارای متدی به نام `kind` است که می‌توانیم آن را برای دریافت مقدار `io::ErrorKind` فراخوانی کنیم. enum `io::ErrorKind` توسط کتابخانه استاندارد ارائه شده و شامل حالت‌هایی است که انواع مختلف خطاهای ممکن در یک عملیات `io` را نمایش می‌دهد. حالتی که می‌خواهیم از آن استفاده کنیم `ErrorKind::NotFound` است که نشان می‌دهد فایل مورد نظر برای باز کردن هنوز وجود ندارد. بنابراین، ما بر روی `greeting_file_result` مطابقت می‌دهیم، اما همچنین یک `match` داخلی بر روی `error.kind()` داریم.

شرطی که می‌خواهیم در `match` داخلی بررسی کنیم این است که آیا مقدار بازگردانده شده توسط `error.kind()` همان حالت `NotFound` از enum `ErrorKind` است یا خیر. اگر چنین باشد، سعی می‌کنیم فایل را با `File::create` ایجاد کنیم. با این حال، از آنجایی که `File::create` نیز ممکن است شکست بخورد، به یک بازوی دوم در عبارت `match` داخلی نیاز داریم. هنگامی که فایل نمی‌تواند ایجاد شود، یک پیام خطای متفاوت چاپ می‌شود. بازوی دوم `match` بیرونی به همان شکل باقی می‌ماند، بنابراین برنامه برای هر خطایی به جز خطای وجود نداشتن فایل، با خطا متوقف می‌شود.

> #### جایگزین‌هایی برای استفاده از `match` با `Result<T, E>`
>
> استفاده از `match` زیاد است! عبارت `match` بسیار مفید است اما همچنان ابتدایی محسوب می‌شود.
> در فصل ۱۳، درباره closures یاد خواهید گرفت که در بسیاری از متدهایی که روی `Result<T, E>`
> تعریف شده‌اند استفاده می‌شوند. این متدها می‌توانند هنگام مدیریت مقادیر `Result<T, E>` در کد شما،
> مختصرتر از استفاده از `match` باشند.
>
> برای مثال، در اینجا راه دیگری برای نوشتن همان منطق نشان داده شده در لیست ۹-۵ آورده شده است،
> این بار با استفاده از closures و متد `unwrap_or_else`:
>
> ```rust,ignore
> use std::fs::File;
> use std::io::ErrorKind;
>
> fn main() {
>     let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create("hello.txt").unwrap_or_else(|error| {
>                 panic!("Problem creating the file: {error:?}");
>             })
>         } else {
>             panic!("Problem opening the file: {error:?}");
>         }
>     });
> }
> ```
>
> اگرچه این کد همان رفتار لیست ۹-۵ را دارد، اما شامل هیچ عبارت `match` نیست و خواندن آن تمیزتر است.
> بعد از خواندن فصل ۱۳، به این مثال بازگردید و متد `unwrap_or_else` را در مستندات کتابخانه استاندارد
> بررسی کنید. بسیاری از این متدها می‌توانند عبارت‌های `match` تو در تو را هنگام کار با خطاها ساده کنند.

#### میان‌برهایی برای توقف برنامه در صورت خطا: `unwrap` و `expect`

استفاده از `match` به اندازه کافی خوب کار می‌کند، اما ممکن است کمی طولانی باشد و همیشه به خوبی نیت
را منتقل نکند. نوع `Result<T, E>` دارای بسیاری از متدهای کمکی است که برای انجام وظایف خاص‌تر تعریف
شده‌اند. متد `unwrap` یک روش میان‌بر است که دقیقاً مانند عبارت `match` که در لیست ۹-۴ نوشتیم،
پیاده‌سازی شده است. اگر مقدار `Result` در حالت `Ok` باشد، `unwrap` مقدار داخل `Ok` را بازمی‌گرداند.
اگر مقدار `Result` در حالت `Err` باشد، `unwrap` ماکروی `panic!` را برای ما فراخوانی می‌کند. در اینجا
یک مثال از استفاده از `unwrap` آورده شده است:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-04-unwrap/src/main.rs}}
```

</Listing>

اگر این کد را بدون فایل _hello.txt_ اجرا کنیم، یک پیام خطا از فراخوانی `panic!` که متد `unwrap` انجام
می‌دهد خواهیم دید:

```text
thread 'main' panicked at src/main.rs:4:49:
called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

به همین ترتیب، متد `expect` به ما اجازه می‌دهد پیام خطای ماکروی `panic!` را نیز انتخاب کنیم. استفاده
از `expect` به جای `unwrap` و ارائه پیام‌های خطای خوب می‌تواند نیت شما را بهتر منتقل کند و پیگیری منبع
یک خطا را آسان‌تر کند. سینتکس `expect` به این شکل است:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-05-expect/src/main.rs}}
```

</Listing>

ما از `expect` به همان شیوه‌ای استفاده می‌کنیم که از `unwrap` استفاده می‌کنیم: برای بازگرداندن فایل هندل یا فراخوانی ماکروی `panic!`. پیام خطایی که توسط `expect` در فراخوانی `panic!` استفاده می‌شود، پارامتری است که ما به `expect` می‌دهیم، به جای پیام پیش‌فرض `panic!` که توسط `unwrap` استفاده می‌شود. اینجا چیزی است که به نظر می‌رسد:

```text
thread 'main' panicked at src/main.rs:5:10:
hello.txt should be included in this project: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

در کد با کیفیت تولید، بیشتر Rustaceanها `expect` را به جای `unwrap` انتخاب می‌کنند و اطلاعات بیشتری درباره اینکه چرا عملیات باید همیشه موفقیت‌آمیز باشد ارائه می‌دهند. به این ترتیب، اگر فرضیات شما هرگز اشتباه ثابت شوند، اطلاعات بیشتری برای استفاده در اشکال‌زدایی خواهید داشت.

### انتشار خطاها (Propagating Errors)

وقتی پیاده‌سازی یک تابع چیزی را فراخوانی می‌کند که ممکن است شکست بخورد، به جای مدیریت خطا درون خود تابع، می‌توانید خطا را به کدی که تابع را فراخوانی کرده است بازگردانید تا تصمیم بگیرد چه کاری انجام دهد. این به عنوان _انتشار خطا_ شناخته می‌شود و کنترل بیشتری به کدی که فراخوانی می‌کند می‌دهد، جایی که ممکن است اطلاعات یا منطقی وجود داشته باشد که تعیین کند چگونه باید خطا مدیریت شود بیشتر از آنچه در زمینه کد شما موجود است.

برای مثال، لیست ۹-۶ یک تابع را نشان می‌دهد که یک نام کاربری را از یک فایل می‌خواند. اگر فایل وجود نداشته باشد یا قابل خواندن نباشد، این تابع آن خطاها را به کدی که تابع را فراخوانی کرده بازمی‌گرداند.

<Listing number="9-6" file-name="src/main.rs" caption="یک تابع که خطاها را به کد فراخوانی‌کننده بازمی‌گرداند با استفاده از `match`">

```rust
{{#include ../listings/ch09-error-handling/listing-09-06/src/main.rs:here}}
```

</Listing>

این تابع می‌تواند به روشی بسیار کوتاه‌تر نوشته شود، اما ما قرار است با انجام بسیاری از کارها به صورت دستی، مدیریت خطاها را بررسی کنیم. در انتها، راه کوتاه‌تر را نشان خواهیم داد. بیایید ابتدا به نوع بازگشتی تابع نگاه کنیم: `Result<String, io::Error>`. این به این معناست که تابع مقداری از نوع `Result<T, E>` بازمی‌گرداند، جایی که پارامتر جنریک `T` با نوع مشخص `String` مقداردهی شده است و نوع جنریک `E` با نوع مشخص `io::Error`.

اگر این تابع بدون هیچ مشکلی موفقیت‌آمیز باشد، کدی که این تابع را فراخوانی می‌کند یک مقدار `Ok` دریافت می‌کند که یک `String` را نگهداری می‌کند—نام کاربری‌ای که این تابع از فایل خوانده است. اگر این تابع با مشکلی مواجه شود، کدی که آن را فراخوانی کرده است یک مقدار `Err` دریافت می‌کند که یک نمونه از `io::Error` را نگهداری می‌کند که اطلاعات بیشتری درباره مشکلاتی که رخ داده‌اند شامل می‌شود. ما `io::Error` را به عنوان نوع بازگشتی این تابع انتخاب کردیم زیرا این همان نوعی است که مقدار خطا از هر دو عملیات فراخوانی شده در بدنه این تابع که ممکن است شکست بخورند بازمی‌گرداند: تابع `File::open` و متد `read_to_string`.

بدنه تابع با فراخوانی تابع `File::open` شروع می‌شود. سپس مقدار `Result` را با یک `match` مشابه
آنچه در لیست ۹-۴ دیدیم مدیریت می‌کنیم. اگر `File::open` موفق شود، هندل فایل در متغیر الگو `file`
به مقدار در متغیر قابل تغییر `username_file` تبدیل می‌شود و تابع ادامه می‌یابد. در حالت `Err`،
به جای فراخوانی `panic!`، از کلیدواژه `return` استفاده می‌کنیم تا زودتر از تابع خارج شویم و مقدار
خطا از `File::open` که اکنون در متغیر الگو `e` قرار دارد را به کدی که تابع را فراخوانی کرده بازگردانیم.

بنابراین، اگر یک هندل فایل در `username_file` داشته باشیم، تابع سپس یک `String` جدید در متغیر
`username` ایجاد کرده و متد `read_to_string` را روی هندل فایل در `username_file` فراخوانی می‌کند
تا محتوای فایل را در `username` بخواند. متد `read_to_string` نیز یک مقدار `Result` بازمی‌گرداند
زیرا ممکن است با شکست مواجه شود، حتی اگر `File::open` موفق بوده باشد. بنابراین، به یک `match`
دیگر برای مدیریت آن `Result` نیاز داریم: اگر `read_to_string` موفق شود، آنگاه تابع ما موفقیت‌آمیز
بوده و نام کاربری از فایل که اکنون در `username` است، درون یک `Ok` بازمی‌گرداند. اگر
`read_to_string` شکست بخورد، مقدار خطا را به همان شیوه‌ای که مقدار خطا را در `match` که مقدار
بازگشتی `File::open` را مدیریت می‌کرد بازمی‌گردانیم. با این حال، نیازی نیست که به صراحت بگوییم
`return`، زیرا این آخرین عبارت در تابع است.

کدی که این تابع را فراخوانی می‌کند سپس مدیریت دریافت مقدار `Ok` که شامل یک نام کاربری است یا
مقدار `Err` که شامل یک `io::Error` است را انجام می‌دهد. این به کدی که تابع را فراخوانی می‌کند بستگی دارد
که تصمیم بگیرد با این مقادیر چه کاری انجام دهد. اگر کد فراخوانی‌کننده یک مقدار `Err` دریافت کند،
می‌تواند `panic!` را فراخوانی کرده و برنامه را متوقف کند، از یک نام کاربری پیش‌فرض استفاده کند، یا
به جای فایل نام کاربری را از مکان دیگری جستجو کند، برای مثال. ما اطلاعات کافی درباره اینکه کد فراخوانی‌کننده
دقیقاً چه می‌خواهد انجام دهد نداریم، بنابراین تمام اطلاعات موفقیت یا خطا را به بالا منتقل می‌کنیم
تا آن را به درستی مدیریت کند.

این الگوی انتشار خطاها در Rust آن‌قدر رایج است که Rust عملگر `?` را برای آسان‌تر کردن این کار
فراهم می‌کند.

#### یک میان‌بر برای انتشار خطاها: عملگر `?`

لیست ۹-۷ پیاده‌سازی `read_username_from_file` را نشان می‌دهد که همان عملکرد لیست ۹-۶ را دارد،
اما این پیاده‌سازی از عملگر `?` استفاده می‌کند.

<Listing number="9-7" file-name="src/main.rs" caption="یک تابع که خطاها را به کد فراخوانی‌کننده با استفاده از عملگر `?` بازمی‌گرداند">

```rust
{{#include ../listings/ch09-error-handling/listing-09-07/src/main.rs:here}}
```

</Listing>

عملگر `?` که پس از یک مقدار `Result` قرار می‌گیرد تقریباً به همان شیوه‌ای عمل می‌کند که عبارات
`match` که برای مدیریت مقادیر `Result` در لیست ۹-۶ تعریف کردیم. اگر مقدار `Result` در حالت
`Ok` باشد، مقدار درون `Ok` از این عبارت بازگردانده می‌شود و برنامه ادامه می‌یابد. اگر مقدار در حالت
`Err` باشد، مقدار `Err` از کل تابع بازگردانده می‌شود به گونه‌ای که انگار کلیدواژه `return` را
استفاده کرده‌ایم تا مقدار خطا به کد فراخوانی‌کننده منتقل شود.

تفاوتی بین کاری که عبارت `match` در لیست ۹-۶ انجام می‌دهد و کاری که عملگر `?` انجام می‌دهد وجود
دارد: مقادیر خطا که عملگر `?` روی آن‌ها فراخوانی می‌شود از طریق تابع `from` که در ویژگی
`From` کتابخانه استاندارد تعریف شده است عبور می‌کنند، که برای تبدیل مقادیر از یک نوع به نوع دیگر
استفاده می‌شود. وقتی عملگر `?` تابع `from` را فراخوانی می‌کند، نوع خطای دریافت شده به نوع خطای
تعریف شده در نوع بازگشتی تابع فعلی تبدیل می‌شود. این موضوع زمانی مفید است که یک تابع یک نوع خطا
را برای نمایش تمام راه‌هایی که ممکن است یک تابع شکست بخورد بازگرداند، حتی اگر بخش‌هایی ممکن است
به دلایل بسیار مختلفی شکست بخورند.

برای مثال، می‌توانیم تابع `read_username_from_file` در لیست ۹-۷ را تغییر دهیم تا یک نوع خطای سفارشی به نام `OurError` که تعریف کرده‌ایم بازگرداند. اگر همچنین `impl From<io::Error> for OurError` را تعریف کنیم تا یک نمونه از `OurError` را از یک `io::Error` بسازد، سپس فراخوانی‌های عملگر `?` در بدنه تابع `read_username_from_file` تابع `from` را فراخوانی کرده و نوع خطاها را بدون نیاز به افزودن کد اضافی به تابع تبدیل می‌کنند.

در زمینه لیست ۹-۷، عملگر `?` در انتهای فراخوانی `File::open` مقدار درون یک `Ok` را به متغیر `username_file` بازمی‌گرداند. اگر خطایی رخ دهد، عملگر `?` زودتر از کل تابع خارج شده و هر مقدار `Err` را به کد فراخوانی‌کننده بازمی‌گرداند. همین موضوع برای عملگر `?` در انتهای فراخوانی `read_to_string` صدق می‌کند.

عملگر `?` مقدار زیادی از کد اضافی را حذف کرده و پیاده‌سازی این تابع را ساده‌تر می‌کند. حتی می‌توانیم این کد را بیشتر کوتاه کنیم با زنجیره کردن فراخوانی متدها بلافاصله بعد از `?`، همانطور که در لیست ۹-۸ نشان داده شده است.

<Listing number="9-8" file-name="src/main.rs" caption="زنجیره کردن فراخوانی متدها پس از عملگر `?`">

```rust
{{#include ../listings/ch09-error-handling/listing-09-08/src/main.rs:here}}
```

</Listing>

ما ایجاد `String` جدید در `username` را به ابتدای تابع منتقل کرده‌ایم؛ آن قسمت تغییر نکرده است. به جای ایجاد یک متغیر `username_file`، ما فراخوانی `read_to_string` را مستقیماً به نتیجه `File::open("hello.txt")?` زنجیره کرده‌ایم. همچنان یک عملگر `?` در انتهای فراخوانی `read_to_string` داریم و همچنان مقدار `Ok` شامل `username` را زمانی که هر دو `File::open` و `read_to_string` موفق هستند بازمی‌گردانیم، به جای بازگرداندن خطاها. عملکرد دوباره همانند لیست ۹-۶ و لیست ۹-۷ است؛ این فقط یک روش متفاوت و کاربرپسندتر برای نوشتن آن است.

لیست ۹-۹ روشی برای کوتاه‌تر کردن این کد با استفاده از `fs::read_to_string` را نشان می‌دهد.

<Listing number="9-9" file-name="src/main.rs" caption="استفاده از `fs::read_to_string` به جای باز کردن و سپس خواندن فایل">

```rust
{{#include ../listings/ch09-error-handling/listing-09-09/src/main.rs:here}}
```

</Listing>

خواندن یک فایل به یک رشته یک عملیات نسبتاً رایج است، بنابراین کتابخانه استاندارد تابع مناسب `fs::read_to_string` را فراهم می‌کند که فایل را باز می‌کند، یک `String` جدید ایجاد می‌کند، محتوای فایل را می‌خواند، محتوا را در آن `String` قرار می‌دهد و آن را بازمی‌گرداند. البته، استفاده از `fs::read_to_string` به ما فرصتی برای توضیح تمام مدیریت خطاها نمی‌دهد، بنابراین ابتدا آن را به روش طولانی‌تر انجام دادیم.

#### جایی که می‌توان از عملگر `?` استفاده کرد

عملگر `?` فقط در توابعی استفاده می‌شود که نوع بازگشتی آن‌ها با مقدار استفاده شده توسط `?` سازگار باشد. این به این دلیل است که عملگر `?` برای بازگرداندن زودهنگام یک مقدار از تابع تعریف شده است، به همان شیوه‌ای که عبارت `match` در لیست ۹-۶ تعریف شده است. در لیست ۹-۶، `match` از یک مقدار `Result` استفاده می‌کرد و بازوی بازگشتی زودهنگام یک مقدار `Err(e)` را بازمی‌گرداند. نوع بازگشتی تابع باید یک `Result` باشد تا با این بازگشت سازگار باشد.

در لیست ۹-۱۰، بیایید به خطایی که دریافت می‌کنیم وقتی که از عملگر `?` در یک تابع `main` با نوع بازگشتی‌ای که با نوع مقدار استفاده شده در `?` سازگار نیست استفاده می‌کنیم نگاه کنیم.

<Listing number="9-10" file-name="src/main.rs" caption="تلاش برای استفاده از `?` در تابع `main` که نوع بازگشتی آن `()` است و کامپایل نمی‌شود.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-10/src/main.rs}}
```

</Listing>

این کد یک فایل را باز می‌کند، که ممکن است شکست بخورد. عملگر `?` مقدار `Result` بازگردانده شده توسط `File::open` را دنبال می‌کند، اما این تابع `main` نوع بازگشتی `()` دارد، نه `Result`. وقتی این کد را کامپایل می‌کنیم، پیام خطای زیر را دریافت می‌کنیم:

```console
{{#include ../listings/ch09-error-handling/listing-09-10/output.txt}}
```

این خطا نشان می‌دهد که فقط می‌توان از عملگر `?` در توابعی که نوع بازگشتی آن‌ها `Result`، `Option`، یا نوع دیگری که `FromResidual` را پیاده‌سازی می‌کند استفاده کرد.

برای رفع این خطا، دو انتخاب دارید. یکی این است که نوع بازگشتی تابع خود را تغییر دهید تا با مقداری که از عملگر `?` استفاده می‌کنید سازگار باشد، به شرطی که محدودیتی مانع از انجام این کار نداشته باشید. انتخاب دیگر این است که از یک `match` یا یکی از متدهای `Result<T, E>` برای مدیریت `Result<T, E>` به شیوه‌ای که مناسب است استفاده کنید.

پیام خطا همچنین اشاره کرد که `?` می‌تواند با مقادیر `Option<T>` نیز استفاده شود. همانند استفاده از `?` روی `Result`، فقط می‌توانید از `?` روی `Option` در تابعی استفاده کنید که یک `Option` بازمی‌گرداند. رفتار عملگر `?` وقتی روی یک `Option<T>` فراخوانی می‌شود شبیه به رفتار آن وقتی روی یک `Result<T, E>` فراخوانی می‌شود: اگر مقدار `None` باشد، `None` زودهنگام از تابع بازگردانده می‌شود. اگر مقدار `Some` باشد، مقدار داخل `Some` مقدار نتیجه عبارت است و تابع ادامه می‌دهد. لیست ۹-۱۱ مثالی از تابعی را نشان می‌دهد که آخرین کاراکتر خط اول متن داده شده را پیدا می‌کند.

<Listing number="9-11" caption="استفاده از عملگر `?` روی یک مقدار `Option<T>`">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-11/src/main.rs:here}}
```

</Listing>

این تابع `Option<char>` بازمی‌گرداند زیرا ممکن است یک کاراکتر وجود داشته باشد، اما ممکن است وجود نداشته باشد. این کد آرگومان قطعه رشته `text` را می‌گیرد و متد `lines` را روی آن فراخوانی می‌کند، که یک iterator روی خطوط درون رشته بازمی‌گرداند. چون این تابع می‌خواهد خط اول را بررسی کند، `next` را روی iterator فراخوانی می‌کند تا اولین مقدار از iterator را دریافت کند. اگر `text` رشته‌ای خالی باشد، این فراخوانی به `next` مقدار `None` بازمی‌گرداند، که در این صورت از `?` برای متوقف کردن و بازگرداندن `None` از `last_char_of_first_line` استفاده می‌کنیم. اگر `text` رشته خالی نباشد، `next` یک مقدار `Some` شامل یک قطعه رشته از خط اول در `text` بازمی‌گرداند.

عملگر `?` قطعه رشته را استخراج می‌کند و می‌توانیم متد `chars` را روی آن فراخوانی کنیم تا یک iterator از کاراکترهای آن دریافت کنیم. ما به آخرین کاراکتر در این خط اول علاقه‌مند هستیم، بنابراین متد `last` را فراخوانی می‌کنیم تا آخرین مورد در iterator را بازگرداند. این یک `Option` است زیرا ممکن است خط اول رشته‌ای خالی باشد؛ برای مثال، اگر `text` با یک خط خالی شروع شود اما کاراکترهایی در خطوط دیگر داشته باشد، مانند `"\nhi"`. با این حال، اگر آخرین کاراکتری در خط اول وجود داشته باشد، در حالت `Some` بازگردانده می‌شود. عملگر `?` در میانه به ما راهی مختصر برای بیان این منطق می‌دهد و اجازه می‌دهد تابع را در یک خط پیاده‌سازی کنیم. اگر نمی‌توانستیم از عملگر `?` روی `Option` استفاده کنیم، باید این منطق را با فراخوانی متدهای بیشتر یا یک عبارت `match` پیاده‌سازی می‌کردیم.

توجه داشته باشید که می‌توانید از عملگر `?` روی یک `Result` در یک تابع که یک `Result` بازمی‌گرداند استفاده کنید، و می‌توانید از عملگر `?` روی یک `Option` در یک تابع که یک `Option` بازمی‌گرداند استفاده کنید، اما نمی‌توانید این دو را با هم ترکیب کنید. عملگر `?` به طور خودکار یک `Result` را به یک `Option` یا برعکس تبدیل نمی‌کند؛ در این موارد، می‌توانید از متدهایی مانند `ok` روی `Result` یا `ok_or` روی `Option` برای تبدیل صریح استفاده کنید.

تا کنون، تمام توابع `main` که استفاده کرده‌ایم مقدار `()` بازمی‌گرداندند. تابع `main` خاص است زیرا نقطه ورود و خروج یک برنامه اجرایی است، و محدودیت‌هایی در نوع بازگشتی آن وجود دارد تا برنامه همانطور که انتظار می‌رود رفتار کند.

خوشبختانه، `main` می‌تواند یک `Result<(), E>` نیز بازگرداند. لیست ۹-۱۲ کد لیست ۹-۱۰ را دارد، اما نوع بازگشتی `main` را به `Result<(), Box<dyn Error>>` تغییر داده‌ایم و یک مقدار بازگشتی `Ok(())` به انتهای آن اضافه کرده‌ایم. این کد اکنون کامپایل می‌شود.

<Listing number="9-12" file-name="src/main.rs" caption="تغییر تابع `main` برای بازگرداندن `Result<(), E>` اجازه می‌دهد از عملگر `?` روی مقادیر `Result` استفاده شود.">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-12/src/main.rs}}
```

</Listing>

نوع `Box<dyn Error>` یک _شیء ویژگی_ (trait object) است که در بخش [“Using Trait Objects that Allow for Values of Different Types”][trait-objects]<!-- ignore --> در فصل ۱۸ درباره آن صحبت خواهیم کرد. در حال حاضر، می‌توانید `Box<dyn Error>` را به معنای "هر نوع خطا" در نظر بگیرید. استفاده از `?` روی یک مقدار `Result` در یک تابع `main` با نوع خطای `Box<dyn Error>` مجاز است زیرا این امکان را می‌دهد که هر مقدار `Err` زودتر بازگردانده شود. اگرچه بدنه این تابع `main` فقط خطاهای نوع `std::io::Error` را بازمی‌گرداند، با مشخص کردن `Box<dyn Error>`، این امضا حتی اگر کد بیشتری که خطاهای دیگری بازمی‌گرداند به بدنه `main` اضافه شود، صحیح باقی می‌ماند.

وقتی یک تابع `main` یک `Result<(), E>` بازمی‌گرداند، برنامه اجرایی با مقدار `0` خارج می‌شود اگر `main` مقدار `Ok(())` بازگرداند و با یک مقدار غیر صفر خارج می‌شود اگر `main` مقدار `Err` بازگرداند. برنامه‌های اجرایی نوشته شده در C هنگام خروج مقادیر صحیح بازمی‌گردانند: برنامه‌هایی که با موفقیت خارج می‌شوند مقدار صحیح `0` را بازمی‌گردانند و برنامه‌هایی که دچار خطا می‌شوند مقداری غیر از `0` بازمی‌گردانند. Rust نیز مقادیر صحیح را از برنامه‌های اجرایی بازمی‌گرداند تا با این قرارداد سازگار باشد.

تابع `main` می‌تواند هر نوعی را که ویژگی [`std::process::Termination`][termination]<!-- ignore --> را پیاده‌سازی می‌کند بازگرداند، که شامل تابع `report` است که یک `ExitCode` بازمی‌گرداند. مستندات کتابخانه استاندارد را برای اطلاعات بیشتر درباره پیاده‌سازی ویژگی `Termination` برای انواع خودتان مطالعه کنید.

اکنون که جزئیات فراخوانی `panic!` یا بازگرداندن `Result` را بررسی کردیم، بیایید به موضوع نحوه تصمیم‌گیری درباره اینکه کدامیک در چه مواردی مناسب است بازگردیم.

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result  
[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types  
[termination]: https://doc.rust-lang.org/std/process/trait.Termination.html
