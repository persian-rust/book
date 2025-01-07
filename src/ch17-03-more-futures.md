```markdown
## کار با هر تعداد آینده

وقتی در بخش قبلی از استفاده از دو آینده به سه آینده تغییر دادیم، مجبور شدیم از `join` به `join3` تغییر دهیم. این که هر بار که تعداد آینده‌هایی که می‌خواهیم منتظرشان بمانیم تغییر کند، مجبور باشیم تابع متفاوتی فراخوانی کنیم آزاردهنده است. خوشبختانه، فرم ماکروی `join` وجود دارد که می‌توانیم هر تعداد آرگومان دلخواه به آن ارسال کنیم. این ماکرو همچنین منتظر ماندن برای آینده‌ها را خود مدیریت می‌کند. بنابراین، می‌توانیم کد فهرست 17-13 را بازنویسی کنیم تا به جای `join3` از `join!` استفاده کنیم، همان‌طور که در فهرست 17-14 آمده است:

<Listing number="17-14" caption="استفاده از `join!` برای منتظر ماندن چندین آینده" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-14/src/main.rs:here}}
```

</Listing>

این قطعاً بهبود خوبی نسبت به نیاز به تعویض بین `join`، `join3`، `join4` و ... است! با این حال، حتی این فرم ماکرو فقط زمانی کار می‌کند که تعداد آینده‌ها را از قبل بدانیم. اما در راست دنیای واقعی، افزودن آینده‌ها به یک مجموعه و سپس منتظر ماندن برای برخی یا همه آینده‌های آن مجموعه تا تکمیل، یک الگوی رایج است.

برای بررسی تمام آینده‌ها در یک مجموعه، باید روی _همه_ آن‌ها تکرار کرده و منتظر بمانیم. تابع `trpl::join_all` هر نوعی را که ویژگی `Iterator` را پیاده‌سازی کرده باشد می‌پذیرد، که درباره آن در فصل 13 آموختیم، بنابراین به نظر می‌رسد دقیقاً چیزی است که نیاز داریم. بیایید آینده‌های خود را در یک بردار قرار دهیم و `join!` را با `join_all` جایگزین کنیم.

<Listing number="17-15" caption="ذخیره آینده‌های ناشناس در یک بردار و فراخوانی `join_all`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-15/src/main.rs:here}}
```

</Listing>

متأسفانه، این کد کامپایل نمی‌شود. در عوض، این خطا را دریافت می‌کنیم:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-15/
cargo build
copy just the compiler error
-->

```text
error[E0308]: mismatched types
  --> src/main.rs:45:37
   |
