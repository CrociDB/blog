---
title: "Simple Vector Implementation in C"
description: ""
date: 2023-10-08T01:58:51+02:00
draft: false
math: true
tags:
 - programming
 - programming-languages
 - c
---

 I've been watching the excellent series [Creating a Compiler from Scratch, Recreationally](https://www.youtube.com/playlist?list=PLzaJkF8lpTDa-DAU4w6Znk4k1SmqCTNm-) by [nashiora](https://github.com/nashiora) where he's writing a compiler for his language [Laye](https://codeberg.org/laye-lang/laye), that is C-compatible, in C. I'm having so much fun and I'm learning so much watching that, that I couldn't recommend more. Please go there, subscribe to his channel, and watch the videos if you're interested in programming languages, or just want to watch a skilled C programmer writing code.

One thing, though, that caught my eyes on the second entry of the series is his quick implementation of a vector. I don't know if that's a common practice or not, because I never had to implement one, I usually would just allocate a big enough array and hope for the best, but this implementation looked very interesting to me for reasons such as 1) it's simple; 2) most of the implementation is in macros; 3) it uses regular arrays and keep the metadata _hidden_.

I'm still on the [second episode in the series](https://www.youtube.com/watch?v=CwnPpX9Omu0), where he actually introduces that part of the code, but I saw on the source code that he's already expanded that. For that reason, I will only use an excerpt of the code enought to expose my point. The current complete implementation [is](https://codeberg.org/laye-lang/laye/src/branch/main/include/layec/vector.h) [here](https://codeberg.org/laye-lang/laye/src/branch/main/lib/vector.c). Also for simplification I included the implementation of the function `layec_vector_maybe_expand` together.

> Obs: worth mentioning that I didn't write the original source code, but its license is MIT. So if you're using any of this, make sure to credit the original author.

```c
/// Header data for a light-weight implelentation of typed vectors.
struct layec_vector_header
{
    long long capacity;
    long long count;
};

void layec_vector_maybe_expand(void** vector_ref, long long element_size, long long required_count);

#define vector(T) T*
#define vector_get_header(V) (((struct layec_vector_header*)(V)) - 1)
#define vector_count(V) ((V) ? vector_get_header(V)->count : 0)
#define vector_push(V, E) do { layec_vector_maybe_expand((void**)&(V), (long long)sizeof *(V), vector_count(V) + 1); (V)[vector_count(V)] = E; vector_get_header(V)->count++; } while (0)
#define vector_pop(V) do { if (vector_get_header(V)->count) vector_get_header(V)->count--; } while (0)
#define vector_free(V) do { if (V) { memset(V, 0, (unsigned long long)vector_count(V) * (sizeof *(V))); free(vector_get_header(V)); (V) = NULL; } } while (0)
#define vector_free_all(V, F) do { if (V) { for (long long vector_index = 0; vector_index < vector_count(V); vector_index++) F((V)[vector_index]); memset(V, 0, (unsigned long long)vector_count(V) * (sizeof *(V))); free(vector_get_header(V)); (V) = NULL; } } while (0)

void layec_vector_maybe_expand(void** vector_ref, long long element_size, long long required_count)
{
    if (required_count <= 0) return;
    
    struct layec_vector_header* header = vector_get_header(*vector_ref);
    if (!*vector_ref)
    {
        long long initial_capacity = 32;
        void* new_data = malloc((sizeof *header) + (unsigned long long)(initial_capacity * element_size));
        header = (layec_vector_header*)new_data;

        header->capacity = initial_capacity;
        header->count = 0;

    }
    else if (required_count > header->capacity)
    {
        while (required_count > header->capacity)
            header->capacity *= 2;
        header = (layec_vector_header*)realloc(header, (sizeof *header) + (unsigned long long)(header->capacity * element_size));
    }
    
    *vector_ref = (void*)(header + 1);
}
```

This is enough for most simple uses of a vector. And what really draws my attention here are 1) it's of course just an array, but the metadata is _hidden_; 2) it uses macros for most of the implementation, so it's light-weight and fast.

## Vector Structure

Since `vector(T)` is expanded to `T*`, what we have here is a simple array of the defined type. But in order to keep track of the capacity of the array and the current count, so it can perform operations like resizing, it's necessary to store that information somewhere. The struct `layec_vector_header` contains all the information needed.

Looking at the `layec_vector_maybe_expand` function will give us all clues to where that header is stored. In the first condition there's the logic for creating a new vector and on the else routine for expanding it:

