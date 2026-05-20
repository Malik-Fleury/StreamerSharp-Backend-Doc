# Architecture

The solution is composed of four main projects:

## Authentication Service

Centralized authentication service.

### Responsibilities

- generation of signed JWT tokens;
- identity validation;
- exposure of a public certificate allowing other services to verify tokens.

---

## Twitch Service

Intermediate service acting as a bot connected to Twitch.

It simplifies interactions with the Twitch ecosystem and exposes a consistent interface to client applications.

### Responsibilities

- JWT validation;
- communication with the Twitch API;
- data transformation;
- exposure of a consistent API for client applications.

---

## Spotify Service

Intermediate service simplifying interactions with the Spotify API.

### Responsibilities

- JWT validation;
- communication with the Spotify API;
- data normalization;
- exposure of a consistent API for client applications.

---

## Shared

Shared library containing contracts used between services and client applications.

### Contents

- DTOs;
- requests;
- responses;
- communication models.

### Purpose

Provide consistent interfaces across services and reduce duplication on the client side.

## Flow
```mermaid
flowchart TD

    User[Client / Application] -->|Login| Auth[Authentication Service]

    Auth -->|Generate JWT\nPrivate Key Signature| JWT[(JWT Token)]
    Auth -->|Expose Public Certificate\n/.well-known| Cert[(Public Certificate)]

    User -->|JWT Request| Twitch[Twitch Service]
    User -->|JWT Request| Spotify[Spotify Service]

    Cert -->|Fetch Public Key| Twitch
    Cert -->|Fetch Public Key| Spotify

    JWT --> Twitch
    JWT --> Spotify

    Twitch -->|Validate Signature\nAuthorize Request| TwitchLogic[Twitch API Wrapper\nEvents / Chat / Streams]

    Spotify -->|Validate Signature\nAuthorize Request| SpotifyLogic[Spotify API Wrapper\nTracks / Playback]

    TwitchLogic --> User
    SpotifyLogic --> User
```

## Sequence

```mermaid
sequenceDiagram

    participant C as Client
    participant A as Auth Service
    participant T as Twitch Service
    participant S as Spotify Service

    C->>A: Login(credentials)
    A->>C: JWT (signed with private key)

    T->>A: Request public certificate
    A->>T: Public certificate

    C->>T: API request + JWT
    T->>T: Verify JWT signature
    T->>C: Authorized response

    S->>A: Request public certificate
    A->>S: Public certificate

    C->>S: API request + JWT
    S->>S: Verify JWT signature
    S->>C: Authorized response
```