10 |         let tx1_fut = async move {
   |                       ---------- the expected `async` block
...
24 |         let rx_fut = async {
   |                      ----- the found `async` block
...
45 |         let futures = vec![tx1_fut, rx_fut, tx_fut];
   |                                     ^^^^^^ expected `async` block, found a different `async` block
   |
   = note: expected `async` block `{async block@src/main.rs:10:23: 10:33}`
              found `async` block `{async block@src/main.rs:24:22: 24:27}`
   = note: no two async blocks, even if identical, have the same type
   = help: consider pinning your async block and casting it to a trait object
```

```markdown
این ممکن است تعجب‌آور باشد. بالاخره هیچ‌کدام چیزی برنمی‌گردانند، بنابراین هر بلوک یک `Future<Output = ()>` تولید می‌کند. با این حال، `Future` یک ویژگی (trait) است، نه یک نوع مشخص (concrete type). نوع‌های مشخص، ساختارهای داده‌ای هستند که توسط کامپایلر برای بلوک‌های async تولید می‌شوند. نمی‌توانید دو ساختار مختلف نوشته‌شده با دست را در یک `Vec` قرار دهید، و همین امر در مورد ساختارهای مختلف تولیدشده توسط کامپایلر نیز صدق می‌کند.

برای کار کردن این کد، باید از _شیءهای ویژگی_ (trait objects) استفاده کنیم، همانطور که در [«بازگرداندن خطاها از تابع run»][dyn]<!-- ignore --> در فصل 12 انجام دادیم. (در فصل 18 شیءهای ویژگی را به‌طور مفصل بررسی خواهیم کرد.) استفاده از شیءهای ویژگی به ما اجازه می‌دهد که هر یک از آینده‌های ناشناس تولیدشده توسط این نوع‌ها را به‌عنوان یک نوع مشابه در نظر بگیریم، زیرا همه آن‌ها ویژگی `Future` را پیاده‌سازی می‌کنند.

> نکته: در فصل 8، روش دیگری برای شامل کردن چندین نوع در یک `Vec` را بررسی کردیم: استفاده از یک enum برای نشان دادن هر یک از نوع‌های مختلف که می‌توانند در بردار ظاهر شوند. با این حال، اینجا نمی‌توانیم از آن استفاده کنیم. اولاً، راهی برای نام‌گذاری نوع‌های مختلف نداریم، زیرا آن‌ها ناشناس هستند. ثانیاً، دلیلی که در ابتدا به بردار و `join_all` روی آوردیم این بود که بتوانیم با مجموعه‌ای پویا از آینده‌ها کار کنیم که نمی‌دانیم همه آن‌ها چه خواهند بود تا زمان اجرا.

ابتدا هر یک از آینده‌ها در `vec!` را در یک `Box::new` قرار می‌دهیم، همانطور که در فهرست 17-16 نشان داده شده است.

<Listing number="17-16" caption="تلاش برای استفاده از `Box::new` برای یکسان کردن نوع آینده‌ها در یک `Vec`" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-16/src/main.rs:here}}
```

</Listing>

متأسفانه، این همچنان کامپایل نمی‌شود. در واقع، همان خطای اصلی قبلی را دریافت می‌کنیم، اما این بار برای هر دو فراخوانی `Box::new` دوم و سوم، و همچنین خطاهای جدیدی مربوط به ویژگی `Unpin`. به خطاهای مربوط به `Unpin` در لحظه بازمی‌گردیم. ابتدا خطاهای نوع روی فراخوانی‌های `Box::new` را با صراحتاً حاشیه‌نویسی کردن نوع متغیر `futures` برطرف کنیم:

<Listing number="17-17" caption="برطرف کردن بقیه خطاهای ناسازگاری نوع با استفاده از اعلان صریح نوع" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-17/src/main.rs:here}}
```

</Listing>

نوعی که اینجا باید می‌نوشتیم کمی پیچیده است، بنابراین بیایید آن را بررسی کنیم:

- نوع داخلی‌ترین، آینده خود است. به‌طور صریح بیان می‌کنیم که خروجی آینده نوع واحد `()` است، با نوشتن `Future<Output = ()>`.
- سپس ویژگی را با `dyn` برای علامت‌گذاری آن به‌عنوان پویا (dynamic) حاشیه‌نویسی می‌کنیم.
- کل مرجع ویژگی در یک `Box` قرار داده می‌شود.
- در نهایت، به‌صراحت بیان می‌کنیم که `futures` یک `Vec` است که این آیتم‌ها را شامل می‌شود.

این تغییرات تأثیر زیادی داشت. حالا وقتی کامپایلر را اجرا می‌کنیم، فقط خطاهایی که به `Unpin` اشاره دارند باقی می‌مانند. اگرچه سه خطا وجود دارند، توجه کنید که هرکدام در محتوا بسیار مشابه هستند.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-16
cargo build
# copy *only* the errors
# fix the paths
-->

