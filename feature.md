# E-commerce + CMS — Feature & Architecture Document

<!--==========================================================
                  1. Overview & Current Status
==========================================================-->

> **آخر تحديث:** 2026-11-07
> **الحالة:** ⏸️ **موقوف مؤقتًا** (قسم 1.1)
> **آخر تشغيل للتيستات:** ✅ **165 ناجح + 2 متخطّيين = 167 تيست** (894 assertion، 67.66 ثانية)
> **الهدف الأساسي:** تعلّم بناء CMS حقيقي فوق متجر اكسسوارات (مشروع تعليمي شخصي).
> **الملف ده للحالة والقرارات.** النطاق (Scope) في `BRD.md`، والقواعد في `rules-*.md`.

المشروع متجر إلكتروني بنظام **Guest Checkout** ولوحة إدارة للأدمن فقط. الـ Shop والـ Catalog والـ Orders والـ Contact خلصوا ومغطيين بتيستات، والـ CMS القديم اتلغى (ماكانش مكتمل وكان ملغبط) وماتبناش من جديد (قسم 6).

| الجزء | الحالة | ملاحظات |
|---|---|---|
| Testing | ✅ 165 ناجح، 2 متخطّيين | PHPUnit (Unit + Feature)، 894 assertion. جرد الملفات في قسم 8. مفيش E2E آلي |
| Authentication | ✅ مكتمل | Fortify + Two-Factor + Passkeys، تسجيل دخول الأدمن بس. `User` بيطبّق `MustVerifyEmail` |
| Authorization | ✅ مكتمل ومغطى بتيستات | `/dashboard` و`/admin` للأدمن فقط. مفيش اختبار مخصص لـ `ADMIN_EMAILS` (قسم 1.1) |
| Catalog | ✅ مكتمل ومغطى بتيستات | Products, Categories, Images, Filters, Discounts |
| Cart | ✅ مكتمل | Pinia store + localStorage. مفيش اختبارات آلية للفرونت |
| Guest Checkout | ✅ مكتمل ومغطى بتيستات | إنشاء الطلب، حجز المخزون، صفحة النجاح، "طلباتي" |
| Order Management | ✅ مكتمل ومغطى بتيستات | الحالات، الحجز، الإلغاء، استرجاع المخزون |
| Expired Reservations | ✅ مكتمل ومغطى بتيستات | Scheduled command كل ساعة |
| Shop UI/UX | ✅ مكتمل (هيتظبط تاني) | Dark Mode، Navbar scroll، إعادة تصميم الصفحات |
| Contact / Messages | ✅ مكتمل ومغطى بتيستات | فورم عام + Admin inbox وتغيير الحالة |
| CMS | ⏸️ ماتبناش | الخطة في قسم 6 |

## 1.1 الحالة الحالية: موقوف (2026-11-07)

**السبب:** قرار بالتحول لتعلم ستاك جديد (React ثم API خارجية ثم .NET). مش بسبب مشكلة في المشروع.

**آخر تشغيل كامل:** `Tests: 2 skipped, 165 passed (894 assertions)`، المدة 67.66 ثانية (2026-11-07). الـ 2 المتخطّيين غالبًا تيستات التسجيل (Fortify registration مقفول عمدًا)، زي ما ظهر في تشغيلات سابقة.

**اللي اتحسم بعد التيستات:**

| # | الموضوع | النتيجة |
|---|---|---|
| 1 | صفحة `admin/categories/{id}/edit` رجّعت 500 في تشغيل سابق | اختفت، والتيستات كلها خضرا. **سبب الإصلاح غير موثق**، سجّله هنا لو فاكره |
| 2 | الأدمن غير المفعّل | `User` بيطبّق `MustVerifyEmail`، وفيه اختبارات بتغطي منعه (`AdminAccessTest`, `SecurityHardeningTest`) |
| 3 | الرجوع لحالة `reserved` | اتحسم في الكود (قسم 9) |

**اللي لسه مفتوح:**

| # | الموضوع | التفاصيل |
|---|---|---|
| 1 | `ADMIN_EMAILS` | جزء من التفويض، ومفيش اختبار مخصص له في أسماء التيستات. قرر لو محتاج تغطية |
| 2 | `verified` في بيئة `local` | ملاحظة مراجعة سابقة بتقول إن مسارات الإدارة في `local` مش بتضيف `verified`. `routes/web.php` لما اتراجع قبل كده كان فيه `verified`، فراجع الـ routes الحالية قبل النشر |
| 3 | تغطية الـ CRUD | تيستات الـ Catalog بتركز على الإنشاء وبعض التعديل والتحقق، وممكن تكون ناقصة سيناريوهات |
| 4 | الفحوص التانية | `npm run build` وPHPStan و`vue-tsc` ما اتسجلوش بعد التحديثات الأخيرة |
| 5 | النشر | المشروع غير منشور، وscheduler worker ما اتشغّلش فعليًا، وأداة الـ E2E ما اتحددتش |
| 6 | بقايا CMS قديم | ملفات Settings القديمة موجودة وغير موصولة (قسم 6) |

