---
{"dg-publish":true,"permalink":"/pl/o-caml-revisit/","noteIcon":"","created":"2024-07-21T04:17:21.814+08:00","updated":"2024-07-21T16:14:05.196+08:00"}
---

#OCaml #PL 
> Based on Version 5.2, take some notes that forgot before~

# Chapter 1 The core language
1. If there is not enough information to disambiguate between different fields or constructors, OCaml picks the last defined type amongst all locally valid choices. [link](https://ocaml.org/manual/5.2/coreexamples.html#ss:record-and-variant-disambiguation)
2. Imperative features
```ocaml
(* Arrays and for loop *)
let add_vect v1 v2 =
  let len = min (Array.length v1) (Array.length v2) in
  let res = Array.make len 0.0 in
  for i = 0 to len - 1 do
    res.(i) <- v1.(i) +. v2.(i)
  done;
  res
;;
(* val add_vect : float array -> float array -> float array = <fun> *)
add_vect [| 1.0; 2.0 |] [| 3.0; 4.0 |]
(* - : float array = [|4.; 6.|] *)

(* mutable record type *)
type mutable_point =
  { mutable x : float
  ; mutable y : float
  }

let translate p dx dy =
  p.x <- p.x +. dx;
  p.y <- p.y +. dy
;;
(* val translate : mutable_point -> float -> float -> unit = <fun> *)
```
- OCaml has no built-in notion of variable – identifiers whose current value can be changed by assignment. (The let binding is not an assignment, it introduces a new identifier with a new scope.)
	- The standard library provides references, which are mutable indirection cells, with operators `!` to fetch the current contents of the reference and `:=` to assign the contents. Variables can then be emulated by let-binding a reference.
```ocaml
let insertion_sort a =
  for i = 1 to Array.length a - 1 do
    let val_i = a.(i) in
    let j = ref i in
    while !j > 0 && val_i < a.(!j - 1) do
      a.(!j) <- a.(!j - 1);
      j := !j - 1
    done;
    a.(!j) <- val_i
  done
;;
(* val insertion_sort : 'a array -> unit = <fun> *)
```
- store a polymorphic function in a data structure, keeping its polymorphism
```ocaml
# type idref = { mutable id: 'a. 'a -> 'a };;
type idref = { mutable id : 'a. 'a -> 'a; }
# let r = {id = fun x -> x};;
val r : idref = {id = <fun>}
# let g s = (s.id 1, s.id true);;
val g : idref -> int * bool = <fun>
# r.id <- (fun x -> print_string "called id\n"; x);;
- : unit = ()
# g r;;
called id
called id
- : int * bool = (1, true)
type bothref = { mutable both : 'a 'b. 'a -> 'b -> 'a * 'b }
let b = { both = (fun x y -> x, y) }
```
3. Exception
```ocaml
exception Empty_list

let head l =
  match l with
  | [] -> raise Empty_list
  | hd :: _ -> hd
;;

let rec first_named_value values names =
  try List.assoc (head values) names with
  | Empty_list -> "no named value"
  | Not_found -> first_named_value (List.tl values) names
;;
(* val first_named_value : 'a list -> ('a * string) list -> string = <fun> *)
# first_named_value [0; 10] [1, "one"; 10, "ten"];;
- : string = "ten"

(* finalization perfrom *)
let temporarily_set_reference ref newval funct =
  let oldval = !ref in
  try
    ref := newval;
    let res = funct () in
    ref := oldval;
    res
  with
  | x ->
    ref := oldval;
    raise x
;;
(* Use as local *)
let fixpoint f x =
    let exception Done in
    let x = ref x in
    try while true do
        let y = f !x in
        if !x = y then raise Done else x := y
      done
    with Done -> !x;;
- val fixpoint : ('a -> 'a) -> 'a -> 'a = <fun>
```
4. Lazy expressions, use `lazy (expr)` to delay the evaluation of some expression `expr`.
```ocaml
let lazy_two =
  lazy
    (print_endline "lazy_two evaluation";
     1 + 1)
;;

# Lazy.force lazy_two;;
lazy_two evaluation
- : int = 2
(* Lazy.force memoizes the result of the forced expression *)
# lazy_two;;
- : int lazy_t = lazy 2


(* Another way to force lazy *)
# let lazy_l = lazy ([1; 2] @ [3; 4]);;
val lazy_l : int list lazy_t = <lazy>
# let lazy l = lazy_l;;
val l : int list = [1; 2; 3; 4]


# let maybe_eval lazy_guard lazy_expr =
    match lazy_guard, lazy_expr with
    | lazy false, _ -> "matches if (Lazy.force lazy_guard = false); lazy_expr not forced"
    | lazy true, lazy _ -> "matches if (Lazy.force lazy_guard = true); lazy_expr forced";;
val maybe_eval : bool lazy_t -> 'a lazy_t -> string = <fun>
```
5. OCaml code can also be compiled separately and executed non-interactively using the batch compilers ocamlc and ocamlopt. The source code must be put in a file with extension `.ml`. It consists of a sequence of phrases, which will be evaluated at runtime in their order of appearance in the source file. Unlike in interactive mode, types and values are not printed automatically; the program must call printing functions explicitly to produce some output. The `;;` used in the interactive examples is not required in source files created for use with OCaml compilers, but can be helpful to mark the end of a top-level expression unambiguously even when there are syntax errors.

# Chapter 2 The module system
1. A primary motivation for *modules* is to package together related definitions (such as the definitions of a data type and associated operations over that type) and enforce a consistent naming scheme for these definitions. This avoids running out of names or accidentally confusing names. 
	- Such a package is called a _structure_ and is introduced by the struct…end construct, which contains an arbitrary sequence of definitions. The structure is usually given a name with the module binding.
	- It is also possible to copy the components of a module inside another module by using an `include` statement. This can be particularly useful to extend existing modules. As an illustration, we could add functions that return an optional value rather than an exception when the queue is empty.
```ocaml
# module Fifo =
    struct
      type 'a queue = { front: 'a list; rear: 'a list }
      let make front rear =
        match front with
        | [] -> { front = List.rev rear; rear = [] }
        | _  -> { front; rear }
      let empty = { front = []; rear = [] }
      let is_empty = function { front = []; _ } -> true | _ -> false
      let add x q = make q.front (x :: q.rear)
      exception Empty
      let top = function
        | { front = []; _ } -> raise Empty
        | { front = x :: _; _ } -> x
      let pop = function
        | { front = []; _ } -> raise Empty
        | { front = _ :: f; rear = r } -> make f r
    end;;
module Fifo :
  sig
    type 'a queue = { front : 'a list; rear : 'a list; }
    val make : 'a list -> 'a list -> 'a queue
    val empty : 'a queue
    val is_empty : 'a queue -> bool
    val add : 'a -> 'a queue -> 'a queue
    exception Empty
    val top : 'a queue -> 'a
    val pop : 'a queue -> 'a queue
  end
# Fifo.add "hello" Fifo.empty;;

- : string Fifo.queue = {Fifo.front = ["hello"]; rear = []}
Fifo.(add "hello" empty);;
let open Fifo in add "hello" empty;;


# module FifoOpt =
  struct
    include Fifo
    let top_opt q = if is_empty q then None else Some(top q)
    let pop_opt q = if is_empty q then None else Some(pop q)
  end;;
module FifoOpt :
  sig
    type 'a queue = 'a Fifo.queue = { front : 'a list; rear : 'a list; }
    val make : 'a list -> 'a list -> 'a queue
    val empty : 'a queue
    val is_empty : 'a queue -> bool
    val add : 'a -> 'a queue -> 'a queue
    exception Empty
    val top : 'a queue -> 'a
    val pop : 'a queue -> 'a queue
    val top_opt : 'a queue -> 'a option
    val pop_opt : 'a queue -> 'a queue option
  end
```
2. *Signatures* are interfaces for structures. A signature specifies which components of a structure are accessible from the outside, and with which type. It can be used to hide some components of a structure (e.g. local function definitions) or export some components with a restricted type.
	- it is possible to include a signature to copy its components inside the current signature.
```ocaml
# module type FIFO =
    sig
      type 'a queue               (* now an abstract type *)
      val empty : 'a queue
      val add : 'a -> 'a queue -> 'a queue
      val top : 'a queue -> 'a
      val pop : 'a queue -> 'a queue
      exception Empty
    end;;
module type FIFO =
  sig
    type 'a queue
    val empty : 'a queue
    val add : 'a -> 'a queue -> 'a queue
    val top : 'a queue -> 'a
    val pop : 'a queue -> 'a queue
    exception Empty
  end
  
# module AbstractQueue = (Fifo : FIFO);;
module AbstractQueue : FIFO
# AbstractQueue.make [1] [2;3] ;;
Error: Unbound value AbstractQueue.make
# AbstractQueue.add "hello" AbstractQueue.empty;;
- : string AbstractQueue.queue = <abstr>

module Fifo = (struct ... end : FIFO);;

# module type FIFO_WITH_OPT =
    sig
      include FIFO
      val top_opt: 'a queue -> 'a option
      val pop_opt: 'a queue -> 'a queue option
    end;;
module type FIFO_WITH_OPT =
  sig
    type 'a queue
    val empty : 'a queue
    val add : 'a -> 'a queue -> 'a queue
    val top : 'a queue -> 'a
    val pop : 'a queue -> 'a queue
    exception Empty
    val top_opt : 'a queue -> 'a option
    val pop_opt : 'a queue -> 'a queue option
  end
```
3. *Functors* are “functions” from modules to modules. Functors let you create parameterized modules and then provide other modules as parameter(s) to get a specific implementation.


- [Compiler Development: Rust or OCaml?](https://hirrolot.github.io/posts/compiler-development-rust-or-ocaml.html#)
- [The OCaml Manual](https://ocaml.org/manual/5.2/index.html#)
- 