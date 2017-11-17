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

The getData function returns a data object as stored with the editor.

The propagateLayoutChanges function informs the host environment that the dimensions of the content has changed.

The sendXapiStatment function is for logging learning relevant user (inter)actions. It takes an [xAPI statement](https://github.com/adlnet/xAPI-Spec/blob/master/xAPI-Data.md) as a parameter. The host environment will add actor, object.objectId and context.platform.

The registerOnSubmitListener takes a parameterless function that will be called when the user clickes on the submit solution button in the host enviroment. It should present the solution, feedback and insight to the user.

The getBoundingClientRect function returns the bounding rectangle of the embedded content and is needed to calculate relative mouse positions on certain elements of the content. (e.g. canvas onClick handler).

The injectIframWithSrc function allows embedding videos from youtube or vimeo. Since the html is sanitized it is not possible to use iframes with src inside. The function takes two parameters `containerClass` which is used to pick the element where the iframe is injected (class names are mangled during sanitizing) and `src` which should be a href to the youtube video (use youtube-nocookie.com) or vimeo video. The method only allows whitelisted sources.

## Examples

You can find examples in the github organization [MicroContent](https://github.com/MicroContent)

## Get Started

To get started you only need a text editor.
To test what you develop I suggest using a simple static content serving utility like [http-server](https://www.npmjs.com/package/http-server) and the [MicroContentTestHost](https://github.com/bgoeschi/MicroContentTestHost).
