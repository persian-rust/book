## Async and Await

Many operations we ask the computer to do can take a while to finish. For
example, if you used a video editor to create a video of a family celebration,
exporting it could take anywhere from minutes to hours. Similarly, downloading a
video shared by someone in your family might take a long time. It would be nice
if we could do something else while we are waiting for those long-running
processes to complete.

The video export will use as much CPU and GPU power as it can. If you only had
one CPU core, and your operating system never paused that export until it
completed, you couldn’t do anything else on your computer while it was running.
That would be a pretty frustrating experience, though. Instead, your computer’s
operating system can—and does!—invisibly interrupt the export often enough to
let you get other work done along the way.

The file download is different. It does not take up very much CPU time. Instead,
the CPU needs to wait on data to arrive from the network. While you can start
reading the data once some of it is present, it might take a while for the rest
to show up. Even once the data is all present, a video can be quite large, so it
might take some time to load it all. Maybe it only takes a second or two—but
that’s a very long time for a modern processor, which can do billions of
operations every second. It would be nice to be able to put the CPU to use for
other work while waiting for the network call to finish—so, again, your
operating system will invisibly interrupt your program so other things can
happen while the network operation is still ongoing.

> Note: The video export is the kind of operation which is often described as
> “CPU-bound” or “compute-bound”. It’s limited by the speed of the computer’s
> ability to process data within the _CPU_ or _GPU_, and how much of that speed
> it can use. The video download is the kind of operation which is often
> described as “IO-bound,” because it’s limited by the speed of the computer’s
> _input and output_. It can only go as fast as the data can be sent across the
> network.

In both of these examples, the operating system’s invisible interrupts provide a
form of concurrency. That concurrency only happens at the level of a whole
program, though: the operating system interrupts one program to let other
programs get work done. In many cases, because we understand our programs at a
much more granular level than the operating system does, we can spot lots of
opportunities for concurrency that the operating system cannot see.

For example, if we’re building a tool to manage file downloads, we should be
able to write our program in such a way that starting one download does not lock
up the UI, and users should be able to start multiple downloads at the same
time. Many operating system APIs for interacting with the network are
_blocking_, though. That is, these APIs block the program’s progress until the
data that they are processing is completely ready.

> Note: This is how _most_ function calls work, if you think about it! However,
> we normally reserve the term “blocking” for function calls which interact with
> files, the network, or other resources on the computer, because those are the
> places where an individual program would benefit from the operation being
> _non_-blocking.

We could avoid blocking our main thread by spawning a dedicated thread to
download each file. However, we would eventually find that the overhead of those
threads was a problem. It would also be nicer if the call were not blocking in
the first place. Last but not least, it would be better if we could write in the
same direct style we use in blocking code. Something similar to this:

```rust,ignore,does_not_compile
let data = fetch_data_from(url).await;
println!("{data}");
```

That is exactly what Rust’s async abstraction gives us. Before we see how this
works in practice, though, we need to take a short detour into the differences
between parallelism and concurrency.

### تفاوت بین موازی‌سازی و همزمانی

در فصل قبل، موازی‌سازی و همزمانی را تقریباً به صورت قابل تعویض در نظر گرفتیم. اکنون باید آن‌ها را به طور دقیق‌تری متمایز کنیم، زیرا تفاوت‌های آن‌ها در حین کار نشان داده می‌شود.

وقتی یک فرد روی چند کار مختلف قبل از اتمام هر یک کار می‌کند، این به عنوان **همزمانی** توصیف می‌شود. شما ممکن است دو پروژه مختلف را روی کامپیوتر خود بررسی کنید، و زمانی که از یک پروژه خسته یا گیر کردید، به پروژه دیگر بروید. شما تنها یک نفر هستید، بنابراین نمی‌توانید همزمان روی هر دو کار پیشرفت کنید، اما می‌توانید با جابه‌جایی بین آن‌ها روی چند کار پیشرفت کنید.

<figure>

<img alt="Concurrent work flow" src="img/trpl17-01.svg" class="center" />

<figcaption>Figure 17-1: A concurrent workflow, switching between Task A and Task B.</figcaption>

</figure>