**ملاحظات من قراءة سابقة للكود (تحتاج إعادة فحص، مش أعطال مؤكدة):**

| الملاحظة | الدرجة |
|---|---|
| احتمال عدم تطابق نوع `ignoreId` في `ProductService::generateUniqueSlug()` مع UUID | اشتباه سابق؛ اختبارات slug موجودة لكن يلزم فحص التوقيع الحالي |
| إدخال `stock` فارغ في طلب المنتج | اشتباه سابق؛ فيه اختبار لحالة المخزون الفارغ في `ProductFormValidationEdgeCasesTest` |
| كشف تفاصيل الاستثناء في رد Checkout | ملاحظة سابقة، تحتاج مراجعة `CheckoutController` الحالي |
| سعر المنتج في صفحة إنشاء طلب الأدمن مقابل السعر المخفّض | فيه اختبارات لسعر الطلب المخفّض، ويلزم التحقق من الواجهة |
| تطابق `discount_price` القادم من الواجهة مع `discount_type` و`discount_value` | ملاحظة سابقة، تحتاج مراجعة قواعد التحقق الحالية |
| `slug` و`created_by` و`updated_by` في `$fillable` | `test_product_create_ignores_client_supplied_slug_and_creator_id` بيغطي تجاهل slug وcreator id، والحقول التانية تحتاج تحقق |
| بقايا CMS قديم غير موصول | `Setting` و`SettingService` و`Admin\SettingController` و`UpdateSettingsRequest` وmigration جدول settings. مفيش لها routes إدارية ولا صفحة. و`Shop/StoreOrderRequest.php` ما بيظهرش مستخدم |
| الـ route بتاع `/dashboard` فيه logic كتير جوه closure، ومخالف لقاعدة Controllers رفيعة | refactor |
| `BRD.md` أقسام 5.4 و5.5 و8 قديمة (Sections منفصلة، Theming، Navbar، Policies) | توثيق |

**لو رجعت للمشروع (بالترتيب):**
1. `herd isolate 8.5`، وأي أمر بـ `herd php` و`herd composer` (مش `php` العادي).
2. شغّل `herd php artisan test` وقارن بآخر نتيجة (165 ناجح، 2 متخطّيين).
3. راجع بند 1 و2 في "اللي لسه مفتوح".
4. شغّل `npm run build` وPHPStan و`vue-tsc`.
5. راجع بقايا CMS القديم: تتحذف ولا تتعاد استخدامها.
6. جهّز النشر وشغّل scheduler worker، وحدد أداة E2E.
7. بعد كده ابدأ الـ CMS من المرحلة 1 (قسم 6).

<!--==========================================================
                  2. Final Architecture Decisions
==========================================================-->

## 2. القرارات المعمارية النهائية

- **Stack:** Laravel + Fortify + Inertia.js (v3) + Vue 3 (`<script setup>`) + TypeScript + Pinia + Tailwind CSS.
- **Primary Keys:** UUID في كل الجداول (`uuid('id')->primary()` و`foreignUuid()`). الموديلز بتستخدم `HasUuids`. القرار كان **تمرين تعليمي مقصود** بناءً على نصيحة سنيور. أثره على الفرونت: أي `id` في الـ types و`routes.ts` بقى `string`.
- **Models:** Flat (`app/Models/Product.php` مباشرة). فولدر `Models/Concerns/` فضل لأنه convention في Laravel.
- **تقسيم باقي الطبقات:** Feature-based subfolders للـ Controllers/Services/Requests (`Catalog`, `Orders`, ...).
- **Business Logic:** Service من البداية والـ Controllers رفيعة (`rules-backend.md` بيكسب على فكرة "ابدأ جوه الـ Controller" القديمة).
- **Auth:** أدمن بس. العملاء Guest دايمًا ومفيش `/register`. دمج سلة الزائر **اتلغى نهائيًا**.
- **Authorization:** `role = admin`، أو إيميل موجود في `ADMIN_EMAILS` في `.env` المحلي فقط. مفيش setup route عام لإنشاء أدمن.
- **Order ownership:** طلبات الأدمن بتحفظ `auth()->user()` و`source = admin`. طلبات الضيف `user_id = null` و`source = customer`.
- **Inventory:** الخصم فوري وقت إنشاء الطلب جوه `DB::transaction()` مع `lockForUpdate()`. الإلغاء بيرجّع الكمية.
- **Frontend Structure:** كل منطق حقيقي في composables (`resources/js/composables/`)، والـ types في ملفات مركزية (`resources/js/types/`)، والـ Pinia stores زي ما هي.
- **Validation:** الباك هو source of truth، والفرونت بنفس القواعد بالظبط (validators مكتوبة يدويًا + `error-messages.js` مركزي).

