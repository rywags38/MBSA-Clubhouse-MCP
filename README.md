# MBSA Clubhouse connector for Claude

This is a small server that lets Claude read **your own** MBSA Clubhouse data: your schedule, registrations, boats, crew commitments, sailing resume, and the public list of events, regattas and race series.

It uses the Clubhouse's official agent API (the same one described at members.massbaysailing.org/api/skill.md) with a **read-only agent token** you create yourself. It can't register you for anything, accept crew invites or change your profile; the Clubhouse blocks that for agent tokens, and the server only ever reads.

## What Claude can ask it

| Tool | What it returns |
|---|---|
| whoami | Confirms the token works (your name, memberships) |
| my_schedule | Everything upcoming: your events, bookmarks, registrations, crew commitments |
| my_registrations | Events you've paid to register for |
| my_profile, my_boats, boats_i_crew_on, my_sailing_resume | Your own account details |
| boat_crew_pools | Crew pools for one of your boats |
| upcoming_events, event_details, event_tickets | Event listings and details |
| my_registration_for_event, event_crew | Your registration and the crew roster for an event |
| regattas, regatta_details, race_series, series_races | Regattas and series |

Left out on purpose: the member directory and Crew Finder search, because they hold other members' personal details. They can be added later if you decide you want them.

## What it costs you

- **Hosting:** Render's free plan works. It goes to sleep after about 15 minutes idle, so the first question after a break takes 30 to 60 seconds. Render's paid Starter plan (about $7 a month) keeps it awake. Your choice.
- **Upkeep:** your Clubhouse token expires after 6 months. When it does, make a new one and paste it into Render (step 4).
- **Risk:** anyone who has the full connector address can read your Clubhouse data through it. Keep that address private, like a password.

## Setup (about 20 minutes, no coding)

### 1. Make a fresh Clubhouse token
1. Sign in to the Clubhouse and go to **Profile → AI agent access → Manage tokens** (or members.massbaysailing.org/members/agents).
2. **Revoke the token you pasted into chat earlier.** It has been seen, so it shouldn't be used.
3. Click **New token**, name it "Claude connector", and copy it. It's shown only once. Don't paste it into any chat; it goes only into Render in step 4.

### 2. Put these files on GitHub
1. On github.com, create a new **private** repository (for example `mbsa-clubhouse-mcp`).
2. Click **Add file → Upload files** and upload `server.py`, `requirements.txt` and `render.yaml` from this folder. Commit.

### 3. Create the service on Render
1. Sign in to render.com (you can sign in with GitHub).
2. Click **New → Blueprint**, pick the repository, and click **Apply**. Render reads `render.yaml` and sets up the service.
   (Or **New → Web Service**: build command `pip install -r requirements.txt`, start command `python server.py`.)

### 4. Add the two settings
In the service's **Environment** tab:
- `CONCORDA_TOKEN`: paste your new Clubhouse token.
- `SERVER_SECRET`: Render fills this in with a random value if you used the Blueprint. Otherwise make up a random string of at least 24 letters and numbers.

Save. Render redeploys. When the log says it's live, note the service address, for example `https://mbsa-clubhouse-mcp.onrender.com`.

### 5. Add it to Claude
1. Copy your `SERVER_SECRET` value from the Environment tab.
2. Build your connector address: service address + `/mcp/` + secret, for example
   `https://mbsa-clubhouse-mcp.onrender.com/mcp/YOUR_SERVER_SECRET`
3. In Claude (claude.ai), go to **Settings → Connectors → Add custom connector**, name it "MBSA Clubhouse", paste the address and add it. No sign-in is needed; the secret in the address is the lock.
4. Test it by asking Claude: "What's on my MBSA schedule?"

## If something goes wrong

- **"The Clubhouse rejected the token":** the token expired or was revoked. Make a new one (step 1) and update `CONCORDA_TOKEN` (step 4).
- **Service won't start:** the Render log names the missing setting (`CONCORDA_TOKEN` or `SERVER_SECRET`).
- **Address leaked:** change `SERVER_SECRET` in Render and update the connector in Claude. If you're worried, also revoke the Clubhouse token and make a new one.

## Notes for whoever maintains the Clubhouse

The cleanest long-term setup would be an MCP endpoint built into the Clubhouse itself (the API is FastAPI, so a library such as fastapi-mcp could expose the read-only routes behind OAuth). Then any member could connect Claude without hosting anything or copying tokens.

## Files

- `server.py`: the server (Python, official MCP SDK, read-only GET calls only, never logs the token)
- `requirements.txt`: the libraries Render installs
- `render.yaml`: Render setup file
