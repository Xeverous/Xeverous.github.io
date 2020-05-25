---
layout: article
---

There has been a lot of confusion around how to allocate a multidimentional array (I had it too while learning) so here is a detailed article about this sole problem.

First, let's explain a 1-dimentional array. I will allocate for `double`s so that it will be easier to distinguish between array type and integers that just store array size.

```c++
const int size = 1024;
double* arr = new double[size];
// ...
delete[] arr;
```

Things you should notice:

- Arrays are handled through pointers.

How is it going to affect multi-dimentional arrays?

- Multidimentional arrays are arrays of arrays. Adding more dimentions requires loops.
- Arrays are handled through pointers. Arrays of arrays will be handled through pointers to pointers.

```c++
const int size_x = 64;
const int size_y = 32;

// allocation
double** arr = new double*[size_y]; // allocate pointers
for (int y = 0; y < size_y; ++y)
	arr[y] = new double[size_x]; // allocate elements

// usage
for (int y = 0; y < size_y; ++y)
	for (int x = 0; x < size_x; ++x)
		// use arr[y][x]

// deallocation
for (int y = 0; y < size_y; ++y)
	delete[] arr[y]; // deallocate elements
delete[] arr; // deallocate pointers
```

Remember that the first line does not create elements, but pointers to elements. Each of these pointers needs to be initialized. In total we perform `size_y + 1` allocations. Cleanup code is doing exactly reverse - first we deallocate in a loop elements allocated by each pointer and then the array of pointers.

To make it easier so see the pattern, here is a 3-dimentional array:

```c++
const int size_x = 64;
const int size_y = 32;
const int size_z = 8;

// allocation
double*** arr = new double**[size_z];
for (int z = 0; z < size_z; ++z)
{
	arr[z] = new double*[size_y];
	for (int y = 0; y < size_y; ++y)
		arr[z][y] = new double[size_x];
}

// usage
for (int z = 0; z < size_z; ++z)
	for (int y = 0; y < size_y; ++y)
		for (int x = 0; x < size_x; ++x)
			// use arr[z][y][x]

// deallocation
for (int z = 0; z < size_z; ++z)
{
	for (int y = 0; y < size_y; ++y)
		delete[] arr[z][y];
	delete[] arr[z];
}
delete[] arr;
```

In total, we perform `size_z * size_y + size_z + 1` allocations/deallocations.

## N dimentions

Obviously we can write a template that will create a nested array of any dimentions, the pleasuring implementation can be found in templates tutorial TODO link.

## in practice

Multidimentional arrays are rarely used. Each additional dimention complicates code and negatively impacts performance - a better approach is to just allocate a 1-dimentional array and map indexes:

```c++
const int size_x = 64;
const int size_y = 32;

// allocation
double* arr = new double[size_x * size_y];

// usage for element at pos (x, y)
arr[y * size_x + x]

// deallocation
delete[] arr;
```

Such code can be put into further abstractions to make working with multidimentional arrays easier.
