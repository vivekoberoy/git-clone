# OpenTok JS SDK Getting Started Sample App


This sample client app shows how to accomplish the following using the OpenTok Javascript Client Library:

* Initialize and connect to an OpenTok session, publish to a stream and subscribe to a stream
* The ability to record the session, stop the recording, and view the recording
* Text chat for the participants


The code for this sample is found the following git branches:

basics -- This branch shows you how to connect to the OpenTok session, publish a stream, subscribe to a stream, and mute audio.

archiving -- This branch shows you how to record the session.

signaling -- This branch shows you how to use the OpenTok signaling API to implement text chat.

You will also need to clone the OpenTok Getting Started repo and run its code on a PHP-enabled web server. See the next section for more information.

# Setting up the test web service

The Getting Started repo includes code for setting up a web service that handles the following API calls:

* "/session" -- The JS client calls this endpoint to get an OpenTok session ID, token, and API key.
* "/start" -- The JS client calls this endpoint to start recording the OpenTok session to an archive.
* "/stop" -- The JS client calls this endpoint to stop recording the archive.
* "/view" -- The JS client loads this endpoint in a web browser to display the archive recording.

Clone the [Getting Started repo](https://github.com/opentok/GettingStarted.git) and follow the instructions listed under the `Installation` section of the README file. Once that is done, proceed to the next section.


# Configuring the application

1. Clone this repository.This repository as mentioned earlier has three branches. 
2. When you do a git clone, you get the `basics` branch. To clone other branches you need to execute the     following commands -

		git fetch
		git checkout signaling

		git fetch
		git checkout archiving

3. Now, switch to `basics` branch using `git checkout basics` command. Navigate to `web/js` directory and rename the sampleconfig.js file to config.js
4. Edit the file and define the variable SAMPLE_SERVER_BASE_URL. For our test web service, this value should be set to http://localhost:8080/

# Getting an OpenTok session ID, token, and API key

An OpenTok session connects different clients letting them share audio-video streams and send messages. Clients in the same session can include iOS, Android, and web browsers.

**Session ID** -- Each client that connects to the session needs the session ID, which identifies the session. Think of a session as a room, in which clients meet. Depending on the requirements of your application, you will either reuse the same session (and session ID) repeatedly or generate new session IDs for new groups of clients.

*Important*: This demo application assumes that only two clients -- the local Web client and another client -- will connect in the same OpenTok session. For test purposes, you can reuse the same session ID each time two clients connect. However, in a production application, your server-side code must create a unique session ID for each pair of clients. In other applications, you may want to connect many clients in one OpenTok session (for instance, a meeting room) and connect others in another session (another meeting room). For examples of apps that connect users in different ways, see the OpenTok ScheduleKit, Presence Kit, and Link Kit [Starter Kit apps](https://tokbox.com/opentok/starter-kits/).

Since this app uses the OpenTok archiving feature to record the session, the session must be set to use the routed media mode, indicating that it will use the OpenTok Media Router. The OpenTok Media Router provides other advanced features (see [The OpenTok Media Router and media modes](https://tokbox.com/opentok/tutorials/create-session/#media-mode)). If your application does not require the features provided by the OpenTok Media Router, you can set the media mode to relayed.

Token -- The client also needs a token, which grants them access to the session. Each client is issued a unique token when they connect to the session. Since the user publishes an audio-video stream to the session, the token generated must include the publish role (the default). For more information about tokens, see the OpenTok [Token creation overview](https://tokbox.com/opentok/tutorials/create-token/).

API key -- The API key identifies your OpenTok developer account.

Upon starting up, the application calls the `getApiAndToken()` method defined in the app.js file. This method makes an XHR(or Ajax request) to the "/session" endpoint of the web service. The web service returns an HTTP response that includes the session ID, the token, and API key formatted as JSON data:


	{
 		 "sessionId": "2_MX40NDQ0MzEyMn5-fn4",
  		 "apiKey": "12345",
  		 "token": "T1==cGFydG5lcl9pZD00jg="
	}
	
	
# Connecting to the session

Upon obtaining the session ID, token, and API, the getApiAndToken() method calls the initializeSession()  method. This method initializes an OTSession object and connects to the OpenTok session:


The initSession call is shown below. It takes 2 parameters - the OpenTok apiKey and the sessionId. It initializes and returns an  OpenTok session object.


    //Initialize Session Object
    var session = OT.initSession(apiKey, sessionId);


The session.connect() method connects the Javascript client application to the OpenTok session. You must connect before sending or receiving audio-video streams in the session (or before interacting with the session in any way). The connect() method takes 2 parameters - a token and a completion handler function:

    //Connect to the Session
    session.connect(token, function(error) {

        //If the connection is successful, initialize a publisher and publish to the session
        if (!error) {
            var publisher = OT.initPublisher('publisher', {
                insertMode: 'append',
                width: '100%',
                height: '100%'
            });

            session.publish(publisher);

        } else {

            console.log("There was an error connecting to the session", error.code, error.message);
        }

    });
    
    
An error object is passed into the completion handler of the connect event when the client fails to connect to the OpenTok session. Otherwise, no error object is passed in, indicating that the client connected successfully to the session. 
    
The session object dispatches streamCreated event when a new stream is created in the session. It dispatches a sessionDisconnected event when your client disconnects from the session. The application defines handlers to listen to these two events.


# Publishing an audio video stream to the session

Upon successfully connecting to the OpenTok session (see the previous section), the application initializes an OpenTok Publisher object and publishes to the session. This is done inside the completion handler for the connect() method since you should only publish to the session once you are connected to it. 


The publisher object is initialized as shown below . It takes 3 optional parameters (two of which are shown in the code below). The first parameter is the DOM element that the publisher video replaces. The second parameter specifies the properties of the publisher. The third parameter (not shown) specifies the completion Handler.

            var publisher = OT.initPublisher('publisher', {
                insertMode: 'append',
                width: '100%',
                height: '100%'
            });
            
            
Once the publisher object is initialized, we publish to the session using the session.publish method shown below -            

            session.publish(publisher);



# Subscribing to another client's audio-video stream

The Session object dispatches a streamCreated event when a new stream (other than your own) is created in a session. A stream is created when a client publishes to the session. The streamCreated event is also dispatched for each existing stream in the session when you first connect. This event is defined by the StreamEvent, which has a stream property, representing stream that was created. The application listens to the streamCreated event and subscribes to all streams created in the session using the session.subscribe() method as shown below:
  		
  		//Subscribe to a newly created stream
    	
    	session.on('streamCreated', function(event) {
        	session.subscribe(event.stream, 'subscriber', {
            	insertMode: 'append',
            	width: '100%',
            	height: '100%'
        	});
    	});        
    	
    	
The subscribe method takes parameters - the first parameter is the stream object to which we are subscribing, the second parameter(optional) is the DOM element that the subscriber video replaces. The third parameter(optional) represents the properties that customize the appearence of the subscriber view in the HTML page. The fourth parameter (optional) represents the completionHandler function that is called when the subscribe() method completes successfully or fails. This completes a discussion of the basics - initialize and create a session, publish an audio-video stream and subscribe to another client's audio-video stream. Let's move on to archiving.

# Recording the session to an archive

Important -- To view the code for this functionality, switch to the *archiving* branch of this git repository.    

The OpenTok archiving API lets you record audio-video streams in a session to MP4 files. You use server-side code to start and stop archive recordings. In the config.js file, you set the following constant to the base URL of the web service the app calls to start archive recording, stop recording, and play back the recorded video:	
    	
    	var SAMPLE_SERVER_BASE_URL = 'URL_OF_YOUR_OPENTOK_WEB_SERVER';
    	

If you do not set this string, the Start Recording, Stop Recording, and View Archive buttons will not be available in the app.

The archiving application uses the same code available in the basics branch to initialize an OpenTok session, connect to the session, publish a stream and subscribe to stream in the session. If you have not already gotten familiar with the code in that branch, consider doing so before continuing.


To start recording the video stream, the user clicks the Start Recording button which invokes the startArchive() method in app.js. This method in turn sends an XHR(or Ajax) request to server. The sessionId of the session that needs to be recorded is passed as a URL parameter to the server. As soon as the startArchive() method is called, the Start Recording button is hidden and the Stop Recording button is displayed.

	function startArchive() {
	
		$.post(SAMPLE_SERVER_BASE_URL + "/start/" + sessionId);
		$("#start").hide();
    	$("#stop").show();
	}
    	
    
To stop the recording, the user clicks on the Stop Recording button, which invokes the stopArchive() method. This method is similar to the startArchive() method in that it sends an Ajax request to the server to stop the archive. The only difference is that this method passes the archiveID of the archive that needs to be stopped as a URL parameter instead of the sessionId. The Stop Recording button is hidden and the View Archive button is enabled.
    	
	function stopArchive() {
	
    	$.post(SAMPLE_SERVER_BASE_URL + "/stop/" + archiveID);
    	$("#stop").hide();
    	$("#start").show();
    	$("#view").prop('disabled', false);
    	
	}    	


To download the archive that has just been recorded, the user clicks on View Archive button which invokes the viewArchive() method. This method is similar to the startArchive() and stopArchive() methods in that it sends an Ajax request to the server. The server code has the logic to check if the archive is available for download or not. If it is available, the application is redirected to the archive page. If not, a new page is loaded which continuously checks whether the archive is available for download or not and loads it when it is available.


# Using the signaling API to implement text chat

Important -- To view the code for this functionality, switch to the *signaling* branch of this git repository. 
	
Text chat is implemented using the OpenTok Signaling API. A signal is sent using the signal() method of the session object. To receive a signal a client needs to listen to the signal event dispatched by the session object. 

In our application, when the user enters text in the input text field, the form.addEventListener method is called:

	form.addEventListener('submit', function(event) {
    event.preventDefault();

    	session.signal({
            	type: 'chat',
            	data: msgTxt.value
        	}, function(error) {
        	if (!error) {
            	msgTxt.value = '';
        	}
    	});
	});


This method calls the session.signal method which sends a signal to all clients connected to the OpenTok session.If you need to send a signal to a specific client in the session, then you need to use the `to` property and specify the `connection` object that corresponds to the client connected to the session that you want to signal. Each signal is defined by a `type` property identifying the type of message (in this case "chat") and a `data` property containing the message. The text entered is sent in the data property of the signal method.

When another client connected to the session (in this app, there is only one) sends a message, the session.on event handler is invoked.

	session.on('signal:chat', function(event) {
        var msg = document.createElement('p');
        msg.innerHTML = event.data;
        msg.className = event.from.connectionId === session.connection.connectionId ? 'mine' : 'theirs';
        msgHistory.appendChild(msg);
        msg.scrollIntoView();
    });
    
    
This method checks to see if the signal was sent by the local web client or by the other client connected to the session:
        
        
        event.from.connectionId === session.connection.connectionId ? 'mine' : 'theirs';
        
The session object represents your OpenTok session.It has a `connection` property with a `connectionId` property. The event object represents the event associated with this signal. It has a `from` property (which is a connection object) with a `connectionId` property. This represents the connectionId of the client sending the signal. If these match, the signal was sent by the local web client.

The data associated with the event is then appended as a child of the `history` div in the HTML document.

This app uses the OpenTok signaling API to implement text chat. However, you can use the signaling API to send messages to other clients (individually or collectively) connected to the session.

# Other resources

See the following:

* [API reference](https://tokbox.com/opentok/libraries/client/js/) -- Provides details on the API's available in the OpenTok Javascript client library
* [Tutorials](https://tokbox.com/opentok/tutorials/) -- Includes conceptual information and code samples for all OpenTok features      

	