```text
error[E0308]: mismatched types
   --> src/main.rs:46:46
    |
10  |         let tx1_fut = async move {
    |                       ---------- the expected `async` block
...
24  |         let rx_fut = async {
    |                      ----- the found `async` block
...
46  |             vec![Box::new(tx1_fut), Box::new(rx_fut), Box::new(tx_fut)];
    |                                     -------- ^^^^^^ expected `async` block, found a different `async` block
    |                                     |
    |                                     arguments to this function are incorrect
    |
    = note: expected `async` block `{async block@src/main.rs:10:23: 10:33}`
               found `async` block `{async block@src/main.rs:24:22: 24:27}`
    = note: no two async blocks, even if identical, have the same type
    = help: consider pinning your async block and casting it to a trait object
note: associated function defined here
   --> file:///home/.rustup/toolchains/1.82/lib/rustlib/src/rust/library/alloc/src/boxed.rs:255:12
    |
255 |     pub fn new(x: T) -> Self {
    |            ^^^

error[E0308]: mismatched types
   --> src/main.rs:46:64
    |
10  |         let tx1_fut = async move {
    |                       ---------- the expected `async` block
...
30  |         let tx_fut = async move {
    |                      ---------- the found `async` block
...
46  |             vec![Box::new(tx1_fut), Box::new(rx_fut), Box::new(tx_fut)];
    |                                                       -------- ^^^^^^ expected `async` block, found a different `async` block
    |                                                       |
    |                                                       arguments to this function are incorrect
    |
    = note: expected `async` block `{async block@src/main.rs:10:23: 10:33}`
               found `async` block `{async block@src/main.rs:30:22: 30:32}`
    = note: no two async blocks, even if identical, have the same type
    = help: consider pinning your async block and casting it to a trait object
note: associated function defined here
   --> file:///home/.rustup/toolchains/1.82/lib/rustlib/src/rust/library/alloc/src/boxed.rs:255:12
    |
255 |     pub fn new(x: T) -> Self {
    |            ^^^

error[E0277]: `{async block@src/main.rs:10:23: 10:33}` cannot be unpinned
   --> src/main.rs:48:24
    |
48  |         trpl::join_all(futures).await;
    |         -------------- ^^^^^^^ the trait `Unpin` is not implemented for `{async block@src/main.rs:10:23: 10:33}`, which is required by `Box<{async block@src/main.rs:10:23: 10:33}>: Future`
    |         |
    |         required by a bound introduced by this call
    |
    = note: consider using the `pin!` macro
            consider using `Box::pin` if you need to access the pinned value outside of the current scope
    = note: required for `Box<{async block@src/main.rs:10:23: 10:33}>` to implement `Future`
note: required by a bound in `join_all`
   --> file:///home/.cargo/registry/src/index.crates.io-6f17d22bba15001f/futures-util-0.3.30/src/future/join_all.rs:105:14
    |
102 | pub fn join_all<I>(iter: I) -> JoinAll<I::Item>
    |        -------- required by a bound in this function
...
105 |     I::Item: Future,
    |              ^^^^^^ required by this bound in `join_all`

error[E0277]: `{async block@src/main.rs:10:23: 10:33}` cannot be unpinned
  --> src/main.rs:48:9
   |
48 |         trpl::join_all(futures).await;
   |         ^^^^^^^^^^^^^^^^^^^^^^^ the trait `Unpin` is not implemented for `{async block@src/main.rs:10:23: 10:33}`, which is required by `Box<{async block@src/main.rs:10:23: 10:33}>: Future`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<{async block@src/main.rs:10:23: 10:33}>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-6f17d22bba15001f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`

error[E0277]: `{async block@src/main.rs:10:23: 10:33}` cannot be unpinned
  --> src/main.rs:48:33
   |
48 |         trpl::join_all(futures).await;
   |                                 ^^^^^ the trait `Unpin` is not implemented for `{async block@src/main.rs:10:23: 10:33}`, which is required by `Box<{async block@src/main.rs:10:23: 10:33}>: Future`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<{async block@src/main.rs:10:23: 10:33}>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-6f17d22bba15001f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`
```

```markdown
این یک _مقدار_ برای هضم است، بنابراین بیایید آن را جدا کنیم. بخش اول پیام به ما می‌گوید که بلوک async اول (`src/main.rs:8:23: 20:10`) ویژگی `Unpin` را پیاده‌سازی نمی‌کند و پیشنهاد می‌دهد از `pin!` یا `Box::pin` برای حل آن استفاده کنیم. بعداً در فصل، به جزئیات بیشتری درباره `Pin` و `Unpin` خواهیم پرداخت. اما فعلاً، می‌توانیم فقط از توصیه کامپایلر برای برطرف کردن مشکل پیروی کنیم! در فهرست 17-18، ابتدا با اضافه کردن حاشیه‌نویسی نوع برای `futures`، با یک `Pin` که هر `Box` را می‌پیچد، شروع می‌کنیم. دوم، از `Box::pin` برای pin کردن آینده‌ها استفاده می‌کنیم.

<Listing number="17-18" caption="استفاده از `Pin` و `Box::pin` برای برطرف کردن نوع `Vec`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-18/src/main.rs:here}}
```

</Listing>

