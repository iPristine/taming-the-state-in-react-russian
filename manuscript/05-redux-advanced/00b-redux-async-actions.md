# Асинхронный Redux

В книге вы пока что использовали только синхронные действия. Там нет задержки диспетчеризации действия. Тем не менее, иногда вы хотите отложить действие. Пример может быть простым: представьте, что вы хотите получить уведомление для пользователя приложения о создании элемента todo. Уведомление должно отображаться, а потом скрываться через одну секунду. Первое действие показало бы уведомление,  устанавливая свойство `isShowingNotification` в true в хранилище Redux. После этого вам понадобится второе действие с задержкой, чтобы скрыть уведомление. В простейшем случае это будет выглядеть следующим образом:

{title="Code Playground",lang="javascript"}
~~~~~~~~
store.dispatch({ type: 'NOTIFICATION_SHOW', text: 'Todo created.' });
setTimeout(() => {
  store.dispatch({ type: 'NOTIFICATION_HIDE' })
}, 1000);
~~~~~~~~

Нет ничего плохого в простой функции `setTimeout()` в вашем приложении. Иногда проще запомнить основы JavaScript, чем пытаться применить еще одну библиотеку для решения проблемы. Поскольку вы знаете о создателях действий, следующим шагом может быть извлечение этих действий в соответствии с создателями действий и типами действий.

