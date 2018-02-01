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

    * [Memo1.lhs][memo1]

[hgl]: https://hackage.haskell.org/package/HGL
[caasi-soe]: https://github.com/caasi/school-of-expression
[memo1]: https://hackage.haskell.org/package/IrrHaskell-0.2/src/Memo1.lhs

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

```ActionScript
public class DisplayObject {}

public class Rectangle extends DisplayObject {}

public class Ellipse extends DisplayObject {}

public class RtTriangle extends DisplayObject {}

public class Polygon extends DisplayObject {}
```

---

# Teaching Techniques

  * Use `IO` without introducing Monads.

  * Introduce higher-order functions later.

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

```ActionScript
public class DisplayObject {
  public var parent: DisplayObjectContainer;
  // translation
  public var x: Number;
  public var y: Number;
  // scaling
  public var scaleX: Number;
  public var scaleY: Number;
  // rotation
  public var rotationX: Number;
  public var rotationY: Number;
  // complement, union, intersect
  public var blendMode: String;
  // or group them in a matrix
}

// the Container Pattern
public class DisplayObjectContainer extends DisplayObject {
  private var _children: Array; // Vector.<DisplayObject>
}
```

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

# Modeling the Problem in OOP: Picture

```ActionScript
class DisplayObject {
  // or `paintComponent` in Java Swing
  public function render() {}
}

class Rectangle extends DisplayObject {
  public function render() {}
}

class DisplayObjectContainer extends DisplayObject {
  public function render() {}
}
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

# Animation in JavaScript

```JavaScript
// Time -> a
var evalCanvas = function(t) { /* ... */ };

var parseOne = function(src) {
  if (match = src.match(/something/))
    return function(t) { /* ... */ }
  if (match = src.match(/something else/))
    return function(t) { /* ... */ }
}

var parseTerm = function(src) {
  /* do something */

  return function(t) { /* ... */ }
}

// lift0 pi
var PI = function(t) { return ['num', Math.PI] };
```

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

# Without Type Classes

```JavaScript
var parseExpr = function(src) {
  /* do something */

  return function(t) {
    var a = term_a(t)
    var b = term_b(t)

    return ['num', op(a, b)]
  }
}
```

---

# Modeling the Problem: Reactive Animation

  * Hard mode

  * DSL

```
newtype Beh a = Beh (Time -> a)

newtype Behavior1 a = Behavior1 ([(UserAction, Time)] -> Time -> a)

newtype Behavior2 a = Behavior2 ([(UserAction, Time)] -> [Time] -> a)

newtype Behavior3 a = Behavior3 ([UserAction] -> [Time] -> a)

-- (a^b)^c
newtype Behavior4 a = Behavior4 ([Maybe UserAction] -> [Time] -> a)

-- a^(b*c)
newtype Behavior a = Behavior (([Maybe UserAction], [Time]) -> a)

type Event a = Behavior (Maybe a)
```

---

# What is an UserAction anyway?

```
data G.Event
  = Key { char :: Char, isDown :: Bool }
  | Button { pt :: Point, isLeft :: Bool, isDown :: Bool }
  | MouseMove { pt :: Point }
  | Resize
  | Closed
  deriving Show

type UserAction = G.Event
```

---

# the UserAction List

```
uts =
  ( [ Nothing
    , Just (Button (10, 10) True True)
    , Just (MouseMove (15, 15))
    ..
    ]
  , [ 0.016
    , 0.032
    , 0.048
    ..
    ]
  )
```

---

# Events

```
lbp :: Event ()
lbp = Event (\(uas, _) -> map getlbp uas)
        where getlbp (Just (Button _ True True)) = Just ()
              getlbp _                           = Nothing
-- lbp uts
-- => [Nothing, Just (), Nothing..]
```

```
(=>>) :: Event a -> (a -> b) -> Event b
Event fe =>> f = Event (map (fmap f) . fe)

(->>) :: Event a -> b -> Event b
e ->> v = e =>> (\_ -> v)

lbp ->> blue :: Event (Behavior Color)
-- lbp ->> blue $ uts
-- => [Nothing, Just (Behavior Blue), Nothing..]
```

---

# Behaviors

```
untilB :: Behavior a -> Event (Behavior a) -> Behavior a
Behavior fb `untilB` Event fe
  = memoB $ Behavior (\uts@(us, ts) -> loop us ts (fe uts) (fb uts))
      where loop (_:us) (_:ts) ~(e:es) (b:bs)
              = b : case e of
                      Nothing             -> loop us ts es bs
                      Just (Behavior fb') -> fb' (us, ts)
```

```
red `untilB` (lbp ->> blue) :: Behavior Color
-- red `untilB` (lbp ->> blue) $ uts
-- => [Red, Blue, Blue..]
```

---

# Functional Reactive Animation

  * Compose small behaviors(`([Maybe UserAction], [Time]) -> [a]`) into a big behavior(`Behavior Picture`)

  * Transform `([Maybe UserAction], [Time]) -> [Picture]` into a series of `IO () >> IO () >> IO () >> ...` actions

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

