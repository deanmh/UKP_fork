# UKP Kickball Roster Manager

A web application for managing kickball team rosters, lineups, and game planning. Built with Flask (Python) backend and vanilla HTML/CSS/JavaScript frontend.

## Features

- **Main Roster Management**: Add and manage your core team roster with gender tracking
- **Substitute Players**: Manage substitute players that can be added on a game-by-game basis
- **Game Lineup Setup**: 
  - 7 innings with 11 field positions + "Out" position
  - Spreadsheet-style interface for setting lineups
  - Player status management (IN/OUT for each game)
  - Kicking order management with up/down controls
- **Validation & Warnings**:
  - Female player count per inning (requires 4)
  - Duplicate position detection
  - Unused position tracking
  - Player sit-out counts per game
- **Game Details**:
  - Date picker (defaults to next Thursday)
  - Editable team name (default: "Unsolicited Kick Pics")
  - Editable opponent name
- **Authentication**: 
  - Public viewing of lineups (read-only)
  - Login required for editing
- **Modern UI**:
  - Dark theme with sports aesthetic
  - Mobile responsive design
  - Smooth animations

## Quick Start

### Local Development

1. Install dependencies:
```bash
pip install -r requirements.txt
```

2. Run the application:
```bash
python app.py
```

3. Open http://localhost:8080 in your browser

### Docker

Build and run with Docker:

```bash
docker build -t ukp-kickball .
docker run -p 8080:8080 -v $(pwd)/data:/app/data ukp-kickball
```

Or use Docker Compose:

```bash
docker-compose up -d
```

## First Time Setup

1. Open the application in your browser
2. Click "Login" and create the first user account
3. Login with your credentials
4. Add players to your main roster in the "Roster" tab
5. Optionally add substitute players

## Setting Up a Game

1. Go to the "Game Lineup" tab
2. Adjust the game date, team name, and opponent name as needed
3. Toggle players IN (green) or OUT (gray) for the game
4. Expand "Substitutes" to add substitutes to the game
5. Use the spreadsheet view to set positions for each inning
6. Use ↑↓ buttons to reorder the kicking lineup
7. Click "Copy Inning 1 to All" to quickly fill all innings

## Viewing Lineups

- Use the "View Lineup" tab to see lineups for any game
- This view is accessible without login (read-only)
- Select different games from the dropdown

## Positions

Each inning has 12 positions:
- **P** - Pitcher
- **C** - Catcher
- **1st** - First Base
- **2nd** - Second Base
- **3rd** - Third Base
- **SS** - Short Stop
- **LF** - Left Field
- **LC** - Left Center
- **CF** - Center Field
- **RC** - Right Center
- **RF** - Right Field
- **Out** - Sitting out the inning

## Tech Stack

- **Backend**: Flask (Python)
- **Frontend**: Vanilla HTML/CSS/JavaScript
- **Database**: SQLite
- **Production Server**: Gunicorn

## API Reference

All API endpoints return JSON. Endpoints marked with 🔒 require authentication.

### Authentication

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/auth/status` | Check if user is authenticated |
| POST | `/api/auth/login` | Login with username/password |
| POST | `/api/auth/logout` | Logout current user |
| POST | `/api/auth/register` | Register first user (only works if no users exist) |
| GET | `/api/auth/has-users` | Check if any users exist |

### Roster Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/roster` | Get all main roster players |
| POST 🔒 | `/api/roster` | Add a player to main roster |
| DELETE 🔒 | `/api/roster/<name>` | Delete a player from main roster |
| PUT 🔒 | `/api/roster/<name>/gender` | Toggle player gender |

### Substitutes

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/substitutes` | Get all substitute players |
| POST 🔒 | `/api/substitutes` | Add a substitute player |
| DELETE 🔒 | `/api/substitutes/<name>` | Delete a substitute |
| PUT 🔒 | `/api/substitutes/<name>/gender` | Toggle substitute gender |

### Games

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/games` | Get all games |
| GET | `/api/games/current` | Get or create current game (next Thursday) |
| GET | `/api/games/<id>` | Get a specific game |
| PUT 🔒 | `/api/games/<id>` | Update game details (date, team name, opponent) |
| POST 🔒 | `/api/games/<id>/logo` | Upload team logo (multipart form) |
| DELETE 🔒 | `/api/games/<id>/logo` | Delete team logo |

