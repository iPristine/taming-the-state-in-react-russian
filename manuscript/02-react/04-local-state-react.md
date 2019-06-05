## Локльное состояние в React

The book uses React as view layer for demonstrating the local state in a web application. The following chapter focusses on the local state in React before it dives into sophisticated state management with Redux and MobX. As mentioned, the concept of local state should be known in other SPA solutions, too, and thus be applicable in those solutions.
Книга использует React в качестве слоя представления для демонстрации локального состояния в веб-приложении. В следующей главе основное внимание уделяется локальному состоянию в React, прежде чем перейти к глобальному управлению состоянием с помощью Redux и MobX. Как уже упоминалось, концепция локального состояния должна быть известна и в других решениях SPA и, следовательно, должна применяться в этих решениях.

Итак, как выглядит локальное состояние в React компоненте?

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class Counter extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      counter: 0
    };
  }

  render() {
    return (
      <div>
        <p>{this.state.counter}</p>
      </div>
    );
  }
}
~~~~~~~~

The example shows a `Counter` component that has a `counter` property in the local state object. It is defined with a value of `0` when the component gets instantiated by its constructor. In addition, the `counter` property from the local state object is used in the render method of the component to display its current value.
В примере показан компонент `Counter`, который имеет свойство `counter` в локальном объекте состояния. Он определяется со значением `0`, когда компонент создается его конструктором. Кроме того, свойство counter из локального объекта состояния используется в методе рендеринга компонента для отображения его текущего значения.

There is no state manipulation in place yet. Before you start to manipulate your state, you should know that you are never allowed to mutate the state directly: `this.state.counter = 1`. That would be a direct mutation. Instead, you have to use the React component API to change the state explicitly by using the `this.setState()` method. It keeps the state object immutable, because the state object isn't changed but a new modified copy of it is created.
Пока нет манипуляций с состоянием. Прежде чем вы начнете манипулировать состоянием, вы должны знать, что вам никогда не разрешается изменять его напрямую: `this.state.counter = 1`. Это было бы прямым изменением. Вместо этого вы должны использовать компонент API React для явного изменения состояния с помощью метода `this.setState()`. Он сохраняет объект состояния неизменным, поскольку объект состояния не изменяется, а создается его новая измененная копия.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class Counter extends React.Component {
  constructor(props) {
    ...

# leanpub-start-insert
    this.onIncrement = this.onIncrement.bind(this);
    this.onDecrement = this.onDecrement.bind(this);
# leanpub-end-insert
  }

# leanpub-start-insert
  onIncrement() {
    this.setState({
      counter: this.state.counter + 1
    });
  }

  onDecrement() {
    this.setState({
      counter: this.state.counter - 1
    });
  }
# leanpub-end-insert

  render() {
    ...
  }
}
~~~~~~~~

The class methods can be used in the `render()` method to trigger the local state changes.
Методы класса могут использоваться в методе `render()` для запуска локальных изменений состояния.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class Counter extends React.Component {
  ...

  render() {
    return (
      <div>
        <p>{this.state.counter}</p>
# leanpub-start-insert
        <button type="button" onClick={this.onIncrement}>
          Increment
        </button>
        <button type="button" onClick={this.onDecrement}>
          Decrement
        </button>
# leanpub-end-insert
      </div>
    );
  }
}
~~~~~~~~

Now, the button `onClick` handler should invoke the class methods to alter the state by either incrementing or decrementing the counter value. Then, the update functionality with `this.setState()` is performing a **shallow merge** of objects. What does a shallow merge mean? Imagine you had the following state in your component, two arrays with objects:
Теперь обработчик кнопки `onClick` должен вызывать методы класса для изменения состояния путем увеличения или уменьшения значения счетчика. Затем функция обновления с помощью `this.setState()` выполняет **частичное слияние** объектов. Что означает частичное слияние? Представьте, что у вас есть следующее состояние в вашем компоненте, два массива с объектами:

{title="Code Playground",lang="javascript"}
~~~~~~~~
this.state = {
  authors: [...],
  articles: [...],
};
~~~~~~~~

When updating the state only partly, for instance the authors, the other part, in this case the articles, are left intact.
При обновлении состояния только частично, например, авторов, другая часть, в данном случае статьи, остается без изменений.

