## Продвинутые Редукторы

Apart from the advanced topics about actions, there is more to know about reducers, too. Again, not everything is mandatory, but you should at least know about the following things to embrace best practices, common usage patterns and practices on scaling your state architecture.
Помимо продвинутых тем о действиях, есть еще много информации о редукторах. Опять же, не все является обязательным, но вы должны по крайней мере знать о следующих вещах, чтобы охватить лучшие практики, общие шаблоны использования и практики по масштабированию архитектуры состояния.

### Начальное состояние

So far, you have provided your store with an initial state. It was an empty list of todos.
До сих пор вы предоставили вашему хранилищу начальное состояние. Это был пустой список задач.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const store = createStore(reducer, []);
~~~~~~~~

That's the initial state for the whole Redux store. However, you can apply the initial state on a more fine-grained level. Before you dispatch your first action, the Redux store will initialize by running through all reducers once. You can try it by removing all of your dispatches in the editor and add a `console.log()` in the reducer. You will see that it runs with an initializing action once, even though there is no dispatched action.
Это начальное состояние всего хранилища Redux. Тем не менее, вы можете применить начальное состояние на более мелком уровне. Перед отправкой вашего первого действия, хранилище Redux будет инициализировано, пройдя все редукторы один раз. Вы можете попробовать это, удалив все ваши отправки в редакторе и добавив `console.log()` в редуктор. Вы увидите, что он запускается с инициализирующим действием один раз, хотя отправленного действия нет.

The initializing action, that is received in the reducer, is accompanied by the initial state that is specified in the `createStore()` function. However, if you leave out the initial state in the store initialization, the incoming state in the reducer will be `undefined`.
Действие инициализации, полученное в редукторе, сопровождается начальным состоянием, которое указано в функции `createStore()`. Однако если вы пропустите начальное состояние при инициализации хранилища, входящее состояние в редукторе будет `undefined`.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const store = createStore(reducer);
~~~~~~~~

That's where you can opt-in to specify an initial state on a more fine-grained level. If the incoming state is `undefined`, you can default with a [JavaScript ES6 default parameter](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Default_parameters) to a default initial state.
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

The initial state will be the same as before, but you defined it on a more fine-grained level. Later on, that will help you when you specify more than one reducer and your state object becomes more complex because it is split up into multiple reducers. Then every reducer can define its fine-grained initial state.
Начальное состояние будет таким же, как и раньше, но вы определили его на более детальном уровне. Позже это поможет вам, когда вы укажете более одного редуктора, и ваш объект состояния станет более сложным, потому что он разделен на несколько редукторов. Тогда каждый редуктор может определить свое детальное начальное состояние.

### Вложенные структуры данных

The initial state is an empty list of todos. However, in a growing application you want to operate on more than todos. You might have a `currentUser` object that represents the logged in user in your application. In addition, you want to have a `filter` property to filter todos by their `completed` property. In a growing application, more objects and arrays will gather in the global state. That's why your initial state shouldn't be an empty array but an object that represents the state object. This object then has nested properties for `todos`, `currentUser` and `filter`.
Начальное состояние - пустой список задач. Однако в растущем приложении вы хотите работать не только над задачами. У вас может быть объект `currentUser`, который представляет зарегистрированного пользователя в вашем приложении. Кроме того, вы хотите иметь свойство `filter` для фильтрации задач по их свойству` completed`. В растущем приложении все больше объектов и массивов будут собираться в глобальном состоянии. Вот почему ваше начальное состояние должно быть не пустым массивом, а объектом, представляющим объект состояния. Этот объект затем имеет вложенные свойства для `todos`,` currentUser` и `filter`.

Disregarding the initial state in the reducer from the last chapter, you would define your initial state in the store like in the following:
Не принимая во внимание начальное состояние редуктора из предыдущей главы, вы должны определить свое начальное состояние в хранилище, как показано ниже:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const initialState = {
  todos: [],
};
const store = createStore(reducer, initialState);
~~~~~~~~

Now you can use the space horizontally in your `initialState` object. It might grow to the following at some point:
Теперь вы можете использовать пространство горизонтально в вашем объекте `initialState`. В какой-то момент это может вырасти до следующего:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const initialState = {
  currentUser: null,
  todos: [],
  filter: 'SHOW_ALL',
};
~~~~~~~~