اگر این کد را کامپایل و اجرا کنیم، در نهایت خروجی موردنظر خود را دریافت می‌کنیم:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
received 'hi'
received 'more'
received 'from'
received 'messages'
received 'the'
received 'for'
received 'future'
received 'you'
```

آه!

هنوز کمی بیشتر برای بررسی وجود دارد. برای مثال، استفاده از `Pin<Box<T>>` با مقدار کمی سربار اضافی همراه است، زیرا این آینده‌ها را با `Box` روی heap قرار می‌دهیم—و ما فقط این کار را انجام می‌دهیم تا نوع‌ها را هماهنگ کنیم. اما در واقع نیازی به تخصیص heap نداریم، زیرا این آینده‌ها محلی برای این تابع خاص هستند. همان‌طور که در بالا ذکر شد، `Pin` خود یک نوع wrapper است، بنابراین می‌توانیم از مزیت داشتن یک نوع یکتا در `Vec`—دلیل اصلی که به سمت `Box` رفتیم—بدون انجام تخصیص heap استفاده کنیم. می‌توانیم مستقیماً از `Pin` با هر آینده استفاده کنیم، با استفاده از ماکروی `std::pin::pin`.


```markdown
با این حال، باید به‌صراحت نوع مرجع pinned را مشخص کنیم؛ در غیر این صورت، راست همچنان نمی‌داند که این‌ها را به‌عنوان شیءهای ویژگی دینامیک تفسیر کند، که همان چیزی است که برای قرار گرفتن در `Vec` نیاز داریم. بنابراین، هر آینده را وقتی تعریف می‌کنیم `pin!` می‌کنیم و `futures` را به‌عنوان یک `Vec` که شامل مراجع متغیر pinned به نوع ویژگی دینامیک `Future` است تعریف می‌کنیم، همانطور که در فهرست 17-19 نشان داده شده است.

<Listing number="17-19" caption="استفاده مستقیم از `Pin` با ماکروی `pin!` برای اجتناب از تخصیص‌های غیرضروری heap" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-19/src/main.rs:here}}
```

</Listing>

تا اینجا با نادیده گرفتن این واقعیت که ممکن است نوع‌های `Output` مختلفی داشته باشیم، پیش رفتیم. برای مثال، در فهرست 17-20، آینده ناشناس برای `a` ویژگی `Future<Output = u32>` را پیاده‌سازی می‌کند، آینده ناشناس برای `b` ویژگی `Future<Output = &str>` را پیاده‌سازی می‌کند، و آینده ناشناس برای `c` ویژگی `Future<Output = bool>` را پیاده‌سازی می‌کند.

<Listing number="17-20" caption="سه آینده با نوع‌های متفاوت" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-20/src/main.rs:here}}
```

</Listing>

ما می‌توانیم از `trpl::join!` برای منتظر ماندن این آینده‌ها استفاده کنیم، زیرا این ماکرو به شما اجازه می‌دهد چندین نوع آینده را ارسال کنید و یک تاپل از آن نوع‌ها تولید می‌کند. ما _نمی‌توانیم_ از `trpl::join_all` استفاده کنیم، زیرا این تابع نیاز دارد که همه آینده‌های ارسال‌شده به آن نوع یکسانی داشته باشند. به یاد داشته باشید، این همان خطایی است که ما را در این ماجراجویی با `Pin` شروع کرد!

این یک معامله اساسی است: ما می‌توانیم با یک تعداد دینامیک از آینده‌ها با `join_all` کار کنیم، تا زمانی که همه آن‌ها نوع یکسانی داشته باشند، یا می‌توانیم با یک تعداد ثابت از آینده‌ها با توابع `join` یا ماکروی `join!` کار کنیم، حتی اگر نوع‌های متفاوتی داشته باشند. این درست مثل کار با هر نوع دیگری در راست است. آینده‌ها خاص نیستند، حتی اگر سینتکس خوبی برای کار با آن‌ها داشته باشیم، و این چیز خوبی است.

### Racing futures

وقتی آینده‌ها را با خانواده توابع و ماکروهای `join` "منتظر می‌مانیم"، نیاز داریم _همه_ آن‌ها تمام شوند قبل از اینکه به مرحله بعدی برویم. گاهی اوقات، اما، فقط نیاز داریم _یکی_ از آینده‌ها از مجموعه‌ای تمام شود قبل از اینکه به مرحله بعدی برویم—کمی شبیه به مسابقه دادن یک آینده در برابر دیگری.

در فهرست 17-21، دوباره از `trpl::race` برای اجرای دو آینده، `slow` و `fast`، در برابر یکدیگر استفاده می‌کنیم. هرکدام پیامی را هنگام شروع اجرا چاپ می‌کنند، مدتی با فراخوانی و انتظار `sleep` متوقف می‌شوند، و سپس پیامی دیگر را هنگام اتمام چاپ می‌کنند. سپس هر دو را به `trpl::race` ارسال می‌کنیم و منتظر می‌مانیم تا یکی از آن‌ها تمام شود. (نتیجه در اینجا چندان تعجب‌آور نخواهد بود: `fast` برنده می‌شود!) برخلاف زمانی که در [«اولین برنامه async ما»][async-program]<!-- ignore --> از `race` استفاده کردیم، اینجا نمونه `Either` که بازمی‌گرداند را نادیده می‌گیریم، زیرا تمام رفتار جالب در بدنه بلوک‌های async اتفاق می‌افتد.

