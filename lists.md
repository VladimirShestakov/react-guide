# Списки

Компонент списка выполняет рендер массива данных. Для кастомизации рендера отдельной строки 
используется переданная в props функция `renderItem()`. Компонент списка перебирает переданный
массив данных и на каждый элемент массива вызывает функцию рендера строки `renderItem()`. 
За счёт этого в рамках одного списка можно по разному выводить строки в зависимости от свойств объекта.

В теге элемента списка необходимо указывать уникальный атрибут `key`, но так как структура данных в 
массиве может быть любой, определение ключа реализуется внешней функцией `props.getKey()`.

По подобию функции рендера строки реализуется рендер разделителя между строками `renderSeparator()`.
Отличительной особенностью является передача в функцию предыдущего и следующего элемента массива.
В функции разделителя можно реализовывать различные условия вывода. Например, в качестве 
разделителя может быть простая линия, а может быть блок с датой.

```javascript
class ListBase extends Component {

  static propTypes = {
    items: PropTypes.array.isRequired,
    renderItem: PropTypes.func.isRequired,
    renderSeparator: PropTypes.func,
    getKey: PropTypes.func,
  };

  static defaultProps = {
    getKey: (item, index) => item.id || index,
    renderSeparator: (prevItem, nextItem, prevIndex) => null
  };

  render() {
    const {renderItem, renderSeparator, items} = this.props;
    const sep = items.length ? renderSeparator(null, items[0], null) : null;
    return (
      <ul className="ListBase">
        {sep ? <li className="ListBase__separator" key={`sep-first`}>{sep}</li> : null}
        {items.map((item, index) => {
          const key = this.props.getKey(item, index);
          const row = [
            <li className="ListBase__item" key={key} >
              {renderItem(item, index)}
            </li>
          ];
          const sep = renderSeparator(item, items.length > index + 1 ? items[index + 1] : null, index);
          if (sep) {
            row.push(<li className="ListBase__separator" key={`sep-${key}`}>{sep}</li>);
          }
          return row;
        })}
      </ul>
    );
  }
}
```

Пример интеграции списка. В props передаются массив данных, функция рендера строки и функция 
выборки ключа. 

```javascript
const data = [
   {id: 1, title: 'Строка 1'},
   {id: 2, title: 'Строка 2'},
   {id: 3, title: 'Строка 3'}
];

class Example extends Component {
  
  renderItem(item, index) {
    return (
      <div className="ExampleItem">{item.title}</div>
    );
  }
  
  getKey(item, index) {
    return item.id;
  }
  
  render(){
    return (
      <ListBase
        items={data}
        getKey={this.getKey}
        renderItem={this.renderItem}
      />
    )
  }
}
```

Списковые данные часто представляют собой таблицу, свёрстанную тегом `<table>`. 
Тогда шапку (заголовки колонок) или футер списка необходимо реализовывать внутри тега `<table>`, 
получается внутри компонента списка. Для их кастомизации реализуются внешние функции рендера 
шапки и футера наподобие рендера строки. Только нужно обусловиться об использовании в них тегов 
таблицы `<th>, <td>...`.