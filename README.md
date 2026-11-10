# claude-session-copy

Copy a [Claude Code](https://claude.com/claude-code) session, and optionally its memory
notes, to another folder's project, so you can resume it from there.

Claude Code stores sessions per folder, so a session started in one folder doesn't show up
in another. This script copies one across. The original stays in place, and nothing is
overwritten.

> **Unofficial.** This relies on how Claude Code stores sessions today
> (`~/.claude/projects/<encoded-path>/`), which isn't a public interface and may change.

## Install

Requires bash and GNU coreutils and findutils (written and tested on Linux).

```bash
install -m 755 claude-session-copy ~/.local/bin/
```

Make sure `~/.local/bin` is on your `PATH`.

## Usage

Find the session ID, for example in the file names under `~/.claude/projects/`. Exit that
session first, then:

```bash
claude-session-copy SESSION_ID              # copy to your home folder's project
claude-session-copy SESSION_ID /some/folder # copy to another folder's project
claude-session-copy --dry-run SESSION_ID    # show the plan and change nothing
claude-session-copy --no-memory SESSION_ID  # copy only the conversation
claude-session-copy --memory SESSION_ID     # also copy memory notes into your home folder's project
```

It prints the command to resume the session, for example:

```bash
cd /some/folder && claude --resume SESSION_ID
```

## What gets copied

- The conversation (`SESSION_ID.jsonl`) and its session folder, if there is one
- Memory notes from the source project, unless you pass `--no-memory`. A note the target
  already has is kept as it is, and new notes get their index line added to `MEMORY.md`.
- **Except into your home folder (the default target) or a folder above it:** notes there
  load in every session you start from `~`, so they're skipped unless you pass `--memory`.
  Check the plan with `--dry-run` first; a note can hold anything, including text an
  untrusted session planted.

Files are copied under a temporary name and then linked into place, so a file that appears
in the target while the script runs (from another session or a second run) is never
replaced. The script stops instead.

## Limits

- **One extra copy per session:** if a session already exists in two projects, the script
  stops rather than guess which to use. Delete the extra copy first.
- **Exit the session first:** it warns if the session changed in the last minute, which
  usually means it's still open.
- **Conversation history keeps old paths:** the transcript still mentions the original folder.
- **Some folders share a project:** Claude Code replaces every non-alphanumeric character in
  the path with `-`, so `/a/b.c` and `/a/b-c` map to the same project folder.
- **Wider access:** resuming from a broader folder such as `~` gives Claude that folder as
  its working area.
- **Only memory notes and session data are copied:** project settings and `CLAUDE.md` files
  aren't. The target folder's own apply instead.

Session files can contain anything said in a conversation, including secrets and private
details. Don't commit or share them.

## License

[MIT](LICENSE)
