## Streams

تا اینجای این فصل، بیشتر به آینده‌های فردی پایبند بوده‌ایم. یک استثنای بزرگ کانال async بود که استفاده کردیم. به یاد بیاورید چگونه از گیرنده برای کانال async خود در بخش [«ارسال پیام»][17-02-messages]<!-- ignore --> اوایل این فصل استفاده کردیم. متد async `recv` یک دنباله‌ای از آیتم‌ها را در طول زمان تولید می‌کند. این نمونه‌ای از یک الگوی بسیار عمومی‌تر است که اغلب به آن _stream_ گفته می‌شود.

یک دنباله از آیتم‌ها چیزی است که قبلاً دیده‌ایم، زمانی که به ویژگی `Iterator` در فصل 13 نگاه کردیم. با این حال، دو تفاوت بین iteratorها و گیرنده کانال async وجود دارد. اولین مورد عنصر زمان است: iteratorها همگام هستند، در حالی که گیرنده کانال ناهمگام است. دومین مورد API است. هنگام کار مستقیم با یک `Iterator`، متد همگام `next` آن را فراخوانی می‌کنیم. به‌طور خاص با stream `trpl::Receiver`، به جای آن یک متد ناهمگام `recv` فراخوانی کردیم. این API‌ها در غیر این صورت بسیار شبیه به نظر می‌رسند.

این شباهت تصادفی نیست. یک stream شبیه به یک فرم ناهمگام از iteration است. با این حال، در حالی که `trpl::Receiver` به‌طور خاص منتظر دریافت پیام‌ها است، API stream عمومی بسیار عمومی‌تر است: آیتم بعدی را به روشی که `Iterator` انجام می‌دهد، اما به‌صورت ناهمگام ارائه می‌دهد. شباهت بین iteratorها و stream‌ها در راست به این معنی است که ما می‌توانیم در واقع از هر iterator یک stream ایجاد کنیم. مانند یک iterator، می‌توانیم با فراخوانی متد `next` روی یک stream کار کنیم و سپس خروجی را منتظر بمانیم، همان‌طور که در فهرست 17-30 نشان داده شده است.

<Listing number="17-30" caption="ایجاد یک stream از یک iterator و چاپ مقادیر آن" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-30/src/main.rs:stream}}
```

</Listing>

ما با یک آرایه از اعداد شروع می‌کنیم، که آن را به یک iterator تبدیل می‌کنیم و سپس `map` را برای دو برابر کردن تمام مقادیر فراخوانی می‌کنیم. سپس iterator را با استفاده از تابع `trpl::stream_from_iter` به یک stream تبدیل می‌کنیم. سپس با استفاده از حلقه `while let` روی آیتم‌های موجود در stream که می‌رسند پیمایش می‌کنیم.

متأسفانه، وقتی سعی می‌کنیم کد را اجرا کنیم، کامپایل نمی‌شود. در عوض، همان‌طور که در خروجی می‌بینیم، گزارش می‌دهد که متد `next` در دسترس نیست.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-30
cargo build
copy only the error output
-->

```console
error[E0599]: no method named `next` found for struct `Iter` in the current scope
  --> src/main.rs:10:40
   |
