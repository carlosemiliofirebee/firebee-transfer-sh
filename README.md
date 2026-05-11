# transfer-sh-firebee

Self-hosted file sharing service for the Firebee team, powered by [transfer.sh](https://github.com/dutchcoders/transfer.sh).

## Access

```
http://transfer.firebee.com.br:3210
```

> Port `:3210` is required in all URLs.

## Quick Usage

**Upload:**
```bash
curl --upload-file ./file.txt http://transfer.firebee.com.br:3210
# → http://transfer.firebee.com.br:3210/G2k6d6aMSl/file.txt
```

**Download:**
```bash
curl -O http://transfer.firebee.com.br:3210/G2k6d6aMSl/file.txt
```

**Shell function (add to `~/.bashrc`):**
```bash
transfer() {
  curl --upload-file "$1" "http://transfer.firebee.com.br:3210/$(basename "$1")"
  echo
}
```

## Service Limits

| Parameter | Value |
|---|---|
| Max file size | 5 GB |
| Retention | 14 days (auto-deleted) |
| Rate limit | 10 requests per window |
| Storage | Local disk (VPS) |

## Documentation

Full usage guides (pt-BR and en-US) in Markdown and PDF:

- [Guia pt-BR](doc/pt-BR/md/2026-05-11_guia-transfer-sh.md) — [PDF](doc/pt-BR/pdf/2026-05-11_guia-transfer-sh.pdf)
- [English Guide](doc/en-US/md/2026-05-11_transfer-sh-guide.md) — [PDF](doc/en-US/pdf/2026-05-11_transfer-sh-guide.pdf)

## Infrastructure

Runs via Docker Compose using the official `dutchcoders/transfer.sh:latest` image. Storage is mounted at `./data`.

```bash
docker compose up -d
```