If you get back to your Todo application, you will have to adjust your reducer. The reducer deals with a list of todos as state, but now it is a complex global state object.
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

Those **nested data structures** are fine in Redux, but you want to avoid **deeply nested data structures**. As you can see, it adds complexity to create your new state object. Another chapter which you will read up later in the book will pick up this topic again. It will showcase how you can avoid deeply nested data structures by using a neat helper library with the name normalizr.
Эти **вложенные структуры данных** хороши в Redux, но вы должны избегать **глубоко вложенных структур данных**. Как вы можете видеть, это добавляет сложности для создания вашего нового объекта состояния. Другая глава, которую вы прочтете позже в книге, снова поднимет эту тему. Он продемонстрирует, как вы можете избежать глубоко вложенных структур данных, используя лаконичную вспомогательную библиотеку с именем normalizr.

### Комбинированный редуктор

The next chapter is crucial to understand the principles of scaling state by using substates in Redux. In the previous chapters, you have heard about multiple reducers, but haven't used them yet. As you have learned, a reducer grows horizontally when it deals with more than one action type. But this doesn't scale at some point. You want to split your reducer up into two reducers or introduce another reducer right from the beginning. Imagine you had a reducer that served your todos, but you want to have a reducer for the `filter` state for your todos eventually. Another use case could be to have a reducer for the `currentUser` that is logged in into your Todo application.
Следующая глава крайне важна для понимания принципов масштабирования состояния используя подсостояния в Redux. В предыдущих главах вы слышали о нескольких редукторах, но еще не использовали их. Как вы узнали, редуктор растет горизонтально, когда имеет дело с более чем одним типом действия. Но это не масштабируется в какой-то момент. Вы хотите разделить ваш редуктор на два редуктора или ввести другой редуктор с самого начала. Представьте, что у вас есть редуктор, который обслуживал ваши задачи, но вы хотите иметь редуктор для состояния `filter` для ваших задач. Другой вариант использования может состоять в том, чтобы иметь редуктор для `currentUser`, который зарегистрирован в вашем приложении Todo.

You can already see a pattern on how to separate your reducers. It is usually by domain like todos, filter, or user. A todo reducer might be responsible to add, remove, edit and complete todos. A filter reducer is responsible to manage the filter state of your todos. A user reducer cares about user entities that could be the `currentUser` who is logged in in your application or a list of users who are assigned to todos. That's where you could again split up the user reducer to a `currentUser` reducer and a `assignedUsers` reducer. You can imagine how this approach, introducing reducers by domain, scales very well.
Вы уже можете увидеть шаблон, как разделить ваши редукторы. Обычно это области применения такие как задачи, фильтры или пользователи. Редуктор задач может быть ответственным за добавление, удаление, редактирование и заполнение задач. Редуктор фильтр отвечает за управление состоянием фильтра ваших задач. Пользовательский редуктор заботится о пользовательских объектах, которыми может быть `currentUser`, пользователь вошедший в ваше приложение, или список пользователей, которым назначены задачи. Вот где вы могли бы снова разделить пользовательский редуктор на редуктор `currentUser` и редуктор` assignUsers`. Вы можете представить, как этот подход, представляя редукторы по областям, очень хорошо масштабируется.

Let's enter **combined reducers** to enable you using multiple reducers. Redux gives you a helper to combine multiple reducers into one root reducer: `combineReducers()`. The function takes an object as argument which can have multiple reducers assigned to their state which they are going to manage.
Давайте введем **комбинированные редукторы**, чтобы вы могли использовать несколько редукторов. Redux дает вам помощника для объединения нескольких редукторов в один корневой редуктор: `combineReducers()`. Функция принимает в качестве аргумента объект, которому может быть присвоено несколько редукторов для их состояния, которым они будут управлять.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const rootReducer = combineReducers({
  todoState: todoReducer,
  filterState: filterReducer,
});
~~~~~~~~