```c
long long initial_capacity = 32;
void* new_data = malloc((sizeof *header) + (unsigned long long)(initial_capacity * element_size));
header = (layec_vector_header*)new_data;

header->capacity = initial_capacity;
header->count = 0;
```

It allocates the space for the header and then the space for the initial capacity of that type, then it returns the address of the element zero:

```c
*vector_ref = (void*)(header + 1);
```

After the creation, the layout in memory is like this:

$$
\def\\arraystretch{1.1}
\begin{array}{c:c:c:c:c:c}
    & \downarrow \\\ \hline
    header & 0 & 1 & 2 & ... & 31 \\\ \hline
    \begin{bmatrix} 32 & 0 \end{bmatrix} &  \\\
\end{array}
$$

It's clear when we expand the macro `vector_get_header`:

```c
(((struct layec_vector_header*)(V)) - 1)
```

Which pretty much casts the array to `struct layec_vector_header*` and then subtracts 1. Pointer arithmetics in C is always relative to the type. That way it get access to the pointer of the header.

Expanding logic rely basically on `realloc`:

```c
while (required_count > header->capacity)
    header->capacity *= 2;
header = (layec_vector_header*)realloc(header, (sizeof *header) + (unsigned long long)(header->capacity * element_size));
    }
```

It doubles the capacity until it satisfy the new required count and then reallocates the new size. Important to know that [realloc] will try to allocate more bytes in the same memory block first, but more commonly will allocate a new memory block, copy the contents from the previous one, freeing it,  and return the new address. That's why `layec_vector_maybe_expand` accepts a `void** vector_ref` (pointer of a pointer), because the address of the vector can change. That is a very important information because if multiple parts of the code need direct access to that vector, they'll have to store a pointer to the vector.

## Push/Pop Operations

Let's expand these macros:

```c
// vector_push(V, E)
do 
{ 
    layec_vector_maybe_expand((void**)&(V), (long long)sizeof *(V), vector_count(V) + 1);
    (V)[vector_count(V)] = E;
    vector_get_header(V)->count++; 
} while (0)

// vector_pop(V)
do 
{
    if (vector_get_header(V)->count)
        vector_get_header(V)->count--;
} while (0)
```

Interesting to note that both use **do...while** blocks. That's a litte trick to make the call of the macro look more similar to a function. There's a better explanation [in this article](https://hownot2code.wordpress.com/2016/12/05/do-while-0-in-macros/), but it's besically because of the `;` token in braceless conditional statements. If `vector_pop(V)` was defined with simple blocks like this:

```c
#define vector_pop(V) { if (vector_get_header(V)->count) vector_get_header(V)->count--; }
```

If you ever used that within a condition like this:

```c
if (something) 
    vector_pop(vec);
else 
    foo();
```

That would expand to:

```c
if (something) 
    { if (vector_get_header(V)->count) vector_get_header(V)->count--; };
else 
    foo();
```

See the `;` at the end of the macro expansion? That would raise an error and it wouldn't compile, forcing you to remove the semicolon to the end of the `vector_pop(vec)`, which definitely looks wrong.

Back to the operations, `vector_push` will first make a call to `layec_vector_maybe_expand`, which will ensure the vector exists and has enough allocated size to insert the new value, then `(V)[vector_count(V)] = E;` will get the current count (which is also the index of the possible next item in the vector) and set that to be the inserted value `E`, then lastly increment the counter. `vector_pop` will literally only decrement that counter, so the next insertion overrides the one we're deleting.

## Usage

The usage is pretty straightforward:

```c
// new int vector
vector(int) v = NULL;

// inserting values in it
for (int i = 0; i < 20; i++)
{
    vector_push(v, i * 2);
}

printf("Vector count: %lld\n", vector_count(v)); // 'Vector count: 20'
printf("Value: %d\n", v[9]); // 'Value: 18'

// direct access is also possible:
v[9] = 99;

printf("Value: %d\n", v[9]); // 'Value: 99'

// however there's no boundary-checking
// values out of count won't be aknowledged
v[21] = 30;

// out of capacity is just a regular random memory access
// thus undefined behavior
v[35] = 30;

printf("Vector count: %lld\n", vector_count(v)); // 'Vector count: 20'

// deleting last element
vector_pop(v);

printf("Vector count: %lld\n", vector_count(v)); // 'Vector count: 19'

// freeing the vector
vector_free(v);
```

The example usage along with the vector code can be found in the [compiler explorer](https://godbolt.org/z/so3onfn3Y), where you can play with it, see the post-processed code and execute it.
