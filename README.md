INDENG 164 Final project: Better Call Saul 

By Ted Dodampegamage

















Context of the Company
Public defenders are a vital part of the justice system and perform some of the most selfless work in the country. Yet, they are incredibly underpaid and under-resourced. On average, a public defender manages 50 to 70 cases all at once, and sometimes upward of 100. With little to no time to prepare before a hearing, conduct client interviews, or perform extensive case research, the ability to rapidly access accurate legal information can be the difference between winning and losing a case.
This is where my AI company, Better Call Saul, comes in. The program would be a research assistant and case builder, giving public defenders the help they need for their cases. At its core, Saul is an intelligent routing layer that accepts natural-language legal queries from attorneys. For example:
 "Find Ninth Circuit precedent on warrantless cell phone searches*," 
"Draft* a motion to suppress based on lack of probable cause"*
Then it delivers each prompt to the most suitable Large Language Model (LLM). Because public defender offices operate on strictly limited public funds, Saul ensures that users spend as little money as possible while still receiving high-fidelity legal reasoning.

The Central Problem
The central problem Better Call Saul addresses is LLM routing: given a pool of candidate models ranging from expensive flagship models (such as Claude 3.5 Sonnet or GPT-4o) to cheap or open-source alternatives (such as DeepSeek-Qwen-7B, deployable locally at zero API cost), how should an incoming legal query be assigned to maximize response quality while minimizing operational costs?


Research Questions
We formalize this as a constrained mathematical optimization problem and explore the following two research questions:
1. How can we leverage low-cost or zero-cost models as our primary responders to save money, and at what point does the probability of them failing necessitate a more expensive fallback model? 
2. How does increasing the penalty for an incorrect legal answer (the alpha parameter) shift our optimization program, and how does it change our outcome? Furthermore, how can we find the sweet spot for us to get the highest accuracy for a reasonable price? 

What model should we use?
For the model, we can use a Two-Stage Stochastic Optimization framework, but why?
Before any specific prompt's outcome is known, our policy must select exactly one Primary Model and exactly one Fallback Model to handle incoming queries. We can know the cost of the Primary Model immediately.
The uncertainty, or omega, is that we don’t know if the primary model we have chosen will generate a successful response for the lawyers' questions. We only care about the rate of failure.
We can then use stage two to recover if the primary model fails, as it will trigger a recourse action and the prompt will go to a higher-cost model for a more in-depth or correct answer.  
By utilizing this two-stage stochastic model, Better Call Saul actively fights against uncertainty. mathematically balances the guaranteed costs of initial routing with the expected, probability-driven costs of error correction, giving public defenders exactly what they need to win their cases within budget.

Parameters for my model and my Model Written out:


Code implementing the Two-Stage Stochastic model. 
Link for code - GitHub


Visualizations and Key Takeaways
The generated Cost-Quality Frontier plot visually demonstrates the powerful trade-offs discovered by our two-stage stochastic model. At the lowest budget thresholds (alpha = 0.01 to 0.05), the model relies on a highly constrained pairing (MiniCPM4.1-8B as primary, backed by deepseek-v3.1-terminus), which yields a baseline expected solve rate of 75.0% for a negligible cost of $0.0006 per prompt.
As we increase the penalty for failure (alpha), the frontier curves upward steeply, showing us that intelligent routing will be very cost-effective. The red dashed line on our graph represents the best model we can send it to, which is just routing everything to GPT-5. (FIG 1) This would yield an 88.75% accuracy, yet at the high cost of  $0.0425 per prompt. However, at alpha = 0.5, our optimal cascade (deepseek-v3.1-terminus as primary, escalating to gpt-5) pushes the expected solve rate up to 92.08% while keeping the expected cost at just $0.0216. Our optimized policy achieves higher accuracy than the single best model while cutting the operational cost in half.
The domain sweep visualization further proves that the optimal routing strategy is highly sensitive to the type of legal query. While deepseek-v3.1-terminus is a robust general primary, queries heavily focused on specific domains (like GPQA) require a shift in the optimal policy, favoring kimi-k2-0905 as the primary to maintain high accuracy (94.17%) at lower expected costs ($0.015). (FIG 2)




