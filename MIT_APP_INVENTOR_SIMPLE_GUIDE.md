# MIT App Inventor - Simple Blocks Guide

## 📱 Components Needed

### Screen 1 (Launcher)
- `ButtonPlay` - Button
- `TinyDB1` - Storage
- `PlayerBackground` - Background music

### Screen 2 (Lessons Menu)
- `WebViewerLessons` - WebViewer
- `TinyDB1` - Same instance
- `ClockMonitorLessons` - Clock (TimerInterval: 100ms)

### Screen 3 (Game)
- `WebViewerGame` - WebViewer
- `TinyDB1` - Same instance
- `ClockMonitorGame` - Clock (TimerInterval: 100ms)
- `PlayerCorrect` - Sound player
- `PlayerVictory` - Sound player
- `PlayerWrong` - Sound player
- `PlayerBoost` - Sound player
- `PlayerGameOver` - Sound player

---

## 🔧 Global Variables (Create in Variables drawer)

**Initialize ALL in Screen1.Initialize:**

```
completedLevels (List)
unlockedLevels (List)
musicStarted (Boolean)
selectedLevel (Number)
currentGameLevel (Number)
levelText (Text)
jsonString (Text)
jsonObjectString (Text)
firstItem (Boolean)
webViewString (Text)
gameWebViewString (Text)
levelNum (List)
completedLevel (Number)
nextLevel (Number)
levelParts (List)
savedCompleted (List)
soundParts (List)
soundName (Text)
startLevelCommand (Text)
tempLevel (Number)
tempLevelNum (Number)
```

---

## 📄 SCREEN 1: Launcher

### 1. Screen1.Initialize

```
When Screen1.Initialize
  Do:
    set global completedLevels to (make a list)
    set global unlockedLevels to (make a list)
    add items to list (get global unlockedLevels) (1)
    
    set global jsonString to ""
    set global jsonObjectString to ""
    set global firstItem to true
    set global webViewString to ""
    set global gameWebViewString to ""
    
    set global savedCompleted to (call TinyDB1.GetValue with tag "completedLevels" valueIfTagNotThere (make a list))
    
    If (is a list? (get global savedCompleted))
      set global completedLevels to (make a list)
      
      For each item item in (get global savedCompleted)
        If (is a number? (get item))
          set global tempLevel to (get item)
          If (and ((get global tempLevel) >= 1) ((get global tempLevel) <= 6))
            add items to list (get global completedLevels) (get global tempLevel)
          End If
        End If
      End For
      
      set global unlockedLevels to (make a list)
      add items to list (get global unlockedLevels) (1)
      
      For each item levelNum in (get global completedLevels)
        If (is a number? (get levelNum))
          set global tempLevelNum to (get levelNum)
          If (and ((get global tempLevelNum) >= 1) ((get global tempLevelNum) < 6))
            set global nextLevel to ((get global tempLevelNum) + 1)
            If (not (is in list (get global nextLevel) (get global unlockedLevels)))
              add items to list (get global unlockedLevels) (get global nextLevel)
            End If
          End If
        End If
      End For
    End If
    
    If (not (get global musicStarted))
      call PlayerBackground.Play
      set global musicStarted to true
    End If
End
```

### 2. ButtonPlay.Click

```
When ButtonPlay.Click
  Do:
    open another screen Screen2
End
```

---

## 📚 SCREEN 2: Lessons Menu

### 1. Screen2.Initialize

```
When Screen2.Initialize
  Do:
    set global completedLevels to (call TinyDB1.GetValue with tag "completedLevels" valueIfTagNotThere (make a list))
    
    If (not (is a list? (get global completedLevels)))
      set global completedLevels to (make a list)
    End If
    
    set global unlockedLevels to (make a list)
    
    For each item levelNum in (get global completedLevels)
      If (is a number? (get levelNum))
        set global tempLevelNum to (get levelNum)
        If (and ((get global tempLevelNum) >= 1) ((get global tempLevelNum) < 6))
          set global nextLevel to ((get global tempLevelNum) + 1)
          add items to list (get global unlockedLevels) (get global nextLevel)
        End If
      End If
    End For
    
    If (not (is in list (1) (get global unlockedLevels)))
      add items to list (get global unlockedLevels) (1)
    End If
    
    call JSONEncode with data (get global unlockedLevels)
    set global jsonObjectString to (join "{\"unlockedLevels\":" (join (get global jsonString) "}"))
    set WebViewerLessons.WebViewString to (get global jsonObjectString)
    
    set WebViewerLessons.HomeUrl to "file:///android_asset/lessons.html"
    call WebViewerLessons.GoHome
End
```

### 2. Screen2.BackPressed

```
When Screen2.BackPressed
  Do:
    set ClockMonitorLessons.TimerEnabled to false
    close screen
End
```

### 3. ClockMonitorLessons.Timer

