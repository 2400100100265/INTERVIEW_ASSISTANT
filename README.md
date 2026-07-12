# 🤖 AI Interview Assistant Pro

![Python](https://img.shields.io/badge/Python-3.10-blue)
![Streamlit](https://img.shields.io/badge/Streamlit-App-red)
![Gemini](https://img.shields.io/badge/Google-Gemini_AI-green)

An AI-powered interview preparation platform that analyzes resumes, generates personalized interview questions, evaluates candidate answers, and tracks performance using Google Gemini AI.

---

## 📸 Application Preview

### Home Page
<img width="1247" height="810" alt="Screenshot 2026-07-12 120913" src="https://github.com/user-attachments/assets/ac952f05-62e0-47be-9bed-311662594967" />



---

### Resume Analysis



---

### ATS Resume Score

![ATS Score](screenshots/ats_score.png)

---

### Interview Session

![Interview Session](screenshots/interview.png)

---

### Performance Dashboard

![Dashboard](screenshots/dashboard.png)

---

# 🚀 Features

✅ Resume Analysis

✅ ATS Resume Scoring

✅ Company Specific Interview Questions

✅ AI Answer Evaluation

✅ Voice-Based Question Reading

✅ Interview History Tracking

✅ Performance Dashboard

✅ Multi-Model Gemini Support

---

# 🏗 System Architecture

```text
Resume Upload
      │
      ▼
Resume Parser
(PyPDF2)
      │
      ▼
Gemini AI
      │
 ┌─────────────┐
 │ Analysis    │
 └─────────────┘
      │
      ▼
Question Generator
      │
      ▼
Interview Session
      │
      ▼
Answer Evaluation
      │
      ▼
Performance Dashboard
```

---

# 🛠 Tech Stack

| Technology | Purpose |
|------------|----------|
| Python | Backend |
| Streamlit | Frontend |
| Gemini AI | LLM |
| PyPDF2 | Resume Parsing |
| Plotly | Dashboard |
| Pandas | Data Analysis |
| gTTS | Text-to-Speech |

---

# ⚙ Installation

## Clone Repository

```bash
git clone https://github.com/yourusername/AI-Interview-Assistant-Pro.git

cd AI-Interview-Assistant-Pro
```

## Install Dependencies

```bash
pip install -r requirements.txt
```

## Configure Gemini API

Linux / Mac

```bash
export GEMINI_API_KEY=your_api_key
```

Windows

```bash
set GEMINI_API_KEY=your_api_key
```

## Run

```bash
streamlit run app.py
```

---

# 🎯 Workflow

1. Upload Resume
2. Analyze Resume
3. Check ATS Score
4. Generate Questions
5. Answer Questions
6. Receive AI Feedback
7. View Dashboard
8. Review History

---

# 🔮 Future Scope

- Speech-to-Text Answers
- Video Interview Simulation
- Eye Contact Detection
- Emotion Recognition
- RAG-Based Company Interviews
- OpenAI Integration
- Claude Integration
- PDF Report Generation

---

# 📚 Learning Outcomes

- Generative AI Integration
- Prompt Engineering
- Resume Parsing
- Streamlit Development
- Data Visualization
- LLM-Based Evaluation Systems

---

# 👨‍💻 Author

**Madhvendra Pandey**

Computer Science Engineering

AI Enthusiast | Content Creator | Python Developer

---