<Listing number="17-21" caption="استفاده از `race` برای دریافت نتیجه اولین آینده‌ای که تمام می‌شود" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-21/src/main.rs:here}}
```

</Listing>

```markdown
توجه داشته باشید که اگر ترتیب آرگومان‌ها را در `race` تغییر دهید، ترتیب پیام‌های "started" تغییر می‌کند، حتی اگر آینده `fast` همیشه زودتر تمام شود. این به این دلیل است که پیاده‌سازی این تابع خاص `race` منصفانه نیست. همیشه آینده‌هایی که به‌عنوان آرگومان ارسال شده‌اند را به ترتیبی که ارسال شده‌اند اجرا می‌کند. پیاده‌سازی‌های دیگر _منصفانه_ هستند و به‌طور تصادفی انتخاب می‌کنند که کدام آینده را ابتدا بررسی (poll) کنند. با این حال، صرف‌نظر از اینکه پیاده‌سازی `race` که استفاده می‌کنیم منصفانه است یا نه، _یکی_ از آینده‌ها تا اولین `await` در بدنه خود اجرا می‌شود قبل از اینکه تسک دیگری بتواند شروع شود.

به یاد بیاورید از [اولین برنامه async ما][async-program]<!-- ignore --> که در هر نقطه انتظار (await point)، راست به runtime این فرصت را می‌دهد که تسک را متوقف کند و به تسک دیگری سوئیچ کند اگر آینده‌ای که منتظر آن هستید آماده نباشد. عکس آن نیز صادق است: راست _فقط_ بلوک‌های async را در نقاط انتظار متوقف می‌کند و کنترل را به runtime بازمی‌گرداند. همه چیز بین نقاط انتظار همگام (synchronous) است.

این به این معنی است که اگر در یک بلوک async بدون نقطه انتظار کار زیادی انجام دهید، آن آینده مانع از پیشرفت سایر آینده‌ها خواهد شد. ممکن است گاهی اوقات به این موضوع به‌عنوان یک آینده که _آینده‌های دیگر را گرسنه_ می‌کند اشاره شود. در برخی موارد، این ممکن است مشکل بزرگی نباشد. با این حال، اگر در حال انجام برخی تنظیمات گران‌قیمت یا کار طولانی‌مدت هستید، یا اگر آینده‌ای دارید که به‌طور نامحدود یک کار خاص انجام می‌دهد، باید در مورد زمان و مکان بازگرداندن کنترل به runtime فکر کنید.

به همان اندازه، اگر عملیات‌های مسدودکننده طولانی‌مدت دارید، async می‌تواند ابزاری مفید برای ارائه راه‌هایی باشد که بخش‌های مختلف برنامه بتوانند با یکدیگر تعامل داشته باشند.

اما در این موارد _چگونه_ کنترل را به runtime بازمی‌گردانید؟

### Yielding

بیایید یک عملیات طولانی‌مدت را شبیه‌سازی کنیم. فهرست 17-22 یک تابع `slow` معرفی می‌کند. از `std::thread::sleep` به جای `trpl::sleep` استفاده می‌کند تا فراخوانی `slow` نخ فعلی را به مدت مشخصی از میلی‌ثانیه مسدود کند. می‌توانیم از `slow` به‌عنوان جایگزین عملیات‌های واقعی دنیای واقعی استفاده کنیم که هم طولانی‌مدت و هم مسدودکننده هستند.

<Listing number="17-22" caption="استفاده از `thread::sleep` برای شبیه‌سازی عملیات کند" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-22/src/main.rs:slow}}
```

</Listing>

در فهرست 17-23، از `slow` برای شبیه‌سازی انجام این نوع کارهای وابسته به CPU در یک جفت آینده استفاده می‌کنیم. برای شروع، هر آینده فقط پس از انجام یک سری عملیات کند، کنترل را به runtime بازمی‌گرداند.

<Listing number="17-23" caption="استفاده از `thread::sleep` برای شبیه‌سازی عملیات کند" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-23/src/main.rs:slow-futures}}
```