<!--==========================================================
                    3. Completed Domains
==========================================================-->

## 3. الـ Domains المخلّصة

### 3.1 Catalog (Products + Categories)

**الجداول**
- `categories`: `name`, `slug` (unique), `gender` (male/female/unisex), `order`, `active`, `parent_id` (self-reference لكاتيجوري فرعية)، `created_by`/`updated_by` بـ `restrictOnDelete()`.
- `products`: `price`, `discount_type` (`fixed`/`percentage`), `discount_value`, `stock`, `gender`, `category_id`, `sub_category_id` (nullable)، `created_by`/`updated_by`.
- `products_images`: صور متعددة بعمود `order` و`is_primary`.

**اللي موجود**
- Admin CRUD كامل: Index (فلاتر + بحث + Stats Cards تفاعلية)، Create, Edit, Show.
- الفلترة كلها في الباك عبر Query Scopes (Trait مشترك `HasFilterableScopes`).
- رفع الصور: `hashName()` + `storeAs('products', $fileName, 'public')`. الـ `file_name` بيتخزن **شامل** `products/`، فالرابط `/storage/${file_name}` من غير تكرار الفولدر.
- Slug تلقائي وفريد عبر `generateUniqueSlug()` مع `withTrashed()` (عشان الـ soft deletes ما تعملش Duplicate entry).
- ⚠️ `slug` و`created_by` و`updated_by` **موجودين** في `$fillable` بتاع `Product`، والقصد الأصلي كان إنهم يتحطوا من الـ Service بس. آمن طالما الـ Request مبيسمحش بيهم، وفيه تيست (`test_product_create_ignores_client_supplied_slug_and_creator_id`) بيغطي slug وcreator id.
- Components: `ProductCard.vue`, `CategoryRow.vue`.

### 3.2 Orders + Checkout

**الجداول**
- `orders`: بيانات العميل، `total_price`, `status`, `reserved_until`, `source` (enum: `admin`/`customer`).
- `order_product` (Pivot): `quantity` و`price` وقت الطلب (snapshot).

**قواعد العمل**
- الحالات (من كود `Admin\Orders\OrderController`): `reserved`, `approved`, `shipped`, `delivering`, `completed`, `cancelled`. الطلب بيبدأ `reserved` (مفيش `pending`) والحجز 24 ساعة. حالة `expired` مش مستخدمة: الـ command بيلغي الحجز المنتهي بـ `cancelled`.
- مفيش `update()`/`destroy()` عاديين، عمليات محددة بس (`updateStatus`, `extendReservation`, `cancelOrder`).
- المنتجات غير النشطة مش بتتطلب، وتكرار نفس المنتج في السلة بيتجمّع قبل خصم المخزون.
- الرجوع التلقائي للمخزون: `orders:cancel-expired-reservations` بتشتغل `hourly` من `routes/console.php`.
- عداد حجز حي في الداش (`setInterval` + `onUnmounted` لمنع Memory Leak).

**ملفات أساسية**
- Backend: `OrderService::create(?User, array, array, OrderSource)`، `OrderSource` enum، `Shop\Checkout\CheckoutController`، `PlaceOrderRequest`، `ListMyOrdersRequest`، `Shop\Orders\OrderController`، `CancelExpiredReservationsCommand`.
- Routes: `GET/POST /checkout`, `GET /checkout/success/{order}`, صفحة "طلباتي".
- Frontend: `Web/Checkout/Index.vue`, `Web/Checkout/Success.vue`, `Web/Orders/Index.vue`, `useMyOrders.ts`.
- متابعة طلبات الضيف: الـ IDs بتتحفظ في `localStorage` (`my_order_ids`)، والباك بيحتفظ بقائمة session مصرّح بيها عشان محدش يطلب UUID عشوائي.

### 3.3 Contact / Messages

- `customer_messages` (UUID، `name/email/subject/message/status`، index على `status`) + `CustomerMessageStatus` enum.
- `Shop\Contact\ContactController` (استقبال الفورم) و`Admin\Messages\MessageController` (inbox على `/admin/messages`).
- مفيش Reply من الواجهة. الحالة بتتغير يدويًا.

### 3.4 الحساب والمصادقة

