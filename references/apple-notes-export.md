# Apple Notes Export — Working osascript Pattern

## 核心問題
直接用 f-string 把長文章內容嵌入 AppleScript 的 `body:` 屬性，會遇到：
- 雙引號/換行符轉義地獄
- 中文長字串語法錯誤
- `SIGPIPE` 錯誤（osascript 提前終止）

## 正確做法：檔案中介

### Python 端（FastAPI）
```python
import os, re, subprocess

content_file = f"/tmp/note_body_{os.urandom(4).hex()}.txt"
script_file = f"/tmp/note_script_{os.urandom(4).hex()}.scpt"

# 寫內容到暫存檔（不在 f-string 裡）
with open(content_file, 'w', encoding='utf-8') as f:
    f.write(plain_text)

# 標題用變數，長度限制
safe_title = title.replace('"', "'").replace('\\', '\\\\').replace('\n', ' ')[:200]

apple_script = f'''
set noteTitle to "{safe_title}"
set bodyPath to "{content_file}"

tell application "Notes"
    try
        set folderName to "Hermes"
        if not (exists folder folderName) then
            make new folder with properties {{name:folderName}}
        end if
        set targetFolder to folder folderName

        set fRef to open for access file bodyPath
        set noteBody to read fRef as «class utf8»
        close access fRef

        make new note at targetFolder with properties {{name:noteTitle, body:noteBody}}
    on error err
        try
            close access file bodyPath
        end try
        return "Error: " & err
    end try
end tell
'''

with open(script_file, 'w', encoding='utf-8') as f:
    f.write(apple_script)

result = subprocess.run(['osascript', script_file], capture_output=True, timeout=30)
if result.returncode != 0:
    err = result.stderr.decode('utf-8', errors='replace').strip()
    return {'error': err}

# 清理
for f in [content_file, script_file]:
    if os.path.exists(f):
        os.remove(f)
```

## 關鍵要點
- 內容寫 `.txt`，腳本寫 `.scpt`，各自獨立
- `bodyPath` 是檔案路徑字串，AppleScript 用 `open for access file` 讀
- `as «class utf8»` 是 AppleScript 讀 UTF-8 文字的標準寫法
- `finally` 塊確保暫存檔一定被清理
- 30 秒 timeout，防止 Notes 無回應卡住整個服務