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

## Frontend-Components

ToDo: intro about TSX components, build a few of them, ... .

````typescript
// external library imports
import Component from 'inferno-component';
import createElement from 'inferno-create-element';

// better statically type the central data structure :-)
export interface ITodo {
  text: string
  checked: boolean
}

// internal 'state' data structure of this component
export interface IState {
  list: ITodo[]
  input: string
}

// the (stateful) component for the page with type checking
export default class TodosPage extends Component<{}, IState> {
  constructor(props) {
    super(props);
    this.state = {
      list: [],
      input: ''
    }
  }

  setInputText = (event) => this.setState({ input: event.target.value })

  addTodo = (event) => {
    const newTodo = { checked: false, text: this.state.input }
    this.setState({ input: '', list: this.state.list.concat(newTodo) })
  }

  deleteTodo = (index: number) => {
    const list = this.state.list
    list.splice(index, 1)
    this.setState({ list })
  }

  toggleChecked = (index: number) => {
    const list = this.state.list
    list[index].checked = !list[index].checked
    this.setState({ list })
  }

  render() {
    return (
      <div>
        <table>
          <tbody>
            {this.state.list.map((todo, index) =>
              <tr>
                <td><input type="checkbox" checked={ todo.checked } onClick={ e => this.toggleChecked(index) } /></td>
                <td>{ todo.checked ? (
                  <strike>{ todo.text }</strike>
                ) : (
                  <span>{ todo.text }</span>
                )}</td>
                <td><button onClick={ e => this.deleteTodo(index)}>delete</button></td>
              </tr>
            )}
          </tbody>
        </table>

        <div>
          <input type="text" value={ this.state.input } onChange={ this.setInputText } placeholder="enter some text" />
          <button onClick={ this.addTodo }>add todo</button>
        </div>
      </div>
    )
  }
}
````