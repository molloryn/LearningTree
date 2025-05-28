# AHK

## Principle

### Problems

#### 系统窗口没有作用

需要右键`.ahk`在管理员身份下运行

### Hotkeys

It is a key or key combination that the person at the keyboard presses to trigger some actions.

```ahk
^j::
{
    Send "My First Script"
}
```

`^j::` is the hotkey.

`^` means `Ctrl`,

`j` is the letter `J`.

Anything to the left of `::` are the keys you need to press.

`Send "My First Script"` is how you send keystrokes.

`Send` is the function, anything after the space inside the quotes will be typed.

`{` and `}` marks the start and end of the hotkey.

### Hotstrings

`Hotstrings` are mainly used to expand abbreviations as you type them (auto-replace), they can also be used to launch any scripted action. For example:

`::ftw::Free the whales`

the hotstring will convert your typed `"ftw"` into `"Free the whales"`.

## Syntax

[Quick Reference | AutoHotkey v2](https://www.autohotkey.com/docs/v2/)

### Symbol Map

[Send - Syntax & Usage | AutoHotkey v2](https://www.autohotkey.com/docs/v2/lib/Send.htm)

| Symbol | Description                                                                                          | Remark |
| ------ | ---------------------------------------------------------------------------------------------------- | ------ |
| `#`    | `Win` (Windows logo key)                                                                             |        |
| `!`    | `Alt`                                                                                                |        |
| `^`    | `Ctrl`                                                                                               |        |
| `+`    | `Shift`                                                                                              |        |
| `&`    | An ampersand may be used between any two keys or mouse buttons to combine them into a custom hotkey. |        |

### Functions

|   |   |   |
|---|---|---|
|   |   |   |
|   |   |   |
|   |   |   |
|   |   |   |
