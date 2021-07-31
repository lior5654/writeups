## lior@wildboar:/# ls

[Google CTF 2021](./2021-google-ctf/index.md)
[CryptoCTF 2021](./2021-crypto-ctf/index.md)

## lior@wildboar:/# cat ./page_description.cpp

```c++
signed main(void)
{
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);
    std::cout << "Main Menu" << '\n';
    std::cout << "press the name of a CTF in the above list if you wish to see" <<
    " a writeup for a challenge that appeared on that CTF." << '\n';
    
    return 0x1337;
}
```
