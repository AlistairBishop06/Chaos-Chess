# Chaos Chess

Chaos Chess is a full-stack multiplayer chess variant that turns a normal match into a live strategy party game. Players make standard chess moves while periodically drafting rule cards that can alter pieces, board geometry, hazards, win conditions and even the interface itself.

The project combines a custom chess engine with an authoritative Node.js multiplayer server, persistent player accounts, singleplayer progression and a canvas-based browser client. Socket.IO keeps matches synchronised in realtime while the server remains responsible for move legality, timers and rule resolution.

**Live demo:** https://chaoschess.onrender.com/

## Features

### Realtime multiplayer

* Create public or private two-player lobbies
* Join private matches using six-character lobby codes
* Browse currently available public games
* Recover player connections after temporary disconnects
* Synchronise player-specific game state through Socket.IO
* Keep move validation, timers and rule effects authoritative on the server

### Custom chess and rule engine

Chaos Chess implements its chess logic directly rather than delegating matches to a third-party chess engine.

* Legal move generation and server-side move validation
* Check and checkmate handling
* Castling, en passant and promotion
* Rule effects that can change pieces, squares and movement behaviour
* Instant, delayed, timed and permanent rule lifecycles
* Targeted rules that require additional player interaction

The rule system extends normal chess with effects such as portals, lava, black holes, missing squares, piece mutation, shields, board rotation and alternative win conditions.

### Rule drafting and mini-games

Matches periodically enter a rule-selection phase where players choose cards before play resumes.

Several rules introduce their own interactive systems, including:

* Rock Paper Scissors
* Coinflip Wager
* Supermarket
* Fruit Machine
* Mutant piece fusion
* Pawn Soldier targeting

These interactions are controlled by the same server-side match state as normal chess turns rather than running as isolated client effects.

### Accounts and progression

Players can create persistent accounts containing gameplay and profile state.

* Wins, losses, draws and rating
* Match history
* Achievements and coin rewards
* Rule collection progress
* Friends, rivals and clubs
* Profile customisation
* Equipped cosmetics
* Singleplayer campaign progress

### Singleplayer campaign

A singleplayer mode runs matches against Chaos Bot using the same underlying game systems as multiplayer.

Campaign progress can unlock additional rule pools and is stored alongside the player's account data.

### Cosmetics

Coins earned through play can be spent on cosmetic items including:

* Avatar styles
* Profile banners
* Board skins
* Piece skins
* Animated profile borders
* Emotes
* Rule card backs

Avatar graphics are generated with DiceBear using stable player-specific seeds so an account can retain a consistent identity across available styles.

## Basic workflow

1. Create an account or sign in.
2. Start a singleplayer game, create a lobby or join another player's lobby.
3. Make a legal chess move from the browser.
4. The server validates the move and broadcasts the resulting player-specific state.
5. When a rule draft begins, each player chooses from the available rule cards.
6. The server resolves the selected effects and updates the match.
7. Continue through normal moves, rule events and mini-games until a win condition is reached.
8. Completed matches update persistent statistics, progression and rewards.

## Architecture

Chaos Chess uses an authoritative client-server architecture.

The browser is responsible for presentation, input and animation. It renders the board and game effects, displays menus and modals, and sends player intentions to the backend.

The Node.js server owns the actual match state. Chess moves, rule effects, timers, lobby membership, mini-games and results are processed on the server before updated state is sent back through Socket.IO. This prevents each browser from maintaining an independent version of the game.

Core chess rules and Chaos-specific behaviour are separated into different modules. `ChessEngine.js` handles standard chess mechanics, while `Game.js` coordinates match state and `RuleManager.js` manages the lifecycle of rule effects.

Account handling is kept outside the game engine. The account service manages authentication, profiles, progression, cosmetics, social information and persistence. Match results are passed into a separate recorder which updates the relevant account statistics after a game.

Persistence can use PostgreSQL when a database connection is configured or fall back to a local JSON store during development.

## Tech stack

* **Frontend:** HTML, CSS, vanilla JavaScript, Canvas API
* **Backend:** Node.js, Express
* **Realtime networking:** Socket.IO
* **Persistence:** PostgreSQL or local JSON storage
* **Database client:** `pg`
* **Avatar generation:** DiceBear
* **Package management:** npm

## Project structure

