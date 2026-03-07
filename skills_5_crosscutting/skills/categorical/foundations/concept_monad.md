# Concept: Monad — (T,η,μ)
## T:C→C endofunctor, η:Id⇒T unit, μ:T²⇒T mult. Laws: left unit, right unit, associativity.
## Bind form: bind(pure(x),f)=f(x); bind(m,pure)=m; bind(bind(m,f),g)=bind(m,λx.bind(f(x),g)).
## Kleisli: C_T objects=Ob(C), morphisms A→T(B), composition via bind. Composition IS Kleisli(Monad).
## Sources: Mac Lane (1971) Ch.VI; Moggi (1991) Info&Comp 93(1):55–92; Wadler (1995) AFP Springer.
