require "import"
import "android.widget.*"
import "android.content.*"
import "android.speech.*"
import "android.view.*"
import "android.os.*"
import "com.androlua.Http"
import "cjson"

local prefs = service.getSharedPreferences("GeminiTyperSettings", Context.MODE_PRIVATE)
local editor = prefs.edit()

local GEMINI_API_KEY = prefs.getString("api_key", "")
local selectedModel = prefs.getString("selected_model", "gemini-1.5-flash")
local currentLang = prefs.getString("selected_lang", "sd-PK")

function processWithGemini(rawText, mode, isRomanChecked, callback)
  if GEMINI_API_KEY == "" then
    service.speak("Please configure AI Engine first")
    return
  end

  local prompt = ""
  
  if isRomanChecked then
    prompt = "The following text is spoken in Urdu, English, or Hindi. Translate and convert it into proper Sindhi Arabic script. Only return the translated text: " .. rawText
  else
    if mode == "sd-PK" then
      prompt = "Correct this Sindhi (Pakistan) text, fix spelling and grammar. Only return the result: " .. rawText
    elseif mode == "sd-IN" then
      prompt = "Correct this Sindhi (India/Devanagari or Arabic script as appropriate) text. Only return the result: " .. rawText
    end
  end

  local url = "https://generativelanguage.googleapis.com/v1beta/models/" .. selectedModel .. ":generateContent?key=" .. GEMINI_API_KEY
  local payload = {
    contents = {{ parts = {{ text = prompt }} }}
  }

  Http.post(url, cjson.encode(payload), {["Content-Type"]="application/json"}, function(status, data)
    if status == 200 then
      local ok, res = pcall(cjson.decode, data)
      if ok and res.candidates then
        local aiText = res.candidates[1].content.parts[1].text
        aiText = aiText:gsub("\n", "")
        callback(aiText)
      end
    else
      service.speak("API Error: " .. status)
    end
  end)
end

function startVoiceTyping(isRomanChecked)
  local edit = service.getEditText()
  if not edit then
    service.speak("No text field detected")
    return
  end

  local intent = Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH)
  intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL, RecognizerIntent.LANGUAGE_MODEL_FREE_FORM)
  
  if isRomanChecked then
    intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE, "ur-PK")
  else
    intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE, currentLang)
  end

  local rec = SpeechRecognizer.createSpeechRecognizer(service)
  rec.setRecognitionListener{
    onResults = function(res)
      local list = res.getStringArrayList(SpeechRecognizer.RESULTS_RECOGNITION)
      local spokenText = list.get(0)
      service.speak("Processing...") 
      processWithGemini(spokenText, currentLang, isRomanChecked, function(refinedText)
        service.insertText(edit, refinedText)
        service.speak("Done")
      end)
      rec.destroy()
    end,
    onError = function() rec.destroy() end
  }
  rec.startListening(intent)
end

function showAiEngineDialog()
  local builder = AlertDialog.Builder(service)
  builder.setTitle("AI Engine Settings")
  local layout = LinearLayout(service)
  layout.setOrientation(1)
  layout.setPadding(40, 30, 40, 30)

  local keyInput = EditText(service)
  keyInput.setHint("Enter Gemini API Key")
  keyInput.setText(GEMINI_API_KEY)
  layout.addView(keyInput)

  local modelSpinner = Spinner(service)
  local models = {"gemini-1.5-flash", "gemini-1.5-pro", "gemini-pro"}
  local adapter = ArrayAdapter(service, android.R.layout.simple_spinner_item, models)
  modelSpinner.setAdapter(adapter)
  for i, m in ipairs(models) do if m == selectedModel then modelSpinner.setSelection(i-1) end end
  layout.addView(modelSpinner)

  local btnSave = Button(service)
  btnSave.setText("SAVE CONFIGURATION")
  layout.addView(btnSave)

  builder.setView(layout)
  local diag = builder.create()
  diag.getWindow().setType(WindowManager.LayoutParams.TYPE_ACCESSIBILITY_OVERLAY)

  btnSave.setOnClickListener{onClick=function()
    GEMINI_API_KEY = keyInput.getText().toString()
    selectedModel = models[modelSpinner.getSelectedItemPosition() + 1]
    editor.putString("api_key", GEMINI_API_KEY)
    editor.putString("selected_model", selectedModel)
    editor.commit()
    service.speak("Settings Saved")
    diag.dismiss()
  end}
  diag.show()
end

function showMainUI()
  local builder = AlertDialog.Builder(service)
  builder.setTitle("Gemini Typer Pro")

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
  
  -- Default Selection
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
    service.speak("Gemini Typer with Advanced Sindhi Support")
  end}

  btnExit.setOnClickListener{onClick=function()
    dialog.dismiss()
  end}

  dialog.show()
end

showMainUI()
return true
