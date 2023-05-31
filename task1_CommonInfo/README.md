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

Сформував шлях для встановлення проекту за допомогою  Directory:
~~~wxs
<Fragment>
    <Directory Id="TARGETDIR" Name="SourceDir">
        <Directory Id="ProgramFilesFolder">
            <Directory Id="INSTALLFOLDER" Name="$(var.ProductName)" />
        </Directory>
    </Directory>
</Fragment>
~~~

Додав до ComponentGroup один єдиний компонент, яким в нашому випадку є калькулятор
~~~wxs
<Fragment>
    <ComponentGroup Id="ProductComponents" Directory="INSTALLFOLDER">
            <Component Id="CalculatorComponent" Guid="52793640-1944-4A19-9337-0B9760C54E95">
                <File Id='Calc' Source='C:\WINDOWS\system32\calc.exe'/>
            </Component> 
    </ComponentGroup>
</Fragment>
~~~

Технічно у нас лише один файл тож він у будь якому випадку має бути встановлений. Але тим не менш ми повинні зв'язати його з Feature 
~~~wxs
<Feature Id="ProductFeature" Title="$(var.ProductName)" Level="1">
	<ComponentGroupRef Id="CalculatorComponent" />
</Feature>
~~~

Додаємо ярлик програми у меню "Пуск":
~~~wxs
<Property Id="WIXUI_INSTALLDIR" Value="INSTALLLOCATION" ></Property>
<UIRef Id="WixUI_InstallDir"/>
~~~



