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
5. В описании стилей компонента не использовать каскад с классом от другого компонента.
6. Динамическая стилизация компонента через `props` - это передача названия класса *модификатора блока*. 
    - Нежелательно передавать через `props` свойства стиля (цвет, например) - тогда реализация отображения переместиться из компонента в контейнер.
    - По стилям компонента можно узнать все его варианты отображения.
7. Для комбинации нескольких классов и условий подставления в теге использовать библиотеку [`classnames`](https://www.npmjs.com/package/classnames). 
8. Для подставления модификатора варианта оформления всего компонента лучше применить функцию `@utils/theme`
   - Основана на classnames
   - Автоматом всем классам подставляет название блока с модификатором <br/>
   ```theme('ComponentName', 'skin1 skin2') => "ComponentName_theme_skin1 ComponentName_theme_skin2"```.
9. Для разметки областей создавать компоненты Layout* в `@components/layout/`. 
    - Компонентами разметки определяются области, в которые вставляются соответствующие свойства из `props`.
    - Учитывать вариативность компонента разметки, чтобы повторно использовать в разных местах.
10. В компоненте разметка одного блока. Желание заверстать ещё один блок - признак создать отдельный компонент. 


```js
/**
 * Компонент разметки страницы с тремя областями для вставки
 */
import React from 'react';
import PropTypes from 'prop-types';
import './style.less';
import themes from '@utils/themes';
import cn from 'classnames';

function LayoutPage (props) {
  return (
    <div className={cn(`LayoutPage`, themes('LayoutPage', props.theme))}>
      <div className="LayoutPage__header">{props.header}</div>
      <div className="LayoutPage__content">{props.children || props.content}</div>
      <div className="LayoutPage__footer">{props.footer}</div>
    </div>
  );
}

LayoutPage.propTypes = {
  header: PropTypes.node,
  content: PropTypes.node,
  footer: PropTypes.node,
  children: PropTypes.node,
  theme: PropTypes.oneOfType([PropTypes.string, PropTypes.array]), // можно передать несколько тем через пробел или массивом
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
      background: black;
      color: #eeeeee;
    }
  }

  &__header{
  }

  &__content{
    flex: 1;
  }

  &__footer{
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
