<!-- Improved compatibility of back to top link: See: https://github.com/othneildrew/Best-README-Template/pull/73 -->
<a id="readme-top"></a>
<!-- PROJECT SHIELDS -->
<!--
*** I'm using markdown "reference style" links for readability.
*** Reference links are enclosed in brackets [ ] instead of parentheses ( ).
*** See the bottom of this document for the declaration of the reference variables
*** for contributors-url, forks-url, etc. This is an optional, concise syntax you may use.
*** I left this header up here to give credit where credit is do for this nice Read.me template.
*** https://www.markdownguide.org/basic-syntax/#reference-style-links
-->
[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![project_license][license-shield]][license-url]
[![LinkedIn][linkedin-shield]][linkedin-url]

<!-- PROJECT Title -->
<div align="center">
<h3 align="center">cAlgoImplementations</h3>
  <p align="center">
    Implementations of some basic libraries that I like to use as starting points for various projects. 
    <br />
    <a href="https://github.com/indyIOT/cAlgoImplementations/rissues/new?labels=bug&template=bug-report---.md">Report Bug</a>
    &middot;
    <a href="https://github.com/indyIOT/cAlgoImplementations/issues/new?labels=enhancement&template=feature-request---.md">Request Feature</a>
  </p>
</div>

<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#built-with">Built With</a></li>
        <li><a href="#repository-layout">Repository Layout</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a>
        <ul>
        <li><a href="#adding-a-module">Adding A Module</a></li>
        <ul>
            <li><a href="#wiring-in-an-existing-github repo">Wiring in an existing GitHub repo</a></li>
            <li><a href="#scaffolding-a-brand-new-module">Scaffolding a brand-new module</a></li>
        </ul>
        <li><a href="#removing-a-module">Removing a Module</a></li>
        <li><a href="#building-an-individual-module">Building an Individual Module</a></li>
      </ul>
    </li>
    <li><a href="#top-contributors">Top Contributors</a></li>
    <li><a href="#change-Log">Change Log</a></li>
    <li><a href="#license">License</a></li>
    <li><a href="#contact">Contact</a></li>
  </ol>
</details>

<!-- ABOUT THE PROJECT -->
## About The Project
CMake project that is an overall repository that pulls all of my basic mocules and algos used for different independant projects. This allows me to work on them all at the same time with the proper folder
structure. It is a monorepo of independent git submodules, one per library/module, plus a couple of shared pieces every module depends on:

### Repository Layout
| Path                    | What it is                                                              |
|--------------------------|--------------------------------------------------------------------------|
| `cCommonMacros/`         | Shared, header-only macros (`AG_UNUSED`, `AG_DEBUG_ASSERT`, etc.). Every module depends on this. |
| `cCommonTypes/`          | Shared, header-only types (`sErrorCompact_t`, callback typedefs, CRC configs, etc.). Every module depends on this. |
| `c<Name>Driver/`         | An individual driver/library module (`cErrorDriver`, `cIntegrityDriver`, `cMutexDriver`, ...). |
| `basicTestApplication/`  | Scratch application for exercising libraries together. |
| `projectTools/`          | Cross-module tooling, e.g. `cModulesConst.py` (see below). |
| `docs/coding standard.txt` | The C coding standard all module source is expected to follow. |

Each module is its own git repository (own history, own GitHub remote) pulled in here as a
git submodule, so the module can also be cloned and built completely on its own.

### Built With

* [![CMake][CMake.js]][CMake-url]

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- Usage -->
## Usage
Since this is just a repo of repos Using it is regulated to adding, removing and building modules.

### Adding a Module

There are two cases: the module's repository already exists on GitHub (just needs to be wired
in), or you're starting a brand-new module from scratch.

#### Wiring in an Existing GitHub Repo

```bash
git submodule add git@github.com:indyIOT/<repo-name>.git <repo-name>
git commit -m "Add <repo-name> as submodule"
```