```
When ClockMonitorLessons.Timer
  Do:
    set global webViewString to (get WebViewerLessons.WebViewString)
    
    If ((get global webViewString) contains "levelCompleted:")
      set global levelNum to (call split at ":" with text (get global webViewString))
      set global completedLevelText to (select list item 2 from (get global levelNum))
      set global completedLevel to ((get global completedLevelText) + 0)
      
      If (is a number? (get global completedLevel))
        If (and ((get global completedLevel) >= 1) ((get global completedLevel) <= 6))
          If (not (is in list (get global completedLevel) (get global completedLevels)))
            add items to list (get global completedLevels) (get global completedLevel)
            call TinyDB1.StoreValue with tag "completedLevels" and value completedLevels
            
            If ((get global completedLevel) < 6)
              set global nextLevel to ((get global completedLevel) + 1)
              If (not (is in list (get global nextLevel) (get global unlockedLevels)))
                add items to list (get global unlockedLevels) (get global nextLevel)
              End If
            End If
            
            call JSONEncode with data (get global unlockedLevels)
            set global jsonObjectString to (join "{\"unlockedLevels\":" (join (get global jsonString) "}"))
            set WebViewerLessons.WebViewString to (get global jsonObjectString)
          End If
        End If
      End If
      
      set WebViewerLessons.WebViewString to ""
    End If
    
    If ((get global webViewString) contains "startLevel:")
      set global levelParts to (call split at ":" with text (get global webViewString))
      set global selectedLevel to ((select list item 2 from (get global levelParts)) + 0)
      
      set ClockMonitorLessons.TimerEnabled to false
      set WebViewerLessons.Visible to false
      open another screen Screen3
      set WebViewerLessons.WebViewString to ""
    End If
    
    If ((get global webViewString) contains "resetProgress")
      set global completedLevels to (make a list)
      set global unlockedLevels to (make a list)
      add items to list (get global unlockedLevels) (1)
      
      call TinyDB1.StoreValue with tag "completedLevels" and value completedLevels
      call TinyDB1.StoreValue with tag "unlockedLevels" and value unlockedLevels
      
      call WebViewerLessons.Reload
      set WebViewerLessons.WebViewString to ""
    End If
End
```

---

## 🎮 SCREEN 3: Game

### 1. Screen3.Initialize

```
When Screen3.Initialize
  Do:
    set WebViewerGame.Visible to true
    
    set global currentGameLevel to (get global selectedLevel)
    
    If ((get global currentGameLevel) = 0)
      set global currentGameLevel to 1
    End If
    
    set global levelText to (join "" (get global currentGameLevel))
    
    set WebViewerGame.HomeUrl to "file:///android_asset/FINAL_FIXED.html"
    call WebViewerGame.GoHome
    
    set global startLevelCommand to (join "startLevel:" (get global levelText))
    set WebViewerGame.WebViewString to (get global startLevelCommand)
    
    set ClockMonitorGame.TimerEnabled to true
End
```

### 2. ClockMonitorGame.Timer

```
When ClockMonitorGame.Timer
  Do:
    set global gameWebViewString to (get WebViewerGame.WebViewString)
    
    // Screen visibility commands
    If ((get global gameWebViewString) contains "switchToScreen3")
      set WebViewerGame.Visible to true
      set WebViewerGame.WebViewString to ""
    End If
    
    If ((get global gameWebViewString) contains "gameStarting:Screen3")
      set WebViewerGame.Visible to true
      set WebViewerGame.WebViewString to ""
    End If
    
    If (or ((get global gameWebViewString) contains "gameScreenVisible") ((get global gameWebViewString) contains "quizScreenActive"))
      set WebViewerGame.Visible to true
      set WebViewerGame.WebViewString to ""
    End If
    
    // Sound effects
    If ((get global gameWebViewString) contains "playSound:")
      set global soundParts to (call split at ":" with text (get global gameWebViewString))
      set global soundName to (select list item 2 from (get global soundParts))
      
      If ((get global soundName) = "correct")
        call PlayerCorrect.Play
      End If
      If ((get global soundName) = "victory")
        call PlayerVictory.Play
      End If
      If ((get global soundName) = "wrong")
        call PlayerWrong.Play
      End If
      If ((get global soundName) = "boost")
        call PlayerBoost.Play
      End If
      If ((get global soundName) = "gameover")
        call PlayerGameOver.Play
      End If
      
      set WebViewerGame.WebViewString to ""
    End If
    
    // Level completion
    If ((get global gameWebViewString) contains "levelCompleted:")
      set global levelParts to (call split at ":" with text (get global gameWebViewString))
      set global completedLevelText to (select list item 2 from (get global levelParts))
      set global completedLevel to ((get global completedLevelText) + 0)
      
      If (is a number? (get global completedLevel))
        If (and ((get global completedLevel) >= 1) ((get global completedLevel) <= 6))
          set global completedLevels to (call TinyDB1.GetValue with tag "completedLevels" valueIfTagNotThere (make a list))
          
          If (not (is a list? (get global completedLevels)))
            set global completedLevels to (make a list)
          End If
          
          If (not (is in list (get global completedLevel) (get global completedLevels)))
            add items to list (get global completedLevels) (get global completedLevel)
            call TinyDB1.StoreValue with tag "completedLevels" and value completedLevels
          End If
          
          set global nextLevel to ((get global completedLevel) + 1)
          
          If (and ((get global nextLevel) >= 1) ((get global nextLevel) <= 6))
            set global currentGameLevel to (get global nextLevel)
            set global levelText to (join "" (get global currentGameLevel))
            
            set global startLevelCommand to (join "startLevel:" (get global levelText))
            set WebViewerGame.WebViewString to (get global startLevelCommand)
            
            set global fullUrl to (join "file:///android_asset/FINAL_FIXED.html?level=" (get global levelText))
            call WebViewerGame.GoToUrl with url (get global fullUrl)
          Else
            set ClockMonitorGame.TimerEnabled to false
            close screen
          End If
        End If
      End If
      
      set WebViewerGame.WebViewString to ""
    End If
    
    // Back button navigation
    If ((get global gameWebViewString) contains "goBackToMenu")
      set WebViewerGame.WebViewString to ""
      set ClockMonitorGame.TimerEnabled to false
      close screen
    End If
    
    // Reset progress
    If ((get global gameWebViewString) contains "resetProgress")
      set global completedLevels to (make a list)
      set global unlockedLevels to (make a list)
      add items to list (get global unlockedLevels) (1)
      
      call TinyDB1.StoreValue with tag "completedLevels" and value completedLevels
      call TinyDB1.StoreValue with tag "unlockedLevels" and value unlockedLevels
      
      call WebViewerGame.Reload
      set WebViewerGame.WebViewString to ""
    End If
End
```

