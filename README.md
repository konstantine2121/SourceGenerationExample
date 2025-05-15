# SourceGenerationExample

Для версии .NET 8.0

https://metanit.com/sharp/dotnet/4.1.php



----

### Определение генератора кода

Source Generator/генератор кода представляет класс, который реализует интерфейс Microsoft.CodeAnalysis.ISourceGenerator и применяет атрибут Microsoft.CodeAnalysis.GeneratorAttribute:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14|`using` `Microsoft.CodeAnalysis;`<br><br>`[Generator]`<br><br>`public` `class` `MySourceGenerator : ISourceGenerator`<br><br>`{`<br><br>    `public` `void` `Execute(GeneratorExecutionContext context)`<br><br>    `{`<br><br>        `throw` `new` `NotImplementedException();`<br><br>    `}`<br><br>    `public` `void` `Initialize(GeneratorInitializationContext context)`<br><br>    `{`<br><br>        `throw` `new` `NotImplementedException();`<br><br>    `}`<br><br>`}`|

интерфейс ISourceGenerator предоставляет два метода:

- Метод Initialize() выполняет инициализацию и вызывается перед генерацией кода. Его параметр `context` применяется для регистрации обработчиков событий, которые возникают при генерации кода
    
- Метод Execute() выполняет генерацию исходного кода. Его параметр `context` применяется для добавления файлов с исходным кодом
    

Рассмотрим создание и применение простейшего генератора кода.

### Создание библиотеки классов для Source Generators

Для хранения генератора кода создадим новый проект библиотеки классов, который, пусть называется GeneratorsLib.

Для работы с Source Generators прежде всего добавим в проект библиотеки классов Nuget-пакеты Microsoft.CodeAnalysis.Analyzers и Microsoft.CodeAnalysis.CSharp

Следует учитывать, что на данный момент генераторы кода поддерживаются только для целевого фреймворка "netstandard2.0". Поэтому изменим csproj-файл проекта следующим образом:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15|`<``Project` `Sdk``=``"Microsoft.NET.Sdk"``>`<br><br>  `<``PropertyGroup``>`<br><br>    `<``TargetFramework``>netstandard2.0</``TargetFramework``>`<br><br>  `</``PropertyGroup``>`<br><br>  `<``ItemGroup``>`<br><br>    `<``PackageReference` `Include``=``"Microsoft.CodeAnalysis.Analyzers"` `Version``=``"3.3.3"``>`<br><br>      `<``PrivateAssets``>all</``PrivateAssets``>`<br><br>      `<``IncludeAssets``>runtime; build; native; contentfiles; analyzers; buildtransitive</``IncludeAssets``>`<br><br>    `</``PackageReference``>`<br><br>    `<``PackageReference` `Include``=``"Microsoft.CodeAnalysis.CSharp"` `Version``=``"4.4.0"` `/>`<br><br>  `</``ItemGroup``>`<br><br>`</``Project``>`|

Теперь определим в библиотеке классов класс MySourceGenerator, который будет выполнять роль генератора кода:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5<br><br>6<br><br>7<br><br>8<br><br>9<br><br>10<br><br>11<br><br>12<br><br>13<br><br>14<br><br>15<br><br>16<br><br>17<br><br>18<br><br>19<br><br>20<br><br>21<br><br>22<br><br>23<br><br>24<br><br>25<br><br>26<br><br>27|`using` `Microsoft.CodeAnalysis;`<br><br>`namespace` `GeneratorsLib`<br><br>`{`<br><br>    `[Generator]`<br><br>    `public` `class` `MySourceGenerator : ISourceGenerator`<br><br>    `{`<br><br>        `public` `void` `Execute(GeneratorExecutionContext context)`<br><br>        `{`<br><br>            `// определяем генерируемый код`<br><br>            `var code =` `@"`<br><br>            `namespace Metanit`<br><br>            `{`<br><br>                `public static class Welcome`<br><br>                `{`<br><br>                    `public const string Name = ""Eugene"";`<br><br>                    `public static void Print() => Console.WriteLine($""Hello {Name}!"");`<br><br>                `}`<br><br>            `}"``;`<br><br>            `context.AddSource(``"metanit.welcome.generated.cs"``,code);`<br><br>        `}`<br><br>        `public` `void` `Initialize(GeneratorInitializationContext context)`<br><br>        `{`<br><br>            `// инициализация не нужна`<br><br>        `}`<br><br>    `}`<br><br>`}`|

Здесь в методе `Execute()` выполняем генерацию кода. Генерируемый код фактически представляет переменную code. В этом коде определяется стандартный код C# - в пространстве имен Metanit располагается статический класс Welcome, который имеет метод Print. Метод Print выводит на консоль приветствие.

Чтобы этот код принял участие в компиляции, у объекта context вызываем метод AddSource(). Первый параметр этого метода представляет уникальное имя файла, который будет генерироваться. Обычно он завершается на ".g.cs" или на ".generated.cs". Второй параметр представляет строку кода.

Стоит отметить, что в данном случае не применяется инициализация, поэтому метод Initialize пуст.

### Применение Source Generator

Теперь применим выше определенную библиотеку классов. Для этого возьмем проект консольного приложения и в него подключим библиотеку классов. В моем случае, например, оба проекта находятся в одном решении.

После добавления ссылки на проект библиотеки классов csproj-файл консольного проекта должен содержать ссылку на проект в следующем виде:

|   |   |
|---|---|
|1<br><br>2<br><br>3|`<``ItemGroup``>`<br><br>    `<``ProjectReference` `Include``=``"..\Путь_к_проекту_библиотеки_классов\GeneratorsLib.csproj"``/>`<br><br>`</``ItemGroup``>`|

Изменим этот код следующим образом:

|   |   |
|---|---|
|1<br><br>2<br><br>3<br><br>4<br><br>5|`<``ItemGroup``>`<br><br>    `<``ProjectReference` `Include``=``"..\Путь_к_проекту_библиотеки_классов\GeneratorsLib.csproj"`<br><br>                        `OutputItemType``=``"Analyzer"`<br><br>                        `ReferenceOutputAssembly``=``"false"` `/>`<br><br>`</``ItemGroup``>`|

В данном случае у элемента `ProjectReference` устанавливаются два дополнительных атрибута. Атрибут OutputItemType указывает на тип проекта - он должен иметь значение `"Analyzer"`. А атрибут ReferenceOutputAssembly получает значение `false`, которое указывает, что скомпилированное консольное приложение не будет содержать ссылку на этот проект библиотеки классов.

И в конце изменим в проекте консольного приложения файл Program.cs следующим образом:

|   |   |
|---|---|
|1|`Metanit.Welcome.Print();`|

То есть наш генератор генерирует пространство имен Metanit, в котором есть класс Welcome с методом Print. И в данном случае мы вызываем этот метод. Построим проект и запустим консольный проект на выполнение, и будет выполняться метод `Metanit.Welcome.Print()` из сгенерированного кода:

Hello Eugene!

(Если Visual Stuidio подчеркивает строку кода как некорректную, то лучше выполнить построение проекта и перезапустить Visual Studio).

Если используется Visual Studio, то мы можем увидеть сгенерированные файлы. Так, в структуре проекта перейдем к узлу Dependencies -> Analyzers -> GeneratorsLib и там мы сможем увидеть сгенерированный файл и посмотреть его содержимое.

(Я у себя их не вижу - но проект запускается и билдится нормально)