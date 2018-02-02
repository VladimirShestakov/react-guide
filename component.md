#Компоненты {#top}

При декомпозции проекта выделяются повторно используемые компоненты. Такие компоненты абстрагированы
от бизнес логики приложения. В соответсвии с переданными свойствами выполняют рендер html
разметки и предоставляют обратную связь на различные действия (клики, ввод с клавиатуры и др.).

##Файловая структура {#file}

Повторно используемые компоненты создаются в директории `./components`. 
В больших проектах создаются поддиректории для группировки компонентов по назначению, например: 
элементы, формы, разметки, списки, меню.

```
components 
├── elements
|   ├── component-name 
|   ├── button
|   ├── icon
|   └── logo
├── forms
└── layouts
```

Каждый компонент в своей папке. Название папки в соответствии с названием класса компонента, но в 
нижнем регистре через дефис (kebbab стиль).

*Компонент ComponentName*
```
component-name 
├── index.js // допустимо component-name.js
└── style.less
```

- В `index.js` или `compoent-name.js` класс компонента. Расширение `.js`.
- В `style.less` стили компонента. По умолчанию используем `.less`, допустимы другие.

Название файлов и директорий всегда в нижнем регистре, чтобы избежать проблем с регистром на разных 
платформах. Именование `index.js` рекомендуется для быстроты копипастов, единства внутренней структуры, коротких 
импортов. Именование `component-name.js` если в IDE/отладке неудобно различать файлы или `index.js` нужен для 
реэкспорта.

В директории могут быть другие ресурсы компонента, в том числе вложенные компоненты, если они 
используются только в рамках компонента (его составные части).

```
component-name 
├── assets
|   └── bg.png
├── sub-component
|   ├── index.js
|   └── style.less
├── index.js
└── style.less
```

##Класс компонента {#class}

Компонент определяется через **класс**. Опредление через функцию нежелательно, чтобы экономить 
время при доработки внутренней логики и иметь единую структуру всех компонент.

> Для оптимизации рендера использовать `shouldComponentUpdate()`, `React.PureComponent` и не делать 
вычислений в `render()`

Название класса всегда с большой буквы (CamelCase).
Начинается с обобщенного названия и последующими частями уточняется назначение/особенность компонента. 
Например: `Button`, `ButtonNext`, `LayoutPage`, `MenuTop`, `FormNewsCreate`. 
Польза в быстром улавливании сути компонентов, в упорядоченности файлов. 
Допустимы исключения, например `RadioButton` общепринятое имя (можно считать обобщенным названием).

```javascript
class MenuTop extend Component {

}

export MenuTop;
```

Описание входных свойств делать в начале класса статическим свойством `propTypes` вместо его 
присвоения после определения класса. Статических свойств нет в стандарте ES2016/2017, но через 
babel поддерживаются. 

```javascript
class MenuTop extend Component {
  static propTypes = {
    items: PropTypes.array
  }
}
```

##Стили {#style}

Использовать [БЭМ](https://ru.bem.info/methodology/naming-convention/). 
Название класса блока соответствует классу компонента для консистентности и используется в первом 
теге рендера.

`.MenuTop`

Внутренняя разметка - это элементы, им названия css классов через двойное подчеркивание. 

`.MenuTop__item`. 

Для задания модификаций блоку или элементу использовать одно нижнее подчёркивание. 

`.MenuTop_theme_cool` 

```javascript
class MenuTop extend Component {
  render(){
    return (
      <div className="MenuTop MenuTop_theme_cool">
        <div className="MenuTop__item">
  
        </div>
        <div className="MenuTop__item MenuTop__item_active">
  
      </div>
      </div>
    );
  }
}
```

По БЭМ элементы есть только у блока. Элементов у элементов нет. Если по разметке что-то подобное 
напрашивается, то речь идет о вложенном блоке. В простых случаях его можно определить 
css классом в рамках компонента, но лучше подумать о выносе в отдельный компонент. 

Для указания нескольких классов или с условием использовать библиотеку 
[classnames](https://github.com/JedWatson/classnames), импортируя её в короткую константу `cs`

```JS
import cs from "classnames";

class MenuTop extend Component {
  render(){
    return (
      <div className="MenuTop">
        <div className="MenuTop__item">
  
        </div>
        <div className={cn("MenuTop__item", {"MenuTop__item_active": this.props.isActive})}>
  
      </div>
      </div>
    );
  }
}
```

##События (callbacks) {#callbacks}

Функции обратного вызова, используемые для событий DOM/компонент, определять методами класса 
стрелочным способом.  

```javascript
class MenuTop extend Component {
  onClick = () => {
    
  };
}
```

В ряде случаях используется каррирование для замыкания параметра, определяемого при рендере. 
Тогда стрелочное определение метода не требуется.

```javascript
class MenuTop extend Component {
  onClick(key){
    return (e) => {
      e.preventDefault();
      this.prop.doSome(key);
    }
  }
  
  renderItem(item){
    return (
      <li key={item.key} onClick={this.onClick(item.key)}></li>
    )
  }
}
```

Нужно учитывать, что при каждом рендере будет создаваться новая функция-обработчик

Если внешний обработчик (callback из props) вешается на DOM событие, то нужно переопределять его, 
не отдавая во вне объект DOM события. Например, компонент поля ввода в событии onChange должен 
вернуть значение поля.

```javascript
class Input extend Component {
  onChange(e) => {
    if (this.props.onChange){	
      this.props.onChange(e.target.value);
    }
  }
  
  render(){
    return (
      <input onChange={this.onChange} value={this.props.value}>
    )
  }  
}
```

Чтобы не проверять, определено ли необязательное свойство в `this.props`, рекомендуется описывать 
значения по умолчанию после определения `propTypes`

```javascript
class Input extend Component {
  
  static propTypes = {
    onChange: PropTypes.func
  }
  
  static defaultProps = {
    onChange: ()=>{}
  }
}
```