### 3. Screen3.BackPressed

```
When Screen3.BackPressed
  Do:
    set WebViewerGame.WebViewString to "backButtonPressed"
End
```

---

## 🔨 PROCEDURE: JSONEncode

**Create this procedure in Procedures drawer:**

```
To JSONEncode with data
  Do:
    set global jsonString to "["
    set global firstItem to true
    
    For each item num in (get data)
      If (not (get global firstItem))
        set global jsonString to (join (get global jsonString) ",")
      End If
      set global jsonString to (join (get global jsonString) (join "" (get num)))
      set global firstItem to false
    End For
    
    set global jsonString to (join (get global jsonString) "]")
End
```

**How to create:**
1. Go to Procedures drawer
2. Drag "to procedure" block (NOT "to procedure result")
3. Rename to "JSONEncode"
4. Click gear icon ⚙️
5. Drag "name" block into "inputs" section
6. Rename "x" to "data"
7. Close gear menu
8. Add blocks above inside "do" socket

---

## 📝 Quick Reference

### Block Locations

| Block | Drawer |
|-------|--------|
| `make a list` | Lists |
| `is a list?` | Lists |
| `is in list` | Lists |
| `add items to list` | Lists |
| `select list item` | Lists |
| `for each item in list` | Control |
| `if` | Control |
| `and` | Logic |
| `not` | Logic |
| `>=`, `<`, `=` | Logic |
| `is a number?` | Math |
| `+` | Math |
| `join` | Text |
| `split at` | Text |
| `contains` | Text |
| `set global` | Variables |
| `get global` | Variables |
| `open another screen` | Control |
| `close screen` | Control |

### File Paths

**Screen 2:**
```
file:///android_asset/lessons.html
```

**Screen 3:**
```
file:///android_asset/FINAL_FIXED.html
```

⚠️ **IMPORTANT:** Use exact filename as uploaded to Media (case-sensitive!)

### Sound Files

Upload these to Media:
- `correct.mp3` or `correct.wav`
- `victory.mp3` or `victory.wav`
- `wrong.mp3` or `wrong.wav`
- `boost.mp3` or `boost.wav`
- `gameover.mp3` or `gameover.wav`
- `bgMusic.mp3` or `bgMusic.wav`

Set Source property of each Player component to corresponding file.

---

## ✅ Checklist

- [ ] All components added to screens
- [ ] All global variables created
- [ ] All blocks added correctly
- [ ] JSONEncode procedure created
- [ ] HTML files uploaded to Media
- [ ] Sound files uploaded to Media
- [ ] File paths match Media filenames (case-sensitive)
- [ ] Clock timers set to 100ms
- [ ] WebViewer Visible = true
- [ ] PlayerBackground added to Screen1 only

---

## 🐛 Common Issues

**Problem:** Music restarts when returning to Screen1
**Fix:** Check `musicStarted` variable is initialized and checked before `Play`

**Problem:** "Webpage cannot be displayed"
**Fix:** 
- Check file uploaded to Media
- Check filename matches exactly (case-sensitive)
- Use `file:///android_asset/` (3 slashes)

**Problem:** "cannot accept arguments" error
**Fix:** Convert number to text before JOIN:
```
set global levelText to (join "" (get global currentGameLevel))
```

**Problem:** JOIN block error
**Fix:** Store JOIN result in variable first:
```
set global fullUrl to (join "file:///android_asset/FINAL_FIXED.html?level=" (get global levelText))
call WebViewerGame.GoToUrl with url (get global fullUrl)
```

---

## 📦 Data Storage

**TinyDB Tags:**
- `completedLevels` - List of completed levels `[1, 2, 3]`
- `unlockedLevels` - List of unlocked levels `[1, 2, 3, 4]`

Data persists after app closes!

