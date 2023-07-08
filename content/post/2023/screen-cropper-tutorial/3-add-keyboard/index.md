---
title: "Add Keyboard"
slug: "add-keyboard"
tags: ["Go"]
categories: ["用Go製作截圖工具"]
description: 
date: "2023-06-29T08:24:16+08:00"
lastmod: "2023-06-29T08:24:16+08:00"
image: 
license: "CC BY-NC-ND"
hidden: false
comments: true
draft: false
---

## 建立 keyboard 的 package

這個 package 用來監聽鍵盤事件，當按下特定的鍵盤按鍵時，會觸發一個 callback function。

```go
// pkg/keyboard/keyboard.go
package keyboard

import (
    "strings"

    "github.com/moutend/go-hook/pkg/keyboard"
    "github.com/moutend/go-hook/pkg/types"
    "golang.org/x/exp/slices"
)

const (
    defaultEventChanSize = 256
    defaultListenerSize  = 256
    defaultPressedSize   = 256
)

var (
    // eventChan is a channel to receive keyboard events.
    eventChan chan types.KeyboardEvent
    // listeners is a map of hotkey listeners.
    listeners map[string]*hotkeyListener
    // pressed is a map of pressed keys.
    pressed [defaultPressedSize]bool
)

// init initializes variables.
func init() {
    reset()
}

// reset resets all variables.
func reset() {
    eventChan = make(chan types.KeyboardEvent, defaultEventChanSize)
    listeners = make(map[string]*hotkeyListener, defaultListenerSize)
    pressed = [defaultPressedSize]bool{}
}

// Hotkey represents a hotkey.
type Hotkey []types.VKCode

// hotkeyListener listens the hotkey.
type hotkeyListener struct {
    // hotkey is a hotkey to listen.
    hotkey Hotkey
    // callback is a callback function which is called when the hotkey is pressed.
    callback func()
}

// NewHotkey creates a new hotkey.
func NewHotkey(keys ...types.VKCode) Hotkey {
    slices.Sort(keys)
    return Hotkey(keys)
}

// String returns a string representation of the hotkey.
func (h Hotkey) String() string {
    b := strings.Builder{}
    for i, key := range h {
        if i > 0 {
            b.WriteString("+")
        }
        b.WriteString(key.String())
    }
    return b.String()
}

// Pressed returns true if the hotkey is pressed.
func (h Hotkey) Pressed() bool {
    for _, key := range h {
        if !pressed[key] {
            return false
        }
    }
    return true
}

// newHotkeyListener creates a new hotkey listener.
func newHotkeyListener(hotkey Hotkey, callback func()) *hotkeyListener {
    if callback == nil {
        callback = func() {}
    }
    return &hotkeyListener{
        hotkey:   hotkey,
        callback: callback,
    }
}

// Notify calls the callback function if the hotkey is pressed.
func (l *hotkeyListener) Notify() {
    l.mu.Lock()
    defer l.mu.Unlock()
    if !l.hotkey.Pressed() {
        l.justPressed = false
        return
    }
    if l.justPressed {
        return
    }
    l.justPressed = true
    l.callback()
}

// RegisterHotkey registers the hotkey.
func RegisterHotkey(hotkey Hotkey, callback func()) {
    listeners[hotkey.String()] = newHotkeyListener(hotkey, callback)
}

// UnregisterHotkey unregisters the hotkey.
func UnregisterHotkey(hotkey Hotkey) {
    delete(listeners, hotkey.String())
}

// handleEvent handles the keyboard event.
func handleEvent(event types.KeyboardEvent) {
    switch event.Message {
    case types.WM_KEYDOWN, types.WM_SYSKEYDOWN:
        pressed[event.VKCode] = true
        for _, listener := range listeners {
            listener.Notify()
        }
    case types.WM_KEYUP, types.WM_SYSKEYUP:
        pressed[event.VKCode] = false
    }
}

// Start starts the keyboard hook.
func Start() error {
    go func() {
        for event := range eventChan {
            handleEvent(event)
        }
    }()
    return keyboard.Install(nil, eventChan)
}

// Stop stops the keyboard hook.
func Stop() error {
    err := keyboard.Uninstall()
    if err != nil {
        return err
    }
    close(eventChan)
    reset()
    return nil
}

```

接著講解每個部份的設計：

### init()

`init()` 會在 package 被載入時執行，這裡會呼叫 `reset()` 來初始化變數。

```go
func init() {
    reset()
}
```

### reset()

`reset()` 會初始化所有變數，這裡會初始化 `eventChan`、`listeners`、`pressed`。

```go
func reset() {
    eventChan = make(chan types.KeyboardEvent, defaultEventChanSize)
    listeners = make(map[string]*hotkeyListener, defaultListenerSize)
    pressed = [defaultPressedSize]bool{}
}
```

### Hotkey

Hotkey 是一個按鍵組合，例如 `Ctrl+Shift+P`，這個按鍵組合可以用 `VKCode` 來表示：

```go
type Hotkey []types.VKCode
```

`Hotkey` 有幾個 method：

#### NewHotkey()

`NewHotkey()` 用來建立一個新的 `Hotkey`，並且將 `VKCode` 依照順序排序。

```go
func NewHotkey(keys ...types.VKCode) Hotkey {
    slices.Sort(keys)
    return Hotkey(keys)
}
```

#### String()

`String()` 用來將 `Hotkey` 轉換成字串，例如 `Ctrl+Shift+P`。
`String()` 也被用來識別 `Hotkey`，因為 `Hotkey` 是一個 slice，所以無法直接用 `==` 來比較，必須要用 `String()` 來比較。

```go
func (h Hotkey) String() string {
    b := strings.Builder{}
    for i, key := range h {
        if i > 0 {
            b.WriteString("+")
        }
        b.WriteString(key.String())
    }
    return b.String()
}
```

