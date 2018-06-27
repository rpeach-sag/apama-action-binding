# Action Binding [![Build Status](https://travis-ci.org/rpeach-sag/apama-action-binding.svg)](https://travis-ci.org/rpeach-sag/apama-action-binding) [![Coverage Status](https://coveralls.io/repos/github/rpeach-sag/apama-action-binding/badge.svg)](https://coveralls.io/github/rpeach-sag/apama-action-binding)

Action binding brings a popular JavaScript feature to [Apama](http://www.apamacommunity.com/)'s Event Programming Language (EPL): Function Binding.
```javascript
action<any> helloName := Bind.create(concatStrings)
	.bind1("Hello ")
	.bindRight1("!")
	.call1;
...
helloName("World"); // Prints: Hello World!
helloName("Developer"); // Prints: Hello Developer!
...
action concatStrings(string a, string b, string c) {
	log a + b + c;
}
```
## Contents
* [Installation](#install)
* [Quickstart](#quickstart)
* [When is it useful?](#usefulness)
* Operators
    *  [Bind](#bind) - Bind the arguments
    *  [Apply](#apply) - Call the action
    *  [Call](#call) - Call the action (No return)

## <a id="install"></a>Installation
First head over to the [Release Area](https://github.com/rpeach-sag/apama-action-binding/releases) and download the latest release.

The deployment script (Inside the release) provides a way to make ActionBinding globally available to all SoftwareAG Designer workspaces.
### 1. Installing into Designer
1. Place the ActionBinding folder somewhere safe (somewhere not likely to be moved or deleted)
2. Run the deploy.bat
3. Follow the instructions
4. Restart any running instances of SoftwareAG Designer

### 2. Adding to a Project
1. From SoftwareAG Designer right click on your project in `Project Explorer`
2. Select `Apama` from the drop down menu;
3. Select `Add Bundle`
4. Scroll down to `ACTION_BINDING/bundles` and select `ActionBinding`
5. Click `Ok`

When run via Designer, it will automatically inject all of the dependencies.

### 3. Packaging a project (For use outside Designer)
The Apama tool `engine_deploy` packages a project so that it can be run outside of designer.
1. Start an Apama Command Prompt (Start menu, Software AG, Tools, Apama, Apama Command Prompt)
2. `cd` to your project directory
3. Run `engine_deploy --outputDeployDir output.zip . <action_binding_install_dir>/actionbinding.properties`.
You'll end up with a zip of your entire project.
4. Unzip it on whichever machine you'd like to use the project.
5. Run `correlator --config initialization.yaml --config initialization.properties` from within the unzipped directory to run the project.

## <a id="quickstart"></a> Quickstart
1.  First check out the  [Installation Instructions](#install), steps 1 & 2.
2. Start Software AG Designer.
3. Copy the following code into a new file: "Main.mon"


```javascript
using com.industry.actions.Bind;

monitor Main {
	action<any> returns any helloName := Bind.create(concatStrings).bind1("Hello ").bindRight1("!").apply1;
	
	action onload() {
		log <string> helloName("World");
		log <string> helloName("Developer");
	}
	
	action concatStrings(string a, string b, string c) returns string {
		return a + b + c;
	}
}
```
4.  Run the project.

## <a id="usefulness"></a> When is it useful?
Action binding is most useful in a functional programming approach, where you are passing actions around to perform operations on data. It is particularly useful when used in conjunction with libraries such as [RxEPL](https://github.com/SoftwareAG/apama-rxepl) and [Lambdas](https://github.com/SoftwareAG/apama-lambdas). 
For example:
```javascript
action startDeviceAnalytics(Device d) {
	any analytics := Observable.fromChannel("DeviceData")
		.filter(Bind.create(isDataFromDevice).bindRight1(d).apply1)
		...
}

action isDataFromDevice(Data data, Device d) returns boolean {
	return data.sourceId = d.id;
}
```
## <a id="bind"></a> Different Types of Binding

**bind/bindRight** - Takes a sequence of arguments to apply to the leftmost/rightmost arguments of the function
```javascript
Bind.create(concatStrings).bind(["Hello ", "World"]).apply1("!");
// returns "Hello World!"
Bind.create(concatStrings).bindRight(["World", "!"]).apply1("Hello ");
// returns "Hello World!"
```
**bind1/bindRight1** - Takes a single argument to apply to the leftmost/rightmost arguments of the function
```javascript
Bind.create(concatStrings).bind1("Hello ").apply2("World", "!");
// returns "Hello World!"
Bind.create(concatStrings).bindRight1("!").apply2("Hello ", "World");
// returns "Hello World!"
```
**bind2/bindRight2** - Takes 2 arguments to apply to the leftmost/rightmost arguments of the function
```javascript
Bind.create(concatStrings).bind2("Hello ", "World").apply1("!");
// returns "Hello World!"
Bind.create(concatStrings).bindRight2("World", "!").apply1("Hello ");
// returns "Hello World!"
```
**bindIndex** - Bind any argument, just pick the index of the argument
```javascript
Bind.create(concatStrings).bindIndex(1, "World").apply2("Hello ", "!");
// returns "Hello World!"
```
## <a id="apply"></a>Apply

Apply allows you to call a bound action and get a returned value. (Use [Call](#call) if you don't want a return)

**apply/getGenericAction()** - Call the action with all arguments from the provided sequence and return a value.
```javascript
Bind.create(concatStrings).bind1("Hello ").apply(["World", "!"]);
// returns "Hello World!"
``` 
**apply0** - Call the action with no arguments and return a value. Useful if all arguments have already been bound.
```javascript
Bind.create(concatStrings).bind3("Hello ", "World", "!").apply0();
// returns "Hello World!"
``` 
**apply1** - Call the action with 1 argument and return a value.
```javascript
Bind.create(concatStrings).bind2("Hello ", "World").apply1("!");
// returns "Hello World!"
``` 
**apply2** - Call the action with 2 arguments and return a value.
```javascript
Bind.create(concatStrings).bind1("Hello ").apply1("World", "!");
// returns "Hello World!"
``` 
**apply3** - Call the action with 3 arguments and return a value.
```javascript
Bind.create(concatStrings).apply3("Hello ", "World", "!");
// returns "Hello World!"
``` 
## <a id="call"></a>Call
Call allows you to call a bound action without getting a returned value. (Use [Apply](#apply) if you want a return)

**call** - Call the action with all arguments from the provided sequence but don't return a value (Even if the original returned a value).
```javascript
Bind.create(concatStrings).bind1("Hello ").call(["World", "!"]);
// returns "Hello World!"
``` 
**call0** - Call the action with no arguments but don't return a value (Even if the original returned a value). Useful if all arguments have already been bound.
```javascript
Bind.create(concatStrings).bind3("Hello ", "World", "!").call0();
// returns "Hello World!"
``` 
**call1** - Call the action with 1 argument but don't return a value (Even if the original returned a value).
```javascript
Bind.create(concatStrings).bind2("Hello ", "World").call1("!");
// returns "Hello World!"
``` 
**call2** - Call the action with 2 arguments but don't return a value (Even if the original returned a value).
```javascript
Bind.create(concatStrings).bind1("Hello ").call2("World", "!");
// returns "Hello World!"
``` 
**call3** - Call the action with 3 arguments but don't return a value (Even if the original returned a value).
```javascript
Bind.create(concatStrings).call3("Hello ", "World", "!");
// returns "Hello World!"
``` 
------------------------------

These tools are provided as-is and without warranty or support. They do not constitute part of the Software AG product suite. Users are free to use, fork and modify them, subject to the license agreement. While Software AG welcomes contributions, we cannot guarantee to include every contribution in the master project.
