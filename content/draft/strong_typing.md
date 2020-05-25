---
layout: article
---

## ???

- expressive code (readability)
- catching errors at compile time
- explain `typedef` does not create new type?

This article presents general concept, pretty much everything of this article (except C++ `explicit` keyword) will apply to any programming language which supports OOP.

## the problem

```c++
class circle
{
public:
	explicit circle(double radius);
	explicit circle(double diameter);
};
```

## factory method

```c++
class circle
{
public:
	circle() = default;

	static circle from_radius(double radius)
	{ return circle(radius); }

	static circle from_diameter(double diameter)
	{ return circle(diameter / 2); }

private:
	explicit circle(double radius)
	: radius(radius) {}

	radius = 0;
};
```

- pros:
  - very simple to implement
  - very good for polymorphism
- cons:
  - only for ctors (?)

## the builder pattern

- pros:
  - very good if you can easily provide reasonable defaults
  - very clear, expressive code
- cons:
  - may incur overhead (object copy or inefficient initialization)
  - moves error checking to runtime

## tag dispatching

```c++
struct radius_tag {};
struct diameter_tag {};

class circle
{
public:
	explicit circle(radius_tag, double radius);
	explicit circle(diameter_tag, double diameter);

private:
	m_radius = 0;
};
```

- pros:
  - very good for multiple function template overloads
- cons:
  - scales well only for 1 parameter where all choices are exclusive

performance impact - ???

Notes:

- types designated to be tags should be named `*_tag` or `*_t` for brevity
- tag should be taken as the first parameter

## custom literal suffixes

Very good for units. Basically the way `std::chrono` and `boost::units` are implemented.

```c++
// example
```

## strong typedef - previous works

The idea is not new:

- libraries:
  - `BOOST_STRONG_TYPEDEF` - Robert Ramley, 2002
  - `true_typedef` - Matthew Wilson, 2003
  - strong `typedef` for integer / floating-point types - Akira Takahashi, 2012
  - `DESALT_NEWTYPE` - Oyama Koichi, 2013
  - `type_safe` - Jonathan Müller, 2016
- proposals:
  - Toward Opaque Typedefs for C++1y - N3741, N3515, [P0109](wg21.link/P0109), Walter Brown, 2015
  - newtype - [P0027](wg21.link/P0027) - Troy Korjuslommi

## the naive implementation

```c++
template <typename T>
class named_type
{
public:
	explicit named_type(const T& val)
	: val(val) {}
	explicit named_type(T&& val)
	: val(std::move(val)) {}

	const T& value() const noexcept { return val; }
	T& value() noexcept { return value; }

private:
	T val;
};

// oops, these are still the same types
using radius_t   = named_type<double>;
using diameter_t = named_type<double>;
```

The fix - unused template type name

```c++
template <typename T, typename /* Tag */>
class named_type {/* ... */};

// note that we do not even need to define tags, forward declaration is enough
using radius_t   = named_type<double, struct radius_tag>;
using diameter_t = named_type<double, struct diameter_tag>;
```

## performance impact

- Zero cost with `-O2` or sometimes even with `-O1`.
- Negative cost when strict aliasing optimizations kicks in

## potential overuse

Any form of strong typing requires more code. That's not necessarily bad, but more code may mean more bugs or just more work.

```c++
button.set_position(1, 2);
```

Using strong typing in such case would be a waste. Unless you work with idiots who would implement a different order than (X, Y).

___

Sources:

- [Meeting C++ 2017: Jonathan Boccara "Strong types for strong interfaces"](https://www.youtube.com/watch?v=WVleZqzTw2k)
