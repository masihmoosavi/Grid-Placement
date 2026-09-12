# Unity Grid Building System

این پروژه یک سیستم ساده و قابل توسعه برای **ساخت‌وساز (Building System) مبتنی بر Grid در Unity** است. بازیکن می‌تواند اشیاء مختلف را روی خانه‌های Grid قرار دهد، اعتبار محل قرارگیری را مشاهده کند و در صورت نیاز اشیاء ساخته‌شده را حذف کند.

سیستم از معماری **State Pattern** برای مدیریت حالت‌های مختلف ساخت‌وساز استفاده می‌کند و شامل قابلیت‌های Placement، Removing، Preview و Sound Feedback است.

---

## ✨ Features

- قرار دادن اشیاء روی Grid
- حذف اشیاء قرار داده‌شده
- سیستم Preview قبل از قرار دادن Object
- نمایش معتبر یا نامعتبر بودن محل قرارگیری
- پشتیبانی از اشیاء با اندازه‌های مختلف
- جلوگیری از قرار گرفتن چند Object روی یک خانه
- سیستم جداگانه برای Floor و Furniture
- استفاده از State Pattern برای مدیریت حالت‌ها
- سیستم صوتی برای عملیات مختلف
- جلوگیری از Placement هنگام کلیک روی UI
- خروج از حالت ساخت‌وساز با کلید `Escape`

---

# 🏗️ Architecture

سیستم اصلی ساخت‌وساز با استفاده از الگوی **State Pattern** طراحی شده است.

رابط `IBuildingState` رفتارهای مشترک تمام حالت‌های ساخت‌وساز را تعریف می‌کند:

```csharp
public interface IBuildingState
{
    void EndState();
    void OnAction(Vector3Int gridPosition);
    void UpdateState(Vector3Int gridPosition);
}
```

در حال حاضر سیستم دارای دو حالت اصلی است:

- `PlacementState`
- `RemovingState`

کلاس `PlacementSystem` مسئول مدیریت و تغییر بین این حالت‌ها است.

---

# 📦 PlacementState

کلاس `PlacementState` مسئول قرار دادن Object جدید روی Grid است.

مهم‌ترین وظایف این کلاس:

- پیدا کردن Object موردنظر از Database
- نمایش Preview Object
- بررسی معتبر بودن محل Placement
- جلوگیری از قرار دادن Object روی خانه‌های اشغال‌شده
- ایجاد Object در Scene
- ثبت اطلاعات Object در Grid
- پخش صدای مناسب هنگام Placement

هر Object می‌تواند اندازه متفاوتی داشته باشد:

```csharp
public Vector2Int Size { get; private set; }
```

بنابراین یک Object می‌تواند بیش از یک Cell از Grid را اشغال کند.

---

# 🗑️ RemovingState

کلاس `RemovingState` مسئول حذف Objectهای قرار داده‌شده است.

این State بررسی می‌کند که آیا در موقعیت انتخاب‌شده Objectی وجود دارد یا خیر.

در صورت وجود Object:

1. اطلاعات Object از `GridData` حذف می‌شود.
2. GameObject مربوطه از Scene حذف می‌شود.
3. صدای حذف پخش می‌شود.

اگر هیچ Objectی در موقعیت انتخاب‌شده وجود نداشته باشد، صدای خطا پخش خواهد شد.

---

# 🟩 GridData

کلاس `GridData` مسئول نگهداری اطلاعات مربوط به خانه‌های اشغال‌شده Grid است.

این کلاس از یک `Dictionary` استفاده می‌کند:

```csharp
Dictionary<Vector3Int, PlacementData> placedObjects
```

هر موقعیت Grid اطلاعات Object قرار گرفته در آن را نگهداری می‌کند.

مهم‌ترین وظایف `GridData` عبارت‌اند از:

- ثبت Object جدید
- بررسی امکان قرار دادن Object
- محاسبه خانه‌های اشغال‌شده توسط Object
- پیدا کردن Representation Index
- حذف Object از Grid

سیستم از دو `GridData` جداگانه استفاده می‌کند:

```text
Floor Data
Furniture Data
```

این موضوع باعث می‌شود Floor و Furniture بتوانند به صورت مستقل مدیریت شوند.

---

# 👁️ Preview System

کلاس `PreviewSystem` مسئول نمایش پیش‌نمایش Object قبل از Placement است.

هنگام حرکت Mouse روی Grid، سیستم وضعیت محل انتخاب‌شده را بررسی می‌کند.

### محل معتبر

Preview به رنگ سفید نمایش داده می‌شود.

### محل نامعتبر

Preview و Cell Indicator به رنگ قرمز نمایش داده می‌شوند.

