> [برگشت به فهرست](README.md) · قبلی: [۰۷ — CLAUDE.md و AGENTS.md](07-claude-md-agents-md.md) · بعدی: [۰۹ — ابزارهای ضروری](09-essential-tools.md)

# فصل ۰۸ — مهندسی پرامپت

## چارچوب کلی

Anthropic تکنیک‌ها را **به ترتیب اثرگذاری** مرتب می‌کند. از بالا شروع کنید؛ اگر جواب داد، پایین‌تر نروید.

```
۱. دستور واضح و مستقیم        ← بیشترین اثر، کمترین هزینه
۲. مثال (multishot)
۳. اجازهٔ فکر کردن (CoT)
۴. تگ XML
۵. نقش دادن (system prompt)
۶. پیش‌پر کردن پاسخ
۷. زنجیرهٔ پرامپت
۸. بهینه‌سازی برای کار طولانی
```

---

## بخش ۱ — آناتومی یک پرامپت خوب

| جزء | سؤالی که پاسخ می‌دهد | مثال |
|---|---|---|
| **Role** | تو کی هستی؟ | «تو یک مهندس امنیت هستی» |
| **Task** | چه کار کن؟ | «این تابع را برای SQL injection بررسی کن» |
| **Context** | چه بدانی؟ | کد، اسکیمای دیتابیس، محدودیت‌ها |
| **Constraints** | چه نکن؟ | «امضای تابع را عوض نکن» |
| **Format** | خروجی چه شکلی؟ | «جدول: file، line، severity، fix» |
| **Examples** | نمونهٔ درست | یک ورودی/خروجی واقعی |
| **Success** | کی تمام است؟ | «وقتی `npm test` سبز شد» |

---

## بخش ۲ — واضح و مستقیم باشید

**تست طلایی:** پرامپت را به یک همکار جدید نشان دهید که هیچ context ندارد. اگر گیج شد، مدل هم گیج می‌شود.

| ❌ | ✅ |
|---|---|
| «این کد را بهتر کن» | «این تابع را طوری refactor کن که هر خطای شبکه را با backoff نمایی ۳ بار retry کند. امضای تابع تغییر نکند.» |
| «تست بنویس» | «برای `calculateTax` در `src/tax.ts` تست Jest بنویس. موارد: نرخ صفر، عدد منفی، عدد خیلی بزرگ، ورودی null.» |
| «باگ را پیدا کن» | «کاربران گزارش می‌دهند بعد از ریست پسورد لاگین می‌مانند. جریان را از `POST /auth/reset` تا ساخت session دنبال کن و علت را پیدا کن.» |

**قواعد:**
- به‌جای «چه نکن»، بگویید «چه بکن»
- عدد بدهید: «۳ مورد» نه «چند مورد»
- خروجی مطلوب را توصیف کنید

---

## بخش ۳ — مثال بدهید (Multishot)

مؤثرترین تکنیک بعد از وضوح. ۳ تا ۵ مثال متنوع.

```xml
<examples>
<example>
<input>getUserById</input>
<output>get_user_by_id</output>
</example>
<example>
<input>parseHTTPResponse</input>
<output>parse_http_response</output>
</example>
<example>
<input>id</input>
<output>id</output>
</example>
</examples>

Convert this to snake_case: <input>fetchAPIToken2</input>
```

مثال‌ها باید **حالت‌های لبه** را پوشش دهند، نه فقط حالت عادی.

---

## بخش ۴ — اجازهٔ فکر کردن (Chain of Thought)

برای تحلیل، ریاضی، دیباگ، تصمیم چندمرحله‌ای.

```xml
Debug this failing test.

Think through it in <thinking> tags first:
1. What does the test assert?
2. What does the code actually return?
3. Where do they diverge?

Then give your answer in <answer> tags.
```

در Claude Code کلمات کلیدی budget فکر کردن را افزایش می‌دهند:

```
think < think hard < think harder < ultrathink
```

```
ultrathink about the best architecture for this rate limiter
```

⚠️ فکر کردن بیشتر = توکن بیشتر = هزینهٔ بیشتر. برای کار ساده استفاده نکنید.

---

## بخش ۵ — تگ XML

Claude به‌طور خاص روی تگ XML آموزش دیده. برای جدا کردن بخش‌ها بهترین ابزار است.

