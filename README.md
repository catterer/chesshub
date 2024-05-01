# **chesshub - design of a scalable and robust chess match system**

## Scope

To limit the scope of this design, I introduced certain limitations:

1. **Security.** I assume that authorization has already happened, and we can trust all the queries within the system. I also disregard any potential DoS attacks and the like. If required, access control can easily be implemented with a stateless gateway responsible for authorization and rate limits.
2. **Protocol & RPC.** I chose to use gRPC; it is by no means a hard requirement. I'm just more familiar with it, and schema makes it easier to describe API.
3. **DB.** The design described below requires the DB to support horizontal scalability, K8s integration, and subscriptions to stored object modifications. MongoDB satisfies those, so I decided to go with it. There are probably more suitable DBs, and for a real application, I would spend more time comparing them (one thing that comes to mind is stored procedure support, which would allow me to integrate the whole chess engine into the DB and get rid of ChessEngine ReplicaSet).

## Definitions

**Player** is an actor uniquely identified by PlayerID (string) who requests games and makes moves. Players can be humans or bots.

**Game** is a full game context uniquely identified by GameID (UUID). It is created once the Player requests a new game (that is, even before the Matching happens). The game context includes the player or players participating in the game, the current state of the game (like PLAYING and FINISHED), and the state of the board.

**Matching** means connecting two Players in the same Game.

**Bot** is a special kind of Player that implements some strategy and runs as a dedicated ReplicaSet.

All gRPC definitions should be available [here](proto/chesshub.proto).

## Player perspective

Here, I describe the progression of one game from a Player's perspective. Note that Player can have any number of games concurrently.

![pp](https://www.plantuml.com/plantuml/png/bL9TIyCm57tFhyZZ2kj4VHCsrirkAxgsj4KHX12wkotOvjOaS_hlJQnMaUBmzTppSSzDfjfmPGvrHNXfKD6quc-WI6D1KOg6IqDpK2yM8ks8-fDFv8gkkIdt0uz8D43HGjsaLC0jjkCrq60sFx-u9Et8oPrH9yz0DoWr31oNYSsufNjTdF_OkQPOLKjBo-3v0DhyaWnfYHKgrYZOWW9PmlYu5mQyBje_wxATpHobWLSpk0-Y8egNR95aB4dJ90xZeg-KH5gxbOVqo8KHSZSQZNfesDX280tTua5kJeLdON3zm8g4vKMG5LxFbNFtGavYBrqXDXzocYhSAT1Qe3pPRnKL8LidT-6NlJ_vVi8dMDlrJtdCReFvJUlnL-NQscdAQMt7_raBxA5-yFleYoYEyfL7oDIIxOTz1m00)

## Server architecture

K8s manages all server processes. The main components are:
### ChessEngine ReplicaSet
Here all the incoming RPCs end up. ChessEngine has the following logic:
- **Match RPC**: try to find any existing games in the Storage with state=WAITING_FOR_OPPONENT. If found, update (CAS) this game and return GameID to the sender.
- 
Incoming requests end up on one of the ChessEngine nodes. Game contexts are stored in MongoDB storage (a StatefulSet of master-slave pairs, sharded by GameID).
