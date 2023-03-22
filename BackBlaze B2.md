Some notes on using the service which, as of this writing, offers

- The lowest fees: $0.005 GB/Month
- 10GB for free
- Free data egress [through CloudFlare's CDN and Edge Network](https://www.backblaze.com/blog/backblaze-and-cloudflare-partner-to-provide-free-data-transfer/) which is something I'll explore soon.
- [A CLI client](https://www.backblaze.com/b2/docs/quick_command_line.html) 

### Installation

```bash
# On macOS. The command is `b2`
brew install b2-tools

# On Ubuntu. The command is `backblaze-b2`
apt install backblaze-b2
```

### Setup

Log into your profile and set up an "Application Key". It has two components: an ID and a Value. That's what you'll be configuring the `b2` command with.

```
b2 authorize-account {keyID} {applicationKey}
```

### Usage

```bash
# List all buckets
b2 list-buckets

# Upload a file
b2 upload-file my-bucket/ riffraff.tgz

# Upload a file and call it something else
b2 upload-file my-bucket/ riffraff.tgz foobar.tgz
```

