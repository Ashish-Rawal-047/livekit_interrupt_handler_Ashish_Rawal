# 🗣️ LiveKit Agent: STT Filler Word Filtering

This project demonstrates a **LiveKit Agent (v1.2.18+)** implementation that filters out common **filler words** at the Speech-to-Text (STT) level — preventing them from being sent to the LLM for processing.

---

## 🚀 Overview

The primary change is the **override of the `stt_node` method** within the `MyAgent` class.

This new implementation **intercepts transcription events** from the default STT service and filters out filler words in real-time.

```python
async def stt_node(self, audio, model_settings):
    se = Agent.default.stt_node(self, audio, model_settings)
    async for event in se:
        if event.type == "final_transcript":

            ## List of filler words — modify as needed
            filler_words = ["umm", "uh", "um", "haan", "hmm", " i know "]
            text = event.alternatives[0].text.lower()

            if any(filler in text for filler in filler_words):
                print("found filler words, filtering...")
            else:
                yield event
                
    yield se
```

---

## 🧠 Logic Overview

1. Receives all speech events (`se`) from the default STT node: `Agent.default.stt_node`.
2. Inspects each `final_transcript` event.
3. Converts transcript text to lowercase.
4. Checks if any word in `filler_words` appears in the text.
5. **If filler word found:** prints “filtering” and discards the event.
6. **If no filler word found:** yields the event to the LLM for normal processing.

---

## ✅ What Works

- **Filler Word Rejection:**  
  When a user says only a filler word (e.g., “umm”, “hmm”), the agent detects and discards it correctly.

- **Valid Speech Handling:**  
  When a user speaks a valid sentence (e.g., “What’s the weather in London?”), the transcript is processed normally, and the agent responds.

---

## ⚠️ Known Issues

| Issue | Description |
|--------|-------------|
| **Whole-Sentence Filtering** | If a filler word appears anywhere in a sentence (e.g., “Uh, what’s the weather?”), the entire sentence is discarded. |
| **Simple Substring Match** | Using `filler in text` can cause false positives (e.g., “um” in “drum” or “summary”). |
| **Hardcoded List** | The `filler_words` list is static and must be manually updated for more comprehensive filtering. |

---

## 🧪 Steps to Test

### 1. Environment Setup

Ensure you have Python **3.10+** installed.  
Create a `requirements.txt` file with the following:

```
livekit-agents
python-dotenv
```

Then install dependencies:

```bash
pip install -r requirements.txt
```

---

### 2. Configuration

Create a `.env` file in the same directory with your API keys:

```
# For 'openai/gpt-4.1-mini' LLM
OPENAI_API_KEY="sk-..."

# For 'assemblyai/universal-streaming:en' STT
ASSEMBLYAI_API_KEY="..."
```

---

### 3. Run the Agent

Start the agent from your terminal:

```bash
python -m examples.voice_agents.basic_agent console
```

Wait for the **“Livekit Agents - Console”** message to appear.

---

## 🎙️ Test Scenarios

### 🧩 Test Case 1: Filler Word Only
**Action:** Say “umm” or “hmm”  
**Expected Result:**  
Console prints `found filler words, filtering...`  
Agent does **not** respond.

---

### ☀️ Test Case 2: Valid Speech Only
**Action:** Say “What is the weather in London?”  
**Expected Result:**  
Agent responds, e.g., “Sunny with a temperature of 70 degrees.”

---

### ⚡ Test Case 3: Mixed Speech (Known Issue)
**Action:** Say “Uhh, what’s the weather?”  
**Expected Result:**  
Console prints `found filler words, filtering...`  
Agent does **not** respond (entire transcript discarded).

---

## 🧩 Improvements for Future Versions

- Implement **token-level filtering** to remove only the filler word, not the entire sentence.
- Use **regex or NLP models** to detect filler usage more accurately.
- Allow dynamic updates to the filler word list from a configuration file or dashboard.

---

## 🧾 License

This project is distributed under the **MIT License**.

---

**Author:** Ashish Rawal  
**Version:** 1.0  
**Last Updated:** November 2025