Afterward, the `rootReducer` can be used to initialize the Redux store instead of the single todo reducer. After all, only one reducer is needed to initialize the Redux store. So it has to be a combination of multiple reducers when having more than one reducer.
После этого `rootReducer` может использоваться для инициализации хранилища Redux вместо редуктора todo. В конце концов, для инициализации хранилища Redux необходим только один редуктор. Таким образом, это должно быть сочетание нескольких редукторов при наличии более одного редуктора.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const store = createStore(rootReducer);
~~~~~~~~

That's it for the initialization of the Redux store with combined reducers. But what about the reducers themselves? There is already one reducer that cares about the todos, you would only have the rename it.
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

The `filterReducer` is not there yet. It would look something like the following to set a filter for the todos (e.g. SHOW_ALL, SHOW_COMPLETED, SHOW_INCOMPLETED):
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

Now, there comes the important part: The `combineReducers()` function introduces an intermediate state layer for the global state object. The global state object, when using the combined reducers as shown before, would look like the following:
Теперь перейдем к важной части: функция `combineReducers()` вводит промежуточный уровень состояния для объекта глобального состояния. Глобальный объект состояния при использовании комбинированных редукторов, как показано выше, будет выглядеть следующим образом:

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  todoState: ...,
  filterState: ...,
}
~~~~~~~~

The property keys for the intermediate layer are those defined in the `combineReducers()` function. However, in your reducers the incoming state is not the global state object anymore. It is their defined substate from the `combinedReducers()` function. The `todoReducer` doesn't know anything about the `filterState` and the `filterReducer` doesn't know anything about the `todoState`.
Ключи свойств для промежуточного уровня - это те, которые определены в функции `combineReducers()`. Однако в ваших редукторах входящее состояние больше не является глобальным объектом состояния. Это их определенное подсостояние из функции `combinedReducers() `. `todoReducer` ничего не знает о` filterState`, а `filterReducer` ничего не знает о `todoState`.

The `filterReducer` and `todoReducer` can use the JavaScript ES6 default parameter to define their initial state.
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

Now your `apply` function in the reducers have to operate on these substates again.
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

It seems like a lot of hassle to define multiple reducers and combining them eventually. But it is crucial knowledge to split up your global state into substates. These substates can be managed by their reducers who only operate on these substates. In addition, each reducer can be responsible to define the initial substate.
Кажется, очень сложно определить несколько редукторов и в конечном итоге объединить их. Но это крайне важные сведения, чтобы разбить ваше глобальное состояние на подсостояния. Этими подсостояниями могут управлять их редукторы, которые работают только в этих подсостояниях. Кроме того, каждый редуктор может отвечать за определение начального подсостояния.

### Разъяснение по Начальному состоянию

The last chapters moved around the initial state initialization from `createStore()` to the reducer(s) a few times. You might wonder where to initialize your state after all. Therefore, you have to distinguish whether you are using combined reducers or only one plain reducer.
Последние главы несколько раз перемещались от инициализации начального состояния в `createStore()` до инициализации в редукторе(ах). Вы можете спросить, где инициализировать ваше состояние в конце концов. Для этого вы должны понимать, использовать ли комбинированные редукторы или только один простой редуктор.

**One plain reducer:** When using only one plain reducer, the initial state in `createStore()` dominates the initial state in the reducer. The initial state in the reducer only works when the incoming initial state is `undefined` because then it can apply the default state from the default parameter. But the initial state is already defined in `createStore()` and thus utilized by the reducer.
**Один простой редуктор:** При использовании только одного простого редуктора начальное состояние в `createStore ()` доминирует над начальным состоянием в редукторе. Начальное состояние в редукторе работает только тогда, когда входящее начальное состояние равно `undefined`, потому что тогда оно может применить состояние по умолчанию из параметра по умолчанию. Но начальное состояние уже определено в `createStore ()` и, таким образом, используется редуктором.

**Combined reducers:** When using combined reducers, you can embrace a more nuanced usage of the state initialization. The initial state object that is used for the `createStore()` function doesn't have to include all substates that are introduced by the `combineReducers()` function. Thus, when a substate is `undefined`, the reducer can define the default substate. Otherwise, the default substate from the `createStore()` is used.
**Комбинированные редукторы:** При использовании комбинированных редукторов вы можете использовать более детальное использование инициализации состояния. Объект начального состояния, который используется для функции `createStore()`, не обязательно должен включать все подсостояния, которые вводятся функцией `combineReducers()`. Таким образом, когда подсостояние `undefined`, редуктор может определить подсостояние по умолчанию. В противном случае используется подсостояние по умолчанию из `createStore()`.

