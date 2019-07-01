# Redux в React

В последних главах вы познакомились с чистым Redux. Он помогает вам управлять предсказуемым объектом состояния. Однако вы захотите использовать этот объект состояния в приложении в конце концов. Это может быть любое приложение JavaScript, которое имеет дело с управлением состоянием. В конце концов, принцип Redux может быть развернут на любом языке программирования для управления объектом состояния.

Управление состоянием в одностраничных приложениях (SPA) является одним из таких случаев использования Redux. Эти приложения обычно создаются на основе фреймворка (Angular) или библиотеки уровня представления (React, Vue), но чаще всего в этих решениях отсутствует сложное управление состоянием. Вот где Redux вступает в игру. Книга фокусируется на React, но вы можете применить полученные знания и к другим решениям, таким как Angular и Vue.

Следующие сценарии могут обойтись без Redux, потому что они не будут сталкиваться с проблемами управления состоянием с использованием в первую очередь только локального состояния. Но ради демонстрации Redux в React они будут опускать локальное управление состояниями и применять сложное управление состояниями с Redux.

## Подключение состояния

С одной стороны, у вас есть React в качестве слоя просмотра. В нем есть все необходимое для построения иерархии компонентов. Вы можете составлять компоненты друг в друга. Кроме того, методы компонента обеспечивают постоянную привязку к их жизненному циклу.

С другой стороны, у вас есть Redux. К настоящему времени вы должны знать, как управлять состоянием в Redux. Сначала вы инициализируете все, устанавливая редуктор (ы), действия и их необязательных создателей действий. После этого (комбинированный) редуктор используется для создания хранилища Redux. Во-вторых, вы можете взаимодействовать с хранилищем, отправляя действия с объектами простого действия или создателями действий, подписываясь на хранилище и получая текущее состояние из хранилища.

В конце концов, к этим трем взаимодействиям нужно получить доступ с вашего слоя представления. Как уже упоминалось, слой представления может быть чем угодно, но в этой книге это будет React.

Если вы вспомните однонаправленный поток данных в Redux, который был адаптирован на основе архитектуры Flux, вы заметите, что у вас уже есть все части в вашем распоряжении.

{title="Concept Playground",lang="text"}
~~~~~~~~
View -> Action -> Reducer(s) -> Store -> View
~~~~~~~~

Каким образом `dispatch()`, `subscribe()` и `getState()` могут быть доступны в представлении React? По сути, слой представления должен иметь возможность отправлять действия на одном конце, в то время как он должен прослушивать обновления из хранилища и получать свое состояние, чтобы обновлять себя, на другом конце. Все три функции доступны как методы в хранилище Redux.

### Практика: Bootstrap React App with Redux
Настоятельно рекомендуется использовать приложение create-react-app для начальной конфигурации React. Однако решать вам - следовать ли этому совету. Если вы решите использовать create-react-app и никогда не использовали его раньше, вы должны сначала установить его из командной строки с помощью пакетного менджера Node (npm):

{title="Command Line",lang="text"}
~~~~~~~~
npm install -g create-react-app
~~~~~~~~

Теперь вы можете загрузить приложение React с помощью create-react-app, перейти в папку и запустить его:

{title="Command Line",lang="text"}
~~~~~~~~
create-react-app taming-the-state-todo-app
cd taming-the-state-todo-app
npm start
~~~~~~~~