- تسجيل دخول الأدمن وإعادة تعيين كلمة المرور والتحقق من البريد وTwo-Factor Authentication وPasskeys عبر Laravel Fortify.
- إعدادات الحساب تشمل تعديل الاسم والبريد، تغيير كلمة المرور، حذف الحساب، وإعدادات المظهر والأمان.
- صفحات/مسارات الحساب معرفة في `routes/settings.php`، ومنطق الملف الشخصي في `Settings\ProfileController` والأمان في `Settings\SecurityController`.
- لا يوجد تسجيل عام للعملاء؛ الشراء يظل Guest Checkout.

<!--==========================================================
                      4. Shop Frontend
==========================================================-->

## 4. الـ Shop Frontend

- **Layout:** `ShopLayout.vue` بيستدعي `<Navbar />`. في `app.ts`، صفحات `Web/` بدون `AppLayout`. اسم الصفحة في `Inertia::render('Web/Home')` لازم يطابق `pages/Web/Home.vue` حرفيًا.
- **Pinia + Inertia v3:** `withApp(app) { app.use(pinia); }` بدل `setup()` التقليدية.
- **الصفحات:** Home، Products (Index/Show بـ Gallery)، Cart، Checkout، Success، Orders، Contact.
- **Components:** `Navbar.vue` (Dark Mode toggle، scroll animation، Hamburger، Cart dropdown)، `Footer.vue`، `ProductCard.vue`.
- **Composables (`composables/Web/`):**

| الملف | الوظيفة |
|---|---|
| `useDarkMode.ts` | Dark Mode: تخزين الاختيار في `localStorage`، class على `<html>`، دعم system preference |
| `useNavbarScroll.ts` | إخفاء/إظهار الـ Navbar مع threshold، ومبيختفيش في أعلى الصفحة |
| `useCartActions.ts` | إضافة/تعديل/حذف عناصر السلة |
| `useProductDisplay.ts` | الصورة الرئيسية والـ thumbnails |
| `useWebProductFilters.ts` | فلاتر الشوب |

- **Admin composables:** `useProductForm`, `useCategoryDeletion`, `useOrderStatus`, `useAdminProductFilters`.
- **مفاهيم اتعلمناها:** Debounce، `preserveState`/`preserveScroll`/`withQueryString()`، `routes.ts` كملف مركزي، `onClickOutside` من `@vueuse/core`، وأي متغير في `<template>` لازم يتعرّف صراحة في `<script setup>`.

<!--==========================================================
                       5. Quality Gate
==========================================================-->

## 5. الـ Quality Gate

| الفحص | آخر نتيجة | التاريخ |
|---|---|---|
| تيستات PHPUnit | ✅ 165 ناجح، 2 متخطّيين (894 assertion، 67.66 ثانية) | 2026-11-07 |
| `npm exec -- vue-tsc --noEmit` | passing | قبل التحديثات الأخيرة |
| Manual MVP flow + Dark Mode + Navbar scroll | ✅ | قبل التحديثات الأخيرة |

**لسه ناقص قبل النشر**
- [ ] تشغيل وتسجيل `npm run build` وPHPStan و`vue-tsc` بعد التغييرات الأخيرة.
- [ ] اختبار يدوي كامل في بيئة شبه الإنتاج.
- [ ] تشغيل scheduler worker فعليًا (تعريف الجدولة وحده مش كفاية).

<!--==========================================================
                    6. CMS (Current Plan)
==========================================================-->

## 6. الـ CMS — الخطة الحالية

**المبدأ:** نخلّصه بأي شكل بسيط الأول، وبعدين نطوّره حتة حتة. الـ CMS القديم (Settings + Navigation + Builder) **ماكانش مكتمل وكان ملغبط**، فاتلغى وبيتبني من الصفر بخطوات صغيرة، كل خطوة تخلص وتتجرّب قبل اللي بعدها.

**ملاحظة عن حالة الكود:** فيه بقايا من تنفيذ Settings القديم (`Setting`, `SettingService`, `Admin\SettingController`, `UpdateSettingsRequest` وmigration جدول `settings`)، لكنها غير موصولة بصفحات أو routes إدارة، و`HandleInertiaRequests` مش بيشارك إعدادات المتجر. وجودها لا يعني أن المرحلة 1 منفذة؛ لازم تقرر تعيد استخدامها ولا تحذفها قبل أي تنفيذ جديد.

### المرحلة 1 — Settings (فورم واحد في الداش)
- تعديل **لون الـ Navbar**.
- رفع **اللوجو** (صورة).
- مفيش إعدادات تانية تتضاف غير لما تتقرر.
- التخزين: جدول key/value بسيط (مرونة بيانات من غير migration لكل إعداد جديد)، وتوصيله للشوب عبر Inertia Shared Props.

