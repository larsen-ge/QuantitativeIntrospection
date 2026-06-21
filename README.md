# Quantitative Measure of Direct Introspection

## Intro

Much AI research on "introspection", though brilliant, is convoluted. This leaves daylight between what it actually demonstrates and what it hoped to investigate. For instance, some purported "introspection" research actually asks "can the model predict itself" — this could represent introspection, but it has the confound that it could just be accurately rerunning itself and getting similar results rather than looking inside at the process. A standard approach to circumventing this confound is to compare if model A can predict model A better than A can predict B (or B, A). But this still doesn't necessarily show introspection — one might imagine that model A could just generally be more likely to perform like model A than model B. The confound persists.

I'm hoping to sharpen this picture, by quantitatively querying a model about a specific internal. So I want to investigate the model's ability to predict its logprobs of its next token.

## Implementation

Some model APIs allow output of a vector of logprobs, but not all. GPT (with thinking off), Gemini (prior to 3), and open-source can all be made to work. I'm starting with GPT 5.5 as the most frontier of these options, but will explore others to see how results generalize.

I created a pipeline that'll take a list of prompts, wrap them in some insistence[^1] that the response be a single word, get the model's logprobs for the first token of a response and its output at the default temperature, and then ask a fresh instance of the model to estimate its likelihood of outputting that token in response to that prompt. (This is all done with thinking off.) Finally, the estimate is compared to the true probability in a scatterplot, and *r*² is calculated.

Ideally, the list of questions result in a broad distribution of probabilities — some nearly deterministic, others 50/50, and in between. It's conceivable different sorts of questions are more easily introspected than others, but I don't expect this. For starters, I had Claude Code generate two lists of 100 questions:

- **"Advice"**: Since the primary persona of chat LLMs is the "assistant" persona, ask simple user queries: "Should I do [x] or [y]"?
- **"Lightning Round"**: Make the LLM pick a side in a popular debate: "Chocolate: dark or milk?"

### Code

<https://github.com/larsen-ge/QuantitativeIntrospection>

## Results

For the Advice questions:

![Scatterplot of estimated vs. true probability — Advice questions][image1]

*r*² = 0.008.

For the Lightning Round questions:

![Scatterplot of estimated vs. true probability — Lightning Round questions][image2]

*r*² = 0.00009.

In either case, the results are completely uncorrelated. The "Lightning Round" results are consistent with a uniform distribution of estimates, independent of true probability; the "Advice" results are nearly so, except that GPT *never* guessed higher 0.8 (and only exceeded 0.67 when true probability was ≥ 0.95), perhaps suggesting a prompt-dependent bias towards hedging.

## Discussion

This is essentially a null result: At this level, GPT is not capable of this form of introspection. This demands followup: Is this true of other models? Does turning on thinking for the estimation stage improve performance? Can models be trained to succeed at this task?

[Rewrite more carefully and think through natural followups…] This does shed light on the broader discussion: When a paper comes out saying in some complicated experiment, a model demonstrates it's capable of some form of introspective, it's instructive to know that it is likely not doing anything like this. (And presumably this is why the setups are more elaborate.)

I also publish this negative result to mitigate publication bias against negative results. (Indeed, I suspect work very similar to this has been done by many and gone unpublished.)

## ToDos

- See if enabling thinking improves GPT's capability.
- Run on other models.
- Vary the prompt requesting the estimate — maybe the failure to introspect is a prompting skill issue.
- Investigate if models can be trained to do this.

## Related Work

### 2025/10 — Lindsey

Emergent Introspective Awareness
<https://transformer-circuits.pub/2025/introspection/index.html>

### 2025/03 — Song/Hu/Mahowald

Language Models Fail to Introspect About Their Knowledge of Language
<https://arxiv.org/abs/2503.07513>

### 2024/10 — Binder/Chua *et al.*

Looking Inward: Language Models Can Learn About Themselves by Introspection
<https://arxiv.org/abs/2410.13787>

### 2022/05 — Lin/Hilton/Evans

Teaching Models to Express Their Uncertainty in Words
<https://arxiv.org/abs/2205.14334>

[^1]: Specifically, it prepends the line `"Please answer the following in a single word.\n"` and appends the line `"\nRemember, only one-word answers are permitted."`

[image1]: Advice.png
[image2]: Lightning.png
