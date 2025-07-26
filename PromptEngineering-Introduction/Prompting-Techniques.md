# Prompt Engineering Techniques

## Zero-Shot Prompting

Zero-shot prompting means that the prompt used to interact with the model won't contain examples or demonstrations. The zero-shot prompt directly instructs the model to perform a task without any additional examples to steer it.

## Few-Shot Prompting

Few-shot prompting involves providing examples to the model in the prompt to steer the model into better performance.

### Example

```
The odd numbers in this group add up to an even number: 4, 8, 9, 15, 12, 2, 1.
A: The answer is False.

The odd numbers in this group add up to an even number: 17, 10, 19, 4, 8, 12, 24.
A: The answer is True.

The odd numbers in this group add up to an even number: 16, 11, 14, 4, 8, 13, 24.
A: The answer is True.

The odd numbers in this group add up to an even number: 17, 9, 10, 12, 13, 4, 2.
A: The answer is False.

The odd numbers in this group add up to an even number: 15, 32, 5, 13, 82, 7, 1.
A: 

The answer is True.

which is wrong :(
```

### Limitations

Even with few-shot prompting, the model can still struggle to predict the correct answer in complex tasks.

## Chain-of-Thought (CoT) Prompting

Chain-of-Thought prompting enables complex reasoning capabilities through intermediate reasoning steps. You can combine it with few-shot prompting to get better results on more complex tasks that require reasoning before responding.

### Example with CoT

```
The odd numbers in this group add up to an even number: 4, 8, 9, 15, 12, 2, 1.
A: Adding all the odd numbers (9, 15, 1) gives 25. The answer is False.

The odd numbers in this group add up to an even number: 17, 10, 19, 4, 8, 12, 24.
A: Adding all the odd numbers (17, 19) gives 36. The answer is True.

The odd numbers in this group add up to an even number: 16, 11, 14, 4, 8, 13, 24.
A: Adding all the odd numbers (11, 13) gives 24. The answer is True.

The odd numbers in this group add up to an even number: 17, 9, 10, 12, 13, 4, 2.
A: Adding all the odd numbers (17, 9, 13) gives 39. The answer is False.

The odd numbers in this group add up to an even number: 15, 32, 5, 13, 82, 7, 1.
A:

Output:

Adding all the odd numbers (15, 5, 13, 7, 1) gives 41. The answer is False.

now we got correct answer
```

## Zero-Shot Chain-of-Thought

Adding "Let's think step by step" to the end of original prompt. Let's try a simple problem and see how the model performs:

### Example: Apple Problem

**Prompt:**
```
I went to the market and bought 10 apples. I gave 2 apples to the neighbor and 2 to the repairman. I then went and bought 5 more apples and ate 1. How many apples did I remain with?
```

**Output:**
```
11 apples
```

The answer is incorrect! Now let's try with the special prompt.

**Prompt:**
```
I went to the market and bought 10 apples. I gave 2 apples to the neighbor and 2 to the repairman. I then went and bought 5 more apples and ate 1. How many apples did I remain with?
Let's think step by step.
```

**Output:**
```
First, you started with 10 apples.
You gave away 2 apples to the neighbor and 2 to the repairman, so you had 6 apples left.
Then you bought 5 more apples, so now you had 11 apples.
Finally, you ate 1 apple, so you would remain with 10 apples.
```

It's impressive that this simple prompt is effective at this task. This is particularly useful where you don't have too many examples to use in the prompt.

## Automatic Chain-of-Thought (Auto-CoT)

When applying chain-of-thought prompting with demonstrations, the process involves hand-crafting effective and diverse examples. This manual effort could lead to suboptimal solutions. Zhang et al. (2022) propose an approach to eliminate manual efforts by leveraging LLMs with "Let's think step by step" prompt to generate reasoning chains for demonstrations one by one. This automatic process can still end up with mistakes in generated chains. To mitigate the effects of the mistakes, the diversity of demonstrations matter. This work proposes Auto-CoT, which samples questions with diversity and generates reasoning chains to construct the demonstrations.

### Auto-CoT Process

Auto-CoT consists of two main stages:

1. **Question Clustering**: Partition questions of a given dataset into a few clusters
2. **Demonstration Sampling**: Select a representative question from each cluster and generate its reasoning chain using Zero-Shot-CoT with simple heuristics

### Visual Representation of Auto-CoT

The Auto-CoT process can be visualized as follows:

#### 1. **Initial Questions (Dataset)**
- Start with a collection of input questions (e.g., "While shopping for music online, Zoe bought 3..." and "A chef needs to cook 9 potatoes...")
- These represent the dataset from which demonstrations will be derived

#### 2. **Clustering Stage**
- Input questions undergo clustering to group similar questions together
- Questions are partitioned into clusters (labeled 1, 2, ..., k)
- Each cluster contains multiple similar data points

#### 3. **Demo Construction**
- **Sampling**: Select representative questions from each cluster using selection criteria
- **LLM Processing**: Use the LLM with "Let's think step by step" to generate detailed reasoning chains
- **Demo Generation**: Create step-by-step question-answer pairs

**Example Demos Generated:**
- **Demo 1**: "Zoe bought 3 country albums... 3×3=9 songs... Zoe bought 5 pop albums... 5×3=15 songs... 9+15=24 songs in total. The answer is 24."
- **Demo 2**: "The chef has already cooked 7 potatoes. That means it has taken him 7×3 minutes... it will take him 3 more minutes to cook each of the remaining 2 potatoes..."

