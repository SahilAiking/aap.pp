import streamlit as st
import google.generativeai as genai
from PIL import Image

# ---------------- PAGE CONFIG ----------------
st.set_page_config(
    page_title="NoorSNK.AI",
    page_icon="🧠",
    layout="centered"
)

# ---------------- UI STYLE ----------------
st.markdown("""
<style>
body {background-color: #0e1117;}
.card {
    background-color: #161b22;
    padding: 20px;
    border-radius: 12px;
    margin-bottom: 15px;
}
.small-btn button {
    width: 100%;
}
</style>
""", unsafe_allow_html=True)

st.markdown("<h1 style='text-align:center;'>NoorSNK.AI 🧠</h1>", unsafe_allow_html=True)
st.markdown("<p style='text-align:center;'>Education • Career • Fitness (Professional AI)</p>", unsafe_allow_html=True)

# ---------------- GEMINI SETUP ----------------
genai.configure(api_key=st.secrets["GEMINI_API_KEY"])
model = genai.GenerativeModel("gemini-1.5-flash")

# ---------------- SESSION STATE ----------------
if "logged_in" not in st.session_state:
    st.session_state.logged_in = False

if "memory" not in st.session_state:
    st.session_state.memory = {}

if "last_answer" not in st.session_state:
    st.session_state.last_answer = ""

# ---------------- MASTER PROMPT ----------------
def get_master_prompt(memory):
    return f"""
Tum NoorSNK.AI ho.
Tum ek PROFESSIONAL, DISCIPLINED, ADVANCED AI MODEL ho.

Tumhara kaam:
- Exact
- Point-to-point
- High-value jawab dena

User Context (Always Follow):
{memory}

Rules:
- Zero bakchodi
- Clear steps
- Hinglish / Simple Hindi
- Memory follow karo
- Unsafe sawal par strict rejection

Image Mode:
- Question paper / notes / diagram detect karo
- Exam-oriented step-by-step solution do

Education Mode:
- Class + Stream ke bahar ka content mat do
- Daily / Weekly / Monthly roadmap do

Fitness Mode:
- Beginner / Intermediate / Advanced
- Daily / Weekly / Monthly plan

Career Mode:
- Steps
- Skills
- Time estimate

Safety Reply (ONLY THIS):
“Mujhe is question ke related train nahi kiya gaya hai.
Agar aapka koi aur sawal ho (education, career, fitness, job),
to main uska answer de sakta hoon.”
"""

# ---------------- LOGIN UI ----------------
if not st.session_state.logged_in:
    st.markdown("<div class='card'>", unsafe_allow_html=True)
    st.subheader("Signup / Login")

    name = st.text_input("Name")
    user_class = st.selectbox("Class", ["8th","10th","12th","Dropper"])
    stream = st.selectbox("Stream", ["Science","Medical","Non-Medical","Commerce","Arts"])
    goal = st.selectbox("Goal", ["Board","Competitive","Career"])

    if st.button("Enter NoorSNK.AI"):
        if name:
            st.session_state.logged_in = True
            st.session_state.memory = {
                "Name": name,
                "Class": user_class,
                "Stream": stream,
                "Goal": goal
            }
            st.success("Access Granted")
        else:
            st.error("Name required")
    st.markdown("</div>", unsafe_allow_html=True)

# ---------------- MAIN AI INTERFACE ----------------
else:
    # -------- TOP BAR --------
    col1, col2, col3 = st.columns([2, 1, 1])

    with col1:
        st.markdown(f"### Welcome {st.session_state.memory['Name']}")

    with col2:
        if st.button("🧹 Clear Chat"):
            st.session_state.last_answer = ""
            st.success("Chat cleared")

    with col3:
        if st.button("🚪 Logout"):
            st.session_state.clear()
            st.experimental_rerun()

    # -------- PROFILE SECTION --------
    with st.expander("👤 Profile / Account"):
        st.write("**Name:**", st.session_state.memory.get("Name"))
        st.write("**Class:**", st.session_state.memory.get("Class"))
        st.write("**Stream:**", st.session_state.memory.get("Stream"))
        st.write("**Goal:**", st.session_state.memory.get("Goal"))

    # -------- MAIN CARD --------
    st.markdown("<div class='card'>", unsafe_allow_html=True)

    uploaded_file = st.file_uploader(
        "➕ Upload Image (Question / Notes / Diagram)",
        type=["jpg","jpeg","png"]
    )

    question = st.text_input("Apna sawal likho")

    if st.button("Ask NoorSNK.AI"):
        if question or uploaded_file:
            master_prompt = get_master_prompt(st.session_state.memory)
            final_input = master_prompt + "\n\nUser Question:\n" + question

            if uploaded_file:
                image = Image.open(uploaded_file)
                response = model.generate_content([final_input, image])
            else:
                response = model.generate_content(final_input)

            st.session_state.last_answer = response.text

    if st.session_state.last_answer:
        st.markdown("### AI Answer")
        st.write(st.session_state.last_answer)

        st.markdown("#### 📋 Copy Answer")
        st.code(st.session_state.last_answer, language="text")

    st.markdown("</div>", unsafe_allow_html=True)

# ---------------- FOOTER ----------------
st.markdown("---")
st.caption("NoorSNK.AI © 2026")
