# Hello World in x86 Assembly

## Prerequisites

1. nasm (`brew install nasm`)

## How To

```
nasm -f macho64 helloworld.asm
ld -o helloworld helloworld.o /usr/lib/libc.dylib
```

## References
- https://padamthapa.com/blog/my-first-x86-64-assembly-in-macos/
- https://medium.com/@thisura1998/hello-world-assembly-program-on-macos-mojave-d5d65f0ce7c6
- https://pastebin.com/x2pmuF4a

