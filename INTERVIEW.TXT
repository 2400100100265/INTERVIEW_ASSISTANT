import os
import re
import tempfile
import pandas as pd
import plotly.express as px
import streamlit as st
import google.generativeai as genai

from PyPDF2 import PdfReader
from gtts import gTTS

# -------------------------
# PAGE CONFIG
# -------------------------

st.set_page_config(
    page_title="AI Interview Assistant Pro",
    page_icon="🤖",
    layout="wide"
)

# -------------------------
# API CONFIG
# -------------------------

API_KEY = os.getenv("GEMINI_API_KEY")

if not API_KEY:
    st.error("Set GEMINI_API_KEY environment variable.")
    st.stop()

genai.configure(api_key=API_KEY)

# -------------------------
# SIDEBAR
# -------------------------

st.sidebar.title("Settings")

selected_model = st.sidebar.selectbox(
    "Choose Gemini Model",
    [
        "gemini-2.0-flash",
        "gemini-2.5-flash"
    ]
)

model = genai.GenerativeModel(selected_model)

# -------------------------
# SESSION STATE
# -------------------------

if "questions" not in st.session_state:
    st.session_state.questions = []

if "current_question" not in st.session_state:
    st.session_state.current_question = 0

if "history" not in st.session_state:
    st.session_state.history = []

if "scores" not in st.session_state:
    st.session_state.scores = []

# -------------------------
# PDF READER
# -------------------------

def extract_text_from_pdf(uploaded_file):
    text = ""
    reader = PdfReader(uploaded_file)

    for page in reader.pages:
        content = page.extract_text()
        if content:
            text += content + "\n"

    return text

# -------------------------
# TTS
# -------------------------

def speak_question(question):

    tts = gTTS(question)

    tmp = tempfile.NamedTemporaryFile(
        delete=False,
        suffix=".mp3"
    )

    tts.save(tmp.name)

    return tmp.name

# -------------------------
# GEMINI SAFE CALL
# -------------------------

def generate_response(prompt):

    try:
        response = model.generate_content(prompt)
        return response.text

    except Exception as e:
        return f"Error: {str(e)}"

# -------------------------
# RESUME ANALYSIS
# -------------------------

def analyze_resume(resume_text):

    prompt = f"""
    Analyze this resume.

    Return:

    1. Candidate Summary
    2. Key Skills
    3. Strengths
    4. Weaknesses
    5. Interview Focus Areas

    Resume:

    {resume_text}
    """

    return generate_response(prompt)

# -------------------------
# ATS SCORE
# -------------------------

def ats_score_resume(resume_text, role):

    prompt = f"""
    Act as an ATS system.

    Job Role:
    {role}

    Resume:
    {resume_text}

    Return:

    ATS Score out of 100

    Missing Keywords

    Strengths

    Weaknesses

    Suggestions
    """

    return generate_response(prompt)

# -------------------------
# QUESTION GENERATOR
# -------------------------

def generate_questions(
    resume_text,
    role,
    company,
    difficulty
):

    prompt = f"""
    Candidate Resume:

    {resume_text}

    Target Role:
    {role}

    Target Company:
    {company}

    Difficulty:
    {difficulty}

    Generate 10 interview questions.

    Mix:

    - Technical
    - Behavioral
    - Problem Solving
    - Company Specific

    Return numbered questions only.
    """

    return generate_response(prompt)

# -------------------------
# ANSWER EVALUATION
# -------------------------

def evaluate_answer(question, answer):

    prompt = f"""
    Interview Question:

    {question}

    Candidate Answer:

    {answer}

    Evaluate:

    Technical Accuracy (0-10)

    Communication (0-10)

    Confidence (0-10)

    Give detailed feedback.

    Return:

    Technical:
    Communication:
    Confidence:
    Overall:

    Feedback:
    """

    return generate_response(prompt)

# -------------------------
# TITLE
# -------------------------

st.title("🤖 AI Interview Assistant Pro")

st.markdown(
    "Upload your resume and practice AI interviews."
)

# -------------------------
# INPUTS
# -------------------------

uploaded_file = st.file_uploader(
    "Upload Resume",
    type=["pdf", "txt"]
)

role = st.text_input(
    "Target Role",
    "Software Engineer"
)

