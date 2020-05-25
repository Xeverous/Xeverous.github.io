---
layout: category
---

## how to simplify??

```c++
template <typename T>
struct foo
{
	explicit foo(const T&);

	// when T is a reference, this will collapse to T& and clash with other overload
	explicit foo(T&&);

	// SFINAE - attempt 1
	template <typename U = T>
	explicit foo(
		T&& val,
		std::enable_if_t<!std::is_reference_v<U>, std::nullptr_t> = nullptr);

	// SFINAE - attempt 2
	template <typename U = T, typename = IsReference<U>>
	explicit foo(T&& val);

};
```