{title="Code Playground",lang="javascript"}
~~~~~~~~
this.setState({
  authors: [
    { name: 'Robin', id: '1' }
  ]
});
~~~~~~~~

It only updates the `authors` array with a new array without touching the `articles` array. That's called a shallow merge. It simplifies the local state management for you so that you don't have to keep an eye on all properties at once in the local state.
Он только обновляет массив `authors` новым массивом, не затрагивая массив `articles`. Это называется мелким слиянием. Это упрощает локальное управление состояниями, так что вам не нужно следить за всеми свойствами одновременно в локальном состоянии.

### SКомпоненты с состоянием и без

Local state can only be used in React ES6 class components. The component becomes a **stateful component** when state is used. Otherwise, it can be called **stateless component** even though it is still a React ES6 class component. This can be the case if you still need to use React's lifecycle methods.
Локальное состояние может использоваться только в React компонентах основанных на классах ES6. Компонент становится **компонентом с состоянием**, когда используется состояние. В противном случае его можно назвать **компонентом без состояния**, даже если он все еще является React компонентом основанным на классах ES6. Это может быть в том случае, если вам все еще нужно использовать методы жизненного цикла React.

On the other hand, **functional stateless components** have no state, because, as the name implies, they are only functions and thus, they are stateless. They get input as props and return output as JSX. In a stateless component, state can only be passed as props from a parent component. However, the functional stateless component is unaware of the props being state in the parent component. In addition, callback functions can be passed down to the functional stateless component to have an indirect way of altering the state in the parent component again. A functional stateless component for the Counter example could look like the following:
С другой стороны, **функциональные компоненты без состояния**  как следует из названия не имеют состояния, потому что, они являются только функциями. Они получают входные данные как пропсы и возвращают выходные данные как JSX. В компоненте без состояния состояние может передаваться только как пропсы родительского компонента. Тем не менее, функциональный компонент без сохранения состояния не знает о том что пропс это состоянии в родительском компоненте. Кроме того, функции обратного вызова могут быть переданы функциональному компоненту без сохранения состояния, чтобы иметь косвенный способ изменения состояния в родительском компоненте снова. Функциональный компонент без сохранения состояния Counter может выглядеть следующим образом:

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

function CounterPresenter(props) {
  return (
    <div>
      <p>{props.counter}</p>
      <button type="button" onClick={props.onIncrement}>
        Increment
      </button>
      <button type="button" onClick={props.onDecrement}>
        Decrement
      </button>
    </div>
  );
}
~~~~~~~~

Now only the props from the parent component would be used in this functional stateless component. The `counter` prop would be displayed and the two callback functions, `onIncrement()` and `onDecrement()` would be used for the buttons. However, the functional stateless component is not aware whether the passed properties are state, props or some other derived properties. The origin of the props doesn't need to be in the parent component after all, it could be somewhere higher up the component tree. The parent component would only pass the properties or derived properties along the way. In addition, the component is unaware of what the callback functions are doing. It doesn't know that these alter the local state of the parent component.
Теперь в этом функциональном компоненте без состояния будут использоваться только пропсы родительского компонента. Будет отображаться пропс `counter`, и для кнопок будут использоваться две функции обратного вызова `onIncrement()` и `onDecrement()`. Однако функциональный компонент без сохранения состояния не знает, являются ли переданные свойства состоянием, пропсом или другими производными свойствами. Источник пропса не обязательно должен находиться в родительском компоненте, он может находиться где-то выше в дереве компонентов. Родительский компонент будет только передавать свойства или производные свойства по пути. Кроме того, компонент не знает, что делают функции обратного вызова. Он не знает, что они изменяют локальное состояние родительского компонента.

After all, the callback functions in the stateless component would make it possible to alter the state somewhere above in one of the parent components. Once the state was manipulated, the new state flows down as props into the child component again. The new `counter` prop would be displayed correctly, because the render method of the child component runs again with the incoming changed props.
В конце концов, функции обратного вызова в компоненте без состояния позволят изменить состояние где-то выше в одном из родительских компонентов. После того, как состояние изменится, новое состояние снова передается как пропс в дочерний компонент. Новый пропс `counter` будет отображаться правильно, потому что метод рендеринга дочернего компонента запускается снова с входящими измененными пропсами.

