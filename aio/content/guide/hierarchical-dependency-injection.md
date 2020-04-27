{@a hierarchical-injectors}
# Иерархические инжекторы

У инжекторов в Angular есть правила, которые вы можете использовать
добиться желаемой видимости инъекций в ваших приложениях.
Поняв эти правила, вы сможете определить, в каких
NgModule, Component или Directive вы должны объявить провайдером.

{@a two-injector-hierarchies}
## Две иерархии инжекторов

Есть два инжекторных иерархий в угловом:

1. `ModuleInjector` Иерархия - настроить `ModuleInjector` 
в этой иерархии, используя `@NgModule()` или `@Injectable()` аннотация.
1. `ElementInjector` Иерархия - создается неявно на каждом
Элемент DOM. `ElementInjector` пуст по умолчанию
если вы не настроите его в `providers` собственность
 `@Directive() ` или ` @Component()`.

{@a register-providers-injectable}

{@a moduleinjector}
### `ModuleInjector` 

 `ModuleInjector` может быть сконфигурирован в одном из двух способов:

* С использованием `@Injectable()` `providedIn` собственность
Ссылаться на `@NgModule()` или `root`.
* С использованием `@NgModule()` `providers` массив.

<div class="is-helpful alert">

<h4>Тряска деревьев и др <code>@Injectable()</code></h4>

С использованием `@Injectable()` `providedIn` свойство является предпочтительным
к `@NgModule()` `providers` 
массив, потому что с `@Injectable()` `providedIn` оптимизации
инструменты могут выполнять
встряхивание дерева, которое удаляет сервисы, которыми не является ваше приложение
использование и приводит к меньшим размерам пучков.

