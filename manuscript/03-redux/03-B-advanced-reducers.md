## Продвинутые Редукторы

Помимо продвинутых тем о действиях, есть еще много информации о редукторах. Опять же, не все является обязательным, но вы должны по крайней мере знать о следующих вещах, чтобы охватить лучшие практики, общие шаблоны использования и практики по масштабированию архитектуры состояния.

### Начальное состояние

До сих пор вы предоставили вашему хранилищу начальное состояние. Это был пустой список задач.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const store = createStore(reducer, []);
~~~~~~~~

Это начальное состояние всего хранилища Redux. Тем не менее, вы можете применить начальное состояние на более мелком уровне. Перед отправкой вашего первого действия, хранилище Redux будет инициализировано, пройдя все редукторы один раз. Вы можете попробовать это, удалив все ваши отправки в редакторе и добавив `console.log()` в редуктор. Вы увидите, что он запускается с инициализирующим действием один раз, хотя отправленного действия нет.

Действие инициализации, полученное в редукторе, сопровождается начальным состоянием, которое указано в функции `createStore()`. Однако если вы пропустите начальное состояние при инициализации хранилища, входящее состояние в редукторе будет `undefined`.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const store = createStore(reducer);
~~~~~~~~

Здесь вы можете указать начальное состояние на более детальном уровне. Если входящее состояние равно «undefined», вы можете [параметром JavaScript ES6 по умолчанию](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Default_parameters) установить начальное состояние.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function reducer(state = [], action) {
# leanpub-end-insert
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
~~~~~~~~

Начальное состояние будет таким же, как и раньше, но вы определили его на более детальном уровне. Позже это поможет вам, когда вы укажете более одного редуктора, и ваш объект состояния станет более сложным, потому что он разделен на несколько редукторов. Тогда каждый редуктор может определить свое детальное начальное состояние.

### Вложенные структуры данных

Начальное состояние - пустой список задач. Однако в растущем приложении вы хотите работать не только над задачами. У вас может быть объект `currentUser`, который представляет зарегистрированного пользователя в вашем приложении. Кроме того, вы хотите иметь свойство `filter` для фильтрации задач по их свойству` completed`. В растущем приложении все больше объектов и массивов будут собираться в глобальном состоянии. Вот почему ваше начальное состояние должно быть не пустым массивом, а объектом, представляющим объект состояния. Этот объект затем имеет вложенные свойства для `todos`,` currentUser` и `filter`.

Не принимая во внимание начальное состояние редуктора из предыдущей главы, вы должны определить свое начальное состояние в хранилище, как показано ниже:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const initialState = {
  todos: [],
};
const store = createStore(reducer, initialState);
~~~~~~~~

Теперь вы можете использовать пространство горизонтально в вашем объекте `initialState`. В какой-то момент это может вырасти до следующего:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const initialState = {
  currentUser: null,
  todos: [],
  filter: 'SHOW_ALL',
};
~~~~~~~~

Если вы вернетесь к своему приложению Todo, вам придется настроить редуктор. Редуктор обрабатывает список задач как состояние, но теперь это сложный объект глобального состояния.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function reducer(state, action) {
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
# leanpub-start-insert
  const todo = Object.assign({}, action.todo, { completed: false });
  const todos = state.todos.concat(todo);
  return Object.assign({}, state, { todos });
# leanpub-end-insert
}

function applyToggleTodo(state, action) {
# leanpub-start-insert
  const todos = state.todos.map(todo =>
    todo.id === action.todo.id
      ? Object.assign({}, todo, { completed: !todo.completed })
      : todo
  );
  return Object.assign({}, state, { todos });
# leanpub-end-insert
}
~~~~~~~~

Эти **вложенные структуры данных** хороши в Redux, но вы должны избегать **глубоко вложенных структур данных**. Как вы можете видеть, это добавляет сложности для создания вашего нового объекта состояния. Другая глава, которую вы прочтете позже в книге, снова поднимет эту тему. Он продемонстрирует, как вы можете избежать глубоко вложенных структур данных, используя лаконичную вспомогательную библиотеку с именем normalizr.

### Комбинированный редуктор

Следующая глава крайне важна для понимания принципов масштабирования состояния используя подсостояния в Redux. В предыдущих главах вы слышали о нескольких редукторах, но еще не использовали их. Как вы узнали, редуктор растет горизонтально, когда имеет дело с более чем одним типом действия. Но это не масштабируется в какой-то момент. Вы хотите разделить ваш редуктор на два редуктора или ввести другой редуктор с самого начала. Представьте, что у вас есть редуктор, который обслуживал ваши задачи, но вы хотите иметь редуктор для состояния `filter` для ваших задач. Другой вариант использования может состоять в том, чтобы иметь редуктор для `currentUser`, который зарегистрирован в вашем приложении Todo.

Вы уже можете увидеть шаблон, как разделить ваши редукторы. Обычно это области применения такие как задачи, фильтры или пользователи. Редуктор задач может быть ответственным за добавление, удаление, редактирование и заполнение задач. Редуктор фильтр отвечает за управление состоянием фильтра ваших задач. Пользовательский редуктор заботится о пользовательских объектах, которыми может быть `currentUser`, пользователь вошедший в ваше приложение, или список пользователей, которым назначены задачи. Вот где вы могли бы снова разделить пользовательский редуктор на редуктор `currentUser` и редуктор` assignUsers`. Вы можете представить, как этот подход, представляя редукторы по областям, очень хорошо масштабируется.

Давайте введем **комбинированные редукторы**, чтобы вы могли использовать несколько редукторов. Redux дает вам помощника для объединения нескольких редукторов в один корневой редуктор: `combineReducers()`. Функция принимает в качестве аргумента объект, которому может быть присвоено несколько редукторов для их состояния, которым они будут управлять.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const rootReducer = combineReducers({
  todoState: todoReducer,
  filterState: filterReducer,
});
~~~~~~~~

После этого `rootReducer` может использоваться для инициализации хранилища Redux вместо редуктора todo. В конце концов, для инициализации хранилища Redux необходим только один редуктор. Таким образом, это должно быть сочетание нескольких редукторов при наличии более одного редуктора.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const store = createStore(rootReducer);
~~~~~~~~

Вот и все для инициализации магазина Redux с комбинированными редукторами. Но как насчет самих редукторов? К нас уже есть один редуктор, который заботится о задачах, вы можете переименовать его.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function todoReducer(state, action) {
# leanpub-end-insert
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
~~~~~~~~

`filterReducer` еще не существует. Это будет выглядеть примерно так, чтобы установить фильтр для задач (например, SHOW_ALL, SHOW_COMPLETED, SHOW_INCOMPLETED):

{title="Code Playground",lang="javascript"}
~~~~~~~~
function filterReducer(state, action) {
  switch(action.type) {
    case FILTER_SET : {
      return applySetFilter(state, action);
    }
    default : return state;
  }
}
~~~~~~~~

Теперь перейдем к важной части: функция `combineReducers()` вводит промежуточный уровень состояния для объекта глобального состояния. Глобальный объект состояния при использовании комбинированных редукторов, как показано выше, будет выглядеть следующим образом:

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  todoState: ...,
  filterState: ...,
}
~~~~~~~~

Ключи свойств для промежуточного уровня - это те, которые определены в функции `combineReducers()`. Однако в ваших редукторах входящее состояние больше не является глобальным объектом состояния. Это их определенное подсостояние из функции `combinedReducers() `. `todoReducer` ничего не знает о` filterState`, а `filterReducer` ничего не знает о `todoState`.

`filterReducer` и` todoReducer` могут использовать параметр по умолчанию JavaScript ES6 для определения своего начального состояния.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function todoReducer(state = [], action) {
# leanpub-end-insert
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

# leanpub-start-insert
function filterReducer(state = 'SHOW_ALL', action) {
# leanpub-end-insert
  switch(action.type) {
    case FILTER_SET : {
      return applySetFilter(state, action);
    }
    default : return state;
  }
}
~~~~~~~~

Теперь ваша функция `apply` в редукторах должна снова работать с этими подсостояниями.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function applyAddTodo(state, action) {
# leanpub-start-insert
  const todo = Object.assign({}, action.todo, { completed: false });
  return state.concat(todo);
# leanpub-end-insert
}

function applyToggleTodo(state, action) {
# leanpub-start-insert
  return state.map(todo =>
    todo.id === action.todo.id
      ? Object.assign({}, todo, { completed: !todo.completed })
      : todo
  );
# leanpub-end-insert
}

# leanpub-start-insert
function applySetFilter(state, action) {
  return action.filter;
}
# leanpub-end-insert
~~~~~~~~

Кажется, очень сложно определить несколько редукторов и в конечном итоге объединить их. Но это крайне важные сведения, чтобы разбить ваше глобальное состояние на подсостояния. Этими подсостояниями могут управлять их редукторы, которые работают только в этих подсостояниях. Кроме того, каждый редуктор может отвечать за определение начального подсостояния.

### Разъяснение по Начальному состоянию

Последние главы несколько раз перемещались от инициализации начального состояния в `createStore()` до инициализации в редукторе(ах). Вы можете спросить, где инициализировать ваше состояние в конце концов. Для этого вы должны понимать, использовать ли комбинированные редукторы или только один простой редуктор.

**Один простой редуктор:** При использовании только одного простого редуктора начальное состояние в `createStore ()` доминирует над начальным состоянием в редукторе. Начальное состояние в редукторе работает только тогда, когда входящее начальное состояние равно `undefined`, потому что тогда оно может применить состояние по умолчанию из параметра по умолчанию. Но начальное состояние уже определено в `createStore ()` и, таким образом, используется редуктором.

**Комбинированные редукторы:** При использовании комбинированных редукторов вы можете использовать более детальное использование инициализации состояния. Объект начального состояния, который используется для функции `createStore()`, не обязательно должен включать все подсостояния, которые вводятся функцией `combineReducers()`. Таким образом, когда подсостояние `undefined`, редуктор может определить подсостояние по умолчанию. В противном случае используется подсостояние по умолчанию из `createStore()`.

### Вложенные редукторы

К настоящему времени вы знаете две вещи о масштабирующих редукторах в растущем приложении, которые требуют сложного управления состоянием:

* редуктор может заботиться о различных типах действий
* редуктор может быть разделен на несколько редукторов, но может быть объединен в один корневой редуктор для инициализации хранилища.

Эти шаги используются для горизонтального масштабирования редукторов (даже если комбинированные редукторы добавляют как минимум один вертикальный уровень). Редуктор работает в глобальном состоянии или в подсостоянии при использовании комбинированных редукторов. Тем не менее, вы можете использовать вложенные редукторы, чтобы вводить вертикальные уровни подсостояния.

Возьмем, к примеру, `todoReducer`, который работает со списком задач. С технической точки зрения, список задач имеет отдельные объекты задач. Так почему бы не ввести вложенный редуктор, который обрабатывает подсостояние todo как сущности?

{title="Code Playground",lang="javascript"}
~~~~~~~~
function todoReducer(state = [], action) {
  switch(action.type) {
    case TODO_ADD : {
# leanpub-start-insert
      return state.concat(todoEntityReducer(undefined, action));
# leanpub-end-insert
    }
    case TODO_TOGGLE : {
# leanpub-start-insert
      return state.map(todo => todoEntityReducer(todo, action));
# leanpub-end-insert
    }
    default : return state;
  }
}

# leanpub-start-insert
function todoEntityReducer(state, action) {
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
  return Object.assign({}, action.todo, { completed: false });
}

function applyToggleTodo(todo, action) {
  return todo.id === action.todo.id
    ? Object.assign({}, todo, { completed: !todo.completed })
    : todo
}
# leanpub-end-insert
~~~~~~~~

Вы можете использовать вложенные редукторы, чтобы ввести промежуточные уровни для подсостояний. Кроме того, они могут быть использованы повторно. Вы можете столкнуться со случаями, когда вы можете повторно использовать вложенный редуктор в другом месте. Однако, хотя вложенные редукторы могут дать вам лучшее представление о вашем состоянии, они также повышают уровень сложности вашего состояния. Во-первых, вы должны придерживаться практики не слишком глубоко вкладывать свое состояние. Тогда вы не будете часто сталкиваться с вложенными редукторами.

### Практика: Redux с продвинутыми Редукторами

Давайте снова окунемся в игровую площадку Redux с приобретенными знаниями о редукторах. Опять же, вы можете взять [проект JS Bin, который вы сделали в предыдущей главе](https://jsbin.com/kopohur/29/edit?html,js,console). Проект будет использован для демонстрации продвинутых редукторов. Вы можете попробовать это самостоятельно. В противном случае следующая часть проведет вас через рефакторинг. Сначала добавим второй редуктор для фильтрации задач.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const FILTER_SET = 'FILTER_SET';

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

Редуктор имеет исходное состояние. Пока нет действия, которое бы служило редуктору. Вы можете добавить простого создателя действий, который получает только строку в качестве аргумента для свойства фильтра. Позже вы узнаете, как это можно использовать в реальном приложении.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function doSetFilter(filter) {
  return {
    type: FILTER_SET,
    filter,
  };
}
~~~~~~~~

Во-вторых, вы можете переименовать ваш первый редуктор в `todoReducer` и присвоить ему начальное состояние пустого списка задач.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function todoReducer(state = [], action) {
# leanpub-end-insert
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
~~~~~~~~

Начальное состояние больше не инициализируется в функции `createStore()`. Он инициализируется на более низком уровне в редукторах. Когда вы вспомните последние уроки, извлеченные из главы «Продвинутые редукторы», вы заметите, что вы позаботились о начальном состоянии. Теперь `todoReducer` все еще работает в подсостоянии `todos`, а новый `filterReducer` работает в подсостоянии` filter`. В качестве третьего и последнего шага вы должны объединить оба редуктора, чтобы получить этот промежуточный слой подсостояний в первую очередь. В JS Bin у вас есть Redux, доступный как глобальная переменная, чтобы получить функцию `combReducer`. В противном случае вы можете импортировать его с помощью JavaScript ES6.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const rootReducer = Redux.combineReducers({
  todoState: todoReducer,
  filterState: filterReducer,
});
~~~~~~~~

Теперь вы можете использовать комбинированный `rootReducer` в инициализации хранилища.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const store = createStore(rootReducer);
~~~~~~~~

При повторном запуске приложения все должно работать. Теперь вы также можете использовать `filterReducer`.

{title="Code Playground",lang="javascript"}
~~~~~~~~
store.dispatch(doAddTodo('0', 'learn redux'));
store.dispatch(doAddTodo('1', 'learn mobx'));
store.dispatch(doToggleTodo('0'));
# leanpub-start-insert
store.dispatch(doSetFilter('COMPLETED'));
# leanpub-end-insert
~~~~~~~~

Приложение Todo предпочитает начальное состояние в редукторах начальному состоянию в `createStore()`. Кроме того, оно не будет использовать вложенный `todoEntityReducer` ради простоты иерархии редуктора. Хотя сейчас есть один промежуточный слой из-за комбинированных редукторов.

[Окончательное приложение Todo можно найти в этом JS Bin](https://jsbin.com/kopohur/30/edit?html,js,console). Вы можете провести дальнейшие эксперименты с ним, прежде чем перейти к следующей главе.