وقتی موافقت می‌کنید که گروهی از وظایف را بین افراد تیم تقسیم کنید، به طوری که هر فرد یک وظیفه را گرفته و به تنهایی روی آن کار کند، این به عنوان **موازی‌سازی** توصیف می‌شود. هر فرد در تیم می‌تواند به طور همزمان پیشرفت کند.

<figure>
    <img alt="جریان کار موازی" src="img/trpl17-02.svg" class="center" />
    <figcaption>شکل 17-2: یک جریان کاری موازی که در آن کار روی وظایف A و B به صورت مستقل انجام می‌شود.</figcaption>
</figure>

در هر دو این وضعیت‌ها، ممکن است نیاز به هماهنگی بین وظایف مختلف داشته باشید. شاید **فکر کرده‌اید** که وظیفه‌ای که یک نفر روی آن کار می‌کند کاملاً مستقل از کار دیگران است، اما در واقع به چیزی که توسط فرد دیگری در تیم به اتمام رسیده نیاز دارد. برخی از کارها می‌توانند به صورت موازی انجام شوند، اما برخی از آن‌ها در واقع **سریالی** هستند: آن‌ها فقط می‌توانند به ترتیب، یکی پس از دیگری انجام شوند، مانند شکل 17-3.

<figure>
    <img alt="جریان کار همزمان" src="img/trpl17-03.svg" class="center" />
    <figcaption>شکل 17-3: یک جریان کاری تا حدودی موازی که در آن کار روی وظایف A و B به صورت مستقل انجام می‌شود تا زمانی که وظیفه A3 بر اساس نتایج وظیفه B3 مسدود می‌شود.</figcaption>
</figure>

به همین ترتیب، ممکن است متوجه شوید که یکی از وظایف شما به وظیفه دیگری از کارهای شما بستگی دارد. اکنون کار همزمان شما نیز سریالی شده است.

موازی‌سازی و همزمانی می‌توانند با یکدیگر تقاطع داشته باشند. اگر متوجه شوید که یک همکار تا زمانی که یکی از وظایف شما به پایان نرسیده گیر کرده است، احتمالاً تمام تلاش خود را روی آن وظیفه متمرکز می‌کنید تا "همکارتان را از بن‌بست خارج کنید." شما و همکارتان دیگر نمی‌توانید به صورت موازی کار کنید، و همچنین دیگر نمی‌توانید به صورت همزمان روی وظایف خودتان کار کنید.

همین پویایی‌های اساسی در نرم‌افزار و سخت‌افزار نیز مطرح می‌شوند. در ماشینی با یک هسته CPU، پردازنده تنها می‌تواند یک عملیات را در یک زمان انجام دهد، اما همچنان می‌تواند به صورت همزمان کار کند. با استفاده از ابزارهایی مانند نخ‌ها، فرآیندها، و async، کامپیوتر می‌تواند یک فعالیت را متوقف کند و به فعالیت‌های دیگر سوئیچ کند و در نهایت دوباره به فعالیت اول بازگردد. در ماشینی با چندین هسته CPU، می‌تواند به صورت موازی نیز کار کند. یک هسته می‌تواند یک کار انجام دهد در حالی که هسته دیگری کاری کاملاً غیرمرتبط انجام می‌دهد، و این کارها در واقع به طور همزمان انجام می‌شوند.

در هنگام کار با async در Rust، همیشه با همزمانی سروکار داریم. بسته به سخت‌افزار، سیستم‌عامل، و محیط اجرایی async که استفاده می‌کنیم—که به زودی بیشتر درباره محیط‌های اجرایی async صحبت خواهیم کرد!—این همزمانی ممکن است در پشت صحنه از موازی‌سازی نیز استفاده کند.

حال، بیایید به نحوه عملکرد برنامه‌نویسی async در Rust بپردازیم! در ادامه این فصل:

- می‌بینیم که چگونه از نحو `async` و `await` در Rust استفاده کنیم،
- بررسی می‌کنیم که چگونه می‌توان از مدل async برای حل برخی از چالش‌هایی که در فصل 16 دیدیم استفاده کرد،
- و نگاهی می‌اندازیم به اینکه چگونه چندنخی و async راه‌حل‌های مکملی ارائه می‌دهند که حتی می‌توانید در بسیاری از موارد با هم استفاده کنید.
