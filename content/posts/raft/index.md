---
title: "Raft Consensus: A Beginner's Guide to Distributed Systems Harmony"
date: 2024-01-16T20:02:19+05:00
draft: false

description: "Unlock the basics of Raft Consensus Algorithm, covering State Machine Replication, election triggers, and voting mechanisms. This beginner-friendly guide simplifies distributed systems, making Raft's role in achieving consensus easily understandable."

resources:
  - name: "featured-image"
    src: "featured-image.webp"

tags: ["underthehood"]
categories: ["Distributed Systems"]
theme: "full"
images: ["https://theundersurfers.com/raft/featured-image.webp"]
seo:
  images: ["https://theundersurfers.com/raft/featured-image.webp"]
---

<!--more-->


# Raft

In this article, I will try to explain how the raft algorithm works. I will try to cover all the major aspects of the raft algorithm

![Distributed systems](https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/Raft%2Fraft-blog-img.jpg?alt=media&token=bc606e7f-92d8-4bd7-84d0-150716896b09)

## Intro

- If we have only a single server for the whole system, it can cause a single point of failure.
- Now to handle this problem we add multiple server instances.
- But now there is a new problem, consensus.
- All instances should be synced with the latest data and should be consistence,  now we need to handle this.

## Consensus

- All the servers should agree on a statement and achieve consensus (general agreement)
- There are many ways to achieve consensus
    - Proof of Work (popular in the decentralized networks)
    - Paxos Algo (popular in Distributed systems but difficult to understand)
    - Raft (simple and easy to implement)

## Elements of Raft

Each instance would have three main elements

- State machine
- Log
- Raft-protocol

### State machine

**state machine is actually** is aduplicated program that runs on each server. Let's say in our example the state machine is a program that server some endpoint i.e. GET, DELETE, POST a `key`

If we have three instances and all 3 are fed up with the same commands all 3 would have the same state at the end. This is called state machine replication

#### SMR

In computer science, **state machine replication** (**SMR**) or **state machine approach**
 is a general method for implementing a fault-tolerant service by 
replicating servers and coordinating client interactions with server 
replicas. T

### Log 

Each command that a replica gets, get saved in the log

Every replica should have the same sequence of commands

## Who commands? 
Now who commands this replica? For this, there is a single leadership election

### States of Replicas
There is a single leader election and each replica is given a set of 3 states. Each replica could be in only one state at a time.

- Leader
- Candidate
- Follower

#### Follower 👷
- Each replica is a follower at the start.
- It gets accepted by the leader.
- The follower cannot receive request from the client

When there is no leader, follower elect a new Leader

#### Leader 👑
- A Leader can send commands to the follower
- Can receive requests from the Client

#### Candidate 🤚
- The Candidate starts the election and votes himself

**Client:**

- Client can only send messages to the Leader
- If it tries to send a message to the follower, the load balancer (in front of the cluster) forwards the command to the leader

## When to elect?

An election can be held due to any of the reasons.

- Leader goes offline
- Network Latency issues that can cause election timeout.

### Election Timeout:

There is an election time out for each follower. A replica must hear from the leader within this time. 

Election timeout would be random for each follower. Typically it is between 150ms to 300ms

## How to Elect? 

If no response from the leader then the replica starts the election

- It becomes the candidate
- Votes himself
- Request votes from other followers

### Requesting votes

In order to request the votes from other followers the candidates send the RPC to the other candidates and then wait for them to reply back with the votes

#### RPC

Raft uses remote procedure calls  (RPC) for incluster communication. The raft protocol uses two types of RPCs

- **Request Voting** RPC is sent by the Candidate nodes to gather votes during an election
- **Append Entries** are used by the Leader node for replicating the log entries and also as a heartbeat mechanism to check if a server is still up.

Now for requesting vote, the candidate sends the message as an RPC. The message contains two elements.

- Total number of entries in the candidate’s log
- Term of the latest entry

#### Term

A term is a counter value that represents an arbitrary time period during the lifetime of a raft cluster. 

- Each node maintains its own term number.
- Each replica starts with the term number zero.
- Terms increments whenever an election happens.
- A term can have at most one Leader

#### Votes

Follower won't give the vote if there is any inconsistency in the candidate's logs.

- If a candidate gets the majority of the votes and becomes the leader
- If the candidate fails to get  elected, it reverts to a follower

## After Election

### APPENDENTRIES

This is another type of RPC that the leader sends to all the followers after he becomes the leader. It tells the followers to replicate the log entries and also serves for the heartbeat mechanism.

#### Heartbeat

A heartbeat timeout determines how often these messages are sent to the followers. So that they know the leader is still alive.

## Work Flow

Let's take an example in which a client approaches the leader. 

![Raft algorithm](https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/Raft%2FRaft_dgxsd6xv_1705420379977..bmp?alt=media&token=b873ee86-e953-41c6-a9a3-7a6b2e5584db "Raft Working")

- The Client commands the leader with the `SET key` operation.
- Leader appends new log entry `SET`
    - Appending the entry only doesn’t perform the operation it
    - The Operation would only be done on `COMMIT`
    - Every follower should have the log entry `SET` appended in order to perform the operation.
- The Leader sends the `APPENDENTRIES`  messages to its followers
    - On receiving RPC followers do a consistency check of the logs with  the leader log
    - After passing the consistency check the follower appends the entry to its log
- Once all the followers append the entries the leader `COMMITS` the entry and applies the operation
    - Now we have the leader node with the operation performed,  the key value has been set.
- Now leader again sends the `APPENDENTRIES` messages but this time to notify that entry has been committed and they should too.
- All the followers commit the entry too
- Now all the clusters have come to a consensus.

This is known as log replication,  as the replicas agree on a sequence of values and not with a single value like the Paxos algorithm.

## Trade-Off

- The Single Leader model increases the linearizability and is the strongest consistency model.
- However, scalability is compromised. Having a single leader for multiples clients can be a bottleneck.

## Imp Scenarios

- Changes with the leader node
    - entries appended in the follower nodes
    - every follower returns to the leader with a signal **appended**
    - leader gets appended signal from all f nodes —> leader commits

- Follower left —> other keep synced with the server
- Follower again wanna join —> server also sends index of the previous log entry

    - If new follower doesn’t have this one
    - The server will simply send the next previous index  and keep on
    - until the index gets matched with the index of the follower

  
## Acknowledgment
- https://www.geeksforgeeks.org/raft-consensus-algorithm/
- https://www.youtube.com/watch?v=ZyqAbQkpeUo
