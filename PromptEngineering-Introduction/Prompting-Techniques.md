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

One recent idea that came out more recently is the idea of zero-shot CoT (Kojima et al. 2022) that essentially involves adding "Let's think step by step" to the original prompt. Let's try a simple problem and see how the model performs:

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



