Plataphorma
===========

Meteor pet project created to teach my students the following Meteor functionality: 

* Collection's publish/subscribe 
* Deps.autorun 
* Meteor.methods/call 
* Integration of non-Meteor code in compatibility folder (HTML5 games Alien Invasion and Froot Wars)
* Usage of allow to control client access to collections

![ScreenShot](/screenshot.png)


Plataphorma offers the possibility to run 2 different HTML5 games: Alien Invasion and Froot Wars. 

On the right side of the screen the best players of each game are shown, updated in real time each time a signed in player finishes a game. If no game is selected, the best players overall are shown.

On the left side of the screen a chatroom for the current game is available. Only signed in users can post messages. If no game is selected a general chatroom is shown.

The original code of the two HTML5 games integrated in this project is available here:
* Alien Invasion: https://github.com/cykod/AlienInvasion
* Froot Wars: http://www.wrox.com/WileyCDA/WroxTitle/Professional-HTML5-Mobile-Game-Development.productCd-1118301323,descCd-DOWNLOAD.html

Bootstrap style (file bootstrap.min.css) provided by http://bootswatch.com


Running the project
-------------------

A live version of this code is running here: http://plataphorma.meteor.com

To run the project locally, clone the repo and run ```meteor``` inside it. You can see in .meteor/packages that this Meteor project uses these packages:
* ```meteor remove autopublish```
* ```meteor remove insecure```
* ```meteor add bootstrap```
* ```meteor add accounts-ui```
* ```meteor add accounts-password```

1-)CLICK EN BOTON DE JUEGO
-----EN SERVIDOR
Template.choose_game.events = {
    'click #AlienInvasion': function () {
	$('#gamecontainer').hide();
	$('#container').show();
	var game = Games.findOne({name:"AlienInvasion"});
	Session.set("current_game", game._id);
    },
    'click #FrootWars': function () {
	$('#container').hide();
	$('#gamecontainer').show();
	var game = Games.findOne({name:"FrootWars"});
	Session.set("current_game", game._id);
    },
    'click #none': function () {
	$('#container').hide();
	$('#gamecontainer').hide();
	Session.set("current_game", "none");
    }

    ----EN SERVIDOR
    */
Meteor.startup(function() {
    // At startup, fill collection of games if it's empty
    if (Games.find().count() == 0) {
	Games.insert({name: "FrootWars"});
	Games.insert({name: "AlienInvasion"});
    };

>>>>en el cliente
Se utiliza un template creado para cuando hacemos click en el boton referencia del juego determinado se esconde el ('#gamecontainer ') que es un objeto q coge las puntuaciones de todos los juegos y muestra ('#container') que es el del juego determinado.Despues crea una variable para buskar en tu conjunto de juegos y lo pone como siguiente juego a ejecutar.
>>>>en el servidor 
Se introducen ambos juegos en la coleccion para tener acceso a ellos en GAMES.


2-)SE ESCRIBE MENSAJE CHAT SIN ESTAR AUTENTIFICADO
<template name="chat">
  {{#if none }}
  <h3> Chat Plataphorma </h3>
  {{else}}
  <h3> Chat {{gameName}} </h3>
  {{/if}}
  
  {{> input    }}
  {{> messages }}
</template>


<template name="welcome">
  <h1>Plataphorma</h1>
  <p>  Let's play together </p>
</template>


<template name="input">
  <p>Message: <input type="text" id="message"></p>

  
  <div> class="alert fade in" id="login-error" style="display:none;">
    <button type="button" class="close">x</button>
    You must be signed in to post messages.
  </div>

</template>


>>>>>>Lo que ocurre es que al utilizar el template chat utilizamos otro template que es input(otro template ) el cual tiene un identificado de error (login-error) expuesto en el siguiente codigo:
Template.input.events = {
    'keydown input#message' : function (event) {
	if (event.which == 13) { 
	    if (Meteor.userId()){
		var user_id = Meteor.user()._id;	    
		var message = $('#message');
		if (message.value != '') {
		    Messages.insert({
			user_id: user_id,
			message: message.val(),
			time: Date.now(),
			game_id: Session.get("current_game")
		    });
		    message.val('')
		}
	    }
	    else {
		$("#login-error").show();
	    }
	}
    } el cual nos dice que si estas registrado [[[[if (Meteor.userId())]]] te puedas ir a [[[$('#message');]]] y sino te lleve hasta 
    $("#login-error").show(); el cual hara que nos salga un mensaje de error, como ocurre cuando ejecutamos.


3-)SE ESCRIBE MENSAJE CHAT  ESTANDO AUTENTIFICADO
Lo he explicado antes....el cual nos dice que si estas registrado [[[[if (Meteor.userId())]]] te puedas ir a [[[$('#message');]]:::
ejecutandose el siguiente codigo y permitiendo enviar mensajes :

<template name="input">
  <p>Message: <input type="text" id="message"></p>


4-)SE TERMINA LA PARTIDA CON PUNTUACION ALTA SIN ESTAR AUTENTIFICADO

Meteor.methods({
    matchFinish: function (game, points) {
	// Don't insert in the Matches collection a match if the user
	// has not signed in
	if (this.userId)
	    Matches.insert ({user_id: this.userId, 
			     time_end: Date.now(),
			     points: points,
			     game_id: game
			    });
    }
});
Si el usuario esta identificado inserta el id de user , la fecha , los puntos y el juego en coleccion MATCHES , sino, no lo incluye en esta colección:

5-)SE TERMINA LA PARTIDA CON PUNTUACION ALTA ESTANDO AUTENTIFICADO
Meteor.methods({
    matchFinish: function (game, points) {
	// Don't insert in the Matches collection a match if the user
	// has not signed in
	if (this.userId)
	    Matches.insert ({user_id: this.userId, 
			     time_end: Date.now(),
			     points: points,
			     game_id: game
			    });
    }
});
Si el usuario esta identificado inserta el id de user , la fecha , los puntos y el juego en coleccion MATCHES , sino, no lo incluye en esta colección.
Meteor.publish("matches_current_game", function(current_game){
    var game_criteria;

    if (current_game == "none")
	game_criteria = {};
    else 
	game_criteria = {game_id: current_game};

    // publish every field of the latest 5 matches sorted by points in
    // descending order
    return Matches.find(game_criteria, 
			{limit:5, sort: {points:-1}});
    
}); Despues, utilizando publish devolverá, en función del criterio (las 5 mejores puntuaciones), el id del usuario en funcion del juego o si estamos en NONE el criterio no estará referenciado en función del juego sino solo en funcion del valor de la puntualcion

6-)QUE SALE EN CONSOLA CUANDO TE AUTENTIFICAS (SIGN IN)?
--
[12:25:11.731] "current user: RTvs7XwLBdWh4mowe"

// Aux function used in Messages.allow
function adminUser(userId) {
    var adminUser = Meteor.users.findOne({username: "admin"});
    return (userId && adminUser && userId === adminUser._id);
}

// Permissions for client trying to access Messages collection
Messages.allow({
    insert: function(userId, doc){
	// Only authenticated users can insert messages
	return Meteor.userId();
    },
   Lo insertamos en nuestra base de datos de Usuarios y nos devuelve Meteor.userId();

7-)QUE SALE EN CONSOLA CUANDO TE SALES (SIGN OUT)?
null

 remove: function (userId, docs){
        // Only admin user can remove chat messages.
        // No UI for this, you can test from console
	return adminUser(userId);
    }
});
lo eliminamos de nuestra base de datos y nos va a devolver null.
