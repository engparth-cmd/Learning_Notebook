# Learning_Notebook

Practical Ubuntu commands and beginner OpenFOAM lessons.

## Start here

- [Ubuntu and Git notes](learning.txt): existing file-search and Git notes.
- [Ubuntu application commands](ubuntu_command.txt): existing RustDesk installation and IP-address commands.
- [OpenFOAM Lesson 1: blockMeshDict](openfoam/01-blockMeshDict.md): build a small mesh, interpret its bounding box, and calculate cell sizes.

The existing command notes remain in their original files so there is one place to maintain each topic. The Git note currently demonstrates a token embedded in a remote URL; prefer a credential manager or SSH rather than storing a token in a URL.

## Learning sequence

1. Mesh geometry and resolution: the lesson linked above.
2. Next: case folders, initial fields, and boundary conditions.
3. Later: run a solver, inspect results, and compare mesh resolutions.

Only add a new lesson when its content is ready. Keep worked examples with the lesson until a reusable complete simulation case is needed.

## Working rule

Run examples in a separate working directory. Keep generated meshes, time directories, and logs outside this notes repository.
