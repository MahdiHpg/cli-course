# فصل ۷ — جستجو: grep و find (غول‌چراغ دولوپرها) 🔍

> 🎯 **هدف:** دو ابزاری که بیشترین «وقت‌صرفه‌جویی» رو تو کار روزمره دارن: `grep` برای جستجو در *محتوای* فایل‌ها و `find` برای جستجو در *درخت* فایل‌ها. آخر فصل، کلیدهای Ctrl+F پروژه‌های چند-هزار-فایلی رو داری.

---

## ۷.۱ — grep: جستجو در متن

grep = خطوطی که الگو رو دارن، چاپ کن:

```bash
$ grep "todo" app.js                 # خطوط حاوی todo در یک فایل
$ grep -i "todo" app.js              # بدون حساسیت به بزرگ/کوچک ⭐
$ grep -n "todo" app.js              # شماره خط هم نشان بده
$ grep -r "console.log" src/         # ⭐ recursive: کل پوشه را بگرد
$ grep -c "ERROR" server.log         # فقط بشمار: چند خط؟
$ grep -v "comment" file.txt         # برعکس: خطوطی که ندارند
$ grep -A 2 -B 2 "error" log.txt     # ۲ خط قبل و بعد هم نشان بده (context)
```

خروجی `-rn` روی پروژه واقعی (پرکاربردترین شکل):

```
src/api/user.ts:14:  console.log("fetching user");
src/hooks/useAuth.ts:31:  console.log("auth state", state);
```

`فایل:خط:محتوا` — در VS Code همون خط رو Ctrl+Click کنی می‌ری سراغش. (تقریباً همون Ctrl+Shift+F خود VS Code، ولی سریع‌تر و قابل‌زنجیره با pipes.)

## ۷.۲ — grep در زنجیره: جایی که می‌درخشه

```bash
# لاگ بیلد بلند است — فقط خطاها را ببین:
$ npm run build 2>&1 | grep -i error

# کدام کامیت‌های دیروز فلان کلمه را دارند:
$ git log --oneline | grep -i "login"

# چند خط ERROR در لاگ امروز:
$ grep -c "ERROR" app.log

# همه فایل‌هایی که axios مصرف می‌کنند (فقط اسم فایل):
$ grep -rl "axios" src/
# (-l = فقط اسم فایل‌ها، بدون خط‌ها)
```

> 💡 الگوی ذهنی: **«چیزی که دنبالشه در محتواست → grep؛ چیزی که دنبالشه یک فایله → find.»**

## ۷.۳ — find: شکار فایل‌ها در درخت

```bash
$ find . -name "*.test.js"              # همه تست‌ها از همینجا به پایین
$ find . -name "*.md" -not -path "*/node_modules/*"   # بدون node_modules ⭐
$ find . -type d -name "components"     # فقط پوشه (type d)، نه فایل
$ find . -size +10M                     # فایل‌های بزرگ‌تر از ۱۰ مگ
$ find . -name "*.log" -delete          # ⚠️ حذف همه لاگ‌ها (اول با -print تست کن!)
```

- الگوی اسم باید توی کوتیشن باشه (`"*.log"`) — چون شل می‌خواد خودش بازش کنه!
- قانون ایمنی همیشگی: قبل از `-delete`، همون find رو بدون `-delete` اجرا کن تا لیست رو ببینی.

## ۷.۴ — ترکیب find با ابزارهای دیگر (xargs بدون معرفی رسمی)

find خروجی‌اش اسم فایل است → می‌شه به دستور دیگه داد:

```bash
# چند خط در کل فایل‌های md حاوی TODO؟
$ find . -name "*.md" -exec grep -c "TODO" {} \;

# یا خواناتر با pipe به xargs:
$ find . -name "*.js" | xargs grep -l "console.log"
```

فعلاً فقط ببین الگو: find لیست می‌سازه → grep روش عملیات می‌کنه. (xargs = «خروجی قبلی رو به عنوان آرگومان بده».)

## ۷.۵ — دو ابزار مکمل که بهتره بدونی (شناختی)

| ابزار | چیست | چرا بدانیش |
|---|---|---|
| `du -sh *` | حجم هر آیتم این پوشه | «دیسک چرا پره؟» — با `sort -rh` عالی می‌شه |
| `df -h` | فضای کل دیسک‌ها | قبل از دانلود بزرگ، چک کن |

```bash
$ du -sh * | sort -rh | head -5
# 5 پوشه/فایل پرمصرف دیسک — پیدا شدن در ۲ ثانیه
```

## ۷.۶ — سناریوی کامل: «این تابع رو کجا تعریف و کجا صدا زدیم؟»

واقعی‌ترین موقعیت دولوپری: بازسازی/refactor یک تابع:

```bash
$ grep -rn "calcTotal" src/
src/utils/invoice.ts:8:   export function calcTotal(items) {
src/components/Cart.tsx:42:   const t = calcTotal(items);
src/components/Invoice.tsx:19:   return calcTotal(items);
```

در سه ثانیه فهمیدی: تعریف در invoice.ts خط ۸، دو مصرف‌کننده — و اینکه تغییر امضاش چه فایل‌هایی رو می‌شکنه. این همون «قدرت»یه که IDE ها با ده‌ها منو می‌دن؛ CLI با یک خط.

---

## ✅ جمع‌بندی فصل

- **grep** = جستجو در محتوا: `-i` بی‌حساسیت، `-n` شماره خط، `-r` بازگشتی، `-c` شمارش، `-v` معکوس، `-l` فقط اسم فایل
- **find** = جستجو در درخت: `-name` (با کوتیشن!)، `-type d`، `-size`؛ قبل از `-delete` بدونش اجرا کن
- انتخاب: محتوا → grep؛ فایل → find؛ ترکیب با pipe/xargs = قدرت واقعی
- `du` و `df` برای دیباگ دیسک

## 📝 تمرین فصل ۷ (در cli-playground)

1. داده بساز: سه فایل js بساز (`touch a.js b.js c.js`) و با `echo "console.log(1)" > a.js` (و b و c با متن‌های مختلف — یکی شامل `error` باش) پرشون کن.
2. با grep -rn پیدا کن کدوم فایل `error` داره — با شماره خط.
3. همه فایل‌های `.js` رو با find پیدا کن، بعد همین‌و برای `.md`.
4. یک فایل بزرگ‌فایک بساز (`head -c 20M /dev/zero > big.bin` — ۲۰ مگ بایت صفر!) و با find -size پیداش کن، بعد حذفش کن (اول با find بدون delete!).
5. چالش واقعی ⭐: در یکی از پروژه‌های خود، همه جاهای استفاده از یک dependency (مثلاً axios یا clsx) رو با grep -rn پیدا کن و ببین چند فایل بهش وابسته‌ان.

<details><summary>جواب تمرین ۱-۴</summary>

```bash
# ۲
grep -in "error" *.js        # (کوتیشن لازم نیست — کلمه ساده)
# ۳
find . -name "*.js"
find . -name "*.md"
# ۴
head -c 20M /dev/zero > big.bin
find . -size +10M            # لیست: ./big.bin
find . -size +10M -delete
```
</details>

➡️ **فصل بعد:** دنیای دسترسی‌ها (chmod/sudo) و اولین اسکریپت اجرایی خودت.
