1. Design Overview
1.1 Purpose

This document describes the system architecture, frontend design, real-time communication model, data flow, security mechanisms, and deployment strategy for CodeSus.

CodeSus is a real-time multiplayer coding game platform designed to increase engagement in programming education by combining:

Coding challenges

AI detection gameplay

Live voting

Real-time leaderboard

2. High-Level Architecture
2.1 System Architecture Overview
Users (Browser Clients)
        │
        ▼
Frontend (SPA - Vanilla JS)
        │
        ▼
WebSocket (Socket.io)
        │
        ▼
Backend Server (Node.js)
        │
        ├── Game Engine
        ├── Room Manager
        ├── AI Code Generator
        └── Score Engine
        │
        ▼
Deployment (AWS S3 + CloudFront + EC2/Lambda)

3. Frontend Architecture
3.1 Architecture Pattern

Single Page Application (SPA)

State-driven UI rendering

Event-based screen transitions

No full-page reloads

Screen Controller System

The app maintains a global state object:

const gameState = {
  phase: "home", 
  roomCode: null,
  players: [],
  submissions: [],
  votes: [],
  timer: 0
}


Each screen:

Subscribes to state updates

Re-renders when phase changes

Cleans up listeners on exit

3.2 Component Structure
Core UI Modules
/js
  ├── app.js
  ├── socket.js
  ├── stateManager.js
  ├── screens/
  │     ├── home.js
  │     ├── lobby.js
  │     ├── coding.js
  │     ├── voting.js
  │     └── results.js
  └── utils/
        ├── validation.js
        ├── animations.js
        └── toast.js

Design Principles

Modular JavaScript files

Separation of UI logic & socket logic

Centralized event dispatching

Reusable components (Cards, Buttons, Modals)

4. Backend Architecture
4.1 Server Stack

Node.js

Express.js

Socket.io

In-memory state management (hackathon scope)

4.2 Core Backend Modules
4.2.1 Room Manager

Responsibilities:

Generate unique 6-character room codes

Track active rooms

Assign host

Handle player join/leave

Auto-assign new host if needed

Data structure:

rooms = {
  ABC123: {
    host: socketId,
    players: [],
    phase: "lobby",
    challenge: {},
    submissions: [],
    votes: [],
    scores: {}
  }
}

4.2.2 Game Engine

Handles:

Phase transitions

Timer countdown

AI submission injection

Voting calculation

Score updates

Round progression

Phase flow:

Lobby → Coding → Voting → Results → Lobby

4.2.3 AI Code Generator

Predefined AI-generated solutions

Injected into submissions anonymously

Marked internally for result reveal

Not identifiable during voting

5. Real-Time Communication Design
5.1 WebSocket Lifecycle

Client connects on page load

Joins room

Receives real-time events

Sends actions (submit, vote, leave)

Server broadcasts updates

5.2 Event Architecture
Client → Server Events
Event	Purpose
createRoom	Create new game
joinRoom	Join existing room
startGame	Begin game
submitCode	Submit solution
castVote	Vote submission
leaveRoom	Exit game
Server → Client Events
Event	Purpose
roomUpdate	Player list update
phaseChange	Game phase transition
timerUpdate	Sync timer
submissionUpdate	Track submissions
voteUpdate	Track voting progress
resultsReveal	Reveal results
5.3 State Synchronization Strategy

Server is single source of truth

Clients render based on server updates

On reconnect → full state sync

Out-of-order message handling via timestamps

6. Data Flow Design
6.1 Coding Phase
Player writes code
      │
      ▼
Client validates input
      │
      ▼
WebSocket submitCode
      │
      ▼
Server stores submission
      │
      ▼
Broadcast submission count

6.2 Voting Phase
Player selects submission
      │
      ▼
castVote event
      │
      ▼
Server stores vote
      │
      ▼
Broadcast vote count
      │
      ▼
All voted → Calculate results

7. UI/UX Design System
7.1 Design Theme

Cyberpunk gaming aesthetic:

Neon Cyan (#00ffff)

Neon Pink (#ff00ff)

Deep Space Background (#0a0e27)

Retro fonts (Press Start 2P)

7.2 Animation Design

Fade transitions (0.3s)

Neon glow hover effects

Timer pulse when < 30s

Toast slide-in animation

Confetti for winners

All animations respect:

prefers-reduced-motion

8. Security Design
8.1 Frontend Security

Sanitize all inputs

Escape HTML (no innerHTML)

Disable eval()

Validate room codes

Prevent double submission

8.2 Network Security

HTTPS only

WSS WebSocket

Rate limiting

IP throttling

Origin whitelisting

9. Performance Design
9.1 Load Optimization

Total bundle < 500KB

Gzip compression

Async font loading

Lazy load demo games

CloudFront CDN caching

9.2 Runtime Optimization

RequestAnimationFrame animations

Clean up event listeners

Clear intervals on phase change

Minimal DOM nodes

Delta state updates

10. Deployment Architecture (AWS)
10.1 Hosting Setup
Component	Service
Static Frontend	AWS S3
CDN	CloudFront
Backend Server	EC2 / AWS Lambda
SSL	AWS Certificate Manager
10.2 Deployment Flow

Build static frontend

Upload to S3

Enable CloudFront CDN

Deploy Node.js server

Attach WSS endpoint

Enable HTTPS

11. Scalability Considerations

Although hackathon scope is limited, system supports:

Horizontal scaling via load balancer

Redis session store (future)

Room sharding

Auto-scaling EC2 instances

Serverless migration possible

12. Failure Handling Strategy
Failure	Recovery
WebSocket disconnect	Auto-reconnect (5 retries)
Host leaves	Auto-assign new host
Player drop mid-game	Continue game
Server crash	Stateless restart
Partial submission	Auto-save draft
13. Future Enhancements

Persistent accounts

Global leaderboard

AI behavior analytics

Code execution sandbox

Skill tracking dashboard

Mobile app version

Multiplayer tournaments

AWS Bedrock AI integration

14. Design Tradeoffs
Decision	Reason
Vanilla JS over React	Faster development
In-memory storage	Hackathon simplicity
No authentication	Lower friction
Predefined AI code	Deterministic results
15. Conclusion

CodeSus is designed as:

⚡ Real-time

🎮 Highly engaging

🚀 Lightweight

☁ AWS deployable

🔒 Secure by design

📱 Fully responsive

The architecture ensures:

Smooth multiplayer experience

Low latency gameplay

High engagement through interactive phases

Scalability beyond hackathon prototype

