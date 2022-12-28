# Extension Tutorial README

## Setting it up

There are three things you need to have installed

- [Node.js](https://nodejs.org/en/)
- [yeoman](https://yeoman.io/)
- [vs code extension generator](https://github.com/Microsoft/vscode-generator-code)

`npm install -g yo generator-code`

### Generate extension

Run `yo code` and configure some values to get the project scaffolded.

To test, run the extension with `f5`, in the new window, open the command pallete and run hello world. Super easy.

### Configuration

The [official documentation](https://code.visualstudio.com/api) has a lot of information but I’ll cover some important nodes.

- Commands
    - Assign a title and other UI elements to a command. The title is used in the Command Pallete for example
- Activation Events
    - When a command is invoked, VS Code will emit an activationEvent and the extension becomes activated
    - Can be when VS Code loads and is ready to go `onStartupFinished`
    - Can be when a given command is called `onCommand:<namespace>.<function>`
    - You must define `onCommand` for any commands that:
        - Can be invoked using the Command Palette
        - Can be invoked using a keybinding
        - Can be invoked through the VS Code UI, such as through the editor title bar
        - Is intended as an API for other extensions to consume
- Configuration
    - Provides the user with editable parameters
    - Displayed under `Preferences` > `Settings`
    - Settings are shown alphabetically but can be grouped with another level of namespace
    - Value can be string, number, boolean
    - Can add a markdown description to include links
    - Can set min/max values
    - Can set defaults

```jsx
    "configuration": {
      "title": "Tutorial",
      "properties": {
        "tutorial.string": {
          "type": "string",
          "default": null,
          "markdownDescription": "The [string](https://www.web.site) example of settings"
        },
        "tutorial.number": {
          "type": "number",
          "default": 0.7,
          "minimum": 0,
          "maximum": 1,
          "markdownDescription": "The [number](https://www.web.site) example of settings"
}
```

>Commands, activation events, and configuration are namespaced.

- Keybindings
    - Can associate a command with a keybinding
    - `key` for Windows and `mac` for Apple
    - Can set scope of when keybinding applies (ex. editorTextFocus)
    - User always has the ability to change/override keybindings

```jsx
"keybindings": [
      {
        "command": "tutorial.helloWorld",
        "key": "alt+h",
        "mac": "option+h",
        "when": "editorTextFocus"
      }
]
```

- Metadata
    - Data displayed on Marketplace page
        - publisher, name, description
        - links (repo, bugs, homepage)
        - icon/banner colour
        - categories/keywords


### Logic

The `activate` function is what is called by the activation event. It returns a ‘disposable’ object, which allows for safe cleanup. You’ll typically find these for event listeners, timers, and commands.

At the end of the `activate` function, you need to make sure that each command is registered with
`context.subscriptions.push(disposable);`

If you don’t include this line of code for each command, the omitted command will still be registered and will still be available for the user to execute. However, if the extension is deactivated or uninstalled, the command will not be cleaned up properly, which could potentially lead to memory leaks or other issues. Just like a hike in the woods, be sure to clean up after yourself

Inside the `activate` function, you will register the commands that you defined in `package.json`. 

```jsx
let disposable = vscode.commands.registerCommand('tutorial.helloWorld', () => {
		vscode.window.showInformationMessage('Hello World from extension-tutorial!');
	});
```

The default command is very basic, it just displays an information message.

The `vscode` object is very powerful

- can interact with the editor environment, tests, commands, other extensions

The `vscode.window` object is featured in this tutorial and has a lot of capabilities for you to take advantage of such as

- interactive with terminals, colour theme, or notebook
- show input boxes, status bar items, or error messages

To selected highlighted code:

- Get the editor with `const editor = vscode.window.activeTextEditor;`
- Get the selected text - `const selectedText = editor.document.getText(editor.selection);`

Now that we can capture text from the user, here’s how you integrate with OpenAI

### OpenAI

run `npm install openai`

Import into your file with `import { Configuration, OpenAIApi } from "openai";`

Configure your API Key

```jsx
const configuration = new Configuration({
				apiKey: OPENAI_API_KEY,
			});
```

    ⚠️Big important security note!⚠️

    I have seen too many extensions that store API Keys in settings. This is completely insecure. All extensions have access to all settings, so if you store it there (which is in plain text) any malicious extension can take your key. What you need to do is store the key in `Secret Storage` which is scoped to each individual extension.

    `await context.secrets.store(OPENAI_API_KEY, secret);`

    - where OPENAI_API_KEY is the key and `secret` is your API Key from OpenAI

To pass the configuration object into a new instance of OpenAIApi

```jsx
const openai = new OpenAIApi(configuration);
```

To create a completion by making a request to OpenAI

```jsx
	const response = await openai.createCompletion({
				model: "text-davinci-003",
				prompt: `${prompt}`,
				max_tokens: 250,
				temperature: 0.4,
			});
```

`model` - the OpenAI model you want to use

`prompt` - the text you want to send to GPT-3  for a response. You can send the selected text directly, or you can build a stronger prompt by adding context/direction before or after the user’s text

`max_tokens` - the maxmimum number of tokens for the prompt + request. Higher value = possible for longer responses but more expensive. Remember prompts count too, so a more detailed prompt uses more tokens

`temperature` - the sampling temperature - higher values = more risk

---


When you get your response, you can capture the output with
`const output = response.data.choices[0].text`

- choices is an array because you can set a `n` variable to return more than one completion for a given promp. But default `n` is 1

And that’s it. It really is that easy. You can now have a user highlight code, hit a keybinding to run a command, have that command take the highlighted code and insert it into a prompt for GPT-3, then get a response that you can present to the user. 

It's an amazing feeling to see someone using a tool that I've built and knowing that it is helping them in some way. It could be something as small as saving them a few minutes of time each day or as significant as changing the way they work for the better. With VS Code extensions, knowing the potential impact is 10s of millions of developers is so exciting