company = st.text_input(
    "Target Company",
    "Google"
)

difficulty = st.selectbox(
    "Difficulty",
    ["Easy", "Medium", "Hard"]
)

# -------------------------
# RESUME LOAD
# -------------------------

if uploaded_file:

    if uploaded_file.type == "application/pdf":
        resume_text = extract_text_from_pdf(
            uploaded_file
        )
    else:
        resume_text = uploaded_file.read().decode(
            "utf-8"
        )

    if not resume_text.strip():
        st.error("Could not extract text.")
        st.stop()

    st.success("Resume Loaded")

    with st.expander("Resume Preview"):

        st.text_area(
            "",
            resume_text[:5000],
            height=250
        )

    # ---------------------
    # ANALYSIS
    # ---------------------

    col1, col2 = st.columns(2)

    with col1:

        if st.button("Analyze Resume"):

            with st.spinner("Analyzing..."):

                result = analyze_resume(
                    resume_text
                )

            st.subheader("Resume Analysis")
            st.markdown(result)

    with col2:

        if st.button("ATS Resume Score"):

            with st.spinner("Scoring Resume..."):

                result = ats_score_resume(
                    resume_text,
                    role
                )

            st.subheader("ATS Report")
            st.markdown(result)

    # ---------------------
    # GENERATE QUESTIONS
    # ---------------------

    if st.button("Generate Interview Questions"):

        with st.spinner(
            "Generating Questions..."
        ):

            raw = generate_questions(
                resume_text,
                role,
                company,
                difficulty
            )

        questions = []

        for line in raw.split("\n"):

            line = line.strip()

            if len(line) > 5:
                questions.append(line)

        st.session_state.questions = questions
        st.session_state.current_question = 0

        st.success("Questions Generated")

# -------------------------
# INTERVIEW
# -------------------------

if st.session_state.questions:

    idx = st.session_state.current_question

    if idx < len(st.session_state.questions):

        question = st.session_state.questions[idx]

        st.header(
            f"Question {idx+1}/{len(st.session_state.questions)}"
        )

        st.info(question)

        audio_file = speak_question(question)

        st.audio(audio_file)

        answer = st.text_area(
            "Your Answer",
            height=200,
            key=f"ans_{idx}"
        )

        c1, c2 = st.columns(2)

        with c1:

            if st.button("Evaluate Answer"):

                if answer.strip():

                    result = evaluate_answer(
                        question,
                        answer
                    )

                    st.markdown(result)

                    scores = re.findall(
                        r'\b([0-9]|10)\b',
                        result
                    )

                    if len(scores) >= 3:

                        st.session_state.scores.append(
                            {
                                "Technical":
                                int(scores[0]),

                                "Communication":
                                int(scores[1]),

                                "Confidence":
                                int(scores[2])
                            }
                        )

                    st.session_state.history.append(
                        {
                            "question": question,
                            "answer": answer,
                            "feedback": result
                        }
                    )

                else:
                    st.warning(
                        "Enter an answer."
                    )

        with c2:

            if st.button("Next Question"):

                st.session_state.current_question += 1
                st.rerun()

# -------------------------
# DASHBOARD
# -------------------------

if st.session_state.scores:

    st.header("Performance Dashboard")

    df = pd.DataFrame(
        st.session_state.scores
    )

    st.dataframe(df)

    fig = px.line(
        df,
        title="Performance Trend"
    )

    st.plotly_chart(
        fig,
        use_container_width=True
    )

# -------------------------
# HISTORY
# -------------------------

if st.session_state.history:

    st.header("Interview History")

    for item in st.session_state.history:

        with st.expander(
            item["question"]
        ):

            st.write(
                "**Answer:**"
            )

            st.write(
                item["answer"]
            )

            st.write(
                "**Feedback:**"
            )

            st.markdown(
                item["feedback"]
            )

# -------------------------
# COMPLETION
# -------------------------

if (
    st.session_state.questions
    and
    st.session_state.current_question
    >= len(st.session_state.questions)
):

    st.success(
        "🎉 Interview Completed"
    )

    st.balloons()

    st.subheader(
        "Final Summary"
    )

    st.metric(
        "Questions Answered",
        len(
            st.session_state.history
        )
    )
