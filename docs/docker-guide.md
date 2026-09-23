# Docker Guide

Markdown viewer for `public/specs/`. Start the container.

```bash
./dev up
```

Viewer runs at `http://localhost:8088`.

The `public` directory is bind mounted read-only as the web root, so edits to any file under it are live, no restart. Only `nginx.conf` needs one.

```bash
./dev restart
```

On localhost the viewer polls `files.json` and the open document every two seconds and repaints on a change. A deployed copy skips the poll and loads once.

## Services

| Service | Detail |
| --- | --- |
| `flows` | Nginx serving `public/` on port 8088 |

## Viewing logs

Nginx runs inside the container, so its output goes to the container logs, not your terminal.

```bash
./dev logs
```

## Shortcuts

| Command | Runs |
| --- | --- |
| `./dev up` | Start the container, recreating it |
| `./dev down` | Stop the container |
| `./dev restart` | Down, then up |
| `./dev logs` | Follow the nginx logs |
| `./dev sh` | A shell in the container |
