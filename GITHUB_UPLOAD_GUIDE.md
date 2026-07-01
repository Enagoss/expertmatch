# 📤 GitHub Upload Guide - All Files You Need

## 📋 Complete File List

Upload these **6 files** to GitHub:

### Root Level Files (Upload directly to repo root)
1. ✅ `app.py` — Flask backend server
2. ✅ `data.json` — Expert database (1.8MB)
3. ✅ `requirements_web.txt` — Python dependencies
4. ✅ `Procfile` — Deployment configuration
5. ✅ `README.md` — Documentation
6. ✅ `.gitignore` — Files to ignore

### Folder: `templates/` (Create folder, then upload inside)
7. ✅ `templates/index.html` — Web interface

---

## 🎯 Step-by-Step GitHub Upload

### Step 1: Create GitHub Repository

Go to: https://github.com/new

Fill in:
- **Repository name:** `expertmatch-web`
- **Description:** Publication-Based Expert Ranking System
- **Public/Private:** Public (so Railway can see it)
- Click **"Create repository"**

---

### Step 2: Upload Files to GitHub

#### Option A: Web Upload (Easiest - Recommended)

**Step 2a: Upload Root Files**

1. Go to your new repo: https://github.com/YOUR-USERNAME/expertmatch-web

2. Click **"Add file"** → **"Upload files"**

3. Drag and drop these files to browser:
   ```
   ✓ app.py
   ✓ data.json
   ✓ requirements_web.txt
   ✓ Procfile
   ✓ README.md
   ✓ .gitignore
   ```

4. Click **"Commit changes"**

**Step 2b: Create templates/ Folder**

1. Click **"Add file"** → **"Create new file"**

2. In filename field, type:
   ```
   templates/index.html
   ```

3. GitHub automatically creates the `templates/` folder!

4. Paste the HTML code (see below) into the editor

5. Click **"Commit new file"**

---

#### Option B: Git Command Line

```bash
# Clone your new repo
git clone https://github.com/YOUR-USERNAME/expertmatch-web.git
cd expertmatch-web

# Copy these files to this folder:
# app.py
# data.json
# requirements_web.txt
# Procfile
# README.md
# .gitignore
# templates/index.html (in templates/ subfolder)

# Upload to GitHub
git add .
git commit -m "Add ExpertMatch web app"
git push origin main
```

---

## 📄 File Contents

### 1️⃣ app.py
```python
#!/usr/bin/env python3
"""
ExpertMatch Web App - OPTIMIZED VERSION
"""

from flask import Flask, render_template, request, jsonify
from flask_cors import CORS
import json
import requests

app = Flask(__name__)
CORS(app)

# Load expert data
EXPERTS = []

def load_experts():
    global EXPERTS
    try:
        with open('data.json', 'r') as f:
            data = json.load(f)
            EXPERTS = data.get('experts', data)
        print(f"✓ Loaded {len(EXPERTS)} experts")
    except Exception as e:
        print(f"❌ Error loading data.json: {e}")
        EXPERTS = []

load_experts()

OPENROUTER_API = "https://openrouter.ai/api/v1/chat/completions"

def call_openrouter(prompt: str, api_key: str, timeout: int = 12) -> str:
    """Call OpenRouter API with short timeout"""
    headers = {
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json",
    }
    
    data = {
        "model": "meta-llama/llama-2-70b-chat",
        "messages": [{"role": "user", "content": prompt}],
        "max_tokens": 800,
        "temperature": 0.5
    }
    
    try:
        response = requests.post(OPENROUTER_API, headers=headers, json=data, timeout=timeout)
        response.raise_for_status()
        result = response.json()
        return result['choices'][0]['message']['content']
    except requests.exceptions.Timeout:
        return None
    except Exception as e:
        return None

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/api/rank', methods=['POST'])
def rank_experts():
    """Rank experts - OPTIMIZED for speed"""
    data = request.json
    
    api_key = data.get('api_key', '').strip()
    title = data.get('title', '').strip()
    abstract = data.get('abstract', '').strip()
    keywords = data.get('keywords', '').strip()
    primary_kw = data.get('primary_keywords', '').strip()
    
    # Validate
    if not api_key:
        return jsonify({'error': 'API key required'}), 400
    if not title or not abstract:
        return jsonify({'error': 'Title and abstract required'}), 400
    if not keywords:
        return jsonify({'error': 'Keywords required'}), 400
    
    # Filter by primary keywords
    qualified = EXPERTS
    if primary_kw:
        pkws = [k.strip().lower() for k in primary_kw.split(',')]
        qualified = [
            e for e in EXPERTS
            if any(
                kw in (e.get('okw', '') + ' ' + e.get('inst', '')).lower()
                for kw in pkws
            )
        ]
    
    if not qualified:
        return jsonify({'error': 'No experts match primary keywords'}), 400
    
    rankings = []
    
    # Process experts - OPTIMIZED: max 80 experts, top 5 pubs each
    for idx, expert in enumerate(qualified[:80]):
        expert_code = expert.get('c', '')
        expert_name = f"{expert.get('fn', '')} {expert.get('ln', '')}".strip()
        pubs = expert.get('pubs', [])
        
        if len(pubs) < 2:
            continue
        
        # ONLY ANALYZE TOP 5 PUBLICATIONS
        top_pubs = pubs[:5]
        
        pub_list = [{'title': p.get('t', '')[:60], 'year': p.get('y', '')} for p in top_pubs]
        
        # FAST PROMPT
        prompt = f"""Rate each publication 0-100 for relevance to: {title[:60]}
Keywords: {keywords}

Publications: {json.dumps(pub_list)}

Return ONLY: [{{"title":"...", "score": 75}}]"""
        
        response = call_openrouter(prompt, api_key, timeout=12)
        
        if not response:
            continue
        
        try:
            import re
            match = re.search(r'\[.*\]', response, re.DOTALL)
            if not match:
                continue
            pub_scores = json.loads(match.group())
            if not pub_scores:
                continue
        except:
            continue
        
        # Calculate
        relevant = sum(1 for p in pub_scores if p.get('score', 0) >= 50)
        avg_rel = sum(p.get('score', 0) for p in pub_scores) / len(pub_scores) if pub_scores else 0
        
        if relevant == 0:
            continue
        
        primary_bonus = 20 if primary_kw and any(kw in (expert.get('okw', '') + expert.get('inst', '')).lower() for kw in [k.strip().lower() for k in primary_kw.split(',')]) else 0
        sa_bonus = 10 if expert.get('sa2') or expert.get('sa3') else 0
        
        final_score = min(100, (relevant * 12 + avg_rel / 2 + (primary_bonus + sa_bonus) / 5))
        
        confidence = 'High' if avg_rel >= 70 else 'Medium' if avg_rel >= 50 else 'Low'
        
        # Top publications
        top_pubs_result = []
        for i, pub_score in enumerate(pub_scores[:3]):
            if pub_score.get('score', 0) >= 60 and i < len(top_pubs):
                pub = top_pubs[i]
                top_pubs_result.append({
                    'title': pub.get('t', '')[:100],
                    'year': pub.get('y', ''),
                    'score': pub_score.get('score', 0),
                    'reason': 'Matches manuscript topic',
                    'doi': pub.get('d', ''),
                    'citations': pub.get('c', 0)
                })
        
        rankings.append({
            'code': expert_code,
            'name': expert_name,
            'match_score': round(final_score),
            'confidence': confidence,
            'h_index': expert.get('hi', ''),
            'institution': expert.get('inst', ''),
            'sa1': expert.get('sa1', ''),
            'sa2': expert.get('sa2', ''),
            'sa3': expert.get('sa3', ''),
            'total_publications': len(pubs),
            'relevant_publications': relevant,
            'avg_relevance': round(avg_rel),
            'top_matching_publications': top_pubs_result,
            'reasons': [
                f"{relevant} of {len(pubs)} publications match",
                f"Relevance: {round(avg_rel)}%",
                f"Expertise: {expert.get('sa2', 'Varied')}"
            ]
        })
    
    rankings.sort(key=lambda x: -x['match_score'])
    
    return jsonify({
        'rankings': rankings[:15],
        'stats': {
            'total_experts': len(EXPERTS),
            'qualified': len(qualified),
            'analyzed': len(rankings),
            'top_match': rankings[0]['match_score'] if rankings else 0
        }
    })

@app.route('/api/stats', methods=['GET'])
def get_stats():
    total_pubs = sum(len(e.get('pubs', [])) for e in EXPERTS)
    return jsonify({
        'total_experts': len(EXPERTS),
        'total_publications': total_pubs,
        'experts_with_sa1': sum(1 for e in EXPERTS if e.get('sa1')),
        'experts_with_sa2': sum(1 for e in EXPERTS if e.get('sa2')),
        'experts_with_sa3': sum(1 for e in EXPERTS if e.get('sa3')),
    })

if __name__ == '__main__':
    app.run(debug=True, port=5000)
```

