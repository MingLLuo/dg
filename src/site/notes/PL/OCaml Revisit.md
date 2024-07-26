---
{"dg-publish":true,"permalink":"/pl/o-caml-revisit/","noteIcon":"","created":"2024-07-21T04:17:21.814+08:00","updated":"2024-07-27T03:28:10.138+08:00"}
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
```ocaml
type comparison = Less | Equal | Greater;;

# module type ORDERED_TYPE =
    sig
      type t
      val compare: t -> t -> comparison
    end;;
module type ORDERED_TYPE = sig type t val compare : t -> t -> comparison end
# module Set =
    functor (Elt: ORDERED_TYPE) ->
      struct
		...
        let rec add x s =
          match s with
            [] -> [x]
          | hd::tl ->
             match Elt.compare x hd with
               Equal   -> s         (* x is already in s *)
             | Less    -> x :: s    (* x is smaller than all elements of s *)
             | Greater -> hd :: add x tl
		...
      end;;
# module OrderedString =
    struct
      type t = string
      let compare x y = if x = y then Equal else if x < y then Less else Greater
    end;;
module OrderedString :
  sig type t = string val compare : 'a -> 'a -> comparison end
  
# module StringSet = Set(OrderedString);; 
  
# StringSet.member "bar" (StringSet.add "foo" StringSet.empty);;
- : bool = false

# module AbstractSet2 =
    (Set : functor(Elt: ORDERED_TYPE) -> (SET with type element = Elt.t));;
```
- Hard: [Functors and type abstraction](https://ocaml.org/manual/5.2/moduleexamples.html#s:functors-and-abstraction)
5. In OCaml, compilation units are special cases of structures and signatures, and the relationship between the units can be explained easily in terms of the module system. A compilation unit A comprises two files:
- the implementation file A.ml, which contains a sequence of definitions, analogous to the inside of a struct…end construct;
- the interface file A.mli, which contains a sequence of specifications, analogous to the inside of a sig…end construct.
```ocaml
(* These two files together define a structure named A as if the following definition was entered at top-level *)
module A: sig (* contents of file A.mli *) end
        = struct (* contents of file A.ml *) end;;
```
The files that define the compilation units can be compiled separately using the `ocamlc -c` command (the -c option means *“compile only, do not try to link”*); this produces *compiled interface files* (with extension `.cmi`) and *compiled object code files* (with extension `.cmo`). When all units have been compiled, their .cmo files are linked together using the ocamlc command. For instance, the following commands compile and link a program composed of two compilation units Aux and Main:
```ocaml
$ ocamlc -c Aux.mli                     # produces aux.cmi
$ ocamlc -c Aux.ml                      # produces aux.cmo
$ ocamlc -c Main.mli                    # produces main.cmi
$ ocamlc -c Main.ml                     # produces main.cmo
$ ocamlc -o theprogram Aux.cmo Main.cmo
```
The program behaves exactly as if the following phrases were entered at top-level:
```ocaml
module Aux: sig (* contents of Aux.mli *) end
          = struct (* contents of Aux.ml *) end;;
module Main: sig (* contents of Main.mli *) end
           = struct (* contents of Main.ml *) end;;
```
# Chapter 3 Objects in OCaml
This chapter gives an overview of the object-oriented features of OCaml.
Note that the relationship between object, class and type in OCaml is different than in mainstream object-oriented languages such as Java and C++, so you shouldn’t assume that similar keywords mean the same thing. Object-oriented features are used much less frequently in OCaml than in those languages. OCaml has alternatives that are often more appropriate, such as modules and functors. Indeed, many OCaml programs do not use objects at all.
```ocaml
# class point x_init =
    object
      val mutable x = x_init
      method get_x = x
      method move d = x <- x + d
    end;;
class point :
  int ->
  object val mutable x : int method get_x : int method move : int -> unit end

# new point;;
- : int -> point = <fun>

# let p = new point 7;;
val p : point = <obj>

p#get_x;;

# class adjusted_point x_init =  point ((x_init / 10) * 10);;
class adjusted_point : int -> point
```
1. Immediate objects: The syntax is exactly the same as for class expressions, but the result is a single object rather than a class.
```ocaml
# let p =
    object
      val mutable x = 0
      method get_x = x
      method move d = x <- x + d
    end;;
val p : < get_x : int; move : int -> unit > = <obj>
```
2. Self reference: A method or an initializer can invoke methods on self (that is, the current object). For that, self must be explicitly bound, here to the variable s (s could be any identifier, even though we will often choose the name self.)
```ocaml
# class printable_point x_init =
    object (s)
      val mutable x = x_init
      method get_x = x
      method move d = x <- x + d
      method print = print_int s#get_x
    end;;
class printable_point :
  int ->
  object
    val mutable x : int
    method get_x : int
    method move : int -> unit
    method print : unit
  end
```
3. Initializers: It is also possible to evaluate an expression immediately after the object has been built. Such code is written as an anonymous hidden method called an initializer. Therefore, it can access self and the instance variables.
```ocaml
# class printable_point x_init =
    let origin = (x_init / 10) * 10 in
    object (self)
      val mutable x = origin
      method get_x = x
      method move d = x <- x + d
      method print = print_int self#get_x
      initializer print_string "new point at "; self#print; print_newline ()
    end;;
class printable_point :
  int ->
  object
    val mutable x : int
    method get_x : int
    method move : int -> unit
    method print : unit
  end
# let p = new printable_point 17;;
new point at 10
val p : printable_point = <obj>
```
4. Virtual methods: A class containing virtual methods must be flagged `virtual`, and cannot be instantiated (that is, no object of this class can be created). It still defines type abbreviations (treating virtual methods as other methods.)
```ocaml
# class virtual abstract_point x_init =
    object (self)
      method virtual get_x : int
      method get_offset = self#get_x - x_init
      method virtual move : int -> unit
    end;;
# class point x_init =
    object
      inherit abstract_point x_init
      val mutable x = x_init
      method get_x = x
      method move d = x <- x + d
    end;;
(* Instance variables can also be declared as virtual, with the same effect as with methods. *)

# class virtual abstract_point2 =
    object
      val mutable virtual x : int
      method move d = x <- x + d
    end;;
class virtual abstract_point2 :
  object val mutable virtual x : int method move : int -> unit end
# class point2 x_init =
    object
      inherit abstract_point2
      val mutable x = x_init
      method get_offset = x - x_init
    end;;
class point2 :
  int ->
  object
    val mutable x : int
    method get_offset : int
    method move : int -> unit
  end
```
5. Private methods: Private methods are methods that do not appear in object interfaces. *They can only be invoked from other methods of the same object*.
```ocaml
# class restricted_point x_init =
    object (self)
      val mutable x = x_init
      method get_x = x
      method private move d = x <- x + d
      method bump = self#move 1
    end;;
class restricted_point :
  int ->
  object
    val mutable x : int
    method bump : unit
    method get_x : int
    method private move : int -> unit
  end
(* Private methods can be made public in a subclass. *)
# class point_again x =
    object (self)
      inherit restricted_point x
      method virtual move : _
    end;;
# class point_again x =
    object (self : < move : _; ..> )
      inherit restricted_point x
    end;;
# class point_again x =
    object
      inherit restricted_point x as super
      method move = super#move
    end;;
class point_again :
  int ->
  object
    val mutable x : int
    method bump : unit
    method get_x : int
    method move : int -> unit
  end
```
6. Parameterized classes: At least one of the methods has a polymorphic type (here, the type of the value stored in the reference cell), thus either the class should be parametric, or the method type should be constrained to a monomorphic type.
```ocaml
# class oref x_init =
    object
      val mutable x = x_init
      method get = x
      method set y = x <- y
    end;;
Error: Some type variables are unbound in this type:
         class oref :
           'a ->
           object
             val mutable x : 'a
             method get : 'a
             method set : 'a -> unit
           end
       The method get has type 'a where 'a is unbound
(* A monomorphic instance of the class could be defined by *)
# class oref (x_init:int) =
    object
      val mutable x = x_init
      method get = x
      method set y = x <- y
    end;;
class oref :
  int ->
  object val mutable x : int method get : int method set : int -> unit end
(* Note that since immediate objects do not define a class type, they have no such restriction *)
```
- The type parameter in the declaration may actually be constrained in the body of the class definition. In the class type, the actual value of the type parameter is displayed in the constraint clause.
```ocaml
# class ['a] circle (c : 'a) =
    object
      val mutable center = c
      method center = center
      method set_center c = center <- c
      method move = (center#move : int -> unit)
    end;;
class ['a] circle :
  'a ->
  object
    constraint 'a = < move : int -> unit; .. >
    val mutable center : 'a
    method center : 'a
    method move : int -> unit
    method set_center : 'a -> unit
  end
```
7. Polymorphic methods
```ocaml
(* giving an explicitly polymorphic type in the method definition *)
# class intlist (l : int list) =
    object
      method empty = (l = [])
      method fold : 'a. ('a -> int -> 'a) -> 'a -> 'a =
        fun f accu -> List.fold_left f accu l
    end;;
class intlist :
  int list ->
  object method empty : bool method fold : ('a -> int -> 'a) -> 'a -> 'a end

(* The following idiom separates description and definition. *)

# class type ['a] iterator =
    object method fold : ('b -> 'a -> 'b) -> 'b -> 'b end;;
# class intlist' l =
    object (self : int #iterator)
      method empty = (l = [])
      method fold f accu = List.fold_left f accu l
    end;;
```
8. Functional objects: The override construct {< ... >} returns a copy of “self” (that is, the current object), possibly changing the value of some instance variables.
```ocaml
# class functional_point y =
    object
      val x = y
      method get_x = x
      method move d = {< x = x + d >}
      method move_to x = {< x >}
    end;;
class functional_point :
  int ->
  object ('a)
    val x : int
    method get_x : int
    method move : int -> 'a
    method move_to : int -> 'a
  end
# let p = new functional_point 7;;
val p : functional_point = <obj>
# p#get_x;;
- : int = 7
# (p#move 3)#get_x;;
- : int = 10
# (p#move_to 15)#get_x;;
- : int = 15
# p#get_x;;
- : int = 7
```

# Chapter 5 Polymorphic variants
It is a variant tag does not belong to any type in particular, the type system will just check that it is an admissible value according to its use. You need not define a type before using a variant tag. A variant type will be inferred independently for each of its uses.
1. In programs, polymorphic variants work like usual ones. You just have to prefix their names with a backquote character \`.
```ocaml
# [`On; `Off];;
- : [> `Off | `On ] list = [`On; `Off]
(* [>`Off|`On] list means that to match this list, you should at least be able to match `Off and `On, without argument *)
# `Number 1;;
- : [> `Number of int ] = `Number 1
# let f = function `On -> 1 | `Off -> 0 | `Number n -> n;;
val f : [< `Number of int | `Off | `On ] -> int = <fun>
(* [<`On|`Off|`Number of int] means that f may be applied to `Off, `On (both without argument), or `Number n where n is an integer. *)
# List.map f [`On; `Off];;
- : int list = [1; 0]
```
- The `>` and `<` inside the variant types show that they may still be refined, either by defining more tags or by allowing less.
- The above variant types were polymorphic, allowing further refinement. When writing type annotations, one will most often describe fixed variant types, that is types that cannot be refined. This is also the case for type abbreviations. Such types do not contain < or >, but just an enumeration of the tags and their associated types, just like in a normal datatype definition.
```ocaml
# type 'a vlist = [`Nil | `Cons of 'a * 'a vlist];;
type 'a vlist = [ `Cons of 'a * 'a vlist | `Nil ]
# let rec map f : 'a vlist -> 'b vlist = function
    | `Nil -> `Nil
    | `Cons(a, l) -> `Cons(f a, map f l)
  ;;
val map : ('a -> 'b) -> 'a vlist -> 'b vlist = <fun>
```
2. Type-checking polymorphic variants is a subtle thing, and some expressions may result in more complex type information.
```ocaml
# let f = function `A -> `C | `B -> `D | x -> x;;
val f : ([> `A | `B | `C | `D ] as 'a) -> 'a = <fun>
# f `E;;
- : [> `A | `B | `C | `D | `E ] = `E
(* If we apply f to yet another tag `E, it gets added to the list. *)
# f `A;;
- : [> `A | `B | `C | `D ] = `C
```
- First, since this matching is open (the last case catches any tag), we obtain the type \[> \`A | \`B] rather than \[< \`A | \`B] in a closed matching
- Then, since x is returned as is, input and return types are identical. The notation as 'a denotes such type sharing.
```ocaml
# let f1 = function `A x -> x = 1 | `B -> true | `C -> false
  let f2 = function `A x -> x = "a" | `B -> true ;;
val f1 : [< `A of int | `B | `C ] -> bool = <fun>
val f2 : [< `A of string | `B ] -> bool = <fun>
# let f x = f1 x && f2 x;;
val f : [< `A of string & int | `B ] -> bool = <fun>
(** Here f1 and f2 both accept the variant tags `A and `B, but the argument of `A is int for f1 and string for f2. In f’s type `C, only accepted by f1, disappears, but both argument types appear for `A as int & string. This means that if we pass the variant tag `A to f, its argument should be _both_ int and string. Since there is no such value, f cannot be applied to `A, and `B is the only accepted input. **)
```
- Even if a value has a fixed variant type, one can still give it a larger type through coercions. Coercions are normally written with both the source type and the destination type, but in simple cases the source type may be omitted.
```ocaml
# type 'a wlist = [`Nil | `Cons of 'a * 'a wlist | `Snoc of 'a wlist * 'a];;
type 'a wlist = [ `Cons of 'a * 'a wlist | `Nil | `Snoc of 'a wlist * 'a ]
# let wlist_of_vlist  l = (l : 'a vlist :> 'a wlist);;
val wlist_of_vlist : 'a vlist -> 'a wlist = <fun>
# let open_vlist l = (l : 'a vlist :> [> 'a vlist]);;
val open_vlist : 'a vlist -> [> 'a vlist ] = <fun>
# fun x -> (x :> [`A|`B|`C]);;
- : [< `A | `B | `C ] -> [ `A | `B | `C ] = <fun>
# let split_cases = function
    | `Nil | `Cons _ as x -> `A x
    | `Snoc _ as x -> `B x
  ;;
val split_cases :
  [< `Cons of 'a | `Nil | `Snoc of 'b ] ->
  [> `A of [> `Cons of 'a | `Nil ] | `B of [> `Snoc of 'b ] ] = <fun>
```
- When an or-pattern composed of variant tags is wrapped inside an alias-pattern, the alias is given a type containing only the tags enumerated in the or-pattern. This allows for many useful idioms, like *incremental definition of functions.*
```ocaml
# let num x = `Num x
  let eval1 eval (`Num x) = x
  let rec eval x = eval1 eval x ;;
val num : 'a -> [> `Num of 'a ] = <fun>
val eval1 : 'a -> [< `Num of 'b ] -> 'b = <fun>
val eval : [< `Num of 'a ] -> 'a = <fun>
# let plus x y = `Plus(x,y)
  let eval2 eval = function
    | `Plus(x,y) -> eval x + eval y
    | `Num _ as x -> eval1 eval x
  let rec eval x = eval2 eval x ;;
val plus : 'a -> 'b -> [> `Plus of 'a * 'b ] = <fun>
val eval2 : ('a -> int) -> [< `Num of int | `Plus of 'a * 'a ] -> int = <fun>
val eval : ([< `Num of int | `Plus of 'a * 'a ] as 'a) -> int 


type myvariant = [`Tag1 of int | `Tag2 of bool]
(* #myvariant equivalent to writing (`Tag1(_ : int) | `Tag2(_ : bool)) *)
# let f = function
    | #myvariant -> "myvariant"
    | `Tag3 -> "Tag3";;
val f : [< `Tag1 of int | `Tag2 of bool | `Tag3 ] -> string = <fun>
# let g1 = function `Tag1 _ -> "Tag1" | `Tag2 _ -> "Tag2";;
val g1 : [< `Tag1 of 'a | `Tag2 of 'b ] -> string = <fun>
# let g = function
    | #myvariant as x -> g1 x
    | `Tag3 -> "Tag3";;
val g : [< `Tag1 of int | `Tag2 of bool | `Tag3 ] -> string
```
# Chapter 6 Polymorphism and its limitations
- TODO

# Chapter 7 Generalized algebraic datatypes
Generalized algebraic datatypes, or GADTs, extend usual sum types(ADT) in two ways
- constraints on type parameters may change depending on the value constructor
- some type variables may be existentially quantified
Adding constraints is done by giving an explicit return type, where type parameters are instantiated:
```ocaml
type _ term =
  | Int : int -> int term
  | Add : (int -> int -> int) term
  | App : ('b -> 'a) term * 'b term -> 'a term
```
This return type must use the same type constructor as the type being defined, and have the same number of parameters.
>The constraints associated to each constructor can be recovered through pattern-matching. Namely, if the type of the scrutinee of a pattern-matching contains a locally abstract type, this type can be refined according to the constructor used. These extra constraints are only valid inside the corresponding branch of the pattern-matching. If a constructor has some existential variables, fresh locally abstract types are generated, and they must not escape the scope of this branch.

1. Recursive functions
```ocaml
let rec eval : type a. a term -> a = function
  | Int n    -> n                 (* a = int *)
  | Add      -> (fun x y -> x+y)  (* a = int -> int -> int *)
  | App(f,x) -> (eval f) (eval x)
          (* eval called at types (b->a) and b for fresh b *)

(* And use it *)
let two = eval (App (App (Add, Int 1), Int 1))
val two : int = 2
```
It is **important** to remark that the function eval is using *the polymorphic syntax* for locally abstract types. When defining a recursive function that manipulates a GADT, explicit polymorphic recursion should generally be used. For instance, the following definition fails with a type error:
```ocaml
let rec eval (type a) : a term -> a = function
  | Int n    -> n
  | Add      -> (fun x y -> x+y)
  | App(f,x) -> (eval f) (eval x)
Error: This expression has type ($b -> a) term
       but an expression was expected of type 'a
       The type constructor $b would escape its scope
       Hint: $b is an existential type bound by the constructor App.
(* happens in the branch App(f,x) when eval is called with f as an argument. In this branch, the type of f is ($App_'b -> a) term. The prefix $ in $App_'b denotes an existential type named by the compiler. Since the type of eval is 'a term -> 'a, the call eval f makes the existential type $App_'b flow to the type variable 'a and escape its scope. This triggers the above error. *)
```
In absence of an explicit polymorphic annotation, a monomorphic type is inferred for the recursive function. If a recursive call occurs inside the function definition at a type that involves an existential GADT type variable, this variable flows to the type of the recursive function, and thus escapes its scope.

2. Type inference for GADTs is notoriously hard. This is due to the fact some types may become ambiguous when escaping from a branch.
	- The Int case above, n could have either type int or a, and they are not equivalent outside of that branch. 
	- As a first approximation, type inference will always work if a pattern-matching is annotated with types containing no free type variables (both on the scrutinee and the return type). 
	- This is the case in the above example, thanks to the type annotation containing only locally abstract types.

The exhaustiveness check is aware of GADT constraints, and can automatically infer that some cases cannot happen. For instance, the following pattern matching is correctly seen as exhaustive (the Add case cannot happen).
```ocaml
let get_int : int term -> int = function
  | Int n    -> n
  | App(_,_) -> 0
```

# Chapter 9 Parallel programming
OCaml distinguishes concurrency and parallelism and provides distinct mechanisms for expressing them. Concurrency is overlapped execution of tasks (section [12.24.2](https://ocaml.org/manual/5.2/effects.html#s%3Aeffects-concurrency)) whereas parallelism is simultaneous execution of tasks. In particular, parallel tasks overlap in time but concurrent tasks may or may not overlap in time. Tasks may execute concurrently by yielding control to each other. While concurrency is a program structuring mechanism, parallelism is a mechanism to make your programs run faster. If you are interested in the concurrent programming mechanisms in OCaml, please refer to the section [12.24](https://ocaml.org/manual/5.2/effects.html#s%3Aeffect-handlers) on effect handlers and the chapter [34](https://ocaml.org/manual/5.2/libthreads.html#c%3Athreads) on the threads library.

1. Let us attempt to parallelise the Fibonacci function. The two recursive calls may be executed in parallel. However, naively parallelising the recursive calls by spawning domains for each one will not work as it spawns too many domains.
```ocaml
(* fib_par1.ml *)
let n = try int_of_string Sys.argv.(1) with _ -> 1

let rec fib n =
  if n < 2 then 1 else begin
    let d1 = Domain.spawn (fun _ -> fib (n - 1)) in
    let d2 = Domain.spawn (fun _ -> fib (n - 2)) in
    Domain.join d1 + Domain.join d2
  end

let main () =
  let r = fib n in
  Printf.printf "fib(%d) = %d\n%!" n r

let _ = main ()

ocamlopt -o fib_par1.exe fib_par1.ml
$ ./fib_par1.exe 42
Fatal error: exception Failure("failed to allocate domain")
```
- OCaml has a limit of 128 domains that can be active at the same time. An attempt to spawn more domains will raise an exception. How then can we parallelise the Fibonacci function?

2. The OCaml standard library provides *only low-level primitives* for concurrent and parallel programming, leaving high-level programming libraries to be developed and distributed outside the core compiler distribution. [Domainslib](https://github.com/ocaml-multicore/domainslib) is such a library for nested-parallel programming, which is epitomised by the parallelism available in the recursive Fibonacci computation. Let us use domainslib to parallelise the recursive Fibonacci program.
- Domainslib provides an async/await mechanism for spawning parallel tasks and awaiting their results. On top of this mechanism, domainslib provides parallel iterators. At its core, domainslib has an efficient implementation of *[[Parallel/work-stealing queue\|work-stealing queue]]* in order to efficiently share tasks with other domains. A parallel implementation of the Fibonacci program is given below.
```ocaml
(* fib_par2.ml *)
let num_domains = int_of_string Sys.argv.(1)
let n = int_of_string Sys.argv.(2)

let rec fib n = if n < 2 then 1 else fib (n - 1) + fib (n - 2)

module T = Domainslib.Task

let rec fib_par pool n =
  if n > 20 then begin
    let a = T.async pool (fun _ -> fib_par pool (n-1)) in
    let b = T.async pool (fun _ -> fib_par pool (n-2)) in
    T.await pool a + T.await pool b
  end else fib n

let main () =
  let pool = T.setup_pool ~num_domains:(num_domains - 1) () in
  let res = T.run pool (fun _ -> fib_par pool n) in
  T.teardown_pool pool;
  Printf.printf "fib(%d) = %d\n" n res

let _ = main ()
```
3. Parallel garbage collection
- An important aspect of the scalability of parallel OCaml programs is the scalability of the garbage collector (GC). The OCaml GC is designed to have both low latency and good parallel scalability. OCaml has a generational garbage collector with a small minor heap and a large major heap. 
- New objects (upto a certain size) are allocated in the minor heap. Each domain has its own domain-local minor heap arena into which new objects are allocated without synchronising with the other domains. When a domain exhausts its minor heap arena, it calls for a stop-the-world collection of the minor heaps. In the stop-the-world section, all the domains collect their minor heap arenas in parallel evacuating the survivors to the major heap.
- For the major heap, each domain maintains domain-local, size-segmented pools of memory into which large objects and survivors from the minor collection are allocated. Having domain-local pools avoids synchronisation for most major heap allocations. The major heap is collected by a concurrent mark-and-sweep algorithm that involves a few short stop-the-world pauses for each major cycle.
- Overall, the users should expect the garbage collector to scale well with increasing number of domains, with the latency remaining low. For more information on the design and evaluation of the garbage collector, please have a look at the ICFP 2020 paper on [Retrofitting Parallelism onto OCaml](https://arxiv.org/abs/2004.11663).

- [Compiler Development: Rust or OCaml?](https://hirrolot.github.io/posts/compiler-development-rust-or-ocaml.html#)
- [The OCaml Manual](https://ocaml.org/manual/5.2/index.html#)
- [Is the original OCaml compiler still available?](https://www.reddit.com/r/rust/comments/18b808/is_the_original_ocaml_compiler_still_available/)
- [Flamebait question, I know. But why OCaml over Rust?](https://www.reddit.com/r/ocaml/comments/fefcfv/flamebait_question_i_know_but_why_ocaml_over_rust/)