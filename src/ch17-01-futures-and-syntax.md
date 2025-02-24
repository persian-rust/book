## Futures و سینتکس Async

عناصر کلیدی برنامه‌نویسی ناهمزمان در Rust شامل _futures_ و کلمات کلیدی `async` و `await` هستند.

یک _future_ مقداری است که ممکن است اکنون آماده نباشد، اما در آینده در نقطه‌ای آماده خواهد شد. (این مفهوم در بسیاری از زبان‌ها وجود دارد، گاهی با نام‌های دیگر مانند _task_ یا _promise_.) Rust یک ویژگی `Future` به عنوان یک بلوک سازنده فراهم می‌کند تا عملیات‌های async مختلف با ساختارهای داده متفاوت اما با یک رابط مشترک پیاده‌سازی شوند. در Rust، futures نوع‌هایی هستند که ویژگی `Future` را پیاده‌سازی می‌کنند. هر future اطلاعات خود را در مورد پیشرفت و اینکه "آماده" به چه معناست نگه می‌دارد.

می‌توانید کلمه کلیدی `async` را به بلوک‌ها و توابع اعمال کنید تا مشخص کنید که می‌توانند متوقف شده و از سر گرفته شوند. درون یک بلوک async یا تابع async، می‌توانید از کلمه کلیدی `await` برای _انتظار یک future_ (یعنی منتظر ماندن تا آماده شود) استفاده کنید. هر نقطه‌ای که در آن یک future را در یک بلوک یا تابع async انتظار می‌کشید، یک نقطه بالقوه برای متوقف و از سر گرفتن آن بلوک یا تابع async است. فرآیند بررسی یک future برای اینکه ببیند مقدار آن هنوز آماده است یا خیر، _polling_ نامیده می‌شود.

برخی زبان‌های دیگر، مانند C# و JavaScript، نیز از کلمات کلیدی `async` و `await` برای برنامه‌نویسی ناهمزمان استفاده می‌کنند. اگر با این زبان‌ها آشنا هستید، ممکن است تفاوت‌های قابل توجهی در نحوه عملکرد Rust، از جمله نحوه مدیریت سینتکس آن، مشاهده کنید. این تفاوت‌ها دلایل خوبی دارند، همان‌طور که خواهیم دید!

هنگام نوشتن کد async در Rust، بیشتر اوقات از کلمات کلیدی `async` و `await` استفاده می‌کنیم. Rust آن‌ها را به کدی معادل با استفاده از ویژگی `Future` کامپایل می‌کند، همان‌طور که حلقه‌های `for` را به کدی معادل با استفاده از ویژگی `Iterator` کامپایل می‌کند. با این حال، از آنجا که Rust ویژگی `Future` را ارائه می‌دهد، می‌توانید آن را برای نوع‌های داده خودتان نیز پیاده‌سازی کنید. بسیاری از توابعی که در طول این فصل مشاهده خواهیم کرد نوع‌هایی را بازمی‌گردانند که پیاده‌سازی‌های خود از `Future` را دارند. در انتهای فصل به تعریف این ویژگی بازمی‌گردیم و بیشتر در مورد نحوه عملکرد آن بحث می‌کنیم، اما این توضیحات برای ادامه کافی است.

ممکن است این توضیحات کمی انتزاعی به نظر برسند، بنابراین بیایید اولین برنامه async خود را بنویسیم: یک web scraper کوچک. ما دو URL را از خط فرمان دریافت می‌کنیم، هر دو را به صورت همزمان دریافت می‌کنیم و نتیجه اولین URL که به پایان می‌رسد را بازمی‌گردانیم. این مثال دارای سینتکس جدیدی خواهد بود، اما نگران نباشید—همه چیزهایی که باید بدانید را در طول مسیر توضیح خواهیم داد.

## اولین برنامه Async ما

