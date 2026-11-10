# otp

A single-file command-line tool that prints the current 6-digit 2FA (TOTP) code for a saved secret. No installs, no UI.

## Requirements

Bash, `openssl`, `xxd`, and `awk`. These come with macOS and most Linux distributions.

## Install

```sh
cp otp /usr/local/bin/otp   # or any directory on your PATH
chmod +x /usr/local/bin/otp
```

## Usage

```sh
otp -set "JBSWY3DPEHPK3PXP"                       # save a secret (replaces the previous one)
otp -set "otpauth://totp/Site:me?secret=...&..."  # or paste the URI from a QR decoder
otp -set                                          # prompts for the secret with hidden input
otp                                               # print the current code
otp -c                                            # print and copy to clipboard
otp -h                                            # help
```

Example:

```
$ otp
537372
valid for 9s
```

The code goes to stdout and the countdown goes to stderr, so `$(otp)` and pipes work:

```sh
echo "$(otp)" | pbcopy
```

If fewer than 5 seconds remain in the current 30-second window, `otp` waits for the next window so you don't type a code that expires mid-entry.

## Where the secret is stored

`~/.config/otp/secret`, mode 600, inside a directory with mode 700. Each `-set` replaces the file atomically. An invalid secret is rejected before anything is written, so the existing secret is kept.

## Supported formats

- Base32 secrets. Spaces, dashes, and `=` padding are ignored, and case doesn't matter.
- `otpauth://totp/...` URIs. The `secret` parameter is used.
- Standard TOTP only: SHA-1, 6 digits, 30-second period. A URI that asks for anything else is rejected with an error instead of producing wrong codes. Almost all services use these defaults.

## Security notes

- The secret is stored in plaintext. Anyone who can read your home directory can generate your 2FA codes, which means this tool removes the second factor for that machine. Use it only where that tradeoff is acceptable.
- `otp -set "SECRET"` puts the secret in your shell history. Use `otp -set` with no argument to get the hidden prompt instead.

## Troubleshooting

- `no secret saved; run: otp -set "SECRET"`: nothing has been saved yet. Run `-set` first.
- `secret is not valid base32`: check the secret for typos. Only letters A–Z and digits 2–7 are valid.
- `unsupported algorithm/digits/period`: the service uses non-default TOTP settings, which this tool does not support.
- `no clipboard tool (pbcopy/xclip)`: `-c` needs `pbcopy` (macOS) or `xclip` (Linux).
