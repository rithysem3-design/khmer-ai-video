# AI Subtitle Editor & Translator

កម្មវិធីកែ subtitle លើទូរស័ព្ទ (Flutter) + backend (Python FastAPI)។ APK ត្រូវបាន build នៅលើ **GitHub Actions**។

## រចនាសម្ព័ន្ធ
```
app/        Flutter (UI, SRT, drag subtitle, style, settings)
backend/    FastAPI (translate, edge-tts, transcribe, export ជាមួយ FFmpeg)
scripts/    fetch_fonts.sh (ទាញ font ខ្មែរ OFL), patch_manifest.py
.github/workflows/build-apk.yml
```

## ១. Build APK នៅលើ GitHub
1. បង្កើត repository ថ្មីលើ GitHub រួច upload ឯកសារទាំងអស់ក្នុង zip នេះ (ឬ `git push`) ទៅ branch `main`។
2. ចូលផ្ទាំង **Actions** → workflow **Build APK** នឹងរត់ដោយស្វ័យប្រវត្តិ (ឬចុច *Run workflow*)។
3. ពេលរួច (ប្រហែល ៥–១០ នាទី) ចុចលើ run នោះ → **Artifacts** → ទាញ `AI-Subtitle-Editor-apk` → ដាក់ `app-release.apk` ក្នុងទូរស័ព្ទ ហើយ install។
4. ប្រសិនបើ build បរាជ័យ សូមចម្លងសារ error ពី Actions មកខ្ញុំ។

workflow នេះ៖ ទាញ font → បង្កើត android scaffolding → `flutter test` → `flutter build apk --release`។ APK ហត្ថលេខាដោយ debug key (install បាន ប៉ុន្តែមិនសម្រាប់ Play Store)។

## ២. បើក backend (ចាំបាច់សម្រាប់ Generate Sub / Translate / Generate / Export)
APK មិនអាចរត់ FFmpeg/edge-tts ខ្លួនឯងបានទេ ត្រូវមាន server (កុំព្យូទ័រ ឬ VPS)។
```bash
./scripts/fetch_fonts.sh            # ទាញ font ទៅ backend/fonts
cd backend
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
# ចង់បាន Generate Sub: pip install faster-whisper
# ត្រូវមាន ffmpeg ក្នុងម៉ាស៊ីន (apt install ffmpeg)
export API_KEY=លេខសម្ងាត់របស់អ្នក
export FONTS_DIR=./fonts
export LIBRETRANSLATE_URL=http://localhost:5000   # សម្រាប់ Translate
uvicorn app.main:app --host 0.0.0.0 --port 8000
```
ឬប្រើ Docker: `cd backend && docker build -t subeditor . && docker run -p 8000:8000 -e API_KEY=xxx subeditor`

ក្នុងកម្មវិធី: **Settings** → ដាក់ Server URL (ឧ. `http://192.168.1.10:8000` – ទូរស័ព្ទ និងកុំព្យូទ័រត្រូវនៅ Wi-Fi ដូចគ្នា) និង API key → ចុច **Check Now**។ ស្ថានភាព៖ Healthy / Warning / Failed / Not Configured។

## ៣. របៀបប្រើ
1. **Upload File** → Import video ឬ Import .srt។
2. **Generate Sub** → បង្កើត subtitle ពីសំឡេង (ត្រូវ faster-whisper លើ server)។
3. **Translate** → ជ្រើសភាសា → បញ្ជាក់មុនជំនួសអត្ថបទ (Undo បាន)។
4. ចុចជួរ subtitle ណាមួយ → កែអត្ថបទ/ពេលវេលា, Duplicate, Split, Merge, Add, Delete។
5. **Text** → ជ្រើស font ខ្មែរ ទំហំ ពណ៌ ផ្ទៃខាងក្រោយ។ **អូស subtitle លើវីដេអូ** ដើម្បីប្តូរទីតាំង។
6. ជ្រើស **Male/Female** នៅជួរនីមួយៗ → **Generate** បង្កើតសំឡេង (edge-tts រកសំឡេងខ្មែរដោយស្វ័យប្រវត្តិ)។
7. **Save** → រក្សាទុក .srt / .mp3 / Export វីដេអូដែលដុត subtitle។ ឯកសារនៅ `Android/data/com.subtitleeditor.subtitle_editor/files/`។

## អ្វីដែលធ្វើរួច និងដែនកំណត់ (ស្មោះត្រង់)
ធ្វើរួច៖ import/edit/export SRT, undo/redo, autosave, drag subtitle (normalized), style panel, theme Dark/Light/System, diagnostics + clear temp, backend API (translate provider interface, edge-tts voices/generate, transcribe, export, jobs), tests។

**ពិនិត្យរួច**៖ តែ test ផ្នែក Python (SRT, ASS, Khmer, position clamp) ប៉ុណ្ណោះ។ កូដ Flutter និង endpoint FastAPI **មិនទាន់បាន compile/រត់** ក្នុងបរិស្ថានខ្ញុំ (គ្មាន Flutter SDK/network) — GitHub Actions ជាការ test ដំបូង។
ខ្វះ/ដែនកំណត់៖
- Font "Khmer OS" មិនអាចចែកចាយបានទេ → ប្រើ Battambang, Siemreap, Moul (ស្រដៀង Muol), Noto Sans Khmer (OFL)។ Server ត្រូវមាន font ដូចគ្នា (`FONTS_DIR`)។
- Export: ជ្រុងមូលនៃផ្ទៃខាងក្រោយមិនទាន់ដុតចូលវីដេអូ (ASS ធ្វើបានតែប្រអប់ការ៉េ); preview បង្ហាញជ្រុងមូល។
- Export ផ្ញើវីដេអូទាំងមូលទៅ server (ហើយទាញលទ្ធផលចូល memory) → វីដេអូធំៗអាចយឺត/ញ៉ាំ RAM។
- Job នៅក្នុង memory (restart server ភ្លេច), គ្មាន cancel, គ្មាន rate limit, គ្មាន update-check endpoint, preview សំឡេងមិនទាន់មាន UI, គ្មាន timeline view, គ្មាន diagnostic report export, speaker ជ្រើសបានតែ Male/Female (ជ្រើសសំឡេងដំបូងតាមភេទ), timing strategy សម្រាប់សំឡេងវែងមានតែការព្រមាន។
- Transcribe ដំណើរការ synchronous (ឯកសារវែងអាច timeout)។
- Khmer voice មិនធានាថាមាននៅក្នុង edge-tts ទេ — កម្មវិធីបង្ហាញសារ ប្រសិនបើមិនមាន។
