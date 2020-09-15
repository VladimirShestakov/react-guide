# Вёрстка

1. Вёрстка (теги html) только в *компонентах* в директории `@components/`, в других частях приложения вёрстка недопустима.
2. Импорт файлов стилей (css, less...) тоже только в *компонентах*.
3. Желательно использовать препроцессор less, но допустимы и другие варианты - главное аргументировать их.
4. CSS классы именовать по [БЭМ](https://ru.bem.info/methodology/quick-start/) методологии: 
    - `.ComponentName` - название *блока*, соответствует названию компонента, указывается для корневого тега компонента; 
    - `.ComponentName_modify` - *модификация блока* (модификация стиля корневого тега компонента);
    - `.ComponentName__elementName` - *элемент* блока (практически все вложенные теги в разметке компонента); 
    - `.ComponentName__elementName_modify` - *модификация элемента*;
    - `.modify` - допускаются простые классы для модификации блока или элемента, но их определение должно быть в паре с классом блока или элемента `.CompoenntName.modify{}`;
    - Коллизия именований классов избегается за счёт уникальности именования компонент.
5. В описании стилей компонента не использовать каскад с классом от другого компонента.
6. Динамическая стилизация компонента через `props` - это передача названия класса *модификатора блока*. 
    - Нежелательно передавать через `props` свойства стиля (цвет, например) - тогда реализация отображения переместится из компонента в контейнер.
    - По стилям компонента можно узнать все его варианты отображения.
7. Для комбинации нескольких классов в атрибуте `className` использовать библиотеку [`classnames`](https://www.npmjs.com/package/classnames). 
8. Для подставления модификатора в атрибуте `className` лучше применить функцию [`@utils/themes`](https://github.com/ylabio/react-skeleton/blob/master/src/utils/themes.js)
    - Основана на classnames
    - Автоматом всем классам подставляет название блока с модификатором `_theme_${modify}`:
   ```js
   themes('ComponentName', 'skin1 skin2'); //→ "ComponentName ComponentName_theme_skin1 ComponentName_theme_skin2"
   themes('ComponentName', {skin3: true, skin4: false}); //→ "ComponentName ComponentName_theme_skin3"
   ```
9. [Модульный импорт стилей](https://github.com/css-modules/css-modules) использовать по согласованию с командой разработчиков. 
    - В сборке ccs классы получат короткие хэш названия. Усложнится отладка вёрстки на проде и поиск соответствия в коде. (-)
    - Вместо строки класса будет свойство объекта классов. `classes.ComponentName`. Указание класса все равно будет длинным. (-)  
    - Можно будет отказаться от БЭМ, но избавимся от семантики разметки, сложнее будет найти границы компонента в документе. (-)
    - Возможны конфликты с импортом стилей сторонних библиотек. Потребуются исключения в опциях webpack. (-)
    - Неоднозначность именования свойств (названий классов) в объекте стилей. (-)
    - Автодополнения при вводе названия класса (+) 
    - Без :global не получится сделать каскад от класса другого компонента. (+)
    - Если нужно усложнить разбор кода сборки. (+)
    - Устраняется вероятность коллизий в именовании классов разных компонент.(+)
10. Стилизация на js, styled-components и прочее не используются, так как ограничивают эффективность верстальщика.
11. Для разметки областей создавать компоненты Layout* в `@components/layout/`. 
    - Компонентами разметки определяются области, в которые вставляются соответствующие свойства из `props`.
    - Учитывать вариативность компонента разметки, чтобы повторно использовать в разных местах.
12. В компоненте разметка одного блока. Желание заверстать ещё один блок - признак создать отдельный компонент. 
13. Использовать HTML5 теги для семантической разметки, но им все равно назначать классы по БЭМ для стилизации.
14. Общие стили, шрифты, css переменные, нормалайзеры, картинки и другие ресурсы разметки помещаются в директории `@theme/`

```js
/**
 * Компонент разметки страницы с тремя областями для вставки
 */
import React from 'react';
import PropTypes from 'prop-types';
import cn from 'classnames';
import themes from '@utils/themes';
import './style.less';

function LayoutPage (props) {
  return (
    <div className={themes('LayoutPage', props.theme)}>
      <header className="LayoutPage__header">{props.header}</header>
      <div className="LayoutPage__content">{props.children || props.content}</div>
      <footer className="LayoutPage__footer">{props.footer}</footer>
    </div>
  );
}

LayoutPage.propTypes = {
  header: PropTypes.node,
  content: PropTypes.node,
  footer: PropTypes.node,
  children: PropTypes.node,
  theme: PropTypes.oneOfType([PropTypes.string, PropTypes.array]),//можно передать несколько тем через пробел или массив
};

LayoutPage.defaultProps = {
  theme: ''
}

export default React.memo(LayoutPage);
```

Пример стилей с темой `night`
```css   
.LayoutPage{
  display: flex;
  min-height: 100vh;
  flex-direction: column;

  &_theme {
    &_night {
      background: #000000;
      color: #eeeeee;
    }
  }

  &__content{
    flex: 1;
  }
}
```

Применение компонента
```js
function AboutContainer(){
  return (
    <LayoutPage header={<MainHeader/>} footer={<MainFooter/>} theme="night">
      <About/>
    </LayoutPage>
  );
}

```