The example shows how local state can traverse down from one component to the component tree. To make the example with the functional stateless component complete, let's quickly show what a potential parent component, that manages the local state, would look like. It is a React ES6 class component in order to be stateful.
Этот пример показывает, как локальное состояние может передаваться от одного компонента к дереву компонентов. Чтобы завершить пример с функциональным компонентом без состояния, давайте быстро покажем, как будет выглядеть потенциальный родительский компонент, который управляет локальным состоянием. Это React компонент основанный на классах ES6 для сохранения состояния.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class CounterContainer extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      counter: 0
    };

    this.onIncrement = this.onIncrement.bind(this);
    this.onDecrement = this.onDecrement.bind(this);
  }

  onIncrement() {
    this.setState({
      counter: this.state.counter + 1
    });
  }

  onDecrement() {
    this.setState({
      counter: this.state.counter - 1
    });
  }

  render() {
    return <CounterPresenter
      counter={this.state.counter}
      onIncrement={this.onIncrement}
      onDecrement={this.onDecrement}
    />
  }
}
~~~~~~~~

It is not by accident that the suffixes in the naming of both `Counter` components are `Container` and `Presenter`. It is called the [container and presentational component pattern](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0). It is most often applied in React, but could live in other component centred libraries and frameworks, too. If you have never heard about it, I recommend reading the referenced article. It is a widely used pattern, where the container component deals with "How things work" and the presenter component deals with "How things look". In this case, the container component cares about the state while the presenter component only displays the counter value and provides a handful of click handler yet without knowing that these click handlers manipulate the state. Note that the presenter component is called presentational component in the referenced article. I shortened the name from presentational to presenter component for the sake of convencience.
Не случайно суффиксами в именовании обоих компонентов «Counter» являются «Container» и «Presenter». Он называется [шаблон компонентов контейнера и представления](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0). Чаще всего он применяется в React, но может существовать и в других компонентно-ориентированных библиотеках и средах. Если вы никогда не слышали об этом, я рекомендую прочитать статью ссылке. Это широко используемый шаблон, где компонент контейнера это «Как все работает», а компонент презентатора «Как все выглядит». В нашем случае компонент контейнера заботится о состоянии, в то время как компонент представления отображает только значение счетчика и предоставляет несколько обработчиков кликов, не зная, что эти обработчики кликов манипулируют состоянием. Обратите внимание, что в указанной статье компонент представления называется компонентом презентации. Ради удобства я сократил название с презентационного компонента (`presentational`) до компонента представления (`presenter`).

Container components are the ideal candidates to manage state while the presenter components only display it and act on callback functions. You will encounter these container components more often in the book, when dealing with the concepts of higher-order components, that could potentially manage local state, and connected components.
 Компоненты контейнера являются идеальными кандидатами для управления состоянием, в то время как компоненты представления только отображают его и воздействуют на функции обратного вызова. Эти компоненты контейнеров вы встретите чаще в книге, когда будете иметь дело с понятиями компонентов более высокого порядка, которые потенциально могут управлять локальным состоянием и связанными компонентами.

### Пропс vs. Состояния

The previous example made clear that there is a difference between state and props in React. When properties are passed to a child component, whether it is state, props or derived properties, the child component isn't aware of the kind of properties. It sees the incoming properties as props. That's perfect, because the component shouldn't care at all about the kind of properties. It should only make use of them as simple props.
В предыдущем примере было ясно, что в React существует разница между состоянием и пропсом. Когда свойства передаются дочернему компоненту, будь то состояние, пропс или производные свойства, дочерний компонент не знает о типе свойств. Он видит входящие свойства в качестве пропса. Это прекрасно, потому что компонент не должен заботиться о типе свойств. Он должен использовать их только как простой пропс.

The props come from a parent component. In the parent component these props can be state, props or derived properties. It depends on the parent component, if it manages the properties itself (state), if it gets the properties from a parent component itself (props) or if it derives new properties from the incoming props coming from its parent component along the way (derived properties).
Пропс поступает из родительского компонента. В родительском компоненте эти пропсы могут быть состоянием, пропсами или производными свойствами. Это зависит от родительского компонента, если он сам управляет свойствами (состоянием), получает свойства от самого родительского компонента (пропса) или получает новые свойства от входящих пропсов, поступающих от его родительского компонента по пути (производный свойства).

