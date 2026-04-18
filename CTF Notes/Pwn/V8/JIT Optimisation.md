Challenges: pwn college quarterly quiz level 7

There are three tiers of optimisations for v8:
1) Sparkplug (idk)
2) Maglev (idk)
3) Turbofan (~100k loops)

Looping too many times can result in it not getting optimised by Turbofan at all. I learnt this the hard way.

We can use 

```
%PrepareFunctionForOptimization(fn);
%OptimizeFunctionOnNextCall(fn);
```

To force optimisation, and use `%GetOptimizationStatus(fn)` to see how its optimised. Taken from [here](https://gist.github.com/naugtur/4b03a9f9f72346a9f79d7969728a849f) (Likely outdated, should get a newer source)

```
0 0 0 0 0 1 0 0 0 0 0 1
┬ ┬ ┬ ┬ ┬ ┬ ┬ ┬ ┬ ┬ ┬ ┬
│ │ │ │ │ │ │ │ │ │ │ └─╸ is function
│ │ │ │ │ │ │ │ │ │ └───╸ is never optimized
│ │ │ │ │ │ │ │ │ └─────╸ is always optimized
│ │ │ │ │ │ │ │ └───────╸ is maybe deoptimized
│ │ │ │ │ │ │ └─────────╸ is optimized
│ │ │ │ │ │ └───────────╸ is optimized by TurboFan
│ │ │ │ │ └─────────────╸ is interpreted
│ │ │ │ └───────────────╸ is marked for optimization
│ │ │ └─────────────────╸ is marked for concurrent optimization
│ │ └───────────────────╸ is optimizing concurrently
│ └─────────────────────╸ is executing
└───────────────────────╸ topmost frame is turbo fanned
```