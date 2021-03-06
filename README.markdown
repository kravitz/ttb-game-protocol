Protocol description
====================
Types of server response
------------------------
This section describes the responses of server. In general each
response have a status, which resides in _"status"_ field. Only response
with status "ok" means that last query have been processed clearly, any
status, that differs from this, stands for error and his "message" field
describes exactly what bad thing have happened.

#### OK ####
Every successful query will be followed by this response, it have a status field
with value "ok" and all necessary data after it.
    {
        "status": "ok",
        ...
    }
#### Bad Request ####
This is an error response, and have a meaning of syntax error in JSON query parsing.
    {
        "status": "badRequest",
        "message": "<description>",
    }
#### Bad Command ####
This error appears when command is incorrect, unknown, don't have enough fields for command or
have extra fields, that don't required by command. Also may be used for error reporting when other
error statuses can't be used.
    {
        "status": "badCommand",
        "message": "<description>",
    }
#### Internal Error ####
This error returned if something unhandled happened inside your engine.
    {
        "status": "internalError",
        "message": "<description>",
    }
#### Bad Password ####
Used when a bad password supplied to a register command, when username already exists.
    {
        "status": "badPassword",
        "message": "<description>",
    }
#### Bad Sid ####
Thats it, invalid sid supplied to any command, that requires a sid.
    {
        "status": "badSid",
        "message": "<description>",
    }
#### Already Exists ####
Stands for an error, when a name of game already reserved.
    {
        "status": "alreadyExists",
        "message": "<description>"
    }
#### Already in Game ####
This is an error, when some command supplied, that couldn't be executed, when user have status "in game".
    {
        "status": "alreadyInGame",
        "message": "<description>",
    }
#### Not in Game ####
Same as above, but with user status "not in game"
    {
        "status": "notInGame",
        "message": "<description>",
    }

