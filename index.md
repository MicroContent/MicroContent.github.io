# MicroContent for Social MicroLearning
## Concept and Idea

MicroContent is a type of digital learning material that is
* small (size, time to consume)
* interactive (requires the user to do something and gives feedback)

Clearly many types of content fullfil these criteria and it is impossible to implement and include each potentially desired content type into a single system.

Therefore, for our Social MicroLearning system we use the concept of external or third-party content types.
A content type provider has to provide two things:
* a content viewer that is able to display data appropriatly
* an editor that is able to edit the data or create a new instance

Those two elements have to be permanently hosted as static html content. CORS has to be enabled. We recommend to use githubs raw-view. This also has the advantage that you can directly adress a specific version of the viewer/editor. [Example](https://raw.githubusercontent.com/MicroContent/BinaryNumberContentViewer/2.0.0/index.html).

## Seamless IFrames
Since Google deprecated their [Caja](https://developers.google.com/caja) project MicroContent now utilizes the [Seamless](https://github.com/travist/seamless.js/) library by Travis Tidwell to ensure efficient communication between the MicroContent units and the Social MicroLearning system. For this to work both the content editor and the content viewer need to be able to access the [seamless child script](https://raw.githubusercontent.com/travist/seamless.js/master/build/seamless.child.min.js). Either included in the respective MicroContent repositories or linked directly to travisst's project. 

To enable the communication between the seamless IFrame and the parent window, three functions need to be implemented. See travist' documentation or the [AudioContentEditor](https://github.com/MicroContent/AudioContentEditor) for an example implementation.
* connection establishment
    ```javascript
    var parent = window.seamless.connect({
        url: window.location.url,
        allowStyleInjection: true,
    });
    ```
* receiving messages 
    ```javascript
    parent.receive(function (data) {
       switch (data.type) {
           case 'setContent':
               // parent for varying type of input sends data as special object
               // ; data.main[0] is the array of arrays with your message inside
               window.data = data.main[0];
               setContent();
               break;
       }
    });
    ```
* sending messages
    ```javascript
    parent.send({
        type: 'updateLayout'
    });
    ```


## Content Editor

A content editor should provide means to enter the data for a specific instance of your learning content type. The [AudioContentEditor](https://github.com/MicroContent/AudioContentEditor) for example allows a user to provide input for a single or multiple choice question and attach an audio file to it. 

An editor has three types of messages which are exchanged with its hosting environment.
* `setContent`
* `toViewer`
* `updateLayout`

The setContent message is received from the parent and contains an object holding  data for the editor (in the case a user is editing existing content).

The toViewer message is sent to the parent and should contian a data object representing the current state of user input. The host environment will use this object to display in the viewer and store the data for this instance.

The updateLayout message is sent to the parent to inform the host environment that the content of the editor IFrame needs to be updated.

MicroContent is often shared through social media and other communication channels using link preview. The following properties of your data will be picked up for link previews by the host system as a convention:
* `title`
* `description`
* `imageUrl`

If the MicroContent requires media (audio, video, images), these can be sent to the hosting environment within the `toViewer` message via a `mediaUrlArray` attribute. To be processed correctly the media should be sent either as a already publicly available link or as files encoded by `FileReader().readAsDataUrl(file)`.

## Content Viewer 
A content viewer shall display data and allow users to interact with it. The host environment provides a submit solution button to users and content viewers should be designed accordingly. The following types of messages are supported for communication with the host environment:
* host environment to viewer
    * `setContent`
    * `submit`
    * `seamless_styles`
* viewer to host environment
    * `updateLayout`
    * `sendXapiStatement`
    * `getXapiStatments`


### setContent (important)
The getData function returns a data object as stored with the editor. The data object essentially defines an instance of the MicroContent type. E.g. a multiple choice card might have a title (string), a question (string) and an answer array. You should handle the processing of the `setContent` message before/when you initialize your view.

### submit (important)
The submit message informs the viewer that a submit button in the host environment has been clicked, prompting the viewer for an action. Typically, you will want to display the solution and send an xAPI statement.

### seamless_styles (if needed)
The seamless_styles message is native to the Seamless library and is used to inject CSS style into an IFrame. It is mostly handled by the seamless IFrame by default; however, in some cases it might require the triggering of an `updateLayout` message to refresh the view.

### updateLayout (very recommended)
This informs the hosting environment of changes in the viewer's UI. It is needed when the viewer updates CSS styles of its content dynamically.

### sendXapistatement (very recommended)
The sendXapiStatment message is for logging learning relevant user (inter)actions. It takes an xAPI statement as a parameter.
The xAPI or Experience API is a proposed standard to log, track and analyze learning activity data. Its heart is the a RESTful Endpoint to send and receive so called xAPI-Statements. You can find the full spec here. You can play around with the xapi-lab, an online statement generator to test what statements can look like. The essence of a statement are actor, verb and object (Who did what with/to what?). A MicroContent unit does not handle authentication, object identification and can be embedded in various contexts. Therefore the host environment will add actor, object.objectId and context.platform to the statement after you pass it to sendXapiStatement and before it is actually sent to an xAPI Endpoint. The host environment also takes care about where to send it to and how to authenticate the statement.

### getXapiStatements (if needed)
The getXapiStatements message can be used to request all interations associated with an instance of the MicroContent unit. They can be used e.g. to display statistics.

## Examples

You can find examples in the github organization [MicroContent](https://github.com/MicroContent).

Since the MicroContent project is in the process of migrating from Google Caja, some of the repositories are deprecated. MicroContent units that have already been successfully migrated are:
* [BinaryContentEditor](https://github.com/MicroContent/BinaryNumberContentEditor) and [BinaryContentViewer](https://github.com/MicroContent/BinaryNumberContentViewer)
* [AudioContentEditor](https://github.com/MicroContent/AudioContentEditor) and [AudioContentViewer](https://github.com/MicroContent/AudioContentViewer)


## Get Started

To get started you only need a text editor.
To make your first steps as easy as possible we provide a MicroContent development toolkit. It provides a static http server for local development, that already includes a MicroContentTestHost. You can find it [here](https://github.com/MicroContent/MicroContentToolkit-2). Just download and run it as described [here](https://github.com/MicroContent/MicroContentToolkit-2#start-and-run-the-servertoolkit) and you're good to go!
