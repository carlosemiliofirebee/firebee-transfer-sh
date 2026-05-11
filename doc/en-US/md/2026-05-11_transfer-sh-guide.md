# Usage Guide — transfer.firebee.com.br

## Overview

**transfer.sh** is a lightweight, self-hosted file sharing service accessed entirely via the command line with `curl`. There is no web upload interface and no account to create — simply upload a file and share the generated URL.

The Firebee instance is available at:

```
http://transfer.firebee.com.br:3210
```

> **Important:** port `:3210` is required in all URLs. The service does not respond on the default port 80/443.

---

## Uploading Files

### Basic syntax

```bash
curl --upload-file <file-path> http://transfer.firebee.com.br:3210
```

The server returns a unique URL for the file. Real example:

```bash
$ curl --upload-file ./teste.txt http://transfer.firebee.com.br:3210
http://transfer.firebee.com.br:3210/G2k6d6aMSl/teste.txt
```

The generated URL contains a random identifier (`G2k6d6aMSl`) followed by the original filename.

### Examples by file type

**Text file:**
```bash
curl --upload-file ./report.txt http://transfer.firebee.com.br:3210
```

**Compressed archive:**
```bash
curl --upload-file ./backup.tar.gz http://transfer.firebee.com.br:3210
```

**Image:**
```bash
curl --upload-file ./screenshot.png http://transfer.firebee.com.br:3210
```

**Any extension works** — the filename in the URL preserves the original extension.

---

## Downloading Files

To download, use the URL returned on upload directly in a browser or via `curl`:

```bash
curl http://transfer.firebee.com.br:3210/G2k6d6aMSl/teste.txt -o teste.txt
```

Or save with the original filename automatically:

```bash
curl -O http://transfer.firebee.com.br:3210/G2k6d6aMSl/teste.txt
```

---

## Uploading via Pipe (stdin)

You can send data directly from a pipe without creating an intermediate file:

```bash
cat file.txt | curl -X PUT --upload-file "-" http://transfer.firebee.com.br:3210/file.txt
```

**Pipe use cases:**

Compress and send in a single command:
```bash
tar czf - ./folder/ | curl -X PUT --upload-file "-" http://transfer.firebee.com.br:3210/folder.tar.gz
```

Send command output:
```bash
ps aux | curl -X PUT --upload-file "-" http://transfer.firebee.com.br:3210/processes.txt
```

Database dump directly to the service:
```bash
mongodump --archive | curl -X PUT --upload-file "-" http://transfer.firebee.com.br:3210/backup.archive
```

---

## Shell Alias / Convenience Function

Add to `~/.bashrc` or `~/.zshrc` to simplify usage:

```bash
transfer() {
  if [ -z "$1" ]; then
    echo "Usage: transfer <file>"
    return 1
  fi
  curl --upload-file "$1" "http://transfer.firebee.com.br:3210/$(basename "$1")"
  echo  # newline after the URL
}
```

After reloading the shell (`source ~/.bashrc`), usage becomes simple:

```bash
transfer report.pdf
transfer backup.tar.gz
```

---

## Sharing Files with Others

The URL returned by the upload can be shared with anyone who has access to the network where the service is exposed. Just send the URL — no authentication is required to download.

```
http://transfer.firebee.com.br:3210/G2k6d6aMSl/teste.txt
                                    ^^^^^^^^^
                                    Randomly generated unique ID
```

> **Security note:** the URL is the only "secret" for the file. Anyone with the URL can download it. Do not share URLs of sensitive files in public channels.

---

## Service Limits

| Parameter | Value |
|---|---|
| Maximum file size | **5 GB** |
| Retention period | **14 days** (files are deleted automatically) |
| Rate limit | **10 requests** per time window |
| Storage provider | Local (VPS server disk) |

Files are automatically deleted after 14 days. If you need longer persistence, re-upload before expiry.

---

## Troubleshooting

### The URL does not open in the browser
Make sure the `:3210` port is included in the URL. The service does not respond at `http://transfer.firebee.com.br` without the port.

### Rate limit error
If you receive an HTTP 429 error, wait a moment and try again. The limit is 10 requests per window.

### Slow or interrupted upload
For large files, use the `--limit-rate` flag to control speed and avoid timeouts:
```bash
curl --upload-file ./large-file.tar.gz --limit-rate 10M http://transfer.firebee.com.br:3210
```

### Check upload progress
```bash
curl --upload-file ./file.tar.gz http://transfer.firebee.com.br:3210 --progress-bar
```

---

## Service Configuration (Reference)

The service runs via Docker Compose on the VPS:

```yaml
image: dutchcoders/transfer.sh:latest
port: 3210 → 8080 (internal)
storage: local volume /data
max upload: 5 GB
purge: 14 days
```

Configuration repository: `transfer-sh-firebee/docker-compose.yaml`
