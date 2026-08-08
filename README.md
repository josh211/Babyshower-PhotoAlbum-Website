# Baby Shower Photo Album

A self-hosted photo album for the baby shower — pictures and the memories behind them — built on [Lychee](https://github.com/electerious/Lychee) (the modern [LycheeOrg/Lychee](https://github.com/LycheeOrg/Lychee) fork), running on your own computer and published to your Cloudflare domain via Cloudflare Tunnel. No cloud photo host, no monthly fee — the machine in your house is the server.

## How it works

- **Lychee** runs locally in Docker and serves the gallery (albums, photo grid, lightbox, captions/descriptions per photo).
- **Cloudflare Tunnel** exposes it securely on your domain (e.g. `photos.yourdomain.com`) without opening any ports on your router or exposing your home IP.
- Everything (photos, database) lives in the `lychee/` folder on this machine.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and Docker Compose installed on this computer.
- A domain already added to your Cloudflare account.
- A free Cloudflare account with Zero Trust enabled (just visiting the Zero Trust dashboard the first time turns it on).

## 1. Configure

```bash
cp .env.example .env
```

Edit `.env` and set:
- `DB_PASSWORD` / `DB_ROOT_PASSWORD` — pick strong passwords.
- `APP_URL` — the public URL you intend to use, e.g. `https://photos.yourdomain.com`.
- `TIMEZONE` — your local timezone (e.g. `America/New_York`).

Leave `CLOUDFLARE_TUNNEL_TOKEN` blank for now — you'll get it in step 3.

## 2. Start Lychee

```bash
docker compose up -d lychee_cache lychee_db lychee
```

Open `http://localhost:90` (or whatever `LYCHEE_PORT` you set) in a browser on this machine. Lychee will prompt you to create the admin account on first visit — do that now.

Once logged in:
1. Create an album, e.g. **"Baby Shower 2026"**.
2. Upload photos into it (drag-and-drop is supported).
3. Click each photo → **Edit** to add a caption/description — this is where you write the memory that goes with it ("Grandma guessing the due date", "the diaper-changing relay", etc.).
4. You can also add a description to the album itself as an intro to the day.

## 3. Create the Cloudflare Tunnel

1. Go to [one.dash.cloudflare.com](https://one.dash.cloudflare.com) → **Networks → Tunnels → Create a tunnel**.
2. Choose **Cloudflared**, name it something like `baby-shower-album`.
3. On the install step, copy the **tunnel token** shown (a long string after `--token`) and paste it into `.env` as `CLOUDFLARE_TUNNEL_TOKEN`.
4. Still in the dashboard, add a **Public Hostname**:
   - Subdomain: e.g. `photos`
   - Domain: your domain
   - Service Type: `HTTP`
   - URL: `lychee:80` (the internal Docker service name/port — not `localhost`)
5. Save.

## 4. Start the tunnel

```bash
docker compose --profile tunnel up -d cloudflared
```

Cloudflare will start routing `https://photos.yourdomain.com` (or whatever subdomain you chose) straight to Lychee running on this machine, over an outbound-only encrypted connection — nothing needs to be port-forwarded.

## Everyday use

- Start everything (e.g. after a reboot):
  ```bash
  docker compose --profile tunnel up -d
  ```
- Stop everything:
  ```bash
  docker compose --profile tunnel down
  ```
- Check logs:
  ```bash
  docker compose logs -f lychee
  docker compose logs -f cloudflared
  ```
- Your photos and database live in `./lychee/` on this machine — back that folder up periodically.

## Sharing with guests

Once the tunnel is up, just send people the link (`https://photos.yourdomain.com`). In Lychee you can:
- Make an album **public** (viewable without login) so guests can browse without an account.
- Optionally set a **password** on the album for a bit of privacy.
- Enable **downloads** if you want guests to be able to save full-resolution photos.
- Turn on guest **uploads** for an album if you want attendees to add their own photos from the shower.

These are all toggles on the album's settings page in Lychee.
