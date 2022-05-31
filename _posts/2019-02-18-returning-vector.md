---
layout:     post
title:      Returning std::vector      
date:       2019-02-18 20:40:00
author:     "Jan Kleszczyński"
---
# Returning std::vector

## A pretty long introduction

In a past two years I've been moving from being a programmer to a product manager job
and my engagement in code development has dropped significantly.
At work I've moved form C# to Python and limit my self to simple tools helping me with the project management.
Less coding means less interesting or unexpected cases to happen therefore less topics to write about. But couple days ago I've rebooted my homebrewed game engine - nothing fancy just a simple engine where I can test new libraries like [Dear-ImGUI](https://github.com/ocornut/imgui) for developer UI or [V-HACD](https://github.com/kmammou/v-hacd) for mesh convex decomposition (I know that these libraries aren't new but they're new to me). Also it's a great opportunity to master code architecture and topics like Data-oriented Design, Data Driven Development or automated test in game development. Anyway I hope that I'll have much more to write about while constantly pushing forward my engine.

I've decided to put my core engine parts in a separate library and recently I've moved from static to dynamic. Using CMake it's a pretty easy task. At least I've thought so. Since I'm at the beginning of development I've decided to stick to STL for my containers and smart pointers (yes, I know it's not a popular choice in games) and I was very surprised when I've realized that using STL containers in my classes have created bunch of linker warnings. That make me think how to get rid of ```std::vector``` in a function when I'm returning an array of elements. I've thought about something like ```vector_view``` with only a size and a pointer but it would work only if I want to return whole vector. I haven't found any way for a selection of elements but I've started wonder what's the best way to return a vector with selection of elements of another vector.

## Returning selection of elements from a vector
Let''s assume that we've a 1000 element vector of ints and we want to return copy of all even elements. I've come up with three different schemas:

1. Create new vector, fill it with data, return; possible many allocations
``` c++
std::vector<int> getEvenPushBack()
{
  std::vector<int> r;
  for(auto v : vec)
  {
    if(v % 2 == 0)
      r.push_back(v);
  }
  return r;
}
```
2. Preallocate new vector for size of the given one, fill with data, resize to fit, return; one big allocation and one deallocation
``` c++
std::vector<int> getEvenReserveResize()
{
  std::vector<int> r(vec.size());
  std::size_t i = 0;
  for(auto v : vec)
  {
    if(v % 2 == 0)
       r[i++] = v;
  }
  r.resize(i);
  return r;
}
```
3. Count how many elements will qualify, preallocate for that exact number, fill with data, return; one allocation only but two loops
``` c++
std::vector<int> getEvenCountReserve()
{
  std::size_t s = 0;
  for(auto v : vec)
  {
    if(v % 2 == 0)
      s++;
  }
  std::vector<int> r(s);
  std::size_t i = 0;
  for(auto v : vec)
  {
    if(v % 2 == 0)
      r[i++] = v;
  }
  return r;
}
```
I was sure that the 3rd option will be the fastest. But after I've run a [benchmark](http://quick-bench.com/xzXteH9AqvAKL9DaskM19gDMSGA) it comes out that having loop twice over the same continuous memory is worse than a allocation/deallocation pair. Of course trivial push back method was the worst.

<img src="/blog/img/2019-02-18-benchmark.png" alt="Benchmark chart. PushBack is the slowest, ContReserve is faster but ReserveResize is the fastes." class="inline"/>