### المرحلة 2 — الصور والمحتوى الثابت
- بدل الصور الثابتة في الصفحة: **إضافة وحذف الصور** من الداش.
- تعديل **المحتوى الثابت** (النصوص) من الداش.

### المرحلة 3 — Sections (آخر مرحلة)
- **Section** = مكعب جاهز بنوع ثابت (مثلًا Hero، Banner، نص) بيتضاف من الداش، **ومحتواه بس هو اللي بيتعدل**. الشكل نفسه ثابت في الكود.
- إظهار/إخفاء (`is_visible`) والترتيب عند الحاجة.
- دي نفس فكرة "Blocks" القديمة بس مبسّطة: المستخدم بيختار مكعب جاهز ويعدّل محتواه، مش بيصمم الصفحة.
- التفاصيل (الجداول، الأنواع) تتحدد في الـ execution plan قبل أي كود، وبعد ما المرحلتين 1 و2 يشتغلوا فعليًا.

<!--==========================================================
                    7. Post-MVP Backlog
==========================================================-->

## 7. Post-MVP Backlog

| الفكرة | ملاحظة / سبب التأجيل |
|---|---|
| Theming متقدم (ألوان متعددة + خطوط) | الأول لون الـ Navbar واللوجو بس |
| Navbar management (أسامي/روابط من الداش) | بعد ما الـ Settings والـ Sections يشتغلوا |
| Page Builder حر (صفحات ديناميكية بـ blocks) | الـ Sections المبسّطة تغطي المطلوب دلوقتي |
| i18n (تعدد اللغات) | `laravel-vue-i18n` لتوحيد الترجمة بين Laravel وVue |
| بوابة دفع (Paymob/Fawry) | تفصيل تكاملي، مش بيضيف خبرة CMS |
| Notifications (إيميل/SMS) | مش أساس الـ CMS |
| Wishlist / Reviews / Coupons | فيتشرز تجارية |
| فحص مخزون معقد | يتعمل لما تظهر مشاكل فعلية |
| Clone لمشروع مطعم (Backend جديد + React) | بعد اكتمال المشروع |
| مشروع إدارة مهام شبيه بـ Jira | مشروع منفصل بالكامل |

<!--==========================================================
                         8. Testing
==========================================================-->

## 8. Testing

> الـ checklists الجاهزة قبل التسليم موجودة في `task_delivery_checklist.md`. القسم ده للاستراتيجية والجرد ونتايج التشغيل.

**آخر نتيجة:** 165 ناجح + 2 متخطّيين = 167 تيست، 894 assertion (2026-11-07).

### 8.0 جرد الاختبارات الحالية

