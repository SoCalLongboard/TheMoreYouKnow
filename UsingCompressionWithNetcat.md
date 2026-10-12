# Using Compression with Netcat

Netcat (`nc`) moves raw bytes over a TCP or UDP connection. It has no built-in compression, but because it reads from stdin and writes to stdout, you can pipe data through a compressor on the sending side and a decompressor on the receiving side.

## Contents

- [Basic pattern](#basic-pattern)
- [Transferring directories with tar](#transferring-directories-with-tar)
- [Choosing a compressor](#choosing-a-compressor)
- [Disk images](#disk-images)
- [Tips and caveats](#tips-and-caveats)
- [Secure alternative: SSH](#secure-alternative-ssh)

## Basic pattern

Start the receiver first so it is listening before the sender connects.

```bash
# Receiver
nc -l 9000 | gunzip > file.txt        # some versions: nc -l -p 9000

# Sender
gzip -c file.txt | nc receiver_host 9000
```

The sender compresses to stdout and netcat ships the compressed bytes. The receiver decompresses them back into the original file.

## Transferring directories with tar

`tar` bundles a directory into a single stream, which makes it a natural fit for netcat.

```bash
# Receiver
nc -l 9000 | tar xzf -

# Sender
tar czf - /path/to/dir | nc receiver_host 9000
```

## Choosing a compressor

gzip is universal, but on a fast network it can become the bottleneck. Faster compressors keep the pipe full.

### zstd (usually the best tradeoff)

```bash
# Sender
tar cf - /path/to/dir | zstd -T0 | nc host 9000

# Receiver
nc -l 9000 | zstd -d | tar xf -
```

### lz4 (fastest, lighter compression)

```bash
# Sender
tar cf - dir | lz4 | nc host 9000

# Receiver
nc -l 9000 | lz4 -d | tar xf -
```

| Compressor | Speed | Ratio | Best for |
|------------|-------|-------|----------|
| `lz4` | Fastest | Lowest | Fast LANs, CPU-limited hosts |
| `zstd` (low level) | Very fast | Good | General use |
| `gzip` | Moderate | Good | Maximum compatibility |
| `zstd -9`, `xz` | Slow | Highest | Slow or metered links |

## Disk images

```bash
# Sender
dd if=/dev/sda bs=4M | zstd | nc host 9000

# Receiver
nc -l 9000 | zstd -d | dd of=/dev/sdb bs=4M
```

Double-check the `of=` target. `dd` will overwrite whatever device you point it at.

## Tips and caveats

- **Both sides must match.** If the sender compresses with zstd, the receiver must decompress with zstd.
- **Skip already-compressed data.** JPEGs, MP4s, and zip files gain almost nothing and cost CPU time.
- **Pick by bottleneck.** On a slow link, heavier compression (`zstd -9`, `xz`) pays off. On a fast LAN, use lz4 or low-level zstd.
- **No encryption or integrity checking.** Netcat sends plaintext and does not verify the data. For important transfers, compare checksums (for example `sha256sum`) on both ends.
- **Netcat variants differ.** OpenBSD, traditional, and ncat behave differently. Traditional netcat needs `-p` to specify the listen port, and some versions need `-q 0` or `-N` on the sender so the connection closes at end of input.

## Secure alternative: SSH

If the data is sensitive or the network is untrusted, tunnel through SSH instead of netcat:

```bash
tar czf - dir | ssh host 'tar xzf - -C /dest'
```

SSH also has built-in compression via `ssh -C`.

## Summary

Netcat does not compress, but it composes well with tools that do. Pipe through `gzip`, `zstd`, or `lz4` on the sender, reverse it on the receiver, and match the compressor to your network speed. Use SSH when you need confidentiality.
