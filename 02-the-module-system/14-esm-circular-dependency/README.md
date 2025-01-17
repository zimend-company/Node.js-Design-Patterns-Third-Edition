### Circular dependency resolution

<p dir="rtl" align="right">
مثال وابستکی های حلقه ای که با commonJS نوشتیم رو اینجا دوباره بررسی میکنیم:
</p>

```
// a.js
import * as bModule from './b.js'
export let loaded = false
export const b = bModule
loaded = true
// b.js
import * as aModule from './a.js'
export let loaded = false
export const a = aModule
loaded = true
```
<p dir="rtl" align="right">
ایمپورت این دو ماژول:
</p>

```
// main.js
import * as a from './a.js'
import * as b from './b.js'
console.log('a ->', a)
console.log('b ->', b)
```

<p dir="rtl" align="right">
دقت کنید در اینجا ما از JSON.stringify استفاده نکردیم چون خطای TypeError: Converting circular structure to JSON مس دهد. دلیلش: چون در ES این یک وابستگی دایره ای واقعی بین a.js و b.js است
</p>

<p dir="rtl" align="right">
خروجی:
</p>

```
a -> <ref *1> [Module] {
  b: [Module] { a: [Circular *1], loaded: true },
  loaded: true
}
b -> <ref *1> [Module] {
  a: [Module] { b: [Circular *1], loaded: true },
  loaded: true
}
```

<p dir="rtl" align="right">
در ES ماژول های a.js و b.js یک تصویر کامل از هم دارند. برخلاف commonJS که اطلاعات جزیی از هم داشتند. و می بینیم که همه loaded روی درست تنظیم شده است. همچنین b  درون a یک مرجع واقعی به همان نمونه b موجود در محدوده ی فعلی است. (و برعکس) به همین دلیل از JSON.stringify() برای سریال سازی این ماژول ها نیم توان استفاده کرد. اگر ترتیب واردات برای a.js و b.js را تغییر دهیم در خروجی هیچ تغییری نمیکند. (برخلاف commonjs)
</p>

<p dir="rtl" align="right">
فاز یک: تجزیه
</p>

  <p dir="rtl" align="right">
کد از نقطه ورود (main.js) بررسی می شود. مفسر فقط به دنبال دستورات import برای یافتن همه ماژول های لازم و بارگذاری سورس کد از فایل های ماژول است. نمودار وابستگی به طور عمیق بررسی می شود و هر ماژول فقط یکبار بازدید می شود. پس نمایی از وابستگی ها شبیه درخت ایجاد میکند. 
</p>

![image](https://user-images.githubusercontent.com/45192069/133125766-cba97fa9-d056-4413-a7c4-e46da46a80d1.png)

 <p dir="rtl" align="right">
مراحل مختلف فاز اول:
 </p>
 
 
 <p dir="rtl" align="right">
 ۱. از main.js اولین import ما را به a.js می برد
 </p>
 
 <p dir="rtl" align="right">
 ۲. در a.js ایمپورتی را مشاهده که به b.js اشاره دارد.
 </p>
 
 <p dir="rtl" align="right">
 ۳. در b.js ما همچنین ایمپورت a.js را داریم. اما چون a.js قبل بازدید شده است دوباره بررسی نمی شود
 </p>
 
 <p dir="rtl" align="right">
 ۴. در این مرحله اکتشاف به عقب برمیگردد. b.js ایمپورت دیگری ندارد بنابراین به a.js بارمیگردیم. که اینم ایمپورت دیگری ندارد پس به main.js برمی گردیم. که در اینجا ایمپورت دیگری را مشاهده که به b.js اشاره دارد ولی چون قبلا این ماژول بررسی شده نادیده گرفته می شود. 
  </p>
  
  
  
 <p dir="rtl" align="right"> 
 فاز دوم: پیاده سازی
</p>

![image](https://user-images.githubusercontent.com/45192069/133127830-f9250bbf-24a5-4e15-93c4-0825e268577c.png)

 <p dir="rtl" align="right">
مفسر نمای درخت مرحله قبل را از پایین به بالا پیاده میکند. برای هرماژول، ابتدا تمام ویژگی های ایمپورت شده را جستجو کرده و نقشه ای از نام های ایمپورت شده در حافظه ایجاد میکند
</p>

<p dir="rtl" align="right">
۱. مترجم از b.js شروع می کند و متوجه می شود که این ماژول، loaded و a را اکسپورت می کند
</p>

<p dir="rtl" align="right">
۲. سپس به a.js حرکت میکند. که loaded  و b را اکسپورت کرده است
</p>

<p dir="rtl" align="right">
۳. در پایان، به main.js حرکت میکند که هیچ تابعی را اکسپورت نکرده است
</p>

<p dir="rtl" align="right">
4. در این مرحله، نقشه اکسپورت فقط نام های صادرشده را دنبال میکند. و برای این نام ها هیچ مقداری در نظر نمیگیرد.
</p>

![image](https://user-images.githubusercontent.com/45192069/133127866-ce2a493b-94b4-4b6d-9e6b-9e99a51d63af.png)

<p dir="rtl" align="right">
چیزی که در تصویر بالا می بینید را می توان اینگونه شرح داد:
</p>

<p dir="rtl" align="right">
۱. ماژول b.js به تمام اکسپورت ها را از a.js لینک میکند و از ان به عنوان aModule یاد میکند
</p>

<p dir="rtl" align="right">
۲. به نوبه ی خود، a.js به تمام اکسپورت ها از b.js لینک میزند و ان انها به عنوان bModule یاد میکند
</p>

<p dir="rtl" align="right">
۳. سرانجام main.js تمام اکسپورت های a.js را ایمپورت و ازانها به عنوان a یاد میکند. و به همین صورت برای b.js انجام می دهد
</p>

<p dir="rtl" align="right">
۴. هنوز هیچ مقداری قرار نمیگیرد
</p>

<p dir="rtl" align="right">
فاز سوم: ارزیابی
</p>

<p dir="rtl" align="right">
تمام کدهای هر فایل اجرا می شود. اجرا هم از پایین به بالا می باشد پس main,js اخرین فایلی است که باید اجرا شود. پس ما مطمئن هستیم که همه مقادیر اکسپورت شده مقدار دهی شده اند. 
</p>

![image](https://user-images.githubusercontent.com/45192069/133129722-0fb0e719-5da5-4dc8-9d1f-d919a60a687f.png)


# 13-esm-circular-dependency

This sample demonstrates that ESM can effectively resolve circular dependencies.

## Run

```bash
node main.js
```
