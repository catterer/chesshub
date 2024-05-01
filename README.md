# **chesshub - design of a scalable and robust chess match system**

## Scope

To limit the scope of this design, I introduce certain limitations:

1. **Security.** I assume that authorization has already happened, and we can trust all the queries within the system. I also disregard any potential DoS attacks and the like. If required, access control can easily be implemented with a stateless gateway responsible for authorization and rate limits.
2. **Protocol & RPC.** I chose to use gRPC; it is by no means a hard requirement. I'm just more familiar with it, and schema makes it easier to describe API.
3. **DB.** The design described below requires the DB to support horizontal scalability, K8s integration, and subscriptions to stored object modifications. MongoDB satisfies those, so I decided to go with it. There are probably more suitable DBs, and for a real application, I would spend more time comparing them (one thing that comes to mind is stored procedure support, which would allow me to integrate the whole chess engine into the DB and get rid of ChessEngine ReplicaSet).

## Definitions

**Player** is an actor uniquely identified by PlayerID (string) who requests games and makes moves. Players can be humans or bots.

**Game** is a full game context uniquely identified by GameID (UUID). It is created once the Player requests a new game (that is, even before the Matching happens). Game context includes player or players participating in the game, the current state of the game (started, 

**Matching** means connecting two Players in the same Game.

**Bot** is a special kind of Player that implements some strategy and runs as a dedicated ReplicaSet.

## Player workflow