```xml
<context>
This is a Next.js 15 app using the App Router and Drizzle ORM.
</context>

<code>
export async function getUser(id: string) { ... }
</code>

<task>
Add caching with a 60-second TTL.
</task>

<constraints>
- Do not change the function signature
- Use the existing Redis client from src/lib/redis.ts
- Cache misses must not throw
</constraints>
```

تگ‌های نام استاندارد ندارند — فقط **ثابت** باشید.

---

## بخش ۶ — نقش دادن

```
You are a senior database engineer specializing in PostgreSQL
performance. You care about query plans and index selectivity,
not ORM syntax.
```

اثرگذاری: عمق تخصص، واژگان، چیزی که مدل مهم می‌داند.

در Claude Code این کار را در [Subagent](05-subagents.md) انجام می‌دهید.

---

## بخش ۷ — زنجیرهٔ پرامپت

کار پیچیده را نشکنید = خروجی متوسط در همه چیز.

```
پرامپت ۱: «کدبیس را بخوان و معماری احراز هویت فعلی را توضیح بده»
   ↓ (خروجی را بررسی کنید)
پرامپت ۲: «بر اساس آن، نقشهٔ افزودن OAuth را بده»
   ↓ (نقشه را اصلاح کنید)
پرامپت ۳: «مرحلهٔ ۱ نقشه را پیاده‌سازی کن»
```

در Claude Code: **Plan Mode** دقیقاً همین است، خودکار.

---

## بخش ۸ — الگوهای مخصوص کد

### الگوی TDD

```
1. Write tests for this spec. Do not write implementation yet.
2. Run them and confirm they fail for the right reason.
3. Now implement until they pass. Do not modify the tests.
```

قوی‌ترین الگو در کدنویسی با AI. تست، معیار عینی موفقیت می‌دهد و جلوی «تظاهر به کار کردن» را می‌گیرد.

### الگوی Explore → Plan → Code → Commit

```
1. Read src/auth/ and tell me how sessions work. Don't write code.
2. Plan how to add refresh tokens. Show me the plan.
3. Implement step 1 of the plan.
4. Run the tests.
5. Commit.
```

### الگوی حلقهٔ بازخورد بصری

```
Take a screenshot of localhost:3000 with Playwright.
Compare it to mockup.png.
Fix the differences. Screenshot again. Repeat until they match.
```

معیار عینی = خروجی به‌مراتب بهتر.

---

## بخش ۹ — ضدالگوها

| ضدالگو | چرا بد است | به‌جایش |
|---|---|---|
| «همه چیز را درست کن» | هدف مبهم = خروجی مبهم | یک کار مشخص |
| context بی‌نهایت | attention رقیق می‌شود | `/clear` بین کارها |
| بدون معیار موفقیت | مدل خودش تصمیم می‌گیرد کی تمام شده | تست/شرط عینی |
| «مطمئنی؟» بدون دلیل | مدل ممکن است پاسخ درست را عوض کند | خطای مشخص را نشان دهید |
| پذیرش بی‌قیدوشرط خروجی | باگ وارد production می‌شود | همیشه review |

---

## بخش ۱۰ — چک‌لیست جیبی

```
□ آیا یک همکار جدید این را می‌فهمد؟
□ آیا هدف یکی است، نه پنج تا؟
□ آیا context لازم داده شده؟
□ آیا محدودیت‌ها صریح‌اند؟
□ آیا فرمت خروجی مشخص است؟
□ آیا معیار «تمام شد» عینی است؟
□ آیا مثال لازم است؟
□ آیا باید به چند پرامپت بشکنم؟
```

---

## منابع

- نمای کلی مهندسی پرامپت — https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview
- واضح و مستقیم — https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/be-clear-and-direct
- Multishot — https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/multishot-prompting
- Chain of Thought — https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/chain-of-thought
- تگ XML — https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/use-xml-tags
- زنجیرهٔ پرامپت — https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/chain-prompts
- کتابخانهٔ پرامپت‌ها — https://docs.claude.com/en/resources/prompt-library
- آموزش تعاملی — https://github.com/anthropics/prompt-eng-interactive-tutorial
- بهترین شیوه‌های Claude Code — https://www.anthropic.com/engineering/claude-code-best-practices