</Listing>

اگر این کد را اجرا کنید، این خروجی را خواهید دید:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-23/
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'a' ran for 10ms
'a' ran for 20ms
'b' started.
'b' ran for 75ms
'b' ran for 10ms
'b' ran for 15ms
'b' ran for 350ms
'a' finished.
```

```markdown
همانطور که در مثال قبلی، `race` همچنان به محض اینکه آینده `a` کامل شود، تمام می‌شود. با این حال، هیچ درهم‌تنیدگی‌ای بین دو آینده وجود ندارد. آینده `a` تمام کار خود را انجام می‌دهد تا زمانی که فراخوانی `trpl::sleep` منتظر بماند، سپس آینده `b` تمام کار خود را انجام می‌دهد تا زمانی که فراخوانی `trpl::sleep` خودش منتظر بماند، و سپس آینده `a` کامل می‌شود. برای اجازه دادن به هر دو آینده برای پیشرفت بین کارهای کند خود، نیاز به نقاط انتظار داریم تا بتوانیم کنترل را به runtime بازگردانیم. این بدان معناست که نیاز به چیزی داریم که بتوانیم منتظر آن بمانیم!

ما از قبل می‌توانیم این نوع واگذاری کنترل را در فهرست 17-23 ببینیم: اگر `trpl::sleep` را در انتهای آینده `a` حذف کنیم، این آینده بدون اجرای آینده `b` _به‌کلی_ کامل می‌شود. شاید بتوانیم از تابع `sleep` به‌عنوان نقطه شروع استفاده کنیم؟

<Listing number="17-24" caption="استفاده از `sleep` برای اجازه دادن به عملیات‌ها برای پیشرفت متناوب" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-24/src/main.rs:here}}
```

</Listing>

در فهرست 17-24، فراخوانی‌های `trpl::sleep` با نقاط انتظار بین هر فراخوانی به `slow` اضافه می‌کنیم. اکنون کار دو آینده درهم‌تنیده شده است:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-24
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'b' started.
'b' ran for 75ms
'a' ran for 10ms
'b' ran for 10ms
'a' ran for 20ms
'b' ran for 15ms
'a' finished.
```

```markdown
آینده `a` هنوز برای مدتی اجرا می‌شود قبل از اینکه کنترل را به `b` واگذار کند، زیرا ابتدا `slow` را فراخوانی می‌کند قبل از اینکه اصلاً `trpl::sleep` را فراخوانی کند، اما پس از آن آینده‌ها هر بار که یکی از آن‌ها به یک نقطه انتظار برخورد می‌کند، به‌طور متناوب جابه‌جا می‌شوند. در این حالت، ما این کار را پس از هر فراخوانی به `slow` انجام داده‌ایم، اما می‌توانیم کار را به هر شکلی که برایمان منطقی‌تر است تقسیم کنیم.

با این حال، واقعاً نمی‌خواهیم اینجا _sleep_ کنیم؛ می‌خواهیم به سریع‌ترین شکلی که می‌توانیم پیشرفت کنیم. فقط نیاز داریم کنترل را به runtime بازگردانیم. می‌توانیم این کار را به‌طور مستقیم با استفاده از تابع `yield_now` انجام دهیم. در فهرست 17-25، تمام این فراخوانی‌های `sleep` را با `yield_now` جایگزین می‌کنیم.

<Listing number="17-25" caption="استفاده از `yield_now` برای اجازه دادن به عملیات‌ها برای پیشرفت متناوب" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-25/src/main.rs:yields}}
```

</Listing>

این روش هم از نظر هدف واضح‌تر است و هم می‌تواند به‌طور قابل‌توجهی سریع‌تر از استفاده از `sleep` باشد، زیرا تایمرها مانند تایمری که توسط `sleep` استفاده می‌شود اغلب محدودیت‌هایی در مورد دقت خود دارند. نسخه‌ای از `sleep` که ما استفاده می‌کنیم، برای مثال، همیشه حداقل به مدت یک میلی‌ثانیه خواب خواهد رفت، حتی اگر یک `Duration` یک نانوثانیه به آن بدهیم. دوباره، کامپیوترهای مدرن _سریع_ هستند: آن‌ها می‌توانند در یک میلی‌ثانیه کارهای زیادی انجام دهند!

