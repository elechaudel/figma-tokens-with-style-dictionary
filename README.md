# Use Figma Tokens Plugin with Style Dictionary

This repository serves as an example to illustrate how you can locally transform your tokens stored on [Figma Tokens](https://docs.tokens.studio/) so they are up and ready for [Style Dictionary](https://amzn.github.io/style-dictionary/#/) to compile.

---

1. [Why the need for Token Transformer?](#why-the-need-for-token-transformer)
2. [Tokens Structure](#tokens-structure)
3. [How to use this project in 3 steps](#how-to-use-this-project-in-3-steps)
4. [Other considerations](#other-considerations)

---

First of all, let me begin with why we need [Token Transformer](https://www.npmjs.com/package/token-transformer) in the first place.

## Why the need for Token Transformer?

**The short answer is:** The way tokens are referenced across Token Sets in the Figma Plugin is not reflected in the Token Keys after tokens are exported. This results in Style Dictionary not being able to retrieve the token references.

For more information, please read this [thread on Github](https://github.com/six7/figma-tokens/issues/691#issuecomment-1144889277) and the [suggested workflow by @jam-awake](https://github.com/six7/figma-tokens/issues/691#issuecomment-1144984836).

In the end, Token Transformer helps us remove any aliases that would result in errors during the Style Dictionary build. Token Transformer prepares a file that only output raw values that Style Dictionary can process.

## Tokens Structure

At Apart, our tokens follow a multi-layers approach.

| Layer | Layer name | Description |
|---|---|---|
| 1 | Core<br>_(private tokens)_ | Tokens in this Layer store the raw values and with this build the basis of the Design Tokens. This Layer is mainly responsible for the look of the final product by defining all the values that can be used. |
| 2 | Semantic<br>_(public tokens)_ | Tokens in this Layer reference Core Tokens. Their name describes the intended use of the Token. Tokens in this Layer are those which are used the most throughout the application. They represent the choices the Design team made in regards to when to use which Token. |
| 3 | Component<br>_(scoped and overwrite tokens)_ | Tokens in this Layer reference Semantic Tokens and tie them to a specific Component value. A Component Token abstracts the value of a Semantic Token for use in a specific context or for a specific purpose. The name of the Component Token makes it clear where and how it applies. |

for more information read our [Design Tokens Guide](https://docs.google.com/document/d/1riMIYd7VIVZzHqp4x13GQ_DD3lg0E2YxrzpykbVkE9I/).

### Tokens structure in the Figma Tokens Plugin

To replicate this layers structure in the Figma Tokens Plugin we use [Token Sets](https://docs.tokens.studio/themes/token-sets). A feature that allows us to semantically separate tokens and keep having the ability to make references (aliases) between them.

In a basic setup, we usually end up with two main Token Sets:
- ```core``` (all the private tokens)
- ```public``` (all the semantic and component tokens that reference core tokens)

A third Token Set can also co-exist but is not useful for developers: ```mobile-overrides```. This Token Set can be ignored. This is for design purposes only.

### Tokens structure after Figma Tokens file export

Figma Tokens generates for you a single file with all your tokens organised by Token Set. The exported file should look something like this:

```jsonc
// ./tokens/tokens.json
{
    "core": {...},
    "public": {...}
    // mobile-overrides if it exists
    // ... Plus any other Token Sets you might have created
}
```

With this file, references between public tokens and core tokens do not work out of the box with Style Dictionary. That is where Token Transformer comes into play.

### Tokens structure after the Token Transformer compilation

Token Transformer generates a tokens file that get rid off any math operations or aliases only resulting in raw values. Your transformed tokens file should look something like this:

```jsonc
// ./transformed-tokens/tokens.json
{
    "core": {...},
    // List of all the basic public tokens you make use of organized by family:
    "font": {...},
    "color": {...},
    "space": {...},
    "size": {...},
    "opacity": {...},
    "borderWidth": {...},
    "borderRadius": {...},
    "boxShadow": {...},
    // ... Plus any component tokens
}
```

From there, you can use this file for your Style Dictionary build.

## How to use this project in 3 steps

_Do not forget to run ```npm i``` to install all dependencies first._

### Step 1 - Export your tokens from the Figma Tokens Plugin

If your tokens are not sync anywhere:
1. Open your Figma Project
2. Open the Figma Tokens Plugin
3. Click Export at the bottom of the window
4. Check "All token sets"
5. Check "Expand Typography" and "Expand Shadows"
6. Click download and name your file tokens.json

If your tokens are sync and stored somewhere, maybe you can automate a flow and query this file at build time (this workflow is not documented here).

### Step 2 - Transform your tokens with Token Transformer

With your ```tokens.json``` file available locally in your project you can run the ```transform-tokens``` command. This command expect your ```tokens.json``` file to be under a tokens directory at the root of your project: ```tokens/tokens.json```.

This command will generate a ```transformed-tokens``` directory with a ```tokens.json``` file inside.

### Step 3 - Compile your tokens with Style Dictionary

Define whatever you want Style Dictionary to do with your tokens in the ```sd.config.json``` file. Then, you can run the ```style-dictionary``` command. This command expect your transformed tokens to be in the ```transformed-tokens``` directory: ```transformed-tokens/tokens.json```.

This command will generate a ```build``` directory with the output results configured in your Style Dictionary config file: ```sd.config.json```.

For example, the default configuration integrated to this project will generate a ```_variables.sass``` file and a ```variables.js```file.

For more information, see [Style Dictionary configuration options](https://amzn.github.io/style-dictionary/#/config).

## Other considerations

### Use transformers to get unit formats

By using this project without any additional Style Dictionary configuration you might end up with font, size or shadows properties without unit formats like %, em, px, ... You can use transformers to do that at build time. Read this [github thread](https://github.com/six7/figma-tokens/issues/703#issuecomment-1130246775) for more information.