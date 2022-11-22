# Passing arguments to npm commands

In many cases we find ourselves in situations where we need to automate specific operations, npm brings us the option of creating custom commands that make these operations much easier. The only problem is when it starts to become "spam", a very large amount of commands makes `package.json` very large and doesn't scale.

In this example, we will use typescript (with execution guaranteed by `ts-node`), but this does not prevent us from using basic JavaScript to create the commands/scripts.

## The implementation

First of all, let's install the `ts-node` in your project, like the example:

```bash
npm i ts-node --save-dev
```

Go to your `package.json` and create a npm command with the name that you want, like the example:

```json
"scripts": {
    "example": "ts-node src/scripts/hello.ts"
}
```

Now, create a file in the patch `src/scripts` called `hello.ts` with the content:

```ts
#! /usr/bin/env node

const args = process.argv.slice(2)
```

The `slice(2)` will strip the command execution arguments and get only the arguments passed by the user who is actually executing the code.

Now, `args` will contain all your passed arguments and you can do anything with that!

## What now?

You can also use these arguments to invoke new terminal commands like the example:

```ts
#! /usr/bin/env node

const args = process.argv.slice(2);
const echo = spawn('echo', args, { stdio: "inherit" });

echo.on('data', console.log)
```
