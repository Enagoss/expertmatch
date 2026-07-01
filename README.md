# 🤖 ExpertMatch Web — Publication-Based Expert Ranking

Find the best peer reviewers by analyzing expert publications against your manuscript.

## ⚡ Quick Start

### Option 1: Use Live Version (Easiest)
- Just visit the deployed URL
- No installation needed!
- Works on any browser

### Option 2: Run Locally
```bash
pip install -r requirements_web.txt
python app.py
# Visit http://localhost:5000
```

## 📋 Features

✅ **Publication Analysis** — Analyzes expert publications for relevance
✅ **AI-Powered Matching** — Uses OpenRouter API for intelligent ranking
✅ **Real-time Results** — Get expert rankings in 30-60 seconds
✅ **Export Options** — CSV for QA teams, JSON for analysis
✅ **Subject Areas** — SA1/SA2/SA3 integrated
✅ **Beautiful UI** — Modern, responsive web interface

## 📊 Database

- **156 Expert Reviewers**
- **3,690 Publications**
- **h-index Scores**
- **Institution Data**
- **Subject Area Classifications**

## 🔧 Setup

### What You Need

1. **OpenRouter API Key**
   - Sign up: https://openrouter.ai
   - Get key: https://openrouter.ai/keys
   - Free account with credits

2. **Files in Repo**
   - `app.py` — Flask backend
   - `data.json` — Expert database
   - `requirements_web.txt` — Python dependencies
   - `templates/index.html` — Web interface
   - `Procfile` — Deployment config

### Local Installation

```bash
# 1. Clone repo
git clone https://github.com/YOUR-USERNAME/expertmatch-web.git
cd expertmatch-web

# 2. Install dependencies
pip install -r requirements_web.txt

# 3. Run app
python app.py

# 4. Open browser
# http://localhost:5000
```

## 🚀 Deployment

### Railway (Recommended)

1. Go to https://railway.app
2. Sign up with GitHub
3. Click "New Project"
4. Select "Deploy from GitHub repo"
5. Choose your `expertmatch-web` repo
6. Click "Deploy"
7. Wait 2-3 minutes
8. Get your live URL!

### Render

1. Go to https://render.com
2. Sign up with GitHub
3. Click "New +" → "Web Service"
4. Connect GitHub repo
5. Click "Create Web Service"
6. Wait 3-5 minutes
7. Get your live URL!

## 📖 How to Use

### Step 1: Enter Manuscript Details
- **Title:** Your manuscript title
- **Abstract:** Full manuscript abstract
- **Keywords:** Comma-separated keywords
- **Primary Keywords (optional):** Filter to specific domains

### Step 2: Add API Key
- Get from: https://openrouter.ai/keys
- Paste into "OpenRouter API Key" field

### Step 3: Click "🚀 Rank Experts"
- Wait 30-60 seconds
- Tool analyzes expert publications
- Returns ranked list with reasons

### Step 4: Review Results
- See match score (0-100%)
- Read why each expert matches
- Click DOI links to verify papers
- Review publication details

### Step 5: Export Results
- **CSV** — For QA team (open in Excel)
- **JSON** — For further analysis

## 💡 Understanding Results

### Match Score
- **90-100%:** Excellent match
- **70-89%:** Good match
- **50-69%:** Fair match
- **<50%:** Poor match

### Confidence
- **High:** Expert has 70%+ average publication relevance
- **Medium:** Expert has 50-70% relevance
- **Low:** Expert has <50% relevance

### Publication Relevance
Shows how closely each publication relates to your manuscript (0-100%)

## 🔑 API Key Management

⚠️ **Security:**
- API key never stored
- Only sent for analysis
- Cleared from memory after use
- Use your own OpenRouter account

📍 **Get Free Credits:**
- Sign up at https://openrouter.ai
- Free tier includes API credits
- No credit card required initially

## 📁 File Structure

```
expertmatch-web/
├── app.py                    # Flask backend
├── data.json                 # Expert database (1.8MB)
├── requirements_web.txt      # Python dependencies
├── Procfile                  # Deployment config
├── templates/
│   └── index.html           # Web interface
└── README.md                # This file
```

## ⚙️ Technology Stack

- **Backend:** Flask (Python)
- **Frontend:** HTML5 + CSS3 + JavaScript
- **API:** OpenRouter (LLM)
- **Database:** JSON (embedded)
- **Hosting:** Railway/Render/Heroku

## 🆘 Troubleshooting

### "Failed to fetch" Error
- Make sure Flask backend is running
- Check http://localhost:5000 in browser
- Verify all files are in correct folders

### "API key invalid"
- Go to https://openrouter.ai/keys
- Verify key is correct
- Check account has credits
- Create new key if needed

### No Results Found
- Try without "Primary Keywords" filter
- Use broader keywords
- Check manuscript details are complete

### Slow Response (>2 minutes)
- OpenRouter API might be congested
- Try again in a few minutes
- Check internet connection

## 📊 Cost Estimate

- **Per search:** ~$0.10
- **Per 100 searches:** ~$10
- **Per year (1,000 searches):** ~$100

Using OpenRouter's affordable models

## 📝 Notes

- Analysis time: 30-60 seconds per manuscript
- Experts analyzed: Top 80 (by relevance)
- Publications analyzed: Top 5 per expert
- Results returned: Top 15 matches
- Database updated: As needed

## 🤝 Contributing

Found a bug? Have suggestions?
- Create an issue on GitHub
- Submit a pull request
- Share feedback!

## 📄 License

Open source - feel free to use and modify

## 🎯 Next Steps

1. Get OpenRouter API key (https://openrouter.ai/keys)
2. Deploy to Railway/Render (click deploy button)
3. Visit your live URL
4. Enter manuscript details
5. Click "Rank Experts"
6. Export and review results

---

**Built with ❤️ for finding the best peer reviewers**

Have questions? Check the documentation or create an issue!
