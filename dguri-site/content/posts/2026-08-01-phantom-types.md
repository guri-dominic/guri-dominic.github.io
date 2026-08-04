+++
title = "Phantom Types for Safer Coordinate Transforms"
date = 2026-08-03
+++

Over the past few months, I have enjoyed learning Haskell on and off. Haskell piqued my interest because of its weirdness and notoriety online. I started reading about it and eventually picked up *Effective Haskell* by Rebecca Skinner, which I have been working through whenever I have time.

As is often the case with Haskell ideas, I found phantom types interesting but I was not sure how to apply them to domains I care about, like robotics.

Enter [Sguaba](https://blog.helsing.ai/posts/sguaba-hard-to-misuse-rigid-body-transforms-for-engineers-with-other-things-to-worry-about-than-line/), a Rust rigid-body transformation library designed to be hard to misuse. Yes, a Rust library showed me how phantom types, an idea I first encountered in Haskell, can be useful in practice.

I am still a beginner in Haskell, and I know even less about Rust. However, phantom types are a feature I now hope to use in spatial-transformation libraries going forward.

The basic idea is simple: a phantom type parameter appears in a type but is not represented in the underlying runtime data. Instead, it carries information that the compiler can use for type checking.

In a spatial-transformation library, phantom types can label coordinate frames. The compiler can then reject attempts to apply a transform to a coordinate expressed in the wrong frame, or to compose transforms whose intermediate frames do not match.

This does not prove that the numerical transformation is correct, but it does eliminate an important class of bookkeeping errors.

---

## The Original Problem

Starting with vectors and matrices such as `V3` and `Matrix4`, we might define a function that transforms a point from one coordinate frame to another as:

```haskell
transform' :: Matrix4 Double -> V3 Double -> V3 Double
```

The first argument is a transformation matrix, the second is a point, and the result is another point. However, the type signature says nothing about the coordinate frames to which the points belong or frames the matrix is supposed to map to and fro.

Hence, the user is responsible for tracking the frames and ensuring that the correct transformation is applied to the right points. That bookkeeping becomes increasingly difficult as the transformation tree grows.

## How Phantom Types Help

I think of a phantom type as an annotation attached to a wrapper around some underlying data. For example:

```haskell
newtype Coordinate frame = Coordinate (V3 Double)
```

The `frame` parameter does not correspond to a value stored inside `Coordinate`. It exists only at the type level, where the compiler can use it.

We can introduce empty marker types for the coordinate frames we care about:

```haskell
data World
data Body
data Camera
```

These types have no constructors, but they are still distinct types. 

We can now represent a point expressed in the camera frame:

```haskell
cameraPoint :: Coordinate Camera
cameraPoint = undefined   
-- `undefined` allows us compile and validate the compile-time correctness
```

Similarly, we can define a transformation type with `from` and `to` phantom parameters:

```haskell
newtype Transform from to = Transform (Matrix4 Double)
```

Now, a value of type `Transform World Body` maps coordinates expressed in `World` into coordinates expressed in `Body`.

The transformation function can now encode that relationship directly:

```haskell
transform :: Transform from to -> Coordinate from -> Coordinate to
```

For example:

```haskell
worldToBody :: Transform World Body
worldToBody = undefined

cameraPoint :: Coordinate Camera
cameraPoint = undefined

transform worldToBody cameraPoint       -- Compile-time error: expected Coordinate World

cameraToBody :: Transform Camera Body
cameraToBody = undefined

transform cameraToBody cameraPoint      -- Compiles successfully
```

The compiler rejects the first application because `worldToBody` expects a `Coordinate World`, not a `Coordinate Camera`.

We can also encode valid transformation composition:

```haskell
compose :: Transform a b -> Transform b c -> Transform a c
```

The first transform maps from `a` to `b`, and the second maps from `b` to `c`. The type signature requires the codomain of the first transform to match the domain of the second, and the compiler enforces this constraint.

## Minimal Haskell Example

The following example uses `Double` in place of vectors and matrices so that the frame-checking behavior is easier to see:

```haskell
{-# LANGUAGE EmptyDataDecls #-}
{-# LANGUAGE RoleAnnotations #-}

module Main where

data World
data Body
data Camera

newtype Coordinate frame = Coordinate Double
newtype Transform from to = Transform Double

-- role is required to ensure uses empty types as phantom types
type role Coordinate nominal
type role Transform nominal nominal

transform :: Transform from to -> Coordinate from -> Coordinate to
transform (Transform t) (Coordinate x) = Coordinate (t + x)

compose :: Transform a b -> Transform b c -> Transform a c
compose (Transform x) (Transform y) = Transform (x + y)

worldToBody :: Transform World Body
worldToBody = Transform 1

cameraPoint :: Coordinate Camera
cameraPoint = Coordinate 2

t = transform worldToBody cameraPoint
```

The role annotations are important in a library that treats frame labels as a safety boundary. Because the parameters are otherwise absent from the runtime representation, the Haskell compiler then uses them as phantom types, otherwise, it which `coerce` to bypass the distinction.

Loading the file in GHCi produces a clear error:

```text
phantoms.hs:25:27: error: [GHC-83865]
    • Couldn't match type ‘Camera’ with ‘World’
      Expected: Coordinate World
        Actual: Coordinate Camera
    • In the second argument of ‘transform’, namely ‘cameraPoint’
      In the expression: transform worldToBody cameraPoint
      In an equation for ‘t’: t = transform worldToBody cameraPoint
```


---

## A C++ Approach

I wanted to implement the same idea in C++ so that it would be easier to integrate into my workflow. C++ templates can provide the same compile-time frame labels.

First, we define a base tag and several frame types:

```cpp
struct Frame {};

struct WorldFrame  : Frame {};
struct BodyFrame   : Frame {};
struct CameraFrame : Frame {};
```

The tags do not need to be stored in a coordinate or transform. They only need to participate in the static type.

### Constraining Frames with Concepts

A C++20 concept can prevent unrelated types from being used as coordinate frames:

```cpp
#include <concepts>

template<class T>
concept CoordinateFrame = std::derived_from<T, Frame>;
```

For example:

```cpp
static_assert(CoordinateFrame<WorldFrame>);
static_assert(!CoordinateFrame<int>);
```

We can then define a coordinate whose frame is part of its type:

```cpp
template<CoordinateFrame F>
class Coordinate {
public:
    explicit Coordinate(Eigen::Vector3d value)
        : value_(std::move(value)) {}

    const Eigen::Vector3d& value() const {
        return value_;
    }

private:
    Eigen::Vector3d value_;
};
```

`Coordinate<WorldFrame>` and `Coordinate<BodyFrame>` contain the same kind of numerical data, but they are different C++ types. The compiler therefore rejects assignments between them:

```cpp
Coordinate<WorldFrame> world_point{Eigen::Vector3d::Zero()};
Coordinate<BodyFrame> body_point{Eigen::Vector3d::Zero()};

world_point = body_point; // Compile-time error
```

### A Typed Transform

The transform type carries both its source and destination frames:

```cpp
#include <concepts>
#include <utility>
#include <Eigen/Core>
#include <Eigen/Geometry>

struct Frame {};

struct WorldFrame  : Frame {};
struct BodyFrame   : Frame {};
struct CameraFrame : Frame {};

template<class T>
concept CoordinateFrame = std::derived_from<T, Frame>;

template<CoordinateFrame F>
class Coordinate {
public:
    explicit Coordinate(Eigen::Vector3d value)
        : value_(std::move(value)) {}

    const Eigen::Vector3d& value() const {
        return value_;
    }

private:
    Eigen::Vector3d value_;
};

template<CoordinateFrame From, CoordinateFrame To>
class Transform {
public:
    Transform(Eigen::Quaterniond rotation,
              Eigen::Vector3d translation)
        : rotation_(std::move(rotation)),
          translation_(std::move(translation)) {}

    Coordinate<To> operator()(
        const Coordinate<From>& point) const
    {
        return Coordinate<To>{
            rotation_ * point.value() + translation_
        };
    }

    Transform<To, From> inverse() const {
        const Eigen::Quaterniond inverse_rotation =
            rotation_.inverse();

        return Transform<To, From>{
            inverse_rotation,
            -(inverse_rotation * translation_)
        };
    }

    template<CoordinateFrame Next>
    Transform<From, Next> and_then(
        const Transform<To, Next>& next) const
    {
        return Transform<From, Next>{
            next.rotation_ * rotation_,
            next.rotation_ * translation_ + next.translation_
        };
    }

private:
    Eigen::Quaterniond rotation_;
    Eigen::Vector3d translation_;

    template<CoordinateFrame, CoordinateFrame>
    friend class Transform;
};
```

The call operator applies only to a coordinate in the transform's source frame:

```cpp
Transform<WorldFrame, BodyFrame> world_to_body{
    Eigen::Quaterniond::Identity(),
    Eigen::Vector3d{1.0, 0.0, 0.0}
};

Coordinate<WorldFrame> world_point{
    Eigen::Vector3d{2.0, 3.0, 4.0}
};

Coordinate<CameraFrame> camera_point{
    Eigen::Vector3d{2.0, 3.0, 4.0}
};

auto body_point = world_to_body(world_point);   // Compiles
auto invalid = world_to_body(camera_point);     // Compile-time error
```

For the invalid call, GCC reports:

```text
error: no match for call to
‘(Transform<WorldFrame, BodyFrame>) (Coordinate<CameraFrame>&)’

note: no known conversion for argument 1 from
‘Coordinate<CameraFrame>’ to ‘const Coordinate<WorldFrame>&’
```

The compiler identifies the actual semantic error: a camera-frame point was passed to a transform that expects a world-frame point.

### Composition as `and_then`

The `and_then` member function is transform composition written in left-to-right application order:

```cpp
Transform<WorldFrame, BodyFrame> world_to_body{
    Eigen::Quaterniond::Identity(),
    Eigen::Vector3d::Zero()
};

Transform<BodyFrame, CameraFrame> body_to_camera{
    Eigen::Quaterniond::Identity(),
    Eigen::Vector3d::Zero()
};

auto world_to_camera = world_to_body.and_then(body_to_camera);
```

The chain is:

```text
WorldFrame -> BodyFrame -> CameraFrame
```

The inferred result type is:

```cpp
Transform<WorldFrame, CameraFrame>
```

The method signature enforces the matching intermediate frame:

```cpp
template<CoordinateFrame Next>
Transform<From, Next> and_then(
    const Transform<To, Next>& next) const;
```

For a `Transform<WorldFrame, BodyFrame>`, the argument must begin at `BodyFrame`. A transform beginning at any other frame is rejected.

For example, these two transforms cannot be composed because both end at `CameraFrame`:

```cpp
Transform<WorldFrame, CameraFrame> world_to_camera{
    Eigen::Quaterniond::Identity(),
    Eigen::Vector3d::Zero()
};

Transform<BodyFrame, CameraFrame> body_to_camera{
    Eigen::Quaterniond::Identity(),
    Eigen::Vector3d::Zero()
};

auto invalid = world_to_camera.and_then(body_to_camera);
```

### Testing the Implementation

The following `main()` checks transform application, composition, and inversion:

```cpp
#include <cassert>
#include <iostream>
#include <numbers>

int main()
{
    Transform<WorldFrame, BodyFrame> world_to_body{
        Eigen::Quaterniond::Identity(),
        Eigen::Vector3d{1.0, 0.0, 0.0}
    };

    Transform<BodyFrame, CameraFrame> body_to_camera{
        Eigen::Quaterniond{
            Eigen::AngleAxisd{
                std::numbers::pi / 2.0,
                Eigen::Vector3d::UnitZ()
            }
        },
        Eigen::Vector3d{0.0, 1.0, 0.0}
    };

    Coordinate<WorldFrame> point_world{
        Eigen::Vector3d{2.0, 3.0, 4.0}
    };

    const Coordinate<BodyFrame> point_body =
        world_to_body(point_world);

    const Coordinate<CameraFrame> point_camera_sequential =
        body_to_camera(point_body);

    const Transform<WorldFrame, CameraFrame> world_to_camera =
        world_to_body.and_then(body_to_camera);

    const Coordinate<CameraFrame> point_camera_composed =
        world_to_camera(point_world);

    assert(
        point_camera_sequential.value().isApprox(
            point_camera_composed.value()
        )
    );

    const Transform<CameraFrame, WorldFrame> camera_to_world =
        world_to_camera.inverse();

    const Coordinate<WorldFrame> recovered_world_point =
        camera_to_world(point_camera_composed);

    assert(
        recovered_world_point.value().isApprox(
            point_world.value()
        )
    );

    std::cout << "All runtime tests passed.\n";
}
```

On a typical Linux installation with Eigen in `/usr/include/eigen3`, it can be compiled with:

```bash
g++ -std=c++20 -Wall -Wextra -Wpedantic -O2 \
    -I/usr/include/eigen3 \
    phantom-types.cpp -o phantom-types

./phantom-types
```

The expected output is:

```text
All runtime tests passed.
```

The runtime tests verify the transform mathematics. The type system separately prevents frame-incompatible operations from compiling.

## What the Types Cannot Prove

The compiler can verify that the labels match, but it cannot verify that a quaternion loaded is truly a unit-quaternion.

The public constructor keeps this example short. In a production library, I would likely hide raw construction behind smart constructors that validate properties such as quaternion normalization and finite translation values. External data would still form a trust boundary, especially if it does not carry reliable frame metadata.

## Existing C++ Libraries

In the course of writing this post, I learned about [refx](https://github.com/mosaico-labs/refx) and [wave_geometry](https://github.com/wavelab/wave_geometry), two C++ spatial-transformation libraries that use the same general phantom-type style to enforce coordinate-frame compatibility at compile time. This is therefore not a new technique. The useful part of this exercise was implementing the pattern directly and seeing how naturally it maps from Haskell to modern C++. For me, phantom types no longer an esoteric Haskell feature, but they are now a practical tool for making one of the common spatial transformation mistakes harder to express.