Real-world constraints in my model
Its hard for anyone to maintain dozens of different API integrations due to the engineering overhead required for monitoring, security, and maintenance. My model addresses this by having a constraint that ensures that only one model is being used. This limits the "model pool size" to having one primary or one fallback, to reduce pressure on the system. 
To introduce robustness to the system, I utilized Sample Average Approximation (SAA) to test how the routing policy holds up under "Reweighted" distributions. By reweighting the importance of specific domains like LCB or GPQA in the code, this way I can show that the optimal policy adapts to prioritize the most relevant expertise for the specific office's needs.
FIGURES
Budget Tier (α)
Primary Model
Fallback Model
Exp. Accuracy
Exp. Cost
Low (0.01)
MiniCPM4.1-8B
deepseek-v3.1
75.0%
$0.0006
Medium (0.1)
deepseek-v3.1
qwen3-235b
88.3%
$0.0085
High (0.5+)
deepseek-v3.1
gpt-5
92.1%
$0.0216

Only some of the variations were used, but these were the most important for my report. 
FIG 1
FIG 2

FIG 3
Answering the Research Questions
Q1: The cost-performance frontier demonstrates that highly efficient open-weight models, such as MiniCPM4.1-8B, are highly effective as Stage 1 primary routers when operational budgets are strictly constrained. However, the model reveals a sharp point of diminishing returns. Relying on this highly constrained cascade results in a baseline failure rate of 25.0%. The optimization proved that to maintain a viable legal research assistant for public defenders, we cannot rely on the cheapest models alone; they must be paired with mid-tier or flagship resource models to catch inevitable reasoning failures.
Q2: By sweeping the alpha penalty parameter, I mapped the optimal routing policy across various budget levels (FIG 3). For a moderately constrained public defender office, we can have alpha = 0.1; the optimal policy assigns deepseek-v3.1-terminus as the primary router and qwen3-235b-a22b-thinking-2507 as the fallback, yielding an expected accuracy of 88.33% at a cost of just $0.008 per query. As the budget increases to accommodate high-stakes queries alpha, the policy shifts to using GPT-5 as the ultimate safety net. This combination maximizes the system accuracy at 92.08%, proving that a targeted fallback strategy is the better way to go about using models to answer questions instead of just having one.
If granted more time and resources:
I could incorporate a latency part into the model, as time is a very important resource to Public Defences, and having these cascading models takes a longer time delay than just putting it into one model. To combat this, I could have had a parameter that penalizes longer routes that take longer to generate a response. However, I couldn’t get the code to work for this constraint, and I wasn’t able to emulate the time parameter well in test runs. 
I could have had the model run through a lot more models before settling on an output, but that would have taken a lot more time and more supplies from the user. I think if I had more time, I could have implemented more tiers for the cascade to make the model even more accurate and maybe go over 99%. 


Personal Takeaways
Through this project, I thought a lot about the business side of the model and the “usefulness to society” side of the model, and it helped me make a decision based on which side I would have more of a benefit. For example, which models to use as the primary models and how to optimize them in a way that is profitable and good for the user. I chose to make my model mostly about the user in general to focus more on the model being as optimized for use as possible. 
I also learned to work with LLMs that go wrong or give incorrect answers by using a two-stage stochastic model with cascading to achieve higher accuracy than a flagship model like GPT-5 while cutting costs in half. This showed that using specific models for certain tasks yields cheaper and more accurate work. This could be applied to many different things, but most importantly, using and creating very specific models for certain tasks. Take healthcare, for example: in one of my other classes, I built a model to predict the outcomes of CT scans using image data. Using models like this and a routing system, as I built in this class, could be very effective for people in healthcare to get answers fast based on the data they receive from their patients. 

I used Gemini for report structuring and code debugging.
