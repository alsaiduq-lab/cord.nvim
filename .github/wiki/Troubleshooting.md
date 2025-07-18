# 🔧 Troubleshooting

Running into issues with Cord? Here are some common problems and how to fix them.

## 🛠️ General Steps

Start here - these steps often resolve most issues:

1. Make sure Discord is running, restart Neovim, and check whether Cord is fully loaded.
2. Run `:Cord update` to check for server updates.
3. Double-check your [Discord Activity Privacy settings](https://github.com/vyfor/cord.nvim/assets/92883017/c0c8c410-e90e-425e-bf10-8b59f04f15ce) — Rich Presence needs permission to show.
4. Add `log_level = vim.log.levels.TRACE` to your `cord.setup()` to see debug logs via `:messages`. Don't forget to switch back to a lower level after (e.g. `WARN` or `OFF`).
5. Run `:checkhealth cord` for a quick config check.
6. Make sure the Discord IPC pipe exists:

   * **Windows:** 
     ```pwsh
     Test-Path \\.\pipe\discord-ipc-0
     ```
   * **Linux/macOS:**
     ```sh
     find /tmp ${XDG_RUNTIME_DIR:+$XDG_RUNTIME_DIR} ${TMPDIR:+$TMPDIR} ${TMP:+$TMP} ${TEMP:+$TEMP} -type s -name 'discord-ipc-*' 2>/dev/null
     ```

   If you get nothing or "False", the pipe may not be available.

## 🎛️ No Buttons in Rich Presence

You might not be seeing buttons in your own Rich Presence. This is due to a client-sided bug on the app.

But there is actually a way to see them, join a voice channel and hover over your name. Buttons should show up in the tooltip.

## ⏱️ Rich Presence Timer Stuck at 00:00

This usually means your system clock is off.

Make sure your date, time, and timezone are correct and synced (using automatic time sync is best).

## 💻 Special Environments

### 🌐 Using Discord in a Browser

Cord doesn't support browser Discord out of the box. Use [arrpc](https://github.com/OpenAsar/arrpc) as a bridge. Follow its instructions closely.

>[!IMPORTANT]
> arrpc has been left unmaintained for quite some time.

### 🐧 Running inside WSL

WSL doesn't expose Windows named pipes by default, which Discord needs. To work around that, use `socat` and `npiperelay`.
This method is based on [this gist](https://gist.github.com/mousebyte/af45cbecaf0028ea78d0c882c477644a#aliasing-nvim).

1. **Install `socat`** in WSL: `sudo apt install socat`
2. **Get `npiperelay.exe`** from [here](https://github.com/jstarks/npiperelay/releases) and place it in a path accessible from WSL, preferably add it to PATH.
3. **Add this `nvim` alias** in your `.bashrc`, `.zshrc`, etc.:

   ```sh
   nvim() {
       if ! pidof socat > /dev/null 2>&1; then
           [ -e /tmp/discord-ipc-0 ] && rm -f /tmp/discord-ipc-0
           socat UNIX-LISTEN:/tmp/discord-ipc-0,fork \
               EXEC:"npiperelay.exe //./pipe/discord-ipc-0" 2>/dev/null &
       fi

       if [ $# -eq 0 ]; then
           command nvim
       else
           command nvim "$@"
       fi
   }
   ```

   > Update the path to `npiperelay.exe` if needed.
   > Always launch Neovim using this alias in WSL.

### 🖥️ Remote Server (SSH)

You can forward the Discord IPC socket over SSH. This [article explains how](https://carlosbecker.com/posts/discord-rpc-ssh/).

## ❓ Still Having Trouble?

If nothing above works:

1. Check the [FAQ](./FAQ.md) for other common questions.
2. Look through [existing GitHub issues](https://github.com/vyfor/cord.nvim/issues).
3. If it seems like a bug, [open a new issue](https://github.com/vyfor/cord.nvim/issues/new/choose).
   Be clear and include as much detail as possible.

You can also ask in Discussions or Discord if you're unsure.
