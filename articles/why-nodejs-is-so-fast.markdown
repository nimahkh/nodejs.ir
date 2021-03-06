Title: چرا Node.js خیلی سریع است؟
Author: توحید ارسطو
Date: Sat 9 Jan 2016 12:00:00 PM (IRST)
Categories: node, node.js, synchronous, asynchronous, event loop, thread pool

پلتفرم Node.js امروزه بعد از سالها جای خود را در میان توسعه دهنده‌ها و شرکت‌ها باز کرده است، بطور عمده امروزه Node.js در مواردی استفاده می‌شود که نیاز است به حجم زیادی از درخواستها در زمان کم پاسخ داده شود. شرکتهایی که از Node.js در زیرساخت‌های خویش استفاده می‌کنند در زیر آمده اند.

+ [New Yorker](http://stackshare.io/new-yorker/new-yorker)
+ [Change.org](http://stackshare.io/change-org/change-org)
+ [Klout](http://stackshare.io/klout/klout)
+ [medium.com](http://stackshare.io/medium/medium-com)
+ [500px](http://stackshare.io/500px/500px)
+ [Fluxible](http://stackshare.io/yahoo/fluxible)
+ [Arabiaweather Inc.](http://stackshare.io/arabiaweather-inc-/arabiaweather-inc)
+ [Netflix](http://stackshare.io/netflix/netflix)
+ [Uber](http://stackshare.io/uber/uber)
+ [Flipboard](http://stackshare.io/flipboard/flipboard)

وجود ویژگی‌هایی در Node.js و موتور V8 باعث شده که ند پلتفرمی بسیار سریع در سناریوهایی باشد که نیاز به پردازش سریع تعداد زیادی درخواست دارند. در این مقاله به دلایل سریع بودن ند و به طور مشخص به مفهوم «زمان پاسخگویی» می‌پردازیم.
<!--more-->

## منظور از «زمان پاسخگویی» چیست؟

وقتی در مورد وب سرویس‌ها(Web Services) حرف می‌زنیم، «زمان پاسخگویی» شامل برایند همه زمانهایی است که برای پردازش یک درخواست(Request) و فرستادن پاسخ(Response) به یک Client نیاز است. تعریف من از «زمان پاسخگویی» این است: مقدار زمانی که برای پردازش یک درخواست(Request)، از زمان باز شدن ارتباط(Connection) از سوی Client تا در دریافت پاسخ(Response) آن درخواست،  صرف می شود.

به محض اینکه بفهمید هنگام پردازش یک درخواست(Request) در یک سرور Node.js چه اتفاقی می افتد، دلیل سریع بودن آن را در می‌یابید. اما قبل از اینکه در مورد Node.js صحبت کنیم، بیایید به پردازش درخواست‌ها(Requests) در دیگر زبانها(تکنولوژی ها) نگاه کنیم. به توجه به آشنایی اکثر توسعه دهنده‌ها با زبان php پلتفرم Node.js را با سکوی PHP مقایسه می‌کنیم. اکثر Benchmark ها نشان داده که Node.js در پردازش درخواستها نسبت به PHP سریعتر است. اینجا به  دلایلی که در PHP  «زمان پاسخگویی» طولانی‌تر است اشاره می‌کنیم.

##دلایل بالا بودن زمان پاسخگویی در PHP

+ ابتدا اینکه PHP یک زبان synchronous است. این یعنی وقتی که شما یک درخواست(Request) را در این زبان پردازش(Processing) می‌کنید -مثلا نوشتن در یک پایگاه داده(Database)- همه عملکردهای دیگر متوقف می‌شود تا این کار تمام شود، شما مجبورید منتظر بمانید تا این کار تمام شود و هیچ کار دیگری قابل انجام نیست.

+ هر درخواستی(Request) که به وب سرویس(Web Service) می‌فرستید در سرور یک Process(در بعضی موارد به جای یک Process وب سرویس یک Thread ایجاد می‌کند) جداگانه مفسر PHP ایجاد می کند که کد شما را اجرا کند. اگر هزار Connection داشته باشید هزار Process در حال اجرا خواهید داشت که رم(Ram) می‌خورند :).

+ سکوی PHP در حالت پیشفرض فاقد« JIT compilation» - کامپایل کد در زمان اجرا- می باشد(البته در ماشین مجازی HHVM که برای اجرای کدهای PHP و Hack ساخته شده است مفهوم JIT پیاده سازی شده است.)، کامپایل کد در زمان اجرا هنگامی که کدی دارید که به دفعات اجرا می شود خیلی مهم است، زیرا شما معمولا دوست دارید، برای بازدهی بیشتر، مطمئن شوید که این کد تقریبا از نظر زمان اجرا نزدیک به کد نوشته شده به زبان ماشین عمل کند.

حال بیاید ببینیم Node.js چگونه این مشکلات را مدیریت و حل می‌کند.

## دلایل پایین بودن زمان پاسخگویی در Node.js

+ سکوی Node.js یک پلتفرم single-threaded و asynchronous است. هیچ کدام از عملکردهای مرتبط با I/O بقیه عملکردها را متوقف نمیکند. این به معنای آن است که شما می‌توانید در یک زمان هم از روی دیسک یک فایل را بخوانید هم یک ایمیل بفرستید و هم بر روی پایگاه داده Query بزنید.

+ هر کدام از درخواست هایی(Request) که به وب سرویس(Web Service) می رسند یک Process جدید Node.js ایجاد نمی کنند، به جای آن در اغلب اوقات فقط و فقط یک Process مربوط به Node.js در حال اجرا است که به ارتباطات(Connections) و درخواست‌ها(Requests) گوش می دهد. کدهای جاوااسکریپت در Thread اصلی و عملکردهای مرتبط با I/O در Thread های دیگری اجرا می‌شوند.

+ ماشین مجازی(Google V8) در Node.js که کدهای جاوااسکریپت را اجرا می  کند دارای ویژگی کامپایل در زمان اجرا(JIT Compilation) می باشد. وقتی این ماشین مجازی کدهای جاوااسکریپت را می‌گیرد در زمان اجرا آنها را به کدهایی نزدیک به کدهای زبان ماشین کامپایل می‌کند، این کار باعث می‌شود توابعی که به دفعات صدا زده می‌شوند با تبدیل شدن به کدهای شبیه کد ماشین به طور قابل ملاحظه‌ای سرعت اجرای کدها را بهبود دهد.

حال که مزیت‌های مفهوم asynchronous در Node.js را دیدیم اجازه دهید برایتان توضیح دهم که این مفهوم در Node.js چگونه کار می‌کنند.

## در مسیر شناخت مفهوم asynchronous

مفهوم پردازش asynchronous را می خواهم در قالب مثال برایتان بیان می‌کنم.

فرض کنید در بالای یک کوه ۱۰۰۰ توپ  در اختیار شماست، شما باید همه این  توپها را به پایین کوه بیاورید(هل دهید).قاعدتا شما نمی توانید همه این توپها را در یک زمان به پایین بفرستید، شما احتمالا مجبور خواهید بود توپها را تک تک به پایین بفرستید اماخب این به معنای آن نیست که باید منتظر بمانید تا یک توپ به پایین کوه برسد تا توپ بعدی را بفرستید.

در این مثال رویکرد synchronous به این معنا است که شما باید منتظر باشید که یک توپ به پایین برسد تا توپ بعدی را بفرستید، واقعا زمان زیادی می‌برد، درسته؟

رویکرد Asynchronous در مثال بالا یعنی همه توپها را به سرعت پشت سر هم رها کنید سپس صبر کنید تا هر کدام به پایین برسند(مثل اینکه Notification دریافت کنید).

خب سوال اصلی این است که این(رویکرد Asynchronous) چگونه به بهبود کارایی یک وب سرویس یا وب سرور کمک می‌کند؟

بیایید در نظر بگیریم هر توپ یک Query در پایگاه داده است. شما یک پروژه بزرگ دارید با تعداد زیادی aggregations، Query و … وقتی شما همه چیز را با رویکرد synchronous پردازش و مدیریت می‌کنید هر کدام از این کارها باعث متوقف شدن اجرای کد می‌شود. وقتی با رویکرد asynchronous همه چیز را پردازش و مدیریت می‌کنید شما می‌توانید همه کارها را یکدفعه‌ای انجام دهید و سپس نتیجه‌ها را وصول کنید.

در دنیای واقعی، وقتی شما تعداد زیادی ارتباط(Connections) داشته باشید، این رویکرد -asynchronous- به طور قابل ملاحظه‌ای کارایی و عملکرد برنامه شما را بهبود می‌بخشد.

مفهوم asynchronous چگونه در Node.js پیاده سازی شده است؟

## Event loop

Event loop یک ساختار است که مسئول مخابره(dispatch) کردن رویدادها(Events) در یک برنامه است که تقریبا همیشه با سازنده پیام(Message) بصورت asynchronous رفتار می‌کند. وقتی شما یک عملکرد مرتبط با I/O را صدا می‌زنید Node.js یک Callback را به این عملکرد متصل می‌کند و به پردازش کد ادامه می‌دهد. وقتی همه اطلاعات لازم مرتبط با عملکردی که یک Callback به آن ملحق شد جمع آوری شد، Node.js آن Callback را اجرا می‌کند.

تعریف دقیقتر Event Loop از ویکی پدیا در زیر آماده است:

> The event loop, message dispatcher, message loop, message pump, or run loop is a programming construct that waits for and dispatches events or messages in a program. It works by making a request to some internal or external “event provider” (which generally blocks the request until an event has arrived), and then it calls the relevant event handler(“dispatches the event”). The event-loop may be used in conjunction with a reactor, if the event provider follows the file interface, which can be selected or ‘polled’ (the Unix system call, not actual polling). The event loop almost always operates asynchronously with the message originator.

بیایید به این تصویر ساده که شیوه کارکرد Event Loop را تشریح می‌کند نگاه بیاندازیم.

![NodeJS Event Loop](http://arastu.ir/nd/event-loop.jpg "NodeJS Event Loop")

وقتی یک درخواست توسط وب سرور دریافت می‌شود این درخواست وارد Event Loop می‌شود، Event Loop این عملیات را به Thread Pool تحویل می‌دهد و برای آن یک Callback مشخص می‌کند، این Callback زمانی اجرا می‌شود که درخواست پردازش و آماده گردد. این Callback می‌تواند درون خود دیگر عملیات سنگین مانند Query زدن در پایگاه داده را داشته باشد که هر کدام از این عملیات‌ها دقیقا مانند پردازش یک درخواست با آنها رفتار می‌شود. یعنی هر کدام از عملیات پیچیده تحویل Thread Pool می‌شود و …

اما اجرای کد چگونه انجام می‌شود، خوب در بخش بعدی این مقاله ما در مورد ماشین مجازی که کدهای جاوااسکریپت را اجرا می‌کند صحبت می‌کنیم. در مورد موتور V8.

## چگونه V8 کدهای شما را بهینه می‌کند که به سرعت اجرا شوند!؟

من می‌خواهم در مورد مفاهیم پایه V8 صحبت کنم و اینکه این موتو چگونه کدهای جاوااسکریپت را برای اجرا شدن بهینه سازی می‌کند، با توجه با این که این مفاهیم به شدت پیچیده و فنی هستند بنابراین می‌توانید با خیال راحت این مفاهیم را نخوانید :)، برای آشنایی بیشتر با V8 می توانید [منابع V8](http://mrale.ph/v8/resources.html) را ببینید.

موتور V8 دارای دو نوع کامپایلر است(البته در واقع سومین کامپایلر با نام Turbofan در حال توسعه است) کامپایلر «Full» و کامپایلر «Crankshaft»

کامپایلر Full بسیار سریع است و وظیفه اش تولید کد generic است. این کامپایلر ابتدا یک AST یا [Abstract Syntax Tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree) از توابع جاوااسکریپت تولید می کند و آن را به کد پایه ماشین ترجمه می کند. در این مرحله  فقط یک بهبود رخ می‌دهد این بهبود [Inline Caching](https://en.wikipedia.org/wiki/Inline_caching) نام دارد.

وقتی تابع کامپایل می‌شود و کد در حال اجراست، V8 یک Thread پیشفیلتر(Profiler) را آغاز می‌کند که بفهمد کدام تابع به اصطلاح Hot است (Hot به معنای توابعی است که زیاد صدا زده می‌شود) و کدام نیست.

وقتی V8 توابع Hot را تشخیص داد، کد AST مربوط به آن را از طریق کامپایلر Crankshaft اجرا می کند.

کامپایلر Crankshaft خیلی سریع نیست بلکه بیشتر سعی می کند کد بهبودیافته و سریع تولید کند، این کامپایلر از دو بخش تولید شده است، هیدروژن و لیتیم

هیدروژن CFG - [Control Flow Graph](https://en.wikipedia.org/wiki/Control_flow_graph) را با استفاده از AST تولید می‌کند. این گراف بصورت SSA - [Static Single Assignment](https://en.wikipedia.org/wiki/Static_single_assignment_form) نمایان می‌شود. بسته به ساختار ساده HIR - [High-Level Intermediate Representation](https://en.wikipedia.org/wiki/Intermediate_language) و فرم نمایشی SSA کامپایلر می تواند تعداد زیادی بهبود را اعمال کند، بهبودهایی مانند [constant folding](https://en.wikipedia.org/wiki/Constant_folding) و [method inlining](https://en.wikipedia.org/wiki/Inline_expansion) و ...

کامپایلر لیتیوم HIR بهبود یافه را به LIR - [Low-Level Intermediate Representation](https://en.wikipedia.org/wiki/Intermediate_language) تبدیل می‌کند. LIR اساسا بسیار به کد ماشین شبیه است، البته هنوز کاملا واببسته به پلتفرم است در مقام مقایسه با HIR ساختار LIR به [three-address code](https://en.wikipedia.org/wiki/Three-address_code) نزدیک تر است.

این روند در نهایت همیشه کد بهبود یافته را با کد کند تر جایگزین می‌کند. و این چنین به اجرای کد شما بصورت سریعتر ادامه می‌یابد.

**این مقاله با استفاده از ویرایشگر [مرتب](http://www.sobhe.ir/moratab/) نوشته شده است.**

[+منبع](https://medium.com/@ghaiklor/why-nodejs-is-so-fast-a0ff67858f48#.8ygv6r4pu)
