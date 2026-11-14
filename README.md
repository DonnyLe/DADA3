# DADA3: A bug-fixing team

**Deployed app:** your Render URL here

## Write-up

### Setup

I used Gemini 3.5 Flash lite for everything, in both the team and the baseline, with Daytona as the sandbox. I ran each system twice. The first team run is from notebook 02, and the first baseline run plus the second run of each are from notebook 03. I kept the default settings (MAX_ROUNDS = 3, MAX_PARALLEL = 2) and didn't change any of the prompts.

While I was writing the code I used Ollama with Qwen 3.5, 9b to save Gemini quota. It worked okay for the fixer, but the verifier almost never called accept_fix or transfer_to_fixer, so every decision came back as "none". I switched to Gemini for all the runs below.

### Results

Team, run 1: resolved 9/10, triage 10/10, bugs fixed 6/7, 1 false claim. 124 calls, 2,779,085 tokens, 663 seconds

Baseline, run 1: resolved 9/10, triage 10/10, bugs fixed 6/7, 1 false claim. 46 calls, 476,165 tokens, 152 seconds

Team, run 2: resolved 9/10, triage 10/10, bugs fixed 6/7, 1 false claim. 105 calls, 2,971,152 tokens, 712 seconds

Baseline, run 2: resolved 10/10, triage 10/10, bugs fixed 7/7, 0 false claims. 43 calls, 659,763 tokens, 185 seconds

### Did the team do better?

No, the team did not do better. In the first runs they tied exactly. In the second runs the baseline got all 10 with no false claims and the team stayed at 9 with one. But the baseline's two runs were also one issue apart (9 and 10), so a one-issue difference between the systems is about the same as the difference between two runs of the same system. With only ten issues and two runs each, I can't say either one is clearly better. 

The team got #2 (splitting a negative payment) wrong in both runs, and both times the verifier accepted the fix, so it counts as a false claim. When I read the verifier's tests, they had the wrong expected value for the negative example. The docstring says split_amount(-1000, 3) should be [-334, -333, -333], but run 1's test expected [-334, -334, -333] and run 2's expected [-333, -333, -334]. So the test and the fix agreed with each other but not with the docstring. A separate verifier only helps if it actually copies the spec correctly.

The baseline missed #1 (slugify) in its first run. Its report says it checked "Webcam (HD)" and "Hello, World! T-shirt" from the issue, but it doesn't mention the accented example in the docstring ("Crème Brûlée Mug"), so I think that's what the hidden tests caught. It's basically the same mistake the team made: checking the issue but not the whole docstring.

Both systems got triage 10/10 every time, so neither one changed any of the three non-bugs.

### The handoff

There was only one handback in all my team runs: #2 in the first run. The first fix gave the extra cent to the last share for negative amounts, and the verifier sent it back saying it "splits negative amounts starting from the end / wrong direction (or returns [-333, -333, -334] for split_amount(-1000, 3) instead of [-334, -333, -333] as specified in the docstring and issue)."

The verifier was right. The docstring says the first shares get the extra cent, and its own example is [-334, -333, -333].

However, the second fix still didn't pass. It moved the extra cent to the front, but it used abs(cents % people) to decide how many extra cents to hand out. In Python -1000 % 3 is 2, not 1, so it returns [-334, -334, -333], which adds up to -1001 instead of -1000. The hidden tests failed it. The verifier still accepted it because in the second round its test expected [-334, -334, -333], which matched the fix and not the docstring. So the handoff passed along the right information, but the fixer only half-fixed it, and the verifier went along with it the second time.

The second team run had no handbacks at all. Every fix was accepted on the first try, including a wrong one for #2.

### What did it cost?

Averaging the two runs, the team used about 115 model calls and 2.9 million tokens and took about 11 and a half minutes. The baseline used about 45 calls and 570 thousand tokens and took under 3 minutes. When comparing to baseline to the team, the team uses 2.5x the calls, 5x the tokens and 4x the time. Most of the extra usage/time goes to the verifiers, since each one is a deep agent reading files and running tests in its own sandbox for every bug.

For this repo it was not worth it. The team never resolved more issues than the baseline and made a false claim in both runs, while the baseline tied once and did better once, for about afifth of the tokens and about 4x faster.

In our runs the team's weak point was the verifier's tests. Both false claims on #2 came from a test with the wrong expected value for the docstring's own example, and in run 1 the verifier had the right value in round one and a wrong one in round two. Since everything here ran on gemini-3.5-flash-lite, I think a stronger model for the verifier would write more accurateand thorough tests, and that's where the team could start to pay off. We didn't test that however and a stronger model would probably help the baseline too, so I can't say the gap would favor the team. But within this repository and this configuration, the team was not worth the extra cost.