After all, you can't modify props. Props are only properties passed from a parent component to a child component. On the other hand, the local state lives in the component itself. You can access it by using `this.state`, modify it by using `this.setState()`, and pass it down as props to child components.
В конце концов, вы не можете изменить пропс. Пропсы - это только свойства, передаваемые из родительского компонента в дочерний компонент. С другой стороны, локальное состояние живет в самом компоненте. Вы можете получить к нему доступ с помощью `this.state`, изменить его с помощью `this.setState()` и передать его в качестве пропса дочерним компонентам.

When one of these objects changes, whether it is the props that come from the parent component or the state in the component, the update lifecycle methods of the component will run. One of these lifecycle methods is the `render()` method that updates your component instance based on the props and state. The correct values will be used and displayed after the update ran in your component.
Когда один из этих объектов изменяется, будь то пропсы, исходящие от родительского компонента, или состояние в компоненте, будут запущены методы обновления жизненного цикла компонента. Одним из этих методов жизненного цикла является метод `render ()`, который обновляет экземпляр вашего компонента на основе пропсов и состояния. Правильные значения будут использоваться и отображаться после запуска обновления в вашем компоненте.

When you start to use React, it might be difficult to identify props and state. Personally, I like the [rules in the official React documentation](https://reactjs.org/docs/thinking-in-react.html) to identify state and props:
Когда вы начинаете использовать React, может быть трудно определить пропс это или состояние. Лично мне нравятся [правила в официальной документации React](https://reactjs.org/docs/thinking-in-react.html) для определения состояния и пропсов:

* Are the properties passed from the parent component? If yes, the likelihood is high that they aren't state. Though it is possible to save props as state, there are little use cases. It should be avoided to save props as state. Use them as props as they are.
Свойства передаются от родительского компонента? Если да, высока вероятность того, что они не являются состоянием. Хотя возможно сохранить пропс как состояние, вариантов использования мало. Следует избегать сохранения пропсов как состояния. Используйте их в качестве пропсов, как они есть.

* Are the properties unchanged over time? If yes, they don't need to be stateful, because they don't get modified.
Изменяется ли свойства с течением времени? Если да, они не должны быть состоянием, потому что они не модифицируются.

* Are the properties derivable from local state or props? If yes, you don't need them as state, because you can derive them. If you allocated extra state, the state has to be managed and can get out of sync when you miss to derive the new properties at some point.
Свойства выводятся из локального состояния или пропса? Если да, они вам не нужны как состояние, потому что вы можете их получить. Если вы выделили дополнительное состояние, этим состоянием нужно управлять, и оно может перестать синхронизироваться, если в какой-то момент вы не сможете получить новые свойства.

### Состояние формы

A common use case in applications is to use HTML forms. For instance, you might need to retrieve user information like a name or credit card number or submit a search query to an external API. Forms are used everywhere in web applications.
Распространенным вариантом использования в приложениях является использование HTML-форм. Например, вам может потребоваться получить информацию о пользователе, такую как имя или номер кредитной карты, или отправить поисковый запрос во внешний API. Формы используются везде в веб-приложениях.

There are two ways to use forms in React. You can use the ref attribute or local state. It is recommended to use the local state approach, because the ref attribute is reserved for only a few use cases. If you want to read about these use cases when using the ref attribute, I encourage you to read the following article: [When to use Ref on a DOM node in React](https://www.robinwieruch.de/react-ref-attribute-dom-node/).
Есть два способа использовать формы в React. Вы можете использовать атрибут ref или локальное состояние. Рекомендуется использовать подход локального состояния, поскольку атрибут ref зарезервирован только для нескольких случаев использования. Если вы хотите прочитать об этих случаях использования атрибута ref, я рекомендую вам прочитать следующую статью: [Когда использовать Ref на DOM элементе в React](https://www.robinwieruch.de/react-ref-attribute-dom-node/).

The following code snippet is a quick demonstration on how form state can be used by using the ref attribute. Afterward, the code snippet will get refactored to use the local state which is the best practice anyway.
Следующий фрагмент кода является демонстрацией того, как можно использовать состояние формы с помощью атрибута ref. После этого фрагмент кода будет подвергнут рефакторингу для использования локального состояния, что в любом случае является наилучшей практикой.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class Search extends React.Component {
  constructor(props) {
    super(props);

    this.onSubmit = this.onSubmit.bind(this);
  }

  onSubmit(event) {
    const { value } = this.input;

    // do something with the search value
    // e.g. propagate it up to the parent component
    this.props.onSearch(value);

    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.onSubmit}>
        <input
          ref={node => this.input = node}
          type="text"
        />
        <button type="submit">
          Search
        </button>
      </form>
    );
  }
}
~~~~~~~~

The value from the input node is retrieved by using the reference to the DOM node. It happens in the `onSubmit()` method. The reference is created by using the ref attribute in the `render()` method.
Значение из инпута извлекается с использованием ссылки на DOM элемент. Это происходит в методе `onSubmit()`. Ссылка создается с помощью атрибута ref в методе `render()`.

Now let's see how to make use of local state to embrace best practices rather than using the reserved ref attribute.
Теперь давайте посмотрим, как использовать локальное состояние, чтобы использовать распостраненные практики, без использоваия зарезервированного атрибута ref.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class Search extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      query: ''
    };

    this.onChange = this.onChange.bind(this);
    this.onSubmit = this.onSubmit.bind(this);
  }

  onChange(event) {
    const { value } = event.target;

    this.setState({
      query: value
    });
  }

  onSubmit(event) {
    const { query } = this.state;

    // do something with the search value
    // e.g. propagate it up to the parent component
    this.props.onSearch(query);

    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.onSubmit}>
        <input
          onChange={this.onChange}
          type="text"
        />
        <button type="submit">
          Search
        </button>
      </form>
    );
  }
}
~~~~~~~~