Если вы ранее не использовали create-react-app, я рекомендую вам ознакомиться с основами в  [официальной документации](https://github.com/facebookincubator/create-react-app). По сути, ваша папка *src/* содержит несколько файлов. В этом приложении вы не будете использовать файл *src/App.js*, а только файл *src/index.js*. Откройте редактор и отредактируйте файл *src/index.js* следующим образом:

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';

function TodoApp() {
  return <div>Todo App</div>;
}

ReactDOM.render(<TodoApp />, document.getElementById('root'));
~~~~~~~~

Теперь, когда вы снова запустите свое приложение с `npm start`, вы должны увидеть отображаемое "приложение Todo"из компонента `TodoApp`. Прежде чем продолжить создание приложения React сейчас, давайте подключим весь код Redux, который вы написали в предыдущих главах. Сначала установите Redux в свой проект:

{title="Command Line: /",lang="text"}
~~~~~~~~
npm install --save redux
~~~~~~~~

Во-вторых, повторно используйте код Redux из предыдущих глав в файле *src/index.js*. Вы начинаете сверху, чтобы импортировать две функции Redux, которые вы использовали до сих пор. Они относятся к импорту, который уже существует:

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
# leanpub-start-insert
import { combineReducers, createStore } from 'redux';
# leanpub-end-insert
import './index.css';
~~~~~~~~

Теперь, между вашими импортами и вашим кодом React, вы располагаете свои функциональные возможности Redux. Во-первых, типы действий:

{title="src/index.js",lang="javascript"}
~~~~~~~~
// action types

const TODO_ADD = 'TODO_ADD';
const TODO_TOGGLE = 'TODO_TOGGLE';
const FILTER_SET = 'FILTER_SET';
~~~~~~~~

Во-вторых, редукторы с их начальным состоянием:

{title="src/index.js",lang="javascript"}
~~~~~~~~
// reducers

const todos = [
  { id: '0', name: 'learn redux' },
  { id: '1', name: 'learn mobx' },
];

function todoReducer(state = todos, action) {
  switch(action.type) {
    case TODO_ADD : {
      return applyAddTodo(state, action);
    }
    case TODO_TOGGLE : {
      return applyToggleTodo(state, action);
    }
    default : return state;
  }
}

function applyAddTodo(state, action) {
  const todo = Object.assign({}, action.todo, { completed: false });
  return state.concat(todo);
}

function applyToggleTodo(state, action) {
  return state.map(todo =>
    todo.id === action.todo.id
      ? Object.assign({}, todo, { completed: !todo.completed })
      : todo
  );
}

function filterReducer(state = 'SHOW_ALL', action) {
  switch(action.type) {
    case FILTER_SET : {
      return applySetFilter(state, action);
    }
    default : return state;
  }
}

function applySetFilter(state, action) {
  return action.filter;
}
~~~~~~~~

Third, the action creators:
В третьих, создателей экшенов:

{title="src/index.js",lang="javascript"}
~~~~~~~~
// action creators

function doAddTodo(id, name) {
  return {
    type: TODO_ADD,
    todo: { id, name },
  };
}

function doToggleTodo(id) {
  return {
    type: TODO_TOGGLE,
    todo: { id },
  };
}

function doSetFilter(filter) {
  return {
    type: FILTER_SET,
    filter,
  };
}
~~~~~~~~

И последнее, но не менее важное, создание хранилища с обоими редукторами в качестве одного комбинированного редуктора:

{title="src/index.js",lang="javascript"}
~~~~~~~~
// store

const rootReducer = combineReducers({
  todoState: todoReducer,
  filterState: filterReducer,
});

const store = createStore(rootReducer);
~~~~~~~~

После этого следует ваш код React. Он уже должен быть в том же файле.

{title="src/index.js",lang="javascript"}
~~~~~~~~
// view layer

function TodoApp() {
  return <div>Todo App</div>;
}

ReactDOM.render(<TodoApp />, document.getElementById('root'));
~~~~~~~~

Настройка React и Redux завершена. Это приложение можно найти в [GitHub-репозитории](https://github.com/rwieruch/taming-the-state-todo-app/tree/0.0.0). Теперь у вас есть работающее приложение React и хранилище Redux. Но они еще не работают вместе. Следующий шаг - соединить оба вместе.

### Практика: Простое Todo с React и Redux

Далее будет продемонстрирован простой сценарий объединения Redux в React. Пока у вас есть только компонент `TodoApp` в React. Однако вы хотите запустить дерево компонентов, которое может отображать список задач и дает пользователю возможность переключить эти задачи в завершенное состояние. Помимо компонента `TodoApp` у вас будет компонент `TodoList` и компонент `TodoItem`. `TodoItem` показывает название задачи и имеет функциональность, которая используется в кнопке для завершения задачи.

Сначала, компонент `TodoApp`:

{title="src/index.js",lang="javascript"}
~~~~~~~~
function TodoApp({ todos, onToggleTodo }) {
  return <TodoList
    todos={todos}
    onToggleTodo={onToggleTodo}
  />;
}
~~~~~~~~

Затем, компонент `TodoList`:

{title="src/index.js",lang="javascript"}
~~~~~~~~
function TodoList({ todos, onToggleTodo }) {
  return (
    <div>
      {todos.map(todo => <TodoItem
        key={todo.id}
        todo={todo}
        onToggleTodo={onToggleTodo}
      />)}
    </div>
  );
}
~~~~~~~~

Затем, компонент `TodoItem`:

{title="src/index.js",lang="javascript"}
~~~~~~~~
function TodoItem({ todo, onToggleTodo }) {
  const { name, id, completed } = todo;
  return (
    <div>
      {name}
      <button
        type="button"
        onClick={() => onToggleTodo(id)}
      >
        {completed ? "Incomplete" : "Complete"}
    </button>
    </div>
  );
}
~~~~~~~~

Обратите внимание, что ни один из этих компонентов не знает о Redux. Они просто отображают задачи и используют функцию обратного вызова, чтобы переключать элементы задачи на полные или неполные. Теперь, на последнем шаге, вы соединяете вместе Redux и React. Вы можете использовать созданный экземпляр Redux `store` в своем корневом компоненте React, где React подключается к HTML.

{title="src/index.js",lang="javascript"}
~~~~~~~~
ReactDOM.render(
  <TodoApp
    todos={store.getState().todoState}
    onToggleTodo={id => store.dispatch(doToggleTodo(id))}
  />,
  document.getElementById('root')
);
~~~~~~~~

Хранилище делает две вещи: он делает состояние доступным и предоставляет функциональные возможности для изменения состояния. Пропсы `todos` передаются в `TodoApp` путем извлечения их из экземпляра Redux `store`. Кроме того, передается свойство `onToggleTodo`, которое является функцией. Эта функция является функцией высокого порядка, которая упаковывает диспетчеризацию действия, созданного его создателем действия. Однако компонент `TodoApp` полностью не знает о том, что` todos` извлекается из хранилища Redux или что `onToggleTodo()`является отправленным действием в хранилище Redux. Эти переданные свойства являются пропсами для `TodoApp`. Вы можете снова запустить свое приложение с помощью `npm start` и увидеть, какие задачи отображаются, но еще не обновлены после нажатия кнопки.

Так что насчет механизма обновления? Когда отправляется действие, кто-то должен подписаться на хранилище Redux. При простом подходе вы можете сделать следующее, чтобы принудительно обновить представление в React. Во-первых, оберните ваш корневой компонент React в функцию.

{title="src/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function render() {
# leanpub-end-insert
  ReactDOM.render(
    <TodoApp
      todos={store.getState().todoState}
      onToggleTodo={id => store.dispatch(doToggleTodo(id))}
    />,
    document.getElementById('root')
  );
# leanpub-start-insert
}
# leanpub-end-insert
~~~~~~~~

Во-вторых, вы можете передать функцию в метод `subscribe()` хранилища Redux в качестве функции обратного вызова. Таким образом, функция вызывается каждый раз, когда изменяется состояние в хранилище Redux. И последнее, но не менее важное: вы должны вызывать функцию один раз для первоначального рендеринга вашего компонента React.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function render() {
  ReactDOM.render(
    <TodoApp
      todos={store.getState().todoState}
      onToggleTodo={id => store.dispatch(doToggleTodo(id))}
    />,
    document.getElementById('root')
  );
}

# leanpub-start-insert
store.subscribe(render);
render();
# leanpub-end-insert
~~~~~~~~

Приложение Todo должно отображать задачи и обновлять завершенное состояние после его переключения. Окончательное применение этого подхода можно найти в [GitHub-репозитарии](https://github.com/rwieruch/taming-the-state-todo-app/tree/1.0.1).

Подход продемонстрировал, как можно связать дерево компонентов React с хранилищем Redux. Компоненты вообще не должны знать о хранилище Redux, но корневой компонент React должен. Кроме того, все принудительно перерисовывается, когда обновляется глобальное состояние в хранилище Redux, потому что функция `render` вызывается при каждом изменении состояния.

Хотя предыдущий подход прагматичен и показывает упрощенную версию того, как соединить все эти вещи, это наивный подход. Это почему? В реальном приложении вы хотите избежать следующих плохих практик:

* перерисовка каждого компонента: вы хотите перерисовать только те компоненты, на которые влияет глобальное состояние, обновленное в хранилище Redux. В противном случае вы столкнетесь с проблемами производительности в более крупном приложении, потому что каждый компонент должен рендериться заново.

* использование экземпляра хранилища напрямую: вы хотите избежать непосредственной работы с экземпляром хранилища Redux. Хранилище должно быть каким-то образом вставлено в дерево компонентов React, чтобы сделать его доступным для компонентов, которым необходим доступ к хранилищу.

* сделать магазин доступным для всех: магазин не должен быть доступен для всех компонентов. В предыдущем примере его использует только корневой компонент React, но кто мешает вам использовать (импортировать) его непосредственно в компоненте `TodoItem` для отправки действия?

К счастью, существует библиотека, которая заботится об этих вещах и дает вам мост из Redux в мир React. Он соединяет ваш слой состояния с вашим слоем представления и четко разделяет оба ограничения.

## Соединяя состояние, но практично и изящно

Библиотека под названием [react-redux](https://github.com/reactjs/react-redux) дает вам две вещи для подключения Redux к React. Во-первых, он дает вам компонент `<Provider />`. При использовании Redux с React компонент `Provider` должен быть компонентом верхнего уровня вашего приложения. Компонент имеет один пропс в качестве входных данных: хранилище Redux, которое вы создали с помощью `createStore()`.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { Provider } from 'react-redux'

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
~~~~~~~~

После того, как вы это сделали, каждый дочерний компонент во всем дереве компонентов имеет неявный доступ к хранилищу. Таким образом, каждый компонент может отправлять действия и прослушивать обновления для повторного рендеринга. Но не каждый компонент должен слушать обновления. Как это работает, не передавая хранилище в качестве реквизита каждому дочернему компоненту? Он использует шаблон провайдера, который вы узнали в предыдущей главе, когда выполняете только управление состоянием с помощью React. Под капотом он использует контекстный API React:

*«В некоторых случаях вы хотите передавать данные через дерево компонентов без необходимости вручную передавать реквизиты на каждом уровне. Это можно сделать напрямую в React с помощью мощного «контекстного» API.»*

Это была первая часть использования Redux в React. Во-вторых, вы можете использовать компонент  высокого порядка, который называется `connect` из новой библиотеки. Это делает доступными функциональность хранилища Redux и состояние из самого хранилища для компонентов, улучшенных этим компонентом высокого порядка.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { connect } from 'react-redux'

function Component(props) {
  ...
}

const ConnectedComponent = connect(...)(Component);
~~~~~~~~

КВП (компонент высшео порядка) `connect` может иметь до четырех аргументов в качестве конфигурации:

{title="Code Playground",lang="javascript"}
~~~~~~~~
connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])(...);
~~~~~~~~

Обычно вы будете использовать только два из них: `mapStateToProps()` и `mapDispatchToProps()`. Вы узнаете о двух других аргументах `mergeProps()` и `options` позже в этой книге.

**mapStateToProps(state, [props]) => derivedProps:** Это функция, которую можно передать в КВП `connect`. Если он пропущен, компонент ввода подключения HOC подпишется на обновления из хранилища Redux. Таким образом, это означает, что каждый раз, когда подписка хранилища замечает обновление, запускается функция `mapStateToProps()`. Функция `mapStateToProps()` сама имеет два аргумента в сигнатуре функции: глобальный объект состояния из предоставленного хранилища Redux и, опционально, пропсы из родительского компонента, где в конечном итоге используется расширенный компонент. В конце концов, функция возвращает объект, который получен из глобального состояния и, возможно, из пропсов родительского компонента. Возвращенный объект будет объединен с остальными элементами, которые поступают в качестве входных данных от родительского компонента.

**mapDispatchToProps(dispatch, [props]):** Это функция (или объект), которую можно передать в подключенный КВП. В то время как `mapStateToProps()` предоставляет доступ к глобальному состоянию, `mapDispatchToProps()` предоставляет доступ к методу отправки хранилища Redux. Это позволяет диспетчеризировать действия, но пропускает только простые функции, которые связывают диспетчеризацию в функции более высокого порядка. В конце концов, это позволяет передавать функции во входной компонент КВП connect для изменения состояния. По желанию, здесь вы также можете использовать входящие пропсы, чтобы обернуть их в отправленное действие.

Это много знаний, чтобы переварить. Обе функции, `mapStateToProps()` и `mapDispatchToProps()`, могут пугать в начале. Кроме того, они используются в компоненте высокого порядка. Тем не менее, они только дают вам доступ к состоянию и способу отправки хранилища Redux.

{title="Concept Playground",lang="text"}
~~~~~~~~
View -> (mapDispatchToProps) -> Action -> Reducer(s) -> Store -> (mapStateToProps) -> View
~~~~~~~~

В следующих примерах вы увидите, что эти функции вообще не должны пугать. Они являются вашими обычные инструментами для соединения слоя состояний с вашим слоем представления на обоих концах.

### Практика: Усложненая Todo с React и Redux

Теперь вы будете использовать react-redux для соединения React с Redux. Давайте вернемся к вашему приложению Todo из предыдущей главы. Сначала вам нужно установить новую библиотеку, чтобы соединить оба мира:

{title="Command Line: /",lang="text"}
~~~~~~~~
npm install --save react-redux
~~~~~~~~

Во-вторых, вместо того, чтобы оборачивать корневой компонент React в функцию `render()` и подписывать его на метод `store.subscribe()`, вы снова будете использовать простой корневой компонент React, но будете использовать компонент `Provider`, получаемый из react-redux.

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import { combineReducers, createStore } from 'redux';
# leanpub-start-insert
import { Provider } from 'react-redux';
# leanpub-end-insert
import './index.css';

...

# leanpub-start-insert
ReactDOM.render(
  <Provider store={store}>
    <TodoApp />
  </Provider>,
  document.getElementById('root')
);
# leanpub-end-insert
~~~~~~~~

Он использует простой компонент `TodoApp`. Компонент по-прежнему ожидает `todos` и` onToggleTodo` в качестве пропсов. Но у него нет этих пропсов. Давайте использовать компонент высшего порядка `connect`, чтобы представить их компоненту `TodoApp`. Компонент `TodoApp` станет подключенным компонентом `TodoApp`.

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import { combineReducers, createStore } from 'redux';
# leanpub-start-insert
import { Provider, connect } from 'react-redux';
# leanpub-end-insert
import './index.css';

...

# leanpub-start-insert
const ConnectedTodoApp = connect(mapStateToProps, mapDispatchToProps)(TodoApp);
# leanpub-end-insert

ReactDOM.render(
  <Provider store={store}>
# leanpub-start-insert
    <ConnectedTodoApp />
# leanpub-end-insert
  </Provider>,
  document.getElementById('root')
);
~~~~~~~~

Теперь только соединения, `mapStateToProps()` и `mapDispatchToProps()` отсутствуют. Они очень похожи на наивную версию React с Redux.

{title="src/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function mapStateToProps(state) {
  return {
    todos: state.todoState,
  };
}

function mapDispatchToProps(dispatch) {
  return {
     onToggleTodo: id => dispatch(doToggleTodo(id)),
  };
}
# leanpub-end-insert

const ConnectedTodoApp = connect(mapStateToProps, mapDispatchToProps)(TodoApp);

...
~~~~~~~~

Вот и все. В `mapStateToProps()` возвращается только подсостояние. В `mapDispatchToProps()` возвращается только функция высокого порядка, которая инкапсулирует диспетчеризацию действия. Дочерние компоненты не знают о каком-либо состоянии или действиях. Они получают только пропсы. В этом случае реквизитами являются `todos` и `onToggleTodo`. Окончательное применение этого подхода можно найти в [GitHub-репозитории](https://github.com/rwieruch/taming-the-state-todo-app/tree/2.0.0). Я бы посоветовал вам сравнить его с наивной версией, которая соединяет React и Redux вместе. Это ничем не отличается от этого.

### Практика: Соединяем состояние везде

Есть один последний ключ к пониманию основ соединения React и Redux вместе. В предыдущем примере вы использовали только один подключенный компонент, который находится в корне дерева компонентов. Но вы можете использовать подключенные компоненты везде. Итак, давайте попробуем это на практике в вашем приложении Todo.

Вариант использования: только ваш компонент `TodoApp` имеет доступ к состоянию и позволяет изменять его. Вместо того чтобы использовать корневой компонент для подключения к нему в хранилище, вы можете добавлять подключенные компоненты повсюду в дереве компонентов React. Например, функция `onToggleTodo()` должна пройти несколько компонентов, пока не достигнет своего назначения в компоненте `TodoItem`. Почему бы не подключить компонент `TodoItem`, чтобы сделать функциональность прямо рядом с ним доступной, вместо того, чтобы передавать его нескольким компонентам, которые не заинтересованы в нем? То же самое относится и к компоненту `TodoList`. Он может быть подключен для получения списка задач вместо того, чтобы получать его из компонента `TodoApp`, который не нуждается в этом списке.

В приложении Todo вы можете оставить как `mapStateToProps()`, так и `mapDispatchToProps ()`, там бизнес-логика остается прежней, но вы можете использовать их где-то еще в дереве компонентов React. Хотя компоненту TodoApp они больше не нужны, они будут использоваться в подключенном компоненте `TodoItem` и подключенном компоненте `TodoList`.

{title="src/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const ConnectedTodoList = connect(mapStateToProps)(TodoList);
const ConnectedTodoItem = connect(null, mapDispatchToProps)(TodoItem);
# leanpub-end-insert

ReactDOM.render(
  <Provider store={store}>
# leanpub-start-insert
    <TodoApp />
# leanpub-end-insert
  </Provider>,
  document.getElementById('root')
);
~~~~~~~~

Теперь вам больше не нужно пропускать пропсы `onToggleTodo()` через компонент `TodoApp` и `TodoList`. То же самое относится к `todos`, которые не нужно пропускать через компонент `TodoApp`.

{title="src/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function TodoApp() {
  return <ConnectedTodoList />;
}

function TodoList({ todos }) {
  return (
    <div>
      {todos.map(todo => <ConnectedTodoItem
        key={todo.id}
        todo={todo}
      />)}
    </div>
  );
}
# leanpub-end-insert
~~~~~~~~

Финальное приложение Todo можно найти в [GitHub-репозитарии](https://github.com/rwieruch/taming-the-state-todo-app/tree/3.0.0).

Как вы уже можете себе представить, вы можете подключить свое состояние везде к слою представления. Вы можете извлечь его с помощью `mapStateToProps()` и изменить его с помощью `mapDispatchToProps()` из любой точки вашего дерева компонентов. Эти компоненты, которые добавляют этот промежуточный клей между представлением и состоянием, называются связанными компонентами. Они являются подмножеством компонентов контейнера из шаблона контейнера и презентатора, в котором компоненты презентатора не имеют представления и не знают, получены ли пропсы из хранилища Redux, из локального состояния или действий. Они просто используют эти пропсы.

В конце концов, это в основном все, что вам нужно, чтобы подключить ваш слой состояний (Redux) к слою представления (React). Как уже упоминалось, ваш слой представления может быть реализован с другой библиотекой, такой как Vue, например.