می‌توانید این موضوع را خودتان با تنظیم یک بنچمارک کوچک مشاهده کنید، مانند چیزی که در فهرست 17-26 آورده شده است. (این روش به‌طور خاص برای آزمایش عملکرد دقیق نیست، اما برای نشان دادن تفاوت اینجا کافی است.) اینجا، ما تمام چاپ وضعیت را نادیده می‌گیریم، یک `Duration` یک نانوثانیه به `trpl::sleep` ارسال می‌کنیم و اجازه می‌دهیم هر آینده به‌طور مستقل اجرا شود، بدون سوئیچ بین آینده‌ها. سپس برای 1,000 تکرار اجرا می‌کنیم و می‌بینیم آینده‌ای که از `trpl::sleep` استفاده می‌کند چقدر طول می‌کشد در مقایسه با آینده‌ای که از `trpl::yield_now` استفاده می‌کند.

<Listing number="17-26" caption="مقایسه عملکرد `sleep` و `yield_now`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-26/src/main.rs:here}}
```

</Listing>

نسخه با `yield_now` _خیلی_ سریع‌تر است!

این بدان معناست که async حتی برای وظایف وابسته به CPU می‌تواند مفید باشد، بسته به اینکه برنامه شما چه کار دیگری انجام می‌دهد، زیرا ابزاری مفید برای ساختاردهی روابط بین بخش‌های مختلف برنامه فراهم می‌کند. این نوعی از _چندوظیفه‌گی مشارکتی_ است، جایی که هر آینده قدرت تصمیم‌گیری درباره زمان واگذاری کنترل از طریق نقاط انتظار را دارد. بنابراین، هر آینده نیز مسئولیت دارد که از مسدود کردن بیش از حد طولانی اجتناب کند. در برخی سیستم‌عامل‌های مبتنی بر راست برای سیستم‌های تعبیه‌شده، این _تنها_ نوع چندوظیفه‌گی است!

در کد دنیای واقعی، معمولاً با هر خط یک نقطه انتظار جایگزین نمی‌کنید. در حالی که واگذاری کنترل به این روش نسبتاً کم‌هزینه است، رایگان نیست! در بسیاری از موارد، تلاش برای تقسیم یک وظیفه وابسته به CPU ممکن است آن را به‌طور قابل‌توجهی کندتر کند، بنابراین گاهی اوقات برای عملکرد _کلی_ بهتر است که به یک عملیات اجازه دهید برای مدت کوتاهی مسدود شود. همیشه باید عملکرد کد خود را اندازه‌گیری کنید تا ببینید تنگناهای واقعی آن کجاست. دینامیک اساسی نکته‌ای است که باید به خاطر بسپارید اگر _واقعاً_ شاهد انجام مقدار زیادی از کار به‌صورت متوالی هستید که انتظار داشتید به‌طور همزمان انجام شود!

```markdown
### ساخت انتزاعات Async خودمان

ما همچنین می‌توانیم آینده‌ها را با هم ترکیب کنیم تا الگوهای جدیدی ایجاد کنیم. برای مثال، می‌توانیم با استفاده از بلوک‌های سازنده async که از قبل داریم، یک تابع `timeout` بسازیم. وقتی کارمان تمام شد، نتیجه یک بلوک سازنده دیگر خواهد بود که می‌توانیم از آن برای ساخت انتزاعات async بیشتری استفاده کنیم.

فهرست 17-27 نشان می‌دهد که چگونه انتظار داریم این `timeout` با یک آینده کند کار کند.

<Listing number="17-27" caption="تعریف نحوه کار `timeout` با یک آینده کند" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-27/src/main.rs:here}}
```

</Listing>

```markdown
بیایید این را پیاده‌سازی کنیم! برای شروع، بیایید به API مورد نیاز برای `timeout` فکر کنیم:

- باید خودش یک تابع async باشد تا بتوانیم منتظر آن بمانیم.
- پارامتر اول آن باید یک آینده برای اجرا باشد. می‌توانیم آن را عمومی کنیم تا بتواند با هر آینده‌ای کار کند.
- پارامتر دوم آن مدت‌زمان حداکثری برای انتظار خواهد بود. اگر از یک `Duration` استفاده کنیم، این کار ارسال آن به `trpl::sleep` را آسان می‌کند.
- باید یک `Result` بازگرداند. اگر آینده با موفقیت کامل شود، `Result` شامل `Ok` با مقدار تولیدشده توسط آینده خواهد بود. اگر زمان محدودیت زودتر سپری شود، `Result` شامل `Err` با مدت‌زمانی که زمان محدودیت برای آن منتظر ماند خواهد بود.

