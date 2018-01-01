class: inverse, center, middle

# the Haskell School of Expression
## a Book Review
### caasih

2018.01.04

---

# Can I run it today?

  * [HGL][hgl]

    * `Graphics.SOE`

  * [caasi/school-of-expression][caasi-soe]

    * XQuartz

    * `export LIBRARY_PATH="/opt/X11/lib:$LIBRARY_PATH"`

  [hgl]: https://hackage.haskell.org/package/HGL
  [caasi-soe]: https://github.com/caasi/school-of-expression

---

# Modeling the Problem

  * Shape

  * Region

  * Picture

  * Animation

  * Reactive Animation

---

# Modeling the Problem: Shape

```
data Shape
  = Rectangle Side Side
  | Ellipse Radius Radius
  | RtTriangle Side Side
  | Polygon [Vertex]
  deriving Show

type Radius = Float
type Side = Float
type Vertex = (Float, Float)
```

---

# Modeling the Problem in OOP: Shape

  * WIP

---

# Teaching Techniques

  * Use `IO` without introducing Monads.

  * Introduce high-order functions later.

  * Introduce recursive data structures later.

---

# Modeling the Problem: Region

  * Collection of Shapes

```
data Region
  = Shape Shape
  | Translate Vector Region
  | Scale Vector Region
  | Complement Region
  | Region `Union` Region
  | Region `Intersect` Region
  | Empty
  deriving Show

type Vector = (Float, Float)
```

---

# Teaching Techniques?

  * Hide low-level details in the library(`Graphics.SOE`).

  * Introduce infix operators later.

---

# Modeling the Problem in OOP: Region

  * WIP

---

# Modeling the Problem: Picture

  * The bridge between our data and the graphic library

```
data Picture
  = Region Color Region
  | Picture `Over` Picture
  | EmptyPic
  deriving Show
```

---

# Modeling the Problem: Animation

  * Easy mode

```
type Animation a = Time -> a
type Time = Float

rubberBall :: Animation Shape
rubberBall t = Ellipse (sin t) (cos t)
```

---

# Modeling the Problem in OOP: Animation

  * WIP: CanvasDesigner here

---

# Modeling the Problem: Animation

  * Normal mode

  * The power of type classes

```
newtype Behavior a = Beh (Time -> a)

instance Num a => Num (Behavior a) where
  (+) = lift2 (+)
  -- ...

instance Floating a => Floating (Behavior a) where
  pi = lift0 pi
  sin = lift1 sin
  cos = lift1 cos
  -- ...

time :: Behavior Time
time = Beh (\t -> t)
```

---

# Modeling the Problem: Animation

```
flashingBall :: Behavior Picture
flashingBall
  = let ball = shape (ell 0.2 0.2)
        flash = cond (sin time >* 0) red yellow
        cond = lift3 (\p c a -> if p then c else a)
    in reg (timeTrans (8 * time) flash)
           (translate (sin time) (cos time) ball)
```

---

# Modeling the Problem: Reactive Animation

  * DSL

  * Hard mode

```
newtype Beh a = Beh (Time -> a)

newtype Behavior1 a = Behavior1 ([(UserAction, Time)] -> Time -> a)

newtype Behavior2 a = Behavior2 ([(UserAction, Time)] -> [Time] -> a)

newtype Behavior3 a = Behavior3 ([UserAction] -> [Time] -> a)

newtype Behavior a = Behavior ([Maybe UserAction] -> [Time] -> a)

type Event a = Behavior (Maybe a)
```

---

# Modoling the Problem: Reactive Animation

  * WIP

---

# There is More!

  * An Imperative Robot Language

  * Functional Music Composition

---

# References

  * [the Haskell School of Expression: Learning Functional Programming through Multimedia][soe]

  * [CanvasDesigner][canvas-designer]

  * [Functional Reactive Animation][fra]

[soe]: https://www.amazon.com/Haskell-School-Expression-Functional-Programming/dp/0521644089
[canvas-designer]: https://github.com/CindyLinz/CanvasDesigner
[fra]: http://conal.net/papers/icfp97/

---

class: inverse, center, middle

# fin