---

### 2️⃣ requirements_web.txt
```
flask==2.3.0
flask-cors==4.0.0
requests==2.31.0
```

---

### 3️⃣ Procfile
```
web: python app.py
```

---

### 4️⃣ .gitignore
```
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
ENV/
.vscode/
.idea/
.DS_Store
*.egg-info/
dist/
build/
.env
.env.local
```

---

### 5️⃣ data.json
**Already in your outputs folder** — Just upload the file directly

File should be ~1.8MB with 156 experts and their data

---

### 6️⃣ README.md
**See the README.md file provided** — Copy and paste its contents

---

### 7️⃣ templates/index.html
**See the HTML file provided** — Copy and paste its contents

When uploading via web:
- Click "Add file" → "Create new file"
- Type filename: `templates/index.html`
- GitHub auto-creates the templates/ folder
- Paste HTML contents
- Commit

---

## ✅ After Upload - What Your Repo Should Look Like

```
expertmatch-web/
├── app.py                    ✓
├── data.json                 ✓
├── requirements_web.txt      ✓
├── Procfile                  ✓
├── README.md                 ✓
├── .gitignore                ✓
└── templates/
    └── index.html            ✓
```

---

## 🚀 Next: Deploy to Railway

Once all files are on GitHub:

1. Go to: https://railway.app
2. Sign up with GitHub
3. Click "New Project"
4. Select "Deploy from GitHub repo"
5. Choose `expertmatch-web`
6. Click "Deploy"
7. Wait 2-3 minutes
8. Get your live URL!

---

## 📋 Quick Checklist

Before uploading:
- [ ] All 7 files downloaded/created
- [ ] Files ready in one folder locally
- [ ] GitHub account created
- [ ] New repo created (`expertmatch-web`)
- [ ] Ready to upload

During upload:
- [ ] Upload 6 root files (app.py, data.json, etc.)
- [ ] Create templates/ folder
- [ ] Upload index.html into templates/
- [ ] Verify all files appear in repo

After upload:
- [ ] Go to Railway.app
- [ ] Deploy from GitHub
- [ ] Get live URL
- [ ] Test in browser!

---

## 🎯 Files Download Links

All files are ready in `/mnt/user-data/outputs/`:

- `app.py` ✓
- `data.json` ✓
- `requirements_web.txt` ✓
- `Procfile` ✓
- `README.md` ✓
- `.gitignore` ✓
- `templates/index.html` ✓

**Just download and upload to GitHub!**

---

**Ready to upload? Follow the steps above!** 🚀