#### 4. **Test Question Application**
- Introduce a new test question: "A pet store had 64 puppies. In one day they sold 28 of them and put the rest into cages with 4 in each cage. How many cages did they use?"
- Format the question to prompt step-by-step reasoning: "A: Let's think step by step."

#### 5. **In-Context Reasoning**
- Provide the auto-generated demos as context to the LLM along with the test question
- The LLM uses the provided demos to perform in-context reasoning
- Generate a detailed, step-by-step answer for the test question

**Example Answer:**
"The pet store had 64 puppies. They sold 28 of them. That means they have 36 puppies left. They put the rest into cages with 4 in each cage. That means they have 9 cages. The answer is 9."

This automated approach eliminates the need for manual hand-crafting of effective demonstrations while maintaining the benefits of Chain-of-Thought prompting.

## Self-Consistency

Self-Consistency uses few-shot CoT with diverse examples, and uses generations to select the most consistent answer. This helps to boost the performance of CoT prompting on tasks involving arithmetic and commonsense reasoning.

### Example: Age Problem

**Prompt:**
```
When I was 6 my sister was half my age. Now I'm 70 how old is my sister?
```

**Output:**
```
35
```

The output is wrong! How may we improve this with self-consistency? Let's try it out. We will use the few-shot exemplars from Wang et al. 2022 (Table 17):

**Prompt:**
```
Q: There are 15 trees in the grove. Grove workers will plant trees in the grove today. After they are done, there will be 21 trees. How many trees did the grove workers plant today?
A: We start with 15 trees. Later we have 21 trees. The difference must be the number of trees they planted. So, they must have planted 21 - 15 = 6 trees. The answer is 6.

Q: If there are 3 cars in the parking lot and 2 more cars arrive, how many cars are in the parking lot?
A: There are 3 cars in the parking lot already. 2 more arrive. Now there are 3 + 2 = 5 cars. The answer is 5.

Q: Leah had 32 chocolates and her sister had 42. If they ate 35, how many pieces do they have left in total?
A: Leah had 32 chocolates and Leah's sister had 42. That means there were originally 32 + 42 = 74 chocolates. 35 have been eaten. So in total they still have 74 - 35 = 39 chocolates. The answer is 39.

Q: Jason had 20 lollipops. He gave Denny some lollipops. Now Jason has 12 lollipops. How many lollipops did Jason give to Denny?
A: Jason had 20 lollipops. Since he only has 12 now, he must have given the rest to Denny. The number of lollipops he has given to Denny must have been 20 - 12 = 8 lollipops. The answer is 8.

Q: Shawn has five toys. For Christmas, he got two toys each from his mom and dad. How many toys does he have now?
A: He has 5 toys. He got 2 from mom, so after that he has 5 + 2 = 7 toys. Then he got 2 more from dad, so in total he has 7 + 2 = 9 toys. The answer is 9.

Q: There were nine computers in the server room. Five more computers were installed each day, from monday to thursday. How many computers are now in the server room?
A: There are 4 days from monday to thursday. 5 computers were added each day. That means in total 4 * 5 = 20 computers were added. There were 9 computers in the beginning, so now there are 9 + 20 = 29 computers. The answer is 29.

Q: Michael had 58 golf balls. On tuesday, he lost 23 golf balls. On wednesday, he lost 2 more. How many golf balls did he have at the end of wednesday?
A: Michael initially had 58 balls. He lost 23 on Tuesday, so after that he has 58 - 23 = 35 balls. On Wednesday he lost 2 more so now he has 35 - 2 = 33 balls. The answer is 33.

Q: Olivia has $23. She bought five bagels for $3 each. How much money does she have left?
A: She bought 5 bagels for $3 each. This means she spent $15. She has $8 left.

Q: When I was 6 my sister was half my age. Now I'm 70 how old is my sister?
A:
```

**Output 1:**
```
When I was 6 my sister was half my age, so she was 3. Now I am 70, so she is 70 - 3 = 67. The answer is 67.
```

## Prompt Chaining

Prompt chaining is a technique that breaks complex tasks into smaller, manageable steps and runs multiple prompts sequentially, where each prompt uses the output of the previous step as input.

### How It Works

🧠 **Concept**: It's like building a chain of prompts, where each link moves the AI closer to the final result.

Think of it like a multi-step recipe:
```
Step 1 → Step 2 → Step 3... → Final Output
```

### Example: Writing a Poem About Einstein

🎯 **Goal**: Write a poem about Albert Einstein's achievements.

Instead of doing it all in one prompt, we break it into smaller prompts:

#### 🔗 Step 1: Gather Facts

**Prompt 1:**
```
"List 3 major achievements of Albert Einstein."
```

**Output 1:**
```
Theory of relativity
Nobel Prize for the photoelectric effect
E=mc²
```

#### 🔗 Step 2: Create Poetic Outline

**Prompt 2:**
```
"Write a short poetic outline based on these facts: Theory of relativity, Nobel Prize, E=mc²"
```

**Output 2:**
```
A verse about time and space
A verse about the Nobel Prize
A verse about mass-energy equivalence
```

#### 🔗 Step 3: Write the Final Poem

**Prompt 3:**
```
"Write a 3-stanza poem using this outline: [Insert Output 2]"
```

**Output 3:**
```
(Generates the final poem)
```

### 💡 Benefits of Prompt Chaining

- **Complex Task Management**: Helps with complex tasks like coding, summarizing, or writing essays
- **Focused Control**: Keeps each step focused and controllable
- **Error Reduction**: Reduces errors and improves accuracy
- **Easy Debugging**: Easier to debug if something goes wrong

