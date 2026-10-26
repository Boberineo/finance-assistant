# ISYS2001-A2-finance-assistant

A finance assistant for a uni student who ran out of money before payday. Upload a month of bank transactions to see where the money went, then ask Penny, a Gemini-powered money coach, questions about your own numbers.

### How To Run

Get a free Gemini API key from Google AI Studio.

Open `spending_analyser.ipynb` in Colab.

Add the key in Colab Secrets. Name it `GOOGLE_API_KEY` and turn on Notebook access.

Runtime -> Run all, then open the `gradio.live` link printed near the end.

Upload a CSV (try `data/transactions_september.csv`) and ask Penny a question.

### Input 

A CSV with the columns `date, category, amount, description`. Spending is negative, income is positive.

### What's In The Repo

`spending_analyser.ipynb` - the full project, including planning, code, the app and 31 tests

`DIARY.md` - developer's diary

`data/` - sample months plus broken files for testing
