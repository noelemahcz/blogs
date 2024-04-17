# Ω Combinator


The definition in λ-calculus:

``` haskell
Ω = (λx.x x) (λx.x x)
```


In haskell, we need to define the recursive function type first:

``` haskell
-- (Rec a) and (Rec a -> a) are isomorphic,
-- and have the same object representation in memory.
newtype Rec a = Rec (Rec a -> a)

{-# NOINLINE unRec #-}
unRec :: Rec a -> Rec a -> a
unRec (Rec f) = f
```

> The intent of the NOINLINE pragma above is to prevent GHC's inliner from working on functions with parameters of recursive types, see [Known bugs in GHC][1].


Then define the **Ω combinator** consistent with the λ-calculus form:

``` haskell
omega =
  (\x -> (unRec x) x)
  $ Rec
  (\x -> (unRec x) x)
```

Of course `omega` is useless because it doesn't do anything.

Let's add some extra logic/behavior to it:

``` haskell
omegaPrint =
  (\x -> putStrLn "Ω" >> (unRec x) x)
  $ Rec
  (\x -> putStrLn "Ω" >> (unRec x) x)
```

Or a more concise form:

``` haskell
omegaPrint = f (Rec f)
  where f x = putStrLn "Ω" >> (unRec x) x
```

Evaluating `omegaPrint` will result in printing infinitely.


# Y Combinator


According to the definition of `omegaPrint`, if we extract the printing behaviour outside the function:

``` haskell
omegaWithExtraBehavior f =
  (\x -> f $ (unRec x) x)
  $ Rec
  (\x -> f $ (unRec x) x)

print x = putStrLn "Ω" >> x

omegaPrint = omegaWithExtraBehavior print
```

The `omegaWithExtraBehavior` here is actually the **Y Combinator** (or Fixed-point Combinator) in Haskell.

``` haskell
-- The Y Combinator
y f =
  (\x -> f $ (unRec x) x)
  $ Rec
  (\x -> f $ (unRec x) x)

printNTimes x n = when (n /= 0) $ putStrLn "Y" >> x (n - 1)

yPrintNTimes = y g
```

Refer to its definition in λ-calculus:

``` haskell
Y = λf. (λx.f (x x)) (λx.f (x x))
```


[1]: https://downloads.haskell.org/ghc/latest/docs/users_guide/bugs.html#bugs-in-ghc
