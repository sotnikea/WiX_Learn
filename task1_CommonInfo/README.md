# Умови завдання
Створити інсталяційний дистрибутив для встановлення звичайного калькулятора (C:\WINDOWS\system32\calc.exe), що буде містити наступні діалогові вікни:
- Привітання
- Ліцензійна угода
- Вибір директорії
- Початок встановлення

## Обзор рішення
Для використання в інсталяційному пакеті української мови додана локалізація. Для цього у ключі Product для атрибута Language встановлено значення 1058

~~~wxs
<Product Id="*" Name="task1_CommonInfo" Language="1058" Version="1.0.0.0" Manufacturer="" UpgradeCode="97028dd3-6ab9-4122-a8a0-a347be02bd28">
~~~
В налаштуваннях Solution Explorer -> Properties на закладці Build в поле Cultures to build додано uk-UA. Окрім того з репозиторію WiX скопійовано та додано до проекту файл WixUI_uk-UA.wxl.

Додав ряд макросів, щоб спростити звернення до найбільш вживаних значень:
~~~wxs
<?define ProductName="task1_CommonInfo" ?>
<?define ProductVersion="1.0.0.0" ?>
<?define ProductId="b7bc7c6f-9a4e-4973-be84-eca8e3427c97"?>
<?define ProductUpgradeCode="97028dd3-6ab9-4122-a8a0-a347be02bd28"?>
<?define ProductManufacturer="MyCompany"?>
~~~

GUID для Id та UpgradeCode згенерував за допомогою VS.   
Заповнив загальну інформацію про продукт
~~~wxs
<Product Id="$(var.ProductId)" Name="$(var.ProductName)" Language="1058" Version="$(var.ProductVersion)" Manufacturer="$(var.ProductManufacturer)" UpgradeCode="$(var.ProductUpgradeCode)">
~~~

Також додав до опису продукту опис версії інсталера, перевірку версії продукту (на випадок, якщо вже стоїть новіша). Структура інсталера залишається за замовчуванням
~~~wxs
    <!-- Package settings -->
    <Package InstallerVersion="200" Compressed="yes" />

    <!-- Used for upgrade -->
		<MajorUpgrade DowngradeErrorMessage="A newer version of [ProductName] is already installed." />

    <!-- Structure of file in the instaler -->
    <MediaTemplate />
~~~

Далі оголошується набір фіч (елементів), що будуть встановлюватись. В нашому випадку, це сам файл калькулятора та його іконка

~~~wxs
    <!-- Feature that must be installed -->
	<Feature Id="ProductFeature" Title="$(var.ProductName)" Level="1">
		<ComponentRef Id="CalculatorComponent" />
		<ComponentRef Id="ApplicationShortcutCalc" />
	</Feature>
~~~

І останнє з елементів продукту це UI, що буде використовуватись протягом інсталяції. Складається вона з оголошення змінної, що буде вказувати місце встановлення продукту для якого запускається UI, файлу ліцензії, що має при цьому підвантажитись (в даному випадку просто створив License.rtf в межах проекту та заповнив його довільним текстом) та конкретний сет UI компонентів, що мають використовуватись

~~~wxs
<!-- Setting up windows for installation process -->
<!-- Set variable that save installation path choosed by user -->
<Property Id="WIXUI_INSTALLDIR" Value="INSTALLLOCATION" ></Property>

<!-- License is neccessary when use windows from constructor -->
<WixVariable Id="WixUILicenseRtf" Overridable="yes" Value="License.rtf"/>

<!-- Use set of interface element for installation from WixUI_InstallDir that located in WixUIExtension.dll -->
<UIRef Id="WixUI_InstallDir"/>
~~~

Далі починається блок Fragment, який конкретизує, що за елементи необхідно інсталювати.   
Вказуємо кореневу папку інсталятора, з якої він буде шукати потрібні йому елементи

~~~wxs
<!-- Root folder of installation -->
<Directory Id="TARGETDIR" Name="SourceDir">
~~~

Тепер описуємо де взяти файл калькулятора, який ми маймо встановити, та власне вказуємо куди його треба інсталювати. В нашому випадку ми у ProgramFiles створюємо папку з назвою продукту (ця частина доречі зберігається та використовується як шлях для запуску нашого UI)

~~~wxs
<!-- Set location for main part of product -->
<!-- Reference to ProgramFiles folder -->
<Directory Id="ProgramFilesFolder">

    <!-- Create subdirectory in Program Files and set name as product name-->
	<Directory Id="INSTALLLOCATION" Name="$(var.ProductName)">

        <!-- Set component name which must be placed here -->
        <Component Id="CalculatorComponent" Guid="52793640-1944-4A19-9337-0B9760C54E95" KeyPath="yes">

            <!-- Set component element -->
            <File Id='Calc' Source='C:\WINDOWS\system32\calc.exe'/>
			    
        </Component> 
	</Directory>      
</Directory>
~~~

І залишилось лише описати де покласти іконку застосунку. В нашому випадку у меню Пуск - Всі програми - Назва нашого продукту - Іконка   
Окрім того додаємо в реєстр інформацію про стан інсталяції та кажемо, папка має бути видалена в разі видалення всього продукту

~~~wxs
<!-- Set shortcuts for our product -->
<!-- Directory where Start menu shortcuts are located -->
<Directory Id="ProgramMenuFolder">

    <!-- All Programs section of the Start menu -->
    <Directory Id="ApplicationProgramsFolder" Name="$(var.ProductName)">    

        <!-- Set component for shortcut (must be defined in feature) -->
        <Component Id="ApplicationShortcutCalc" Guid="7435E631-0BBD-4FDC-8B19-9C972DEB49D8">
            
            <!-- Set component element - in this case it shortcut -->
            <Shortcut Id="ShortcutCalc"
                 Name="Calc"
                 Description="$(var.ProductName)"
                 Target="[INSTALLLOCATION]Calc.exe"
                 WorkingDirectory="INSTALLLOCATION"/>
            
            <!-- Define that directory must be delete in case of uninstall product -->
            <RemoveFolder Id="ApplicationProgramsFolder" On="uninstall"/>
            
            <!-- Set in registry information about install status -->
            <RegistryValue Root="HKCU" Key="Software\$(var.ProductManufacturer)\$(var.ProductName)" Name="installed" Type="integer" Value="1" KeyPath="yes"/>

          </Component>
        </Directory>
      </Directory>
~~~

Робимо білд, запускаємо його та отримуємо наступний набір UI:   
Початок інсталяції:   
![img1](https://github.com/sotnikea/WiX_Learn/raw/main/task1_CommonInfo/img/img1.png)  

Ліцензійна угода:   
![img2](https://github.com/sotnikea/WiX_Learn/raw/main/task1_CommonInfo/img/img2.png) 

Шлях встановлення:   
![img3](https://github.com/sotnikea/WiX_Learn/raw/main/task1_CommonInfo/img/img1.png) 

Початок інсталяції:   
![img4](https://github.com/sotnikea/WiX_Learn/raw/main/task1_CommonInfo/img/img4.png) 

Завершення інсталяції:   
![img5](https://github.com/sotnikea/WiX_Learn/raw/main/task1_CommonInfo/img/img5.png) 

Результат:   
![img6](https://github.com/sotnikea/WiX_Learn/raw/main/task1_CommonInfo/img/img6.png) 