10 |         while let Some(value) = stream.next().await {
   |                                        ^^^^
   |
   = note: the full type name has been written to 'file:///projects/async_await/target/debug/deps/async_await-9de943556a6001b8.long-type-1281356139287206597.txt'
   = note: consider using `--verbose` to print the full type name to the console
   = help: items from traits can only be used if the trait is in scope
help: the following traits which provide `next` are implemented but not in scope; perhaps you want to import one of them
   |
1  + use crate::trpl::StreamExt;
   |
1  + use futures_util::stream::stream::StreamExt;
   |
1  + use std::iter::Iterator;
   |
1  + use std::str::pattern::Searcher;
   |
help: there is a method `try_next` with a similar name
   |
10 |         while let Some(value) = stream.try_next().await {
   |                                        ~~~~~~~~
```

همان‌طور که خروجی نشان می‌دهد، دلیل خطای کامپایل این است که برای استفاده از متد `next` به ویژگی مناسبی در دامنه نیاز داریم. با توجه به بحث‌های قبلی، ممکن است منطقی باشد که انتظار داشته باشید این ویژگی `Stream` باشد، اما ویژگی‌ای که اینجا نیاز داریم در واقع `StreamExt` است. `Ext` مخفف "extension" است: این یک الگوی رایج در جامعه راست برای گسترش یک ویژگی با ویژگی دیگر است.

چرا به `StreamExt` به جای `Stream` نیاز داریم و ویژگی `Stream` خودش چه کار می‌کند؟ به طور خلاصه، پاسخ این است که در سراسر اکوسیستم راست، ویژگی `Stream` یک رابط سطح پایین تعریف می‌کند که به‌طور مؤثر ویژگی‌های `Iterator` و `Future` را ترکیب می‌کند. ویژگی `StreamExt` یک مجموعه API سطح بالاتر روی `Stream` فراهم می‌کند، شامل متد `next` و همچنین متدهای کاربردی دیگر مشابه آنچه توسط ویژگی `Iterator` ارائه می‌شود. در انتهای فصل به جزئیات بیشتری درباره ویژگی‌های `Stream` و `StreamExt` خواهیم پرداخت. فعلاً این توضیحات برای ادامه کار کافی است.

برای رفع خطای کامپایل، باید یک دستور `use` برای `trpl::StreamExt` اضافه کنیم، همان‌طور که در فهرست 17-31 آمده است.

<Listing number="17-31" caption="استفاده موفق از یک iterator به‌عنوان پایه‌ای برای یک stream" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-31/src/main.rs:all}}
```

</Listing>

با قرار دادن همه این قطعات در کنار هم، این کد به همان روشی که می‌خواهیم کار می‌کند! مهم‌تر از همه، اکنون که `StreamExt` در دامنه داریم، می‌توانیم از تمام متدهای کاربردی آن استفاده کنیم، درست مانند iteratorها. برای مثال، در فهرست 17-32، از متد `filter` برای فیلتر کردن همه چیز به جز مضرب‌های سه و پنج استفاده می‌کنیم.

<Listing number="17-32" caption="فیلتر کردن یک `Stream` با استفاده از متد `StreamExt::filter`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-32/src/main.rs:all}}
```

</Listing>

البته، این خیلی جذاب نیست. می‌توانستیم این کار را با iteratorهای معمولی و بدون نیاز به async انجام دهیم. پس بیایید به برخی از کارهای دیگری که منحصراً برای streams ممکن است نگاهی بیندازیم.

### Composing Streams

بسیاری از مفاهیم به‌طور طبیعی به‌صورت streams نمایش داده می‌شوند: آیتم‌هایی که در یک صف در دسترس قرار می‌گیرند، یا کار کردن با داده‌هایی که نمی‌توانند در حافظه کامپیوتر جا شوند با کشیدن بخش‌هایی از آن‌ها از سیستم فایل در هر بار، یا داده‌هایی که به مرور زمان از طریق شبکه می‌رسند. از آنجا که streams آینده‌ها هستند، می‌توانیم از آن‌ها با هر نوع آینده دیگری استفاده کنیم و می‌توانیم آن‌ها را به روش‌های جالبی ترکیب کنیم. برای مثال، می‌توانیم رویدادها را به صورت دسته‌ای جمع کنیم تا از ایجاد تماس‌های بیش از حد شبکه جلوگیری کنیم، زمان محدودیت برای توالی عملیات طولانی‌مدت تعیین کنیم، یا رویدادهای رابط کاربری را به‌منظور جلوگیری از کارهای بیهوده محدود کنیم.

بیایید با ساخت یک stream کوچک از پیام‌ها شروع کنیم، به‌عنوان یک جایگزین برای یک stream داده‌ای که ممکن است از یک WebSocket یا یک پروتکل ارتباطی بلادرنگ دیگر ببینیم. در فهرست 17-33، یک تابع به نام `get_messages` ایجاد می‌کنیم که `impl Stream<Item = String>` را بازمی‌گرداند. برای پیاده‌سازی آن، یک کانال async ایجاد می‌کنیم، روی اولین ده حرف الفبای انگلیسی حلقه می‌زنیم و آن‌ها را از کانال ارسال می‌کنیم.

ما همچنین از یک نوع جدید استفاده می‌کنیم: `ReceiverStream`، که گیرنده `rx` از `trpl::channel` را به یک `Stream` با یک متد `next` تبدیل می‌کند. در `main`، از یک حلقه `while let` برای چاپ همه پیام‌ها از stream استفاده می‌کنیم.

<Listing number="17-33" caption="استفاده از گیرنده `rx` به‌عنوان یک `ReceiverStream`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-33/src/main.rs:all}}
```

</Listing>

وقتی این کد را اجرا می‌کنیم، دقیقاً نتایجی را که انتظار داریم دریافت می‌کنیم:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Message: 'a'
Message: 'b'
Message: 'c'
Message: 'd'
Message: 'e'
Message: 'f'
Message: 'g'
Message: 'h'
Message: 'i'
Message: 'j'
```

ما می‌توانستیم این کار را با استفاده از API معمولی `Receiver` یا حتی API معمولی `Iterator` انجام دهیم. حالا بیایید چیزی اضافه کنیم که نیاز به استفاده از streams داشته باشد: اضافه کردن یک زمان محدود که برای هر آیتم در stream اعمال می‌شود و همچنین یک تأخیر برای آیتم‌هایی که ارسال می‌کنیم.

در فهرست 17-34، ابتدا با اضافه کردن زمان محدود به stream با متد `timeout` که از ویژگی `StreamExt` می‌آید شروع می‌کنیم. سپس بدنه حلقه `while let` را به‌روزرسانی می‌کنیم، زیرا stream اکنون یک `Result` بازمی‌گرداند. نوع `Ok` نشان می‌دهد که پیام به‌موقع رسیده است؛ نوع `Err` نشان می‌دهد که زمان محدود قبل از رسیدن هر پیام سپری شده است. ما روی آن نتیجه `match` می‌کنیم و یا پیام را وقتی با موفقیت دریافت می‌کنیم چاپ می‌کنیم، یا یک اعلان درباره زمان محدود چاپ می‌کنیم. در نهایت، توجه کنید که پس از اعمال زمان محدود به پیام‌ها، آن‌ها را pin می‌کنیم، زیرا کمک‌کننده زمان محدود یک stream تولید می‌کند که برای بررسی نیاز به pin شدن دارد.

<Listing number="17-34" caption="استفاده از متد `StreamExt::timeout` برای تعیین یک محدودیت زمانی برای آیتم‌های موجود در یک stream" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-34/src/main.rs:timeout}}
```

</Listing>

با این حال، به دلیل اینکه بین پیام‌ها هیچ تأخیری وجود ندارد، این زمان محدود رفتار برنامه را تغییر نمی‌دهد. بیایید یک تأخیر متغیر به پیام‌هایی که ارسال می‌کنیم اضافه کنیم. در `get_messages`، از متد iterator `enumerate` با آرایه `messages` استفاده می‌کنیم تا بتوانیم شاخص هر آیتمی که همراه با خود آیتم ارسال می‌کنیم دریافت کنیم. سپس یک تأخیر 100 میلی‌ثانیه برای آیتم‌های با شاخص زوج و یک تأخیر 300 میلی‌ثانیه برای آیتم‌های با شاخص فرد اعمال می‌کنیم تا تأخیرهای مختلفی که ممکن است از یک stream پیام در دنیای واقعی ببینیم را شبیه‌سازی کنیم. از آنجا که زمان محدود ما برای 200 میلی‌ثانیه است، این باید نیمی از پیام‌ها را تحت تأثیر قرار دهد.

<Listing number="17-35" caption="ارسال پیام‌ها از طریق `tx` با یک تأخیر async بدون تبدیل `get_messages` به یک تابع async" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-35/src/main.rs:messages}}
```

</Listing>

برای خوابیدن بین پیام‌ها در تابع `get_messages` بدون مسدود کردن، باید از async استفاده کنیم. با این حال، نمی‌توانیم خود `get_messages` را به یک تابع async تبدیل کنیم، زیرا در این صورت یک `Future<Output = Stream<Item = String>>` به جای یک `Stream<Item = String>` بازمی‌گرداند. کاربر باید خود `get_messages` را منتظر بماند تا به stream دسترسی پیدا کند. اما به یاد داشته باشید: هر چیزی در یک آینده مشخص به‌صورت خطی اتفاق می‌افتد؛ همزمانی _بین_ آینده‌ها اتفاق می‌افتد. انتظار برای `get_messages` نیاز دارد که تمام پیام‌ها را ارسال کند، از جمله خوابیدن بین ارسال هر پیام، قبل از بازگرداندن stream گیرنده. در نتیجه، زمان محدود بی‌فایده می‌شود. هیچ تأخیری در خود stream وجود نخواهد داشت: تمام تأخیرها قبل از در دسترس قرار گرفتن stream اتفاق می‌افتد.

در عوض، `get_messages` را به‌عنوان یک تابع معمولی که یک stream بازمی‌گرداند باقی می‌گذاریم و یک تسک برای مدیریت فراخوانی‌های async `sleep` ایجاد می‌کنیم.

> نکته: فراخوانی `spawn_task` به این روش کار می‌کند زیرا ما از قبل runtime خود را تنظیم کرده‌ایم. فراخوانی این پیاده‌سازی خاص از `spawn_task` _بدون_ تنظیم اولیه یک runtime باعث panic می‌شود. پیاده‌سازی‌های دیگر معاملات متفاوتی انتخاب می‌کنند: ممکن است یک runtime جدید ایجاد کنند و بنابراین از panic اجتناب کنند، اما با کمی سربار اضافی مواجه شوند، یا به سادگی راهی مستقل برای ایجاد تسک‌ها بدون ارجاع به یک runtime ارائه ندهند. باید مطمئن شوید که می‌دانید runtime شما چه معامله‌ای انتخاب کرده است و کد خود را بر این اساس بنویسید!

اکنون کد ما نتیجه بسیار جالب‌تری دارد! بین هر جفت پیام، یک خطا گزارش می‌شود: `Problem: Elapsed(())`.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-35
cargo run
copy only the program output, *not* the compiler output
-->

```text
Message: 'a'
Problem: Elapsed(())
Message: 'b'
Message: 'c'
Problem: Elapsed(())
Message: 'd'
Message: 'e'
Problem: Elapsed(())
Message: 'f'
Message: 'g'
Problem: Elapsed(())
Message: 'h'
Message: 'i'
Problem: Elapsed(())
Message: 'j'
```

زمان محدود مانع از رسیدن پیام‌ها در نهایت نمی‌شود—ما همچنان همه پیام‌های اصلی را دریافت می‌کنیم. این به این دلیل است که کانال ما بدون حد است: می‌تواند به‌اندازه‌ای که حافظه داریم پیام نگه دارد. اگر پیام قبل از زمان محدود نرسد، handler stream ما این موضوع را مدیریت می‌کند، اما وقتی دوباره stream را بررسی می‌کند، ممکن است پیام اکنون رسیده باشد.

می‌توانید رفتار متفاوتی را در صورت نیاز با استفاده از انواع دیگر کانال‌ها یا انواع دیگر streams به‌طور کلی دریافت کنید. بیایید یکی از این‌ها را در عمل در مثال نهایی این بخش ببینیم، با ترکیب یک stream از بازه‌های زمانی با این stream پیام‌ها.

### Merging Streams

ابتدا، بیایید یک stream دیگر ایجاد کنیم که اگر مستقیماً اجرا شود، هر میلی‌ثانیه یک آیتم ارسال می‌کند. برای سادگی، می‌توانیم از تابع `sleep` برای ارسال یک پیام با تأخیر استفاده کنیم و آن را با همان روش ایجاد یک stream از یک کانال که در `get_messages` استفاده کردیم ترکیب کنیم. تفاوت این است که این بار قصد داریم تعداد بازه‌های زمانی سپری‌شده را بازگردانیم، بنابراین نوع بازگشتی `impl Stream<Item = u32>` خواهد بود، و می‌توانیم این تابع را `get_intervals` بنامیم.

در فهرست 17-36، با تعریف یک `count` در تسک شروع می‌کنیم. (ما می‌توانستیم آن را خارج از تسک تعریف کنیم، اما محدود کردن دامنه هر متغیر معین واضح‌تر است.) سپس یک حلقه بی‌نهایت ایجاد می‌کنیم. هر تکرار حلقه به‌صورت ناهمگام به مدت یک میلی‌ثانیه می‌خوابد، تعداد را افزایش می‌دهد، و سپس آن را از طریق کانال ارسال می‌کند. از آنجا که همه این‌ها در تسکی که توسط `spawn_task` ایجاد شده است پیچیده شده، همه آن همراه با runtime پاک‌سازی می‌شود، از جمله حلقه بی‌نهایت.

<Listing number="17-36" caption="ایجاد یک stream با یک شمارنده که هر میلی‌ثانیه یک بار ارسال می‌شود" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-36/src/main.rs:intervals}}
```