| النوع | اسم ملف/كلاس الاختبار | النطاق الأساسي |
|---|---|---|
| Unit | `tests/Unit/ProductPricingTest.php` | السعر النهائي والخصومات والنسبة المئوية والقيم الحدية مثل الخصم صفر والسعر صفر |
| Feature | `tests/Feature/Admin/AdminAccessTest.php` | الزائر يتحول للدخول، غير الأدمن 403، الأدمن غير المفعّل، وفتح كل صفحات الأدمن |
| Feature | `tests/Feature/Auth/AuthAuthorizationCoverageTest.php` | صلاحيات لوحة التحكم ووصول العميل للوحة الإدارة |
| Feature | `tests/Feature/Admin/AdminCatalogCoverageTest.php` | إنشاء تصنيف وتوليد slug، منع حذف تصنيف مستخدم، إنشاء منتج بخصم وتصنيف فرعي، وفلاتر المنتجات الإدارية |
| Feature | `tests/Feature/Admin/CategoryFormValidationEdgeCasesTest.php` | الحقول المطلوبة، القيم والأنواع غير الصحيحة، حدود الحقول، وقيود parent/child عند الإنشاء والتعديل |
| Feature | `tests/Feature/Admin/ProductFormValidationEdgeCasesTest.php` | التحقق عند إنشاء وتعديل المنتج، الحدود، المخزون الفارغ، الصور، وتجاهل حقول معينة يرسلها العميل |
| Feature | `tests/Feature/Orders/OrderServiceTest.php` | إنشاء الطلب والسعر والـ snapshot والمخزون والحجز والإلغاء ومنع الرجوع لـ `reserved` |
| Feature | `tests/Feature/Orders/CancelExpiredReservationsCommandTest.php` | إلغاء الحجوزات المنتهية، إبقاء الحجز الساري، تجاهل الحالات الأخرى، ومعالجة عدة طلبات |
| Feature | `tests/Feature/Shop/CheckoutTest.php` | Checkout كضيف، بيانات الطلب، المخزون والمنتجات غير النشطة/المحذوفة، وصفحة النجاح |
| Feature | `tests/Feature/Shop/OrderFormValidationEdgeCasesTest.php` | حقول ونوع/حدود بيانات الطلب في Checkout وطلب الأدمن، وصيغ الهاتف والحقول المحمية |
| Feature | `tests/Feature/Admin/AdminOrderTest.php` | إنشاء طلب الأدمن، التحقق، تحديث الحالة، الإلغاء، إعادة المخزون وتمديد الحجز |
| Feature | `tests/Feature/Shop/PublicCatalogCoverageTest.php` | الرئيسية، فلاتر الكتالوج العام، إخفاء المنتجات غير النشطة وصفحة التفاصيل |
| Feature | `tests/Feature/Shop/ContactMessagesCoverageTest.php` | إرسال رسالة، التحقق من البيانات، inbox الأدمن وتغيير الحالة والفلترة |
| Feature | `tests/Feature/Shop/ContactFormValidationEdgeCasesTest.php` | الحقول المطلوبة والفارغة والأنواع والأطوال والبريد، الحالة المرسلة من العميل وصلاحية تغيير الحالة |
| Feature | `tests/Feature/Security/SecurityHardeningTest.php` | منع العميل من تحديث طلب إداري، UUID غير معروف في صفحة النجاح، ومنع الأدمن غير المفعّل |
| Feature | `tests/Feature/Settings/AccountFormValidationEdgeCasesTest.php` | التحقق من بيانات الملف الشخصي وكلمة المرور وحذف الحساب |
| Feature | `tests/Feature/Settings/ProfileUpdateTest.php` | تحديث الملف الشخصي |
| Feature | `tests/Feature/Settings/SecurityTest.php` | إعدادات الأمان وكلمة المرور والمصادقة الإضافية |
| Feature | `tests/Feature/Auth/RegistrationPasswordValidationTest.php` | تحقق كلمة مرور التسجيل وفق القواعد الحالية |
| Feature (starter kit) | `tests/Feature/Auth/AuthenticationTest.php`, `EmailVerificationTest.php`, `PasswordConfirmationTest.php`, `PasswordResetTest.php`, `RegistrationTest.php`, `TwoFactorChallengeTest.php`, `VerificationNotificationTest.php` | تدفقات المصادقة والتحقق وإعادة تعيين كلمة المرور والتسجيل و2FA |
| Feature (starter kit) | `tests/Feature/DashboardTest.php` | وصول الضيف والأدمن والمستخدم العادي إلى `/dashboard` |
| PHPUnit examples | `tests/Unit/ExampleTest.php`, `tests/Feature/ExampleTest.php` | اختبارات مثال افتراضية، مش تغطية تجارية |

**ملاحظات على التغطية:** نجاح كل الاختبارات لا يعني اكتمال كل السيناريوهات. اختبارات الـ Catalog بتركز على الإنشاء وبعض التعديل والتحقق، ومفيش اختبار مخصص لـ `ADMIN_EMAILS`. ومفيش اختبارات متصفح E2E آلية، ولا اختبارات آلية لسلة Pinia أو واجهة Checkout في Vue.

### 8.1 المراحل الثلاثة

| المرحلة | بتختبر إيه | الأداة |
|---|---|---|
| 1. Function (Unit) | function/Service لوحدها (حساب الخصم، `OrderService::create()`، إرجاع المخزون) | PHPUnit |
| 2. Feature (Integration) | endpoint كامل: Route → Controller → Service → DB (الـ Checkout، الصلاحيات، المخزون) | PHPUnit Feature |
| 3. End-to-End | رحلة المستخدم كاملة من متصفح حقيقي (إضافة للسلة → Checkout → نجاح → إلغاء → رجوع المخزون) | **لسه هتتحدد** (مرشحين: Playwright أو Laravel Dusk) |

**قرار أداة الـ E2E:** مفتوح. بعد ما تقارن الأدوات يتسجل القرار والسبب في سجل القرارات (قسم 9). حتى وقتها الـ E2E بيتعمل يدويًا.

### 8.2 مجموعات التغطية (خط نهاية مرحلة التيستات)