You don't need to make use of the ref attribute anymore. You can solve the problem by using local state only. The example demonstrates it with only one input field yet it can be used with multiple input fields, too. You would only need to allocate more properties, one for each input field, in the local state.
Вам больше не нужно использовать атрибут ref. Вы можете решить проблему, используя только местное состояние. Пример демонстрирует это только с одним полем ввода, но также может использоваться с несколькими полями ввода. Вам нужно будет только выделить больше свойств, по одному для каждого поля ввода, в локальном состоянии.

### Контролируемые компоненты

The previous example of using form state with local state has one flaw. It doesn't make use of **controlled components**. Naturally, a HTML input field holds its own state. When you enter a value into the input field, the DOM node knows about the value. That's the native behavior of HTML elements, otherwise they wouldn't work on their own.
Предыдущий пример использования состояния формы с локальным состоянием имеет один недостаток. Он не использует **контролируемые компоненты**. Поле ввода HTML содержит свое собственное состояние. Когда вы вводите значение в поле ввода, узел DOM знает о значении. Это естественное поведение элементов HTML, иначе они не будут работать сами по себе.

However, the value lives in your local state, too. You have it in both, the native DOM node state and local state. But you want to make use of a single source of truth. It is a best practice to overwrite the native DOM node state by using the `value` attribute on the HTML element and the value from the local state from the React component.
Однако значение живет и в вашем локальном состоянии. Знчение есть как в нативном состоянии узла DOM, так и в локальном состоянии. Но вы хотите использовать единый источник правды. Рекомендуется перезаписывать нативное состояние узла DOM, используя атрибут `value` в элементе HTML и значение из локального состояния из компонента React.

Let's consider the previous example again. The input field had no value attribute assigned. By using the native value attribute and passing the local state as value, you convert an uncontrolled component to a controlled component.
Давайте снова рассмотрим предыдущий пример. Поле ввода не имеет назначенного атрибута значения. Используя нативный атрибут значения и передавая локальное состояние в качестве значения, вы преобразуете неуправляемый компонент в контролируемый компонент.

{title="Code Playground",lang="javascript"}
~~~~~~~~
import React from 'react';

class Search extends React.Component {

  ...

  render() {
    return (
      <form onSubmit={this.onSubmit}>
        <input
# leanpub-start-insert
          value={this.state.query}
# leanpub-end-insert
          onChange={this.onChange}
          type="text"
        />
        <button type="submit">
          Search
        </button>
      </form>
    );
  }
}
~~~~~~~~

