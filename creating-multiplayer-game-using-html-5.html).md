# [Creating a Multiplayer game using HTML 5, Websockets and the Netty game server, Nadron](http://nerdronix.blogspot.com/2013/06/creating-multiplayer-game-using-html-5.html)

> http://nerdronix.blogspot.com/2013/06/creating-multiplayer-game-using-html-5.html

This example is based on the following [post](http://www.lostdecadegames.com/how-to-make-a-simple-html5-canvas-game/). The idea was to take a simple single-player client only game and make it a  multi-player game. For this purpose I have used a[ java game server](https://github.com/menacher/java-game-server/tree/netty4), [Nadron](https://github.com/menacher/java-game-server/tree/netty4/nadron)(formerly [jetserver](https://github.com/menacher/java-game-server/tree/master/jetserver)) which is a wrapper over [Netty4 ](http://netty.io/)with some built in semantics for games creation.
[![img](http://2.bp.blogspot.com/-6cIAz3dFN08/UcaQpLHqeLI/AAAAAAAABek/n1k8CoMTDhs/s640/GamePlay.png)](http://2.bp.blogspot.com/-6cIAz3dFN08/UcaQpLHqeLI/AAAAAAAABek/n1k8CoMTDhs/s1600/GamePlay.png)
The [lostdecade blog post](http://www.lostdecadegames.com/how-to-make-a-simple-html5-canvas-game/) provides a good write up on the single player game . Basically it has a monster, a hero and the hero catching the monster. To make it multi-player, a few changes were done to the original. The full java script file can be found [here](https://github.com/menacher/java-game-server/blob/netty4/nadclient-js/test/LDGame.html).
Javascript ClientThe main changes are as follows to convert to multi-player
1) The state of players, which is their x,y co-ordinates on the screen needs to be captured, for this a new variable `var players = []` is added. i.e. an array of all players.
2) In the single player version, the main game loop was invoking a render function to draw all the images on screen. But now, this function is invoked in response to server events. The following line of code does that.
`
function sessionCreationCallback(session){
​    session.onmessage = render;
​    ...`3) The update function and render function code was modified so that it will handle the array of players instead of a single player.
4) A config object is created to login to remote Nadron server, it basically contains the username, password and the game room to which this player is trying to login.
`// simple object used to store credentials for logging in to remote Nadron server
var config = {
​    user: "user",
​    pass: "pass",
​    connectionKey: "LDGameRoom"
};`5) For network communication with remote Nadron server, the following [script ](https://github.com/menacher/java-game-server/blob/netty4/nadclient-js/src/nad-0.1.js)was included in the LDGame html file.
So how does the data go from client to server and back? Lets do it line by line.
The following lines takes care of logging into remote server
`
// connect to remote Nadron server.
function startGame(){
​    nad.sessionFactory("ws://localhost:18090/nadsocket", config, sessionCreationCallback);
}
`Notice the callback function? When session is created it will be invoked and in turn set the render method as the message listener on the newly created session.
`
// This is the callback function that gets invoked
// when session is connected to Nadron server.
function sessionCreationCallback(session){
​    session.onmessage = render;
`
 So, now whenever the server sends data to browser, the render method will be invoked. The `sessionCreationCallback`function also sets up the 'gameloop' at the client side using javascript `setInterval`. This loop will continuously invoke the update function which will capture the current position of the hero and transmit it to Nadron using the following lines 
`
var message = {
​    "hero": hero
};
// send new position to gameroom
session.send(nad.NEvent(nad.NETWORK_MESSAGE, message));
`
That's it at the client side. To summarize, we use a list to store all players, render is now invoked via server events and update takes care of sending json objects back to the server. All cool! so now lets move on to the server side of things.
Nadron ServerAt the server side, game logic is written in the package [lostdecade](https://github.com/menacher/java-game-server/tree/netty4/example-games/src/main/java/io/nadron/example/lostdecade). The `Entity` and `LDState` are straight forward game beans, by which we can hold player information, no rocket science there. In fact, no rocket science anywhere!
The `LDEvent` is a bit more interesting, even though the class has only a getter and setter, notice that it inherits from the `DefaultEvent` class which in turn implements the `Event` interface. Nadron communicates using events, hence this hierarchy. So the whole purpose of this class is to just "wrap" the game state(`LDState` class in our example) for network communication, that's it.

This brings us to [`LDRoom` ](https://github.com/menacher/java-game-server/blob/netty4/example-games/src/main/java/io/nadron/example/lostdecade/LDRoom.java)which is actually the only logic bearing class in this whole game. For those un-initiated to Nadron/jetserver, GameRooms' are just a grouping mechanism for Player Sessions. Since its kind of a central part which is seen by all players, its also a good fit for game logic.
When a player logs in, the `onLogin` method gets invoked on the GameRoom, so we override it to do all the "connections".
[![img](http://1.bp.blogspot.com/-vOY69nEWcVM/UccUhxE1boI/AAAAAAAABe0/Waw3OEmWJeU/s640/SessionLogin.png)](http://1.bp.blogspot.com/-vOY69nEWcVM/UccUhxE1boI/AAAAAAAABe0/Waw3OEmWJeU/s1600/SessionLogin.png)
A player session which is just logged in has no event handlers attached, so any data that is sent by browser/client will not get handled. So our first order of business is to add a handler to handle data and events, the following lines of code show how to do that.
`
// Add a session handler to player session. So that it can receive events.
playerSession.addHandler(new DefaultSessionEventHandler(playerSession) {
​    @Override
​    protected void onDataIn(Event event) {
​        if (null != event.getSource()) {
​            // Pass the player session in the event context so that the
​            // game room knows which player session send the message.
​            event.setEventContext(new DefaultEventContext(playerSession, null));
​            // pass the event to the game room
​            playerSession.getGameRoom().send(event);
​        }
​    }
});
`
As is visible from inline comments, these 2 lines just pass the incoming data to the game room and ensure that a "context" is attached so that the game room knows who is speaking!

The other 2 actions of the onLogin method are 1) initialize game objects for this newly logged in player and 2) inform everyone else that a new kid is in town i.e do a broadcast. Code below
`
// Now add the hero to the set of entities managed by LDGameState
Entity hero = createHero(playerSession);
LDGameState state = (LDGameState) getStateManager().getState();
state.getEntities().add(hero);
// We do broadcast instead of send since all connected players need to
// know about the new player's arrival so that this hero can be drawn on
// their screens.
sendBroadcast(Events.networkEvent(new LDGameState(state.getEntities(),
​    state.getMonster(), hero)));`GameLogic The inner class `GameSessionHandler` in `LDRoom` has most of the game logic. This handler is a special handler which will only see events of type SESSION_MESSAGE, reason is that a game room is not interested in other events, at least for this simple game. Notice the below line in the `LDRoom` constructor? 
`
addHandler(new GameSessionHandler(this));
`That's where this handler starts listening on the game room. So data flow is something like this now
Incoming network data -> Player Session -> Session Event Handler(anonymous inner class in onLogin method) -> Game Room -> GameSessionHandler.
[![img](http://3.bp.blogspot.com/-3dMkM8RmgWU/UccYip_Ja0I/AAAAAAAABfE/z8AJRn9mA1k/s640/GameServerDataFlow.png)](http://3.bp.blogspot.com/-3dMkM8RmgWU/UccYip_Ja0I/AAAAAAAABfE/z8AJRn9mA1k/s1600/GameServerDataFlow.png)
The `onEvent` method of `GameSessionHandler` will receive events from the player session and invoke the update method which is just to check if the hero and monster are touching. If they are, then state is reset, i.e, monster is thrown randomly on the screen somewhere and the heroes are all put in the middle of the screen. If they are not touching the new x,y co-ordinates of this hero is transmitted to all other players so that they can update their screens.
Thats it game over!
...
ConfigurationOk I lied, So who configured this room? how was the client able to login with that specific room name? Where is the main class?
If you take a look at the [SpringConfig ](https://github.com/menacher/java-game-server/blob/netty4/example-games/src/main/java/io/nadron/example/SpringConfig.java)class you will notice 2 beans toward the end of the file, 1 is the `ldGame` and the other is the `ldGameRoom`
`
public @Bean(name = "LDGame")
Game ldGame()
{
​    return new SimpleGame(2, "LDGame");
}

public @Bean(name = "LDGameRoom")
GameRoom ldGameRoom()
{
​    GameRoomSessionBuilder sessionBuilder = new GameRoomSessionBuilder();
​    sessionBuilder.parentGame(ldGame()).gameRoomName("LDGameRoom")
​        .protocol(webSocketProtocol);
​    LDRoom room = new LDRoom(sessionBuilder);
​    return room;
}
`
These are the methods which are responsible for creating the room beans, the `lookupservice` bean will now register the room in its map with the name "LDGameRoom" the same name that's used by client when logging in, this is how the lookup happens. Note, that in a more sophisticated environment, you will be picking up these values from a DB, rather than use a map.
The main class is [`GameServer` ](https://github.com/menacher/java-game-server/blob/netty4/example-games/src/main/java/io/nadron/example/GameServer.java)which as you can notice is loading the spring context.
`
//Initialize spring context to load games and game rooms and other beans.
AbstractApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
// For the destroy method to work.
ctx.registerShutdownHook();
// Start the main game server
ServerManager serverManager = ctx.getBean(ServerManager.class);
serverManager.startServers();
`ExecutionFirst start the server, the [GameServer ](https://github.com/menacher/java-game-server/blob/netty4/example-games/src/main/java/io/nadron/example/GameServer.java)is the main class and you can start it from eclipse. Be sure to run it with the following vm configuration `-Dlog4j.configuration=GameServerLog4j.properties `otherwise it wont log properly.
[![img](http://1.bp.blogspot.com/-GFtK_761yww/Uccx-6TinmI/AAAAAAAABfU/R1QF8G0oL0A/s640/EclipseExecution.png)](http://1.bp.blogspot.com/-GFtK_761yww/Uccx-6TinmI/AAAAAAAABfU/R1QF8G0oL0A/s1600/EclipseExecution.png)
If you want to start from command prompt, then please include the[ nadron jar](https://github.com/menacher/java-game-server/tree/netty4/nadron/binaries) and the [dependent libraries](https://github.com/menacher/java-game-server/tree/netty4/nadron/lib) in the path.
The client can be started by right clicking on the [LDGame.html](https://github.com/menacher/java-game-server/blob/netty4/nadclient-js/test/LDGame.html) file and running it in a HTML5 compatible browser.
[![img](http://2.bp.blogspot.com/-l0GsFZJCo2U/UcaOosA2JfI/AAAAAAAABeM/x94ndJUIgEQ/s640/ExecuteClient.png)](http://2.bp.blogspot.com/-l0GsFZJCo2U/UcaOosA2JfI/AAAAAAAABeM/x94ndJUIgEQ/s1600/ExecuteClient.png)Below screenshot shows multiple players on the board. Reason is that I opened this game in multiple browser tabs and started playing simultaneously[![img](http://4.bp.blogspot.com/-6cIAz3dFN08/UcaQpLHqeLI/AAAAAAAABeo/vE4hO6ocfn8/s640/GamePlay.png)](http://4.bp.blogspot.com/-6cIAz3dFN08/UcaQpLHqeLI/AAAAAAAABeo/vE4hO6ocfn8/s1600/GamePlay.png)
AdvancedThis section mostly covers data transfer over network and related netty protocols. If you want to know the internals of Nadron and how it leverages Netty you might want to take a look at this [wiki ](https://github.com/menacher/java-game-server/wiki/Jetserver-internal-details-and-how-it-works.)page. The page is a little bit outdated since it was written for jetserver but the core concepts are very much the same.

The data is transferred from client to server using json using a simple `JSON.stringify(e).`At the server side, things are a bit more complicated since the Jackson parser which deserializes this json to Java object needs to know which class to deserialize to. For this reason the client needs to send an initial class name event to server, the following line does that.
`
// Send the java event class name for Jackson to work properly.
session.send(nad.CNameEvent("io.nadron.example.lostdecade.LDEvent"));
`
At the server side Jackson library is used to convert the incoming json to `LDEvent`. The [`TextWebsocketDecoder` ](https://github.com/menacher/java-game-server/blob/netty4/nadron/src/main/java/io/nadron/handlers/netty/TextWebsocketDecoder.java)class in the Netty pipeline does that. Here is the sample code.
`@Override
protected void decode(ChannelHandlerContext ctx, TextWebSocketFrame frame,
 MessageList out) throws Exception
{
   // Get the existing class from the context. If not available, then
   // default to DefaultEvent.class
   Attribute> attr = ctx.attr(eventClass);
   Class theClass = attr.get();
   boolean unknownClass = false;
   if (null == theClass)
   {
​      unknownClass = true;
​      theClass = DefaultEvent.class;
   }
   String json = frame.text();
   Event event = jackson.readValue(json, theClass);
   ...
`Netty pipeline structureFollowing [Websocket protocol code ](https://github.com/menacher/java-game-server/blob/netty4/nadron/src/main/java/io/nadron/protocols/impl/WebSocketProtocol.java)shows the encoders and decoders in the server pipeline which handle network communication from client. Unless you need your specific wire protocol you shouldn't have to touch this part.
`@Override
public void applyProtocol(PlayerSession playerSession)
​    LOG.trace("Going to apply {} on session: {}", getProtocolName(),
 playerSession);
​    ChannelPipeline pipeline = NettyUtils.getPipeLineOfConnection(playerSession);
​    pipeline.addLast("textWebsocketDecoder", textWebsocketDecoder);
​    pipeline.addLast("eventHandler", new DefaultToServerHandler(
 playerSession));
​    pipeline.addLast("textWebsocketEncoder", textWebsocketEncoder);
​    ...
`What if I want to change the port, other configuration, deploy my own protocol?Spring to the rescue, all the configuration is in the following[ resources folder](https://github.com/menacher/java-game-server/tree/netty4/nadron/src/main/resources/nadron/beans). You can override(actually hide) whichever bean you want by re-defining same bean in your SpringConfig file. The configuration like port numbers etc are in[ this file](https://github.com/menacher/java-game-server/blob/netty4/nadron/src/main/resources/nadron/props/nadron.properties), again overridable by re-declaring the properties bean.
How about multiple games and game rooms on the same server?The [SpringConfig ](https://github.com/menacher/java-game-server/blob/netty4/example-games/src/main/java/io/nadron/example/SpringConfig.java)file already hosts 2 games, with 3 different protocols and multiple game rooms. Take a look at the following beans in this file to see how its done.
`
public @Bean
Game zombieGame() {
...
public @Bean(name = "Zombie_Rooms")
List zombieRooms() {
...
public @Bean(name = "Zombie_Room_Websocket")
GameRoom zombieRoom2() {
...
public @Bean(name = "LDGame")
Game ldGame() {
...
public @Bean(name = "LDGameRoom")
GameRoom ldGameRoom() {
...

public @Bean(name = "lookupService")
LookupService lookupService() {`The `LookupService` then stores all the rooms in a map and client can decide which one to log in to. Depending on the clients choice, the network protocol will change, for e.g. if client chooses the room *Zombie_Room_1* then it has to use a binary protocol instead of web socket. But if it chooses *Zombie_Room_Websocket* then it can play the same game but with web socket protocol.
Troubleshooting

1. Client not connecting to server - are the port numbers on both sides correct? default port is 18090


2. Data is not received by server/client - check log files or put a break point in TextWebsocketDecoder/Encoder, LDRoom handlers etc to see where it is getting dropped.
3. At client side put break point in the render function to view incoming data.
4. For more information on the network packets, follow[ this tutorial](http://www.mkyong.com/webservices/jax-ws/how-to-trace-soap-message-in-eclipse-ide/) on how to setup eclipse to trace tcp traffic.