### Вложенные редукторы

By now, you know two things about scaling reducers in a growing application that demand sophisticated state management:
By now, you know two things about scaling reducers in a growing application that demand sophisticated state management:

* a reducer can care about different action types
* редуктор может заботиться о различных типах действий
* a reducer can be split up into multiple reducers yet be combined as one root reducer for the store initialization
* редуктор может быть разделен на несколько редукторов, но может быть объединен в один корневой редуктор для инициализации хранилища.

These steps are used to scale the reducers horizontally (even though combined reducers add at least one vertical level). A reducer operates on the global state or on a substate when using combined reducers. However, you can use nested reducers to introduce vertically levels of substate too.
Эти шаги используются для горизонтального масштабирования редукторов (даже если комбинированные редукторы добавляют как минимум один вертикальный уровень). Редуктор работает в глобальном состоянии или в подсостоянии при использовании комбинированных редукторов. Тем не менее, вы можете использовать вложенные редукторы, чтобы вводить вертикальные уровни подсостояния.

Take for example the `todoReducer` that operates on a list of todos. From a technical perspective, a list of todos has single todo entities. So why not introduce a nested reducer that deals with the todo substate as entities?
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

You can use nested reducers to introduce intermediate layers for substates. In addition, they can be reused. You might run into cases where you can reuse a nested reducer somewhere else. However, while nested reducers can give you a better picture on your state, they add more levels of complexity to your state, too. You should follow the practice of not nesting your state too deeply in the first place. Then you won't run into nested reducers often.
Вы можете использовать вложенные редукторы, чтобы ввести промежуточные уровни для подсостояний. Кроме того, они могут быть использованы повторно. Вы можете столкнуться со случаями, когда вы можете повторно использовать вложенный редуктор в другом месте. Однако, хотя вложенные редукторы могут дать вам лучшее представление о вашем состоянии, они также повышают уровень сложности вашего состояния. Во-первых, вы должны придерживаться практики не слишком глубоко вкладывать свое состояние. Тогда вы не будете часто сталкиваться с вложенными редукторами.

### Практика: Redux с продвинутыми Редукторами

Let's dip into the Redux Playground again with the acquired knowledge about reducers. Again, you can take the [JS Bin project that you have done in the last chapter](https://jsbin.com/kopohur/29/edit?html,js,console). The project will be used to show the advanced reducers. You can try it on your own. Otherwise, the following part will guide you through the refactorings. First, let's add the second reducer to filter the todos.
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

The reducer has an initial state. There is no action yet that serves the reducer though. You can add a simple action creator that only receives a string as argument for the filter property. You will experience later on, how this can be used in a real application.
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

The initial state isn't initialized in the `createStore()` function anymore. It is initialized on a more fine-grained level in the reducers. When you recap the last lessons learned from the advanced reducers chapter, you will notice that you spared the back and forth with the initial state. Now, the `todoReducer` still operates on the `todos` substate and the new `filterReducer` operates on the `filter` substate. As third and last step, you have to combine both reducers to get this intermediate layer of substates in the first place. In the JS Bin you have Redux available as global variable to get the `combineReducer` function. Otherwise, you could import it with JavaScript ES6.
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

The Todo application favors initial state in reducers over initial state in `createStore()`. In addition, it will not use a nested `todoEntityReducer` for the sake of keeping the reducer hierarchy simple. Even though there is one intermediate state layer because of the combined reducers now.
Приложение Todo предпочитает начальное состояние в редукторах начальному состоянию в `createStore()`. Кроме того, оно не будет использовать вложенный `todoEntityReducer` ради простоты иерархии редуктора. Хотя сейчас есть один промежуточный слой из-за комбинированных редукторов.

The [final Todo application can be found in this JS Bin](https://jsbin.com/kopohur/30/edit?html,js,console). You can do further experiments with it before continuing with the next chapter.
[Окончательное приложение Todo можно найти в этом JS Bin](https://jsbin.com/kopohur/30/edit?html,js,console). Вы можете провести дальнейшие эксперименты с ним, прежде чем перейти к следующей главе.