برای تمرکز این فصل روی یادگیری async به جای مدیریت بخش‌های اکوسیستم، یک crate به نام `trpl` ایجاد کرده‌ایم (`trpl` مخفف "The Rust Programming Language" است). این crate همه نوع‌ها، ویژگی‌ها، و توابع مورد نیاز شما را بازصادر می‌کند، عمدتاً از crateهای [`futures`][futures-crate]<!-- ignore --> و [`tokio`][tokio]<!-- ignore -->. crate `futures` خانه رسمی برای آزمایش کد async در Rust است و در واقع جایی است که ویژگی `Future` در ابتدا طراحی شد. `tokio` امروز رایج‌ترین Runtime async در Rust است، به ویژه برای برنامه‌های وب. Runtimeهای عالی دیگری نیز وجود دارند که ممکن است برای اهداف شما مناسب‌تر باشند. ما از crate `tokio` در زیرساخت `trpl` استفاده می‌کنیم زیرا به خوبی تست شده و به طور گسترده استفاده می‌شود.

در برخی موارد، `trpl` همچنین APIهای اصلی را تغییر نام داده یا آن‌ها را پوشش می‌دهد تا شما را بر روی جزئیات مرتبط با این فصل متمرکز نگه دارد. اگر می‌خواهید بفهمید این crate چه می‌کند، ما شما را تشویق می‌کنیم که [سورس کد آن][crate-source]<!-- ignore --> را بررسی کنید. می‌توانید ببینید که هر بازصادر از کدام crate می‌آید، و توضیحات گسترده‌ای در مورد آنچه که crate انجام می‌دهد گذاشته‌ایم.

یک پروژه باینری جدید به نام `hello-async` ایجاد کنید و crate `trpl` را به عنوان وابستگی اضافه کنید:

```console
$ cargo new hello-async
$ cd hello-async
$ cargo add trpl
```

اکنون می‌توانیم از بخش‌های مختلف ارائه‌شده توسط `trpl` استفاده کنیم تا اولین برنامه async خود را بنویسیم. ما یک ابزار کوچک خط فرمان ایجاد خواهیم کرد که دو صفحه وب را دریافت می‌کند، عنصر `<title>` را از هرکدام استخراج می‌کند و عنوان صفحه‌ای که سریع‌تر کل این فرآیند را تکمیل می‌کند، چاپ می‌کند.

### تعریف تابع `page_title`

بیایید با نوشتن یک تابع که یک URL صفحه را به عنوان پارامتر می‌گیرد، یک درخواست به آن ارسال می‌کند و متن عنصر `<title>` را بازمی‌گرداند شروع کنیم (نگاه کنید به لیست ۱۷-۱).

<Listing number="17-1" file-name="src/main.rs" caption="تعریف یک تابع async برای دریافت عنصر `<title>` از یک صفحه HTML">


```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-01/src/main.rs:all}}
```

</Listing>

ابتدا یک تابع به نام `page_title` تعریف می‌کنیم و آن را با کلمه کلیدی `async` علامت‌گذاری می‌کنیم. سپس از تابع `trpl::get` برای دریافت هر URL که به آن ارسال می‌شود استفاده می‌کنیم و کلمه کلیدی `await` را اضافه می‌کنیم تا منتظر پاسخ بمانیم. برای دریافت متن پاسخ، متد `text` را فراخوانی می‌کنیم و دوباره با کلمه کلیدی `await` منتظر آن می‌مانیم. هر دو این مراحل ناهمزمان هستند. برای تابع `get`، باید منتظر باشیم تا سرور اولین قسمت از پاسخ خود را ارسال کند که شامل هدرهای HTTP، کوکی‌ها و غیره است و می‌تواند جدا از بدنه پاسخ ارسال شود. به ویژه اگر بدنه بسیار بزرگ باشد، ممکن است مدتی طول بکشد تا همه آن برسد. از آنجا که باید منتظر _تمامیت_ پاسخ بمانیم، متد `text` نیز async است.