{title="Code Playground",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
const NOTIFICATION_SHOW = 'NOTIFICATION_SHOW';
const NOTIFICATION_HIDE = 'NOTIFICATION_HIDE';

function doShowNotification(text) {
  return {
    type: NOTIFICATION_SHOW,
    text,
  };
}

function doHideNotification() {
  return {
    type: NOTIFICATION_HIDE,
  };
}
# leanpub-end-insert

# leanpub-start-insert
store.dispatch(doShowNotification('Todo created.'));
# leanpub-end-insert
setTimeout(() => {
# leanpub-start-insert
  store.dispatch(doHideNotification());
# leanpub-end-insert
}, 1000);
~~~~~~~~

Сейчас в нашем растущем приложении есть две проблемы. Во-первых, вам приходится дублировать логику отложенного действия с помощью `setTimeout()` каждый раз, когда вы хотите показать уведомление и скрыть его снова. Что может помочь в этом случае? Правильно, извлечение функциональности как функции. Во-вторых, как только ваше приложение отправляет несколько уведомлений в разные места одновременно, первое действие, которое скрывает уведомление, скрывает их все. Что может помочь в этом случае? Правильно, вам нужно дать каждому уведомлению идентификатор.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// actions
const NOTIFICATION_SHOW = 'NOTIFICATION_SHOW';
const NOTIFICATION_HIDE = 'NOTIFICATION_HIDE';

function doShowNotification(text, id) {
  return {
    type: NOTIFICATION_SHOW,
    text,
    id,
  };
}

function doHideNotification(id) {
  return {
    type: NOTIFICATION_HIDE,
    id,
  };
}

// extracted functionality
let naiveId = 0;
function showNotificationWithDelay(dispatch, text) {
  dispatch(doShowNotification(text, naiveId));
  setTimeout(() => {
    dispatch(doHideNotification(naiveId));
  }, 1000);

  naiveId++;
}

// usage
showNotificationWithDelay(store.dispatch, 'Todo created.');
~~~~~~~~

Теперь каждое уведомление может быть идентифицировано в редукторе и, таким образом, может быть показано или скрыто. Извлеченная функция получает контроль над методом `dispatch()` из хранилища Redux, потому что в конце концов ей необходимо отправить отложенное действие.

**Почему бы не передать хранилище Redux вместо этого?** Обычно вы хотите избегать прямой передачи хранилища. Вы сталкивались с тем же рассуждением в книге, когда впервые создавали хранилище Redux в своем приложении React. Вы хотели сделать доступными функциональные возможности хранилища, но не все хранилище. Вот почему у вас есть только метод `dispatch()`, а не все хранилище в вашей функции `mapDispatchToProps()` при использовании react-redux. Подключенный компонент никогда не имеет доступа к самому хранилищу, и поэтому никакие другие функции не должны иметь прямой доступ к нему, чтобы не нарушать этих ограничений.

Приведенного выше шаблона достаточно для простых приложений Redux, которым требуется отложенное (асинхронное) действие. Однако в масштабирующихся приложениях он имеет недостаток. Подход создает второй тип действия. Помимо того, что существуют синхронные действия, которые можно отправлять напрямую, как вы уже делалли ранее, существуют и псевдоасинхронные действия. Эти псевдоасинхронные действия не могут быть отправлены напрямую, но используются внутри функции, которая принимает метод диспетчеризации в качестве аргумента, чтобы в конечном итоге диспетчеризировать действия. Это то, что вы сейчас реализовали с помощью функции `showNotificationWithDelay()`. Разве не было бы замечательно использовать оба типа действий одинаково, не беспокоясь об обходе метода диспетчеризации и не беспокоясь об асинхронных или синхронных действиях? Вы узнаете об этом в следующей главе.

## Redux Thunk

Предыдущий вопрос побудил Дэна Абрамова, создателя Redux, подумать об общей схеме проблемы асинхронных действий. Он придумал библиотеку под названием [redux-thunk](https://github.com/gaearon/redux-thunk), чтобы утвердить концепцию: синхронные и асинхронные действия должны отправляться аналогичным образом из хранилища Redux. Библиотека redux-thunk используется в качестве промежуточного ПО в вашем хранилище Redux.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import { createStore, applyMiddleware } from 'redux';
# leanpub-start-insert
import thunk from 'redux-thunk';
# leanpub-end-insert

...

const store = createStore(
  rootReducer,
  undefined,
# leanpub-start-insert
  applyMiddleware(thunk)
# leanpub-end-insert
);
~~~~~~~~

По сути, он создает промежуточное ПО для ваших создателей действий. В этом промежуточном ПО вы можете отправлять асинхронные действия. Помимо диспетчеризации объектов, вы также можете отправлять функции с Redux Thunk. Прежде вы всегда отправляли объекты как действия в этой книге. Само действие является объектом, и создатель действия возвращает объект действия.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// with plain action
store.dispatch({ type: 'NOTIFICATION_SHOW', text: 'Todo created.' });

// with action creator
function doShowNotification(text) {
  return {
    type: NOTIFICATION_SHOW,
    text,
  };
}

store.dispatch(doShowNotification('Todo created.'));
~~~~~~~~

Тем не менее, теперь вы также можете передать метод отправки функции. Функция всегда даст вам доступ к методу отправки снова.

{title="Code Playground",lang="javascript"}
~~~~~~~~
// with thunk function
let naiveId = 0;
store.dispatch(function (dispatch) {
  dispatch(doShowNotification('Todo created.', naiveId));
  setTimeout(() => {
    dispatch(doHideNotification(naiveId));
  }, 1000);

  naiveId++;
});
~~~~~~~~

Метод отправки хранилища Redux при использовании Redux Thunk будет различать передаваемые объекты и функции. Переданная функция называется **thunk**. Вы можете отправлять столько действий синхронно и асинхронно, сколько вы хотите в thunk. Когда блок управления растет и обрабатывает сложную бизнес-логику в какой-то момент в вашем приложении, он называется **fat thunk**. Вы можете извлечь функцию thunk как создателя асинхронных действий, то есть функцию высшего порядка, которая возвращает функцию thunk и может быть отправлена так же, как и создатель синхронных действий.

{title="Code Playground",lang="javascript"}
~~~~~~~~
let naiveId = 0;
function showNotificationWithDelay(text) {
  return function (dispatch) {
    dispatch(doShowNotification(text, naiveId));
    setTimeout(() => {
      dispatch(doHideNotification(naiveId));
    }, 1000);

    naiveId++;
  };
}

store.dispatch(showNotificationWithDelay('Todo created.'));
~~~~~~~~

Это похоже на решение без Redux Thunk. Однако на этот раз вам не нужно передавать метод dispatch и вместо этого имеете доступ к нему в возвращенной функции thunk. Теперь, когда он используется в компоненте React, компонент все еще выполняет только функцию обратного вызова, которую он получает через свои пропсы. Подключенный компонент затем отправляет действие, независимо от того, выполняется ли действие синхронно или асинхронно, в функции `mapDispatchToProps()`.

Это основы Redux Thunk. Есть еще несколько вещей, о которых стоит знать:

* **getState():** Функция thunk предоставляет вам метод `getState()` хранилища Redux в качестве второго аргумента: `function (dispatch, getState)`. Тем не менее, вы должны в основном избегать этого. Лучше всего передавать все необходимое состояние создателю действия, а не извлекать его в блок.
* **Промисы:** Thunks прекрасно работают в сочетании с промисами. Вы можете вернуть промис из своего thunk и использовать его, например, для ожидания его выполнения: `store.dispatch (showNotificationWithDelay ('Todo made.')). Then (...)`.
* **Рекурсивные Thunks:** Метод отправки в thunk снова можно использовать для отправки асинхронного действия. Таким образом, вы можете применить шаблон thunk рекурсивно.

### Практика: Todo с уведомлениями

Узнав об асинхронных действиях, приложение Todo может использовать уведомления, не так ли? Вы можете продолжить с приложением Todo, которое вы создали в последних главах. В качестве альтернативы вы можете клонировать его из этого [GitHub-репозитория](https://github.com/rwieruch/taming-the-state-todo-app/tree/8.0.0).

Первая часть этой главы посвящена по большому счету повторению использования всего, что вы узнали до асинхронных действий. Во-первых, вы должны реализовать редуктор уведомлений, который оценивает все действия, которые должны генерировать уведомления. В этом случае действие, которое создает элемент todo, следует повторно использовать для уведомления. Это замечательная вещь в Redux: все редукторы, которые заботятся о действии, могут использовать его. Также `id` элемента todo может быть использован для хранения уведомления со своим собственным идентификатором, чтобы позже его скрыть.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function notificationReducer(state = {}, action) {
  switch(action.type) {
    case TODO_ADD : {
      return applySetNotifyAboutAddTodo(state, action);
    }
    default : return state;
  }
}

function applySetNotifyAboutAddTodo(state, action) {
  const { name, id } = action.todo;
  return { ...state, [id]: 'Todo Created: ' + name  };
}
~~~~~~~~

Во-вторых, вы должны включить редуктор в ваш комбинированный редуктор, чтобы сделать его доступным для хранилища Redux.

{title="src/index.js",lang="javascript"}
~~~~~~~~
const rootReducer = combineReducers({
  todoState: todoReducer,
  filterState: filterReducer,
# leanpub-start-insert
  notificationState: notificationReducer,
# leanpub-end-insert
});
~~~~~~~~

Часть Redux готова. Это только редуктор, который нужно включить в хранилище Redux. Действие повторно используется. В-третьих, вы должны реализовать компонент React, который отображает все ваши уведомления.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function Notifications({ notifications }) {
  return (
    <div>
      {notifications.map(note => <div key={note}>{note}</div>)}
    </div>
  );
}
~~~~~~~~

В-четвертых, вам следует добавить подключенную версию компонента `Notifications` в свой компонент `TodoApp`.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function TodoApp() {
  return (
    <div>
      <ConnectedFilter />
      <ConnectedTodoCreate />
      <ConnectedTodoList />
# leanpub-start-insert
      <ConnectedNotifications />
# leanpub-end-insert
    </div>
  );
}
~~~~~~~~

И последнее, но не менее важное: вы должны связать React и Redux в подключенном компоненте `ConnectedNotifications`.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function mapStateToPropsNotifications(state, props) {
  return {
     notifications: getNotifications(state),
  };
}

const ConnectedNotifications = connect(mapStateToPropsNotifications)(Notifications);
~~~~~~~~

Осталось только реализовать отсутствующий селектор `getNotifications()`. Поскольку уведомления в хранилище Redux сохраняются как объект, так как они представляют собой карту пар идентификатора и уведомления, необходимо использовать вспомогательную функцию для преобразования его в массив. Вы можете извлечь вспомогательную функцию, потому что вам могут понадобиться такие функции чаще, и вы не должны связывать ее с разделом уведомлений.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function getNotifications(state) {
  return getArrayOfObject(state.notificationState);
}

function getArrayOfObject(object) {
  return Object.keys(object).map(key => object[key]);
}
~~~~~~~~

Первая часть этой главы сделана. Вы должны увидеть уведомление в вашем приложении Todo, как только вы создадите элемент todo. Вторая часть реализует действие `NOTIFICATION_HIDE` и использует его в `notificationReducer`, чтобы удалить уведомление из состояния. Сначала вы должны ввести тип действия:

{title="src/index.js",lang="javascript"}
~~~~~~~~
const TODO_ADD = 'TODO_ADD';
const TODO_TOGGLE = 'TODO_TOGGLE';
const FILTER_SET = 'FILTER_SET';
# leanpub-start-insert
const NOTIFICATION_HIDE = 'NOTIFICATION_HIDE';
# leanpub-end-insert
~~~~~~~~

Во-вторых, вы можете реализовать создатель действия, который использует тип действия. Он будет скрывать (удалять) уведомления по идентификатору, потому что они хранятся по идентификатору в хранилище Redux:

{title="src/index.js",lang="javascript"}
~~~~~~~~
function doHideNotification(id) {
  return {
    type: NOTIFICATION_HIDE,
    id
  };
}
~~~~~~~~

В-третьих, вы можете захватить его в новом `notificationReducer`. Функциональность деструктуризации JavaScript можно использовать для исключения свойства из объекта. Вы можете просто исключить уведомление и вернуть оставшийся объект. Это хороший прием, если вы хотите избавиться от свойства объекта, зная его ключ.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function notificationReducer(state = {}, action) {
  switch(action.type) {
    case TODO_ADD : {
      return applySetNotifyAboutAddTodo(state, action);
    }
# leanpub-start-insert
    case NOTIFICATION_HIDE : {
      return applyRemoveNotification(state, action);
    }
# leanpub-end-insert
    default : return state;
  }
}

# leanpub-start-insert
function applyRemoveNotification(state, action) {
  const {
    [action.id]: notificationToRemove,
    ...restNotifications,
  } = state;
  return restNotifications;
}
# leanpub-end-insert
~~~~~~~~

Это была вторая часть этой главы, в которой была представлена функция скрытия уведомлений. Но вы еще не используете эту функциональность. В третьей и последней части этой главы будут представлены асинхронные действия, позволяющие скрыть уведомление через пару секунд. Как упоминалось ранее, вам не понадобится библиотека для решения этой проблемы. Вы можете просто использовать функциональность JavaScript `setTimeout()`. Но для изучения асинхронных действий в Redux вы будете использовать Redux Thunk. Вы можете сменить ее на другую библиотеку асинхронных действий впоследствии, чтобы узнать об альтернативах. Вы узнаете об этих альтернативных библиотеках позже.

Во-первых, вы должны установить библиотеку [redux-thunk](https://github.com/gaearon/redux-thunk) в командной строке:

{title="Command Line: /",lang="text"}
~~~~~~~~
npm install --save redux-thunk
~~~~~~~~

Second, you can import it in your code:
Затем импортироать в свой код:

{title="src/index.js",lang="javascript"}
~~~~~~~~
...
# leanpub-start-insert
import thunk from 'redux-thunk';
# leanpub-end-insert
...
~~~~~~~~

And third, use it in your Redux store middleware:
И затем использовать в вашем промежуточном ПО Redux

{title="src/index.js",lang="javascript"}
~~~~~~~~
const store = createStore(
  rootReducer,
  undefined,
# leanpub-start-insert
  applyMiddleware(thunk, logger)
# leanpub-end-insert
);
~~~~~~~~

Приложение все еще должно работать. При использовании Redux Thunk вы можете отправлять объекты действий, как и раньше. Тем не менее, теперь вы также можете отправлять thunks (функции). Вместо того, чтобы отправлять объект действия, который только создает элемент todo, вы можете отправить thunk-функцию, которая создает элемент todo и скрывает уведомление о создании через пару секунд. У вас есть два простых создателя действий, `doAddTodo()` и `doHideNotification `, уже на месте. Вы только должны использовать их в своей функции Thunk.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function doAddTodoWithNotification(id, name) {
  return function (dispatch) {
    dispatch(doAddTodo(id, name));

    setTimeout(function () {
      dispatch(doHideNotification(id));
    }, 5000);
  }
}
~~~~~~~~

На последнем шаге вы должны использовать `doAddTodoWithNotification()` вместо `doAddTodo()` создателя действий при соединении Redux и React в вашем компоненте `TodoCreate`.

{title="src/index.js",lang="javascript"}
~~~~~~~~
function mapDispatchToPropsCreate(dispatch) {
  return {
# leanpub-start-insert
    onAddTodo: name => dispatch(
      doAddTodoWithNotification(uuid(), name)
    ),
# leanpub-end-insert
  };
}
~~~~~~~~

Вот и все. Ваши уведомления должны работать и скрываться через пять секунд. По сути, вы создали основу для системы уведомлений в своем приложении Todo. Вы можете использовать его и для других действий. Например, при завершении элемента todo вы также можете вызвать уведомление. Проект снова можно найти в этом [GitHub-репозитарии](https://github.com/rwieruch/taming-the-state-todo-app/tree/9.0.0).

## Альтернативные асинхронные действия

В целом концепция асинхронных действий привела к появлению нескольких библиотек, которые решают эту конкретную проблему. Redux Thunk был только первым, представленным Дэном Абрамовым. Тем не менее, он согласен с тем, что есть случаи, когда Redux Thunk не решает проблему эффективным способом. Redux Thunk можно использовать, когда вы впервые сталкиваетесь с вариантом использования асинхронных действий. Но когда речь идет о более сложных сценариях, вы можете использовать продвинутые решения помимо Redux Thunk.

Все эти решения решают проблему побочных эффектов в Redux. Асинхронные действия используются для устранения этих побочных эффектов. Они чаще всего используются при выполнении не чистых операций, которые также часто бывают асинхронными: выборка данных из API, задержка выполнения (например, скрытие уведомления) или доступ к кэшу браузера. Все эти операции являются асинхронными и не чистыми, поэтому решаются с помощью асинхронных действий. В приложении, которое использует парадигму функционального программирования, вы хотите, чтобы все эти не чистые операции находились на краю вашего приложения. Вы не хотите, чтобы это было близко к вашему основному приложению.

В этой главе кратко показаны альтернативы, которые вы можете использовать вместо Redux Thunk. Из всех возможных альтернатив я лишь хочу познакомить вас с самыми популярными и инновационными.

### Redux Saga

[Redux Saga](https://github.com/redux-saga/redux-saga) - самая популярная библиотека асинхронных действий для Redux. *«Ментальная модель заключается в том, что сага похожа на отдельный поток в вашем приложении, который несет единоличную ответственность за побочные эффекты».* По сути, она переносит не чистые операции в отдельные потоки. Эти потоки могут быть запущены, приостановлены или отменены обычными действиями Redux из вашего основного приложения. Таким образом, потоки в Redux Saga упрощают защиту основного приложения от побочных эффектов. Однако потоки могут отправлять действия и иметь доступ к состоянию.

Redux Saga использует [Генераторы JavaScript ES6](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Generator) в качестве базовой технологии. Вот почему код читается как синхронный код. Вы избегаете этого, имея функции обратного вызова. Преимущество над Redux Thunk в том, что ваши действия остаются чистыми и, следовательно, их можно хорошо проверить.

Помимо Redux Thunk и Redux Saga, есть и другие библиотеки побочных эффектов для Redux. Если вы хотите попробовать наблюдателей в JavaScript, вы можете попробовать [Redux Observable](https://github.com/redux-observable/redux-observable). Он основан на RxJS, универсальной библиотеке для реактивного программирования. Если вы заинтересованы в другой библиотеке, которая также использует принципы реактивного программирования, вы можете попробовать [Redux Cycles](https://github.com/cyclejs-community/redux-cycles).

В заключение, как вы можете видеть, все эти библиотеки, Redux Saga, Redux Observable и Redux Cycles, используют различные методы JavaScript. Вы можете попробовать поэкспериментировать с новейшими технологиями JavaScript: генераторами или наблюдателями. Вся экосистема вокруг асинхронных действий - отличная площадка для того, чтобы в конце концов попробовать что-то новое в JavaScript.

### Практика: Todo с Redux Saga

В предыдущей главе вы использовали Redux Thunk для отправки асинхронных действий. Они использовались для добавления элемента todo с уведомлением, затем уведомление исчезает через пару секунд. В этой главе вы будете использовать Redux Saga вместо Redux Thunk. Следовательно, вы можете установить первую библиотеку и удалить последнюю. Вы можете продолжить работу с предыдущим приложением Todo.

{title="Command Line: /",lang="text"}
~~~~~~~~
npm uninstall --save redux-thunk
npm install --save redux-saga
~~~~~~~~

Действие, которое вы использовали ранее с Thunk, теперь становится чистым создателем действия. Он будет использован для запуска потока саги.

{title="src/index.js",lang="javascript"}
~~~~~~~~
const TODO_ADD_WITH_NOTIFICATION = 'TODO_ADD_WITH_NOTIFICATION';

...

function doAddTodoWithNotification(id, name) {
  return {
    type: TODO_ADD_WITH_NOTIFICATION,
    todo: { id, name },
  };
}
~~~~~~~~

Теперь вы можете представить свою первую сагу, которая слушает это конкретное действие, потому что это действие используется исключительно для запуска потока саги.

{title="src/index.js",lang="javascript"}
~~~~~~~~
import { takeEvery } from 'redux-saga/effects';

...

// sagas

function* watchAddTodoWithNotification() {
  yield takeEvery(TODO_ADD_WITH_NOTIFICATION, ...);
}
~~~~~~~~

Most often you will find one part of the saga watching incoming actions and evaluating them. If an evaluation applies truthfully, it will often call another generator function, identified with the asterisk, that handles the side-effect. That way, you can keep your side-effect watcher maintainable and don't clutter them with business logic. In the previous example, a `takeEvery()` effect of Redux Saga is used to handle every action with the specified action type. Yet there are other effects in Redux Saga such as `takeLatest()` which only takes the last of the incoming actions by action type.
Чаще всего вы найдете одну часть саги, которая наблюдает за поступающими действиями и оценивает их. Если оценка применяется правдиво, она часто вызывает другую функцию генератора, обозначенную звездочкой, которая обрабатывает побочный эффект. Таким образом, вы можете поддерживать отслеживаемость побочных эффектов и не загромождать их бизнес-логикой. В предыдущем примере эффект `takeEvery()` Redux Saga используется для обработки каждого действия с указанным типом действия. Тем не менее, в Redux Saga есть и другие эффекты, такие как `takeLatest()`, который выполняет последнее из поступающих действий только по типу действия.

{title="src/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import { delay } from 'redux-saga';
import { put, takeEvery } from 'redux-saga/effects';
# leanpub-end-insert

function* watchAddTodoWithNotification() {
# leanpub-start-insert
  yield takeEvery(TODO_ADD_WITH_NOTIFICATION, handleAddTodoWithNotification);
# leanpub-end-insert
}

# leanpub-start-insert
function* handleAddTodoWithNotification(action) {
  const { todo } = action;
  const { id, name } = todo;
  yield put(doAddTodo(id, name));
  yield delay(5000);
  yield put(doHideNotification(id));
}
# leanpub-end-insert
~~~~~~~~

Как вы можете видеть, в генераторах JavaScript вы используете оператор `yield` для синхронного выполнения асинхронного кода. Только когда функция после `yield` разрешится, выполнится следующая строка кода. Redux Saga поставляется с вспомогательными функциями, такими как `delay()`, которые могут использоваться для задержки выполнения на некоторое время. Это было бы то же самое, что и использование `setTimeout()` в JavaScript, но помощник `delay()` делает его более лаконичным при использовании генераторов JavaScript и может использоваться синхронно при использовании оператора yield.

В конце, вам нужно всего лишь обменять промежуточное ПО в вашем хранилище Redux с использования Redux Thunk на Redux Saga.

{title="src/index.js",lang="javascript"}
~~~~~~~~
# leanpub-start-insert
import createSagaMiddleware, { delay } from 'redux-saga';
# leanpub-end-insert
import { put, takeEvery } from 'redux-saga/effects';

...

const rootReducer = combineReducers({
  todoState: todoReducer,
  filterState: filterReducer,
  notificationState: notificationReducer,
});

const logger = createLogger();
# leanpub-start-insert
const saga = createSagaMiddleware();
# leanpub-end-insert

const store = createStore(
  rootReducer,
  undefined,
# leanpub-start-insert
  applyMiddleware(saga, logger)
# leanpub-end-insert
);

# leanpub-start-insert
saga.run(watchAddTodoWithNotification);
# leanpub-end-insert
~~~~~~~~

That's it. You Todo application should run with Redux Saga instead of Redux Thunk now. The final application can be found in the [GitHub repository](https://github.com/rwieruch/taming-the-state-todo-app/tree/10.0.0) again. In the future, it is up to you to decide on an asynchronous actions library when using Redux. Is it Redux Thunk or Redux Saga? Or do you decide to try something new with Redux Observable or Redux Cycles? The Redux ecosystem is full of amenities to try cutting edge JavaScript features such as generators or observables. However, you need to decide yourself if you need a asynchronous actions library in the first place.
Вот и все. Ваше приложение Todo теперь должно работать с Redux Saga вместо Redux Thunk. Финальное приложение снова можно найти в [GitHub-репозитории](https://github.com/rwieruch/taming-the-state-todo-app/tree/10.0.0). В будущем вам придется выбирать библиотеку асинхронных действий при использовании Redux. Это будет Redux Thunk или Redux Saga? Или вы решите попробовать что-то новое с Redux Observable или Redux Cycles? Экосистема Redux полна удобств, чтобы попробовать передовые функции JavaScript, такие как генераторы или наблюдатели. Однако вам нужно решить, нужна ли вам библиотека асинхронных действий.
