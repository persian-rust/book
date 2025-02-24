## مقایسه عملکرد: حلقه‌ها در برابر Iteratorها

برای تعیین اینکه از حلقه‌ها یا iteratorها استفاده کنید، باید بدانید کدام پیاده‌سازی سریع‌تر است: نسخه تابع `search` با حلقه صریح `for` یا نسخه با iteratorها.

ما یک بنچمارک اجرا کردیم که در آن تمام محتوای کتاب _The Adventures of Sherlock Holmes_ اثر سر آرتور کانن دویل را در یک `String` بارگذاری کردیم و به دنبال کلمه _the_ در محتوا گشتیم. نتایج بنچمارک برای نسخه `search` با استفاده از حلقه `for` و نسخه با iteratorها به شرح زیر است:

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

دو پیاده‌سازی عملکرد مشابهی دارند! ما کد بنچمارک (benchmark) را اینجا توضیح نمی‌دهیم، زیرا هدف این نیست که ثابت کنیم این دو نسخه معادل هستند، بلکه هدف این است که به یک درک کلی از نحوه مقایسه عملکردی این دو پیاده‌سازی برسیم.

برای یک بنچمارک جامع‌تر، باید از متن‌های مختلف با اندازه‌های گوناگون به‌عنوان `contents`، کلمات مختلف و کلماتی با طول‌های متفاوت به‌عنوان `query`، و انواع دیگری از تغییرات استفاده کنید. نکته این است: iteratorها، اگرچه یک انتزاع سطح بالا هستند، به کدی که تقریباً همان سطح پایینی دارد کامپایل می‌شوند، انگار خودتان کد سطح پایین را نوشته باشید. iteratorها یکی از _انتزاع‌های بدون هزینه_ Rust هستند، به این معنی که استفاده از انتزاع هیچ هزینه اضافی زمان اجرای برنامه را تحمیل نمی‌کند. این موضوع مشابه تعریفی است که بیارنه استراس‌تروپ، طراح و پیاده‌ساز اصلی ++C، در مقاله "Foundations of C++" (2012) برای _بدون هزینه اضافی_ ارائه می‌دهد:

> به طور کلی، پیاده‌سازی‌های ++C از اصل بدون هزینه اضافی پیروی می‌کنند: چیزی که استفاده نمی‌کنید، هزینه‌ای برای شما ندارد. و علاوه بر این: چیزی که استفاده می‌کنید، نمی‌توانید بهتر از این دستی کدنویسی کنید.


به‌عنوان یک مثال دیگر، کد زیر از یک دیکودر صوتی گرفته شده است. الگوریتم دیکودینگ از عملیات ریاضی پیش‌بینی خطی برای تخمین مقادیر آینده بر اساس یک تابع خطی از نمونه‌های قبلی استفاده می‌کند. این کد از یک زنجیره iterator برای انجام برخی محاسبات بر روی سه متغیر در محدوده استفاده می‌کند: یک برش داده‌ای `buffer`، یک آرایه از ۱۲ `coefficients`، و مقداری برای جابجایی داده‌ها در `qlp_shift`. ما متغیرها را در این مثال تعریف کرده‌ایم اما به آن‌ها مقداری نداده‌ایم؛ اگرچه این کد خارج از زمینه خود معنای زیادی ندارد، اما همچنان یک مثال مختصر و واقعی از نحوه تبدیل ایده‌های سطح بالا به کد سطح پایین در Rust است.

```rust,ignore
let buffer: &mut [i32];
let coefficients: [i64; 12];
let qlp_shift: i16;

for i in 12..buffer.len() {
    let prediction = coefficients.iter()
                                 .zip(&buffer[i - 12..i])
                                 .map(|(&c, &s)| c * s as i64)
                                 .sum::<i64>() >> qlp_shift;
    let delta = buffer[i];
    buffer[i] = prediction as i32 + delta;
}
```

برای محاسبه مقدار `prediction`، این کد از طریق هر یک از ۱۲ مقدار در `coefficients` پیمایش می‌کند و از متد `zip` برای جفت کردن مقادیر coefficients با ۱۲ مقدار قبلی در `buffer` استفاده می‌کند. سپس، برای هر جفت، مقادیر را در هم ضرب می‌کنیم، تمام نتایج را جمع می‌کنیم، و بیت‌های حاصل را به اندازه `qlp_shift` بیت به سمت راست جابجا می‌کنیم.

محاسبات در برنامه‌هایی مانند دیکودرهای صوتی اغلب عملکرد را در اولویت قرار می‌دهند. در اینجا، ما یک iterator ایجاد می‌کنیم، از دو تطبیق‌دهنده استفاده می‌کنیم، و سپس مقدار را مصرف می‌کنیم. کد اسمبلی که این کد Rust به آن کامپایل می‌شود چیست؟ خب، در زمان نگارش این متن، این کد به همان اسمبلی‌ای که ممکن است دستی بنویسید کامپایل می‌شود. هیچ حلقه‌ای وجود ندارد که با پیمایش روی مقادیر در `coefficients` مطابقت داشته باشد: Rust می‌داند که ۱۲ تکرار وجود دارد، بنابراین حلقه را "بازمی‌پیچد". _بازپیچیدن_ یک بهینه‌سازی است که سربار کد کنترل‌کننده حلقه را حذف می‌کند و به جای آن کد تکراری برای هر تکرار حلقه تولید می‌کند.

تمام مقادیر coefficients در ثبات‌ها ذخیره می‌شوند، به این معنی که دسترسی به مقادیر بسیار سریع است. در زمان اجرا هیچ بررسی حدودی برای دسترسی به آرایه انجام نمی‌شود. تمام این بهینه‌سازی‌هایی که Rust می‌تواند اعمال کند کد نهایی را به شدت کارآمد می‌سازد. حالا که این را می‌دانید، می‌توانید از iteratorها و closureها بدون ترس استفاده کنید! آن‌ها باعث می‌شوند کد سطح بالاتر به نظر برسد اما هیچ هزینه عملکردی در زمان اجرا اعمال نمی‌کنند.

## خلاصه

<div dir="rtl">
closureها و iteratorها ویژگی‌های Rust هستند که از ایده‌های زبان‌های برنامه‌نویسی تابعی الهام گرفته‌اند. آن‌ها به توانایی Rust در بیان واضح ایده‌های سطح بالا با عملکرد سطح پایین کمک می‌کنند. پیاده‌سازی closureها و iteratorها به گونه‌ای است که عملکرد زمان اجرا تحت تأثیر قرار نمی‌گیرد. این بخشی از هدف Rust برای ارائه انتزاع‌های بدون هزینه است.
</div>

اکنون که قابلیت بیان پروژه I/O خود را بهبود داده‌ایم، بیایید نگاهی به برخی ویژگی‌های بیشتر `cargo` بیندازیم که به ما کمک می‌کنند پروژه را با دنیا به اشتراک بگذاریم.