باید به‌صراحت منتظر هر دو future باشیم، زیرا futures در Rust _تنبل_ هستند: تا زمانی که از آن‌ها با کلمه کلیدی `await` درخواست نشود، هیچ کاری انجام نمی‌دهند. (در واقع، Rust یک هشدار کامپایلر نمایش می‌دهد اگر از یک future استفاده نکنید.) این ممکن است شما را به یاد بحث فصل ۱۳ درباره iteratorها در بخش [پردازش یک سری از آیتم‌ها با iteratorها][iterators-lazy]<!-- ignore --> بیندازد. iteratorها هیچ کاری انجام نمی‌دهند مگر اینکه متد `next` آن‌ها را فراخوانی کنید—چه به صورت مستقیم یا با استفاده از حلقه‌های `for` یا متدهایی مانند `map` که در پشت صحنه از `next` استفاده می‌کنند. به همین ترتیب، futures هیچ کاری انجام نمی‌دهند مگر اینکه به‌صراحت از آن‌ها درخواست شود. این ویژگی تنبلی به Rust اجازه می‌دهد تا کد async را تا زمانی که واقعاً مورد نیاز است، اجرا نکند.

> نکته: این رفتار متفاوت از چیزی است که در فصل قبلی هنگام استفاده از `thread::spawn` در [ایجاد یک Thread جدید با `spawn`][thread-spawn]<!-- ignore --> مشاهده کردیم، جایی که Closureی که به یک Thread دیگر ارسال کردیم بلافاصله شروع به اجرا کرد. همچنین، این رفتار با نحوه استفاده بسیاری از زبان‌های دیگر از async متفاوت است. اما این برای Rust مهم است و بعداً خواهیم دید چرا.

وقتی `response_text` را داریم، می‌توانیم آن را با استفاده از `Html::parse` به یک نمونه از نوع `Html` تجزیه کنیم. به جای یک رشته خام، اکنون یک نوع داده داریم که می‌توانیم از آن برای کار با HTML به عنوان یک ساختار داده غنی‌تر استفاده کنیم. به طور خاص، می‌توانیم از متد `select_first` برای پیدا کردن اولین نمونه از یک انتخابگر CSS خاص استفاده کنیم. با ارسال رشته `"title"`، اولین عنصر `<title>` در سند را دریافت خواهیم کرد، اگر وجود داشته باشد. چون ممکن است هیچ عنصر مطابقتی وجود نداشته باشد، `select_first` یک `Option<ElementRef>` بازمی‌گرداند. در نهایت، از متد `Option::map` استفاده می‌کنیم که به ما اجازه می‌دهد با آیتم موجود در `Option` کار کنیم، اگر موجود باشد، و اگر موجود نباشد، هیچ کاری انجام ندهیم. (می‌توانستیم از یک عبارت `match` هم استفاده کنیم، اما `map` بیشتر idiomatic است.) در بدنه تابعی که به `map` می‌دهیم، متد `inner_html` را روی `title_element` فراخوانی می‌کنیم تا محتوای آن را که یک `String` است، دریافت کنیم. وقتی همه چیز انجام شد، یک `Option<String>` خواهیم داشت.

توجه کنید که کلمه کلیدی `await` در Rust _بعد از_ عبارت مورد انتظار قرار می‌گیرد، نه قبل از آن. یعنی این یک کلمه کلیدی _postfix_ است. این ممکن است با چیزی که به آن عادت دارید اگر از async در زبان‌های دیگر استفاده کرده باشید، متفاوت باشد، اما در Rust این کار زنجیره‌ای از متدها را بسیار راحت‌تر می‌کند. در نتیجه، می‌توانیم بدنه `page_url_for` را تغییر دهیم تا فراخوانی‌های تابع `trpl::get` و `text` را با `await` بین آن‌ها به هم زنجیر کنیم، همان‌طور که در لیست ۱۷-۲ نشان داده شده است.

