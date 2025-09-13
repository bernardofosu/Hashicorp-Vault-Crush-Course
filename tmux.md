# 🖥️ Tmux Guide

## ✅ Starting Tmux

To start a new tmux session, use:
```bash
tmux new -s session_name
```

## 📜 List Sessions

To see the available sessions:
```bash
tmux ls
```

## 🔌 Attach to a Session

If a session exists, you can attach to it:
```bash
tmux attach -t session_name
```

## 💡 Split Windows

- **Split Vertically**:
  ```bash
  Ctrl + b %
  ```
- **Split Horizontally**:
  ```bash
  Ctrl + b "
  ```

## 🚀 Navigate Between Panes

- **Move to the Next Pane**:
  ```bash
  Ctrl + b o
  ```
- **Navigate with Arrow Keys**:
  ```bash
  Ctrl + b (Up/Down/Left/Right)
  ```

## 🔎 Resize Panes

You can resize panes using the following commands:
```bash
Ctrl + b : resize-pane -D 5  # Down
Ctrl + b : resize-pane -U 5  # Up
Ctrl + b : resize-pane -L 5  # Left
Ctrl + b : resize-pane -R 5  # Right
```

## 🗂️ Manage Windows

- **Create a New Window**:
  ```bash
  Ctrl + b c
  ```
- **Switch to Next Window**:
  ```bash
  Ctrl + b n
  ```
- **Switch to Previous Window**:
  ```bash
  Ctrl + b p
  ```
- **List All Windows**:
  ```bash
  Ctrl + b w
  ```

## ❎ Close and Exit

- **Close a Pane**:
  ```bash
  Ctrl + d
  ```
- **Kill a Session**:
  ```bash
  Ctrl + b : kill-session
  ```
- **Detach from a Session**:
  ```bash
  Ctrl + b d
  ```

## 🌿 Additional Tips

- If you see the error `sessions should be nested with care`, it means you're already in a tmux session.
- You can force reattach using:
  ```bash
  tmux attach -t session_name
  ```
- To start fresh, kill the session using:
  ```bash
  tmux kill-session -t session_name
  ```

