require "import"
import "android.widget.*"
import "android.content.*"
import "android.speech.*"
import "android.view.*"
import "android.os.*"
import "com.androlua.Http"
import "cjson"

-- ورژن اب 2.0 کر دیا گیا ہے
local current_version = "2.0"
local version_url = "https://raw.githubusercontent.com/shahzaibmemon7856-tech/Sindhi-typer-/refs/heads/main/Version.txt"
local notes_url = "https://raw.githubusercontent.com/shahzaibmemon7856-tech/Sindhi-typer-/refs/heads/main/Notes.txt"
local update_lua_url = "https://raw.githubusercontent.com/shahzaibmemon7856-tech/Sindhi-typer-/refs/heads/main/Updates.lua"

local prefs = service.getSharedPreferences("GeminiTyperSettings", Context.MODE_PRIVATE)
local editor = prefs.edit()

local GEMINI_API_KEY = prefs.getString("api_key", "")
local selectedModel = prefs.getString("selected_model", "gemini-1.5-flash")
local currentLang = prefs.getString("selected_lang", "sd-PK")

-- اپڈیٹ چیک کرنے کا فنکشن
function checkUpdates()
  Http.get(version_url, function(code, server_version)
    if code == 200 and server_version then
      server_version = server_version:gsub("%s+", "")
      -- اگر سرور پر ورژن موجودہ ورژن سے مختلف ہو
      if server_version ~= current_version then
        Http.get(notes_url, function(code2, notes)
          showUpdateDialog(server_version, notes or "New update available!")
        end)
      end
    end
  end)
end

function showUpdateDialog(new_ver, notes)
  local builder = AlertDialog.Builder(service)
  builder.setTitle("Update Available: " .. new_ver)
  
  local layout = LinearLayout(service)
  layout.setOrientation(1)
  layout.setPadding(40, 20, 40, 20)
  
  local txtNotes = TextView(service)
  txtNotes.setText(notes)
  txtNotes.setTextSize(16)
  layout.addView(txtNotes)
  
  builder.setView(layout)
  builder.setPositiveButton("Update Now", {onClick=function()
    Http.get(update_lua_url, function(code, content)
      if code == 200 then
        -- نیا کوڈ لوڈ اور رن کرنا
        assert(load(content))()
        service.speak("Update successful to version " .. new_ver)
      else
        service.speak("Update failed")
      end
    end)
  end})
  
  builder.setNegativeButton("Later", nil)
  
  local diag = builder.create()
  diag.getWindow().setType(WindowManager.LayoutParams.TYPE_ACCESSIBILITY_OVERLAY)
  diag.show()
end

-- باقی تمام فنکشنز (processWithGemini, startVoiceTyping وغیرہ) ویسے ہی رہیں گے

function showMainUI()
  -- UI دکھانے سے پہلے اپڈیٹ چیک کریں
  checkUpdates()
  
  local builder = AlertDialog.Builder(service)
  builder.setTitle("Gemini Typer Pro V" .. current_version)

  -- (باقی UI کا کوڈ یہاں آئے گا...)
  -- آپ کا پرانا UI کوڈ یہاں برقرار ہے
  
  local layout = LinearLayout(service)
  layout.setOrientation(1)
  layout.setPadding(40, 20, 40, 20)

  local chkRoman = CheckBox(service)
  chkRoman.setText("Roman Sindhi (Urdu/Eng/Hindi to Sindhi)")
  layout.addView(chkRoman)

  local langGroup = RadioGroup(service)
  local rbSdPK = RadioButton(service) rbSdPK.setText("Sindhi Pakistan") rbSdPK.setId(1)
  local rbSdIN = RadioButton(service) rbSdIN.setText("Sindhi India") rbSdIN.setId(2)
  
  langGroup.addView(rbSdPK) 
  langGroup.addView(rbSdIN) 
  layout.addView(langGroup)
  
  if currentLang == "sd-IN" then rbSdIN.setChecked(true) else rbSdPK.setChecked(true) end

  local btnStart = Button(service)
  btnStart.setText("START VOICE TYPING")
  btnStart.setBackgroundColor(0xFF4CAF50)
  btnStart.setTextColor(0xFFFFFFFF)
  layout.addView(btnStart)

  local btnEngine = Button(service)
  btnEngine.setText("AI ENGINE SETTINGS")
  layout.addView(btnEngine)

  local footerLayout = LinearLayout(service)
  footerLayout.setOrientation(0)
  
  local btnAbout = Button(service)
  btnAbout.setText("ABOUT")
  footerLayout.addView(btnAbout)

  local btnExit = Button(service)
  btnExit.setText("EXIT")
  footerLayout.addView(btnExit)
  
  layout.addView(footerLayout)

  builder.setView(layout)
  local dialog = builder.create()
  dialog.getWindow().setType(WindowManager.LayoutParams.TYPE_ACCESSIBILITY_OVERLAY)

  btnStart.setOnClickListener{onClick=function()
    local checkedId = langGroup.getCheckedRadioButtonId()
    if checkedId == 1 then currentLang = "sd-PK"
    else currentLang = "sd-IN" end
    
    editor.putString("selected_lang", currentLang).commit()
    dialog.dismiss()
    startVoiceTyping(chkRoman.isChecked())
  end}

  btnEngine.setOnClickListener{onClick=function()
    dialog.dismiss()
    showAiEngineDialog()
  end}

  btnAbout.setOnClickListener{onClick=function()
    service.speak("Gemini Typer with Advanced Sindhi Support. Current Version " .. current_version)
  end}

  btnExit.setOnClickListener{onClick=function()
    dialog.dismiss()
  end}

  dialog.show()
end

showMainUI()
return true
