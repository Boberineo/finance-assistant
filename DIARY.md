# Developer's Diary

### Notebook Created

Before I even start this diary I want to say for the record that I understand this is supposed to be a record over many weeks. However, because of my personal circumstances, I have only been able to start this assignment recently. So I'm basically doing this all in one go especially since I'm over due. Also, for the sake of declaring AI, I used AI quite heavily to complete this app. The method I use usually goes something like me starting with the frame work of the code, then I continue until I get stuck and ask AI what direction I should go from there and if it can give me any suggestions on what to add next. If I need more help, I may ask the AI to make me the next part of what I'm doing like if it can make me the next function I'm working on, but I make sure to completely learn what it gives me before I add it to the notebook. But, I made sure to understand everything my AI assistant gave me and to add as much of my own input into the code as possible before I have AI introduce more of its code. This makes sure I can still explain any line of code for the last assignment(A3).

So I'm going to try and get this A2 project done quickly since my schedule is super backed up. I'll make a spending analyzer for a uni student. Upload a month of transactions, see totals and percentages per category, and chat with a Gemini finance assistant about the results. This is closest to what I've already done in previous modules so it’s achievable in the time I got.

I will need a Gemini API key for this project so I got one from Google AI studio. Had a problem with my account being not verified as 18+ so I fixed that and got the key. I had AI make code to test the key in a test notebook and it worked fine. 
  
I chose this app to be a spending analyzer for a university student because I am most familiar with how to code this from previous modules, and it should be a straight forward enough topic to make a good project out of. 

1. Plan and steps 1-4 before any code.

What I asked the AI:

"I'm a beginner. I've covered variables, if/elif, loops and lists, functions, dictionaries, pandas basics, the Gemini API, Gradio Blocks, try/except and assert tests. Keep any suggestions to those. I'm building a Student Spending Analyser for a uni student on a part time wage who ran out of money before payday: upload a CSV (date, category, amount, description; spending negative, income positive), show totals and % per category, then chat with Gemini about it. Here's my step 3 worked example and step 4 pseudocode [pasted]. Can you check my arithmetic and tell me what I've missed?"
I listed what I've covered on purpose. Module 3 warns that AI over engineers unless you set constraints.
What it returned (key parts):

"Your arithmetic is correct: 54 / 93 = 58.06%, 27 / 93 = 29.03%, 12 / 93 = 12.90%, and they sum to 100.0. A few gaps: (1) what happens if a month has income but no spending? Your percentage line divides by total_spent. (2) Category names from bank exports are often inconsistent ('Food' vs 'food '). (3) You might also want to add a monthly budget feature where the student sets a limit per category and the app warns them when they go over, plus a matplotlib bar chart of spending by category."

What I kept:

Point 1. I added a divide by zero guard to the pseudocode.

Point 2. Tidying the category text (strip + Title Case) went into the tool's pseudocode, so 'takeaway ' and 'Takeaway' count as one category. Both points went on my step 2 list of things to test.

What I rejected and why:

The budget limit feature and the bar chart. I know how to do a bar chart from Module 7, but both are extra outputs, and the checklist only needs one custom tool done well. My user's problem is "I don't know where my money went", and the category table answers that directly. I'd rather finish all the steps properly than half build extras.

2. Loading and checking the CSV

What I asked the AI:

"Write a Python function that loads a CSV of transactions with pandas and validates it."

What it returned:

 ```python
 def load_and_validate(path):
     df = pd.read_csv(path, sep=None, engine="python")
     df = df.dropna()
     assert set(["date","category","amount","description"]).issubset(df.columns)
     return df
 ```

What I rejected and why:

`sep=None, engine="python"` makes pandas guess the separator. I couldn't explain how it guesses, and my files are always comma-separated, so it's a black box I don't need.

`df.dropna()` drops a row if any column is blank. That would throw away a real $8 spend just because the category was empty.

Using `assert` to check the user's file is wrong for this job, it crashes the app with an `AssertionError`. Module 11 uses asserts for my tests. For user input I need a message they can understand.

3. Custom tool

What I was trying to do: write my own function that adds up spending per category and works out each category's share of total spending, matching my step 3 hand calculation.

What I asked the AI:

"How do I get total spending per category and the percentage of the total from a pandas DataFrame with category and amount columns?"

What it returned:

```python
 spend = df[df.amount < 0].groupby("category")["amount"].sum().abs()
 pct = (spend / spend.sum() * 100).round(1).sort_values(ascending=False)
```

What I rejected and why: it's correct and short, and I recognise `groupby` from Module 7. But I want to try and keep the custom tool to be my own logic, not a library call standing in for it. I kept it in mind as a way to double-check my answer later.

What I did instead: I followed my pseudocode with a `for` loop over a list of dictionaries, using the Module 6 grouping pattern `totals[category] = totals.get(category, 0) + spend`. The tool takes a plain list, so I can test it with tiny lists I write by hand.

4. Gemini Chat Working

What I was trying to do: connect a real Gemini model with a finance persona that answers from the student's own summary, without the key ever being in the notebook.

What I asked the AI:

"Show me how to call Gemini from Python with a system instruction."

What it returned:

```python
import google.generativeai as genai
genai.configure(api_key="YOUR_API_KEY_HERE")
model = genai.GenerativeModel("gemini-pro", system_instruction="You are a helpful finance assistant.")
print(model.generate_content("...").text)
```

What I rejected and why:

`google.generativeai` is Google's old library. Current quickstart use `from google import genai` with `genai.Client(...)`.

`api_key="YOUR_API_KEY_HERE"` invites pasting the key into the notebook, where it would end up on GitHub. As in Module 8, I read it from Colab Secrets with `userdata.get("GOOGLE_API_KEY")`.

`"gemini-pro"` isn't a current free model. I used `gemini-flash-latest`, as the checklist says, with `gemini-3.5-flash-lite` as the fallback.

"You are a helpful finance assistant" is too vague to be a persona. I wrote my own: Penny, a coach for Australian uni students, with rules for using only the figures given, staying short, handling off-topic and unclear questions, and avoiding investment and loan advice.

What I kept: passing the persona as `system_instruction` in a config (`types.GenerateContentConfig`). A system instruction specifically is needed, and this is how the current library sets one.

Design decision: I split the work into `build_prompt` (summary + conversation so far + question), `ask_gemini` (calls the model), and `check_question` / `answer_question` (catch blank, too-long or too-early questions first). Because the prompt is built separately, I can print exactly what Gemini receives. For September and October, the same question carries completely different numbers. My tests can also check `build_prompt` without calling the API.