<Listing number="17-2" file-name="src/main.rs" caption="زنجیره کردن با کلمه کلیدی `await`">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-02/src/main.rs:chaining}}
```

</Listing>

با این توضیحات، ما اولین تابع async خود را با موفقیت نوشتیم! پیش از اضافه کردن کدی در `main` برای فراخوانی آن، بیایید کمی بیشتر درباره آنچه نوشته‌ایم و معنای آن صحبت کنیم.

هنگامی که Rust یک بلوک که با کلمه کلیدی `async` علامت‌گذاری شده است را می‌بیند، آن را به یک نوع داده منحصربه‌فرد و ناشناس که ویژگی `Future` را پیاده‌سازی می‌کند، کامپایل می‌کند. هنگامی که Rust یک تابع که با `async` علامت‌گذاری شده است را می‌بیند، آن را به یک تابع غیر-async که بدنه آن یک بلوک async است، کامپایل می‌کند. نوع بازگشتی یک تابع async نوع داده ناشناسی است که کامپایلر برای آن بلوک async ایجاد می‌کند.

بنابراین، نوشتن `async fn` معادل نوشتن تابعی است که یک _future_ از نوع بازگشتی برمی‌گرداند. برای کامپایلر، یک تعریف تابع مانند `async fn page_title` در لیست ۱۷-۱ معادل یک تابع غیر-async به شکل زیر است:


```rust
# extern crate trpl; // required for mdbook test
use std::future::Future;
use trpl::Html;

fn page_title(url: &str) -> impl Future<Output = Option<String>> + '_ {
    async move {
        let text = trpl::get(url).await.text().await;
        Html::parse(&text)
            .select_first("title")
            .map(|title| title.inner_html())
    }
}
```

بیایید هر بخش از نسخه تبدیل‌شده را بررسی کنیم:

- از سینتکس `impl Trait` که در فصل ۱۰ در بخش [“ویژگی‌ها به عنوان پارامتر”][impl-trait]<!-- ignore --> بحث کردیم، استفاده می‌کند.
- ویژگی بازگردانده‌شده یک `Future` با یک نوع وابسته به نام `Output` است. توجه کنید که نوع `Output` برابر با `Option<String>` است، که همان نوع بازگشتی نسخه اصلی `async fn` تابع `page_title` است.
- تمام کدی که در بدنه تابع اصلی فراخوانی شده است، در یک بلوک `async move` بسته‌بندی شده است. به یاد داشته باشید که بلوک‌ها بیان (_expression_) هستند. این بلوک کامل، بیانی است که از تابع بازگردانده می‌شود.
- این بلوک async یک مقداری با نوع `Option<String>` تولید می‌کند، همان‌طور که توضیح داده شد. این مقدار با نوع `Output` در نوع بازگشتی مطابقت دارد. این درست مانند بلوک‌های دیگری است که قبلاً دیده‌اید.
- بدنه جدید تابع یک بلوک `async move` است به دلیل نحوه استفاده از پارامتر `url`. (در ادامه فصل بیشتر درباره تفاوت `async` و `async move` صحبت خواهیم کرد.)
- نسخه جدید تابع دارای نوعی طول عمر است که قبلاً ندیده‌ایم: `'_`. از آنجا که تابع یک future بازمی‌گرداند که به یک مرجع اشاره می‌کند—در این مورد، مرجعی که از پارامتر `url` آمده است—باید به Rust بگوییم که می‌خواهیم آن مرجع شامل شود. نیازی نیست طول عمر را اینجا نام‌گذاری کنیم، زیرا Rust به اندازه کافی هوشمند است که بفهمد فقط یک مرجع می‌تواند درگیر باشد، اما باید صراحتاً مشخص کنیم که future حاصل به آن طول عمر محدود شده است.

حالا می‌توانیم `page_title` را در `main` فراخوانی کنیم.

## تعیین عنوان یک صفحه

برای شروع، فقط عنوان یک صفحه را دریافت می‌کنیم. در لیست ۱۷-۳، همان الگویی که در فصل ۱۲ برای دریافت آرگومان‌های خط فرمان در بخش [پذیرفتن آرگومان‌های خط فرمان][cli-args]<!-- ignore --> استفاده کردیم را دنبال می‌کنیم. سپس URL اول را به `page_title` ارسال کرده و نتیجه را انتظار می‌کشیم. چون مقداری که توسط future تولید می‌شود یک `Option<String>` است، از یک عبارت `match` برای چاپ پیام‌های مختلف استفاده می‌کنیم تا مشخص شود آیا صفحه یک `<title>` داشته است یا خیر.

