# Your First Project
Starting out with C3, you probably want to get a feel for the language, without using the integrated build system.

Open a text editor and enter the following in a file you call `hello_world.c3`:
```rust
module hello_world;
import std::io;

fn int main(String[] argv)
{
    io::printn("Hello World!");
    return 0;
}
```

Now in the terminal type:
```bash
$ c3c compile hello_world.c3
$ ./hello_world
Hello World
```

# A real project
Once you go beyond simple files, you want to create a real project. Do so by entering `c3c init hello_world`.

You will get the following structure:
```bash
$ c3c init hello_world
$ tree .
.
└── hello_world
    ├── project.json
    ├── LICENSE
    ├── README.md
    ├── build
    ├── docs
    ├── lib
    ├── resources
    ├── src
    │   └── main.c3
    └── test
```

Enter main.c3 and write the same code as above, then anywhere in the project structure:
```bash
$ c3c run
Hello World
```