Now the value comes from the local state as single source of truth. It cannot get out of sync with the native DOM node state. This way, you can provide an initial state for the DOM node state too. Otherwise, try to have an initial local state for the `query` in your local state, but don't provide the `value` attribute to the input field. Your state would be out of sync in the beginning, because the input field would be empty even though the local state of the React component says something else.
Теперь значение исходит от локального состояния как единственного источника правды. Он не может выйти из синхронизации с собственным состоянием узла DOM. Таким образом, вы также можете указать начальное состояние для состояния узла DOM. В противном случае попытайтесь получить начальное локальное состояние для `query` в вашем локальном состоянии, но не предоставляйте атрибут `value` для поля ввода. Ваше состояние было бы не синхронизировано в начале, потому что поле ввода было бы пустым, даже если в локальном состоянии компонента React есть что-то еще.

### Однонаправленный поток данных

In the previous example, you experienced a typical unidirectional data flow. The Flux architecture, the underlying architecture for several sophisticated state management solutions such as Redux, coined the term **unidirectional data flow**. You will get to know more about the Flux architecture in a later chapter. But the essence of an unidirectional data flow is embraced by local state in React, too. State in React flows only in one direction. State gets updated by using `this.setState()` and is displayed due to the `render()` lifecycle method by accessing `this.state`. Then again, it can be updated via `this.setState()` and a component re-renders.
В предыдущем примере вы увидели типичный однонаправленный поток данных. Архитектура Flux, лежащая в основе нескольких сложных решений для управления состоянием, таких как Redux, ввела термин **однонаправленный поток данных**. Вы узнаете больше об архитектуре Flux в следующей главе. Но сущность однонаправленного потока данных охватывает и локальное состояние в React. Состояние в React течет только в одном направлении. Состояние обновляется с помощью `this.setState ()` и отображается методом жизненного цикла `render()` путем доступа к `this.state`. И снова, он может быть обновлен с помощью `this.setState()`, и компонент будет перерисован.

The previous example, where you have used controlled components, shows the perfect loop of the unidirectional data flow. The input field triggers the `onChange` handler when the input changes. The handler alters the local state. The changed local state triggers an update lifecycle of the component. The update lifecycle runs the `render()` lifecycle method again. The `render()` method makes use of the updated state. The state flows back to the input field to make it a controlled component. The loop is closed. A new loop can be triggered by typing something into the input field again.
В предыдущем примере, где вы использовали контролируемые компоненты, показан идеальный цикл однонаправленного потока данных. Поле ввода запускает обработчик `onChange` при изменении значения инпута. Обработчик изменяет локальное состояние. Измененное локальное состояние запускает жизненный цикл обновления компонента. Жизненный цикл обновления снова запускает метод жизненного цикла `render()`. Метод `render()` использует обновленное состояние. Состояние возвращается обратно в поле ввода, чтобы сделать его управляемым компонентом. Цикл замкнут. Новый цикл можно запустить, снова набрав что-то в поле ввода.

The unidirectional data flow makes state management predictable and maintainable. The best practice already spread to other state libraries, view layer libraries and SPA solutions. In the previous generation of SPAs, most often other mechanics were used. For instance, in Angular 1.x you would have used two-way data binding in a model-view-controller (MVC) architecture. That means, once you changed the value in the view, let's say in an input field by typing something, the value got changed in the controller. But it worked vice versa, too. Once you had changed the value in the controller programmatically, the view, to be more specific the input field, displayed the new value. You might wonder: What's the problem with this approach? Why is everybody using unidirectional data flow instead of bidirectional data flow now?
Однонаправленный поток данных делает управление состоянием предсказуемым и обслуживаемым. Лучшая практика уже распространилась на другие библиотеки состояний, библиотеки представления и решения SPA. В предыдущем поколении SPA чаще всего использовались другие механики. Например, в Angular 1.x вы бы использовали двустороннее связывание данных в архитектуре модель-представление-контроллер (MVC). Это означает, что как только вы изменили значение в представлении, скажем, в поле ввода, введя что-то, значение изменилось в контроллере. Но это сработало и наоборот. После того как вы изменили значение в контроллере программным способом, в представлении, чтобы быть более точным в поле ввода, отобразилось новое значение. Вы можете спросить: в чем проблема с этим подходом? Почему сейчас все используют однонаправленный поток данных вместо двунаправленного потока данных?

#### Однонаправленный vs. Двунаправленного поток данных

