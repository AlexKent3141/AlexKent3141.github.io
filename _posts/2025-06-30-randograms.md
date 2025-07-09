---
layout: post
title: "PRNGs & Randograms"
date: 2025-07-01 12:00:00 +0000
categories: []
tags: [C, PRNG, experiment]
---

I'll kick-off this blog by re-visiting one of my early experiments. I first heard about randograms through this [nullprogram post](https://nullprogram.com/blog/2020/11/17/) where Chris Wellons used one to visualise the (poor) quality of the original 24-bit pseudo-random number generator (PRNG) used in QBasic.

The idea is to repeatedly call the PRNG, extract some bytes from the output on each iteration (in this post I'll look at the two lowest bytes), and plot these against each other. The result is a 256x256 image which gives us a visual clue about the quality of the PRNG.

Let's see how these look for a few common PRNGs.

I'll also introduce a heatmap idea to give us a slightly deeper picture of where the weaknesses of each PRNG lie. I produce the heatmaps at the same time as the randograms, but instead of plotting the bytes against each other I look at the individual bits to see if there are any obvious correlations.

# QBasic
The QBasic PRNG is a simple Linear Congruential Generator (LCG). Translating into C the code looks like this:
```c
uint32_t qbasic(uint32_t* s)
{
    *s = (*s*0xfd43fd + 0xc39ec3) & 0xffffff;
    return *s;
}
```

Here are a few QBasic randograms with different seeds:
![QBasic randograms](/assets/qb_randograms.png)

There's definitely a pattern here! As we vary the seed there are these horizontal line features that always appear in similar positions.

How does the heatmap look? For seed 914585500 we've got:

| Bit index | 8 | 9  | 10 | 11 | 12 | 13  | 14 | 15 | Avg.
| 0 | 4065 | 4066 | 4066 | 4064 | 4060 | 4062 | 4078 | 4066 | 4065
| 1 | 4065 | 4073 | 4064 | 4060 | 4066 | 4068 | 4125 | 3985 | 4063
| 2 | 4067 | 4072 | 4064 | 4060 | 4062 | 4069 | 4159 | 3973 | 4065
| 3 | 4061 | 4066 | 4066 | 4060 | 4064 | 4065 | 4084 | 4033 | 4062
| 4 | 4071 | 4065 | 4070 | 4060 | 4063 | 4066 | 4159 | 3999 | 4069
| 5 | 4067 | 4062 | 4067 | 4064 | 4064 | 4067 | 4170 | 4030 | 4073
| 6 | 4063 | 4064 | 4065 | 4062 | 4063 | 4066 | 4110 | 4012 | 4063
| 7 | 4077 | 4073 | 4065 | 4069 | 4064 | 4067 | 4121 | 3991 | 4065
| Avg. | 4067 | 4067 | 4065 | 4062 | 4063 | 4066 | 4125 | 4011

We've got some anomalies in the last two columns. Bit 14 is on more frequently than average and Bit 15 is on less frequently. Additionally the row averages are remarkably consistent - if the output was truly random there would be more variation here.

I'm not going to look at this very formally, but given that I see similar patterns for all of the seeds I've tried, I think we're seeing clear evidence of dependence between the lower bits. This is what you would expect for an LCG.

# C rand()
Next I wanted to take a look at the venerable C `rand()` function. This is another LCG, so we're going to see a similar picture, right?

![C rand randograms](/assets/rand_randograms.png)

Well that's a lot better than I expected! If I squint at these randograms then maybe I can make out some 2D structures, but it's a lot less obvious than in the previous example.

The heatmap for 3439755361 looks like this:

| Bit index | 8 | 9  | 10 | 11 | 12 | 13  | 14 | 15 | Avg. |
| 0 | 4080 | 4131 | 4016 | 4017 | 4074 | 4078 | 4082 | 4066 | 4068
| 1 | 4119 | 4038 | 4074 | 4068 | 4081 | 4081 | 4121 | 4070 | 4081
| 2 | 4152 | 4076 | 4110 | 4100 | 4107 | 4147 | 4093 | 4066 | 4106
| 3 | 4053 | 4027 | 4038 | 4056 | 4045 | 4132 | 4085 | 4102 | 4067
| 4 | 4103 | 4151 | 4089 | 4096 | 4085 | 4143 | 4149 | 4197 | 4126
| 5 | 4092 | 4083 | 4016 | 3982 | 4160 | 4086 | 4058 | 4058 | 4066
| 6 | 3986 | 4061 | 3970 | 4054 | 4013 | 4045 | 4013 | 4030 | 4021
| 7 | 4067 | 4031 | 3989 | 4016 | 4022 | 4086 | 4143 | 4042 | 4049
| Avg. | 4081 | 4074 | 4037 | 4048 | 4073 | 4099 | 4093 | 4078

Again, this looks reasonable. I've tried a few seeds and can't pick out any obvious correlations between the bits. The really uniform row/column averages aren't visible here.

So why does `rand` seem so much better than the QBasic examples? Well, the C Standard doesn't *actually* say how `rand` should be implemented. It's commonly an LCG, but could be something better.

On my system `rand` is implemented as part of libc, specifically version `GLIBC 2.39-0ubuntu8.4`. I've done some digging and in fact locally my `rand` implements a Linear Feedback Shift Generator (LFSG), which is more sophisticated and has much better randomness properties.

<div markdown="1" class="sidenote-box">
#### Side note: LCG improvements
It wouldn't be surprising if another LCG had better properties than the original QBasic implementation. The key improvements we could make are:
- Use the full 32-bit period: the QBasic implementation is limited to 24-bits for no good reason.
- Introduce mixing: we can reduce linear dependence between bits by combining different parts of the state using bit shifts and XOR operations.
</div>

# Xorshift
The `xorshift` family of PRNGs are some of my favourites: they are simple to implement and behave very nicely. They're actually related to the well-behaving LFSG we saw in the previous section.

I've tested the 32-bit XOR-based version (ref: Algorithm "xor" from p. 4 of Marsaglia, "Xorshift RNGs"):

```c
uint32_t xorshift32(uint32_t* s)
{
  *s ^= *s << 13;
  *s ^= *s >> 17;
  *s ^= *s << 5;
  return *s;
}
```

![xs32 randograms](/assets/xs32_randograms.png)

Heatmap for seed 2191022762:

| Bit index | 8 | 9  | 10 | 11 | 12 | 13  | 14 | 15 | Avg. |
| 0 | 4054 | 3999 | 4011 | 3972 | 4132 | 4077 | 4108 | 4151 | 4063
| 1 | 3978 | 4045 | 3958 | 3946 | 4087 | 3995 | 4048 | 4111 | 4021
| 2 | 4032 | 4050 | 4037 | 4028 | 4038 | 4079 | 3991 | 4074 | 4041
| 3 | 4059 | 4079 | 4047 | 4000 | 4136 | 4090 | 4083 | 4087 | 4072
| 4 | 4076 | 4002 | 4043 | 4007 | 4082 | 4022 | 4026 | 4077 | 4041
| 5 | 4096 | 4025 | 4065 | 4016 | 4151 | 4167 | 4091 | 4141 | 4094
| 6 | 4043 | 4012 | 4003 | 4024 | 4083 | 4114 | 4031 | 4060 | 4046
| 7 | 4028 | 4035 | 4026 | 3944 | 4087 | 4030 | 4011 | 4065 | 4028
| Avg. | 4045 | 4030 | 4023 | 3992 | 4099 | 4071 | 4048 | 4095

A nice, simple algorithm, plus the randogram and heatmap are comparable to the LFSG `rand` implementation. We can further improve this algorithm by switching to `xorshift*` which includes an extra multiplication and reduces linear dependence.

# Closing thoughts
I think randograms are fun and give us some nice intuition into the quality of a PRNG's randomness.

They're obviously not great for giving us a measurable output - even when combined with my ad hoc bitwise comparisons - for that we need test suites like [Big Crush](https://en.wikipedia.org/wiki/TestU01).

When choosing which PRNG to use for a project you need to take into account details of the problem domain. None of the algorithms I've shown above are suitable for cryptographic purposes, but if the quality of the randomness isn't paramount and performance is then I'd recommend one of the xorshift family (e.g. for Monte Carlo simulations).

Despite the C `rand` function making a good showing, I would not recommend using it unless you don't care about quality of randomness at all. The implementation you have access to might be a lot worse than mine.

The source code for this experiment is on my Github at [randogram](https://github.com/AlexKent3141/randogram).
