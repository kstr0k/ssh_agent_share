# `ssh_agent_share`

## _Share `ssh-agent` credentials across shell sessions_

## Usage

```
ssh_agent_share [OPTION]...

  --help           print this help message
  --[no-]-inherit  use/ignore SSH_AUTH_SOCK from environment
  --lockwait W     seconds to wait for lock (default: 1.5)
  --timeout T      default expiry time for all identities (3 days in secs)
  --dir PATH       agent credential cache path (~/.ssh/.ssh_agent_share/)
  --host HOST      override default hostname

Example:

  eval >/dev/null "$(ssh_agent_share --lockwait 3)"
  # optimized
  [ -w "${SSH_AUTH_SOCK:-} ]" ||
    eval >/dev/null "$(exec ssh_agent_share --lockwait 3)"
```

## Operation

This program caches `ssh-agent` output (`SSH_AUTH_SOCK=`, `SSH_AGENT_PID=`) so that credentials are available across shell sessions. If `--inherit` is active and the environment contains a valid `SSH_AUTH_SOCK`, the program exits (there's nothing to configure). Otherwise, it:
- checks for a valid cache
- starts a new agent if necessary, caching the output
- sends the cached credentials to stdout (which may be `eval`'d by `.bashrc` or other shell scripts).

Access to the cache file is protected by a lock. This avoids race conditions when starting multiple shell sessions (e.g. via [`tmux resurrect`](https://github.com/tmux-plugins/tmux-resurrect)). The program tries to acquire the lock for `--lockwait` seconds, then gives up (to avoid hangs).

### Performance

This program is implemented in `perl`. On the one hand, `perl` provides integrated, cross-platform `flock` and timeout facilities; on the other, while the interpreter startup cost is negligible, the same cannot be said about various imported modules (in particular, this is why I've eschewed using `Getopt::Long` and `pod2usage`, and why the optimized usage above doesn't call the program at all if a valid `SSH_AUTH_SOCK` is present).

It's possible to achieve the same functionality in `.bashrc` using `timeout` (from GNU `coreutils`) and `flock` from `util-linux` (or [equivalents](https://github.com/discoteq/flock)). This will reduce portability across platforms / shells, but turns out to be a bit faster (even though external executables need to be called).

## See also

[keychain](https://github.com/funtoo/keychain)

## Copyright

Alin Mr <almr.oss@outlook.com> / MIT license.