فهرست 17-28 این اعلان را نشان می‌دهد.

<!-- This is not tested because it intentionally does not compile. -->

<Listing number="17-28" caption="تعریف امضای `timeout`" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-28/src/main.rs:declaration}}
```

</Listing>

این اهداف ما برای نوع‌ها را برآورده می‌کند. حالا بیایید به _رفتاری_ که نیاز داریم فکر کنیم: می‌خواهیم آینده ارسال‌شده به آن را در برابر مدت‌زمان محدودیت مسابقه دهیم. می‌توانیم از `trpl::sleep` برای ساختن یک آینده تایمر از مدت‌زمان استفاده کنیم و از `trpl::race` برای اجرای آن تایمر با آینده‌ای که کاربر ارسال می‌کند استفاده کنیم.

همچنین می‌دانیم که `race` منصفانه نیست و آرگومان‌ها را به ترتیبی که ارسال شده‌اند بررسی می‌کند. بنابراین، ابتدا `future_to_try` را به `race` ارسال می‌کنیم تا حتی اگر `max_time` یک مدت‌زمان بسیار کوتاه باشد، فرصتی برای تکمیل داشته باشد. اگر `future_to_try` اول کامل شود، `race` مقدار `Left` با خروجی از `future` بازمی‌گرداند. اگر `timer` اول کامل شود، `race` مقدار `Right` با خروجی تایمر از نوع `()` بازمی‌گرداند.

در فهرست 17-29، نتیجه انتظار برای `trpl::race` را با `match` مدیریت می‌کنیم. اگر `future_to_try` موفق شد و مقدار `Left(output)` دریافت کردیم، مقدار `Ok(output)` را بازمی‌گردانیم. اگر تایمر خواب به جای آن سپری شد و مقدار `Right(())` دریافت کردیم، `()` را با `_` نادیده می‌گیریم و مقدار `Err(max_time)` را بازمی‌گردانیم.

<Listing number="17-29" caption="تعریف `timeout` با استفاده از `race` و `sleep`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-29/src/main.rs:implementation}}
```

</Listing>

با این کار، یک `timeout` کاربردی داریم که از دو ابزار کمکی async دیگر ساخته شده است. اگر کد خود را اجرا کنیم، حالت شکست پس از زمان محدودیت را چاپ می‌کند:

```text
Failed after 2 seconds
```

از آنجا که آینده‌ها با آینده‌های دیگر ترکیب می‌شوند، می‌توانید با استفاده از بلوک‌های سازنده async کوچک‌تر ابزارهای بسیار قدرتمندی بسازید. برای مثال، می‌توانید از همین روش برای ترکیب زمان‌های محدودیت با تلاش‌های مجدد استفاده کنید، و سپس از آن‌ها با چیزهایی مانند تماس‌های شبکه‌ای—یکی از مثال‌های ابتدای فصل—استفاده کنید!

در عمل، معمولاً مستقیماً با `async` و `await` کار خواهید کرد و به‌طور ثانویه با توابع و ماکروهایی مانند `join`، `join_all`، `race` و غیره. شما فقط گاهی نیاز خواهید داشت از `pin` استفاده کنید تا آن‌ها را با این API‌ها استفاده کنید.

ما اکنون چندین روش برای کار با آینده‌های متعدد به‌طور همزمان دیده‌ایم. در مرحله بعدی، به این خواهیم پرداخت که چگونه می‌توانیم با آینده‌های متعدد در یک توالی زمانی کار کنیم، با _جریان‌ها (streams)_. با این حال، در اینجا چند نکته دیگر برای بررسی وجود دارد:

- ما از یک `Vec` با `join_all` استفاده کردیم تا منتظر تکمیل همه آینده‌ها در یک گروه باشیم. چگونه می‌توانید از یک `Vec` برای پردازش یک گروه از آینده‌ها به‌صورت متوالی استفاده کنید؟ مزایا و معایب انجام این کار چیست؟

- نگاهی به نوع `futures::stream::FuturesUnordered` از crate `futures` بیندازید. چگونه استفاده از آن با استفاده از یک `Vec` متفاوت خواهد بود؟ (نگران این نباشید که این نوع از بخش `stream` crate است؛ این نوع با هر مجموعه‌ای از آینده‌ها به‌خوبی کار می‌کند.)

[dyn]: ch12-03-improving-error-handling-and-modularity.html
[async-program]: ch17-01-futures-and-syntax.html#our-first-async-program