Preview همچنین اندازه Object را در Grid نمایش می‌دهد تا بازیکن بتواند فضای اشغال‌شده توسط Object را مشاهده کند.

---

# 📚 Objects Database

اطلاعات Objectهای قابل Placement در یک `ScriptableObject` ذخیره می‌شوند:

```csharp
ObjectsDatabaseSO
```

هر Object دارای اطلاعات زیر است:

- Name
- ID
- Size
- Prefab

ساختار اطلاعات هر Object:

```csharp
public class ObjectData
{
    public string Name { get; private set; }
    public int ID { get; private set; }
    public Vector2Int Size { get; private set; }
    public GameObject Prefab { get; private set; }
}
```

این روش باعث می‌شود اضافه کردن Objectهای جدید بدون تغییر کد اصلی سیستم انجام شود.

---

# 🖱️ Input System

کلاس `InputManager` مسئول دریافت ورودی بازیکن است.

عملیات اصلی:

- کلیک چپ Mouse برای Placement یا Removing
- کلید `Escape` برای خروج از حالت فعلی
- Raycast برای پیدا کردن موقعیت Mouse روی Map
- جلوگیری از اجرای عملیات هنگام قرار داشتن Pointer روی UI

موقعیت Mouse با استفاده از Raycast به موقعیت مناسب روی Grid تبدیل می‌شود.

---

# 🔊 Sound Feedback

کلاس `SoundFeedback` مسئول مدیریت صداهای سیستم است.

صداهای موجود:

- Click
- Place
- Remove
- Wrong Placement

نوع صدا با استفاده از Enum مشخص می‌شود:

```csharp
public enum SoundType
{
    Click,
    Place,
    Remove,
    wrongPlacement
}
```

این سیستم باعث می‌شود بخش صوتی از منطق اصلی Placement جدا باقی بماند.

---

# 🎮 Controls

| Input | Action |
|---|---|
| Left Mouse Click | Place / Remove Object |
| Mouse Movement | Update Preview |
| Escape | Exit Building Mode |

---

# 🔄 System Workflow

روند کلی سیستم به شکل زیر است:

```text
Player Selects Object
        ↓
PlacementSystem Creates PlacementState
        ↓
Preview System Starts
        ↓
Mouse Position → Raycast
        ↓
World Position → Grid Position
        ↓
Check Placement Validity
        ↓
Valid? ── No → Show Red Preview
   │
  Yes
   ↓
Place Object
   ↓
Save Object Data in GridData
```

برای حذف Object نیز:

```text
Player Starts Removing Mode
        ↓
RemovingState Starts
        ↓
Player Selects Grid Cell
        ↓
Check Object in GridData
        ↓
Object Found?
   │
Yes ───────── No
│              │
Remove         Play Error Sound
│
Update GridData
│
Destroy GameObject
```

---

# 🛠️ Technologies

- Unity
- C#
- Unity Grid System
- Physics Raycasting
- ScriptableObject
- State Pattern
- Dictionary Data Structure

---

# 📁 Main Scripts

```text
Scripts
│
├── PlacementSystem.cs
├── PlacementState.cs
├── RemovingState.cs
├── IBuildingState.cs
│
├── GridData.cs
├── ObjectPlacer.cs
├── PreviewSystem.cs
│
├── InputManager.cs
├── SoundFeedback.cs
│
└── ObjectsDatabaseSO.cs
```

---

# 🚀 Future Improvements

برخی قابلیت‌هایی که می‌توان در آینده به سیستم اضافه کرد:

- چرخاندن Object قبل از Placement
- Undo / Redo System
- ذخیره و Load کردن ساختمان‌ها
- سیستم هزینه و منابع
- Object Selection
- ارتقای سیستم Preview
- پشتیبانی از چند Layer مختلف
- پشتیبانی از Buildingهای پیچیده‌تر
- استفاده از New Input System
- بهینه‌سازی سیستم برای پروژه‌های بزرگ‌تر

---

# 📌 Summary

این پروژه یک سیستم **Grid-Based Building System** برای Unity است که امکان قرار دادن و حذف Objectها را روی Grid فراهم می‌کند. معماری سیستم به گونه‌ای طراحی شده که بخش‌های مختلف مانند Input، Preview، Data Management، Object Placement و Sound Feedback از یکدیگر جدا باشند.

استفاده از **State Pattern** باعث شده مدیریت حالت‌های Placement و Removing ساده‌تر و قابل توسعه‌تر شود. همچنین استفاده از `GridData` امکان مدیریت دقیق خانه‌های اشغال‌شده و جلوگیری از قرارگیری Objectها روی یکدیگر را فراهم می‌کند.

این سیستم می‌تواند به عنوان پایه‌ای برای بازی‌های **City Builder، Simulation، Strategy و Building Games** توسعه داده شود.