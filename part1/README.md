## Зміст
1. [Початкові налаштування](#r1)
2. [Базове наповнення Product.wxs файлу](#r2)
3. [Системні шляхи для Directory](#r3)
3. [Локалізація](#r4)
4. [Додавання діалогових вікон](#r5)
    - [Самостійне додавання - НЕ РОЗГЛЯНУТО](#r5_1)
    - [Використання готового набору](#5_2)
5. [Макроси препроцесору (змінні)](#6)
6. [Створення ярлика](#r7)
7. [Інсталяція готового пакету](#r8)
14. [Корисності](#extra)
    - [Умова встановлення адміністратором](#extra_1)
15. [Посилання](#references)
______


## <a name="r1">Початкові налаштування</a>
Необхідно завантажити та встановити
- WiX Toolset - https://wixtoolset.org/ За посиланням переходимо в секцію завантажень та знаходимо актуальну на зараз версію. Після цього потрапляємо на сторінку в git де в розділі асетів знаходимо та завантажуємо exe файл
- Компонент .NET Framework. Після запуску інсталяції WiX він може просигналізувати, що не вистачає цього компоненту та вкаже потрібну версію. ЇЇ можна нагуглити в прив'язці до потрібної версії ОС. Посилання варто брати з сайту Майкрософт
- Для інтеграції у VS знаходим розширення для потрібної версії студії тут https://marketplace.visualstudio.com/items?itemName=WixToolset.WiXToolset Завантажуємо та встановлюємо його
- При створенні проекту обираємо Setup Project for WiX

## <a name="r2">Базове наповнення Product.wxs файлу</a>
Після створення проекту ми отримуємо Product.wxs файл з наступним вмістом
~~~wxs
<?xml version="1.0" encoding="UTF-8"?>
<Wix xmlns="http://schemas.microsoft.com/wix/2006/wi">
	<Product Id="*" Name="task1_CommonInfo" Language="1033" Version="1.0.0.0" Manufacturer="" UpgradeCode="97028dd3-6ab9-4122-a8a0-a347be02bd28">
		<Package InstallerVersion="200" Compressed="yes" InstallScope="perMachine" />

		<MajorUpgrade DowngradeErrorMessage="A newer version of [ProductName] is already installed." />
		<MediaTemplate />

		<Feature Id="ProductFeature" Title="task1_CommonInfo" Level="1">
			<ComponentGroupRef Id="ProductComponents" />
		</Feature>

	</Product>

	<Fragment>
		<Directory Id="TARGETDIR" Name="SourceDir">
			<Directory Id="ProgramFilesFolder">
				<Directory Id="INSTALLFOLDER" Name="task1_CommonInfo" />
			</Directory>
		</Directory>
	</Fragment>

	<Fragment>
		<ComponentGroup Id="ProductComponents" Directory="INSTALLFOLDER">
			<!-- TODO: Remove the comments around this Component element and the ComponentRef below in order to add resources to this installer. -->
			<!-- <Component Id="ProductComponent"> -->
				<!-- TODO: Insert files, registry keys, and other resources here. -->
			<!-- </Component> -->
		</ComponentGroup>
	</Fragment>
</Wix>

~~~
Розберемо призначення кожного рядку:
~~~wxs
<?xml version="1.0" encoding="UTF-8"?>
~~~
Це декларація XML документа, яка вказує парсеру XML на те, що він має робити з файлом.  
В даному випадку, цей рядок означає, що ми маємо справу з XML файлом версії 1.0 та що використовується кодування UTF-8 для символів в цьому файлі.
___
~~~wxs
<Wix xmlns="http://schemas.microsoft.com/wix/2006/wi">
~~~
Фактично вказівка на те, що даний файл є WiX файлом. Він має простір імен "http://schemas.microsoft.com/wix/2006/wi", який вказує на версію Wix, що використовується в файлі (2006 рік).

Цей рядок має синтаксичне значення для парсера XML, який відповідає за зчитування вихідного коду Wix. Він гарантує, що файл буде правильно проаналізований і опрацьований програмним забезпеченням Wix, що буде використовуватися для створення MSI-пакету.
___
~~~wxs
<Product Id="*" Name="task1_CommonInfo" Language="1058" Version="1.0.0.0" Manufacturer="" UpgradeCode="97028dd3-6ab9-4122-a8a0-a347be02bd28">
~~~
**Product** це елемент (також ключ, або тег) в мові розмітки XML. Значення в середині є атрибутами цього елементу:
- `Id` - унікальний ідентифікатор продукту GUID, що дозволяє визначити його в системі. Це спеціальний шістнадцятковий ключ. За замовчуванням в ньому стоїть значення `*` що говорить WiX автоматично згенерувати його. Однак в такому випадку кожний раз буде генеруватись новий ключ. Це добре якщо забуваєш змінити ключ від продукту до продукту. Але для одного конкретного продукту цей ключ має бути однаковим протягом всього часу його існування. Тож його треба генерувати самостійно. Якщо між версіями продукту змінити id, він буде вважатись новим та система встановить новую версію поряд зі старою. Технічно цей ідентифікатор може міститись в коді самого продукту. Згенерувати GUID можна у Visual Studio через Tools - Create GUID - Registry format
- `Name` - назва продукту. За замовчуванням сюди підставлено назву проекту. Звичайно за потреби необхідно вказати назву продукту самостійно
- `Language` - код мови продукту. За замовчуванням тут знаходиться код англійської мови (1033). Цей параметр встановлює мову, яку використовує інсталятор для свого інтерфейсу та повідомлень
- `Version` - версія продукту. Використовується наприклад для перевірки чи встановлена вже більш рання чи пізня версія продукту. А також для сумісності між різними операційними системами
- `Manufacturer` - виробник продукту
- `UpgradeCode` - унікальний 16-вий ідентифікатор GUID. Він має залишатись однаковим протягом всього існування продукту. Саме перевіряючи однаковість цього ідентифікатору, система розумію чи встановлює новий продукт, чи оновлює старий. Якщо для різних версій одного продукту використати різний UpgradeCode це призведе до встановлення цих версій паралельно. Це значення генерується при створенні проекту автоматично

Окрім того, ключ *Product* може містити також наступні атрибути:
- `Codepage`: вказує кодову сторінку, яку слід використовувати для розкодування текстових рядків. За замовчуванням використовується кодова сторінка Windows-1252;
- `Comments`: містить коментарі до продукту;
- `Keywords`: вказує ключові слова, що допомагають користувачам знайти продукт у пошукових системах;
- `Description`: містить опис продукту;
- `Licensing`: вказує тип ліцензії для продукту;
- `ManufacturerUrl`: містить URL-адресу виробника продукту;
- `HelpTelephone`: вказує телефонну підтримку для продукту;
- `HelpLink`: містить посилання на сторінку підтримки продукту;
- `UpdateUrl`: містить URL-адресу для перевірки наявності оновлень для продукту;
___
~~~wxs
<Package InstallerVersion="200" Compressed="yes" InstallScope="perMachine" />
~~~
**Package** - ключ, який відповідає за налаштування пакета, що буде створений при компіляції проекту за допомогою Wix Toolset
- `InstallerVersion` - вказує версію Windows Installer, яку необхідно використовувати для встановлення пакету. У цьому випадку використовується версія 2.0.
- `Compressed` - вказує, що файли у пакеті повинні бути стиснуті. Це зменшує розмір пакету і поліпшує час завантаження
- `InstallScope` - визначає область, в яку буде встановлений продукт, має два значення:
    - perUser - встановлення буде доступне лише для користувача, який встановив продукт
    - perMachine - встановлення буде доступне для всіх користувачів, які використовують комп'ютер

Окрім того, ключ *Package* може містити також наступні атрибути:
- `InstallPrivileges`- вказує, чи потрібно запускати інсталятор в режимі підвищених привілеїв. Можливі значення: *limited* (запустити в обмеженому режимі), *elevated* (запустити з підвищеними привілеями), *elevatedRequireSignature* (запустити з підвищеними привілеями, якщо встановлено цифровий підпис);
- `Languages`- дозволяє вказати список мов, які повинні бути встановлені разом з продуктом;
- `SummaryCodepage`- вказує кодування для короткого опису продукту.
___
~~~wxs
<MajorUpgrade DowngradeErrorMessage="A newer version of [ProductName] is already installed." />
~~~
**MajorUpgrade** - це ключ, який залежить від атрибуту *Version* ключа *Product* та працює тільки у випадку оновлення продукту. Ключ:
- `DowngradeErrorMessage` - вказує яке повідомлення буде виведен при спробі встановити стару версію продукту, якщо в системі наявна більш нова версія

Окрім того цей ключ також може мати наступні атрибути:
- `AllowDowngrades` - дозволяє встановлення більш старої версії продукту, що може бути корисним для тестування, але не рекомендується для використання в продуктивному середовищі;
- `IgnoreRemoveFailure` - дозволяє проходження процесу оновлення навіть у випадку, коли не вдалося видалити попередню версію продукту;
- `MigrateFeatures` - дозволяє зберігати налаштування властивостей компонентів при оновленні версії продукту;
- `Schedule` - визначає порядок виконання дій під час процесу оновлення (може бути "afterInstallExecute" або "afterInstallFinalize").
___
~~~wxs
<MediaTemplate />
~~~
**MediaTemplate** - ключ який використовується для створення Windows Installer пакетів (.msi). Він описує структуру і компоненти установочних пакетів. Може використовуватись для опису як інсталяційні файли мають розміщуватись на носії. Але може бути корисним і при встановленні безпосередньо з пам'яті пристрою. Наприклад атрибут EmbedCab дозволяє вказати, що всі файли інсталяції мають бути вбудовані безпосередньо в .msi файл

*MediaTemplate* може мати наступні атрибути: 
- `EmbedCab`: Встановлює значення yes або no для вбудовування кабінет-файлів (CAB-файлів) безпосередньо в .msi файл. За замовчуванням, значення встановлено як yes;
- `CompressionLevel`: Встановлює рівень стиснення для кабінет-файлів. Значення можуть бути: none, low, medium, high, maximum. За замовчуванням, значення встановлено як high;

Наступні аргументи є застарілими та використовуються дуже рідко оскільки зазвичай установочні пакети розміщуються на одному медіа-носії або передаються через мережу
- `DiskPrompt`: Встановлює текстовий рядок, який відображається користувачу, коли потрібно вставити наступний диск або медіа-носій під час встановлення. За замовчуванням використовується системний текстовий рядок;
- `VolumeLabel`: Встановлює текстовий рядок, який буде використовуватися як мітка для медіа-дисків або медіа-носіїв, на яких розміщується установочний пакет;
- `MaximumUncompressedMediaSize`: Встановлює максимальний розмір кабінет-файлів в мегабайтах, які не стиснуті. За замовчуванням, значення встановлено як 0, що означає, що розмір не обмежений.
___
~~~wxs
<Feature Id="ProductFeature" Title="task1_CommonInfo" Level="1">
	<ComponentGroupRef Id="ProductComponents" />
</Feature>
~~~
**Feature** - це логічна група компонентів, які пов'язані між собою якоюсь спільною логікою і мають бути встановлені на комп'ютер користувача разом
- `Id` - Унікальний ідентифікатор поточної групи компонентів. Його можна використовувати в умовах задаючи послідовність встановлення компонентів, забороняючи встановлення та інш.
- `Title` - Назва групи компонентів. Може відображатися в інтерфейсі установки, коли користувач вибирає, які компоненти встановлювати.
- `Level` - Рівень видимості та доступності копонентів під час встановлення. Значення Level - це ціле число, яке може варіюватися від 0 до 32767. Різні значення Level мають різне значення:
    - 0: Компонент не буде доступним і користувач не зможе вибрати його під час встановлення. Це може використовуватися, якщо необхідно приховати компонент та наприклад встановлювати його в межах інших компонентів за допомогою залежностей
	- 1: Компонент буде встановлений за замовчуванням, якщо користувач не вибере інші опції (наприклад не відміне їх)
	- 2-32767: Ці значення використовуються для опціональних компонентів, які користувач може вибрати під час встановлення. Вищі значення означають меншу ймовірність встановлення компонентів за замовчуванням. Наприклад, якщо ми маємо групи компонентів з Level="1", Level="2", Level="3" та користувач самостійно не вибер жоден з них, то компоненти Level="1" встановляться обов'язково. Level="2" та Level="3" не є обов'язковими, тож якщо користувач самостійно не вказав їх то встановляться лише компоненти з вищим пріоритетом, тобто з Level="2".

Окрім того цей ключ також може мати наступні атрибути:
- `Absent`: Вказує, чи компонент має бути встановлений за замовчуванням. Значення може бути "allow" (встановлювати за замовчуванням) або "disallow" (не встановлювати за замовчуванням)
`AllowAdvertise`: Вказує, чи може компонент бути доступним для реклами. Значення може бути "yes" (дозволити рекламу) або "no" (заборонити рекламу)
`ConfigurableDirectory`: Вказує, чи може користувач вибрати папку для встановлення групи компонентів. Значення може бути "yes" (користувач може вибрати папку) або "no" (папка встановлення компонентів фіксована)
`Description`: Опис групи компонентів, який відображатиметься в діалогових вікнах інсталяції
`Display`: Порядок відображення групи компонентів в діалогових вікнах інсталяції. Чим менший номер, тим раніше буде відображатися група
`Hidden`: Вказує, чи група є прихованою. Значення може бути "yes" (група прихована) або "no" (група відкрита).
`InstallDefault`: Вказує, чи група має бути встановлена за замовчуванням в режимі інсталяції за замовчуванням. Значення може бути "followParent" (слідкувати за встановленими групами) або "true" (встановити за замовчуванням)
`TypicalDefault`: Вказує, чи група має бути встановлена за замовчуванням в типовому режимі інсталяції. Значення може бути "followParent" (слідкувати за встановленими групами) або "true" (встановити за замовчуванням).
`Vital`: Вказує, чи група є необхідною для функціонування програми. Значення може бути "yes" (група є важливою) або "no" (група не є важливою).

**ComponentGroupRef** - елемент, що говорить, які саме компоненти входять в визначену у Feature групу компонентів. Цей ключ містить лише один атрибут id який повністю має співпадати з id ключа ComponentGroup, який власне і містить опис компонентів групи, що мають бути встановлені
___
~~~wxs
<Fragment></Fragment>
~~~
**Fragment** - використовується для групування елементів, які складаються на структуру інсталяційного пакета, але не є самі по собі чинниками інсталяції. Може містити в собі наступні елементи: Component, ComponentGroup, Directory, Feature, Icon, IniFile, Shortcut.

Fragment може мати наступні атрибути:
- `Id`: Унікальний ідентифікатор фрагмента, що відповідає за його ідентифікацію в межах файлу .wxs
- `DiskId`: Ідентифікатор диску, на якому містяться файли, пов'язані з цим фрагментом
- `Language`: Мова, яку використовує фрагмент
- `Version`: Версія фрагменту
Є також і інші атрибути, що можуть бути використані, але вони різняться в залежності від контексту фрагменту
___
~~~wix
<Fragment>
	<Directory Id="TARGETDIR" Name="SourceDir">
		<Directory Id="ProgramFilesFolder">
			<Directory Id="INSTALLFOLDER" Name="task1_CommonInfo" />
		</Directory>
	</Directory>
</Fragment>
~~~
**Directory** - це елемент, що дозволяє визначити структуру директорів для встановлення продукту
- `<Directory Id="TARGETDIR" Name="SourceDir">`: Коренева директорія для всіх папок, що будуть використовуватись для встановлення продукту. *TARGETDIR* - це внутрішній ідентифікатор для *WiX*, який вказує на директорію, де відбувається встановлення. *SourceDir* - це псевдонім для *TARGETDIR*, який представляє кореневу директорію вихідних файлів.

- `<Directory Id="ProgramFilesFolder">`: Директорія всередині TARGETDIR. ProgramFilesFolder - це внутрішній ідентифікатор для WiX, який вказує на директорію "Program Files" або "Program Files (x86)" в залежності від архітектури продукту

- `<Directory Id="INSTALLFOLDER" Name="task1_CommonInfo" />`: Директорія всередині `ProgramFilesFolder`. `INSTALLFOLDER` - ідентифікатор, назву якого можна придумати самостійно, використовується для посилання на цю директорію в інших частинах WiX-скрипта. Name="task1_CommonInfo" вказує на назву директорії, яка буде створена в директорії "Program Files" під час встановлення  продукту
___
~~~wxs
<ComponentGroup Id="ProductComponents" Directory="INSTALLFOLDER">
</ComponentGroup>
~~~
**ComponentGroup** -  використовується для групування компонентів, які можуть бути встановлені на комп'ютер користувача. Цей ключ може містити в собі один або більше компонентів, які мають бути встановлені як частина інсталяційного пакету.
- `Id` - Унікальний ідентифікатор групи компонентів, який використовується для посилання на групу компонентів в ComponentGroupRef
- `Directory` - Цей атрибут вказує на директорію, в яку будуть встановлені компоненти з даної групи компонентів. Якщо директорія не вказана, то встановлювач самостійно визначає місце встановлення компонентів

Також можуть бути використані наступні атрибути:
- `ComponentIdGenerationSeed`: Цей атрибут використовується для генерації унікальних ідентифікаторів компонентів в групі компонентів. Якщо вказати цей атрибут, то унікальні ідентифікатори будуть генеруватися від заданого значення. Якщо цей атрибут не вказано, то ідентифікатори будуть згенеровані автоматично
- `Source`: Цей атрибут вказує на джерело, з якого потрібно взяти файли, пов'язані з компонентами групи компонентів. Це може бути шлях до папки з файлами або шлях до файлу .cab

Окрім того можуть бути додаткові атрибути в залежності від контексту використання ключа

~~~wxs
<Component Id="ProductComponent">
~~~
**Component** - це атомарна одиниця встановлення в Windows Installer, яка містить один або декілька ресурсів, таких як файли, ключі реєстру, шрифти, сервіси тощо. Тобто це конкретна вказівка, що саме має бути встановлено.  
Може містити наступні атрибути:
- `Id`: унікальний ідентифікатор компонента (обов'язковий атрибут)
- `Guid`: GUID компонента - необов'язковий атрибут, якщо встановлюється атрибут KeyPath. Проте, на практиці, часто рекомендується використовувати GUID для компонентів, щоб забезпечити унікальність та можливість прослідкувати встановлені компоненти в системі. GUID допомагає Windows Installer визначати, які компоненти можуть бути оновлені або видалені. Коли продукт оновлюється або видаляється , Windows Installer шукає GUID компонентів, щоб визначити, які з них потребують оновлення або видалення. Якщо GUID компоненту змінено, Windows Installer може вважати його за новий компонент і не оновити або не видалити попередній компонент
- `Directory`: ідентифікатор каталогу, в якому повинен бути розміщений компонент (обов'язковий атрибут)
- `KeyPath`: шлях до ключового ресурсу компонента (необов'язковий атрибут, якщо встановлено атрибут Guid)
- `NeverOverwrite`: вказує, чи потрібно перезаписувати існуючі файли (за замовчуванням no)
- `SharedDllRefCount`: вказує, чи потрібно підтримувати спільний лічильник посилань на DLL (за замовчуванням no)
- `Transitive`: вказує, чи повинен бути включений компонент у відповідний пакет (за замовчуванням no)
- `Win64`: вказує, чи повинен компонент бути встановлений на 64-бітній платформі (за замовчуванням no)
- `Location`: визначає, де розміщувати файли компонента на диску (може бути абсолютним або відносним шляхом)
- `Permanent`: вказує, чи має бути компонент встановлений постійно, не залежно від наявності відповідного продукту (за замовчуванням no)
- `Shared`: вказує, чи можуть бути файли компонента використані декількома продуктами (за замовчуванням no)
- `UninstallWhenSuperseded`: вказує, чи потрібно видаляти компонент при встановленні більш нової версії (за замовчуванням no)
- `Vital`: вказує, чи є компонент життєво важливим

## <a name="r3">Системні шляхи для Directory</a>
Як вже було сказано у попередньому розділі Directory використовується для визначення структури каталогів в межах інсталяційного пакету. Рядк його ідентифікаторів пов'язаний з системними папками, а саме:
- `TARGETDIR`: головна системна папка, зазвичай C:\
- `ProgramFilesFolder`: папка, куди зазвичай встановлюються програми на комп'ютері. Зазвичай це C:\Program Files
- `ProgramFiles64Folder`: папка, куди зазвичай встановлюються 64-бітні програми на комп'ютері. Зазвичай це C:\Program Files (x86)
- `SystemFolder`: системна папка з виконуваними файлами, бібліотеками та іншими файлами, що необхідні для роботи операційної системи. Зазвичай це C:\Windows\System32
- `SysWOW64Folder`: системна папка з виконуваними файлами, бібліотеками та іншими файлами, що необхідні для роботи 32-бітних програм на 64-бітній операційній системі. Зазвичай це C:\Windows\SysWOW64
- `DesktopFolder`: папка робочого столу користувача. Зазвичай це C:\Users\UserName\Desktop
- `CommonDesktopFolder`: загальна папка робочого столу для всіх користувачів. Зазвичай це C:\Users\Public\Desktop
- `AppDataFolder`: папка з даними, які використовуються програмами. Зазвичай це C:\Users\UserName\AppData\Roaming
- `CommonAppDataFolder`: загальна папка з даними, які використовуються програмами для всіх користувачів. Зазвичай це C:\ProgramData
- `ProgramMenuFolder`: Директорія, де знаходяться ярлики меню Пуск. Зазвичай вона розташована за шляхом %AppData%\Microsoft\Windows\Start Menu\Programs (для поточного користувача) або %ProgramData%\Microsoft\Windows\Start Menu\Programs (для всіх користувачів)
- `ApplicationProgramsFolder` - якщо вкладений у `ProgramMenuFolder` посилається на Пуск - Всі програми

## <a name="r4">Локалізація</a>
Розглянемо локалізацію на прикладі додавання української мови:
- У файлі Product.wxs для атрибута Language встановлюємо значення 1058, яке відповідає українській мові. Нажаль повнго списку цих ключів не знайшов навіть в офіційній документації. В цілому цей ключ використовується для встановлення мови локалізації для вихідного коду, що включає в себе текстові рядки, назви файлів та директорій, повідомлення, ярлики, та інші тексти.
- В налаштуваннях проекту *Solution Explorer -> Properties* на закладці *Build* в поле *Cultures to build* необхідно додати uk-UA. Повний перелік мов можна знайти тут: https://wixtoolset.org/docs/v3/wixui/wixui_localization/. В цілому дане налаштування використовується для вибору культур, для яких буде створюватися вихідний код та мовні пакети
- Далі необхідно налаштувати підтримку вибраної мови в інтерфейсі встановлення (для показу повідомлень та написів на кнопках та інших елементах). Для цього:
    - З репозиторія, з якого було викачано інсталер WiX (https://wixtoolset.org/) завантажумо архів wix311-debug.zip.
    - Розпаковуємо архів та переходимо до папки: wix311-debug.zip\src\ext\UIExtension\wixlib\
    - Копіюємо з неї файл WixUI_uk-UA.wxl в папку з проектом
    - Додаємо файл до проекту *Add -> Existing Item*

## <a name="r5">Додавання діалогових вікон</a>
Існує два варіанти додавання діалогових вікон. Самостійно або з використанням готового набору діалогових вікон.

### <a name="r5_1">Самостійне додавання</a>

### <a name="r5_2">Використання готового набору</a>

Щоб скористатись готовими діалоговими вікнами їх спочатку необхідно додати до проекту. Для цього:
- Додаємо в проекті посилання на *"C:\Program Files (x86)\WiX Toolset v3.11\bin\WixUIExtension.dll"* за допомогою *Add -> Reference*
- В кінці розділу Product вказуємо який набір буде використовуватись

~~~wxs
<Property Id="WIXUI_INSTALLDIR" Value="INSTALLFOLDER" ></Property>
<UIRef Id="WixUI_InstallDir"/>
~~~

Розберемо додані рядки:
- `Id="WIXUI_INSTALLDIR"` - ідентифікатор (по суті тип) який дозволяє зберігти шлях, вибраний користувачем для встановлення додатку
- `INSTALLLOCATION` -назва змінної, в якій зберігається вибраний шлях встановлення додатку (можна вибрати будь яку іншу назву). Варто бути дуже обережним з цим значенням. Воно напряму пов'язане з блоком `<Directory Id="ProgramFilesFolder">` конкретніше з `<Directory Id="INSTALLLOCATION"`. Назва id тут має повністю співпадати з назвою створеної змінної. В разі невідповідності скоріше за все під час інсталяції буде отримана помилка 2343. Замість INSTALLLOCATION не рідко використовують назву INSTALLFOLDER.
- `UIRef` - по суті посилання на доданий раніше файл *WixUIExtension.dll*
- `Id="WixUI_InstallDir"` - набір елементів інтерфейсу, що відповідають етапу інсталяції додатку

В ціломуіснують наступні набори елементів інтерфейсу:
1. **WixUI_Advanced** - набір елементів управління для забезпечення більш розширених можливостей інсталяції, включаючи діалоги вибору властивостей, вибору компонентів та мов
2. **WixUI_Basic** - простий набір елементів управління, який містить діалоги для вибору мови та каталогу встановлення
3. **WixUI_InstallDir** - набір елементів управління, який містить діалог для вибору каталогу встановлення
4. **WixUI_FeatureTree** - набір елементів управління, який містить діалог для вибору функцій, які користувач може встановити або не встановити
5. **WixUI_Minimal** - мінімальний набір елементів управління, який містить лише діалог вибору каталогу встановлення.
6. **WixUI_Mondo** - набір елементів управління, який містить діалоги для вибору мови, каталогу встановлення, вибору функцій та налаштування опцій встановлення.
7. **WixUI_ProgressOnly** - набір елементів управління, який містить лише діалог відображення ходу встановлення.
8. **WixUI_Simple** - простий набір елементів управління, який містить діалоги для вибору мови та каталогу встановлення, а також діалог завершення встановлення.
9. **WixUI_Web** - набір елементів управління, який містить діалоги для вибору мови, каталогу встановлення, вибору функцій та введення налаштувань зв'язку з веб-сервером.

## <a name="r6">Макроси препроцесора (змінні)</a>
Макрос у файлі .wxs визначається за допомогою директиви `<?define ... ?>`. Виглядає це наступним чином
~~~wxs
<?define Назва_змінної="Значення_змінної" ?>
~~~
Після створення макрос можна підставляти замість значень атрибутів наступним чином:
~~~wxs
SomeAttribute="$(var.Назва_змінної)"
~~~
Наприклад маємо рядок
~~~wxs
<Product Id="*" Name="SomeProj" Language="1058" Version="1.0.0.0" Manufacturer="" UpgradeCode="97028dd3-6ab9-4122-a8a0-a347be02bd28">
~~~
І розуміємо, що версія продукту буде постійно змінюватись і в усіх місцях її використання прийдеться робити коррекцію. Тож створюємо макрос:
~~~wxs
<?define ProductVersion="Значення_змінної" ?>
~~~
Та змінюємо початковий рядок з урахуванням створеного макросу
~~~wxs
<Product Id="*" Name="SomeProj" Language="1058" Version="$(var.ProductVersion)" Manufacturer="" UpgradeCode="97028dd3-6ab9-4122-a8a0-a347be02bd28">
~~~

## <a name="r7">Створення ярлика</a>
Для створення ярлика в меню Пуск необхідно наступне
- Задати директорією створення `ProgramMenuFolder`
- Створити підрозділ для свого продукту за допомогою `ApplicationProgramsFolder` та назви цієї директорії (зазвичай визначається макросом)
- Задати компонент та заповнити його елементом `Shortcut`
- Задекларувати, що дана директорія має бути видалена в разі деінсталяції продукту за допомогою `RemoveFolder`
- [Опціонально] Додати в реєстр ключ з інформацією про те, що продукт встановлений (або присутній його ярлик в меню пуск)

Приблизний приклад того, як це може виглядати:
~~~wxs
<!-- Set shortcuts for our product -->
      <!-- Directory where Start menu shortcuts are located -->
      <Directory Id="ProgramMenuFolder">
        <!-- All Programs section of the Start menu -->
        <Directory Id="ApplicationProgramsFolder" Name="$(var.ProductName)">     
          <!-- Set component for shortcut (must be defined in feature) -->
          <Component Id="SomeApplicationShortcut" Guid="4CEBD68F-E933-47f9-B02C-A4FC69FDB551">
            <!-- Set component element - in this case it shortcut -->
            <Shortcut Id="ShortcutSomeProgram"
                 Name="SomeProgram"
                 Description="$(var.ProductName)"
                 Target="[INSTALLLOCATION]SomeProgram.exe"
                 WorkingDirectory="INSTALLLOCATION"/>
            <!-- Define that directory must be delete in case of uninstall product -->
            <RemoveFolder Id="ApplicationProgramsFolder" On="uninstall"/>
            <!-- Set in registry information about install status -->
            <RegistryValue Root="HKCU" Key="Software\$(var.Manufacturer)\$(var.ProductName)" Name="installed" Type="integer" Value="1" KeyPath="yes"/>
          </Component>
        </Directory>
      </Directory>>
~~~
Розберемо детальніше:
- `<Directory Id="ProgramMenuFolder">` - місцем розташування задається папка "Всі програми" в меню пуск
- `<Directory Id="ApplicationProgramsFolder" Name="$(var.ProductName)">`
    - `ApplicationProgramsFolder` - створює підпапку для продукту у папці "Всі програми"
	- `Name="$(var.ProductName)"` - задає ім'я папки (в даному випадку ім'я визначене в макросі ProductName)
- `<Component Id="SomeApplicationShortcut" Guid="7435E631-0BBD-4FDC-8B19-9C972DEB49D8">` - налаштовуємо компонент, що має бути розміщеним у обраній директорії
    - `SomeApplicationShortcut` - id компоненту, він має бути оголошений у блоці `ComponentRef` десь у програмі з тим самим id
    - `Guid` - унікальний код, необхідно згенерувати самостійно
- `Shortcut Id="ShortcutSomeProgram"` - елемент, що встановлюється у вказаному компоненті. В даному випадку цим елементом є ярлик
    - `Name` - унікальний ідентифікатор ярлика
    - `Description` - опис продукту, в даному випадку в ньому просто виводиться назва цього продукту збережена у змінній ProductName
    - `Target` - для якої програми створюється ярлик, в даному випадку INSTALLLOCATION містить шлях до папки, куди вона встановлена
    - `WorkingDirectory` - робоча директорія, в якій буде виконуватись програма. Зазвичай це та сама директорія в якій знаходиться сама програма
- `RemoveFolder` - по суті відповідає за видалення папки. Але в парі з атрибутом On="uninstall" автоматично запустить видалення папки `ApplicationProgramsFolder` під час видалення програми
- `RegistryValue` - зберігає в реєстр необхідні дані, наприклад це може бути зміна, що містить статус встановлення програми
    - `Root` - гілка реєстру куди буде збережене значення, в даному випадку HKCU це HKEY_CURRENT_USER
    - `Key`- шлях до ключа реєстру, в якому буде створено запис
    - `Name` - ім'я значення, яке буде створено в реєстрі
    - `Type`- тип значення, яке буде зберігатися в реєстрі
    - `Value` - значення, яке буде збережено в реєстрі
	- `KeyPath` - вказує, що цей запис в реєстрі є ключовим для компонента, який він представляє. Це означає, що інсталяційний пакет використовує цей запис для визначення наявності компонента на комп'ютері.

## <a name="r8">Інсталяція готового пакету</a>
Для встановлення достатньо запустити msi файл, але в разі проблем інсталяцію можна запустити разом зі створенням файлу логів
~~~
msiexec /i InstallerName.msi /l* LogFileName.log
~~~
Або більш деталізований файл логів
~~~
msiexec /i InstallerName.msi /l*v LogFileName.log
~~~
Видалити встановлену програму можна за допомогою контрольної панелі або командою
~~~
msiexec /x InstallerName.msi
~~~

## <a name="extra">Корисності</a>
## <a name="extra_1">Умова встановлення адміністратором</a>
Щоб обмежити можливість встановлення програми та надати її тільки адміністратору, додаємо:
~~~wxs
<Condition Message="You need to be an administrator to install this product.">
    Privileged
</Condition>
Як видно в умові ми описуємо ситуацію, в якій інсталяція продовжиться
~~~


## <a name="references">Посилання</a>
1. Стаття на хабрі: "Створення інсталятора за допомогою WiX. Частина 1" - https://habr.com/ru/articles/68616/
2. Послідовність встановлення WiX - https://www.educba.com/install-wix/ 
3. Документація по Product - https://wixtoolset.org/docs/v3/xsd/wix/product/
4. Туторіал по WiX - https://www.firegiant.com/wix/tutorial/
5. Назви системних папок - https://learn.microsoft.com/en-us/previous-versions//aa372057(v=vs.85)?redirectedfrom=MSDN
6. Властивості, які можна використовувати в умовах - https://learn.microsoft.com/uk-ua/windows/win32/msi/property-reference?redirectedfrom=MSDN#operating_system_properties 