</Listing>

این نوع حلقه بی‌نهایت که فقط زمانی پایان می‌یابد که کل runtime متوقف شود، در async راست نسبتاً رایج است: بسیاری از برنامه‌ها باید به‌طور نامحدود اجرا شوند. با async، این کار چیزی دیگری را مسدود نمی‌کند، تا زمانی که حداقل یک نقطه انتظار در هر تکرار حلقه وجود داشته باشد.

در بلوک async تابع اصلی خود، با فراخوانی `get_intervals` شروع می‌کنیم. سپس streams `messages` و `intervals` را با متد `merge` ترکیب می‌کنیم، که چندین stream را در یک stream ترکیب می‌کند که آیتم‌ها را از هر یک از streams منبع به محض اینکه آیتم‌ها در دسترس باشند تولید می‌کند، بدون تحمیل ترتیب خاصی. در نهایت، روی آن stream ترکیبی به‌جای `messages` حلقه می‌زنیم (فهرست 17-37).

<Listing number="17-37" caption="تلاش برای ترکیب streams پیام‌ها و بازه‌های زمانی" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-37/src/main.rs:main}}
```

</Listing>

در این مرحله، نه `messages` و نه `intervals` نیاز به pin یا mutable بودن ندارند، زیرا هر دو به stream واحد `merged` ترکیب خواهند شد. با این حال، این فراخوانی به `merge` کامپایل نمی‌شود! (فراخوانی `next` در حلقه `while let` نیز کامپایل نمی‌شود، اما پس از رفع این مشکل به آن بازمی‌گردیم.) این دو stream نوع‌های متفاوتی دارند. stream `messages` نوع `Timeout<impl Stream<Item = String>>` دارد، که در آن `Timeout` نوعی است که ویژگی `Stream` را برای یک فراخوانی `timeout` پیاده‌سازی می‌کند. در همین حال، stream `intervals` نوع `impl Stream<Item = u32>` دارد. برای ترکیب این دو stream، باید یکی از آن‌ها را برای مطابقت با دیگری تبدیل کنیم.

در فهرست 17-38، stream `intervals` را بازنویسی می‌کنیم، زیرا `messages` از قبل در قالب پایه‌ای که می‌خواهیم است و باید خطاهای زمان محدود را مدیریت کند. اول، می‌توانیم از متد کمکی `map` برای تبدیل `intervals` به یک رشته استفاده کنیم. دوم، باید با `Timeout` از `messages` مطابقت داشته باشیم. با این حال، چون در واقع نمی‌خواهیم برای `intervals` زمان محدود داشته باشیم، می‌توانیم به سادگی یک زمان محدود طولانی‌تر از دیگر مدت‌زمان‌هایی که استفاده می‌کنیم ایجاد کنیم. اینجا، یک زمان محدود 10 ثانیه‌ای با `Duration::from_secs(10)` ایجاد می‌کنیم. در نهایت، باید `stream` را متغیر کنیم تا فراخوانی‌های `next` حلقه `while let` بتوانند در stream پیمایش کنند و آن را pin کنیم تا انجام این کار ایمن باشد.

<!-- We cannot directly test this one, because it never stops. -->

<Listing number="17-38" caption="هماهنگ کردن نوع‌های stream `intervals` با نوع stream `messages`" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-38/src/main.rs:main}}
```

