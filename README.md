<h1 align="center">Leap</h1>

`Leap` (**L**ua **e**xtension **a**ccessibility **p**re-processor) is a fast pre-processor of a "modernized" and extended version of lua that pre-processes in pure executable lua.  
Think of it as an effective "modernity" leap into the future to make lua a feature-rich language.  

Leap is inspired by the functionality and syntax found in [JS](https://www.javascript.com) and is primarily intended for [**FiveM**](https://fivem.net), this however does not deny the possibility of extension to wider horizons.

## Usage
To use leap you can simply download it and use its functions within the resource at your choice (you will need to add leap between the resource's dependencies), when the resource will start leap will take the job of preprocessing the necessary files.

Example:
`your_resource_that_use_leap > fxmanifest.lua`:
```lua
fx_version "cerulean"
game "gta5"

server_script "server.lua"

dependency "leap" -- This is necessary to have the resource automatically preprocessed
```

You can also directly use the `leap restart your_resource_that_use_leap` command to preprocess the file directly with shadow writing.

### Escrow

To use leap on the escrow or outside the leap ecosystem you can make leap build the files into a standalone version with the command `leap build your_resource_that_use_leap`


## Under the hood
### Shadow writing
Leap uses a **shadow writing** system (*we called it that*) that works in the following way:
After preprocessing the files, [**FiveM**](https://fivem.net) will read the files, cache them and start the resource with those cached files as soon as we start the resource, we instantly rewrite the old files so that it looks like nothing happened. This will be done in very few ms (5/10), so from [VSC](https://code.visualstudio.com) or any other IDE that has the auto refresh feature when updating a file it will look like it was never overwritten

<div align="center">
 <img src="https://user-images.githubusercontent.com/55803068/223179277-45c58f41-6a65-4e42-a18e-a5aa786155e0.png"  width="60%" height="60%">
</div>

## Commands
### leap restart
`leap restart <resource>` restarts the resource by preprocessing it before restarting it using *shadow writing*, this command is quite useful when creating a resource, since you don't have to rebuild it every time and restart the resource but it's like everything in one command.

**TIP**: these commands can also be used in the cfg after the leap startup.

---

### leap build
`leap build <resource>` pre-processes the resource by creating a subfolder named `build` that contains the pre-processed version of the resource

resource structure tree after build:
```
│   fxmanifest.lua
│
├───build
│   └───server
│           main.lua
│
└───server
        main.lua
```

---

### leap rebuild
> **Warning** Command designed only for development

`leap rebuild` rebuild with esbuild directly from fxserver instead of having to open a separate process (simply run `npm run build`)

## Features

### Arrow function
An arrow function expression is a compact alternative to a traditional function expression.
Is simply an alternative to writing anonymous functions

[Read more here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)

Syntaxes:
```lua
(param1, paramN) => {
  -- code
}
```
```lua
param => {
  -- code
}
```

Example:
```lua
Citizen.CreateThread(() => {
    print("test")
})

AddEventHandler("eventName", (text) => {
    print(('I just received %s'):format(text))
})
```

---

### Classes
Classes are a model for creating objects (a particular data structure), providing initial values for state (member variables or attributes), and implementing behavior (member functions or methods).  
It is possible as well to extend already existing classes, each method of the class that extends the other class will have as a base a variable named `super` which is an instantiated object of the original class, calling this variable as a function will call the constructor of the original class, otherwise the data of the original class can be accessed.  
Constructor parameters are those passed when a new object is instantiated. [Read more here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)  
> **Note** Classes have their own type, so calling `type(obj)` with an instantiated class will return the class name

---

Syntax:  
> **Note** Methods automatically hold the `self` variable referring to the instantiated object
```lua
class className {
    var1 = 1,
    constructor = function()
        print(self.var1)
    end
}
```

Example:
```lua
class BaseClass {
    someVariable = 100,
    constructor = function()
        print("instantiated base class")
    end
}

class AdvancedClass extends BaseClass {
    constructor = function()
        print("instantiated advanced class")
        self.super()
        
        -- right now they should printout the same value, since they have not been touched
        print(self.super.someVariable)
        print(self.someVariable)
    end
}

AdvancedClass()
-- output:
--[[
    instantiated advanced class
    instantiated base class
    100
    100
]]
```

### Decorators
By definition, a decorator is a function that takes another function and extends the behavior of the latter function without explicitly modifying it.
Its like [wrappers](https://en.wikipedia.org/wiki/Wrapper_function)
You can also pass parameters to the decorators.

[Read more here](https://realpython.com/primer-on-python-decorators/#simple-decorators)

Syntax:  
```lua
function decoratorName(func, a, b)
    return function(c, d)
        func(c,d)
    end
end

@decoratorName(a, b)
function fnName(c, d)
    -- code
end
```

Example:
```lua
function stopwatch(func)
    return function(...)
        local time = GetGameTimer()
        local data = func(...)
        print(func.name.." taken "..(GetGameTimer() - time).."ms to execute")
        return data
    end
end

@stopwatch
function someMathIntensiveFunction(a, b, pow)
    for i=1, 500000 do
        math.pow(a*b, pow)
    end

    return math.pow(a*b, pow)
end

someMathIntensiveFunction(10, 50, 100)
-- output: 
--[[
    someMathIntensiveFunction taken 2ms to execute
]]
```

### Default value
Default function parameters allow named parameters to be initialized with default values if no value or nil is passed.

Syntax:  
```lua
function fnName(param1 = defaultValue1, ..., paramN = defaultValueN) {
  -- code
}
```

Example:
```lua
function multiply(a, b = 1)
  return a * b
end

print(multiply(5, 2))
-- Expected output: 10

print(multiply(5))
-- Expected output: 5
```

### New
The new operator lets create an instance of an object.
Is actually converted during the preprocessing process with empty,so it is simply eliminated.
Its use is simply to make it clear in the code when you are instantiating an object or calling a function

Syntax:  
```lua
new Class()
```

Example:
```lua
class BaseClass {
    someVariable = 100,
    constructor = function()
        print("instantiated base class")
    end
}

local base = new BaseClass()
```

### Not equal (!=)
Another method of writing the not equal operator (~=) 

Syntax:  
```lua
if true != false then
    -- code
end
```

Example:
```lua
local a = 10
local b = 20

if a != b then
    print("not equal")
end
```

### Unpack (...)
> **Warning** for now this operator only works with variables, so writing `...{100, 200, 300}` will not work

This operator allows you to unpack arrays (not hashmaps) into multiple variables.
The preprocessor converts this operator into the `table.unpack` function.

Syntax:  
```lua
a,b,c = ...array
```

Example:
```lua
local numbers = {100, 200, 300}

local number1, number2, number3 = ...numbers

console.log(number1, number2, number3)
-- output
--[[
    100    200    300
]]
```

### Type checking
We have added the ability to add types in function parameters, this will give you the ability to specify the required types and prevent bugs.
> **Note** Classes can also be used as types

Syntax:  
```lua
function funcName(<type> param1)
    -- code
end
```

Example:  
```lua
function DebugText(<string> text)
    print("debug: "..text)
end


DebugText("test") -- debug: test
DebugText(100) -- Error loading script *.lua in resource ****: @****/*.lua:7: text: string expected, got number
```
```lua
class Example {
    myVar = true
}

class Another {
    myVar = false
}

local exampleObj = new Example()
local anotherObj = new Another()

function FunctionThatAcceptOnlyExampleClass(<Example> example)
    print("You passed the right variable!")
end

FunctionThatAcceptOnlyExampleClass(exampleObj) -- You passed the right variable!
FunctionThatAcceptOnlyExampleClass(exampleObj) -- Error loading script *.lua in resource ****: @****/*.lua:16: example: Example expected, got Another
```

# Building leap
As first thing, thank you for coming down this far and thus wanting to contribute to leap development :heart:.  

> **Warning** Obviously, before running the `npm run build` command or the `leap rebuild` command, you need to install the npm modules with the command `npm i`

To develop leap you just need to download or clone the repository and edit the files inside the `src` folder, the files are divided by feature and by supporting modules, the `out.js` file is the output file, you can build everything with the `npm run build` command or on the cfx server with the command `leap rebuild`.  

We use [esbuild](https://esbuild.github.io) since FiveM (from what we have tested and understood) doesnt support ES (ECMAScript) or just doesnt support the import declaration.  

after building everything and restarted leap you just need to build or restart with leap the interested resource.