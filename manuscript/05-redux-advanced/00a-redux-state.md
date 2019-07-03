# Извлечение и структура состояния Redux

В предыдущих главах вы узнали о одиночном Redux и Redux в React. Вы уже можете создавать приложения малого и среднего размера. Прежде чем углубиться в Redux, я рекомендую вам поэкспериментировать со своими недавними знаниями и применить их в своих приложениях. Если вы перейдете прямо к следующим главам, у вас может сложиться впечатление, что Redux излишний (что и есть для большинства приложений). Однако в следующих главах вы получите подробные рекомендации по масштабированию управления состоянием с помощью Redux в более крупных приложениях. Есть несколько методов, которые вы сможете применить тогда.

Следующая глава проведет вас через более сложные темы в Redux для управления вашим состоянием. Вы познакомитесь с промежуточным программным обеспечением в Redux, узнаете больше о нормализованной и неизменной структуре состояний и о том, как улучшенным образом извлечь подсостояние из глобального состояния с помощью селекторов.

## Промежууточное программное обеспечение в Redux

В Redux вы можете использовать промежуточное ПО. Каждое отправленное действие в Redux пройдет через это промежуточное ПО. Вы можете включить определенную функцию между отправкой действия и моментом, когда оно достигает редуктора.

Существуют полезные библиотеки для включения функций в ваше промежуточное ПО Redux. Далее вы познакомитесь с одним из них: [redux-logger](https://github.com/evgenyrodionov/redux-logger). Когда вы используете его, это ничего не меняет в вашем приложении. Но это облегчит вашу жизнь как разработчика при работе с Redux. Что оно делает? Он просто регистрирует действия в консоли разработчика вашего браузера с помощью `console.log()`. Как разработчику, он дает вам четкое представление о том, какое действие отправляется и как структурируется предыдущее и новое состояние.

Но где применить это промежуточное ПО в Redux? Это магазин Redux, который можно инициализировать с ним. Функциональность `createStore()` из Redux принимает в качестве третьего аргумента так называемый улучшитель. Библиотека Redux поставляется с одним из следующих улучшителей: `applyMiddleware()`.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { applyMiddleware, createStore } from 'redux';

const store = createStore(
  reducer,
  undefined,
  applyMiddleware(...)
);
~~~~~~~~

Если у вас нет начального состояния для вашего состояния Redux, вы можете использовать для него `undefined`. Теперь при использовании redux-logger вы можете передать экземпляр `logger` в функцию `applyMiddleware()`.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { applyMiddleware, createStore } from 'redux';
# leanpub-start-insert
import { createLogger } from 'redux-logger';

const logger = createLogger();
# leanpub-end-insert

const store = createStore(
  reducer,
  undefined,
# leanpub-start-insert
  applyMiddleware(logger)
# leanpub-end-insert
);
~~~~~~~~

Вот и все. Теперь каждое действие должно отображаться в консоли разработчика вашего браузера при их отправке. И, таким образом, изменения вашего состояния становятся более предсказуемыми при разработке, не логгируя каждое действие самостоятельно.

Функциональность `applyMiddleware()` принимает любое количество экземпляров промежуточного программного обеспечения: `applyMiddleware(firstMiddleware, secondMiddleware, ...);`. Действие будет проходить через все экземпляры промежуточного программного обеспечения, прежде чем оно достигнет редукторов. Иногда вы должны убедиться, что применяете их в правильном порядке. Например, промежуточное программное обеспечение `redux-logger` должно быть последним в цепочке для вывода правильных действий и состояний.

Тем не менее, это только промежуточное программное обеспечение redux-logger. На пути к реализации приложений Redux вы наверняка узнаете больше о полезных функциях, которые можно применять с промежуточным ПО Redux. Большинство этих функций уже реализованы в библиотеках, которые вы найдете опубликованными с помощью npm. Например, асинхронные действия в Redux возможны при использовании промежуточного программного обеспечения Redux. Эти асинхронные действия будут объяснены позже в этой книге.

## Неизменное состояние

Redux охватывает неизменное состояние. Ваши редукторы всегда будут возвращать новый объект состояния. Вы никогда не будете мутировать входящее состояние. Поэтому вам, возможно, придется привыкнуть к различным функциям и синтаксису JavaScript, чтобы охватить неизменные структуры данных.

До сих пор вы использовали встроенные функции JavaScript для сохранения неизменности ваших структур данных. Например, `array.map()` и `array.concat(item)` для массивов или `Object.assign()` для объектов. Все эти функции возвращают новые экземпляры массивов или объектов без изменения старых массивов и объектов. Часто вам нужно прочитать официальную документацию JavaScript, чтобы убедиться, что они возвращают новый экземпляр массива или объекта. В противном случае вы нарушили бы ограничения Redux, потому что изменили бы предыдущий экземпляр, который в этом случае часто является состоянием или действием.

Но этим не заканчивается. Вы должны знать о своих инструментах, чтобы сохранить неизменность структур данных в JavaScript. Существует несколько сторонних библиотек, которые могут помочь вам сохранить их неизменяемыми.

* [immutable.js](https://github.com/facebook/immutable-js)
* [immer.js](https://github.com/mweststrate/immer)
* [mori.js](https://github.com/swannodette/mori)
* [seamless-immutable.js](https://github.com/rtfeldman/seamless-immutable)
* [baobab.js](https://github.com/Yomguithereal/baobab)

Но все они имеют три недостатка. Во-первых, они добавляют еще один уровень сложности вашему приложению. Во-вторых, вы должны изучить еще одну библиотеку. И в-третьих, вы должны извлекать и повторно вносить ваши данные, потому что большинство этих библиотек обертывают ваши ванильные объекты JavaScript и массивы в неизменяемый объект данных и массив для конкретной библиотеки. Это неизменный объект/массив данных в вашем хранилище Redux, но как только вы захотите использовать его в React, вам придется преобразовать его в простой объект/массив JavaScript. Лично я бы рекомендовал использовать такие библиотеки только в двух сценариях:

* Вам неудобно поддерживать неизменность своих структур данных с помощью JavaScript ES5, JavaScript ES6 и более поздних версий.
* Вы хотите повысить производительность неизменяемых структур данных при использовании огромных объемов данных.

Если оба утверждения ложны, я настоятельно рекомендую вам придерживаться простого JavaScript. Как вы уже видели, встроенные функции JavaScript уже очень помогают поддерживать неизменность структур данных. В JavaScript ES6 и более поздних версиях вы получаете еще одну функциональность для сохранения неизменности ваших структур данных: [spread операторы](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Spread_operator). Распространение объекта или массива в новый объект или новый массив всегда дает вам новый объект или новый массив.

Помните ли вы, как вы добавили новый элемент todo или как вы выполнили элемент todo в ваших редукторах?

{title="Code Playground",lang="javascript"}
~~~~~~~~
// adding todo
const todo = Object.assign({}, action.todo, { completed: false });
const newTodos = todos.concat(todo);

// toggling todo
const newTodos = todos.map(todo =>
  todo.id === action.todo.id
    ? Object.assign({}, todo, { completed: !todo.completed })
    : todo
  );
~~~~~~~~

Если вы добавили больше JavaScript ES6 с помощью spread оператора, вы можете сделать их еще более краткими.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// adding todo
const todo = { ...action.todo, completed: false };
const newTodos = [ ...todos, todo ];

// toggling todo
const newTodos = todos.map(todo =>
  todo.id === action.todo.id
    ? { ...todo, completed: !todo.completed }
    : todo
  );
~~~~~~~~

JavaScript дает вам достаточно инструментов, чтобы ваши структуры данных оставались неизменными. Нет необходимости использовать стороннюю библиотеку, за исключением двух упомянутых вариантов использования. Тем не менее, может быть третий вариант использования, где такая библиотека может помочь: глубоко вложенные структуры данных в Redux, которые необходимо сохранять неизменяемыми. Это правда, что становится сложнее поддерживать неизменность структур данных, когда они вложены. Но, как упоминалось ранее в этой книге, плохая практика - иметь глубоко вложенные структуры данных в Redux. Вот тут-то и вступает в игру следующая глава книги, которую можно использовать, чтобы сохранить структуру данных в хранилище Redux.

## Нормализованное состояние

Лучшая практика в Redux - это плоская структура состояния. Вы не захотите поддерживать неизменную структуру для своего состояния, когда оно глубоко вложено. Это становится утомительным и нечитаемым даже со spread операторами. Но часто вы не можете контролировать свою структуру данных, потому что она поступает из внутреннего приложения с помощью его API. При наличии глубоко вложенной структуры данных у вас есть два варианта:

* сохранить его в хранилище как есть и отложить проблему (не хорошо)
* сохранение в виде нормализованной структуры данных в магазине (хорошо)

Вы должны пытаться выполнить второй вариант по умолчанию. Вы решаете проблему только один раз, и все последующие части вашего приложения будут вам благодарны. Давайте рассмотрим один сценарий, чтобы проиллюстрировать нормализацию данных. Представьте, что у вас есть следующая вложенная структура данных:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const todos = [
  {
    id: '0',
    name: 'create redux',
    completed: true,
    assignedTo: {
      id: '99',
      name: 'Dan Abramov',
    },
  },
  {
    id: '1',
    name: 'create mobx',
    completed: true,
    assignedTo: {
      id: '77',
      name: 'Michel Weststrate',
    },
  }
];
~~~~~~~~

Оба создателя библиотеки, Дан Абрамов и Мишель Вестстрайт, проделали большую работу: они создали популярные библиотеки для управления состоянием. Первым вариантом, как уже упоминалось, будет сохранение задач в том виде, в каком они находятся в магазине. Сами задачи могут иметь глубоко вложенную информацию об объекте `assignTo` в объекте `todo`. Теперь, если действие хотело бы исправить назначенного пользователя, редуктор должен был бы иметь дело с глубоко вложенной структурой данных. Давайте добавим Эндрю Кларка как создателя Redux.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const ASSIGNED_TO_CHANGE = 'ASSIGNED_TO_CHANGE';

store.dispatch({
  type: ASSIGNED_TO_CHANGE,
  payload: {
    todoId: '0',
    name: 'Dan Abramov and Andrew Clark',
  },
});

function todoReducer(state = [], action) {
  switch(action.type) {
    case ASSIGNED_TO_CHANGE : {
      return applyChangeAssignedTo(state, action);
    }

    ...

    default : return state;
  }
}

function applyChangeAssignedTo(state, action) {
  return state.map(todo => {
    if (todo.id === action.payload.todoId) {
      const assignedTo = { ...todo.assignedTo, name: action.payload.name };
      return { ...todo, assignedTo };
    } else {
      return todo;
    }
  });
}
~~~~~~~~

Это приведет к следующему списку задач в вашем хранилище Redux после отправки действия:

{title="Code Playground",lang="javascript"}
~~~~~~~~
const todos = [
  {
    id: '0',
    name: 'create redux',
    completed: true,
    assignedTo: {
      id: '99',
      name: 'Dan Abramov and Andrew Clark',
    },
  },
  {
    id: '1',
    name: 'create mobx',
    completed: true,
    assignedTo: {
      id: '77',
      name: 'Michel Weststrate',
    },
  }
];
~~~~~~~~

Как вы можете видеть из редуктора, чем дальше вы должны проникнуть в глубоко вложенную структуру данных, тем больше вы должны быть осторожны, чтобы сохранить вашу структуру данных неизменной. Каждый уровень вложенных данных добавляет более утомительную работу по его обслуживанию. Следовательно, вы можете использовать библиотеку [normalizr](https://github.com/paularmstrong/normalizr), чтобы сгладить (нормализовать) ваше состояние. Библиотека использует определения схемы для преобразования глубоко вложенных структур данных в словари, которые имеют сущности и соответствующий список идентификаторов.

Как это будет выглядеть? Давайте возьмем предыдущий список элементов todo в качестве примера. Во-первых, вам нужно будет определить схемы для ваших объектов только один раз:

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { schema } from 'normalizr';

const assignedToSchema = new schema.Entity('assignedTo');

const todoSchema = new schema.Entity('todo', {
  assignedTo: assignedToSchema,
});
~~~~~~~~

Во-вторых, вы можете нормализовать ваши данные, когда захотите:

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { schema, normalize } from 'normalizr';
# leanpub-end-insert

const assignedToSchema = new schema.Entity('assignedTo');

const todoSchema = new schema.Entity('todo', {
  assignedTo: assignedToSchema,
});

# leanpub-start-insert
const normalizedData = normalize(todos, [ todoSchema ]);
# leanpub-end-insert
~~~~~~~~

Вывод будет следующим:

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  entities: {
    assignedTo: {
      77: {
        id: "77",
        name: "Michel Weststrate"
      },
      99: {
        id: "99",
        name: "Dan Abramov"
      }
    },
    todo: {
      0: {
        assignedTo: "99",
        completed: true,
        id: "0",
        name: "create redux"
      },
      1: {
        assignedTo: "77",
        completed: true,
        id: "1",
        name: "create mobx"
      }
    }
  },
  result: ["0", "1"]
}
~~~~~~~~

Глубоко вложенные данные стали плоской структурой данных, сгруппированных по сущностям. Каждый объект может ссылаться на другой объект по его идентификатору. Это как если бы вы хранили эти объекты в базе данных. Они отделены сейчас. После этого в вашем приложении Redux у вас может быть один редуктор, который хранит и имеет дело с сущностями `assignTo`, и один редуктор, который имеет дело с сущностями `todo`. Структура данных является плоской и сгруппированной по сущностям, что облегчает доступ и манипулирование ею.

Есть еще одно преимущество в нормализации ваших данных. Когда ваши данные не нормализованы, сущности часто дублируются в вашей вложенной структуре данных. Представьте себе следующий объект с задачами.

{title="Code Playground",lang="javascript"}
~~~~~~~~
const todos = [
  {
    id: '0',
    name: 'write a book',
    completed: true,
    assignedTo: {
      id: '55',
      name: 'Robin Wieruch',
    },
  },
  {
    id: '1',
    name: 'call it taming the state in react',
    completed: true,
    assignedTo: {
      id: '55',
      name: 'Robin Wieruch',
    },
  }
];
~~~~~~~~

Если вы сохраняете такие денормализованные данные в своем хранилище Redux, вы, скорее всего, столкнетесь с серьезной проблемой: представьте, что вы хотите обновить свойство name `Robin Wieruch` для всех свойств `assignTo`. Вам нужно будет пройти через все задачи, чтобы обновить все свойства `assignedTo` с идентификатором `55`. Проблема: единого источника правды не существует. Скорее всего, вы забудете обновить сущность и в конечном итоге перейдете в устаревшее состояние. Следовательно, наилучшей практикой является сохранение вашего состояния нормализованным, чтобы каждая сущность была единственным источником правды. При обновлении одного единственного источника истины не будет дублирования сущностей и, следовательно, нет устаревшего состояния, потому что каждая задача будет ссылаться на обновленную сущность `assignTo`:

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  entities: {
    assignedTo: {
      55: {
        id: "55",
        name: "Robin Wieruch"
      }
    },
    todo: {
      0: {
        assignedTo: "55",
        completed: true,
        id: "0",
        name: "write a book"
      },
      1: {
        assignedTo: "55",
        completed: true,
        id: "1",
        name: "call it taming the state in react"
      }
    }
  },
  result: ["0", "1"]
}
~~~~~~~~

В заключение, нормализация вашего состояния имеет два преимущества. Он сохраняет состояние вашего состояния и, таким образом, легче управляется с помощью неизменяемых структур данных. Кроме того, он группирует сущности в единые источники правды без каких-либо дубликатов. Когда вы нормализуете свое состояние, вы автоматически получите группы объектов, которые могут привести к тому, что их собственные редукторы будут управлять ими. Однако вы не хотите начинать с нормализации своего состояния с самого начала. Сначала вы должны попытаться внедрить сам Redux в ваше приложение. Если вы испытываете трудности при изменении состояния, потому что оно глубоко вложено, или у вас возникают проблемы с обновлением отдельных сущностей, потому что нет единого источника правды, вы можете захотеть ввести нормализацию состояния в вашем приложении Redux.

Прежде чем вы примените нормализацию в своем собственном приложении Todo в качестве упражнения, существует еще одно преимущество при нормализации вашего состояния с помощью такой библиотеки, как normalizr. Речь идет о денормализации: как компоненты получают нормализованное состояние? Вы узнаете это в следующей главе.

## Селекторы

В Redux существует концепция селекторов для извлечения производных свойств из вашего состояния. Селектор - это функция, которая принимает состояние в качестве аргумента и возвращает его подсостояние или производные свойства. Может случиться так, что они возвращают только подсостояние вашего глобального состояния или что они уже предварительно обрабатывают ваше состояние для возврата производных свойств. Функция может принимать необязательные аргументы для поддержки процесса выбора.

### Чистые селекторы

Селекторы обычно следуют одному синтаксису. Обязательным аргументом селектора является состояние, из которого он должен выбирать. Могут быть необязательные аргументы, которые играют вспомогательную роль для выбора подсостояния или производных свойств.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// selector
(state) => derived properties
// selector with optional arguments
(state, args) => derived properties
~~~~~~~~

Селекторы не обязательны. При рассмотрении всех частей Redux обязательными являются только действие(я), редуктор(ы) и хранилище Redux. Подобно создателям действий, селекторы могут быть использованы для достижения улучшенного опыта разработчика в архитектуре Redux, но вам не обязательно их использовать. Как выглядит селектор? Как уже упоминалось, это простая функция, которая может жить где угодно в вашем приложении. Тем не менее, вы могли бы использовать его, возможно импортировать его при использовании Redux в React, в вашей функции `mapStateToProps()`. Так что вместо получения состояния явно:

{title="Code Playground",lang="javascript"}
~~~~~~~~
function mapStateToProps(state) {
  return {
    todos: state.todoState,
  };
}
~~~~~~~~

Вы бы извлекли его неявно через селектор:

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function getTodos(state) {
  return state.todoState;
}
# leanpub-end-insert

function mapStateToProps(state) {
  return {
# leanpub-start-insert
    todos: getTodos(state),
# leanpub-end-insert
  };
}
~~~~~~~~

Это похоже на концепцию действия и редуктора. Вместо того, чтобы манипулировать состоянием непосредственно в хранилище Redux, вы будете использовать действие(я) и редуктор(ы) для его косвенного изменения. То же самое относится к селекторам, которые не извлекают производные свойства прямо, но косвенно из глобального состояния.

Вы можете спросить: почему селекторы являются преимуществом? Есть несколько плюсов. Селектор можно использовать повторно. Вы столкнетесь со случаями, когда выбираете производные свойства или чаще выполняете подстановку из глобального состояния. Например, у вас может быть несколько мест в приложении, где вы хотите показывать только завершенные элементы задач. И это всегда хороший знак, чтобы использовать функцию в первую очередь. Кроме того, селекторы могут тестироваться отдельно. Они являются чистыми функциями и, следовательно, легко тестируемой частью вашего приложения и общей архитектуры Redux. Наконец, что не менее важно, получение свойств из состояния может стать сложной задачей в масштабирующем приложении. Как уже упоминалось, селектор может получить необязательные аргументы для получения более сложных свойств из состояния. Селекторная функция сама по себе станет более сложной, но она будет заключена в одну функцию, а не, например, в React и Redux, в несколько функций `mapStateToProps()`.

### Денормализация состояния в селекторах

В предыдущей главе о нормализации было одно преимущество, которое осталось необъяснимым. Речь идет о выборе нормализованного состояния из вашего слоя состояний, чтобы передать его слою представления. Лично я бы сказал, что нормализованная структура состояний делает гораздо более удобным выбор из нее подсостояния. Когда вы вспоминаете структуру нормализованного состояния, это выглядело примерно так:

{title="Code Playground",lang="javascript"}
~~~~~~~~
{
  entities: ...
  ids: ...
}
~~~~~~~~

Например, в реальном рабочем приложении это будет выглядеть следующим образом:

{title="Code Playground",lang="javascript"}
~~~~~~~~
// состояние
[
  { id: '0', name: 'learn redux' },
  { id: '1', name: 'learn mobx' },
]

// нормализванное состояние
{
  entities: {
    0: {
      id: '0',
      name: 'learn redux',
    },
    1: {
      id: '1',
      name: 'learn redux',
    },
  },
  ids: ['0', '1'],
}
~~~~~~~~

Если вы вспомните главу Redux in React, вы передали список задач из компонента `TodoList`, поскольку он является связанным компонентом, вплоть до всего дерева компонентов. Как бы вы решили это с нормализованным состоянием?

Предполагая, что редуктор хранит состояние в нормализованной неизменной структуре данных, вы бы только передали список идентификаторов todo `ids` вашему компоненту `TodoList`. Поскольку этот компонент управляет списком, а не самими сущностями, имеет смысл, что он получает список только со ссылками на сущности.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function TodoList({ todosAsIds }) {
  return (
    <div>
      {todosAsIds.map(todoId => <ConnectedTodoItem
        key={todoId}
        todoId={todoId}
      />)}
    </div>
  );
}

function getTodosAsIds(state) {
  return state.todoState.ids;
}

function mapStateToProps(state) {
  return {
    todosAsIds: getTodosAsIds(state),
  };
}

const ConnectedTodoList = connect(mapStateToProps)(TodoList);
~~~~~~~~

Теперь компонент `ConnectedTodoItem`, который уже передает обработчик `onToggleTodo()` через функцию `mapDispatchToProps()` в свой простой компонент `TodoItem`, выбирает сущность todo, соответствующий входящему свойству `todoId`.

{title="Code Playground",lang="javascript"}
~~~~~~~~
function getTodoAsEntity(state, id) {
  return state.todoState.entities[id];
}

function mapStateToProps(state, props) {
  return {
    todo: getTodoAsEntity(state, props.todoId),
  };
}

function mapDispatchToProps(dispatch) {
  return {
     onToggleTodo: id => dispatch(doToggleTodo(id)),
  };
}

const ConnectedTodoItem = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoItem);
~~~~~~~~

Сам компонент `TodoItem` остался бы прежним. Он по-прежнему получает элемент `todo` и обработчик `onToggleTodo()` в качестве аргументов. Кроме того, вы можете увидеть еще две концепции, которые были объяснены ранее. Во-первых, селектор становится все сложнее, поскольку получает необязательные аргументы для выбора производных свойств из состояния. Во-вторых, функция `mapStateToProps()` использует входящие пропсы из компонента `TodoList`, который использует компонент `ConnectedTodoItem`.

Как видите, нормализованное состояние требует использования большего количества подключенных компонентов. Больше компонентов отвечают за выбор необходимых им производных свойств. Но в растущем приложении, следуя этому шаблону, можно легче рассуждать об этом. Вы передаете только те свойства, которые действительно необходимы компоненту. В последнем случае компонент `TodoList` заботится только о списке ссылок, а сам компонент `TodoItem` заботится о сущности, которая выбрана с использованием ссылки, передаваемой компонентом `TodoList`.

Существует другой способ денормализовать ваше нормализованное состояние при использовании библиотеки, такой как normalizr. Предыдущий сценарий позволял передавать только минимальные свойства компонентам. Каждый компонент отвечал за выбор своего состояния. В следующем сценарии вы денормализуете свое состояние в одном компоненте, в то время как другие компоненты не должны заботиться об этом. Вы будете использовать определенную схему, которую вы использовали для начальной нормализации, чтобы отменить нормализацию.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { denormalize, schema } from 'normalizr';

const todoSchema = new schema.Entity('todos');
const todosSchema = { todos: [ todoSchema ] };

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

function getTodos(state) {
  const entities = state.todoState.entities;
  const ids = state.todoState.ids;
  return denormalize(ids, [ todoSchema ], entities);
}

function mapStateToProps(state) {
  return {
    todos: getTodos(state),
  };
}

const ConnectedTodoList = connect(mapStateToProps)(TodoList);
~~~~~~~~

В этом случае вся нормализованная структура данных денормализуется в селекторе. У вас будет полный список задач в вашем компоненте `TodoList`. Компоненту `TodoItem` не нужно заботиться о денормализации.

Как вы можете видеть, есть два основных способа работы с нормализованным состоянием: в ваших селекторах или вообще в функциях `mapStateToProps()`. Это зависит от вас, чтобы найти наиболее подходящую реализацию для вашего собственного варианта использования. Возможно, вам даже не нужно сначала нормализовать свое состояние, потому что оно уже плоское или не очень глубоко вложенное.

### Reselect

При использовании селекторов в масштабирующем приложении вам следует рассмотреть библиотеку под названием [reselect](https://github.com/reactjs/reselect), которая предоставляет вам расширенные селекторы. По сути, она использует ту же концепцию простых селекторов, как вы узнали ранее, но имеет два улучшения.

Простой селектор имеет одно ограничение:

* *"Селекторы могут вычислять производные данные, позволяя Redux сохранять минимально возможное состояние."*

Есть еще два ограничения при использовании селекторов из библиотеки reselect:

* *"Селекторы эффективны. Селектор не пересчитывается, пока не изменится один из его аргументов."*
* *«Селекторы являются составными. Их можно использовать в качестве входных данных для других селекторов.»*

Селекторы - это чистые функции без каких-либо побочных эффектов. Результат не меняется, если входные данные остаются неизменными. Следовательно, когда функция вызывается дважды и ее аргументы не меняются, она возвращает один и тот же результат. Это утверждение используется в селекторах reselect. Это называется мемоизация в программировании. Селектору не нужно вычислять все заново, когда его входные данные не изменились. Он просто вернет тот же результат, потому что это чистая функция. При мемоизации он запоминает предыдущий входные данные и, если они не изменились, возвращает предыдущий результат. В масштабирующем приложении это может повлиять на производительность.

Другое преимущество при использовании reselect - возможность состовлять селекторы. Это поддерживает случай реализации многоразовых селекторов, которые решают только одну проблему. После этого они могут быть составлены в стиле функционального программирования.

Книга не будет погружаться глубже в библиотеку reselect. При изучении Redux полезно знать об этих расширенных селекторах, но в начале вы можете использовать простые селекторы. Если вы не хотите оставаться на месте, вы можете прочитать примеры использования в [официальном репозитории GitHub](https://github.com/reactjs/reselect) и применять в своих проектах, читая книгу.

## Практика: Todo с продвинутым Redux

Теперь в приложении Todo вы можете реорганизовать все, чтобы использовать продвинутые методы, которые вы изучили в предыдущих главах: промежуточное программное обеспечение, неизменную структуру данных с использованием spread операторов JavaScript, нормализованную структуру данных и селекторы. Давайте продолжим с приложением Todo, которое вы создали при подключении React и Redux. Последняя версия может быть найдена в этом [GitHub-репозитории](https://github.com/rwieruch/taming-the-state-todo-app/tree/3.0.0).

В первой части давайте используем промежуточное программное обеспечение [redux-logger] (https://github.com/evgenyrodionov/redux-logger). Вы можете установить его в командной строке:

{title="Command Line: /",lang="text"}
~~~~~~~~
npm install --save redux-logger
~~~~~~~~

Теперь вы можете использовать его при создании хранилища

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
# leanpub-start-insert
import { applyMiddleware, combineReducers, createStore } from 'redux';
# leanpub-end-insert
import { Provider, connect } from 'react-redux';
# leanpub-start-insert
import { createLogger } from 'redux-logger';
# leanpub-end-insert
import './index.css';

...

// store

const rootReducer = combineReducers({
  todoState: todoReducer,
  filterState: filterReducer,
});

# leanpub-start-insert
const logger = createLogger();

const store = createStore(
  rootReducer,
  undefined,
  applyMiddleware(logger)
);
# leanpub-end-insert
~~~~~~~~

Когда вы сейчас запускаете приложение Todo, вы должны увидеть вывод `logger` в консоли разработчика вашего браузера при отправке действий. Приложение Todo с промежуточным программным обеспечением, использующим redux-logger, можно найти в этом [GitHub-репозитарии](https://github.com/rwieruch/taming-the-state-todo-app/tree/4.0.0).

Вторая часть этой главы использует spread операторы JavaScript вместо функции `Object.assign()` для сохранения неизменной структуры данных. Вы можете применять его в своих функциях редуктора:

{title="src/index.js",lang="javascript"}
~~~~~~~~
function applyAddTodo(state, action) {
# leanpub-start-insert
  const todo = { ...action.todo, completed: false };
  return [ ...state, todo ];
# leanpub-end-insert
}

function applyToggleTodo(state, action) {
  return state.map(todo =>
    todo.id === action.todo.id
# leanpub-start-insert
      ? { ...todo, completed: !todo.completed }
# leanpub-end-insert
      : todo
  );
}
~~~~~~~~

Приложение должно работать так же, как и раньше, но на этот раз со spread оператором для сохранения неизменяемой структуры данных и, следовательно, неизменяемого объекта состояния. Исходный код снова можно найти в этом [GitHub-репозитории](https://github.com/rwieruch/taming-the-state-todo-app/tree/5.0.0).

В третьей части применения передовых методов из предыдущих глав вы будете использовать нормализованного структуру состояния. Поэтому вы можете установить аккуратную библиотеку [normalizr](https://github.com/paularmstrong/normalizr) в командной строке:

{title="Command Line: /",lang="text"}
~~~~~~~~
npm install --save normalizr
~~~~~~~~

Давайте посмотрим на начальное состояние `todoReducer`. Вы можете сделать начальное состояние для него. Например, как насчет того, чтобы завершить все примеры кодая в этой книге, имея для каждого элемент todo?

{title="src/index.js",lang="javascript"}
~~~~~~~~
const todos = [
  { id: '1', name: 'Redux Standalone with advanced Actions' },
  { id: '2', name: 'Redux Standalone with advanced Reducers' },
  { id: '3', name: 'Bootstrap App with Redux' },
  { id: '4', name: 'Naive Todo with React and Redux' },
  { id: '5', name: 'Sophisticated Todo with React and Redux' },
  { id: '6', name: 'Connecting State Everywhere' },
  { id: '7', name: 'Todo with advanced Redux' },
  { id: '8', name: 'Todo but more Features' },
  { id: '9', name: 'Todo with Notifications' },
  { id: '10', name: 'Hacker News with Redux' },
];

function todoReducer(state = todos, action) {
  ...
}
~~~~~~~~

Вы можете использовать normalizr для нормализации этой структуры данных. Сначала вы должны определить схему:

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import { applyMiddleware, combineReducers, createStore } from 'redux';
import { Provider, connect } from 'react-redux';
import { createLogger } from 'redux-logger';
# leanpub-start-insert
import { schema, normalize } from 'normalizr';
# leanpub-end-insert
import './index.css';

# leanpub-start-insert
// schemas

const todoSchema = new schema.Entity('todo');
# leanpub-end-insert
~~~~~~~~

Во-вторых, вы можете использовать схему для нормализации ваших начальных задач и использовать их в качестве параметра по умолчанию в вашем `todoReducer`.

{title="src/index.js",lang="javascript"}
~~~~~~~~
// reducers

const todos = [
  ...
];

# leanpub-start-insert
const normalizedTodos = normalize(todos, [todoSchema]);
const initialTodoState = {
  entities: normalizedTodos.entities.todo,
  ids: normalizedTodos.result,
};

function todoReducer(state = initialTodoState, action) {
# leanpub-end-insert
  ...
}
~~~~~~~~

В-третьих, ваш `todoReducer` должен обрабатывать структуру нормализованного состояния. Он должен иметь дело с сущностями и результатом (список идентификаторов). Вы можете вывести нормализованные задачи, даже если приложение Todo падает при попытке его запуска.

{title="src/index.js",lang="javascript"}
~~~~~~~~
const normalizedTodos = normalize(todos, [todoSchema]);
console.log(normalizedTodos);
~~~~~~~~

Настроенный редуктор будет иметь следующие внутренние функции:

{title="src/index.js",lang="javascript"}
~~~~~~~~
function applyAddTodo(state, action) {
  const todo = { ...action.todo, completed: false };
  const entities = { ...state.entities, [todo.id]: todo };
  const ids = [ ...state.ids, action.todo.id ];
  return { ...state, entities, ids };
}

function applyToggleTodo(state, action) {
  const id = action.todo.id;
  const todo = state.entities[id];
  const toggledTodo = { ...todo, completed: !todo.completed };
  const entities = { ...state.entities, [id]: toggledTodo };
  return { ...state, entities };
}
~~~~~~~~

Он работает с `entities` и `ids`, потому что это выходные данные нормализации. И последнее, но не менее важное: при подключении Redux к React компоненты должны знать о нормализованной структуре данных. Во-первых, связь между магазином и компонентами:

{title="src/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
function mapStateToPropsList(state) {
  return {
    todosAsIds: state.todoState.ids,
  };
}

function mapStateToPropsItem(state, props) {
  return {
     todo: state.todoState.entities[props.todoId],
  };
}
# leanpub-end-insert

# leanpub-start-insert
function mapDispatchToPropsItem(dispatch) {
# leanpub-end-insert
  return {
     onToggleTodo: id => dispatch(doToggleTodo(id)),
  };
}

# leanpub-start-insert
const ConnectedTodoList = connect(
  mapStateToPropsList
)(TodoList);
const ConnectedTodoItem = connect(
  mapStateToPropsItem,
  mapDispatchToPropsItem
)(TodoItem);
# leanpub-end-insert
~~~~~~~~

Во-вторых, компонент `TodoList` получает только `todosAsIds`, а `TodoItem` получает сущность `todo`.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function TodoApp() {
  return <ConnectedTodoList />;
}

# leanpub-start-insert
function TodoList({ todosAsIds }) {
# leanpub-end-insert
  return (
    <div>
# leanpub-start-insert
      {todosAsIds.map(todoId => <ConnectedTodoItem
        key={todoId}
        todoId={todoId}
      />)}
# leanpub-end-insert
    </div>
  );
}

function TodoItem({ todo, onToggleTodo }) {
  ...
}
~~~~~~~~

Приложение должно снова работать. Запустите его и поиграйте с ним. Вы можете найти исходный код в [GitHub-репозитарии](https://github.com/rwieruch/taming-the-state-todo-app/tree/6.0.0). Вы нормализовали структуру исходного состояния и настроили редуктор для работы с новой структурой данных.

В четвертой и последней части вы используете селекторы для вашей архитектуры Redux. Этот рефакторинг довольно прост. Вы должны извлечь части, которые воздействуют на состояние в ваших функциях `mapStateToProps()`, в функции селекторов. Сначала определим функции селектора:

{title="src/index.js",lang="javascript"}
~~~~~~~~
// selectors

function getTodosAsIds(state) {
  return state.todoState.ids;
}

function getTodo(state, todoId) {
  return state.todoState.entities[todoId];
}
~~~~~~~~

Во-вторых, вы можете использовать эти функции вместо того, чтобы работать с состоянием непосредственно в ваших функциях `mapStateToProps()`:

{title="src/index.js",lang="javascript"}
~~~~~~~~
// Connecting React and Redux

function mapStateToPropsList(state) {
  return {
# leanpub-start-insert
    todosAsIds: getTodosAsIds(state),
# leanpub-end-insert
  };
}

function mapStateToPropsItem(state, props) {
  return {
# leanpub-start-insert
     todo: getTodo(state, props.todoId),
# leanpub-end-insert
  };
}
~~~~~~~~

Приложение Todo теперь должно работать с селекторами. Вы можете снова найти его в [GitHub-репозиторий](https://github.com/rwieruch/taming-the-state-todo-app/tree/7.0.0).

## Практика: Todo но с большими преимуществами

В приложении Todo отсутствуют две функциональные составляющие: возможность добавить задачу и отфильтровать задачи по их полному состоянию. Давайте начнем с создания элемента todo. Во-первых, должен быть компонент, в котором вы можете ввести имя todo и выполнить его создание.

{title="src/index.js",lang="javascript"}
~~~~~~~~
class TodoCreate extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      value: '',
    };

    this.onCreateTodo = this.onCreateTodo.bind(this);
    this.onChangeName = this.onChangeName.bind(this);
  }

  onChangeName(event) {
    this.setState({ value: event.target.value });
  }

  onCreateTodo(event) {
    this.props.onAddTodo(this.state.value);
    this.setState({ value: '' });
    event.preventDefault();
  }

  render() {
    return (
      <div>
        <form onSubmit={this.onCreateTodo}>
          <input
            type="text"
            placeholder="Add Todo..."
            value={this.state.value}
            onChange={this.onChangeName}
          />
          <button type="submit">Add</button>
        </form>
      </div>
    );
  }
}
~~~~~~~~

Еще раз обратите внимание, что компонент полностью не знает о Redux. Он только обновляет свое локальное состояние `value` и после отправки формы использует локальное состояние `value` для функции обратного вызова `onAddTodo()`, которая доступна в объекте `props`. Компонент не знает, обновляет ли функция обратного вызова локальное состояние родительского компонента или хранилища Redux. Затем вы можете использовать подключенную версию этого компонента в компоненте `TodoApp`.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function TodoApp() {
  return (
# leanpub-start-insert
    <div>
      <ConnectedTodoCreate />
# leanpub-end-insert
      <ConnectedTodoList />
# leanpub-start-insert
    </div>
# leanpub-end-insert
  );
}
~~~~~~~~

Последний шаг - подключить компонент React к хранилищу Redux, сделав его подключенным компонентом.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function mapDispatchToPropsCreate(dispatch) {
  return {
    onAddTodo: name => dispatch(doAddTodo(uuid(), name)),
  };
}

const ConnectedTodoCreate = connect(null, mapDispatchToPropsCreate)(TodoCreate);
~~~~~~~~

Он использует функцию `mapDispatchToPropsCreate ()`, чтобы получить доступ к методу диспетчеризации хранилища Redux. Создатель действия `doAddTodo()` берет имя элемента todo, полученного из компонента `TodoCreate`, и генерирует уникальный идентификатор с помощью функции `uuid()`. Функция `uuid()` - это аккуратная маленькая вспомогательная функция, которая поступает из библиотеки [uuid](https://github.com/kelektiv/node-uuid). Сначала вы должны установить его:

{title="Command Line: /",lang="text"}
~~~~~~~~
npm install --save uuid
~~~~~~~~

И, во-вторых, вы можете импортировать его для генерации уникальных идентификаторов для вас:

{title="src/index.js",lang="javascript"}
~~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';
import { applyMiddleware, combineReducers, createStore } from 'redux';
import { Provider, connect } from 'react-redux';
import { createLogger } from 'redux-logger';
import { schema, normalize } from 'normalizr';
# leanpub-start-insert
import uuid from 'uuid/v4';
# leanpub-end-insert
import './index.css';
~~~~~~~~

Вы можете попробовать создать элемент todo в вашем приложении Todo прямо сейчас. Это должен работать. Затем вы хотите использовать функциональность вашего фильтра для фильтрации списка элементов задач по их свойству `complete`. Во-первых, вы должны добавить компонент `Filter`.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function Filter({ onSetFilter }) {
  return (
    <div>
      Show
      <button
        type="button"
        onClick={() => onSetFilter('SHOW_ALL')}>
        All</button>
      <button
        type="button"
        onClick={() => onSetFilter('SHOW_COMPLETED')}>
        Completed</button>
      <button
        type="button"
        onClick={() => onSetFilter('SHOW_INCOMPLETED')}>
        Incompleted</button>
    </div>
  );
}
~~~~~~~~

Компонент `Filter` получает только функцию обратного вызова. Опять же, он ничего не знает об управлении состоянием, которое происходит выше в Redux или где-то еще. Функция обратного вызова используется только в разных кнопках для установки определенных типов фильтров. Вы можете снова использовать подключенный компонент в компоненте `TodoApp`.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function TodoApp() {
  return (
    <div>
# leanpub-start-insert
      <ConnectedFilter />
# leanpub-end-insert
      <ConnectedTodoCreate />
      <ConnectedTodoList />
    </div>
  );
}
~~~~~~~~

И последнее, но не менее важное: вам нужно подключить компонент `Filter`, чтобы фактически использовать его в компоненте `TodoApp`. Он отправит создателю действия `doSetFilter`, передав тип фильтра из соответствующих кнопок в компоненте `Filter`.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function mapDispatchToPropsFilter(dispatch) {
  return {
    onSetFilter: filterType => dispatch(doSetFilter(filterType)),
  };
}

const ConnectedFilter = connect(null, mapDispatchToPropsFilter)(Filter);
~~~~~~~~

Когда вы сейчас запустите приложение Todo, вы увидите, что `filterState` изменится, когда вы нажмете одну из кнопок фильтра. Но ничего не происходит с вашими показанными задачами. Они не фильтруются, и это потому, что в вашем селекторе вы выбираете весь список задач. Следующим шагом будет настройка селектора для выбора только тех задач в списке, которые соответствуют фильтру. Во-первых, вы можете определить функции фильтра, которые соответствуют задачам в соответствии с их состоянием `completed`.

{title="src/index.js",lang="javascript"}
~~~~~~~~
// filters

const VISIBILITY_FILTERS = {
  SHOW_COMPLETED: item => item.completed,
  SHOW_INCOMPLETED: item => !item.completed,
  SHOW_ALL: item => true,
};
~~~~~~~~

Во-вторых, вы можете использовать свой селектор, чтобы выбрать только те задачи, которые соответствуют фильтру. У вас уже есть все селекторы на месте. Но вам нужно настроить один из них для фильтрации задач в соответствии с `filterState` из хранилища Redux.

{title="src/index.js",lang="javascript"}
~~~~~~~~
// selectors

function getTodosAsIds(state) {
# leanpub-start-insert
  return state.todoState.ids
    .map(id => state.todoState.entities[id])
    .filter(VISIBILITY_FILTERS[state.filterState])
    .map(todo => todo.id);
# leanpub-end-insert
}

function getTodo(state, todoId) {
  return state.todoState.entities[todoId];
}
~~~~~~~~

Поскольку ваше состояние нормализовано, вы должны пройти через все ваши `ids`, чтобы получить список `todos`, отфильтровать их по `filterState` и отобразить их обратно в 'ids'. Это компромисс, на который вы идете, когда нормализуете свою структуру данных, потому что вы всегда должны денормализовать ее в какой-то момент. Функциональность вашего фильтра должна работать, как только вы снова запустите свое приложение.

Финальное приложение вы можете найти в этом [GitHub-репозитории](https://github.com/rwieruch/taming-the-state-todo-app/tree/8.0.0). Он содержит применение всех знаний о промежуточном программном обеспечении Redux, неизменяемых и нормализованных структурах данных и селекторах.