<Listing number="17-3" file-name="src/main.rs" caption="Calling the `page_title` function from `main` with a user-supplied argument">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-03/src/main.rs:main}}
```

</Listing>

متأسفانه، این کد کامپایل نمی‌شود. تنها جایی که می‌توانیم از کلمه کلیدی `await` استفاده کنیم، در توابع یا بلوک‌های async است، و Rust اجازه نمی‌دهد تابع ویژه `main` را به‌عنوان `async` علامت‌گذاری کنیم.


<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-03
cargo build
copy just the compiler error
-->

```text
error[E0752]: `main` function is not allowed to be `async`
 --> src/main.rs:6:1
  |
6 | async fn main() {
  | ^^^^^^^^^^^^^^^ `main` function is not allowed to be `async`
```

دلیل اینکه نمی‌توان `main` را به‌عنوان `async` علامت‌گذاری کرد این است که کد async به یک _runtime_ نیاز دارد: یک crate در Rust که جزئیات اجرای کد ناهمزمان را مدیریت می‌کند. تابع `main` یک برنامه می‌تواند یک runtime را _مقداردهی اولیه_ کند، اما خودش یک runtime نیست. (در ادامه، بیشتر خواهیم دید که چرا این‌گونه است.) هر برنامه Rust که کد async اجرا می‌کند، حداقل یک مکان دارد که در آن یک runtime راه‌اندازی کرده و futures را اجرا می‌کند.

بیشتر زبان‌هایی که از async پشتیبانی می‌کنند، یک runtime همراه دارند، اما Rust این کار را نمی‌کند. در عوض، بسیاری از runtimeهای async مختلف موجود هستند که هرکدام موازنه‌های متفاوتی برای موارد استفاده خاص خود ارائه می‌دهند. برای مثال، یک وب سرور با توان عملیاتی بالا که دارای هسته‌های CPU متعدد و مقدار زیادی RAM است، نیازهای بسیار متفاوتی نسبت به یک میکروکنترلر با یک هسته، مقدار کمی RAM و بدون قابلیت تخصیص heap دارد. crateهایی که این runtimeها را فراهم می‌کنند اغلب نسخه‌های async از قابلیت‌های عمومی مانند I/O فایل یا شبکه را نیز ارائه می‌دهند.

اینجا و در بقیه این فصل، از تابع `run` از crate `trpl` استفاده خواهیم کرد، که یک future را به‌عنوان آرگومان می‌گیرد و آن را تا پایان اجرا می‌کند. در پشت صحنه، فراخوانی `run` یک runtime راه‌اندازی می‌کند که برای اجرای future ارسال‌شده استفاده می‌شود. وقتی future کامل شد، `run` هر مقداری که future تولید کرده باشد، بازمی‌گرداند.

می‌توانستیم future بازگردانده‌شده توسط `page_title` را مستقیماً به `run` ارسال کنیم، و وقتی کامل شد، می‌توانستیم بر اساس `Option<String>` نتیجه، یک `match` انجام دهیم، همان‌طور که در لیست ۱۷-۳ تلاش کردیم. با این حال، برای بیشتر مثال‌های این فصل (و بیشتر کد async در دنیای واقعی)، بیش از یک فراخوانی تابع async انجام خواهیم داد، بنابراین به‌جای آن یک بلوک `async` ارسال می‌کنیم و صراحتاً نتیجه فراخوانی `page_title` را انتظار می‌کشیم، همان‌طور که در لیست ۱۷-۴ نشان داده شده است.

<Listing number="17-4" caption="منتظر ماندن یک بلوک async با `trpl::run`" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook test does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-04/src/main.rs:run}}
```

</Listing>

وقتی این کد را اجرا می‌کنیم، رفتاری را که ممکن است ابتدا انتظار داشتیم دریافت می‌کنیم:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-04
cargo build # skip all the build noise
cargo run https://www.rust-lang.org
# copy the output here
-->

```console
$ cargo run -- https://www.rust-lang.org
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/async_await 'https://www.rust-lang.org'`
The title for https://www.rust-lang.org was
            Rust Programming Language
```

پوووف—بالاخره مقداری کد async کارا داریم! اما قبل از اینکه کدی اضافه کنیم که دو سایت را در مقابل یکدیگر رقابت دهد، بیایید به‌طور مختصر دوباره به نحوه کار futures توجه کنیم.

هر _نقطه انتظار_—یعنی هر جایی که کد از کلمه کلیدی `await` استفاده می‌کند—نمایانگر جایی است که کنترل به runtime بازمی‌گردد. برای اینکه این کار انجام شود، Rust نیاز دارد وضعیت مربوط به بلوک async را پیگیری کند تا runtime بتواند کار دیگری را آغاز کند و سپس وقتی آماده شد دوباره برای پیشرفت بلوک اول بازگردد. این یک ماشین حالت نامرئی است، گویی که شما یک enum مانند این نوشته‌اید تا وضعیت فعلی را در هر نقطه انتظار ذخیره کند:

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-state-machine/src/lib.rs:enum}}
```

نوشتن کدی که به صورت دستی بین هر حالت انتقال یابد خسته‌کننده و مستعد خطا خواهد بود، به‌ویژه زمانی که بخواهید عملکرد بیشتری اضافه کرده و حالات بیشتری به کد اضافه کنید. خوشبختانه، کامپایلر Rust به طور خودکار ساختارهای داده مربوط به ماشین حالت را برای کد async ایجاد و مدیریت می‌کند. قوانین عادی مالکیت و قرض‌گیری در مورد ساختارهای داده همچنان اعمال می‌شوند، و خوشبختانه، کامپایلر بررسی این موارد را نیز برای ما انجام می‌دهد و پیام‌های خطای مفیدی ارائه می‌دهد. در ادامه فصل چند مورد از این پیام‌ها را بررسی خواهیم کرد.

در نهایت، چیزی باید این ماشین حالت را اجرا کند، و آن چیز یک runtime است. (به همین دلیل ممکن است در بررسی runtimeها به ارجاعاتی به _executors_ برخورد کنید: یک executor بخشی از runtime است که مسئول اجرای کد async است.)

حالا می‌توانید بفهمید چرا کامپایلر مانع شد که `main` خودش به عنوان یک تابع async در لیست ۱۷-۳ تعریف شود. اگر `main` یک تابع async بود، چیزی دیگری باید ماشین حالت را برای futureی که `main` بازمی‌گرداند مدیریت می‌کرد، اما `main` نقطه شروع برنامه است! در عوض، ما تابع `trpl::run` را در `main` فراخوانی کردیم تا یک runtime راه‌اندازی کند و future بازگردانده‌شده توسط بلوک `async` را تا زمانی که `Ready` بازگرداند، اجرا کند.

> نکته: برخی runtimeها ماکروهایی ارائه می‌دهند که به شما اجازه می‌دهند یک تابع async برای `main` بنویسید. این ماکروها `async fn main() { ... }` را به یک `fn main` عادی تبدیل می‌کنند، که همان کاری را انجام می‌دهد که ما به صورت دستی در لیست ۱۷-۵ انجام دادیم: فراخوانی یک تابع که یک future را به طور کامل اجرا می‌کند، همان‌طور که `trpl::run` انجام می‌دهد.

حالا بیایید این بخش‌ها را کنار هم قرار دهیم و ببینیم چگونه می‌توان کدی همزمان نوشت.

### رقابت بین دو URL

در لیست ۱۷-۵، ما `page_title` را با دو URL مختلف که از خط فرمان ارسال شده‌اند، فراخوانی کرده و آن‌ها را با یکدیگر رقابت می‌دهیم.


<Listing number="17-5" caption="" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-05/src/main.rs:all}}
```

</Listing>

ما با فراخوانی `page_title` برای هر یک از URLهایی که توسط کاربر ارسال شده‌اند، شروع می‌کنیم. Futureهای حاصل را به نام‌های `title_fut_1` و `title_fut_2` ذخیره می‌کنیم. به یاد داشته باشید، این‌ها هنوز کاری انجام نمی‌دهند، زیرا futures تنبل هستند و هنوز منتظر آن‌ها نمانده‌ایم. سپس این futures را به `trpl::race` ارسال می‌کنیم، که مقداری بازمی‌گرداند تا نشان دهد کدام یک از futures ارسال‌شده به آن ابتدا کامل شده است.

> نکته: در پشت صحنه، `race` بر اساس یک تابع عمومی‌تر به نام `select` ساخته شده است، که اغلب در کدهای واقعی Rust با آن مواجه خواهید شد. یک تابع `select` می‌تواند کارهایی انجام دهد که تابع `trpl::race` نمی‌تواند، اما همچنین دارای پیچیدگی‌های اضافی است که فعلاً می‌توانیم از آن صرف‌نظر کنیم.

هرکدام از futures می‌توانند به طور قانونی "برنده" شوند، بنابراین بازگرداندن یک `Result` منطقی نیست. در عوض، `race` نوعی را بازمی‌گرداند که قبلاً ندیده‌ایم: `trpl::Either`. نوع `Either` تا حدودی شبیه به `Result` است به این معنا که دو حالت دارد. اما برخلاف `Result`، هیچ مفهومی از موفقیت یا شکست در `Either` وجود ندارد. در عوض، از `Left` و `Right` برای نشان دادن "یکی یا دیگری" استفاده می‌کند:

```rust
enum Either<A, B> {
    Left(A),
    Right(B),
}
```

تابع `race` مقدار `Left` را با خروجی future اول بازمی‌گرداند اگر آرگومان اول برنده شود، و مقدار `Right` را با خروجی future دوم بازمی‌گرداند اگر _آن_ یکی برنده شود. این ترتیب با ترتیبی که آرگومان‌ها هنگام فراخوانی تابع ظاهر می‌شوند مطابقت دارد: آرگومان اول در سمت چپ آرگومان دوم قرار دارد.

همچنین تابع `page_title` را به‌روزرسانی می‌کنیم تا همان URL ارسال‌شده را بازگرداند. به این ترتیب، اگر صفحه‌ای که ابتدا بازمی‌گردد، دارای یک `<title>` نباشد که بتوانیم آن را استخراج کنیم، همچنان می‌توانیم یک پیام معنادار چاپ کنیم. با در دسترس بودن این اطلاعات، خروجی `println!` خود را به‌روزرسانی می‌کنیم تا مشخص کند کدام URL اول کامل شده است و `<title>` صفحه وب در آن URL چیست (اگر وجود داشته باشد).

شما اکنون یک web scraper کوچک و کارا ساخته‌اید! چند URL انتخاب کنید و ابزار خط فرمان را اجرا کنید. ممکن است متوجه شوید که برخی سایت‌ها به طور مداوم سریع‌تر از بقیه هستند، در حالی که در موارد دیگر، سایت سریع‌تر از اجرای به اجرای دیگر متفاوت است. مهم‌تر از همه، شما اصول کار با futures را آموخته‌اید، بنابراین حالا می‌توانیم عمیق‌تر به آنچه می‌توان با async انجام داد، بپردازیم.


[impl-trait]: ch10-02-traits.html#traits-as-parameters
[iterators-lazy]: ch13-02-iterators.html
[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[cli-args]: ch12-01-accepting-command-line-arguments.html

<!-- TODO: map source link version to version of Rust? -->

[crate-source]: https://github.com/persian-rust/book/tree/main/packages/trpl
[futures-crate]: https://crates.io/crates/futures
[tokio]: https://tokio.rs
