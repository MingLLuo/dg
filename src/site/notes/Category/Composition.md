---
{"dg-publish":true,"permalink":"/category/composition/","noteIcon":"","created":"2024-07-27T13:21:47.175+08:00","updated":"2024-07-27T13:25:43.249+08:00"}
---

#Catagory 
There are two extremely important properties that the composition in any category must satisfy.
1. Composition is associative
```haskell
f :: A -> B 
g :: B -> C 
h :: C -> D 
h . (g . f) == (h . g) . f == h . g . f
```
2. For every object $A$ there is an arrow which is a unit of composition. This arrow loops from the object to itself.
```ocaml
let id x = x
```