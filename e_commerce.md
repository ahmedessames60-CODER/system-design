# مرجعي الشخصي — بنية مشروع Laravel + Vue + Inertia

## الفكرة العامة

المشروع مقسم بنظام **Feature-based subfolders** جوه كل نوع (Controllers, Models, ...).
يعني مش كل الـ Controllers مع بعض flat، لكن مقسمين بـ Domain: `Catalog`, `Cms`, `Orders`, `Users`.
ده بيدّي تنظيم قوي من غير ما تتحمل تعقيد DDD الكامل

---

## بنية الفولدرات

```
app/
├── Http/
│   ├── Controllers/
│   │   ├── Admin/{Catalog,Cms,Orders,Users}/...
│   │   └── Shop/{Catalog,Cart,Checkout}/...
│   ├── Requests/
│   │   ├── Admin/Catalog/StoreProductRequest.php
│   │   └── Shop/Checkout/PlaceOrderRequest.php
│   ├── Resources/
│   │   ├── ProductResource.php
│   │   └── OrderResource.php
│   └── Middleware/
├── Models/
│   ├── Catalog/{Product,Category}.php
│   ├── Cms/{Section,Setting,NavigationItem}.php
│   └── Orders/Order.php
├── Services/
│   ├── Catalog/ProductService.php
│   ├── Cart/CartService.php
│   └── Checkout/CheckoutService.php
├── Actions/
│   ├── Orders/CreateOrderAction.php
│   ├── Orders/ApplyCouponAction.php
│   └── Cms/ReorderSectionsAction.php
├── Enums/
│   ├── OrderStatus.php
│   └── SectionType.php
├── Events/
│   └── OrderPlaced.php
├── Listeners/
│   ├── SendOrderConfirmationEmail.php
│   └── DecreaseStock.php
├── Jobs/
│   └── ProcessProductImage.php
└── Policies/
    ├── ProductPolicy.php
    └── OrderPolicy.php
```

---

## شرح كل مفهوم

### Controllers
بس بيستقبل الـ request وبينده على Service/Action ويرجع Response.
**مفيهوش لوجيك تقيل** — لو لقيت نفسك بتكتب كذا سطر منطق جوه الكنترولر، ده إشارة إنه المفروض يروح Service أو Action.

### Requests (Form Requests)
الـ validation بتاع كل عملية، منفصل عن الكنترولر.
مثال: `StoreProductRequest` → لازم الاسم موجود، السعر رقم، الصورة صيغتها صح.

### Resources
بتحول الـ Model لشكل JSON محدد قبل ما تتبعت لـ Vue عن طريق Inertia.
بتتحكم بالظبط ايه اللي يتبعت وايه لأ (مثلاً متبعتش `cost_price` الداخلي للعميل).

### Middleware
فلتر بيتنفذ **قبل** ما الـ Request يوصل للـ Controller.
مثال: "تأكد إن المستخدم أدمن قبل ما يدخل صفحات الداش".

### Models
تمثيل جدول الداتا بيز + العلاقات بينهم (`hasMany`, `belongsTo`, ...).

### Services
مجموعة عمليات **مترابطة** حول موضوع واحد، بتتنادى من أكتر من مكان في المشروع.
مثال: `CartService` → `add()`, `remove()`, `getTotal()`, `applyCoupon()`.

### Actions
عملية **واحدة** محددة، بتتنفذ وتخلص، اسمها فعل واضح.
مثال: `CreateOrderAction` → خطوة واحدة بس، تحول السلة لطلب فعلي.

**الفرق بين Service و Action:**
| لو... | استخدم |
|---|---|
| العملية بتتنادى من أكتر من مكان بنفس الشكل بالظبط | Action |
| فيه كذا عملية مترابطة حول نفس الموضوع | Service |

### Enums
قيم ثابتة معرّفة بأسماء واضحة بدل ما تكتب strings يدوي في أماكن كتير.
مثال: `OrderStatus::Pending`, `OrderStatus::Shipped` بدل `"pending"` مكتوبة يدوي.

### Events & Listeners
- **Event** = إعلان إن حاجة حصلت (مش عملية بحد ذاتها). مثال: `OrderPlaced`.
- **Listener** = اللي بيحصل كنتيجة للـ Event، كل واحد شغلته منفصلة.
  - `SendOrderConfirmationEmail` → يبعت إيميل
  - `DecreaseStock` → يقلل المخزون

**الفرق بين Action و Event/Listener:**
- Action بتتنادى **مباشرة** من الكنترولر ("نفّذ العملية دي دلوقتي").
- Event/Listener بيحصل **كنتيجة جانبية** لعملية تانية، من غير ما الكنترولر يعرف تفاصيلها.

مثال تطبيقي: `CreateOrderAction` بينفذ إنشاء الطلب، وفي آخره بيطلق `OrderPlaced`،
والـ Listeners بتتكفل بالإيميل وتقليل المخزون من غير ما تتحشر جوه الـ Action نفسه.

### Jobs
مهمة بطيئة أو مش لازم تحصل فورًا، بتتنفذ في الخلفية (Queue).
مثال: `ProcessProductImage` → ضغط/تصغير الصورة بعد الرفع من غير ما يستنى المستخدم.

### Policies
قواعد الصلاحيات: مين يقدر يعمل ايه بالظبط على أي Model.
مثال: `ProductPolicy` → بس الأدمن يقدر يمسح منتج. `OrderPolicy` → العميل يشوف بس طلباته هو.

---

## قاعدة مهمة — متكسرهاش

> ابدأ بسيط جوه الـ Controller، ولما تلاقي نفسك بتكرر نفس الكود في مكانين،
> وقتها اسحبه لـ Service أو Action.

متبدأش بالتجريد (abstraction) من الأول لكل حاجة — دي أكبر غلطة بتاخد وقت
وتبطّئ التسليم من غير فايدة حقيقية في مشروع لسه بيتبني.

---

## ملاحظات CMS خاصة بالمشروع

- **Settings** (لون، لوجو، عنوان الموقع): جدول key-value أو JSON، مش أعمدة ثابتة —
  عشان تضيف إعداد جديد من غير migration.
- **Sections** (ترتيب السكاشن بالـ drag & drop): جدول `sections` بـ
  `type`, `order`, `config` (JSON), `is_visible`. الفرونت بيستخدم component map
  (`{ hero: HeroSection, products: ProductsSection }`) عشان يعرض كل نوع بالـ component بتاعه.
- **Navbar**: جدول `navigation_items` بـ `label`, `url`, `order`, `parent_id`.
- **اللغات (i18n)**: Laravel Blade i18n لوحده مش كافي مع Inertia/Vue —
  استخدم `laravel-vue-i18n` عشان توحد مصدر الترجمة بين الباك والفرونت.

---

## لو المشروع كبر أوي مستقبلاً

فيه package اسمه `nwidart/laravel-modules` بيخليك تعمل كل feature كـ module مستقل
(Routes, Controllers, Views, Migrations خاصة بيه) من غير ما تخرج بره Laravel القياسي.
**متستخدمهاش دلوقتي** — سيبها للمرحلة اللي المشروع فيها فعلاً 5-6 features كبيرة مستقلة عن بعض.