這邊使用 `strings.Builder` 來建立字串，因為 `strings.Builder` 是一個 buffer，可以有效率的建立字串。

#### Pressed()

`Pressed()` 用來檢查 `Hotkey` 是否被按下。

```go
func (h Hotkey) Pressed() bool {
    for _, key := range h {
        if !pressed[key] {
            return false
        }
    }
    return true
}
```

### hotkeyListener

`hotkeyListener` 是一個監聽器，當按下特定的按鍵組合時，會觸發一個 callback function。

```go
type hotkeyListener struct {
    // hotkey is a hotkey to listen.
    hotkey Hotkey
    // callback is a callback function which is called when the hotkey is pressed.
    callback func()
}
```

`hotkeyListener` 有幾個 method：

#### newHotkeyListener()

`newHotkeyListener()` 用來建立一個新的 `hotkeyListener`。

```go
func newHotkeyListener(hotkey Hotkey, callback func()) *hotkeyListener {
    if callback == nil {
        callback = func() {}
    }
    return &hotkeyListener{
        hotkey:   hotkey,
        callback: callback,
    }
}
```

hotkeyListener 會儲存 `Hotkey` 與 callback function。

#### Notify()

`Notify()` 會檢查 `Hotkey` 是否被按下，如果有被按下，就會呼叫 callback function。

```go
func (l *hotkeyListener) Notify() {
    l.mu.Lock()
    defer l.mu.Unlock()
    if !l.hotkey.Pressed() {
        l.justPressed = false
        return
    }
    if l.justPressed {
        return
    }
    l.justPressed = true
    l.callback()
}
```

這邊使用 `justPressed` 來避免重複觸發 callback function。
除此之外，`Notify()` 也使用 `sync.Mutex` 來避免同時觸發 callback function。

### RegisterHotkey()

`RegisterHotkey()` 用來註冊 `Hotkey`。

```go
func RegisterHotkey(hotkey Hotkey, callback func()) {
    listeners[hotkey.String()] = newHotkeyListener(hotkey, callback)
}
```

這邊使用 `listeners` 來儲存 `Hotkey`，`listeners` 是一個 `map`，`key` 是 `Hotkey` 的字串表示，`value` 是 `hotkeyListener`。

### UnregisterHotkey()

`UnregisterHotkey()` 用來取消註冊 `Hotkey`。

```go
func UnregisterHotkey(hotkey Hotkey) {
    delete(listeners, hotkey.String())
}
```

`UnregisterHotkey()` 的作法很簡單，就是從 `listeners` 中刪除 `Hotkey`。

### handleEvent()

`handleEvent()` 用來處理鍵盤事件，當鍵盤事件發生時，會通知所有的 `hotkeyListener`。

```go
func handleEvent(event types.KeyboardEvent) {
    switch event.Message {
    case types.WM_KEYDOWN, types.WM_SYSKEYDOWN:
        pressed[event.VKCode] = true
        for _, listener := range listeners {
            listener.Notify()
        }
    case types.WM_KEYUP, types.WM_SYSKEYUP:
        pressed[event.VKCode] = false
    }
}
```

### Start()

`Start()` 用來啟動鍵盤監聽器。

```go
func Start() error {
    go func() {
        for event := range eventChan {
            handleEvent(event)
        }
    }()
    return keyboard.Install(nil, eventChan)
}
```

當 `Start()` 被呼叫時，會使用一個 goroutine 來處理鍵盤事件，當事件發生時就透過 `handleEvent()` 來處理建盤事件，並且呼叫 `keyboard.Install()` 來啟動鍵盤監聽器。

### Stop()

`Stop()` 用來停止鍵盤監聽器。

```go
func Stop() error {
    err := keyboard.Uninstall()
    if err != nil {
        return err
    }
    close(eventChan)
    reset()
    return nil
}
```

當 `Stop()` 被呼叫時，會呼叫 `keyboard.Uninstall()` 來停止鍵盤監聽器，同時關閉 `eventChan`，並呼叫 `reset()` 來重設所有的按鍵狀態。

## 修改 app.go

### 新增 bindKeyboard()

```go
// bindKeyboard binds the keyboard to the app
func (a *App) bindKeyboard() {
    // Register the hotkey
    keyboard.RegisterHotkey(keyboard.NewHotkey(types.VK_LCONTROL, types.VK_LMENU, types.VK_A), func() {
        runtime.EventsEmit(a.ctx, "capture")
    })
    // Start the keyboard
    if err := keyboard.Start(); err != nil {
        log.Fatal(err)
    }
    // Stop the keyboard when the app closes
    go func() {
        <-a.ctx.Done()
        if err := keyboard.Stop(); err != nil {
            log.Fatal(err)
        }
    }()
}
```

這邊註冊了一個 `Ctrl+Alt+A` 的熱鍵，當按下熱鍵時，就會觸發 `runtime.EventsEmit(a.ctx, "capture")`。
而 `runtime.EventsEmit(a.ctx, "capture")` 會發出一個 `Wails` 的 `capture` 事件，讓 `frontend` 可以接收到。

### 修改 startup()

將 bindKeyboard() 加入到 `startup()` 中。

```go
func (a *App) startup(ctx context.Context) {
    a.ctx = ctx
    a.bindSystray()
    a.bindKeyboard()
}
```

## 修改 frontend

### 接收 capture 事件

修改 `App.svelte`，加入 `EventsOn("capture", () => {})`。

```js
import { EventsOn } from "../wailsjs/runtime/runtime.js"

...

EventsOn("capture", () => {
    greet()
})
```

## 測試

執行程式，按下 `Ctrl+Alt+A`，就會看到成功觸發 `capture` 事件並顯示截圖。
