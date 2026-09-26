# 06 · Vim Editor

> **Audience:** DevOps Engineers · Senior SRE · Senior Production Engineers
> **Goal:** Edit configs on any server confidently — Vim (or `vi`) is installed on virtually every Linux box, including minimal containers and rescue shells.

---

## Table of Contents

1. [Why Vim for SREs](#1-why-vim-for-sres)
2. [Modes — The Key Concept](#2-modes--the-key-concept)
3. [Opening Files](#3-opening-files)
4. [Save & Quit](#4-save--quit)
5. [Moving Around (Navigation)](#5-moving-around-navigation)
6. [Inserting Text](#6-inserting-text)
7. [Editing: Delete, Copy, Paste, Undo](#7-editing-delete-copy-paste-undo)
8. [Search & Replace](#8-search--replace)
9. [Visual Mode & Block Editing](#9-visual-mode--block-editing)
10. [Multiple Files, Buffers, Splits, Tabs](#10-multiple-files-buffers-splits-tabs)
11. [Useful Settings (`:set`)](#11-useful-settings-set)
12. [Registers, Macros & Marks](#12-registers-macros--marks)
13. [Running Shell Commands from Vim](#13-running-shell-commands-from-vim)
14. [Recommended `~/.vimrc` for DevOps](#14-recommended-vimrc-for-devops)
15. [Real-World Scenarios](#15-real-world-scenarios)
16. [Troubleshooting Vim](#16-troubleshooting-vim)
17. [Interview Questions](#17-interview-questions)
18. [Cheat Sheet](#18-cheat-sheet)

---

## 1. Why Vim for SREs

- Always available (`vi` is part of POSIX) — even in rescue mode.
- Works over slow SSH connections.
- Used by `visudo`, `crontab -e`, `kubectl edit`, `git commit`, `systemctl edit` (via `$EDITOR`).

```bash
export EDITOR=vim VISUAL=vim     # Make vim the default editor
vimtutor                          # 30-minute built-in tutorial ← do this once
```

---

## 2. Modes — The Key Concept

| Mode | How to enter | Purpose |
|------|--------------|---------|
| **Normal** | `Esc` | Move, delete, copy, commands (default) |
| **Insert** | `i`, `a`, `o` | Type text |
| **Visual** | `v`, `V`, `Ctrl+v` | Select text |
| **Command-line** | `:` | Save, quit, search/replace, settings |
| **Replace** | `R` | Overwrite characters |

> **Lost? Press `Esc` twice** → you're back in Normal mode.

**Grammar:** `[count] operator motion` → `d3w` = delete 3 words · `y$` = copy to end of line · `c2j` = change 2 lines down.

---

## 3. Opening Files

```bash
vim file.conf                 # Open / create
vim +25 file.conf             # Open at line 25
vim +/ERROR app.log           # Open at first "ERROR"
vim + file                    # Open at last line
vim -R file / view file       # Read-only
vim -O a.conf b.conf          # Side-by-side (vertical split)
vim -o a b                    # Horizontal split
vim -d old.conf new.conf      # Diff mode (vimdiff)
vim scp://user@host//etc/nginx/nginx.conf   # Edit remote file
sudo vim /etc/hosts           # Edit as root
sudoedit /etc/hosts           # Safer: edit copy with your settings
```

| Option | What it does |
|--------|--------------|
| `+N` | Open file at line N |
| `+/pattern` | Open at first pattern match |
| `-R` | Open read-only, prevent accidental writes |
| `-O` / `-o` | Split vertically / horizontally |
| `-d` | Diff two or more files |
| `-u NONE` | Start without vimrc, clean |
| `-r` | Recover from swap file after crash |
| `-b` | Binary mode, safe for binaries |

---

## 4. Save & Quit

| Command | What it does |
|---------|--------------|
| `:w` | Save the file |
| `:q` | Quit (fails if unsaved) |
| `:wq` / `:x` / `ZZ` | Save and quit |
| `:q!` / `ZQ` | Quit, discard all changes |
| `:w newname` | Save as a new file |
| `:w !sudo tee %` | Save when you forgot sudo |
| `:wa` / `:qa` / `:wqa` | Write / quit / both, all files |
| `:e!` | Reload file, discard unsaved changes |
| `:10,20w part.txt` | Save lines 10-20 to file |

---

## 5. Moving Around (Navigation)

### Basic

| Key | Moves |
|-----|-------|
| `h j k l` | Left, down, up, right |
| `w` / `b` | Next / previous word start |
| `e` | End of word |
| `0` / `^` / `$` | Line start / first char / line end |
| `gg` / `G` | First line / last line |
| `:42` or `42G` | Go to line 42 |
| `Ctrl+f` / `Ctrl+b` | Page down / page up |
| `Ctrl+d` / `Ctrl+u` | Half page down / up |
| `H` / `M` / `L` | Top / middle / bottom of screen |
| `%` | Jump to matching `( ) { } [ ]` |
| `{` / `}` | Previous / next paragraph (blank line) |
| `f<char>` / `t<char>` | Jump to / before char on line |
| `;` / `,` | Repeat last f/t forward / back |
| `*` / `#` | Search word under cursor fwd / back |
| `Ctrl+o` / `Ctrl+i` | Jump back / forward in history |
| `zz` | Center screen on cursor |
| `gd` | Go to local definition |

---

## 6. Inserting Text

| Key | Where insert starts |
|-----|---------------------|
| `i` | Before cursor |
| `a` | After cursor |
| `I` | Start of line |
| `A` | End of line ← very common |
| `o` | New line below |
| `O` | New line above |
| `s` | Delete char and insert |
| `S` / `cc` | Delete line and insert |
| `C` | Delete to end of line and insert |
| `R` | Replace mode (overwrite) |
| `r<char>` | Replace one character |

---

## 7. Editing: Delete, Copy, Paste, Undo

### Delete (cut)

| Key | Deletes |
|-----|---------|
| `x` / `X` | Char under / before cursor |
| `dw` | To start of next word |
| `diw` | Whole word under cursor |
| `dd` | Whole line |
| `5dd` | 5 lines |
| `D` / `d$` | To end of line |
| `d0` | To start of line |
| `dG` | To end of file |
| `dgg` | To start of file |
| `:10,20d` | Lines 10 to 20 |
| `:g/^#/d` | All comment lines |
| `:g/^$/d` | All empty lines |
| `di"` / `di(` / `di{` | Inside quotes / parens / braces |
| `da"` | Including the quotes |

### Change (delete + insert)

| Key | Changes |
|-----|---------|
| `cw` / `ciw` | Word |
| `ci"` | Text inside quotes ← edit config values |
| `cc` | Whole line |
| `c$` / `C` | To end of line |

### Copy (yank) & paste

| Key | Action |
|-----|--------|
| `yy` / `Y` | Copy line |
| `5yy` | Copy 5 lines |
| `yw` / `yiw` | Copy word |
| `y$` | Copy to end of line |
| `p` / `P` | Paste after / before cursor |
| `:10,20y` | Yank lines 10-20 |
| `:10,20m 30` | Move lines 10-20 after line 30 |
| `:10,20t 30` | Copy lines 10-20 after line 30 |

### Undo / redo / repeat

| Key | Action |
|-----|--------|
| `u` | Undo last change |
| `U` | Undo all changes on line |
| `Ctrl+r` | Redo |
| `.` | Repeat last change ← super powerful |
| `J` | Join line below to current |
| `>>` / `<<` | Indent / unindent line |
| `==` / `gg=G` | Auto-indent line / whole file |
| `~` | Toggle case |
| `gUU` / `guu` | Line to UPPER / lower |
| `Ctrl+a` / `Ctrl+x` | Increment / decrement number |

---

## 8. Search & Replace

### Search

| Command | What it does |
|---------|--------------|
| `/pattern` | Search forward |
| `?pattern` | Search backward |
| `n` / `N` | Next / previous match |
| `/\cpattern` | Case-insensitive search once |
| `/\<word\>` | Whole word only |
| `:noh` | Clear search highlighting |

### Substitute — `:[range]s/old/new/[flags]`

```vim
:s/foo/bar/            " First match on current line
:s/foo/bar/g           " All matches on current line
:%s/foo/bar/g          " Whole file
:%s/foo/bar/gc         " Whole file, confirm each (y/n/a/q)
:%s/foo/bar/gi         " Case-insensitive
:10,20s/foo/bar/g      " Lines 10-20
:'<,'>s/foo/bar/g      " Visual selection
:%s#/usr/local#/opt#g  " Alternate delimiter for paths
:%s/\s\+$//e           " Remove trailing whitespace
:%s/^/# /              " Comment every line
:%s/^# //              " Uncomment
:%s/\r//g              " Remove Windows ^M
:%s/old/new/gn         " Count matches only, no replace
:g/pattern/d           " Delete lines matching
:v/pattern/d           " Delete lines NOT matching (keep only matches)
:g/ERROR/p             " Print all matching lines
```

| Flag | What it does |
|------|--------------|
| `g` | Replace all matches in line |
| `c` | Confirm before each replacement |
| `i` / `I` | Ignore case / match case |
| `e` | No error if pattern missing |
| `n` | Count matches, don't replace |
| `%` (range) | Apply to every line in file |

---

## 9. Visual Mode & Block Editing

| Key | Selects |
|-----|---------|
| `v` | Character-wise |
| `V` | Line-wise |
| `Ctrl+v` | Block (column) ← comment many lines |
| `gv` | Reselect last selection |
| `o` | Jump to other end of selection |

After selecting: `d` delete · `y` copy · `c` change · `>` / `<` indent · `u`/`U` case · `:` command on selection.

**Comment out lines 10–20 (YAML/shell):**
```
Ctrl+v → select down with j → Shift+I → type "# " → Esc
```

**Uncomment:**
```
Ctrl+v → select the "# " columns → d
```

---

## 10. Multiple Files, Buffers, Splits, Tabs

| Command | What it does |
|---------|--------------|
| `:e file` | Open another file |
| `:ls` | List open buffers |
| `:bn` / `:bp` | Next / previous buffer |
| `:b 2` | Go to buffer 2 |
| `:bd` | Close buffer |
| `:sp file` | Horizontal split |
| `:vsp file` | Vertical split |
| `Ctrl+w w` | Cycle between windows |
| `Ctrl+w h/j/k/l` | Move to window in direction |
| `Ctrl+w q` | Close window |
| `Ctrl+w =` | Equalize split sizes |
| `:tabnew file` | Open in new tab |
| `gt` / `gT` | Next / previous tab |
| `:r file` | Insert file contents below cursor |
| `:r !cmd` | Insert command output below cursor |

### vimdiff

```bash
vimdiff prod.conf staging.conf
```

| Key | What it does |
|-----|--------------|
| `]c` / `[c` | Next / previous difference |
| `do` | Obtain change from other file |
| `dp` | Put change into other file |
| `:diffupdate` | Rescan both files for differences |

---

## 11. Useful Settings (`:set`)

| Setting | What it does |
|---------|--------------|
| `:set nu` / `:set nonu` | Show / hide line numbers |
| `:set rnu` | Relative line numbers |
| `:set paste` / `:set nopaste` | Paste without auto-indent mess ← vital |
| `:set list` | Show tabs and line ends |
| `:set hlsearch` | Highlight all search matches |
| `:set incsearch` | Highlight while typing search |
| `:set ignorecase smartcase` | Smart case-insensitive search |
| `:set expandtab` | Insert spaces instead of tabs |
| `:set tabstop=2 shiftwidth=2` | Tab and indent width |
| `:set ff=unix` | Convert line endings to Unix |
| `:set ff?` | Show current file format |
| `:set wrap` / `:set nowrap` | Wrap / don't wrap long lines |
| `:syntax on` | Enable syntax highlighting |
| `:set ft=yaml` | Force file type (highlighting) |
| `:set cursorline` | Highlight the current line |

> **YAML / Kubernetes golden rule:** spaces only, 2-space indent → `:set et ts=2 sw=2`. Tabs break YAML.

---

## 12. Registers, Macros & Marks

### Registers

| Register | Contains |
|----------|----------|
| `""` | Default (last yank/delete) |
| `"0` | Last yank only |
| `"a`–`"z` | Named — `"ayy` copy line to a, `"ap` paste |
| `"+` | System clipboard (`"+y`) |
| `:reg` | Show all registers |

### Macros — automate repeated edits

```
qa          start recording into register a
...edits... (e.g. I- <Esc> j)
q           stop recording
@a          play once
50@a        play 50 times
@@          replay last macro
```

### Marks

| Key | Action |
|-----|--------|
| `ma` | Set mark a |
| `'a` | Jump to line of mark a |
| `` `a `` | Jump to exact position |
| `''` | Jump back to previous spot |

---

## 13. Running Shell Commands from Vim

```vim
:!ls -l                    " Run command, show output
:!nginx -t                 " Test config without leaving vim
:r !date                   " Insert date into file
:%!jq .                    " Pretty-format whole JSON file
:%!sort -u                 " Sort + dedupe whole file
:'<,'>!column -t           " Align selection into columns
:w !sudo tee % >/dev/null  " Save file as root
:sh                        " Drop to shell; exit to return
Ctrl+z / fg                " Suspend vim / return to it
```

---

## 14. Recommended `~/.vimrc` for DevOps

```vim
syntax on
filetype plugin indent on
set number
set ruler
set showcmd
set hlsearch incsearch ignorecase smartcase
set expandtab tabstop=2 shiftwidth=2 softtabstop=2
set autoindent
set backspace=indent,eol,start
set encoding=utf-8
set list listchars=tab:»·,trail:·
set nowrap
set mouse=
autocmd FileType yaml,yml setlocal ts=2 sw=2 et
autocmd FileType python setlocal ts=4 sw=4 et
autocmd FileType make setlocal noexpandtab
" Strip trailing whitespace on save
autocmd BufWritePre * :%s/\s\+$//e
```

---

## 15. Real-World Scenarios

**Change a config value fast**
```bash
vim +/^PermitRootLogin /etc/ssh/sshd_config   # opens on the line
# cw → type "no" → Esc → :wq
sshd -t && systemctl reload sshd
```

**Edit a Kubernetes resource live**
```bash
KUBE_EDITOR=vim kubectl edit deploy/api -n prod
# /replicas → Ctrl+a to increment → :wq
```

**Clean a pasted YAML that got auto-indented**
```
:set paste → i → paste → Esc → :set nopaste
```

**Find all ERRORs in a log with context**
```bash
vim -R app.log
:g/ERROR/p        " list matching lines
:vimgrep /ERROR/ % | copen   " quickfix list, jump through
```

**Fix "bad interpreter: /bin/bash^M"**
```
:set ff=unix → :wq
```

---

## 16. Troubleshooting Vim

| Problem | Fix |
|---------|-----|
| Screen frozen | You pressed `Ctrl+s` → press `Ctrl+q` |
| Can't quit | `Esc` then `:q!` |
| "E325: ATTENTION swap file exists" | Someone else editing, or crash. `R` recover, `D` delete swap (`.file.swp`) |
| "E45: readonly option is set" | `:w !sudo tee %` or `:wq!` if you own it |
| Weird characters on paste | Use `:set paste` first |
| Recording `qa` by accident | Press `q` to stop |
| Stuck in `Ex` mode | Type `visual` and Enter |
| Arrow keys print A B C D | Use `vim` not `vi`, or `set nocompatible` |

---

## 17. Interview Questions

1. **How to exit Vim without saving?** `Esc` → `:q!`
2. **Replace all occurrences in file?** `:%s/old/new/g`
3. **Delete lines 5–15?** `:5,15d`
4. **Delete all commented lines?** `:g/^#/d`
5. **Saved a root file without sudo?** `:w !sudo tee %`
6. **Go to line 120?** `:120` or `120G`
7. **What is a swap file?** `.file.swp` holding unsaved edits for crash recovery / edit lock.
8. **Why use `visudo` instead of vim on `/etc/sudoers`?** Syntax check before save — prevents locking out sudo.

---

## 18. Cheat Sheet

```text
MODES     Esc normal | i a I A o O insert | v V Ctrl+v visual | : command
SAVE/QUIT :w :q :wq :x ZZ :q! :w !sudo tee %
MOVE      h j k l  w b e  0 ^ $  gg G :N  Ctrl+f/b  %  *  Ctrl+o
EDIT      x dd 5dd D dw diw ci" cc C  yy p P  u Ctrl+r  .  J  >> <<
SEARCH    /pat ?pat n N  :noh  :%s/a/b/gc  :g/pat/d  :v/pat/d
VISUAL    Ctrl+v → I# → Esc  (block comment)
FILES     :e :sp :vsp Ctrl+w w  :ls :bn  vimdiff ]c do dp
SETTINGS  :set nu paste list et ts=2 sw=2 ff=unix
SHELL     :!cmd  :r !cmd  :%!jq .
MACRO     qa ... q  @a  50@a
```