</Listing>

این کد ما را _تقریباً_ به جایی که باید باشیم می‌رساند. همه چیز از نظر نوع بررسی می‌شود. با این حال، اگر این کد را اجرا کنید، دو مشکل وجود خواهد داشت. اول، هرگز متوقف نمی‌شود! باید آن را با <span class="keystroke">ctrl-c</span> متوقف کنید. دوم، پیام‌های مربوط به حروف الفبای انگلیسی در میان تمام پیام‌های شمارنده بازه‌های زمانی دفن خواهند شد:


<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the tasks running differently rather than
changes in the compiler -->

```text
--snip--
Interval: 38
Interval: 39
Interval: 40
Message: 'a'
Interval: 41
Interval: 42
Interval: 43
--snip--
```

فهرست 17-39 یک روش برای حل این دو مشکل نهایی را نشان می‌دهد. اول، از متد `throttle` روی stream `intervals` استفاده می‌کنیم تا این stream بر stream `messages` غلبه نکند. Throttling روشی برای محدود کردن نرخ فراخوانی یک تابع است—یا در این مورد، اینکه چند بار stream بررسی (polled) می‌شود. یک بار هر صد میلی‌ثانیه کافی است، زیرا این تقریباً با زمانی که پیام‌های ما می‌رسند مطابقت دارد.

