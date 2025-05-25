## File System Navigation & Management

| Old    | Modern                                                                            | Language |
| ------ | --------------------------------------------------------------------------------- | -------- |
| `cat`  | [bat](https://github.com/sharkdp/bat)                                             | Rust     |
| `cd`   | [zoxide](https://github.com/ajeetdsouza/zoxide)                                   | Rust     |
| `du`   | [dust](https://github.com/bootandy/dust), [dua](https://github.com/Byron/dua-cli) | Rust     |
| `find` | [fd](https://github.com/sharkdp/fd)                                               | Rust     |
| `ls`   | [eza](https://github.com/eza-community/eza), [lsd](https://github.com/lsd-rs/lsd) | Rust     |
| `rm`   | [trash](https://github.com/sindresorhus/trash-cli)                                | Go       |
| `time` | [hyperfine](https://github.com/sharkdp/hyperfine)                                 | Rust     |

## System Monitoring

| Old          | Modern                                                                                            | Language         |
| ------------ | ------------------------------------------------------------------------------------------------- | ---------------- |
| `top`/`htop` | [bottom (btm)](https://github.com/ClementTsang/bottom), [gtop](https://github.com/aksakalli/gtop) | Rust, JavaScript |
| `ps`         | [procs](https://github.com/dalance/procs)                                                         | Rust             |

## Text Processing & Search

| Old           | Modern                                                                                                                                      | Language     |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------- | ------------ |
| `awk`         | [angle-grinder](https://github.com/rcoh/angle-grinder)                                                                                      | Rust         |
| `awk`/`csv`   | [xsv](https://github.com/BurntSushi/xsv)                                                                                                    | Rust         |
| `awk`/`cut`   | [choose](https://github.com/theryangeary/choose)                                                                                            | Rust         |
| `awk`/`cut`   | [hck](https://github.com/sstadick/hck)                                                                                                      | Rust         |
| `awk`/`sed`   | [miller (mlr)](https://github.com/johnkerl/miller)                                                                                          | Go           |
| `diff`        | [delta](https://github.com/dandavison/delta)                                                                                                | Rust         |
| `grep`        | [rg](https://github.com/BurntSushi/ripgrep), [ag](https://github.com/ggreer/the_silver_searcher), [ugrep](https://github.com/Genivia/ugrep) | Rust, C, C++ |
| `sed`         | [sd](https://github.com/chmln/sd)                                                                                                           | Rust         |
| JSON tools    | [jq](https://github.com/stedolan/jq), [jless](https://github.com/PaulJuliusMartinez/jless)                                                  | C, Rust      |
| YAML tools    | [yq](https://github.com/mikefarah/yq)                                                                                                       | Go           |
| Data selector | [dasel](https://github.com/TomWright/dasel)                                                                                                 | Go           |

## Network Tools

| Old              | Modern                                              | Language |
| ---------------- | --------------------------------------------------- | -------- |
| `dig`            | [doggo](https://github.com/mr-karan/doggo)          | Go       |
| `dig`/`nslookup` | [dog](https://github.com/ogham/dog)                 | Rust     |
| `whois`          | [whois.rs](https://github.com/utopiabound/rs-whois) | Rust     |
| `curl`           | [xh](https://github.com/ducaale/xh)                 | Rust     |
| `netstat`        | [bandwhich](https://github.com/imsnif/bandwhich)    | Rust     |
| `ping`           | [gping](https://github.com/orf/gping)               | Rust     |
| DNS monitoring   | [dnspeep](https://github.com/jvns/dnspeep)          | Rust     |
| `httpie`         | [xh](https://github.com/ducaale/xh)                 | Rust     |


## File & Directory Tools

| Old            | Modern                                   | Language |
| -------------- | ---------------------------------------- | -------- |
| `find`         | [fd-find](https://github.com/sharkdp/fd) | Rust     |
| `find`/Various | [fzf](https://github.com/junegunn/fzf)   | Go       |
| Rename tools   | [rnr](https://github.com/ismaelgv/rnr)   | Rust     |

## Text Viewing & Editing

| Old         | Modern                                      | Language |
| ----------- | ------------------------------------------- | -------- |
| `hexdump`   | [hexyl](https://github.com/sharkdp/hexyl)   | Rust     |
| Hex editors | [hx (hex)](https://github.com/sitkevij/hex) | Rust     |

## Other

| Old   | Modern                                      | Language |
| ---   | ---                                         | ---      |
| `git` | [gitui](https://github.com/gitui-org/gitui) | Rust     |

Note: You'll need to `ssh-add ~/.ssh/key_id` for `gitui` to push to remote on macOS

There's [`uutils/coreutils`](https://github.com/uutils/coreutils) which just replaces all GNU Coreutils with Rust reimplemntations 🤷‍♂️ Seems risky. They do note that "some options might be missing or different behavior might be experienced". I have a lot `bash` scripts that depend on stable implementations and won't be using this one (again, they do note that "differences with GNU are treated as bugs"). Maybe one day.

[`fselect`](https://github.com/jhspetersson/fselect) allows you to search your filesystem with SQL-like queries!

`starship` allows you to customize your `bash` (or other) prompts. Written in Rust and is advertized to be [_blazingly fast_](https://starship.rs/).
