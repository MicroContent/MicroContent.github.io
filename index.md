# MicroContent for Social MicroLearning

## Concept and Idea

MicroContent is a type of digital learning material that is
* small (size, time to consume)
* interactive (requires the user to do something and gives feedback)

Clearly many types of content fullfil these criteria
and it is impossible to implement and include each potentially desired content type into a single system.

Therefore, for our Social MicroLearning system we use the concept of external or third-party content types.
A content type provider has to provide two things:
* a content viewer that is able to display data appropriatly
* an editor that is able to edit the data or create a new instance

Those two elements have to be permanently hosted as static html content. CORS has to be enabled. We recommend to use githubs raw-view. This also has the advantage that you can directly adress a specific version of the viewer/editor. [Example](https://raw.githubusercontent.com/MicroContent/BinaryNumberContentViewer/6611935c2494d958bbd765a9fa0ddcc1094e0713/index.html).

## Content Editor

A content editor should provide means to enter the data for a specific instance of your learning content type. The [BinaryNumberContentEditor](https://github.com/MicroContent/BinaryNumberContentEditor) for example provides a text field to enter a number between 0 and 255.
An editor has two methods that allow communication with it's hosting environment:
* `getData()`
* `setDataGetter(func)`

The getData function returns an object containing data for the editor (in case a user edits existing content) or undefined if there is no data yet (in case a user creates new content).

The setDataGetter function takes a parameterless function as argument. The function should return a data object representing the current state of user input. The host environment will call this function to store the data for this instance.

MicroContent is often shared through social media and other communication channels using link preview. The following properties of your data will be picked up for link previews by the host system as a convention:
* `title`
* `description`
* `imageUrl`


## Content Viewer

A content viewer shall display data and allow users to interact with it. The host environment provides a submit solution button to users and content viewers should be designed accordingly. The host environment provides the following methods:
* `getData()`
* `propagateLayoutChanges()`
* `sendXapiStatement(statement)`
* `registerOnSubmitListener(listener)`
* `setXapiObjectGetter(getter)`
* `getBoundingClientRect()`
* `injectIframeWithSrc(containerClass, src)`

### getData (important)

The getData function returns a data object as stored with the editor. The data object essentially defines an instance of the MicroContent type. E.g. a multiple choice card might have a title (string), a question (string) and an answer array. You should call the `getData` function when/before you initialize your view.

### propagateLayoutChanges (often necessary)

The propagateLayoutChanges function informs the host environment that the dimensions of the content has changed.
This might be necessary when you change your view in a way that influences its bounds. E.g. if you display a hint after a user clicks on a button and your content needs more space. You might also need this when you display the solution (see `registerOnSubmitListener`).

### sendXapiStatement (very recommended)

The sendXapiStatment function is for logging learning relevant user (inter)actions. It takes an xAPI statement as a parameter.
The xAPI or Experience API is a proposed standard to log, track and analyze learning activity data. Its heart is the a RESTful Endpoint to send and receive so called xAPI-Statements. You can find the full spec [here](https://github.com/adlnet/xAPI-Spec/blob/master/xAPI-Data.md). You can play around with the [xapi-lab](http://adlnet.github.io/xapi-lab/), an online statement generator to test what statements can look like. The essence of a statement are actor, verb and object (Who did what with/to what?). A MicroContent unit does not handle authentication, object identification and can be embedded in various contexts. Therefore the host environment will add actor, object.objectId and context.platform to the statement after you pass it to `sendXapiStatement` and before it is actually sent to an xAPI Endpoint. The host environment also takes care about where to send it to and how to authenticate the statement.

### registerOnSubmitListener (important)

The registerOnSubmitListener takes a parameterless function that will be called when the user clickes on the submit solution button in the host enviroment. It should present the solution, feedback and insight to the user.
MicroContent follows the pattern of beeing submittable. The host that embedds the MicroContent will also provide a submit button and call the registered function. Typically you want to display the solution (which may need more space, see `propagateLayoutChanges`) and send an xAPI statement with a verb like "answered" or "attempted".

### getBoundingClientRect (if needed)

The getBoundingClientRect function returns the bounding rectangle of the embedded content and is needed to calculate relative mouse positions on certain elements of the content (e.g. canvas onClick handler).
If you don't use canvas, you probably wont need it and can ignore it.
As google-caja does not allow you to get information about the DOM outside of your box, this is the only way to calculate coordinates of mouse events on a canvas. If you want to see an example for that take a look at the [BinaryNumberContentViewer](https://github.com/MicroContent/BinaryNumberContentViewer).

### injectIframeWithSrc (if needed)

The injectIframWithSrc function allows embedding videos from youtube or vimeo.
If you don't need that, you can ignore it.
Since the html is sanitized it is not possible to use iframes with src inside. The function takes two parameters `containerClass` which is used to pick the element where the iframe is injected (class names are mangled during sanitizing) and `src` which should be a href to the youtube video (use youtube-nocookie.com) or vimeo video. The method only allows whitelisted sources.

## Examples

You can find examples in the github organization [MicroContent](https://github.com/MicroContent).

You can start from the simple empty templates [AbstractContentEditor](https://github.com/MicroContent/AbstractContentEditor) and [AbstactContentViewer](https://github.com/MicroContent/AbstractContentViewer).

## Get Started

To get started you only need a text editor.
To make your first steps as easy as possible we provide a MicroContent development toolset. It provides a static http server for local development, that already includes a MicroContentTestHost. You can find it [here](https://github.com/MicroContent/McToolset). Just download the [latest release](https://github.com/MicroContent/McToolset/releases) and run it as described and you're good to go!

Alternatively you could use any other static content serving utility like [http-server](https://www.npmjs.com/package/http-server) and the [MicroContentTestHost](https://github.com/bgoeschi/MicroContentTestHost). If you're interrested you can also have a look at the code of the MicroContentTestHost and read the documentation of [google-caja](https://developers.google.com/caja/) to better understand what is going on under the hood.
