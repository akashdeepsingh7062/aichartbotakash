# aichartbotakash
chatbot
import streamlit as st
from groq import Groq

# ---------------- PAGE CONFIG ----------------
st.set_page_config(
    page_title="AI Chatbot",
    page_icon="🤖",
    layout="centered"
)

# ---------------- API KEY ----------------
# OPTION 1: Paste your Groq API key below.
# Example: GROQ_API_KEY = "gsk_your_actual_key"
GROQ_API_KEY = ""

# OPTION 2: Recommended - use Streamlit secrets.
# If the key above is empty, the app checks secrets
# and then allows you to enter the key in the sidebar.

# ---------------- CUSTOM CSS ----------------
st.markdown("""
<style>
    .stApp {
        background-color: #0e1117;
        color: #fafafa;
    }
    .main-title {
        text-align: center;
        font-size: 38px;
        font-weight: 800;
        color: #60a5fa;
        margin-bottom: 0;
    }
    .subtitle {
        text-align: center;
        color: #a1a1aa;
        margin-bottom: 25px;
    }
    [data-testid="stChatMessage"] {
        border-radius: 14px;
        padding: 12px;
    }
</style>
""", unsafe_allow_html=True)

# ---------------- SIDEBAR ----------------
with st.sidebar:
    st.title("⚙️ Settings")

    st.markdown("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")

    # Prefer the key entered directly in the code.
    api_key ="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"

    # Otherwise, try Streamlit secrets.
    if not api_key:
        try:
            api_key = st.secrets.get("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", "")
        except Exception:
            api_key = ""

    # Allow entering a key without editing the code.
    if not api_key:
        api_key =  api_key

    model = st.selectbox(
        "Choose AI model",
        [
            "openai/gpt-oss-20b",
            "openai/gpt-oss-120b"
        ],
        index=0
    )

    temperature = st.slider(
        "Creativity",
        min_value=0.0,
        max_value=1.5,
        value=0.7,
        step=0.1
    )

    max_tokens = st.slider(
        "Maximum response length",
        min_value=256,
        max_value=4096,
        value=2048,
        step=256
    )

    if st.button("🗑️ Clear Chat", use_container_width=True):
        st.session_state.messages = []
        st.rerun()

    st.markdown("---")
    st.caption("Powered by Streamlit + Groq")
    st.caption("Your API key should remain private.")

# ---------------- MAIN UI ----------------
st.markdown(
    '<div class="main-title">🤖 AI Chatbot</div>',
    unsafe_allow_html=True
)
st.markdown(
    '<div class="subtitle">Your intelligent AI assistant, powered by Groq</div>',
    unsafe_allow_html=True
)

if "messages" not in st.session_state:
    st.session_state.messages = []

# Welcome message
if not st.session_state.messages:
    st.info(
        "👋 Hello! I'm your AI assistant. "
        "Ask me anything to get started."
    )

# Display existing conversation
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

# ---------------- CHAT INPUT ----------------
prompt = st.chat_input("Message your AI assistant...")

if prompt:
    # Display and save user message
    st.session_state.messages.append({
        "role": "user",
        "content": prompt
    })

    with st.chat_message("user"):
        st.markdown(prompt)

    # Check API key
    if not api_key:
        st.error(
            "Please add your Groq API key in the sidebar "
            "or in the GROQ_API_KEY variable in the code."
        )
    else:
        try:
            client = Groq(api_key=api_key)

            with st.chat_message("assistant"):
                with st.spinner("Thinking..."):
                    response = client.chat.completions.create(
                        model=model,
                        messages=[
                            {
                                "role": "system",
                                "content": (
                                    "You are a helpful, friendly AI "
                                    "assistant. Give accurate, clear "
                                    "and well-formatted answers."
                                )
                            },
                            *st.session_state.messages
                        ],
                        temperature=temperature,
                        max_tokens=max_tokens
                    )

                    answer = response.choices[0].message.content
                    answer = answer or "I couldn't generate a response."

                st.markdown(answer)

            # Save AI response
            st.session_state.messages.append({
                "role": "assistant",
                "content": answer
            })

        except Exception as e:
            st.error(f"Groq API error: {e}")
            st.info(
                "Check your API key, internet connection, "
                "model availability, and Groq usage limits."
            )
