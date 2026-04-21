Challenges: pwn college quarterly quiz level 7/8

There are three tiers of optimisations for v8:
1) Sparkplug (idk)
2) Maglev (idk)
3) TurboFan (~100k loops)

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

#### Simplified Lowering

Case study: pwncollege v8 exploitation level 8

```diff
diff --git a/src/compiler/simplified-lowering.cc b/src/compiler/simplified-lowering.cc
index 02a53ebcc21..006351a3f08 100644
--- a/src/compiler/simplified-lowering.cc
+++ b/src/compiler/simplified-lowering.cc
@@ -1888,11 +1888,11 @@ class RepresentationSelector {
         if (lower<T>()) {
           if (index_type.IsNone() || length_type.IsNone() ||
               (index_type.Min() >= 0.0 &&
-               index_type.Max() < length_type.Min())) {
+               index_type.Min() < length_type.Min())) {
             // The bounds check is redundant if we already know that
             // the index is within the bounds of [0.0, length[.
             // TODO(neis): Move this into TypedOptimization?
-            if (v8_flags.turbo_typer_hardening) {
+            if (false /*v8_flags.turbo_typer_hardening*/) {
               new_flags |= CheckBoundsFlag::kAbortOnOutOfBounds;
             } else {
               DeferReplacement(node, NodeProperties::GetValueInput(node, 0));
```

If the minimum of the range of the input is less than the minimum of the range of the length of the array, it gets optimised and there isn't any bounds checks.

This gets lowered without bounds checks:

```js
function array_oob(idx) {
  var arr = [1.1, 2.2];
  var y = idx & 0xff;
  return arr[y];
}
// length_type: range(2, 2)
// inddx_type: range(0, 255)
```

This still has bounds checks:

```js
var arr = [1.1, 2.2];
function array_oob(idx) {
    var y = idx & 0xff;
    if(arr.length >= 2){
        return arr[y];
    }
}
```

This is because the `if(arr.length >= 2){` line doesn't get fed into TurboFan. Instead, declaring a new array of constant length on each function call makes it much simpler for TurboFan to optimise.