### Player Status (per game)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/games/<id>/status` | Get all player statuses for a game |
| PUT 🔒 | `/api/games/<id>/status/<player>` | Toggle player IN/OUT status |

### Lineup

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/games/<id>/lineup` | Get lineup for a game (for editing) |
| GET | `/api/games/<id>/lineup/published` | Get **published** lineup (public view) |
| PUT 🔒 | `/api/games/<id>/lineup/<player>/<inning>` | Set player position for an inning |
| POST 🔒 | `/api/games/<id>/lineup/copy` | Copy inning 1 to all innings |
| POST 🔒 | `/api/games/<id>/lineup/reset` | Reset all lineup positions |
| PUT 🔒 | `/api/games/<id>/order/<player>` | Move player up/down in kicking order |

### Publishing

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST 🔒 | `/api/games/<id>/publish` | Publish lineup (makes it publicly visible) |
| POST 🔒 | `/api/games/<id>/unpublish` | Unpublish lineup (hides from public) |

### Example API Calls

```bash
# Get all games
curl https://your-domain.com/api/games

# Get published lineup for game 1 (public, no auth required)
curl https://your-domain.com/api/games/1/lineup/published

# Login
curl -X POST https://your-domain.com/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": "yourpassword"}'
```

## Project Structure

```
UKP/
├── app.py              # Flask backend API
├── requirements.txt    # Python dependencies
├── Dockerfile         # Container configuration
├── docker-compose.yml # Docker Compose config
├── static/
│   ├── index.html     # Main HTML page
│   ├── css/
│   │   └── style.css  # Styles
│   ├── js/
│   │   └── app.js     # Frontend JavaScript
│   └── images/
│       └── logo.png   # Team logo
└── data/
    └── kickball_roster.db  # SQLite database (created automatically)
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `SECRET_KEY` | Flask session secret key | `ukp-kickball-secret-key-change-in-production` |
| `PORT` | Server port | `8080` |
| `FLASK_DEBUG` | Enable debug mode | `false` |

## Data Persistence

**Important**: All data (users, rosters, games, lineups) is stored in the SQLite database located in the `data/` directory.

- When using Docker Compose, the `./data:/app/data` volume mount ensures your data persists across container restarts and redeployments
- **Your admin user and all data will be preserved** when you rebuild and redeploy the container
- To backup your data, simply copy the `data/` folder
- To reset all data, delete the `data/kickball_roster.db` file and the `data/logos/` folder

## Security Notes

- Passwords are hashed using SHA256
- Only the first user can be created through the UI
- For production, set a strong `SECRET_KEY` environment variable
- Consider using HTTPS (via reverse proxy like nginx)
- The SQLite database is stored in `/app/data` (Docker) or `./data` (local)

## File Cleanup Guide

This project was migrated from Streamlit to Flask. The following files are **legacy/unused** and can be safely deleted:

### Files to DELETE (not needed)

| File | Reason |
|------|--------|
| `.streamlit/config.toml` | Streamlit configuration - not used with Flask |
| `deploy.sh` | Old Streamlit deployment script |
| `DEPLOYMENT.md` | Outdated Streamlit deployment guide |
| `AWS_DEPLOYMENT.md` | AWS guide for Streamlit |
| `EC2_QUICK_START.md` | EC2 guide for Streamlit |
| `SETUP_QUICK.md` | Outdated setup guide |
| `launch-ec2.sh` | EC2 launch script (Streamlit) |
| `setup-aws.sh` | AWS setup script (Streamlit) |
| `.github/workflows/deploy-aws.yml` | GitHub Actions configured for Streamlit |

### Files to KEEP (required)

| File | Purpose |
|------|---------|
| `app.py` | Flask backend API |
| `static/index.html` | Main HTML page |
| `static/css/style.css` | Styles |
| `static/js/app.js` | Frontend JavaScript |
| `static/images/logo.png` | Team logo |
| `Dockerfile` | Docker container build |
| `docker-compose.yml` | Docker Compose config |
| `requirements.txt` | Python dependencies |
| `README.md` | This documentation |
| `.gitignore` | Git ignore rules |
| `.dockerignore` | Docker ignore rules |

### Folders

| Folder | Status |
|--------|--------|
| `data/` | Auto-created, contains database - should be in `.gitignore` |
| `static/` | Required - frontend assets |
| `.streamlit/` | Can delete entire folder |
| `.github/` | Can delete if not using AWS deployment |

## License

MIT License