```text
server.js                         Express and Socket.IO application entry point

public/
  index.html                      Main browser interface and modal structure
  client.js                       Game renderer, interactions and client behaviour
  style.css                       Layout, themes, animations and responsive styling
  js/                             Smaller browser-side modules

src/server/
  accountService.js               Authentication, profiles, persistence and progression
  botController.js                Singleplayer opponent behaviour
  lobby.js                        Lobby and room helpers
  matchRecorder.js                Match results, ratings and account updates
  realtimeController.js           Socket.IO lobby and game event handling

src/server/game/
  ChessEngine.js                  Standard chess rules and legal move generation
  Game.js                         Match state and Chaos-specific game orchestration
  miniGames.js                    Interactive rule mini-games
  results.js                      Match result handling
  stateUtils.js                   Shared game-state helpers

src/server/game/rules/
  RuleManager.js                  Rule timing and lifecycle management
  ruleset.js                      Chaos rule definitions
```

## Configuration

The application reads configuration from environment variables and can also load local `.env` or `env` files.

| Variable            | Required | Purpose                                                                     |
| ------------------- | -------- | --------------------------------------------------------------------------- |
| `PORT`              | No       | HTTP server port. Defaults to `3000`.                                       |
| `DATABASE_URL`      | No       | PostgreSQL connection string used for persistent account storage.           |
| `NEON_DATABASE_URL` | No       | Alternative PostgreSQL connection variable.                                 |
| `POSTGRES_URL`      | No       | Alternative PostgreSQL connection variable.                                 |
| `DISABLE_DATABASE`  | No       | Forces the application to use local JSON persistence instead of PostgreSQL. |
| `DEBUG_MODE`        | No       | Controls the application's runtime debug behaviour.                         |

When no database URL is supplied, user, session and club data are stored locally instead.

Never commit real database credentials or other secrets to the repository.

## Run locally

Requires Node.js and npm.

Install the dependencies:

```bash
npm install
```

Start the server:

```bash
npm start
```

The application is then available at:

```text
http://localhost:3000
```

The development script currently runs the same server entry point:

```bash
npm run dev
```

## Persistence

Chaos Chess supports two persistence modes.

### Local development

Without a configured PostgreSQL connection, account state is stored in a JSON file. This keeps local setup lightweight and allows the application to run without an external database service.

### PostgreSQL

When a supported database URL is available, the account service creates and uses PostgreSQL tables for users, sessions and clubs.

Database changes are serialised through the account service and written using parameterised queries and transactions where multiple related operations need to remain consistent.

## Security and privacy

Game actions are validated by the authoritative server rather than trusting the state calculated by individual clients.

Account and session handling is also performed by the backend, while authenticated API operations use session credentials rather than relying on browser-provided identity alone.

The application accepts JSON request bodies with a defined size limit and keeps database credentials outside the source through environment configuration.

Player account information, game statistics, progression, social data and session state are persisted either locally or in PostgreSQL depending on configuration.

## Testing

The repository does not currently contain an automated test suite.

The existing npm test command is a placeholder:

```bash
npm test
```

It currently reports that no tests are configured rather than running unit or integration tests.

## Deployment

The application runs as a single Node.js service containing both the Express API and Socket.IO server. Static browser files are served directly by Express, so the frontend and realtime backend can be deployed together.

The current public deployment is hosted on Render:

https://chaoschess.onrender.com/

Production deployments can use PostgreSQL for persistent account data while local development can operate without an external database.

## Engineering decisions

### Authoritative multiplayer state

Clients send player intentions rather than directly deciding the result of a move. The server validates actions and distributes the resulting state, keeping chess rules and Chaos effects consistent between both players.

### Separate chess and Chaos logic

Standard chess behaviour is isolated inside `ChessEngine.js`, while match orchestration and variant effects live in separate game and rule modules. This prevents every new Chaos rule from being embedded directly into basic move generation.

### Player-specific synchronisation

The realtime controller can send state tailored to an individual player. This allows mechanics involving hidden or private information without exposing the complete server state to every connected client.

### Dual persistence

Supporting both local JSON and PostgreSQL keeps development setup simple while allowing the deployed application to use a proper database without requiring two separate account systems.

### Connection recovery

Socket.IO connection-state recovery and player reconnection logic allow a temporary network interruption to be handled without immediately discarding an active match.

## Limitations

* Automated tests are not currently configured.
* The browser client is implemented primarily in a large vanilla JavaScript application rather than a component framework.
* Local JSON persistence is intended as a lightweight development fallback rather than a replacement for the PostgreSQL-backed deployment.
