#zed

## settings

```json
// Zed settings
//
// For information on how to configure Zed, see the Zed
// documentation: https://zed.dev/docs/configuring-zed
//
// To see all of Zed's default settings without changing your
// custom settings, run `zed: open default settings` from the // command palette (cmd-shift-p / ctrl-shift-p)
{
  "assistant": {
    "default_model": {
      "provider": "zed.dev",
      "model": "claude-3-7-sonnet-latest"
    },
    "version": "2"
  },
  "icon_theme": "Material Icon Theme",
  "base_keymap": "VSCode",
  "vim_mode": true,
  "ui_font_size": 16,
  "buffer_font_size": 15.0,
  "theme": {
    "mode": "system",
    "light": "One Light",
    "dark": "One Dark"
  },
  "buffer_font_features": {
    "calt": false
  },
  "cursor_blink": false,
  "relative_line_numbers": true,
  "project_panel": {
    "dock": "right",
    "auto_fold_dirs": false,
    "entry_spacing": "standard",
    "indent_guides": {
      "show": "never"
    }
  },
  "git_panel": {
    "dock": "right"
  },
  "languages": {
    "Markdown": { "format_on_save": "on" },
    "Dart": { "always_treat_brackets_as_autoclosed": false }
  },
  "diagnostics": {
    "include_warnings": false,
    "inline": {
      "enabled": true,
      "update_debounce_ms": 150,
      "padding": 2,
      "min_column": 0,
      "max_severity": null
    }
  },
  "task": { "show_status_indicator": false }
}
```

## tasks

```json
// Static tasks configuration.
//
// Example:
[
  {
    "label": "Example task",
    "command": "for i in {1..5}; do echo \"Hello $i/5\"; sleep 1; done",
    //"args": [],
    // Env overrides for the command, will be appended to the terminal's environment from the settings.
    "env": { "foo": "bar" },
    // Current working directory to spawn the command into, defaults to current project root.
    //"cwd": "/path/to/working/directory",
    // Whether to use a new terminal tab or reuse the existing one to spawn the process, defaults to `false`.
    "use_new_terminal": false,
    // Whether to allow multiple instances of the same task to be run, or rather wait for the existing ones to finish, defaults to `false`.
    "allow_concurrent_runs": false,
    // What to do with the terminal pane and tab, after the command was started:
    // * `always` — always show the task's pane, and focus the corresponding tab in it (default)
    // * `no_focus` — always show the task's pane, add the task's tab in it, but don't focus it
    // * `never` — do not alter focus, but still add/reuse the task's tab in its pane
    "reveal": "always",
    // Where to place the task's terminal item after starting the task:
    // * `dock` — in the terminal dock, "regular" terminal items' place (default)
    // * `center` — in the central pane group, "main" editor area
    "reveal_target": "dock",
    // What to do with the terminal pane and tab, after the command had finished:
    // * `never` — Do nothing when the command finishes (default)
    // * `always` — always hide the terminal tab, hide the pane also if it was the last tab in it
    // * `on_success` — hide the terminal tab on task success only, otherwise behaves similar to `always`
    "hide": "never",
    // Which shell to use when running a task inside the terminal.
    // May take 3 values:
    // 1. (default) Use the system's default terminal configuration in /etc/passwd
    //      "shell": "system"
    // 2. A program:
    //      "shell": {
    //        "program": "sh"
    //      }
    // 3. A program with arguments:
    //     "shell": {
    //         "with_arguments": {
    //           "program": "/bin/bash",
    //           "args": ["--login"]
    //         }
    //     }
    "shell": "system"
  },
  {
    "label": "dart run $ZED_FILENAME",
    "command": "dart",
    "args": ["$ZED_FILE"],
    "env": {},
    "use_new_terminal": true,
    "allow_concurrent_runs": false,
    "show_summary": false,
    "show_command": false
  },
  {
    "label": "dart test all",
    "command": "dart test",
    "args": ["-r github"],
    "env": {},
    "use_new_terminal": true,
    "allow_concurrent_runs": false,
    "show_summary": false,
    "show_command": false
  },
  {
    "label": "dart test $ZED_FILENAME",
    "command": "dart test",
    "args": ["-r github $ZED_FILE"],
    "env": {},
    "use_new_terminal": true,
    "allow_concurrent_runs": false,
    "show_summary": false,
    "show_command": false
  },
  {
    "label": "Open Pixel 9 Pro",
    "command": "emulator -avd Pixel_9_Pro",
    "args": [],
    "env": {},
    "use_new_terminal": false,
    "allow_concurrent_runs": false,
    "reveal": "no_focus",
    "tags": ["Flutter"]
  },
  {
    "label": "start lazygit",
    "command": "lazygit -p $ZED_WORKTREE_ROOT",
    "hide": "always"
  }
]

```

## keymap

```json
// Zed keymap
//
// For information on binding keys, see the Zed
// documentation: https://zed.dev/docs/key-bindings
//
// To see the default key bindings run `zed: open default keymap`
// from the command palette.
[
  {
    "context": "Editor && vim_mode == insert",
    "bindings": {
      "j k": ["workspace::SendKeystrokes", "escape"]
    }
  },
  {
    "context": "ProjectPanel && not_editing",
    "bindings": {
      "a": "project_panel::NewFile"
    }
  },
  {
    "context": "ProjectPanel && not_editing",
    "bindings": {
      "A": "project_panel::NewDirectory"
    }
  },
  {
    "context": "Workspace",
    "bindings": {
      "cmd-g": [
        "task::Spawn",
        { "task_name": "start lazygit", "reveal_target": "center" }
      ]
    }
  },
  {
    "context": "Workspace",
    "bindings": {
      "shift shift": "file_finder::Toggle"
    }
  },
  {
    "context": "Workspace",
    "bindings": {
      "cmd-t": ["task::Spawn", {}]
    }
  }
]

```


## Zed Snippets
```json
{
  "Given When Then": {
    "prefix": "tdd",
    "body": ["test('$1', () {\n// Given\n\n// When\n\n// Then\n});", "$0"],
    "description": "create test method and Given When Then comments."
  }
}

```