Use `indyIOT` as the org for library/module repos -- that's the org every
existing module in `.gitmodules` actually points to. Keep `<folder-name>` identical to
`<repo-name>` (every existing entry does this; it's what the rest of this doc, and every
module's own relative `../cCommonMacros` / `../cCommonTypes` includes, assume).

After adding, verify it actually cloned content and not just an empty gitlink:
```bash
git submodule status <repo-name>   # leading "-" means not initialized -- run `git submodule update --init <repo-name>`
```
<p align="right">(<a href="#readme-top">back to top</a>)</p>

#### Scaffolding a New Module

Follow the shape of an existing module (`cErrorDriver` and `cIntegrityDriver` are the cleanest
references) rather than inventing a new layout:

```
c<Name>Driver/
├── cMakeLists.txt              <- lowercase 'c' -- yes, really, matches every other module
├── Makefile                    <- `prepare` target: wipes/recreates build/
├── README.md
├── .gitignore                  <- copy from an existing module
├── app/
│   ├── cMakeLists.txt
│   └── source/main.c           <- dev/demo entry point, wires up fake callbacks and calls the lib
└── c<Name>DriverLib/
    ├── CMakeLists.txt          <- standard casing here (only the module-root one is lowercase)
    ├── generateVersion.cmake   <- generates <publicInclude>/generated/c<Name>DriverVersion.h at build time
    ├── include/                <- private headers (module-internal only)
    ├── publicInclude/
    │   ├── c<Name>DriverPub.h  <- public API: types, function prototypes, config macros
    │   └── c<Name>DriverConfig.h <- overridable compile-time config (guarded by CUSTOM_<NAME>_DRIVER_CONFIG)
    ├── source/
    │   └── c<Name>Driver.c     <- implementation
    └── unittest/
        └── c<Name>Driver_test.cpp  <- gtest; only compiled in if UNIT_TESTS is ON
```

Key conventions to match (don't deviate without a reason):

1. **Module identity.** Every `.c` file gets a private module name/ID pair right after its
   `Static Global Variables` section comment:
   ```c
   static const uint8_t moduleName[] = "c<Name>Driver";
   #define MODULE_ID <crc16-ccitt-false of the filename>
   ```
   Don't hand-compute the ID -- run the tooling from the repo root after adding your `.c`
   files:
   ```bash
   python projectTools/cModulesConst.py
   ```
   It scans for `.c` files, and for any missing/blank `moduleName`/`MODULE_ID` it inserts the
   filename and the CRC16-CCITT-FALSE of the filename right after that anchor comment. It skips
   `app/`, `build/`, `unittest/`, and a few other directories, so only real library source gets
   touched.

2. **Shared dependencies via relative path, not submodules-of-submodules.** A module's
   `cMakeLists.txt` pulls in the two shared libs like this (copy verbatim, just don't nest
   another `.gitmodules` inside a module):
   ```cmake
   add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/../cCommonMacros ${CMAKE_CURRENT_BINARY_DIR}/cCommonMacros)
   add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/../cCommonTypes ${CMAKE_CURRENT_BINARY_DIR}/cCommonTypes)
   ```
   This means a module only builds standalone if it's checked out as a sibling of
   `cCommonMacros`/`cCommonTypes` -- i.e. inside this monorepo (or another checkout with the
   same layout). That's expected.

3. **Test-only hooks are opt-in via `UNIT_TESTS`.** If your driver is a static singleton (most
   are -- see `cErrorDriver`/`cIntegrityDriver`'s `THIS` pattern), add a
   `reset<Name>DriverForTest()` function guarded by `#ifdef UNIT_TESTS` in both the public
   header and the source file, and propagate the define from the library's `CMakeLists.txt`:
   ```cmake
   if( UNIT_TESTS )
       target_compile_definitions(${<NAME>_DRIVER_LIBRARY_NAME} PUBLIC UNIT_TESTS)
   endif()
   ```
   `PUBLIC` matters here -- it's what lets the unit test executable (which links against the
   library) see `UNIT_TESTS` too. Without a reset hook, only the *first* gtest case that calls
   `init<Name>Driver()` in the whole test binary will ever see it succeed.

4. **`CREATE_ERROR` must be safe before init.** Any macro/helper a module uses to build an
   `sErrorCompact_t` (see `cIntegrityDriver.h`'s `CREATE_ERROR`) must not dereference a
   callback function pointer that's only set once `init<Name>Driver()` has run. Route it
   through a small helper that falls back to filling the struct directly (still with a real
   CRC16 over it) when the driver isn't initialized yet, rather than crashing on a null
   function pointer.

5. **Follow `docs/coding standard.txt`.** Doxygen headers on every function (prototype *and*
   implementation), Allman braces, `/* */` comments, single return point, no magic numbers
   except `0`, `s`/`e` type prefixes, etc.

6. **Version header is generated, not committed.** `generateVersion.cmake` regenerates
   `publicInclude/generated/c<Name>DriverVersion.h` on every build (timestamp changes every
   time), so it should be `.gitignore`d, not checked in.

Once the module builds and its tests pass on their own, push it to a new `indyIOT/c<Name>Driver`
GitHub repo and wire it in per **section A** above.
<p align="right">(<a href="#readme-top">back to top</a>)</p>

### Removing a Module

Removing a submodule is not just deleting the folder -- git tracks it in three places
(`.gitmodules`, the superproject's `.git/config`, and `.git/modules/<name>`), and skipping any
of them leaves a broken half-removed submodule behind. Do it in this order:

```bash
# 1. Unregister it locally and wipe its working directory
git submodule deinit -f <module-name>

# 2. Remove it from the index and .gitmodules
git rm -f <module-name>

# 3. Clean up its leftover internal git data (deinit doesn't always remove this)
rm -rf .git/modules/<module-name>

# 4. Confirm nothing else in the tree still references it
git grep -il "<module-name>" -- ':!*/build/*'
```

Step 4 matters: if another module's `CMakeLists.txt`/`cMakeLists.txt` ever added the removed
module as a dependency (`add_subdirectory`, `target_link_libraries`), that reference has to be
removed too or the dependent module will fail to configure. None of the current modules
cross-depend on each other beyond `cCommonMacros`/`cCommonTypes`, so this is normally a no-op,
but check anyway.

```bash
git status --short   # should show "D <module-name>" and "M .gitmodules"
git commit -m "Remove <module-name>"
```

**Note:** if a submodule was ever manually `git clone`d into place instead of added via
`git submodule add` (it happens -- `git submodule status` shows a leading `-` for these,
meaning git considers them uninitialized even though the folder has content), steps 1-3 above
still apply and still work; `git submodule deinit -f` will absorb its embedded `.git` directory
first before cleaning it up.
<p align="right">(<a href="#readme-top">back to top</a>)</p>

### Building an Individual module

Every module is self-contained once its shared-lib siblings are present:
```bash
cd c<Name>Driver
mkdir build && cd build
cmake ..
cmake --build .
```
or use the module's own `Makefile` (`make prepare`) to just get a clean `build/` directory.
Unit tests build to `cErrorDriverTest.exe`, run via
`build/c<Name>DriverLib/Debug/c<Name>DriverTest.exe --gtest_color=yes`.

<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- Top Contributors -->
## Top Contributors

<a href="https://github.com/indyIOT/cAlgoImplementations/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=indyIOT/cAlgoImplementations" alt="contrib.rocks image" />
</a>
<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- Changelog -->
## Change Log
<p align="right">(<a href="#readme-top">back to top</a>)</p>

<!-- LICENSE -->
## License

Distributed under the project_license. See `LICENSE` for more information.

<p align="right">(<a href="#readme-top">back to top</a>)</p>



<!-- CONTACT -->
## Contact

Project Link: [https://github.com/indyIOT/cAlgoImplementations](https://github.com/indyIOT/cAlgoImplementations)

<p align="right">(<a href="#readme-top">back to top</a>)</p>


<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->
[contributors-shield]: https://img.shields.io/github/contributors/indyIOT/cAlgoImplementations.svg?style=for-the-badge
[contributors-url]: https://github.com/indyIOT/cAlgoImplementations/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/indyIOT/cAlgoImplementations.svg?style=for-the-badge
[forks-url]: https://github.com/indyIOT/cAlgoImplementations/network/members
[stars-shield]: https://img.shields.io/github/stars/indyIOT/cAlgoImplementations.svg?style=for-the-badge
[stars-url]: https://github.com/indyIOT/cAlgoImplementations/stargazers
[issues-shield]: https://img.shields.io/github/issues/indyIOT/cAlgoImplementations.svg?style=for-the-badge
[issues-url]: https://github.com/indyIOT/cAlgoImplementations/issues
[license-shield]: https://img.shields.io/github/license/indyIOT/cErrorDriver.svg?style=for-the-badge
[license-url]: https://github.com/indyIOT/cAlgoImplementations/blob/main/LICENSE
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://www.linkedin.com/in/anthonygarza2020

<!-- Shields.io badges. You can a comprehensive list with many more badges at: https://github.com/inttter/md-badges -->
[CMake.js]: https://img.shields.io/badge/CMake-064F8C?style=for-the-badge&logo=cmake&logoColor=fff
[CMake-url]: https://cmake.org/