React embraces unidirectional data flow. In the past, frameworks like Angular 1.x embraced bidirectional data flow. It was known as two-way data binding. It was one of the reasons that made Angular popular in the first place. But it failed in this particular area, too. Especially, in my opinion, this particular flaw led a lot of people to switch to React. But at this point I don't want to get too opinionated. So why did the bidirectional data flow fail? Why is everyone adopting the unidirectional data flow?
React охватывает однонаправленный поток данных. В прошлом фреймворки, такие как Angular 1.x, охватывали двунаправленный поток данных. Это было известно как двусторонняя привязка данных. Это была одна из причин, по которой Angular был популярен в первую очередь. Но и в этой конкретной области это не удалось. Особенно, на мой взгляд, именно этот недостаток заставил многих людей перейти на React. Но на данный момент я не хочу быть слишком самоуверенным. Так почему же двунаправленный поток данных потерпел неудачу? Почему все принимают однонаправленный поток данных?

The three advantages in unidirectional data flow over bidirectional data flow are predicability, maintainability and performance.
Тремя преимуществами однонаправленного потока данных над двунаправленным потоком данных являются предсказуемость, удобство обслуживания и производительность.

**Predicability**: In a scaling application, state management needs to stay predictable. When you alter your state, it should be clear which components care about it. It should also be clear who alters the state in the first place. In an unidirectional data flow one stakeholder alters the state, the state gets stored, and the state flows down from one place, for instance a stateful component, to all child components that are interested in the state.
**Предсказуемость**: в масштабирующем приложении управление состоянием должно оставаться предсказуемым. Когда вы изменяете свое состояние, должно быть ясно, какие компоненты заботятся об этом. Также должно быть понятно, кто в первую очередь меняет состояние. В однонаправленном потоке данных один участник изменяет состояние, состояние сохраняется, и состояние передается из одного места, например, компонента с состоянием, ко всем дочерним компонентам, которые заинтересованы в этом состоянии.

**Maintainability:** When collaborating in a team on a scaling application, one requirement of state management is predictability. Humans are not capable to keep track of a growing bidirectional data flow. It is a limitation by nature. That's why the state management stays more maintainable when it is predictable. Otherwise, when people cannot reason about the state, they introduce inefficient state handling. But maintainability doesn't come without any cost in a unidirectional data flow. Even though the state is predictable, it often needs to be refactored thoughtfully. In a later chapter, you will read about those refactorings such as lifting state or higher-order components for local state.
**Поддерживаемость:** При совместной работе в команде над масштабируемым приложением одним из требований управления состоянием является предсказуемость. Люди не способны отслеживать растущий двунаправленный поток данных. Это ограничение по природе. Вот почему управление состоянием остается более управляемым, пока оно предсказуемо. В противном случае, когда люди не могут основываться на состоянии, они вводят неэффективное управление состоянием. Но поддерживаемость не обходится без затрат в однонаправленном потоке данных. Хотя состояние предсказуемо, его часто необходимо тщательно продумать. В следующей главе вы прочтете об этих рефакторингах, таких как поднятие состояния или компоненты более высокого порядка для локального состояния.

**Performance:** In a unidirectional data flow, the state flows down the component tree. All components that depend on the state have the chance to re-render. Contrary to a bidirectional data flow, it is not always clear who has to update according to state changes. The state flows in too many directions. The model layer depends on the view layer and the view layer depends on the model layer. It's a vice versa dependency that leads to performance issues in the update lifecycle.
**Производительность:** В однонаправленном потоке данных состояние передается по дереву компонентов. Все компоненты, зависящие от состояния, имеют возможность повторного рендеринга. В отличие от двунаправленного потока данных, где не всегда ясно, кто должен обновляться в соответствии с изменениями состояния. Состояние течет в слишком многих направлениях. Слой модели зависит от слоя представления, а слой представления зависит от слоя модели. Это обратная зависимость, которая приводит к проблемам с производительностью в жизненном цикле обновления.

These three advantages show the benefits of using a unidirectional data flow over an bidirectional data flow. That's why so many state management and SPA solutions thrive for the former one nowadays.
Эти три преимущества демонстрируют превосходство использования однонаправленного потока данных над двунаправленным потоком данных. Вот почему в наши дни так много решений для управления состоянием и SPA решения процветают в отличии от предшественников.
