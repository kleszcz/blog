---
layout:     post
title:      UTF conversions in C++ 11
date:       2017-02-28 20:40:00
author:     "Jan Kleszczyński"
---

## Converting strings on Windows

Text is almost always an important part of a game. To achieve better recognition and reach audience as big as posible we translate
content to many languages, some of them (including mine) have more glyphs than English so we need a way to encode those,
and ASCII isn't a good option. But there are many Unicode standards which are widly used and the most common seems to be UTF-8.

In Windows API uses either Multibyte Character Set (which is not a UTF-8) or UTF-16 to represent localized strings.
To use UTF-8 one needs to convert MBCS string to UTF-16 using system function `MultiByteToWideChar` and use it with wide character functions
(e.g. `MessageBoxW` instead of `MessageBoxA`). And calling it is very "C":

```C++
LPCSTR c_buff = u8"utf-8 text \u0105\u0107\u0119\u015B";  // use of u8 literal to force utf8 encoding 
int buff_len = lstrlen(c_buff);

int size = MultiByteToWideChar(                           // how many wide chars we need
  CP_UTF8, 0, c_buff, buff_len, NULL, 0
);

size += 1;                                                // add one
buff_len += 1;                                            // for null char

LPWSTR w_buff = (LPWSTR)malloc(sizeof(WCHAR) * (size));   // allocate wide char buffer
ZeroMemory(w_buff, size);

MultiByteToWideChar(                                      // convert UTF-8 to UTF-16
  CP_UTF8, 0, c_buff, buff_len, w_buff, size
);

MessageBoxW(NULL, (LPCWSTR)w_buff, NULL, 0);              // use it
free(w_buff);
```

Message in the box should be "utf-8 text ąćęś". Ok, so it is a lot of work to convert strings, at least enought to create a single
header lib like one by [Randy Gaul](http://www.randygaul.net/2017/02/23/game-localization-and-utf-8/).
But there is another way to do it and it's in the C++11 standard!

## C++ Localization library

Localization library is being used for many things, you can find it [here](http://en.cppreference.com/w/cpp/locale),
but since C++11 two important templates have beed added: `std::wstring_convert` and `std::codecvt_utf8_utf16`.
Class `wstring_convert` has methods `to_bytes`, returning `std::string` and `from_bytes` which return type depends
on `wstring_convert`'s second template parameter.
As a first parameter one needs to pass a conversion facet type defining how to perform a conversion. And that is where
`std::codecvt_utf8_utf16` goes.

To use standard's conversion two headers are needed. First one is `<locale>` for converter class* and the second
is `<codecvt>` for conversion facet.

\* However, on MS Visual Studio 2015 it works without including `<locale>`.

Below is a simple implementation of helper functions which I now use for string encoding conversions.

```c++
#include <locale>
#include <codecvt>

namespace utf_converter
{       
    using converter = std::wstring_convert<std::codecvt_utf8_utf16<wchar_t>, wchar_t>;

    std::string utf16_to_utf8(std::wstring utf16_string)
    {
        return converter{}.to_bytes(utf16_string);
    }

    std::wstring utf8_to_utf16(std::string utf8_string)
    {
        return converter{}.from_bytes(utf8_string);
    }
}
```