Тряска деревьев особенно полезна для библиотеки
потому что приложение, которое использует библиотеку, может не иметь
необходимость ввести его. Подробнее
о [древовидные провайдеры](guide/dependency-injection-providers#tree-shakable-providers)
в [провайдеры DI](guide/dependency-injection-providers).

</div>

 `ModuleInjector ` настраивается ` @NgModule.providers` и
 `NgModule.imports ` . ` ModuleInjector` является уплощением
все массивы провайдеров, к которым можно обратиться, следуя
 `NgModule.imports` рекурсивно.

ребенок `ModuleInjector` создаются при ленивой загрузке других `@NgModules`.

Предоставлять услуги с `providedIn` собственность `@Injectable()` следующим образом :

```ts

import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root' // <--provides this service in the root ModuleInjector
})
export class ItemService {
  name = 'telephone';
}

```

 `@Injectable()` определяет класс обслуживания.
 `providedIn` свойство настраивает конкретный `ModuleInjector`,
Вот `root`, что делает сервис доступным в `root` `ModuleInjector`.

{@a platform-injector}
#### Платформа инжектора

Есть еще два инжектора выше `root`,
дополнительный `ModuleInjector` и `NullInjector()`.

Посмотрите, как Angular загружает приложение с помощью
следуя в `main.ts` :

```javascript
platformBrowserDynamic().bootstrapModule(AppModule).then(ref => {...})
```

 `bootstrapModule()` Метод создает дочерний инжектор
инжектор платформы, который настроен `AppModule`.
Это `root` `ModuleInjector`.

 `platformBrowserDynamic()` Метод создает инжектор
настроен `PlatformModule`, который содержит платформо-зависимые
зависимостей. Это позволяет нескольким приложениям совместно использовать платформу
конфигурации.
Например, браузер имеет только одну строку URL, независимо от того, как
много приложений у вас работает.
Вы можете настроить дополнительных провайдеров для конкретной платформы на
уровень платформы, поставляя `extraProviders` с использованием `platformBrowser()` Функция.

Следующим родительским инжектором в иерархии является `NullInjector()`,
которая является вершиной дерева. Если ты зашел так далеко вверх по дереву
что вы ищете услугу в `NullInjector()`, вы будете
получить ошибку, если вы не использовали `@Optional()` потому что в конечном итоге
все заканчивается на `NullInjector()` и возвращает ошибку или
в случае `@Optional()`, `null` . Для получения дополнительной информации о
 `@Optional()`, см. [ `@Optional ()` раздел](guide/hierarchical-dependency-injection#optional)этого руководства.

Следующая диаграмма представляет отношения между
 `root` `ModuleInjector` и его родительские инжекторы как
предыдущие абзацы описывают.

<div class="lightbox">
  <img src="generated/images/guide/dependency-injection/injectors.svg" alt="NullInjector, ModuleInjector, root injector">
</div>

Пока название `root` - это специальный псевдоним, другой `ModuleInjector` s
нет псевдонимов У вас есть возможность создать `ModuleInjector` s
всякий раз, когда динамически загружаемый компонент создается, например, с
Маршрутизатор, который создаст ребенка `ModuleInjector` s.

Все запросы перенаправляются вплоть до корневого инжектора, независимо от того, настроили ли вы его
с `bootstrapModule()` метод, или зарегистрировал всех провайдеров
с `root` в собственных сервисах.

<div class="alert is-helpful">

<h4><code>@Injectable()</code>против <code>@NgModule()</code></h4>

Если вы настраиваете провайдера для всего приложения в `@NgModule()` из
 `AppModule`, он переопределяет настроенный для `root` в
 `@Injectable()` метаданные. Вы можете сделать это, чтобы настроить
нестандартный поставщик сервиса, который используется несколькими приложениями.

Вот пример случая, когда компонент маршрутизатора
Конфигурация включает в себя
не по умолчанию [стратегия размещения](guide/router#location-strategy)
перечисляя своего провайдера
в `providers` список `AppModule`.

<code-example path="dependency-injection-in-action/src/app/app.module.ts" region="providers" header="src/app/app.module.ts (providers)">

</code-example>

</div>

{@a elementinjector}
### `ElementInjector` 

Angular создает `ElementInjector` s неявно для каждого элемента DOM.

Предоставление услуги в `@Component()` с помощью декоратора
его `providers` или `viewProviders` 
свойство настраивает `ElementInjector`.
Например, следующее `TestComponent` настраивает `ElementInjector` 
предоставляя услугу следующим образом :

```ts
@Component({
  ...
  providers: [{ provide: ItemService, useValue: { name: 'lamp' } }]
})
export class TestComponent

```

<div class="alert is-helpful">

**Примечание:** пожалуйста, смотрите
[правила разрешения](guide/hierarchical-dependency-injection#resolution-rules)
раздел, чтобы понять связь между `ModuleInjector` дерево и
 `ElementInjector` дерево.

</div>


Когда вы предоставляете услуги в компоненте, эта услуга доступна через
 `ElementInjector` в этом экземпляре компонента.
Это также может быть видно на
дочерний компонент / директивы, основанные на правилах видимости, описанных в разделе [правила разрешения](guide/hierarchical-dependency-injection#resolution-rules).

Когда экземпляр компонента уничтожается, то же самое происходит и с экземпляром службы.

{@a @directive-and-@component}
#### `@Directive() ` и ` @Component()` 

Компонент - это особый тип директивы, что означает это
как только `@Directive()` имеет `providers` собственность, `@Component()` тоже.
Это означает, что директивы, а также компоненты могут быть настроены
провайдеры, используя `providers` собственность.
Когда вы настраиваете поставщика для компонента или директивы
с использованием `providers` собственность
этот поставщик принадлежит `ElementInjector` этого компонента или
директивы.
Компоненты и директивы на одном элементе совместно используют инжектор.


{@a resolution-rules}

{@a resolution-rules}
## Правила разрешения

При разрешении токена для компонента / директивы Angular
решает его в два этапа:

1. Против `ElementInjector` Иерархия (его родители)
1. Против `ModuleInjector` иерархия (его родители)

Когда компонент объявляет зависимость, Angular пытается удовлетворить это
зависимость со своим `ElementInjector`.
Если у инжектора компонента отсутствует поставщик, он передает запрос
до его родительского компонента `ElementInjector`.

Запросы продолжаются до тех пор, пока Angular не найдет инжектор, который сможет
обработать запрос или не хватает предка `ElementInjector` s.

Если Angular не находит провайдера в каком-либо `ElementInjector` с,
это возвращается к элементу, где запрос возник и смотрит
в `ModuleInjector` иерархия.
Если Angular все еще не находит провайдера, он выдает ошибку.

Если вы зарегистрировали провайдера для того же токена DI по адресу
на разных уровнях, первый - Angular
он использует для разрешения зависимости. Если, например, провайдер
зарегистрирован локально в компоненте, который нуждается в обслуживании
Angular не ищет другого поставщика той же услуги.


{@a resolution-modifiers}
## Модификаторы разрешения

Поведение разрешения Angular можно изменить с помощью `@Optional()`, `@Self()`,
 `@SkipSelf() ` и ` @Host()` . Импортируйте каждый из них из `@angular/core` 
и использовать каждый в конструкторе класса компонента, когда вы внедряете свой сервис.

Для работающего приложения, демонстрирующего модификаторы разрешения
этот раздел охватывает, см. <live-example name="resolution-modifiers">пример модификаторов разрешения </live-example>.

{@a types-of-modifiers}
### Типы модификаторов

Модификаторы разрешения делятся на три категории:

1. Что делать, если Angular не находит того, кем ты являешься
ищу, то есть `@Optional()` 
2. С чего начать искать, то есть `@SkipSelf()` 
3. Где перестать смотреть, `@Host()` и `@Self()` 

По умолчанию Angular всегда начинается с текущего `Injector` и держит
поиск до конца. Модификаторы позволяют изменить старт
(самостоятельно) или конечное местоположение.

Кроме того, вы можете комбинировать все модификаторы, кроме `@Host()` и `@Self()` и конечно `@SkipSelf()` и `@Self()`.

{@a optional}

{@a @optional}
### `@Optional()` 

 `@Optional()` позволяет Angular считать сервис, вами, необязательным.
Таким образом, если это не может быть решено во время выполнения, Angular просто
разрешает службу как `null`, а не выбрасывать ошибку. В
следующий пример, сервис, `OptionalService`, не предоставляется в
сервис, `@NgModule()` или класс компонента, поэтому он недоступен
в любом месте приложения.

<code-example path="resolution-modifiers/src/app/optional/optional.component.ts" header="resolution-modifiers/src/app/optional/optional.component.ts" region="optional-component">

</code-example>


{@a @self}
### `@Self()` 

использование `@Self()` чтобы Angular смотрел только на `ElementInjector` для текущего компонента или директивы.

Хороший вариант использования для `@Self()` должен внедрить сервис, но только если это так
доступно на текущем элементе хоста. Чтобы избежать ошибок в этой ситуации
скомбинировать `@Self()` с `@Optional()`.

Например, в следующем `SelfComponent`, уведомление
впрыснутый `LeafService` in
конструктор

<code-example path="resolution-modifiers/src/app/self-no-data/self-no-data.component.ts" header="resolution-modifiers/src/app/self-no-data/self-no-data.component.ts" region="self-no-data-component">

</code-example>

В этом примере есть родительский поставщик и внедряющий
сервис вернет значение, впрыскивая сервис
с `@Self()` и `@Optional()` вернется `null` из - за
 `@Self()` сообщает инжектору прекратить поиск в текущем
хост-элемент.

Другой пример показывает класс компонента с поставщиком
для `FlowerService` . В этом случае инжектор выглядит не дальше
чем текущий `ElementInjector` потому что он находит `FlowerService` и возвращает желтый цветок.


<code-example path="resolution-modifiers/src/app/self/self.component.ts" header="resolution-modifiers/src/app/self/self.component.ts" region="self-component">

</code-example>

{@a @skipself}
### `@SkipSelf()` 

 `@SkipSelf()` является противоположностью `@Self()` . С `@SkipSelf()`, Angular
начинает поиск службы в родительском `ElementInjector`, а не
в текущем. Так что если родитель `ElementInjector` использовали значение `🌿` (папоротник)
для `emoji`, но у тебя было `🍁` (кленовый лист) в компоненте `providers` массив
Angular будет игнорировать `🍁` (кленовый лист) и использовать `🌿` (папоротник).

Чтобы увидеть это в коде, предположим, что следующее значение для `emoji` является то, что родительский компонент использует, как в этой услуге:

<code-example path="resolution-modifiers/src/app/leaf.service.ts" header="resolution-modifiers/src/app/leaf.service.ts" region="leafservice">

</code-example>

Представьте, что в дочернем компоненте у вас было другое значение, `🍁` (кленовый лист), но вместо этого вы хотели использовать значение родителя. Это когда вы будете использовать `@SkipSelf()` :

<code-example path="resolution-modifiers/src/app/skipself/skipself.component.ts" header="resolution-modifiers/src/app/skipself/skipself.component.ts" region="skipself-component">

</code-example>

В этом случае значение, которое вы получите за `emoji` будет `🌿` (папоротник), не `🍁` (кленовый лист).

{@a @skipself-with-@optional}
#### `@SkipSelf() ` с ` @Optional()` 

использование `@SkipSelf()` с `@Optional()` чтобы предотвратить ошибку, если значение `null` . В следующем примере `Person` сервис вводится в конструктор. `@SkipSelf()` говорит Angular пропустить текущий инжектор и `@Optional()` предотвратит ошибку, если `Person` сервис будет `null`.

``` ts
class Person {
  constructor(@Optional() @SkipSelf() parent?: Person) {}
}
```

{@a @host}
### `@Host()` 

 `@Host()` позволяет назначить компонент последней остановкой в ​​дереве инжекторов при поиске поставщиков. Даже если в дереве есть экземпляр сервиса, Angular не будет продолжать искать. использование `@Host()` следующим образом :

<code-example path="resolution-modifiers/src/app/host/host.component.ts" header="resolution-modifiers/src/app/host/host.component.ts" region="host-component">

</code-example>


поскольку `HostComponent` имеет `@Host()` в своем конструкторе, нет
независимо от того, что родитель `HostComponent` может иметь как
 `flower.emoji` Значение
 `HostComponent` будет использовать `🌼` (желтый цветок).


{@a logical-structure-of-the-template}
## Логическая структура шаблона

Когда вы предоставляете услуги в классе компонентов, услуги
видимый внутри `ElementInjector` дерево относительно где
и как вы предоставляете эти услуги.

Понимание основной логической структуры Angular
Шаблон даст вам основу для настройки сервисов
и в свою очередь контролировать их видимость.

Компоненты используются в шаблонах, как показано в следующем примере:

```
<app-root>
    <app-child></app-child>
</app-root>
```

<div class="alert is-helpful">

**Примечание: как** правило, вы объявляете компоненты и их
шаблоны в отдельных файлах. Для целей понимания
Как работает система впрыска, полезно взглянуть на них
с точки зрения комбинированного логического дерева. Срок
логически отличает его от дерева рендеринга (ваше приложение
дерево DOM). Чтобы отметить места, где компонент
шаблоны расположены, это руководство использует `<#VIEW>` 
псевдоэлемент, который на самом деле не существует в дереве визуализации
и присутствует только в целях ментальной модели.

</div>

Ниже приведен пример того, как `<app-root>` и `<app-child>` вид деревья объединяются в одно логическое дерево:

```
<app-root>
  <#VIEW>
    <app-child>
     <#VIEW>
       ...content goes here...
     </#VIEW>
    </app-child>
  <#VIEW>
</app-root>
 ```

Понимание идеи `<#VIEW>` особенно важно при настройке служб в классе компонентов.

{@a providing-services-in-@component}
## Предоставление услуг в `@Component()` 

Как вы предоставляете услуги через `@Component()` (или `@Directive()` )
Декоратор определяет их видимость. Следующие разделы
демонстрировать `providers` и `viewProviders` вместе со способами
изменить видимость сервиса с `@SkipSelf()` и `@Host()`.

Компонент класса может предоставлять услуги по двум направлениям:

1. с `providers` массив

```typescript=
@Component({
  ...
  providers: [
    {provide: FlowerService, useValue: {emoji: '🌺'}}
  ]
})
```

2. с `viewProviders` массив

```typescript=
@Component({
  ...
  viewProviders: [
    {provide: AnimalService, useValue: {emoji: '🐶'}}
  ]
})
```

Чтобы понять, как `providers` и `viewProviders` влияют на сервис
видимость по-разному, следующие разделы построить
а <live-example name="providers-viewproviders"></live-example>
пошагово и сравните использование `providers`  и  `viewProviders` 
в коде и логическом дереве.

<div class="alert is-helpful">

**ПРИМЕЧАНИЕ.** В логическом дереве вы увидите  `@Provide`, `@Inject`, и
 `@NgModule`, которые не являются настоящими HTML-атрибутами, но здесь для демонстрации
что происходит под капотом.

-  `@Inject(Token)=>Value` показывает, что если  `Token`  вводится в
это место в логическом дереве его значение будет  `Value`.
-  `@Provide(Token=Value)` демонстрирует, что существует объявление
 `Token` провайдер со значением  `Value`  в этом месте в логическом дереве.
-  `@NgModule(Token)` демонстрирует, что запасной вариант  `NgModule`  Инжектор
следует использовать в этом месте.

</div>


{@a example-app-structure}
### Пример структуры приложения

В примере приложения есть  `FlowerService`  предоставляется в  `root`  с  `emoji` 
ценность  `ðŸŒº`  ( (красный гибискус).

<code-example path="providers-viewproviders/src/app/flower.service.ts" header="providers-viewproviders/src/app/flower.service.ts" region="flowerservice">

</code-example>

Рассмотрим простое приложение только с  `AppComponent`  и  `ChildComponent`.
Самый базовый представленный вид будет выглядеть как вложенные элементы HTML, такие как
следующее:

```
<app-root> <!-- AppComponent selector -->
    <app-child> <!-- ChildComponent selector -->
    </app-child>
</app-root>
```

Однако за кулисами Angular использует логическое представление
представление следующим образом при разрешении запросов инъекций:

```
<app-root> <!-- AppComponent selector -->
    <#VIEW>
        <app-child> <!-- ChildComponent selector -->
            <#VIEW>
            </#VIEW>
        </app-child>
    </#VIEW>
</app-root>
 ```

 `<#VIEW>` здесь представляет экземпляр шаблона.
Обратите внимание, что у каждого компонента есть свой  `<#VIEW>`.

Знание этой структуры может сообщить, как вы предоставляете и
внедрить ваши услуги и дать вам полный контроль над видимостью услуг.

Теперь посмотрим, что  `<app-root>`  просто внедряет  `FlowerService`  :


<code-example path="providers-viewproviders/src/app/app.component.1.ts" header="providers-viewproviders/src/app/app.component.ts" region="injection">

</code-example>

Добавьте привязку к  `<app-root>`  шаблон для визуализации результата:

<code-example path="providers-viewproviders/src/app/app.component.html" header="providers-viewproviders/src/app/app.component.html" region="binding-flower">

</code-example>


Выход в представлении будет:

```
Emoji from FlowerService: 🌺
```

В логическом дереве, это будет представлено следующим образом :

```
<app-root @NgModule(AppModule)
        @Inject(FlowerService) flower=>"🌺">
  <#VIEW>
    <p>Emoji from FlowerService: {{flower.emoji}} (🌺)</p>
    <app-child>
      <#VIEW>
      </#VIEW>
     </app-child>
  </#VIEW>
</app-root>
```

когда  `<app-root>`  запрашивает  `FlowerService`, это работа инжектора
разрешить  `FlowerService`  Токен . Разрешение токена происходит
в два этапа:

1. Инжектор определяет начальное местоположение в логическом дереве и
конечное местоположение поиска. Инжектор начинается со старта
местоположение и ищет токен на каждом уровне в логическом дереве. Если
токен найден, он возвращен.
2. Если токен не найден, инжектор ищет ближайший
родитель  `@NgModule()`  для делегирования запроса.

В случае примера, ограничения являются:

1. Начать с  `<#VIEW>`  принадлежащий  `<app-root>`  и заканчивается  `<app-root>`.

  - Обычно отправной точкой для поиска является точка
  впрыска. Однако в этом случае  `<app-root>`     `@Component` с
  особенным в том, что они также включают свои  `viewProviders`,
  поэтому поиск начинается с  `<#VIEW>`  принадлежащий  `<app-root>`.
  (Это не относится к директиве, совпадающей в том же месте).
  - Конечное местоположение просто совпадает с компонентом
  сам, потому что это самый верхний компонент в этом приложении.

2. The  `AppModule`  действует как запасной инжектор, когда
токен инъекции не найден в  `ElementInjector`  s.

{@a using-the-providers-array}
### С использованием  `providers`  массив

Теперь в  `ChildComponent`  Класс, добавьте провайдера для  `FlowerService` 
продемонстрировать более сложные правила разрешения в следующих разделах:

<code-example path="providers-viewproviders/src/app/child/child.component.1.ts" header="providers-viewproviders/src/app/child.component.ts" region="flowerservice">

</code-example>

Теперь, когда  `FlowerService`  предоставляется в  `@Component()`  декоратор
когда  `<app-child>`  запрашивает сервис, инжектор остается только посмотреть
насколько  `<app-child>`  's's own  `ElementInjector`  . Это не должно будет
продолжить поиск через дерево инжекторов.

Следующим шагом является добавление привязки к  `ChildComponent`  Шаблон.

<code-example path="providers-viewproviders/src/app/child/child.component.html" header="providers-viewproviders/src/app/child.component.html" region="flower-binding">

</code-example>

Чтобы отобразить новые значения, добавьте  `<app-child>`  в нижней части
 `AppComponent` шаблон таким образом, представление также отображает подсолнечник:

```
Child Component
Emoji from FlowerService: 🌻
```

В логическом дереве, это будет представлено следующим образом :

```
<app-root @NgModule(AppModule)
        @Inject(FlowerService) flower=>"🌺">
  <#VIEW>
    <p>Emoji from FlowerService: {{flower.emoji}} (🌺)</p>
    <app-child @Provide(FlowerService="🌻")
               @Inject(FlowerService)=>"🌻"> <!-- search ends here -->
      <#VIEW> <!-- search starts here -->
        <h2>Parent Component</h2>
        <p>Emoji from FlowerService: {{flower.emoji}} (🌻)</p>
      </#VIEW>
     </app-child>
  </#VIEW>
</app-root>
```

когда  `<app-child>`  запрашивает  `FlowerService`, начинается инжектор
его поиск в  `<#VIEW>`  принадлежащий  `<app-child>`  (  `<#VIEW>`  есть
включен, потому что он вводится из  `@Component()`  ) и заканчивается на
 `<app-child>` . В этом случае  `FlowerService`  разрешается в
 `<app-child> ` 's ` providers` массив с подсолнухами ». Инжектор не делает
надо смотреть дальше в инжекторное дерево. Это останавливается, как только это
находит  `FlowerService`  и никогда не видит 🌻º (красный гибискус).


{@a use-view-providers}

{@a using-the-viewproviders-array}
### С использованием  `viewProviders`  массив

Использовать  `viewProviders`  Массив как еще один способ предоставления услуг в
 `@Component()` декоратор. С помощью  `viewProviders`  делает услуги
видимый в  `<#VIEW>`.

<div class="is-helpful alert">

Шаги такие же, как при использовании  `providers`  массив
за исключением использования  `viewProviders`  массив.

Для получения пошаговых инструкций перейдите к этому разделу. Если вы не можете
настройте его самостоятельно, перейдите к [Изменение доступности службы](guide/hierarchical-dependency-injection#modify-visibility).

</div>


В примере приложения есть второй сервис,  `AnimalService`  для
демонстрировать  `viewProviders`.

Сначала создайте  `AnimalService`  с  `emoji`  свойство Dy ?? ³ (кит)

<code-example path="providers-viewproviders/src/app/animal.service.ts" header="providers-viewproviders/src/app/animal.service.ts" region="animal-service">

</code-example>


Следуя той же схеме, что и с  `FlowerService`,
 `AnimalService ` в ` AppComponent` класс:

<code-example path="providers-viewproviders/src/app/app.component.ts" header="providers-viewproviders/src/app/app.component.ts" region="inject-animal-service">

</code-example>

<div class="alert is-helpful">

**Примечание:** вы можете оставить все  `FlowerService`  связанный код на месте
так как это позволит сравнение с  `AnimalService`.

</div>

Добавить  `viewProviders`  массив и внедрить  `AnimalService`  в
 `<app-child>` класс тоже, но дают  `emoji`  другое значение. Здесь
имеет значение 🐶 (щенок).


<code-example path="providers-viewproviders/src/app/child/child.component.ts" header="providers-viewproviders/src/app/child.component.ts" region="provide-animal-service">

</code-example>

Добавьте привязки к  `ChildComponent`  и  `AppComponent`  шаблоны.
в  `ChildComponent`  шаблон, добавьте следующие привязки:

<code-example path="providers-viewproviders/src/app/child/child.component.html" header="providers-viewproviders/src/app/child.component.html" region="animal-binding">

</code-example>

Кроме того, добавьте то же самое к  `AppComponent`  шаблон:

<code-example path="providers-viewproviders/src/app/app.component.html" header="providers-viewproviders/src/app/app.component.html" region="binding-animal">

</code-example>

Теперь вы должны увидеть оба значения в браузере:

```
AppComponent
Emoji from AnimalService: 🐳

Child Component
Emoji from AnimalService: 🐶

```

Логическое дерево для этого примера  `viewProviders`  выглядит следующим образом :


```
<app-root @NgModule(AppModule)
        @Inject(AnimalService) animal=>"🐳">
  <#VIEW>
    <app-child>
      <#VIEW
       @Provide(AnimalService="🐶")
       @Inject(AnimalService=>"🐶")>
       <!-- ^^using viewProviders means AnimalService is available in <#VIEW>-->
       <p>Emoji from AnimalService: {{animal.emoji}} (🐶)</p>
      </#VIEW>
     </app-child>
  </#VIEW>
</app-root>
```

Так же, как с  `FlowerService`, `AnimalService`  предоставляется
в  `<app-child>`    `@Component()` декоратор. Это значит что со времен
Впервые инжектор заглядывает в  `ElementInjector`  компонента, он находит
 `AnimalService` значение 🐶 (щенок). Не нужно продолжать поиск
 `ElementInjector`, и не нужно искать  `ModuleInjector`.

{@a providers-vs.-viewproviders}
###  `providers ` против ` viewProviders` 

Чтобы увидеть разницу между использованием  `providers`  и  `viewProviders`, доп
другой компонент к примеру и назовите его  `InspectorComponent`.
 `InspectorComponent` будет дочерним  `ChildComponent`  . В
 `inspector.component.ts`, введите  `FlowerService`  и  `AnimalService`  в
конструктор:


<code-example path="providers-viewproviders/src/app/inspector/inspector.component.ts" header="providers-viewproviders/src/app/inspector/inspector.component.ts" region="injection">

</code-example>

Вам не нужно  `providers`  или  `viewProviders`  массив . Далее в
 `inspector.component.html`, добавьте ту же разметку из предыдущих компонентов:

<code-example path="providers-viewproviders/src/app/inspector/inspector.component.html" header="providers-viewproviders/src/app/inspector/inspector.component.html" region="binding">

</code-example>

Не забудьте добавить  `InspectorComponent`  к  `AppModule`    `declarations` массив.

<code-example path="providers-viewproviders/src/app/app.module.ts" header="providers-viewproviders/src/app/app.module.ts" region="appmodule">

</code-example>


Затем убедитесь, что ваш  `child.component.html`  содержит следующее:

<code-example path="providers-viewproviders/src/app/child/child.component.html" header="providers-viewproviders/src/app/child/child.component.html" region="child-component">

</code-example>

Первые две строки, с привязками, взяты из предыдущих шагов.
новые запчасти   `<ng-content> ` и ` <app-inspector> ` . ` <ng-content>` позволяет
вы проектировать контент, и  `<app-inspector>`  внутри  `ChildComponent` 
шаблон делает  `InspectorComponent`  дочерний компонент
  `ChildComponent`.

Затем добавьте следующее  `app.component.html`  чтобы воспользоваться преимуществами контент-проекции.

<code-example path="providers-viewproviders/src/app/app.component.html" header="providers-viewproviders/src/app/app.component.html" region="content-projection">

</code-example>

Браузер теперь отображает следующее, опуская предыдущие примеры
для краткости:

```

//...Omitting previous examples. The following applies to this section.

Content projection: This is coming from content. Doesn't get to see
puppy because the puppy is declared inside the view only.

Emoji from FlowerService: 🌻
Emoji from AnimalService: 🐳

Emoji from FlowerService: 🌻
Emoji from AnimalService: 🐶

```

Эти четыре привязки демонстрируют разницу между  `providers` 
и  `viewProviders`  . Так как 🐶 (щенок) объявляется внутри < #VIEW>,
он не виден для проецируемого контента. Вместо этого прогнозируется
Контент видит 🐳 (кит).

Следующий раздел, хотя, где  `InspectorComponent`  является дочерним компонентом
из  `ChildComponent`, `InspectorComponent`  находится внутри  `<#VIEW>`, так
когда он просит  `AnimalService`, он видит 🐶 (щенок).

 `AnimalService` в логическом дереве будет выглядеть следующим образом :

```
<app-root @NgModule(AppModule)
        @Inject(AnimalService) animal=>"🐳">
  <#VIEW>
    <app-child>
      <#VIEW
       @Provide(AnimalService="🐶")
       @Inject(AnimalService=>"🐶")>
       <!-- ^^using viewProviders means AnimalService is available in <#VIEW>-->
       <p>Emoji from AnimalService: {{animal.emoji}} (🐶)</p>
       <app-inspector>
        <p>Emoji from AnimalService: {{animal.emoji}} (🐶)</p>
       </app-inspector>
      </#VIEW>
      <app-inspector>
        <#VIEW>
          <p>Emoji from AnimalService: {{animal.emoji}} (🐳)</p>
        </#VIEW>
      </app-inspector>
     </app-child>
  </#VIEW>
</app-root>
```

Прогнозируемое содержание  `<app-inspector>`  видит 🐳 (кит), а не
🐶 (щенок), потому что
🐶 (щенок) находится внутри  `<app-child>`    `<#VIEW>` .  `<app-inspector>` может
только см. 🐶 (щенок)
если это также в пределах  `<#VIEW>`.

{@a modify-visibility}

{@a modifying-service-visibility}
## Изменение видимости сервиса

В этом разделе описывается, как ограничить рамки начала и
окончание  `ElementInjector`  с использованием декораторов видимости  `@Host()`,
 `@Self() ` и ` @SkipSelf()`.

{@a visibility-of-provided-tokens}
### Видимость предоставленных токенов

Видимость декораторов влияет на то, где происходит поиск инъекции
токен начинается и заканчивается в дереве логики. Для этого место
видимость декораторов в точке впрыска, то есть
 `constructor()`, а не в точке объявления.

Чтобы изменить, где инжектор начинает искать  `FlowerService`, добавить
 `@SkipSelf() ` для ` <app-child>`   `@Inject` декларация для
 `FlowerService` . Эта декларация находится в  `<app-child>`  конструктор
как показано в  `child.component.ts`  :

```typescript=
  constructor(@SkipSelf() public flower : FlowerService) { }
```

С  `@SkipSelf()`, `<app-child>`  инжектор не ищет для себя
 `FlowerService` . Вместо этого инжектор начинает искать
 `FlowerService ` на ` <app-root> ` «s ` ElementInjector`, где он находит
ничего. Затем он возвращается к  `<app-child>`    `ModuleInjector` и находит
значение 🌺 (красный гибискус), которое доступно, потому что  `<app-child>` 
 `ModuleInjector ` и ` <app-root>`   `ModuleInjector` в одну
  `ModuleInjector` . Таким образом, пользовательский интерфейс делает следующее:

```
Emoji from FlowerService: 🌺
```

В логическом дереве, эта же идея может выглядеть следующим образом :

```
<app-root @NgModule(AppModule)
        @Inject(FlowerService) flower=>"🌺">
  <#VIEW>
    <app-child @Provide(FlowerService="🌻")>
      <#VIEW @Inject(FlowerService, SkipSelf)=>"🌺">
      <!-- With SkipSelf, the injector looks to the next injector up the tree -->
      </#VIEW>
      </app-child>
  </#VIEW>
</app-root>
```

Хоть  `<app-child>`  предоставляет 🌻 (подсолнух), приложение отображает
(ŸŒº (красный гибискус), потому что  `@SkipSelf()`    вызывает ток
инжектор пропустить
сам и посмотри на своего родителя.

Если вы сейчас добавите  `@Host()`  (в дополнение к  `@SkipSelf()`  ) в
 `@Inject ` of ` FlowerService`, результат будет  `null`  . Это
потому что  `@Host()`  ограничивает верхнюю границу поиска до
 `<#VIEW>` . Вот идея в логическом дереве:

```
<app-root @NgModule(AppModule)
        @Inject(FlowerService) flower=>"🌺">
  <#VIEW> <!-- end search here with null-->
    <app-child @Provide(FlowerService="🌻")> <!-- start search here -->
      <#VIEW @Inject(FlowerService, @SkipSelf, @Host, @Optional)=>null>
      </#VIEW>
      </app-parent>
  </#VIEW>
</app-root>
```

Здесь услуги и их значения одинаковы, но  `@Host()` 
мешает инжектору смотреть дальше, чем  `<#VIEW>` 
для  `FlowerService`, поэтому он не находит его и возвращает  `null`.

<div class="alert is-helpful">

**Примечание:** пример приложения использует  `@Optional()`  так приложение делает
не выкинуть ошибку, но принципы те же.

</div>

{@a @skipself-and-viewproviders}
###  `@SkipSelf() ` и ` viewProviders` 

 `<app-child>` настоящее время предоставляет  `AnimalService`  в
 `viewProviders` Массив со значением 🐶 (puppy). Потому что
инжектор должен только смотреть на  `<app-child>`  's  `ElementInjector` 
для  `AnimalService`, он никогда не видит 🐳 (кит).

Так же, как в  `FlowerService`, если вы добавите  `@SkipSelf()` 
конструктору для  `AnimalService`, инжектор не будет
смотри в течении  `<app-child>`  's  `ElementInjector`  для
 `AnimalService`.

```typescript=
export class ChildComponent {

// add @SkipSelf()
  constructor(@SkipSelf() public animal : AnimalService) { }

}
```

Вместо этого инжектор начнется в  `<app-root>` 
 `ElementInjector` . Помните, что  `<app-child>`  класс
обеспечивает  `AnimalService`  в  `viewProviders`  массив
со значением 🐶 (щенок):

```ts
@Component({
  selector: 'app-child',
  ...
  viewProviders:
  [{ provide: AnimalService, useValue: { emoji: '🐶' } }]
})
```

Логическое дерево выглядит так с  `@SkipSelf()`  в  `<app-child>`  :

```
  <app-root @NgModule(AppModule)
          @Inject(AnimalService=>"🐳")>
    <#VIEW><!-- search begins here -->
      <app-child>
        <#VIEW
         @Provide(AnimalService="🐶")
         @Inject(AnimalService, SkipSelf=>"🐳")>
         <!--Add @SkipSelf -->
        </#VIEW>
        </app-child>
    </#VIEW>
  </app-root>
```

С  `@SkipSelf()`  в  `<app-child>`, инжектор начинает свое
искать  `AnimalService`  в  `<app-root>`    `ElementInjector` 
и находит 🐳 (кит).

{@a @host-and-viewproviders}
###  `@Host() ` и ` viewProviders` 

Если вы добавите  `@Host()`  в конструктор для  `AnimalService`, то
результат - 🐶 (щенок), потому что инжектор находит  `AnimalService` 
в  `<app-child>`    `<#VIEW>` . Здесь  `viewProviders`  массив
в  `<app-child>`  класс и  `@Host()`  в конструкторе:

```typescript=
@Component({
  selector: 'app-child',
  ...
  viewProviders:
  [{ provide: AnimalService, useValue: { emoji: '🐶' } }]

})
export class ChildComponent {
  constructor(@Host() public animal : AnimalService) { }
}
```

 `@Host()` заставляет инжектор смотреть, пока он не края  `<#VIEW>`.

```
  <app-root @NgModule(AppModule)
          @Inject(AnimalService=>"🐳")>
    <#VIEW>
      <app-child>
        <#VIEW
         @Provide(AnimalService="🐶")
         @Inject(AnimalService, @Host=>"🐶")> <!-- @Host stops search here -->
        </#VIEW>
        </app-child>
    </#VIEW>
  </app-root>
```

Добавить  `viewProviders`  массив с третьим животным, ,Ÿ¦ (еж), к
 `app.component.ts`   `@Component()` метаданные:

```typescript
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: [ './app.component.css' ],
  viewProviders: [{ provide: AnimalService, useValue: { emoji: '🦔' } }]
})
```

Затем добавьте  `@SkipSelf()`  вместе с  `@Host()`  для конструктора для
 `Animal Service ` в ` child.component.ts` . Вот  `@Host()` 
и  `@SkipSelf()`  в  `<app-child>` 
Конструктор:

```ts
export class ChildComponent {

  constructor(
  @Host() @SkipSelf() public animal : AnimalService) { }

}
```

когда  `@Host()`  и  `SkipSelf()`  были применены к  `FlowerService`,
который находится в  `providers`  массив, результат был  `null`  из - за
 `@SkipSelf()` начинает поиск в  `<app-child>`  инжектор, но
 `@Host()` прекращает поиск на  `<#VIEW>` &mdash;там, где его нет
 `FlowerService` . В логическом дереве вы можете видеть, что
 `FlowerService` виден в  `<app-child>`, а не его  `<#VIEW>`.

Тем не менее  `AnimalService`, который предоставляется в
 `AppComponent`   `viewProviders` Массив, виден.

Логическое дерево представление показывает, почему это:

```html
<app-root @NgModule(AppModule)
        @Inject(AnimalService=>"🐳")>
  <#VIEW @Provide(AnimalService="🦔")
         @Inject(AnimalService, @SkipSelf, @Host, @Optional)=>"🦔">
    <!-- ^^@SkipSelf() starts here,  @Host() stops here^^ -->
    <app-child>
      <#VIEW @Provide(AnimalService="🐶")
             @Inject(AnimalService, @SkipSelf, @Host, @Optional)=>"🐶">
               <!-- Add @SkipSelf ^^-->
      </#VIEW>
      </app-child>
  </#VIEW>
</app-root>
```

 `@SkipSelf()`, заставляет инжектор начать свой поиск
 `AnimalService ` на ` <app-root>`, а не  `<app-child>`,
где запрос происходит, и  `@Host()`  останавливает поиск
на  `<app-root>`    `<#VIEW>` . поскольку  `AnimalService`  есть
предоставляется через  `viewProviders`  массив, инжектор находит «»
(ежик) в  `<#VIEW>`.


{@a component-injectors}

{@a elementinjector-use-case-examples}
##  `ElementInjector` использования

Возможность настройки одного или нескольких провайдеров на разных уровнях
открывает полезные возможности.
Для просмотра следующих сценариев в рабочем приложении см. <live-example>Примеры использования героев </live-example>.

{@a scenario-service-isolation}
### Сценарий: сервисная изоляция

Архитектурные причины могут привести к тому, что вы ограничите доступ к сервису доменом приложения, к которому он принадлежит.
Например, образец руководства включает  `VillainsListComponent`  который отображает список злодеев.
Это получает эти злодеи от  `VillainsService`.

Если вы предоставили  `VillainsService`  в корне  `AppModule` 
(где вы зарегистрировали  `HeroesService`)
это сделало бы  `VillainsService`  виден повсеместно
приложение, включая рабочие процессы _Hero_. Если ты позже
изменил  `VillainsService`, вы можете сломать что-то в
где-то герой

Вместо этого вы можете предоставить  `VillainsService`  в  `providers`  метаданные  `VillainsListComponent` как это:


<code-example path="hierarchical-dependency-injection/src/app/villains-list.component.ts" header="src/app/villains-list.component.ts (metadata)" region="metadata">

</code-example>

Предоставляя  `VillainsService`  в  `VillainsListComponent`  метаданные и больше нигде
услуга становится доступной только в  `VillainsListComponent`  и его подкомпонентное дерево.

 `VillainService` является в отношении  `VillainsListComponent` 
потому что это где это объявлено. Так долго как  `VillainsListComponent` 
не разрушится, это будет тот же экземпляр  `VillainService` 
но если есть несколько экземпляров  `VillainsListComponent`, затем каждый
случай  `VillainsListComponent`  будет иметь свой собственный экземпляр  `VillainService`.



{@a scenario-multiple-edit-sessions}
### Сценарий: несколько сеансов редактирования

Многие приложения позволяют пользователям работать над несколькими открытыми задачами одновременно.
Например, в заявлении на подготовку налогового отчета составитель может работать с несколькими налоговыми декларациями
переключение с одного на другое в течение дня.

Это руководство демонстрирует этот сценарий на примере темы «Тур героев».
Представьте себе внешнее  `HeroListComponent`  который отображает список супер героев.

Чтобы открыть налоговую декларацию героя, составитель нажимает на имя героя, что открывает компонент для редактирования этой декларации.
Каждая выбранная налоговая декларация героя открывается в своем собственном компоненте, и одновременно можно открыть несколько деклараций.

Каждая налоговая декларация компонент имеет следующие характеристики:

* Это собственный сеанс редактирования налоговой декларации.
* Может изменить налоговую декларацию, не влияя на возврат в другой компонент.
* Имеет возможность сохранить изменения в своей налоговой декларации или отменить их.

<div class="lightbox">
  <img src="generated/images/guide/dependency-injection/hid-heroes-anim.gif" alt="Heroes in action">
</div>

Предположим, что  `HeroTaxReturnComponent`  обладает логикой для управления и восстановления изменений.
Это было бы довольно простой задачей для простой налоговой декларации героя.
В реальном мире с богатой моделью данных налоговой декларации управление изменениями было бы сложным.
Вы можете делегировать это управление вспомогательной службе, как в этом примере.

 `HeroTaxReturnService` кэширует один  `HeroTaxReturn`  отслеживает изменения этого возврата и может сохранить или восстановить его.
Он также делегирует синглтон всего приложения  `HeroService`, который он получает инъекцией.


<code-example path="hierarchical-dependency-injection/src/app/hero-tax-return.service.ts" header="src/app/hero-tax-return.service.ts">

</code-example>

Здесь  `HeroTaxReturnComponent`  который использует  `HeroTaxReturnService`.


<code-example path="hierarchical-dependency-injection/src/app/hero-tax-return.component.ts" header="src/app/hero-tax-return.component.ts">

</code-example>


_Tax-return-to-edit_ приходит через  `@Input()`, которое реализуется с помощью методов получения и установки.
Сеттер инициализирует собственный экземпляр компонента  `HeroTaxReturnService`  с входящим возвратом.
Получатель всегда возвращает то, что говорит этот сервис, о текущем состоянии героя.
Компонент также просит сервис сохранить и восстановить эту налоговую декларацию.

Это не сработает, если служба является синглтоном для всего приложения.
Каждый компонент будет использовать один и тот же экземпляр службы, и каждый компонент будет перезаписывать налоговую декларацию, принадлежащую другому герою.

Чтобы предотвратить это, настройте инжектор уровня компонента  `HeroTaxReturnComponent`  для предоставления услуги, используя   `providers` Свойство в метаданных компонента.



<code-example path="hierarchical-dependency-injection/src/app/hero-tax-return.component.ts" header="src/app/hero-tax-return.component.ts (providers)" region="providers">

</code-example>

 `HeroTaxReturnComponent` имеет свой собственный поставщик  `HeroTaxReturnService`.
Напомним, что каждый компонент _instance_ имеет свой собственный инжектор.
Предоставление услуги на уровне компонента гарантирует, что _every_ экземпляр компонента получает свой собственный, частный экземпляр службы, и никакая налоговая декларация не перезаписывается.


<div class="alert is-helpful">

Остальная часть кода сценария опирается на другие функции и методы Angular, о которых вы можете узнать в других разделах документации.
Вы можете просмотреть его и загрузить с сайта <live-example></live-example>.

</div>



{@a scenario-specialized-providers}
### Сценарий: специализированные провайдеры

Еще одна причина для повторного предоставления сервиса на другом уровне - это замена более специализированной реализации этого сервиса, находящейся глубже в дереве компонентов.

Рассмотрим автомобильный компонент, который зависит от нескольких услуг.
Предположим, вы настроили корневой инжектор (помеченный как A) с _generic_ провайдерами для
 `CarService `, ` EngineService ` и ` TiresService`.

Вы создаете автомобильный компонент (A), который отображает автомобиль, созданный из этих трех общих служб.

Затем вы создаете дочерний компонент (B), который определяет своих собственных _specialized_ провайдеров для  `CarService`  и  `EngineService` 
которые имеют специальные возможности, подходящие для всего, что происходит в компоненте (B).

Компонент (B) является родителем другого компонента (C), который определяет свой собственный, даже более специализированный поставщик для  `CarService`.


<div class="lightbox">
  <img src="generated/images/guide/dependency-injection/car-components.png" alt="car components">
</div>

За кулисами каждый компонент устанавливает свой собственный инжектор с нулем, одним или несколькими поставщиками, определенными для этого компонента.

Когда вы решаете экземпляр  `Car`  на самом глубоком компоненте (С)
его инжектор производит экземпляр  `Car`  разрешенный инжектором (C) с  `Engine`  разрешен инжектором (B) и
 `Tires` разрешенные корневым инжектором (A).


<div class="lightbox">
  <img src="generated/images/guide/dependency-injection/injector-tree.png" alt="car injector tree">
</div>


<hr />

{@a more-on-dependency-injection}
## Больше о внедрении зависимости

Для получения дополнительной информации о внедрении Angular зависимости см. [Поставщики DI](guide/dependency-injection-providers)и [DI в действии](guide/dependency-injection-in-action)Руководства.