برای محدود کردن تعداد آیتم‌هایی که از یک stream می‌پذیریم، می‌توانیم از متد `take` استفاده کنیم. این متد را روی stream _merged_ اعمال می‌کنیم، زیرا می‌خواهیم خروجی نهایی را محدود کنیم، نه فقط یک stream یا دیگری.

<Listing number="17-39" caption="استفاده از `throttle` و `take` برای مدیریت streams ترکیب‌شده" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-39/src/main.rs:throttle}}
```

</Listing>

اکنون وقتی برنامه را اجرا می‌کنیم، پس از دریافت بیست آیتم از stream متوقف می‌شود، و بازه‌ها (intervals) بر پیام‌ها غلبه نمی‌کنند. همچنین به جای `Interval: 100` یا `Interval: 200` و غیره، مقادیری مانند `Interval: 1`، `Interval: 2` و غیره دریافت می‌کنیم—حتی اگر یک stream منبع داشته باشیم که _می‌تواند_ هر میلی‌ثانیه یک رویداد تولید کند. این به این دلیل است که فراخوانی `throttle` یک stream جدید تولید می‌کند که stream اصلی را بسته‌بندی می‌کند، به‌طوری که stream اصلی فقط با نرخ throttled بررسی می‌شود، نه با نرخ "ذاتی" خودش. ما مجموعه‌ای از پیام‌های بازه‌ای بدون پردازش که تصمیم به نادیده گرفتن آن‌ها گرفته‌ایم نداریم. در عوض، اصلاً این پیام‌های بازه‌ای تولید نمی‌شوند! این همان "تنبلی" ذاتی آینده‌های راست است که دوباره در کار است و به ما اجازه می‌دهد تا ویژگی‌های عملکرد خود را انتخاب کنیم.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-39
cargo run
copy and paste only the program output
-->

```text
Interval: 1
Message: 'a'
Interval: 2
Interval: 3
Problem: Elapsed(())
Interval: 4
Message: 'b'
Interval: 5
Message: 'c'
Interval: 6
Interval: 7
Problem: Elapsed(())
Interval: 8
Message: 'd'
Interval: 9
Message: 'e'
Interval: 10
Interval: 11
Problem: Elapsed(())
Interval: 12
```

یک مورد نهایی که باید مدیریت کنیم: خطاها! با هر دو stream مبتنی بر کانال، فراخوانی‌های `send` ممکن است زمانی که طرف دیگر کانال بسته می‌شود شکست بخورند—و این فقط به نحوه اجرای runtime آینده‌هایی که stream را تشکیل می‌دهند مربوط می‌شود. تا به حال، با فراخوانی `unwrap` این موضوع را نادیده گرفته‌ایم، اما در یک برنامه خوب، باید به‌صراحت خطا را مدیریت کنیم، حداقل با خاتمه دادن به حلقه تا دیگر سعی در ارسال پیام‌های بیشتر نکنیم! فهرست 17-40 یک استراتژی ساده برای خطاها را نشان می‌دهد: مشکل را چاپ کرده و سپس با `break` از حلقه خارج شوید. همان‌طور که معمولاً، روش صحیح مدیریت یک خطای ارسال پیام متفاوت خواهد بود—فقط مطمئن شوید که یک استراتژی دارید.

<Listing number="17-40" caption="مدیریت خطاها و خاتمه دادن به حلقه‌ها">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-40/src/main.rs:errors}}
```

</Listing>

اکنون که مقدار زیادی از async را در عمل دیده‌ایم، بیایید کمی عقب‌تر برویم و به برخی از جزئیات درباره نحوه کار `Future`، `Stream`، و ویژگی‌های کلیدی دیگر که راست برای کار async استفاده می‌کند، بپردازیم.

[17-02-messages]: ch17-02-concurrency-with-async.html#message-passing
