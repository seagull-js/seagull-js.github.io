# Quickstart

This is a fast whirlwind tour for all you developers tipping their toes into
modern web development. First things first, if you're reading this at night or
other dark light conditions, click on the "A" icon on top of the page and tune
the color theme to sepia or night. Docs being easy on the eyes helps to get
into a more relaxed state.

## Installation

Seagull builts on top of JavaScript, so you need node.js installed. I'll assume
that you are developing on Mac OS X and have homebrew installed:

````bash
# node version manager, like rvm or rbenv for ruby
$ brew install nvm

# reload current shell with tweaked environment for nvm
$ echo "source $(brew --prefix nvm)/nvm.sh" >> ~/.profile

# latest LTS version which AWS Lambda supports (import for later deployments)
$ nvm install 6.10.3

# always use this node.js version as default in new shells
$ nvm alias default 6.10.3
````

For other cases, download node.js with your package manager of choice or head
over to [node.js official downloads page](https://nodejs.org/en/download/) and
get the latest LTS version 6.X.

Once you have node.js on your machine, installing seagull is as easy as
installing any global NPM package:

````bash
$ npm install -g @seagull-js/seagull-cli
````

This CLI tool is now available on your machine as `seagull` and contains all
tooling you'll need for developing seagull apps.

## Optional: VSCode

Of course you're totally free to choose any code editor or IDE for coding on
seagull projects, the recommended code editor is Visual Studio Code (VSCode).
It has strong support for typescript and tsx out of the box, plus ships with
a native git integration and looks good by default.

## Scaffolding

This may come as a shocking surprise for you, but we will create a simple ToDo-
App. By keeping the problem domain super simple, there is more room to highlight
some technological details and still get something up and running in a few
minutes. Having the elephant in the room out of the way, let's go:

````bash
# you probably have s special place on your disk for coding stuff, right?
$ cd path/to/your/projects/folder

# scaffold a new app from scratch
$ seagull new todos

# change into the directory of the new app, optionally fire up VSCode
$ cd todos && code .
````

All basic files and configurations are generated your you, as well as app
dependencies resolved (they reside in `node_modules/`). If you inspect the
central `package.json` file, you'll see only a light dependency on the Seagull
core. The heavy lifting of toolchains is outsourced into the CLI tool you just
installed above, which means *your actual app is featherlight*. Okay, let's see
that everything worked out correctly:

````bash
$ seagull dev
````

This starts a local development server on
[localhost:3000/](http://localhost:3000/). If you navigate your browser there,
you *should* see a classical "hello world" message. You can stop this dev server
any time with the key kombination `Ctrl` + `C`.

## Frontend-Layout

Now let's build some frontend stuff. You should have some basic knowledge about
what a SPA is, how react works and JSX files look like and have an idea about
the latest ECMAScript/Typescript goodies.

First things first, we update the generated `frontend/layout.tsx` file to look
like this:

````jsx
import createElement from 'inferno-create-element'

// this is what your index.html looks like
export default function Layout({ children }) {
  return (
    <html>
      <head>
        <title>Seagull ToDo App</title>
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" />
      </head>
      <body>
        <div className="container">
          <h1>Seagull ToDo App</h1>
          <div id='root' className="">{children}</div>
        </div>
        <script src="/assets/bundle.js"></script>
      </body>
    </html>
  );
}
````

This is the equivalent of "index.html" files in other frameworks. We added
Bootstrap3 to it for some basic styles and responsiveness. Note that in TSX,
every tag must be closed properly (even the `link` in the `head` section) and
the `class` attribute of HTML is called `className`. Since TSX is just
Javascript in the end, the word "class" has a special meaning in JS already, so
this got mapped. If you reload the page in your browser, you should notice the
changes (you still have `seagull dev` running, have you?).

## Frontend-State

We need some model to represent a single todo item. The next steps are
a bit of data plumbing and show you how easy it is to wire things up.
Create a file `frontend/models/todo.ts` with the following content:

````typescript
// the reactive state management library of choice
import { observable } from 'mobx'

// this is a single todo item
export default class Todo {
  // id will never change after creation
  id: string

  // checked will change on user interactions
  @observable checked: boolean = false

  // text will never change after creation
  text: string

  // create a new instance from a given text and create a uuid
  constructor (text: string) {
    this.text = text
    this.id = 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, (c) => {
      const r = Math.random()*16|0, v = c == 'x' ? r : (r&0x3|0x8)
      return v.toString(16)
    })
  }
}
````

This is just a ordinary typescript class. An important note is the mobx-import
on top: we will use a reactive state management library called
[MobX](https://mobx.js.org) for the frontend. Seagull ships with MobX already,
so there is no need for installing it on your own. We also
[decorate](https://www.typescriptlang.org/docs/handbook/decorators.html) the
`checked` property as `observable`, which means that whenever this property
changes, some part of the UI might re-render. Because we want to reference
individual todo items later on, we generate a uuid on creation.

Now that we have an idea about how a todo item looks like, we need a store
which manages a list of todo items. This store should also include just enough
functionality to perform the basic operations needed: add a new todo, toggle
specific todos as checked or unchecked, delete a specific todo. Create a file
`frontend/stores/todos.ts`:

````typescript
import { observable } from 'mobx'
import Todo from '../models/todo'

export default class TodoStore {
  @observable list: Todo[] = []

  constructor() {
    this.addTodo('wake up')
    this.addTodo('go shopping')
    this.addTodo('buy milk')
  }

  addTodo (text: string) {
    this.list.push(new Todo(text))
  }

  deleteTodo (id: string) {
    const index = this.list.findIndex(todo => todo.id === id)
    this.list.splice(index, 1)
  }

  toggleTodo (id: string) {
    const index = this.list.findIndex(todo => todo.id === id)
    this.list[index].checked = !this.list[index].checked
  }
}
````

For a bit more convenience, some todos are preloaded on creation. Note how this
again is just an ordinary typescript class, with the small addition that the
`list` property is decorated as `observable`.

## Frontend-Routing

Until now, these files are not used yet in the demo frontent, so let's change
that. All the wiring happens in the `frontend/routes.tsx` file, which you
should modify to look like this:

````jsx
// library imports
import { history } from '@seagull-js/seagull'
import createElement from 'inferno-create-element';
import { Provider } from 'inferno-mobx'
import { Route, Router } from 'inferno-router';

// import of stores
import Todos from './stores/todos'

// import of individual pages
import HelloPage from './pages/hello'

// routing structure
const routes = (
  <Provider todos={ new Todos() }>
    <Router history={ history }>
      <Route path='/' component={ HelloPage }/>
    </Router>
  </Provider>
)

export default routes
````

Here we just imported our todos store and made it available to all routes and
their components by wrapping all with a `Provider`. If you have more stores,
just add more attributes to the one Provider.

Now that the data and business logic is in-place, it is time to build the actual
components!

## Frontend-Components

We start with the individual todo item, which will be rendered
within a table. Create a file `frontend/components/todo_item.tsx`with this
content:

````jsx
import Component from 'inferno-component';
import createElement from 'inferno-create-element';
import { connect } from 'inferno-mobx'
import Todo from '../models/todo'
import Todos from '../stores/todos'

interface IProps {
  todo: Todo,
  todos: Todos
}

@connect(['todos'])
export default class TodoItem extends Component<IProps, {}> {
  render() {
    return (
      <tr>

      {/* column 1: the checkbox for the 'checked' state */}
      <td>
        <input
          type="checkbox"
          checked={ this.props.todo.checked }
          onClick={ e => this.props.todos.toggleTodo(this.props.todo.id) }
        />
      </td>

      {/* column 2: the actual text, either striked through or not */}
      <td>{ this.props.todo.checked ? (
        <strike>{ this.props.todo.text }</strike>
      ) : (
        <span>{ this.props.todo.text }</span>
      )}</td>

      {/* column 3: a delete button for this todo item */}
      <td>
        <button
          onClick={ e => this.props.todos.deleteTodo(this.props.todo.id) }
          className="btn btn-danger btn-xs"
        >delete</button>
      </td>

    </tr>
    )
  }
}
````

The `Component` class from which we're deriving here is generic over:

- the props we're passing into the component (here: IProps)
- the internal state the component will hold (here: {}, which means none)

We will pass in a specific `Todo`, and the `connect` decorator will extend the
props to include the exact store we just provided in the `routes.tsx` above.
This is *the* central magic of the frontend with inferno. No need for complex
Flux architectures or setting up massive amounts of boilerplate, there are just
(reactive) stores, components, and some connection between them.

Also note that since we defined our todos store explicitly within `IProps`, we
get fully typesafe intellisense while writing this all.

Now we're increasing the complexity a bit. We need a component which contains an
text input field with a submit button, which triggers the `addTodo` method of
our todos store. Create a file `frontend/components/input_form.tsx`:

````jsx
// external library imports
import createElement from 'inferno-create-element';
import Component from 'inferno-component';
import { connect } from 'inferno-mobx'
import Todos from '../stores/todos'

// external data that gets passed into this component
export interface IProps {
  todos: Todos
}

// internal 'state' data structure of this component
export interface IState {
  input: string
}

// the (stateful) component for the page with type checking
@connect(['todos'])
export default class InputForm extends Component<IProps, IState> {
  constructor(props) {
    super(props)
    this.state = { input: '' }
  }

  setInputText = (event) => this.setState({ input: event.target.value })

  invoke = (_event) => {
    this.props.todos.addTodo(this.state.input)
    this.setState({ input: '' })
  }

  render() {
    return (
      <div className="input-group">
        <input
          className="form-control"
          value={ this.state.input }
          onChange={ this.setInputText }
          placeholder="enter some todo text..."
        />
        <span className="input-group-btn">
          <button className="btn btn-default" onClick={ this.invoke }>add todo</button>
        </span>
      </div>
    )
  }
}
````

We will keep the value of the input field in sync with the internal state of
the component, and on button click trigger our store method and reset the input
afterwards. The difference compared with the above component is just that we
here use a bit of an internal state (statically typed as `IState`) for some
component-local caching of a value until we really need it.

Now there is only one last step to do: updating our `frontend/pages/hello.tsx`
page to show us the new logic. Add the following code:

````jsx
import Component from 'inferno-component';
import createElement from 'inferno-create-element';
import { connect } from 'inferno-mobx'
import Todos from '../stores/todos'
import InputForm from '../components/input_form'
import TodoItem from '../components/todo_item'

interface IProps {
  todos: Todos
}

// the (stateful) component for the page with type checking
@connect(['todos'])
export default class HelloPage extends Component<IProps, {}> {

  render() {
    return (
      <div>
        <table className="table table-bordered table-hover">
          <thead>
            <tr>
              <td>Status</td>
              <td>Text</td>
              <td>Actions</td>
            </tr>
          </thead>
          <tbody>
            {this.props.todos.list.map((todo, index) =>
              <TodoItem todo={ todo } />
            )}
          </tbody>
        </table>
        <InputForm />
      </div>
    )
  }
}
````

And reload your browser. You should see a fully functional Todo-App in front
of you! Congratulations, you just mastered the probably most difficult part
of modern web development: building complex single page apps.

## Backend-SSR

As you might know, a SPA is rendered dynamically on the browser, which means
that the browser must load some initial html file with a bunch of scripts,
execute those scripts, optionally fetch data from an API and then renders the UI.
Quite a few steps until a user can "see" something. And what about old school
bots and scrapers, what do they see? the initial (empty) HTML? Does Google
really execute Javascript while crawling (they do, a bit) and do they wait for
data fetching before pagers can be rendered (currently: mostly not)?

These are the actual weak points of a SPA: *initial load time* for users and
*crawlability for bots*. The solution for these issues is called SSR. Since
Seagull builds upon node.js for all things backend, it can natively execute the
todo frontend you just built above. Need a proof? Just click the `view source`
button in your browser or run `curl http://localhost:3000/` in a terminal.
There is fully rendered HTML, instantly!

You didn't have to configure anything by yourself, the magic happens within the
`api/Frontend.ts` api handler which got generated when you started the project
with `seagull new ...`.

## Deployment

todo