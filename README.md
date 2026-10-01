# internee_intern_support_chatbot_system
# Chatbot for Internship Support

An AI chatbot that answers intern queries in real time and provides support, built with **NLP** and **Hugging Face Transformers**. It combines an internship FAQ document with a large set of historical customer-support tickets to find the best answer for each question.

**Author:** Mahnoor, BS Computer Science, UET Peshawar (Bannu Campus)

---

## 1. Objective

Develop an AI chatbot that automates real-time responses to intern queries (tasks, submissions, deadlines, certificates, technical help) so interns get instant support without waiting for a human.

---

## 2. Key Features

- Understands the **meaning** of a question, not just exact keywords (for example, "how do I hand in my work" matches "How do I submit my task?").
- **Two-level answer strategy:** internship FAQ first, historical support tickets second.
- **Fallback response** that directs the intern to the support team when the bot is not confident.
- Shows the **source and similarity score** of every answer, which makes the bot transparent and easy to debug.
- Runs on a normal CPU and works in **Google Colab**.
- Built-in **evaluation** of retrieval accuracy on held-out data.

---

## 3. Dataset / Data Used

This project does not use a database server. All data is loaded into memory from two sources.

| Data | Source | Format | Size | Role |
|---|---|---|---|---|
| **FAQ documents** | `internship_faq.csv` (written for this project from the internship rules) | CSV (`question`, `answer`) | 55 question-answer pairs | Intern-specific answers: tasks, deadlines, links, intro video, certificate, support |
| **Historical support tickets** | [Bitext Customer Support LLM Chatbot Training Dataset](https://huggingface.co/datasets/bitext/Bitext-customer-support-llm-chatbot-training-dataset) (Hugging Face) | Loaded with the `datasets` library | About 27,000 rows in total, reduced to a medium-sized subset (maximum 150 rows per intent, roughly 4,000 rows) | General support conversations covering many intents (orders, refunds, accounts, complaints, and more) |

**Why a subset?** Keeping at most 150 rows per intent gives a balanced, medium-sized dataset that is fast to encode and does not let any one intent dominate.

Please check the dataset page for its license and citation requirements.

---

## 4. Technologies and Tools

| Category | Tool |
|---|---|
| Language | Python 3 |
| NLP / Transformers | Hugging Face `sentence-transformers` (model: `all-MiniLM-L6-v2`) |
| Data loading | Hugging Face `datasets` |
| Data handling | `pandas`, `numpy` |
| Evaluation | `scikit-learn` (stratified train/test split) |
| Environment | Google Colab / Jupyter Notebook / VS Code |
| Version control | Git and GitHub |

---

## 5. Techniques and Methods

1. **Data cleaning**
   - Removed duplicate questions.
   - Replaced template placeholders such as `{{Order Number}}` with readable text like `[Order Number]`.
2. **Balanced sampling**
   - Shuffled the data and kept up to 150 rows per intent.
3. **Sentence embeddings**
   - Each question is converted into a 384-dimensional vector by a pre-trained Transformer model (`all-MiniLM-L6-v2`).
   - Vectors are normalized, so similarity is a simple dot product.
4. **Semantic search (retrieval-based chatbot)**
   - The user's message is embedded the same way.
   - **Cosine similarity** is computed against all stored questions, and the answer of the closest question is returned.
5. **Confidence thresholds**
   - FAQ answers are used only if similarity is at least `0.55`.
   - Ticket answers are used only if similarity is at least `0.50`.
   - Otherwise the bot returns a safe fallback message.
6. **Evaluation**
   - 20% of the tickets are held out using a stratified split.
   - For each held-out query, the bot finds the nearest training question and we check whether its **intent** matches the true intent (intent accuracy).

### How the pipeline works

```
User question
      |
      v
Sentence-Transformer embedding
      |
      v
Search FAQ (cosine similarity) --- score >= 0.55 ---> FAQ answer
      |
      | below threshold
      v
Search support tickets ----------- score >= 0.50 ---> Ticket answer
      |
      | below threshold
      v
Fallback: "Please contact your mentor or support team."
```

### Why retrieval-based instead of generative?

| Retrieval-based (used here) | Generative |
|---|---|
| Returns pre-written, verified answers | Writes new text, may invent facts |
| Fast, runs on CPU | Slower, needs more compute |
| Easy to control and update (edit the CSV) | Harder to control |

For an internship support bot, correct and consistent answers matter more than creative ones.

---

## 6. Project Structure

```
internship-support-chatbot/
|-- internship_chatbot_colab.ipynb   # Google Colab notebook (run this)
|-- chatbot.py                       # Same project as a local Python script
|-- internship_faq.csv               # FAQ documents
|-- README.md
```

---

## 7. How to Run

### Option A: Google Colab (recommended)

1. Open [colab.research.google.com](https://colab.research.google.com) and choose **File > Upload notebook**.
2. Select `internship_chatbot_colab.ipynb`.
3. Run the cells from top to bottom.
4. When Cell 3 asks, upload `internship_faq.csv` from your computer.
5. Run the last cell and start chatting. Type `quit` to stop.

### Option B: Locally

```bash
pip install datasets sentence-transformers pandas scikit-learn
python chatbot.py            # start the chatbot
python chatbot.py --eval     # run the accuracy check
```

Keep `chatbot.py` and `internship_faq.csv` in the same folder. The dataset and model are downloaded automatically on the first run.

---

## 8. Results

- **Intent accuracy on held-out tickets:** `XX.XX%` (replace with your result from the evaluation cell)
- **Sample conversations:** replace the examples below with your real output.

```
You: How do I submit my task?
Bot: <answer from the FAQ>
     [FAQ | similarity 0.xx]

You: Do I need an intro video?
Bot: <answer from the FAQ>
     [FAQ | similarity 0.xx]

You: What is the capital of France?
Bot: Sorry, I'm not sure about that. Please contact your mentor or the support team and they will help you.
     [fallback | similarity 0.xx]
```

---

## 9. Skills Demonstrated

- Natural Language Processing (NLP)
- Working with pre-trained Transformer models (Hugging Face)
- Semantic similarity and sentence embeddings
- Data cleaning and preprocessing with pandas
- Building and evaluating a retrieval-based chatbot
- Threshold tuning and fallback design
- Python development and working in Google Colab
- Documentation and project structuring with GitHub

---

## 10. Limitations and Future Improvements

- The bot only answers from stored data, so it cannot handle questions that are not covered. Add more FAQ rows to improve coverage.
- The Bitext tickets are general customer-support data, not internship-specific. Real internship tickets would make the ticket search more useful.
- Possible upgrades:
  - Fine-tune the Transformer model on internship-specific questions.
  - Add a **Rasa** pipeline for dialogue management and multi-turn conversations.
  - Build a web interface with **Streamlit** or **Gradio**.
  - Log unanswered questions so the FAQ can be extended automatically.

---

## 11. Credits

- Dataset: Bitext Customer Support LLM Chatbot Training Dataset (Hugging Face)
- Model: `all-MiniLM-L6-v2` from the Sentence-Transformers library

---

`