| # | المجموعة | الملفات اللي بتغطيها |
|---|---|---|
| A | Checkout والمخزون | `CheckoutTest`, `OrderFormValidationEdgeCasesTest`, `OrderServiceTest` |
| B | الإلغاء والحجوزات المنتهية | `OrderServiceTest`, `CancelExpiredReservationsCommandTest`, واختبارات الإلغاء في `AdminOrderTest` |
| C | صلاحيات `/admin` | `AdminAccessTest`, `AuthAuthorizationCoverageTest`, `SecurityHardeningTest` |
| D | CRUD الكاتيجوري والمنتج | `AdminCatalogCoverageTest`, `CategoryFormValidationEdgeCasesTest`, `ProductFormValidationEdgeCasesTest` |
| E | الطلب اليدوي من الأدمن | `AdminOrderTest`, `OrderFormValidationEdgeCasesTest`, `OrderServiceTest` |
| F | رسايل Contact | `ContactMessagesCoverageTest`, `ContactFormValidationEdgeCasesTest` |

**قواعد عمل متفق عليها (التيستات بتتكتب على أساسها):**
- الطلب بيتسجّل بسعر الخصم (`final_price`) مش السعر الأصلي، وسعر الـ pivot (`order_product.price`) هو اللي اتدفع فعلًا.
- الإلغاء مسموح للطلب في حالة `reserved` بس. أي حالة تانية (مشحون، مسلّم...) ما تتلغيش.

**قواعد الكتابة:** 2 أو 3 edge cases لكل endpoint بس. التيست بيتأكد من الداتابيز والـ response مش الـ HTML. التيست الجديد يفشل الأول، وبعدين يتصلّح الكود.

### 8.3 سجل التشغيل

كل مرة تشغّل فيها فحص مهم بتسجّل سطر هنا (الأحدث فوق):

| التاريخ | المرحلة | الأمر / الطريقة | النتيجة |
|---|---|---|---|
| 2026-11-07 | الكل | `php artisan test` | ✅ **165 passed, 2 skipped (894 assertions)، 67.66 ثانية** |
| 2026-11-07 | Feature (مرحلة مبكرة) | `--filter=AdminAccessTest` | 2 passed، 2 failed (500 في `categories/{id}/edit`، وأدمن غير مفعّل دخل). اتصلّحوا لاحقًا |
| 2026-11-07 | الكل (مرحلة مبكرة) | `herd php artisan test` بعد HTTP Checkout + طلب الأدمن + الـ command | 100 passed, 2 skipped (308 assertions) |
| 2026-11-07 | الكل (مرحلة مبكرة) | `herd php artisan test` | 73 passed, 2 skipped (211 assertions) |
| 2026-11-07 | Feature (مرحلة مبكرة) | `--filter=OrderServiceTest` | 24 passed (66 assertions) بعد تصليح سعر الخصم وشرط الإلغاء. كشف باجين فعليين |
| 2026-11-07 | Unit (مرحلة مبكرة) | `--filter=ProductPricingTest` | 10 passed، 1 failed: `discount_percentage` مع خصم 0 بيرجّع 100 (باج، اتصلّح بشرط `> 0`) |
| 2026-11-07 | الكل (نقطة البداية) | `herd php artisan test` (PHP 8.5.12) | 38 passed, 2 skipped (133 assertions)، كلها starter kit |
| قبل 2026-11-04 | Types | `npm exec -- vue-tsc --noEmit` | passing |
| قبل 2026-11-04 | E2E يدوي | إضافة منتج → Checkout → إلغاء → رجوع المخزون | ✅ |

<!--==========================================================
                      9. Decision Log
==========================================================-->

## 9. سجل القرارات