Basic commands
--------------
This section contains a list of commands, with short descriptions ( I hope it'll happen soon).

Note: The commands with a star(\*) in a beginning of name should work only in test mode, and
return _Bad Command_ in a normal mode.

## Registration ##

### Request ######
    {
        "cmd": "register",
        "username": "<username>",
        "password": "<password>"
    }
### Answer ######
Good:
    {
        "status": "ok",
        "sid": "<sid>"
    }
Errors: _Bad Password_, _Bad Username_

## Logout ##

### Request ######
    {
        "cmd": "unregister",
        "sid": "<sid>"
    }
### Answer ######
Good: _OK_

Errors: _Bad Sid_

## Sending message ##
### Request ######
    {
        "cmd": "sendMessage",
        "sid": "<sid>",
        "gameName": "<name of the game>"
        "message": "<message>"
    }
### Answer ######
Good: _OK_

Errors: _Bad Game_, _Bad Message_, _Bad Sid_

## Getting chat history ##
### Request ######
    {
        "cmd": "getChatHistory",
        "sid": "<sid>",
        "gameName": "<name of the game>"
    }
### Answer ######
Good:
    {
        "status": "ok",
        "chat":  [ { "username": "<username>",
                     "message": "<message>",
                     "time": "<UTC>" }, ... ]
    }

Errors: _Bad Sid_, _Bad Game_

## \*Database cleanup ##
### Request ######
    {
        "cmd": "clear"
    }
### Answer ######
Good: _OK_

Errors: None
## Getting a list of games ##

### Request ######
    {
        "cmd": "getGamesList",
        "sid": "<sid>"
    }
### Answer ######
Good:
    {
        "status": "ok",
        "games": [
            {
                "gameName": "<name of the game>",
                "mapName": "<name of map>",
                "factionName": "<name of faction>",
                "gameStatus": "<status of game>",
                "playersCount": <number>,
                "connectedPlayersCount": <number>,
                "totalCost": <number>
            }, ...  ]
    }

Errors: _Bad Sid_
## Getting a list of players ##

### Request ######
    {
        "cmd": "getPlayersList",
        "sid": "<sid>"
    }
### Answer ######
Good:
    {
        "status": "ok",
        "players": [ { "username": "<username>" }, ...  ]
    }

Errors: _Bad Sid_
## Getting a list of players for the game##

### Request ######
    {
        "cmd": "getPlayersListForGame",
        "sid": "<sid>",
        "gameName": "<name of the game>"
    }
### Answer ######
Good:
    {
        "status": "ok",
        "players": [ { "username": "<username>" }, ...  ]
    }

Errors: _Bad Sid_, _Bad Game_

## Setting player status ##

### Request ######
    {
        "cmd": "setPlayerStatus",
        "sid": "<sid>",
        "status": "<status>"
    }
### Answer ######
Good: _OK_

## Creation of a game ##

### Request ######
    {
        "cmd": "createGame",
        "sid": "<sid>",
        "gameName": "<name of the game>",
        "maxPlayers": <number>,
        "mapName": "<name of map>",
        "factionName": "<name of faction>",
        "totalCost": <number>
    }
### Answer ######
Good: _OK_

## Join a game ##

### Request ######
    {
        "cmd": "joinGame",
        "sid": "<sid>",
        "gameName": "<name of the game>"
    }
### Answer ######
Good: _OK_

## Leaving a game ##

### Request ######
    {
        "cmd": "leaveGame",
        "sid": "<sid>",
        "gameName": "<name of the game>"
    }
### Answer ######
Good: _OK_

Tests format specification
==========================
Each test is a file with .tst extension. The name of a test
must contain only Latin letters, symbol "\_", and digits. In
other words, it must match following regexp */[a-zA-Z0-9\_]+\\.tst/*

The format of test is a set of JSON requests, separated by
newlines and/or white spaces. Only double quotes are allowed. An answer
to a test must have exactly the same name, but with .ans extension.
The format of an answer is absolutely the same as the format of a test.

Consider the following example:

_goodUsername_\__1.tst_
    {
        "cmd": "register",
        "username": "Vasya_Pupkin",
        "password": "123456"
    }
_goodUsername_\__1.ans_
    {
        "sid": "Vasya_Pupkin123456",
        "status": "ok"
    }

The previous example also shows another important aspect - the sid-generator.
While running in testing mode, it must return _sid_, that is generated by simple rule:
    sid = username + password

Also there are some tricks to make tests fully determined. First of all server must use random
number generator, based on Mersenne Twister algorithm. Tester must provide 0 as a seed to a random number
generator before each test. Any time specific test must use "2000-01-01 00:00:00" value.

Rules  of the game
==================
Its a turn-based strategy game, the action takes place on a rectangular field, which is represented
by a sequence of strings, consist only of a 3 types of characters: "x" - obstacle, "." - free cell,
"[1-9]"(digits from 1 to 9) - players deployment spots. For instance the map in JSON query can be drawn
like this
    {
        ...
        "map": ["......222",
                "....x.222",
                "...xxx...",
                "111.x....",
                "111......"]
        ...
    }
Before the game starts players must put their Units on a deployment spots. Each Unit have a cost, and sum of
all Units costs must be equal of lower then totalCost declared for map. Each Unit have a list of characteristics:

 * HP - health
 * MP - count of squares, unit can cross in one turn
 * Defense
 * Attack
 * Range
 * Damage
 * Cost

Extensions to protocol
----------------------

### Upload a map ###
#### Request ####
    {
        "cmd": "uploadMap",
        "sid": "<sid>",
        "name": "<name>",
        "terrain": "[<map strings>,...]",
    }
#### Answer ####
Good: _OK_

Error: _Bad Map_, _Bad Sid_, _Already Exists_

### Delete a map ###
#### Request ####
    {
        "cmd": "deleteMap",
        "sid": "<sid>",
        "name": "<name>"
    }
#### Answer ####
Good: _OK_

### Upload Faction ###
#### Request ####
    {
        "cmd": "uploadFaction",
        "sid": "<sid>",
        "name": "<name of faction",
        "units": [
            {
                "name": "<name of unit>",
                "hp": <number>,
                "mp": <number>,
                "defense": <number>,
                "attack": <number>,
                "range": <number>,
                "damage": <number>,
                "cost": <number>
            }, ... ]
    }
#### Answer ####
Good: _OK_

### Delete Faction ###
#### Request ####
    {
        "cmd": "deleteFaction",
        "sid": "<sid>",
        "name": "<name of faction>"
    }
#### Answer ####
Good: _OK_

### Upload Army ###
#### Request ####
    {
        "cmd": "uploadArmy",
        "sid": "<sid>",
        "name": "<name of army>",
        "factionName": "<name of faction>",
        "units": [
            {
                "name": "<unit name>",
                "count": <number>
            },...]
    }
#### Answer ####
Good: _OK_

### Delete Army ###
#### Request ####
    {
        "cmd": "deleteArmy",
        "sid": "<sid>",
        "name": "<name of army>"
    }
#### Answer ####
Good: _OK_

### Choose army ###
#### Request ####
    {
        "cmd": "chooseArmy",
        "sid": "<sid>",
        "name": "<name of Army>"
    }
#### Answer ####
Good: _OK_

### Get Map ###
#### Request ####
    {
        "cmd": "getMap",
        "sid": "<sid>,
        "name": "<name of map>"
    }
#### Answer ####
Good:
    {
        "status": "ok",
        "map": ["<map strings>",...],
    }

### Get Map List ###
#### Request ####
    {
        "cmd": "getMapList",
        "sid": "<sid>"
    }
#### Answer ####
Good:
    {
        "status": "ok",
        "maps": [
            {
                "name": "<name of map>",
                "players": "<number>",
                "width": "<number>",
                "height": "<number>"
            }
        ]
    }

### Get Faction ###
#### Request ####
    {
        "cmd": "getFaction",
        "sid": "<sid>",
        "name": "<name of faction>"
    }
#### Answer ####
Good:
    {
        "status": "ok",
        "units": [
            {
                "name": "<name of unit>",
                "hp": <number>,
                "mp": <number>,
                "defense": <number>,
                "attack": <number>,
                "range": <number>,
                "damage": <number>,
                "cost": <number>
            }, ...
        ]
    }

### Get Army ###
#### Request ####
    {
        "cmd": "getArmy",
        "sid": "<sid>",
        "name": "<name of army>"
    }
#### Answer ####
Good:
    {
        "status": "ok",
        "units": [
            {
                "name": "<name of unit>",
                "count": <number>
            },...
        ]
    }
### Get Game State ###
#### Request ####
    {
        "cmd": "getGameState",
        "sid": "<sid>",
        "name": "<name of game>"
    }
#### Answer ####
Good:
    {
        "status": "ok",
        "players": [ 
            { 
            	"username": "<username>" 
                "units": [
                    { 
                    	"name": "<name of unit>",
                        "hp": <number>,
                        "mp": <number>,
                		"defense": <number>,
                		"attack": <number>,
                		"range": <number>,
                		"damage": <number>,
                		"cost": <number>
                    }, ... 
                ]
                "isReady": <boolean>
            }, ...  
        ]
        "turn": <number>
    }
### Move ###
#### Request ####
    {
        "cmd": "move",
        "sid": "<sid>",
        "turn": <number>
        "units": [
            { 
                "name": "<name of unit>",
                "posX": <number>,
                "posY": <number>,
                "destX": <number>,
                "destY": <number>,
                "attackX": <number>,
                "attackY": <number>
            }, ... 
        ]               
    }
#### Answer ####
Good: _OK_