| التاريخ | القرار | السبب |
|---|---|---|
| 2026-08-10 | الهدف الأساسي = CMS، مش نظام بيع كامل | مشروع للـ CV بيوري state management ومرونة بيانات |
| قبل الـ CMS | UUID بدل auto-increment + Models flat + composables/types | تدريب مقصود وتنظيف البنية قبل domain جديد |
| 2026-09-05 | إلغاء Auth العميل، Admin-only + Guest checkout | KISS، ومفيش داعي لحساب عميل في مشروع بالحجم ده |
| 2026-09-05 | دمج سلة الزائر **اتلغى نهائيًا** | نتيجة مباشرة لقرار إلغاء Auth العميل |
| 2026-09-05 | Scheduled Task حقيقي كل ساعة لرجوع المخزون | شرط أساسي لصحة الـ Checkout |
| 2026-09-06 | UI/UX Overhaul للشوب + Contact Domain | تحسين الشكل وقناة تواصل مع الزوار |
| 2026-09-16 | خطة نسخة React فوق نفس الباك إند | رغبة في تعلم React |
| 2026-09-17 | ❌ **اتلغت خطة الـ React Clone** | مجهود تحويل زيادة فوق التعلم (حمل مزدوج). React هيتعمل في مشروع منفصل بباك إند جديد |
| 2026-09-23 | بناء CMS (Settings + Navigation + Builder) | تنفيذ خطة الـ CMS الأصلية |
| 2026-11-04 | ❌ **إلغاء الـ CMS القديم بالكامل** | ماكانش مكتمل وكان ملغبط. هيتبني من جديد بخطوات صغيرة (قسم 6) |
| 2026-11-04 | الـ Blocks بقت Sections: مكعبات جاهزة بيتعدّل محتواها فقط، وهي **آخر مرحلة** في الـ CMS | تبسيط، ومفيش Page Builder حر |
| 2026-11-04 | الـ Testing جوه الملف ده (قسم 8) كاستراتيجية وسجل تشغيل مش checklist | الـ checklists موجودة في `task_delivery_checklist.md` |
| 2026-11-07 | PHP 8.5 للمشروع عبر `herd isolate 8.5` (بدل 8.6 RC) | 8.6 كانت نسخة تجريبية وبتكسر تيست بسبب تحذيرات مكتبات `vendor`، ونسخة الإنتاج هتكون مستقرة |
| 2026-11-07 | الطلب بيتحاسب بـ `final_price` (سعر الخصم) | اتأكد يدويًا إن العميل بيشوف سعر الخصم والطلب بيتسجل بالسعر الأصلي، وده تناقض. التصليح بعد كتابة تيست فاشل |
| 2026-11-07 | `cancelOrder()` مسموحة لحالة `reserved` بس | منع رجوع المخزون لبضاعة خرجت فعلًا، ومنع رجوعه مرتين |
| 2026-11-07 | الرجوع لحالة `reserved` بعد مغادرتها ممنوع | كان `UpdateOrderStatusRequest` بيسمح بأي حالة لأي حالة، فطلب راجع لـ `reserved` يفضل بدون `reserved_until` وعمره ما ينتهي. الكود الحالي بيمنعه، وتيست `test_a_cancelled_order_cannot_be_returned_to_reserved` بيغطي الطلب الملغي. ⚠️ بيغيّر قاعدة إعادة التفعيل القديمة (`cancelled` ← `reserved` بحجز جديد) |
| 2026-11-07 | ⏸️ **إيقاف المشروع مؤقتًا** | قرار بالتحول لتعلم ستاك جديد (React ثم API خارجية ثم .NET). الحالة والمفتوح في قسم 1.1 |

<!--==========================================================
                    10. Lessons Learned
==========================================================-->

## 10. دروس مستفادة

- **التحقق الفعلي أهم من تقرير الـ AI Agent.** مرة قال "خلصت" وبعدها `vue-tsc --noEmit` كشف: 17 ملف بيستوردوا من `types/models.ts` بعد ما اتمسح، و`pinia` مش في `package.json`، وباج `localStorage` القديم رجع في `cartStore.ts`، وكومبوننتس Admin في مكان UI عام. الحل: `vue-tsc --noEmit` و`npm run build` بدل الاعتماد على تقرير نصي.
- **تطابق الأسامي:** اسم العمود = `$fillable` = validation rule = اسم الحقل من الفرونت، وإلا فشل صامت من غير error.
- **باج قديم اتكشف بالصدفة:** `Admin\Orders\OrderController::store()` كان بيبعت `array` مكان `?User` لـ `OrderService::create()` وبيرمي `TypeError`. اتصلح بـ `Auth::user()` كأول باراميتر. التجربة الفعلية بتكشف اللي القراءة بتفوّته.
- **`scheduler` لازم يشتغل فعليًا** في الإنتاج، تعريف الأمر وحده مش بيشغّله.
- **ابدأ صغير:** الـ CMS القديم اتفتح فيه كذا حاجة مع بعض وماكملش وبقى ملغبط. الأفضل خطوة صغيرة بتخلص وتتجرّب فعلًا قبل الخطوة اللي بعدها.
- **اختبارات SQLite:** migrations الـ MySQL-specific بتكسر اختبارات SQLite، فلازم تتصلح عشان الـ tests تفضل شغالة.
- **عدد التيستات الناجحة مش معناه إن الـ logic متغطي.** وقت تشغيل الـ 38 تيست الأولى كانت كلها للـ Auth والـ Settings، وكل الـ Checkout والمخزون من غير تغطية. اقرا أسماء التيستات مش الرقم بس.
- **التيست اللي بيفشل الأول بيثبت إنه بيحمي حاجة.** تيست السعر وتيست الإلغاء كشفوا باجين فعليين قبل ما يتصلّحوا.
- **استخدم نسخة PHP المستقرة:** النسخة التجريبية بتجيب أخطاء مش بتاعتك. `herd isolate` بيثبّت النسخة للمشروع، و`herd php` و`herd composer` بيستخدموها.
